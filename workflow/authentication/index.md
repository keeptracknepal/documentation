# KeepTrack Authentication Documentation

## Overview
This directory contains comprehensive documentation for the KeepTrack Authentication Service, which implements a **Hybrid Authentication Architecture** for secure user authentication and authorization across the entire KeepTrack ecosystem.

## Documentation Structure

### 📋 [Summary](summary.md)
Complete overview of the authentication service including:
- Service architecture and components
- Production endpoints
- Hybrid authentication flow
- Frontend integration examples
- Security model and benefits

### 🏗️ [Architecture](architecture.md)
Detailed system architecture documentation including:
- Hybrid authentication architecture overview
- Service components and responsibilities
- Data flow architecture
- Security architecture
- Service communication patterns

### ⚙️ [Detailed Setup](detailed-setup.md)
Comprehensive setup and configuration guide including:
- Service structure and configuration
- Environment setup
- API endpoints and examples
- Security configuration
- Deployment instructions
- Troubleshooting guide

### 🔄 [Process Flow](process-flow.md)
Visual workflow documentation with Mermaid diagrams including:
- Service communication flow
- Authentication states
- Error handling
- Frontend integration
- Environment configuration
- Security considerations

## Key Features

### 🔐 **Hybrid Authentication Architecture**
- **Authentication Service**: Handles login and JWT token generation
- **Foundation Service**: Handles API requests with local JWT validation
- **No Direct Communication**: Services don't talk to each other
- **Shared JWT Secret**: Both services use the same secret for signing/validation

### 🌐 **Production Endpoints**

#### Authentication Service
```
https://authentication.keeptrack.velonovo.com/v1/login
https://authentication.keeptrack.velonovo.com/v1/logout
https://authentication.keeptrack.velonovo.com/v1/me
https://authentication.keeptrack.velonovo.com/v1/permissions
https://authentication.keeptrack.velonovo.com/v1/guard/me
```

#### Foundation Service
```
https://foundation.keeptrack.velonovo.com/v1/dashboard/*
https://foundation.keeptrack.velonovo.com/v1/assets/*
https://foundation.keeptrack.velonovo.com/v1/settings/*
https://foundation.keeptrack.velonovo.com/v1/branches/*
```

### 🚀 **Benefits**
- **Fast Performance**: Direct API calls with local JWT validation
- **Secure**: JWT tokens cryptographically signed
- **Scalable**: Each service can scale independently
- **Maintainable**: Clear separation of concerns

## Quick Start

### 1. **Environment Setup**
Both services require the same JWT_SECRET:
```bash
# Authentication Service (.env)
JWT_SECRET=your-jwt-secret-here
AUTH_PORT=8302
DATABASE_URL=postgresql://...

# Foundation Service (.env)
JWT_SECRET=your-jwt-secret-here
PORT=8300
DATABASE_URL=postgresql://...
```

### 2. **Frontend Integration**
```javascript
// Login
const response = await fetch('https://authentication.keeptrack.velonovo.com/v1/login', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ email, password })
});

// API Request
const token = localStorage.getItem('jwt_token');
const response = await fetch('https://foundation.keeptrack.velonovo.com/v1/dashboard', {
  headers: { 'Authorization': `Bearer ${token}` }
});
```

## Documentation Files

| File | Description |
|-----|-------------|
| `summary.md` | Complete service overview |
| `architecture.md` | Detailed system architecture |
| `detailed-setup.md` | Setup and configuration guide |
| `process-flow.md` | Visual workflow with Mermaid diagrams |
| `index.md` | This overview file |

## Visual Documentation

The `process-flow.md` file includes **Mermaid diagrams** that render automatically in Git repositories (GitHub, GitLab, etc.) for:
- Process flow visualization
- Service communication sequences
- System architecture overview
- JWT token lifecycle

## Security

- **JWT Tokens**: Secure, stateless authentication
- **Password Hashing**: Bcrypt encryption
- **CORS Support**: Cross-origin request handling
- **Shared Secret**: Both services use the same JWT_SECRET
- **Local Validation**: Foundation Service validates tokens locally

## Support

For questions or issues related to the authentication service, refer to the detailed documentation in each file or contact the development team.

---

**Last Updated**: Current
**Version**: Production Ready
**Status**: Active
