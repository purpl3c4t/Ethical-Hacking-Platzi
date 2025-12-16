# Vulnerabilidades Web: Funcionamiento, Detección y Metodología OWASP

El desarrollo de aplicaciones web seguras y la auditoría de las mismas requieren un enfoque estandarizado. Intentar encontrar vulnerabilidades de manera aleatoria es ineficiente y arriesgado. Para ello, la industria se apoya en la **OWASP Web Security Testing Guide (WSTG)**, que proporciona un marco de trabajo integral para realizar pruebas de penetración.
Aunque en la generalidad, cada pentester encuentra el órden de una metodología que le permita confiar en que las pruebas son exhaustivas y consistentes, esta es puede ser una primera aproximación para saber cómo iniciar y estar seguros de que es una metodología probada, con fundamentos en guías internacionales. 

A continuación, se detalla una metodología paso a paso diseñada para principiantes, desglosando cada fase por sus objetivos, entradas, técnicas y herramientas.

---

> **⚠️ Disclaimer Ético:** Esta información se proporciona con fines exclusivamente educativos y para la formación de profesionales en ciberseguridad. La ejecución de estos ataques en redes sin autorización explícita es ilegal y penada por la ley. Practica siempre en tu propio laboratorio controlado.

---

## Fase 1: Recolección de Información (Information Gathering)

Antes de buscar vulnerabilidades, es necesario comprender la superficie de ataque. El objetivo es mapear la aplicación completa y la infraestructura que la soporta.

* **Objetivo:** Identificar todas las posibles vías de entrada, tecnologías utilizadas y estructura lógica de la aplicación.
* **Entradas:** URL principal (Scope), direcciones IP, nombre de la empresa.
* **Técnicas:**
    * **Fingerprinting del Servidor:** Identificar el sistema operativo, servidor web (Apache, Nginx, IIS) y tecnologías backend (PHP, Java, Python).
    * **Enumeración de Subdominios:** Buscar entornos de desarrollo, prueba o administración olvidados.
    * **Spidering/Crawling:** Navegación automática para descubrir todas las páginas, imágenes y scripts enlazados.
    * **Fuerza Bruta de Directorios:** Buscar archivos ocultos que no están enlazados públicamente (ej. `/backup`, `.git`, `/admin`).
* **Herramientas:**
    * Nmap (Escaneo de puertos y servicios).
    * Wappalyzer (Identificación de tecnologías en el navegador).
    * Gobuster / Dirb (Búsqueda de directorios ocultos).
    * OWASP ZAP / Burp Suite (Spidering).
* **Salidas:** Mapa del sitio, lista de directorios ocultos, versiones de software detectadas.
---

## Fase 2: Modelado de Amenazas y Análisis de Configuración

Con la información recolectada, se analiza la seguridad de la infraestructura y se priorizan las áreas críticas.

* **Objetivo:** Verificar si el servidor y la aplicación tienen configuraciones de seguridad robustas antes de atacar el código.
* **Entradas:** Cabeceras HTTP, certificados SSL/TLS, versiones de software obtenidas en la Fase 1.
* **Técnicas:**
    * **Análisis de Cabeceras de Seguridad:** Verificar la presencia de HSTS, X-Frame-Options, Content-Security-Policy (CSP).
    * **Verificación de SSL/TLS:** Buscar cifrados débiles o certificados caducados.
    * **Pruebas de Métodos HTTP:** Verificar si métodos peligrosos como `PUT` o `DELETE` están habilitados innecesariamente.
* **Herramientas:**
    * TestSSL.sh (Auditoría TLS).
    * Nikto (Escáner de vulnerabilidades de servidor web).
    * Burp Suite (Análisis de respuestas).
* **Salidas:** Informe de configuraciones inseguras, lista de métodos HTTP permitidos.
---

## Fase 3: Gestión de Identidad y Autenticación

Esta fase evalúa cómo la aplicación verifica quién es el usuario y cómo protege sus credenciales. Es uno de los puntos de fallo más comunes.

* **Objetivo:** Eludir los mecanismos de inicio de sesión o acceder a cuentas de otros usuarios.
* **Entradas:** Formularios de login, funciones de recuperación de contraseña, formularios de registro.
* **Técnicas:**
    * **Enumeración de Usuarios:** Analizar si los mensajes de error revelan la existencia de cuentas (ej. "Usuario no encontrado" vs "Contraseña incorrecta").
    * **Fuerza Bruta / Credential Stuffing:** Probar diccionarios de contraseñas comunes.
    * **Análisis de la Calidad de la Contraseña:** Verificar si se permiten contraseñas débiles (ej. "123456").
    * **Prueba de "Olvidé mi contraseña":** Evaluar si el mecanismo de recuperación es predecible o inseguro.
* **Herramientas:**
    * Hydra (Fuerza bruta).
    * Burp Suite Intruder (Automatización de intentos).
* **Salidas:** Credenciales válidas, usuarios identificados, debilidades en la política de contraseñas.
---

## Fase 4: Validación de Entrada (Input Validation)

El núcleo del pentesting web. Se basa en la premisa de que cualquier dato enviado por el usuario puede ser malicioso. Aquí se buscan las vulnerabilidades técnicas más críticas (OWASP Top 10).

* **Objetivo:** Inyectar código malicioso para manipular el comportamiento del servidor o del navegador.
* **Entradas:** Todos los campos de entrada identificados: parámetros URL (`?id=1`), campos de formulario, Cookies, Cabeceras HTTP (User-Agent).
* **Técnicas:**
    * **SQL Injection (SQLi):** Inserción de comandos SQL para leer o modificar la base de datos.
        * *Prevención:* Uso de consultas parametrizadas (Prepared Statements).
    * **Cross-Site Scripting (XSS):** Inyección de scripts JavaScript que se ejecutan en el navegador de la víctima.
        * *Prevención:* Codificación de salida (Output Encoding) y validación de entrada.
    * **Command Injection:** Ejecución de comandos del sistema operativo.
* **Herramientas:**
    * SQLMap (Automatización de SQLi).
    * OWASP ZAP / Burp Suite Scanner (Escaneo activo).
    * Listas de Payloads (Secclists).
* **Salidas:** Evidencia de ejecución de código, extracción de datos de la base de datos.

---

## Fase 5: Control de Acceso y Gestión de Sesiones

Una vez autenticado, ¿qué puede hacer el usuario? ¿Puede ver datos de otros?

* **Objetivo:** Escalar privilegios (de usuario a administrador) o acceder horizontalmente a datos de otros usuarios.
* **Entradas:** Cookies de sesión, Tokens JWT, URLs de áreas restringidas.
* **Técnicas:**
    * **IDOR (Insecure Direct Object Reference):** Cambiar identificadores en la URL (ej. cambiar `id=100` a `id=101`) para ver si se accede al perfil de otro usuario.
    * **Predicción de Sesiones:** Analizar si las cookies de sesión son aleatorias o siguen un patrón predecible.
    * **Fijación de Sesión:** Verificar si la sesión cambia tras el inicio de sesión.
* **Herramientas:**
    * Burp Suite Sequencer (Análisis de entropía de tokens).
    * Complementos de navegador (Cookie Editor).
* **Salidas:** Acceso no autorizado a recursos, elevación de privilegios.

## Resumen Metodológico 

La intención de esta metodología es que sea lo más exhaustiva posible, evitando la dependencia exclusiva de herramientas automatizadas. Pero claro, que el primer la comprensión lógica de las vulnerabilidades.
En la tabla están descritas las acciones clave a realizar en cada fase:

| Fase | Herramienta Principal (Open Source/Free) | Acción Clave |
| :--- | :--- | :--- |
| **1. Recolección** | Nmap / Wappalyzer | Mapear la tecnología y estructura. |
| **2. Configuración** | Nikto | Buscar servidores desactualizados o mal configurados. |
| **3. Autenticación** | Burp Suite (Proxy) | Intentar enumerar usuarios y probar contraseñas débiles. |
| **4. Validación (Inyección)** | OWASP ZAP | Probar inputs con caracteres especiales (`' " < >`). |
| **5. Control de Acceso** | Navegador (DevTools) | Manipular IDs y Cookies para ver datos ajenos. |

