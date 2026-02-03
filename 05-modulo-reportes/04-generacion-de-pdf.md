# 5.4 Generación de PDF

El sistema implementó la generación de reportes en formato PDF utilizando **Puppeteer**, una librería que permitió la renderización de documentos PDF server-side con alta fidelidad.

---

## 5.4.1 Arquitectura de Generación de PDF

```mermaid
flowchart LR
    subgraph Frontend["Frontend"]
        B[ExportPDFButton]
    end

    subgraph API["Backend API"]
        C[ReportsController]
        S[PDFExportService]
    end

    subgraph Generation["Generación PDF"]
        T[Template Engine<br/>HTML + CSS]
        P[Puppeteer<br/>Headless Chrome]
    end

    subgraph Output["Salida"]
        PDF[PDF Buffer]
    end

    B -->|GET /export/pdf| C
    C --> S
    S --> T
    T --> P
    P --> PDF
    PDF -->|Binary Response| B
    B -->|Download| U[Usuario]

    style P fill:#c8e6c9
    style PDF fill:#e3f2fd
```

---

## 5.4.2 Proceso de Generación

```mermaid
sequenceDiagram
    actor U as Usuario
    participant F as Frontend
    participant API as Backend API
    participant S as PDFExportService
    participant DB as PostgreSQL
    participant P as Puppeteer

    U->>F: Clic en Exportar PDF
    F->>API: GET /attendance/reports/me/export/pdf
    API->>S: generatePDF(userUuid, date)

    S->>DB: Consulta datos del reporte
    DB-->>S: ReportData

    S->>S: Genera HTML template
    S->>P: lanza puppeteer
    P->>P: Carga HTML
    P->>P: Renderiza PDF
    P-->>S: PDF Buffer

    S-->>API: Buffer PDF
    API-->>F: Binary Response
    F->>F: Crea Blob URL
    F->>U: Descarga archivo
```

---

## 5.4.3 Estructura del PDF Generado

```mermaid
flowchart TD
    subgraph PDF["Estructura del Documento PDF"]
        H[Encabezado]
        D1[Datos del Empleado]
        D2[Período del Reporte]
        T1[Tabla de Asistencias]
        S[Resumen de Métricas]
        F[Pie de Página]
    end

    subgraph Detalles["Detalles"]
        H1[Logo de la institución]
        H2[Título del reporte]
        H3[Fecha de generación]
        D1a[Nombre completo]
        D1b[Documento]
        D1c[Departamento]
        T1a[Filas por día]
        T1b[Columnas de métricas]
        Sa[Total horas trabajadas]
        Sb[Total tardanzas]
        Sc[Total salidas tempranas]
        Sd[Total horas extras]
        F1[Firma digital]
        F2[Número de reporte]
    end

    H --> H1
    H --> H2
    H --> H3
    D1 --> D1a
    D1 --> D1b
    D1 --> D1c
    T1 --> T1a
    T1 --> T1b
    S --> Sa
    S --> Sb
    S --> Sc
    S --> Sd
    F --> F1
    F --> F2
```

### Elementos del PDF

| Sección | Contenido |
|---------|-----------|
| **Encabezado** | Logo, título "Reporte de Asistencia", fecha de generación |
| **Datos del Empleado** | Nombre completo, documento, departamento, cargo |
| **Período** | Rango de fechas del reporte |
| **Tabla de Asistencias** | Una fila por día con: fecha, estado, entrada, salida, trabajados, tardanza, extras |
| **Resumen** | Totales del período y porcentaje de asistencia |
| **Pie de Página** | Número de reporte, fecha de generación, disclaimer |

---

## 5.4.4 Template HTML

El sistema utilizó templates HTML con CSS inline para garantizar la consistencia visual:

```mermaid
flowchart LR
    subgraph Template["Template HTML"]
        HTML["<!DOCTYPE html>"]
        HEAD["<head>"]
        BODY["<body>"]
    end

    subgraph Estilos["Estilos CSS"]
        CSS1[Tipografías]
        CSS2[Colores institucionales]
        CSS3[Layout de tabla]
        CSS4[Spaciados y márgenes]
    end

    subgraph Datos["Datos Dinámicos"]
        DD1[Información del empleado]
        DD2[Filas de asistencia]
        DD3[Resumen de métricas]
    end

    HTML --> HEAD
    HTML --> BODY
    HEAD --> Estilos
    BODY --> Datos

    style Template fill:#e3f2fd
    style Estilos fill:#fff3e0
    style Datos fill:#c8e6c9
```

### Características del Template

| Característica | Implementación |
|----------------|----------------|
| **Responsivo** | CSS media queries para A4 |
| **Identidad visual** | Colores y logo de la institución |
| **Accesibilidad** | Estructura semántica HTML |
| **Impresión** | Optimizado para impresión en papel |
| **Metadata** | Título, autor, fecha en el PDF |

---

## 5.4.5 Configuración de Puppeteer

```mermaid
flowchart TD
    subgraph Opciones["Opciones de Puppeteer"]
        O1[format: A4]
        O2[landscape: false]
        O3[printBackground: true]
        O4[margin: top/bottom 1cm]
        O5[displayHeaderFooter: true]
    end

    subgraph Calidad["Calidad de PDF"]
        Q1[preferCSSPageSize: true]
        Q2[omitBackground: false]
    end

    subgraph Performance["Optimizaciones"]
        P1[headless: true]
        P2[timeout: 30000ms]
        P3[reusable instance]
    end

    Opciones --> Calidad
    Calidad --> Performance

    style Opciones fill:#e3f2fd
    style Calidad fill:#fff3e0
    style Performance fill:#c8e6c9
```

---

## 5.4.6 Flujo de Descarga en el Frontend

```mermaid
flowchart TD
    START([Usuario clic Exportar]) --> B1[Mostrar loading state]
    B1 --> B2{¿Response OK?}
    B2 -->|No| ERR[Mostrar error]
    B2 -->|Sí| B3[Crear Blob desde response]
    B3 --> B4[Crear URL object]
    B4 --> B5[Crear link <a> invisible]
    B5 --> B6[Trigger click]
    B6 --> B7[Revocar URL object]
    B7 --> B8[Mostrar success toast]
    B8 --> END([Usuario recibe PDF])

    style START fill:#e3f2fd
    style END fill:#c8e6c9
    style ERR fill:#ffcdd2
```

### Componente ExportPDFButton

```mermaid
classDiagram
    class ExportPDFButton {
        +onClick: function
        +loading: boolean
        +disabled: boolean
        +handleExport()
        +downloadPDF()
    }

    class State {
        +isPending: boolean
        +error: Error | null
        +data: Blob | null
    }

    class UI {
        +render(): JSX.Element
        +showLoading(): Spinner
        +showError(): Toast
        +showSuccess(): Toast
    }

    ExportPDFButton --> State
    ExportPDFButton --> UI
```

---

## 5.4.7 Manejo de Errores

```mermaid
flowchart TD
    START([Generar PDF]) --> T1{¿Timeout?}
    T1 -->|Sí| E1[Error: Timeout de Puppeteer]
    T1 -->|No| T2{¿Datos inválidos?}
    T2 -->|Sí| E2[Error: Datos vacíos]
    T2 -->|No| T3{¿Error de template?}
    T3 -->|Sí| E3[Error: HTML inválido]
    T3 -->|No| SUCCESS[PDF Generado]

    E1 --> TOAST[Toast: Error al generar PDF]
    E2 --> TOAST
    E3 --> TOAST
    SUCCESS --> DOWNLOAD[Descarga automática]

    style SUCCESS fill:#c8e6c9
    style E1 fill:#ffcdd2
    style E2 fill:#ffcdd2
    style E3 fill:#ffcdd2
```

---

## 5.4.8 Performance y Optimizaciones

### Métricas de Generación

| Métrica | Valor |
|---------|-------|
| **Tiempo promedio** | 5-15 segundos |
| **Tamaño del PDF** | 50-500 KB (depende de filas) |
| **Concurrencia** | Hasta 5 PDFs simultáneos |
| **Memory usage** | ~100MB por instancia |

### Optimizaciones Implementadas

```mermaid
flowchart TD
    subgraph Optimizaciones["Optimizaciones"]
        O1[Instancia reutilizable<br/>de Puppeteer]
        O2[Cache de templates<br/>compilados]
        O3[Streaming response<br/>para archivos grandes]
        O4[Generación asíncrona<br/>con cola de jobs]
    end

    subgraph Beneficios["Beneficios"]
        B1[Reduce cold start]
        B2[Menor uso de memoria]
        B3[Mejor UX]
        B4[Escalar]
    end

    O1 --> B1
    O2 --> B2
    O3 --> B3
    O4 --> B4
```

---

[Anterior: Casos de Uso](./03-casos-de-uso.md) | [Siguiente: Integración de Dispositivos](/06-integracion-dispositivos/01-integracion-zkteco.md)
