# 02 - Tecnologías y Stack Tecnológico

Este documento especifica las tecnologías, bibliotecas y herramientas del stack basado en **Next.js (React + Node.js)** para la plataforma principal y **Python** para la extracción OCR.

---

## 1. Stack Principal

| Capa / Módulo | Tecnología Seleccionada | Justificación / Descripción |
| :--- | :--- | :--- |
| **Framework Web & API** | **Next.js 14/15 (React 18/19 + Node.js)** | Framework full-stack en TypeScript para construir tanto los Webhooks/APIs como el panel web administrativo si se requiere. |
| **Lenguaje Backend Core** | **TypeScript / Node.js 20+** | Tipado estricto, alto rendimiento asíncrono y excelente ecosistema para desarrollo ágil en Vercel. |
| **Base de Datos Cloud** | **Supabase (PostgreSQL 15)** | Instancia administrada de PostgreSQL con cliente oficial `@supabase/supabase-js`. |
| **Plataforma de Despliegue** | **Vercel** | Plataforma nativa para Next.js con soporte inmediato para API Routes, Serverless Functions y entorno de producción. |
| **Módulo OCR Especializado** | **Python 3.11+** | Microservicio / Script asíncrono de extracción utilizando `pdfplumber`, `EasyOCR`, `pyzbar` y `OpenCV`. |
| **Canal de Mensajería** | **Meta WhatsApp Cloud API** | API oficial de Meta para mensajería empresarial con mensajes interactivos y coexistencia. |
| **Almacenamiento de Archivos** | **Google Drive API v3 (`googleapis` npm)** | Integración nativa desde Node.js para crear y gestionar carpetas por proyecto. |

---

## 2. Dependencias de Node.js / JavaScript (`package.json`)

| Paquete npm | Uso Específico |
| :--- | :--- |
| **`next`** | Framework de React para backend API Routes y frontend. |
| **`@supabase/supabase-js`** | SDK oficial de Supabase para consultas a la DB, autenticación y tiempo real. |
| **`googleapis`** | SDK de Google Cloud para autenticación de Service Account y subida a Google Drive. |
| **`axios` / `node-fetch`** | Cliente HTTP para enviar peticiones a Meta WhatsApp Cloud API y al servicio OCR. |
| **`zod`** | Validación de esquemas y payloads de la API de Meta y formularios. |
| **`tailwind css` / `lucide-react`** | Estilizado UI rápido para el panel de administración web en React. |

---

## 3. Módulo OCR en Python (Invocado desde Node.js)

| Biblioteca Python | Uso Específico |
| :--- | :--- |
| **`pdfplumber`** | Parseo tabular de cotizaciones PDF digitales. |
| **`EasyOCR` / `PaddleOCR`** | OCR sobre fotos de cotizaciones impresas o facturas físicas. |
| **`pyzbar`** | Decodificación de códigos QR en comprobantes bancarios. |
| **`google-genai`** | Fallback con Gemini 1.5/2.0 Visión cuando el OCR tradicional no sea suficiente. |

---

## 4. Estructura del Proyecto Next.js + Python

```
automatizacion/
├── 01-arquitectura.md
├── 02-tecnologias.md
├── 03-flujo-procesos.md
├── 04-base-de-datos.md
├── 05-requisitos-funcionales.md
├── 06-requisitos-no-funcionales.md
├── 07-casos-de-uso.md
├── 08-integracion-whatsapp-meta.md
├── 09-estrategia-ocr-extraccion.md
├── 10-integracion-google-drive.md
├── schema.sql
├── package.json                # Proyecto Node.js / Next.js
├── next.config.mjs             # Configuración Next.js
├── tsconfig.json               # Configuración TypeScript
├── vercel.json                 # Configuración de despliegue Vercel
├── ocr_engine/                 # Microservicio de OCR en Python
│   ├── main.py                 # FastAPI / Script extractor
│   └── requirements.txt        # Dependencias de Python
└── src/                        # Código fuente Next.js (Node.js + React)
    ├── app/
    │   ├── layout.tsx          # Layout principal de React
    │   ├── page.tsx            # Dashboard web de administración
    │   └── api/
    │       └── webhook/
    │           └── route.ts    # Webhook principal WhatsApp Meta API (TypeScript)
    └── lib/
        ├── supabase.ts         # Cliente Supabase
        ├── whatsapp.ts         # Servicio de mensajes WhatsApp Cloud API
        ├── drive.ts            # Servicio de Google Drive API (googleapis)
        └── ocr-client.ts       # Cliente conector al extractor de Python
```
