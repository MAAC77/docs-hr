# 4.3 Clasificación de Eventos Biométricos

Esta sección documenta la lógica mediante la cual el sistema clasificó cada registro biométrico como entrada (ENTRY) o salida (EXIT), y cómo se manejaron los casos especiales.

---

## 4.3.1 Tipos de Eventos

El sistema reconoció tres tipos de eventos:

```mermaid
flowchart TD
    subgraph Tipos["Tipos de Eventos"]
        E[ENTRY<br/>Entrada al trabajo]
        X[EXIT<br/>Salida del trabajo]
        I[INFERENCE<br/>Evento inferido por el sistema]
    end

    subgraph Fuentes["Fuentes de Eventos"]
        F1[BIOMETRIC_DEVICE<br/>Dispositivo ZKTeco]
        F2[MANUAL_ENTRY<br/>Ingreso manual por RR.HH.]
        F3[SYSTEM_GENERATED<br/>Generado automáticamente]
    end

    E --> F1
    E --> F2
    X --> F1
    X --> F2
    I --> F3

    style E fill:#c8e6c9
    style X fill:#ffcdd2
    style I fill:#fff3e0
```

---

## 4.3.2 Algoritmo de Clasificación

### Diagrama de Decisión

```mermaid
flowchart TD
    START([Registro biométrico]) --> CHECK{¿El registro tiene<br/>ventanas válidas?}
    CHECK -->|No| REJECT[RECHAZADO<br/>Fuera de ventana]
    CHECK -->|Sí| HOLIDAY{¿Es día<br/>festivo?}
    HOLIDAY -->|Sí| H[PROCESADO<br/>Estado: HOLIDAY]
    HOLIDAY -->|No| LEAVE{¿Tiene<br/>permiso?}
    LEAVE -->|Sí| J[PROCESADO<br/>Estado: JUSTIFIED]
    LEAVE -->|No| WINDOW{¿En qué<br/>ventana está?}
    WINDOW -->|Ventana entrada| ENTRY[ENTRY<br/>event_type = 'ENTRY']
    WINDOW -->|Ventana salida| EXIT[EXIT<br/>event_type = 'EXIT']
    WINDOW -->|Ninguna| ERROR[ERROR<br/>Ventana inválida]

    style ENTRY fill:#c8e6c9
    style EXIT fill:#ffcdd2
    style H fill:#e1bee7
    style J fill:#ffe0b2
    style REJECT fill:#f5f5f5
```

### Prioridad de Clasificación

| Orden | Condición | Resultado |
|-------|-----------|-----------|
| 1 | Día festivo | Procesar sin evento (estado HOLIDAY) |
| 2 | Permiso aprobado | Procesar sin evento (estado JUSTIFIED) |
| 3 | Dentro de ventana de entrada | Clasificar como ENTRY |
| 4 | Dentro de ventana de salida | Clasificar como EXIT |
| 5 | Fuera de ambas ventanas | Rechazar con error |

---

## 4.3.3 Ventanas de Clasificación

Las ventanas de tiempo se configuraron por período de horario:

```mermaid
gantt
    title Ventanas de Tiempo para Clasificación
    dateFormat HH:mm
    axisFormat %H:%M

    section Período
    Horario laboral    :08:00, 540m

    section Ventana Entrada
    Inicio ventana     :06:00, 240m
    Tolera ON_TIME     :07:45, 30m
    Tolera LATE        :08:15, 105m

    section Ventana Salida
    Inicio ventana     :15:00, 240m
    Tolera ON_TIME     :16:45, 30m
    Tolera OVERTIME    :17:15, 105m
```

### Parámetros de Ventana

| Parámetro | Descripción | Ejemplo |
|-----------|-------------|---------|
| `startTime` | Hora oficial de inicio | 08:00 |
| `endTime` | Hora oficial de fin | 17:00 |
| `minEntry` | Hora mínima permitida para entrada | 06:00 |
| `maxEntry` | Hora máxima permitida para entrada | 10:00 |
| `minExit` | Hora mínima permitida para salida | 15:00 |
| `maxExit` | Hora máxima permitida para salida | 19:00 |
| `toleranceMinutes` | Minutos de tolerancia para puntualidad | 15 |

---

## 4.3.4 Determinación de Estados

### Estado de Entrada (entryStatus)

```mermaid
flowchart LR
    subgraph Tolerancia["Ventana de Tolerancia de Entrada"]
        T1[07:45<br/>Inicio ON_TIME]
        T2[08:00<br/>Hora oficial]
        T3[08:15<br/>Fin ON_TIME]
    end

    subgraph Estados["Estados Posibles"]
        E1[EARLY<br/>Antes de 07:45]
        E2[ON_TIME<br/>07:45 - 08:15]
        E3[LATE<br/>Después de 08:15]
    end

    T1 -.->.E1
    T1 -.->.E2
    T3 -.->.E2
    T3 -.->.E3

    style E1 fill:#e3f2fd
    style E2 fill:#c8e6c9
    style E3 fill:#ffcdd2
```

### Tabla de Estados de Entrada

| Hora de Marcación | vs Hora Oficial | vs Tolerancia (±15) | Estado |
|-------------------|-----------------|---------------------|--------|
| 07:30 | Antes de 08:00 | Antes de 07:45 | **EARLY** |
| 07:50 | Antes de 08:00 | Dentro de ±15 | **ON_TIME** |
| 08:10 | Después de 08:00 | Dentro de ±15 | **ON_TIME** |
| 08:30 | Después de 08:00 | Después de 08:15 | **LATE** |

### Estado de Salida (exitStatus)

```mermaid
flowchart LR
    subgraph Tolerancia["Ventana de Tolerancia de Salida"]
        T1[16:45<br/>Inicio ON_TIME]
        T2[17:00<br/>Hora oficial]
        T3[17:15<br/>Fin ON_TIME]
    end

    subgraph Estados["Estados Posibles"]
        E1[EARLY_EXIT<br/>Antes de 16:45]
        E2[ON_TIME<br/>16:45 - 17:15]
        E3[OVERTIME<br/>Después de 17:15]
    end

    T1 -.->.E1
    T1 -.->.E2
    T3 -.->.E2
    T3 -.->.E3

    style E1 fill:#ffe0b2
    style E2 fill:#c8e6c9
    style E3 fill:#e3f2fd
```

### Tabla de Estados de Salida

| Hora de Marcación | vs Hora Oficial | vs Tolerancia (±15) | Estado |
|-------------------|-----------------|---------------------|--------|
| 16:30 | Antes de 17:00 | Antes de 16:45 | **EARLY_EXIT** |
| 16:50 | Antes de 17:00 | Dentro de ±15 | **ON_TIME** |
| 17:10 | Después de 17:00 | Dentro de ±15 | **ON_TIME** |
| 17:30 | Después de 17:00 | Después de 17:15 | **OVERTIME** |

---

## 4.3.5 Manejo de Múltiples Eventos

### Múltiples Entradas en el Mismo Período

```mermaid
flowchart TD
    A[Evento 1: 07:55 ENTRY] --> B[Evento 2: 08:05 ENTRY]
    B --> C[Evento 3: 08:30 ENTRY]

    A --> USAR[Usar para estado]
    B --> IGNORAR[Ignorar]
    C --> IGNORAR

    style USAR fill:#c8e6c9
    style IGNORAR fill:#f5f5f5
```

**Regla:** Se utilizó la **PRIMERA entrada** para calcular el estado de entrada. Las entradas adicionales se registraron pero no afectaron el estado.

### Múltiples Salidas en el Mismo Período

```mermaid
flowchart TD
    A[Evento 1: 16:30 EXIT] --> B[Evento 2: 17:00 EXIT]
    B --> C[Evento 3: 18:00 EXIT]

    A --> IGNORAR[Ignorar]
    B --> IGNORAR
    C --> USAR[Usar para estado]

    style USAR fill:#c8e6c9
    style IGNORAR fill:#f5f5f5
```

**Regla:** Se utilizó la **ÚLTIMA salida** para calcular el estado de salida. Las salidas adicionales se registraron pero no afectaron el estado.

---

## 4.3.6 Casos Especiales de Clasificación

### Entrada sin Salida Correspondiente

```mermaid
stateDiagram-v2
    [*] --> ENTRY_REGISTRADA: Se marca entrada
    ENTRY_REGISTRADA --> ESPERANDO_SALIDA: Dentro de ventana
    ESPERANDO_SALIDA --> INCOMPLETO: Ventana de salida cierra
    INCOMPLETO --> [*]: Sistema genera MISSING_EXIT

    note right of INCOMPLETO
        Estado final:
        entryStatus: ON_TIME/LATE
        exitStatus: NO_EXIT
        attendanceStatus: INCOMPLETE
    end note
```

### Salida sin Entrada Correspondiente

```mermaid
stateDiagram-v2
    [*] --> EXIT_RECIBIDA: Se marca salida
    EXIT_RECIBIDA --> VERIFICAR: ¿Hay entrada previa?
    VERIFICAR --> COMPLETAR: No hay entrada
    COMPLETAR --> [*]: Sistema genera MISSING_ENTRY

    note right of COMPLETAR
        Estado final:
        entryStatus: NO_ENTRY
        exitStatus: ON_TIME/EARLY_EXIT
        attendanceStatus: INCOMPLETE
    end note
```

### Registros en Horario No Laborable

```mermaid
flowchart TD
    R[Registro recibido] --> W{¿Dentro de<br/>ventana válida?}
    W -->|No| SKIP[Skipped Record]
    W -->|Sí| PROC[Procesar normalmente]

    SKIP --> MSG[Message:<br/>'Skipped: Outside valid time windows']
    MSG --> MARK[Mark processed<br/>with error]

    style SKIP fill:#ffcdd2
    style PROC fill:#c8e6c9
```

---

## 4.3.7 Matriz de Clasificación Completa

| Escenario | Hay ENTRY | Hay EXIT | Estado Asistencia | Estado Entrada | Estado Salida |
|-----------|-----------|----------|-------------------|----------------|---------------|
| Día perfecto | ✅ | ✅ | COMPLETE | ON_TIME | ON_TIME |
| Llegada tarde | ✅ | ✅ | COMPLETE | LATE | ON_TIME |
| Salida temprana | ✅ | ✅ | COMPLETE | ON_TIME | EARLY_EXIT |
 | Tarde y sale temprano | ✅ | ✅ | COMPLETE | LATE | EARLY_EXIT |
| Horas extra | ✅ | ✅ | COMPLETE | ON_TIME | OVERTIME |
| Olvidó marcar entrada | ❌ | ✅ | INCOMPLETE | NO_ENTRY | ON_TIME |
| Olvidó marcar salida | ✅ | ❌ | INCOMPLETE | ON_TIME | NO_EXIT |
| No se presentó | ❌ | ❌ | ABSENCE | NO_ENTRY | NO_EXIT |
| Día festivo | ❌ | ❌ | HOLIDAY | - | - |
| Permiso aprobado | ❌ | ❌ | JUSTIFIED | - | - |

---

[Siguiente: Ventanas de Tiempo](./04-ventanas-de-tiempo.md) | [Anterior: Pipeline de Procesamiento](./02-pipeline-de-procesamiento.md)
