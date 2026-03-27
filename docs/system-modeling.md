# System Modeling

## Data Models

### Entity Relationship Diagram (ERD)

```mermaid
erDiagram
    Webhook {
        UUID id PK
        string event_type
        text event
        datetime updated_at
        datetime created_at
    }
```

### Webhook Model

| Attribute | Type | Constraints | Description |
|-----------|------|-------------|-------------|
| `id` | UUID | Primary Key, Auto-generated | Unique identifier |
| `event_type` | String(50) | Nullable | Type of webhook event |
| `event` | Text | Nullable | Full JSON payload |
| `updated_at` | DateTime | Auto-now | Last modification timestamp |
| `created_at` | DateTime | Auto-now-add | Creation timestamp |

### Model Relationships

The `Webhook` model is a standalone entity with no foreign key relationships. It serves as an audit log for all incoming webhook events.

---

## System Architecture

### High-Level Architecture

```mermaid
flowchart TB
    subgraph "Main Inventory System"
        A[Inventory Management]
    end
    
    subgraph "Webhooks Management System"
        B[WebhookOrderView]
        C[Webhook Model]
        D[CallMeBot Service]
        E[Email Service]
    end
    
    subgraph "External Services"
        F[CallMeBot API]
        G[SMTP Server]
    end
    
    subgraph "Data Storage"
        H[(SQLite Database)]
    end
    
    A -->|POST /api/V1/webhooks/order/| B
    B -->|Persist| C
    C --> H
    B -->|Send Message| D
    B -->|Send Email| E
    D --> F
    E --> G
```

### Component Architecture

```mermaid
flowchart LR
    subgraph "Presentation Layer"
        A[REST API Endpoint]
        B[Django Admin]
    end
    
    subgraph "Business Logic Layer"
        C[WebhookOrderView]
        D[Business Calculations]
    end
    
    subgraph "Data Access Layer"
        E[Webhook Model]
        F[Django ORM]
    end
    
    subgraph "Integration Layer"
        G[CallMeBot Service]
        H[Email Service]
    end
    
    A --> C
    B --> E
    C --> D
    C --> E
    E --> F
    C --> G
    C --> H
```

### Deployment Architecture

```mermaid
flowchart TB
    subgraph "Client Layer"
        A[Main Inventory System]
        B[Admin User]
    end
    
    subgraph "Web Server"
        C[Nginx / Apache]
        D[Gunicorn / uWSGI]
    end
    
    subgraph "Application Layer"
        E[Django Application]
        F[CallMeBot Service]
        G[Email Service]
    end
    
    subgraph "Data Layer"
        H[(SQLite / PostgreSQL)]
        I[Static Files]
    end
    
    subgraph "External Services"
        J[CallMeBot API]
        K[SMTP Server]
    end
    
    A --> C
    B --> C
    C --> D
    D --> E
    E --> F
    E --> G
    E --> H
    E --> I
    F --> J
    G --> K
```

---

## Authentication Flow

### Current Authentication Flow (No Authentication)

```mermaid
flowchart LR
    A[Client] -->|POST Request| B[WebhookOrderView]
    B -->|No Auth Check| C[Process Event]
    C --> D[Return Response]
    
    style A fill:#e1f5ff
    style B fill:#fff3cd
    style C fill:#d4edda
```

### Recommended Authentication Flow (With API Key)

```mermaid
sequenceDiagram
    participant Client
    participant API
    participant Auth
    participant Database
    participant Notification
    
    Client->>API: POST /api/V1/webhooks/order/<br/>+ API-Key header
    API->>Auth: Validate API Key
    Auth-->>API: Valid/Invalid
    alt Invalid API Key
        API-->>Client: 401 Unauthorized
    else Valid API Key
        API->>Database: Persist Event
        Database-->>API: Event Saved
        API->>Notification: Send Notifications
        Notification-->>API: Notifications Sent
        API-->>Client: 200 OK + Calculated Data
    end
```

### Admin Authentication Flow

```mermaid
flowchart TD
    A[User] -->|Navigate to /admin/| B{Authenticated?}
    B -->|No| C[Login Page]
    C -->|Enter Credentials| D[Django Auth]
    D -->|Validate| E{Valid?}
    E -->|No| C
    E -->|Yes| F[Create Session]
    F --> G[Admin Dashboard]
    B -->|Yes| G
    G -->|Manage Webhooks| H[Webhook CRUD]
```

---

## CRUD Operations Flow

### Webhook Event Creation Flow

```mermaid
flowchart TD
    A[Inventory System] -->|POST Request| B[WebhookOrderView]
    B --> C{Validate Request}
    C -->|Invalid| D[Return 400 Error]
    C -->|Valid| E[Create Webhook Record]
    E --> F[Calculate total_value]
    F --> G[Calculate profit_value]
    G --> H[Send WhatsApp Notification]
    H --> I[Send Email Notification]
    I --> J{Notifications Success?}
    J -->|Yes| K[Return 200 OK]
    J -->|No| K
    K --> L[Inventory System Receives Response]
    
    style A fill:#e1f5ff
    style K fill:#d4edda
    style D fill:#f8d7da
```

### Complete CRUD Operations

```mermaid
flowchart LR
    subgraph "Create"
        A1[POST /api/V1/webhooks/order/] --> A2[Webhook.objects.create]
        A2 --> A3[(Database)]
    end
    
    subgraph "Read"
        B1[GET /admin/] --> B2[Webhook.objects.all]
        B2 --> B3[(Database)]
        B3 --> B4[Admin Display]
    end
    
    subgraph "Update"
        C1[Admin Edit] --> C2[Webhook.save]
        C2 --> C3[(Database)]
    end
    
    subgraph "Delete"
        D1[Admin Delete] --> D2[Webhook.delete]
        D2 --> D3[(Database)]
    end
```

---

## Security Flow

### Current Security Flow (Minimal)

```mermaid
flowchart LR
    A[Request] --> B[No Authentication]
    B --> C[No Authorization]
    C --> D[Process Request]
    D --> E[Return Response]
    
    style A fill:#e1f5ff
    style B fill:#f8d7da
    style C fill:#f8d7da
    style E fill:#fff3cd
```

### Recommended Security Flow

```mermaid
flowchart TD
    A[Incoming Request] --> B{HTTPS?}
    B -->|No| C[Reject Request]
    B -->|Yes| D{API Key Present?}
    D -->|No| E[Return 401]
    D -->|Yes| F{API Key Valid?}
    F -->|No| E
    F -->|Yes| G{Rate Limit OK?}
    G -->|No| H[Return 429]
    G -->|Yes| I{Payload Valid?}
    I -->|No| J[Return 400]
    I -->|Yes| K[Process Request]
    K --> L[Log Event]
    L --> M[Return Response]
    
    style A fill:#e1f5ff
    style C fill:#f8d7da
    style E fill:#f8d7da
    style H fill:#f8d7da
    style J fill:#f8d7da
    style M fill:#d4edda
```

### Data Security Flow

```mermaid
flowchart TB
    subgraph "Data In Transit"
        A[Client] -->|HTTPS/TLS| B[Server]
    end
    
    subgraph "Data at Rest"
        B -->|Encrypt| C[(Database)]
    end
    
    subgraph "Sensitive Data Handling"
        D[Environment Variables] --> E[Application]
        E -->|Never Log| F[API Keys]
        E -->|Never Log| G[Passwords]
        E -->|Never Log| H[Secret Keys]
    end
    
    subgraph "Audit Trail"
        I[All Requests] --> J[Webhook Log]
        J --> K[Admin Access Log]
    end
```

---

## Notification Flow

### WhatsApp Notification Flow

```mermaid
sequenceDiagram
    participant V as WebhookOrderView
    participant M as outflow_message
    participant C as CallMeBot Service
    participant A as CallMeBot API
    participant W as WhatsApp
    
    V->>M: Format Message
    M-->>V: Formatted Message
    V->>C: send_message()
    C->>A: HTTP GET Request
    A->>W: Send WhatsApp Message
    W-->>A: Delivery Confirmation
    A-->>C: Response
    C-->>V: Result
```

### Email Notification Flow

```mermaid
sequenceDiagram
    participant V as WebhookOrderView
    participant T as outflow.html Template
    participant D as Django Email
    participant S as SMTP Server
    participant R as Recipient
    
    V->>T: Render Template
    T-->>V: HTML Content
    V->>D: send_mail()
    D->>S: SMTP Connection
    S->>R: Deliver Email
    R-->>S: Acknowledgment
    S-->>D: Send Status
    D-->>V: Result
```

---

## Business Logic Flow

### Value Calculation Flow

```mermaid
flowchart LR
    A[Request Data] --> B[Extract Values]
    B --> C[Calculate total_value]
    C --> D[selling_price × quantity]
    D --> E[Calculate profit_value]
    E --> F[total_value - cost_price × quantity]
    F --> G[Return Results]
    
    style A fill:#e1f5ff
    style G fill:#d4edda
```

### Complete Request Processing Flow

```mermaid
flowchart TD
    A[POST Request] --> B[Parse JSON Body]
    B --> C[Extract Fields]
    C --> D{Required Fields Present?}
    D -->|No| E[Return 400]
    D -->|Yes| F[Create Webhook Record]
    F --> G[Calculate Business Values]
    G --> H[Format WhatsApp Message]
    H --> I[Send WhatsApp Notification]
    I --> J[Render Email Template]
    J --> K[Send Email Notification]
    K --> L[Build Response]
    L --> M[Return 200 OK]
    
    style A fill:#e1f5ff
    style E fill:#f8d7da
    style M fill:#d4edda
```

---

## Error Handling Flow

### Exception Flow

```mermaid
flowchart TD
    A[Request Processing] --> B{Error Occurs?}
    B -->|No| C[Success Response]
    B -->|Yes| D{Error Type}
    D -->|KeyError| E[Log Missing Field]
    E --> F[Return 400]
    D -->|RequestException| G[Log Service Error]
    G --> H[Continue Processing]
    D -->|Exception| I[Log Unexpected Error]
    I --> J[Return 500]
    
    style A fill:#e1f5ff
    style C fill:#d4edda
    style F fill:#fff3cd
    style H fill:#fff3cd
    style J fill:#f8d7da
```

---

## Next Steps

- Review [API Endpoints](api-endpoints.md) for detailed endpoint documentation
- Check [Authentication & Security](authentication-security.md) for security implementation details
- See [Development Guide](development.md) for implementation guidelines
