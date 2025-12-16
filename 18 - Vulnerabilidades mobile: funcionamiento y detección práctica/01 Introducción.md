# Vulnerabilidades Mobile: Funcionamiento, Detección y Metodología OWASP

A diferencia de las aplicaciones web, donde el código reside y se ejecuta principalmente en un servidor remoto bajo el control de la empresa, las aplicaciones móviles se descargan y ejecutan en el dispositivo del usuario. Esto entrega el código binario directamente al atacante, permitiendo técnicas de ingeniería inversa y manipulación en tiempo real.
Para abordar este entorno hostil, la industria utiliza el **OWASP Mobile Application Security Verification Standard (MASVS)** como referencia de requisitos y la **OWASP Mobile Application Security Testing Guide (MASTG)** como manual de ejecución. 
Al igual que la clase anterior, esta clase es una introducción para alinearnos a una metodología con la seguridad de que estamos haciendo las pruebas correspondientes y suficientes, apoyados en una organización global y con herramientas para cada fase.

---

> **⚠️ Disclaimer Ético:** Esta información se proporciona con fines exclusivamente educativos y para la formación de profesionales en ciberseguridad. La ejecución de estos ataques en redes sin autorización explícita es ilegal y penada por la ley. Practica siempre en tu propio laboratorio controlado.

---

## Fase 1: Recolección de Información y Arquitectura

Antes de iniciar el análisis técnico, se debe comprender el ecosistema de la aplicación.

* **Objetivo:** Determinar la tecnología de desarrollo, la superficie de ataque y los endpoints del servidor con los que se comunica la app.
* **Entradas:** Archivo instalador (APK para Android, IPA para iOS), enlace a la tienda de aplicaciones.
* **Técnicas:**
    * **Identificación del Framework:** Determinar si es una app Nativa (Java/Kotlin, Swift/Objective-C) o Híbrida (Flutter, React Native, Xamarin). Esto define las herramientas de ingeniería inversa a utilizar.
    * **OSINT:** Buscar versiones antiguas de la app que puedan tener menos protecciones de seguridad.
* **Herramientas:**
    * **MobSF (Mobile Security Framework):** Herramienta "todo en uno" que realiza un escaneo automático inicial, identificando permisos, URLs y tecnologías.
* **Salidas:** Reporte de reconocimiento, mapa de la arquitectura (Cliente-API).

---

## Fase 2: Análisis Estático (Static Application Security Testing - SAST)

Esta fase se centra en examinar el código fuente o el binario compilado sin ejecutar la aplicación. Dado que el atacante posee el archivo, puede intentar entender su lógica interna.

* **Objetivo:** Encontrar credenciales embebidas (Hardcoded Secrets), lógica de negocio insegura y configuraciones erróneas en el manifiesto.
* **Entradas:** Archivo APK o IPA desencriptado.
* **Técnicas:**
    * **Decompilación/Desensamblaje:** Convertir el código máquina (Bytecode/DEX) nuevamente a un formato legible (Java o Smali) para leer la lógica.
    * **Análisis del Manifiesto:** Revisar el archivo `AndroidManifest.xml` en busca de componentes exportados (Activities, Receivers) que permitan a otras apps maliciosas interactuar con la nuestra.
    * **Búsqueda de Secretos:** Utilizar expresiones regulares (Grepping) para encontrar claves API, tokens de AWS o URLs de entornos de desarrollo.
* **Herramientas:**
    * **jadx-gui:** Decompilador estándar para Android (convierte APK a Java).
    * **apktool:** Para obtener el código Smali y recursos.
    * **Ghidra:** Para análisis de librerías nativas (.so) escritas en C/C++.
* **Salidas:** Código fuente reconstruido, lista de secretos expuestos.

---

## Fase 3: Análisis Dinámico (Dynamic Application Security Testing - DAST)

Aquí se ejecuta la aplicación en un dispositivo real o emulador para analizar su comportamiento en tiempo real, manipular su memoria e interceptar su tráfico.

* **Objetivo:** Eludir controles de seguridad (Root Detection, SSL Pinning) y manipular la lógica en tiempo de ejecución.
* **Entradas:** Dispositivo móvil con privilegios elevados (Rooted Android / Jailbroken iOS).
* **Técnicas:**
    * **Instrumentación (Hooking):** Inyectar scripts en el proceso de la aplicación mientras corre para modificar su comportamiento. Por ejemplo, si una función `isRooted()` devuelve `True`, se inyecta un script para que siempre devuelva `False`.
    * **Bypass de SSL Pinning:** Deshabilitar la validación de certificados para poder interceptar el tráfico HTTPS con un proxy.
    * **Análisis de Logs:** Monitorear `Logcat` (Android) o la consola del sistema (iOS) para ver si la app imprime información sensible durante su uso.
* **Herramientas:**
    * **Frida:** Motor de instrumentación dinámica. Permite escribir scripts en JavaScript para interactuar con la memoria de la app.
    * **Objection:** Kit de herramientas basado en Frida que permite realizar acciones complejas (como bypass de SSL Pinning) con simples comandos, ideal para principiantes.
* **Salidas:** Acceso a tráfico cifrado, elusión de controles de seguridad.

---

## Fase 4: Almacenamiento Local Inseguro

Los dispositivos móviles se pierden o son robados. Si la seguridad de los datos depende de que nadie tenga acceso físico al teléfono, la seguridad es nula.

* **Objetivo:** Verificar si la aplicación guarda datos sensibles sin cifrar en el sistema de archivos del dispositivo.
* **Entradas:** Acceso al sistema de archivos del dispositivo (vía ADB o SSH).
* **Técnicas:**
    * **Exploración de Directorios:** Navegar a `/data/data/[paquete.app]` en Android.
    * **Análisis de Bases de Datos:** Abrir archivos `.sqlite` o `.db` para buscar tablas con información de usuario, cookies o tokens de sesión.
    * **Archivos de Preferencias:** Revisar `SharedPreferences` (XML) buscando configuraciones o datos de usuario en texto plano.
* **Herramientas:**
    * **ADB (Android Debug Bridge):** Para acceder a la shell del dispositivo.
    * **SQLite Browser:** Para leer las bases de datos extraídas.
* **Salidas:** Evidencia de datos personales (PII) o credenciales almacenadas en texto plano.

---

## Fase 5: Seguridad en la Comunicación de Red

Aunque el servidor sea seguro, si el canal de comunicación es vulnerable, los datos pueden ser interceptados.

* **Objetivo:** Interceptar y manipular las peticiones entre el móvil y el servidor (API).
* **Entradas:** Configuración de Proxy en el dispositivo móvil apuntando al ordenador del auditor.
* **Técnicas:**
    * **Man-in-the-Middle (MitM):** Instalar el certificado CA del auditor en el dispositivo móvil para descifrar el tráfico HTTPS.
    * **Manipulación de Parámetros:** Una vez interceptado el tráfico en el Proxy, modificar valores (precios, IDs de usuario) antes de enviarlos al servidor.
* **Herramientas:**
    * **Burp Suite / OWASP ZAP:** Proxies de interceptación.
* **Salidas:** Flujos HTTP descifrados, vulnerabilidades de API detectadas.

---

## Resumen Metodológico 

| Fase | Enfoque | Herramienta Clave (Open Source) | Acción Principal |
| :--- | :--- | :--- | :--- |
| **1. Reconocimiento** | Automático | MobSF | Escaneo rápido para ver permisos y estructura. |
| **2. Estático (SAST)** | Ingeniería Inversa | jadx-gui | Leer el código Java para buscar claves API. |
| **3. Dinámico (DAST)** | Manipulación | Frida / Objection | "Engañar" a la app para saltar protecciones. |
| **4. Almacenamiento** | Forense | ADB (Shell) | Buscar contraseñas en archivos locales XML/DB. |
| **5. Red** | Intercepción | Burp Suite | Leer y modificar el tráfico hacia el servidor. |

## Prevención en el Desarrollo

Para mitigar estas vulnerabilidades, los desarrolladores deben aplicar controles específicos:

1.  **Ofuscación de Código:** Utilizar herramientas como ProGuard o R8 para hacer que el código decompilado sea difícil de entender, renombrando clases y variables a letras sin sentido.
2.  **Almacenamiento Seguro:** Utilizar `EncryptedSharedPreferences` (Android) o Keychain (iOS) para guardar datos sensibles, nunca archivos de texto plano.
3.  **SSL Pinning:** Implementar validación estricta del certificado del servidor dentro de la app para prevenir ataques de Man-in-the-Middle.
4.  **Detección de Entorno:** Implementar controles (SafetyNet / Play Integrity API) para verificar si el dispositivo está rooteado o comprometido y limitar la funcionalidad de la app en esos casos.
