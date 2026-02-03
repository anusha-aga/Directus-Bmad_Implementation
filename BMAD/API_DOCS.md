# API Documentation

## Overview

Directus provides a comprehensive REST API that dynamically mirrors your database schema. All endpoints require
authentication unless explicitly stated otherwise. The API follows RESTful conventions and returns JSON responses.

**Base URL**: `http://your-directus-instance/`

**Authentication**: Most endpoints require a valid JWT token in the `Authorization` header:

```
Authorization: Bearer <access_token>
```

## Common Response Format

```json
{
  "data": { ... },
  "meta": {
    "filter_count": 10,
    "total_count": 100
  }
}
```

## Common Error Responses

| Status Code | Error                   | Description                                          |
| ----------- | ----------------------- | ---------------------------------------------------- |
| `400`       | `INVALID_PAYLOAD`       | Request body is malformed or missing required fields |
| `401`       | `INVALID_CREDENTIALS`   | Authentication token is missing or invalid           |
| `403`       | `FORBIDDEN`             | User lacks permission to perform the action          |
| `404`       | `NOT_FOUND`             | Resource does not exist                              |
| `422`       | `INVALID_QUERY`         | Query parameters are invalid                         |
| `429`       | `RATE_LIMIT_EXCEEDED`   | Too many requests                                    |
| `500`       | `INTERNAL_SERVER_ERROR` | Server error                                         |

---

## Authentication Endpoints

### POST `/auth/login`

**Purpose**: Authenticate user and obtain access/refresh tokens  
**Auth Required**: No  
**Request Body**:

```json
{
	"email": "user@example.com",
	"password": "password123",
	"mode": "json" // or "cookie" or "session"
}
```

**Response**:

```json
{
	"data": {
		"access_token": "eyJhbGc...",
		"refresh_token": "abc123...",
		"expires": 900000
	}
}
```

### POST `/auth/login/:provider`

**Purpose**: Authenticate via SSO provider (OAuth2, OpenID, LDAP, SAML)  
**Auth Required**: No  
**Providers**: Configured via environment variables

### POST `/auth/refresh`

**Purpose**: Refresh access token using refresh token  
**Auth Required**: No (requires refresh token)  
**Request Body**:

```json
{
	"refresh_token": "abc123...",
	"mode": "json"
}
```

### POST `/auth/logout`

**Purpose**: Invalidate refresh token and logout  
**Auth Required**: No (requires refresh token)  
**Request Body**:

```json
{
	"refresh_token": "abc123..."
}
```

### POST `/auth/password/request`

**Purpose**: Request password reset email  
**Auth Required**: No  
**Request Body**:

```json
{
	"email": "user@example.com",
	"reset_url": "https://example.com/reset" // optional
}
```

### POST `/auth/password/reset`

**Purpose**: Reset password using token from email  
**Auth Required**: No  
**Request Body**:

```json
{
	"token": "reset-token-from-email",
	"password": "newpassword123"
}
```

### GET `/auth`

**Purpose**: List available authentication providers  
**Auth Required**: No  
**Response**:

```json
{
	"data": [
		{ "name": "default", "driver": "local" },
		{ "name": "google", "driver": "oauth2" }
	],
	"disableDefault": false
}
```

---

## Items Endpoints (Dynamic Collections)

All custom collections are accessible via `/items/:collection`. These endpoints work with any collection in your
database.

### POST `/items/:collection`

**Purpose**: Create new item(s) in a collection  
**Auth Required**: Yes  
**Request Body** (single):

```json
{
	"title": "My Article",
	"status": "published"
}
```

**Request Body** (multiple):

```json
[{ "title": "Article 1" }, { "title": "Article 2" }]
```

**Response**:

```json
{
	"data": {
		"id": 1,
		"title": "My Article",
		"status": "published"
	}
}
```

**Common Errors**:

- `403 FORBIDDEN`: User lacks create permission
- `400 INVALID_PAYLOAD`: Missing required fields

### GET `/items/:collection`

**Purpose**: List items in a collection  
**Auth Required**: Yes  
**Query Parameters**:

- `fields`: Comma-separated list of fields to return
- `filter`: JSON filter object
- `sort`: Sort field(s), prefix with `-` for descending
- `limit`: Number of items to return
- `offset`: Number of items to skip
- `meta`: Include metadata (total count, etc.)

**Example**: `GET /items/articles?fields=id,title&limit=10&sort=-date_created`

**Response**:

```json
{
	"data": [
		{ "id": 1, "title": "Article 1" },
		{ "id": 2, "title": "Article 2" }
	],
	"meta": {
		"filter_count": 2,
		"total_count": 100
	}
}
```

### SEARCH `/items/:collection`

**Purpose**: Search items using POST body (for complex queries)  
**Auth Required**: Yes  
**Request Body**:

```json
{
	"query": {
		"filter": { "status": { "_eq": "published" } },
		"sort": ["-date_created"],
		"limit": 10
	}
}
```

### GET `/items/:collection/:id`

**Purpose**: Get a single item by ID  
**Auth Required**: Yes  
**Response**:

```json
{
	"data": {
		"id": 1,
		"title": "My Article",
		"status": "published"
	}
}
```

### PATCH `/items/:collection`

**Purpose**: Update multiple items  
**Auth Required**: Yes  
**Request Body** (by keys):

```json
{
	"keys": [1, 2, 3],
	"data": { "status": "archived" }
}
```

**Request Body** (by query):

```json
{
	"query": { "filter": { "status": { "_eq": "draft" } } },
	"data": { "status": "published" }
}
```

### PATCH `/items/:collection/:id`

**Purpose**: Update a single item  
**Auth Required**: Yes  
**Request Body**:

```json
{
	"title": "Updated Title",
	"status": "published"
}
```

### DELETE `/items/:collection`

**Purpose**: Delete multiple items  
**Auth Required**: Yes  
**Request Body**:

```json
{
	"keys": [1, 2, 3]
}
```

### DELETE `/items/:collection/:id`

**Purpose**: Delete a single item  
**Auth Required**: Yes

---

## Users Endpoints

### POST `/users`

**Purpose**: Create new user(s)  
**Auth Required**: Yes (admin)  
**Request Body**:

```json
{
	"email": "newuser@example.com",
	"password": "password123",
	"role": "role-uuid",
	"first_name": "John",
	"last_name": "Doe"
}
```

### GET `/users`

**Purpose**: List all users  
**Auth Required**: Yes  
**Query Parameters**: Same as items endpoints

### GET `/users/me`

**Purpose**: Get current authenticated user  
**Auth Required**: Yes  
**Response**:

```json
{
	"data": {
		"id": "user-uuid",
		"email": "user@example.com",
		"first_name": "John",
		"role": "role-uuid"
	}
}
```

### GET `/users/:id`

**Purpose**: Get a specific user by ID  
**Auth Required**: Yes

### PATCH `/users/me`

**Purpose**: Update current user's profile  
**Auth Required**: Yes  
**Request Body**:

```json
{
	"first_name": "Jane",
	"avatar": "file-uuid"
}
```

### PATCH `/users/me/track/page`

**Purpose**: Track last visited page (for UI state)  
**Auth Required**: Yes  
**Request Body**:

```json
{
	"last_page": "/content/articles"
}
```

### PATCH `/users/:id`

**Purpose**: Update a specific user  
**Auth Required**: Yes (admin or self)

### DELETE `/users/:id`

**Purpose**: Delete a user  
**Auth Required**: Yes (admin)

### POST `/users/invite`

**Purpose**: Invite a new user via email  
**Auth Required**: Yes (admin)  
**Request Body**:

```json
{
	"email": "newuser@example.com",
	"role": "role-uuid",
	"invite_url": "https://example.com/accept-invite"
}
```

### POST `/users/invite/accept`

**Purpose**: Accept user invitation  
**Auth Required**: No  
**Request Body**:

```json
{
	"token": "invite-token",
	"password": "newpassword123"
}
```

### POST `/users/register`

**Purpose**: Self-registration (if enabled)  
**Auth Required**: No  
**Rate Limited**: Yes  
**Request Body**:

```json
{
	"email": "user@example.com",
	"password": "password123",
	"first_name": "John",
	"verification_url": "https://example.com/verify"
}
```

### GET `/users/register/verify-email`

**Purpose**: Verify email after registration  
**Auth Required**: No  
**Query Parameters**: `?token=verification-token`

### POST `/users/me/tfa/generate`

**Purpose**: Generate 2FA secret for current user  
**Auth Required**: Yes  
**Request Body**:

```json
{
	"password": "current-password" // required for local auth
}
```

**Response**:

```json
{
	"data": {
		"secret": "JBSWY3DPEHPK3PXP",
		"otpauth_url": "otpauth://totp/..."
	}
}
```

### POST `/users/me/tfa/enable`

**Purpose**: Enable 2FA for current user  
**Auth Required**: Yes  
**Request Body**:

```json
{
	"secret": "JBSWY3DPEHPK3PXP",
	"otp": "123456"
}
```

### POST `/users/me/tfa/disable`

**Purpose**: Disable 2FA for current user  
**Auth Required**: Yes  
**Request Body**:

```json
{
	"otp": "123456"
}
```

### POST `/users/:id/tfa/disable`

**Purpose**: Admin disable 2FA for a user  
**Auth Required**: Yes (admin only)

---

## Files Endpoints

### POST `/files`

**Purpose**: Upload file(s)  
**Auth Required**: Yes  
**Content-Type**: `multipart/form-data` or `application/json`  
**Multipart Upload**:

```
POST /files
Content-Type: multipart/form-data

--boundary
Content-Disposition: form-data; name="title"

My Image
--boundary
Content-Disposition: form-data; name="file"; filename="image.jpg"
Content-Type: image/jpeg

<binary data>
--boundary--
```

**JSON Upload** (metadata only):

```json
{
	"title": "My File",
	"storage": "local",
	"filename_download": "document.pdf"
}
```

### POST `/files/import`

**Purpose**: Import file from URL  
**Auth Required**: Yes  
**Request Body**:

```json
{
	"url": "https://example.com/image.jpg",
	"data": {
		"title": "Imported Image",
		"folder": "folder-uuid"
	}
}
```

### GET `/files`

**Purpose**: List files  
**Auth Required**: Yes  
**Query Parameters**: Same as items endpoints

### GET `/files/:id`

**Purpose**: Get file metadata  
**Auth Required**: Yes  
**Response**:

```json
{
	"data": {
		"id": "file-uuid",
		"title": "My Image",
		"filename_disk": "abc123.jpg",
		"filename_download": "image.jpg",
		"type": "image/jpeg",
		"filesize": 102400,
		"width": 1920,
		"height": 1080
	}
}
```

### PATCH `/files/:id`

**Purpose**: Update file metadata or replace file  
**Auth Required**: Yes  
**Content-Type**: `multipart/form-data` or `application/json`

### DELETE `/files/:id`

**Purpose**: Delete a file  
**Auth Required**: Yes

### POST `/files/tus` (if TUS enabled)

**Purpose**: Resumable file upload using TUS protocol  
**Auth Required**: Yes  
**Note**: Requires `TUS_ENABLED=true` environment variable

---

## Collections Endpoints

### POST `/collections`

**Purpose**: Create new collection(s)  
**Auth Required**: Yes (admin)  
**Request Body**:

```json
{
	"collection": "articles",
	"meta": {
		"icon": "article",
		"note": "Blog articles"
	},
	"schema": {
		"name": "articles"
	},
	"fields": [
		{
			"field": "id",
			"type": "integer",
			"schema": { "is_primary_key": true }
		},
		{
			"field": "title",
			"type": "string"
		}
	]
}
```

### GET `/collections`

**Purpose**: List all collections  
**Auth Required**: Yes  
**Response**:

```json
{
	"data": [
		{
			"collection": "articles",
			"meta": { "icon": "article" },
			"schema": { "name": "articles" }
		}
	]
}
```

### GET `/collections/:collection`

**Purpose**: Get collection metadata  
**Auth Required**: Yes

### PATCH `/collections/:collection`

**Purpose**: Update collection metadata  
**Auth Required**: Yes (admin)  
**Request Body**:

```json
{
	"meta": {
		"icon": "article",
		"note": "Updated note"
	}
}
```

### DELETE `/collections/:collection`

**Purpose**: Delete a collection  
**Auth Required**: Yes (admin)  
**Warning**: This deletes the database table and all data

---

## Additional Endpoints

### GraphQL

#### POST `/graphql`

**Purpose**: Execute GraphQL queries  
**Auth Required**: Yes  
**Request Body**:

```json
{
	"query": "query { articles { id title } }"
}
```

### Assets

#### GET `/assets/:id`

**Purpose**: Get transformed/resized image  
**Auth Required**: Conditional (based on permissions)  
**Query Parameters**:

- `width`: Target width
- `height`: Target height
- `fit`: Fit mode (cover, contain, inside, outside)
- `quality`: JPEG/WebP quality (1-100)
- `format`: Output format (jpg, png, webp, tiff)

**Example**: `GET /assets/file-uuid?width=400&height=300&fit=cover&quality=80`

### Server Info

#### GET `/server/info`

**Purpose**: Get server information  
**Auth Required**: No  
**Response**:

```json
{
	"data": {
		"project": {
			"project_name": "My Project",
			"project_logo": "logo-uuid"
		}
	}
}
```

#### GET `/server/ping`

**Purpose**: Health check endpoint  
**Auth Required**: No  
**Response**: `"pong"`

#### GET `/server/health`

**Purpose**: Detailed health check  
**Auth Required**: No  
**Response**:

```json
{
	"status": "ok",
	"releaseId": "v10.0.0",
	"serviceId": "abc123",
	"checks": {
		"database": "ok",
		"cache": "ok",
		"storage": "ok"
	}
}
```

### Schema

#### GET `/schema/snapshot`

**Purpose**: Get complete database schema snapshot  
**Auth Required**: Yes (admin)

#### POST `/schema/apply`

**Purpose**: Apply schema diff/migration  
**Auth Required**: Yes (admin)

#### GET `/schema/diff`

**Purpose**: Get schema differences  
**Auth Required**: Yes (admin)

### Permissions

#### GET `/permissions`

**Purpose**: List all permissions  
**Auth Required**: Yes (admin)

#### POST `/permissions`

**Purpose**: Create new permission rule  
**Auth Required**: Yes (admin)

#### PATCH `/permissions/:id`

**Purpose**: Update permission rule  
**Auth Required**: Yes (admin)

#### DELETE `/permissions/:id`

**Purpose**: Delete permission rule  
**Auth Required**: Yes (admin)

### Roles

#### GET `/roles`

**Purpose**: List all roles  
**Auth Required**: Yes

#### POST `/roles`

**Purpose**: Create new role  
**Auth Required**: Yes (admin)

#### PATCH `/roles/:id`

**Purpose**: Update role  
**Auth Required**: Yes (admin)

#### DELETE `/roles/:id`

**Purpose**: Delete role  
**Auth Required**: Yes (admin)

### Activity

#### GET `/activity`

**Purpose**: Get activity log (audit trail)  
**Auth Required**: Yes  
**Query Parameters**: Same as items endpoints

### Revisions

#### GET `/revisions`

**Purpose**: Get item revisions  
**Auth Required**: Yes  
**Query Parameters**: `?filter[item]=:id&filter[collection]=:collection`

#### GET `/revisions/:id`

**Purpose**: Get specific revision  
**Auth Required**: Yes

### Metrics (if enabled)

#### GET `/metrics`

**Purpose**: Prometheus metrics endpoint  
**Auth Required**: No  
**Note**: Requires `METRICS_ENABLED=true`

### AI Chat (if enabled)

#### POST `/ai/chat`

**Purpose**: AI assistant chat endpoint  
**Auth Required**: Yes  
**Note**: Requires `AI_ENABLED=true`

### MCP (if enabled)

#### POST `/mcp`

**Purpose**: Model Context Protocol endpoint  
**Auth Required**: Yes  
**Note**: Requires `MCP_ENABLED=true`

---

## Query Parameters

All `GET` endpoints support the following query parameters:

| Parameter | Description                         | Example                                          |
| --------- | ----------------------------------- | ------------------------------------------------ |
| `fields`  | Comma-separated list of fields      | `?fields=id,title,author.name`                   |
| `filter`  | JSON filter object                  | `?filter[status][_eq]=published`                 |
| `search`  | Full-text search                    | `?search=keyword`                                |
| `sort`    | Sort fields (prefix `-` for DESC)   | `?sort=-date_created,title`                      |
| `limit`   | Number of items to return           | `?limit=10`                                      |
| `offset`  | Number of items to skip             | `?offset=20`                                     |
| `page`    | Page number (alternative to offset) | `?page=2`                                        |
| `meta`    | Include metadata                    | `?meta=*` or `?meta=total_count`                 |
| `deep`    | Deep query for nested relations     | `?deep[translations][_filter][language][_eq]=en` |

## Filter Operators

| Operator       | Description           | Example                                         |
| -------------- | --------------------- | ----------------------------------------------- |
| `_eq`          | Equal to              | `?filter[status][_eq]=published`                |
| `_neq`         | Not equal to          | `?filter[status][_neq]=draft`                   |
| `_lt`          | Less than             | `?filter[price][_lt]=100`                       |
| `_lte`         | Less than or equal    | `?filter[price][_lte]=100`                      |
| `_gt`          | Greater than          | `?filter[views][_gt]=1000`                      |
| `_gte`         | Greater than or equal | `?filter[views][_gte]=1000`                     |
| `_in`          | In array              | `?filter[status][_in]=published,archived`       |
| `_nin`         | Not in array          | `?filter[status][_nin]=draft,deleted`           |
| `_null`        | Is null               | `?filter[deleted_at][_null]=true`               |
| `_nnull`       | Is not null           | `?filter[published_at][_nnull]=true`            |
| `_contains`    | Contains substring    | `?filter[title][_contains]=tutorial`            |
| `_ncontains`   | Doesn't contain       | `?filter[title][_ncontains]=draft`              |
| `_starts_with` | Starts with           | `?filter[slug][_starts_with]=blog-`             |
| `_ends_with`   | Ends with             | `?filter[email][_ends_with]=@example.com`       |
| `_between`     | Between two values    | `?filter[date][_between]=2024-01-01,2024-12-31` |
| `_empty`       | Is empty              | `?filter[description][_empty]=true`             |
| `_nempty`      | Is not empty          | `?filter[description][_nempty]=true`            |

## Rate Limiting

Certain endpoints are rate-limited:

- `/auth/login`: Configurable via `AUTH_LOGIN_ATTEMPTS` and `AUTH_LOGIN_ATTEMPTS_DURATION`
- `/auth/password/request`: Configurable via rate limiter settings
- `/users/register`: Rate limited to prevent abuse

When rate limit is exceeded, the API returns:

```json
{
	"errors": [
		{
			"message": "Rate limit exceeded",
			"extensions": {
				"code": "RATE_LIMIT_EXCEEDED"
			}
		}
	]
}
```
