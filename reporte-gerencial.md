# Reporte Gerencial de Seguridad de la Información
## Distribuidora Barinas Total C.A. — Octubre 2026

**Para:** Gerencia General, Junta de Socios
**De:** Área de Tecnología de la Información
**Clasificación:** Confidencial — Uso interno gerencial

---

## Lo más importante en 4 líneas

> En los últimos 18 meses perdimos más de $4,700 USD por cuatro ataques informáticos. El último expuso los datos de nuestros 1,400 clientes en internet. Los próximos ataques son altamente probables porque las mismas vulnerabilidades siguen abiertas. Podemos cerrarlas por menos de $500 USD.

---

## 1. Qué pasó — Los cuatro ataques en lenguaje simple

### Marzo 2025 — El engaño por correo
Un vendedor recibió un correo que parecía de Gmail pidiéndole actualizar su contraseña. Era falso. El atacante obtuvo la contraseña real y accedió a los datos de 320 clientes. Lo descubrimos 11 días después.

**Por qué pudo pasar:** no teníamos el doble factor de verificación activado en los correos.

---

### Agosto 2025 — El "secuestro" del sistema de facturación
Un archivo adjunto en un correo instaló un programa malicioso en la PC de un vendedor. Ese programa se extendió por nuestra red interna hasta el servidor y cifró (bloqueó) todos los archivos de Profit Plus. Durante 72 horas no pudimos vender, despachar ni facturar.

**Pérdida directa: $4,200 USD en ventas no procesadas.**
El respaldo que teníamos tenía 3 semanas de antigüedad — perdimos datos de casi un mes.

**Por qué pudo pasar:** el servidor usaba un sistema operativo que Microsoft dejó de proteger, no había separación entre la red de los vendedores y el servidor, y el respaldo estaba conectado permanentemente (también fue bloqueado).

---

### Febrero 2026 — El robo de la cuenta bancaria
Un atacante accedió a nuestra banca online empresarial y realizó una transferencia de Bs. 18,500 (aproximadamente $510 USD). Lo detectó el banco dos días después, no nosotros.

**Por qué pudo pasar:** la cuenta bancaria no tenía verificación en dos pasos. Probablemente usaron la misma contraseña robada en el incidente de marzo 2025. Las PCs de contabilidad nunca fueron revisadas para descartar que estuvieran infectadas.

---

### Mayo 2026 — Los datos de nuestros clientes en internet
Un atacante accedió al panel de administración de nuestro sitio web usando la contraseña `admin123`. Desde allí descargó el archivo Excel con los datos de nuestros 1,400 clientes (nombre, RIF, dirección, teléfono, historial de compras) y nuestro catálogo de precios para distribuidores. Esos datos fueron publicados en un foro de internet.

**Por qué pudo pasar:** el panel web no tenía doble verificación, la contraseña era trivial y habíamos guardado archivos confidenciales en el servidor web por error.

---

## 2. El patrón: cada ataque abrió la puerta al siguiente

```
Mar 2025: Phishing → contraseña robada
              ↓ No pusimos doble verificación
Ago 2025: Ransomware → 72h sin operar, $4,200 USD
              ↓ No actualizamos el servidor ni la red
Feb 2026: Fraude bancario → $510 USD robados
              ↓ No revisamos las PCs ni activamos doble verificación bancaria
May 2026: Datos de clientes publicados en internet
              ↓ Sin acción → próximo ataque antes de abril 2027
```

**La señal más preocupante:** cada incidente fue más grave y dañino que el anterior.

---

## 3. Nuestra situación legal

La Ley Orgánica de Protección al Consumidor (LOPCU) y el artículo 60 de la Constitución venezolana obligan a las empresas a proteger los datos de sus clientes. La Ley Especial contra Delitos Informáticos (LECDI 2001) penaliza el sabotaje de sistemas con hasta 6 años de prisión — para el atacante, pero también implica que la empresa puede ser señalada por no tomar medidas preventivas razonables.

**En concreto:** si alguno de nuestros 1,400 clientes cuya información fue publicada en internet sufre un daño (fraude, extorsión) y puede demostrar que provino de nuestra filtración, la empresa podría enfrentar una demanda civil.

**Actuar ahora es también una protección legal.**

---

## 4. Lo que ya hicimos (gratis, esta semana)

| Acción tomada | Qué protege |
|---|---|
| Activamos el sistema de protección del servidor con reglas estrictas | Ahora solo los equipos autorizados pueden conectarse al servidor |
| Separamos la red WiFi de invitados del sistema interno | Visitantes y técnicos externos ya no tienen acceso a nuestra red de trabajo |
| Actualizamos el sitio web y eliminamos los archivos de clientes del hosting | Cerramos la puerta por donde entraron en mayo 2026 |
| Configuramos una alerta automática que detecta el inicio de un ransomware | Si vuelve a ocurrir, lo sabemos en menos de 1 minuto, no cuando ya terminó |

---

## 5. Lo que necesitamos que la Gerencia apruebe

### Aprobación A — Doble verificación en correos, banca y sitio web
**Qué es en simple:** además de la contraseña, el sistema pide un código de 6 dígitos que llega al teléfono. Aunque alguien robe la contraseña, no puede entrar sin el teléfono del empleado.

**¿Por qué no lo hemos hecho?** Requiere que cada empleado configure la app en su teléfono. Necesitamos 2 horas de trabajo conjunto de TI con los empleados.

**Costo:** $0. Solo tiempo.

**Urgencia:** Esta semana. Es la medida más efectiva y no cuesta nada.

---

### Aprobación B — Equipo de respaldo (NAS) y copia en la nube
**Qué es en simple:** en agosto 2025, cuando el ransomware cifró el servidor, también cifró el USB de respaldo porque estaba conectado al mismo servidor. Necesitamos un sistema de respaldo con tres copias: una en el nuevo equipo NAS (guardado en el cuarto de TI), una en disco externo guardado en la oficina de gerencia, y una copia automática cifrada en internet.

**¿Para qué sirve?** Si mañana ocurriera otro ransomware, tendríamos el sistema funcionando de nuevo en menos de 4 horas, no 72 horas.

**Costo estimado:**
- Equipo NAS Synology: ~$180–$220 USD (compra única)
- Servicio de respaldo en la nube: ~$10–$15 USD al mes

**Urgencia:** Este mes.

---

### Aprobación C — Actualización del servidor
**Qué es en simple:** el servidor que corre nuestro sistema de facturación (Profit Plus) usa Windows Server 2012 R2 — un sistema que Microsoft dejó de proteger en octubre de 2023. Tiene más de 3 años sin recibir actualizaciones de seguridad. Es como tener la puerta de nuestro almacén con una cerradura que el fabricante ya no repara y de la que existen llaves maestras circulando.

**La solución:** migrar a Ubuntu Server 22.04, un sistema gratuito con soporte de seguridad hasta 2032.

**Costo estimado:** $0 en licencias. $0–$300 USD en soporte técnico externo si se necesita.

**Urgencia:** Próximos 60 días.

---

### Aprobación D — Taller mensual de 30 minutos para el equipo
**Qué es en simple:** los 7 vendedores aprenden a reconocer correos falsos y adjuntos maliciosos. El ataque de marzo 2025 (que inició la cadena) fue posible porque un vendedor no reconoció el engaño.

**Costo:** $0. Solo 30 minutos al mes del tiempo del equipo.

**Urgencia:** Primera sesión en las próximas 2 semanas.

---

## 6. Cuánto cuesta actuar vs. cuánto cuesta no actuar

| | Actuar ahora | No actuar |
|---|---|---|
| **Costo monetario** | $180–$500 USD única vez + $15/mes | Historial muestra: > $4,700 USD en 18 meses |
| **Tiempo de recuperación ante un ataque** | < 4 horas (con respaldo 3-2-1) | 72+ horas (como en agosto 2025) |
| **Riesgo de exposición de clientes** | Muy bajo (con MFA y controles activos) | Alto (ya ocurrió en mayo 2026) |
| **Riesgo legal** | Bajo (empresa tomó medidas razonables) | Alto (segunda filtración = reclamos posibles) |

---

## 7. Próximos pasos concretos

| Qué | Quién | Cuándo |
|---|---|---|
| Activar doble verificación en Gmail (todos los empleados) | TI coordina con cada persona | Esta semana |
| Activar doble verificación en banca online | Gerente General contacta al banco | Esta semana |
| Comprar equipo NAS y contratar servicio de nube cifrada | Gerencia autoriza compra | Antes del 31 oct. |
| Reinstalar las PCs de contabilidad desde cero | TI | Próximos 7 días |
| Primer taller de phishing para vendedores (30 min) | TI organiza | Antes del 20 oct. |
| Iniciar migración del servidor Profit Plus | TI (con soporte externo si es necesario) | Nov.–Dic. 2026 |

---

## 8. Solicitud formal

Solicitamos a la Gerencia General la aprobación formal de las Decisiones A, B, C y D descritas en este reporte, con asignación de presupuesto para la compra del NAS (≈ $200 USD) y autorización del gasto mensual del servicio de nube (≈ $15 USD/mes).

Adicionalmente, solicitamos una reunión de 30 minutos esta semana para coordinar la activación del doble factor de verificación con todos los empleados.

---

*Tecnología de la Información — Distribuidora Barinas Total C.A.*
*Octubre 2026 · Laboratorio educativo ficticio — RJRB.8*
