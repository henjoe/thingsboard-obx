# Copilot Instructions — ThingsBoard (ObX Fork)

## Project Overview

ThingsBoard is an open-source IoT platform for data collection, processing, visualization, and device management. This is a multi-module Maven project (version 4.4.0-SNAPSHOT) licensed under Apache License 2.0. The platform supports multi-tenancy, rule engine processing, multiple transport protocols, and edge computing.

## Tech Stack

### Backend
- **Java 17** with **Spring Boot 3.4.10**
- **Maven** multi-module build system
- **Lombok** for boilerplate reduction (see `lombok.config` — `stopBubbling=true`, `addConstructorProperties=true`)
- **PostgreSQL** (primary) and **Cassandra** (NoSQL alternative) for persistence
- **Spring Data JPA / Hibernate** for ORM (`dao/` module)
- **Protobuf 3.25.5** and **gRPC 1.76.0** for inter-service communication (`common/proto/`)
- **Apache Kafka 3.9.1** as the default message queue
- **Valkey** (Redis-compatible) for caching
- **Jackson** for JSON serialization (`JacksonUtil` utility class)
- **Guava** `ListenableFuture` for async operations (not `CompletableFuture`)
- **Swagger / SpringDoc OpenAPI** for REST API documentation

### Frontend (`ui-ngx/`)
- **Angular 20** with **TypeScript 5.9**
- **Angular Material 20** and **Angular CDK**
- **NgRx** for state management (`@ngrx/store`, `@ngrx/effects`)
- **TailwindCSS 3** for utility-first styling
- **ESLint** with `angular-eslint` and `tailwindcss` plugins
- **Yarn** as package manager
- **Ace Editor**, **ECharts**, **Leaflet**, **TinyMCE** for specialized UI components

## Module Structure

```
thingsboard-obx/
├── application/        # Main Spring Boot application, REST controllers, actor system
├── common/             # Shared libraries
│   ├── actor/          # Actor system framework
│   ├── cache/          # Cache abstractions (Valkey/Redis)
│   ├── cluster-api/    # Cluster communication (protobuf-generated)
│   ├── coap-server/    # CoAP server core
│   ├── dao-api/        # DAO interfaces and service contracts
│   ├── data/           # Domain model / data classes (POJOs, IDs, enums)
│   ├── edge-api/       # Edge communication (protobuf-generated)
│   ├── edqs/           # Entity Data Query Service shared code
│   ├── message/        # Internal message types
│   ├── proto/          # Protobuf definitions (transport.proto, queue.proto, jsinvoke.proto)
│   ├── queue/          # Message queue abstractions (Kafka, in-memory, etc.)
│   ├── script/         # Script engine (TBEL, JS)
│   ├── transport/      # Transport protocol common code (API, MQTT, CoAP, HTTP, LwM2M, SNMP)
│   ├── util/           # Common utilities
│   └── version-control/# Version control (Git-based entity export/import)
├── dao/                # Data access layer (JPA entities, repositories, SQL)
├── edqs/               # Entity Data Query Service (standalone)
├── msa/                # Microservices architecture (Docker images, JS executor, web UI)
├── netty-mqtt/         # Custom Netty-based MQTT codec
├── rest-client/        # Java REST client SDK
├── rule-engine/        # Rule engine API and built-in rule nodes
│   ├── rule-engine-api/
│   └── rule-engine-components/
├── tools/              # CLI and migration tools
├── transport/          # Standalone transport microservices
│   ├── coap/
│   ├── http/
│   ├── lwm2m/
│   ├── mqtt/
│   └── snmp/
├── ui-ngx/             # Angular frontend application
└── monitoring/         # Platform monitoring service
```

## Code Conventions

### Java / Backend

- **Package root**: `org.thingsboard.server`
  - Controllers: `org.thingsboard.server.controller`
  - Services: `org.thingsboard.server.dao.<entity>` (interfaces + implementations)
  - Domain models: `org.thingsboard.server.common.data`
  - Entity IDs: `org.thingsboard.server.common.data.id`
  - SQL entities: `org.thingsboard.server.dao.model.sql`
  - SQL DAOs: `org.thingsboard.server.dao.sql.<entity>`
  - Rule engine nodes: `org.thingsboard.rule.engine`

- **License Header**: Every source file must include the Apache 2.0 license header (copyright 2016-2026 The Thingsboard Authors). Use `mvn license:format` to enforce.

- **Lombok Usage**: Use `@Slf4j`, `@RequiredArgsConstructor`, `@Getter`, `@Setter`, `@EqualsAndHashCode`, `@ToString`, `@Data` as appropriate. Prefer `@RequiredArgsConstructor` with `final` fields over `@Autowired` on fields for dependency injection (controllers follow this pattern).

- **Controller Pattern**:
  - Extend `BaseController` for access to common helper methods
  - Annotate with `@RestController`, `@RequestMapping("/api")`, and `@TbCoreComponent`
  - Use `@PreAuthorize` for security, `@ApiOperation` for Swagger docs
  - Use Swagger v3 annotations (`io.swagger.v3.oas.annotations`)

- **Service Layer**:
  - Service interfaces in `common/dao-api/` module
  - Implementations in `dao/` module, annotated with `@Service`
  - Use `@Transactional` for write operations
  - Use `DataValidator<T>` for entity validation before persistence
  - Event sourcing via `SaveEntityEvent` / `DeleteEntityEvent`

- **DAO Layer**:
  - DAOs extend `JpaAbstractDao<E, D>` (entity, domain)
  - Entity models extend `BaseEntity<D>`
  - Use Spring Data `JpaRepository` interfaces
  - Annotate SQL DAOs with `@SqlDao`
  - Use `DaoUtil` for common conversions

- **ID Types**: All entity IDs are UUID-based, wrapped in typed classes (e.g., `DeviceId`, `TenantId`, `CustomerId`). They implement `EntityId` interface. Use `EntityIdFactory` for dynamic creation.

- **Async Pattern**: Use Guava's `ListenableFuture` and `DonAsynchron` utility for async callbacks. Use `DeferredResult` for async REST responses.

- **Error Handling**: Throw `ThingsboardException` with `ThingsboardErrorCode`. Controllers use `@ExceptionHandler` in `BaseController`.

- **Configuration**: Application config is in `thingsboard.yml` with environment variable overrides using `${ENV_VAR:default}` syntax.

### Angular / Frontend (`ui-ngx/`)

- **Path Aliases** (defined in `tsconfig.json`):
  - `@app/*` → `src/app/*`
  - `@core/*` → `src/app/core/*`
  - `@env/*` → `src/environments/*`
  - `@modules/*` → `src/app/modules/*`
  - `@shared/*` → `src/app/shared/*`
  - `@home/*` → `src/app/modules/home/*`

- **Component Selector Prefix**: All components must use the `tb` prefix (e.g., `<tb-device-list>`).

- **Structure**:
  - `core/` — Core services, HTTP API services, auth, guards, interceptors, WebSocket
  - `core/http/` — One service per entity type (e.g., `device.service.ts`, `alarm.service.ts`)
  - `modules/home/` — Main authenticated UI (pages, components, dialogs)
  - `modules/login/` — Authentication pages
  - `shared/` — Reusable components, directives, pipes, models

- **State Management**: NgRx store with effects. Import from `@ngrx/store` and `@ngrx/effects`.

- **Styling**: Use SCSS. TailwindCSS utility classes are available. Material theming via `src/theme/`.

- **i18n**: Use `@ngx-translate/core` with `TranslateService`. Translations use MessageFormat.

- **ESLint Rules**: No `any` in identifiers, component selector must start with `tb`, standalone components not enforced yet.

## Build & Run

### Full Build (skip tests)
```bash
MAVEN_OPTS="-Xmx1024m" NODE_OPTIONS="--max_old_space_size=4096" \
mvn -T2 license:format clean install -DskipTests
```

### Build Protobuf Packages Only
```bash
./build_proto.sh
# Builds: common/cluster-api, common/edge-api, common/transport/*
```

### Run Tests
```bash
# Set memory limits
export MAVEN_OPTS="-Xmx1024m"
export SUREFIRE_JAVA_OPTS="-Xmx1200m -Xss256k -XX:+ExitOnOutOfMemoryError"

# Unit tests (excluding heavy integration modules)
mvn test -pl='!application,!dao,!ui-ngx,!msa/js-executor,!msa/web-ui' -T4

# DAO tests with parallel packages
mvn test -pl dao -Dparallel=packages -DforkCount=4

# Application controller tests
mvn test -pl application -Dtest='!**/nosql/**,org.thingsboard.server.controller.**' \
  -DforkCount=6 -Dparallel=classes
```

### Frontend Development
```bash
cd ui-ngx
yarn install
yarn start  # Starts dev server on 0.0.0.0 with proxy
```

### Docker Compose
```bash
cd docker
docker-compose -f docker-compose.yml -f docker-compose.postgres.yml up
```

## Testing Conventions

### Backend Tests
- **Framework**: JUnit 4 with `SpringRunner`, Mockito, Awaitility
- **Base Classes**:
  - `AbstractControllerTest` → for REST controller integration tests (full Spring Boot context, random port, WebSocket support)
  - `AbstractWebTest` → provides MockMvc helpers, authentication utilities, HTTP test methods
  - `AbstractNotifyEntityTest` → for entity notification/event testing
- **Annotation**: Use `@DaoSqlTest` on test classes to configure SQL test properties
- **Mocking**: Use `@MockitoBean` / `@MockitoSpyBean` for Spring beans; use `@Primary @Bean` pattern for DAO spying
- **Test Data**: Use Testcontainers for database integration tests
- **Naming**: Test classes follow `<ClassName>Test.java` convention
- **Profiles**: Tests run with `@ActiveProfiles("test")`

### Frontend Tests
- Standard Angular testing with Jasmine/Karma (when applicable)

## Key Patterns to Follow

1. **Entity CRUD Flow**: Controller → TbEntityService (application layer) → DaoService (dao module) → JpaDao → JpaRepository
2. **Multi-tenancy**: Nearly all operations are scoped by `TenantId`. Always pass and validate `TenantId`.
3. **Pagination**: Use `PageLink` / `PageData<T>` for all list endpoints.
4. **Validation**: Use `@NoXss` and `@Length` annotations on domain model fields. Use `DataValidator<T>` in services.
5. **Caching**: Use `CachedVersionedEntityService` base class for entities that need cache support.
6. **Event Sourcing**: Publish `SaveEntityEvent` / `DeleteEntityEvent` via `@TransactionalEventListener`.
7. **Rule Engine Nodes**: Implement `TbNode` interface. Annotate with `@RuleNode`. Write corresponding `*Test.java`.
8. **Transport**: Each protocol has its own module. Shared transport API is in `common/transport/transport-api/`.
9. **Edge**: Edge-related code uses protobuf messages defined in `common/edge-api/`.
10. **Queue Abstraction**: Message queue providers are pluggable (Kafka, in-memory, etc.) via `common/queue/`.

## Important Notes

- **Kafka NetworkReceive Override**: If updating the Kafka client version, synchronize the overwritten `org.apache.kafka.common.network.NetworkReceive` class in the `application` module (see KAFKA-4090).
- **Protobuf Version**: Pinned to v3.x (not v4) due to Google Pub/Sub compatibility.
- **Testcontainers**: Requires Docker API v1.32+. If issues arise, check `~/.testcontainers.properties` and Docker's `daemon.json` for min API version.
- **Environment Variables**: All configuration supports environment variable overrides. See `thingsboard.yml` for the full list.
- **Actor System**: Device management uses an actor model (`application/src/main/java/.../actors/`). Each device has its own actor for session and RPC management.
