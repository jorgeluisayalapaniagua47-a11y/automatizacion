# 01 - Arquitectura del Sistema

Este documento describe la arquitectura general del **Bot de WhatsApp para Gestión de Solicitudes de Pago, Cotizaciones u OCR** para Ecoartec, desarrollado con **Next.js (React + Node.js API Routes)** en **Vercel** con **Supabase (PostgreSQL)** y un servicio de extracción OCR en **Python**.

---

## 1. Visión General de la Arquitectura

El sistema utiliza **Next.js (App Router / Node.js)** como el núcleo web y API principal. Escucha los webhooks de WhatsApp Cloud API de Meta, interactúa con Supabase y orquesta las subidas a Google Drive. Para el OCR, invoca un módulo microservicio de **Python** especializado en la lectura de PDFs e imágenes.

```mermaid
graph TD
    subgraph Cliente ["Canal de Comunicación"]
        WA[WhatsApp App / Business]
    end

    subgraph MetaCloud ["Meta Cloud Platform"]
        WAPI[WhatsApp Cloud API Webhook]
    end

    subgraph VercelNextJS ["Next.js (Node.js + React) - Vercel"]
        WebhookAPI[App Router API: app/api/webhook/route.ts]
        AuthGuard[Auth & Number Guard]
        FlowEngine[Flow & State Engine Node.js]
        
        subgraph ServicesNode ["Servicios Node.js"]
            DriveSvc[Google Drive Service - googleapis npm]
            NotifSvc[WhatsApp Meta Notification Service]
            SupabaseSvc[@supabase/supabase-js]
        end
    end

    subgraph OCRModule ["Módulo de Extracción (Python Microservice / Script)"]
        PyOCR[Python OCR Engine: pdfplumber / EasyOCR]
        Gemini[Gemini Vision API - Fallback IA]
    end

    subgraph Storage ["Infraestructura Persistencia"]
        SupabaseDB[(Supabase PostgreSQL Database)]
        GDrive[Google Drive Workspace]
    end

    WA <--> WAPI
    WAPI <-->|HTTP POST Webhook| WebhookAPI
    WebhookAPI --> AuthGuard
    AuthGuard --> FlowEngine
    FlowEngine --> ServicesNode
    
    FlowEngine <-->|REST / Function Call| PyOCR
    PyOCR -. Fallback .-> Gemini

    ServicesNode <-->|@supabase/supabase-js| SupabaseDB
    DriveSvc <-->|googleapis| GDrive
    NotifSvc --> WAPI
```

---

## 2. Componentes Principales

### 2.1 Webhook Router en Next.js (`app/api/webhook/route.ts`)
- Implementado como **Next.js App Router Route Handler (TypeScript / Node.js)**.
- Recibe peticiones `GET` (verificación de Webhook de Meta) y `POST` (mensajes entrantes).
- Valida la firma `X-Hub-Signature-256` utilizando la librería nativa `crypto` de Node.js.

### 2.2 Motor de Estado de Conversación (`lib/flow-engine.ts`)
- Escrito en TypeScript / Node.js.
- Administra la máquina de estados del bot conversacional interactuando directamente con **Supabase** a través del cliente oficial `@supabase/supabase-js`.

### 2.3 Servicio de Extracción OCR Híbrido (`lib/ocr-service.ts`)
- Invocador TypeScript que canaliza la cotización recibida hacia el motor de extracción en **Python** (`pdfplumber` / `EasyOCR` / `regex`).
- Si la confianza es menor al 80%, invoca el fallback con Gemini Visión API.

### 2.4 Servicio de Google Drive (`lib/drive-service.ts`)
- Implementado en Node.js mediante el paquete oficial `googleapis`.
- Organiza los archivos en las carpetas `Drive/[Nombre_Proyecto]/Cotizaciones/`, `QR_Pagos/` y `Comprobantes_Pago/`.

---

## 3. Diagrama de Despliegue en Vercel

```mermaid
graph LR
    subgraph VercelPlatform ["Vercel Infrastructure"]
        NextApp[Next.js 14/15 App Router - Node.js Runtime]
        PyService[Python Serverless OCR Function]
    end

    subgraph SupabaseCloud ["Supabase Platform"]
        Postgres[(PostgreSQL Cloud Database)]
    end

    subgraph ExternalServices ["APIs Externas"]
        Meta[Meta WhatsApp Cloud API]
        GDriveAPI[Google Drive API v3]
        GeminiAPI[Google Gemini API]
    end

    Meta <-->|Webhook HTTP| NextApp
    NextApp <-->|@supabase/supabase-js| Postgres
    NextApp <--> PyService
    NextApp <--> GDriveAPI
    PyService -.-> GeminiAPI
```
