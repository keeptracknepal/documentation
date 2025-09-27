# Go Backend - MVC Architecture Standards

## Overview
This document defines the Model-View-Controller (MVC) architecture standards for Go backend projects using GoFiber framework. The architecture emphasizes clean separation of concerns, scalability, and maintainability.

## Core Architecture Principles

### 1. MVC Pattern
- **Model**: Data structures and business entities
- **View**: API responses and data presentation  
- **Controller**: HTTP request/response handling

### 2. Layered Architecture
```
┌─────────────────┐
│    Handler      │ ← HTTP Layer (Controllers)
├─────────────────┤
│    Service      │ ← Business Logic Layer
├─────────────────┤
│   Repository    │ ← Data Access Layer
├─────────────────┤
│     Models      │ ← Data Structures
├─────────────────┤
│      DTO        │ ← Data Transfer Objects
└─────────────────┘
```

### 3. Separation of Concerns
- **Single Responsibility**: Each component has one reason to change
- **Dependency Inversion**: High-level modules don't depend on low-level modules
- **Interface Segregation**: Small, focused interfaces

## Layer Responsibilities

### 1. Handler Layer (Controllers)
**Purpose**: Handle HTTP requests and responses
**Responsibilities**:
- Parse and validate incoming requests
- Call appropriate service methods
- Format and return responses
- Handle HTTP-specific concerns (status codes, headers)

```go
// ✅ Correct: Clean handler implementation
type AssetHandler struct {
    assetService AssetService
}

func (h *AssetHandler) CreateAsset(c *fiber.Ctx) error {
    var req CreateAssetRequest
    if err := c.BodyParser(&req); err != nil {
        return c.Status(400).JSON(fiber.Map{
            "success": false,
            "message": "Invalid request body",
            "error": err.Error(),
        })
    }

    if err := validate.Struct(req); err != nil {
        return c.Status(400).JSON(fiber.Map{
            "success": false,
            "message": "Validation failed",
            "error": err.Error(),
        })
    }

    asset, err := h.assetService.CreateAsset(req)
    if err != nil {
        return c.Status(500).JSON(fiber.Map{
            "success": false,
            "message": "Failed to create asset",
            "error": err.Error(),
        })
    }

    return c.Status(201).JSON(fiber.Map{
        "success": true,
        "message": "Asset created successfully",
        "data": asset,
    })
}

// ❌ Incorrect: Business logic in handler
func (h *AssetHandler) CreateAsset(c *fiber.Ctx) error {
    var req CreateAssetRequest
    c.BodyParser(&req)
    
    // Business logic should be in service layer
    if req.Name == "" {
        return c.Status(400).JSON(fiber.Map{"error": "Name required"})
    }
    
    // Database operations should be in repository layer
    db.Create(&req)
    
    return c.JSON(req)
}
```

### 2. Service Layer (Business Logic)
**Purpose**: Implement business logic and orchestrate operations
**Responsibilities**:
- Implement business rules and validation
- Coordinate between multiple repositories
- Handle business-specific error cases
- Transform data between layers

```go
// ✅ Correct: Service layer with business logic
type AssetService interface {
    CreateAsset(req CreateAssetRequest) (*Asset, error)
    UpdateAsset(id string, req UpdateAssetRequest) (*Asset, error)
    DeleteAsset(id string) error
    GetAsset(id string) (*Asset, error)
    ListAssets(params ListAssetsParams) (*PaginatedResponse[Asset], error)
}

type assetService struct {
    assetRepo AssetRepository
    userRepo UserRepository
}

func (s *assetService) CreateAsset(req CreateAssetRequest) (*Asset, error) {
    // Business validation
    if req.Name == "" {
        return nil, errors.New("asset name is required")
    }
    
    // Check if user has permission
    user, err := s.userRepo.GetByID(req.CreatedBy)
    if err != nil {
        return nil, fmt.Errorf("failed to get user: %w", err)
    }
    
    if !user.CanCreateAssets() {
        return nil, errors.New("insufficient permissions")
    }
    
    // Check for duplicate names in organization
    exists, err := s.assetRepo.ExistsByName(req.Name, req.OrganizationID)
    if err != nil {
        return nil, fmt.Errorf("failed to check duplicate: %w", err)
    }
    
    if exists {
        return nil, errors.New("asset with this name already exists")
    }
    
    // Create asset
    asset := &Asset{
        ID:             generateID(),
        Name:           req.Name,
        ModelNumber:    req.ModelNumber,
        Manufacturer:   req.Manufacturer,
        Barcode:        req.Barcode,
        Status:         AssetStatusActive,
        PurchaseAmount: req.PurchaseAmount,
        CategoryID:     req.CategoryID,
        BranchID:       req.BranchID,
        OrganizationID: req.OrganizationID,
        CreatedBy:      req.CreatedBy,
        CreatedAt:      time.Now(),
    }
    
    return s.assetRepo.Create(asset)
}

// ❌ Incorrect: Service doing database operations directly
func (s *assetService) CreateAsset(req CreateAssetRequest) (*Asset, error) {
    // Should use repository, not direct database access
    db := database.GetDB()
    asset := &Asset{...}
    db.Create(asset)
    return asset, nil
}
```

### 3. Repository Layer (Data Access)
**Purpose**: Handle data persistence and retrieval
**Responsibilities**:
- Database operations (CRUD)
- Query optimization
- Data mapping between database and models
- Transaction management

```go
// ✅ Correct: Repository with proper data access
type AssetRepository interface {
    Create(asset *Asset) (*Asset, error)
    GetByID(id string) (*Asset, error)
    Update(asset *Asset) (*Asset, error)
    Delete(id string) error
    List(params ListParams) (*PaginatedResponse[Asset], error)
    ExistsByName(name, orgID string) (bool, error)
}

type assetRepository struct {
    db *gorm.DB
}

func (r *assetRepository) Create(asset *Asset) (*Asset, error) {
    if err := r.db.Create(asset).Error; err != nil {
        return nil, fmt.Errorf("failed to create asset: %w", err)
    }
    return asset, nil
}

func (r *assetRepository) GetByID(id string) (*Asset, error) {
    var asset Asset
    if err := r.db.Where("id = ?", id).First(&asset).Error; err != nil {
        if errors.Is(err, gorm.ErrRecordNotFound) {
            return nil, errors.New("asset not found")
        }
        return nil, fmt.Errorf("failed to get asset: %w", err)
    }
    return &asset, nil
}

func (r *assetRepository) List(params ListParams) (*PaginatedResponse[Asset], error) {
    var assets []Asset
    var total int64
    
    query := r.db.Model(&Asset{})
    
    // Apply filters
    if params.OrganizationID != "" {
        query = query.Where("organization_id = ?", params.OrganizationID)
    }
    if params.Status != "" {
        query = query.Where("status = ?", params.Status)
    }
    if params.Search != "" {
        query = query.Where("name ILIKE ?", "%"+params.Search+"%")
    }
    
    // Get total count
    if err := query.Count(&total).Error; err != nil {
        return nil, fmt.Errorf("failed to count assets: %w", err)
    }
    
    // Apply pagination
    offset := (params.Page - 1) * params.Limit
    if err := query.Offset(offset).Limit(params.Limit).Find(&assets).Error; err != nil {
        return nil, fmt.Errorf("failed to list assets: %w", err)
    }
    
    return &PaginatedResponse[Asset]{
        Data:       assets,
        Total:      total,
        Page:       params.Page,
        Limit:      params.Limit,
        TotalPages: int(math.Ceil(float64(total) / float64(params.Limit))),
    }, nil
}

// ❌ Incorrect: Repository doing business logic
func (r *assetRepository) Create(asset *Asset) (*Asset, error) {
    // Business logic should be in service layer
    if asset.Name == "" {
        return nil, errors.New("name required")
    }
    
    // Check permissions (should be in service)
    if !user.CanCreateAssets() {
        return nil, errors.New("no permission")
    }
    
    return r.db.Create(asset).Error
}
```

### 4. Model Layer (Data Structures)
**Purpose**: Define data structures and business entities
**Responsibilities**:
- Define data models
- Include validation rules
- Handle data transformation
- Define relationships

```go
// ✅ Correct: Clean model with proper validation
type Asset struct {
    ID             string      `db:"public.assets.id" json:"id" validate:"required,uuid"`
    Name           string      `db:"public.assets.name" json:"name" validate:"required,min=1,max=255"`
    ModelNumber   *string     `db:"public.assets.model_number" json:"model_number" validate:"omitempty,max=100"`
    Manufacturer   *string     `db:"public.assets.manufacturer" json:"manufacturer" validate:"omitempty,max=100"`
    Barcode        *string     `db:"public.assets.barcode" json:"barcode" validate:"omitempty,max=50"`
    Status         AssetStatus `db:"public.assets.status" json:"status" validate:"required,oneof=active inactive maintenance retired scrapped"`
    PurchaseAmount *float64    `db:"public.assets.purchase_amount" json:"purchase_amount" validate:"omitempty,min=0"`
    CategoryID     string      `db:"public.assets.category_id" json:"category_id" validate:"required,uuid"`
    BranchID       string      `db:"public.assets.branch_id" json:"branch_id" validate:"required,uuid"`
    OrganizationID string      `db:"public.assets.organization_id" json:"id" validate:"required,uuid"`
    CreatedBy      string      `db:"public.assets.created_by" json:"created_by" validate:"required,uuid"`
    CreatedAt      time.Time   `db:"public.assets.created_at" json:"created_at"`
    UpdatedAt      *time.Time  `db:"public.assets.updated_at" json:"updated_at"`
    UpdatedBy      *string     `db:"public.assets.updated_by" json:"updated_by" validate:"omitempty,uuid"`
}

type AssetStatus string

const (
    AssetStatusActive     AssetStatus = "active"
    AssetStatusInactive   AssetStatus = "inactive"
    AssetStatusMaintenance AssetStatus = "maintenance"
    AssetStatusRetired    AssetStatus = "retired"
    AssetStatusScrapped   AssetStatus = "scrapped"
)

func (s AssetStatus) IsValid() bool {
    switch s {
    case AssetStatusActive, AssetStatusInactive, AssetStatusMaintenance, AssetStatusRetired, AssetStatusScrapped:
        return true
    }
    return false
}

// ❌ Incorrect: Model with business logic
type Asset struct {
    ID   string `json:"id"`
    Name string `json:"name"`
}

func (a *Asset) Create() error {
    // Business logic should be in service layer
    if a.Name == "" {
        return errors.New("name required")
    }
    return database.Create(a)
}
```

### 5. DTO Layer (Data Transfer Objects)
**Purpose**: Define data structures for API communication
**Responsibilities**:
- Define request/response structures
- Handle data validation
- Transform between internal and external formats
- Version API contracts

```go
// ✅ Correct: Clean DTOs for API communication
type CreateAssetRequest struct {
    Name           string   `json:"name" validate:"required,min=1,max=255"`
    ModelNumber    *string  `json:"model_number" validate:"omitempty,max=100"`
    Manufacturer   *string  `json:"manufacturer" validate:"omitempty,max=100"`
    Barcode        *string  `json:"barcode" validate:"omitempty,max=50"`
    PurchaseAmount *float64 `json:"purchase_amount" validate:"omitempty,min=0"`
    CategoryID     string   `json:"category_id" validate:"required,uuid"`
    BranchID       string   `json:"branch_id" validate:"required,uuid"`
    OrganizationID string   `json:"organization_id" validate:"required,uuid"`
}

type UpdateAssetRequest struct {
    Name           *string  `json:"name" validate:"omitempty,min=1,max=255"`
    ModelNumber    *string  `json:"model_number" validate:"omitempty,max=100"`
    Manufacturer   *string  `json:"manufacturer" validate:"omitempty,max=100"`
    Barcode        *string  `json:"barcode" validate:"omitempty,max=50"`
    Status         *string  `json:"status" validate:"omitempty,oneof=active inactive maintenance retired scrapped"`
    PurchaseAmount *float64 `json:"purchase_amount" validate:"omitempty,min=0"`
    CategoryID     *string  `json:"category_id" validate:"omitempty,uuid"`
    BranchID       *string  `json:"branch_id" validate:"omitempty,uuid"`
}

type AssetResponse struct {
    Success bool   `json:"success"`
    Message string `json:"message"`
    Data    *Asset `json:"data,omitempty"`
    Error   *Error `json:"error,omitempty"`
}

type Error struct {
    Code    string `json:"code"`
    Details string `json:"details"`
}

// ❌ Incorrect: DTOs with business logic
type CreateAssetRequest struct {
    Name string `json:"name"`
}

func (r *CreateAssetRequest) Validate() error {
    // Validation should be handled by validation tags or service layer
    if r.Name == "" {
        return errors.New("name required")
    }
    return nil
}
```

## Interface Design Standards

### 1. Service Interface Design
```go
// ✅ Correct: Clean service interfaces
type AssetService interface {
    CreateAsset(req CreateAssetRequest) (*Asset, error)
    UpdateAsset(id string, req UpdateAssetRequest) (*Asset, error)
    DeleteAsset(id string) error
    GetAsset(id string) (*Asset, error)
    ListAssets(params ListAssetsParams) (*PaginatedResponse[Asset], error)
}

// ✅ Correct: Repository interfaces
type AssetRepository interface {
    Create(asset *Asset) (*Asset, error)
    GetByID(id string) (*Asset, error)
    Update(asset *Asset) (*Asset, error)
    Delete(id string) error
    List(params ListParams) (*PaginatedResponse[Asset], error)
}
```

### 2. Interface Segregation
```go
// ✅ Correct: Small, focused interfaces
type AssetReader interface {
    GetByID(id string) (*Asset, error)
    List(params ListParams) (*PaginatedResponse[Asset], error)
}

type AssetWriter interface {
    Create(asset *Asset) (*Asset, error)
    Update(asset *Asset) (*Asset, error)
    Delete(id string) error
}

type AssetRepository interface {
    AssetReader
    AssetWriter
}
```

## Dependency Injection Patterns

### 1. Constructor Injection
```go
// ✅ Correct: Constructor injection
type AssetHandler struct {
    assetService AssetService
}

func NewAssetHandler(assetService AssetService) *AssetHandler {
    return &AssetHandler{
        assetService: assetService,
    }
}

// ✅ Correct: Service constructor
type assetService struct {
    assetRepo AssetRepository
    userRepo  UserRepository
}

func NewAssetService(assetRepo AssetRepository, userRepo UserRepository) AssetService {
    return &assetService{
        assetRepo: assetRepo,
        userRepo:  userRepo,
    }
}
```

### 2. Interface Segregation
```go
// ✅ Correct: Small, focused interfaces
type AssetReader interface {
    GetByID(id string) (*Asset, error)
    List(params ListParams) (*PaginatedResponse[Asset], error)
}

type AssetWriter interface {
    Create(asset *Asset) (*Asset, error)
    Update(asset *Asset) (*Asset, error)
    Delete(id string) error
}

type AssetRepository interface {
    AssetReader
    AssetWriter
}
```

## Error Handling Standards

### 1. Layer-Specific Error Handling
```go
// ✅ Correct: Handler error handling
func (h *AssetHandler) CreateAsset(c *fiber.Ctx) error {
    var req CreateAssetRequest
    if err := c.BodyParser(&req); err != nil {
        return c.Status(400).JSON(ErrorResponse{
            Success: false,
            Message: "Invalid request body",
            Error:   &Error{Code: "INVALID_BODY", Details: err.Error()},
        })
    }

    asset, err := h.assetService.CreateAsset(req)
    if err != nil {
        return c.Status(500).JSON(ErrorResponse{
            Success: false,
            Message: "Failed to create asset",
            Error:   &Error{Code: "CREATE_FAILED", Details: err.Error()},
        })
    }

    return c.Status(201).JSON(SuccessResponse{
        Success: true,
        Message: "Asset created successfully",
        Data:    asset,
    })
}

// ✅ Correct: Service error handling
func (s *assetService) CreateAsset(req CreateAssetRequest) (*Asset, error) {
    if req.Name == "" {
        return nil, errors.New("asset name is required")
    }
    
    user, err := s.userRepo.GetByID(req.CreatedBy)
    if err != nil {
        return nil, fmt.Errorf("failed to get user: %w", err)
    }
    
    if !user.CanCreateAssets() {
        return nil, errors.New("insufficient permissions")
    }
    
    return s.assetRepo.Create(&Asset{...})
}
```

## Testing Standards

### 1. Layer-Specific Testing
```go
// ✅ Correct: Handler testing
func TestAssetHandler_CreateAsset(t *testing.T) {
    mockService := &MockAssetService{}
    handler := NewAssetHandler(mockService)
    
    app := fiber.New()
    app.Post("/assets", handler.CreateAsset)
    
    req := CreateAssetRequest{
        Name: "Test Asset",
        // ... other fields
    }
    
    body, _ := json.Marshal(req)
    req := httptest.NewRequest("POST", "/assets", bytes.NewReader(body))
    req.Header.Set("Content-Type", "application/json")
    
    resp, err := app.Test(req)
    assert.NoError(t, err)
    assert.Equal(t, 201, resp.StatusCode)
}

// ✅ Correct: Service testing
func TestAssetService_CreateAsset(t *testing.T) {
    mockRepo := &MockAssetRepository{}
    mockUserRepo := &MockUserRepository{}
    service := NewAssetService(mockRepo, mockUserRepo)
    
    req := CreateAssetRequest{
        Name: "Test Asset",
        // ... other fields
    }
    
    mockUserRepo.On("GetByID", req.CreatedBy).Return(&User{CanCreateAssets: true}, nil)
    mockRepo.On("Create", mock.Anything).Return(&Asset{ID: "123"}, nil)
    
    asset, err := service.CreateAsset(req)
    assert.NoError(t, err)
    assert.NotNil(t, asset)
    assert.Equal(t, "123", asset.ID)
}
```

## Best Practices Summary

### 1. Layer Responsibilities
- **Handler**: HTTP concerns only
- **Service**: Business logic and orchestration
- **Repository**: Data access only
- **Model**: Data structure and validation
- **DTO**: API contract definition

### 2. Dependency Flow
```
Handler → Service → Repository → Database
   ↓        ↓         ↓
  DTO    Models    Models
```

### 3. Error Handling
- Handle errors at appropriate layers
- Use proper error wrapping
- Return meaningful error messages
- Log errors appropriately

### 4. Testing
- Test each layer independently
- Use mocks for dependencies
- Test error scenarios
- Maintain high test coverage

This MVC architecture ensures clean separation of concerns, maintainability, and scalability for your Go backend project.
