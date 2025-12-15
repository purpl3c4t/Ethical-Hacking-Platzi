# Framework Metasploit para pruebas de penetración y explotación

El Metasploit Framework (MSF) es la plataforma de desarrollo de exploits más utilizada en el mundo. Más que una simple herramienta, es un ecosistema modular basado en Ruby que permite a los profesionales de seguridad desarrollar, probar y ejecutar código de explotación contra objetivos remotos. Su diseño facilita la estandarización del proceso de intrusión, separando la carga útil (payload) del exploit en sí mismo.


---

> **⚠️ Disclaimer Ético:** Esta información se proporciona con fines exclusivamente educativos y para la formación de profesionales en ciberseguridad. La ejecución de estos ataques en redes sin autorización explícita es ilegal y penada por la ley. Practica siempre en tu propio laboratorio controlado.

---


## Arquitectura y Estructura del Framework

Para operar Metasploit de manera efectiva, es fundamental comprender cómo organiza sus componentes internos. La arquitectura modular permite combinar diferentes piezas de código para adaptarse a casi cualquier escenario de ataque.

### Interfaces Principales
Aunque el núcleo es el mismo, el usuario interactúa con el framework a través de diferentes interfaces:
* **msfconsole:** Es la interfaz de línea de comandos (CLI) principal. Es el método estándar para interactuar con todas las funcionalidades del framework.
* **msfvenom:** Herramienta independiente utilizada para generar y codificar cargas útiles (payloads) en diversos formatos (exe, elf, apk, python, etc.) sin necesidad de lanzar la consola principal.

### Los Módulos de Metasploit
El framework organiza sus scripts en categorías funcionales. Comprender esta taxonomía es vital para la navegación:

#### A. Exploits
Son fragmentos de código diseñados para aprovechar una vulnerabilidad específica (buffer overflow, inyección SQL, ejecución remota de comandos) en un sistema objetivo. El exploit es el "vehículo" que transporta la carga útil a través de la brecha de seguridad.

#### B. Payloads (Cargas Útiles)
Es el código que se ejecuta en el sistema objetivo una vez que el exploit ha tenido éxito. Se dividen en tres tipos principales:
* **Singles (Stageless):** Cargas autónomas que contienen todo el código necesario. Son más estables pero de mayor tamaño.
* **Stagers:** Pequeños fragmentos de código cuya única función es establecer una conexión de red y descargar el resto de la carga útil. Son vitales cuando el espacio de memoria en el buffer overflow es limitado.
* **Stages:** Los componentes más grandes (como Meterpreter o VNC) que son descargados por el Stager.

#### C. Auxiliary (Auxiliares)
Módulos que no están diseñados para obtener una shell, sino para realizar tareas de apoyo. Incluyen:
* Escáneres de puertos y servicios.
* Sniffers de tráfico.
* Herramientas de fuzzing.
* Módulos de denegación de servicio (DoS).

#### D. Post (Post-Explotación)
Módulos diseñados para ejecutarse *después* de haber comprometido el sistema. Se utilizan para:
* Escalada de privilegios.
* Recolección de evidencias (hashdump, capturas de pantalla).
* Movimiento lateral.
* Persistencia.

#### E. Encoders (Codificadores)
Se utilizan para ofuscar el payload con el objetivo de evadir firmas de antivirus o para eliminar "bad characters" (caracteres nulos o restringidos) que podrían corromper el exploit durante la transmisión.

---

## 2. Metodología de Pentesting a partir del Framework

El uso de Metasploit no debe ser aislado. Se integra en una metodología estructurada que va desde el reconocimiento hasta la post-explotación, apoyándose en herramientas externas como Nmap y Searchsploit.

### Fase 1: Reconocimiento y Base de Datos (Workspace)
Metasploit utiliza una base de datos PostgreSQL para almacenar los resultados del reconocimiento. Esto permite gestionar proyectos complejos con múltiples objetivos.

* **Inicialización:** Se debe iniciar la base de datos (`msfdb init`) antes de operar.
* **Integración con Nmap:** Dentro de la consola (`msfconsole`), se utiliza el comando `db_nmap`. Esto ejecuta Nmap y guarda automáticamente los hosts, puertos y servicios descubiertos directamente en la base de datos de Metasploit.

### Fase 2: Análisis de Vulnerabilidades (Scripting y Searchsploit)
Una vez identificados los puertos abiertos y las versiones de servicio, se debe determinar si son vulnerables.

1.  **Validación con Nmap Scripts (NSE):** Antes de lanzar un exploit, es recomendable confirmar la vulnerabilidad. Metasploit permite invocar escáneres auxiliares, pero a menudo se utilizan los scripts NSE de Nmap para una detección ligera.
2.  **Correlación con Searchsploit:** `searchsploit` es la herramienta de línea de comandos para Exploit-DB. Permite buscar exploits conocidos para el software detectado.
    * *Flujo:* Si `searchsploit` indica que existe un exploit para "Apache Struts 2", el auditor buscará el módulo equivalente en Metasploit usando el comando `search struts` dentro de la consola.

### Fase 3: Selección y Configuración del Exploit
Tras confirmar la vulnerabilidad, se selecciona el módulo correspondiente (`use`). La configuración requiere precisión en los parámetros:

* **RHOSTS (Remote Hosts):** La dirección IP del objetivo.
* **LHOST (Local Host):** La dirección IP del atacante (es crucial para las conexiones inversas).
* **Target:** Define el sistema operativo y la versión exacta de la víctima (ej: Windows 7 SP1 - Español). Un target incorrecto puede hacer fallar el exploit o colgar el servicio remoto (crash).

**Concepto de Shell: Bind vs Reverse**
* **Bind Shell:** El atacante abre un puerto en la víctima y se conecta a él. (Problemático si hay firewalls).
* **Reverse Shell:** La víctima se conecta hacia el atacante. Es el método preferido para evadir firewalls que bloquean el tráfico entrante pero permiten el saliente.

### Fase 4: Explotación y Post-Explotación (Meterpreter)
Al ejecutar el comando `exploit`, si el ataque es exitoso, se obtiene una sesión. La carga útil más avanzada de Metasploit es **Meterpreter**.

**Meterpreter (The Meta-Interpreter):**
Es un payload avanzado que opera completamente en la memoria del sistema víctima (in-memory injection), evitando escribir archivos en el disco duro, lo que reduce la probabilidad de detección forense.

---

Funcionalidades clave de post-explotación:

* **Migración de procesos:** Moverse de un proceso inestable (como un navegador explotado) a uno estable (como `explorer.exe`).
* **Getsystem:** Intentar la elevación de privilegios automatizada.
* **Hashdump:** Extraer los hashes de las contraseñas del sistema.
* **Pivoting:** Utilizar la máquina comprometida como un enrutador para atacar otras máquinas en la red interna que no son visibles desde el exterior.

---
