# Escalada de privilegios en sistemas Linux: de usuario básico a root

La escalada de privilegios es el proceso mediante el cual un atacante, que ya tiene acceso a un sistema con una cuenta de bajos privilegios, intenta obtener permisos superiores, generalmente los de administrador o "root".

En esta lección, analizaremos los fundamentos teóricos detrás de la explotación de configuraciones permisivas de `sudo` y cómo transformar un acceso básico en una sesión de control total utilizando herramientas como Metasploit.


---

> **⚠️ Disclaimer Ético:** Esta información se proporciona con fines exclusivamente educativos y para la formación de profesionales en ciberseguridad. La ejecución de estos ataques en redes sin autorización explícita es ilegal y penada por la ley. Practica siempre en tu propio laboratorio controlado.

---


## 1. El Concepto de Root (UID 0)

En sistemas basados en Linux y Unix, el modelo de seguridad se basa en identificadores de usuario (UID).

* **Usuario Estándar:** Tiene un UID superior a 0 (generalmente 1000 en adelante) y su acceso está restringido a su directorio personal (`/home/user`) y a archivos temporales.
* **Root (Superusuario):** Se define por tener el **UID 0**. Este usuario tiene control absoluto sobre el sistema operativo: puede leer cualquier archivo, modificar configuraciones del sistema, detener servicios y cargar módulos del kernel.

El objetivo final de la escalada de privilegios analizada aquí es cambiar el contexto de seguridad actual para que el comando `id` devuelva `uid=0(root)`.

## 2. El Mecanismo SUDO y Configuraciones NOPASSWD

La vulnerabilidad explorada en el caso de estudio no es un error de software (bug) ni un fallo en el código del kernel, sino una **debilidad de configuración**.

### Sudoers
El comando `sudo` (SuperUser DO) permite a un usuario permitido ejecutar un comando como superusuario u otro usuario. Estas reglas se definen en el archivo `/etc/sudoers`.



### La Directiva NOPASSWD
Por defecto, `sudo` solicita la contraseña del usuario actual para verificar su identidad antes de elevar privilegios. Sin embargo, los administradores pueden configurar la directiva `NOPASSWD` para ciertos usuarios o comandos por conveniencia o automatización.

* **Implicación de seguridad:** Si un atacante compromete una cuenta que tiene permisos `sudo` con `NOPASSWD`, la barrera de seguridad desaparece. El atacante puede elevar privilegios instantáneamente sin necesidad de realizar fuerza bruta ni conocer credenciales.

## 3. Entornos de Shell: Interactivo vs. No Interactivo

Al escalar privilegios mediante la línea de comandos (CLI), es importante distinguir cómo se invoca la nueva sesión de root.

### `sudo /bin/bash`
Ejecuta el binario de la shell (`bash`) con privilegios de root.
* **Resultado:** Se obtiene una terminal con privilegios de root (`#`), pero se conservan muchas variables de entorno del usuario original.

### `sudo -i`
Simula un inicio de sesión completo (login shell).
* **Resultado:** Además de otorgar privilegios de root, carga las variables de entorno de root (definidas en `/root/.profile` o `/root/.bashrc`) y cambia el directorio de trabajo al directorio home de root. Esto es preferible para asegurar que todas las herramientas administrativas estén en el `PATH`.

## 4. Weaponización con Payloads Binarios (ELF)

Cuando el acceso CLI simple no es suficiente y se requiere una sesión avanzada (como Meterpreter) con privilegios de root, es necesario "armar" (weaponize) la vulnerabilidad. Esto implica crear y ejecutar software malicioso personalizado.

### Formato ELF (Executable and Linkable Format)
Así como Windows utiliza `.exe`, Linux utiliza el formato ELF para sus binarios ejecutables. Para elevar una sesión en Linux, generamos un binario ELF malicioso.

### Msfvenom y Payloads
La herramienta `msfvenom` permite compilar este binario con una carga útil específica.

* **Payload (linux/x64/meterpreter/reverse_tcp):**
    * **Linux/x64:** Define la arquitectura del sistema objetivo (64 bits).
    * **Meterpreter:** Es un agente avanzado que corre en memoria, permitiendo cargar scripts, migrar procesos y extraer hashes.
    * **Reverse TCP:** Establece la conexión desde la víctima hacia el atacante (conexión inversa). Esto es crucial para evadir firewalls que suelen bloquear el tráfico entrante pero permiten el saliente.



## 5. El Ciclo de Vida de Ejecución en Linux

Para que el ataque de "actualización de sesión" (Upgrade) funcione, se deben respetar los principios de ejecución del sistema operativo Linux:

1.  **Transferencia (Upload):** El archivo ELF debe ser colocado en un directorio donde el usuario actual tenga permisos de escritura. El directorio `/tmp` es el estándar universal para esto, ya que suele tener permisos de escritura para todos los usuarios y permite ejecución.
2.  **Permisos de Ejecución (chmod):** En Linux, un archivo no es ejecutable simplemente por su extensión. El sistema de permisos de archivos requiere que el bit de ejecución (`+x`) esté activo. Sin ejecutar `chmod +x archivo.elf`, el sistema denegará el intento de ejecución, independientemente de quién lo intente.
3.  **Ejecución Privilegiada:** Finalmente, se utiliza el vector de ataque (`sudo`) para ejecutar el binario malicioso. Al hacerlo, el binario ELF se ejecuta en el contexto del usuario root, y la conexión inversa resultante (Reverse Shell) heredará esos privilegios máximos (UID 0).

## 6. Gestión de Sesiones (C2)

En un entorno de Comando y Control (C2) como Metasploit, es fundamental entender la diferencia entre las sesiones:



* **Handler (Multi/Handler):** Es el proceso en la máquina del atacante que queda a la espera ("escuchando") de la conexión entrante. Debe configurarse con los mismos parámetros (`LHOST`, `LPORT`) que el payload generado.
* **Backgrounding:** Proceso de enviar una sesión activa a segundo plano para liberar la consola y configurar nuevos módulos (como el listener) sin perder el acceso inicial.
