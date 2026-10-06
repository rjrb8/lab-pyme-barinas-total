# 05 — Respuesta a Incidentes y Continuidad
**Función NIST CSF 2.0:** RESPONDER (RS.RP, RS.CO) | RECUPERAR (RC.RP, RC.BC)
**Organización:** Distribuidora Barinas Total C.A. | **Clasificación:** Uso interno

---

## Contexto

Los tres incidentes críticos de 2025–2026 tuvieron en común un factor que amplificó el daño: **no existía un plan de respuesta**. Las decisiones se tomaron bajo pánico, sin roles definidos, sin pasos claros y sin saber si el respaldo era recuperable. El resultado fue:

- **Agosto 2025:** 72 horas de parálisis y $4,200 USD en ventas perdidas (respaldo de 3 semanas de antigüedad).
- **Febrero 2026:** Fraude detectado 48 horas después por el banco, no por la empresa.
- **Mayo 2026:** Datos de 1,400 clientes publicados en darkweb antes de que la empresa supiera que había sido atacada.

> *"Desarrollar y mantener un plan de respuesta a incidentes, así como entrenar al personal encargado de reaccionar."* — Módulo 4

Este documento define el plan de respuesta para el incidente más probable y dañino: **un nuevo ataque de ransomware**.

---

## Mini Plan de Respuesta a Incidentes — Ransomware en Profit Plus

### Roles y contactos de emergencia

| Rol | Persona | Contacto | Responsabilidad |
|---|---|---|---|
| **Líder de incidente** | Carlos Peñaloza (Gerente) | +58-273-XXX-XXXX | Tomar decisiones ejecutivas; comunicar a clientes si es necesario |
| **Responsable técnico** | Rafael Ojeda (TI) | +58-414-XXX-XXXX | Contención técnica, análisis, recuperación |
| **Coordinador operativo** | Ana Vargas (Ventas) | +58-416-XXX-XXXX | Activar procedimientos manuales; coordinar con equipos |
| **Responsable financiero** | Jorge Blanco (Contador) | +58-412-XXX-XXXX | Monitorear cuenta bancaria; contactar banco si hay fraude simultáneo |
| **Canal de emergencia** | Grupo WhatsApp "BT-Emergencia-TI" | — | Comunicación entre los 4 roles durante el incidente |

---

### FASE 0 — Preparación (antes del incidente)

| Tarea | Responsable | Frecuencia |
|---|---|---|
| Verificar que el respaldo 3-2-1 se ejecutó correctamente | Rafael Ojeda (TI) | Diaria (9:00 a.m.) |
| Probar restauración del respaldo en VM de prueba | Rafael Ojeda (TI) | Mensual |
| Revisar el log de alertas de seguridad | Rafael Ojeda (TI) | Diaria (9:00 a.m.) |
| Simular un incidente de ransomware (tabletop exercise) | Todos los roles | Semestral |
| Mantener impreso este plan en el cuarto de TI | Rafael Ojeda (TI) | Siempre actualizado |
| Mantener en sobre sellado: contraseña de recuperación del NAS + credenciales de hosting | Gerente General | Caja fuerte de la empresa |

---

### FASE 1 — Identificación

**¿Cómo nos damos cuenta?**

```
Señal A: El CDU-01 (script de detección) envía alerta por correo/WhatsApp
Señal B: Un empleado reporta que "sus archivos se ven raros" o tienen extensión desconocida
Señal C: El sistema de Profit Plus deja de responder para todos los usuarios simultáneamente
Señal D: Aparece una nota de rescate en el escritorio del servidor o en carpetas compartidas
```

**Acciones de identificación (primeros 5 minutos):**

```bash
# Verificar en el servidor si hay archivos con extensión de ransomware:
find /srv/profit-plus/ -name "*.locked" -o -name "*.encrypted" -o \
     -name "*.ransom" -o -name "README_DECRYPT*" 2>/dev/null | head -20

# Verificar procesos con alto consumo de CPU (posible cifrado activo):
ps aux --sort=-%cpu | head -10

# Verificar conexiones de red activas salientes (posible C2):
ss -tunp | grep ESTABLISHED | grep -v "192.168"

# Verificar modificaciones recientes en el directorio crítico:
find /srv/profit-plus/ -newer /tmp/marker_ayer -type f | wc -l
```

**Clasificación del incidente:**

| Nivel | Criterio | Respuesta |
|---|---|---|
| 🟡 **Medio** | 1 PC con archivos cifrados; servidor intacto | Aislar PC; contener; notificar TI |
| 🔴 **Alto** | Servidor con archivos cifrados; Profit Plus inaccesible | Activar plan completo; notificar Gerencia |
| 🔴 **Crítico** | Servidor + NAS comprometidos; sin respaldo accesible | Activar plan completo + contratar soporte externo urgente |

---

### FASE 2 — Contención (primeros 15 minutos)

```
PASO 1 — DESCONECTAR FÍSICAMENTE el servidor de la red:
         → Desenchufar el cable de red del servidor
         → Esto detiene la propagación del ransomware a otras PCs
         → El servidor queda inaccesible pero el cifrado se detiene en la red compartida
         ⚠ NO APAGAR EL SERVIDOR AÚN — puede destruir evidencia en RAM

PASO 2 — DESCONECTAR el NAS de la red:
         → El NAS contiene el respaldo más reciente; protegerlo es PRIORIDAD ABSOLUTA
         → Desconectar también físicamente el cable de red del NAS

PASO 3 — IDENTIFICAR la PC de origen:
         → El CDU-01 registró la IP de origen antes de la alerta
         → Desconectar físicamente esa PC de la red
         → NO reiniciar la PC — preservar evidencia

PASO 4 — NOTIFICAR al grupo de emergencia:
         → Mensaje al grupo "BT-Emergencia-TI" con: hora, qué se detectó, qué se hizo
         → Llamar directamente al Gerente General

PASO 5 — ACTIVAR operaciones manuales:
         → Ana Vargas (Ventas) activa el cuaderno de control manual
         → Despacho continúa con el último inventario impreso disponible
```

```bash
# Capturar evidencia antes de apagar (si el servidor sigue encendido):
# Listar los procesos activos sospechosos
ps aux > /tmp/procesos_activos_$(date +%Y%m%d_%H%M).txt

# Listar conexiones de red activas
ss -tunp > /tmp/conexiones_$(date +%Y%m%d_%H%M).txt

# Verificar si el atacante creó usuarios nuevos
cat /etc/passwd | tail -10

# Copiar los archivos de log a USB antes de apagar
cp /var/log/auth.log /media/usb-evidencia/
cp /var/log/audit/audit.log /media/usb-evidencia/
cp /var/log/barinas-total/alertas-seguridad.log /media/usb-evidencia/
```

---

### FASE 3 — Erradicación

```
1. IDENTIFICAR el vector de entrada:
   → Revisar logs del router: ¿hubo conexiones entrantes inusuales antes del incidente?
   → Revisar /var/log/auth.log: ¿hubo accesos SSH sospechosos?
   → Revisar correos del equipo de ventas: ¿alguien abrió un adjunto?
   → Si es la misma PC que en agosto 2025 → el vector de phishing sigue activo

2. FORMATEAR los sistemas comprometidos:
   → Servidor: reinstalar Ubuntu Server 22.04 LTS desde cero (imagen limpia)
   → PC de origen: reinstalar Windows 10 desde imagen limpia
   → NO intentar limpiar el malware — puede dejar backdoors

3. CERRAR el vector antes de reconectar:
   → Si entró por phishing → capacitación de emergencia a ese empleado
   → Si entró por vulnerabilidad del SO → verificar que el nuevo SO tiene parches al día

4. CAMBIAR TODAS LAS CONTRASEÑAS:
   → Profit Plus (admin y todos los usuarios)
   → Correos corporativos Gmail
   → Router y WiFi
   → Cuenta bancaria (desde dispositivo limpio)
   → WordPress y hosting
```

---

### FASE 4 — Recuperación

```bash
# PASO 1: Verificar integridad del respaldo 3-2-1
# El respaldo más reciente debería ser de máximo 24h antes del incidente

# Verificar checksum del respaldo cifrado:
sha256sum /mnt/nas-backup/profit-plus/backup-$(date -d yesterday '+%Y-%m-%d').tar.gz.enc
# Comparar con el valor registrado en /mnt/nas-backup/checksums.txt

# PASO 2: Restaurar en el servidor recién instalado
# Descifrar el respaldo:
openssl enc -d -aes-256-cbc -pbkdf2 \
  -pass file:/etc/backup/backup.key \
  -in /mnt/nas-backup/profit-plus/backup-2026-10-05.tar.gz.enc \
  -out /tmp/restore/backup-profit-plus.tar.gz

# Descomprimir:
tar -xzf /tmp/restore/backup-profit-plus.tar.gz -C /srv/profit-plus/

# PASO 3: Verificar que los datos son consistentes
# Contar registros en la base de datos restaurada:
# (Comparar con el número de facturas conocido antes del incidente)

# PASO 4: Reconexión gradual a la red
# 1. Conectar solo a la red de TI para pruebas internas
# 2. Verificar que Profit Plus funciona: abrir, crear una factura de prueba, eliminarla
# 3. Conectar a la red de ventas: que los vendedores puedan conectarse
# 4. Monitoreo intensivo durante 72h post-recuperación
```

---

### FASE 5 — Lecciones aprendidas

**Reunión post-incidente (dentro de 5 días):**

```
Preguntas obligatorias:
1. ¿Cuál fue el vector de entrada? ¿Era el mismo que en agosto 2025?
2. ¿Cuánto tiempo entre infección y detección? ¿El CDU-01 funcionó?
3. ¿El respaldo 3-2-1 era recuperable? ¿En cuánto tiempo?
4. ¿Los roles del plan estaban claros para todos?
5. ¿Se cumplió el RTO (< 4 horas)?
```

**Documento de lecciones aprendidas:** registrar en `post-incidente-YYYY-MM-DD.md` en el repositorio de GitHub.

---

## Esquema de Respaldo 3-2-1

### Diseño del esquema

```
           DATOS ORIGINALES
     [Servidor Ubuntu — /srv/profit-plus/]
     [WordPress — /var/www/html/]
          │
    ┌─────┴───────┐
    ▼             ▼
[COPIA 1]    [COPIA 2]              [COPIA 3]
  Local        Medio 2               Nube
NAS Synology  Disco USB 2TB         Backblaze B2
  DS223+       Seagate              (cifrado AES-256
 (RAID 1)      Cifrado              cliente)
  En cuarto    Guardado en          Miami / Brasil
  de TI        Oficina Gerencia
  AES-256      (distinto piso)

Medio 1 (NAS)  +  Medio 2 (USB)  +  1 fuera del sitio (nube)
```

### Política de respaldos

| Parámetro | Valor |
|---|---|
| **Frecuencia** | Diaria — respaldo nocturno automático a las 11:00 p.m. |
| **RPO objetivo** | < 24 horas |
| **RTO objetivo** | < 4 horas desde inicio de restauración |
| **Retención NAS** | 30 días de respaldos diarios |
| **Retención USB** | 1 respaldo semanal, rotación 4 semanas |
| **Retención nube** | 90 días |
| **Cifrado** | AES-256 en todos los medios |
| **Aislamiento del NAS** | Conectado a la red solo 11:00–11:45 p.m. vía regla UFW temporal |
| **Prueba de restauración** | Mensual — restaurar en VM de prueba y verificar |

### Script de respaldo automatizado

```bash
#!/bin/bash
# /opt/backup/backup-pyme.sh
# Respaldo automatizado — Distribuidora Barinas Total C.A.
# Cron: 0 23 * * * /opt/backup/backup-pyme.sh

DATE=$(date +%Y-%m-%d)
BACKUP_DIR="/mnt/nas-backup/profit-plus"
SOURCE_DIR="/srv/profit-plus"
WP_DIR="/var/www/html/barinas-total"
KEY_FILE="/etc/backup/backup.key"
LOG="/var/log/barinas-total/backup.log"
CHECKSUM_FILE="$BACKUP_DIR/checksums.txt"

echo "[$DATE $(date +%H:%M)] Iniciando respaldo..." >> "$LOG"

# 1. Respaldo de Profit Plus (datos y configuración)
tar czf - "$SOURCE_DIR" | \
  openssl enc -aes-256-cbc -pbkdf2 -pass file:"$KEY_FILE" \
  -out "$BACKUP_DIR/profit-plus-$DATE.tar.gz.enc"

# 2. Respaldo de WordPress
tar czf - "$WP_DIR" | \
  openssl enc -aes-256-cbc -pbkdf2 -pass file:"$KEY_FILE" \
  -out "$BACKUP_DIR/wordpress-$DATE.tar.gz.enc"

# 3. Registrar checksums
sha256sum "$BACKUP_DIR/profit-plus-$DATE.tar.gz.enc" >> "$CHECKSUM_FILE"
sha256sum "$BACKUP_DIR/wordpress-$DATE.tar.gz.enc" >> "$CHECKSUM_FILE"

# 4. Sincronizar con nube (Backblaze B2)
b2 sync --noProgress "$BACKUP_DIR/" "b2://barinas-total-backup/"

# 5. Eliminar respaldos locales > 30 días
find "$BACKUP_DIR" -name "*.enc" -mtime +30 -delete

echo "[$DATE $(date +%H:%M)] Respaldo completado." >> "$LOG"
```

---

## Plan de operaciones manuales (sistema caído)

```
Si el sistema de Profit Plus está inaccesible:

VENTAS:
→ Activar cuaderno de pedidos manual (uno por vendedor)
→ Verificar existencias físicamente en almacén antes de confirmar pedidos
→ No hacer promesas de entrega sin verificar inventario físico

DESPACHO:
→ Usar el último inventario impreso disponible (imprimirlo cada lunes)
→ Registrar salidas en planilla física "Control de despacho manual"

FACTURACIÓN:
→ Emitir facturas manuales prenumeradas con sello de la empresa
→ Registrar en cuaderno: fecha, cliente, RIF, monto, condición de pago
→ Digitalizar todo al recuperar el sistema (máximo 48h después)

COMUNICACIÓN CON CLIENTES:
→ Si la caída supera 4 horas, el Gerente General notifica a los
  principales clientes por WhatsApp que el sistema está en mantenimiento
→ No mencionar "ataque" o "hackeo" hasta tener información confirmada
```

---

*Documento generado como parte del laboratorio educativo. Datos ficticios.*
*Referencia: Módulo 3 (cifrado en reposo, respaldo 3-2-1) · Módulo 4 (respuesta a incidentes)*
