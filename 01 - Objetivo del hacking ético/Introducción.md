# 📡 ARP Spoofing y WiFi: El arte de la intercepción

Bienvenido a esta lección práctica sobre seguridad en redes locales. En este módulo, desmitificaremos cómo funcionan las comunicaciones en tu red local y entenderemos una de las técnicas de ataque (y auditoría) más antiguas y efectivas: el **ARP Spoofing**.

> **⚠️ Disclaimer Ético:** Esta información se proporciona con fines exclusivamente educativos y para la formación de profesionales en ciberseguridad. La ejecución de estos ataques en redes sin autorización explícita es ilegal y penada por la ley. Practica siempre en tu propio laboratorio controlado.

---

## 1. Fundamentos Teóricos: ¿Por qué confiamos en extraños?

Para entender el ataque, primero debemos entender cómo las máquinas se encuentran entre sí.

### ¿Qué es el protocolo ARP?
En una red local (LAN), las computadoras no se comunican usando direcciones IP (como `192.168.1.15`), sino usando **Direcciones MAC** (Direcciones físicas, como `AA:BB:CC:11:22:33`).

El protocolo **ARP (Address Resolution Protocol)** actúa como un traductor:
1. Tu PC quiere enviar datos al Router (`192.168.1.1`).
2. Tu PC grita a toda la red: *"¿Quién tiene la IP 192.168.1.1?"*.
3. El Router responde: *"Soy yo, y mi dirección física es `XX:XX:XX:XX:XX:XX`"*.
4. Tu PC guarda esta relación en su **Tabla ARP**.

### La Vulnerabilidad
El protocolo ARP nació en una época de confianza. **No existe validación.** Si yo le grito a tu PC: *"Hola, soy el Router"*, tu PC me creerá ciegamente y actualizará su tabla ARP, enviándome a mí todo tu tráfico de internet.

### El Ataque: Man-in-the-Middle (MitM)
Al falsificar estos mensajes (Spoofing), logramos ponernos en medio de la comunicación.
* **Víctima** ➡️ **Atacante** ➡️ **Router** ➡️ **Internet**

---
