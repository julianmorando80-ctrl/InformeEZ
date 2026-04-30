# CLAUDE.md — Proyecto EZ

## Descripción del Proyecto

Sistema de gestión y comunicación para un estudio contable. Tiene dos funciones principales:

1. **Supervisión interna** — seguimiento de empleados, clientes asignados, tareas y horas.
2. **Diagnóstico mensual para clientes** — documento que se genera por cliente, se completa con datos del Excel de la base y se envía como PDF por email.

El sistema vive en el navegador (HTML + JS puro, sin frameworks, sin backend). Los datos vienen de un archivo Excel (.xlsx) que el usuario carga manualmente desde la interfaz.

---

## Stack Técnico

- **HTML + CSS + JavaScript** puro — sin frameworks, sin Node, sin build steps
- **SheetJS (xlsx.js)** — para leer el archivo Excel en el navegador
- **Sin backend** — todo corre localmente en el navegador
- **Sin localStorage** — los datos no persisten entre sesiones (el usuario carga el Excel cada vez)
- Apto para abrir directamente con doble clic en el archivo `.html`, sin servidor

---

## Estructura de Archivos del Proyecto

```
proyecto-ez/
├── CLAUDE.md               ← este archivo
├── index.html              ← diagnóstico mensual (app principal)
├── data/
│   └── clientes.xlsx       ← base de datos de clientes (el usuario lo reemplaza con el suyo)
├── assets/
│   ├── logo.png            ← logo del estudio (reemplazable)
│   └── style.css           ← estilos compartidos (si se separa del HTML)
└── README.md               ← instrucciones de uso para el usuario final
```

---

## Paleta de Colores y Estética

- **Fondo principal:** blanco `#FFFFFF`
- **Color primario:** celeste suave `#E3F2FD` / azul medio `#1565C0`
- **Color acento:** rojo suave `#FFEBEE` / rojo medio `#C62828`
- **Texto principal:** `#1A1A2E`
- **Texto secundario / labels:** `#6B7280`
- **Bordes y separadores:** `#E0E0E0`
- **Tipografía:** `'Lato'` para cuerpo, `'Playfair Display'` para títulos y encabezados (Google Fonts)
- **Estética:** profesional, limpia, de estudio contable — sin efectos llamativos, sin sombras dramáticas, sin gradientes agresivos

---

## Estructura del Excel de Clientes (`clientes.xlsx`)

El Excel tiene **una fila por cliente**. La primera fila son los encabezados exactos. El HTML lee estas columnas:

| Columna | Nombre exacto en Excel | Descripción |
|---|---|---|
| A | `razon_social` | Nombre o razón social del cliente |
| B | `cuit` | CUIT o CUIL |
| C | `tipo` | `"empresa"` o `"persona_fisica"` |
| D | `condicion_fiscal` | Ej: `"Responsable Inscripto"`, `"Monotributista"` |
| E | `responsable` | Empleado del estudio asignado |
| F | `cierre_ejercicio` | Fecha de cierre. Ej: `"31/12"`, `"30/06"` |
| G | `anticipo_ganancias` | Descripción de anticipos. Ej: `"5 cuotas bimestrales"` |
| H | `proximo_anticipo_fecha` | Fecha próximo anticipo. Ej: `"15/06/2025"` |
| I | `proximo_anticipo_monto` | Monto. Ej: `"85000"` |
| J | `bienes_personales` | `"si"` o `"no"` |
| K | `vto_bienes_personales` | Fecha vencimiento DDJJ Bienes Personales |
| L | `categoria_monotributo` | Categoría si es monotributista. Ej: `"H"` |
| M | `facturacion_anual` | Facturación últimos 12 meses (número) |
| N | `limite_categoria` | Límite de la categoría actual (número) |
| O | `empleados_cantidad` | Cantidad de empleados (solo empresas) |
| P | `honorarios_mensuales` | Honorarios del estudio (número) |
| Q | `saldo_cuenta` | Saldo cuenta corriente con el estudio (número, negativo = deuda) |
| R | `email_cliente` | Email para el envío |
| S | `observaciones` | Texto libre con observaciones previas |

---

## Funcionamiento del `index.html`

### Flujo de uso
1. El usuario abre `index.html` en el navegador
2. Hace clic en **"Cargar Excel"** → selecciona su `clientes.xlsx`
3. Aparece un **selector desplegable** con todos los clientes del Excel
4. Selecciona un cliente → el formulario se pre-carga con todos sus datos
5. Los campos que vienen del Excel aparecen **pre-cargados pero editables**
6. Los campos que NO están en el Excel aparecen **vacíos y editables** (para completar a mano)
7. El usuario completa/ajusta lo necesario
8. Hace clic en **"Generar PDF"** → se abre el diagnóstico listo para imprimir/guardar como PDF

### Campos pre-cargados desde Excel (automáticos)
- Razón social / nombre
- CUIT / CUIL
- Condición fiscal
- Responsable asignado
- Cierre de ejercicio
- Anticipos de ganancias (fecha y monto)
- Bienes personales (aplica o no)
- Categoría monotributo
- Honorarios mensuales
- Saldo cuenta corriente
- Email del cliente
- Observaciones

### Campos siempre editables (se completan mes a mes)
- Período del diagnóstico (mes/año)
- Fecha de emisión
- Estado de cada impuesto (IVA, IIBB, Ganancias, etc.)
- Importes liquidados ese mes
- Detalle de la situación previsional del período
- Observaciones y recomendaciones del mes

---

## Secciones del Diagnóstico

El documento generado tiene estas secciones **en este orden**:

### Encabezado
- Logo del estudio (desde `assets/logo.png`)
- Nombre, teléfono y email del estudio
- Período del diagnóstico
- Datos del cliente (razón social, CUIT, condición fiscal, responsable, cierre de ejercicio)

### 1. Situación Impositiva
Tabla con columnas: Impuesto | Período | Vencimiento | Importe | Estado

Estados posibles con badge de color:
- ✅ `Presentado` — verde
- 🔄 `En curso` — azul
- ⚠️ `Pendiente` — amarillo
- 🔴 `Sin presentar` — rojo

Impuestos que aparecen (se muestran u ocultan según tipo de cliente):
- IVA (solo Resp. Inscripto)
- Ganancias — Anticipo
- Ganancias — DDJJ Anual (solo cuando aplica)
- Ingresos Brutos
- Monotributo (solo monotributistas)
- Bienes Personales (solo personas físicas)
- Impuesto sobre Cheques (solo empresas, opcional)

### 2. Situación Previsional
- Para **empresas**: F.931, cantidad de empleados, importe cargas sociales, vencimiento, ART
- Para **personas físicas monotributistas**: incluye obra social y jubilación en cuota, control de facturación vs límite de categoría, fecha de recategorización

### 3. Fechas y Vencimientos Clave
Grid de tarjetas con los próximos vencimientos del mes. Colores según urgencia:
- Rojo: vence en menos de 7 días
- Amarillo: vence en 8-15 días
- Verde: vence en más de 15 días

### 4. Cuenta Corriente con el Estudio
- Honorarios del mes
- Saldo (deuda en rojo, saldo a favor en verde, al día en gris)

### 5. Observaciones y Recomendaciones
Texto libre editable. Aparece pre-cargado con las observaciones del Excel si las hay.

---

## Comportamiento de Campos Editables

- Todos los valores son editables en pantalla antes de imprimir
- Campos con datos del Excel: se muestran pre-cargados, con borde punteado celeste al hacer foco
- Campos vacíos (sin dato en Excel): se muestran con placeholder en gris claro y borde punteado rojo suave para indicar que requieren carga manual
- Al imprimir (`Ctrl+P` o botón "Generar PDF"), los bordes y placeholders desaparecen

---

## Reglas de Desarrollo

- **No usar frameworks** (React, Vue, Angular, etc.)
- **No usar Node.js ni npm** — el proyecto se abre directamente en el navegador
- **No usar localStorage** — sin persistencia entre sesiones
- **SheetJS** se carga desde CDN: `https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js`
- **Google Fonts** se cargan desde CDN (Lato + Playfair Display)
- El HTML debe funcionar con doble clic, sin servidor
- Todo el JS va en el mismo `index.html` o en un archivo `app.js` en la raíz
- Los estilos de impresión van en un bloque `@media print` dentro del mismo archivo
- **No inventar datos** — si un campo no está en el Excel, mostrar placeholder, nunca hardcodear un valor

---

## Datos del Estudio (configurables arriba del HTML)

Al inicio del archivo `index.html`, en un bloque de constantes JS claramente comentado, definir:

```js
// ── CONFIGURACIÓN DEL ESTUDIO ─────────────────────
const ESTUDIO = {
  nombre: "Estudio Contable & Asociados",
  telefono: "(011) 4000-0000",
  email: "info@estudio.com.ar",
  logo: "assets/logo.png"
};
// ─────────────────────────────────────────────────
```

El usuario edita esta sección una sola vez para personalizar el estudio.

---

## Lo que NO hace este sistema

- No envía emails automáticamente (el PDF se genera y el usuario lo adjunta manualmente)
- No se conecta a AFIP ni a ningún sistema contable
- No guarda datos entre sesiones
- No tiene login ni usuarios
- No tiene base de datos propia — la base es el Excel del usuario
