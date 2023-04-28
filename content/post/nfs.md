---
title: "Montaje NFS mediante systemd"
date: 2023-04-28T11:59:22+02:00
draft: true
tags: ["ASO"]
bigimg: [{src: "/img/triangle.jpg", desc: "Triangle"}, {src: "/img/sphere.jpg", desc: "Sphere"}, {src: "/img/hexagon.jpg", desc: "Hexagon"}]
---

## En una instancia del cloud, basada en la distribución de tu elección, anexa un volumen de 2GB. En dicha instancia deberás configurar el servicio nfs de exportación y en el volumen un punto de montaje de la exportación mediante systemd.

### 1. Preparar escenario

#### 1.1 Pasos previos

En mi máquina host llamada tars voy a instalar openstackclient.

```bash
sudo apt install python3-venv

python3 -m venv ~/venv/openstackclient

source ~/venv/openstackclient/bin/activate

pip3 install openstackclient
```

Vamos a Openstack para conseguir el fichero Openstack RC.

![](/img/nfs/1.png)

Uso el fichero para entrar con mis credenciales.

```bash
source Descargas/Proyecto\ de\ arantxa.fernandez-openrc.sh
```

Y ya podremos hacer uso de openstackclient.

```bash
(openstackclient) arantxa@tars:~$ openstack server list
+--------------------------------------+---------+--------+-----------------------------------------------------------------------------------+--------------------------+-----------+
| ID                                   | Name    | Status | Networks                                                                          | Image                    | Flavor    |
+--------------------------------------+---------+--------+-----------------------------------------------------------------------------------+--------------------------+-----------+
| d4ad21c8-d607-4c4e-9dc5-cc89ea5922b1 | maquina | ACTIVE | red de arantxa.fernandez=10.0.0.50, 172.22.200.144                                | Debian 11 Bullseye       | m1.mini   |
| f4b90904-5aa8-488d-b58c-6a6fdcdabdbc | aislada | ACTIVE | privada_aislada=192.168.22.3; red de arantxa.fernandez=10.0.0.190, 172.22.201.239 | N/A (booted from volume) | m1.normal |
+--------------------------------------+---------+--------+-----------------------------------------------------------------------------------+--------------------------+-----------+
```

#### 1.2 Creación servidor NFS

Creo la máquina que hará de servidor NFS.

```bash
openstack server create --flavor m1.mini --image "Debian 11 Bullseye" --security-group default --key-name Mi_Clave --network "red de arantxa.fernandez" servidor_nfs
```

Me aparece lo siguiente:

```bash
+-----------------------------+------------------------------------------------------------------+
| Field                       | Value                                                            |
+-----------------------------+------------------------------------------------------------------+
| OS-DCF:diskConfig           | MANUAL                                                           |
| OS-EXT-AZ:availability_zone |                                                                  |
| OS-EXT-STS:power_state      | NOSTATE                                                          |
| OS-EXT-STS:task_state       | scheduling                                                       |
| OS-EXT-STS:vm_state         | building                                                         |
| OS-SRV-USG:launched_at      | None                                                             |
| OS-SRV-USG:terminated_at    | None                                                             |
| accessIPv4                  |                                                                  |
| accessIPv6                  |                                                                  |
| addresses                   |                                                                  |
| adminPass                   | yvAGTmki8FWw                                                     |
| config_drive                |                                                                  |
| created                     | 2023-04-28T10:47:04Z                                             |
| flavor                      | m1.mini (3)                                                      |
| hostId                      |                                                                  |
| id                          | b85d66d8-db4f-44ac-a771-378fee658833                             |
| image                       | Debian 11 Bullseye (6d992898-7e4f-44b9-a681-6dcf32d24a1f)        |
| key_name                    | Mi_Clave                                                         |
| name                        | servidor_nfs                                                     |
| progress                    | 0                                                                |
| project_id                  | e928f4126bb343ec8048871da80c2a39                                 |
| properties                  |                                                                  |
| security_groups             | name='0ad08d7e-fb7f-414b-917c-8152a0910246'                      |
| status                      | BUILD                                                            |
| updated                     | 2023-04-28T10:47:04Z                                             |
| user_id                     | 276b52c5951960d5b90f6d1d4d717e43ecb9c157971ba4ddd15f9728ae476f9a |
| volumes_attached            |                                                                  |
+-----------------------------+------------------------------------------------------------------+
```

A continuación creo una IP flotante.

```bash
openstack floating ip create ext-net
```

```bash
+---------------------+--------------------------------------+
| Field               | Value                                |
+---------------------+--------------------------------------+
| created_at          | 2023-04-28T10:49:26Z                 |
| description         |                                      |
| dns_domain          | None                                 |
| dns_name            | None                                 |
| fixed_ip_address    | None                                 |
| floating_ip_address | 172.22.200.58                        |
| floating_network_id | 2ebd4d15-00e3-44c6-a9a7-aeebef5f6540 |
| id                  | a968d5af-62bb-4b81-88e9-1577173aa561 |
| name                | 172.22.200.58                        |
| port_details        | None                                 |
| port_id             | None                                 |
| project_id          | e928f4126bb343ec8048871da80c2a39     |
| qos_policy_id       | None                                 |
| revision_number     | 0                                    |
| router_id           | None                                 |
| status              | DOWN                                 |
| subnet_id           | None                                 |
| tags                | []                                   |
| updated_at          | 2023-04-28T10:49:26Z                 |
+---------------------+--------------------------------------+
```

Se la asigno a la máquina creada para poder usarla.

```bash
openstack server add floating ip servidor_nfs 172.22.200.58
```

Creo el volumen de 2 GB.

```bash
openstack volume create --size 2 --description "Volumen para servidor_nfs" --availability-zone "nova" --bootable volumen_nfs
```

```bash
+---------------------+------------------------------------------------------------------+
| Field               | Value                                                            |
+---------------------+------------------------------------------------------------------+
| attachments         | []                                                               |
| availability_zone   | nova                                                             |
| bootable            | false                                                            |
| consistencygroup_id | None                                                             |
| created_at          | 2023-04-28T10:54:12.092015                                       |
| description         | Volumen para servidor_nfs                                        |
| encrypted           | False                                                            |
| id                  | 5e08d606-d74a-4564-9a0b-f88fbcb671b1                             |
| multiattach         | False                                                            |
| name                | volumen_nfs                                                      |
| properties          |                                                                  |
| replication_status  | None                                                             |
| size                | 2                                                                |
| snapshot_id         | None                                                             |
| source_volid        | None                                                             |
| status              | creating                                                         |
| type                | __DEFAULT__                                                      |
| updated_at          | None                                                             |
| user_id             | 276b52c5951960d5b90f6d1d4d717e43ecb9c157971ba4ddd15f9728ae476f9a |
+---------------------+------------------------------------------------------------------+
```

Asigno el volumen a la máquina.

```bash
openstack server add volume servidor_nfs volumen_nfs
```

```bash
+-----------------------+--------------------------------------+
| Field                 | Value                                |
+-----------------------+--------------------------------------+
| ID                    | 5e08d606-d74a-4564-9a0b-f88fbcb671b1 |
| Server ID             | b85d66d8-db4f-44ac-a771-378fee658833 |
| Volume ID             | 5e08d606-d74a-4564-9a0b-f88fbcb671b1 |
| Device                | /dev/vdb                             |
| Tag                   | None                                 |
| Delete On Termination | False                                |
+-----------------------+--------------------------------------+
```

Accedo a la máquina para comprobar que se ha asignado correctamente el volumen.

```bash
ssh debian@172.22.200.58
```

```bash
debian@servidor-nfs:~$ lsblk

NAME    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
vda     254:0    0   10G  0 disk 
├─vda1  254:1    0  9.9G  0 part /
├─vda14 254:14   0    3M  0 part 
└─vda15 254:15   0  124M  0 part /boot/efi
vdb     254:16   0    2G  0 disk
```

Formateo el volumen añadido.

```bash
debian@servidor-nfs:~$ sudo mkfs -t ext4 /dev/vdb

mke2fs 1.46.2 (28-Feb-2021)
Discarding device blocks: done                            
Creating filesystem with 524288 4k blocks and 131072 inodes
Filesystem UUID: 08e0df64-1306-4add-9318-40e2dc8e51fd
Superblock backups stored on blocks: 
	32768, 98304, 163840, 229376, 294912

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done 
```

#### 1.3 Creación máquina cliente

Creo la máquina cliente a la que he llamado cliente_nfs de la siguiente forma:

```bash
openstack server create --flavor m1.mini --image "Debian 11 Bullseye" --security-group default --key-name Mi_Clave --network "red de arantxa.fernandez" cliente_nfs
```

```bash
+-----------------------------+------------------------------------------------------------------+
| Field                       | Value                                                            |
+-----------------------------+------------------------------------------------------------------+
| OS-DCF:diskConfig           | MANUAL                                                           |
| OS-EXT-AZ:availability_zone |                                                                  |
| OS-EXT-STS:power_state      | NOSTATE                                                          |
| OS-EXT-STS:task_state       | scheduling                                                       |
| OS-EXT-STS:vm_state         | building                                                         |
| OS-SRV-USG:launched_at      | None                                                             |
| OS-SRV-USG:terminated_at    | None                                                             |
| accessIPv4                  |                                                                  |
| accessIPv6                  |                                                                  |
| addresses                   |                                                                  |
| adminPass                   | Tp5DhgNB74VA                                                     |
| config_drive                |                                                                  |
| created                     | 2023-04-28T11:00:06Z                                             |
| flavor                      | m1.mini (3)                                                      |
| hostId                      |                                                                  |
| id                          | c4d27ef2-0c70-4fc7-a07f-8af79f985228                             |
| image                       | Debian 11 Bullseye (6d992898-7e4f-44b9-a681-6dcf32d24a1f)        |
| key_name                    | Mi_Clave                                                         |
| name                        | cliente_nfs                                                      |
| progress                    | 0                                                                |
| project_id                  | e928f4126bb343ec8048871da80c2a39                                 |
| properties                  |                                                                  |
| security_groups             | name='0ad08d7e-fb7f-414b-917c-8152a0910246'                      |
| status                      | BUILD                                                            |
| updated                     | 2023-04-28T11:00:06Z                                             |
| user_id                     | 276b52c5951960d5b90f6d1d4d717e43ecb9c157971ba4ddd15f9728ae476f9a |
| volumes_attached            |                                                                  |
+-----------------------------+------------------------------------------------------------------+
```

Como en la máquina anterior creo una ip flotante y se la asigno:

```bash
openstack floating ip create ext-net
```

```bash
+---------------------+--------------------------------------+
| Field               | Value                                |
+---------------------+--------------------------------------+
| created_at          | 2023-04-28T11:02:53Z                 |
| description         |                                      |
| dns_domain          | None                                 |
| dns_name            | None                                 |
| fixed_ip_address    | None                                 |
| floating_ip_address | 172.22.201.158                       |
| floating_network_id | 2ebd4d15-00e3-44c6-a9a7-aeebef5f6540 |
| id                  | 7ca627cc-87e8-4330-9c89-12c6ac30d51b |
| name                | 172.22.201.158                       |
| port_details        | None                                 |
| port_id             | None                                 |
| project_id          | e928f4126bb343ec8048871da80c2a39     |
| qos_policy_id       | None                                 |
| revision_number     | 0                                    |
| router_id           | None                                 |
| status              | DOWN                                 |
| subnet_id           | None                                 |
| tags                | []                                   |
| updated_at          | 2023-04-28T11:02:53Z                 |
+---------------------+--------------------------------------+
```

```bash
openstack server add floating ip cliente_nfs 172.22.201.158 
```

Ya puedo acceder a la máquina cliente.

```bash
ssh debian@172.22.201.158

debian@cliente-nfs:~$ lsblk
NAME    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
vda     254:0    0   10G  0 disk 
├─vda1  254:1    0  9.9G  0 part /
├─vda14 254:14   0    3M  0 part 
└─vda15 254:15   0  124M  0 part /boot/efi
```

### 2. Configuración de servidor_nfs

#### 2.1 Unidad de montaje

Creo la unidad de montaje de la siguiente manera:

```bash
sudo nano /etc/systemd/system/mnt.mount
```

Le añado:

```conf
[Unit]
Description=Montaje de disco NFS con volumen vdb 2GB

[Mount]
What=/dev/vdb
Where=/mnt
Type=ext4
Options=defaults

[Install]
WantedBy=multi-user.target
```

Activo el servicio:

```bash
sudo systemctl start mnt.mount
sudo systemctl enable mnt.mount
```

Comprobar estado de la unidad de montaje:

```bash
debian@servidor-nfs:~$ sudo systemctl status mnt.mount
● mnt.mount - Montaje de disco NFS con volumen vdb 2GB
     Loaded: loaded (/proc/self/mountinfo; enabled; vendor preset: enabled)
     Active: active (mounted) since Fri 2023-04-28 11:35:43 UTC; 10min ago
      Where: /mnt
       What: /dev/vdb
      Tasks: 0 (limit: 527)
     Memory: 28.0K
        CPU: 4ms
     CGroup: /system.slice/mnt.mount

Apr 28 11:35:43 servidor-nfs systemd[1]: Mounting Montaje de disco NFS con volumen vdb 2GB...
Apr 28 11:35:43 servidor-nfs systemd[1]: Mounted Montaje de disco NFS con volumen vdb 2GB.
```

Comprobar el volumen montado:

```bash
debian@servidor-nfs:~$ lsblk -f
NAME    FSTYPE FSVER LABEL UUID                                 FSAVAIL FSUSE% MOUNTPOINT
vda                                                                            
├─vda1  ext4   1.0         ce1282a1-ced7-40a2-8b01-eed78dc14d62    8.4G     9% /
├─vda14                                                                        
└─vda15 vfat   FAT16       4A0A-DB16                             117.8M     5% /boot/efi
vdb     ext4   1.0         08e0df64-1306-4add-9318-40e2dc8e51fd    1.8G     0% /mnt
```

#### 2.2 Servidor NFS

Instalar los paquetes necesarios para usar NFS.

```bash
sudo apt update
sudo apt install nfs-kernel-server nfs-common
```

Para que se puedan conectar los clientes NFS añadimos a /etc/exports lo siguiente:

```bash
sudo nano /etc/exports

/mnt 10.0.0.0/24(rw,sync,no_subtree_check,no_root_squash)
```

Reiniciar el servicio NFS.

```bash
sudo systemctl restart nfs-kernel-server
```

Actualizo la lista de directorios compartidos:

```bash
sudo exportfs
```

### 3. Configuración de cliente_nfs

#### 3.1 Cliente NFS

Vamos a la máquina cliente_nfs e instalamos el paquete necesario para conectarme como cliente NFS.

```bash
sudo apt update
sudo apt install nfs-common
```

#### 3.2 Unidad de montaje

Creo la unidad de montaje.

```bash
sudo nano /etc/systemd/system/mnt.mount
```

```conf
[Unit]
Description=Montaje de disco NFS compartido con servidor_nfs (vdb)

[Mount]
What=10.0.0.58:/mnt     --ipservidor:directoriocompartido
Where=/mnt
Options=defaults
Type=nfs

[Install]
WantedBy=multi-user.target
```

Activo el servicio.

```bash
sudo systemctl start mnt.mount
sudo systemctl enable mnt.mount
```

#### 3.3 Comprobaciones

Comprobar unidad de montaje funcionando.

```bash
debian@cliente-nfs:~$ sudo systemctl status mnt.mount

● mnt.mount - Montaje de disco NFS compartido con servidor_nfs (vdb)
     Loaded: loaded (/proc/self/mountinfo; enabled; vendor preset: enabled)
     Active: active (mounted) since Fri 2023-04-28 12:08:23 UTC; 18s ago
      Where: /mnt
       What: 10.0.0.58:/mnt
      Tasks: 0 (limit: 527)
     Memory: 76.0K
        CPU: 7ms
     CGroup: /system.slice/mnt.mount

Apr 28 12:08:23 cliente-nfs systemd[1]: Mounting Montaje de disco NFS compartido con servidor_nfs (vdb)>
Apr 28 12:08:23 cliente-nfs systemd[1]: Mounted Montaje de disco NFS compartido con servidor_nfs (vdb).
```

Comprobar el directorio remoto montado.

```bash
debian@cliente-nfs:~$ df -h

Filesystem      Size  Used Avail Use% Mounted on
udev            220M     0  220M   0% /dev
tmpfs            48M  476K   47M   1% /run
/dev/vda1       9.7G  1.1G  8.3G  12% /
tmpfs           237M     0  237M   0% /dev/shm
tmpfs           5.0M     0  5.0M   0% /run/lock
/dev/vda15      124M  5.9M  118M   5% /boot/efi
tmpfs            48M     0   48M   0% /run/user/1000
10.0.0.58:/mnt  2.0G     0  1.8G   0% /mnt
```

Creo un fichero de prueba en el directorio compartido.

```bash
sudo nano /mnt/pruebanfs.txt
```

Añado:

```text
hola esto es una prueba para la practica de nfs mediante systemd
```

Voy a servidor_nfs y compruebo el contenido del directorio.

```bash
debian@servidor-nfs:~$ ls -l /mnt/
total 20
drwx------ 2 root root 16384 Apr 28 11:26 lost+found
-rw-r--r-- 1 root root    65 Apr 28 12:11 pruebanfs.txt
```

```bash
debian@servidor-nfs:~$ cat /mnt/pruebanfs.txt
hola esto es una prueba para la practica de nfs mediante systemd
```

**Captura de prueba:**

![](/img/nfs/2.png)
