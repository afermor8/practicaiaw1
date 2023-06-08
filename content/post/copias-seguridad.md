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


### Preparación


La aplicación que he decidido utilizar para realizar el proceso de copias de seguridad es ***rsync***.

Rsync es ........


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

Intalo el paquete xfsprogs:

```bash
sudo apt-get install xfsprogs
```

Formateamos el volumen usando el sistema de ficheros *xfs*.

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

### Información a guardar

Voy a guardar el contenido de /home, /etc, /var, /opt, /usr/share


Críticos:

alfa	Reglas iptables
bravo	Web (/var/www, /etc/httpd)
charlie	Servidor dns (/var/chache/bind, /etc/bind, ...)
delta	Servidor correo (/etc/postfix)



### Información a excluir


### Copias de seguriadad (diaria-diferencial, semanal-completa, mensual-completa)

He decidido que la copia diaria será de tipo diferencial.
Requiere más espacio de almacenamiento y más tiempo de respaldo, pero la restauración es más rápida, ya que solo se necesita acceder a la copia completa y la última copia diferencial. Es adecuada si deseas minimizar el tiempo de restauración y no tienes restricciones de espacio de almacenamiento.

Se guardarán las copias de todos los días de la semana, y cuando termine la semana se empezará a sobreescribir los datos de la copia diferencial más antigua, de forma que siempre tengamos acceso a las copias diferenciales de los últimos 7 días.



Se guardarán copias completas semanalmente.

Cada cuatro semanas se borrarán las 3 copias completas más nuevas, de esta forma siempre se quedará guardada una copia completa al mes (unque habrá meses que tengan dos copias ya que lo meses no tienen cuatro semanas justas). 


!!!MAL, en l futuro deberían haber decenas de copias completas a lo largo de los años
Si 4 copias o mas entonces
  borrar las 3 más nuevas
  crear copia completa




### Scripts

Directorios:

/backup/completa/latest
/backup/completa/old

/backup/diff/lun
/backup/diff/mar
/backup/diff/mie
/backup/diff/jue
/backup/diff/vie
/backup/diff/sab
/backup/diff/dom



3 SCRIPTS:


1. Copia completa

    Si existe carpeta
      copia a /backup/completa/old
      borrar carpeta de /backup/completa/latest
    tminar if
    Crear copia completa en /backup/completa/latest
      Crear carpeta con la fecha de la copia donde estará todo el contenido

*Crontab: ejecutar cada 7 días




2. Planificación

    Borrar 3 últimas carpetas de /backup/completa/old

*Crontab: ejecutar cada 4 semanas




3. Copia diferencial (sobre la ultima copia completa)

    Si exite carpeta con ese día de la semana entonces
      eliminar carpeta
    terminar if
    Crear copia diferencial
      Añadir carpeta con día de la semana
      Nos basamos en la última copia completa
      
*Crontab: ejecutar diariamente





1. SCRIPT COMPLETA

```bash
#!/bin/bash

# Ruta de origen (directorio que deseas respaldar)
SOURCE_DIR="/ruta/de/origen"

# Ruta de destino (directorio donde se guardarán las copias de seguridad)
DEST_DIR="/backup/completa"

# Obtener la fecha actual
DATE=$(date +"%Y-%m-%d")

# Mover la copia completa anterior a la carpeta de copias antiguas
if [ -d "$DEST_DIR/latest/" ]; then
    mv "$DEST_DIR/latest/*" "$DEST_DIR/old/"
fi

# Realizar la copia completa en la carpeta latest
rsync -a --delete --quiet "$SOURCE_DIR" "$DEST_DIR/latest/$DATE"
```




2. SCRIPT PLAN

cd /backup/completa/old
ls -t | head -3 | xargs rm -r



3. SCRIPT DIFF

```bash
#!/bin/bash

# Ruta de origen (directorio que deseas respaldar)
SOURCE_DIR="/ruta/de/origen"

# Ruta de destino (directorio donde se guardarán las copias de seguridad)
DEST_DIR="/backup/diff"

# Obtener el día de la semana (lun, mar, mie, jue, vie, sab, dom)
DAY=$(date +%a)

# Si existe una copia diferencial anterior para el día actual, eliminarla
if [ -d "$DEST_DIR/$DAY" ]; then
    rm -rf "$DEST_DIR/$DAY"
fi

# Realizar la copia diferencial basada en la última copia completa
rsync -a --delete --quiet --link-dest="$DEST_DIR/../completa/latest/*/" "$SOURCE_DIR" "$DEST_DIR/$DAY/"
```







### Automatizar con crontab











### Comandos

rsync -ab origen/ destino/


Esto agregará el sufijo .viejo a los archivos de respaldo, es decir, archivo01.txt será copiado en archivo01.txt.viejo, y entonces archivo01.txt sería actualizado con la nueva información. Podemos agregar la fecha de el respaldo al sufijo utilizando el comando date junto con el parámetro sufijo

rsync -ab --suffix=.viejo origen/ destino/

Ó

rsync -ab --suffix=_`date +%F` origen/ destino/



--backup-dir, esto usa la carpeta en el directorio de destino, a menos que se especifique otra cosa. Si el directorio especificado no existe, rsync lo creará. Por ejemplo:

rsync -ab --backup-dir=respaldo origen/ destino/





## Crontab

Si fuéramos a hacer un respaldo completo diario, podríamos utilizar el siguiente crontab:

@daily rsync -au --delete origen/ destino/


Para respaldo incremental:

@daily rsync -ab --backup-dir=viejo_`date +%F` --delete --exclude=viejo_* origen/ destino/




## Script

creo una entrada en ~/.ssh/config como se ve en Definiendo servidores SSH, y la nombro respaldo_remoto. La entrada se ve algo así:

Host respaldo_remoto
HostName nombre_servidor.dreamhost.com
User nombre_usuario


me conecto por medio de SFTP, creo la carpeta .ssh, y copio mi id_rsa.pub recien generada o ya existente en .ssh/authorized_keys en el servidor remoto, y ya que estoy aquí, creo antes de salir las carpetas Respaldos y mi_trabajo que voy a utilizar:

sftp respaldo_remoto
(Escribe la contraseña. Después de entrar estamos en un shell de SFTP y usamos estos comandos)
mkdir .ssh
cd .ssh/
put -p /home/juan/.ssh/id_rsa.pub authorized_keys
cd ..
mkdir Respaldos
mkdir Respaldos/mi_trabajo
exit


nano ~/Scripts/a_respaldar.txt

.vimrc
.vim/
Proyectos/
Documentos/Trabajo/
Scripts/


touch ~/Scripts/respaldo_auto.sh
chmod u+x ~/Scripts/respaldo_auto.sh

```
#!/bin/sh

SUFIJO=$(date +%j)

# Borrar el directorio en el servidor remoto vía SSH
# ssh respaldo_remoto 'ls Respaldos/mi_trabajo/respaldo_'$SUFIJO' && rm -r Respaldos/mi_trabajo/respaldo_'$SUFIJO

# Borrar el directorio en el servidor remoto vía SFTP
sftp -b /dev/fd/0 respaldo_remoto <<EOF
cd Respaldos/mi_trabajo
rmdir respaldo_$SUFIJO
exit
EOF

# Actualizar la información, creando una carpeta de respaldo e ignorando el
# el resto de los directorios de respaldo
rsync -ab --recursive --files-from='a_respaldar.txt' --backup-dir=respaldo_$SUFIJO --delete --filter='protect respaldo_*' /home/juan/ respaldo_remoto:Respaldos/mi_trabajo/
```


Probar script:

sh ~/Scripts/respaldo_auto.sh



editar nuestro archivo crontab y hacerlo correr el script diariamente:

crontab -e

@daily /home/juan/Scripts/respaldo_auto.sh







## Realizar copia de seguridad automatizada con SSH
Si realiza una copia de seguridad en un servidor remoto utilizando SSH, utilice este script en su lugar:

#!/bin/bash
rsync -a --delete --quiet -e ssh /folder/to/backup remoteuser@remotehost:/location/of/backup



## Realizar copia de seguridad diferencial por semana
Esta es una opción útil de rsync, que consiste en realizar una copia de seguridad completa (en cada ejecución) y mantener una copia de seguridad diferencial solo de los archivos modificados en un directorio separado para cada día de la semana.

Primero, cree un script que contenga las opciones de órdenes apropiadas:

/etc/cron.daily/backup
#!/bin/bash

DAY=$(date +%A)

if [ -e /location/to/backup/incr/$DAY ] ; then
  rm -fr /location/to/backup/incr/$DAY
fi

rsync -a --delete --quiet --inplace --backup --backup-dir=/location/to/backup/incr/$DAY /folder/to/backup/ /location/to/backup/full/


--inplace
implica la actualización --partial de los archivos de destino en el lugar


## Realizar copia de seguridad completa del sistema
Esta sección trata sobre el uso de rsync para transferir una copia de todo el árbol /, excluyendo algunas carpetas seleccionadas. Se considera que este enfoque es mejor que clonar un disco con dd dado que ello utilizar un tamaño diferente, una tabla de partición y un sistema de archivos, y mejor que copiar con cp -a también, porque permite un mayor control sobre los permisos de archivos, atributos, Listas de control de acceso y Atributos extendidos.

rsync funcionará incluso mientras el sistema se está ejecutando, pero los archivos modificados durante la transferencia pueden o no transferirse, lo que puede causar un comportamiento inesperado de algunos programas que usan los archivos transferidos.

Este enfoque funciona bien para migrar una instalación existente a un nuevo disco duro o SSD.

Ejecute la siguiente orden como root para asegurarse de que rsync pueda acceder a todos los archivos del sistema y preservar su propiedad:

rsync -aAXv --exclude={"/dev/*","/proc/*","/sys/*","/tmp/*","/run/*","/mnt/*","/media/*","/lost+found"} / /path/to/backup/folder


Al utilizar el conjunto de opciones -aAX, los archivos se transfieren en modo de comprimido, lo que garantiza que los enlaces simbólicos, los dispositivos, los permisos, las propiedades, los tiempos de modificación, los ACL y los atributos extendidos se conserven, suponiendo que el sistema de archivos de destino dé soporte a esta función.



!!!!!!!!!!!!!!!!!Nota:
Considere excluir subdirectorios sin importancia como /home/*/.thumbnails/*, /home/*/.cache/mozilla/*, /home/*/.cache/chromium/* y /home/*/.local/share/Trash/*, dependiendo del software instalado en el sistema. Si GVFS está instalado, /home/*/.gvfs debe excluirse para evitar errores de rsync.


agregar la opción --delete de rsync si está ejecutando esto varias veces en la misma carpeta de respaldo. En este caso, asegúrese de que la ruta de origen no termine con /*, o esta opción solo tendrá efecto en los archivos dentro de los subdirectorios del directorio de origen, pero no tendrá ningún efecto en los archivos que residen directamente dentro del directorio de origen.


Si utiliza archivos dispersos, como discos virtuales, imágenes Docker y similares, debe agregar la opción -S.


Si planea hacer una copia de seguridad de su sistema en otro lugar que no sea /mnt o /media, no olvide agregarlo a la lista de patrones de exclusión para evitar un bucle infinito.
----- 
En mi caso tengo que excluir el directorio /backup que es donde he montado el volumen donde se guardarán las copias de seguridad. Si no lo excluyo se creará un bucle intentando copiar lo de este fichero que no para de crecer con su propio contenido.



### Restaurar cambios

Si necesitáramos revertir a un respaldo previo, todo lo que necesitamos hacer es copiar de vuelta los archivos de el respaldo sobre los actualizados y borrar cualquier archivo que fuera creado después de que realizamos el respaldo que estamos restaurando.









https://wiki.archlinux.org/title/Rsync_(Espa%C3%B1ol)
https://web.archive.org/web/20210516191405/https://www.vicente-navarro.com/blog/2008/01/13/backups-con-rsync/
https://www.comoinstalarlinux.com/rsync-backup/
https://jumpcloud.com/blog/how-to-backup-linux-system-rsync

https://www.ionos.es/digitalguide/servidores/herramientas/como-crear-copias-de-seguridad-del-servidor-con-rsync/
