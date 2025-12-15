# Footprinting: Reconocimiento de Información Pública de Sistemas

El éxito de un ciberataque o una auditoría de seguridad no depende de cuán rápido escribas en el teclado, sino de **cuánto sabes sobre tu objetivo**. En esta clase, exploraremos el **Footprinting** (huella digital), la primera fase del hacking ético, donde el objetivo es recolectar tanta información como sea posible para mapear la superficie de ataque.

---

> **⚠️ Disclaimer Ético:** Esta información se proporciona con fines exclusivamente educativos y para la formación de profesionales en ciberseguridad. La ejecución de estos ataques en redes sin autorización explícita es ilegal y penada por la ley. Practica siempre en tu propio laboratorio controlado.

---

**La Regla del 90/10:** Un pentester profesional pasa el 90% del tiempo investigando y planificando, y solo el 10% ejecutando el ataque.

---

##  1. ¿Qué es el Footprinting?

El Footprinting es el proceso metodológico de recolectar información sobre un sistema, red u organización objetivo. Su finalidad es crear un "mapa" completo de la infraestructura de seguridad, identificar puntos de entrada y entender la tecnología que utiliza la víctima.

Existen dos enfoques principales:
* **Reconocimiento Pasivo:** Recolectar información sin interactuar directamente con los sistemas del objetivo (el objetivo no sabe que lo estás investigando).
* **Reconocimiento Activo:** Interactuar directamente con el sistema (tocar la puerta), lo que deja rastro en los logs y podría alertar a los sistemas de seguridad (IDS/IPS).

---

## 2. External Footprinting (Desde Internet)

El **External Footprinting** asume el rol de un atacante remoto que no tiene acceso a la red interna ni a las instalaciones físicas. Todo se hace desde "afuera", utilizando internet.

### Conceptos Clave y Fuentes:

#### A. OSINT (Open Source Intelligence)
Es la inteligencia obtenida de fuentes de acceso público.
* **Redes Sociales:** LinkedIn (para encontrar empleados y tecnologías que usan), Twitter/X.
* **Sitios Corporativos:** Ofertas de empleo (revelan versiones de software: *"Buscamos experto en Windows Server 2019"*), noticias financieras.

#### B. La Infraestructura de Internet (DNS y WHOIS)
* **WHOIS:** Es el registro público de dominios. Nos dice quién registró el dominio `empresa.com`, su dirección física, correo de contacto y servidores de nombres.
* **Enumeración DNS:** El DNS es la guía telefónica de internet. Buscamos subdominios (`dev.empresa.com`, `vpn.empresa.com`) que a menudo son menos seguros que la página principal.

#### C. Hacking con Motores de Búsqueda (Google Dorks)
Usar operadores avanzados de búsqueda para encontrar información sensible que fue indexada por error.
* *Ejemplo:* `site:empresa.com filetype:pdf "confidential"` para buscar documentos PDF confidenciales expuestos públicamente.

#### D. Motores de Búsqueda de IoT (Shodan / Censys)
A diferencia de Google (que busca webs), **Shodan** busca dispositivos conectados a internet. Permite ver si la empresa tiene cámaras, servidores, routers o sistemas industriales expuestos públicamente sin necesidad de enviar un solo paquete nosotros mismos.

---

## 3. Internal Footprinting (Desde Adentro)

El **Internal Footprinting** ocurre cuando el atacante ya ha logrado algún nivel de acceso (ej: un empleado descontento, un visitante conectado al Wi-Fi de invitados, o una máquina comprometida remotamente). El objetivo cambia: ya no buscamos entrar, sino **movernos lateralmente**.

### Conceptos Clave y Objetivos:

#### A. Escucha Pasiva (Sniffing)
Antes de hacer ruido, escuchamos. Al capturar tráfico de la red interna (Wireshark/TCPDump), podemos identificar:
* Rangos de direcciones IP en uso.
* Protocolos de broadcast (ARP, DHCP) que revelan la ubicación de impresoras y servidores.

#### B. Mapeo de la Red Interna
Determinar la topología de la red.
* ¿Dónde está la puerta de enlace (Gateway)?
* ¿Existen subredes separadas para Finanzas y RRHH?
* **Traceroute Interno:** Para ver cuántos saltos hay hasta el servidor principal.

#### C. Enumeración de Servicios Internos
Buscar activos que no son visibles desde internet:
* **Directorios Activos (LDAP/AD):** Identificar el controlador de dominio es el "Santo Grial" interno.
* **Servidores de Archivos (SMB/NFS):** Carpetas compartidas donde los empleados guardan información.
* **Intranets y Portales de Empleados:** Sitios web internos que suelen tener seguridad más relajada que la web pública.

---

##  4. Metodología General de Footprinting

Para que el proceso sea efectivo y profesional, debe seguir un ciclo estructurado:

1.  **Definir el Alcance (Scope):** ¿Qué empresa o red estamos investigando?
2.  **Recolección de Información (Information Gathering):**
    * Consultar WHOIS y DNS.
    * Buscar en motores de búsqueda (Google/Shodan).
    * Extraer metadatos de documentos públicos.
3.  **Análisis y Correlación:** Unir los puntos.
    * *Ejemplo:* "En LinkedIn vi que el administrador usa Ubuntu, y en Shodan veo un servidor con puerto 22 abierto en su rango de IP. Probablemente es un servidor SSH Linux".
4.  **Mapeo de la Superficie de Ataque:** Crear un diagrama con todos los puntos de entrada potenciales (IPs, dominios, correos electrónicos de empleados) para pasar a la siguiente fase: el Escaneo.

---

### Compración: Externo vs Interno

| Característica | External Footprinting | Internal Footprinting |
| :--- | :--- | :--- |
| **Posición** | Desde Internet (Remoto) | Desde la LAN (Local/Físico) |
| **Técnica Principal** | OSINT, DNS, Shodan | Sniffing, Escaneo de puertos, LDAP |
| **Riesgo de Detección** | Muy Bajo (Pasivo) | Medio/Alto (Activo) |
| **Objetivo** | Encontrar una IP/Dominio vulnerable | Mapear la red y escalar privilegios |
