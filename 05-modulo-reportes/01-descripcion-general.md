# 5.1 Descripción General del Módulo de Reportes

Este módulo constituyó el tercer objetivo específico del proyecto: **desarrollar un módulo que generó reportes de asistencia actualizados y confiables**, facilitando la aplicación de descuentos por atrasos y respaldando decisiones oportunas en la administración del recurso humano.

---

## 5.1.1 Propósito del Módulo

El módulo de reportes tuvo como finalidad:

1. **Generar reportes consolidados** de asistencia con múltiples criterios de filtrado.
2. **Calcular métricas precisas** de tardanzas, salidas tempranas y horas extras.
3. **Facilitar la exportación** a PDF para integración con sistemas de nómina.
4. **Proporcionar una única fuente de verdad** para todos los cálculos de asistencia.

---

## 5.1.2 Tipos de Reportes Implementados

```mermaid
flowchart TD
    subgraph Tipos["Tipos de Reportes"]
        R1[Reporte Diario<br/>Asistencia de un día específico]
        R2[Reporte Mensual<br/>Consolidado de un mes]
        R3[Reporte por Período<br/>Rango de fechas personalizado]
        R4[Reporte Departamental<br/>Por departamento o área]
        R5[Reporte Individual<br/>Por usuario específico]
    end

    subgraph Formatos["Formatos de Salida"]
        F1[Vista Web<br/>Tabla interactiva]
        F2[PDF<br/>Documento para nómina]
        F3[CSV<br/>Para análisis externo]
    end

    R1 --> F1
    R1 --> F2
    R2 --> F1
    R2 --> F2
    R2 --> F3
    R3 --> F1
    R3 --> F2
    R4 --> F1
    R4 --> F2
    R5 --> F1
    R5 --> F2

    style R1 fill:#e3f2fd
    style R2 fill:#c8e6c9
    style R3 fill:#fff3e0
    style R4 fill:#f3e5f5
    style R5 fill:#ffe0b2
```

### Descripción de Tipos de Reporte

| Tipo | Descripción | Caso de Uso |
|------|-------------|-------------|
| **Diario** | Asistencia de un día específico | Verificación diaria de asistencia |
| **Mensual** | Consolidado de todas las sesiones de un mes | Resumen para nómina mensual |
| **Por Período** | Rango de fechas personalizado | Reportes de quincena o semana |
| **Departamental** | Agregado por departamento | Análisis por área |
| **Individual** | Historial de un usuario específico | Consulta personal o auditoría |

---

## 5.1.3 Componentes del Módulo

### Arquitectura de Servicios de Reportes

```mermaid
classDiagram
    class AttendanceReportService {
        +getTodayReport(userUuid, date)
        +getMonthlyReport(userUuid, month, year)
        +getReportByDateRange(userUuid, startDate, endDate)
    }

    class AttendanceSummaryReportService {
        +getDepartmentSummary(deptUuid, dateRange)
        +getAllUsersSummary(dateRange)
        +getAbsentUsersReport(date)
    }

    class PDFExportService {
        +generateAttendancePDF(reportData)
        +generateDepartmentPDF(reportData)
    }

    class AttendanceCalculations {
        +calculateWorkedMinutes(events, period)
        +calculateLateMinutes(events, period)
        +calculateEarlyExitMinutes(events, period)
        +calculateOvertimeMinutes(events, period)
    }

    AttendanceReportService --> AttendanceCalculations
    AttendanceSummaryReportService --> AttendanceCalculations
    PDFExportService --> AttendanceReportService
    PDFExportService --> AttendanceSummaryReportService

    style AttendanceCalculations fill:#fff3e0
```

### Single Source of Truth

Todos los cálculos de asistencia se centralizaron en `/src/utils/reports/attendance-calculations.ts`:

```mermaid
flowchart TD
    subgraph Consumidores["Consumidores de Cálculos"]
        C1[AttendanceReportService]
        C2[AttendanceSummaryReportService]
        C3[PayrollService]
    end

    subgraph Source["Single Source of Truth"]
        CALC[attendance-calculations.ts<br/>Funciones puras de cálculo]
    end

    C1 --> CALC
    C2 --> CALC
    C3 --> CALC

    style CALC fill:#c8e6c9
```

---

## 5.1.4 Métricas Calculadas

El módulo calculó las siguientes métricas para cada sesión de asistencia:

| Métrica | Descripción | Fórmula Base |
|---------|-------------|--------------|
| **Minutos Trabajados** | Total de tiempo entre entradas y salidas | Σ(SALIDA - ENTRADA) |
| **Minutos de Tardanza** | Tiempo después de la tolerancia de entrada | max(0, entrada - (inicio + tolerancia)) |
| **Minutos de Salida Temprana** | Tiempo antes de la tolerancia de salida | max(0, (fin - tolerancia) - salida) |
| **Minutos de Horas Extras** | Tiempo después de la tolerancia de salida | max(0, salida - (fin + tolerancia)) |
| **Porcentaje de Asistencia** | Ratio de días completos vs. laborables | (completos / laborables) × 100 |

---

## 5.1.5 Endpoints de la API

### Reportes Individuales

| Endpoint | Método | Descripción |
|----------|--------|-------------|
| `/attendance/reports/me` | GET | Reporte del usuario autenticado |
| `/attendance/reports/me/events` | GET | Eventos del usuario autenticado |
| `/attendance/reports/me/export/pdf` | GET | Exportar a PDF |
| `/device-raw-records/my-records` | GET | Registros crudos del usuario |

### Reportes Administrativos

| Endpoint | Método | Descripción |
|----------|--------|-------------|
| `/attendance/reports/admin` | GET | Reporte administrativo filtrable |
| `/attendance/reports/monthly` | GET | Reporte mensual consolidado |
| `/attendance/reports/department` | GET | Reporte por departamento |
| `/attendance/reports/absences` | GET | Reporte de ausencias |
| `/device-raw-records` | GET | Todos los registros (admin) |

---

## 5.1.6 Flujo de Generación de Reportes

```mermaid
sequenceDiagram
    actor U as Usuario
    participant F as Frontend
    participant API as Backend API
    participant R as ReportService
    participant C as Calculations
    participant DB as PostgreSQL
    participant P as Puppeteer

    U->>F: Solicita reporte (filtros)
    F->>API: GET /attendance/reports/...
    API->>R: Invoca servicio
    R->>DB: Consulta sesiones y eventos
    DB-->>R: Datos crudos
    R->>C: Calcula métricas
    C-->>R: Métricas calculadas
    R-->>API: ReportDataDTO
    API-->>F: JSON Response
    F-->>U: Muestra tabla

    Note over U,P: Si solicita PDF
    U->>F: Clic en Exportar PDF
    F->>API: GET /export/pdf
    API->>R: Genera datos
    R->>P: Genera PDF
    P-->>API: Buffer PDF
    API-->>F: Blob PDF
    F-->>U: Descarga archivo
```

---

[Siguiente: Cálculos de Asistencia](./02-calculos-asistencia.md) | [Anterior: Ventanas de Tiempo](/documentacion/04-modulo-procesamiento-biometrico/04-ventanas-de-tiempo.md)
