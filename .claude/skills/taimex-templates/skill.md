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

## ⚠️ CRITICAL DESIGN RULES

### ❌ NO EMOJI ICONS

**NEVER use emoji icons in the HTML/CSS design.**

This includes:
- ❌ No emojis in HTML content (📋, 🔧, 📝, ℹ️, etc.)
- ❌ No emojis in CSS pseudo-elements (`::before`, `::after`)
- ❌ No emojis in button labels
- ❌ No emojis in empty states
- ❌ No emojis in headers or footers

**Use instead:**
- ✅ Plain text labels
- ✅ Unicode symbols (×, ☰, •, ▸, etc.)
- ✅ CSS borders, shapes, and backgrounds
- ✅ SVG icons if needed
- ✅ Font Awesome or similar icon fonts (if approved)

**Example:**

```html
❌ INCORRECT:
<h1>📋 Part Description Notes</h1>
<button>🖨️ Print</button>
<div class="info">ℹ️ Information</div>

✅ CORRECT:
<h1>Part Description Notes</h1>
<button>Print</button>
<div class="info">Information</div>
```

**Reason:** Professional enterprise applications should not use emoji icons as they:
- Appear unprofessional in business settings
- May render inconsistently across browsers/OS
- Are not appropriate for industrial/manufacturing context

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

**Note:** The hamburger menu ☰ is an acceptable Unicode symbol (not an emoji).
Acceptable Unicode: ×, ☰, •, ▸, ►, ◄, ▲, ▼, ★, ☆, □, ■

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
- Cancel action: `name="CANCEL2"` (standard across all forms)

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

### ❌ BUG: Error [B10] "Variable has not been assigned a value; zero used"

**Problema**: Al enviar formularios, aparece el error de servidor:
```
[B10] in program "WEB$0002XXXX", Line XX:
Variable has not been assigned a value; zero used.
```

**Causas Comunes**:

1. **Entidades HTML en código BASIC**: El código PicLan-IP/BASIC contiene `&quot;` en lugar de comillas simples.
2. **Campos de formulario duplicados**: Existe un `<select>` o `<input>` visible y un `<input type="hidden">` con el mismo `name=""`, causando conflictos de valores.

**Síntomas**:
- Error 500 Internal Server Error
- Variables BASIC sin valor asignado
- Formulario no procesa correctamente

**Solución 1 - Entidades HTML**:

Buscar en el bloque `<PRE>` todas las líneas con `&quot;` y reemplazar con comillas simples:

```basic
❌ INCORRECTO:
IF CANCEL NE &quot;&quot; THEN
IF FIELD NE &quot;value&quot; THEN

✅ CORRECTO:
IF CANCEL NE '' THEN
IF FIELD NE 'value' THEN
```

**Solución 2 - Campos Duplicados**:

Eliminar el campo `<input type="hidden">` cuando ya existe un `<select>` o `<input>` visible con el mismo nombre:

```html
❌ INCORRECTO (duplicado):
<select name="COO">...</select>
<input type="hidden" name="COO" value="^COO^">

✅ CORRECTO (sin duplicado):
<select name="COO">...</select>
<!-- Sin hidden input - el select ya envía el valor -->
```

**Patrón de Detección**:

Cuando veas este tipo de error:
1. Buscar `&quot;` en el código BASIC con: `grep -n "&quot;" filename.HTM`
2. Buscar campos duplicados con: `grep -n 'name="FIELDNAME"' filename.HTM`
3. Corregir ambos problemas si existen

**Prevención**:
- Al crear nuevos templates, usar SIEMPRE comillas simples `''` en código BASIC
- NO duplicar campos de formulario entre elementos visibles y hidden inputs
- Usar hidden inputs SOLO para campos que no tiene el usuario (IDs internos, estados, etc.)

### ❌ BUG: Error [B10] por Tag `<pre>` en Minúsculas

**Problema**: Al enviar formularios, aparece el error de servidor [B10] y las variables no se asignan correctamente.

**Causa**: El tag del bloque BASIC está en minúsculas `<pre>` en vez de mayúsculas `<PRE>`.

**Síntomas**:
- Error 500 Internal Server Error
- Error [B10] "Variable has not been assigned a value; zero used"
- El código BASIC no se ejecuta correctamente

**Explicación**: PicLan-IP/BASIC requiere que el tag sea `<PRE>` en mayúsculas para reconocer y ejecutar el bloque de código del servidor. Si se usa `<pre>` en minúsculas, el sistema no procesa el código BASIC y todas las variables quedan sin asignar.

**Solución**:

Siempre usar `<PRE>` en mayúsculas para el bloque de código BASIC:

```html
❌ INCORRECTO:
<pre>
PicLan-IP/BASIC ^ ^
...
</pre>

✅ CORRECTO:
<PRE>
PicLan-IP/BASIC ^ ^
...
</PRE>
```

**Patrón de Detección**:

```bash
grep -n '<pre>' filename.HTM
grep -n '</pre>' filename.HTM
```

Si encuentras resultados, cámbialos a mayúsculas.

**Prevención**:
- Al crear o modernizar templates, verificar que el tag sea siempre `<PRE>` en mayúsculas
- Agregar este check a la revisión de código antes de commitear
- Los linters/formatters HTML pueden cambiar a minúsculas - verificar después de formatear

### ❌ BUG: Error "Exceeded maximum 32767 file opens" - Loop Infinito de Redirects

**Problema**: El servidor muestra el error `[33] Exceeded the maximum number of 32767 file opens!` y el sistema se bloquea después de múltiples requests.

**Causa**: Un loop infinito de redirects causado por un `RETURN` sin `PLW.PAGE` en el backend. Cuando hay un `RETURN` vacío sin redirect, PicLan-IP vuelve a cargar la misma página infinitamente, agotando los file handles del sistema.

**Síntomas**:
- Error en servidor: `Exceeded the maximum number of 32767 file opens!`
- Sistema se congela o se vuelve muy lento
- Logs muestran requests repetidos a la misma página
- Puede aparecer `Frame Out of Range` después del error

**Código Problemático**:

```basic
❌ INCORRECTO (causa loop infinito):
IF COO[1,3] = 'Ple' OR COO = '' THEN
   * User clicked Next without selecting valid country - reload page
   RETURN  ← SIN REDIRECT = LOOP INFINITO
END
```

También causado por inicializaciones dummy que no existen en versiones legacy y hacen que el flujo continúe con datos inválidos:

```basic
❌ INCORRECTO (no debe existir):
* Initialize empty values if not provided (for initial page load)
IF PARTNUM = '' THEN PARTNUM = 'N/A'
IF QTY = '' THEN QTY = '0'
IF DCODE = '' THEN DCODE = 'N/A'
IF COO = '' THEN COO = 'N/A'
IF ROHS = '' THEN ROHS = 'N'
```

**Solución**:

1. **NUNCA usar `RETURN` sin un redirect previo**. Si la validación falla, simplemente no ejecutar el bloque de redirect usando lógica positiva:

```basic
✅ CORRECTO (validación positiva):
IF COO[1,3] NE 'Ple' AND GON1 NE '' THEN
   PL_PUTVAR PARTNUM IN 'PARTNUM' ELSE NULL
   PL_PUTVAR QTY IN 'QTY' ELSE NULL
   PL_PUTVAR DCODE IN 'DCODE' ELSE NULL
   PL_PUTVAR COO IN 'COO' ELSE NULL
   ERR = ''
   CALL PLW.PAGE('NEXTPAGE.HTM','',ERR)
   RETURN
END
* Si la validación falla, el código termina naturalmente
* sin RETURN y la página se mantiene sin recargar
```

2. **NO agregar inicializaciones dummy**: Si las variables vienen vacías desde el frontend, déjalas vacías. El frontend debe manejar valores vacíos correctamente con template syntax (`^VAR^`).

3. **Comparar con versión legacy**: Antes de agregar lógica nueva, siempre verificar que exista en la versión legacy original.

**Patrón de Validación Estándar**:

```basic
❌ INCORRECTO (causa loop):
IF FIELD = '' THEN
   RETURN  ← Recarga la página infinitamente
END

✅ CORRECTO (lógica positiva):
IF FIELD NE '' AND BUTTON NE '' THEN
   * Process and redirect
   PL_PUTVAR FIELD IN 'FIELD' ELSE NULL
   ERR = ''
   CALL PLW.PAGE('NEXTPAGE.HTM','',ERR)
   RETURN
END
* Termina naturalmente, página se mantiene
```

**CRÍTICO**:
- Todo `RETURN` DEBE estar precedido por `CALL PLW.PAGE` o ser parte de un redirect explícito (como el redirect de autenticación a `LOGINWH.HTM`).
- Si necesitas que la página se quede en su lugar cuando la validación falla, simplemente NO ejecutar el bloque de redirect (usar lógica positiva en el `IF`).
- Las variables vacías (`''`) son válidas y esperadas en carga inicial - NO inicializar con valores dummy como `'N/A'` o `'0'`.
- El único `RETURN` aceptable sin validación previa es el del check de autenticación (`PL_GET_COOKIE`).

**Patrón de Detección**:

```bash
# Buscar RETURNs peligrosos (que no son parte de un bloque PLW.PAGE)
grep -B3 "RETURN" filename.HTM | grep -v "CALL PLW.PAGE"

# Buscar inicializaciones dummy
grep "IF.*= '' THEN.*=" filename.HTM
```

**Prevención**:
- Al crear/modernizar templates, copiar la lógica BASIC exacta de la versión legacy
- No agregar validaciones o lógica extra sin verificar que existen en legacy
- Siempre usar validaciones positivas (checking NE '') en lugar de negativas (checking = '' con RETURN)
- Las variables vacías no necesitan inicialización - déjalas vacías

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
