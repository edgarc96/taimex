---
description: TAIMEX HTML template standards and patterns. Use when working with .HTM files, creating templates, fixing bugs, or modernizing legacy pages. (project, gitignored)
---

# TAIMEX Frontend Template Guidelines

You are working on the TAIMEX project. This skill provides the standard approach for creating, editing, and maintaining HTML frontend templates.

## When This Skill Activates

This skill should be active whenever you:
- Create or edit any `.HTM` or `.HTML` file
- Work on templates with PicLan-IP/BASIC backend code
- Fix bugs in existing templates
- Modernize legacy templates
- Receive requests about "plantillas", "templates", "forms", or "páginas HTML"

## Quick Reference Card

### Essential Form Elements
- **Form action**: Always use "EDIT" suffix → `/PAGEEDIT.HTM`
- **Primary button**: `name="OK"` `tabindex="2"`
- **Cancel button**: `name="CANCEL#"` (match backend - CANCEL2, CANCEL3, CANCEL4, etc.) `tabindex="3"`
- **Input pattern**: `id="lowercase"` `name="UPPERCASE"` `value="^UPPERCASE^"`

### Common Issues Checklist
- ✅ Sidebar close handler included?
- ✅ Auto-focus JavaScript added?
- ✅ All redirects use "EDIT" suffix?
- ✅ Template variables use `^VAR^` syntax?
- ✅ Backend PicLan-IP/BASIC preserved?
- ✅ Backend quotes use `&quot;&quot;` not `""`?
- ✅ Tab index sequence correct?
- ✅ No duplicate validations? (Check backend first!)

## Template Structure

All HTML templates in this project follow a hybrid structure:
1. **Modern HTML/CSS Frontend** - User-facing interface with modern styling
2. **PicLan-IP/BASIC Backend** - Server-side logic embedded in `<PRE>` tags at the bottom

## Standard Template Anatomy

### 1. Document Head
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>TAIMEX - [Page Title]</title>
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
  <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
```

### 2. CSS Variables (Always Include)
```css
:root {
  --bg-body: #f1f5f9;
  --bg-card: #ffffff;
  --accent-blue: #1d4ed8;
  --accent-green: #10b981;
  --text-primary: #0f172a;
  --text-secondary: #475569;
  --shadow-card: 0 18px 46px rgba(15, 23, 42, 0.18);
  --radius-lg: 20px;
}
```

### 3. Standard Layout Components

#### Header
```html
<header class="header">
  <div class="header-left">
    <label class="hamburger-menu" for="sb-toggle" aria-label="Open menu">☰</label>
    <div class="logo-section">
      <img src="http://54.189.226.145/uploads/assets/images/taimex_logo.png" alt="TAIMEX INDUSTRIAL">
    </div>
  </div>
  <div class="title-section">
    <h2>TAIMEX - [Page Title]</h2>
    <p>[Subtitle]</p>
  </div>
</header>
```

#### Sidebar Menu
```html
<input type="checkbox" id="sb-toggle">
<div class="sidebar-overlay"></div>

<aside class="sidebar" aria-label="Main menu">
  <div class="sidebar-header">
    <h3>Menu</h3>
  </div>
  <div class="sidebar-content">
    <ul class="sidebar-menu">
      <li><label>Main Menu</label></li>
      <!-- Add menu items here -->
      <!-- IMPORTANT: Do NOT add for="sb-toggle" to menu labels -->
      <!-- JavaScript handles closing via event listeners -->
    </ul>
  </div>
</aside>
```

### 4. Form Structure

#### Basic Form Pattern
```html
<form method="GET" action="/YOURPAGE.HTM" class="form">
  <div class="form-group">
    <label for="fieldname">Field Label</label>
    <input
      type="text"
      id="fieldname"
      name="FIELDNAME"
      value="^FIELDNAME^"
      autocomplete="off"
      inputmode="text"
      tabindex="1"
      required
    >
  </div>

  <div class="form-actions">
    <button type="submit" name="OK" value="OK" class="btn btn-primary" tabindex="2">
      OK
    </button>
    <button type="submit" name="CANCEL2" value="Cancel" class="btn btn-secondary" tabindex="3">
      Cancel
    </button>
  </div>

  <!-- Hidden fields for backend processing -->
  <input type="hidden" name="FIELDNAME" value="^FIELDNAME^">
</form>
```

### 5. JavaScript Patterns

#### Auto-Focus First Field
```javascript
<script>
  (function() {
    'use strict';

    // Auto-focus first input field
    var firstField = document.getElementById('fieldname');
    if (firstField) {
      firstField.focus();
      if (firstField.value.trim() !== '' && firstField.value !== '^FIELDNAME^') {
        firstField.select();
      }
    }

    // Sidebar overlay click handler
    var overlay = document.querySelector('.sidebar-overlay');
    var checkbox = document.getElementById('sb-toggle');

    if (overlay && checkbox) {
      overlay.addEventListener('click', function() {
        checkbox.checked = false;
      });
    }

    // Close sidebar when clicking menu items
    if (checkbox) {
      var menuLabels = document.querySelectorAll('.sidebar-menu label');
      menuLabels.forEach(function(label) {
        label.addEventListener('click', function() {
          checkbox.checked = false;
        });
      });
    }

    // Form validation
    var form = document.querySelector('form');
    if (form) {
      form.addEventListener('submit', function(e) {
        if (firstField && firstField.value.trim() === '') {
          e.preventDefault();
          alert('Please enter a value');
          firstField.focus();
        }
      });
    }
  })();
</script>
```

### 6. Backend Logic (PicLan-IP/BASIC)

Always placed at the bottom in `<PRE>` tags:

```html
<PRE>
PicLan-IP/BASIC ^ ^
*
 PL_GET_COOKIE INITIALS FROM 'INIT' ELSE
    ERR = '' ; INITIALS=  ''
    CALL PLW.PAGE('LOGINWH.HTM','',ERR)
    RETURN
 END
*
PL_GETVAR FIELDNAME FROM 'FIELDNAME' ELSE FIELDNAME = ''
PL_GETVAR CANCEL2 FROM 'CANCEL2' ELSE CANCEL2 = ''
*
IF CANCEL2 NE "" THEN
   ERR = ''
   CALL PLW.PAGE('PREVIOUSPAGE.HTM','',ERR)
   RETURN
END
*
IF FIELDNAME NE "" THEN
   * Process the data here
   ERR = ''
   CALL PLW.PAGE('NEXTPAGE.HTM','',ERR)
   RETURN
END
</PRE>
```

## Key Conventions

### Variable Naming
- **Frontend**: Use lowercase with hyphens for IDs/classes (`part-number`, `submit-btn`)
- **Backend**: Use UPPERCASE for form field names (`PARTNUM`, `INVNUM`)
- **Template Variables**: Use caret notation (`^PARTNUM^`, `^INVNUM^`)

### Form Field Pattern
1. Visible input with `name="FIELDNAME"` and `id="fieldname"`
2. Value set to `^FIELDNAME^` for template replacement
3. Hidden input with same name/value for backend processing

### Button Naming
- Primary action: `name="OK"` or descriptive name like `name="PRINT"`
- Cancel action: `name="CANCEL#"` where # is dynamic (CANCEL2, CANCEL3, CANCEL4, etc.)
  - **CRITICAL**: Match the cancel button name to what the backend PicLan-IP/BASIC expects
  - Check backend code: `PL_GETVAR CANCEL3 FROM 'CANCEL3'` means use `name="CANCEL3"`
  - **DO NOT** force all cancel buttons to be CANCEL2

### Tab Index
1. First input field: `tabindex="1"`
2. Primary button: `tabindex="2"`
3. Cancel button: `tabindex="3"`

## CSS Classes Reference

### Layout
- `.container` - Main wrapper
- `.header` - Top navigation bar
- `.main-content` - Content area
- `.card` - Content card container

### Form Elements
- `.form` - Form wrapper
- `.form-group` - Input group (label + input)
- `.form-actions` - Button container
- `.btn` - Base button class
- `.btn-primary` - Primary action button (blue)
- `.btn-secondary` - Secondary action button (gray)

### Informational
- `.info-box` - Green instruction box
- `.card-header` - Card title section

### Sidebar
- `.sidebar` - Sidebar menu
- `.sidebar-overlay` - Backdrop overlay
- `.sidebar-menu` - Menu list

## Common Patterns

### Scan/Input Field Page
Used for: Part scanning, invoice entry, etc.
- Single input field with auto-focus
- OK and Cancel buttons
- Green info box with instructions
- Backend validates and redirects

### List/Grid Page
Used for: Displaying records, search results, etc.
- Table or grid layout
- Filters at top
- Action buttons per row
- Pagination if needed

### Edit/Detail Page
Used for: Editing records, detailed views
- Multiple form fields
- Save and Cancel buttons
- Validation before submit
- Backend updates database

## Example: Creating a New Page

1. **Copy base structure** from existing template (e.g., `SHIPILABEL1EDIT.HTM`)
2. **Update page-specific elements**:
   - Title in `<title>` and `.title-section h2`
   - Card header text in `.card-header`
   - Instructions in `.info-box`
3. **Modify form fields** for your data
4. **Update JavaScript** variable names and validation
5. **Write backend logic** in `<PRE>` section
6. **Test flow**: Input → Validation → Processing → Redirect

## Important Notes

- **Never remove** the PicLan-IP/BASIC section at the bottom
- **Always maintain** the `^VARIABLE^` template syntax for backend values
- **Keep consistent** styling with existing pages
- **Test auto-focus** and tab navigation
- **Verify** form submission flows to correct next page

## File Organization

```
project-root/
├── inventory/     # Inventory/shipping related pages
├── po/           # Purchase order pages
├── so/           # Sales order pages
├── parts/        # Parts management
├── customer/     # Customer management
├── vendor/       # Vendor management
├── login/        # Authentication
├── menu/         # Main menus
└── styles/       # Shared stylesheets (if any)
```

## Maintenance

When updating templates:
1. Check all form field names match backend expectations
2. Verify tabindex sequence is correct
3. Test JavaScript auto-focus behavior
4. Ensure cancel button redirects to correct page
5. Validate template variable replacement (`^VAR^`)

## Common Bugs & Fixes

### ❌ BUG: Sidebar No Cierra al Hacer Clic en Menu Items

**Problema**: El sidebar permanece abierto después de hacer clic en un item del menú.

**Causa**: Los labels del menú tienen `for="sb-toggle"` que causa un comportamiento de toggle (abrir/cerrar) en lugar de solo cerrar. También falta el event listener para cerrar el sidebar.

**Solución**: Siempre agregar este código después del overlay click handler:

```javascript
// Close sidebar when clicking menu items
if (checkbox) {
  var menuLabels = document.querySelectorAll('.sidebar-menu label');
  menuLabels.forEach(function(label) {
    label.addEventListener('click', function() {
      checkbox.checked = false;
    });
  });
}
```

**IMPORTANTE**:
- Este patrón DEBE incluirse en TODOS los templates con sidebar para garantizar UX correcta.
- El `checkbox` debe estar declarado FUERA del bloque `if (overlay && checkbox)` para que esté disponible en el scope del event listener de menuLabels.
- El overlay debe ser un `<div>` (NO un `<label>`) para que JavaScript maneje el cierre correctamente sin comportamiento de toggle.

### ❌ BUG: Error 500 en PicLan-IP/BASIC - Comillas Dobles

**Problema**: El servidor devuelve "500 INTERNAL ERROR" al cargar la página.

**Causa**: Usar comillas dobles directas `""` en el código PicLan-IP/BASIC dentro de HTML causa errores de parsing.

**Solución**: SIEMPRE usar HTML entities `&quot;` en lugar de comillas dobles `"` en el código BASIC:

```basic
❌ INCORRECTO:
IF CANCEL2 NE "" THEN
IF DCODE NE "" THEN

✅ CORRECTO:
IF CANCEL2 NE &quot;&quot; THEN
IF DCODE NE &quot;&quot; THEN
```

**IMPORTANTE**:
- Este error es CRÍTICO y causa que la página no cargue
- SIEMPRE revisar el código backend al modernizar templates legacy
- Buscar todas las instancias de `""` y reemplazar con `&quot;&quot;`
- Aplicar esta regla a TODAS las comparaciones de strings vacíos en PicLan-IP/BASIC

### ⚠️ PATRÓN: No Duplicar Validaciones del Backend en JavaScript

**Problema**: Agregar validaciones JavaScript cuando el backend ya tiene validaciones implementadas puede causar conflictos, bloqueos innecesarios, o comportamiento inconsistente.

**Regla**: Si el backend PicLan-IP/BASIC ya tiene validaciones, **NO las repliques en JavaScript**.

**Razón**:
- El backend ya maneja los errores correctamente (ej. redirigiendo a páginas de error)
- Duplicar validaciones crea mantenimiento doble
- Las validaciones del frontend pueden bloquear casos válidos que el backend manejaría correctamente
- La lógica de negocio debe vivir en UN solo lugar (el backend)

**Solución**: Dejar que el formulario se envíe libremente al backend:

```javascript
❌ INCORRECTO (validación duplicada):
form.addEventListener('submit', function(e) {
  var submitButton = e.submitter;

  if (submitButton && submitButton.name === 'OK') {
    if (dcodeField.value.trim() === '' || dcodeField.value === '^DCODE^') {
      e.preventDefault();
      alert('Please enter a Date Code');
      return;
    }

    // Validando formato que el backend ya valida
    var dcodeValue = dcodeField.value.trim();
    if (dcodeValue.length !== 4 || !/^\d{4}$/.test(dcodeValue)) {
      e.preventDefault();
      alert('Date Code must be 4 digits in YYWW format');
      return;
    }
  }
});

✅ CORRECTO (dejar que el backend valide):
form.addEventListener('submit', function(e) {
  // El formulario se envía directamente
  // El backend valida en PicLan-IP/BASIC:
  // - IF LEN(DCODE) = 4 THEN ... ELSE error
  // - CALL PLW.PAGE('PALMERROR2.HTM','',ERR)
});
```

**Cuándo SÍ agregar validaciones JavaScript**:
- Solo para mejorar UX básica (ej. evitar enviar formularios completamente vacíos)
- Cuando NO exista validación equivalente en el backend
- Para feedback visual inmediato (ej. resaltar campos requeridos)

**Cuándo NO agregar validaciones JavaScript**:
- Si el backend ya tiene la validación implementada
- Si el backend redirige a páginas de error específicas (ej. PALMERROR2.HTM)
- Si no estás 100% seguro de la lógica de validación

**Ejemplo de código backend que YA valida**:
```basic
IF DCODE NE &quot;&quot; THEN
   IF LEN(DCODE) = 4 THEN
      NULL
   END ELSE
      MSG = 'DATE CODE MUST BE IN FORMAT YYWW'
      PL_PUTVAR MSG IN 'MSG' ELSE NULL
      CALL PLW.PAGE('PALMERROR2.HTM','',ERR)
      RETURN
   END
```

Si ves este patrón en el backend, **NO agregues validación JavaScript equivalente**.

**IMPORTANTE**:
- Siempre revisar el código PicLan-IP/BASIC antes de agregar validaciones JavaScript
- La regla de oro: **Una sola fuente de verdad** (Single Source of Truth)
- Cuando tengas duda, deja que el backend maneje las validaciones

### ⚠️ PATRÓN: Nombre Dinámico del Botón Cancel

**Regla CRÍTICA**: El nombre del botón cancel es **dinámico** y debe coincidir **exactamente** con lo que el backend PicLan-IP/BASIC espera.

**Problema**: Cambiar todos los botones cancel a `CANCEL2` rompe el flujo cuando el backend espera `CANCEL3`, `CANCEL4`, etc.

**Solución**: Siempre revisar el código backend ANTES de definir el nombre del botón:

```basic
❌ INCORRECTO (forzar CANCEL2 sin revisar backend):
<button name="CANCEL2" ...>Cancel</button>

Backend espera:
PL_GETVAR CANCEL3 FROM 'CANCEL3' ELSE CANCEL3 = ''
IF CANCEL3 NE "" THEN ...

Resultado: El botón Cancel NO funciona ❌

✅ CORRECTO (match con el backend):
Backend dice:
PL_GETVAR CANCEL3 FROM 'CANCEL3' ELSE CANCEL3 = ''

Frontend debe usar:
<button name="CANCEL3" ...>Cancel</button>
```

**Proceso**:
1. **Leer el backend PRIMERO**
2. Buscar línea: `PL_GETVAR CANCEL# FROM 'CANCEL#'`
3. Usar ese mismo número en el frontend
4. No cambiar nombres arbitrariamente

**Ejemplos reales**:
- Si backend tiene `CANCEL2` → usa `name="CANCEL2"`
- Si backend tiene `CANCEL3` → usa `name="CANCEL3"`
- Si backend tiene `CANCEL4` → usa `name="CANCEL4"`

**IMPORTANTE**:
- No hay un estándar fijo para el número
- El número puede ser diferente en cada página
- Siempre confiar en lo que el backend espera
- Cuando modernicez legacy, preserva el número original del CANCEL

## Modernization Process for Legacy Files

When modernizing legacy HTML files (files without "EDIT" suffix):

### File Naming Convention
- **Legacy files**: `FILENAME.HTM` (e.g., `SHIPLCLABELP1.HTM`)
- **Modern files**: `FILENAMEEDIT.HTM` (e.g., `SHIPLCLABELP1EDIT.HTM`)

### Process Steps

1. **Read Legacy File**: Examine the original legacy file to understand:
   - Form fields and their names
   - Backend logic and redirects
   - Special functionality (dropdowns, validations, etc.)

2. **Modernize in Place**: Overwrite the same file with modern version:
   - Apply modern HTML5/CSS3 styling
   - Add responsive sidebar navigation
   - Implement modern JavaScript patterns
   - **PRESERVE** all backend PicLan-IP/BASIC logic exactly

3. **Update Redirects**: Change all internal redirects to point to "EDIT" versions:
   ```
   OLD: CALL PLW.PAGE('NEXTPAGE.HTM','',ERR)
   NEW: CALL PLW.PAGE('NEXTPAGEEDIT.HTM','',ERR)
   ```

   ```
   OLD: <form action="/CURRENTPAGE.HTM">
   NEW: <form action="/CURRENTPAGEEDIT.HTM">
   ```

4. **Exception - Menu Redirects**: Keep menu/system page redirects as-is:
   - `LOGINWH.HTM` - stays as-is
   - `STOCKMENU.HTM` - stays as-is
   - Other system-level pages - stays as-is

### Example Modernization

**Legacy File**: `SHIPLCLABELP1.HTM`
```html
<body bgcolor="#99FF99">
  <form action="/SHIPLCLABELP1.HTM">
    <!-- old styling -->
  </form>
  <pre>
    CALL PLW.PAGE('SHIPLCLABELP2.HTM','',ERR)
  </pre>
</body>
```

**Modernized Version** (overwrites same file):
```html
<body>
  <!-- Modern sidebar, header, card design -->
  <form action="/SHIPLCLABELP1EDIT.HTM">
    <!-- modern styling -->
  </form>
  <PRE>
    CALL PLW.PAGE('SHIPLCLABELP2EDIT.HTM','',ERR)
  </PRE>
</body>
```

### Automatic Detection Pattern

When you see files named without "EDIT" suffix:
1. **Recognize** it's a legacy file to be modernized
2. **Extract** backend logic and form fields
3. **Modernize** the entire file structure
4. **Update** all page redirects to use "EDIT" versions
5. **Overwrite** the original file (don't create a new file)

### Key Points
- **Overwrite, don't create**: Replace the legacy file in-place
- **Preserve backend logic**: Keep all PicLan-IP/BASIC code intact
- **Update form actions**: Change to point to current page with "EDIT"
- **Update redirects**: Change all next-page redirects to include "EDIT"
- **System pages**: Don't add "EDIT" to LOGINWH.HTM, STOCKMENU.HTM, etc.

### Quick Checklist
- [ ] Read legacy file completely
- [ ] Identify all form fields and backend variables
- [ ] Apply modern template structure
- [ ] Add sidebar with fixed close functionality
- [ ] Update form action to include "EDIT"
- [ ] Update all PLW.PAGE redirects to include "EDIT" (except system pages)
- [ ] Add proper JavaScript (auto-focus, validation, sidebar)
- [ ] Test that all variables match between frontend and backend
- [ ] Verify redirect chain works correctly

---

## Learning from New Bugs and Solutions

When you discover new bugs or create solutions while working on TAIMEX templates:

1. **Document the bug** with a clear description of the problem
2. **Explain the root cause** so it can be prevented in future
3. **Provide the solution** with code examples
4. **Add to this skill** by updating the "Common Bugs & Fixes" section

This skill will evolve as the project grows and new patterns emerge.
