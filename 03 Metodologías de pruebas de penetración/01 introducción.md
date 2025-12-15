# Metodología de Pruebas de Penetración y Análisis de Vulnerabilidades

El hacking ético deja de ser un pasatiempo y se convierte en una profesión cuando se le aplica **metodología**. Sin un proceso estructurado, una auditoría es solo un caos de intentos aleatorios. En esta clase, aprenderemos cómo estructurar legal y técnicamente un compromiso de seguridad y qué estándares internacionales guían nuestro trabajo.

---

> **⚠️ Disclaimer Ético:** Esta información se proporciona con fines exclusivamente educativos y para la formación de profesionales en ciberseguridad. La ejecución de estos ataques en redes sin autorización explícita es ilegal y penada por la ley. Practica siempre en tu propio laboratorio controlado.

---

## Pre-Engagement

Antes de lanzar el primer escáner, debemos construir el marco operativo y legal. Esta fase es crítica: un error aquí puede convertir un trabajo legítimo en un delito o en una demanda civil.

### 1. Contratos y Disclaimers

El pentesting simula un ciberataque, lo cual es ilegal por defecto.
* **Contrato de Servicios o Statement Of Work:** Define la relación comercial, confidencialidad (NDA) y pagos.
* **Autorización de Prueba:** El documento más importante. Es el permiso explícito ("Get out of jail free card") firmado por un representante legal de la empresa, autorizándote a atacar sus sistemas.
* **Disclaimer:** Una cláusula donde el cliente reconoce que las pruebas pueden causar inestabilidad o caída de servicios, eximiendo al auditor de daños operativos accidentales (siempre que se siga la metodología).

### 2. Reglas del Engagement (RoE)
Es el manual operativo del ataque. Define el "cómo" y el "cuándo".
* **Horarios:** ¿Se puede atacar en horario de oficina o solo de madrugada?
* **Vectores permitidos:** ¿Se permite ingeniería social? ¿Ataques físicos? ¿DDoS (casi siempre prohibido)?
* **Procedimiento de comunicación:** Si encuentras una falla crítica (ej: base de datos expuesta), ¿a quién llamas a las 3:00 AM?

### 3. Alcance (Scope) y Lista de Activos
Define el campo de batalla.
* **White Box vs. Black Box vs Gray Box:** ¿Cuánta información vamos a disponer del activo inicialmentes?
* **Lista de Objetivos (Target List):** IPs, Dominios, URLs específicas.
* **Out of Scope (Fuera de Alcance):** Activos que **NO** se deben tocar bajo ninguna circunstancia (ej: servidores de producción críticos, IPs de terceros proveedores).

---

## Metodologías y Estándares Internacionales

Usamos estándares probados para garantizar calidad y repetibilidad. Podemos dividirlos en tres categorías:

### A. El Estándar de Ejecución (El Proceso)

**PTES (Penetration Testing Execution Standard)**
Es el estándar más aceptado para definir el *ciclo de vida* de un pentest. Divide el trabajo en 7 fases claras:
1.  Pre-acuerdo (Pre-engagement).
2.  Inteligencia (Information Gathering).
3.  Modelado de Amenazas.
4.  Análisis de Vulnerabilidades.
5.  Explotación.
6.  Post-Explotación (Determinar el valor del dato robado).
7.  Reporte.

### B. Guías Técnicas y Métricas (El Cómo y Cuánto)

**OSSTMM (Open Source Security Testing Methodology Manual)**
A diferencia de otros que buscan "bugs", OSSTMM es científico y se centra en la **seguridad operativa**.
* Se basa en hechos y métricas, no en suposiciones.
* Introduce el concepto de **RAVs (Risk Assessment Values)** para calcular una puntuación de seguridad real.
* Cubre seguridad humana, física, inalámbrica, de telecomunicaciones y redes de datos.

**NIST SP 800-115**
Publicación Especial. Es la guía del gobierno de EE.UU. (National Institute of Standards and Technology).
* Es el estándar de facto para cumplimiento normativo (FISMA, HIPAA).
* Es menos agresivo que PTES y se enfoca mucho en la documentación y en procedimientos formales de evaluación.

**ISSAF (Information Systems Security Assessment Framework)**
Aunque es un proyecto más antiguo, destaca por su nivel de detalle técnico.
* Vincula cada paso de la evaluación con herramientas específicas. Es excelente para auditores que necesitan una "lista de chequeo" técnica muy granular.

### C. La GOAT de las Pruebas en Aplicaciones Web 

**OWASP (Open Web Application Security Project)**
No es una metodología de red general, sino el estándar absoluto para **seguridad web y de software**.
* **OWASP Top 10:** Lista de las 10 vulnerabilidades más críticas del momento.
* **Testing Guide (WSTG):** Un "libro de cocina" con pruebas específicas para cada posible error en una web (SQL Injection, XSS, fallos de sesión). Si auditas una web, usas OWASP.

---

## Cumplimiento y Riesgo

### Análisis de Vulnerabilidades vs. Pentesting
* **Análisis de Vulnerabilidades:** Es como caminar por un vecindario revisando si las puertas están abiertas (usualmente automatizado). Identifica brechas potenciales.
* **Pentesting:** Es entrar por la puerta abierta y ver hasta dónde puedes llegar (robar la joya de la corona). Confirma el riesgo real.

### Cumplimiento (Compliance)
Muchas metodologías se aplican porque la ley lo exige. Un reporte de pentest basado en **NIST** u **OSSTMM** sirve como evidencia auditora para normas como:
* **PCI-DSS:** Para procesar tarjetas de crédito.
* **ISO 27001:** Estándar de seguridad de la información.
* **GDPR/RGPD:** Protección de datos personales.

El objetivo final no es "hackear", sino reducir el **Riesgo** a un nivel aceptable para que la organización pueda operar con confianza.
