# 6.1 Integración con Dispositivos Biométricos ZKTeco

El sistema se integró con dispositivos biométricos de la marca **ZKTeco** para la captura automática de marcaciones de asistencia mediante huella digital.

---

## 6.1.1 Arquitectura de Integración

```mermaid
flowchart LR
    subgraph Devices["Dispositivos ZKTeco"]
        ZK1[ZKTeco K40]
        ZK2[ZKTeco iFace]
        ZK3[ZKTeco TF1600]
    end

    subgraph Network["Red TCP/IP"]
        TCP[Puerto 4370<br/>Protocolo ZK]
    end

    subgraph Backend["Backend Service"]
        LIB[node-zklib<br/>Librería cliente]
        SYNC[DeviceSyncService]
        API[DeviceController]
    end

    subgraph Storage["Almacenamiento"]
        RAW[(device_raw_records)]
        DEV[(devices)]
    end

    ZK1 --> TCP
    ZK2 --> TCP
    ZK3 --> TCP
    TCP --> LIB
    LIB --> SYNC
    SYNC --> RAW
    SYNC --> DEV
    API --> SYNC

    style ZK1 fill:#e3f2fd
    style ZK2 fill:#e3f2fd
    style ZK3 fill:#e3f2fd
    style LIB fill:#c8e6c9
```

---

## 6.1.2 Características de los Dispositivos

| Modelo | Tipo | Capacidad | Usuarios | Registros |
|--------|------|-----------|----------|-----------|
| **K40** | Huella + RFID | 3,000 | 10,000 | 100,000 |
| **iFace** | Facial | 3,000 | 10,000 | 100,000 |
| **TF1600** | Huella TCP/IP | 3,000 | 10,000 | 100,000 |

### Protocolo de Comunicación

```mermaid
flowchart TD
    subgraph ZK_Protocol["Protocolo ZKTeco"]
        C1[Conexión TCP/IP]
        C2[Autenticación]
        C3[Comandos]
        C4[Desconexión]
    end

    subgraph Comandos_Principales["Comandos Principales"]
        CMD1[Connect<br/>Conectar a dispositivo]
        CMD2[GetUsers<br/>Obtener usuarios]
        CMD3[GetAttendance<br/>Obtener registros]
        CMD4[Disconnect<br/>Desconectar]
    end

    C1 --> CMD1
    C2 --> CMD1
    C3 --> CMD2
    C3 --> CMD3
    C4 --> CMD4

    style ZK_Protocol fill:#e3f2fd
    style Comandos_Principales fill:#fff3e0
```

---

## 6.1.3 Librería node-zklib

El sistema utilizó la librería `node-zklib` como cliente para comunicarse con los dispositivos:

```mermaid
classDiagram
    class ZKLib {
        +ip: string
        +port: number
        +timeout: number
        +connect()
        +disconnect()
        +getUsers()
        +getAttendance()
        +getDeviceStatus()
    }

    class DeviceSyncService {
        +syncDevice(deviceUuid)
        +syncUsers(device)
        +syncRecords(device)
        +handleConnectionError()
    }

    class DeviceController {
        +getDevices()
        +createDevice()
        +updateDevice()
        +deleteDevice()
        +testConnection()
        +syncNow()
    }

    DeviceSyncService --> ZKLib : uses
    DeviceController --> DeviceSyncService : calls

    style ZKLib fill:#c8e6c9
    style DeviceSyncService fill:#fff3e0
    style DeviceController fill:#e3f2fd
```

---

## 6.1.4 Flujo de Conexión

```mermaid
sequenceDiagram
    participant API as Backend API
    participant SYNC as DeviceSyncService
    participant ZK as node-zklib
    participant DEV as Dispositivo ZKTeco

    API->>SYNC: syncDevice(deviceUuid)
    SYNC->>SYNC: obtener configuración del dispositivo
    SYNC->>ZK: createZKLib(ip, port, timeout)
    ZK->>DEV: TCP Connect (port 4370)
    DEV-->>ZK: Connection Established

    ZK->>DEV: Auth (comando de autenticación)
    DEV-->>ZK: Auth OK

    ZK-->>SYNC: emit('connect')

    SYNC->>ZK: getAttendance()
    ZK->>DEV: Solicitar registros
    DEV-->>ZK: Buffer de registros
    ZK-->>SYNC: emit('attendance', records)

    SYNC->>SYNC: procesar y guardar registros
    SYNC->>ZK: disconnect()
    ZK->>DEV: TCP Disconnect
    ZK-->>SYNC: emit('disconnect')

    SYNC-->>API: Sync completado
```

---

## 6.1.5 Estructura de Registro ZKTeco

Los dispositivos ZKTeco generaron registros con el siguiente formato:

```
Crudo (del dispositivo):
  - User SN:    Serial number del usuario (bigint)
  - Device ID:  Identificador del dispositivo (string)
  - Record Time: Timestamp de la marcación (uint32)
  - Verify Mode: Tipo de verificación (huella, password, cardface)
```

```mermaid
flowchart LR
    subgraph Dispositivo["Dispositivo ZKTeco"]
        D1[User SN: 123]
        D2[Record Time: 1704067200]
    end

    subgraph Transformacion["Transformación"]
        T1[Mapeo User SN → user_uuid]
        T2[Unix timestamp → timestamptz]
        T3[Generar unique_signature]
    end

    subgraph BaseDatos["DeviceRawRecord"]
        BD1[user_sn: 123]
        BD2[record_time: 2024-01-01 08:00:00]
        BD3[processed: false]
        BD4[unique_signature: hash]
    end

    D1 --> T1
    D2 --> T2
    T1 --> BD1
    T2 --> BD2
    T3 --> BD4

    style Dispositivo fill:#e3f2fd
    style Transformacion fill:#fff3e0
    style BaseDatos fill:#c8e6c9
```

---

## 6.1.6 Gestión de Dispositivos

### Entidad Device

```mermaid
classDiagram
    class Device {
        +uuid: UUID
        +name: string
        +ipAddress: string
        +port: number
        +deviceType: string
        +status: DeviceStatus
        +lastSyncAt: DateTime
        +isActive: boolean
    }

    class DeviceStatus {
        <<enumeration>>
        ONLINE
        OFFLINE
        ERROR
    }

    Device --> DeviceStatus : uses
```

### Operaciones CRUD

| Operación | Endpoint | Descripción |
|-----------|----------|-------------|
| **Crear** | `POST /devices` | Registrar nuevo dispositivo |
| **Listar** | `GET /devices` | Obtener todos los dispositivos |
| **Actualizar** | `PATCH /devices/:id` | Modificar configuración |
| **Eliminar** | `DELETE /devices/:id` | Eliminar dispositivo |
| **Probar conexión** | `POST /devices/:id/test` | Verificar conectividad |
| **Sincronizar ahora** | `POST /devices/:id/sync` | Forzar sincronización |

---

## 6.1.7 Mapeo de Usuarios

El sistema requirió mapear los usuarios del dispositivo con los usuarios del sistema:

```mermaid
flowchart TD
    subgraph Sistema["Sistema HR"]
        SU[user_uuid: UUID<br/>document_number: string]
    end

    subgraph Dispositivo["Dispositivo ZKTeco"]
        DU[device_user_sn: bigint<br/>device_user_id: string]
    end

    subgraph Mapping["Mapeo"]
        M1[User.device_user_sn<br/>Almacena el SN del dispositivo]
        M2[Sync Service<br/>Relaciona por documento o SN]
    end

    SU --> M1
    DU --> M2
    M1 <--> M2

    style Sistema fill:#e3f2fd
    style Dispositivo fill:#fff3e0
    style Mapping fill:#c8e6c9
```

### Estrategias de Mapeo

| Estrategia | Descripción | Ventajas |
|------------|-------------|----------|
| **Por documento** | El `device_user_id` contiene el documento | Mapeo directo |
| **Por SN asignado** | Se asigna manualmente el `device_user_sn` | Mayor control |
| **Por importación** | Se importa desde el dispositivo | Automático |

---

## 6.1.8 Manejo de Errores de Conexión

```mermaid
flowchart TD
    START([Intentar conectar]) --> T1{¿Timeout?}
    T1 -->|Sí| E1[Error: Connection timeout]
    T1 -->|No| T2{¿Refused?}
    T2 -->|Sí| E2[Error: Connection refused]
    T2 -->|No| T3{¿Auth error?}
    T3 -->|Sí| E3[Error: Authentication failed]
    T3 -->|No| SUCCESS[Conexión exitosa]

    E1 --> RETRY[Reintentar en 5 min]
    E2 --> RETRY
    E3 --> ALERT[Notificar al administrador]
    RETRY --> T4{Reintentos < 3?}
    T4 -->|Sí| START
    T4 -->|No| MARK[Marcar dispositivo como ERROR]

    style SUCCESS fill:#c8e6c9
    style E1 fill:#ffcdd2
    style E2 fill:#ffcdd2
    style E3 fill:#ffcdd2
    style MARK fill:#ffe0b2
```

---

[Siguiente: Sincronización de Datos](./02-sincronizacion-de-datos.md) | [Anterior: Generación de PDF](../../05-modulo-reportes/04-generacion-de-pdf.md)
