# 10.3 Inventario de Módulos y Manuales Sugeridos

---

## Criterio

Este inventario se basa en los módulos y pantallas activas identificadas en backend y frontend del sistema.

Se usa para decidir:

- qué sí debe documentarse
- para qué rol
- en qué manual debe vivir cada tema

---

## Módulos Esenciales

| Módulo | Usuario Final | RRHH | Administrador | Documento recomendado |
|--------|---------------|------|---------------|-----------------------|
| Acceso al sistema y perfil | Sí | Sí | Sí | Manual por rol |
| Dashboard | Sí | Sí | Sí | Manual por rol |
| Asistencia personal | Sí | No | No | Manual del Usuario Final |
| Marcaciones personales | Sí | No | No | Manual del Usuario Final |
| Horarios personales | Sí | No | No | Manual del Usuario Final |
| Solicitudes personales | Sí | No | No | Manual del Usuario Final |
| Departamentos | No | Sí | Opcional | Manual de RRHH |
| Asignaciones de personal | No | Sí | Opcional | Manual de RRHH |
| Feriados | No | Sí | Opcional | Manual de RRHH |
| Reportes de asistencia | No | Sí | Opcional | Manual de RRHH |
| Tabla mensual del personal | No | Sí | Opcional | Manual de RRHH |
| Horarios | No | Sí | No | Manual de RRHH |
| Asignaciones de horario | No | Sí | No | Manual de RRHH |
| Solicitudes de permisos | No | Sí | No | Manual de RRHH |
| Pendientes de aprobación | No | Sí | Sí | Manual de RRHH y aprobaciones |
| Tipos de permiso | No | Sí | No | Manual de RRHH |
| Usuarios | No | No | Sí | Manual del Administrador |
| Dispositivos | No | No | Sí | Manual del Administrador |
| Mapeo de usuarios con dispositivos | No | No | Sí | Manual del Administrador |
| Registros crudos de dispositivos | No | No | Sí | Manual del Administrador |
| Configuraciones | No | No | Sí | Manual del Administrador |
| Reportes globales | No | No | Sí | Manual del Administrador |

---

## Módulos Secundarios o en Evolución

Estos módulos existen en código y conviene documentarlos cuando se confirme su salida operativa:

- Nómina
- Rangos de descuento
- Flujo detallado de aprobaciones multinivel
- Exportaciones avanzadas

---

## Recomendación de Paquetes Documentales

### Paquete 1: Usuario Final

- ingreso y cierre de sesión
- dashboard
- tabla mensual
- asistencia
- marcaciones
- horarios
- solicitudes

### Paquete 2: RRHH

- gestión organizacional
- asistencia global
- horarios y asignaciones
- permisos
- reportes

### Paquete 3: Administración

- administración de usuarios
- infraestructura funcional de dispositivos
- parámetros del sistema
- supervisión global

---

## Qué No Mezclar

No mezclar en el mismo manual:

- instrucciones técnicas de instalación
- arquitectura de software
- endpoints de API
- lógica interna de procesamiento

Eso debe seguir en la documentación técnica.
