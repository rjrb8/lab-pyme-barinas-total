# 03 — Endurecimiento (Hardening)
**Función NIST CSF 2.0:** PROTEGER (PR.PS, PR.AC, PR.IP)
**Organización:** Distribuidora Barinas Total C.A. | **Clasificación:** Uso interno — Técnico

---

## Contexto

Basado en los **Módulos 1, 2 y 4** del curso, el endurecimiento es la aplicación práctica de controles para reducir las vulnerabilidades que permitieron los tres incidentes críticos del período 2025–2026. Cada control ataca directamente un vector documentado.

> *"Un control es cualquier medida que aplicas para proteger un activo: una regla, un dispositivo o una configuración."* — Módulo 1

### Principio guía: defensa en profundidad

Ningún control único es suficiente. Los tres controles implementados forman capas complementarias:

| Control | Capa | Incidente que previene |
|---|---|---|
| C1 — Firewall UFW + segmentación de red | Red | R1 ransomware (propagación lateral) |
| C2 — MFA + política de contraseñas | Acceso e identidad | R2 fraude bancario + R3 WordPress |
| C3 — WordPress hardening | Aplicación web | R3 filtración de datos de clientes |

---

## Control 1 — Firewall de host (UFW) en el servidor Windows migrado a Ubuntu

### Objetivo
Aislar el servidor de facturación de la red general de la distribuidora. Aplicar el principio de **mínimo privilegio de red**: solo los equipos y usuarios autorizados pueden comunicarse con el servidor.

### Base teórica
> *"Un firewall permite o bloquea comunicaciones según un conjunto de reglas: por dirección IP, por puerto o por protocolo."* — Módulo 2

### Escenario de migración
El servidor Windows Server 2012 R2 (sin soporte) será migrado a **Ubuntu Server 22.04 LTS** con Profit Plus ejecutándose vía Wine o mediante una migración a un ERP alternativo. Mientras tanto, UFW se aplica como control inmediato.

```bash
# ============================================================
# CONTROL C1: Configuración UFW en servidor Ubuntu 22.04 LTS
# Servidor: 192.168.10.10 (segmento de servidores - nuevo)
# Red ventas/admin: 192.168.20.0/24
# Red TI: 192.168.10.0/24
# ============================================================

# 1. Instalar UFW
sudo apt update && sudo apt install ufw -y

# 2. Política por defecto: denegar todo lo entrante
sudo ufw default deny incoming
sudo ufw default allow outgoing

# 3. SSH solo desde la PC del técnico de TI (192.168.10.5)
sudo ufw allow from 192.168.10.5 to any port 22 proto tcp comment 'SSH solo TI'

# 4. Profit Plus (puerto 8080) solo desde la red de ventas y admin
sudo ufw allow from 192.168.20.0/24 to any port 8080 proto tcp comment 'Profit Plus ventas'

# 5. Samba (carpetas compartidas) solo desde red interna autorizada
sudo ufw allow from 192.168.20.0/24 to any port 445 proto tcp comment 'SMB interno'
sudo ufw allow from 192.168.20.0/24 to any port 139 proto tcp comment 'SMB interno'

# 6. Bloquear explícitamente acceso desde la red WiFi de invitados
sudo ufw deny from 192.168.50.0/24 to any comment 'Bloquear red invitados'

# 7. Activar UFW
sudo ufw enable

# 8. Verificar estado
sudo ufw status verbose
```

**Resultado esperado:**
```
Status: active
Default: deny (incoming), allow (outgoing)

To                    Action      From
--                    ------      ----
22/tcp                ALLOW IN    192.168.10.5
8080/tcp              ALLOW IN    192.168.20.0/24
445/tcp               ALLOW IN    192.168.20.0/24
139/tcp               ALLOW IN    192.168.20.0/24
Anywhere              DENY IN     192.168.50.0/24
```

### Segmentación WiFi en router TP-Link (configuración complementaria)

```
SSIDs recomendados:
├── "BT-Empleados"   → VLAN 20 (192.168.20.0/24) — con contraseña WPA3
├── "BT-Invitados"   → VLAN 50 (192.168.50.0/24) — solo Internet, aislada
└── "BT-TI"          → VLAN 10 (192.168.10.0/24) — solo técnico TI, oculto
```

### Verificación del control
```bash
# Desde una PC de la red de invitados (192.168.50.x), intentar conectar al servidor:
nc -zv 192.168.10.10 8080
# Resultado esperado: Connection refused / timeout

# Desde una PC de ventas (192.168.20.x):
nc -zv 192.168.10.10 8080
# Resultado esperado: Connection to 192.168.10.10 8080 port [tcp/*] succeeded!
```

### Impacto en los incidentes documentados
- **Agosto 2025 (ransomware):** la red plana permitió que el ransomware se propagara desde la PC de ventas hasta el servidor en segundos. Con este control, incluso si una PC se infecta, **no puede alcanzar el servidor** directamente.
- **Reducción de superficie de ataque:** el servidor solo es accesible en los puertos estrictamente necesarios, desde las IPs autorizadas.

---

## Control 2 — Política de contraseñas robusta y MFA

### Objetivo
Implementar el control administrativo y técnico que cierra el vector de credenciales comprometidas, responsable de los incidentes de marzo 2025, febrero 2026 y mayo 2026.

### Base teórica
> *"La jerarquía: política (qué y por qué) → estándar (qué específico y medible) → procedimiento (cómo, paso a paso)."* — Módulo 2

---

### Política (nivel: QUÉ y POR QUÉ)

> **Política de gestión de acceso — Distribuidora Barinas Total C.A.**
> *"Toda credencial de acceso a sistemas de la empresa (Profit Plus, correo, sitio web, banca online) es un activo de información que debe protegerse con la misma diligencia que el efectivo o el inventario. El compromiso de una sola contraseña puede paralizar las operaciones, vaciar la cuenta bancaria o exponer los datos de nuestros 1,400 clientes."*

---

### Estándar (nivel: ESPECÍFICO Y MEDIBLE)

| Requisito | Valor mínimo |
|---|---|
| Longitud mínima | **14 caracteres** (NIST SP 800-63B — mayor que el estándar anterior por el historial de incidentes) |
| Tipos de caracteres | Mayúscula + minúscula + número + carácter especial |
| Historial | No repetir las últimas 8 contraseñas |
| Cambio obligatorio | Solo ante sospecha de compromiso (no periódico) |
| MFA | Obligatorio: Profit Plus, banca online, correo, WordPress, hosting |
| Bloqueo por intentos | 5 intentos fallidos → bloqueo 30 minutos |
| Baja de cuenta | Desactivación en < 2 horas tras salida del empleado |
| Contraseñas compartidas | **Prohibido** — cada empleado tiene credenciales individuales |

---

### Procedimiento — Implementar MFA en Gmail (correo empresarial)

```
1. Acceder a myaccount.google.com con la cuenta de la empresa
2. Seguridad → Verificación en dos pasos → Comenzar
3. Seleccionar: "Aplicación de autenticación" (Google Authenticator o Aegis)
4. Escanear el código QR con el teléfono
5. Verificar con el código de 6 dígitos generado
6. Guardar los códigos de respaldo en lugar seguro (no en el mismo teléfono)
7. Repetir para TODOS los empleados con correo de la empresa
```

### Procedimiento — Implementar MFA en WordPress

```bash
# Instalar plugin WP 2FA en WordPress:
# Panel admin → Plugins → Añadir nuevo → buscar "WP 2FA" → Instalar → Activar

# Configurar política desde el plugin:
# WP 2FA → Políticas → Usuarios obligados: Todos los usuarios con rol Admin/Editor
# Método: TOTP (aplicación autenticadora)
# Gracia: 3 días para configurar antes de bloqueo

# Cambiar la URL del login (ocultar /wp-admin del público):
# Plugin: WPS Hide Login
# Nueva URL: /acceso-bt-2026 (solo conocida por los admins)
```

### Procedimiento — Gestión de contraseñas para el equipo

```
Para el Coordinador de TI:

1. Instalar Bitwarden (gestor de contraseñas open source) en el servidor:
   docker run -d --name bitwarden -p 8081:80 vaultwarden/server:latest

2. Crear organización en Bitwarden para Barinas Total
3. Invitar a los usuarios por función:
   - Gerente: acceso a banca, Profit Plus (admin), correo gerencia
   - Contador: acceso a banca (lectura), Profit Plus (contabilidad)
   - Vendedores: acceso a Profit Plus (ventas), correo individual

4. Generar contraseñas aleatorias de 16+ caracteres para cada sistema
5. Eliminar contraseñas del bloc de notas, Excel y cuadernos físicos
```

### Verificación del control
```bash
# Verificar que el login de WordPress requiere MFA:
# Abrir navegador en modo incógnito → ir a /acceso-bt-2026
# Ingresar usuario y contraseña correctos → debe solicitar código TOTP

# Verificar política de contraseñas en Ubuntu (si se migra el servidor):
sudo nano /etc/security/pwquality.conf
# Verificar:
# minlen = 14
# minclass = 4
# maxrepeat = 2
```

---

## Control 3 — Hardening de WordPress

### Objetivo
Cerrar el vector que permitió el defacement y la exfiltración de datos de clientes en mayo 2026, aplicando controles específicos sobre la instalación de WordPress.

### Base teórica
> *"Un puerto → un servicio → una vulnerabilidad potencial. Minimizar lo expuesto reduce la superficie de ataque."* — Módulo 3 (puertos y servicios)

### Comandos y configuraciones aplicadas

```bash
# ============================================================
# HARDENING WORDPRESS — ejecutar desde el servidor de hosting
# vía SSH o desde el panel de hosting (File Manager / Terminal)
# ============================================================

# 1. Actualizar WordPress core a la versión más reciente
wp core update --allow-root

# 2. Actualizar TODOS los plugins
wp plugin update --all --allow-root

# 3. Actualizar TODOS los temas
wp theme update --all --allow-root

# 4. Eliminar plugins y temas inactivos (superficie de ataque innecesaria)
wp plugin delete hello akismet --allow-root   # Ejemplos de plugins no usados
wp theme delete twentytwenty twentytwentyone --allow-root

# 5. Verificar versión actual y estado
wp core version --allow-root
wp plugin list --allow-root
```

```bash
# 6. Permisos correctos en archivos WordPress (CRÍTICO)
# Desde SSH en el servidor de hosting:
find /var/www/html/barinas-total/ -type f -exec chmod 644 {} \;
find /var/www/html/barinas-total/ -type d -exec chmod 755 {} \;
chmod 600 /var/www/html/barinas-total/wp-config.php

# 7. Eliminar archivos sensibles del servidor de hosting
# (Los .xlsx de clientes NO deben estar en el servidor web)
find /var/www/html/barinas-total/ -name "*.xlsx" -delete
find /var/www/html/barinas-total/ -name "*.csv" -delete
find /var/www/html/barinas-total/ -name "clientes*" -delete
echo "Archivos sensibles eliminados del hosting"
```

```apache
# 8. Agregar reglas de seguridad en .htaccess
# Archivo: /var/www/html/barinas-total/.htaccess

# Bloquear acceso directo a wp-config.php
<files wp-config.php>
order allow,deny
deny from all
</files>

# Bloquear acceso al directorio wp-includes
<IfModule mod_rewrite.c>
RewriteEngine On
RewriteBase /
RewriteRule ^wp-admin/includes/ - [F,L]
RewriteRule !^wp-includes/ - [S=3]
RewriteRule ^wp-includes/[^/]+\.php$ - [F,L]
RewriteRule ^wp-includes/js/tinymce/langs/.+\.php - [F,L]
RewriteRule ^wp-includes/theme-compat/ - [F,L]
</IfModule>

# Deshabilitar listado de directorios
Options -Indexes

# Bloquear ejecución de PHP en uploads (previene webshells)
<Directory "/var/www/html/barinas-total/wp-content/uploads">
<Files "*.php">
Order Allow,Deny
Deny from all
</Files>
</Directory>
```

```php
// 9. Agregar configuraciones de seguridad en wp-config.php
// Archivo: /var/www/html/barinas-total/wp-config.php

// Deshabilitar edición de archivos desde el panel de WordPress
define('DISALLOW_FILE_EDIT', true);

// Forzar HTTPS
define('FORCE_SSL_ADMIN', true);

// Limitar revisiones de posts (reduce tamaño de BD)
define('WP_POST_REVISIONS', 3);

// Deshabilitar el reporte de errores PHP en el sitio público
define('WP_DEBUG', false);
define('WP_DEBUG_DISPLAY', false);
```

```bash
# 10. Instalar plugin de seguridad Wordfence (open source, capa WAF)
wp plugin install wordfence --activate --allow-root

# Configurar Wordfence desde el panel:
# → Activar firewall en modo aprendizaje (24h) luego Extended Protection
# → Activar escaneo de malware programado: diario
# → Activar bloqueo de IPs con > 5 intentos de login fallidos
# → Configurar alertas por correo al coordinador de TI
```

### Verificación del control

```bash
# Verificar que wp-config.php no es accesible públicamente:
curl -I https://barinas-total.com.ve/wp-config.php
# Resultado esperado: 403 Forbidden

# Verificar que no hay archivos .xlsx en el servidor:
find /var/www/html/barinas-total/ -name "*.xlsx" 2>/dev/null
# Resultado esperado: sin salida (no hay archivos)

# Verificar que el directorio uploads no ejecuta PHP:
# Crear archivo de prueba y verificar que devuelve 403
echo "<?php echo 'test'; ?>" > /var/www/html/barinas-total/wp-content/uploads/test.php
curl https://barinas-total.com.ve/wp-content/uploads/test.php
# Resultado esperado: 403 Forbidden
rm /var/www/html/barinas-total/wp-content/uploads/test.php
```

---

## Resumen de controles implementados

| # | Control | Tipo | Riesgos que mitiga | Estado |
|---|---|---|---|---|
| C1 | UFW + segmentación de red WiFi | Preventivo – Técnico | R1 (ransomware propagación) | ✅ Implementado |
| C2 | MFA + política de contraseñas + gestor Bitwarden | Preventivo – Admin+Técnico | R1, R2, R3 | ✅ Implementado |
| C3 | WordPress hardening completo | Preventivo – Técnico | R3 (filtración clientes/precios) | ✅ Implementado |

---

## Modelo de defensa en profundidad para la PYME

```
Capa externa:    [Router TP-Link con reglas básicas + VLANs WiFi]
     ↓
Capa de red:     [UFW en servidor — solo puertos necesarios desde IPs autorizadas]
     ↓
Capa de acceso:  [MFA en todos los sistemas + contraseñas 14+ chars + Bitwarden]
     ↓
Capa de SO:      [Ubuntu Server 22.04 LTS con parches automáticos]
     ↓
Capa de app:     [WordPress actualizado + Wordfence WAF + .htaccess endurecido]
     ↓
Capa de datos:   [Archivos sensibles fuera del hosting + respaldo 3-2-1 cifrado]
     ↓
Capa humana:     [Capacitación mensual phishing para los 7 vendedores]
```

---

*Documento generado como parte del laboratorio educativo. Datos ficticios.*
*Referencia: Módulo 1 (Controles) · Módulo 2 (Técnicos y Administrativos) · Módulo 4 (Proyecto integrador)*
