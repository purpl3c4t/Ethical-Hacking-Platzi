#  Detección Automatizada de Vulnerabilididades

En el vasto océano de la ciberseguridad, intentar encontrar cada fallo de seguridad manualmente es como buscar una aguja en un pajar con los ojos vendados. Los **escáneres de vulnerabilidades** son nuestros detectores de metales: herramientas automatizadas diseñadas para interrogar masivamente a los sistemas y reportar posibles fallos de seguridad basándose en una base de datos de firmas conocidas.

---

> **⚠️ Disclaimer Ético:** Esta información se proporciona con fines exclusivamente educativos y para la formación de profesionales en ciberseguridad. La ejecución de estos ataques en redes sin autorización explícita es ilegal y penada por la ley. Practica siempre en tu propio laboratorio controlado.

---

## Un escáner en el Pentesting

Un escáner es solo una herramienta de diagnóstico, no un auditor.

### Fases del Escaneo
1.  **Descubrimiento:** El escáner mapea qué equipos están vivos.
2.  **Identificación de Puertos y Servicios:** Determina qué puertas están abiertas.
3.  **Detección de Versiones:** Pregunta al servicio "¿Qué software eres?" (ej: Apache 2.4.49).
4.  **Comparación con Base de Datos:** Compara esa versión contra su base de datos de CVEs (Common Vulnerabilities and Exposures).
5.  **Prueba de Concepto (Opcional):** Envía paquetes específicos para confirmar la existencia del fallo (sin explotarlo dañinamente).

---

## ⚙️ 2. Tipos de Escaneo: ¿Credenciales o no?

Esta es la decisión técnica más importante antes de iniciar un escáner y pese a nuestra recomendación, debe ser elegida por el cliente. Aunque nosotros podemos ayudar a tomar esta decisión basado en lo siguiente:

### A. Escaneo Sin Credenciales (Unauthenticated Scan)
El escáner simula ser un hacker que llega desde internet sin usuario ni contraseña. Es la visión del "atacante externo". 
* **Alcance:** Limitado a lo que se ve desde fuera (servicios de red, banners, cifrados SSL débiles). 
* **Uso:** Ideal para simular ataques Black Box y revisar el perímetro externo.

### B. Escaneo Con Credenciales (Authenticated Scan)
Le damos al escáner un usuario y contraseña (administrador o root) para que inicie sesión en la máquina objetivo.
* **Alcance:** Total. El escáner puede revisar el registro de Windows, paquetes de software instalados localmente (Adobe, Chrome), configuraciones de parches y hardening del sistema operativo.
* **Precisión:** Mucho mayor. Reduce drásticamente los falsos positivos porque ve la realidad interna del sistema.

---

## Nessus: El Estándar de la Industria

**Tenable Nessus** es una de las herramientas más utilizadas a nivel mundial. Para usarla profesionalmente, debemos entender cómo ajustar sus "Policies" (Políticas de Escaneo) para no tirar la red abajo.

### Conceptos de Configuración en Nessus (Settings)

#### 1. Discovery 
Define cómo encuentra Nessus los objetivos.
* **Host Discovery:** Si estás en una red interna y ya sabes qué IPs atacar, a veces conviene desactivar el "Ping" (ICMP), ya que los firewalls de Windows suelen bloquearlo y Nessus podría pensar erróneamente que el host está apagado.

#### 2. Port Scanning 
Define qué puertos revisará.
* **Rango de Puertos:** Por defecto, escanea los puertos "comunes" (aprox. 4,000). En una auditoría crítica, debes cambiar esto a "All ports" (1-65535), aunque tomará mucho más tiempo.
* **Técnica:** SYN Scan (medio abierto) es el estándar por ser rápido y sigiloso.

#### 3. Assessment 
Aquí definimos qué tan agresivo será el escáner.
* **Web Applications:** Nessus puede intentar escanear vulnerabilidades web (SQLi, XSS), pero **cuidado**. Los escáneres web son muy agresivos y pueden llenar bases de datos con basura o bloquear cuentas de usuario. Generalmente, se recomienda usar herramientas especializadas (como OWASP ZAP o Burp Suite) para la web y dejar a Nessus para la infraestructura.
* **Brute Force:** Nessus puede intentar adivinar contraseñas (Hydra style). En entornos de producción, esto suele desactivarse para no bloquear cuentas de Directorio Activo por intentos fallidos.

#### 4. Report 
Configura cómo se presenta la información.
* **Paranoia Level (Nivel de Paranoia):** Si lo subes, Nessus reportará vulnerabilidades incluso si no está 100% seguro. Útil para entornos de ultra-seguridad donde no quieres que se escape nada, pero prepárate para muchos falsos positivos.

---

## ⚠️ 4. Riesgos y Consideraciones Éticas

Ejecutar un escáner no es inocuo. Genera una cantidad masiva de tráfico y conexiones simultáneas.

* **Denegación de Servicio (DoS) Accidental:** Escanear sistemas antiguos (legacy), impresoras industriales o dispositivos médicos (IoT) puede hacer que se cuelguen o reinicien.
* **Horarios:** Siempre coordina los escaneos en ventanas de mantenimiento o de bajo tráfico.
* **Ancho de Banda:** Un escaneo agresivo puede saturar la red Wi-Fi o VPN de una oficina pequeña.
