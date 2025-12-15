# Ataques DoS y DDoS: Tipos, Defensas y Pruebas de Seguridad

La Disponibilidad es uno de los tres pilares fundamentales de la seguridad de la información (Tríada CIA). Los ataques de Denegación de Servicio (DoS) y Denegación de Servicio Distribuida (DDoS) atentan directamente contra este pilar, buscando inutilizar un sistema, servidor o red para que sus usuarios legítimos no puedan acceder a él.

En esta sesión, analizaremos la anatomía de estos ataques, cómo se clasifican según la capa del modelo OSI afectada y, lo más importante, las estrategias arquitectónicas de defensa necesarias para mitigarlos.


---

> **⚠️ Disclaimer Ético:** Esta información se proporciona con fines exclusivamente educativos y para la formación de profesionales en ciberseguridad. La ejecución de estos ataques en redes sin autorización explícita es ilegal y penada por la ley. Practica siempre en tu propio laboratorio controlado.

---


## Definiciones Fundamentales

### DoS (Denial of Service)
Ocurre cuando un único atacante (una sola conexión de red o IP) inunda un objetivo con tráfico malicioso o explota una vulnerabilidad para agotar sus recursos (CPU, RAM, ancho de banda), provocando una interrupción del servicio.

### DDoS (Distributed Denial of Service)
Es una evolución del anterior donde el ataque proviene de múltiples fuentes simultáneamente. Generalmente, se orquesta mediante una **Botnet** (red de dispositivos infectados o "zombies") controlada remotamente por un servidor de Comando y Control (C2). Al distribuir el origen, el bloqueo por IP se vuelve ineficaz, ya que el tráfico malicioso se mezcla con el tráfico legítimo desde miles de ubicaciones globales.

## Taxonomía de los Ataques

Los ataques se clasifican generalmente en tres categorías según su comportamiento y objetivo técnico.

### A. Ataques Volumétricos (Saturación de Ancho de Banda)
El objetivo es consumir todo el ancho de banda disponible entre el objetivo e internet. Se mide en **Bits por segundo (Bps)**.

* **Inundación UDP (UDP Flood):** Envío masivo de paquetes UDP a puertos aleatorios. El host debe verificar qué aplicación escucha en ese puerto y responder con un "ICMP Destination Unreachable", saturando su capacidad de procesamiento.
* **Amplificación (DNS/NTP Amplification):** El atacante falsifica la IP de la víctima (IP Spoofing) y envía una petición pequeña a un servidor público (como un DNS abierto). El servidor responde con un paquete mucho más grande dirigido a la víctima.
    * *Concepto Clave:* El factor de amplificación. Una petición de 60 bytes puede generar una respuesta de 3000 bytes, multiplicando el poder de fuego del atacante.



### B. Ataques de Protocolo (Agotamiento de Estado)
El objetivo es consumir los recursos de los dispositivos de infraestructura (Firewalls, Balanceadores de Carga) o la pila TCP/IP del servidor. Se mide en **Paquetes por segundo (Pps)**.

* **SYN Flood:** El atacante envía miles de peticiones de inicio de conexión TCP (SYN) pero nunca completa el saludo de tres vías (nunca envía el ACK final). El servidor reserva memoria para estas "conexiones a medias" hasta que su tabla de conexiones se llena y deja de aceptar usuarios legítimos.
* **Ping of Death / Fragmentación:** Envío de paquetes malformados o fragmentados de manera que el sistema operativo destino colapse al intentar reensamblarlos.

### C. Ataques de Capa de Aplicación (Capa 7)
Son ataques sigilosos y de bajo volumen de tráfico. Buscan agotar los recursos del servidor web (CPU/Memoria) simulando comportamiento humano. Se mide en **Solicitudes por segundo (Rps)**.

* **HTTP Flood:** Peticiones GET o POST legítimas pero masivas hacia recursos pesados (ej. un buscador o un generador de reportes PDF).
* **Slowloris:** El atacante abre muchas conexiones al servidor web y las mantiene abiertas el mayor tiempo posible enviando cabeceras HTTP parciales muy lentamente. El servidor espera a que termine la cabecera, ocupando todos sus hilos de ejecución disponibles.

## Estrategias de Ciberseguridad y Defensa

La defensa contra DDoS no es una solución única, sino una arquitectura de defensa en profundidad.

### Fase 1: Arquitectura Preventiva
Diseñar la red para absorber el impacto antes de que ocurra.
* **Reducción de la Superficie de Ataque:** No exponer servidores directamente a internet. Utilizar zonas desmilitarizadas (DMZ) y restringir el tráfico solo a los puertos necesarios.
* **Balanceo de Carga (Load Balancing):** Distribuir el tráfico entre múltiples servidores evita que un solo nodo se convierta en un punto único de fallo.
* **CDN (Content Delivery Network):** Almacenar contenido estático en una red global. Si atacan el sitio, la CDN absorbe el golpe volumétrico, protegiendo al servidor de origen (Origin Server).

### Detección y Mitigación Activa
Herramientas que reaccionan durante el ataque.

* **Rate Limiting (Limitación de Tasa):** Configurar reglas en el WAF (Web Application Firewall) para limitar cuántas peticiones puede hacer una IP en un minuto.
* **SYN Cookies:** Técnica de defensa en el servidor. Cuando llega una petición SYN, el servidor no reserva memoria inmediatamente; en su lugar, envía una respuesta cifrada (cookie) al cliente. Solo si el cliente responde correctamente se asigna memoria, neutralizando el SYN Flood.
* **Blackholing / Null Routing:** En ataques masivos donde la infraestructura está por colapsar, el ISP puede "tirar a la basura" todo el tráfico dirigido a la IP atacada. Esto salva la red del proveedor, pero desconecta efectivamente al objetivo (el ataque tiene éxito, pero no afecta a otros clientes).
* **Scrubbing Centers (Centros de Limpieza):** La solución empresarial. El tráfico se redirige vía DNS o BGP a un centro de datos especializado en mitigación. Allí, se filtra el tráfico malicioso y solo se reenvía el tráfico limpio a la red de la empresa mediante un túnel GRE.


## Pruebas de Estrés y Simulación

Para validar las defensas, las organizaciones realizan pruebas controladas. Es vital distinguir entre una prueba de estrés y un ataque.

* **Stress Testing:** Uso de herramientas (como JMeter o Apache Benchmark) para determinar el punto de ruptura de una aplicación bajo carga pesada pero predecible.
* **DDoS Simulation (Red Teaming):** Contratación de servicios autorizados para lanzar ataques simulados (volumétricos o de capa 7) contra la infraestructura durante ventanas de mantenimiento, con el fin de verificar si los firewalls, el WAF y el equipo de respuesta a incidentes reaccionan adecuadamente.

Para el alcance del curso de hacking ético y para esta clase, se considera que este tipo de simulación, por necesitar alto rendimiento de los dispositivos, se tengan en cuenta, sin embargo, no sertán simuladas.
