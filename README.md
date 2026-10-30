# Implementación de AAA con RADIUS y Windows Server NPS

<p align="center">
  <img src="Imagenes/Infraestructura.png" alt="Topología de la infraestructura" width="800">
</p>

<p align="center">
  <strong>Laboratorio de administración y seguridad de redes</strong><br>
  Autenticación, autorización y auditoría centralizada mediante AAA, RADIUS y Network Policy Server.
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Network-Cisco-blue?style=for-the-badge" alt="Cisco">
  <img src="https://img.shields.io/badge/Authentication-RADIUS-green?style=for-the-badge" alt="RADIUS">
  <img src="https://img.shields.io/badge/Server-Windows_Server-0078D4?style=for-the-badge" alt="Windows Server">
  <img src="https://img.shields.io/badge/OS-Ubuntu-E95420?style=for-the-badge" alt="Ubuntu">
</p>

## 📑 Índice

- [1. Introducción](#-1-introducción)
- [2. Objetivos](#-2-objetivos)
- [3. Descripción de la infraestructura](#-3-descripción-de-la-infraestructura)
- [4. Direccionamiento IP](#-4-direccionamiento-ip)
- [5. Topología de red](#-5-topología-de-red)
- [6. Configuración de los dispositivos](#-6-configuración-de-los-dispositivos)
- [7. Configuración de Windows Server NPS](#-7-configuración-de-windows-server-nps)
- [8. Usuarios y privilegios](#-8-usuarios-y-privilegios)
- [9. Pruebas y validaciones](#-9-pruebas-y-validaciones)
- [10. Registros y auditoría](#-10-registros-y-auditoría)
- [11. Configuraciones del laboratorio](#-11-configuraciones-del-laboratorio)
- [12. Video de demostración](#-12-video-de-demostración)
- [13. Conclusiones](#-13-conclusiones)

## 📌 1. Introducción

Este proyecto consiste en la implementación de un laboratorio de redes orientado a la seguridad y la administración centralizada de usuarios mediante el modelo AAA (Authentication, Authorization and Accounting).

La infraestructura integra dispositivos Cisco, un servidor Windows con Network Policy Server (NPS) y un equipo Ubuntu. El objetivo es centralizar la autenticación de los accesos administrativos a los dispositivos de red mediante el protocolo RADIUS, gestionar los niveles de autorización y disponer de registros que permitan revisar los intentos de acceso.

El laboratorio permite estudiar cómo interactúan los dispositivos de red con un servidor de autenticación y cómo las políticas de acceso contribuyen a mejorar el control y la seguridad de la infraestructura.

## 🎯 2. Objetivos

### Objetivo general

Implementar un sistema de autenticación, autorización y auditoría centralizada para la administración de dispositivos de red utilizando AAA, RADIUS y Windows Server NPS.

### Objetivos específicos

- Configurar el direccionamiento IP de los dispositivos del laboratorio.
- Establecer la conectividad entre el router, el switch y los servidores.
- Configurar los parámetros AAA en los dispositivos Cisco.
- Integrar Windows Server NPS como servidor RADIUS.
- Definir usuarios, permisos y niveles de privilegio.
- Comprobar los accesos correctos y los intentos de autenticación fallidos.
- Revisar los registros de eventos para verificar la actividad de autenticación.
- Documentar las configuraciones y las evidencias obtenidas durante las pruebas.

## 🖥️ 3. Descripción de la infraestructura

La infraestructura está compuesta por los siguientes dispositivos:

| Dispositivo | Función |
|---|---|
| Router Cisco | Proporciona conectividad de capa 3 y permite administrar los accesos al dispositivo. |
| Switch Cisco | Conecta los equipos de la red local y permite administrar los accesos al dispositivo. |
| Windows Server | Aloja los servicios y las políticas de NPS para la autenticación RADIUS. |
| Ubuntu Server | Equipo Linux integrado en el laboratorio para tareas de conectividad y administración. |

La comunicación entre los dispositivos permite realizar las pruebas de conectividad y validar la configuración del sistema AAA.

## 🌐 4. Direccionamiento IP

El laboratorio utiliza la red `10.21.68.0/24`.

| Dispositivo | Dirección IP | Máscara de subred | Puerta de enlace |
|---|---|---|---|
| Router Cisco | `10.21.68.1` | `255.255.255.0` | No aplica |
| Switch Cisco | `10.21.68.2` | `255.255.255.0` | `10.21.68.1` |
| Windows Server / NPS | `10.21.68.10` | `255.255.255.0` | `10.21.68.1` |
| Ubuntu Server | `10.21.68.100` | `255.255.255.0` | `10.21.68.1` |

> **Nota:** La tabla documenta el direccionamiento previsto para el laboratorio. Las máscaras y puertas de enlace deben coincidir con los valores configurados realmente en cada equipo.

### Configuración IP de Windows Server

Windows Server utiliza la dirección `10.21.68.10` para alojar los servicios de red correspondientes al laboratorio.

![Configuración IP del servidor Windows](<Imagenes/IP CONF servidor de window.png>)

### Configuración IP de Ubuntu

Ubuntu Server utiliza la dirección `10.21.68.100` para su integración en la red local.

![Configuración IP de Ubuntu](<Imagenes/IP CONSOLA DE UBSV.png>)

## 🗺️ 5. Topología de red

La siguiente imagen presenta la infraestructura implementada para el laboratorio.

![Topología de la infraestructura](<Imagenes/Infraestructura.png>)

La red está organizada para permitir la comunicación entre los dispositivos Cisco y los servidores. El router utiliza la dirección `10.21.68.1`, mientras que el switch utiliza `10.21.68.2`.

Windows Server, con la dirección `10.21.68.10`, participa en la implementación de los servicios de autenticación y autorización. Ubuntu Server utiliza la dirección `10.21.68.100`.

El flujo de autenticación RADIUS depende de que el router y el switch estén configurados como clientes RADIUS y puedan comunicarse con el servidor NPS.

## ⚙️ 6. Configuración de los dispositivos

### 6.1. Configuración del router Cisco

El router se configura con el direccionamiento correspondiente y los parámetros de administración y autenticación requeridos por el laboratorio.

La configuración AAA permite definir los métodos de autenticación que utilizará el dispositivo y, según los métodos configurados, recurrir al servidor RADIUS para validar las credenciales de los usuarios.

**Evidencia de configuración:**

![Configuración del router](<Imagenes/Config router.png>)

### 6.2. Configuración del switch Cisco

El switch utiliza la dirección `10.21.68.2` para su administración. Su configuración contempla los parámetros de red y los mecanismos de autenticación necesarios para las pruebas.

**Evidencia de configuración:**

![Configuración del switch](<Imagenes/config SW.png>)

### 6.3. Verificación de interfaces

El comando `show ip interface brief` permite consultar las interfaces, sus direcciones IP y su estado operativo.

En el switch se utiliza `show interfaces status` o `show interfaces` para revisar el estado de los puertos, según la información que se necesite comprobar.

**Evidencias:**

![Interfaces del switch](<Imagenes/SW SHOW INT BRIEF.png>)

![Resumen de interfaces IP](<Imagenes/show ip interface brief.png>)

## 🔐 7. Configuración de Windows Server NPS

Network Policy Server (NPS) es el componente de Windows Server que permite implementar políticas de acceso a la red y utilizar RADIUS para procesar solicitudes de autenticación y autorización.

En este laboratorio, NPS se utiliza para centralizar la validación de los accesos administrativos de los dispositivos de red.

### 7.1. Configuración de clientes RADIUS

Para que el servidor NPS pueda procesar las solicitudes, los dispositivos que actúan como clientes RADIUS deben estar registrados con sus direcciones IP y sus respectivos secretos compartidos.

En este laboratorio, las direcciones de los dispositivos Cisco son:

- Router: `10.21.68.1`
- Switch: `10.21.68.2`

El secreto compartido debe coincidir en el dispositivo Cisco y en la configuración del cliente RADIUS correspondiente en NPS.

![Clientes RADIUS del router y el switch](<Imagenes/Clientes de Radius R1 Y SW1.png>)

### 7.2. Configuración de directivas de grupo

Las directivas de grupo permiten administrar determinadas configuraciones de Windows de manera centralizada, cuando el equipo está integrado en un dominio compatible con estas funciones.

Las directivas que se muestran en las evidencias forman parte de la configuración administrativa documentada en el laboratorio.

![Directivas de grupo](<Imagenes/Directivas de grupo.png>)

### 7.3. Detalles de las directivas

La siguiente evidencia muestra los parámetros de una de las directivas configuradas.

![Detalles de una directiva](<Imagenes/detalles de una de las directivas.png>)

**Importante:** Las directivas de grupo de Windows y las políticas de red de NPS son mecanismos distintos. Las primeras administran configuraciones de usuarios y equipos Windows; las segundas determinan cómo se procesan y autorizan las solicitudes de acceso RADIUS.

## 👤 8. Usuarios y privilegios

La gestión de usuarios permite comprobar cómo se aplican diferentes niveles de acceso a los dispositivos de red.

En Cisco IOS, el nivel de privilegio 15 corresponde al nivel administrativo más alto. Los permisos efectivos dependen de la configuración del dispositivo y de las políticas de autorización aplicadas.

### 8.1. Evidencias de usuarios

**Usuario configurado:**

![Usuario configurado](<Imagenes/User.png>)

**Usuario en el switch:**

![Usuario del switch](<Imagenes/User0 sw.png>)

**Usuario adicional en el switch:**

![Usuario adicional del switch](<Imagenes/User5 SW.png>)

**Usuario en el router:**

![Usuario en el router](<Imagenes/user0 en r1.png>)

**Usuario 10 en el switch:**

![Usuario 10 en el switch](<Imagenes/user10 SW.png>)

### 8.2. Verificación de privilegios

Las siguientes capturas documentan las pruebas relacionadas con los niveles de privilegio asignados a los usuarios.

![Privilegios de usuario en el router](<Imagenes/User 5 privilegios r1.png>)

![Privilegios de usuario 10 en el router](<Imagenes/user10 privilegios R1.png>)

![Privilegio administrativo nivel 15](<Imagenes/privilege de admin en 15.png>)

Estas pruebas permiten documentar los niveles de acceso disponibles y comprobar si los permisos observados corresponden a la configuración realizada.

## 🧪 9. Pruebas y validaciones

Las pruebas de validación permiten comprobar la conectividad de la infraestructura y el comportamiento del sistema de autenticación.

### 9.1. Pruebas de conectividad

Se realizan pruebas de comunicación entre los equipos para verificar que exista conectividad IP entre los dispositivos que participan en el laboratorio.

![Prueba de conectividad](<Imagenes/ping desde el sv al router y el sw.png>)

### 9.2. Pruebas de autenticación AAA

Las pruebas AAA permiten revisar el comportamiento de la autenticación de usuarios y comprobar si las solicitudes son aceptadas o rechazadas según la configuración aplicada.

![Pruebas de autenticación AAA](<Imagenes/Pruebas test AAA.png>)

### 9.3. Prueba de autenticación fallida

Los intentos de autenticación fallidos permiten comprobar el comportamiento del sistema ante credenciales incorrectas o solicitudes que no cumplen las políticas de acceso.

![Inicio de sesión fallido](<Imagenes/loggin fallido.png>)

Para confirmar que el rechazo fue procesado por NPS, debe revisarse el registro correspondiente en el servidor.

## 📊 10. Registros y auditoría

El registro de eventos permite analizar los intentos de acceso y consultar información relacionada con la autenticación.

En Windows Server, los eventos de NPS pueden revisarse mediante el Visor de eventos y los registros de seguridad correspondientes.

### 10.1. Registros de actividad

![Registros de logging](<Imagenes/logs de logging.png>)

### 10.2. Visor de eventos de NPS

![Visor de eventos de NPS](<Imagenes/visor de eventos de NPS.png>)

La revisión de los registros ayuda a identificar errores de autenticación, comprobar solicitudes procesadas y facilitar el diagnóstico de problemas.

## 📁 11. Configuraciones del laboratorio

El repositorio contiene una carpeta llamada `show running`, en la que se almacenan los archivos de configuración y las salidas de los comandos utilizados para documentar los dispositivos.

| Archivo | Descripción |
|---|---|
| [`Router.txt`](<show running/Router.txt>) | Configuración y comandos documentados del router. |
| [`SW1#show running.txt`](<show running/SW1#show running.txt>) | Configuración y comandos documentados del switch. |

Estos archivos permiten consultar las configuraciones utilizadas durante el laboratorio sin depender únicamente de las capturas de pantalla.

## 🎥 12. Video de demostración

En el siguiente video se presenta la demostración del laboratorio y las pruebas realizadas durante su implementación.

<p align="center">
  <a href="https://youtu.be/-LEGZHsq84I">
    <img src="https://img.youtube.com/vi/-LEGZHsq84I/hqdefault.jpg" alt="Video de demostración del laboratorio AAA RADIUS NPS" width="700">
  </a>
</p>

<p align="center">
  <a href="https://youtu.be/-LEGZHsq84I"><strong>▶ Ver video de demostración en YouTube</strong></a>
</p>

## ✅ 13. Conclusiones

Con este laboratorio pude trabajar con la configuración de dispositivos Cisco y la implementación de un sistema de autenticación centralizada mediante RADIUS y Windows Server NPS. También pude documentar las direcciones IP, revisar los privilegios de los usuarios y realizar pruebas de conectividad y autenticación.

La práctica me permitió comprender mejor la importancia de centralizar el control de acceso y mantener registros que ayuden a revisar los intentos de inicio de sesión. Además, la documentación de las configuraciones y las evidencias facilita la verificación del trabajo realizado.

---

<p align="center">
  <strong>Laboratorio de redes y seguridad informática</strong><br>
  Documentación técnica del proyecto AAA, RADIUS y Windows Server NPS.
</p>
