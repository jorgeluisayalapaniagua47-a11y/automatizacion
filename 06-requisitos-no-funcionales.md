# 06 - Requisitos No Funcionales

Este documento establece las directrices relativas a **rendimiento, seguridad, disponibilidad, mantenibilidad y escalabilidad** del sistema.

---

## 1. Rendimiento y Tiempos de Respuesta
- **Respuesta a Webhooks:** El endpoint de FastAPI debe responder a los webhooks de WhatsApp Cloud API en **menos de 2000 ms** (respondiendo HTTP `200 OK` inmediatamente y procesando tareas pesadas como OCR de forma asíncrona mediante trabajadores en segundo plano).
- **Procesamiento de OCR sin IA:** El análisis de PDFs digitales mediante `pdfplumber` no debe demorar más de **3 segundos por documento**.
- **Procesamiento de OCR con Fallback de IA:** En caso de invocar la API de Gemini Visión, el tiempo total de extracción no debe superar los **8 segundos**.

---

## 2. Seguridad y Coexistencia
- **Validación de Webhooks:** Verificación estricta de la firma HMAC SHA-256 (`X-Hub-Signature-256`) enviada por Meta en cada petición.
- **Credenciales y Secretos:** Cero hardcoding de claves API. Se deben almacenar en variables de entorno `.env` (API Keys de Gemini, Meta App Secret, Google Service Account JSON).
- **Aislamiento de Accesos en WhatsApp:** Los usuarios no autorizados no podrán ejecutar ninguna acción del menú ni interactuar con la base de datos.
- **Coexistencia con WhatsApp Business App:** El webhook debe filtrar y diferenciar eventos de mensajería automatizada sin bloquear los chats atendidos manualmente por agentes humanos en la app móvil.

---

## 3. Disponibilidad y Escalabilidad
- **Disponibilidad del Servicio (SLA):** El servicio bot debe garantizar un uptime mínimo del 99.5%.
- **Resiliencia en Subida a Drive:** En caso de fallas temporales de red al subir a Google Drive, el sistema debe reintentar automáticamente con retardo exponencial (Backoff).

---

## 4. Mantenibilidad y Trazabilidad
- **Patrón Strategy para OCR:** El módulo de extracción debe estar completamente desacoplado del flujo de WhatsApp, permitiendo agregar nuevos parsers o modelos de IA sin modificar el flujo conversacional.
- **Auditoría:** Registro estructurado de logs para cada webhook recibido, tiempo de ejecución de OCR, subidas a Drive y cambios de estado en las solicitudes de compra.
