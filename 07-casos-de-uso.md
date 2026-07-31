# 07 - Casos de Uso

Este documento documenta la especificación técnica de los principales **Casos de Uso (CU)** del sistema.

---

## CU-01: Registrar Solicitud de Pago y Cotización (Residente)

| Atributo | Detalle |
| :--- | :--- |
| **Actor Principal** | Residente de Obra / Comprador |
| **Precondición** | El número de WhatsApp del residente debe estar registrado en la tabla `USUARIOS` con rol `RESIDENTE` o `COMPRADOR`. |
| **Flujo Principal** | 1. El residente envía un mensaje al bot.<br>2. El bot saluda y presenta el menú de opciones.<br>3. El residente selecciona "1. Pago Estándar".<br>4. El residente elige la Obra/Proyecto.<br>5. El residente elige la Categoría de material/gasto.<br>6. El residente envía el documento PDF o imagen de la cotización.<br>7. El bot procesa el archivo mediante el módulo OCR, sube la cotización a Google Drive y muestra una vista previa de los datos extraídos.<br>8. El residente indica la Fecha Límite de pago.<br>9. El residente envía la foto del Código QR de Pago.<br>10. El bot sube el QR a Google Drive, genera el registro de compra con estado `PENDIENTE` e ID `#OBRA 501`, y notifica el éxito. |
| **Postcondición** | Registro en `COMPRAS`, `COMPROBANTES`, `COMPRA_COMPROBANTES` y `DETALLE_COMPRA` guardados; archivos almacenados en Drive. |

---

## CU-02: Extracción OCR Híbrida de Cotización (Sistema)

| Atributo | Detalle |
| :--- | :--- |
| **Actor Principal** | Engine OCR (Sistema) |
| **Precondición** | Se ha recibido un archivo PDF o imagen de cotización válido. |
| **Flujo Principal** | 1. El sistema invoca la **Estrategia A (Python nativo - pdfplumber)**.<br>2. Extrae el texto vectorial del PDF y busca patrones regex para NIT, ítems, cantidades, precios unitarios y subtotal/total.<br>3. Si la confianza del parser es mayor a 80%, estructura el resultado JSON y lo retorna.<br>4. Si el documento carece de texto vectorial o no supera la confianza mínima, invoca la **Estrategia B (Gemini Vision API)** enviando el prompt de extracción JSON.<br>5. Guarda el resultado en `OCR_RESULTADOS`. |
| **Excepción** | Si la cotización es completamente ilegible, solicita al residente ingresar el monto total manualmente. |

---

## CU-03: Aprobar o Rechazar Solicitud de Pago (Tesorería)

| Atributo | Detalle |
| :--- | :--- |
| **Actor Principal** | Personal de Contabilidad / Tesorería |
| **Precondición** | Existen compras registradas en estado `PENDIENTE`. |
| **Flujo Principal** | 1. El tesorero envía un mensaje y el bot le reconoce el rol `TESORERIA`.<br>2. El tesorero selecciona consultar solicitudes (Filtradas por Obra o Lista por Vencimiento).<br>3. El bot envía la ficha detallada de la compra y la imagen del QR de Pago.<br>4. El tesorero decide:<br>&nbsp;&nbsp;&nbsp;&nbsp;a) **Rechazar:** Ingresa la razón del rechazo. El bot cambia el estado a `RECHAZADO`, guarda en `APROBACIONES` y notifica al residente.<br>&nbsp;&nbsp;&nbsp;&nbsp;b) **Confirmar Pago:** Realiza la transferencia en la banca en línea y sube la foto/PDF del comprobante bancario al bot.<br>5. El bot sube el comprobante bancario a Google Drive, actualiza el estado a `PAGADO`, guarda en `APROBACIONES` y notifica al residente con el comprobante adjunto. |
