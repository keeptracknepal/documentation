# Media System Summary

## Overview

The KeepTrack Media System is a file upload and storage service built with Go Fiber and MinIO. It provides a RESTful API for uploading, storing, and serving media files (images and videos) with organization-based isolation.

## Architecture

### Components
- **Media Server**: Go Fiber web server running on port 8301
- **MinIO**: Object storage backend for file persistence
- **Domain**: `media.keeptrack.velonovo.com`
- **Storage**: Organization-based bucket isolation

### Technology Stack
- **Backend**: Go with Fiber web framework
- **Storage**: MinIO object storage
- **File Handling**: Multipart form data processing
- **Validation**: File type, size, and format validation

## API Endpoints

### Upload Endpoint
```
POST /upload
Content-Type: multipart/form-data
```

**Required Form Fields:**
- `file`: The media file to upload
- `type`: Media type (`image` or `video`)
- `bucket`: Organization ID (used as bucket name)
- `filename`: Filename provided by frontend

### Serving Endpoints
```
GET /image/{bucket}/{filename}  # Serve image files
GET /video/{bucket}/{filename}  # Serve video files
GET /health                     # Health check
```

## File Organization

### Bucket Structure
```
{organization_id}/
├── images/
│   └── {filename}
└── videos/
    └── {filename}
```

### File Naming
- **Frontend Responsibility**: Frontend generates unique filenames
- **Format**: `{frontend_provided_filename}`
- **Example**: `user-avatar-123.jpg`, `document-456.pdf`

## File Validation

### Image Files
- **Allowed Types**: JPEG, PNG, GIF, WebP
- **Max Size**: 10MB
- **Content Types**: `image/jpeg`, `image/png`, `image/gif`, `image/webp`

### Video Files
- **Allowed Types**: MP4, AVI, MOV, WMV, WebM
- **Max Size**: 100MB
- **Content Types**: `video/mp4`, `video/avi`, `video/quicktime`, `video/x-ms-wmv`, `video/webm`

## Upload Process

1. **File Validation**: Check file type, size, and format
2. **Temporary Storage**: Save uploaded file to temporary location
3. **Bucket Creation**: Create bucket if it doesn't exist
4. **MinIO Upload**: Upload file to MinIO with proper content type
5. **URL Generation**: Generate public URL for the uploaded file
6. **Cleanup**: Remove temporary file

## Response Format

### Success Response (200)
```json
{
  "success": true,
  "url": "https://media.keeptrack.velonovo.com/image/org_12345/photo.jpg",
  "filename": "photo.jpg",
  "originalName": "photo.jpg",
  "size": 1024000,
  "type": "image/jpeg",
  "mediaType": "image"
}
```

### Error Responses (400/500)
```json
{
  "success": false,
  "error": "Error description"
}
```

## Security Features

### File Validation
- **Type Validation**: Strict file type checking
- **Size Limits**: Prevents abuse through large uploads
- **Format Validation**: Ensures file integrity

### Organization Isolation
- **Bucket Separation**: Each organization's files are isolated
- **Access Control**: Public read access with organization-based structure
- **Frontend Naming**: Frontend controls filename generation and uniqueness

## Storage Features

### Automatic Bucket Management
- **Auto-Creation**: Buckets created automatically if they don't exist
- **Policy Setting**: Public read access configured automatically
- **Subdirectory Creation**: `images/` and `videos/` directories created as needed

### Content Delivery
- **Caching Headers**: `Cache-Control: public, max-age=31536000, immutable`
- **Content-Type Detection**: Automatic MIME type detection
- **Direct Serving**: Files served directly from MinIO

## Usage Examples

### JavaScript/Fetch
```javascript
const formData = new FormData();
formData.append('file', fileInput.files[0]);
formData.append('type', 'image');
formData.append('bucket', 'org_12345');
formData.append('filename', 'user-avatar-123.jpg');

const response = await fetch('https://media.keeptrack.velonovo.com/upload', {
  method: 'POST',
  body: formData
});

const result = await response.json();
console.log('Upload URL:', result.url);
```

### cURL
```bash
curl -X POST https://media.keeptrack.velonovo.com/upload \
  -F "file=@/path/to/image.jpg" \
  -F "type=image" \
  -F "bucket=org_12345" \
  -F "filename=user-avatar-123.jpg"
```

## Error Handling

### Common Error Scenarios
- **"No file provided"**: Missing file in form data
- **"Invalid media type"**: Use `image` or `video` for type field
- **"File too large"**: Exceeds size limits (10MB images, 100MB videos)
- **"Invalid file format"**: Unsupported file type
- **"Bucket name is required"**: Missing organization ID
- **"Filename is required"**: Missing filename

### Debug Tips
- Check file size before upload
- Verify file type is supported
- Ensure proper form data encoding
- Check network connectivity to MinIO

## Configuration

### Environment Variables
- **MinIO Endpoint**: Configured for MinIO connection
- **Access Keys**: MinIO authentication credentials
- **SSL Settings**: HTTPS configuration for MinIO
- **Region**: MinIO region configuration

### Server Configuration
- **Port**: 8301
- **Domain**: media.keeptrack.velonovo.com
- **Static Files**: Served from `./public` directory
- **Health Check**: Available at `/health`

## Integration Points

### Frontend Integration
- **File Upload**: Use multipart form data
- **Progress Tracking**: Implement client-side progress indicators
- **Error Handling**: Handle various error scenarios
- **URL Generation**: Use returned URLs for display

### Backend Integration
- **Database Storage**: Store file URLs in application database
- **User Management**: Link files to users/organizations
- **Access Control**: Implement application-level permissions

## Performance Considerations

### Caching
- **Browser Caching**: Long-term caching headers for uploaded files
- **CDN Ready**: Structure supports CDN integration
- **Immutable URLs**: Files can be cached indefinitely

### Scalability
- **MinIO Scaling**: MinIO supports horizontal scaling
- **Load Balancing**: Multiple media server instances possible
- **Storage Distribution**: Files distributed across MinIO nodes

## Monitoring and Maintenance

### Health Checks
- **Server Health**: `/health` endpoint for monitoring
- **MinIO Connectivity**: Automatic connection validation
- **File Operations**: Error logging for troubleshooting

### Maintenance Tasks
- **Bucket Cleanup**: Remove unused organization buckets
- **File Cleanup**: Remove orphaned files
- **Storage Monitoring**: Monitor MinIO storage usage
- **Performance Monitoring**: Track upload/download metrics
