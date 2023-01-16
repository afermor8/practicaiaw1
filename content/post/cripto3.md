---
title: "Criptografía III. Certificados digiales. HTTPS"
date: 2023-01-11T13:42:27+01:00
draft: true
tags: ["SAD"]
bigimg: [{src: "/img/triangle.jpg", desc: "Triangle"}, {src: "/img/sphere.jpg", desc: "Sphere"}, {src: "/img/hexagon.jpg", desc: "Hexagon"}]
---

-------------------------

## Certificado digital de persona física

-------------------------

### Tarea 1: Instalación del certificado

#### 1. Una vez que hayas obtenido tu certificado, explica brevemente como se instala en tu navegador favorito

Este punto lo explico junto al siguiente punto para mostrar las capturas.

#### 2. Muestra una captura de pantalla donde se vea las preferencias del navegador donde se ve instalado tu certificado

Una vez tenemos el certificado digital, para agregarlo a Google Chrome vamos a los tres puntos de la barra superior y hacemos click en **Configuración**. Abrimos el desplegable de la izquierda y seleccionamos **Privacidad y seguridad**. Ahí abrimos la pestaña **Seguridad** y entre las opciones que nos da seleccionamos **Gestionar certificados**. Importamos el certificado que tenemos.

![certificado](/img/cripto3/1.png)

Introducimos la contraseña del certificado y ya lo tendremos agregado.

![certificado](/img/cripto3/2.png)

Si le damos a ver el certificado comprobamos que se ha agregado correctamente mi certificado de la FNMT-RCM.

![certificado](/img/cripto3/3.png)


#### 3. ¿Cómo puedes hacer una copia de tu certificado?, ¿Como vas a realizar la copia de seguridad de tu certificado?. Razona la respuesta.

Para hacer una copia de este certificado, en la misma página anterior de configuración del navegador Chrome seleccionamos **Exportar**.

![certificado](/img/cripto3/4.png)

Seleccionamos la carpeta donde lo queremos guardar, en mi caso he elegido la carpeta **Documentos**. Introduciomos la contraseña del certificado.

![certificado](/img/cripto3/5.png)

Y ya tendremos la copia del certificado.

![certificado](/img/cripto3/6.png)


#### 4. Investiga como exportar la clave pública de tu certificado.

Para exportar la clave pública del certificado sin la clave privada lo he hecho desde Mozilla Firefox, donde también tengo instalado mi certificado. Voy a las opciones de **Ajustes** y aquí clickamos sobre **Privacidad & Seguridad**. Vamos a la opción **Ver certificados...** y se nos abre la siguiente ventana.

![certificado](/img/cripto3/7.png)

Selecciono la opción **Ver** y me aparecerá la siguiente información sobre el certificado:

![certificado](/img/cripto3/8.png)

En Misceláneo selecciono la opción **PEM (cert)** y se descargará la clave pública.

![certificado](/img/cripto3/9.png)

![certificado](/img/cripto3/10.png)


### Tarea 2: Validación del certificado

#### 1. Instala en tu ordenador el software autofirma y desde la página de VALIDe valida tu certificado. Muestra capturas de pantalla donde se comprueba la validación.

##### AutoFirma

Primero instalar los siguientes paquetes si no los tenemos.

```
sudo apt install default-jdk libnss3-tools
```

Descargo con wget la aplicación **AutoFirma** en la carpeta Documentos.

```
wget https://estaticos.redsara.es/comunes/autofirma/1/7/1/AutoFirma_Linux.zip
```

Extraemos los ficheros del .zip descargado para proceder a instalarlo.

```
unzip AutoFirma_Linux.zip
rm Autofirma_Linux.zip

sudo dpkg -i AutoFirma_1_7_1.deb
```

Ya podemos usar AutoFirma.

```
AutoFirma
```

![certificado](/img/cripto3/23.png)


##### VALIDe

Vamos a validar mi certificado desde la página [VALIDe](https://valide.redsara.es/valide/validarCertificado/ejecutar.html;jsessionid=EDCDF2E1A8EEA1A761F4DAC78667757C).

Hacemos click en **Seleccionar Certificado**.

![certificado](/img/cripto3/16.png)

Nos pedirá permiso para abrir con AutoFirma.

![certificado](/img/cripto3/17.png)

Seleccionamos el certificado.

![certificado](/img/cripto3/18.png)

Introducimos el código de seguridad y validamos.

![certificado](/img/cripto3/19.png)


### Tarea 3: Firma electrónica

#### 1. Utilizando la página VALIDe y el programa autofirma, firma un documento con tu certificado y envíalo por correo a un compañero.

He creado dos archivos que firmaré, uno con Autofirma y otro con VALIDe.

![certificado](/img/cripto3/24.png)

![certificado](/img/cripto3/25.png)


En AutoFirma seleccionamos el fichero y lo firmamos.

![certificado](/img/cripto3/11.png)

Nos aparece el siguiente mensaje informativo:

![certificado](/img/cripto3/12.png)

Aceptamos y nos dirá que seleccionemos el certificado.

![certificado](/img/cripto3/13.png)

Guardamos el fichero firmado.

![certificado](/img/cripto3/14.png)

![certificado](/img/cripto3/15.png)

Ahora vamos a [VALIDe](https://valide.redsara.es/valide/firmar/ejecutar.html) y firmamos el otro fichero.

![certificado](/img/cripto3/26.png)

Seleccionamos el fichero a firmar.

![certificado](/img/cripto3/27.png)

Y seleccionamos el certificado.

![certificado](/img/cripto3/28.png)

El fichero ya se ha firmado. Ahora tendremos que seleccionar **Guardar firma** para que se guarde el fichero.

![certificado](/img/cripto3/29.png)

Envío estos ficheros firmados a un compañero.


#### 2. Tu debes recibir otro documento firmado por un compañero y utilizando las herramientas anteriores debes visualizar la firma (Visualizar Firma) y (Verificar Firma). ¿Puedes verificar la firma aunque no tengas la clave pública de tu compañero?, ¿Es necesario estar conectado a internet para hacer la validación de la firma?. Razona tus respuestas.

He recibido dos ficheros de mi compañero.

![certificado](/img/cripto3/30.png)

Visualizo el primero con AutoFirma.

![certificado](/img/cripto3/31.png)

Comprobamos que el fichero está firmado correctamente.

![certificado](/img/cripto3/32.png)

Ahora visualizamos la firma con [VALIDe](https://valide.redsara.es/valide/validarFirma/ejecutar.html).

![certificado](/img/cripto3/36.png)

![certificado](/img/cripto3/35.png)

Y a continuación la validamos.

![certificado](/img/cripto3/33.png)

![certificado](/img/cripto3/34.png)

Podemos comprobar que se pueden verificar las firmas de los ficheros aunque no tengamos su clave pública, ya que esta se adjunta al fichero firmado.

En cuanto a si es necesario estar conectados a Internet para validar las firmas, con AutoFirma no es necesario ya que es una aplicación de escritorio. Para VALIDe si necesitaremos conección a Internet para acceder a su página web.


#### 3. Entre dos compañeros, firmar los dos un documento, verificar la firma para comprobar que está firmado por los dos.

Voy a firmar uno de los documentos que me envió mi compañero en el ejercicio anterior. Usaré AutoFirma para firmarlo.

![certificado](/img/cripto3/37.png)

![certificado](/img/cripto3/38.png)

![certificado](/img/cripto3/39.png)

![certificado](/img/cripto3/40.png)


### Tarea 4: Autentificación

#### 1. Utilizando tu certificado accede a alguna página de la administración pública )cita médica, becas, puntos del carnet,…). Entrega capturas de pantalla donde se demuestre el acceso a ellas.

Voy a acceder a ClicSalud+ con mi certificado digital, para ello me voy a su [web](https://www.sspa.juntadeandalucia.es/servicioandaluzdesalud/clicsalud/) y hago click en la opción de Acceder con Certificado digital.

![certificado](/img/cripto3/20.png)

Selecciono mi certificado y le doy a Aceptar.

![certificado](/img/cripto3/21.png)

Compruebo que ya he accedido con mi usuario y que puedo ver mis datos.

![certificado](/img/cripto3/22.png)


---------------------------

## HTTPS / SSL

> Antes de hacer esta práctica vamos a crear una página web (puedes usar una página estática o instalar una aplicación web) en un servidor web apache2 que se acceda con el nombre `tunombre.iesgn.org`.

```
sudo apt install apache2
sudo mkdir /var/www/arantxa.iesgn.org
sudo nano /var/www/arantxa.iesgn.org/index.html

<!DOCTYPE html lang="es">
      <head>
          <meta charset="utf-8">
          <title>arantxa.iesgn.org</title>
      </head>
      <body>
          <h1>Arantxa Fernández Morató</h1>
          <h2>Criptografia III. Certificados digitales y https.</h2>
          <p>Esto es una web de prueba para la práctica de criptografía de SAD.</p>
      </body>
</html>

cd /etc/apache2/sites-available
sudo cp 000-default.conf arantxa.iesgn.org.conf
sudo nano arantxa.iesgn.org.conf

<VirtualHost *:80>
        ServerName arantxa.iesgn.org

        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/arantxa.iesgn.org

        ErrorLog ${APACHE_LOG_DIR}/error-arantxaiesgn.log
        CustomLog ${APACHE_LOG_DIR}/access-arantxaiesgn.log combined
</VirtualHost>

sudo a2ensite arantxa.iesgn.org.conf
sudo systemctl restart apache2
sudo nano /etc/hosts

192.168.122.192     arantxa.iesgn.org
```


### Tarea 1: Certificado autofirmado

Esta práctica la vamos a realizar con un compañero. En un primer momento un alumno creará una Autoridad Certficadora y firmará un certificado para la página del otro alumno. Posteriormente se volverá a realizar la práctica con los roles cambiados.

Para hacer esta práctica puedes buscar información en internet, algunos enlaces interesantes:

- [Phil’s X509/SSL Guide](https://www.phildev.net/ssl/)
- [How to setup your own CA with OpenSSL](https://gist.github.com/Soarez/9688998)
- [Crear autoridad certificadora (CA) y certificados autofirmados en Linux](https://blog.guillen.io/2018/09/29/crear-autoridad-certificadora-ca-y-certificados-autofirmados-en-linux/)


### Autoridad certificadora

El alumno que hace de Autoridad Certificadora deberá entregar una documentación donde explique los siguientes puntos:

#### 1. Crear su autoridad certificadora (generar el certificado digital de la CA). Mostrar el fichero de configuración de la AC.

En la máquina virtual donde crearé la web estática voy a crear la autoridad certificadora.

Accedo con root y creo los siguientes directorios:

```
sudo su
mkdir -p CA/{certsdb,certreqs,crl,private,certs}
cd CA
```

Cambio los permisos al fichero private por motivos de seguridad.

```
chmod 700 private
```

Creamos un fichero donde se guardarán los certificados existentes.

```
touch index.txt
```

Copiamos el fichero openssl para adaptarlo a nuestras necesidades.

```
cp /usr/lib/ssl/openssl.cnf .
nano openssl.cnf
```

Lo modificamos y lo dejamos como se muestra a continuación:

```
#
# OpenSSL example configuration file.
# This is mostly being used for generation of certificate requests.
#

# Note that you can include other files from the main configuration
# file using the .include directive.
#.include filename

# This definition stops the following lines choking if HOME isn't
# defined.
HOME			= .

# Extra OBJECT IDENTIFIER info:
#oid_file		= $ENV::HOME/.oid
oid_section		= new_oids

# System default
openssl_conf = default_conf

# To use this configuration file with the "-extfile" option of the
# "openssl x509" utility, name here the section containing the
# X.509v3 extensions to use:
# extensions		=
# (Alternatively, use a configuration file that has only
# X.509v3 extensions in its main [= default] section.)

[ new_oids ]

# We can add new OIDs in here for use by 'ca', 'req' and 'ts'.
# Add a simple OID like this:
# testoid1=1.2.3.4
# Or use config file substitution like this:
# testoid2=${testoid1}.5.6

# Policies used by the TSA examples.
tsa_policy1 = 1.2.3.4.1
tsa_policy2 = 1.2.3.4.5.6
tsa_policy3 = 1.2.3.4.5.7

####################################################################
[ ca ]
default_ca	= CA_default		# The default ca section

####################################################################
[ CA_default ]

dir		= /root/CA		# Where everything is kept
certs		= $dir/certsdb		# Where the issued certs are kept
crl_dir		= $dir/crl		# Where the issued crl are kept
database	= $dir/index.txt	# database index file.
#unique_subject	= no			# Set to 'no' to allow creation of
					# several certs with same subject.
new_certs_dir	= $dir/certs		# default place for new certs.

certificate	= $dir/cacert.pem 	# The CA certificate
serial		= $dir/serial 		# The current serial number
crlnumber	= $dir/crlnumber	# the current crl number
					# must be commented out to leave a V1 CRL
crl		= $dir/crl.pem 		# The current CRL
private_key	= $dir/private/cakey.pem# The private key

x509_extensions	= usr_cert		# The extensions to add to the cert

# Comment out the following two lines for the "traditional"
# (and highly broken) format.
name_opt 	= ca_default		# Subject Name options
cert_opt 	= ca_default		# Certificate field options

# Extension copying option: use with caution.
# copy_extensions = copy

# Extensions to add to a CRL. Note: Netscape communicator chokes on V2 CRLs
# so this is commented out by default to leave a V1 CRL.
# crlnumber must also be commented out to leave a V1 CRL.
# crl_extensions	= crl_ext

default_days	= 365			# how long to certify for
default_crl_days= 30			# how long before next CRL
default_md	= default		# use public key default MD
preserve	= no			# keep passed DN ordering

# A few difference way of specifying how similar the request should look
# For type CA, the listed attributes must be the same, and the optional
# and supplied fields are just that :-)
policy		= policy_match

# For the CA policy
[ policy_match ]
countryName		= match
stateOrProvinceName	= match
organizationName	= match
organizationalUnitName	= optional
commonName		= supplied
emailAddress		= optional

# For the 'anything' policy
# At this point in time, you must list all acceptable 'object'
# types.
[ policy_anything ]
countryName		= optional
stateOrProvinceName	= optional
localityName		= optional
organizationName	= optional
organizationalUnitName	= optional
commonName		= supplied
emailAddress		= optional

####################################################################
[ req ]
default_bits		= 2048
default_keyfile 	= privkey.pem
distinguished_name	= req_distinguished_name
attributes		= req_attributes
x509_extensions	= v3_ca	# The extensions to add to the self signed cert

# Passwords for private keys if not present they will be prompted for
# input_password = secret
# output_password = secret

# This sets a mask for permitted string types. There are several options.
# default: PrintableString, T61String, BMPString.
# pkix	 : PrintableString, BMPString (PKIX recommendation before 2004)
# utf8only: only UTF8Strings (PKIX recommendation after 2004).
# nombstr : PrintableString, T61String (no BMPStrings or UTF8Strings).
# MASK:XXXX a literal mask value.
# WARNING: ancient versions of Netscape crash on BMPStrings or UTF8Strings.
string_mask = utf8only

# req_extensions = v3_req # The extensions to add to a certificate request

[ req_distinguished_name ]
countryName			= Country Name (2 letter code)
countryName_default		= ES
countryName_min			= 2
countryName_max			= 2

stateOrProvinceName		= State or Province Name (full name)
stateOrProvinceName_default	= Sevilla

localityName			= Locality Name (eg, city)
localityName_default		= Dos Hermanas
0.organizationName		= Organization Name (eg, company)
0.organizationName_default	= Afermor8

# we can do this but it is not needed normally :-)
#1.organizationName		= Second Organization Name (eg, company)
#1.organizationName_default	= World Wide Web Pty Ltd

organizationalUnitName		= Organizational Unit Name (eg, section)
organizationalUnitName_default	= GN

commonName			= Common Name (e.g. server FQDN or YOUR name)
commonName_max			= 64

emailAddress			= Email Address
emailAddress_max		= 64

# SET-ex3			= SET extension number 3

[ req_attributes ]
#challengePassword		= A challenge password
#challengePassword_min		= 4
#challengePassword_max		= 20

#unstructuredName		= An optional company name

[ usr_cert ]

# These extensions are added when 'ca' signs a request.

# This goes against PKIX guidelines but some CAs do it and some software
# requires this to avoid interpreting an end user certificate as a CA.

basicConstraints=CA:FALSE

# Here are some examples of the usage of nsCertType. If it is omitted
# the certificate can be used for anything *except* object signing.

# This is OK for an SSL server.
# nsCertType			= server

# For an object signing certificate this would be used.
# nsCertType = objsign

# For normal client use this is typical
# nsCertType = client, email

# and for everything including object signing:
# nsCertType = client, email, objsign

# This is typical in keyUsage for a client certificate.
# keyUsage = nonRepudiation, digitalSignature, keyEncipherment

# This will be displayed in Netscape's comment listbox.
nsComment			= "OpenSSL Generated Certificate"

# PKIX recommendations harmless if included in all certificates.
subjectKeyIdentifier=hash
authorityKeyIdentifier=keyid,issuer

# This stuff is for subjectAltName and issuerAltname.
# Import the email address.
# subjectAltName=email:copy
# An alternative to produce certificates that aren't
# deprecated according to PKIX.
# subjectAltName=email:move

# Copy subject details
# issuerAltName=issuer:copy

#nsCaRevocationUrl		= http://www.domain.dom/ca-crl.pem
#nsBaseUrl
#nsRevocationUrl
#nsRenewalUrl
#nsCaPolicyUrl
#nsSslServerName

# This is required for TSA certificates.
# extendedKeyUsage = critical,timeStamping

[ v3_req ]

# Extensions to add to a certificate request

basicConstraints = CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment

[ v3_ca ]


# Extensions for a typical CA


# PKIX recommendation.

subjectKeyIdentifier=hash

authorityKeyIdentifier=keyid:always,issuer

basicConstraints = critical,CA:true

# Key usage: this is typical for a CA certificate. However since it will
# prevent it being used as an test self-signed certificate it is best
# left out by default.
# keyUsage = cRLSign, keyCertSign

# Some might want this also
# nsCertType = sslCA, emailCA

# Include email address in subject alt name: another PKIX recommendation
# subjectAltName=email:copy
# Copy issuer details
# issuerAltName=issuer:copy

# DER hex encoding of an extension: beware experts only!
# obj=DER:02:03
# Where 'obj' is a standard or added object
# You can even override a supported extension:
# basicConstraints= critical, DER:30:03:01:01:FF

[ crl_ext ]

# CRL extensions.
# Only issuerAltName and authorityKeyIdentifier make any sense in a CRL.

# issuerAltName=issuer:copy
authorityKeyIdentifier=keyid:always

[ proxy_cert_ext ]
# These extensions should be added when creating a proxy certificate

# This goes against PKIX guidelines but some CAs do it and some software
# requires this to avoid interpreting an end user certificate as a CA.

basicConstraints=CA:FALSE

# Here are some examples of the usage of nsCertType. If it is omitted
# the certificate can be used for anything *except* object signing.

# This is OK for an SSL server.
# nsCertType			= server

# For an object signing certificate this would be used.
# nsCertType = objsign

# For normal client use this is typical
# nsCertType = client, email

# and for everything including object signing:
# nsCertType = client, email, objsign

# This is typical in keyUsage for a client certificate.
# keyUsage = nonRepudiation, digitalSignature, keyEncipherment

# This will be displayed in Netscape's comment listbox.
nsComment			= "OpenSSL Generated Certificate"

# PKIX recommendations harmless if included in all certificates.
subjectKeyIdentifier=hash
authorityKeyIdentifier=keyid,issuer

# This stuff is for subjectAltName and issuerAltname.
# Import the email address.
# subjectAltName=email:copy
# An alternative to produce certificates that aren't
# deprecated according to PKIX.
# subjectAltName=email:move

# Copy subject details
# issuerAltName=issuer:copy

#nsCaRevocationUrl		= http://www.domain.dom/ca-crl.pem
#nsBaseUrl
#nsRevocationUrl
#nsRenewalUrl
#nsCaPolicyUrl
#nsSslServerName

# This really needs to be in place for it to be a proxy certificate.
proxyCertInfo=critical,language:id-ppl-anyLanguage,pathlen:3,policy:foo

####################################################################
[ tsa ]

default_tsa = tsa_config1	# the default TSA section

[ tsa_config1 ]

# These are used by the TSA reply generation only.
dir		= ./demoCA		# TSA root directory
serial		= $dir/tsaserial	# The current serial number (mandatory)
crypto_device	= builtin		# OpenSSL engine to use for signing
signer_cert	= $dir/tsacert.pem 	# The TSA signing certificate
					# (optional)
certs		= $dir/cacert.pem	# Certificate chain to include in reply
					# (optional)
signer_key	= $dir/private/tsakey.pem # The TSA private key (optional)
signer_digest  = sha256			# Signing digest to use. (Optional)
default_policy	= tsa_policy1		# Policy if request did not specify it
					# (optional)
other_policies	= tsa_policy2, tsa_policy3	# acceptable policies (optional)
digests     = sha1, sha256, sha384, sha512  # Acceptable message digests (mandatory)
accuracy	= secs:1, millisecs:500, microsecs:100	# (optional)
clock_precision_digits  = 0	# number of digits after dot. (optional)
ordering		= yes	# Is ordering defined for timestamps?
				# (optional, default: no)
tsa_name		= yes	# Must the TSA name be included in the reply?
				# (optional, default: no)
ess_cert_id_chain	= no	# Must the ESS cert id chain be included?
				# (optional, default: no)
ess_cert_id_alg		= sha1	# algorithm to compute certificate
				# identifier (optional, default: sha1)
[default_conf]
ssl_conf = ssl_sect

[ssl_sect]
system_default = system_default_sect

[system_default_sect]
MinProtocol = TLSv1.2
CipherString = DEFAULT@SECLEVEL=2
```

Creamos a continuación la clave y el fichero de solicitud de firma.

```
openssl req -new -newkey rsa:2048 -keyout private/cakey.pem -out careq.pem -config ./openssl.cnf
```

![clave](/img/cripto3/46.png)

Nos autofirmamos el certificado generado.

```
openssl ca -create_serial -out cacert.pem -days 365 -keyfile private/cakey.pem -selfsign -extensions v3_ca -config ./openssl.cnf -infiles careq.pem
```

![clave](/img/cripto3/47.png)


#### 2. Debe recibir el fichero CSR (Solicitud de Firmar un Certificado) de su compañero, debe firmarlo y enviar el certificado generado a su compañero.

Mi compañero me envía su fichero CSR mediante scp, el cual he movido al directorio **/root/CA/certreqs/**, y se lo firmo del siguiente modo:

```
openssl ca -config openssl.cnf -out certsdb/arturo.crt -infiles certreqs/arturo.csr 
```

Una vez firmado se lo vuelvo a enviar.


#### 3. ¿Qué otra información debes aportar a tu compañero para que éste configure de forma adecuada su servidor web con el certificado generado?

Habría que enviar a nuestro compañero además el certificado de la autoridad certificadora (cacert.pem) para que pueda verificar la firma sobre su propio certificado.


### Administrador servidor web

El alumno que hace de administrador del servidor web, debe entregar una documentación que describa los siguientes puntos:

#### 1. Crea una clave privada RSA de 4096 bits para identificar el servidor.

```
sudo openssl genrsa -aes256 -out /etc/ssl/private/clavearantxa.key 4096
sudo chmod 400 /etc/ssl/private/clavearantxa.key
```

#### 2. Utiliza la clave anterior para generar un CSR, considerando que deseas acceder al servidor con el FQDN (tunombre.iesgn.org).

```
openssl req -new -sha256 -key /etc/ssl/private/clavearantxa.key -out arantxa.csr
```

![certificado](/img/cripto3/41.png)


#### 3. Envía la solicitud de firma a la entidad certificadora (su compañero).

Le envío mi csr (arantxa.csr) a mi compañero.

![certificado](/img/cripto3/42.png)


#### 4. Recibe como respuesta un certificado X.509 para el servidor firmado y el certificado de la autoridad certificadora.

Recibo mi csr firmado por la autoridad certificadora de mi compañero.

```
ls -l /home/usuario1/ | egrep '(cacert|arantxa)'
```

![csr](/img/cripto3/43.png)


#### 5. Configura tu servidor web con https en el puerto 443, haciendo que las peticiones http se redireccionen a https (forzar https).

Para empezar copio los ficheros a /etc/ssl/certs/, cambiamos el usuario al que pertenece y cambiamos los permisos.

```
sudo cp /home/usuario1/arantxa.crt /etc/ssl/certs/
sudo cp /home/usuario1/cacert.pem /etc/ssl/certs/
sudo chown root:root /etc/ssl/certs/arantxa.crt
sudo chown root:root /etc/ssl/certs/cacert.pem
sudo chmod 644 /etc/ssl/certs/arantxa.crt
sudo chmod 644 /etc/ssl/certs/cacert.pem
```

Modificamos el virtualhost para https.

```
nano /etc/apache2/sites-available/default-ssl.conf

<IfModule mod_ssl.c>
        <VirtualHost *:443>
                ServerAdmin webmaster@localhost
                ServerName arantxa.iesgn.org
                DocumentRoot /var/www/arantxa.iesgn.org

                ErrorLog ${APACHE_LOG_DIR}/error-arantxaiesgn.log
                CustomLog ${APACHE_LOG_DIR}/access-arantxaiesgn.log combined

                SSLEngine on

                SSLCertificateFile      /etc/ssl/certs/arantxa.crt
                SSLCertificateKeyFile /etc/ssl/private/clavearantxa.key
                SSLCACertificateFile /etc/ssl/certs/cacert.pem

                <FilesMatch "\.(cgi|shtml|phtml|php)$">
                                SSLOptions +StdEnvVars
                </FilesMatch>
                <Directory /usr/lib/cgi-bin>
                                SSLOptions +StdEnvVars
                </Directory>


        </VirtualHost>
</IfModule>
```

Habilito el virualhost para https y el módulo ssl.

```
sudo a2ensite default-ssl
sudo a2enmod ssl
```

Añadimos la redirección al virtualhost y reiniciamos el servicio:

```
sudo nano /etc/apache2/sites-available/arantxa.iesgn.org.conf

<VirtualHost *:80>
        ServerName arantxa.iesgn.org

        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/arantxa.iesgn.org

        ErrorLog ${APACHE_LOG_DIR}/error-arantxaiesgn.log
        CustomLog ${APACHE_LOG_DIR}/access-arantxaiesgn.log combined
        Redirect 301 / https://arantxa.iesgn.org/
</VirtualHost>

sudo systemctl restart apache2
```

Al intentar acceder nos aparece lo siguiente:

![web](/img/cripto3/44.png)

Tendremos que importar el certificado cacert.pem que nos mandó nuestro compañero para que nos permita acceder de forma segura.

![web](/img/cripto3/45.png)

![web](/img/cripto3/48.png)

![web](/img/cripto3/49.png)

Ya debería funcionar correctamente mi página web desde https.

![web](/img/cripto3/50.png)

#### 6. Instala ahora un servidor nginx, y realiza la misma configuración que anteriormente para que se sirva la página con HTTPS.

```
systemctl disable --now apache2
apt install nginx
nano /etc/nginx/sites-available/arantxa.iesgn.org.nginx.conf

server {
        listen 80;
        listen [::]:80;

        server_name arantxa.iesgn.org;

        return 301 https://$server_name$request_uri;
}

server {
        listen 443 ssl http2;
        listen [::]:443 ssl http2;
        server_name arantxa.iesgn.org;

        ssl_certificate    /etc/ssl/certs/arantxa.crt;
        ssl_certificate_key    /etc/ssl/private/clavearantxa.key;
        ssl_trusted_certificate /etc/ssl/certs/cacert.crt;

        root /var/www/arantxa.iesgn.org;
        index index.html index.htm index.nginx-debian.html;

        location / {
                try_files $uri $uri/ =404;
        }
}
```


Modifico el html:

```

<!DOCTYPE html lang="es">
      <head>
          <meta charset="utf-8">
          <title>arantxa.iesgn.org</title>
      </head>
      <body>
          <h1>Arantxa Fernández Morató</h1>
          <h2>Criptografía III. Certificados digitales y https.</h2>
          <p>Esto es una web de prueba para la práctica de criptografía de SAD.</p>
          <p>NGINX funcionando!!!</p>
      </body>
</html>
```

Crear el enlace simbólico y reiniciar servicio nginx:

```
sudo ln -s /etc/nginx/sites-available/arantxa.iesgn.org.nginx.conf /etc/nginx/sites-enabled/
sudo systemctl restart nginx
```

Compruebo que la web funciona correctamente.

![web-nginx](/img/cripto3/46.png)

![web-nginx2](/img/cripto3/47.png)
