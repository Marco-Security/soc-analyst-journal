# Día 9 — Defender for Cloud: Defender Plans

**Fecha:** 24 de mayo de 2026
**Duración:** ~45 minutos
**Área:** Microsoft Defender for Cloud — Planes de protección
**Plataforma:** portal.azure.com → Defender for Cloud → Configuración del entorno

---

## Objetivo del día

Identificar todos los Defender Plans disponibles para la suscripción
de Azure, documentar cuáles están activos vs desactivados, entender
qué protege cada uno, y evaluar el impacto de la cobertura actual.

---

## 1. Ruta de acceso — observación importante

Para llegar a la configuración de planes se requirió navegar a través
de la URL directa:

```
portal.azure.com/#view/Microsoft_Azure_Security/SecurityMenuBlade/~/EnvironmentSettings
```

La suscripción `Azure subscription 1` aparece en la lista pero no es
clickeable con la cuenta `equipo11`. El motivo: la cuenta tiene permisos
de lectura sobre el tenant pero no permisos de Owner/Contributor sobre
la suscripción de Azure. Esto es un escenario realista en un SOC — un
analista L1 puede ver la configuración pero no modificarla.

Para ver la cobertura de planes se usó el libro integrado:
`Configuración del entorno → Cobertura de planes de Defender`

---

## 2. Estructura de recursos del tenant

```
Tenant:  labmxsc200.onmicrosoft.com
Suscripción:  Azure subscription 1 (660ff11e-270e-48e0-bc04-50e32b649b1c)
Recursos totales en suscripción:  10

Workspaces de Log Analytics detectados:
  - machinelearntke4237897238
  - sentinel-lab
  - PruebasentinelG
  - sentinelam11116892598
  - amiworkspacefo7284808248
  - gerasent29207467611
  - SentinelE14
  - azuremIworkspa6448905034

Todos los workspaces: 0/2 planes activos
```

El tenant tiene múltiples workspaces de Log Analytics — esto refleja
que el lab fue configurado por varios estudiantes con suscripciones E5
independientes. Desde la perspectiva de Defender for Cloud, cada
workspace es un recurso separado que requiere cobertura.

---

## 3. Defender Plans disponibles — Microsoft Azure

El libro de cobertura muestra 14 categorías de planes para Azure:

| Plan | Tipo de recurso protegido | Estado |
|------|--------------------------|--------|
| Defender CSPM | Postura cloud completa (recomendaciones avanzadas) | Off |
| Defender for Servers | VMs y servidores en Azure, AWS, GCP | Off |
| Defender for App Service | Web apps en Azure App Service | Off |
| Defender for SQL (Azure) | Bases de datos SQL en Azure | Off |
| Defender for SQL Server on machines | SQL Server en VMs | Off |
| Defender for Open-source relational DBs | PostgreSQL, MySQL, MariaDB | Off |
| Defender for Cosmos DB | Bases de datos NoSQL Cosmos DB | Off |
| Defender for Storage | Cuentas de Azure Storage | Off |
| Defender for Containers | AKS, registros de contenedores, Kubernetes | Off |
| Defender for Key Vault | Azure Key Vault (secretos, claves, certificados) | Off |
| Defender for Resource Manager | Operaciones del plano de control de Azure | Off |
| Defender for DNS | Consultas DNS (marcado como en desuso) | Off |
| Defender for Kubernetes (deprecated) | Kubernetes legacy | Off |
| Defender for Container Registry | Registros de contenedores legacy | Off |

**Resultado:** 0 de 14 planes activos en la suscripción.

### Cobertura absoluta

```
"La consulta no devolvió ningún resultado"
```

Esto confirma que no existe ninguna suscripción en el tenant con
cobertura completa en ningún plan. El resultado es consistente con
lo observado el Día 8: Secure Score N/D, 0 recursos evaluados,
0 recomendaciones.

---

## 4. Qué protege cada plan — análisis por prioridad

No todos los planes tienen el mismo valor para este tenant. La
prioridad depende de qué recursos existen realmente en la suscripción.

### Prioridad Alta — recursos presentes en el tenant

**Defender for Servers**
Protege: VMs de Azure, servidores en AWS/GCP onboardeados
Relevancia: `maquinasc-200` (anclalab-vm) es una VM de Azure Server 2025.
Sin este plan, esa VM no tiene evaluación de postura cloud.
Capacidades que activa: Just-in-time VM access, evaluación de
vulnerabilidades integrada, monitoreo de integridad de archivos,
detección de amenazas en tiempo real en la VM.

**Defender CSPM**
Protege: Postura de seguridad global de todos los recursos
Relevancia: Es el plan que habilita el Secure Score avanzado,
attack paths, Cloud Security Explorer y recomendaciones de riesgo.
Sin CSPM habilitado, el Secure Score permanece en N/D incluso si
otros planes están activos. Hay una versión gratuita (básica) que
activa recomendaciones fundamentales.

**Defender for Storage**
Protege: Cuentas de Azure Storage (blobs, archivos, colas)
Relevancia: El tenant tiene workspaces de Log Analytics que
internamente usan storage accounts. Sin este plan, cualquier
acceso anómalo o exfiltración de datos desde esos storage accounts
no generaría alertas.

**Defender for Key Vault**
Protege: Acceso a secretos, claves criptográficas y certificados
Relevancia: Alta en cualquier tenant con aplicaciones. El robo
de secretos de Key Vault es un vector de ataque frecuente en
entornos Azure comprometidos.

### Prioridad Media — recursos posiblemente presentes

**Defender for Resource Manager**
Protege: Operaciones del plano de control de Azure (ARM)
Relevancia: Detecta actividad maliciosa en la capa de gestión —
por ejemplo, creación de nuevos recursos por un atacante que
comprometió credenciales de Azure. No requiere recursos
específicos: protege toda la suscripción.

**Defender for Containers**
Protege: AKS, registros de contenedores
Relevancia: Media — solo si el tenant usa contenedores activamente.
En un lab de SC-200, es posible que algunos workspaces usen
contenedores internamente.

### Prioridad Baja / No aplica

**Defender for SQL, Cosmos DB, App Service, Open-source DBs**
Relevancia: Baja para este tenant específico — no se han
identificado bases de datos o aplicaciones web desplegadas.
Sin embargo, en un entorno empresarial real serían alta prioridad.

**DNS, K8s, Container Registry (deprecated)**
Estos planes están marcados como en desuso — Microsoft los está
migrando a versiones más nuevas. No deben habilitarse en tenants
nuevos.

---

## 5. CSPM gratuito vs CSPM de pago — distinción importante para SC-200

Defender for Cloud ofrece dos niveles de CSPM:

```
CSPM Básico (gratuito)
├── Secure Score calculado
├── Recomendaciones fundamentales de seguridad
├── Controles básicos por recurso
└── Disponible para todas las suscripciones sin costo

CSPM Defender (de pago)
├── Todo lo del básico +
├── Attack path analysis (rutas de ataque)
├── Cloud Security Explorer (consultas de grafo)
├── Agentless scanning (sin instalar agente en VMs)
├── Governance rules y asignación de propietarios
└── Recomendaciones de seguridad avanzadas con IA
```

El tenant actualmente no tiene ni el básico habilitado efectivamente
— por eso el Secure Score es N/D en lugar de mostrar al menos
las recomendaciones fundamentales.

---

## 6. Impacto de habilitar Defender for Servers en maquinasc-200

Si se habilitara Defender for Servers Plan 1 (el más básico) sobre
la suscripción, el efecto inmediato sería:

```
Antes:
  maquinasc-200 → invisible para Defender for Cloud
  Secure Score → N/D
  Recomendaciones → 0

Después (estimado):
  maquinasc-200 → evaluada contra controles de seguridad
  Secure Score → calculable (probablemente bajo, dado el estado actual)
  Recomendaciones → aparecerían misconfiguraciones:
    - Puertos de administración expuestos (RDP/SSH)
    - Actualizaciones pendientes
    - Endpoint protection (si no hay AV activo)
    - Diagnósticos de boot sin configurar
    - Acceso sin Just-in-time
```

Esta VM probablemente tiene puertos abiertos para el lab — RDP para
conectarse remotamente. Eso generaría al menos una recomendación de
prioridad alta inmediatamente.

---

## 7. Conclusiones del día

**1. 0 de 14 planes activos = punto ciego total en capa cloud.**
El tenant tiene infraestructura en Azure (VM, workspaces, storage
implícito) pero ningún plan que la evalúe. Cualquier ataque que
afecte esos recursos desde la capa cloud pasaría sin detección.

**2. El orden correcto para habilitar planes en este tenant sería:**
```
1. CSPM básico (gratuito) → activa Secure Score y recomendaciones base
2. Defender for Servers   → cubre maquinasc-200 inmediatamente
3. Defender for Storage   → cubre los workspaces de Sentinel
4. Defender for Resource Manager → cubre toda la suscripción, bajo costo
5. Defender for Key Vault → si hay secretos en uso
```

**3. La limitación de permisos es un hallazgo de proceso.**
Un analista con permisos de lectura puede identificar que los planes
están desactivados pero no puede activarlos — requiere escalar al
equipo de Cloud/IT con un hallazgo documentado. El journal es
exactamente esa documentación.

**4. Los planes deprecated (DNS, K8s legacy) no deben habilitarse.**
Microsoft está migrando esas capacidades a Defender for Containers
y otras soluciones consolidadas.

---

## Lecciones aprendidas

- Defender for Cloud tiene dos capas: CSPM (postura) y CWP
  (protección de cargas de trabajo). CSPM es el que calcula el
  Secure Score; CWP son los planes por tipo de recurso.
- Sin CSPM habilitado, el Secure Score es N/D aunque otros planes
  estén activos.
- Defender for Servers es el plan más relevante en cualquier tenant
  que tenga VMs — es también el más caro, razón por la que suele
  estar desactivado en labs.
- Un SOC Analyst con permisos de lectura puede documentar gaps de
  cobertura y escalar — no puede activar planes directamente.
- Los planes deprecated no deben considerarse para activar en
  tenants nuevos.

---

*SOC Analyst Journal — Ancla Lab | Marco | Día 9 de 20*
