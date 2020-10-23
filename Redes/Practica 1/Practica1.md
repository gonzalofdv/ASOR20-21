# Práctica 1.1. Protocolo IPv4. Servicio DHCP
## Objetivos
esta práctica se presentan las herramientas que se utilizarán en la asignatura y se repasan brevemente los aspectos básicos del protocolo IPv4. Además, se analizan las características del protocolo DHCP.

## Preparación del entorno para la práctica
Configuraremos la topología de red que se muestra en la siguiente figura:

Todos los elementos -el router y las máquinas virtuales VM- son clones enlazados de la máquina base ASOR-FE. 

## Configuración estática
En primer lugar, configuraremos cada red de forma estática asignando a cada máquina una dirección IP adecuada.

**Ejercicio 1 [VM1].** Determinar los interfaces de red que tiene la máquina y las direcciones IP y MAC que tienen asignadas. Utilizar el comando `ip`.


    ip addr

**Ejercicio 2 [VM1, VM2, Router].** Activar los interfaces eth0 en VM1, VM2 y Router, y asignar una dirección IP adecuada. La configuración debe realizarse con la utilidad `ip`, en particular los comandos `ip address` e `ip link`.


    VM1: sudo ip addr add 192.168.0.1/24 broadcast 192.168.0.255 dev eth0
         sudo ip link set eth0 up
    VM2: sudo ip addr add 192.168.0.2/24 broadcast 192.168.0.255 dev eth0
         sudo ip link set eth0 up
    Router: sudo ip addr add 192.168.0.3/24 broadcast 192.168.0.255 dev eth0
            sudo ip link set eth0 up
            sudo ip addr add 172.16.0.2/16 broadcast 172.16.255.255  dev eth1
            sudo ip link set eth1 up

**Ejercicio 3 [VM1, VM2].** Abrir la herramienta Wireshark en VM1 e iniciar una captura en el interfaz de red. Comprobar la conectividad entre VM1 y VM2 con la orden `ping`. Observar el tráfico generado, especialmente los protocolos encapsulados en cada datagrama y las direcciones origen y destino. Para ver correctamente el tráfico ARP, puede ser necesario eliminar la tabla ARP en VM1 con la orden `ip neigh flush dev eth0`.

Completar la siguiente tabla para todos los mensajes intercambiados hasta la recepción de la primera respuesta Echo Reply:

- Anotar las direcciones MAC e IP de los mensajes.
- Para cada protocolo, anotar las características importantes (p. ej. pregunta/respuesta ARP o tipo ICMP) en el campo “Tipo de mensaje”.
- Comparar los datos observados durante la captura con el formato de los mensajes estudiados en clase.

| **MAC Origen**    | **MAC Destino**   | **Protocolo** | **IP Origen** | **IP Destino** | **Tipo Mensaje**                 |
| ----------------- | ----------------- | ------------- | ------------- | -------------- | -------------------------------- |
| 08:00:27:2a:9b:16 | ff:ff:ff:ff:ff:ff | ARP           | 192.168.0.1   | 192.168.0.255  | Pregunta ARP                     |
| 08:00:27:79:56:e5 | 08:00:27:2a:9b:16 | ARP           | 192.168.0.2   | 192.168.0.1    | Respuesta ARP                    |
| 08:00:27:2a:9b:16 | 08:00:27:79:56:e5 | ICMP          | 192.168.0.1   | 192.168.0.2    | ECHO_REQUEST                     |
| 08:00:27:79:56:e5 | 08:00:27:2a:9b:16 | ICMP          | 192.168.0.2   | 192.168.0.1    | ECHO_REPLY                       |


Adjuntar una captura de pantalla de Wireshark con los mensajes ICMP y ARP




**Ejercicio 4 [VM1, VM2].** Ejecutar de nuevo la orden ping entre VM1 y VM2 y, a continuación, comprobar el estado de la tabla ARP en VM1 y VM2 usando el comando `ip neigh`. El significado del estado de cada entrada de la tabla se puede consultar en la página de manual del comando.
Adjuntar la salida del comando `ip neigh` y describir el estado de cada entrada



    192.168.0.2 dev eth0 lladdr 08:00:27:79:56:e5 REACHABLE
    La dirección ip 192.168.0.2 es alcanzable desde la maquina VM1

**Ejercicio 5 [Router, VM4].** Configurar Router y VM4 y comprobar su conectividad con el comando `ping`.
Adjuntar los comandos utilizados y la salida del comando ping

    Router: especificado en ejercicio 2
    VM4: sudo ip addr add 172.16.0.1/16 broadcast 172.16.255.255 dev eth0
         sudo ip link set eth0 up
    ping 172.16.0.2



## Encaminamiento estático

Según la topología de esta práctica, Router puede encaminar el tráfico entre ambas redes. En esta sección, vamos a configurar el encaminamiento estático, basado en rutas que fijaremos manualmente en todas las máquinas virtuales.

**Ejercicio 6 [Router].** Activar el reenvío de paquetes (forwarding) en Router para que efectivamente pueda funcionar como encaminador entre las redes. Ejecutar el siguiente comando:

    $ sudo sysctl net.ipv4.ip_forward=1

**Ejercicio 7 [VM1, VM2].** Añadir Router como encaminador por defecto para VM1 y VM2. Usar el comando `ip route`.
Adjuntar el comando utilizado

    sudo ip route add default via 192.168.0.3



**Ejercicio 8 [VM4].** Aunque la configuración adecuada para la tabla de rutas en redes como las consideradas en esta práctica consiste en añadir una ruta por defecto, es posible incluir rutas para redes concretas. Añadir en VM4 una ruta a la red 192.168.0.0/24 via Router. Usar el comando `ip route`.
Adjuntar el comando utilizado

    sudo ip route add 192.168.0.0/24  via 172.16.0.2 dev eth0

**Ejercicio 9 [VM1, VM4, Router].** Abrir la herramienta Wireshark en Router e iniciar una captura en sus dos interfaces de red. Eliminar la tabla ARP en VM1 y Router. Usar la orden ping entre VM1 y VM4. Completar la siguiente tabla para todos los paquetes intercambiados hasta la recepción del primer Echo Reply.

***Red 192.168.0.0/24 - Router (eth0)***

| **MAC Origen**    | **MAC Destino**   | **Protocolo** | **IP Origen** | **IP Destino** | **Tipo Mensaje**                 |
| ----------------- | ----------------- | ------------- | ------------- | -------------- | -------------------------------- |
| 08:00:27:2a:9b:16 | ff:ff:ff:ff:ff:ff | ARP           | 192.168.0.1   | 192.168.0.255  | Pregunta ARP                     |
| 08:00:27:1a:ae:3d | 08:00:27:2a:9b:16 | ARP           | 192.168.0.3   | 192.168.0.1    | Respuesta ARP                    |
| 08:00:27:2a:9b:16 | 08:00:27:1a:ae:3d | ICMP          | 192.168.0.1   | 192.168.0.3    | ECHO_REQUEST                     |
| 08:00:27:1a:ae:3d | 08:00:27:2a:9b:16 | ICMP          | 192.168.0.3   | 192.168.0.1    | ECHO_REPLY                       |


***Red 172.16.0.0/16 - Router (eth1)***

| **MAC Origen**    | **MAC Destino**   | **Protocolo** | **IP Origen** | **IP Destino** | **Tipo Mensaje**                 |
| ----------------- | ----------------- | ------------- | ------------- | -------------- | -------------------------------- |
| 08:00:27:d4:4a:6a | ff:ff:ff:ff:ff:ff | ARP           | 172.16.0.2    | 172.16.255.255 | Pregunta ARP                     |
| 08:00:27:9c:52:23 | 08:00:27:d4:4a:6a | ARP           | 172.16.0.1    | 172.16.0.2     | Respuesta ARP                    |
| 08:00:27:d4:4a:6a | 08:00:27:9c:52:23 | ICMP          | 192.168.0.1   | 172.16.0.1     | ECHO_REQUEST                     |
| 08:00:27:9c:52:23 | 08:00:27:d4:4a:6a | ICMP          | 172.16.0.1    | 192.168.0.1    | ECHO_REPLY                       |


Adjuntar una captura de pantalla de Wireshark con los mensajes ICMP y ARP.




## Configuración dinámica
El protocolo DHCP permite configurar dinámicamente los parámetros de red de una máquina. En esta sección configuraremos Router como servidor DHCP para las dos redes. Aunque DHCP puede incluir muchos parámetros de configuración, en esta práctica sólo fijaremos el encaminador por defecto.

***Ejercicio 10 [VM1, VM2, VM4].*** Eliminar las direcciones IP de los interfaces `(ip addr del)` de todas las máquinas salvo Router.

***Ejercicio 11 [Router].*** Configurar el servidor DHCP para las dos redes:

- Editar el fichero /etc/dhcp/dhcpd.conf y añadir dos secciones subnet, una para cada red, que definan los rangos de direcciones, 192.168.0.50-192.168.0.100 y 172.16.0.50-172.16.0.100, respectivamente. Además, incluir la opción `routers` con la dirección IP de Router en cada red. Ejemplo:

      subnet 192.168.0.0 netmask 255.255.255.0 {
        range 192.168.0.11 192.168.0.50;
        option routers 192.168.0.3;
        option broadcast-address 192.168.0.255;
      }
    
- Arrancar el servicio con el comando `service dhcpd start`.

***Ejercicio 12 [Router, VM1].*** Iniciar una captura de paquetes en Router. Arrancar el cliente DHCP en VM1 con `dhclient -d eth0` y observar el proceso de configuración. Completar la siguiente tabla:

| **IP Origen** | **IP Destino**  | **Mensaje DHCP** | **Opciones DHCP**        |
| ------------- | --------------- | ---------------- | ------------------------ |
|               | 255.255.255.255 | Discover         |                          |
|               |                 | Offer            |                          |
|               | 255.255.255.255 | Request          |                          |
|               |                 | ACK              |                          |



Adjuntar la salida del comando dhclient una captura de pantalla de Wireshark con los mensajes DHCP. 

Tras diferentes pruebas y preguntar a algún compañero, no he conseguido que funcionase correctamente la configuración dinámica, aún así, adjunto los mensajes que me han aparecido al ejecutar los comandos (en teoría he realizado todos los pasos correctamente)



***Ejercicio 13 [VM4].*** Durante el arranque del sistema se pueden configurar automáticamente interfaces según la información almacenada en el disco del servidor (configuración persistente). Consultar el fichero `/etc/sysconfig/network-scripts/ifcfg-eth0` de VM4, que configura automáticamente eth0 usando DHCP. Para configuración estática, se usarían las siguientes opciones:
    
    TYPE=Ethernet
    BOOTPROTO=none
    IPADDR=<dirección IP estática en formato CIDR>
    GATEWAY=<dirección IP estática del encaminador por defecto (si existe)>
    DEVICE=eth0
    
***Nota:*** Estas opciones se describen en detalle en `/usr/share/doc/initscripts-*/sysconfig.txt.`

***Ejercicio 14 [VM4].*** Comprobar la configuración persistente con las órdenes ifup e ifdown. Verificar la conectividad entre todas las máquinas de las dos redes.
