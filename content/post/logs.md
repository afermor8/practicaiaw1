---
title: "Recolección centralizada de logs de sistema, mediante journald."
date: 2023-05-27T19:09:34+02:00
draft: true
tags: ["ASO"]
bigimg: [{src: "/img/triangle.jpg", desc: "Triangle"}, {src: "/img/sphere.jpg", desc: "Sphere"}, {src: "/img/hexagon.jpg", desc: "Hexagon"}]
---

## Implementa en tu escenario de trabajo de Openstack, un sistema de recolección de log mediante journald. Para ello debes, implementar un sistema de recolección de log mediante el paquete systemd-journal-remote, o similares


Empezamos instalando el paquete systemd-journal-remote en alfa, bravo, charlie y delta. Bravo es un sistema Rocky Linux 9 por lo que habrá que instalarlo con DNF, pero en el resto de máquinas tenemos sistemas Debian (alfa) y Ubuntu (charlie y delta), por lo que lo instalaremos con APT.

**Alfa, Charlie y Delta:**

```bash
sudo apt install systemd-journal-remote
```

**Bravo:**

```bash
sudo dnf install systemd-journal-remote
```

### Configuración en ALFA

En alfa se recogerán los logs de las demás máqinas por lo que configuraremos systemd-journal-remote siguiendo los pasos descritos a continuación.

Primero activamos las unidades systemd.

```bash
sudo systemctl enable --now systemd-journal-remote.socket
sudo systemctl enable systemd-journal-remote.service
```


Editamos la unidad systemd-journal-remote para que no escuche por https.


```bash
sudo nano /lib/systemd/system/systemd-journal-remote.service 
```

Cambiamos la siguiente línea:

```conf
[Service]
ExecStart=/lib/systemd/systemd-journal-remote --listen-http=-3 --output=/var/log/journal/remote/
```

Guardamos la configuración y reiniciamos el servicio:

```bash
sudo systemctl daemon-reload
```

A continuación configuramos el servicio.

```bash
sudo nano /etc/systemd/journal-remote.conf
```

```conf
[Remote]
Seal=false
SplitMode=host
```


Activamos el servicio.

```bash
sudo systemctl start systemd-journal-remote.service
```

Al activar el servicio se crea el directorio **/var/log/journal/remote**.

Comprobamos que el servicio funciona correctamente y que se ha abierto el puerto tcp que utiliza este servicio.

```bash
sudo systemctl status systemd-journal-remote.service
sudo systemctl status systemd-journal-remote.socket
sudo ss -puntal
```

![](/img/logs/1.png)



### Configuración en BRAVO, CHARLIE y DELTA

Iniciamos systemd-journal-upload.

```bash
sudo systemctl enable systemd-journal-upload.service
```

Añadimos la dirección de la máquina Alfa al fichero de configuración journal-upload.

```bash
sudo nano /etc/systemd/journal-upload.conf
```

```conf
[Upload]
URL=http://192.168.0.1:19532
```

Reiniciamos el servicio.

```bash
sudo systemctl restart systemd-journal-upload
```


## Comprobación

En el directorio **/var/log/journal/remote/** de la máquina **Alfa** se han creado los ficheros de logs de los servidores **Bravo**, **Charlie** y **Delta**.

```bash
sudo ls -la /var/log/journal/remote/
```

![](/img/logs/2.png)


Para comprobar que recoge los logs del sistema vamos a mandar un mensaje desde cada máquina de la siguiente forma:

```bash
logger -p syslog.debug "### Mensaje de prueba desde BRAVO ###"
logger -p syslog.debug "### Mensaje de prueba desde CHARLIE ###"
logger -p syslog.debug "### Mensaje de prueba desde DELTA ###"
```

Y si vemos los ficheros de log de las máquinas podremos ver esos mensajes:

- **BRAVO:**

```bash
sudo journalctl -e --file=/var/log/journal/remote/remote-172.16.0.200.journal
```

![](/img/logs/3.png)

- **CHARLIE:**

```bash
sudo journalctl -e --file=/var/log/journal/remote/remote-192.168.0.2.journal
```

![](/img/logs/4.png)

- **DELTA:**

```bash
sudo journalctl -e --file=/var/log/journal/remote/remote-192.168.0.3.journal
```

![](/img/logs/5.png)
