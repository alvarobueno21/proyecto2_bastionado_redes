author: Grupo 4
summary: Proyecto 2
id: grupo4
categories: codelab,markdown
environments: Web
status: Published

# Packet Tracer - Enrutamiento de Red y Configuración VLANs

# **Introducción**

Esta guía detalla el proceso de configuración de una red de área local (LAN) utilizando tecnologías de VLAN y enrutamiento realizando una configuración router on a stick.

La red consta de un router Cisco ISR4331 conectado a un switch Cisco 2960 al cual se conectan tres switches Cisco 2960. Estos dispositivos están interconectados formando una estructura que permite la segmentación de la red en diferentes subredes, cada una representada por una VLAN diferente. El router actúa como el dispositivo de enrutamiento para el tráfico entre estas VLANs, ya que este está configurado como router on a stick.

El principal objetivo es configurar tres VLANs distintas para organizar los 150 dispositivos finales (PCs, impresoras y teléfonos) entre las distintas subredes, cada uno conectado a su propio switch. 

# **Preparación de la Red**

Listado de Equipos y Dispositivos Necesarios

- 1 Router Cisco ISR4331

- 3 Switches Cisco 2960

- 150 PCs o dispositivos finales

- Una Impresora para cada uno de los PCs

**Descripción de la Topología Física y Lógica:**

Los switches están conectados al router mediante un switch que hace de mode trunk. Cada switch de cada subred alojará una VLAN individual que corresponde a un segmento de red diferente. En cada subred encontramos ordenadores, impresoras y teléfonos. El router está configurado con subinterfaces para manejar el tráfico entre estas VLANs a través de una única interfaz física.

**Requisitos de Direccionamiento IP y Planificación de VLSM**

Mediante VLSM se asignarán de manera eficiente las direcciones IP a cada VLAN, maximizando el uso del espacio de direcciones disponible. Cada VLAN recibirá un bloque de 64 direcciones de basado en la cantidad de hosts que se esperan en cada segmento, sabemos que actualmente solo hay 50 equipos en cada subred, pero está previsto que pronto se incorporen nuevos trabajadores por ellos será suficiente con 64 direcciones para cada subred, ya que quedan libre 12 direcciones asignables.

# **Configuración de VLANs en los Switches**

## **1. Definir VLANs**

Primero, decidimos cuántas VLANs necesitamos y como denominaremos a cada una de ellas.

**VLAN 10:** Departamento de Administración A departamento A

**VLAN 20:** Departamento de Ventas A departamento B

**VLAN 30:** Departamento de IT A departamento C

## **2. Planificar el Esquema de Direccionamiento (VLSM)**

Utilizaremos el esquema de direccionamiento VLSM para asignar bloques de direcciones IP a cada VLAN. Esto te permite utilizar el espacio de direcciones IP de manera eficiente.

Realizamos el siguiente esquema de direccionamiento con la dirección red: 192.168.1.0 /24 para 3 subredes, y quedaría de la siguiente manera el reparto de direcciones:

<img src="img/vlsm.png" />

## **3. Configurar VLANs en los Switches**

Para crear VLANs en cada switch planteadas anteriormente, utilizaremos los siguente comandos en cada uno de los switches de cada una de las tres subredes:

**Swtich 1:**
`````
vlan 10

name VLAN10

exit
`````
**Swtich 2:**
`````
vlan 20

name VLAN20

exit
`````

**Swtich 3:**
`````
vlan 30

name VLAN30

exit
`````

## **4. Asignar Puertos a VLANs**

**Swtich 1:**
`````
interface range fa0/1 - 50

switchport mode access

switchport access vlan 10

exit
`````

**Swtich 2:**
`````
interface range fa0/1 - 50

switchport mode access

switchport access vlan 20

exit
`````
**Swtich 3:**
`````
interface range fa0/1 - 50

switchport mode access

switchport access vlan 20

exit
`````
## **5. Configuración de Trunk en los Switches**

En cada switch, identificamos el puerto que conecta al switch central y así configurar el modo trunk en el puerto de salida de cada switch de cada subred.

**Swtich 1:**
`````
interface GigabitEthernet0/1

switchport mode trunk

switchport trunk allowed vlan 10

exit
`````
**Swtich 2:**
`````
interface GigabitEthernet0/1

switchport mode trunk

switchport trunk allowed vlan 20

exit
`````
**Swtich 3:**
`````
interface GigabitEthernet0/1

switchport mode trunk

switchport trunk allowed vlan 30

exit
`````
## **6. Configuración del Switch Central (Switch Trunk)**

**6.1. Crear las VLANs en el Switch Central:** 

Aunque este switch actuará como un enlace troncal, aun así necesita tener las VLANs definidas para que pueda entender y direccionar el tráfico de VLANs apropiadamente.
`````
vlan 10 

name departamentoA 

exit 
`````
`````
vlan 20 

name departamentoB 

exit 
`````
`````
vlan 30 

name departamentoC

exit 
`````
**6.2. Configurar los puertos que se conectan a los switches de acceso como Trunk:** Los puertos Fa0/1, Fa0/2, Fa0/3 del switch central, se conectan al switch de cada subred. Por tanto, estableceremos los siguientes comandos: 
`````
interface range GigabitEthernet0/1 - 3 

switchport trunk encapsulation dot1q 

switchport mode trunk 

switchport trunk allowed vlan 10

switchport trunk allowed vlan 20

switchport trunk allowed vlan 30 

exit 
`````
**6.3.  Configurar el puerto que se conecta al router como Trunk:** 

El puerto GigabitEthernet0/1 del switch se conecta al router, este también será necesario configurarlo como Trunk.
`````
interface GigabitEthernet0/0 

switchport trunk encapsulation dot1q 

switchport mode trunk 

switchport trunk allowed vlan 10

switchport trunk allowed vlan 20

switchport trunk allowed vlan 30 

exit 
`````
## **7. Configuración Router-On-A-Stick**
`````
interface gigabitethernet 0/0/0

no shutdown

interface GigabitEthernet0/0.10

encapsulation dot1Q 10

ip address 192.168.1.1 255.255.255.192

no shutdown

exit
`````
`````
interface GigabitEthernet0/0.20

encapsulation dot1Q 20

ip address 192.168.1.65 255.255.255.192

no shutdown

exit
`````
`````
interface GigabitEthernet0/0.30

encapsulation dot1Q 30

ip address 192.168.1.129 255.255.255.192

no shutdown

exit
`````

