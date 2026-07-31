# 11 - Opciones y Alternativas a Google Drive para Almacenamiento

Este documento analiza las principales **alternativas de almacenamiento de archivos** frente a Google Drive para guardar cotizaciones, imágenes de QR y comprobantes de pago.

---

## 1. Tabla Comparativa de Alternativas

| Servicio | Tipo | Ventajas Principales | Desventajas / Consideraciones | Costo / Tier Gratuito | Ideal Para |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Supabase Storage** | Object Storage (S3 API) | • **100% Integrado con tu DB Supabase**.<br>• Usas el mismo SDK (`@supabase/supabase-js`).<br>• Permisos con Row Level Security (RLS).<br>• URLs públicas y firmadas al instante. | • Los archivos no se ven como una carpeta tradicional de Drive en la PC (requiere dashboard). | **1 GB Gratis**<br>($0.021 / GB adicional) | **Opción #1 Recomendada** por simplicidad y cero configuración extra. |
| **Cloudflare R2** | S3-Compatible Storage | • **$0 de costo de ancho de banda (Egress)**.<br>• Altísima velocidad global.<br>• Compatible con el SDK oficial de AWS S3. | • Requiere configurar un bucket en Cloudflare. | **10 GB Gratis / mes**<br>($0.015 / GB almacenado) | Empresas que buscan **mínimo costo a escala** y descargas ilimitadas. |
| **AWS S3** | Object Storage Estándar | • Estándar de la industria mundial.<br>• Máxima durabilidad (99.999999999%).<br>• Vencimiento y archivado automático a Glacier. | • Cobran por descarga de ancho de banda (Egress).<br>• Consola de AWS más compleja. | **5 GB Gratis** (1er año)<br>($0.023 / GB) | Proyectos empresariales grandes con infraestructura en Amazon AWS. |
| **Cloudinary** | CDN de Medios & Fotos | • Optimización y compresión automática de fotos.<br>• Detección y OCR nativo de texto en imágenes.<br>• Generación de miniaturas (Thumbnails). | • Orientado principalmente a imágenes (menos enfocado en PDFs complejos). | **25 Créditos Gratis** (~25 GB / mes) | Si el 90% de tus archivos son **fotos/capturas de celular** y códigos QR. |
| **Microsoft OneDrive / SharePoint** | Cloud Drive Corporativo | • Si la empresa ya paga Microsoft 365 / Office.<br>• Los archivos aparecen sincronizados en el explorador de Windows de la empresa. | • La API de Microsoft Graph es más compleja de autenticar (OAuth 2.0 Azure AD). | Incluido en planes M365 existentes | Empresas integradas en el ecosistema Microsoft. |

---

## 2. Opción Recomendada #1: Supabase Storage

Dado que ya estás utilizando **Supabase** como base de datos, **Supabase Storage** es la opción más limpia y eficiente, ya que no requiere gestionar credenciales de Google ni servicios de terceros adicionales.

### 2.1 Estructura de Buckets en Supabase Storage
1. **`cotizaciones`** (Bucket público/privado para PDFs y fotos de proveedores).
2. **`qr-pagos`** (Bucket para las fotos de los códigos QR de pago).
3. **`comprobantes-pago`** (Bucket para los recibos de transferencia bancaria).

---

### 2.2 Ejemplo de Subida desde Next.js con `@supabase/supabase-js` (`lib/storage-service.ts`)

```typescript
import { createClient } from '@supabase/supabase-js';

const supabase = createClient(
  process.env.NEXT_PUBLIC_SUPABASE_URL!,
  process.env.SUPABASE_SERVICE_ROLE_KEY!
);

/**
 * Sube un archivo a Supabase Storage y retorna su URL pública.
 */
export async function uploadToSupabaseStorage(
  fileBuffer: Buffer,
  fileName: string,
  bucketName: 'cotizaciones' | 'qr-pagos' | 'comprobantes-pago',
  proyectoFolder: string,
  contentType: string
) {
  // Ruta dentro del bucket: ej: "Proyecto-Condominio-A/COT-OBRA501.pdf"
  const filePath = `${proyectoFolder}/${Date.now()}_${fileName}`;

  const { data, error } = await supabase.storage
    .from(bucketName)
    .upload(filePath, fileBuffer, {
      contentType: contentType,
      upsert: true,
    });

  if (error) {
    throw new Error(`Error al subir a Supabase Storage: ${error.message}`);
  }

  // Obtener URL Pública del archivo
  const { data: publicUrlData } = supabase.storage
    .from(bucketName)
    .getPublicUrl(data.path);

  return {
    path: data.path,
    publicUrl: publicUrlData.publicUrl,
  };
}
```

---

## 3. Ejemplo Opción #2: Cloudflare R2 (Cero costos de transferencia)

Si se desea almacenar terabytes de archivos sin pagar por el tráfico de datos al descargarlos:

```typescript
import { S3Client, PutObjectCommand } from '@aws-sdk/client-s3';

const r2Client = new S3Client({
  region: 'auto',
  endpoint: `https://${process.env.CLOUDFLARE_ACCOUNT_ID}.r2.cloudflarestorage.com`,
  credentials: {
    accessKeyId: process.env.R2_ACCESS_KEY_ID!,
    secretAccessKey: process.env.R2_SECRET_ACCESS_KEY!,
  },
});

export async function uploadToCloudflareR2(
  fileBuffer: Buffer,
  bucketName: string,
  key: string,
  mimeType: string
) {
  const command = new PutObjectCommand({
    Bucket: bucketName,
    Key: key,
    Body: fileBuffer,
    ContentType: mimeType,
  });

  await r2Client.send(command);

  return `https://${process.env.R2_PUBLIC_DOMAIN}/${key}`;
}
```

---

## 4. Conclusión y Recomendación para Ecoartec

1. **Si quieres simplicidad inmediata y 0 APIs externas adicionales:** Usa **Supabase Storage**. Todo vive en la misma plataforma donde tienes tu base de datos y no necesitas autenticar Service Accounts de Google.
2. **Si el equipo administrativo necesita ver los archivos sincronizados en la PC como si fuera una carpeta normal de Windows:** Mantén **Google Drive** o usa **OneDrive / SharePoint**.
3. **Si el volumen de compras y comprobantes crece a miles por mes:** Migra a **Cloudflare R2** para costo $0 de transferencia.
