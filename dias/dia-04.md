# Día 4 — Action Center y ciclo de vida de un incidente

**Fecha:** 21/05/2026
**Herramientas:** Microsoft Defender XDR, MDE
**Autor:** Marco — Ancla Lab

---

## Objetivo

Revisar el Action Center para entender qué acciones automatizadas ejecutó AIR,
y analizar el ciclo de vida completo del incidente ID 37 a través de su log
de actividades.

---

## Action Center

Sin acciones pendientes al momento de la revisión.

**Historial — últimas 3 acciones:**

| Fecha | Tipo | Detalle | Dispositivo | Estado |
|-------|------|---------|-------------|--------|
| 18 may 23:17 | Comando de respuesta inmediata | connect | eq13 | - |
| 18 may 23:16 | Comando de respuesta inmediata | connect | equipo18 | - |
| 15 may 16:20 | Deshacer una acción | unconfirmed 95867.crdownload | PC-TEST-DEFENDE | Timed Out |

Las dos primeras corresponden a sesiones de Live Response — un analista
se conectó remotamente a eq13 y equipo18 para investigar algo el 18 de mayo.

---

## Caso EICAR — flujo de AIR

La acción del 15 de mayo es un caso interesante. El archivo
`unconfirmed 95867.crdownload` fue descargado por Equipo 6 en PC-TEST-DEFENDE.
EICAR es un archivo de prueba estándar usado para verificar que el antivirus
funciona — no es malware real.

**Lo que pasó:**

```
Equipo 6 descarga EICAR (prueba intencional)
        ↓
Windows Defender lo bloquea inmediatamente
        ↓
AIR intenta deshacer la acción sobre el archivo
        ↓
Ya estaba remediado por el antivirus → acción "Omitida"
        ↓
Dispositivo quedó Inactive → Timed Out
```

Detalles del archivo:
- Veredicto: Malicioso
- VirusTotal: 62/68 motores lo detectan
- Técnica MITRE: T1204.001 — Malicious Link
- Remediación: Pre-remediada por Windows Defender

Esto muestra que el antivirus y AIR trabajan en capas. Cuando el antivirus
actúa primero, AIR reconoce que no hay nada que hacer y omite la acción.

---

## Ciclo de vida del incidente ID 37

El log de actividades muestra la cronología completa desde la detección
hasta el cierre:

| Fecha | Actividad | Tipo |
|-------|-----------|------|
| 3 may 23:08 | Alerta 1 correlacionada automáticamente al incidente 37 | Vínculo de alerta |
| 3 may 23:19 | Alerta 2 correlacionada automáticamente al incidente 37 | Vínculo de alerta |
| 9 may 00:49 | Incidente asignado a equipo6@labmxsc200 | Cambio de asignación |
| 18 may 08:40 | Clasificado como "Positivo verdadero" | Cambio de clasificación |
| 18 may 08:40 | Determinación cambiada a "Malware" | Cambio de determinación |
| 18 may 08:40 | Etiquetas agregadas: TA0005:Defense-Evasion | Cambio de etiqueta |
| 18 may 11:14 | Reasignado de equipo6 a arascon | Cambio de asignación |
| 18 may 11:14 | Estado cambiado de "Active" a "Resolved" | Cambio de estado |

**Tiempo de respuesta total: 15 días** (3 mayo → 18 mayo)

En un SOC real, un True Alert de severidad Alta debería resolverse en horas,
no en días. Este tiempo de respuesta indica falta de procesos definidos o
capacidad insuficiente del equipo.

---

## Hallazgos

1. El Action Center sin acciones pendientes indica que AIR está en modo
   full-automated o que no hay amenazas activas que requieran aprobación manual.

2. El caso EICAR muestra un comportamiento correcto del sistema — las capas
   de defensa funcionaron en orden: antivirus primero, AIR como respaldo.

3. 15 días de tiempo de resolución para una alerta Alta es un indicador
   de madurez baja del SOC. El SLA típico para incidentes de este tipo
   es de 4 a 24 horas.

4. La correlación automática de las dos alertas al mismo incidente funcionó
   correctamente — Sentinel/XDR identificó que eran parte del mismo ataque
   sin intervención humana.

---

## Lecciones aprendidas

- El log de actividades de un incidente es la fuente de verdad para entender
  quién hizo qué y cuándo. Es el equivalente a una cadena de custodia.
- AIR no reemplaza al analista — complementa. Cuando el antivirus ya actuó,
  AIR omite la acción. Cuando no hay remediación previa, AIR propone o ejecuta.
- El tiempo de resolución es una métrica clave del SOC. Un incidente de
  severidad Alta sin resolver por 15 días es un hallazgo en sí mismo.

---

*SOC Analyst Journal — Ancla Lab | Marco | Día 4 de 20*
