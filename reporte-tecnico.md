# Reporte Técnico de Seguridad — Distribuidora Barinas Total C.A.
**Tipo:** Informe técnico de evaluación de seguridad (simulado)
**Audiencia:** Coordinador de TI, técnicos externos contratados
**Metodología:** NIST CSF 2.0 | Blue Team
**Clasificación:** Confidencial — Solo equipo técnico

---

## 1. Información del entorno evaluado

| Componente | Detalle |
|---|---|
| **Servidor de facturación** | Windows Server 2012 R2 (EOL oct. 2023) — Profit Plus 2K19 |
| **Base de datos** | SQL Server Express (embebido en Profit Plus) |
| **Sistema objetivo de migración** | Ubuntu Server 22.04 LTS |
| **Sitio web** | WordPress 6.2 — hosting compartido — plugins desactualizados |
| **Red** | Router TP-Link Archer AX55 — switch no gestionado 16 puertos |
| **WiFi** | Red única (empleados + invitados + servidor) — sin VLANs |
| **Respaldo actual** | USB 1 TB conectado permanentemente al servidor — sin cifrado — sin rotación |
| **Antivirus** | Windows Defender — sin consola centralizada |
| **Firewall** | Solo el del router TP-Link (configuración por defecto) |
| **Correo** | Gmail gratuito (sin Workspace) — sin MFA en ninguna cuenta |
| **Banca online** | Acceso desde PCs compartidas de contabilidad — sin MFA |
| **SIEM / Logs centralizados** | Ninguno |
| **Usuarios activos** | 12 en Profit Plus, 7 con correo Gmail empresarial |

---

## 2. Historial de incidentes — Análisis técnico

### INC-2025-001 (Medio) — Phishing con compromiso de credenciales
**Fecha:** Marzo 2025
**Vector:** Correo phishing con enlace a página falsa de "actualización de Gmail"
**Técnica:** Credential harvesting (página de phishing clonada)
**Impacto:** Credenciales de Gmail de un vendedor comprometidas; acceso a 320 registros de clientes en carpeta compartida adjunta al correo
**Detección:** 11 días — por comportamiento anómalo de la cuenta (envíos desde IP extranjera)
**Causa raíz:** Sin MFA en Gmail; enlace malicioso abierto sin verificación
**Estado a oct. 2026:** ⚠️ Vector aún activo — sin MFA implementado en todas las cuentas

---

### INC-2025-002 (Crítico) — Ransomware LockBit en servidor Profit Plus
**Fecha:** Agosto 2025
**Vector:** Macro maliciosa en archivo .docx adjunto a correo en PC de ventas
**Técnica:** Dropper → descarga LockBit → propagación SMB lateral en red plana → cifrado del servidor
**CVE relacionado (estimado):** EternalBlue (MS17-010) posiblemente explotado dado que WS2012R2 sin parches tiene esta vulnerabilidad activa
**Impacto:** 100% de archivos de Profit Plus cifrados; parálisis 72h; pérdida $4,200 USD; recuperación parcial desde respaldo de 3 semanas
**Detección:** Inmediata (archivos inaccesibles) pero sin protocolo de respuesta
**Causa raíz técnica:**
  - Windows Server 2012 R2 sin parches (CVEs activos)
  - Red plana sin segmentación → propagación libre
  - USB de respaldo conectado permanentemente → también cifrado
  - Sin firewall de host en el servidor
**Estado a oct. 2026:** ⚠️ Causa raíz principal aún activa — mismo SO, misma red plana

---

### INC-2026-001 (Crítico) — Fraude bancario digital
**Fecha:** Febrero 2026
**Vector probable:** Reutilización de credenciales del INC-2025-001 + posible keylogger en PC de contabilidad
**Técnica:** Account takeover — acceso a banca online con credenciales válidas
**Indicadores de compromiso (IoCs):**
  - Login desde IP fuera de Venezuela (no registrado por el banco inicialmente)
  - Transferencia única en horario 2:47 a.m. (fuera del horario operativo)
  - Monto: Bs. 18,500 (≈ $510 USD tipo oficial)
**Detección:** 48 horas — por el banco, no por la empresa
**Causa raíz técnica:**
  - Sin MFA en banca online
  - PCs de contabilidad compartidas sin análisis de malware post-INC-2025-002
  - Sin alertas configuradas para transacciones nocturnas
**Estado a oct. 2026:** ⚠️ Sin MFA en banca; PCs de contabilidad no reinstaladas

---

### INC-2026-002 (Crítico) — Defacement y exfiltración de datos
**Fecha:** Mayo 2026
**Vector:** Credenciales del panel WordPress (`admin` / `admin123`) sin MFA
**Técnica:** Acceso directo a /wp-admin → instalación de webshell → exfiltración de archivos .xlsx del servidor → defacement del sitio
**Archivos exfiltrados:**
  - `clientes_barinas_total_2026.xlsx` (1,400 registros con nombre, RIF, teléfono, dirección, historial)
  - `catalogo_precios_distribuidores_jun2026.xlsx` (precios y márgenes comerciales)
**Destino confirmado:** Foro darkweb (confirmado por reporte de cliente)
**Detección:** Días después — por reporte de un cliente que encontró sus datos online
**Causa raíz técnica:**
  - Contraseña `admin123` — trivialmente adivinable
  - Sin MFA en WordPress
  - Archivos .xlsx con datos sensibles almacenados en el servidor de hosting (fuera del servidor interno)
  - WordPress 6.2 con plugins desactualizados (múltiples CVEs en WooCommerce y Elementor activos)
**Estado a oct. 2026:** WordPress actualizado, archivos eliminados — MFA aún pendiente de activar

---

## 3. Hallazgos técnicos activos (post-evaluación)

### HLG-001 — Windows Server 2012 R2 sin soporte de seguridad
- **Severidad:** 🔴 Crítica
- **CVSSv3 estimado:** 9.8 (superficie total de ataque sin parches)
- **CVEs relevantes activos:** MS17-010 (EternalBlue), CVE-2019-0708 (BlueKeep), CVE-2021-34527 (PrintNightmare)
- **Evidencia:**
```powershell
# Verificar versión y fecha de último parche (en servidor Windows):
Get-WmiObject Win32_OperatingSystem | Select-Object Caption, Version, LastBootUpTime
systeminfo | findstr /B /C:"OS Name" /C:"OS Version" /C:"Hotfix"
# Resultado esperado: Windows Server 2012 R2 — sin hotfixes recientes
```
- **Remediación:** Migrar a Ubuntu Server 22.04 LTS en 60 días

---

### HLG-002 — Red WiFi plana sin segmentación (flat network)
- **Severidad:** 🔴 Crítica
- **Descripción:** Empleados, invitados, servidor de facturación y equipos de contabilidad comparten la misma red 192.168.1.0/24. Movimiento lateral trivial para un atacante con acceso a cualquier dispositivo.
- **Evidencia:**
```bash
# Desde cualquier PC de la red actual:
nmap -sn 192.168.1.0/24
# Resultado: el servidor aparece junto con PCs de vendedores e invitados
ping 192.168.1.100    # → servidor Profit Plus — accesible desde cualquier punto
```
- **Remediación:** Configurar VLANs en el router TP-Link:
```
VLAN 10 — Servidores:    192.168.10.0/24  [Servidor Profit Plus, NAS]
VLAN 20 — Empleados:     192.168.20.0/24  [PCs ventas, contabilidad]
VLAN 30 — TI:            192.168.30.0/24  [PC del técnico TI]
VLAN 50 — Invitados:     192.168.50.0/24  [Aislada — solo Internet]
```

---

### HLG-003 — USB de respaldo conectado permanentemente
- **Severidad:** 🔴 Crítica
- **Descripción:** El USB de respaldo de 1 TB está conectado de forma permanente al servidor. En el ransomware de agosto 2025, el USB también fue cifrado, eliminando el respaldo. No hay cifrado, no hay rotación.
- **Remediación:** Implementar respaldo 3-2-1 (ver documento 05). El NAS debe estar aislado de la red excepto durante la ventana de respaldo:
```bash
# Regla UFW para aislar NAS (192.168.10.20):
# Bloquear acceso al NAS desde la red de empleados por defecto:
sudo ufw deny from 192.168.20.0/24 to 192.168.10.20

# Abrir acceso solo durante la ventana de respaldo (via cron):
# 55 22 * * * sudo ufw allow from 192.168.10.10 to 192.168.10.20  # Abrir
# 5  23 * * * sudo ufw deny from 192.168.10.10 to 192.168.10.20   # Cerrar
```

---

### HLG-004 — Sin MFA en ningún sistema crítico
- **Severidad:** 🔴 Crítica
- **Sistemas afectados:** Gmail (7 cuentas), banca online, WordPress, hosting
- **Descripción:** El 100% de los sistemas críticos solo requieren usuario y contraseña. Los tres incidentes críticos de 2026 y 2025 hubieran requerido un segundo factor para ser exitosos.
- **Remediación por sistema:**

```bash
# Gmail — activar desde myaccount.google.com/security
# → Verificación en 2 pasos → Aplicación de autenticación

# WordPress — instalar y configurar WP 2FA:
wp plugin install wp-2fa --activate --allow-root
# Forzar MFA para todos los admins desde el panel del plugin

# Hosting cPanel — activar desde:
# cPanel → Seguridad → Autenticación de dos factores

# Banca online — contactar directamente al banco para activar token
```

---

### HLG-005 — Archivos sensibles en servidor de hosting (corregido parcialmente)
- **Severidad:** 🟠 Alta (corregida en mayo 2026 post-incidente)
- **Descripción:** Se almacenaban archivos .xlsx con datos de 1,400 clientes y catálogo de precios en el servidor de hosting de WordPress. Estos fueron exfiltrados en INC-2026-002.
- **Estado actual:** Archivos eliminados post-incidente. Sin embargo, no se auditó si quedan archivos sensibles en otras rutas del hosting.
- **Verificación:**
```bash
# Auditoría completa de archivos sensibles en el hosting:
find /var/www/html/barinas-total/ \
  -name "*.xlsx" -o -name "*.csv" -o \
  -name "*cliente*" -o -name "*precio*" -o \
  -name "*distribuidor*" 2>/dev/null
# Resultado esperado: sin salida (ningún archivo sensible)
```

---

### HLG-006 — Sin logs centralizados ni SIEM
- **Severidad:** 🟠 Alta
- **Descripción:** Cada sistema genera logs de forma aislada. El INC-2026-001 (fraude bancario) fue detectado 48 horas después porque no había correlación de eventos. Un SIEM hubiera detectado el login bancario desde IP extranjera en tiempo real.
- **Remediación:**
```bash
# Instalación de Wazuh Manager (VM Ubuntu 22.04, 4GB RAM, 80GB disco):
curl -sO https://packages.wazuh.com/4.7/wazuh-install.sh
sudo bash ./wazuh-install.sh -a

# Agente en servidor Ubuntu (post-migración):
wget https://packages.wazuh.com/4.x/apt/pool/main/w/wazuh-agent/wazuh-agent_4.7.0-1_amd64.deb
sudo WAZUH_MANAGER='192.168.30.10' dpkg -i wazuh-agent_4.7.0-1_amd64.deb
sudo systemctl start wazuh-agent

# Agente en PCs Windows (descargar MSI desde packages.wazuh.com):
# Instalar con: WAZUH_MANAGER=192.168.30.10 msiexec /i wazuh-agent.msi /q
```

---

## 4. Controles implementados — Verificación técnica

### C1 — UFW activo en servidor Ubuntu (post-migración)
```bash
sudo ufw status verbose
# Expected output:
# Status: active
# Default: deny (incoming), allow (outgoing)
# 22/tcp     ALLOW IN    192.168.30.5      [SSH solo TI]
# 8080/tcp   ALLOW IN    192.168.20.0/24   [Profit Plus ventas]
# 445/tcp    ALLOW IN    192.168.20.0/24   [SMB interno]
# Anywhere   DENY IN     192.168.50.0/24   [Bloquear invitados]
```

### C2 — MFA verificado en WordPress
```bash
# Verificar que el plugin WP 2FA está activo:
wp plugin status wp-2fa --allow-root
# Expected: Plugin wp-2fa is active.

# Verificar que la URL de login fue modificada:
grep -r "wps-hide-login" /var/www/html/barinas-total/wp-config.php
# O verificar acceso a /wp-admin → debe redirigir a 404
curl -I https://barinas-total.com.ve/wp-admin
# Expected: HTTP/1.1 404 Not Found
```

### C3 — WordPress actualizado y hardening verificado
```bash
wp core version --allow-root
# Expected: 6.6.x (o la versión estable más reciente)

wp plugin list --update=available --allow-root
# Expected: lista vacía (todos actualizados)

# Verificar permisos de wp-config.php:
ls -la /var/www/html/barinas-total/wp-config.php
# Expected: -rw------- (600) - solo el propietario puede leer/escribir
```

---

## 5. Backlog técnico priorizado

| Prioridad | Tarea | Plazo | Responsable |
|---|---|---|---|
| 🔴 P0 | Activar MFA en Gmail (todas las cuentas) | 3 días | TI + todos los empleados |
| 🔴 P0 | Activar MFA en banca online (coordinación con banco) | 7 días | Gerente + Contador + TI |
| 🔴 P0 | Reinstalar PCs de contabilidad (posible keylogger activo) | 7 días | TI |
| 🔴 P1 | Implementar respaldo 3-2-1 con NAS + nube cifrada | 30 días | TI |
| 🔴 P1 | Migrar servidor Windows 2012 R2 → Ubuntu 22.04 | 60 días | TI (+ soporte externo opcional) |
| 🟠 P2 | Configurar VLANs en router TP-Link | 45 días | TI |
| 🟠 P2 | Instalar Wazuh en VM dedicada | 60 días | TI |
| 🟠 P2 | Capacitación phishing para los 7 vendedores | 15 días | TI |
| 🟡 P3 | Instalar CDU-01 (script de detección de cifrado masivo) | 30 días | TI |
| 🟡 P3 | Evaluar migración de Gmail gratuito → Google Workspace | 90 días | Gerencia + TI |

---

## 6. Referencias técnicas

- MS17-010 (EternalBlue) — https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-0144
- CVE-2019-0708 (BlueKeep) — https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-0708
- Wazuh Documentation — https://documentation.wazuh.com/
- WordPress Security Hardening — https://wordpress.org/documentation/article/hardening-wordpress/
- NIST SP 800-63B (contraseñas) — https://pages.nist.gov/800-63-3/sp800-63b.html
- CIS Controls v8.1 — https://www.cisecurity.org/controls/v8
- LECDI Venezuela 2001 — Gaceta Oficial N° 37.313

---

*Reporte técnico — Laboratorio educativo ficticio*
*Distribuidora Barinas Total C.A. — RJRB.8 — Octubre 2026*
