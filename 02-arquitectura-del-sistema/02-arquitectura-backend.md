# 2.2 Arquitectura del Backend

El backend se implementó utilizando **NestJS 11** con **TypeScript 5.7**, siguiendo una arquitectura modular basada en Domain-Driven Design (DDD).

---

## 2.2.1 Vista de Contenedores (C4 Level 2)

```mermaid
C4Container
    title Arquitectura Backend - Vista de Contenedores

    Container(api, "API REST", "NestJS 11 + TypeScript", "Expone endpoints REST para consumo del frontend")
    ContainerDb(db, "Base de Datos", "PostgreSQL 15+ + TypeORM", "Almacena todos los datos del sistema")
    Container_Ext(keycloak, "Keycloak", "Proveedor de Identidad", "Gestiona autenticación y autorización")
    Container_Ext(zkteco, "Dispositivos ZKTeco", "Biométricos", "Capturan marcaciones de asistencia")

    Rel(api, db, "Lee/Escribe datos", "TypeORM + PostgreSQL Driver")
    Rel(api, keycloak, "Valida tokens", "OAuth 2.0 / OIDC")
    Rel(api, zkteco, "Sincroniza registros", "node-zklib (TCP/IP)")
```

---

## 2.2.2 Vista de Componentes (C4 Level 3)

```mermaid
C4Component
    title Arquitectura Backend - Vista de Componentes

    Container_Boundary(api, "API REST") {
        Component(guards, "Guards", "Validación de autenticación y autorización", "KeycloakGuard, RolesGuard")
        Component(interceptors, "Interceptors", "Aspectos transversales", "TransactionInterceptor, ResponseInterceptor")
        Component(controllers, "Controllers", "Manejadores de peticiones HTTP", "AttendanceController, ReportsController, etc.")
        Component(services, "Services", "Lógica de negocio", "AttendanceEngineService, ReportService, etc.")
        Component(repositories, "Repositories", "Acceso a datos", "TypeORM Repositories")
        Component(engine, "Motor de Asistencia", "Procesamiento de registros biométricos", "AttendanceEngineService")
        Component(utils, "Utilidades de Cálculo", "Funciones puras de cálculo", "attendance-calculations.ts")
    }

    Container_Boundary(db, "Base de Datos") {
        ComponentDb(tables, "Tablas", "Entidades persistidas", "User, AttendanceSession, AttendanceEvent, etc.")
    }

    Rel(guards, controllers, "Protege rutas")
    Rel(controllers, interceptors, "Pre/Post-procesa")
    Rel(interceptors, services, "Invoca con transacción")
    Rel(services, repositories, "Consulta/Actualiza")
    Rel(services, engine, "Procesa biométricos")
    Rel(engine, utils, "Calcula métricas")
    Rel(repositories, tables, "Lee/Escribe")
    Rel(guards, keycloak, "Valida tokens")
```

---

## 2.2.3 Estructura de Módulos

El backend se organizó en módulos por dominio de negocio:

```
hr-backend/src/
├── auth/                    # Autenticación con Keycloak
├── attendance/              # Control de asistencia y reportes
│   ├── controller/          # Controladores REST
│   ├── service/             # Servicios de negocio
│   │   └── engine/          # Motor de procesamiento biométrico
│   ├── entity/              # Entidades de base de datos
│   ├── dto/                 # Data Transfer Objects
│   └── jobs/                # Tareas programadas
├── users/                   # Gestión de usuarios
├── department/              # Estructura organizacional
├── schedule/                # Horarios de trabajo
├── device/                  # Gestión de dispositivos biométricos
├── holiday/                 # Calendario y días festivos
├── reports/                 # Generación de reportes
├── payroll/                 # Integración con nómina
├── common/                  # Componentes compartidos
│   ├── controllers/         # BaseController
│   ├── service/             # BaseService
│   ├── entity/              # AuditEntity
│   ├── interceptors/        # Interceptors globales
│   └── dto/                 # DTOs comunes
└── utils/                   # Utilidades
    └── reports/             # Cálculos centralizados
```

---

## 2.2.4 Ciclo de Vida de una Petición

El siguiente diagrama muestra el flujo completo de una petición HTTP a través del backend:

```mermaid
sequenceDiagram
    actor C as Cliente (Frontend)
    participant G as Keycloak Guard
    participant P as Pipes
    participant I as Interceptors
    participant Ctrl as Controller
    participant Svc as Service
    participant Repo as Repository
    participant DB as PostgreSQL

    C->>G: 1. HTTP Request + Bearer Token
    G->>G: 2. Valida token con Keycloak
    G->>G: 3. Verifica roles/permisos
    G->>P: 4. Request validada
    P->>P: 5. Valida y transforma DTO
    P->>I: 6. Request validada
    I->>I: 7. Inicia transacción (@Transactional)
    I->>Ctrl: 8. Request procesada
    Ctrl->>Svc: 9. Invoca lógica de negocio
    Svc->>Repo: 10. Consulta/Actualiza datos
    Repo->>DB: 11. Ejecuta SQL
    DB-->>Repo: 12. Resultado
    Repo-->>Svc: 13. Entidades
    Svc-->>Ctrl: 14. Datos de negocio
    Ctrl-->>I: 15. Respuesta
    I->>I: 16. Commit transacción
    I->>I: 17. Formatea respuesta (ApiResponseDto)
    I-->>C: 18. JSON Response
```

### Descripción de Componentes

| Componente | Responsabilidad |
|-----------|-----------------|
| **Guards** | Validan que el cliente esté autenticado y tenga los permisos necesarios |
| **Pipes** | Validan y transforman los datos de entrada según los DTOs |
| **Interceptors** | Manejan transacciones, formatean respuestas y registran logs |
| **Controllers** | Exponen endpoints REST y coordinan la respuesta |
| **Services** | Contienen la lógica de negocio del dominio |
| **Repositories** | Abstraen el acceso a datos mediante TypeORM |
| **PostgreSQL** | Almacena persistentemente todos los datos |

---

## 2.2.5 Transaccionalidad

El sistema utilizó `@nestjs-cls/transactional` para garantizar la integridad de los datos:

```mermaid
flowchart TD
    subgraph Request["Petición HTTP"]
        A[Controller]
    end

    subgraph Transaction["Contexto Transaccional"]
        B[Service 1]
        C[Service 2]
        D[Service 3]
    end

    subgraph Database["Base de Datos"]
        E[(PostgreSQL)]
    end

    A -->|"@Transactional"| B
    B --> C
    C --> D
    B --> E
    C --> E
    D --> E

    F[Si todo OK] --> G[Commit]
    H[Si error] --> I[Rollback]

    style F fill:#90EE90
    style H fill:#FFB6C1
```

**Características:**

- Todas las operaciones dentro de un contexto transaccional se ejecutan en una sola transacción de base de datos.
- Si cualquier operación falla, todas las cambios se revierten automáticamente (rollback).
- El commit solo se realiza al finalizar exitosamente todo el flujo.

---

## 2.2.6 Modelo de Auditoría

Todas las entidades del sistema heredaron de `AuditEntity`:

```mermaid
classDiagram
    class AuditEntity {
        +uuid: string
        +status: EntityStatus
        +transactionType: TransactionType
        +createdBy: string
        +createdAt: DateTime
        +updatedBy: string
        +updatedAt: DateTime
        +version: number
    }

    class User {
        +email: string
        +fullName: string
        +documentNumber: string
    }

    class AttendanceSession {
        +workDate: Date
        +attendanceStatus: AttendanceStatus
        +entryStatus: EntryStatus
        +exitStatus: ExitStatus
    }

    class DeviceRawRecord {
        +userSn: BigInteger
        +recordTime: DateTime
        +processed: Boolean
    }

    AuditEntity <|-- User
    AuditEntity <|-- AttendanceSession
    AuditEntity <|-- DeviceRawRecord
```

**Campos de Auditoría:**

| Campo | Descripción |
|-------|-------------|
| `status` | Estado del registro (ACTIVE, INACTIVE, DELETED) |
| `transactionType` | Tipo de operación (INSERT, UPDATE, UPSERT) |
| `createdBy` | Usuario que creó el registro |
| `createdAt` | Fecha y hora de creación |
| `updatedBy` | Último usuario que modificó el registro |
| `updatedAt` | Fecha y hora de última actualización |
| `version` | Versión para control de concurrencia optimista |

---

[Siguiente: Arquitectura Frontend](./03-arquitectura-frontend.md) | [Anterior: Visión General](./01-vision-general.md)
