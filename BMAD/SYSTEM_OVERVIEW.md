# System Overview

## High-Level Description

Directus is a real-time, open-source Headless CMS and Backend-as-a-Service (BaaS). It acts as a dynamic wrapper around a
SQL database, instantly providing a REST and GraphQL API, along with an intuitive Admin App for managing content and
configurations. Unlike traditional CMSs, Directus mirrors the database schema directly, meaning it has no opinionated
structure—what you build in the database is exactly what you get in the API and App.

The system is built as a monorepo, primarily separated into a Node.js backend (`@directus/api`) and a Vue.js frontend
(`@directus/app`), supported by a suite of modular packages for utilities, storage drivers, and extension SDKs.

## User Personas & Purpose

- **Developers**: Use Directus to rapidly spin up backend infrastructure, manage database schemas programmatically or
  visually, and extend functionality via the Extensions SDK.
- **Content Editors / Non-Technical Users**: Use the Admin App to manage content, assets, and user data without needing
  to write SQL or interact with the API directly.
- **Data Analysts**: Utilize the insights and dashboarding features to visualize data stored in the connected SQL
  database.

## Core Features

- **Dynamic API Generation**: Automatically generates REST and GraphQL endpoints based on the introspection of the
  underlying SQL database schema.
- **Admin App**: A Vue.js-based Single Page Application (SPA) that serves as a control panel for content, users, and
  system settings.
- **Database Abstraction**: Supports multiple SQL dialects (PostgreSQL, MySQL, SQLite, CockroachDB, etc.) via Knex.js,
  allowing the system to work with new or existing databases.
- **Modular Storage**: detailed abstraction for file storage, supporting local disk, AWS S3, Google Cloud Storage,
  Azure, and others via dedicated driver packages (`@directus/storage-driver-*`).
- **Extensibility**: A robust extension ecosystem allowing custom interfaces, layouts, displays, panels, hooks, and
  endpoints (`@directus/extensions-sdk`).
- **Authentication & RBAC**: Built-in granular permission system and authentication handling (JWT), configurable via the
  Admin App and enforced at the API layer.

## Key Workflows

### Schema Management

1.  **Definition**: Administrators define data models (tables/collections and columns/fields) via the Admin App or
    modify the database directly.
2.  **Introspection**: The API layer introspects these changes.
3.  **Propagation**: The system updates the internal schema cache, instantly reflecting changes in the API endpoints and
    App interfaces.

### Content Management

1.  **Input**: Users enter data via the Admin App or POST to the API.
2.  **Validation**: input is validated against defined field rules and types.
3.  **Persistance**: Data is sanitized and stored in the SQL database.
4.  **Retrieval**: Data is retrievable via the API or App, respecting configured read permissions.

### Extension Development

1.  **Creation**: Developers scaffold a new extension using the CLI (`create-directus-extension`).
2.  **Implementation**: Logic or UI is implemented using Vue.js (frontend) or Node.js (backend).
3.  **Integration**: The extension is built and placed in the extensions directory.
4.  **Loading**: On startup, the API loads the extension, making the new functionality available in the system.
