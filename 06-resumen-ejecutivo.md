# 06 — Resumen Ejecutivo (Gobernar)
**Función NIST CSF 2.0:** GOBERNAR (GV.OC, GV.RM, GV.SC)
**Organización:** Distribuidora Barinas Total C.A. | **Clasificación:** Uso interno — Gerencial

---

## Para: Gerencia General y Junta de Socios
## De: Área de Tecnología de la Información
## Fecha: Octubre 2026

---

## 1. Punto de partida: lo que ya ocurrió

Este no es un reporte preventivo hipotético. **Los ataques ya ocurrieron.** En los últimos 18 meses, Distribuidora Barinas Total sufrió cuatro incidentes de seguridad documentados:

| Fecha | Severidad | Qué pasó | Costo estimado |
|---|---|---|---|
| Marzo 2025 | 🟠 Medio | Credenciales de ventas robadas por phishing; 320 registros de clientes expuestos | Daño reputacional; horas de gestión |
| Agosto 2025 | 🔴 Crítico | Ransomware LockBit cifró el servidor de facturación por 72 horas | **$4,200 USD** en ventas perdidas |
| Febrero 2026 | 🔴 Crítico | Fraude bancario digital: transferencia no autorizada de Bs. 18,500 | **~$510 USD** (recuperación parcial en proceso) |
| Mayo 2026 | 🔴 Crítico | Hackeo al sitio web: datos de 1,400 clientes y catálogo de precios publicados en darkweb | Daño reputacional; riesgo legal LOPCU |

> **Total de pérdidas directas estimadas 2025–2026: > $4,700 USD + daño reputacional no cuantificado.**

---

## 2. ¿Por qué seguimos siendo vulnerables?

Cada incidente ocurrió porque **las vulnerabilidades del anterior no fueron corregidas**:

```
Phishing (mar. 2025) → credenciales robadas
         ↓ no se implementó MFA
Ransomware (ago. 2025) → 72h sin operar
         ↓ no se cambió el sistema operativo del servidor
Fraude bancario (feb. 2026) → dinero perdido
         ↓ no se activó MFA en banca ni se revisaron PCs
Defacement + fuga (may. 2026) → clientes expuestos
         ↓ sin controles → próximo incidente en < 6 meses
```

**Conclusión directa:** sin acción correctiva, la probabilidad de un quinto incidente antes de marzo 2027 es **alta**.

---

## 3. Lo que ya hicimos (sin costo adicional)

| Acción | Qué protege |
|---|---|
| Activamos firewall en el servidor con reglas de acceso mínimo | Limita qué equipos pueden conectarse al servidor de facturación |
| Bloqueamos acceso desde la red WiFi de invitados al servidor | Visitantes y técnicos externos no pueden ver el sistema interno |
| Configuramos alertas automáticas de escritura masiva de archivos | Si un ransomware comienza a cifrar, lo detectamos en < 1 minuto |
| Actualizamos WordPress y eliminamos archivos de clientes del hosting | Cerramos el vector del incidente de mayo 2026 |

---

## 4. Tres decisiones urgentes que necesitan aprobación de la Gerencia

### Decisión A — Migrar el servidor de facturación a un sistema operativo con soporte
**En qué consiste:** El servidor que corre Profit Plus usa Windows Server 2012 R2, un sistema que Microsoft dejó de actualizar en octubre de 2023. Tiene más de 3 años sin recibir parches de seguridad. Es la vulnerabilidad que facilitó el ransomware de agosto 2025 y sigue activa hoy.

**La solución:** migrar el servidor a Ubuntu Server 22.04 LTS (gratuito y con soporte hasta 2032) con las aplicaciones necesarias.

**Inversión estimada:** $0 en licencias (Ubuntu es gratuito). Costo de implementación: 2–3 días de trabajo del técnico de TI. Si se requiere soporte externo: $150–$300 USD una sola vez.

**Riesgo de no hacerlo:** el servidor actual es un objetivo conocido para ransomware. La probabilidad de un nuevo ataque es alta.

**Plazo recomendado:** 60 días.

---

### Decisión B — Implementar MFA en todos los sistemas y activar respaldo 3-2-1
**En qué consiste:** (1) Doble verificación de identidad al acceder a Gmail, banca online y WordPress — además de contraseña, se necesita un código del teléfono. (2) Respaldo de datos en tres copias: una en el servidor local (NAS), una en disco externo guardado en la oficina de gerencia, y una copia cifrada en la nube.

**Por qué es urgente:** el fraude bancario de febrero 2026 y el hackeo de WordPress de mayo 2026 se hubieran evitado con MFA. El ransomware de agosto 2025 se hubiera recuperado en horas (no 72h) con un respaldo 3-2-1 correcto.

**Inversión estimada:**
- MFA en Gmail y WordPress: sin costo (apps gratuitas como Aegis/Google Authenticator)
- MFA en banca online: activación gratuita a través del banco
- NAS de respaldo (Synology DS223+): ~$180–$220 USD
- Servicio de nube cifrada (Backblaze B2): ~$10–$15 USD/mes

**Plazo recomendado:** MFA en 7 días; NAS y nube en 30 días.

---

### Decisión C — Capacitación mensual del personal en seguridad digital
**En qué consiste:** Un taller de 30 minutos al mes para los 7 vendedores y el resto del personal, enfocado en reconocer correos de phishing, no abrir adjuntos desconocidos y reportar comportamientos sospechosos.

**Por qué es urgente:** el incidente de marzo 2025 (phishing) y el de agosto 2025 (ransomware que entró por un adjunto en ventas) tienen el mismo origen: un empleado que no reconoció el engaño. Esta capacitación es la defensa más económica y más efectiva que existe.

**Inversión estimada:** tiempo del Coordinador de TI (2 horas de preparación mensual). Costo monetario: $0.

**Plazo recomendado:** primera sesión en 15 días.

---

## 5. Marco legal: lo que nos puede pasar si no actuamos

| Ley | Obligación | Consecuencia de incumplimiento |
|---|---|---|
| **LOPCU** | Proteger datos de los 1,400 clientes registrados | Reclamos civiles de clientes afectados; sanciones |
| **LECDI 2001** | Proteger sistemas contra acceso indebido y sabotaje | Responsabilidad de reportar; imagen ante autoridades |
| **CRBV Art. 60** | Respetar la privacidad de los datos personales | Fundamento de cualquier reclamación legal de los clientes |
| **Código de Comercio** | Conservar integridad de registros contables | Nulidad de registros; problemas con declaraciones ante el SENIAT |

> Los datos de nuestros clientes ya fueron publicados en la darkweb en mayo 2026. Si algún cliente demuestra daño derivado de esa filtración, la empresa puede enfrentar una acción civil. Corregir estas vulnerabilidades ahora es también una medida de protección legal.

---

## 6. Tabla de decisiones pendientes

| Decisión | Inversión | Urgencia | Aprobar antes de |
|---|---|---|---|
| A — Migrar servidor Windows 2012 → Ubuntu 22.04 | $0–$300 USD (una vez) | 🔴 Alta | 30 nov. 2026 |
| B — MFA en todos los sistemas | $0 | 🔴 Urgente | 15 oct. 2026 |
| B — NAS de respaldo 3-2-1 | ~$200 USD + $15/mes | 🔴 Alta | 31 oct. 2026 |
| C — Capacitación mensual personal | $0 | 🟠 Media | 20 oct. 2026 |

**Inversión total estimada: $200–$500 USD única vez + $15 USD/mes.**
**Costo de un quinto incidente crítico (basado en historial): > $4,000 USD + daño reputacional.**

---

## 7. Mensaje final

Distribuidora Barinas Total ha perdido más de $4,700 USD en 18 meses por incidentes de seguridad que compartían causas comunes: contraseñas sin doble factor, sistemas sin actualizar, respaldos insuficientes y personal sin capacitación. Cada incidente fue más grave que el anterior porque las vulnerabilidades no se cerraron.

Las medidas propuestas en este reporte no son costosas — su inversión combinada es menor a los $4,200 USD perdidos en un solo fin de semana de agosto 2025. Lo que sí es costoso es seguir operando con las mismas condiciones que ya produjeron cuatro incidentes documentados.

**Solicitamos autorización de la Gerencia General para iniciar las acciones B (MFA) esta semana, y agendar una reunión de 30 minutos para revisar el cronograma completo.**

---

*Tecnología de la Información — Distribuidora Barinas Total C.A.*
*Octubre 2026 · Laboratorio educativo ficticio — RJRB.8*
