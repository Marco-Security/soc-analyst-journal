# Día 7 — Remediation Tasks y seguimiento

**Fecha:** 24 de mayo de 2026
**Duración:** ~1 hora
**Área:** Threat & Vulnerability Management — Corrección
**Plataforma:** Microsoft Defender XDR → Administración de vulnerabilidades → Corrección

---

## Objetivo del día

Revisar el estado de las Remediation Tasks existentes en el tenant, analizar
los dispositivos afectados por la misconfiguration crítica del sensor EDR,
y documentar el problema de proceso que impide la corrección real.

---

## 1. Estado general de Remediation Tasks

Ruta: `Ir a administración de la exposición → Administración de vulnerabilidades → Corrección`

```
Actividades en curso:  3
Actividades vencidas:  3  ← el 100% de las tareas están vencidas
```

Las tres tareas activas están todas vencidas — ninguna fue completada
dentro del plazo establecido. Esto no es un problema técnico,
es un problema de proceso.

### Tareas existentes

| # | Nombre | Prioridad | Estado | Progreso |
|---|--------|-----------|--------|----------|
| 1 | Turn on Microsoft Defender for Endpoint sensor | Alto | Vencida | 0/27 |
| 2 | Turn on Microsoft Defender for Endpoint sensor | Alto | Vencida | — |
| 3 | Fix Microsoft Defender for Endpoint sensor data collection | Medio | Vencida | — |

Observación: las 3 tareas apuntan al mismo problema — el sensor EDR
desactivado en los dispositivos del tenant. No hay diversificación en
las tareas; todo el backlog de corrección está concentrado en una sola
misconfiguration sin resolver.

---

## 2. Análisis de la Task 1 — detalle

La tarea con mayor detalle disponible es la primera:

```
Nombre:     Turn on Microsoft Defender for Endpoint sensor
Estado:     Activo (vencida)
Progreso:   0 / 27 dispositivos corregidos
Prioridad:  Alto
Creada el:  13 may. 2026 10:15
Creada por: equipo4@labmxsc200.onmicrosoft.com
Vencida:    13 may. 2026 18:00
Nota:       "Revisar equipos"
Etiqueta:   COVID-19
```

### Hallazgos críticos de esta tarea

**1. Tiempo de vida: menos de 8 horas**
La tarea fue creada a las 10:15 y el vencimiento se fijó a las 18:00
del mismo día. Una tarea de remediación que involucra 27 dispositivos
no puede resolverse en 8 horas — el plazo fue irreal desde el inicio.

**2. Progreso: 0/27 después de 11 días**
Once días después de vencida, ningún dispositivo ha sido corregido.
La tarea existe en el sistema pero no genera acción real.

**3. Desconexión entre tarea y recomendación fuente**
Al navegar a la recomendación vinculada (`scid-2000`), el tab
"Actividades" muestra 0 elementos. Esto indica que las Remediation
Tasks del tenant no están correctamente vinculadas a la recomendación
fuente — fueron creadas de forma aislada, sin trazabilidad completa
en el sistema.

---

## 3. Recomendación fuente — scid-2000

Ruta: `Recomendaciones → Turn on Microsoft Defender for Endpoint sensor`

```
Estado:               Corrección necesaria
Categoría:            deviceMisconfiguration (EDR)
Config ID:            scid-2000
Etiquetas:            COVID-19 | Accesible desde Internet
Dispositivos exp.:    28 / 29
Actividades vinc.:    0 elementos  ← sin Remediation Tasks asociadas
```

La etiqueta **"Accesible desde Internet"** eleva la criticidad de esta
misconfiguration. Dispositivos sin sensor EDR que además tienen
exposición a internet son objetivos de alto valor para atacantes —
MDE no puede detectar ni responder a actividad maliciosa en estos
endpoints.

---

## 4. Dispositivos expuestos — análisis del CSV

Total de dispositivos afectados: **29 de 29** (el CSV incluye la VM
de lab maquinasc-200, que aparece como expuesta también).

### Distribución por sistema operativo

| OS | Dispositivos |
|----|-------------|
| Windows 10 | 15 |
| Windows Server 2025 | 12 |
| Windows 11 | 1 |
| **Total** | **28** |

Los **12 servidores Windows Server 2025** son el riesgo más alto.
Un servidor sin EDR activo es un punto ciego total para el SOC —
cualquier actividad maliciosa (movimiento lateral, exfiltración,
persistencia) pasaría desapercibida.

### Dispositivos sin reporte reciente (last seen 27 abr.)

Cinco dispositivos llevan casi 4 semanas sin enviar datos al sensor:

```
pc-test-defende   Windows Server 2025   27 abr. 2026 17:13
pc-testing-defe   Windows 10            27 abr. 2026 17:56
maquinadefender   Windows Server 2025   27 abr. 2026 18:00
maquinadefender   Windows Server 2025   27 abr. 2026 22:45
equipo21          Windows Server 2025   27 abr. 2026 23:16
```

Estos dispositivos no solo tienen el EDR desactivado — llevan semanas
sin reportar ningún dato. Desde la perspectiva del SOC, son
completamente ciegos.

### Dispositivo conocido: equipo10

equipo10 aparece en esta lista. Es el mismo dispositivo que analizamos
el Día 3 con 530 vulnerabilidades, 4 CVEs críticas y estado Inactive.
Confirma que es el endpoint con la peor postura de seguridad del
tenant — sin EDR, sin actualizaciones, sin visibilidad.

### Dispositivo: maquinasc-200

La VM de lab del tenant también aparece en la lista de expuestos.
A diferencia del resto, su last seen es del 4 de mayo — más reciente
que la mayoría, pero el sensor sigue sin estar activo correctamente.

---

## 5. Flujo de Remediation Task — cómo debería funcionar

Para el examen SC-200, el flujo correcto es:

```
TVM detecta misconfiguration o CVE
        ↓
Recomendación aparece en TVM con impacto calculado
        ↓
SOC Analyst hace clic en "Solicitar corrección"
        ↓
Se crea Remediation Task vinculada a la recomendación (scid)
        ↓
La tarea aparece en Intune como ticket para el equipo IT
        ↓
IT aplica la corrección en los dispositivos afectados
        ↓
TVM actualiza el progreso automáticamente
        ↓
Remediation Task se cierra cuando progreso = 100%
```

**Problema en este tenant:** El paso de vinculación no se realizó
correctamente. Las tareas existen en Corrección pero no están
enlazadas a la recomendación scid-2000, por lo que el progreso no
se trackea de forma automatizada.

---

## 6. Conclusiones del día

**1. El problema de proceso es el hallazgo principal del Día 7.**
No hay vulnerabilidades nuevas que descubrir — el problema ya estaba
identificado desde el Día 3. Lo que falla es el ciclo de corrección:
detección → tarea → seguimiento → cierre. Ese ciclo está roto.

**2. 28/29 dispositivos sin sensor EDR es el mayor punto ciego
del tenant.**
El SOC puede detectar amenazas en incidentes y alertas, pero no
puede hacer Live Response ni contener un ataque activo en ninguno
de esos 28 dispositivos.

**3. Las Remediation Tasks sin seguimiento real no agregan valor.**
Crear una tarea y fijarle un vencimiento de 8 horas no es gestión
de vulnerabilidades — es apariencia de gestión. Sin responsable
asignado, sin escalamiento y sin integración correcta con Intune,
las tareas son documentación muerta.

**4. Prioridad de corrección recomendada:**

```
1. Los 12 Windows Server 2025 sin EDR activo
   → Riesgo más alto: servidores expuestos sin visibilidad
2. Los 5 dispositivos sin reporte desde el 27 de abril
   → Posiblemente desconectados o comprometidos
3. equipo10 específicamente
   → Acumulación de riesgos: EDR + 530 CVEs + estado Inactive
4. El resto de Windows 10
   → Riesgo alto pero menor que servidores
```

---

## Lecciones aprendidas

- Una Remediation Task correctamente creada desde TVM se vincula
  automáticamente a la recomendación fuente — la tarea aparece en
  el tab "Actividades" de esa recomendación.
- Si el tab "Actividades" muestra 0 elementos, la tarea fue creada
  de forma manual sin vinculación correcta al scid.
- El progreso de una Remediation Task solo se actualiza
  automáticamente cuando fue creada desde la recomendación TVM
  usando "Solicitar corrección" — no si fue creada manualmente.
- Un dispositivo con last seen de semanas atrás puede estar
  offline, desconectado del dominio, o potencialmente comprometido.
  Requiere investigación activa, no solo una tarea vencida.

---

*SOC Analyst Journal — Ancla Lab | Marco | Día 7 de 20*
