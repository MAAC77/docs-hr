# Sistema Web de Gestión de asistencia y Administración de RRHH

## Documentación Técnica y Aplicativa

Proyecto de Grado

**📋 [Resumen Técnico](./RESUMEN_TECNICO.md)** - Vista general simplificada del sistema

---

## Índice General

### [00. Portada](./00-portada.md)
Objetivo general y objetivos específicos del proyecto.

### [01. Introducción](./01-introduccion.md)
- Planteamiento del problema
- Justificación
- Alcance del sistema
- Beneficios obtenidos

---

## 2. Arquitectura del Sistema

### [2.1 Visión General](./02-arquitectura-del-sistema/01-vision-general.md)
- Contexto del sistema (C4 Level 1)
- Flujo de datos de alto nivel
- Principios arquitectónicos
- Patrones arquitectónicos implementados

### [2.2 Arquitectura Backend](./02-arquitectura-del-sistema/02-arquitectura-backend.md)
- Vista de contenedores (C4 Level 2)
- Vista de componentes (C4 Level 3)
- Estructura de módulos
- Ciclo de vida de una petición
- Transaccionalidad y auditoría

### [2.3 Arquitectura Frontend](./02-arquitectura-del-sistema/03-arquitectura-frontend.md)
- Vista de contenedores
- Vista de componentes
- Arquitectura de componentes React
- Gestión del estado del servidor (TanStack Query)
- Componentes UI principales

### [2.4 Modelo de Datos](./02-arquitectura-del-sistema/04-modelo-de-datos.md)
- Diagrama entidad-relación
- Entidades principales
- Estados de asistencia
- Índices y optimizaciones

---

## 3. Módulo de Asistencia en Tiempo Real

### [3.1 Descripción General](./03-modulo-asistencia-en-tiempo-real/01-descripcion-general.md)
- Propósito del módulo
- Componentes principales (TodayStatusCard, EventsTimeline)
- Actualización en tiempo real
- Estados de una sesión de asistencia

### [3.2 Casos de Uso](./03-modulo-asistencia-en-tiempo-real/02-casos-de-uso.md)
- Diagrama de casos de uso
- Descripción detallada de cada caso
- Matriz de actores vs. casos de uso

### [3.3 Flujo de Datos](./03-modulo-asistencia-en-tiempo-real/03-flujo-de-datos.md)
- Diagrama de secuencia completo
- Flujo de consulta del dashboard
- Estrategia de actualización en tiempo real
- Transformación de datos por capa

### [3.4 Procedimiento Aplicativo](./03-modulo-asistencia-en-tiempo-real/04-procedimiento-aplicativo.md)
- Procedimientos paso a paso
- Interpretación de estados e indicadores
- Procedimientos comunes
- Interpretación de métricas

---

## 4. Módulo de Procesamiento Biométrico

### [4.1 Descripción General](./04-modulo-procesamiento-biometrico/01-descripcion-general.md)
- El motor de procesamiento (AttendanceEngineService)
- Pipeline de 6 pasos
- Context Object y estados de procesamiento
- Normalización de datos

### [4.2 Pipeline de Procesamiento](./04-modulo-procesamiento-biometrico/02-pipeline-de-procesamiento.md)
- Vista general del pipeline
- Paso 1: LoadData
- Paso 1.5: EnsureSessions
- Paso 2: FilterEvents
- Paso 3: ClassifyEvents
- Paso 4: PersistEvents
- Paso 5: CalculateSessions
- Paso 6: MarkProcessed
- Performance del pipeline

### [4.3 Clasificación de Eventos](./04-modulo-procesamiento-biometrico/03-clasificacion-eventos.md)
- Tipos de eventos
- Algoritmo de clasificación
- Ventanas de clasificación
- Determinación de estados
- Manejo de múltiples eventos
- Matriz de clasificación completa

### [4.4 Ventanas de Tiempo](./04-modulo-procesamiento-biometrico/04-ventanas-de-tiempo.md)
- Concepto de ventanas de tiempo
- Parámetros de configuración
- Visualización de ventanas
- Algoritmo de validación
- Casos de ejemplo
- Ventanas para múltiples períodos

---

## 5. Módulo de Reportes

### [5.1 Descripción General](./05-modulo-reportes/01-descripcion-general.md)
- Tipos de reportes implementados
- Componentes del módulo
- Single Source of Truth
- Métricas calculadas
- Endpoints de la API

### [5.2 Cálculos de Asistencia](./05-modulo-reportes/02-calculos-asistencia.md)
- Single Source of Truth
- Cálculo de minutos trabajados
- Cálculo de minutos de tardanza
- Cálculo de minutos de salida temprana
- Cálculo de minutos de horas extras
- Matriz de cálculos
- Agregaciones para reportes

### [5.3 Casos de Uso](./05-modulo-reportes/03-casos-de-uso.md)
- Diagrama de casos de uso
- Descripción de casos (CU-R01 a CU-R08)
- Matriz de actores vs. casos de uso

### [5.4 Generación de PDF](./05-modulo-reportes/04-generacion-de-pdf.md)
- Arquitectura de generación
- Proceso de generación
- Estructura del PDF generado
- Template HTML
- Configuración de Puppeteer
- Performance y optimizaciones

---

## 6. Integración con Dispositivos Biométricos

### [6.1 Integración ZKTeco](./06-integracion-dispositivos/01-integracion-zkteco.md)
- Arquitectura de integración
- Características de los dispositivos
- Librería node-zklib
- Flujo de conexión
- Estructura de registro ZKTeco
- Gestión de dispositivos
- Mapeo de usuarios
- Manejo de errores

### [6.2 Sincronización de Datos](./06-integracion-dispositivos/02-sincronizacion-de-datos.md)
- Proceso de sincronización
- Job programado
- Flujo detallado
- Prevención de duplicados
- Transformación de datos
- Estados de sincronización
- Monitoreo y alertas
- Sincronización manual

---

## 7. Seguridad y Autenticación

### [7. Seguridad y Autenticación](./07-seguridad-y-autenticacion.md)
- Arquitectura de seguridad
- Flujo de autenticación
- Configuración de Keycloak
- Protección de endpoints
- Gestión de sesiones en frontend
- Token management
- Auditoría y logging
- Consideraciones de seguridad

---

## 8. Tecnologías Utilizadas

### [8. Tecnologías Utilizadas](./08-tecnologias-utilizadas.md)
- Stack tecnológico completo
- Tabla detallada de tecnologías
- Patrones de diseño implementados
- Arquitectura de despliegue
- Justificación de selección tecnológica

---

## 9. Conclusiones

### [9. Conclusiones](./09-conclusiones.md)
- Cumplimiento de objetivos
- Métricas de éxito
- Lecciones aprendidas arquitectónicas
- Recomendaciones futuras
- Consideraciones finales

---

## Tecnologías del Proyecto

| Capa | Tecnología |
|------|------------|
| Backend | NestJS 11 + TypeScript |
| Base de Datos | PostgreSQL + TypeORM |
| Frontend | Next.js 15 + React 19 |
| UI | Tailwind CSS + Shadcn/ui |
| Estado Servidor | TanStack Query |
| Autenticación | Keycloak + NextAuth |
| Biométricos | node-zklib (ZKTeco) |
| PDF | Puppeteer |

---

## Objetivos del Proyecto

1. ✅ **Asistencia en Tiempo Real**: Dashboard que permitió visualizar marcaciones al momento.

2. ✅ **Procesamiento Automático**: Motor que transformó registros biométricos sin intervención manual.

3. ✅ **Reportes Confiables**: Sistema de generación de reportes con exportación a PDF.

---
