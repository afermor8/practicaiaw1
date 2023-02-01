---
title: "Vpn. Redes Privadas Virtuales"
date: 2023-01-24T18:15:29+01:00
draft: true
tags: ["SAD"]
bigimg: [{src: "/img/triangle.jpg", desc: "Triangle"}, {src: "/img/sphere.jpg", desc: "Sphere"}, {src: "/img/hexagon.jpg", desc: "Hexagon"}]
---

-------------------------

## VPN. Redes Privadas Virtuales

-------------------------

### A) VPN de acceso remoto con OpenVPN y certificados x509 (5 puntos)

Configura una conexión VPN de acceso remoto entre dos equipos del cloud:

- Uno de los dos equipos (el que actuará como servidor) estará conectado a dos redes 
- Para la autenticación de los extremos se usarán obligatoriamente certificados digitales, que se generarán utilizando openssl y se almacenarán en el directorio /etc/openvpn, junto con  los parámetros Diffie-Helman y el certificado de la propia Autoridad de Certificación. 
- Se utilizarán direcciones de la red 10.99.99.0/24 para las direcciones virtuales de la VPN. La dirección 10.99.99.1 se asignará al servidor VPN. 
- Los ficheros de configuración del servidor y del cliente se crearán en el directorio /etc/openvpn de cada máquina, y se llamarán servidor.conf y cliente.conf respectivamente. 
- Tras el establecimiento de la VPN, la máquina cliente debe ser capaz de acceder a una máquina que esté en la otra red a la que está conectado el servidor. 

Documenta el proceso detalladamente.

-------------------------

Para empezar he creado el escenario con vagrant. He creado tres máquinas: servidorvpn (192.168.0.3 y 192.168.22.2), cliente1 (192.168.0.2) y aislada (192.168.22.3). Cada uno tendrá una configuración que pasaré a comentar a continuación. El Vagrantfile es el siguiente:

```bash
Vagrant.configure("2") do |config|

config.vm.synced_folder ".", "/vagrant", disabled: true

  config.vm.provider :libvirt do |libvirt|
    libvirt.cpus = 1
    libvirt.memory = 512
  end

  config.vm.define :cliente1 do |cliente1|
    cliente1.vm.box = "debian/bullseye64"
    cliente1.vm.hostname = "cliente1"
    cliente1.vm.network :private_network,
      :libvirt__network_name => "externa",
      :libvirt__dhcp_enabled => false,
      :ip => "192.168.0.2",
      :libvirt__netmask => '255.255.255.0',
      :libvirt__forward_mode => "veryisolated"
  end

  config.vm.define :servidorvpn do |servidorvpn|
    servidorvpn.vm.box = "debian/bullseye64"
    servidorvpn.vm.hostname = "servidorvpn"
    servidorvpn.vm.network :private_network,
      :libvirt__network_name => "externa",
      :libvirt__dhcp_enabled => false,
      :ip => "192.168.0.3",
      :libvirt__netmask => '255.255.255.0',
      :libvirt__forward_mode => "veryisolated"
    servidorvpn.vm.network :private_network,
      :libvirt__network_name => "interna",
      :libvirt__dhcp_enabled => false,
      :ip => "192.168.22.2",
      :libvirt__netmask => '255.255.255.0',
      :libvirt__forward_mode => "veryisolated"
  end

  config.vm.define :aislada do |aislada|
    aislada.vm.box = "debian/bullseye64"
    aislada.vm.hostname = "aislada"
    aislada.vm.network :private_network,
      :libvirt__network_name => "interna",
      :libvirt__dhcp_enabled => false,
      :ip => "192.168.22.3",
      :libvirt__netmask => '255.255.255.0',
      :libvirt__forward_mode => "veryisolated"
  end

end
```

**En *servidorvpn*:**

Instalamos openvpn y activamos el bit de forwarding.

```bash
sudo apt update
sudo apt install openvpn

sudo nano /etc/sysctl.conf

net.ipv4.ip_forward=1
```

Ejecutar `sudo sysctl -p` para que se active el bit de forwarding.

Vamos a crear el certificado y las claves con Openssl mediante easy-rsa. Para ello vamos a trabajar en el directorio /etc/openvpn. Copiamos aquí easy-rsa.

```bash
sudo cp -r /usr/share/easy-rsa /etc/openvpn

cd /etc/openvpn/easy-rsa
```

Activamos la infraestructura con init-pki:

```bash
sudo ./easyrsa init-pki
```

Creamos el certificado de la **CA** y la clave privada.

```bash
sudo ./easyrsa build-ca
```

![instancias](/img/vpn/2.png)

El certificado generado **ca.crt** y la clave privada **ca.key** se encuentran en /etc/openvpn/easy-rsa/pki/ y pki/private/

Ahora creamos el certificado y la clave privada del **servidor VPN**.

```bash
sudo ./easyrsa build-server-full server nopass
```

![instancias](/img/vpn/3.png)

El certificado es **server.crt** y la clave privada **server.key** (en pki/issued/ y pki/private/).

Creamos los parámetros **Diffie-Helman** para que haya un intercambio seguro entre los nodos (esto tarda un poco en generarse).

```bash
sudo ./easyrsa gen-dh
```

Los parámetros **dh.pem** se crean en el directorio pki/.

A continuación genero el certificado y la clave privada del **cliente** (seguimos en la máquina servidorvpn), los cuales enviaremos mediante scp a la máquina cliente1.

```bash
sudo ./easyrsa build-client-full clientevpn nopass
```

![instancias](/img/vpn/4.png)

El certificado generado firmado por la clave privada de la CA se llama **clientevpn.crt** y la clave sería **clientevpn.key**.

Los enviamos a cliente1 junto con el certificado de la CA.

```bash
sudo cp -rp /etc/openvpn/easy-rsa/pki/{ca.crt,issued/clientevpn.crt,private/clientevpn.key} /home/vagrant/
cd
sudo chown vagrant:vagrant {ca.crt,clientevpn.crt,clientevpn.key}
scp {ca.crt,clientevpn.crt,clientevpn.key} vagrant@192.168.0.2:.
```

![instancias](/img/vpn/5.png)

Creo la configuración del tunel para el servidorvpn. Para ello primero copiamos la configuración por defecto de Openvpn y la modificamos según necesitemos. 

```bash
sudo cp /usr/share/doc/openvpn/examples/sample-config-files/server.conf /etc/openvpn/server/servidor.conf
sudo nano /etc/openvpn/server/servidor.conf
```

En **server** he puesto el rango de ip virtuales. La máquina servidor-vpn cogerá por defecto la primera, es decir 10.99.99.1. El documento de configuración quedaría de la siguiente forma:

```bash
port 1194

;proto tcp
proto udp

;dev tap
dev tun

;dev-node MyTap

ca /etc/openvpn/easy-rsa/pki/ca.crt
cert /etc/openvpn/easy-rsa/pki/issued/server.crt
key /etc/openvpn/easy-rsa/pki/private/server.key  # This file should be kept secret

dh /etc/openvpn/easy-rsa/pki/dh.pem

topology subnet

server 10.99.99.0 255.255.255.0

ifconfig-pool-persist /var/log/openvpn/ipp.txt

;server-bridge 10.8.0.4 255.255.255.0 10.8.0.50 10.8.0.100

;server-bridge

push "route 192.168.22.0 255.255.255.0"
;push "route 192.168.20.0 255.255.255.0"

;client-config-dir ccd
;route 192.168.40.128 255.255.255.248
;client-config-dir ccd
;route 10.9.0.0 255.255.255.252
;learn-address ./script
;push "redirect-gateway def1 bypass-dhcp"
;push "dhcp-option DNS 208.67.222.222"
;push "dhcp-option DNS 208.67.220.220"
;client-to-client
;duplicate-cn

keepalive 10 120

#tls-auth ta.key 0 # This file is secret

cipher AES-256-CBC

;compress lz4-v2
;push "compress lz4-v2"
;comp-lzo
;max-clients 100
;user nobody
;group nogroup

persist-key
persist-tun

status /var/log/openvpn/openvpn-status.log

;log         /var/log/openvpn/openvpn.log
;log-append  /var/log/openvpn/openvpn.log

verb 3

;mute 20

explicit-exit-notify 1

```

Iniciamos el servicio openvpn y comprobamos su funcionamiento:

```bash
sudo systemctl enable --now openvpn-server@servidor
sudo systemctl status openvpn-server@servidor
```

![instancias](/img/vpn/7.png)

**En *cliente1*:**

Instalamos openvpn y, a continuación, movemos los ficheros que recibimos anteriormente del servidor-vpn al directorio /etc/openvpn/client. Cambiamos el usuario de todos los ficheros a root.

```bash
sudo apt update
sudo apt install openvpn
sudo mv {ca.crt,clientevpn.crt,clientevpn.key} /etc/openvpn/client
sudo chown root: /etc/openvpn/client/*
```

Creamos la configuración de openvpn para nuestro cliente.

```bash
sudo cp /usr/share/doc/openvpn/examples/sample-config-files/client.conf /etc/openvpn/client/cliente.conf
sudo nano /etc/openvpn/client/cliente.conf
```

```bash
client

;dev tap
dev tun

;dev-node MyTap

;proto tcp
proto udp

remote 192.168.0.3 1194
;remote my-server-2 1194

;remote-random

resolv-retry infinite
nobind

;user nobody
;group nogroup

persist-key
persist-tun

;http-proxy-retry # retry on connection failures
;http-proxy [proxy server] [proxy port #]

;mute-replay-warnings

ca /etc/openvpn/client/ca.crt
cert /etc/openvpn/client/clientevpn.crt
key /etc/openvpn/client/clientevpn.key

remote-cert-tls server

#tls-auth ta.key 1

cipher AES-256-CBC

verb 3

;mute 20
```

Iniciamos el servicio openvpn y comprobamos su estado.

```bash
sudo systemctl enable --now openvpn-client@cliente
sudo systemctl status openvpn-client@cliente
```

![cliente](/img/vpn/8.png)

**En *aislada*:**

Cambiar la ruta por defecto:

```bash
sudo ip route del default
sudo ip route add default via 192.168.22.2
```

**Comprobaciones:**

Hacemos ping desde la máquina cliente1 al otro extremo de servidor-vpn (192.168.22.2). Y hacemos traceroute desde la máquina cliente1 al otro extremo de servidor-vpn (192.168.22.2).

![prueba](/img/vpn/prueba.png)

Hacemos ping y traceroute al otro extremo del tunel (10.99.99.1).

![prueba](/img/vpn/prueba2.png)

Hacemos ping y traceroute desde la máquina cliente1 a la máquina **aislada** (192.168.22.3).

![prueba](/img/vpn/prueba3.png)


-------------------------

### B) VPN sitio a sitio con OpenVPN y certificados x509 (10 puntos)

Configura una conexión VPN sitio a sitio entre dos equipos del cloud:

- Cada equipo estará conectado a dos redes, una de ellas en común.
- Para la autenticación de los extremos se usarán obligatoriamente certificados digitales, que se generarán utilizando openssl y se almacenarán en el directorio /etc/openvpn, junto con los parámetros Diffie-Helman y el certificado de la propia Autoridad de Certificación.
- Se utilizarán direcciones de la red 10.99.99.0/24 para las direcciones virtuales de la VPN.
- Tras el establecimiento de la VPN, una máquina de cada red detrás de cada servidor VPN debe ser capaz de acceder a una máquina del otro extremo.
Documenta el proceso detalladamente.

-------------------------

Primero voy a crear el escenario. Borro las máquinas anteriores con vagrant destroy y modifico el Vagrantfile, quedando de la siguiente forma:

```bash
Vagrant.configure("2") do |config|

config.vm.synced_folder ".", "/vagrant", disabled: true

  config.vm.provider :libvirt do |libvirt|
    libvirt.cpus = 1
    libvirt.memory = 512
  end

  config.vm.define :cliente1 do |cliente1|
    cliente1.vm.box = "debian/bullseye64"
    cliente1.vm.hostname = "cliente1"
    cliente1.vm.network :private_network,
      :libvirt__network_name => "privada1",
      :libvirt__dhcp_enabled => false,
      :ip => "192.168.0.2",
      :libvirt__netmask => '255.255.255.0',
      :libvirt__forward_mode => "veryisolated"
  end

  config.vm.define :clientevpn do |clientevpn|
    clientevpn.vm.box = "debian/bullseye64"
    clientevpn.vm.hostname = "clientevpn"
    clientevpn.vm.network :private_network,
      :libvirt__network_name => "privada1",
      :libvirt__dhcp_enabled => false,
      :ip => "192.168.0.1",
      :libvirt__netmask => '255.255.255.0',
      :libvirt__forward_mode => "veryisolated"
    clientevpn.vm.network :private_network,
      :libvirt__network_name => "externa",
      :libvirt__dhcp_enabled => false,
      :ip => "192.168.10.1",
      :libvirt__netmask => '255.255.255.0',
      :libvirt__forward_mode => "veryisolated"
  end

  config.vm.define :servervpn do |servervpn|
    servervpn.vm.box = "debian/bullseye64"
    servervpn.vm.hostname = "servervpn"
    servervpn.vm.network :private_network,
      :libvirt__network_name => "externa",
      :libvirt__dhcp_enabled => false,
      :ip => "192.168.10.2",
      :libvirt__netmask => '255.255.255.0',
      :libvirt__forward_mode => "veryisolated"
    servervpn.vm.network :private_network,
      :libvirt__network_name => "privada2",
      :libvirt__dhcp_enabled => false,
      :ip => "192.168.20.1",
      :libvirt__netmask => '255.255.255.0',
      :libvirt__forward_mode => "veryisolated"
  end

  config.vm.define :cliente2 do |cliente2|
    cliente2.vm.box = "debian/bullseye64"
    cliente2.vm.hostname = "cliente2"
    cliente2.vm.network :private_network,
      :libvirt__network_name => "privada2",
      :libvirt__dhcp_enabled => false,
      :ip => "192.168.20.2",
      :libvirt__netmask => '255.255.255.0',
      :libvirt__forward_mode => "veryisolated"
  end

end
```

Hago `vagrant up` y ya tendré mi nuevo escenario montado.

**En *servervpn*:**

Los primeros pasos serán iguales que en el anterior caso, por lo que no los explicaré:

```bash
sudo apt update
sudo apt install openvpn
sudo nano /etc/sysctl.conf

net.ipv4.ip_forward=1

sudo sysctl -p
sudo cp -r /usr/share/easy-rsa /etc/openvpn
cd /etc/openvpn/easy-rsa
sudo ./easyrsa init-pki
sudo ./easyrsa build-ca
sudo ./easyrsa build-server-full server nopass
sudo ./easyrsa gen-dh
sudo ./easyrsa build-client-full clientevpn nopass
cd
sudo cp -rp /etc/openvpn/easy-rsa/pki/{ca.crt,issued/clientevpn.crt,private/clientevpn.key} .
sudo chown vagrant:vagrant {ca.crt,clientevpn.crt,clientevpn.key}
scp {ca.crt,clientevpn.crt,clientevpn.key} vagrant@192.168.10.1:.
```

A partir de ahora empiezan las diferencias con respecto al caso anterior.

Creo el directorio ccd (client-config-dir), el cual contendrá un fichero llamado **clientevpn** que contendrá la otra red a la que se conecta clientevpn.

```bash
cd /etc/openvpn
sudo mkdir ccd
cd ccd
sudo nano clientevpn

iroute 192.168.0.0 255.255.255.0
```

Como en el apartado anterior voy a copiar el fichero de configuración de ejemplo de openvpn.

```bash
sudo cp /usr/share/doc/openvpn/examples/sample-config-files/server.conf /etc/openvpn/server/servidor.conf
```

Modifico el fichero anterior según mis necesidades. Quedaría de la siguiente forma:

```bash
sudo nano /etc/openvpn/server/servidor.conf
```

```bash
port 1194
proto udp
dev tun

ca /etc/openvpn/easy-rsa/pki/ca.crt
cert /etc/openvpn/easy-rsa/pki/issued/server.crt
key /etc/openvpn/easy-rsa/pki/private/server.key
dh /etc/openvpn/easy-rsa/pki/dh.pem

topology subnet

server 10.99.99.0 255.255.255.0
ifconfig-pool-persist /var/log/openvpn/ipp.txt

push "route 192.168.20.0 255.255.255.0"

client-config-dir /etc/openvpn/ccd
route 192.168.0.0 255.255.255.0

keepalive 10 120
cipher AES-256-CBC
persist-key
persist-tun
status /var/log/openvpn/openvpn-status.log
verb 3
explicit-exit-notify 1
```

Iniciamos el servidor y comprobamos que funcione.

```bash
sudo systemctl enable --now openvpn-server@servidor
sudo systemctl status openvpn-server@servidor
```

![prueba](/img/vpn/9.png)

**En *clientevpn*:**

Actualizamos e instalamos openvpn.

```bash
sudo apt update
sudo apt install openvpn
```

Activamos el bit de forwarding.

```bash
sudo nano /etc/sysctl.conf

net.ipv4.ip_forward=1

sudo sysctl -p
```

Movemos a /etc/openvpn/client los ficheros que habíamos pasado anteriormente desde el servidor.

```bash
sudo mv {ca.crt,clientevpn.crt,clientevpn.key} /etc/openvpn/client
sudo chown root:root /etc/openvpn/client/*
```

Y creamos el fichero de configuración para el cliente VPN.

```bash
sudo cp /usr/share/doc/openvpn/examples/sample-config-files/client.conf /etc/openvpn/client/cliente.conf

sudo nano /etc/openvpn/client/cliente.conf
```

```bash
client
dev tun
proto udp

remote 192.168.10.2 1194
resolv-retry infinite
nobind

persist-key
persist-tun

ca /etc/openvpn/client/ca.crt
cert /etc/openvpn/client/clientevpn.crt
key /etc/openvpn/client/clientevpn.key

remote-cert-tls server
cipher AES-256-CBC
verb 3
```

Habilitamos el servicio y comprobamos que funciona correctamente.

```bash
sudo systemctl enable --now openvpn-client@cliente
sudo systemctl status openvpn-client@cliente
```

![prueba](/img/vpn/10.png)

**En *cliente1*:**

```bash
sudo ip route del default
sudo ip route add default via 192.168.0.1
```

![prueba](/img/vpn/11.png)

**En *cliente2*:**

```bash
sudo ip route del default
sudo ip route add default via 192.168.20.1
```

![prueba](/img/vpn/12.png)

**Comprobación:**

Hacemos ping y traceroute desde clientevpn al otro extremo del túnel (10.99.99.1).

![prueba](/img/vpn/prueba5.png)

Hacemos ping y traceroute desde cliente1 a cliente2 (192.168.20.2).

![prueba](/img/vpn/prueba6.png)

Hacemos ping y traceroute desde cliente2 a cliente1 (192.168.0.2).

![prueba](/img/vpn/prueba7.png)


-------------------------

### C) VPN de acceso remoto con WireGuard (5 puntos)

Monta una VPN de acceso remoto usando Wireguard. Intenta probarla con clientes Windows, Linux y Android. Documenta el proceso adecuadamente y compáralo con el del apartado A.

-------------------------

#### Desde cliente Debian 11:

Primero creo el escenario para hacerlo desde un cliente Debian 11, que será el siguiente:

```bash
Vagrant.configure("2") do |config|

config.vm.synced_folder ".", "/vagrant", disabled: true

  config.vm.provider :libvirt do |libvirt|
    libvirt.cpus = 1
    libvirt.memory = 512
  end

  config.vm.define :clientelx do |clientelx|
    clientelx.vm.box = "debian/bullseye64"
    clientelx.vm.hostname = "clientelx"
    clientelx.vm.network :private_network,
      :libvirt__network_name => "redexterna",
      :libvirt__dhcp_enabled => false,
      :ip => "192.168.50.2",
      :libvirt__netmask => '255.255.255.0',
      :libvirt__forward_mode => "veryisolated"
  end

  config.vm.define :servervpn do |servervpn|
    servervpn.vm.box = "debian/bullseye64"
    servervpn.vm.hostname = "servervpn"
    servervpn.vm.network :private_network,
      :libvirt__network_name => "redexterna",
      :libvirt__dhcp_enabled => false,
      :ip => "192.168.50.1",
      :libvirt__netmask => '255.255.255.0',
      :libvirt__forward_mode => "veryisolated"
    servervpn.vm.network :private_network,
      :libvirt__network_name => "redinterna",
      :libvirt__dhcp_enabled => false,
      :ip => "192.168.80.1",
      :libvirt__netmask => '255.255.255.0',
      :libvirt__forward_mode => "veryisolated"
  end

  config.vm.define :aislada do |aislada|
    aislada.vm.box = "debian/bullseye64"
    aislada.vm.hostname = "aislada"
    aislada.vm.network :private_network,
      :libvirt__network_name => "redinterna",
      :libvirt__dhcp_enabled => false,
      :ip => "192.168.80.2",
      :libvirt__netmask => '255.255.255.0',
      :libvirt__forward_mode => "veryisolated"
  end

end
```

**En *servervpn*:**

Instalamos Wireguard.

```bash
sudo apt update
sudo apt install wireguard
```

En este caso no activaremos el bit de forwarding manualmente ya que lo haremos directamente desde el fichero de configuración de Wireguard.

Generamos el par de claves en **/etc/wireguard/**.

```bash
sudo su
cd /etc/wireguard/
wg genkey | tee serverprivatekey | wg pubkey > serverpublickey
```

Creamos el fichero de configuración (en la opción 'PrivateKey' debemos copiar la clave privada generada anteriormente).

```bash
cat serverprivatekey
nano wg0.conf
```

```bash
# Server config
[Interface]
Address = 10.99.99.1
PrivateKey = yG48lnvU/+blnbVOSmIDz5yyDRsGAFmCqLSxQRous2Q=
ListenPort = 51820
# IP forwarding
PreUp = sysctl -w net.ipv4.ip_forward=1

#iptables rules
PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -A FORWARD -o %i -j ACCEPT; iptables -t nat -A POSTROUTING -o wlp3s0 -j MASQUERADE
PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -D FORWARD -o %i -j ACCEPT; iptables -t nat -D POSTROUTING -o wlp3s0 -j MASQUERADE
```

Modificamos los permisos por seguridad.

```bash
chmod 600 /etc/wireguard/ -R
```

Avtivamos la interfaz creada y comprobamos que funciona.

```bash
wg-quick up wg0
wg
ip -br a
```

![prueba](/img/vpn/13.png)

**En *clientelx*:**

Instalo Wireguard y creo el par de claves en /etc/wireguard.

```bash
sudo apt update
sudo apt install wireguard
sudo su
cd /etc/wireguard
wg genkey | tee clientprivatekey | wg pubkey > clientpublickey
```

Creo el fichero de configuración.

```bash
cat clientprivatekey
cat clientpublickey 
nano wg0.conf

[Interface]
Address = 10.99.99.2
PrivateKey = 4HH9loDo2Bj90nYw4ZlKOGeWS0Bk9lLa/Aw/jSnuSFU=
ListenPort = 51820
PostUp = ip route add 192.168.80.0/24 dev wg0
PostDown = ip route del 192.168.80.0/24 dev wg0

[Peer]
PublicKey = p/Fg6cK6MPh76HWi7B0dj44ISBJjb8D263a8b5cSQ18=
AllowedIPs = 0.0.0.0/0
Endpoint = 192.168.50.1:51820
PersistentKeepalive = 25
```

En PrivateKey copio la clave privada del cliente con `cat clientprivatekey`.

En Postup y Postdown habrá que poner la red interna a la que nos conectaremos.

En Endpoint tendremos que poner la IP del servidor Wireguard (servervpn), el que configuramos en el punto anterior, y en PublicKey voy a copiar la **clave pública** de **servervpn** con `cat clientpublickey`.

Cambiamos los permisos.

```bash
chmod 600 /etc/wireguard/ -R
```

Ahora activamos la interfaz y comprobamos que funciona.

```bash
wg-quick up wg0
ip -br a
```

![prueba](/img/vpn/13.png)

Por último, añadimos **en servervpn** el bloque Peer del cliente que acabamos de configurar. Añadimos al fichero de configuración **wg0.conf** lo siguiente:

```bash
# Cliente Debian 11

[Peer]
Publickey = Xs2GEu0honQOS3clIjosaLUa7D0VwnrbSTIALr7cvk4=
AllowedIPs = 10.99.99.2/32
PersistentKeepAlive = 25
```

En Publickey hay que poner la clave pública del cliente.

Reiniciamos el servicio y comprobamos que funciona.

```bash
wg-quick down wg0
wg-quick up wg0
wg
ip r
```

![prueba](/img/vpn/15.png)

**En *aislada*:**

Cambiamos la ruta por defecto en la máquina aislada.

```bash
sudo ip r del default
sudo ip r add default via 192.168.80.1
```

![prueba](/img/vpn/17.png)

**Comprobación:**

Volvemos a la máquina **clientelx** y probamos que se puede hacer ping y traceroute a la máquina **aislada** de la red interna.

```bash
ping 192.168.80.2
traceroute 192.168.80.2
```

![prueba](/img/vpn/18.png)

Hacemos ping y traceroute desde **clientelx** al otro extremo del túnel.

```bash
ping 10.99.99.1
traceroute 10.99.99.1
```

![prueba](/img/vpn/20.png)

Hacemos ping y traceroute desde la máquina **aislada** de la red interna a la interfaz del túnel del cliente externo.

```bash
ping 10.99.99.2
traceroute 10.99.99.2
```

![prueba](/img/vpn/19.png)


#### Desde *Windows 10*:

Iniciamos la máquina y lo primero que haremos es instalar Wireguard desde la [página oficial](https://www.wireguard.com/install/).

![prueba](/img/vpn/24.png)

Ahora conectamos la máquina Windows 10 a la red **redexterna** creada anteriormente.

![prueba](/img/vpn/21.png)

En Símbolo del sistema (como administrador) configuramos la interfaz de forma estática.

```bash
netsh interface ip set address name="Ethernet" static 192.168.50.3 255.255.255.0 192.168.50.1
```

Compruebo la configuración y si tenemos conectividad.

![prueba](/img/vpn/22.png)
![prueba](/img/vpn/23.png)

Iniciamos el programa y creamos un túnel vacío.

![prueba](/img/vpn/25.png)

Se generan las claves de forma automática.

![prueba](/img/vpn/26.png)

Completamos la configuración.

![prueba](/img/vpn/27.png)

Ya nos aparecerá el tunel creado en el programa Wireguard.

![prueba](/img/vpn/28.png)

**En *servervpn*:**

A continuación vamos a la máquina servervpn y añadimos al fichero wg0.conf lo siguiente.

```bash
#Cliente Windows 10

[Peer]
PublicKey = p9xaV3uyUxqq89E79sgdj52AGcapJ+D4GqMJn/ZI31g=
AllowedIPs = 10.99.99.3/32
PersistentKeepAlive = 25
```

Reiniciamos wireguard.

```bash
wg-quick down wg0
wg-quick up wg0
wg
```

![prueba](/img/vpn/29.png)

**Volvemos al cliente Windows 10:**

Activamos el túnel que creamos anteriormente.

![prueba](/img/vpn/30.png)

![prueba](/img/vpn/31.png)

Comprobamos que se nos ha creado la interfaz del túnel.

![prueba](/img/vpn/32.png)

**Comprobación:**

Hacemos ping y tracert desde Windows al otro extremo del túnel:

![prueba](/img/vpn/.png)

Hacemos ping y tracert desde Windows a la máquina **aislada**.

![prueba](/img/vpn/.png)


#### Desde *Android*:

En este caso voy a utilizar mi teléfono móvil Android (192.168.1.130) conectado a mi máquina host (192.168.1.248) para crear un túnel y poderme conectar a una máquina llamada aislada (192.168.121.12) que está conectada a mi host (192.168.121.1).

Para hacer esto primero tendremos que configurar mi host con wireguard como hicimos en el primer caso.

Instalamos Wireguard y generamos el par de claves.

```bash
sudo apt install wireguard
sudo su
cd /etc/wireguard/
wg genkey | tee serverprivatekey | wg pubkey > serverpublickey
```

Creamos el fichero de configuración.

```bash
cat serverprivatekey
nano wg0.conf
```

```bash
# Server config
[Interface]
Address = 10.99.99.1
PrivateKey = mPGSoAZLQZg/B4Dqy3htpv4sMJgUIiZBSYaNandPA0A=
ListenPort = 51820
# IP forwarding
PreUp = sysctl -w net.ipv4.ip_forward=1

#iptables rules
PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -A FORWARD -o %i -j ACCEPT; iptables -t nat -A POSTROUTING -o wlp3s0 -j MASQUERADE
PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -D FORWARD -o %i -j ACCEPT; iptables -t nat -D POSTROUTING -o wlp3s0 -j MASQUERADE
```

Modificamos los permisos.

```bash
chmod 600 /etc/wireguard/ -R
```

Avtivamos la interfaz creada y comprobamos que funciona.

```bash
wg-quick up wg0
wg
ip -br a
```

![prueba](/img/vpn/35.png)

Vamos ahora al teléfono Android y descargamos Wireguard.

![prueba](/img/vpn/movil1.jpeg)

Para crear el túnel vamos a usar la opción de crear desde cero.

![prueba](/img/vpn/movil2.jpeg)

Generamos el par de claves y completamos la configuración.

![prueba](/img/vpn/movil3.jpeg)

![prueba](/img/vpn/movil4.jpeg)

![prueba](/img/vpn/movil5.jpeg)

![prueba](/img/vpn/movil6.jpeg)

En la captura anterior me faltó poner el puerto, pero en la siguiente se ve correctamente creado.

![prueba](/img/vpn/movil7.jpeg)

Vamos a nuestro host y modificamos el fichero de configuración wg0.conf. Añadimos lo siguiente:

```bash
# Cliente Android
[Peer]
PublicKey = bwYcPk16D5PLJL1yf82J/BiaS0aVTIt+fVUgUNrPkkE=
AllowedIPs = 10.99.99.2/32
```

Reiniciamos el servidor vpn.

```bash
wg-quick down wg0
wg-quick up wg0
wg
```

![prueba](/img/vpn/36.png)

Ahora sí vamos a nuestro teléfono y activamos el túnel creado.

![prueba](/img/vpn/movil8.jpeg)

Con Terminal Emulator comprobamos que se ha creado el túnel.

**Comprobación:**

Hacemos ping y traceroute al otro extremo del túnel (10.99.99.1).

![prueba](/img/vpn/.png)

Hacemos ping y traceroute a la máquina aislada (192.168.121.12).

![prueba](/img/vpn/.png)

-------------------------

### D) VPN sitio a sitio con WireGuard (10 puntos)

Configura una VPN sitio a sitio usando WireGuard. Documenta el proceso adecuadamente y compáralo con el del apartado B.




-------------------------

### Extra 1) VPN de acceso remoto con Ipsec (5 puntos)

Elige una aplicación por software (por ejemplo, FreeS/Wan) y monta la configuración. Documenta el proceso detalladamente.



-------------------------

### Extra 2) VPN sitio a sitio con IPsec (10 puntos)

Montando el escenario en GNS3 usando routers CISCO o con una aplicación por software (por ejemplo, FreeS/Wan) despliega la configuración solicitada. Documenta el proceso detalladamente.