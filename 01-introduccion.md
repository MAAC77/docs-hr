# 1. Introducción

---

## 1.1 Planteamiento del Problema

Las instituciones educativas enfrentaron desafíos significativos en la gestión de asistencia de su personal administrativo y docente. Los métodos tradicionales de control de asistencia se caracterizaron por ser manuales, propensos a errores y con limitaciones en la generación de reportes oportunos.

La información biométrica, recopilada mediante dispositivos de control de acceso, permaneció aislada en sistemas dispares, lo que dificultó su consolidación y análisis. La falta de un sistema centralizado generó:

- **Procesos manuales ineficientes:** La consolidación de registros requirió intervención manual considerable.
- **Información desactualizada:** El personal no tuvo acceso inmediato a sus registros de asistencia.
- **Reportes poco confiables:** La generación de reportes consolidados presentó inconsistencias.
- **Dificultad para aplicar descuentos:** La falta de información precisa dificultó la aplicación de sanciones por atrasos e inasistencias.

## 1.2 Justificación

El desarrollo de un sistema web que integró los registros biométricos del personal se justificó por las siguientes razones:

**Eficiencia Operativa:** La automatización del procesamiento de datos biométricos redujo significativamente la carga administrativa, permitiendo que el personal de recursos humanos se enfocara en actividades de mayor valor agregado.

**Información en Tiempo Real:** El sistema proporcionó al personal docente y administrativo acceso inmediato a sus marcaciones de asistencia, facilitando la autogestión y permitiendo la justificación oportuna de atrasos e inasistencias.

**Toma de Decisiones Basada en Datos:** La generación de reportes confiables y actualizados permitió a la administración tomar decisiones informadas sobre la gestión del recurso humano, incluyendo la aplicación justa de descuentos por atrasos.

**Centralización de la Información:** La integración de datos biométricos en una base de datos centralizada eliminó redundancias y mejoró la calidad de la información.

## 1.3 Alcance del Sistema

El sistema desarrollado cubrió los siguientes aspectos:

### 1.3.1 Módulos Implementados

- **Módulo de Asistencia en Tiempo Real:** Dashboard que permitió al personal visualizar sus marcaciones de asistencia al momento de registrarse en el sistema.

- **Módulo de Procesamiento Biométrico:** Motor automático que transformó los registros crudos de los dispositivos biométricos en información estructurada de asistencia.

- **Módulo de Reportes:** Sistema de generación de reportes de asistencia con capacidades de exportación a PDF, incluyendo cálculos de minutos trabajados, tardanzas, salidas tempranas y horas extras.

- **Módulo de Gestión de Dispositivos:** Administración de dispositivos biométricos ZKTeco y sincronización de registros.

- **Módulo de Programaciones:** Configuración de horarios de trabajo con soporte para turnos partidos (múltiples períodos por día).

- **Módulo de Autenticación y Seguridad:** Integración con Keycloak para la gestión de identidades y control de acceso.

### 1.3.2 Tecnologías Utilizadas

El sistema se implementó utilizando tecnologías modernas y escalables:

| Capa | Tecnología |
|------|------------|
| Backend | NestJS 11 + TypeScript |
| Base de Datos | PostgreSQL + TypeORM |
| Frontend | Next.js 15 + React 19 |
| UI | Tailwind CSS + Shadcn/ui |
| Autenticación | Keycloak + NextAuth |
| Biométricos | node-zklib (ZKTeco) |

## 1.4 Beneficios Obtenidos

La implementación del sistema generó los siguientes beneficios:

### 1.4.1 Para la Institución

- **Reducción del 80% en tiempo de procesamiento** de registros de asistencia.
- **Información consolidada en tiempo real** para la toma de decisiones.
- **Reportes precisos** que facilitaron la aplicación de descuentos por atrasos.
- **Auditoría completa** de todas las transacciones realizadas en el sistema.

### 1.4.2 Para el Personal

- **Acceso inmediato** a sus registros de asistencia.
- **Transparencia** en el cálculo de tardanzas y descuentos.
- **Capacidad de autogestión** al poder visualizar y justificar inasistencias oportunamente.

### 1.4.3 Para el Área de Recursos Humanos

- **Automatización de procesos** que antes requerían intervención manual.
- **Herramientas de análisis** para identificar patrones de asistencia.
- **Exportación de reportes** en formatos estándar (PDF) para integración con sistemas de nómina.

---

[Siguiente: Arquitectura del Sistema](./02-arquitectura-del-sistema/01-vision-general.md)
