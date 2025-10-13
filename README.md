# Sistema de Gestión de Préstamo de Servidores

Sistema completo para la gestión de préstamo de servidores académicos con Firebase como backend.

## 🚀 Características

### Módulo de Acceso (Registro y Login)
- ✅ Registro de alumnos con código institucional
- ✅ Login con correo o código de alumno
- ✅ Autenticación segura con Firebase Authentication

### Portal del Alumno
- 📝 Formulario completo de solicitud de préstamo
- 👥 Gestión de integrantes del grupo
- 📅 Selección de fecha y horario
- 🖥️ Selección de servidor y accesorios
- 📊 Historial de solicitudes
- 🔔 Visualización de respuestas del administrador

### Panel del Administrador
- 📈 Dashboard con indicadores numéricos
- 📊 Gráfica de pastel (distribución por estado)
- 📊 Gráfica de barras (demanda por servidor)
- 📋 Tabla de gestión de solicitudes
- ✅ Aprobar solicitudes con asignación de lugar
- ❌ Rechazar solicitudes con motivo
- 👁️ Ver detalles completos de cada solicitud

## 📦 Estructura del Proyecto

```
ea/
├── index.html          # Página de login y registro
├── alumno.html         # Portal del alumno
├── admin.html          # Panel del administrador
├── css/
│   └── styles.css      # Estilos globales
├── package.json        # Dependencias (Firebase)
└── README.md          # Este archivo
```

## 🔧 Configuración de Firebase

### 1. Crear un proyecto en Firebase

1. Ve a [Firebase Console](https://console.firebase.google.com/)
2. Crea un nuevo proyecto
3. Activa **Authentication** (Email/Password)
4. Activa **Firestore Database**

### 2. Configurar Authentication

En Firebase Console:
- Ve a **Authentication** > **Sign-in method**
- Activa **Email/Password**

### 3. Configurar Firestore Database

En Firebase Console:
- Ve a **Firestore Database** > **Create database**
- Modo: **Modo de prueba** (para desarrollo)

#### Reglas de Seguridad (Firestore Rules):

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    // Colección de alumnos
    match /alumnos/{alumnoId} {
      allow read, write: if request.auth != null;
    }
    
    // Colección de solicitudes
    match /solicitudes/{solicitudId} {
      allow read: if request.auth != null;
      allow create: if request.auth != null;
      allow update: if request.auth != null; // Solo admins en producción
    }
  }
}
```

### 4. Obtener Configuración

En Firebase Console:
1. Ve a **Project Settings** (⚙️)
2. En **Your apps** > **Web app** > **SDK setup and configuration**
3. Copia el objeto `firebaseConfig`

### 5. Actualizar la Configuración en los Archivos HTML

Reemplaza la configuración en los 3 archivos HTML (`index.html`, `alumno.html`, `admin.html`):

```javascript
const firebaseConfig = {
    apiKey: "TU_API_KEY",
    authDomain: "TU_AUTH_DOMAIN",
    projectId: "TU_PROJECT_ID",
    storageBucket: "TU_STORAGE_BUCKET",
    messagingSenderId: "TU_MESSAGING_SENDER_ID",
    appId: "TU_APP_ID"
};
```

## 🗄️ Estructura de la Base de Datos

### Colección: `alumnos`

```javascript
{
  uid: "firebase_auth_uid",
  codigo: "2021001234",
  nombre: "Juan Pérez",
  email: "juan.perez@institucion.edu",
  rol: "alumno",
  fechaRegistro: "2025-10-13T10:00:00.000Z"
}
```

### Colección: `solicitudes`

```javascript
{
  // Identificación
  nombreResponsable: "Juan Pérez",
  codigoResponsable: "2021001234",
  uidAlumno: "firebase_auth_uid",
  emailAlumno: "juan.perez@institucion.edu",
  
  // Contexto
  docente: "Dr. Carlos López",
  curso: "Redes y Comunicaciones",
  semestre: "2025-II",
  
  // Horario
  fecha: "2025-10-20",
  horaEntrada: "14:00",
  horaSalida: "16:00",
  
  // Recursos
  servidor: "Server1",
  tipoServidor: "Dell PowerEdge",
  caracteristicas: "32GB RAM, Intel Xeon",
  
  // Logística
  accesorios: ["Monitor", "Teclado", "Mouse"],
  integrantes: ["María García", "Luis Rodríguez"],
  
  // Estado
  estado: "PENDIENTE", // PENDIENTE | ACEPTADO | RECHAZADO
  fechaSolicitud: "2025-10-13T10:00:00.000Z",
  respuestaAdmin: null, // o "Solicitud APROBADA. Lugar: LAB-302"
  fechaRespuesta: null
}
```

## 🚀 Cómo Ejecutar el Proyecto

### Opción 1: Con Live Server (Recomendado)

1. Instala la extensión "Live Server" en VS Code
2. Haz clic derecho en `index.html`
3. Selecciona "Open with Live Server"

### Opción 2: Con un Servidor Local

```bash
# Usando Python 3
python -m http.server 8000

# Usando Node.js (npx)
npx http-server
```

Luego abre: `http://localhost:8000`

### Opción 3: Abrir Directamente

Simplemente abre `index.html` en tu navegador (puede tener limitaciones con módulos ES6).

## 📝 Uso del Sistema

### Para Alumnos:

1. **Registrarse**: Crear cuenta con código, nombre, correo y contraseña
2. **Iniciar Sesión**: Con correo o código de alumno
3. **Nueva Solicitud**: Llenar formulario completo
4. **Mis Solicitudes**: Ver el historial y respuestas

### Para Administradores:

1. **Acceder**: Usar el enlace "Acceso Administrador" en login
2. **Dashboard**: Ver estadísticas y gráficas
3. **Gestión**: Revisar solicitudes pendientes
4. **Aprobar**: Asignar lugar y aprobar
5. **Rechazar**: Indicar motivo del rechazo

## 🔐 Seguridad

### Para Producción:

1. **Restringir el Panel Admin**: Crear una colección `admins` y verificar rol
2. **Reglas de Firestore**: Actualizar reglas para que solo admins puedan actualizar solicitudes
3. **Validación**: Agregar validación del lado del servidor con Cloud Functions

### Ejemplo de Regla Admin:

```javascript
match /solicitudes/{solicitudId} {
  allow read: if request.auth != null;
  allow create: if request.auth != null;
  allow update: if request.auth != null && 
    get(/databases/$(database)/documents/admins/$(request.auth.uid)).data.rol == 'admin';
}
```

## 📊 Características del Dashboard

- **Indicadores Numéricos**: Total, Pendientes, Aprobadas, Rechazadas
- **Gráfica de Pastel**: Distribución porcentual por estado
- **Gráfica de Barras**: Demanda por servidor
- **Filtros**: Ver todas, pendientes, aprobadas o rechazadas
- **Acciones**: Detalles, Aprobar, Rechazar

## 🎨 Personalización

- **Colores**: Edita las variables CSS en `css/styles.css`
- **Servidores**: Modifica el `<select>` en `alumno.html`
- **Campos**: Agrega campos al formulario según necesites

## 📱 Responsive Design

El sistema es completamente responsive y funciona en:
- 💻 Desktop
- 📱 Tablets
- 📱 Móviles

## 🛠️ Tecnologías Utilizadas

- **Frontend**: HTML5, CSS3, JavaScript (ES6+)
- **Backend**: Firebase (Authentication + Firestore)
- **Gráficas**: Chart.js
- **Hosting**: Firebase Hosting (opcional)

## 📦 Despliegue en Firebase Hosting (Opcional)

```bash
# Instalar Firebase CLI
npm install -g firebase-tools

# Login
firebase login

# Inicializar
firebase init hosting

# Desplegar
firebase deploy
```

## 🐛 Solución de Problemas

### Error de CORS
- Usa Live Server o un servidor local
- No abras directamente el archivo HTML

### Firebase no se conecta
- Verifica que la configuración sea correcta
- Revisa las reglas de Firestore
- Activa Authentication en Firebase Console

### Las gráficas no aparecen
- Verifica que Chart.js se cargue correctamente
- Revisa la consola del navegador

## 📄 Licencia

Este proyecto es de código abierto para fines educativos.

## 👨‍💻 Autor

Sistema desarrollado para la gestión académica de préstamo de servidores.

---

**¡Listo para usar! 🎉**

Cualquier duda, revisa la consola del navegador o los logs de Firebase.
