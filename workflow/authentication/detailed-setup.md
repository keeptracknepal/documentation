# Authentication Service - Detailed Setup

## Overview
This document provides detailed setup instructions for the KeepTrack Authentication Service, including environment configuration, service deployment, and integration steps.

## Service Structure

### Directory Structure
```
authentication/
├── connection/          # Database connection (Supabase)
│   └── supabase.go     # Database connection logic
├── server/             # HTTP server setup
│   └── server.go       # Server configuration
├── routes/             # API route definitions
│   └── routes.go       # Route configuration
├── v1/                 # Authentication modules
│   ├── guard/          # Permission management
│   │   ├── handler/    # HTTP handlers
│   │   ├── service/    # Business logic
│   │   ├── repository/ # Data access
│   │   ├── dto/        # Data transfer objects
│   │   └── models/     # Data models
│   ├── login/          # User authentication
│   │   ├── handler/    # HTTP handlers
│   │   ├── service/    # Business logic
│   │   ├── repository/ # Data access
│   │   ├── dto/        # Data transfer objects
│   │   └── models/     # Data models
│   └── authentication.module.go
├── main.go             # Service entry point
├── go.mod              # Dependencies
├── .env.development    # Development environment
└── .env.production     # Production environment
```

## Environment Configuration

### Development Environment (.env.development)
```bash
# JWT Configuration
JWT_SECRET=your-jwt-secret-here

# Service Configuration
AUTH_PORT=8302

# Database Configuration
DATABASE_URL=postgresql://user:password@host:port/database?sslmode=require
```

### Production Environment (.env.production)
```bash
# JWT Configuration
JWT_SECRET=your-jwt-secret-here

# Service Configuration
AUTH_PORT=8302

# Database Configuration
DATABASE_URL=postgresql://user:password@host:port/database?sslmode=require
```

## Service Setup

### 1. Database Connection
The authentication service uses Supabase PostgreSQL for data storage:

```go
// connection/supabase.go
type AuthenticationDatabase struct {
    DB *sql.DB
}

func NewAuthenticationDatabase() *AuthenticationDatabase {
    databaseURL := os.Getenv("DATABASE_URL")
    // Connection logic with retry mechanism
}
```

### 2. Server Configuration
The server is configured with Fiber framework:

```go
// server/server.go
func StartServer() {
    srv := NewServer()
    routes.SetupAuthenticationRoutes(srv.GetApp(), srv.GetDB())
    
    port := os.Getenv("AUTH_PORT")
    if port == "" {
        port = "8302"
    }
    
    srv.Start(":" + port)
}
```

### 3. Route Configuration
Authentication routes are organized under `/v1` prefix:

```go
// routes/routes.go
func SetupAuthenticationRoutes(app *fiber.App, db *connection.AuthenticationDatabase) {
    v1 := app.Group("/v1")
    
    // Authentication routes
    authRoutes := authModule.GetRoutes()
    for _, route := range authRoutes {
        v1.Add(route.Method, route.Path, route.Handler)
    }
    
    // Guard routes
    authModule.RegisterRoutes(v1)
}
```

## API Endpoints

### Authentication Endpoints
```
POST   /v1/login           - User authentication
POST   /v1/logout          - User logout
GET    /v1/me             - Current user info
GET    /v1/permissions    - User permissions
GET    /v1/guard/me       - Guard permissions
```

### Request/Response Examples

#### Login Request
```json
POST /v1/login
{
    "email": "user@example.com",
    "password": "password123"
}
```

#### Login Response
```json
{
    "success": true,
    "message": "Login successful",
    "data": {
        "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
        "user": {
            "id": "123",
            "email": "user@example.com",
            "name": "John Doe",
            "role": "admin"
        },
        "permissions": {
            "dashboard": ["read", "write"],
            "users": ["read"]
        }
    }
}
```

#### Logout Request
```json
POST /v1/logout
Authorization: Bearer <jwt-token>
```

#### Logout Response
```json
{
    "success": true,
    "message": "Logout successful"
}
```

## Security Configuration

### JWT Token Configuration
- **Algorithm**: HS256 (HMAC with SHA-256)
- **Expiration**: Configurable (default: 24 hours)
- **Secret**: Shared between Authentication and Foundation services
- **Claims**: User ID, email, role, permissions

### Password Security
- **Hashing**: Bcrypt with salt rounds
- **Minimum Length**: 8 characters
- **Complexity**: Not enforced (can be added)

### CORS Configuration
```go
app.Use(cors.New(cors.Config{
    AllowOrigins: "https://keeptrack.velonovo.com",
    AllowMethods: "GET,POST,PUT,DELETE,OPTIONS",
    AllowHeaders: "Origin,Content-Type,Accept,Authorization",
}))
```

## Database Schema

### Users Table
```sql
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    name VARCHAR(255) NOT NULL,
    role VARCHAR(50) NOT NULL,
    access_list JSONB,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);
```

### Access List Structure
```json
{
    "dashboard": ["read", "write"],
    "users": ["read"],
    "settings": ["read", "write"],
    "reports": ["read"]
}
```

## Deployment

### Docker Configuration
```dockerfile
FROM golang:1.24-alpine AS builder
WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN go build -o authentication main.go

FROM alpine:latest
RUN apk --no-cache add ca-certificates
WORKDIR /root/
COPY --from=builder /app/authentication .
COPY --from=builder /app/.env.production .
CMD ["./authentication"]
```

### Docker Compose
```yaml
version: '3.8'
services:
  authentication:
    build: .
    ports:
      - "8302:8302"
    environment:
      - JWT_SECRET=${JWT_SECRET}
      - AUTH_PORT=8302
      - DATABASE_URL=${DATABASE_URL}
    volumes:
      - ./.env.production:/app/.env.production
```

## Monitoring and Logging

### Health Checks
The service includes built-in health monitoring:
- Database connection status
- Service availability
- Performance metrics

### Logging Configuration
```go
// Structured logging with context
log.Printf("🔍 Authentication request from IP: %s", c.IP())
log.Printf("✅ Authentication successful for user: %s", user.Email)
log.Printf("❌ Authentication failed: %v", err)
```

## Integration with Foundation Service

### Shared Configuration
Both services must use the same JWT_SECRET:
```bash
# Authentication Service
JWT_SECRET=your-jwt-secret-here

# Foundation Service  
JWT_SECRET=your-jwt-secret-here
```

### Token Validation
The Foundation Service validates tokens locally using the shared secret:
```go
// foundation/middleware/jwt.go
func JWTMiddleware() fiber.Handler {
    return func(c *fiber.Ctx) error {
        // Validate JWT token using shared secret
        token, err := jwt.Parse(tokenString, func(token *jwt.Token) (interface{}, error) {
            return []byte(jwtSecret), nil
        })
    }
}
```

## Troubleshooting

### Common Issues

#### 1. Database Connection Failed
```bash
# Check database URL
echo $DATABASE_URL

# Test connection
psql $DATABASE_URL -c "SELECT 1;"
```

#### 2. JWT Secret Mismatch
```bash
# Verify both services use same secret
echo $JWT_SECRET
```

#### 3. Port Already in Use
```bash
# Check port usage
netstat -tulpn | grep 8302

# Kill process if needed
sudo kill -9 <PID>
```

### Debug Mode
Enable debug logging:
```bash
export DEBUG=true
./authentication
```

## Performance Optimization

### Database Connection Pool
```go
db.SetMaxOpenConns(15)
db.SetMaxIdleConns(5)
db.SetConnMaxLifetime(30 * time.Minute)
db.SetConnMaxIdleTime(15 * time.Minute)
```

### JWT Token Optimization
- Use short expiration times
- Implement token refresh mechanism
- Cache user permissions

## Security Best Practices

1. **Never commit JWT secrets to version control**
2. **Use environment variables for sensitive data**
3. **Implement rate limiting for login attempts**
4. **Use HTTPS in production**
5. **Regular security audits**
6. **Monitor for suspicious activity**

## Maintenance

### Regular Tasks
- Monitor service health
- Update dependencies
- Review security logs
- Backup database
- Performance monitoring

### Updates
- Test in development first
- Use blue-green deployment
- Monitor after deployment
- Rollback plan ready
