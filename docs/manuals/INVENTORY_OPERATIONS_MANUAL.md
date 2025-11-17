# MANUAL DE OPERACIONES - MÓDULO INVENTORY

**Sistema**: TAIMEX ERP
**Módulo**: Inventory Management
**Versión**: 1.0
**Fecha**: 2025-11-16

---

## 📋 TABLA DE CONTENIDOS

1. [Visión General](#1-visión-general)
2. [Procesos Principales](#2-procesos-principales)
   - 2.1 [Shipping Labels - Local (SHIPLCLABEL)](#21-shipping-labels---local-shiplclabel)
   - 2.2 [Shipping Labels - Inner Box (SHIPILABEL)](#22-shipping-labels---inner-box-shipilabel)
   - 2.3 [Stock Checks (SHIPCHECK/STOCKCHECK)](#23-stock-checks-shipcheckstockcheck)
   - 2.4 [Inventory Adjustments (INVIR/INVPIN)](#24-inventory-adjustments-invirinvpin)
   - 2.5 [Inventory Audit Process (IAP)](#25-inventory-audit-process-iap)
   - 2.6 [Ship Out (SHIPOUT)](#26-ship-out-shipout)
   - 2.7 [Stock Transactions (STOCKTRANS)](#27-stock-transactions-stocktrans)
3. [Referencia de Endpoints](#3-referencia-de-endpoints)
4. [Servicios XML/API](#4-servicios-xmlapi)
5. [Variables de Sesión](#5-variables-de-sesión)
6. [Base de Datos](#6-base-de-datos)
7. [Glosario](#7-glosario)

---

## 1. VISIÓN GENERAL

### 1.1 Propósito del Módulo

El módulo **Inventory** de TAIMEX gestiona todas las operaciones relacionadas con:
- Generación de etiquetas de envío (locales e inner box)
- Verificación de embarques mediante escaneo de códigos de barras
- Consultas y ajustes de inventario
- Auditoría de inventario físico
- Transacciones de stock (entradas/salidas)
- Procesamiento de embarques salientes

### 1.2 Estructura General

```
inventory/
├── Shipping Labels (Local)      → 8 archivos (P1-P8)
├── Shipping Labels (Inner Box)  → 6 archivos
├── Stock Checks                 → 8 archivos
├── Inventory Adjustments        → 8 archivos
├── Inventory Audit Process      → 10 archivos (legacy + modern)
├── Ship Out                     → 6 archivos
├── Stock Transactions           → 3 archivos
├── Stock Locations              → 1 archivo
└── XML Services                 → 7 archivos
```

### 1.3 Puntos de Acceso Principales

**Desde MAIN_MENU.HTM:**
- Stock Transfers → `STOCKTRANS_EDIT.HTM`
- Create Stock Transaction → `STOCKTRANS_CREATE.HTM`
- Stock Locations → `stockloc_edit.htm`
- Part Inquiry → `NBPART.HTM`

**Desde NBMMEDIT.HTM** (menú no encontrado físicamente, referenciado en archivos):
- Shipping Labels Local → `SHIPLCLABELP1EDIT.HTM`
- Shipping Labels Inner Box → `SHIPILABELP1.HTM`
- Ship Check → `SHIPCHECKEDIT.HTM`
- Inventory Inquiry → `INVPIN.HTM`
- Raw Material Inquiry → `INVIRMEDIT.HTM`
- Inventory Audit → `IAPEDIT1.HTM`

---

## 2. PROCESOS PRINCIPALES

## 2.1 Shipping Labels - Local (SHIPLCLABEL)

### **Descripción**
Proceso para generar etiquetas de envío para embarques locales/domésticos a partir de órdenes de venta.

### **Flujo Completo**

```
┌─────────────────────────────────────────────────────────────┐
│  SHIPLCLABELP1EDIT.HTM                                      │
│  ┌───────────────────────────────────────┐                 │
│  │ Escanear Número de Orden              │                 │
│  │ Input: ORDNUM                         │                 │
│  └───────────────────────────────────────┘                 │
│           │                                                 │
│           ├─ [CANCEL] → NBMMEDIT.HTM                      │
│           ├─ [Error] → PALMERROR2EDIT.HTM                  │
│           └─ [Valid] ↓                                      │
└─────────────────────────────────────────────────────────────┘
                        │
                        ↓
┌─────────────────────────────────────────────────────────────┐
│  SHIPLCLABELP2EDIT.HTM                                      │
│  ┌───────────────────────────────────────┐                 │
│  │ Seleccionar Cliente                   │                 │
│  │ Input: CUSTNO (dropdown)              │                 │
│  │ Display: ORDNUM                       │                 │
│  └───────────────────────────────────────┘                 │
│           │                                                 │
│           ├─ [CANCEL] → NBMMEDIT.HTM                      │
│           └─ [NEXT] ↓                                       │
└─────────────────────────────────────────────────────────────┘
                        │
                        ↓
┌─────────────────────────────────────────────────────────────┐
│  SHIPLCLABELP3EDIT.HTM                                      │
│  ┌───────────────────────────────────────┐                 │
│  │ Captura de Dirección y Detalles       │                 │
│  │ Inputs:                                │                 │
│  │  - SHIP_NAME (nombre destinatario)    │                 │
│  │  - SHIP_ADDR1-4 (dirección)           │                 │
│  │  - SHIP_CITY, SHIP_STATE, SHIP_ZIP    │                 │
│  │  - CONTACT (contacto)                 │                 │
│  │  - PHONE (teléfono)                   │                 │
│  │  - PKGS (número de paquetes)          │                 │
│  │  - WT (peso)                          │                 │
│  └───────────────────────────────────────┘                 │
│           │                                                 │
│           ├─ [CANCEL] → NBMMEDIT.HTM                      │
│           └─ [PRINT LABEL] ↓                                │
└─────────────────────────────────────────────────────────────┘
                        │
                        ↓
┌─────────────────────────────────────────────────────────────┐
│  SHIPLCLABELP4EDIT.HTM                                      │
│  ┌───────────────────────────────────────┐                 │
│  │ Vista Previa de Etiqueta               │                 │
│  │ Display: Todos los datos capturados   │                 │
│  └───────────────────────────────────────┘                 │
│           │                                                 │
│           ├─ [CANCEL] → NBMMEDIT.HTM                      │
│           ├─ [EDIT] → SHIPLCLABELP3EDIT.HTM                │
│           └─ [CONFIRM & PRINT] ↓                            │
└─────────────────────────────────────────────────────────────┘
                        │
                        ↓
┌─────────────────────────────────────────────────────────────┐
│  SHIPLCLABELP5EDIT.HTM                                      │
│  ┌───────────────────────────────────────┐                 │
│  │ Generación de Etiqueta                 │                 │
│  │ Action: Llama PRINTSHIPLABEL          │                 │
│  └───────────────────────────────────────┘                 │
│           │                                                 │
│           ├─ [PRINT ANOTHER] → SHIPLCLABELP1EDIT.HTM       │
│           └─ [MAIN MENU] → NBMMEDIT.HTM                   │
└─────────────────────────────────────────────────────────────┘
```

### **Endpoints Utilizados**

| Archivo | URL | Método | Propósito |
|---------|-----|--------|-----------|
| SHIPLCLABELP1EDIT.HTM | /SHIPLCLABELP1EDIT.HTM | GET | Entrada de orden |
| SHIPLCLABELP2EDIT.HTM | /SHIPLCLABELP2EDIT.HTM | GET | Selección de cliente |
| SHIPLCLABELP3EDIT.HTM | /SHIPLCLABELP3EDIT.HTM | GET | Captura de dirección |
| SHIPLCLABELP4EDIT.HTM | /SHIPLCLABELP4EDIT.HTM | GET | Vista previa |
| SHIPLCLABELP5EDIT.HTM | /SHIPLCLABELP5EDIT.HTM | GET | Generación de label |

### **Reglas de Negocio**

1. **Validación de Orden**: La orden debe existir en el sistema
2. **Cliente**: Se puede seleccionar de una lista de clientes asociados
3. **Dirección**: Todos los campos de dirección son requeridos
4. **Paquetes y Peso**: Campos numéricos, validados en backend
5. **Impresión**: Llama a subrrutina BASIC `PRINTSHIPLABEL`

### **Variables de Sesión**

- `ORDNUM` - Número de orden de venta
- `CUSTNO` - Número de cliente
- `SHIP_NAME` - Nombre del destinatario
- `SHIP_ADDR1-4` - Líneas de dirección
- `SHIP_CITY`, `SHIP_STATE`, `SHIP_ZIP` - Ciudad, estado, código postal
- `CONTACT` - Persona de contacto
- `PHONE` - Teléfono
- `PKGS` - Número de paquetes
- `WT` - Peso del embarque

---

## 2.2 Shipping Labels - Inner Box (SHIPILABEL)

### **Descripción**
Proceso para generar etiquetas de caja interior para partes individuales de inventario.

### **Flujo Completo**

```
┌─────────────────────────────────────────────────────────────┐
│  SHIPILABELP1.HTM / SHIPILABEL1EDIT.HTM                     │
│  ┌───────────────────────────────────────┐                 │
│  │ Escanear Número de Parte              │                 │
│  │ Input: PARTNUM                        │                 │
│  │ Nota: Strip leading 'P' si presente   │                 │
│  └───────────────────────────────────────┘                 │
│           │                                                 │
│           ├─ [CANCEL/CANCEL2] → NBMMEDIT.HTM              │
│           ├─ [Error] → PALMERROR2EDIT.HTM                  │
│           └─ [Valid Part] ↓                                 │
└─────────────────────────────────────────────────────────────┘
                        │
                        ↓
┌─────────────────────────────────────────────────────────────┐
│  SHIPILABELP2.HTM / SHIPILABELP2A.HTM                       │
│  ┌───────────────────────────────────────┐                 │
│  │ Paso 2 del Workflow                    │                 │
│  │ (Detalles de implementación varían)   │                 │
│  └───────────────────────────────────────┘                 │
│           │                                                 │
│           └─ [Continue] ↓                                   │
└─────────────────────────────────────────────────────────────┘
                        │
                        ↓
┌─────────────────────────────────────────────────────────────┐
│  SHIPILABELP3.HTM / SHIPILABELP3EDIT.HTM                    │
│  ┌───────────────────────────────────────┐                 │
│  │ Generación de Etiqueta Inner Box       │                 │
│  │ Inputs: PARTNUM, QTY, otros campos    │                 │
│  └───────────────────────────────────────┘                 │
│           │                                                 │
│           ├─ [CANCEL] → NBMMEDIT.HTM                      │
│           └─ [PRINT] → Genera etiqueta                      │
└─────────────────────────────────────────────────────────────┘
```

### **Endpoints Utilizados**

| Archivo | URL | Método | Propósito |
|---------|-----|--------|-----------|
| SHIPILABELP1.HTM | /SHIPILABELP1EDIT.HTM | GET | Entrada de parte |
| SHIPILABEL1EDIT.HTM | /SHIPILABEL1EDIT.HTM | GET | Entrada alternativa |
| SHIPILABELP2.HTM | /SHIPILABELP2.HTM | GET | Paso 2 |
| SHIPILABELP2A.HTM | /SHIPILABELP2A.HTM | GET | Paso 2 alternativo |
| SHIPILABELP3.HTM | /SHIPILABELP3.HTM | GET | Generación |
| SHIPILABELP3EDIT.HTM | /SHIPILABELP3EDIT.HTM | GET | Generación (modern) |

### **Reglas de Negocio**

1. **Part Number**:
   - Si comienza con 'P', se elimina automáticamente
   - Debe existir en archivo PARTS
2. **Validación**: Backend valida contra base de datos PARTS
3. **Cantidad**: Campo requerido para etiqueta

### **Variables de Sesión**

- `PARTNUM` - Número de parte (sin 'P' inicial)
- `QTY` - Cantidad

---

## 2.3 Stock Checks (SHIPCHECK/STOCKCHECK)

### **Descripción**
Sistema de verificación de embarques mediante escaneo de códigos de barras de packing lists y partes.

### **Flujo Completo - SHIPCHECK (Verificación de Embarque)**

```
┌─────────────────────────────────────────────────────────────┐
│  SHIPCHECKEDIT.HTM                                          │
│  ┌───────────────────────────────────────┐                 │
│  │ Escanear Packing List                 │                 │
│  │ Input: INVNUM (número de factura)     │                 │
│  │ Nota: Strip '11K' prefix si presente  │                 │
│  └───────────────────────────────────────┘                 │
│           │                                                 │
│           ├─ [MENU] → NBMMEDIT.HTM                         │
│           ├─ [Error] → Error display                       │
│           └─ [Valid Invoice] ↓                              │
└─────────────────────────────────────────────────────────────┘
                        │
                        ↓
┌─────────────────────────────────────────────────────────────┐
│  SHIPCHECKEDIT2.HTM                                         │
│  ┌───────────────────────────────────────┐                 │
│  │ Escanear Partes en Cajas               │                 │
│  │ Input: PARTNO (repetible)             │                 │
│  │ Nota: Strip '1P' prefix si presente   │                 │
│  │ Display: "GOOD SCAN!" animation       │                 │
│  └───────────────────────────────────────┘                 │
│           │                                                 │
│           │ [OK] → Escanear siguiente parte (loop)         │
│           ├─ [BACK] → SHIPCHECKEDIT.HTM (borra data)       │
│           ├─ [DONE] ↓                                       │
│           └─ [MENU] → NBMMEDIT.HTM                         │
└─────────────────────────────────────────────────────────────┘
                        │
                        ↓
┌─────────────────────────────────────────────────────────────┐
│  SHIPCHECK3.HTM                                             │
│  ┌───────────────────────────────────────┐                 │
│  │ Resultados de Verificación             │                 │
│  │ Action: BUILDSHIPSCAN subroutine      │                 │
│  │ Writes to: UPSADD file                │                 │
│  └───────────────────────────────────────┘                 │
│           │                                                 │
│           ├─ [BACK] → SHIPCHECK2.HTM                       │
│           └─ [COMPLETE] → SHIPCHECK.HTM (reset)            │
└─────────────────────────────────────────────────────────────┘
```

### **Endpoints Utilizados**

| Archivo | URL | Método | Propósito |
|---------|-----|--------|-----------|
| SHIPCHECKEDIT.HTM | /SHIPCHECKEDIT.HTM | GET | Escanear packing list |
| SHIPCHECKEDIT2.HTM | /SHIPCHECKEDIT2.HTM | GET | Escanear partes |
| SHIPCHECK3.HTM | /SHIPCHECK3.HTM | GET | Resultados |

### **Reglas de Negocio**

1. **Packing List**:
   - Strip prefix '11K' si presente
   - Debe existir en archivo SHIPMENTS
2. **Escaneo de Partes**:
   - Strip prefix '1P' si presente
   - Valida contra SHIPMENTS y SO (Sales Orders)
   - Escribe cada escaneo a archivo SHIPSCAN
3. **Verificación**:
   - Compara partes escaneadas vs. esperadas
   - Genera reporte en UPSADD
   - Llama subrrutina BUILDSHIPSCAN

### **Archivos de Base de Datos**

- `SHIPMENTS` - Datos de embarques
- `SO` - Órdenes de venta
- `SHIPSCAN` - Registro de escaneos
- `UPSADD` - Resultados de verificación UPS

---

## 2.4 Inventory Adjustments (INVIR/INVPIN)

### **Descripción**
Consultas de inventario para partes y materias primas.

### **Flujo INVPIN (Inventory Part Inquiry)**

```
┌─────────────────────────────────────────────────────────────┐
│  INVPIN.HTM                                                 │
│  ┌───────────────────────────────────────┐                 │
│  │ Consulta de Inventario                 │                 │
│  │ Input: PARTNUM                        │                 │
│  └───────────────────────────────────────┘                 │
│           │                                                 │
│           ├─ [Error] → PALMERROR2EDIT.HTM                  │
│           └─ [Valid] ↓                                      │
└─────────────────────────────────────────────────────────────┘
                        │
                        ↓
┌─────────────────────────────────────────────────────────────┐
│  INVPIN2.HTM                                                │
│  ┌───────────────────────────────────────┐                 │
│  │ Resultados de Inventario               │                 │
│  │ Action: INVINQ subroutine             │                 │
│  │ Display: Niveles de inventario        │                 │
│  └───────────────────────────────────────┘                 │
│           │                                                 │
│           └─ [MENU] → NBMMEDIT.HTM                        │
└─────────────────────────────────────────────────────────────┘
```

### **Flujo INVIRMEDIT (Raw Material Inquiry)**

```
┌─────────────────────────────────────────────────────────────┐
│  INVIRMEDIT.HTM                                             │
│  ┌───────────────────────────────────────┐                 │
│  │ Consulta Materia Prima                 │                 │
│  │ Input: PARTNUM                        │                 │
│  └───────────────────────────────────────┘                 │
│           │                                                 │
│           ├─ [CLEAR] → Limpia formulario                   │
│           ├─ [MENU] → NBMMEDIT.HTM                        │
│           ├─ [Error] → PALMERROR2EDIT.HTM                  │
│           └─ [Valid] ↓                                      │
└─────────────────────────────────────────────────────────────┘
                        │
                        ↓
┌─────────────────────────────────────────────────────────────┐
│  INVIP2RMEDIT.HTM                                           │
│  ┌───────────────────────────────────────┐                 │
│  │ Resultados Raw Material + IQC          │                 │
│  │ Display:                               │                 │
│  │  - BOH (Balance on Hand)              │                 │
│  │  - IQC data                           │                 │
│  │  - "No Items Found" si BOH = 0        │                 │
│  └───────────────────────────────────────┘                 │
│           │                                                 │
│           ├─ [BACK] → INVIRMEDIT.HTM                       │
│           ├─ [CREATE] → INVPULL1EDIT.HTM                   │
│           └─ [MENU] → NBMMEDIT.HTM                        │
└─────────────────────────────────────────────────────────────┘
```

### **Flujo IQEDIT (Inventory by Location)**

```
┌─────────────────────────────────────────────────────────────┐
│  IQEDIT.HTM                                                 │
│  ┌───────────────────────────────────────┐                 │
│  │ Búsqueda por Ubicación                 │                 │
│  │ Input: LOCTN (location)               │                 │
│  │ Action: RUN RPROGS LOC.XREF           │                 │
│  └───────────────────────────────────────┘                 │
│           │                                                 │
│           ├─ [MENU] → STOCKCHECKEDIT.HTM                   │
│           └─ [Valid] ↓                                      │
└─────────────────────────────────────────────────────────────┘
                        │
                        ↓
┌─────────────────────────────────────────────────────────────┐
│  IQEDIT2.HTM                                                │
│  ┌───────────────────────────────────────┐                 │
│  │ Inventario en Ubicación                │                 │
│  │ Action: IQSCREEN subroutine           │                 │
│  └───────────────────────────────────────┘                 │
│           │                                                 │
│           ├─ [BACK] → IQEDIT.HTM                           │
│           └─ [MENU] → NBMMEDIT.HTM                         │
└─────────────────────────────────────────────────────────────┘
```

### **Endpoints Utilizados**

| Archivo | URL | Propósito |
|---------|-----|-----------|
| INVPIN.HTM | /INVPIN.HTM | Consulta de parte |
| INVPIN2.HTM | /INVPIN2.HTM | Resultados parte |
| INVIRMEDIT.HTM | /INVIRMEDIT.HTM | Consulta materia prima |
| INVIP2RMEDIT.HTM | /INVIP2RMEDIT.HTM | Resultados RM + IQC |
| IQEDIT.HTM | /IQEDIT.HTM | Búsqueda por ubicación |
| IQEDIT2.HTM | /IQEDIT2.HTM | Resultados ubicación |

---

## 2.5 Inventory Audit Process (IAP)

### **Descripción**
Proceso de auditoría física de inventario: agregar/ajustar inventario en ubicaciones con date codes y tracking completo.

### **Flujo Completo (Versión Modernizada)**

```
┌─────────────────────────────────────────────────────────────┐
│  IAPEDIT1.HTM                                               │
│  ┌───────────────────────────────────────┐                 │
│  │ Paso 1: Ubicación                      │                 │
│  │ Input: LOCTN (max 6 caracteres)      │                 │
│  └───────────────────────────────────────┘                 │
│           │                                                 │
│           ├─ [MENU] → STOCKCHECKEDIT.HTM                   │
│           ├─ [Error] → PALMERROR2.HTM                      │
│           └─ [Valid] ↓                                      │
└─────────────────────────────────────────────────────────────┘
                        │
                        ↓
┌─────────────────────────────────────────────────────────────┐
│  IAPEDIT2.HTM                                               │
│  ┌───────────────────────────────────────┐                 │
│  │ Paso 2: Virtual Warehouse              │                 │
│  │ Input: VWHSE (Yes/No)                 │                 │
│  │ Nota: Sólo para Liberty dropships    │                 │
│  │       (Deshabilitado desde 2019)      │                 │
│  └───────────────────────────────────────┘                 │
│           │                                                 │
│           ├─ [MENU] → STOCKCHECKEDIT.HTM                   │
│           ├─ [VNO] → Skip virtual warehouse                │
│           └─ [VYES] → Activa VWHSE ↓                       │
└─────────────────────────────────────────────────────────────┘
                        │
                        ↓
┌─────────────────────────────────────────────────────────────┐
│  IAPEDIT3.HTM                                               │
│  ┌───────────────────────────────────────┐                 │
│  │ Paso 3: Date Code                      │                 │
│  │ Input: YCODE (YYWW format, 4 dígitos) │                 │
│  │ Example: 2352 = Year 23, Week 52     │                 │
│  └───────────────────────────────────────┘                 │
│           │                                                 │
│           ├─ [MENU] → STOCKCHECKEDIT.HTM                   │
│           ├─ [NEWLOC] → IAPEDIT1.HTM (restart)             │
│           ├─ [Error] → PALMERROR2.HTM                      │
│           └─ [Valid] ↓                                      │
└─────────────────────────────────────────────────────────────┘
                        │
                        ↓
┌─────────────────────────────────────────────────────────────┐
│  IAPEDIT4.HTM                                               │
│  ┌───────────────────────────────────────┐                 │
│  │ Paso 4: Escanear Número de Parte      │                 │
│  │ Input: PARTNO                         │                 │
│  │ Display:                               │                 │
│  │  - BOXNUM (número de caja)            │                 │
│  │  - LOCTOT (total en ubicación)        │                 │
│  └───────────────────────────────────────┘                 │
│           │                                                 │
│           ├─ [MENU] → STOCKCHECKEDIT.HTM                   │
│           ├─ [NEWLOC] → IAPEDIT1.HTM                       │
│           ├─ [Error] → PALMERROREDIT2.HTM                  │
│           └─ [Valid Part] ↓                                 │
└─────────────────────────────────────────────────────────────┘
                        │
                        ↓
┌─────────────────────────────────────────────────────────────┐
│  IAPEDIT5.HTM                                               │
│  ┌───────────────────────────────────────┐                 │
│  │ Paso 5: Cantidad                       │                 │
│  │ Input: QTY (+ o - permitido)          │                 │
│  │ Actions:                               │                 │
│  │  - Escribe a INV file                 │                 │
│  │  - Escribe a TRACK.REC file           │                 │
│  │  - Incrementa BOXNUM                  │                 │
│  │  - Actualiza LOCTOT                   │                 │
│  │  - VWHSE ops si aplica                │                 │
│  └───────────────────────────────────────┘                 │
│           │                                                 │
│           ├─ [MENU] → STOCKCHECK.HTM                       │
│           ├─ [NEWLOC] → IAPEDIT1.HTM                       │
│           └─ [OK] → IAPEDIT4.HTM (loop: siguiente parte)   │
└─────────────────────────────────────────────────────────────┘
```

### **Endpoints Utilizados**

| Archivo | URL | Propósito |
|---------|-----|-----------|
| IAPEDIT1.HTM | /IAPEDIT1.HTM | Entrada de ubicación |
| IAPEDIT2.HTM | /IAPEDIT2.HTM | Virtual warehouse |
| IAPEDIT3.HTM | /IAPEDIT3.HTM | Date code (YYWW) |
| IAPEDIT4.HTM | /IAPEDIT4.HTM | Escanear parte |
| IAPEDIT5.HTM | /IAPEDIT5.HTM | Cantidad y commit |

### **Reglas de Negocio**

1. **Location**: Máximo 6 caracteres
2. **Virtual Warehouse**:
   - Feature deshabilitado desde 2019
   - Sólo para Liberty dropships
   - Si activo: llama PUTVWHSELOC/TAKEVWHSELOC
3. **Date Code**:
   - Formato YYWW (Year + Week)
   - 4 dígitos numéricos
   - Ejemplo: 2352 = 2023, semana 52
4. **Part Number**: Valida contra archivo PARTS
5. **Cantidad**:
   - Soporta valores positivos (agregar)
   - Soporta valores negativos (restar)
   - Escribe a archivos INV y TRACK.REC
6. **Tracking**:
   - Usuario registrado como 'SYS'
   - Cada transacción incrementa BOXNUM
   - LOCTOT acumula total en ubicación

### **Archivos de Base de Datos**

- `PARTS` - Maestro de partes
- `INV` - Registros de inventario
- `TRACK.REC` - Auditoría de transacciones
- `VWHSE` - Virtual warehouse (si aplica)

### **Variables de Sesión**

- `LOCTN` - Ubicación física
- `MCODE` - Código M (interno)
- `VWHSE` - Flag de virtual warehouse
- `YCODE` - Date code YYWW
- `PARTNO`/`PARTNUM` - Número de parte
- `QTY` - Cantidad
- `BOXNUM` - Número de caja (auto-incrementa)
- `LOCTOT` - Total acumulado en ubicación

---

## 2.6 Ship Out (SHIPOUT)

### **Descripción**
Procesamiento de embarques salientes (outbound shipments).

### **Archivos Disponibles**

- **SHIPOUTP1A.HTM** - Ship out (legacy)
- **SHIPOUTP1AEDIT.HTM** - Ship out (modernizado)
- **SHIPOUTPEDIT1.HTM** - Procesamiento principal
- **SHIPOUTPEDIT1_NEW.HTM** - Nueva versión
- **SHIPOUTPEDIT1_OLD.HTM** - Versión archivada
- **SHIPOUTLIBEDIT1.HTM** - Procesamiento específico Liberty dropships

### **Nota**
Proceso especializado para embarques. Los archivos requieren análisis adicional para documentación completa del flujo.

---

## 2.7 Stock Transactions (STOCKTRANS)

### **Descripción**
Sistema completo de gestión de transacciones de stock (entradas/salidas) con interfaz tipo Excel.

### **Componentes Principales**

#### **STOCKTRANS_CREATE.HTM**
- **Propósito**: Crear nuevas transacciones de stock
- **Inputs**:
  - From/To Location
  - Part Number
  - Quantity
  - Reason/Type
- **Actions**: Creates transaction → returns to list

#### **STOCKTRANS_EDIT.HTM**
- **Propósito**: Interfaz completa de gestión (CRUD)
- **Características**:
  - Interfaz tipo Excel grid
  - Tabs para diferentes vistas
  - Sortable/filterable data grid
  - Real-time notifications
  - Operaciones: Save, Edit, Delete
- **Tamaño**: ~1400+ líneas (archivo más grande del módulo)

#### **STOCKTRANS_PREVIEW.HTM**
- **Propósito**: Vista previa antes de confirmar
- **Actions**:
  - CONFIRM → Commits transaction
  - EDIT → Regresa a CREATE
  - CANCEL → Cancela operación

### **Endpoints Utilizados**

| Archivo | URL | Propósito |
|---------|-----|-----------|
| STOCKTRANS_CREATE.HTM | /STOCKTRANS_CREATE.HTM | Crear transacción |
| STOCKTRANS_EDIT.HTM | /STOCKTRANS_EDIT.HTM | Gestión CRUD |
| STOCKTRANS_PREVIEW.HTM | /STOCKTRANS_PREVIEW.HTM | Vista previa |

### **Integración con XML Services**

Utiliza los siguientes servicios XML:
- `CREATE_STK_TRANSACTION.XML` - Crear transacción
- `VALIDATE_STK_CREATE.XML` - Validar antes de crear
- `GET_STKS_TRNS.XML` - Obtener tipos de transacción
- `UPDATE_STKS_TRNS.XML` - Actualizar transacción

---

## 3. REFERENCIA DE ENDPOINTS

### 3.1 Shipping Labels - Local

| Endpoint | Método | Inputs | Outputs | Siguiente |
|----------|--------|--------|---------|-----------|
| /SHIPLCLABELP1EDIT.HTM | GET | ORDNUM | Valida orden | P2EDIT |
| /SHIPLCLABELP2EDIT.HTM | GET | ORDNUM, CUSTNO | Selecciona cliente | P3EDIT |
| /SHIPLCLABELP3EDIT.HTM | GET | Datos de envío | Captura dirección | P4EDIT |
| /SHIPLCLABELP4EDIT.HTM | GET | Todos los datos | Vista previa | P5EDIT |
| /SHIPLCLABELP5EDIT.HTM | GET | Todos los datos | Genera label | P1 o MENU |

### 3.2 Shipping Labels - Inner Box

| Endpoint | Método | Inputs | Outputs | Siguiente |
|----------|--------|--------|---------|-----------|
| /SHIPILABELP1.HTM | GET | PARTNUM | Valida parte | P2 |
| /SHIPILABEL1EDIT.HTM | GET | PARTNUM | Valida parte (alt) | P2EDIT |
| /SHIPILABELP3EDIT.HTM | GET | PARTNUM, QTY | Genera label | MENU |

### 3.3 Stock Checks

| Endpoint | Método | Inputs | Outputs | Siguiente |
|----------|--------|--------|---------|-----------|
| /SHIPCHECKEDIT.HTM | GET | INVNUM | Valida packing list | EDIT2 |
| /SHIPCHECKEDIT2.HTM | GET | INVNUM, PARTNO | Escanea partes | Loop o DONE |
| /SHIPCHECK3.HTM | GET | INVNUM | Genera reporte | COMPLETE |

### 3.4 Inventory Adjustments

| Endpoint | Método | Inputs | Outputs | Siguiente |
|----------|--------|--------|---------|-----------|
| /INVPIN.HTM | GET | PARTNUM | Valida parte | INVPIN2 |
| /INVPIN2.HTM | GET | PARTNUM | Display inventory | MENU |
| /INVIRMEDIT.HTM | GET | PARTNUM | Valida RM parte | INVIP2RMEDIT |
| /INVIP2RMEDIT.HTM | GET | PARTNUM | Display RM + IQC | BACK/CREATE/MENU |
| /IQEDIT.HTM | GET | LOCTN | Busca ubicación | IQEDIT2 |
| /IQEDIT2.HTM | GET | LOCTN | Display inventory | BACK/MENU |

### 3.5 Inventory Audit Process

| Endpoint | Método | Inputs | Outputs | Siguiente |
|----------|--------|--------|---------|-----------|
| /IAPEDIT1.HTM | GET | LOCTN | Valida ubicación | EDIT2 |
| /IAPEDIT2.HTM | GET | LOCTN, VWHSE | Virtual warehouse | EDIT3 |
| /IAPEDIT3.HTM | GET | LOCTN, VWHSE, YCODE | Date code | EDIT4 |
| /IAPEDIT4.HTM | GET | Todos + PARTNO | Escanea parte | EDIT5 |
| /IAPEDIT5.HTM | GET | Todos + QTY | Escribe INV/TRACK | EDIT4 (loop) |

### 3.6 Stock Transactions

| Endpoint | Método | Inputs | Outputs | Siguiente |
|----------|--------|--------|---------|-----------|
| /STOCKTRANS_CREATE.HTM | GET | Transaction data | Creates transaction | LIST |
| /STOCKTRANS_EDIT.HTM | GET | - | CRUD interface | - |
| /STOCKTRANS_PREVIEW.HTM | GET | Transaction data | Preview | CONFIRM/EDIT |

---

## 4. SERVICIOS XML/API

### 4.1 CREATE_STK_TRANSACTION.XML

**Endpoint**: `/inventory/CREATE_STK_TRANSACTION.XML`

**Propósito**: Crear transacción de stock después de validación

**Parámetros**:
```
PARTNUM   - Número de parte
QTY       - Cantidad
MTLTYPE   - Tipo de transacción material
REFERENCE - Referencia
NOTES     - Notas
WHBIN     - Bin de warehouse
YYMM      - Date code (año/mes)
```

**Archivos de Base de Datos**:
- INV - Inventario
- LIID - Generador de IDs
- MTLTRANS - Tipos de transacciones
- STOCK - Transacciones de stock

**Proceso**:
1. Valida tipo de transacción (MTLTRANS)
2. Lee dirección (IN/OUT) de transacción
3. Construye registro STK con 26 campos
4. Genera LIID (Stock ID) secuencial
5. Escribe a STOCK database
6. Ejecuta programa STOCKER con modo "M"

**Response XML**:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<CreateStockResult>
  <status>success|error</status>
  <message>error message</message>
  <stockId>generated ID</stockId>
  <partNumber>PARTNUM</partNumber>
  <quantity>QTY</quantity>
  <transactionType>MTLTYPE</transactionType>
  <direction>IN|OUT</direction>
  <createdBy>INITIALS</createdBy>
  <createdDate>DATE</createdDate>
  <createdTime>TIME</createdTime>
</CreateStockResult>
```

---

### 4.2 GET_STKS_TRNS.XML

**Endpoint**: `/inventory/GET_STKS_TRNS.XML`

**Propósito**: Obtener todos los tipos de transacciones de stock

**Parámetros**: Ninguno (opcional CODE para filtrado)

**Archivos de Base de Datos**:
- MTLTRANS - Tipos de transacciones

**Proceso**:
1. SSELECT MTLTRANS (todos los registros)
2. Para cada registro extrae:
   - Code (clave del registro)
   - Description (campo 3)
   - Type (campo 6)
   - Reference (campo 7)
   - DateCode (campo 8)
   - Notes (campo 9)

**Response XML**:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<TransactionTypes>
  <TransactionType>
    <Code>código</Code>
    <Description>descripción</Description>
    <Type>tipo</Type>
    <Reference>referencia</Reference>
    <DateCode>datecode</DateCode>
    <Notes>notas</Notes>
  </TransactionType>
  ...
</TransactionTypes>
```

---

### 4.3 GET_STOCKS2.XML

**Endpoint**: `/inventory/GET_STOCKS2.XML`

**Propósito**: Obtener información de stocks/transacciones

**Parámetros**: CODE (opcional)

**Archivos de Base de Datos**:
- MTLTRANS

**Proceso**:
1. SSELECT MTLTRANS
2. Para cada registro:
   - Code
   - Description (campo 3)
   - Type (campo 6)
   - Opers (campo 7 - multivalue)
     - Incluye conversión TOPERS para nombres

**Response XML**:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<stocktrans>
  <stock>
    <code>código</code>
    <description>descripción</description>
    <type>tipo</type>
    <opers>
      <user>iniciales</user>
      <user_name>nombre completo</user_name>
      ...
    </opers>
  </stock>
  ...
</stocktrans>
```

---

### 4.4 UPDATE_STKS_TRNS.XML

**Endpoint**: `/inventory/UPDATE_STKS_TRNS.XML`

**Propósito**: Actualizar registro de transacción en MTLTRANS

**Parámetros**:
```
CODE        - Código (required)
DESCRIPTION - Descripción
TYPE        - Tipo (IN/OUT)
REFERENCE   - Referencia
DATECODE    - Date code
NOTES       - Notas
```

**Archivos de Base de Datos**:
- MTLTRANS

**Proceso**:
1. Valida CODE requerido
2. Lee registro existente o crea nuevo
3. Actualiza campos si parámetro provisto:
   - Campo 3 = DESCRIPTION
   - Campo 6 = TYPE
   - Campo 7 = REFERENCE
   - Campo 8 = DATECODE
   - Campo 9 = NOTES
4. Escribe registro actualizado

**Response XML**:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<update>
  <status>success|error</status>
  <transaction>
    <Code>código</Code>
    <Description>descripción</Description>
    <Type>tipo</Type>
    <Reference>referencia</Reference>
    <DateCode>datecode</DateCode>
    <Notes>notas</Notes>
  </transaction>
</update>
```

---

### 4.5 UPDATE_STOCKS.XML

**Endpoint**: `/inventory/UPDATE_STOCKS.XML`

**Propósito**: Gestión completa de usuarios en transacciones (agregar/eliminar)

**Parámetros**:
```
CODE         - Código transacción (required)
DESCRIPTION  - Descripción (required para no-DELETE)
INITIALS     - Iniciales usuario (2 letras, required)
STATUS       - IN/OUT/DELETE
NAME         - Nombre usuario (opcional)
INITIALS_ALL - Bulk rewrite de lista completa (opcional)
```

**Archivos de Base de Datos**:
- MTLTRANS
- XMLDATA (para logging)

**Proceso**:

**Para STATUS = DELETE**:
1. Busca y elimina INITIALS del campo 7 (multivalue)
2. Si campo 7 queda vacío → DELETE record completo
3. Si aún hay usuarios → WRITE record actualizado

**Para STATUS = IN/OUT**:
1. Set campo 3 = DESCRIPTION (sólo si record nuevo)
2. Set campo 6 = STATUS
3. Agrega INITIALS a campo 7 si no existe
4. WRITE record

**Con INITIALS_ALL**:
1. Parsea string completo de iniciales
2. Extrae pares de 2 letras
3. Reescribe campo 7 completo
4. Si queda vacío → DELETE record

**Normalización**:
- Iniciales convertidas a uppercase
- Solo primeras 2 letras alfabéticas
- Deduplicación automática

**Logging**:
- Escribe a UPD_STOCKS.LOG
- Formato: DATE TIME | STATUS | KEY | PARAMS

**Response XML**:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<stocktrans>
  <status>success</status>
  <stock>
    <code>código</code>
    <description>descripción</description>
    <type>IN|OUT</type>
    <opers>
      <user>iniciales</user>
      <user_name>nombre</user_name>
      ...
    </opers>
  </stock>
</stocktrans>
```

---

### 4.6 UPDATE_STOCKS2.XML

**Endpoint**: `/inventory/UPDATE_STOCKS2.XML`

**Propósito**: Versión simplificada de UPDATE_STOCKS

**Parámetros**:
```
CODE        - Código (required)
DESCRIPTION - Descripción (required)
INITIALS    - Iniciales (required)
STATUS      - IN/OUT/DELETE (default: IN)
NAME        - Nombre (opcional)
INITIALS_ALL - Bulk rewrite (opcional)
```

**Diferencias con UPDATE_STOCKS**:
- Normalización más estricta de iniciales (solo 2 letras)
- Limpieza automática de campo 7
- Deduplicación por token VM
- Soporte para bulk rewrite completo

**Response**: Similar a UPDATE_STOCKS.XML

---

### 4.7 VALIDATE_STK_CREATE.XML

**Endpoint**: `/inventory/VALIDATE_STK_CREATE.XML`

**Propósito**: Validar transacción antes de crearla (pre-flight check)

**Parámetros**:
```
PARTNUM   - Número de parte (required)
QTY       - Cantidad (required, > 0)
MTLTYPE   - Tipo transacción (required)
REFERENCE - Referencia
NOTES     - Notas
WHBIN     - Warehouse bin
YYMM      - Date code (4 dígitos)
```

**Archivos de Base de Datos**:
- INV - Inventario
- TABLE - Configuración
- PARTS - Maestro de partes
- MTLTRANS - Tipos de transacciones

**Validaciones**:

1. **Physical Inventory Check**:
   - Lee flag STOPIPAQ de TABLE
   - Si = 1 → Error: "PHYSICAL INVENTORY IN PROGRESS"

2. **Required Fields**:
   - PARTNUM no vacío
   - QTY numérico y > 0
   - MTLTYPE no vacío

3. **YYMM Format**:
   - Si provisto: 4 dígitos numéricos

4. **Part Number**:
   - Debe existir en PARTS file
   - Error: "INVALID PART NUMBER - {PARTNUM} NOT FOUND"

5. **Transaction Type**:
   - Debe existir en MTLTRANS
   - Error: "INVALID TRANSACTION TYPE - {MTLTYPE} NOT FOUND"

6. **Warehouse Bin (para OUT transactions)**:
   - Bin debe existir en INV para PARTNUM
   - Verifica cantidad disponible
   - Calcula balance proyectado
   - Error si QTY > disponible

**Response XML**:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<ValidationResult>
  <status>success|error</status>
  <message>error message</message>
  <ValidatedData>
    <partNumber>PARTNUM</partNumber>
    <partDescription>descripción</partDescription>
    <quantity>QTY</quantity>
    <transactionType>MTLTYPE</transactionType>
    <transactionDescription>descripción</transactionDescription>
    <transactionDirection>IN|OUT</transactionDirection>
    <reference>REFERENCE</reference>
    <notes>NOTES</notes>
    <warehouseBin>WHBIN</warehouseBin>
    <dateCode>YYMM</dateCode>
    <initials>INITIALS</initials>
    <availableQty>cantidad disponible</availableQty>
    <projectedBalance>balance después</projectedBalance>
  </ValidatedData>
</ValidationResult>
```

---

## 5. VARIABLES DE SESIÓN

### 5.1 Autenticación

| Variable | Uso | Tipo |
|----------|-----|------|
| INIT | Cookie de autenticación (iniciales usuario) | Cookie |
| INITIALS | Iniciales del usuario logueado | Session |

### 5.2 Shipping Labels - Local

| Variable | Descripción | Tipo |
|----------|-------------|------|
| ORDNUM | Número de orden de venta | String |
| CUSTNO | Número de cliente | String |
| SHIP_NAME | Nombre destinatario | String |
| SHIP_ADDR1-4 | Líneas de dirección | String |
| SHIP_CITY | Ciudad | String |
| SHIP_STATE | Estado | String |
| SHIP_ZIP | Código postal | String |
| CONTACT | Persona de contacto | String |
| PHONE | Teléfono | String |
| PKGS | Número de paquetes | Number |
| WT | Peso | Number |

### 5.3 Shipping Labels - Inner Box

| Variable | Descripción | Tipo |
|----------|-------------|------|
| PARTNUM | Número de parte (sin 'P') | String |
| QTY | Cantidad | Number |

### 5.4 Stock Checks

| Variable | Descripción | Tipo |
|----------|-------------|------|
| INVNUM | Número de packing list (sin '11K') | String |
| PARTNO | Número de parte (sin '1P') | String |

### 5.5 Inventory Audit

| Variable | Descripción | Tipo |
|----------|-------------|------|
| LOCTN | Ubicación (max 6 chars) | String |
| MCODE | Código M interno | String |
| VWHSE | Virtual warehouse flag | String |
| YCODE | Date code YYWW | String(4) |
| PARTNO/PARTNUM | Número de parte | String |
| QTY | Cantidad (+ o -) | Number |
| BOXNUM | Número de caja (auto) | Number |
| LOCTOT | Total en ubicación (auto) | Number |

### 5.6 Stock Transactions

| Variable | Descripción | Tipo |
|----------|-------------|------|
| MTLTYPE | Tipo de transacción | String |
| REFERENCE | Referencia | String |
| NOTES | Notas | String |
| WHBIN | Warehouse bin | String |
| YYMM | Date code año/mes | String(4) |

---

## 6. BASE DE DATOS

### 6.1 Archivos Pick/BASIC

| Archivo | Propósito | Campos Clave |
|---------|-----------|--------------|
| PARTS | Maestro de partes | Part number, descripción, specs |
| INV | Registros de inventario | Part, location, quantity, datecode |
| SHIPMENTS | Datos de embarques | Invoice, parts, quantities |
| SO | Órdenes de venta | Order number, customer, items |
| SHIPSCAN | Registro de escaneos | Scan timestamp, part, invoice |
| TRACK.REC | Auditoría de transacciones | User, date, part, qty, location |
| VWHSE | Virtual warehouse | Dropship data (Liberty) |
| UPSADD | Direcciones UPS shipping | Shipping addresses, labels |
| MTLTRANS | Tipos de transacciones | Type code, description, IN/OUT |
| STOCK | Transacciones de stock | Stock ID, part, qty, type, date |
| LIID | Generador de IDs | Counters for auto-increment |
| TABLE | Configuración sistema | Flags, settings (e.g., STOPIPAQ) |

### 6.2 Estructura MTLTRANS

```
Campo 1: [Reserved]
Campo 2: [Reserved]
Campo 3: Description (descripción de transacción)
Campo 6: Type (IN/OUT)
Campo 7: Reference / Users (multivalue)
Campo 8: DateCode
Campo 9: Notes
```

### 6.3 Estructura STOCK (STK)

```
Campo 1:  Transaction date
Campo 2:  MTLTYPE + REFERENCE
Campo 3:  Part number (uppercase)
Campo 4:  Quantity
Campo 5:  Quantity (duplicate)
Campo 6:  Transaction description (from MTLTRANS<1>)
Campo 7:  Transaction type description (from MTLTRANS<2>)
Campo 9:  Notes
Campo 10: DATE*INITIALS*TIME
Campo 12: Status flag ('1')
Campo 26: WHBIN + YYMM (warehouse bin + datecode)
```

---

## 7. GLOSARIO

### Términos Técnicos

| Término | Definición |
|---------|------------|
| **PicLan-IP/BASIC** | Lenguaje de programación server-side embebido en HTML vía tags `<pre>` |
| **BASIC** | Business Application System Integrated Computer (Pick/MultiValue DB) |
| **BOH** | Balance on Hand (cantidad disponible en inventario) |
| **IQC** | Incoming Quality Control (control de calidad de entrada) |
| **YYWW** | Year + Week format (ej: 2352 = año 2023, semana 52) |
| **YYMM** | Year + Month format (ej: 2312 = diciembre 2023) |
| **Date Code** | Código que identifica cuándo fue fabricada o recibida una parte |

### Prefijos de Códigos de Barras

| Prefijo | Significado | Acción |
|---------|-------------|--------|
| **P** | Part Number | Se elimina automáticamente (ej: P12345 → 12345) |
| **1P** | Part Number (shipping) | Se elimina en SHIPCHECKEDIT2 |
| **11K** | Packing List | Se elimina en SHIPCHECKEDIT |

### Tipos de Archivos

| Extensión | Tipo | Propósito |
|-----------|------|-----------|
| **.HTM** | HTML + PicLan-IP/BASIC | Páginas web con backend embebido |
| **.XML** | PicLan-IP/BASIC XML | Servicios/API que retornan XML |
| **EDIT.HTM** | Modern UI | Versión modernizada con mejor UX |
| **sin EDIT** | Legacy | Versiones antiguas del sistema |

### Operaciones de Base de Datos

| Comando | Función |
|---------|---------|
| **PL_GETVAR** | Obtener variable de request/session |
| **PL_PUTVAR** | Guardar variable en session |
| **PL_GET_COOKIE** | Leer cookie (autenticación) |
| **READ** | Leer registro de archivo |
| **WRITE** | Escribir registro a archivo |
| **DELETE** | Eliminar registro de archivo |
| **READVU** | Read value update (field level lock) |
| **WRITEVU** | Write value update (field level) |
| **SSELECT** | Select all records from file |
| **READNEXT** | Read next record in select list |
| **EXECUTE** | Execute Pick/BASIC program |

### Convenciones de Navegación

| Botón | Acción Común |
|-------|--------------|
| **CANCEL** | Regresar a NBMMEDIT.HTM |
| **MENU** / **Main Menu** | Regresar a menú principal |
| **BACK** | Página anterior en flujo |
| **NEXT** | Siguiente paso en flujo |
| **OK** | Confirmar y continuar |
| **NEWLOC** | Nueva ubicación (reinicia flujo IAP) |

### Estados de Virtual Warehouse

| Estado | Descripción |
|--------|-------------|
| **VYES** | Habilita virtual warehouse |
| **VNO** | Deshabilita virtual warehouse |
| **PUTVWHSELOC** | Rutina: agregar a virtual warehouse |
| **TAKEVWHSELOC** | Rutina: remover de virtual warehouse |

---

## APÉNDICE A: PATRONES COMUNES

### A.1 Patrón de Autenticación

Todos los archivos .HTM/.XML verifican autenticación:

```basic
PL_GET_COOKIE INITIALS FROM 'INIT' ELSE
   ERR = '' ; INITIALS = ''
   CALL PLW.PAGE('LOGINWH.HTM','',ERR)
   RETURN
END
```

### A.2 Patrón de Limpieza de Part Number

```basic
' Strip leading 'P' if present
IF PARTNUM[1,1] = 'P' THEN
   PARTNUM = PARTNUM[2,99]
END
```

```basic
' Strip '1P' prefix (shipping scan)
IF PARTNO[1,2] = '1P' THEN
   PARTNO = PARTNO[3,99]
END
```

### A.3 Patrón de Validación de Parte

```basic
OPEN '','PARTS' TO PARTSF ELSE
   ' Error handling
END
READ PREC FROM PARTSF,PARTNUM ELSE
   ' Part not found
   CALL PLW.PAGE('PALMERROR2EDIT.HTM','',ERR)
   RETURN
END
```

### A.4 Patrón de Redirección

```basic
IF CANCEL NE "" THEN
   ERR = ''
   CALL PLW.PAGE('NBMMEDIT.HTM','',ERR)
   RETURN
END
```

### A.5 Patrón de Uppercase Conversion

```basic
WHBIN = OCONV(WHBIN,'MCU')
```

### A.6 Patrón de Entities HTML en BASIC

```basic
' En código BASIC dentro de <pre>, usar &quot; en lugar de "
IF CANCEL3 NE &quot;&quot; THEN
   CALL PLW.PAGE('NBMMEDIT.HTM','',ERR)
END
```

---

## APÉNDICE B: MATRIZ DE NAVEGACIÓN

### Shipping Labels - Local

| Desde | CANCEL → | NEXT/OK → | Error → |
|-------|----------|-----------|---------|
| P1EDIT | NBMMEDIT | P2EDIT | PALMERROR2 |
| P2EDIT | NBMMEDIT | P3EDIT | - |
| P3EDIT | NBMMEDIT | P4EDIT | - |
| P4EDIT | NBMMEDIT | P5EDIT (CONFIRM) | P3EDIT (EDIT) |
| P5EDIT | NBMMEDIT | P1EDIT (ANOTHER) | - |

### Inventory Audit Process

| Desde | MENU → | NEWLOC → | Next → |
|-------|--------|----------|--------|
| IAPEDIT1 | STOCKCHECKEDIT | - | IAPEDIT2 |
| IAPEDIT2 | STOCKCHECKEDIT | - | IAPEDIT3 |
| IAPEDIT3 | STOCKCHECKEDIT | IAPEDIT1 | IAPEDIT4 |
| IAPEDIT4 | STOCKCHECKEDIT | IAPEDIT1 | IAPEDIT5 |
| IAPEDIT5 | STOCKCHECK | IAPEDIT1 | IAPEDIT4 (loop) |

### Stock Checks

| Desde | MENU → | BACK → | Action → |
|-------|--------|--------|----------|
| SHIPCHECKEDIT | NBMMEDIT | - | SHIPCHECKEDIT2 |
| SHIPCHECKEDIT2 | NBMMEDIT | SHIPCHECKEDIT | Loop (OK) / SHIPCHECK3 (DONE) |
| SHIPCHECK3 | - | SHIPCHECK2 | SHIPCHECK (COMPLETE) |

---

## APÉNDICE C: DIAGRAMAS DE FLUJO ASCII

### Flujo Completo IAP (Modernizado)

```
START
  │
  ├─→ IAPEDIT1: Captura LOCTN (max 6 chars)
  │      │
  │      ├─ [MENU] → Exit to STOCKCHECKEDIT
  │      └─ [Valid] ↓
  │
  ├─→ IAPEDIT2: Virtual Warehouse?
  │      │
  │      ├─ [VYES] → Set VWHSE flag
  │      ├─ [VNO] → Skip VWHSE
  │      └─ [Continue] ↓
  │
  ├─→ IAPEDIT3: Captura YCODE (YYWW format)
  │      │
  │      ├─ [NEWLOC] → Back to IAPEDIT1
  │      └─ [Valid] ↓
  │
  ├─→ IAPEDIT4: Escanear PARTNO ◄─┐
  │      │                         │
  │      ├─ [NEWLOC] → IAPEDIT1   │
  │      └─ [Valid Part] ↓         │
  │                                 │
  ├─→ IAPEDIT5: Captura QTY        │
  │      │                         │
  │      ├─ Write to INV           │
  │      ├─ Write to TRACK.REC     │
  │      ├─ Increment BOXNUM       │
  │      ├─ Update LOCTOT          │
  │      └─ [OK] ─────────────────┘
  │      │
  │      └─ [NEWLOC] → IAPEDIT1
  │
END
```

---

## NOTAS FINALES

### Mantenimiento del Manual

- **Actualizar** cuando se agreguen nuevos archivos .HTM
- **Versionar** cambios en flujos de proceso
- **Documentar** nuevos servicios XML
- **Revisar** periódicamente reglas de negocio

### Convenciones de Naming

- **P[N]EDIT** - Paso N del proceso (modernizado)
- **P[N]** - Paso N del proceso (legacy)
- **EDIT** - Versión moderna con mejor UI
- **Sin EDIT** - Versión legacy

### Skill de TAIMEX

Este manual complementa el skill de TAIMEX ubicado en:
`.claude/skills/taimex-templates/skill.md`

Para desarrollo de nuevas páginas, consultar el skill para:
- Estándares de diseño
- Patrones de código
- Bugs conocidos
- Mejores prácticas

---

**Fin del Manual de Operaciones - Módulo Inventory**

*Documento generado el 2025-11-16*
*Versión 1.0*
