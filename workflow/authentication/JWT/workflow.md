# JWT Authentication System - Workflow

## 🔄 **Complete Authentication Workflow**

This document outlines the complete workflow for the JWT authentication system, including user login, API access, and security operations.

## 🚀 **1. User Login Workflow**

### **Login Process:**
```mermaid
sequenceDiagram
    participant F as Frontend
    participant A as Authentication Service
    participant D as Database
    
    F->>A: POST /v1/authentication/login
    Note over F,A: { email, password }
    
    A->>D: Validate User Credentials
    D-->>A: User Data & Permissions
    
    A->>A: Generate JWT Token
    Note over A: JWT with user claims
    
    A-->>F: Login Response
    Note over F,A: { success, token, user }
```

### **Login Endpoint:**
```http
POST https://authentication.keeptrack.velonovo.com/v1/authentication/login
Content-Type: application/json

{
  "email": "user@example.com",
  "password": "userpassword"
}
```

### **Login Response:**
```json
{
  "success": true,
  "message": "Login successful",
  "data": {
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "user": {
      "id": "user-123",
      "email": "user@example.com",
      "name": "John Doe",
      "permissions": ["read", "write"]
    }
  }
}
```

## 🛡️ **2. API Request Workflow**

### **Protected API Access:**
```mermaid
sequenceDiagram
    participant F as Frontend
    participant FO as Foundation Service
    participant A as Authentication Service
    participant D as Database
    
    F->>FO: API Request + JWT
    Note over F,FO: Authorization: Bearer <token>
    
    FO->>A: POST /v1/guard/jwt/validate
    Note over FO,A: { token }
    
    A->>D: Check Token Validity
    A->>D: Update Request Counter
    
    D-->>A: Validation Result
    
    alt Token Valid
        A-->>FO: { valid: true, user }
        FO->>FO: Process API Request
        FO-->>F: API Response
    else Token Invalid
        A-->>FO: { valid: false, error }
        FO-->>F: 401 Unauthorized
    end
```

### **JWT Validation Endpoint:**
```http
POST https://authentication.keeptrack.velonovo.com/v1/guard/jwt/validate
Content-Type: application/json

{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

### **Validation Response:**
```json
{
  "success": true,
  "valid": true,
  "user": {
    "id": "user-123",
    "email": "user@example.com",
    "permissions": ["read", "write"]
  }
}
```

## 🔄 **3. JWT Refresh Workflow**

### **Token Refresh Process:**
```mermaid
sequenceDiagram
    participant F as Frontend
    participant A as Authentication Service
    participant D as Database
    
    F->>A: POST /v1/guard/jwt/refresh
    Note over F,A: { token }
    
    A->>D: Validate Refresh Token
    D-->>A: Token Validity
    
    alt Token Valid
        A->>A: Generate New JWT
        A-->>F: New Token
    else Token Invalid
        A-->>F: 401 Unauthorized
    end
```

### **Refresh Endpoint:**
```http
POST https://authentication.keeptrack.velonovo.com/v1/guard/jwt/refresh
Content-Type: application/json

{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

## 🚫 **4. JWT Revocation Workflow**

### **Token Revocation Process:**
```mermaid
sequenceDiagram
    participant F as Frontend
    participant A as Authentication Service
    participant D as Database
    
    F->>A: POST /v1/guard/jwt/revoke
    Note over F,A: { token }
    
    A->>D: Mark Token as Revoked
    D-->>A: Revocation Confirmed
    
    A-->>F: Revocation Success
```

### **Revocation Endpoint:**
```http
POST https://authentication.keeptrack.velonovo.com/v1/guard/jwt/revoke
Content-Type: application/json

{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

## 🔒 **5. Security Workflow**

### **Request Counter System:**
```mermaid
sequenceDiagram
    participant FO as Foundation Service
    participant A as Authentication Service
    participant D as Database
    
    FO->>A: JWT Validation Request
    A->>D: Check Token Validity
    
    alt Token Invalid
        A->>D: Increment Failed Count
        D-->>A: Updated Counter
        
        alt Failed Count >= 3
            A->>D: Block User
            A-->>FO: User Blocked
        else Failed Count < 3
            A-->>FO: Invalid Token
        end
    else Token Valid
        A->>D: Reset Failed Count
        A-->>FO: Valid Token
    end
```

### **User Blocking Logic:**
- **Threshold**: 3 failed JWT validation attempts
- **Blocking**: User blocked for 15 minutes
- **Recovery**: Automatic unblocking after timeout
- **Reset**: Successful validation resets counter

## 📊 **6. Monitoring Workflow**

### **JWT Statistics:**
```mermaid
sequenceDiagram
    participant A as Admin
    participant A as Authentication Service
    participant D as Database
    
    A->>A: GET /v1/guard/jwt/stats
    A->>D: Query JWT Statistics
    D-->>A: Statistics Data
    A-->>A: JWT Stats Response
```

### **Statistics Endpoint:**
```http
GET https://authentication.keeptrack.velonovo.com/v1/guard/jwt/stats
```

### **Statistics Response:**
```json
{
  "success": true,
  "data": {
    "total_validations": 1250,
    "successful_validations": 1200,
    "failed_validations": 50,
    "blocked_users": 5,
    "active_tokens": 150
  }
}
```

## 🔍 **7. Blocked Users Workflow**

### **Blocked Users Management:**
```mermaid
sequenceDiagram
    participant A as Admin
    participant A as Authentication Service
    participant D as Database
    
    A->>A: GET /v1/guard/jwt/blocked-users
    A->>D: Query Blocked Users
    D-->>A: Blocked Users Data
    A-->>A: Blocked Users Response
```

### **Blocked Users Endpoint:**
```http
GET https://authentication.keeptrack.velonovo.com/v1/guard/jwt/blocked-users
```

### **Blocked Users Response:**
```json
{
  "success": true,
  "data": [
    {
      "user_id": "user-123",
      "failed_count": 3,
      "blocked_until": "2024-01-15T10:30:00Z",
      "last_attempt": "2024-01-15T10:15:00Z"
    }
  ]
}
```

## 🔄 **8. Complete System Workflow**

### **End-to-End Authentication:**
```mermaid
graph TB
    subgraph "Frontend"
        F1[User Login]
        F2[API Requests]
        F3[Token Refresh]
        F4[Logout]
    end
    
    subgraph "Authentication Service"
        A1[Login Module]
        A2[Permission Module]
        A3[JWT Module]
        A4[Request Counter]
    end
    
    subgraph "Foundation Service"
        F5[JWT Middleware]
        F6[API Processing]
        F7[Response]
    end
    
    subgraph "Database"
        D1[User Data]
        D2[JWT Counters]
        D3[API Data]
    end
    
    F1 --> A1
    A1 --> A2
    A2 --> A3
    A3 --> D1
    
    F2 --> F5
    F5 --> A3
    A3 --> A4
    A4 --> D2
    
    F3 --> A3
    F4 --> A3
    
    F5 --> F6
    F6 --> D3
    F6 --> F7
```

## 🚀 **9. Production Workflow**

### **Production Endpoints:**
- **Authentication Service**: `https://authentication.keeptrack.velonovo.com`
- **Foundation Service**: `https://foundation.keeptrack.velonovo.com`

### **Frontend Integration:**
```javascript
// Login
const loginResponse = await fetch('https://authentication.keeptrack.velonovo.com/v1/authentication/login', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ email, password })
});

const { data } = await loginResponse.json();
const token = data.token;

// API Request
const apiResponse = await fetch('https://foundation.keeptrack.velonovo.com/v1/dashboard/assets', {
  headers: { 'Authorization': `Bearer ${token}` }
});
```

## 🔧 **10. Error Handling Workflow**

### **Common Error Scenarios:**
- **Invalid Credentials**: 401 Unauthorized
- **Invalid Token**: 401 Unauthorized
- **Blocked User**: 403 Forbidden
- **Token Expired**: 401 Unauthorized
- **Server Error**: 500 Internal Server Error

### **Error Response Format:**
```json
{
  "success": false,
  "message": "Authentication failed",
  "error": {
    "code": "INVALID_CREDENTIALS",
    "details": "Invalid email or password"
  }
}
```

## 📈 **11. Performance Workflow**

### **Optimization Strategies:**
- **JWT Validation**: Local validation in Foundation Service
- **Database**: Optimized queries with indexes
- **Caching**: JWT validation results cached
- **Load Balancing**: Multiple service instances

### **Monitoring Metrics:**
- **Response Time**: JWT validation latency
- **Throughput**: Requests per second
- **Error Rate**: Failed authentication percentage
- **Blocked Users**: Security incident tracking