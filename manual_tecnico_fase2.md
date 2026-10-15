# Manual Técnico de Configuraciones — Proyecto Final Fase 2

**Despliegue de infraestructura de red y servicios empresariales**

> **Cómo usar este archivo:** las imágenes están en la carpeta `images/`, junto a este `.md`. Los nombres de archivo coinciden con los de las capturas. Las partes marcadas como **Evidencia pendiente** no tienen captura todavía y traen los comandos y el resultado esperado. Para buscar algo, usar el índice o la sección 15 (comandos de referencia).

---

## Carátula

**Universidad:** Universidad de San Carlos de Guatemala (USAC)
**Facultad/Escuela:** Facultad de Ingeniería, Escuela de Ciencias y Sistemas
**Curso:** Prácticas Iniciales
**Catedrático/Auxiliar:** Inga. Floriza Felipa Ávila de Medinilla

**Integrantes:**
- Sergio Roberto Gudiel Sian: red, DNS, usuarios, DHCP, servidor web (IIS) y cliente Linux (SRV1 y Ubuntu)
- Rudin Alexander Lopez Salvatierra: FTP, servidor de correo y webmail (SRV2)

**Fecha de entrega:** 15 de octubre de 2026

---

## Índice

- [1. Introducción](#1-introducción)
- [2. Objetivos](#2-objetivos)
- [3. Arquitectura de la red](#3-arquitectura-de-la-red)
- [4. Red y conectividad entre equipos](#4-red-y-conectividad-entre-equipos)
   - [4.1 Qué se hizo y por qué](#41-qué-se-hizo-y-por-qué)
   - [4.2 Comandos usados](#42-comandos-usados)
   - [4.3 Resultado verificado](#43-resultado-verificado)
- [5. Usuarios y registros DNS (SRV1)](#5-usuarios-y-registros-dns-srv1)
   - [5.1 Qué es y para qué sirve](#51-qué-es-y-para-qué-sirve)
   - [5.2 Creación de usuarios](#52-creación-de-usuarios)
   - [5.3 Registros DNS](#53-registros-dns)
- [6. Servicio DHCP (SRV1)](#6-servicio-dhcp-srv1)
   - [6.1 Qué es y para qué sirve](#61-qué-es-y-para-qué-sirve)
   - [6.2 Configuración](#62-configuración)
   - [6.3 Comprobación](#63-comprobación)
- [7. Servidor web IIS (SRV1)](#7-servidor-web-iis-srv1)
   - [7.1 Qué es y para qué sirve](#71-qué-es-y-para-qué-sirve)
   - [7.2 Instalación del rol](#72-instalación-del-rol)
   - [7.3 Sitio público: publicweb](#73-sitio-público-publicweb)
   - [7.4 Sitio privado: privadaweb (puerto 1080)](#74-sitio-privado-privadaweb-puerto-1080)
   - [7.5 Redirección de /correo](#75-redirección-de-correo)
   - [7.6 Configuración adicional de Default Web Site](#76-configuración-adicional-de-default-web-site)
- [8. Cliente Linux y cliente de correo](#8-cliente-linux-y-cliente-de-correo)
   - [8.1 Cliente Ubuntu unido al dominio](#81-cliente-ubuntu-unido-al-dominio)
   - [8.2 Thunderbird](#82-thunderbird)
   - [8.3 Comprobación de envío y recepción](#83-comprobación-de-envío-y-recepción)
- [9. Servidor FTP (SRV2)](#9-servidor-ftp-srv2)
   - [9.1 Qué es y para qué sirve](#91-qué-es-y-para-qué-sirve)
   - [9.2 Configuración](#92-configuración)
   - [9.3 Comprobación desde cliente Linux (Ubuntu)](#93-comprobación-desde-cliente-linux-ubuntu)
   - [9.4 Comprobación desde cliente Windows](#94-comprobación-desde-cliente-windows)
- [10. Servidor de correo (SRV2)](#10-servidor-de-correo-srv2)
   - [10.1 Qué es y para qué sirve](#101-qué-es-y-para-qué-sirve)
   - [10.2 Configuración](#102-configuración)
   - [10.3 Comprobación](#103-comprobación)
- [11. Webmail SnappyMail (SRV2)](#11-webmail-snappymail-srv2)
   - [11.1 Qué es y para qué sirve](#111-qué-es-y-para-qué-sirve)
   - [11.2 Configuración](#112-configuración)
   - [11.3 Comprobación](#113-comprobación)
   - [11.4 Redirección desde /correo](#114-redirección-desde-correo)
- [12. Pruebas de funcionamiento](#12-pruebas-de-funcionamiento)
- [13. Problemas encontrados y soluciones](#13-problemas-encontrados-y-soluciones)
- [14. Resumen de usuarios y accesos](#14-resumen-de-usuarios-y-accesos)
- [15. Comandos de referencia](#15-comandos-de-referencia)
- [16. Conclusiones](#16-conclusiones)

---

## 1. Introducción

Este manual documenta la Fase 2 de la infraestructura de red implementada en un entorno virtualizado, sobre el dominio `servidor1.com` creado en la Fase 1. Los dos servidores Windows Server 2022 están en dos computadoras físicas distintas, interconectadas mediante Radmin VPN.

En esta fase se agregan los servicios de **servidor web** (IIS, con un sitio público, un sitio privado con autenticación y una redirección), **correo electrónico** (hMailServer, con webmail SnappyMail accesible por redirección) y **transferencia de archivos** (FTP sobre IIS). Todos los servicios se prueban con nombres de dominio y no con direcciones IP, y se demuestra su uso desde un cliente Linux (Ubuntu) unido al dominio.

---

## 2. Objetivos

- Publicar un sitio web público y un sitio web privado protegido con autenticación mediante IIS.
- Redirigir la ruta `/correo` del servidor web hacia el webmail del servidor de correo, sin código HTML, JS ni PHP.
- Registrar en el DNS del dominio los nombres de los servicios (`mail`, `ftp`) y el registro MX.
- Poner en funcionamiento un servidor FTP y un servidor de correo integrados con los usuarios de Active Directory.
- Garantizar que el cliente Ubuntu reciba su IP por DHCP y llegue a ambos servidores.
- Demostrar desde el cliente Linux unido al dominio el uso de los servicios.

---

## 3. Arquitectura de la red

| Rol | Equipo | Sistema operativo | IP |
|---|---|---|---|
| Controlador de dominio, DNS, DHCP, IIS (SRV1) | Laptop 1 (VM "SRV1") | Windows Server 2022 | 192.168.0.20 (en "Puente de red") |
| Servidor miembro: FTP, correo y webmail (SRV2) | Laptop 2 (VM "Windows Server") | Windows Server 2022 | 192.168.0.21 |
| Cliente Linux | Laptop 1 (VM "Linux 1") | Ubuntu Desktop | 192.168.0.30 por DHCP (reservada) |
| Interconexión | Radmin VPN y adaptador UDPTunnel de VirtualBox | n/a | Red virtual `26.x.x.x` |

**Dominio:** `servidor1.com` (NetBIOS `SERVIDOR1`)

**Nombres de servicio (DNS):**

| Nombre | Tipo | Valor |
|---|---|---|
| `mail.servidor1.com` | A | 192.168.0.21 |
| `ftp.servidor1.com` | A | 192.168.0.21 |
| `servidor1.com` | MX (prioridad 10) | `mail.servidor1.com` |

**Usuarios del dominio usados en esta fase:**

| Usuario | Uso |
|---|---|
| `usuario1`, `usuario2` | FTP y sesión en el cliente Linux |
| `correo1`, `correo2` | Cuentas de correo |
| `privado` | Acceso al sitio web privado |

**Servicios y puertos:**

| Servicio | Equipo | Puerto |
|---|---|---|
| Sitio público `publicweb` | SRV1 | 80 |
| Sitio privado `privadaweb` | SRV1 | 1080 |
| FTP (control) | SRV2 | 21 |
| FTP (datos pasivos) | SRV2 | 50000-50100 |
| SMTP / POP3 / IMAP | SRV2 | 25 / 110 / 143 |
| Webmail SnappyMail (IIS) | SRV2 | 80 |

---

## 4. Red y conectividad entre equipos

### 4.1 Qué se hizo y por qué

SRV1 tiene dos adaptadores de VirtualBox: el adaptador 1 es un **UDPTunnel** (llega a SRV2 por Radmin VPN) y el adaptador 2 es una **red interna** `lab-servidor1` (donde está Ubuntu). Al principio eran dos redes separadas dentro de SRV1: el túnel tenía la IP 192.168.0.20 y la red interna no tenía IP válida. Por eso el DHCP no atendía a Ubuntu y Ubuntu no llegaba a SRV2.

La solución fue activar el **modo promiscuo** en el adaptador 2 y crear en Windows un **puente de red** que une "Ethernet" (túnel) y "Ethernet 2" (red interna), de modo que Ubuntu, SRV1 y SRV2 queden en un mismo segmento 192.168.0.0/24.

| Adaptador de SRV1 | Tipo en VirtualBox | Nombre en Windows | MAC |
|---|---|---|---|
| Adaptador 1 | Controlador genérico, UDPTunnel | Ethernet | 08002703840E |
| Adaptador 2 | Red interna `lab-servidor1`, promiscuo `allow-all` | Ethernet 2 | 08002737119E |

### 4.2 Comandos usados

Con SRV1 apagado, en la laptop física (PowerShell):

```powershell
$vb = "C:\Program Files\Oracle\VirtualBox\VBoxManage.exe"
& $vb modifyvm "SRV1" --nicpromisc2 allow-all
& $vb startvm "SRV1"
```

En SRV1: `ncpa.cpl`, seleccionar **Ethernet** y **Ethernet 2**, clic derecho, **Conexiones de puente**. Después, con PowerShell como administrador:

```powershell
Get-NetAdapter -IncludeHidden | Format-Table Name,InterfaceDescription,Status
New-NetIPAddress -InterfaceAlias "Puente de red" -IPAddress 192.168.0.20 -PrefixLength 24
Set-DnsClientServerAddress -InterfaceAlias "Puente de red" -ServerAddresses 192.168.0.20
Restart-Service DHCPServer
Get-DhcpServerv4Binding
```

### 4.3 Resultado verificado

- El puente "Puente de red" quedó con la IP 192.168.0.20 en estado `Preferred`.
- El servicio DHCP quedó enlazado al puente (`BindingState True`).
- `ping 192.168.0.21` desde SRV1 y `ping 192.168.0.20` desde SRV2 respondieron con 0% de pérdida.
- Ubuntu obtuvo 192.168.0.30, hizo ping a 192.168.0.21 y resolvió `mail.servidor1.com`.
- Los puertos 25 y 143 de `mail.servidor1.com` respondieron `succeeded` desde Ubuntu (`nc -vz`).

La conectividad entre los dos servidores quedó documentada con la prueba conjunta:

![Ping de SRV2 a SRV1 y resolución de mail y ftp con nslookup](./images/r2-11-prueba-conjunta.png)

**Evidencia pendiente (sin captura):** ventana `ncpa.cpl` con "Puente de red" y sus dos interfaces; configuración de red de SRV1 en VirtualBox con los dos adaptadores; salida de `Get-DhcpServerv4Binding` con `Puente de red` en `True`.

---

## 5. Usuarios y registros DNS (SRV1)

### 5.1 Qué es y para qué sirve

Las cuentas de usuario en Active Directory permiten que una misma identidad sea válida en todos los servicios del dominio (correo, web privada, FTP). El DNS traduce nombres a direcciones IP: el registro **A** asocia un nombre con una IP y el registro **MX** indica qué servidor recibe el correo de un dominio. Sin el MX, otros servidores no sabrían a dónde entregar los mensajes de `@servidor1.com`.

### 5.2 Creación de usuarios

Se crearon las cuentas `correo1`, `correo2` y `privado` en Active Directory con PowerShell (`New-ADUser`), habilitadas y con su UserPrincipalName (`correo1@servidor1.com`, etc.). Se verificó con `Get-ADUser`.

![Creación de los usuarios correo1, correo2 y privado con New-ADUser](./images/f2-11-ad-crear-usuarios.png)

![Usuarios correo1, correo2 y privado habilitados, verificados con Get-ADUser](./images/f2-01-usuarios-ad.png)

### 5.3 Registros DNS

Se crearon en la zona `servidor1.com` los registros A `mail` y `ftp` apuntando a 192.168.0.21 (SRV2) y un registro MX con prioridad 10 que apunta a `mail.servidor1.com` (`Add-DnsServerResourceRecordA` y `Add-DnsServerResourceRecordMX`). Se verificó con `nslookup`, consultando al DNS del dominio (192.168.0.20). El aviso `DNS request timed out` que muestra `nslookup` se debe a que busca el nombre inverso del servidor y no afecta el resultado.

![Creación de los registros A (mail, ftp) y MX](./images/f2-12-dns-crear-registros.png)

![Verificación con nslookup de mail, ftp y el registro MX](./images/f2-02-dns-mail-ftp-mx.png)

Verificación final de los registros A de la zona (ninguno apunta a la IP de la tarjeta NAT de SRV2):

```powershell
Get-DnsServerResourceRecord -ZoneName servidor1.com -RRType A | Select-Object HostName,@{n='IP';e={$_.RecordData.IPv4Address}}
```

| HostName | IP |
|---|---|
| `@`, `DomainDnsZones`, `ForestDnsZones`, `srv1` | 192.168.0.20 |
| `ftp`, `mail`, `SRV2` | 192.168.0.21 |

SRV2 tiene una segunda tarjeta (NAT) para descargas. Se desactivó su registro en el DNS del dominio para evitar fallos intermitentes:

```powershell
Set-DnsClient -InterfaceAlias "Ethernet 2" -RegisterThisConnectionsAddress $false
```

---

## 6. Servicio DHCP (SRV1)

### 6.1 Qué es y para qué sirve

DHCP entrega automáticamente a los clientes su dirección IP, el DNS y la puerta de enlace, y evita colisiones de direcciones.

### 6.2 Configuración

Ámbito `RedProyecto`: 192.168.0.1 a 192.168.0.100. El servicio escucha en el adaptador "Puente de red" (192.168.0.20). Ubuntu recibe la dirección reservada 192.168.0.30.

### 6.3 Comprobación

Concesión registrada en SRV1:

```powershell
Get-DhcpServerv4Lease -ScopeId 192.168.0.0 | Format-Table IPAddress,HostName,ClientId
```

| IPAddress | HostName | ClientId |
|---|---|---|
| 192.168.0.30 | Ubuntu.servidor1.com | 08-00-27-fa-94-00 |

Reinicio de red en Ubuntu (volvió a recibir 192.168.0.30):

```bash
sudo nmcli connection down netplan-enp0s3 && sudo nmcli connection up netplan-enp0s3
hostname -I
```

En Windows se renueva con `ipconfig /renew` en un cliente con DHCP.

![Ubuntu con IP 192.168.0.30, dominio servidor1.com y resolución de mail.servidor1.com](./images/f2-27-ubuntu-dominio-ip.png)

**Evidencia pendiente (sin captura):** salida de `Get-DhcpServerv4Lease` y del reinicio de red en Ubuntu con `hostname -I` mostrando 192.168.0.30.

---

## 7. Servidor web IIS (SRV1)

### 7.1 Qué es y para qué sirve

IIS (Internet Information Services) es el servidor web de Windows Server. Publica sitios y aplicaciones por HTTP. En esta fase se publican dos sitios:

- **publicweb:** acceso libre, sin credenciales.
- **privadaweb:** acceso restringido con **autenticación básica**. El navegador pide usuario y contraseña y el servidor responde `401 Unauthorized` hasta recibir credenciales válidas. La autenticación básica envía las credenciales sin cifrar, lo que es aceptable en este laboratorio pero no en producción sin HTTPS.

Además IIS redirige la ruta `/correo` hacia el webmail con una **redirección HTTP** (código `302`, que indica al navegador que la dirección está temporalmente en otra URL).

### 7.2 Instalación del rol

Se instaló el rol Servidor web (IIS) con sus herramientas de administración mediante `Install-WindowsFeature`. Después se instalaron las características **Autenticación básica** y **Redirección HTTP**.

![Instalación del rol IIS con PowerShell](./images/f2-10-iis-instalacion.png)

![Instalación de Autenticación básica y Redirección HTTP](./images/f2-13-iis-roles-basica-redireccion.png)

### 7.3 Sitio público: publicweb

Se creó la carpeta `C:\inetpub\wwwroot\publicweb` con su `index.html` y se registró como **aplicación** dentro de Default Web Site (`New-WebApplication`, puerto 80). Responde en `http://servidor1.com/publicweb`.

![Creación de la aplicación publicweb](./images/f2-14-iis-crear-publicweb.png)

![publicweb abierto desde el cliente Ubuntu por nombre de dominio](./images/f2-04-publicweb-ubuntu.png)

### 7.4 Sitio privado: privadaweb (puerto 1080)

Se creó la carpeta `C:\inetpub\wwwroot\privadaweb` con su `index.html`, se concedió permiso NTFS de lectura y ejecución al usuario `SERVIDOR1\privado` (`icacls`) y se creó un **sitio** nuevo `privadaweb` en el **puerto 1080**, con un directorio virtual `privadaweb`.

![Creación del sitio privadaweb (puerto 1080), permisos NTFS y directorio virtual](./images/f2-15-iis-crear-privadaweb.png)

Se **deshabilitó la autenticación anónima** y se **habilitó la básica**, tanto en el sitio del puerto 1080 como en el directorio `privadaweb` de Default Web Site (puerto 80), para que no se pueda saltar el acceso entrando por el puerto 80.

![Comando que deshabilita la autenticación anónima y habilita la básica](./images/f2-16-iis-privadaweb-auth-comando.png)

![Autenticación de privadaweb: anónima deshabilitada y básica habilitada](./images/f2-08-iis-autenticacion-privadaweb.png)

Se creó una regla de firewall de entrada para permitir el tráfico TCP al puerto 1080.

![Regla de firewall IIS 1080 (entrada, TCP, permitir)](./images/f2-18-firewall-1080.png)

Se verificó desde SRV1 que el DNS resuelve `mail.servidor1.com` a 192.168.0.21, que el MX apunta a `mail.servidor1.com` con prioridad 10 y que la regla `IIS 1080` está habilitada.

![Verificación de DNS (A y MX) y de la regla de firewall IIS 1080](./images/f2-26-verificacion-dns-firewall.png)

El Administrador de IIS muestra la estructura de sitios y los puertos de cada uno.

![Estructura de sitios en el Administrador de IIS](./images/f2-07-iis-sitios.png)

![Sitios con sus enlaces: Default Web Site (80) y privadaweb (1080)](./images/f2-07b-iis-sitios-puertos.png)

**Comprobación desde Ubuntu:** `http://servidor1.com:1080/privadaweb` pide credenciales (`401`). Con el usuario `privado` se muestra la página privada.

![Cuadro de inicio de sesión del sitio privado](./images/f2-05-privadaweb-login.png)

![Contenido de privadaweb tras autenticarse con el usuario privado](./images/f2-06-privadaweb-contenido.png)

### 7.5 Redirección de /correo

Se creó la carpeta `correo` en `C:\inetpub\wwwroot` y se habilitó en ella la **redirección HTTP** (`httpRedirect`) con destino `http://mail.servidor1.com/webmail`, redirección al destino exacto y código 302. La carpeta no contiene archivos ni código: la redirección es nativa de IIS.

![Comando que configura la redirección de /correo](./images/f2-17-iis-redireccion-comando.png)

![Redirección HTTP de /correo en el Administrador de IIS](./images/f2-09-iis-redireccion-correo.png)

Se comprobó con `curl -I`, que devuelve `302` y el encabezado `Location` con el destino. La prueba se repitió desde Ubuntu:

```
$ curl -I http://servidor1.com/correo
HTTP/1.1 302 Redirect
Location: http://mail.servidor1.com/webmail
Server: Microsoft-IIS/10.0
```

![Respuesta 302 con Location http://mail.servidor1.com/webmail](./images/f2-03-redireccion-correo.png)

### 7.6 Configuración adicional de Default Web Site

![Ruta física de Default Web Site](./images/f2-19-iis-default-web-site-ruta.png)

![Enlace de Default Web Site (http, puerto 80)](./images/f2-20-iis-enlaces-puerto80.png)

![Documento predeterminado index.html](./images/f2-21-iis-documento-predeterminado.png)

---

## 8. Cliente Linux y cliente de correo

### 8.1 Cliente Ubuntu unido al dominio

El cliente Ubuntu, unido al dominio en la Fase 1 con `realmd` y `sssd`, se usa para demostrar los servicios desde Linux. Tras reiniciar los equipos en el orden SRV1 y luego Ubuntu, obtuvo por DHCP la IP reservada 192.168.0.30 y mantiene su unión al dominio (`realm list`). El comando `id` reconoce a `usuario1@servidor1.com` y `mail.servidor1.com` se resuelve por nombre a 192.168.0.21.

![Resolución de mail.servidor1.com desde Ubuntu](./images/f2-22-ubuntu-resolucion-mail.png)

Nota: `usuario1` no tiene `sudo`. Para tareas administrativas se usa la cuenta local `vboxuser` (`su - vboxuser`).

### 8.2 Thunderbird

Se instaló **Mozilla Thunderbird** en Ubuntu y se configuraron las cuentas `correo1@servidor1.com` y `correo2@servidor1.com` apuntando al servidor `mail.servidor1.com`:

| Parámetro | Entrante | Saliente |
|---|---|---|
| Protocolo | IMAP | SMTP |
| Servidor | `mail.servidor1.com` | `mail.servidor1.com` |
| Puerto | 143 | 25 |
| Seguridad de conexión | Ninguna | Ninguna |
| Autenticación | Contraseña normal | Contraseña normal |
| Usuario | `correoN@servidor1.com` | `correoN@servidor1.com` |

**IMAP** mantiene los mensajes en el servidor y los sincroniza entre dispositivos; **POP3** los descarga al equipo; **SMTP** es el protocolo de envío. En el laboratorio no se usa cifrado (SSL/TLS) por simplicidad.

![Configuración IMAP de correo1 (servidor, puerto 143, sin cifrado)](./images/f2-23-thunderbird-imap.png)

![Configuración del servidor saliente SMTP (puerto 25, sin cifrado)](./images/f2-24-thunderbird-smtp.png)

![Cuentas correo1 y correo2 configuradas en Thunderbird](./images/f2-25-thunderbird-cuentas.png)

### 8.3 Comprobación de envío y recepción

Se verificó conectividad a los puertos del servidor desde Ubuntu y después el envío y la recepción de correo entre `correo1` y `correo2` en ambos sentidos.

```bash
nc -vz mail.servidor1.com 143
nc -vz mail.servidor1.com 25
```

![Correo enviado y recibido entre correo1 y correo2 en Thunderbird](./images/f2-28-correo-enviado-recibido.png)

---

## 9. Servidor FTP (SRV2)

### 9.1 Qué es y para qué sirve

FTP (File Transfer Protocol) permite subir y descargar archivos de un servidor. Funciona con un canal de control (puerto 21) y un canal de datos. En **modo activo** el servidor abre la conexión de datos hacia el cliente; en **modo pasivo** el cliente abre ambas conexiones, hacia un rango de puertos del servidor, lo que funciona mejor tras firewalls. Se publica con IIS y se accede por `ftp.servidor1.com`.

### 9.2 Configuración

Se usó el servicio FTP de IIS en Windows Server y se creó el sitio **FTP-Servidor1** con la dirección 192.168.0.21 y el puerto 21. Su directorio físico es `C:\Compartidos\CarpetaPública`, la carpeta compartida de la Fase 1. Se habilitó la **autenticación básica** y se configuraron reglas de autorización para `SERVIDOR1\usuario1` y `SERVIDOR1\usuario2` con permisos de lectura y escritura. Se configuró el rango de **puertos pasivos 50000-50100** y se habilitaron las reglas correspondientes en el firewall de Windows.

![Configuración de red de SRV2: IP 192.168.0.21 y DNS 192.168.0.20](./images/r2-01-ipconfig-srv2.png)

![Sitio FTP-Servidor1 en el Administrador de IIS (192.168.0.21:21, ruta CarpetaPública)](./images/r2-02-ftp-sitio-iis.png)

![Compatibilidad con el firewall de FTP: rango de puertos pasivos 50000-50100](./images/r2-03-ftp-autenticacion-autorizacion.png)

### 9.3 Comprobación desde cliente Linux (Ubuntu)

Desde Ubuntu, como `usuario1`, se subió y se bajó un archivo por nombre de dominio:

```bash
lftp -u usuario1,Password ftp.servidor1.com
put /etc/hostname -o prueba_ubuntu.txt
ls
get prueba_ubuntu.txt -o /tmp/bajado.txt
!cat /tmp/bajado.txt
```

En `lftp`, `put origen -o destino` es la sintaxis para subir con otro nombre. El archivo se subió (7 bytes), apareció en el listado y se descargó con su contenido intacto.

![Subida y bajada de archivos con lftp desde Ubuntu](./images/f2-33-ftp-ubuntu.png)

También se subió `evidencia_ftp_ubuntu.txt` con `curl`, con la respuesta `226 Transfer complete`, y se verificó en el Explorador de archivos de SRV2 que el archivo llegó a la carpeta compartida.

![Archivo evidencia_ftp_ubuntu.txt subido desde Ubuntu y presente en CarpetaPública de SRV2](./images/r2-04-ftp-prueba-cliente.png)

### 9.4 Comprobación desde cliente Windows

**Evidencia pendiente (sin captura):** conexión desde un cliente Windows con el comando de abajo, iniciando sesión con `usuario1` y subiendo y descargando un archivo.

```
ftp ftp.servidor1.com
put prueba.txt
get prueba.txt
ls
```

Resultado esperado: inicio de sesión aceptado (`230`), transferencias completadas (`226 Transfer complete`) y el archivo visible en `CarpetaPública`.

---

## 10. Servidor de correo (SRV2)

### 10.1 Qué es y para qué sirve

Un servidor de correo recibe, almacena y entrega mensajes. Usa **SMTP** (puerto 25) para enviar y entregar entre servidores, y **POP3** (110) o **IMAP** (143) para que los usuarios lean su correo. El servidor sabe a dónde entregar el correo de `@servidor1.com` gracias al registro MX del DNS. Se integra con Active Directory para que `correo1` y `correo2` usen la misma contraseña que tienen en el dominio.

### 10.2 Configuración

Se usó **hMailServer 5.7.1**. Se creó el dominio `servidor1.com` y las cuentas `correo1@servidor1.com` y `correo2@servidor1.com` de tipo Active Directory. Se verificó la habilitación de SMTP, POP3 e IMAP en los puertos 25, 110 y 143. El nombre del servidor SMTP es `mail.servidor1.com` y se crearon las reglas correspondientes en el firewall de Windows.

![Dominio servidor1.com en hMailServer](./images/r2-05-hmail-dominio.png)

![Cuentas correo1 y correo2 en hMailServer](./images/r2-06-hmail-cuentas-ad.png)

![Puertos TCP/IP de hMailServer: SMTP 25, POP3 110, IMAP 143](./images/r2-07-hmail-puertos.png)

### 10.3 Comprobación

Con PowerShell se verificó que el servicio hMailServer estuviera en ejecución y que los puertos 25, 110 y 143 estuvieran en estado `Listen`. Los puertos 25 y 143 también respondieron desde Ubuntu (sección 8.3).

---

## 11. Webmail SnappyMail (SRV2)

### 11.1 Qué es y para qué sirve

Un webmail es un cliente de correo que se usa desde el navegador, sin instalar programas. Se publica en `http://mail.servidor1.com/webmail` y es el destino de la redirección de SRV1 (`http://servidor1.com/correo`).

### 11.2 Configuración

Se instaló **SnappyMail 2.38.2** sobre IIS con **PHP 8.3.35** y FastCGI. Se configuró el dominio `servidor1.com` con IMAP a `mail.servidor1.com:143` y SMTP a `mail.servidor1.com:25`, sin SSL. Se habilitaron los permisos necesarios para la aplicación y se creó una regla de filtrado en IIS que impide el acceso HTTP a la carpeta interna `data`.

### 11.3 Comprobación

Se inició sesión con `correo1@servidor1.com` y `correo2@servidor1.com`, se envió un mensaje desde una cuenta y se verificó su recepción desde la otra.

![Inicio de sesión del webmail con la URL mail.servidor1.com/webmail](./images/r2-08-webmail-login.png)

![Correo enviado y recibido entre correo1 y correo2 en el webmail](./images/r2-09-webmail-correo-enviado.png)

### 11.4 Redirección desde /correo

Al abrir `http://servidor1.com/correo` el navegador llega al webmail de SnappyMail.

![http://servidor1.com/correo llegando al inicio de sesión del webmail](./images/r2-10-redirect-correo.png)

---

## 12. Pruebas de funcionamiento

| # | Prueba | Resultado esperado | Estado |
|---|---|---|---|
| 1 | `http://servidor1.com/publicweb` | 200, página "Carpeta Pública" | Verificada |
| 2 | `http://servidor1.com:1080/privadaweb` sin credenciales | 401, cuadro de inicio de sesión | Verificada |
| 3 | Mismo sitio con `privado` y su contraseña | 200, página "Carpeta Privada" | Verificada |
| 4 | Mismo sitio con contraseña incorrecta | 401 | Verificada |
| 5 | `http://servidor1.com/privadaweb` (puerto 80) | 401 | Verificada |
| 6 | `http://servidor1.com/correo` | 302 a `http://mail.servidor1.com/webmail` | Verificada |
| 7 | `mail`, `ftp` y MX en el DNS | 192.168.0.21 y MX a `mail.servidor1.com` | Verificada |
| 8 | Ubuntu unido al dominio, IP 192.168.0.30 | `realm list` y `id` correctos | Verificada |
| 9 | Ping SRV1 y SRV2 en ambos sentidos | 0% de pérdida | Verificada |
| 10 | DHCP: concesión y reinicio de red en Ubuntu | Vuelve 192.168.0.30 | Verificada |
| 11 | Puertos 25 y 143 desde Ubuntu | `succeeded` | Verificada |
| 12 | Envío y recepción en Thunderbird | Mensaje correo1 a correo2 y respuesta | Verificada |
| 13 | Webmail y redirección desde `/correo` | Inicio de sesión y correo en el navegador | Verificada |
| 14 | FTP desde Ubuntu con `usuario1` | Subir y bajar archivo | Verificada |
| 15 | FTP desde cliente Windows | Subir y bajar archivo | Pendiente de captura |

---

## 13. Problemas encontrados y soluciones

| Problema | Causa | Solución |
|---|---|---|
| Ubuntu sin IP y sin acceso a SRV2 | Dos redes separadas en SRV1 (túnel y red interna) | Modo promiscuo en el adaptador 2 y puente de red (sección 4) |
| Fallos intermitentes en el DNS | Una tarjeta NAT registraba su IP 10.x en el DNS del dominio | Desactivar "Registrar las direcciones de esta conexión en DNS" y borrar los registros 10.x |
| `sssd` en "Backend is offline" | Caché de sssd dañada | Detener sssd, borrar `/var/lib/sss/db/*` y arrancarlo |
| Error de control de acceso por GPO en Ubuntu | Política de acceso de sssd | `ad_gpo_access_control = permissive` en `/etc/sssd/sssd.conf` |
| `put archivo nombre` falla en lftp | `put` no renombra | Usar `put origen -o destino` |
| `nslookup` muestra "DNS request timed out" | Busca el nombre inverso del servidor | Es cosmético. Para una captura limpia: `nslookup servidor1.com 192.168.0.20` |
| Windows Server no responde al ping | Bloquea ICMP entrante por defecto | `New-NetFirewallRule -DisplayName "Permitir ping ICMPv4" -Protocol ICMPv4 -IcmpType 8 -Direction Inbound -Action Allow` |
| Fallos intermitentes entre equipos | El túnel UDP de Radmin puede perder paquetes | Reintentar y revisar Radmin VPN |

**Orden de encendido:** primero SRV1 (dominio y DNS), luego SRV2 y los clientes.

---

## 14. Resumen de usuarios y accesos

| Usuario | Servicio | Acceso |
|---|---|---|
| `usuario1`, `usuario2` | FTP y sesión Ubuntu | `ftp.servidor1.com`, sesión de dominio |
| `correo1`, `correo2` | Correo, Thunderbird y webmail | `mail.servidor1.com` |
| `privado` | Web privada | `http://servidor1.com:1080/privadaweb` |

---

## 15. Comandos de referencia

| Para qué | Comando | Dónde |
|---|---|---|
| Ver adaptadores e IP | `Get-NetAdapter`, `Get-NetIPAddress -AddressFamily IPv4` | SRV1/SRV2 |
| Estado del DHCP | `Get-DhcpServerv4Binding` | SRV1 |
| Concesiones DHCP | `Get-DhcpServerv4Lease -ScopeId 192.168.0.0` | SRV1 |
| Registros DNS | `Get-DnsServerResourceRecord -ZoneName servidor1.com -RRType A` | SRV1 |
| Resolver por nombre | `nslookup mail.servidor1.com 192.168.0.20` | Cualquiera |
| Probar redirección | `curl -I http://servidor1.com/correo` | Ubuntu |
| Probar puertos de correo | `nc -vz mail.servidor1.com 143` y `nc -vz mail.servidor1.com 25` | Ubuntu |
| FTP en Linux | `lftp -u usuario1,Password ftp.servidor1.com` | Ubuntu |
| FTP en Windows | `ftp ftp.servidor1.com` | Windows |
| Reinicio de red | `sudo nmcli connection down netplan-enp0s3 && sudo nmcli connection up netplan-enp0s3` | Ubuntu (`vboxuser`) |
| Dominio en Ubuntu | `realm list`, `id usuario1@servidor1.com` | Ubuntu |
| Renovar IP en Windows | `ipconfig /renew` | Windows |

---

## 16. Conclusiones

La integración de los servicios de esta fase mostró cuánto depende cada uno del DNS: el correo funciona por el registro MX y el acceso por nombre a los sitios y al FTP funciona porque el dominio los resuelve. La autenticación centralizada en Active Directory permite usar las mismas cuentas en varios servicios.

Distribuir los servicios en dos servidores obligó a coordinar el orden de encendido y a cuidar la conectividad entre las dos máquinas físicas por Radmin VPN. El puente de red en SRV1 resolvió que el cliente Ubuntu y SRV2 quedaran en el mismo segmento, y con ello el DHCP, el correo y el FTP pudieron usarse desde Linux por nombre de dominio.

Del lado de SRV2, hMailServer integrado con Active Directory, el FTP de IIS sobre la carpeta compartida de la Fase 1 y el webmail SnappyMail cumplieron el objetivo de ofrecer correo y transferencia de archivos a los usuarios del dominio.
