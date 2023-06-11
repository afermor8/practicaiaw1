---
title: "Instalación y configuración inicial de OpenLDAP"
date: 2023-06-10T19:21:47+02:00
draft: true
tags: ["ASO"]
bigimg: [{src: "/img/triangle.jpg", desc: "Triangle"}, {src: "/img/sphere.jpg", desc: "Sphere"}, {src: "/img/hexagon.jpg", desc: "Hexagon"}]
---


#### Realiza la instalación y configuración básica de OpenLDAP en alfa,utilizando como base el nombre DNS asignado. Deberás crear un usuario llamado prueba y configurar una máquina cliente basada en Debian y Rocky para que pueda validarse en servidor ldap configurado anteriormente con el usuario prueba.

### 0. Preparación del escenario

Voy a instalar OpenLDAP en la máquina alfa de mi escenario de Openstack. Alfa tiene el siguiente FQDN:

```bash
arantxa@alfa:~$ hostname -f
alfa.arantxa.gonzalonazareno.org
```

*NOTA 1: No tenemos que crear el usuario prueba con `adduser` ya que lo creará OpenLDAP.

*NOTA 2: Como clientes del servidor LDAP tendremos a la máquina bravo (Rocky Linux) y la máquina delta (Ubuntu).

---------------------------------------------------

### 1. Instalación OpenLDAP

Instalar los paquetes necesarios:

```bash
sudo apt install slapd ldap-utils
```

Durante la instalación indicamos el ususario y contraseña que queramos usar para la administración. Como este es un caso de prueba mi usuario y contraseña será **admin**.

Cuando termine la instalación comprobamos que el servicio está activo y funcionando correctamente.

```bash
sudo systemctl status slapd
```

![status](/img/ldap/2.png)

Comprobamos el puerto que utiliza LDAP.

```bash
sudo netstat -putanl | egrep slapd
```

![port](/img/ldap/3.png)

---------------------------------------------------

### 2. Configuración de OpenLDAP (alfa)

Antes de empezar hay que tener claro algunos conceptos.

La estructura de OpenLDAP se basa en el modelo de datos de directorio llamado Árbol de Entradas (Entry Tree). El árbol de entradas se organiza jerárquicamente utilizando el esquema de datos definido en el servidor LDAP. Una entrada es un objeto individual en el directorio LDAP y se representa mediante un Distinguished Name (DN) único que identifica de manera exclusiva la posición de la entrada en el árbol de entradas.

- Cada entrada se compone de uno o más atributos. Estos son unidades de información que describen una característica específica de una entrada. Cada atributo tiene un nombre y uno o más valores asociados.

- Una clase de objeto (Object Class) define el conjunto de atributos que puede tener una entrada. Proporciona una plantilla para crear nuevas entradas y establece las reglas y restricciones para los atributos.

- El esquema (Schema) define la estructura y las reglas para los datos almacenados en el directorio. Define las clases de objetos, los atributos y las relaciones entre ellos.

- El directorio base (Base DN) es el punto de entrada principal en el árbol de entradas. Todas las búsquedas y operaciones se realizan en relación con el directorio base.

Ejemplo de DN: "cn=user, ou=groups, dc=acme, dc=org"

* "cn" --> es el atributo "commonName" (nombre común) de la entrada.
* "ou" --> es el atributo "organizationalUnit" (unidad organizativa).
* "dc" --> es el atributo "domainComponent" (componente de dominio).

![ejemplo](/img/ldap/1.png)

Teniendo en cuenta la anterior explicación vamos a usar el comando **ldapsearch** para ver el estado original de nuestro Entry Tree.

```bash
ldapsearch -x -b "dc=arantxa,dc=gonzalonazareno,dc=org"
```

![directorio](/img/ldap/4.png)

También podemos usar el siguiente comando:

```bash
sudo slapcat
```

![directorio](/img/ldap/5.png)

Para organizar mejor nuestro directorio vamos a agregar dos entradas (Usuarios y Grupos) que se definirán en un fichero con extensión `.ldif`.

```bash
sudo nano user-groups.ldif
```

```txt
dn: ou=Usuarios,dc=arantxa,dc=gonzalonazareno,dc=org
objectClass: organizationalUnit
ou: Usuarios 

dn: ou=Grupos,dc=arantxa,dc=gonzalonazareno,dc=org
objectClass: organizationalUnit
ou: Grupos
```

Añadimos las entradas anteriores con `ldapadd`.

```bash
ldapadd -x -D "cn=admin,dc=arantxa,dc=gonzalonazareno,dc=org" -f user-groups.ldif -W
```

![entradas](/img/ldap/6.png)

Comprobamos los cambios del directorio:

```bash
ldapsearch -x -b "dc=arantxa,dc=gonzalonazareno,dc=org"
```

![entradas](/img/ldap/7.png)

#### 2.1 Usuario prueba

Creo una contraseña cifrada para el usuario prueba.

```bash
arantxa@alfa:~$ sudo slappasswd
New password: 
Re-enter new password: 
{SSHA}pxwn1veQGhNQV0fdA+KPz/42s0Lp3nGk
```

Creo la entrada .ldif para agregar el usuario prueba y el grupo prueba al OU Usuarios y Grupos que hemos creado antes.

```bash
sudo nano user-prueba.ldif
```

```txt
dn: uid=prueba,ou=Usuarios,dc=arantxa,dc=gonzalonazareno,dc=org
objectClass: inetOrgPerson
objectClass: posixAccount
objectClass: shadowAccount
cn: prueba
sn: prueba
uidNumber: 2000
gidNumber: 2000
userPassword: {SSHA}pxwn1veQGhNQV0fdA+KPz/42s0Lp3nGk
loginShell: /bin/bash
homeDirectory: /srv/homes/prueba
```

```bash
sudo nano group-prueba.ldif
```

```txt
dn: cn=prueba,ou=Grupos,dc=arantxa,dc=gonzalonazareno,dc=org
objectClass: posixGroup
gidNumber: 2000
cn: prueba
```

Añadimos las entradas:

```bash
ldapadd -x -D "cn=admin,dc=arantxa,dc=gonzalonazareno,dc=org" -f user-prueba.ldif -W
ldapadd -x -D "cn=admin,dc=arantxa,dc=gonzalonazareno,dc=org" -f group-prueba.ldif -W
```

![entradas](/img/ldap/8.png)

Comprobamos nuestro directorio.

```bash
ldapsearch -x -b "dc=arantxa,dc=gonzalonazareno,dc=org"
```

```txt
# extended LDIF
#
# LDAPv3
# base <dc=arantxa,dc=gonzalonazareno,dc=org> with scope subtree
# filter: (objectclass=*)
# requesting: ALL
#

# arantxa.gonzalonazareno.org
dn: dc=arantxa,dc=gonzalonazareno,dc=org
objectClass: top
objectClass: dcObject
objectClass: organization
o: arantxa.gonzalonazareno.org
dc: arantxa

# Usuarios, arantxa.gonzalonazareno.org
dn: ou=Usuarios,dc=arantxa,dc=gonzalonazareno,dc=org
objectClass: organizationalUnit
ou:: VXN1YXJpb3Mg

# Grupos, arantxa.gonzalonazareno.org
dn: ou=Grupos,dc=arantxa,dc=gonzalonazareno,dc=org
objectClass: organizationalUnit
ou: Grupos

# prueba, Usuarios, arantxa.gonzalonazareno.org
dn: uid=prueba,ou=Usuarios,dc=arantxa,dc=gonzalonazareno,dc=org
objectClass: inetOrgPerson
objectClass: posixAccount
objectClass: shadowAccount
cn: prueba
sn: prueba
uidNumber: 2000
gidNumber: 2000
loginShell: /bin/bash
homeDirectory: /srv/homes/prueba
uid: prueba

# prueba, Grupos, arantxa.gonzalonazareno.org
dn: cn=prueba,ou=Grupos,dc=arantxa,dc=gonzalonazareno,dc=org
objectClass: posixGroup
gidNumber: 2000
cn: prueba

# search result
search: 2
result: 0 Success

# numResponses: 6
# numEntries: 5
```

#### 2.2 NFS para compartir directorio home

He indicado que el home del usuario se creará en /srv/homes/ (que será el directorio a compartir), pero este directorio no está creado. Por lo que debemos crear el directorio para que al conectarse el usuario desde un cliente se cree el home en dicho directorio.

```bash
sudo mkdir /srv/homes
```

Instalamos NFS.

```bash
sudo apt install nfs-kernel-server
```

Modificamos /etc/exports añadiendo la siguiente línea.

```bash
sudo nano /etc/exports
```

```conf
/srv/homes *(rw,sync,no_subtree_check,no_root_squash)
```

* /srv/homes: Es la ruta del directorio que se compartirá.
* *: Indica que cualquier cliente puede acceder al directorio compartido.
* rw: Permite tanto la lectura (read) como la escritura (write) en el directorio compartido.
* sync: Realiza operaciones de escritura sincrónicas, lo que significa que los cambios se escriben de inmediato en el disco antes de confirmar la operación al cliente.
* no_subtree_check: Desactiva la verificación de subdirectorios. En algunos casos, el sistema NFS realiza una comprobación recursiva de los permisos y opciones de configuración para cada subdirectorio. Esta opción desactiva esa verificación para mejorar el rendimiento.
* no_root_squash: Por defecto, cuando el cliente accede al directorio compartido como el usuario root, NFS mapea ese usuario a un usuario anónimo. Esta opción desactiva ese mapeo, permitiendo al cliente acceder al directorio compartido con los privilegios del usuario root.

Reinicio nfs y actualizo la lista de directorios compartidos:

```bash
sudo systemctl restart nfs-kernel-server
sudo exportfs
```

---------------------------------------------------

### 3. Configuración del cliente bravo

Instalo los paquetes que voy a necesitar para configurar el cliente Rocky.

```bash
sudo dnf install openldap-clients sssd sssd-ldap oddjob-mkhomedir sssd-tools
```

Usamos el siguiente comando para seleccionar y aplicar un perfil de autenticación basado en SSSD.

```bash
sudo authselect select sssd with-mkhomedir --force
```

*La opción with-mkhomedir indica que se habilitará la funcionalidad de creación automática de directorios de inicio cuando los usuarios inicien sesión por primera vez.

Activamos el servicio oddjobd y comprobamos que está activo.

```bash
sudo systemctl enable --now oddjobd
sudo systemctl status oddjobd
```

Indicamos el servidor LDAP y la búsqueda base.

```bash
sudo nano /etc/openldap/ldap.conf
---
BASE dc=arantxa,dc=gonzalonazareno,dc=org
URI ldap://alfa.arantxa.gonzalonazareno.org
```

Configuramos el servicio SSSD.

```bash
sudo nano /etc/sssd/sssd.conf
```

```conf
[domain/default]
id_provider = ldap
autofs_provider = ldap
auth_provider = ldap
chpass_provider = ldap
ldap_uri = ldap://alfa.arantxa.gonzalonazareno.org
ldap_search_base = dc=arantxa,dc=gonzalonazareno,dc=org
#ldap_id_use_start_tls = True
#ldap_tls_cacertdir = /etc/openldap/cacerts
cache_credentials = True
#ldap_tls_reqcert = allow

[sssd]
services = nss, pam, autofs
domains = default

[nss]
homedir_substring = /srv/homes
```

Cambiamos los permisos del servicio SSSD para que solo sea accesible por el propietario.

```bash
sudo chmod 0600 /etc/sssd/sssd.conf
```

Reiniciamos el servicio y vemos que está activo.

```bash
sudo systemctl restart sssd
sudo systemctl status sssd
```

![status](/img/ldap/18.png)

Comprobamos que podemos ver las entradas del servidor LDAP.

```bash
ldapsearch -x -b dc=arantxa,dc=gonzalonazareno,dc=org
```

![comprobacion](/img/ldap/9.png)

Creo el punto de montaje y lo monto.

```bash
sudo mkdir /srv/homes
sudo mount alfa.arantxa.gonzalonazareno.org:/srv/homes /srv/homes
```

Hago el montaje permanente:

```bash
sudo nano /etc/fstab
```

```conf
alfa.arantxa.gonzalonazareno.org:/srv/homes   /srv/homes   nfs  defaults   0  0
```

Comprobar:

![comprobacion](/img/ldap/10.png)

***NOTA: Las comprobaciones para ver si se puede escribir en el directorio y si aparecen los cambios del servidor se harán en el punto final ("5. Comprobación final")**

---------------------------------------------------

### 4. Configuración del cliente delta

Instalar ldap, nss y pam.

```bash
sudo apt update
sudo apt install ldap-utils libnss-ldapd libpam-ldapd
```

Nos aparecerá una pantalla en la que tendremos que añadir el servidor ldap (ldap://alfa.arantxa.gonzalonazareno.org) y para la busqueda base (dc=arantxa,dc=gonzalonazareno,dc=org).

![server](/img/ldap/11.png)

![server](/img/ldap/12.png)

También indicamos los servicios que queremos que NSS configure (passwd, group y shadow).

![server](/img/ldap/14.png)

Añadir al fichero de configuración de ldap el servidor y la base de búsqueda:

```bash
sudo nano /etc/ldap/ldap.conf
```

```conf
BASE dc=arantxa,dc=gonzalonazareno,dc=org
URI ldap://alfa.arantxa.gonzalonazareno.org
```

Añadir lo siguiente al fichero/etc/pam.d/common-session para que se cree un directorio home del usuario si no existiera.

```bash
sudo nano /etc/pam.d/common-session
```

```conf
session required        pam_mkhomedir.so skel=/etc/skel umask=077
```

Reiniciar los servicios:

```bash
sudo systemctl restart nscd nslcd
```

Crear el punto de montaje y montarlo con el servidor ldap.

```bash
sudo mkdir /srv/homes
sudo mount alfa.arantxa.gonzalonazareno.org:/srv/homes /srv/homes
```

```bash
arantxa@delta:~$ df -h
Filesystem                                   Size  Used Avail Use% Mounted on
/dev/vda1                                     30G  4.0G   25G  14% /
none                                         492K  4.0K  488K   1% /dev
tmpfs                                        986M     0  986M   0% /dev/shm
tmpfs                                        198M  144K  197M   1% /run
tmpfs                                        5.0M     0  5.0M   0% /run/lock
tmpfs                                        198M     0  198M   0% /run/user/1001
alfa.arantxa.gonzalonazareno.org:/srv/homes   30G  4.0G   25G  14% /srv/homes
```

Hacer el montaje permanente:

```bash
sudo nano /etc/fstab
```

```conf
alfa.arantxa.gonzalonazareno.org:/srv/homes   /srv/homes   nfs  defaults   0  0
```

Comprobar que se pueden ver las entradas del servidor LDAP:

```bash
ldapsearch -x -b dc=arantxa,dc=gonzalonazareno,dc=org
```

![entradas](/img/ldap/13.png)

---------------------------------------------------

### 5. Comprobación final

**En servidor LDAP:**

```bash
sudo ls -l /srv/homes/
```

**En cliente Delta:**

Entramos con el usuario prueba.

```bash
sudo su prueba
cd
pwd
```

Crear fichero de prueba:

```bash
nano prueba.txt
```

```txt
Este fichero ha sido creado desde Delta
```

```bash
cat prueba.txt
ls -l
```

**En servidor LDAP:**

```bash
sudo ls -l /srv/homes/prueba/
sudo cat /srv/homes/prueba/prueba.txt
```

**En cliente Bravo:**


```bash
sudo su prueba
cd
pwd
ls -l
cat prueba.txt
nano prueba.txt
```

```txt
Este fichero ha sido creado desde Delta
Modificando desde Bravo
```

```bash
cat prueba.txt
nano prueba2.txt
```

```txt
Fichero creado en Bravo
```

```bash
cat prueba2.txt
```

**En servidor LDAP:**

```bash
sudo ls -l /srv/homes/prueba/
sudo cat /srv/homes/prueba/prueba.txt
sudo cat /srv/homes/prueba/prueba2.txt
```

#### Captura de la comprobación anterior en las tres máquinas al mismo tiempo:

![prueba-final](/img/ldap/17.png)
