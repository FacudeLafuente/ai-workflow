# Skill overlay: repo-viability-scan — BE profile

> Overlay applied on top of [`repo-viability-scan.md`](repo-viability-scan.md)
> when the active profile is `be`.

## Autodetection hints (BE)

The base skill autodetects `be` when it finds any of:

- **.NET:** `*.sln`, `*.csproj`, `Program.cs`, `Startup.cs`.
- **Java:** `pom.xml`, `build.gradle`, `settings.gradle`, `src/main/java/**`.
- **Node.js server:** `package.json` + `express|fastify|@nestjs/*|koa|hapi`
  (and no FE framework).
- **Python:** `pyproject.toml`, `requirements*.txt`, `setup.py`,
  `django|flask|fastapi|starlette` imports.
- **Go:** `go.mod`, `main.go`.
- **Rust:** `Cargo.toml`, `src/main.rs`.
- **Ruby:** `Gemfile`, `config.ru`.
- **PHP:** `composer.json`, `artisan`, `symfony.lock`.

If both FE and BE stacks are present in the repo, the base skill halts and
asks the human to declare the profile.

## Touchpoint map to scan (BE)

Scan these concerns (adjust to your repo's declared layout in `AGENTS.md`):

- **Controllers / handlers / routers** — `Controllers/`, `handlers/`,
  `routes/`, per-endpoint files.
- **Services** — `<Core|Services|Application>/Services/`,
  `<domain>/services/`, use-case classes.
- **Interfaces** — `<Core|Application>/Interfaces/` (declared abstraction
  boundary).
- **Repositories** — `<DAL|Persistence|Infrastructure>/Repositories/`,
  `dao/`, `stores/`.
- **DbContext / ORM entry point** — EF Core `DbContext`, JPA `EntityManager`,
  SQLAlchemy `Session`, Prisma client, etc.
- **DTOs / models:**
  - Request/response DTOs (Web-layer).
  - Domain DTOs (application layer).
  - Persistence entities / models.
- **Mappers** — AutoMapper profiles (`ApiProfile.cs`), MapStruct mappers,
  manual `mapTo…()` functions.
- **Global exception / error handler** — `GlobalExceptionHandler.cs`,
  `@ControllerAdvice`, Express error middleware, FastAPI exception handlers.
- **Auth policies / guards** — `Auth/Policies/*` + `PolicyNames.cs`;
  `@PreAuthorize`; NestJS Guards; Passport strategies; middleware.
- **OpenAPI / Swagger** — `AddSwaggerGen`, `springdoc`, `drf-yasg`, Nest
  `@ApiTags`, decorators declaring response types.
- **Migrations** — `Migrations/`, `db/migrate/`, `alembic/`, dbt models,
  Liquibase `changelog.xml`. **If no migrations folder exists**, treat any
  schema change as `RED` and escalate.
- **Configuration files** — `appsettings.json`, `application.yml`, `.env`,
  Vault references.
- **Testing projects** — `*.Tests/`, `src/test/`, `tests/`, harness
  (`xUnit`, `JUnit`, `pytest`, `Jest`).
- **CI pipelines** — `.pipelines/`, `.github/workflows/`, `.gitlab-ci.yml`
  (read-only unless the ticket touches CI explicitly).

## Landmines (BE)

Explicitly review these repo-wide rules from `AGENTS.md` against the change:

- **Layering discipline** — Controller → Service → Repository (or the
  repo's variant). No DB calls from controllers. No business logic in
  controllers.
- **Uniform error envelope** — errors flow through the declared global
  handler; ad-hoc `return BadRequest(...)` with bespoke shapes is forbidden.
  If the change requires a new mapping (new exception type → new status
  code), that is a **cross-cutting change** and raises the rating.
- **Auth discipline** — every non-public endpoint carries the declared auth
  annotation / guard; no relaxation of an existing policy.
- **Response-type metadata** — every documented status code is declared.
- **DTO placement** — Web-layer DTOs vs Domain DTOs vs Entities live in the
  layers declared by `AGENTS.md`. Cross-layer leaks raise the rating.
- **Mapping discipline** — additions to the declared mapping layer
  (`ApiProfile` / MapStruct / manual mapper) not ad-hoc mapping in
  controllers.
- **Kebab-case / routing style** — action names respect the declared style
  (typical: kebab-case URL segments enforced by a transformer / attribute).
- **Migrations flow** — schema changes without a confirmed flow are `RED`.
- **Breaking API change** on a shipped route — always `RED`; requires a
  versioning strategy.
- **Idempotency** — non-idempotent methods that must be idempotent per the
  ticket declare their strategy.

## Viability Report additions (BE)

Add these rows to the Touchpoints table in the base template:

| Layer      | Path (examples)                                             |
|------------|-------------------------------------------------------------|
| Controller | `Controllers/<...>Controller.<ext>`                          |
| Service    | `<Core>/Services/<...>Service.<ext>`                         |
| Interface  | `<Core>/Interfaces/I<...>Service.<ext>`                      |
| Repository | `<DAL>/Repositories/<...>Repository.<ext>`                   |
| DTO (Web)  | `DTOs/Requests|Responses/<...>.<ext>`                        |
| DTO (Core) | `<Core>/DTOs/<...>.<ext>`                                    |
| Mapping    | `Mapper/ApiProfile.<ext>` (or MapStruct / manual)            |
| Auth       | `Auth/Policies/PolicyNames.<ext>` (or `@PreAuthorize` roots) |
| Errors     | `Handlers/GlobalExceptionHandler.<ext>`                      |
| Entity     | `<Core>/Entities/<...>.<ext>`                                |
| DbContext  | `<DAL>/Data/<...>DbContext.<ext>`                            |
| Migrations | `<DAL>/Migrations/` (or missing → RED for schema changes)    |

## Additional rubric rows (BE)

Merge into the base rubric of `repo-viability-scan.md`:

| # | Check                         | Pass criterion                                                                                              |
|---|-------------------------------|-------------------------------------------------------------------------------------------------------------|
| B1 | Layering respected            | Every touchpoint sits in the layer declared by `AGENTS.md`; no cross-layer leaks proposed.                   |
| B2 | Uniform error envelope        | Every documented error status is reachable via a known exception; new mappings flagged as cross-cutting.    |
| B3 | Auth policy sweep             | Every non-public endpoint names an auth annotation / guard.                                                 |
| B4 | Response-type metadata sweep  | Every documented status code is declared via the repo's metadata mechanism.                                 |
| B5 | Migrations rule               | Any schema change is either backed by a confirmed migrations decision or rated `RED`.                       |
| B6 | Breaking-change rule          | Any breaking API change on a shipped route is `RED` until a versioning strategy is recorded.                |

## Tools / MCP hooks (BE)

- Repository search / read.
- **OpenAPI MCP** (optional / roadmap) — cross-check the existing contract at
  `/swagger/v1/swagger.json` (or equivalent).
- Language servers (Roslyn, JDT, TypeScript, Pyright) for symbol references.
