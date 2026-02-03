# 7. Seguridad y Autenticación

El sistema implementó un modelo de seguridad robusto basado en **Keycloak** como proveedor de identidad y **NextAuth** para la integración con el frontend.

---

## 7.1 Arquitectura de Seguridad

```mermaid
flowchart TD
    subgraph Frontend["Frontend Next.js"]
        NA[NextAuth Provider]
        SC[Session Management]
    end

    subgraph Backend["Backend NestJS"]
        KG[Keycloak Guard]
        RS[Role Guard]
        TC[Token Refresher]
    end

    subgraph Identity["Proveedor de Identidad"]
        KC[Keycloak Server]
        UR[User Registry]
        RL[Role Mapping]
    end

    subgraph Resources["Recursos Protegidos"]
        API1[API Endpoints]
        API2[Dashboard Pages]
        API3[Admin Routes]
    end

    NA <-->|OIDC| KC
    NA --> SC
    SC -->|Bearer Token| API2
    API1 --> KG
    KG --> KC
    API1 --> RS
    API2 --> KG
    API3 --> RS
    KC --> UR
    KC --> RL

    style KC fill:#c8e6c9
    style KG fill:#fff3e0
    style RS fill:#e3f2fd
```

---

## 7.2 Flujo de Autenticación

```mermaid
sequenceDiagram
    actor U as Usuario
    participant F as Frontend
    participant NA as NextAuth
    participant KC as Keycloak
    participant B as Backend API
    participant R como Resource

    U->>F: Ingresa a la aplicación
    F->>NA: checkSession()
    NA->>NA: ¿Hay sesión activa?
    alt No hay sesión
        NA-->>F: Redirect a /login
        F-->>U: Muestra formulario de login
        U->>F: Ingresa credenciales
        F->>NA: signIn("keycloak")
        NA->>KC: Authorization Code Flow
        KC-->>NA: Authorization Code
        NA->>KC: Exchange code for tokens
        KC-->>NA: Access Token + ID Token + Refresh Token
        NA->>NA: Validate tokens
        NA-->>F: Session establecida
        F->>R: Redirect a /dashboard
    end

    F->>B: API Request con Bearer Token
    B->>B: Keycloak Guard valida token
    B-->>F: Response con datos
    F-->>U: Muestra información
```

---

## 7.3 Configuración de Keycloak

### Realm Configuration

```mermaid
flowchart TD
    subgraph Realm["hr-system Realm"]
        C[Clients]
        U[Users]
        R[Roles]
        G[Groups]
    end

    subgraph Client["hr-frontend Client"]
        CT[Client Type: public]
        CA[Auth: OIDC]
        CR[Redirect URIs]
    end

    subgraph Roles_Def["Roles Definidos"]
        R1[DOCENTE]
        R2[ADMINISTRATIVO]
        R3[RRHH]
        R4[ADMIN]
    end

    Realm --> Client
    Realm --> Roles_Def
    Client --> CT
    Client --> CA
    Client --> CR

    style Realm fill:#c8e6c9
    style Roles_Def fill:#fff3e0
```

### Roles y Permisos

| Rol | Permisos | Acceso |
|-----|----------|--------|
| **DOCENTE** | Ver su propio reporte | Dashboard personal |
| **ADMINISTRATIVO** | Ver su propio reporte | Dashboard personal |
| **RRHH** | Ver todos los reportes, gestionar usuarios | Reportes administrativos |
| **ADMIN** | Configuración completa, gestión de dispositivos | Panel de administración |

---

## 7.4 Protección de Endpoints

### Guards en NestJS

```mermaid
flowchart LR
    subgraph Request["HTTP Request"]
        R[Incoming Request]
    end

    subgraph Guards["Guards de Seguridad"]
        G1[KeycloakGuard<br/>Valida token]
        G2[RolesGuard<br/>Valida roles]
        G3[PermissionsGuard<br/>Valida permisos]
    end

    subgraph Controller["Controller"]
        C[Endpoint Handler]
    end

    R --> G1
    G1 -->|Token válido| G2
    G1 -->|Token inválido| E1[401 Unauthorized]
    G2 -->|Rol OK| G3
    G2 -->|Rol insuficiente| E2[403 Forbidden]
    G3 -->|Permiso OK| C
    G3 -->|Permiso denegado| E2

    style E1 fill:#ffcdd2
    style E2 fill:#ffe0b2
    style C fill:#c8e6c9
```

### Ejemplo de Decoradores

```typescript
// Endpoint protegido con autenticación
@UseGuards(KeycloakGuard)
@Get('me')
getMyReport(@CurrentUser() user: User) {
  return this.reportService.getTodayReport(user.uuid);
}

// Endpoint protegido con rol
@UseGuards(KeycloakGuard, RolesGuard)
@Roles('RRHH', 'ADMIN')
@Get('admin')
getAdminReport() {
  return this.reportService.getAllReports();
}
```

---

## 7.5 Gestión de Sesiones en Frontend

### NextAuth Configuration

```mermaid
flowchart TD
    subgraph Config["NextAuth Config"]
        P[Provider: Keycloak]
        C[Callbacks]
        S[Session Strategy]
        J[JWT Strategy]
    end

    subgraph Callbacks["Callbacks Implementados"]
        CB1[jwt callback<br/>Agregar claims al token]
        CB2[session callback<br/>Actualizar sesión]
        CB3[signIn callback<br/>Validar usuario]
    end

    subgraph Session["Session Management"]
        SM[Update session]
        SR[Refresh token]
        SD[Delete session]
    end

    Config --> Callbacks
    Config --> Session
    P --> CB1
    C --> CB2
    S --> SM
    J --> SR
    S --> SD

    style Config fill:#e3f2fd
    style Callbacks fill:#fff3e0
    style Session fill:#c8e6c9
```

---

## 7.6 Token Management

### Tipos de Token

| Tipo | Propósito | Duración |
|------|-----------|----------|
| **Access Token** | Acceso a recursos API | 5 minutos |
| **Refresh Token** | Obtener nuevo access token | 30 días |
| **ID Token** | Información del usuario | 5 minutos |

### Flujo de Refresh

```mermaid
sequenceDiagram
    participant C as Client
    participant S as Session Storage
    participant NA as NextAuth
    participant KC as Keycloak

    Note over C,KC: Access Token expirado

    C->>S: Obtener refresh_token
    C->>NA: refreshAccessToken()
    NA->>KC: POST /token con refresh_token
    KC-->>NA: Nuevo access_token
    NA-->>C: Actualizada sesión
    C->>C: Reintentar request original
```

---

## 7.7 Auditoría y Logging

### Registro de Eventos de Seguridad

```mermaid
flowchart TD
    subgraph Events["Eventos de Seguridad Auditados"]
        E1[Login exitoso]
        E2[Login fallido]
        E3[Cambio de contraseña]
        E4[Logout]
        E5[Acceso a recurso denegado]
        E6[Acceso a recurso administrativo]
    end

    subgraph Log["Información Registrada"]
        L1[Timestamp]
        L2[Usuario]
        L3[IP Address]
        L4[User Agent]
        L5[Acción]
        L6[Resultado]
    end

    Events --> Log

    style Events fill:#e3f2fd
    style Log fill:#fff3e0
```

---

## 7.8 Consideraciones de Seguridad

### Mejores Prácticas Implementadas

| Aspecto | Implementación |
|---------|----------------|
| **HTTPS** | Todo el tráfico sobre TLS |
| **Tokens** | Almacenamiento seguro en cookies httpOnly |
| **CSRF** | Tokens CSRF en formularios |
| **XSS** | Sanitización de inputs, CSP headers |
| **SQL Injection** | Queries parametrizadas con TypeORM |
| **Authorization** | Verificación de roles en cada endpoint |
| **Audit Trail** | Todas las entidades con auditoría |

---

[Anterior: Integración de Dispositivos](./06-integracion-dispositivos/02-sincronizacion-de-datos.md) | [Siguiente: Tecnologías Utilizadas](./08-tecnologias-utilizadas.md)
