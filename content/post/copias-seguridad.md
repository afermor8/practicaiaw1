---
title: "Sistema de copias de seguridad"
date: 2023-06-03T12:29:08+02:00
draft: true
tags: ["ASO"]
bigimg: [{src: "/img/triangle.jpg", desc: "Triangle"}, {src: "/img/sphere.jpg", desc: "Sphere"}, {src: "/img/hexagon.jpg", desc: "Hexagon"}]
---


## Implementar un sistema de copias de seguridad para las instancias del cloud, teniendo en cuenta las siguientes características:

- **Selecciona una aplicación para realizar el proceso: bacula, amanda, shell script con tar, rsync, dar, afio, etc.**
- **Utiliza una de las instancias como servidor de copias de seguridad, añadiéndole un volumen y almacenando localmente las copias de seguridad que consideres adecuadas en él.**
- **El proceso debe realizarse de forma completamente automática**
- **Selecciona qué información es necesaria guardar (listado de paquetes, ficheros de configuración, documentos, datos, etc.)**
- **Realiza semanalmente una copia completa**
- **Realiza diariamente una copia incremental o diferencial (decidir cual es más adecuada)**
- **Implementa una planificación del almacenamiento de copias de seguridad para una ejecución prevista de varios años, detallando qué copias completas se almacenarán de forma permanente y cuales se irán borrando**
- **Selecciona un directorio de datos "críticos" que deberá almacenarse cifrado en la copia de seguridad, bien encargándote de hacer la copia manualmente o incluyendo la contraseña de cifrado en el sistema**
- **Incluye en la copia los datos de las nuevas aplicaciones que se vayan instalando durante el resto del curso**
- **Utiliza una ubicación secundaria para almacenar las copias de seguridad. Solicita acceso o la instalación de las aplicaciones que sean precisas.**

## La corrección consistirá tanto en la restauración puntual de un fichero en cualquier fecha como la restauración completa de una de las instancias la última semana de curso.

---------------------------------------

***!!!NOTA: Si estás viendo esto y eres de otro año, decirte que esta entrada la creé mientras hacía investigación sobre cómo funcionaba Rsync y como implementarlo para hacer copias de seguridad usando scripts bash y crontab. Pero debido a que mañana es mi último día de curso y no creo que siga actualizando más sobre el tema, decirte que NO HE PODIDO PROBAR NADA y NO SÉ SI FUNCIONA realmente nada de lo ecrito en este post. Aún así te puede servir de base para seguir investigando y terminar de hacerlo por ti mismo. Suerte!!***

----------------------------------

### Introducción

La aplicación que he decidido utilizar para realizar el proceso de copias de seguridad es ***rsync***.

Rsync es una herramienta de sincronización y copia de archivos utilizada principalmente en sistemas operativos tipo Unix. Proporciona una forma eficiente de transferir datos entre sistemas locales o remotos, ya sea en la misma máquina o a través de una red.

La característica distintiva de rsync es su capacidad para copiar y sincronizar solo las diferencias entre archivos existentes, lo que se conoce como "differential synchronization". En lugar de transferir archivos completos cada vez, rsync analiza los archivos de origen y destino y transfiere solo las partes que han cambiado, lo que minimiza el ancho de banda utilizado y acelera la sincronización.

Rsync también tiene otras características útiles, como la compresión de datos durante la transferencia, la posibilidad de preservar los atributos del archivo, la opción de copiar enlaces simbólicos y la capacidad de sincronizar directorios de forma recursiva.

Además de la transferencia local, rsync es ampliamente utilizado para realizar copias de seguridad remota, replicación de servidores y sincronización de archivos en sistemas distribuidos. Su versatilidad y eficiencia lo convierten en una herramienta muy popular en entornos de administración de sistemas y respaldo de datos.

### 1. Instalación y preparación del volumen

En Debian viene instalado por defecto, pero si no lo tuviéramos se puede instalar fácilmente con apt.

```bash
sudo apt install rsync
```

Voy a utilizar rsync en mi escenario montado en openstack, que es el siguiente:

![](/img/backup/3.png)


| Máquina         | IPs                                     | SO                    |
|:--------------: |:---------------------------------------:| :--------------------:|
| alfa            | 172.22.201.158<br>172.16.0.1<br>192.168.0.1 | Debian 11             |
| bravo           | 172.16.0.200                            | Rocky Linux 9         |
| charlie         | 192.168.0.2                             | Contenedor LXC Ubuntu |
| delta           | 192.168.0.3                             | Contenedor LXC Ubuntu |

El lugar donde se guardarán las copias de seguridad será un nuevo volumen asociado a la máquina **alfa**. El volumen que he creado tiene 20GB, que para esta actividad debería ser suficiente, pero en un entorno laboral se necesitaría mucho más epacio Para ello he creado el nuevo volumen en Openstack.

![](/img/backup/1.png)

Este volumen lo voy a asociar a **alfa**.

![](/img/backup/2.png)

**En alfa:**

```bash
arantxa@alfa:~$ lsblk
NAME    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
vda     254:0    0   30G  0 disk 
├─vda1  254:1    0 29.9G  0 part /
├─vda14 254:14   0    3M  0 part 
└─vda15 254:15   0  124M  0 part /boot/efi
vdb     254:16   0   20G  0 disk 
```

En mi caso no está montado, pero si se hubiera montado lo desmontamos.

```bash
arantxa@alfa:~$ sudo umount /dev/vdb
umount: /dev/vdb: not mounted.
```

Instalo el paquete xfsprogs:

```bash
sudo apt-get install xfsprogs
```

Formateamos el volumen usando el sistema de ficheros ***xfs***.

```bash
arantxa@alfa:~$ sudo mkfs.xfs /dev/vdb
meta-data=/dev/vdb               isize=512    agcount=4, agsize=1310720 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=0
         =                       reflink=1    bigtime=0
data     =                       bsize=4096   blocks=5242880, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1
log      =internal log           bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
Discarding blocks...Done.
```

Ahora voy a crear un punto de montaje:

```bash
sudo mkdir /backup
```

Voy a usar fstab para que el volumen se monte automáticamente al iniciar el sistema. Creo un respaldo del fichero /etc/fstab por seguridad.

```bash
sudo cp /etc/fstab /etc/fstab.backup
```

Edito fstab para montar el volumen automáticamente con el sistema de archivos XFS en /backup.

```bash
sudo nano /etc/fstab
```

```txt
/dev/vdb   /backup   xfs   defaults   0   0
```

Y monto el volumen externo en el punto de montaje:

```bash
arantxa@alfa:~$ sudo mount /dev/vdb /backup
```

Si reiniciamos la máquina podemos comprobar que el volumen se ha montado autmática y correctamente:

```bash
arantxa@alfa:~$ df
Filesystem     1K-blocks    Used Available Use% Mounted on
udev              990444       0    990444   0% /dev
tmpfs             201816     520    201296   1% /run
/dev/vda1       30791432 4029844  25472532  14% /
tmpfs            1009076       0   1009076   0% /dev/shm
tmpfs               5120       0      5120   0% /run/lock
/dev/vda15        126678   10900    115778   9% /boot/efi
tmpfs             201812       0    201812   0% /run/user/1000
/dev/vdb        20466256      24  19401272   1% /backup

arantxa@alfa:~$ lsblk
NAME    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
vda     254:0    0   30G  0 disk 
├─vda1  254:1    0 29.9G  0 part /
├─vda14 254:14   0    3M  0 part 
└─vda15 254:15   0  124M  0 part /boot/efi
vdb     254:16   0   20G  0 disk /backup
```

### 2. Información a guardar

**Fichero de directorios a respaldar:**

Voy a crear un fichero de texto en el que agregaré línea por línea los directorios a los que hay que hacerle copias de seguridad. Este fichero lo usaré más tarde con el comando rsync.

```bash
cd /backup/
nano a_respaldar.txt
```

```txt
/etc
/var/log
/var/lib
/root
/home
/opt
/srv
/usr/share
```

Todo ese será el contenido del que se hará copia de seguridad. Pero en mis servidores hay algunos ficheros "críticos" que deberían ser guardados de forma cifrada (sin embargo en este post no me centraré en ello, investígalo por ti mismo). Estos ficheros podrían ser:

- Contraseñas
- Configuración de red de las máquinas
- En alfa: fichero de reglas iptables, config del servidor LDAP (ldap.conf, nsswitch.conf, sssd.conf...), SSHD (sshd_config)...
- En bravo: servidor web (/var/www, /etc/httpd), cliente LDAP (ldap.conf, sssd.conf...), cliente ssh (ssh_config)...
- En charlie: servidor dns (/var/chache/bind, /etc/bind...)
- En delta: servidor de correo (/etc/postfix), cliente LDAP...

### 3. Información a excluir

Aunque en mi caso voy a indicar los directorios de los que quiero hacer copia de seguridad es una buena práctica indicar en el comando rsync algunos directorios a expluir si hacemos una copia completa de / (raiz). Por ejemplo:

```bash
rsync -aAXv --exclude={"/dev/*","/proc/*","/sys/*","/tmp/*","/run/*","/mnt/*","/media/*","/lost+found"} / /path/to/backup/folder
```

Al utilizar las opciones -aAX, los archivos se preserva los permisos, propietarios, grupos, atributos extendidos y los atributos de contexto de seguridad de los archivos en sistemas de archivos compatibles, entre otros (enlaces simbólicos, dispositivos, permisos, propiedades, tiempos de modificación, ACLs...).

Como he dicho, en mi caso, indicaré los directorios de los que voy a hacer copia de seguridad, por lo que no es necesario indicar todos esos directorio innecesarios.

Habría que tener en cuenta que si no se especifican los directorios que se quieren respaldar, sino que se epecifican los que se quieren excluir, hay que indicar el directorio donde tenemos nuestro volumen montado. En mi caso sería /backup, es decir, donde se guardan las copias de seguridad, de forma que al realizar una copia de seguridad no se cree un bucle infinito que pueda llenar el disco rápidamente. Se puede hacer especificandolo en el comando:

```bash
--exclude={"/backup"}
```

O creando un fichero donde añadiremos todos los directorios a excluir. Esta opción es mejor por si en el futuro quisiéramos agregar alguno más:

```bash
nano archivos_a_excluir.txt
```

```
/backup
```

```bash
--exclude-from="archivos_a_excluir.txt"
```

Otra cosa a tener en cuenta es decidir si queremos excluir subdirectorios sin importancia como:

- /home/*/.thumbnails/
- /home/*/.cache/mozilla/
- /home/*/.cache/chromium/
- /home/*/.local/share/Trash/
- Y si GVFS está instalado debe excluirse /home/*/.gvfs para evitar errores de rsync.

### 4. Esquema de copias de seguriadad

**Copia diferencial (diaria):**

He decidido que la copia diaria será de tipo diferencial. Las copias diferenciales requieren más espacio de almacenamiento y más tiempo de respaldo, pero la restauración es más rápida, ya que solo se necesita acceder a la copia completa y la última copia diferencial. Es adecuada si se desea minimizar el tiempo de restauración y no se tienen restricciones de espacio de almacenamiento.

Se guardarán las copias de todos los días de la semana, y cuando termine la semana se empezará a sobreescribir los datos de la copia diferencial más antigua, de forma que siempre tengamos acceso a las copias diferenciales de los últimos 7 días.

**Copia completa (semanal y mensual):**

Se guardarán copias completas semanalmente y cada cuatro semanas se borrarán las 3 copias completas más nuevas, de esta forma siempre se quedará guardada una copia completa al mes (aunque habrá meses que tengan dos copias ya que lo meses no tienen cuatro semanas justas).

**Esquema de directorios:**

```txt
/backup/completa/latest
/backup/completa/old

/backup/diff/lun
/backup/diff/mar
/backup/diff/mie
/backup/diff/jue
/backup/diff/vie
/backup/diff/sab
/backup/diff/dom
```

### 5. Scripts

Los script los he creado en el directorio /backup. Se realizarán 3 scripts que tendrán el siguiente formato:

**5.1. SCRIPT COPIA COMPLETA**

```bash
nano completa.sh
sudo chmod 755 completa.sh
```

```bash
#!/bin/bash

# Ruta de destino (directorio donde se guardarán las copias de seguridad)
DEST_DIR="/backup/completa"

# Obtener la fecha actual
DATE=$(date +"%Y-%m-%d")

# Si existe algo en la carpeta comprimir la información guardada en old y borrar lo de latest
if [ -d "$DEST_DIR/latest/" ]; then
    tar -zcf /backup/completa/old/$DATE.tar /backup/completa/latest/*
    rm -r /backup/completa/latest/*
    #mv "$DEST_DIR/latest/*" "$DEST_DIR/old/"
fi

# Realizar la copia completa en /backup/completa/latest/ en un directorio con la fecha del backup
rsync -aAX --files-from="/backup/a_respaldar.txt" --quiet / "$DEST_DIR/latest/$DATE"
```

- --archive, -a       archive mode is -rlptgoD (no -A,-X,-U,-N,-H)
  -  -r              recurse into directories
  -  -l              copy symlinks as symlinks
  -  -p              preserve permissions
  -  -t              preserve modification times
  -  -g              preserve group
  -  -o              preserve owner (super-user only)
  -  -D              The -D option is equivalent to --devices --specials.
      - --devices             This option causes rsync to transfer character and block device  files  to  the  remote system  to recreate these devices.  This option has no effect if the receiving rsync is not run as the super-user (see also the --super and --fake-super options).
      - --specials             This option causes rsync to transfer special files such as named sockets and fifos.
- --acls, -A          preserve ACLs (implies --perms)
- --xattrs, -X        preserve extended attributes
- --files-from='a_respaldar.txt': lee la lista de archivos y directorios a respaldar desde el archivo a_respaldar.txt

**5.2. SCRIPT PLANIFICACIÓN**

```bash
nano plan.sh
sudo chmod 755 plan.sh
```

```bash
#!/bin/bash
# Borrar 3 últimas carpetas de /backup/completa/old
cd /backup/completa/old
ls -t | head -3 | xargs rm -r
```

**5.3. SCRIPT COPIA DIFERENCIAL (sobre la ultima copia completa)**

```bash
nano diff.sh
sudo chmod 755 diff.sh
```

```bash
#!/bin/bash

# Ruta de destino (directorio donde se guardarán las copias de seguridad)
DEST_DIR="/backup/diff"

# Obtener el día de la semana (lun, mar, mie, jue, vie, sab, dom)
DAY=$(date +%a)

# Si existe una copia diferencial anterior para el día actual, eliminarla
if [ -d "$DEST_DIR/$DAY" ]; then
    rm -rf "$DEST_DIR/$DAY"
fi

# Realizar la copia diferencial basada en la última copia completa en una carpeta con el día de la semana
rsync -aAXS --delete --files-from="/backup/a_respaldar.txt" --quiet --link-dest="/backup/completa/latest/" / "$DEST_DIR/$DAY/"
```

- -S: para optimizar el almacenamiento de archivos que contienen muchos caracteres nulos o ceros consecutivos. En lugar de asignar espacio de almacenamiento para cada carácter nulo individual, los bloques dispersos permiten que se almacene solo la información necesaria para recrear los caracteres nulos consecutivos. La opción -S viene bien cuando se tienen archivos dispersos, como discos virtuales, imágenes Docker y similares.

- --delete: Elimina los archivos en el directorio de destino que no existen en el directorio de origen. Esto garantiza que el directorio de destino refleje exactamente el contenido del directorio de origen.

- --quiet: Ejecuta rsync en modo silencioso, sin mostrar información detallada sobre los archivos y directorios que se están sincronizando.

- --link-dest=: Establece el directorio de destino como una copia enlazada de un directorio de referencia, que en este caso es el directorio más reciente (latest) dentro de la carpeta de respaldo completa (completa). Los archivos idénticos en el directorio de origen se enlazarán en lugar de copiarse, lo que permite ahorrar espacio en el almacenamiento.

- /: Especifica el directorio de origen desde el cual se sincronizarán los archivos y directorios. En este caso, se está utilizando la raíz del sistema de archivos.

- $DEST_DIR/$DAY/: Indica el directorio de destino donde se guardarán los archivos y directorios sincronizados. El nombre del directorio de destino se construye utilizando el valor de la variable $DAY.

### 6. Automatizar con crontab

Editar nuestro archivo crontab y hacer correr cada script en su tiempo establecido:

```bash
crontab -e
```

```conf
#Ejecutar copia completa semanalmente
@weekly root /backup/completa.sh

#Ejecutar planificación cada 4 semanas los lunes a las 23:00
0 23 * * 1  [ $(expr $(date +\%s) / 86400 / 7 \% 4) -eq 0 ] && /backup/plan.sh

#Ejecutar copia diferencial diariamente
@daily root /backup/diff.sh
```

*La parte "[ $(($(date +%s) / (7 * 24 * 3600))) % 4 -eq 0 ]" es una verificación adicional dentro del comando cron. Divide el número de segundos transcurridos desde el 1 de enero de 1970 por el número de segundos en 4 semanas (28 días), y luego verifica si el resultado es un múltiplo de 4. Esto asegura que el script se ejecute cada 4 semanas. Pero no lo he probado para saber si es verdad que se ejecuta cada cuatro semanas.

### 7. Desde Bravo, Charlie y Delta

Se crearía la configuración cron y los scripts en cada máquina, pero estos últimos serían un poco diferentes ya que habría que conectarse al servidor Alfa. Habría que crear en /backup un directorio por cada servidor y realizar los mismos pasos de los script pero conectándose desde cada servidor a Alfa.

Ejemplo de rsync usando ssh:

```bash
#!/bin/bash
rsync -a --delete --quiet -e ssh /folder/to/backup remoteuser@remotehost:/location/of/backup
```

### 8. Restaurar cambios

Si necesitáramos revertir a un respaldo previo, todo lo que necesitamos hacer es copiar de vuelta los archivos del respaldo sobre los actualizados y borrar cualquier archivo que fuera creado después de que realizamos el respaldo que estamos restaurando.

### 9. Webs de interés

- https://wiki.archlinux.org/title/Rsync_(Espa%C3%B1ol)
- https://web.archive.org/web/20210516191405/https://www.vicente-navarro.com/blog/2008/01/13/backups-con-rsync/
- https://www.comoinstalarlinux.com/rsync-backup/
- https://jumpcloud.com/blog/how-to-backup-linux-system-rsync
- https://www.ionos.es/digitalguide/servidores/herramientas/como-crear-copias-de-seguridad-del-servidor-con-rsync/

**Otras herramientas/aplicaciones interesantes para crear copias de seguridad:**

- https://www.neoguias.com/mejores-aplicaciones-backup-linux/