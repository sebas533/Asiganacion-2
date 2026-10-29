# Asignacion #2 — AAA con RADIUS (NPS) en Router y Switch Cisco

> **Video de demostración:** [Ver video](#https://youtu.be/rcTKsu8TtT4)

**Asignatura:** SR | **Estudiante:** Luis Sebastian Roble Perez | **Matrícula:** 2168

---

## 1. Objetivo

Implementar en GNS3 un esquema **AAA** (autenticación, autorización y registro) en un router y un switch Cisco, usando un servidor **RADIUS (NPS)** sobre **Windows Server 2022** integrado con **Active Directory**.

Lo que se busca demostrar:

- El router y el switch tienen seguridad AAA de autenticación y autorización.
- Ambos equipos están configurados como clientes RADIUS del servidor NPS.
- Cada equipo tiene un usuario local de respaldo.
- RADIUS solo se aplica a los puertos de consola y auxiliar; las líneas VTY usan el usuario local.
- Active Directory tiene tres grupos (nivel 0, 5 y 10) y NPS asigna el privilegio según el grupo del usuario.
- Los logs de autenticación, autorización y RADIUS están habilitados.
- El servicio SSH está activo en el router y en el switch.

---

## 2. Topología

```mermaid
flowchart LR
    R1["R1 (Router)<br/>Fa0/0<br/>10.21.68.1/24"]
    SW["SW1 (Switch)<br/>VLAN 1<br/>10.21.68.2/24"]
    SRV["SRV-AD<br/>Windows Server 2022<br/>AD + NPS<br/>10.21.68.10/24"]
    UB["UbuntuSv<br/>Cliente SSH<br/>10.21.68.100/24"]

    R1 ---|"Fa0/0 a Gi0/0"| SW
    SW ---|"Gi0/1 a Cloud-WS (VMnet2)"| SRV
    SW ---|"Gi0/2"| UB
```

![Topología en GNS3](https://raw.githubusercontent.com/sebas533/Asiganacion-2/main/imagenes/Topologia.png)

| Equipo | Rol |
|---|---|
| `R1` | Router, cliente RADIUS y servidor SSH |
| `SW1` | Switch, cliente RADIUS y servidor SSH |
| `Cloud-WS` | Enlace hacia la VM de Windows Server (VMware, VMnet2) |
| `SRV-AD` | Windows Server 2022 con Active Directory y NPS |
| `UbuntuSv` | Cliente para las pruebas de SSH |

---

## 3. Direccionamiento

El direccionamiento se basa en la matrícula **2168**: red `10.21.68.0/24`, máscara `255.255.255.0` y gateway `10.21.68.1`.

| Equipo | Interfaz | Dirección |
|---|---|---|
| R1 | Fa0/0 (hacia SW1) | `10.21.68.1/24` |
| SW1 | Gi0/0 (hacia R1) | sin IP |
| SW1 | Gi0/1 (hacia servidor) | sin IP |
| SW1 | Gi0/2 (hacia UbuntuSv) | sin IP |
| SW1 | VLAN 1 (administración) | `10.21.68.2/24` |
| SRV-AD | Ethernet1 (VMnet2) | `10.21.68.10/24` |
| UbuntuSv | ens3 | `10.21.68.100/24` |

---

## 4. Servidor Windows

### 4.1 Active Directory

Dominio `lab.local`, con un grupo por nivel de privilegio y un usuario en cada uno.

| Grupo | Usuario | Nivel |
|---|---|---|
| `Nivel0` | `user0` | 0 |
| `Nivel5` | `user5` | 5 |
| `Nivel10` | `user10` | 10 |

### 4.2 NPS (RADIUS)

Clientes RADIUS: `R1` (`10.21.68.1`) y `SW1` (`10.21.68.2`), puertos UDP `1812` (autenticación) y `1813` (accounting).

Tres políticas de red con autenticación PAP, `Service-Type = Login` y el atributo `Cisco-AV-Pair`:

| Orden | Política | Grupo | Cisco-AV-Pair |
|---|---|---|---|
| 1 | `Nivel-10` | `LAB\Nivel10` | `shell:priv-lvl=10` |
| 2 | `Nivel-5` | `LAB\Nivel5` | `shell:priv-lvl=5` |
| 3 | `Nivel-0` | `LAB\Nivel0` | `shell:priv-lvl=0` |

Las tres se procesan **antes** de las políticas por defecto de "Denegar acceso".

![Políticas de red en NPS](https://raw.githubusercontent.com/sebas533/Asiganacion-2/main/imagenes/Politicas.png)

---

## 5. Configuración de los equipos

### 5.1 Router (R1)

```
aaa new-model
radius server NPS
 address ipv4 10.21.68.10 auth-port 1812 acct-port 1813
 key ********

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

![Configuración del router](https://raw.githubusercontent.com/sebas533/Asiganacion-2/main/imagenes/user%20local%2Bradius%2Baaa%2Bsec%20line%2Bssh%20y%20ineterface%20brief%20router.png)

### 5.2 Switch (SW1)

Misma configuración de AAA, RADIUS, usuario local, líneas y SSH. La IP de administración está en la `VLAN 1` (`10.21.68.2/24`).

![Configuración del switch](https://raw.githubusercontent.com/sebas533/Asiganacion-2/main/imagenes/user%20local%2Bradius%2Baaa%2Bsec%20line%2Bssh%20y%20ineterface%20brief%20switch.png)

### 5.3 SSH

Se prueba desde UbuntuSv con el usuario local `admin` (privilegio 15) hacia `10.21.68.1` y `10.21.68.2`. Las líneas VTY no usan RADIUS.

---

## 6. Usuarios y niveles de privilegio

| Usuario | Origen | Acceso | Nivel |
|---|---|---|---|
| `admin` | Local | SSH y respaldo si RADIUS no responde | 15 |
| `user0` | Active Directory (RADIUS) | Consola | 0 |
| `user5` | Active Directory (RADIUS) | Consola | 5 |
| `user10` | Active Directory (RADIUS) | Consola | 10 |

El nivel se verifica con `show privilege` justo después de iniciar sesión, antes de usar `enable`.

### 6.1 `user0` (nivel 0)

![user0](https://raw.githubusercontent.com/sebas533/Asiganacion-2/main/imagenes/user%200%20sin%20enablke%20privilegio%20para%20muestra.png)

### 6.2 `user5` (nivel 5)

![user5](https://raw.githubusercontent.com/sebas533/Asiganacion-2/main/imagenes/user%205%20y%20privilegio.png)

### 6.3 `user10` (nivel 10)

![user10](https://raw.githubusercontent.com/sebas533/Asiganacion-2/main/imagenes/Usuario%20y%20privilegio%2010.png)

### 6.4 Demostración de los tres niveles

![Niveles de privilegio](https://raw.githubusercontent.com/sebas533/Asiganacion-2/main/imagenes/Usuarios%20y%20niveles%20demostracion.png)

---

## 7. Logs

Logs de autenticación, autorización y RADIUS en los equipos (`show logging`) y eventos de acceso de NPS en el servidor.

![Logs](https://raw.githubusercontent.com/sebas533/Asiganacion-2/main/imagenes/logs.png)

---

