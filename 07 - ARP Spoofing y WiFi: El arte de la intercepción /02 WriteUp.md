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
* **Target:** Router propio (SSID: `Laboratorio_WiFi`) y un cliente conectado (móvil o laptop).
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

```bash
cd airgeddon/
sudo bash airgeddon.sh
*Describe los pasos iniciales. Configuración de red, escaneo de puertos o despliegue de infraestructura.*

**Paso 1.1: [Nombre del Paso]**
Descripción técnica de la acción realizada.

```bash
# Inserte aquí el comando ejecutado
$ comando --parametro valor
