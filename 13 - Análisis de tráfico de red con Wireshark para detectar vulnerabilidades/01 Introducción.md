# Análisis de tráfico de red con Wireshark para detectar vulnerabilidades

El análisis de tráfico de red es una habilidad fundamental tanto para auditores de seguridad (Red Team) como para analistas de defensa (Blue Team). Mientras que los escáneres de vulnerabilidades identifican fallos en el software estático, el análisis de tráfico permite detectar explotaciones en tiempo real, configuraciones inseguras y comportamientos anómalos que indican un compromiso activo.

En esta sesión, profundizaremos en cómo utilizar Wireshark no solo para capturar paquetes, sino para interpretar los patrones de comunicación y distinguir entre tráfico legítimo y actividad maliciosa.

---

> **⚠️ Disclaimer Ético:** Esta información se proporciona con fines exclusivamente educativos y para la formación de profesionales en ciberseguridad. La ejecución de estos ataques en redes sin autorización explícita es ilegal y penada por la ley. Practica siempre en tu propio laboratorio controlado.

---

## Análisis de tráfico

Antes de poder identificar lo "malicioso", es indispensable entender lo "normal". La detección de anomalías se basa en la desviación de un patrón estándar.

* **Protocolos Esperados:** En una red corporativa de usuarios, se espera ver HTTP/HTTPS, DNS, SMB y tráfico de correo. Ver tráfico IRC o BitTorrent es una anomalía inmediata.
* **Volumen de Tráfico:** Un servidor de base de datos suele recibir consultas pequeñas y devolver respuestas medianas. Si el servidor comienza a enviar gigabytes de datos hacia una IP externa desconocida, es un indicador de exfiltración.
* **Jerarquía de Protocolos:** La herramienta "Protocol Hierarchy" en Wireshark permite visualizar rápidamente la distribución del tráfico. Si el 90% del tráfico es ICMP, es probable que estemos ante un ataque de inundación o un problema de red, no un uso normal.
---
## Detección de Reconocimiento y Escaneo

La primera fase de un ataque es el escaneo. Los atacantes intentan descubrir hosts activos y puertos abiertos. Este comportamiento es ruidoso y deja firmas claras en el tráfico.

### Escaneos TCP (TCP Connect y SYN Scans)
Un escaneo de puertos genera una cantidad masiva de intentos de conexión en un periodo corto.

* **Patrón Normal:** Un cliente envía `SYN`, el servidor responde `SYN/ACK`, el cliente envía `ACK` y luego fluyen datos (HTTP, SSH, etc.).
* **Patrón de Escaneo SYN (Stealth):** El atacante envía `SYN`. Si recibe `SYN/ACK` (puerto abierto), el atacante responde inmediatamente con `RST` (Reset) en lugar de completar la conexión. Esto evita que la conexión quede registrada en los logs de la aplicación, pero es visible en la red.
* **Filtro Wireshark:**
    ```
    tcp.flags.syn == 1 and tcp.flags.ack == 0
    ```
    Si observas miles de paquetes con este filtro dirigidos a diferentes puertos de una misma IP, es un escaneo vertical.

### Escaneos UDP
UDP es un protocolo sin conexión, lo que hace su análisis más sutil.
* **Patrón Malicioso:** El atacante envía paquetes UDP vacíos a puertos aleatorios. Si el puerto está cerrado, el sistema operativo objetivo responde con un mensaje ICMP "Destination Unreachable (Port Unreachable)".
* **Indicador:** Una avalancha de mensajes ICMP Tipo 3, Código 3 saliendo del servidor objetivo hacia una única IP externa indica que alguien está escaneando sus puertos UDP.

--- 

## Detección de Ataques de Fuerza Bruta

Los ataques de fuerza bruta contra servicios como SSH, FTP, RDP o paneles Web generan un patrón de tráfico repetitivo y predecible.

* **Análisis de Flujo (Stream):** Cada intento de login fallido genera una nueva sesión o transacción.
* **Tamaño del Paquete:** En protocolos no cifrados o parcialmente cifrados, las respuestas de "Login Failed" tienen un tamaño casi idéntico.
* **Indicador Visual:** En Wireshark, verás una secuencia rápida de:
    1.  `SYN` -> `SYN/ACK` -> `ACK`
    2.  Intercambio de datos pequeño (Intento de credencial).
    3.  `FIN` o `RST` (Cierre de conexión).
    4.  Repetición inmediata del ciclo cientos de veces por minuto.

* **Filtro para RDP/SSH:**
    Analizar estadísticas de conversaciones TCP (`Statistics -> Conversations -> TCP`) ayuda a identificar una IP origen que abre miles de conexiones contra el puerto 22 o 3389 pero transfiere muy pocos bytes (indicando que nunca logra entrar).

## Detección de Envenenamiento y Spoofing

Los ataques de "Man-in-the-Middle" (MitM) a nivel de red local a menudo se basan en la manipulación del protocolo ARP.

### ARP Spoofing (ARP Cache Poisoning)
El atacante inunda la red con mensajes ARP falsos diciendo "Yo soy la IP del Router".

* **Patrón de Tráfico:** Aparición de mensajes "Gratuitous ARP" o respuestas ARP no solicitadas (is-at) con una frecuencia anormalmente alta.
* **Detección en Wireshark:** Wireshark tiene una heurística interna para esto. Busca paquetes marcados con la etiqueta amarilla/negra: **"Duplicate IP address detected"**. Esto significa que dos direcciones MAC diferentes están reclamando tener la misma dirección IP en un corto periodo de tiempo.
* **Filtro:** `arp.duplicate-address-detected`

---

## Detección de Malware y Canales de Comando y Control (C2)

Una vez que un sistema está comprometido, el malware debe comunicarse con su operador. Este tráfico (Beaconing) es clave para la detección.

### Beaconing (Faros)
El malware suele comunicarse con el servidor C2 a intervalos regulares para pedir instrucciones.
* **Patrón:** Conexiones HTTP/HTTPS o DNS breves y rítmicas. Por ejemplo, una petición a `update.info-bad.com` exactamente cada 60 segundos con un tamaño de payload (carga útil) muy similar.
* **Análisis de Tiempo (Delta Time):** Configurar Wireshark para mostrar el tiempo transcurrido desde el paquete anterior de la misma conversación ayuda a identificar esta regularidad mecánica.

### DNS Tunneling y DGA
Algunos malwares usan DNS para exfiltrar datos o recibir comandos, evadiendo firewalls que permiten tráfico UDP 53.
* **Patrón DGA (Domain Generation Algorithm):** Consultas a dominios sin sentido semántico, generados aleatoriamente (por ejemplo. `xya-123-bbq-99.com`, `asdfkjhasdkjf.net`).
* **Tunneling:** Registros TXT o consultas A inusualmente largas. Un nombre de dominio que contiene una cadena codificada en Base64 muy larga (ej. `d8sa9d8sa9d8s.a8d9s8d9s.evil.com`) sugiere que se están enviando datos dentro de la petición DNS.

## Exfiltración de Datos

El objetivo final de muchos ataques es el robo de información.
* **Indicador de Volumen:** Una conexión TCP persistente que envía una gran cantidad de datos hacia afuera (Outbound) en comparación con lo que recibe.
* **Protocolos Inusuales:** Transferencia de archivos grandes sobre puertos no estándar (por ejemplo. tráfico cifrado saliendo por el puerto 8080 o 443 hacia una IP sin reputación).

---

## Metodología de Análisis Forense de Red

Para aplicar estos conceptos, podemos apoyarnos de la herramienta Wareshark para llevar a cabo la siguiente estructura de análisis:

1.  **Sanity Check:** Usar `Statistics -> Protocol Hierarchy` para ver si la distribución de protocolos tiene sentido.
2.  **Identificar Top Talkers:** Usar `Statistics -> Conversations` y ordenar por "Bytes" o "Packets" para ver quién está hablando más y con quién.
3.  **Filtrado de Ruido:** Eliminar tráfico de broadcast y multicast legítimo para limpiar la vista (`not arp and not ssdp`).
4.  **Análisis de Flujos:** Usar "Follow TCP/UDP Stream" para reconstruir la sesión y ver el contenido de los datos (si no están cifrados) o el comportamiento de la sesión.
