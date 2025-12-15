# ARP Spoofing y WiFi: El arte de la intercepción

En esta clase, nos alejamos del cable de red para entrar en el dominio de las radiofrecuencias. Analizaremos la anatomía de un ataque WiFi completo: desde la escucha silenciosa en el aire hasta la desencriptación total del tráfico de una red protegida.

---

> **⚠️ Disclaimer Ético:** Esta información se proporciona con fines exclusivamente educativos y para la formación de profesionales en ciberseguridad. La ejecución de estos ataques en redes sin autorización explícita es ilegal y penada por la ley. Practica siempre en tu propio laboratorio controlado.

---

## Modo Monitor vs. Modo Gestionado

Para interceptar WiFi, primero debemos "ver" lo invisible. Las tarjetas de red convencionales operan en **Modo Gestionado (Managed Mode)**: solo escuchan los paquetes dirigidos específicamente a su dirección MAC. Ignoran el resto para no procesar "ruido".

Para un atacante o auditor, esto no sirve. Necesitamos el **Modo Monitor**.

### ¿Qué hace el modo Monitor?
Permite a la tarjeta de red capturar **todas** las tramas inalámbricas que viajan por el aire en una frecuencia específica, sin necesidad de estar asociada (conectada) a ningún Punto de Acceso (AP). Es el equivalente a sintonizar una radio y escuchar todo lo que se transmite, sea para ti o no.

---

## Desautenticación (Deauth Attack)

Para capturar la contraseña, necesitamos que un usuario legítimo se conecte. ¿Cómo forzamos esto? Expulsándolo.

### Tramas de Gestión 802.11
El protocolo WiFi utiliza tramas especiales para gestionar conexiones. Una de ellas es la **Trama de Desautenticación**.
* **La Vulnerabilidad:** En redes WPA2 estándar, estas tramas de gestión *no están cifradas*.
* **El Ataque:** El atacante, usando su tarjeta en modo monitor, falsifica (Spoofing) la dirección MAC del Router y envía un paquete de "Desconexión" a la víctima.
* **Resultado:** La víctima pierde la conexión momentáneamente. Su dispositivo, programado para tener internet, intentará reconectarse automáticamente de inmediato.

---

## El 4-Way Handshake

Cuando el dispositivo de la víctima se reconecta automáticamente tras el ataque de Deauth, debe demostrarle al Router que conoce la contraseña, pero **nunca envía la contraseña en texto plano**.

En su lugar, realizan un proceso matemático llamado **4-Way Handshake** (Saludo de 4 vías).

### ¿Qué contiene el Handshake?
El atacante captura este intercambio. No contiene la contraseña, pero contiene los ingredientes para calcularla:
1.  **ANonce:** Un número aleatorio generado por el Access Point (Router).
2.  **SNonce:** Un número aleatorio generado por la Station (Cliente/Víctima).
3.  **Direcciones MAC:** De ambos dispositivos.
4.  **MIC (Message Integrity Code):** Una "firma" digital. Es una prueba matemática de que el cliente conoce la contraseña.

---

## Cracking: pasar de la Captura a la Clave (WPA-PSK)

Una vez que tenemos el handshake capturado en un archivo (usualmente `.cap` o `.pcap`), nos llevamos el archivo a casa ("Offline Cracking").

### El Proceso de Fuerza Bruta / Diccionario
Dado que no podemos revertir el hash MIC para ver la contraseña, tenemos que adivinarla:
1.  El atacante toma una lista de millones de palabras (Diccionario).
2.  Toma la primera palabra + el nombre de la red (SSID) + los Nonces capturados.
3.  Aplica el algoritmo **PBKDF2** (la fórmula matemática de WPA2).
4.  Si el resultado coincide con el **MIC** que capturamos en el handshake... ¡Bingo! Esa palabra es la contraseña.

---

## Descifrado de Tráfico (Wireshark)

Este es el paso final y el más crítico de este escenario. Muchos creen que tener la contraseña del WiFi solo sirve para tener internet gratis, pero en auditoría significa **visibilidad total**.

Si tienes la contraseña de la red y estás escuchando en Modo Monitor:

1.  **Cifrado WPA2 (AES-CCMP):** El tráfico WiFi viaja cifrado por el aire. Aunque lo captures con Wireshark, verás datos ilegibles (Data Layers cifradas).
2.  **La Llave Maestra (PMK):** Al introducir la contraseña del WiFi en Wireshark, el software genera la *Pairwise Master Key*.
3.  **La Llave Temporal (PTK):** Para desencriptar una sesión específica, Wireshark necesita observar el **Handshake** de esa sesión en vivo (o tenerlo en la captura). Usando la PMK + los Nonces del handshake, deriva la **PTK** (Pairwise Transient Key).

---

    **Resultado:** Wireshark usa la PTK para desencriptar cada paquete de datos en tiempo real. Ahora puedes ver las peticiones HTTP, DNS y todo el tráfico no cifrado por SSL/TLS de la víctima, tal como si estuvieras conectado por cable a su dispositivo.
