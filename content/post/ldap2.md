---
title: "Poblar un directorio LDAP desde un fichero CSV"
date: 2023-06-14T10:55:58+02:00
draft: true
tags: ["ASO"]
bigimg: [{src: "/img/triangle.jpg", desc: "Triangle"}, {src: "/img/sphere.jpg", desc: "Sphere"}, {src: "/img/hexagon.jpg", desc: "Hexagon"}]
---


**Crear entre todos los alumnos de la clase que vayan a hacer esta tarea un fichero CSV que incluya información personal de cada uno incluyendo los siguientes datos:**

- **Nombre**
- **Apellidos**
- **Dirección de correo electrónico**
- **Nombre de usuario**
- **Clave pública ssh**

**Añadir otro fichero con la información de las máquinas de los alumnos:**

- **Hostname**
- **IPv4**
- **Clave pública ssh de la máquina**

**Añadir el esquema openssh-lpk al directorio para poder incluir claves públicas ssh en un directorio LDAP.**

**Hacer un script en bash o en python que utilice el fichero como entrada y pueble el directorio LDAP con un objeto para cada alumno utilizando los ObjectClass posixAccount e inetOrgPerson.**

**Configurar el sistema para que sean válidos los usuarios del LDAP.**

**Configurar el servicio ssh para que permita acceder a los usuarios del LDAP utilizando las claves públicas que hay allí, en lugar de almacenarlas en .ssh/authorized_keys, que sólo permita acceder a los equipos que estén en el LDAP en lugar del fichero .ssh/known_hosts y que se cree el directorio "home" al vuelo.**

---------------------------------------------

## 1. Ficheros CSV

Antes de crear los ficheros debemos saber qué es un fichero CSV. Un fichero CSV (Comma-Separated Values) es un formato de archivo que consiste en una serie de líneas donde cada línea representa una fila de datos, y los valores de cada columna están separados por comas. Por lo general, el primer registro de un fichero CSV contiene los nombres de las columnas.

Sabiendo esto voy a crear un fichero CSV para lo usuarios en mi servidor LDAP, Alfa, cuyo contenido será el siguiente:

```txt
nombre,apellidos,correo,usuario,clave-publica
```

Y otro para las máquinas:

```txt
hostname,ip,clave-publica-maquina
```

Conociendo los datos de mis compañeros voy a crear el siguiente fichero CSV.

```bash
cd ldap/
nano usuarios.csv
```

```csv
pepito,bravo,pepitobravo@gmail.com,pepitobravo,ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQC/b4KSUJ9k9syYwQp7YEJWfV/ovSEnYQvSRJFaXReRMhwGWxS9UKfY/CdIBRUM3toQglYqydgjBeIZU4LjLLdvVZr8TKp5SVhmh4E4igQTDV2FYQ/ejGnS6laFNGB2q3aJ98faA+VXBgvKasKI1GZK9FlAlV6Qqmfo0TSjph/2IkA4Hu6k6vGEB1wUiIxK6dXYcxrHjP9DT9SPgWq8gqoKYzJSFQiVdgVT0d3U5RpMzAawlAPY0mUqKQuVHW6DIgWVGXO6GryXeLMh+5ViyT3evUzSws/6kvSSO9tVt+FcLOawEnvIazfGZNcwsAWIFK/5Qdg57uQH6pSeeybHAOse0c3D5nK9RF8AZHZGREuQ4L5koTdJOgKV7tO6f5H/o3GXVEwLuQXs+ffjQ2gD6mNwQ4xepg5DB1AHEVXqGB8y4SPhbhF9Pu9Dr57pSn2IetFMNa8yocL1Ymi4cRDrqOLQ1imO4tqaLPHhbbT0KSJQ+G+WcG3RoMU4L/+y6IUxuDE= pepitobravo@bravo.arantxa.gonzalonazareno.org
pepito,perez perez,pepito@gmail.com,pepito,ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDLrLKXcNE/pYgh0SjNy/KHYp4PD7vGa+q1Kpk3ynkJooe56xmWTDLFGyMeUUyp+Mpb5FEHQ3EP04TsedU/+POmmU6p0u3WEZsZjb5EqgGV2lE5asgZGhcZ8EnRZS3LkLO7ktRkB6LmIVRGJ3wKqpITUmCaFuyBPbNLTx191AhX0DhKDQhQzyKIvXC5WNMVKNniQDJoSH4qrsgDpoF9VMjrNCHPrnjLpuDYgy5qWevRtjbegj26jE9xhCPhO6h/aLeN2g1aH90DkWjmEzwXMGRXuGhdWRB4y1VEPhEZd6eCR4cKp4qwh02gRvR6y3nGkgqUy9k5MyaBwn9nm3WTJ7149CmLbZIR3wnIFefXVMzTON+xajMOqBWJeGHYGYIqXegmWhOabvwO+m4UG+glE3PtC1fojj8vEYyilyKTc/L1wJkYXyubyoOGSNEc0tSWygic9pei+39yGo1gI9TpDdjYirmU8CmL4tLd7eT4hrUtY4n4fiz/4F0jBB7wiWz1e7s= pepito@tars
```

(*En mi caso, esta tarea la estoy realizando en Junio donde ya quedan pocos alumnos y horas de clase así que la prueba para ver si funciona la haré con un usuario llamado pepitobravo desde la máquina bravo y pepito desde mi host)

![usuario-pepito](/img/ldap2/1.png)

![usuario-pepito](/img/ldap2/2.png)

![usuario-pepito](/img/ldap2/3.png)

![usuario-pepito](/img/ldap2/4.png)

Creo el fichero CSV para la información de las máquinas.

```bash
nano maquinas.csv
```

```csv
bravo.arantxa.gonzalonazareno.org,172.16.0.200,ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCqLx+FTXirsOd2C/eRc9zAoxVtpIidq+nmIIIqi+Pxgszc+PJK514yHiMgkZYQr03DDDNAbzlEVrbvyo3gFWMmlBpQcRMIYg2SHbuFdFRhW65TdpvED0T0L0qWHY2j1itXaS3eD6fDTMwnuy2uVPMnkc/5RbwW3ebjskbKIJpepl//OcU0hXnR6wXuLmSHKI2ISxSGMtHG5adV6yXRNfWBcWUqyldOs9ITwVHKq7WVhgOM+IHm/B6eGY9SObbrEuJ0NMnoV2xeATA/pUGxjYjFDMGmH3eN0CIeQSO7Lo5bUvfoWOfTyAkeISiiaG7KDZL/WXZGwHRyjG0+KBxvyvBJz3ubuIrKecOPc9jHhoAgX6CaVlnFIxHXz8g4zKKnB9QBo2+RbLErUxgRPg5kXUK8p1S31ZD1f+fPNxx4Tt2CQAgOSNfjskW6HPuyKR5liS+U3c8mbBk9jl90cxr/S4gRu0POMTtyo2wRxxMHUidnAmjYb5nXFdhE836msH2LcHs= root@bravo.arantxa.gonzalonazareno.org
tars,172.29.0.46,ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQC0ZD0pDFKGbdan/kOe5gwFENrJZ2o4r5mZ0/OQhQvOODUN8TalhuSrREDnIFwTIhC26o+Gbyh0ztwjhbB4RO+SDkBiygWfvx0B6TSuDnQ6ZSVLrEr7Ru5IznbLjzAXzhNNQ3hc7Sve2sqIvqEQjwzjqnnfQz6870WJMB0KYdmQ5ZM/YFzCvjTMXKBndKHB8DxOslHVbDuRtUmjLF60aWwnIn1WUIE//jNC61hnnQmtgg/rYXk2wPzYDaalfwjNChkbCR17XLraoagdD2aqi9MqedM9qBVO+xwKlI7Z69H7y9HjBsHKuYg+ptTS6yV4APGgbzqHiH6VLDauL2+Zx+4jO46EjPfZZF4bbGhVgRGkBlf8rrvlOh0/r/U3g9pBcnrulrsHWFERl10JHRQPiAM1HNKKM5PI+dMwcV/2iKGAyh6SI876EHsxoQr7TzyBHqSRf4gJEAcmVLLKqx9D0wQR197n4m0hA2Kp69XinoKhLJTPkLITIdSR2hGDe6vFsws= root@debian
```

Para saber la clave pública de las máquinas:

```bash
cat /etc/ssh/ssh_host_rsa_key.pub
```

![usuario-pepito](/img/ldap2/5.png)

![usuario-pepito](/img/ldap2/6.png)

## 2. Entrada para máquinas

Los usuarios se agregarán a la entrada que creamos en la [práctica anterior](https://afermor8.github.io/post/ldap1/), llamada Usuarios. Tendríamos que agregar una entrada para agregar la información de las máquinas de esos usuarios.
Así pues, creamos una entrada .ldif para añadir al directorio de LDAP que se llamará Máquinas.

```bash
nano maquinas.ldif
```

```txt
dn: ou=Maquinas,dc=arantxa,dc=gonzalonazareno,dc=org
objectClass: organizationalUnit
ou: Maquinas
```

La añadimos al directorio.

```bash
ldapadd -x -D "cn=admin,dc=arantxa,dc=gonzalonazareno,dc=org" -f maquinas.ldif -W
```

(*Recordatorio: mi contraseña era "admin")

Comprobamos que se ha añadido correctamente:

```bash
ldapsearch -x -b "dc=arantxa,dc=gonzalonazareno,dc=org"
```

```bash
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

# Maquinas, arantxa.gonzalonazareno.org
dn: ou=Maquinas,dc=arantxa,dc=gonzalonazareno,dc=org
objectClass: organizationalUnit
ou: Maquinas

# search result
search: 2
result: 0 Success

# numResponses: 7
# numEntries: 6
```

## 3. Esquema Openssh-LPK

El esquema OpenSSH-LPK (OpenSSH LDAP Public Key) es una extensión del servidor OpenSSH que permite la autenticación basada en claves públicas almacenadas en un servidor LDAP. Cuando un usuario intenta autenticarse en un servidor SSH habilitado con OpenSSH-LPK, el servidor consulta el servidor LDAP para obtener la clave pública asociada con el usuario. Si la clave pública coincide con la clave privada utilizada por el cliente SSH, se permite la autenticación y se concede el acceso.

Para crear el equema lo haré añadiendo una entrada .ldif con el siguiente contenido:

```bash
sudo nano /etc/ldap/schema/openssh-lpk.ldif
```

```conf
dn: cn=openssh-lpk,cn=schema,cn=config
objectClass: olcSchemaConfig
cn: openssh-lpk
olcAttributeTypes: ( 1.3.6.1.4.1.24552.500.1.1.1.13 NAME 'sshPublicKey'
  DESC 'MANDATORY: OpenSSH Public key'
  EQUALITY octetStringMatch
  SYNTAX 1.3.6.1.4.1.1466.115.121.1.40 )
olcObjectClasses: ( 1.3.6.1.4.1.24552.500.1.1.2.0 NAME 'ldapPublicKey'
  SUP top AUXILIARY
  DESC 'MANDATORY: OpenSSH LPK objectclass'
  MAY ( sshPublicKey $ uid )
  )
```

- olcAttributeTypes: El número OID "1.3.6.1.4.1.24552.500.1.1.1.13" se utiliza como identificador único para el atributo "sshPublicKey". Este número OID es específico del esquema OpenSSH-LPK. Tiene características específicas, como su nombre (NAME), descripción (DESC), igualdad (EQUALITY), sintaxis (SYNTAX), etc.
- olcObjectClasses: El número OID "1.3.6.1.4.1.24552.500.1.1.2.0" se utiliza como identificador único para la clase de objeto "ldapPublicKey". Al igual que el anterior, este número OID es específico del esquema OpenSSH-LPK. También tiene sus características, como su nombre (NAME), superclase (SUP), atributos auxiliares (AUXILIARY), descripción (DESC), y los atributos opcionales que puede contener (MAY).

Añadimos el esquema al directorio del servidor LDAP:

```bash
sudo ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/ldap/schema/openssh-lpk.ldif
```

```bash
SASL/EXTERNAL authentication started
SASL username: gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth
SASL SSF: 0
adding new entry "cn=openssh-lpk,cn=schema,cn=config"
```

## 4. Script python

Voy a crear un script en python que servirá para leer los ficheros csv y añadir la información de los usuarios y de las máquinas al directorio del servidor LDAP.
Se creará un entorno virtual y se instalará un módulo llamado "pyhton3-ldap".

```bash
sudo apt install python3-venv
python3 -m venv entorno-ldap
source entorno-ldap/bin/activate
```

Ahora instalamos el módulo python3-ldap.

```bash
pip install python3-ldap ldap3==2.6
```

Y creo el script python que tendrá el siguiente código (me he basado en un código anterior al que le he hecho algunas modificaciones para mejorarlo, pero es un código básico y se le podrían seguir haciendo mejoras):

```bash
nano ldap-csv-users-hosts.py
```

```py
#!/usr/bin/env python

import ldap3
from ldap3 import Connection, ALL
from getpass import getpass
from sys import exit

# Shell que se le asigna a los usuarios
shell = '/bin/bash'

# Ruta absoluta del directorio que contiene los directorios personales de los usuarios. Terminado en "/"
home_dir = '/home/'

# El valor inicial para los UID que se asignan al insertar usuarios. 
uid_number = 5000

# El GID que se le asigna a los usuarios. Si no se manda al anadir el usuario da error.
gid = 5000

# Leemos el fichero .csv de los usuarios y guardamos cada linea en una lista.
with open('usuarios.csv', 'r') as fichero:
  usuarios = fichero.readlines()
# Leemos el fichero .csv de las máquinas y guardamos cada linea en una lista.
with open('maquinas.csv', 'r') as fichero_maquinas:
  maquinas = fichero_maquinas.readlines()

### Parametros para la conexion
ldap_ip = 'ldap://alfa.arantxa.gonzalonazareno.org:389'
dominio_base = 'dc=arantxa,dc=gonzalonazareno,dc=org'
user_admin = 'admin' 
contrasena = getpass('Contrasena: ')

# Intenta realizar la conexion.
conn = Connection(ldap_ip, 'cn={},{}'.format(user_admin, dominio_base),contrasena)
# conn.bind() devuelve "True" si se ha establecido la conexion y "False" en caso contrario.
# Si no se establece la conexion imprime por pantalla un error de conexion.
if not conn.bind():
  print('No se ha podido conectar con ldap') 
  if conn.result['description'] == 'invalidCredentials':
    print('Credenciales no validas.')
  # Termina el script.
  exit(0)

# Recorre la lista de usuarios
for user in usuarios:
  # Separa los valores del usuario usando como delimitador la ","
  # y asigna cada valor a la variable correspondiente.
  user = user.split(',')
  cn = user[0]
  sn = user[1]
  mail = user[2]
  uid = user[3]
  ssh = user[4]
  #Anade el usuario.
  conn.add(
    'uid={},ou=Usuarios,{}'.format(uid, dominio_base),
    object_class = 
      [
      'top',
      'inetOrgPerson',
      'posixAccount', 
      'ldapPublicKey'
      ],
    attributes =
      {
      'cn': cn,
      'sn': sn,
      'mail': mail,
      'uid': uid,
      'uidNumber': str(uid_number),
      'gidNumber': str(gid),
      'homeDirectory': '{}{}'.format(home_dir,uid),
      'loginShell': shell,
      'sshPublicKey': str(ssh)
      })
  #Si la entrada ya existe aparece un mensaje de que ya existe,
  #si se añade dice que ha sido agregada y si hay un error dice cuál es el error
  if conn.result['description'] == 'entryAlreadyExists':
    print('El usuario {} ya existe.'.format(uid))
  elif conn.result['description'] == 'success':
    print('El usuario {} ha sido agregado.'.format(uid))
  else:
    print('Error al agregar el usuario {}: {}'.format(uid, conn.result['description']))
  # Aumenta el contador para asignar un UID diferente a cada usuario (cada vez que ejecutemos el script debemos asegurarnos de ante mano que no existe dicho uid en el directorio ldap, o se solaparian los datos)
  uid_number += 1

#Recorre la lista de máquinas y separa los valores usando split en la ","
for maquina in maquinas:
  maquina = maquina.split(',')
  hostname = maquina[0]
  ipv4 = maquina[1]
  ssh_key = maquina[2]
  # Añade la información de la máquina al directorio LDAP
  conn.add(
    'cn={},ou=Maquinas,{}'.format(hostname, dominio_base),
    object_class=[
      'top',
      'device',
      'ipHost',
      'ldapPublicKey'
    ],
    attributes={
      'cn': hostname,
      'ipHostNumber': ipv4,
      'sshPublicKey': ssh_key
    }
  )
  #Si la entrada ya existe aparece un mensaje de que ya existe,
  #si se añade dice que ha sido agregada y si hay un error dice cuál es el error
  if conn.result['description'] == 'entryAlreadyExists':
    print('La máquina {} ya existe.'.format(hostname))
  elif conn.result['description'] == 'success':
    print('La máquina {} ha sido agregada.'.format(hostname))
  else:
    print('Error al agregar la máquina {}: {}'.format(hostname, conn.result['description']))

#Cierra los ficheros.
fichero.close()
fichero_maquinas.close()

#Cierra la conexion.
conn.unbind()
```

Ejecutamos el código python.

```bash
(entorno-ldap) arantxa@alfa:~/ldap$ python3 ldap-csv-users-hosts.py 
Contrasena: 
El usuario pepitobravo ha sido agregado.
El usuario pepito ha sido agregado.
La máquina bravo.arantxa.gonzalonazareno.org ha sido agregada.
La máquina tars ha sido agregada.
```

Comprobamos que se han añadido los usuarios y las máquinas. (He puesto solo una parte de las claves para que no sea tan largo)

```bash
ldapsearch -x -b "ou=Usuarios,dc=arantxa,dc=gonzalonazareno,dc=org"
```

```txt
# extended LDIF
#
# LDAPv3
# base <ou=Usuarios,dc=arantxa,dc=gonzalonazareno,dc=org> with scope subtree
# filter: (objectclass=*)
# requesting: ALL
#

# Usuarios, arantxa.gonzalonazareno.org
dn: ou=Usuarios,dc=arantxa,dc=gonzalonazareno,dc=org
objectClass: organizationalUnit
ou:: VXN1YXJpb3Mg

# pepito, Usuarios, arantxa.gonzalonazareno.org
dn: uid=pepito,ou=Usuarios,dc=arantxa,dc=gonzalonazareno,dc=org
cn: pepito
sn: perez perez
mail: pepito@gmail.com
uid: pepito
uidNumber: 5001
gidNumber: 5000
homeDirectory: /home/pepito
loginShell: /bin/bash
sshPublicKey:: c3NoLXJzYSBBQUFBQjNOemFDMXljMkVBQUFBREFRQUJBQUFCZ1FETHJMS1hjTkU...
objectClass: top
objectClass: inetOrgPerson
objectClass: posixAccount
objectClass: ldapPublicKey

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

# pepitobravo, Usuarios, arantxa.gonzalonazareno.org
dn: uid=pepitobravo,ou=Usuarios,dc=arantxa,dc=gonzalonazareno,dc=org
cn: pepito
sn: bravo
mail: pepitobravo@gmail.com
uid: pepitobravo
uidNumber: 5000
gidNumber: 5000
homeDirectory: /home/pepitobravo
loginShell: /bin/bash
sshPublicKey:: c3NoLXJzYSBBQUFBQjNOemFDMXljMkVBQUFBREFRQUJBQUFCZ1FDL2I0S1NVSjl...
objectClass: top
objectClass: inetOrgPerson
objectClass: posixAccount
objectClass: ldapPublicKey

# search result
search: 2
result: 0 Success

# numResponses: 5
# numEntries: 4
```

```bash
ldapsearch -x -b "ou=Maquinas,dc=arantxa,dc=gonzalonazareno,dc=org"
```

```txt
# extended LDIF
#
# LDAPv3
# base <ou=Maquinas,dc=arantxa,dc=gonzalonazareno,dc=org> with scope subtree
# filter: (objectclass=*)
# requesting: ALL
#

# Maquinas, arantxa.gonzalonazareno.org
dn: ou=Maquinas,dc=arantxa,dc=gonzalonazareno,dc=org
objectClass: organizationalUnit
ou: Maquinas

# tars, Maquinas, arantxa.gonzalonazareno.org
dn: cn=tars,ou=Maquinas,dc=arantxa,dc=gonzalonazareno,dc=org
cn: tars
ipHostNumber: 172.29.0.46
sshPublicKey:: c3NoLXJzYSBBQUFBQjNOemFDMXljMkVBQUFBREFRQUJBQUFCZ1FDMFpEMHBERkt...
objectClass: top
objectClass: device
objectClass: ipHost
objectClass: ldapPublicKey

# bravo.arantxa.gonzalonazareno.org, Maquinas, arantxa.gonzalonazareno.org
dn: cn=bravo.arantxa.gonzalonazareno.org,ou=Maquinas,dc=arantxa,dc=gonzalonaza
 reno,dc=org
cn: bravo.arantxa.gonzalonazareno.org
ipHostNumber: 172.16.0.200
sshPublicKey:: c3NoLXJzYSBBQUFBQjNOemFDMXljMkVBQUFBREFRQUJBQUFCZ1FDcUx4K0ZUWGl...
objectClass: top
objectClass: device
objectClass: ipHost
objectClass: ldapPublicKey

# search result
search: 2
result: 0 Success

# numResponses: 4
# numEntries: 3
```

## 5. Esquema del escenario que queremos montar

El escenario que vamo a montar sería el siguiente:

- **Alfa:**
  - **Servidor LDAP:** almacenamiento de la información de usuarios y sus máquinas, incluyendo las claves públicas.
  - **Servidor SSHD:** se configura sshd_config para que cuando haya una conexión del cliente se utilice el comando /usr/bin/sss_ssh_authorizedkeys para obtener las claves públicas de los usuarios que intentan iniciar sesión
  - Se acedería a esta máquina por ssh al usuario del directorio LDAP.

- **Bravo:**
  - **Cliente LDAP:** se conecta al servidor LDAP y realiza las búsquedas de información de usuarios y hosts y sus claves públicas.
  - **Cliente SSH:** se configura ssh_config para establecer que, en lugar de conectarse directamente al servidor SSH de destino, el cliente SSH utilice el comando de proxy /usr/bin/sss_ssh_knownhostsproxy para establecer la conexión SSH. El comando de proxy se encargará de realizar acciones adicionales, como obtener y verificar los hosts conocidos para establecer la conexión con el servidor SSHD
  - **Demonio SSSD:** estaría configurado para especificar la ubicación del servidor LDAP, la configuración de autenticación y la configuración de búsqueda de claves públicas.
  - Esta máquina se encargaría de conectarse al servidor LDAP y buscar las claves públicas del usuario y el host almacenadas en el directorio LDAP (no debe buscarlas en authorized_keys y known_hosts).

*NOTA: hay que tener en cuenta que ya tenemos instalado y configurado el servidor LDAP y el cliente de la [práctica anterior](https://afermor8.github.io/post/ldap1/). Habiendo realizado esa práctica solo tendremos que realizar los siguientes pasos*

## 6. Configuración del servidor SSHD (Alfa)

Instalamos el paquete sssd para que nos aparezca el comando que usaremos para buscar las claves públicas de los usuarios. Y activamos la creación automática de directorios de los usuarios si no estuviera ya configurado.

```bash
sudo apt install sssd-ldap
sudo pam-auth-update --enable mkhomedir
```

Modificamos el fichero de configuración del servidor SSHD y añadimo lo siguiente:

```bash
sudo nano /etc/ssh/sshd_config
```

```conf
# Ruta del comando que se ejecutará para obtener las claves de autenticación SSH, en este caso sss_ssh_authorizedkeys, mediante SSSD, con el usuario nobody
AuthorizedKeysCommand /usr/bin/sss_ssh_authorizedkeys 
AuthorizedKeysCommandUser nobody
```

El comando sss_ssh_authorizedkeys se usa para buscar las claves de los usuarios en el servidor LDAP usando SSSD.

Si queremos comprobar que el comando funciona correctamente podemos usarlo con uno de los usuario del directorio LDAP y deberíamos obtener su clave pública.

```bash
/usr/bin/sss_ssh_authorizedkeys pepito
```

![clave-usuario-pepito](/img/ldap2/7.png)

Por último comentar que en el servidor tengo configurado sssd de la siguiente forma.

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
debug_level = 10

[sssd]
services = nss, pam, autofs, ssh
domains = default
debug_level = 6

[ssh]
debug_level = 8
```

```bash
sudo chmod 0600 /etc/sssd/sssd.conf
sudo systemctl restart sssd
sudo systemctl status sssd
```

## 7. Configuracion del cliente LDAP, SSH y SSSD (en Bravo)

**Configuración del cliente SSH:**

```bash
sudo nano /etc/ssh/ssh_config
```

```conf
#ProxyCommand indica el comando que se utilizará como intermediario para manejar los hosts conocidos de SSH, en este caso sss_ssh_knownhostsproxy, mediante SSSD
Match all
    ProxyCommand /usr/bin/sss_ssh_knownhostsproxy -p %p %h
    GlobalKnownHostsFile /var/lib/sss/pubconf/known_hosts
#%p %h: Son placeholders que se reemplazarán con el puerto (%p) y el host (%h) durante la ejecución del comando ProxyCommand
```

Reiniciar el servicio sshd y comprobar que funciona correctamente.

```bash
sudo systemctl restart sshd
sudo systemctl status sshd
```

**SSSD:**

En mi caso ya tengo instalado SSSD en el cliente Bravo. Pero si quisiéramos hacerlo en un cliente Debian lo tendríamos que instalar y configurar de la siguiente forma:

```bash
sudo apt update
sudo apt install ldap-utils sssd-ldap
sudo pam-auth-update --enable mkhomedir
```

Si ya lo teníamos instalado pasamos a modificar el fichero sssd.conf.

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

ldap_host_search_base = ou=Maquinas,dc=arantxa,dc=gonzalonazareno,dc=org
ldap_host_object_class = ipHost
ldap_host_fqdn = cn

#ldap_id_use_start_tls = True
#ldap_tls_cacertdir = /etc/openldap/cacerts
cache_credentials = True
#ldap_tls_reqcert = allow
debug_level = 10

[sssd]
services = nss, pam, autofs, ssh
domains = default
debug_level = 6

[ssh]
debug_level = 8
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

## 8. Comprobaciones

Tras mucha prueba y error no he conseguido que la clave pública de las máquinas se obtenga del directorio LDAP. He leído en algún sitio que con FreeIPA debería funcionar.

Para saber si se obtiene la clave usando el comando y guardándose en el directorio especificado podemos probar lo siguiente:

```bash
#Para conectarnos mediante ssh y que nos aparezca información de lo que ha ido ocurriendo en la conexión
ssh -vvv -p 22 -l pepitobravo alfa.arantxa.gonzalonazareno.org

#Se pueden ver los logs de sssd ya que pusimos en el fichero la opción debug_level a un nivel alto
sudo tail -f /var/log/sssd/sssd.log
sudo tail -f /var/log/sssd/sssd_default.log

#Podemos ver auth.log y syslog
sudo tail -f /var/log/syslog
sudo tail -f /var/log/auth.log
```

Con alguno de los comandos anteriores también se puede comprobar que la búsqueda de la clave pública del usuario en el directorio LDAP es satisfactoria.

Comprobamos que se puede acceder al servidor LDAP haciendo SSH desde un cliente y que se ha creado su home.

Desde bravo:

```
sudo su pepitobravo
ssh pepitobravo@alfa.arantxa.gonzalonazareno.org
pwd
nano hola.txt
hola soy el usuario pepitobravo

cat hola.txt
id
```

![bravoprueba](/img/ldap2/8.png)

Desde mi portátil:

```
sudo nano /etc/hosts
172.22.201.158  alfa.arantxa.gozalonazareno.org

sudo su pepito
ssh pepito@alfa.arantxa.gonzalonazareno.org
pwd
nano hola-desde-tars.txt
hola soy el usuario pepito

cat hola-desde-tars.txt
id
```

![tarsprueba](/img/ldap2/9.png)

Desde alfa:

```bash
sudo ls -l /home/pepitobravo
sudo cat /home/pepitobravo/hola.txt
sudo ls -l /home/pepito
sudo cat /home/pepito/hola-desde-tars.txt
```

![alfaprueba](/img/ldap2/10.png)

Podemos también comprobar si podemos obtener la información del usuario usandp `getent`. Si el sistema conoce la información del usuario es que funciona correctamente.

```bash
sudo getent passwd pepitobravo
sudo getent passwd pepito
```

![alfa prueba](/img/ldap2/11.png)

## **9. WEBS de interés:**

Posts y webs oficiales:

- https://www.burnison.ca/notes/managing-ssh-known-hosts-with-sssd-and-ldap
- https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/linux_domain_identity_authentication_and_policy_guide/openssh-sssd
- https://manpages.ubuntu.com/manpages/lunar/en/man1/sss_ssh_knownhostsproxy.1.html
- https://wiki.debian.org/AuthenticatingLinuxWithActiveDirectorySssd
- https://linux.die.net/man/5/sssd-ldap
- https://linux.die.net/man/1/sss_ssh_knownhostsproxy
- https://www.freeipa.org/images/1/10/Freeipa30_SSSD_OpenSSH_integration.pdf

Webs de respuestas a dudas:

- https://askubuntu.com/questions/906170/ssh-with-ldap-authentication-activedirectory-and-ssh-keys-stored-in-ad
- https://github.com/SSSD/sssd/issues/6100
- https://github.com/SSSD/sssd/issues/6294
- https://lists.fedorahosted.org/archives/list/sssd-users@lists.fedorahosted.org/thread/NC3AMBO62EQWIR7AXPRA2OKAZ263ESIV/

**Extra:**

Para ver la lista de esquemas por su 'dn':

```bash
arantxa@alfa:~$ sudo ldapsearch -Q -LLL -Y EXTERNAL -H ldapi:/// -b cn=schema,cn=config dn

dn: cn=schema,cn=config

dn: cn={0}core,cn=schema,cn=config

dn: cn={1}cosine,cn=schema,cn=config

dn: cn={2}nis,cn=schema,cn=config

dn: cn={3}inetorgperson,cn=schema,cn=config

dn: cn={4}openssh-lpk,cn=schema,cn=config
```