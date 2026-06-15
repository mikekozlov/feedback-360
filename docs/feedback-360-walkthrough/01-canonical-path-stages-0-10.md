## Канонический путь — Стадии 0–10

> 🧭 [[feedback-360-master-rebuild-walkthrough|Master Rebuild Walkthrough — оглавление]]


### Стадия 0 — Методология и mindset

> Исход: код ещё **не пишем**. На этой стадии вы перестраиваете не репозиторий, а голову.

#### 🎯 Зачем эта стадия (на пальцах)

Представьте, что вы наняли гениального, но абсолютно безынициативного подрядчика: он выполняет ТЗ буквально и забывает всё между задачами. Это и есть AI-агент. Дайте ему расплывчатую «хотелку» — он **выдумает** недостающее. И это не баг, а ровно то, за что мы ИИ ценим… но сейчас нам нужен **наш** результат, а не его фантазия. Значит, всю неопределённость надо снять **до** кода, а правила проекта сложить так, чтобы подрядчик каждый раз находил их сам. Вся методология — про одно: сделать работу агента **предсказуемой**.

Рамка от автора: жизненный цикл разработки не изменился (хотелка → спецификация → план → код → верификация → деплой) — **изменился исполнитель**. Раньше человек проходил эти этапы руками; теперь их проходит агент, а человек **ставит задачу и проверяет результат**.

#### 🧠 Ключевые концепции простыми словами

- **Полный цикл, другой исполнитель.** Суть: те же 6 этапов, но их ведёт агент. Аналогия: конвейер тот же, у станка теперь робот. Почему важно: вы проектируете не «чат с ИИ», а управляемый конвейер.
- **Docs раньше кода (specs-first).** Суть: сначала фиксируем WHAT/WHY, потом код. Аналогия: чертёж до стройки. Почему важно: агент (и новый человек) читает Memory Bank и сразу работает, не спрашивая «а где это и почему так».
- **Прогрев контекста.** Суть: первый промпт заставляет агента **изучить** проект, второй — **специализировать** контекст под подсистему. Аналогия: прежде чем поручить задачу новичку, дать ему день на чтение вики. Почему важно: без прогрева агент опирается на догадки.
- **Вертикальный слайс.** Суть: единица ценности через **все** слои (UI/API/БД/сервер), а не «кусок слоя». Аналогия: не «залить все фундаменты в городе», а «построить один дом целиком». Почему важно: локализованная зона изменений = естественная единица приёмочного теста.
- **CLI-first, пирамида знаний, evidence-first** вводятся здесь как принципы и разворачиваются в Стадиях 4, 9 и Справочнике A. Расширенное «почему» — [[03-part-2-philosophy#ЧАСТЬ II — Философия подхода (the why)|Справочник: Часть II «Философия»]].

#### 🔧 Иллюстративный пример

«Хочу систему рекомендаций в интернет-магазине» — это **не** постановка задачи. Хорошая постановка возникает только после встречных вопросов агента: для каких клиентов? какие товары включены? нужны ли нишевые сегменты? выдавать список, подсказывать категории или показывать «эфемерный товар» в корзине одним кликом? Тот же приём для feedback-360: «хочу 360-оценку» → агент обязан спросить про роли, оргструктуру, модели компетенций, политику анонимности — иначе **выдумает**.

#### 📐 Диаграмма: цикл и точка обратной связи

```mermaid
flowchart LR
    B["Хотелка / Brief<br/>прогретый контекст"] --> S["Спецификация<br/>WHAT + grounding"]
    S --> P["План<br/>Epic / Feature, слайсы"]
    P --> C["Код<br/>vertical slice"]
    C --> V["Верификация<br/>acceptance + checks"]
    V --> D["Деплой<br/>local → beta → prod"]
    V -.->|"расхождение с критерием:<br/>агент добивает разницу"| C
```

#### 🏗️ Что построить (REBUILD_ROOT)

- Кода — **ноль**. Создайте `LEARNING_NOTES.md`.
- Запишите своими словами ответы: почему docs раньше кода; чем `spec` отличается от `plans` и `adr`; зачем нужен CLI-first; почему UI намеренно откладывается.
- Сформулируйте «формулу подхода» одним абзацем **без** слов React/Next/Drizzle (метод ≠ стек).

#### 📖 Что прочитать (REFERENCE_ROOT)

- `_articles/Agentic Engineering … DEKSDEN часть 1_best.notes.md`, темы 1–8 — что лежит: первоисточник метода; зачем: услышать формулировки автора напрямую.
- `.memory-bank/adr/0001-core-client-cli-first.md` — что лежит: короткое ADR; зачем: увидеть «почему» в формате решения с trade-offs.
- [[03-part-2-philosophy#ЧАСТЬ II — Философия подхода (the why)|Справочник: Часть II]] — что лежит: 8 причин «почему этот проект учебный»; зачем: расширенное обоснование метода.

#### ✅ Верификация (стадия пройдена, когда)

Стадия концептуальная — критерий **через объяснение** (по Фейнману: «понял = можешь объяснить просто»):

- [ ] Объясняете формулу подхода устно за 2 минуты, без названий фреймворков.
- [ ] Отвечаете на 3 контрольных вопроса (эталоны — в [[08-part-7-antipatterns#7 контрольных вопросов для самопроверки]]): чем ADR отличается от spec? почему `apps/web` не импортирует `core` напрямую? что такое evidence?
- [ ] В `LEARNING_NOTES.md` есть абзац «почему docs раньше кода» — и он про **предсказуемость агента**, а не про «так аккуратнее».

Given-when-then самопроверки: **Given** вам дали расплывчатую хотелку, **When** вы начинаете работу, **Then** ваш первый шаг — не код, а встречные вопросы и grounding, и вы можете объяснить, почему.

#### 🔗 Выравнивание с автором

Концепции 1–5, 17 из [[#Выравнивание с методологией автора (DEKSDEN, части 1–2)]]. Якорь: «цикл разработки не изменился — изменился исполнитель» (часть 1, тема 1).

#### ⚠️ Антипаттерны стадии · 🧩 Артефакты

- Антипаттерны: начать со стека/кода вместо метода ([[08-part-7-antipatterns#Антипаттерн 1: Начать с UI]]); принять первую формулировку хотелки как ТЗ (нет gap-closing).
- Артефакты: `LEARNING_NOTES.md` с формулой подхода и ответами на «почему».

---

### Стадия 1 — Monorepo foundation

> Исход: минимальный технический каркас (`pnpm install` + `pnpm lint` зелёные), на который навешиваются все остальные идеи.

#### 🎯 Зачем эта стадия (на пальцах)

В Стадии 0 вы перестроили голову, но кода всё ещё ноль. Теперь нужна **мастерская**, в которой подрядчик-агент будет работать предсказуемо: один и тот же верстак, один набор инструментов, один способ запустить проверку. Представьте бригаду, где у каждого своя линейка, своя пила и своё «как правильно отступать»: сборка превратится в спор о форматировании вместо стройки. Стадия 1 — это про то, чтобы **до** первой доменной фичи зафиксировать инструменты, версии и команды так, чтобы `pnpm install` и `pnpm lint` были зелёными на пустом каркасе, а любой пакет внутри monorepo подчинялся одному контракту.

Рамка от автора: смысл каркаса — **детерминизм и экономия внимания модели**. Один инструмент вместо трёх (Biome вместо ESLint+Prettier), запинненные версии вместо «латест», единый `tsconfig.base.json` вместо копий — это не аккуратность ради аккуратности, это `simplify`, чтобы агент каждый раз попадал в одну и ту же среду и не выдумывал недостающее. Каркас сам по себе ничего не делает (`configPackageReady = true` — единственная строка кода) — но именно на него потом навешиваются все остальные идеи.

#### 🧠 Ключевые концепции простыми словами

- **pnpm workspaces (monorepo как единый контекст).** Суть: `apps/*` и `packages/*` живут в одном репозитории под одним lockfile-ом и одним менеджером пакетов. Аналогия: не десять отдельных квартир с разными счётчиками, а один дом с общим щитком. Почему важно: агент видит весь проект сразу, нет phantom-зависимостей (нельзя случайно импортировать то, что не объявлено), а workspace-ссылки `workspace:*` связывают пакеты без публикации в реестр.
- **Recursive fan-out (`pnpm -r`).** Суть: корневой `pnpm -r lint` прогоняет одноимённый скрипт `lint` в каждом пакете; то же для `typecheck`, `test`, `format`. Аналогия: один рубильник на щитке включает свет во всех комнатах сразу. Почему важно: единая команда-вход на весь monorepo, и при этом каждый пакет обязан соблюдать один контракт скриптов — иначе fan-out на нём провалится.
- **Один инструмент = lint + format + organizeImports (Biome).** Суть: Biome заменяет связку ESLint+Prettier — один конфиг, один CLI, одна зависимость. Аналогия: швейцарский нож вместо трёх отдельных лезвий в разных карманах. Почему важно: меньше конфигов и версий = меньше точек рассинхрона = меньше внимания модели тратится на «какой линтер тут главный». Это прямое проявление принципа `simplify`.
- **Запинненные версии (детерминированный lockfile).** Суть: `packageManager: "pnpm@10.5.0"` и точные версии devDependencies (`typescript 5.8.2`, `vitest 3.2.4`, `biome 1.9.4`) без диапазонов `^`/`~`. Аналогия: рецепт, где написано «200 г муки», а не «сколько-то муки». Почему важно: у вас и у агента сборка байт-в-байт одинаковая; обновление инструмента — это осознанный коммит, а не сюрприз от «латеста».
- **Единый `strict`-tsconfig для всех пакетов.** Суть: корневого `tsconfig.json` нет — есть общий `packages/config/tsconfig.base.json` (`strict: true`, плюс `noUncheckedIndexedAccess`), который наследуют все пакеты. Аналогия: единый ГОСТ, на который ссылается каждый чертёж, вместо своих допусков в каждом цеху. Почему важно: типобезопасность одинакова везде, и `pnpm typecheck` на каркасе зелёный именно потому, что все наследуют один строгий базис.

#### 🔧 Иллюстративный пример: пакет @feedback-360/config — минимальный гражданин monorepo

Самый маленький полноценный пакет каркаса показывает контракт, которому подчинится любой будущий доменный пакет (`core`, `db`, `client`) — это эталонный минимальный «гражданин» monorepo:

1. **Манифест** (`packages/config/package.json`): `name: "@feedback-360/config"`, `type: "module"`, и ровно четыре скрипта — `lint: "biome check ."`, `format: "biome format --write ."`, `typecheck: "tsc --noEmit -p tsconfig.json"`, `test: "vitest run --passWithNoTests"`. Эти имена — то, что собирает корневой `pnpm -r`.
2. **Доменного кода нет** (`packages/config/src/index.ts`): единственная строка `export const configPackageReady = true;` — smoke-якорь, доказывающий, что каркас компилируется и экспортируется.
3. **Smoke-тест** (`packages/config/src/smoke.test.ts`): `describe("config package")` + `it("exposes a smoke constant")` ждёт `expect(configPackageReady).toBe(true)`. Это доказывает, что `vitest 3.2.4` реально запускается на пустом каркасе, а не просто «настроен на бумаге».

Когда позже появится `@feedback-360/core` — он принесёт ровно тот же контракт скриптов (`lint`/`format`/`typecheck` идентичны, `test` лишь добавит `cross-env FEEDBACK360_SKIP_DB_TESTS=1`). То есть `pnpm -r` агрегирует **один и тот же контракт** через все пакеты — `config` просто его эталонный минимум.

#### 📐 Диаграмма: monorepo и recursive fan-out

```mermaid
flowchart TD
    ROOT["feedback-360 (root)<br/>package.json · pnpm-workspace.yaml<br/>biome.json"]
    ROOT --> WS["pnpm workspaces<br/>globs: apps/* · packages/*"]
    WS --> APPS["apps/*<br/>deployment artefacts"]
    WS --> PKGS["packages/*<br/>libs @feedback-360/*"]
    PKGS --> CFG["packages/config<br/>tsconfig.base.json (strict)<br/>src/index.ts + smoke.test.ts"]
    ROOT -->|"pnpm -r (fan-out)"| FAN["в каждом пакете:<br/>lint · typecheck · test · format"]
    FAN -->|"biome check ."| L["lint зелёный"]
    FAN -->|"tsc --noEmit"| T["typecheck зелёный"]
    FAN -->|"vitest run"| TST["smoke-тест проходит"]
    CFG -.->|"наследуют strict"| PKGS
```

#### 🏗️ Что построить (REBUILD_ROOT)

Снизу вверх — каркас, не код (пошаговая реконструкция — [[06-part-5-technical-restoration#Глава 1: Monorepo Foundation|Справочник: Глава 1, шаги 1.1–1.6]] и [[07-part-6-curriculum#Module 1: Поднять monorepo foundation|Module 1]]):

1. **Инициализация** (`Шаг 1.1`): `git init` + `pnpm init` в `REBUILD_ROOT`. Первый коммит — это артефакт стадии (см. Module 1 Artifacts).
2. **Корневой `package.json`** (`Шаг 1.2`): `private: true`, `packageManager: "pnpm@10.5.0"`, запинненные `devDependencies` (`@biomejs/biome 1.9.4`, `typescript 5.8.2`, `vitest 3.2.4`, `@playwright/test 1.54.2`, `@types/node 22.13.8`, `cross-env 7.0.3`), recursive fan-out скрипты `lint`/`typecheck`/`test`/`format` (каждый — `pnpm -r <name>`) и агрегат `checks`. Примечание: в Главе 1 `checks` показан упрощённо (`lint && typecheck && test`); реальный корневой `checks` добавляет `test:db` и `pnpm --filter @feedback-360/web build` — они станут зелёными позже, когда появятся `db`/`web`.
3. **`pnpm-workspace.yaml`** (`Шаг 1.3`): два glob-а — `apps/*` (deployment artefacts) и `packages/*` (библиотеки `@feedback-360/*`).
4. **`biome.json`** (`Шаг 1.4`): `$schema` на `1.9.4`, `organizeImports.enabled: true`, `linter.rules.recommended: true`, `formatter` со `space`/`indentWidth 2`/`lineWidth 100`, `files.ignore` (`node_modules`, `dist`, `.next`, `coverage`, `test-results`, `playwright-report`).
5. **`packages/config/tsconfig.base.json`** (`Шаг 1.5`): общий `strict`-базис (`target ES2022`, `module ESNext`, `moduleResolution Bundler`, `strict: true`, `noUncheckedIndexedAccess: true`, `noEmit: true`, `types: ["node"]`). Корневого `tsconfig.json` **нет** — пакеты наследуют этот файл.
6. **Каталоги и эталонный пакет** (`Шаг 1.6`): `mkdir` верхнего уровня `apps/`, `packages/`, `.memory-bank/`; внутри — `packages/config` с per-package скриптами, `src/index.ts` (`configPackageReady`) и `src/smoke.test.ts`. Верхний уровень должен **читаться как monorepo** с первого взгляда.

#### 📖 Что прочитать (REFERENCE_ROOT)

- [[06-part-5-technical-restoration#Глава 1: Monorepo Foundation]] — что лежит: пошаговый справочник стадии (почему pnpm workspaces, шаги 1.1–1.6, Checkpoint `pnpm install` + `pnpm lint`); зачем: основная обучающая реконструкция каркаса.
- [[04-part-3-architecture#3.1. Технологический стек с версиями]] — что лежит: таблица стек/версия/зачем (TypeScript 5.8.x, pnpm 10+, Biome 1.9.x, Vitest 3.2.x …); зачем: обоснование пиннинга версий и фраза «Biome заменяет одновременно ESLint + Prettier … пример упрощения кода из принципа 2.7».
- [[04-part-3-architecture#3.2. Структура monorepo]] — что лежит: mermaid-дерево целевой структуры (`apps/web`, `packages/cli`, `packages/*`, root config) с цветовой кодировкой; зачем: визуально зафиксировать целевую форму и место `packages/config` и `tsconfig.base.json`.
- [[07-part-6-curriculum#Module 1: Поднять monorepo foundation]] — что лежит: учебный модуль с Goal/Build/Read/Done when/Artifacts; зачем: критерии «готово» и список артефактов для воспроизведения.

#### ✅ Верификация (стадия пройдена, когда)

Запускаемый критерий (это и есть **evidence**, а не «вроде поставилось»):

```bash
# Given: собран минимальный каркас (root config + packages/config)
pnpm install            # детерминированный lockfile под pnpm@10.5.0, без ошибок
pnpm lint               # pnpm -r lint → biome check . в каждом пакете — зелёный

# When: прогоняем остальной fan-out на каркасе
pnpm typecheck          # pnpm -r typecheck → tsc --noEmit — зелёный (все наследуют strict)
pnpm -r test            # в packages/config: vitest run --passWithNoTests → smoke.test.ts проходит
pnpm format             # biome format --write . — идемпотентно (нет диффа на отформатированном каркасе)

# Then: агрегатный quality gate запускается на минимальном scaffold
pnpm checks             # lint && typecheck && test && test:db && web build
                        # на Стадии 1 запускается; полностью зелёным станет после db/web
```

Чеклист:

- [ ] `pnpm install` проходит без ошибок, lockfile детерминирован под `pnpm@10.5.0`.
- [ ] `pnpm lint` зелёный на пустом каркасе (первый verifiable-критерий стадии).
- [ ] `pnpm -r test` запускает `vitest`, и `smoke.test.ts` проходит (`configPackageReady === true`).
- [ ] Верхний уровень читается как monorepo: видны `apps/`, `packages/`, root config (`pnpm-workspace.yaml`, `biome.json`, `tsconfig.base.json`).
- [ ] Сделан первый git commit; сохранён вывод/скрин успешного `pnpm checks`.

Given-when-then: **Given** собраны корневые конфиги и `packages/config`, **When** вы запускаете `pnpm install` и затем `pnpm lint`, **Then** обе команды зелёные на пустом каркасе — и вы можете объяснить, почему один инструмент (Biome) и запинненные версии дают агенту детерминированную среду.

#### 🔗 Выравнивание с автором

Концепция 10 из [[#Выравнивание с методологией автора (DEKSDEN, части 1–2)]] («Цикл под управлением; Opus „срезает углы“; simplify»). Оговорка по карте соответствия: в таблице эта концепция формально закреплена за Стадией 9, здесь она применяется к каркасу Стадии 1 как **ранний гардрейл** — `simplify` начинается не с ревью кода, а с самого верстака. Якоря: «упрощение дает нам экономию внимания модели.» (часть 1, тема 8 — эволюция верификации, Opus и `simplify`) — ровно об этом один инструмент Biome и единый `tsconfig.base.json`; «Боритесь с over-engineering **и в промпте планирования, и на ревью** (simplify); особенно с Opus — обкладывайте гардрейлами.» (часть 1, раздел «Что взять с собой») — каркас и есть первый гардрейл: запинненные версии, recursive scripts, strict TS.

#### ⚠️ Антипаттерны стадии · 🧩 Артефакты

- Антипаттерны: начать с UI/React/Next вместо каркаса ([[08-part-7-antipatterns#Антипаттерн 1: Начать с UI]]) — Стадия 1 как раз ставит non-UI каркас (`packages/config`, recursive scripts, Biome, TS strict) до любого UI; скопировать только финальную структуру без понимания «почему» ([[08-part-7-antipatterns#Антипаттерн 6: Копировать только финальную структуру]]) — Module 1 требует совпадения **роли** каждого элемента и повторения эволюции, а не byte-to-byte клона готового дерева.
- Артефакты: корневой `package.json` (`packageManager pnpm@10.5.0`, запинненные devDependencies, recursive fan-out скрипты + агрегат `checks`); `pnpm-workspace.yaml` (`apps/*`, `packages/*`); `biome.json` (organizeImports on, recommended, space/2/100, files.ignore); `packages/config/tsconfig.base.json` (общий strict, корневого tsconfig нет); пакет `@feedback-360/config` (per-package скрипты, `src/index.ts` с `configPackageReady`, `src/smoke.test.ts`); верхнеуровневые `apps/` и `packages/` (+ `.memory-bank/`), читаемые как monorepo; первый git commit; текстовый вывод/скрин успешного `pnpm checks` (`pnpm install` + `pnpm lint` зелёные на пустом каркасе).

---

### Стадия 2 — Memory Bank и documentation skeleton

> Исход: Memory Bank skeleton (MBB principles, `index.md` с annotated links, разделённые `spec`/`plans`/`adr`, ADR-0001, implementation-playbook).

#### 🎯 Зачем эта стадия (на пальцах)

Представьте, что вы передаёте проект новому сотруднику, но у вас есть всего одно правило: он не имеет права задать ни одного вопроса. Всё, что он не найдёт записанным, он **домыслит сам** — и с большой вероятностью неправильно. Ровно так работает AI-агент. Стадия 2 — это про то, чтобы заранее разложить знание о проекте по полочкам так, чтобы агент **сам** находил, где правило, где план и где обоснование, не выкачивая в контекст весь репозиторий и не выдумывая недостающее. Memory Bank — не «папка с заметками», а **движок контекста**: структурированная память, по которой агент добирает ровно то, что нужно для текущей задачи.

Рамка от автора: код хранит **как** сделано; документация над кодом хранит **почему** и **для чего**. Поэтому скелет документации строится **до** первого боевого кода (в реальном репозитории коммиты с Memory Bank появились раньше первого кода — `37139c0`/`ee0f0a7`/`b1962de`). Сначала вы фиксируете границы знания, и только потом наполняете их.

#### 🧠 Ключевые концепции простыми словами

- **SSoT (Single Source of Truth).** Суть: у каждой концепции/правила есть **ровно один** нормативный документ, остальные на него ссылаются. Аналогия: один официальный паспорт, а не пять разных копий с расхождениями. Почему важно: когда фактов нет дублей между `spec/` и кодом, между `plans/` и README, агент никогда не получает два противоречивых ответа на «а как правильно».
- **WHY / WHAT / HOW (пирамида знаний).** Суть: `spec/` отвечает на WHAT (норматив), `adr/` — на WHY (почему так решили), `plans/` — на HOW of delivery (в каком порядке делаем и как проверяем), а сам код — на HOW (детали реализации). Аналогия: чертёж (WHAT), пояснительная записка архитектора (WHY), график стройки (delivery) и сам дом (HOW). Почему важно: агент на вопрос «откуда это правило?» всегда показывает **одно** место, а не пожимает плечами.
- **Annotated links (аннотированные ссылки).** Суть: каждая ссылка несёт два блока — «что лежит по ссылке» и (важнее) «зачем это читать». Аналогия: не просто адрес дома, а табличка «здесь архив договоров, зайди если ищешь условия аренды». Почему важно: агент решает, **стоит ли** открывать документ, не тратя контекст на его выкачивание вслепую.
- **Progressive disclosure.** Суть: контекст добирается кусочками по структуре Memory Bank — сначала обзор, потом ссылки «углубиться», а не дамп всего сразу. Аналогия: читаете оглавление книги и идёте в нужную главу, а не глотаете все 500 страниц. Почему важно: root `index.md` раскрывает на 2-3 уровня вниз самое важное, и агент не тонет в файлах.
- **Index-first navigation.** Суть: каждый новый документ обязан появиться в индексе своей папки; файл без входной ссылки (orphan) — это **дефект**. Аналогия: книга, которой нет в каталоге библиотеки, как будто не существует. Почему важно: навигация остаётся целостной, а аудит может механически ловить «потерянные» документы.

#### 🔧 Иллюстративный пример: путь вопроса «почему CLI, а не REST?»

Агент получает задачу и натыкается на правило «бизнес-логика живёт в `core`, а не в CLI». Он хочет понять, **почему** так. Вот как Memory Bank его ведёт:

1. **Root index** (`.memory-bank/index.md`): в секции «Key folders (SSoT map)» аннотированная ссылка `[ADR (adr/)](adr/index.md)` с подписью «решения уровня "почему" и ключевые компромиссы». Агент понимает: за «почему» — туда.
2. **ADR index** (`adr/index.md`): определение «ADR — короткие документы WHY» и аннотированная ссылка на `0001 core-client-cli-first`.
3. **ADR-0001** (`adr/0001-core-client-cli-first.md`): минимальная структура `Decision / Why / Trade-offs`. Decision: «Сначала строим core + typed client + CLI, и только потом UI». Why: «логику можно тестировать быстрее без браузера; CLI удобен для ИИ-агента». Trade-offs: «больше дисциплины на старте, меньше регрессий в будущем».

Тот же вопрос про «как именно делать фичу по шагам» уводит агента **не** в `adr/`, а в `plans/implementation-playbook.md` (чеклист 0-9: contract → core → db → adapters → client → cli → tests → seed → docs). Один вопрос — один канонический адрес. Это и есть SSoT + WHY/WHAT/HOW в действии.

#### 📐 Диаграмма: раскладка Memory Bank и маршрут вопроса

```mermaid
flowchart TD
    ROOT["index.md<br/>root: annotated links<br/>partial disclosure 2-3 уровня"]
    ROOT --> MBB["mbb/<br/>RULES — как вести банк"]
    ROOT --> SPEC["spec/<br/>WHAT — норматив требований"]
    ROOT --> PLANS["plans/<br/>DELIVERY — эпики, фичи, playbook"]
    ROOT --> ADR["adr/<br/>WHY — решения + trade-offs"]
    ROOT --> GUIDES["guides/<br/>Diataxis consumer docs"]

    MBB --> PRIN["principles.md<br/>SSoT, annotated links,<br/>index-first, grounding-first"]
    ADR --> ADR1["0001-core-client-cli-first.md<br/>Decision / Why / Trade-offs"]
    PLANS --> PLAY["implementation-playbook.md<br/>0-9: contract..seed..docs"]

    Q["Вопрос агента:<br/>'почему CLI, а не REST?'"] -.->|"WHY"| ADR
    Q2["Вопрос агента:<br/>'как делать фичу по шагам?'"] -.->|"DELIVERY"| PLANS
```

#### 🏗️ Что построить (REBUILD_ROOT)

Минимальный skeleton (детали — [[06-part-5-technical-restoration#Глава 2: Documentation Skeleton — Memory Bank|Справочник: глава 2]] и [[07-part-6-curriculum#Module 2: Собрать documentation skeleton|Module 2]]):

1. **Каркас папок**: `mkdir` структуры `.memory-bank/spec`, `.memory-bank/plans`, `.memory-bank/adr`, `.memory-bank/mbb`, `.memory-bank/guides`. Это физическое воплощение разделения WHAT/WHY/delivery/rules.
2. **Root `index.md`** с annotated links: секции `## Quick start (for agents)`, `## Key folders (SSoT map)`, `## … important docs`. Каждая ссылка = путь + «что лежит» + «зачем читать», disclosure на 2-3 уровня вниз (реализует принцип 11).
3. **`structure.md`**: repo layout + явная карта `## Memory bank structure`, где зафиксированы роли `spec/` / `guides/` / `plans/` / `adr/` / `mbb/` / `assets/` / `evidence/`.
4. **MBB-ядро**: `mbb/index.md` (operating manual) + `mbb/principles.md` с принципами 1-17 (SSoT, progressive disclosure, WHY/WHAT/HOW, annotated links, index-first, root index usefulness, grounding-first).
5. **Индексы разделов**: `spec/index.md` (SSoT по WHAT), `plans/index.md` (SSoT по delivery), `adr/index.md` (определение «ADR — документы WHY» + перечень).
6. **ADR-0001** (`adr/0001-core-client-cli-first.md`) в формате `Decision / Why / Trade-offs` — первое зафиксированное «почему».
7. **Implementation playbook** (`plans/implementation-playbook.md`): пошаговый чеклист 0-9; и `plans/how-we-plan.md` с units Epic/Feature + Epic DoD + Feature DoD + seed-as-contract.
8. **Feature template** (`mbb/templates/feature.md`): каноничный шаблон vertical slice, чтобы по нему можно было завести новую feature page.

> **Memory Bank — живой процесс, не разовая сборка (две дисциплины, которые заводят сразу):**
>
> **ADR-триггер — когда заводить новое решение.** Создавайте `adr/NNNN-*.md` (формат `Decision / Why / Trade-offs`), как только: (а) принято архитектурное решение с trade-offs, (б) выбран один из ≥2 вариантов, или (в) меняется граница areas/контракт. Шаги: завести ADR → добавить в `adr/index.md` (иначе orphan) → связать со spec/feature, которые его реализуют. Антипаттерн: решение «зашито» в коде/коммите без ADR — через полгода никто не вспомнит «почему так» (деградирует пирамида знаний, концепция 17).
>
> **Maintenance-loop — на КАЖДОМ слайсе.** Слайс изменил код → обнови затронутые `spec`/`FT`/`operation catalog`/`screen registry` → прогони `pnpm docs:audit` (зелёный `{ "ok": true }`) → нет orphan-файлов. Без этого банк превращается в «build-once артефакт Стадии 2». Механический enforcement — `docs:audit`; дисциплину держите вы.

#### 📖 Что прочитать (REFERENCE_ROOT)

- [[06-part-5-technical-restoration#Глава 2: Documentation Skeleton — Memory Bank]] — что лежит: canonical narrative стадии (Шаги 2.1-2.5: `mkdir` структуры, root index с annotated links, базовые спеки, формат ADR, playbook 0-9); зачем: пройти сборку skeleton ДО кода по шагам и увидеть паттерн аннотированных ссылок вживую.
- [[03-part-2-philosophy#(2) Memory Bank — это structured agent memory, а не свалка заметок]] — что лежит: концептуальное ядро, таблица «Папка | Назначение | Пример» (spec=WHAT, plans=delivery, adr=WHY, mbb=Rules, guides=Diataxis) и большая mermaid-карта банка; зачем: понять, почему разделение WHAT/WHY/delivery и SSoT — это не вкусовщина.
- [[07-part-6-curriculum#Module 2: Собрать documentation skeleton]] — что лежит: hands-on чеклист воспроизведения (Goal/Build/Read/Compare/Done when/Artifacts); зачем: получить точный список артефактов и критериев готовности для REBUILD_ROOT.
- [[11-part-10-appendices#Appendix G — Связь с MBB principles]] — что лежит: таблица трассировки «17 принципов из `principles.md` → где обсуждается в walkthrough»; зачем: убедиться, что стадия стоит на реальных MBB-принципах, а не на выдуманных правилах.
- `.memory-bank/mbb/principles.md` — что лежит: нормативные принципы 1-17; зачем: первоисточник правил SSoT (1), WHY/WHAT/HOW (4), annotated links (7), index-first (6), root index usefulness (11).
- `.memory-bank/index.md` — что лежит: живой root index с annotated links; зачем: эталон того, как должна выглядеть ваша версия `index.md` после стадии.

#### ✅ Верификация (стадия пройдена, когда)

Запускаемый критерий (это и есть **evidence**, а не «вроде разложено»):

```powershell
# Given: собран skeleton .memory-bank/ (index, structure, mbb, spec, plans, adr)

# When: смотрим, что root index реально реализует annotated links + partial disclosure
Get-Content .memory-bank/index.md -TotalCount 30
# Then: видны '# Memory Bank — Index', '## Quick start (for agents)',
#       '## Key folders (SSoT map)' и первые annotated-ссылки
#       (каждая = путь + 'что лежит' + 'Читать ..., чтобы ...')

# When: гоняем инвариант структуры банка через docs-аудит
pnpm docs:audit
# Then: при зелёном банке traceability-аудит печатает
#       { "ok": true, "mode": "traceability", routePages, screenSpecs, referenceDocs, warnings }
#       при нарушении (orphan code без @docs, рассинхрон registry) — 'Memory-bank traceability audit failed:' + exit 1

# When (forward check / smoke на будущих стадиях): epic-аудит конкретного эпика.
#       На Стадии 2 (skeleton) у EP-001 ещё нет FT-evidence, поэтому это не инвариант
#       именно этой стадии, а предпросмотр того, как банк будет проверяться на Стадии 4.
node scripts/audit-memory-bank.mjs --ep EP-001
# Then: { ok:true, mode:'epic', epicId:'EP-001', ... } либо
#       'Memory-bank epic audit failed for EP-001:' + exit 1
```

Чеклист:

- [ ] По root `index.md` за минуту понятно, где **правила** (`mbb/`), где **планы** (`plans/`), где **rationale** (`adr/`), где **норматив** (`spec/`).
- [ ] Каждая ссылка в `index.md` аннотирована: есть «что лежит» **и** «зачем читать» (принцип 7).
- [ ] `adr/0001-core-client-cli-first.md` существует и держит формат `Decision / Why / Trade-offs`.
- [ ] `plans/implementation-playbook.md` содержит шаги 0-9 (contract → core → db → adapters → client → cli → tests → seed → docs).
- [ ] Нет orphan-файлов: каждый новый документ есть в индексе своей папки (принцип 6).
- [ ] `pnpm docs:audit` зелёный (`{ "ok": true }`), а не `exit 1`.

Given-when-then самопроверки: **Given** агенту дали задачу с правилом «логика в `core`, не в CLI», **When** он спрашивает «откуда это и почему?», **Then** через root `index.md` → `adr/index.md` → `adr/0001` он находит **ровно один** канонический ответ с trade-offs, не выкачивая весь репозиторий.

#### 🔗 Выравнивание с автором

Концепции 2, 5, 13, 14, 15, 16, 17, 18 из [[#Выравнивание с методологией автора (DEKSDEN, части 1–2)]]. Якоря: «MemoryBank это вспомогательный инструмент для формирования контекста агента» (часть 2, тема 3, концепция 13); «Если добавить „зачем“ — шанс, что агент прочитает файл в нужный момент, резко растёт» (часть 2, тема 3, концепция 14) — это про аннотированные ссылки, где на каждую ссылку даётся два блока: «что лежит» и (важнее) «зачем это читать»; «Это когда мы в коде содержим то, как это сделано, а выше в документации мы пишем, почему это сделано и для чего это сделано.» (часть 2, тема 5, концепция 17 — пирамида знаний); «Progressive Disclosure — добор контекста кусочками по структуре Memory Bank, без выкачивания всего сразу.» (часть 2, тема 8 / глоссарий, концепция 18 — дословная формулировка взята из глоссария, обсуждается в теме 8).

#### ⚠️ Антипаттерны стадии · 🧩 Артефакты

- Антипаттерны: смешать `spec`, `plans` и `adr` в одной папке `docs/` (ADR рядом со спеками, эпики в README) — система теряет границу WHAT/WHY/delivery, и агент не может ответить «откуда правило» ([[08-part-7-antipatterns#Антипаттерн 2: Смешать `spec`, `plans` и `adr`]]); `shared`/банк как свалка вместо осмысленных границ — антипод structured agent memory, ownership размазан ([[08-part-7-antipatterns#Антипаттерн 4: `shared` как свалка]]).
- Артефакты: `.memory-bank/index.md` (root index с annotated links, partial disclosure на 2-3 уровня); `.memory-bank/structure.md` (карта Memory bank structure); `.memory-bank/mbb/principles.md` (принципы 1-17) и `.memory-bank/mbb/index.md` (operating manual MBB); индексы `.memory-bank/spec/index.md`, `.memory-bank/plans/index.md`, `.memory-bank/adr/index.md`; `.memory-bank/adr/0001-core-client-cli-first.md` (WHY, формат `Decision/Why/Trade-offs`); `.memory-bank/plans/implementation-playbook.md` (0-9) и `.memory-bank/plans/how-we-plan.md` (Epic/Feature DoD + seed-as-contract); `.memory-bank/mbb/templates/feature.md` (каноничный feature template); `scripts/audit-memory-bank.mjs` (аудит `pnpm docs:audit`, держит структуру банка в инварианте).

---

### Стадия 3 — DB baseline и seed runner

> Исход: deterministic data foundation (Drizzle, multi-tenancy через `company_id`, soft delete, temporal history, snapshot, миграции, seed с handles).

#### 🎯 Зачем эта стадия (на пальцах)

До этого момента у вас были метод (Стадия 0), документы (Стадия 2) и каркас репозитория (Стадия 1) — но не было **места, где живут данные**. А любая фича 360-оценки в конце концов про данные: кто кого оценивает, в каком отделе, у какого руководителя, в какой кампании. Если эти данные каждый раз набивать руками, проверка фичи превращается в гадание: «у меня в базе сейчас два сотрудника или восемь? а отдел `root` уже есть?». Стадия 3 строит **детерминированный фундамент данных**: одна команда поднимает все таблицы, другая заливает заранее известный набор сущностей и возвращает **handles** — стабильные имена-ярлыки (`company.main`, `employee.hr_admin`) вместо «иди посмотри id руками».

Рамка от автора: `seed` — это «предопределённый набор данных, заливаемый в БД для теста и вычищаемый после». То есть фундамент стадии — не «накидать табличек», а сделать состояние базы **воспроизводимым и адресуемым**: запустил сценарий → получил ровно те же UUID и ровно те же handles, что и вчера, что и у коллеги, что и в CI. Без этого приёмочный тест не на что опереть.

#### 🧠 Ключевые концепции простыми словами

- **Multi-tenancy через `company_id`.** Суть: каждая бизнес-таблица несёт `company_id` с FK на `companies(id)` (`onDelete: cascade`) — в схеме базы это встречается 20 раз. Аналогия: в общежитии у каждой вещи бирка с номером комнаты, и по умолчанию ты видишь только свою комнату. Почему важно: изоляция тенантов начинается в **структуре данных** (PK/FK), а не в middleware; RLS поверх — лишь второй рубеж (концепция 24).
- **Soft delete.** Суть: вместо физического `DELETE` ставим `deleted_at` (есть только на `companies`, `employees`, `departments` — 3 таблицы). Аналогия: документ не сжигают, а кладут в архив с датой «убрано». Почему важно: snapshot кампании может ссылаться на сотрудника, которого «удалили» — историческая целостность не рвётся.
- **Temporal history.** Суть: переходы между отделами/руководителями/позициями хранятся в history-таблицах со `start_at NOT NULL` и nullable `end_at`; `end_at IS NULL` значит «действует сейчас». Аналогия: трудовая книжка — не одна строка «где сейчас», а лента записей с датами. Почему важно: позволяет спросить базу «а где работал человек **на момент** старта кампании», а не «где он сейчас».
- **Snapshot (заморозка org state).** Суть: на старте кампании в `campaign_employee_snapshots` морозится копия оргсостояния каждого участника, разрешённая через temporal-историю по правилу `start_at <= t AND (end_at IS NULL OR end_at > t)`. Аналогия: на входе на экзамен делают фото состава группы — потом кто-то переведётся, но оценивают по фото со старта. Почему важно: это **доменное** решение о справедливости, а не техническая оптимизация (концепция 24).
- **Seed с handles.** Суть: `runSeedScenario` под advisory-lock делает `TRUNCATE ... RESTART IDENTITY CASCADE`, в одной транзакции вставляет сценарий с **детерминированными литеральными UUID** и возвращает `{ scenario, handles }`. Аналогия: не «найди в зале человека по паспорту», а «возьми того, у кого на груди бейдж `employee.hr_admin`». Почему важно: тест адресует сущности по стабильным именам, а не по «вчерашним» id (концепции 6, 7, 11).

#### 🔧 Иллюстративный пример: сценарий `S2_org_basic` и адресация по handles

1. **Поднимаем схему.** `pnpm db:migrate` применяет нумерованные SQL-миграции `0000..0013` по порядку. `0000_ft0002_baseline.sql` создаёт `pgcrypto`, затем `companies`, `employees`, `departments` (с `deleted_at`) и history-таблицы (`start_at`/`end_at`).
2. **Заливаем сценарий.** `runSeedScenario({ scenario: "S2_org_basic" })` берёт `pg_advisory_xact_lock(360360360)`, чистит базу и в одной транзакции вставляет фиксированную оргструктуру: `companyMain` всегда `10000000-0000-4000-8000-000000000001`, `seededAt` всегда `2026-01-01T09:00:00.000Z`. CLI-вход — `pnpm seed seed --scenario S2_org_basic --json` (двойной `seed`: первый — npm-скрипт-обёртка над CLI, второй — подкоманда commander; либо напрямую `pnpm --filter @feedback-360/cli cli seed --scenario S2_org_basic --json`).
3. **Получаем handles.** Возвращается карта стабильных ключей → UUID: `company.main`, `department.root`, `employee.ceo`, `employee.head_a`, `employee.hr_admin`, `user.hr_admin`.
4. **Проверяем счётчики.** Состояние воспроизводимо: `companies = 1`, `employees = 8`, `departments = 3` — ровно это и утверждает `ft-0003-seed-runner.test.ts`.
5. **Адресуем дальше.** Следующая стадия зовёт `cli campaign-start --id <handle department.root>` — и никто не «лезет в базу руками» за id.

Тот же контракт даёт `pnpm seed seed --scenario S4_campaign_draft --variant no_participants --json`: handles включают `campaign.main` и `department.a`, а база при этом содержит `campaigns = 1`, `campaign_participants = 0`.

#### 📐 Диаграмма: от temporal-истории к замороженному snapshot

```mermaid
flowchart LR
    MIG["pnpm db:migrate<br/>SQL 0000..0013 (fallback)"] --> SCHEMA["Схема<br/>company_id · deleted_at · start_at/end_at"]
    SCHEMA --> SEED["runSeedScenario()<br/>advisory lock + truncate + tx"]
    SEED --> HND["handles<br/>company.main · employee.hr_admin"]
    SCHEMA --> HIST["history-таблицы<br/>as-of: start_at до t · end_at пусто/после t"]
    HIST -->|"as-of по campaign.startAt"| SNAP["campaign_employee_snapshots<br/>заморозка org state"]
    HND --> SNAP
    HEALTH["pnpm db:health<br/>select 1 as ok"] -.->|"ровно 1 строка"| SCHEMA
```

#### 🏗️ Что построить (REBUILD_ROOT)

Снизу вверх (детали — [[06-part-5-technical-restoration#Глава 3: Database Layer — Drizzle ORM]] и [[07-part-6-curriculum#Module 3: DB baseline и seed runner]]):

0. **Окружение БД — предпосылка всех ✅ ниже.** Без поднятого Postgres и заданного URL команды `db:migrate`/`db:health`/`seed`/`test:db` стартуют с недостижимого Given. Подними Postgres и создай `.env` (`copy-verbatim` из `.env.example`):

```bash
# Postgres через Docker (или используй уже установленный локальный Postgres с БД feedback360)
docker run --name fb360-pg -e POSTGRES_PASSWORD=postgres -e POSTGRES_DB=feedback360 -p 5432:5432 -d postgres:15
```

```bash
# .env в корне репозитория
DATABASE_URL=postgres://postgres:postgres@localhost:5432/feedback360
# SUPABASE_DB_POOLER_URL=...   # облако Supabase (Session pooler) — приоритетнее DATABASE_URL
```

Рантайм читает `.env` через `getDatabaseUrl()` (`packages/db/src/connection-string.ts`), иначе бросает «Database URL is not set»; резолюция URL — `SUPABASE_DB_POOLER_URL ?? DATABASE_URL`. `createPool()` (`db.ts`) по умолчанию `serviceRole: true`, поэтому `migrate`/`seed`/`health` проходят FORCE-RLS из миграции `0002` под service-role-контекстом (`-c app.is_service_role=on`). В `drizzle.config.ts` тот же `??` плюс локальный дефолт `postgres://postgres:postgres@localhost:5432/feedback360` для drizzle-kit.

1. **Пакет `packages/db`** и DB layer: `drizzle.config.ts` (`schema ./src/schema/index.ts`, `out ./drizzle`, dialect `postgresql`; URL = `SUPABASE_DB_POOLER_URL` `??` `DATABASE_URL` `??` локальный postgres).
2. **Schema** (`src/schema/tables.ts`): multi-tenancy через `company_id` ([[04-part-3-architecture#3.12. Multi-tenancy через company_id]]), `deleted_at` soft delete на `companies`/`employees`/`departments` ([[04-part-3-architecture#3.13. Soft delete pattern]]), history-таблицы `start_at`/`end_at` ([[04-part-3-architecture#3.14. Temporal history pattern]]), таблица `campaign_employee_snapshots` ([[04-part-3-architecture#3.15. Snapshot pattern]]).
3. **Миграции** в `drizzle/` (`0000..0013`): `0000_ft0002_baseline.sql` (`create table if not exists`, soft delete, temporal), `0002_ft0023_rls_deny_by_default.sql` (схема `app`, функции `app.is_service_role()`/`app.has_company_access(uuid)`, `ENABLE` + `FORCE` RLS — deny-by-default), `0003_ft0032_campaign_snapshots.sql`.
4. **Migration/health runtime** (`src/migrations.ts`): `applyMigrations()` — поскольку `drizzle/meta/_journal.json` отсутствует, идёт **SQL-fallback** (читает все `.sql` и сортирует их лексикографически (`.sort()`), что для нулёво-дополненных префиксов `0000..0013` совпадает с числовым порядком, исполняет через `db.execute(sql.raw(...))`); `runHealthCheck()` гоняет `select 1 as ok` и проверяет ровно одну строку.
5. **Pool/DB factory** (`src/db.ts`): `createPool(context)` ставит per-connection GUC `-c app.is_service_role=on/off` и опционально `-c app.current_user_id=<userId>` — эти GUC питают RLS-функции из миграции `0002`.
6. **CLI-обёртки**: `src/scripts/migrate.ts` (`db:migrate` → `Migrations applied successfully.`) и `src/scripts/health.ts` (`db:health` → `DB health-check passed.`), оба ставят `process.exitCode = 1` при падении.
7. **Seed runner** (`src/seeds.ts`): `runSeedScenario` с advisory-lock `360360360`, `TRUNCATE ... RESTART IDENTITY CASCADE`, вставкой в одной `db.transaction`, детерминированными UUID и стабильными handles; snapshot-мост `src/snapshots.ts`. CLI-вход — команда `seed` в `packages/cli/src/legacy.ts` (`--scenario`, `--variant`, `--json`).

> **Литеральные константы детерминизма** (`copy-verbatim` из `REFERENCE_ROOT` — иначе инвариант «те же UUID» недостижим). В начале `seeds.ts` зафиксированы `const seededAt = new Date("2026-01-01T09:00:00.000Z")` и карта `const ids = { companyMain: "10000000-0000-4000-8000-000000000001", companyA: "10000000-…-010", membershipHrAdmin: "11000000-…-001", employeeHrAdmin: "12000000-…-001", … }` с конвенцией префиксов (`1x…` — компании, `11x…` — membership, `12x…` — employee). UUID в строках берутся из `ids`, а не из `uuid().defaultRandom()`, поэтому каждый прогон даёт **те же** значения, что и в ✅. Свой сценарий → задайте собственные фиксированные `const` и сверяйтесь со **стабильностью** вывода, а не с цифрами документа (см. [[feedback-360-master-rebuild-walkthrough#Контракт детерминизма ✅-блоков]]).

#### 📖 Что прочитать (REFERENCE_ROOT)

- [[06-part-5-technical-restoration#Глава 3: Database Layer — Drizzle ORM]] — что лежит: «Почему Drizzle», шаги 3.1–3.6 (package setup, multi-tenancy, soft delete, temporal, uniqueness, migrations+health), Checkpoint; зачем: основная опора всей стадии и обоснование выбора Drizzle.
- [[04-part-3-architecture#3.12. Multi-tenancy через company_id]] — что лежит: `companies` pgTable + Aha «`company_id` — это структурный дизайн данных, а не RLS alone»; зачем: понять первый рубеж изоляции тенантов (концепция 24).
- [[04-part-3-architecture#3.13. Soft delete pattern]] — что лежит: `isActive` + `deletedAt` вместо физического удаления; зачем: обосновать `deleted_at` в baseline `0000`.
- [[04-part-3-architecture#3.14. Temporal history pattern]] — что лежит: `employeeDepartmentHistory` со `start_at`/`end_at`, правило «`end_at = null` = текущая»; зачем: понять as-of-резолюцию, на которой стоит snapshot.
- [[04-part-3-architecture#3.15. Snapshot pattern]] — что лежит: `campaignEmployeeSnapshots` на старте кампании + Aha «snapshot — доменное решение о справедливости»; зачем: обосновать `0003_ft0032` и `snapshots.ts`.
- [[07-part-6-curriculum#Module 3: DB baseline и seed runner]] — что лежит: Goal/Build/Read/Done-when/Artifacts стадии (включая FT-0002, FT-0003 и git range `508ba7e..9eaddee`); зачем: actionable-спека приёмки и артефактов.

#### ✅ Верификация (стадия пройдена, когда)

Запускаемый критерий (это и есть **evidence**, а не «база вроде поднялась»):

```bash
# Given: собран packages/db и задан DATABASE_URL

# When: поднимаем схему и проверяем соединение
pnpm db:migrate          # применяет SQL 0000..0013 (fallback, идемпотентно)
# Then: "Migrations applied successfully."

pnpm db:health           # select 1 as ok — ровно одна строка
# Then: "DB health-check passed." (exit 0)

# When: заливаем детерминированный сценарий (двойной seed: npm-обёртка + подкоманда commander)
pnpm seed seed --scenario S1_company_min --json
# Then: { "scenario": "S1_company_min", "handles": { "company.main": "10000000-0000-4000-8000-000000000001", … } }

pnpm seed seed --scenario S4_campaign_draft --variant no_participants --json
# Then: handles содержат campaign.main и department.a; в базе campaigns=1, campaign_participants=0

# When: гоняем приёмочные тесты
pnpm --filter @feedback-360/db test:db
pnpm --filter @feedback-360/db exec vitest run src/migrations/ft-0003-seed-runner.test.ts
```

Чеклист:

- [ ] `pnpm db:migrate` печатает `Migrations applied successfully.` и **повторный** запуск не падает (идемпотентность: `create table/extension if not exists`).
- [ ] `pnpm db:health` печатает `DB health-check passed.`; при отсутствии/неверном `DATABASE_URL` — `DB health-check failed.` и exit `1`.
- [ ] `pnpm seed seed --scenario S1_company_min --json` отдаёт стабильные handles `company.main`, `employee.hr_admin`, `user.hr_admin` с фиксированными UUID.
- [ ] `ft-0003-seed-runner.test.ts` зелёный: `S2_org_basic` даёт `companies=1`, `employees=8`, `departments=3`; `S9` completed `submitted=4`.
- [ ] Evidence-блок записан: дата + команды + вывод `db:migrate`/`db:health` + пример JSON seed (Module 3 Artifacts).

Given-when-then: **Given** пустая база и заданный `DATABASE_URL`, **When** вы запускаете `pnpm db:migrate && pnpm db:health && pnpm seed seed --scenario S2_org_basic --json`, **Then** схема поднята, health зелёный, а в ответе — те же handles и те же UUID, что и при любом другом запуске (воспроизводимое состояние).

#### 🔗 Выравнивание с автором

Концепции 5, 6, 7, 11, 24 из [[#Выравнивание с методологией автора (DEKSDEN, части 1–2)]]: спецификация=шаги+grounding и вертикальные слайсы под приёмочный сценарий (5, 6), сам acceptance given-when-then (7), изоляция окружений и deterministic-данные (11), доменная multi-tenancy/snapshot как ядро feedback-360 (24). Якоря: «**seed** — предопределённый набор данных, заливаемый в БД для теста и вычищаемый после» (часть 1, глоссарий, строка 226); «при тестировании он заливает **seed** в БД и чистит после, но может ошибиться и снести не только результаты теста» (часть 1, тема 9 «Изоляция окружений local/beta/prod», строка 118) — поэтому seed-runner держит advisory-lock и блокирует `seed.run` в beta **только при активной (непросроченной) XE-блокировке** беты (`getXeLock('beta')`), а не на каждом `APP_ENV === 'beta'`.

#### ⚠️ Антипаттерны стадии · 🧩 Артефакты

- Антипаттерны: acceptance без seed/handles — «посмотреть id в базе руками» вместо стабильных handles ([[08-part-7-antipatterns#Антипаттерн 5: Acceptance без seed/handles]]); считать стадию готовой без evidence — без записанного вывода `db:migrate`/`db:health` и примера JSON seed ([[08-part-7-antipatterns#Антипаттерн 7: Считать feature готовой без evidence]]).
- Артефакты: текстовый вывод `pnpm db:migrate` (`Migrations applied successfully.`) и `pnpm db:health` (`DB health-check passed.`); пример JSON результата seed-runner с handles; нумерованные миграции `0000..0013` (`packages/db/drizzle/`); зелёные `ft-0002-migrations-health.test.ts` и `ft-0003-seed-runner.test.ts`.

---

### Стадия 4 — Deterministic delivery path: contract + core + client + CLI

> Исход: первая операция проходит весь путь от CLI до core и возвращает **одинаковый** `OperationResult<T>`. Это «позвоночник» системы — дальше каждая фича добавляется по одному шаблону.

#### 🎯 Зачем эта стадия (на пальцах)

До этой стадии у вас были «кости» (monorepo, docs, БД), но не было **спинного мозга** — единого нерва, по которому идут все команды. Сейчас вы его прокладываете. Идея автора: вместо россыпи REST-контроллеров (каждый со своим форматом ошибки и своим способом тестирования) — **одна функция-маршрутизатор** `dispatchOperation()` и **один формат ответа** `OperationResult<T>`. Почему это меняет всё для агента: «одна операция = один handler = один parse-step», нет распределённого знания, и клиентскую логику можно гонять **текстом через CLI**, не поднимая браузер. Агент проверяет ценность детерминированно и быстро.

#### 🧠 Ключевые концепции простыми словами

- **`OperationResult<T>` (discriminated union).** Суть: ответ — это либо `{ ok: true; data }`, либо `{ ok: false; error }`. Аналогия: посылка приходит либо с товаром, либо с квитанцией «почему не доставили» — но всегда в одинаковой коробке. Почему важно: TypeScript **не даст** прочитать `data`, не проверив `ok` — ошибку нельзя «забыть».
- **Dispatcher.** Суть: `dispatchOperation()` принимает `{ operation, input, context }`, валидирует, находит handler по имени, вызывает. Аналогия: одно окно регистратуры, которое маршрутизирует вас к нужному кабинету. Почему важно: транспортная независимость — CLI и web зовут одно и то же.
- **Transport abstraction (in-proc vs HTTP).** Суть: `OperationTransport` — это «как доставить запрос до core». In-proc вызывает функцию напрямую (тесты, CLI); HTTP делает POST (браузер). Аналогия: одно письмо можно отнести лично или отправить почтой — содержимое не меняется. Почему важно: тесты гоняют in-proc детерминированно, прод — HTTP, бизнес-логика **не дублируется**.
- **CLI-first dual output.** Суть: CLI — тонкая оболочка над клиентом; `--json` даёт машиночитаемый ответ. Аналогия: у прибора есть и кнопки для людей, и розетка-API для робота. Почему важно: агент всегда использует `--json` и проверяет результат программно.

#### 🔧 Иллюстративный пример: операция `system.ping` от CLI до core

1. **Контракт** (`api-contract`): тип `OperationResult<T>`, helpers `okResult`/`errorResult`, парсеры `parseSystemPingInput`/`parseSystemPingOutput`.
2. **Core handler** (`runSystemPing`): `parse вход → okResult({ pong: "ok", timestamp })`.
3. **Dispatcher**: `dispatchOperation({ operation: "system.ping" })` находит `runSystemPing`.
4. **Client**: `client.systemPing()` через in-proc transport.
5. **CLI**: `ping --json`.

Результат — стабильный `OperationResult`:

```json
{ "ok": true, "data": { "pong": "ok" } }
```

Тот же `OperationResult` вы получите и через HTTP transport в браузере — отличается только доставка, не содержимое.

> **Сверка с кодом (REFERENCE_ROOT).** В оригинальном репозитории `system.ping` доступен как **клиентский метод** `client.systemPing()` (in-proc), а **отдельной CLI-команды `ping` в `packages/cli` нет**. CLI запускается без `bin`, через `tsx src/index.ts` (root-обёртка `pnpm seed -- <args>`). Здесь вы заводите тривиальную `ping`-команду в **своём** `REBUILD_ROOT` как первый сквозной слайс; форма `pong: "ok"` совпадает с реальным `runSystemPing`.

#### 📐 Диаграмма: путь запроса (deterministic delivery path)

```mermaid
flowchart LR
    CLI["packages/cli<br/>ping --json"] --> CLIENT["Feedback360Client<br/>client.systemPing()"]
    CLIENT --> RT["ClientRuntime<br/>invokeOperation()"]
    RT --> TR["OperationTransport<br/>in-proc"]
    TR --> CORE["dispatchOperation<br/>core router"]
    CORE --> H["runSystemPing<br/>handler"]
    H -->|"OperationResult"| CORE
    CORE -->|"{ ok, data }"| TR
    TR --> RT2["ClientRuntime"]
    RT2 --> OUT["stdout JSON<br/>{ ok:true, data:{ pong:ok } }"]
```

#### 🏗️ Что построить (REBUILD_ROOT)

Снизу вверх (детали — [[04-part-3-architecture#3.4. Dispatcher pattern|Справочник A §3.4–3.10]] и [[06-part-5-technical-restoration#Глава 4: Typed Contract — api-contract|Справочник: Часть V главы 4–7]]):

1. `packages/api-contract`: `OperationResult<T>`, `OperationError` + typed коды, `okResult`/`errorResult`, `DispatchOperationInput`, `KnownOperation`, parse-функции.
2. `packages/core`: `operationHandlers` map + `dispatchOperation()` (parse → known? → handler) + первый handler `runSystemPing`.
3. `packages/client`: `OperationTransport` interface, `createInprocTransport`, `ClientRuntime.invokeOperation()`, `Feedback360Client` с методом `systemPing()`.
4. `packages/cli`: `commander`-команда `ping` с `--json`, использующая `createInprocClient()`.

**Wiring, без которого команды ✅ не резолвятся** (`copy-verbatim` — в оригинале **нет `bin`**, точка входа `src/index.ts`):

```jsonc
// packages/cli/package.json
"exports": "./src/index.ts",
"scripts": { "cli": "tsx src/index.ts" }

// root package.json → scripts
"db:migrate": "pnpm --filter @feedback-360/db db:migrate",
"db:health":  "pnpm --filter @feedback-360/db db:health",
"seed":       "pnpm --filter @feedback-360/cli cli --"
```

Тогда `pnpm seed seed --scenario S1_company_min --json` разворачивается в `tsx src/index.ts seed --scenario …` (первый `seed` — корневая обёртка, второй — `commander`-подкоманда), а `pnpm --filter @feedback-360/cli exec tsx src/index.ts -- ping --json` бьёт ту же точку входа напрямую.

**Реестр операций** (`packages/core/src/index.ts`) — то, что связывает имя операции из CLI/клиента с core-хендлером:

```ts
const operationHandlers: Partial<Record<KnownOperation, OperationHandler>> = {
  "system.ping": (request) => runSystemPing(request.input),  // sync: обёртка вытаскивает input
  "membership.list": runMembershipList,                       // async: handler берёт request целиком
};
// dispatchOperation: parse → operationHandlers[op] → handler (нет записи в map → unknown_operation)
```

#### 📖 Что прочитать (REFERENCE_ROOT)

- [[04-part-3-architecture#3.4. Dispatcher pattern]] — что лежит: код роутера; зачем: понять единую точку входа.
- [[04-part-3-architecture#3.5. OperationResult — центральный тип]] — что лежит: центральный тип; зачем: type-driven error handling.
- [[04-part-3-architecture#3.10. Transport abstraction]] — что лежит: in-proc/HTTP; зачем: почему логика не дублируется.
- `.memory-bank/adr/0001-core-client-cli-first.md` — зачем: решение «почему так, а не REST».

#### ✅ Верификация (стадия пройдена, когда)

Запускаемый критерий (это и есть **evidence**, а не «вроде работает»):

```bash
# Given: собран каркас contract + core + client + CLI
pnpm install
pnpm typecheck                                   # tsc --noEmit — зелёный
pnpm lint                                        # Biome — зелёный

# When: вызываем первую операцию через CLI в JSON-режиме
pnpm --filter @feedback-360/cli exec tsx src/index.ts -- ping --json

# Then: получаем ровно этот OperationResult
# { "ok": true, "data": { "pong": "ok" } }
```

Чеклист:

- [ ] `ping --json` печатает `{ "ok": true, "data": { "pong": "ok" } }`.
- [ ] Неизвестная операция возвращает `{ ok: false, error: { code: "not_found", … } }`, а **не** throw.
- [ ] Тот же handler вызывается через in-proc transport в unit-тесте без mock-ов HTTP.
- [ ] Evidence-блок записан: дата + команды + вывод.

Given-when-then: **Given** собран каркас contract→core→client→CLI, **When** `ping --json`, **Then** `{ ok: true, data: { pong: "ok" } }` — один и тот же `OperationResult` приходит и через in-proc CLI, и через HTTP-transport.

#### 🔗 Выравнивание с автором

Концепции 1, 5, 6, 7, 12 из [[#Выравнивание с методологией автора (DEKSDEN, части 1–2)]]. Якоря: «когда модель работает с CLI-интерфейсом, она делает это легко, непринуждённо» (часть 1, тема 12); «вершиной всей проверки является acceptance test, который тестирует вертикальный слайс» (часть 1, тема 7).

#### ⚠️ Антипаттерны стадии · 🧩 Артефакты

- Антипаттерны: бизнес-логика в CLI/route handler вместо core ([[08-part-7-antipatterns#Антипаттерн 3: Бизнес-логика в route handlers, CLI или UI]]); acceptance без seed/handles ([[08-part-7-antipatterns#Антипаттерн 5: Acceptance без seed/handles]]).
- Артефакты: рабочий `ping --json`; первый зелёный acceptance через CLI; evidence-блок в FT-документе.

---

### Стадия 5 — Первые domain slices

> Исход: первая реальная domain behavior (identity-tenancy, org, models) — полный контракт→core→db→client→cli→test→docs на узких слайсах.

#### 🎯 Зачем эта стадия (на пальцах)

В Стадии 4 вы проложили **спинной мозг** — одну операцию `system.ping`, которая прошла от CLI до core и вернула стабильный `OperationResult<T>`. Но `ping` ничего не знает про бизнес: это пустой нерв без мышцы. Сейчас вы первый раз заставляете эту вертикаль **делать что-то осмысленное** — выдать список компаний пользователя, завести сотрудника, опубликовать версию модели компетенций. Ключевой сдвиг мышления автора: добавлять функциональность **не по слоям, а вертикальными слайсами**. Не «сегодня пишем все контракты, завтра весь core, послезавтра всю БД» — а «один слайс целиком: contract → core → db → client → cli → test → docs, и он реально доставляет одну единицу ценности».

Рамка от автора: вертикальный слайс — это **единица приёмочного теста**. Если фичу нельзя проверить одним сквозным acceptance-сценарием на seed-данных, она «не доставлена», сколько бы кода вы ни написали. Поэтому первые три домена — `identity-tenancy`, `org`, `models` — берутся узкими: не весь HR-домен сразу, а ровно столько, чтобы поймать **ритм** доставки слайса и повторять его дальше механически.

#### 🧠 Ключевые концепции простыми словами

- **Vertical slice (вертикальный слайс).** Суть: одна единица ценности, протянутая через **все** слои сразу, а не «кусок слоя». Аналогия: не «залить фундаменты для всего квартала», а «построить один дом целиком — с крышей, дверью и водопроводом». Почему важно: только целый слайс можно показать пользователю и накрыть одним acceptance-тестом; полуслой не доставляет ничего.
- **Один слой — одна ответственность.** Суть: `contract` — только Zod-parse и типы (нуль логики); `core` — auth-check → parseInput → бизнес/DB → parseOutput; `client` — локальная валидация input + делегирование в `runtime`; `db` — SQL через Drizzle. Аналогия: на конвейере у каждого станка ровно одна операция — кто-то только красит, кто-то только сверлит. Почему важно: смешали ответственности (бизнес-логика в CLI, SQL в core) — архитектура «поплыла», и агент перестаёт находить, где что чинить.
- **RBAC через явные роли.** Суть: каждый handler начинается с проверки роли — мутации требуют `'hr_admin'`, чтение допускает `'hr_admin'`/`'hr_reader'`; иначе `forbidden`. Аналогия: пропуск в здание проверяют **на входе**, а не после того, как вы уже сели за чужой стол. Почему важно: auth-check стоит **первым** в handler-е — невозможно «забыть» проверку и случайно пустить `employee` мутировать оргструктуру.
- **Контекст компании (tenancy).** Суть: `ensureContextCompany` гарантирует, что операция выполняется **в рамках одной компании**; `runMembershipList` без `context.userId` отвечает `unauthenticated`. Аналогия: у сотрудника может быть бейдж сразу нескольких офисов, но внутри он всегда стоит в **одной** двери. Почему важно: мультитенантность означает, что данные одной компании не должны протечь в другую — контекст это и охраняет.
- **Acceptance на seed + handles.** Суть: слайс «доставлен» только когда проходит given-when-then сценарий на **детерминированных** seed-данных со стабильными UUID-handles (`company.main`, `employee.hr_admin`), а не на случайных строках из БД. Аналогия: тест-драйв на стандартной трассе с разметкой, а не «прокатился где попало». Почему важно: воспроизводимость — без неё acceptance не повторить, а значит, нечем доказать, что фича работает.

#### 🔧 Иллюстративный пример: слайс `membership.list` на seed `S1_company_min`

Возьмём первый домен `identity-tenancy` и проследим **одну** операцию — «покажи мне компании, в которых я состою» — целиком через слайс на детерминированном seed.

1. **Seed.** `pnpm seed seed --scenario S1_company_min --json` создаёт стабильные handles: `company.main`, `employee.hr_admin`, `user.hr_admin` с фиксированными UUID (например `company.main = 10000000-0000-4000-8000-000000000001`). Каждый запуск — **идентичный** результат.
2. **Contract** (`packages/api-contract/src/identity-tenancy.ts`): `parseMembershipListInput`/`Output` + типы `MembershipList`/`MembershipListItem`. Слой — только Zod-parse, без логики.
3. **Core** (`runMembershipList`): auth-check `context.userId` (иначе `unauthenticated`) → `parseMembershipListInput` → `db.listMemberships({ userId })` → `parseMembershipListOutput` → `okResult`.
4. **DB** (`packages/db/src/memberships.ts`): `SELECT companyMemberships INNER JOIN companies WHERE userId AND companies.isActive AND deletedAt IS NULL ORDER BY companies.name`. Возвращает `{ items: [{ companyId, companyName, role }] }`.
5. **Client → CLI**: `createIdentityTenancyClientMethods(runtime).membershipList()` делегирует в `runtime.invokeOperation({ operation: 'membership.list' })`; CLI печатает `--json`.

Сквозной результат — стабильный `OperationResult`:

```json
{ "ok": true, "data": { "items": [ { "companyId": "...", "companyName": "...", "role": "hr_admin" } ] } }
```

Важная **doc-vs-code сверка**: реальный выходной ключ db-слоя — `items`. В справочнике (раздел «Шаг 8.1: Contract») для иллюстрации показан ключ `memberships` — это расхождение «документ против кода»; ваш слайс должен следовать коду (`items`). И ещё: `membership.list` в оригинале достижим через клиентский метод `client.membershipList()` — отдельной CLI-команды `membership list` в `packages/cli` пока нет (в каталоге команд помечена `(future)`), поэтому cli-слой этого слайса вы заводите в своём `REBUILD_ROOT`.

#### 📐 Диаграмма: три первых слайса по одному шаблону

```mermaid
flowchart LR
    subgraph SLICE["Один слайс = один шаблон доставки"]
        CT["contract<br/>Zod-parse + типы"] --> CORE["core handler<br/>auth → parse → db → parse"]
        CORE --> DB["db<br/>SQL через Drizzle"]
        CORE --> CL["client<br/>validate + delegate"]
        CL --> CLI["cli<br/>--json"]
        CORE --> TST["test<br/>acceptance: seed + handles"]
        CORE --> DOC["docs<br/>FT page"]
    end
    IDN["identity-tenancy<br/>4 операции"] --> SLICE
    ORG["org<br/>6 операций"] --> SLICE
    MOD["models<br/>6 операций"] --> SLICE
```

#### 🏗️ Что построить (REBUILD_ROOT)

Берёте **1–2 слайса** (не весь HR-домен сразу) и для каждого проходите весь шаблон. Детали доменов — [[06-part-5-technical-restoration#Глава 8: Первый Vertical Slice — Identity & Tenancy]] и [[06-part-5-technical-restoration#Глава 9: Domain Modeling — Org, Models, Campaigns]]; ритм доставки — [[07-part-6-curriculum#Module 5: Первые domain slices]].

1. **`identity-tenancy` (4 операции).** Contract `packages/api-contract/src/identity-tenancy.ts` — barrel-реэкспорт parse-функций и типов (`MembershipList`, `IdentityProvisionAccess`, `ClientSetActiveCompany`, `CompanyUpdateProfile`, `SystemPing`). Core `packages/core/src/features/identity-tenancy.ts` — handler-ы `runSystemPing`, `runCompanyUpdateProfile` (требует `role==='hr_admin'`), `runMembershipList` (требует `context.userId`), `runIdentityProvisionAccess` (`hr_admin` + `ensureContextCompany`). DB `packages/db/src/memberships.ts` (`listMemberships`) и `packages/db/src/identity.ts` (`provisionIdentityAccess` внутри `db.transaction`). Client `packages/client/src/features/identity-tenancy.ts` — `createIdentityTenancyClientMethods(runtime)`.
2. **`org` (6 операций).** Contract `packages/api-contract/src/org.ts` (Department List/Upsert, EmployeeDirectoryList, EmployeeProfileGet, EmployeeListActive, EmployeeUpsert, OrgDepartmentMove, OrgManagerSet). Core `packages/core/src/features/org.ts` — RBAC через `hasRole(request, [...])`: мутации `'hr_admin'`, чтение `'hr_admin'`/`'hr_reader'`, все через `ensureContextCompany`. DB: `packages/db/src/org.ts` содержит 4 production-операции (`moveEmployeeDepartment`, `setEmployeeManager`, `listDepartments`, `upsertDepartment` — с temporal-history полями `effectiveAt`/`changed`/`previousDepartmentId`) — отдельно от двух debug-хелперов `getDepartmentHistoryForEmployeeDebug`/`getManagerHistoryForEmployeeDebug`; функции `upsertEmployee`, `listActiveEmployees`, `listEmployeeDirectory`, `getEmployeeProfile` физически живут в `packages/db/src/employees.ts` — core импортирует всё единым `from @feedback-360/db` (barrel), поэтому распределение прозрачно.
3. **`models` (6 операций).** Contract `packages/api-contract/src/models.ts` (ModelVersion Create/CloneDraft/Get/List/Publish/UpsertDraft + типы `ModelVersionStatus`, `ModelCompetencyInput`, `ModelGroupInput`, `ModelIndicatorInput`, `ModelLevelInput`). Core `packages/core/src/features/models.ts` — state machine `draft → published`; ошибка публикации маппится в код `'invalid_transition'`.
4. **Для каждого слайса — test + docs.** Acceptance-сценарий given-when-then на seed `S1_company_min`/`S2_org_basic`; FT page по шаблону `.memory-bank/mbb/templates/feature.md`; запись в `LEARNING_NOTES.md`.

#### 📖 Что прочитать (REFERENCE_ROOT)

- [[06-part-5-technical-restoration#Глава 8: Первый Vertical Slice — Identity & Tenancy]] — что лежит: опорная вертикаль `membership.list` через CLI→Client→Runtime→Transport→Core→Handler→DB→PG, таблица «один слой = одна ответственность»; зачем: эталон одного запроса через все слои, который слайс повторяет на каждую операцию.
- [[06-part-5-technical-restoration#Глава 9: Domain Modeling — Org, Models, Campaigns]] — что лежит: org с temporal history, competency models indicators(1–5+NA)/levels(1–4+UNSURE), версионирование `draft → published`; зачем: доменная опора, объясняющая поля в `core`/`db` файлах org- и models-слайсов.
- [[04-part-3-architecture#3.19. 10 canonical feature areas]] — что лежит: дерево `packages/core/src/features/` из 10 файлов + каталог 53 операций; зачем: канонический счёт первых слайсов — Identity&Tenancy 4, Organization 6, Models 6.
- [[07-part-6-curriculum#Module 5: Первые domain slices]] — что лежит: Goal/Build/Read/Done-when для slice delivery; зачем: прямое практическое задание стадии — освоить ритм, не повторяя весь HR-домен сразу.
- `.memory-bank/mbb/templates/feature.md` — что лежит: шаблон FT page; зачем: единый формат feature-документа для обоих слайсов.

#### ✅ Верификация (стадия пройдена, когда)

Запускаемый критерий (это и есть **evidence**, а не «вроде работает»):

```bash
# Given: схема поднята и seed загружен детерминированно
pnpm db:migrate && pnpm db:health         # схема + health зелёный
pnpm seed seed --scenario S1_company_min --json   # стабильные handles (company.main, employee.hr_admin)

# When: гоняем первый слайс через всю вертикаль
pnpm seed membership list --json      # от лица hrAdmin

# Then: стабильный OperationResult (реальный db-ключ строк — items)
# { "ok": true, "data": { "items": [ ... ] } }

# И второй слайс на org-сценарии:
pnpm seed seed --scenario S2_org_basic --json   # companies=1, employees=8, departments=3
pnpm --filter @feedback-360/core test     # зелёные unit-тесты run*-handler-ов
```

Чеклист:

- [ ] Минимум **2 слайса** доставлены целиком (например `identity-tenancy` + `org`) одинаковым ритмом contract→core→db→client→cli→test→docs.
- [ ] `membership list --json` от `hrAdmin` на `S1_company_min` печатает `{ ok: true, data: { items: [...] } }` со **стабильными** handles, а не случайными UUID.
- [ ] `S2_org_basic` даёт инварианты `companies=1`, `employees=8`, `departments=3` — детерминированно при каждом запуске.
- [ ] Мутация без нужной роли (`employee` зовёт `runEmployeeUpsert`) возвращает `{ ok: false, error: { code: 'forbidden' } }`, а **не** throw; `runMembershipList` без `context.userId` → `unauthenticated`.
- [ ] Публикация модели из недопустимого состояния маппится в код `'invalid_transition'` (state machine `draft → published`).
- [ ] `pnpm checks` (lint + typecheck + test + test:db + web build) зелёный — quality gate перед переводом фичи в Completed.
- [ ] Для каждого слайса есть FT page и evidence-блок с датой, командами и выводом.

Given-when-then: **Given** seed `S1_company_min` загружен со стабильными handles, **When** вызываем `membership list --json` от лица `hrAdmin`, **Then** ответ `{ ok: true, data: { items: [...] } }`, и тот же сценарий воспроизводится при каждом прогоне seed.

#### 🔗 Выравнивание с автором

Концепции 6, 7, 5, 15 из [[#Выравнивание с методологией автора (DEKSDEN, части 1–2)]]. Якоря: «вершиной всей проверки является, по мне, сценарий, acceptance test, который тестирует вертикальный слайс» (часть 1, тема 7); «если не сделать вот этот acceptance test, мы с вами можем нагенерировать фичу и она не будет работать» (часть 1, тема 7).

#### ⚠️ Антипаттерны стадии · 🧩 Артефакты

- Антипаттерны: acceptance без seed/handles — проверять слайс на случайных UUID из БД вместо seed-сценария с handles ([[08-part-7-antipatterns#Антипаттерн 5: Acceptance без seed/handles]]); считать feature готовой без evidence — фиксировать completion только кодом, без прогона acceptance и записи доказательств ([[08-part-7-antipatterns#Антипаттерн 7: Считать feature готовой без evidence]]).
- Артефакты: два вертикальных слайса (например `identity-tenancy` + `org`) через все слои contract→core→db→client→cli→test→docs; две FT/feature pages по шаблону `.memory-bank/mbb/templates/feature.md`; два проходящих acceptance/test-сценария given-when-then на seed+handles; запись в `LEARNING_NOTES.md` о самом трудном в slice-based delivery; evidence-блок с выводом `pnpm db:migrate` / `db:health` / `pnpm seed --scenario ... --json` (команды, результаты, дата); реальные файлы слайсов `packages/{api-contract,core,client}/src/.../identity-tenancy.ts|org.ts|models.ts` + `packages/db/src/{identity,org,memberships,employees}.ts`.

---

### Стадия 6 — Policy-heavy домен

> Исход: campaigns как state machine, matrix, questionnaires, results; политика анонимности (порог 3 оценщика: строки с `n_valid < 3` скрываются), snapshot-freeze, нормализация весов, role-based views.

#### 🎯 Зачем эта стадия (на пальцах)

Представьте сейф в банке с несколькими ключами и часами на двери. До этой стадии система умела принимать команды (Стадия 4) и хранить данные (Стадия 5), но не умела говорить «нет» правильным людям в неправильный момент. Стадия 6 — это про **правила, которые нельзя обойти**: кампания живёт как автомат состояний (нельзя «закрыть» то, что не «открыто»); после первого ответа сотрудника матрица назначений **замораживается** (как бетон — пока жидкий, лепи что хочешь, застыл — только ломать); а результаты показываются только если оценщиков набралось достаточно, чтобы никого нельзя было вычислить по голосу. Это policy-heavy домен: ценность тут не в «новой кнопке», а в **гарантиях**.

Авторская рамка (перефразировка walkthrough'а, не цитата): доменное правило живёт как policy в одном месте, где его можно протестировать текстом, а не «на доверии к фронту». Автор формулирует это на примере анонимности — «политика анонимности» держится как доменное правило, а не как код в UI (часть 2), и шире: «вся клиентская логика в общей библиотеке, тестируется текстом детерминированно» (часть 1). Если правило анонимности живёт в React-компоненте, его обходят, открыв DevTools. Если в route handler — его дублируют в трёх местах и забывают синхронизировать. Поэтому threshold-проверки, freeze, нормализация весов и переходы состояний реализованы в `core`/`db` — а CLI и UI остаются тонкими «вызвать и показать». Агент проверяет каждое правило детерминированным acceptance-сценарием, а не кликами в браузере.

#### 🧠 Ключевые концепции простыми словами

- **State machine кампании.** Суть: кампания проходит фиксированный путь `draft → started → ended → processing_ai → completed` (плюс `ai_failed ↔ processing_ai` для retry), и переходы разрешены только по рёбрам этого графа. Аналогия: светофор не прыгает с красного сразу на зелёный — есть законный порядок. Почему важно: `started` разрешён **только** из `draft`, `ended` — **только** из `started`; всё остальное → `invalid_transition`. Невозможные состояния просто недостижимы.
- **Idempotent transition.** Суть: повторный `start` уже запущенной кампании не падает и не запускает её дважды — возвращает `changed: false`. Аналогия: нажать «вкл» на уже включённой лампе — она просто остаётся включённой. Почему важно: агент и сеть могут повторить команду (retry), и система не должна ломаться от дубля.
- **Snapshot-freeze (замороженное org-состояние).** Суть: на переходе в `started` система делает снимок оргструктуры (`email`/имя/отдел/менеджер/позиция на момент старта), а первый `saveDraft` любой анкеты ставит `campaign.lockedAt = now` и запрещает менять матрицу и веса. Аналогия: фотография класса в день экзамена — даже если ученик потом перевёлся, оценивают того, кто был в кадре. Почему важно: это **доменное решение справедливости** (ADR-0003), а не технический кэш — результаты и manager-доступ опираются на снимок, а не на текущий HR-справочник.
- **Anonymity threshold.** Суть: группа оценщиков (peers/subordinates) и каждая отдельная компетенция показываются только если набралось `>= 3` валидных submitted-ответов; иначе блок/строка скрывается или сливается в `Other`. Аналогия: анонимный опрос на двоих анонимным не бывает — по второму голосу вычислишь первого. Почему важно: `defaultAnonymityThreshold = 3` (ADR-0002); manager и self **всегда** показываются (manager — личный, не-анонимный фидбэк), `NA`/`UNSURE` исключаются из счёта, чтобы не раздувать эффективный `n`.
- **Weight normalization.** Суть: базовые веса `manager 40 / peers 30 / subordinates 30 / self 0`; если группа отсутствует или скрыта, оставшиеся **ренормализуются** так, чтобы дать в сумме 100; `self` исключён из weighted `overallScore` всегда. Аналогия: если один из судей не пришёл, его долю баллов распределяют между оставшимися, а не считают пустоту нулём. Почему важно: один источник правды (`buildEffectiveGroupWeights` в `db`), а не «магическая константа» в UI.

#### 🔧 Иллюстративный пример: кампания `cmp1` от старта до результатов

Берём seed-сценарий `S4_campaign_draft → { campaign: cmp1, hr: hrAdmin }` и прогоняем policy-путь на реальных операциях:

1. **Старт.** `campaign.start` от `hrAdmin`: `draft → started`. Внутри `transitionCampaignStatus` за одну транзакцию: `ensureQuestionnairesForCampaignAssignments` + `createCampaignEmployeeSnapshotsForCampaignStart` (снимок оргструктуры) + `enqueueCampaignInvitesOnStart`. Повторный `start` → `changed: false`. `start` не из `draft` → `invalid_transition` («Campaign can be started only from draft.»).
2. **Freeze.** Первый `questionnaire.saveDraft` сотрудника: если `campaign.lockedAt` пуст — в той же транзакции ставится `lockedAt = now`. После этого `matrix.set` → `campaign_locked` («Campaign matrix is locked.»), а `weights.set` вне `draft`/`started` → `invalid_transition`. «Первый ответ = draft save = lock».
3. **Анонимность.** В кампании 2 peers заполнили анкету (`submitted`), а subordinates — ни одного. При `2 < 3` блок peers скрывается/сливается в `Other` (по `smallGroupPolicy`, default `"hide"`), а группа subordinates исчезает из расчёта. Manager (1 человек) виден всегда — он не-анонимный.
4. **Нормализация весов.** Раз subordinates выпали, остаются manager и peers. Конфигурация была `40/30/30/0`. ВНИМАНИЕ на расхождение doc ↔ код: §3.18 приводит **пропорциональный** пример `manager 57% / peers 43%` (`40/(40+30)`), а реализация `buildEffectiveGroupWeights` для случая **ровно двух** групп жёстко даёт `50/50` (строки 246–251). Пропорция включается только при `3+` группах. `self` (вес `0`) в `overallScore` не входит никогда.
5. **Role-based views.** Те же данные через три операции: `results.getHrView` (hr_admin — всё; hr_reader — без `rawText` open-комментариев), `results.getTeamDashboard` (manager — только своя команда, проверка `isCampaignSubjectManagedByEmployee`), `results.getMyDashboard` (employee — свой `employeeId`). Все три внутри зовут `getResultsHrView` — single source of truth.

#### 📐 Диаграмма: state machine кампании и точки policy-гейтов

```mermaid
stateDiagram-v2
    [*] --> draft : campaign.create
    draft --> draft : updateDraft / weights.set<br/>participants add / remove
    draft --> started : campaign.start<br/>snapshot taken + matrix frozen
    started --> ended : campaign.end / stop
    ended --> processing_ai : ai.runForCampaign
    processing_ai --> completed : webhook success
    processing_ai --> ai_failed : webhook failure
    ai_failed --> processing_ai : retry
    completed --> [*]

    note right of draft
        Всё редактируется.
        Первый saveDraft анкеты
        ставит lockedAt = now
        и замораживает matrix + weights
        ADR-0003
    end note

    note right of started
        read-only для end-users.
        matrix / model / weights менять нельзя
        иначе campaign_started_immutable
        или campaign_locked
    end note
```

#### 🏗️ Что построить (REBUILD_ROOT)

Снизу вверх, по feature-областям (детали — [[04-part-3-architecture#3.16. State machine кампании|Справочник A §3.15–3.18]] и [[06-part-5-technical-restoration#Глава 10: Results, Anonymity, Calculations|Справочник: Глава 10]]):

1. **Контракт** (`packages/api-contract/src/v1/legacy.ts`, реэкспорт через `packages/api-contract/src/campaigns.ts`): `campaignLifecycleStatuses` = `["draft","started","ended","processing_ai","ai_failed","completed"]` `as const` + тип `CampaignLifecycleStatus`; код ошибки `invalid_transition` в реестре; парсеры `parseCampaignTransitionInput/Output`, `parseCampaignWeightsSetInput/Output`, `parseCampaignUpdateDraftInput/Output`.
2. **DB state machine** (`packages/db/src/campaigns.ts`): `transitionCampaignStatus` (`targetStatus` ограничен `started`|`ended`), идемпотентность через `changed:false`, и три тонкие обёртки `startCampaign`/`endCampaign`/`stopCampaign`. При переходе в `started` — вызвать snapshot + invites + ensure-questionnaires.
3. **Freeze + lock** (`packages/db/src/questionnaires.ts` и `packages/db/src/matrix.ts`): в `saveQuestionnaireDraft` — установка `campaigns.lockedAt = now` при первом draft; `throwIfCampaignLocked` (`campaign_locked`) и `ensureCampaignParticipantsMutable` (требует `status === "draft"`).
4. **Snapshot** (`packages/db/src/snapshots.ts`): `createCampaignEmployeeSnapshotsUsingDb` — только при `status === "started"`, копия из `*History`-таблиц через `lte(startAt)`, `onConflictDoNothing` по `(campaignId, employeeId)` для идемпотентности.
5. **Anonymity + calculations** (`packages/db/src/results.ts`): `getResultsHrView` с `defaultAnonymityThreshold = 3`, `defaultSmallGroupPolicy = "hide"`; учитывать только `status = "submitted"`; `buildEffectiveGroupWeights` (ренормализация: `1 группа → 100`, `2 → 50/50`, `3+ → пропорция`); `self` исключён из `overallScore`.
6. **RBAC-гейты в core** (`packages/core/src/features/campaigns.ts`, `matrix.ts`, `questionnaires.ts`, `results.ts`): мутации — `hr_admin`; чтения — `hr_admin` + `hr_reader`; `recordAuditEvent` на `updateDraft`/transition/`weights.set`/`matrix.updated`. Три role-based view как отдельные операции поверх единого `getResultsHrView`.
7. **Схема** (`packages/db/src/schema/tables.ts`): `campaigns.status default "draft"`, `managerWeight 40`, `peersWeight 30`, `subordinatesWeight 30`, `selfWeight 0`, `lockedAt` nullable timestamptz.

#### 📖 Что прочитать (REFERENCE_ROOT)

- [[04-part-3-architecture#3.15. Snapshot pattern]] — что лежит: DDL `campaignEmployeeSnapshots` и Aha про «справедливость замороженного org-состояния»; зачем: понять, почему snapshot — доменная гарантия, на которую опираются результаты и manager-доступ.
- [[04-part-3-architecture#3.16. State machine кампании]] — что лежит: mermaid stateDiagram, таблица «можно/нельзя» по статусам, поля `campaigns`; зачем: каноническая опора для переходов и `invalid_transition`, на конкретные рёбра вешаются verifiable-критерии.
- [[04-part-3-architecture#3.18. Anonymity policy]] — что лежит: порог 3 (ADR-0002), merge-to-`Other`, `self = 0`, пример ренормализации и таблица трёх views; зачем: центральный policy-якорь стадии (ВНИМАНИЕ: doc-пример `57/43` — пропорция, а код для 2 групп даёт `50/50`).
- [[06-part-5-technical-restoration#Глава 10: Results, Anonymity, Calculations]] — что лежит: lifecycle анкеты (`not_started → in_progress → submitted`, `hidden ≠ deleted`), weight normalization; зачем: подтверждает, что в aggregation идут только `submitted`, а скрытые — не удалённые.
- `.memory-bank/adr/0002-anonymity-threshold.md` и `.memory-bank/adr/0003-freeze-on-draft-save.md` — зачем: «почему» порога и freeze в формате решения с trade-offs.

#### ✅ Верификация (стадия пройдена, когда)

Запускаемый критерий (это и есть **evidence**, а не «вроде работает»):

```bash
# Given: собран policy-домен (campaigns + matrix + questionnaires + results)
pnpm typecheck            # union "draft"|"started"|"ended"|"processing_ai"|"ai_failed"|"completed" согласован api-contract↔db
pnpm test                 # buildEffectiveGroupWeights: 1 группа→100, 2→50/50, 3+→пропорция; self вне overallScore
pnpm --filter @feedback-360/db test:db   # anonymity threshold=3, freeze-on-draft, visibility/merge_to_other — зелёные

# When: сначала ставим контекст актёра, затем гоним state-transition через CLI на seed S4_campaign_draft
pnpm --filter @feedback-360/cli cli -- company use $companyMain --role hr_admin --user-id $hrAdmin
pnpm --filter @feedback-360/cli cli -- campaign start $cmp1 --json
#   campaign start (через пробел), id — позиционный <campaign_id>, на transition есть только --json;
#   роль/актёр задаются на `company use --role ... --user-id ...`, а не на самом переходе

# Then: result.ok===true и result.data.status==="started";
#       повторный вызов → changed:false; start не из draft → ok:false, code "invalid_transition"

pnpm checks               # единый gate стадии: lint + typecheck + test + test:db + web build
```

Чеклист:

- [ ] `company use $companyMain --role hr_admin --user-id $hrAdmin` задаёт контекст актёра; затем `campaign start $cmp1 --json` на `draft` → `{ ok: true, data: { status: "started" } }`; повторный вызов → `changed: false`; не из `draft` → `{ ok: false, error: { code: "invalid_transition", … } }` («Campaign can be started only from draft.»).
- [ ] `campaign.end` разрешён только из `started`; `stop` и `end` оба ведут в `ended`.
- [ ] Первый `questionnaire.saveDraft` ставит `lockedAt = now`; после lock `matrix.set` → `campaign_locked`, изменение matrix/model в `started` → `campaign_started_immutable`.
- [ ] Snapshot создаётся **только** при `status === "started"`; повторный старт не плодит дублей (`onConflictDoNothing`).
- [ ] Группа с `< 3` валидными `submitted` скрыта/слита в `Other`; manager и self видны всегда; `NA`/`UNSURE` не учитываются в счёте оценщиков.
- [ ] `buildEffectiveGroupWeights` даёт сумму `100`; `self` (вес `0`) исключён из `overallScore`; зафиксировано расхождение doc `57/43` ↔ код `50/50` для двух групп.
- [ ] Зафиксировано расхождение заметки↔код по порогу анонимности: автор в заметках называет порог `<2` (скрыть при менее двух ответов, часть 2); проект реализует `threshold=3` (ADR-0002, §3.18, `results.ts`). Трактовать как сознательное усиление политики, а не соответствие 1:1.

Given-when-then: **Given** seed `S4_campaign_draft` с `{ campaign: cmp1, hr: hrAdmin }` и заданный контекст актёра `company use $companyMain --role hr_admin --user-id $hrAdmin`, **When** `cli campaign start $cmp1 --json`, **Then** `result.data.status === "started"`, снимок оргструктуры заморожен, а первый ответ сотрудника блокирует матрицу — каждый из этих эффектов подтверждается отдельным acceptance-сценарием, а не ручной проверкой.

#### 🔗 Выравнивание с автором

Концепции 24, 7, 6 из [[#Выравнивание с методологией автора (DEKSDEN, части 1–2)]]: доменная специфика feedback-360 — анонимность/роли/кампании (24, ч2·т4), acceptance given-when-then покрывает всю хотелку на стадиях 4/5/6/9 (7, ч1·т7), вертикальный слайс как единица приёмки (6, ч1·т4). Сверка по строкам таблицы выравнивания: 24 → строка 708 (якорь «<2 ответов — группа не показывается»), 7 → строка 691, 6 → строка 690. Якоря-цитаты: «вершиной всей проверки является, по мне, сценарий, acceptance test, который тестирует вертикальный слайс» (часть 1, тема 7); «На каждую хотелку — **приёмочный сценарий по given/when/then**, покрывающий её целиком (несколько сценариев, если нужно)» (часть 1, раздел «Практические выводы»).

ВНИМАНИЕ на расхождение заметки↔код по концепции 24: якорь концепции — порог `<2` («если от группы респондентов получено меньше двух оценок, группа не показывается», часть 2), а реализация и ADR-0002/§3.18 используют `threshold=3` («минимум 3 оценщика»). Это второе, более фундаментальное расхождение (помимо doc `57/43` ↔ код `50/50`): фиксируем его как сознательное усиление политики анонимности, а не как соответствие 1:1 заметке автора.

#### ⚠️ Антипаттерны стадии · 🧩 Артефакты

- Антипаттерны: бизнес-логика анонимности/freeze/переходов в route handler, CLI или React вместо core ([[08-part-7-antipatterns#Антипаттерн 3: Бизнес-логика в route handlers, CLI или UI]]) — `threshold=3`, merge-to-`Other`, нормализация весов и started-immutability живут в `core`/`db` (`results.ts`, `campaigns.ts`, `matrix.ts`), а UI/CLI остаются thin; acceptance на случайных UUID вместо seed/handles ([[08-part-7-antipatterns#Антипаттерн 5: Acceptance без seed/handles]]) — policy-сценарии гоняются на `S4_campaign_draft → { cmp1, hrAdmin }`, а не на сырых ID из БД.
- Артефакты: рабочая state machine `campaign.start`/`end`/`stop` с идемпотентностью и `invalid_transition`; snapshot-freeze (`lockedAt`, `campaignEmployeeSnapshots`); anonymity-движок `getResultsHrView` с порогом `3` и нормализацией весов; три role-based view поверх единого источника правды; зелёный `pnpm checks` и evidence-блок с зафиксированными расхождениями doc `57/43` ↔ код `50/50` и заметка `<2` ↔ код `threshold=3`.

---

### Стадия 7 — Notifications: outbox и scheduling

> Исход: notifications через outbox (generate → dispatch → retry), timezone-aware scheduling.

#### 🎯 Зачем эта стадия (на пальцах)

Представьте официанта, который принял заказ и тут же побежал на кухню — но на кухне пожар. Если он держал заказ только в голове, заказ потерян: гость уйдёт голодным. Хороший ресторан работает иначе — заказ сначала **записывают на бумажку и вешают на стойку** (это решение «гостю нужна еда»), а уже потом отдельный человек носит блюда (это доставка). Если доставка сорвалась — бумажка осталась, повторить можно без переспрашивания гостя. Outbox — ровно эта бумажка на стойке. Бизнес-решение «надо отправить напоминание» фиксируется в таблице `notification_outbox` **в той же транзакции**, что и сам бизнес-факт, а отправку email берёт на себя отдельная операция, которую можно сколько угодно ретраить.

Рамка от автора: уведомления — это reliability-домен, а reliability **проверяется acceptance-тестом на детерминированном seed**, а не фразой «вроде письма уходят». Поэтому стадия не про «прикрутить SMTP», а про разделение `generate → dispatch → retry` так, чтобы каждый шаг был воспроизводим: идемпотентность не плодит дубли даже при race, ретраи идут по предсказуемому графику, а stub-провайдер даёт детерминированный сбой по требованию теста.

#### 🧠 Ключевые концепции простыми словами

- **Outbox pattern.** Суть: «решение уведомить» пишется в `notification_outbox` со `status='pending'` отдельно от факта отправки. Аналогия: бумажка с заказом на стойке отдельно от официанта, который носит блюда. Почему важно: при сбое email бизнес-логику **не пересчитывают** — запись уже в БД, dispatch просто доводит её до канала.
- **Idempotency key.** Суть: у каждой записи есть детерминированный ключ вида `campaign:{id}:event:campaign_reminder:employee:{id}:day:{dateBucket}` и UNIQUE-констрейнт `uq_notification_outbox_idempotency`; повторная генерация делает `.onConflictDoNothing()`. Аналогия: на стойке нельзя повесить две одинаковые бумажки на один заказ. Почему важно: повторный `generateReminders` за тот же день даёт `deduplicated > 0`, а не дубли писем — даже при гонке двух процессов.
- **Exponential backoff и dead-letter.** Суть: retryable-сбой переводит запись обратно в очередь с `nextRetryAt = BASE * 2**(attemptNo-1)` (capped), а после исчерпания попыток — в терминальный `dead_letter`. Аналогия: курьеру, который не дозвонился, говорят «перезвони через 1, потом 2, потом 4 минуты», но не бесконечно. Почему важно: временный сбой провайдера не теряет уведомление и не заваливает его мгновенными повторами; `MAX_ATTEMPTS=10`, `BASE_RETRY_DELAY_MS=60_000` (1 мин), `MAX_RETRY_DELAY_MS=24ч`.
- **Timezone-aware scheduling.** Суть: `evaluateReminderSchedule` через `Intl.DateTimeFormat` с `timeZone` проверяет quiet hours и weekday в **локальном** времени компании; настройки лежат в `notification_settings`. Аналогия: не звонить человеку в 3 ночи по его часовому поясу, даже если у тебя полдень. Почему важно: рассылка уважает рабочее окно получателя — по умолчанию `quiet_hours_start=8`, `quiet_hours_end=20`, `reminder_weekdays=[1,3,5]`.
- **Generate ≠ dispatch (две операции).** Суть: `notifications.generateReminders` (создать записи) и `notifications.dispatchOutbox` (отправить/ретраить) — это **разные операции** core-слоя, обе под ролью `hr_admin`. Аналогия: повар, который выписывает заказ, и официант, который его носит, — это не один и тот же человек. Почему важно: ровно это разделение и есть outbox; их можно тестировать и запускать независимо.

#### 🔧 Иллюстративный пример: жизнь одного reminder от seed до `sent`

Возьмём детерминированный seed с handles `$cmp1` (компания) и `$hrAdmin` (роль `hr_admin`), стартовавшую кампанию и сотрудника с незавершённой оценкой. Зафиксируем `now` так, чтобы попасть в рабочее окно (`weekday=3`, `localHour=10`), и `dateBucket = "2026-03-06"`.

1. **Generate.** `runNotificationsGenerateReminders` проверяет `hasRole(["hr_admin"])` и `ensureContextCompany`, делегирует в `generateReminderOutbox`. Тот в одной транзакции пишет запись со `status='pending'` и `idempotencyKey = "campaign:$cmp1:event:campaign_reminder:employee:$emp:day:2026-03-06"`. Результат: `{ generated: 1, deduplicated: 0 }`.
2. **Повторный generate (race-проверка).** Запускаем ту же операцию ещё раз за тот же день. UNIQUE срабатывает, `.onConflictDoNothing()` глотает вставку: `{ generated: 0, deduplicated: 1 }` — **дубля письма не будет**.
3. **Dispatch с временным сбоем.** Stub-провайдер настроен `__stubFailUntilAttempt = 1` (`attemptNo <= failUntilAttempt` → падает только первая попытка). `dispatchNotificationOutbox` берёт `pending`, пробует отправить — retryable-сбой. Запись возвращается в очередь: `status='pending'`, `nextRetryAt = now + 60_000` (1 мин), в `notification_attempts` пишется строка `attempt_no=1`. Для диагностики `resolveDeliveryStatus` покажет `retry_scheduled` (pending + есть `nextRetryAt` + `attempts>0`).
4. **Retry и успех.** Сдвигаем время за `nextRetryAt` (в тестах — `setNotificationOutboxNextRetryAtForDebug`), повторяем dispatch. Вторая попытка проходит: `status='sent'`, `sentAt` проставлен, `notification_attempts` пополняется строкой `attempt_no=2`. Итог dispatch: `{ sent: 1, failed: 0, remainingPending: 0 }`.

Тот же цикл срабатывает на старте кампании: `enqueueCampaignInvitesOnStartInDb` кладёт invite-записи в тот же outbox (idempotencyKey без day-bucket), и `dispatchOutbox` доставляет их единообразно — invite и reminder проходят одну трубу.

#### 📐 Диаграмма: жизненный цикл записи outbox

```mermaid
stateDiagram-v2
    [*] --> pending : generateReminders<br/>onConflictDoNothing
    pending --> sent : dispatch ok<br/>sentAt проставлен
    pending --> retry_scheduled : retryable сбой<br/>nextRetryAt = backoff
    retry_scheduled --> pending : now >= nextRetryAt<br/>попытка снова
    pending --> failed : non-retryable сбой
    retry_scheduled --> dead_letter : attemptNo >= MAX_ATTEMPTS=10
    sent --> [*]
    failed --> [*]
    dead_letter --> [*]
```

#### 🏗️ Что построить (REBUILD_ROOT)

Снизу вверх (детали — [[04-part-3-architecture#3.17. Outbox pattern для notifications|Справочник A §3.17]] и [[06-part-5-technical-restoration#Глава 11: Notifications — Outbox Pattern|Справочник: Глава 11]]):

1. **Миграции БД** (`packages/db/drizzle`): `0010_ft0061_notifications_outbox.sql` — таблицы `notification_outbox` (`status` default `'pending'`, `idempotency_key` not null + UNIQUE `uq_notification_outbox_idempotency`, `attempts` default 0, `next_retry_at`, `sent_at`, индексы `idx_notification_outbox_status_created_at` и `idx_notification_outbox_campaign`) и `notification_attempts` (FK `outbox_id` on delete cascade, UNIQUE `uq_notification_attempt_outbox_attempt`). На обеих — `enable` + `force row level security` с политикой `app.is_service_role() OR app.has_company_access(company_id)`. Затем `0011_ft0181_notification_settings.sql` — `notification_settings` (`reminder_scheduled_hour` default 10, `quiet_hours_start` default 8, `quiet_hours_end` default 20, `reminder_weekdays` default `[1,3,5]`, UNIQUE `uq_notification_settings_company`).
2. **DB-реализация** (`packages/db/src/notifications.ts`): `generateReminderOutbox` (транзакция + `.onConflictDoNothing()`), `dispatchNotificationOutbox` (выборка `pending` где `nextRetryAt IS NULL` или `<= now` плюс `failed` где `nextRetryAt <= now` и `attempts <= MAX_ATTEMPTS-1`; на успех `status='sent'`/`sentAt`, на retryable — backoff через `buildRetryDelayMs`, иначе `failed`/`dead_letter`; строка в `notification_attempts` на каждую попытку). Константы `MAX_ATTEMPTS=10`, `BASE_RETRY_DELAY_MS=60_000`, `MAX_RETRY_DELAY_MS=24ч`. `evaluateReminderSchedule` через `Intl.DateTimeFormat`. Провайдеры `'stub'` (с `__stubFailUntilAttempt`/`__stubPermanentFailure`) и `'resend'`. Плюс `resolveDeliveryStatus` и `listNotificationDeliveryDiagnostics`.
3. **Core-оркестрация** (`packages/core/src/features/notifications.ts`): `runNotificationsGenerateReminders` и `runNotificationsDispatchOutbox` — обе `hasRole(["hr_admin"])` + `ensureContextCompany`, делегируют в db через `OperationResult`-обёртку. Плюс `settings get/upsert/preview` (upsert пишет audit `notifications.settings_updated`), `templateCatalog/Preview`, `deliveryDiagnostics`.
4. **Контракты** (`packages/api-contract/src/notifications.ts`): re-export parse-функций и типов (`NotificationsDispatchProvider`, `NotificationsGenerateRemindersInput/Output`, settings/templates/diagnostics).
5. **Acceptance вертикального слайса**: FT-тест `packages/core/src/ft/ft-0061-outbox-dispatch.test.ts` (цикл dispatch+retry на stub) и CLI/client-тесты `ft-0061-notifications-cli.test.ts`, `ft-0061-notifications-client.test.ts`, `ft-0181-notification-center-cli.test.ts`.

#### 📖 Что прочитать (REFERENCE_ROOT)

- [[04-part-3-architecture#3.17. Outbox pattern для notifications]] — что лежит: минимальный drizzle-фрагмент `notificationOutbox`, цикл `generate → dispatch → retry`, Aha-момент про разделение «решения» и «отправки»; зачем: каноническая опора схемы и инварианта идемпотентности/ретраев.
- [[06-part-5-technical-restoration#Глава 11: Notifications — Outbox Pattern]] — что лежит: путь-по-шагам стадии, шаг 11.1 template system, шаг 11.2 timezone-aware scheduling, checkpoint «`campaign.start` → invite в outbox»; зачем: пошаговый маршрут (учтите: в коде quiet hours/weekday реализованы в `notification_settings` + `evaluateReminderSchedule`, глава упрощает иллюстративно).
- `.memory-bank/spec/notifications/outbox-and-retries.md` — что лежит: spec по outbox/retries, на который ссылается `@see` core-файла; зачем: зафиксированные WHAT/WHY поведения ретраев и dead-letter.

#### ✅ Верификация (стадия пройдена, когда)

Запускаемый критерий (это и есть **evidence**, а не «email вроде уходит»):

```bash
# Given: собраны миграции + db/core/contract слой и детерминированный seed с handles
pnpm --filter @feedback-360/db db:migrate
# → применяет 0010_ft0061_notifications_outbox.sql и 0011_ft0181_notification_settings.sql, exit 0

pnpm --filter @feedback-360/db db:health
# → таблицы notification_* и RLS-политики на месте, миграции применены

# When: гоняем acceptance вертикального слайса (generate → dispatch → retry)
pnpm --filter @feedback-360/core test -- src/ft/ft-0061-outbox-dispatch.test.ts
pnpm --filter @feedback-360/cli  test -- src/ft-0061-notifications-cli.test.ts

# Then: stub-провайдер с __stubFailUntilAttempt ретраит и в итоге отправляет,
# повторный generate даёт deduplicated > 0, дублей писем нет
```

Чеклист:

- [ ] `db:migrate` применяет `0010`/`0011`; на `notification_outbox` и `notification_attempts` включён `force row level security` с политикой `app.is_service_role() OR app.has_company_access(company_id)`.
- [ ] Повторный `notifications.generateReminders` за тот же `dateBucket` даёт `{ generated: 0, deduplicated: N }` — UNIQUE `uq_notification_outbox_idempotency` + `.onConflictDoNothing()` работают.
- [ ] Retryable-сбой даёт `status='pending'` с `nextRetryAt`, после `MAX_ATTEMPTS=10` — `dead_letter`; backoff = `BASE_RETRY_DELAY_MS=60_000 * 2**(attemptNo-1)`, capped на `24ч`.
- [ ] Quiet hours соблюдены: по умолчанию `quiet_hours_start=8`, `quiet_hours_end=20`, `reminder_weekdays=[1,3,5]`, `reminder_scheduled_hour=10`.
- [ ] `pnpm checks` (lint + typecheck + test + test:db + web build) зелёный — quality gate стадии.
- [ ] Evidence-блок записан: дата + команды + JSON-вывод `dispatchOutbox` с `{ sent, failed, remainingPending }`.

Given-when-then: **Given** загружен seed с `$cmp1`/`$hrAdmin`, стартовавшей кампанией и зафиксированным `now`/`dateBucket`, **When** от лица `hrAdmin` выполнить `generateReminders → dispatchOutbox` со stub-провайдером `__stubFailUntilAttempt=1`, **Then** первая попытка → `retry_scheduled` (`attempt_no=1`), вторая (после `nextRetryAt`) → `{ sent: 1, failed: 0, remainingPending: 0 }` (`attempt_no=2`), а повторный generate → `deduplicated > 0`.

#### 🔗 Выравнивание с автором

Концепции 6, 7, 9 из [[#Выравнивание с методологией автора (DEKSDEN, части 1–2)]]. Якоря: «вершиной всей проверки является, по мне, сценарий, acceptance test, который тестирует вертикальный слайс» (часть 1, тема 7); «На каждую хотелку — детерминированный приёмочный сценарий с подготовленным seed; без него фичу „проверить невозможно“» (часть 2, раздел «Что взять с собой», тема 6).

#### ⚠️ Антипаттерны стадии · 🧩 Артефакты

- Антипаттерны: acceptance без seed/handles ([[08-part-7-antipatterns#Антипаттерн 5: Acceptance без seed/handles]]) — ретраи и идемпотентность проверяемы только на фиксированных `now`/`dateBucket` и handles `$cmp1`/`$hrAdmin`, иначе тест привязан к случайному UUID/времени и становится brittle; считать feature готовой без evidence ([[08-part-7-antipatterns#Антипаттерн 7: Считать feature готовой без evidence]]) — «email вроде уходит» не доказательство, нужен `pnpm checks` + прогнанный dispatch-с-ретраями + записанный JSON-вывод в memory bank.
- Артефакты: `packages/db/drizzle/0010_ft0061_notifications_outbox.sql` (таблицы `notification_outbox` + `notification_attempts`, idempotency UNIQUE, индекс status+created_at, RLS force); `packages/db/drizzle/0011_ft0181_notification_settings.sql` (timezone-aware scheduling); `packages/db/src/notifications.ts` (`generateReminderOutbox`/`dispatchNotificationOutbox` с backoff, dead-letter, stub/resend); `packages/core/src/features/notifications.ts` (оркестрация с RBAC + audit); `packages/api-contract/src/notifications.ts` (контракты); `packages/core/src/ft/ft-0061-outbox-dispatch.test.ts` + `packages/cli/src/ft-0061-notifications-cli.test.ts` + `packages/client/src/ft-0061-notifications-client.test.ts` + `packages/cli/src/ft-0181-notification-center-cli.test.ts` (acceptance); `.memory-bank/spec/notifications/outbox-and-retries.md` (spec).

---

### Стадия 8 — Тонкий web layer

> Исход: Next.js как thin layer (route handlers без бизнес-логики, server components, shadcn, стабильные `data-testid`, Page Object Model, screen registry).

#### 🎯 Зачем эта стадия (на пальцах)

Представьте, что у вас уже есть надёжный мотор и коробка передач (`core` + `client` + CLI из Стадии 4), и теперь вы надеваете на машину **кузов**. Кузов красивый и его видно, но он ничего не решает: руль крутит не он, тормозит не он — он только передаёт нажатия педалей на тот же мотор. Ровно так автор требует строить web: Next.js — это **delivery adapter**, тонкая оболочка, которая парсит HTTP/форму, резолвит сессию в `OperationContext`, зовёт **тот же** `Feedback360Client` и мапит typed-ошибку в HTTP-статус или редирект. Бизнес-правила в `core`, а не в React-компоненте.

Рамка от автора: UI приходит **последним**, поверх уже стабильного non-UI delivery path — и проверяется он не «на глаз», а Playwright-сценариями через стабильные `data-testid`. Критерий тонкости предельно жёсткий: «вы можете удалить UI и не потерять доменную логику». Если, выкинув `apps/web`, вы теряете хоть одно правило — слой не thin, и это баг архитектуры, а не фича.

#### 🧠 Ключевые концепции простыми словами

- **Thin route handler (delivery adapter).** Суть: handler делает ровно `createInprocClient → client.someOperation → отдать result`, ~30-50 строк, zero business logic. Аналогия: официант передаёт заказ на кухню и приносит блюдо, но сам не готовит. Почему важно: правило живёт в `core`, тестируется in-proc, а не через дорогой браузер; UI остаётся выкидываемым.
- **Server Component, который грузит данные.** Суть: `page.tsx` — `async` Server Component, делегирует загрузку в loader (`loadHomeDashboard`/`resolveAppOperationContext`) и мапит typed-error в `redirect(...)` или `PageErrorState`. Аналогия: витрина магазина — показывает товар, но не производит его. Почему важно: один экран = один `screenId`, данные приходят из того же client, презентация отделена от логики.
- **Стабильные `data-testid` + screen registry.** Суть: каждый route-level экран получает канонический `screen_id` (`SCR-APP-HOME`), а интерактивные элементы — предсказуемые `data-testid` через `testIdScope`. Аналогия: у каждой комнаты в здании есть постоянный номер на двери — по нему всегда найдёшь её, даже если перекрасили стены. Почему важно: селекторы переживают рестайлинг, AI-агент и Playwright «бегают надёжно».
- **Page Object Model (POM).** Суть: класс-обёртка над экраном (`QuestionnairePagePom`, `HrResultsPagePom`) — объект = API к экрану, все селекторы только через `getByTestId`. Аналогия: пульт от телевизора — кнопки стабильны, неважно, как перерисовали меню внутри. Почему важно: сценарии пишутся на языке домена (`submit()`, `readOverallScore()`), а не на CSS-классах.
- **Design system как SSoT визуального языка.** Суть: повторяющиеся визуальные правила (токены, статус-цвета, page header, summary strip) живут в `spec/ui/design-system/`, shadcn подключён через `components.json`. Аналогия: фирменный гайдбук — один источник стиля для всех вывесок. Почему важно: визуальный язык не расползается по экранам, GUI эволюционирует без потери тестируемости (принципы 15-17).

#### 🔧 Иллюстративный пример: экран `SCR-APP-HOME` от запроса до селектора

1. **Route-level экран** (`apps/web/src/app/page.tsx`): `async` Server Component `HomePage` с JSDoc `@screenId SCR-APP-HOME`, `@testIdScope scr-app-home`.
2. **Делегирование данных**: `resolveAppOperationContext()` резолвит сессию, `loadHomeDashboard()` зовёт client — **никаких** расчётов в компоненте.
3. **Маппинг typed-error**: нет сессии → `redirect('/auth/login')`; нет выбранной компании → `redirect('/select-company')`; иная ошибка → `PageErrorState`.
4. **Стабильные `data-testid`**: корень `scr-app-home-root`, маркер роли `home-role-${role}` (например `home-role-hr_admin`), а также `home-intro-description`, `home-empty-state`, `metric.testId`, `task.testId`.
5. **Проверка из POM/теста**: Playwright делает `page.getByTestId('home-role-hr_admin')` — селектор стабилен, расчётов в UI нет.

Результат — детерминированная связка: `screen_id (SCR-APP-HOME) ↔ testIdScope (scr-app-home) ↔ @screenId в JSDoc ↔ data-testid в DOM`. FT-0212 прогоняет её на seed-сценарии `S9_campaign_completed_with_ai` по маршрутам `/`, `/hr/employees`, `/hr/org`, `/questionnaires`, `/results/hr` и снимает screenshot-evidence с суффиксом `__(SCR-RESULTS-HR).png`.

> ⓘ **ai-слайс — вне rebuild-scope.** Сам AI-пайплайн (`ai.runForCampaign`, переходы `processing_ai → completed`/`ai_failed`) в учебном репо **не строится**: seed `S9_campaign_completed_with_ai` ставит `campaigns.status = completed` напрямую, поэтому web-стадия не требует построенного AI. В [[#Стадия 6 — Policy-heavy домен|Стадии 6]] эти статусы лишь **объявлены** в `campaignLifecycleStatuses`; своими руками кампанию доводим до `ended`. Если решите строить ai-слайс — поставьте его между Стадиями 7 и 8.

#### 📐 Диаграмма: thin web layer поверх того же client

```mermaid
flowchart LR
    BROWSER["Браузер<br/>Playwright getByTestId"] --> PAGE["page.tsx<br/>Server Component SCR-APP-HOME"]
    PAGE --> CTX["resolveAppOperationContext<br/>session → OperationContext"]
    CTX --> LOADER["loadHomeDashboard<br/>тонкий вызов"]
    LOADER --> CLIENT["Feedback360Client<br/>тот же client, что и CLI"]
    CLIENT --> CORE["core: dispatchOperation<br/>ВСЯ бизнес-логика здесь"]
    CORE -->|"OperationResult"| CLIENT
    CLIENT -->|"ok: data → render"| PAGE
    CLIENT -.->|"ok:false → typed error"| MAP["redirect или PageErrorState"]
    PAGE -->|"stable data-testid"| BROWSER
```

#### 🏗️ Что построить (REBUILD_ROOT)

Поверх готового delivery path (детали — [[06-part-5-technical-restoration#Глава 12: Web Application — Next.js как Thin Layer|Справочник: Глава 12]] и [[07-part-6-curriculum#Module 6: Тонкий web layer|Module 6]]):

1. `apps/web` (`@feedback-360/web`): Next.js 15 app, зависит от `@feedback-360/client` (workspace), `@feedback-360/api-contract`, `@feedback-360/db` — **не** от `core` напрямую. Скрипты `dev`/`build`/`start`/`lint`/`typecheck`/`test`/`test:smoke:beta`.
2. `apps/web/next.config.mjs`: `transpilePackages` для `@feedback-360/api-contract` и `@feedback-360/db`, `outputFileTracingRoot` (монорепо), опциональный `withSentryConfig` (только при заданных `SENTRY_*`), tunnel `/monitoring`. Сборочная конфигурация, без бизнес-логики.
3. `apps/web/components.json` + `apps/web/src/app/layout.tsx`: shadcn (`new-york`, `baseColor slate`, `lucide`, RSC) и root-shell (`html lang="ru"`, шрифт Inter latin+cyrillic, metadata `go360go (beta)`).
4. `apps/web/src/app/page.tsx`: route-level Server Component с `@screenId`/`@testIdScope`, делегированием в loader и стабильными `data-testid`. Минимум один thin route handler поверх существующих operations.
5. `packages/xe-runner/src/pom/*`: `QuestionnairePagePom`, `Employee/Manager/HrResultsPagePom` — все селекторы через `getByTestId`.
6. `apps/web/playwright/tests/ft-0212-*`: smoke-сценарий нормализации scope-`testid` по ключевым маршрутам со screenshot-evidence.

#### 📖 Что прочитать (REFERENCE_ROOT)

- [[06-part-5-technical-restoration#Глава 12: Web Application — Next.js как Thin Layer]] — что лежит: route handler pattern (~30-50 строк, zero business logic), UI stack (Tailwind v4, shadcn, Server/Client Components); зачем: главный справочник стадии — web как delivery adapter (guardrail ADR-0001).
- [[04-part-3-architecture#3.10. Transport abstraction]] — что лежит: один typed API независимо от in-proc/HTTP; зачем: понять, почему web **не дублирует** логику — тот же `Feedback360Client`, web берёт HTTP transport, результат `OperationResult<T>` одинаков.
- [[06-part-5-technical-restoration#Глава 15: XE Scenarios, Guides, UI Traceability]] — что лежит: screen registry, test ID registry, POM, суффикс `__(SCR-...)` в скриншотах; зачем: UI traceability и deterministic assertions для AI-агента (принципы 15-17).
- [[07-part-6-curriculum#Module 6: Тонкий web layer]] — что лежит: curriculum-якорь, build/read/done-критерии; зачем: критерий тонкости — «вы можете удалить UI и не потерять доменную логику».
- `.memory-bank/spec/ui/screen-registry.md` — что лежит: канонические `screen_id`; зачем: источник `SCR-*` для frontmatter, JSDoc и `testIdScope` (принцип 16).

#### ✅ Верификация (стадия пройдена, когда)

Запускаемый критерий (это evidence, а не «вроде открывается»):

```bash
# Given: собран apps/web поверх стабильного client/core path
pnpm --filter @feedback-360/web typecheck        # tsc --noEmit -p tsconfig.json — зелёный
pnpm --filter @feedback-360/web lint             # biome check . — без нарушений
pnpm --filter @feedback-360/web build            # next build собирает app-dir

# When: гоняем FT-0212 (нормализация scope-testid по маршрутам, включая /results/hr)
pnpm --filter @feedback-360/web exec playwright test --config playwright/playwright.config.mjs tests/ft-0212-testid-normalization.spec.ts

# Then: трассировка SCR-* и getByTestId на месте
rg -n "@screenId SCR-" apps/web/src/app          # @screenId SCR-APP-HOME в page.tsx
rg -n "getByTestId" packages/xe-runner/src/pom   # все POM-селекторы через getByTestId
```

Чеклист:

- [ ] `apps/web` зависит от `@feedback-360/client`, а **не** от `@feedback-360/core` — thin layer поверх того же client.
- [ ] `page.tsx` не содержит бизнес-расчётов: данные через loader/client, typed-error мапится в `redirect(...)`/`PageErrorState`.
- [ ] Связка `SCR-APP-HOME ↔ scr-app-home ↔ home-role-${role}` присутствует (FT-0212 видит `home-role-hr_admin` на `/`).
- [ ] В `packages/xe-runner/src/pom/*` **нет** CSS/класс-локаторов — только `getByTestId` (анти-Антипаттерн 8).
- [ ] Screenshot-evidence записан с суффиксом `__(SCR-RESULTS-HR).png` для трассировки.

Given-when-then: **Given** seed `S9_campaign_completed_with_ai` загружен и сессия залогинена через `/api/dev/test-login`, **When** FT-0212 (`ft-0212-testid-normalization.spec.ts`) проходит маршрут `/results/hr`, **Then** видны нормализованные scope-`testid` `scr-results-hr-root` и `results-layout-hero`, и assertion срабатывает детерминированно (стабильные id → сценарии «бегают надёжно»).

#### 🔗 Выравнивание с автором

Концепции 12, 19, 20 из [[#Выравнивание с методологией автора (DEKSDEN, части 1–2)]] (12 — CLI вместо GUI; 19 — POM + `data-testid`; 20 — N2N vs «гуляющий» AI-QA). Screen-id трассировка идёт от **Принципа 16** (`Screen IDs are mandatory for UI traceability`) из `.memory-bank/mbb/principles.md`, как и в теле стадии. Якоря: «когда модель работает с CLI-интерфейсом, она делает это легко, непринужденно, она этому обучена» (часть 1, тема 12) — поэтому web остаётся тонким поверх того же client; «в вебе идентификаторы уходят в атрибут **`data-testid`**, который умеют использовать Playwright или Chrome CDP… Обёртки экранов — **Page Object Model** (объект = API к экрану)» (часть 2, тема 9).

#### ⚠️ Антипаттерны стадии · 🧩 Артефакты

- Антипаттерны: селекторы по Tailwind-классам (`page.locator('.bg-blue-500')`) вместо `data-testid` ([[08-part-7-antipatterns#Антипаттерн 8: UI без stable identifiers]]); валидация бизнес-правил/расчёты в route handler или React-компоненте, делающие UI не-thin ([[08-part-7-antipatterns#Антипаттерн 3: Бизнес-логика в route handlers, CLI или UI]]).
- Артефакты: `apps/web/src/app/layout.tsx` (root-shell, `lang=ru`, Inter, metadata `go360go (beta)`); `apps/web/src/app/page.tsx` (`SCR-APP-HOME`, `@screenId`/`@testIdScope`, стабильные `data-testid`); `apps/web/package.json` (скрипты + dep `@feedback-360/client`); `apps/web/components.json` и `apps/web/next.config.mjs`; `packages/xe-runner/src/pom/questionnaire.ts` и `packages/xe-runner/src/pom/results.ts`; `apps/web/playwright/tests/ft-0212-testid-normalization.spec.ts`; screenshot `.memory-bank/evidence/EP-021/FT-0212/2026-03-07/step-01-normalized-selectors__(SCR-RESULTS-HR).png`; `.memory-bank/spec/ui/screen-registry.md` и `.memory-bank/spec/ui/design-system/`.

---

### Стадия 9 — Hardening, quality gates, evidence

> Исход: 4 уровня тестов, `pnpm checks` как gate, simplify против over-engineering, evidence-based acceptance, изоляция окружений local/beta/prod.

#### 🎯 Зачем эта стадия (на пальцах)

Представьте автомеханика, который собрал двигатель и говорит «ну, вроде заводится». Вы сядете в такую машину? Нет — вам нужен **штамп ОТК**: прогон по стенду, протокол, дата. Стадия 9 — это про то, как перестать говорить «вроде работает» и начать предъявлять **доказательства**. Автор формулирует это жёстче всего: тесты — не «дополнительная работа, когда будет время», а **гигиена**, как помыть руки перед едой. Их отсутствие не «снижает качество» — оно делает движение вперёд невозможным, потому что без объективной проверки вы не знаете, что вообще построили.

Рамка от автора: у фичи два разных гейта, которые нельзя смешивать. Первый — **quality gate** (`pnpm checks`): код вообще валиден — линтер, типы, тесты, сборка зелёные. Второй — **acceptance**: эта конкретная ценность доехала до пользователя, проверена сценарием на стабильных handles, и результат **записан** с датой. Плюс две операционные дисциплины поверх: изоляция окружений (`local` → `beta` → `prod`, агента нельзя пускать к проду в принципе) и упрощение против over-engineering (особенно с Opus, который «срезает углы» и оборачивает всё в лишние абстракции). Фича не «готова», пока все четыре не закрыты.

#### 🧠 Ключевые концепции простыми словами

- **Пирамида из 4 уровней тестов.** Суть: Unit (политики/расчёты/state machine, `packages/core`, мгновенно) → Integration (use-case + реальная БД, `core`/`db`, секунды) → Contract (parse-функции, форма DTO, `api-contract`, мгновенно) → E2E (сквозной user journey, `apps/web/playwright`, минуты). Аналогия: много дешёвых датчиков внизу и несколько дорогих краш-тестов наверху. Почему важно: разные уровни ловят **разные** баги; быстрый низ гоняется постоянно, дорогой верх — реже, но именно он подтверждает ценность.
- **`pnpm checks` как единый gate.** Суть: одна команда последовательно гоняет `lint` (Biome) → `typecheck` (`tsc --noEmit`) → `test` (unit/contract) → `test:db` (интеграция с реальной БД) → web build. Аналогия: один рубильник «выпускать / не выпускать», а не пять разрозненных кнопок, про которые легко забыть. Почему важно: gate либо целиком зелёный, либо фичи нет — нельзя «почти прошло».
- **Evidence-based acceptance.** Суть: фича `Completed` только когда (1) `checks` зелёный, (2) acceptance-сценарий прогнан **после** реализации, (3) команды + вывод + дата записаны в memory bank. Аналогия: не «я сделал», а подписанный акт выполненных работ с протоколом испытаний. Почему важно: «закодировал, типы проходят, в браузере работает» — это не evidence, это обещание.
- **Изоляция окружений `local`/`beta`/`prod`.** Суть: три окружения, которые создаются всегда; агентские/тестовые операции структурно разрешены только в `local` и `beta`, прод исключён на уровне кода. Аналогия: испытания взрывчатки на полигоне, а не в жилом доме. Почему важно: агент с широкими полномочиями рано или поздно ошибётся — бета ловит ошибку до того, как она сотрёт боевую базу.
- **Упрощение против over-engineering.** Суть: отдельная фаза ревью — вычистить ненужные абстракции, которые модель добавила «на всякий случай». Аналогия: редактор вычёркивает лишние слова, чтобы текст читался. Почему важно: упрощение — это **экономия внимания модели**: чем проще код, тем лучше будущий агент с ним справится.
- **Happy path и error-ветки — в РАЗНЫХ свежих контекстах (концепция 8).** Суть: после зелёного happy-path открой **новый** контекст (или сабагент-оркестратор) с единственной задачей — перечислить все error/edge-ветки слайса и написать given-when-then на каждый `OperationError` code. Аналогия: корректор, который текст не писал, видит опечатки, которых не видит автор. Почему важно: «happy path в контексте слепит агента к ошибкам» — из того же контекста error-ветки систематически недопокрываются. Мини-шаг в 🏗️: на каждый typed-код ошибки слайса — отдельный acceptance, написанный в свежем контексте.

#### 🔧 Иллюстративный пример: как закрывается слайс results-views

Возьмём фичу «дашборды результатов» (`FT` на операциях `results.getMyDashboard` / `getHrView`). Как агент доводит её до `Completed` через все 4 уровня:

1. **Unit (level 1):** проверка расчёта итогового балла на весах `manager=40 / peers=30 / subordinates=30 / self=0` и порога анонимности (группа с `< 3` оценщиками по умолчанию **скрывается** — `small_group_policy=hide`; при политике `merge_to_other` peers + subordinates сливаются в `Other`, и порог применяется к объединённой группе). БД не нужна — `cross-env FEEDBACK360_SKIP_DB_TESTS=1 vitest run`.
2. **Contract (level 1):** `parseResultsGetHrViewOutput` принимает валидный DTO и отвергает кривой — форма ответа зафиксирована.
3. **Integration (level 2):** `ft-0055-results-views.test.ts` поднимает реальную БД, прогоняет use-case целиком; в curated fast-lane (`test:db`) он один из трёх файлов.
4. **CLI-acceptance (level 3):** seed сценария (например `S8_campaign_ended`) выдаёт стабильные handles, given-when-then проверяется на них, а не на «id, подсмотренном в базе руками».
5. **E2E (level 4):** Playwright-спека на **beta**-стенде убеждается, что `employee` не видит сырые комментарии, а `hr_admin` видит.

Финал — `pnpm checks` зелёный целиком, и в `verification-matrix.md` появляется строка с `what / where / how / quality_gate / acceptance_gate / result` и датой. Только теперь `FT` — `Completed`.

#### 📐 Диаграмма: пирамида тестов и два гейта

```mermaid
flowchart TD
    subgraph Pyramid["Пирамида тестов"]
        U["L1 Unit + Contract<br/>core / api-contract<br/>pnpm -r test · мгновенно"]
        I["L2 Integration<br/>реальная БД · core/db/client<br/>pnpm test:db · секунды"]
        A["L3 CLI acceptance<br/>seed + handles · --json<br/>given-when-then"]
        E["L4 E2E browser<br/>apps/web/playwright<br/>beta-стенд · минуты"]
    end
    U --> I --> A --> E
    GATE["pnpm checks<br/>lint · typecheck · test · test:db · web build"]
    U -.-> GATE
    I -.-> GATE
    GATE -->|"всё зелёное"| EV["Evidence-блок<br/>what/where/how + дата"]
    A -->|"сценарий прогнан"| EV
    E -->|"только local / beta,<br/>прод исключён"| EV
    EV --> DONE["FT = Completed"]
```

#### 🏗️ Что построить (REBUILD_ROOT)

Доводите слой проверки до того состояния, при котором «готово» = «доказано» (детали — [[06-part-5-technical-restoration#Глава 13: Testing Strategy & Quality Gates|Справочник: Глава 13]] и [[07-part-6-curriculum#Module 7: Hardening и quality gates|Справочник: Module 7]]):

1. **Leaf-скрипты в каждом пакете.** В `packages/core/package.json`: `test` = `cross-env FEEDBACK360_SKIP_DB_TESTS=1 vitest run` (level 1, без БД); `test:db` = `vitest run --testTimeout=45000 --maxWorkers=1 --no-file-parallelism` по трём curated-файлам (level 2 fast-lane); `test:db:full` — полный свип `src/ft/*.test.ts` минус `*-no-db.test.ts` и `ft-0011-op-errors.test.ts`.
2. **Композиция в root `package.json`.** Соберите gate как композицию листьев: `checks` = `pnpm lint && pnpm typecheck && pnpm test && pnpm test:db && pnpm --filter @feedback-360/web build` — ровно в этом порядке. `test:db` агрегирует `@feedback-360/db`, `@feedback-360/client`, `@feedback-360/core`.
3. **E2E-уровень.** Заведите `apps/web/playwright` (34 `ft-*.spec.ts` плюс lane `tests/smoke/*-beta.spec.ts`) и пакет-раннер `@feedback-360/xe-runner` (`@playwright/test`, `vitest run --testTimeout=30000`).
4. **Структурный guard окружений.** В `packages/db/src/xe.ts` — `ensureXeAllowedEnvironment(environment)`: возвращает env только для `"local"` / `"beta"`, иначе бросает `forbidden`. Прод исключён из XE-операций на уровне кода (концепция 11).
5. **Evidence policy.** В `verification-matrix.md` сделайте SSoT «что именно запускать» для каждого `FT`/`GS` и обязательную секцию `### EP-XXX execution evidence (YYYY-MM-DD)` с полями `what/where/how/quality_gate/acceptance_gate/result`. `EP-009` (`FT-0091..FT-0094`) — эталонный hardening-эпик.
6. **Дисциплина процесса.** Маленький слайс хардненинга держите слайсом (см. `post-ep023-hardening-backlog.md`): правило эскалации — если разрослось за 4–5 слайсов или нужны новые acceptance-правила/CI-lane, повышайте до полного эпика, иначе не усложняйте процесс.
7. **3 окружения + post-deploy verification (концепция 11 — выполнимая процедура, не только код-guard).** Объявите три окружения ключами `.env`: `DATABASE_URL` = local, `SUPABASE_BETA_DB_POOLER_URL` = beta, `SUPABASE_PROD_DB_POOLER_URL` = prod. После деплоя на beta — smoke против развёрнутого URL:

```bash
PLAYWRIGHT_BASE_URL=https://beta.go360go.ru pnpm --filter @feedback-360/web test:smoke:beta
# apps/web/package.json → playwright --config playwright/playwright.config.mjs tests/smoke; CI: .github/workflows/beta-smoke.yml
```

   Критерий выхода: beta-smoke зелёный (`N passed`) ДО любого касания prod; прод агентами/XE — никогда (guard `ensureXeAllowedEnvironment` + ручная дисциплина). Полный runbook — `.memory-bank/spec/operations/runbook.md`.
8. **Simplify & verify-against-plan pass (концепция 10 — отдельный проход перед `Completed`).** Триггер: gate зелёный, но фича ещё не `Completed`. Шаги: (1) сверь реализацию с планом/spec построчно — нет ли недовыполнения или «лишних» абстракций «на всякий случай»; (2) вычисти абстракции глубже 2 уровней; (3) перепрогони `pnpm checks`. Копируемый промпт: «Сверь реализацию FT-XXXX с её планом и spec построчно: отметь (а) пункты плана без реализации, (б) слои/абстракции, не оправданные ни одним требованием; предложи удаления, ничего не добавляй.» Критерий выхода: каждый пункт плана покрыт, лишних слоёв нет, `checks` всё ещё зелёный.

#### 📖 Что прочитать (REFERENCE_ROOT)

- [[06-part-5-technical-restoration#Глава 13: Testing Strategy & Quality Gates]] — что лежит: канонический референс стадии, таблица пирамиды из 4 уровней и `pnpm checks` построчно; зачем: связать пирамиду и единый gate с реальными скриптами `package.json`.
- [[07-part-6-curriculum#Module 7: Hardening и quality gates]] — что лежит: учебное определение «готово» (merge-able ≠ verified) и какие артефакты обязан произвести читатель; зачем: понять curriculum-level definition of done.
- [[10-part-9-author-quotes#9.6. О тестах как «гигиене»]] — что лежит: авторская рамка «тесты = гигиена» + цитаты Транскрипт-1; зачем: понять, почему gate **не обсуждается** (концепция 9).
- [[10-part-9-author-quotes#9.8. О stage окружениях]] — что лежит: обоснование изоляции `local`/`beta`/`prod` и цитата про прод; зачем: операционная безопасность как отдельная дисциплина (концепция 11), пара к guard в `xe.ts`.
- [[10-part-9-author-quotes#9.9. О Code Review и simplification]] — что лежит: simplify против over-engineering, Opus «срезает углы»; зачем: упрощение как отдельная фаза ревью (концепция 10).
- `.memory-bank/plans/verification-matrix.md` — что лежит: SSoT `FT`→тест + Evidence policy, эпик `EP-009`; зачем: увидеть 4 уровня тестов в реальных evidence-записях.

#### ✅ Верификация (стадия пройдена, когда)

Запускаемый критерий — это и есть **evidence**, а не «вроде зелёное»:

```bash
# Given: слайс реализован, leaf-скрипты на месте
pnpm -r test          # L1 unit/contract: в core это FEEDBACK360_SKIP_DB_TESTS=1 vitest run — БД не нужна
pnpm test:db          # L2 fast-lane: db + client + core (ft-0042, ft-0055, ft-0071), single worker, 45s timeout
                      # требует достижимый DB URL (beta pooler), иначе integration-subtest'ы skip

# When: гоняем единый quality gate
pnpm checks           # lint && typecheck && test && test:db && web build — последовательно

# Then: gate целиком зелёный + прогнан acceptance на seed/handles
pnpm --filter @feedback-360/cli exec tsx src/index.ts -- --scenario S8_campaign_ended --json
```

Числовые инварианты (берутся из домена, не выдумываются): расчёт итога идёт на весах `managerWeight=40 / peersWeight=30 / subordinatesWeight=30 / selfWeight=0`; порог анонимности — `3`: группа с `< 3` оценщиками по умолчанию не показывается (`small_group_policy=hide`), и только при политике `merge_to_other` peers + subordinates сливаются в `Other` с порогом по объединённой группе; `manager` всегда не-анонимный, `self` имеет вес 0%.

Чеклист:

- [ ] `pnpm checks` зелёный **целиком** — ни один из пяти шагов (`lint` / `typecheck` / `test` / `test:db` / web build) не упал.
- [ ] Acceptance прогнан **после** реализации на стабильных seed-handles через `--json`, а не на id, подсмотренных в базе руками.
- [ ] Все 4 уровня пирамиды покрыты для слайса: unit + contract (L1), integration с реальной БД (L2), CLI-acceptance (L3), e2e на beta (L4).
- [ ] `ensureXeAllowedEnvironment` отвергает любой env, кроме `local`/`beta` — прод структурно недоступен для XE.
- [ ] В `verification-matrix.md` есть строка evidence: `what/where/how/quality_gate/acceptance_gate/result` + дата.

Given-when-then: **Given** `pnpm checks` зелёный и `FT` помечен `Completed`, **When** вы открываете эпик в `verification-matrix.md`, **Then** там есть `### EP-XXX execution evidence (...)` с командами и результатом — без этого блока фича **не** считается завершённой.

#### 🔗 Выравнивание с автором

Концепции 7, 8, 9, 10, 11 из [[#Выравнивание с методологией автора (DEKSDEN, части 1–2)]]. Якоря: «оцениваю как необходимую гигиену. Примерно как руки перед едой.» (Заметки автора, часть 1, тема 7); «пускать агента к проду — это всё-таки стать героем новостей очередных, что мне агент там стёр какую-то базу.» (Заметки автора, часть 1, тема 9). Дополнительно: «вершиной всей проверки является, по мне, сценарий, acceptance test, который тестирует вертикальный слайс.» (тема 7) и «упрощение дает нам экономию внимания модели.» (тема 8).

#### ⚠️ Антипаттерны стадии · 🧩 Артефакты

- Антипаттерны: считать feature готовой без evidence — `FT` `Completed` без блока доказательств, «закодировал, типы проходят, в браузере работает» ([[08-part-7-antipatterns#Антипаттерн 7: Считать feature готовой без evidence]]); acceptance без seed/handles — проверять «по id в базе руками» вместо стабильных handles, что делает приёмку невоспроизводимой ([[08-part-7-antipatterns#Антипаттерн 5: Acceptance без seed/handles]]); пускать агентские/XE-операции к проду в обход guard; over-engineering — оставлять лишние абстракции, которые модель добавила «на всякий случай».
- Артефакты: зелёный вывод `pnpm checks` + строка evidence в `verification-matrix.md` (`what/where/how/quality_gate/acceptance_gate/result` + дата); `package.json` root (`test` / `test:db` / `test:db:full` / gate `checks`); `packages/core/package.json` (unit `FEEDBACK360_SKIP_DB_TESTS=1` vs `--no-file-parallelism --maxWorkers=1 --testTimeout=45000`); `packages/db/src/xe.ts` (`ensureXeAllowedEnvironment` — только `local`/`beta`); `apps/web/playwright/` (34 `ft-*.spec.ts` e2e + `tests/smoke/*-beta.spec.ts`); `packages/xe-runner/package.json` (Playwright-раннер, 30s timeout); `post-ep023-hardening-backlog.md` (пример маленького закрытого слайса + правило эскалации против over-engineering процесса). Примечание: `packages/testkit/src/index.ts` пока заглушка (`export const testkitReady = true;`) — общих тест-хелперов в нём ещё нет, это scaffold.

---

### Стадия 10 — Зрелость: feature-area refactor, XE, guides, traceability

> Исход: structural refactor в feature areas (docs-first + regression-evidence), XE-сценарии, Diataxis guides, UI traceability, design system.

#### 🎯 Зачем эта стадия (на пальцах)

Представьте, что вы построили большой дом по одному чертежу-«слою»: сначала по всему дому проложили проводку, потом по всему дому трубы, потом везде стены. Пока дом маленький — терпимо. Но когда комнат становится много, файл «вся проводка» разрастается до сотен строк, и чтобы понять, что относится к кухне, приходится читать про весь дом. Layer-flat организация (`index.ts` на 500+ строк) — ровно такой случай: она удобна на старте и **мешает** на зрелости. Стадия 10 — это **запланированная эволюция**, а не починка ошибки: вы фиксируете границы feature-area в ADR, потом переносите код по владельцу так, чтобы каждый файл «принадлежал» конкретной области, а не «болтался». Ключевая дисциплина — **structural refactor не меняет public behavior**: operation names, DTO-формы и бизнес-поведение остаются прежними, доказательством служат regression-тесты, а не «вроде ничего не сломалось».

Рамка от автора: зрелый проект — это не только код и тесты, но и **сценарный runtime** (cross-epic XE), **операторские guides** (Diataxis) и **строгая UI traceability** (screen IDs). Важнейший message стадии: XE и traceability — **поздняя зрелость, а не стартовая точка**. Их добавляют тогда, когда есть что трассировать и что прогонять целиком, а не на первом коммите.

#### 🧠 Ключевые концепции простыми словами

- **Feature-area slicing (docs-first).** Суть: код режется не по слоям, а по **областям владения**, и сначала границы фиксируются в ADR, потом переносится код. ADR 0004 канонически перечисляет **9 областей** (`identity-tenancy`, `org`, `models`, `campaigns`, `matrix`, `questionnaires`, `results`, `notifications`, `ai`); `ops` присутствует в коде как 10-й модуль `features/ops` и упомянут в rationale ADR, но в канонический список **не вынесен** — то есть в `features/` реально 10 модулей (расхождение docs↔code, которое стоит держать в голове). Аналогия: переезд по комнатам с описью «что чьё» до того, как таскать коробки. Почему важно: god-files ухудшают локальность — рефактор по ownership возвращает «каждый файл связан с фичей».
- **Thin composition point.** Суть: корневой `index.ts` после рефактора — **тонкая точка сборки** (~295 строк): импортирует run-функции из `features/*` и держит `operationHandlers` (таблицу `operation-name → handler`), но логики в нём нет. Аналогия: щиток в подъезде только разводит провода, а сами приборы — в квартирах. Почему важно: root остаётся entrypoint-ом, но перестаёт быть свалкой.
- **Regression-evidence для рефактора.** Суть: structural-перенос считается «done» только когда зелёные `ft-0142` (layout-regression в `api-contract`/`client`/`core`/`cli`) и `ft-0143` (Playwright slice-refactor-regression), а также пройден полный quality gate. Аналогия: после переноса мебели проверяешь, что вода и свет всё ещё работают во всех комнатах. Почему важно: рефактор без evidence — это «вроде работает», а не доказательство, что public behavior сохранён.
- **XE-сценарии (cross-epic, executable).** Суть: декларативный сценарий (`scenario.json`) описывает фазы, а runner их исполняет — seed данных и notification/bootstrap делает **runner**, не человек в браузере. Аналогия: записанный N2N-прогон вместо «походи руками, авось найдёшь баг». Почему важно: повторяемость — ошибка, найденная агентом вручную, считается провалом тест-системы.
- **UI traceability через screen IDs.** Суть: каждый экран получает стабильный `SCR-*` идентификатор, который протягивается через frontmatter спеков, guide-frontmatter (`screen_ids`), JSDoc `@screenId`, `data-testid` и имена скриншотов. Аналогия: инвентарный номер на каждой двери — по нему всегда найдёшь и тест, и документацию. Почему важно: без stable identifiers selector-ы привязываются к Tailwind-классам и ломаются от любого изменения стиля.

#### 🔧 Иллюстративный пример: golden run XE-001 поверх feature-area кода

Сценарий **XE-001 «First 360 campaign happy path»** — это полный цикл 360 поверх отрефакторенного кода, и он отлично показывает обе половины стадии сразу.

1. **`phase-01-seed`** (`runXe001PhaseSeed`): `applyXeSeed('XE-001-first-campaign')` поднимает изолированную компанию, пишет `seed-summary.json`, фиксирует bindings (company/campaign/actors).
2. **`phase-02-start-campaign`**: `hrClient.campaignStart` — вызывает ту же operation `campaign.start`, что и раньше: рефактор перенёс её в `features/campaigns`, но имя и DTO не изменились.
3. **`phase-03-bootstrap-sessions`**: runner сам выдаёт storage-state каждому из 8 актёров (`subject`, `manager`, `peer_1..3`, `subordinate_1..3`) — человек в UI это **не** делает.
4. **`phase-04-fill-questionnaires`**: по `fixtures/answers.json` каждый актёр делает `questionnaireSaveDraft` + `questionnaireSubmit`, затем `campaignProgressGet`.
5. **`phase-05-verify-results`**: `campaignEnd` + `aiRunForCampaign`, потом проверка результатов для employee/manager/HR с `anonymityThreshold: 3` и `smallGroupPolicy: "merge_to_other"`; скриншоты `employee/manager/hr-results.png` снимаются через POM, который опирается на стабильные `data-testid` (`results-summary`), а traceability к `SCR-RESULTS-*` задаётся через `screen_ids` во frontmatter гайда — не как селекторы внутри POM.

Анонимность здесь — наблюдаемый инвариант: `openText.rawText` равен `undefined` для employee и manager (порог 3 не достигнут на их срезе), но **present** для HR. Это и есть та бизнес-логика, которую рефактор обязан был сохранить, а XE — продемонстрировать.

#### 📐 Диаграмма: data-driven XE-прогон поверх thin core

```mermaid
flowchart LR
    SJ["scenario.json<br/>5 фаз (data)"] --> REG["scenario-registry.ts<br/>parseScenarioDefinition"]
    REG --> RUN["runner.ts<br/>acquireXeEnvironmentLock"]
    RUN --> PH["phaseHandlers<br/>xe-001 phase-01..05"]
    PH --> CORE["core/src/index.ts<br/>thin composition ~295"]
    CORE --> FA["features/*<br/>10 модулей (9 в ADR + ops)"]
    PH -->|"writeXeState +<br/>assertions.json ok:true/false"| ART["артефакты .xe-runs<br/>results.png (SCR-*)"]
    RUN -.->|"finally"| REL["releaseXeEnvironmentLock"]
```

#### 🏗️ Что построить (REBUILD_ROOT)

Порядок — строго **docs → code → evidence** (детали — [[06-part-5-technical-restoration#Глава 14: Feature-Area Refactor]] и [[06-part-5-technical-restoration#Глава 15: XE Scenarios, Guides, UI Traceability]]):

1. **ADR границ.** `.memory-bank/adr/0004-feature-area-slicing-boundaries.md`: перечислите canonical feature areas (в ADR их **9** — `ops` пока в каноническом списке нет, он живёт лишь в rationale-прозе и как код-модуль), зафиксируйте guardrail «сначала docs/ADR, потом перенос кода» и правило `shared` (только для модулей без явного owner — `context`, `audit`).
2. **Перенос по ownership.** `packages/core/src/features/{identity-tenancy,org,models,campaigns,matrix,questionnaires,results,notifications,ai,ops}` — **9 канонических областей в ADR + реально 10 модулей под `features/` (включая `ops`)** — плюс `packages/core/src/shared/`. Корневой `packages/core/src/index.ts` оставьте **thin composition point** (~295 строк): импорты run-функций + `operationHandlers: Partial<Record<KnownOperation, OperationHandler>>` + `dispatchOperation`; public operation names и DTO-типы из `@feedback-360/api-contract` **не трогать**.
3. **Regression-evidence.** Добавьте `ft-0142-*` layout-тесты в `api-contract`/`client`/`core`/`cli` и `apps/web/playwright/tests/ft-0143-slice-refactor-regression.spec.ts`.
4. **XE-runner.** `packages/xe-runner/src/{runner.ts,scenario-registry.ts}` + `scenarios/xe-001.ts` + POM `pom/results.ts`; декларативный каркас в `scenarios/XE-001/scenario.json` (5 фаз, `requiredArtifacts`, `phasePolicy.defaultFailurePolicy: "fail_run"`).
5. **Guides + traceability.** `.memory-bank/guides/{tutorials,how-to,explanation,reference}/*` по Diataxis; протяните `SCR-*` через frontmatter (`screen_ids` в `explanation/xe-001-walkthrough.md`), `data-testid` и имена скриншотов.

#### 📖 Что прочитать (REFERENCE_ROOT)

- [[06-part-5-technical-restoration#Глава 14: Feature-Area Refactor]] — что лежит: переход layer-flat → feature-area, схема «до/после», порядок рефакторинга и checkpoint; зачем: это главный справочник docs-first рефактора (его «~295 строк» совпадают с реальными 295 строками `core/src/index.ts` — `awk NR=295`).
- [[06-part-5-technical-restoration#Глава 15: XE Scenarios, Guides, UI Traceability]] — что лежит: XE-цикл XE-001, Diataxis guides с реальными путями, механика `screen_id`/`data-testid`; зачем: справочник по XE/guides/UI-traceability.
- [[07-part-6-curriculum#Module 8: Feature-area refactor]] — что лежит: учебный модуль refactor (ADR rationale → target areas → перенос по ownership); зачем: критерий «если ownership всё ещё непонятен — refactor не удался».
- [[07-part-6-curriculum#Module 9: XE, guides, UI traceability]] — что лежит: учебный модуль XE/guides/traceability; зачем: явно фиксирует «не стартовая точка, а поздняя зрелость».
- `.memory-bank/adr/0004-feature-area-slicing-boundaries.md` — зачем: первоисточник границ и guardrail-ов («structural refactor не должен менять public operation names, DTO shapes и behavior без отдельного решения»).
- `scenarios/XE-001/scenario.json` и `.memory-bank/guides/explanation/xe-001-walkthrough.md` — зачем: связка data ↔ loader ↔ executor и реальные `SCR-*` traceability в frontmatter.

#### ✅ Верификация (стадия пройдена, когда)

Запускаемый критерий (это и есть **evidence**, а не «вроде работает»):

```bash
# Given: docs-first рефактор завершён, XE-001 заведён
# When: проверяем, что public behavior сохранён во всех слоях
pnpm --filter @feedback-360/core test    # включает ft-0142-feature-layout-no-db.test.ts — зелёный
pnpm -r test                             # ft-0142 regression в api-contract/client/core/cli — зелёный
pnpm checks                              # lint + typecheck + test + test:db + web build — зелёный

# Then: фазы XE-001 декларативны и совпадают с тем, что исполняет runner
node -e "const f=require('fs');const j=JSON.parse(f.readFileSync('scenarios/XE-001/scenario.json','utf8'));console.log(j.phases.map(p=>p.phaseId).join(','))"
# phase-01-seed,phase-02-start-campaign,phase-03-bootstrap-sessions,phase-04-fill-questionnaires,phase-05-verify-results
```

Чеклист:

- [ ] `packages/core/src/index.ts` — thin composition (~295 строк): только импорты `features/*` + `operationHandlers` + `dispatchOperation`, бизнес-логики нет.
- [ ] Вы можете назвать canonical feature areas (в ADR их 9), а `shared/` обоснован (только модули без явного owner).
- [ ] `ft-0142` (4 пакета) и `ft-0143` зелёные — public operation names и DTO behavior не изменились.
- [ ] `pnpm --filter @feedback-360/xe-runner test` проходит; XE-001 берёт environment lock и при двойном запуске бросает `invalid_transition`, повторный прогон **idempotent** (passed-фазы пропускаются).
- [ ] Анонимность по факту: `openText.rawText` = `undefined` для employee/manager и present для HR при `anonymityThreshold: 3`, `smallGroupPolicy: "merge_to_other"` (XE-сценарий явно переопределяет дефолтный `"hide"`).
- [ ] Каждый экран имеет `SCR-*`, протянутый в `data-testid` и имена скриншотов; selector-ы **не** опираются на Tailwind-классы.
- [ ] Evidence-блок записан: дата + команды + результаты.

Given-when-then: **Given** другой XE-run уже активен, **When** вы стартуете XE-001 повторно, **Then** runner бросает `invalid_transition` («Another XE run is already active»), а в `finally` гарантированно делает `releaseXeEnvironmentLock` — lock не «протекает».

#### 🔗 Выравнивание с автором

Концепции 16, 20 из [[#Выравнивание с методологией автора (DEKSDEN, части 1–2)]]. Якоря: «Каждый файл кода связан с конкретной фичей, а не «болтается».» (часть 2, тема 4 — концепция 16, трассировка намерение→реализация); «Предпочитайте классические **N2N-тесты** «гуляющему» AI-QA ради повторяемости; ошибка, найденная агентом вручную, — это провал тест-системы.» (часть 2, тема 10 — концепция 20, XE). Тематически рядом стоят концепция 17 (пирамида знаний: «в коде — как, в документации — почему/для чего») и концепция 22 (reverse-engineering, мета), но их основной якорь — на Стадиях 0/2 и в мета-слое, поэтому здесь они идут без отдельной цитаты.

#### ⚠️ Антипаттерны стадии · 🧩 Артефакты

- Антипаттерны: считать feature/рефактор готовым без evidence — статус Completed без прогона `pnpm checks` и regression-сценария ([[08-part-7-antipatterns#Антипаттерн 7: Считать feature готовой без evidence]]); UI без stable identifiers — Tailwind-классы как Playwright-селекторы (`page.locator('.bg-blue-500')`), из-за чего agentic-проверки и скриншоты невоспроизводимы ([[08-part-7-antipatterns#Антипаттерн 8: UI без stable identifiers]]).
- Артефакты: `adr/0004-feature-area-slicing-boundaries.md`; thin `core/src/index.ts` + `features/` (9 областей в ADR + 10-й модуль `ops`) + `shared/{context,audit}`; `xe-runner/src/{runner.ts,scenario-registry.ts,scenarios/xe-001.ts,pom/results.ts}`; `scenarios/XE-001/{scenario.json,scenario.md,fixtures/answers.json,fixtures/expected-results.json}`; Diataxis-guides `.memory-bank/guides/{tutorials,how-to,explanation,reference}/*`; regression `ft-0142-*` (api-contract/client/core/cli) и `ft-0143-slice-refactor-regression.spec.ts`.

---

## Выравнивание с методологией автора (DEKSDEN, части 1–2)

Эта таблица — **карта соответствия**: 24 концепции из заметок автора → где они в стадийном пути и справочниках → цитата/тема-якорь у автора. Источник истины — `_articles/…DEKSDEN часть 1/2_best.notes.md`.

| # | Концепция автора | Стадия | Справочник / секция | Якорь у автора |
|---|---|---|---|---|
| 1 | Полный цикл, другой исполнитель | 0 | Часть II 2.8 | «цикл не изменился — изменился исполнитель» (ч1·т1) |
| 2 | Прогрев + специализация контекста | 0, 2 | Часть II 2.7; Часть IV фаза 4 | «работаю только с прогретыми контекстами» (ч1·т2) |
| 3 | Gap closing + встречные вопросы | 0 | Часть II 2.6 | «галлюцинации — не баг, а то, ради чего юзаем ИИ» (ч2·т2) |
| 4 | Позиция токенов; Escape-Escape; Obsidian | 0 | Часть II 2.5; Часть IX 9.1 | «каждый токен в начале значит немножко больше» (ч1·т3) |
| 5 | Спецификация = шаги + grounding | 0, 2 | Часть IV фаза 2; Часть V гл.0 | «спецификация = набор шагов + грудинг» (ч1·т4) |
| 6 | Вертикальные слайсы | 4, 5 | Часть II 2.1(3); Часть IX 9.4 | «единица ценности через все слои» (ч1·т4) |
| 7 | Acceptance given-when-then, покрывает всю хотелку | 4, 5, 6, 7, 9 | Часть IX 9.5; антипаттерн 5 | «acceptance test тестирует вертикальный слайс» (ч1·т7) |
| 8 | Happy Path + error в свежем контексте; оркестратор | 9 | Стадия 9 (🧠 happy/error) + прим. ниже | «из одного контекста агент не видит все ошибки» (ч1·т6) |
| 9 | Верификация как гигиена; пирамида тестов | 7, 9 | Часть V гл13; Часть IX 9.6 | «гигиена, как руки перед едой» (ч1·т7) |
| 10 | Цикл под управлением; Opus «срезает углы»; simplify | 9 | Часть IX 9.9, 9.15, 9.17 | «упрощение даёт экономию внимания модели» (ч1·т8) |
| 11 | 3 окружения local/beta/prod; верификация деплоя | 9 | Часть IX 9.8 | «пускать агента в прод — стать героем новостей» (ч1·т9) |
| 12 | CLI вместо GUI; SDK→CLI→GUI | 4, 8 | Часть II 2.1(4); Часть V гл7,12; Справочник A §3.10 | «модель с CLI работает легко, непринуждённо» (ч1·т11–12) |
| 13 | Memory Bank + MBB principles | 2 | Часть II 2.1(2); Часть V гл2; Appendix G | «MemoryBank — инструмент формирования контекста агента» (ч2·т3) |
| 14 | Annotated links «что + зачем» | 2 | Часть V гл2; Часть IX 9.11 | «добавишь „зачем“ — шанс, что прочитает, растёт» (ч2·т3) |
| 15 | Структура Memory Bank: spec/plans/домен; Epic/Feature | 2, 5 | Часть II 2.1(2); Часть IV фаза 3 | «Epic ≈ roadmap + FrontMatter; Feature = min delivery» (ч2·т4) |
| 16 | Трассировка намерение→реализация | 2, 10 | Часть II 2.1(5); Часть V гл15 | «каждый файл кода связан с фичей» (ч2·т4) |
| 17 | Пирамида знаний: код=как, доки=зачем | 0, 2 | Часть II 2.4; Часть IX 9.13 | «в коде — как, в документации — почему/для чего» (ч2·т5) |
| 18 | Progressive Disclosure | 2 | Appendix G; MBB principles | «добор контекста кусочками» (ч2·т8) |
| 19 | QA: Page Object Model + `data-testid` | 8 | Часть V гл15; антипаттерн 8 | «стабильный id → тесты бегают надёжно» (ч2·т9) |
| 20 | N2N vs «гуляющий» AI-QA | 8, 9 | Часть IX 9.7 | «ошибка, найденная агентом вручную — провал тест-системы» (ч2·т10) |
| 21 | Skill + CLI вместо «толстого» MCP | *(мета)* | прим. ниже | «MCP не даёт никакой дополнительной ценности» (ч2·т11) |
| 22 | Reverse-engineering в обобщённый скилл | *(мета)* | прим. ниже | «у нас уже есть результат — промпт выводим из него» (ч2·т7) |
| 23 | Язык/токены; файлы > таск-трекеры | *(мета)* | прим. ниже | «для агента сильно удобнее писать файлик» (ч2·т8,12) |
| 24 | Доменная специфика feedback-360 (анонимность `<3`, роли, кампании) | 5, 6 | Часть III; Часть V гл9–11 | «<2 ответов — группа не показывается» (ч2·т4) — *автор оговорился «меньше двух»; код реализует порог 3 (ADR-0002, `results.ts`), см. [[#Стадия 6 — Policy-heavy домен]]* |

**Примечание о мета-концепциях (21–23).** Это концепции **уровня личного workflow автора**, не специфичные для самого feedback-360: они про инструментарий (MCP→Skill+CLI через MCPorter, reverse-engineering проекта в обобщённый скилл, выбор «файлы vs таск-трекеры», русский промпт / английский код). Для **воспроизведения проекта** они не обязательны, но важны для понимания, *как автор работает*. Если вы строите свой агентный workflow поверх этого rebuild — держите их в виду; в самом маршруте 0–10 они намеренно не превращены в стадии.

**Примечание о концепции 8 (error-ветки в свежем контексте).** Автор подчёркивает: happy path и error-ветки лучше прорабатывать в **отдельных свежих контекстах** (или через оркестратор), потому что «happy path в контексте слепит агента к ошибкам». В стадийном пути это **прожито** как отдельная концепция и мини-шаг в [[#Стадия 9 — Hardening, quality gates, evidence|Стадии 9]] (на каждый typed-код ошибки — отдельный acceptance в свежем контексте) и в acceptance-практике Стадий 5–6. Полноценная **оркестрация** (под-агенты, авто-разветвление контекстов) — за рамками rebuild, но сам приём разделения контекстов воспроизводим вручную.