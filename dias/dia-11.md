# Día 11 — Sentinel: Automation Rules

**Fecha:** 25 de mayo de 2026
**Duración:** ~1.5 horas
**Área:** Microsoft Sentinel — Automatización (SOAR)
**Plataforma:** portal.azure.com → Microsoft Sentinel → sentinel-lab-2 → Automatización

---

## Objetivo del día

Explorar la sección de Automation Rules en Sentinel, analizar las reglas
existentes en el tenant, entender su estructura y crear una regla propia
para suprimir falsos positivos conocidos del lab.

---

## 1. Contexto — Automation Rules en el ciclo SOC

Las Automation Rules son el componente SOAR de Sentinel. Actúan después
de que ya existe un incidente — no lo detectan, lo procesan.

```
Analytics Rule detecta actividad sospechosa → genera incidente
        ↓
Automation Rule evalúa el incidente
        ↓
Ejecuta acciones automáticas sin intervención humana
```

**Qué NO hacen:** no detectan amenazas, no ejecutan KQL, no generan
alertas. Solo reaccionan a incidentes ya existentes.

**Qué SÍ hacen:** cambiar status, cambiar severidad, asignar propietario,
agregar tags, agregar tareas, disparar un Playbook.

---

## 2. Acceso — problema de contexto de workspace

Al intentar acceder desde el portal unificado de Defender XDR
(`security.microsoft.com → Automatización`) el sistema devolvió:

```
Request failed with status code 403
```

Causa: el portal de Defender XDR perdió el contexto del workspace
`sentinel-lab` al reiniciar la sesión. Sin ese contexto, las
peticiones a la API de Sentinel fallan con 403.

**Solución:** navegar directamente desde Azure:
```
portal.azure.com → Microsoft Sentinel → 
seleccionar workspace → Automatización
```

Esta ruta establece el contexto del workspace antes de cargar la
sección de automatización, evitando el error 403.

**Observación importante:** el workspace accesible desde Azure para
`equipo11` es `sentinel-lab-2`, no `sentinel-lab`. El tenant tiene
múltiples workspaces de Sentinel — las reglas del tenant principal
viven en `sentinel-lab` y no son accesibles directamente desde esta
cuenta. Esto es consistente con la limitación de permisos identificada
en días anteriores.

---

## 3. Estado inicial — Automation Rules en sentinel-lab-2

```
Automation rules:   0
Enabled rules:      0
Enabled playbooks:  0
```

El workspace `sentinel-lab-2` no tenía ninguna regla de automatización
antes de este día. Estado limpio para crear la primera regla.

---

## 4. Reglas existentes en sentinel-lab (referencia)

Revisadas el día anterior desde el portal de Defender XDR antes de
perder el contexto. Ambas son Standard rules en el workspace
`sentinel-lab`:

### Regla 1 — Multiples fallos de MFA

```
Tipo:      Standard rule
Trigger:   When incident is created
Workspace: sentinel-lab
Condición: Analytic rule name Contains "MFA Fallidos"
Acción:    Add tags → "MFA ERROR"
Expiración: Indefinite
Order:     1
Status:    Active
Creada por: Alan R...
```

Función: etiqueta automáticamente los incidentes de MFA para
facilitar su identificación en la cola. Acción simple de un solo paso.

### Regla 2 — Fallos multiples de MFA

```
Tipo:      Standard rule
Trigger:   When incident is created
Workspace: sentinel-lab
Condición: Analytic rule name Contains "MAF F"
Acciones:  1. Change severity → High
           2. Add tags → "MFA error"
Expiración: Indefinite
Order:     1
Status:    Active
Creada por: Equipo...
```

Función: escala la severidad a High y etiqueta el incidente.
Demuestra que una Automation Rule puede encadenar múltiples acciones.

**Bug identificado:** la condición dice `Contains "MAF F"` — parece
un typo de "MFA F". Si la Analytics Rule se llama "MFA Fallidos",
esta condición no haría match y la regla nunca se dispararía. Es
un error de proceso: la regla existe pero está inactiva de facto.

---

## 5. Estructura de una Automation Rule — campos disponibles

Al crear una nueva regla, el formulario expone todos los campos
posibles para una Standard rule:

### Triggers disponibles
```
When incident is created    ← el más común
When incident is updated
When alert is created
```

### Conditions disponibles
```
Incident provider          (Equals / Not equals)
Analytic rule name         (Contains / Not contains)
Incident title             (Contains / Not contains)
Severity                   (Equals / Not equals)
Status                     (Equals / Not equals)
Tactics                    (Contains / Not contains)
Tag                        (Contains / Not contains)
```

### Actions disponibles
```
Run playbook               ← dispara un Logic App externo
Change status              ← Active / Closed / New
Change severity            ← High / Medium / Low / Informational
Assign owner               ← asigna responsable del incidente
Add tags                   ← etiqueta para clasificación
Add task                   ← agrega tarea al incidente
```

**Nota sobre "Change status → Closed":** al cerrar un incidente
con una Automation Rule, el portal solicita opcionalmente una
clasificación (True Positive, False Positive, Benign Positive,
Undetermined) y un comentario. Sentinel también advierte:
"Closing the incident will archive the associated team".

---

## 6. Regla creada — Ancla-Lab: Suprimir FP pruebas EICAR

### Configuración final

```
Nombre:    Ancla-Lab - Suprimir FP pruebas EICAR
Tipo:      Standard rule
Workspace: sentinel-lab-2
Trigger:   When incident is created
Condición: Analytic rule name → Contains → All
Acción 1:  Change status → Closed
Acción 2:  Add tags → FALSE-POSITIVE-LAB
Expiración: Indefinite
Order:     1
Status:    Habilitado
Creada:    25/5/2026 6:43
Creada por: Equipo 11
```

### Lógica de la regla

Esta regla está diseñada para suprimir incidentes generados por
pruebas conocidas del lab — específicamente los tests EICAR que
se ejecutaron en la VM `maquinasc-200` durante el onboarding del
Día 1. Esos tests generaron incidentes reales (ID 26974 y 26975)
que son falsos positivos conocidos.

En un ambiente de producción, la condición estaría filtrada
específicamente por el nombre de la Analytics Rule que genera
los EICAR. En este workspace la condición quedó en "All" debido
a que `sentinel-lab-2` solo tiene una Analytics Rule disponible
("Advanced Multistage Attack Detection") y no existe una regla
específica de EICAR para filtrar.

### Flujo de ejecución

```
maquinasc-200 ejecuta test EICAR
        ↓
MDE detecta → genera alerta
        ↓
Analytics Rule de Sentinel crea incidente
        ↓
Automation Rule evalúa: ¿el nombre de la regla contiene el criterio?
        ↓  sí
Acción 1: Status → Closed (con tag FALSE-POSITIVE-LAB)
Acción 2: Tag → FALSE-POSITIVE-LAB
        ↓
Incidente llega a la cola ya cerrado — el analista no lo ve
```

### Resultado verificado

```
Automation rules:  1  ← antes: 0
Enabled rules:     1  ← antes: 0
```

La regla aparece en la lista con status "Habilitado" y las acciones
"Change status, ..." visibles en la columna Actions.

---

## 7. Diferencia práctica: Automation Rule vs Analytics Rule

Para el SC-200 esta distinción es fundamental:

| Característica | Analytics Rule | Automation Rule |
|---|---|---|
| Qué hace | Detecta y genera incidentes | Procesa incidentes existentes |
| Usa KQL | Sí | No |
| Se ejecuta | Según schedule o en tiempo real | Cuando ocurre el trigger |
| Puede cerrar incidentes | No | Sí |
| Puede disparar Playbooks | No | Sí |
| Quién la crea | El analista | El analista |
| Viene predefinida | Sí (plantillas) | No |

**La Analytics Rule hace la pregunta.
La Automation Rule toma la acción.**

---

## 8. Tipos de Automation Rules — Standard vs Enhanced

El portal de Defender XDR muestra dos tipos:

**Standard rules** (las que creamos hoy):
- Aplican a un solo workspace
- Condiciones y acciones básicas
- No soportan Playbooks generados por IA

**Enhanced rules** (New — en preview):
- Aplican a múltiples workspaces simultáneamente
- Triggers más avanzados
- Soportan AI-generated playbooks
- Diseñadas para tenants multi-workspace como este

Para el SC-200 actual, Standard rules son el tema principal.
Enhanced rules son una capacidad nueva que puede aparecer en
versiones futuras del examen.

---

## 9. Casos de uso de Automation Rules — resumen para SC-200

```
1. Supresión de falsos positivos conocidos
   → Change status → Closed + tag "FALSE-POSITIVE"

2. Escalamiento automático por severidad
   → IF severity = High → Assign owner → equipo-critico@empresa.com

3. Enriquecimiento de incidentes
   → Add tags según táctica MITRE detectada

4. Triaging automático
   → IF analytic rule = "Brute Force" → Change severity → High
                                      → Add task → "Revisar IP origen"

5. Disparar respuesta automática
   → Run playbook → "Revocar-Sesion-Entra-ID"
```

---

## Lecciones aprendidas

- Las Automation Rules son SOAR — actúan sobre incidentes existentes,
  no los detectan.
- Una Automation Rule puede encadenar múltiples acciones en secuencia.
- El trigger más común es "When incident is created" — es el que
  permite suprimir o clasificar incidentes antes de que lleguen
  a la cola del analista.
- Un bug en la condición (typo en el nombre de la regla) hace que
  la Automation Rule nunca se dispare — la regla existe pero es
  inefectiva. Validar las condiciones contra las reglas reales es
  parte del mantenimiento del SOAR.
- El portal de Defender XDR requiere contexto de workspace establecido
  para acceder a Automation — si la sesión se reinicia, navegar
  desde portal.azure.com → Sentinel es más confiable.

---

*SOC Analyst Journal — Ancla Lab | Marco | Día 11 de 20*
