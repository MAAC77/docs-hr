# 6.2 Sincronización de Datos Biométricos

La sincronización de datos fue el proceso mediante el cual el sistema recuperó los registros biométricos de los dispositivos y los almacenó en la base de datos centralizada.

---

## 6.2.1 Proceso de Sincronización

```mermaid
flowchart TD
    START([Inicio de sincronización]) --> S1[Obtener dispositivos activos]
    S1 --> S2{¿Hay dispositivos?}
    S2 -->|No| END1([Finalizar])
    S2 -->|Sí| S3[Para cada dispositivo]

    S3 --> C1[Conectar a dispositivo]
    C1 --> C2{¿Conexión exitosa?}
    C2 -->|No| ERR[Marcar error y continuar]
    C2 -->|Sí| R1[Obtener registros nuevos]

    R1 --> R2[Transformar a DeviceRawRecord]
    R2 --> R3[Generar unique_signature]
    R3 --> R4[Verificar duplicados]
    R4 --> R5{¿Existe registro?}
    R5 -->|Sí| SKIP[Omitir duplicado]
    R5 -->|No| R6[Guardar en BD]
    R6 --> U1[Actualizar lastSyncAt]

    SKIP --> S4
    ERR --> S4
    U1 --> S4{¿Más dispositivos?}
    S4 -->|Sí| S3
    S4 -->|No| END2([Sincronización completa])

    style START fill:#e3f2fd
    style END2 fill:#c8e6c9
    style ERR fill:#ffcdd2
    style SKIP fill:#fff3e0
```

---

## 6.2.2 Job Programado de Sincronización

El sistema implementó un job programado que ejecutó la sincronización automáticamente:

```mermaid
gantt
    title Ejecución del Job de Sincronización
    dateFormat HH:mm
    axisFormat %H:%M

    section Job Programado
    Inicio del job       :milestone, s1, 00:00, 0m
    Primera ejecución    :active, exec1, 00:00, 2m
    Espera               :wait1, 02:00, 3m
    Segunda ejecución    :exec2, 05:00, 2m
    Espera               :wait2, 07:00, 3m
    Tercera ejecución    :exec3, 10:00, 2m
```

### Configuración del Job

| Parámetro | Valor |
|-----------|-------|
| **Cron expression** | `*/5 * * * *` (cada 5 minutos) |
| **Timezone** | Tiempo local del servidor |
| **Concurrent** | false (no ejecutar si ya hay uno corriendo) |
| **Max retries** | 3 |
| **Retry delay** | 1 minuto |

---

## 6.2.3 Flujo Detallado de Sincronización

```mermaid
sequenceDiagram
    participant Job as Cron Job
    participant Sync as DeviceSyncService
    participant ZK as node-zklib
    participant DB as PostgreSQL
    participant Engine as AttendanceEngine

    Note over Job,Engine: Se ejecuta cada 5 minutos
    Job->>Sync: trigger()
    Sync->>DB: SELECT devices WHERE isActive=true
    DB-->>Sync: Lista de dispositivos

    loop Para cada dispositivo
        Sync->>ZK: connect(ip, port)
        ZK-->>Sync: connected
        Sync->>ZK: getAttendance(lastSyncAt)
        ZK-->>Sync: Registros de asistencia

        Sync->>DB: SELECT WHERE unique_signature IN (...)
        DB-->>Sync: Registros existentes

        Sync->>Sync: Filtrar duplicados

        par Insertar nuevos registros
            Sync->>DB: INSERT device_raw_records
        end

        Sync->>ZK: disconnect()
    end

    Sync->>DB: UPDATE lastSyncAt
    Sync-->>Job: Completado

    Note over Job,Engine: Motor procesa nuevos registros automáticamente
    Engine->>DB: SELECT WHERE processed=false
    Engine->>Engine: processRecords()
```

---

## 6.2.4 Prevención de Duplicados

El sistema implementó múltiples mecanismos para prevenir el procesamiento duplicado de registros:

```mermaid
flowchart TD
    subgraph Mecanismos["Mecanismos Anti-Duplicación"]
        M1[unique_signature<br/>Hash compuesto]
        M2[Unique Constraint<br/>En base de datos]
        M3[processed flag<br/>Marca de procesado]
    end

    subgraph Proceso["Proceso de Validación"]
        V1[Generar hash]
        V2[Consultar existente]
        V3{¿Existe?}
        V4[Omitir registro]
        V5[Insertar nuevo]
    end

    M1 --> V1
    M2 --> V2
    M3 --> V2
    V2 --> V3
    V3 -->|Sí| V4
    V3 -->|No| V5

    style Mecanismos fill:#e3f2fd
    style Proceso fill:#fff3e0
```

### Unique Signature

El `unique_signature` se generó como un hash de:

```mermaid
flowchart LR
    A[device_uuid] --> C
    B[user_sn] --> C
    D[record_time] --> C

    C[SHA-256 Hash] --> E[unique_signature]

    style A fill:#e3f2fd
    style B fill:#fff3e0
    style D fill:#c8e6c9
    style E fill:#f3e5f5
```

---

## 6.2.5 Transformación de Datos

### Formato de Registro ZKTeco → DeviceRawRecord

```mermaid
flowchart LR
    subgraph ZKFormat["Formato ZKTeco"]
        Z1[userid: 16 bytes]
        Z2[timestamp: 4 bytes uint32]
        Z3[verify_mode: 1 byte]
        Z4[reserved: 8 bytes]
    end

    subgraph Transformation["Transformación"]
        T1[userid → user_sn: bigint]
        T2[timestamp → record_time: timestamptz]
        T3[verify_mode → inference]
    end

    subgraph DBFormat["DeviceRawRecord"]
        DB1[uuid: UUID]
        DB2[device_uuid: UUID]
        DB3[user_sn: bigint]
        DB4[record_time: timestamptz]
        DB5[processed: boolean]
        DB6[unique_signature: string]
    end

    ZKFormat --> Transformation
    Transformation --> DBFormat

    style ZKFormat fill:#e3f2fd
    style Transformation fill:#fff3e0
    style DBFormat fill:#c8e6c9
```

---

## 6.2.6 Estados de Sincronización

### Estados de un Dispositivo

```mermaid
stateDiagram-v2
    [*] --> OFFLINE: Dispositivo creado
    OFFLINE --> CONNECTING: Inicia sincronización
    CONNECTING --> ONLINE: Conexión exitosa
    CONNECTING --> ERROR: Falla conexión
    ONLINE --> SYNCING: Obteniendo registros
    SYNCING --> ONLINE: Sincronización completa
    SYNCING --> ERROR: Error durante sync
    ERROR --> CONNECTING: Reintento automático
    ONLINE --> OFFLINE: Desconexión normal

    note right of ONLINE
        Listo para sincronizar
        lastSyncAt actualizado
    end note

    note right of ERROR
        Reintentar en 5 min
        Notificar administrador
        si persiste
    end note
```

### Estados de un Registro

```mermaid
stateDiagram-v2
    [*] --> CREATED: Insertado en BD
    CREATED --> PROCESSED: Motor procesa
    CREATED --> SKIPPED: Fuera de ventana
    CREATED --> HOLIDAY: Día festivo
    CREATED --> JUSTIFIED: Permiso aprobado
    PROCESSED --> [*]
    SKIPPED --> [*]
    HOLIDAY --> [*]
    JUSTIFIED --> [*]
```

---

## 6.2.7 Monitoreo y Alertas

### Métricas de Sincronización

| Métrica | Descripción |
|---------|-------------|
| **Registros por sincronización** | Cantidad de registros recuperados |
| **Tiempo de sincronización** | Duración del proceso por dispositivo |
| **Última sincronización exitosa** | Timestamp de última sync correcta |
| **Estado del dispositivo** | ONLINE, OFFLINE, ERROR |
| **Registros duplicados** | Cantidad omitidos por duplicidad |

### Alertas Implementadas

```mermaid
flowchart TD
    subgraph Condiciones["Condiciones de Alerta"]
        C1[Dispositivo OFFLINE > 1 hora]
        C2[Error de conexión > 3 intentos]
        C3[No nuevos registros > 24 horas]
        C4[Registros duplicados > 50%]
    end

    subgraph Acciones["Acciones"]
        A1[Notificar a administradores]
        A2[Enviar email]
        A3[Marcar dispositivo con error]
        A4[Crear ticket de soporte]
    end

    C1 --> A1
    C2 --> A1
    C3 --> A2
    C4 --> A3
    C4 --> A4

    style Condiciones fill:#e3f2fd
    style Acciones fill:#fff3e0
```

---

## 6.2.8 Sincronización Manual

Además del job automático, el sistema permitió la sincronización manual:

```mermaid
sequenceDiagram
    actor A as Administrador
    participant UI as Frontend
    participant API as Backend
    participant Sync as DeviceSyncService
    participant ZK as Dispositivo

    A->>UI: Selecciona dispositivo
    A->>UI: Clic en "Sincronizar ahora"
    UI->>API: POST /devices/:id/sync
    API->>Sync: syncDevice(deviceUuid)
    Sync->>ZK: connect + getAttendance
    ZK-->>Sync: Registros
    Sync->>Sync: Procesar registros
    Sync-->>API: Resultado (registros insertados)
    API-->>UI: Response con conteo
    UI-->>A: Toast: "Sincronizados X registros"
```

---

[Anterior: Integración ZKTeco](./01-integracion-zkteco.md) | [Siguiente: Seguridad y Autenticación](/documentacion/07-seguridad-y-autenticacion.md)
