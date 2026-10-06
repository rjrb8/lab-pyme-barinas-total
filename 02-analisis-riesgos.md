# 02 — Análisis de Riesgo
**Función NIST CSF 2.0:** IDENTIFICAR (ID.RA)
**Organización:** Distribuidora Barinas Total C.A. | **Clasificación:** Uso interno

---

## Contexto

Basado en la lección *"Activo, amenaza, vulnerabilidad y riesgo"* del **Módulo 1** y su aplicación práctica en el **Módulo 2**, los tres riesgos analizados en este documento tienen un punto de partida diferente al de una evaluación teórica: **ya ocurrieron o están ocurriendo**. El historial de incidentes 2025–2026 de Distribuidora Barinas Total es la evidencia empírica que eleva la probabilidad de cada riesgo de "posible" a "probable" o "casi cierto".

> *"No puedes eliminar las amenazas. Pero sí puedes reducir las vulnerabilidades."* — Módulo 1

### Cadena lógica aplicada

```
AMENAZA  →  explota  →  VULNERABILIDAD  →  daña  →  ACTIVO
El RIESGO mide la probabilidad de que eso ocurra y el daño que causaría.
```

### Matriz de estimación cualitativa

| Probabilidad / Impacto | Bajo | Medio | Alto |
|---|---|---|---|
| **Baja** | Bajo | Bajo | Medio |
| **Media** | Bajo | Medio | **Alto** |
| **Alta** | Medio | **Alto** | **Crítico** |

---

## Marco legal venezolano aplicable

| Riesgo | Instrumento | Consecuencia para la empresa |
|---|---|---|
| Filtración de datos de clientes | LOPCU + CRBV Art. 60 | Responsabilidad civil ante los 1,400 clientes afectados |
| Fraude bancario digital | LECDI 2001, Art. 14 | Denuncia penal; proceso de recuperación bancaria incierto |
| Ransomware / sabotaje de sistemas | LECDI 2001, Art. 7 | Pena hasta 6 años para el atacante; pérdida operativa para la empresa |

---

## Riesgo 1 — Nuevo ataque de ransomware sobre el servidor de facturación

### Cadena de riesgo

| Elemento | Descripción |
|---|---|
| **Activo afectado** | Servidor Profit Plus (facturación + inventario) — activo de disponibilidad crítica |
| **Amenaza** | Ransomware (nueva variante) distribuido por phishing por correo electrónico o por explotación de vulnerabilidades del Windows Server 2012 R2 (sin soporte desde oct. 2023) |
| **Vulnerabilidad** | Windows Server 2012 R2 fuera de soporte (sin parches de seguridad en 3 años), red WiFi sin segmentación (empleados y visitas en la misma red), USB de respaldo conectado permanentemente (también cifrable), personal sin capacitación en phishing |

### Estimación

| Factor | Valor | Justificación |
|---|---|---|
| **Probabilidad** | **Alta** | El ransomware ya ocurrió en agosto 2025 (LockBit). El servidor sigue corriendo sobre el mismo sistema operativo sin soporte. El vector (phishing en PC de ventas) no ha sido cerrado. Probabilidad Alta porque las mismas condiciones persisten |
| **Impacto** | **Alto** | Paralización total de ventas, despacho y facturación. La vez anterior fueron 72h y $4,200 USD en pérdidas. Sin mejoras en el respaldo, el próximo evento podría ser irrecuperable |
| **Nivel de riesgo** | 🔴 **CRÍTICO** | Alta × Alto = Crítico |

### Tratamiento: MITIGAR

**Acciones prioritarias:**

1. **Migrar el servidor** de Windows Server 2012 R2 a Windows Server 2022 o Ubuntu Server 22.04 LTS — **elimina la vulnerabilidad de SO sin soporte**
2. **Implementar respaldo 3-2-1 con cifrado AES-256** y NAS aislado de la red de producción
3. **Segmentar la red WiFi:** VLAN separada para empleados, invitados e IoT
4. **Capacitación mensual en phishing** para los 7 vendedores (vector de entrada original)
5. **Instalar Wazuh (SIEM open source)** para detección temprana de comportamiento de cifrado masivo

**Responsable:** Coordinador de TI + Gerencia General
**Plazo:** Migración del servidor en 60 días; respaldo 3-2-1 en 30 días; capacitación en 15 días

---

## Riesgo 2 — Nuevo fraude por acceso no autorizado a banca en línea

### Cadena de riesgo

| Elemento | Descripción |
|---|---|
| **Activo afectado** | Cuenta bancaria empresarial + credenciales de banca online |
| **Amenaza** | Atacante que ya comprometió credenciales en 2025 (phishing) y las reutilizó en febrero 2026. Si las credenciales no se cambiaron completamente y se activó MFA, el riesgo persiste. Adicionalmente: keylogger instalado en PC compartida de contabilidad |
| **Vulnerabilidad** | Acceso a banca online desde PCs compartidas sin MFA, contraseñas probablemente reutilizadas del incidente anterior, 3 personas conocen las mismas credenciales (gerente, contador, asistente), sin alerta de transacciones en tiempo real configurada |

### Estimación

| Factor | Valor | Justificación |
|---|---|---|
| **Probabilidad** | **Alta** | El fraude ya ocurrió en febrero 2026 y fue detectado 48 horas después. Si el atacante instaló un keylogger o backdoor en la PC de contabilidad, puede tener acceso activo en este momento. La probabilidad es Alta porque el vector no está cerrado con certeza |
| **Impacto** | **Alto** | Una PYME con capital de trabajo limitado puede ver comprometida su operación mensual completa con una sola transferencia fraudulenta. En Venezuela, la recuperación de fondos por fraude bancario digital es lenta e incierta |
| **Nivel de riesgo** | 🔴 **CRÍTICO** | Alta × Alto = Crítico |

### Tratamiento: MITIGAR + TRANSFERIR (parcial)

**Acciones prioritarias:**

1. **Cambiar INMEDIATAMENTE** todas las credenciales bancarias desde un dispositivo limpio (no desde las PCs de oficina que pueden estar comprometidas)
2. **Activar MFA** en la banca online empresarial — contactar al banco para habilitar autenticación por token o SMS
3. **Configurar alertas bancarias** de transacciones en tiempo real (correo + SMS) para cualquier movimiento > Bs. 500
4. **Reinstalar el sistema operativo** de las PCs de contabilidad para eliminar posibles keyloggers o backdoors
5. **Política de acceso exclusivo:** solo el contador y el gerente conocen las credenciales; el asistente no
6. **PC dedicada para banca:** una sola PC sin acceso a correo ni navegación general, exclusiva para banca online

**Transferencia parcial:** Consultar con el banco si existe un seguro de fraude digital o protocolo de compensación para PYME.

**Responsable:** Gerente General + Contador + Coordinador de TI
**Plazo:** Cambio de credenciales y MFA — **inmediato (< 24 horas)**

---

## Riesgo 3 — Nueva filtración de datos de clientes por vulnerabilidad en WordPress

### Cadena de riesgo

| Elemento | Descripción |
|---|---|
| **Activo afectado** | Base de datos de clientes (1,400 registros) + catálogo de precios |
| **Amenaza** | Atacante que explota vulnerabilidades en WordPress desactualizado (v6.2) o en plugins sin parches (WooCommerce, Contact Form 7, Elementor) para obtener acceso al panel de administración y de allí a los archivos exportados del servidor |
| **Vulnerabilidad** | WordPress 6.2 con plugins desactualizados (múltiples CVEs conocidos), contraseña `admin123` en panel /wp-admin sin MFA, archivos de exportación de clientes (.xlsx) accesibles desde el hosting, credenciales del hosting probablemente comprometidas también |

### Estimación

| Factor | Valor | Justificación |
|---|---|---|
| **Probabilidad** | **Alta** | El defacement y la exfiltración ya ocurrieron en mayo 2026 con exactamente este vector. Si WordPress no fue actualizado y las credenciales del panel no fueron completamente renovadas, el atacante puede tener acceso activo o puede volver usando las mismas credenciales |
| **Impacto** | **Alto** | Los 1,400 clientes ya fueron expuestos una vez. Una segunda filtración con más datos (condiciones de crédito, historial de compras completo) puede derivar en reclamos legales bajo la LOPCU y pérdida definitiva de la confianza de la cartera de clientes. El catálogo de precios en manos de la competencia tiene impacto comercial permanente |
| **Nivel de riesgo** | 🔴 **CRÍTICO** | Alta × Alto = Crítico |

### Tratamiento: MITIGAR + EVITAR (parcial)

**Acciones prioritarias:**

1. **Actualizar WordPress** a la versión estable más reciente y todos sus plugins — **inmediato**
2. **Cambiar todas las credenciales** del panel de WordPress y del hosting desde un dispositivo limpio
3. **Activar MFA** en el panel de administración de WordPress (plugin: WP 2FA o Google Authenticator)
4. **Eliminar los archivos .xlsx de clientes** del servidor de hosting — no deben existir en el entorno web
5. **Mover el catálogo de precios** detrás de autenticación (área privada para distribuidores) o eliminarlo del sitio público
6. **Evitar (parcial):** evaluar si el sitio web necesita realmente almacenar datos de clientes, o si ese proceso puede hacerse solo internamente

**Responsable:** Coordinador de TI + freelance web (supervisado)
**Plazo:** Actualización de WordPress — **inmediato**; eliminación de archivos sensibles — **inmediato**

---

## Matriz consolidada de riesgos

| # | Riesgo | Amenaza | Vulnerabilidad | Probabilidad | Impacto | Nivel | Tratamiento | Incidente previo |
|---|---|---|---|---|---|---|---|---|
| R1 | Ransomware en servidor Profit Plus | Malware / phishing | WinServer 2012 R2 sin soporte; red plana | Alta | Alto | 🔴 Crítico | Mitigar | Ago. 2025 |
| R2 | Fraude bancario digital | Credenciales comprometidas / keylogger | Sin MFA; PC compartida; 3 conocen clave | Alta | Alto | 🔴 Crítico | Mitigar + Transferir | Feb. 2026 |
| R3 | Filtración de datos de clientes vía WordPress | Exploit web / credenciales débiles | WP desactualizado; sin MFA; archivos .xlsx expuestos | Alta | Alto | 🔴 Crítico | Mitigar + Evitar | May. 2026 |

> **Observación crítica:** Los tres riesgos tienen probabilidad **Alta** porque los tres ya se materializaron. No son hipotéticos — son recurrentes. Esto eleva la urgencia de todos los controles de mitigación al nivel de **emergencia operativa**.

---

## Patrón de escalada — Análisis del Loop de IA

```
Mar 2025 (MEDIO)    → Phishing en ventas → credenciales robadas
        ↓
Ago 2025 (CRÍTICO)  → Las mismas credenciales + red plana → ransomware
        ↓
Feb 2026 (CRÍTICO)  → Credenciales bancarias reutilizadas → fraude
        ↓
May 2026 (CRÍTICO)  → WordPress sin MFA → defacement + exfiltración clientes
        ↓
PRÓXIMO (CRÍTICO)   → Sin controles = nuevo incidente en < 6 meses
```

El patrón muestra que **cada incidente no resuelto se convierte en el vector del siguiente**. La respuesta reactiva sin controles preventivos garantiza la escalada.

---

*Documento generado como parte del laboratorio educativo. Datos ficticios.*
*Referencia: Módulo 1, Lección 2 — Activo, amenaza, vulnerabilidad y riesgo · Módulo 2, Lección 2*
