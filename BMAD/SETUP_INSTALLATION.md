# Setup and Installation

## Prerequisites

- **Node.js**: Version 22 (Enforced by `.npmrc` and `engines` field).
- **Package Manager**: pnpm (Version >= 10 and < 11).
- **Database**: A supported SQL database (Postgres, MySQL, SQLite, MSSQL, MariaDB, CockroachDB, Oracle, or Redshift).
- **Docker** (Optional): Recommended for spinning up local database instances.

## Environment Variables

The system configuration is driven by environment variables. A full list of defaults can be found in
`packages/env/src/constants/defaults.ts`.

### Core

- `HOST`: Host to bind to (Default: `0.0.0.0`)
- `PORT`: Port to listen on (Default: `8055`)
- `PUBLIC_URL`: Public URL of the project (Default: `/`)
- `KEY`: (Required for production) Unique key for signing tokens.
- `SECRET`: (Required for production) Secret for signing tokens.

### Database

Configuration depends on `DB_CLIENT` (e.g., `postgres`, `mysql`, `sqlite`).

- `DB_CLIENT`: Database driver (Required).
- `DB_HOST`: Database host.
- `DB_PORT`: Database port.
- `DB_DATABASE`: Database name.
- `DB_USER`: Database user.
- `DB_PASSWORD`: Database password.
- `DB_FILENAME`: Path to SQLite file (if `DB_CLIENT` is `sqlite3`).
- `DB_CONNECTION_STRING`: Connection string override (Postgres/CockroachDB).

> **Note**: Inspect `api/src/database/index.ts` for logic regarding specific database requirements.

## Installation

1.  **Clone the repository**:
    ```bash
    git clone https://github.com/directus/directus.git
    cd directus
    ```
2.  **Install dependencies**:
    ```bash
    pnpm install
    ```
3.  **Build the project**: The system is a monorepo. You can build all packages from the root:
    ```bash
    pnpm build
    ```

## Local Development

To run the project locally, you typically start the API and App independently or use the provided scripts.

### Backend (API)

1.  Ensure you have a database running. You can use the provided `docker-compose.yml` for debugging/development:
    ```bash
    docker-compose up -d postgres
    # or mysql, redis, etc.
    ```
2.  Navigate to the API package or run from root using filters:
    ```bash
    pnpm --filter @directus/api dev
    ```
    This script runs `tsx watch src/start.ts`.

### Frontend (Admin App)

1.  Navigate to the App package:
    ```bash
    pnpm --filter @directus/app dev
    ```
    This starts the Vite development server.

## Testing

- **Unit/Integration Tests**: Run `pnpm test` from the root to run tests across packages using Vitest.
- **Blackbox Tests**: `pnpm test:blackbox` (Requires building and deploying a distribution first).
- **Coverage**: `pnpm test:coverage`.

## Known Issues / Gaps

- **Missing .env.example**: The repository does not include a root `.env.example` file. Developers must infer
  requirements from `packages/env` or documentation.
- **Database Setup**: There are no automated scripts to seed the database for local dev beyond what the API handles on
  startup (migrations).
