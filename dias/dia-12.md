# Día 12 — Sentinel: Analytics Rules (Scheduled)

**Fecha:** 25 de mayo de 2026
**Duración:** ~1.5 horas
**Área:** Microsoft Sentinel — Analytics Rules + KQL
**Plataforma:** portal.azure.com → Log Analytics → sentinellab24225127856

---

## Objetivo del día

Crear una Analytics Rule tipo Scheduled en Sentinel usando KQL para
detectar brute force — múltiples logins fallidos del mismo usuario
en una ventana de 5 minutos. Validar la consulta contra logs reales
del workspace.

---

## 1. Hallazgo de infraestructura — arquitectura real del tenant

Al intentar acceder a Analytics Rules desde el portal de Defender XDR
(`security.microsoft.com → Microsoft Sentinel → Análisis`), el portal
redirigió a la selección de workspace con el siguiente mensaje:

```
"No tiene ningún área de trabajo conectada.
Para continuar, conecte un área de trabajo."
```

La investigación reveló que el tenant tiene **dos workspaces de
Log Analytics** con estados distintos:

```
sentinellab24225127856   → Conectado (Primary) a Defender XDR
sentinel-lab-2           → Desconectado de Defender XDR
```

Ambos en el mismo grupo de recursos `sentinel-lab-2`, región Central US,
suscripción Azure subscription 1.

La sección de **Análisis fue migrada al portal de Defender XDR** — ya
no existe como vista independiente en Azure. Sin embargo, la cuenta
`equipo11` no tiene permisos suficientes para conectar el workspace
al portal unificado, lo que bloquea el acceso a la creación de
Analytics Rules desde la UI.

**Implicación operacional:** un SOC Analyst L1 con permisos de lectura
puede ver incidentes y alertas generados por reglas existentes, pero
no puede crear nuevas Analytics Rules. Esa capacidad requiere el rol
de **Microsoft Sentinel Contributor** como mínimo.

---

## 2. Validación de KQL — Log Analytics directo

Para validar la consulta KQL sin depender de la UI de Analytics Rules,
se accedió al editor de logs directamente:

```
portal.azure.com → Log Analytics Workspaces →
sentinellab24225127856 → Registros
```

### Consulta ejecutada — detección de brute force

```kql
SigninLogs
| where ResultType != 0
| summarize FailedAttempts = count() by UserPrincipalName, bin(TimeGenerated, 5m)
| where FailedAttempts >= 5
| order by FailedAttempts desc
```

### Resultado

```
Tiempo de ejecución: 1s 775ms
Resultado: "No se encontraron resultados"
Intervalo probado: 25 abr. 2026 → 26 may. 2026 (30 días)
```

### Diagnóstico

La tabla `SigninLogs` existe en el workspace pero está vacía.
Confirmado con la consulta de inventario de tablas:

```kql
search *
| summarize count() by $table
| order by count_ desc
```

```
Resultado: "No se encontraron resultados"
```

El workspace `sentinellab24225127856` no tiene ningún dato ingestado —
es un workspace nuevo sin Data Connectors configurados. Sin el conector
de **Microsoft Entra ID** (anteriormente Azure AD), los `SigninLogs`
no fluyen hacia Sentinel.

---

## 3. Anatomía de la consulta — explicación línea por línea

Aunque no se pudieron obtener resultados con datos reales, la consulta
es válida y ejecutable. Cada línea hace lo siguiente:

```kql
SigninLogs
```
Selecciona la tabla de logs de inicio de sesión de Entra ID. Cada
fila es un intento de login — exitoso o fallido.

```kql
| where ResultType != 0
```
Filtra solo los intentos fallidos. `ResultType = 0` es login exitoso;
cualquier otro valor es un error. Los códigos más comunes:
- `50126` → credenciales incorrectas
- `50053` → cuenta bloqueada
- `50057` → cuenta deshabilitada
- `70011` → scope inválido

```kql
| summarize FailedAttempts = count() by UserPrincipalName, bin(TimeGenerated, 5m)
```
Agrupa los intentos fallidos por usuario y por ventanas de 5 minutos.
`bin(TimeGenerated, 5m)` divide el tiempo en buckets de 5 minutos —
todos los eventos dentro del mismo bucket se cuentan juntos.
El resultado es: cuántos fallos tuvo cada usuario en cada ventana de
5 minutos.

```kql
| where FailedAttempts >= 5
```
Filtra solo los casos donde hubo 5 o más fallos en esa ventana. Este
es el threshold de brute force — menos de 5 puede ser un usuario
olvidando su contraseña; 5 o más en 5 minutos sugiere ataque.

```kql
| order by FailedAttempts desc
```
Ordena los resultados de mayor a menor para ver los casos más críticos
primero.

---

## 4. Configuración completa de la Analytics Rule

Aunque no se pudo crear desde la UI por permisos, esta es la
configuración que se aplicaría al crear la regla:

### Pestaña General
```
Nombre:      Ancla-Lab - Brute Force Login Entra ID
Descripción: Detecta múltiples intentos fallidos de login del mismo
             usuario en una ventana de 5 minutos (threshold: 5)
Táctica:     Credential Access
Técnica:     T1110 - Brute Force
Severidad:   Medium
Estado:      Habilitado
```

### Pestaña Lógica de regla
```kql
SigninLogs
| where ResultType != 0
| summarize FailedAttempts = count() by UserPrincipalName, bin(TimeGenerated, 5m)
| where FailedAttempts >= 5
| order by FailedAttempts desc
```

```
Asignación de entidades:
  Entidad: Account
  Identificador: UserPrincipalName → column: UserPrincipalName

Programación de consultas:
  Ejecutar consulta cada:   5 minutos
  Buscar datos de los últimos: 5 minutos

Umbral de alerta:
  Generar alerta cuando el número de resultados: Es mayor que 0
```

### Pestaña Configuración de incidentes
```
Agrupación de alertas: Habilitada
  Agrupar alertas en un incidente si: Entidades coinciden
  Agrupar alertas de los últimos: 5 minutos
```

### Pestaña Respuesta automatizada
```
Automation Rule: Ancla-Lab - Suprimir FP pruebas EICAR
(la que creamos el Día 11 — aunque no aplica a esta regla
específicamente, ilustra la vinculación posible)
```

---

## 5. Tipos de Analytics Rules — resumen para SC-200

| Tipo | Usa KQL | Frecuencia | Caso de uso |
|---|---|---|---|
| Scheduled | Sí | Configurable (5 min a 14 días) | Detección personalizada con lógica compleja |
| NRT (Near Real-Time) | Sí | ~1 minuto | Detección casi instantánea sin ventana configurable |
| Microsoft Security | No | Tiempo real | Convertir alertas de MDE/DfC/Entra en incidentes Sentinel |
| Anomaly | No (ML) | Diario | Detección de comportamiento anómalo con ML |
| Fusion | No (ML) | Tiempo real | Correlación multi-señal para ataques multi-etapa |
| Threat Intelligence | Sí | Cada 1 hora | Match de IOCs contra logs |

Para el SC-200, los tipos más importantes son **Scheduled**, **NRT**
y **Microsoft Security**.

---

## 6. Data Connectors necesarios para esta regla

La consulta usa `SigninLogs` — para que esta tabla tenga datos
en Sentinel, se requiere:

```
Data Connector: Microsoft Entra ID
Tablas que activa:
  - SigninLogs          ← logins interactivos
  - AADNonInteractiveUserSignInLogs  ← logins no interactivos
  - AADServicePrincipalSignInLogs    ← logins de service principals
  - AADManagedIdentitySignInLogs     ← logins de identidades administradas
  - AuditLogs           ← cambios en directorio
  - AADRiskyUsers       ← usuarios marcados como riesgo por Entra ID Protection

Prerrequisito: licencia Entra ID P1 o P2
```

Sin este conector activo, la Analytics Rule existiría en Sentinel
pero nunca generaría alertas — ejecutaría la consulta contra una
tabla vacía y siempre devolvería 0 resultados.

---

## 7. Hallazgo del día — Data Connectors como prerequisito

El descubrimiento más importante del Día 12 no es la regla KQL
en sí, sino la comprensión de la dependencia completa:

```
Data Connector activo
        ↓
Tabla con datos en Log Analytics
        ↓
Analytics Rule con KQL válido
        ↓
Alerta generada cuando KQL devuelve resultados
        ↓
Incidente creado en Sentinel
        ↓
Automation Rule lo procesa
```

Si cualquier eslabón de esta cadena está roto, la detección no
funciona. En este tenant, el eslabón roto es el Data Connector
de Entra ID — las reglas de Analytics pueden existir, pero sin
datos que consultar son inefectivas.

Esto conecta con el hallazgo del **Día 1**: el tenant tenía solo
1 Data Connector activo. Sin saber cuál era ese conector, ahora
sabemos que `SigninLogs` — una de las fuentes más críticas para
detección de identidad — no está fluyendo hacia Sentinel.

---

## 8. KQL adicional — variantes de la consulta

### Variante con IP de origen (enriquecida)

```kql
SigninLogs
| where ResultType != 0
| summarize
    FailedAttempts = count(),
    IPs = make_set(IPAddress),
    Locations = make_set(Location)
    by UserPrincipalName, bin(TimeGenerated, 5m)
| where FailedAttempts >= 5
| extend IPCount = array_length(IPs)
| order by FailedAttempts desc
```

Esta versión agrega la lista de IPs y ubicaciones desde donde
llegaron los intentos — útil para distinguir un usuario
olvidando su contraseña (1 IP) de un ataque real (múltiples IPs).

### Variante con múltiples IPs como señal de ataque distribuido

```kql
SigninLogs
| where ResultType != 0
| summarize
    FailedAttempts = count(),
    UniqueIPs = dcount(IPAddress)
    by UserPrincipalName, bin(TimeGenerated, 1h)
| where FailedAttempts >= 10 and UniqueIPs >= 3
| order by FailedAttempts desc
```

Detecta ataques de password spray — muchos intentos desde
múltiples IPs distintas hacia el mismo usuario en 1 hora.

---

## Lecciones aprendidas

- Una Analytics Rule tipo Scheduled requiere: KQL válido + tabla
  con datos + Data Connector activo. Sin los tres, la regla existe
  pero no detecta nada.
- `SigninLogs` requiere el Data Connector de Microsoft Entra ID —
  sin él, la tabla está vacía aunque Sentinel esté configurado.
- El rol mínimo para crear Analytics Rules es **Microsoft Sentinel
  Contributor** — un analista con solo permisos de lectura puede
  ver reglas existentes pero no crear nuevas.
- `bin(TimeGenerated, 5m)` es el operador KQL para agrupar eventos
  en ventanas de tiempo — fundamental para detección de ataques
  basados en frecuencia.
- `ResultType != 0` filtra logins fallidos en `SigninLogs` — el
  código 0 siempre indica éxito; cualquier otro valor es un error.
- La sección de Análisis de Sentinel fue migrada al portal de
  Defender XDR — ya no es accesible directamente desde Azure para
  workspaces conectados al portal unificado.

---

*SOC Analyst Journal — Ancla Lab | Marco | Día 12 de 20*
