# [Challenge:0004]: [Footprinting: reconocimiento de información pública de sistemas]

| Información | Detalle |
| :--- | :--- |
| **Tema** | Footprinting: reconocimiento de información pública de sistemas |
| **Dificultad** | Básica / Intermedia |
| **Tiempo Estimado** | 2 Horas |
| **Tipo** | Black Box (Caja Negra) |

---

## 1. Contexto del Desafío

**Escenario:** Usted ha sido contratado como consultor de seguridad para realizar una auditoría de caja negra (Black Box) a una organización objetivo. Antes de lanzar cualquier escáner activo o interactuar directamente con sus servidores (lo cual generaría ruido y alertaría al Blue Team), necesita recopilar la mayor cantidad de información posible utilizando fuentes de acceso público (OSINT).

El objetivo es construir un perfil detallado de la superficie de exposición de la organización para alimentar la siguiente fase de Enumeración. Sin este paso, un ataque es ciego, ruidoso e ineficiente.

## 2. Objetivos

Al completar este desafío, serás capaz de:

1.  **Mapear la infraestructura de red:** Identificar rangos de IP, servidores de nombres (DNS) y proveedores de alojamiento sin enviar paquetes directos al objetivo.
2.  **Descubrir activos ocultos:** Localizar subdominios y servidores de desarrollo olvidados mediante el uso de motores de búsqueda avanzados.
3.  **Perfilar una organización:** Recopilar direcciones de correo electrónico y metadatos de documentos públicos para futuros ataques de ingeniería social o fuerza bruta.

## 3. Pre-requisitos

* Haber completado exitosamente la lectura del módulo "Footprinting: reconocimiento de información pública de sistemas".
* Tener acceso a un navegador web y una terminal Linux (Kali/Parrot) con acceso a internet.
* Herramientas sugeridas instaladas o accesibles: `whois`, `nslookup` (o `dig`), `theHarvester` y acceso a Google.

---

## 4. Instrucciones del Desafío

Para este desafío, utilizaremos el dominio de práctica `megacorpone.com` (propiedad de Offensive Security para fines educativos) o un objetivo de un programa público de Bug Bounty (como `hackerone.com`). *No realice estas acciones contra objetivos sin autorización.*

### Tarea 1: Reconocimiento de Infraestructura y Red
Debe identificar quién controla el dominio y dónde están alojados los servicios principales.

* **Instrucción:** Utilice herramientas de registro y consulta DNS para obtener los datos técnicos del dominio `megacorpone.com`.
* **Requisito A:** Identifique el Registrador del dominio y los Servidores de Nombres (Name Servers - NS) autoritativos.
* **Requisito B:** Obtenga los registros MX (Mail Exchange) para determinar qué proveedor gestiona el correo electrónico corporativo.
* **Requisito C:** Determine la dirección IP del servidor web principal y, utilizando servicios de geolocalización de IP (como `iplocation.net`), identifique el país y el ISP de alojamiento.

### Tarea 2: Descubrimiento de Subdominios y Google Dorking
Las vulnerabilidades suelen encontrarse en subdominios olvidados (como portales de prueba o VPNs antiguas).

* **Instrucción:** Utilice operadores avanzados de Google (Google Dorks) y herramientas de búsqueda pasiva para listar subdominios.
* **Requisito A:** Encuentre al menos 3 subdominios válidos (ej. `www`, `mail`, `test`, `dev`, `intranet`).
    * *Pista:* Use el operador `site:megacorpone.com -www` para filtrar la página principal.
* **Requisito B:** Utilice el operador `filetype:` para buscar documentos públicos (PDF, DOCX, XLSX) alojados en el dominio. Descargue uno (sin abrirlo directamente si no es en entorno seguro) y extraiga sus metadatos (Autor, Software de creación) usando herramientas como `exiftool`.

### Tarea 3: Recolección de Inteligencia Organizacional 
Para preparar ataques de contraseñas o ingeniería social, necesita nombres de usuarios reales.

* **Instrucción:** Utilice herramientas automatizadas de OSINT para recolectar direcciones de correo electrónico asociadas al dominio.
* **Requisito A:** Ejecute la herramienta `theHarvester` limitando la búsqueda a motores públicos (como Google, Bing o LinkedIn).
* **Requisito B:** Genere una lista de al menos 5 posibles nombres de usuario basados en los correos encontrados (ej. si el correo es `j.doe@megacorpone.com`, el usuario podría ser `jdoe` o `john.doe`).

---

## 5. Completitud

Para validar este desafío, debes generar un reporte simple (archivo de texto o markdown) con la información recopilada.

**Resultado final esperado:**

1.  **Información de Dominio:**
    * Registrador: [Nombre]
    * DNS Servers: [ns1.ejemplo.com, ns2.ejemplo.com]
    * Servidor de Correo (MX): [Proveedor/Dirección]
2.  **Activos Descubiertos:**
    * IP Principal: [Dirección IP]
    * Subdominios detectados: [Lista de subdominios]
3.  **Metadatos y Usuarios:**
    * Software detectado en metadatos: [Ej. Microsoft Word 2013]
    * Usuarios potenciales: [Lista de usuarios/correos]

**Comenta:**
*¿Qué riesgo de seguridad representa para una empresa tener documentos PDF con metadatos de "Autor" y "Versión de Software" expuestos públicamente en internet?*
