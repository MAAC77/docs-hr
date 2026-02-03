# 5.2 Cálculos de Asistencia

Todos los cálculos de asistencia se centralizaron en funciones puras ubicadas en `/src/utils/reports/attendance-calculations.ts`, garantizando consistencia y facilitando las pruebas unitarias.

---

## 5.2.1 Single Source of Truth

```mermaid
flowchart TD
    subgraph Modules["Módulos que Consumen Cálculos"]
        M1[AttendanceReportService]
        M2[AttendanceSummaryReportService]
        M3[PayrollService]
        M4[StatisticsService]
    end

    subgraph Calculations["/src/utils/reports/"]
        C1[attendance-calculations.ts<br/>Métricas de asistencia]
        C2[time-calculations.ts<br/>Utilidades de tiempo]
        C3[schedule-calculations.ts<br/>Cálculos de horarios]
        C4[report-aggregations.ts<br/>Agregaciones]
    end

    M1 --> C1
    M2 --> C1
    M2 --> C4
    M3 --> C1
    M4 --> C4

    C1 --> C2
    C1 --> C3

    style C1 fill:#c8e6c9
    style C2 fill:#fff3e0
    style C3 fill:#e3f2fd
    style C4 fill:#f3e5f5
```

---

## 5.2.2 Cálculo de Minutos Trabajados

### Algoritmo

```mermaid
flowchart TD
    START([Eventos del día]) --> S1[Separar ENTRY y EXIT]
    S1 --> S2[Ordenar cronológicamente]
    S2 --> S3[Emparejar ENTRADA i con SALIDA i]
    S3 --> S4{¿Hay par válido?}
    S4 -->|Sí| S5[Calcular: SALIDA - ENTRADA]
    S4 -->|No| S6[Ignorar evento huérfano]
    S5 --> S7[Sumar al total]
    S6 --> S8{¿Más pares?}
    S7 --> S8
    S8 -->|Sí| S3
    S8 -->|No| END([Total minutos trabajados])

    style START fill:#e3f2fd
    style END fill:#c8e6c9
```

### Ejemplo Práctico

```
Eventos del día:
  ENTRADA: 08:00
  SALIDA:  12:00
  ENTRADA: 14:00
  SALIDA:  18:00

Cálculo:
  Par 1: 12:00 - 08:00 = 240 minutos (4 horas)
  Par 2: 18:00 - 14:00 = 240 minutos (4 horas)
  Total: 480 minutos (8 horas)
```

### Función: calculateWorkedMinutes()

```
Entrada:
  events: Array de AttendanceEvent
  period: SchedulePeriod opcional

Salida:
  minutos: Número entero de minutos trabajados

Lógica:
  1. Filtrar eventos por tipo (ENTRY vs EXIT)
  2. Ordenar cada lista cronológicamente
  3. Emparejar ENTRY[i] con EXIT[i]
  4. Sumar diferencia de tiempo de cada par válido
```

---

## 5.2.3 Cálculo de Minutos de Tardanza

### Algoritmo

```mermaid
flowchart TD
    START([Primer evento ENTRY]) --> C1{¿Hay período?}
    C1 -->|No| ZERO1[Retornar 0]
    C1 -->|Sí| C2{¿Estado es LATE?}
    C2 -->|No| ZERO2[Retornar 0]
    C2 -->|Sí| C3[Obtener primera entrada]
    C3 --> C4[Calcular tolerancia: inicio + tol]
    C4 --> C5{¿Entrada > tolerancia?}
    C5 -->|No| ZERO3[Retornar 0]
    C5 -->|Sí| C6[Calcular: entrada - tolerancia]
    C6 --> END([Minutos de tardanza])

    style ZERO1 fill:#fff3e0
    style ZERO2 fill:#fff3e0
    style ZERO3 fill:#fff3e0
    style END fill:#ffcdd2
```

### Ejemplo Práctico

```
Configuración:
  Hora oficial: 08:00
  Tolerancia: 15 minutos
  Ventana ON_TIME: 07:45 - 08:15

Caso 1: Llegada a las 08:45
  Tolerancia fin: 08:00 + 15 = 08:15
  Tardanza: 08:45 - 08:15 = 30 minutos

Caso 2: Llegada a las 07:50 (EARLY)
  No se considera tardanza
  Tardanza: 0 minutos

Caso 3: Llegada a las 08:10 (ON_TIME)
  Dentro de tolerancia
  Tardanza: 0 minutos
```

### Función: calculateLateMinutes()

```
Entrada:
  events: Array de AttendanceEvent
  period: SchedulePeriod
  entryStatus: EntryStatus
  workDate: Date

Salida:
  minutos: Número entero de minutos de tardanza

Lógica:
  1. Si no hay período o entryStatus ≠ LATE, retornar 0
  2. Obtener el primer evento ENTRY
  3. Calcular hora fin de tolerancia (startTime + toleranceMinutes)
  4. Si entrada > finTolerancia, retornar diferencia
  5. Si no, retornar 0
```

---

## 5.2.4 Cálculo de Minutos de Salida Temprana

### Algoritmo

```mermaid
flowchart TD
    START([Último evento EXIT]) --> C1{¿Hay período?}
    C1 -->|No| ZERO1[Retornar 0]
    C1 -->|Sí| C2{¿Estado es EARLY_EXIT?}
    C2 -->|No| ZERO2[Retornar 0]
    C2 -->|Sí| C3[Obtener última salida]
    C3 --> C4[Calcular tolerancia: fin - tol]
    C4 --> C5{¿Salida < tolerancia?}
    C5 -->|No| ZERO3[Retornar 0]
    C5 -->|Sí| C6[Calcular: tolerancia - salida]
    C6 --> END([Minutos de salida temprana])

    style ZERO1 fill:#fff3e0
    style ZERO2 fill:#fff3e0
    style ZERO3 fill:#fff3e0
    style END fill:#ffe0b2
```

### Ejemplo Práctico

```
Configuración:
  Hora oficial: 17:00
  Tolerancia: 15 minutos
  Ventana ON_TIME: 16:45 - 17:15

Caso 1: Salida a las 16:20
  Tolerancia inicio: 17:00 - 15 = 16:45
  Salida temprana: 16:45 - 16:20 = 25 minutos

Caso 2: Salida a las 17:05 (ON_TIME)
  Dentro de tolerancia
  Salida temprana: 0 minutos

Caso 3: Salida a las 18:00 (OVERTIME)
  No es salida temprana
  Salida temprana: 0 minutos
```

---

## 5.2.5 Cálculo de Minutos de Horas Extras

### Algoritmo

```mermaid
flowchart TD
    START([Último evento EXIT]) --> C1{¿Hay período?}
    C1 -->|No| ZERO1[Retornar 0]
    C1 -->|Sí| C2{¿Estado es OVERTIME?}
    C2 -->|No| ZERO2[Retornar 0]
    C2 -->|Sí| C3[Obtener última salida]
    C3 --> C4[Calcular tolerancia: fin + tol]
    C4 --> C5{¿Salida > tolerancia?}
    C5 -->|No| ZERO3[Retornar 0]
    C5 -->|Sí| C6[Calcular: salida - tolerancia]
    C6 --> END([Minutos de horas extras])

    style ZERO1 fill:#fff3e0
    style ZERO2 fill:#fff3e0
    style ZERO3 fill:#fff3e0
    style END fill:#e3f2fd
```

### Ejemplo Práctico

```
Configuración:
  Hora oficial: 17:00
  Tolerancia: 15 minutos
  Ventana ON_TIME: 16:45 - 17:15

Caso 1: Salida a las 18:30
  Tolerancia fin: 17:00 + 15 = 17:15
  Horas extras: 18:30 - 17:15 = 75 minutos
```

---

## 5.2.6 Matriz de Cálculos

| Estado Entrada | Estado Salida | Trabajados | Tardanza | Salida Temprana | Horas Extras |
|----------------|---------------|------------|----------|-----------------|--------------|
| ON_TIME | ON_TIME | ✅ Calculado | 0 | 0 | 0 |
| LATE | ON_TIME | ✅ Calculado | ✅ Calculado | 0 | 0 |
| ON_TIME | EARLY_EXIT | ✅ Calculado | 0 | ✅ Calculado | 0 |
| LATE | EARLY_EXIT | ✅ Calculado | ✅ Calculado | ✅ Calculado | 0 |
| ON_TIME | OVERTIME | ✅ Calculado | 0 | 0 | ✅ Calculado |
| LATE | OVERTIME | ✅ Calculado | ✅ Calculado | 0 | ✅ Calculado |
| NO_ENTRY | NO_EXIT | 0 | 0 | 0 | 0 |
| NO_ENTRY | ON_TIME | ✅ Calculado | 0 | 0 | 0 |
| ON_TIME | NO_EXIT | ✅ Parcial | 0 | 0 | 0 |

---

## 5.2.7 Agregaciones para Reportes

### Nivel Sesión

```mermaid
flowchart LR
    subgraph Sesion["AttendanceSession"]
        S1[workDate]
        S2[entryStatus]
        S3[exitStatus]
        S4[workedMinutes]
        S5[lateMinutes]
        S6[earlyExitMinutes]
        S7[overtimeMinutes]
    end

    subgraph Calculado["Calculado por Motor"]
        C1[engineService]
    end

    C1 --> S4
    C1 --> S5
    C1 --> S6
    C1 --> S7

    style Calculado fill:#c8e6c9
```

### Nivel Diario

```
Resumen del día:
  - Total minutos trabajados = Σ(sessions.workedMinutes)
  - Total minutos tardanza = Σ(sessions.lateMinutes)
  - Total minutos salida temprana = Σ(sessions.earlyExitMinutes)
  - Total minutos horas extras = Σ(sessions.overtimeMinutes)
  - Estado del día = Determinado por lógica de negocio
```

### Nivel Mensual

```
Resumen del mes:
  - Total días trabajados = Count(sessions WHERE attendanceStatus = COMPLETE)
  - Total días ausencia = Count(sessions WHERE attendanceStatus = ABSENCE)
  - Total días justificados = Count(sessions WHERE attendanceStatus = JUSTIFIED)
  - Total días festivos = Count(sessions WHERE attendanceStatus = HOLIDAY)
  - Porcentaje asistencia = (trabajados / laborables) × 100
```

---

[Siguiente: Casos de Uso](./03-casos-de-uso.md) | [Anterior: Descripción General](./01-descripcion-general.md)
