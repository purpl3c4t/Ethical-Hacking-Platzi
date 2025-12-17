# Challenge [0006]: [Enumeración de sistemas y detección de vulnerabilidades]

| Información | Detalle |
| :--- | :--- |
| **Tema** | Enumeración de sistemas y detección de vulnerabilidades |
| **Dificultad** | Intermedia |
| **Tiempo Estimado** | 2 Horas |
| **Entorno** | Laboratorio Offensive-Pentesting-Lab (o simulación basada en logs) |

---

## 1. Contexto del Desafío

**Escenario:** Has completado la fase de descubrimiento de puertos y has obtenido un resultado de Nmap prometedor, pero ruidoso. El escáner ha devuelto múltiples puertos abiertos (servicios de correo, FTP, SMB en puertos no estándar y web) y varios scripts de NSE han alertado sobre posibles CVEs y módulos de Metasploit.

Un pentester profesional sabe que no todo lo que brilla es oro. Muchos de estos avisos pueden ser falsos positivos o vulnerabilidades de denegación de servicio (DoS) que no otorgan acceso y podrían tirar el servidor y alertar a tu cliente. 

*Tu misión es **enumerar usuarios válidos** para futuros ataques de fuerza bruta y **descartar vulnerabilidades** para identificar cuál es el vector de entrada real (RCE) y utilizando Metasploit para validar sin dañar.*

## 2. Objetivos

Al completar este desafío, serás capaz de:

1.  **Enumerar Usuarios:** Extraer listas de usuarios válidos interactuando con servicios de mensajería (SMTP/IMAP).
2.  **Validar Vulnerabilidades:** Investigar CVEs reportados automáticamente para distinguir entre riesgo teórico y explotable.
3.  **Mapear Exploits:** Asociar hallazgos de Nmap con módulos específicos de Metasploit Framework y configurarlos correctamente para puertos no estándar.

## 3. Pre-requisitos

* Haber completado el Laboratorio Guiado de Nmap y Enumeración básica.
* Tener acceso a la máquina virtual *VICTIMA1 O VICTIMA2* (basada en el repo `InfoSecWarrior/Offensive-Pentesting-Lab`) o trabajar sobre la simulación de red que coincida con el output provisto (te la dejo en los materiales de esta clase).
* Metasploit Framework (`msfconsole`) instalado y actualizado. (`msfconsole`)

---

## 4. Instrucciones del Desafío

A partir del escaneo de nmap que obtengas en el WriteUp, realiza lo siguiente:

### Tarea 1: Enumeración de Usuarios vía SMTP
El puerto 25 (SMTP) está abierto con `Postfix`. A menudo, los servidores mal configurados permiten verificar si un usuario existe mediante comandos como `VRFY` o `RCPT TO`.

* **Instrucción:** Utiliza un módulo auxiliar de Metasploit o una herramienta manual para enumerar usuarios contra el puerto 25.
* **Requisito A:** Busca el módulo `auxiliary/scanner/smtp/smtp_enum` en Metasploit.
* **Requisito B:** Busca dentro de los directorios de SecList por diferentes listas de usuarios, acorde a cada servicio que estés validando y ejecuta el escáner.

### Tarea 2: Análisis y Descarte de Vulnerabilidades 
El reporte de Nmap muestra alertas para el servicio FTP (puerto 2121) y SMB (puerto 4451). Debes investigar si son viables.

* **Instrucción:** Investiga los CVEs reportados en el output para el puerto 2121 (`vsftpd 3.0.3`).
* **Requisito A:** Busca en Google o bases de datos de exploits los CVE-2021-30047 y CVE-2021-3618 asociados a esa versión.
* **Requisito B:** Determina si estos CVEs permiten Ejecución Remota de Código (RCE) o si son principalmente vectores de Denegación de Servicio (DoS). Basado en esto, justifica en tu reporte por qué descartarías atacar este servicio en una primera fase para evitar tirar el servidor.

### Tarea 3: Selección y Configuración de Exploit Web (RCE)
El puerto 8002 muestra un servidor `Apache httpd 2.4.6 (CentOS)`. El script de Nmap sugiere explícitamente: `MSF: EXPLOIT-MULTI-HTTP-APACHE_NORMALIZE_PATH_RCE`.

* **Instrucción:** Prepara el ataque utilizando Metasploit contra el servicio web del puerto 8002.
* **Requisito A:** Carga el módulo `exploit/multi/http/apache_normalize_path_rce` en `msfconsole`.
* **Requisito B:** Configura el payload y, **muy importante**, ajusta el `RPORT` ya que el servicio no corre en el puerto 80 estándar, sino en el **8002**.
* **Requisito C:** Ejecuta el comando `check` (no `exploit` todavía) para confirmar que Metasploit ve el objetivo como vulnerable.

---

## 5. Completitud

Para validar que este desafío está cubierto, asegúrate de que encontraste todo lo siguiente 

1.  **Lista de Usuarios:** Los nombres de usuario válidos encontrados en la **Tarea 1.**
2.  **Exploits a utilizar**

**Resultado final:**
[Estarás listo para pasar a la fase de explotación de estas vulnerabilidades]

**Obligatorio:**
Comenta en los comentarios de la clase:
*¿Por qué es crítico ejecutar el comando `check` en Metasploit antes de lanzar el exploit `run` en un entorno de producción real, especialmente con servicios inestables?*

---

## 6. Pistas / Troubleshooting (Opcional)

* **Si te quedas atascado:** Revisa los logs en `/var/log/syslog` o `/var/log/apache2/error.log` (si tienes acceso local) para ver si tus escaneos están llegando.
* **Conexión fallida en Metasploit:** Recuerda que el output de Nmap muestra puertos no estándar (ej. 4451 para SMB, 8002 para HTTP). Si olvidas cambiar la opción `RPORT` en Metasploit (que por defecto usa 445 u 80), el ataque fallará.
* **Verificación de Reglas:** Recuerda verificar las reglas de firewall/Security Groups; si el comando `check` da "Connection Refused", asegura que tu IP atacante esté en la lista blanca del laboratorio.
