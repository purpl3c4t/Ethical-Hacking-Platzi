# Phishing: Diseño y Detección de Campañas de Ingeniería Social

El phishing ha evolucionado desde correos electrónicos masivos con errores ortográficos evidentes hacia operaciones de ingeniería social de alta precisión. Para comprender cómo defenderse, es necesario deconstruir la metodología de un atacante. Una campaña de phishing exitosa no es un evento aislado, sino un proceso cíclico y estructurado.

En esta lección, analizaremos el ciclo de vida de una campaña de phishing y cómo la Inteligencia Artificial está transformando el panorama de amenazas.


---

> **⚠️ Disclaimer Ético:** Esta información se proporciona con fines exclusivamente educativos y para la formación de profesionales en ciberseguridad. La ejecución de estos ataques en redes sin autorización explícita es ilegal y penada por la ley. Practica siempre en tu propio laboratorio controlado.

---

## El Ciclo de Vida de una Campaña de Phishing

Para que un ataque sea efectivo, los actores maliciosos siguen una metodología similar a la "Kill Chain" o cadena de ataque, adaptada a la manipulación humana.

### Fase 1: Recolección de Información (Reconocimiento)
Antes de enviar el primer correo, el atacante debe conocer a su objetivo. Cuanta más información posea, más legítimo parecerá el ataque.

* **OSINT (Open Source Intelligence):** Recopilación de correos electrónicos corporativos, nombres de empleados clave y estructura jerárquica a través de LinkedIn y redes sociales.
* **Identificación Tecnológica:** Análisis de registros DNS (MX Records) para saber si la empresa usa Microsoft 365 o Google Workspace, lo que determina el diseño de la página de inicio de sesión falsa.
* **Análisis de Proveedores:** Identificar qué servicios externos utiliza la empresa (ej. Salesforce, DocuSign, Dropbox) para suplantar notificaciones legítimas de estas plataformas.

### Fase 2: Establecimiento de Confianza (Pretexting)
El objetivo es construir un escenario creíble ("Pretexto") y una infraestructura que pase las verificaciones superficiales del usuario.

* **Typosquatting y Dominios Homográficos:** Registro de dominios visualmente similares al legítimo (ej. `soporte-google.com` en lugar de `google.com`, o uso de caracteres cirílicos que se ven idénticos a los latinos).
* **Clonación de Portales:** Copia exacta del código HTML/CSS del sitio legítimo para que la víctima se sienta en un entorno familiar.
* **Certificados SSL:** Implementación de HTTPS en el sitio fraudulento. El candado verde ya no garantiza seguridad, solo cifra la conexión entre la víctima y el atacante.

### Fase 3: Compromiso y Explotación (Engagement)
Es el momento de contacto. El atacante envía el mensaje (Vector de entrega) esperando una acción del usuario (Call to Action).

* **El Gancho Emocional:** Se apela a la urgencia (cuenta bloqueada), curiosidad (sueldo de ejecutivos), miedo (citación judicial) o complacencia.
* **Evasión de Filtros:** Uso de técnicas para evitar que el correo caiga en Spam, como ofuscación de enlaces, uso de redirecciones abiertas legítimas o envío desde cuentas comprometidas reales.
* **La Acción:**
    * **Credential Harvesting:** El usuario ingresa sus credenciales en el formulario falso.
    * **Distribución de Malware:** El usuario descarga un archivo adjunto (Factura.pdf.exe o un documento Office con macros maliciosas).

### Fase 4: Cobertura de Huellas y Persistencia
Una vez obtenido el dato o el acceso, el atacante debe evitar levantar sospechas inmediatas.

* **Redirección Post-Login:** Una vez que la víctima ingresa la contraseña en el sitio falso, el script la guarda y redirige al usuario automáticamente al sitio web real legítimo. El usuario cree que simplemente hubo un error de sesión y vuelve a loguearse, sin sospechar el robo.
* **Rotación de Infraestructura:** Uso de dominios de vida corta y servidores proxy para que, cuando el equipo de seguridad detecte e intente bloquear la IP, el atacante ya se haya movido.

---

## AI Weaponization

La democratización de los Grandes Modelos de Lenguaje (LLM) y la IA generativa ha eliminado muchas de las barreras de entrada para los ciberdelincuentes, perfeccionando la calidad de los ataques.

### Generación de Texto Perfecto (LLMs)
Históricamente, la mala gramática y redacción eran indicadores clave de phishing. Herramientas como ChatGPT (o sus variantes sin censura en la Dark Web como WormGPT o FraudGPT) permiten a atacantes no nativos generar correos electrónicos con redacción corporativa perfecta, tono persuasivo y en cualquier idioma.

### Deepfakes de Audio y Video (Vishing Avanzado)
El Vishing (Phishing de voz) ha escalado peligrosamente.
* **Clonación de Voz:** Con solo unos segundos de audio de un CEO (obtenido de una entrevista en YouTube), la IA puede sintetizar su voz para llamar a un empleado de finanzas y autorizar una transferencia urgente.
* **Video Fake en Tiempo Real:** Uso de deepfakes en videollamadas para suplantar identidad en procesos de onboarding remoto o verificaciones de seguridad.

### Automatización y Personalización Masiva
La IA permite escalar el "Spear Phishing". Scripts automatizados pueden leer perfiles de miles de víctimas, entender su contexto laboral y generar miles de correos únicos y personalizados para cada uno, aumentando drásticamente la tasa de clics en comparación con campañas genéricas.

---

## Indicadores de Detección (Defensa)

La detección requiere una combinación de análisis técnico y humano.

### Análisis de Cabeceras (Headers)
* **Return-Path vs. From:** Verificar si la dirección de envío visible coincide con la dirección de retorno técnica.
* **Autenticación de Dominio:** Analizar los resultados de SPF (Sender Policy Framework), DKIM (DomainKeys Identified Mail) y DMARC. Un fallo en estas verificaciones indica que el remitente no está autorizado para usar ese dominio.

### Análisis de Enlaces y Adjuntos
* **Inspección de URL:** Pasar el cursor sobre el enlace sin hacer clic para revelar el destino real. Buscar inconsistencias o uso de acortadores de URL inusuales.
* **Sandboxing:** Ejecutar archivos adjuntos en un entorno aislado y seguro para observar su comportamiento (conexiones de red, cambios en el registro) antes de permitir que lleguen al usuario final.
