# 8. Tecnologías Utilizadas

El sistema se implementó utilizando un stack tecnológico moderno, seleccionado por su robustez, escalabilidad y soporte a largo plazo.

---

## 8.1 Stack Tecnológico Completo

```mermaid
flowchart TB
    subgraph Frontend["Frontend - Capa de Presentación"]
        F1[Next.js 15<br/>Framework React]
        F2[React 19<br/>Librería UI]
        F3[Tailwind CSS 4<br/>Estilos]
        F4[Shadcn/ui<br/>Componentes]
        F5[TanStack Query<br/>Estado servidor]
        F6[NextAuth v5<br/>Autenticación]
    end

    subgraph Backend["Backend - Capa de Negocio"]
        B1[NestJS 11<br/>Framework]
        B2[TypeScript 5.7<br/>Lenguaje]
        B3[TypeORM 0.3.20<br/>ORM]
        B4[Keycloak<br/>Identidad]
        B5[node-zklib<br/>Dispositivos]
        B6[Puppeteer<br/>PDF Generation]
    end

    subgraph Database["Datos - Persistencia"]
        D1[PostgreSQL 15+<br/>Base de datos]
        D2[Redis<br/>Cache opcional]
    end

    subgraph DevOps["Desarrollo y Despliegue"]
        O1[Docker<br/>Contenedores]
        O2[Git<br/>Control de versiones]
        O3[Jest<br/>Testing]
        O4[ESLint/Prettier<br/>Calidad de código]
    end

    Frontend --> Backend
    Backend --> Database
    DevOps --> Frontend
    DevOps --> Backend

    style Frontend fill:#e3f2fd
    style Backend fill:#fff3e0
    style Database fill:#c8e6c9
    style DevOps fill:#f3e5f5
```

---

## 8.2 Tabla Detallada de Tecnologías

### Backend

| Tecnología | Versión | Propósito | Descripción |
|-----------|---------|-----------|-------------|
| **Node.js** | 20 LTS | Runtime | Entorno de ejecución JavaScript |
| **NestJS** | 11.x | Framework | Framework progresivo para aplicaciones Node.js |
| **TypeScript** | 5.7 | Lenguaje | Superset tipado de JavaScript |
| **TypeORM** | 0.3.20 | ORM | Mapeo objeto-relacional para PostgreSQL |
| **PostgreSQL** | 15+ | Base de datos | Sistema de base de datos relacional |
| **Keycloak** | 24+ | Identidad | Servidor de identidad y acceso |
| **nest-keycloak-connect** | Latest | Integración | Integración de Keycloak con NestJS |
| **@nestjs-cls/transactional** | Latest | Transacciones | Soporte transaccional con CLS |
| **node-zklib** | Latest | Dispositivos | Cliente para dispositivos ZKTeco |
| **Puppeteer** | Latest | PDF | Generación de PDF server-side |
| **PapaParse** | Latest | CSV | Procesamiento de archivos CSV |
| **Zod** | 4.x | Validación | Validación de esquemas |
| **Jest** | 29.x | Testing | Framework de pruebas |
| **Supertest** | Latest | Testing E2E | Pruebas de integración HTTP |

### Frontend

| Tecnología | Versión | Propósito | Descripción |
|-----------|---------|-----------|-------------|
| **Next.js** | 15.x | Framework | Framework React con App Router |
| **React** | 19.x | Librería UI | Librería para construir interfaces |
| **TypeScript** | 5.9 | Lenguaje | Tipado estático para JavaScript |
| **Tailwind CSS** | 4.0 | Estilos | Framework de utilidades CSS |
| **Shadcn/ui** | Latest | Componentes | Componentes UI reutilizables |
| **Radix UI** | Latest | Componentes | Primitivos accesibles |
| **TanStack Query** | 5.90 | Estado servidor | Gestión de estado y cache |
| **Zustand** | 5.0 | Estado cliente | Gestión de estado local |
| **React Hook Form** | 7.71 | Formularios | Gestión de formularios |
| **NextAuth** | v5 | Autenticación | Autenticación para Next.js |
| **Axios** | Latest | HTTP | Cliente HTTP |
| **Recharts** | 3.7 | Gráficos | Librería de visualización |
| **Framer Motion** | 12.28 | Animaciones | Librería de animaciones |
| **Lucide React** | Latest | Iconos | Librería de iconos |
| **Sonner** | Latest | Notificaciones | Toast notifications |

### Infraestructura

| Tecnología | Versión | Propósito | Descripción |
|-----------|---------|-----------|-------------|
| **Docker** | Latest | Contenedores | Empaquetado y despliegue |
| **Docker Compose** | Latest | Orquestación | Definición de servicios |
| **PostgreSQL** | 15-alpine | Imagen BD | Base de datos en contenedor |
| **Nginx** | Latest | Web Server | Servidor web y proxy reverso |

---

## 8.3 Patrones de Diseño Implementados

### Arquitectura Backend

```mermaid
flowchart TD
    subgraph Patrones["Patrones de Diseño Backend"]
        P1[DDD<br/>Domain-Driven Design]
        P2[Repository Pattern<br/>Acceso a datos]
        P3[Service Layer<br/>Lógica de negocio]
        P4[Interceptor Pattern<br/>Aspectos transversales]
        P5[Pipeline Pattern<br/>Procesamiento secuencial]
        P6[Strategy Pattern<br/>Algoritmos intercambiables]
        P7[Dependency Injection<br/>Inyección de dependencias]
    end

    subgraph Implementacion["Implementación"]
        I1[Módulos por dominio]
        I2[TypeORM Repositories]
        I3[NestJS Services]
        I4[NestJS Interceptors]
        I5[AttendanceEngine]
        I6[Report Strategies]
        I7[NestJS DI Container]
    end

    P1 --> I1
    P2 --> I2
    P3 --> I3
    P4 --> I4
    P5 --> I5
    P6 --> I6
    P7 --> I7

    style Patrones fill:#e3f2fd
    style Implementacion fill:#fff3e0
```

### Arquitectura Frontend

```mermaid
flowchart TD
    subgraph Patrones["Patrones de Diseño Frontend"]
        FP1[Feature-based<br/>Organización]
        FP2[Server Components<br/>Renderizado servidor]
        FP3[Cache-First<br/>Estrategia de datos]
        FP4[Compound Components<br/>Componentes compuestos]
        FP5[Custom Hooks<br/>Lógica reutilizable]
    end

    subgraph Implementacion["Implementación"]
        FI1[src/features/]
        FI2[Next.js App Router]
        FI3[TanStack Query]
        FI4[Shadcn/ui Patterns]
        FI5[useApiResponse, etc.]
    end

    FP1 --> FI1
    FP2 --> FI2
    FP3 --> FI3
    FP4 --> FI4
    FP5 --> FI5

    style Patrones fill:#e3f2fd
    style Implementacion fill:#fff3e0
```

---

## 8.4 Arquitectura de Despliegue

```mermaid
flowchart LR
    subgraph Client["Cliente"]
        B[Navegador Web]
    end

    subgraph Server["Servidor"]
        NX[Nginx<br/>Port 80/443]
        APP[Next.js App<br/>Port 3000]
        API[NestJS API<br/>Port 8000]
        KC[Keycloak<br/>Port 8080]
        PG[PostgreSQL<br/>Port 5432]
    end

    B --> NX
    NX --> APP
    NX --> API
    APP --> KC
    API --> KC
    API --> PG

    style NX fill:#c8e6c9
    style APP fill:#e3f2fd
    style API fill:#fff3e0
    style KC fill:#f3e5f5
    style PG fill:#e8f5e9
```

---

## 8.5 Justificación de Selección Tecnológica

### ¿ Por qué NestJS ?

| Criterio | Justificación |
|----------|---------------|
| **Arquitectura** | Estructura modular basada en DDD |
| **TypeScript** | Soporte nativo de tipado |
| **Inyección de dependencias** | Facilita testing y desacoplamiento |
| **Documentación** | Amplia documentación y comunidad |
| **Escalabilidad** | Arquitectura lista para escalar |

### ¿ Por qué Next.js ?

| Criterio | Justificación |
|----------|---------------|
| **Server Components** | Mejor performance y SEO |
| **App Router** | Estructura de archivos moderna |
| **TypeScript** | Tipado de extremo a extremo |
| **Ecosistema** | Amplia biblioteca de componentes |
| **Optimización** | Compilación automática y code splitting |

### ¿ Por qué Keycloak ?

| Criterio | Justificación |
|----------|---------------|
| **Estándar OIDC** | Compatible con OAuth 2.0 / OpenID Connect |
| **Funcionalidad** | Gestión completa de usuarios y roles |
| **Independencia** | No acopla a un proveedor específico |
| **Administración** | Consola de administración completa |
| **Seguridad** | Seguridad probada en producción |

### ¿ Por qué PostgreSQL ?

| Criterio | Justificación |
|----------|---------------|
| **Confiable** | Base de datos probada en producción |
| **Features** | Soporte de JSON, full-text search |
| **Transacciones** | ACID compliance |
| **Performance** | Optimizado para operaciones complejas |
| **Open Source** | Sin costos de licenciamiento |

---

[Anterior: Seguridad y Autenticación](./07-seguridad-y-autenticacion.md) | [Siguiente: Conclusiones](./09-conclusiones.md)
