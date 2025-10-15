# Guía de Administración - Módulo Arduino

## Nuevas Funcionalidades en Panel de Administrador

### 🎯 Resumen de Cambios

Se ha agregado un módulo completo para gestionar las solicitudes de Arduino en el panel de administración, con las siguientes características:

---

## 📋 Características Implementadas

### 1. **Navegación entre Secciones**

En la parte superior del panel, ahora hay dos botones de navegación:

- **🖥️ Solicitudes de Servidores**: Gestión de solicitudes de préstamo de servidores (original)
- **🔧 Solicitudes de Arduino**: Nueva sección para gestionar solicitudes de kits Arduino

**Funcionamiento:**
- Click en cada botón cambia entre las secciones
- El botón activo se resalta visualmente
- Cada sección es independiente con sus propios datos y gráficos

---

### 2. **Dashboard de Arduino**

Incluye estadísticas en tiempo real:

#### Indicadores Numéricos:
- **Total Solicitudes**: Contador total de solicitudes Arduino
- **Pendientes**: Solicitudes que esperan respuesta (⏳)
- **Aprobadas**: Solicitudes aceptadas (✅)
- **Rechazadas**: Solicitudes rechazadas (❌)

#### Gráficas:
1. **Tipo de Alquiler** (Gráfico de Pastel):
   - Kit Completo vs Partes Individuales
   - Visualización de preferencias de los estudiantes

2. **Componentes Más Solicitados** (Gráfico de Barras):
   - Top 10 componentes individuales más pedidos
   - Útil para gestión de inventario
   - Solo cuenta solicitudes de partes individuales

---

### 3. **Tabla de Gestión de Solicitudes Arduino**

#### Filtros Disponibles:
- **Todas**: Muestra todas las solicitudes
- **Pendientes**: Solo las que necesitan revisión
- **Aprobadas**: Historial de solicitudes aceptadas
- **Rechazadas**: Historial de solicitudes rechazadas

#### Columnas de la Tabla:
- **ID**: Número secuencial
- **Alumno**: Nombre y código del estudiante
- **Docente**: Profesor supervisor
- **Curso**: Nombre del curso
- **Fecha**: Fecha programada de uso
- **Horario**: Hora de entrada y salida
- **Tipo Alquiler**: "Kit Completo" o "Partes Individuales"
- **Estado**: Badge visual del estado actual
- **Acciones**: Botones de gestión

---

### 4. **Acciones Disponibles**

Para cada solicitud, el administrador puede:

#### A. **👁️ Ver Detalles**
Abre un modal con información completa:
- Datos del responsable (nombre, código, email)
- Contexto académico (docente, curso, semestre)
- Horario detallado
- **Recursos Arduino**:
  - Si es Kit Completo: Indica que es kit completo
  - Si son Partes Individuales: Lista completa de componentes seleccionados
- Propósito del préstamo (descripción del proyecto)
- Lista de integrantes del grupo
- Estado actual y respuesta del admin (si existe)

#### B. **✅ Aprobar** (solo para pendientes)
Abre modal de aprobación con campos:
- **Lugar de Retiro** (requerido): Dónde recoger el equipo
  - Ej: "Almacén de Electrónica - Piso 3"
- **Notas Adicionales** (opcional): Instrucciones extra
  - Horarios de retiro
  - Precauciones
  - Documentación necesaria

**Proceso:**
1. Completa los campos
2. Click en "Confirmar Aprobación"
3. El estado cambia a ACEPTADO
4. Se genera respuesta automática para el alumno
5. Se actualiza en Firebase
6. El alumno puede ver la respuesta en su portal

#### C. **❌ Rechazar** (solo para pendientes)
Abre modal de rechazo con campo:
- **Motivo del Rechazo** (requerido): Explicación clara del rechazo
  - Ej: "Componentes no disponibles en las fechas solicitadas"
  - Ej: "El proyecto no justifica el uso de estos recursos"

**Proceso:**
1. Escribe el motivo
2. Click en "Confirmar Rechazo"
3. El estado cambia a RECHAZADO
4. Se envía respuesta al alumno
5. Se actualiza en Firebase

---

## 🔄 Flujo de Trabajo Recomendado

### Para Solicitudes Pendientes:

1. **Revisar Nueva Solicitud**
   - Acceder a la pestaña "🔧 Solicitudes de Arduino"
   - Filtrar por "Pendientes"
   - Click en "👁️ Detalles" para ver información completa

2. **Evaluar la Solicitud**
   - Verificar que el propósito sea válido
   - Revisar disponibilidad de recursos
   - Confirmar que la fecha/hora sea viable
   - Validar que el proyecto justifique los recursos

3. **Tomar Decisión**
   
   **Si se APRUEBA:**
   - Click en "✅ Aprobar"
   - Especificar lugar de retiro
   - Agregar instrucciones (horarios, documentación necesaria)
   - Confirmar
   
   **Si se RECHAZA:**
   - Click en "❌ Rechazar"
   - Explicar claramente el motivo
   - Sugerir alternativas si es posible
   - Confirmar

4. **Seguimiento**
   - La solicitud se mueve automáticamente al filtro correspondiente
   - El alumno recibe notificación en su portal
   - Las estadísticas se actualizan automáticamente

---

## 📊 Uso de las Gráficas

### Gráfico de Tipo de Alquiler
**Utilidad:**
- Identificar tendencia de uso (completo vs individual)
- Planificar compras de kits adicionales
- Detectar si hay demanda de componentes específicos

### Gráfico de Componentes Más Solicitados
**Utilidad:**
- Gestión de inventario
- Identificar componentes que necesitan más stock
- Detectar patrones de uso en cursos específicos
- Priorizar compras de reposición

---

## 🗄️ Estructura en Firebase

### Colección: `solicitudes_arduino`

Cada documento contiene:
```javascript
{
  nombreResponsable: string,
  codigoResponsable: string,
  uidAlumno: string,
  emailAlumno: string,
  docente: string,
  curso: string,
  semestre: string,
  fecha: string,
  horaEntrada: string,
  horaSalida: string,
  tipoAlquiler: "completo" | "individual",
  partesSeleccionadas: array,
  proposito: string,
  integrantes: array,
  estado: "PENDIENTE" | "ACEPTADO" | "RECHAZADO",
  tipoSolicitud: "ARDUINO",
  fechaSolicitud: ISO string,
  timestamp: number,
  respuestaAdmin: string | null,
  fechaRespuesta: ISO string (cuando se responde)
}
```

---

## 🔐 Permisos y Seguridad

- Solo usuarios con email `admin@gmail.com` pueden acceder
- Verificación automática en carga de página
- Redirección automática si no es administrador
- Actualizaciones directas a Firebase con validación

---

## 💡 Consejos de Uso

### Para Aprobar Solicitudes:
✅ **SÍ hacer:**
- Verificar disponibilidad real de componentes
- Proporcionar instrucciones claras de retiro
- Especificar horarios de disponibilidad
- Mencionar documentación necesaria (DNI, carnet, etc.)

❌ **NO hacer:**
- Aprobar sin verificar stock
- Dejar campo de lugar vacío
- Ser ambiguo en las instrucciones

### Para Rechazar Solicitudes:
✅ **SÍ hacer:**
- Explicar claramente el motivo
- Ser específico sobre el problema
- Sugerir alternativas cuando sea posible
- Mantener tono profesional y educativo

❌ **NO hacer:**
- Rechazar sin justificación
- Usar lenguaje poco profesional
- Dejar al alumno sin opciones

### Gestión General:
- Revisar solicitudes pendientes diariamente
- Mantener comunicación clara con los alumnos
- Actualizar estado de solicitudes prontamente
- Usar las gráficas para planificación de recursos

---

## 🐛 Solución de Problemas

### Problema: No se cargan las solicitudes Arduino
**Solución:** 
- Verificar conexión a internet
- Revisar consola de navegador (F12) para errores
- Confirmar que la colección `solicitudes_arduino` existe en Firebase

### Problema: No se pueden aprobar/rechazar solicitudes
**Solución:**
- Verificar permisos en Firestore Rules
- Asegurar que el usuario está autenticado como admin
- Revisar consola para errores de Firebase

### Problema: Las gráficas no se muestran correctamente
**Solución:**
- Recargar la página
- Verificar que Chart.js esté cargado correctamente
- Comprobar que hay datos en la colección

---

## 📈 Métricas Recomendadas a Monitorear

1. **Tasa de Aprobación**: % de solicitudes aprobadas vs rechazadas
2. **Tiempo de Respuesta**: Días entre solicitud y respuesta
3. **Componentes Críticos**: Piezas con alta demanda
4. **Cursos con Mayor Demanda**: Para planificación de recursos
5. **Tendencias Temporales**: Picos de demanda según calendario académico

---

## 🔄 Actualizaciones Futuras Sugeridas

- [ ] Sistema de notificaciones por email
- [ ] Integración de calendario para gestión de disponibilidad
- [ ] Sistema de reservas automático
- [ ] Control de inventario en tiempo real
- [ ] Historial de préstamos por alumno
- [ ] Reportes exportables (PDF/Excel)
- [ ] Sistema de multas por entregas tardías
- [ ] Checklist de devolución de componentes
