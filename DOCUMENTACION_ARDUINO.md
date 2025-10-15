# Documentación - Sistema de Solicitud de Arduino

## Cambios Implementados

### 1. Nueva Pestaña "Nueva Solicitud Arduino"
Se ha agregado una nueva pestaña en el portal del alumno para solicitar el préstamo de kits Arduino.

### 2. Formulario de Solicitud Arduino

#### Campos Básicos (Autocompletados):
- **Nombre del Responsable**: Se carga automáticamente desde el perfil del alumno
- **Código del Responsable**: Se carga automáticamente desde el perfil del alumno
- **Semestre**: Precargado con "2025-II" (readonly)

#### Contexto Académico:
- **Docente Responsable**: Campo de texto para ingresar el nombre del profesor supervisor
- **Curso**: Campo de texto para especificar el curso (ej: Redes y Comunicaciones)

#### Horario de Uso:
- **Fecha de Uso**: Selector de fecha con validación (no permite fechas pasadas)
- **Hora de Entrada**: Selector de hora con validación en tiempo real
- **Hora de Salida**: Selector de hora (debe ser posterior a la hora de entrada)

#### Tipo de Alquiler:
Dos opciones disponibles:

**A) Kit Completo**
- Incluye todos los componentes del kit Arduino estándar
- Muestra automáticamente la lista de componentes incluidos:
  - 1x Placa Arduino UNO
  - 1x Pantalla LCD 16x2
  - 1x Display 7 Segmentos
  - 1x Sensor Ultrasónico HC-SR04
  - 1x Sensor DHT11 (Temperatura/Humedad)
  - 1x Sensor PIR (Movimiento)
  - 1x Módulo Bluetooth HC-05
  - 2x Servomotor SG90
  - 1x Motor DC con Driver L298N
  - 1x Protoboard 830 puntos
  - Set de Cables Jumper (40 unidades)
  - Set de LEDs (5 colores, 25 unidades)
  - Set de Resistencias (valores variados)
  - 5x Botones Pulsadores
  - Cable USB para programación

**B) Partes Individuales**
- Permite seleccionar componentes específicos mediante checkboxes:
  - Placas: Arduino UNO, MEGA, NANO
  - Pantallas: LCD 16x2, LCD 20x4, Display 7 Segmentos, Display OLED 0.96"
  - Sensores: Ultrasónico HC-SR04, DHT11, DHT22, PIR
  - Módulos de comunicación: Bluetooth HC-05, WiFi ESP8266
  - Motores: Servomotor SG90, Motor DC
  - Componentes básicos: Protoboard, Cables Jumper, LEDs, Resistencias, Botones

#### Propósito del Préstamo:
- Campo de texto largo para describir el proyecto a realizar

#### Integrantes del Grupo:
- Lista dinámica de integrantes
- Botón para agregar más integrantes
- Botón para eliminar integrantes

### 3. Almacenamiento en Firebase

#### Colección: `solicitudes_arduino`
Cada solicitud se guarda con la siguiente estructura:

```javascript
{
    // Identificación
    nombreResponsable: string,
    codigoResponsable: string,
    uidAlumno: string,
    emailAlumno: string,
    
    // Contexto
    docente: string,
    curso: string,
    semestre: string,
    
    // Horario
    fecha: string,
    horaEntrada: string,
    horaSalida: string,
    
    // Recursos Arduino
    tipoAlquiler: "completo" | "individual",
    partesSeleccionadas: array,
    proposito: string,
    
    // Logística
    integrantes: array,
    
    // Estado
    estado: "PENDIENTE" | "ACEPTADO" | "RECHAZADO",
    tipoSolicitud: "ARDUINO",
    fechaSolicitud: ISO string,
    timestamp: number,
    respuestaAdmin: string | null
}
```

### 4. Visualización en Historial

Las solicitudes de Arduino y de servidores se muestran juntas en la pestaña "Mis Solicitudes", diferenciadas por:

- **Icono**: 🔧 para Arduino, 🖥️ para servidores
- **Título**: "Arduino - [Curso]" vs "[Servidor] - [Curso]"
- **Información específica**: 
  - Arduino muestra el tipo de alquiler y propósito
  - Servidores muestran el nombre del servidor

### 5. Validaciones Implementadas

#### Fecha y Hora:
- No se permiten fechas pasadas
- Si se selecciona el día actual, la hora mínima es la hora actual
- La hora de salida debe ser posterior a la hora de entrada
- Validaciones en tiempo real

#### Partes Individuales:
- Si se selecciona "Partes Individuales", debe elegirse al menos un componente
- Validación antes de enviar el formulario

#### Campos Requeridos:
- Todos los campos marcados con (*) son obligatorios
- El formulario no se puede enviar si falta algún campo

### 6. Experiencia de Usuario

#### Feedback Visual:
- Mensajes de éxito al enviar solicitud
- Mensajes de error con explicaciones claras
- Botón de envío se deshabilita durante el proceso para evitar envíos duplicados
- Muestra "Enviando..." mientras se procesa

#### Flujo Post-Envío:
- Se limpia el formulario automáticamente
- Se restablecen los valores automáticos (nombre, código, semestre)
- Se cambia automáticamente a la pestaña de historial
- Se recargan las solicitudes para mostrar la nueva

## Configuración de Firebase

### Reglas de Firestore Recomendadas:

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    // Solicitudes de Arduino
    match /solicitudes_arduino/{solicitudId} {
      allow read: if request.auth != null;
      allow create: if request.auth != null && 
                      request.resource.data.uidAlumno == request.auth.uid;
      allow update: if request.auth != null && 
                      (resource.data.uidAlumno == request.auth.uid || 
                       get(/databases/$(database)/documents/administradores/$(request.auth.uid)).data.role == 'admin');
    }
  }
}
```

## Notas Técnicas

1. **Separación de Colecciones**: Las solicitudes de Arduino se guardan en `solicitudes_arduino` separadas de las solicitudes de servidores en `solicitudes`
2. **Compatibilidad**: El sistema es compatible con el sistema existente de solicitudes de servidores
3. **Escalabilidad**: Es fácil agregar más tipos de solicitudes siguiendo el mismo patrón
4. **Mantenimiento**: El código está modularizado y comentado para facilitar futuras modificaciones

## Próximos Pasos Sugeridos

1. Crear un panel de administración para gestionar solicitudes de Arduino
2. Agregar sistema de inventario para controlar disponibilidad de componentes
3. Implementar notificaciones por email al cambiar estado de solicitud
4. Agregar historial de mantenimiento de componentes
5. Crear reportes de uso y estadísticas
