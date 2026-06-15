## ЧАСТЬ VI — Curriculum, обучающий маршрут из 9 модулей

> 🧭 [[feedback-360-master-rebuild-walkthrough|Master Rebuild Walkthrough — оглавление]]


> 📚 **Справочник D (Reference).** Это не отдельный маршрут, а curriculum-разбор (9 обучающих модулей + 9 правил обучения), на который ссылается раздел «Как проходить». Канонический путь — Стадии 0–10.

Эта часть отличается от Части V. Часть V рассказывает, **как технически собрать** проект. Часть VI рассказывает, **как учиться, не превращая rebuild в бездумное копирование**.

Здесь используются два фиксированных пути:

- `REFERENCE_ROOT = C:\Users\mykha\Projects\feedback-360`
- `REBUILD_ROOT = <ваш отдельный учебный repo>`

`REFERENCE_ROOT` вы **читаете и изучаете**, но не используете как место для практики. `REBUILD_ROOT` — это ваш отдельный учебный repo, где вы **строите собственную версию подхода**.

### 9 правил обучения

Чтобы этот путь действительно научил подходу, а не только повторению файлов, соблюдайте следующие правила:

1. **Не копируйте готовые файлы из `REFERENCE_ROOT` до checkpoint.** Сначала попробуйте восстановить идею и структуру сами.
2. **Сначала читайте контекст, потом кодите.** Если модуль требует сначала открыть ADR, spec или companion-guide, делайте это до начала работы в `REBUILD_ROOT`.
3. **Сравнивайте себя с оригиналом только в контрольных точках.** Не держите оригинальный код открытым как «шпаргалку на каждый шаг».
4. **Не пытайтесь сразу «догнать весь проект».** В этом подходе важна последовательность: knowledge structure → deterministic delivery path → domain slices → UI → hardening → refactor → mature traceability.
5. **Не начинайте с UI.** Если у вас в `REBUILD_ROOT` раньше времени появляются React-экраны, значит вы ушли от метода.
6. **Не смешивайте `spec`, `plans` и `adr`.** Если не можете ответить, где у вас WHAT, где WHY, а где delivery slicing, структура уже поплыла.
7. **Не прячьте business logic в route handlers, CLI или UI.** Любая такая «маленькая валидация для удобства» быстро ломает thin-layer architecture.
8. **Не превращайте `shared` в свалку.** Если модуль имеет явного owner по feature area, он не должен жить в `shared`.
9. **Не считайте модуль завершённым без доказательства.** `Done when` в этом документе важнее внутреннего ощущения «ну, вроде сделал».

### Общая последовательность модулей

| Module | Theme | Main outcome |
|---|---|---|
| 0 | Method before code | Вы понимаете порядок и правила подхода |
| 1 | Monorepo foundation | У вас есть базовый workspace |
| 2 | Documentation skeleton | У вас есть Memory Bank skeleton |
| 3 | DB baseline + seeds | У вас есть deterministic data foundation |
| 4 | Contract + core + client + CLI | Появляется первый deterministic delivery path |
| 5 | First domain slices | Появляется реальная domain behavior |
| 6 | Thin web layer | UI/API поверх уже работающего client path |
| 7 | Hardening + quality gates | Вы учитесь закрывать delivery, а не только писать код |
| 8 | Feature-area refactor | Вы понимаете pressure и смысл structural refactor |
| 9 | XE + guides + UI traceability | Вы понимаете позднюю зрелость системы |

### Контракт модуля

Каждый модуль имеет одну и ту же структуру:

- **Goal** — что вы должны понять и построить.
- **Build in REBUILD_ROOT** — что именно делать в своём lab repo.
- **Read in REFERENCE_ROOT** — что изучить в оригинальном проекте.
- **Compare** — когда и как сверить свою реализацию с оригиналом.
- **Done when** — объективный checkpoint.
- **Artifacts** — что сохранить как доказательство и учебную заметку.

---

### Module 0: Понять методологию

#### Goal

Понять, что вы rebuild-ите не «набор технологий», а связанный workflow:

- docs-first SSoT;
- core + contract + client + CLI-first;
- vertical slices как delivery units;
- evidence-driven completion.

#### Build in REBUILD_ROOT

- Пока **ничего не кодить**.
- Создать в корне `REBUILD_ROOT` один личный файл заметок: `LEARNING_NOTES.md`.
- Зафиксировать в нём своими словами:
  - почему в этом подходе docs идут раньше кода;
  - чем `spec` отличается от `plans`;
  - зачем нужен CLI-first path;
  - почему UI deliberately delayed.

#### Read in REFERENCE_ROOT

- `docs/feedback-360-faithful-rebuild-guide.md`.
- `docs/feedback-360-step-by-step-rebuild.md`: Глава 0.
- `.memory-bank/index.md`.
- `.memory-bank/plans/implementation-playbook.md` (если есть; иначе аналог).
- `.memory-bank/adr/0001-core-client-cli-first.md`.

#### Compare

- Сравните свои заметки с формулировкой из `faithful-rebuild-guide`.
- Убедитесь, что вы можете объяснить подход **без ссылок на конкретный фреймворк**.

#### Done when

Вы можете устно или письменно ответить:

- почему первые документы важнее первого кода;
- почему `client` и CLI появляются раньше UI;
- почему slice здесь важнее «сначала весь DB layer».

#### Artifacts

- `LEARNING_NOTES.md` с коротким summary метода.

---

### Module 1: Поднять monorepo foundation

#### Goal

Собрать минимальный технический каркас в `REBUILD_ROOT`, на который потом можно навешивать все остальные идеи.

#### Build in REBUILD_ROOT

- Инициализировать новый Git repo.
- Поднять pnpm workspace.
- Создать root `package.json`, `pnpm-workspace.yaml`, `.gitignore`.
- Завести минимальные root scripts: `lint`, `typecheck`, `test`, `checks`.
- Создать верхнеуровневые каталоги: `apps/`, `packages/`, `.memory-bank/`.

#### Read in REFERENCE_ROOT

- `docs/feedback-360-step-by-step-rebuild.md`: Глава 1.
- `package.json`.
- `pnpm-workspace.yaml`.
- `.memory-bank/plans/epics/EP-000-foundation/features/index.md`.

#### Compare

- Сверьте свои root scripts и workspace layout с оригиналом.
- Не добивайтесь byte-to-byte совпадения; добивайтесь **совпадения роли каждого элемента**.

#### Done when

- `pnpm install` проходит.
- `pnpm checks` запускается хотя бы на минимальном scaffold.
- Структура верхнего уровня читается как monorepo, а не как случайная папка с кодом.

#### Artifacts

- Первый commit в `REBUILD_ROOT`.
- Скрин или текстовый вывод успешного `pnpm checks`.

---

### Module 2: Собрать documentation skeleton

#### Goal

Понять и воспроизвести knowledge structure, без которой весь остальной rebuild теряет смысл.

#### Build in REBUILD_ROOT

- Создать минимальный skeleton для `.memory-bank/spec/`, `.memory-bank/plans/`, `.memory-bank/adr/`, `.memory-bank/mbb/`.
- Добавить root index и индексы ключевых разделов.
- Добавить минимум:
  - repo structure;
  - layers and vertical slices;
  - architecture guardrails;
  - implementation playbook;
  - feature template;
  - ADR-0001.

#### Read in REFERENCE_ROOT

- `docs/feedback-360-step-by-step-rebuild.md`: Глава 2.
- `.memory-bank/index.md`.
- `.memory-bank/mbb/index.md`.
- `.memory-bank/mbb/principles.md`.
- `.memory-bank/spec/project/layers-and-vertical-slices.md`.
- `.memory-bank/spec/engineering/architecture-guardrails.md`.
- `.memory-bank/mbb/templates/feature.md`.

#### Compare

- Проверьте, что ваши документы разделяют **WHAT / WHY / HOW of delivery**.
- Сравните не wording, а **structure and purpose**.

#### Done when

- По root index можно понять, где искать правила, планы и rationale.
- Вы можете создать новую feature page по своему template без изобретения формата с нуля.

#### Artifacts

- Документ `REBUILD_ROOT/.memory-bank/index.md`.
- Черновой ADR-0001.
- Один пустой feature template или example FT page.

---

### Module 3: DB baseline и seed runner

#### Goal

Собрать deterministic foundation данных, чтобы следующие модули опирались не на «ручную базу», а на воспроизводимые fixtures.

#### Build in REBUILD_ROOT

- Создать пакет `packages/db`.
- Выбрать и зафиксировать DB layer (рекомендуется Drizzle, но можно другой).
- Добавить:
  - baseline schema/migrations;
  - health-check;
  - seed runner;
  - **handles-идею** вместо жёстких numeric ids.

#### Read in REFERENCE_ROOT

- `docs/feedback-360-step-by-step-rebuild.md`: Глава 3.
- `.memory-bank/plans/epics/EP-000-foundation/features/FT-0002-db-migrations-baseline/index.md`.
- `.memory-bank/plans/epics/EP-000-foundation/features/FT-0003-seed-runner-handles/index.md`.
- `.memory-bank/spec/testing/traceability.md`.
- git range: `508ba7e..9eaddee`.

#### Compare

- Сравните не только migration files, но и **саму идею deterministic seed contract**.
- Проверьте, что ваши seeds возвращают stable handles, а не требуют «посмотреть в базе руками».

#### Done when

- Вы можете поднять пустую базу.
- Миграции применяются детерминированно.
- Один seed scenario создаёт воспроизводимое состояние и возвращает handles.

#### Artifacts

- Текстовый вывод миграций/health-check.
- Пример JSON результата seed runner.

---

### Module 4: api-contract + core + client + CLI

#### Goal

Построить первый полноценный deterministic delivery path **без UI**.

#### Build in REBUILD_ROOT

- Создать пакеты: `api-contract`, `core`, `client`, `cli`.
- Добавить:
  - typed operation model;
  - typed error model;
  - input/output parsing;
  - runtime invoke layer;
  - один CLI command с human output и `--json`.

**Одна работающая operation важнее десяти абстракций.**

#### Read in REFERENCE_ROOT

- `docs/feedback-360-step-by-step-rebuild.md`: Главы 4–7.
- `.memory-bank/adr/0001-core-client-cli-first.md`.
- `packages/api-contract/src/campaigns.ts`.
- `packages/client/src/features/campaigns.ts`.
- `packages/core/src/features/campaigns.ts`.
- git range: `d91182e..0d34647`.

#### Compare

- Проверьте, что ваш CLI **не знает бизнес-правил**.
- Проверьте, что ваш client **не дублирует** route/web behavior.
- Проверьте, что core можно тестировать без UI.

#### Done when

- Одна операция проходит путь `contract → core → db → client → cli`.
- `CLI --json` возвращает стабильную схему.
- Ошибки typed, а не «raw exception dump».

#### Artifacts

- Пример вызова CLI в human mode.
- Пример вызова CLI в `--json`.
- Короткая схема слоёв для вашей первой операции.

---

### Module 5: Первые domain slices

#### Goal

Научиться добавлять функциональность **не по слоям, а по vertical slices** с user value, acceptance и traceability.

#### Build in REBUILD_ROOT

- Выберите 1–2 первых учебных доменных slice-а.
- Для каждого сделайте:
  - FT page;
  - contract updates;
  - core use-case/policy;
  - DB changes if needed;
  - tests;
  - docs update.

**Не пытайтесь сразу повторить весь HR domain.** Важнее освоить ритм slice delivery.

#### Read in REFERENCE_ROOT

- `.memory-bank/mbb/templates/feature.md`.
- `.memory-bank/plans/implementation-playbook.md`.
- `.memory-bank/plans/epics/EP-001-core-contract-client-cli/features/index.md`.
- Примеры ранних domain slices из `EP-002`..`EP-006`.
- git ranges: `8edb962..7473a5f`, `886dbb4..328bdb0`.

#### Compare

- Сверьте, что ваш slice можно объяснить как user value, **а не как «добавил таблицу и пару методов»**.
- Сверьте, что completion зафиксирован **не только кодом**, но и acceptance.

#### Done when

- У вас есть хотя бы два вертикальных slice-а с одинаковым delivery rhythm.
- Вы можете показать, где в каждом slice находится: спецификация, код, тест, evidence.

#### Artifacts

- Две feature pages.
- Два проходящих acceptance/test scenarios.
- Запись в `LEARNING_NOTES.md`: что оказалось самым трудным в slice-based delivery.

---

### Module 6: Тонкий web layer

#### Goal

Понять, как UI/API слой приходит **после того, как non-UI delivery path уже стабилен**.

#### Build in REBUILD_ROOT

- Создать `apps/web`.
- Добавить минимальный runtime shell.
- Реализовать один-два route handlers поверх уже существующих operations.
- Следить, чтобы web layer:
  - резолвил context;
  - валидировал input;
  - делегировал в client/core path;
  - **не встраивал business rules**.

#### Read in REFERENCE_ROOT

- `docs/feedback-360-step-by-step-rebuild.md`: Глава 12.
- `apps/web/src/app/api/hr/campaigns/draft/route.ts`.
- Ранние UI коммиты `b9037e9..7868ea0`.
- `.memory-bank/spec/ui/` (если есть).

#### Compare

- Сравните, где заканчивается parsing/adapter logic и начинается use-case.
- Если web layer начал тянуть в себя rules, **остановитесь и вернитесь назад**.

#### Done when

- Ваш web layer работает поверх существующего deterministic path.
- **Вы можете удалить UI и не потерять доменную логику.**

#### Artifacts

- Один working route flow.
- Короткая заметка: что в web layer допустимо, а что уже должно жить в core.

---

### Module 7: Hardening и quality gates

#### Goal

Научиться **закрывать работу по engineering standards**, а не только «дописывать feature».

#### Build in REBUILD_ROOT

- Ввести обязательный `checks` ritual.
- Добавить/усилить:
  - lint;
  - typecheck;
  - tests;
  - basic build step;
  - minimal acceptance evidence discipline.
- Описать у себя, что считается completed slice.

#### Read in REFERENCE_ROOT

- `.memory-bank/spec/engineering/testing-standards.md`.
- `.memory-bank/spec/engineering/delivery-standards.md`.
- `.memory-bank/plans/verification-matrix.md`.
- git range: `cf6106f..462a487`.

#### Compare

- Сверьте, что ваш «готово» означает не только код merge-able, но и **доказуемо проверено**.
- Сверьте, что acceptance и quality gates разделены.

#### Done when

- Вы не переходите к следующему slice без объективной проверки.
- В `REBUILD_ROOT` есть простая, но формальная delivery discipline.

#### Artifacts

- Вывод `checks`.
- Один example acceptance report или verification note.

---

### Module 8: Feature-area refactor

#### Goal

Понять, **почему layer-flat structure сначала терпима, а потом начинает мешать**, и как правильно делать structural refactor как отдельную волну.

#### Build in REBUILD_ROOT

- **Не делайте этот модуль слишком рано.**
- Дождитесь момента, когда у вас уже есть несколько domain slices и растущее количество root files.
- Затем:
  - зафиксируйте rationale в ADR;
  - опишите target feature areas;
  - перенесите модули по ownership;
  - сохраните public behavior;
  - обновите docs.

#### Read in REFERENCE_ROOT

- `.memory-bank/adr/0004-feature-area-slicing-boundaries.md`.
- `.memory-bank/spec/project/feature-area-boundaries.md` (если есть).
- `.memory-bank/plans/epics/EP-014-feature-area-slices-refactor/index.md` (если есть).
- git range: `7ffa297..674f596`.

#### Compare

- Сравнивайте не просто новое дерево папок, а то, **локализовалась ли зона изменений**.
- Если после refactor ownership всё ещё непонятен — **refactor не удался**.

#### Done when

- Вы можете назвать свои canonical feature areas.
- Root entrypoints стали thin.
- Shared-модули имеют явное обоснование, почему они shared.

#### Artifacts

- Ваш ADR про boundaries.
- Diff-схема «до/после».
- Короткий postmortem: какие pain points заставили вас рефакторить.

---

### Module 9: XE, guides, UI traceability

#### Goal

Понять, что зрелый проект — это не только код и tests, но и **сценарный runtime, операторские guides и строгая UI traceability**.

#### Build in REBUILD_ROOT

- Не обязательно fully реализовывать весь XE stack.
- На учебном уровне достаточно:
  - описать один cross-slice scenario;
  - добавить structured artifacts expectation;
  - завести stable UI identifiers, если у вас уже есть UI;
  - попробовать сделать хотя бы skeleton scenario runner conceptually.

#### Read in REFERENCE_ROOT

- `.memory-bank/plans/epics/EP-020-cross-epic-scenarios/index.md` (если есть).
- `.memory-bank/spec/ui/screen-registry.md` (если есть).
- `.memory-bank/spec/ui/test-id-registry.md` (если есть).
- `.memory-bank/spec/ui/pom/index.md` (если есть).
- git ranges: `c85a165..45496ff`, `9f6f5a3..a8bc502`.

#### Compare

- Проверьте, что вы понимаете: XE и UI traceability — это **не стартовая точка**, а поздняя зрелость.
- Не пытайтесь загнать это в `REBUILD_ROOT` слишком рано.

#### Done when

- Вы можете объяснить, почему XE scenarios и UI traceability появились поздно, а не в первых коммитах.
- У вас есть хотя бы один собственный пример cross-slice scenario description.

#### Artifacts

- Один scenario note.
- Один список stable UI identifiers, если ваш rebuild уже дошёл до UI.

---

### Как сверять себя с оригинальным проектом

После каждого модуля делайте compare в **три шага**:

1. Сначала сравните **идею** (используйте `faithful-rebuild-guide`).
2. Потом сравните **практический способ сборки** (используйте `step-by-step-rebuild` или Часть V этого файла).
3. Только после этого сравните **реальные файлы и commit phases** в `REFERENCE_ROOT`.

Используйте такую дисциплину:

- После Module 1–3 смотрите в основном `EP-000`.
- После Module 4–5 — `EP-001` и ранние domain epics.
- После Module 6 — раннюю UI phase.
- После Module 7 — prod-readiness/hardening wave.
- После Module 8 — `EP-014`.
- После Module 9 — `EP-020`, `EP-021`, `EP-022`, `EP-023`.

**Чего не делать при сверке:**

- Не копировать файл только потому, что он «красивее».
- Не пытаться совпасть по каждому имени symbol.
- Не использовать оригинальный repo как development workspace для rebuild.

**Правильный вопрос при сверке:**

> «Я воспроизвёл тот же engineering move?»

**Неправильный вопрос:**

> «У меня такой же файл до последней строки?»

### Признаки усвоения подхода

Вы усвоили подход, если можете объяснить следующее **без подсказок**:

1. Почему порядок именно такой: docs → foundation → contract/core/client/cli → domain slices → UI → hardening → feature-area refactor → XE/traceability.
2. Почему CLI появляется раньше rich UI.
3. Почему Memory Bank здесь не просто docs folder.
4. Почему vertical slice важнее «сначала полностью база, потом полностью API».
5. Почему feature-area refactor был выделен в отдельную волну.
6. Почему quality gates и evidence — часть delivery, а не decoration.
7. Почему `shared` должен быть исключением, а не default bucket.

**Слабый признак усвоения:** «Я собрал похожую структуру папок».

**Сильный признак усвоения:** «Я могу обосновать архитектурный порядок, pressure points и правила завершения работы».

### Рекомендуемый ритм самообучения

Так как путь self-paced, ориентируйтесь не на недели, а на качество прохождения модулей.

**Хороший ритм:**

- Один модуль за сессию или за несколько коротких сессий.
- Compare делается только **после собственного build attempt**.
- После каждого модуля вы пишете 5–10 строк reflection в `LEARNING_NOTES.md`.

**Если модуль «проходится слишком быстро»**, это обычно значит одно из двух:

- либо вы уже понимаете метод и честно сокращаете шаги;
- либо **вы начали копировать вместо rebuilding**.