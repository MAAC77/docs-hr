# 3.3 Flujo de Datos del Módulo de Asistencia en Tiempo Real

Esta sección documenta el flujo completo de datos desde que el usuario marca su huella digital hasta que la información se visualiza en el dashboard.

---

## 3.3.1 Diagrama de Secuencia Completo

```mermaid
sequenceDiagram
    autonumber
    actor U as Usuario
    participant ZK as Dispositivo ZKTeco
    participant S as Servicio de Sincronización
    participant E as Motor de Procesamiento
    participant DB as PostgreSQL
    participant Q as TanStack Query Cache
    participant C as Componente React

    Note over U,C: 1. Marcación Biométrica
    U->>ZK: Coloca huella digital
    ZK->>ZK: Valida huella
    ZK->>ZK: Almacena registro localmente

    Note over U,C: 2. Sincronización (automática cada 5 min)
    S->>ZK: Conecta vía TCP/IP
    ZK-->>S: Registros nuevos
    S->>DB: INSERT device_raw_records

    Note over U,C: 3. Procesamiento Automático
    E->>DB: SELECT registros sin procesar
    E->>E: Carga configuraciones
    E->>E: Clasifica eventos (ENTRY/EXIT)
    E->>DB: INSERT attendance_events
    E->>DB: UPDATE attendance_sessions

    Note over U,C: 4. Visualización en Tiempo Real
    C->>Q: Verifica cache
    alt Cache stale o invalidado
        Q->>DB: SELECT asistencia del día
        DB-->>Q: Datos actualizados
        Q-->>C: Nuevo estado
    end
    C->>C: Re-renderiza con nuevos datos
    C-->>U: Muestra marcación actualizada
```

---

## 3.3.2 Flujo Detallado de Consulta del Dashboard

```mermaid
sequenceDiagram
    autonumber
    actor U as Usuario
    participant C as TodayStatusCard
    participant Q as useQuery Hook
    participant API as API Client
    participant G as Keycloak Guard
    participant Ctrl as AttendanceController
    participant Svc as ReportService
    participant R as AttendanceRepository
    participant DB as PostgreSQL

    Note over U,DB: Usuario abre el dashboard
    U->>C: Monta componente
    C->>Q: useQuery(["attendance-reports", "today"])

    Note over Q,DB: Primera carga
    Q->>Q: Verifica cache (miss)
    Q->>API: getTodayAttendanceReport()
    API->>API: Agrega Bearer Token
    API->>G: GET /attendance/reports/me?date=today
    G->>G: Valida token

    G->>Ctrl: Request autorizada
    Ctrl->>Svc: getTodayReport(userUuid, date)
    Svc->>R: findSessionsByDate(userUuid, date)
    R->>DB: SELECT * FROM attendance_sessions WHERE...

    DB-->>R: Filas de AttendanceSession
    R-->>Svc: Entidades AttendanceSession
    Svc->>Svc: Calcula métricas totales

    Svc-->>Ctrl: AttendanceReportDto
    Ctrl-->>API: JSON Response
    API-->>Q: ApiResponseDto
    Q->>Q: Almacena en cache
    Q-->>C: { data, isLoading, error }

    Note over U,C: Renderizado inicial
    C->>C: Renderiza estado, horas, alertas
    C-->>U: Dashboard visible

    Note over U,DB: Actualización automática (2 min)
    Q->>Q: refetchInterval activado
    Q->>API: Refresca datos (mismo flujo)
```

---

## 3.3.3 Flujo de Actualización en Tiempo Real

El sistema implementó actualización en tiempo real mediante **invalidación de cache**:

```mermaid
flowchart TD
    subgraph Eventos["Eventos que Trigger Actualización"]
        E1[Dispositivo sincroniza registros]
        E2[Motor procesa nuevos registros]
        E3[Usuario cambia fecha en filtro]
        E4[Ventana gana foco]
    end

    subgraph Cache["TanStack Query Cache"]
        Q[Query Cache<br/>Key: attendance-reports]
    end

    subgraph API["Backend API"]
        EP[GET /attendance/reports/me]
    end

    subgraph UI["Componente React"]
        COMP[TodayStatusCard]
    end

    E1 -->|trigger| Q
    E2 -->|invalidar| Q
    E3 -->|refetch| Q
    E4 -->|refetchOnWindowFocus| Q

    Q -->|stale| EP
    EP -->|datos actualizados| Q
    Q -->|nuevo estado| COMP

    style Q fill:#fff3e0
    style COMP fill:#e3f2fd
```

### Estrategia de Refresco

| Evento | Acción | Configuración |
|--------|--------|---------------|
| **Montaje del componente** | Carga inicial | `staleTime: 30s` |
| **Ventana gana foco** | Refresco automático | `refetchOnWindowFocus: true` |
| **Intervalo periódico** | Refresco cada 2 min | `refetchInterval: 120000ms` |
| **Acción del usuario** | Invalidación manual | `queryClient.invalidateQueries()` |
| **Conexión restablecida** | Reintento automático | `retry: 3` |

---

## 3.3.4 Clas Type Diagram: Componentes del Módulo

```mermaid
classDiagram
    class TodayStatusCard {
        +date: Date
        +userUuid: string
        +useQuery()
        +render()
        +calculateProgress()
    }

    class NextActionAlert {
        +attendanceStatus: AttendanceStatus
        +entryStatus: EntryStatus
        +exitStatus: ExitStatus
        +getMessage()
        +getSeverity()
    }

    class TodayPeriodsList {
        +periods: SessionWithPeriod[]
        +renderPeriod()
        +getStatusIcon()
        +getStatusColor()
    }

    class EventsTimeline {
        +events: AttendanceEvent[]
        +renderTimeline()
        +formatEventTime()
        +getEventIcon()
    }

    class useAttendanceReport {
        +date: Date
        +queryKey: string
        +queryFn: function
        +staleTime: number
        +refetchInterval: number
        +data: AttendanceReport
        +isLoading: boolean
        +error: Error
    }

    class AttendanceReport {
        +workDate: Date
        +totalWorkedMinutes: number
        +totalLateMinutes: number
        +totalEarlyExitMinutes: number
        +totalOvertimeMinutes: number
        +sessions: AttendanceSession[]
    }

    TodayStatusCard --> useAttendanceReport : uses
    TodayStatusCard --> NextActionAlert : renders
    TodayStatusCard --> TodayPeriodsList : renders
    TodayStatusCard --> EventsTimeline : renders
    useAttendanceReport --> AttendanceReport : returns
```

---

## 3.3.5 Flujo de Datos de una Marcación Individual

```mermaid
stateDiagram-v2
    [*] --> Dispositivo: Usuario marca huella
    Dispositivo --> RegistroCrudo: Almacena localmente
    RegistroCrudo --> RegistroCrudo: Espera sincronización
    RegistroCrudo --> Sincronizado: Servicio sincroniza
    Sincronizado --> Procesando: Motor de procesamiento
    Procesando --> Clasificado: ENTRY o EXIT
    Clasificado --> EventoCreado: AttendanceEvent
    EventoCreado --> SessionActualizada: AttendanceSession
    SessionActualizada --> CacheActualizada: TanStack Query
    CacheActualizada --> UIDashboard: Componente React
    UIDashboard --> UsuarioVisible: Usuario ve marcación
    UsuarioVisible --> [*]
```

---

## 3.3.6 Transformación de Datos

Los datos sufrieron múltiples transformaciones desde su origen hasta la visualización:

```mermaid
flowchart LR
    subgraph Capa1["Capa 1: Dispositivo"]
        D1[Registro ZKTeco<br/>user_sn, record_time]
    end

    subgraph Capa2["Capa 2: Base de Datos"]
        D2[DeviceRawRecord<br/>uuid, processed, unique_signature]
        D3[AttendanceEvent<br/>event_type, event_timestamp]
        D4[AttendanceSession<br/>states, metrics]
    end

    subgraph Capa3["Capa 3: API"]
        D5[AttendanceReportDto<br/>aggregated data]
    end

    subgraph Capa4["Capa 4: Frontend"]
        D6[AttendanceReport<br/>TypeScript interface]
        D7[UI Components<br/>formatted display]
    end

    D1 --> D2
    D2 --> D3
    D3 --> D4
    D4 --> D5
    D5 --> D6
    D6 --> D7

    style D1 fill:#ffebee
    style D2 fill:#fff3e0
    style D3 fill:#e8f5e9
    style D4 fill:#e8f5e9
    style D5 fill:#e3f2fd
    style D6 fill:#f3e5f5
    style D7 fill:#fce4ec
```

### Transformaciones por Capa

| Capa | Transformación |
|------|---------------|
| **Dispositivo** | Datos crudos del protocolo ZKTeco |
| **Ingesta** | Mapeo a `DeviceRawRecord` con `uniqueSignature` |
| **Procesamiento** | Clificación a `AttendanceEvent` y agregación en `AttendanceSession` |
| **API** | Transformación a DTOs para transferencia HTTP |
| **Frontend** | Mapeo a interfaces TypeScript con validación |
| **UI** | Formateo para display (horas legibles, colores, iconos) |

---

[Siguiente: Procedimiento Aplicativo](./04-procedimiento-aplicativo.md) | [Anterior: Casos de Uso](./02-casos-de-uso.md)
