## ЧАСТЬ I — Контекст и навигация

> 🧭 [[feedback-360-master-rebuild-walkthrough|Master Rebuild Walkthrough — оглавление]]


### 1.1. О каком проекте речь и почему он учебный

`feedback-360` — SaaS-приложение для проведения корпоративной оценки сотрудников методом 360 градусов. Технически это TypeScript-монорепо на pnpm с Next.js-фронтендом, PostgreSQL и CLI-инструментом. Доменно — это multi-tenant HR-система с кампаниями оценки, моделями компетенций, анонимностью и AI-постобработкой open text.

Но интересен этот репозиторий **не как ещё один Next.js+Supabase монорепо**. Интересен он как **демонстрация связного engineering workflow с агентами**, в котором:

1. **Docs появляются раньше кода** и задают форму проекта.
2. **Memory Bank** используется не как «свалка заметок», а как structured agent memory.
3. **Delivery идёт через vertical slices** — сквозные куски от contract до CLI/UI, — а не «сначала все слои, потом когда-нибудь сценарий».
4. Для агентов и automation основной delivery path строится через **`core + contract + client + CLI`**, а UI приходит позже.
5. **Traceability** между intent, docs, code, tests и evidence делается сознательно, а не «как получится».
6. **Completion считается доказанным** только после checks, acceptance scenario и записи evidence, а не после «код вроде написан».
7. **Структурная эволюция** (feature-area refactor) — это отдельная осознанная волна с собственным ADR, а не «под шумок во время feature work».
8. **Agentic workflow** — это не «попросить ChatGPT сгенерировать код», а связанный цикл brief → spec → epic/feature → grounding → implementation → verification → evidence.

Сочетание этих восьми пунктов делает проект **редким полным примером**: видеть только финальный код в репо недостаточно, чтобы понять, как такая система выросла. Этот walkthrough даёт реконструкцию роста.

### 1.2. Карта источников

Этот файл синтезирует материал из шести источников:

| Источник | Где лежит | Что даёт |
|---|---|---|
| `feedback-360-faithful-rebuild-guide.md` | `docs/` | Концептуальный reverse-engineering: что построено и почему именно так |
| `feedback-360-step-by-step-rebuild.md` | `docs/` | Технический манул с code-snippets и aha-моментами |
| `feedback-360-learning-path.md` | `docs/` | Curriculum-слой: правила обучения, 9 модулей |
| `feedback-360-agentic-rebuild-process.md` | `docs/` | 18 промптов для LLM по фазам |
| YouTube-транскрипт часть 1 | `_articles/Agentic Engineering AI Workflow с DEKSDEN часть 1/` | Brief, vertical slices, тестирование, agentic test, верификация, code review |
| YouTube-транскрипт часть 2 | `_articles/Agentic Engineering AI Workflow с DEKSDEN часть 2/` | Memory Bank как knowledge base, аннотированные ссылки, AGENTS.md как роутинг, «пирамидка знаний» |

Плюс **первичный материал — сам Memory Bank** в `.memory-bank/`: ADR, специации, плановые эпики/фичи, шаблоны MBB. Когда в тексте упоминается конкретный документ MBB, дана относительная ссылка от корня репозитория.

### 1.3. Ключевая формула подхода

Если свести весь подход к одной фразе, она звучит так:

> **Сначала структурируй знание → построй deterministic non-UI delivery path → выращивай домен vertical slices → добавь UI как thin surface → сделай traceability сильным настолько, чтобы система оставалась понятной агентам и людям по мере роста.**

Эту фразу стоит запомнить: каждый шаг walkthrough в конечном счёте к ней возвращается.

### 1.4. Глоссарий критичных терминов

Эти термины будут встречаться постоянно. Если хотя бы один из них непонятен — вернитесь к этому разделу.

- **SSoT (Single Source of Truth)** — каждая концепция или правило имеет ровно один нормативный документ. Дубли запрещены: если нашли — объединить.
- **MBB (Memory Bank Bible)** — operating manual меморибанка, лежащий в `.memory-bank/mbb/`. Описывает правила ведения самой документации.
- **EP (Epic)** — группа пользовательской ценности. Например, `EP-001-core-contract-client-cli` или `EP-004-campaigns-questionnaires`.
- **FT (Feature)** — минимальный vertical slice с проверяемой ценностью. Например, `FT-0011-op-plumbing-errors`.
- **Vertical slice** — сквозной кусок от contract до CLI/UI, реализующий одну единицу ценности. Не «кусок слоя».
- **Grounding** — обязательный шаг перед кодированием: агент читает FT-документ и связанные specs, чтобы знать контекст.
- **Evidence-first completion** — фича не считается готовой, пока не записаны: пройденный `pnpm checks`, выполненный acceptance scenario, evidence-блок (дата + команды + результат).
- **`OperationResult<T>`** — discriminated union `{ ok: true; data: T } | { ok: false; error: OperationError }`, центральный тип системы.
- **Dispatcher** — функция `dispatchOperation()` в `packages/core/src/index.ts`, маршрутизирующая 50+ операций по имени.
- **In-proc transport** — реализация `OperationTransport`, которая вызывает core напрямую (используется в тестах и CLI).
- **HTTP transport** — реализация `OperationTransport`, которая делает POST к серверу (используется в web-app).
- **Annotated link** — markdown-ссылка с двумя предложениями: «что лежит» + «зачем читать». Обязательное правило в Memory Bank автора.
- **Screen ID** — канонический идентификатор экрана в `spec/ui/screen-registry.md`, используется в frontmatter, JSDoc, `data-testid`, скриншотах.
- **XE (Cross-Epic) scenario** — сквозной сценарий, тестирующий полный user journey через несколько feature areas.
- **Feature area** — каноническая зона владения кодом (10 областей: identity-tenancy, org, models, campaigns, matrix, questionnaires, results, notifications, ai, ops).
- **REBUILD_ROOT** — ваш учебный repo, в котором вы строите свою версию подхода.
- **REFERENCE_ROOT** — оригинальный `feedback-360`, который вы читаете как живой образец, но не мутируете.

### 1.5. С чего начать прямо сейчас (если у вас есть 30 минут)

1. Прочитайте этот раздел до конца (5 минут).
2. Прочитайте Часть II — Философия (15 минут).
3. Откройте `.memory-bank/index.md` в исходном репо и пробегите его глазами (3 минуты).
4. Откройте `.memory-bank/adr/0001-core-client-cli-first.md` — прочитайте полностью, это коротко (2 минуты).
5. Запишите в свой `LEARNING_NOTES.md`: «Что я понял после первых 30 минут» (5 минут).

После этих 30 минут вы уже способны объяснить кому-то ключевую формулу подхода и зачем нужен Memory Bank. Дальше можно идти по Режиму A или B.