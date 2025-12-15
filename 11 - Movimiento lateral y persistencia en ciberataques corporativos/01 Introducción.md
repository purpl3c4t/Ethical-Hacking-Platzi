# Movimiento Lateral y Persistencia en Ciberataques Corporativos

En el ciclo de vida de un ciberataque avanzado, obtener acceso inicial es solo el comienzo. Para alcanzar los objetivos críticos (generalmente bases de datos o controladores de dominio), un atacante debe navegar a través de la infraestructura interna y asegurar su permanencia en el sistema.

Esta lección cubre los fundamentos teóricos del **Pivoting** (uso de sistemas comprometidos para enrutar tráfico) y la **Persistencia** (mantenimiento del acceso a largo plazo).


---

> **⚠️ Disclaimer Ético:** Esta información se proporciona con fines exclusivamente educativos y para la formación de profesionales en ciberseguridad. La ejecución de estos ataques en redes sin autorización explícita es ilegal y penada por la ley. Practica siempre en tu propio laboratorio controlado.

---

## 1. Segmentación de Red y Necesidad del Movimiento Lateral

Las redes corporativas rara vez son planas. Por diseño de seguridad, suelen estar segmentadas en zonas de confianza:

* **DMZ (Zona Desmilitarizada):** Contiene servicios expuestos a internet (Servidores Web, Proxies).
* **Red Interna:** Contiene activos de alto valor (Bases de datos, Servidores de archivos, Controladores de dominio) que no tienen acceso directo desde internet.

### Pivoting
El Pivoting es la técnica de utilizar una instancia comprometida (generalmente en la DMZ) como un enrutador o proxy para atacar otras redes que no son accesibles directamente.

El sistema comprometido actúa como un Jump Host. Desde la perspectiva de la red interna, el tráfico de ataque parece provenir del servidor de la DMZ, no de la máquina externa del atacante.


### Enrutamiento de Tráfico en Frameworks C2
Herramientas como Metasploit gestionan el pivoting mediante tablas de enrutamiento virtuales.
* **Identificación de Interfaces:** Mediante comandos como `ipconfig` o `ifconfig` en el host comprometido, se descubren interfaces de red adicionales conectadas a segmentos internos (ej. `172.18.0.x`).
* **Autorouting:** Se instruye al framework para que cualquier paquete destinado a la subred interna sea encapsulado y enviado a través de la sesión establecida (Meterpreter) en el host comprometido.

## Reconocimiento Interno y Tunneling

Una vez establecido el pivote, se debe explorar el nuevo entorno.

### Escaneo a través de Proxies
El escaneo de puertos en una red pivotada es más lento y complejo que un escaneo local. No se pueden utilizar técnicas de bajo nivel como "SYN Stealth Scan" fácilmente, ya que no se tiene control directo sobre la pila TCP/IP del host intermedio. 
Se suelen utilizar escaneos de conexión completa (TCP Connect Scan) gestionados por el framework de comando y control: C2.

### Port Forwarding (Reenvío de Puertos)
A menudo, las herramientas del framework C2 no son suficientes para explotar un servicio complejo (como una base de datos MySQL o un servidor RDP). Se requiere utilizar herramientas nativas del atacante (cliente mysql, rdesktop, navegador web).

El **Port Forwarding** resuelve esto creando un túnel directo:
1.  Se abre un puerto de escucha en la máquina local del atacante (ej. puerto 3306 local).
2.  Todo el tráfico enviado a ese puerto se encapsula y viaja por el túnel hasta el host comprometido.
3.  El host comprometido reenvía el tráfico al destino final (ej. base de datos interna 172.18.0.4).

Esto permite al atacante interactuar con servicios internos profundos como si estuvieran instalados en su propia máquina local.


## Persistencia: Asegurando el Acceso

La persistencia consiste en técnicas utilizadas para mantener el acceso a los sistemas a través de reinicios, cambios de credenciales y otras interrupciones. Sin persistencia, un atacante perdería todo el progreso si el administrador reinicia el servidor o cierra la vulnerabilidad inicial.

### Estrategias de Persistencia
Existen múltiples vectores, como la creación de servicios, tareas programadas (cron jobs) o modificación de registros. En entornos Linux y Unix, una de las técnicas más robustas y sigilosas es la manipulación de claves SSH.

### Inyección de Claves SSH
El protocolo SSH (Secure Shell) permite la autenticación mediante pares de claves criptográficas (pública/privada) como alternativa a las contraseñas.

* **Authorized Keys:** Los servidores SSH buscan en el archivo `~/.ssh/authorized_keys` del usuario las claves públicas permitidas para iniciar sesión.
* **El Ataque:** Si un atacante tiene permisos de escritura en este archivo, puede inyectar su propia clave pública.
* **Resultado:** El atacante puede iniciar sesión posteriormente usando su clave privada correspondiente.

Esta técnica es particularmente peligrosa porque:
1.  **Independencia de la Contraseña:** Incluso si el administrador cambia la contraseña del usuario, la clave SSH sigue siendo válida.
2.  **Legitimidad Aparente:** El tráfico SSH suele estar permitido en los firewalls y se mezcla con la actividad normal de administración.

### Fases del Flujo de Ataque

Hasta este momento, hemos revisado ya las fases para lograr la ejecución completa de un pentest. Vamos a recapitular hasta el momento los conceptos de las tres últimas:

1.  **Acceso Inicial:** Compromiso del borde de la red.
2.  **Enumeración de Red:** Detección de interfaces internas duales (`ipconfig`).
3.  **Pivoting:** Configuración de rutas para alcanzar la red interna (`route add`).
4.  **Descubrimiento:** Hallazgo de servicios ocultos (`portscan`).
5.  **Tunneling:** Acceso directo a servicios con herramientas locales (`portfwd`).
6.  **Persistencia:** Aseguramiento del acceso futuro (`sshkey_persistence`).
