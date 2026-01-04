# [Clase0007]: [ARP Spoofing y WiFi: El arte de la intercepción]

| Metadatos | Detalle |
| :--- | :--- |
| **Fecha** | 2024-01-03 |
| **Tiempo estimado** | 2 horas |
| **Tema** | Hacking Wireless: WPA2 Cracking & Decryption |
| **Resultado** | Credenciales obtenidas y tráfico desencriptado |

---

> **⚠️ Disclaimer Ético:** Esta información se proporciona con fines exclusivamente educativos. La intercepción de comunicaciones sin consentimiento es un delito federal en la mayoría de los países. Ejecuta estos pasos **únicamente** contra tu propio router y dispositivos.

---

## 1. Objetivo General
*Demostrar la inseguridad de redes WPA2 ante ataques de desautenticación, capturar el "Handshake" criptográfico, obtener la contraseña y utilizarla para desencriptar el tráfico de la red en tiempo real.*

### Objetivos Específicos
1.  Transformar la tarjeta de red a "Modo Monitor" para escuchar tráfico promiscuo.
2.  Utilizar **Airgeddon** para orquestar un ataque de desautenticación y capturar el 4-Way Handshake.
3.  Crackear el hash capturado mediante fuerza bruta basada en diccionario.
4.  Configurar **Wireshark** con la clave obtenida para traducir el tráfico cifrado a texto legible.

---

## 2. Requisitos y Entorno
* **OS:** Kali Linux.
* **Hardware:** Antena WiFi USB (Chipset Atheros AR9271 o Ralink RT3070 recomendados).
* **Target:** Router propio (SSID: Router propio) y un cliente conectado (móvil o laptop). Los dispositivos en la WLAN local, siempre que sean propios, funcionan.
* **Software:** Airgeddon, Aircrack-ng, Wireshark.
* **Wordlist:** `/usr/share/wordlists/rockyou.txt` (descomprimido).

---

## 3. Escenario
*Estás realizando una auditoría interna. Se ha detectado una red WiFi corporativa que utiliza WPA2-Personal. Debes demostrar que, si la contraseña es débil, un atacante externo puede capturar el tráfico confidencial (HTTP, DNS) simplemente estando en el estacionamiento, sin necesidad de conectarse físicamente a la red.*

---

## 4. Desarrollo Paso a Paso (Walkthrough)

### Fase 1: Preparación del Entorno (Airgeddon)
*Iniciamos la suite y preparamos la tarjeta de red.*

**Paso 1.1: Ejecución de la Herramienta**
Abrimos una terminal con permisos de root y lanzamos Airgeddon.

`
cd airgeddon/
sudo bash airgeddon.sh
`

**Paso 1.2: Selección de Interfaz**
Airgeddon realizará una comprobación de dependencias. Presiona **`[ENTER]`** hasta llegar al menú de selección de interfaz.
* El script listará tus interfaces de red. Identifica tu tarjeta WiFi USB (ej. `wlan0`).
* **Acción en Menú:**
    > Escribe el **número** correspondiente a tu interfaz (ej. `2`) y presiona **`[ENTER]`**.

**Paso 1.3: Activación de Modo Monitor**
Necesitamos cambiar la tarjeta de modo "Managed" a "Monitor".
* **Acción en Menú Principal:**
    > Selecciona la opción **`2`** (Put interface in monitor mode) y presiona **`[ENTER]`**.
* *Resultado:* El script recargará y tu interfaz cambiará de nombre (ej. a `wlan0mon`). Presiona **`[ENTER]`** para volver al menú.

**Evidencia 1.3:**
`[Captura de pantalla mostrando el asterisco (*) indicando modo monitor activo]`

---

### Fase 2: Reconocimiento y Selección de Objetivo
*Identificamos la red víctima.*

**Paso 2.1: Menú de Herramientas Handshake**
* **Acción en Menú Principal:**
    > Selecciona la opción **`5`** (Handshake tools menu) y presiona **`[ENTER]`**.

**Paso 2.2: Exploración de Redes (Scanning)**
* **Acción en Menú Handshake:**
    > Selecciona la opción **`4`** (Explore for targets) y presiona **`[ENTER]`**.
* Se abrirá una ventana de `airodump-ng`. Espera unos 20-30 segundos hasta ver tu red objetivo (`Laboratorio_WiFi`) y que aparezcan clientes en la parte inferior (STATION).
* **Acción:** Presiona **`[CTRL+C]`** para detener el escaneo.

**Paso 2.3: Selección del Target**
Airgeddon mostrará una lista numerada de las redes detectadas.
* **Acción en Lista de Objetivos:**
    > Escribe el **número** de tu red (ej. `14`) y presiona **`[ENTER]`**.

---

### Fase 3: Ataque de Desautenticación (Captura del Handshake)
*Desconectamos al usuario legítimo para forzar una reconexión y capturar las credenciales cifradas.*

**Paso 3.1: Selección del Tipo de Ataque**
* **Acción en Menú de Captura:**
    > Selecciona la opción **`2`** (Deauth aireplay attack) y presiona **`[ENTER]`**.
* Pregunta: "¿Deseas buscar un handshake pasivamente?" -> Responde **`n`** (No).
* Pregunta: "Timeout" (Tiempo de ataque) -> Escribe **`20`** (segundos) y presiona **`[ENTER]`**.

**Paso 3.2: Ejecución y Verificación**
Se abrirán varias ventanas. Una enviará paquetes de "Deauth" y otra escuchará el tráfico.
* Observa la ventana superior derecha. Espera hasta que aparezca el texto: **`[ WPA HANDSHAKE CAPTURED ]`**.
* Si aparece, presiona **`[ENTER]`** en la ventana principal para guardar el archivo `.cap`.
* Pregunta: "¿Dónde guardar el archivo?" -> Presiona **`[ENTER]`** (Ruta por defecto).

**Evidencia 3.2:**
`[Captura de pantalla donde se confirma la captura del Handshake WPA]`

---

### Fase 4: Obtención de la Contraseña (Cracking)
*Rompemos el cifrado del handshake capturado.*

**Paso 4.1: Menú de Ataque Offline**
Airgeddon preguntará automáticamente si quieres intentar crackear el archivo capturado.
* **Acción:** Responde **`y`** (Yes).

**Paso 4.2: Ataque de Diccionario**
* **Acción en Menú de Cracking:**
    > Selecciona la opción **`1`** (Dictionary attack) y presiona **`[ENTER]`**.

**Paso 4.3: Configuración del Diccionario**
Airgeddon pedirá la ruta del diccionario.
* **Acción:** Escribe `/usr/share/wordlists/rockyou.txt` y presiona **`[ENTER]`**.
* Presiona **`[ENTER]`** dos veces más para confirmar los BSSID.

**Resultado:**
Si la contraseña es débil, aparecerá en verde:
> **KEY FOUND! [ 12345678 ]**

**Evidencia 4.3:**
`[Captura de pantalla mostrando la clave desencriptada]`

---

### Fase 5: Desencriptación de Tráfico (Wireshark)
*Usamos la contraseña obtenida para espiar la red.*

**Paso 5.1: Cargar la Captura**
No cierres Airgeddon, pero abre Wireshark en otra terminal.
* **Acción:**
    > Archivo > Abrir > Selecciona el archivo `.cap` que generó Airgeddon (usualmente en `/root/airgeddon/captured/`).

**Paso 5.2: Configuración de Claves WPA**
El tráfico se verá azul oscuro (Protocolo 802.11) e ilegible.
* **Acción en Wireshark:**
    1.  Menú **Edit** -> **Preferences**.
    2.  En el panel izquierdo, despliega **Protocols** -> busca **IEEE 802.11**.
    3.  Marca la casilla: `[x] Enable decryption`.
    4.  Haz clic en el botón **Edit...** (junto a Decryption Keys).
    5.  Haz clic en el botón **(+)** para agregar una nueva regla.
    6.  Selecciona **Key Type:** `wpa-pwd`.
    7.  En **Key**, escribe la contraseña y el SSID en este formato exacto: `contraseña:SSID` (Ej: `12345678:Laboratorio_WiFi`).
    8.  Clic **OK** -> **OK**.

**Paso 5.3: Visualización de Datos**
Wireshark recargará los paquetes.
* **Acción:**
    > En la barra de filtro escribe: `http || dns`
* Verás que las líneas azules se transforman en verdes (HTTP) o azules claras (DNS), mostrando las páginas visitadas por la víctima.

**Evidencia 5.3:**
`[Captura de pantalla de Wireshark mostrando una petición GET HTTP legible]`

---

## 5. Resolución de Problemas (Troubleshooting)

* **Error:** Airgeddon no detecta mi tarjeta.
    * **Solución:** Ejecuta `iwconfig` para verificar que el sistema la reconoce. Si estás en VM, verifica el USB Passthrough.
* **Error:** No capturo el Handshake tras varios intentos.
    * **Solución:** El cliente debe estar transfiriendo datos activamente. Abre un video de YouTube en el dispositivo víctima mientras lanzas el ataque de desautenticación.
* **Error:** Wireshark no desencripta aunque puse la clave.
    * **Solución:** Wireshark necesita los 4 paquetes del Handshake (EAPOL) al inicio de la captura para derivar las claves de sesión. Asegúrate de estar abriendo el archivo `.cap` que contiene el handshake, no una captura nueva vacía.

---

## 6. Conclusiones
Has demostrado con éxito que una red WPA2 puede ser comprometida si la contraseña es débil. Al obtener la clave Pre-Shared Key (PSK), es posible descifrar todo el tráfico aéreo pasado y futuro, comprometiendo la confidencialidad de la organización.
