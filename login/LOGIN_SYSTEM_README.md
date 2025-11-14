# Sistema de Login y Autenticación TAIMEX

## 🔐 Descripción General

Sistema completo de autenticación que conecta la página de login con todas las páginas del sistema (SO_EDIT, PO_EDIT, etc.), usando la base de datos **OPERS** para validación de usuarios.

---

## 📁 Archivos del Sistema

### **1. LOGINEDIT.HTM** (Frontend - Login Page)
- **Descripción:** Página de login con diseño profesional y minimalista
- **Características:**
  - Fondo con imagen blur (taimex_stamping.jpg)
  - Formulario AJAX sin recargar página
  - Validación en tiempo real
  - Mensajes de error animados
  - Auto-uppercase en iniciales
  - Loading state durante login

### **2. LOGIN.XML** (Backend - Authentication)
- **Descripción:** Programa Pick/BASIC que valida credenciales contra OPERS
- **Base de datos:** OPERS
- **Campos utilizados:**
  - `<1>` - Nombre completo del usuario
  - `<2>` - Password (PICKPWD)
  - `<12>` - Menu asignado (página de redirección)

### **3. SO_EDIT.HTM** (Modificado)
- **Descripción:** Página de edición de Sales Orders con autenticación
- **Funciones agregadas:**
  - `checkUserSession()` - Verifica sesión activa
  - `getUserInitials()` - Obtiene iniciales del usuario logueado
  - `getUserName()` - Obtiene nombre completo del usuario
  - Redirección automática a login si no está autenticado

---

## 🔄 Flujo de Autenticación

```
┌─────────────────────────────────────────────────────────────┐
│                    FLUJO COMPLETO                           │
└─────────────────────────────────────────────────────────────┘

1. Usuario accede a SO_EDIT.HTM
   ↓
2. checkUserSession() verifica sessionStorage
   ↓
3A. SI NO HAY SESIÓN:
    - Redirige a /LOGINEDIT.HTM
    ↓
4. Usuario ingresa INITIALS y PWD
   ↓
5. AJAX POST a LOGIN.XML
   ↓
6. LOGIN.XML valida contra OPERS:
   - Lee OPER desde OPERSF usando INITIALS
   - Compara PASSWORD (field <2>)
   - Verifica que tenga MENU (field <12>)
   ↓
7A. SI VÁLIDO:
    - Crea session con SESSIONSAVE
    - Retorna XML con success=true, redirect_url, user_name
    - Frontend guarda en sessionStorage:
      * userInitials
      * userName
    - Redirige a MENU (ej: /SO_EDIT.HTM)
    ↓
7B. SI INVÁLIDO:
    - Retorna XML con success=false, mensaje de error
    - Muestra error en pantalla
    - Permite reintentar
    ↓
3B. SI HAY SESIÓN ACTIVA:
    - Carga SO_EDIT normalmente
    - Usa userInitials para filtrar drafts (TMP)
```

---

## 🗄️ Base de Datos OPERS

### **Estructura:**
```basic
OPERS Record:
├── Field 1:  USER_NAME (Nombre completo)
├── Field 2:  PASSWORD (Código personal)
├── Field 12: WEB_MENU (Página de inicio)
└── Record ID: INITIALS (Clave primaria)
```

### **Ejemplo de Registro:**
```
ID: EC
<1> = "Edgar Cabrera"
<2> = "1234"
<12> = "SO_EDIT.HTM"
```

---

## 💾 Gestión de Sesión

### **Backend (Pick/BASIC):**
```basic
SESSION_NAME = 'TAIMEX_SESSION_ID'
SESSION_DATA<1> = 'true'           # Authenticated
SESSION_DATA<2> = INITIALS          # User ID
SESSION_DATA<3> = USER_NAME         # Full Name

CALL SESSIONGENERATEID(SESSION_ID)
CALL SESSIONSAVE(SESSION_DATA, SESSION_ID)

* Set cookie
PL_ADD_HDR 'Set-Cookie: TAIMEX_SESSION_ID=...; path=/; HttpOnly'
```

### **Frontend (JavaScript):**
```javascript
// Después de login exitoso:
sessionStorage.setItem('userInitials', 'EC');
sessionStorage.setItem('userName', 'Edgar Cabrera');

// En SO_EDIT.HTM al cargar:
const initials = sessionStorage.getItem('userInitials');
if (!initials) {
  window.location.href = '/LOGINEDIT.HTM';
}
```

---

## 🔗 Integración con Sistema TMP

### **Filtrado de Drafts por Usuario:**

**GET_SO_TMP.XML:**
```basic
PL_GETVAR USER_FILTER FROM 'user_initials' ELSE USER_FILTER = ''

IF USER_FILTER <> '' THEN
  * Solo mostrar drafts de este usuario
  IF INDEX(TEMP_ID, USER_FILTER, 1) = 0 THEN
    INCLUDE_RECORD = 0
  END
END
```

**Frontend (SO_EDIT.HTM):**
```javascript
function loadSavedDrafts() {
  const userInitials = getUserInitials(); // Obtiene de sessionStorage
  
  fetch(`GET_SO_TMP.XML?user_initials=${userInitials}`)
    .then(response => response.text())
    .then(xmlText => {
      // Muestra solo drafts del usuario logueado
    });
}
```

---

## 🎨 Características del Login UI

### **Diseño Profesional:**
- ✅ Fondo con imagen blur + overlays graduales
- ✅ Card con efecto glass (backdrop-filter)
- ✅ Logo TAIMEX con fallback
- ✅ Inputs con animaciones de focus
- ✅ Botón con loading spinner
- ✅ Mensajes de error animados

### **Validaciones:**
- ✅ Campos requeridos
- ✅ Auto-uppercase en initials
- ✅ Validación en tiempo real
- ✅ Mensajes de error claros
- ✅ Auto-hide de errores (10 segundos)

---

## 📝 Respuesta XML de LOGIN.XML

### **Éxito:**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<login_response>
  <success>true</success>
  <message>Login successful</message>
  <initials>EC</initials>
  <redirect_url>/SO_EDIT.HTM</redirect_url>
  <user_name>Edgar Cabrera</user_name>
</login_response>
```

### **Error:**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<login_response>
  <success>false</success>
  <message>Invalid password - Please check your credentials</message>
  <initials>EC</initials>
</login_response>
```

---

## 🔒 Seguridad Implementada

1. **Session Management:**
   - Cookies HttpOnly
   - Session IDs únicos generados por SESSIONGENERATEID
   - Datos de sesión almacenados server-side

2. **Validación Frontend:**
   - Verificación de sesión en cada página
   - Redirección automática si no está autenticado
   - sessionStorage para datos no sensibles

3. **Validación Backend:**
   - Contraseña validada contra OPERS<2>
   - Verificación de menu asignado
   - Mensajes de error genéricos (no revela si usuario existe)

---

## 🚀 Próximos Pasos

### **Para Implementar en Otras Páginas:**

1. **PO_EDIT.HTM:**
```javascript
// Agregar al inicio del DOMContentLoaded:
if (!checkUserSession()) {
  return;
}
const userInitials = getUserInitials();
```

2. **OP_EDIT.HTM:**
```javascript
// Copiar funciones de SO_EDIT.HTM:
// - checkUserSession()
// - getUserInitials()
// - getUserName()
```

3. **COMMODITIES_EDIT.HTM:**
```javascript
// Mismo patrón para todas las páginas
```

---

## 🧪 Testing

### **Test Cases:**

1. **Login Exitoso:**
   - Initials: EC
   - Password: (el correcto de OPERS<2>)
   - Expected: Redirige a menu asignado

2. **Password Incorrecto:**
   - Expected: "Invalid password - Please check your credentials"

3. **Usuario No Existe:**
   - Expected: "Invalid user - User not found in system"

4. **Sin Menu Asignado:**
   - Expected: "Access denied - User has no configured web menu"

5. **Acceso Directo sin Login:**
   - Ir a SO_EDIT.HTM sin login
   - Expected: Redirige automáticamente a LOGINEDIT.HTM

---

## 📊 Mensajes de Error

| Código | Mensaje | Causa |
|--------|---------|-------|
| E001 | "Invalid user - User not found in system" | INITIALS no existe en OPERS |
| E002 | "Invalid password - Please check your credentials" | Password incorrecto |
| E003 | "Access denied - User has no configured web menu" | OPER<12> está vacío |
| E004 | "Please enter both initials and password" | Campos vacíos |
| E005 | "Connection error. Please try again." | Error de red/servidor |

---

## 🔧 Mantenimiento

### **Agregar Nuevo Usuario en OPERS:**
```basic
* En TCL:
ED OPERS NEWUSER
001 John Doe
002 password123
012 SO_EDIT.HTM
```

### **Cambiar Password:**
```basic
ED OPERS EC
002 newpassword456
```

### **Cambiar Menu:**
```basic
ED OPERS EC
012 PO_EDIT.HTM
```

---

## ✅ Checklist de Implementación

- [x] LOGIN.XML creado y funcional
- [x] LOGINEDIT.HTM diseñado y conectado
- [x] SO_EDIT.HTM protegido con autenticación
- [x] Session management implementado
- [x] Filtrado de drafts por usuario
- [ ] PO_EDIT.HTM protegido
- [ ] OP_EDIT.HTM protegido
- [ ] COMMODITIES_EDIT.HTM protegido
- [ ] Botón de Logout implementado
- [ ] "Remember me" opcional
- [ ] Password recovery

---

## 📞 Soporte

**Desarrollado por:** AI Assistant
**Fecha:** 2025-10-03
**Versión:** 1.0

**Basado en:** Subroutina PIK original adaptada para XML
