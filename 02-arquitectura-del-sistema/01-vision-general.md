# 2.1 Visión General del Sistema

Esta sección presenta la arquitectura del Sistema de Gestión de Recursos Humanos con Integración Biométrica utilizando el modelo C4, que proporciona una vista jerárquica del sistema en diferentes niveles de abstracción.

---

## 2.1.1 Contexto del Sistema (C4 Level 1)

El sistema se integró con diversos actores y sistemas externos para su operación:

```mermaid
C4Context
    title Contexto del Sistema de Gestión de Asistencia Biométrica

    Person(docente, "Docente", "Usuario final que registra su asistencia")
    Person(administrativo, "Administrativo", "Usuario final que registra su asistencia")
    Person(rrhh, "Personal de RR.HH.", "Gestiona usuarios y reportes")
    Person(admin, "Administrador", "Configura el sistema")

    System_Boundary(hr_system, "Sistema de Gestión de RR.HH.") {
        System(sistema_web, "Sistema Web", "Aplicación web de gestión de asistencia")
    }

    System_Ext(keycloak, "Keycloak", "Servicio de autenticación y autorización")
    System_Ext(biometrico, "Dispositivos Biométricos", "Lectores de huella digital ZKTeco")
    System_Ext(payroll, "Sistema de Nómina", "Sistema externo para procesamiento de salarios")

    Rel(docente, sistema_web, "Accede a dashboard de asistencia", "HTTPS")
    Rel(administrativo, sistema_web, "Accede a dashboard de asistencia", "HTTPS")
    Rel(rrhh, sistema_web, "Genera reportes y gestiona usuarios", "HTTPS")
    Rel(admin, sistema_web, "Configura horarios y dispositivos", "HTTPS")

    Rel(sistema_web, keycloak, "Autentica usuarios", "OIDC")
    Rel(sistema_web, biometrico, "Sincroniza registros biométricos", "TCP/IP")
    Rel(sistema_web, payroll, "Exporta datos de asistencia", "CSV/PDF")
```

### Descripción de Actores y Sistemas

| Actor/Sistema | Descripción | Rol Principal |
|--------------|-------------|---------------|
| **Docente** | Personal docente de la institución | Registra asistencia mediante biométricos y consulta su estado |
| **Administrativo** | Personal administrativo | Registra asistencia y consulta su estado |
| **Personal de RR.HH.** | Gestores de recursos humanos | Genera reportes, gestiona usuarios y justificaciones |
| **Administrador** | Responsable de configuración | Configura horarios, dispositivos y parámetros del sistema |
| **Keycloak** | Servidor de identidad | Gestiona autenticación y autorización |
| **Dispositivos Biométricos** | Lectores ZKTeco | Capturan marcaciones de huella digital |
| **Sistema de Nómina** | Sistema externo | Procesa salarios basados en datos de asistencia |

---

## 2.1.2 Flujo de Datos de Alto Nivel

El siguiente diagrama muestra el flujo de información a través del sistema:

```mermaid
flowchart LR
    subgraph Dispositivos["Dispositivos Biométricos"]
        ZK1[ZKTeco 1]
        ZK2[ZKTeco 2]
        ZK3[ZKTeco N]
    end

    subgraph Backend["Backend API"]
        SYNC[Servicio de Sincronización]
        ENGINE[Motor de Procesamiento]
        API[REST API]
    end

    subgraph Frontend["Frontend Web"]
        DASH[Dashboard]
        REPORTS[Reportes]
        ADMIN[Administración]
    end

    subgraph Datos["Almacenamiento"]
        PG[(PostgreSQL)]
    end

    subgraph Auth["Autenticación"]
        KC[Keycloak]
    end

    ZK1 -->|TCP/IP| SYNC
    ZK2 -->|TCP/IP| SYNC
    ZK3 -->|TCP/IP| SYNC
    SYNC -->|Registros crudos| PG
    ENGINE -->|Lee y procesa| PG
    ENGINE -->|Registros procesados| PG
    KC <-->|OIDC| API
    DASH <-->|HTTP/JSON| API
    REPORTS <-->|HTTP/JSON| API
    ADMIN <-->|HTTP/JSON| API
    API <-->|SQL| PG
```

---

## 2.1.3 Principios Arquitectónicos

El sistema se diseñó siguiendo los siguientes principios arquitectónicos:

### 1. 100% Basado en Datos

Todos los cálculos se basaron en configuraciones explícitas de `ScheduleAssignment`. El sistema no asumió semanas de 5 días ni jornadas de 8 horas; todo fue parametrizable.

### 2. Única Fuente de Verdad (Single Source of Truth)

Todos los cálculos de asistencia se centralizaron en `/src/utils/reports/` del backend, garantizando consistencia en todo el sistema.

### 3. Arquitectura Normalizada

Cada sesión de asistencia mantuvo tres estados independientes (ortogonales):
- **Estado de asistencia:** COMPLETE, INCOMPLETE, ABSENCE, etc.
- **Estado de entrada:** ON_TIME, LATE, EARLY, NO_ENTRY
- **Estado de salida:** ON_TIME, EARLY_EXIT, OVERTIME, NO_EXIT

### 4. Trazabilidad Completa

Todas las entidades heredaron de `AuditEntity`, proporcionando campos de auditoría:
- Fecha de creación y actualización
- Usuario que creó/modificó
- Estado del registro (ACTIVE, INACTIVE, DELETED)
- Tipo de transacción (INSERT, UPDATE, UPSERT)

---

## 2.1.4 Patrones Arquitectónicos Implementados

### Domain-Driven Design (DDD)

El sistema se organizó por dominios de negocio con límites claros:

```mermaid
flowchart TD
    subgraph Dominios["Dominios de Negocio"]
        ATTEND[Asistencia]
        SCHED[Programaciones]
        USERS[Usuarios]
        DEPT[Departamentos]
        DEV[Dispositivos]
        RPT[Reportes]
    end

    ATTEND --> SCHED
    ATTEND --> USERS
    USERS --> DEPT
    ATTEND --> DEV
    RPT --> ATTEND
    RPT --> SCHED
```

### Repository Pattern

El acceso a datos se abstrajo mediante repositorios TypeORM, permitiendo cambiar la implementación de persistencia sin afectar la lógica de negocio.

### Service Layer Pattern

La lógica de negocio se aisló en servicios, mientras que los controladores solo manejaron concerns HTTP (rutas, validación de entrada, respuestas).

### Interceptor Pattern

Los aspectos transversales (transacciones, formateo de respuestas, logging) se manejaron mediante interceptores globales.

---

[Siguiente: Arquitectura Backend](./02-arquitectura-backend.md) | [Anterior: Introducción](/documentacion/01-introduccion.md)
