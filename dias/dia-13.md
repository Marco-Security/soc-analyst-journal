# Día 13 — Sentinel: Analytics Rule avanzada

**Fecha:** 26 de mayo de 2026
**Duración:** ~2 horas
**Área:** Microsoft Sentinel — KQL avanzado + Threat Hunting de identidad
**Plataforma:** portal.azure.com → Log Analytics → sentinel-lab-2 → Registros

---

## Objetivo del día

Construir una Analytics Rule más compleja que detecte acceso anómalo
por país — usuarios que inician sesión exitosamente desde múltiples
países distintos. Validar la consulta contra logs reales y analizar
los resultados encontrados.

---

## 1. Workspace utilizado — corrección de arquitectura

A diferencia del Día 12 donde se usó `sentinellab24225127856` (vacío),
hoy se trabajó con `sentinel-lab-2` que sí tiene datos reales:

```
Tablas disponibles en sentinel-lab-2:
  MicrosoftGraphActivityLogs   2628 registros
  AuditLogs                     648 registros
  SigninLogs                    616 registros  ← usado hoy
  AlertInfo                     564 registros
  LAQueryLogs                   119 registros
  Usage                          63 registros
```

`SigninLogs` tiene 616 registros — suficiente para detección real.

---

## 2. Validación retroactiva — Día 12

Antes de la consulta principal, se ejecutó la consulta de brute force
del Día 12 en `sentinel-lab-2` con intervalo de 90 días. Resultado:

```
Usuario:         equipo12@labmxsc200.onmicrosoft.com
Ventana:         25/5/2026 22:15:00 UTC
FailedAttempts:  5
```

**`equipo12` tuvo exactamente 5 intentos fallidos en una ventana de
5 minutos el 25 de mayo.** Si la Analytics Rule del Día 12 hubiera
estado activa en este workspace, hubiera generado una alerta real.
Esto valida retroactivamente la lógica de detección.

---

## 3. Consulta 1 — detección de acceso desde múltiples países

### Query

```kql
SigninLogs
| where ResultType == 0
| extend Country = tostring(LocationDetails.countryOrRegion)
| where isnotempty(Country)
| summarize
    LoginCount = count(),
    Countries = make_set(Country),
    FirstSeen = min(TimeGenerated),
    LastSeen = max(TimeGenerated)
    by UserPrincipalName
| extend CountryCount = array_length(Countries)
| where CountryCount > 1
| order by CountryCount desc
```

### Explicación línea por línea

```kql
SigninLogs
```
Tabla de logs de inicio de sesión de Entra ID. Cada fila es un
intento de login — exitoso o fallido.

```kql
| where ResultType == 0
```
Filtra solo logins **exitosos**. A diferencia del brute force donde
buscábamos fallos, aquí nos interesan los accesos que sí funcionaron
— queremos saber desde dónde entró el usuario realmente.

```kql
| extend Country = tostring(LocationDetails.countryOrRegion)
```
`extend` crea una columna nueva sin modificar las existentes.
`LocationDetails` es un campo JSON dentro de `SigninLogs` con
geolocalización. `countryOrRegion` extrae el país; `tostring()`
lo convierte a texto para poder filtrar y agrupar por él.

```kql
| where isnotempty(Country)
```
Descarta filas sin país — VPNs, IPs privadas y algunos logins no
interactivos no tienen geolocalización disponible. Sin este filtro
contaminarían el análisis con valores vacíos.

```kql
| summarize
    LoginCount = count(),
    Countries = make_set(Country),
    FirstSeen = min(TimeGenerated),
    LastSeen = max(TimeGenerated)
    by UserPrincipalName
```
Colapsa todas las filas de cada usuario en una sola fila resumen.
`summarize` hace dos cosas: colapsa filas y crea columnas nuevas
con los resultados de los cálculos. Solo sobreviven al `summarize`
las columnas que defines explícitamente o las del `by`. El campo
`Country` individual desaparece y es reemplazado por `Countries`
(lista de países únicos via `make_set`).

```kql
| extend CountryCount = array_length(Countries)
```
Crea otra columna nueva: cuántos países distintos tiene ese usuario.
`array_length()` cuenta los elementos del array generado por
`make_set`.

```kql
| where CountryCount > 1
```
Filtra solo usuarios con más de un país. Un usuario con siempre
el mismo país no es anómalo; dos o más países merecen revisión.

```kql
| order by CountryCount desc
```
Ordena de mayor a menor — los casos más anómalos primero.

### Resultado

```
Tiempo de ejecución: 800ms
Resultados: 2 usuarios

equipo6@labmxsc200.onmicrosoft.com
  LoginCount:   60
  Countries:    ['US', 'MX']
  FirstSeen:    25/5/2026 16:15 UTC
  LastSeen:     26/5/2026 07:49 UTC
  CountryCount: 2

equipo19@labmxsc200.onmicrosoft.com
  LoginCount:   53
  Countries:    ['MX', 'US']
  FirstSeen:    25/5/2026 10:11 UTC
  LastSeen:     26/5/2026 06:34 UTC
  CountryCount: 2
```

Ambos usuarios iniciaron sesión exitosamente desde **México y Estados
Unidos** en el mismo período de 24 horas. Requieren investigación.

---

## 4. Consulta 2 — detalle cronológico (Impossible Travel)

Para determinar si los logins desde distintos países ocurrieron
simultáneamente o en momentos distintos:

### Query

```kql
SigninLogs
| where ResultType == 0
| where UserPrincipalName in ("equipo6@labmxsc200.onmicrosoft.com",
                               "equipo19@labmxsc200.onmicrosoft.com")
| extend Country = tostring(LocationDetails.countryOrRegion)
| project TimeGenerated, UserPrincipalName, Country, IPAddress
| order by UserPrincipalName, TimeGenerated asc
```

### Explicación línea por línea

```kql
| where UserPrincipalName in ("equipo6...", "equipo19...")
```
`in` filtra por múltiples valores — equivalente a dos `where` con
`or`. Solo procesa los dos usuarios identificados como anómalos.

```kql
| project TimeGenerated, UserPrincipalName, Country, IPAddress
```
`project` selecciona solo las columnas que queremos ver. No colapsa
filas (eso es `summarize`) — solo decide qué columnas mostrar y
descarta el resto. `SigninLogs` tiene decenas de columnas; pedimos
solo cuatro: cuándo, quién, desde qué país, desde qué IP.

```kql
| order by UserPrincipalName, TimeGenerated asc
```
Ordena por usuario y luego cronológicamente — agrupa todos los
logins de cada usuario en orden de tiempo para detectar alternancias
imposibles entre países.

### Resultado — equipo6 (13 de 113 registros visibles)

```
25/5/2026 16:15  US   79.127.147.91
25/5/2026 16:16  MX   201.141.109.36   ← 1 minuto después en MX
25/5/2026 16:19  US   79.127.147.91
25/5/2026 16:46  MX   201.141.109.36
25/5/2026 16:52  MX   201.141.109.36
25/5/2026 16:54  MX   201.141.109.36
25/5/2026 17:14  US   79.127.147.91
25/5/2026 17:18  US   79.127.147.91
25/5/2026 17:22  MX   201.141.109.36
25/5/2026 17:51  MX   201.141.109.36
25/5/2026 17:59  MX   201.141.109.36
```

---

## 5. Hallazgo crítico — Impossible Travel en equipo6

### Análisis forense

```
16:15 UTC → login desde US  (IP: 79.127.147.91)
16:16 UTC → login desde MX  (IP: 201.141.109.36)
Diferencia: 1 minuto
```

**Un minuto entre un login en Estados Unidos y uno en México es
físicamente imposible.** No existe vuelo de 1 minuto entre ambos
países. Esto confirma que **dos sesiones activas simultáneas** están
accediendo a la cuenta de `equipo6` desde dos ubicaciones distintas.

### Patrón identificado

```
IP 79.127.147.91  →  geolocaliza en US  →  sesión A
IP 201.141.109.36 →  geolocaliza en MX  →  sesión B
```

Las dos IPs alternan durante horas el 25 de mayo. No es un viaje
real — son dos actores distintos (o el mismo usuario con VPN activa)
accediendo a la misma cuenta de forma concurrente.

### Clasificación del hallazgo

```
Técnica MITRE:  T1078 — Valid Accounts
Táctica:        Initial Access / Persistence
Severidad:      Alta
Acción sugerida:
  1. Revocar todas las sesiones activas de equipo6 en Entra ID
  2. Forzar cambio de contraseña
  3. Verificar actividad en AuditLogs: cambios de configuración,
     creación de reglas de reenvío, acceso a datos sensibles
  4. Revisar si equipo19 tiene el mismo patrón
```

---

## 6. Concepto — Impossible Travel

Impossible Travel es una técnica de detección basada en geolocalización
y tiempo. La lógica es simple:

```
Si usuario X hace login desde Ciudad A en tiempo T1
Y hace login desde Ciudad B en tiempo T2
Y la distancia(A, B) / (T2 - T1) > velocidad_máxima_posible
→ al menos uno de los dos logins es anómalo
```

**Entra ID Protection** detecta esto automáticamente y genera una
señal de riesgo de usuario. Para que esa señal llegue a Sentinel
se requiere el Data Connector de Microsoft Entra ID con las tablas
`AADRiskyUsers` y `AADUserRiskEvents` habilitadas, y licencia
Entra ID P2.

En este caso lo detectamos manualmente con KQL — haciendo el trabajo
que Entra ID Protection haría automáticamente en un tenant con P2.

---

## 7. Configuración de la Analytics Rule avanzada

Configuración para convertir la consulta en una regla de Sentinel:

### Pestaña General
```
Nombre:      Ancla-Lab - Impossible Travel Entra ID
Descripción: Detecta usuarios con logins exitosos desde múltiples
             países en una ventana de 24 horas. Posible Impossible
             Travel o cuenta comprometida con sesiones concurrentes.
Táctica:     Initial Access
Técnica:     T1078 - Valid Accounts
Severidad:   High
Estado:      Habilitado
```

### Pestaña Lógica de regla

```kql
SigninLogs
| where ResultType == 0
| extend Country = tostring(LocationDetails.countryOrRegion)
| where isnotempty(Country)
| summarize
    LoginCount = count(),
    Countries = make_set(Country),
    IPs = make_set(IPAddress),
    FirstSeen = min(TimeGenerated),
    LastSeen = max(TimeGenerated)
    by UserPrincipalName
| extend CountryCount = array_length(Countries)
| where CountryCount > 1
```

```
Programación:
  Ejecutar cada:        1 hora
  Buscar datos de los:  Últimas 24 horas

Umbral:
  Generar alerta cuando resultados: Mayor que 0

Asignación de entidades:
  Account → UserPrincipalName
```

### Diferencia con la regla de brute force

| Característica | Brute Force | Impossible Travel |
|---|---|---|
| Tipo de evento | Logins fallidos | Logins exitosos |
| Ventana | 5 minutos | 24 horas |
| Frecuencia | Cada 5 min | Cada 1 hora |
| Severidad | Medium | High |
| Acción inmediata | Bloquear IP | Revocar sesiones |

---

## 8. Operadores KQL nuevos usados hoy

| Operador | Qué hace | Ejemplo |
|---|---|---|
| `extend` | Crea columna nueva sin colapsar filas | `extend Country = tostring(...)` |
| `summarize` | Colapsa filas y crea columnas de agregación | `summarize count() by User` |
| `project` | Selecciona columnas, descarta el resto | `project Time, User, IP` |
| `make_set` | Crea array de valores únicos | `make_set(Country)` |
| `array_length` | Cuenta elementos de un array | `array_length(Countries)` |
| `isnotempty` | Filtra valores no vacíos | `where isnotempty(Country)` |
| `tostring` | Convierte a texto | `tostring(LocationDetails.x)` |
| `in` | Filtra por lista de valores | `where User in ("a", "b")` |

**Diferencia clave — `summarize` vs `project`:**
- `summarize` cambia cuántas filas hay (colapsa)
- `project` cambia cuántas columnas ves (selecciona)
- Se pueden usar juntos: `summarize` primero colapsa, `project`
  después selecciona columnas del resultado

---

## 9. Lecciones aprendidas

- `LocationDetails.countryOrRegion` es el campo KQL para extraer
  el país de un login en `SigninLogs` — requiere `tostring()` porque
  es un campo JSON anidado.
- `make_set()` dentro de `summarize` crea un array de valores únicos
  — ideal para acumular todos los países de un usuario sin duplicados.
- `array_length()` permite filtrar por el tamaño de ese array —
  la combinación `make_set + array_length` es el patrón estándar
  para detectar "más de N valores distintos".
- Impossible Travel se detecta comparando el tiempo entre logins
  desde países distintos — si la diferencia es menor que el tiempo
  mínimo de viaje, la cuenta está comprometida o hay sesiones
  concurrentes.
- `project` no es `summarize` — `project` solo selecciona columnas,
  no toca las filas. `summarize` colapsa filas y crea columnas nuevas.
- Un hallazgo real encontrado hoy: `equipo6` tiene Impossible Travel
  confirmado — logins desde US y MX con 1 minuto de diferencia,
  dos IPs distintas alternando durante horas.

---

*SOC Analyst Journal — Ancla Lab | Marco | Día 13 de 20*
