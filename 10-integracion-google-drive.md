# 10 - Integración con Google Drive en Node.js / Next.js

Este documento describe la arquitectura para conectar el sistema desarrollado en **Node.js / Next.js** con **Google Drive API v3**, permitiendo crear y organizar las carpetas por Proyecto/Obra.

---

## 1. Estructura de Carpetas en Google Drive

```
[Google Drive Raíz Ecoartec]
│
├── Proyecto: Edificio Condominio A/
│   ├── Cotizaciones/
│   │   ├── COT_OBRA501_FerreteriaCentral.pdf
│   │   └── COT_OBRA502_Hormigones.pdf
│   ├── QR_Pagos/
│   │   └── QR_OBRA501.jpg
│   └── Comprobantes_Pago/
│       └── COMP_OBRA501_TransfBancaria.pdf
```

---

## 2. Servicio de Google Drive en TypeScript (`src/lib/drive.ts`)

```typescript
import { google } from 'googleapis';
import fs from 'fs';

const SCOPES = ['https://www.googleapis.com/auth/drive.file'];
const ROOT_FOLDER_ID = process.env.GOOGLE_DRIVE_ROOT_FOLDER_ID || '';

// Inicializar autenticación con Service Account
const auth = new google.auth.GoogleAuth({
  credentials: {
    client_email: process.env.GOOGLE_CLIENT_EMAIL,
    private_key: (process.env.GOOGLE_PRIVATE_KEY || '').replace(/\\n/g, '\n'),
  },
  scopes: SCOPES,
});

const drive = google.drive({ version: 'v3', auth });

/**
 * Busca o crea una carpeta en Google Drive por nombre.
 */
async function getOrCreateFolder(folderName: string, parentId: string): Promise<string> {
  const query = `name = '${folderName}' and '${parentId}' in parents and mimeType = 'application/vnd.google-apps.folder' and trashed = false`;
  
  const res = await drive.files.list({
    q: query,
    fields: 'files(id, name)',
  });

  if (res.data.files && res.data.files.length > 0) {
    return res.data.files[0].id!;
  }

  const fileMetadata = {
    name: folderName,
    mimeType: 'application/vnd.google-apps.folder',
    parents: [parentId],
  };

  const folder = await drive.files.create({
    requestBody: fileMetadata,
    fields: 'id',
  });

  return folder.data.id!;
}

/**
 * Sube un archivo a Drive en la jerarquía: Drive/[Proyecto]/[TipoDocumento]/
 */
export async function uploadToDrive(
  localFilePath: string,
  projectName: string,
  docType: 'Cotizaciones' | 'QR_Pagos' | 'Comprobantes_Pago',
  fileName: string,
  mimeType: string
) {
  // 1. Carpeta del Proyecto
  const projectFolderId = await getOrCreateFolder(projectName, ROOT_FOLDER_ID);

  // 2. Subcarpeta por tipo
  const subFolderId = await getOrCreateFolder(docType, projectFolderId);

  // 3. Subir archivo
  const fileMetadata = {
    name: fileName,
    parents: [subFolderId],
  };

  const media = {
    mimeType: mimeType,
    body: fs.createReadStream(localFilePath),
  };

  const file = await drive.files.create({
    requestBody: fileMetadata,
    media: media,
    fields: 'id, webViewLink',
  });

  // 4. Conceder permisos de lectura pública por enlace
  await drive.permissions.create({
    fileId: file.data.id!,
    requestBody: {
      type: 'anyone',
      role: 'reader',
    },
  });

  return {
    driveId: file.data.id,
    webViewLink: file.data.webViewLink,
  };
}
```
