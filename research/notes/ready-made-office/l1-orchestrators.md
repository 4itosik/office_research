# Слой 1. Оркестраторы «AI-компании»: что брать готовым, что как референс, что писать самому

> Состояние на 2026-10-09. Заметки исследователя по слою 1 build-vs-buy: оркестраторы с оргструктурой агентов, бюджетами на агента, гейтами одобрения человеком, расписаниями и журналом аудита. Контекст: solo-фаундер в РФ. LLM-агенты играют роли директора, аналитика и маркетолога, человек выступает советом директоров. Недельный цикл: скаут → стоп-факторы → Wordstat / прогноз Директа / ГИР БО → unit-экономика → турнир → smoke-test с лестницей бюджета (Директ + лендинг + предоплата через ЮKassa).
>
> **Как собирались данные и чего не удалось проверить.**
> - Метрики репозиториев (звёзды, форки, watchers, открытые issues и PR, число коммитов) сняты 2026-10-09 со страниц github.com через WebFetch.
> - Даты релизов взяты из реестров npm, PyPI, RubyGems и proxy.golang.org (curl).
> - Недоступны из окружения: GitHub API (403), star-history, HN (hn.algolia.com), api.npmjs.org со статистикой скачиваний.
> - Сайты документации не резолвились в WebFetch: docs.paperclip.ing, paperclip.ing, docs.crewai.com, docs.langchain.com, docs.temporal.io, docs.n8n.io. Поэтому документацию я читал из её исходников в GitHub-репозиториях (raw.githubusercontent.com). Ссылки ведут на соответствующие blob-URL на github.com.
> - Лимит WebSearch (общий для всех агентов) закончился в середине работы. Из-за этого часть внешних сигналов осталась непроверенной: обсуждения на HN, новости, динамика звёзд.
> - Пометки: «не проверено» — нет подтверждения из первичного источника; «по сниппету поиска» — факт известен только из выдачи поиска.
>
> **Шкала в таблице возможностей:** 2 — есть из коробки как продуктовая сущность; 1 — частично, собирается из примитивов или есть только в платной редакции; 0 — нет или не найдено.

---

## 1. Таблица кандидатов (шорт-лист из 6)

### Takeaway
Все пять обязательных возможностей как продуктовые сущности с UI для «совета директоров» есть только у Paperclip: оргструктура, бюджеты, гейты, расписания, аудит. Но проекту около 7,5 месяцев, бюджеты в нём считают только LLM-расходы, а изменения в коде идут очень быстро.

Temporal — лучший self-hosted движок для детерминированного недельного конвейера и денежного контура. У него нативные SDK для Go и Ruby, нет зависимости от SaaS, но нет понятий «агент», «роль» и «бюджет».

У остальных четырёх кандидатов есть блокеры:
- Claude Agent SDK и Managed Agents: Anthropic не обслуживает РФ.
- LangGraph: сервер распространяется под Elastic-2.0, требует лицензионного ключа и связи с beacon.langchain.com.
- CrewAI: в OSS нет денежных бюджетов и расписаний, webhook-HITL доступен только в Enterprise.
- n8n: fair-code лицензия, нет оргструктуры и бюджетов.

### Таблица 1. Паспорт и активность (на 2026-10-09)

| Кандидат | URL | Лицензия | Стек | Активность | Аномалии |
|---|---|---|---|---|---|
| **Paperclip** | [github.com/paperclipai/paperclip](https://github.com/paperclipai/paperclip) | MIT, «MIT © 2026 Paperclip Labs, Inc» ([README](https://github.com/paperclipai/paperclip/blob/master/README.md)); в npm также MIT ([npm](https://registry.npmjs.org/paperclipai)) | Сервер на Node.js 24.11+ и React UI ([README](https://github.com/paperclipai/paperclip/blob/master/README.md)). TypeScript + Express REST, PostgreSQL ([SPEC](https://github.com/paperclipai/paperclip/blob/master/doc/SPEC.md)), Drizzle ORM ([database](https://github.com/paperclipai/paperclip/blob/master/docs/deploy/database.md)). При сборке из исходников собирается нативный Runner на Rust ([README](https://github.com/paperclipai/paperclip/blob/master/README.md)) | 99.1k звёзд / 16.7k форков / 463 watchers / 2.9k открытых issues / 3.4k открытых PR / 4,930 коммитов ([repo](https://github.com/paperclipai/paperclip)). Последний коммит — 2026-10-09 ([commits](https://github.com/paperclipai/paperclip/commits/master)). Последний релиз npm `paperclipai` 2026.1005.0 вышел 2026-10-06; всего 1,763 версии, первая публикация 2026-03-03 ([npm](https://registry.npmjs.org/paperclipai)). Самые ранние коммиты датированы 2026-02-20; до 2026-02-15 истории нет ([until 02-20](https://github.com/paperclipai/paperclip/commits/master/?until=2026-02-20), [until 02-15](https://github.com/paperclipai/paperclip/commits/master/?until=2026-02-15)). Число контрибьюторов на странице не показано — не проверено | **Есть.** Около 99k звёзд за ~7,5 месяцев. Это намного быстрее, чем CrewAI (59.5k с 2023 года) и LangGraph (43.0k с 2024 года). 3.4k открытых PR. Коммиты делают в основном 1–2 core-разработчика в соавторстве с «claude» или «Paperclip-Paperclip». Подробнее — в разделе 2 |
| **Claude Agent SDK + Claude Managed Agents** | [claude-agent-sdk-python](https://github.com/anthropics/claude-agent-sdk-python), [claude-agent-sdk-typescript](https://github.com/anthropics/claude-agent-sdk-typescript), [Managed Agents](https://platform.claude.com/docs/en/managed-agents/overview) | Python-репозиторий под MIT ([GitHub](https://github.com/anthropics/claude-agent-sdk-python), [PyPI](https://pypi.org/pypi/claude-agent-sdk/json)). TS-репозиторий — под Commercial Terms Anthropic ([TS repo](https://github.com/anthropics/claude-agent-sdk-typescript)). Использование SDK в целом регулируется Commercial Terms ([overview](https://code.claude.com/docs/en/agent-sdk/overview)). Пакет `@anthropic-ai/claude-code` — «SEE LICENSE IN README.md» ([npm](https://registry.npmjs.org/@anthropic-ai/claude-code)). Managed Agents — облачный сервис Anthropic в beta ([MA](https://platform.claude.com/docs/en/managed-agents/overview)) | Python ≥3.10 или TypeScript. SDK запускает встроенный бинарь Claude Code ([Python README](https://github.com/anthropics/claude-agent-sdk-python)) | Python: 8.2k звёзд / 1.3k форков / 216 issues / 323 PR / 907 коммитов. TS: 1.8k / 231 / 16 watchers / 217 / 11 / 330 ([py](https://github.com/anthropics/claude-agent-sdk-python), [ts](https://github.com/anthropics/claude-agent-sdk-typescript)). PyPI 0.2.165 от 2026-10-08: 150 релизов, первый — 2025-09-28 ([PyPI](https://pypi.org/pypi/claude-agent-sdk/json)). npm 0.3.295 от 2026-10-08: 320 версий, первая — 2025-09-27 ([npm](https://registry.npmjs.org/@anthropic-ai/claude-agent-sdk)) | Нет |
| **CrewAI** | [github.com/crewAIInc/crewAI](https://github.com/crewAIInc/crewAI) | MIT, «Copyright (c) 2025 crewAI, Inc.» ([LICENSE](https://github.com/crewAIInc/crewAI/blob/main/LICENSE)). AMP — коммерческий «control plane» ([README](https://github.com/crewAIInc/crewAI)) | Python от 3.10 до 3.13 включительно ([PyPI](https://pypi.org/pypi/crewai/json)) | 59.5k / 8.7k / 397 / 226 / 359 / 2,951 ([repo](https://github.com/crewAIInc/crewAI)). Последний коммит — 2026-10-09 ([commits](https://github.com/crewAIInc/crewAI/commits/main)). PyPI 1.15.26 от 2026-10-08: 465 релизов, первый 0.1.0 — 2023-11-14 ([PyPI](https://pypi.org/pypi/crewai/json)) | Нет |
| **LangGraph** | [github.com/langchain-ai/langgraph](https://github.com/langchain-ai/langgraph) | Библиотека под MIT ([repo](https://github.com/langchain-ai/langgraph), [PyPI](https://pypi.org/pypi/langgraph/json)). Agent Server (`langgraph-api`) — **Elastic-2.0** ([PyPI](https://pypi.org/pypi/langgraph-api/json)); отдельно развёрнутому серверу нужен лицензионный ключ ([standalone](https://github.com/langchain-ai/docs/blob/main/src/langsmith/deploy-standalone-server.mdx)) | Python ≥3.10 и JS (в документации есть примеры на обоих языках) ([interrupts](https://github.com/langchain-ai/docs/blob/main/src/oss/langgraph/interrupts.mdx)) | 43.0k / 7.3k / 190 / 569 / 226 / 7,148 ([repo](https://github.com/langchain-ai/langgraph)). `langgraph` 1.2.14 от 2026-10-06: 279 релизов, первый — 2024-01-08 ([PyPI](https://pypi.org/pypi/langgraph/json)). `langgraph-api` 0.15.4 от 2026-10-08 ([PyPI](https://pypi.org/pypi/langgraph-api/json)). Дата последнего коммита не проверена: страница коммитов не отдалась | Нет |
| **Temporal** | [github.com/temporalio/temporal](https://github.com/temporalio/temporal) | Сервер под MIT ([repo](https://github.com/temporalio/temporal)). Ruby gem под MIT ([RubyGems](https://rubygems.org/api/v1/versions/temporalio.json)), Python SDK тоже под MIT ([PyPI](https://pypi.org/pypi/temporalio/json)) | Сервер на Go. Проверены SDK для Go (`go.temporal.io/sdk`), Ruby (gem `temporalio`; Ruby 3.3, 3.4, 4.0; только Linux и macOS) ([sdk-ruby](https://github.com/temporalio/sdk-ruby)) и Python. Другие SDK не проверялись | 23.6k / 2.0k / 123 / 578 / 479 / 9,973 ([repo](https://github.com/temporalio/temporal)). Последний коммит — 2026-10-09 ([commits](https://github.com/temporalio/temporal/commits/main)). Сервер v1.32.1 от 2026-10-07 ([Go proxy](https://proxy.golang.org/github.com/temporalio/temporal/@latest)), сервер v1.0.0 вышел 2020-09-30 ([Go proxy](https://proxy.golang.org/go.temporal.io/server/@v/v1.0.0.info)). Go SDK v1.49.0 от 2026-09-14 ([Go proxy](https://proxy.golang.org/go.temporal.io/sdk/@latest)). Ruby gem: 1.9.0 от 2026-09-14, 1.0.0 от 2025-09-29, 0.1.0 от 2023-03-23 ([RubyGems](https://rubygems.org/api/v1/versions/temporalio.json)) | Нет |
| **n8n** | [github.com/n8n-io/n8n](https://github.com/n8n-io/n8n) | Sustainable Use License (fair-code, не OSI). Файлы `.ee` распространяются под n8n Enterprise License ([LICENSE.md](https://github.com/n8n-io/n8n/blob/master/LICENSE.md)) | TypeScript / Node.js. БД — SQLite или PostgreSQL: среди зависимостей есть `sqlite3` и `pg` ([npm](https://registry.npmjs.org/n8n)) | 206.8k / 61.0k / 1.2k / 359 / 815 / 25,547 ([repo](https://github.com/n8n-io/n8n)). Последний коммит — 2026-10-09 ([commits](https://github.com/n8n-io/n8n/commits/master)). npm 2.42.6 от 2026-10-09, первая публикация — 2019-04-24 ([npm](https://registry.npmjs.org/n8n)) | Нет |

### Таблица 2. Пять обязательных возможностей (оценка 0–2 с доказательствами)

| Кандидат | (a) Оргструктура агентов | (b) Бюджеты на агента | (c) Гейты одобрения | (d) Расписание | (e) Аудит |
|---|---|---|---|---|---|
| **Paperclip** | **2.** Строгое дерево: у каждого агента ровно один менеджер (`reportsTo`), CEO подчиняется человеку (совету). Подзадачи делегируются вниз, эскалация идёт вверх по `chainOfCommand` ([org-structure](https://github.com/paperclipai/paperclip/blob/master/docs/guides/board-operator/org-structure.md)). В одну оргструктуру входят и люди, и агенты ([README](https://github.com/paperclipai/paperclip/blob/master/README.md)) | **2.** Поле `budgetMonthlyCents` у компании и у агента. На 80% бюджета — мягкое предупреждение, на 100% — **hard stop**: агент ставится на автопаузу. Окно — календарный месяц по UTC ([costs API](https://github.com/paperclipai/paperclip/blob/master/docs/api/costs.md)). Учитываются только токены и стоимость LLM; учёт внешних доходов и расходов — «future plugin» ([SPEC](https://github.com/paperclipai/paperclip/blob/master/doc/SPEC.md)) | **2.** Типы одобрений: `hire_agent`, `approve_ceo_strategy`, `budget_override_required`, `request_board_approval` ([SPEC-implementation](https://github.com/paperclipai/paperclip/blob/master/doc/SPEC-implementation.md)). Жизненный цикл: pending → approved / rejected / revision_requested → resubmitted ([approvals API](https://github.com/paperclipai/paperclip/blob/master/docs/api/approvals.md)). Для действий MCP-шлюза задаётся режим Allowed, Ask first или Off ([README](https://github.com/paperclipai/paperclip/blob/master/README.md)) | **2.** Routines с триггерами cron, webhook и API; есть политики конкурентности и догоняющих запусков (catch-up); каждый запуск создаёт задачу ([README](https://github.com/paperclipai/paperclip/blob/master/README.md)). Heartbeat срабатывает по таймеру, при назначении задачи, по комментарию, вручную или после решения по approval ([core concepts](https://github.com/paperclipai/paperclip/blob/master/docs/start/core-concepts.md)) | **2.** Журнал всех мутаций: append-only, неизменяемый, поля actor / action / entity / details ([activity API](https://github.com/paperclipai/paperclip/blob/master/docs/api/activity.md)). Cost events по агенту, провайдеру и модели ([costs API](https://github.com/paperclipai/paperclip/blob/master/docs/api/costs.md)). Заголовок `X-Paperclip-Run-Id` связывает мутации с прогоном ([API overview](https://github.com/paperclipai/paperclip/blob/master/docs/api/overview.md)) |
| **Claude Agent SDK** (в скобках — Managed Agents) | **1.** Есть субагенты для подзадач, но нет оргструктуры и ролей как сущностей ([overview](https://code.claude.com/docs/en/agent-sdk/overview)) | **1 (MA: 2).** В SDK `max_budget_usd` / `maxBudgetUsd` на один вызов; при превышении результат `error_max_budget_usd`. Но `total_cost_usd` — клиентская оценка, документация предупреждает: «Do not bill end users or trigger financial decisions from these fields» ([cost tracking](https://code.claude.com/docs/en/agent-sdk/cost-tracking)). В Managed Agents жёсткий потолок на сессию в центах USD по list-цене, при достижении — пауза `budget_reached` ([MA budgets](https://platform.claude.com/docs/en/managed-agents/budgets)) | **1 (MA: 2).** В SDK: permissions (какие инструменты требуют одобрения), хук `PreToolUse` с deny, `can_use_tool` ([overview](https://code.claude.com/docs/en/agent-sdk/overview), [Python README](https://github.com/anthropics/claude-agent-sdk-python)). В MA: событие `user.tool_confirmation` и статус сессии `requires_action` ([MA budgets](https://platform.claude.com/docs/en/managed-agents/budgets)) | **0 (MA: 2).** В SDK расписания нет. В MA есть scheduled deployments: POSIX cron и часовой пояс IANA, до 1 000 на организацию ([MA deployments](https://platform.claude.com/docs/en/managed-agents/scheduled-deployments)) | **1 (MA: 2).** В SDK — usage по шагам и `modelUsage` ([cost tracking](https://code.claude.com/docs/en/agent-sdk/cost-tracking)). В MA история событий хранится на сервере, есть записи deployment runs и webhooks ([MA overview](https://platform.claude.com/docs/en/managed-agents/overview), [MA deployments](https://platform.claude.com/docs/en/managed-agents/scheduled-deployments)) |
| **CrewAI** | **2.** Агенты с role, goal и backstory. В hierarchical process менеджер-агент распределяет задачи и проверяет результат; делегирование по умолчанию выключено ([hierarchical](https://github.com/crewAIInc/crewAI/blob/main/docs/edge/en/learn/hierarchical-process.mdx)) | **0.** Есть только `max_iter` (по умолчанию 20), `max_rpm`, `max_execution_time`, `max_retry_limit`; денежных лимитов нет ([agents](https://github.com/crewAIInc/crewAI/blob/main/docs/edge/en/concepts/agents.mdx)) | **1.** Декоратор `@human_feedback` во Flows (с версии 1.8.0): маршрутизация `emit=["approved","rejected",...]`, асинхронный `provider` ([HF in Flows](https://github.com/crewAIInc/crewAI/blob/main/docs/edge/en/learn/human-feedback-in-flows.mdx)). Webhook-HITL для продакшена — только в Enterprise ([HITL](https://github.com/crewAIInc/crewAI/blob/main/docs/edge/en/learn/human-in-the-loop.mdx)) | **0.** В README OSS-версии расписание не упоминается ([repo](https://github.com/crewAIInc/crewAI)). Есть ли оно в AMP — не проверено | **1.** Checkpointing в JSON или SQLite ([checkpointing](https://github.com/crewAIInc/crewAI/blob/main/docs/edge/en/concepts/checkpointing.mdx)). Трассировка и наблюдаемость — в AMP ([README](https://github.com/crewAIInc/crewAI)) |
| **LangGraph** | **1.** Встроенной оргструктуры нет; несколько агентов собираются графом. Детали мультиагентных паттернов не проверялись | **0.** Не найдено | **2.** `interrupt()` можно вызвать в любой ноде, продолжение — через `Command(resume=...)`; состояние хранится в checkpointer по `thread_id` ([interrupts](https://github.com/langchain-ai/docs/blob/main/src/oss/langgraph/interrupts.mdx)) | **1.** Cron jobs есть только в Agent Server / LangSmith Deployment, расписание задаётся в UTC ([cron-jobs](https://github.com/langchain-ai/docs/blob/main/src/langsmith/cron-jobs.mdx)). В самой библиотеке расписания нет | **1.** Состояние сохраняется чекпоинтами на каждом шаге ([persistence](https://github.com/langchain-ai/docs/blob/main/src/oss/langgraph/persistence.mdx)). Полноценная трассировка — в LangSmith (SaaS или self-hosted в рамках Enterprise) ([self-hosted](https://github.com/langchain-ai/docs/blob/main/src/langsmith/self-hosted.mdx)) |
| **Temporal** | **0.** Понятий «агент», «роль», «иерархия» нет. Иерархию можно выразить только через parent/child workflows (это мой вывод) | **0.** Бюджетов нет; лимиты придётся писать в коде workflow | **2.** Signals (асинхронные) и Updates (синхронные, с результатом и валидаторами) вместе с `Workflow.wait_condition` дают надёжное ожидание решения человека, которое переживает сбои ([message passing](https://github.com/temporalio/documentation/blob/main/docs/encyclopedia/workflow-message-passing/workflow-message-passing.mdx), [sdk-ruby](https://github.com/temporalio/sdk-ruby)) | **2.** Schedules: интервальные и календарные (cron) спецификации, часовые пояса, пауза с заметками, overlap policy, catch-up window, jitter, backfill ([schedule](https://github.com/temporalio/documentation/blob/main/docs/encyclopedia/workflow/schedule.mdx)). В Ruby — `create_schedule` ([ruby schedules](https://github.com/temporalio/documentation/blob/main/docs/develop/ruby/workflows/schedules.mdx)) | **2 по действиям, 0 по стоимости.** Event History — «a complete and durable log of everything that has happened» ([event history](https://github.com/temporalio/documentation/blob/main/docs/encyclopedia/event-history/event-history.mdx)). Стоимость LLM и рекламы не учитывается |
| **n8n** | **0** | **0.** Не найдено | **2.** HITL для инструментов AI Agent: Approve или Deny через Slack, Telegram или n8n Chat ([HITL for tools](https://github.com/n8n-io/n8n-docs/blob/main/docs/build/integrate-ai/ai-examples/human-in-the-loop-for-tools.md)) | **2.** Schedule Trigger поддерживает интервалы вплоть до недель и cron-выражения ([ScheduleTrigger.node.ts](https://github.com/n8n-io/n8n/blob/master/packages/nodes-base/nodes/Schedule/ScheduleTrigger.node.ts)) | **1.** Есть история выполнений; детали и лог-стриминг не проверены |

### Таблица 3. Хранилище, UI для совета, API, интеграция с Rails и Go

| Кандидат | Хранилище | UI для «совета» | API | Как подключить к Rails / Go |
|---|---|---|---|---|
| **Paperclip** | PostgreSQL: по умолчанию встроенный, для продакшена внешний (Drizzle). Файлы хранятся локально или в S3-совместимом хранилище ([database](https://github.com/paperclipai/paperclip/blob/master/docs/deploy/database.md), [README](https://github.com/paperclipai/paperclip/blob/master/README.md)) | React UI: оргчарт, доска задач, страница Approvals, дашборд расходов против бюджета, лента активности, мобильный режим ([README](https://github.com/paperclipai/paperclip/blob/master/README.md), [approvals guide](https://github.com/paperclipai/paperclip/blob/master/docs/guides/board-operator/approvals.md), [costs guide](https://github.com/paperclipai/paperclip/blob/master/docs/guides/board-operator/costs-and-budgets.md)) | REST JSON по пути `/api`: companies, agents, issues, approvals, goals и projects, costs, secrets, activity, dashboard ([docs.json](https://github.com/paperclipai/paperclip/blob/master/docs/docs.json), [API overview](https://github.com/paperclipai/paperclip/blob/master/docs/api/overview.md)). Входящие webhook- и API-триггеры для routines, MCP-шлюз, плагины, CLI ([README](https://github.com/paperclipai/paperclip/blob/master/README.md)) | (1) Обычный REST-клиент из Rails или Go: создаёт задачи и approvals, читает costs и activity. В SPEC прямо сказано: «REST API, not tRPC — need non-TS clients» ([SPEC](https://github.com/paperclipai/paperclip/blob/master/doc/SPEC.md)). (2) Свой сервис на Go или Rails становится «сотрудником» через адаптер `http`: Paperclip отправляет POST с `runId`, `agentId` и контекстом, сервис отвечает через API с `PAPERCLIP_API_URL` и ключом ([http adapter](https://github.com/paperclipai/paperclip/blob/master/docs/adapters/http.md)). Второй вариант — адаптер `process` запускает ruby- или go-команду с переданными `PAPERCLIP_*` ([process adapter](https://github.com/paperclipai/paperclip/blob/master/docs/adapters/process.md)) |
| **Claude Agent SDK / MA** | SDK хранит транскрипты сессий Claude Code локально ([cost tracking](https://code.claude.com/docs/en/agent-sdk/cost-tracking)). MA хранит всё на серверах Anthropic; Zero Data Retention на него не распространяется ([MA overview](https://platform.claude.com/docs/en/managed-agents/overview)) | В SDK UI нет. У MA — Claude Console с трассировкой и деплойментами ([MA deployments](https://platform.claude.com/docs/en/managed-agents/scheduled-deployments)) | SDK — библиотека для Python и TS. Из других языков: «run the CLI as a subprocess with the `-p` flag and `--output-format json`» ([overview](https://code.claude.com/docs/en/agent-sdk/overview)). У MA — REST и SDK, включая Go и Ruby, плюс webhooks ([MA deployments](https://platform.claude.com/docs/en/managed-agents/scheduled-deployments)) | Rails или Go запускают `claude -p` подпроцессом либо вызывают REST или SDK Managed Agents (в документации есть примеры на Go и Ruby) |
| **CrewAI** | Чекпоинты в JSON или SQLite ([checkpointing](https://github.com/crewAIInc/crewAI/blob/main/docs/edge/en/concepts/checkpointing.mdx)) | В OSS нет. AMP — коммерческий control plane ([README](https://github.com/crewAIInc/crewAI)) | Python-библиотека. В AMP: REST `/kickoff` с `humanInputWebhook` ([HITL](https://github.com/crewAIInc/crewAI/blob/main/docs/edge/en/learn/human-in-the-loop.mdx)) | Только через собственную обёртку на Python (FastAPI и т.п.) или через платный AMP (мой вывод) |
| **LangGraph** | Checkpointers (InMemory и др.) ([persistence](https://github.com/langchain-ai/docs/blob/main/src/oss/langgraph/persistence.mdx)). Agent Server: PostgreSQL (assistants, threads, runs, cron jobs, чекпоинты) и Redis ([agent-server](https://github.com/langchain-ai/docs/blob/main/src/langsmith/agent-server.mdx)) | LangSmith Studio (SaaS или Enterprise) ([README](https://github.com/langchain-ai/langgraph), [self-hosted](https://github.com/langchain-ai/docs/blob/main/src/langsmith/self-hosted.mdx)) | Библиотека для Python и JS. Agent Server: REST и `langgraph_sdk` ([cron-jobs](https://github.com/langchain-ai/docs/blob/main/src/langsmith/cron-jobs.mdx)) | HTTP к Agent Server (нужна лицензия) или собственная Python-обёртка |
| **Temporal** | PostgreSQL 13–16, MySQL 5.7 и 8.0, Cassandra. SQLite — только для разработки и тестов ([persistence](https://github.com/temporalio/documentation/blob/main/docs/encyclopedia/temporal-service/persistence.mdx)) | Temporal Web UI — технический интерфейс уровня workflow, в dev-режиме на localhost:8233 ([README](https://github.com/temporalio/temporal)). На UI совета не тянет | SDK и CLI ([README](https://github.com/temporalio/temporal)) | Ruby: gem `temporalio`. В README SDK есть раздел про Rails, сэмпл `rails_app`, советы не переиспользовать модели ActiveRecord и загружать код workflow заранее (eager loading) ([sdk-ruby](https://github.com/temporalio/sdk-ruby)). Go: нативный `go.temporal.io/sdk` ([Go proxy](https://proxy.golang.org/go.temporal.io/sdk/@latest)) |
| **n8n** | SQLite или PostgreSQL ([npm](https://registry.npmjs.org/n8n)) | Редактор workflow. Запросы на одобрение уходят в Slack, Telegram или Chat ([HITL](https://github.com/n8n-io/n8n-docs/blob/main/docs/build/integrate-ai/ai-examples/human-in-the-loop-for-tools.md)) | MCP Server Trigger с поддержкой SSE и streamable HTTP ([MCP trigger](https://github.com/n8n-io/n8n-docs/blob/main/docs/integrations/builtin/core-nodes/n8n-nodes-langchain.mcptrigger.md)). Публичный REST API и webhook-ноды в этой сессии не проверены | HTTP в обе стороны (мой вывод) |

### Таблица 4. Покрытие недельного цикла, применимость в РФ, секреты, вердикт

| Кандидат | Какие роли и этапы цикла покрывает | Работает ли в РФ | Где хранит секреты и токены | Вердикт |
|---|---|---|---|---|
| **Paperclip** | **Покрывает:** роли (CEO-директор, аналитик, маркетолог), еженедельную routine «скаут», задачу на каждую стадию, гейты совета (бюджет, домены, юрвопросы, GO) через `request_board_approval` и режим Ask first у MCP-инструментов с деньгами, LLM-бюджеты, аудит. **Не покрывает:** сбор данных из Wordstat, Директа и ГИР БО; детерминированную unit-экономику; рублёвый учёт трат на рекламу; ЮKassa | **Условно да.** Self-hosted, аккаунт Paperclip не нужен ([README](https://github.com/paperclipai/paperclip/blob/master/README.md)). Дефолтные рантаймы Claude Code и Codex требуют Anthropic или OpenAI, а РФ они официально не обслуживают. Путь для РФ: адаптер `opencode_local` (несколько провайдеров) ([adapters](https://github.com/paperclipai/paperclip/blob/master/docs/adapters/overview.md)), а OpenCode принимает любой OpenAI-совместимый `baseURL` ([OpenCode providers](https://github.com/sst/opencode/blob/dev/packages/web/src/content/docs/providers.mdx)). Так подключаются Yandex AI Studio, GigaChat через gpt2giga или vLLM. Телеметрия включена по умолчанию; выключается `PAPERCLIP_TELEMETRY_DISABLED=1` ([README](https://github.com/paperclipai/paperclip/blob/master/README.md)) | Секреты шифруются at rest локальным мастер-ключом (провайдер `local_encrypted`) или хранятся в AWS Secrets Manager. К агенту, проекту или окружению секрет привязывается по ссылке ([secrets](https://github.com/paperclipai/paperclip/blob/master/docs/deploy/secrets.md)). Ключи агентов хранятся в хэшированном виде ([auth](https://github.com/paperclipai/paperclip/blob/master/docs/api/authentication.md)) | **Брать в режиме пилота** — как оболочку «компании и совета»: оргчарт, approvals, LLM-бюджеты, аудит, UI. Ядро конвейера и деньги в нём не держать: проекту ~7,5 месяцев, код меняется очень быстро, бюджеты считают только LLM |
| **Claude Agent SDK / MA** | Рантайм отдельного агента-роли (аналитик, маркетолог) с MCP-инструментами, лимитом в долларах на прогон и одобрением инструментов. У MA сверху — cron | **Нет как продакшен-рантайм.** России нет в списке поддерживаемых стран Anthropic ([supported countries](https://www.anthropic.com/supported-countries)). Anthropic «doesn't support routing Claude Code to non-Claude models through any gateway» ([LLM gateway](https://code.claude.com/docs/en/llm-gateway)). Технически gpt2giga даёт Anthropic-совместимый `/messages` поверх GigaChat ([gpt2giga](https://github.com/ai-forever/gpt2giga)), но это неподдерживаемый путь | В SDK — переменные окружения и API-ключ. В MA — vaults ([MA deployments](https://platform.claude.com/docs/en/managed-agents/scheduled-deployments)) | **Референс:** дизайн бюджетов, cron и подтверждения инструментов. Для фаундера это ещё и инструмент разработки |
| **CrewAI** | Роли (директор как manager, аналитик, маркетолог); «турнир» можно реализовать как crew. Flows подходят для конвейера с ручным approve. Расписаний и денежных бюджетов нет | **Да для OSS.** OpenAI SDK с `base_url` или `OPENAI_BASE_URL`, остальные провайдеры через LiteLLM ([llms](https://github.com/crewAIInc/crewAI/blob/main/docs/edge/en/concepts/llms.mdx)): Yandex, GigaChat через gpt2giga, vLLM. Телеметрия включена по умолчанию, выключается `CREWAI_DISABLE_TELEMETRY` ([telemetry](https://github.com/crewAIInc/crewAI/blob/main/docs/edge/en/telemetry.mdx)). Доступность AMP в РФ не проверена | Переменные окружения ([llms](https://github.com/crewAIInc/crewAI/blob/main/docs/edge/en/concepts/llms.mdx)) | **Референс** |
| **LangGraph** | Агентные графы с прерываниями для одобрений. Расписание только в лицензируемом сервере | **Библиотека — да.** У отдельно развёрнутого Agent Server обязательные зависимости от вендора: `LANGGRAPH_CLOUD_LICENSE_KEY` и исходящий доступ к `beacon.langchain.com` для проверки лицензии и отчёта об использовании, если сервер не в air-gapped режиме ([standalone](https://github.com/langchain-ai/docs/blob/main/src/langsmith/deploy-standalone-server.mdx)). Self-hosted LangSmith — дополнение к Enterprise-плану ([self-hosted](https://github.com/langchain-ai/docs/blob/main/src/langsmith/self-hosted.mdx)) | Переменные окружения: `DATABASE_URI`, `REDIS_URI`, ключи ([standalone](https://github.com/langchain-ai/docs/blob/main/src/langsmith/deploy-standalone-server.mdx)) | **Референс** (паттерны interrupt и checkpoint) |
| **Temporal** | Весь детерминированный недельный конвейер: сбор данных как activities с ретраями, фильтры, расчёты, турнир через child workflows, гейты через signals и updates, smoke-test с лестницей бюджета на таймерах, вебхуки ЮKassa как signals. Нет ролей, LLM-бюджетов и UI совета | **Да.** Self-hosted под MIT, обязательной SaaS-зависимости нет (вывод по README и persistence). К LLM-провайдеру не привязан: любая модель вызывается как activity (мой вывод) | Встроенного хранилища секретов нет, секреты живут в окружении воркеров. Шифрование payload — через раздел data conversion в документации (не проверялось) | **Брать** — как движок собственного конвейера и денежного контура |
| **n8n** | Расписание, HTTP-интеграции, AI Agent с approve и deny через Telegram. Нет оргструктуры и бюджетов | **Да, self-hosted.** SUL разрешает использование «only for your own internal business purposes» ([LICENSE.md](https://github.com/n8n-io/n8n/blob/master/LICENSE.md)). В кредитиве OpenAI есть поля Base URL и Add Custom Header ([OpenAiApi.credentials.ts](https://github.com/n8n-io/n8n/blob/master/packages/nodes-base/credentials/OpenAiApi.credentials.ts)), так что подключается OpenAI-совместимый endpoint Yandex (нужен заголовок с folder ID — по сниппету поиска). Доступность n8n Cloud в РФ не проверена | Credentials хранятся в БД n8n; механизм шифрования не проверен | **Нет как ядро.** Допустим как временный «клей» для прототипа |

### Cited Findings (общие для РФ, не вошедшие в таблицы)
- **Anthropic.** России и Беларуси нет в списке поддерживаемых стран; «Any country or region not listed is unsupported». Казахстан, Армения, Грузия, Сербия, ОАЭ и Турция в список входят — [Anthropic supported countries](https://www.anthropic.com/supported-countries).
- **Yandex AI Studio** документирует совместимость с OpenAI API (по сниппету поиска) — [Yandex Cloud: OpenAI compatibility](https://yandex.cloud/en/docs/ai-studio/concepts/openai-compatibility):
  - Completions API доступен по `https://llm.api.cloud.yandex.net/v1`, совместимость частичная;
  - Responses API — по `rest-assistant.api.cloud.yandex.net/v1`;
  - в более новой версии страницы все сервисы перечислены по `https://ai.api.cloud.yandex.net/v1`.
- **Habr:** Claude Code нельзя направить на endpoint Yandex напрямую, нужен шлюз-транслятор протокола (по сниппету поиска) — [Habr](https://habr.com/ru/articles/1049322).
- **gpt2giga** (ai-forever) — FastAPI-прокси, который даёт OpenAI-, Anthropic- и Gemini-совместимые эндпоинты поверх GigaChat API. Лицензия MIT, 137 звёзд. Авторы предупреждают, что это не drop-in replacement — [gpt2giga](https://github.com/ai-forever/gpt2giga).
- **OpenCode** (харнесс в адаптере `opencode_local` у Paperclip) подключает любой OpenAI-совместимый API через `@ai-sdk/openai-compatible` с `options.baseURL` — [OpenCode providers](https://github.com/sst/opencode/blob/dev/packages/web/src/content/docs/providers.mdx).

### Inferences
- Метафору «LLM-компании с советом директоров» напрямую моделирует только Paperclip. В нём есть CEO, отчитывающийся перед board, одобрение стратегии и найма, бюджеты, routines и журнал. Остальные кандидаты — либо фреймворки для отдельных агентов, либо движки workflow.
- **Ни один кандидат не даёт доменных коннекторов:** Wordstat, прогноз бюджета Директа, ГИР БО, ЮKassa. Их придётся писать как MCP-серверы или API-клиенты.
- **Ни у одного кандидата нет рублёвого денежного реестра.** Бюджеты везде означают стоимость LLM-токенов в долларах (Paperclip, Claude) или их нет вовсе. Жёсткие лимиты на рекламу и учёт предоплат придётся писать самим.
- **Для РФ решающий фактор — путь к LLM-провайдеру.** Подходят те, у кого настраивается OpenAI-совместимый `base_url`: Paperclip через OpenCode, CrewAI, библиотека LangGraph, n8n, а Temporal вызывает любую модель как activity. Claude Agent SDK и Managed Agents этот фильтр не проходят.

### Gaps
- Число контрибьюторов: GitHub не показал счётчик ни для одного репозитория, GitHub API недоступен — не проверено.
- Дата последнего коммита LangGraph: страница коммитов не отдалась. Как замену использовал дату релиза (2026-10-06).
- **Не вошли в шорт-лист** (лимит — 6 кандидатов), возможности не оценивались:
  - Mastra: `@mastra/core` под Apache-2.0, версия 1.75.0 от 2026-10-07, первая публикация 2024-10-02 — [npm](https://registry.npmjs.org/@mastra/core);
  - Microsoft Agent Framework: `agent-framework`, классификатор MIT, 1.21.0 от 2026-10-08, первая beta 2025-10-01 — [PyPI](https://pypi.org/pypi/agent-framework/json);
  - Agency Swarm: MIT, 1.11.0 от 2026-08-03 — [PyPI](https://pypi.org/pypi/agency-swarm/json);
  - MetaGPT и ChatDev (референсы оргструктуры) не проверялись.
- Насколько надёжно YandexGPT и GigaChat вызывают инструменты внутри агентных харнессов (OpenCode, CrewAI) — не проверено. Для РФ-пути это критично.
- Доступ из РФ к npm, PyPI, Docker Hub и GitHub для установки и обновлений — не проверено.

---

## 2. Deep-dive 1: Paperclip

### Takeaway
Paperclip — self-hosted control plane «компании из агентов» под MIT. В нём есть оргчарт, задачи-тикеты, heartbeats, routines по cron, approvals совета, месячные LLM-бюджеты с hard stop, неизменяемый журнал активности, REST API и React UI.

Концептуально это почти точное совпадение с запросом. Но:
- проекту около 7,5 месяцев, а популярность растёт аномально быстро (99.1k звёзд);
- код в значительной мере пишется AI-агентами, изменений очень много;
- бюджеты считают только LLM-затраты;
- в РФ его можно запустить только через мультипровайдерные харнессы (OpenCode, Hermes, Pi) или свои `http`- и `process`-адаптеры.

### Cited Findings

**Что это и как позиционируется**
- «Paperclip is a Node.js server and React UI that orchestrates a team of AI agents to run a business»; «It looks like a task manager. Under the hood: org charts, budgets, governance, goal alignment, and agent coordination» — [README](https://github.com/paperclipai/paperclip/blob/master/README.md).
- Сценарий из README: Define the goal → Hire the team («CEO, CTO, engineers, designers, marketers — any bot, any provider») → Approve and run («Review strategy. Set budgets. Hit go.») — [README](https://github.com/paperclipai/paperclip/blob/master/README.md).
- «Not an agent framework. We don't tell you how to build agents. We tell you how to run a company made of them.» — [README](https://github.com/paperclipai/paperclip/blob/master/README.md).

**Концепции**
- **Организация (company):**
  - у компании есть goal, сотрудники-агенты, оргструктура, месячный бюджет в центах и иерархия задач от цели компании;
  - один инстанс обслуживает несколько компаний — [core concepts](https://github.com/paperclipai/paperclip/blob/master/docs/start/core-concepts.md).
- **Агент:**
  - у агента есть тип адаптера и его конфиг, роль и отношения подчинения, capabilities, бюджет и статус (active, idle, running, error, paused, terminated) — [core concepts](https://github.com/paperclipai/paperclip/blob/master/docs/start/core-concepts.md).
- **Оргструктура:**
  - строгое дерево без циклов, у каждого агента ровно один менеджер, CEO подчиняется board;
  - `GET /api/companies/{companyId}/org`;
  - эскалация и делегирование идут по `chainOfCommand`;
  - задачу из другой ветки агент получить может, но отменить её не может — [org-structure](https://github.com/paperclipai/paperclip/blob/master/docs/guides/board-operator/org-structure.md).
- **Задачи (issues):**
  - статусы backlog → todo → in_progress → in_review → done, отдельно blocked;
  - переход в in_progress требует атомарного checkout, при конфликте — `409 Conflict` — [core concepts](https://github.com/paperclipai/paperclip/blob/master/docs/start/core-concepts.md);
  - «There is no separate messaging or chat system. Tasks are the communication channel… creates a natural audit trail» — [SPEC](https://github.com/paperclipai/paperclip/blob/master/doc/SPEC.md);
  - агенты могут ставить задачи людям — [SPEC](https://github.com/paperclipai/paperclip/blob/master/doc/SPEC.md).
- **Делегирование:**
  - CEO составляет стратегию и отправляет её на одобрение;
  - после одобрения разбивает цели на задачи, назначает их и нанимает новых агентов (найм можно закрыть гейтом) — [core concepts](https://github.com/paperclipai/paperclip/blob/master/docs/start/core-concepts.md).
- **Heartbeats:**
  - агенты работают короткими окнами;
  - триггеры: Schedule, Assignment, Comment, Manual, Approval resolution — [core concepts](https://github.com/paperclipai/paperclip/blob/master/docs/start/core-concepts.md);
  - «The heartbeat is a protocol, not a runtime»;
  - два режима передачи контекста: «fat payload» и «thin ping»;
  - контракт адаптера — `invoke` / `status` / `cancel` — [SPEC](https://github.com/paperclipai/paperclip/blob/master/doc/SPEC.md);
  - в README: «DB-backed wakeup queue with coalescing, budget checks… secret injection» — [README](https://github.com/paperclipai/paperclip/blob/master/README.md).
- **Routines:**
  - «Recurring tasks with cron, webhook, and API triggers. Concurrency and catch-up policies. Each routine execution creates a tracked issue and wakes the assigned agent» — [README](https://github.com/paperclipai/paperclip/blob/master/README.md).
- **Governance:**
  - в V1 гейты board — найм новых агентов и первоначальная стратегия CEO — [SPEC](https://github.com/paperclipai/paperclip/blob/master/doc/SPEC.md);
  - модель данных `approvals.type`: `hire_agent | approve_ceo_strategy | budget_override_required | request_board_approval`, статусы `pending | revision_requested | approved | rejected | cancelled` — [SPEC-implementation](https://github.com/paperclipai/paperclip/blob/master/doc/SPEC-implementation.md);
  - эндпоинты: list, get, create, `agent-hires`, approve, reject, request-revision, resubmit, linked issues, comments — [approvals API](https://github.com/paperclipai/paperclip/blob/master/docs/api/approvals.md);
  - права board: pause и resume любого агента, terminate (необратимо), переназначение задач, override бюджетов, создание агентов в обход approvals — [board approvals guide](https://github.com/paperclipai/paperclip/blob/master/docs/guides/board-operator/approvals.md);
  - «Approval gates are enforced, config changes are revisioned, and bad changes can be rolled back safely» — [README](https://github.com/paperclipai/paperclip/blob/master/README.md).
- **Бюджеты:**
  - «Token and cost tracking by company, agent, project, goal, issue, provider, and model. Scoped budget policies with warning thresholds and hard stops. Enforcement uses recorded spend; usage reporting and in-flight work can delay a stop» — [README](https://github.com/paperclipai/paperclip/blob/master/README.md);
  - пороги 80% (soft) и 100% (hard stop, автопауза), окно обнуляется 1-го числа по UTC — [costs API](https://github.com/paperclipai/paperclip/blob/master/docs/api/costs.md);
  - стоимость сообщают адаптеры, разбирая вывод агента; можно прислать и напрямую: `POST /api/companies/{companyId}/cost-events` с полями provider, model, inputTokens, outputTokens, costCents — [cost reporting](https://github.com/paperclipai/paperclip/blob/master/docs/guides/agent-developer/cost-reporting.md);
  - каскадное делегирование бюджетов — «TBD»;
  - «External revenue/expense tracking — future plugin. Token/LLM cost budgeting is core» — [SPEC](https://github.com/paperclipai/paperclip/blob/master/doc/SPEC.md).
- **Аудит:**
  - журнал append-only и неизменяемый;
  - логируются мутации задач, агентов, approvals, комментарии, изменения бюджетов и конфигурации компании — [activity API](https://github.com/paperclipai/paperclip/blob/master/docs/api/activity.md);
  - в таблице `activity_log` поле `actor_type: agent | user | system` — [SPEC-implementation](https://github.com/paperclipai/paperclip/blob/master/doc/SPEC-implementation.md).

**Рантаймы и адаптеры**
- Встроенные адаптеры — [adapters overview](https://github.com/paperclipai/paperclip/blob/master/docs/adapters/overview.md):
  - `claude_local`, `codex_local`;
  - `gemini_local` (экспериментальный), `kimi_local`;
  - `opencode_local` («multi-provider `provider/model`»), `cursor`, `pi_local`;
  - `hermes_local` и `hermes_gateway`, `openclaw_gateway`;
  - универсальные `process` и `http`;
  - внешние адаптеры подключаются как npm-плагины (например, `droid_local`).
- Минимальное требование к агенту — «be callable» — [SPEC](https://github.com/paperclipai/paperclip/blob/master/doc/SPEC.md).
- Примеры test-drive в README используют `ANTHROPIC_API_KEY`, `OPENAI_API_KEY` или `OPENROUTER_API_KEY` с `--harness opencode` — [README](https://github.com/paperclipai/paperclip/blob/master/README.md).
- В зависимостях `@paperclipai/server` — пакеты всех адаптеров и chat-адаптеры для Slack, Teams, GitHub, Discord и Telegram, а также `embedded-postgres`, `drizzle-orm`, `better-auth` и `svix` — [npm @paperclipai/server](https://registry.npmjs.org/@paperclipai/server).
- Песочницы: e2b, Cloudflare, Daytona, Modal, Novita, self-hosted Kubernetes — [README, Roadmap](https://github.com/paperclipai/paperclip/blob/master/README.md).

**API, БД, секреты, развёртывание**
- **API:**
  - «Paperclip exposes a RESTful JSON API for all control plane operations», базовый путь `http://localhost:3100/api`;
  - токены: agent API keys, короткоживущие run JWT (`PAPERCLIP_API_KEY`), сессионные cookie пользователей;
  - на мутирующих запросах во время heartbeat передаётся `X-Paperclip-Run-Id` — [API overview](https://github.com/paperclipai/paperclip/blob/master/docs/api/overview.md).
- Один и тот же API обслуживает UI и агентов, права определяются авторизацией — [SPEC](https://github.com/paperclipai/paperclip/blob/master/doc/SPEC.md).
- **Режимы доступа:** local trusted без аутентификации или authenticated через сессии Better Auth — [authentication](https://github.com/paperclipai/paperclip/blob/master/docs/api/authentication.md).
- **БД:**
  - PostgreSQL через Drizzle;
  - по умолчанию встроенный PostgreSQL, данные в `~/.paperclip/instances/default/db/`;
  - локально можно поднять PostgreSQL 17 в Docker, для продакшена — внешний Postgres (например, Supabase) — [database](https://github.com/paperclipai/paperclip/blob/master/docs/deploy/database.md);
  - в SPEC для dev упоминается PGlite — это расходится с README, где речь о встроенном PostgreSQL — [SPEC](https://github.com/paperclipai/paperclip/blob/master/doc/SPEC.md), [README](https://github.com/paperclipai/paperclip/blob/master/README.md).
- **Секреты:**
  - «Paperclip encrypts secrets at rest using a local master key»;
  - провайдеры `local_encrypted` (по умолчанию) и `aws_secrets_manager` — [secrets](https://github.com/paperclipai/paperclip/blob/master/docs/deploy/secrets.md).
- **Развёртывание:**
  - `npx paperclipai@latest onboard --yes`;
  - режимы привязки `--bind lan` и `--bind tailnet`, есть Docker;
  - «Open source. Self-hosted. No Paperclip account required»;
  - Paperclip Cloud — пока лист ожидания — [README](https://github.com/paperclipai/paperclip/blob/master/README.md).
- **Наблюдаемость:** OpenTelemetry и Sentry по желанию. Анонимная телеметрия «enabled by default», выключается через `PAPERCLIP_TELEMETRY_DISABLED=1` или `DO_NOT_TRACK=1` — [README](https://github.com/paperclipai/paperclip/blob/master/README.md).

**MCP и «не-кодовые» бизнес-агенты**
- «Coding and PR review fit alongside research, operations, content, and other work»; «Not only for code review» — [README](https://github.com/paperclipai/paperclip/blob/master/README.md).
- **MCP-шлюз:**
  - «Connect services such as GitHub, Notion, and Railway, or your own MCP server. Set gateway actions to Allowed, Ask first, or Off»;
  - в Roadmap отмечено как сделанное: «MCP Tool Gateway & Apps (governed tool access)» и «Secrets Manager with per-agent access» — [README](https://github.com/paperclipai/paperclip/blob/master/README.md);
  - проверки вызовов инструментов показываются в истории задачи: человек одобряет, отклоняет или сохраняет разрешение с ограниченной областью действия — [SPEC](https://github.com/paperclipai/paperclip/blob/master/doc/SPEC.md).
- Свежий коммит 2026-10-09: «feat(connections): add verified MCP providers and setup fixes (#15621)» — [commits](https://github.com/paperclipai/paperclip/commits/master).

**Активность и органичность популярности**
- **Метрики на 2026-10-09:** 99.1k звёзд, 16.7k форков, 463 watchers, 2.9k открытых issues, 3.4k открытых PR, 4,930 коммитов; на странице видны «12 alerts» в Security and quality — [repo](https://github.com/paperclipai/paperclip).
- В FAQ: «We've merged over 2,700 pull requests» — [README](https://github.com/paperclipai/paperclip/blob/master/README.md).
- **История коммитов:**
  - на 2026-02-20 видно 35 коммитов от `forgottendev` в соавторстве с `claude`, до 2026-02-15 коммитов нет — [until 02-20](https://github.com/paperclipai/paperclip/commits/master/?until=2026-02-20), [until 02-15](https://github.com/paperclipai/paperclip/commits/master/?until=2026-02-15);
  - 3–4 марта 2026 года — коммиты `cryppadotta` (часто с `claude`) и Numman Ali, релизы v0.2.2–v0.2.7, смена домена `paperclip.dev` на `paperclip.ing` — [until 03-04](https://github.com/paperclipai/paperclip/commits/master/?until=2026-03-04);
  - 8–9 октября 2026 года — коммиты `cryppadotta` и `devinfoley` в соавторстве с `Paperclip-Paperclip`, плюс боты; идут крупные рефакторинги ядра heartbeat — [commits](https://github.com/paperclipai/paperclip/commits/master).
- **Релизы в npm:**
  - `paperclipai` создан 2026-03-03, всего 1,763 версии;
  - по месяцам: март 94, апрель 178, май 115, июнь 205, июль 329, август 391, сентябрь 318;
  - последняя стабильная — 2026.1005.0 от 2026-10-06 — [npm](https://registry.npmjs.org/paperclipai).
- **Сторонний обзор:** проект запущен 2 марта 2026 года разработчиком @dotta; «over 71,000 stars and 13,400 forks as of late June 2026» (по сниппету поиска) — [fast.io review](https://fast.io/resources/paperclip-ai-review-2026/).

### Inferences
- **Недельный цикл на Paperclip** (схема):

  | Элемент цикла | Чем реализуется в Paperclip |
  |---|---|
  | Директор | CEO-агент |
  | Аналитик, маркетолог | Агенты в подчинении CEO |
  | Еженедельный запуск | Routine «Скаут: 50 идей» по cron, создаёт задачу |
  | Стадии цикла | Подзадачи по стадиям |
  | Гейты совета (бюджет smoke-test, домены, юр. вопросы, финальный GO) | Approvals `request_board_approval`, плюс режим Ask first для MCP-инструментов, которые тратят деньги (Директ, ЮKassa) |
  | Журнал решений | Activity log |

  Детерминированные стадии (сбор Wordstat, Директ, ГИР БО, формула unit-экономики) лучше не отдавать LLM-агенту. Их делает отдельный «сотрудник»-сервис на Go или Rails через адаптер `http`.
- **Семантика бюджетов не совпадает с задачей:**
  - окно месячное и считается в центах (по контексту — USD);
  - учитываются только LLM-расходы;
  - лестницу рекламного бюджета в рублях и предоплаты нужно вести во внешнем реестре.
- **Риск: бюджеты могут не работать с российскими моделями.** Cost events разбираются из вывода адаптера. Для OpenCode с собственным провайдером (YandexGPT или GigaChat) цен может не оказаться, и тогда hard stop просто не сработает. Вероятно, придётся самим отправлять cost events через `POST /cost-events` с пересчётом рублей в центы. Это вывод, не проверено.
- **Популярность.** Рост аномально быстрый (около 99k звёзд за ~7,5 месяцев). Но его подкрепляют сигналы реальной активности: тысячи PR и issues, 16.7k форков, сотни релизов в месяц. Соотношения форков и watchers к звёздам такие же, как у LangGraph (около 17% и 0,44%).

  Если сниппет fast.io верен, рост шёл не одним всплеском: около 71k к концу июня и 99k к октябрю. Органичность всё равно не доказана, потому что star-history и HN недоступны.

  Отдельный сигнал: 3.4k открытых PR при >2,700 смёрженных. Похоже, проверять входящие PR не успевают, и часть PR может быть сгенерирована агентами.
- **Риски эксплуатации:**
  - основная часть кода написана или соавторена AI;
  - ежедневные рефакторинги ядра heartbeat;
  - 300+ версий npm в месяц.

  Отсюда меры: зафиксировать версию, обновляться редко, держать бизнес-логику вне Paperclip (в своём сервисе) и пользоваться export/import компании как путём выхода.
- **Интеграция с Rails или Go проста:** REST по Bearer, адаптеры `http` и `process`, входящие триггеры routines. Работать придётся со вторым рантаймом (Node 24) рядом с основным стеком.

### Gaps
- Не проверено: число контрибьюторов, история звёзд (star-history), упоминания на HN и в новостях, скачивания npm, сведения об инвестициях Paperclip Labs, Inc.
- Документация читалась из папки `docs/` репозитория. Сайт docs.paperclip.ing недоступен, поэтому часть страниц могла отстать от кода. Пример: в API-доках есть примеры только для approvals типов `hire_agent` и `approve_ceo_strategy`. Как UI и API обрабатывают `request_board_approval` с произвольным payload — не проверено.
- Сохраняется ли стоимость для `opencode_local` с собственным OpenAI-совместимым провайдером — не проверено.
- Что именно скрывается за «12 alerts» в Security and quality — не проверено.

---

## 3. Deep-dive 2: Temporal (движок для собственного конвейера)

### Takeaway
Temporal — зрелая платформа durable execution под MIT: серверу v1.0.0 с 2020 года, свежий релиз v1.32.1 вышел 2026-10-07. У неё нативные SDK для Go и Ruby (gem вышел в GA 1.0.0 2025-09-29).

Из коробки она закрывает три из пяти требований на уровне «2»: расписания (Schedules), гейты (Signals и Updates с надёжным ожиданием) и аудит действий (Event History).

Оргструктуры агентов, бюджетов и UI для совета у неё нет. Поэтому это не «AI-компания», а надёжный хребет для конвейера и денег, которые пишешь сам.

### Cited Findings
- **Что это:**
  - «Temporal is a durable execution platform…»;
  - «originated as a fork of Uber's Cadence», разрабатывается Temporal Technologies;
  - локальный запуск — `temporal server start-dev`, Web UI на localhost:8233 — [README](https://github.com/temporalio/temporal).
- **Хранилище:**
  - Cassandra 3.11, 4.0 и 5.0.4+;
  - PostgreSQL 13.18, 14.15, 15.10, 16.6;
  - MySQL 5.7 и 8.0 (8.0.19+);
  - SQLite — «only for development and testing»;
  - расширенный поиск по выполнениям (advanced Visibility) работает на SQL-базах и Elasticsearch — [persistence](https://github.com/temporalio/documentation/blob/main/docs/encyclopedia/temporal-service/persistence.mdx).
- **Schedules** — [schedule](https://github.com/temporalio/documentation/blob/main/docs/encyclopedia/workflow/schedule.mdx):
  - «Schedules provide a more flexible and user-friendly approach than Temporal Cron Jobs»;
  - спецификации: интервал или календарь (классическая cron-строка либо JSON);
  - часовой пояс: по умолчанию UTC, можно указать имя пояса;
  - Pause (с полем notes), Overlap Policy, Catchup Window, Jitter, Backfill.
- В Ruby расписание создаётся `create_schedule` — [ruby schedules](https://github.com/temporalio/documentation/blob/main/docs/develop/ruby/workflows/schedules.mdx).
- **Сообщения** — [message passing](https://github.com/temporalio/documentation/blob/main/docs/encyclopedia/workflow-message-passing/workflow-message-passing.mdx), [sending messages](https://github.com/temporalio/documentation/blob/main/docs/encyclopedia/workflow-message-passing/sending-messages.mdx):
  - Signals — «asynchronous write requests»;
  - Updates — «synchronous, tracked write requests»: отправитель ждёт результат или исключение, есть валидаторы;
  - Queries — чтение состояния.
- **Event History** — «a complete and durable log of everything that has happened in the lifecycle of a Workflow Execution» — [event history](https://github.com/temporalio/documentation/blob/main/docs/encyclopedia/event-history/event-history.mdx).
- **Ruby SDK** — [sdk-ruby README](https://github.com/temporalio/sdk-ruby):
  - поддерживает Ruby 3.3, 3.4, 4.0;
  - «Only macOS ARM/x64 and Linux ARM/x64 are supported»;
  - раздел Rails и сэмпл `rails_app`;
  - не рекомендуется переиспользовать модели ActiveRecord как модели Temporal;
  - в dev и test код workflow нужно загружать заранее (`config.eager_load = true` или явный `require`), иначе ленивая загрузка Zeitwerk даёт ошибку;
  - объявление обработчиков `workflow_signal`, `workflow_query`, `workflow_update` и ожидание `Temporalio::Workflow.wait_condition`.
- **Релизы:**
  - gem `temporalio`: 0.1.0 (2023-03-23), 1.0.0 (2025-09-29), 1.9.0 (2026-09-14) — [RubyGems](https://rubygems.org/api/v1/versions/temporalio.json);
  - Go SDK v1.49.0 (2026-09-14) — [Go proxy](https://proxy.golang.org/go.temporal.io/sdk/@latest);
  - сервер v1.32.1 (2026-10-07) — [Go proxy](https://proxy.golang.org/github.com/temporalio/temporal/@latest);
  - Python SDK 1.34.0 (2026-09-30) — [PyPI](https://pypi.org/pypi/temporalio/json).
- **Активность:** 23.6k звёзд, 2.0k форков, 578 issues, 479 PR, 9,973 коммита, последний коммит 2026-10-09 — [repo](https://github.com/temporalio/temporal), [commits](https://github.com/temporalio/temporal/commits/main).

### Inferences
- **Отображение недельного цикла на Temporal:**

  | Элемент цикла | Реализация в Temporal |
  |---|---|
  | Запуск цикла | `WeeklyCycleWorkflow` по Schedule, например каждый понедельник в 06:00 по `Europe/Moscow` |
  | Скаут | LLM-вызов как activity (YandexGPT, GigaChat, vLLM) |
  | Обработка идей | Child workflow на каждую идею: стоп-факторы (включая проверку монополиста), Wordstat, прогноз Директа, ГИР БО, unit-экономика (`max CPC = цена × lifetime × конверсия ÷ 3` — детерминированная activity) |
  | Турнир финалистов | Отдельный workflow |
  | Гейт совета | Update с валидатором: решение принимает только член совета, личность передаётся в payload |
  | Smoke-test с лестницей бюджета | Таймеры и ступени бюджета, каждая ступень ждёт Update «повысить» |
  | Оплаты ЮKassa | Вебхук приходит в Rails и пересылается как Signal |

  Event History даёт технический аудит, но рубли и стоимость LLM нужно писать в свой реестр.
- **Что придётся дописать:** оргроли агентов, LLM-бюджеты и UI для совета. Технический Web UI совету не подойдёт. Это закрывает Paperclip или собственная админка на Rails.
- **Цена эксплуатации для solo-фаундера:** сервер Temporal, отдельная Postgres (SQLite только для dev), процессы-воркеры, ограничения детерминизма в коде workflow, особенности Rails (eager loading). При одном прогоне в неделю это может быть избыточно. Альтернатива — Rails-джобы с конечным автоматом в БД; в этом слое она не исследовалась.
- **В РФ:** в self-hosted варианте нет обязательных внешних SaaS. Код и модели под MIT, к LLM-провайдеру Temporal не привязан.

### Gaps
- Нет данных из первичной документации о доступности Temporal Cloud в РФ. Для self-hosted это не нужно.
- Аутентификация и RBAC в self-hosted Web UI, а также его лицензия — не проверены.
- Опыт продакшен-эксплуатации Ruby SDK в Rails (после GA в 2025-09) — не проверен.
- Шифрование payload (Data Converter или Codec) — раздел документации есть, детали не читались.

---

## 4. Deep-dive 3: Claude Agent SDK, headless `claude -p` и Claude Managed Agents

### Takeaway
**Claude Agent SDK** — это «Claude Code как библиотека» для Python и TypeScript. Из других языков (Ruby, Go) его вызывают подпроцессом `claude -p --output-format json`. В нём есть permissions, хуки, MCP, субагенты и лимит `max_budget_usd` на вызов.

**Claude Managed Agents** — хостинг агентов у Anthropic, в beta с апреля 2026 года. В нём есть почти весь нужный набор:
- cron-деплойменты с часовым поясом;
- жёсткий бюджет в долларах на прогон;
- подтверждение вызовов инструментов (`requires_action`);
- серверная история событий и webhooks.

Но Anthropic официально не обслуживает РФ и не поддерживает подмену моделей через шлюзы. Поэтому это референс дизайна и инструмент разработки, а не продакшен-рантайм.

### Cited Findings
- **Что такое Agent SDK** — [overview](https://code.claude.com/docs/en/agent-sdk/overview):
  - «The Agent SDK gives you the same tools, agent loop, and context management that power Claude Code, programmable in Python and TypeScript»;
  - для других языков: «run the CLI as a subprocess with the `-p` flag and `--output-format json`»;
  - возможности: встроенные инструменты, Hooks, Subagents, MCP, Permissions («which tools run automatically, which need approval»), Sessions, Skills, Plugins.
- **Python SDK** — [Python README](https://github.com/anthropics/claude-agent-sdk-python):
  - CLI Claude Code входит в пакет;
  - `query()` и `ClaudeSDKClient`;
  - собственные инструменты как in-process MCP-серверы (`@tool`, `create_sdk_mcp_server`);
  - хук `PreToolUse` может вернуть `permissionDecision: "deny"`;
  - `allowed_tools`, `disallowed_tools`, `permission_mode`, `can_use_tool`, `max_turns`.
- **Стоимость** — [cost tracking](https://code.claude.com/docs/en/agent-sdk/cost-tracking):
  - `total_cost_usd` и `modelUsage` / `model_usage`;
  - лимит `maxBudgetUsd` (TS) или `max_budget_usd` (Python), при превышении результат `error_max_budget_usd`;
  - «client-side estimates, not authoritative billing data… Do not bill end users or trigger financial decisions from these fields».
- **Условия использования** — [overview](https://code.claude.com/docs/en/agent-sdk/overview):
  - использование регулируется Anthropic Commercial Terms;
  - «Unless previously approved, Anthropic does not allow third party developers to offer claude.ai login or rate limits for their products».
- **Шлюзы:** «Anthropic doesn't endorse, maintain, or audit third-party gateway products, and doesn't support routing Claude Code to non-Claude models through any gateway» — [LLM gateway](https://code.claude.com/docs/en/llm-gateway).
- **Managed Agents: общее** — [MA overview](https://platform.claude.com/docs/en/managed-agents/overview):
  - «Pre-built, configurable agent harness that runs in managed infrastructure»;
  - beta-заголовок `managed-agents-2026-04-01`;
  - концепции Agent, Environment (облачная песочница Anthropic или self-hosted sandbox), Session, Events;
  - встроенные инструменты: Bash, файлы, web search и fetch, MCP;
  - «Scheduled execution… on a cron schedule»;
  - история событий хранится на сервере;
  - Zero Data Retention и HIPAA BAA не распространяются;
  - доступно и на Claude Platform on AWS.
- **Managed Agents: деплойменты по расписанию** — [MA deployments](https://platform.claude.com/docs/en/managed-agents/scheduled-deployments):
  - POSIX cron и часовой пояс IANA;
  - jitter до 15% интервала (от 5 секунд до 9 минут);
  - максимум 1 000 деплойментов на организацию;
  - `budget` копируется в каждую сессию, то есть ограничивает каждый прогон отдельно;
  - журнал `deployment_runs`, webhook-события, pause, unpause, archive, ручной запуск;
  - SDK-примеры на Python, TS, Go, Ruby и других языках.
- **Managed Agents: бюджеты** — [MA budgets](https://platform.claude.com/docs/en/managed-agents/budgets):
  - жёсткий потолок в центах USD (строкой);
  - list cost складывается из токенов, web search по $10 за 1 000 запросов и времени работы сессии по $0.08 в час;
  - при достижении сессия уходит в паузу со `stop_reason: budget_reached`;
  - на паузе принимаются только завершающие события, в том числе `user.tool_confirmation`;
  - у мультиагентной сессии один общий бюджет;
  - незакрытый запрос (`requires_action`) важнее капа.
- **Запуск:** Managed Agents стал публичной beta 8 апреля 2026 года (по сниппету поиска) — [IT Brief](https://itbrief.com.au/story/anthropic-launches-claude-managed-agents-in-public-beta), [AlternativeTo](https://alternativeto.net/news/2026/4/anthropic-launches-claude-managed-agents-to-accelerate-ai-agent-development-and-deployment).
- **РФ:** Россия не входит в список поддерживаемых стран — [supported countries](https://www.anthropic.com/supported-countries). gpt2giga предоставляет Anthropic-совместимые `POST /messages` и `/messages/count_tokens` поверх GigaChat — [gpt2giga](https://github.com/ai-forever/gpt2giga).

### Inferences
- **Как продакшен-рантайм в РФ не годится.** Мешают ограничение по стране и условия использования. Схема «Claude Agent SDK → gpt2giga → GigaChat» технически возможна, но Anthropic её не поддерживает. Новые возможности Claude Code могут ломаться на шлюзе, а gpt2giga сам предупреждает о неполной совместимости.
- **Ценность как референс.** Managed Agents показывает правильные примитивы для этого фаундера. Их стоит повторить у себя (в Paperclip или Temporal):
  - бюджет на каждый прогон, а не только месячный;
  - cron с часовым поясом;
  - статус `requires_action` и подтверждение инструментов;
  - пауза при исчерпании бюджета вместо завершения;
  - серверная история событий и webhooks о результатах запуска.
- **Для разработки** (фаундер пишет код с Claude Code) это вне рамок слоя. Юридический статус использования из РФ не проверялся.

### Gaps
- Не проверено:
  - можно ли легально использовать Managed Agents через юрлицо в поддерживаемой юрисдикции (например, в Казахстане или Армении) — это юридический вопрос, вне слоя;
  - полная тарификация MA сверх времени работы и web search;
  - есть ли у CLI флаг `--max-budget-usd`.
- Число контрибьюторов и дата последнего коммита для обоих SDK-репозиториев не проверены. Как замену использовал даты релизов: 2026-10-08.

---

## 5. Остальные кандидаты (по абзацу)

### CrewAI
CrewAI — Python-фреймворк под MIT: 59.5k звёзд, последний релиз 1.15.26 вышел 2026-10-08 ([repo](https://github.com/crewAIInc/crewAI), [PyPI](https://pypi.org/pypi/crewai/json)).

- **Роли:** даёт самую «человеческую» модель ролей — role, goal, backstory — и hierarchical process, где менеджер-агент делегирует задачи и проверяет результат; делегирование по умолчанию выключено ([hierarchical](https://github.com/crewAIInc/crewAI/blob/main/docs/edge/en/learn/hierarchical-process.mdx)).
- **Crews и Flows:** Crews отвечают за автономию, Flows — за управляемые событийные конвейеры ([README](https://github.com/crewAIInc/crewAI)).
- **Одобрения человеком:**
  - в OSS есть `@human_feedback` во Flows (с версии 1.8.0): исполнение встаёт на паузу, свободный текст ответа LLM сводит к одному из вариантов `emit` (approved, rejected, needs_revision), есть асинхронный `provider` ([HF in Flows](https://github.com/crewAIInc/crewAI/blob/main/docs/edge/en/learn/human-feedback-in-flows.mdx));
  - продакшен-вариант через вебхуки (Slack, Teams) есть только в Enterprise ([HITL](https://github.com/crewAIInc/crewAI/blob/main/docs/edge/en/learn/human-in-the-loop.mdx)).
- **Бюджеты:** денежных нет, только `max_iter`, `max_rpm`, `max_execution_time` ([agents](https://github.com/crewAIInc/crewAI/blob/main/docs/edge/en/concepts/agents.mdx)).
- **Расписание:** в README OSS-версии не упоминается.
- **Состояние:** checkpointing в JSON или SQLite ([checkpointing](https://github.com/crewAIInc/crewAI/blob/main/docs/edge/en/concepts/checkpointing.mdx)).
- **Провайдеры:** нативные SDK для OpenAI (с `base_url` и `OPENAI_BASE_URL`), Anthropic, Gemini, Azure, Bedrock; остальные — через LiteLLM ([llms](https://github.com/crewAIInc/crewAI/blob/main/docs/edge/en/concepts/llms.mdx)). Значит, YandexGPT и GigaChat (через gpt2giga) подключаются.
- **Телеметрия** включена по умолчанию, выключается `CREWAI_DISABLE_TELEMETRY` ([telemetry](https://github.com/crewAIInc/crewAI/blob/main/docs/edge/en/telemetry.mdx)).
- **AMP** — коммерческий control plane: «managed deployment, observability, governance, security, and enterprise support», on-premise или облако ([README](https://github.com/crewAIInc/crewAI)). Цены и доступность в РФ не проверены.

**Вердикт: референс.** Ролевые промпты, менеджер-делегатор и маршрутизация `emit` полезны как шаблоны. Как оркестратор не подходит: нет бюджетов и расписаний, стек только Python, продакшен-HITL платный.

### LangGraph
LangGraph — низкоуровневая библиотека для агентов с состоянием, под MIT: 43.0k звёзд, `langgraph` 1.2.14 вышел 2026-10-06 ([repo](https://github.com/langchain-ai/langgraph), [PyPI](https://pypi.org/pypi/langgraph/json)).

- **Одобрения человеком** — самый чистый механизм из всех кандидатов ([interrupts](https://github.com/langchain-ai/docs/blob/main/src/oss/langgraph/interrupts.mdx)):
  - `interrupt()` можно вызвать в любой ноде, пауза длится сколько угодно;
  - продолжение — `Command(resume=...)`;
  - состояние хранится в checkpointer по `thread_id`.
- **Память:** checkpointers для краткосрочной и stores для долговременной ([persistence](https://github.com/langchain-ai/docs/blob/main/src/oss/langgraph/persistence.mdx)).
- **Расписание (cron jobs)** есть только в LangSmith Deployment и Agent Server, расписание задаётся в UTC ([cron-jobs](https://github.com/langchain-ai/docs/blob/main/src/langsmith/cron-jobs.mdx)).
- **Agent Server** ([agent-server](https://github.com/langchain-ai/docs/blob/main/src/langsmith/agent-server.mdx), [PyPI langgraph-api](https://pypi.org/pypi/langgraph-api/json), [standalone](https://github.com/langchain-ai/docs/blob/main/src/langsmith/deploy-standalone-server.mdx)):
  - пакет `langgraph-api` под **Elastic-2.0**, нужны PostgreSQL и Redis;
  - при отдельном развёртывании требует `LANGGRAPH_CLOUD_LICENSE_KEY` и «egress to `https://beacon.langchain.com`… required for license verification and usage reporting if not running in air-gapped mode».
- **Self-hosted LangSmith** (трассировка, Studio) — «an add-on to the Enterprise plan» ([self-hosted](https://github.com/langchain-ai/docs/blob/main/src/langsmith/self-hosted.mdx)).
- **Бюджеты:** денежных бюджетов и оргструктуры нет (не найдено).

**Вердикт: референс.** Паттерн interrupt и checkpoint — образец для гейтов. Платформенная часть в РФ рискованна: лицензионный ключ и обязательная связь с серверами вендора в США. Стек только Python или JS.

### n8n
n8n — самая популярная платформа автоматизации: 206.8k звёзд, релиз 2.42.6 вышел 2026-10-09 ([repo](https://github.com/n8n-io/n8n), [npm](https://registry.npmjs.org/n8n)).

- **Лицензия:** Sustainable Use License — разрешено использование «only for your own internal business purposes». Файлы `.ee` распространяются под Enterprise License ([LICENSE.md](https://github.com/n8n-io/n8n/blob/master/LICENSE.md)).
- **Что есть:**
  - Schedule Trigger с интервалами в неделях и cron ([ScheduleTrigger](https://github.com/n8n-io/n8n/blob/master/packages/nodes-base/nodes/Schedule/ScheduleTrigger.node.ts));
  - HITL для вызовов инструментов AI Agent: Approve или Deny через Slack, Telegram или n8n Chat ([HITL](https://github.com/n8n-io/n8n-docs/blob/main/docs/build/integrate-ai/ai-examples/human-in-the-loop-for-tools.md));
  - MCP Server Trigger ([MCP trigger](https://github.com/n8n-io/n8n-docs/blob/main/docs/integrations/builtin/core-nodes/n8n-nodes-langchain.mcptrigger.md));
  - кредитив OpenAI с полями Base URL и custom header ([OpenAiApi.credentials.ts](https://github.com/n8n-io/n8n/blob/master/packages/nodes-base/credentials/OpenAiApi.credentials.ts)).
- **Критика вендора-конкурента** (по сниппету поиска; Pushary — заинтересованная сторона): ожидание без ответа «паркует» выполнение, а не отклоняет его, и запись об одобрении остаётся только в истории чата ([Pushary](https://pushary.com/human-in-the-loop-n8n)).
- **Чего нет:** оргструктуры агентов и бюджетов.

**Вердикт: нет как ядро.** Лицензия fair-code, второй low-code рантайм рядом с Rails или Go, нет оргструктуры и бюджетов. Допустим как временный «клей» для прототипа: approvals через Telegram, HTTP к API Яндекса.

### Gaps
- CrewAI: расписания и триггеры в AMP, цена AMP — не проверены. Причина: docs.crewai.com недоступен, лимит поиска исчерпан.
- LangGraph: условия получения лицензионного ключа для самостоятельного развёртывания, наличие бесплатного тарифа, цена — не проверены. В текущей документации standalone-сервера бесплатный «Lite» не упоминается.
- n8n: публичный REST API, хранение и шифрование credentials, набор `.ee`-функций (внешние секреты, лог-стриминг) — не проверены.

---

## 6. Вывод по слою: что взять готовым / что как референс / что писать самому

### Takeaway
**Готовым брать два продукта с чётким разделением ролей.**
- **Paperclip** — оболочка «компании и совета директоров»: оргчарт ролей, approvals, LLM-бюджеты, routines, журнал, UI. Брать в режиме пилота: self-hosted, с зафиксированной версией, без телеметрии, с RF-моделями через `opencode_local` или свои `http`-адаптеры.
- **Temporal** — durable-движок собственного детерминированного недельного конвейера и денежного контура, на Go или Ruby SDK.

**Как референсы:** Claude Managed Agents (бюджет на прогон, cron с часовым поясом, подтверждение инструментов), LangGraph (interrupt и checkpoint), CrewAI (role, goal, backstory, менеджер-делегатор, маршрутизация `emit`).

**Писать самому:** доменный конвейер, рублёвый денежный реестр с жёсткими лимитами, интеграции с Директом, Wordstat, ГИР БО и ЮKassa, мост «решение совета → действие с деньгами».

### Взять готовым (с условиями)
1. **Paperclip, пилот на 1–2 цикла.**
   - **Почему.** Единственный кандидат, у которого все пять обязательных возможностей — продуктовые сущности с UI совета (таблица 2). К тому же MIT, self-hosted и REST API для клиентов не на TypeScript ([SPEC](https://github.com/paperclipai/paperclip/blob/master/doc/SPEC.md)).
   - **Как подключить:**
     - роли CEO, аналитика и маркетолога запускать через `opencode_local` с OpenAI-совместимым endpoint (Yandex AI Studio, gpt2giga или vLLM);
     - детерминированные стадии делать отдельным «сотрудником» через адаптер `http`, который смотрит в сервис на Go или Rails;
     - гейты оформлять через `request_board_approval`;
     - MCP-инструменты Директа и ЮKassa перевести в режим Ask first;
     - стоимость RF-моделей отправлять самим через `POST /cost-events`;
     - выключить телеметрию.
   - **Чего не доверять Paperclip:** деньги на рекламу и предоплаты (его бюджеты — только LLM, окно месячное) и критичную логику конвейера (высокий темп изменений, молодой и во многом AI-сгенерированный код).
2. **Temporal**, если нужен надёжный конвейер с долгими ожиданиями.
   - **Почему.** Schedules с часовым поясом, ожидание решения совета через Updates и Signals, которое переживает сбои, таймеры для лестницы бюджета, Event History как технический аудит. Нативные SDK для Go и Ruby, GA для Ruby с 2025-09 ([sdk-ruby](https://github.com/temporalio/sdk-ruby), [RubyGems](https://rubygems.org/api/v1/versions/temporalio.json)).
   - **Что учесть:** это заметная операционная нагрузка для solo-фаундера (сервер, Postgres, воркеры, правила детерминизма). Если она окажется избыточной при одном прогоне в неделю, можно взять простую альтернативу — Rails-джобы с конечным автоматом. Альтернатива в этом слое не исследовалась.

### Использовать как референс
- **Claude Managed Agents и Agent SDK:**
  - бюджет на прогон с паузой `budget_reached` вместо завершения;
  - деплойменты по cron с часовым поясом и журналом запусков;
  - `requires_action` и `user.tool_confirmation`;
  - серверная история событий и webhooks;
  - `max_budget_usd` на вызов агента.

  Как продакшен-рантайм в РФ не годится: страна вне списка поддерживаемых, подмена моделей не поддерживается ([supported countries](https://www.anthropic.com/supported-countries), [LLM gateway](https://code.claude.com/docs/en/llm-gateway)).
- **LangGraph:** семантика `interrupt()` и `Command(resume)`, чекпоинты на каждом шаге. Agent Server не брать: Elastic-2.0, лицензионный ключ, обязательный исходящий доступ к beacon.langchain.com.
- **CrewAI:** шаблоны ролей (role, goal, backstory), иерархический менеджер с явно включаемым делегированием, `@human_feedback` с `emit`, где свободный текст совета сводится к одному из исходов.
- **n8n:** UX одобрения вызова инструмента через Telegram (approve или deny с параметрами вызова). Использовать максимум как временный «клей».

### Писать самому
- **Доменный конвейер:**
  - скаут на ~50 идей;
  - фильтр стоп-факторов, включая признак монополиста в нише;
  - коллекторы Wordstat, прогноза бюджета Директа и ГИР БО;
  - детерминированный расчёт unit-экономики (`max CPC = цена × lifetime × конверсия ÷ 3`);
  - турнир финалистов;
  - smoke-test с лестницей бюджета.

  Ни один кандидат этого не даёт (вывод из таблиц 2 и 4).
- **Рублёвый денежный реестр:**
  - жёсткие лимиты на каждую ступень лестницы и на кампанию;
  - идемпотентные операции с Директом;
  - учёт предоплат ЮKassa и вебхуков.

  Бюджеты Paperclip и Claude покрывают только LLM ([SPEC](https://github.com/paperclipai/paperclip/blob/master/doc/SPEC.md), [MA budgets](https://platform.claude.com/docs/en/managed-agents/budgets)).
- **Мост «решение совета → действие»:**
  - approval в Paperclip → Update или Signal в Temporal или вызов в своём сервисе;
  - запись личности одобрившего и суммы в собственный журнал;
  - журнал связывается с activity log Paperclip через ID прогона или задачи.
- **MCP-серверы или API-клиенты** для Яндекс Директа, Wordstat, ГИР БО и ЮKassa. Есть ли готовые — вне рамок слоя, не проверено.
- **Отчёт о стоимости RF-моделей** в Paperclip (`cost-events`) с пересчётом рублей в центы.

### Предлагаемая связка (вывод)
```
Совет (человек) ── Paperclip UI: approvals / costs / activity / org chart
        │
Paperclip (self-hosted, Postgres) ── агенты: CEO-директор, аналитик, маркетолог
        │   рантайм: opencode_local → OpenAI-compatible (Yandex AI Studio / gpt2giga→GigaChat / vLLM)
        │   MCP-шлюз: инструменты Директа/ЮKassa в режиме "Ask first"
        │   routine (cron, раз в неделю) → задача "недельный цикл"
        │
        └── агент-"конвейер" (http adapter) → сервис Go/Rails
                 └── Temporal (Schedules, Updates/Signals, Timers, Event History)
                        ├─ activities: Wordstat, прогноз Директа, ГИР БО, unit-экономика, турнир (LLM)
                        ├─ smoke-test: лестница бюджета (ступень = Update от совета)
                        └─ денежный реестр (₽) + ЮKassa webhooks → Signal
```

### Gaps
- **Риск-1.** Насколько надёжно YandexGPT и GigaChat вызывают инструменты внутри агентных харнессов (OpenCode) — не проверено. Если качество окажется слабым, LLM-роли придётся сводить к узким вызовам без автономии, и ценность Paperclip уменьшится.
- **Риск-2.** Органичность популярности Paperclip и его долгосрочная поддержка не проверены: нет star-history, HN, данных о финансировании и числа контрибьюторов.
- **Риск-3.** Доступ из РФ к npm, PyPI, Docker Hub и GitHub для обновлений — не проверен.
- **Не сравнивалось** в этом слое: Temporal против «Rails-джобы + конечный автомат» по стоимости владения для одного прогона в неделю.

---

## 7. Источники

**Paperclip**
- Репозиторий (метрики, лицензия, «12 alerts»): https://github.com/paperclipai/paperclip
- README: https://github.com/paperclipai/paperclip/blob/master/README.md (прочитан через raw.githubusercontent.com)
- Коммиты: https://github.com/paperclipai/paperclip/commits/master ; https://github.com/paperclipai/paperclip/commits/master/?until=2026-03-04 ; https://github.com/paperclipai/paperclip/commits/master/?until=2026-02-20 ; https://github.com/paperclipai/paperclip/commits/master/?until=2026-02-15
- SPEC: https://github.com/paperclipai/paperclip/blob/master/doc/SPEC.md
- SPEC-implementation: https://github.com/paperclipai/paperclip/blob/master/doc/SPEC-implementation.md
- Навигация документации: https://github.com/paperclipai/paperclip/blob/master/docs/docs.json
- API overview: https://github.com/paperclipai/paperclip/blob/master/docs/api/overview.md
- API approvals: https://github.com/paperclipai/paperclip/blob/master/docs/api/approvals.md
- API costs: https://github.com/paperclipai/paperclip/blob/master/docs/api/costs.md
- API activity: https://github.com/paperclipai/paperclip/blob/master/docs/api/activity.md
- API authentication: https://github.com/paperclipai/paperclip/blob/master/docs/api/authentication.md
- Board guide, approvals: https://github.com/paperclipai/paperclip/blob/master/docs/guides/board-operator/approvals.md
- Board guide, costs and budgets: https://github.com/paperclipai/paperclip/blob/master/docs/guides/board-operator/costs-and-budgets.md
- Board guide, activity log: https://github.com/paperclipai/paperclip/blob/master/docs/guides/board-operator/activity-log.md
- Board guide, org structure: https://github.com/paperclipai/paperclip/blob/master/docs/guides/board-operator/org-structure.md
- Agent guide, cost reporting: https://github.com/paperclipai/paperclip/blob/master/docs/guides/agent-developer/cost-reporting.md
- Core concepts: https://github.com/paperclipai/paperclip/blob/master/docs/start/core-concepts.md
- Adapters overview: https://github.com/paperclipai/paperclip/blob/master/docs/adapters/overview.md
- HTTP adapter: https://github.com/paperclipai/paperclip/blob/master/docs/adapters/http.md
- Process adapter: https://github.com/paperclipai/paperclip/blob/master/docs/adapters/process.md
- Secrets: https://github.com/paperclipai/paperclip/blob/master/docs/deploy/secrets.md
- Database: https://github.com/paperclipai/paperclip/blob/master/docs/deploy/database.md
- npm `paperclipai`: https://registry.npmjs.org/paperclipai
- npm `@paperclipai/server`: https://registry.npmjs.org/@paperclipai/server
- Обзор fast.io (по сниппету поиска; сам сайт недоступен): https://fast.io/resources/paperclip-ai-review-2026/
- OpenCode providers (харнесс адаптера `opencode_local`): https://github.com/sst/opencode/blob/dev/packages/web/src/content/docs/providers.mdx

**Claude Agent SDK и Managed Agents**
- Agent SDK overview: https://code.claude.com/docs/en/agent-sdk/overview
- Cost tracking: https://code.claude.com/docs/en/agent-sdk/cost-tracking
- LLM gateways: https://code.claude.com/docs/en/llm-gateway
- Managed Agents overview: https://platform.claude.com/docs/en/managed-agents/overview
- Scheduled deployments: https://platform.claude.com/docs/en/managed-agents/scheduled-deployments
- Session budgets: https://platform.claude.com/docs/en/managed-agents/budgets
- Python SDK: https://github.com/anthropics/claude-agent-sdk-python
- TypeScript SDK: https://github.com/anthropics/claude-agent-sdk-typescript
- PyPI `claude-agent-sdk`: https://pypi.org/pypi/claude-agent-sdk/json
- npm `@anthropic-ai/claude-agent-sdk`: https://registry.npmjs.org/@anthropic-ai/claude-agent-sdk
- npm `@anthropic-ai/claude-code`: https://registry.npmjs.org/@anthropic-ai/claude-code
- Поддерживаемые страны Anthropic: https://www.anthropic.com/supported-countries
- Дата запуска MA (по сниппету поиска): https://itbrief.com.au/story/anthropic-launches-claude-managed-agents-in-public-beta ; https://alternativeto.net/news/2026/4/anthropic-launches-claude-managed-agents-to-accelerate-ai-agent-development-and-deployment

**CrewAI**
- Репозиторий: https://github.com/crewAIInc/crewAI
- Коммиты: https://github.com/crewAIInc/crewAI/commits/main
- Лицензия: https://github.com/crewAIInc/crewAI/blob/main/LICENSE
- Human feedback в Flows: https://github.com/crewAIInc/crewAI/blob/main/docs/edge/en/learn/human-feedback-in-flows.mdx
- Human-in-the-loop: https://github.com/crewAIInc/crewAI/blob/main/docs/edge/en/learn/human-in-the-loop.mdx
- LLMs: https://github.com/crewAIInc/crewAI/blob/main/docs/edge/en/concepts/llms.mdx
- Hierarchical process: https://github.com/crewAIInc/crewAI/blob/main/docs/edge/en/learn/hierarchical-process.mdx
- Agents: https://github.com/crewAIInc/crewAI/blob/main/docs/edge/en/concepts/agents.mdx
- Checkpointing: https://github.com/crewAIInc/crewAI/blob/main/docs/edge/en/concepts/checkpointing.mdx
- Telemetry: https://github.com/crewAIInc/crewAI/blob/main/docs/edge/en/telemetry.mdx
- PyPI `crewai`: https://pypi.org/pypi/crewai/json

**LangGraph**
- Репозиторий: https://github.com/langchain-ai/langgraph
- Interrupts: https://github.com/langchain-ai/docs/blob/main/src/oss/langgraph/interrupts.mdx
- Persistence: https://github.com/langchain-ai/docs/blob/main/src/oss/langgraph/persistence.mdx
- Cron jobs: https://github.com/langchain-ai/docs/blob/main/src/langsmith/cron-jobs.mdx
- Agent Server: https://github.com/langchain-ai/docs/blob/main/src/langsmith/agent-server.mdx
- Standalone server: https://github.com/langchain-ai/docs/blob/main/src/langsmith/deploy-standalone-server.mdx
- Self-hosted LangSmith: https://github.com/langchain-ai/docs/blob/main/src/langsmith/self-hosted.mdx
- Hybrid: https://github.com/langchain-ai/docs/blob/main/src/langsmith/hybrid.mdx
- PyPI: https://pypi.org/pypi/langgraph/json ; https://pypi.org/pypi/langgraph-api/json ; https://pypi.org/pypi/langgraph-cli/json

**Temporal**
- Репозиторий: https://github.com/temporalio/temporal
- Коммиты: https://github.com/temporalio/temporal/commits/main
- Ruby SDK: https://github.com/temporalio/sdk-ruby
- Schedules: https://github.com/temporalio/documentation/blob/main/docs/encyclopedia/workflow/schedule.mdx
- Message passing: https://github.com/temporalio/documentation/blob/main/docs/encyclopedia/workflow-message-passing/workflow-message-passing.mdx
- Sending messages: https://github.com/temporalio/documentation/blob/main/docs/encyclopedia/workflow-message-passing/sending-messages.mdx
- Event History: https://github.com/temporalio/documentation/blob/main/docs/encyclopedia/event-history/event-history.mdx
- Persistence: https://github.com/temporalio/documentation/blob/main/docs/encyclopedia/temporal-service/persistence.mdx
- Ruby schedules: https://github.com/temporalio/documentation/blob/main/docs/develop/ruby/workflows/schedules.mdx
- RubyGems `temporalio`: https://rubygems.org/api/v1/versions/temporalio.json
- Go proxy: https://proxy.golang.org/go.temporal.io/sdk/@latest ; https://proxy.golang.org/github.com/temporalio/temporal/@latest ; https://proxy.golang.org/go.temporal.io/server/@v/v1.0.0.info
- PyPI `temporalio`: https://pypi.org/pypi/temporalio/json

**n8n**
- Репозиторий: https://github.com/n8n-io/n8n
- Коммиты: https://github.com/n8n-io/n8n/commits/master
- Лицензия: https://github.com/n8n-io/n8n/blob/master/LICENSE.md
- HITL для инструментов: https://github.com/n8n-io/n8n-docs/blob/main/docs/build/integrate-ai/ai-examples/human-in-the-loop-for-tools.md
- MCP Server Trigger: https://github.com/n8n-io/n8n-docs/blob/main/docs/integrations/builtin/core-nodes/n8n-nodes-langchain.mcptrigger.md
- Кредитив OpenAI (исходник): https://github.com/n8n-io/n8n/blob/master/packages/nodes-base/credentials/OpenAiApi.credentials.ts
- Schedule Trigger (исходник): https://github.com/n8n-io/n8n/blob/master/packages/nodes-base/nodes/Schedule/ScheduleTrigger.node.ts
- npm `n8n`: https://registry.npmjs.org/n8n
- Критика вендора (по сниппету поиска): https://pushary.com/human-in-the-loop-n8n

**РФ-провайдеры и прочее**
- Yandex AI Studio, OpenAI compatibility (по сниппету поиска): https://yandex.cloud/en/docs/ai-studio/concepts/openai-compatibility
- Habr о подключении к Yandex (по сниппету поиска): https://habr.com/ru/articles/1049322
- gpt2giga: https://github.com/ai-forever/gpt2giga
- Не вошли в шорт-лист: https://registry.npmjs.org/@mastra/core ; https://pypi.org/pypi/agent-framework/json ; https://pypi.org/pypi/agency-swarm/json
