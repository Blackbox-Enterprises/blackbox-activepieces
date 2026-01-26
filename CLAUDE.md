# CLAUDE.md - AI Assistant Guide for Activepieces

This document provides guidance for AI assistants working with the Activepieces codebase.

## Project Overview

**Activepieces** is an open-source automation platform (Zapier alternative) that enables users to create workflows connecting various applications and services. This is a **BlackRoad OS, Inc.** proprietary fork.

- **Version**: 0.76.1
- **License**: BlackRoad OS Proprietary License
- **Repository Type**: Nx Monorepo with Bun package manager

## Codebase Structure

```
blackbox-activepieces/
├── packages/
│   ├── server/              # Backend (Fastify API)
│   │   ├── api/             # Main API server
│   │   ├── worker/          # Job execution worker
│   │   └── shared/          # Backend shared utilities
│   ├── react-ui/            # Frontend (React + Vite)
│   ├── engine/              # Flow execution engine
│   ├── pieces/              # Integration connectors
│   │   ├── community/       # 580+ community pieces
│   │   └── custom/          # Custom pieces
│   ├── shared/              # Shared TypeScript types
│   ├── cli/                 # CLI tool for piece management
│   ├── ee/                  # Enterprise Edition features
│   │   ├── shared/          # EE shared types
│   │   └── ui/              # EE UI components
│   └── tests-e2e/           # Playwright E2E tests
├── docs/                    # Documentation (Mintlify)
├── deploy/                  # Deployment configurations
│   └── pulumi/              # Pulumi infrastructure
└── tools/                   # Build scripts and utilities
```

## Key Entry Points

| Component | Entry Point |
|-----------|-------------|
| Backend API | `packages/server/api/src/main.ts` |
| Frontend | `packages/react-ui/src/main.tsx` |
| Engine | `packages/engine/src/main.ts` |
| CLI | `packages/cli/src/index.ts` |

## Technology Stack

### Backend
- **Framework**: Fastify 5.4.0
- **ORM**: TypeORM 0.3.26
- **Database**: PostgreSQL 14+ (primary), PGLite, SQLite
- **Cache/Queue**: Redis + BullMQ
- **Logging**: Pino
- **WebSockets**: Socket.io
- **Validation**: TypeBox (Sinclair)

### Frontend
- **Framework**: React 18.3.1
- **Build Tool**: Vite 6.4.1
- **Styling**: TailwindCSS 4.1.17
- **Routing**: React Router 6
- **State/Data**: React Query (TanStack)
- **Components**: Radix UI

### Infrastructure
- **Monorepo**: Nx 22.0.1
- **Package Manager**: Bun
- **Language**: TypeScript 5.5.4
- **Testing**: Jest 30 + Playwright
- **Linting**: ESLint 8.57.0
- **Formatting**: Prettier

## Development Commands

### Starting Development

```bash
# Full stack (Frontend + Backend + Engine)
npm run dev

# Backend only (API + Engine)
npm run dev:backend

# Frontend only
npm run dev:frontend

# Individual services
npm run serve:frontend    # Port 4300
npm run serve:backend
npm run serve:engine
```

### Piece Development

```bash
# Create a new piece
npm run create-piece

# Add action to existing piece
npm run create-action

# Add trigger to existing piece
npm run create-trigger

# Build a specific piece
npm run build-piece

# Sync pieces with API
npm run sync-pieces
```

### Testing

```bash
# Run all server tests
nx test server-api

# Community Edition tests
nx test-ce server-api

# Enterprise Edition tests
nx test-ee server-api

# Cloud Edition tests
nx test-cloud server-api

# End-to-end tests
npm run test:e2e
```

### Code Quality

```bash
# Lint all projects
npx nx run-many --target=lint

# Lint and fix
npx nx run-many --target=lint --fix

# Lint + push (convenience script)
npm run push
```

### Database Operations

```bash
# Generate migration
nx db server-api -- migration:generate -p ./src/app/database/migration/postgres

# Run migrations
nx db-migration server-api --name=migration_name

# Check migrations
nx check-migrations server-api
```

## Code Architecture Patterns

### Backend Module Pattern

Features are organized as Fastify plugins:

```typescript
// feature.module.ts
export const featureModule: FastifyPluginAsyncTypebox = async (app) => {
    await app.register(featureController, { prefix: '/v1/feature' })
}

// feature.controller.ts
export const featureController: FastifyPluginAsyncTypebox = async (app) => {
    app.get('/', GetFeatureRequest, async (request, reply) => {
        return featureService.get(request.params.id)
    })
}

// feature.service.ts
export const featureService = {
    async get(id: string) {
        return repo.findOne({ where: { id } })
    }
}
```

### Directory Structure for Features

```
feature/
├── feature.module.ts       # Fastify plugin registration
├── feature.controller.ts   # Route handlers
├── feature.service.ts      # Business logic
├── feature.entity.ts       # TypeORM entity
└── test/
    └── feature.test.ts     # Tests
```

### Piece Structure

```
pieces/community/piece-name/
├── src/
│   ├── index.ts            # Piece definition (auth, actions, triggers)
│   └── lib/
│       ├── actions/        # Action implementations
│       ├── trigger/        # Trigger implementations
│       └── common/         # Shared utilities
├── package.json
└── project.json            # Nx configuration
```

### Piece Definition Example

```typescript
import { createPiece, PieceAuth } from '@activepieces/pieces-framework';

export const myPiece = createPiece({
    displayName: 'My Piece',
    auth: PieceAuth.SecretText({
        displayName: 'API Key',
        required: true,
    }),
    minimumSupportedRelease: '0.20.0',
    logoUrl: 'https://...',
    authors: ['author'],
    actions: [myAction],
    triggers: [myTrigger],
});
```

## Environment Variables

### Required for Development

```bash
# Database
AP_DB_TYPE=POSTGRES              # POSTGRES, PGLITE, SQLITE
AP_POSTGRES_HOST=localhost
AP_POSTGRES_PORT=5432
AP_POSTGRES_DATABASE=activepieces
AP_POSTGRES_USERNAME=postgres
AP_POSTGRES_PASSWORD=password

# Server
AP_FRONTEND_URL=http://localhost:4200
AP_JWT_SECRET=your-secret-key
AP_ENCRYPTION_KEY=32-char-hex-key

# Redis
AP_REDIS_HOST=localhost
AP_REDIS_PORT=6379

# Development
AP_ENVIRONMENT=dev
AP_DEV_PIECES=piece1,piece2      # Pieces to load in dev mode
```

### Edition Configuration

```bash
AP_EDITION=COMMUNITY             # COMMUNITY, ENTERPRISE, CLOUD
```

## Testing Guidelines

### Test File Naming
- Unit tests: `*.spec.ts` or `*.test.ts`
- Integration tests: Located in `test/integration/`

### Test Organization
```
packages/server/api/test/integration/
├── ce/      # Community Edition tests
├── ee/      # Enterprise Edition tests
└── cloud/   # Cloud Edition tests
```

### Writing Tests

```typescript
describe('Feature', () => {
    beforeAll(async () => {
        // Setup database, create test data
    });

    afterAll(async () => {
        // Cleanup
    });

    it('should do something', async () => {
        // Test implementation
    });
});
```

## Important Conventions

### TypeScript
- Strict mode enabled
- Use TypeBox for API validation schemas
- Shared types go in `packages/shared`

### Git Workflow
- Conventional commits enforced (`feat:`, `fix:`, `chore:`, etc.)
- No direct pushes to main branch (protected by hook)
- `.env` files cannot be committed (protected by hook)

### Code Style
- Single quotes (Prettier)
- ESLint with Nx module boundary enforcement
- No lodash imports (restricted by ESLint)

### API Design
- RESTful endpoints
- Version prefix: `/v1/`
- TypeBox for request/response validation
- OpenAPI/Swagger documentation generated

### Error Handling

```typescript
import { ActivepiecesError, ErrorCode } from '@activepieces/shared';

throw new ActivepiecesError({
    code: ErrorCode.ENTITY_NOT_FOUND,
    params: { entityType: 'flow', entityId: id },
});
```

## Database Entities

Key entities (111+ total):

| Entity | Description |
|--------|-------------|
| `Flow` | Workflow definition |
| `FlowVersion` | Versioned flow configuration |
| `FlowRun` | Execution instance |
| `Project` | Workspace container |
| `User` | User account |
| `AppConnection` | OAuth/API credentials |
| `Piece` | Integration metadata |
| `Trigger` | Flow trigger configuration |
| `Table` | Custom data tables |

## Execution Modes

```bash
AP_EXECUTION_MODE=UNSANDBOXED    # Direct execution (dev)
AP_EXECUTION_MODE=SANDBOX_PROCESS # Process isolation
```

## Path Aliases

Defined in `tsconfig.base.json`:

```typescript
import { ... } from '@activepieces/shared';
import { ... } from '@activepieces/server-shared';
import { ... } from '@activepieces/pieces-framework';
import { ... } from '@activepieces/pieces-common';
import { ... } from '@activepieces/ee-shared';
```

## Documentation

- **Docs location**: `/docs/`
- **Engineering handbook**: `/docs/handbook/engineering/`
- **Piece development**: `/docs/build-pieces/`
- **API reference**: `/docs/endpoints/`

## Docker

### Development

```bash
docker-compose -f docker-compose.dev.yml up
```

### Production

```bash
docker-compose up -d
```

### Build

```bash
docker build -t activepieces .
```

## Common Tasks for AI Assistants

### Adding a New Piece

1. Run `npm run create-piece`
2. Implement authentication in `src/index.ts`
3. Add actions in `src/lib/actions/`
4. Add triggers in `src/lib/trigger/`
5. Test with `AP_DEV_PIECES=your-piece npm run dev`

### Adding a Backend Feature

1. Create module in `packages/server/api/src/app/`
2. Define entity if needed
3. Create controller and service
4. Register module in `app.ts`
5. Add tests in `test/integration/`

### Adding a Frontend Feature

1. Create feature folder in `packages/react-ui/src/features/`
2. Add components, hooks, and API calls
3. Register routes in app router
4. Use existing UI components from `src/components/`

### Debugging

- Backend logs: Pino (structured JSON)
- Frontend: Browser DevTools
- Database: TypeORM logging via `AP_LOG_LEVEL=debug`

## Useful File References

| Purpose | File Path |
|---------|-----------|
| App setup | `packages/server/api/src/app/app.ts` |
| Server config | `packages/server/api/src/app/server.ts` |
| Database setup | `packages/server/api/src/app/database/database-connection.ts` |
| System properties | `packages/server/api/src/app/helper/system/system.ts` |
| Shared types | `packages/shared/src/lib/` |
| Piece framework | `packages/pieces/community/framework/` |
| React app shell | `packages/react-ui/src/app/` |
| E2E tests | `packages/tests-e2e/` |

## Notes for AI Assistants

1. **Always check existing patterns** before implementing new features
2. **Use TypeBox** for API validation, not raw TypeScript types
3. **Follow module structure** for backend features
4. **Run tests** after making changes: `nx test <project>`
5. **Check ESLint** before committing: `npx nx run-many --target=lint`
6. **Use shared types** from `@activepieces/shared`
7. **Respect edition boundaries** (CE vs EE vs Cloud)
8. **Document new pieces** with proper `displayName` and `description`
