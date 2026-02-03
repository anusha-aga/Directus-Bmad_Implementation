# Usage Workflows

This document outlines realistic end-to-end workflows for common tasks in Directus. Each workflow is based on actual
system functionality and API endpoints.

---

## Workflow 1: Setting Up a New Content Collection

**Scenario**: A developer wants to create a new "Blog Articles" collection with fields, permissions, and relationships.

### Preconditions

- User is authenticated with admin privileges
- Database connection is established
- User has access to the Directus Admin App or API

### Steps

#### 1. Create the Collection

**Via API**:

```bash
POST /collections
Authorization: Bearer <admin_token>
Content-Type: application/json

{
  "collection": "articles",
  "meta": {
    "icon": "article",
    "note": "Blog articles collection",
    "display_template": "{{title}}"
  },
  "schema": {
    "name": "articles"
  },
  "fields": [
    {
      "field": "id",
      "type": "integer",
      "meta": { "hidden": true },
      "schema": {
        "is_primary_key": true,
        "has_auto_increment": true
      }
    }
  ]
}
```

**Expected Response**:

```json
{
	"data": {
		"collection": "articles",
		"meta": { "icon": "article" }
	}
}
```

#### 2. Add Fields to the Collection

**Via API**:

```bash
POST /fields/articles
Authorization: Bearer <admin_token>

{
  "field": "title",
  "type": "string",
  "meta": {
    "interface": "input",
    "required": true,
    "width": "full"
  },
  "schema": {
    "max_length": 255
  }
}
```

Repeat for additional fields:

- `slug` (string, unique)
- `content` (text)
- `status` (string, with dropdown interface)
- `author` (many-to-one relation to `directus_users`)
- `date_created` (timestamp)
- `date_updated` (timestamp)

#### 3. Create a Relation to Users

**Via API**:

```bash
POST /relations
Authorization: Bearer <admin_token>

{
  "collection": "articles",
  "field": "author",
  "related_collection": "directus_users",
  "meta": {
    "many_collection": "articles",
    "many_field": "author",
    "one_collection": "directus_users"
  },
  "schema": {
    "on_delete": "SET NULL"
  }
}
```

#### 4. Set Up Permissions

**Via API**:

```bash
POST /permissions
Authorization: Bearer <admin_token>

{
  "role": "<editor_role_uuid>",
  "collection": "articles",
  "action": "create",
  "permissions": {},
  "fields": ["*"]
}
```

Repeat for other actions: `read`, `update`, `delete` with appropriate filters.

#### 5. Create a Preset (Default View)

**Via API**:

```bash
POST /presets
Authorization: Bearer <admin_token>

{
  "collection": "articles",
  "layout": "tabular",
  "layout_query": {
    "tabular": {
      "fields": ["title", "author", "status", "date_created"]
    }
  },
  "layout_options": {
    "tabular": {
      "widths": {
        "title": 300,
        "author": 150,
        "status": 100,
        "date_created": 150
      }
    }
  }
}
```

### Expected Outcomes

- ✅ New `articles` collection exists in the database
- ✅ Collection has all required fields with proper types
- ✅ Relation to `directus_users` is established
- ✅ Permissions are configured for editor role
- ✅ Default view preset is saved
- ✅ Collection appears in Admin App sidebar

### Verification

```bash
GET /collections/articles
GET /fields/articles
GET /relations?filter[many_collection][_eq]=articles
```

---

## Workflow 2: Content Editor Publishing an Article

**Scenario**: A content editor creates, edits, and publishes a blog article through the API or Admin App.

### Preconditions

- User is authenticated as a content editor
- `articles` collection exists with proper permissions
- User has `create` and `update` permissions on `articles`
- `status` field has values: `draft`, `published`, `archived`

### Steps

#### 1. Create a Draft Article

**Via API**:

```bash
POST /items/articles
Authorization: Bearer <editor_token>
Content-Type: application/json

{
  "title": "Getting Started with Directus",
  "slug": "getting-started-with-directus",
  "content": "Directus is a powerful headless CMS...",
  "status": "draft",
  "author": "<current_user_uuid>"
}
```

**Expected Response**:

```json
{
	"data": {
		"id": 1,
		"title": "Getting Started with Directus",
		"slug": "getting-started-with-directus",
		"status": "draft",
		"author": "<user_uuid>",
		"date_created": "2024-02-03T16:00:00Z"
	}
}
```

#### 2. Upload a Featured Image

**Via API**:

```bash
POST /files
Authorization: Bearer <editor_token>
Content-Type: multipart/form-data

--boundary
Content-Disposition: form-data; name="title"
Featured Image
--boundary
Content-Disposition: form-data; name="folder"
<articles_folder_uuid>
--boundary
Content-Disposition: form-data; name="file"; filename="featured.jpg"
Content-Type: image/jpeg

<binary image data>
--boundary--
```

**Expected Response**:

```json
{
	"data": {
		"id": "<file_uuid>",
		"title": "Featured Image",
		"filename_download": "featured.jpg",
		"type": "image/jpeg"
	}
}
```

#### 3. Update Article with Featured Image

**Via API**:

```bash
PATCH /items/articles/1
Authorization: Bearer <editor_token>

{
  "featured_image": "<file_uuid>"
}
```

#### 4. Preview the Article

**Via API**:

```bash
GET /items/articles/1?fields=*,author.first_name,author.last_name,featured_image.*
Authorization: Bearer <editor_token>
```

**Expected Response**:

```json
{
	"data": {
		"id": 1,
		"title": "Getting Started with Directus",
		"content": "Directus is a powerful headless CMS...",
		"status": "draft",
		"author": {
			"first_name": "John",
			"last_name": "Doe"
		},
		"featured_image": {
			"id": "<file_uuid>",
			"filename_download": "featured.jpg",
			"type": "image/jpeg"
		}
	}
}
```

#### 5. Publish the Article

**Via API**:

```bash
PATCH /items/articles/1
Authorization: Bearer <editor_token>

{
  "status": "published"
}
```

#### 6. Verify Publication

**Via API**:

```bash
GET /items/articles?filter[status][_eq]=published&fields=id,title,status
Authorization: Bearer <editor_token>
```

### Expected Outcomes

- ✅ Article is created with `draft` status
- ✅ Featured image is uploaded and linked
- ✅ Article status is updated to `published`
- ✅ Article appears in published articles list
- ✅ Activity log records all changes
- ✅ Revision history is maintained

### Verification

```bash
GET /items/articles/1
GET /activity?filter[collection][_eq]=articles&filter[item][_eq]=1
GET /revisions?filter[collection][_eq]=articles&filter[item][_eq]=1
```

---

## Workflow 3: User Registration and Authentication

**Scenario**: A new user registers, verifies their email, and logs in to access the system.

### Preconditions

- Public registration is enabled (`PUBLIC_REGISTRATION=true`)
- Email service is configured
- Default role for new users is set (`PUBLIC_REGISTRATION_ROLE`)

### Steps

#### 1. User Registers

**Via API**:

```bash
POST /users/register
Content-Type: application/json

{
  "email": "newuser@example.com",
  "password": "SecurePassword123!",
  "first_name": "Jane",
  "last_name": "Smith",
  "verification_url": "https://myapp.com/verify"
}
```

**Expected Response**:

```json
{
	"data": null
}
```

**System Actions**:

- User record created with `status: unverified`
- Verification email sent to `newuser@example.com`
- Email contains verification link with token

#### 2. User Verifies Email

**Via Browser**: User clicks link in email:

```
GET /users/register/verify-email?token=<verification_token>
```

**Expected Response**:

- Redirect to `/admin/users/<user_id>`
- User status updated to `active`

#### 3. User Logs In

**Via API**:

```bash
POST /auth/login
Content-Type: application/json

{
  "email": "newuser@example.com",
  "password": "SecurePassword123!",
  "mode": "json"
}
```

**Expected Response**:

```json
{
	"data": {
		"access_token": "eyJhbGciOiJIUzI1NiIs...",
		"refresh_token": "abc123def456...",
		"expires": 900000
	}
}
```

#### 4. User Accesses Protected Resource

**Via API**:

```bash
GET /users/me
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

**Expected Response**:

```json
{
	"data": {
		"id": "<user_uuid>",
		"email": "newuser@example.com",
		"first_name": "Jane",
		"last_name": "Smith",
		"role": "<default_role_uuid>",
		"status": "active"
	}
}
```

#### 5. Token Refresh (Before Expiry)

**Via API**:

```bash
POST /auth/refresh
Content-Type: application/json

{
  "refresh_token": "abc123def456...",
  "mode": "json"
}
```

**Expected Response**:

```json
{
	"data": {
		"access_token": "eyJhbGciOiJIUzI1NiIs...",
		"refresh_token": "xyz789ghi012...",
		"expires": 900000
	}
}
```

### Expected Outcomes

- ✅ User account is created
- ✅ Verification email is sent and received
- ✅ Email is verified successfully
- ✅ User can log in with credentials
- ✅ Access token is issued
- ✅ User can access protected resources
- ✅ Token can be refreshed

### Verification

```bash
GET /users?filter[email][_eq]=newuser@example.com
GET /activity?filter[user][_eq]=<user_uuid>
```

---

## Workflow 4: File Upload and Image Transformation

**Scenario**: A user uploads an image and retrieves transformed versions for different use cases.

### Preconditions

- User is authenticated
- User has `create` permission on `directus_files`
- Storage driver is configured (local, S3, etc.)

### Steps

#### 1. Upload Original Image

**Via API**:

```bash
POST /files
Authorization: Bearer <token>
Content-Type: multipart/form-data

--boundary
Content-Disposition: form-data; name="title"
Product Photo
--boundary
Content-Disposition: form-data; name="folder"
<products_folder_uuid>
--boundary
Content-Disposition: form-data; name="file"; filename="product.jpg"
Content-Type: image/jpeg

<binary image data>
--boundary--
```

**Expected Response**:

```json
{
	"data": {
		"id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
		"title": "Product Photo",
		"filename_download": "product.jpg",
		"filename_disk": "a1b2c3d4-e5f6-7890-abcd-ef1234567890.jpg",
		"type": "image/jpeg",
		"filesize": 2048576,
		"width": 3000,
		"height": 2000,
		"storage": "local"
	}
}
```

#### 2. Get Thumbnail (400x400, Cover)

**Via Browser or API**:

```
GET /assets/a1b2c3d4-e5f6-7890-abcd-ef1234567890?width=400&height=400&fit=cover&quality=80
```

**Expected Response**:

- Binary image data (JPEG)
- Dimensions: 400x400px
- Cropped to cover the area

#### 3. Get Banner Image (1200x300, Cover)

**Via Browser or API**:

```
GET /assets/a1b2c3d4-e5f6-7890-abcd-ef1234567890?width=1200&height=300&fit=cover&format=webp&quality=85
```

**Expected Response**:

- Binary image data (WebP)
- Dimensions: 1200x300px
- Optimized for web

#### 4. Get Full-Width Responsive (Max 1920px)

**Via Browser or API**:

```
GET /assets/a1b2c3d4-e5f6-7890-abcd-ef1234567890?width=1920&fit=inside&quality=90
```

**Expected Response**:

- Binary image data (JPEG)
- Max width: 1920px
- Maintains aspect ratio

#### 5. Update File Metadata

**Via API**:

```bash
PATCH /files/a1b2c3d4-e5f6-7890-abcd-ef1234567890
Authorization: Bearer <token>

{
  "title": "Premium Product Photo",
  "description": "High-quality product image",
  "tags": ["product", "featured"]
}
```

### Expected Outcomes

- ✅ Original image is uploaded and stored
- ✅ File metadata is saved in database
- ✅ Transformed images are generated on-demand
- ✅ Transformations are cached for performance
- ✅ Metadata can be updated independently

### Verification

```bash
GET /files/a1b2c3d4-e5f6-7890-abcd-ef1234567890
GET /activity?filter[collection][_eq]=directus_files&filter[item][_eq]=a1b2c3d4-e5f6-7890-abcd-ef1234567890
```

---

## Workflow 5: Managing Permissions and Roles

**Scenario**: An admin creates a new role with specific permissions for content editors.

### Preconditions

- User is authenticated as admin
- Collections exist (`articles`, `categories`)
- Understanding of RBAC model

### Steps

#### 1. Create a New Role

**Via API**:

```bash
POST /roles
Authorization: Bearer <admin_token>

{
  "name": "Content Editor",
  "icon": "edit",
  "description": "Can create and edit articles",
  "admin_access": false,
  "app_access": true
}
```

**Expected Response**:

```json
{
	"data": {
		"id": "<role_uuid>",
		"name": "Content Editor",
		"icon": "edit",
		"admin_access": false,
		"app_access": true
	}
}
```

#### 2. Grant Read Permission on Articles

**Via API**:

```bash
POST /permissions
Authorization: Bearer <admin_token>

{
  "role": "<role_uuid>",
  "collection": "articles",
  "action": "read",
  "permissions": {},
  "fields": ["*"]
}
```

#### 3. Grant Create Permission with Field Restrictions

**Via API**:

```bash
POST /permissions
Authorization: Bearer <admin_token>

{
  "role": "<role_uuid>",
  "collection": "articles",
  "action": "create",
  "permissions": {},
  "fields": ["title", "slug", "content", "status", "author", "featured_image"],
  "presets": {
    "author": "$CURRENT_USER"
  }
}
```

#### 4. Grant Update Permission (Own Items Only)

**Via API**:

```bash
POST /permissions
Authorization: Bearer <admin_token>

{
  "role": "<role_uuid>",
  "collection": "articles",
  "action": "update",
  "permissions": {
    "author": {
      "_eq": "$CURRENT_USER"
    }
  },
  "fields": ["title", "content", "status", "featured_image"]
}
```

#### 5. Grant Delete Permission (Drafts Only)

**Via API**:

```bash
POST /permissions
Authorization: Bearer <admin_token>

{
  "role": "<role_uuid>",
  "collection": "articles",
  "action": "delete",
  "permissions": {
    "_and": [
      { "author": { "_eq": "$CURRENT_USER" } },
      { "status": { "_eq": "draft" } }
    ]
  }
}
```

#### 6. Assign Role to User

**Via API**:

```bash
PATCH /users/<user_uuid>
Authorization: Bearer <admin_token>

{
  "role": "<role_uuid>"
}
```

### Expected Outcomes

- ✅ New role is created
- ✅ Permissions are configured for CRUD operations
- ✅ Field-level restrictions are applied
- ✅ Dynamic filters limit access to own items
- ✅ User is assigned to the role
- ✅ User can only perform allowed actions

### Verification

```bash
GET /roles/<role_uuid>
GET /permissions?filter[role][_eq]=<role_uuid>
GET /users/<user_uuid>?fields=role.*
```

**Test as Editor**:

```bash
# Should succeed (own draft)
DELETE /items/articles/1
Authorization: Bearer <editor_token>

# Should fail (published article)
DELETE /items/articles/2
Authorization: Bearer <editor_token>
```

---

## Workflow 6: Exporting and Importing Data

**Scenario**: An admin exports collection data for backup and imports it to another instance.

### Preconditions

- User is authenticated as admin
- Source and destination instances are accessible
- Collections have compatible schemas

### Steps

#### 1. Export Collection Data

**Via API**:

```bash
GET /items/articles?limit=-1&export=json
Authorization: Bearer <admin_token>
```

**Expected Response**:

```json
{
	"data": [
		{
			"id": 1,
			"title": "Article 1",
			"content": "...",
			"status": "published"
		},
		{
			"id": 2,
			"title": "Article 2",
			"content": "...",
			"status": "draft"
		}
	]
}
```

Save response to `articles_export.json`

#### 2. Export Schema Snapshot

**Via API**:

```bash
GET /schema/snapshot
Authorization: Bearer <admin_token>
```

Save response to `schema_snapshot.json`

#### 3. Apply Schema to Destination Instance

**Via API** (on destination):

```bash
POST /schema/apply
Authorization: Bearer <destination_admin_token>
Content-Type: application/json

<contents of schema_snapshot.json>
```

#### 4. Import Data to Destination

**Via API** (on destination):

```bash
POST /items/articles
Authorization: Bearer <destination_admin_token>
Content-Type: application/json

[
  {
    "title": "Article 1",
    "content": "...",
    "status": "published"
  },
  {
    "title": "Article 2",
    "content": "...",
    "status": "draft"
  }
]
```

**Note**: Remove `id` fields to allow auto-increment on destination.

### Expected Outcomes

- ✅ Data is exported from source instance
- ✅ Schema is exported and applied to destination
- ✅ Data is imported to destination instance
- ✅ All relationships are maintained
- ✅ No data loss occurs

### Verification

```bash
GET /items/articles?limit=-1
GET /schema/snapshot
```

Compare counts and data integrity between source and destination.

---

## Common Patterns

### Error Handling

All workflows should handle common errors:

```javascript
try {
	const response = await fetch('/items/articles', {
		method: 'POST',
		headers: {
			Authorization: `Bearer ${token}`,
			'Content-Type': 'application/json',
		},
		body: JSON.stringify(data),
	});

	if (!response.ok) {
		const error = await response.json();
		console.error('API Error:', error);
		// Handle specific error codes
		if (error.errors[0].extensions.code === 'FORBIDDEN') {
			// Handle permission error
		}
	}
} catch (err) {
	console.error('Network Error:', err);
}
```

### Pagination

For large datasets:

```bash
GET /items/articles?limit=100&page=1
GET /items/articles?limit=100&page=2
GET /items/articles?limit=100&page=3
```

### Filtering and Searching

```bash
GET /items/articles?filter[status][_eq]=published&filter[author][_eq]=<user_uuid>&sort=-date_created&limit=10
```

### Deep Queries

```bash
GET /items/articles?fields=*,author.first_name,author.last_name,categories.categories_id.*&deep[categories][_filter][status][_eq]=active
```
