---
title: "Cortafuegos II: Perimetral con nftables"
date: 2023-03-06T17:14:20+01:00
draft: true
tags: ["SAD"]
bigimg: [{src: "/img/triangle.jpg", desc: "Triangle"}, {src: "/img/sphere.jpg", desc: "Sphere"}, {src: "/img/hexagon.jpg", desc: "Hexagon"}]
---

Realiza con nftables el ejercicio de la página https://fp.josedomingo.org/seguridadgs/u03/perimetral_iptables.html documentando las pruebas de funcionamiento realizadas.

Debes añadir después las reglas necesarias para que se permitan las siguientes operaciones:

- a) Permite poder hacer conexiones ssh al exterior desde la máquina cortafuegos.
- b) Permite hacer consultas DNS desde la máquina cortafuegos sólo al servidor 192.168.202.2. Comprueba que no puedes hacer un dig @1.1.1.1.
- c) Permite que la máquina cortafuegos pueda navegar por internet.
- d) Los equipos de la red local deben poder tener conexión al exterior.
- e) Permitimos el ssh desde el cortafuego a la LAN
- f) Permitimos hacer ping desde la LAN a la máquina cortafuegos.
- g) Permite realizar conexiones ssh desde los equipos de la LAN
- h) Instala un servidor de correos en la máquina de la LAN. Permite el acceso desde el exterior y desde el cortafuego al servidor de correos. Para probarlo puedes ejecutar un telnet al puerto 25 tcp.
- i) Permite poder hacer conexiones ssh desde exterior a la LAN
- j) Modifica la regla anterior, para que al acceder desde el exterior por ssh tengamos que conectar al puerto 2222, aunque el servidor ssh este configurado para acceder por el puerto 22.
- k) Permite hacer consultas DNS desde la LAN sólo al servidor 192.168.202.2. Comprueba que no puedes hacer un dig @1.1.1.1.
- l) Permite que los equipos de la LAN puedan navegar por internet.

--------------------------------------

## Implementación de un cortafuegos perimetral con iptables

Vamos a realizar los primeros pasos para implementar un cortafuegos que protege la red interna.

**Esquema de red:**

Vamos a utilizar dos máquinas en openstack, que vamos a crear con la receta heat: [escenario2.yaml](https://fp.josedomingo.org/seguridadgs/u03/escenario2.yaml). La receta heat ha deshabilitado el cortafuego que nos ofrece openstack (todos los puertos de todos los protocolos están abiertos). Una máquina (que tiene asignada una IP flotante) hará de cortafuegos, y la otra será una máquina de la red interna 192.168.100.0/24.

![mvs](/img/firewall/21.png)

### LAN

No conectamos a la máquina **lan** y configuramos la ruta por defecto:

```bash
ssh -AJ debian@172.22.201.82 debian@192.168.100.10

sudo su
ip route del default
ip route add default via 192.168.100.2
```

### Router-fw

Si no tenemos instalado nftables lo instalamos.

```bash
sudo su
apt update
apt install nftables
```

A continuación, para crear las tablas usamos el comando `nft add table`. Para crear las cadenas el comando `nft add chain`. Y para las reglas `nft add rule`.

**Creo las tablas filter y nat:**

De esta forma quedan las reglas ordenadas según el esquema de red definido.

```bash
nft add table inet filter
nft add table inet nat
```

**Creo las cadenas de la tabla filter (input, output y forward):**

Estas se encargarán de filtrar el tráfico.

```bash
nft add chain inet filter input { type filter hook input priority 0 \; counter \; policy accept \; }
nft add chain inet filter output { type filter hook output priority 0 \; counter \; policy accept \; }
nft add chain inet filter forward { type filter hook forward priority 0 \; counter \; policy accept \; }
```

**Creo las cadenas de la tabla nat (prerouting y postrouting):**

Estas se encargarán de hacer SNAT y DNAT.

```bash
nft add chain inet nat prerouting { type nat hook prerouting priority 0 \; }
nft add chain inet nat postrouting { type nat hook postrouting priority 100 \; }
```

**Vamos a permitir ssh al cortafuegos:**

Como estamos conectados a la máquina por ssh, vamos a permitir la conexión ssh desde la red 172.22.0.0/16 para no perder la conexión.

```bash
nft add rule inet filter input ip saddr 172.22.0.0/16 tcp dport 22 ct state new,established counter accept
nft add rule inet filter output ip daddr 172.22.0.0/16 tcp sport 22 ct state established counter accept
nft add rule inet filter input ip saddr 172.29.0.0/16 tcp dport 22 ct state new,established counter accept
nft add rule inet filter output ip daddr 172.29.0.0/16 tcp sport 22 ct state established counter accept
```

**Política por defecto:**

Por defecto, la política de la tabla filter es **accept**. Para que no se permita la conexión a nada que no hayamos especificado la cambiamos a **drop**.

```bash
nft chain inet filter input { policy drop \; }
nft chain inet filter output { policy drop \; }
nft chain inet filter forward { policy drop \; }
```

Comprobamos, haciendo ping, que el equipo no puede acceder a ningún servicio ni de Internet ni de la red local, ya que la política lo impide.

```bash
ping -c 1 8.8.8.8
ping -c 1 127.0.0.1
```

![ping](/img/firewall/22.png)

**Activar el bit de forward:**

El primer paso común es habilitar el bit de forward mediante la instrucción:

```bash
echo 1 > /proc/sys/net/ipv4/ip_forward
```

Para habilitar el forward de forma permanente, habilitamos la línea **net.ipv4.ip_forward=1** del fichero **/etc/sysctl.conf** y ejecutando posteriormente `sysctl -p /etc/sysctl.conf`.

**SNAT:**

Hacemos SNAT para que los equipos de la LAN puedan acceder al exterior:

```bash
nft add rule inet nat postrouting oifname "ens3" ip saddr 192.168.100.0/24 counter masquerade
```

**Permitimos el ssh desde el cortafuego a la LAN (por el puerto 22):**

```bash
nft add rule inet filter output oifname "ens4" ip daddr 192.168.100.0/24 tcp dport 22 ct state new,established counter accept
nft add rule inet filter input iifname "ens4" ip saddr 192.168.100.0/24 tcp sport 22 ct state established counter accept
```

Comprobamos que nos podemos conectar a la máquina lan mediante ssh.

```bash
ssh debian@192.168.100.10
```

![ssh](/img/firewall/23.png)

**Permitimos tráfico para la interfaz loopback:**

```bash
nft add rule inet filter output oifname "lo" counter accept
nft add rule inet filter input iifname "lo" counter accept
```

Comprobamos que podemos hacer ping a nuestra propia máquina pero no a internet.

![ping](/img/firewall/24.png)

**Peticiones y respuestas protocolo ICMP:**

ICMP de entrada:

```bash
nft add rule inet filter input iifname "ens3" icmp type echo-request counter accept
nft add rule inet filter output oifname "ens3" icmp type echo-reply counter accept
```

Comprobamos que funciona haciendo ping desde mi máquina host llamada tars.

![ping](/img/firewall/25.png)

ICMP de salida hacia la LAN:

```bash
nft add rule inet filter output oifname "ens4" icmp type echo-request counter accept
nft add rule inet filter input iifname "ens4" icmp type echo-reply counter accept
```

Comprobamos que funciona haciendo ping desde la máquina que estoy configurando (router-fw) hacia la máquina lan.

![ping](/img/firewall/26.png)

**REGLAS FORWARD:**

Aunque ya hemos configurado SNAT, como hemos puesto la política por defecto FORWARD a DROP, los equipos de la LAN están incomunicadas, ya que no permitimos que ningún paquete pase por el cortafuego. Por lo tanto ahora tenemos que ir configurando los pares de reglas (forward en ambas direcciones) para permitir a la LAN acceder a internet.

**Para permitir hacer ping desde la LAN:**

Para que la LAN haga ping al exterior los paquetes ICMP tiene que estar permitidos que pasen por el cortafuego:

```bash
nft add rule inet filter forward iifname "ens4" oifname "ens3" ip saddr 192.168.100.0/24 icmp type echo-request counter accept
nft add rule inet filter forward iifname "ens3" oifname "ens4" ip daddr 192.168.100.0/24 icmp type echo-reply counter accept
```

Comprobamos:

![ping](/img/firewall/27.png)

**Para consultas y respuestas DNS desde la LAN:**

```bash
nft add rule inet filter forward iifname "ens4" oifname "ens3" ip saddr 192.168.100.0/24 udp dport 53 ct state new,established counter accept
nft add rule inet filter forward iifname "ens3" oifname "ens4" ip daddr 192.168.100.0/24 udp sport 53 ct state established counter accept
```

Comprobamos su funcionamiento con una consulta DNS desde la máquina lan:

```bash
dig @1.1.1.1 www.josedomingo.org
```

![dns](/img/firewall/31.png)

**Para permitir la navegación web desde la LAN:**

- Tráfico de salida:

```bash
nft add rule inet filter forward iifname "ens4" oifname "ens3" ip protocol tcp ip saddr 192.168.100.0/24 tcp dport { 80,443 } ct state new,established counter accept
nft add rule inet filter forward iifname "ens3" oifname "ens4" ip protocol tcp ip daddr 192.168.100.0/24 tcp sport { 80,443 } ct state established counter accept
```

Comprobamos que el tráfico http funciona:

```bash
curl portquiz.net:80
```

Comprobamos que el tráfico https funciona:

```bash
curl portquiz.net:443
```

![curl](/img/firewall/28.png)

- Tráfico entrante:

```bash
nft add rule inet filter forward iifname "ens3" oifname "ens4" ip daddr 192.168.100.0/24 tcp dport 80 ct state new,established counter accept
nft add rule inet filter forward iifname "ens4" oifname "ens3" ip saddr 192.168.100.0/24 tcp sport 80 ct state established counter accept
```

**Permitimos el acceso a nuestro servidor web de la LAN desde el exterior:**

En un primer momento tenemos que permitir que la consulta pase por el cortafuegos, para ello aplicamos una regla DNAT para que cuando se acceda al puerto 80 de la máquina router-fw, se redirija al puerto 80 de la máquina lan.

```bash
nft add rule inet nat prerouting iifname "ens3" tcp dport 80 counter dnat ip to 192.168.100.10
```

He instalado apache2 en la máquina lan y he creado una página de prueba a la que accederemos externamente para comprobar que funcionan las reglas creadas.

Primero desde mi host compruebo que funciona haciendo telnet a la dirección pública de la máquina router-fw, que redirigirá a la página web de la máquina lan.

![telnet](/img/firewall/29.png)

Desde un navegador comprobamos que funciona.

![web](/img/firewall/30.png)

**Hacemos persistentes las reglas creadas:**

Para que las reglas se mantengan tras reiniciar la máquina tenemos que hacer los cambios persistentes. Para ello guardamos las reglas en un fichero de configuración.

```bash
nft list ruleset > /etc/nftables.conf
systemctl enable nftables
systemctl start nftables
```

--------------------------------------

## Operaciones a realizar

**a) Permite poder hacer conexiones ssh al exterior desde la máquina cortafuegos**

```bash
nft add rule inet filter output oifname "ens3" tcp dport 22 ct state new,established counter accept
nft add rule inet filter input iifname "ens3" tcp sport 22 ct state established counter accept
```

Comprobamos:

```bash
ssh arantxa@172.29.0.90
```

![ssh](/img/firewall/32.png)

**b) Permite hacer consultas DNS desde la máquina cortafuegos sólo al servidor 192.168.202.2. Comprueba que no puedes hacer un dig @1.1.1.1.**

```bash
nft add rule inet filter output ip daddr 192.168.202.2 udp dport 53 ct state new,established counter accept
nft add rule inet filter input ip saddr 192.168.202.2 udp sport 53 ct state established counter accept
```

Comprobamos que podemos hacer consultas DNS al servidor 192.168.202.2 pero no a 1.1.1.1.

```bash
dig @192.168.202.2 www.josedomingo.org
dig @1.1.1.1 www.josedomingo.org
```

![dns](/img/firewall/34.png)

![dns](/img/firewall/35.png)

**c) Permite que la máquina cortafuegos pueda navegar por internet.**

```bash
nft add rule inet filter output oifname "ens3" ip protocol tcp tcp dport { 80,443 } ct state new,established counter accept
nft add rule inet filter input iifname "ens3" ip protocol tcp tcp sport { 80,443 } ct state established counter accept
```

Comprobamos para http y https:

```bash
curl portquiz.net:80
curl portquiz.net:443
```

![curl](/img/firewall/33.png)

**d) Los equipos de la red local deben poder tener conexión al exterior.**

Este paso ya lo realizamos anteriormente al crear las REGLAS FORWARD. Si hacemos ping desde la máquina lan podemos comprobar que tenemos conexión con el exterior.

![ping](/img/firewall/36.png)

**e) Permitimos el ssh desde el cortafuego a la LAN**

Este paso también se realizó anteriormente. Podemos comprobar que podemos hacer ssh desde la máquina router-fw a la máquina lan.

![ssh](/img/firewall/37.png)

**f) Permitimos hacer ping desde la LAN a la máquina cortafuegos.**

```bash
nft add rule inet filter input iifname "ens4" icmp type echo-request counter accept
nft add rule inet filter output oifname "ens4" icmp type echo-reply counter accept
```

Comprobamos:

![ping](/img/firewall/38.png)

**g) Permite realizar conexiones ssh desde los equipos de la LAN.**

```bash
nft add rule inet filter forward iifname "ens4" oifname "ens3" ip saddr 192.168.100.0/24 tcp dport 22 ct state new,established counter accept
nft add rule inet filter forward iifname "ens3" oifname "ens4" ip daddr 192.168.100.0/24 tcp sport 22 ct state established counter accept
```

Comprobamos:

![ping](/img/firewall/39.png)

**h) Instala un servidor de correos en la máquina de la LAN. Permite el acceso desde el exterior y desde el cortafuego al servidor de correos. Para probarlo puedes ejecutar un telnet al puerto 25 tcp.**

Tras instalar el servidor de correo en la máquina lan, creamos las reglas en router-fw.

```bash
nft add rule inet filter forward iifname "ens3" oifname "ens4" ip daddr 192.168.100.0/24 tcp dport 25 counter accept
nft add rule inet filter forward iifname "ens4" oifname "ens3" ip saddr 192.168.100.0/24 tcp sport 25 counter accept
nft add rule inet nat prerouting iifname "ens3" tcp dport 25 counter dnat ip to 192.168.100.10
```

Permitimos la conexión desde router-fw al servidor de correos de lan.

```bash
nft add rule inet filter output ip daddr 192.168.100.10 tcp dport 25 counter accept
nft add rule inet filter input ip saddr 192.168.100.10 tcp sport 25 counter accept
```

Comprobamos desde mi host.

![telnet](/img/firewall/40.png)

Comprobamos desde router-fw.

![telnet](/img/firewall/41.png)

**i) Permite poder hacer conexiones ssh desde el exterior a la LAN.**

```bash
nft add rule inet filter forward iifname "ens3" oifname "ens4" ip daddr 192.168.100.0/24 tcp dport 22 ct state new,established counter accept
nft add rule inet filter forward iifname "ens4" oifname "ens3" ip saddr 192.168.100.0/24 tcp sport 22 ct state established counter accept
nft add rule inet nat prerouting iifname "ens3" tcp dport 22 counter dnat ip to 192.168.100.10
```

Comprobamos:

![ssh](/img/firewall/42.png)

**j) Modifica la regla anterior, para que al acceder desde el exterior por ssh tengamos que conectar al puerto 2222, aunque el servidor ssh este configurado para acceder por el puerto 22.**

Cambiamos la regla DNAT:

```bash
nft add rule inet nat prerouting iifname "ens3" tcp dport 2222 counter dnat ip to 192.168.100.10:22
```

Cambiamos las reglas de filtrado para que se acepten las conexiones ssh en el puerto 2222:

```bash
nft add rule inet filter forward iifname "ens3" oifname "ens4" ip daddr 192.168.100.0/24 tcp dport 2222 ct state new,established counter accept
nft add rule inet filter forward iifname "ens4" oifname "ens3" ip saddr 192.168.100.0/24 tcp sport 2222 ct state established counter accept
```

Comprobamos:

![ssh](/img/firewall/43.png)

**k) Permite hacer consultas DNS desde la LAN sólo al servidor 192.168.202.2. Comprueba que no puedes hacer un dig @1.1.1.1.**

Comprobamos cuáles son las reglas que nos permite hacer consultas DNS a todos los servidores. Nos quedamos con el número de handle

```bash
nft -a list ruleset | egrep 53
```

![handles](/img/firewall/44.png)

*Aunque no estén señaladas en la imagen, los handle 22 y 23 también habría que eliminarlos.

Eliminamos esas reglas.

```bash
nft delete rule inet filter forward handle 22
nft delete rule inet filter forward handle 23
nft delete rule inet filter forward handle 46
nft delete rule inet filter forward handle 47
```

Y creamos las nuevas reglas que nos permitirán hacer consultas DNS al servidor 192.168.202.2.

```bash
nft add rule inet filter forward iifname "ens4" oifname "ens3" ip saddr 192.168.100.0/24 ip daddr 192.168.202.2 udp dport 53 ct state new,established counter accept
nft add rule inet filter forward iifname "ens3" oifname "ens4" ip saddr 192.168.202.2 ip daddr 192.168.100.0/24 udp sport 53 ct state established counter accept
```

Comprobamos desde la máquina lan.

```bash
dig @192.168.202.2 www.josedomingo.org
dig @1.1.1.1 www.josedomingo.org
```

![dns](/img/firewall/45.png)

![dns](/img/firewall/46.png)

**l) Permite que los equipos de la LAN puedan navegar por internet.**

```bash
nft add rule inet filter forward iifname "ens4" oifname "ens3" ip protocol tcp ip saddr 192.168.100.0/24 tcp dport { 80,443 } ct state new,established counter accept
nft add rule inet filter forward iifname "ens3" oifname "ens4" ip protocol tcp
```

Comprobamos:

![curl](/img/firewall/47.png)
