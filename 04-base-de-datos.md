# 04 - Base de Datos y Modelo Entidad-Relación (Supabase)

Este documento define la estructura del modelo relacional de la base de datos administrada en **Supabase (PostgreSQL Cloud)**, el diagrama ER en Mermaid y el script DDL ejecutable desde la consola SQL de Supabase.

---

## 1. Diagrama Entidad-Relación (Mermaid)

```mermaid
erDiagram
    CLIENTES ||--o{ PROYECTOS : "posee"
    PROYECTOS ||--o{ COMPRAS : "contiene"
    PROVEEDORES ||--o{ COMPRAS : "provee"
    CATEGORIAS ||--o{ MATERIALES : "clasifica"
    CATEGORIAS ||--o{ COMPRAS : "categoriza"
    UNIDADES_MEDIDA ||--o{ MATERIALES : "mide"
    MATERIALES ||--o{ DETALLE_COMPRA : "incluido_en"
    MATERIALES ||--o{ INVENTARIO : "mantiene_stock"
    
    COMPRAS ||--o{ DETALLE_COMPRA : "tiene"
    COMPRAS ||--o{ COMPRA_COMPROBANTES : "vincula"
    COMPROBANTES ||--o{ COMPRA_COMPROBANTES : "asociado_a"
    COMPROBANTES ||--o{ OCR_RESULTADOS : "procesado_por"
    
    INVENTARIO ||--o{ MOVIMIENTOS_INVENTARIO : "registra"
    DETALLE_COMPRA ||--o{ MOVIMIENTOS_INVENTARIO : "origina"
    
    USUARIOS ||--o{ APROBACIONES : "realiza"
    COMPRAS ||--o{ APROBACIONES : "recibe"
```

---

## 2. Conexión desde Vercel a Supabase

En Supabase, para el despliegue en Vercel Serverless, se debe usar la **cadena de conexión de PgBouncer (Transaction Pooler)** en el puerto `6543`:

```env
# URL de Conexión en Vercel (.env)
DATABASE_URL="postgres://postgres.[PROJECT_REF]:[PASSWORD]@aws-0-[REGION].pooler.supabase.com:6543/postgres?pgbouncer=true"
SUPABASE_URL="https://[PROJECT_REF].supabase.co"
SUPABASE_ANON_KEY="eyJhbGci..."
```

---

## 3. Script DDL SQL Compatible con Supabase Studio (`schema.sql`)

Ejecutar el siguiente script en el **SQL Editor de Supabase**:

```sql
-- DDL para Supabase PostgreSQL

CREATE TABLE IF NOT EXISTS CLIENTES (
    cliente_id BIGSERIAL PRIMARY KEY,
    razon_social VARCHAR(255) NOT NULL,
    nit VARCHAR(50),
    telefono VARCHAR(50),
    correo VARCHAR(100)
);

CREATE TABLE IF NOT EXISTS PROYECTOS (
    proyecto_id BIGSERIAL PRIMARY KEY,
    cliente_id BIGINT REFERENCES CLIENTES(cliente_id),
    codigo VARCHAR(50) UNIQUE NOT NULL,
    nombre VARCHAR(255) NOT NULL,
    presupuesto DECIMAL(14,2) DEFAULT 0.00,
    fecha_inicio DATE,
    fecha_fin DATE,
    estado VARCHAR(50) DEFAULT 'ACTIVO'
);

CREATE TABLE IF NOT EXISTS PROVEEDORES (
    proveedor_id BIGSERIAL PRIMARY KEY,
    razon_social VARCHAR(255) NOT NULL,
    nit VARCHAR(50) UNIQUE,
    telefono VARCHAR(50),
    correo VARCHAR(100)
);

CREATE TABLE IF NOT EXISTS CATEGORIAS (
    categoria_id BIGSERIAL PRIMARY KEY,
    nombre VARCHAR(100) NOT NULL,
    descripcion TEXT
);

CREATE TABLE IF NOT EXISTS UNIDADES_MEDIDA (
    unidad_id BIGSERIAL PRIMARY KEY,
    nombre VARCHAR(50) NOT NULL,
    abreviatura VARCHAR(20) NOT NULL
);

CREATE TABLE IF NOT EXISTS MATERIALES (
    material_id BIGSERIAL PRIMARY KEY,
    categoria_id BIGINT REFERENCES CATEGORIAS(categoria_id),
    unidad_id BIGINT REFERENCES UNIDADES_MEDIDA(unidad_id),
    codigo VARCHAR(50) UNIQUE,
    nombre VARCHAR(255) NOT NULL,
    stock_minimo DECIMAL(12,2) DEFAULT 0.00,
    activo BOOLEAN DEFAULT TRUE
);

CREATE TABLE IF NOT EXISTS COMPROBANTES (
    comprobante_id BIGSERIAL PRIMARY KEY,
    nombre_archivo VARCHAR(255) NOT NULL,
    url_drive TEXT NOT NULL,
    hash_archivo VARCHAR(64),
    estado VARCHAR(50) DEFAULT 'SUBIDO',
    fecha_subida TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE IF NOT EXISTS OCR_RESULTADOS (
    ocr_id BIGSERIAL PRIMARY KEY,
    comprobante_id BIGINT REFERENCES COMPROBANTES(comprobante_id) ON DELETE CASCADE,
    modelo_ia VARCHAR(100) NOT NULL,
    confianza DECIMAL(5,2),
    json_extraido TEXT NOT NULL,
    fecha_proceso TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE IF NOT EXISTS COMPRAS (
    compra_id BIGSERIAL PRIMARY KEY,
    proyecto_id BIGINT REFERENCES PROYECTOS(proyecto_id),
    proveedor_id BIGINT REFERENCES PROVEEDORES(proveedor_id),
    categoria_id BIGINT REFERENCES CATEGORIAS(categoria_id),
    numero_factura VARCHAR(100),
    total DECIMAL(14,2) NOT NULL,
    fecha_compra DATE DEFAULT CURRENT_DATE,
    fecha_limite DATE,
    estado VARCHAR(50) DEFAULT 'PENDIENTE'
);

CREATE TABLE IF NOT EXISTS COMPRA_COMPROBANTES (
    relacion_id BIGSERIAL PRIMARY KEY,
    compra_id BIGINT REFERENCES COMPRAS(compra_id) ON DELETE CASCADE,
    comprobante_id BIGINT REFERENCES COMPROBANTES(comprobante_id) ON DELETE CASCADE,
    tipo_comprobante VARCHAR(50) NOT NULL
);

CREATE TABLE IF NOT EXISTS DETALLE_COMPRA (
    detalle_compra_id BIGSERIAL PRIMARY KEY,
    compra_id BIGINT REFERENCES COMPRAS(compra_id) ON DELETE CASCADE,
    material_id BIGINT REFERENCES MATERIALES(material_id),
    cantidad DECIMAL(12,2) NOT NULL,
    precio_unitario DECIMAL(12,2) NOT NULL,
    subtotal DECIMAL(14,2) NOT NULL
);

CREATE TABLE IF NOT EXISTS INVENTARIO (
    inventario_id BIGSERIAL PRIMARY KEY,
    material_id BIGINT UNIQUE REFERENCES MATERIALES(material_id),
    stock_actual DECIMAL(12,2) DEFAULT 0.00
);

CREATE TABLE IF NOT EXISTS MOVIMIENTOS_INVENTARIO (
    movimiento_id BIGSERIAL PRIMARY KEY,
    inventario_id BIGINT REFERENCES INVENTARIO(inventario_id),
    detalle_compra_id BIGINT REFERENCES DETALLE_COMPRA(detalle_compra_id),
    tipo VARCHAR(20) NOT NULL,
    cantidad DECIMAL(12,2) NOT NULL,
    fecha_movimiento TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE IF NOT EXISTS USUARIOS (
    usuario_id BIGSERIAL PRIMARY KEY,
    nombre VARCHAR(150) NOT NULL,
    rol VARCHAR(50) NOT NULL,
    telefono VARCHAR(50) UNIQUE NOT NULL
);

CREATE TABLE IF NOT EXISTS APROBACIONES (
    aprobacion_id BIGSERIAL PRIMARY KEY,
    compra_id BIGINT REFERENCES COMPRAS(compra_id) ON DELETE CASCADE,
    usuario_id BIGINT REFERENCES USUARIOS(usuario_id),
    decision VARCHAR(50) NOT NULL,
    observacion TEXT,
    fecha TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);
```
