# TribeCRM Workspace Guide for Claude Code

**Last Updated**: 2026-01-26 | **Workspace**: C:\git | **Projects**: 4 (API, App, Auth, Tests)

## Quick Reference

## Process documentation



### Projects & URLs
```
C:\git\
├── tribecrm-api\              # Backend (Java/Spring Boot)
├── tribecrm-app\              # Frontend (React/TypeScript)
├── tribecrm-authentication\   # OAuth/SSO (Node.js)
└── tribecrm-test-automation-qa\ # E2E Tests (Cypress)
```

| Env | Frontend | Backend | Auth |
|-----|----------|---------|------|
| Local | localhost:3000 | localhost:8080 | localhost:2000 |
| Staging | app-staging.tribecrm.nl | api-staging.tribecrm.nl | auth-staging.tribecrm.nl |
| Staging (RSBuild) | app-rsbuild.tribecrm.nl | backend-staging.tribecrm.nl | auth-staging.tribecrm.nl |
| Production | app.tribecrm.nl | api.tribecrm.nl | auth.tribecrm.nl |
| Beta | beta.tribecrm.nl | backend.tribecrm.nl | auth.tribecrm.nl |

### Quick Commands
```bash
# Backend: mvn clean install, mvn test, mvn verify, mvn spring-boot:run -Dspring-boot.run.profiles=local
# Frontend: npm install, npm run start.development, npm run start.development.staging, npm run build.staging
# Auth: npm install, cd docker && docker-compose up -d, cd .. && npm run start.development
# Tests: npx cypress open --env envName=fe-stage, npx cypress run --env envName=fe-stage,grepTags=@smoke --browser chrome
```

## Workspace Overview

**TribeCRM**: Multi-tenant CRM with dynamic entity system enabling runtime schema customization without code deployment.

### Technology Stack
| Project | Lang | Framework | Database | Key Tech |
|---------|------|-----------|----------|----------|
| API | Java 17 | Spring Boot 2.4.2 | PostgreSQL/MySQL | Hibernate 5.4, QueryDSL 5.0, OData 4.10, GCP, Flyway 8.5 |
| App | TS 5.9 | React 17 | - | MobX 4.15, MUI 5.17, RSBuild 1.3, Jest 29 |
| Auth | JS | Express 4.17 | PostgreSQL | ORY Hydra 1.5.2, Passport 0.4, SAML, OIDC |
| Tests | JS | Cypress 15.8.2 | - | @bahmutov/cy-grep, cypress-plugin-api |

### Key Features
- **Dynamic Entity System**: Entity types in DB, not code
- **Multi-Tenant**: Org-based isolation
- **OData 4.10**: Flexible API querying
- **OAuth2/SSO**: SAML 2.0, OIDC, 2FA
- **Real-Time**: GCP Pub/Sub events
- **Feature Flags**: Flagsmith integration
- **AI Features**: Assistant, sentiment analysis

## Architecture

### Dynamic Entity System (Core)
**Six Core Tables** ("Holy Trinity" × 2):
- **Metadata**: `entity_type`, `entity_field`, `entity_relationship_definition`
- **Data**: `entity`, `entity_value`, `entity_relationship`

**Benefits**: Runtime schema changes, tenant customization, dynamic forms, type-safe abstraction

### Architecture Diagram
```
USER ──► Frontend (React:3000) ──► Backend (Spring:8080) ──► DB (PostgreSQL/MySQL)
              │                          │
              └──► Auth (OAuth:2000) ────┘
                   (ORY Hydra)
         Tests (Cypress) ──► All Services
```

### Communication
- Frontend ↔ Backend: REST + OData 4.10
- Frontend ↔ Auth: OAuth2 Authorization Code Flow
- Backend ↔ DB: JDBC + HikariCP (dual: MySQL truth, PostgreSQL query)
- Backend ↔ GCP: Pub/Sub, Storage, Vertex AI
- Tests: Cypress E2E, Direct HTTP API

### Pack System
- **Pack.System** (497): Core TRIBE metadata (read-only)
- **Pack.Application** (500): Optional modules
- **Pack.Environment** (503): Tenant pack (read/write)
- **Pack.User** (568): Per-user data

## Project Details

### 1. tribecrm-api (Backend)
**Location**: C:\git\tribecrm-api
**Purpose**: Multi-tenant Spring Boot CRM with dynamic entities, OData, GCP integrations

**Key Tech**: Spring Boot 2.4.2, Hibernate 5.4.27, QueryDSL 5.0, OData 4.10, GCP BOM 26.40.0, Flyway 8.5.13, Flagsmith 7.4.3, JUnit 5.11.4, Mockito 5.2.0, Cucumber 7.21.1, Testcontainers 1.20.4

**Source Structure**:
```
src/main/java/com/perfectview/api/
├── entity/         # Dynamic entity system (18 subdirs): model, repository, metadata, mutation, query, action, commit, automation, bespoke
├── odata/          # OData controller
├── connector/      # 40+ integrations
├── migration/      # YAML migrations
├── pubsub/         # GCP Pub/Sub
├── ai/assistant/   # AI features
├── flagsmith/      # Feature flags
├── security/       # Auth & rights
└── [88 modules]    # Email, geocoding, file handling
```

**Key Patterns**:
```java
// ALWAYS use EntityFactory
Entity entity = entityFactory.create(entityType, organization);

// ApiContext thread-local
Organization org = apiContext.getOrganization();
RightProfile rights = apiContext.getRightProfile();

// Query DSL
EntitySelectionBuilder builder = new EntitySelectionBuilder(entityType)
    .withConstraints(Constraint.equals("status", "active"))
    .withFields("name", "email")
    .withOrderBy("name", OrderDirection.ASC);
List<Entity> results = selectionExecutor.execute(builder.build());

// Mutation Pipeline
EntityMutationService.mutate(EntityMutationRequest.builder()
    .entity(entity).mutationType(MutationType.CREATE).build());
```

**Testing**:
- Unit: JUnit 5 + Mockito, MetadataMock (483+ entity types, 1,189+ fields)
- Integration: Cucumber 7.21.1, Testcontainers (6 features)

**Docs**: `.ai-support/docs/` (tribe-architecture.md, tribe-data-model.md, tribe-query-system.md, tribe-migration-system.md)

### 2. tribecrm-app (Frontend)
**Location**: C:\git\tribecrm-app
**Purpose**: React SPA with real-time collaboration, dynamic forms, Material-UI

**Key Tech**: React 17.0.2, TypeScript 5.9.3, MobX 4.15.7, MUI 5.17.1, RSBuild 1.3.3, Node 22+, ESLint 9.37.0, Flagsmith 4.1.3, Sentry, Amplitude

**Source Structure**:
```
src/
├── @Api/          # API client, models, metadata
├── @Component/    # App, Domain (40+), Generic (60+), Service
├── @Framework/    # Data, Store, Component
├── @Service/      # ApiClient, Auth, Navigation, Localization
├── @Theme/        # MUI theme
└── @UILibrary/    # Reusable UI
```

**Patterns**:
```typescript
// Component
export const ComponentName = observer(() => (
    <div className={styles.container}>...</div>
));

// MobX Store
class EntityStore {
    @observable entities: Entity[] = [];
    @action async loadEntities() {
        this.loading = true;
        this.entities = await apiClient.getEntities();
        this.loading = false;
    }
}
```

**Local Dev vs Staging**:
1. Edit hosts: `C:\Windows\System32\drivers\etc\hosts` → `127.0.0.1 dev.tribe`
2. Run: `npm run start.development.staging`
3. Access: `https://dev.tribe:3000`

**Config**: 16 .env files (development, staging, production, beta, dev-specific), rsbuild.config.ts, eslint.config.js

### 3. tribecrm-authentication (OAuth)
**Location**: C:\git\tribecrm-authentication
**Purpose**: OAuth2/OIDC with SSO (SAML, OIDC), 2FA, portal theming via ORY Hydra

**Key Tech**: Express 4.17.1, Node 10, ORY Hydra 1.5.2, Passport 0.4.0, passport-saml/oidc, React 16.9 Login UI

**Structure**:
```
├── app.js, bin/www        # Express entry (port 2000)
├── routes/                # login (2FA), logout, consent, strategy, portal-theme, sso-test-connection
├── strategy/              # getAuthenticationStrategy, hasValidLicense
├── services/hydra.js      # Hydra Admin API
├── config/strategy/       # local, saml, openidconnect
└── docker/                # Hydra + PostgreSQL
```

**Auth Flow**: App → `/login` → Strategy Detection → Auth (Default/SAML/OIDC) → Validate → License check → Hydra tokens → Redirect

**New Features**: 2FA (TOTP), Portal theming, SSO test, Multi-language, Blocked user prevention

**Env Vars**: `HYDRA_ADMIN_URL`, `AUTH_BASE_URL`, `API_CREDENTIALS_VALIDATION_ENDPOINT`, `API_LOGIN_STRATEGY_ENDPOINT`, `API_LICENSE_VALIDATION_ENDPOINT`, `SECURE_COOKIES`

### 4. tribecrm-test-automation-qa (Cypress)
**Location**: C:\git\tribecrm-test-automation-qa
**Purpose**: E2E & API tests with parallel execution, Cypress Cloud integration

**Structure**:
```
cypress/
├── configs/       # fe-local, fe-stage, fe-prod
├── data/          # envSpecificData.js
├── support/       # 95 custom commands, selectors, fieldUtils, apiUtils
└── tests/         # 18 files: api/oDataAPI, e2e/activities, e2e/integrations, e2e/relations
```

**Tags**: `@sanity` (critical), `@smoke` (quick), `@locale-agnostic`

**Custom Commands**: `cy.loginToTribe()`, `cy.fillTextField()`, `cy.addActivityOrPersonOrOrganization()`, `cy.waitUntilLoaderDisappears()`

**Run**:
```bash
npx cypress open --env envName=fe-stage
npx cypress run --env envName=fe-stage,grepTags=@smoke --browser chrome
```

**CI/CD**: GitHub Actions (3 workflows), GCP Cloud Build (4 pipelines), Cypress Cloud (4 parallel runners)

## Development Workflows

### Git Workflow (MANDATORY)
**Branch**: `<JIRA-TICKET>_<description>` (e.g., TRIBE-1234_add-person-entity)
**Commit**: `<JIRA-TICKET>: <type>(<scope>): <summary>` (e.g., TRIBE-1234: feat(entity): add Person entity)
**Types**: feat, fix, refactor, test, docs, chore

**Workflow**: Create branch → Commit → Push → PR → Attach changelog to Jira (NEVER commit) → Merge → Delete branch

### Local Setup
**Backend**: Java 17, Maven, PostgreSQL → `mvn clean install -DskipTests` → `mvn spring-boot:run -Dspring-boot.run.profiles=local`
**Frontend**: Node 22+ → `npm install` → `npm run start.development` or `npm run start.development.staging`
**Auth**: Node, Docker → `npm install` → `cd docker && docker-compose up -d` → `npm run start.development`

### Building
**Backend**: `mvn clean install`, `mvn -T 1C clean install` (parallel)
**Frontend**: `npm run build.staging`, `npm run build.production`

## Testing

### Backend
- Unit: `mvn test`, `mvn test -Dtest=EntityServiceTest`, `mvn test -Pcoverage`
- Integration: `mvn verify`, `mvn verify -DskipUnitTests`
- MetadataMock: 483+ entity types available

### Frontend
- Unit: `npm test`, `npm test -- --coverage` (minimal coverage)

### E2E (Cypress)
- Interactive: `npx cypress open --env envName=fe-stage`
- Headless: `npx cypress run --env envName=fe-stage --browser chrome`
- Tags: `npx cypress run --env envName=fe-stage,grepTags="@sanity+@locale-agnostic" --browser chrome`

### Coverage
| Project | Unit | Integration | E2E | Coverage |
|---------|------|-------------|-----|----------|
| API | ✅ Extensive | ✅ Cucumber | - | 60-70% |
| App | ⚠️ Minimal | - | External | Low |
| Auth | ❌ None | External | External | Unknown |
| Tests | - | - | ✅ 18 files | ~80% |

## Deployment

**Targets**: API/Auth (GKE europe-west4), Frontend (App Engine)

**Backend**: `gcloud builds submit --config=cloudbuild.staging.yaml` → Maven → Docker → GCR → GKE
**Frontend**: `npm run build.staging` → Docker (Nginx) → GCR → App Engine
**Auth**: PostgreSQL → Hydra → Express service

**Rollback**:
- Backend (GKE): `kubectl rollout undo deployment/api-deployment -n tribecrm-staging`
- Frontend (App Engine): `gcloud app services set-traffic default --splits <version>=1`

**Health**:
- Backend: `/actuator/health/liveness`, `/actuator/health/readiness`
- Frontend: `/` (200 OK)

## Conventions & Best Practices

### Code Style
**Backend**: Spring Boot conventions, Lombok, ALWAYS EntityFactory, ApiContext scoping
**Frontend**: Functional components + hooks, MobX observer, CSS modules, strict TypeScript, ESLint rules
**Auth**: Express conventions, Passport strategies, subject encoding (user+org+portal)

### Testing
**Unit**: AAA pattern, descriptive names (`should_createPerson_when_validData`), MetadataMock
**E2E**: data-test-id selectors, cleanup in `after()`, unique names with `Date.now()`, `cy.waitUntilLoaderDisappears()`

### Security
- NEVER log sensitive data
- Validate all user input
- Parameterized queries (SQL injection)
- HTTPS for all external comms
- HttpOnly cookies, CSRF protection

## Troubleshooting

**Backend**:
- DB connection fail: Check PostgreSQL running, verify `application-local.properties`
- OOM: `export MAVEN_OPTS="-Xmx2g"`
- OData issues: Check `/odata/$metadata`, SelectionBuilder constraints

**Frontend**:
- dev.tribe not resolving: Edit hosts file, flush DNS (`ipconfig /flushdns`)
- RSBuild SSL error: Check `rsbuild.config.ts`, certificates (10-day validity)
- 401 Unauthorized: Check token expiration, backend URL in `.env`, CORS

**Tests**:
- Timeout: Increase `defaultCommandTimeout` (25s), use `cy.waitUntilLoaderDisappears()`
- CI failures: Check viewport (1600x900), test cleanup, env vars

**Auth**:
- Login error: Check Hydra running (`docker ps`), OAuth client config, callback URLs
- SSO fail: Verify SAML/OIDC config, certificate validity, test `/sso/test-connection`
- 2FA fail: Check `API_TWO_FACTOR_CODE_EXPIRATION_IN_SECONDS_ENDPOINT`, TOTP secret, time sync

## Key File Locations

**Config**:
- Backend: `pom.xml`, `application.properties`, `application-{env}.properties`
- Frontend: `package.json`, `rsbuild.config.ts`, `.env.{environment}`
- Auth: `package.json`, `.env`, `docker/docker-compose.yml`
- Tests: `package.json`, `cypress.config.js`, `cypress/configs/fe-{env}.json`

**Docs**:
- Workspace: `.github/copilot-instructions.md`
- Backend: `README.md`, `CLAUDE.md`, `.ai-support/docs/*.md` (8 files)
- Frontend: `README.md`, `rsbuild.md`, `Typography.md`
- Auth: `README.md`
- Tests: `README.md`, `.github/prompts/generate-e2e-tests.prompt.md`

**Core Source**:
- Backend: `entity/model/Entity.java`, `entity/EntityFactory.java`, `entity/query/SelectionBuilder.java`, `odata/ODataController.java`
- Frontend: `@Api/Api/ApiClient.ts`, `@Api/Metadata/MetadataService.ts`
- Auth: `app.js`, `routes/login.js`, `services/hydra.js`
- Tests: `cypress/support/commands.js`, `cypress/support/fieldUtils.js`

## Session Persistence

**Conversation Logs**: `.ai-support/temp/conversation-log-YYYY-MM-DD-<task>.md`
**Changelogs**: `.ai-support/changelogs/JIRA-TICKET-changelog.md` (attach to Jira, NEVER commit)
**Analysis**: `.ai-support/analysis/<topic>.md`

## Useful Links
- Staging: app-staging.tribecrm.nl, api-staging.tribecrm.nl, app-rsbuild.tribecrm.nl
- Production: app.tribecrm.nl, api.tribecrm.nl
- Beta: beta.tribecrm.nl
- Cypress Cloud: cloud.cypress.io/projects/sow3it/runs

---
**Version 1.2** (2026-01-26): Minified version retaining all critical information