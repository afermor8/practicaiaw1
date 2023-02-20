---
title: "Informática Forense"
date: 2023-02-06T12:50:51+01:00
draft: true
tags: ["SAD"]
bigimg: [{src: "/img/triangle.jpg", desc: "Triangle"}, {src: "/img/sphere.jpg", desc: "Sphere"}, {src: "/img/hexagon.jpg", desc: "Hexagon"}]
---

## Descripción

La informática forense es el conjunto de técnicas que nos permite obtener la máxima información posible tras un incidente o delito informático.

En esta práctica, realizarás la fase de toma de evidencias y análisis de las mismas sobre una máquina Linux y otra Windows. Supondremos que pillamos al delincuente in fraganti y las máquinas se encontraban encendidas. Opcionalmente, podéis realizar el análisis de un dispositivo Android.

Sobre cada una de las máquinas debes realizar un volcado de memoria y otro de disco duro, tomando las medidas necesarias para certificar posteriormente la cadena de custodia.

Debes tratar de obtener las siguientes informaciones:

### Apartado A) Máquina Windows.

#### Por comandos:

**1. Procesos en ejecución.**

```bash
tasklist
```

![procesos](/img/forense/1.png)

Para ver los procesos en ejecución:

```bash
tasklist /fi "STATUS eq running"
```

![procesos](/img/forense/2.png)

**2. Servicios en ejecución.**

Mostrar solo los servicios iniciados:

```bash
net start
```

![servicios](/img/forense/3.png)

Mostrar todos los servicios con nombre corto:

```bash
wmic service get name
```

![servicios](/img/forense/4.png)

Mostrar todos los servicios con información adicional.

```bash
sc query
```

![servicios](/img/forense/5.png)

**3. Puertos abiertos.**

```bash
netstat -a
```

![puertos](/img/forense/6.png)

**4. Conexiones establecidas por la máquina.**

```bash
netstat -n
```

![conn](/img/forense/7.png)

**5. Sesiones de usuario establecidas remotamente.**

```bash
query user
query session
```

> Este comando no funciona en Windows 10 Home, pero sí en Windows 10 Pro o Windows Server.

![sesiones](/img/forense/8.png)

**6. Ficheros transferidos recientemente por NetBios.**


MIRAR INCIBE PAG 30

No es posible ver los archivos transferidos recientemente por NetBios en Windows 10, ya que NetBios es un protocolo antiguo que ha sido reemplazado por otros protocolos de red más seguros.

NetBIOS es un protocolo de red que viene activado por defecto en Windows 10 pero que no se utiliza hoy en día. Al estar obsoleto pero activado podría llegar a ser una puerta de entrada para ciberdelincuentes, por lo que es recomendable desactivarlo.

Para monitorear la transferencia de archivos en nuestra red, habría que considerar la implementación de un sistema de monitoreo de red o un software de seguimiento de archivos que se ajuste a nuestras necesidades.

**7. Contenido de la caché DNS.**

```bash
ipconfig /displaydns
```

![cachedns](/img/forense/9.png)

**8. Variables de entorno.**

Se obtiene la lista de variables de entorno con el comando `set`.

![variables](/img/forense/10.png)

#### Analizando el Registro de Windows:

El registro de Windows almacena información detallada sobre eventos y acciones que tienen lugar en un sistema operativo Windows. La información relacionada con las acciones mencionadas anteriormente se encuentra en diferentes archivos de registro de Windows. Para acceder a ellos y ver la información específica, uno puede usar la utilidad de registro de eventos o el Editor del Registro de Windows (regedit).

**9. Dispositivos USB conectados.**

En Registro de Windows vamos a `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Enum\USB\` y ahí veremos los dispositivos conectados. Por ejemplo:

![usbs](/img/forense/11.png)

**10. Redes wifi utilizadas recientemente.**

Para ver las redes wifi utilizadas recientemente vamos a `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\NetworkList\Profiles`.

![redes](/img/forense/12.png)

**11. Configuración del firewall de nodo.**

En `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\SharedAccess\Parameters\FirewallPolicy` estará toda la configuración del Firewall.

![firewall](/img/forense/13.png)
![firewall](/img/forense/14.png)


**12. Programas que se ejecutan en el Inicio.**

Miramos en `HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Run`.

![redes](/img/forense/15.png)

**13. Asociación de extensiones de ficheros y aplicaciones.**

En `HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Explorer\FileExts` nos aparecerán todas las exteniones de ficheros y aplicaciones y su asociación. Por ejemplo en la extenión .html, si vemos la opción elegida por el usuario vemos que FirefoxHTML.

![redes](/img/forense/16.png)

**14. Aplicaciones usadas recientemente.**

No hay una ubicación específica en el Registro de Windows 10 que proporcione información sobre las aplicaciones que se han usado recientemente. Algunos programas y aplicaciones pueden almacenar información en el Registro de Windows, pero esto varía según el programa y no es una función estándar de Windows 10.

Una forma de ver Elementos recientes utilizados en Windows (ficheros, aplicaciones...) es presionando el símbolo de Windows + R y escribimos Shell:Recent. Se nos abrirá lo siguiente:

![recientes](/img/forense/17.png)

**15. Ficheros abiertos recientemente.**

En `HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Explorer\RecentDocs`.

![ficheros](/img/forense/18.png)

**16. Software Instalado.**

Los programas instalados se encuentran en `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall`.

![ficheros](/img/forense/19.png)

**17. Contraseñas guardadas.**

En el Registro de Windows solo se puede ver información relacionada con las contraseñas pero no podemos ver las contraseñas guardadas ya que se encuentran cifradas. Una forma de ver las contraseñas de Windows y las contraseñas guardadas de internet es con el **Administrador de credenciales**.

![contraseñas](/img/forense/20.png)

**18. Cuentas de Usuario.**

Podemos ver las cuentas de usuario creadas en `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\ProfileList`. En mi máquina solo tengo una cuenta de usuario llamada 'usuario' por lo que solo aparecerá una.

![usuarios](/img/forense/21.png)

#### Con Aplicaciones de terceros:

**19. Historial de navegación y descargas. Cookies.**

Con la herramienta **BrowsingHistoryView** podemos comprobar el historial de las webs más utilizadas.

Me descargo la herramienta de la [web oficial](https://www.nirsoft.net/utils/browsinghistoryview.zip) y hago una búsqueda de todos los navegadores en los últimos diez días.

![bhv](/img/forense/23.png)

Entre lás búsquedas del historial de navegación podemos ver también las descargas.

![bhv](/img/forense/24.png)

Para las coockies me he descargado la herramienta **MozillaCookiesView** ya que Firefox es el navegador que más se ha utilizado. Me descargo el programa de la [web oficial](http://www.nirsoft.net/utils/mzcv.html) y lo ejecuto. Me aparecerán automáticamente las coockies guardadas.

![mcv](/img/forense/25.png)

**20. Volúmenes cifrados.**

He creado un volumen cifrado mediante VeraCrypt. Usando **Encrypted Disk Detector (EDD)** voy a comprobar los volúmenes cifrados que tiene la máquina. Para ello me he [descargado](https://www.softpedia.com/get/Security/Security-Related/Encrypted-Disk-Detector.shtml#download) EDD y lo ejecuto. Me aparecerá lo siguiente.

![edd](/img/forense/22.png)

En la captura vemos que el volumen cifrado está montado en **X:** y que lo ha detectado correctamente.

#### Sobre la imagen del disco:

Para analizar los puntos anteriores utilizaremos **Autopsy**. Lo descargamos desde la [web oficial](https://www.autopsy.com/download/). Tras instalarlo lo ejecutamos como administrador y empezamos rellenando un nuevo caso.

![autopsy](/img/forense/26.png)

![autopsy](/img/forense/27.png)

![autopsy](/img/forense/28.png)

Ahora seleccionamos el host y el tipo de fuente de datos.

![autopsy](/img/forense/29.png)

![autopsy](/img/forense/30.png)

Seleccionamos el disco que se analizará y los módulos ingest.

![autopsy](/img/forense/31.png)

![autopsy](/img/forense/32.png)

Entonces comenzará el proceso para añadir todos los módulos a la base de datos local.

![autopsy](/img/forense/33.png)

Cuando se añada todo podremos analizar cada uno de los puntos.

![autopsy](/img/forense/34.png)

**21. Archivos con extensión cambiada.**

En Analysis Results -> Extension Mismatch Detected podemos ver los archivos con extensión cambiada. Como prueba cambié de extensión los archivos x.doc y z.pdf.

![autopsy](/img/forense/38.png)

Si hago doble click sobre x.doc podemos comprobar que el fichero en realidad es una imagen.

![autopsy](/img/forense/39.png)

Si seleccionamos z.pdf también podremos ver que era una imagen anteriormente.

![autopsy](/img/forense/40.png)

**22. Archivos eliminados.**

Para ver los archivos eliminados nos vamos a la pestaña File Views -> Deleted Files.

![autopsy](/img/forense/35.png)

**23. Archivos Ocultos.**

No hay una funcionalidad para ver los ficheros ocultos, pero sí podemos ver si un fichero es oculto o no. Para comprobarlo voy a seleccionar un fichero oculto que creé a modo de prueba. Entre la información del fichero vemos que tiene un Flag llamada Hidden (oculto).

![autopsy](/img/forense/46.png)

**24. Archivos que contienen una cadena determinada.**

Para esto primero vamos a realizar una búsqueda por palabra clave.

![autopsy](/img/forense/36.png)

Ahora nos vamos a la opción Analysis Results > Keyword Hits. Aquí podremos ver los resultados de búsquedas por palabras claves cuyas coincidencias pueden ser subcadena, correos, palabras literales, expresiones regulares...

![autopsy](/img/forense/37.png)

**25. Búsqueda de imágenes por ubicación.**

En la opción Images/Video de la barra superior hacemos click y se nos abre una nueva ventana en la que podemos buscar las imágener por ubicación. En mi caso me voy a ir a la ubicación Pictures -> Saved Pictures, y ahí vemos algunas imágenes que he descargado anteriormente.

![autopsy](/img/forense/41.png)

**26. Búsqueda de archivos por autor.**

------------------------

Por último generamos el informe. Hacemos click en Tools y Generate Report. Se nos abre la siguiente ventana:

![autopsy](/img/forense/42.png)

He seleccionado que el informe sea formato texto. Le damos a siguiente y finalizar.

![autopsy](/img/forense/43.png)

![autopsy](/img/forense/44.png)

Comienza la generación. Cuando termine seleccionar la ruta y se nos abrirá el informe.

![autopsy](/img/forense/45.png)

![autopsy](/img/forense/47.png)

#### Volcado de memoria y de disco duro

Me he descargado [DumpIt](https://github.com/thimbleweed/All-In-USB/blob/master/utilities/DumpIt/DumpIt.exe?raw=true) para Windows 10.

Ejecuto el .exe como administrador.

![autopsy](/img/forense/48.png)

Escribimos 'y' para continuar y cuando termine el proceso tendremos en la dirección de destino el volcado de memoria.

![autopsy](/img/forense/49.png)

![autopsy](/img/forense/50.png)

También puede resultar de gran interés para el análisis forense la obtención de la memoria virtual, por lo que es recomendable adquirir el fichero pagefile.sys siempre que sea posible. Para ello utilizaremos la herramienta especializada [NTFSCopy](https://tzworks.com/download_links.php).

Ejecutamos Símbolo del Sistema como administrador y escribimos lo siguiente:

```bash
ntfscopy64.exe c:\pagefile.sys o:\pagefile.sys –raw –MD5
```

#### Cadena de custodia

La cadena de custodia es un documento que habrá que rellenar con todas las evidencias recogidas. Debe estar claramente documentada y se deben detallar los siguientes puntos:

```text
CADENA DE CUSTODIA DE EVIDENCIAS
Número de caso:
Tipo de incidente:
Empresa afectada:
Dirección:
Teléfono:
Fecha y Hora:
Investigador:
Observaciones

Listado de evidencias
Número de caso:
Número de página:

Evidencia (Código): 
Cantidad: 
Descripción del artículo (Marca, Modelo, Número de serie, estado, etc):

Evidencia (Código): 
Cantidad: 
Descripción del artículo (Marca, Modelo, Número de serie, estado, etc):

Evidencia (Código): 
Cantidad: 
Descripción del artículo (Marca, Modelo, Número de serie, estado, etc):
```

### Apartado B) Máquina Linux.

Intenta realizar las mismas operaciones en una máquina Linux para aquellos apartados que tengan sentido y no se realicen de manera idéntica a Windows.

#### Por comandos en Debian 11:

**1. Procesos en ejecución.**

```bash
ps aux
```

**2. Servicios en ejecución.**

Para ver los servicios activos en Debian se puede hacer de varias formas, por ejemplo:

```bash
sudo systemctl status
```

Ó

```bash
systemctl list-units --type=service
```

**3. Puertos abiertos.**

```bash
ss -puntal
```

**4. Conexiones establecidas por la máquina.**

```bash
ss -punta | egrep ESTAB
```

**5. Sesiones de usuario establecidas.**

```bash
w
```

**7. Contenido de la caché DNS.**

En otras distribuciones Linux se podría utilizar systemd-resolve, pero en Debian 11 no se puede.

**8. Variables de entorno.**

```bash
printenv
```

**9. Dispositivos USB**

```bash
lsusb
```

**10. Redes wifi utilizadas recientemente.**

```bash
sudo iwconfig
ip a
```

**11. Configuración del firewall.**

```bash
sudo nft list ruleset
sudo iptables-save 
sudo ip6tables-save 
```

**16. Software Instalado.**

```bash
dpkg -l
dpkg --get-selections
```

**18. Cuentas de Usuario**

Para ver los usuarios que tiene creados la máquina.

```bash
cat /etc/passwd
```


#### Con Aplicaciones de terceros:


**12. Programas que se ejecutan en el Inicio.**

**13. Asociación de extensiones de ficheros y aplicaciones.**

**17. Contraseñas guardadas.**



**14. Aplicaciones usadas y ficheros abiertos recientemente.**

Instalar Zeitgeist Explorer.

```bash
sudo apt install zeitgeist-explorer
```

Ejecutamos el programa y podremos ver las aplicaciones y ficheros usados recientemente.

https://www.rootsolutions.com.ar/desinstalando-el-software-espia-en-debian-8/


**19. Historial de navegación y descargas. Cookies.**

Con Zeitgeist también podemos ver el historial de navegación.

CAPTURA

Para ver las cookies me voy a descargar el programa MozillaCookiesView.

https://unix.stackexchange.com/questions/82597/where-does-firefox-store-its-cookies-on-linux


**20. Volúmenes cifrados**

Mirar Encrypted Disk Detector (EDD)

#### Sobre la imagen del disco:

https://www.solvetic.com/tutoriales/article/2325-analisis-forense-de-discos-duros-y-particiones-con-autopsy/

https://byte-mind.net/analisis-forense-con-volatility/

https://urjc-ctf.github.io/web/2021-22/mod2/2_forense_sesion2.pdf


```bash
sudo apt install autopsy
```

**21. Archivos con extensión cambiada.**

**22. Archivos eliminados.**

**23. Archivos Ocultos.**

**24. Archivos que contienen una cadena determinada.**

**25. Búsqueda de imágenes por ubicación.**

**26. Búsqueda de archivos por autor.**



#### Volcado de memoria y de disco duro

tomando las medidas necesarias para certificar posteriormente la cadena de custodia





### Apartado C)

En un dispositivo Android, trata de hacer un volcado de memoria y recuperar información de ubicación, llamadas, mensajes, aplicaciones de mensajería, perfiles en redes sociales, etc...







----------------------------

Algunos kits de herramientas forenses open-source:

- Caine
- DFF (Digital Forensics Framework)
- The Sleuth Kit
- Helix Live CD
- Digital Evidence and Forensics Toolkit (DEFT)

Para análisis de memoria:

- Volatility
- Access Data FTK Imager
- MDD

Para análisis de registro de Windows:

- AccessData Registry Viewer
- Kape
- Fred (Forensic Registry Editor)

Para análisis de discos duros:

- Autopsy

Herramientas gratuitas para Android:

- AFLogical OSE
- OSAF (Open Source Android Forensics)
- Andriller
- ADEL (Android Data Extractor Lite)
- WhatsApp Xtract
- Skype Xtractor
- Android Pattern Lock Cracker

Para saber algo más:

- Cursos básicos de Análisis Forense Linux y Windows en OpenWebinars.
- Cursos gratuitos en TryHackMe.
