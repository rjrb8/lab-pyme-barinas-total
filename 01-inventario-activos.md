# 01 — Inventario de Activos
**Función NIST CSF 2.0:** IDENTIFICAR (ID.AM)
**Organización:** Distribuidora Barinas Total C.A. | **Clasificación:** Uso interno

---

## Contexto

El primer paso de todo programa de seguridad es saber **qué se protege**. Siguiendo el Módulo 1 del curso, un activo es cualquier recurso con valor para la organización que deba ser resguardado — tangible (un servidor) o intangible (la lista de clientes, los precios de distribución).

> *"La ciberseguridad se ocupa de que la información, los equipos y los servicios sigan siendo confiables."* — Módulo 1

El historial de incidentes de Distribuidora Barinas Total (2025–2026) demuestra que sus activos de información son objetivos reales: la lista de 1,400 clientes fue publicada en la darkweb, la cuenta bancaria fue vaciada parcialmente, y el sistema de facturación fue cifrado por ransomware. Este inventario mapea los 5 activos críticos que generaron o pudieron generar esos impactos.

---

## Tríada CIA aplicada al contexto comercial

| Propiedad | Pregunta clave | Se rompe cuando… | Ejemplo en la PYME |
|---|---|---|---|
| **Confidencialidad (C)** | ¿Quién puede verlo? | Dato accedido por no autorizado | Lista de clientes publicada en darkweb (mayo 2026) |
| **Integridad (I)** | ¿Es correcto y sin cambios? | Dato alterado sin permiso | Precio de un producto modificado en el sistema |
| **Disponibilidad (D)** | ¿Está cuando se necesita? | Sistema inaccesible | Facturación cifrada 72h por ransomware (ago. 2025) |

---

## Marco legal aplicable

Los datos de los 1,400 clientes registrados (nombre, RIF, dirección, historial de compras) están protegidos por:
- **Art. 60 CRBV:** derecho a la privacidad de datos personales.
- **LOPCU:** obligación del comerciante de proteger datos del consumidor registrado.
- **LECDI 2001, Art. 6–10:** acceso indebido, sabotaje y fraude informático.
- **Código de Comercio:** integridad de registros contables.

---

## Activo 1 — Sistema de facturación e inventario (Profit Plus)

| Campo | Detalle |
|---|---|
| **Tipo** | Software ERP + base de datos |
| **Descripción** | Profit Plus 2K19 corriendo sobre Windows Server 2012 R2 (sin soporte desde oct. 2023). Contiene facturas, cuentas por cobrar, inventario de ~850 SKUs, precios de venta y márgenes comerciales |
| **Propietario** | Gerencia General / Coordinador de TI |
| **Ubicación** | Servidor físico — cuarto de equipos, planta baja |
| **Usuarios** | 12 usuarios concurrentes (ventas, contabilidad, despacho) |
| **Propiedad CIA prioritaria** | **DISPONIBILIDAD** |
| **Justificación** | El sistema de facturación es el corazón operativo de la distribuidora. Sin él, no se puede vender, despachar ni cobrar. El ransomware de agosto 2025 lo demostró con 72 horas de paralización total y pérdida de $4,200 USD en ventas. Ningún proceso manual puede reemplazarlo durante más de pocas horas. La disponibilidad es prioritaria porque la interrupción tiene impacto económico inmediato y directo |
| **Vulnerabilidad detectada** | Windows Server 2012 R2 sin soporte → sin parches de seguridad desde oct. 2023 |
| **Clasificación** | 🔴 Crítico |

---

## Activo 2 — Base de datos de clientes (1,400 registros)

| Campo | Detalle |
|---|---|
| **Tipo** | Base de datos — activo intangible |
| **Descripción** | Tabla de clientes en la base de datos de Profit Plus y exportaciones en Excel almacenadas en carpetas compartidas de red. Contiene: nombre/razón social, RIF, dirección, teléfono, correo, historial de compras, condiciones de crédito (plazo, límite) |
| **Propietario** | Gerencia Comercial |
| **Ubicación** | Servidor Profit Plus + carpetas compartidas \\servidor\clientes\ |
| **Propiedad CIA prioritaria** | **CONFIDENCIALIDAD** |
| **Justificación** | La filtración de mayo 2026 lo confirmó: los datos de 1,400 clientes fueron publicados en un foro de la darkweb, dañando la confianza de los clientes y exponiendo a la empresa a responsabilidades legales bajo la LOPCU y el Art. 60 CRBV. Los datos incluyen condiciones de crédito y hábitos de compra — información que los competidores pueden usar para ofertar directamente a nuestra cartera |
| **Clasificación** | 🔴 Crítico |

---

## Activo 3 — Cuenta bancaria empresarial y credenciales de banca en línea

| Campo | Detalle |
|---|---|
| **Tipo** | Activo financiero + credenciales digitales |
| **Descripción** | Cuenta corriente empresarial con acceso de banca en línea utilizada para pagos a proveedores, nómina y cobros. Las credenciales son conocidas por 3 personas (gerente, contador, asistente). Se accede desde PCs compartidas de la oficina sin MFA |
| **Propietario** | Gerencia General / Contabilidad |
| **Propiedad CIA prioritaria** | **CONFIDENCIALIDAD** |
| **Justificación** | El incidente de febrero 2026 (transferencia fraudulenta de Bs. 18,500 ≈ $510 USD) fue consecuencia directa del compromiso de estas credenciales. En el entorno venezolano, una transferencia no autorizada puede ser muy difícil de revertir, y el saldo de una cuenta empresarial PYME puede representar semanas de capital operativo. La confidencialidad de las credenciales es el único control que diferencia al propietario legítimo de un atacante |
| **Vulnerabilidad detectada** | Sin MFA, credenciales probablemente reutilizadas desde incidente de 2025 |
| **Clasificación** | 🔴 Crítico |

---

## Activo 4 — Sitio web corporativo (WordPress)

| Campo | Detalle |
|---|---|
| **Tipo** | Aplicación web — presencia digital |
| **Descripción** | Sitio WordPress 6.2 con catálogo de productos y formulario de contacto. Alojado en hosting compartido. Panel de administración accesible en /wp-admin con usuario `admin` y sin MFA. Plugins desactualizados (WooCommerce, Contact Form 7, Elementor) |
| **Propietario** | Gerencia Comercial (gestión operativa: freelance externo) |
| **Propiedad CIA prioritaria** | **INTEGRIDAD** |
| **Justificación** | El defacement de mayo 2026 demostró que alterar el sitio web daña directamente la imagen de la empresa y sirvió como vector para exfiltrar el catálogo de precios. En una distribuidora, el catálogo de precios es información comercial sensible: si la competencia lo obtiene, puede ajustar sus precios por debajo. La integridad del sitio y del catálogo es la propiedad más crítica — un sitio caído es temporal; un catálogo de precios filtrado a la competencia tiene impacto permanente |
| **Vulnerabilidad detectada** | WordPress desactualizado, panel /wp-admin sin MFA, contraseña `admin123` |
| **Clasificación** | 🔴 Crítico |

---

## Activo 5 — Equipos de usuario final (PCs de ventas y contabilidad)

| Campo | Detalle |
|---|---|
| **Tipo** | Hardware — endpoints |
| **Descripción** | 12 PCs de escritorio (Windows 10 Home, algunas Windows 7) distribuidas en ventas (7), contabilidad (3) y gerencia (2). Sin consola de antivirus centralizada. USB externo del servidor accesible desde cualquier PC. Red WiFi sin segmentación (empleados y visitas en la misma red) |
| **Propietario** | Coordinador de TI |
| **Propiedad CIA prioritaria** | **DISPONIBILIDAD** |
| **Justificación** | El vector de entrada del ransomware de agosto 2025 fue un archivo adjunto abierto en una PC de ventas. Dado que todas las PCs comparten la misma red sin segmentación y tienen acceso al servidor, una sola PC comprometida puede propagar ransomware a toda la organización en minutos. La disponibilidad es prioritaria porque estas PCs son la interfaz de trabajo del 100% del equipo comercial; si caen, la empresa para |
| **Vulnerabilidad detectada** | PCs con Windows 7 (sin soporte), red sin segmentación, USB del servidor accesible |
| **Clasificación** | 🟠 Alto |

---

## Matriz resumen de activos

| # | Activo | Tipo | Propiedad CIA | Clasificación | Incidente relacionado |
|---|---|---|---|---|---|
| 1 | Sistema Profit Plus (facturación/inventario) | Software + BD | **Disponibilidad** | 🔴 Crítico | Ransomware ago. 2025 |
| 2 | Base de datos de clientes (1,400) | Intangible | **Confidencialidad** | 🔴 Crítico | Defacement/exfiltración may. 2026 |
| 3 | Cuenta bancaria + credenciales banca online | Financiero | **Confidencialidad** | 🔴 Crítico | Fraude bancario feb. 2026 |
| 4 | Sitio web WordPress + catálogo precios | Aplicación web | **Integridad** | 🔴 Crítico | Defacement may. 2026 |
| 5 | PCs de ventas y contabilidad | Hardware | **Disponibilidad** | 🟠 Alto | Vector ransomware ago. 2025 |

---

## Lección aplicada — Módulo 1

> *"Una cooperativa perdió la lista de socios por una USB. Un corte de energía paralizó la plataforma."*

En Distribuidora Barinas Total:
- **Confidencialidad rota:** lista de clientes exportada a un Excel en carpeta compartida → freelance externo accedió por WordPress comprometido → publicada en darkweb.
- **Disponibilidad rota:** PC de ventas con archivo adjunto de phishing → ransomware propagado por red plana → servidor de Profit Plus cifrado 72h.
- **Integridad amenazada:** panel WordPress con contraseña débil → atacante modificó contenido del sitio y extrajo catálogo de precios.

---

*Documento generado como parte del laboratorio educativo. Datos ficticios.*
*Referencia: Módulo 1 — Fundamentos, amenazas e identidad · Tríada CIA*
