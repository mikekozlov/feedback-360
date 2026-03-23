# feedback-360: faithful rebuild guide

## Что это за документ

Этот документ делает две вещи одновременно:

1. Reverse-engineers подход, реально использованный в `feedback-360`.
2. Даёт faithful plan, как воспроизвести этот подход в новом пустом репозитории, чтобы изучить его по шагам.

Источники, на которых основан разбор:

- Memory Bank как SSoT процесса, архитектуры и delivery discipline:
  - [`../.memory-bank/index.md`](../.memory-bank/index.md)
  - [`../.memory-bank/plans/implementation-playbook.md`](../.memory-bank/plans/implementation-playbook.md)
  - [`../.memory-bank/spec/engineering/architecture-guardrails.md`](../.memory-bank/spec/engineering/architecture-guardrails.md)
  - [`../.memory-bank/spec/project/layers-and-vertical-slices.md`](../.memory-bank/spec/project/layers-and-vertical-slices.md)
  - [`../.memory-bank/spec/testing/traceability.md`](../.memory-bank/spec/testing/traceability.md)
  - [`../.memory-bank/mbb/templates/feature.md`](../.memory-bank/mbb/templates/feature.md)
- Реальная структура кода как подтверждение того, что подход дошёл до working system:
  - [`../packages/api-contract/src/campaigns.ts`](../packages/api-contract/src/campaigns.ts)
  - [`../packages/client/src/features/campaigns.ts`](../packages/client/src/features/campaigns.ts)
  - [`../packages/core/src/features/campaigns.ts`](../packages/core/src/features/campaigns.ts)
  - [`../packages/db/src/campaigns.ts`](../packages/db/src/campaigns.ts)
  - [`../apps/web/src/app/api/hr/campaigns/draft/route.ts`](../apps/web/src/app/api/hr/campaigns/draft/route.ts)
- Git history как карта эволюции:
  - `git log --reverse --oneline`
- Транскрипты автора как explanation of intent and trade-offs:
  - [`../_articles/Agentic Engineering AI Workflow с DEKSDEN часть 1/Agentic Engineering AI Workflow с DEKSDEN часть 1 YOUTUBE SCRIPT.md`](../_articles/Agentic%20Engineering%20AI%20Workflow%20с%20DEKSDEN%20часть%201/Agentic%20Engineering%20AI%20Workflow%20с%20DEKSDEN%20часть%201%20YOUTUBE%20SCRIPT.md)
  - [`../_articles/Agentic Engineering AI Workflow с DEKSDEN часть 2/Agentic Engineering AI Workflow с DEKSDEN часть 2 YOUTUBE SCRIPT.md`](../_articles/Agentic%20Engineering%20AI%20Workflow%20с%20DEKSDEN%20часть%202/Agentic%20Engineering%20AI%20Workflow%20с%20DEKSDEN%20часть%202%20YOUTUBE%20SCRIPT.md)

Важно: этот guide не меняет runtime behavior проекта и не переопределяет `.memory-bank/` как SSoT. Он добавляет внешний reverse-engineering handbook в верхнеуровневый `docs/`.

## Why this repo matters

Этот репозиторий интересен не просто как “ещё один Next.js + Supabase monorepo”, а как демонстрация связанного engineering workflow, где:

- docs появляются раньше основного кода и задают форму проекта;
- Memory Bank используется не как свалка заметок, а как structured agent memory;
- delivery идёт через vertical slices, а не через “сначала все слои, потом когда-нибудь сценарий”;
- для агентов и automation основной delivery path строится через `core + contract + client + CLI`, а UI приходит позже;
- traceability между intent, docs, code, tests и evidence делается сознательно;
- completion считается доказанным только после checks, acceptance и evidence, а не после “код вроде написан”.

Именно сочетание этих пунктов делает проект хорошим объектом для обучения: здесь можно изучать не только архитектуру, но и способ роста системы.

## Approach in one page

Ниже сжатая модель подхода автора.

1. Сначала фиксируется структура знания о проекте, а потом код.
2. `Memory Bank` делится минимум на `spec` (WHAT), `plans` (delivery units), `adr` (WHY) и правила ведения самого банка.
3. Brief сначала раскладывается на `epics` и `features`, а уже потом каждая feature grounded в текущее состояние системы.
4. Минимальная единица delivery здесь не “слой” и не “компонент”, а vertical slice с user value, deliverables, acceptance и docs updates.
5. Доменная логика должна жить в `core`, а не в UI, route handlers или CLI.
6. Между delivery-слоями должен быть typed contract, чтобы `client`, `CLI` и UI говорили на одном языке DTO и ошибок.
7. `CLI-first for agents` нужен не ради эстетики, а ради deterministic verification без постоянной борьбы с browser automation.
8. UI рассматривается как thin delivery layer поверх уже работающего `client`.
9. Когда кодовая база доросла, она была дополнительно перестроена из layer-flat layout в feature-area layout, чтобы локализовать ownership и зону изменений.
10. Документация, тесты и evidence обновляются как часть slice, а не постфактум “если останется время”.

Опорные принципы, которые надо сохранить при faithful rebuild:

- `Memory Bank as structured agent memory`
- `core + contract + client + CLI-first`
- `thin UI`
- `vertical slices`
- `traceability`
- `deterministic verification`

## Analysis from different angles

### 1. Documentation / planning angle

**Принцип**

Documentation в этом проекте не обслуживает код задним числом. Она задаёт рамку, в которой код вообще может появляться.

**Как это проявляется в текущем repo**

- Главный вход в знание проекта начинается с [`../.memory-bank/index.md`](../.memory-bank/index.md), где есть curated quick-start и SSoT map.
- [`../.memory-bank/plans/implementation-playbook.md`](../.memory-bank/plans/implementation-playbook.md) превращает идею vertical slice в рабочий чеклист `contract -> core -> db -> adapters -> client -> cli -> tests -> docs`.
- [`../.memory-bank/mbb/templates/feature.md`](../.memory-bank/mbb/templates/feature.md) заставляет каждую FT иметь user value, deliverables, grounding, implementation plan, scenarios, tests, docs updates, evidence.
- [`../.memory-bank/spec/testing/traceability.md`](../.memory-bank/spec/testing/traceability.md) связывает domain invariants с tests и seed scenarios.
- Из истории видно, что repo был seeded сначала Memory Bank-документами, а потом scaffold/кодом: `317c858`, `37139c0`, `ee0f0a7`, `b1962de`, `8a67af4`, `27bfaf3`, `3c09dfe`.

**Как это повторить в новом repo**

- Начать не с `src/`, а с документационного skeleton.
- Сразу разделить знание на `spec`, `plans`, `adr`, `guides`, `mbb`.
- Принять, что feature definition of done включает не только код, но и grounding, acceptance, tests и evidence.
- Не разрешать себе “скрытые требования в голове”: если правило нужно агенту или новому инженеру, оно должно иметь каноническое место в Memory Bank.

### 2. Architecture angle

**Принцип**

Архитектура строится вокруг thin delivery layers и жирного, проверяемого middle: `contract + client + core + db`.

**Как это проявляется в текущем repo**

- [`../.memory-bank/adr/0001-core-client-cli-first.md`](../.memory-bank/adr/0001-core-client-cli-first.md) фиксирует исходное решение: сначала core, typed contract, typed client и CLI, потом UI.
- [`../.memory-bank/spec/engineering/architecture-guardrails.md`](../.memory-bank/spec/engineering/architecture-guardrails.md) запрещает `apps/web` и `packages/cli` ходить в доменный core напрямую.
- [`../.memory-bank/spec/project/layers-and-vertical-slices.md`](../.memory-bank/spec/project/layers-and-vertical-slices.md) формализует базовые слои и minimum DoD для slice.
- [`../.memory-bank/adr/0004-feature-area-slicing-boundaries.md`](../.memory-bank/adr/0004-feature-area-slicing-boundaries.md) показывает, что после первой волны delivery проект не застыл: его сознательно refactor-нули в feature areas, потому что growing root files стали мешать локальности изменений.

**Как это повторить в новом repo**

- С самого начала развести `core`, `api-contract`, `client`, `cli`, `db`, `web`.
- Не тащить business rules в HTTP handlers, React components или CLI commands.
- На ранней стадии можно жить в layer-oriented структуре, но надо заранее знать, что при росте системы, вероятно, понадобится feature-area refactor.
- `shared` вводить только для реально общих модулей без собственного product ownership.

### 3. Code angle

**Принцип**

Один сценарий должен читаться через цепочку слоёв, а не растворяться в наборе случайных файлов.

**Как это проявляется в текущем repo**

На примере кампаний цепочка выглядит так:

1. [`../packages/api-contract/src/campaigns.ts`](../packages/api-contract/src/campaigns.ts)
   - даёт feature-level export surface для типов и parser-ов;
   - наружу выглядит стабильно, даже если внутри часть runtime/schema логики пока живёт в transitional `v1/legacy`.
2. [`../packages/client/src/features/campaigns.ts`](../packages/client/src/features/campaigns.ts)
   - валидирует input через contract parsers;
   - вызывает `runtime.invokeOperation(...)`;
   - не содержит доменной логики, только typed access к операциям.
3. [`../packages/core/src/features/campaigns.ts`](../packages/core/src/features/campaigns.ts)
   - применяет RBAC/context guardrails;
   - парсит input;
   - orchestrates use-case;
   - мапит ошибки в typed `OperationResult`;
   - записывает audit там, где это часть application behavior.
4. [`../packages/db/src/campaigns.ts`](../packages/db/src/campaigns.ts)
   - реализует transactional persistence и status transitions;
   - на уровне базы и транзакции обеспечивает часть критичных инвариантов;
   - умеет вызывать связанные операции, например snapshots/questionnaires/notifications при старте кампании.
5. [`../apps/web/src/app/api/hr/campaigns/draft/route.ts`](../apps/web/src/app/api/hr/campaigns/draft/route.ts)
   - это delivery adapter;
   - он парсит HTTP/form payload;
   - резолвит app/session context;
   - вызывает in-proc client;
   - мапит typed errors в HTTP status и redirects.

Именно это сочетание важно: route handler не “знает бизнес”, а `client` и `core` не знают ничего о `NextResponse`, form redirects или browser flow.

**Как это повторить в новом repo**

- Сначала договориться, как выглядит operation surface для хотя бы одной feature area.
- Сделать один рабочий vertical slice через все слои, прежде чем плодить новые домены.
- Проверять, что chain остаётся читаемой:
  - contract определяет язык;
  - client даёт единый API;
  - core решает бизнес;
  - db хранит и транзакционно поддерживает состояние;
  - web/cli только доставляют вызов.

### 4. Testing / QA angle

**Принцип**

Главный путь проверки должен быть максимально детерминированным, быстрым и дешёвым. Browser automation нужна, но не как основной способ мыслить о системе.

**Как это проявляется в текущем repo**

- В transcripts автор несколько раз проговаривает, что агентам тяжело и дорого жить в браузере как в основном execution medium.
- ADR и playbook толкают проект к `CLI-first` и `client/in-proc` verification.
- В корне repo есть единый `checks` pipeline в [`../package.json`](../package.json), а quality gates вынесены в обязательную часть процесса.
- В `packages/core`, `packages/client`, `packages/cli`, `packages/db` много FT-level tests, которые подтверждают поведение без обязательного запуска UI.
- Browser automation есть, но она идёт позже и опирается на стабильные selectors и screen contracts в `apps/web/playwright` и UI specs.

**Как это повторить в новом repo**

- Для каждого slice сначала обеспечивать unit/integration/contract verification через `core/client/cli`.
- E2E в браузере добавлять только на критичные user journeys и на регрессионные key paths.
- Acceptance не писать без seed/handles, потому что иначе сценарий будет зависеть от случайных ID и нестабильного состояния.
- Как только появляется UI, сразу закладывать stable identifiers и screen-level contracts.

### 5. Git evolution angle

**Принцип**

Подход не был “сразу весь” с первого коммита. Он вырос по фазам, и это важно повторить при обучении.

**Как это проявляется в текущем repo**

- Сначала появились docs, structure и планы.
- Потом foundation и DB.
- Потом `core + contract + client + cli`.
- Потом domain-heavy backend slices.
- Потом первая волна UI.
- Потом hardening/release discipline.
- Потом feature-area refactor.
- Потом GUI wave, XE scenarios, guides, UI traceability и docs hardening.

**Как это повторить в новом repo**

- Не пытаться воспроизвести репозиторий в одном гигантском стартовом коммите.
- Повторять именно эволюцию, а не только финальное дерево файлов.
- Каждый крупный architectural move делать тогда, когда для него созрел pressure:
  - `CLI-first` появляется, когда нужен deterministic execution;
  - feature-area slicing появляется, когда root files перестают быть локальными;
  - UI traceability появляется, когда GUI уже вырос до самостоятельной системы.

## What the author explicitly says in the sessions

Ниже distilled summary того, что автор явно проговаривает в двух сессиях и что подтверждается репозиторием.

1. Brief сначала режется на `epics` и `features`.
   - Автор прямо объясняет, что просит модель разложить большой brief на value-oriented delivery units, и каждая feature должна получить value, deliverables и verification scenario.
2. Feature должна быть минимальной единицей полезного результата.
   - Не “кусок слоя”, а минимальная работающая ценность для пользователя.
3. `CLI` нужен как deterministic client for agents.
   - Не потому, что UI не нужен, а потому что browser-first execution для агентов медленный и хрупкий.
4. Общая клиентская логика не должна дублироваться между UI и CLI.
   - Поэтому строится единый `client`, а `CLI` и UI становятся thin shells.
5. `Memory Bank` нужен не как набор заметок, а как structured knowledge base, task tracker и traceability layer.
   - Он фиксирует текущее состояние системы, планы работ, rationale и evidence.
6. Документация должна избегать дублей.
   - Автор отдельно акцентирует `single source of truth`, индексы, annotated links, progressive disclosure и C4-уровни документации.
7. UI должен иметь stable identifiers и screen-level structure.
   - Это нужно не только для manual QA, но и для того, чтобы агент мог строить deterministic tests и screen/object abstractions поверх интерфейса.
8. Vertical slices улучшают качество не только delivery, но и тестирования.
   - Чем локальнее change surface, тем проще агенту и инженеру продумать затронутые кейсы.

Практический вывод: если вы хотите faithfully повторить этот workflow, вам нужно копировать не только каталог папок, но и эти рабочие убеждения.

## How the repo actually evolved

Ниже обучающая разбивка истории на фазы.

### Phase 1. Docs-first foundation and scaffold

Характерные коммиты: `317c858..508ba7e`

- `317c858` — initial commit.
- `37139c0`, `ee0f0a7`, `b1962de`, `8a67af4`, `27bfaf3`, `3c09dfe` — Memory Bank, specs, plans, templates, structure.
- `e81a9f8` — workspace scaffold.
- `508ba7e` — baseline DB migrations and health checks.

Чему это учит: сначала формируется map of the system и delivery discipline, потом появляется foundation-код.

### Phase 2. Operation dispatcher, client transport, CLI-first

Характерные коммиты: `d91182e..0d34647`

- `d91182e` — operation dispatcher and typed errors.
- `da26521` — HTTP/in-proc transport and active company context.
- `0d34647` — questionnaire ops and CLI flow.

Чему это учит: до серьёзного UI система уже должна иметь единый operation language и executable client surface.

### Phase 3. Domain slices and policy-heavy backend

Характерные коммиты: `8edb962..328bdb0`

- identity/tenancy, RBAC, RLS;
- org history, snapshots, matrix autogeneration;
- campaign lifecycle, freeze, read-only, progress;
- results, anonymity, normalization;
- notifications, retries, scheduling, invites.

Чему это учит: business-heavy rules в этом подходе строятся как серия FT slices поверх уже существующей delivery loop, а не как “один большой domain layer потом”.

### Phase 4. UI приходит после backend/CLI

Характерные коммиты: `b9037e9..7868ea0`

- UI foundation, Tailwind/shadcn, auth/company switcher;
- затем questionnaire UI, results UI, HR campaign workbench.

Чему это учит: UI не запускает проект, а приходит на уже работающий слой операций и сценариев.

### Phase 5. Hardening, release discipline, prod-readiness

Характерные коммиты: `cf6106f..462a487`

- test/release hardening;
- browser-smoke gates;
- privacy hardening;
- observability baseline;
- prod-readiness closeout with evidence.

Чему это учит: после первой working version начинается не новая feature race, а stabilisation wave.

### Phase 6. Structural refactor into feature areas

Характерные коммиты: `7ffa297..674f596`

- сначала docs/rationale;
- потом runtime refactor into feature-area slices;
- затем CI/beta evidence.

Чему это учит: structural refactor делается как отдельная осознанная волна, а не “под шумок” во время feature work.

### Phase 7. GUI wave поверх стабилизированной архитектуры

Характерные коммиты: `17e0a62..8c6dd43`

- app shell and dashboards;
- HR campaigns UX;
- questionnaire UX;
- results dashboards;
- people/org admin;
- models/matrix UI;
- notification center;
- ops console.

Чему это учит: после архитектурной стабилизации UI можно наращивать сериями продуктовых волн.

### Phase 8. XE scenarios and user-facing operational guides

Характерные коммиты: `c85a165..45496ff`

- XE runtime foundation;
- token helpers;
- scenario scripts;
- manual walkthroughs;
- Diataxis guides scaffold;
- first-campaign tutorial.

Чему это учит: mature system получает не только tests, но и executable scenarios и operator-facing guides.

### Phase 9. UI traceability, design system, docs hardening

Характерные коммиты: `9f6f5a3..a8bc502`

- UI traceability;
- screen IDs and predictable test IDs;
- SaaS polish;
- unified visual system rollout;
- documentation traceability hardening;
- runtime UI traceability follow-up.

Чему это учит: когда UI вырос, его тоже пришлось подчинить той же дисциплине traceability, что и backend/docs/testing.

## Faithful rebuild plan for a new empty repo

Ниже не абстрактный blueprint, а именно предлагаемый порядок восстановления подхода в новом repo.

### Phase 1. Initialize empty pnpm workspace and baseline scaffold

- **Goal:** создать минимальный executable shell репозитория.
- **Minimal artifacts:** `package.json`, `pnpm-workspace.yaml`, корневые scripts для `lint`, `typecheck`, `test`, `.gitignore`, `README.md`.
- **Commit:** один foundation commit без бизнес-логики.
- **Verify before moving on:** workspace устанавливается, корневые команды запускаются хотя бы в no-op/placeholder режиме.

### Phase 2. Add docs skeleton immediately

- **Goal:** зафиксировать knowledge structure до начала feature work.
- **Minimal artifacts:** `.memory-bank/mbb/*`, `.memory-bank/index.md`, `spec/index.md`, `plans/index.md`, `adr/index.md`, templates для epic/feature/component/subsystem.
- **Commit:** docs-only commit или короткая docs-only серия коммитов.
- **Verify before moving on:** из root index можно дойти до project/spec/plans/adr; для новых документов уже есть шаблоны и правила SSoT.

### Phase 3. Freeze structure, layers and engineering guardrails

- **Goal:** сделать architecture rules explicit.
- **Minimal artifacts:** repo structure spec, layers & vertical slices, architecture guardrails, implementation playbook, testing standards, traceability rules.
- **Commit:** отдельный “rules of the game” commit.
- **Verify before moving on:** понятно, где будет жить `core`, `api-contract`, `client`, `cli`, `db`, `web`, и что запрещено delivery layers.

### Phase 4. Build the foundation slice

- **Goal:** получить reproducible repo, который умеет жить технически, даже если домен ещё почти пустой.
- **Minimal artifacts:** monorepo packages/apps scaffold, DB baseline, migrations health, seed runner contract, web scaffold, error/reporting baseline.
- **Commit:** несколько FT-level commits внутри первого foundation epic.
- **Verify before moving on:** можно поднять базу, прогнать миграции и выполнить хотя бы один seed scenario через automation path.

### Phase 5. Implement `core + api-contract + client + cli` as the first delivery loop

- **Goal:** создать первый рабочий сценарий без UI.
- **Minimal artifacts:** typed operations, input/output parsers, typed error model, runtime invoke mechanism, HTTP/in-proc parity, CLI with human and `--json` output.
- **Commit:** отдельная волна, похожая на EP-001.
- **Verify before moving on:** хотя бы одна real feature проходит end-to-end через `client/cli` без браузера.

### Phase 6. Build business features only as vertical slices

- **Goal:** выращивать домен кусками, которые заканчиваются working behavior.
- **Minimal artifacts:** FT docs, use-cases, schema/migration changes when needed, tests, docs updates, evidence.
- **Commit:** один FT = одна локальная учебная волна.
- **Verify before moving on:** каждый slice заканчивается `checks`, acceptance scenario и recorded evidence.

### Phase 7. Add UI only after client/CLI flows are deterministic

- **Goal:** сделать UI thin delivery layer, а не место, где впервые появляется поведение.
- **Minimal artifacts:** web screens, route handlers, app/session resolution, minimal Playwright regression coverage.
- **Commit:** сначала UI foundation, потом feature-specific UI slices.
- **Verify before moving on:** UI вызывает те же operations, которые уже проверены через `client/cli`; browser tests покрывают journeys, а не домен целиком.

### Phase 8. Perform feature-area refactor after behavior stabilizes

- **Goal:** перестроить код под локальный ownership до того, как GUI и scenarios разрастутся ещё сильнее.
- **Minimal artifacts:** ADR for slicing boundaries, updated structure docs, moved modules across `core/client/api-contract/cli/web`, regression evidence.
- **Commit:** сначала docs/rationale, потом code move, потом evidence closeout.
- **Verify before moving on:** public behavior не изменился, root entrypoints стали thin, feature areas читаются локально.

### Phase 9. Expand GUI, XE scenarios, guides and UI traceability

- **Goal:** довести систему до состояния, где она не только работает, но и обучаема, проверяема и операционно воспроизводима.
- **Minimal artifacts:** GUI waves, `XE` scenarios, token helpers if needed, tutorials/how-to/reference guides, screen registry, test ID registry, design system docs.
- **Commit:** короткие тематические волны, а не один мегакоммит.
- **Verify before moving on:** у UI есть stable identifiers, у сценариев есть deterministic execution path, у операторов есть reproducible guides.

## Suggested commit-by-commit learning path

Если ваша цель именно обучение, а не “быстро собрать клон”, используйте такой ритм.

1. `INIT` — пустой repo, pnpm workspace, README, базовые scripts.
2. `DOCS-01` — Memory Bank Bible и root indexes.
3. `DOCS-02` — project/spec/plans/adr skeleton.
4. `FT-0001` — workspace scaffold.
5. `FT-0002` — DB baseline and migration health.
6. `FT-0003` — seed runner with handles.
7. `FT-0005/0006` — web scaffold and baseline observability.
8. `FT-0011` — operation dispatcher and typed errors.
9. `FT-0012` — typed client transport and context.
10. `FT-0013` — first real CLI-backed vertical slice.
11. `EP-domain-*` — доменные FT slices сериями, каждая с acceptance и evidence.
12. `EP-ui-foundation` — UI shell only after previous loops are stable.
13. `EP-hardening` — checks, smoke, release discipline, observability.
14. `EP-structure` — feature-area refactor as a separate conscious move.
15. `EP-gui-wave` — richer UI surfaces.
16. `EP-scenarios-docs` — XE scenarios, guides, tutorials.
17. `EP-traceability` — screen IDs, test IDs, design system, docs hardening.

Учебное правило для каждой волны:

- один `EP` = одна тема роста системы;
- один `FT` = один vertical slice;
- каждый slice заканчивается кодом, тестами, docs updates и evidence;
- новый слой добавляется только тогда, когда предыдущий уже детерминированно работает.

## Common mistakes if recreating this approach

1. Начать с UI.
   - Это ломает саму идею deterministic delivery path и превращает браузер в место, где впервые появляется бизнес-логика.
2. Смешать `spec`, `plans` и `adr`.
   - Тогда система перестаёт понимать, где WHAT, где WHY, а где delivery slicing.
3. Хранить business rules в route handlers или CLI.
   - В таком случае UI/CLI перестают быть thin, и каждое расширение начинает дублировать домен.
4. Сделать `shared` свалкой.
   - Тогда feature-area refactor формально есть, а ownership всё равно размазан.
5. Писать acceptance без seed/handles.
   - Это почти гарантированно приводит к brittle сценариям и скрытым зависимостям от случайного состояния.
6. Воспроизводить только финальную структуру, но не эволюцию.
   - Тогда вы копируете форму, но не понимаете, почему и когда этот проект принял именно такие решения.
7. Пытаться закрывать feature “без evidence”.
   - В этом подходе completion без checks, acceptance и traceability считается недоказанным.
8. Игнорировать stable identifiers в UI.
   - Тогда browser automation и runtime traceability быстро становятся дорогими и ненадёжными.

## Reading order for study

Если вы хотите понять подход быстро, а не читать repo хаотично, идите так:

1. [`../.memory-bank/index.md`](../.memory-bank/index.md)
2. [`../.memory-bank/spec/project/system-overview.md`](../.memory-bank/spec/project/system-overview.md)
3. [`../.memory-bank/spec/project/layers-and-vertical-slices.md`](../.memory-bank/spec/project/layers-and-vertical-slices.md)
4. [`../.memory-bank/spec/engineering/architecture-guardrails.md`](../.memory-bank/spec/engineering/architecture-guardrails.md)
5. [`../.memory-bank/plans/implementation-playbook.md`](../.memory-bank/plans/implementation-playbook.md)
6. [`../.memory-bank/adr/0001-core-client-cli-first.md`](../.memory-bank/adr/0001-core-client-cli-first.md)
7. [`../.memory-bank/adr/0004-feature-area-slicing-boundaries.md`](../.memory-bank/adr/0004-feature-area-slicing-boundaries.md)
8. [`../packages/api-contract/src/campaigns.ts`](../packages/api-contract/src/campaigns.ts)
9. [`../packages/client/src/features/campaigns.ts`](../packages/client/src/features/campaigns.ts)
10. [`../packages/core/src/features/campaigns.ts`](../packages/core/src/features/campaigns.ts)
11. [`../packages/db/src/campaigns.ts`](../packages/db/src/campaigns.ts)
12. [`../apps/web/src/app/api/hr/campaigns/draft/route.ts`](../apps/web/src/app/api/hr/campaigns/draft/route.ts)
13. `git log --reverse --oneline`
14. Транскрипты из `_articles`, чтобы сопоставить repo reality с прямыми словами автора.

## Final takeaway

Если свести всё к одной формуле, то подход `feedback-360` можно описать так:

**Сначала структурируй знание, затем построй deterministic non-UI delivery path, затем выращивай домен vertical slices, затем добавь UI как thin surface, затем сделай structure and traceability strong enough, чтобы система оставалась понятной агентам и людям по мере роста.**

Именно эту последовательность и стоит faithfully воспроизводить в новом репозитории.
