# 08 - Integración con WhatsApp Cloud API (Meta) en Next.js (Node.js / TypeScript)

Este documento detalla la implementación del **Webhook de WhatsApp Cloud API en Next.js App Router (Node.js / TypeScript)** y la estrategia de **coexistencia con WhatsApp Business App**.

---

## 1. Webhook Handler en Next.js (`src/app/api/webhook/route.ts`)

A continuación se presenta la implementación oficial para recibir eventos de Meta WhatsApp API en Next.js mediante **Route Handlers**:

```typescript
import { NextRequest, NextResponse } from 'next/server';
import crypto from 'crypto';

const META_VERIFY_TOKEN = process.env.META_VERIFY_TOKEN || 'mi_token_seguro';
const META_APP_SECRET = process.env.META_APP_SECRET || 'secret_de_meta';

/**
 * Handshake de verificación de Meta (GET)
 */
export async function GET(request: NextRequest) {
  const searchParams = request.nextUrl.searchParams;
  const mode = searchParams.get('hub.mode');
  const token = searchParams.get('hub.verify_token');
  const challenge = searchParams.get('hub.challenge');

  if (mode === 'subscribe' && token === META_VERIFY_TOKEN) {
    return new NextResponse(challenge, { status: 200 });
  }

  return new NextResponse('Verification token mismatch', { status: 403 });
}

/**
 * Recepción de Mensajes de WhatsApp (POST)
 */
export async function POST(request: NextRequest) {
  try {
    const rawBody = await request.text();
    const signature = request.headers.get('x-hub-signature-256') || '';

    // Validar firma HMAC SHA256 con crypto de Node.js
    const hmac = crypto.createHmac('sha256', META_APP_SECRET);
    const expectedSignature = `sha256=${hmac.update(rawBody).digest('hex')}`;

    if (signature !== expectedSignature) {
      return NextResponse.json({ error: 'Invalid signature' }, { status: 401 });
    }

    const payload = JSON.parse(rawBody);

    // Procesar evento conversacional con el motor de estados Node.js + Supabase
    // await processWhatsappEvent(payload);

    return NextResponse.json({ status: 'ok' }, { status: 200 });
  } catch (error) {
    console.error('Error procesando webhook:', error);
    return NextResponse.json({ error: 'Internal Server Error' }, { status: 500 });
  }
}
```

---

## 2. Envío de Mensajes Interactivos desde Node.js (`src/lib/whatsapp.ts`)

```typescript
import axios from 'axios';

const PHONE_NUMBER_ID = process.env.META_PHONE_NUMBER_ID;
const ACCESS_TOKEN = process.env.META_ACCESS_TOKEN;

/**
 * Envía un mensaje interactivo con botones al usuario de WhatsApp.
 */
export async function sendInteractiveButtons(
  to: string,
  bodyText: string,
  buttons: Array<{ id: string; title: string }>
) {
  const url = `https://graph.facebook.com/v19.0/${PHONE_NUMBER_ID}/messages`;

  const payload = {
    messaging_product: 'whatsapp',
    recipient_type: 'individual',
    to,
    type: 'interactive',
    interactive: {
      type: 'button',
      body: { text: bodyText },
      action: {
        buttons: buttons.map((b) => ({
          type: 'reply',
          reply: { id: b.id, title: b.title },
        })),
      },
    },
  };

  return axios.post(url, payload, {
    headers: {
      Authorization: `Bearer ${ACCESS_TOKEN}`,
      'Content-Type': 'application/json',
    },
  });
}
```

---

## 3. Coexistencia con WhatsApp Business App

La **Coexistencia** permite usar el mismo número en la App de WhatsApp Business y en Meta Cloud API:
- **Modo Atención Manual:** Cuando el residente presiona "Atención manual", el estado se actualiza en **Supabase** a `MODO_MANUAL`.
- El webhook de Next.js ignora temporalmente las entradas de ese usuario, permitiendo que el equipo contable o administrativo responda directamente desde el celular con la app de WhatsApp Business.
- Se puede reanudar enviando el mensaje `/bot`.
