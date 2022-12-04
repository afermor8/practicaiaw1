---
title: "Criptografia I. Cifrado asimétrico con gpg y openssl"
date: 2022-12-04T22:05:16+01:00
draft: true
tags: ["SAD"]
bigimg: [{src: "/img/triangle.jpg", desc: "Triangle"}, {src: "/img/sphere.jpg", desc: "Sphere"}, {src: "/img/hexagon.jpg", desc: "Hexagon"}]
---



# Tarea 1: Generación de claves

--------------------

## 1. Genera un par de claves (pública y privada). ¿En que directorio se guarda las claves de un usuario?

Para generar las claves utilizamos el siguiente comando:

```
gpg --gen-key
```

Nos pedirá nombre y apellidos, correo electrónico y confirmación. Lo próximo que nos pedirá será una frase de paso para proteger la clave privada. La escribimos dos veces y a continuación se generará el par de claves. 

![](/img/criptografia/1.png)

Las claves creadas se guardarán en /home/usuario/.gnupg.

![](/img/criptografia/2.png)



## 2. Lista las claves públicas que tienes en tu almacén de claves. Explica los distintos datos que nos muestra. ¿Cómo deberías haber generado las claves para indicar, por ejemplo, que tenga un 1 mes de validez?

Para listar las claves públicas:

```
gpg --list-keys
```

![](/img/criptografia/3.png)

La información que nos aparece al listar las claves públicas es la siguiente:

- pub: public primary key (clave primaria pública)
- uid: unique identifier (identificador único)
- sub: public sub-key (subclave pública)

Cuando listamos las claves privadas **pub** y **sub** cambiarán por **sec** (clave primaria secreta) y **ssb** (subclave secreta). 

**rsa3072** es el algoritmo usado por el par de claves. Lo siguiente es la fech de creación del par de claves (2022-12-04), y, por último, los flags y la fecha de caducidad (2024-12-03). El par de claves maestro tiene los flags **SC** y el par de claves secundario tiene el flag **E**, que tienen el siguiente significado:

- S (Signing): firmar archivos
- C (Certify): certificar una clave, es decir, comprobar firmas de archivos y claves
- E (Encrypt): encriptar y desencriptar información

Para terminar, para generar la clave indicando el tiempo de validez se haría de la siguiente forma:

```
gpg --full-gen-key
```

Durante la creación del par de claves se nos pediría especificar el periodo de validez de la clave. Para que la validez sea de un mes escribimos "1m".

![](/img/criptografia/5.png)


## 3. Lista las claves privadas de tu almacén de claves.

Para listar las claves secretas:

```
gpg --list-secret-keys
```

![](/img/criptografia/4.png)
  
  
  
  
# Tarea 2: Importar / exportar clave pública

--------------------

## 1. Exporta tu clave pública en formato ASCII y guardalo en un archivo nombre_apellido.asc y envíalo al compañero con el que vas a hacer esta práctica. 

Para exportar mi clave pública utilizo el siguiente comando:

```
gpg --export -a "Arantxa" > arantxa.asc
```

Comprobamos que se ha creado el fichero:

![](/img/criptografia/6.png)

Le he enviado este fichero a mi compañero por correo electrónico (gmail).


## 2. Importa las claves públicas recibidas de vuestro compañero.

El fichero que me ha mandado mi compañero por correo es el siguiente:

![](/img/criptografia/7.png)

Ahora importo esta clave pública:

```
gpg --import arturo.asc
```

![](/img/criptografia/8.png)



## 3. Comprueba que las claves se han incluido correctamente en vuestro keyring.
  
Vuelvo a listar las claves para comprobar que se ha incluido correctamente.

```
gpg --list-keys
```

![](/img/criptografia/9.png)



# Tarea 3: Cifrado asimétrico con claves públicas

--------------------

## 1. Cifraremos un archivo cualquiera y lo remitiremos por email a uno de nuestros compañeros que nos proporcionó su clave pública.

Para enviar un fichero cifrado sigo los siguientes pasos.

Primero he creado un fichero de texto de prueba que será el qe envíe a mi compañero.

![](/img/criptografia/10.png)

![](/img/criptografia/11.png)

A continuación uso el comando:

```
gpg -e -u "Árantxa" -r "Arturo Borrero Gonzalez" prueba_gpg.txt
```

donde -u será el remitente y -r el destinatario.

Esto creará el fichero cifrado prueba_gpg.txt.gpg.

![](/img/criptografia/12.png)

Este fichero será el que envíe a mi compañero por correo electrónico.


## 2. Nuestro compañero, a su vez, nos remitirá un archivo cifrado para que nosotros lo descifremos.

Me ha enviado el siguiente fichero cifrado.

![](/img/criptografia/13.png)


## 3. Tanto nosotros como nuestro compañero comprobaremos que hemos podido descifrar los mensajes recibidos respectivamente.

Descargo y descifro el fichero enviado por mi compañero.

```
gpg -d Descargas/captura_pantalla.png.gpg > captura.png
```

![](/img/criptografia/14.png)

Abro captura.png para ver el contenido. Vemos que la imagen que nos ha enviado es una captura del fichero que yo le he enviado, por tanto él también ha podido descifrarlo.

![](/img/criptografia/15.png)



## 4. Por último, enviaremos el documento cifrado a alguien que no estaba en la lista de destinatarios y comprobaremos que este usuario no podrá descifrar este archivo.

Envío mi fichero a otro compañero que no se encuentra en la lista de destinatarios, e intenta descifrarlo. Como no tiene mi clave aparece este error:

![](/img/criptografia/16.png)



## 5. Para terminar, indica los comandos necesarios para borrar las claves públicas y privadas que posees.
  
Primero tendremos que eliminar la clave privada, para ello usamos:

```
gpg --delete-secret-key "Arantxa"
```

Ahora borramos la clave pública.

```
gpg --delete-key "Arantxa"
```

Listamos las claves que tenemos para comprobar que se han borrado.

![](/img/criptografia/17.png)




# Tarea 4: Exportar clave a un servidor público de claves PGP 

--------------------

## 1. Genera la clave de revocación de tu clave pública para utilizarla en caso de que haya problemas.

Gpg generó la clave de revocación automáticamente al crear el par de claves. 

![](/img/criptografia/18.png)

Pero si, por lo que fuera, queremos generar una clave de revocación usamos el comando:

```
gpg --gen-revoke "Arantxa"
```

![](/img/criptografia/19.png)


El certificado de revocación es el que he marcado en la captura. 

Al final se nos advierte de conservar de una manera segura el certificado para que nadie pueda inutilizar mi clave. En este caso no me importa mostrarla ya que la clave la borraré al terminar la práctica.


## 2. Exporta tu clave pública al servidor pgp.rediris.es

Para exportar la clave usamos el siguiente comando con la ID de la clave:

```
gpg --keyserver pgp.rediris.es --send-keys 43140D1577041C18C5830A9B195360276022F67D
```

Cuando he ido a hacer la exportación me ha aparecido el siguiente error:

![](/img/criptografia/20.png)

![](/img/criptografia/21.png)


El servidor de Rediris está caído, por lo que he subido la clave al servidor keys.openpgp.org.

```
gpg --keyserver keys.openpgp.org --send-keys 43140D1577041C18C5830A9B195360276022F67D
```

![](/img/criptografia/22.png)

En el navegador he buscado en keys.openpgp.org la ID de mi clave y aparece la entrada.

![](/img/criptografia/23.png)




## 3. Borra la clave pública de alguno de tus compañeros de clase e impórtala ahora del servidor público de rediris.
  
Borro la clave de mi compañero como hemos explicado anteriormente.

```
gpg --delete-key "Arturo Borrero Gonzalez"
```

![](/img/criptografia/24.png)

Ahora la voy a importar desde el servidor de la siguiente manera:

```
gpg --keyserver keys.openpgp.org --recv-keys AA66280D4EF0BFCC6BFC2104DA5ECB231C8F04C4
```

![](/img/criptografia/25.png)



# Tarea 5: Cifrado asimétrico con openssl

--------------------

## 1. Genera un par de claves (pública y privada).

Para generar las claves utilizaré genrsa con la opción -aes128 para escribir una frase de paso. El nombre que le he dado al fichero donde se guardarán es clave_arantxa.pem. 2048 es el tamaño en bits que tendrá la clave.

```
sudo openssl genrsa -aes128 -out clave_arantxa.pem 2048
```

![](/img/criptografia/26.png)

![](/img/criptografia/27.png)



## 2.  Envía tu clave pública a un compañero.

Para enviar la clave pública primero tendremos que extraerla del fichero que se ha creado (ya que en este fichero están ambas claves, pública y privada). Usamos el siguiente comando:

```
sudo openssl rsa -in clave_arantxa.pem -pubout > clave_arantxa.pub.pem
```

![](/img/criptografia/28.png)

He creado el fichero clave_arantxa.pub.pem que contiene mi clave pública, que será el que le envíe a mi compañero a través de correo electrónico.

Mi compañero a su vez me ha enviado su clave pública.

![](/img/criptografia/29.png)



## 3.  Utilizando la clave pública cifra un fichero de texto y envíalo a tu compañero.

Creo un fichero prueba_openssl.txt.

![](/img/criptografia/30.png)

A continuación cifro el fichero creado del siguiente modo. Tengo que indicar el nombre del fichero que quiero cifrar, el nombre de salida del fichero encriptado y la clave pública que utilizaré:

```
openssl rsautl -encrypt -in prueba_openssl.txt -out prueba_openssl.enc -inkey Descargas/arturobg_sslkey.pub.pem -pubin
```

![](/img/criptografia/31.png)

Le mando el fichero prueba_openssl.enc por correo.


## 4.  Tu compañero te ha mandado un fichero cifrado, muestra el proceso para el descifrado.

He recibido el siguiente fichero de mi compañero.

![](/img/criptografia/33.png)

Descifro el fichero con la opción -decrypt. Además tendré que indicar el fichero que quiero descifrar y el nombre que reibirá el fichero descifrado. Por último indicar la clave que se utilizará para descifrarlo, que en mi caso será mi propia clave, ya que suponemos que nuestro compañero a cifrado el fichero con nuestra clave pública. El comando quedaría así:

```
sudo openssl rsautl -decrypt -in Descargas/openssl.enc -out openssl.txt -inkey clave_arantxa.pem 
```

![](/img/criptografia/34.png)

Ya puedo ver el contenido del fichero.

![](/img/criptografia/35.png)
