# 🛒 Laboratorio de Ciberseguridad — Distribuidora Barinas Total C.A.
> **Repositorio:** `rjrb8/lab-pyme-barinas-total`
> **Metodología:** NIST CSF 2.0 | **Marco legal:** Venezuela — LECDI 2001, Ley Orgánica de Protección al Consumidor (LOPCU), Constitución Art. 60
> **Nivel:** Blue Team | Introductorio–Intermedio
> **Módulos de referencia:** Curso de Ciberseguridad — Módulos 1, 2, 3 y 4

---

## 1. Descripción general

Este repositorio contiene un **laboratorio simulado de ciberseguridad** para una organización ficticia del sector comercial (PYME distribuidora) ubicada en el estado Barinas, Venezuela. El propósito es aplicar de forma práctica las seis funciones del **NIST CSF 2.0** (Gobernar, Identificar, Proteger, Detectar, Responder y Recuperar) sobre un entorno realista con historial de incidentes documentado, bajo el marco legal venezolano vigente.

---
<img width="2844" height="1130" alt="NotebookLM Mind Map" src="https://github.com/user-attachments/assets/65136e1e-b53a-4720-9ae6-96bf8d096586" />

---


> ⚠️ **Aviso:** Todos los datos, nombres, RIF, transacciones y sistemas descritos en este repositorio son **completamente ficticios**. Ningún cliente, empleado ni tercero real está involucrado. El laboratorio se realiza exclusivamente con fines educativos.

---

## 2. Historial de incidentes — Contexto del laboratorio

Este laboratorio nace de una **escalada de incidentes reales** en la organización:

### Año 2025 (año anterior)

| Mes | Clasificación | Descripción |
|---|---|---|
| Marzo 2025 | 🟠 **Medio** | Empleado de ventas víctima de phishing: credenciales del sistema de facturación comprometidas. Acceso no autorizado a 320 registros de clientes. Detectado 11 días después por comportamiento anómalo de la cuenta. |
| Agosto 2025 | 🔴 **Crítico** | Ransomware (variante LockBit) cifró el servidor de facturación y la base de datos de inventario. Paralización total de operaciones 72 horas. Pago de rescate no realizado; recuperación parcial desde respaldo de 3 semanas de antigüedad. Pérdida estimada: $4,200 USD en ventas no procesadas. |

### Año 2026 — Primeros 5 meses (enero–mayo)

| Mes | Clasificación | Descripción |
|---|---|---|
| Febrero 2026 | 🔴 **Crítico** | Acceso no autorizado a la cuenta bancaria empresarial mediante credenciales robadas (probable reutilización de contraseña del incidente de 2025). Transferencia fraudulenta de Bs. 18,500 (aprox. $510 USD al tipo oficial). Detectado 48 horas después por el banco. Recuperación parcial en proceso. |
| Mayo 2026 | 🔴 **Crítico** | Defacement del sitio web corporativo y exfiltración del catálogo de precios y lista de 1,400 clientes registrados. Vector: credenciales del panel de administración WordPress sin MFA y con contraseña débil (`admin123`). Datos publicados en foro de la darkweb. |

> **Conclusión del análisis:** La escalada de 1 incidente medio en 2025 a 3 críticos en 18 meses evidencia la ausencia de controles preventivos formalizados y la falta de un plan de respuesta documentado. Este laboratorio define la hoja de ruta correctiva.

---

## 3. La organización ficticia

### 3.1 Ficha institucional

| Campo | Detalle |
|---|---|
| **Nombre** | Distribuidora Barinas Total C.A. |
| **Tipo** | PYME privada — Distribución y comercialización mayorista/minorista |
| **Sector** | Comercial — Productos de consumo masivo, ferretería y papelería |
| **Ubicación** | Av. Marqués del Pumar, Local 14, Barinas, Estado Barinas, Venezuela |
| **RIF ficticio** | J-31045678-2 |
| **Fundación** | 2014 |
| **Empleados** | ~32 personas |
| **Facturación anual estimada** | Bs. 2,400,000 / año (aprox. $66,000 USD al tipo oficial 2026) |
| **Canales de venta** | Local físico, WhatsApp Business, sitio web WordPress, distribuidores zonales |
| **Horario** | Lunes a viernes 7:30 a.m. – 6:00 p.m. · Sábados 8:00 a.m. – 1:00 p.m. |

---
<img width="1536" height="2752" alt="Organigrama_de_Distribuidora_Barinas_Total" src="https://github.com/user-attachments/assets/04f4d5c9-1e61-4e98-ae52-b3ebfbd1298f" />

---


### 3.2 Misión

> Ser el proveedor de referencia de productos de consumo masivo, ferretería y papelería en el estado Barinas, ofreciendo a nuestros clientes y distribuidores un servicio ágil, confiable y transparente, respaldado por sistemas de información seguros que protejan sus datos y los de nuestra operación conforme al marco legal venezolano vigente.

---

### 3.3 Visión

> Consolidarnos en 2028 como la PYME distribuidora líder del estado Barinas en términos de cobertura territorial y confiabilidad operativa, incorporando progresivamente buenas prácticas de seguridad de la información (NIST CSF, CIS Controls) adaptadas al contexto de las PYME venezolanas, para proteger nuestra continuidad de negocio y la confianza de nuestros más de 1,400 clientes registrados.

---

### 3.4 Objetivos institucionales

1. Garantizar la **confidencialidad** de los datos de clientes, precios y acuerdos comerciales conforme al artículo 60 de la Constitución CRBV y la LECDI 2001.
2. Mantener la **disponibilidad** de los sistemas de facturación e inventario con un RTO < 4 horas ante cualquier incidente.
3. Asegurar la **integridad** de los registros de ventas, inventario y cuentas por cobrar, evitando alteraciones no autorizadas.
4. Eliminar los incidentes de seguridad **Críticos** recurrentes documentados en 2025–2026 mediante controles formalizados.
5. Capacitar al **100% del personal** en reconocimiento de phishing y buenas prácticas de contraseñas antes de diciembre de 2026.
6. Cumplir las obligaciones de la **LECDI 2001** y la **Ley Orgánica de Protección al Consumidor (LOPCU)** en el manejo de datos de clientes.

---

## 4. Estructura organizacional

### 4.1 Organigrama

```
                    ┌──────────────────────────────┐
                    │     Junta de Socios           │
                    └─────────────┬────────────────┘
                                  │
                    ┌─────────────▼────────────────┐
                    │    Gerencia General            │
                    │    (Lcdo. Carlos Peñaloza)     │
                    └──┬──────────┬────────────────┘
                       │          │
          ┌────────────▼──┐  ┌────▼──────────────────┐
          │  Gerencia     │  │  Gerencia              │
          │  Comercial    │  │  Administrativa        │
          └──┬────────────┘  └──┬────────────────────┘
             │                  │
    ┌─────────┼──────┐    ┌──────┼─────────────────────┐
    │         │      │    │      │                     │
 Ventas   Despacho  Web  RRHH  Contabilidad /       TI /
 (7)      y Logíst. (1)  (2)   Cuentas por Cobrar  Sistemas
           (4)                  (3)                 (1)
```

### 4.2 Estructura comercial y operativa

| Área | Responsable | Personal | Funciones principales |
|---|---|---|---|
| Ventas | Coord. Ana Vargas | 7 vendedores | Atención al cliente, pedidos, WhatsApp Business |
| Despacho y Logística | Coord. Pedro Ríos | 4 operadores | Preparación de pedidos, despacho, control de almacén |
| Presencia web | (externo freelance) | 1 | WordPress corporativo, catálogo online |
| RRHH | Lic. María Hernández | 2 | Nómina, contrataciones, control de personal |
| Contabilidad / CxC | Cont. Jorge Blanco | 3 | Facturación, cuentas por cobrar, relación bancaria |
| Gerencia General | Lcdo. Carlos Peñaloza | 1 | Dirección estratégica, relaciones con proveedores |

### 4.3 Estructura de Tecnología de la Información (IT)

| Cargo | Nombre ficticio | Responsabilidades |
|---|---|---|
| Coordinador de TI | Téc. Rafael Ojeda | Infraestructura, soporte técnico, seguridad básica |

> ⚠️ **Nota crítica:** La empresa cuenta con **un solo técnico de TI** que atiende todas las necesidades tecnológicas de forma reactiva. No existe un proceso formal de gestión de cambios, ni procedimientos documentados de seguridad.

---
<img width="1536" height="2752" alt="Diagnóstico_de_ciberseguridad_en_PYME" src="https://github.com/user-attachments/assets/3887cd57-d757-464d-8fec-a42f13ecdf05" />

---
<img width="1536" height="2752" alt="Escalada_de_brechas_de_seguridad" src="https://github.com/user-attachments/assets/9e1a41de-ec38-467b-b35a-ee306819772a" />

---


**Stack tecnológico actual:**

| Componente | Detalle |
|---|---|
| **Sistema de facturación** | Profit Plus 2K19 (servidor Windows Server 2012 R2 — sin soporte desde 2023) |
| **Inventario** | Módulo integrado en Profit Plus |
| **Sitio web** | WordPress 6.2 (desactualizado, plugins sin actualizar) |
| **Red** | Router TP-Link Archer AX55, switch no gestionado 16 puertos |
| **WiFi** | Red única para empleados y visitas (sin segmentación) |
| **Correo** | Gmail gratuito (sin Workspace) |
| **Respaldo** | USB externo de 1 TB conectado permanentemente al servidor (sin rotación, sin cifrado) |
| **Antivirus** | Windows Defender (sin consola centralizada) |
| **Firewall** | Solo el del router TP-Link (sin configuración adicional) |
| **Banco** | Banca en línea empresarial desde PCs compartidas de la oficina |

---

## 5. Marco legal aplicable en Venezuela (sector comercial)

| Instrumento | Relevancia para la distribuidora |
|---|---|
| **Constitución CRBV — Art. 60** | Protección de datos personales de clientes (nombre, dirección, teléfono, historial de compras) |
| **Ley Especial contra Delitos Informáticos (LECDI, 2001)** | Tipifica acceso indebido (Art. 6), sabotaje (Art. 7), fraude (Art. 14); obliga a proteger sistemas |
| **Ley Orgánica de Protección al Consumidor (LOPCU)** | Obligación de proteger datos de consumidores registrados |
| **Código de Comercio venezolano** | Obligación de conservar libros y registros contables; integridad de la información contable |
| **Código Penal — Art. 405 y ss.** | Revelación de secretos comerciales y datos de terceros |

---
<img width="2752" height="1536" alt="Hoja_de_ruta_de_ciberseguridad" src="https://github.com/user-attachments/assets/c963aa54-93aa-46d7-9beb-12a364b9073e" />

---

<img width="1536" height="2752" alt="Marco_legal_venezolano_de_ciberseguridad" src="https://github.com/user-attachments/assets/c1276334-a432-4791-96aa-4a9556523fba" />


---


## 6. Estructura del laboratorio

```
lab-pyme-barinas-total/
│
├── README.md                          ← Este documento
│
├── 01-inventario-activos.md          ← Función: IDENTIFICAR
├── 02-analisis-riesgos.md            ← Función: IDENTIFICAR
├── 03-endurecimiento.md              ← Función: PROTEGER
├── 04-deteccion.md                   ← Función: DETECTAR
├── 05-respuesta-continuidad.md       ← Función: RESPONDER / RECUPERAR
├── 06-resumen-ejecutivo.md           ← Función: GOBERNAR
│
├── reporte-tecnico.md                ← Reporte técnico completo
└── reporte-gerencial.md              ← Reporte gerencial ejecutivo
```

---

## 7. Metodología: Loop de IA

```
  ┌──────────────────────────────────────────────┐
  │                                              │
  │  1. OBSERVAR   → Historial de incidentes     │
  │       ↓           2025–2026 como punto de   │
  │                   partida real               │
  │  2. ANALIZAR   → Evaluar activos,            │
  │       ↓           vulnerabilidades y riesgos │
  │  3. DECIDIR    → Seleccionar controles y     │
  │       ↓           tratamientos prioritarios  │
  │  4. ACTUAR     → Implementar y verificar     │
  │       ↓           controles técnicos         │
  │  5. REVISAR    → Medir efectividad y         │
  │       └──────────── reiniciar el ciclo ──────┘
  │
  └──────────────────────────────────────────────┘
```

---

## 8. Referencias

- NIST Cybersecurity Framework 2.0 — https://www.nist.gov/cyberframework
- Ley Especial contra Delitos Informáticos, Venezuela (2001)
- Ley Orgánica de Protección al Consumidor (LOPCU), Venezuela
- Constitución de la República Bolivariana de Venezuela (1999), Art. 60
- CIS Controls v8.1
- Módulos 1–4 del Curso de Ciberseguridad (base académica de este laboratorio)

---

*Laboratorio ficticio con fines exclusivamente educativos.*
*Autor de referencia: RJRB.8 — rjrb8 @ GitHub*
