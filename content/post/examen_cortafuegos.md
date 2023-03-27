---
title: "Examen Cortafuegos"
date: 2023-03-07T10:53:58+01:00
draft: true
tags: ["SAD"]
bigimg: [{src: "/img/triangle.jpg", desc: "Triangle"}, {src: "/img/sphere.jpg", desc: "Sphere"}, {src: "/img/hexagon.jpg", desc: "Hexagon"}]
---

# Creación máquinas

```txt
eat_template_version: newton

description: 1 router,  1 lan

parameters:
  flavor:
    type: string
    default: "m1.mini"
  image:
    type: string
    default: "6d992898-7e4f-44b9-a681-6dcf32d24a1f"
  red_externa:
    type: string
    description: red para conectarse a internet
    constraints:
      - custom_constraint: neutron.network

  key_name:
    type: string
    description: SSH key pair
    constraints:
      - custom_constraint: nova.keypair
  
resources:

  red_examen:
    type: OS::Neutron::Net
    properties:
      name: "Red_examen"
  
  subnet1:
    type: OS::Neutron::Subnet
    properties:
      network: { get_resource: red_examen }
      cidr: 192.168.50.0/24
      dns_nameservers: [192.168.202.2]
      gateway_ip: ""

  pc1_red_examen:
    type: OS::Neutron::Port
    properties:
      network: { get_resource: red_examen }
      port_security_enabled: False
      fixed_ips: [{"ip_address": "192.168.50.10", "subnet": { get_resource: subnet1 }}]
      security_groups: []
      
  r1_red_examen:
    type: OS::Neutron::Port
    properties:
      network: { get_resource: red_examen }
      port_security_enabled: False
      fixed_ips: [{"ip_address": "192.168.50.2", "subnet": { get_resource: subnet1 }}]
      security_groups: []
  
  
  r1_network_ext:
    type: OS::Neutron::Port
    properties:
      network: {get_param: red_externa}
      device_owner: "compute:nova"
      port_security_enabled: False
      security_groups: []
  
  r1:
    type: OS::Nova::Server
    properties:
      name: "FWSAD"
      flavor: { get_param: flavor }
      image: { get_param: image }
      key_name: { get_param: key_name }
      networks:
        - { port: { get_resource: r1_network_ext } }
        - { port: { get_resource: r1_red_examen } }
      user_data: |
          #!/bin/bash -v
          sudo ip r del default
          sudo ip r add default via 10.0.0.1
    
            
        

 
  r1_ip:
    type: OS::Neutron::FloatingIP
    properties:
      floating_network: ext-net
      port_id: { get_resource: r1_network_ext  }
  pc1:
    type: OS::Nova::Server
    properties:
      name: "pruebas"
      flavor: { get_param: flavor }
      image: { get_param: image }
      key_name: { get_param: key_name }
      networks:
        - { port: { get_resource: pc1_red_examen } }
      user_data: |
          #!/bin/bash -v
          sudo ip r add default via 192.168.50.2
  
outputs:
```

Tras configurar las máquinas y realizar los pasos previos vamos a realizar cada apartado.

# Apartado 1

FWSAD accesible por ssh por todas las maquinas de su red y poder establecer conexiones ssh a maquina pruebas.

Primero permitimos ssh al cortafuegos:

```bash
nft add rule inet filter input ip saddr 172.22.0.0/16 tcp dport 22 ct state new,established counter accept
nft add rule inet filter output ip daddr 172.22.0.0/16 tcp sport 22 ct state established counter accept
nft add rule inet filter input ip saddr 172.29.0.0/16 tcp dport 22 ct state new,established counter accept
nft add rule inet filter output ip daddr 172.29.0.0/16 tcp sport 22 ct state established counter accept
```

Permitimos ssh a la máquina pruebas.

```bash
nft add rule inet filter output oifname "ens4" ip daddr 192.168.50.0/24 tcp dport 22 ct state new,established counter accept
nft add rule inet filter input iifname "ens4" ip saddr 192.168.50.0/24 tcp sport 22 ct state established counter accept
```

Comprobación:

Desde FWSAD a pruebas:

![ssh](/img/examen/1.png)

Desde FWSAD a otra máquina (no debería funcionar):

![ssh](/img/examen/4.png)

Desde máquina pruebas a FWSAD:

![ssh](/img/examen/2.png)

Desde máquina externa a FWSAD:

![ssh](/img/examen/3.png)


# Apartado 2

Política por defecto DROP.

```bash
nft chain inet filter input { policy drop \; }
nft chain inet filter output { policy drop \; }
nft chain inet filter forward { policy drop \; }
```

Comprobamos, haciendo ping, que el equipo no puede acceder a servicios de Internet ni de la red local.

```bash
ping 8.8.8.8
ping 127.0.0.1
```

![ssh](/img/examen/5.png)

# Apartado 3

# Apartado 4

Instalado servidor Postgresl accesible desde máquina pruebas. Prueba de funcionamiento: realizar consulta desde cliente.

Tras instalar el servidor postgresql, creamos la regla nftables que dará acceso a la máquina pruebas al puerto de postgresql 5432.

```bash
nft add rule inet filter input ip saddr 192.168.50.10 tcp dport 5432 ct state new,established counter accept
```

# Apartado 5

Puede realizar ocnsultas DNS pero no al servidor 8.8.8.8.

```bash
nft add rule inet filter output ip daddr != 8.8.8.8 udp dport 53 ct state new,established counter accept
nft add rule inet filter input ip saddr != 8.8.8.8 udp sport 53 ct state established counter accept
```

Comprobamos que podemos hacer consultas DNS pero no al servidor 8.8.8.8.

```bash
dig @192.168.202.2 www.josedomingo.org
dig @8.8.8.8 www.josedomingo.org
```

![ssh](/img/examen/6.png)

![ssh](/img/examen/7.png)

# Apartado 6

Puede navegar usando HTTPS pero no HTTP.

```bash
nft add rule inet filter output oifname "ens3" ip protocol tcp tcp dport 443 ct state new,established counter accept
nft add rule inet filter input iifname "ens3" ip protocol tcp tcp sport 443 ct state established counter accept
```

Comprobamos que no obtenemos respuesta desde http pero sí desde https.

```bash
curl portquiz.net:80
curl portquiz.net:443
```

![ssh](/img/examen/8.png)

# Apartado 7

# Apartado 8

# Apartado 9

```bash
nft list ruleset > /etc/nftables.conf
systemctl enable nftables
systemctl start nftables
```

------------------------------------

He adjuntado el fichero con la lista de reglas a la entrega, pero las añado a continuación por si hubiera algún problema:

```txt
table inet filter {
	chain input {
		type filter hook input priority filter; policy drop;
		ip saddr 172.22.0.0/16 tcp dport 22 ct state established,new counter packets 0 bytes 0 accept
		ip saddr 172.29.0.0/16 tcp dport 22 ct state established,new counter packets 6602 bytes 503204 accept
		iifname "ens4" ip saddr 192.168.50.0/24 tcp sport 22 ct state established counter packets 864 bytes 103640 accept
		iifname "lo" counter packets 521 bytes 178816 accept
		iifname "ens3" ip protocol tcp tcp sport 443 ct state established counter packets 15 bytes 1434 accept
		ip saddr != 8.8.8.8 udp sport 53 ct state established counter packets 66 bytes 22842 accept
		iifname "ens4" ip saddr 192.168.50.0/24 tcp sport 22 ct state established counter packets 0 bytes 0 accept
		ip saddr 192.168.50.10 tcp dport 5432 ct state established,new counter packets 0 bytes 0 accept
	}

	chain output {
		type filter hook output priority filter; policy drop;
		ip daddr 172.22.0.0/16 tcp sport 22 ct state established counter packets 0 bytes 0 accept
		ip daddr 172.29.0.0/16 tcp sport 22 ct state established counter packets 4528 bytes 788095 accept
		oifname "ens4" ip daddr 192.168.50.0/24 tcp dport 22 ct state established,new counter packets 1333 bytes 98392 accept
		oifname "lo" counter packets 521 bytes 178816 accept
		oifname "ens3" ip protocol tcp tcp dport 443 ct state established,new counter packets 16 bytes 1096 accept
		ip daddr != 8.8.8.8 udp dport 53 ct state established,new counter packets 66 bytes 4528 accept
		oifname "ens4" ip daddr 192.168.50.0/24 tcp dport 22 ct state established,new counter packets 0 bytes 0 accept
	}

	chain forward {
		type filter hook forward priority filter; policy drop;
		iifname "ens4" oifname "ens3" ip saddr 192.168.50.0/24 udp dport 53 ct state established,new counter packets 4 bytes 263 accept
		iifname "ens3" oifname "ens4" ip daddr 192.168.50.0/24 udp sport 53 ct state established counter packets 0 bytes 0 accept
	}
}
table inet nat {
	chain prerouting {
		type nat hook prerouting priority filter; policy accept;
	}

	chain postrouting {
		type nat hook postrouting priority srcnat; policy accept;
	}
}
```

