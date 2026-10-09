# Claude Code как рантайм недельного цикла без человека за клавиатурой: расписание, одобрения совета, аудит, бюджеты, стоимость (на 2026-10-09)

> **Как читать.** «Не проверено» — нет подтверждения первичным источником. «По сниппету поиска» — факт взят только из сводки поисковой выдачи, страница не открывалась. «Предложение» — собственный синтез или рекомендация. «Оценка» — расчёт на явных допущениях. Документация Claude Code считана 2026-10-09 (страницы `code.claude.com/docs/en/<page>.md`). Актуальные версии на дату исследования: Claude Code 2.1.295 от 2026-10-08, Agent SDK (TypeScript) 0.3.295 от 2026-10-08 ([npm claude-code](https://registry.npmjs.org/@anthropic-ai/claude-code), [npm claude-agent-sdk](https://registry.npmjs.org/@anthropic-ai/claude-agent-sdk)). Цены — в USD, как в первоисточниках. docs.github.com и techcrunch.com из среды исследования недоступны: факты GitHub взяты из исходников документации в репозитории `github/docs`.

## 1. Сводная таблица механизмов

### Takeaway
Без человека за клавиатурой недельный цикл могут запускать четыре механизма: облачные routines, GitHub Actions по cron, свой cron/systemd с `claude -p` или Agent SDK и, с оговорками, Desktop scheduled tasks. Одобрение совета можно встроить тремя способами: PR-гейтом, Remote Control / Telegram-каналом с пересылкой запросов на разрешение и паузой `defer` / `canUseTool`. Аудит дают OpenTelemetry, транскрипты и логи Actions. Жёсткие денежные лимиты есть только на стороне биллинга: лимиты Console, лимиты подписки и месячные API-кредиты Max. Флаги CLI ограничивают лишь клиентскую оценку расходов.

### Cited Findings

**Расписание**

| Механизм | Что даёт | Где исполняется | Требования | Ограничения | URL |
|---|---|---|---|---|---|
| `/loop` и `CronCreate` (задачи внутри сессии) | Повтор промпта по cron или самоподбираемому интервалу, разовые напоминания; до 50 задач на сессию | Локально, внутри открытой сессии CLI | Любая аутентификация, в т.ч. API-ключ и Bedrock (для них `/loop` — рекомендованная замена `/schedule`) | Срабатывает только пока сессия открыта и простаивает; повторяющиеся задачи истекают через 7 дней; пропущенные запуски не догоняются; джиттер до 30 мин | [scheduled-tasks](https://code.claude.com/docs/en/scheduled-tasks), [feature-availability](https://code.claude.com/docs/en/feature-availability) |
| Routines (`/schedule`, claude.ai/code/routines) | Сохранённые промпт, репозитории, среда и коннекторы; триггеры: расписание, HTTP `/fire`, события GitHub (PR, release); каждый запуск — новая облачная сессия | Облако Anthropic (или self-hosted среда Team/Enterprise) | Только вход через подписку claude.ai (Pro, Max, Team, Enterprise); с API-ключом Console `/schedule` недоступен | Research preview; минимальный интервал — 1 час; запросов на разрешение нет, коннекторы пишут без спроса; 100 плановых запусков в час на аккаунт; тратит лимиты подписки | [routines](https://code.claude.com/docs/en/routines), [feature-availability](https://code.claude.com/docs/en/feature-availability) |
| Desktop scheduled tasks | Локальные задачи с доступом к файлам, локальным MCP и режимом разрешений на задачу | Ваш компьютер (Desktop-приложение; на Linux — beta, только Debian-семейство) | Подписка: Desktop не работает с API-ключом | Только пока приложение открыто и компьютер не спит; пропуск — один догоняющий запуск за 7 дней; в Manual-режиме запуск висит до одобрения | [desktop-scheduled-tasks](https://code.claude.com/docs/en/desktop-scheduled-tasks), [desktop-linux](https://code.claude.com/docs/en/desktop-linux) |
| GitHub Actions + `anthropics/claude-code-action` по `schedule` | Автоматический режим по cron или событию; выходы `execution_file`, `session_id`, `structured_output`; в `claude_args` проходят любые флаги CLI | Раннеры GitHub (или self-hosted) | API-ключ Console, `CLAUDE_CODE_OAUTH_TOKEN` (подписка Pro/Max/Team/Enterprise, токен из `claude setup-token`) или Bedrock/Vertex/Foundry | Cron задерживается и может выпадать в начале часа; job не дольше 6 ч; в публичном репо cron отключается после 60 дней без активности; тратит минуты Actions | [github-actions](https://code.claude.com/docs/en/github-actions), [action.yml](https://github.com/anthropics/claude-code-action/blob/main/action.yml), [github/docs: schedule](https://github.com/github/docs/blob/main/content/actions/reference/workflows-and-actions/events-that-trigger-workflows.md) |
| cron/systemd + `claude -p` или Agent SDK | Полный контроль: `--max-budget-usd`, `--max-turns`, `--permission-prompts none`, JSON с `total_cost_usd`, `--resume` | Свой сервер или контейнер | `--bare` — только API-ключ; без `--bare` работает `CLAUDE_CODE_OAUTH_TOKEN`; для продуктов на Agent SDK — API-ключ | Перезапуски, хранение транскриптов и алерты — на вас | [headless](https://code.claude.com/docs/en/headless), [cli-reference](https://code.claude.com/docs/en/cli-reference), [agent-sdk/overview](https://code.claude.com/docs/en/agent-sdk/overview) |
| `claude -p "…" --cloud <session-id>` | Внешний планировщик ставит сообщение в очередь уже идущей облачной сессии, например долгоживущего «директора» | Облако Anthropic | Аккаунт claude.ai | Только отправка: команда завершается, не дожидаясь ответа | [claude-code-on-the-web](https://code.claude.com/docs/en/claude-code-on-the-web) |
| Claude Projects | Координирующий диалог плюс параллельные облачные «треды», вкладки Routines и «Waiting on you» | Облако Anthropic | Public beta только Pro/Max, раскатывается постепенно | Не больше 200 новых тредов в сутки; лимиты подписки тратятся быстрее | [claude-projects](https://code.claude.com/docs/en/claude-projects) |

**Человек в контуре (HITL)**

| Механизм | Что даёт | Где исполняется | Требования | Ограничения | URL |
|---|---|---|---|---|---|
| `--permission-prompts none`, режимы `dontAsk` и `auto` | Прогон без ожидания человека: всё, что требует запроса, отклоняется; `AskUserQuestion` убирается | Headless | Claude Code v2.1.259+ | Отклонённое не выполняется — это стоп, а не одобрение | [headless](https://code.claude.com/docs/en/headless), [permission-modes](https://code.claude.com/docs/en/permission-modes) |
| Remote Control + push-уведомления | Ответ на запросы разрешений и `AskUserQuestion` с телефона или claude.ai; push «when actions required» | Локальный процесс `claude` | Только подписка, API-ключи не поддерживаются | Компьютер и процесс должны работать; серверный режим завершается после ~10 мин без сети | [remote-control](https://code.claude.com/docs/en/remote-control), [mobile](https://code.claude.com/docs/en/mobile) |
| Channels (Telegram, Discord, iMessage) + permission relay | События в работающую сессию; запрос разрешения уходит в Telegram с кнопками Allow/Deny или ответом `yes <id>` | Локальная сессия с `--channels` | claude.ai или API-ключ Console; на Team/Enterprise включает админ | Research preview; нужен Bun; принимаются только отправители из allowlist; в `-p` инструменты с вопросами отключены | [channels](https://code.claude.com/docs/en/channels), [channels-reference](https://code.claude.com/docs/en/channels-reference), [telegram server.ts](https://github.com/anthropics/claude-plugins-official/blob/main/external_plugins/telegram/server.ts) |
| Хук `PreToolUse` → `"defer"` | Пауза на вызове инструмента: процесс выходит с `stop_reason: "tool_deferred"`, совет отвечает вне Claude, затем `--resume` | Свой сервер (`claude -p`, SDK) | Нужен permission host | Только при одном вызове инструмента за ход; таймаута нет | [hooks](https://code.claude.com/docs/en/hooks) |
| Agent SDK: `canUseTool` + хук `PermissionRequest` | Программное одобрение через своего бота, уведомления в Slack, почту или push | Своё приложение | API-ключ | Callback может висеть бесконечно — для долгих ожиданий нужен `defer` | [agent-sdk/user-input](https://code.claude.com/docs/en/agent-sdk/user-input) |
| MCP-инструмент с `anthropic/requiresUserInteraction` | Инструмент всегда требует человека, даже в `bypassPermissions`; `--permission-prompt-tool` одобрить его не может | Любая поверхность | — | В `dontAsk` вызов отклоняется; запуски Desktop scheduled tasks стопорятся на каждом вызове | [mcp](https://code.claude.com/docs/en/mcp), [desktop-scheduled-tasks](https://code.claude.com/docs/en/desktop-scheduled-tasks) |
| PR как одобрение | Routine с GitHub-триггером `pull_request.closed` и фильтром «Is merged»; workflow на `pull_request: closed` с `merged == true`; environments с required reviewers | Облако Anthropic или раннеры GitHub | Claude GitHub App на репозитории | Required reviewers на GitHub Free/Pro/Team — только для публичных репо | [routines](https://code.claude.com/docs/en/routines), [github/docs: environments](https://github.com/github/docs/blob/main/content/actions/reference/workflows-and-actions/deployments-and-environments.md) |

**Наблюдаемость и аудит**

| Механизм | Что даёт | Где исполняется | Требования | Ограничения | URL |
|---|---|---|---|---|---|
| OpenTelemetry (метрики, события, трейсы — beta) | Метрики стоимости и токенов по модели, субагенту и навыку; события `tool_decision`, `tool_result`, `api_request`, `api_error`, `auth`, `mcp_server_connection` | Любая поверхность; для облачных сессий — через переменные среды или server-managed settings | Любая аутентификация; server-managed settings — только Team/Enterprise | Промпты, ответы и параметры инструментов по умолчанию скрыты; аномалии и алерты — на стороне SIEM | [monitoring-usage](https://code.claude.com/docs/en/monitoring-usage) |
| Транскрипты JSONL | Полная история сессии: `~/.claude/projects/<project>/<session-id>.jsonl` | Локально | — | Удаляются через 30 дней (`cleanupPeriodDays`); формат внутренний и меняется | [sessions](https://code.claude.com/docs/en/sessions), [claude-directory](https://code.claude.com/docs/en/claude-directory) |
| Сессии запусков routines + `/schedule why…` | Каждый запуск — полноценная сессия на claude.ai; Claude читает лог запуска | Облако | Подписка | Зелёный статус значит «без инфраструктурной ошибки», а не «задача выполнена» | [routines](https://code.claude.com/docs/en/routines) |
| Хуки (`command`, `http`) | POST JSON по каждому событию; `tool_use_id` совпадает с OTel | Где запущен Claude Code | — | Зависший `command`/`http` хук не блокирует вызов инструмента | [hooks](https://code.claude.com/docs/en/hooks) |
| Логи GitHub Actions | Лог прогона, `execution_file`, Step Summary | GitHub | — | `show_full_output` может вывести секреты в публичные логи | [action.yml](https://github.com/anthropics/claude-code-action/blob/main/action.yml) |
| Дашборды аналитики | Использование, расходы, вклад | claude.ai/analytics, platform.claude.com/claude-code | Только Team/Enterprise и Console, не Pro/Max | — | [analytics](https://code.claude.com/docs/en/analytics) |

**Бюджеты**

| Механизм | Что даёт | Где исполняется | Требования | Ограничения | URL |
|---|---|---|---|---|---|
| `--max-budget-usd` / `max_budget_usd` | Остановка прогона по оценке расходов; учитываются субагенты | `-p` и SDK | Любая | Клиентская оценка; лимит может быть превышен на один ответ и работу активных субагентов | [cli-reference](https://code.claude.com/docs/en/cli-reference), [agent-sdk/agent-loop](https://code.claude.com/docs/en/agent-sdk/agent-loop) |
| `--max-turns`, таймауты и `concurrency` workflow | Лимит ходов и времени | `-p`, Actions | Любая | Выход с ошибкой при достижении | [github-actions](https://code.claude.com/docs/en/github-actions) |
| Лимиты Console: потолок по tier, свой лимит организации, месячные лимиты workspace | Жёсткая остановка API: HTTP 429 или 400 до сброса | Серверная сторона Anthropic | API-ключ | Нельзя задать на Default Workspace; лимита на отдельный ключ не найдено | [rate-limits](https://platform.claude.com/docs/en/api/rate-limits), [workspaces](https://platform.claude.com/docs/en/build-with-claude/workspaces) |
| Лимиты подписки + usage credits | Окно 5 ч и недельный лимит (на Max — отдельно для Fable); дальше — оплата по API-тарифам с месячным потолком | claude.ai | Pro/Max (Team/Enterprise — лимиты места) | Квоты в токенах не публикуются | [support 11145838](https://support.claude.com/en/articles/11145838-using-claude-code-with-your-pro-or-max-plan), [support 12429409](https://support.claude.com/en/articles/12429409-extra-usage-for-paid-claude-plans) |
| Месячные API-кредиты Max ($100 у 5x, $200 у 20x) | Кредиты на API, Agent SDK и Managed Agents; без докупки и auto-reload запросы останавливаются | Console | Max или Team | Не работают в интерактивном Claude Code и для usage credits | [support 17154008](https://support.claude.com/en/articles/17154008-monthly-api-credits-for-max-and-team-plans) |
| `availableModels` / `deniedModels` | Запрет дорогих моделей | Managed settings | — | `deniedModels` — с v2.1.283 | [model-config](https://code.claude.com/docs/en/model-config) |

### Inferences
- Ни один механизм Claude Code не закрывает все четыре задачи сразу. Типовая связка (предложение) — расписание через routines или Actions, одобрение через PR или Telegram, аудит через OTel и логи, деньги через внешний шлюз из первого отчёта.
- Подписочные механизмы (routines, Remote Control, Desktop) удобнее, но их нельзя ограничить API-ключом и лимитами Console. Механизмы на API-ключе (Actions, `claude -p --bare`, SDK) получают жёсткий денежный потолок на стороне Anthropic.

### Gaps
- Публичные квоты недельных лимитов Pro/Max в токенах или часах на 2026 год не найдены.

## 2. Расписание: что запускает цикл без человека

### Takeaway
Для недельного цикла подходят три механизма: облачные routines (без включённого компьютера, только на подписке, research preview), GitHub Actions по cron (API-ключ или OAuth-токен подписки) и собственный планировщик с `claude -p` или Agent SDK. `/loop` и Desktop требуют живой сессии или включённого компьютера.

### Cited Findings
- Сравнение в документации: облачные routines работают без включённого компьютера и без открытой сессии, переживают перезапуски, но не видят локальные файлы (свежий clone), а MCP подключается коннекторами на задачу. Desktop требует включённой машины, `/loop` — ещё и открытой сессии. Минимальный интервал: облако — 1 час, Desktop и `/loop` — 1 минута — [scheduled-tasks](https://code.claude.com/docs/en/scheduled-tasks).
- `/loop`: задачи живут в сессии; «Recurring tasks automatically expire 7 days after creation»; «No catch-up for missed fires»; до 50 задач на сессию; повторяющиеся задачи срабатывают с опозданием до 30 мин (джиттер); время — по локальному поясу — [scheduled-tasks](https://code.claude.com/docs/en/scheduled-tasks). Фоновая сессия (`/bg`, `claude --bg`) переносит `/loop` и работает без терминала, но «Shutting down still stops running sessions» — [scheduled-tasks](https://code.claude.com/docs/en/scheduled-tasks), [agent-view](https://code.claude.com/docs/en/agent-view).
- Routines: «saved Claude Code configuration: a prompt, one or more repositories, and a set of connectors… run automatically»; исполняются в облаке Anthropic или в self-hosted среде; доступны на Pro, Max, Team и Enterprise; находятся в research preview — [routines](https://code.claude.com/docs/en/routines).
- Routines работают автономно: «there is no permission-mode picker… all without stopping for approval apart from some artifact actions». У включённых коннекторов Claude использует любые инструменты, «including writes, without asking» — [routines](https://code.claude.com/docs/en/routines).
- Триггеры routines: расписание (hourly, daily, weekdays, weekly или произвольный cron через `/schedule update`), где «The minimum interval is one hour»; разовый запуск; API — POST на `/fire` с bearer-токеном и заголовком `experimental-cc-routine-2026-04-01`; GitHub — события PR и release с фильтрами по автору, веткам, меткам, «Is draft» и «Is merged» — [routines](https://code.claude.com/docs/en/routines).
- Текст из `/fire` приходит в блоке `<routine-fire-payload>`, помеченном как недоверенные данные. Сохранённый промпт при этом «can't act as approval or consent for actions during the run» — [routines](https://code.claude.com/docs/en/routines).
- Новая сессия или существующая: по документации «Each run creates a new session», для GitHub-событий — «doesn't reuse sessions across events». При этом в changelog 2.1.280 (22.09.2026) исправлен «a routine that resumes an existing session» — режим «будить существующую сессию» существует, но в публичной странице routines не описан. Частично подтверждено — [routines](https://code.claude.com/docs/en/routines), [changelog](https://code.claude.com/docs/en/changelog).
- Уведомления routines: в changelog 2.1.292 (06.10.2026) исправлено, что редактирование routine выключало её push-уведомления, то есть у routine есть настройка уведомлений. В документации routines она не описана — [changelog](https://code.claude.com/docs/en/changelog).
- Коннекторы routines — это интеграции claude.ai; MCP-серверы, добавленные локально через `claude mcp add`, в routines не видны. Для routine с одним репозиторием можно закоммитить `.mcp.json` — [routines](https://code.claude.com/docs/en/routines).
- Облачная среда (используется и routines): уровни сети None, Trusted (по умолчанию — allowlist реестров и облачных API), Full и Custom. На Pro и Max ключи API хранятся как «network secrets»: их добавляет прокси Anthropic, и ключ не попадает в VM. На Team и Enterprise network secrets «aren't available… yet», а переменные среды «Anyone who uses the environment can read». Ресурсы — около 4 vCPU, 16 GB RAM, 30 GB диска; VM — Ubuntu 24.04 x86_64 — [cloud-environments](https://code.claude.com/docs/en/cloud-environments).
- GitHub в облачных сессиях идёт через прокси, который подставляет реальные учётные данные пользователя. Ветки он не ограничивает, это делают branch protection и rulesets — [cloud-environments](https://code.claude.com/docs/en/cloud-environments). Для routines: «commits and pull requests carry your GitHub user», а правило, которое ваш доступ может обойти, «doesn't block a run's push» — [routines](https://code.claude.com/docs/en/routines).
- Self-hosted среды (облачные сессии и routines на своей инфраструктуре) — public beta только для Team/Enterprise — [self-hosted-environments](https://code.claude.com/docs/en/self-hosted-environments).
- Desktop scheduled tasks: «Tasks only run while the desktop app is running and your computer is awake». Пропущенные за 7 дней запуски заменяются одним догоняющим. Режим разрешений задаётся на задачу, в Manual «the run stalls until you approve it». Задачу можно изолировать в git worktree — [desktop-scheduled-tasks](https://code.claude.com/docs/en/desktop-scheduled-tasks). Desktop работает только с подпиской — [feature-availability](https://code.claude.com/docs/en/feature-availability).
- GitHub Actions: при заданном `prompt` действие работает в automation mode на любом событии GitHub, «including a cron schedule». Для текстового промпта у Claude нет shell и GitHub API, пока их не разрешат через `--allowedTools` — [github-actions](https://code.claude.com/docs/en/github-actions).
- Аутентификация в Actions: `ANTHROPIC_API_KEY` или `CLAUDE_CODE_OAUTH_TOKEN` — «authenticates with your Claude subscription, available on Pro, Max, Team, and Enterprise plans», генерируется `claude setup-token`. «If you authenticate with an OAuth token, runs use your Claude subscription instead of API billing» — [github-actions](https://code.claude.com/docs/en/github-actions). `claude setup-token` выпускает OAuth-токен на год «For CI pipelines, scripts, or other environments where interactive browser login isn't available» — [authentication](https://code.claude.com/docs/en/authentication).
- Cron в GitHub: событие `schedule` «can be delayed during periods of high loads… High load times include the start of every hour… some queued jobs may be dropped» — [github/docs: schedule-delay](https://github.com/github/docs/blob/main/data/reusables/actions/schedule-delay.md). Запуск идёт только с ветки по умолчанию; «In a public repository, scheduled workflows are automatically disabled when no repository activity has occurred in 60 days» — [github/docs: events](https://github.com/github/docs/blob/main/content/actions/reference/workflows-and-actions/events-that-trigger-workflows.md).
- Headless: `--bare` пропускает автообнаружение хуков, навыков, MCP и CLAUDE.md; «bare mode doesn't use your subscription login» и «will become the default for `-p` in a future release». Без `--bare` прогон `-p` выполняет хуки и подключает `.mcp.json` проекта даже в недоверенной папке — [headless](https://code.claude.com/docs/en/headless).
- Порядок выбора учётных данных: облачный провайдер → `ANTHROPIC_AUTH_TOKEN` → `ANTHROPIC_API_KEY` → `apiKeyHelper` → `CLAUDE_CODE_OAUTH_TOKEN` → профили → вход через `/login`. Облачные сессии всегда используют подписку, `ANTHROPIC_API_KEY` в облачной среде её не заменяет — [authentication](https://code.claude.com/docs/en/authentication).
- Подписочные функции (облачные сессии, Desktop, routines, Remote Control) с API-ключом Console недоступны. Channels доступны и с ключом Console — [feature-availability](https://code.claude.com/docs/en/feature-availability).

### Inferences
- Под цикл «понедельник утром» лучше всего подходят routines или cron в GitHub Actions. Время ставится не ровно на :00 — у обоих механизмов в начале часа задержки (предложение).
- Routines исполняются от GitHub-личности основателя. Значит, в варианте на routines «merge = одобрение совета» не отличить технически от действия агента: он теоретически может смёржить PR сам. Через REST-прокси это, вероятно, возможно (не проверено). В GitHub Actions агент коммитит как `claude[bot]` (`bot_name` по умолчанию — [action.yml](https://github.com/anthropics/claude-code-action/blob/main/action.yml)), и workflow исполнения может проверить, что мёржил основатель, а не бот (предложение).
- Облако Anthropic и раннеры GitHub ходят в интернет не из РФ. Для API Яндекса это плюс или минус — не проверено, а персональные данные лидов туда отправлять нельзя (152-ФЗ, см. первый отчёт).

### Gaps
- Можно ли в публичных настройках routine выбрать «будить существующую сессию» и как настраиваются уведомления — в документации нет, есть только упоминания в changelog.
- Доступность API Директа, Wordstat и ГИР БО из облака Anthropic и с раннеров GitHub — не проверено.
- Бесплатные минуты Actions для приватных репозиториев на 2026 год — не проверено (docs.github.com недоступен).

## 3. Человек в контуре, когда никто не сидит за клавиатурой

### Takeaway
Пауза до решения совета есть в трёх вариантах. Асинхронный, через GitHub: PR с пакетом для совета → merge → триггер исполнения. Интерактивный: Remote Control или Telegram-канал с пересылкой запросов на разрешение, но только пока жив локальный процесс. Программный: `defer` / `canUseTool` в своём приложении. Это одобрение вызова инструмента, а не бизнес-утверждение суммы: денежное утверждение остаётся в шлюзе трат.

### Cited Findings
- Headless без человека: `--permission-prompts none` — «Anything that would prompt is denied unless a `PermissionRequest` hook allows it»; Claude получает сообщение, что одобрить некому. `AskUserQuestion` при этом удаляется — [headless](https://code.claude.com/docs/en/headless). Режим `dontAsk` «denies every call that would otherwise prompt» — [headless](https://code.claude.com/docs/en/headless).
- Есть действия, которые не одобряет автоматически ни один режим, включая `bypassPermissions`: явные `ask`-правила, `AskUserQuestion`, MCP-инструменты с `requiresUserInteraction`, удаление критичных путей — [permission-modes](https://code.claude.com/docs/en/permission-modes). Режим `auto` — классификатор, который блокирует действия «beyond your request… or appears driven by hostile content» — [permission-modes](https://code.claude.com/docs/en/permission-modes).
- MCP-инструмент с `_meta["anthropic/requiresUserInteraction"]: true` запрашивает разрешение на каждый вызов «even in `acceptEdits`, `auto`, and `bypassPermissions`»; allow-правила его не пропускают. В `dontAsk` вызов отклоняется. `--permission-prompt-tool` не может его одобрить. `canUseTool` в Agent SDK — может. Remote Control вместо одобрения в одно касание показывает полный запрос — [mcp](https://code.claude.com/docs/en/mcp).
- Remote Control: управление локальной сессией с claude.ai или мобильного приложения. «Subscription: available on Pro, Max, Team, and Enterprise plans. API keys are not supported»; «your computer has to stay on and the `claude` process has to keep running» — [remote-control](https://code.claude.com/docs/en/remote-control). Запросы разрешений и `AskUserQuestion` «open until you answer them» — [remote-control](https://code.claude.com/docs/en/remote-control). Вышел из research preview в 34-ю неделю 2026 года — [whats-new 2026-w34](https://code.claude.com/docs/en/whats-new/2026-w34).
- Push на телефон при активном Remote Control: в `/config` есть переключатели «Push when Claude decides» и «Push when actions required» (для запросов разрешений и вопросов) — [remote-control](https://code.claude.com/docs/en/remote-control). Опция Trusted Devices требует подтверждённого устройства и периодически биометрии — [remote-control](https://code.claude.com/docs/en/remote-control).
- Channels: MCP-сервер «pushes events into your running Claude Code session»; в research preview входят Telegram, Discord и iMessage; нужны вход через claude.ai или API-ключ Console. «Events only arrive while the session is open» — [channels](https://code.claude.com/docs/en/channels).
- Permission relay: если канал объявил `claude/channel/permission`, запрос разрешения уходит в чат. Ответ `yes <id>` / `no <id>` применяется, только если ID совпадает с открытым запросом; терминальный диалог тоже остаётся открытым, срабатывает первый ответ. «Anyone who can reply through the channel can approve or deny tool use in your session» — [channels-reference](https://code.claude.com/docs/en/channels-reference), [channels](https://code.claude.com/docs/en/channels). Relay покрывает одобрения инструментов вроде Bash, Write и Edit; доверие к проекту и согласие на MCP-сервер — только в терминале — [channels-reference](https://code.claude.com/docs/en/channels-reference).
- Официальный Telegram-плагин объявляет `'claude/channel/permission': {}`, рассылает запрос с кнопками «See more / Allow / Deny» всем `allowFrom` и разбирает ответы регуляркой `^(y|yes|n|no)\s+([a-km-z]{5})$`. Отправителей не из allowlist он отбрасывает — [telegram server.ts](https://github.com/anthropics/claude-plugins-official/blob/main/external_plugins/telegram/server.ts).
- В `-p` с каналами «tools that need terminal input, such as multiple-choice questions and plan mode approval, are disabled» — [channels](https://code.claude.com/docs/en/channels).
- `defer`: хук `PreToolUse` возвращает `permissionDecision: "defer"`, процесс завершается с `stop_reason: "tool_deferred"` и `deferred_tool_use`. Внешний процесс получает ответ и запускает `claude -p --resume <session-id>`, после чего хук возвращает `allow` с ответом в `updatedInput`. «There is no timeout or retry limit»; работает, только если в ходе один вызов инструмента — [hooks](https://code.claude.com/docs/en/hooks).
- Agent SDK: `canUseTool` срабатывает на запросы разрешений и на `AskUserQuestion`. «The callback can stay pending indefinitely»; для долгих ожиданий документация советует `defer`. Внешние уведомления (Slack, email, push) шлются через хук `PermissionRequest` — [agent-sdk/user-input](https://code.claude.com/docs/en/agent-sdk/user-input). Порядок проверки разрешений: хуки → deny → ask → режим → allow → `canUseTool`. Deny блокирует «even in `bypassPermissions` mode» — [agent-sdk/permissions](https://code.claude.com/docs/en/agent-sdk/permissions).
- PR как одобрение, официальные кирпичи:
  - routine с GitHub-триггером на `pull_request.closed` с фильтром «Is merged» — так в документации описан пример «Library port» — [routines](https://code.claude.com/docs/en/routines);
  - в Actions — `on: pull_request: types: [closed]` и `if: github.event.pull_request.merged == true` — [github/docs: events](https://github.com/github/docs/blob/main/content/actions/reference/workflows-and-actions/events-that-trigger-workflows.md);
  - environments с required reviewers: до 6 рецензентов, достаточно одного одобрения, есть «Prevent self-review». На GitHub Free/Pro/Team «required reviewers are only available for public repositories» — [github/docs: environments](https://github.com/github/docs/blob/main/content/actions/reference/workflows-and-actions/deployments-and-environments.md);
  - ожидание одобрения environment — до 30 дней, весь прогон workflow — до 35 дней — [github/docs: limits](https://github.com/github/docs/blob/main/content/actions/reference/limits.md).
- Реальный пример гейта «PR одобрен → исполнение»: Atlantis для Terraform. Требование `approved` «will prevent applies unless the pull request is approved by at least one person», `undiverged` запрещает apply при изменениях базовой ветки после plan — [Atlantis](https://github.com/runatlantis/atlantis/blob/main/runatlantis.io/docs/command-requirements.md). Готовых примеров «LLM-агент кладёт план в PR → merge → исполнение» поиск не нашёл. Ближайшее — `awo`, где merge разрешён только человеку-мейнтейнеру (по сниппету поиска) — [awo](https://github.com/supanut9/awo).
- Облачные сессии: на вопрос Claude можно ответить позже, «up to environment expiry». Сессия, которая ждёт одобрения вызова MCP-коннектора, считается неактивной и «can expire during that wait» — [claude-code-on-the-web](https://code.claude.com/docs/en/claude-code-on-the-web).

### Inferences
- Предложение. Совет утверждает две вещи по-разному.
  - План (пакет для совета, финалисты, GO на лестницу) — merge PR с телефона через GitHub. Это асинхронно, переживает дни ожидания и оставляет аудит в истории git.
  - Каждую трату — в собственном шлюзе (Telegram или Rails UI), как в первом отчёте. Пересылка разрешений через Telegram-канал Claude Code для денег не годится: она одобряет вызов инструмента без суммы и бизнес-контекста, работает только при живой локальной сессии и находится в research preview.
- Инструмент шлюза, который «запросить списание или ставку», стоит пометить `requiresUserInteraction` (предложение). Тогда в автономных прогонах (`dontAsk`, routines без человека) его вызов гарантированно не пройдёт без человека — это эшелонированная защита поверх одобрения в самом шлюзе.
- Промпт routine должен запрещать вопросы человеку и требовать, чтобы при неясности работа завершалась артефактом «нужен ответ совета». Иначе сессия будет ждать до истечения среды (предложение).

### Gaps
- Работает ли permission relay каналов в фоновых (`--bg`) и `-p` сессиях — не проверено: документация говорит только об отключении инструментов с вопросами в `-p`.
- Можно ли в облачной сессии смёржить PR через GitHub-прокси (REST) — не проверено.

## 4. Наблюдаемость и аудит

### Takeaway
Аудит собирается из трёх слоёв: события и метрики OpenTelemetry (решения по инструментам, вызовы MCP, стоимость), транскрипты сессий (локальные JSONL или сессии запусков на claude.ai) и логи GitHub Actions. Всё это сырые данные: хранение, алерты и неизменяемость журнала обеспечиваются снаружи.

### Cited Findings
- OTel: метрики — сессии, строки кода, PR, коммиты, стоимость (`cost`), токены по типам (input, output, cacheRead, cacheCreation) с атрибутами `model`, `query_source`, `agent.name`, `skill.name`, `mcp_server.name`. События: `user_prompt`, `tool_result`, `api_request`, `api_error`, `tool_decision`, `permission_mode_changed`, `auth`, `mcp_server_connection`, хуки, компакция, `subagent_completed`; трейсы — beta — [monitoring-usage](https://code.claude.com/docs/en/monitoring-usage).
- Скрытие по умолчанию: `OTEL_LOG_USER_PROMPTS`, `OTEL_LOG_ASSISTANT_RESPONSES`, `OTEL_LOG_TOOL_DETAILS`, `OTEL_LOG_TOOL_CONTENT`, `OTEL_LOG_RAW_API_BODIES` выключены. Без `OTEL_LOG_TOOL_DETAILS` имена пользовательских MCP-инструментов заменяются на `"mcp_tool"`, а пользовательские имена агентов — на `"custom"` — [monitoring-usage](https://code.claude.com/docs/en/monitoring-usage).
- `tool_decision` фиксирует `decision` (accept или reject) и `source`: `config`, `hook`, `user_permanent`, `user_temporary`, `user_abort`, `user_reject`. Через `tool_use_id` событие сопоставляется с данными хуков — [monitoring-usage](https://code.claude.com/docs/en/monitoring-usage).
- «OpenTelemetry events are the audit data source for Claude Code activity». При входе по API-ключу заполнены только `user.id` и `session.id`, личность добавляется через `OTEL_RESOURCE_ATTRIBUTES`. «Claude Code emits the raw event stream only. Anomaly detection… and alerting are the responsibility of your SIEM» — [monitoring-usage](https://code.claude.com/docs/en/monitoring-usage).
- Телеметрия облачных сессий настраивается через server-managed settings или переменные облачной среды. Переменные видят все пользователи среды, network secret к экспорту телеметрии не прикрепляется. Экспорт идёт через сеть сессии, поэтому коллектор должен быть в allowlist. События несут `ccr.session.id` — [monitoring-usage](https://code.claude.com/docs/en/monitoring-usage), [cloud-environments](https://code.claude.com/docs/en/cloud-environments).
- Транскрипты: `~/.claude/projects/<project>/<session-id>.jsonl`. «The entry format is internal to Claude Code and changes between versions»; для своих инструментов документация советует `/export` или скриптовые интерфейсы — [sessions](https://code.claude.com/docs/en/sessions). Файлы старше `cleanupPeriodDays` (по умолчанию 30 дней, минимум 1) удаляются — [claude-directory](https://code.claude.com/docs/en/claude-directory). `--no-session-persistence` вообще не пишет сессию на диск — [cli-reference](https://code.claude.com/docs/en/cli-reference).
- Agent SDK: экспорт OTLP-трейсов, метрик и событий — [agent-sdk/observability](https://code.claude.com/docs/en/agent-sdk/observability). Локальный диск контейнера теряется при рестарте, поэтому транскрипты стоит зеркалить через адаптер `SessionStore` — [agent-sdk/hosting](https://code.claude.com/docs/en/agent-sdk/hosting).
- Routines: каждый запуск открывается как полноценная сессия на claude.ai. «A green status… does not mean the task in your prompt succeeded». `/schedule why did my nightly review do nothing…` читает лог запуска с ошибками инструментов и отказами в разрешениях (v2.1.227+) — [routines](https://code.claude.com/docs/en/routines).
- Хуки: HTTP-хук отправляет JSON события POST-запросом на URL; заголовки подставляют только разрешённые переменные — [hooks](https://code.claude.com/docs/en/hooks). Таймаут: зависший `command`/`http`/`mcp_tool` хук на `PreToolUse` «doesn't block the tool call… so don't count on a stalled hook to act as a gate». Callback-хук Agent SDK по таймауту, наоборот, блокирует вызов — [hooks](https://code.claude.com/docs/en/hooks).
- GitHub Actions: выходы `execution_file`, `session_id`, `structured_output`, `conclusion`. `show_full_output` предупреждает: вывод может содержать секреты, «These logs are publicly visible in GitHub Actions» — [action.yml](https://github.com/anthropics/claude-code-action/blob/main/action.yml).
- Дашборды: claude.ai/analytics — только Team/Enterprise; platform.claude.com/claude-code — для Console, с расходами по дням — [analytics](https://code.claude.com/docs/en/analytics). На Pro и Max дашборда нет — [feature-availability](https://code.claude.com/docs/en/feature-availability). На Pro/Max/Team/Enterprise `/usage` с 35-й недели 2026 года показывает разбивку «Loops» по `/loop` и задачам по расписанию: число запусков и токены — [whats-new 2026-w35](https://code.claude.com/docs/en/whats-new/2026-w35).

### Inferences
- Минимальный аудит цикла (предложение):
  - OTel-логи в свой коллектор с `OTEL_LOG_TOOL_DETAILS=1` (иначе MCP-вызовы неразличимы);
  - копия JSONL-транскриптов и `execution_file` в неизменяемое хранилище до истечения 30 дней;
  - журнал PR и merge как журнал решений совета;
  - собственный append-only реестр денег в шлюзе.
- `OTEL_LOG_USER_PROMPTS` и `OTEL_LOG_RAW_API_BODIES` пишут в коллектор полные тексты. Если в них есть персональные данные, коллектор должен стоять в РФ (вывод из 152-ФЗ, см. первый отчёт).

### Gaps
- Как экспортировать OTel из routines на Pro/Max с аутентифицированным коллектором без раскрытия токена в переменных среды — документация отвечает только для Team/Enterprise (server-managed settings).

## 5. Бюджеты: какие лимиты на LLM-расходы действительно жёсткие

### Takeaway
Жёсткие лимиты — только серверные: лимиты организации и workspace в Console, лимиты подписки (окно 5 ч и неделя) с опциональным месячным потолком usage credits и месячные API-кредиты Max, которые без докупки просто заканчиваются. `--max-budget-usd` и `max_budget_usd` — полезный предохранитель на прогон, но это клиентская оценка, которая может перерасходоваться.

### Cited Findings
- `--max-budget-usd`: «Stop the run once estimated spend on API calls reaches this amount (print mode only)». Сверяется с клиентской оценкой; расходы субагентов учитываются; «Spend can pass the cap, so leave headroom». После `--resume` восстановленные суммы не считаются. Принудительная остановка — с v2.1.217 — [cli-reference](https://code.claude.com/docs/en/cli-reference). Превышение — до стоимости одного ответа плюс траты ещё работающих субагентов — [agent-sdk/agent-loop](https://code.claude.com/docs/en/agent-sdk/agent-loop). В SDK — `maxBudgetUsd` / `max_budget_usd`, результат с подтипом `error_max_budget_usd` — [agent-sdk/cost-tracking](https://code.claude.com/docs/en/agent-sdk/cost-tracking).
- `total_cost_usd` — «client-side estimates, not authoritative billing data… Do not bill end users or trigger financial decisions from these fields» — [agent-sdk/cost-tracking](https://code.claude.com/docs/en/agent-sdk/cost-tracking).
- Console: у тиров есть месячные потолки — Start $500, Build $1 000, Scale $200 000. После потолка API отвечает 429 до 1-го числа. Свой лимит ниже потолка даёт HTTP 400 «You have reached your specified API usage limits» — [rate-limits](https://platform.claude.com/docs/en/api/rate-limits).
- Месячные лимиты workspace: «Cap monthly spending for a workspace» плюс лимиты RPM и TPM. На Default Workspace лимит поставить нельзя, лимиты организации действуют всегда — [workspaces](https://platform.claude.com/docs/en/build-with-claude/workspaces). Для Claude Code в Console автоматически создаётся отдельный workspace — [costs](https://code.claude.com/docs/en/costs). Лимита расходов на отдельный API-ключ в документации не найдено.
- Подписка Pro/Max: «Both Pro and Max plans have a five-hour session limit and a weekly limit. Max plans also have a separate weekly limit for Fable», лимиты общие для Claude и Claude Code — [support 11145838](https://support.claude.com/en/articles/11145838-using-claude-code-with-your-pro-or-max-plan). Usage credits продолжают работу по стандартным API-тарифам «until you reach your monthly spend limit, or your balance runs out with auto-reload off»; лимит можно оставить «unlimited» — [support 12429409](https://support.claude.com/en/articles/12429409-extra-usage-for-paid-claude-plans).
- Routines при исчерпании лимита подписки без usage credits: «additional runs are rejected until your usage window resets». У часовых лимитов на число запусков «None… has overage» — [routines](https://code.claude.com/docs/en/routines). Треды Projects у лимита «waits and continues on its own when the limit resets» — [claude-projects](https://code.claude.com/docs/en/claude-projects).
- Max 5x и 20x получают $100 и $200 в месяц API-кредитов на Claude API, Batch API, Managed Agents и Agent SDK. На интерактивный Claude Code и usage credits они не действуют, не переносятся на следующий месяц. «If the organization has no other credits, API requests stop until your next monthly credits arrive» — [support 17154008](https://support.claude.com/en/articles/17154008-monthly-api-credits-for-max-and-team-plans).
- Team и Enterprise: лимит места со скользящим окном 5 ч и недельным окном; лимиты usage credits задаются на уровне организации, группы и участника — [costs](https://code.claude.com/docs/en/costs).
- Ловушка Fable. На части планов Fable списывается из usage credits. «In non-interactive mode with the `-p` flag… Claude Code never asks for consent… bills it without asking» — [model-config](https://code.claude.com/docs/en/model-config). Запретить модели можно через `availableModels` и `deniedModels` в managed settings (`deniedModels` — с v2.1.283) — [model-config](https://code.claude.com/docs/en/model-config).
- Выбор модели и кеш. Документация советует Sonnet для большинства задач, Opus — для сложных, `model: haiku` — для простых субагентов. Agent teams расходуют «approximately 7x more tokens» в режиме плана — [costs](https://code.claude.com/docs/en/costs). TTL кеша: на подписке в пределах лимита основной диалог кешируется на 1 час, на API-ключе и usage credits — на 5 минут. `ENABLE_PROMPT_CACHING_1H=1` включает часовой TTL — [prompt-caching](https://code.claude.com/docs/en/prompt-caching).
- Средний уровень расходов в корпоративных внедрениях — «around $13 per developer per active day and $150-250 per developer per month» — [costs](https://code.claude.com/docs/en/costs).
- В Actions расходы ограничиваются через `--max-turns` в `claude_args`, таймауты workflow и `concurrency` — [github-actions](https://code.claude.com/docs/en/github-actions).

### Inferences
- Предложение. Две линии обороны вокруг LLM-расходов:
  1. Жёсткая — отдельный workspace Console под «компанию» с месячным лимитом. Если работать в рамках месячных API-кредитов Max без докупки и auto-reload, то запросы сами остановятся на нуле.
  2. Мягкая — `--max-budget-usd` на каждый этап с запасом около 20%, `--max-turns` и `fallbackModel` без Fable.
- Usage credits на подписке включать с явным месячным лимитом, а не «unlimited»: иначе `-p`-прогоны на Fable спишут деньги без вопроса.
- Учёт в рублях и реклама остаются вне Claude (первый отчёт): эти лимиты — только про токены.

### Gaps
- Как именно ведёт себя `--max-budget-usd` при входе через подписку (оценка по прейскуранту при фактически «бесплатном» лимите) — не проверено.
- Лимиты расходов на отдельный API-ключ: в документации не найдены, вероятно, их нет.

## 6. Цены на 2026-10 и допустимость автоматизации на подписке

### Takeaway
Подписки: Pro $20 в месяц ($17 при оплате за год), Max 5x $100, Max 20x $200. Ключевые API-тарифы за MTok: Haiku 5.5 — $0,10 / $0,50, Sonnet 5.5 — $2 / $10, Opus 5.5 — $4 / $20, Fable 5.1 — $10 / $50; веб-поиск — $10 за 1 000 запросов. Подписку можно автоматизировать официальными механизмами: routines, Desktop, Actions с OAuth-токеном. Собственный харнесс на Agent SDK Anthropic велит вести на API-ключе.

### Cited Findings
- Подписки:
  - Pro: «$17 Per month with annual subscription discount ($200 billed up front). $20 if billed monthly». Max: «From $100», «Choose 5x or 20x more usage than Pro» — [claude.com/pricing](https://claude.com/pricing). Max 5x — $100 в месяц, Max 20x — $200 в месяц — [support 11049741](https://support.claude.com/en/articles/11049741-what-is-the-max-plan).
  - Team: место Standard — $20 в месяц при годовой оплате ($25 помесячно), Premium — $100 ($125). Enterprise self-serve: «Seat price + usage at API rates… US$20/seat/month», лимиты расходов на пользователя и организацию, журналы аудита — [claude.com/pricing](https://claude.com/pricing).
- API, $ за MTok (вход / запись в кеш на 5 мин / чтение кеша / выход) — [pricing](https://platform.claude.com/docs/en/about-claude/pricing):

  | Модель | Вход | Запись в кеш 5 мин | Чтение кеша | Выход |
  |---|---|---|---|---|
  | Fable 5.1 | 10 | 12,50 | 0,25 | 50 |
  | Opus 5.5 | 4 | 5 | 0,20 | 20 |
  | Opus 5 / 4.8 | 5 | 6,25 | 0,50 | 25 |
  | Sonnet 5.5 | 2 | 2,50 | 0,10 | 10 |
  | Sonnet 5 | 2 | 2,50 | 0,20 | 10 |
  | Haiku 5.5, промпт до 100K | 0,10 | 0,125 | 0,01 | 0,50 |
  | Haiku 5.5, промпт свыше 100K | 0,50 | 0,625 | 0,05 | 2,50 |
  | Haiku 4.5 | 1 | 1,25 | 0,10 | 5 |

  Batch API — скидка 50%. Опция `inference_geo: "us"` — ×1,1. Модели Claude 4.7+ используют новый токенизатор: «approximately 30% more tokens for the same text» — [pricing](https://platform.claude.com/docs/en/about-claude/pricing).
- Инструменты: веб-поиск — «$10 per 1,000 searches» плюс токены результатов; web fetch — без доплаты — [pricing](https://platform.claude.com/docs/en/about-claude/pricing). WebSearch в Claude Code «may issue up to eight backend searches per call»; интерактивная сессия ограничена 200 вызовами WebSearch — [tools-reference](https://code.claude.com/docs/en/tools-reference). Managed Agents: «$0.08 per session-hour for active runtime» — [claude.com/pricing](https://claude.com/pricing).
- Алиасы моделей в Claude Code: `fable` → Fable 5.1, `best` → Fable, где он доступен. Fable 5.1, Fable 5, Sonnet 5+, Haiku 5.5 и Opus 4.7+ работают с окном 1M по умолчанию. На Opus 5.5, Sonnet 5.5, Haiku 5.5 и Fable мышление выключить нельзя — [model-config](https://code.claude.com/docs/en/model-config).
- Допустимость автоматизации:
  - Consumer Terms (Free, Pro, Max) запрещают «Except when you are accessing our Services via an Anthropic API Key or where we otherwise explicitly permit it, to access the Services through automated or non-human means, whether through a bot, script, or otherwise» — [Consumer Terms, ред. от 08.10.2025](https://www.anthropic.com/legal/consumer-terms).
  - «Advertised usage limits for Pro and Max plans assume ordinary, individual usage of Claude Code and the Agent SDK». OAuth «is intended exclusively for purchasers of… subscription plans and is designed to support ordinary use of Claude Code and other native Anthropic applications». Разработчики продуктов, в том числе на Agent SDK, «should use API key authentication». Anthropic может применять меры «without prior notice» — [legal-and-compliance](https://code.claude.com/docs/en/legal-and-compliance).
  - Agent SDK: «Unless previously approved, Anthropic does not allow third party developers to offer claude.ai login or rate limits for their products, including agents built on the Claude Agent SDK» — [agent-sdk/overview](https://code.claude.com/docs/en/agent-sdk/overview).
  - Подписочная автоматизация, которую Anthropic предусмотрела явно: routines — «Put Claude Code on autopilot» — [routines](https://code.claude.com/docs/en/routines); `claude setup-token` для CI и скриптов — [authentication](https://code.claude.com/docs/en/authentication); `claude_code_oauth_token` в Actions — [github-actions](https://code.claude.com/docs/en/github-actions).
- Недельные лимиты ввели с 28.08.2025 из-за пользователей, которые держали Claude Code «running… continuously in the background, 24/7», и из-за перепродажи аккаунтов (по сниппету поиска) — [TechCrunch](https://techcrunch.com/2025/07/28/anthropic-unveils-new-rate-limits-to-curb-claude-code-power-users/).
- Россия: в списке поддерживаемых стран её нет, «Any country or region not listed is unsupported» — [supported countries](https://www.anthropic.com/supported-countries). Документация Claude Code при постоянной ошибке подключения допускает, что сервис «may not be available in your country» — [errors](https://code.claude.com/docs/en/errors).

### Inferences
- Для «LLM-компании» на подписке безопаснее всего укладываться в явно предусмотренные механизмы: routines, GitHub Actions с OAuth-токеном, Desktop. Самописный рантайм на Agent SDK, тем более круглосуточный, — на API-ключе. Это предложение на основе трёх процитированных документов. Граница «ordinary, individual usage» не определена количественно — серая зона.
- Цены 2026 года резко удешевили массовые этапы: стоп-фильтр на Haiku 5.5 стоит центы. Основная статья — длинные агентные сессии на Opus и Fable (см. раздел 8).

### Gaps
- Можно ли из РФ оплатить подписку, получить API-кредиты Max и пользоваться ими, не нарушая региональную политику, — не проверено и вне рамок (см. «Риски» первого отчёта).
- Насколько больше токенов даёт русский текст на новом токенизаторе — Anthropic не публикует.

## 7. Надёжность: сбои, таймауты, идемпотентность, возобновление

### Takeaway
Ни один режим не даёт exactly-once: routines могут ждать, отклоняться по лимитам и молча «зеленеть» при провале задачи; cron в GitHub задерживается и выпадает; `/loop` и Desktop теряют запуски, если машина выключена. Идемпотентность и возобновление нужно проектировать самому: стадии как конечный автомат в репозитории или БД, фиксированные ID сессий, `--resume`.

### Cited Findings
- `/loop`: пропущенное не догоняется; при `--resume` задачи `CronCreate` восстанавливаются, кроме истёкших, а самоподбираемый `/loop` не восстанавливается — [scheduled-tasks](https://code.claude.com/docs/en/scheduled-tasks).
- Desktop: при пробуждении один догоняющий запуск за последние 7 дней. Пропуск, если компьютер спал, предыдущий запуск ещё идёт или заняты другие задачи. Промпт должен сам защищаться от «запуска в 23:00 вместо 9:00» — [desktop-scheduled-tasks](https://code.claude.com/docs/en/desktop-scheduled-tasks).
- Routines:
  - запуски, назначенные ровно на начало часа, стартуют с опозданием в несколько минут; при превышении 100 плановых запусков в час запуск ждёт;
  - без подключения GitHub запуски пропускаются до 72 ч, затем routine выключается;
  - на паузе подписки routines стоят — [routines](https://code.claude.com/docs/en/routines);
  - свежие исправления в changelog: запуски, чья облачная сессия не стартовала, показывались как Succeeded (исправлено в 2.1.286, 30.09.2026); запуски висели «running» часами после завершения (2.1.292, 06.10.2026); «some older routines starting their runs without ever giving Claude their saved prompt» (2.1.295, 08.10.2026) — [changelog](https://code.claude.com/docs/en/changelog).
- Облачная VM: при простое ставится на паузу, затем может быть переработана. При повторном открытии восстанавливается история, но «Not restored: background work… such as subagents and shell commands» — [claude-code-on-the-web](https://code.claude.com/docs/en/claude-code-on-the-web), [cloud-environments](https://code.claude.com/docs/en/cloud-environments).
- GitHub Actions: job — до 6 ч, прогон workflow — до 35 дней с учётом ожиданий, одобрение environment — до 30 дней — [github/docs: limits](https://github.com/github/docs/blob/main/content/actions/reference/limits.md). Cron задерживается и может выпадать — [github/docs: schedule-delay](https://github.com/github/docs/blob/main/data/reusables/actions/schedule-delay.md). Выход действия `session_id` можно передать в `--resume` — [action.yml](https://github.com/anthropics/claude-code-action/blob/main/action.yml).
- `claude -p`:
  - SIGTERM завершает процесс с кодом 143, незаконченный ход остаётся без результата. `CLAUDE_CODE_RESUME_INTERRUPTED_TURN=1` продолжает прерванный ход при `--resume`;
  - фоновая работа после ответа ждёт не дольше 10 мин (`CLAUDE_CODE_PRINT_BG_WAIT_CEILING_MS`), при достижении `--max-budget-usd` останавливается;
  - событие `system/api_retry` сообщает категории ошибок `rate_limit`, `billing_error`, `account_on_hold`, `overloaded` и другие — [headless](https://code.claude.com/docs/en/headless);
  - `--session-id <uuid>` задаёт ID сессии заранее, `--resume` принимает ID или путь к JSONL — [cli-reference](https://code.claude.com/docs/en/cli-reference);
  - `--fallback-model` задаёт цепочку резервных моделей на случай перегрузки или недоступности основной — [cli-reference](https://code.claude.com/docs/en/cli-reference).
- Remote Control: в серверном режиме при отсутствии сети процесс сдаётся примерно через 10 минут и завершается — [remote-control](https://code.claude.com/docs/en/remote-control). Фоновые сессии переживают сон машины и рестарты супервизора, но не выключение — [agent-view](https://code.claude.com/docs/en/agent-view).
- Agent SDK: состояние на диске контейнера не переживает рестарт, поэтому нужны `SessionStore` и шаблоны ephemeral, long-running или hybrid — [agent-sdk/hosting](https://code.claude.com/docs/en/agent-sdk/hosting).

### Inferences
- Предложение по идемпотентности:
  - каждая стадия цикла (`scout` → `filter` → `enrich` → `tournament` → `board-pack` → `approved` → `smoke`) пишет результат в `cycles/2026-W41/<stage>.json` с хешем входов; перед стартом стадия проверяет, нет ли уже результата;
  - ID сессии стадии детерминирован (UUIDv5 от «неделя + стадия») и передаётся через `--session-id`, чтобы перезапуск продолжал ту же сессию через `--resume`;
  - всё денежное — через шлюз с ключами идемпотентности.
- Для каждой стадии нужен «сторож»: если к T+N часов нет артефакта, уходит алерт в Telegram. Зелёному статусу routine доверять нельзя (предложение).

### Gaps
- SLA и гарантии доставки для запусков routines не опубликованы (research preview).

## 8. Оценка стоимости недельного цикла (оценка)

### Takeaway
По API-тарифам октября 2026 года один недельный цикл стоит ориентировочно $13–27 в экономном и базовом сценариях и $54–81 в дорогом (Opus и Fable везде плюс накладные Claude Code и переделки). В месяц это около $60, $115 и $235–350 соответственно. Это порядок «пары рабочих дней разработчика» по корпоративному среднему Anthropic. Расходы на рекламу, API Яндекса, хостинг и минуты Actions не включены.

### Cited Findings (входные цены и константы)
- Цены за MTok: Haiku 5.5 — $0,10 / $0,50, чтение кеша $0,01; Sonnet 5.5 — $2 / $10, запись в кеш на 5 мин $2,50, чтение $0,10; Opus 5.5 — $4 / $20, запись $5, чтение $0,20; Fable 5.1 — $10 / $50, запись $12,50, чтение $0,25. Batch API — −50%. Веб-поиск — $0,01 за поиск — [pricing](https://platform.claude.com/docs/en/about-claude/pricing).
- WebSearch в Claude Code «may issue up to eight backend searches per call» — [tools-reference](https://code.claude.com/docs/en/tools-reference). Каждый веб-поиск тарифицируется как одно использование — [pricing](https://platform.claude.com/docs/en/about-claude/pricing). Что каждый поиск бэкенда — отдельное платное использование, — допущение (не проверено).
- Стартовый контекст сессии Claude Code в иллюстрации документации: системный промпт около 4 200 токенов плюс память, CLAUDE.md и описания навыков, всего порядка 8K. «Token counts are illustrative» — [context-window](https://code.claude.com/docs/en/context-window).
- Корпоративный ориентир — около $13 на разработчика за активный день — [costs](https://code.claude.com/docs/en/costs).

### Допущения и расчёт (оценка)
Модель расчёта для агентной сессии: N запросов × (кешированный контекст × цена чтения + новые входные токены × цена записи в кеш на 5 мин + выход × цена выхода). Для одиночных вызовов — вход без кеша. Все объёмы — в токенах нового токенизатора; на русском тексте реальные объёмы могут быть выше (не измерено).

| Этап | Модель (базовый сценарий) | Допущения | Стоимость, $ |
|---|---|---|---|
| A. Скаут, 50 идей с веб-поиском | Sonnet 5.5 | 100 запросов; средний кешированный контекст 60K; новых входных 3K и выход 1,2K на запрос; 30 вызовов WebSearch → от 30 до 240 платных поисков (допущение: каждый поиск бэкенда тарифицируется); около 60 страниц через WebFetch обрабатывает вспомогательная модель (~$0,1–0,5) | токены $2,55 + поиск $0,30–2,40 + WebFetch → **≈$3–5** (на Opus 5.5 токены — $5,10) |
| B. Стоп-фильтр | Sonnet 5.5 (или Haiku 5.5) | 50 одиночных вызовов: кешированная рубрика 5K, карточка идеи 2K, ответ 0,8K | **$0,63** (Haiku — $0,03; как 50 отдельных сессий Claude Code с накладными ~15K — $2,50) |
| C. Обогащение (Wordstat, прогноз Директа, отчётность) и юнит-экономика | Sonnet 5.5 | 15 идей после фильтра × 20 запросов; контекст 40K; новых 3K; выход 0,8K | **$5,85** (Opus 5.5 — $11,70) |
| D. Турнир | Opus 5.5 | 10 финалистов, 5 туров швейцарской системы = 25 пар × 2 порядка = 50 судейств; вход 14K (два досье по 6K и рубрика), выход 1,5K | **$4,30**; Batch API — $2,15; Sonnet 5.5 — $2,15 (batch $1,08); Fable 5.1 — $10,75; каждое судейство как субагент Claude Code с накладными ~15K — $8,05 на Opus и $20,13 на Fable |
| E. Пакет для совета | Opus 5.5 | 30 запросов; контекст 100K; новых 4K; выход 3K | **$3,00** (Sonnet 5.5 — $1,50) |
| F. Оркестрация «директором» за неделю | Opus 5.5 | 60 запросов; контекст 120K; новых 3K; выход 1K | **$3,54** (Sonnet 5.5 — $1,77) |
| G. Мониторинг smoke-теста, 7 дней | Sonnet 5.5 (или Haiku 5.5) | каждые 4 ч = 42 холодных запуска; старт 20K в кеш; 6 запросов; контекст 25K; новых 2K; выход 0,5K | **$5,25** (Haiku — $0,29; Sonnet раз в день — $0,88) |

| Сценарий | Состав | Неделя, $ | Месяц (×4,33), $ |
|---|---|---|---|
| Экономный | Sonnet 5.5 на A, C, E, F; Haiku 5.5 на B и G; турнир на Sonnet 5.5 через Batch API | ≈13,5 | ≈59 |
| Базовый | Как в таблице этапов выше (Sonnet 5.5 и Opus 5.5) | ≈26,6 | ≈115 |
| Дорогой | Opus 5.5 на A и C; судьи — Fable 5.1 как субагенты Claude Code; Sonnet 5.5 с накладными сессий на B; G каждые 4 ч | ≈54; с коэффициентом 1,5 на ретраи и переделки — ≈81 | ≈234–352 |

Сравнение с подписками (оценка):
- Базовый сценарий ($115 в месяц по API) по деньгам сопоставим с Max 5x ($100) и меньше Max 20x ($200). Но уложится ли он в недельные лимиты, проверить нельзя: квоты не публикуются.
- Max 5x даёт ещё $100 в месяц API-кредитов на API, Batch и Agent SDK. Экономный сценарий (≈$59 в месяц), запущенный через SDK или API-ключ, укладывается в эти кредиты, причём с естественным жёстким потолком — [support 17154008](https://support.claude.com/en/articles/17154008-monthly-api-credits-for-max-and-team-plans).
- Не входит в оценку: собственная интерактивная работа основателя (на подписке она делит те же лимиты), API Яндекса (Wordstat через Yandex Cloud), бюджет smoke-теста, минуты Actions, сервер.

### Inferences
- Главные рычаги экономии (предложение):
  - судейство турнира — пакетом через Batch API или `claude -p --bare` с минимальным контекстом, а не субагентами в длинной сессии: накладные Claude Code почти удваивают цену судейства на Opus;
  - фильтр и мониторинг — на Haiku 5.5;
  - Fable — только точечно и через явный `--model`, с запретом в `deniedModels` для остальных этапов.
- При таких суммах важнее не цена токенов, а риск неуправляемого цикла (ретраи, зацикливание, Fable в `-p`). Поэтому `--max-budget-usd` на стадию и лимит workspace важнее, чем оптимизация промптов.

### Gaps
- Объёмы токенов на этап — допущения без замеров. Нужен пилотный прогон с OTel, чтобы откалибровать (метрики `token` и `cost` с `agent.name` и `skill.name`).

## 9. Вывод: может ли Claude Code работать без человека за клавиатурой

### Takeaway
Технически да. Claude Code в 2026-10 умеет запускать недельный цикл без основателя (routines, cron в GitHub Actions, свой cron с `claude -p` или SDK), останавливаться до решения совета (PR и merge, Remote Control с push, Telegram-канал с пересылкой разрешений, `defer` / `canUseTool`), писать аудит (OTel, транскрипты, логи Actions) и ограничивать LLM-расходы (лимиты Console и workspace, лимиты подписки, API-кредиты Max, `--max-budget-usd`). Как единственный рантайм «компании» он не годится по четырём причинам ниже. Как харнесс для исследовательских стадий цикла — от скаута до пакета для совета — годится при четырёх условиях.

### Cited Findings (опорные факты вывода)
- Routines работают в облаке без включённого компьютера, но только на подписке, в research preview, с минимальным интервалом 1 час и без запросов на разрешение; коннекторы пишут без спроса — [routines](https://code.claude.com/docs/en/routines).
- GitHub Actions принимают и API-ключ, и OAuth-токен подписки, работают по cron и по событию merge — [github-actions](https://code.claude.com/docs/en/github-actions), [github/docs: events](https://github.com/github/docs/blob/main/content/actions/reference/workflows-and-actions/events-that-trigger-workflows.md).
- Одобрение в Claude Code — это решение о вызове инструмента. Удалённое одобрение через Remote Control и каналы требует живого локального процесса — [remote-control](https://code.claude.com/docs/en/remote-control), [channels](https://code.claude.com/docs/en/channels).
- Бюджеты на стороне клиента — оценки: «Do not… trigger financial decisions from these fields» — [agent-sdk/cost-tracking](https://code.claude.com/docs/en/agent-sdk/cost-tracking).
- Подписка рассчитана на «ordinary, individual usage»; продукты на SDK — на API-ключе — [legal-and-compliance](https://code.claude.com/docs/en/legal-and-compliance).
- Россия не входит в поддерживаемые страны — [supported countries](https://www.anthropic.com/supported-countries).

### Inferences
**Почему не единственный рантайм** (предложение):
1. **Деньги.** Одобрения Claude Code не несут суммы и бизнес-контекста, а бюджеты — клиентские оценки. Routines действуют от GitHub-личности основателя и пишут через коннекторы без спроса. Вывод первого отчёта о шлюзе трат вне LLM подтверждается: write-токены Директа и ЮKassa в среду Claude класть нельзя.
2. **Зрелость.** Routines и channels — research preview; в сентябре и октябре 2026 года в changelog исправляли ложный «Succeeded» и запуски без сохранённого промпта.
3. **Условия использования.** Подписочная автоматизация допустима в явно предусмотренных механизмах. Круглосуточный самописный харнесс на OAuth подписки — серая зона, для SDK нужен API-ключ.
4. **Доступ из РФ.** Облако Anthropic и раннеры GitHub решают вопрос «откуда IP», но не региональную политику, которая привязана к клиенту (первый отчёт). Рантайм может остановиться без предупреждения — нужен план выхода.

**Рекомендуемая пилотная связка** (предложение):
- **Расписание.** GitHub Actions, cron в понедельник в нестандартную минуту, приватный репозиторий `company-ops`. Аутентификация — API-ключ отдельного workspace Console с месячным лимитом и `--max-budget-usd` на каждый job. Альтернатива — routines, если основателю удобнее подписка и облачный UI. Каждая стадия — отдельный job: лимит 6 ч на job, артефакты стадий — в репозитории.
- **Совет.** Агент открывает PR «board-pack/2026-WNN» с пакетом для совета; основатель мёржит с телефона. Workflow на `pull_request: closed` + `merged == true` + проверка `merged_by` (не бот) переводит план в статус approved. Деньги тратит только Rails-шлюз после отдельного одобрения в Telegram. Инструмент «запросить трату» в MCP помечен `requiresUserInteraction`.
- **Аудит.** OTel в свой коллектор с `OTEL_LOG_TOOL_DETAILS=1`; `execution_file` и транскрипты — артефактами в неизменяемое хранилище; журнал merge — журнал решений.
- **Выход.** Промпты и навыки — переносимые файлы в репо; состояние цикла — в файлах или БД, а не в сессиях. Тогда смена рантайма на RubyLLM или российские модели из первого отчёта затронет только исполнителя стадий.

### Gaps
- Нужен пилотный прогон: реальные объёмы токенов, укладывается ли цикл в лимиты подписки, достижимость API Яндекса из облака и раннеров, поведение Telegram-relay в фоновых сессиях.

## 10. Источники

Документация Claude Code (считана 2026-10-09):
- https://code.claude.com/docs/llms.txt
- https://code.claude.com/docs/en/scheduled-tasks
- https://code.claude.com/docs/en/routines
- https://code.claude.com/docs/en/desktop-scheduled-tasks
- https://code.claude.com/docs/en/desktop-linux
- https://code.claude.com/docs/en/github-actions
- https://code.claude.com/docs/en/headless
- https://code.claude.com/docs/en/cli-reference
- https://code.claude.com/docs/en/claude-code-on-the-web
- https://code.claude.com/docs/en/cloud-environments
- https://code.claude.com/docs/en/self-hosted-environments
- https://code.claude.com/docs/en/claude-projects
- https://code.claude.com/docs/en/agent-view
- https://code.claude.com/docs/en/remote-control
- https://code.claude.com/docs/en/mobile
- https://code.claude.com/docs/en/channels
- https://code.claude.com/docs/en/channels-reference
- https://code.claude.com/docs/en/permission-modes
- https://code.claude.com/docs/en/mcp
- https://code.claude.com/docs/en/hooks
- https://code.claude.com/docs/en/monitoring-usage
- https://code.claude.com/docs/en/sessions
- https://code.claude.com/docs/en/claude-directory
- https://code.claude.com/docs/en/analytics
- https://code.claude.com/docs/en/costs
- https://code.claude.com/docs/en/model-config
- https://code.claude.com/docs/en/prompt-caching
- https://code.claude.com/docs/en/tools-reference
- https://code.claude.com/docs/en/context-window
- https://code.claude.com/docs/en/feature-availability
- https://code.claude.com/docs/en/authentication
- https://code.claude.com/docs/en/legal-and-compliance
- https://code.claude.com/docs/en/errors
- https://code.claude.com/docs/en/changelog
- https://code.claude.com/docs/en/whats-new/2026-w34
- https://code.claude.com/docs/en/whats-new/2026-w35
- https://code.claude.com/docs/en/agent-sdk/overview
- https://code.claude.com/docs/en/agent-sdk/user-input
- https://code.claude.com/docs/en/agent-sdk/permissions
- https://code.claude.com/docs/en/agent-sdk/cost-tracking
- https://code.claude.com/docs/en/agent-sdk/agent-loop
- https://code.claude.com/docs/en/agent-sdk/observability
- https://code.claude.com/docs/en/agent-sdk/hosting

Claude Platform, тарифы и условия:
- https://platform.claude.com/docs/en/about-claude/pricing
- https://platform.claude.com/docs/en/api/rate-limits
- https://platform.claude.com/docs/en/build-with-claude/workspaces
- https://claude.com/pricing
- https://support.claude.com/en/articles/11145838-using-claude-code-with-your-pro-or-max-plan
- https://support.claude.com/en/articles/12429409-extra-usage-for-paid-claude-plans
- https://support.claude.com/en/articles/11049741-what-is-the-max-plan
- https://support.claude.com/en/articles/17154008-monthly-api-credits-for-max-and-team-plans
- https://www.anthropic.com/legal/consumer-terms
- https://www.anthropic.com/supported-countries
- https://registry.npmjs.org/@anthropic-ai/claude-code
- https://registry.npmjs.org/@anthropic-ai/claude-agent-sdk

GitHub и примеры:
- https://github.com/anthropics/claude-code-action/blob/main/action.yml
- https://github.com/anthropics/claude-code-action/blob/main/docs/usage.md
- https://github.com/anthropics/claude-plugins-official/blob/main/external_plugins/telegram/server.ts
- https://github.com/anthropics/claude-plugins-official/tree/main/external_plugins/telegram
- https://github.com/github/docs/blob/main/content/actions/reference/workflows-and-actions/events-that-trigger-workflows.md
- https://github.com/github/docs/blob/main/data/reusables/actions/schedule-delay.md
- https://github.com/github/docs/blob/main/content/actions/reference/workflows-and-actions/deployments-and-environments.md
- https://github.com/github/docs/blob/main/content/actions/reference/limits.md
- https://github.com/runatlantis/atlantis/blob/main/runatlantis.io/docs/command-requirements.md
- https://github.com/supanut9/awo (по сниппету поиска)

Вторичные:
- https://techcrunch.com/2025/07/28/anthropic-unveils-new-rate-limits-to-curb-claude-code-power-users/ (по сниппету поиска)
- Первый отчёт: /home/user/office_research/research/ready-made-office.md (разделы «Деньги и токены», «Риски»)
