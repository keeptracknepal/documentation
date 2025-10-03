# Authentication Process Flow

## Hybrid Authentication Architecture

```mermaid
flowchart TD
    %% User Interaction
    A[User] --> B[Frontend Application]
    B --> C{Authentication Required?}
    
    %% Login Flow
    C -->|Yes| D[POST /auth/login]
    D --> E[Authentication Service<br/>Port 8302]
    E --> F[Validate Credentials]
    F --> G{Valid?}
    G -->|Yes| H[Generate JWT Token]
    G -->|No| I[Return 401 Error]
    H --> J[Return JWT + User Info]
    J --> K[Store Token in Frontend]
    
    %% API Request Flow
    C -->|No| L[API Request with JWT]
    K --> L
    L --> M[Foundation Service<br/>Port 8300]
    M --> N[JWT Middleware]
    N --> O{Token Valid?}
    O -->|Yes| P[Process Business Logic]
    O -->|No| Q[Return 401/403 Error]
    P --> R[Database Operations]
    R --> S[Return API Response]
    S --> T[Frontend Updates UI]
    
    %% Logout Flow
    T --> U{User Logout?}
    U -->|Yes| V[POST /auth/logout]
    V --> E
    E --> W[Invalidate JWT Token]
    W --> X[Clear Token from Frontend]
    U -->|No| L
    
    %% Error Handling
    I --> Y[Show Login Form]
    Q --> Y
    Y --> D
    
    %% Styling
    classDef authService fill:#1976d2,stroke:#0d47a1,stroke-width:2px,color:#ffffff
    classDef foundationService fill:#7b1fa2,stroke:#4a148c,stroke-width:2px,color:#ffffff
    classDef frontend fill:#388e3c,stroke:#1b5e20,stroke-width:2px,color:#ffffff
    classDef database fill:#f57c00,stroke:#e65100,stroke-width:2px,color:#ffffff
    classDef error fill:#d32f2f,stroke:#b71c1c,stroke-width:2px,color:#ffffff
    
    class E,H,W authService
    class M,N,P,R foundationService
    class B,K,T,X frontend
    class F,R database
    class I,Q,Y error
```

## Service Communication Flow

```mermaid
sequenceDiagram
    participant U as User
    participant F as Frontend
    participant A as Authentication Service (8302)
    participant FO as Foundation Service (8300)
    participant D as Database
    
    %% Login Sequence
    U->>F: Enter credentials
    F->>A: POST /auth/login
    A->>D: Validate user credentials
    D-->>A: User data
    A->>A: Generate JWT token
    A-->>F: JWT token + user info
    F->>F: Store JWT token
    
    %% API Request Sequence
    U->>F: Make API request
    F->>FO: API request + JWT token
    FO->>FO: Validate JWT locally
    alt Token Valid
        FO->>D: Business logic operations
        D-->>FO: Business data
        FO-->>F: API response
        F-->>U: Update UI
    else Token Invalid
        FO-->>F: 401/403 Error
        F-->>U: Redirect to login
    end
    
    %% Logout Sequence
    U->>F: Logout
    F->>A: POST /auth/logout
    A->>A: Invalidate JWT token
    A-->>F: Logout confirmation
    F->>F: Clear stored token
    F-->>U: Redirect to login
```

## System Architecture Overview

```mermaid
graph TB
    subgraph "Frontend Layer"
        WA[Web App]
        MA[Mobile App]
        AP[Admin Panel]
    end
    
    subgraph "Authentication Layer"
        AS[Authentication Service<br/>Port 8302]
        AS1[Login Module]
        AS2[JWT Management]
        AS3[User Management]
    end
    
    subgraph "Business Logic Layer"
        FS[Foundation Service<br/>Port 8300]
        FS1[JWT Middleware]
        FS2[Dashboard Module]
        FS3[Assets Module]
        FS4[Settings Module]
    end
    
    subgraph "Data Layer"
        DB[(Supabase Database)]
        T1[Users Table]
        T2[Permissions Table]
        T3[Business Data]
    end
    
    %% Connections
    WA --> AS
    MA --> AS
    AP --> AS
    
    WA --> FS
    MA --> FS
    AP --> FS
    
    AS --> AS1
    AS --> AS2
    AS --> AS3
    
    FS --> FS1
    FS --> FS2
    FS --> FS3
    FS --> FS4
    
    AS --> DB
    FS --> DB
    
    DB --> T1
    DB --> T2
    DB --> T3
    
    %% Styling
    classDef frontend fill:#388e3c,stroke:#1b5e20,stroke-width:2px,color:#ffffff
    classDef auth fill:#1976d2,stroke:#01579b,stroke-width:2px,color:#ffffff
    classDef foundation fill:#7b1fa2,stroke:#4a148c,stroke-width:2px,color:#ffffff
    classDef database fill:#f57c00,stroke:#e65100,stroke-width:2px,color:#ffffff
    
    class WA,MA,AP frontend
    class AS,AS1,AS2,AS3 auth
    class FS,FS1,FS2,FS3,FS4 foundation
    class DB,T1,T2,T3 database
```

## JWT Token Lifecycle

```mermaid
stateDiagram-v2
    [*] --> LoginRequest : User submits credentials
    
    LoginRequest --> ValidateCredentials : POST /auth/login
    ValidateCredentials --> CredentialsValid : Database check
    ValidateCredentials --> CredentialsInvalid : Invalid credentials
    
    CredentialsValid --> GenerateJWT : Create JWT token
    GenerateJWT --> TokenIssued : Sign with JWT_SECRET
    TokenIssued --> StoreToken : Frontend stores token
    StoreToken --> APIRequest : User makes API call
    
    APIRequest --> ValidateJWT : Foundation Service
    ValidateJWT --> JWTValid : Local validation
    ValidateJWT --> JWTInvalid : Invalid/expired token
    
    JWTValid --> ProcessRequest : Business logic
    ProcessRequest --> APIResponse : Return data
    APIResponse --> APIRequest : Continue using token
    
    JWTInvalid --> LoginRequest : Redirect to login
    CredentialsInvalid --> LoginRequest : Show error message
    
    APIRequest --> LogoutRequest : User logs out
    LogoutRequest --> InvalidateToken : POST /auth/logout
    InvalidateToken --> ClearToken : Remove from frontend
    ClearToken --> [*] : Return to login
```

## Environment Configuration

```mermaid
graph LR
    subgraph "Authentication Service"
        A1[.env file]
        A2[JWT_SECRET]
        A3[AUTH_PORT=8302]
        A4[DATABASE_URL]
    end
    
    subgraph "Foundation Service"
        F1[.env file]
        F2[JWT_SECRET]
        F3[PORT=8300]
        F4[DATABASE_URL]
    end
    
    A2 -.->|Same Secret| F2
    A4 -.->|Same Database| F4
    
    A1 --> A2
    A1 --> A3
    A1 --> A4
    
    F1 --> F2
    F1 --> F3
    F1 --> F4
    
    classDef env fill:#1976d2,stroke:#0277bd,stroke-width:2px,color:#ffffff
    classDef secret fill:#388e3c,stroke:#33691e,stroke-width:2px,color:#ffffff
    
    class A1,F1 env
    class A2,F2 secret
```

## Key Benefits

```mermaid
mindmap
  root((Hybrid Authentication))
    Performance
      Fast API Calls
      Local JWT Validation
      No Gateway Overhead
    Security
      JWT Cryptographically Signed
      Shared Secret Validation
      No Token Exposure
    Scalability
      Independent Service Scaling
      Clear Separation of Concerns
      Simple Deployment
    Maintainability
      Centralized Authentication
      Isolated Business Logic
      Easy Debugging
```
