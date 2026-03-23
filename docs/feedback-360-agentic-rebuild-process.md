# feedback-360: Agentic Engineering Rebuild Process

## Что это за документ

Это **процессный** документ. Он описывает не "что получилось" и не "как устроена архитектура", а **как автор реально работал с LLM шаг за шагом от идеи до production**.

Документ содержит:
- Полные промпты для каждого этапа (copy-paste ready).
- Реконструированный brief для 360-feedback системы.
- Конкретные артефакты каждой фазы.
- Checkpoints для проверки готовности перед переходом к следующей фазе.

### Три companion-документа

| Документ | Фокус | Роль |
|----------|-------|------|
| [`feedback-360-faithful-rebuild-guide.md`](feedback-360-faithful-rebuild-guide.md) | Аналитический reverse-engineering | **"Что" и "почему"**: разбор подхода с 5 разных углов + 9 фаз эволюции |
| [`feedback-360-step-by-step-rebuild.md`](feedback-360-step-by-step-rebuild.md) | Технический rebuild | **"Как воссоздать архитектуру"**: работающие примеры кода каждого слоя |
| **Этот документ** | Процесс работы с LLM | **"Как автор реально делал"**: промпты, brief, итерации, checkpoints |

Каждый документ самодостаточен, но они покрывают разные грани одного и того же проекта. Рекомендуемый порядок чтения: сначала faithful rebuild guide (понять "почему"), затем этот документ (повторить процесс), затем step-by-step rebuild (углубиться в код).

### Инструменты автора

- **Obsidian** — работа с brief, структурирование знаний, навигация по Memory Bank.
- **ChatGPT** — итерации над brief через технику "iterative edit" (правка одного сообщения вместо продолжения диалога).
- **Claude / Cursor** — генерация кода через агентов с file-writing tools, specs-driven development.

### Ключевые принципы процесса

1. **Specs-first** — документация появляется раньше кода и задаёт форму проекта.
2. **MBB (Memory Bank Bible)** — правила ведения документации создаются до всех specs.
3. **Vertical slices** — каждая feature — это сквозной кусок от contract до CLI/UI.
4. **CLI-first** — deterministic verification без browser automation.
5. **Thin UI** — web-интерфейс как тонкий слой поверх уже работающего delivery path.

---

## Фаза 0: Идея → Brief

**Что делается**: Свободная "хотелка" превращается в структурированный brief через итерации с LLM.

**Артефакты**: Полный brief, покрывающий domain + technical решения.

**Инструмент**: ChatGPT (или любая модель с возможностью редактировать предыдущее сообщение).

### Как формулировать первоначальную "хотелку"

Начинайте не с технических деталей, а с продуктовой "хотелки". Опишите свободным языком:
- Для кого система (целевая аудитория).
- Что она должна делать (ключевые сценарии).
- Какие ограничения (бюджет, стек, сроки).
- Что точно НЕ входит в первую версию.

Не пытайтесь сразу получить идеальный brief. Первая итерация — это "вброс идеи", который модель поможет структурировать.

### Техника "iterative edit"

Вместо того чтобы продолжать диалог ("а ещё добавь...", "поменяй это..."), автор использует технику **правки одного сообщения**:

1. Написать первое сообщение с идеей.
2. Получить ответ модели.
3. **Отредактировать** своё первое сообщение, включив уточнения.
4. Получить обновлённый ответ.
5. Повторять, пока модель не начнёт спрашивать тривиальности.

Это работает лучше последовательного диалога, потому что:
- Модель всегда видит **полный** контекст, а не "хвост" разговора.
- Brief растёт как **единый документ**, а не как цепочка уточнений.
- Нет риска потерять ранние решения в длинном диалоге.

### Промпт #1: Первичная подача идеи

```
Я хочу создать внутреннюю корпоративную систему для проведения оценки
сотрудников методом 360 градусов.

Вот мои начальные требования:

ДОМЕН:
- Система multi-tenant: несколько компаний в одной инсталляции.
- В компании есть HR (администрирует процесс), руководители и сотрудники.
- HR создаёт "кампанию" оценки: задаёт период, привязывает модель компетенций,
  добавляет участников.
- По каждому оцениваемому формируется матрица оценщиков: руководитель,
  коллеги, подчинённые, самооценка.
- Оценщики заполняют анкеты по шкалам (индикаторы 1-5 + NA, или уровни 1-4
  + UNSURE) и оставляют открытые комментарии.
- Результаты агрегируются с весами по группам оценщиков (менеджер, коллеги,
  подчинённые, self).
- Анонимность: если в группе < 3 оценщиков, группа скрывается или
  объединяется.
- HR видит полную картину, руководитель — своих подчинённых,
  сотрудник — только свои результаты.

ТЕХНИЧЕСКИЙ СТЕК:
- TypeScript monorepo (pnpm workspaces).
- Next.js App Router для web UI.
- Supabase (PostgreSQL) для БД.
- Tailwind CSS + shadcn/ui для компонентов.
- CLI для автоматизации и тестирования через агентов.

Помоги мне закрыть пробелы в этом описании. Задай мне вопросы, которые
помогут превратить это в полный brief. Сгруппируй вопросы по темам:
- Домен (бизнес-правила, которые я не описал)
- Безопасность (аутентификация, авторизация, данные)
- Технические решения (архитектура, деплой, интеграции)
- Границы MVP (что точно входит, что точно нет)
```

### Критерии остановки

Итерации brief заканчиваются, когда:
- Модель начинает спрашивать **тривиальные** вопросы (цвет кнопок, формулировки UI).
- Вы можете ответить "да" на вопрос: "Может ли другой инженер (или AI-агент) начать писать specs по этому brief без обращения ко мне?"
- Brief покрывает оба измерения: **domain** (что система делает) и **technical** (как она устроена).

### Структура brief

Brief должен содержать два раздела:

**Domain brief** — описание продукта на языке бизнеса:
- Что система делает (actors, scenarios, constraints).
- Как устроен домен (entities, relationships, lifecycle).
- Что входит в MVP, а что нет.

**Technical brief** — технические решения:
- Стек и архитектура.
- Подход к multi-tenancy.
- Аутентификация и авторизация.
- Подход к тестированию.
- Деплой и инфраструктура.

### Реконструированный brief для feedback-360

Ниже — brief, реконструированный из существующих specs, ADR и транскриптов автора. Это то, что было на входе у автора перед началом specs.

---

#### DOMAIN BRIEF

**Назначение**: feedback-360 — внутренняя корпоративная система для HR-оценки сотрудников методом 360 градусов. Система поддерживает HR в проведении оценки по лучшим практикам отрасли.

**Multi-tenancy**: несколько компаний в одной инсталляции. `company_id` присутствует почти во всех таблицах. Пользователь (User) может состоять в нескольких компаниях.

**Actors и роли**:
- **HR Admin** — создаёт и управляет кампаниями, видит все результаты.
- **HR Reader** — только чтение HR-данных.
- **Руководитель (Manager)** — видит результаты своих подчинённых.
- **Сотрудник (Employee)** — заполняет анкеты, видит свои результаты.

Важно: Employee (HR-сущность, привязана к компании) и User (Auth-сущность, привязана к email) — разные entities. User ↔ Employee связываются через `employee_user_links`.

**Оргструктура**:
- Департаменты с иерархией (`parent_id`).
- Привязка сотрудника к департаменту с историей (`employee_department_history`).
- Привязка руководителя с историей (`employee_manager_history`).
- Должности с историей (`employee_positions`).
- Ручной ввод оргструктуры (MVP — без HR-интеграций).

**Модели компетенций**:
- Версии моделей (`competency_model_versions`) со статусами `draft`/`published`.
- Группы компетенций с весами.
- Компетенции внутри групп.
- Два вида шкал: indicators (1..5 + NA) и levels (1..4 + UNSURE).
- Кампания привязывается к конкретной версии модели.

**Кампании**:
- Lifecycle: `draft` → `started` → `ended` → `processing_ai` → `ai_failed` | `completed` | `stopped`.
- При старте кампании: snapshot оргданных, генерация assignments, создание questionnaires.
- Freeze: матрица замораживается на первом draft-save анкеты (`locked_at`).
- Веса по группам оценщиков: manager (по умолчанию 40%), peers (30%), subordinates (30%), self (0%).
- Динамический пересчёт весов при отсутствии/скрытии групп.

**Матрица назначений**:
- Ручное назначение + автогенерация по оргструктуре.
- Типы отношений: `self`, `manager`, `peer`, `subordinate`.
- Freeze на первом draft-save.
- Добавление участников из департаментов (bulk).

**Анкеты (Questionnaires)**:
- Рейтер (оценщик) заполняет анкету по оцениваемому.
- Lifecycle: draft-save → submit. После submit — read-only.
- После окончания кампании — все анкеты read-only.
- Ответы: рейтинг по шкале + открытый комментарий по каждой компетенции.

**Результаты и анонимность**:
- Aggregation: средние значения по компетенциям/группам с весами.
- Anonymity threshold = 3: если в группе оценщиков < 3 человек, группа скрывается или объединяется с другой.
- Менеджер всегда НЕ анонимен.
- Self weight = 0% (самооценка показывается, но не влияет на итог).
- Видимость результатов: HR — всё, Manager — подчинённые, Employee — свои.
- Открытые комментарии: правила показа зависят от роли и anonymity policy.

**Уведомления**:
- MVP — email-only (через Resend).
- Outbox pattern + idempotency key для надёжной доставки.
- Reminder-ы по расписанию с учётом таймзоны компании.
- Invite при старте кампании.
- Telegram (`telegram_user_id/chat_id`) — закладываем в схему, реализуем позже.

**AI-обработка**:
- Внешний сервис постобработки открытых текстов.
- Запуск по `campaign_id` после `ended`.
- Webhook обратно: HMAC signature + `ai_job_id` + idempotency + retries.
- Статусы: `processing_ai` → `completed` | `ai_failed`.
- HR может перезапустить AI (retry button).

**MVP scope (IN)**:
- HR-справочник + оргструктура (ручной ввод).
- Модели компетенций (версии, indicators, levels).
- Кампании (full lifecycle).
- Матрица назначений (ручная + авто).
- Анкеты (draft/save/submit).
- Результаты с анонимностью.
- Email-уведомления.
- AI-постобработка (stub в MVP, real endpoint позже).
- Web UI (Next.js).
- CLI для automation и агентов.

**MVP scope (OUT)**:
- Telegram login / OAuth.
- HR-интеграции (1С, SAP).
- Кастомизация шаблонов анкет.
- Мобильное приложение.
- Экспорт отчётов в PDF/Excel (MVP показывает на экране).

#### TECHNICAL BRIEF

**Стек**:
- TypeScript monorepo (pnpm workspaces).
- Runtime: Node.js.
- Web: Next.js App Router + Tailwind CSS v4 + shadcn/ui.
- DB: Supabase PostgreSQL + Drizzle ORM.
- Email: Resend.
- Testing: Vitest (unit/integration), Playwright (e2e).
- Linting: Biome.
- Deploy: Vercel (web) + Supabase (DB).

**Архитектура (слои)**:
- **Core** — domain logic: use-cases, policies, state machines, calculations. Здесь живут инварианты.
- **API Contract** — typed DTO, operations, errors. SSoT контракта между слоями.
- **Client** — typed client с transport abstraction (HTTP / in-proc). Единый API для UI и CLI.
- **DB** — Drizzle schema, migrations, queries. Multi-tenant (`company_id`), soft delete, history.
- **CLI** — Commander, thin shell поверх Client. `--json` для automation.
- **Web** — Next.js App Router, thin UI поверх Client. Route handlers как adapters.

**Ключевые architectural decisions**:
- CLI-first: сначала core + contract + client + CLI, потом UI (ADR 0001).
- UI и CLI не содержат доменных правил — только отображение и вызовы operations.
- Аутентификация MVP: magic link (email). OAuth / Telegram — позже.
- Multi-tenancy через `company_id` + RLS (deny-by-default).
- Soft delete + history tables для оргданных.

**Подход к тестированию**:
- Unit: политики, расчёты, state machines (быстро, без БД).
- Integration: use-case + реальная БД (локальная Supabase).
- Contract: DTO schemas + golden payloads.
- E2E: Playwright только для критичных user journeys.
- Seed scenarios (S0-S9) с handles (именованные ID, не числовые).
- Golden scenarios (GS1-GS13) — минимальный набор для MVP confidence.

---

### Checkpoint Фазы 0

- [ ] Brief покрывает domain (actors, entities, lifecycle, rules) без критических пробелов.
- [ ] Brief покрывает technical (стек, архитектура, деплой) без критических пробелов.
- [ ] Модель перестала задавать нетривиальные вопросы.
- [ ] Другой инженер (или AI-агент) может начать specs без обращения к автору.

---

## Фаза 1: Создание MBB — "операционная система" документации

**Что делается**: До любых specs создаём правила ведения Memory Bank — "операционную систему" для документации проекта.

**Артефакты**: `.memory-bank/mbb/` — principles, templates, indexing rules.

**Почему MBB первым**: Из git history проекта видно, что MBB-коммиты (правила документации, шаблоны, индексы) были сделаны за 3-4 коммита до первых specs. Это не случайность: если начать писать specs без правил, документация расползается, появляются дубли, теряются связи.

MBB — это набор правил, по которым агент (и человек) ведёт документацию: где что хранить, как связывать, как индексировать, когда считать задачу закрытой.

### Промпт #2: Создание MBB principles

```
Я создаю Memory Bank — структурированную базу знаний проекта, которая
будет использоваться AI-агентами и инженерами для навигации, реализации
и проверки фич.

Мне нужен документ `.memory-bank/mbb/principles.md` — набор принципов
ведения Memory Bank.

Требования к принципам:

1. SINGLE SOURCE OF TRUTH — каждая концепция имеет один нормативный документ.
2. АТОМАРНОСТЬ — один файл = одна тема.
3. PROGRESSIVE DISCLOSURE — сначала обзор, потом детали.
4. WHY / WHAT / HOW разделение:
   - `spec/` — WHAT (нормативные требования)
   - `plans/` — delivery units (эпики, фичи, roadmap)
   - `adr/` — WHY (архитектурные решения)
   - `guides/` — user-facing документация
   - `mbb/` — правила ведения самого банка
5. NO DUPLICATION WITH CODE — не копировать структуру кода в документы.
6. INDEX-FIRST NAVIGATION — orphan файлы без входной ссылки = дефект.
7. ANNOTATED LINKS — каждая ссылка объясняет "что по ней" и "зачем читать".
8. KEEP DOCUMENTS SMALL — дробить, когда документ трудно читать.
9. COMMIT TAGGING — коммиты трассируемы к FT/EP через теги.
10. EVIDENCE-FIRST COMPLETION — фича не закрыта без evidence.
11. ROOT INDEX USEFULNESS — `index.md` содержит curated ссылки на 2-3
    уровня вниз, а не просто список папок.
12. GROUNDING-FIRST IMPLEMENTATION — агент обязан прочитать контекст до кода.

Формат: Markdown, каждый принцип — секция с номером и кратким пояснением.
Язык: русский с техническими терминами на английском.
Создай файл `.memory-bank/mbb/principles.md`.
```

### 17 принципов MBB (reference)

В реальном проекте MBB вырос до 17 принципов. Ключевые, которых нет в первоначальном промпте и которые появились позже:

- **#13 Visual references are inspiration, not behavior** — UI-макеты не заменяют specs.
- **#14 Boundary rationale must be documented** — новые архитектурные границы требуют и `spec/` и `adr/`.
- **#15 UI specs and POM mapping are first-class documentation** — screen specs для automation.
- **#16 Screen IDs are mandatory for UI traceability** — канонические идентификаторы экранов.
- **#17 Design system is SSoT for repeated visual language** — повторяющиеся UI-правила в design system.

Принципы 13-17 добавлялись по мере роста UI и потребности в traceability. Это нормально — MBB растёт вместе с проектом.

### Промпт #3: Генерация templates для epic/feature

```
Создай шаблоны документов для эпиков и фич в `.memory-bank/mbb/templates/`.

Нужны два файла:

1. `epic.md` — шаблон эпика:
   - Goal (user value) — кому и какую ценность.
   - Scope (in / out).
   - Features (vertical slices) — список фич со ссылками.
   - Progress report (evidence-based) — as_of, total, completed, evidence.
   - Dependencies.
   - Risks & mitigations.
   - Definition of done — все фичи имеют сценарии, golden scenarios покрывают
     риски, evidence записано.

2. `feature.md` — шаблон фичи (vertical slice):
   - Traceability (mandatory) — epic link, PR link, commits.
   - User value.
   - Deliverables — contract/core/db/cli/ui.
   - Context (SSoT links) — аннотированные ссылки на specs.
   - Project grounding (mandatory) — чеклист того, что агент прочитал.
   - Implementation plan.
   - Scenarios (auto acceptance) в формате Setup/Action/Assert:
     - Setup: seed scenario + actors.
     - Action: CLI/API шаги с `--json`.
     - Assert: конкретные проверки (статусы, RBAC, error codes).
   - Tests — unit/integration/contract/e2e.
   - Docs updates — какие specs обновить.
   - Quality checks evidence — `pnpm checks` results.
   - Acceptance evidence — commands, results, date.

Формат: Markdown с placeholder-ами `<EP-XXX>`, `<FT-XXX-YY>`.
Принципы: handles вместо UUID в сценариях, `--json` для детерминизма,
Setup/Action/Assert структура.
```

### Структура директорий MBB

После Фазы 1 в проекте должна появиться такая структура:

```
.memory-bank/
├── index.md                 # Root index (пока минимальный)
├── mbb/
│   ├── index.md             # Навигация по MBB
│   ├── principles.md        # 12+ принципов
│   ├── indexing.md           # Правила построения индексов
│   ├── duo-pattern.md        # Когда дробить на summary + detail
│   └── templates/
│       ├── index.md
│       ├── epic.md           # Шаблон эпика
│       └── feature.md        # Шаблон фичи
├── spec/                     # (пока пустой, заполнится в Фазе 2)
├── plans/                    # (пока пустой, заполнится в Фазе 3)
└── adr/                      # (пока пустой)
```

### Checkpoint Фазы 1

- [ ] `.memory-bank/mbb/principles.md` содержит 12+ принципов.
- [ ] Шаблоны `epic.md` и `feature.md` готовы.
- [ ] Root `index.md` создан и навигируем.
- [ ] Агент, прочитавший `mbb/principles.md`, понимает правила ведения документации.

---

## Фаза 2: Brief → Specifications

**Что делается**: Brief конвертируется в полные specs через агента с file-writing tools.

**Артефакты**: `.memory-bank/spec/` — 90+ файлов по домену, безопасности, engineering, testing, client-api, cli, ui.

**Ключевой переход**: От "chat" к "agent with tools". Specs требуют создания файлов — здесь впервые нужен агент с возможностью писать в файловую систему (Claude с Cursor, Claude Code, или аналогичный инструмент).

### Порядок создания specs

Specs создаются в определённом порядке, где каждый слой опирается на предыдущий:

1. **Domain specs** — бизнес-правила (campaign lifecycle, assignments, anonymity, calculations).
2. **Security specs** — auth, RBAC, RLS.
3. **Engineering specs** — architecture guardrails, coding style, testing standards.
4. **API и subsystem specs** — operations, CLI, notifications, AI.
5. **Testing specs** — golden scenarios, seed scenarios, traceability.

### Промпт #4: Конвертация brief в domain specs

```
У меня есть brief проекта feedback-360 (см. ниже). Конвертируй его в
полную domain specification.

Создай файлы в `.memory-bank/spec/`:

1. `project/system-overview.md` — общая картина продукта (1 страница).
2. `project/mvp-scope.md` — что входит/не входит в MVP.
3. `project/layers-and-vertical-slices.md` — определение слоёв и vertical
   slice DoD.
4. `c4/index.md` — C4 architecture:
   - L1: System Context (actors, external systems).
   - L2: Container (web, cli, core, db, email, AI service).
   - L3: Component (packages, features).
5. `domain/index.md` — оглавление domain specs с annotated links.
6. `domain/campaign-lifecycle.md` — статусы, переходы, freeze-правила.
7. `domain/assignments-and-matrix.md` — назначения, автогенерация,
   ограничения.
8. `domain/org-structure.md` — оргданные, snapshot при старте кампании.
9. `domain/questionnaires.md` — анкеты, draft/submit, read-only правила.
10. `domain/competency-models.md` — версии модели, два вида шкал.
11. `domain/calculations.md` — формулы агрегации (weights, distributions).
12. `domain/anonymity-policy.md` — threshold=3, скрытие/слияние групп,
    правила показа текста.
13. `domain/soft-delete-and-history.md` — is_active/deleted_at, историзация.
14. `domain/results-visibility.md` — кто что видит (HR/manager/employee).
15. `security/index.md` — оглавление security specs.
16. `security/auth-and-identity.md` — magic link, User vs Employee.
17. `security/rbac.md` — матрица ролей и разрешений.
18. `security/rls.md` — Row Level Security, deny-by-default.
19. `engineering/architecture-guardrails.md` — запреты на импорт core
    в UI/CLI, vertical slice conventions.
20. `engineering/coding-style.md` — TypeScript, Biome, error format.
21. `engineering/testing-standards.md` — уровни тестов, quality gates.

Правила:
- Каждый файл описывает ОДНУ тему (атомарность).
- `index.md` содержит annotated links (что по ссылке + зачем читать).
- Формат: Markdown с `Status: Draft (YYYY-MM-DD)` в начале.
- Язык: русский, тех. термины на английском.
- Не копируй код — давай ссылки и описывай инварианты.

[Вставить brief из Фазы 0]
```

### Промпт #5: Создание API и subsystem specs

```
На основе domain specs (уже созданных в `.memory-bank/spec/domain/`)
создай спецификации API, CLI и подсистем.

Создай файлы:

1. `spec/client-api/index.md` — оглавление с annotated links.
2. `spec/client-api/operation-catalog.md` — нормативный список всех
   операций:
   - Формат: `operation.name` | Input type | Output type | Required role
   - Группировка по feature areas: system, identity, org, models,
     campaigns, matrix, questionnaires, results, notifications, ai, ops.
3. `spec/client-api/transport.md` — HTTP vs in-proc, единый endpoint
   `/api/v1/operations`, формат request/response.
4. `spec/client-api/error-model.md` — typed errors (`code/message/details`),
   список error codes.
5. `spec/cli/index.md` — оглавление.
6. `spec/cli/command-catalog.md` — mapping `CLI command → operation`,
   `--json` формат, exit codes.
7. `spec/notifications/index.md` — события, outbox/idempotency,
   RU-шаблоны, расписания.
8. `spec/ai/index.md` — AI job lifecycle, webhook security, retry.
9. `spec/operations/index.md` — git flow, deployment architecture.

Правила:
- Операции именуются как `area.entity.action` (напр. `campaign.start`,
  `model.version.create`).
- Typed errors: конечный набор `OperationErrorCode`.
- CLI: thin shell, 1:1 mapping к operations, human по умолчанию,
  `--json` для automation.
- Notification outbox: idempotency_key, attempts, retry schedule.
```

### Промпт #6: Testing strategy и traceability

```
Создай спецификацию тестирования для feedback-360.

Файлы:

1. `spec/testing/index.md` — оглавление testing specs.
2. `spec/testing/golden-scenarios.md` — минимальный набор сквозных
   сценариев для MVP confidence:
   - GS1: Happy path — полный цикл от создания кампании до результатов.
   - GS4: Freeze на первом draft-save.
   - GS5: Anonymity threshold (< 3 оценщика → группа скрыта).
   - GS6: RBAC — employee не может видеть чужие результаты.
   - GS8: Notification outbox — invite при старте кампании.
   - GS10: RLS smoke — deny-by-default.
   - GS11-GS13: Campaign transitions, weights, AI lifecycle.
3. `spec/testing/seed-scenarios.md` — детерминированные seed сценарии:
   - S0_empty — чистая БД.
   - S1_company_min — минимальная компания с HR.
   - S2_org_basic — оргструктура (департаменты, руководители).
   - S4_campaign_draft — кампания в статусе draft.
   - S5_campaign_started_no_answers — запущенная кампания без ответов.
   - S6_campaign_started_some_drafts — есть черновики анкет.
   - S7_campaign_started_some_submitted — часть анкет отправлена.
   - S8_campaign_ended — кампания завершена.
   - S9_campaign_completed_with_ai — AI обработка завершена.
   Принципы: handles вместо UUID, deterministic data, variant support.
4. `spec/testing/traceability.md` — маппинг: domain invariant → test →
   seed scenario. Каждый инвариант из domain specs должен быть покрыт
   тестом и иметь seed для воспроизведения.

Формат golden scenario:
- Precondition: seed + actors.
- Steps: CLI/API операции.
- Expected: конкретные проверки.
- Covers: список domain invariants.
```

### Реальная структура specs из проекта

После Фазы 2 в проекте появляется такая структура (показаны ключевые файлы):

```
.memory-bank/spec/
├── index.md
├── glossary.md
├── project/
│   ├── system-overview.md
│   ├── mvp-scope.md
│   ├── layers-and-vertical-slices.md
│   ├── repo-structure.md
│   └── feature-area-boundaries.md
├── c4/
│   └── index.md (L1/L2/L3)
├── domain/
│   ├── index.md
│   ├── campaign-lifecycle.md
│   ├── assignments-and-matrix.md
│   ├── org-structure.md
│   ├── questionnaires.md
│   ├── competency-models.md
│   ├── calculations.md
│   ├── anonymity-policy.md
│   ├── soft-delete-and-history.md
│   └── results-visibility.md
├── security/
│   ├── index.md
│   ├── auth-and-identity.md
│   ├── rbac.md
│   └── rls.md
├── engineering/
│   ├── architecture-guardrails.md
│   ├── coding-style.md
│   ├── testing-standards.md
│   ├── delivery-standards.md
│   └── documentation-standards.md
├── client-api/
│   ├── index.md
│   ├── operation-catalog.md
│   ├── transport.md
│   └── error-model.md
├── cli/
│   ├── index.md
│   └── command-catalog.md
├── testing/
│   ├── index.md
│   ├── golden-scenarios.md
│   ├── seed-scenarios.md
│   └── traceability.md
├── notifications/
│   └── index.md
├── ai/
│   └── index.md
└── operations/
    ├── index.md
    ├── git-flow.md
    └── deployment-architecture.md
```

### Checkpoint Фазы 2

- [ ] Полная спецификация до единой строки кода.
- [ ] Domain specs покрывают все entities и lifecycle rules.
- [ ] Security specs определяют RBAC matrix и RLS strategy.
- [ ] Operation catalog содержит все planned operations.
- [ ] Testing specs определяют golden scenarios и seed scenarios.
- [ ] Все `index.md` содержат annotated links.

---

## Фаза 3: Specs → Epics/Features

**Что делается**: Specs нарезаются на delivery units — epics и features.

**Артефакты**: `.memory-bank/plans/` — roadmap, epics, features, playbook, verification matrix.

### Промпт #7: Нарезка specs на epics

```
На основе specs в `.memory-bank/spec/` нарежь работу на эпики.

Правила нарезки:
- Эпик = value-oriented группа фич, которые вместе дают meaningful
  capability пользователю.
- Эпики упорядочены по зависимостям: foundation → core delivery → domain
  → UI → hardening.
- Каждый эпик содержит 3-8 features (vertical slices).
- Используй формат из `.memory-bank/mbb/templates/epic.md`.

Создай файлы:
1. `plans/roadmap.md` — порядок эпиков с обоснованием.
2. `plans/epics.md` — полный список EP/FT.
3. `plans/epics/EP-000-foundation/index.md`
4. `plans/epics/EP-001-core-contract-client-cli/index.md`
5. `plans/epics/EP-002-identity-tenancy-rbac/index.md`
6. `plans/epics/EP-003-org-snapshots/index.md`
7. `plans/epics/EP-004-models-campaigns-questionnaires/index.md`
8. `plans/epics/EP-005-results-aggregation/index.md`
9. `plans/epics/EP-006-notifications/index.md`
10. `plans/epics/EP-007-ai-processing/index.md`
11. Для каждого эпика: `features/index.md` со списком FT.

Рекомендуемый порядок epics для MVP:
1. EP-000 Foundation — scaffold, DB, seeds.
2. EP-001 Core + Contract + Client + CLI — operation framework.
3. EP-002 Identity + Tenancy + RBAC — multi-tenant, auth, roles.
4. EP-003 Org + Snapshots — departments, reporting lines.
5. EP-004 Models + Campaigns + Questionnaires.
6. EP-005 Results + Aggregation + Anonymity.
7. EP-006 Notifications.
8. EP-007 AI Processing.
```

### Реальные 8 эпиков проекта

В реальном проекте эпики выглядят так (из `plans/roadmap.md`):

| # | Epic | Описание |
|---|------|----------|
| EP-000 | Foundation | Monorepo scaffold, DB baseline, seeds, web scaffold, Sentry |
| EP-001 | Core + Contract + Client + CLI | Operation dispatcher, typed errors, client transport, CLI |
| EP-002 | Identity + Tenancy + RBAC | Multi-tenant, auth, membership, RLS |
| EP-003 | Org + Snapshots | Departments, reporting lines, campaign snapshots |
| EP-004 | Models + Campaigns + Questionnaires | Competency models, campaign lifecycle, matrix, questionnaires |
| EP-005 | Results + Aggregation | Calculations, anonymity, visibility |
| EP-006 | Notifications | Outbox, reminders, templates, delivery |
| EP-007 | AI Processing | AI jobs, webhook, retry |

После MVP delivery добавляются ещё эпики: UI wave (EP-011..EP-019), UI traceability (EP-021), Visual system (EP-022), Documentation hardening (EP-023).

### Промпт #8: Детализация features

```
Для эпика EP-000 (Foundation) создай детальные feature documents.

Используй шаблон из `.memory-bank/mbb/templates/feature.md`.

Features для EP-000:
1. FT-0001 Workspace scaffold — pnpm workspace, packages, apps, biome, TS.
2. FT-0002 DB migrations baseline — Drizzle schema, migrations, health check.
3. FT-0003 Seed runner + handles — deterministic seed contract.
4. FT-0004 Domains + DNS + Resend — email provider setup.
5. FT-0005 Web app router scaffold — Next.js baseline.
6. FT-0006 Sentry integration — error reporting baseline.

Для каждой FT создай файл `plans/epics/EP-000-foundation/features/FT-XXXX-name/index.md`.

Каждая FT должна содержать:
- User value — что это даёт.
- Deliverables — конкретные артефакты.
- Context (SSoT links) — что прочитать перед реализацией.
- Scenarios — acceptance в формате Setup/Action/Assert.
- Tests — что тестировать.
- Docs updates — какие specs обновить.

Правила:
- Handles вместо UUID в сценариях.
- `--json` формат для CLI acceptance.
- Setup/Action/Assert структура сценариев.
- Deliverables группированы по слоям: contract/core/db/client/cli/tests/docs.
```

### Implementation playbook — 7-слойный чеклист

В реальном проекте каждая FT проходит через 7-слойный чеклист из `implementation-playbook.md`:

| Слой | Что делать | Target files |
|------|-----------|-------------|
| 0. Grounding | Прочитать FT + SSoT ссылки + каталоги | FT document |
| 1. Contract | Операция, DTO, ошибки | `packages/api-contract/src/<area>.ts` |
| 2. Core | Use-case + policy + инварианты | `packages/core/src/features/<area>.ts` |
| 3. Data/DB | Schema + migrations + RLS | `packages/db/src/schema/*.ts` |
| 4. Adapters | HTTP handlers, auth | `apps/web/src/app/api/...` |
| 5. Client | Transport + удобные методы | `packages/client/src/features/<area>.ts` |
| 6. CLI | Команда 1:1 к операции | `packages/cli/src/...` |
| 7. Tests | Unit/integration/contract/e2e | `packages/*/tests/` |

После всех слоёв: `pnpm checks` + acceptance scenario + evidence.

### Verification matrix

Verification matrix маппит features → scenarios → evidence:

```
EP-000 / FT-0001 → pnpm install succeeds → ✅ 2026-03-03
EP-000 / FT-0002 → pnpm db:health returns ok → ✅ 2026-03-03
EP-001 / FT-0011 → system.ping --json returns { ok: true } → ✅ 2026-03-04
...
```

### Checkpoint Фазы 3

- [ ] Roadmap определяет порядок и зависимости epics.
- [ ] Каждый epic имеет Goal, Scope, Features, DoD.
- [ ] Каждая FT имеет сценарий в формате Setup/Action/Assert.
- [ ] Implementation playbook описывает 7-слойный чеклист.
- [ ] Verification matrix готова к заполнению evidence.
- [ ] Delivery path прослеживается от specs через epics/features до конкретных файлов.

---

## Фаза 4: AGENTS.md + Context Warming

**Что делается**: Подготовка проекта для агентов-исполнителей. Создание инструкции, которая позволит AI-агенту самостоятельно навигировать по проекту.

**Артефакты**: `AGENTS.md`, `.memory-bank/index.md` (curated quick-start).

### Промпт #9: Создание AGENTS.md

```
Создай файл `AGENTS.md` в корне проекта — инструкцию для AI-агентов,
работающих с этим репозиторием.

AGENTS.md должен содержать:

1. QUICK START — что прочитать первым:
   - `.memory-bank/index.md` — главный вход в знание проекта.
   - `spec/project/system-overview.md` — быстрый domain context.
   - `spec/engineering/architecture-guardrails.md` — что запрещено.
   - `plans/implementation-playbook.md` — как реализовывать фичи.

2. PRINCIPLES — ключевые правила работы:
   - Specs-first: читай spec до кода.
   - Grounding-first: прочитай FT document и SSoT ссылки до реализации.
   - Evidence-first: фича не закрыта без checks + acceptance.
   - Thin clients: UI/CLI не содержат доменных правил.

3. WORKFLOW — как агент работает с фичей:
   - Прочитать FT document.
   - Пройти project grounding (SSoT links).
   - Реализовать по слоям (contract → core → db → client → cli → tests).
   - Запустить `pnpm checks`.
   - Прогнать acceptance scenario.
   - Записать evidence.

4. KEY COMMANDS:
   - `pnpm checks` — полный quality gate.
   - `pnpm test:db` — integration tests с БД.
   - `pnpm db:health` — проверка подключения к БД.
   - `pnpm seed` — запуск seed runner.

5. NAVIGATION — как находить информацию:
   - Specs: `.memory-bank/spec/`
   - Plans: `.memory-bank/plans/`
   - ADR: `.memory-bank/adr/`
   - Templates: `.memory-bank/mbb/templates/`
```

### Техника context warming

**Context warming** — это загрузка ключевой информации в контекст агента перед началом работы. В feedback-360 это реализовано через `.memory-bank/index.md`.

Root index — не просто "список папок". Это **curated quick-start**, где каждая ссылка аннотирована: "что по ссылке" и "зачем читать". В реальном проекте `index.md` содержит ~100 аннотированных ссылок, сгруппированных по темам:

```markdown
## Quick start (for agents)
- [System overview](spec/project/system-overview.md): краткая картина
  продукта, акторов и ключевых ограничений MVP. Читать для быстрого
  доменного контекста перед реализацией.
- [Architecture guardrails](spec/engineering/architecture-guardrails.md):
  границы core/client/web и правила слоёв+vertical slices. Читать
  перед кодом, чтобы не утащить бизнес-логику в UI/CLI.
- [Implementation playbook](plans/implementation-playbook.md): пошаговый
  чеклист реализации фичи. Читать как рабочую инструкцию.
```

Ключевой приём: **annotated links** — "что по ссылке" и "зачем читать". Это позволяет агенту принимать решение о том, какие документы читать, не открывая каждый файл.

### Checkpoint Фазы 4

- [ ] `AGENTS.md` содержит quick start, principles, workflow, commands, navigation.
- [ ] `.memory-bank/index.md` — curated quick-start с annotated links.
- [ ] Агент, прочитавший `AGENTS.md` + `index.md`, понимает проект и может начать работу.

---

## Фаза 5: Workspace Scaffold + DB Baseline

**Что делается**: Первый код — monorepo scaffold, DB schema, migrations, health check.

**Артефакты**: `packages/`, `apps/`, migrations, health check endpoint.

Здесь начинается переход от документации к коду. Важно: агент начинает с **grounding** — чтения FT-документа и связанных specs.

### Промпт #10: FT-0001 Workspace scaffold

```
Реализуй FT-0001 — Workspace scaffold.

GROUNDING (прочитай перед реализацией):
- [ ] `.memory-bank/plans/epics/EP-000-foundation/features/FT-0001-workspace-scaffold/index.md`
- [ ] `.memory-bank/spec/project/repo-structure.md`
- [ ] `.memory-bank/spec/engineering/coding-style.md`
- [ ] `.memory-bank/spec/engineering/architecture-guardrails.md`

DELIVERABLES:
1. Root `package.json` с scripts:
   - `lint` — `pnpm -r lint`
   - `typecheck` — `pnpm -r typecheck`
   - `test` — `pnpm -r test`
   - `checks` — полный pipeline (lint + typecheck + test + test:db + build)
   - `format` — `pnpm -r format`
2. `pnpm-workspace.yaml` с packages: `packages/*`, `apps/*`.
3. Root `tsconfig.json`.
4. Root `biome.json` — linter/formatter config.
5. `.gitignore`.
6. Packages (пустые scaffolds с package.json + tsconfig.json):
   - `packages/api-contract` — typed operations, DTO, errors.
   - `packages/core` — domain logic, dispatcher.
   - `packages/client` — typed client, transport.
   - `packages/db` — Drizzle schema, migrations.
   - `packages/cli` — Commander CLI.
7. Apps:
   - `apps/web` — Next.js App Router scaffold.

ACCEPTANCE:
- `pnpm install` проходит без ошибок.
- `pnpm lint` проходит (пустой, но без ошибок).
- `pnpm typecheck` проходит.
- Структура packages/apps соответствует spec.

COMMIT: `[FT-0001] Workspace scaffold`
```

В реальном проекте `package.json` выглядит так:

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
    "test:db": "pnpm --filter @feedback-360/db test:db && pnpm --filter @feedback-360/client test:db && pnpm --filter @feedback-360/core test:db",
    "checks": "pnpm lint && pnpm typecheck && pnpm test && pnpm test:db && pnpm --filter @feedback-360/web build",
    "format": "pnpm -r format",
    "db:migrate": "pnpm --filter @feedback-360/db db:migrate",
    "db:health": "pnpm --filter @feedback-360/db db:health",
    "seed": "pnpm --filter @feedback-360/cli cli --"
  },
  "devDependencies": {
    "@biomejs/biome": "1.9.4",
    "@playwright/test": "1.54.2",
    "@types/node": "22.13.8",
    "cross-env": "7.0.3",
    "typescript": "5.8.2",
    "vitest": "3.2.4"
  }
}
```

### Промпт #11: FT-0002 DB baseline

```
Реализуй FT-0002 — DB migrations baseline.

GROUNDING (прочитай перед реализацией):
- [ ] `.memory-bank/plans/epics/EP-000-foundation/features/FT-0002-db-migrations-baseline/index.md`
- [ ] `.memory-bank/spec/domain/index.md` (все domain specs для schema)
- [ ] `.memory-bank/spec/domain/campaign-lifecycle.md`
- [ ] `.memory-bank/spec/domain/org-structure.md`
- [ ] `.memory-bank/spec/domain/competency-models.md`
- [ ] `.memory-bank/spec/domain/soft-delete-and-history.md`
- [ ] `.memory-bank/spec/security/rls.md`

DELIVERABLES:
1. `packages/db/src/schema/tables.ts` — полная Drizzle schema:
   - companies (id, name, timezone, is_active, deleted_at)
   - company_memberships (company_id, user_id, role)
   - employees (company_id, email, names, phone, telegram, is_active)
   - employee_user_links (company_id, employee_id, user_id)
   - departments (company_id, parent_id, name, is_active)
   - employee_department_history (employee_id, department_id, start_at, end_at)
   - employee_manager_history (employee_id, manager_employee_id, start_at, end_at)
   - employee_positions (employee_id, title, level, start_at, end_at)
   - competency_model_versions (company_id, name, kind, version, status)
   - competency_groups (model_version_id, name, weight, order)
   - competencies (group_id, name, order)
   - competency_indicators (competency_id, text, order)
   - competency_levels (competency_id, level, text)
   - campaigns (company_id, model_version_id, name, status, dates, weights, locked_at)
   - campaign_employee_snapshots (campaign_id, employee data snapshot)
   - ai_jobs (campaign_id, provider, status, idempotency)
   - ai_webhook_receipts (ai_job_id, idempotency)
   - notification_outbox (campaign_id, recipient, channel, status, idempotency)
   - notification_attempts (outbox_id, attempt_no, status)
   - notification_settings (company_id, schedule, locale)
   - audit_events (company_id, event_type, summary)

2. Migrations: `packages/db/src/migrations/` — initial migration.
3. Health check: `packages/db/src/health.ts` — проверка подключения.

KEY PATTERNS:
- `company_id` почти на всех таблицах (multi-tenant).
- Soft delete: `is_active` + `deleted_at` (не удаляем физически).
- History tables: `start_at` + `end_at` (для оргданных).
- Idempotency keys на notifications и AI webhooks.
- UUID primary keys (defaultRandom).
- Timestamps: `created_at`, `updated_at` with timezone.

ACCEPTANCE:
- `pnpm db:migrate` проходит.
- `pnpm db:health` возвращает ok.
- Schema соответствует domain specs.

COMMIT: `[FT-0002] DB migrations baseline`
```

### Checkpoint Фазы 5

- [ ] `pnpm install` проходит без ошибок.
- [ ] `pnpm db:health` возвращает ok.
- [ ] Schema покрывает все entities из domain specs.
- [ ] Multi-tenancy (`company_id`) на всех таблицах.
- [ ] Soft delete + history tables по spec.

---

## Фаза 6: Core + Contract + Client + CLI — Delivery Loop

**Что делается**: Строится non-UI delivery path — typed operations end-to-end. Это ключевой момент архитектуры: все бизнес-операции становятся доступны через единый typed contract, и их можно вызывать как из CLI, так и из UI.

**Артефакты**: `api-contract`, `core` (dispatcher + handlers), `client` (transport), CLI.

**Почему это перед UI**: ADR 0001 фиксирует: "Сначала строим core use-cases + typed contract + typed client + CLI, и только потом UI." Причина — логику можно тестировать быстрее и дешевле (без браузера), CLI удобен для AI-агента, UI и CLI остаются thin.

### Промпт #12: FT-0011 api-contract + dispatcher

```
Реализуй FT-0011 — Operation plumbing + errors.

GROUNDING:
- [ ] `.memory-bank/plans/epics/EP-001-core-contract-client-cli/features/FT-0011-op-plumbing-errors/index.md`
- [ ] `.memory-bank/spec/client-api/operation-catalog.md`
- [ ] `.memory-bank/spec/client-api/transport.md`
- [ ] `.memory-bank/spec/client-api/error-model.md`
- [ ] `.memory-bank/spec/engineering/architecture-guardrails.md`

DELIVERABLES:

1. `packages/api-contract/src/v1/legacy.ts` — runtime types и parsers:
   ```typescript
   // Typed error codes
   export const operationErrorCodes = [
     "invalid_input", "unauthenticated", "forbidden",
     "not_found", "invalid_transition",
     "campaign_started_immutable", "campaign_locked",
     "campaign_ended_readonly",
     "webhook_invalid_signature", "webhook_timestamp_invalid",
     "ai_job_conflict",
   ] as const;
   export type OperationErrorCode = (typeof operationErrorCodes)[number];

   // Typed result (discriminated union)
   export type OperationResult<Output> =
     | { ok: true; data: Output }
     | { ok: false; error: OperationError };

   // Operation context (multi-tenant)
   export type OperationContext = {
     userId?: string;
     employeeId?: string;
     companyId?: string;
     role?: MembershipRole;
   };

   // Dispatch input
   export type DispatchOperationInput = {
     operation: string;
     input: unknown;
     context?: OperationContext;
   };

   // Known operations (exhaustive list)
   export const knownOperations = [
     "system.ping", "seed.run",
     "company.updateProfile", "membership.list",
     // ...все операции из operation catalog
   ] as const;
   export type KnownOperation = (typeof knownOperations)[number];
   ```

2. `packages/core/src/index.ts` — dispatcher:
   ```typescript
   // Operation handler type
   type OperationHandler = (
     request: DispatchOperationInput,
   ) => Promise<OperationResult<DispatchOutput>>
      | OperationResult<DispatchOutput>;

   // Handler registry
   const operationHandlers: Partial<Record<KnownOperation, OperationHandler>> = {
     "system.ping": (request) => runSystemPing(request.input),
     // ...handlers per operation
   };

   // Dispatch function
   export const dispatchOperation = (
     request: DispatchOperationInput,
   ): Promise<OperationResult<DispatchOutput>> => {
     // 1. Parse & validate request
     // 2. Check if operation is known
     // 3. Find handler
     // 4. Execute handler
     // 5. Return typed result
   };
   ```

3. Первый handler — `system.ping`:
   ```typescript
   export const runSystemPing = (
     input: unknown
   ): OperationResult<SystemPingOutput> => {
     parseSystemPingInput(input);
     return okResult(parseSystemPingOutput({
       pong: "ok",
       timestamp: new Date().toISOString(),
     }));
   };
   ```

KEY PATTERNS:
- `OperationResult<T>` = discriminated union (`ok: true/false`).
- Every operation has typed Input/Output + parse function.
- Dispatcher: string operation name → handler lookup → typed result.
- Error model: `code` + `message` + `details` (structured, не raw exceptions).
- `seed.run` и `client.setActiveCompany` — client-local, не в dispatcher.

ACCEPTANCE:
- Dispatcher resolves `system.ping` → `{ ok: true, data: { pong: "ok" } }`.
- Unknown operation → `{ ok: false, error: { code: "not_found" } }`.
- Invalid input → `{ ok: false, error: { code: "invalid_input" } }`.

COMMIT: `[FT-0011] Operation plumbing and typed errors`
```

### Промпт #13: FT-0012 Client transport + runtime

```
Реализуй FT-0012 — Typed client transport.

GROUNDING:
- [ ] `.memory-bank/plans/epics/EP-001-core-contract-client-cli/features/FT-0012-typed-client-transport/index.md`
- [ ] `.memory-bank/spec/client-api/transport.md`
- [ ] `.memory-bank/spec/engineering/architecture-guardrails.md`
- [ ] Уже реализованный `packages/api-contract/`

DELIVERABLES:

1. `packages/client/src/shared/runtime.ts` — transport abstraction:
   ```typescript
   // Transport interface (HTTP or in-proc)
   export type OperationTransport = {
     invoke(request: DispatchOperationInput): Promise<unknown>;
   };

   // In-proc transport (calls core directly)
   export const createInprocTransport = (): OperationTransport => ({
     invoke: async (request) => dispatchOperation(request),
   });

   // HTTP transport (calls API endpoint)
   export const createHttpTransport = (
     options: CreateHttpTransportOptions
   ): OperationTransport => ({
     invoke: async (request) => {
       const response = await fetch(endpointUrl, {
         method: "POST",
         headers: { "content-type": "application/json" },
         body: JSON.stringify(request),
       });
       return response.json();
     },
   });

   // Client runtime (wraps transport + active context)
   export type ClientRuntime = {
     invokeOperation: <Output>(params: InvokeOperationParams<Output>)
       => Promise<OperationResult<Output>>;
     setActiveContext: (context: OperationContext)
       => OperationResult<OperationContext>;
     getActiveContext: () => OperationContext;
     setActiveCompany: (companyId: string)
       => OperationResult<ClientSetActiveCompanyOutput>;
   };
   ```

2. `packages/client/src/index.ts` — client factory:
   ```typescript
   // Composite client type (all feature methods)
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

   // Factory
   export const createClient = (
     transport: OperationTransport
   ): Feedback360Client => {
     const runtime = createClientRuntime(transport);
     return {
       invokeOperation: runtime.invokeOperation,
       ...createSystemClientMethods(runtime),
       ...createIdentityTenancyClientMethods(runtime),
       // ...other feature areas
     };
   };

   // Convenience: in-proc client for CLI / tests
   export const createInprocClient = (): Feedback360Client =>
     createClient(createInprocTransport());
   ```

3. Feature-area client methods (пример):
   ```typescript
   // packages/client/src/features/identity-tenancy.ts
   export const createIdentityTenancyClientMethods = (
     runtime: ClientRuntime
   ) => ({
     systemPing: (context?: OperationContext) =>
       runtime.invokeOperation({
         operation: "system.ping",
         input: {},
         context,
         parseOutput: parseSystemPingOutput,
       }),
   });
   ```

KEY PATTERNS:
- Один contract для HTTP и in-proc (одинаковые DTO/ошибки).
- Client runtime хранит active context (companyId, userId, role).
- Feature-area methods — typed wrappers вокруг `invokeOperation`.
- Client НЕ содержит бизнес-правил — только вызов + parse.

ACCEPTANCE:
- `createInprocClient().systemPing()` → `{ ok: true, data: { pong: "ok" } }`.
- `createHttpTransport({ baseUrl: "..." })` создаётся без ошибок.
- Client methods typed — TypeScript ловит ошибки типов.

COMMIT: `[FT-0012] Typed client transport and runtime`
```

### Промпт #14: FT-0013 CLI-first vertical slice

```
Реализуй FT-0013 — CLI-first vertical slice.

GROUNDING:
- [ ] `.memory-bank/plans/epics/EP-001-core-contract-client-cli/features/FT-0013-questionnaire-ops-cli/index.md`
- [ ] `.memory-bank/spec/cli/command-catalog.md`
- [ ] `.memory-bank/spec/cli/index.md`
- [ ] Уже реализованные `packages/api-contract/`, `packages/core/`, `packages/client/`

DELIVERABLES:

1. `packages/cli/src/index.ts` — Commander-based CLI:
   ```typescript
   const program = new Command();
   program
     .name("feedback-360")
     .description("feedback-360 CLI")
     .version("0.1.0");

   // System commands
   program.command("ping")
     .description("Check system health")
     .option("--json", "Output as JSON")
     .action(async (options) => {
       const client = createInprocClient();
       const result = await client.systemPing();
       if (options.json) {
         console.log(JSON.stringify(result, null, 2));
       } else {
         console.log(result.ok ? "pong" : `Error: ${result.error.message}`);
       }
     });

   // Seed command
   program.command("seed")
     .argument("<scenario>", "Seed scenario name")
     .option("--json", "Output as JSON")
     .action(async (scenario, options) => {
       const client = createInprocClient();
       const result = await client.seedRun({ scenario });
       // ...output
     });
   ```

2. Root `package.json` script:
   ```json
   "seed": "pnpm --filter @feedback-360/cli cli --"
   ```

3. Первый end-to-end тест: `system.ping` через все слои:
   - CLI → Client (in-proc) → Core dispatcher → handler → response.

KEY PATTERNS:
- CLI по умолчанию human-readable, `--json` отдаёт stable schema.
- Команда НЕ содержит доменных правил — только сбор args + вызов client.
- Commander для парсинга команд и аргументов.
- In-proc client для CLI (не HTTP).

ACCEPTANCE:
- `pnpm --filter @feedback-360/cli cli -- ping --json` →
  `{ "ok": true, "data": { "pong": "ok", "timestamp": "..." } }`
- `pnpm seed S0_empty --json` → seed runner works.
- `pnpm checks` проходит.

COMMIT: `[FT-0013] CLI-first vertical slice`
```

### Checkpoint Фазы 6

- [ ] CLI `system.ping --json` возвращает `{ ok: true, data: { pong: "ok" } }`.
- [ ] In-proc и HTTP transports создаются без ошибок.
- [ ] Seed runner работает.
- [ ] `pnpm checks` проходит.
- [ ] Все слои typed: contract → core → client → CLI.

---

## Фаза 7: Domain Features — Vertical Slices

**Что делается**: Бизнес-фичи реализуются как серия vertical slices. Каждый slice проходит через полный 7-слойный чеклист.

**Артефакты**: 40+ operations across identity, org, models, campaigns, questionnaires, results, notifications, AI.

### Промпт #15: Шаблон реализации одной FT

Этот промпт используется для **каждой** domain feature. Меняются только ссылки на FT-документ и конкретные операции.

```
Реализуй фичу [FT-XXXX — Name].

## 1. PROJECT GROUNDING (обязательно перед кодом)

Прочитай и зафиксируй что прочитано:
- [ ] FT-документ: `.memory-bank/plans/epics/[EP-XXX]/features/[FT-XXXX]/index.md`
- [ ] Связанные SSoT документы из секции `Context (SSoT links)` в FT.
- [ ] Operation catalog: `.memory-bank/spec/client-api/operation-catalog.md`
- [ ] CLI command catalog: `.memory-bank/spec/cli/command-catalog.md`
- [ ] Traceability: `.memory-bank/spec/testing/traceability.md`
- [ ] Architecture guardrails: `.memory-bank/spec/engineering/architecture-guardrails.md`

Зафиксируй в FT какие слои/файлы затрагиваются.

## 2. IMPLEMENTATION (по слоям)

### Layer 1: Contract
- Добавить/обновить операцию в `packages/api-contract/src/<area>.ts`.
- Typed Input/Output + parse functions.
- Добавить в `knownOperations` list.
- Error codes если нужны новые.

### Layer 2: Core
- Добавить handler в `packages/core/src/features/<area>.ts`.
- RBAC check: `hasRole(request, allowedRoles)`.
- Context: `ensureContextCompany(request)`.
- Parse input → business logic → okResult/errorResult.
- Зарегистрировать в dispatcher (`packages/core/src/index.ts`).

### Layer 3: DB
- Schema changes в `packages/db/src/schema/tables.ts` (если нужно).
- New queries в `packages/db/src/<area>.ts`.
- Migrations (если schema changed).

### Layer 4: Client
- Typed method в `packages/client/src/features/<area>.ts`.
- Wire через `createXxxClientMethods(runtime)`.
- Add to `Feedback360Client` type.

### Layer 5: CLI
- Command в `packages/cli/src/...`.
- Human output + `--json` output.
- 1:1 mapping к operation.

### Layer 6: Tests
- Unit: policies/calculations в isolation.
- Integration: full operation through dispatcher + real DB.
- Acceptance: по сценарию из FT document.

### Layer 7: Docs
- Обновить operation catalog.
- Обновить CLI command catalog.
- Обновить traceability.

## 3. VERIFICATION

```bash
pnpm checks          # quality gate
# Прогнать acceptance scenario из FT document
# Записать evidence в FT document + verification matrix
```

COMMIT: `[FT-XXXX] <description>`
```

### Пример: Реализация `campaign.start`

Одна из самых сложных операций — `campaign.start`. При старте кампании происходит:

1. **Проверка RBAC**: только HR Admin.
2. **Проверка состояния**: кампания должна быть в `draft` и иметь привязанную модель компетенций.
3. **Snapshot оргданных**: фиксация состояния employees/departments/managers на момент старта.
4. **Генерация assignments**: создание матрицы оценщиков (если авто).
5. **Создание questionnaires**: по одной анкете на каждый assignment.
6. **Переход статуса**: `draft` → `started`.
7. **Уведомления**: invite в outbox для всех участников.

В коде это цепочка:
```
CLI: campaign start <id> --json
  → Client: client.campaignStart({ campaignId })
    → Core: runCampaignStart(request)
      → RBAC check → status check → DB transaction:
        → snapshot employees
        → generate assignments
        → create questionnaires
        → update status → started
        → enqueue notifications
      → okResult({ campaignId, status: "started" })
    → OperationResult<CampaignTransitionOutput>
  → JSON output
```

### Порядок domain slices

Domain features реализуются в таком порядке (каждая группа — серия FT внутри соответствующего EP):

| # | Область | Операции | EP |
|---|---------|----------|-----|
| 1 | Identity + Tenancy | `identity.provisionAccess`, `membership.list`, `company.updateProfile` | EP-002 |
| 2 | Org structure | `employee.upsert`, `department.upsert`, `org.manager.set`, `org.department.move` | EP-003 |
| 3 | Models | `model.version.create/list/get/cloneDraft/upsertDraft/publish` | EP-004 |
| 4 | Campaigns | `campaign.create/updateDraft/start/stop/end`, `campaign.setModelVersion/weights.set` | EP-004 |
| 5 | Matrix | `matrix.generateSuggested/list/set`, `campaign.participants.add/remove/addFromDepartments` | EP-004 |
| 6 | Questionnaires | `questionnaire.listAssigned/getDraft/saveDraft/submit` | EP-004 |
| 7 | Results | `results.getMyDashboard/getTeamDashboard/getHrView` | EP-005 |
| 8 | Notifications | `notifications.generateReminders/dispatchOutbox`, settings, templates, deliveries | EP-006 |
| 9 | AI | `ai.runForCampaign` | EP-007 |
| 10 | Ops | `ops.health.get`, `ops.aiDiagnostics.list`, `ops.audit.list` | — |

### Acceptance + evidence pattern

После реализации каждой FT агент:

1. Запускает `pnpm checks` (quality gate).
2. Прогоняет acceptance scenario из FT document.
3. Записывает evidence:

```markdown
### Quality checks evidence
- Date: 2026-03-05
- Checks run: `pnpm checks`
- Result: passed

### Acceptance evidence
- Date: 2026-03-05
- Commands: `pnpm seed S5_campaign_started_no_answers --json`
  → `pnpm --filter @feedback-360/cli cli -- campaign start <handle> --json`
- Result: `{ "ok": true, "data": { "status": "started" } }`
```

### Checkpoint Фазы 7

- [ ] Каждый slice заканчивается `pnpm checks` + evidence.
- [ ] 40+ operations реализованы через все слои.
- [ ] Все golden scenarios (GS1-GS13) проходимы через CLI.
- [ ] Domain invariants покрыты тестами (anonymity, freeze, RBAC, transitions).

---

## Фаза 8: UI как Thin Layer

**Что делается**: Web app поверх уже работающего delivery path. UI не добавляет новую логику — он вызывает те же operations, что и CLI.

**Артефакты**: Next.js routes, pages, components.

### Почему UI последний

Из ADR 0001:
> "Сначала строим core use-cases + typed contract + typed client + CLI, и только потом UI."

Причины:
- UI и CLI остаются thin и не дублируют правила.
- Логику можно тестировать быстрее и дешевле (без браузера).
- CLI удобен для AI-агента и для отладки сценариев/сидов.
- К моменту появления UI все операции уже протестированы и стабильны.

### Промпт #16: UI foundation

```
Реализуй UI foundation для feedback-360.

GROUNDING:
- [ ] `.memory-bank/spec/engineering/frontend-ui-stack.md`
- [ ] `.memory-bank/spec/ui/index.md`
- [ ] `.memory-bank/spec/engineering/architecture-guardrails.md`
- [ ] `.memory-bank/adr/0001-core-client-cli-first.md`
- [ ] Уже работающий `packages/client/` с in-proc transport.

DELIVERABLES:

1. Auth flow:
   - Magic link login page.
   - Session management (Supabase Auth).
   - Middleware для protected routes.

2. App shell:
   - Layout с navigation.
   - Company switcher (для multi-tenant).
   - Role-based navigation (HR видит admin pages, employee — свои).

3. Базовые компоненты:
   - Tailwind CSS v4 + shadcn/ui setup.
   - Common layout components (PageHeader, DataTable, etc.).

4. API route handler pattern:
   ```typescript
   // apps/web/src/app/api/v1/operations/route.ts
   export async function POST(req: Request) {
     const body = await req.json();
     const client = createInprocClient();
     // resolve session → set context
     const result = await client.invokeOperation({
       operation: body.operation,
       input: body.input,
       context: sessionContext,
       parseOutput: (v) => v, // pass-through
     });
     return Response.json(result);
   }
   ```

KEY PATTERNS:
- Route handler = thin adapter (parse HTTP → call client → return response).
- UI НЕ импортирует core напрямую — только через client.
- Server components для data loading.
- `data-testid` на ключевых элементах (для automation).

ACCEPTANCE:
- Login flow работает (magic link).
- Company switcher переключает контекст.
- Navigation отображается по роли.
- `pnpm --filter @feedback-360/web build` проходит.

COMMIT: `[EP-ui] UI foundation`
```

### Промпт #17: Feature-specific UI page

```
Реализуй UI страницу для [feature area] (напр. HR Campaign list).

GROUNDING:
- [ ] `.memory-bank/spec/ui/screens/[screen-spec].md` (если есть)
- [ ] `.memory-bank/spec/ui/screen-registry.md`
- [ ] Уже работающие операции через client.

DELIVERABLES:

1. Route handler как thin adapter:
   ```typescript
   // apps/web/src/app/api/hr/campaigns/route.ts
   export async function GET() {
     const client = createInprocClient();
     const context = await resolveSessionContext();
     const result = await client.campaignList(context);
     return Response.json(result);
   }
   ```

2. Server component для data loading:
   ```typescript
   // apps/web/src/app/hr/campaigns/page.tsx
   export default async function CampaignsPage() {
     const data = await fetchCampaigns(); // server-side
     return <CampaignListView campaigns={data} />;
   }
   ```

3. Client component для interactions:
   ```typescript
   // apps/web/src/features/campaigns/components/CampaignListView.tsx
   export function CampaignListView({ campaigns }) {
     return (
       <div data-testid="campaign-list">
         {campaigns.map(c => (
           <CampaignCard key={c.id} campaign={c}
             data-testid={`campaign-card-${c.id}`} />
         ))}
       </div>
     );
   }
   ```

KEY PATTERNS:
- Route handler: thin, no business logic, only HTTP adapter.
- Server components: fetch data on server side.
- Client components: interactive UI, forms, state.
- Screen identifiers: `data-testid` для automation.
- UI вызывает те же operations, что и CLI.

ACCEPTANCE:
- Страница отображает данные из API.
- Операции на странице работают (create, edit, delete).
- `data-testid` присутствуют на ключевых элементах.
- UI не содержит доменных правил.
```

### Checkpoint Фазы 8

- [ ] UI вызывает те же operations, что CLI.
- [ ] Route handlers — thin adapters (без бизнес-логики).
- [ ] `data-testid` на ключевых элементах.
- [ ] `pnpm --filter @feedback-360/web build` проходит.
- [ ] Login → navigation → feature pages работают.

---

## Фаза 9: Hardening, Refactor, Traceability

**Что делается**: Стабилизация, feature-area refactor, UI traceability, documentation hardening.

**Артефакты**: ADR 0004, feature-area structure, screen registry, XE scenarios.

### Когда делать refactor

Feature-area refactor делается **не сразу**, а когда появляется pressure:
- Root files перестают быть локальными (один `legacy.ts` содержит 50+ операций).
- Изменение одной feature затрагивает файлы, принадлежащие другой.
- Навигация по коду становится сложной.

В feedback-360 refactor произошёл после первой волны delivery, когда все основные domain features были реализованы и стабилизированы.

### Промпт #18: Feature-area refactor

```
Проведи feature-area refactor кодовой базы.

ВАЖНО: Сначала docs/ADR, потом code move.

## Шаг 1: Документация и rationale

1. Создай ADR `adr/0004-feature-area-slicing-boundaries.md`:
   - Decision: разделить код на feature areas вместо flat root files.
   - Why: root files перестали быть локальными, ownership размазан.
   - Trade-offs: больше файлов, но лучше locality и ownership.

2. Создай `spec/project/feature-area-boundaries.md`:
   - Canonical feature areas: campaigns, results, questionnaires,
     identity-tenancy, org, models, matrix, notifications, ai, ops, system.
   - Shared: только для truly shared helpers (context, errors).
   - Root composition points: `index.ts` только для dispatch/export.

## Шаг 2: Code move

Для каждого package (`api-contract`, `core`, `client`, `cli`):
1. Создать `src/features/<area>.ts` для каждой feature area.
2. Перенести operations/handlers/methods из root files.
3. Root `index.ts` → thin composition/dispatch only.
4. Update imports.

Target structure:
```
packages/core/src/
├── index.ts                    # dispatcher only
├── features/
│   ├── identity-tenancy.ts
│   ├── org.ts
│   ├── models.ts
│   ├── campaigns.ts
│   ├── matrix.ts
│   ├── questionnaires.ts
│   ├── results.ts
│   ├── notifications.ts
│   ├── ai.ts
│   └── ops.ts
└── shared/
    └── context.ts              # truly shared helpers
```

## Шаг 3: Verification

- `pnpm checks` проходит (behavior не изменился).
- Root entrypoints стали thin.
- Feature areas читаются локально.
- Public API не изменился.

COMMIT: `[EP-refactor] Feature-area slicing`
```

### XE scenarios — сквозные сценарии

После стабилизации architecture появляются XE (Cross-Epic) scenarios — сквозные сценарии, проходящие через несколько эпиков:

```
XE-001: Полный цикл оценки
  Phase 1: Setup org (EP-002, EP-003)
  Phase 2: Create model + campaign (EP-004)
  Phase 3: Fill questionnaires (EP-004)
  Phase 4: End campaign + AI (EP-005, EP-007)
  Phase 5: View results (EP-005)
  Phase 6: Verify anonymity (EP-005)
```

### Screen registry и test ID registry

Для UI traceability каждый экран получает:
- `screen_id` — канонический идентификатор (напр. `SCR-CAMPAIGNS-LIST`).
- `testIdScope` — префикс для `data-testid` (напр. `campaigns-list`).

Это позволяет:
- Automation scripts ссылаться на стабильные идентификаторы.
- Docs/screenshots привязываться к конкретным экранам.
- Impact analysis при изменении экрана начинаться от registry.

### Evidence-based completion

Ни одна фича не считается завершённой без:
1. `pnpm checks` — quality gate.
2. Acceptance scenario — прогон сценария из FT.
3. Evidence — записанные команды, результаты, дата.
4. Docs audit — `pnpm docs:audit` для проверки memory bank.

### Checkpoint Фазы 9

- [ ] ADR 0004 документирует rationale refactor-а.
- [ ] Feature-area boundaries задокументированы в specs.
- [ ] `pnpm checks` проходит после refactor.
- [ ] Screen registry содержит все экраны.
- [ ] XE scenario проходит end-to-end.
- [ ] Evidence записано для всех FT.

---

## Итог: Полный цикл

### Таблица: фаза → промпт → артефакт → checkpoint

| Фаза | Промпты | Ключевые артефакты | Checkpoint |
|-------|---------|-------------------|------------|
| 0. Идея → Brief | #1 | Реконструированный brief | Brief покрывает domain + technical |
| 1. MBB | #2, #3 | `mbb/principles.md`, templates | MBB navigable, templates готовы |
| 2. Brief → Specs | #4, #5, #6 | 90+ spec files | Полная спецификация до кода |
| 3. Specs → Epics | #7, #8 | Roadmap, epics, features, playbook | Каждая FT имеет сценарий |
| 4. AGENTS.md | #9 | `AGENTS.md`, `index.md` | Агент понимает проект |
| 5. Scaffold + DB | #10, #11 | packages/, schema, migrations | `pnpm install` + `db:health` ok |
| 6. Core Loop | #12, #13, #14 | contract, core, client, CLI | `system.ping --json` works |
| 7. Domain Slices | #15 (×N) | 40+ operations | Все GS проходят через CLI |
| 8. Thin UI | #16, #17 | Next.js routes, pages | UI calls same operations as CLI |
| 9. Hardening | #18 | ADR 0004, feature areas, XE | `pnpm checks` + XE e2e passes |

### Антипаттерны: 8 ошибок при воспроизведении

1. **Начать с UI** — ломает deterministic delivery path. Браузер становится местом, где впервые появляется бизнес-логика.

2. **Смешать spec/plans/adr** — система теряет разделение WHAT / WHY / delivery. Непонятно, где нормативные требования, а где план работ.

3. **Хранить business rules в route handlers или CLI** — UI/CLI перестают быть thin. Каждое расширение начинает дублировать домен.

4. **Сделать `shared` свалкой** — feature-area refactor формально есть, но ownership всё равно размазан. `shared` должен содержать только truly shared helpers.

5. **Писать acceptance без seed/handles** — brittle сценарии, скрытые зависимости от случайного состояния. Handles дают детерминизм.

6. **Воспроизводить только финальную структуру** — копируется форма, но не понимание. Важна эволюция и причины каждого решения.

7. **Закрывать feature без evidence** — completion без checks, acceptance и traceability считается недоказанным. Evidence — часть DoD.

8. **Игнорировать stable identifiers в UI** — browser automation и runtime traceability быстро становятся дорогими. `data-testid` и screen IDs с самого начала.

### Reading order для изучения оригинального проекта

Если хотите понять подход быстро, идите по этому маршруту:

1. `.memory-bank/index.md` — входная точка, curated quick-start.
2. `.memory-bank/spec/project/system-overview.md` — общая картина за 2 минуты.
3. `.memory-bank/spec/project/layers-and-vertical-slices.md` — слои и vertical slice DoD.
4. `.memory-bank/spec/engineering/architecture-guardrails.md` — что запрещено.
5. `.memory-bank/plans/implementation-playbook.md` — как реализуется фича.
6. `.memory-bank/adr/0001-core-client-cli-first.md` — почему CLI-first.
7. `.memory-bank/mbb/principles.md` — правила ведения документации.
8. `packages/api-contract/src/v1/legacy.ts` — types, operations, errors.
9. `packages/core/src/index.ts` — dispatcher.
10. `packages/core/src/features/campaigns.ts` — пример domain handler.
11. `packages/client/src/shared/runtime.ts` — transport abstraction.
12. `packages/client/src/index.ts` — client factory.
13. `packages/db/src/schema/tables.ts` — полная schema.
14. `git log --reverse --oneline` — эволюция проекта.
15. `docs/feedback-360-faithful-rebuild-guide.md` — reverse-engineering анализ.
16. `docs/feedback-360-step-by-step-rebuild.md` — технический rebuild guide.

### Формула подхода

> **Сначала структурируй знание** (MBB + specs),
> **затем нарежь на delivery units** (epics + features),
> **затем построй deterministic non-UI delivery path** (contract + core + client + CLI),
> **затем выращивай домен vertical slices** (каждый с evidence),
> **затем добавь UI как thin surface**,
> **затем сделай structure и traceability достаточно сильными**,
> чтобы система оставалась понятной агентам и людям по мере роста.