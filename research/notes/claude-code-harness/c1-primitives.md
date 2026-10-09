# C1. Примитивы Claude Code как рантайм «LLM-компании»: карта «потребность → примитив» на 2026-10-09

> **Как читать.** Факты взяты из официальной документации Claude Code (code.claude.com; страницы `.md` скачаны 2026-10-09) и из changelog. Последняя версия в changelog — v2.1.295 от 2026-10-08 ([changelog](https://code.claude.com/docs/en/changelog#2-1-295)). Пометки: «не проверено» — нет подтверждения первичным источником; «по сниппету поиска» — в этих заметках не встречается, WebSearch не использовался; «по листингу GitHub» — заголовок issue взят со страницы поиска GitHub Issues, сами issue не открывались, это сообщения пользователей, Anthropic их не подтверждала; «предложение» — синтез исследователя. Зрелость: «research preview», «public beta», «experimental» — так, как помечено в документации или еженедельном дайджесте; **GA\*** — в документации нет пометок preview/beta/experimental, но и явного объявления GA нет (классификация исследователя). Внутри раздела 2 каждый примитив разобран по схеме «Кратко / Факты / Выводы и предложения / Пробелы».

## 1. Карта «потребность компании → примитив → как → зрелость → ограничения»

«Как именно» — предложение исследователя, если не сказано иное. Ограничения и зрелость — факты со ссылками в разделе 2.

| # | Потребность компании | Примитив Claude Code | Как именно | Зрелость | Главные ограничения | URL |
|---|---|---|---|---|---|---|
| 1 | Устав: миссия, роли, стоп-факторы, правила совета — текстом | `CLAUDE.md` (managed / project / local) + `.claude/rules/*.md` + `@import` | Устав — в `./CLAUDE.md` (меньше 200 строк); правила по темам — в `.claude/rules/` с `paths:`; неизменяемый текст политики совета — в managed `/etc/claude-code/CLAUDE.md` | GA\* | Это контекст, а не принуждение («context, not enforced configuration»); противоречащие инструкции Claude выбирает произвольно | [memory](https://code.claude.com/docs/en/memory) |
| 2 | Неотменяемые запреты совета | Файловые managed settings `/etc/claude-code/managed-settings.json` + `permissions.deny` + ключи-«замки» | `allowManagedPermissionRulesOnly`, `allowManagedHooksOnly`, `allowManagedMcpServersOnly`, `strictPluginOnlyCustomization`, `strictKnownMarketplaces`, `disableBypassPermissionsMode`; файл принадлежит root, агенты работают от обычного пользователя | GA\*; файл работает у любого провайдера, серверные managed settings — только Team/Enterprise | Запрет действует только на вызовы через Claude Code; без managed settings mod может одобрить вызов поверх deny | [managed-settings](https://code.claude.com/docs/en/managed-settings#keys-only-a-managed-source-can-set), [feature-availability](https://code.claude.com/docs/en/feature-availability) |
| 3 | Директор, который раздаёт работу | Агент главной сессии: `claude --agent director` или `"agent": "director"` в настройках | `.claude/agents/director.md` с `tools: Agent(scout, analyst, marketer, judge), Read, Grep`, `initialPrompt`, `permissionMode` | GA\* | Allowlist `Agent(тип)` работает только у агента главной сессии | [sub-agents](https://code.claude.com/docs/en/sub-agents#restrict-which-subagents-can-be-spawned) |
| 4 | Скаут, аналитик, маркетолог | Субагенты `.claude/agents/*.md` | `tools`/`disallowedTools` (включая `mcp__server`), `model`, `effort`, `permissionMode`, inline `mcpServers`, `skills` (до 32), `memory: project`, `maxTurns`, `isolation: worktree` | GA\* | Субагент не видит историю диалога; плагинные субагенты игнорируют `hooks`, `mcpServers` и `permissionMode` | [sub-agents](https://code.claude.com/docs/en/sub-agents#supported-frontmatter-fields) |
| 5 | Критик-судья на другой модели | Поле `model` субагента или агента воркфлоу | Судья — `model: opus` с отдельным промптом и инструментами только для чтения; генераторы — `sonnet` или `haiku` | GA\* | Работники — только сессии Claude; на не-Claude модели через шлюз маршрутизация не поддерживается; судья другого вендора возможен лишь как MCP-инструмент | [agents](https://code.claude.com/docs/en/agents), [llm-gateway](https://code.claude.com/docs/en/llm-gateway) |
| 6 | SOP каждой стадии цикла | Skills (`SKILL.md`); кастомные команды слиты со skills | По skill на стадию: `disable-model-invocation: true`, `arguments`, `context: fork` + `agent: <роль>`, `allowed-tools`, `hooks` во frontmatter, `scripts/` и справочные файлы рядом | GA\* | Skill — инструкция: Claude может перестать ей следовать; после компакции остаются первые 5 000 токенов каждого skill (всего 25 000) | [skills](https://code.claude.com/docs/en/skills) |
| 7 | Детерминированные расчёты: max CPC, HHI, юнит-экономика | Скрипты в skill + `` !`cmd` `` + `allowed-tools: Bash(${CLAUDE_SKILL_DIR}/scripts/… *)` или свой MCP-инструмент | Формула живёт в `scripts/unit_econ.py`; LLM только вызывает её и интерпретирует результат | GA\* | Падение внедрённой команды прерывает весь вызов skill; таймаут Bash — 2 минуты | [skills](https://code.claude.com/docs/en/skills#inject-dynamic-context) |
| 8 | Конвейер «~50 идей → стоп-фильтр → данные → турнир» | Dynamic workflows: JS-скрипт с `agent()`, `pipeline()`, `parallel()`, `phase()` | Сохранённые воркфлоу `.claude/workflows/*.js` вызываются как `/имя`; вход — через `args` | Запущены как research preview 2026-05-28; на текущей странице пометки нет; на Pro включаются в `/config` | Нельзя спросить совет посреди прогона; по умолчанию 16 параллельных агентов (до 256); не больше 1 000 агентов за прогон; возобновление — только в той же сессии | [workflows](https://code.claude.com/docs/en/workflows#behavior-and-limits) |
| 9 | Турнир финалистов: попарный LLM-суд | Воркфлоу: `parallel()` судей + `agent({schema})` | Сетку пар задаёт скрипт; `Math.random`/`Date.now` запрещены, поэтому seed передаётся через `args`; позиции A/B переставляются; судья — другая модель Claude | Как в п. 8 | Невалидный структурный вывод повторяется до 5 раз; прогон крупнее 25 агентов или 1,5 млн токенов получает только предупреждение | [workflows](https://code.claude.com/docs/en/workflows#what-the-saved-script-looks-like) |
| 10 | Карточки идей и оценки в строгом формате | `--json-schema`, `outputFormat` в SDK, `agent({schema})` | JSON Schema для карточки идеи, вердикта стоп-фильтра, решения судьи | GA\* | Ключевое слово `format` не проверяется | [headless](https://code.claude.com/docs/en/headless#get-structured-output) |
| 11 | Еженедельный запуск цикла | Внешний cron или systemd + `claude -p`/SDK; альтернативы — Desktop scheduled tasks, routines, `/loop` | Таймер запускает `claude --bare -p …` по стадиям | `-p` — GA\*; routines — research preview | Routines работают в облаке, со свежим клоном, без локальных файлов, минимальный интервал — 1 час; `/loop` живёт внутри сессии и истекает через 7 дней | [scheduled-tasks](https://code.claude.com/docs/en/scheduled-tasks#compare-scheduling-options), [routines](https://code.claude.com/docs/en/routines) |
| 12 | Гейты совета: бюджет, домены, юридика, финальный GO | `permissions.ask`; `_meta["anthropic/requiresUserInteraction"]` на MCP-инструментах; PreToolUse `ask`/`defer`; `canUseTool` в SDK; relay разрешений через channels; Remote Control | Пишущие инструменты шлюза трат помечены `requiresUserInteraction` и требуют человека при каждом вызове — даже в режимах auto и bypass; в `-p` — `defer`, внешний UI и `--resume`; в SDK — `canUseTool` с выходом в Telegram или веб | GA\*; channels — research preview | `dontAsk` такие вызовы отклоняет; `--permission-prompt-tool` не может их одобрить; `defer` работает, только если в ходе один вызов инструмента; журнала решений совета нет | [mcp](https://code.claude.com/docs/en/mcp#require-approval-for-a-specific-tool), [hooks](https://code.claude.com/docs/en/hooks#defer-a-tool-call-for-later) |
| 13 | Жёсткий рублёвый бюджет рекламы | Частично: `permissions.deny` на прямые пишущие инструменты + PreToolUse-хук, который читает внешний реестр | Хук отказывает, если ступень лестницы исчерпана; с v2.1.295 — `onFailure: "block"` | Хуки — GA\*; `onFailure` вышел 2026-10-08 и пока описан только в changelog | Без `onFailure: "block"` хук пропускает действие при таймауте, падении, неверном пути, exit 1 или сбое HTTP; поле `if` — «best-effort»; mod может перекрыть блок хука | [hooks](https://code.claude.com/docs/en/hooks#exit-code-output), [changelog](https://code.claude.com/docs/en/changelog#2-1-295) |
| 14 | Бюджет на LLM-токены | `--max-budget-usd` / `maxBudgetUsd`; лимиты воркфлоу; статус-строка | Лимит на каждый прогон стадии; дешёвые модели для скаута | GA\* | Это клиентская оценка в USD; лимит превышается на стоимость одного ответа и работающих субагентов; «Do not … trigger financial decisions from these fields» | [cli-reference](https://code.claude.com/docs/en/cli-reference#cli-flags), [cost-tracking](https://code.claude.com/docs/en/agent-sdk/cost-tracking) |
| 15 | Минимальные привилегии ролей | `tools`/`disallowedTools`, правила `mcp__server__tool`, режим `dontAsk`, inline `mcpServers` | Скауту — чтение и веб; аналитику — MCP Wordstat и RFSD только на чтение; маркетологу — генерация лендинга без доступа к деньгам | GA\* | Запись с уточнением в `disallowedTools`, например `Bash(git push *)`, снимает инструмент целиком; правила Bash ловят команду только в том виде, как она записана | [sub-agents](https://code.claude.com/docs/en/sub-agents#available-tools), [permissions](https://code.claude.com/docs/en/permissions#mcp) |
| 16 | Секреты: токены Директа, ключ ЮKassa | `sandbox.credentials` (deny/mask), `CLAUDE_CODE_SUBPROCESS_ENV_SCRUB=1`, `userConfig.sensitive` у плагина, `${VAR}` в `.mcp.json` | Пишущих секретов в окружении Claude Code нет вообще; у агентов — только токены чтения | GA\*; маскирование через `network.tlsTerminate` — experimental | Окружение наследуют Bash, хуки и MCP-серверы; встроенного списка запретов нет | [sandboxing](https://code.claude.com/docs/en/sandboxing#protect-credentials) |
| 17 | Изоляция исполнения | Sandbox для Bash (уровень ОС, allowlist доменов); `isolation: worktree`; devcontainer или VM | Sandbox включён, `allowedDomains` — только API данных; весь процесс — в контейнере | GA\* | Sandbox покрывает только shell, а хуки, MCP-серверы, статус-строка и mods работают вне его; TLS не инспектируется; на нативной Windows sandbox нет | [sandboxing](https://code.claude.com/docs/en/sandboxing#what-runs-outside-the-sandbox) |
| 18 | Аудит действий агентов | Хуки PreToolUse, PostToolUse, PermissionRequest, PermissionDenied, SubagentStop → JSONL или HTTP; события OpenTelemetry; транскрипты | HTTP-хук пишет в собственный append-only журнал; OTel-событие `tool_decision` показывает источник решения | Хуки, OTel-метрики и логи — GA\*; трейсы — beta | Логи на той же машине не защищены от подмены; транскрипты по умолчанию удаляются через 30 дней (`cleanupPeriodDays`); алертинг — на вас | [monitoring-usage](https://code.claude.com/docs/en/monitoring-usage#audit-security-events), [hooks](https://code.claude.com/docs/en/hooks#defer-a-tool-call-for-later) |
| 19 | База знаний и память ролей | Auto memory; `memory: project` у субагента; `.claude/rules/` | Эвристики аналитика — в `.claude/agent-memory/analyst/` под git, совет видит изменения | GA\* | Auto memory привязана к машине; загружаются первые 200 строк или 25 KB `MEMORY.md`; для реестра идей и решений не годится | [memory](https://code.claude.com/docs/en/memory#auto-memory) |
| 20 | Упаковка «компании» с версиями | Плагин (skills, agents, hooks, MCP, workflows, output styles, monitors, `bin/`) + приватный git-маркетплейс | `company@ourmarket`; версии — через `version` или `ref`/`sha`; каналы — `#stable` | Плагины — GA\*, с 2025-10-09 | Правила разрешений плагин нести не может: из его `settings.json` действуют только `agent` и `subagentStatusLine`; автообновление меняет проверенные файлы | [plugins/components](https://code.claude.com/docs/en/plugins/components#default-settings), [plugins/security](https://code.claude.com/docs/en/plugins/security) |
| 21 | Тестирование SOP и регрессии | `claude plugin eval` + skill-creator + `/skill-doctor` + `claude plugin validate` | Кейсы вида «стоп-фильтр вызывает RFSD до вердикта» (`tool_order`), «запуск кампании не вызывается никогда» (`tool_used`, max 0); моки MCP Wordstat и Директа с `expect` | `plugin eval` — GA\* с v2.1.269 (2026-09-11) | Каждый прогон и каждый LLM-грейдер — реальный вызов модели за ваш счёт; по умолчанию 3 прогона на кейс | [plugin-evals](https://code.claude.com/docs/en/plugin-evals) |
| 22 | Панель совета: расходы и статус | Статус-строка с `refreshInterval`, `/workflows`, agent view, панели mods, OTel → дашборд | Скрипт статус-строки читает рублёвый реестр шлюза и лимиты подписки | Статус-строка — GA\*; agent view — research preview; mods — с 2026-10-01 | Это отображение, а не учёт; `cost.total_cost_usd` — клиентская оценка | [statusline](https://code.claude.com/docs/en/statusline#available-data) |
| 23 | Тон и формат ответов ролей | Output styles (главная сессия); системные промпты субагентов | Proactive или Concise — для директора; тон маркетолога — в промпте его субагента | GA\* | Output style не действует на субагентов, кроме fork, и ничего не гарантирует | [output-styles](https://code.claude.com/docs/en/output-styles) |
| 24 | Реакция на внешние события: вебхук ЮKassa, ответ совета | Channels: MCP-сервер, который присылает события в сессию (Telegram, Discord, iMessage); API-триггер routines | Канал Telegram с relay разрешений: совет одобряет вызов с телефона | Research preview | Только плагины из allowlist, свои — через dev-флаг; нужна авторизация claude.ai или Console; события приходят только в открытую сессию | [channels](https://code.claude.com/docs/en/channels), [channels-reference](https://code.claude.com/docs/en/channels-reference#relay-permission-prompts) |
| 25 | Программируемый рантайм компании | `claude -p` + Agent SDK (Python/TS) | В SDK: `canUseTool`, хуки-колбэки, `maxBudgetUsd`, `outputFormat`, `agents`, `plugins`, `sessionStore` | GA\*; SDK — по Commercial Terms | Только модели Claude; для продуктов на SDK — API-ключ, а не логин claude.ai | [agent-sdk/overview](https://code.claude.com/docs/en/agent-sdk/overview), [legal](https://code.claude.com/docs/en/legal-and-compliance) |
| 26 | «Команда» агентов, которая обсуждает между собой | Agent teams | Лид, напарники, общий список задач, почтовые ящики | Experimental, выключены по умолчанию | In-process напарники не восстанавливаются при возобновлении; одна команда на сессию; вложенных команд нет | [agent-teams](https://code.claude.com/docs/en/agent-teams#limitations) |
| 27 | Многодневная работа в облаке | Projects на claude.ai/code | Поток задач превращается в параллельные облачные «треды» | Public beta (Pro, Max) | Недоступно для Team и Enterprise; в облаке нет локальных файлов | [claude-projects](https://code.claude.com/docs/en/claude-projects) |
| 28 | Эскалация трудного решения директором | Advisor tool | Директор советуется с более сильной моделью | Experimental | Только Anthropic API; нет на Bedrock, Vertex, Foundry | [advisor](https://code.claude.com/docs/en/advisor) |

## 2. Примитивы: ключевые факты

### 2.1. Skills и команды: SOP каждой стадии

#### Кратко
Каждую стадию недельного цикла можно оформить как SOP-skill: ручной запуск, изолированный субагент, встроенные скрипты, свои хуки. Но skill остаётся инструкцией, а не гарантией. Детерминизм дают скрипты, хуки и воркфлоу, а не текст skill.

#### Факты
- Кастомные команды слиты со skills: «Custom commands have been merged into skills». `.claude/commands/deploy.md` и `.claude/skills/deploy/SKILL.md` одинаково создают `/deploy`; старые файлы команд продолжают работать ([skills](https://code.claude.com/docs/en/skills)). В changelog: v2.1.3 от 2026-01-09 — «Merged slash commands and skills… with no change in behavior» ([changelog](https://code.claude.com/docs/en/changelog#2-1-3)).
- Skills следуют открытому стандарту Agent Skills (agentskills.io). Claude Code расширяет его управлением вызовом, запуском в субагенте и внедрением динамического контекста ([skills](https://code.claude.com/docs/en/skills)). За пределами Claude Code (claude.ai, Skills API) допустимы только поля `name`, `description`, `license`, `compatibility`, `metadata`, `allowed-tools`, иначе загрузка падает с ошибкой ([skills](https://code.claude.com/docs/en/skills#frontmatter-reference)).
- Где лежат skills: enterprise (каталог managed settings), personal `~/.claude/skills/`, project `.claude/skills/`, вложенные каталоги, `--add-dir`, плагин (`/плагин:skill`), skills, синхронизированные с аккаунта claude.ai ([skills](https://code.claude.com/docs/en/skills#where-skills-live)).
- Поля frontmatter: `name`, `description`, `when_to_use`, `argument-hint`, `arguments`, `disable-model-invocation`, `user-invocable`, `allowed-tools`, `disallowed-tools`, `model`, `effort`, `context: fork`, `agent`, `background`, `hooks`, `paths`, `shell`, `metadata`. Неизвестное поле молча игнорируется. Если YAML не парсится, skill загружается без полей ([skills](https://code.claude.com/docs/en/skills#frontmatter-reference)).
- Постепенное раскрытие (progressive disclosure). Описание skill всегда в контексте, полный текст загружается при вызове. Справочные файлы читаются по необходимости, скрипты исполняются, а не загружаются. Совет документации — держать `SKILL.md` короче 500 строк ([skills](https://code.claude.com/docs/en/skills#add-supporting-files)).
- Бюджет листинга skills — 1% окна контекста; запасное значение — 8 000 символов (`SLASH_COMMAND_TOOL_CHAR_BUDGET`). При переполнении первыми теряют описания самые редко вызываемые skills. Описание вместе с `when_to_use` обрезается до 1 536 символов ([skills](https://code.claude.com/docs/en/skills#skill-descriptions-are-cut-short), [env-vars](https://code.claude.com/docs/en/env-vars)).
- Жизненный цикл. Отрендеренный `SKILL.md` остаётся в контексте на следующих ходах и заново не перечитывается. После автокомпакции Claude Code возвращает первые 5 000 токенов каждого вызванного skill, всего не больше 25 000 токенов; старые skills могут выпасть целиком ([skills](https://code.claude.com/docs/en/skills#skill-content-lifecycle)).
- Аргументы: `$ARGUMENTS`, `$0…$N`, именованные `arguments: [issue, branch]` ([skills](https://code.claude.com/docs/en/skills#pass-arguments-to-skills)). В одном сообщении можно связать до шести skills ([commands](https://code.claude.com/docs/en/commands)).
- `` !`команда` `` выполняется до отправки skill модели. Ошибка прерывает весь вызов skill. Запрос разрешения при этом не показывается: deny-правило или любой ответ, кроме allow, прерывает вызов (вне auto mode). Таймаут — 2 минуты, как у Bash ([skills](https://code.claude.com/docs/en/skills#inject-dynamic-context)).
- `allowed-tools` выдаёт разрешения только на ход, в котором skill вызван, и не ограничивает набор инструментов. Workspace trust это поле не проверяет, даже в `-p` в недоверенной папке, поэтому skills из репозитория нужно ревьюить. При `allowManagedPermissionRulesOnly` (v2.1.282+) поле игнорируется ([skills](https://code.claude.com/docs/en/skills#pre-approve-tools-for-a-skill)). Переменные `${CLAUDE_SKILL_DIR}` и `${CLAUDE_PROJECT_DIR}` подставляются и в тело, и в `allowed-tools`. Поэтому встроенный скрипт можно разрешить без запроса ([skills](https://code.claude.com/docs/en/skills#available-string-substitutions)).
- `disable-model-invocation: true` не даёт Claude вызывать skill самостоятельно, исключает его предзагрузку в субагентов и (с v2.1.196) запуск из scheduled task. Если Claude всё же попробует, Claude Code заблокирует вызов ([skills](https://code.claude.com/docs/en/skills#control-who-invokes-a-skill)).
- `model` в skill переопределяет модель до конца текущего хода. При `context: fork` поле задаёт модель форкнутого субагента ([skills](https://code.claude.com/docs/en/skills#frontmatter-reference)).
- `context: fork` запускает skill в новом субагенте типа `agent` (Explore, Plan, general-purpose или свой). История диалога субагенту недоступна. По умолчанию он работает в фоне (с v2.1.218), а в `-p` и SDK ход ждёт результата. У фонового форка набор инструментов уже. Его правки не попадают в checkpoints ([skills](https://code.claude.com/docs/en/skills#run-skills-in-a-subagent)).
- Доступ к skills регулируют правила `Skill`, `Skill(name)` и `Skill(name *)`. `skillOverrides` (`on`, `name-only`, `user-invocable-only`, `off`) не действует на плагинные skills ([skills](https://code.claude.com/docs/en/skills#restrict-claudes-skill-access)).
- Хуки во frontmatter skill регистрируются при вызове и работают до конца сессии. Флаг `once` учитывается только здесь ([hooks](https://code.claude.com/docs/en/hooks#common-fields)).
- Документация: если Claude пропускает правило, которое должно выполняться всегда, — «move the rule into a hook» ([skills](https://code.claude.com/docs/en/skills#claude-stops-following-a-skill)).
- В SDK skills загружаются только из файловой системы, программного API для них нет ([agent-sdk/skills](https://code.claude.com/docs/en/agent-sdk/skills)).
- Зрелость: skills, хуки, команды и субагенты доступны у всех провайдеров ([feature-availability](https://code.claude.com/docs/en/feature-availability)); пометок preview на странице skills нет.

#### Выводы и предложения
- Предложение: по skill на каждую стадию — `scout-ideas`, `stop-filter`, `collect-market-data`, `unit-economics`, `tournament-brief`, `smoke-test-plan`. У всех `disable-model-invocation: true`, чтобы стадию запускал директор или совет, а не случайное совпадение описания. `context: fork` с нужной ролью в `agent`. Формулы — в `scripts/`, а skill разрешает ровно эту команду через `allowed-tools: Bash(${CLAUDE_SKILL_DIR}/scripts/calc.py *)`.
- Предложение: всё, что обязано выполняться — проверки бюджета, запрет пишущих инструментов, обязательные поля карточки, — вынести в хуки, JSON Schema и код. Документация прямо говорит, что skill «Claude follows», а не гарантия.
- Предложение: держать `SKILL.md` коротким, важное — в начале файла: после компакции остаются только первые 5 000 токенов.

#### Пробелы
- Официальных метрик надёжности следования skills нет. Насколько стабильно Claude выполняет многошаговый SOP, документация не измеряет — это делает только свой eval (раздел 2.9).
- Может ли агент внутри воркфлоу (`agent()`) вызывать skills и с каким набором инструментов: публичная страница воркфлоу этого не перечисляет. Полный API скрипта описан в бандл-скилле `/workflow-authoring`, который здесь не открывался — не проверено.
- Версия появления `/skill-doctor` расходится: на странице skills — «requires Claude Code v2.1.252 or later» ([skills](https://code.claude.com/docs/en/skills#find-unused-skills)), в changelog — «Added `/skill-doctor`» в v2.1.261 от 2026-09-04 ([changelog](https://code.claude.com/docs/en/changelog#2-1-261)).

### 2.2. Субагенты: роли директора, скаута, аналитика, маркетолога и судьи

#### Кратко
Роли естественно ложатся на субагентов, а директор — на агента главной сессии. Субагенты могут порождать субагентов: по умолчанию до трёх уровней, не больше 20 одновременно. «Другая модель» для судьи — это всегда другая модель Claude.

#### Факты
- У каждого субагента своё окно контекста, свой системный промпт, свои инструменты и разрешения. Его запросы расходуют те же лимиты, что и основной диалог ([sub-agents](https://code.claude.com/docs/en/sub-agents)). Если описания субагентов суммарно больше 15 000 токенов, при старте выводится предупреждение ([sub-agents](https://code.claude.com/docs/en/sub-agents)).
- Приоритет определений: managed settings → флаг `--agents` → `.claude/agents/` → `~/.claude/agents/` → `agents/` плагина. Плагинные субагенты из соображений безопасности игнорируют `hooks`, `mcpServers` и `permissionMode` ([sub-agents](https://code.claude.com/docs/en/sub-agents#choose-the-subagent-scope)). Изменённые файлы агентов подхватываются без перезапуска ([sub-agents](https://code.claude.com/docs/en/sub-agents#write-subagent-files)).
- Поля frontmatter: `name` и `description` обязательны; остальные — `tools`, `disallowedTools`, `model`, `permissionMode`, `maxTurns`, `skills`, `mcpServers`, `hooks`, `memory`, `background`, `omitClaudeMd`, `effort`, `isolation`, `color`, `initialPrompt`, `experimental.cacheTtl` ([sub-agents](https://code.claude.com/docs/en/sub-agents#supported-frontmatter-fields)).
- Модель субагента: `sonnet`, `opus`, `haiku`, `fable`, полный ID вида `claude-opus-5-5` или `inherit`. Порядок выбора: параметр вызова → frontmatter → `CLAUDE_CODE_SUBAGENT_MODEL` → модель основного диалога. `CLAUDE_CODE_SUBAGENT_MODEL_FORCE=1` навязывает одну модель всем субагентам, напарникам и агентам воркфлоу. Allowlist `availableModels` заменяет запрещённую модель ([sub-agents](https://code.claude.com/docs/en/sub-agents#choose-a-model)).
- Во всех подходах к параллельной работе «the workers are Claude sessions. To involve a different tool, expose it to Claude as an MCP server» ([agents](https://code.claude.com/docs/en/agents)). Anthropic «doesn't support routing Claude Code to non-Claude models through any gateway» ([llm-gateway](https://code.claude.com/docs/en/llm-gateway)).
- Инструменты. У всех субагентов убираются `AskUserQuestion`, `EnterPlanMode`, `Workflow`, `ScheduleWakeup` и ряд других. Фоновый субагент (по умолчанию) сохраняет все MCP-инструменты и сокращённый набор встроенных. В `tools`/`disallowedTools` допустимы шаблоны `mcp__<server>`. Запись `Bash(git push *)` в `disallowedTools` снимает Bash целиком ([sub-agents](https://code.claude.com/docs/en/sub-agents#available-tools)).
- `tools: Agent(worker, researcher)` — allowlist порождаемых типов. Он работает только у агента главной сессии (`claude --agent`). В определении субагента типы в скобках игнорируются ([sub-agents](https://code.claude.com/docs/en/sub-agents#restrict-which-subagents-can-be-spawned)).
- Inline `mcpServers` подключаются на время жизни субагента, и их инструменты не попадают в контекст главного диалога. Для project-агентов нужен trust папки (с v2.1.238) ([sub-agents](https://code.claude.com/docs/en/sub-agents#scope-mcp-servers-to-a-subagent)).
- `permissionMode` субагента игнорируется, если главный диалог работает в `bypassPermissions`, `acceptEdits` или `auto`: тогда субагент наследует режим родителя ([sub-agents](https://code.claude.com/docs/en/sub-agents#permission-modes)).
- Вложенность: по умолчанию субагент может порождать субагентов до трёх уровней ниже главного диалога (`CLAUDE_CODE_MAX_SUBAGENT_SPAWN_DEPTH`; `1` отключает вложенность). В `-p` и SDK родитель не ждёт вложенных фоновых субагентов ([sub-agents](https://code.claude.com/docs/en/sub-agents#let-subagents-spawn-their-own-subagents)). Умолчания менялись: v2.1.172 (2026-06-10) — до 5 уровней; v2.1.217 (2026-07-21) — вложенность выключена; v2.1.219 (2026-07-24) — глубина 3 ([changelog](https://code.claude.com/docs/en/changelog#2-1-219), [whats-new w24](https://code.claude.com/docs/en/whats-new/2026-w24)).
- Параллельность: при 20 работающих субагентах новый вызов Agent падает с ошибкой «Concurrent subagent limit reached» (`CLAUDE_CODE_MAX_CONCURRENT_SUBAGENTS`). Общего лимита на сессию нет. У агентов воркфлоу и напарников из agent teams свои лимиты ([sub-agents](https://code.claude.com/docs/en/sub-agents#concurrent-subagent-limit)).
- Фон и передний план. В интерактивной сессии fork mode включён и субагенты работают в фоне; в `-p` и SDK — выключен. Запросы разрешений фоновых субагентов показываются в главной сессии ([sub-agents](https://code.claude.com/docs/en/sub-agents#run-subagents-in-foreground-or-background)).
- Стартовый контекст: делегирующее сообщение, CLAUDE.md, git status, предзагруженные skills (до 32), список соседних агентов. История диалога, output style и auto memory главного диалога не передаются. Окно контекста определяется моделью самого субагента ([sub-agents](https://code.claude.com/docs/en/sub-agents#what-loads-at-startup), [changelog v2.1.295](https://code.claude.com/docs/en/changelog#2-1-295)).
- `memory: user | project | local` даёт субагенту постоянный каталог, например `.claude/agent-memory/<имя>/` для `project`. В промпт загружаются первые 200 строк или 25 KB `MEMORY.md` ([sub-agents](https://code.claude.com/docs/en/sub-agents#enable-persistent-memory)).
- `isolation: worktree` — временный git worktree от ветки по умолчанию. Если субагент ничего не изменил, worktree удаляется. Команды, ушедшие в основной checkout, блокируются ([sub-agents](https://code.claude.com/docs/en/sub-agents#write-subagent-files)).
- Возобновление — через `SendMessage` по ID или имени. При достижении `maxTurns` результат помечается как частичный ([sub-agents](https://code.claude.com/docs/en/sub-agents#resume-subagents)).
- В результатах SDK есть поле `agents`: «Tokens attributed to each custom subagent definition» ([agent-sdk/typescript](https://code.claude.com/docs/en/agent-sdk/typescript)).

#### Выводы и предложения
- Предложение: директор запускается как `claude --agent director` с `tools: Agent(scout, analyst, marketer, judge), Read, Grep, Glob`; у остальных ролей `Agent` в `tools` нет, чтобы не было неконтролируемой вложенности. Скаут — `model: haiku` или `sonnet`, `tools: WebSearch, WebFetch, Read, Write` (только в свой каталог). Аналитик — `mcpServers` с read-only Wordstat, RFSD и Checko, inline. Маркетолог — генерация `content.json` лендинга без инструментов денег. Судья — `model: opus`, `tools: Read`, `omitClaudeMd: true`, чтобы устав не влиял на оценку, и `effort: high`.
- Предложение: фан-аут на 50 идей через субагентов упирается в лимит 20 одновременных и в контекст директора. Воркфлоу (раздел 2.11) подходит лучше.
- Учёт токенов по ролям: в SDK результат делит токены по определениям субагентов. Перевод в рубли — своим кодом.

#### Пробелы
- Межвендорного судьи (YandexGPT, GigaChat) примитивы не дают: только MCP-инструмент, который сам вызывает внешнюю модель. Качество такого судьи — вне Claude Code.
- Как ведёт себя вложенность внутри воркфлоу и agent teams: у них свои лимиты, детали не раскрыты — не проверено.

### 2.3. Хуки: бюджетный guard, гейты одобрения, аудит

#### Кратко
Хуки — единственный детерминированный перехватчик вызовов внутри Claude Code. Есть 33 события, 5 типов обработчиков и решения allow/deny/ask/defer с переписыванием ввода. Но до v2.1.295 хуки на командах и HTTP при сбое пропускали действие (fail-open), их может перекрыть mod, а пользователи сообщают о случаях, когда хуки не срабатывают. Жёсткий денежный лимит на одних хуках держать нельзя.

#### Факты
- Типы обработчиков: `command`, `http`, `mcp_tool`, `prompt` (однократная оценка LLM), `agent` (субагент-верификатор, до 50 ходов). Agent-хуки — «experimental and may change» ([hooks](https://code.claude.com/docs/en/hooks#hook-handler-fields), [hooks](https://code.claude.com/docs/en/hooks#agent-based-hooks)). HTTP-хуки появились в v2.1.63 (2026-02-28), `mcp_tool` — в v2.1.118 (2026-04-23) ([changelog](https://code.claude.com/docs/en/changelog#2-1-63), [changelog](https://code.claude.com/docs/en/changelog#2-1-118)).
- События: SessionStart, Setup, UserPromptSubmit, UserPromptExpansion, PreToolUse, PermissionRequest, PermissionDenied, PostToolUse, PostToolUseFailure, PostToolBatch, Notification, MessageDisplay, SubagentStart, SubagentStop, TaskCreated, TaskCompleted, Stop, StopFailure, TeammateIdle, InstructionsLoaded, ConfigChange, CwdChanged, DirectoryAdded, FileChanged, WorktreeCreate, WorktreeRemove, PreCompact, PostCompact, PreModelSwitch, PostModelSwitch, Elicitation, ElicitationResult, SessionEnd ([hooks](https://code.claude.com/docs/en/hooks#hook-lifecycle)).
- Где задаются хуки: `~/.claude/settings.json`, `.claude/settings.json`, `.claude/settings.local.json`, managed policy, `hooks/hooks.json` плагина, frontmatter skill и субагента. Хуки разных уровней складываются; `disableAllHooks` вне managed settings не отключает managed-хуки. Хуки из настроек срабатывают и внутри субагентов, а во входных данных есть `agent_id` и `agent_type` ([hooks](https://code.claude.com/docs/en/hooks#hook-locations)).
- Матчеры для MCP: `mcp__<server>__<tool>`; все инструменты сервера — только с `.*`, например `mcp__memory__.*`. Без `.*` строка сравнивается точно и ничего не ловит. Инструменты MCP из плагина называются `mcp__plugin_<plugin>_<server>__<tool>` ([hooks](https://code.claude.com/docs/en/hooks#match-mcp-tools)). Вход хука для MCP-инструмента содержит объект `mcp_server` с `name` и `source`. Документация советует доверять полю `source`, а не имени (v2.1.274+) ([hooks](https://code.claude.com/docs/en/hooks#pretooluse)).
- Поле `if` фильтрует обработчик по синтаксису правил разрешений, но «Because the `if` filter is best-effort, use the permission system rather than a hook to enforce a hard allow or deny» ([hooks](https://code.claude.com/docs/en/hooks#common-fields)).
- Решения PreToolUse: `permissionDecision` — `allow`, `deny`, `ask` или `defer`; плюс `updatedInput` и `additionalContext`. Приоритет при нескольких хуках: deny > defer > ask > allow. Deny- и ask-правила проверяются независимо от ответа хука. `ask` от хука заставляет показать запрос и в auto mode ([hooks](https://code.claude.com/docs/en/hooks#pretooluse-decision-control)).
- `defer` работает только в `-p`. Процесс завершается с `stop_reason: "tool_deferred"` и полем `deferred_tool_use`; внешний процесс собирает ответ и делает `claude -p --resume <id>`. Таймаута нет. Если в ходе несколько вызовов, `defer` игнорируется ([hooks](https://code.claude.com/docs/en/hooks#defer-a-tool-call-for-later)).
- PermissionRequest: `decision.behavior` — allow или deny, плюс `updatedInput` и `updatedPermissions`. Exit 2 здесь не учитывается. В сессиях, где нельзя показать запрос (фоновые субагенты в `-p`), отсутствие решения хука означает отказ ([hooks](https://code.claude.com/docs/en/hooks#permissionrequest), [hooks](https://code.claude.com/docs/en/hooks#permissionrequest-decision-control)).
- PostToolUse может вернуть `decision: "block"` (обратная связь Claude) и `updatedToolOutput`. Последнее меняет только то, что видит Claude: инструмент к этому моменту уже выполнился ([hooks](https://code.claude.com/docs/en/hooks#posttooluse-decision-control)). Stop и SubagentStop с `decision: "block"` продолжают работу; после 8 продолжений подряд Claude Code блок игнорирует ([hooks](https://code.claude.com/docs/en/hooks#stop)).
- Когда хук пропускает действие (fail-open):
  - exit 1 — неблокирующая ошибка, «If your hook is meant to enforce a policy, use `exit 2`»;
  - хук, который не может стартовать, — тоже неблокирующая ошибка: «a mistyped path in `settings.json` leaves the gate silently disabled» ([hooks](https://code.claude.com/docs/en/hooks#exit-code-output));
  - хук `command`, `http` или `mcp_tool` на PreToolUse с истёкшим таймаутом не блокирует вызов: «don't count on a stalled hook to act as a gate» ([hooks](https://code.claude.com/docs/en/hooks#timeouts));
  - HTTP-хук при ответе не-2xx или обрыве соединения возвращает неблокирующую ошибку ([hooks](https://code.claude.com/docs/en/hooks#http-response-handling)).
- **Новое:** v2.1.295 от 2026-10-08 — «Added `onFailure: "block"` for command and HTTP hooks: a hook that can't start, times out, or exits with an unexpected code blocks the action instead of letting it through» ([changelog](https://code.claude.com/docs/en/changelog#2-1-295)). В справочнике хуков на 2026-10-09 поле не описано: поиск по скачанным страницам находит его только в changelog.
- В SDK колбэк-хук на PreToolUse, не уложившийся в таймаут, блокирует вызов: инструмент не выполняется ([agent-sdk/hooks](https://code.claude.com/docs/en/agent-sdk/hooks#hook-timeout)).
- Лимиты: таймауты по умолчанию — 600 с для `command`, `http` и `mcp_tool`, 30 с для `prompt`, 60 с для `agent`. Поля `additionalContext`, `systemMessage` и stdout ограничены 10 000 символов ([hooks](https://code.claude.com/docs/en/hooks#common-fields), [hooks](https://code.claude.com/docs/en/hooks#json-output)). Async-хуки ничего не блокируют ([hooks](https://code.claude.com/docs/en/hooks#run-hooks-in-the-background)).
- Безопасность: command-хуки выполняются с полными правами пользователя. В `-p` и SDK папка считается доверенной, поэтому хуки из `.claude/settings.json` репозитория выполняются без диалога доверия ([hooks](https://code.claude.com/docs/en/hooks#workspace-trust)).
- Mod с обработчиком `tool.check` отвечает после правил и PreToolUse-хуков. Он может одобрить вызов, заблокированный хуком (если хук не из managed settings), и вызов под ask-правилом; в auto mode такой вызов идёт без классификатора. Deny-правила устоят перед mod только при managed settings или входе в план Team/Enterprise; «Anywhere else, the mod can approve a call that a deny rule refuses» ([permissions](https://code.claude.com/docs/en/permissions#extend-permissions-with-hooks)).
- Открытые issue о хуках (по листингу GitHub; поиск «PreToolUse hook bypass» дал 759 результатов, поиск по словам нестрогий) ([поиск](https://github.com/anthropics/claude-code/issues?q=is%3Aissue+PreToolUse+hook+bypass)):
  - PreToolUse и UserPromptSubmit не срабатывают в расширении VS Code ([#92074](https://github.com/anthropics/claude-code/issues/92074));
  - PreToolUse не срабатывают во вкладке Code приложения Desktop ([#95833](https://github.com/anthropics/claude-code/issues/95833));
  - PreToolUse-хук «silently stops firing mid-session» ([#88738](https://github.com/anthropics/claude-code/issues/88738));
  - запись файлов идёт через Bash в обход хуков на Write|Edit ([#89251](https://github.com/anthropics/claude-code/issues/89251));
  - `ask` от PreToolUse молча превращается в отказ в headless/stream-json ([#95726](https://github.com/anthropics/claude-code/issues/95726));
  - уведомление «file changed on disk» в обход хука защиты секретов ([#94082](https://github.com/anthropics/claude-code/issues/94082)).

#### Выводы и предложения
- **Бюджетный guard (предложение).** Слой 1 — `permissions.deny` в managed settings на все прямые пишущие инструменты Директа и ЮKassa: их у агентов вообще нет. Слой 2 — шлюз трат как MCP-сервер, чьи пишущие инструменты помечены `requiresUserInteraction` (раздел 2.7). Слой 3 — PreToolUse command-хук в exec-форме с `onFailure: "block"` (v2.1.295+), который спрашивает у шлюза остаток ступени и отказывает при нехватке. Это защита в глубину, а не источник истины. Источник истины — сам шлюз и его реестр (раздел 3).
- **Гейты совета (предложение).** В интерактиве — `ask`. В `-p` — `defer` с ботом Telegram или веб-формой, затем `--resume`. В SDK — `canUseTool` (раздел 2.14). Исход каждого гейта писать в журнал через PermissionRequest/PostToolUse-хук.
- **Аудит (предложение).** HTTP-хук на PreToolUse, PostToolUse, PermissionRequest, PermissionDenied и SubagentStop отправляет JSON (`session_id`, `prompt_id`, `tool_use_id`, `agent_type`, `tool_input`) в свой append-only журнал вне машины агентов. Сверка — с OTel `tool_decision` по `tool_use_id` (раздел 2.16).
- Запретить mods (`strictKnownMarketplaces`, `strictPluginOnlyCustomization`) и держать хуки-ограничители в managed settings: тогда mod не перекроет блок.

#### Пробелы
- Как `onFailure: "block"` ведёт себя на практике и для каких событий он применим, не описано: поле появилось за день до даты заметок — не проверено.
- Подтверждены ли и исправлены ли issue из списка выше — не проверено (issue не открывались).

### 2.4. Разрешения, режимы, auto mode

#### Кратко
Правила allow/ask/deny применяет сам Claude Code, а не модель, и deny держится в любом режиме. Для конвейера компании безопаснее детерминированный `dontAsk` со списком разрешений, чем вероятностный auto mode. Денежные инструменты должны всегда требовать человека.

#### Факты
- Правила проверяются в порядке deny → ask → allow; срабатывает первое совпадение, специфичность правила порядок не меняет. Deny по голому имени (`Bash`) убирает инструмент из контекста. «Permission rules are enforced by Claude Code, not by the model» ([permissions](https://code.claude.com/docs/en/permissions#manage-permissions)).
- Правила для MCP: `mcp__puppeteer` (весь сервер), `mcp__puppeteer__*`, `mcp__puppeteer__puppeteer_navigate` ([permissions](https://code.claude.com/docs/en/permissions#mcp)). Субагентов можно запретить правилом `Agent(имя)` ([permissions](https://code.claude.com/docs/en/permissions#agent-subagents)).
- Режимы: `default` (Manual), `acceptEdits`, `plan`, `auto`, `dontAsk`, `bypassPermissions`. Deny-правила блокируют во всех режимах, включая bypass; allow-правила в bypass ничего не меняют ([permission-modes](https://code.claude.com/docs/en/permission-modes#available-modes)).
- Ни один режим, даже bypass, не одобряет автоматически: ask-правила, `AskUserQuestion`, MCP-инструменты с `requiresUserInteraction`, удаление критических путей через `rm` ([permission-modes](https://code.claude.com/docs/en/permission-modes#actions-no-mode-auto-approves)).
- `dontAsk` отклоняет всё, что потребовало бы запроса; работают только явно разрешённое и то, что не требует одобрения. Документация описывает его как режим для «locked-down CI and scripts» ([permission-modes](https://code.claude.com/docs/en/permission-modes#allow-only-pre-approved-tools-with-dontask-mode)).
- Защищённые пути: `.git`, `.claude` (с исключениями для памяти и worktrees), `.mcp.json`, `.claude.json`, rc-файлы shell и др. Запись туда автоматически не одобряется нигде, кроме bypass. В `dontAsk` она отклоняется, в auto — уходит классификатору ([permission-modes](https://code.claude.com/docs/en/permission-modes#protected-paths)).
- Стартовый режим. Интерактивный терминал — `auto` (v2.1.283+, от 2026-09-25). `claude -p` и SDK — `default` при загрузке feature flags; `auto` (v2.1.285+), когда флаги не грузятся (сторонний провайдер, выключенная телеметрия). `"auto"` в `.claude/settings.json` не действует ([permission-modes](https://code.claude.com/docs/en/permission-modes#which-mode-a-session-starts-in)).
- Auto mode — классификатор, который блокирует по умолчанию `curl | bash`, отправку чувствительных данных наружу, продакшен-деплои, массовое удаление в облаке, выдачу прав IAM и изменение общей инфраструктуры. Доверены только рабочая папка и её remotes ([permission-modes](https://code.claude.com/docs/en/permission-modes#what-the-classifier-blocks-by-default)). «Auto mode reduces permission prompts but does not guarantee safety» ([permission-modes](https://code.claude.com/docs/en/permission-modes#eliminate-prompts-with-auto-mode)).
- После 3 блокировок подряд или 20 всего auto mode возвращается к запросам; пороги не настраиваются. В `-p` без prompt-tool заблокированное действие не выполняется, а Claude продолжает работу ([permission-modes](https://code.claude.com/docs/en/permission-modes#when-auto-mode-falls-back)).
- Границы, сказанные в разговоре («не пушь, пока не проверю»), теряются при компакции; для надёжности нужны ask- или deny-правила ([auto-mode-config](https://code.claude.com/docs/en/auto-mode-config#add-a-human-checkpoint)). Промпт, который скрипт воркфлоу передаёт в `agent()`, классификатор не считает просьбой пользователя ([workflows](https://code.claude.com/docs/en/workflows#what-the-saved-script-looks-like)).
- История auto mode: research preview в дайджесте недели 13 (март 2026) ([whats-new w13](https://code.claude.com/docs/en/whats-new/2026-w13)); с 14 августа 2026 — режим по умолчанию для Pro, Max и Team, а вызовы классификатора перестали расходовать лимиты ([whats-new w32](https://code.claude.com/docs/en/whats-new/2026-w32)). На текущих страницах пометки preview нет.
- Issue (по листингу GitHub): «Auto mode classifier allowed unauthorized push to main branch» ([#95749](https://github.com/anthropics/claude-code/issues/95749)); жалобы на избыточные блокировки ([#97654](https://github.com/anthropics/claude-code/issues/97654), [#95200](https://github.com/anthropics/claude-code/issues/95200)); `ask`-правила для Bash перестают работать под `--dangerously-skip-permissions` ([#95741](https://github.com/anthropics/claude-code/issues/95741)).

#### Выводы и предложения
- Предложение: headless-стадии запускать в `dontAsk` с явным allowlist по ролям. Auto mode — только для интерактивной работы основателя.
- Предложение: денежные и юридически значимые инструменты (запуск кампании, повышение ступени, покупка домена, публикация оферты) — `permissions.ask` в managed settings плюс `requiresUserInteraction` на сервере. Тогда ни классификатор, ни bypass их не одобрят, а `dontAsk` отклонит.
- `bypassPermissions` запретить через `disableBypassPermissionsMode` в managed settings ([permissions](https://code.claude.com/docs/en/permissions#permission-modes)).

#### Пробелы
- Метрики точности классификатора auto mode (ложные срабатывания и пропуски) не опубликованы — не проверено.

### 2.5. Sandbox и секреты

#### Кратко
Sandbox изолирует на уровне ОС только shell-команды. MCP-серверы, хуки и mods работают вне него с полными правами. Поэтому пишущих секретов не должно быть ни в окружении Claude Code, ни в его MCP-серверах.

#### Факты
- Sandbox ограничивает Bash, PowerShell и Monitor. Он выключен по умолчанию и работает на macOS, Linux и WSL2; на нативной Windows его нет. Запись разрешена в рабочую папку; чтение — почти везде, включая `~/.ssh` и `~/.aws/credentials`. Сеть идёт только через прокси с `allowedDomains`, список изначально пуст. Переменные окружения наследуются вместе с секретами ([sandboxing](https://code.claude.com/docs/en/sandboxing#what-the-sandbox-restricts)).
- Вне sandbox: Read, Edit, Write, WebFetch, WebSearch (их регулируют правила разрешений), а также хуки, локальные MCP-серверы, мониторы плагинов, LSP, статус-строка, `apiKeyHelper` и процессы mods ([sandboxing](https://code.claude.com/docs/en/sandboxing#what-runs-outside-the-sandbox)).
- `sandbox.credentials` с режимом `deny` закрывает файлы и снимает переменные окружения для sandbox-команд; `mask` подставляет реальное значение только на разрешённые хосты. Маскирование требует экспериментального `network.tlsTerminate`. Встроенного списка запрещённых credentials нет ([sandboxing](https://code.claude.com/docs/en/sandboxing#protect-credentials), [sandboxing](https://code.claude.com/docs/en/sandboxing#mask-credentials)).
- `CLAUDE_CODE_SUBPROCESS_ENV_SCRUB=1` вычищает credentials из окружения всех подпроцессов — Bash, хуков, stdio MCP-серверов ([env-vars](https://code.claude.com/docs/en/env-vars)).
- Ограничения: TLS не инспектируется, возможен domain fronting; «Allowing broad domains such as `github.com` can create paths for data exfiltration». Полную изоляцию даёт запуск всего процесса в контейнере, VM или devcontainer ([sandboxing](https://code.claude.com/docs/en/sandboxing#limitations)).
- В `.mcp.json` работают `${VAR}` и `${VAR:-default}`. Имена credentials (`ANTHROPIC_API_KEY`, `NPM_TOKEN` и т. п.) в `url` и `headers` удалённых серверов читаются как пустые ([mcp](https://code.claude.com/docs/en/mcp#credential-variables-that-read-as-empty)).
- Опции плагина с `sensitive: true` хранятся в системном хранилище секретов, а не в `settings.json` ([plugins/manifest-reference](https://code.claude.com/docs/en/plugins/manifest-reference)).
- `apiKeyHelper` — shell-команда, которая выдаёт ключ для запросов к модели; кэш — 5 минут ([settings-reference](https://code.claude.com/docs/en/settings-reference)). К секретам бизнеса отношения не имеет.
- Зрелость: sandbox для Bash выпущен в v2.0.24 (2025-10-20) ([changelog](https://code.claude.com/docs/en/changelog#2-0-24)); `network.tlsTerminate` — «experimental» ([sandboxing](https://code.claude.com/docs/en/sandboxing#limitations)).

#### Выводы и предложения
- Предложение: в контейнере, где работают агенты, лежат только токены чтения: Wordstat через Cloud, Директ с ролью только на просмотр (если она есть — не проверено в предыдущем отчёте), RFSD, Checko. Write-токен Директа и секретный ключ ЮKassa живут только в шлюзе трат на другом хосте. Sandbox включён, `allowedDomains` — список API данных, `CLAUDE_CODE_SUBPROCESS_ENV_SCRUB=1`.

#### Пробелы
- Где Claude Code хранит OAuth-токены удалённых MCP-серверов и насколько они защищены от процессов того же пользователя — не проверено.

### 2.6. Managed settings и managed MCP: «замки» совета

#### Кратко
Файловые managed settings — дешёвый способ закрепить политику совета так, чтобы агенты не могли её переписать. Plan Team/Enterprise не нужен.

#### Факты
- Путь на Linux — `/etc/claude-code/managed-settings.json`, плюс `managed-settings.d/` и `managed-mcp.json`. Файл перечитывается при изменении ([managed-settings](https://code.claude.com/docs/en/managed-settings#deploy-a-managed-settings-file)). Файл managed settings доступен у любого провайдера, серверные managed settings — только Team и Enterprise ([feature-availability](https://code.claude.com/docs/en/feature-availability)).
- Ключи, которые читаются только из managed-источника: `allowManagedHooksOnly`, `allowManagedMcpServersOnly`, `allowManagedPermissionRulesOnly`, `strictPluginOnlyCustomization`, `strictKnownMarketplaces`, `blockedMarketplaces`, `disableSideloadFlags`, `sandbox.network.allowManagedDomainsOnly`, `channelsEnabled`, `allowedChannelPlugins` и др. ([managed-settings](https://code.claude.com/docs/en/managed-settings#keys-only-a-managed-source-can-set)).
- В managed-каталог можно положить CLAUDE.md, skills и субагентов. Managed-субагенты имеют наивысший приоритет ([memory](https://code.claude.com/docs/en/memory#choose-where-to-put-claudemd-files), [sub-agents](https://code.claude.com/docs/en/sub-agents#choose-the-subagent-scope)).
- `managed-mcp.json` задаёт фиксированный набор серверов; `allowedMcpServers` и `deniedMcpServers` фильтруют по имени, команде или URL. «Anthropic … doesn't security-audit or manage any MCP server» ([managed-mcp](https://code.claude.com/docs/en/managed-mcp)).
- `disableWorkflows` и `disableAutoMode` тоже задаются в managed settings ([workflows](https://code.claude.com/docs/en/workflows#turn-workflows-off), [permission-modes](https://code.claude.com/docs/en/permission-modes#eliminate-prompts-with-auto-mode)).

#### Выводы и предложения
- Предложение: «устав совета» в виде `/etc/claude-code/managed-settings.json` под root:
  - deny на пишущие денежные инструменты и на `Read` секретов;
  - ask на гейты;
  - `disableBypassPermissionsMode`;
  - `allowManagedPermissionRulesOnly`, `allowManagedHooksOnly`, `allowManagedMcpServersOnly`;
  - `strictKnownMarketplaces` — только свой маркетплейс, что заодно отсекает чужие mods;
  - `sandbox.network.allowManagedDomainsOnly`.

  Агенты работают от непривилегированного пользователя.

#### Пробелы
- Применяется ли «замок» к уже запущенной долгоживущей сессии сразу при изменении файла: документация пишет «reloaded when a file changes» — поведение для каждого ключа не проверено.

### 2.7. MCP: подключение данных и шлюза трат

#### Кратко
MCP — главный способ дать ролям инструменты: read-only данные и шлюз трат. В MCP есть встроенный примитив «каждый вызов подтверждает человек» — `requiresUserInteraction`.

#### Факты
- Транспорты: HTTP, SSE, stdio, WebSocket. Области видимости: local и user (`~/.claude.json`), project (`.mcp.json` в git), плюс managed ([mcp](https://code.claude.com/docs/en/mcp#mcp-installation-scopes)).
- Серверы проекта требуют одобрения и trust; репозиторий не может одобрить свои серверы сам ([mcp](https://code.claude.com/docs/en/mcp#project-server-approvals-and-workspace-trust)). Но сессия `-p` без `--bare` подключает серверы из `.mcp.json` даже в недоверенной папке и не показывает запроса для каждого сервера ([headless](https://code.claude.com/docs/en/headless#start-faster-with-bare-mode)).
- `_meta["anthropic/requiresUserInteraction"]: true` в `tools/list` заставляет показывать запрос на каждый вызов — даже в `acceptEdits`, `auto` и `bypassPermissions`, без варианта «больше не спрашивать». Allow-правила его не снимают. `dontAsk` такой вызов отклоняет. Через `--permission-prompt-tool` ответ allow превращается в deny. Колбэк `canUseTool` в SDK такие вызовы получает и может одобрить. На Remote Control одно касание для одобрения не предлагается ([mcp](https://code.claude.com/docs/en/mcp#require-approval-for-a-specific-tool)).
- Хук тоже не может снять запрос для такого инструмента: «a hook can't skip its approval prompt with `"allow"`, with or without `updatedInput`» ([hooks](https://code.claude.com/docs/en/hooks#pretooluse-decision-control)).
- OAuth 2.0 для удалённых серверов с обновлением токена; `headersHelper` для динамических заголовков ([mcp](https://code.claude.com/docs/en/mcp#authenticate-with-remote-mcp-servers)). Elicitation (form и URL) с хуком `Elicitation` для автоответа ([mcp](https://code.claude.com/docs/en/mcp#respond-to-mcp-elicitation-requests)).
- Tool search включён по умолчанию: определения инструментов подгружаются по требованию. Он выключается, если `ANTHROPIC_BASE_URL` указывает на сторонний хост ([mcp](https://code.claude.com/docs/en/mcp#scale-with-mcp-tool-search)).
- v2.1.295 — описания MCP-инструментов, загружаемых через tool search, обрезаются на 16 384 символах вместо 2 048 ([changelog](https://code.claude.com/docs/en/changelog#2-1-295)).

#### Выводы и предложения
- Предложение: шлюз трат (Rails) выставляет MCP-сервер:
  - инструменты чтения — `get_ladder_state`, `get_campaign_stats` — без пометок;
  - инструменты действий — `propose_campaign`, `request_step_up`, `create_payment_link` — с `requiresUserInteraction: true`.

  Шлюз сам проверяет белый список логинов и лимит ступени, делает readback и пишет аудит. Агент держит только токен MCP-шлюза с правом «предложить», а не «потратить».

#### Пробелы
- Сколько стоит поддержка `requiresUserInteraction` и elicitation в Ruby MCP SDK (`mcp`) — проверить по [ruby-sdk](https://github.com/modelcontextprotocol/ruby-sdk) — не проверено.

### 2.8. Плагины, маркетплейсы, mods: упаковка «компании»

#### Кратко
«Компанию» удобно упаковать как версионируемый плагин в приватном git-маркетплейсе. Но политику безопасности плагин нести не может, а плагин — это произвольный код с правами пользователя.

#### Факты
- Компоненты плагина: skills, commands, agents, hooks (`hooks/hooks.json`), hooks module (mod), MCP-серверы (`.mcp.json`), LSP, исполняемые файлы `bin/`, настройки по умолчанию, темы и output styles, channels, monitors, workflows (поле манифеста `workflows` или каталог `workflows/`) ([plugins/components](https://code.claude.com/docs/en/plugins/components), [workflows](https://code.claude.com/docs/en/workflows#distribute-a-workflow-in-a-plugin)).
- Из `settings.json` плагина действуют только `agent` и `subagentStatusLine`, остальные ключи отбрасываются ([plugins/components](https://code.claude.com/docs/en/plugins/components#default-settings)). Плагинные субагенты не поддерживают `hooks`, `mcpServers` и `permissionMode` ([sub-agents](https://code.claude.com/docs/en/sub-agents#choose-the-subagent-scope)).
- Включённый плагин присутствует в каждой сессии. Описания его skills и агентов расходуют контекст на каждом ходу ([plugins/overview](https://code.claude.com/docs/en/plugins/overview#what-an-enabled-plugin-adds-to-your-sessions)).
- «A Claude Code plugin you install can execute arbitrary code on your machine with your user privileges». Хуки, мониторы, MCP- и LSP-серверы и процессы mods работают вне sandbox. Автообновление может поменять файлы, которые вы проверили ([plugins/security](https://code.claude.com/docs/en/plugins/security#understand-what-a-plugin-can-do)).
- Официальные имена маркетплейсов принимаются только из `github.com/anthropics/`. Для archive-источника есть пин `sha256` ([plugins/security](https://code.claude.com/docs/en/plugins/security#untrusted-marketplace-sources-and-failed-integrity-checks)).
- Приватный маркетплейс работает через git-учётные данные пользователя (SSH или HTTPS); поля для токена в `marketplace.json` нет ([plugins/host-marketplace](https://code.claude.com/docs/en/plugins/host-marketplace#grant-access-to-a-private-marketplace)).
- Версии. Новая версия доходит до пользователей, только если меняется `version`; без `version` они следуют за коммитами. Удержать версию можно через `ref`/`sha` и каналы `#stable` ([plugins/host-marketplace](https://code.claude.com/docs/en/plugins/host-marketplace#release-a-new-version), [plugins/host-marketplace](https://code.claude.com/docs/en/plugins/host-marketplace#hold-users-on-one-version)).
- Плагины выпущены в v2.0.12 от 2025-10-09 ([changelog](https://code.claude.com/docs/en/changelog#2-0-12)).
- Mods — «Claude Mods: plugins may now modify deeper behavior», v2.1.287 от 2026-10-01 ([changelog](https://code.claude.com/docs/en/changelog#2-1-287)). Mod выполняется внутри Claude Code с правами пользователя:
  - читает переменные окружения и секреты;
  - видит каждый prompt и каждый вызов инструмента;
  - переписывает их;
  - одобряет вызовы без вопроса;
  - тратит лимиты.

  Хуки mod работают и в `-p`, и в SDK ([plugins/mods/overview](https://code.claude.com/docs/en/plugins/mods/overview#what-a-mod-can-reach), [plugins/mods/overview](https://code.claude.com/docs/en/plugins/mods/overview#where-mods-run)). Пометки зрелости для mods в документации не найдено.

#### Выводы и предложения
- Предложение: плагин `company` содержит skills стадий, агентов ролей, хуки аудита, MCP-конфигурацию read-only серверов и воркфлоу. Политика (deny, ask, sandbox, замки) приходит отдельно, из managed settings. Версию закрепить через `ref`/`sha` или явный `version`; автообновление маркетплейса выключить.
- Предложение: mods запретить политикой, пока нет оснований им доверять.

#### Пробелы
- Зрелость mods (preview или GA) и может ли mod обойти managed-хуки другим путём — не проверено.

### 2.9. Evals и инструменты качества skills: как тестировать SOP компании

#### Кратко
`claude plugin eval` (GA\* с сентября 2026) даёт регрессионные тесты SOP: грейдеры, моки MCP, сравнение с прогоном без плагина и порог для CI. Это единственный встроенный способ измерить, что SOP работает, — и он стоит реальных токенов.

#### Факты
- Требуется v2.1.269+ и git ≥ 2.31, если git установлен ([plugin-evals](https://code.claude.com/docs/en/plugin-evals#requirements)). Появился в v2.1.269 от 2026-09-11 ([whats-new w37](https://code.claude.com/docs/en/whats-new/2026-w37)). Сообщение «currently in early access» означает, что сборка старше GA команды; Anthropic может выключить команду на сервере ([plugin-evals](https://code.claude.com/docs/en/plugin-evals#troubleshooting)).
- Каждый кейс по умолчанию прогоняется 3 раза. Оценка прогона — доля пройденных грейдеров, кейс проходит при `--threshold`, по умолчанию 1.0. Есть плечо без плагина (`WITH`, `W/OUT`, `Δ`) ([plugin-evals](https://code.claude.com/docs/en/plugin-evals#how-a-case-is-scored), [plugin-evals](https://code.claude.com/docs/en/plugin-evals#the-no-plugin-baseline)).
- Грейдеры ([plugin-evals](https://code.claude.com/docs/en/plugin-evals)):
  - `regex`;
  - `tool_used` с `min` и `max`; `min: 0, max: 0` — «никогда не вызывался»;
  - `tool_order`;
  - `file_exists`;
  - `llm` — судья, нужно 2 голоса из 3;
  - `baseline` — сравнение с эталонным транскриптом.
- Моки MCP-серверов: `evals/mocks/<server>/<tool>.md`. Мок бывает `fixed` или `agent` (судья играет роль сервера). Поле `expect` проверяет входные аргументы и при нарушении обнуляет прогон. `_tools.json` подставляет реальные схемы ([plugin-evals](https://code.claude.com/docs/en/plugin-evals#set-up-fixtures-and-mocks)).
- Каждый прогон и каждый LLM-грейдер — реальный вызов модели за счёт вашего плана или API ([plugin-evals](https://code.claude.com/docs/en/plugin-evals)).
- Плагин skill-creator. Тест-кейсы в `evals/evals.json`, изолированный субагент на кейс, `grading.json`, `benchmark.json` с плагином и без. Есть слепое A/B двух версий и подбор описания по долям попаданий. Форматы skill-creator и `plugin eval` несовместимы ([skills](https://code.claude.com/docs/en/skills#run-evals-with-skill-creator)).
- `/skill-doctor` показывает контекстную стоимость и частоту использования skills и отмечает никогда не вызывавшиеся ([skills](https://code.claude.com/docs/en/skills#find-unused-skills)). `claude plugin validate` находит битый frontmatter ([skills](https://code.claude.com/docs/en/skills#skill-not-triggering)).

#### Выводы и предложения
- Предложение — набор eval для SOP компании:
  - «стоп-фильтр вызывает `mcp__rfsd__concentration` до вердикта» (`tool_order`);
  - «скаут не вызывает инструменты шлюза» (`tool_used`, max 0);
  - «карточка идеи создана и валидна» (`file_exists` + `regex`);
  - «вердикт о монополисте обоснован цифрами CR3/HHI» (`llm`).

  Моки Wordstat, прогноза Директа и Checko с фиксированными ответами; `expect` на суммы и логины в `propose_campaign`. Запускать при каждом изменении SOP и при смене модели.

#### Пробелы
- Типичная стоимость прогона набора из 10–20 кейсов в токенах не опубликована — не проверено.

### 2.10. Память: CLAUDE.md, rules, auto memory как устав и база знаний

#### Кратко
CLAUDE.md и rules хорошо подходят для устава и регламентов, но остаются контекстом, а не принуждением. Auto memory — заметки модели, а не реестр компании.

#### Факты
- Иерархия:
  - managed policy — `/etc/claude-code/CLAUDE.md` на Linux;
  - пользователь — `~/.claude/CLAUDE.md`;
  - проект — `./CLAUDE.md` или `./.claude/CLAUDE.md`, иногда AGENTS.md;
  - локальный — `./CLAUDE.local.md`.

  Файлы конкатенируются, а не перекрывают друг друга. Файлы из подкаталогов загружаются по требованию ([memory](https://code.claude.com/docs/en/memory#choose-where-to-put-claudemd-files), [memory](https://code.claude.com/docs/en/memory#how-claudemd-files-load)).
- Рекомендация — меньше 200 строк на файл. Импорт `@path` — до 4 уровней; импортированные файлы тоже грузятся при старте. CLAUDE.md размером до 4 MiB загружается целиком ([memory](https://code.claude.com/docs/en/memory#write-effective-instructions), [memory](https://code.claude.com/docs/en/memory#import-additional-files), [memory](https://code.claude.com/docs/en/memory#how-it-works)).
- `.claude/rules/*.md`: без `paths` — при старте; с `paths` — только при работе с подходящими файлами ([memory](https://code.claude.com/docs/en/memory#path-specific-rules)).
- «Claude treats them as context, not enforced configuration. To block an action regardless of what Claude decides, use a PreToolUse hook» ([memory](https://code.claude.com/docs/en/memory)). Корневой CLAUDE.md переживает компакцию ([memory](https://code.claude.com/docs/en/memory#instructions-seem-lost-after-compact)).
- Auto memory. Включена по умолчанию, хранится в `~/.claude/projects/<project>/memory/`, привязана к машине. При старте загружаются первые 200 строк или 25 KB `MEMORY.md`. Типы заметок — `user`, `feedback`, `project`, `reference`. Retention sweep её не удаляет ([memory](https://code.claude.com/docs/en/memory#auto-memory), [memory](https://code.claude.com/docs/en/memory#storage-location)). В субагентов не загружается ([memory](https://code.claude.com/docs/en/memory#how-it-works)). Появилась в v2.1.59 от 2026-02-26 ([changelog](https://code.claude.com/docs/en/changelog#2-1-59)).
- Issue о соблюдении правил (по листингу GitHub): «A complete CLAUDE.md rule contract governed nothing…» ([#90542](https://github.com/anthropics/claude-code/issues/90542)); «Rules that constrain or halt work stop binding, while rules that expand work continue to bind» ([#89244](https://github.com/anthropics/claude-code/issues/89244)).

#### Выводы и предложения
- Предложение: устав и регламент совета — в CLAUDE.md и rules, ссылками на skills. Всё, что обязано выполняться, — в managed settings и хуках. Реестр идей, оценок, решений совета и денег — в БД или версионируемых JSON через MCP, а не в memory. У ролей `memory: project`, чтобы совет видел их «выученное» в git-диффах.

#### Пробелы
- Количественных данных о том, как соблюдение CLAUDE.md падает с длиной файла, нет — только качественная рекомендация.

### 2.11. Dynamic workflows: «50 идей параллельно → фильтр → турнир»

#### Кратко
Воркфлоу — лучший из встроенных примитивов для недельного фан-аута: план живёт в коде, промежуточные результаты — в переменных, вывод проверяется схемой, повтор детерминирован. Но в прогоне нельзя спросить совет, возобновить можно только в той же сессии, а при запуске в мае 2026 функция была research preview.

#### Факты
- Определение: JS-скрипт, который оркестрирует много субагентов в фоне; «Who decides what runs next: The script»; масштаб — «Dozens to hundreds of agents per run» ([workflows](https://code.claude.com/docs/en/workflows#when-to-use-a-workflow)).
- Доступность: все платные планы, Anthropic API, Bedrock, Vertex, Foundry. На Pro включается в `/config` ([workflows](https://code.claude.com/docs/en/workflows)). Запуск — v2.1.154 от 2026-05-28 ([changelog](https://code.claude.com/docs/en/changelog#2-1-154)); в дайджесте недели 22 — «research preview» ([whats-new w22](https://code.claude.com/docs/en/whats-new/2026-w22)). На текущей странице пометки нет.
- API скрипта: `agent()` (с `schema` — структурный JSON), `pipeline()`, `parallel()`, `phase()`, `log()`, глобал `args`. `Date.now()`, `Math.random()` и `new Date()` без аргументов бросают исключение — ради детерминированного повтора. Невалидный вывод повторяется до 5 раз (`MAX_STRUCTURED_OUTPUT_RETRIES`) ([workflows](https://code.claude.com/docs/en/workflows#what-the-saved-script-looks-like), [workflows](https://code.claude.com/docs/en/workflows#edit-a-saved-script)).
- Ограничения ([workflows](https://code.claude.com/docs/en/workflows#behavior-and-limits)):
  - нет ввода пользователя посреди прогона: «For sign-off between stages, run each stage as its own workflow»;
  - у самого скрипта нет доступа к файлам и shell;
  - `import()` нельзя;
  - по умолчанию до 16 параллельных агентов (`CLAUDE_CODE_WORKFLOW_MAX_CONCURRENT_AGENTS`, 1–256);
  - до 4 096 элементов в одном `parallel()` или `pipeline()`;
  - 1 000 агентов на прогон.
- Сохранение: `/workflows` → `s` → `.claude/workflows/` (в git) или `~/.claude/workflows/`; дальше запуск как `/имя`. Распространение через плагин — `/плагин:имя`. Вход — через `args` ([workflows](https://code.claude.com/docs/en/workflows#save-the-workflow-for-reuse), [workflows](https://code.claude.com/docs/en/workflows#pass-input-to-a-saved-workflow)).
- Одобрение. В интерактиве показывается запрос с фазами. В `-p` и SDK запроса нет: нужно allow-правило `Workflow` или `Workflow(<name>)`, auto mode, hook, prompt-tool или `canUseTool`. Субагенты воркфлоу живут по вашим правилам разрешений ([workflows](https://code.claude.com/docs/en/workflows#approve-the-plan-before-it-runs)).
- Возобновление — в той же сессии. Завершённые агенты отдают сохранённый результат, но падение в середине фан-аута перезапускает всех агентов, стартовавших позже. В новой сессии предыдущего прогона нет ([workflows](https://code.claude.com/docs/en/workflows#resume-after-a-pause)).
- При исчерпании лимита прогон ставится на паузу, только если сессия интерактивная и с подпиской claude.ai. В `-p`, SDK и фоновых сессиях агенты падают ([workflows](https://code.claude.com/docs/en/workflows#when-a-run-hits-your-usage-limit)). Зависший агент перезапускается до 5 раз ([workflows](https://code.claude.com/docs/en/workflows#when-an-agent-stalls-and-restarts)).
- Стоимость. Прогоны расходуют лимиты плана. Предупреждение «Large workflow» показывается при более чем 25 агентах или прогнозе больше 1,5 млн токенов; оно только информирует. Ориентир размера: `small` <5, `medium` <10 (по умолчанию; на Pro с v2.1.271 — `small`), `large` <50 агентов; это совет модели, а не лимит. Модель агента выбирается так же, как у субагентов; модель, названная в скрипте, считается моделью вызова ([workflows](https://code.claude.com/docs/en/workflows#cost), [workflows](https://code.claude.com/docs/en/workflows#set-a-size-guideline)).
- Ultracode — Claude сам решает, когда нужен воркфлоу. В этом режиме отключаются предупреждение о размере, лимит одновременных субагентов и одобрение в auto ([workflows](https://code.claude.com/docs/en/workflows#let-claude-decide-with-ultracode)).
- Выключить: `disableWorkflows` или `CLAUDE_CODE_DISABLE_WORKFLOWS=1`; для организации — в managed settings ([workflows](https://code.claude.com/docs/en/workflows#turn-workflows-off)).

#### Выводы и предложения
- Предложение: неделя разбивается на отдельные сохранённые воркфлоу с гейтами совета между ними, потому что внутри прогона спросить нельзя:
  - W1 `/weekly-scout` — генерация около 50 идей + дедупликация + стоп-фильтр, `pipeline()` со `schema`; результат — `shortlist.json`;
  - гейт совета;
  - W2 `/weekly-data` — Wordstat, прогноз и RFSD через read-only MCP + скрипт экономики;
  - W3 `/weekly-tournament` — пары судей через `parallel()` с seed из `args`, финальный рейтинг;
  - гейт GO.

  Smoke-test с деньгами — не воркфлоу, а шлюз трат.
- Оценка масштаба (допущение: 50 идей × 1–3 агента на стадию + турнир 8 финалистов по швейцарской системе): около 100–250 агентов в неделю — укладывается в лимит 1 000, но вызовет «Large workflow». Реальную стоимость нужно мерить пилотом на 5–10 идеях.
- Для надёжности в `-p` (нет паузы на лимитах) воркфлоу запускать из интерактивной или фоновой сессии с подпиской — или принять, что агенты будут падать при лимите.

#### Пробелы
- Полные опции `agent()` — тип агента, инструменты, изоляция — описаны в бандл-скилле `/workflow-authoring`, публичная страница их не перечисляет — не проверено.
- Объявлен ли воркфлоу GA после research preview, явно не сказано.

### 2.12. Agent teams и другие многоагентные поверхности

#### Кратко
Agent teams экспериментальны, выключены по умолчанию, не восстанавливаются при возобновлении и линейно дорожают. Для детерминированного недельного конвейера они не подходят; годятся разве что для «конкурирующих гипотез» по финалистам.

#### Факты
- «Agent teams are experimental and disabled by default». Включаются через `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` ([agent-teams](https://code.claude.com/docs/en/agent-teams)). Запуск — v2.1.32 от 2026-02-05 как research preview, «token-intensive» ([changelog](https://code.claude.com/docs/en/changelog#2-1-32)).
- Архитектура: лид, напарники (отдельные экземпляры Claude Code), общий список задач и почтовые ящики в JSON-файлах `~/.claude/teams/{team}/inboxes/`. Задачи — в `~/.claude/tasks/{team}/` ([agent-teams](https://code.claude.com/docs/en/agent-teams#architecture)).
- Разрешения. Напарники стартуют в режиме лида (кроме `dontAsk`), их запросы приходят в сессию лида. Сообщение от агента не считается одобрением пользователя ([agent-teams](https://code.claude.com/docs/en/agent-teams#permissions)). Гейты качества — хуки TeammateIdle, TaskCreated и TaskCompleted ([agent-teams](https://code.claude.com/docs/en/agent-teams#enforce-quality-gates-with-hooks)).
- Ограничения: in-process напарники не восстанавливаются через `/resume`; статус задач отстаёт; одна команда на сессию; вложенных команд нет; лид фиксирован ([agent-teams](https://code.claude.com/docs/en/agent-teams#limitations)). Документация рекомендует 3–5 напарников; токены растут линейно ([agent-teams](https://code.claude.com/docs/en/agent-teams#choose-an-appropriate-team-size), [costs](https://code.claude.com/docs/en/costs#agent-team-token-costs)).
- Agent view (`claude agents`, фоновые сессии) — research preview с v2.1.139 от 2026-05-11 ([agent-view](https://code.claude.com/docs/en/agent-view), [changelog](https://code.claude.com/docs/en/changelog#2-1-139)). Projects — public beta на Pro и Max, облачные треды на дни и недели; на Team и Enterprise недоступно ([claude-projects](https://code.claude.com/docs/en/claude-projects)). Cross-session messaging с API-ключом работает только на одной машине ([feature-availability](https://code.claude.com/docs/en/feature-availability)).
- Advisor tool — experimental и только Anthropic API: консультация с более сильной моделью ([advisor](https://code.claude.com/docs/en/advisor)).

#### Выводы и предложения
- Предложение: метафора «компании» соблазнительно ложится на agent teams, но для цикла с деньгами нужны восстановимость и детерминизм, которых у них нет. Брать воркфлоу и субагентов.

#### Пробелы
- Сроки выхода agent teams из experimental не объявлены.

### 2.13. Headless (`claude -p`): рантайм стадий из cron

#### Кратко
`claude -p` — самый простой рантайм стадий из внешнего планировщика. Есть структурный вывод, лимиты ходов и бюджета, режим `dontAsk` и отложенные одобрения через `defer`. `--bare` делает запуск воспроизводимым.

#### Факты
- Ключевые флаги ([cli-reference](https://code.claude.com/docs/en/cli-reference#cli-flags)):
  - `--output-format text|json|stream-json`, `--json-schema` (результат в `structured_output`);
  - `--max-turns` (выход с ошибкой при достижении);
  - `--max-budget-usd`;
  - `--allowedTools`, `--disallowedTools`, `--tools`;
  - `--permission-mode`, `--permission-prompts none` (v2.1.259+), `--permission-prompt-tool`;
  - `--mcp-config`, `--strict-mcp-config`;
  - `--agents` (JSON или путь к файлу, v2.1.281+), `--agent`;
  - `--append-system-prompt`, `--append-subagent-system-prompt`;
  - `--resume`, `--continue`, `--fork-session`, `--session-id`, `--no-session-persistence`;
  - `--settings`, `--setting-sources`;
  - `--fallback-model`, `--model`, `--effort`;
  - `--init` и `--maintenance` (Setup-хуки);
  - `--bare`.
- `--max-budget-usd` проверяется по клиентской оценке. Траты субагентов входят в лимит. На лимите новые субагенты не запускаются, а фоновые останавливаются (v2.1.217+). Траты могут превысить лимит ([cli-reference](https://code.claude.com/docs/en/cli-reference#cli-flags), [agent-sdk/agent-loop](https://code.claude.com/docs/en/agent-sdk/agent-loop#budget-headroom)).
- `--bare` пропускает автообнаружение хуков, skills, команд, субагентов, плагинов, MCP, auto memory и CLAUDE.md; всё передаётся флагами. «`--bare` is the recommended mode for scripted and SDK calls, and will become the default for `-p` in a future release». Без него `-p` выполняет хуки и подключает `.mcp.json` проекта даже в недоверенной папке ([headless](https://code.claude.com/docs/en/headless#start-faster-with-bare-mode)).
- `--permission-prompts none`: всё, что потребовало бы запроса, отклоняется, если PermissionRequest-хук его не разрешил. `AskUserQuestion` убирается. Отказы попадают в `permission_denials` ([headless](https://code.claude.com/docs/en/headless#turn-off-permission-prompts-in-unattended-runs)).
- Продолжение сессии — `--continue` или `--resume <id>`, в том числе из другой папки (v2.1.223+) ([headless](https://code.claude.com/docs/en/headless#continue-conversations)).

#### Выводы и предложения
- Предложение — шаблон запуска стадии:

  ```
  claude --bare -p "<стадия>" --agents ./company/agents.json \
    --mcp-config ./company/mcp.read.json --strict-mcp-config \
    --settings ./company/settings.json --permission-mode dontAsk \
    --json-schema "$(cat schema.json)" --max-turns 40 --max-budget-usd 3 \
    --output-format json
  ```

  Результат кладётся в БД. Для гейтов — `defer`, затем `--resume`.

### 2.14. Agent SDK: программируемый харнесс

#### Кратко
SDK — лучший вариант харнесса, если нужны собственные одобрения (`canUseTool` с выходом в Telegram или веб), колбэк-хуки, которые закрываются при сбое (fail-closed), бюджет на вызов и внешнее хранилище сессий. Но SDK — это тот же Claude Code и те же модели Claude.

#### Факты
- SDK для Python и TypeScript «runs the Claude Code binary». Возможности: инструменты, хуки, субагенты, MCP, разрешения, сессии, skills и плагины по локальному пути. Из других языков — запуск `-p --output-format json` подпроцессом ([agent-sdk/overview](https://code.claude.com/docs/en/agent-sdk/overview)). Лицензия — Commercial Terms of Service ([agent-sdk/overview](https://code.claude.com/docs/en/agent-sdk/overview#license-and-terms)).
- Порядок проверки разрешений: хуки → deny → ask → режим → allow → `canUseTool`. Ask-правила и инструменты с `requiresUserInteraction` всегда доходят до колбэка, даже при allow-правиле и в bypass. В `dontAsk` колбэк не вызывается и действие отклоняется ([agent-sdk/permissions](https://code.claude.com/docs/en/agent-sdk/permissions#how-permissions-are-evaluated)).
- Хуки-колбэки. В TS доступны все события. В Python — подмножество: PreToolUse, PostToolUse, PostToolUseFailure, UserPromptSubmit, Stop, SubagentStart, SubagentStop, PreCompact, PermissionRequest, Notification ([agent-sdk/hooks](https://code.claude.com/docs/en/agent-sdk/hooks#available-hooks)). Таймаут PreToolUse-колбэка означает, что инструмент не выполняется; таймаут на UserPromptSubmit блокирует prompt ([agent-sdk/hooks](https://code.claude.com/docs/en/agent-sdk/hooks#hook-timeout)).
- Опции: `maxBudgetUsd`, `maxTurns`, `outputFormat: {type: 'json_schema'}`, `agents`, `mcpServers`, `plugins`, `sessionStore` (зеркало транскриптов во внешнем хранилище), `resume`, `forkSession`, `persistSession`, `settingSources`, `sandbox`, `permissionPrompts`, `taskBudget` (alpha) ([agent-sdk/typescript](https://code.claude.com/docs/en/agent-sdk/typescript)).
- Стоимость в SDK — клиентская оценка: «Do not bill end users or trigger financial decisions from these fields» ([agent-sdk/cost-tracking](https://code.claude.com/docs/en/agent-sdk/cost-tracking)).
- Авторизация: «Unless previously approved, Anthropic does not allow third party developers to offer claude.ai login or rate limits for their products, including agents built on the Claude Agent SDK» ([agent-sdk/overview](https://code.claude.com/docs/en/agent-sdk/overview)). OAuth предназначен для «ordinary use of Claude Code»; разработчикам на SDK — API-ключ ([legal-and-compliance](https://code.claude.com/docs/en/legal-and-compliance)).
- Workflow tool доступен в Agent SDK начиная с v0.3.149 ([agent-sdk/typescript](https://code.claude.com/docs/en/agent-sdk/typescript)).

#### Выводы и предложения
- Предложение: если строить на Claude, то тонкий Python- или TS-сервис на SDK:
  - `canUseTool` отправляет запрос в Telegram совета и ждёт ответа;
  - PreToolUse-колбэк сверяет остаток бюджета со шлюзом (fail-closed);
  - `maxBudgetUsd` ставится на каждую стадию;
  - `sessionStore` переносит сессии в свою БД.

  Авторизация — API-ключ Console. Rails-ядро из предыдущего отчёта вызывает этот сервис по HTTP или подпроцессом.

#### Пробелы
- Квалифицирует ли Anthropic собственную внутреннюю «компанию» на подписке Pro/Max как «ordinary use» — прямо не сказано — не проверено.

### 2.15. Расписание и события: routines, `/loop`, Desktop tasks, channels

#### Кратко
Надёжного локального планировщика внутри Claude Code нет. Облачные routines и channels с relay одобрений в Telegram — в research preview.

#### Факты
- Routines: «research preview». Запуск в облаке Anthropic по расписанию, через API или по событиям GitHub. Доступны на Pro, Max, Team и Enterprise. Расходуют лимиты подписки и имеют часовые лимиты запусков ([routines](https://code.claude.com/docs/en/routines), [routines](https://code.claude.com/docs/en/routines#usage-and-limits)). Без локальных файлов (свежий клон), минимальный интервал — 1 час ([scheduled-tasks](https://code.claude.com/docs/en/scheduled-tasks#compare-scheduling-options)). Нужна подписка claude.ai ([feature-availability](https://code.claude.com/docs/en/feature-availability)).
- `/loop` и cron-инструменты привязаны к сессии; повторяющиеся задачи истекают через 7 дней. Desktop scheduled tasks требуют включённой машины ([scheduled-tasks](https://code.claude.com/docs/en/scheduled-tasks)).
- Channels — «research preview». MCP-сервер присылает события в открытую сессию. Встроены Telegram, Discord и iMessage. Есть allowlist отправителей. Свои каналы во время preview работают только через `--dangerously-load-development-channels`. Нужна авторизация claude.ai или Console; на Bedrock, Vertex и Foundry недоступно ([channels](https://code.claude.com/docs/en/channels), [channels](https://code.claude.com/docs/en/channels#research-preview)).
- Relay разрешений: двусторонний канал дублирует запрос одобрения инструмента на другое устройство, применяется первый ответ. «Anyone who can reply through the channel can approve or deny tool use in your session» ([channels-reference](https://code.claude.com/docs/en/channels-reference#relay-permission-prompts), [channels](https://code.claude.com/docs/en/channels#security)).
- `/goal` — встроенная оболочка над prompt-based Stop-хуком: продолжать до выполнения условия ([hooks](https://code.claude.com/docs/en/hooks#stop), [goal](https://code.claude.com/docs/en/goal)).

#### Выводы и предложения
- Предложение: расписание недели — systemd timer или cron на своём сервере, запускающий `claude -p` или SDK. Telegram-канал совета — либо собственный бот поверх SDK `canUseTool`, либо channels — когда выйдут из preview.

### 2.16. Output styles, статус-строка, наблюдаемость

#### Кратко
Это инструменты отображения и персонализации, а не учёта. OTel даёт хороший независимый поток событий для аудита.

#### Факты
- Output styles: Default, Proactive, Concise, Explanatory, Learning и свои. «It doesn't guarantee that something always happens» ([output-styles](https://code.claude.com/docs/en/output-styles)). На субагентов не действуют, кроме fork ([sub-agents](https://code.claude.com/docs/en/sub-agents#what-loads-at-startup)). В v2.0.30 их объявили устаревшими, в v2.0.32 вернули ([changelog](https://code.claude.com/docs/en/changelog#2-0-32)).
- Статус-строка — shell-скрипт, который получает JSON на stdin:
  - `cost.total_cost_usd` — клиентская оценка;
  - `rate_limits.five_hour` и `rate_limits.seven_day`;
  - `spend_limit` за Claude apps gateway.

  `refreshInterval` перезапускает скрипт по таймеру ([statusline](https://code.claude.com/docs/en/statusline#available-data)). Скрипт работает вне sandbox ([sandboxing](https://code.claude.com/docs/en/sandboxing#what-runs-outside-the-sandbox)).
- OTel-события:
  - `tool_decision` с `decision` и `source` (`config`, `hook`, `user_permanent`, `user_temporary`, `user_abort`, `user_reject`);
  - `tool_result`, `permission_mode_changed`, `hook_execution_complete`, `mcp_server_connection`, `plugin_installed`, `skill_activated`, `subagent_completed` (полные имена — с префиксом `claude_code.`, например `claude_code.tool_decision`).

  Имена MCP и аргументы появляются только с `OTEL_LOG_TOOL_DETAILS=1`. «Claude Code emits the raw event stream only» — алертинг остаётся на вас ([monitoring-usage](https://code.claude.com/docs/en/monitoring-usage#tool-decision-event), [monitoring-usage](https://code.claude.com/docs/en/monitoring-usage#audit-security-events)). Трейсы — beta ([agent-sdk/observability](https://code.claude.com/docs/en/agent-sdk/observability)).
- При входе по API-ключу на событиях есть только `user.id` и `session.id`; личность нужно добавить самим через `OTEL_RESOURCE_ATTRIBUTES` ([monitoring-usage](https://code.claude.com/docs/en/monitoring-usage#audit-security-events)).

#### Выводы и предложения
- Предложение: статус-строка показывает «неделя: X ₽ из Y ₽ (реестр шлюза) · LLM: $Z (оценка) · 7-дневный лимит: N%». Источник рублей — API шлюза. OTel-поток и HTTP-хук аудита сходятся в одном внешнем хранилище.

### 2.17. Сквозные условия: провайдер, юридика, темп изменений

#### Факты
- Модели — только Claude: работники — сессии Claude ([agents](https://code.claude.com/docs/en/agents)); маршрутизация на не-Claude модели не поддерживается ([llm-gateway](https://code.claude.com/docs/en/llm-gateway)).
- Routines, Desktop, Remote Control, облачные сессии и Projects требуют подписки claude.ai ([feature-availability](https://code.claude.com/docs/en/feature-availability)). Pro и Max — Consumer Terms; Team, Enterprise и API — Commercial Terms ([legal-and-compliance](https://code.claude.com/docs/en/legal-and-compliance)).
- Россия не входит в список поддерживаемых стран Anthropic — по данным предыдущего отчёта ([supported countries](https://www.anthropic.com/supported-countries); здесь не перепроверялось; разбор — в `research/ready-made-office.md`, раздел «Риски»).
- Темп изменений: с 1 января по 8 октября 2026 года в changelog 241 релиз, 20–28 в месяц (подсчёт по меткам версий на [changelog](https://code.claude.com/docs/en/changelog)). Умолчания менялись:
  - глубина вложенности субагентов: 5 → выкл. → 3;
  - субагенты в фоне по умолчанию — v2.1.198;
  - fork mode по умолчанию — v2.1.232;
  - auto mode по умолчанию — v2.1.283.

  Источники: [changelog](https://code.claude.com/docs/en/changelog#2-1-219), [whats-new w27](https://code.claude.com/docs/en/whats-new/2026-w27), [whats-new w33](https://code.claude.com/docs/en/whats-new/2026-w33), [permission-modes](https://code.claude.com/docs/en/permission-modes#which-mode-a-session-starts-in).
- Управление версией самого Claude Code: `autoUpdatesChannel: "stable"` (версия примерно недельной давности без крупных регрессий), `minimumVersion`, `DISABLE_AUTOUPDATER`, `DISABLE_UPDATES` ([settings-reference](https://code.claude.com/docs/en/settings-reference#autoupdateschannel), [env-vars](https://code.claude.com/docs/en/env-vars)).

#### Выводы и предложения
- Предложение: для «компании» — канал `stable`, фиксированная версия плагина и прогон `plugin eval` перед каждым обновлением Claude Code.

## 3. Что не покрывается примитивами и требует кода или сервиса вне Claude Code

1. **Деньги.** Нужны рублёвый append-only реестр, шлюз трат с write-токеном Директа и секретным ключом ЮKassa, лестница бюджета, readback и автопауза. Бюджеты Claude Code — это оценки токенов в USD, «not … financial decisions» ([cost-tracking](https://code.claude.com/docs/en/agent-sdk/cost-tracking)). Хуки работают на клиенте и при сбое пропускают действие, если нет `onFailure: "block"` ([hooks](https://code.claude.com/docs/en/hooks#timeouts)); mod может их перекрыть ([permissions](https://code.claude.com/docs/en/permissions#extend-permissions-with-hooks)). Предложение — как в предыдущем отчёте: шлюз на Rails, агентам — только MCP «предложить».
2. **Состояние цикла между неделями.** Идеи, оценки, статусы стадий, история решений — конечный автомат в БД. Воркфлоу возобновляются только в той же сессии ([workflows](https://code.claude.com/docs/en/workflows#resume-after-a-pause)); agent teams не восстанавливаются ([agent-teams](https://code.claude.com/docs/en/agent-teams#limitations)); memory — это заметки, а транскрипты удаляются через 30 дней ([hooks](https://code.claude.com/docs/en/hooks#defer-a-tool-call-for-later)).
3. **Реестр решений совета и UI одобрений.** Нужны личность одобрившего, сумма, обоснование, история. Claude Code даёт запрос в терминале, relay через channels или `canUseTool`, но не хранит одобрения как объекты ([channels-reference](https://code.claude.com/docs/en/channels-reference#relay-permission-prompts), [agent-sdk/permissions](https://code.claude.com/docs/en/agent-sdk/permissions)). Кандидаты — Paperclip-пилот или Rails UI (предыдущий отчёт).
4. **Надёжный планировщик.** Routines — research preview, облако без локальных файлов, подписка ([routines](https://code.claude.com/docs/en/routines)). `/loop` истекает через 7 дней; Desktop tasks требуют включённой машины ([scheduled-tasks](https://code.claude.com/docs/en/scheduled-tasks)). Нужен systemd или cron, Solid Queue или GoodJob.
5. **Коннекторы к данным РФ.** Wordstat, Директ с прогнозом Live 4, RFSD через DuckDB, Checko, Метрика, ЮKassa. MCP — только протокол; серверы нужны сторонние или свои (предыдущий отчёт, слой 4).
6. **Детерминированная математика.** Формула max CPC, концентрация CR/HHI, рейтинг турнира (Swiss или Bradley–Terry) — скрипты или сервис. В skills они живут как исполняемые файлы ([skills](https://code.claude.com/docs/en/skills#add-supporting-files)).
7. **Лендинг, хостинг, платёжный сервис, вебхуки, офлайн-конверсии.** Всё это внешняя инфраструктура.
8. **Аудит, защищённый от подмены.** Хуки и OTel пишут с той же машины; хранилище и алертинг — свои ([monitoring-usage](https://code.claude.com/docs/en/monitoring-usage#audit-security-events)).
9. **Независимость от провайдера LLM.** Claude Code работает только с Claude ([llm-gateway](https://code.claude.com/docs/en/llm-gateway)). Ролям на YandexGPT или GigaChat нужен другой рантайм; Claude Code их не запустит, разве что вызовет как MCP-инструмент.
10. **Юридический гейт.** РКН и 152-ФЗ, маркировка рекламы, оферта предзаказа — решения человека и юриста. Claude Code может только остановиться и спросить.
11. **Калибровка судей и стоп-фильтра на русских данных.** `plugin eval` меряет регрессию SOP, но наборы эталонов, разметка и межмодельная проверка — своя работа ([plugin-evals](https://code.claude.com/docs/en/plugin-evals)).

## 4. Вывод

**Короткий ответ.** «Мозги» компании — роли, SOP и аналитический конвейер — строить на примитивах Claude Code можно и удобно. Деньги, учёт, реестр решений и надёжное расписание на них строить нельзя. Это вывод исследователя (предложение) из фактов раздела 2.

**Что ложится хорошо.**
- Роли — субагенты с минимальными правами и своими моделями.
- Директор — агент главной сессии с allowlist порождаемых ролей.
- SOP — skills с ручным запуском и встроенными скриптами.
- Недельный фан-аут и турнир — сохранённые воркфлоу со схемами и детерминированным повтором.
- Устав — CLAUDE.md, rules и root-owned managed settings.
- Упаковка — версионируемый плагин в приватном маркетплейсе.
- Регрессия SOP — `claude plugin eval` с моками MCP.
- Гейты — `requiresUserInteraction`, `ask` и `defer` или `canUseTool`.
- Аудит — HTTP-хуки плюс OTel.

Для основателя, который уже живёт в Claude Code с OpenSpec, skills и субагентами, это продолжение привычной среды, а не новая платформа.

**Почему не «вся компания».**
1. Защитные примитивы работают на клиенте и местами пропускают действие при сбое. Опция `onFailure: "block"` появилась только 2026-10-08, есть пользовательские сообщения о хуках, которые не срабатывают, а mod может перекрыть блок.
2. Многоагентная оркестрация молода. Воркфлоу при запуске были research preview: в прогоне нельзя спросить совет, возобновление — только в той же сессии. Agent teams экспериментальны; routines и channels — research preview.
3. Темп изменений — 241 релиз за 2026 год, умолчания меняются.
4. Рантайм — только Claude. Для РФ это единая точка отказа: при потере доступа встаёт весь цикл, а не только инструмент разработки.

**Предложение по архитектуре — гибрид.**
- Claude Code (плагин `company`, канал `stable`, контейнер, `--bare -p` или SDK) — «офис аналитиков» без доступа к деньгам: только read-only MCP; результаты — JSON по схемам в БД.
- Rails-ядро из предыдущего отчёта — реестр, конечный автомат цикла, планировщик, шлюз трат.
- Шлюз трат как MCP-сервер с `requiresUserInteraction` на действиях.
- Совет одобряет через свой бот или UI: SDK `canUseTool` или `defer` + `--resume`.
- Managed settings — неотменяемая политика.

Если доступ к Claude из РФ признан неприемлемым риском для рантайма, Claude Code остаётся средой разработки и прототипирования SOP. Те же skills, правила и eval-кейсы переносятся в RubyLLM-агентов, а MCP-серверы данных и шлюз используются обоими рантаймами.

**Что проверить первым (предложение).**
1. Поведение `onFailure: "block"` (v2.1.295) на PreToolUse для MCP-инструментов в `-p` и в той поверхности, где работает основатель. Есть issue о несрабатывании хуков в VS Code и Desktop.
2. Сквозной сценарий `requiresUserInteraction` + `defer` + `--resume` с ботом Telegram.
3. Пилотный воркфлоу «скаут → стоп-фильтр» на 5–10 идеях с замером токенов и времени.
4. Набор `plugin eval` с моками Wordstat и Директа и порогом в CI.
5. Managed settings с `allowManagedPermissionRulesOnly` и запретом mods: проверить, что агенты не обходят deny.

## 5. Источники

Официальная документация Claude Code (страницы скачаны 2026-10-09):
- Индекс: https://code.claude.com/docs/llms.txt
- Skills: https://code.claude.com/docs/en/skills
- Субагенты: https://code.claude.com/docs/en/sub-agents
- Команды: https://code.claude.com/docs/en/commands
- Хуки (справочник): https://code.claude.com/docs/en/hooks ; гайд: https://code.claude.com/docs/en/hooks-guide
- Разрешения: https://code.claude.com/docs/en/permissions ; режимы: https://code.claude.com/docs/en/permission-modes ; auto mode: https://code.claude.com/docs/en/auto-mode-config
- Sandbox: https://code.claude.com/docs/en/sandboxing
- Managed settings: https://code.claude.com/docs/en/managed-settings ; managed MCP: https://code.claude.com/docs/en/managed-mcp ; справочник настроек: https://code.claude.com/docs/en/settings-reference ; переменные окружения: https://code.claude.com/docs/en/env-vars
- MCP: https://code.claude.com/docs/en/mcp
- Плагины: https://code.claude.com/docs/en/plugins/overview ; https://code.claude.com/docs/en/plugins/security ; https://code.claude.com/docs/en/plugins/components ; https://code.claude.com/docs/en/plugins/host-marketplace ; https://code.claude.com/docs/en/plugins/manifest-reference ; mods: https://code.claude.com/docs/en/plugins/mods/overview
- Plugin evals: https://code.claude.com/docs/en/plugin-evals
- Память: https://code.claude.com/docs/en/memory
- Dynamic workflows: https://code.claude.com/docs/en/workflows
- Agent teams: https://code.claude.com/docs/en/agent-teams ; сравнение подходов: https://code.claude.com/docs/en/agents ; agent view: https://code.claude.com/docs/en/agent-view ; Projects: https://code.claude.com/docs/en/claude-projects ; advisor: https://code.claude.com/docs/en/advisor ; goal: https://code.claude.com/docs/en/goal
- Headless: https://code.claude.com/docs/en/headless ; CLI: https://code.claude.com/docs/en/cli-reference
- Agent SDK: https://code.claude.com/docs/en/agent-sdk/overview ; https://code.claude.com/docs/en/agent-sdk/permissions ; https://code.claude.com/docs/en/agent-sdk/hooks ; https://code.claude.com/docs/en/agent-sdk/cost-tracking ; https://code.claude.com/docs/en/agent-sdk/agent-loop ; https://code.claude.com/docs/en/agent-sdk/typescript ; https://code.claude.com/docs/en/agent-sdk/skills ; https://code.claude.com/docs/en/agent-sdk/observability
- Routines: https://code.claude.com/docs/en/routines ; scheduled tasks: https://code.claude.com/docs/en/scheduled-tasks ; channels: https://code.claude.com/docs/en/channels ; https://code.claude.com/docs/en/channels-reference
- Output styles: https://code.claude.com/docs/en/output-styles ; статус-строка: https://code.claude.com/docs/en/statusline ; мониторинг (OTel): https://code.claude.com/docs/en/monitoring-usage ; costs: https://code.claude.com/docs/en/costs
- Доступность функций: https://code.claude.com/docs/en/feature-availability ; юридика: https://code.claude.com/docs/en/legal-and-compliance ; LLM-шлюзы: https://code.claude.com/docs/en/llm-gateway
- Changelog: https://code.claude.com/docs/en/changelog (генерируется из https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md)
- Еженедельные дайджесты: https://code.claude.com/docs/en/whats-new/2026-w13 ; https://code.claude.com/docs/en/whats-new/2026-w22 ; https://code.claude.com/docs/en/whats-new/2026-w24 ; https://code.claude.com/docs/en/whats-new/2026-w27 ; https://code.claude.com/docs/en/whats-new/2026-w32 ; https://code.claude.com/docs/en/whats-new/2026-w33 ; https://code.claude.com/docs/en/whats-new/2026-w37

GitHub Issues (по листингу поиска; issue не открывались; сообщения пользователей):
- Поиск: https://github.com/anthropics/claude-code/issues?q=is%3Aissue+PreToolUse+hook+bypass ; https://github.com/anthropics/claude-code/issues?q=is%3Aissue+hook+not+firing+subagent
- https://github.com/anthropics/claude-code/issues/92074 ; /95833 ; /88738 ; /89251 ; /95726 ; /94082 ; /95741 ; /95749 ; /97654 ; /95200 ; /90542 ; /89244

Внешние и из предыдущего отчёта:
- Anthropic supported countries: https://www.anthropic.com/supported-countries (по предыдущему отчёту `research/ready-made-office.md`, здесь не перепроверялось)
- Agent Skills (открытый стандарт): https://agentskills.io (упомянут в документации skills; сам сайт не открывался)
- MCP Ruby SDK: https://github.com/modelcontextprotocol/ruby-sdk (упомянут как кандидат для шлюза; в этих заметках не проверялся)
