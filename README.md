# Ecoartec - Sistema de Automatización de Solicitudes de Pago, Cotizaciones u OCR por WhatsApp

Este repositorio contiene la arquitectura y especificaciones técnicas para el **Bot de WhatsApp de Gestión de Solicitudes de Pago**, desarrollado con **Next.js (Node.js + React)**, desplegado en **Vercel**, conectado a **Supabase (PostgreSQL)** y con un motor especializado en **Python** para el procesamiento OCR de cotizaciones y comprobantes.

---

## 🚀 Despliegue Rápido (Next.js en Vercel + Supabase)

1. **Base de Datos:** Ejecuta el archivo [schema.sql](schema.sql) en el **SQL Editor** de **Supabase**.
2. **Aplicación Web / Webhook:** Despliega este repositorio en **Vercel** (`npm run build`).
3. **Variables de Entorno en Vercel:**
   - `NEXT_PUBLIC_SUPABASE_URL`: URL de tu proyecto Supabase.
   - `SUPABASE_SERVICE_ROLE_KEY`: Clave de servicio de Supabase.
   - `META_VERIFY_TOKEN`: Token de verificación del Webhook de Meta.
   - `META_APP_SECRET`: Secret de tu App en Meta for Developers.
   - `META_PHONE_NUMBER_ID`: ID de número en Meta.
   - `META_ACCESS_TOKEN`: Token de acceso permanente de Meta WhatsApp API.
   - `GOOGLE_CLIENT_EMAIL`: Email del Service Account de Google Cloud.
   - `GOOGLE_PRIVATE_KEY`: Clave privada RSA del Service Account.
   - `GOOGLE_DRIVE_ROOT_FOLDER_ID`: ID de la carpeta raíz compartida en Google Drive.

---

## 📚 Índice de Documentación

| Documento | Descripción |
| :--- | :--- |
| 📄 [01-arquitectura.md](01-arquitectura.md) | Arquitectura desacoplada basada en Next.js (Node.js/React), Vercel, Supabase y Python OCR. |
| 📄 [02-tecnologias.md](02-tecnologias.md) | Stack técnico (Next.js 14/15, TypeScript, Node.js, `@supabase/supabase-js`, `googleapis`, `pdfplumber`, `EasyOCR`). |
| 📄 [03-flujo-procesos.md](03-flujo-procesos.md) | Diagrama de flujo detallado en Mermaid basado en la interacción conversacional por WhatsApp. |
| 📄 [04-base-de-datos.md](04-base-de-datos.md) | Diagrama ERD de la base de datos relacional y configuración en Supabase Studio. |
| 📄 [05-requisitos-funcionales.md](05-requisitos-funcionales.md) | Matriz de Requisitos Funcionales (RF-01 a RF-15). |
| 📄 [06-requisitos-no-funcionales.md](06-requisitos-no-funcionales.md) | Requisitos de rendimiento, seguridad, disponibilidad y mantenibilidad. |
| 📄 [07-casos-de-uso.md](07-casos-de-uso.md) | Especificación técnica de Casos de Uso (CU-01, CU-02, CU-03). |
| 📄 [08-integracion-whatsapp-meta.md](08-integracion-whatsapp-meta.md) | Webhook en Next.js App Router (`src/app/api/webhook/route.ts`), validación HMAC y mensajes interactivos. |
| 📄 [09-estrategia-ocr-extraccion.md](09-estrategia-ocr-extraccion.md) | Extractor OCR en Python (`pdfplumber` + `EasyOCR` + `pyzbar` + `regex`) para cotizaciones y comprobantes. |
| 📄 [10-integracion-google-drive.md](10-integracion-google-drive.md) | Servicio de Google Drive API v3 en Node.js (`googleapis` npm) con Service Account. |
| 🗄️ [schema.sql](schema.sql) | Script ejecutable SQL DDL para creación de la base de datos en Supabase / PostgreSQL. |
| 📦 [package.json](package.json) | Configuración de paquetes y dependencias de Next.js / Node.js. |
