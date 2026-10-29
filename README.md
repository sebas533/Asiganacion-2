Práctica 2: AAA con RADIUS (NPS) en Router y Switch Cisco
Autenticación, autorización y registro (AAA) centralizados con RADIUS sobre Windows Server 2022 (Active Directory y NPS). El direccionamiento está basado en la matrícula 2168.
---
1. Objetivo
Configurar un router y un switch Cisco para que el acceso por consola y auxiliar se autentique y autorice contra un servidor RADIUS (NPS), que asigna el nivel de privilegio según el grupo de Active Directory del usuario. El acceso por SSH usa un usuario local, que también sirve de respaldo si RADIUS no responde.
Requisitos cumplidos en el router y en el switch:
Requisito	Estado
Seguridad de autenticación y autorización	Cumplido
RADIUS configurado como cliente	Cumplido
Usuario local	Cumplido
Autenticación y autorización solo en consola y auxiliar	Cumplido
Logs de autenticación, autorización y RADIUS	Cumplido
Servicio SSH	Cumplido
---
2. Topología
Los equipos son:
Equipo en GNS3	Rol
`R1`	Router, cliente RADIUS y servidor SSH
`Switch`	Switch IOSvL2, cliente RADIUS y servidor SSH
`Cloud-WS`	Enlace hacia la VM Windows Server (VMware, VMnet2)
`SRV-AD`	Windows Server 2022 con Active Directory y NPS
`UbuntuSv`	Cliente para las pruebas de SSH
Conexiones:
```
                 R1 (Fa0/0)
                     |
                  (Gi0/0)
Cloud-WS --(Gi0/1)-- Switch --(Gi0/2)-- UbuntuSv
   |
SRV-AD (Windows Server)
```
![Topología](imagenes/Topologia.png)
---
3. Direccionamiento IP
El direccionamiento está basado en mi matrícula, 2168: 21.68 forma la red `10.21.68.0/24`.
Red	Máscara	Gateway
`10.21.68.0/24`	255.255.255.0	`10.21.68.1`

Equipo	Interfaz	Dirección
R1	Fa0/0 (hacia Switch)	`10.21.68.1/24`
Switch	Gi0/0 (hacia R1)	sin IP
Switch	Gi0/1 (hacia servidor)	sin IP
Switch	Gi0/2 (hacia UbuntuSv)	sin IP
Switch	VLAN 1 (administración)	`10.21.68.2/24`
SRV-AD	Ethernet1 (VMnet2)	`10.21.68.10/24`
UbuntuSv	ens3	`10.21.68.100/24`
---
4. Servidor Windows (Active Directory y NPS)
4.1 Active Directory
Dominio: `lab.local`. Un grupo por cada nivel de privilegio, con un usuario cada uno.
Grupo	Usuario	Nivel
`Nivel0`	`user0`	0
`Nivel5`	`user5`	5
`Nivel10`	`user10`	10
4.2 Network Policy Server (NPS)
Clientes RADIUS:
Nombre	Dirección	Puertos
`R1`	`10.21.68.1`	UDP 1812 (autenticación) y 1813 (accounting)
`SW1`	`10.21.68.2`	UDP 1812 (autenticación) y 1813 (accounting)
Políticas de red. Autenticación PAP, `Service-Type = Login` y atributo `Cisco-AV-Pair` por política:
Orden	Política	Condición (grupo)	Cisco-AV-Pair
1	`Nivel-10`	`LAB\\\\\\\\Nivel10`	`shell:priv-lvl=10`
2	`Nivel-5`	`LAB\\\\\\\\Nivel5`	`shell:priv-lvl=5`
3	`Nivel-0`	`LAB\\\\\\\\Nivel0`	`shell:priv-lvl=0`
Las tres políticas se procesan antes de las dos políticas por defecto de "Denegar acceso"; de lo contrario NPS rechaza a los usuarios.
![Políticas NPS](imagenes/Politicas.png)
---
5. Router y Switch
La configuración es equivalente en ambos equipos; solo cambian el nombre y la interfaz con IP.
Requisito	Implementación
Seguridad AAA	`aaa new-model`, `aaa authentication login`, `aaa authorization exec`
RADIUS como cliente	`radius server NPS` hacia `10.21.68.10`
Usuario local	`username admin privilege 15 secret ...`
Solo consola y auxiliar	Lista `CON-AUX` (RADIUS y luego local) solo en `line con 0` y `line aux 0`. Las líneas VTY usan la lista local
Logs	`login on-success log`, `login on-failure log`, `aaa accounting` hacia RADIUS, `logging buffered`
Servicio SSH	`ip domain-name`, llaves RSA 2048, `ip ssh version 2`, `transport input ssh`
Comandos principales:
```
aaa new-model
radius server NPS
 address ipv4 10.21.68.10 auth-port 1812 acct-port 1813
 key \\\\\\\*\\\\\\\*\\\\\\\*\\\\\\\*\\\\\\\*\\\\\\\*\\\\\\\*\\\\\\\*

aaa authentication login default local
aaa authorization exec default local
aaa authentication login CON-AUX group radius local
aaa authorization exec CON-AUX group radius local
aaa accounting exec default start-stop group radius
aaa accounting system default start-stop group radius

line con 0
 login authentication CON-AUX
 authorization exec CON-AUX
line aux 0
 login authentication CON-AUX
 authorization exec CON-AUX
line vty 0 4
 transport input ssh
 login authentication default
 authorization exec default

login on-success log
login on-failure log
logging buffered 16384 informational
```
---
6. Usuarios y niveles de privilegio
Usuario	Origen	Acceso	Nivel
`admin`	Local	SSH y respaldo si RADIUS no responde	15
`user0`	Active Directory (RADIUS)	Consola	0
`user5`	Active Directory (RADIUS)	Consola	5
`user10`	Active Directory (RADIUS)	Consola	10
---
7. Evidencias
7.1 Configuración del router
Usuario local, RADIUS, AAA, líneas, SSH e interfaces.
![Configuración router](imagenes/user%20local%2Bradius%2Baaa%2Bsec%20line%2Bssh%20y%20ineterface%20brief%20router.png)
7.2 Configuración del switch
Usuario local, RADIUS, AAA, líneas, SSH e interfaces.
![Configuración switch](imagenes/user%20local%2Bradius%2Baaa%2Bsec%20line%2Bssh%20y%20ineterface%20brief%20switch.png)
7.3 Usuario `user0` (nivel 0)
Nivel asignado por NPS, verificado con `show privilege` antes de usar `enable`.
![user0](imagenes/user%200%20sin%20enablke%20privilegio%20para%20muestra.png)
7.4 Usuario `user5` (nivel 5)
![user5](imagenes/user%205%20y%20privilegio.png)
7.5 Usuario `user10` (nivel 10)
![user10](imagenes/Usuario%20y%20privilegio%2010.png)
7.6 Demostración de los tres niveles
![Niveles](imagenes/Usuarios%20y%20niveles%20demostracion.png)
7.7 Logs
Logs de autenticación, autorización y RADIUS.
![Logs](imagenes/logs.png)
---
8. Running-configs
Equipo	Archivo
R1	`configs/R1-running-config.txt`
SW1	`configs/SW1-running-config.txt`
