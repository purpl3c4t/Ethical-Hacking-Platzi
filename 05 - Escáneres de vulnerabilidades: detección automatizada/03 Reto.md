# Challenge [0005]: [Escáneres de vulnerabilidades: detección automatizada]

| Información | Detalle |
| :--- | :--- |
| **Tema** | Escáneres de vulnerabilidades: detección automatizada de debilidades |
| **Dificultad** | Intermedia |
| **Tiempo Estimado** | 1.5 Horas |
| **Herramienta** | Tenable Nessus (Essentials o Pro) |

---

## 1. Contexto del Desafío

**Escenario:** Como analista de vulnerabilidades en una empresa de servicios financieros, tu equipo te ha entregado un reporte generado mediante un escaneo "Advanced Scan" (**ESCANEO AVANZADO_platzi.html**) de Nessus contra un servidor crítico. Este tipo de escaneo fue muy agresivo, tardó mucho tiempo y generó una cantidad abrumadora de información.

Hay dos retos para esta clase: primero, interpretar ese reporte técnico para priorizar qué arreglar. Segundo, demostrar que se puede configurar un escaneo más eficiente. La tarea es ejecutar un nuevo análisis sobre el mismo objetivo utilizando una política diferente ("Basic Network Scan") para comparar los resultados y determinar si una configuración menos agresiva es suficiente para detectar los riesgos principales sin saturar la red.

## 2. Objetivos

Al completar este desafío, serás capaz de:
1.  **Interpretación de Resultados:** Analizar métricas de riesgo (CVSS) y diferenciar entre vulnerabilidades informativas y críticas en un reporte automatizado.
2.  **Configuración de Políticas:** Configurar y lanzar escaneos en Nessus seleccionando las plantillas adecuadas según la necesidad operativa.
3.  **Pensamiento Crítico:** Comparar la eficacia y eficiencia entre diferentes tipos de políticas de escaneo (Avanzado vs. Básico).

## 3. Pre-requisitos

* Haber completado exitosamente el Laboratorio Guiado de instalación y uso básico de Nessus.
* Tener acceso a una instancia de **Nessus Essentials o Pro Trial** (o superior) activa.
* Tener disponible la máquina *VICTIMA 1* en la instancia AWS propuesta
* Contar con el reporte "Advanced Scan" generado en la sesión guiada anterior (Se encuentra en los recursos de esta clase).

---

## 4. Instrucciones del Desafío

Este es un reto sencillo para comparar 2 reportes:

### Tarea 1: Análisis del Reporte "Advanced Scan"
Debe diseccionar el reporte generado previamente para extraer información accionable para la siguiente fase del pentest.

* **Instrucción:** Abre el reporte del "Advanced Scan" en Nessus y filtre los resultados.
* **Requisito A:** Identifica la vulnerabilidad más crítica (CVSS 9.0 o superior) que **no** requiera autenticación para ser explotada. Anote el ID del Plugin y el CVE asociado.
* **Requisito B:** Localiza al menos dos hallazgos que sean considerados "Informational" (Informativos) pero que revelen datos útiles sobre la arquitectura (ej. versión del S.O., NetBIOS name, Traceroute).

### Tarea 2: Ejecución de un Escaneo de Red Básico (Basic Network Scan)
Ahora, debes realizar su propio escaneo utilizando una política estándar de la industria.

* **Instrucción:** Crea un nuevo escaneo en Nessus apuntando a la misma dirección IP del objetivo utilizado en el escaneo avanzado.
* **Requisito A:** Selecciona la plantilla **"Basic Network Scan"**. A diferencia del escaneo avanzado, no modifiques los plugins ni active opciones de "fuerza bruta" o "escaneo web complejo". Manten la configuración predeterminada de la plantilla.
* **Requisito B:** Ejecuta el escaneo y cronometre el tiempo que tarda en completarse en comparación con el tiempo que tomó el "Advanced Scan".

### Tarea 3: Verificación 
Contrasta ambos para entender las diferencias.

* **Instrucción:** Compara los resultados de ambos escaneos (el Avanzado previo y el Básico actual).
* **Prueba de Concepto:** Genera un reporte en formato PDF o HTML del nuevo escaneo básico.
* **Verificación:** Responde técnicamente: *¿El escaneo básico detectó la misma vulnerabilidad crítica identificada en la Tarea 1? ¿Hubo alguna diferencia significativa en el número total de vulnerabilidades detectadas?*

---

## 5. Completitud

Para validar este desafío, tu reto debe concluir con la siguiente información:

1.  El nombre y CVE de la vulnerabilidad crítica analizada en la Tarea 1.
2.  El archivo exportado del reporte del "Basic Network Scan".
3.  Una tabla comparativa simple:
    * Tiempo de Advanced Scan vs. Tiempo de Basic Scan.
    * Cantidad de Vulnerabilidades Críticas (Advanced) vs. (Basic).
**4.  Comenta en los comentarios de la clase:**

*En un entorno de producción real con servidores legados (antiguos y frágiles), ¿por qué podría ser peligroso utilizar siempre la política de "Advanced Scan" con todas las opciones activadas en lugar de un "Basic Network Scan"?*
