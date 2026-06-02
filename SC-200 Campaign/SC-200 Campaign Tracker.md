# 🛡️ Operación Centinela — Defender AnclaBank
### SC-200 Campaign Tracker · Ancla Lab

> Campaña de estudio gamificada para el examen **SC-200: Microsoft Security Operations Analyst**.
> Eres el **SOC Lead de AnclaBank**. Un APT está escalando durante 17 días. Cada día es una misión.

---

## 📋 Briefing

| Campo | Valor |
|---|---|
| Examen | SC-200 (Microsoft Security Operations Analyst) |
| Fecha del examen | **18 de junio de 2026** |
| Score para aprobar | 700 / 1000 |
| Formato | ~40–60 preguntas · 100 min · case studies (5–7 preguntas, no se pueden revisitar) |
| Tenant | E5 — Defender XDR · Sentinel · Azure · Intune · Entra ID · Purview |
| **Rango actual** | `🥉 SOC Operator – Tier 1` |
| **XP total** | `0` |

### Dominios y peso (blueprint del 16-abr-2026)
- **Dominio 1 — Manage a security operations environment** → 40–45% *(casi mitad del examen)*
- **Dominio 2 — Respond to security incidents** → 35–40%
- **Dominio 3 — Perform threat hunting** → 20–25%

---

## 🎮 Sistema de XP y rangos

**Puntos por actividad diaria:** 🧠 Briefing = 10 · 🛠️ Operación de Campo = 25 · 🎯 Checkpoint = 15 → **50 XP/día** ·
**Boss Fights (simuladores):** +50 XP cada uno

| Rango | XP necesario |
|---|---|
| 🥉 SOC Operator – Tier 1 | 0 |
| 🥈 SOC Analyst – Tier 2 | 150 |
| 🥇 Threat Hunter – Tier 3 | 350 |
| 🎖️ SOC Lead | 600 |
| 👑 CISO de AnclaBank *(listo para el examen)* | 850 |

### Badges
- [ ] 🏗️ **Arquitecto del SOC** — completar Dominio 1 (Días 1–7)
- [ ] 🛡️ **Primera Respuesta** — completar Dominio 2 (Días 8–11)
- [ ] 🎯 **Cazador de Amenazas** — completar Dominio 3 (Días 12–14)
- [ ] 🐉 **Mata-dragones** — los 3 Boss Fights con score ≥ 80%
- [ ] 👑 **CISO** — campaña completa

---

## 🗺️ Mapa de la campaña

| Día | Misión | Dominio | Hecho |
|---|---|---|---|
| 1 | Tomando el mando del SOC | D1 | [x] |
| 2 | Automatización y respuesta (AIR + ASR) | D1 | [x] |
| 3 | Attack disruption automática + device groups | D1 | [ ] |
| 4 | Conectar Sentinel — roles y retención (tiers) | D1 | [ ] |
| 5 | Ingesta I — AMA, Windows Security Events, DCRs | D1 | [ ] |
| 6 | Ingesta II — WEF, Syslog/CEF, Azure activities + workbooks + SOC optimization | D1 | [ ] |
| 7 | Detecciones (analytics rules, MITRE, anomalías) + 🐉 **Boss Fight 1** | D1 | [ ] |
| 8 | Responder en Defender XDR I — Office 365, Purview, Defender for Cloud | D2 | [ ] |
| 9 | Responder en Defender XDR II — Cloud Apps, Entra ID, Defender for Identity, case mgmt | D2 | [ ] |
| 10 | Investigar con Copilot embebido + ataques multi-etapa | D2 | [ ] |
| 11 | MDE response (timeline, live response, evidence) + M365 (Purview, Graph logs) | D2 | [ ] |
| 12 | Hunting I — KQL avanzado + tablas + Advanced Hunting | D3 | [ ] |
| 13 | Hunting II — threat analytics + Sentinel Graph + blast radius | D3 | [ ] |
| 14 | Hunting III — Data lake KQL jobs, Summary rules, Notebooks + MCP Server | D3 | [ ] |
| 15 | 🐉🐉 **Boss Fight 2** (completo) + repaso de débiles | — | [ ] |
| 16 | 🐉🐉🐉 **Boss Fight 3** (completo) + repaso de débiles | — | [ ] |
| 17 | Repaso ligero + logística del examen + descansar | — | [ ] |
| 18 | 🏆 **EXAMEN** | — | [ ] |

---

## 📓 Registro diario (Día 1)

> Incidente 31 — Attack Disruption contuvo svc_test_655 (Contain user, automático); contención revertida después.
> La maquinaria que vive debajo del attack disruption es AIR (Automated Investigation & Response)

### Día 1 — Tomando el mando del SOC · `D1`
- [x] 🧠 Briefing
- [x] 🛠️ Operación de Campo
- [x] 🎯 Checkpoint — Score: `3/3`
- **XP ganado:** `150`
- **Queries / hallazgos:**
- **Dudas para repasar:** Onboardear máquina a MDE para generar incidentes

### Día 2 — Automatización y respuesta (AIR + ASR) · `D1`
- [x] 🧠 Briefing
- [x] 🛠️ Operación de Campo
- [x] 🎯 Checkpoint — Score: `3/3`
- **XP ganado:** `50`
- **Notas:** AIR investigó 1,877 entidades en incidente 31; los 3 device groups en Full = por eso Attack Disruption actuó solo; directiva ASR creada en Audit; aprendí ActionType y Audited.

### Día 3 — Attack disruption automática + device groups · `D1`
- [ ] 🧠 Briefing
- [ ] 🛠️ Operación de Campo
- [ ] 🎯 Checkpoint — Score: `__/__`
- **XP ganado:** `__`
- **Notas:**

### Día 4 — Conectar Sentinel: roles y retención · `D1`
- [ ] 🧠 Briefing
- [ ] 🛠️ Operación de Campo
- [ ] 🎯 Checkpoint — Score: `__/__`
- **XP ganado:** `__`
- **Notas:**

### Día 5 — Ingesta I: AMA + Windows Security Events + DCRs · `D1`
- [ ] 🧠 Briefing
- [ ] 🛠️ Operación de Campo
- [ ] 🎯 Checkpoint — Score: `__/__`
- **XP ganado:** `__`
- **Notas:**

### Día 6 — Ingesta II: WEF, Syslog/CEF, Azure activities + workbooks · `D1`
- [ ] 🧠 Briefing
- [ ] 🛠️ Operación de Campo
- [ ] 🎯 Checkpoint — Score: `__/__`
- **XP ganado:** `__`
- **Notas:**

### Día 7 — Detecciones + 🐉 Boss Fight 1 · `D1`
- [ ] 🧠 Briefing
- [ ] 🛠️ Operación de Campo
- [ ] 🎯 Checkpoint — Score: `__/__`
- [ ] 🐉 Boss Fight 1 — Score: `__%`
- **XP ganado:** `__`
- **Notas:**

### Día 8 — Responder en Defender XDR I · `D2`
- [ ] 🧠 Briefing 
- [ ] 🛠️ Operación de Campo 
- [ ] 🎯 Checkpoint `__/__`
- **XP:** `__` 
- **Notas:**

### Día 9 — Responder en Defender XDR II · `D2`
- [ ] 🧠 Briefing ·
- [ ] 🛠️ Operación de Campo
- [ ] 🎯 Checkpoint `__/__`
- **XP:** `__`
- **Notas:**

### Día 10 — Copilot embebido + ataques multi-etapa · `D2`
- [ ] 🧠 Briefing
- [ ] 🛠️ Operación de Campo
- [ ] 🎯 Checkpoint `__/__`
- **XP:** `__`
- **Notas:**

### Día 11 — MDE response + M365 activities · `D2`
- [ ] 🧠 Briefing
- [ ] 🛠️ Operación de Campo
- [ ] 🎯 Checkpoint `__/__`
- **XP:** `__`
- **Notas:**

### Día 12 — Hunting I: KQL avanzado + Advanced Hunting · `D3`
- [ ] 🧠 Briefing
- [ ] 🛠️ Operación de Campo
- [ ] 🎯 Checkpoint `__/__`
- **XP:** `__`
- **Notas:**

### Día 13 — Hunting II: threat analytics + Sentinel Graph · `D3`
- [ ] 🧠 Briefing
- [ ] 🛠️ Operación de Campo
- [ ] 🎯 Checkpoint `__/__`
- **XP:** `__`
- **Notas:**

### Día 14 — Hunting III: Data lake + Summary rules + Notebooks · `D3`
- [ ] 🧠 Briefing
- [ ] 🛠️ Operación de Campo
- [ ] 🎯 Checkpoint `__/__`
- **XP:** `__`
- **Notas:**

### Día 15 — 🐉🐉 Boss Fight 2 + repaso · `Review`
- [ ] 🐉 Boss Fight 2 — Score: `__%`
- [ ] Repaso de áreas débiles
- **Notas:**

### Día 16 — 🐉🐉🐉 Boss Fight 3 + repaso · `Review`
- [ ] 🐉 Boss Fight 3 — Score: `__%`
- [ ] Repaso de áreas débiles
- **Notas:**

### Día 17 — Repaso ligero + logística · `Review`
- [ ] Repaso final de la cheat-sheet
- [ ] Confirmar logística del examen (ID, hora, sistema)
- [ ] Descansar bien 
- **Notas:**

---

## 🐉 Registro de Boss Fights (simuladores)

| # | Fecha | Simulador | Score | Áreas débiles detectadas |
|---|---|---|---|---|
| 1 | | | `__%` | |
| 2 | | | `__%` | |
| 3 | | | `__%` | |

> Recurso clave: **Practice Assessment gratis de Microsoft Learn** (en la página oficial del SC-200).

---

## 🧱 Banco de áreas débiles

> Cada vez que falles algo, anótalo aquí. El Día 15–17 atacas SOLO esta lista.

- "Roles built-in Defender XDR" y "dirección de ago() en filtros de tiempo"
- [ ]
- [ ]
- [ ]

---

## ⚡ Cheat-sheet rápida (la vas llenando)

**Roles Defender XDR:** Security Reader · Security Operator · Security Admin → ¿cuál puede qué?

**KQL operadores core:** `where` · `project` · `summarize` · `join` · `parse` · `make_set` / `make_list` · `bin()` · `ago()`

**Ingesta — cuándo usar qué:**
- Windows Security Events vía AMA → ...
- WEF → ...
- Syslog / CEF vía AMA → ...
- Azure activities (Azure Policy + diagnostic settings) → ...

**Tiers de retención:** Analytics · Data lake · XDR → diferencia clave: ...

**Analytics rules:** Scheduled · NRT · Threat Intelligence · ML → cuándo cada una: ...

---

*Operación Centinela · Ancla Lab · Marco-Security · Camino al SOC* 🚀
