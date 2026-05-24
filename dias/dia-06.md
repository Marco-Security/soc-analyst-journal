# Día 6 — Threat & Vulnerability Management (TVM) a fondo

**Fecha:** 24/05/2026
**Herramientas:** Microsoft Defender XDR — Administración de vulnerabilidades
**Autor:** Marco — Ancla Lab

---

## Objetivo

Revisar el estado de vulnerabilidades del tenant completo usando TVM,
identificar las CVEs más críticas, analizar los dispositivos expuestos
y documentar el flujo completo de una Remediation Task.

---

## Estado general del tenant — 24/05/2026

Cambios respecto a la semana 1:

| Métrica | Semana 1 | Hoy | Cambio |
|---------|----------|-----|--------|
| Secure Score | 40% | 40.19% | +0.19% |
| Dispositivos en riesgo alto | 7 | 9 | +2 (anclalab-vm + otro) |
| Incidentes activos | 26,869 | 26,882 | +13 |
| Exposure Score | - | 54/100 | Primer registro |

anclalab-vm ya aparece en la lista de dispositivos en riesgo alto como
consecuencia de los incidentes generados en el lab del día anterior.

---

## TVM Dashboard

```
Exposure Score:                  54/100 (medio)
Puntos vulnerables en tenant:    713
Vulnerabilidades aprovechables:  12 (con exploit público)
Vulnerabilidades críticas:       6
Vulnerabilidades de día cero:    0
Sin actualización de seguridad:  1
```

**Top recomendaciones por impacto:**

| Recomendación | Impacto | Dispositivos |
|---------------|---------|-------------|
| Actualizar Microsoft Edge Chromium | -31.69 | 29 |
| Actualizar Windows 10 | -29.17 | 15 |
| Actualizar Microsoft Defender Antivirus | -16.57 | 10 |

Una sola actualización de Edge resolvería 204 CVEs y reduciría el
Exposure Score de 54 a aproximadamente 22.

---

## Las 6 CVEs críticas del tenant

| CVE | CVSS | Software | Estado |
|-----|------|----------|--------|
| CVE-2026-8580 | 9.6 | Microsoft Edge Chromium | Parche disponible |
| CVE-2026-8511 | 9.6 | Microsoft Edge Chromium | Parche disponible |
| CVE-2026-41096 | 9.8 | Microsoft Edge Chromium | Parche disponible |
| CVE-2026-33824 | 9.8 | Windows 10 | Parche disponible |
| CVE-2025-60724 | 9.8 | Windows 10 | Sin parche |
| CVE-2026-41089 | 9.8 | Microsoft Edge Chromium | Parche disponible |

Todas son accesibles desde internet — vector de ataque remoto sin
necesidad de acceso físico al dispositivo.

---

## Análisis detallado — CVE-2026-8580

| Campo | Valor |
|-------|-------|
| Tipo | Use After Free en Mojo component de Chrome |
| Impacto | Sandbox escape via página HTML crafteada |
| CVSS | 9.6 (Critical) |
| Exploit público | No |
| Kits de vulnerabilidades | No |
| Accesible desde internet | Sí |
| Publicada | 11 may 2026 |
| Dispositivos expuestos | 28 |

Un atacante remoto podría explotar esta vulnerabilidad enviando una
página HTML crafteada a un usuario de cualquiera de los 28 dispositivos
expuestos — sin necesidad de autenticación previa.

---

## Flujo de Remediation Task

TVM no solo detecta vulnerabilidades — permite crear tickets de
remediación que se envían directamente al equipo de IT:

```
TVM detecta CVE en dispositivos
        ↓
Genera recomendación con impacto calculado
        ↓
Analista hace clic en "Solicitar corrección"
        ↓
Formulario: tipo de corrección, fecha límite, prioridad, notas
        ↓
Se crea ticket en Intune (para dispositivos AAD joined)
        ↓
IT recibe el ticket y parchea los dispositivos
        ↓
TVM verifica la corrección y actualiza el Exposure Score
```

**Opciones del formulario de corrección:**
- Tipo: Actualización de software / Cambio de configuración / Atención requerida
- Herramienta: Vale en Intune (AAD joined) o seguimiento manual
- Fecha de vencimiento: definida por el analista según urgencia
- Prioridad: Alto / Medio / Bajo
- Notas: contexto para el equipo de IT

La Remediation Task no se creó para no afectar el tenant compartido
del curso, pero el flujo fue documentado completamente.

---

## Hallazgos

1. El Exposure Score de 54/100 lleva sin cambios 6 días — ninguna
   recomendación ha sido implementada en el tenant durante ese período.

2. Las 12 vulnerabilidades aprovechables son la prioridad máxima —
   tienen exploit público disponible, lo que facilita el ataque
   incluso a actores con capacidades técnicas bajas.

3. Actualizar Edge en 29 dispositivos resolvería 204 CVEs de una sola
   vez — es el quick win de mayor impacto disponible en el tenant.

4. CVE-2025-60724 (CVSS 9.8) no tiene parche disponible aún —
   es la única vulnerabilidad crítica sin solución publicada.

5. El flujo TVM → Remediation Task → Intune conecta el SOC con IT
   sin emails manuales — reduce el tiempo entre detección y remediación.

---

## Lecciones aprendidas

- TVM prioriza por impacto, no por CVSS. Una CVE con CVSS 9.6 que
  afecta a 29 dispositivos tiene más impacto que una CVSS 9.8 que
  afecta solo a 1.
- El Exposure Score es la métrica ejecutiva de TVM — equivalente al
  Secure Score pero enfocado en vulnerabilidades de endpoints.
- "Accesible desde internet" es el factor de riesgo más crítico en
  una CVE — multiplica la superficie de ataque exponencialmente.

---

*SOC Analyst Journal — Ancla Lab | Marco | Día 6 de 20*
