# feedback-360: пошаговое воссоздание проекта с нуля

## О чём этот документ

Это **практическое руководство** по воссозданию проекта feedback-360 с нуля — шаг за шагом, с примерами кода, конкретными командами и checkpoint-ами после каждого этапа.

Документ разработан как companion к аналитическому guide [`feedback-360-faithful-rebuild-guide.md`](./feedback-360-faithful-rebuild-guide.md), который объясняет "что" и "почему" на концептуальном уровне. Этот же документ отвечает на вопрос **"как именно сделать"**.

### Источники

- **Memory Bank** проекта (`.memory-bank/`) — SSoT по процессу и архитектуре
- **Git history** — 245 коммитов за 17 дней, от первого `initial commit` до hardening
- **Транскрипты автора** (`_articles/`) — объяснения подхода от автора проекта
- **Реальный код** всех packages — живые примеры паттернов

### Ключевая формула подхода

> Сначала структурируй знание, затем построй deterministic non-UI delivery path, затем выращивай домен vertical slices, затем добавь UI как thin surface, затем сделай structure и traceability сильными настолько, чтобы система оставалась понятной агентам и людям по мере роста.

### Предпосылки

- Node.js 20+
- pnpm 10+
- PostgreSQL (локальный или Docker) / Supabase
- Git
- Редактор с TypeScript support (VS Code / Cursor / подобные)

### Как пользоваться

Каждая глава заканчивается **Checkpoint** — конкретной проверкой, что всё работает. Не переходите к следующей главе, пока checkpoint не пройден.

---

## Глава 0: Методология — думай до кода

### Specs-first vs code-first

В этом проекте первые 7 коммитов — это документация: Memory Bank, спецификации, планы, шаблоны. Код появляется _после_ того, как зафиксированы:

- **Что** система делает (spec)
- **Почему** приняты ключевые решения (ADR)
- **Как** организована delivery (plans, playbook)
- **Какие правила** ведения самих документов (MBB)

Это не бюрократия. Это инвестиция в то, чтобы AI-агент (или новый разработчик) мог прочитать Memory Bank и сразу начать работать, не спрашивая "а где это?" и "а почему так?".

### Memory Bank как structured agent memory

Memory Bank — это не набор заметок. Это **структурированная база знаний**, разделённая на категории по назначению:

| Папка | Назначение | Пример |
|-------|-----------|--------|
| `spec/` | WHAT — нормативные требования | `spec/domain/campaign-lifecycle.md` |
| `plans/` | Delivery units — эпики, фичи, roadmap | `plans/epics/EP-001-core-contract-client-cli/` |
| `adr/` | WHY — ключевые решения и компромиссы | `adr/0001-core-client-cli-first.md` |
| `mbb/` | Rules — правила ведения Memory Bank | `mbb/principles.md` |
| `guides/` | Consumer docs — tutorials, how-to, reference | `guides/tutorials/run-first-360-campaign-manually.md` |

Ключевое правило: **Single Source of Truth (SSoT)**. Каждый факт имеет одно каноническое место. Нет дублей между spec и code, между plans и README.

### Vertical slices как единица delivery

Минимальная единица delivery — не "слой" (только DB, или только API), а **vertical slice**: сквозной кусок от contract до CLI/UI, который приносит проверяемую ценность.

Definition of Done для slice:
1. Contract / операция (если нужна)
2. Core use-case + policy
3. DB миграции + seed (если нужно)
4. CLI команда для вызова
5. Автотест(ы): unit / integration
6. Docs updates

### CLI-first для deterministic verification

Из транскриптов автора: _"CLI нужен как deterministic client for agents. Не потому, что UI не нужен, а потому что browser-first execution для агентов медленный и хрупкий."_

CLI позволяет:
- Проверять поведение системы **без браузера**
- Писать скрипты для seed-сценариев
- AI-агенту запускать операции и парсить JSON-ответы

### Роль AI-агентов в workflow

AI-агент в этом подходе — не "генератор кода по промпту", а участник процесса:
1. Читает Memory Bank для получения контекста
2. Получает FT-спецификацию с acceptance scenarios
3. Генерирует код по слоям (contract → core → db → client → cli)
4. Проверяет результат через CLI и тесты
5. Обновляет docs как часть slice

**Checkpoint**: Прочитайте этот раздел и убедитесь, что вы понимаете разницу между specs-first и code-first подходом. Это фундамент всего, что идёт дальше.

---

## Глава 1: Monorepo Foundation

### Почему pnpm workspaces

Монорепозиторий позволяет держать все пакеты (contract, core, client, db, cli, web) в одном месте с общим версионированием и shared tooling. pnpm выбран за скорость и строгость (phantom dependencies не проходят).

### Шаг 1.1: Инициализация

```bash
mkdir my-360 && cd my-360
git init
pnpm init
```

### Шаг 1.2: Root package.json

В реальном проекте он выглядит так:

```json
{
  "name": "feedback-360",
  "version": "0.1.0",
  "private": true,
  "packageManager": "pnpm@10.5.0",
  "scripts": {
    "lint": "pnpm -r lint",
    "typecheck": "pnpm -r typecheck",
    "test": "pnpm -r test",
    "checks": "pnpm lint && pnpm typecheck && pnpm test"
  },
  "devDependencies": {
    "@biomejs/biome": "1.9.4",
    "@types/node": "22.13.8",
    "typescript": "5.8.2",
    "vitest": "3.2.4"
  }
}
```

Ключевой script — `checks`: последовательный прогон lint, typecheck и test. Это **quality gate** — фича не считается готовой, пока `checks` не зелёный.

### Шаг 1.3: Workspace configuration

```yaml
# pnpm-workspace.yaml
packages:
  - apps/*
  - packages/*
```

Два верхнеуровневых каталога:
- `packages/*` — библиотеки (api-contract, core, client, db)
- `apps/*` — приложения (web, cli)

### Шаг 1.4: Biome — единый линтер и форматтер

Вместо ESLint + Prettier используется один инструмент:

```json
{
  "$schema": "https://biomejs.dev/schemas/1.9.4/schema.json",
  "organizeImports": { "enabled": true },
  "linter": {
    "enabled": true,
    "rules": { "recommended": true }
  },
  "formatter": {
    "enabled": true,
    "indentStyle": "space",
    "indentWidth": 2,
    "lineWidth": 100
  },
  "files": {
    "ignore": ["node_modules", "dist", ".next", "coverage"]
  }
}
```

### Шаг 1.5: TypeScript strict mode

Каждый package имеет свой `tsconfig.json`, но все наследуют strict mode. Это критично для typed contracts — без strict mode половина гарантий типов не работает.

### Шаг 1.6: Создание структуры каталогов

```bash
mkdir -p packages/api-contract/src
mkdir -p packages/core/src
mkdir -p packages/client/src
mkdir -p packages/db/src
mkdir -p apps/web/src
mkdir -p apps/cli/src
```

**Checkpoint**: `pnpm install` проходит без ошибок. `pnpm lint` запускается (даже если пока нечего линтить).

---

## Глава 2: Documentation Skeleton — Memory Bank

### Почему до кода

Из git history: коммиты `37139c0`, `ee0f0a7`, `b1962de` — это Memory Bank, specs и планы. Они появились **раньше** первого кода. Это не случайность: если начать с кода, структура знания будет "догонять" код и никогда не догонит.

### Шаг 2.1: Создание структуры

```bash
mkdir -p .memory-bank/spec/domain
mkdir -p .memory-bank/spec/project
mkdir -p .memory-bank/spec/engineering
mkdir -p .memory-bank/spec/security
mkdir -p .memory-bank/spec/testing
mkdir -p .memory-bank/plans/epics
mkdir -p .memory-bank/adr
mkdir -p .memory-bank/mbb/templates
mkdir -p .memory-bank/guides
```

### Шаг 2.2: Root index с аннотированными ссылками

Это центральная точка входа в знание проекта. В реальном проекте паттерн выглядит так:

```markdown
# Memory Bank — Index

## Quick start (for agents)
- [Project structure](structure.md): где лежат `apps/`, `packages/`,
  `.memory-bank/` и как разделены слои.
  Читать в начале работы, чтобы сразу класть изменения в правильные места.
- [Implementation playbook](plans/implementation-playbook.md):
  пошаговый чеклист реализации фичи (contract→core→db→cli→tests→docs).
  Читать как рабочую инструкцию полного цикла разработки.

## Key folders (SSoT map)
- [Specifications (`spec/`)](spec/index.md): все нормативные требования (WHAT).
- [Plans (`plans/`)](plans/index.md): roadmap, эпики/фичи, playbook.
- [ADR (`adr/`)](adr/index.md): решения уровня "почему".
- [MBB (`mbb/`)](mbb/index.md): правила ведения документации.
```

Обратите внимание на паттерн **аннотированных ссылок**: каждая ссылка содержит не только URL, но и аннотацию "зачем читать". Это позволяет агенту и человеку быстро понять, нужен ли им этот документ.

### Шаг 2.3: Базовые спецификации

Минимальный набор документов перед началом кодирования:

1. **`spec/project/layers-and-vertical-slices.md`** — определение слоёв и DoD для slice
2. **`spec/engineering/architecture-guardrails.md`** — запреты (UI не содержит бизнес-логику, CLI не вызывает core напрямую)
3. **`plans/implementation-playbook.md`** — чеклист реализации фичи
4. **`adr/0001-core-client-cli-first.md`** — первое архитектурное решение

### Шаг 2.4: ADR — Architecture Decision Record

Формат ADR прост:

```markdown
# ADR 0001 — Core + Typed Client + CLI-first
Status: Draft

## Decision
Сначала строим core use-cases + typed contract + typed client + CLI,
и только потом UI.

## Why
- UI и CLI остаются "тонкими" и не дублируют правила.
- Логику можно тестировать быстрее и дешевле (без браузера).
- CLI удобен для ИИ-агента и для отладки сценариев/сидов.

## Trade-offs
- На старте больше дисциплины и "каркаса",
  но меньше регрессий в будущем.
```

### Шаг 2.5: Implementation Playbook

Это рабочая инструкция для каждой фичи. Порядок слоёв фиксирован:

```
0) Собрать контекст (прочитать FT-документ и связанные specs)
1) Contract: операция, DTO, ошибки
2) Core: use-case + policy + инварианты
3) Data/DB: schema + миграции
4) Adapters: HTTP handlers
5) Typed client: транспорт + методы
6) CLI: команда 1:1 к операции
7) Tests: от инварианта к уровню
8) Seed variants
9) Memory bank updates
```

**Checkpoint**: из `.memory-bank/index.md` можно дойти до spec, plans, adr. Шаблоны для epic/feature созданы. Агент, прочитавший `index.md`, понимает структуру проекта.

---

## Глава 3: Database Layer — Drizzle ORM

### Почему Drizzle

Drizzle ORM выбран за:
- **Type-safe** — схема TypeScript генерирует SQL и типы одновременно
- **SQL-close** — не скрывает SQL за абстракциями, в отличие от Prisma
- **Migration-friendly** — встроенная генерация и применение миграций

### Шаг 3.1: Package setup

```json
{
  "name": "@feedback-360/db",
  "version": "0.1.0",
  "scripts": {
    "db:migrate": "tsx src/migrations.ts",
    "db:health": "tsx src/health.ts"
  },
  "dependencies": {
    "drizzle-orm": "0.44.5",
    "pg": "8.16.3"
  },
  "devDependencies": {
    "drizzle-kit": "0.31.2"
  }
}
```

### Шаг 3.2: Multi-tenancy через `company_id`

Это архитектурное решение уровня схемы. Каждая бизнес-таблица содержит `company_id`:

```typescript
// packages/db/src/schema/tables.ts
import {
  boolean, integer, jsonb, pgTable, text,
  timestamp, unique, uuid,
} from "drizzle-orm/pg-core";

export const companies = pgTable("companies", {
  id: uuid("id").defaultRandom().primaryKey(),
  name: text("name").notNull(),
  timezone: text("timezone").notNull().default("Europe/Kaliningrad"),
  isActive: boolean("is_active").notNull().default(true),
  deletedAt: timestamp("deleted_at", { withTimezone: true }),
  createdAt: timestamp("created_at", { withTimezone: true })
    .notNull().defaultNow(),
  updatedAt: timestamp("updated_at", { withTimezone: true })
    .notNull().defaultNow(),
});
```

> **Aha-момент**: `company_id` на каждой таблице — это не middleware, не RLS alone, а **структурный дизайн данных**. Изоляция тенантов начинается в схеме, а не в коде приложения.

### Шаг 3.3: Soft delete pattern

Вместо физического удаления — два поля:

```typescript
isActive: boolean("is_active").notNull().default(true),
deletedAt: timestamp("deleted_at", { withTimezone: true }),
```

Это позволяет восстановить данные и сохранить историческую целостность (например, snapshot-ы кампаний ссылаются на сотрудников, которые могли быть "удалены").

### Шаг 3.4: History tables — temporal data

Для отслеживания перемещений сотрудников используется паттерн `start_at` / `end_at`:

```typescript
export const employeeDepartmentHistory = pgTable(
  "employee_department_history", {
    id: uuid("id").defaultRandom().primaryKey(),
    employeeId: uuid("employee_id").notNull()
      .references(() => employees.id, { onDelete: "cascade" }),
    departmentId: uuid("department_id").notNull()
      .references(() => departments.id, { onDelete: "cascade" }),
    startAt: timestamp("start_at", { withTimezone: true }).notNull(),
    endAt: timestamp("end_at", { withTimezone: true }),
    createdAt: timestamp("created_at", { withTimezone: true })
      .notNull().defaultNow(),
  },
);
```

`endAt = null` означает "текущая принадлежность". При перемещении — закрываем старую запись (`endAt = now`) и создаём новую.

### Шаг 3.5: Uniqueness constraints

Для предотвращения дублей используются составные unique constraints:

```typescript
export const companyMemberships = pgTable(
  "company_memberships",
  {
    id: uuid("id").defaultRandom().primaryKey(),
    companyId: uuid("company_id").notNull()
      .references(() => companies.id, { onDelete: "cascade" }),
    userId: uuid("user_id").notNull(),
    role: text("role").notNull(),
    createdAt: timestamp("created_at", { withTimezone: true })
      .notNull().defaultNow(),
  },
  (table) => [
    unique("uq_membership_user_company")
      .on(table.userId, table.companyId)
  ],
);
```

### Шаг 3.6: Миграции и health check

```typescript
// packages/db/src/migrations.ts — упрощённый пример
import { migrate } from "drizzle-orm/node-postgres/migrator";
import { db } from "./db";

export const runMigrations = async () => {
  await migrate(db, { migrationsFolder: "./drizzle" });
};
```

**Checkpoint**: `pnpm db:migrate` создаёт таблицы. `pnpm db:health` подтверждает подключение к БД.

---

## Глава 4: Typed Contract — api-contract

### Зачем отдельный пакет

Contract живёт отдельно от core, client и web, потому что он — **общий язык** между всеми слоями. Core его реализует, client его потребляет, web через него общается. Если положить типы в core — client получит зависимость на core (нарушение guardrails).

### Шаг 4.1: OperationResult — discriminated union

Это **ключевой тип** всей системы:

```typescript
// packages/api-contract/src/v1/legacy.ts

export type OperationError = {
  code: OperationErrorCode;
  message: string;
  details?: Record<string, unknown>;
};

export type OperationResult<Output> =
  | { ok: true; data: Output }
  | { ok: false; error: OperationError };
```

И helper-функции для создания:

```typescript
export const okResult = <T>(data: T): OperationResult<T> => ({
  ok: true, data,
});

export const errorResult = <T>(
  error: OperationError,
): OperationResult<T> => ({
  ok: false, error,
});
```

> **Aha-момент**: `OperationResult` — это **не просто обёртка**. Это discriminated union, который заставляет вызывающий код обрабатывать оба случая. Нельзя случайно прочитать `data`, не проверив `ok`. TypeScript не даст.

### Шаг 4.2: Typed error codes

Ошибки не строки, а типизированный набор:

```typescript
export const operationErrorCodes = [
  "invalid_input",
  "unauthenticated",
  "forbidden",
  "not_found",
  "invalid_transition",
  "campaign_started_immutable",
  "campaign_locked",
  "campaign_ended_readonly",
  "webhook_invalid_signature",
  "webhook_timestamp_invalid",
  "ai_job_conflict",
] as const;

export type OperationErrorCode = (typeof operationErrorCodes)[number];
```

Обратите внимание: доменные ошибки (`campaign_started_immutable`, `campaign_locked`) сосуществуют рядом с инфраструктурными (`unauthenticated`, `invalid_input`). Это позволяет клиенту показывать осмысленные сообщения.

### Шаг 4.3: OperationContext — кто вызывает

```typescript
export const membershipRoles = [
  "hr_admin", "hr_reader", "manager", "employee",
] as const;
export type MembershipRole = (typeof membershipRoles)[number];

export type OperationContext = {
  userId?: string;
  employeeId?: string;
  companyId?: string;
  role?: MembershipRole;
};
```

Четыре роли, четыре уровня доступа. Контекст передаётся с каждым запросом — это фундамент RBAC.

### Шаг 4.4: KnownOperation — каталог операций

```typescript
export const knownOperations = [
  "seed.run",
  "system.ping",
  "company.updateProfile",
  "membership.list",
  "campaign.list",
  "campaign.get",
  "campaign.create",
  "campaign.start",
  "campaign.stop",
  "campaign.end",
  // ... 50+ операций
  "questionnaire.listAssigned",
  "questionnaire.getDraft",
  "questionnaire.saveDraft",
  "questionnaire.submit",
] as const;

export type KnownOperation = (typeof knownOperations)[number];
```

### Шаг 4.5: DispatchOperationInput — конверт запроса

```typescript
export type DispatchOperationInput = {
  operation: string;
  input: unknown;
  context?: OperationContext;
};
```

Это "конверт" для любой операции. `operation` — имя, `input` — нетипизированный payload (будет спарсен в handler), `context` — кто вызывает.

### Шаг 4.6: Parse functions — runtime boundary

Вместо Zod/Yup — простые parse-функции:

```typescript
export const parseSystemPingOutput = (value: unknown): SystemPingOutput => {
  if (!value || typeof value !== "object") {
    throw new Error("Invalid SystemPingOutput");
  }
  const v = value as Record<string, unknown>;
  return {
    pong: String(v.pong ?? ""),
    timestamp: String(v.timestamp ?? ""),
  };
};
```

Почему не Zod: простота, zero dependencies, полный контроль. Parse-функции — это **runtime boundary** между слоями. Каждое значение, пересекающее границу пакета, парсится.

**Checkpoint**: `pnpm typecheck` в api-contract проходит. Parse functions принимают валидный input и отвергают невалидный.

---

## Глава 5: Central Dispatcher — core

### Dispatcher pattern

Вместо REST-контроллеров, вместо GraphQL resolvers — один `dispatchOperation()`, который маршрутизирует запрос по имени операции.

### Шаг 5.1: Operation handlers map

```typescript
// packages/core/src/index.ts

type OperationHandler = (
  request: DispatchOperationInput,
) => Promise<OperationResult<DispatchOutput>>
   | OperationResult<DispatchOutput>;

const operationHandlers: Partial<
  Record<KnownOperation, OperationHandler>
> = {
  "system.ping": (request) => runSystemPing(request.input),
  "company.updateProfile": runCompanyUpdateProfile,
  "membership.list": runMembershipList,
  "campaign.list": runCampaignList,
  "campaign.get": runCampaignGet,
  "campaign.create": runCampaignCreate,
  "campaign.start": runCampaignStart,
  "campaign.stop": runCampaignStop,
  "campaign.end": runCampaignEnd,
  // ... 50+ handlers
};
```

Это **routing table** — просто маппинг имени операции на функцию. Нет middleware chains, нет framework magic.

### Шаг 5.2: dispatchOperation — единая точка входа

```typescript
export const dispatchOperation = (
  request: DispatchOperationInput,
): Promise<OperationResult<DispatchOutput>> => {
  // 1. Parse and validate input
  let parsedRequest: DispatchOperationInput;
  try {
    parsedRequest = parseDispatchOperationInput(request);
  } catch (error) {
    return Promise.resolve(
      errorResult(errorFromUnknown(
        error, "invalid_input", "Invalid dispatch input."
      )),
    );
  }

  // 2. Check if operation is known
  if (!isKnownOperation(parsedRequest.operation)) {
    return Promise.resolve(
      errorResult(createOperationError(
        "not_found",
        `Unknown operation: ${parsedRequest.operation}`,
      )),
    );
  }

  // 3. Block client-local operations
  if (parsedRequest.operation === "client.setActiveCompany"
      || parsedRequest.operation === "seed.run") {
    return Promise.resolve(
      errorResult(createOperationError(
        "not_found",
        "Operation is client-local and unavailable in core.",
      )),
    );
  }

  // 4. Find and execute handler
  const handler = operationHandlers[parsedRequest.operation];
  if (!handler) {
    return Promise.resolve(
      errorResult(createOperationError(
        "not_found",
        `Unknown operation: ${parsedRequest.operation}`,
      )),
    );
  }

  return Promise.resolve(handler(parsedRequest));
};
```

> **Aha-момент**: Весь core — это **одна функция-маршрутизатор**. Одна точка входа, один формат запроса, один формат ответа. Это делает систему предсказуемой: если вы понимаете один handler, вы понимаете все.

### Шаг 5.3: Context helpers — shared utilities

```typescript
// packages/core/src/shared/context.ts

export const ensureContextCompany = (
  request: DispatchOperationInput,
): string | OperationResult<never> => {
  const companyId = request.context?.companyId;
  if (!companyId) {
    return errorResult(
      createOperationError(
        "forbidden",
        "Active company is required for this operation.",
      ),
    );
  }
  return companyId;
};

export const hasRole = (
  request: DispatchOperationInput,
  allowedRoles: readonly string[],
): boolean => {
  const role = request.context?.role;
  return Boolean(role && allowedRoles.includes(role));
};
```

`ensureContextCompany` возвращает **union**: либо `string` (companyId), либо `OperationResult<never>` (ошибка). Вызывающий код проверяет тип:

```typescript
const companyIdOrError = ensureContextCompany(request);
if (typeof companyIdOrError !== "string") {
  return companyIdOrError; // это OperationResult с ошибкой
}
// здесь companyIdOrError точно string
```

### Шаг 5.4: Простейший handler — system.ping

```typescript
// packages/core/src/features/identity-tenancy.ts

export const runSystemPing = (
  input: unknown,
): OperationResult<SystemPingOutput> => {
  try {
    parseSystemPingInput(input);
  } catch (error) {
    return errorResult(errorFromUnknown(
      error, "invalid_input", "Invalid system.ping input."
    ));
  }

  try {
    return okResult(
      parseSystemPingOutput({
        pong: "ok",
        timestamp: new Date().toISOString(),
      }),
    );
  } catch (error) {
    return errorResult(errorFromUnknown(
      error, "invalid_input", "Invalid system.ping output payload."
    ));
  }
};
```

### Шаг 5.5: Типичный handler с DB — membership.list

```typescript
export const runMembershipList = async (
  request: DispatchOperationInput,
): Promise<OperationResult<MembershipListOutput>> => {
  // 1. Auth check
  const userId = request.context?.userId;
  if (!userId) {
    return errorResult(createOperationError(
      "unauthenticated",
      "User context is required for membership list.",
    ));
  }

  // 2. Parse input
  let parsedInput: MembershipListInput;
  try {
    parsedInput = parseMembershipListInput(request.input);
  } catch (error) {
    return errorResult(errorFromUnknown(
      error, "invalid_input", "Invalid membership.list input."
    ));
  }

  // 3. Call DB + parse output
  try {
    const output = await listMemberships({ userId, ...parsedInput });
    return okResult(parseMembershipListOutput(output));
  } catch (error) {
    return errorResult(errorFromUnknown(
      error, "invalid_input", "Failed to list memberships."
    ));
  }
};
```

Паттерн handler-а: **parse → auth/role check → ensure company → call DB → audit → parse output**. Каждый handler следует этой структуре.

### Шаг 5.6: Audit trail

```typescript
// packages/core/src/shared/audit.ts

export const recordAuditEvent = async (
  request: DispatchOperationInput,
  payload: {
    companyId: string;
    campaignId?: string;
    source?: "ui" | "system" | "release" | "webhook" | "cron";
    eventType: string;
    objectType: string;
    objectId?: string;
    summary: string;
    metadataJson?: Record<string, unknown>;
  },
): Promise<void> => {
  await db.createAuditEvent({
    companyId: payload.companyId,
    actorUserId: request.context?.userId,
    actorRole: request.context?.role,
    source: payload.source ?? "ui",
    eventType: payload.eventType,
    objectType: payload.objectType,
    summary: payload.summary,
    metadataJson: payload.metadataJson,
  });
};
```

Audit вызывается **после** мутации. Каждое изменение состояния записывается с актором, ролью и метаданными.

**Checkpoint**: `system.ping` проходит через dispatcher и возвращает `{ ok: true, data: { pong: "ok", timestamp: "..." } }`.

---

## Глава 6: Typed Client — Transport Abstraction

### Два транспорта — один контракт

Client package предоставляет **одинаковый** typed API, независимо от того, вызывается core напрямую (in-process) или через HTTP.

### Шаг 6.1: OperationTransport interface

```typescript
// packages/client/src/shared/runtime.ts

export type OperationTransport = {
  invoke(request: DispatchOperationInput): Promise<unknown>;
};
```

Минимальный интерфейс — одна функция `invoke`. Два варианта реализации:

```typescript
// In-process: прямой вызов core (для тестов и CLI)
export const createInprocTransport = (): OperationTransport => ({
  invoke: async (request) => dispatchOperation(request),
});

// HTTP: POST к серверу (для браузера и remote clients)
export const createHttpTransport = (
  options: CreateHttpTransportOptions,
): OperationTransport => {
  const endpointUrl = `${options.baseUrl}/api/v1/operations`;
  return {
    invoke: async (request) => {
      const response = await fetch(endpointUrl, {
        method: "POST",
        headers: { "content-type": "application/json" },
        body: JSON.stringify(request),
      });
      return response.json();
    },
  };
};
```

> **Aha-момент**: In-proc transport вызывает core напрямую, HTTP transport делает POST к серверу — но оба возвращают **идентичный** `OperationResult<T>`. Тесты используют in-proc; production использует HTTP. Client layer **не добавляет бизнес-логики**.

### Шаг 6.2: ClientRuntime — внутренний движок

```typescript
export const createClientRuntime = (
  transport: OperationTransport,
): ClientRuntime => {
  let activeContext: OperationContext = {};

  return {
    invokeOperation: async <Output>({
      operation, input, context, parseOutput,
    }: InvokeOperationParams<Output>) => {
      // 1. Build and parse request
      const request = parseDispatchOperationInput({
        operation, input,
        context: { ...activeContext, ...context },
      });

      // 2. Invoke transport
      const rawResult = await transport.invoke(request);

      // 3. Parse result
      return parseOperationResult(rawResult, parseOutput);
    },

    setActiveContext: (context) => {
      activeContext = { ...activeContext, ...context };
      return okResult({ ...activeContext });
    },

    setActiveCompany: (companyId) => {
      activeContext = { ...activeContext, companyId };
      return okResult({ companyId });
    },

    getActiveContext: () => ({ ...activeContext }),
    getActiveCompany: () => activeContext.companyId,
  };
};
```

### Шаг 6.3: Feature-area client methods

Каждая feature area предоставляет typed methods:

```typescript
// packages/client/src/features/campaigns.ts

export type CampaignsClientMethods = {
  campaignList(
    input?: CampaignListInput,
    context?: OperationContext,
  ): Promise<OperationResult<CampaignListOutput>>;
  campaignGet(
    input: CampaignGetInput,
    context?: OperationContext,
  ): Promise<OperationResult<CampaignGetOutput>>;
  campaignStart(
    input: CampaignTransitionInput,
    context?: OperationContext,
  ): Promise<OperationResult<CampaignTransitionOutput>>;
  // ...
};

export const createCampaignsClientMethods = (
  runtime: ClientRuntime,
): CampaignsClientMethods => ({
  campaignList: async (input, context) => {
    const parsedInput = parseCampaignListInput(input ?? {});
    return runtime.invokeOperation({
      operation: "campaign.list",
      input: parsedInput,
      context,
      parseOutput: parseCampaignListOutput,
    });
  },
  // ...
});
```

### Шаг 6.4: Composition — Feedback360Client

```typescript
// packages/client/src/index.ts

export type Feedback360Client = BaseClientMethods
  & IdentityTenancyClientMethods
  & ModelsClientMethods
  & CampaignsClientMethods
  & OrgClientMethods
  & NotificationsClientMethods
  & OpsClientMethods
  & MatrixClientMethods
  & AiClientMethods
  & QuestionnairesClientMethods
  & ResultsClientMethods;

export const createClient = (
  transport: OperationTransport,
): Feedback360Client => {
  const runtime = createClientRuntime(transport);
  return {
    invokeOperation: runtime.invokeOperation,
    ...createSystemClientMethods(runtime),
    ...createIdentityTenancyClientMethods(runtime),
    ...createCampaignsClientMethods(runtime),
    // ... все feature areas
  };
};

// Convenience: in-proc client для тестов и CLI
export const createInprocClient = (): Feedback360Client => {
  return createClient(createInprocTransport());
};
```

**Checkpoint**: `createInprocClient().systemPing()` возвращает `{ ok: true, data: { pong: "ok" } }`.

---

## Глава 7: CLI-First Verification

### CLI — не afterthought, а primary verification tool

Из ADR 0001: _"CLI удобен для ИИ-агента и для отладки сценариев/сидов."_

### Шаг 7.1: CLI entry point

CLI строится на Commander над typed client. Каждая команда — 1:1 к операции:

```typescript
// apps/cli/src/index.ts (упрощённый пример)
import { Command } from "commander";
import { createInprocClient } from "@feedback-360/client";

const program = new Command();
const client = createInprocClient();

program
  .command("ping")
  .description("System health check")
  .option("--json", "JSON output")
  .action(async (opts) => {
    const result = await client.systemPing();
    if (opts.json) {
      console.log(JSON.stringify(result, null, 2));
    } else {
      console.log(result.ok ? "pong" : `Error: ${result.error.message}`);
    }
  });

program.parse();
```

### Шаг 7.2: Dual output — human и `--json`

Контракт: `--json` выдаёт **стабильную машиночитаемую** схему. Human output — для людей. AI-агент всегда использует `--json`.

### Шаг 7.3: Seed scenarios с handles

Seed — это детерминированные сценарии тестовых данных:

```typescript
export const seedScenarios = [
  "S0_empty",
  "S1_company_min",
  "S2_org_basic",
  "S4_campaign_draft",
  "S5_campaign_started_no_answers",
  "S7_campaign_started_some_submitted",
  "S9_campaign_completed_with_ai",
] as const;
```

Каждый сценарий создаёт предсказуемое состояние. Тесты ссылаются на **handles** (именованные ссылки), а не на случайные UUID.

**Checkpoint**: `cli ping --json` выдаёт `{ "ok": true, "data": { "pong": "ok" } }`. `cli seed --scenario S1_company_min --json` создаёт компанию и возвращает handles.

---

## Глава 8: Первый Vertical Slice — Identity & Tenancy

### Что такое vertical slice на практике

Это момент, когда все слои соединяются в одну цепочку. Проследим путь операции `membership.list` через все слои:

### Шаг 8.1: Contract

```typescript
// packages/api-contract/src/identity-tenancy.ts (или v1/legacy.ts)
export type MembershipListInput = { /* пустой для этой операции */ };
export type MembershipListOutput = {
  memberships: Array<{
    companyId: string;
    companyName: string;
    role: MembershipRole;
  }>;
};
```

### Шаг 8.2: Core handler

```typescript
// packages/core/src/features/identity-tenancy.ts
export const runMembershipList = async (
  request: DispatchOperationInput,
): Promise<OperationResult<MembershipListOutput>> => {
  const userId = request.context?.userId;
  if (!userId) {
    return errorResult(createOperationError(
      "unauthenticated",
      "User context is required.",
    ));
  }
  const output = await listMemberships({ userId });
  return okResult(parseMembershipListOutput(output));
};
```

### Шаг 8.3: DB function

```typescript
// packages/db/src/identity.ts
export const listMemberships = async ({ userId }) => {
  return db.select(...)
    .from(companyMemberships)
    .innerJoin(companies, eq(...))
    .where(eq(companyMemberships.userId, userId));
};
```

### Шаг 8.4: Client method

```typescript
// packages/client/src/features/identity-tenancy.ts
membershipList: async (input, context) => {
  return runtime.invokeOperation({
    operation: "membership.list",
    input: input ?? {},
    context,
    parseOutput: parseMembershipListOutput,
  });
},
```

### Шаг 8.5: CLI command

```bash
cli membership-list --json
# => { "ok": true, "data": { "memberships": [...] } }
```

### Шаг 8.6: Test

```typescript
test("membership.list returns companies for user", async () => {
  const client = createInprocClient();
  client.setActiveContext({ userId: "test-user-id" });
  const result = await client.membershipList();
  expect(result.ok).toBe(true);
  if (result.ok) {
    expect(result.data.memberships).toBeInstanceOf(Array);
  }
});
```

> **Aha-момент**: Один запрос проходит через 5 слоёв, и в каждом слое ровно одна ответственность. Contract определяет язык. Client даёт API. Core решает бизнес. DB хранит данные. CLI доставляет.

**Checkpoint**: `membership.list` работает через CLI с реальной БД. Тест подтверждает изоляцию по компании.

---

## Глава 9: Domain Modeling — Org, Models, Campaigns

### Org structure с temporal history

Сотрудники, отделы, позиции — это не статичные сущности. Они меняются во времени:

```typescript
// Сотрудник может переходить между отделами
export const employeeDepartmentHistory = pgTable(
  "employee_department_history", { ... });

// И менять руководителей
export const employeeManagerHistory = pgTable(
  "employee_manager_history", { ... });

// И позиции
export const employeePositions = pgTable(
  "employee_positions", { ... });
```

### Competency models — indicators vs levels

Два вида моделей компетенций:
- **Indicators** (1-5 + NA) — непрерывная шкала, средняя оценка
- **Levels** (1-4 + UNSURE) — дискретные уровни, мода

Модели версионируются: `draft` → `published`. Кампания ссылается на конкретную версию. После старта кампании версия замораживается.

### Campaign lifecycle — state machine

```
draft → started → ended → processing_ai → completed
                                        ↘ ai_failed
```

```typescript
export const campaigns = pgTable("campaigns", {
  // ...
  status: text("status").notNull().default("draft"),
  lockedAt: timestamp("locked_at", { withTimezone: true }),
  managerWeight: integer("manager_weight").notNull().default(40),
  peersWeight: integer("peers_weight").notNull().default(30),
  subordinatesWeight: integer("subordinates_weight")
    .notNull().default(30),
  selfWeight: integer("self_weight").notNull().default(0),
});
```

### Snapshot pattern — заморозка org state

При `campaign.start` система делает snapshot всех участников:

```typescript
export const campaignEmployeeSnapshots = pgTable(
  "campaign_employee_snapshots", {
    // ... копия данных сотрудника на момент старта
    email: text("email").notNull(),
    firstName: text("first_name"),
    lastName: text("last_name"),
    departmentId: uuid("department_id"),
    managerEmployeeId: uuid("manager_employee_id"),
    positionTitle: text("position_title"),
    snapshotAt: timestamp("snapshot_at", { withTimezone: true })
      .notNull(),
  },
);
```

> **Aha-момент**: Snapshot — это **доменное решение**, а не техническая оптимизация. Когда org structure меняется после старта кампании, оценка использует **замороженное** состояние. Это "справедливость" в дата-модели.

### Assignments и matrix

```typescript
export const campaignAssignments = pgTable(
  "campaign_assignments", {
    subjectEmployeeId: uuid("subject_employee_id").notNull(),
    raterEmployeeId: uuid("rater_employee_id").notNull(),
    raterRole: text("rater_role").notNull(),
    // raterRole: "manager" | "peer" | "subordinate" | "self"
  },
);
```

Matrix генерируется автоматически из org snapshot и замораживается после первого draft save (ADR 0003).

**Checkpoint**: Кампания может быть создана, запущена (с snapshot), и полностью пройти lifecycle.

---

## Глава 10: Results, Anonymity, Calculations

### Questionnaire lifecycle

```
not_started → in_progress → submitted
```

Draft payload хранится как JSONB — частичные ответы в любом порядке. Submit валидирует, что все компетенции отвечены.

### Anonymity policy

Порог: минимум 3 оценщика в группе для показа результатов. Если меньше:
- Manager — **всегда** не-анонимный (personal feedback)
- Peers / Subordinates — сливаются в "Other" при количестве < порога
- Self — weight = 0% по умолчанию

### Weight normalization

Если группа скрыта или отсутствует, веса перераспределяются:

```
manager: 40%, peers: 30%, subordinates: 30%, self: 0%

Если subordinates отсутствуют:
  manager: 40/(40+30) * 100 = 57%
  peers: 30/(40+30) * 100 = 43%
```

### Role-based result views

Три операции — три уровня доступа:

| Операция | Роль | Видит |
|----------|------|-------|
| `results.getHrView` | hr_admin | Всё: raw, processed, summary, все группы |
| `results.getTeamDashboard` | manager | Агрегаты команды + own group non-anonymous |
| `results.getMyDashboard` | employee | Только processed/summary, фильтр по visibility |

> **Aha-момент**: Анонимность — это не просто "скрыть имя". Это threshold-проверки, merge групп, разные visibility rules по ролям. Эта логика **обязана** жить в core, не в UI.

**Checkpoint**: Результаты видны через `results.getHrView` и `results.getMyDashboard` с корректной фильтрацией по ролям.

---

## Глава 11: Notifications — Outbox Pattern

### Outbox вместо прямой отправки

```typescript
export const notificationOutbox = pgTable(
  "notification_outbox", {
    id: uuid("id").defaultRandom().primaryKey(),
    status: text("status").notNull().default("pending"),
    idempotencyKey: text("idempotency_key").notNull(),
    channel: text("channel").notNull().default("email"),
    templateKey: text("template_key").notNull(),
    attempts: integer("attempts").notNull().default(0),
    nextRetryAt: timestamp("next_retry_at", { withTimezone: true }),
    sentAt: timestamp("sent_at", { withTimezone: true }),
  },
  (table) => [
    unique("uq_notification_outbox_idempotency")
      .on(table.idempotencyKey)
  ],
);
```

### Цикл: generate → dispatch → retry

1. `notifications.generateReminders` — создаёт записи в outbox
2. `notifications.dispatchOutbox` — отправляет pending, ретраит failed
3. Idempotency key предотвращает дубли

> **Aha-момент**: Outbox разделяет "решение уведомить" от "отправки". Если email fails, outbox ретраит автоматически, не пересчитывая бизнес-логику.

**Checkpoint**: `campaign.start` создаёт invite notifications в outbox. `notifications.dispatchOutbox` обрабатывает их.

---

## Глава 12: Web Application — Next.js как Thin Layer

### Принцип: route handler не знает бизнес

Web app (Next.js 15 App Router) — это **delivery adapter**. Он:
1. Парсит HTTP/form payload
2. Резолвит session → `OperationContext`
3. Вызывает in-proc client
4. Мапит typed errors в HTTP status / redirects

### Шаг 12.1: Route handler pattern

```typescript
// apps/web/src/app/api/hr/campaigns/draft/route.ts (структура)
import { createInprocClient } from "@feedback-360/client";

export async function POST(request: NextRequest) {
  // 1. Parse form data
  const formData = await request.formData();
  const name = formData.get("name") as string;

  // 2. Resolve context from session
  const context = await resolveAppOperationContext(request);

  // 3. Call typed client
  const client = createInprocClient();
  client.setActiveContext(context);
  const result = await client.campaignCreate({
    name,
    startAt: formData.get("startAt") as string,
    endAt: formData.get("endAt") as string,
  });

  // 4. Map result to HTTP response
  if (!result.ok) {
    return NextResponse.redirect("/hr/campaigns?error=...");
  }
  return NextResponse.redirect(`/hr/campaigns/${result.data.id}`);
}
```

Route handler — ~30-50 строк. **Zero business logic**. Если правило меняется — меняется только core.

### Шаг 12.2: UI stack

- **Tailwind v4** — utility-first CSS
- **shadcn/ui** — компоненты (Button, Card, Table, Dialog)
- **Server Components** — для загрузки данных
- **Client Components** — для интерактивности

**Checkpoint**: Web app рендерится. Login работает. Хотя бы одна CRUD-страница показывает данные из тех же операций, что и CLI.

---

## Глава 13: Testing Strategy & Quality Gates

### Четыре уровня тестов

| Уровень | Что проверяет | Где | Скорость |
|---------|-------------|-----|----------|
| Unit | Политики, расчёты, state machine | `packages/core/` | Мгновенно |
| Integration | Use-case + реальная БД | `packages/core/`, `packages/db/` | Секунды |
| Contract | Parse functions, DTO shapes | `packages/api-contract/` | Мгновенно |
| E2E | Сквозной user journey | `apps/web/playwright/` | Минуты |

### Quality gate — обязательный пайплайн

```bash
pnpm checks
# Выполняет последовательно:
# pnpm lint          — Biome
# pnpm typecheck     — tsc --noEmit
# pnpm test          — Vitest unit/contract
# pnpm test:db       — integration с реальной БД
# pnpm --filter @feedback-360/web build  — проверка сборки
```

Фича не считается готовой, пока `checks` не зелёный.

### Evidence-based acceptance

Каждая FT должна иметь:
1. Automated test, повторяющий acceptance scenario
2. Quality checks evidence (дата прогона)
3. Acceptance evidence (дата подтверждения)

**Checkpoint**: `pnpm checks` проходит полностью зелёным.

---

## Глава 14: Feature-Area Refactor

### Когда layer-flat → feature-area

Из ADR 0004: _"К моменту завершения EP-013 проект уже имеет working vertical slices, но production-код собран вокруг крупных root entrypoints. Это увеличивает стоимость сопровождения."_

### До рефакторинга

```
packages/core/src/
  index.ts          ← 500+ строк, все handlers
  context.ts
  audit.ts
```

### После рефакторинга

```
packages/core/src/
  index.ts          ← thin composition (~295 строк)
  features/
    identity-tenancy.ts
    campaigns.ts
    questionnaires.ts
    results.ts
    notifications.ts
    org.ts
    models.ts
    matrix.ts
    ai.ts
    ops.ts
  shared/
    context.ts
    audit.ts
```

10 canonical feature areas:
`identity-tenancy`, `org`, `models`, `campaigns`, `matrix`, `questionnaires`, `results`, `notifications`, `ai`, `ops`

### Правила для `shared`

`shared` — **только** для модулей без одного очевидного owner. Если модуль естественно принадлежит одной feature area — он живёт там, даже если используется другими.

### Порядок рефакторинга

1. Сначала docs/rationale (ADR)
2. Потом code move
3. Потом regression evidence

> **Aha-момент**: Рефакторинг — не признак неудачи, а **запланированная** архитектурная эволюция. Проект документирует ПОЧЕМУ перестроил код (ADR 0004) и верифицирует ЧТО не сломалось (regression evidence).

**Checkpoint**: Все тесты проходят после restructuring. Root entrypoints стали thin.

---

## Глава 15: XE Scenarios, Guides, UI Traceability

### Cross-epic (XE) scenarios

Когда feature slices стабильны, появляются **сквозные сценарии**, тестирующие полные user journeys через несколько feature areas:

```
XE-001: Полный цикл 360-review
  1. Создать компанию
  2. Добавить сотрудников и отделы
  3. Создать модель компетенций
  4. Создать и запустить кампанию
  5. Заполнить анкеты
  6. Завершить и получить результаты
```

### Diataxis guides

Пользовательская документация организована по фреймворку Diataxis:
- **Tutorials** — пошаговые прохождения (первая кампания)
- **How-to** — решение конкретных задач
- **Explanation** — объяснение концепций
- **Reference** — справочная информация

### Screen registry и test ID registry

Каждый экран имеет canonical `screen_id`. Каждый интерактивный элемент — `data-testid` по предсказуемому паттерну. Это позволяет:
- Автоматизировать browser tests с надёжными селекторами
- AI-агенту строить deterministic UI assertions

### Design system

Tokens, semantic statuses, component rules — зафиксированы в `.memory-bank/spec/ui/design-system/`. UI-изменения идут через систему, а не как adhoc patches.

**Checkpoint**: Полный сценарий от создания компании до просмотра результатов проходит end-to-end.

---

## Appendix A: Dependency Graph пакетов

```
                    ┌─────────────────┐
                    │  api-contract   │
                    │  (types, parse) │
                    └───────┬─────────┘
                            │
              ┌─────────────┼─────────────┐
              │             │             │
        ┌─────▼─────┐ ┌────▼────┐ ┌──────▼──────┐
        │    core    │ │  client │ │     db      │
        │ (handlers, │ │(runtime,│ │  (schema,   │
        │ dispatcher)│ │ methods)│ │  migrations)│
        └─────┬──┬──┘ └────┬────┘ └──────┬──────┘
              │  │         │             │
              │  └─────────┤             │
              │            │             │
              │      ┌─────▼─────┐       │
              │      │   client  │       │
              │      │  (uses    │       │
              │      │   core    │       │
              │      │   via     │       │
              │      │ transport)│       │
              │      └─────┬─────┘      │
              │            │             │
        ┌─────▼────┐ ┌────▼────┐        │
        │   CLI    │ │   Web   │        │
        │(commands)│ │ (routes │        │
        │          │ │  + UI)  │        │
        └──────────┘ └─────────┘        │
```

Правило: стрелки идут **только вниз**. Web и CLI не импортируют core напрямую — только через client.

## Appendix B: Каталог операций по feature areas

### Identity & Tenancy (4)
- `system.ping` — health check
- `company.updateProfile` — обновление профиля компании (hr_admin)
- `membership.list` — список компаний пользователя
- `identity.provisionAccess` — привязка user↔employee (hr_admin)

### Organization (6)
- `employee.upsert` / `employee.listActive` / `employee.directoryList` / `employee.profileGet`
- `department.list` / `department.upsert`
- `org.department.move` / `org.manager.set`

### Models (6)
- `model.version.create` / `.list` / `.get` / `.cloneDraft` / `.upsertDraft` / `.publish`

### Campaigns (13)
- `campaign.create` / `.list` / `.get` / `.updateDraft`
- `campaign.start` / `.stop` / `.end`
- `campaign.setModelVersion` / `.weights.set`
- `campaign.participants.add` / `.remove` / `.addFromDepartments`
- `campaign.snapshot.list` / `.progress.get`

### Matrix (3)
- `matrix.list` / `.generateSuggested` / `.set`

### Questionnaires (4)
- `questionnaire.listAssigned` / `.getDraft` / `.saveDraft` / `.submit`

### Results (3)
- `results.getHrView` / `.getMyDashboard` / `.getTeamDashboard`

### Notifications (8)
- `notifications.generateReminders` / `.dispatchOutbox`
- `notifications.settings.get` / `.upsert` / `.preview`
- `notifications.templates.list` / `.preview`
- `notifications.deliveries.list`

### AI (1)
- `ai.runForCampaign`

### Ops (3)
- `ops.health.get` / `.aiDiagnostics.list` / `.audit.list`

### Client-local (2)
- `client.setActiveCompany` — переключение активной компании
- `seed.run` — запуск seed-сценария

**Итого: 50+ операций** через единый dispatcher.

## Appendix C: Ключевые ADR

| ADR | Решение | Причина |
|-----|---------|---------|
| 0001 | Core + Typed Client + CLI-first | UI и CLI остаются thin; логику тестируем без браузера |
| 0002 | Anonymity threshold = 3 | Баланс между информативностью и защитой анонимности |
| 0003 | Freeze on first draft save | Матрица и веса фиксируются после начала работы оценщиков |
| 0004 | Feature-area slicing | Root files перестали быть локальными; ownership размывался |

---

## Итог: порядок воссоздания

```
1. INIT     — pnpm workspace, git, biome, TypeScript
2. DOCS-01  — Memory Bank Bible, root indexes, templates
3. DOCS-02  — specs, plans, ADR skeleton
4. FT-0001  — packages/apps scaffold
5. FT-0002  — DB baseline, migrations, health check
6. FT-0003  — seed runner с handles
7. FT-0011  — api-contract: types, parse, errors, dispatcher
8. FT-0012  — client: transport, runtime, factory
9. FT-0013  — CLI: первый vertical slice
10. EP-domain — доменные slices сериями (org → models → campaigns → ...)
11. EP-test  — hardening, smoke, quality gates
12. EP-ui    — web app как thin layer
13. EP-refactor — feature-area slicing
14. EP-gui   — GUI waves
15. EP-xe    — XE scenarios, guides, traceability
```

Каждый шаг заканчивается кодом, тестами и evidence. Новый слой добавляется только когда предыдущий детерминированно работает.