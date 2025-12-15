# Implicaciones Legales y Arquitectura de Laboratorio en AWS

Antes de tocar una sola línea de código o lanzar un escáner, debemos entender las reglas del juego. En esta sesión teórica, cubriremos dos pilares: el marco legal que protege tu carrera profesional y los conceptos de infraestructura en la nube necesarios para montar tu propio laboratorio de hacking en AWS.

---

> **⚠️ Disclaimer Ético:** Esta información se proporciona con fines exclusivamente educativos y para la formación de profesionales en ciberseguridad. La ejecución de estos ataques en redes sin autorización explícita es ilegal y penada por la ley. Practica siempre en tu propio laboratorio controlado.

---

## Implicaciones Legales del Hacking

El hacking ético no es un juego; es una profesión regulada por leyes estrictas. La línea entre un experto en seguridad y un ciberdelincuente suele definirse por un solo documento: **La Autorización**.

### 1. El Principio de Autorización
El acceso no autorizado a sistemas informáticos es un delito en casi todos los países del mundo (CFAA en EE.UU., Código Penal en España/Latam).
* **Regla de Oro:** Si no tienes un permiso escrito, firmado y vigente, no lo toques.
* **Propiedad vs. Uso:** Que tengas acceso a una red (como el Wi-Fi de una cafetería) no te da derecho a escanearla.

### 2. El Alcance (Scope)
Es el contrato que define los límites.
* **¿Qué está permitido?** (Ej: Atacar la IP X, probar SQL Injection en el formulario Y).
* **¿Qué está prohibido?** (Ej: Ingeniería social a empleados, ataques de Denegación de Servicio - DoS).
* *Salirse del alcance puede conllevar demandas civiles incluso si tenías buenas intenciones.*

### 3. Pentesting en la Nube (AWS)
La nube es "la computadora de otro". Cuando atacas tu laboratorio en AWS, estás generando tráfico en la red de Amazon.
* **Responsabilidad Compartida:** Eres responsable de la seguridad *dentro* de la nube (tus instancias, tus aplicaciones), pero no debes atacar la *infraestructura* de la nube (los servidores físicos de AWS, el plano de control).
* **Política de Uso Aceptable:** AWS permite realizar pruebas de penetración contra tus propios recursos (EC2, RDS) sin permiso previo en muchos casos, pero prohíbe estrictamente ataques que saturen sus redes (DDoS) o afecten a otros clientes.

---

## Arquitectura de Laboratorio en AWS

Para montar un laboratorio profesional, dejaremos de lado las máquinas virtuales locales (VirtualBox/VMware) y usaremos la nube. Para ello, debemos dominar el vocabulario de infraestructura de Amazon Web Services.

### 1. VPC (Virtual Private Cloud)
Imagina que la VPC es tu **casa privada** en la nube. Es una red virtual lógicamente aislada del resto del mundo y de otros clientes de AWS.
* Tú defines el rango de direcciones IP (ej: `10.0.0.0/16`).
* Todo lo que crees (servidores, bases de datos) vivirá dentro de esta "casa".

### 2. Subnet (Subred)
Son las **habitaciones** dentro de tu casa (VPC). Dividen tu red grande en segmentos más pequeños para organizar mejor los recursos.
* **Subnet Pública:** Tiene acceso directo a internet (aquí pondremos nuestra máquina atacante, Kali Linux).
* **Subnet Privada:** No tiene acceso directo desde internet (aquí pondremos las máquinas vulnerables para que nadie más pueda atacarlas).

### 3. Internet Gateway (IGW)
Es el **módem** o la puerta principal de la casa.
* Sin un IGW conectado a tu VPC, nadie puede entrar ni salir a internet. Es el componente que permite que tu laboratorio se comunique con el exterior para descargar herramientas o actualizaciones.

### 4. Routing Tables (Tablas de Enrutamiento)
Son los **letreros de señalización** dentro de la casa.
* Le dicen al tráfico de red hacia dónde ir.
* *Ejemplo:* "Si el tráfico quiere ir a Google (0.0.0.0/0), envíalo al Internet Gateway".
* Cada subred debe estar asociada a una tabla de enrutamiento para saber cómo navegar.

### 5. Security Groups (Grupos de Seguridad)
Es el **portero de discoteca** (Firewall Virtual).
* Actúa a nivel de instancia (servidor).
* Por defecto, **bloquea todo el tráfico de entrada** y permite todo el de salida.
* Debes abrir explícitamente los puertos necesarios (ej: Puerto 22 para SSH, Puerto 80 para Web).
* *Importante:* Son "Stateful" (con estado). Si permites que entre una petición, la respuesta se permite automáticamente.

---

## Dockerization (Docker)

En nuestro laboratorio, no siempre querremos instalar sistemas operativos completos para cada víctima. Aquí entra Docker.

### ¿Qué es Docker?
Es una tecnología de **contenedores**. A diferencia de una máquina virtual (que simula un hardware completo y carga un sistema operativo pesado), Docker empaqueta solo la aplicación y sus librerías necesarias.

### Conceptos Clave para el Laboratorio:
* **Imagen:** Es el "plano" o la receta (ej: una imagen de *DVWA* o *Juice Shop*). Es inmutable.
* **Contenedor:** Es la instancia viva de esa imagen. Puedes encender y destruir contenedores en segundos.
* **Port Mapping:** Como el contenedor vive aislado, debemos "conectar" un puerto de la máquina host (tu servidor AWS) con el puerto del contenedor. (Ej: `docker run -p 80:3000` redirige el puerto 80 del servidor al 3000 del contenedor).

### ¿Por qué Docker en Hacking?
1.  **Velocidad:** Despliegas entornos vulnerables complejos en segundos.
2.  **Aislamiento:** Si rompes el contenedor atacándolo, simplemente lo borras y creas uno nuevo (`docker rm` -> `docker run`).
3.  **Portabilidad:** Funciona igual en tu laptop que en AWS.

---

### Resumen de Arquitectura del Laboratorio

Para tu práctica, construiremos lo siguiente:

1.  Una **VPC** (Tu red aislada).
2.  Un **Internet Gateway** (Para tener conexión).
3.  Una **Subnet Pública** con su **Route Table** apuntando al Gateway.
4.  Una instancia **EC2** (Servidor) dentro de esa subnet.
5.  Un **Security Group** que permita solo tu IP (para administrar).
6.  Dentro de la EC2, usaremos **Docker** para levantar aplicaciones vulnerables.
