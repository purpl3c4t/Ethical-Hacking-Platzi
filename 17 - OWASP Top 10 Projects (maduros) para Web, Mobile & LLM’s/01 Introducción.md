# OWASP Top 10 Projects (Maduros) para Web, Mobile & LLM

La Open Web Application Security Project (OWASP) es una fundación sin fines de lucro que funciona como el estándar de facto en la industria de la seguridad de aplicaciones. Para un profesional que se inicia en la seguridad ofensiva, OWASP no es solo una lista de vulnerabilidades, sino la fuente principal de metodologías de prueba, estándares de verificación y entornos de entrenamiento.

En esta sesión, analizaremos los proyectos "Flagship" (Insignia) y "Mature" (Maduros) que estructuran el trabajo de un pentester en entornos Web, Móviles y de Inteligencia Artificial.

---

> **⚠️ Disclaimer Ético:** Esta información se proporciona con fines exclusivamente educativos y para la formación de profesionales en ciberseguridad. La ejecución de estos ataques en redes sin autorización explícita es ilegal y penada por la ley. Practica siempre en tu propio laboratorio controlado.

---

## Seguridad en Aplicaciones Web

El dominio web es el más extenso y documentado. Para un pentester, es crucial distinguir entre el documento de concienciación (Top 10) y la guía técnica de ejecución (WSTG).

### OWASP Top 10 (Web)
Es un documento de concienciación que clasifica los diez riesgos de seguridad más críticos para las aplicaciones web. Se actualiza periódicamente (la versión actual es 2021) basándose en datos de telemetría de la industria.
* **Propósito:** No es una guía de pentesting, sino una lista de prioridades para desarrolladores y gestores de riesgos.
* **Categorías Clave:** Incluye fallos como "A01:2021-Broken Access Control" (Control de Acceso Roto) y "A03:2021-Injection" (Inyección), que agrupan múltiples vectores de ataque técnicos.

--
## OWASP WSTG (Web Security Testing Guide)
Este es el recurso más importante para el pentester. Es la guía metodológica integral que describe **cómo** probar la seguridad de una aplicación web paso a paso.

* **Estructura:** Cubre todo el ciclo de vida, desde la recolección de información hasta la explotación.
* **Metodología:** Cada prueba tiene un código identificador (ej. *WSTG-INPV-05* para SQL Injection). Esto permite a los auditores estandarizar sus informes, citando el código específico de la prueba realizada.
* **Uso:** Un auditor junior debe utilizar la WSTG como una lista de verificación (checklist) para asegurar que no se ha omitido ninguna superficie de ataque durante la auditoría.

### Fase 1: Recolección de Información (Information Gathering)
Antes de atacar, se debe mapear la superficie.
* **Enumeración de Aplicaciones:** Identificar subdominios y aplicaciones asociadas.
* **Fingerprinting (Huella Digital):** Determinar la tecnología del servidor (Apache, Nginx, IIS), frameworks (React, Django, Laravel) y versiones específicas.
* **Spidering/Crawling:** Mapear la estructura de directorios y archivos ocultos (`robots.txt`, `sitemap.xml`, archivos `.bak` olvidados).
* **Puntos de Entrada:** Identificar todos los parámetros URL, formularios, cabeceras HTTP y Cookies que acepta la aplicación.

### Fase 2: Gestión de Identidad y Autenticación
Aquí se prueba la puerta de entrada.
* **Enumeración de Usuarios:** Verificar si la aplicación revela qué usuarios existen (diferentes mensajes de error entre "Usuario no encontrado" y "Contraseña incorrecta").
* **Fuerza Bruta y Bloqueo:** Probar si existe un mecanismo de bloqueo tras intentos fallidos.
* **Bypass de Autenticación:** Intentar acceder a funciones protegidas forzando la navegación directa (Forced Browsing) o manipulando tokens JWT (algoritmo 'None', secretos débiles).

### Fase 3: Validación de Entrada (Input Validation) - El núcleo de la explotación
Se basa en la premisa: "Toda entrada del usuario es maliciosa hasta que se demuestre lo contrario".
* **SQL Injection (SQLi):** Inyectar caracteres de control (`'`, `"`, `;`) en campos de entrada para manipular consultas a la base de datos. Se prueba a ciegas (Time-based/Boolean-based) o basándose en errores.
* **Cross-Site Scripting (XSS):** Inyectar scripts (`<script>alert(1)</script>`) para ver si el navegador los ejecuta. Se divide en Reflejado (en la URL), Almacenado (en la base de datos) y basado en DOM.
* **Command Injection:** Intentar ejecutar comandos del sistema operativo (`| whoami`, `; ls -la`) a través de inputs web.

### Fase 4: Pruebas de Lógica de Negocio
Estas vulnerabilidades no las detectan los escáneres automáticos. Requieren pensamiento humano.
* **Integridad del flujo de trabajo:** ¿Es posible saltarse el paso de pago e ir directamente a la confirmación de envío?
* **Manipulación de precios/cantidades:** ¿Qué sucede si compro "-1" artículos? ¿Se me abona dinero?
* **Race Conditions:** Realizar múltiples solicitudes simultáneas (usando hilos) para intentar usar un cupón de descuento de un solo uso múltiples veces antes de que la base de datos actualice el estado.

### Mobile Application Security Testing Guide (MASTG) 

Las pruebas móviles requieren un entorno híbrido: un ordenador para el análisis y un dispositivo móvil (generalmente "rooteado" o con Jailbreak) para la ejecución. La metodología se divide en dos grandes bloques: Estático y Dinámico.

#### Análisis Estático (Static Analysis - SAST)
Se analiza la aplicación en reposo, sin ejecutarla.
* **Ingeniería Inversa:**
    * **Android:** Descompilar el APK a código Smali o Java (usando herramientas como `jadx` o `apktool`).
    * **iOS:** Desensamblar el binario IPA (usando Ghidra o Hopper) para analizar el código ensamblador ARM.
* **Búsqueda de Secretos:** Buscar en el código fuente claves API, credenciales de AWS, o URLs de entornos de desarrollo (Staging/Dev) hardcodeadas.
* **Análisis del Manifiesto:** Revisar el `AndroidManifest.xml` (Android) o `Info.plist` (iOS) en busca de permisos excesivos, actividades exportadas inseguras (que pueden ser lanzadas por otras apps maliciosas) o configuraciones de backup inseguras.

#### Análisis Dinámico (Dynamic Analysis - DAST)
Se analiza la aplicación mientras se ejecuta en el dispositivo.
* **Bypass de Controles de Seguridad:**
    * **Root/Jailbreak Detection:** La app se cierra si detecta un teléfono modificado. El auditor usa herramientas como **Frida** u **Objection** para enganchar (hook) la función de detección y forzarla a devolver "Falso".
    * **SSL Pinning:** La app rechaza certificados que no sean el original, impidiendo el uso de proxies. El auditor inyecta scripts para deshabilitar esta validación y poder interceptar el tráfico.
* **Análisis de Almacenamiento Local:** Verificar qué datos guarda la app en el teléfono (SharedPreferences, Bases de datos SQLite, Realm). Buscar tokens de sesión, información personal (PII) o contraseñas guardadas en texto plano.
* **Intercepción de Tráfico:** Usar proxies (Burp Suite, Mitmproxy) para analizar las APIs que consume la app móvil, buscando las mismas vulnerabilidades que en la web (Inyecciones, IDORs).

### Pruebas de Seguridad para LLMs (AI Red Teaming)

Dado que no existe una "WSTG" madura para IA, la industria ha adoptado metodologías de "Red Teaming" específicas para modelos generativos, centradas en la manipulación del Prompt y el manejo de salidas.

#### Categoría 1: Prompt Injection & Jailbreaking
El objetivo es engañar al modelo para que ignore sus directrices de seguridad (System Prompt).
* **Ataques Directos (DAN - Do Anything Now):** Crear contextos de rol donde el modelo cree que está en un "modo desarrollador" o "modo sin restricciones" para forzarlo a generar contenido prohibido (odio, creación de malware, instrucciones de bombas).
* **Ataques de Traducción/Codificación:** Pedir al modelo que genere contenido dañino pero codificado en Base64 o escrito en un idioma con pocos recursos de entrenamiento, donde los filtros de seguridad suelen ser más débiles.

#### Categoría 2: Insecure Output Handling (XSS vía LLM)
El LLM genera texto que luego es renderizado por una aplicación web.
* **Prueba:** Pedir al LLM que genere una respuesta que incluya payloads de Javascript. Si la aplicación web que muestra la respuesta del chat no sanitiza la salida, el código se ejecutará en el navegador de la víctima. Esto es una vulnerabilidad de "Cross-Site Scripting persistente generado por IA".

#### Categoría 3: Extracción de Datos de Entrenamiento
Intentar recuperar información memorizada por el modelo.
* **Ataques de Membresía:** Realizar consultas específicas para ver si el modelo completa información privada (ej. "El número de seguridad social de Juan Pérez que vive en Madrid es...").
* **Divergencia:** Forzar al modelo a repetir una palabra indefinidamente. En algunos modelos, esto provoca que el sistema "alucine" y comience a escupir datos crudos de su entrenamiento (correos, teléfonos) para salir del bucle.

#### Categoría 4: Consumo de Recursos (Model DoS)
Atacar la infraestructura que soporta el modelo.
* **Context Window Overflow:** Enviar prompts extremadamente largos o complejos que obliguen al modelo a procesar la máxima cantidad de tokens permitidos, buscando saturar la GPU o aumentar la latencia del servicio para otros usuarios.

---

## Proyectos Complementarios para Principiantes
* **OWASP Juice Shop:** Es la aplicación web vulnerable más moderna y sofisticada para entrenamiento. A diferencia de laboratorios antiguos, Juice Shop utiliza una arquitectura moderna (Node.js, Angular) que refleja las aplicaciones reales del mercado. Es vital para practicar la explotación de vulnerabilidades del Top 10 en un entorno seguro.
* **OWASP ZAP (Zed Attack Proxy):** Es un escáner de seguridad web gratuito y de código abierto. Funciona como un proxy de interceptación (similar a Burp Suite) que permite analizar y modificar el tráfico entre el navegador y la aplicación web.

### Seguridad en Aplicaciones Móviles

El ecosistema móvil requiere un enfoque diferente debido a la interacción con el sistema operativo del dispositivo (Android/iOS) y el almacenamiento local. OWASP unificó sus esfuerzos en el proyecto MAS (Mobile Application Security).

#### OWASP MASVS (Mobile Application Security Verification Standard)
Define "qué" se debe probar. Es un estándar que establece niveles de seguridad según la criticidad de la aplicación.
* **Nivel L1 (Estándar):** Requisitos de seguridad básicos que todas las apps móviles deben cumplir (ej. uso de HTTPS, no guardar contraseñas en texto claro).
* **Nivel L2 (Defensa en Profundidad):** Para apps que manejan datos altamente sensibles (banca, salud). Requiere controles avanzados como SSL Pinning y detección de Root/Jailbreak.
* **Nivel R (Resiliencia):** Protecciones contra ingeniería inversa y manipulación de código.

#### Seguridad en LLM (Large Language Models)

Con el auge de la Inteligencia Artificial Generativa, OWASP lanzó un nuevo proyecto Top 10 específico para abordar los riesgos únicos de esta tecnología, que difieren de las vulnerabilidades web tradicionales.

##### OWASP Top 10 for LLM Applications
Este proyecto identifica las vulnerabilidades críticas en aplicaciones que integran modelos como GPT-4, Llama o Claude.

* **LLM01: Prompt Injection:** La vulnerabilidad insignia. Ocurre cuando un atacante manipula la entrada (prompt) para que el modelo ignore sus instrucciones de seguridad y realice acciones no deseadas. Se divide en inyección directa (Jailbreaking) e indirecta (el LLM lee un correo o web maliciosa).
* **LLM02: Insecure Output Handling:** Ocurre cuando la salida generada por el LLM se acepta ciegamente por otros componentes del sistema sin validación, lo que puede llevar a XSS o ejecución de código en el backend.
* **LLM06: Sensitive Information Disclosure:** El riesgo de que el modelo revele datos confidenciales con los que fue entrenado o que filtre información privada de otros usuarios en la sesión.

---

## Seguridad Ofensiva starter pack de OWASP

Para quien inicia en seguridad ofensiva, la ruta de estudio recomendada basada en estos proyectos es:

1.  **Fundamentos:** Leer el **OWASP Top 10 Web** para entender los riesgos.
2.  **Práctica:** Instalar **OWASP Juice Shop** y usar **OWASP ZAP** para encontrar y explotar esos riesgos.
3.  **Metodología:** Estudiar la **OWASP WSTG** para aprender a documentar y ejecutar pruebas de manera profesional y ordenada.
4.  **Especialización:** Una vez dominada la web, avanzar hacia **OWASP MASTG** (Móvil) o el **Top 10 for LLM** (IA) según el interés de carrera.
