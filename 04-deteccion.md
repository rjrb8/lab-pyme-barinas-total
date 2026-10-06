# 04 — Detección
**Función NIST CSF 2.0:** DETECTAR (DE.AE, DE.CM)
**Organización:** Distribuidora Barinas Total C.A. | **Clasificación:** Uso interno — Técnico

---

## Contexto

El incidente de ransomware de agosto 2025 fue detectado **cuando el daño ya era total**: el servidor ya estaba cifrado, las operaciones ya estaban paralizadas. El fraude bancario de febrero 2026 fue detectado **48 horas después** por el banco, no por la empresa. La exfiltración de datos de mayo 2026 fue detectada **cuando los datos ya estaban publicados** en la darkweb.

**El patrón es claro:** Distribuidora Barinas Total no detecta — reacciona tarde.

> *"No puedes proteger lo que no ves. El monitoreo consiste en observar las comunicaciones y los eventos para reconocer, a tiempo, algo que no debería estar pasando."* — Módulo 4

---

## Línea base del comportamiento normal

Antes de detectar anomalías, se debe conocer qué es "normal":

| Sistema | Comportamiento normal | Señal de alerta |
|---|---|---|
| Servidor Profit Plus | Conexiones desde 192.168.20.0/24, horario 7:30–18:30 | Conexión fuera de horario o desde IP desconocida |
| WordPress admin | 1–2 logins por semana del administrador | 5+ intentos fallidos en < 5 minutos |
| Banca online | 3–8 transacciones por día, horario laboral | Transacción nocturna o desde IP fuera de Venezuela |
| Correo Gmail | 20–50 correos por día por usuario | Envío masivo repentino o login desde IP extranjera |
| Servidor: CPU | Uso promedio < 30% en horario laboral | CPU sostenido > 90% sin tarea programada (cifrado ransomware) |
| Transferencia de red | < 200 MB saliente por día | > 500 MB saliente en horario no laboral (exfiltración) |

---

## Caso de Uso de Detección — CDU-01: Comportamiento de cifrado masivo (pre-ransomware)

### Descripción del escenario

Un empleado de ventas recibe un correo de phishing con un adjunto Word que contiene una macro maliciosa. Al abrir el archivo, se ejecuta un dropper que descarga el ransomware. El malware comienza a cifrar archivos en la PC y luego intenta moverse lateralmente hacia el servidor de Profit Plus. Este caso de uso detecta ese comportamiento antes de que el servidor sea cifrado.

> *"Si conoces cómo se ve tu tráfico y comportamiento normal, lo anormal empieza a saltar a la vista."* — Módulo 4

---

### Lógica del caso de uso

```
SI ocurre X → ALERTAR porque podría ser Y
```

| Componente | Descripción |
|---|---|
| **Evento observado (X)** | El servidor Ubuntu registra > 500 operaciones de escritura/modificación de archivos en < 60 segundos, provenientes de una sola IP de la red interna (192.168.20.x) |
| **Alerta generada** | Notificación inmediata por correo al Coordinador de TI y mensaje al grupo de emergencia WhatsApp |
| **Porque podría ser (Y)** | Ransomware activo en una PC de ventas intentando cifrar los archivos del servidor compartido (SMB) — el mismo patrón del incidente de agosto 2025 |
| **Impacto si no se detecta** | Servidor de Profit Plus completamente cifrado; 72h+ de paralización como en agosto 2025 |

---

### Implementación — Auditd para detectar escritura masiva en el servidor

```bash
# ============================================================
# IMPLEMENTACIÓN CDU-01: Monitoreo de escritura masiva
# Servidor Ubuntu 22.04 LTS — Distribuidora Barinas Total
# ============================================================

# 1. Instalar auditd (sistema de auditoría de Linux)
sudo apt install auditd audispd-plugins -y

# 2. Configurar regla de auditoría para el directorio compartido de Profit Plus
sudo auditctl -w /srv/profit-plus/datos/ -p w -k escritura_masiva
# -w: watch (monitorear el directorio)
# -p w: solo eventos de escritura
# -k: etiqueta para identificar la regla

# 3. Hacer la regla permanente
sudo nano /etc/audit/rules.d/barinas-total.rules
# Agregar:
# -w /srv/profit-plus/datos/ -p w -k escritura_masiva
# -w /srv/compartido/ -p w -k escritura_masiva
# -a always,exit -F arch=b64 -S rename -S unlink -k eliminacion_archivos

# 4. Reiniciar auditd
sudo systemctl restart auditd
sudo systemctl enable auditd
```

```bash
# 5. Script de detección de escritura masiva (ejecutar como cron cada 60 segundos)
# Archivo: /opt/monitoreo/detectar-cifrado.sh

#!/bin/bash
THRESHOLD=500           # Número de escrituras que dispara la alerta
VENTANA=60             # Ventana de tiempo en segundos
LOG_AUDIT="/var/log/audit/audit.log"
ALERTA_CORREO="ti@barinas-total.com.ve"

# Contar escrituras en el último minuto
ESCRITURAS=$(ausearch -k escritura_masiva --start recent -i 2>/dev/null | \
             grep "type=SYSCALL" | wc -l)

if [ "$ESCRITURAS" -gt "$THRESHOLD" ]; then
    TIMESTAMP=$(date '+%Y-%m-%d %H:%M:%S')
    IP_ORIGEN=$(ausearch -k escritura_masiva --start recent -i 2>/dev/null | \
                grep "addr=" | awk -F'addr=' '{print $2}' | cut -d' ' -f1 | \
                sort | uniq -c | sort -rn | head -1)

    MENSAJE="[ALERTA CRITICA - $TIMESTAMP]
    POSIBLE RANSOMWARE DETECTADO
    Escrituras en servidor: $ESCRITURAS en los ultimos 60 segundos
    IP de origen probable: $IP_ORIGEN
    Accion inmediata: Verificar PC de origen y considerar desconectar de la red
    Referencia: CDU-01 - Comportamiento de cifrado masivo"

    # Enviar alerta por correo
    echo "$MENSAJE" | mail -s "[ALERTA] Posible ransomware - Barinas Total" "$ALERTA_CORREO"

    # Registrar en log local
    echo "$MENSAJE" >> /var/log/barinas-total/alertas-seguridad.log

    # Bloquear automáticamente la IP de origen en UFW (contención automática)
    IP_BLOQUEAR=$(echo "$IP_ORIGEN" | grep -oE '192\.168\.[0-9]+\.[0-9]+')
    if [ -n "$IP_BLOQUEAR" ]; then
        sudo ufw deny from "$IP_BLOQUEAR" to any comment "Auto-bloqueo CDU-01 $TIMESTAMP"
        echo "IP $IP_BLOQUEAR bloqueada automáticamente" >> /var/log/barinas-total/alertas-seguridad.log
    fi
fi
```

```bash
# 6. Configurar cron para ejecutar el script cada 60 segundos
sudo crontab -e
# Agregar:
# * * * * * /opt/monitoreo/detectar-cifrado.sh
# * * * * * sleep 30 && /opt/monitoreo/detectar-cifrado.sh
```

---

### Monitoreo complementario — Intentos de login fallidos en WordPress

```bash
# Script: /opt/monitoreo/detectar-bruteforce-wp.sh
# Detectar ataques de fuerza bruta al panel de WordPress
# (Vector del incidente de mayo 2026)

#!/bin/bash
LOG_APACHE="/var/log/apache2/access.log"  # O nginx
THRESHOLD_WP=5
VENTANA_MIN=5

# Contar intentos POST al login de WordPress en los últimos 5 minutos
INTENTOS=$(grep "POST /acceso-bt-2026" "$LOG_APACHE" | \
           awk -v d="$(date -d '5 minutes ago' '+%d/%b/%Y:%H:%M')" '$0 > d' | \
           awk '{print $1}' | sort | uniq -c | sort -rn | head -1 | awk '{print $1}')

IP_ATACANTE=$(grep "POST /acceso-bt-2026" "$LOG_APACHE" | \
              awk -v d="$(date -d '5 minutes ago' '+%d/%b/%Y:%H:%M')" '$0 > d' | \
              awk '{print $1}' | sort | uniq -c | sort -rn | head -1 | awk '{print $2}')

if [ "${INTENTOS:-0}" -gt "$THRESHOLD_WP" ]; then
    echo "[$(date)] ALERTA: $INTENTOS intentos de login en WordPress desde $IP_ATACANTE" \
        >> /var/log/barinas-total/alertas-seguridad.log
    # Bloquear IP en UFW
    sudo ufw deny from "$IP_ATACANTE" to any comment "Brute-force WordPress $(date +%Y%m%d)"
    echo "IP $IP_ATACANTE bloqueada por brute-force WordPress" \
        | mail -s "[ALERTA] Brute-force WordPress" ti@barinas-total.com.ve
fi
```

---

### Monitoreo de logs clave — Revisión diaria del Coordinador de TI

```bash
# Checklist de revisión diaria (5 minutos por la mañana):

# 1. Intentos de login fallidos en el servidor
sudo grep "Failed password\|Invalid user" /var/log/auth.log | \
  awk '{print $11}' | sort | uniq -c | sort -rn | head -10

# 2. Conexiones activas al servidor Profit Plus
sudo ss -tnp | grep :8080

# 3. Alertas generadas en las últimas 24 horas
cat /var/log/barinas-total/alertas-seguridad.log | grep "$(date -d yesterday '+%Y-%m-%d')"

# 4. Uso de CPU y disco (detectar cifrado en progreso)
top -bn1 | grep "Cpu(s)" | awk '{print "CPU uso: " $2 "%"}'
df -h /srv/profit-plus/

# 5. Archivos modificados en las últimas 24h en el directorio crítico
find /srv/profit-plus/ -newer /tmp/marker_ayer -name "*.dat" | head -20
# (crear marker: touch -d "yesterday" /tmp/marker_ayer)
```

---

### IoCs documentados basados en los incidentes de la empresa

| IoC | Descripción | Fuente | Acción |
|---|---|---|---|
| IP externa con > 5 intentos login WordPress | Fuerza bruta al panel admin | Log Apache/Nginx | Bloquear en UFW; alerta TI |
| > 500 escrituras/min en directorio compartido | Comportamiento ransomware | auditd | Bloquear IP origen; alertar; aislar PC |
| Transacción bancaria nocturna (9pm–6am) | Fraude post-compromiso de credenciales | Alerta bancaria SMS | Contactar banco inmediatamente |
| Archivo con extensión `.locked` o `.encrypted` | Ransomware activo cifrando | Monitoreo de directorio | Aislar servidor; activar plan de respuesta |
| Login Gmail desde IP fuera de Venezuela | Credenciales de correo comprometidas | Gmail: actividad reciente | Cambiar contraseña; revisar reenvíos |
| Proceso desconocido con > 80% CPU | Malware activo (cifrado o minería) | top / htop | Aislar equipo; análisis forense |

---

### Esquema de centralización de logs — Objetivo 60 días

```
[Servidor Ubuntu]  ────────────────┐
[Router TP-Link]   ────────────────┤──→ [Wazuh Manager VM] ──→ [Alertas correo/WhatsApp]
[WordPress]        ────────────────┤      Ubuntu 22.04
[PCs Windows]      ────────────────┘      4GB RAM, 80GB disco
(agentes Wazuh)                           (VM en el servidor o PC dedicada)
```

**Herramienta recomendada:** Wazuh (open source, gratuito)
**Beneficio:** correlacionar eventos de todas las fuentes — ver el ataque completo, no solo un fragmento

---

*Documento generado como parte del laboratorio educativo. Datos ficticios.*
*Referencia: Módulo 4 — Monitoreo de tráfico de red; Logs: fuentes, formatos y centralización*
