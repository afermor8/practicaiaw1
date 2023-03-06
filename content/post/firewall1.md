---
title: "Cortafuegos I: de nodo con iptables"
date: 2023-03-06T11:09:59+01:00
draft: true
tags: ["SAD"]
bigimg: [{src: "/img/triangle.jpg", desc: "Triangle"}, {src: "/img/sphere.jpg", desc: "Sphere"}, {src: "/img/hexagon.jpg", desc: "Hexagon"}]
---



Realiza el ejercicio 1 de cortafuegos de nodo con IPtables de la web de Jose Domingo. Puedes encontrarlo en:

https://fp.josedomingo.org/seguridadgs/u03/ejercicio1.html

Debes incluir la demostración de que funciona cada una de las reglas previas del enunciado.

Debéis incluir por cada apartado del ejercicio las reglas y la demostración del funcionamiento

---------------------------------

## Implementación de un cortafuegos personal

Vamos a realizar los primeros pasos para implementar un cortafuegos en un nodo de una red, aquel que se ejecuta en el propio equipo que trata de proteger, lo que a veces se denomina un cortafuegos personal.

**Esquema de red:**

Vamos a utilizar una máquina en openstack, que vamos a crear con la receta heat: [escenario1.yaml](https://fp.josedomingo.org/seguridadgs/u03/escenario1.yaml). La receta heat ha deshabilitado el cortafuego que nos ofrece openstack (todos los puertos de todos los protocolos están abiertos). La máquina creada tendrá un servidor web instalado. Vamos a trabajar con la red de las ips flotantes: 172.22.0.0/16.

![maquina](/img/firewall/1.png)

Entramos en nuestra máquina.

```bash
ssh debian@172.22.200.144
```

**Limpieza de las reglas previas:**

```bash
sudo su
iptables -F
iptables -t nat -F
iptables -Z
iptables -t nat -Z
```

**Vamos a permitir la conexión ssh:**

Como estamos conectados a la máquina por ssh, vamos a permitir la conexión ssh desde la red 172.22.0.0/16, antes de cambiar las políticas por defecto a DROP, para no perder la conexión:

```bash
iptables -A INPUT -s 172.22.0.0/16 -p tcp --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPT
iptables -A OUTPUT -d 172.22.0.0/16 -p tcp --sport 22 -m state --state ESTABLISHED -j ACCEPT
iptables -A INPUT -s 172.29.0.0/16 -p tcp --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPT
iptables -A OUTPUT -d 172.29.0.0/16 -p tcp --sport 22 -m state --state ESTABLISHED -j ACCEPT
```

**Política por defecto:**

Cambiamos a DROP para que no se permita la conexión a nada que no hayamos especificado anteriormente.

```bash
iptables -P INPUT DROP
iptables -P OUTPUT DROP
```

Comprobamos que el equipo no puede acceder a ningún servicio ni de Internet ni de la red local haciendo ping, ya que la política lo impide.

![drop](/img/firewall/2.png)

**Permitimos tráfico para la interfaz loopback:**

```bash
iptables -A INPUT -i lo -p icmp -j ACCEPT
iptables -A OUTPUT -o lo -p icmp -j ACCEPT
```

Comprobamos que podemos hacer ping a nuestra propia máquina pero no a internet.

![ping](/img/firewall/3.png)

**Peticiones y respuestas protocolo ICMP (para permitir la conexión a internet):**

```bash
iptables -A INPUT -i ens3 -p icmp --icmp-type echo-reply -j ACCEPT
iptables -A OUTPUT -o ens3 -p icmp --icmp-type echo-request -j ACCEPT
```

Comprobamos su funcionamiento haciendo ping a una IP pública:

![ping](/img/firewall/4.png)

**Consultas y respuestas DNS:**

```bash
iptables -A OUTPUT -o ens3 -p udp --dport 53 -m state --state NEW,ESTABLISHED -j ACCEPT
iptables -A INPUT -i ens3 -p udp --sport 53 -m state --state ESTABLISHED -j ACCEPT
```

Comprobamos su funcionamiento con una consulta DNS:

```bash
dig @1.1.1.1 www.josedomingo.org
```

![dns](/img/firewall/5.png)

**Tráfico http (que la máquina pueda navegar):**

```bash
iptables -A OUTPUT -o ens3 -p tcp --dport 80 -m state --state NEW,ESTABLISHED -j ACCEPT
iptables -A INPUT -i ens3 -p tcp --sport 80 -m state --state ESTABLISHED -j ACCEPT
```

Comprobamos que funciona accediendo a un servicio http.

```bash
curl http://portquiz.net:80
```

![http](/img/firewall/6.png)

**Tráfico https:**

```bash
iptables -A OUTPUT -o ens3 -p tcp --dport 443 -m state --state NEW,ESTABLISHED -j ACCEPT
iptables -A INPUT -i ens3 -p tcp --sport 443 -m state --state ESTABLISHED -j ACCEPT
```

Comprobamos que funciona abriendo un navegador y accediendo a cualquier sitio web (hoy en día la mayoría son https).

```bash
curl portquiz.net:443
```

![https](/img/firewall/7.png)

**Tráfico http/https:**

Podemos hacer un par de reglas que permitan el tráfico http/https (los dos puntos anteriores) usando la extensión multiport:

```
iptables -A OUTPUT -o ens3 -p tcp -m multiport --dports 80,443 -m state --state NEW,ESTABLISHED -j ACCEPT
iptables -A INPUT -i ens3 -p tcp -m multiport --sports 80,443 -m state --state ESTABLISHED -j ACCEPT
```

**Permitimos el acceso a nuestro servidor web:**

```bash
iptables -A INPUT -i ens3 -p tcp --dport 80 -m state --state NEW,ESTABLISHED -j ACCEPT
iptables -A OUTPUT -o ens3 -p tcp --sport 80 -m state --state ESTABLISHED -j ACCEPT
```

![web](/img/firewall/8.png)

**Configuración en un solo paso:**

Editamos un fichero y añadimos todas las reglas anteriores:

```bash
# Limpiamos las tablas
iptables -F
iptables -t nat -F
iptables -Z
iptables -t nat -Z
# Establecemos la política
iptables -P INPUT DROP
iptables -P OUTPUT DROP

iptables -A INPUT -i lo -p icmp -j ACCEPT
iptables -A OUTPUT -o lo -p icmp -j ACCEPT

iptables -A INPUT -s 172.22.0.0/16 -p tcp --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPT
iptables -A OUTPUT -d 172.22.0.0/16 -p tcp --sport 22 -m state --state ESTABLISHED -j ACCEPT

iptables -A INPUT -i ens3 -p icmp --icmp-type echo-reply -j ACCEPT
iptables -A OUTPUT -o ens3 -p icmp --icmp-type echo-request -j ACCEPT

iptables -A INPUT -i ens3 -p udp --sport 53 -m state --state ESTABLISHED -j ACCEPT
iptables -A OUTPUT -o ens3 -p udp --dport 53 -m state --state NEW,ESTABLISHED -j ACCEPT

iptables -A INPUT -i ens3 -p tcp --sport 80 -m state --state ESTABLISHED -j ACCEPT
iptables -A OUTPUT -o ens3 -p tcp --dport 80 -m state --state NEW,ESTABLISHED -j ACCEPT

iptables -A INPUT -i ens3 -p tcp --sport 443 -m state --state ESTABLISHED -j ACCEPT
iptables -A OUTPUT -o ens3 -p tcp --dport 443 -m state --state NEW,ESTABLISHED -j ACCEPT

iptables -A INPUT -i ens3 -p tcp --dport 80 -m state --state NEW,ESTABLISHED -j ACCEPT
iptables -A OUTPUT -o ens3 -p tcp --sport 80 -m state --state ESTABLISHED -j ACCEPT
```

## Ejercicios

### 1. Permite poder hacer conexiones ssh al exterior.

```bash
iptables -A OUTPUT -p tcp --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPT
iptables -A INPUT -p tcp --sport 22 -m state --state ESTABLISHED -j ACCEPT
```

Comprobamos que podemos hacer una conexión ssh.

![ssh](/img/firewall/9.png)

### 2. Deniega el acceso a tu servidor web desde una ip concreta.

Voy a denegar el acceso a mi web a una máquina que tengo creada en Openstack llamada **aislada** (a la que hicimos ssh en el ejercicio anterior).

Primero comprobamos las reglas que tenemos:

```bash
iptables -L -n -v --line-numbers
```

![reglas](/img/firewall/10.png)

La regla de acceso a nuestro puerto 80 es el número 7 (la marcada en rojo), por lo que al añadir la nueva regla para que la IP 172.22.201.239 no pueda acceder tenemos que añadirla delante de la regla marcada en rojo para que no se permita todo el tráfico.

```bash
iptables -I INPUT 7 -s 172.22.201.239 -p tcp --dport 80 -m state --state NEW,ESTABLISHED -j DROP
```

![reglas](/img/firewall/11.png)

Comprobamos que desde la máquina aislada no podemos acceder a la web pero desde otra máquina sí.

![comprobacion](/img/firewall/12.png)

### 3. Permite hacer consultas DNS sólo al servidor 192.168.202.2. Comprueba que no puedes hacer un dig @1.1.1.1.

```bash
iptables -L -n -v --line-numbers
```

![reglas](/img/firewall/13.png)

Borramos las reglas antiguas que permitían hacer consultas DNS a todo.

```bash
iptables -D OUTPUT -o ens3 -p udp --dport 53 -m state --state NEW,ESTABLISHED -j ACCEPT
iptables -D INPUT -i ens3 -p udp --sport 53 -m state --state ESTABLISHED -j ACCEPT
```

Añadimos las reglas que nos permite hacer consultas DNS al servidor 192.168.202.2.

```bash
iptables -A OUTPUT -d 192.168.202.2 -p udp --dport 53 -m state --state NEW,ESTABLISHED -j ACCEPT
iptables -A INPUT -s 192.168.202.2 -p udp --sport 53 -m state --state ESTABLISHED -j ACCEPT
```

Comprobamos que no podemos hacer una consulta DNS a @1.1.1.1, pero sí a @192.168.202.2.

```bash
dig @1.1.1.1 www.josedomingo.org
dig @192.168.202.2 www.josedomingo.org
```

![dns](/img/firewall/14.png)

![dns](/img/firewall/15.png)

### 4. No permitir el acceso al servidor web de www.josedomingo.org (Tienes que utilizar la ip). ¿Puedes acceder a fp.josedomingo.org?

En la última captura del ejercicio anterior vimos que la IP del servidor www.josedomingo.org es 37.187.119.60. La necesitaremos para crear la nueva regla, pero antes debemos comprobar las reglas que ya tenemos creadas del tráfico http y https.

![reglas](/img/firewall/16.png)

Añadimos la nueva regla delante de las reglas anteriores.

```bash
iptables -I OUTPUT 6 -d 37.187.119.60 -p tcp --dport 80 -m state --state NEW,ESTABLISHED -j DROP
```

Comprobamos que no tenemos acceso a www.josedomingo.org. Si lo intentamos desde otra máquina sí funcionará.

```bash
curl www.josedomingo.org
```

![noaccess](/img/firewall/17.png)

### 5. Permite mandar un correo usando nuestro servidor de correo: babuino-smtp. Para probarlo ejecuta un telnet bubuino-smtp.gonzalonazareno.org 25.

Permitimos el tráfico saliente del puerto 25 para poder enviar correos.

```bash
iptables -A OUTPUT -d 192.168.203.3 -p tcp --dport 25 -j ACCEPT
iptables -A INPUT -s 192.168.203.3 -p tcp --sport 25 -j ACCEPT
```

Comprobamos que nos podemos conectar mediante telnet:

```bash
telnet babuino-smtp.gonzalonazareno.org 25
```

![conexion](/img/firewall/18.png)

### 6. Instala un servidor mariadb, y permite los accesos desde la ip de tu cliente. Comprueba que desde otro cliente no se puede acceder.

Tras instalar el paquete mariadb-server vamos al fichero de configuración /etc/mysql/mariadb.conf.d/50-server.cnf, y modificamos el parámetro bind-address.

```bash
nano /etc/mysql/mariadb.conf.d/50-server.cnf

bind-address = 0.0.0.0
```

Reiniciamos el servicio.

```bash
systemctl restart mariadb
```

Añadimos las reglas iptables para permitir el acceso desde el cliente (máquina aislada).

```bash
iptables -A INPUT -s 172.22.201.239 -p tcp --dport 3306 -j ACCEPT
iptables -A OUTPUT -d 172.22.201.239 -p tcp --sport 3306 -j ACCEPT
```

Desde la máquina aislada, comprobamos que funciona correctamente:

```bash
nc -zvw10 172.22.200.144 3306
```

![netcat](/img/firewall/19.png)

Pero si lo intentamos desde otra máquina nos aparecerá lo siguiente:

![netcat](/img/firewall/20.png)
