---
title: "LDAPs"
date: 2023-06-19T10:53:33+02:00
draft: true
tags: ["ASO"]
bigimg: [{src: "/img/triangle.jpg", desc: "Triangle"}, {src: "/img/sphere.jpg", desc: "Sphere"}, {src: "/img/hexagon.jpg", desc: "Hexagon"}]
---


## Configura el servidor LDAP de alfa para que utilice el protocolo ldaps:// a la vez que el ldap:// utilizando el certificado x509 de la práctica de https o solicitando el correspondiente a través de gestiona. Realiza las modificaciones adecuadas en los clientes ldap de alfa para que todas las consultas se realicen por defecto utilizando ldaps://

------------------------------------

### 0. Escenario

Hay que tener en cuenta que parto de un escenario ya montado que se puede ver en los siguientes posts:

- [Instalación y configuración inicial de Openldap](https://afermor8.github.io/post/ldap1/)
- [Poblar un directorio LDAP desde un fichero CSV](https://afermor8.github.io/post/ldap2/)

### 1. Certificado x509

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
scp Descargas/alfa.crt arantxa@172.22.201.158:.
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
apt install acl
setfacl -m u:openldap:r-x /etc/ssl/private
setfacl -m u:openldap:r-x /etc/ssl/private/alfa.key
setfacl -m u:openldap:r-x /etc/ssl/certs
```

Comprobar:

```bash
root@alfa:~$ getfacl /etc/ssl/private 
getfacl: Removing leading '/' from absolute path names
# file: etc/ssl/private
# owner: root
# group: root
user::rwx
user:openldap:r-x
group::---
mask::r-x
other::---

root@alfa:~$ getfacl /etc/ssl/certs
getfacl: Removing leading '/' from absolute path names
# file: etc/ssl/certs
# owner: root
# group: root
user::rwx
user:openldap:r-x
group::r-x
mask::r-x
other::r-x
```

Salimos de root.

```bash
exit
```

### 2. Configuración del servidor LDAP para que utilice SSL/TLS

Creamos un fichero de entrada que contiene lo siguiente:

```bash
cd /home/arantxa/ldap
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
sudo ldapmodify -Y EXTERNAL -H ldapi:/// -f ldap_config.ldif
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
sudo nano /etc/ldap/ldap.conf
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

```bash
tcp        0      0 0.0.0.0:636             0.0.0.0:*               LISTEN      62575/slapd         
tcp6       0      0 :::636                  :::*                    LISTEN      62575/slapd  
```

Copiamos el certificado del CA del centro en el directorio /usr/local/share/ca-certificates/ añadiéndolo a la lista de certificados de confianza.

```bash
sudo cp /etc/ssl/certs/gonzalonazareno.crt /usr/local/share/ca-certificates/
```

Actualizamos la lista de certificados de confianza.

```bash
sudo update-ca-certificates
```

Realizamos una consulta para comprobar que funciona y nos devuelve resultados:

```bash
ldapsearch -x -b "dc=arantxa,dc=gonzalonazareno,dc=org"
```

Pero si realizamos la consulta desde los clientes no funcionará ya que no los tenemos configurados.

![fallo-consulta-ldap-clientes](/img/ldap3/1.png)

### 3. Configuración del cliente Delta

Primero desde alfa pasamos por scp el certificado de la autoridad certificadora (CA) al cliente Delta.

```bash
sudo cp /root/gonzalonazareno.crt .
scp gonzalonazareno.crt arantxa@192.168.0.3:.
```

Copiamos el certificado de la CA en el directorio /usr/local/share/ca-certificates/.

```bash
sudo cp gonzalonazareno.crt /usr/local/share/ca-certificates/
```

Actualizamos la lista de certificados de confianza.

```bash
sudo update-ca-certificates
```

Modificamos el fichero de configuración del cliente ldap.

```bash
sudo nano /etc/ldap/ldap.conf 
```

```conf
URI ldaps://alfa.arantxa.gonzalonazareno.org
```

Y ya se podrá hacer una consulta al directorio ldap a través usando LDAPs.

```bash
ldapsearch -x -b "dc=arantxa,dc=gonzalonazareno,dc=org"
```

![consulta-delta](/img/ldap3/3.png)

### 4. Configuración del cliente Bravo

Pasamos el certificado de la CA a Bravo mediante scp.

```bash
scp gonzalonazareno.crt arantxa@172.16.0.200:.
```

En Bravo copiamos el certificado en la lista de certificados de confianza.

```bash
sudo cp gonzalonazareno.crt /usr/share/pki/ca-trust-source/anchors/
sudo chown root: /usr/share/pki/ca-trust-source/anchors/gonzalonazareno.crt
```

Actualizamos la lista de crtificados de confianza.

```bash
sudo update-ca-trust
```

Cambiamos el fichero de configuración de ldap.

```bash
sudo nano /etc/openldap/ldap.conf
```

```bash
URI ldaps://alfa.arantxa.gonzalonazareno.org
```

En el fichero sssd.conf modificamos la URI del servidor LDAP (le añadimos la "s") y agregamos la configuración TLS. El fichero quedaría de la siguiente forma:

```bash
sudo nano /etc/sssd/sssd.conf
```

```conf
[domain/default]
id_provider = ldap
autofs_provider = ldap
auth_provider = ldap
chpass_provider = ldap
ldap_uri = ldaps://alfa.arantxa.gonzalonazareno.org
ldap_search_base = dc=arantxa,dc=gonzalonazareno,dc=org
ldap_id_use_start_tls = True
ldap_tls_cacertdir = /etc/openldap/cacerts
cache_credentials = True
ldap_tls_reqcert = allow
debug_level = 10

ldap_host_search_base = ou=Maquinas,dc=arantxa,dc=gonzalonazareno,dc=org
ldap_host_object_class = ipHost
ldap_host_fqdn = cn

[sssd]
services = nss, pam, autofs, ssh
domains = default

[nss]
homedir_substring = /srv/homes

[ssh]
```

Reiniciamos el servicio.

```bash
sudo systemctl restart sssd
```

Y ya podremos realizar consultas:

```bash
ldapsearch -x -b "dc=arantxa,dc=gonzalonazareno,dc=org"
```

![consulta-bravo](/img/ldap3/2.png)

### 5. Extra

**Escucha en el puerto 389:**

Si intentamos conectarnos ahora al puerto 389 no se podrá, por tanto si hubiera alguna máquina sin certificado no se podría conectar.

![consulta-389](/img/ldap3/4.png)

Para que no haya problemas he añadido a /etc/default/slapd en la lista de servicios **"ldap:///"**.

```bash
sudo nano /etc/default/slapd
```

```conf
SLAPD_SERVICES="ldaps:/// ldapi:/// ldap:///"
```

Se reinicia el servidor LDAP y ya podremos hacer consultas sin certificado.

```bash
sudo systemctl reboot slapd
ldapsearch -x -b "dc=arantxa,dc=gonzalonazareno,dc=org" -H ldap://localhost:389
sudo netstat -putanl | egrep slapd
```

![consulta-389](/img/ldap3/5.png)

![netstat-389-port](/img/ldap3/6.png)

**NOTA: El cliente debería estar configurado para escuchar en ldap:// no en ldaps://.*

**NOTA 2: Si queremos comprobar el puerto por el que se conecta podemos utilizar tcpdump*

```bash
sudo tcpdump -i any tcp port 389
sudo tcpdump -i any tcp port 389 or tcp port 636
```
