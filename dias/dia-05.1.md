# Lab — Onboarding, simulación de ataque y análisis en MDE

**Fecha:** 22/05/2026
**Herramientas:** Microsoft Defender XDR, MDE, Azure Portal
**Autor:** Marco — Ancla Lab

---

## Objetivo

Crear una VM en Azure, onboardearla a MDE, ejecutar técnicas de ataque
controladas y analizar los incidentes generados desde el portal de
Defender XDR — replicando el flujo completo de un ejercicio de Purple Team.

---

## Infraestructura creada

| Campo | Valor |
|-------|-------|
| Nombre | anclalab-vm |
| Proveedor | Microsoft Azure (Azure for Students) |
| Región | Canada Central |
| SO | Windows Server 2025 Datacenter |
| Tamaño | Standard D2s v3 (2 vCPU, 8 GB RAM) |
| IP pública | 20.151.233.186 |
| IP privada | 10.0.0.4 |
| Auto-shutdown | 11:00 PM UTC-6 |
| Costo estimado | $0.20 USD/hora |

---

## Onboarding a MDE

Método utilizado: Script local (Windows Server 2019, 2022 y 2025)

Pasos:
1. Descargar paquete de incorporación desde `security.microsoft.com → Settings → Endpoints → Onboarding`
2. Transferir el archivo `.cmd` a la VM via RDP con unidades locales compartidas
3. Ejecutar como administrador dentro de la VM
4. Verificar aparición del dispositivo en `Activos → Dispositivos`

El dispositivo apareció en el portal aproximadamente 2 minutos después de
ejecutar el script. Estado inicial: sin riesgos conocidos, sensor Active.

---

## Técnicas ejecutadas

### 1. Descarga de archivo EICAR
```powershell
Invoke-WebRequest -Uri "https://secure.eicar.org/eicar.com.txt" -OutFile "C:\test-MDATP-test\eicar.txt"
```
EICAR es el archivo de prueba estándar de la industria para verificar
detección de antivirus. MDE lo detectó y bloqueó inmediatamente.

### 2. Ejecución de PowerShell sospechoso
```powershell
Get-Process | Out-File C:\test-MDATP-test\processes.txt
Invoke-WebRequest -Uri "http://127.0.0.1" -ErrorAction SilentlyContinue
```
MDE detectó el comportamiento de PowerShell como sospechoso — táctica
T1047 Windows Management Instrumentation.

### 3. Simulación de inyección de código
```cmd
cmd.exe /c "echo [DllImport("kernel32")] > %TEMP%\test.cs"
```
MDE registró este comando como Remote execution en el timeline del incidente.

### 4. Tarea programada con payload codificado
```powershell
schtasks /create /tn "WindowsUpdate" /tr "powershell.exe -enc SQBFAFgA" /sc onstart /ru System /f
```
Técnica T1053 (Scheduled Task) + T1027 (Obfuscated Files). La tarea fue
eliminada después de la prueba con `schtasks /delete /tn "WindowsUpdate" /f`.

### 5. Reconocimiento de red (sin detección)
```powershell
net view / nbtstat -n / net use / Test-NetConnection
1..254 | ForEach-Object { Test-Connection -ComputerName "10.0.0.$_" -Count 1 -Quiet }
```
Ninguno de estos comandos generó alerta — MDE no marca reconocimiento
genérico de red sin contexto malicioso adicional.

---

## Incidentes generados

### Incidente ID 26974 — 'EICAR_Test_File' detected on one endpoint

| Campo | Valor |
|-------|-------|
| Gravedad | Alto |
| Estado | Active |
| Primera actividad | 22 may 2026 17:23:56 |
| Última actividad | 22 may 2026 18:15:08 |
| Alertas agrupadas | 2 |
| Táctica MITRE | T1047 Windows Management Instrumentation |
| Origen de detección | Detección personalizada + Antivirus |

**Timeline del incidente:**
```
16:49:38  wininit.exe / services.exe    → inicio del sistema
17:23:02  MsSense.exe                   → sensor MDE inicializando
17:23:45  SenseIR.exe                   → IR del sensor activo
17:23:56  powershell.exe                → Alerta de Powershell (Alto)
17:27:05  powershell.exe                → segunda instancia detectada
17:30:46  powershell.exe executed a script
17:32:08  cmd.exe [DllImport kernel32]  → Remote execution detectado
17:32:30  powershell.exe interacted with eicar.txt → Malware bloqueado
```

### Incidente ID 26975 — 'EICAR_Test_File' malware was prevented
| Campo | Valor |
|-------|-------|
| Gravedad | Informativo |
| Estado | Remediated |
| Categoría | Malware |
| Origen | Antivirus |

---

## Hallazgos

1. MDE detectó y bloqueó EICAR en segundos — el antivirus actuó antes
   de que AIR tuviera que intervenir.

2. PowerShell con parámetros `-ExecutionPolicy AllSigned -NoProfile`
   genera alerta inmediata — MDE monitorea PowerShell de forma granular
   registrando cada instancia con su PID y argumentos.

3. `cmd.exe` ejecutando código con `DllImport` fue marcado como
   Remote execution — MDE reconoce patrones de inyección de código
   aunque el comando falle.

4. El reconocimiento de red genérico (ping sweep, net view, net use)
   no generó alertas — MDE requiere contexto malicioso adicional para
   marcar estas técnicas como sospechosas.

5. MDE correlacionó automáticamente las alertas de PowerShell y EICAR
   en un solo incidente porque ocurrieron en el mismo dispositivo
   en el mismo timeframe.

---

## Lecciones aprendidas

- MDE detecta comportamiento conocido malicioso con alta precisión.
  El reconocimiento genérico de red es invisible sin contexto adicional.
  En un ataque real, la detección se activa cuando las técnicas se
  encadenan — no de forma aislada.

- El timeline del incidente muestra exactamente qué proceso generó
  cada evento, con PID y argumentos. Esto es la base del análisis
  forense en MDE.

- Crear la propia infraestructura de lab y generar incidentes reales
  da una comprensión del comportamiento de MDE que no se obtiene
  leyendo documentación.

---

## Próximos pasos

- Apagar la VM cuando no esté en uso (auto-shutdown configurado a 11 PM)
- En el siguiente lab: ejecutar técnicas más avanzadas con Atomic Red Team
- Analizar cómo las mismas técnicas aparecen en Sentinel via el Data Connector

---

*SOC Analyst Journal — Ancla Lab | Marco | Lab VM Azure*
