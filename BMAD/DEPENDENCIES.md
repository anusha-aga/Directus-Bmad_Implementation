# Dependencies

## Overview

Directus is built on a modern Node.js stack with a Vue.js frontend. This document outlines the major runtime and build
dependencies, their purposes, and external service requirements.

---

## Runtime Requirements

### Node.js

- **Version**: Node.js v22.x (LTS)
- **Purpose**: JavaScript runtime for the backend API
- **Why it matters**: Directus is built entirely on Node.js. The specific version ensures compatibility with modern ES
  modules, async/await patterns, and performance optimizations.

### Package Manager

- **Tool**: pnpm
- **Purpose**: Monorepo package management
- **Why it matters**: Manages workspace dependencies efficiently, reduces disk space usage, and ensures consistent
  dependency resolution across the monorepo.

### Database (Required)

Directus requires one of the following SQL databases:

- **PostgreSQL** (recommended)
- **MySQL** / **MariaDB**
- **SQLite** (development only)
- **Microsoft SQL Server**
- **CockroachDB**
- **Oracle Database**

**Why it matters**: Directus is database-first. All content, schema, and configuration are stored in the SQL database.
The system dynamically introspects the database schema at runtime.

---

## Backend API Dependencies (`@directus/api`)

### Core Framework

#### Express.js (`express`)

- **Purpose**: HTTP server framework
- **Why it matters**: Handles all HTTP routing, middleware pipeline, and request/response processing. The entire REST
  API is built on Express.

#### Knex.js (`knex`)

- **Purpose**: SQL query builder and database abstraction
- **Why it matters**: Provides database-agnostic query building, migrations, and connection pooling. Supports all
  database vendors Directus works with.

### Authentication & Security

#### argon2 (`argon2`)

- **Purpose**: Password hashing
- **Why it matters**: Industry-standard password hashing algorithm. More secure than bcrypt, resistant to GPU-based
  attacks.

#### jsonwebtoken (`jsonwebtoken`)

- **Purpose**: JWT token generation and verification
- **Why it matters**: Used for stateless authentication. Access tokens and refresh tokens are JWTs.

#### Helmet (`helmet`)

- **Purpose**: HTTP security headers
- **Why it matters**: Sets security-related HTTP headers (CSP, HSTS, etc.) to protect against common web
  vulnerabilities.

#### otplib (`otplib`)

- **Purpose**: Two-factor authentication (TOTP)
- **Why it matters**: Generates and validates time-based one-time passwords for 2FA.

### SSO & Authentication Providers

#### openid-client (`openid-client`)

- **Purpose**: OpenID Connect authentication
- **Why it matters**: Enables SSO with OpenID providers (Google, Azure AD, etc.).

#### samlify (`samlify`)

- **Purpose**: SAML 2.0 authentication
- **Why it matters**: Enterprise SSO support for SAML-based identity providers.

#### ldapts (`ldapts`)

- **Purpose**: LDAP authentication
- **Why it matters**: Integrates with Active Directory and LDAP servers for enterprise authentication.

### Data Processing

#### sharp (`sharp`)

- **Purpose**: Image processing and transformation
- **Why it matters**: Handles image resizing, format conversion, and optimization for the `/assets` endpoint.
  High-performance native library.

#### archiver (`archiver`)

- **Purpose**: File compression
- **Why it matters**: Creates ZIP archives for bulk file exports.

#### papaparse (`papaparse`)

- **Purpose**: CSV parsing and generation
- **Why it matters**: Handles CSV import/export for data operations.

#### json2csv (`json2csv`)

- **Purpose**: JSON to CSV conversion
- **Why it matters**: Exports collection data as CSV files.

### GraphQL

#### graphql (`graphql`)

- **Purpose**: GraphQL query execution
- **Why it matters**: Provides the GraphQL API endpoint alongside REST.

#### graphql-compose (`graphql-compose`)

- **Purpose**: GraphQL schema composition
- **Why it matters**: Dynamically generates GraphQL schemas from database schema.

#### graphql-ws (`graphql-ws`)

- **Purpose**: GraphQL over WebSocket
- **Why it matters**: Enables real-time GraphQL subscriptions.

### Real-time & WebSocket

#### ws (`ws`)

- **Purpose**: WebSocket server
- **Why it matters**: Powers real-time features, live queries, and subscriptions.

#### eventemitter2 (`eventemitter2`)

- **Purpose**: Event emitter with wildcards
- **Why it matters**: Powers the hook system. Extensions can listen to events like `items.create.before`.

### Caching & Storage

#### ioredis (`ioredis`)

- **Purpose**: Redis client
- **Why it matters**: Optional Redis integration for caching, rate limiting, and session storage.

#### keyv (`keyv`)

- **Purpose**: Key-value storage abstraction
- **Why it matters**: Unified interface for cache storage (memory, Redis, etc.).

### Email

#### nodemailer (`nodemailer`)

- **Purpose**: Email sending
- **Why it matters**: Sends transactional emails (password reset, user invitations, etc.).

#### @aws-sdk/client-sesv2 (`@aws-sdk/client-sesv2`)

- **Purpose**: AWS SES email service
- **Why it matters**: Optional integration with Amazon SES for email delivery.

### Validation & Schemas

#### joi (`joi`)

- **Purpose**: Schema validation
- **Why it matters**: Validates request payloads, environment variables, and configuration.

#### zod (`zod`)

- **Purpose**: TypeScript-first schema validation
- **Why it matters**: Type-safe validation for modern TypeScript code.

### Logging & Monitoring

#### pino (`pino`)

- **Purpose**: High-performance logging
- **Why it matters**: Structured JSON logging with minimal performance overhead.

#### pino-http (`pino-http`)

- **Purpose**: HTTP request logging
- **Why it matters**: Logs all HTTP requests with timing and metadata.

#### prom-client (`prom-client`)

- **Purpose**: Prometheus metrics
- **Why it matters**: Exposes metrics endpoint for monitoring (if `METRICS_ENABLED=true`).

### Scheduling & Background Jobs

#### cron (`cron`)

- **Purpose**: Cron job scheduling
- **Why it matters**: Schedules recurring tasks and flows.

#### p-queue (`p-queue`)

- **Purpose**: Promise queue with concurrency control
- **Why it matters**: Manages background job execution and rate limiting.

### File Upload

#### busboy (`busboy`)

- **Purpose**: Multipart form data parsing
- **Why it matters**: Handles file uploads via `multipart/form-data`.

#### @tus/server (`@tus/server`)

- **Purpose**: Resumable file uploads
- **Why it matters**: Implements TUS protocol for large file uploads with resume capability.

### AI Integration (Optional)

#### ai (`ai`)

- **Purpose**: Vercel AI SDK
- **Why it matters**: Unified interface for AI model interactions.

#### @ai-sdk/openai, @ai-sdk/anthropic, @ai-sdk/google

- **Purpose**: AI provider integrations
- **Why it matters**: Enables AI assistant features (if `AI_ENABLED=true`).

### Templating & Rendering

#### liquidjs (`liquidjs`)

- **Purpose**: Liquid template engine
- **Why it matters**: Used in email templates and dynamic content rendering.

#### marked (`marked`)

- **Purpose**: Markdown parser
- **Why it matters**: Renders markdown content in emails and notifications.

### Utilities

#### date-fns (`date-fns`)

- **Purpose**: Date manipulation
- **Why it matters**: Modern, lightweight alternative to Moment.js for date operations.

#### nanoid (`nanoid`)

- **Purpose**: Unique ID generation
- **Why it matters**: Generates short, URL-safe unique identifiers.

#### lodash-es (`lodash-es`)

- **Purpose**: Utility functions
- **Why it matters**: Common data manipulation utilities (ES module version).

#### axios (`axios`)

- **Purpose**: HTTP client
- **Why it matters**: Makes external HTTP requests (webhooks, file imports, etc.).

---

## Frontend App Dependencies (`@directus/app`)

### Core Framework

#### Vue.js (`vue`)

- **Version**: Vue 3
- **Purpose**: Frontend framework
- **Why it matters**: The entire Admin App is built with Vue 3 Composition API.

#### Vue Router (`vue-router`)

- **Purpose**: Client-side routing
- **Why it matters**: Handles navigation between different views in the Admin App.

#### Pinia (`pinia`)

- **Purpose**: State management
- **Why it matters**: Vue 3's official state management library. Stores user, collections, permissions, etc.

### Build Tools

#### Vite (`vite`)

- **Purpose**: Build tool and dev server
- **Why it matters**: Fast HMR during development, optimized production builds. Replaces Webpack.

#### @vitejs/plugin-vue (`@vitejs/plugin-vue`)

- **Purpose**: Vue plugin for Vite
- **Why it matters**: Enables Vue SFC (Single File Component) support in Vite.

### UI Components & Libraries

#### @vueuse/core (`@vueuse/core`)

- **Purpose**: Vue composition utilities
- **Why it matters**: Collection of essential Vue composition functions (useLocalStorage, useEventListener, etc.).

#### reka-ui (`reka-ui`)

- **Purpose**: Headless UI components
- **Why it matters**: Accessible, unstyled UI primitives for building custom components.

#### @popperjs/core (`@popperjs/core`)

- **Purpose**: Tooltip and popover positioning
- **Why it matters**: Powers dropdowns, tooltips, and context menus.

### Rich Text Editors

#### @editorjs/editorjs (`@editorjs/editorjs`)

- **Purpose**: Block-style editor
- **Why it matters**: Powers the WYSIWYG interface for rich content editing.

#### tinymce (`tinymce`)

- **Purpose**: Traditional WYSIWYG editor
- **Why it matters**: Alternative rich text editor option.

#### codemirror (`codemirror`)

- **Purpose**: Code editor
- **Why it matters**: Syntax-highlighted code editing for JSON, SQL, etc.

### Data Visualization

#### apexcharts (`apexcharts`)

- **Purpose**: Charting library
- **Why it matters**: Renders charts and graphs in insights/dashboards.

#### @fullcalendar/core (`@fullcalendar/core`)

- **Purpose**: Calendar component
- **Why it matters**: Calendar layout for date-based content.

### Maps & Geospatial

#### mapbox-gl (`mapbox-gl`)

- **Purpose**: Interactive maps
- **Why it matters**: Map interface for geospatial data fields.

#### maplibre-gl (`maplibre-gl`)

- **Purpose**: Open-source map library
- **Why it matters**: Alternative to Mapbox (no API key required).

#### @mapbox/mapbox-gl-draw (`@mapbox/mapbox-gl-draw`)

- **Purpose**: Drawing tools for maps
- **Why it matters**: Enables drawing and editing geometries on maps.

### File Handling

#### cropperjs (`cropperjs`)

- **Purpose**: Image cropping
- **Why it matters**: Crop and adjust images before upload.

#### file-saver (`file-saver`)

- **Purpose**: Client-side file saving
- **Why it matters**: Downloads files from the browser.

#### tus-js-client (`tus-js-client`)

- **Purpose**: Resumable uploads client
- **Why it matters**: Client-side implementation of TUS protocol.

### Utilities

#### axios (`axios`)

- **Purpose**: HTTP client
- **Why it matters**: Makes API requests to the Directus backend.

#### date-fns (`date-fns`)

- **Purpose**: Date manipulation
- **Why it matters**: Formats and manipulates dates in the UI.

#### lodash (`lodash`)

- **Purpose**: Utility functions
- **Why it matters**: Data manipulation and transformation.

#### marked (`marked`)

- **Purpose**: Markdown rendering
- **Why it matters**: Displays markdown content in the UI.

#### qrcode (`qrcode`)

- **Purpose**: QR code generation
- **Why it matters**: Generates QR codes for 2FA setup.

#### dompurify (`dompurify`)

- **Purpose**: HTML sanitization
- **Why it matters**: Prevents XSS attacks when rendering user-generated HTML.

---

## Directus Internal Packages (Workspace Dependencies)

These are internal packages within the Directus monorepo:

### Shared Packages

- `@directus/env` - Environment variable handling
- `@directus/errors` - Standardized error classes
- `@directus/schema` - Database schema introspection
- `@directus/types` - TypeScript type definitions
- `@directus/utils` - Shared utility functions
- `@directus/constants` - Shared constants
- `@directus/validation` - Validation utilities
- `@directus/system-data` - System collection definitions

### Storage Drivers

- `@directus/storage` - Storage abstraction layer
- `@directus/storage-driver-local` - Local filesystem storage
- `@directus/storage-driver-s3` - AWS S3 storage
- `@directus/storage-driver-gcs` - Google Cloud Storage
- `@directus/storage-driver-azure` - Azure Blob Storage
- `@directus/storage-driver-cloudinary` - Cloudinary storage
- `@directus/storage-driver-supabase` - Supabase storage

### Extensions

- `@directus/extensions` - Extension loading system
- `@directus/extensions-sdk` - Extension development toolkit
- `@directus/extensions-registry` - Extension registry

### Frontend

- `@directus/composables` - Vue composables
- `@directus/stores` - Pinia stores
- `@directus/themes` - UI theming
- `@directus/sdk` - JavaScript SDK

---

## External Services (Optional)

### Required

- **SQL Database**: PostgreSQL, MySQL, SQLite, MSSQL, CockroachDB, or Oracle

### Optional

#### Caching & Session Storage

- **Redis**: For caching, rate limiting, and session storage
  - Environment: `CACHE_STORE=redis`, `REDIS=redis://localhost:6379`

#### File Storage

- **AWS S3**: Object storage
  - Environment: `STORAGE_LOCATIONS=s3`, `STORAGE_S3_*` variables
- **Google Cloud Storage**: Object storage
  - Environment: `STORAGE_LOCATIONS=gcs`, `STORAGE_GCS_*` variables
- **Azure Blob Storage**: Object storage
  - Environment: `STORAGE_LOCATIONS=azure`, `STORAGE_AZURE_*` variables
- **Cloudinary**: Image and video management
  - Environment: `STORAGE_LOCATIONS=cloudinary`, `STORAGE_CLOUDINARY_*` variables

#### Email Services

- **SMTP Server**: Any SMTP-compatible email service
  - Environment: `EMAIL_TRANSPORT=smtp`, `EMAIL_SMTP_*` variables
- **AWS SES**: Amazon Simple Email Service
  - Environment: `EMAIL_TRANSPORT=ses`, `EMAIL_SES_*` variables
- **Mailgun**: Email delivery service
  - Environment: `EMAIL_TRANSPORT=mailgun`, `EMAIL_MAILGUN_*` variables
- **SendGrid**: Email delivery service
  - Environment: `EMAIL_TRANSPORT=sendgrid`, `EMAIL_SENDGRID_*` variables

#### AI Services (if `AI_ENABLED=true`)

- **OpenAI**: GPT models
  - Environment: `AI_PROVIDER=openai`, `AI_OPENAI_API_KEY`
- **Anthropic**: Claude models
  - Environment: `AI_PROVIDER=anthropic`, `AI_ANTHROPIC_API_KEY`
- **Google AI**: Gemini models
  - Environment: `AI_PROVIDER=google`, `AI_GOOGLE_API_KEY`

#### Monitoring & Observability

- **Prometheus**: Metrics collection (if `METRICS_ENABLED=true`)
- **Sentry**: Error tracking
  - Environment: `TELEMETRY_SENTRY_DSN`

#### Authentication Providers

- **OAuth2 Providers**: Google, GitHub, Facebook, etc.
  - Environment: `AUTH_<PROVIDER>_*` variables
- **LDAP/Active Directory**: Enterprise authentication
  - Environment: `AUTH_LDAP_*` variables
- **SAML Providers**: Enterprise SSO
  - Environment: `AUTH_SAML_*` variables

---

## Development Dependencies

### Testing

- **Vitest**: Unit and integration testing framework
- **@vitest/coverage-v8**: Code coverage reporting
- **@vue/test-utils**: Vue component testing utilities
- **happy-dom**: Lightweight DOM implementation for testing

### TypeScript

- **TypeScript**: Type checking and compilation
- **vue-tsc**: Vue TypeScript compiler
- **tsx**: TypeScript execution for Node.js

### Code Quality

- **ESLint**: JavaScript/TypeScript linting
- **Prettier**: Code formatting
- **Stylelint**: CSS/SCSS linting

---

## System Requirements Summary

### Minimum Requirements

- **Node.js**: v22.x
- **pnpm**: Latest version
- **Database**: PostgreSQL 12+, MySQL 8+, or SQLite 3.35+
- **RAM**: 512MB minimum (2GB+ recommended)
- **Disk**: 500MB for application + database storage

### Recommended Production Setup

- **Node.js**: v22.x LTS
- **Database**: PostgreSQL 14+ (managed service recommended)
- **Redis**: For caching and session storage
- **Object Storage**: S3-compatible service for file storage
- **RAM**: 2GB+ per instance
- **CPU**: 2+ cores
- **Load Balancer**: For horizontal scaling

### Optional Services

- Email service (SMTP, SES, SendGrid, Mailgun)
- AI provider (OpenAI, Anthropic, Google)
- Monitoring (Prometheus, Sentry)
- SSO provider (OAuth2, SAML, LDAP)

---

## Dependency Management

### Version Pinning

Directus uses a `catalog:` reference system in package.json files, which points to versions defined in
`pnpm-workspace.yaml`. This ensures consistent versions across all packages in the monorepo.

### Security Updates

Dependencies are regularly updated to address security vulnerabilities. The project uses automated dependency scanning
and update tools.

### Breaking Changes

Major dependency updates that introduce breaking changes are documented in release notes and migration guides.
