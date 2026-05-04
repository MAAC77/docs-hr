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
| Departamentos | No | Sí | Sí | [Manual de Departamentos](./manual-rrhh-departamentos.md) |
| Asignaciones de personal | No | Sí | Sí | [Manual de Asignaciones de Personal](./manual-rrhh-asignaciones-personal.md) |
| Feriados | No | Sí | Sí | [Manual de Feriados](./manual-rrhh-feriados.md) |
| Reportes de asistencia | No | Sí | Sí | [Manual de Reportes de Asistencia](./manual-rrhh-reportes-asistencia.md) |
| Tabla mensual del personal | No | Sí | Sí | [Manual de Tabla Mensual de Personal](./manual-rrhh-tabla-mensual-personal.md) |
| Horarios | No | Sí | Sí | [Manual de Horarios](./manual-rrhh-horarios.md) |
| Asignaciones de horario | No | Sí | Sí | [Manual de Asignaciones de Horario](./manual-rrhh-asignaciones-horario.md) |
| Solicitudes de permisos | No | Sí | Sí | [Manual de Solicitudes de Permiso](./manual-rrhh-solicitudes-permiso.md) |
| Pendientes de aprobación | No | Sí | Sí | [Manual de Pendientes de Aprobación](./manual-rrhh-pendientes-aprobacion.md) |
| Tipos de permiso | No | Sí | Sí | [Manual de Tipos de Permiso](./manual-rrhh-tipos-permiso.md) |
| Usuarios | No | No | Sí | [Manual de Gestión de Usuarios](./manual-admin-usuarios.md) |
| Dispositivos | No | No | Sí | [Manual de Dispositivos](./manual-admin-dispositivos.md) |
| Mapeo de usuarios con dispositivos | No | No | Sí | [Manual de Mapeo de Usuarios a Dispositivos](./manual-admin-mapeo-usuarios-dispositivos.md) |
| Registros de dispositivos | No | Sí | Sí | [Manual de Registros de Dispositivos](./manual-admin-registros-dispositivos.md) |
| Configuraciones | No | No | Sí | [Manual de Configuraciones](./manual-admin-configuraciones.md) |
| Reportes globales | No | Sí | Sí | [Manual de Reportes Globales](./manual-admin-reportes-globales.md) |

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

- [Manual de RRHH](./manual-rrhh.md)
- reportes de asistencia
- tabla mensual del personal
- departamentos y asignaciones
- feriados
- horarios y asignaciones de horario
- solicitudes, pendientes y tipos de permiso

### Paquete 3: Administración

- [Manual de Administración del Sistema](./manual-administrador.md)
- dashboard administrativo
- dispositivos y mapeos
- registros de dispositivos
- usuarios
- configuraciones
- reportes globales

---

## Qué No Mezclar

No mezclar en el mismo manual:

- instrucciones técnicas de instalación
- arquitectura de software
- endpoints de API
- lógica interna de procesamiento

Eso debe seguir en la documentación técnica.
