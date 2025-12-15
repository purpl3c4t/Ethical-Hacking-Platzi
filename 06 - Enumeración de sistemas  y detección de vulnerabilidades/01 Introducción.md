# Enumeración de Sistemas y Detección de Vulnerabilidades

Una vez que sabemos qué puertos están abiertos (gracias al escaneo), entramos en la fase de **Enumeración**. Aquí, dejamos de mirar la "fachada de la casa" y empezamos a tocar los timbres, girar las perillas de las puertas y leer el correo en el buzón.

El objetivo es pasar de tener una simple dirección IP a tener un mapa detallado de **usuarios, recursos compartidos y versiones de software** que puedan ser explotados.


---

> **⚠️ Disclaimer Ético:** Esta información se proporciona con fines exclusivamente educativos y para la formación de profesionales en ciberseguridad. La ejecución de estos ataques en redes sin autorización explícita es ilegal y penada por la ley. Practica siempre en tu propio laboratorio controlado.

---

## 1. Enumeración

La enumeración es el proceso de establecer una conexión activa con los sistemas objetivo para realizar consultas dirigidas. A diferencia del escaneo (que suele ser rápido y superficial), la enumeración es profunda y específica por protocolo.

**¿Qué buscamos?**
* Nombres de usuario y grupos.
* Nombres de máquinas (Hostnames).
* Recursos compartidos en red.
* Tablas de enrutamiento y configuraciones de servicios.
* Versiones exactas de aplicaciones (Banner Grabbing).

---

## 2. Enumeración por Protocolo

Analizaremos los 7 protocolos más críticos que encontrarás en una auditoría y que son utilizados en los contenedores que desplegaste para formar tu laboratorio:

### DNS (Domain Name System) - Puerto 53
El DNS es la guía telefónica de internet. Si está mal configurado, puede revelarnos la estructura completa de la red interna de una organización.

* **El Objetivo (The Gold Mine): Transferencia de Zona (AXFR).**
    Una transferencia de zona ocurre cuando un servidor DNS le entrega una copia completa de todos sus registros a otro. Si un administrador olvida restringir esto, podemos descargar el mapa completo de la red (IPs internas, servidores de prueba, VPNs) sin disparar una sola alarma.
* **Herramientas:** `dnsenum`, `dig` (comando: `dig axfr @ip dominio`), `fierce`.

### FTP (File Transfer Protocol) - Puerto 21
Un protocolo antiguo y a menudo inseguro para mover archivos.

* **Banner Grabbing:** Al conectarnos, el servidor suele "gritar" su versión (ej: *vsFTPd 2.3.4*). Buscamos vulnerabilidades específicas para esa versión.
* **Anonymous Login:** Muchos servidores permiten entrar con el usuario `anonymous` y cualquier contraseña. Esto permite ver archivos sensibles olvidados.
* **Herramientas:** `nmap` (scripts NSE), conexión manual vía terminal (`ftp <ip>`).

### SMB (Server Message Block) - Puerto 139 / 445
Es el protocolo de compartición de archivos e impresoras de Windows. Es, con diferencia, **la fuente más rica de información** en redes corporativas internas.

* **Null Session:** Es una característica antigua (pero a veces presente) que permite conectarse al servidor sin usuario ni contraseña para listar usuarios y grupos.
* **Recursos Compartidos (Shares):** Buscamos carpetas con permisos de lectura para "Todos" (Everyone) que contengan backups, scripts con contraseñas o documentos de RRHH.
* **Herramientas:**
    * `enum4linux`: La navaja suiza para enumerar Windows desde Linux.
    * `smbclient`: Para navegar como si fuera un FTP.
    * `CrackMapExec`: Para enumeración masiva en redes grandes.

### SNMP (Simple Network Management Protocol) - Puerto 161 (UDP)
Es un protocolo usado por administradores para monitorear routers, impresoras y servidores.

* **Community Strings:** SNMP funciona con una "contraseña" llamada *Community String*.
    * `public`: Solo lectura (Read-only).
    * `private`: Lectura y escritura (Read-write).
* **El Fallo:** La mayoría de los dispositivos vienen con la string `public` por defecto. Si la adivinas, el dispositivo te contará todo: cuánto tiempo lleva encendido, qué procesos corren, interfaces de red y tablas de enrutamiento.
* **Herramientas:** `snmpwalk`, `onesixtyone` (fuerza bruta de community strings).

### SSH (Secure Shell) - Puerto 22
Es el estándar para administración remota segura. Aunque el tráfico va cifrado, la fase de autenticación nos da pistas.

* **Enumeración de Versión:** Identificar si es OpenSSH y qué versión (versiones antiguas tienen vulnerabilidades de enumeración de usuarios).
* **Métodos de Autenticación:** ¿El servidor pide contraseña o solo acepta llaves (Public Key)? Esto define si vale la pena intentar un ataque de fuerza bruta.
* **Herramientas:** `ssh -v <ip>` (modo verbose), `nmap` scripts, `hydra` (para probar credenciales débiles).

### HTTP/HTTPS (Web) - Puerto 80 / 443
La superficie de ataque más grande hoy en día.

* **Tecnologías:** Identificar qué corre detrás (Apache, Nginx, IIS, PHP, ASP.NET).
* **Directorios Ocultos (Fuzzing):** Las webs suelen tener carpetas que no están en el menú principal (ej: `/admin`, `/backup`, `/test`). Usamos diccionarios para encontrarlas.
* **Archivos de Interés:** `robots.txt` (le dice a Google qué no mirar, lo cual nos dice a nosotros qué es interesante) y `sitemap.xml`.
* **Herramientas:** `Nikto` (escáner de vulnerabilidades web básico), `Dirb` / `Gobuster` (fuerza bruta de directorios), `Wappalyzer` (identificador de tecnologías).

### SQL (Bases de Datos) - Puertos 1433 (MSSQL) / 3306 (MySQL)
Donde residen los datos.

* **Detección de Instancias:** A veces un servidor tiene múltiples bases de datos corriendo.
* **Información de Versión:** Vital para saber si existen *exploits* de escalada de privilegios.
* **Usuarios por Defecto:** Intentar enumerar si existen cuentas como `sa` (System Administrator) o `root` con contraseñas vacías o por defecto.
* **Herramientas:** `nmap` (scripts como `mysql-info`, `ms-sql-info`), módulos auxiliares de Metasploit.

---

## 3. Herramientas por Protocolo

| Protocolo | Puerto Típico | Herramienta Principal | Qué buscamos |
| :--- | :--- | :--- | :--- |
| **DNS** | 53 | `dnsenum`, `dig` | Transferencia de zona, IPs internas |
| **FTP** | 21 | `nmap`, `netcat` | Login anónimo, versiones vulnerables |
| **SMB** | 445 | `enum4linux`, `smbclient` | Usuarios, carpetas compartidas |
| **SNMP** | 161 (UDP) | `snmpwalk`, `onesixtyone` | Info del sistema, community strings |
| **SSH** | 22 | `hydra`, `ssh -v` | Versiones, fuerza bruta |
| **HTTP** | 80/443 | `nikto`, `gobuster` | Directorios ocultos, tecnologías |
| **SQL** | 3306/1433 | `nmap`, `metasploit` | Versiones de BD, usuarios default |

---

### ⚠️ Nota
La enumeración es una actividad **intrusiva**. A diferencia del reconocimiento pasivo (OSINT), aquí estás interactuando directamente con los servidores. Los sistemas de detección de intrusos (IDS) y los Firewalls modernos (NGFW) registrarán esta actividad. En un entorno profesional, la enumeración debe hacerse con **extrema precaución** o bajo una ventana de mantenimiento autorizada para evitar bloquear cuentas de usuarios o saturar servicios antiguos.
