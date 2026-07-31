# 05 - Requisitos Funcionales

Este documento contiene la matriz detallada de los **Requisitos Funcionales (RF)** del sistema.

---

## Matriz de Requisitos Funcionales

| ID | Requisito | Descripción | Rol Asociado |
| :--- | :--- | :--- | :--- |
| **RF-01** | Autenticación por Teléfono | El sistema debe interceptar todo mensaje entrante de WhatsApp y validar si el número existe en `USUARIOS`. | Todos |
| **RF-02** | Control de Accesos por Rol | El bot debe ofrecer opciones del menú acordes al rol del usuario (`Residente/Comprador` vs `Contabilidad/Tesorería`). | Sistema |
| **RF-03** | Selección de Obra/Proyecto | El residente debe poder elegir la obra destino a partir de una lista dinámica obtenida de `PROYECTOS`. | Residente |
| **RF-04** | Selección de Categoría | El residente debe seleccionar la categoría del gasto según la tabla `CATEGORIAS`. | Residente |
| **RF-05** | Recepción de Cotizaciones | El bot debe permitir la recepción de archivos en formato PDF o imágenes (JPG/PNG) que correspondan a cotizaciones. | Residente |
| **RF-06** | Extracción Automática OCR (Sin IA) | El motor de Python debe procesar la cotización y extraer: NIT del proveedor, ítems de materiales (nombre, cantidad, precio unitario, subtotal) y el total del documento. | Sistema |
| **RF-07** | Interfaz Preparada para Fallback a IA | Si el documento no es legible por reglas nativas de Python, el sistema debe redirigir la extracción hacia la API de Gemini Visión. | Sistema |
| **RF-08** | Captura de Fecha Límite y QR | El bot debe solicitar al residente la fecha límite de pago y la imagen del código QR bancario o datos de la cuenta de destino. | Residente |
| **RF-09** | Subida Automática a Google Drive | Los archivos adjuntados deben subirse a Google Drive en la ruta: `Drive/[Nombre_Proyecto]/Cotizaciones/` y `Drive/[Nombre_Proyecto]/QR_Pagos/`. | Sistema |
| **RF-10** | Generación de Ticket de Solicitud | El bot debe asignar un código único a la compra (ej: `#OBRA 501`) y notificar la creación exitosa al residente. | Sistema |
| **RF-11** | Consulta y Filtrado por Tesorería | El perfil de Contabilidad/Tesorería debe poder consultar solicitudes pendientes filtrando por Obra o por Fecha de Vencimiento. | Tesorería |
| **RF-12** | Visualización de Ficha de Pago | Al seleccionar un ID de compra, el bot debe enviar al tesorero la ficha resumida de la compra y la imagen del QR de Pago. | Tesorería |
| **RF-13** | Rechazo con Motivo | El tesorero debe poder rechazar una solicitud indicando la razón, registrando el evento en `APROBACIONES` y notificando al residente. | Tesorería |
| **RF-14** | Confirmación con Comprobante | El tesorero debe poder confirmar el pago adjuntando la foto/PDF del comprobante de transferencia bancaria final. | Tesorería |
| **RF-15** | Actualización y Almacenamiento Final | El sistema debe subir el comprobante a `Drive/[Nombre_Proyecto]/Comprobantes_Pago/`, cambiar el estado de la compra a `PAGADO` y notificar al residente. | Sistema |
