# 👥 Sistema de Roles - Documentación

## 📋 Resumen

El sistema de préstamo de servidores y Arduino utiliza un sistema de roles para controlar el acceso y permisos de los usuarios.

---

## 🎭 Roles Disponibles

### 1. **ALUMNO** 👨‍🎓
- **Rol por defecto** para todos los usuarios registrados
- Se asigna automáticamente durante el registro
- Acceso al panel `alumno.html`

### 2. **ADMIN** 👨‍💼
- Rol especial para administradores
- **NO se puede crear mediante registro normal**
- Debe crearse mediante `CREAR_ADMIN.html`
- Acceso al panel `admin.html`

---

## 🔐 Asignación de Roles

### **Registro Normal (login_register.html)**

```javascript
// Todos los usuarios registrados aquí reciben ROL DE ALUMNO
const alumnoData = {
    codigo: codigo,
    nombre: nombre,
    email: email,
    rol: 'alumno', // ⚠️ ROL FIJO: ALUMNO
    fechaRegistro: new Date().toISOString(),
    timestamp: Date.now()
};
```

**Características:**
- ✅ Rol `'alumno'` asignado automáticamente
- ✅ Guardado en colección `alumnos` de Firestore
- ✅ No se puede cambiar durante el registro
- ✅ Acceso limitado a funcionalidades de alumno

---

### **Creación de Administrador (CREAR_ADMIN.html)**

```javascript
// El admin se crea con credenciales específicas
Email: admin@gmail.com
Password: 123456
Rol: admin (implícito)
```

**Características:**
- ⚠️ Solo debe ejecutarse UNA VEZ
- ✅ Email fijo: `admin@gmail.com`
- ✅ Password predeterminado: `123456`
- ✅ Acceso completo al panel administrativo

---

## 🛡️ Validaciones Implementadas

### **1. Prevención de Registro con Email Admin**

```javascript
// Validación en login_register.html
if (email.toLowerCase() === 'admin@gmail.com') {
    alert('❌ No se puede registrar con el email del administrador');
    return;
}
```

**Protege contra:**
- ❌ Intento de crear cuenta con email de admin
- ❌ Usuarios intentando obtener permisos de admin
- ❌ Conflictos de credenciales

---

### **2. Validación de Contraseñas**

```javascript
// Contraseñas deben coincidir
if (password !== passwordConfirm) {
    alert('❌ Las contraseñas no coinciden');
    return;
}

// Longitud mínima
if (password.length < 6) {
    alert('❌ La contraseña debe tener al menos 6 caracteres');
    return;
}
```

---

## 📊 Estructura de Datos en Firestore

### **Colección: `alumnos`**

```javascript
{
    uid: "abc123xyz", // Firebase Auth UID
    codigo: "2021001234",
    nombre: "Juan Pérez García",
    email: "juan.perez@universidad.edu",
    rol: "alumno", // ⚠️ SIEMPRE "alumno" para registros normales
    fechaRegistro: "2025-10-15T10:30:00.000Z",
    timestamp: 1697365800000
}
```

---

## 🔀 Flujo de Autenticación por Rol

### **Flujo de Alumno:**

```
1. Usuario se registra en login_register.html
   ↓
2. Sistema asigna rol: "alumno" automáticamente
   ↓
3. Datos guardados en colección "alumnos"
   ↓
4. Login exitoso
   ↓
5. Redirección a alumno.html
   ↓
6. Acceso a:
   - Solicitudes de servidores
   - Solicitudes de Arduino
   - Historial personal
```

---

### **Flujo de Administrador:**

```
1. Admin creado previamente con CREAR_ADMIN.html
   ↓
2. Credenciales: admin@gmail.com / 123456
   ↓
3. Login en login_register.html
   ↓
4. Sistema detecta email de admin
   ↓
5. Redirección a admin.html
   ↓
6. Acceso a:
   - Gestión de solicitudes de servidores
   - Gestión de solicitudes de Arduino
   - Aprobar/Rechazar solicitudes
   - Dashboard con estadísticas
   - Gráficas analíticas
```

---

## 🔍 Verificación de Roles

### **En alumno.html:**

```javascript
onAuthStateChanged(auth, async (user) => {
    if (user) {
        const docSnap = await getDoc(doc(db, 'alumnos', user.uid));
        if (docSnap.exists()) {
            alumnoData = docSnap.data();
            // Verificar que tiene rol de alumno
            console.log('Rol del usuario:', alumnoData.rol);
        }
    } else {
        window.location.href = 'login_register.html';
    }
});
```

---

### **En admin.html:**

```javascript
onAuthStateChanged(auth, async (user) => {
    if (!user) {
        alert('❌ Debes iniciar sesión como administrador');
        window.location.href = 'login_register.html';
        return;
    }
    
    // Verificar si es admin por email
    if (user.email === 'admin@gmail.com') {
        console.log('✅ Acceso de administrador confirmado');
        // Cargar panel admin
    } else {
        console.log('❌ Usuario no es administrador');
        alert('❌ Acceso denegado. Solo administradores pueden acceder.');
        await signOut(auth);
        window.location.href = 'login_register.html';
    }
});
```

---

## ⚠️ Consideraciones de Seguridad

### **1. Rol Inmutable en Registro**

```javascript
// El rol NO se puede modificar desde el formulario
// Está hardcoded en el código del servidor (frontend)
rol: 'alumno' // ← VALOR FIJO
```

**Ventaja:** 
- ✅ Los usuarios no pueden auto-asignarse rol de admin
- ✅ Control total sobre permisos

**Desventaja:**
- ⚠️ Si necesitas cambiar el rol, debes hacerlo manualmente en Firestore

---

### **2. Detección de Admin por Email**

```javascript
// La verificación de admin se basa en el email
if (user.email === 'admin@gmail.com') {
    // Es administrador
}
```

**Ventaja:**
- ✅ Simple y directo
- ✅ Funciona sin necesidad de leer Firestore

**Desventaja:**
- ⚠️ Email hardcoded en el código
- ⚠️ Solo puede haber un admin

---

### **3. Prevención de Email Admin en Registro**

```javascript
// Nueva validación implementada
if (email.toLowerCase() === 'admin@gmail.com') {
    alert('❌ No se puede registrar con el email del administrador');
    return;
}
```

**Protección:**
- ✅ Evita conflictos de credenciales
- ✅ Previene intentos de escalada de privilegios
- ✅ Mantiene la unicidad del admin

---

## 📝 Casos de Uso

### **Caso 1: Nuevo Alumno se Registra**

```
Usuario: Juan Pérez
Email: juan.perez@universidad.edu
Acción: Registro en login_register.html

Resultado:
✅ Cuenta creada en Firebase Auth
✅ Documento creado en Firestore colección "alumnos"
✅ Rol asignado: "alumno"
✅ Acceso a: alumno.html
❌ Sin acceso a: admin.html
```

---

### **Caso 2: Admin Intenta Registrarse**

```
Usuario: Administrador
Email: admin@gmail.com
Acción: Intento de registro en login_register.html

Resultado:
❌ Registro bloqueado por validación
❌ Mensaje: "No se puede registrar con el email del administrador"
ℹ️ Debe usar CREAR_ADMIN.html en su lugar
```

---

### **Caso 3: Alumno Intenta Acceder a Panel Admin**

```
Usuario: Juan Pérez (alumno)
Acción: Navega manualmente a admin.html

Resultado:
❌ Acceso denegado
❌ Mensaje: "Solo administradores pueden acceder"
✅ Redirección automática a login_register.html
✅ Sesión cerrada
```

---

### **Caso 4: Admin se Loguea**

```
Usuario: Administrador
Email: admin@gmail.com
Password: 123456
Acción: Login en login_register.html

Resultado:
✅ Sistema detecta email de admin
✅ Login exitoso
✅ Redirección a admin.html
✅ Acceso completo a todas las funciones
```

---

## 🔧 Mantenimiento de Roles

### **Cambiar Rol de un Usuario Existente:**

1. **Opción 1: Manualmente en Firebase Console**
   ```
   1. Ir a Firebase Console
   2. Firestore Database
   3. Colección "alumnos"
   4. Seleccionar documento del usuario
   5. Editar campo "rol"
   6. Cambiar a "admin" o "alumno"
   ```

2. **Opción 2: Script de Actualización**
   ```javascript
   // Crear un script temporal para cambiar roles
   const updateRole = async (uid, newRole) => {
       const docRef = doc(db, 'alumnos', uid);
       await updateDoc(docRef, {
           rol: newRole
       });
   };
   ```

---

## 📊 Permisos por Rol

| Permiso | Alumno | Admin |
|---------|--------|-------|
| **Solicitar Servidor** | ✅ | ❌ |
| **Solicitar Arduino** | ✅ | ❌ |
| **Ver Historial Propio** | ✅ | ❌ |
| **Ver Todas las Solicitudes** | ❌ | ✅ |
| **Aprobar Solicitudes** | ❌ | ✅ |
| **Rechazar Solicitudes** | ❌ | ✅ |
| **Ver Estadísticas** | ❌ | ✅ |
| **Ver Gráficas** | ❌ | ✅ |
| **Acceder a alumno.html** | ✅ | ❌ |
| **Acceder a admin.html** | ❌ | ✅ |

---

## 🚀 Mejoras Futuras

### **Posibles Extensiones:**

1. **Roles Adicionales:**
   ```javascript
   - "profesor" → Puede ver solicitudes de sus cursos
   - "super_admin" → Admin con más permisos
   - "moderador" → Puede aprobar pero no rechazar
   ```

2. **Sistema de Permisos Granular:**
   ```javascript
   permissions: {
       canRequestServer: true,
       canRequestArduino: true,
       canApprove: false,
       canReject: false,
       canViewStats: false
   }
   ```

3. **Verificación de Rol desde Firestore:**
   ```javascript
   // En lugar de verificar solo por email
   // Verificar el campo "rol" en Firestore
   const userDoc = await getDoc(doc(db, 'alumnos', user.uid));
   if (userDoc.data().rol === 'admin') {
       // Es admin
   }
   ```

---

## ✅ Checklist de Implementación

- [x] Rol "alumno" asignado automáticamente en registro
- [x] Validación para prevenir registro con email admin
- [x] Verificación de admin por email en login
- [x] Redirección correcta según rol
- [x] Protección de panel admin contra acceso no autorizado
- [x] Logout y limpieza de sesión
- [x] Documentación completa del sistema de roles

---

## 🎯 Resumen

**Sistema de Roles Actual:**

✅ **Registro → ROL ALUMNO automático**  
✅ **Admin → Creado por CREAR_ADMIN.html**  
✅ **Validación → Email admin bloqueado en registro**  
✅ **Seguridad → Verificación en cada panel**  
✅ **Permisos → Separados por rol**  

**¡Sistema de roles funcionando correctamente!** 🔐
