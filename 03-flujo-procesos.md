# 03 - Flujo de Procesos del Bot

Este documento detalla el **flujo conversacional y operacional** del sistema, reflejando fielmente la lógica de interacción diseñada para el canal de WhatsApp.

---

## 1. Diagrama de Flujo Principal (Mermaid)

```mermaid
flowchart TD
    Start([Inicio: Recepción de Mensaje]) --> RegCheck{¿Número registrado?}
    
    RegCheck -- No --> Unauth[Finalizar: Usuario no Autorizado]
    RegCheck -- Sí --> RoleCheck{Evaluar ROL}
    
    %% Rama Residente / Comprador
    RoleCheck -- "Residente / Comprador" --> ResFlow[Carga de Solicitudes y Pagos]
    ResFlow --> MenuPay[MENÚ DE SOLICITUD DE PAGO]
    
    MenuPay --> MenuOpt{Seleccionar opción}
    MenuOpt -- "Opción 2: Atención Manual" --> AdminContact[Hablar con administración]
    AdminContact --> ManualAttn[Atención manual]
    
    MenuOpt -- "Opción 1: Pago Estándar" --> Step11[1.1 Seleccionar Obra / Proyecto]
    Step11 --> Step12[1.2 Seleccionar Categoría]
    Step12 --> Step13[1.3 Ingresar Proveedor, Cotización y Monto]
    
    subgraph OCRExtraction ["Subproceso Extracción OCR / IA"]
        Step13 --> OCREngine[Extracción de Datos de Cotización]
        OCREngine --> StrategyDecision{Estrategia de Extracción}
        StrategyDecision -- "Nativa (Default)" --> PyEngine[Parser Python: pdfplumber / regex]
        StrategyDecision -- "Fallback (Ilegible)" --> GeminiEngine[API Key Gemini Visión]
        PyEngine --> OCRData[Extracción NIT, ítems, cant, subtotal, total]
        GeminiEngine --> OCRData
    end
    
    OCRData --> Step14[1.4 Indicar Fecha Límite]
    Step14 --> Step15[1.5 Enviar Foto / Código QR de Pago]
    Step15 --> Step16["1.6 Generar ID (#OBRA 501), Subir a Drive y Mandar a Cola"]
    
    %% Rama Contabilidad / Administración / Tesorería
    RoleCheck -- "Contabilidad / Administración" --> AdminPanel[Panel de Aprobación y Gestión]
    AdminPanel --> Accounting[CONTABILIDAD]
    Accounting --> AuthNumCheck{Rol de número autorizado}
    
    AuthNumCheck --> ShowOpts[Mostrar opciones de consulta]
    ShowOpts --> FilterObra[Filtrar por Obra]
    ShowOpts --> ViewDueDate[Ver lista general por vencimiento]
    
    FilterObra --> SelectID[Usuario selecciona ID específico]
    ViewDueDate --> SelectID
    
    SelectID --> SystemDeploy[Sistema despliega detalles y envía Foto/QR de Pago al Tesorero]
    SystemDeploy --> ActionCheck{¿Acción del Tesorero?}
    
    %% Rechazo
    ActionCheck -- "A. RECHAZAR" --> RejectFlow[RECHAZAR SOLICITUD]
    RejectFlow --> MarkReject[Sistema marca como RECHAZADO]
    MarkReject --> InputReason[Indicar razón del rechazo]
    InputReason --> SaveApprovalReject[Registrar en APROBACIONES y Notificar]
    
    %% Confirmación de Pago
    ActionCheck -- "B. CONFIRMAR PAGO" --> ConfirmFlow[CONFIRMAR PAGO]
    ConfirmFlow --> StepB1[B.1 Realizar transferencia externa bancaria]
    StepB1 --> StepB2[B.2 Subir foto/PDF del comprobante al bot]
    StepB2 --> StepB3[B.3 Subir comprobante a Drive y actualizar estado a PAGADO]
    StepB3 --> StepB4[B.4 Notificar al Residente de Obra]
```

---

## 2. Descripción Detallada de Pasos

### 2.1 Autenticación e Invocación
- **Evento:** El bot recibe una carga útil (payload) del webhook de Meta WhatsApp Cloud API.
- **Validación:** Compara el `telefono` en formato E.164 (ej: `+59170000000`) contra la tabla `USUARIOS`.
- **Resultado:** Si no existe, responde con un mensaje indicando que el número no está autorizado.

### 2.2 Menú Residente / Comprador
- **Seleccionar Obra:** El bot despliega una lista interactiva de obras activas asociadas al usuario desde la tabla `PROYECTOS`.
- **Seleccionar Categoría:** Despliega las categorías de gasto/material desde `CATEGORIAS`.
- **Carga de Cotización:** El residente adjunta la cotización (documento PDF o foto).
  - El archivo se guarda temporalmente, se extraen sus metadatos y se ejecuta el extractor OCR.
- **Fecha Límite:** El usuario ingresa la fecha máxima para efectuar el pago (formato legible o mediante selección).
- **Foto/QR de Pago:** El residente envía la imagen del código QR bancario o los datos de la cuenta de destino.
- **Confirmación:** El bot genera un código visible para el usuario (ej: `#OBRA-501`), almacena los archivos en Google Drive en la carpeta `Drive/[Nombre_Proyecto]/Cotizaciones/` y crea el registro en `COMPRAS` con estado `PENDIENTE`.

### 2.3 Menú Contabilidad / Tesorería
- **Panel de Control:** El tesorero consulta compras pendientes filtrando por **Obra** o por **Fecha de Vencimiento**.
- **Vista de Detalle:** El bot envía una ficha resumen con:
  - Nombre del Proveedor y NIT.
  - Lista de Ítems extraídos (cantidad, precio unitario, subtotal).
  - Total a pagar y Fecha Límite.
  - La imagen del **QR de Pago** para escaneo directo.
- **Rechazo:** Se le solicita un motivo de texto que se guarda en la tabla `APROBACIONES` y se notifica de inmediato al residente.
- **Confirmación:** Tras transferir en su banca electrónica, el tesorero envía la imagen del comprobante de transferencia al bot. El bot la sube a Google Drive (`Drive/[Nombre_Proyecto]/Comprobantes_Pago/`), actualiza la compra a `PAGADO` y notifica al residente.
