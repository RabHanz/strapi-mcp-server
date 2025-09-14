# Strapi MCP Tool - Best Practices Guide

## Table of Contents
1. [Tool Overview](#tool-overview)
2. [Authentication & Authorization](#authentication--authorization)
3. [Content Type Operations](#content-type-operations)
4. [Component Management](#component-management)
5. [Data Operations](#data-operations)
6. [Best Practices](#best-practices)
7. [Common Patterns](#common-patterns)
8. [Error Handling](#error-handling)
9. [Troubleshooting](#troubleshooting)

## Tool Overview

The Strapi MCP tool provides programmatic access to Strapi CMS instances. It supports both reading and writing operations across content types, components, and media management.

### Available Functions
- `strapi_list_servers` - List configured Strapi servers
- `strapi_get_content_types` - Get all content types and their schemas
- `strapi_get_components` - Get all components with pagination
- `strapi_rest` - Execute REST API requests
- `strapi_upload_media` - Upload media files

## Authentication & Authorization

### Critical Security Pattern
**ALL WRITE OPERATIONS** (POST, PUT, DELETE) require explicit user authorization:

```json
{
  "userAuthorized": true
}
```

### Server Selection
Always specify the server parameter:

```json
{
  "server": "myserver"
}
```

### Write Operation Template
```json
{
  "endpoint": "api/content-type",
  "method": "POST|PUT|DELETE",
  "body": { "data": {...} },
  "server": "myserver",
  "userAuthorized": true
}
```

## Content Type Operations

### 1. Discover Content Types
**Always start by discovering the available content types:**

```json
{
  "server": "myserver"
}
```

This returns the complete schema including:
- Field types and constraints
- Relations and their targets
- Component usage
- i18n configuration
- Validation rules

### 2. Understanding Content Type Kinds

**Collection Types** (multiple records):
- Use `api/{pluralName}` endpoints
- Support standard CRUD operations
- Use POST to create new records

**Single Types** (single record):
- Use `api/{singularName}` endpoints  
- Use PUT to create/update (no POST)
- No ID required in requests

### 3. API Endpoint Patterns

**Collection Types:**
```
GET    /api/{pluralName}          # List records
GET    /api/{pluralName}/{id}     # Get single record
POST   /api/{pluralName}          # Create record
PUT    /api/{pluralName}/{id}     # Update record
DELETE /api/{pluralName}/{id}     # Delete record
```

**Single Types:**
```
GET /api/{singularName}           # Get the single record
PUT /api/{singularName}           # Create/update the single record
```

## Component Management

### 1. Component Structure Discovery
```json
{
  "server": "myserver",
  "page": 1,
  "pageSize": 25
}
```

### 2. Component Data Format
Components are embedded objects within content types:

```json
{
  "data": {
    "componentName": {
      "field1": "value1",
      "field2": "value2"
    }
  }
}
```

**For repeatable components:**
```json
{
  "data": {
    "componentName": [
      { "field1": "value1" },
      { "field1": "value2" }
    ]
  }
}
```

### 3. Component Population
To read components, use populate parameters:

```json
{
  "params": {
    "populate": ["componentName"]
  }
}
```

**Advanced population:**
```json
{
  "params": {
    "populate": {
      "componentName": {
        "fields": ["field1", "field2"]
      }
    }
  }
}
```

## Data Operations

### 1. Reading Data

**Basic Read:**
```json
{
  "endpoint": "api/content-type",
  "method": "GET",
  "server": "myserver"
}
```

**With Population:**
```json
{
  "endpoint": "api/content-type",
  "method": "GET",
  "params": {
    "populate": "*"
  },
  "server": "myserver"
}
```

**With Filtering:**
```json
{
  "endpoint": "api/content-type",
  "method": "GET",
  "params": {
    "filters": {
      "field": "value"
    }
  },
  "server": "myserver"
}
```

### 2. Creating Data

**Collection Type Creation:**
```json
{
  "endpoint": "api/collection-types",
  "method": "POST",
  "body": {
    "data": {
      "field1": "value1",
      "field2": "value2",
      "componentField": {
        "componentProp": "value"
      }
    }
  },
  "server": "myserver",
  "userAuthorized": true
}
```

**Single Type Creation/Update:**
```json
{
  "endpoint": "api/single-type",
  "method": "PUT",
  "body": {
    "data": {
      "field1": "value1",
      "componentField": {
        "componentProp": "value"
      }
    }
  },
  "server": "myserver",
  "userAuthorized": true
}
```

### 3. Relations Handling

**Setting Relations (use document IDs):**
```json
{
  "data": {
    "relationField": "documentId123",
    "multipleRelations": ["docId1", "docId2", "docId3"]
  }
}
```

**One-to-Many Relations:**
```json
{
  "data": {
    "category": "categoryDocumentId"
  }
}
```

## Best Practices

### 1. Always Follow This Sequence

1. **Discover Server and Content Types**
2. **Examine Schema Requirements**
3. **Plan Data Structure**
4. **Execute Operations with Proper Authorization**
5. **Verify Results**

### 2. Schema-First Approach

**Before creating content:**
- Check required fields
- Understand field constraints (maxLength, enum values)
- Verify relation targets
- Check component structure

### 3. Error Prevention

**Common Issues to Avoid:**
- Missing required fields
- Wrong enum values
- Invalid relation targets
- Forgetting userAuthorized for write operations
- Using wrong HTTP methods for single types

### 4. Data Validation

**Always validate:**
- Enum values match exactly
- Required fields are present
- String lengths don't exceed maxLength
- Number ranges are within min/max bounds
- Relations reference valid document IDs

### 5. Efficient Operations

**Batch Operations:**
```json
{
  "endpoint": "api/content-type",
  "method": "POST",
  "body": {
    "data": [
      { "field": "value1" },
      { "field": "value2" }
    ]
  },
  "server": "myserver",
  "userAuthorized": true
}
```

## Common Patterns

### 1. Content Audit Pattern
```javascript
// 1. List all content types
// 2. Check each content type for existing data
// 3. Identify missing content
// 4. Create content systematically
```

### 2. Bulk Content Creation
```javascript
// 1. Prepare all data structures
// 2. Create base content (neighborhoods, categories)
// 3. Create relational content (posts, items)
// 4. Create complex content (pages with components)
```

### 3. Component-Heavy Content
```javascript
// 1. Create simple components first
// 2. Build complex nested structures
// 3. Test component rendering
// 4. Optimize for performance
```

### 4. Multi-language Content
```javascript
// 1. Create content in default language
// 2. Use locale parameter for translations
// 3. Manage localized vs non-localized fields
```

## Error Handling

### 1. Common HTTP Status Codes

- **400 Bad Request**: Invalid data structure or missing required fields
- **404 Not Found**: Wrong endpoint or content doesn't exist
- **405 Method Not Allowed**: Wrong HTTP method (e.g., POST to single type)
- **422 Unprocessable Entity**: Validation errors

### 2. Authorization Errors
```
userAuthorized: Write operations (POST, PUT, DELETE) require explicit user authorization
```
**Solution:** Add `"userAuthorized": true` to request.

### 3. Validation Errors
**Check for:**
- Required field violations
- Enum value mismatches
- String length violations
- Invalid relation references

### 4. Endpoint Errors
**Verify:**
- Correct pluralName/singularName usage
- Proper HTTP method selection
- Valid document IDs for updates/deletes

## Troubleshooting

### 1. 404 Errors on Single Types
**Problem:** Using POST instead of PUT
**Solution:** Single types use PUT for create/update operations

### 2. Component Not Saving
**Problem:** Wrong component structure
**Solution:** Check component schema and match exact field names

### 3. Relations Not Working
**Problem:** Using ID instead of documentId
**Solution:** Use documentId from the relation target

### 4. Enum Validation Failures
**Problem:** Case sensitivity or extra spaces
**Solution:** Use exact enum values from schema

### 5. Required Field Errors
**Problem:** Missing required fields in request
**Solution:** Include all fields marked as required in schema

## Advanced Patterns

### 1. Complex Filtering
```json
{
  "params": {
    "filters": {
      "$and": [
        { "field1": "value1" },
        { "field2": { "$ne": "value2" } }
      ]
    }
  }
}
```

### 2. Sorting and Pagination
```json
{
  "params": {
    "sort": ["createdAt:desc"],
    "pagination": {
      "page": 1,
      "pageSize": 25
    }
  }
}
```

### 3. Media Upload Integration
```json
{
  "url": "https://example.com/image.jpg",
  "metadata": {
    "name": "Professional Image",
    "alternativeText": "Alt text for accessibility",
    "caption": "Image caption"
  },
  "format": "webp",
  "quality": 80,
  "server": "myserver",
  "userAuthorized": true
}
```

## Success Indicators

### 1. Proper Implementation
✅ All write operations include userAuthorized
✅ Correct HTTP methods for content type kinds
✅ Schema validation passes
✅ Relations reference valid documents
✅ Components structure matches schema

### 2. Content Quality
✅ All required fields populated
✅ Realistic, professional content
✅ Proper categorization and tagging
✅ SEO-optimized metadata
✅ Consistent brand voice

### 3. System Health
✅ No validation errors
✅ Fast response times
✅ Proper error handling
✅ Clean data relationships
✅ Scalable content structure

## Final Checklist

Before considering a Strapi implementation complete:

- [ ] All content types properly configured
- [ ] All required content created and published
- [ ] Components working correctly
- [ ] Relations properly established
- [ ] Media assets uploaded and linked
- [ ] User roles and permissions set
- [ ] No validation errors in system
- [ ] Content follows brand guidelines
- [ ] SEO elements properly configured
- [ ] Mobile-responsive content structure

This guide ensures consistent, error-free operations with the Strapi MCP tool while following industry best practices for content management and API usage.
