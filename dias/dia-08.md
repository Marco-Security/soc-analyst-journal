# Día 8 — Defender for Cloud: Secure Score

**Fecha:** 24 de mayo de 2026
**Duración:** ~45 minutos
**Área:** Microsoft Defender for Cloud — Posición de seguridad
**Plataforma:** portal.azure.com → Microsoft Defender for Cloud

---

## Objetivo del día

Explorar Microsoft Defender for Cloud por primera vez en este journal,
analizar el estado actual del Secure Score en la suscripción de Azure,
identificar recomendaciones activas y entender por qué el score no
se está calculando.

---

## 1. Contexto — qué es Defender for Cloud y cómo difiere de MDE

Antes de entrar a los datos, es importante entender qué estamos mirando:

| Característica | MDE (Defender for Endpoint) | Defender for Cloud |
|---|---|---|
| Enfoque | Endpoints (laptops, servidores, workstations) | Recursos cloud (VMs, DBs, storage, containers) |
| Donde vive | security.microsoft.com | portal.azure.com |
| Métrica principal | Exposure Score + Secure Score | Secure Score (diferente) |
| Qué protege | Dispositivos con sensor instalado | Suscripciones de Azure, AWS, GCP |
| Requiere agente | Sí (sensor MDE) | Depende del plan (algunos usan AMA) |

El **Secure Score de Defender for Cloud** mide la postura de seguridad
de los recursos cloud — no de los endpoints. Son dos scores distintos
que coexisten en el ecosistema Microsoft.

---

## 2. Dashboard principal — estado del tenant

Ruta: `portal.azure.com → Microsoft Defender for Cloud → Información general`

```
Suscripciones de Azure:   1
Recursos evaluados:       0  ← nada siendo analizado
Vías de ataque:           0
Alertas de seguridad:     0

Posición de seguridad:
  Recomendaciones críticas:   0
  Vías de ataque activas:     0
  Recomendaciones vencidas:   0/0
  Puntuación total:           N/D  ← no calculable sin recursos

Protecciones de cargas de trabajo:
  Cobertura de recursos:      NaN%  ← 0 planes activos
  Alertas por gravedad:       ninguna

Inventario:
  Recursos totales:           0
```

### Diagnóstico inmediato

El tenant tiene Defender for Cloud disponible — la suscripción de Azure
está conectada — pero el servicio no está evaluando ningún recurso.

La causa es clara: **no hay Defender Plans habilitados**. Sin un plan
activo, Defender for Cloud no sabe qué recursos proteger ni cómo
evaluarlos, por lo que el Secure Score no puede calcularse.

Esto no es un error del sistema — es el estado esperado de una
suscripción donde nadie ha completado la configuración inicial.

---

## 3. Recomendaciones — análisis

Ruta: `Defender for Cloud → Recomendaciones`

```
Crítico:  0
Alto:     0
Medio:    0
Bajo:     0
Rutas de ataque activas:      0
Recomendaciones vencidas:     0

Resultado: "No se encontraron recomendaciones"
```

La ausencia total de recomendaciones confirma el diagnóstico — sin
recursos evaluados, no hay nada sobre qué recomendar. Defender for
Cloud soporta múltiples entornos:

```
Azure:       1 suscripción visible, 0 incorporado efectivamente
AWS:         0 incorporado
GCP:         0 incorporado
GitHub:      0
AzureDevOps: 0
GitLab:      0
Docker Hub:  0
JFrog:       0
```

El dato de Azure dice "Se incorporan todas las suscripciones válidas"
pero 0 están generando evaluaciones. Esto ocurre porque la
incorporación automática de suscripciones no activa la evaluación
por sí sola — requiere que al menos un Defender Plan esté habilitado
en esa suscripción.

---

## 4. Configuración — entornos conectados

Ruta: `Defender for Cloud → Configuración`

La pantalla muestra la página de onboarding de entornos:

```
Microsoft Azure:    0 incorporado
Amazon Web Services: 0 incorporado
Google Cloud Platform: 0 incorporado
Entornos adicionales: GitHub, GitLab, Docker Hub, JFrog, AzureDevOps
```

Y los pasos pendientes que el portal mismo recomienda:

```
1. Configuración del contacto de seguridad para alertas
   → Sin esto, nadie recibe notificaciones de alertas críticas

2. Configurar directivas de seguridad
   → Sin estándares aplicados, no hay baseline de cumplimiento

3. Explorar informes integrados (Libros)
   → Visibilidad de datos de seguridad en Workbooks
```

Ninguno de estos pasos está completado en el tenant.

---

## 5. Cómo funciona el Secure Score de Defender for Cloud

Para el SC-200, el mecanismo del score es importante:

```
Defender Plan habilitado (ej: Defender for Servers)
        ↓
Defender for Cloud evalúa los recursos cubiertos por ese plan
        ↓
Genera recomendaciones por recurso (controles de seguridad)
        ↓
Cada control tiene un peso en puntos
        ↓
Score = puntos obtenidos / puntos máximos posibles × 100
        ↓
El score sube cuando se implementan las recomendaciones
```

Sin el primer paso (plan habilitado), toda la cadena se rompe y el
resultado es N/D.

### Diferencia con el Exposure Score de TVM

| Métrica | Origen | Mide |
|---|---|---|
| Secure Score (Defender XDR) | MDE | Controles de seguridad en endpoints |
| Exposure Score (TVM) | MDE / EASM | Superficie de ataque y vulnerabilidades |
| Secure Score (Defender for Cloud) | DfC | Postura de recursos cloud |

Son tres métricas distintas que se complementan — ninguna reemplaza
a las otras.

---

## 6. Recursos en Azure — estado actual

La VM `maquinasc-200` (anclalab-vm) fue creada en Azure para el lab de
onboarding del Día 1. Sin embargo, no aparece en el inventario de
Defender for Cloud porque ningún Defender Plan la está monitoreando.

Desde la perspectiva de Defender for Cloud, esa VM es invisible — podría
tener misconfigurations de red, puertos expuestos, sin patch management,
o credenciales débiles, y el servicio no lo reportaría.

---

## 7. Conclusiones del día

**1. Defender for Cloud está instalado pero sin configurar.**
La suscripción de Azure existe, el servicio es accesible, pero 0 planes
activos significa 0 evaluaciones, 0 recomendaciones y Secure Score N/D.

**2. El Secure Score N/D no es un score de 0 — es peor.**
Un score de 0% significaría que Defender for Cloud evaluó los recursos
y todos fallaron los controles. N/D significa que nadie siquiera
habilitó la evaluación. El tenant tiene un punto ciego completo en
su postura cloud.

**3. La VM de lab está sin protección desde Defender for Cloud.**
`maquinasc-200` no aparece en el inventario. Si se ejecutaran técnicas
de ataque contra esa VM a nivel de configuración de Azure (puertos
expuestos, acceso público, identidad mal configurada), Defender for
Cloud no generaría ninguna alerta.

**4. El Día 9 es la continuación natural.**
Los Defender Plans son la pieza que falta para que todo esto funcione.
Sin entender qué planes existen y qué protegen, no es posible tomar
una decisión de configuración informada.

---

## Comparativa de postura: Días 1-8

| Área | Métrica | Valor |
|---|---|---|
| Identidad (MDE) | Usuarios sin MFA | 27/27 |
| Endpoints (MDE) | Dispositivos sin EDR | 28/29 |
| Vulnerabilidades (TVM) | Exposure Score | 54/100 |
| Vulnerabilidades (TVM) | CVEs aprovechables | 12 |
| Cloud (Defender for Cloud) | Secure Score | N/D |
| Cloud (Defender for Cloud) | Recursos evaluados | 0 |
| Cloud (Defender for Cloud) | Planes activos | 0 |

El tenant tiene puntos ciegos en las tres capas: identidad, endpoints
y cloud. El hallazgo de hoy añade una capa nueva al mapa de riesgo que
venimos construyendo desde el Día 1.

---

## Lecciones aprendidas

- Defender for Cloud y MDE son servicios distintos que viven en portales
  distintos (Azure vs security.microsoft.com) y protegen capas distintas.
- El Secure Score de Defender for Cloud no se calcula hasta que al menos
  un Defender Plan está habilitado — sin plan, no hay evaluación.
- "0 incorporado" en la configuración de entornos significa que la
  suscripción está visible pero no activamente monitoreada.
- La presencia del servicio en el portal no implica protección activa —
  es necesario configurar planes y directivas explícitamente.

---

*SOC Analyst Journal — Ancla Lab | Marco | Día 8 de 20*
