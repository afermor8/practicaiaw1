---
title: "LDAPs"
date: 2023-06-19T10:53:33+02:00
draft: true
tags: ["ASO"]
bigimg: [{src: "/img/triangle.jpg", desc: "Triangle"}, {src: "/img/sphere.jpg", desc: "Sphere"}, {src: "/img/hexagon.jpg", desc: "Hexagon"}]
---


## Configura el servidor LDAP de alfa para que utilice el protocolo ldaps:// a la vez que el ldap:// utilizando el certificado x509 de la práctica de https o solicitando el correspondiente a través de gestiona. Realiza las modificaciones adecuadas en los clientes ldap de alfa para que todas las consultas se realicen por defecto utilizando ldaps://


### Certificado x509

**En Alfa (servidor LDAP):**

Generamos la clave privada y el fichero csr.

```bash
sudo su
cd
openssl genrsa 4096 > /etc/ssl/private/alfa.key
openssl req -new -key /etc/ssl/private/alfa.key -out alfa.csr
```

Rellenamos los campos que nos aparecen de la siguiente forma:

```bash
...
Country Name (2 letter code) [AU]:ES
State or Province Name (full name) [Some-State]:Sevilla
Locality Name (eg, city) []:Dos Hermanas
Organization Name (eg, company) [Internet Widgits Pty Ltd]:IES Gonzalo Nazareno
Organizational Unit Name (eg, section) []:Informática
Common Name (e.g. server FQDN or YOUR name) []:alfa.arantxa.gonzalonazareno.org
Email Address []:ara.fer.mor@gmail.com

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:
...
```

Lo copiamos en nuestro host para mandarselo a la unidad certificadora.

```bash
scp alfa.csr arantxa@172.29.0.46:.
```

Ahora sí, mandamos el fichero alfa.csr al IES Gonzalo Nazareno para que nos devuelva el certificado firmado. Se sube el correspondiente fichero csr mediante la aplicación [Gestiona](https://dit.gonzalonazareno.org/gestiona), en la barra de menú > Utilidades > Certificados. Se tiene que subir como Certificado de equipo.

Una vez firmado el fichero `.csr`, aparecerá un fichero con extensión `.crt` que se corresponde con el certificado firmado por la autoridad certificadora del Gonzalo Nazareno.

Además me tengo que descargar el certificado de la CA para que se compruebe la firma. El fichero descargado se llama gonzalonazareno.crt

```
wget https://dit.gonzalonazareno.org/gestiona/info/documentacion/doc/gonzalonazareno.crt
```

Una vez lo tengamos todo nos lo enviamos desde nuestro host a la máquina alfa y lo guardamos en /etc/ssl/certs/.

```bash
#Desde nuestro host
scp alfa.crt arantxa@172.22.201.158:.
scp gonzalonazareno.crt arantxa@172.22.201.158:.
```

```bash
#En Alfa
cp /home/arantxa/alfa.crt /root/         
cp /home/arantxa/gonzalonazareno.crt /root/         #Los he copiado a /root para tener una copia disponible
mv /home/arantxa/alfa.crt /etc/ssl/certs/
mv /home/arantxa/gonzalonazareno.crt /etc/ssl/certs/
```

Para que el usuario LDAP pueda hacer uso de la clave, vamos a añadir ACL’s que le permitirán al usuario openldap tener acceso a ambos ficheros. Para ello, ejecutaremos lo siguiente:

```bash
setfacl -m u:openldap:r-x /etc/ssl/private
setfacl -m u:openldap:r-x /etc/ssl/private/alfa.key
getfacl /etc/ssl/private 
```

Salimos de root.

```bash
exit
cd /home/arantxa/ldap
```

### Configuración del servidor LDAP para que utilice SSL/TLS

Creamos un fichero de entrada que contiene lo siguiente:

```bash
nano ldap_config.ldif
```

```txt
dn: cn=config
changetype: modify
replace: olcTLSCACertificateFile
olcTLSCACertificateFile: /etc/ssl/certs/gonzalonazareno.crt
-
replace: olcTLSCertificateKeyFile
olcTLSCertificateKeyFile: /etc/ssl/private/alfa.key
-
replace: olcTLSCertificateFile
olcTLSCertificateFile: /etc/ssl/certs/alfa.crt
```

Aplicamos la modificacion:

```bash
ldapmodify -Y EXTERNAL -H ldapi:/// -f ldap_config.ldif
```

Para que alfa use el protocolo ldaps por defecto, modificamos /etc/default/slapd, de forma que el servidor LDAP escuche en el puerto 636.

```bash
sudo nano /etc/default/slapd
```

```conf
SLAPD_SERVICES="ldaps:/// ldapi:///"
```

Añadimos a /etc/ldap/ldap.conf lo siguiente:

```bash
nano /etc/ldap/ldap.conf
```

```conf
BASE=dc=arantxa,dc=gonzalonazareno,dc=org
URI=ldaps://alfa.arantxa.gonzalonazareno.org
```

Reiniciamos el servicio.

```bash
sudo systemctl restart slapd
```

Para comprobar que el puerto esta escuchando, ejecutamos el siguiente comando:

```bash
sudo netstat -putanl | grep slapd
```

Copiamos el certificado del CA del centro en el directorio /usr/local/share/ca-certificates/ añadiéndolo a la lista de certificados de confianza.

```bash
cp /etc/ssl/certs/gonzalonazareno.crt /usr/local/share/ca-certificates/
```

Actualizamos la lista de certificados de confianza.

```bash
update-ca-certificates
```

Realizamos una consulta para comprobar que funciona y nos devuelve resultados:

```bash
ldapsearch -x -b "dc=arantxa,dc=gonzalonazareno,dc=org"
ldapsearch -x -b "dc=arantxa,dc=gonzalonazareno,dc=org" -H ldaps://
```

