---
title: "Criptografía II. Integridad, firmas y autenticación"
date: 2022-12-19T18:29:33+01:00
draft: true
tags: ["SAD"]
bigimg: [{src: "/img/triangle.jpg", desc: "Triangle"}, {src: "/img/sphere.jpg", desc: "Sphere"}, {src: "/img/hexagon.jpg", desc: "Hexagon"}]
---

--------------------

## Tarea 1: Firmas electrónicas.

--------------------

En este primer apartado vamos a trabajar con las firmas electrónicas, para ello te pueden ayudar los siguientes enlaces:

- [Intercambiar claves](https://www.gnupg.org/gph/es/manual/x75.html)
- [Validar otras claves en nuestro anillo de claves públicas](https://www.gnupg.org/gph/es/manual/x354.html)
- [Firmado de claves (Debian)](https://www.debian.org/events/keysigning.es.html)

**GPG**

### 1. Manda un documento y la firma electrónica del mismo a un compañero. Verifica la firma que tú has recibido

> Antes de empezar he generado un par de claves con `gpg --full-gen-key`.

Creo un fichero llamado prueba_gpg.txt y lo firmo con mi clave.

```
nano prueba_gpg.txt
gpg --detach-sign prueba_gpg.txt
```

Le he enviado mi documento y la firma del mismo a mi compañero Antonio. Él me ha enviado su documnto y firma del documento. Importo su clave y verifico el documento:

```
gpg --import antonio_marchan.asc
gpg --verify antonio.sig antonio_documento.txt
```

![verify](/img/cripto2/1.png)

### 2. ¿Qué significa el mensaje que aparece en el momento de verificar la firma?

Significa que la firma es correcta, pero al no haber firmado la clave con nuestra clave privada o con otra de confianza nos aparece este mensaje de advertencia.

### 3. Vamos a crear un anillo de confianza entre los miembros de nuestra clase, para ello:

Subo mi clave al servidor para que la puedan descargar mis compañeros:

```
gpg --keyserver keyserver.ubuntu.com --send-keys 43140D1577041C18C5830A9B195360276022F67D
```

Consigo las firmas de mis compañeros:

**Juan Jesus**:

```
gpg --keyserver keyserver.ubuntu.com --recv-keys 212357B9A18C9EFA3531CDF994615205285D18A5
```

**María**:

```
gpg --keyserver keyserver.ubuntu.com --recv-keys 2614614FABC5378E657019153C7A682537DE4164
```

**Antonio**:

```
gpg --keyserver keyserver.ubuntu.com --recv-keys DFBAF122C0DC0303EC76CE384C6D5995F2B13373
```

Firmo sus claves:

```
gpg --sign-key 212357B9A18C9EFA3531CDF994615205285D18A5
gpg --sign-key 2614614FABC5378E657019153C7A682537DE4164
gpg --sign-key DFBAF122C0DC0303EC76CE384C6D5995F2B13373
```

Ahora debería volver a subir todas las claves firmadas al servidor pero en nuestro caso no funcionaba correctamente por lo que hemos exportado las claves de cada compañero para después enviarsela. Si se quisiera subir al servidor se haría de la siguiente manera:

```
gpg --keyserver keyserver.ubuntu.com --send-keys 212357B9A18C9EFA3531CDF994615205285D18A5
gpg --keyserver keyserver.ubuntu.com --send-keys 2614614FABC5378E657019153C7A682537DE4164
gpg --keyserver keyserver.ubuntu.com --send-keys DFBAF122C0DC0303EC76CE384C6D5995F2B13373
```

Exporto las claves en una carpeta que he creado llamada firmas:

```
mkdir firmas
cd firmas
gpg --export -a 212357B9A18C9EFA3531CDF994615205285D18A5 > clave-juanje-ara.asc
gpg --export -a 2614614FABC5378E657019153C7A682537DE4164 > clave-maria-ara.asc
gpg --export -a DFBAF122C0DC0303EC76CE384C6D5995F2B13373 > clave-antonio-ara.asc
```

Le envío las claves de nuevo a cada compañero.
Me envían ellos a mi la mía firmada por cada uno y las importo:

```
gpg --import claveArantxa.asc 
gpg --import arantxa_nueva.asc 
gpg --import claveArantxa-maria.asc 
```

### 4. Muestra las firmas que tiene tu clave pública.

Listamos las claves para comprobar que la mía está firmada por cada uno de mis compañeros:

```
gpg --list-sig
```

![firmas](/img/cripto2/17.png)


### 5. Comprueba que ya puedes verificar sin “problemas” una firma recibida por una persona en la que confías.

Como he firmado la clave de Antonio ya no me aparecerá el mensaje de advertencia que me apareció anteriormente.

```
gpg --verify antonio.sig antonio_documento.txt
```

![verify-correcto](/img/cripto2/5.png)

### 6. Comprueba que puedes verificar con confianza una firma de una persona en las que no confías, pero sin embargo sí confía otra persona en la que tu tienes confianza total.

Para este ejercicio la persona con confianza absoluta será Juan Jesús y la persona en la que no confío será Felipe.

Voy a firmar con confianza absoluta la clave de Juan Jesús.

```
gpg --edit-key 212357B9A18C9EFA3531CDF994615205285D18A5

trust

5

quit
```

![confianza-absoluta](/img/cripto2/18.png)

A continuación Juan Jesús me envía la clave de Felipe firmada por él con confianza absoluta. La importo:

```
gpg --import felipefirmada-juanje.asc
```

Por último, Felipe me envía un fichero firmado por él y debería poder verificarlo sin que me apareciera ningún mensaje de advertencia.

```
gpg --verify sad.txt.sig sad.txt
```

![verify-felipe](/img/cripto2/4.png)

--------------------

## Tarea 2: Correo seguro con evolution/thunderbird

--------------------

Ahora vamos a configurar nuestro cliente de correo electrónico para poder mandar correos cifrados, para ello:

### 1. Configura el cliente de correo evolution con tu cuenta de correo habitual

Para empezar instalo thunderbird:

```
sudo apt install thunderbird thunderbird-l10n-es-es
```

Iniciamos thunderbird y nos aparecerá la pestaña de la captura. Añadimos nuestra cuenta de correo.

```
thunderbird
```

![thunderbird](/img/cripto2/6.png)

Comprobamos que se ha creado correctamente:

![thunderbird-conexion](/img/cripto2/7.png)

![thunderbird-conexion](/img/cripto2/19.png)

### 2. Añade a la cuenta las opciones de seguridad para poder enviar correos firmados con tu clave privada o cifrar los mensajes para otros destinatarios

En el desplegable de arriba a la derecha hacemos click sobre "Configuración de la cuenta".

![thunderbird-config](/img/cripto2/20.png)

En la pestaña que se abre hacer seleccionamos "Cifrado extremo a extremo". Importo mi clave personal OpenPGP.

![thunderbird-clave](/img/cripto2/8.png)

![thunderbird-clave2](/img/cripto2/9.png)

![thunderbird-clave3](/img/cripto2/10.png)

Vamos a la opción "Administrador de claves OpenPGP".

![thunderbird-clave4](/img/cripto2/21.png)

Hacemos click en "Importar clave pública desde archivo" y añado la clave de Juan Jesús.

![thunderbird-clave5](/img/cripto2/22.png)

Importamos y ya nos aparecerá la clave.

![thunderbird-clave6](/img/cripto2/23.png)

![thunderbird-clave7](/img/cripto2/11.png)

![thunderbird-clave8](/img/cripto2/12.png)

### 3. Envía y recibe varios mensajes con tus compañeros y comprueba el funcionamiento adecuado de GPG

Le envío un correo cifrado a Juan Jesús. Al escribir su correo electrónico (el mismo que el de la clave pública que añadimos anteriormente) nos aparece la opción "Cifrar". Escribimos el correo y lo mandamos cifrado:

![thunderbird-correo-juanje](/img/cripto2/24.png)

Vemos que al enviarse el correo cifrado nos aparece un archivo adjunto con la firma del correo.

![thunderbird-correo-juanje2](/img/cripto2/25.png)

Juan Jesús me envía a mi un correo y compruebo que se puede leer a pesar de estar cifrado.

![thunderbird-correo](/img/cripto2/13.png)

Si intento ver el correo desde Gmail no aparecerá el contenido, ya que está cifrado y no tengo la clave añadida a Gmail.

![thunderbird-correo2](/img/cripto2/14.png)

### 4. Enviar al profesor por correo electrónico un mensaje firmado por vosotros y que solo pueda descifrar él

Descargo la clave pública de Raúl y la exporto:

```
gpg --keyserver keyserver.ubuntu.com --recv-keys 8DEB2BE5

gpg --export -a 8DEB2BE5 > claveRAUL.asc
```

Añado la clave de Raúl a Thunderbird.

![clave-raul](/img/cripto2/15.png)

Le envío un correo.

![correo-raul](/img/cripto2/16.png)

--------------------

## Tarea 3: Integridad de ficheros

--------------------

Vamos a descargarnos la ISO de debian, y posteriormente vamos a comprobar su integridad.

Puedes encontrar la ISO en la dirección: https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/.

### 1. Para validar el contenido de la imagen CD, solo asegúrese de usar la herramienta apropiada para sumas de verificación. Para cada versión publicada existen archivos de suma de comprobación con algoritmos fuertes (SHA256 y SHA512); debería usar las herramientas sha256sum o sha512sum para trabajar con ellos

Descargamos con wget la ISO de Debian 11, los ficheros SHA256SUMS y SHA512SUMS, y sus firmas SHA256SUMS.sign y SHA512SUMS.sign.

```
wget https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/debian-11.6.0-amd64-netinst.iso
wget https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/SHA256SUMS
wget https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/SHA256SUMS.sign
wget https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/SHA512SUMS
wget https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/SHA512SUMS.sign
```

Los ficheros SHA256SUMS y SHA512SUMS contienen el hash de los tres ficheros de descarga de Debian disponibles en la web. Como yo he descargado debian-11.6.0-amd64-netinst.iso, los hashes que pertenecen a mi fichero son los primeros.

![contenido-ficheros](/img/cripto2/26.png)

Si usamos las utilidades **`sha256sum`** y **`sha512sum`** en el fichero ISO de debian nos aparecerá el hash de este. Solo tendremos que comparar si es igual que el de los ficheros que hemos visto en la captura anterior para saber si el archivo ISO ha sido modificado o no.

![comparación-hashes](/img/cripto2/28.png)

O lo podemos hacer con la opción -c.

![comparación-hashes](/img/cripto2/29.png)

### 2. Verifica que el contenido del hash que has utilizado no ha sido manipulado, usando la firma digital que encontrarás en el repositorio. Puedes encontrar una guía para realizarlo en este artículo: [How to verify an authenticity of downloaded Debian ISO images](https://linuxconfig.org/how-to-verify-an-authenticity-of-downloaded-debian-iso-images). Tenemos que asegurarnos además de que los ficheros donde se encuentran las claves no han sido modificados.

Descargamos del servidor **keyring.debian.org** la clave pública de Debian.

```
gpg --keyserver keyring.debian.org --recv-keys 6294BE9B
```

![clave-debian](/img/cripto2/27.png)

Y ya podremos comprobar con **`--verify`** que los ficheros descargados no han sido modificados y las claves son correctas.

```
gpg --verify SHA256SUMS.sign SHA256SUMS
gpg --verify SHA512SUMS.sign SHA512SUMS
```
![clave-debian](/img/cripto2/30.png)

--------------------

## Tarea 4: Integridad y autenticidad (apt secure)

--------------------

Cuando nos instalamos un paquete en nuestra distribución linux tenemos que asegurarnos que ese paquete es legítimo. Para conseguir este objetivo se utiliza criptografía asimétrica, y en el caso de Debian a este sistema se llama apt secure. Esto lo debemos tener en cuenta al utilizar los repositorios oficiales. Cuando añadamos nuevos repositorios tendremos que añadir las firmas necesarias para confiar en que los paquetes son legítimos y no han sido modificados.

Busca información sobre apt secure y responde las siguientes preguntas:

### 1. ¿Qué software utiliza apt secure para realizar la criptografía asimétrica?

APT utiliza el sistema de criptografía GPG (GNU Privacy Guard) para validar los paquetes .deb descargados y, de esta forma, asegurarse de que no han sido alterados. A esto se le llama Secure APT.

### 2. ¿Para que sirve el comando apt-key? ¿Qué muestra el comando apt-key list?

El comando **`apt-key`** sirve para gestionar la lista de claves de confianza para autenticar los paquetes .deb. Algunas opciones disponibles para este comando son:

- `apt-key add <file>`: Añade la clave contenida en "file"
- `apt-key del <keyid>`: Elimina la clave "keyid"
- `apt-key export <keyid>`: Exporta la clave "keyid"
- `apt-key exportall`: Exporta todas las claves de confianza
- `apt-key update`: Actualiza las claves usando el paquete keyring
- `apt-key net-update`: Actualiza las claves usando la misma red
- `apt-key list`: Lista las claves
- `apt-key finger`: Lista los fingerprint

Con el comando **`apt-key list`**, como hemos dicho se muestran las claves de confianza para APT. En mi caso son las siguientes:

```
usuario@debian-oracle:~/firmas$ apt-key list
Warning: apt-key is deprecated. Manage keyring files in trusted.gpg.d instead (see apt-key(8)).
/etc/apt/trusted.gpg
--------------------
pub   rsa4096 2021-12-14 [SC] [caduca: 2023-12-14]
      859B E8D7 C586 F538 430B  19C2 467B 942D 3A79 BD29
uid        [desconocida] MySQL Release Engineering <mysql-build@oss.oracle.com>
sub   rsa4096 2021-12-14 [E] [caduca: 2023-12-14]

pub   rsa4096 2019-05-28 [SC] [caduca: 2024-05-26]
      2069 1EEC 3521 6C63 CAF6  6CE1 6564 08E3 90CF B1F5
uid        [desconocida] MongoDB 4.4 Release Signing Key <packaging@mongodb.com>

pub   rsa4096 2021-02-16 [SC] [caduca: 2026-02-15]
      F567 9A22 2C64 7C87 527C  2F8C B00A 0BD1 E2C6 3C11
uid        [desconocida] MongoDB 5.0 Release Signing Key <packaging@mongodb.com>

/etc/apt/trusted.gpg.d/debian-archive-bullseye-automatic.gpg
------------------------------------------------------------
pub   rsa4096 2021-01-17 [SC] [caduca: 2029-01-15]
      1F89 983E 0081 FDE0 18F3  CC96 73A4 F27B 8DD4 7936
uid        [desconocida] Debian Archive Automatic Signing Key (11/bullseye) <ftpmaster@debian.org>
sub   rsa4096 2021-01-17 [S] [caduca: 2029-01-15]

/etc/apt/trusted.gpg.d/debian-archive-bullseye-security-automatic.gpg
---------------------------------------------------------------------
pub   rsa4096 2021-01-17 [SC] [caduca: 2029-01-15]
      AC53 0D52 0F2F 3269 F5E9  8313 A484 4904 4AAD 5C5D
uid        [desconocida] Debian Security Archive Automatic Signing Key (11/bullseye) <ftpmaster@debian.org>
sub   rsa4096 2021-01-17 [S] [caduca: 2029-01-15]

/etc/apt/trusted.gpg.d/debian-archive-bullseye-stable.gpg
---------------------------------------------------------
pub   rsa4096 2021-02-13 [SC] [caduca: 2029-02-11]
      A428 5295 FC7B 1A81 6000  62A9 605C 66F0 0D6C 9793
uid        [desconocida] Debian Stable Release Key (11/bullseye) <debian-release@lists.debian.org>

/etc/apt/trusted.gpg.d/debian-archive-buster-automatic.gpg
----------------------------------------------------------
pub   rsa4096 2019-04-14 [SC] [caduca: 2027-04-12]
      80D1 5823 B7FD 1561 F9F7  BCDD DC30 D7C2 3CBB ABEE
uid        [desconocida] Debian Archive Automatic Signing Key (10/buster) <ftpmaster@debian.org>
sub   rsa4096 2019-04-14 [S] [caduca: 2027-04-12]

/etc/apt/trusted.gpg.d/debian-archive-buster-security-automatic.gpg
-------------------------------------------------------------------
pub   rsa4096 2019-04-14 [SC] [caduca: 2027-04-12]
      5E61 B217 265D A980 7A23  C5FF 4DFA B270 CAA9 6DFA
uid        [desconocida] Debian Security Archive Automatic Signing Key (10/buster) <ftpmaster@debian.org>
sub   rsa4096 2019-04-14 [S] [caduca: 2027-04-12]

/etc/apt/trusted.gpg.d/debian-archive-buster-stable.gpg
-------------------------------------------------------
pub   rsa4096 2019-02-05 [SC] [caduca: 2027-02-03]
      6D33 866E DD8F FA41 C014  3AED DCC9 EFBF 77E1 1517
uid        [desconocida] Debian Stable Release Key (10/buster) <debian-release@lists.debian.org>

/etc/apt/trusted.gpg.d/debian-archive-stretch-automatic.gpg
-----------------------------------------------------------
pub   rsa4096 2017-05-22 [SC] [caduca: 2025-05-20]
      E1CF 20DD FFE4 B89E 8026  58F1 E0B1 1894 F66A EC98
uid        [desconocida] Debian Archive Automatic Signing Key (9/stretch) <ftpmaster@debian.org>
sub   rsa4096 2017-05-22 [S] [caduca: 2025-05-20]

/etc/apt/trusted.gpg.d/debian-archive-stretch-security-automatic.gpg
--------------------------------------------------------------------
pub   rsa4096 2017-05-22 [SC] [caduca: 2025-05-20]
      6ED6 F5CB 5FA6 FB2F 460A  E88E EDA0 D238 8AE2 2BA9
uid        [desconocida] Debian Security Archive Automatic Signing Key (9/stretch) <ftpmaster@debian.org>
sub   rsa4096 2017-05-22 [S] [caduca: 2025-05-20]

/etc/apt/trusted.gpg.d/debian-archive-stretch-stable.gpg
--------------------------------------------------------
pub   rsa4096 2017-05-20 [SC] [caduca: 2025-05-18]
      067E 3C45 6BAE 240A CEE8  8F6F EF0F 382A 1A7B 6500
uid        [desconocida] Debian Stable Release Key (9/stretch) <debian-release@lists.debian.org>

```

### 3. En que fichero se guarda el anillo de claves que guarda la herramienta apt-key?

El anillo de claves se irá guardando por defecto en **/etc/apt/trusted.gpg**, aunque también se guardarán algunas clavez en **/etc/apt/trusted.gpg.d**.

### 4. ¿Qué contiene el archivo Release de un repositorio de paquetes?. ¿Y el archivo Release.gpg?. Puedes ver estos archivos en el repositorio http://ftp.debian.org/debian/dists/Debian11.6/. Estos archivos se descargan cuando hacemos un apt update

**Release** contiene una lista de los archivos Packages, y sus hash md5sum, sha1sum y sha256sum.

**Release.gpg** contiene la firma del archivo Release.

### 5. Explica el proceso por el cual el sistema nos asegura que los ficheros que estamos descargando son legítimos

**Apt secure** funciona de la siguiente manera:

- Cuando nos conectamos a un repositorio de paquetes .deb, APT descarga los archivos **Packages.gz** (que contiene el archivo Packages comprimido), **Release** y **Release.gpg**. Cuando nos descargamos un paquete binario .deb APT comprueba que su md5sum coincide con la que figura en Packages.
- Para asegurarse de que el archivo **Packages** no ha sido alterado, APT comprueba que su **md5sum** coincide con la que figura en **Release**. Como hemos dicho, release contiene una lista de los archivos Packages, y sus hash md5sum, sha1sum y sha256sum. De esta forma se puede verificar que los ficheros Packages no han sido manipulados, llevando a cabo una comparación del hash que se nos ha facilitado con el hash calculado del fichero descargado.
- Para asegurarse de que el archivo **Release** no ha sido alterado, APT comprueba su firma en el archivo **Release.pgp**. Para ello, APT debe conocer la **clave pública** del que firma el archivo, es decir, esa clave pública debe estar en el archivo ***/etc/apt/trusted.gpg*** o ***/etc/apt/trusted.gpg.d***.

### 6. Añade de forma correcta el repositorio de virtualbox añadiendo la clave pública de virtualbox como se indica en la documentación.

Intento instalar Virtualbox y vemos que no podemos ya que no está añadido el repositorio a ***/etc/apt/sources.list***.

![virtualbox-fallo-instalacion](/img/cripto2/31.png)

Añado el repositorio de Virtualbox:

```
echo "deb https://download.virtualbox.org/virtualbox/debian bullseye contrib" | sudo tee /etc/apt/sources.list.d/virtualbox.list
```

Añado las claves públicas de Virtualbox:

```
wget -q https://www.virtualbox.org/download/oracle_vbox_2016.asc -O- | sudo apt-key add -
wget -q https://www.virtualbox.org/download/oracle_vbox.asc -O- | sudo apt-key add -
```

Actualizamos los paquetes y comprobamos que ahora sí podemos instalar Virtualbox, elegimos la versión 7.0.

```
sudo apt update
sudo apt install virtualbox-7.0
```

![virtualbox-elegir-version](/img/cripto2/32.png)

![virtualbox-instalacion](/img/cripto2/33.png)

--------------------

## Tarea 5: Autentificación: ejemplo SSH

--------------------

Vamos a estudiar como la criptografía nos ayuda a cifrar las comunicaciones que hacemos utilizando el protocolo ssh, y cómo nos puede servir también para conseguir que un cliente se autentifique contra el servidor. Responde las siguientes cuestiones:

### 1. Explica los pasos que se producen entre el cliente y el servidor para que el protocolo cifre la información que se transmite? ¿Para qué se utiliza la criptografía simétrica? ¿Y la asimétrica?

Para establecer una conexión cliente y servidor, el cliente debe iniciar la conexión SSH iniciando el protocolo TCP con el servidor, se creará una conexión simétrica segura, y se verificará si la identidad mostrada por el servidor coincide con los registros anteriores (grabados en un archivo de almacén de claves RSA). Por último se autenticará la conexión mediante credenciales de usuario o utilizando par de claves públicas y privadas. 
A continuación se explica más detalladamente el establecimiento de la conexión:

- Cuando un cliente intenta conectarse, el servidor presenta los protocolos de cifrado y las versiones respectivas que soporta. Si el cliente tiene un par similar de protocolo y versión, se alcanza un acuerdo y se inicia la conexión con el protocolo aceptado.
- Utilizando el protocolo de cifrado acordado, ambas partes usan un Algoritmo de Intercambio de Claves llamado Diffie-Hellman para crear una clave simétrica compartida. La clave se utilizará en adelante para cifrar toda la sesión de comunicación. La clave simétrica solo podrá ser conocida por los dos equipos que realizan la comunicación de forma que esta comunicación no podrá ser interceptada. La clave secreta será específica para cada sesión SSH y se genera antes de la autenticación del cliente. Todos los paquetes que se envíen entre cliente y servidor serán cifrados por la clave, de esta forma, si el cliente se autentifica mediante contraseña, las credenciales estarán protegidas.
- Una vez establecida la sesión cifrada simétricamente el usuario deberá ser autenticado. Aunque las contraseñas se cifran con el método anterior, para tener una conexión segura no se recomienda usar contraseña, sino un par de claves SSH. Es decir, una clave pública y otra privada utilizadas para autenticar al usuario sin necesidad de introducir ninguna contraseña. El cifrado asimétrico no se utiliza para cifrar toda la sesión SSH, sólo se utiliza durante el algoritmo de intercambio de claves de cifrado simétrico.

### 2. Explica los dos métodos principales de autentificación: por contraseña y utilizando un par de claves públicas y privadas.

- **Por contraseña**: como se explicó en el punto anterior, cuando el cliente se autentifica por contraseña, el servidor  pedirá las credenciales asociadas al usuario con el que se conecta. La contraseña será cifrada simétricamente.

- **Mediante par de claves**: en el fichero ~/.ssh/authorized_keys del servidor estará guardada la clave pública del cliente. Cuando el cliente intente iniciar sesión el servidor desconfiará y mandará un mensaje encriptado con la clave pública al cliente. El cliente deberá descifrar el mensaje haciendo uso de la clave privada asociada a esa clave pública.


### 3. En el cliente para que sirve el contenido que se guarda en el fichero ~/.ssh/know_hosts?

En caso de que se establezca conexión entre cliente y servidor mediante ssh por primera vez, se añadirá la clave del host al fichero situado en **~/.ssh/known_hosts**. Por tanto en este fichero estarán las claves de todos los host a los que nos conectemos. Al permanecer guardados, la siguiente vez que nos conectemos lo primero que hará será buscar en este fichero para comprobar si es de confianza, por tanto no nos volverá a preguntar si queremos continuar con la conexión.

### 4. ¿Qué significa este mensaje que aparece la primera vez que nos conectamos a un servidor?

```
     $ ssh debian@172.22.200.74
     The authenticity of host '172.22.200.74 (172.22.200.74)' can't be established.
     ECDSA key fingerprint is SHA256:7ZoNZPCbQTnDso1meVSNoKszn38ZwUI4i6saebbfL4M.
     Are you sure you want to continue connecting (yes/no)? 
```

Como hemos dicho, la primera vez que nos conectamos por ssh a un servidor nos aparece este mensaje de confirmación. Al ser la primera vez que nos conectamos, el sistema no sabe si es un sitio seguro por lo que nos preguntará si estamos seguros de seguir con la conexión. Nosotros como clientes podemos continuar con la conexión o rechazarla dependiendo de si confiamos en el servidor.

### 5. En ocasiones cuando estamos trabajando en el cloud, y reutilizamos una ip flotante nos aparece este mensaje:

```
     $ ssh debian@172.22.200.74
     @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
     @    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
     @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
     IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
     Someone could be eavesdropping on you right now (man-in-the-middle attack)!
     It is also possible that a host key has just been changed.
     The fingerprint for the ECDSA key sent by the remote host is
     SHA256:W05RrybmcnJxD3fbwJOgSNNWATkVftsQl7EzfeKJgNc.
     Please contact your system administrator.
     Add correct host key in /home/jose/.ssh/known_hosts to get rid of this message.
     Offending ECDSA key in /home/jose/.ssh/known_hosts:103
       remove with:
       ssh-keygen -f "/home/jose/.ssh/known_hosts" -R "172.22.200.74"
     ECDSA host key for 172.22.200.74 has changed and you have requested strict checking.
```

Si la clave del host asociada a una dirección IP cambia, a la hora de realizar una conexión SSH nos aparecerá este mensaje, ya que la clave del host actual no coincide con la guardada en el fichero ~/.ssh/known_hosts.

Si estamos seguros de que esa es la IP y de que la clave del host ha podido cambiar podremos solucionar este problema borrando la línea correspondiente a dicho host en el fichero ~/.ssh/known_hosts, como nos dice el mensaje de advertencia anterior que hagamos.

### 6. ¿Qué guardamos y para qué sirve el fichero en el servidor ~/.ssh/authorized_keys?

Es un fichero que contiene una lista de las claves públicas de los sistemas que están autorizados a conectarse al servidor.

En el lado del servidor el demonio SSH verifica si la clave SSH es correcta o no mediante el cálculo de la huella digital de la clave SSH. Si la clave SSH es correcta, permite que el usuario inicie sesión sin pedir usuario ni contraseña.
