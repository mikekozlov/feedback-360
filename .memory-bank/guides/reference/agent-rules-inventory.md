---
description: Derived quick reference for coding agents working in feedback-360, with source precedence and links to canonical rules.
purpose: Read before implementation work to understand which documents govern agent behavior and what rules are non-negotiable.
status: Active
date: 2026-03-19
parent: .memory-bank/guides/reference/index.md
---

# Agent rules inventory
Status: Active (2026-03-19)

Этот документ — сжатая operational map для coding agents, работающих в `feedback-360`.

Важно:
- это производный summary, а не новый SSoT;
- если есть конфликт, выигрывают каноничные источники из `AGENTS.md`, `.memory-bank/spec/`, `.memory-bank/plans/`, `.memory-bank/adr/` и `.memory-bank/mbb/`.

## Порядок источников

- [`AGENTS.md`](../../../AGENTS.md): краткий repo-level summary системы, ограничений и базовых architectural rules. Читать первым, чтобы быстро понять общий контекст и не нарушить ключевые границы.
- [Memory Bank index](../../index.md): curated карта SSoT-источников по проекту. Читать сразу после `AGENTS.md`, чтобы быстро найти нужный каноничный документ без “поиска по памяти”.
- [Specifications (`spec/`)](../../spec/index.md): нормативные требования к поведению системы, безопасности, тестам, UI и операциям. Читать всегда, когда нужно понять, как система должна работать.
- [Plans (`plans/`)](../../plans/index.md): порядок реализации, acceptance, verification matrix и implementation playbook. Читать, когда нужно понять, как поставлять изменение и чем подтверждать готовность.
- [ADR (`adr/`)](../../adr/index.md): архитектурные решения и rationale. Читать, когда решение кажется странным и нужно понять, почему проект устроен именно так.
- [Memory Bank Bible (`mbb/`)](../../mbb/index.md): правила документации, индексов, annotated links и evidence-first completion. Читать перед созданием или обновлением документов.

## Каноничный рабочий набор

- [Project structure](../../structure.md): где лежат `apps/`, `packages/`, `.memory-bank/` и как проведены границы слоёв. Читать в начале работы, чтобы сразу класть изменения в правильные места.
- [Architecture guardrails](../../spec/engineering/architecture-guardrails.md): запреты на импорт core в UI/CLI, shape vertical slices и transitional exceptions. Читать перед кодом, чтобы не утащить product logic в тонкие клиенты и не наращивать legacy/shim слои.
- [Coding style](../../spec/engineering/coding-style.md): project-wide coding conventions, формат ошибок и правила для UI/CLI. Читать перед реализацией, чтобы новые изменения были консистентными по типам, форматам и layout кода.
- [Implementation playbook](../../plans/implementation-playbook.md): пошаговый workflow `FT -> contract -> core -> db -> adapters -> client -> cli -> tests -> docs`. Читать как основную рабочую инструкцию для delivery одного vertical slice.
- [Testing standards](../../spec/engineering/testing-standards.md): уровни тестов и правило двух независимых гейтов. Читать перед проверками, чтобы не подменять acceptance обычным test run.
- [Delivery standards](../../spec/engineering/delivery-standards.md): traceability, evidence, quality gate и acceptance gate для статуса `Completed`. Читать перед merge или закрытием FT, чтобы “готово” было подтверждено артефактами.
- [Git flow](../../spec/operations/git-flow.md): branch naming, commit tags `[FT-*]/[EP-*]`, PR rules и release path. Читать до создания ветки и PR, чтобы traceability не приходилось восстанавливать задним числом.

## Ненарушаемые правила

- Значимая бизнес-логика живёт в `packages/core`, а UI/CLI остаются тонкими клиентами. Источник: [`AGENTS.md`](../../../AGENTS.md) и [Architecture guardrails](../../spec/engineering/architecture-guardrails.md). Читать, чтобы не дублировать доменные правила по интерфейсным слоям.
- `apps/web` и `packages/cli` работают через typed client API и не импортируют доменный core напрямую. Источник: [Architecture guardrails](../../spec/engineering/architecture-guardrails.md). Читать, чтобы сохранить разделение слоёв и тестируемость.
- Фича поставляется вертикальным слайсом, а не layer-first кусками. Источник: [Implementation playbook](../../plans/implementation-playbook.md). Читать, чтобы каждый changeset закрывал работающий сквозной кусок, а не “часть инфраструктуры”.
- `.memory-bank/` является SSoT по требованиям, решениям, планам и evidence. Источник: [`AGENTS.md`](../../../AGENTS.md) и [Memory Bank index](../../index.md). Читать, чтобы не придумывать правила “из головы” и не расходиться с документацией.
- Завершение фичи является evidence-driven: нужны quality checks, acceptance и зафиксированные артефакты. Источник: [Testing standards](../../spec/engineering/testing-standards.md) и [Delivery standards](../../spec/engineering/delivery-standards.md). Читать, чтобы не считать фичу закрытой только потому, что код написан.
- Traceability в git обязательна: ветки, коммиты и PR должны быть привязаны к `[FT-*]`/`[EP-*]`. Источник: [Git flow](../../spec/operations/git-flow.md). Читать, чтобы код, план и verification можно было связать без ручного расследования.
- Transitional `legacy.ts` и compatibility shims допустимы только там, где это явно зафиксировано; новые feature changes не должны бесконтрольно наращивать эти слои. Источник: [Architecture guardrails](../../spec/engineering/architecture-guardrails.md). Читать перед рефакторингом и перед выбором точки встраивания нового кода.

## Рабочий цикл агента

1. Открыть FT-документ и пройти `Project grounding`.
2. Прочитать связанные `spec/`, `adr/` и контекстные документы из FT.
3. Свериться с operation catalog, CLI catalog и traceability.
4. Обновить contract и каталоги операций, если меняется внешнее поведение.
5. Реализовать `core` use-case, policy и инварианты.
6. Добавить `DB` изменения и adapters, если они нужны.
7. Подключить typed client и CLI; UI остаётся тонкой поверхностью над уже определённым поведением.
8. Добавить или обновить tests по нужному уровню.
9. Синхронизировать docs и evidence.

Источник этого workflow:
- [Implementation playbook](../../plans/implementation-playbook.md): полный пошаговый маршрут реализации фичи. Читать, если нужен не summary, а каноничный чеклист по слоям и проверкам.
- [Architecture guardrails](../../spec/engineering/architecture-guardrails.md): краткая shape vertical slice и слойные ограничения. Читать, если нужно быстро сверить порядок и ownership между слоями.

## Что читать по типу задачи

- Для старта новой фичи: [Implementation playbook](../../plans/implementation-playbook.md), FT-документ в `plans/epics/*` и [Architecture guardrails](../../spec/engineering/architecture-guardrails.md). Читать, чтобы начать не “по памяти”, а от контекста, слоёв и acceptance.
- Для доменных правил: [Domain index](../../spec/domain/index.md) и соответствующий [ADR index](../../adr/index.md). Читать, когда нужно восстановить инварианты и понять rationale по анонимности, freeze, visibility и lifecycle.
- Для API/CLI поверхности: [Client API index](../../spec/client-api/index.md) и [CLI spec index](../../spec/cli/index.md). Читать, когда меняется операция, transport contract или форма CLI-команды.
- Для тестов и verification: [Testing standards](../../spec/engineering/testing-standards.md) и [Verification matrix](../../plans/verification-matrix.md). Читать, когда нужно понять, какие проверки обязательны именно для этой FT/GS.
- Для merge/release: [Delivery standards](../../spec/engineering/delivery-standards.md), [Git flow](../../spec/operations/git-flow.md) и [Runbook](../../spec/operations/runbook.md). Читать, когда change уже реализован и нужно закрыть его по process gate, а не только по коду.
- Для документации: [MBB index](../../mbb/index.md), [Principles](../../mbb/principles.md), [Indexing](../../mbb/indexing.md) и [Frontmatter standards](../../mbb/frontmatter.md). Читать перед созданием новых docs и обновлением индексов, чтобы не плодить orphan files и дубли.

## Важная оговорка

Если для решения задачи нужен спор между этим inventory и каноничным документом, этот inventory должен проиграть.

Его задача — ускорять onboarding и execution, а не подменять `spec`, `plans`, `adr`, `mbb` или `AGENTS.md`.
