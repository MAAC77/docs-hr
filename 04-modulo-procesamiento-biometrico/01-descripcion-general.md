# 4.1 Descripción General del Módulo de Procesamiento Biométrico

Este módulo constituyó el segundo objetivo específico del proyecto: **implementar un módulo que permitió procesar automáticamente los datos biométricos de asistencia**, reduciendo la carga manual y garantizando información oportuna.

---

## 4.1.1 Propósito del Módulo

El módulo de procesamiento biométrico tuvo como finalidad:

1. **Automatizar la transformación** de registros crudos de dispositivos biométricos en información estructurada de asistencia.
2. **Normalizar los datos** provenientes de múltiples dispositivos en un formato unificado.
3. **Garantizar la idempotencia** para evitar procesamiento duplicado de registros.
4. **Proporcionar una única fuente de verdad** para todos los cálculos de asistencia.

---

## 4.1.2 El Motor de Procesamiento (AttendanceEngineService)

El componente central del módulo fue el **AttendanceEngineService**, un servicio diseñado como un pipeline de procesamiento secuencia que transformó los registros biométricos crudos en sesiones de asistencia normalizadas.

```mermaid
flowchart LR
    subgraph Input["Entrada"]
        DRR[DeviceRawRecord<br/>Registros crudos del dispositivo]
    end

    subgraph Engine["AttendanceEngineService"]
        P1[Step 1: LoadData<br/>Cargar configuraciones]
        P2[Step 2: FilterEvents<br/>Validar ventanas de tiempo]
        P3[Step 3: ClassifyEvents<br/>Clasificar ENTRY/EXIT]
        P4[Step 4: PersistEvents<br/>Guardar eventos]
        P5[Step 5: CalculateSessions<br/>Calcular estados]
        P6[Step 6: MarkProcessed<br/>Marcar procesados]
    end

    subgraph Output["Salida"]
        ASE[AttendanceSession<br/>Sesión normalizada]
        AEV[AttendanceEvent<br/>Eventos clasificados]
    end

    DRR --> P1 --> P2 --> P3 --> P4 --> P5 --> P6
    P4 --> AEV
    P5 --> ASE

    style DRR fill:#ffebee
    style ASE fill:#e8f5e9
    style Engine fill:#fff3e0
```

### Ventajas del Diseño en Pipeline

| Ventaja | Descripción |
|---------|-------------|
| **Modularidad** | Cada paso se pudo probar independientemente |
| **Mantenibilidad** | Los cambios en un paso no afectaron a los demás |
| **Escalabilidad** | El pipeline se pudo paralelizar por usuario |
| **Trazabilidad** | Cada paso dejó un registro de auditoría |
| **Performance** | Optimizaciones locales sin afectar el flujo global |

---

## 4.1.3 Componentes del Motor

### Context Object (Objeto de Contexto)

El pipeline utilizó un objeto de contexto para pasar datos entre pasos:

```mermaid
classDiagram
    class NormalizedAttendanceContext {
        +input: InputData
        +data: ReferenceData
        +persist: PersistentState
        +auditTrace: AuditTrail[]
    }

    class InputData {
        +userUuid: string
        +workDate: date
        +records: DeviceRawRecord[]
        +now: DateTime
    }

    class ReferenceData {
        +scheduleAssignments: ScheduleAssignment[]
        +schedulePeriods: SchedulePeriod[]
        +existingEvents: AttendanceEvent[]
        +existingSessions: AttendanceSession[]
        +holidays: Holiday[]
        +leaves: LeaveRequest[]
    }

    class PersistentState {
        +events: AttendanceEvent[]
        +sessionMap: Map~periodUuid, sessionUuid~
        +processedRecords: DeviceRawRecord[]
        +skippedRecords: DeviceRawRecord[]
    }

    class AuditTrail {
        +step: string
        +decision: string
        +reason: string
        +timestamp: DateTime
    }

    NormalizedAttendanceContext --> InputData
    NormalizedAttendanceContext --> ReferenceData
    NormalizedAttendanceContext --> PersistentState
    NormalizedAttendanceContext --> AuditTrail
```

**Propósito del Context Object:**

- Centralizó todos los datos necesarios para el procesamiento
- Permitió pasar información entre pasos del pipeline
- Mantuvo un registro de auditoría para debugging
- Facilitó las pruebas unitarias

---

## 4.1.4 Estados de Procesamiento de un Registro

```mermaid
stateDiagram-v2
    [*] --> Recibido: Dispositivo envía registro
    Recibido --> Pendiente: Almacenado en BD
    Pendiente --> Procesando: Motor toma el registro
    Procesando --> EventoCreado: Clasificado válido
    Procesando --> Festivo: Día festivo
    Procesando --> Justificado: Permiso aprobado
    Procesando --> Rechazado: Fuera de ventana
    EventoCreado --> Completado
    Festivo --> Completado
    Justificado --> Completado
    Rechazado --> Completado
    Completado --> [*]

    note right of Procesando
        El motor evalúa el
        registro y determina
        su destino
    end note

    note right of Rechazado
        Se almacena error
        para auditoría
    end note
```

---

## 4.1.5 Normalización de Datos

El sistema transformó los datos crudos de los dispositivos en un modelo normalizado:

```mermaid
flowchart TD
    subgraph Dispositivo["Dispositivo ZKTeco"]
        ZK1[user_sn: int]
        ZK2[record_time: timestamp]
        ZK3[device_id: string]
    end

    subgraph Crudo["DeviceRawRecord (Capa Cruda)"]
        DR1[uuid: UUID]
        DR2[device_user_sn: bigint]
        DR3[record_time: timestamptz]
        DR4[device_uuid: UUID]
        DR5[processed: boolean]
        DR6[unique_signature: string]
    end

    subgraph Evento["AttendanceEvent (Capa Normalizada)"]
        EV1[uuid: UUID]
        EV2[session_uuid: UUID]
        EV3[event_timestamp: timestamptz]
        EV4[event_type: ENTRY/EXIT]
        EV5[event_source: BIOMETRIC_DEVICE]
        EV6[confidence_score: decimal]
    end

    subgraph Sesion["AttendanceSession (Capa Negocio)"]
        SE1[uuid: UUID]
        SE2[work_date: date]
        SE3[attendance_status: COMPLETE/...]
        SE4[entry_status: ON_TIME/LATE/...]
        SE5[exit_status: ON_TIME/EARYLY_EXIT/...]
        SE6[worked_minutes: int]
        SE7[late_minutes: int]
    end

    ZK1 --> DR2
    ZK2 --> DR3
    ZK3 --> DR4
    DR1 --> EV2
    DR3 --> EV3
    DR4 --> EV5
    EV2 --> SE2
    EV3 --> SE6
    EV4 --> SE3
```

---

## 4.1.6 Características Técnicas del Motor

### Idempotencia

El motor garantizó que cada registro se procesara exactamente una vez:

- **uniqueSignature**: Hash compuesto por `device_uuid + user_sn + record_time`
- **Constraint único**: Previene inserción de duplicados
- **processed flag**: Marca registros ya procesados

### Soporte para Múltiples Períodos

El sistema manejó correctamente turnos partidos (ej: mañana + tarde):

- Cada período generó su propia `AttendanceSession`
- Los eventos se enrutaron al período correspondiente
- Los estados se calcularon independientemente por período

### Manejo de Casos Especiales

| Caso | Comportamiento del Motor |
|------|--------------------------|
| Día festivo | Marca registro procesado, no crea eventos, estado = HOLIDAY |
| Permiso aprobado | Marca registro procesado, no crea eventos, estado = JUSTIFIED |
| Registro fuera de ventana | Marca procesado con error, no crea eventos |
| Múltiples entradas | Usa la primera para estado, ignora el resto |
| Múltiples salidas | Usa la última para estado, ignora el resto |

---

[Siguiente: Pipeline de Procesamiento](./02-pipeline-de-procesamiento.md) | [Anterior: Procedimiento Aplicativo](../../03-modulo-asistencia-en-tiempo-real/04-procedimiento-aplicativo.md)
