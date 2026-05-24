# Día 10 — Reporte semanal: Semana 2

**Fecha:** 24 de mayo de 2026
**Período cubierto:** Días 6 al 9
**Área:** TVM + Defender for Cloud
**Tipo:** Reporte ejecutivo semanal

---

## Resumen ejecutivo

La Semana 2 expandió el análisis más allá de los endpoints hacia dos
nuevas capas: la gestión de vulnerabilidades a profundidad (TVM) y la
postura de seguridad en cloud (Defender for Cloud). El hallazgo
central de la semana es que el tenant tiene puntos ciegos en las tres
capas fundamentales de su infraestructura — identidad, endpoints y
cloud — y que los procesos de corrección existentes no están
funcionando.

---

## Métricas: Semana 1 vs Semana 2

| Métrica | Baseline Día 1 | Estado Semana 2 | Tendencia |
|---------|---------------|-----------------|-----------|
| Secure Score global (MDE) | 40% | 40% | → Sin cambio |
| Usuarios en riesgo | 15 | 15 | → Sin cambio |
| Usuarios sin MFA | 27/27 | 27/27 | → Sin cambio |
| Dispositivos riesgo alto | 7 | 7 | → Sin cambio |
| Dispositivos sin EDR | — | 28/29 | Confirmado |
| Exposure Score (TVM) | 54/100 | 54/100 | → Sin cambio |
| CVEs críticas | 6 | 6 | → Sin cambio |
| CVEs aprovechables | 12 | 12 | → Sin cambio |
| Remediation Tasks activas | 0 | 3 | ↑ Creadas |
| Remediation Tasks vencidas | 0 | 3 (100%) | Todas vencidas |
| Progreso de corrección | — | 0/27 | Sin acción |
| Secure Score (Defender for Cloud) | — | N/D | No calculable |
| Recursos evaluados (DfC) | — | 0 | Sin evaluación |
| Defender Plans activos | — | 0/14 | Sin cobertura |

**Conclusión de métricas:** Ningún indicador mejoró en dos semanas.
Los que existían siguen igual; los nuevos que descubrimos esta semana
son todos críticos.

---

## Hallazgos por día

### Día 6 — TVM a fondo

**Exposure Score: 54/100** — clasificado como exposición alta.

Las 713 vulnerabilidades del tenant se distribuyen así:

```
Críticas:      6    ← requieren acción inmediata
Aprovechables: 12   ← tienen exploit público disponible
Totales:       713
```

El quick win de mayor impacto disponible en el tenant: actualizar
Edge Chromium en 29 dispositivos resolvería 204 CVEs de una vez y
reduciría el Exposure Score en 31.69 puntos — de 54 a ~22.

CVE más crítica identificada: CVE-2026-8580 (Use After Free en
Chrome/Edge, CVSS 9.6). Accesible desde internet, 28 dispositivos
expuestos. Esta CVE tiene exploit público y afecta a casi todo el
tenant.

---

### Día 7 — Remediation Tasks y seguimiento

**Hallazgo principal: el ciclo de corrección está roto.**

Las 3 Remediation Tasks del tenant:
- Todas vencidas desde el día de su creación
- Progreso acumulado: 0 dispositivos corregidos
- Task 1 fue creada a las 10:15 con vencimiento a las 18:00
  del mismo día — plazo irreal para corregir 27 dispositivos
- El tab "Actividades" en la recomendación scid-2000 muestra
  0 elementos — las tareas no están vinculadas correctamente
  a la recomendación fuente

**28/29 dispositivos sin sensor EDR activo**, con la siguiente
distribución de riesgo:

```
12 × Windows Server 2025  ← riesgo más alto
15 × Windows 10
 1 × Windows 11

5 dispositivos sin reporte desde el 27 de abril (casi 4 semanas)
→ posiblemente offline o comprometidos
```

---

### Día 8 — Defender for Cloud: Secure Score

**Secure Score: N/D** — no calculable porque no hay recursos evaluados.

```
Suscripciones:        1
Recursos evaluados:   0
Recomendaciones:      0
Vías de ataque:       0
Cobertura:            NaN%
```

Distinción importante: N/D no es lo mismo que 0%.

- Score de 0% → Defender for Cloud evaluó los recursos y todos fallaron
- Score N/D → nadie habilitó la evaluación

El tenant está en el estado peor posible desde la perspectiva de
Defender for Cloud.

---

### Día 9 — Defender Plans

**0 de 14 planes activos** en la suscripción de Azure.

Planes identificados y su relevancia para el tenant:

```
Alta prioridad (recursos presentes):
  Defender CSPM          → activa Secure Score y recomendaciones
  Defender for Servers   → cubre maquinasc-200
  Defender for Storage   → cubre workspaces de Sentinel
  Defender for Resource Manager → cubre toda la suscripción

Media prioridad:
  Defender for Containers
  Defender for Key Vault

No aplica / deprecated:
  DNS, K8s legacy, Container Registry legacy
```

Limitación operacional identificada: la cuenta `equipo11` no tiene
permisos de Owner/Contributor sobre la suscripción — puede documentar
gaps pero no activar planes. Requiere escalamiento.

---

## Comparativa: Exposure Score (TVM) vs Secure Score (Defender for Cloud)

Para el SC-200, entender la diferencia entre estas dos métricas es
fundamental:

| Característica | Exposure Score (TVM) | Secure Score (DfC) |
|---|---|---|
| Dónde vive | security.microsoft.com | portal.azure.com |
| Qué mide | Superficie de ataque en endpoints | Postura de recursos cloud |
| Rango | 0-100 (menor = mejor) | 0-100% (mayor = mejor) |
| Qué sube el score | Parchear CVEs, corregir configuraciones | Implementar recomendaciones de DfC |
| Estado en este tenant | 54/100 (exposición alta) | N/D (sin evaluación) |
| Requiere agente | Sí (sensor MDE en endpoints) | Depende del plan |

Son métricas complementarias que miden capas distintas. Un tenant
puede tener Exposure Score bajo (endpoints bien parcheados) y Secure
Score bajo (cloud mal configurado), o viceversa. En este tenant
ambos son malos.

---

## Mapa de riesgo acumulado — Semanas 1 y 2

```
CAPA DE IDENTIDAD
  └─ 27/27 usuarios sin MFA
  └─ 15 usuarios en riesgo activo

CAPA DE ENDPOINTS
  └─ 28/29 dispositivos sin sensor EDR activo
  └─ Exposure Score 54/100
  └─ 12 CVEs con exploit público
  └─ CVE-2026-8580 (CVSS 9.6) sin parchear
  └─ 5 dispositivos sin reporte desde 27 abr.
  └─ equipo10: 530 CVEs + EDR off + Inactive

CAPA DE PROCESOS
  └─ 3 Remediation Tasks vencidas, 0% completado
  └─ Tasks sin vinculación correcta a recomendaciones
  └─ Tiempo de resolución promedio incidentes: 15 días

CAPA CLOUD
  └─ 0/14 Defender Plans activos
  └─ Secure Score N/D
  └─ maquinasc-200 sin evaluación cloud
  └─ Workspaces de Sentinel sin cobertura DfC
```

---

## Plan de acción — Semana 3

La Semana 3 entra a Sentinel: Analytics Rules, Automation Rules
y Playbooks. Antes de eso, estos son los hallazgos que deberían
escalarse si este fuera un SOC real:

**Inmediato (esta semana):**
1. Activar CSPM básico (gratuito) en la suscripción de Azure
   → Sin costo, activa Secure Score y recomendaciones base
2. Renovar las 3 Remediation Tasks vencidas con plazos realistas
   → Asignar responsable, no solo crear la tarea
3. Investigar los 5 dispositivos sin reporte desde 27 abr.
   → Confirmar si están offline o comprometidos

**Esta semana (Semana 2, si hubiera recursos):**
4. Habilitar Defender for Servers para maquinasc-200
5. Parchear Edge en los 29 dispositivos (-31.69 Exposure Score)
6. Activar MFA para los 27 usuarios (mayor impacto en Secure Score MDE)

**Semana 3:**
7. Crear Analytics Rules en Sentinel para detección proactiva
8. Configurar Automation Rules para reducir alert fatigue
9. Implementar primer Playbook de respuesta automática

---

## Lecciones aprendidas de la semana

- TVM y Defender for Cloud son plataformas distintas que miden
  capas distintas — ninguna reemplaza a la otra.
- Un Exposure Score alto (54/100) y un Secure Score N/D juntos
  significan que el tenant está ciego tanto en endpoints como en cloud.
- Las Remediation Tasks solo tienen valor si están vinculadas
  correctamente a la recomendación fuente y tienen un responsable real.
- Defender for Cloud requiere planes activos para generar cualquier
  evaluación — la presencia del servicio en el portal no implica
  protección.
- El CSPM básico es gratuito y debería ser el primer paso en
  cualquier suscripción de Azure nueva.

---

*SOC Analyst Journal — Ancla Lab | Marco | Día 10 de 20*
