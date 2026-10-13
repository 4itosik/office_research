# T1. Agent teams в Claude Code: механика, гейты, сбои, стоимость и смежные функции (на 2026-10-13)

Источники — страницы документации `code.claude.com/docs/en/*.md`, скачанные 2026-10-13, и changelog. Последняя версия в changelog — v2.1.296 от 2026-10-09 ([changelog](https://code.claude.com/docs/en/changelog#2-1-296)). Метки: «не проверено» — нет подтверждения первичным источником; «по сниппету поиска» — факт есть только в сводке поисковой выдачи; «предложение» — собственный синтез; «вывод» — логическое следствие из процитированных фактов, отдельно не задокументированное. Модели MVP по ТЗ — Opus 5.5, Sonnet 5.5, Haiku 5.5 (`research/mvp-claude-code.md`, раздел 2).

## 1. Механика: включение, состав команды, координация

### Кратко
Команда — это одна интерактивная сессия-лид, которая через инструмент Agent с параметром `name` порождает напарников. У каждого напарника свой контекст. Связь идёт через почтовые ящики-файлы и (только при наличии Task-инструментов) общий список задач. Роль напарника можно взять из `.claude/agents/*.md`: применяются `tools`, `model`, `disallowedTools`, `effort` и тело промпта. `skills` не применяются никогда, `mcpServers` — только в режиме split-pane. Режим разрешений при спавне общий для всех. Главная новость для MVP: на моделях новее Opus 4.7, Sonnet 4.6 и Haiku 4.5 Task-инструментов по умолчанию нет. На моделях 5.5 общего списка задач (и хуков TaskCreated/TaskCompleted) не будет, пока не задан `CLAUDE_CODE_ENABLE_TODO_TOOLS=1`. «Delegate mode» для лида из текущей документации исчез.

### Факты

#### 1.1. Включение и конфигурация
- Статус и включение: «Agent teams are experimental and disabled by default». Включение — `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` в окружении или в `env` файла settings.json. Без переменной «no team is set up at session start, no team directories are written, and Claude does not spawn or propose teammates» ([agent-teams](https://code.claude.com/docs/en/agent-teams#enable-agent-teams), [env-vars](https://code.claude.com/docs/en/env-vars)).
- Побочный эффект включения: Claude может сам дать субагенту имя, и тогда «a subagent that Claude names launches as a teammate, so teams can form even when you didn't ask for one». Подтверждения запуска нет: «Claude Code doesn't ask you to confirm the launch». Напарником не становится только форк или вызов с `isolation` в самом вызове ([agent-teams](https://code.claude.com/docs/en/agent-teams#how-claude-starts-agent-teams)). `isolation` во frontmatter запуск напарника не отменяет, и «the teammate then runs in the main session's working directory» ([sub-agents](https://code.claude.com/docs/en/sub-agents#subagent-names)).
- Выключение — переменная в значении `0`. Новая сессия не нужна: переменная перечитывается при каждом спавне. Значение `0` в user settings перекрывает export из shell, но project, local, `--settings` и managed settings могут включить команды снова ([agent-teams](https://code.claude.com/docs/en/agent-teams#claude-spawns-teammates-instead-of-subagents)).
- С v2.1.178 инструменты `TeamCreate` и `TeamDelete` удалены. «Every session now has one implicit team — spawn teammates directly with the Agent tool's `name` parameter». Параметр `team_name` принимается, но игнорируется ([changelog](https://code.claude.com/docs/en/changelog#2-1-178)).
- Нужна интерактивная сессия: «In non-interactive mode with the `-p` flag, including Agent SDK sessions, Claude doesn't spawn teammates». Именованный субагент там работает как обычный субагент ([agent-teams](https://code.claude.com/docs/en/agent-teams#enable-agent-teams)).
- Режимы отображения ([agent-teams](https://code.claude.com/docs/en/agent-teams#choose-a-display-mode), [settings-reference](https://code.claude.com/docs/en/settings-reference#teammatemode), [cli-reference](https://code.claude.com/docs/en/cli-reference#cli-flags)):
  - `teammateMode` (любой файл настроек; старое значение читается и из `~/.claude.json`): `"in-process"` — по умолчанию; `"auto"` — split panes внутри tmux или iTerm2, иначе in-process; `"tmux"`; `"iterm2"` (через CLI `it2`, добавлен в v2.1.186 — [changelog](https://code.claude.com/docs/en/changelog#2-1-186)).
  - Флаг `--teammate-mode` на одну сессию — «experimental and doesn't appear in `claude --help`».
  - Split panes не работают во встроенном терминале VS Code, в Windows Terminal и в Ghostty ([agent-teams](https://code.claude.com/docs/en/agent-teams#limitations)).
- Управление в in-process: панель агентов под полем ввода. Стрелки выбирают напарника, Enter открывает его транскрипт и даёт писать ему напрямую, Escape прерывает его текущий ход, `x` останавливает, Ctrl+T показывает список задач. Простаивающие строки прячутся через 30 секунд после того, как простаивает вся панель; больше трёх простаивающих сворачиваются в одну строку `N idle agents` ([agent-teams](https://code.claude.com/docs/en/agent-teams#start-your-first-agent-team), [agent-teams](https://code.claude.com/docs/en/agent-teams#talk-to-teammates-directly)).
- Хранение ([agent-teams](https://code.claude.com/docs/en/agent-teams#architecture)):
  - имя команды — `session-` плюс первые 8 символов ID сессии;
  - конфиг — `~/.claude/teams/{team-name}/config.json`: массив `members` с именами и ID, у лида тип `team-lead`. Напарники могут читать этот файл, чтобы найти коллег;
  - ящик каждого агента — `~/.claude/teams/{team-name}/inboxes/{agent-name}.json`;
  - задачи — `~/.claude/tasks/{team-name}/`.
- Жизненный цикл файлов: каталог конфига удаляется в конце сессии. Каталог задач остаётся локально, никуда не загружается и чистится по `cleanupPeriodDays` (по умолчанию 30 дней) ([agent-teams](https://code.claude.com/docs/en/agent-teams#architecture), [settings-reference](https://code.claude.com/docs/en/settings-reference#cleanupperioddays)).
- Конфиг нельзя писать заранее: «don't edit it by hand or pre-author it». Проектного аналога нет: `.claude/teams/teams.json` — «an ordinary file». Переиспользуемые роли задаются через определения субагентов ([agent-teams](https://code.claude.com/docs/en/agent-teams#architecture)).

#### 1.2. Состав: роли, модели, инструменты, права, размер
- **Напарник из определения субагента.** Тип берётся из project-, user-, managed- или plugin-скоупа, например: «Spawn a teammate using the security-reviewer agent type…». Что применяется ([agent-teams](https://code.claude.com/docs/en/agent-teams#use-subagent-definitions-for-teammates)):
  - `tools` — в обоих режимах. In-process-напарнику дополнительно добавляются `SendMessage`, а в сессии с Task-инструментами — ещё `TaskCreate`, `TaskGet`, `TaskList` и `TaskUpdate`;
  - `model` — в обоих режимах, если промпт спавна не называет модель;
  - `disallowedTools` — только in-process. `SendMessage` и Task-инструменты остаются даже при запрете;
  - `effort` — только in-process;
  - тело файла — in-process дописывает его к системному промпту, split-pane использует вместо системного промпта;
  - `skills` — «doesn't apply … in either display mode»: напарник загружает скиллы проекта и пользователя;
  - `mcpServers` — только split-pane. In-process «ignores the field and loads MCP servers from your project and user settings».
- Поля `permissionMode`, `hooks`, `maxTurns`, `memory` и `background` в этом списке не упомянуты. Применяются ли они к напарнику — не проверено ([agent-teams](https://code.claude.com/docs/en/agent-teams#use-subagent-definitions-for-teammates)).
- Исправленные баги применения определений:
  - v2.1.47: поле `model` из `.claude/agents/*.md` игнорировалось у напарников ([changelog](https://code.claude.com/docs/en/changelog#2-1-47));
  - v2.1.288: plugin-агент, вызванный по имени, получал умолчания вместо своих prompt, tools, disallowedTools и effort ([changelog](https://code.claude.com/docs/en/changelog#2-1-288)).
- Доверие к папке. Когда остановленного напарника возвращают сообщением, его определение из `.claude/agents/` проекта применяется заново, только если папка с файлом агента доверенная; доверие к родительской папке не считается. Иначе напарник возвращается «with the team-essential tools and no custom instructions» ([agent-teams](https://code.claude.com/docs/en/agent-teams#use-subagent-definitions-for-teammates), [errors](https://code.claude.com/docs/en/errors#teammate-agent-definition-not-restored)).
- **Модель напарника** — первое сработавшее из списка ([agent-teams](https://code.claude.com/docs/en/agent-teams#specify-teammates-and-models)):
  1. модель из промпта спавна;
  2. `model` из определения (`inherit` — модель лида);
  3. `CLAUDE_CODE_SUBAGENT_MODEL`, если он не `inherit`;
  4. текущая модель лида.

  Модель, заданная хуком `agent.spawn` мода, заменяет первый пункт. `CLAUDE_CODE_SUBAGENT_MODEL_FORCE=1` (v2.1.257+) отключает пункты 1–2. До v2.1.251 переменная стояла первой в порядке. `teammateDefaultModel` удалён в v2.1.234 ([changelog](https://code.claude.com/docs/en/changelog#2-1-234), [settings-reference](https://code.claude.com/docs/en/settings-reference#teammatedefaultmodel)). Модель проверяется по `availableModels`. На Anthropic API и Claude Platform on AWS заблокированный алиас семейства понижается до новейшей разрешённой версии семейства. В остальных случаях берётся `CLAUDE_CODE_SUBAGENT_MODEL`, если он задан, иначе модель лида.
- «A teammate's model and fast mode are fixed when it spawns». `/model` и `/fast` из окна напарника не выполняются, потому что действуют на лида ([agent-teams](https://code.claude.com/docs/en/agent-teams#talk-to-teammates-directly)). Effort по умолчанию наследуется от лида; у split-pane — с v2.1.186 ([agent-teams](https://code.claude.com/docs/en/agent-teams#specify-teammates-and-models)).
- **Права.** «Teammates start with the lead's permission mode, except `dontAsk` mode, which they don't inherit». С `--dangerously-skip-permissions` у лида так работают все напарники. «You can change an individual teammate's permission mode after spawning, but you can't set per-teammate permission modes at spawn time» ([agent-teams](https://code.claude.com/docs/en/agent-teams#permissions), [agent-teams](https://code.claude.com/docs/en/agent-teams#limitations)). Как именно менять режим у одного напарника, документ не описывает — не проверено.
- Права в Claude Code общие для сессии. Пользователь в issue #54898 (v2.1.123, апрель 2026, переведено в enhancement и закрыто): «Claude Code's permission system is session-wide, not agent-wide» — нельзя запретить запись главному агенту и разрешить субагентам ([#54898](https://github.com/anthropics/claude-code/issues/54898)). Сужать роль можно только через `tools` и `disallowedTools`; спецификатор вида `Bash(git push *)` в `disallowedTools` убирает весь Bash ([sub-agents](https://code.claude.com/docs/en/sub-agents#available-tools)).
- **Свои субагенты напарника.** Напарники сохраняют Task-инструменты и cron-инструменты ([sub-agents](https://code.claude.com/docs/en/sub-agents#available-tools)). Субагенты in-process-напарника работают только на переднем плане. Определение с `background: true` даёт ошибку; `run_in_background: true` — ошибку или тихий запуск на переднем плане ([agent-teams](https://code.claude.com/docs/en/agent-teams#limitations), [sub-agents](https://code.claude.com/docs/en/sub-agents#run-subagents-in-foreground-or-background)). В issue #32731 (март 2026) пользователь писал, что напарники «cannot spawn anything»; текущая документация прямо описывает субагентов напарника, значит поведение с тех пор изменилось ([#32731](https://github.com/anthropics/claude-code/issues/32731)).
- **Структурные ограничения** ([agent-teams](https://code.claude.com/docs/en/agent-teams#limitations)):
  - одна команда на сессию;
  - вложенных команд нет: «teammates cannot spawn their own teammates»;
  - «Lead is fixed»: передать лидерство нельзя.
- **Размер** ([agent-teams](https://code.claude.com/docs/en/agent-teams#choose-an-appropriate-team-size), [agent-teams](https://code.claude.com/docs/en/agent-teams#size-tasks-appropriately)):
  - «There's no hard limit on the number of teammates»;
  - рекомендация: «Start with 3-5 teammates for most workflows»;
  - «If you have 15 independent tasks, 3 teammates is a good starting point»;
  - «Having 5-6 tasks per teammate keeps everyone productive»;
  - «Three focused teammates often outperform five scattered ones».

  Лимит 20 одновременных субагентов на напарников не распространяется: «agent team teammates, follow their own limits instead» ([sub-agents](https://code.claude.com/docs/en/sub-agents#concurrent-subagent-limit)).
- **Лид «только координирует».** Delegate mode в текущей документации нет. Есть только совет промптом: «Wait for your teammates to complete their tasks before proceeding» ([agent-teams](https://code.claude.com/docs/en/agent-teams#wait-for-teammates-to-finish)). Значения `delegate` нет ни в `--permission-mode` ([cli-reference](https://code.claude.com/docs/en/cli-reference#cli-flags)), ни в поле `permission_mode` хуков ([hooks](https://code.claude.com/docs/en/hooks#common-input-fields)).
  - Исторически delegate mode был. Issue #25037 от 2026-02-11: «Delegate mode is toggled with Shift+Tab and restricts the lead to coordination-only tools», но напарники наследовали ограничения. Закрыто как дубликат #23447 «delegate mode cascades to teammates» ([#25037](https://github.com/anthropics/claude-code/issues/25037)).
  - По сниппету поиска: сторонние гайды описывали режим как ограничение лида «spawning, messaging, shutting down teammates, and managing tasks» ([claudefa.st](https://claudefa.st/blog/guide/agents/agent-teams-controls), [addyosmani.com](https://addyosmani.com/blog/claude-code-agent-teams/)).
  - По сниппету поиска: issue #54080 сообщает, что `delegate` появляется в выборе `--permission-mode` только в части сборок ([#54080](https://github.com/anthropics/claude-code/issues/54080)).
- **Задокументированные механизмы, которыми можно ограничить только лида:**
  - поля хуков `agent_id` — «Unique identifier for the subagent or in-process teammate the hook fires in» — и `agent_type` ([hooks](https://code.claude.com/docs/en/hooks#common-input-fields));
  - у хука мода `tool.check` поле `e.agentId` «absent when the main conversation does» (v2.1.290+) ([mods reference](https://code.claude.com/docs/en/plugins/mods/reference#tools));
  - хук мода `agent.spawn` срабатывает и на напарника (`e.isTeammate: true`): может выбрать модель или вернуть `{ deny: reason }` (v2.1.289) ([mods reference](https://code.claude.com/docs/en/plugins/mods/reference#subagents), [changelog](https://code.claude.com/docs/en/changelog#2-1-289)).
- Allowlist `Agent(worker, researcher)` в `tools` «applies only to an agent running as the main thread with `claude --agent`» ([sub-agents](https://code.claude.com/docs/en/sub-agents#restrict-which-subagents-can-be-spawned)). Ограничивает ли он типы напарников (их тоже запускает Agent) — не проверено.

#### 1.3. Координация: задачи, сообщения, ожидание, завершение
- **Список задач** ([agent-teams](https://code.claude.com/docs/en/agent-teams#assign-and-claim-tasks), [agent-teams](https://code.claude.com/docs/en/agent-teams#architecture)):
  - лид создаёт задачи, напарники их выполняют;
  - статусы: pending, in progress, completed;
  - pending-задачу с незакрытыми зависимостями взять нельзя; зависимые задачи разблокируются автоматически;
  - лид назначает задачу явно, или напарник сам берёт следующую свободную незаблокированную;
  - «Task claiming uses file locking to prevent race conditions».
- **Task-инструменты на новых моделях.** Ключевое ограничение ([tools-reference](https://code.claude.com/docs/en/tools-reference#task-tool-availability)):
  - `TaskCreate`, `TaskGet`, `TaskUpdate`, `TaskList` и `TodoWrite` по умолчанию есть «only on Opus 4.7, Sonnet 4.6, Haiku 4.5, and earlier models»;
  - на остальных моделях — только по opt-in: `CLAUDE_CODE_ENABLE_TODO_TOOLS=1` («Export … before you start Claude Code»), имя инструмента в `--allowedTools` или `--tools`;
  - в фоновых и облачных сессиях инструменты есть на любой модели;
  - in-process-напарник «follows your session». Split-pane-напарник — отдельный процесс, «so its own model decides»;
  - «Without the Task tools, an agent coordinates with its team through messages instead of the shared task list».

  Изменение шло в два шага: v2.1.233 убрал инструменты с «Opus 4.8, Sonnet 5, Fable 5, Mythos 5, and newer models»; v2.1.268 оставил их по умолчанию только на «Claude 3.x, Opus 4.0–4.7, Sonnet 4.0–4.6, Haiku 4.5» ([changelog](https://code.claude.com/docs/en/changelog#2-1-233), [changelog](https://code.claude.com/docs/en/changelog#2-1-268)). Без инструментов список задач в UI пуст ([interactive-mode](https://code.claude.com/docs/en/interactive-mode#task-list)), а TaskCreated не срабатывает: «In a session without the Task tools, this event doesn't fire» ([hooks](https://code.claude.com/docs/en/hooks#taskcreated)).
- `CLAUDE_CODE_TASK_LIST_ID` делает общий список задач для нескольких сессий вне команды — тоже только при наличии Task-инструментов ([env-vars](https://code.claude.com/docs/en/env-vars), [interactive-mode](https://code.claude.com/docs/en/interactive-mode#task-list)).
- **Сообщения** ([agent-teams](https://code.claude.com/docs/en/agent-teams#context-and-communication), [agent-teams](https://code.claude.com/docs/en/agent-teams#architecture), [errors](https://code.claude.com/docs/en/errors#failed-to-write-to-a-teammate-inbox)):
  - `SendMessage` пишет конкретному напарнику по имени;
  - широковещательной рассылки нет: «To reach everyone, send one message per recipient»;
  - имена даёт лид при спавне, поэтому для предсказуемых имён их надо задать в инструкции;
  - сообщение считается отправленным, только если запись в файл ящика прошла; иначе отправитель получает ошибку (до v2.1.224 был ложный «Message sent»);
  - битые записи ящика удаляются; до v2.1.207 одна такая запись блокировала ящик.
- Сообщение агента не равно согласию человека: «A teammate can't approve a permission prompt or supply consent on your behalf». Получивший отказ напарник не может обойти его через другого напарника. В auto mode классификатор считает пересланное одобрение недоверенным вводом и проверяет каждое сообщение до доставки; заблокированное сообщение не доходит ([agent-teams](https://code.claude.com/docs/en/agent-teams#messages-between-agents), [permission-modes](https://code.claude.com/docs/en/permission-modes#eliminate-prompts-with-auto-mode)).
- **Как лид ждёт и как к нему возвращаются результаты** ([agent-teams](https://code.claude.com/docs/en/agent-teams#context-and-communication), [changelog](https://code.claude.com/docs/en/changelog#2-1-251), [changelog](https://code.claude.com/docs/en/changelog#2-1-198)):
  - «The lead doesn't need to poll for updates»: сообщения доставляются сами;
  - закончив, напарник уведомляет лида и «includes its final answer in the notification» (исправлено в v2.1.251: раньше приходило пустое «available»);
  - если ход напарника оборвался на ошибке API, лид получает уведомление о сбое с текстом ошибки (v2.1.198).
- Типовые срывы лида:
  - лид начинает делать задачи сам, а не ждёт напарников ([agent-teams](https://code.claude.com/docs/en/agent-teams#wait-for-teammates-to-finish));
  - «The lead can stop early too, deciding the team is finished before all tasks are actually complete» ([agent-teams](https://code.claude.com/docs/en/agent-teams#agents-stopping-early)).
- **Человек и напарники.** Человек может писать любому напарнику напрямую. В окне in-process-напарника «plain text and skills go to that teammate», а встроенные команды идут лиду; `/compact`, `/clear` и `/rewind` спрашивают подтверждение ([agent-teams](https://code.claude.com/docs/en/agent-teams#talk-to-teammates-directly)).
- **Контекст напарника** ([agent-teams](https://code.claude.com/docs/en/agent-teams#context-and-communication)):
  - напарник загружает CLAUDE.md, MCP-серверы и скиллы проекта;
  - получает промпт спавна;
  - «The lead's conversation history does not carry over»;
  - `--setting-sources` лида наследуется; до v2.1.281 split-pane-напарники грузили все источники.
- **Завершение и уборка** ([agent-teams](https://code.claude.com/docs/en/agent-teams#shut-down-teammates), [tools-reference](https://code.claude.com/docs/en/tools-reference), [agent-teams](https://code.claude.com/docs/en/agent-teams#orphaned-tmux-sessions)):
  - лид отправляет shutdown request; напарник одобряет его или отклоняет с объяснением;
  - `TaskStop` принимает напарника по ID или имени;
  - «The team's shared directories are cleaned up automatically when the session ends»;
  - осиротевшие tmux-сессии убираются вручную через `tmux kill-session`.
- **Изоляции файлов нет.** «Agent teams don't isolate teammates in worktrees, so partition the work» ([agents](https://code.claude.com/docs/en/agents#choose-an-approach)); «Two teammates editing the same file leads to overwrites» ([agent-teams](https://code.claude.com/docs/en/agent-teams#avoid-file-conflicts)).
- **Скиллы с `disable-model-invocation: true`.** «Claude can't invoke the skill on its own»: описание такого скилла не попадает в контекст, и в субагента его заранее не загрузить ([skills](https://code.claude.com/docs/en/skills#control-who-invokes-a-skill)).
- **Когда команды не нужны.** «For sequential tasks, same-file edits, or work with many dependencies, a single session or subagents are more effective» ([agent-teams](https://code.claude.com/docs/en/agent-teams#when-to-use-agent-teams)).

### Выводы и предложения
- **Вывод.** Пайплайн стадий 2–7 из ТЗ последовательный, у стадий жёсткие зависимости. Это ровно тот случай, когда документация советует субагентов или одну сессию. Параллелизм в цикле есть только внутри стадий: скаут по сегментам, данные по идеям, пары турнира.
- **Вывод.** Стадийные скиллы ТЗ стоят с `disable-model-invocation: true`. Ни лид, ни напарники не смогут вызвать их сами — только человек, набрав `/scout` в окне напарника.

  Предложение для варианта на командах:
  - SOP стадии перенести в тело определения роли или в промпт спавна;
  - либо снять флаг со стадийных скиллов и защищать запуск хуками (раздел 2), а не флагом.
- **Предложение по раскладке ролей:**
  - лид — `claude --agent director` (Opus 5.5), интерактивно, режим in-process;
  - напарники из `.claude/agents/`: scout (Sonnet 5.5); analyst-sonnet и analyst-haiku — двумя определениями, потому что модель фиксируется при спавне, а аналитику по ТЗ нужны две модели; judge (Opus 5.5); marketer (Sonnet 5.5);
  - имена напарников задаются явно в промпте спавна — на них опираются хуки.
- **Предложение по MCP.** У in-process-напарника MCP из определения не применяется: все напарники видят MCP проекта (yamcp, MCP ЮKassa). Правило ТЗ «скаут без MCP» придётся держать через allowlist `tools` без `mcp__*`, а не через отсутствие серверов. Токены из окружения процесса лида видны всем in-process-напарникам так же, как субагентам (вывод).
- **Предложение.** Задать `CLAUDE_CODE_ENABLE_TODO_TOOLS=1` (export до запуска) или `--allowedTools TaskCreate`. Иначе на моделях 5.5 нет ни общего списка задач, ни хуков TaskCreated/TaskCompleted.
- **Предложение: лид без права записи.**
  - PreToolUse-хук запрещает `Write`, `Edit` и `Bash` вне `scripts/`, когда `agent_id` пуст (это главный поток);
  - либо хук мода `tool.check` с проверкой `e.agentId`.

  Вариант «`--agent director` с узким `tools`», вероятно, каскадирует на in-process-напарников: субагенты наследуют инструменты главного разговора. Так же каскадировал delegate mode (#23447) — не проверено.
- **Предложение.** Коммиты в git делает один агент — лид или человек. Общая рабочая копия без worktree-изоляции даёт гонки за индекс git (вывод).

### Пробелы
- Применяются ли к напарнику поля определения `permissionMode`, `hooks` и `maxTurns`. Документ их не перечисляет.
- Ограничивает ли `Agent(scout, analyst, …)` в определении лида типы напарников.
- Работает ли `CLAUDE_CODE_ENABLE_TODO_TOOLS` через `env` в settings.json. Документ говорит только «export before you start».
- Применимы ли определения из `--agents` (CLI JSON) к напарникам. Документ перечисляет только project, user, managed и plugin.
- Убран ли delegate mode из кода или только из документации. Нашлись лишь пользовательские issue и сниппеты.

## 2. Гейты и хуки команд

### Кратко
«Require plan approval» больше не гейт. Claude Code одобряет план напарника в сессии лида автоматически, без ревью лидом и без вопроса человеку. Человеческие гейты в команде остаются прежними:
- запросы разрешений — они всплывают в сессии лида;
- файловые гейты вроде `decision.md` + `gate_check.py`.

Хуки TeammateIdle, TaskCreated и TaskCompleted умеют блокировать (exit 2), но при сбое открывают путь (fail-open): `onFailure: "block"` на них не действует. У TaskCompleted есть ловушка: сообщение «No such file or directory» делает блокировку неблокирующей. Детерминированный гейт «артефакт проходит JSON Schema» собрать можно, но только с оговорками ниже.

### Факты
- **Plan approval.** Напарник, порождённый, пока лид в plan mode, работает в read-only plan mode до готовности плана. Затем «it sends a plan approval request to the lead. Claude Code approves the plan in the lead's session as soon as the request arrives, without the lead reviewing it». Правки и команды напарника всё равно идут через запросы разрешений ([agent-teams](https://code.claude.com/docs/en/agent-teams#have-teammates-plan-before-implementing)). «Plan approval is the designed exception: the lead session grants teammate plan approvals without a separate prompt to you» ([agent-teams](https://code.claude.com/docs/en/agent-teams#permissions)).
- Протокольные сообщения plan approval request, plan approval и plan rejection по-прежнему существуют ([errors](https://code.claude.com/docs/en/errors#failed-to-write-to-a-teammate-inbox)). Структурные сообщения `shutdown_request` и `plan_approval_response` требуют включённых команд ([sub-agents](https://code.claude.com/docs/en/sub-agents#resume-subagents)). Записи в changelog о переходе к автоодобрению нет. Когда поведение изменилось — не проверено.
- **Запросы разрешений напарников.** «Teammate permission prompts appear in the lead session, so approve them there yourself» ([agent-teams](https://code.claude.com/docs/en/agent-teams#permissions)). Против потока запросов документ советует заранее разрешить частые операции ([agent-teams](https://code.claude.com/docs/en/agent-teams#too-many-permission-prompts)).
  - Если запись в ящик лида не удалась: «The permission request could not be delivered to the team lead (mailbox write failed)» — «nobody approved the tool call» ([errors](https://code.claude.com/docs/en/errors#failed-to-write-to-a-teammate-inbox)).
  - Исправленные баги: двойной ответ на запрос (v2.1.257) и падение диалога (v2.1.114) ([changelog](https://code.claude.com/docs/en/changelog#2-1-257), [changelog](https://code.claude.com/docs/en/changelog#2-1-114)).
- **TeammateIdle** ([hooks](https://code.claude.com/docs/en/hooks#teammateidle), [hooks](https://code.claude.com/docs/en/hooks#teammateidle-decision-control)):
  - срабатывает, когда напарник «is about to go idle after finishing its turn»;
  - матчеров нет, срабатывает на каждое событие;
  - вход: `teammate_name`, `team_name` («Deprecated … will be removed»), `agent_id` (v2.1.290+);
  - exit 2: «the teammate receives the stderr message as feedback and continues working instead of going idle»;
  - JSON `{"continue": false, "stopReason": "..."}` останавливает напарника совсем;
  - с v2.1.290 не срабатывает от субагентов и форков напарника ([changelog](https://code.claude.com/docs/en/changelog#2-1-290)).
- **TaskCreated** ([hooks](https://code.claude.com/docs/en/hooks#taskcreated), [hooks](https://code.claude.com/docs/en/hooks#taskcreated-decision-control)):
  - срабатывает при `TaskCreate`; матчеров нет;
  - вход: `task_id`, `task_subject`, необязательные `task_description` и `teammate_name`, `team_name` (deprecated), `agent_id` (v2.1.290+);
  - exit 2 или `{"decision":"block","reason":...}`: «Claude Code deletes the task and returns your message to Claude as the tool's error»;
  - `continue: false` игнорируется;
  - добавлен в v2.1.84 ([changelog](https://code.claude.com/docs/en/changelog#2-1-84)).
- **TaskCompleted** ([hooks](https://code.claude.com/docs/en/hooks#taskcompleted), [hooks](https://code.claude.com/docs/en/hooks#taskcompleted-decision-control)):
  - срабатывает в двух случаях: агент отмечает задачу выполненной через `TaskUpdate`, или напарник «finishes its turn with in-progress tasks»;
  - матчеров нет; вход как у TaskCreated;
  - exit 2: «the task is not marked as completed and the stderr message is fed back to the model as feedback»;
  - `continue: false` останавливает напарника, только если событие вызвано окончанием его хода; при вызове через `TaskUpdate` игнорируется;
  - TeammateIdle и TaskCompleted добавлены в v2.1.33, поддержка `continue:false` — в v2.1.69 ([changelog](https://code.claude.com/docs/en/changelog#2-1-33), [changelog](https://code.claude.com/docs/en/changelog#2-1-69)).
- **Общая семантика кодов выхода** ([hooks](https://code.claude.com/docs/en/hooks#exit-code-output)):
  - exit 2 блокирует;
  - любой другой ненулевой код — «non-blocking error», действие продолжается: «Claude Code treats exit code 1 as a non-blocking error»;
  - особый случай: «`Stop`, `SubagentStop`, `TaskCompleted`, and a plugin's `UserPromptSubmit` hook: when your hook exits 2 with nothing on stdout and its stderr says a file is missing, such as `No such file or directory`, Claude Code treats the run as a non-blocking error».
- **Fail-open.**
  - `onFailure: "block"` (v2.1.295+) «has no effect» на Stop, SubagentStop, TaskCompleted и TeammateIdle: «Claude can't repair a hook that won't run» ([hooks](https://code.claude.com/docs/en/hooks#block-the-action-when-a-hook-fails)).
  - Хук, упавший по таймауту, отменяется, его вывод отбрасывается, решения нет ([hooks](https://code.claude.com/docs/en/hooks#timeouts)).
- **Prompt- и agent-хуки** поддерживаются на TaskCompleted, TaskCreated и TeammateIdle ([hooks](https://code.claude.com/docs/en/hooks#prompt-based-hooks)).
  - TaskCompleted, `ok:false`: если задачу закрывают во время хода, причина возвращается как ошибка инструмента. Если событие вызвано остановкой напарника — работает как TeammateIdle и по умолчанию останавливает напарника.
  - TeammateIdle, `ok:false`: напарник останавливается, если не задан `continueOnBlock: true`.
  - Agent-хуки экспериментальны: до 50 ходов, Read/Grep/Glob, таймаут по умолчанию 60 с ([hooks](https://code.claude.com/docs/en/hooks#agent-based-hooks)).
- **Лимит повторов.** Лимит в 8 подряд продолжений (`CLAUDE_CODE_STOP_HOOK_BLOCK_CAP`) описан только для Stop и SubagentStop ([env-vars](https://code.claude.com/docs/en/env-vars), [hooks](https://code.claude.com/docs/en/hooks#stop-input)). Для TeammateIdle и TaskCompleted аналогичного лимита в документации нет — не проверено.
- **Другие точки контроля:**
  - SubagentStart срабатывает «each time an in-process agent team teammate handles a new message». Он может добавить `additionalContext`, но не может блокировать ([hooks](https://code.claude.com/docs/en/hooks#subagentstart));
  - во входе Stop массив `background_tasks` содержит записи типа `teammate` ([hooks](https://code.claude.com/docs/en/hooks#stop-input)).

### Выводы и предложения
- **Вывод.** Одобрение планов нельзя использовать как гейт совета: человек его не видит, лид не ревьюит. Людские гейты команды:
  1. `ask`-правила (публикация, MCP ЮKassa) — запрос всплывает в сессии лида;
  2. `decision.md`, который коммитит человек, + `gate_check.py`;
  3. ручной запуск стадии человеком.
- **Предложение: TaskCreated как гейт совета.** Хук запрещает создавать задачи стадий 8–11 (`[stage:landing]`, `[stage:smoke-plan]`, `[stage:monitor]`), пока `gate_check.py` не вернёт 0. Блокировка удаляет задачу. Это работает, только если координация идёт через задачи. Напарник, получивший поручение сообщением, обойдёт гейт. Поэтому дублировать PreToolUse-хуком на запись в `landing/` и `smoke/`.
- **Предложение: TaskCompleted как гейт схемы.** Гейт рабочий при пяти условиях:
  1. Task-инструменты включены (раздел 1);
  2. путь артефакта записан в `task_subject` или `task_description` по соглашению, потому что хук не получает путь к артефакту;
  3. обёртка валидатора на любой сбой выходит с кодом 2 (`trap … ERR`), печатает непустой stdout и не пропускает в stderr строку «No such file or directory»;
  4. TeammateIdle-хук работает как страховка: он проверяет артефакт по имени напарника и не зависит от Task-инструментов;
  5. следующая стадия всё равно сама валидирует вход (`validate.py`), потому что статус задач «can lag» и задачу можно не закрыть вовсе.

  Счётчик попыток хуку нужен свой: после N отказов — `{"continue": false, "stopReason": …}`. Эскиз:

  ```bash
  #!/bin/bash
  # .claude/hooks/task-gate.sh — TaskCompleted (предложение, не тестировалось)
  trap 'echo "gate: blocked"; echo "внутренняя ошибка гейта" >&2; exit 2' ERR
  in=$(cat); subj=$(jq -r '.task_subject // ""' <<<"$in")
  art=$(grep -oE 'cycles/[^ ]+\.json' <<<"$subj" || true)
  [ -z "$art" ] && exit 0                        # задача без артефакта не гейтится
  cd "$CLAUDE_PROJECT_DIR"
  if [ ! -f "$art" ]; then echo "gate: blocked"; echo "артефакт $art отсутствует" >&2; exit 2; fi
  if ! out=$(python3 scripts/validate.py "$art" 2>&1); then
    echo "gate: blocked"; echo "схема не пройдена: ${out//No such file or directory/file missing}" | head -c 4000 >&2; exit 2
  fi
  exit 0
  ```

  Если в системе нет `jq` или `python3`, шелл завершится с кодом 127. Ловушка `ERR` это перехватит, но команду лучше проверить при старте сессии (предложение).
- **Вывод.** Хуки команды остаются «журналом с подталкиванием», а не замком: fail-open при падении, можно обойти через сообщения. Это согласуется с правилом ТЗ «Хуки ненадёжны как гейт» (раздел 16 ТЗ).

### Пробелы
- Формат файлов задач в `~/.claude/tasks/{team}/` не документирован. Читать его из хука — не проверено.
- Настройки, которая делает человека (а не лида) утверждающим планы напарников, в документации нет.
- Есть ли защита от бесконечного цикла exit 2 на TeammateIdle и TaskCompleted — не проверено.

## 3. Автоматизация, сбои и возобновление

### Кратко
Команда существует, только пока жив процесс интерактивной сессии лида. Команд нет:
- в `-p`;
- в Agent SDK;
- значит, и в GitHub Actions — вывод: Action построен на SDK.

`/resume`, `/rewind` и, по всей видимости, переход в фон через agent view in-process-напарников не восстанавливают. Переживают только список задач и файлы в репозитории. Неделю одной командой не прожить. Реалистичная схема — короткие команды на одну стадию или один день, состояние в git.

### Факты
- **Headless.** В `-p`, включая сессии Agent SDK, «Claude doesn't spawn teammates» ([agent-teams](https://code.claude.com/docs/en/agent-teams#enable-agent-teams)). Claude Code GitHub Action «is built on the SDK» ([github-actions](https://code.claude.com/docs/en/github-actions)). Отсюда вывод: в Actions команд нет.
- В v2.1.71 исправлено «`--print` hanging forever when team agents are configured» ([changelog](https://code.claude.com/docs/en/changelog#2-1-71)). Переменная `CLAUDE_CODE_TEAM_TEARDOWN_PARK_TIMEOUT_MS` (v2.1.206+) задаёт, сколько «a non-interactive session waits at exit for its agent team to finish tearing down» ([env-vars](https://code.claude.com/docs/en/env-vars)). Как это сочетается с запретом спавна в `-p` — не проверено. Вероятно, речь о неявной команде сессии без напарников.
- **Облако.** В облачных сессиях команды выключены по умолчанию, но «can be enabled by adding `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` to your environment variables» ([claude-code-on-the-web](https://code.claude.com/docs/en/claude-code-on-the-web#manage-context)). Страница Projects: «an agent team is one session that starts teammate sessions for a single task, on your machine or inside a cloud session, and ends with that task» ([claude-projects](https://code.claude.com/docs/en/claude-projects#how-projects-relate-to-other-claude-code-features)).
- Routines «run autonomously as full Claude Code cloud sessions: there is no permission-mode picker» ([routines](https://code.claude.com/docs/en/routines#create-a-routine)). Спавнит ли рутина напарников — не проверено.
- **Бюджет.** `--max-budget-usd` — «(print mode only)» ([cli-reference](https://code.claude.com/docs/en/cli-reference#cli-flags)). Вывод: к интерактивной команде он неприменим.
- **Возобновление.** «`/resume` and `/rewind` do not restore in-process teammates. After resuming a session, the lead may attempt to message teammates that no longer exist» — лиду нужно породить новых ([agent-teams](https://code.claude.com/docs/en/agent-teams#limitations)). Каталог задач сохраняется, «so resumed sessions keep their tasks», а конфиг команды удаляется ([agent-teams](https://code.claude.com/docs/en/agent-teams#architecture)).
- **Внутри живой сессии.** Остановленного in-process-напарника сообщение возвращает с сохранённым разговором ([agent-teams](https://code.claude.com/docs/en/agent-teams#use-subagent-definitions-for-teammates)). Фоновая работа напарника «can't outlive the lead's process» ([agent-teams](https://code.claude.com/docs/en/agent-teams#limitations)).
- **Переход в фон (agent view).** «Backgrounding starts a fresh process that resumes from the saved conversation». В списке того, что переносится — фоновые shell-команды, фоновые субагенты, воркфлоу, `/loop`, — напарников нет ([agent-view](https://code.claude.com/docs/en/agent-view#what-carries-over-when-you-background)). Вывод: напарники теряются (не проверено).
- **Split panes** ([changelog](https://code.claude.com/docs/en/changelog#2-1-77), [changelog](https://code.claude.com/docs/en/changelog#2-1-257), [agent-teams](https://code.claude.com/docs/en/agent-teams#orphaned-tmux-sessions)):
  - панели закрываются при выходе лида (исправлено в v2.1.77);
  - панели иногда оставались открытыми после shutdown (исправлено в v2.1.257);
  - возможны осиротевшие tmux-сессии.
- **Сбои** ([agent-teams](https://code.claude.com/docs/en/agent-teams#agents-stopping-early), [agent-teams](https://code.claude.com/docs/en/agent-teams#limitations), [agent-teams](https://code.claude.com/docs/en/agent-teams#monitor-and-steer)):
  - «Teammates may stop after encountering errors instead of recovering»;
  - сообщение будит напарника, ждущего повтора API-запроса;
  - «Task status can lag … which blocks dependent tasks»;
  - «Shutdown can be slow»;
  - «Letting a team run unattended for too long increases the risk of wasted effort».
- **Лимиты использования.** Ожидание сброса лимита стартует само только в обычной сессии. В сессиях «Remote Control and agent team teammate sessions» его надо запускать вручную через `/rate-limit-options` ([interactive-mode](https://code.claude.com/docs/en/interactive-mode#start-a-wait-yourself)). Воркфлоу внутри сессии напарника на лимите не ставится на паузу ([workflows](https://code.claude.com/docs/en/workflows#when-a-run-hits-your-usage-limit)).
- **Компакция** ([agent-teams](https://code.claude.com/docs/en/agent-teams#talk-to-teammates-directly), [interactive-mode](https://code.claude.com/docs/en/interactive-mode#task-list), [changelog](https://code.claude.com/docs/en/changelog#2-1-63)):
  - `/compact`, набранный в окне напарника, действует на лида (с подтверждением);
  - у напарников свои окна контекста;
  - «Tasks persist across context compactions»;
  - утечки памяти долгоживущих напарников после компакции исправлены в v2.1.63.
- **Отказ ящика** ([errors](https://code.claude.com/docs/en/errors#failed-to-write-to-a-teammate-inbox)):
  - диск полон, каталог не пишется или ящик заблокирован — сообщение не доставлено;
  - если не дошёл plan approval request, напарник остаётся в plan mode.

### Выводы и предложения
- **Вывод: неделя одной командой.** Нужен непрерывно живой процесс лида: машина включена, терминал или tmux открыт. Человек должен быть у терминала лида, чтобы отвечать на запросы разрешений. Любой рестарт теряет напарников. Это противоречит гейтам ТЗ «понедельник → вторник → ежедневный мониторинг».
- **Предложение: схема на командах.**
  - Цикл — серия отдельных командных прогонов: «стадии 2–7 за одну сессию», затем гейт GO, затем «8–9», затем ежедневный `/monitor`.
  - Состояние только в файлах `cycles/<неделя>/` и `manifest.json`.
  - Каждый прогон начинает с чтения манифеста и порождает напарников заново.
  - Список задач в `~/.claude/tasks/` не служит источником истины: он локальный и чистится через 30 дней.
- **Вывод.** Для запуска без человека (cron, CI) команды неприменимы. Это остаётся зоной `claude -p` с субагентами или воркфлоу (заметки c1, §2.11 и §2.13).

### Пробелы
- Работают ли команды в routines и тредах Projects без человека — не проверено.
- Что происходит с напарниками при переходе сессии в фон — прямо не описано.
- Применяется ли к напарникам `autoCompactWindow`: поле добавлено во frontmatter субагентов в v2.1.296 ([changelog](https://code.claude.com/docs/en/changelog#2-1-296)).

## 4. Стоимость

### Кратко
Документация даёт один численный ориентир: «approximately 7x more tokens than standard sessions when teammates run in plan mode». Остальное — качественно: расход растёт линейно с числом активных напарников. Ограничителя бюджета для интерактивной команды нет (`--max-budget-usd` работает только в `-p`).

### Факты
- «Agent teams use approximately 7x more tokens than standard sessions when teammates run in plan mode» ([costs](https://code.claude.com/docs/en/costs#manage-agent-team-costs)).
- Рекомендации документации ([costs](https://code.claude.com/docs/en/costs#agent-team-token-costs)):
  - «Use Sonnet for teammates»;
  - держать команду маленькой: расход «roughly proportional to team size»;
  - короткие промпты спавна;
  - «Shut down teammates when their work is done. Each active teammate continues consuming tokens until it exits or the session ends».
- «Token costs scale linearly» ([agent-teams](https://code.claude.com/docs/en/agent-teams#choose-an-appropriate-team-size)). Среди причин роста расхода в долгой сессии — «Agent teammates: each active teammate keeps consuming tokens until it exits» ([costs](https://code.claude.com/docs/en/costs#why-usage-climbs-in-a-long-session)).
- **Кэш.** Запросы in-process-напарника не попадают в TTL-корзину основного разговора: кэш живёт 5 минут, «including on a Claude subscription». Час — через `subagentPromptCacheTtl: "1h"`, но запись часового кэша дороже ([agent-teams](https://code.claude.com/docs/en/agent-teams#token-usage), [settings-reference](https://code.claude.com/docs/en/settings-reference#subagentpromptcachettl)). Промахи кэша у напарников чинили в v2.1.261, v2.1.265, v2.1.277 и v2.1.290 ([changelog](https://code.claude.com/docs/en/changelog#2-1-265)).
- В v2.1.212 тела `SendMessage` перестали дублироваться в историю: «Reduced token usage in inter-agent messaging» ([changelog](https://code.claude.com/docs/en/changelog#2-1-212)).
- `--max-budget-usd` — только print mode; траты субагентов в него входят ([cli-reference](https://code.claude.com/docs/en/cli-reference#cli-flags)).

### Выводы и предложения
- **Оценка.** Не проверено, это экстраполяция. Множитель 7× относится к режиму плана и сравнивает команду с одиночной сессией, а не с конвейером субагентов. Если взять его как верхнюю границу к $13–27 в неделю из ТЗ (раздел 0), получится порядка $90–190 в неделю. Реальное значение — только замером пилотного прогона.
- **Предложение по экономии:**
  - напарников-исполнителей держать на Sonnet или Haiku 5.5;
  - plan mode для команды не включать (это главный множитель);
  - гасить напарников после стадии;
  - не держать команду живой между днями.
- **Вывод о потолке.** Потолок денег при командах даёт только внешний лимит: месячный лимит workspace в Console (ТЗ, раздел 3, п. 9) или лимиты подписки. Команды интерактивны, поэтому их можно запускать на подписке. Укладывается ли такое использование в «ordinary use» Consumer Terms — не проверено.

### Пробелы
- Нет официального множителя для режимов, отличных от plan mode, и нет данных о расходе простаивающего напарника.

## 5. История версий и статус

### Кратко
За восемь месяцев (v2.1.32 → v2.1.296) функция осталась «experimental and disabled by default». Было несколько ломающих изменений:
- удаление TeamCreate и TeamDelete (v2.1.178);
- удаление Task-инструментов с новых моделей (v2.1.233 и v2.1.268);
- удаление `teammateDefaultModel` (v2.1.234);
- смена порядка выбора модели (v2.1.251);
- автоодобрение планов (версия не указана).

### Факты
- Текущий ярлык: «experimental and disabled by default» ([agent-teams](https://code.claude.com/docs/en/agent-teams)); в сравнительной таблице — «Experimental and disabled by default» ([agents](https://code.claude.com/docs/en/agents)). При запуске — «research preview» ([changelog](https://code.claude.com/docs/en/changelog#2-1-32)). Объявления GA не найдено.

| Версия, дата | Изменение | Тип |
|---|---|---|
| [2.1.32](https://code.claude.com/docs/en/changelog#2-1-32), 2026-02-05 | «Added research preview agent teams feature … (token-intensive feature, requires setting CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1)» | запуск |
| [2.1.33](https://code.claude.com/docs/en/changelog#2-1-33), 2026-02-06 | Добавлены хуки `TeammateIdle` и `TaskCompleted`; исправлен обмен сообщениями у напарников в tmux | функция |
| [2.1.41](https://code.claude.com/docs/en/changelog#2-1-41) / [2.1.45](https://code.claude.com/docs/en/changelog#2-1-45), 2026-02-13/17 | Исправлены неверные ID моделей и окружение провайдера для Bedrock, Vertex и Foundry | исправление |
| [2.1.47](https://code.claude.com/docs/en/changelog#2-1-47), 2026-02-18 | `model` из `.claude/agents/*.md` игнорировался у напарников — исправлено; навигация через Shift+Down | исправление |
| [2.1.50](https://code.claude.com/docs/en/changelog#2-1-50) / [2.1.63](https://code.claude.com/docs/en/changelog#2-1-63), февраль | Утечки памяти у долгоживущих напарников | исправление |
| [2.1.69](https://code.claude.com/docs/en/changelog#2-1-69), 2026-03-05 | `continue:false` в TeammateIdle и TaskCompleted; закрыт случайный вложенный спавн напарников | функция, исправление |
| [2.1.71](https://code.claude.com/docs/en/changelog#2-1-71), 2026-03-07 | `--print` больше не зависает при настроенных team agents | исправление |
| [2.1.72](https://code.claude.com/docs/en/changelog#2-1-72), 2026-03-10 | Напарники наследуют модель лида | поведение |
| [2.1.77](https://code.claude.com/docs/en/changelog#2-1-77), 2026-03-17 | Панели закрываются при выходе лида; Agent tool больше не принимает `resume` — вместо него `SendMessage` | исправление, ломающее |
| [2.1.84](https://code.claude.com/docs/en/changelog#2-1-84), 2026-03-26 | Хук `TaskCreated` | функция |
| [2.1.98](https://code.claude.com/docs/en/changelog#2-1-98), 2026-04-09 | Напарники не наследовали `--dangerously-skip-permissions` — исправлено | исправление |
| [2.1.147](https://code.claude.com/docs/en/changelog#2-1-147), 2026-05-21 | `CLAUDE_CODE_SUBAGENT_MODEL` применяется к процессам напарников | исправление |
| [2.1.178](https://code.claude.com/docs/en/changelog#2-1-178), 2026-06-15 | Удалены `TeamCreate` и `TeamDelete`; одна неявная команда на сессию; `team_name` игнорируется | **ломающее** |
| [2.1.183](https://code.claude.com/docs/en/changelog#2-1-183), 2026-06-19 | Фоновые задачи напарника гибли в конце его хода — исправлено | исправление |
| [2.1.186](https://code.claude.com/docs/en/changelog#2-1-186), 2026-06-22 | `teammateMode: "iterm2"`; tmux-напарники наследуют `--effort` | функция |
| [2.1.198](https://code.claude.com/docs/en/changelog#2-1-198), 2026-07-01 | Упавший на API напарник сообщает лиду «failed»; сообщение будит его на повтор | исправление |
| [2.1.207](https://code.claude.com/docs/en/changelog#2-1-207), 2026-07-11 | Битая запись ящика вызывала цикл падений — исправлено | исправление |
| [2.1.222](https://code.claude.com/docs/en/changelog#2-1-222), 2026-08-04 | Классификатор auto mode проверяет `SendMessage` до отправки | безопасность |
| [2.1.224](https://code.claude.com/docs/en/changelog#2-1-224), 2026-08-07 | Неудачная запись в ящик теперь ошибка, а не «Message sent»; появился cross-session messaging | исправление, функция |
| [2.1.232](https://code.claude.com/docs/en/changelog#2-1-232), 2026-08-13 | Fork mode включён по умолчанию; спавны, не ставшие напарниками, по умолчанию идут в фон | поведение |
| [2.1.233](https://code.claude.com/docs/en/changelog#2-1-233), 2026-08-14 | Task- и Todo-инструменты убраны с Opus 4.8, Sonnet 5, Fable 5, Mythos 5 и новее; возврат — `CLAUDE_CODE_ENABLE_TODO_TOOLS=1` | **ломающее** для общего списка задач |
| [2.1.234](https://code.claude.com/docs/en/changelog#2-1-234), 2026-08-17 | Удалена настройка «Default teammate model» (`teammateDefaultModel`) | **ломающее** |
| [2.1.239](https://code.claude.com/docs/en/changelog#2-1-239), 2026-08-21 | `ListAgents` показывает живых напарников | функция |
| [2.1.251](https://code.claude.com/docs/en/changelog#2-1-251), 2026-08-28 | Финальный ответ напарника приходит в уведомлении о простое; новый порядок выбора модели — до этой версии `CLAUDE_CODE_SUBAGENT_MODEL` шёл первым ([agent-teams](https://code.claude.com/docs/en/agent-teams#specify-teammates-and-models)) | исправление, поведение |
| [2.1.257](https://code.claude.com/docs/en/changelog#2-1-257), 2026-09-01 | Двойной ответ на запрос разрешения; панели не закрывались после shutdown; `CLAUDE_CODE_SUBAGENT_MODEL_FORCE` | исправление, функция |
| [2.1.268](https://code.claude.com/docs/en/changelog#2-1-268), 2026-09-10 | Task-инструменты по умолчанию только на Claude 3.x, Opus 4.0–4.7, Sonnet 4.0–4.6 и Haiku 4.5; возвращённый напарник подхватывал файл агента из недоверенной папки — исправлено | **ломающее**, безопасность |
| [2.1.281](https://code.claude.com/docs/en/changelog#2-1-281), 2026-09-23 | `--setting-sources` передаётся напарникам | исправление |
| [2.1.286](https://code.claude.com/docs/en/changelog#2-1-286), 2026-09-30 | `/compact`, `/clear` и `/rewind` из окна напарника спрашивают подтверждение | безопасность |
| [2.1.288](https://code.claude.com/docs/en/changelog#2-1-288), 2026-10-02 | Plugin-агент, вызванный по имени, получает свои prompt и tools | исправление |
| [2.1.289](https://code.claude.com/docs/en/changelog#2-1-289), 2026-10-03 | `agent.spawn` для напарников в модах; состояния idle и waiting в `$.agent.list()` | функция |
| [2.1.290](https://code.claude.com/docs/en/changelog#2-1-290), 2026-10-05 | `agent_id` у напарника в хуках; TeammateIdle не срабатывает от субагентов и форков напарника | поведение |
| [2.1.295](https://code.claude.com/docs/en/changelog#2-1-295), 2026-10-08 | Первый `SendMessage` у in-process-напарников падал — исправлено; `onFailure: "block"` у хуков (на хуки команды не действует) | исправление |
| без версии | Автоодобрение плана напарника в сессии лида без ревью ([agent-teams](https://code.claude.com/docs/en/agent-teams#have-teammates-plan-before-implementing)); delegate mode пропал из документации ([#25037](https://github.com/anthropics/claude-code/issues/25037)) | **ломающее** (документ) |

#### Сверка с заметками c1, §2.12
- Подтверждено ([agent-teams](https://code.claude.com/docs/en/agent-teams), [changelog](https://code.claude.com/docs/en/changelog#2-1-32), [costs](https://code.claude.com/docs/en/costs#manage-agent-team-costs)):
  - статус experimental, выключено по умолчанию, переменная;
  - запуск в v2.1.32 от 2026-02-05 как research preview;
  - архитектура «лид, напарники, список задач, ящики»;
  - старт в режиме лида, кроме `dontAsk`;
  - in-process-напарники не восстанавливаются через `/resume` (и через `/rewind`);
  - одна команда на сессию, вложенных нет, лид фиксирован;
  - около 7× в plan mode; 3–5 напарников.
- Уточнено:
  - путь ящика — один JSON-файл на агента;
  - имя команды берётся из ID сессии;
  - конфиг удаляется по завершении, задачи остаются;
  - общий список задач есть только при Task-инструментах, а на моделях 5.5 их по умолчанию нет.
- Новое: отключено в `-p` и SDK; автоодобрение планов; `skills` не применяются, `mcpServers` — только в split-pane; удалены TeamCreate, TeamDelete и `teammateDefaultModel`; ловушки fail-open у хуков.

### Выводы и предложения
- **Вывод.** Темп ломающих изменений высокий: около одного в месяц, плюс тихие правки документации вроде автоодобрения планов. Для MVP версию Claude Code надо закрепить и перепроверять поведение команд после каждого обновления (предложение).

### Пробелы
- Даты исчезновения delegate mode и смены семантики одобрения планов не установлены.
- Сроки выхода из experimental не объявлены.

## 6. Смежные функции

### Кратко
Для недельного цикла с человеческими гейтами лучше всего подходят:
- воркфлоу — детерминированный фан-аут внутри стадии;
- обычные или фоновые сессии (agent view), которые запускает человек, плюс файлы в git.

Projects ближе всех к «работе на дни и недели», но это облачная бета на Pro и Max без детерминированных гейтов. `/goal` и advisor — вспомогательные функции, не оркестрация.

### Факты

#### Agent view (`claude agents`)
- Что это: «one screen for all your background sessions». Каждая фоновая сессия — «a full Claude Code conversation that keeps running without a terminal attached» ([agent-view](https://code.claude.com/docs/en/agent-view)). Сессии запускает и ведёт человек ([agents](https://code.claude.com/docs/en/agents)).
- Статус: «Agent view is in research preview». Появился в v2.1.139 от 2026-05-11 ([agent-view](https://code.claude.com/docs/en/agent-view#version-history), [changelog](https://code.claude.com/docs/en/changelog#2-1-139)).
- Ограничения ([agent-view](https://code.claude.com/docs/en/agent-view#limitations)):
  - десять параллельных агентов тратят квоту примерно в десять раз быстрее;
  - «They are preserved across sleep but stop if the machine shuts down»;
  - удаление сессии удаляет её worktree.
- Сессии ведёт отдельный supervisor-процесс; состояние переживает автообновления и рестарты supervisor ([agent-view](https://code.claude.com/docs/en/agent-view)).
- Запрос разрешения переводит сессию в состояние «Needs input», и ответить на него можно, только подключившись к сессии ([agent-view](https://code.claude.com/docs/en/agent-view)).
- Вердикт (предложение): подходит лучше команд для схемы «человек запускает стадию → сессия работает в фоне → человек отвечает на запросы». Сессии восстанавливаются из сохранённого разговора, а напарники — нет. Минусы — research preview и работа только на включённой машине.

#### Projects (claude.ai/code, desktop, mobile)
- Что это: «one ongoing conversation where Claude coordinates a stream of related work». Треды — облачные сессии или, по просьбе, сессии на своём компьютере через Remote Control. У проекта общие инструкции и память. Подходит для работы, которая «spans many tasks over days or weeks, should keep running when your machine is off» ([claude-projects](https://code.claude.com/docs/en/claude-projects), [agents](https://code.claude.com/docs/en/agents)).
- Статус: «Projects are in public beta on Pro and Max plans and rolling out gradually». На Team и Enterprise недоступно ([claude-projects](https://code.claude.com/docs/en/claude-projects)).
- Где недоступно ([claude-projects](https://code.claude.com/docs/en/claude-projects#limitations)): терминальный CLI, VS Code, JetBrains, Bedrock, Vertex и Foundry. Нужен код на github.com и установленное Claude GitHub App ([claude-projects](https://code.claude.com/docs/en/claude-projects#check-the-prerequisites)).
- Гейты ([claude-projects](https://code.claude.com/docs/en/claude-projects#unblock-a-thread-waiting-on-approval)):
  - треды работают в auto mode, если модель его поддерживает;
  - запрос одобрения висит внутри треда, и «Telling Claude in the project conversation to go ahead doesn't reach it»;
  - при нескольких репозиториях на Anthropic-hosted окружении правила разрешений из репозиториев не применяются.
- Предпочтения вроде «Propose threads and wait for my go-ahead» — «instructions Claude keeps to, not enforced settings» ([claude-projects](https://code.claude.com/docs/en/claude-projects#tune-how-claude-runs-a-project)).
- Лимиты и расход ([claude-projects](https://code.claude.com/docs/en/claude-projects#what-draws-on-your-plan), [claude-projects](https://code.claude.com/docs/en/claude-projects#usage-and-cost), [claude-projects](https://code.claude.com/docs/en/claude-projects#limitations)):
  - «The enforced limit is 200 new threads per day across your projects»;
  - тред на лимите плана ждёт сброса;
  - расход идёт быстрее, чем у одной сессии;
  - облачная песочница при неудачном возобновлении стартует с чистого клона, и незакоммиченное теряется;
  - проект принадлежит одному пользователю.
- Вердикт (предложение): единственная функция, рассчитанная на недели и выключенную машину, и человек в ней — естественный гейт. Против:
  - бета;
  - облако Anthropic (доступность российских API из облака и хранение токенов в окружении — не проверено);
  - гейты держатся на инструкциях, а не на конфигурации.

  Годится как оболочка недели, если стадии остаются файлами в git и проверяются скриптами.

#### `/goal`
- Что это: условие завершения, которое после каждого хода проверяет «small fast model». Это «a wrapper around a session-scoped prompt-based Stop hook». Оценщик «doesn't run commands or read files independently» ([goal](https://code.claude.com/docs/en/goal#how-evaluation-works)).
- Работает в `-p`, в desktop и через Remote Control ([goal](https://code.claude.com/docs/en/goal#run-non-interactively)). При возобновлении сессии цель восстанавливается со сбросом счётчиков ([goal](https://code.claude.com/docs/en/goal#resume-with-an-active-goal)). Появилась в v2.1.139 ([changelog](https://code.claude.com/docs/en/changelog#2-1-139)). Подчиняется правилу доверия к хукам ([goal](https://code.claude.com/docs/en/goal#requirements)).
- Вердикт (предложение): полезна внутри стадии («продолжай, пока `validate.py` не напечатает OK в транскрипт»), но это суждение модели, а не гейт. Детерминированная альтернатива — command-хук Stop с `validate.py`.

#### Advisor
- Что это: основная модель консультируется с более сильной в ключевых точках задачи. Advisor получает весь разговор и работает как серверный инструмент ([advisor](https://code.claude.com/docs/en/advisor)).
- Статус: «experimental and requires the Anthropic API»; нет на Bedrock, Claude Platform on AWS, Vertex и Foundry. Доступен «to both subscription and API-billed accounts». Ярлык experimental стоит как минимум с v2.1.117 ([advisor](https://code.claude.com/docs/en/advisor), [changelog](https://code.claude.com/docs/en/changelog#2-1-117)).
- Вердикт (предложение): усилитель качества для директора или судьи на дешёвой модели, например Sonnet с Opus в роли advisor. Это не способ оркестрации, и на выбор между командами и субагентами он не влияет.

#### Dynamic workflows
- Что это: JS-скрипт оркестрирует субагентов; «Who decides what runs next: The script»; масштаб — «Dozens to hundreds of agents per run» ([workflows](https://code.claude.com/docs/en/workflows#when-to-use-a-workflow)).
- Доступность: все платные планы, API, Bedrock, Vertex и Foundry; на Pro включается в `/config` ([workflows](https://code.claude.com/docs/en/workflows)). Запуск — v2.1.154 от 2026-05-28 ([changelog](https://code.claude.com/docs/en/changelog#2-1-154)). Ярлыка preview на странице нет.
- Ограничения ([workflows](https://code.claude.com/docs/en/workflows#behavior-and-limits)):
  - «No mid-run user input … For sign-off between stages, run each stage as its own workflow»;
  - 16 агентов одновременно по умолчанию;
  - 1 000 агентов на прогон.
- Возобновление ([workflows](https://code.claude.com/docs/en/workflows#resume-after-a-pause)):
  - внутри сессии;
  - через перевод в фон;
  - через `claude --resume`, если результаты прогона сохранены.
- Пауза на лимите только в интерактивной сессии с подпиской ([workflows](https://code.claude.com/docs/en/workflows#when-a-run-hits-your-usage-limit)).
- В таблице сравнения у команд «Interruption: Teammates keep running», у воркфлоу — «Resumable in the same session» ([workflows](https://code.claude.com/docs/en/workflows#when-to-use-a-workflow)).
- Вердикт (предложение): лучший примитив для фан-аут-стадий — карточки скаута, пары турнира, данные по идеям. Гейты совета ставятся между отдельными воркфлоу.

#### Страница сравнения (agents.md)
- Пять способов параллельной работы: subagents, agent view (research preview), agent teams (experimental), Projects (public beta на Pro и Max), dynamic workflows. «In every approach the workers are Claude sessions» ([agents](https://code.claude.com/docs/en/agents)).
- Командам соответствует ответ на вопрос «Claude plans, assigns, and supervises a group of workers» ([agents](https://code.claude.com/docs/en/agents#choose-an-approach)).

#### Cross-session messaging (не было в списке задания, но это ближайшая замена команде)
- `SendMessage` и `ListAgents` между своими сессиями: v2.1.224+ на macOS и Linux, на Windows — v2.1.234+ ([cross-session-messaging](https://code.claude.com/docs/en/cross-session-messaging)).
- `claude -p`-воркер тоже может получать сообщения (при `crossSessionInbound: accept`) ([cross-session-messaging](https://code.claude.com/docs/en/cross-session-messaging#non-interactive-sessions)).
- Вердикт (предложение): позволяет стадийным сессиям, которые запускает человек, передавать друг другу находки без командного режима.

### Выводы и предложения
- **Сводный вердикт (предложение):**

| Функция | Зрелость | Подходит для недельного цикла с гейтами? |
|---|---|---|
| Agent teams | experimental, выключено по умолчанию | Нет как основа: живёт одну сессию, не восстанавливается, нет `-p`, гейты fail-open. Годится локально: дебаты судей, разбор конкурирующих гипотез по финалистам |
| Agent view | research preview | Да, как «пульт» стадийных сессий на своей машине |
| Projects | public beta, Pro и Max | Частично: облако и недели, но гейты держатся на инструкциях, бета, облачные риски |
| Workflows | без ярлыка preview, все платные планы | Да, для фан-аута внутри стадии; одна стадия — один воркфлоу |
| `/goal` | без ярлыка | Вспомогательно, внутри стадии |
| Advisor | experimental, только Anthropic API | Вспомогательно, качество решений директора и судьи |

### Пробелы
- Работают ли российские API (Директ, Метрика, ЮKassa, Yandex Cloud) из облачных тредов Projects — не проверено.
- Можно ли в Projects сделать гейт конфигурацией, а не инструкцией — в документации не найдено.

## 7. Ответы на вопросы применимости к MVP

| Вопрос | Ответ | URL |
|---|---|---|
| Можно ли породить напарника из `.claude/agents/<роль>.md`? | Да: «Spawn a teammate using the <type> agent type». Применяются `tools`, `model`, `disallowedTools`, `effort` (последние два только in-process) и тело промпта. `skills` не применяются; `mcpServers` — только в split-pane. Определение из недоверенной папки при возврате напарника не применяется | [agent-teams](https://code.claude.com/docs/en/agent-teams#use-subagent-definitions-for-teammates) |
| Своя модель у каждого напарника? | Да: из промпта спавна или из `model` определения. Фиксируется при спавне, поэтому аналитику на Haiku и на Sonnet нужны два определения | [agent-teams](https://code.claude.com/docs/en/agent-teams#specify-teammates-and-models) |
| Свои инструменты у каждого? | Да: allowlist `tools`. `SendMessage` и Task-инструменты добавляются всегда и не снимаются | [agent-teams](https://code.claude.com/docs/en/agent-teams#use-subagent-definitions-for-teammates) |
| Свой режим разрешений и свои правила? | Режим при спавне — нет: у всех режим лида, кроме `dontAsk`. Менять можно только после спавна. Правила allow, deny и ask общие для сессии | [agent-teams](https://code.claude.com/docs/en/agent-teams#permissions), [#54898](https://github.com/anthropics/claude-code/issues/54898) |
| Свой набор MCP? | In-process — нет, грузятся MCP проекта; ограничивать через `tools`. Split-pane — да, `mcpServers` определения | [agent-teams](https://code.claude.com/docs/en/agent-teams#use-subagent-definitions-for-teammates) |
| Могут ли напарники запускать стадийные скиллы с `disable-model-invocation: true`? | Сами — нет. Только человек, набрав команду в окне напарника | [skills](https://code.claude.com/docs/en/skills#control-who-invokes-a-skill), [agent-teams](https://code.claude.com/docs/en/agent-teams#talk-to-teammates-directly) |
| Кто утверждает планы напарников: человек или лид? | Ни тот, ни другой по существу: Claude Code одобряет план в сессии лида сразу, «without the lead reviewing it». Человека не спрашивают | [agent-teams](https://code.claude.com/docs/en/agent-teams#have-teammates-plan-before-implementing) |
| Куда идут запросы разрешений напарников? | В сессию лида; отвечает человек там. Сообщение агента не считается согласием | [agent-teams](https://code.claude.com/docs/en/agent-teams#permissions) |
| Может ли TaskCompleted быть гейтом валидации по JSON Schema? | Да, условно: exit 2 не даёт закрыть задачу. Но нужны Task-инструменты (на моделях 5.5 по умолчанию их нет); путь артефакта в теме задачи; обёртка без fail-open (`onFailure` не действует; ENOENT в stderr делает блокировку неблокирующей); дублирующая проверка у следующей стадии | [hooks](https://code.claude.com/docs/en/hooks#taskcompleted), [hooks](https://code.claude.com/docs/en/hooks#exit-code-output), [hooks](https://code.claude.com/docs/en/hooks#block-the-action-when-a-hook-fails), [tools-reference](https://code.claude.com/docs/en/tools-reference#task-tool-availability) |
| Может ли exit 2 удержать напарника за работой? | Да: TeammateIdle с exit 2 — «continues working instead of going idle». Работает и без Task-инструментов | [hooks](https://code.claude.com/docs/en/hooks#teammateidle-decision-control) |
| Есть ли общий список задач на Opus, Sonnet и Haiku 5.5? | Не по умолчанию: нужен `CLAUDE_CODE_ENABLE_TODO_TOOLS=1` или `--allowedTools TaskCreate`. Иначе координация только сообщениями | [tools-reference](https://code.claude.com/docs/en/tools-reference#task-tool-availability), [changelog](https://code.claude.com/docs/en/changelog#2-1-233) |
| Работает ли команда в `claude -p`, SDK, GitHub Actions? | Нет: в `-p` и SDK напарники не порождаются. Action построен на SDK, значит команд там нет (вывод) | [agent-teams](https://code.claude.com/docs/en/agent-teams#enable-agent-teams), [github-actions](https://code.claude.com/docs/en/github-actions) |
| В облачных сессиях, routines, Projects? | В облачной сессии включается переменной окружения. Routines и треды Projects — не проверено | [claude-code-on-the-web](https://code.claude.com/docs/en/claude-code-on-the-web#manage-context), [claude-projects](https://code.claude.com/docs/en/claude-projects#how-projects-relate-to-other-claude-code-features) |
| Переживёт ли команда неделю (resume, рестарт, сон машины)? | Нет. `/resume` и `/rewind` не восстанавливают in-process-напарников; конфиг команды удаляется в конце сессии; остаются только задачи. Фоновая работа напарника не переживает процесс лида | [agent-teams](https://code.claude.com/docs/en/agent-teams#limitations), [agent-teams](https://code.claude.com/docs/en/agent-teams#architecture) |
| Что при лимите использования? | Сессии напарников не ждут сброса сами: нужен `/rate-limit-options` вручную | [interactive-mode](https://code.claude.com/docs/en/interactive-mode#start-a-wait-yourself) |
| Максимальный полезный размер команды? | Жёсткого лимита нет. Рекомендация 3–5; 5–6 задач на напарника; на 15 задач — 3 напарника. Для MVP: лид + 3–5 ролей | [agent-teams](https://code.claude.com/docs/en/agent-teams#choose-an-appropriate-team-size) |
| Можно ли запретить лиду исполнять работу самому? | Встроенного режима нет; delegate mode из документации исчез. Есть промпт «Wait for your teammates…». Детерминированно — PreToolUse-хук по пустому `agent_id` или мод `tool.check` (предложение) | [agent-teams](https://code.claude.com/docs/en/agent-teams#wait-for-teammates-to-finish), [hooks](https://code.claude.com/docs/en/hooks#common-input-fields), [#25037](https://github.com/anthropics/claude-code/issues/25037) |
| Вложенность и несколько команд? | Одна команда на сессию; напарники не порождают напарников, но могут запускать субагентов на переднем плане; лид фиксирован | [agent-teams](https://code.claude.com/docs/en/agent-teams#limitations) |
| Изоляция файлов между ролями? | Worktree-изоляции нет; файлы надо разделить между ролями | [agents](https://code.claude.com/docs/en/agents#choose-an-approach) |
| Потолок бюджета на прогон? | `--max-budget-usd` работает только в `-p`. Для команды — только лимит workspace или плана | [cli-reference](https://code.claude.com/docs/en/cli-reference#cli-flags) |
| Во сколько раз дороже? | Около 7× против обычной сессии в plan mode; в остальном расход линейно растёт с числом напарников | [costs](https://code.claude.com/docs/en/costs#manage-agent-team-costs) |
| Подходит ли тип задачи? | Документация: для последовательных задач с зависимостями эффективнее одна сессия или субагенты | [agent-teams](https://code.claude.com/docs/en/agent-teams#when-to-use-agent-teams) |
| Зрелость? | experimental, выключено по умолчанию (8 месяцев после запуска); несколько ломающих изменений | [agent-teams](https://code.claude.com/docs/en/agent-teams), [changelog](https://code.claude.com/docs/en/changelog#2-1-178) |

## 8. Источники

Документация Claude Code (скачано 2026-10-13):
- [agent-teams](https://code.claude.com/docs/en/agent-teams) · [agents](https://code.claude.com/docs/en/agents) · [sub-agents](https://code.claude.com/docs/en/sub-agents) · [hooks](https://code.claude.com/docs/en/hooks) · [hooks-guide](https://code.claude.com/docs/en/hooks-guide) · [tools-reference](https://code.claude.com/docs/en/tools-reference) · [settings-reference](https://code.claude.com/docs/en/settings-reference) · [env-vars](https://code.claude.com/docs/en/env-vars) · [cli-reference](https://code.claude.com/docs/en/cli-reference) · [costs](https://code.claude.com/docs/en/costs) · [changelog](https://code.claude.com/docs/en/changelog) · [interactive-mode](https://code.claude.com/docs/en/interactive-mode) · [permission-modes](https://code.claude.com/docs/en/permission-modes) · [errors](https://code.claude.com/docs/en/errors) · [skills](https://code.claude.com/docs/en/skills) · [headless](https://code.claude.com/docs/en/headless)
- [agent-view](https://code.claude.com/docs/en/agent-view) · [claude-projects](https://code.claude.com/docs/en/claude-projects) · [goal](https://code.claude.com/docs/en/goal) · [advisor](https://code.claude.com/docs/en/advisor) · [workflows](https://code.claude.com/docs/en/workflows) · [cross-session-messaging](https://code.claude.com/docs/en/cross-session-messaging) · [claude-code-on-the-web](https://code.claude.com/docs/en/claude-code-on-the-web) · [routines](https://code.claude.com/docs/en/routines) · [github-actions](https://code.claude.com/docs/en/github-actions) · [feature-availability](https://code.claude.com/docs/en/feature-availability) · [mods reference](https://code.claude.com/docs/en/plugins/mods/reference) · [mods overview](https://code.claude.com/docs/en/plugins/mods/overview) · [индекс llms.txt](https://code.claude.com/docs/llms.txt)

GitHub, anthropics/claude-code (пользовательские issue, только для подтверждения ограничений):
- [#25037 delegate mode cascades to teammates](https://github.com/anthropics/claude-code/issues/25037) · [#32731 teammates have fewer tools](https://github.com/anthropics/claude-code/issues/32731) · [#54898 per-agent permission control gap](https://github.com/anthropics/claude-code/issues/54898) · [#54080](https://github.com/anthropics/claude-code/issues/54080) (по сниппету поиска)

Сторонние (по сниппету поиска):
- [claudefa.st: agent teams controls](https://claudefa.st/blog/guide/agents/agent-teams-controls) · [addyosmani.com: Claude Code Swarms](https://addyosmani.com/blog/claude-code-agent-teams/)

Внутренние:
- `research/mvp-claude-code.md` (разделы 0, 2, 5, 6, 9) · `research/notes/claude-code-harness/c1-primitives.md` (§2.11–2.15)
