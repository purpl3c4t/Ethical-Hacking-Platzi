# Evasión de Firewalls y Sistemas de Detección

En esta clase, aprenderemos a identificar estas defensas y los tres pilares fundamentales para "volvernos invisibles" ante ellas.
Entre tu máquina y el objetivo, casi siempre habrá un **Firewall** (que bloquea tráfico) o un **IDS/IPS** (Sistema de Detección/Prevención de Intrusos que analiza el tráfico).

---

> **⚠️ Disclaimer Ético:** Esta información se proporciona con fines exclusivamente educativos y para la formación de profesionales en ciberseguridad. La ejecución de estos ataques en redes sin autorización explícita es ilegal y penada por la ley. Practica siempre en tu propio laboratorio controlado.

---

## Reconocimiento del Firewall

Antes de intentar saltar un muro, necesitas saber qué tan alto es y si tiene alambre de púas. La detección del dispositivo de seguridad es el primer paso crítico.

### 1. Huella Pasiva (Passive Footprinting)
Aquí analizamos las respuestas del servidor sin "tocar" el firewall directamente de forma agresiva.
* **Análisis de TTL (Time To Live):** Cada sistema operativo tiene un TTL por defecto (ej: Linux 64, Windows 128). Si al hacer ping recibes un TTL extraño o que disminuye abruptamente, es probable que un dispositivo intermedio (Firewall/Proxy) esté procesando el paquete antes de que llegue al destino.
* **Cabeceras HTTP y Errores:** Los Firewalls de Aplicación Web (WAF) a menudo dejan su firma en las cabeceras de respuesta o en las páginas de error personalizadas (ej: *"Blocked by Cloudflare"* o códigos de error 403 específicos).

### 2. Huella Activa (Active Footprinting)
Aquí interactuamos con la red para provocar una reacción del firewall.
* **Estados de Puerto (Nmap):** La clave está en la diferencia entre `CLOSED` y `FILTERED`.
    * **Closed:** El paquete llegó al servidor, no había nada escuchando y el servidor respondió "RST" (Reset). *Conclusión: No hay firewall bloqueando, el camino está libre.*
    * **Filtered:** El paquete fue enviado pero nunca volvió respuesta. Fue "comido" por un agujero negro. *Conclusión: Un firewall está bloqueando el tráfico.*
* **Firewalking:** Es una técnica similar a *traceroute*. Analiza qué pasa con los paquetes en el salto anterior al destino para determinar qué reglas (ACLs) tiene el firewall de borde.

---

## Los Tres Pilares de la Evasión

Una vez identificado el obstáculo, existen tres enfoques conceptuales principales para evadir la detección y lograr que nuestro tráfico llegue al servicio real.

### 1. Fragmentación y Desincronización
Los IDS/IPS intentan leer el tráfico buscando firmas de ataques (ej: la palabra "SELECT * FROM"). Para leerlo, deben capturar el paquete completo.

* **Concepto:** Si dividimos el paquete malicioso en fragmentos diminutos (IP Fragmentation), el firewall verá piezas individuales que parecen inofensivas.
    * *Fragmento 1:* "SEL"
    * *Fragmento 2:* "ECT *"
    * *Fragmento 3:* " FROM..."
* **La Evasión:** El firewall, para no ralentizar la red, a veces deja pasar los fragmentos pequeños sin reensamblarlos. Sin embargo, el servidor destino (la víctima) sí tiene la obligación de rearmarlos. El ataque se reconstruye *detrás* del muro.
* **MTU (Maximum Transmission Unit):** Forzamos el envío de datos más pequeños que el tamaño estándar para romper el análisis de firmas.

### Enmascaramiento y Señuelos 
Si no puedes volverte invisible, escóndete en la multitud. Este método busca confundir al sistema de detección sobre **quién** está atacando o **desde dónde**.

* **Decoys (Señuelos):** Al escanear, enviamos paquetes desde nuestra IP real, pero simultáneamente enviamos miles de paquetes idénticos falsificando las IPs de otros (IP Spoofing). El IDS verá un ataque proveniente de 50 direcciones diferentes a la vez. No sabrá a cuál bloquear, y si bloquea todas, causará una Denegación de Servicio a usuarios legítimos.
* **Manipulación del Puerto de Origen:** Muchos firewalls mal configurados confían ciegamente en tráfico que parece venir de servicios "sagrados".
    * *Regla común:* "Permitir todo lo que venga del puerto 53 (DNS) o puerto 20 (FTP Data)".
    * *Técnica:* Configuramos nuestro escáner para que nuestros paquetes salgan desde el puerto 53. El firewall cree que es una respuesta DNS y la deja pasar.

### Túneles y Ofuscación 
Si el firewall revisa el contenido de tus paquetes (Deep Packet Inspection), la solución es hacer que el contenido sea ilegible o parezca otra cosa.

* **Tunneling (Túneles):** Encapsular un protocolo prohibido dentro de uno permitido.
    * *Ejemplo:* **ICMP Tunneling**. Si el firewall permite el `ping` (ICMP) pero bloquea la conexión de datos, podemos inyectar datos reales dentro de la carga útil de los paquetes de ping. El firewall ve "pings", pero nosotros estamos transfiriendo archivos o comandos.
* **Encriptación (SSL/TLS):** Si atacamos un servicio HTTPS, el IDS no puede leer el ataque porque va cifrado. A menos que el IDS haga una intercepción SSL (Man-in-the-Middle), el ataque pasará invisible.
* **Codificación (Encoding):** En ataques web, transformar los caracteres maliciosos. El firewall busca `<script>`, pero nosotros enviamos `%3Cscript%3E` (URL Encoded). Si el firewall no decodifica antes de analizar, no verá la firma.

---

### ¿Qué técnica es mejor?

Depende del tipo de dispositivo, configuración e incluso, de la marca del fabricante. Generalmente, durante un ejercicio de pentest, es válido realizar todas las técnicas sin importar las detecciones que se causen.

| Método | Objetivo | Concepto Clave | Herramientas Típicas |
| :--- | :--- | :--- | :--- |
| **Fragmentación** | Romper la firma del ataque | Dividir paquetes (MTU) | `nmap -f`, Fragroute |
| **Enmascaramiento** | Ocultar la identidad | Decoys, Source Port | `nmap -D`, `nmap --source-port` |
| **Ofuscación** | Ocultar el Payload | Tunneling, Encoding | SSH Tunnels, HTTP Encoding |
