---
title: "Renovar certificado digital con el configurador de la FNMT en Debian 11"
date: 2023-06-28T11:27:43+02:00
draft: true
tags: ["Otros"]
bigimg: [{src: "/img/triangle.jpg", desc: "Triangle"}, {src: "/img/sphere.jpg", desc: "Sphere"}, {src: "/img/hexagon.jpg", desc: "Hexagon"}]
---

### Configurador FNMT-RCM

Tras descargarnosMe descargo el paquete .deb de la [web de la FNMT](https://www.sede.fnmt.gob.es/descargas/descarga-software/instalacion-software-generacion-de-claves). Intento instalarlo usando dpkg, pero me aparece un problema de dependencias.

```bash
arantxa@tars:~$ sudo dpkg --install Descargas/configuradorfnmt_1.0.1-0_amd64.deb 
```

```
Seleccionando el paquete configuradorfnmt previamente no seleccionado.
(Leyendo la base de datos ... 236214 ficheros o directorios instalados actualmente.)
Preparando para desempaquetar .../configuradorfnmt_1.0.1-0_amd64.deb ...
Desempaquetando configuradorfnmt (1.0.1-0) ...
dpkg: problemas de dependencias impiden la configuración de configuradorfnmt:
 configuradorfnmt depende de libcanberra-gtk-module; sin embargo:
  El paquete `libcanberra-gtk-module' no está instalado.

dpkg: error al procesar el paquete configuradorfnmt (--install):
 problemas de dependencias - se deja sin configurar
Procesando disparadores para gnome-menus (3.36.0-1) ...
Procesando disparadores para desktop-file-utils (0.26-1) ...
Procesando disparadores para mailcap (3.69) ...
Se encontraron errores al procesar:
 configuradorfnmt
```

Instalamos el paquete necesario, pero al hacerlo nos dice que hay dependencias incumplidas.

```bash
arantxa@tars:~$ sudo apt install libcanberra-gtk-module
```

```txt
Leyendo lista de paquetes... Hecho
Creando árbol de dependencias... Hecho
Leyendo la información de estado... Hecho
Tal vez quiera ejecutar «apt --fix-broken install» para corregirlo.
Los siguientes paquetes tienen dependencias incumplidas:
 libcanberra-gtk-module : Depende: libcanberra-gtk0 (>= 0.2) pero no va a instalarse
E: Dependencias incumplidas. Intente «apt --fix-broken install» sin paquetes (o especifique una solución).
```

Así que utilizo el comando `apt --fix-broken install`.

```bash
arantxa@tars:~$ sudo apt --fix-broken install
Leyendo lista de paquetes... Hecho
Creando árbol de dependencias... Hecho
Leyendo la información de estado... Hecho
Corrigiendo dependencias... Listo
Se instalarán los siguientes paquetes adicionales:
  libcanberra-gtk-module libcanberra-gtk0
Se instalarán los siguientes paquetes NUEVOS:
  libcanberra-gtk-module libcanberra-gtk0
0 actualizados, 2 nuevos se instalarán, 0 para eliminar y 17 no actualizados.
1 no instalados del todo o eliminados.
Se necesita descargar 27,0 kB de archivos.
Se utilizarán 92,2 kB de espacio de disco adicional después de esta operación.
¿Desea continuar? [S/n] s
Des:1 http://deb.debian.org/debian bullseye/main amd64 libcanberra-gtk0 amd64 0.30-7 [12,5 kB]
Des:2 http://deb.debian.org/debian bullseye/main amd64 libcanberra-gtk-module amd64 0.30-7 [14,5 kB]
Descargados 27,0 kB en 0s (181 kB/s)               
Obteniendo informes de fallo... Finalizado
Analizando información Encontrada/Corregida... Finalizado
Seleccionando el paquete libcanberra-gtk0:amd64 previamente no seleccionado.
(Leyendo la base de datos ... 236225 ficheros o directorios instalados actualmen
te.)
Preparando para desempaquetar .../libcanberra-gtk0_0.30-7_amd64.deb ...
Desempaquetando libcanberra-gtk0:amd64 (0.30-7) ...
Seleccionando el paquete libcanberra-gtk-module:amd64 previamente no seleccionad
o.
Preparando para desempaquetar .../libcanberra-gtk-module_0.30-7_amd64.deb ...
Desempaquetando libcanberra-gtk-module:amd64 (0.30-7) ...
Configurando libcanberra-gtk0:amd64 (0.30-7) ...
Configurando libcanberra-gtk-module:amd64 (0.30-7) ...
Configurando configuradorfnmt (1.0.1-0) ...
cp: no se puede efectuar `stat' sobre 'configuradorfnmt.js': No existe el ficher
o o el directorio
cp: no se puede efectuar `stat' sobre 'configuradorfnmt.js': No existe el ficher
o o el directorio
tar (child): jre.tar.gz: No se puede efectuar open: No existe el fichero o el di
rectorio
tar (child): Error is not recoverable: exiting now
tar: Child returned status 2
tar: Error is not recoverable: exiting now
/var/lib/dpkg/info/configuradorfnmt.postinst: 13: gconftool-2: not found
/var/lib/dpkg/info/configuradorfnmt.postinst: 14: gconftool-2: not found
Procesando disparadores para libc-bin (2.31-13+deb11u6) ...
needrestart is being skipped since dpkg has failed
```

Si se intenta instalar el paquete ahora es posible que funcione pero nos aparecerán algunos mensajes de error relacionados con la herramienta **gconftool-2**. Esta herramienta pertenece al paquete **gconf2**, que es utilizado por algunas aplicaciones antiguas que aún lo requieren. 

Por tanto los pasos que seguiré serán los siguientes:

```bash
#Instalar el paquete gconf2
sudo apt install gconf2

#Eliminar el paquete configuradorfnmt y purgar sus archivos de configuración.
sudo rm /var/lib/dpkg/info/configuradorfnmt.prerm
sudo dpkg --purge configuradorfnmt

#Instalar el paquete configuradorfnmt
sudo dpkg -i Descargas/configuradorfnmt_1.0.1-0_amd64.deb 
```

Habiendo realizado estos pasos, todo debería funcionar correctamente. Lanzamos el programa desde la terminal para probarlo.

```bash
arantxa@tars:~$ configuradorfnmt
```

```bash
jun 28, 2023 11:51:24 AM es.gob.fnmt.cert.certrequest.util.FnmtLogManager install
INFORMACIÓN: La ruta para el fichero de registro ('/home/arantxa/.fnmt') no existe, se creara
jun 28, 2023 11:51:24 AM es.gob.fnmt.cert.certrequest.CertRequest main
INFORMACIÓN: Modo de depuracion desactivado
jun 28, 2023 11:51:24 AM es.gob.fnmt.cert.certrequest.CertRequest main
INFORMACIÓN: Version actual de la aplicacion: '1'
jun 28, 2023 11:51:25 AM es.gob.fnmt.cert.certrequest.CertRequest printSystemInfo
INFORMACIÓN: Aplicacion FNMT de solicitud de certificados version 1.
 Sistema operativo: Linux.
 Version del sistema operativo: 5.10.0-23-amd64.
 Version de Java: 1.8.0_251.
 Arquitectura del JRE: 64 bits.
 Fabricante de la JVM: Oracle Corporation.
 Idioma por defecto: es_ES.
 Tamano actual en memoria: 596MB.
 Tamano maximo de memoria: 8839MB.
 Memoria actualmente libre: 480MB.
jun 28, 2023 11:51:25 AM es.gob.fnmt.cert.certrequest.ProxyUtil setDefaultProxy
INFORMACIÓN: No se usara proxy para las conexiones de red
jun 28, 2023 11:51:25 AM es.gob.fnmt.cert.certrequest.Updater$1 checkServerTrusted
INFORMACIÓN: Verificado correctamente el certificado SSL: www.sede.fnmt.gob.es
jun 28, 2023 11:51:25 AM es.gob.afirma.core.misc.http.UrlHttpManagerImpl enableSslChecks
INFORMACIÓN: Habilitadas comprobaciones SSL
jun 28, 2023 11:51:25 AM es.gob.fnmt.cert.certrequest.Updater getLatestVersion
GRAVE: No se ha podido obtener la ultima version disponible desde https://www.sede.fnmt.gob.es/documents/10445900/10528994/certrequest.version.linux: es.gob.afirma.core.misc.http.HttpError
jun 28, 2023 11:51:25 AM es.gob.fnmt.cert.certrequest.Updater isNewVersionAvailable
GRAVE: No se puede comprobar si hay versiones nuevas del aplicativo
jun 28, 2023 11:51:25 AM es.gob.fnmt.cert.certrequest.Updater lambda$checkForUpdates$1
INFORMACIÓN: No se ha podido comprobar la disponibilidad de nueva version: java.lang.UnsupportedOperationException: No se puede comprobar si hay versiones nuevas del aplicativo, no hay informacion sobre la ultima version disponible
```

Me aparece la siguiente ventana:

![](/img/otros/1.png)

Le doy a Aceptar y ya podremos renovar el certificado o realizar la operación que queramos.

### Renovación del certificado

Vamos a la [web de la FNMT para renovar el certificado](https://www.sede.fnmt.gob.es/certificados/persona-fisica/renovar) y vamos al segundo paso ["Solicitar la renovación"](https://www.sede.fnmt.gob.es/certificados/persona-fisica/renovar/solicitar-renovacion). Hago click donde pone "Pulse aquí para consultar y aceptar las condiciones de expedición del certificado", aceptamos y después le damos a siguiente.

![](/img/otros/2.png)

Nos aparecerá la siguiente ventana del configurador de la FNMT donde ponemos nuestra contraseña.

![](/img/otros/3.png)

Rellenamos con nuestros datos:

![](/img/otros/4.png)

Confirmamos que los datos son correctos y firmamos con nuestro anterior certificado.

![](/img/otros/5.png)

![](/img/otros/6.png)

La solicitud ya se habrá realizado con éxito.

![](/img/otros/7.png)

Nos llegará un correo a la dirección que hayamos puesto con la información ncesaria que tendremos que usar para descargar el nuevo certificado.

Vamos al tercer paso de la web de la FNMT de renovación del certificado, donde pone ["Descarga del Certificado"](https://www.sede.fnmt.gob.es/certificados/persona-fisica/renovar/descargar-certificado). Rellenamos con nuestros datos y descargamos el certificado.

![](/img/otros/8.png)

Se procederá a instalar el nuevo certificado.

![](/img/otros/9.png)

Introducimos la contraseña del certificado.

![](/img/otros/10.png)

Creamos una copia de seguridad del certificado generado.

![](/img/otros/11.png)

Volvemos a introducir la contraseña del certificado.

![](/img/otros/12.png)

Y ya se habrá instalado correctamente el certificado en nuestro navegador.

![](/img/otros/13.png)

Nos aparece un mensaje que nos dice que si queremos descargar el certificado debemos usar el mismo navegador.

![](/img/otros/14.png)

Yo ya tengo la copia generada así que no me importaré el certificado.

Podemos comprobar que el certificado funciona correctamente desde la web de [VALIDe](https://valide.redsara.es/valide/validarCertificado/ejecutar.html;jsessionid=BF7A1DE7CEAEA68ADA1EEFDAF2DBBB7B).

![](/img/otros/15.png)
