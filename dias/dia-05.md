# Día 5 — Reporte semanal: Semana 1

**Fecha:** 22/05/2026
**Autor:** Marco — Ancla Lab
**Período:** 20/05/2026 – 22/05/2026

---

## Resumen ejecutivo

El tenant presenta una postura de seguridad media-baja con un Secure Score
global de 40%. Se identificaron hallazgos críticos en tres áreas: gestión
de identidad, dispositivos con riesgo alto y un incidente activo con
prioridad máxima sin resolver. El volumen de incidentes activos (26,869
en 30 días) con cero Automation Rules configuradas indica que todo el
triage se está haciendo de forma manual, lo cual no es sostenible.

---

## Métricas clave — semana 1

| Métrica | Valor |
|---------|-------|
| Secure Score global | 40% |
| Usuarios en riesgo | 15 |
| Dispositivos con riesgo alto | 7 |
| Dispositivos con exposición alta | 13 |
| Usuarios sin MFA | 27 de 27 |
| Vulnerabilidades críticas detectadas | 4 (solo en equipo10) |

---

## Actividades realizadas

**Día 1** — Revisión del dashboard y establecimiento del baseline del tenant.
Se identificó que el Secure Score de Identidad está en 21.95% y que ningún
usuario tiene MFA registrado.

**Día 2** — Investigación del incidente ID 37 (Multi-stage incident involving
Persistence & Defense Evasion). Se analizó la cadena de ataque completa:
PowerShell ejecutado remotamente modificó el registro de Windows para
desactivar Microsoft Defender, seguido del registro de un servicio sospechoso
para lograr persistencia. Tácticas MITRE: T1562.001 y TA0003.

**Día 3** — Revisión de los 7 dispositivos con riesgo alto. Se analizó
equipo10 en detalle: EDR desactivado, 530 vulnerabilidades detectadas
(4 críticas con CVSS 9.8 y 9.6), sin exámenes de antivirus realizados
y con estado Inactive. CVE-2025-60724 lleva más de 6 meses sin parchear.

**Día 4** — Revisión del Action Center y ciclo de vida del incidente ID 37.
Se documentó que el incidente tardó 15 días en resolverse desde la detección
hasta el cierre — tiempo de respuesta inaceptable para una alerta de
severidad Alta.

---

## Hallazgos principales

**1. Incidente activo con prioridad 100**
Hay un incidente en curso con puntuación de prioridad máxima que requiere
investigación inmediata. Es la acción más urgente de la próxima semana.

**2. 27 usuarios sin MFA**
El 100% de los usuarios del tenant no tiene MFA registrado. Esto convierte
cualquier contraseña comprometida en acceso total a la cuenta. Es el
hallazgo de mayor impacto en el Secure Score de Identidad (21.95%).

**3. 7 dispositivos con riesgo alto y EDR problemático**
Varios dispositivos tienen el sensor EDR desactivado o con comunicación
fallida. Sin telemetría, MDE no puede detectar actividad maliciosa en
tiempo real en esos endpoints.

**4. Vulnerabilidades críticas sin parchear**
Solo en equipo10 se detectaron 4 vulnerabilidades críticas, una de ellas
con más de 6 meses sin parchear (CVE-2025-60724, CVSS 9.8). El inventario
completo de vulnerabilidades en los 35 dispositivos aún no ha sido revisado.

**5. Cero Automation Rules**
El tenant no tiene ninguna regla de automatización configurada. Con casi
27,000 incidentes en 30 días, el triage manual es inviable y genera
alert fatigue en el equipo.

---

## Plan de acción — semana 2

1. Investigar el incidente activo con prioridad 100
2. Implementar MFA para todos los usuarios del tenant
3. Revisar y corregir el estado del sensor EDR en los 7 dispositivos
   con riesgo alto
4. Iniciar revisión de vulnerabilidades críticas en el resto de dispositivos
5. Configurar las primeras Automation Rules para suprimir falsos positivos
   conocidos y reducir el volumen de incidentes

---

*SOC Analyst Journal — Ancla Lab | Marco | Día 5 de 20*
