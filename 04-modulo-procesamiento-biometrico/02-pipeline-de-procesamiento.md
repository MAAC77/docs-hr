# 4.2 Pipeline de Procesamiento Biométrico

El motor de procesamiento implementó un pipeline de **6 pasos secuenciales** que transformaron los registros crudos de los dispositivos biométricos en sesiones de asistencia normalizadas.

---

## 4.2.1 Vista General del Pipeline

```mermaid
flowchart TD
    subgraph Pipeline["AttendanceEngineService Pipeline"]
        direction TB
        P1["STEP 1: LoadData<br/>Cargar configuraciones"]
        P2["STEP 1.5: EnsureSessions<br/>Crear sesiones por período"]
        P3["STEP 2: FilterEvents<br/>Validar ventanas de tiempo"]
        P4["STEP 3: ClassifyEvents<br/>Determinar ENTRY vs EXIT"]
        P5["STEP 4: PersistEvents<br/>Guardar eventos"]
        P6["STEP 5: CalculateSessions<br/>Actualizar estados"]
        P7["STEP 6: MarkProcessed<br/>Marcar procesados"]
    end

    subgraph Input["INPUT"]
        I["Registros Dispositivo"]
    end

    subgraph Output["OUTPUT"]
        O["Sesiones de Asistencia"]
    end

    subgraph DB["PostgreSQL"]
        DB[(datos)]
    end

    I --> P1
    P1 --> P2
    P2 --> P3
    P3 --> P4
    P4 --> P5
    P5 --> P6
    P6 --> P7
    P7 --> O

    P1 -.->|consultas| DB
    P5 -.->|INSERT| DB
    P6 -.->|UPDATE| DB
    P7 -.->|UPDATE| DB

    style P1 fill:#e3f2fd
    style P2 fill:#e3f2fd
    style P3 fill:#fff3e0
    style P4 fill:#fff3e0
    style P5 fill:#e8f5e9
    style P6 fill:#e8f5e9
    style P7 fill:#fce4ec
```

---

## 4.2.2 Paso 1: LoadData (Carga de Datos)

**Objetivo:** Cargar todas las configuraciones y datos necesarios para el procesamiento.

```mermaid
flowchart TD
    START([Inicio]) --> L1[Cargar ScheduleAssignment activa]
    L1 --> L2[Resolver versión de Schedule]
    L2 --> L3[Cargar SchedulePeriods del día]
    L3 --> L4[Cargar eventos existentes]
    L4 --> L5[Cargar sesiones existentes]
    L5 --> L6[Cargar días festivos]
    L6 --> L7[Cargar permisos aprobados]
    L7 --> END([Datos cargados en contexto])

    style START fill:#c8e6c9
    style END fill:#c8e6c9
```

**Consultas Realizadas:**

| Consulta | Propósito |
|----------|-----------|
| `ScheduleAssignment` por usuario y fecha | Obtener horario asignado |
| `ScheduleVersion` | Resolver versión histórica del horario |
| `SchedulePeriod` por día de semana | Obtener períodos del día |
| `AttendanceEvent` existentes | Evitar duplicados |
| `AttendanceSession` existentes | Actualizar en lugar de crear |
| `Holiday` | Detectar días festivos |
| `LeaveRequest` aprobados | Detectar permisos |

**Early Exit:** Si no hay `ScheduleAssignment` activa, el processing termina inmediatamente.

---

## 4.2.3 Paso 1.5: EnsureSessions (Garantizar Sesiones)

**Objetivo:** Crear exactamente una sesión por período ANTES de procesar eventos.

```mermaid
flowchart TD
    START([Inicio]) --> E1{Para cada período}
    E1 -->|No existe sesión| E2[Crear nueva AttendanceSession]
    E1 -->|Ya existe sesión| E3[Usar sesión existente]
    E2 --> E4[Guardar en sesión en BD]
    E3 --> E4
    E4 --> M1[Agregar al sessionMap<br/>Map~periodUuid → sessionUuid~]
    M1 --> E5{¿Más períodos?}
    E5 -->|Sí| E1
    E5 -->|No| END([sessionMap completo])

    style START fill:#c8e6c9
    style END fill:#c8e6c9
```

**¿Por qué este paso?**

- Los eventos necesitan un `sessionUuid` válido al crearse
- Crear las sesiones primero evita problemas de llave foránea
- Permite actualizar sesiones existentes en lugar de recrearlas

**Estado Inicial de Sesiones Nuevas:**

```mermaid
stateDiagram-v2
    [*] --> Creada
    Creada --> INCOMPLETE: Estado inicial
    INCOMPLETE --> NO_ENTRY: entryStatus
    INCOMPLETE --> NO_EXIT: exitStatus
```

---

## 4.2.4 Paso 2: FilterEvents (Filtrar Eventos)

**Objetivo:** Validar que los registros estén dentro de las ventanas de tiempo permitidas.

```mermaid
flowchart TD
    START([Inicio]) --> F1{Para cada registro}
    F1 --> F2{¿Está en ventana de entrada?}
    F2 -->|Sí| F3[Válido como ENTRY]
    F2 -->|No| F4{¿Está en ventana de salida?}
    F4 -->|Sí| F5[Válido como EXIT]
    F4 -->|No| F6[Rechazado: Fuera de ventana]
    F3 --> F7{¿Más registros?}
    F5 --> F7
    F6 --> SKIPPED[Agregar a skippedRecords]
    SKIPPED --> F7
    F7 -->|Sí| F1
    F7 -->|No| END([Registros filtrados])

    style START fill:#c8e6c9
    style END fill:#c8e6c9
    style SKIPPED fill:#ffcdd2
```

**Lógica de Validación:**

```mermaid
flowchart LR
    subgraph Period["Período: 08:00 - 17:00"]
        ME[06:00<br/>minEntry]
        MS[08:00<br/>startTime]
        MXE[10:00<br/>maxEntry]
        MXS[15:00<br/>minExit]
        ME2[17:00<br/>endTime]
        MXE2[19:00<br/>maxExit]
    end

    subgraph VentanaE["Ventana de Entrada"]
        VE["━━━ ENTRY ━━━"]
    end

    subgraph VentanaS["Ventana de Salida"]
        VS["━━━ EXIT ━━━"]
    end

    ME --> MS --> MXE
    MXS --> ME2 --> MXE2

    MS -.->.VentanaE
    MXE -.->.VentanaE
    MXS -.->.VentanaS
    MXE2 -.->.VentanaS

    style VentanaE fill:#c8e6c9
    style VentanaS fill:#bbdefb
```

**Ejemplos de Clasificación:**

| Hora | Ventana de Entrada (06-10) | Ventana de Salida (15-19) | Resultado |
|------|----------------------------|---------------------------|-----------|
| 07:30 | ✅ Dentro | ❌ Fuera | Válido ENTRY |
| 11:00 | ❌ Fuera | ❌ Fuera | ⚠️ Rechazado |
| 16:30 | ❌ Fuera | ✅ Dentro | Válido EXIT |
| 20:00 | ❌ Fuera | ❌ Fuera | ⚠️ Rechazado |

---

## 4.2.5 Paso 3: ClassifyEvents (Clasificar Eventos)

**Objetivo:** Determinar si cada registro válido es ENTRY o EXIT, y manejar casos especiales.

```mermaid
flowchart TD
    START([Inicio]) --> C1{Para cada registro válido}
    C1 --> C2{¿Es día festivo?}
    C2 -->|Sí| HOLIDAY[Marcar procesado<br/>No crear evento]
    C2 -->|No| C3{¿Tiene permiso?}
    C3 -->|Sí| LEAVE[Marcar procesado<br/>No crear evento]
    C3 -->|No| C4{¿En ventana de entrada?}
    C4 -->|Sí| ENTRY[Clasificar como ENTRY]
    C4 -->|No| C5{¿En ventana de salida?}
    C5 -->|Sí| EXIT[Clasificar como EXIT]
    C5 -->|No| ERROR[Error: Lógica inválida]
    HOLIDAY --> C6{¿Más registros?}
    LEAVE --> C6
    ENTRY --> C6
    EXIT --> C6
    ERROR --> C6
    C6 -->|Sí| C1
    C6 -->|No| END([Eventos clasificados])

    style START fill:#c8e6c9
    style END fill:#c8e6c9
    style HOLIDAY fill:#e1bee7
    style LEAVE fill:#ffe0b2
```

**Prioridad de Clasificación:**

1. **Día festivo** → Procesar sin evento
2. **Permiso aprobado** → Procesar sin evento
3. **Ventana de entrada** → ENTRY (prioridad)
4. **Ventana de salida** → EXIT

---

## 4.2.6 Paso 4: PersistEvents (Persistir Eventos)

**Objetivo:** Guardar los eventos clasificados en la base de datos.

```mermaid
flowchart LR
    subgraph Input["Eventos Clasificados"]
        E1[Eventos ENTRY]
        E2[Eventos EXIT]
    end

    subgraph Process["Batch Insert"]
        B1[Construir array]
        B2[Inserción en lote]
        B3[Retornar entidades creadas]
    end

    subgraph Output["AttendanceEvent[]"]
        EVT[Eventos persistidos con UUID]
    end

    E1 --> B1
    E2 --> B1
    B1 --> B2
    B2 --> B3
    B3 --> EVT

    style B2 fill:#c8e6c9
```

**Optimización:**

- **Batch insert**: Todos los eventos se insertaron en una sola transacción
- **Single transaction**: Garantizó consistencia (todo o nada)
- **Return entities**: Las entidades creadas incluyeron los UUIDs generados

---

## 4.2.7 Paso 5: CalculateSessions (Calcular Sesiones)

**Objetivo:** Calcular y actualizar los tres estados ortogonales de cada sesión.

```mermaid
flowchart TD
    START([Inicio]) --> S1{Para cada sesión}
    S1 --> S2[Obtener eventos del período]
    S2 --> S3[Separar ENTRY y EXIT]
    S3 --> S4[Calcular entryStatus]
    S4 --> S5[Calcular exitStatus]
    S5 --> S6[Calcular attendanceStatus]
    S6 --> S7[Calcular métricas<br/>worked, late, early_exit, overtime]
    S7 --> S8[UPDATE attendance_session]
    S8 --> S9{¿Más sesiones?}
    S9 -->|Sí| S1
    S9 -->|No| END([Sesiones actualizadas])

    style START fill:#c8e6c9
    style END fill:#c8e6c9
```

**Estados Ortogonales:**

```mermaid
flowchart TD
    subgraph Sesión["AttendanceSession"]
        AS[attendanceStatus]
        ES[entryStatus]
        XS[exitStatus]
    end

    subgraph AS_valores["COMPLETO / INCOMPLETO / AUSENCIA"]
    end

    subgraph ES_valores["ON_TIME / LATE / EARLY / NO_ENTRY"]
    end

    subgraph XS_valores["ON_TIME / EARLY_EXIT / OVERTIME / NO_EXIT"]
    end

    AS --> AS_valores
    ES --> ES_valores
    XS --> XS_valores

    style AS fill:#e8f5e9
    style ES fill:#fff3e0
    style XS fill:#e3f2fd
```

---

## 4.2.8 Paso 6: MarkProcessed (Marcar Procesados)

**Objetivo:** Actualizar el estado de procesamiento de los registros crudos.

```mermaid
flowchart TD
    START([Inicio]) --> M1{Para cada registro}
    M1 --> M2{¿Fue procesado exitosamente?}
    M2 -->|Sí| SUCCESS[UPDATE<br/>processed = true<br/>processed_at = now]
    M2 -->|No| SKIP[UPDATE<br/>processed = true<br/>processing_error = mensaje]
    SUCCESS --> M2{¿Más registros?}
    SKIP --> M2
    M2 -->|Sí| M1
    M2 -->|No| END([Registros marcados])

    style START fill:#c8e6c9
    style END fill:#c8e6c9
    style SUCCESS fill:#c8e6c9
    style SKIP fill:#ffcdd2
```

**Campos Actualizados:**

| Campo | Registros Exitosos | Registros Rechazados |
|-------|-------------------|---------------------|
| `processed` | `true` | `true` |
| `processed_at` | Timestamp actual | Timestamp actual |
| `processing_error` | `null` | Mensaje descriptivo |

---

## 4.2.9 Performance del Pipeline

### Métricas de Ejecución

| Paso | Queries | Tiempo Estimado |
|------|---------|-----------------|
| LoadData | ~5 | ~50ms |
| EnsureSessions | 0-2 | ~20ms |
| FilterEvents | 0 | ~1ms |
| ClassifyEvents | 0 | ~2ms |
| PersistEvents | 1 | ~15ms |
| CalculateSessions | ~2 | ~30ms |
| MarkProcessed | 2 | ~10ms |
| **Total** | **~14** | **~138ms** |

### Optimizaciones Implementadas

```mermaid
flowchart LR
    subgraph Opt["Optimizaciones"]
        O1[Batch Loading<br/>Single query por entidad]
        O2[Batch Insert<br/>Todos los eventos en una TX]
        O3[Idempotencia<br/>processed flag]
        O4[Parallel Processing<br/>Por usuario]
        O5[Single Source of Truth<br/>Cálculos centralizados]
    end

    subgraph Benefits["Beneficios"]
        B1[Elimina N+1 queries]
        B2[Reduce roundtrips a BD]
        B3[Evita duplicados]
        B4[Escalabilidad]
        B5[Consistencia]
    end

    O1 --> B1
    O2 --> B2
    O3 --> B3
    O4 --> B4
    O5 --> B5
```

---

[Siguiente: Clasificación de Eventos](./03-clasificacion-eventos.md) | [Anterior: Descripción General](./01-descripcion-general.md)
