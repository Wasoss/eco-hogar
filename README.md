# EcoHogar-IA 🤖💸

> **Asistente financiero inteligente para Telegram y Google Sheets** — Gestión conversacional de gastos, ingresos, ahorros, alertas de presupuesto y diagnóstico ejecutivo del hogar con IA (Texto y Voz).

![Google Apps Script](https://img.shields.io/badge/Google%20Apps%20Script-V8-4285F4?logo=google&logoColor=white)
![Telegram Bot API](https://img.shields.io/badge/Telegram%20Bot%20API-Webhook-26A5E4?logo=telegram&logoColor=white)
![Groq AI](https://img.shields.io/badge/Groq%20AI-GPT--OSS--20B-F55036)
![Whisper v3](https://img.shields.io/badge/Audio-Whisper%20Large%20v3-10A37F)
![Google Sheets](https://img.shields.io/badge/Database-Google%20Sheets-34A853?logo=googlesheets&logoColor=white)
![Serverless](https://img.shields.io/badge/Deploy-Serverless%2024%2F7-purple)
![License](https://img.shields.io/badge/License-MIT-green.svg)

---

## 📋 Descripción

**EcoHogar-IA** es un asistente contable y financiero multiusuario para Telegram que permite llevar la economía de un hogar en tiempo real mediante **lenguaje natural y notas de voz**, utilizando **Google Sheets** como base de datos transparente y **Google Apps Script** como backend serverless gratuito 24/7.

Diseñado bajo una arquitectura modular de 9 capas, combina clasificación semántica de intenciones en modo JSON estricto (**Groq `openai/gpt-oss-20b`**) y transcripción de audio de alta precisión (**`whisper-large-v3`**) con controles de ciberseguridad, idempotencia transaccional y aislamiento por usuario.

### ✨ Características Principales

| Categoría | Funcionalidades |
|-----------|-----------------|
| 📝 **Registro Conversacional** | Carga de gastos, ingresos y ahorros simples o múltiples en un mismo mensaje (`"almuerzo 4500 efectivo y uber 3200 mercado pago"`) |
| 🎙️ **Notas de Voz (.ogg/.oga)** | Descarga nativa desde Telegram y transcripción automática con **Whisper Large v3** antes de clasificar la intención |
| 📊 **Balances & Filtros Dinámicos** | Resumen del período actual o histórico, filtrado inteligente por persona, categoría, medio de pago o concepto con separación estricta entre ingresos, gastos y ahorros |
| 🗑️ **Smart Undo Aislado** | Deshacer el último movimiento o buscar un gasto específico por concepto/monto (`"deshacer alfajores 4500"`), garantizando que cada integrante solo pueda borrar sus propios registros |
| 🔔 **Alertas & Presupuestos** | Topes personalizables por producto (`"alerta café 15000"`) y alertas automáticas al alcanzar el 80% y 100% del presupuesto mensual por categoría |
| 🎯 **Diagnóstico Ejecutivo IA** | Evaluación financiera a 12 meses (`"¿dónde estoy?"`), cálculo del suelo de seguridad (fondo de contingencia) y sugerencia de 3 decisiones sobre gastos variables controlables |
| 📈 **Revisión Mensual** | Comparativa intermensual (`"revisión"`) que identifica qué rubros mejoraron o empeoraron respecto a meses anteriores y valida el rumbo hacia la meta familiar |
| ⏰ **Recordatorios Automáticos** | Cron diario para avisar vencimientos de gastos fijos (`Alquiler`, `Luz`, `Internet`) y envío automático de cierre semanal todos los domingos |
| 🔄 **Gestión de Ciclos** | Corte de período mensual (`"iniciar nuevo mes"`) sin perder el historial en la planilla, o vaciado total con doble confirmación de seguridad (`"Comenzar de 0"`) |

---

## 🏗️ Arquitectura

```text
ecohogar-telegram-bot/
├── README.md                       # Documentación general y guía de despliegue
├── LICENSE                         # Licencia MIT
├── .gitignore                      # Exclusión de credenciales (.clasp.json, .env)
├── .clasp.json.example             # Plantilla para despliegue CLI con Google Clasp
├── .env.example                    # Plantilla de Script Properties (Variables de Entorno)
├── appsscript.json                 # Manifiesto V8 de Google Apps Script (Timezone AR)
├── src/
│   ├── 01_Config.gs                # Configuración central, Feature Flags y lectura de ScriptProperties
│   ├── 02_Webhook.gs               # Entry point doPost(e), LockService, deduplicación y router IA
│   ├── 03_GroqService.gs           # Integración con Groq API (GPT-OSS-20B JSON Mode + Whisper v3)
│   ├── 04_SheetService.gs          # Escritura batch en Google Sheets, Smart Undo y CRUD de Alertas
│   ├── 05_BalanceService.gs        # Motor contable, balances filtrados y alertas de presupuesto
│   ├── 06_TelegramService.gs       # Cliente Telegram Bot API, Whitelist, Rate Limit y gestión Webhook
│   ├── 07_Tests.gs                 # Suite de tests automatizados (ejecutarTestGeneral y auditoría)
│   ├── 08_CronJobs.gs              # Triggers programados (vencimientos diarios y resumen dominical)
│   └── 09_ObjetivosService.gs      # Diagnóstico ejecutivo a 12 meses, suelo de seguridad y revisión
└── docs/
    └── ESTRUCTURA_PLANILLA.md      # Esquema de columnas y fórmulas para las 5 hojas de Google Sheets
```

### 🔄 Flujo de Procesamiento

```text
[Usuario en Telegram] (Texto o Nota de Voz .oga)
        │
        ▼
[02_Webhook.gs: doPost(e)]
  ├─► 1. LockService + Monotonic LAST_UPDATE_ID + CacheService (Idempotencia contra reintentos)
  ├─► 2. Whitelist Check (06_TelegramService.gs) + Rate Limit (12 msg/min)
  ├─► 3. Si es voz: getTelegramAudioBlob() ──► Whisper Large v3 (03_GroqService.gs)
  └─► 4. Clasificación Semántica JSON (Groq openai/gpt-oss-20b) ──► 14 Intenciones
        │
        ├──► REGISTRAR_MOVIMIENTO ──► Normalización Contable + Batch Append (04_SheetService.gs)
        ├──► BALANCE              ──► Motor de Filtros / Período Actual o Histórico (05_BalanceService.gs)
        ├──► DESHACER             ──► Smart Undo aislado por autor (04_SheetService.gs)
        ├──► DIAGNOSTICO/REVISION ──► Métricas Reales + Tarjeta Ejecutiva IA (09_ObjetivosService.gs)
        └──► ALERTAS / CICLOS     ──► CRUD Hoja Alertas / Corte de Período
```

---

## 🧠 Intenciones Soportadas por la IA

El motor `interpretarMensajeConIA` en `03_GroqService.gs` clasifica cualquier mensaje natural en una de las **14 intenciones estructuradas**:

| Intención | Ejemplos de Mensajes Naturales | Acción Ejecutada |
|-----------|--------------------------------|------------------|
| `REGISTRAR_MOVIMIENTO` | `"almuerzo 8500 efectivo"`, `"cobré sueldo 1.450.000 por transferencia"` | Extrae array de movimientos (`tipo`, `concepto`, `categoria`, `monto`, `moneda`, `medio_pago`, `notas`) y los guarda en lote |
| `BALANCE` | `"balance"`, `"balance Nico"`, `"cuánto gastamos en supermercado"`, `"balance transferencia histórico"` | Filtra por usuario, categoría, medio de pago, concepto o período separando ingresos, gastos y ahorros |
| `DESHACER` | `"deshacer"`, `"borrar último"`, `"eliminar alfajores de 4500"` | Busca y elimina el movimiento más reciente del usuario que envía el comando |
| `DIAGNOSTICO` | `"¿dónde estoy?"`, `"diagnóstico financiero"` | Proyecta el remanente a 12 meses, calcula el suelo de seguridad y propone 3 ajustes sobre gastos variables |
| `REVISION` | `"revisión"`, `"comparar este mes con el anterior"` | Compara la evolución mensual por categorías frente a la meta declarada del hogar |
| `DEFINIR_OBJETIVO` | `"mi situación: entran 2.5M al mes y queremos ahorrar 4M este año"` | Guarda el contexto y objetivos del hogar en `PropertiesService` |
| `ALERTAS_VER` | `"ver alertas"`, `"qué topes tenemos activos"` | Lista las reglas personalizadas configuradas en la pestaña `Alertas` |
| `ALERTAS_CREAR` | `"alerta café 20000"`, `"poner tope a uber en 35000"` | Crea o actualiza un límite máximo mensual en la pestaña `Alertas` |
| `ALERTAS_ELIMINAR` | `"eliminar alerta café"`, `"quitar tope de uber"` | Elimina el tope configurado para ese ítem |
| `GASTOS_FIJOS` | `"gastos fijos"`, `"qué vence este mes"` | Muestra el estado (`Pagado` / `Pendiente`) y día de vencimiento de los servicios del hogar |
| `COMPENSACION` | `"quién le debe a quién"`, `"cuentas claras"` | Calcula la compensación 50/50 sobre gastos compartidos (configurable vía feature flag) |
| `NUEVO_PERIODO` | `"iniciar nuevo mes"`, `"cerrar período"` | Fija la fila de corte (`FILA_INICIO_PERIODO`) para iniciar un nuevo ciclo contable sin borrar el historial |
| `REINICIAR_TODO` | `"Comenzar de 0"`, `"vaciar planilla"` | Solicita confirmación en 2 pasos (`CONFIRMAR`) antes de limpiar las filas de la hoja `Gastos` |
| `AYUDA` | `"/start"`, `"ayuda"`, `"menú de funciones"` | Despliega la guía interactiva de comandos y ejemplos |

---

## 🔒 Seguridad, Idempotencia y Resiliencia

* **Respuesta HTTP 200 Directa (`HtmlService`)**: Evita la redirección HTTP `302 Moved Temporarily` de `ContentService` que provoca bucles de reintentos infinitos en los servidores de Telegram.
* **Triple Barrera de Deduplicación**: Combina `LockService.getScriptLock()`, control monotónico de `LAST_UPDATE_ID` en `PropertiesService` y `CacheService` por `message_id` para garantizar procesamiento **exactamente una vez (idempotente)**.
* **Lista Blanca por ID Numérico (`Whitelist`)**: Solo los IDs de Telegram declarados en `CONFIG.USUARIOS` pueden consultar o modificar la planilla. Los intentos no autorizados quedan registrados en los logs y son rechazados.
* **Normalización Contable Anti-Errores**: Antes de insertar en la planilla o calcular un balance, `esMovimientoIngreso` y `esMovimientoAhorro` verifican tanto el `Tipo` como la `Categoría` y el `Concepto`, evitando que sueldos o ingresos con categoría `"Otros"` se sumen como gastos.

---

## 🚀 Instalación y Despliegue

### 1. Preparar la Planilla de Google Sheets
Crea una planilla en Google Sheets con las 5 pestañas detalladas en [`docs/ESTRUCTURA_PLANILLA.md`](docs/ESTRUCTURA_PLANILLA.md):
* `Gastos`
* `Resumen`
* `Presupuestos`
* `Gastos Fijos`
* `Alertas`

### 2. Configurar el Proyecto en Google Apps Script
1. Desde tu planilla de Google Sheets, abre **Extensiones > Apps Script** (o utiliza [`@google/clasp`](https://github.com/google/clasp) con el archivo `.clasp.json.example`).
2. Copia los 9 archivos de la carpeta `src/` dentro del editor de Apps Script.
3. En **`01_Config.gs`**, coloca los IDs numéricos de Telegram de los miembros del hogar en `CONFIG.USUARIOS` (puedes enviar `/id` al bot para obtener tu ID numérico).
4. Ve a **⚙️ Configuración del proyecto > Propiedades de la secuencia de comandos** y carga las variables indicadas en `.env.example`:
   * `SPREADSHEET_ID`: ID de tu planilla de Google Sheets.
   * `GROQ_API_KEY`: API Key de [Groq Cloud](https://console.groq.com/).
   * `TELEGRAM_TOKEN`: Token generado por `@BotFather` en Telegram.
   * `WEBAPP_URL`: URL de implementación de tu aplicación web terminada en `/exec`.

### 3. Publicar el Webhook y Vincular con Telegram
1. Haz clic en **Implementar > Nueva implementación > Aplicación web**.
2. Configura **Ejecutar como:** *Yo* y **Quién tiene acceso:** *Cualquier persona*.
3. Copia la URL terminada en `/exec` y pégala en la propiedad `WEBAPP_URL`.
4. En el editor de Apps Script, selecciona la función **`configurarWebhookTelegram`** (en `06_TelegramService.gs`) y presiona **▶️ Ejecutar**.

---

## 🧪 Pruebas y Mantenimiento (`07_Tests.gs`)

El módulo `07_Tests.gs` incluye herramientas de verificación ejecutables con 1 clic desde el editor de Apps Script:
* **`ejecutarTestGeneral()`**: Ejecuta una auditoría de 6 etapas (credenciales, existencia de las 5 pestañas, clasificación NLP con Groq, escritura/borrado aislado en lote, cálculo de balances y bloqueo de intrusos en el webhook).
* **`insertarTestEnPlanilla()`**: Inserta una fila de prueba claramente identificada en `Gastos` para verificar permisos de escritura en tiempo real.
* **`repararIngresosYResumenEnPlanilla()`**: Escanea el historial de `Gastos` y corrige automáticamente cualquier ingreso que haya quedado con categoría genérica `"Otros"` o tipo incorrecto.

---

## 📄 Licencia

Este proyecto se distribuye bajo la licencia **MIT**. Consulta el archivo [`LICENSE`](LICENSE) para más detalles.
