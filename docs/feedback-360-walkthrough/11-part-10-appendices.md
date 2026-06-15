## ЧАСТЬ X — Приложения

> 🧭 [[feedback-360-master-rebuild-walkthrough|Master Rebuild Walkthrough — оглавление]]


> 📚 **Приложения (Reference).** Это не маршрут, а справочные приложения (reading order, каталоги, чеклисты). Открываются по ссылкам из стадий и других частей.

### Appendix A — Reading order для оригинального проекта

Если вы хотите понять подход быстро, а не читать repo хаотично, идите так:

1. [`.memory-bank/index.md`](../.memory-bank/index.md).
2. [`.memory-bank/spec/project/system-overview.md`](../.memory-bank/spec/project/system-overview.md) (или ближайший аналог).
3. [`.memory-bank/spec/project/layers-and-vertical-slices.md`](../.memory-bank/spec/project/layers-and-vertical-slices.md).
4. [`.memory-bank/spec/engineering/architecture-guardrails.md`](../.memory-bank/spec/engineering/architecture-guardrails.md) (если есть).
5. [`.memory-bank/plans/implementation-playbook.md`](../.memory-bank/plans/implementation-playbook.md).
6. [`.memory-bank/adr/0001-core-client-cli-first.md`](../.memory-bank/adr/0001-core-client-cli-first.md).
7. [`.memory-bank/adr/0004-feature-area-slicing-boundaries.md`](../.memory-bank/adr/0004-feature-area-slicing-boundaries.md).
8. [`packages/api-contract/src/campaigns.ts`](../packages/api-contract/src/campaigns.ts) (или другая feature по выбору).
9. [`packages/client/src/features/campaigns.ts`](../packages/client/src/features/campaigns.ts).
10. [`packages/core/src/features/campaigns.ts`](../packages/core/src/features/campaigns.ts).
11. [`packages/db/src/campaigns.ts`](../packages/db/src/campaigns.ts).
12. [`apps/web/src/app/api/hr/campaigns/draft/route.ts`](../apps/web/src/app/api/hr/campaigns/draft/route.ts).
13. `git log --reverse --oneline` — увидеть эволюцию.
14. Транскрипты из [`_articles/`](../_articles/) — сопоставить repo reality с прямыми словами автора.

### Appendix B — Каталог 50+ операций по feature areas

#### Identity & Tenancy (4)

- `system.ping` — health check.
- `company.updateProfile` — обновление профиля компании (hr_admin).
- `membership.list` — список компаний пользователя.
- `identity.provisionAccess` — привязка user↔employee (hr_admin).

#### Organization (6)

- `employee.upsert`.
- `employee.listActive`.
- `employee.directoryList`.
- `employee.profileGet`.
- `department.list`.
- `department.upsert`.
- `org.department.move`.
- `org.manager.set`.

#### Models (6)

- `model.version.create`.
- `model.version.list`.
- `model.version.get`.
- `model.version.cloneDraft`.
- `model.version.upsertDraft`.
- `model.version.publish`.

#### Campaigns (13)

- `campaign.create`.
- `campaign.list`.
- `campaign.get`.
- `campaign.updateDraft`.
- `campaign.start`.
- `campaign.stop`.
- `campaign.end`.
- `campaign.setModelVersion`.
- `campaign.weights.set`.
- `campaign.participants.add`.
- `campaign.participants.remove`.
- `campaign.participants.addFromDepartments`.
- `campaign.snapshot.list`.
- `campaign.progress.get`.

#### Matrix (3)

- `matrix.list`.
- `matrix.generateSuggested`.
- `matrix.set`.

#### Questionnaires (4)

- `questionnaire.listAssigned`.
- `questionnaire.getDraft`.
- `questionnaire.saveDraft`.
- `questionnaire.submit`.

#### Results (3)

- `results.getHrView`.
- `results.getMyDashboard`.
- `results.getTeamDashboard`.

#### Notifications (8)

- `notifications.generateReminders`.
- `notifications.dispatchOutbox`.
- `notifications.settings.get`.
- `notifications.settings.upsert`.
- `notifications.preview`.
- `notifications.templates.list`.
- `notifications.templates.preview`.
- `notifications.deliveries.list`.

#### AI (1)

- `ai.runForCampaign`.

#### Ops (3)

- `ops.health.get`.
- `ops.aiDiagnostics.list`.
- `ops.audit.list`.

#### Client-local (2)

- `client.setActiveCompany`.
- `seed.run`.

**Итого: 53 операции** через единый dispatcher.

### Appendix C — Ключевые ADR

| ADR | Решение | Причина | Trade-off |
|---|---|---|---|
| 0001 | Core + Typed Client + CLI-first | UI и CLI остаются thin; логику тестируем без браузера | Больше дисциплины на старте |
| 0002 | Anonymity threshold = 3 | Баланс между информативностью и защитой анонимности | Малые компании теряют детализацию |
| 0003 | Freeze on first draft save | Матрица и веса фиксируются после начала работы оценщиков | Нельзя «подкрутить» в процессе |
| 0004 | Feature-area slicing | Root files перестали быть локальными; ownership размывался | Массовый перенос путей и imports |

### Appendix D — Dependency graph пакетов

Полная Mermaid-диаграмма — в [[04-part-3-architecture#3.3. Dependency graph пакетов|Часть III §3.3]].

**Правила:**

- Стрелки идут **только вниз**.
- `web/cli` импортируют `client`, **не** `core` напрямую.
- `client` зависит от `api-contract` и (в in-proc режиме) использует `core` через transport.
- `core` зависит от `api-contract` и `db`.
- `db` зависит от `api-contract` для type-safe сигнатур.

Эти правила формализованы в `.memory-bank/spec/engineering/architecture-guardrails.md`.

### Appendix E — Технологический стек с версиями

| Слой | Технология | Версия |
|---|---|---|
| Язык | TypeScript | 5.8.x |
| Runtime | Node.js | 20+ |
| Package manager | pnpm | 10.5.0+ |
| Web framework | Next.js | 15 |
| Hosting | Vercel | — |
| Database | PostgreSQL (Supabase pooler) | 15+ |
| ORM | Drizzle ORM | 0.44.5 |
| Migrations | drizzle-kit | 0.31.2 |
| Linter/Formatter | Biome | 1.9.4 |
| Test runner | Vitest | 3.2.4 |
| UI styling | Tailwind | v4 |
| UI components | shadcn/ui | — |
| CLI framework | Commander | — |
| Browser smoke | Playwright | — |
| Observability | Sentry | — |
| Email (MVP) | Resend | — |
| Types | @types/node | 22.13.8 |

### Appendix F — Полный список 18 промптов

Сводный список промптов из Части IV для быстрого доступа:

| # | Фаза | Назначение |
|---|---|---|
| 1 | 0: Идея → Brief | Первичная подача идеи; gap closing |
| 2 | 1: MBB | Создание MBB principles |
| 3 | 1: MBB | Генерация templates (epic, feature) |
| 4 | 2: Specs | Domain specs |
| 5 | 2: Specs | API и subsystem specs |
| 6 | 2: Specs | Testing strategy и traceability |
| 7 | 3: EP/FT | Нарезка на epics |
| 8 | 3: EP/FT | Детализация features |
| 9 | 4: AGENTS.md | AGENTS.md + root index |
| 10 | 5: Scaffold | FT-0001 workspace scaffold |
| 11 | 5: Scaffold | FT-0002 DB baseline |
| 12 | 6: Delivery loop | FT-0011 api-contract + dispatcher |
| 13 | 6: Delivery loop | FT-0012 client transport + runtime |
| 14 | 6: Delivery loop | FT-0013 CLI-first vertical slice |
| 15 | 7: Domain slices | Универсальный шаблон реализации FT |
| 16 | 8: UI | UI foundation |
| 17 | 8: UI | Feature-specific UI page |
| 18a | 9: Hardening | Quality-gate & test-pyramid build-out (→ Стадия 9) |
| 18 | 9: Hardening | Feature-area refactor (→ Стадия 10) |

Полные тексты промптов — в Части IV.

### Appendix G — Связь с MBB principles

17 принципов из `.memory-bank/mbb/principles.md` и где они применяются в этом walkthrough:

| # | Принцип | Где обсуждается |
|---|---|---|
| 1 | SSoT | Часть II §2.1.2, Часть VII (антипаттерн 2) |
| 2 | One file = one concept | Часть II §2.4 (пирамидка знаний) |
| 3 | Progressive disclosure | Глоссарий, оглавление |
| 4 | WHY/WHAT/HOW | Часть II §2.4 |
| 5 | No duplication with code | Часть II §2.4 |
| 6 | Index-first navigation | Часть IV фаза 4, глава 2 Части V |
| 7 | Annotated links | Часть IV §9, глава 2 Части V |
| 8 | Keep documents small | (мета-принцип, не дублируется) |
| 9 | Commit tagging | Чеклист «feature done» |
| 10 | Evidence-first completion | Часть II §2.1.6, чеклист «feature done» |
| 11 | Root index usefulness | Глава 2 Части V |
| 12 | Grounding-first implementation | Промпт #15, чеклист «feature done» |
| 13 | Visual references are inspiration | Глава 15 Части V |
| 14 | Boundary rationale must be documented | ADR-0004, фаза 9, модуль 8 |
| 15 | UI specs and POM mapping first-class | Глава 15 Части V |
| 16 | Screen IDs mandatory | Антипаттерн 8, глава 15 Части V |
| 17 | Design system is SSoT | Глава 15 Части V |

### Appendix H — Чеклист «feature done» (повтор для удобства)

- [ ] FT-документ существует и заполнен.
- [ ] Grounding пройден: записаны прочитанные SSoT docs.
- [ ] Contract обновлён.
- [ ] Core handler реализован по паттерну `parse → auth → company → DB → audit → parse output`.
- [ ] DB-функция + миграция (если нужна).
- [ ] Client method добавлен.
- [ ] CLI команда с dual output.
- [ ] Unit-тесты на политики/расчёты.
- [ ] Integration-тест с реальной БД (seed + handles).
- [ ] `pnpm checks` зелёный.
- [ ] Acceptance scenario прогнан.
- [ ] Evidence-блок записан.
- [ ] Operation catalog обновлён.
- [ ] Screen ID и data-testid зафиксированы (если UI).
- [ ] Коммит таггирован `[FT-XXXX]`.

### Appendix I — Cross-references на companion-документы

#### Companion-документы в `docs/`

- [feedback-360-faithful-rebuild-guide.md](./feedback-360-faithful-rebuild-guide.md) — концептуальный reverse-engineering: что построено и почему. Читать, если хотите глубже понять философию подхода.
- [feedback-360-step-by-step-rebuild.md](./feedback-360-step-by-step-rebuild.md) — технический манул с code-snippets. Читать, если хотите детальнее разобрать конкретные паттерны кода.
- [feedback-360-learning-path.md](./feedback-360-learning-path.md) — curriculum-слой: 9 модулей с правилами обучения. Читать, если хотите альтернативную формулировку Части VI этого файла.
- [feedback-360-agentic-rebuild-process.md](./feedback-360-agentic-rebuild-process.md) — 18 промптов и реконструированный brief. Читать, если нужны промпты в их оригинальной формулировке.

#### YouTube-транскрипты

- [Часть 1 YouTube SCRIPT](../_articles/Agentic%20Engineering%20AI%20Workflow%20с%20DEKSDEN%20часть%201/Agentic%20Engineering%20AI%20Workflow%20с%20DEKSDEN%20часть%201%20YOUTUBE%20SCRIPT.md) — методология, brief, vertical slices, тестирование, agentic test, верификация, code review.
- [Часть 2 YouTube SCRIPT](../_articles/Agentic%20Engineering%20AI%20Workflow%20с%20DEKSDEN%20часть%202/Agentic%20Engineering%20AI%20Workflow%20с%20DEKSDEN%20часть%202%20YOUTUBE%20SCRIPT.md) — Memory Bank как knowledge base, аннотированные ссылки, AGENTS.md как роутинг, «пирамидка знаний».

#### Ключевые точки Memory Bank

- [`.memory-bank/index.md`](../.memory-bank/index.md) — главный entrypoint.
- [`.memory-bank/mbb/index.md`](../.memory-bank/mbb/index.md) — оглавление MBB.
- [`.memory-bank/mbb/principles.md`](../.memory-bank/mbb/principles.md) — 17 принципов.
- [`.memory-bank/adr/index.md`](../.memory-bank/adr/index.md) — каталог ADR.
- [`.memory-bank/adr/0001-core-client-cli-first.md`](../.memory-bank/adr/0001-core-client-cli-first.md).
- [`.memory-bank/adr/0002-anonymity-threshold.md`](../.memory-bank/adr/0002-anonymity-threshold.md).
- [`.memory-bank/adr/0003-freeze-on-draft-save.md`](../.memory-bank/adr/0003-freeze-on-draft-save.md).
- [`.memory-bank/adr/0004-feature-area-slicing-boundaries.md`](../.memory-bank/adr/0004-feature-area-slicing-boundaries.md).
- [`.memory-bank/spec/domain/index.md`](../.memory-bank/spec/domain/index.md) — доменные специации.
- [`.memory-bank/spec/client-api/operations.md`](../.memory-bank/spec/client-api/operations.md) — каталог операций.
- [`.memory-bank/spec/client-api/errors.md`](../.memory-bank/spec/client-api/errors.md) — каталог ошибок.
- [`.memory-bank/spec/testing/test-strategy.md`](../.memory-bank/spec/testing/test-strategy.md).
- [`.memory-bank/spec/testing/golden-scenarios.md`](../.memory-bank/spec/testing/golden-scenarios.md).
- [`.memory-bank/spec/testing/seeds/`](../.memory-bank/spec/testing/seeds/) — каталог seed scenarios.

#### Ключевые файлы кода

- [`packages/api-contract/src/`](../packages/api-contract/) — типы и parse functions.
- [`packages/core/src/index.ts`](../packages/core/) — dispatcher.
- [`packages/client/src/shared/runtime.ts`](../packages/client/) — transport abstraction.
- [`packages/db/src/schema/`](../packages/db/) — Drizzle schema.