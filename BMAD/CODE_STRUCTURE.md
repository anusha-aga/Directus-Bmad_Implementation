# Code Structure

## Repository Overview

Directus is organized as a **pnpm monorepo** with the following top-level structure:

```
directus/
├── api/                    # Backend API (Node.js/Express)
├── app/                    # Frontend Admin App (Vue.js)
├── sdk/                    # JavaScript/TypeScript SDK
├── packages/               # Shared packages and utilities
├── tests/                  # Integration and blackbox tests
├── directus/               # CLI wrapper package
├── docker-compose.yml      # Development database services
└── pnpm-workspace.yaml     # Monorepo workspace configuration
```

## Core Directories

### `/api` - Backend API

The heart of the Directus system. Built with Node.js and Express.

**Key Structure:**

```
api/src/
├── app.ts                  # Express app initialization & middleware mounting
├── server.ts               # HTTP server creation & startup
├── start.ts                # Entry point
├── controllers/            # HTTP request handlers (36 files)
├── services/               # Business logic layer (52 files)
├── middleware/             # Request processing pipeline (19 files)
├── database/               # Database abstraction & migrations
├── permissions/            # Access control logic
├── auth/                   # Authentication strategies
├── extensions/             # Extension loading & management
├── operations/             # Flow operation handlers
├── websocket/              # WebSocket controllers & handlers
├── cli/                    # CLI commands
├── utils/                  # Utility functions
└── types/                  # TypeScript type definitions
```

**Core Business Logic Locations:**

1. **`services/items.ts`** (39KB, 1240 lines)
   - Generic CRUD operations for all collections
   - Handles relational data (M2O, O2M, M2M)
   - Permission validation
   - Event emission (hooks)
   - Activity tracking
   - **This is the most critical service** - all item operations flow through here

2. **`services/authentication.ts`** (16KB)
   - Login/logout logic
   - Token generation and verification
   - Password hashing (argon2)
   - SSO provider integration

3. **`services/files.ts`** (12KB)
   - File upload handling
   - Image transformation
   - Storage driver integration

4. **`services/collections.ts`** (27KB)
   - Collection creation/modification
   - Schema synchronization

5. **`services/fields.ts`** (33KB)
   - Field creation/modification
   - Schema validation

**Controllers:** Each controller maps to a REST endpoint:

- `items.ts` → `/items/:collection`
- `auth.ts` → `/auth/*`
- `files.ts` → `/files/*`
- `users.ts` → `/users/*`
- `graphql.ts` → `/graphql`

**Middleware Pipeline:**

```
Request Flow:
1. cors.ts                  # CORS validation
2. authenticate.ts          # JWT verification
3. rate-limiter-*.ts        # Rate limiting
4. schema.ts                # Schema injection
5. sanitize-query.ts        # Query parameter cleaning
6. cache.ts                 # Response caching (GET only)
→ Controller → Service → Database
```

### `/app` - Frontend Admin App

Vue.js SPA with dynamic UI generation.

**Key Structure:**

```
app/src/
├── main.ts                 # Vue app initialization
├── app.vue                 # Root component
├── router.ts               # Vue Router configuration
├── components/             # Reusable UI components (246 files)
├── views/                  # Page-level components (88 files)
├── modules/                # Feature modules (183 files)
├── interfaces/             # Field input components (257 files)
├── displays/               # Field display components (44 files)
├── layouts/                # Collection layout views (32 files)
├── panels/                 # Dashboard panels (38 files)
├── operations/             # Flow operation UI (17 files)
├── composables/            # Vue composition functions (77 files)
├── stores/                 # Pinia state stores (22 files)
├── utils/                  # Utility functions (125 files)
└── lang/                   # i18n translations (76 files)
```

**Key Modules:**

1. **`modules/content/`** - Main content management interface
2. **`modules/settings/`** - System settings and configuration
3. **`modules/insights/`** - Analytics and dashboards
4. **`modules/files/`** - File library management

**State Management (`stores/`):**

- `user.ts` - Current user and permissions
- `collections.ts` - Collection metadata
- `fields.ts` - Field metadata
- `permissions.ts` - Permission rules
- `server.ts` - Server info and health

### `/packages` - Shared Packages

Modular packages used across the monorepo:

```
packages/
├── env/                    # Environment variable handling
├── schema/                 # Database schema introspection
├── types/                  # TypeScript type definitions
├── errors/                 # Standardized error classes
├── constants/              # Shared constants
├── utils/                  # Utility functions
├── storage/                # Storage abstraction layer
├── storage-driver-*/       # Storage driver implementations
├── extensions-sdk/         # Extension development toolkit
├── extensions/             # Extension loading system
├── composables/            # Vue composables
├── stores/                 # Pinia stores
├── themes/                 # UI theming
└── system-data/            # System collection definitions
```

**Critical Packages:**

1. **`@directus/schema`** - Database introspection
   - Reads database structure
   - Provides unified schema interface
   - Supports all database vendors

2. **`@directus/storage`** - File storage abstraction
   - Unified API for all storage drivers
   - Supports local, S3, GCS, Azure, Cloudinary

3. **`@directus/extensions-sdk`** - Extension development
   - CLI for scaffolding extensions
   - Build tooling
   - Type definitions

### `/sdk` - JavaScript SDK

Client library for interacting with Directus API:

```
sdk/src/
├── index.ts                # Main SDK export
├── rest/                   # REST client
├── graphql/                # GraphQL client
├── auth/                   # Authentication helpers
├── realtime/               # WebSocket/SSE client
└── types/                  # TypeScript definitions
```

### `/tests` - Test Suite

```
tests/
├── blackbox/               # End-to-end API tests
│   ├── routes/            # Route-specific tests
│   └── setup/             # Test database setup
└── unit/                   # Unit tests (distributed in source)
```

## Data Flow Through the System

### 1. Request Lifecycle (Create Item Example)

```
Client Request
    ↓
[Middleware Pipeline]
    ├─ cors.ts              # Validate origin
    ├─ authenticate.ts      # Extract & verify JWT → req.accountability
    ├─ schema.ts            # Load schema → req.schema
    └─ sanitize-query.ts    # Clean query params → req.sanitizedQuery
    ↓
[Controller Layer]
    controllers/items.ts
    ├─ Parse route params (:collection)
    ├─ Instantiate ItemsService
    └─ Call service.createOne(payload)
    ↓
[Service Layer]
    services/items.ts
    ├─ Emit 'items.create.before' hook
    ├─ Validate permissions (permissions/modules/validate-access)
    ├─ Process payload (services/payload.ts)
    │   ├─ Handle M2O relations
    │   ├─ Handle A2O relations
    │   └─ Type casting
    ├─ Begin database transaction
    ├─ INSERT INTO collection
    ├─ Process O2M relations
    ├─ Create activity log (directus_activity)
    ├─ Commit transaction
    └─ Emit 'items.create.after' hook
    ↓
[Database Layer]
    database/index.ts (Knex)
    ├─ Build SQL query
    ├─ Execute on database
    └─ Return primary key
    ↓
Response to Client
```

### 2. Schema Loading Flow

```
App Startup
    ↓
database/index.ts
    ├─ Connect to database
    ├─ Validate migrations
    └─ Create SchemaInspector
    ↓
@directus/schema
    ├─ Introspect tables
    ├─ Introspect columns
    ├─ Introspect relations
    └─ Build SchemaOverview object
    ↓
middleware/schema.ts
    ├─ Cache schema in memory/Redis
    └─ Inject into req.schema
    ↓
Available to all services
```

### 3. Permission Validation Flow

```
Service Operation (e.g., createOne)
    ↓
permissions/modules/validate-access
    ├─ Get user's role from req.accountability
    ├─ Query directus_permissions table
    ├─ Filter by: collection, action, role
    ├─ Check field-level permissions
    ├─ Apply dynamic rules (filters)
    └─ Throw ForbiddenError if denied
    ↓
Continue with operation
```

### 4. Extension Loading Flow

```
App Startup
    ↓
extensions/index.ts
    ├─ Scan extensions/ directory
    ├─ Load package.json metadata
    ├─ Validate extension type
    ├─ Load extension code
    │   ├─ Hooks → Register with emitter
    │   ├─ Endpoints → Mount to Express
    │   ├─ Interfaces → Register with app
    │   └─ Operations → Register with flows
    └─ Initialize extensions
    ↓
Extensions active
```

## Key Files Reference

### Backend Entry Points

- **`api/src/start.ts`** - Application entry point
- **`api/src/server.ts`** - HTTP server creation
- **`api/src/app.ts`** - Express app configuration

### Core Services

- **`api/src/services/items.ts`** - Generic CRUD (most important)
- **`api/src/services/authentication.ts`** - Auth logic
- **`api/src/services/collections.ts`** - Collection management
- **`api/src/services/fields.ts`** - Field management
- **`api/src/services/files.ts`** - File handling

### Database

- **`api/src/database/index.ts`** - Database connection & Knex setup
- **`api/src/database/migrations/`** - Schema migrations
- **`api/src/database/helpers/`** - Database-specific helpers

### Permissions

- **`api/src/permissions/modules/validate-access/`** - Permission checking
- **`api/src/permissions/modules/process-ast/`** - Query filtering
- **`api/src/permissions/modules/process-payload/`** - Payload validation

### Frontend Entry Points

- **`app/src/main.ts`** - Vue app initialization
- **`app/src/router.ts`** - Route configuration
- **`app/src/app.vue`** - Root component

### Frontend State

- **`app/src/stores/user.ts`** - User state
- **`app/src/stores/collections.ts`** - Collections metadata
- **`app/src/stores/permissions.ts`** - Permission rules

## Module Responsibilities

| Module          | Responsibility        | Key Files                  |
| --------------- | --------------------- | -------------------------- |
| **Controllers** | HTTP request handling | `api/src/controllers/*.ts` |
| **Services**    | Business logic        | `api/src/services/*.ts`    |
| **Middleware**  | Request processing    | `api/src/middleware/*.ts`  |
| **Database**    | Data persistence      | `api/src/database/`        |
| **Permissions** | Access control        | `api/src/permissions/`     |
| **Extensions**  | Plugin system         | `api/src/extensions/`      |
| **Auth**        | Authentication        | `api/src/auth/`            |
| **Operations**  | Flow operations       | `api/src/operations/`      |
| **WebSocket**   | Real-time             | `api/src/websocket/`       |
| **CLI**         | Command-line tools    | `api/src/cli/`             |
| **Components**  | UI building blocks    | `app/src/components/`      |
| **Views**       | Page components       | `app/src/views/`           |
| **Modules**     | Feature areas         | `app/src/modules/`         |
| **Interfaces**  | Field inputs          | `app/src/interfaces/`      |
| **Displays**    | Field outputs         | `app/src/displays/`        |
| **Layouts**     | Collection views      | `app/src/layouts/`         |

## Where to Find Things

**Need to modify...**

- **API endpoints?** → `api/src/controllers/`
- **Business logic?** → `api/src/services/`
- **Database queries?** → `api/src/services/` (uses Knex)
- **Permissions?** → `api/src/permissions/`
- **Authentication?** → `api/src/auth/` & `api/src/services/authentication.ts`
- **File handling?** → `api/src/services/files.ts` & `packages/storage/`
- **UI components?** → `app/src/components/`
- **Page layouts?** → `app/src/views/`
- **Field interfaces?** → `app/src/interfaces/`
- **State management?** → `app/src/stores/`
- **Database schema?** → `api/src/database/migrations/`
- **Extension types?** → `packages/extensions-sdk/`
