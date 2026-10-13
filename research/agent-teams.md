# Agent teams не держат неделю LLM-компании

**Ответ: MVP на agent teams не строить.** Оставить субагентов и стадийные скиллы, как в [ТЗ MVP](mvp-claude-code.md). Команда живёт, пока открыт интерактивный терминал лида. В `claude -p` и Agent SDK напарники не создаются ([agent-teams](https://code.claude.com/docs/en/agent-teams#enable-agent-teams)). `/resume` их не восстанавливает ([agent-teams](https://code.claude.com/docs/en/agent-teams#limitations)). А неделя ТЗ — это понедельник, вторник, ежедневный мониторинг и два гейта совета. Стадии 2–7 идут строго по очереди. Для такой работы документация сама советует субагентов или одну сессию ([agent-teams](https://code.claude.com/docs/en/agent-teams#when-to-use-agent-teams)). Гейтов команда не добавляет. План напарника Claude Code одобряет сам, без лида и без человека ([agent-teams](https://code.claude.com/docs/en/agent-teams#have-teammates-plan-before-implementing)). Хуки команды при сбое пропускают действие ([hooks](https://code.claude.com/docs/en/hooks#block-the-action-when-a-hook-fails)). Включённый флаг ломает и то, что уже есть: именованный субагент директора становится напарником, и его результат теряется. Мейнтейнер подтвердил этот баг 17 августа 2026 года ([#71723](https://github.com/anthropics/claude-code/issues/71723)). Цена — около 7× токенов в plan mode по данным Anthropic ([costs](https://code.claude.com/docs/en/costs#manage-agent-team-costs)). В экстраполяции на MVP это $90–190 в неделю вместо $13–27. Переписка агентов совпадает с задачей ровно в одном месте — в споре о топ-3 перед гейтом 1. Его стоит пробовать только как A/B-эксперимент в отдельном профиле. Турнир командой не делать: судьи должны быть независимы.

> **Как читать.** Дата — 2026-10-13. Версия Claude Code — v2.1.296 от 2026-10-09 ([changelog](https://code.claude.com/docs/en/changelog#2-1-296)). Модели — как в ТЗ: Opus 5.5, Sonnet 5.5, Haiku 5.5. Метки: «не проверено» — нет подтверждения первичным источником; «по сниппету поиска» — факт виден только в поисковой выдаче, страница не открывалась; «вывод» — следствие из процитированных фактов, отдельно не задокументированное; «предложение» — рекомендация этого отчёта; «оценка», «допущение», «экстраполяция» — прогнозы и пересчёты без замеров; «[совет]» — параметр, который основатель утверждает до запуска. Происхождение цифр: **[вендор]** — заявление Anthropic; **[независимо]** — внешнее исследование; **[самоотчёт]** — цифры автора issue или поста, их никто не проверял. Термины: «команда» — agent teams, «лид» — lead, «напарник» — teammate. Все источники и детали — в заметках [t1-mechanics](notes/agent-teams/t1-mechanics.md) (документация) и [t2-practice](notes/agent-teams/t2-practice.md) (практика). Отчёт продолжает [первый отчёт](ready-made-office.md), [второй отчёт](claude-code-harness.md) и [ТЗ MVP](mvp-claude-code.md); номера разделов ТЗ даны как «ТЗ, §N».

## 1. Вердикт: MVP остаётся на субагентах, команде — один эксперимент перед гейтом 1

**Решение (предложение): MVP на agent teams не строить.** Роли остаются субагентами, стадии — скиллами, состояние — файлами в git (ТЗ, §2, §5, §6 — [ТЗ MVP](mvp-claude-code.md)). Флаг `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS` в репозитории компании держать в `0`.

**Причина первая — жизненный цикл.** Команда существует, пока жив процесс интерактивной сессии лида. В `claude -p`, включая Agent SDK, «Claude doesn't spawn teammates» ([agent-teams](https://code.claude.com/docs/en/agent-teams#enable-agent-teams)). `/resume` и `/rewind` напарников не восстанавливают ([agent-teams](https://code.claude.com/docs/en/agent-teams#limitations)). Конфиг команды удаляется в конце сессии ([agent-teams](https://code.claude.com/docs/en/agent-teams#architecture)). Неделя ТЗ растянута на семь дней: `/cycle` в понедельник, `/landing` во вторник, `/monitor` каждый день (ТЗ, §12). Одна команда её не проживёт.

**Причина вторая — форма работы.** Стадии 2–7 идут друг за другом. Каждая читает артефакт предыдущей. Документация говорит прямо: «For sequential tasks, same-file edits, or work with many dependencies, a single session or subagents are more effective» ([agent-teams](https://code.claude.com/docs/en/agent-teams#when-to-use-agent-teams)) [вендор]. Параллелизм в цикле есть только внутри стадий: идеи скаута, пары турнира. Там переписка агентов не нужна.

**Причина третья — гарантии.** Команда не даёт ни одного нового гейта. Зато ослабляет три механизма ТЗ: изоляцию ролей, проверку артефактов хуками и работу именованных субагентов директора. Подробности и доказательства — в разделе 4.

**Где команда может пригодиться.** Только на стадии 7 `/board-pack` и только как A/B-эксперимент. Три напарника Sonnet 5.5 спорят о топ-3 перед гейтом 1. Данные они только читают, итог пишет лид (раздел 7). Это единственное место, где сценарий документации «share findings, challenge each other» совпадает с задачей ([agent-teams](https://code.claude.com/docs/en/agent-teams#when-to-use-agent-teams)). Турнир командой не делать. Вееры скаута и турнира лучше отдать параллельным субагентам или воркфлоу (раздел 8).

**Когда пересмотреть (предложение).** При трёх условиях сразу. Anthropic выводит команды из experimental. В коде исправлены потеря команды после компакции, idle-шум лида и самозахват задач без учёта ролей ([#23620](https://github.com/anthropics/claude-code/issues/23620), [#47930](https://github.com/anthropics/claude-code/issues/47930), [#93668](https://github.com/anthropics/claude-code/issues/93668)). Появляется независимый замер качества команд на аналитических задачах.

**Плюсы решения.** ТЗ не меняется по сути. Работают зрелые функции. Открыт путь к запускам без человека через `claude -p` и к гибридному варианту (ТЗ, §18). Расход предсказуем: $13–27 в неделю (оценка [второго отчёта](claude-code-harness.md)). **Минусы.** Живого спора агентов нет. Если команды созреют, вопрос придётся поднять снова. Польза состязательного совета не измерена — её покажет только эксперимент из раздела 10.

## 2. Agent teams на 2026-10-13: лид, напарники, ящики-файлы и восемь месяцев в experimental

### Как устроена команда

**Команда — это одна интерактивная сессия-лид и напарники, которых она порождает.** Лид вызывает инструмент Agent с параметром `name`. С v2.1.178 инструменты `TeamCreate` и `TeamDelete` удалены: «Every session now has one implicit team» ([changelog](https://code.claude.com/docs/en/changelog#2-1-178)). Напарник — отдельный экземпляр Claude Code со своим контекстом. Он загружает CLAUDE.md, MCP-серверы и скиллы проекта и получает промпт спавна. Историю лида он не видит: «The lead's conversation history does not carry over» ([agent-teams](https://code.claude.com/docs/en/agent-teams#context-and-communication)).

**Связь идёт через файлы.** `SendMessage` пишет конкретному напарнику по имени. Широковещательной рассылки нет: «send one message per recipient» ([agent-teams](https://code.claude.com/docs/en/agent-teams#context-and-communication)). Ящик каждого агента — JSON-файл `~/.claude/teams/{team-name}/inboxes/{agent-name}.json`. Конфиг команды лежит там же и удаляется в конце сессии. Задачи хранятся в `~/.claude/tasks/{team-name}/`, остаются локально и чистятся через 30 дней ([agent-teams](https://code.claude.com/docs/en/agent-teams#architecture), [settings-reference](https://code.claude.com/docs/en/settings-reference#cleanupperioddays)). Лиду не нужно опрашивать напарников: «The lead doesn't need to poll for updates» ([agent-teams](https://code.claude.com/docs/en/agent-teams#context-and-communication)). Закончив, напарник присылает уведомление с финальным ответом; до v2.1.251 там было пустое «available» ([changelog](https://code.claude.com/docs/en/changelog#2-1-251)).

**Общий список задач есть, но не на моделях MVP.** Задачи бывают pending, in progress и completed, с зависимостями. Лид назначает задачу или напарник берёт свободную сам; захват защищён файловой блокировкой ([agent-teams](https://code.claude.com/docs/en/agent-teams#assign-and-claim-tasks)). Но инструменты `TaskCreate`, `TaskGet`, `TaskUpdate` и `TaskList` по умолчанию есть «only on Opus 4.7, Sonnet 4.6, Haiku 4.5, and earlier models» ([tools-reference](https://code.claude.com/docs/en/tools-reference#task-tool-availability)). На Opus, Sonnet и Haiku 5.5 команда координируется только сообщениями, пока не задан `CLAUDE_CODE_ENABLE_TODO_TOOLS=1`.

**Права общие, человек отвечает в окне лида.** Напарники стартуют в режиме разрешений лида, кроме `dontAsk`. Задать свой режим каждому при спавне нельзя. Запросы разрешений напарников всплывают в сессии лида ([agent-teams](https://code.claude.com/docs/en/agent-teams#permissions)). Человек может писать любому напарнику напрямую: Enter открывает его транскрипт, Escape прерывает ход, `x` останавливает ([agent-teams](https://code.claude.com/docs/en/agent-teams#talk-to-teammates-directly)). Режимы отображения: in-process по умолчанию, split panes в tmux или iTerm2. Split panes не работают в терминале VS Code, Windows Terminal и Ghostty ([agent-teams](https://code.claude.com/docs/en/agent-teams#limitations)).

**Жёсткие рамки.** Одна команда на сессию. Напарники не порождают своих напарников. Лидерство не передаётся ([agent-teams](https://code.claude.com/docs/en/agent-teams#limitations)). Лимита на размер нет, рекомендация — «Start with 3-5 teammates for most workflows» ([agent-teams](https://code.claude.com/docs/en/agent-teams#choose-an-appropriate-team-size)). Изоляции файлов нет: «Agent teams don't isolate teammates in worktrees, so partition the work» ([agents](https://code.claude.com/docs/en/agents#choose-an-approach)).

### Статус: research preview в феврале, experimental в октябре

**Функция вышла 5 февраля 2026 года в v2.1.32** как «research preview agent teams feature … (token-intensive feature, requires setting CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1)» ([changelog](https://code.claude.com/docs/en/changelog#2-1-32)). Анонс вместе с Opus 4.6 сузил область: «best for tasks that split into independent, read-heavy work like codebase reviews» ([Introducing Claude Opus 4.6](https://www.anthropic.com/news/claude-opus-4-6)) [вендор]. **Через восемь месяцев статус тот же: «experimental and disabled by default»** ([agent-teams](https://code.claude.com/docs/en/agent-teams)). Объявления GA нет. В API-платформе Managed Agents одноранговых команд нет вовсе: только субагенты, dynamic workflows и advisor, а workflows рекомендованы для «Most work that needs more than one agent» ([platform docs](https://platform.claude.com/docs/en/managed-agents/multiagent-orchestration)) [вендор]. Вывод: промышленная ставка Anthropic на многоагентность — субагенты и workflows, а не команды.

**Что уточнилось со второго отчёта.** В [заметках c1](notes/claude-code-harness/c1-primitives.md), §2.12, команды описаны как экспериментальные, невосстановимые и дорогие. Это подтвердилось. Новое: команд нет в `-p` и SDK; план напарника одобряется автоматически; `skills` из определения роли не применяются, `mcpServers` — только в split-pane; на моделях 5.5 нет общего списка задач по умолчанию; хуки команды открывают путь при сбое ([заметки t1](notes/agent-teams/t1-mechanics.md), §5).

### Ломающие изменения: шесть с версией и два без версии

| Версия, дата | Что изменилось | Чем это бьёт по MVP |
|---|---|---|
| [2.1.77](https://code.claude.com/docs/en/changelog#2-1-77), 2026-03-17 | Agent больше не принимает `resume`; вместо него `SendMessage` | Скрипты и SOP с `resume` перестают работать |
| [2.1.178](https://code.claude.com/docs/en/changelog#2-1-178), 2026-06-15 | Удалены `TeamCreate` и `TeamDelete`; одна неявная команда на сессию; `team_name` игнорируется | Нельзя держать две команды, например «совет» и «турнир» |
| [2.1.232](https://code.claude.com/docs/en/changelog#2-1-232), 2026-08-13 | Не ломающее, но важное: fork mode по умолчанию; спавны, не ставшие напарниками, по умолчанию идут в фон | Меняется поведение обычных субагентов `/cycle` — сверить на закреплённой версии |
| [2.1.233](https://code.claude.com/docs/en/changelog#2-1-233), 2026-08-14 | Task- и Todo-инструменты убраны с Opus 4.8, Sonnet 5, Fable 5, Mythos 5 и новее | На моделях 5.5 пропал общий список задач и хуки TaskCreated, TaskCompleted |
| [2.1.234](https://code.claude.com/docs/en/changelog#2-1-234), 2026-08-17 | Удалена настройка `teammateDefaultModel` | Модели напарников задаются иначе |
| [2.1.251](https://code.claude.com/docs/en/changelog#2-1-251), 2026-08-28 | Новый порядок выбора модели напарника | Тот же конфиг даёт другую модель |
| [2.1.268](https://code.claude.com/docs/en/changelog#2-1-268), 2026-09-10 | Task-инструменты по умолчанию только на Claude 3.x, Opus 4.0–4.7, Sonnet 4.0–4.6, Haiku 4.5 | Закрепляет пункт 2.1.233 |
| без версии | План напарника одобряется в сессии лида автоматически ([agent-teams](https://code.claude.com/docs/en/agent-teams#have-teammates-plan-before-implementing)); delegate mode исчез из документации ([#25037](https://github.com/anthropics/claude-code/issues/25037)) | «Утверждение плана» перестало быть гейтом; нет режима «лид только координирует» |

Шесть ломающих изменений с версией за восемь месяцев — примерно одно в месяц. Сверху — две тихие правки документации без версии. Записи в changelog о переходе к автоодобрению планов нет; когда оно появилось — не проверено ([заметки t1](notes/agent-teams/t1-mechanics.md), §2). **Предложение:** если команды где-то используются, версию Claude Code закрепить и перепроверять поведение после каждого обновления. **Плюс:** нет сюрпризов посреди цикла. **Минус:** исправления приходят с задержкой.

## 3. Маппинг ТЗ: из тринадцати элементов команда улучшает один, и то под вопросом

**Как выглядел бы понедельник на командах (вывод).** Основатель запускает `claude --agent director` с включённым флагом. Директор-лид порождает напарников из `.claude/agents/`: scout, два аналитика, judge. Стадии идут по очереди, поэтому напарники в основном ждут. А каждый живой напарник тратит токены, пока не завершится ([costs](https://code.claude.com/docs/en/costs#agent-team-token-costs)). В долгой сессии лид после компакции может потерять команду; в одном отчёте это воспроизводится примерно за 45 минут ([#23620](https://github.com/anthropics/claude-code/issues/23620)) [самоотчёт]. Во вторник `/landing` идёт уже в другой сессии с другой командой. Ежедневный `/monitor` — команда из одного агента, то есть обычная сессия.

Таблица ниже — предложение по фактам заметок. «Вердикт» сравнивает команду с субагентами ТЗ.

| Элемент ТЗ | Как выглядело бы в команде | Что ломается или улучшается | Вердикт |
|---|---|---|---|
| **Роли** (ТЗ, §5): пять субагентов; директор — `claude --agent director` с `Agent(scout, analyst, marketer, judge)` | Лид — director. Напарники из тех же файлов: «Spawn a teammate using the … agent type» ([agent-teams](https://code.claude.com/docs/en/agent-teams#use-subagent-definitions-for-teammates)). Аналитику — два определения, Haiku и Sonnet: модель фиксируется при спавне ([agent-teams](https://code.claude.com/docs/en/agent-teams#talk-to-teammates-directly)) | Ломается: из определения применяются `tools`, `model` и тело; `skills` — никогда; `mcpServers` — только в split-pane; режим разрешений общий для всех ([agent-teams](https://code.claude.com/docs/en/agent-teams#permissions)). Улучшается: человек говорит с любой ролью напрямую | Хуже |
| **Стадии-скиллы** (ТЗ, §6): у всех `disable-model-invocation: true`; запускает человек или `/cycle` | Стадию поручают напарнику. Но скилл с этим флагом модель сама не вызывает: «Claude can't invoke the skill on its own» ([skills](https://code.claude.com/docs/en/skills#control-who-invokes-a-skill)). Вызвать его может только человек, набрав `/scout` в окне напарника ([agent-teams](https://code.claude.com/docs/en/agent-teams#talk-to-teammates-directly)) | Ломается: SOP переносить в тело роли или в промпт спавна. Либо снимать флаг и защищаться хуками, которые при сбое пропускают (раздел 4.5) | Ломается |
| **Недельный запуск** (ТЗ, §2, §12): `/cycle` по очереди запускает стадии 2–7 и ждёт `decision.md` | Лид раздаёт стадии напарникам по одной. Параллелизма нет: стадия ждёт предыдущую | Ломается: в `-p` команд нет ([agent-teams](https://code.claude.com/docs/en/agent-teams#enable-agent-teams)); лид берётся за задачи сам или «can stop early» ([agent-teams](https://code.claude.com/docs/en/agent-teams#agents-stopping-early)); после компакции теряет команду ([#23620](https://github.com/anthropics/claude-code/issues/23620)) [самоотчёт]. Улучшается: ход каждой роли видно вживую | Хуже |
| **Гейт 1 — GO по идее** (ТЗ, §6 `/landing`, §7 `gate_check.py`): `decision.md` коммитит человек | «Require plan approval» — не гейт: план одобряется автоматически ([agent-teams](https://code.claude.com/docs/en/agent-teams#have-teammates-plan-before-implementing)). Хук TaskCreated может удалять задачи стадий 8–9, пока `gate_check.py` не вернёт 0 ([hooks](https://code.claude.com/docs/en/hooks#taskcreated-decision-control)) — предложение | Ломается: поручение сообщением обходит TaskCreated (вывод), нужен ещё PreToolUse на запись в `landing/` и `smoke/`. Улучшается: ничего | Без изменений: гейт — файл и человек |
| **Гейт 2 — ступени бюджета** (ТЗ, §6 `/monitor`, §11.1): ежедневная сводка, решение человека в кабинете | Команда между днями не живёт: фоновая работа напарника «can't outlive the lead's process» ([agent-teams](https://code.claude.com/docs/en/agent-teams#limitations)) | Ломается: держать команду неделю — держать открытым терминал лида; любой рестарт её убивает. Улучшается: ничего | Неприменимо |
| **Бюджет LLM** (ТЗ, §3 п. 9, §15): лимит workspace; `--max-budget-usd` на стадию в `-p` | `--max-budget-usd` — «(print mode only)» ([cli-reference](https://code.claude.com/docs/en/cli-reference#cli-flags)). Потолок у интерактивной команды — только лимит workspace или подписки. Около 7× токенов в plan mode ([costs](https://code.claude.com/docs/en/costs#manage-agent-team-costs)) [вендор] | Ломается: статистика напарника показывает «0 tool uses / 0 tokens», общей стоимости команды нет ([#42005](https://github.com/anthropics/claude-code/issues/42005), [#60591](https://github.com/anthropics/claude-code/issues/60591)) [самоотчёт] — учёт в `manifest.json` труднее | Хуже |
| **Аудит** (ТЗ, §9.3): `audit.sh` на каждый вызов → `audit.jsonl`; git — журнал решений | Хуки срабатывают и у напарников. С v2.1.290 у in-process напарника в хуке есть `agent_id` ([hooks](https://code.claude.com/docs/en/hooks#common-input-fields), [changelog](https://code.claude.com/docs/en/changelog#2-1-290)) | Улучшается: строки аудита размечены по роли. Ломается: одна рабочая копия без worktree — гонки за индекс git, если коммитит не один агент (вывод) | Нейтрально, если коммитит только лид или человек |
| **Состояние между неделями** (ТЗ, §4): `manifest.json` и артефакты в git | Конфиг команды удаляется в конце сессии. Задачи лежат локально в `~/.claude/tasks/` и чистятся через 30 дней ([agent-teams](https://code.claude.com/docs/en/agent-teams#architecture)) | Ломается: появляется второй, ненадёжный источник статуса — список задач. Улучшается: ничего | Без изменений: истина только в git |
| **Турнир** (ТЗ, §6 `/tournament`): judge судит каждую пару дважды, A–B и B–A; пары строит `swiss.py` по seed | Судьи-напарники, пары — задачи из общего списка | Ломается: переписка судей ломает независимость вердиктов; самозахват задач не учитывает роли ([#93668](https://github.com/anthropics/claude-code/issues/93668)) [самоотчёт]; цена растёт кратно | Не командой (раздел 7.2) |
| **Пакет совета** (ТЗ, §6 `/board-pack`): директор пишет `board-pack.md` и шаблон `decision.md` | Три напарника Sonnet — «за», «против», «аудитор цифр»; только чтение; итог пишет лид | Улучшается: спор о топ-3 до гейта 1 — сценарий «challenge each other» ([agent-teams](https://code.claude.com/docs/en/agent-teams#when-to-use-agent-teams)). Ломается: польза дебатов однотипных моделей не доказана (раздел 5.3); стадия дорожает | Опциональный A/B-эксперимент (разделы 7, 10) |
| **Мониторинг** (ТЗ, §6 `/monitor`, §12): Haiku раз в день, `/loop`, позже `-p` | Команда из одного агента бессмысленна. Живые напарники тратят токены, пока не завершатся ([costs](https://code.claude.com/docs/en/costs#agent-team-token-costs)) | Ломается: путь в `-p` закрыт. Улучшается: ничего | Не командой |
| **Скаут** (ТЗ, §6 `/scout`): один субагент; 70–80 сырых идей → 50 карточек; дедупликация `registry.py` | Три напарника по сегментам поиска | Ломается: переписка не нужна — дедупликация уже детерминированная. Без worktree «Two teammates editing the same file leads to overwrites» ([agent-teams](https://code.claude.com/docs/en/agent-teams#avoid-file-conflicts)). Дубли напарников ([#55586](https://github.com/anthropics/claude-code/issues/55586)) [самоотчёт] | Не командой: веер — субагентами или воркфлоу (раздел 8) |
| **Данные только на чтение, токены** (ТЗ, §9.1, §16): пишущих токенов нет; скаут без MCP | In-process напарник игнорирует `mcpServers` роли и грузит MCP проекта — yamcp и MCP ЮKassa ([agent-teams](https://code.claude.com/docs/en/agent-teams#use-subagent-definitions-for-teammates)). Права общие для сессии ([#54898](https://github.com/anthropics/claude-code/issues/54898)) | Ломается: «скаут без MCP» держится только на списке `tools`. Денежный принцип цел: пишущих токенов по-прежнему нет, тратить нечем | Изоляция хуже, деньги в безопасности |

**Что следует из таблицы.** Хуже становятся пять элементов: роли, стадии-скиллы, запуск недели, бюджет и изоляция данных. Ещё четыре командой делать не нужно: гейт 2, турнир, мониторинг, скаут. Три остаются как были: гейт 1, аудит, состояние. Улучшение возможно в одном месте — в пакете совета. И его надо доказать экспериментом. Главный принцип ТЗ «Claude думает, человек платит» команда не ломает: денежные действия всё равно делает человек, а пишущих токенов у агентов нет (ТЗ, §0).

## 4. Девять блокеров: команда не работает без человека, забывает себя после рестарта и не держит гейты

Каждый блокер ниже — факт из документации или issue, затем следствие для ТЗ. Детали — в [заметках t1](notes/agent-teams/t1-mechanics.md), §1–3, и [t2](notes/agent-teams/t2-practice.md), §3.

### 4.1. Нет headless: ни `-p`, ни SDK, ни GitHub Actions

**Факт.** «In non-interactive mode with the `-p` flag, including Agent SDK sessions, Claude doesn't spawn teammates». Именованный субагент там работает как обычный субагент ([agent-teams](https://code.claude.com/docs/en/agent-teams#enable-agent-teams)). Claude Code GitHub Action «is built on the SDK» ([github-actions](https://code.claude.com/docs/en/github-actions)). Вывод: в Actions команд тоже нет. Лимит `--max-budget-usd` работает только в print mode ([cli-reference](https://code.claude.com/docs/en/cli-reference#cli-flags)). В облачной сессии команды включаются переменной окружения ([claude-code-on-the-web](https://code.claude.com/docs/en/claude-code-on-the-web#manage-context)). Работают ли они в routines и тредах Projects — не проверено.

**Для ТЗ.** ТЗ планирует перевести `/monitor` в `-p` позже (ТЗ, §2, §12). А триггер перехода в гибрид — «цикл должен работать без человека за клавиатурой» (ТЗ, §18). Команда закрывает оба пути. Всё, что сделано на командах, при автоматизации придётся переписать на субагентов.

### 4.2. Не переживает `/resume`, рестарт и закрытие сессии

**Факт.** «`/resume` and `/rewind` do not restore in-process teammates. After resuming a session, the lead may attempt to message teammates that no longer exist» ([agent-teams](https://code.claude.com/docs/en/agent-teams#limitations)). Конфиг команды удаляется в конце сессии, задачи остаются ([agent-teams](https://code.claude.com/docs/en/agent-teams#architecture)). Фоновая работа напарника «can't outlive the lead's process» ([agent-teams](https://code.claude.com/docs/en/agent-teams#limitations)). Перевод сессии в фон через agent view стартует новый процесс из сохранённого разговора, и напарников нет в списке того, что переносится ([agent-view](https://code.claude.com/docs/en/agent-view#what-carries-over-when-you-background)); вывод — они теряются (не проверено). Компакция тоже бьёт: после неё лид «completely loses awareness of the team», не может писать напарникам и заново раздаёт идущую работу. Это «the number 1 most frequent bug I hit with Agent Teams»; issue открыт с 6 февраля 2026 года, от мейнтейнеров только метка ([#23620](https://github.com/anthropics/claude-code/issues/23620)) [самоотчёт]. Запрос на «graceful restart» открыт ([#90247](https://github.com/anthropics/claude-code/issues/90247)). На лимите использования сессии напарников не ждут сброса сами — нужен ручной `/rate-limit-options` ([interactive-mode](https://code.claude.com/docs/en/interactive-mode#start-a-wait-yourself)).

**Для ТЗ.** Состояние и так живёт в git (ТЗ, §4, §6). Поэтому данные не пропадут. Пропадёт время: рестарт терминала, обновление Claude Code или лимит посреди `/cycle` — и команду надо собирать заново. Повторный запуск `/cycle` по ТЗ не пересоздаёт готовые стадии (ТЗ, §6); это спасает артефакты, но не потраченные токены.

### 4.3. План напарника одобряется автоматически, без человека

**Факт.** Напарник, порождённый в plan mode, сначала пишет план. Затем «it sends a plan approval request to the lead. Claude Code approves the plan in the lead's session as soon as the request arrives, without the lead reviewing it» ([agent-teams](https://code.claude.com/docs/en/agent-teams#have-teammates-plan-before-implementing)). Документ называет это так: «Plan approval is the designed exception: the lead session grants teammate plan approvals without a separate prompt to you» ([agent-teams](https://code.claude.com/docs/en/agent-teams#permissions)). Настройки, которая делает человека утверждающим, в документации нет ([заметки t1](notes/agent-teams/t1-mechanics.md), §2). Если запрос не записался в ящик, напарник застревает в plan mode ([errors](https://code.claude.com/docs/en/errors#failed-to-write-to-a-teammate-inbox)).

**Для ТЗ.** Гейт совета нельзя строить на «утверждении плана». Человеческие гейты в команде те же, что без неё: `ask`-правила (запрос всплывает у лида), `decision.md` + `gate_check.py`, ручной запуск стадии. Есть и хорошая новость: сообщение агента не заменяет согласие человека. «A teammate can't approve a permission prompt or supply consent on your behalf» ([agent-teams](https://code.claude.com/docs/en/agent-teams#messages-between-agents)).

### 4.4. Инструменты задач по умолчанию выключены на моделях 5.5

**Факт.** `TaskCreate`, `TaskGet`, `TaskUpdate`, `TaskList` и `TodoWrite` по умолчанию есть «only on Opus 4.7, Sonnet 4.6, Haiku 4.5, and earlier models». На остальных — только по opt-in: `CLAUDE_CODE_ENABLE_TODO_TOOLS=1` («Export … before you start Claude Code») или имя инструмента в `--allowedTools` либо `--tools`. In-process напарник «follows your session»; split-pane напарник — отдельный процесс, «so its own model decides». Без инструментов «an agent coordinates with its team through messages instead of the shared task list» ([tools-reference](https://code.claude.com/docs/en/tools-reference#task-tool-availability)). Хук TaskCreated тогда не срабатывает: «In a session without the Task tools, this event doesn't fire» ([hooks](https://code.claude.com/docs/en/hooks#taskcreated)). Изменение шло в два шага — v2.1.233 и v2.1.268 ([changelog](https://code.claude.com/docs/en/changelog#2-1-233), [changelog](https://code.claude.com/docs/en/changelog#2-1-268)).

**Для ТЗ.** Почти все статьи про команды описывают общий список задач. На Opus, Sonnet и Haiku 5.5 его по умолчанию нет. Нет и гейтов на TaskCreated и TaskCompleted. Работает ли переменная через `env` в settings.json — не проверено: документация говорит только «export».

### 4.5. Хуки команды блокируют кодом 2, но при сбое пропускают

**Факт.** Код 2 работает так. TeammateIdle: напарник «continues working instead of going idle» ([hooks](https://code.claude.com/docs/en/hooks#teammateidle-decision-control)). TaskCreated: «Claude Code deletes the task and returns your message to Claude as the tool's error» ([hooks](https://code.claude.com/docs/en/hooks#taskcreated-decision-control)). TaskCompleted: «the task is not marked as completed» ([hooks](https://code.claude.com/docs/en/hooks#taskcompleted-decision-control)). Любой другой ненулевой код — «non-blocking error», действие продолжается ([hooks](https://code.claude.com/docs/en/hooks#exit-code-output)). Хук, упавший по таймауту, отменяется, решения нет ([hooks](https://code.claude.com/docs/en/hooks#timeouts)). Опция `onFailure: "block"` (v2.1.295+) «has no effect» на Stop, SubagentStop, TaskCompleted и TeammateIdle: «Claude can't repair a hook that won't run» ([hooks](https://code.claude.com/docs/en/hooks#block-the-action-when-a-hook-fails)). TaskCreated в этом списке не назван; по умолчанию он, как любой хук, при сбое пропускает. Есть ловушка: если TaskCompleted выходит с кодом 2, stdout пуст, а в stderr есть «No such file or directory», Claude Code считает это неблокирующей ошибкой ([hooks](https://code.claude.com/docs/en/hooks#exit-code-output)). Лимит в 8 продолжений подряд (`CLAUDE_CODE_STOP_HOOK_BLOCK_CAP`) описан только для Stop и SubagentStop ([env-vars](https://code.claude.com/docs/en/env-vars)). Защита от бесконечного цикла кода 2 на TeammateIdle и TaskCompleted — не проверено.

**Для ТЗ.** Это совпадает с правилом ТЗ «Хуки ненадёжны как гейт» (ТЗ, §16). Хуки команды — журнал с подталкиванием, а не замок. Если их всё же использовать, нужны четыре меры: обёртка с выходом 2 при любой ошибке, непустой stdout, никакого «No such file or directory» в stderr и повторная проверка артефакта на стадии, которая его читает (раздел 9).

### 4.6. Флаг команд превращает именованных субагентов в напарников и теряет их результаты

**Факт.** С включённым флагом «a subagent that Claude names launches as a teammate, so teams can form even when you didn't ask for one». Подтверждения нет: «Claude Code doesn't ask you to confirm the launch». Напарником не становится только форк или вызов с `isolation` в самом вызове ([agent-teams](https://code.claude.com/docs/en/agent-teams#how-claude-starts-agent-teams)). `isolation` во frontmatter запуск напарника не отменяет: «the teammate then runs in the main session's working directory» ([sub-agents](https://code.claude.com/docs/en/sub-agents#subagent-names)). В issue #71723 параметр `name` тихо переводит субагента в протокол напарника, и результат до вызывающего не доходит. Около 20 минут потерь на случай; один пользователь потерял трёх воркеров за день; в проекте автора — 138 именованных и 191 безымянный запуск Agent. 17 августа 2026 года мейнтейнер воспроизвёл баг: «Confirmed as a bug», исправление «already in progress». Обход — не передавать `name` ([#71723](https://github.com/anthropics/claude-code/issues/71723)) [самоотчёт + подтверждение мейнтейнера]. Выключение — переменная в `0`; она перечитывается при каждом спавне. `0` в user settings перекрывает export из shell, но project, local, `--settings` и managed settings могут включить команды снова ([agent-teams](https://code.claude.com/docs/en/agent-teams#claude-spawns-teammates-instead-of-subagents)).

**Для ТЗ — прямой риск для директора.** `/cycle` порождает субагентов стадий (ТЗ, §2, §6). Если флаг где-то включён, а директор даст субагенту имя, стадия молча уйдёт в напарника. Он будет работать в общем каталоге, в режиме лида, и его ответ директору не придёт. Файловый обмен ТЗ смягчает удар: артефакт стадии лежит на диске, а следующая стадия его валидирует (вывод). Но директор не узнает, что стадия кончилась. Он будет ждать или запустит её заново. Мера — флаг `0` в проектных настройках и запуск стадий без `name` (раздел 9).

### 4.7. Роли из `.claude/agents/*.md` переносятся частично

**Факт.** Напарника можно породить из определения субагента project-, user-, managed- или plugin-скоупа. Но поля применяются не все ([agent-teams](https://code.claude.com/docs/en/agent-teams#use-subagent-definitions-for-teammates)):

| Поле определения | In-process (по умолчанию) | Split-pane | Что значит для ТЗ |
|---|---|---|---|
| `tools` | Применяется. Всегда добавляется `SendMessage`, а при Task-инструментах — `TaskCreate`, `TaskGet`, `TaskList`, `TaskUpdate` | Применяется | Главный и почти единственный способ сузить роль |
| `model` | Применяется, если промпт спавна не называет модель | Применяется | Модель фиксируется при спавне: аналитику с Haiku и Sonnet нужны два определения |
| `disallowedTools` | Применяется, но `SendMessage` и Task-инструменты остаются | Не применяется | В split-pane запреты роли пропадают |
| `effort` | Применяется по документации; в открытом issue — игнорируется ([#80569](https://github.com/anthropics/claude-code/issues/80569)) [самоотчёт] | Не применяется | Сверить вживую |
| Тело файла | Дописывается к системному промпту | Заменяет системный промпт | SOP роли ведёт себя по-разному в двух режимах |
| `skills` | Не применяется | Не применяется | Напарник грузит все скиллы проекта и пользователя |
| `mcpServers` | Игнорируется: грузятся MCP проекта и пользователя | Применяется | «Скаут без MCP» держится только на `tools` |
| `permissionMode`, `hooks`, `maxTurns`, `memory`, `background` | В списке не упомянуты — не проверено | То же | `maxTurns: 60` скаута из ТЗ, §5, может не действовать |
| `isolation` во frontmatter | Не отменяет запуск напарником; работает в каталоге главной сессии ([sub-agents](https://code.claude.com/docs/en/sub-agents#subagent-names)) | То же | Субагенты «в worktree» теряют изоляцию |

Режим разрешений при спавне у всех общий — режим лида, кроме `dontAsk` ([agent-teams](https://code.claude.com/docs/en/agent-teams#permissions)). Правила allow, deny и ask общие для сессии: «Claude Code's permission system is session-wide, not agent-wide» ([#54898](https://github.com/anthropics/claude-code/issues/54898)) [самоотчёт]. Если остановленного напарника вернули сообщением, а папка с файлом агента не доверенная, он возвращается «with the team-essential tools and no custom instructions» ([errors](https://code.claude.com/docs/en/errors#teammate-agent-definition-not-restored)). С плагинными определениями картина противоречивая. В v2.1.288 исправили случай, когда plugin-агент по имени получал умолчания вместо своих prompt и tools ([changelog](https://code.claude.com/docs/en/changelog#2-1-288)). Но issue «определения из плагинов игнорируются» открыт ([#78234](https://github.com/anthropics/claude-code/issues/78234)) — проверить вживую.

**Для ТЗ.** Роли ТЗ держатся на трёх вещах: свой список инструментов, своя модель, свой набор MCP (ТЗ, §5). В команде остаются первые две. Третья превращается в «все видят MCP проекта, но не все могут вызвать».

### 4.8. Нет режима «лид только координирует»

**Факт.** Delegate mode в текущей документации нет. Есть только совет промптом: «Wait for your teammates to complete their tasks before proceeding» ([agent-teams](https://code.claude.com/docs/en/agent-teams#wait-for-teammates-to-finish)). Значения `delegate` нет ни в `--permission-mode` ([cli-reference](https://code.claude.com/docs/en/cli-reference#cli-flags)), ни в поле `permission_mode` хуков ([hooks](https://code.claude.com/docs/en/hooks#common-input-fields)). Раньше режим был: Shift+Tab ограничивал лида инструментами координации, но ограничения каскадировали на напарников. Issue закрыт как дубликат #23447 «delegate mode cascades to teammates» ([#25037](https://github.com/anthropics/claude-code/issues/25037)). По сниппету поиска, `delegate` виден в выборе `--permission-mode` лишь в части сборок ([#54080](https://github.com/anthropics/claude-code/issues/54080)). Типовые срывы лида: он берётся за задачи сам ([agent-teams](https://code.claude.com/docs/en/agent-teams#wait-for-teammates-to-finish)), а ещё «can stop early too, deciding the team is finished before all tasks are actually complete» ([agent-teams](https://code.claude.com/docs/en/agent-teams#agents-stopping-early)).

**Обходы (предложение, не проверено).** PreToolUse-хук запрещает `Write`, `Edit` и `Bash`, когда `agent_id` пуст — это главный поток ([hooks](https://code.claude.com/docs/en/hooks#common-input-fields)). Или мод с хуком `tool.check`: поле `e.agentId` «absent when the main conversation does» (v2.1.290+) ([mods reference](https://code.claude.com/docs/en/plugins/mods/reference#tools)). Хук мода `agent.spawn` видит и напарников (`e.isTeammate: true`) и может вернуть `{ deny: reason }` ([mods reference](https://code.claude.com/docs/en/plugins/mods/reference#subagents)). Узкий `tools` у `--agent director`, вероятно, каскадирует на in-process напарников так же, как delegate mode, — не проверено.

**Для ТЗ.** Лид, который пишет за напарников, дублирует работу и тратит токены Opus. В ТЗ директор ограничен записью в `cycles/` (ТЗ, §5), а каждая стадия проверяется по схеме (ТЗ, §6). Команда добавляет лишний риск: лид и напарники делят один список задач и могут сделать одно и то же дважды.

### 4.9. Функция экспериментальна восемь месяцев, а баги закрывает бот

**Факт.** С 5 февраля по 13 октября 2026 года статус — «experimental and disabled by default» ([agent-teams](https://code.claude.com/docs/en/agent-teams), [changelog](https://code.claude.com/docs/en/changelog#2-1-32)). Сроки выхода не объявлены. Issues много: 348 со словом «teammate» в заголовке, 359 с «agent team», 54 открытых с «agent teams» ([поиск](https://github.com/anthropics/claude-code/issues?q=is%3Aissue+in%3Atitle+teammate+sort%3Acomments-desc), [поиск](https://github.com/anthropics/claude-code/issues?q=is%3Aissue+in%3Atitle+%22agent+team%22+sort%3Acomments-desc), [поиск](https://github.com/anthropics/claude-code/issues?q=is%3Aissue+is%3Aopen+in%3Atitle+%22agent+teams%22+sort%3Areactions-%2B1-desc)). Тяжёлые баги бот закрывает как «not planned» за неактивность, без исправления: дубликаты напарников ([#55586](https://github.com/anthropics/claude-code/issues/55586), [#32996](https://github.com/anthropics/claude-code/issues/32996)), пустые ходы при старте ([#24414](https://github.com/anthropics/claude-code/issues/24414)).

**Для ТЗ.** Строить денежный цикл на функции без срока созревания, которая ломается раз в месяц, — значит каждую неделю проверять, работает ли каркас, вместо проверки идей.

### 4.10. Ещё шесть трений, которые не блокируют, но съедают время человека

| Трение | Доказательство | Следствие |
|---|---|---|
| Поток запросов разрешений | «dozens of prompts per session» при 100+ allow-правилах; `bypassPermissions` не действует на Bash напарников; `settings.local.json` не наследуется — подтверждено на 2.1.283 ([#26479](https://github.com/anthropics/claude-code/issues/26479)) [самоотчёт]. Хуки PermissionRequest на запросы напарников не срабатывают ([#23983](https://github.com/anthropics/claude-code/issues/23983)) [самоотчёт] | Человек сидит у терминала лида; автоматизировать одобрения нельзя |
| Лид «спит» до нажатия клавиши | Ответы напарников не обрабатываются до ручного ввода: «breaks unattended orchestration» ([#38932](https://github.com/anthropics/claude-code/issues/38932), [#51959](https://github.com/anthropics/claude-code/issues/51959)) — оба закрыты ботом [самоотчёт] | Прогон без присмотра встаёт |
| Повторная доставка | Сообщение или idle-уведомление приходит лиду до 4 раз ([#92244](https://github.com/anthropics/claude-code/issues/92244)); «пинг-понг» idle ([#85047](https://github.com/anthropics/claude-code/issues/85047)) [самоотчёт] | Лишние ходы лида и токены |
| Глухие напарники | События Monitor и фоновых задач не будят простаивающего напарника; будит только `SendMessage` ([#77300](https://github.com/anthropics/claude-code/issues/77300)) [самоотчёт] | Напарник не реагирует на готовность данных |
| Остановка на ошибке | «Teammates may stop after encountering errors instead of recovering»; «Task status can lag» ([agent-teams](https://code.claude.com/docs/en/agent-teams#agents-stopping-early), [agent-teams](https://code.claude.com/docs/en/agent-teams#limitations)) | Зависимые задачи блокируются |
| Нет изоляции файлов | «Agent teams don't isolate teammates in worktrees» ([agents](https://code.claude.com/docs/en/agents#choose-an-approach)) | Каждому напарнику — свой файл, иначе перезапись |

## 5. Практика: компилятор собран не функцией agent teams, а issues считают дубли и потери

### 5.1. Кейсы Anthropic: везде оркестратор и оракул, а не одноранговая команда

**C-компилятор — главный кейс с ярлыком «agent teams».** Николас Карлини 5 февраля 2026 года описал, как Opus 4.6 написал компилятор C на Rust: 16 агентов, около двух недель, почти 2 000 сессий Claude Code, «a total cost just under $20,000», 2 млрд входных и 140 млн выходных токенов, 100 000 строк кода ([Anthropic Engineering](https://www.anthropic.com/engineering/building-c-compiler)) [вендор]. Компилятор собирает загружаемый Linux 6.9 на x86, ARM и RISC-V и проходит 99% большинства тестовых наборов, включая GCC torture ([там же](https://www.anthropic.com/engineering/building-c-compiler)) [вендор]. Расчёт по этим цифрам: около $10 на сессию и около $625 на агента в неделю.

**Поправка: это не функция agent teams из Claude Code.** Слова в посте есть: «We tasked Opus 4.6 using agent teams to build a C Compiler». Но харнесс самодельный. Bash-цикл `while true` запускает `claude --dangerously-skip-permissions`. Каждый агент живёт в своём Docker-контейнере. Общий репозиторий — голый git `/upstream`. Блокировка задачи — текстовый файл в `current_tasks/`. Лида нет: «I don't use an orchestration agent». Сообщений нет: «I haven't yet implemented any other method for communication between agents» ([Anthropic Engineering](https://www.anthropic.com/engineering/building-c-compiler)). Сам харнесс не опубликован. Вторичные СМИ путают кейс с функцией, например «Anthropic Researcher Builds Compiler Using Agent Teams» ([letsdatascience](https://www.letsdatascience.com/news/anthropic-researcher-builds-compiler-using-agent-teams-3ab2ffc4), по сниппету поиска).

**Что там сработало и что сломалось.** Сработал оракул. Пока падающих тестов много, «each agent picks a different failing test to work on». На ядре Linux GCC служил заведомо правильным эталоном, и агенты чинили разные баги в разных файлах. Сломался параллелизм на монолитной задаче: «Having 16 agents running didn't help because each was stuck solving the same task»; «Every agent would hit the same bug, fix that bug, and then overwrite each other's changes». Однажды агент выполнил `pkill -9 bash` и убил собственный цикл. Про человека автор пишет: «Existing agent scaffolds like Claude Code require an operator to be online and available to work jointly» ([Anthropic Engineering](https://www.anthropic.com/engineering/building-c-compiler)). Реакция сообщества скептическая, местами насмешливая: issue «Hello world does not compile» открыт с 5 февраля, в нём 337 комментариев и нет ответа мейнтейнеров ([#1](https://github.com/anthropics/claudes-c-compiler/issues/1)). **Урок для MVP (предложение):** роль оракула в LLM-компании играют детерминированные скрипты ТЗ — `validate.py`, `max_cpc.py`, `swiss.py` (ТЗ, §7), — а не переписка агентов.

**Остальные кейсы Anthropic — про субагентов.** Исследовательская система: лид Opus 4 с субагентами Sonnet 4 обошёл одиночный Opus 4 на 90,2% во внутреннем eval. Агенты тратят около 4× токенов чата, многоагентная система — около 15×. На BrowseComp объём токенов объясняет 80% разброса качества: «Multi-agent systems work mainly because they help spend enough tokens to solve the problem» ([Anthropic Engineering](https://www.anthropic.com/engineering/multi-agent-research-system)) [вендор, внутренняя оценка]. Code Review: параллельные искатели багов плюс отдельный шаг верификации. Доля PR с содержательными комментариями выросла с 16% до 54%. Ошибочными инженеры отметили «less than 1%» находок. Цена — $15–25 за ревью, около 20 минут ([claude.com](https://claude.com/blog/code-review), [docs](https://code.claude.com/docs/en/code-review)) [вендор]. Цитаты клиентов из анонса Opus 4.6 тоже про субагентов: у NBIM Opus 4.6 дал лучший результат в 38 из 40 расследований по кибербезопасности при «up to 9 subagents and 100+ tool calls» ([Introducing Claude Opus 4.6](https://www.anthropic.com/news/claude-opus-4-6)) [вендор]. Количественных публикаций Anthropic о выигрыше agent teams над субагентами нет ([заметки t2](notes/agent-teams/t2-practice.md), §1).

**Советы Anthropic сводятся к осторожности.** Многоагентные системы «typically use 3-10x more tokens than single-agent approaches»; параллелизм даёт «thoroughness, not speed»; в ролевом эксперименте субагенты «spent more tokens on coordination than on actual work»; делить работу надо по нужному контексту, а не по типу работы; «Start with the simplest approach that works, and add complexity only when evidence supports it» ([claude.com](https://claude.com/blog/building-multi-agent-systems-when-and-how-to-use-them)) [вендор]. Документация команд: «Before you set up a team, check whether a lighter option does the job» ([agent-teams](https://code.claude.com/docs/en/agent-teams)) [вендор].

### 5.2. GitHub issues: дубли, токены, потеря команды и чужие задачи

Цифры ниже — самоотчёты авторов issues. Anthropic их не подтверждала, кроме воспроизведения #71723.

| Issue | Что случилось | Цифры [самоотчёт] | Статус на 2026-10-13 |
|---|---|---|---|
| [#55586](https://github.com/anthropics/claude-code/issues/55586) | Один вызов `Agent()` порождает 10–151 невидимый дубликат; все правят файлы и тратят полный контекст | Пик — 389 агентов одновременно. Худшая сессия — **42,9 млрд токенов** (≈$43 000 по API). За 14 недель — 90,8 млрд (≈$91 000), из них ≈$86 000 — дубли. 12 974 правки и 8 836 записей у одной роли. Воспроизведение на macOS: 30 вызовов → 107 транскриптов | Закрыт ботом 25 августа 2026 как «not planned», исправления нет |
| [#32996](https://github.com/anthropics/claude-code/issues/32996) | В in-process режиме каждый напарник создаётся дважды; поздняя копия перезаписала отчёт ранней | **2,2×** по объёму транскриптов: 1 008 КБ субагентами против 2 220 КБ командой. Позже 2,5× за 90 секунд (6 напарников) и 12× в 30-минутном конвейере | Закрыт ботом 11 мая 2026 |
| [#47930](https://github.com/anthropics/claude-code/issues/47930) | Лид зацикливается на idle-уведомлениях и эхо-назначениях | 8 напарников: **138 ходов лида** и 23,3 млн входных токенов против **30–40 ходов** той же работы фоновыми Agent. Шум — 13,0–22,2% входных токенов лида. В комментариях — около 18% потерь за неделю и около 20 пингов за ночь ≈ $8–10 | Открыт с 14 апреля 2026, метка `area:cost`, ответа нет |
| [#23620](https://github.com/anthropics/claude-code/issues/23620) | После компакции лид теряет команду и заново раздаёт идущую работу | Воспроизводится примерно за 45 минут; «the number 1 most frequent bug I hit with Agent Teams» | Открыт с 6 февраля 2026; от мейнтейнеров только метка |
| [#93668](https://github.com/anthropics/claude-code/issues/93668) | Самозахват задач не учитывает роли: ревьюер только на чтение берёт задачи реализации | Лид вручную переназначил все 14 задач | Открыт с 11 сентября 2026, ответа нет |
| [#71723](https://github.com/anthropics/claude-code/issues/71723) | `name` переводит субагента в протокол напарника, результат теряется | Около 20 минут на случай; три потерянных воркера за день | Открыт; баг подтверждён мейнтейнером 17 августа 2026 |
| [#26479](https://github.com/anthropics/claude-code/issues/26479) | Запросы разрешений напарников идут потоком | «dozens of prompts per session» при 100+ allow-правилах | Открыт с 18 февраля 2026; подтверждён на 2.1.283 |
| [#24414](https://github.com/anthropics/claude-code/issues/24414) | Напарник без задачи жжёт ходы при старте | Около 4 ходов на напарника; «N teammates = ~2N wasted lead turns» | Закрыт ботом 10 марта 2026 |
| [#92244](https://github.com/anthropics/claude-code/issues/92244) | Сообщение или уведомление приходит лиду до 4 раз с одной меткой времени | — | Открыт с 5 сентября 2026 |
| [#42005](https://github.com/anthropics/claude-code/issues/42005), [#60591](https://github.com/anthropics/claude-code/issues/60591) | Статистика напарника — «0 tool uses / 0 tokens»; общей стоимости команды нет | — | Оба закрыты ботом |

**Что из этого следует.** Сбои команд — не в качестве рассуждений, а в инфраструктуре координации: доставка сообщений, дубли, жизненный цикл напарников, компакция, разрешения. Именно они делают прогоны недетерминированными и дорогими. Статус «closed» здесь часто не значит «fixed»: бот закрывает за неактивность, и пользователи открывают «наследников» — #92244 после #74112 ([заметки t2](notes/agent-teams/t2-practice.md), §3). Дубли без проверки не видны. Проверка — считать `agent-*.meta.json` и транскрипты в `subagents/` после каждого прогона ([#55586](https://github.com/anthropics/claude-code/issues/55586)).

### 5.3. Некодовые задачи: прямых замеров нет, смежные данные — за независимых исполнителей

**Прямых данных нет.** Измерений качества agent teams на бизнес-аналитике, маркетинге или исследованиях против одиночного агента или субагентов не нашлось ни у Anthropic, ни у сообщества ([заметки t2](notes/agent-teams/t2-practice.md), §4). Документация советует начинать именно с некодовых задач — «Start with research and review… tasks that have clear boundaries and don't require writing code». Она приводит примеры: три напарника по UX, архитектуре и «devil's advocate»; пять напарников, которые опровергают гипотезы друг друга «like a scientific debate». Но это рекомендации без замеров ([agent-teams](https://code.claude.com/docs/en/agent-teams)) [вендор].

**Смежные данные Anthropic — про субагентов и workflows.** Исследовательская система на субагентах (+90,2%) и Code Review (меньше 1% ложных находок) из раздела 5.1. Встроенный workflow `/deep-research` веером ищет, перекрёстно проверяет источники и «votes on each claim» ([workflows](https://code.claude.com/docs/en/workflows)) [вендор]. То есть «перекрёстно проверенное исследование» Anthropic делает воркфлоу, а не командой.

**Независимые работы — только по сниппетам поиска.** Google Research и MIT: централизованная координация даёт +80,9% на задачах, которые параллелятся; на последовательных задачах все многоагентные варианты ухудшили результат на 39–70% ([Google Research](https://research.google/blog/towards-a-science-of-scaling-agent-systems-when-and-why-agent-systems-work/), [arXiv 2512.08296](https://arxiv.org/abs/2512.08296v2)) [независимо, по сниппету поиска]. «Stop Overvaluing Multi-Agent Debate»: на 9 бенчмарках и 4 моделях дебаты часто не превосходят Chain-of-Thought и Self-Consistency даже при большем бюджете вычислений; помогает разнородность моделей ([arXiv 2502.08788](https://arxiv.org/abs/2502.08788v3)) [независимо, по сниппету поиска]. «Should we be going MAD?»: дебаты надёжно не обходят self-consistency и ансамбли ([arXiv 2311.17371](https://arxiv.org/html/2311.17371v3)) [независимо, по сниппету поиска]. MAST: 14 режимов отказа на 1 600+ трассах; сбои идут в основном от проектирования системы и рассогласования агентов, а не от слабости моделей ([arXiv 2503.13657](https://arxiv.org/html/2503.13657v3)) [независимо, по сниппету поиска]. Cursor отказался от плоской самокоординации через общий файл и блокировки: 20 агентов давали пропускную способность 2–3; иерархия «планировщики → воркеры → судья» масштабировалась лучше ([cursor.com](https://cursor.com/blog/scaling-agents)) [вендор, по сниппету поиска]. Cognition пришла к схеме, где интеллект вносят несколько агентов, а запись остаётся однопоточной ([cognition.ai](https://cognition.ai/blog/multi-agents-working)) [вендор, по сниппету поиска].

**Бизнес-применения команд — без сравнений.** Отчёт о рынке «about eight minutes» ([growwstacks](https://growwstacks.com/blog/build-ai-agent-teams-with-claude-code-for-deep-market-research)), персоны-оппоненты для стратегии ([MindStudio](https://www.mindstudio.ai/blog/claude-code-agent-teams-brainstorming)), «Two AI agents whose only job is to kill my ideas» ([calevans.com](https://calevans.com/blog/two-ai-agents-whose-only-job-is-to-kill-my-ideas.html)) — всё по сниппетам поиска. Ни в одном нет сравнения с одиночным агентом. Русскоязычных практических отчётов на Хабре и vc.ru не нашлось ([заметки t2](notes/agent-teams/t2-practice.md), §2).

**Вывод для MVP (предложение).** Качество аналитики растёт от двух вещей: покрытия и независимой проверки. Обе есть у субагентов и воркфлоу без переписки агентов. Переписка полезна в одном сценарии — состязательном. Но доказательная база дебатов однотипных моделей слабая, а в Claude Code все напарники — Claude ([agents](https://code.claude.com/docs/en/agents)). Конвейер ТЗ последовательный — ровно тот класс задач, где в работе Google многоагентность ухудшала результат.

## 6. Стоимость: 7× у вендора, $90–190 в неделю в экстраполяции, 42,9 млрд токенов в худшем самоотчёте

**Официальная цифра одна.** «Agent teams use approximately 7x more tokens than standard sessions when teammates run in plan mode» ([costs](https://code.claude.com/docs/en/costs#manage-agent-team-costs)) [вендор]. Методики нет. Остальное — качественно. «Token costs scale linearly» ([agent-teams](https://code.claude.com/docs/en/agent-teams#choose-an-appropriate-team-size)). «Each active teammate continues consuming tokens until it exits or the session ends» ([costs](https://code.claude.com/docs/en/costs#agent-team-token-costs)). Советы документации: Sonnet для напарников, маленькая команда, короткие промпты спавна, гасить напарников после работы ([costs](https://code.claude.com/docs/en/costs#agent-team-token-costs)).

**Кэш у напарников короче, потолка нет.** Запросы in-process напарника не попадают в TTL-корзину основного разговора. Кэш живёт 5 минут, «including on a Claude subscription». Час даёт `subagentPromptCacheTtl: "1h"`, но запись часового кэша дороже ([agent-teams](https://code.claude.com/docs/en/agent-teams#token-usage), [settings-reference](https://code.claude.com/docs/en/settings-reference#subagentpromptcachettl)). Промахи кэша у напарников чинили в v2.1.261, v2.1.265, v2.1.277 и v2.1.290 ([changelog](https://code.claude.com/docs/en/changelog#2-1-265)). `--max-budget-usd` работает только в print mode ([cli-reference](https://code.claude.com/docs/en/cli-reference#cli-flags)). Встроенного лимита размера команды нет; запрос на `maxTeamSize` был в [#55586](https://github.com/anthropics/claude-code/issues/55586), но issue закрыт ботом. Значит, потолок денег у команды даёт только лимит workspace в Console (ТЗ, §3 п. 9) или лимит подписки. Укладывается ли интерактивная команда на подписке в «ordinary use» Consumer Terms — не проверено.

**Множители из всех источников.**

| Источник | Что сравнивали | Цифра | Метка |
|---|---|---|---|
| Документация Claude Code ([costs](https://code.claude.com/docs/en/costs#manage-agent-team-costs)) | Команда в plan mode против обычной сессии | ≈7× токенов | [вендор] |
| Блог Claude, январь 2026 ([claude.com](https://claude.com/blog/building-multi-agent-systems-when-and-how-to-use-them)) | Многоагентная система против одного агента | 3–10× токенов | [вендор] |
| Исследовательская система ([Anthropic Engineering](https://www.anthropic.com/engineering/multi-agent-research-system)) | Многоагентная система на субагентах против чата | ≈15× токенов | [вендор] |
| C-компилятор ([Anthropic Engineering](https://www.anthropic.com/engineering/building-c-compiler)) | 16 агентов на самодельном харнессе, две недели | <$20 000; ≈$10 на сессию; ≈$625 на агента в неделю | [вендор] + расчёт |
| [#32996](https://github.com/anthropics/claude-code/issues/32996) | Те же роли командой против субагентов | **2,2×** по объёму транскриптов; до 12× в 30-минутном конвейере | [самоотчёт] |
| mejba.me ([mejba.me](https://www.mejba.me/claude-agent-teams-guide)) | Одно приложение: один Opus против лида Opus + 2 Sonnet + 1 Haiku | **≈4×** токенов (≈45 тыс. против ≈180 тыс.) и 2,75× времени (8 против 22 минут) | [самоотчёт], по сниппету поиска |
| [#47930](https://github.com/anthropics/claude-code/issues/47930) | 8 напарников против фоновых Agent | **138 ходов лида против 30–40**; 13–22% входных токенов лида — idle-шум | [самоотчёт] |
| Комментарии к [#47930](https://github.com/anthropics/claude-code/issues/47930) | Ночная сессия, 4 агента | ≈20 одинаковых пингов ≈ $8–10 накладных | [самоотчёт] |
| [#55586](https://github.com/anthropics/claude-code/issues/55586) | Невидимые дубли напарников, 14 недель | Худшая сессия — **42,9 млрд токенов** (≈$43 000 по API); всего 90,8 млрд (≈$91 000) | [самоотчёт] |

Один блог-замер говорит об обратном: 2–3 минуты командой против 5–10 минут субагентами, без методики ([growwstacks](https://growwstacks.com/blog/claude-agent-teams-vs-sub-agents/)) [вендорский блог, по сниппету поиска]. Контролируемых сравнений «команда против субагентов» на одной задаче с повторами нет ([заметки t2](notes/agent-teams/t2-practice.md), §2).

**Что это значит для MVP.**

| Сценарий | Неделя | Месяц (×4,33) | Метка |
|---|---|---|---|
| ТЗ на субагентах | $13–27 | $59–115 | Оценка [второго отчёта](claude-code-harness.md) по ценам [pricing](https://platform.claude.com/docs/en/about-claude/pricing) |
| Весь цикл командой, верхняя граница 7× | ≈$90–190 | ≈$390–820 | **Экстраполяция** |
| Только стадия 7 командой: ≈$3 стадии × (k − 1), k = 3–7 | +$6–18 к базе | +$26–78 | Оценка |

Как читать вторую строку. Множитель 7× сравнивает команду с одиночной сессией в plan mode, а не с конвейером субагентов. Поэтому $90–190 — верхняя граница, а не прогноз. Третья строка взята из таблицы этапов второго отчёта: пакет совета на Opus 5.5 — около $3 в неделю. Множитель k = 3–7 — диапазон вендора и самоотчётов. Реальное число даст только замер (раздел 10).

**Главный риск — не 7×, а редкая катастрофа.** Дубли напарников невидимы в интерфейсе и стоили автору #55586 десятки тысяч долларов [самоотчёт]. Встроенного стоп-крана нет. **Предложение:** любой опыт с командами — только в отдельном workspace Console с жёстким месячным лимитом и с подсчётом транскриптов после каждого прогона. **Плюс:** худший случай ограничен лимитом. **Минус:** нужен отдельный ключ API, а эксперимент на подписке так не ограничить.

## 7. Где команда может помочь: состязательный совет — как эксперимент, турнир — нет

### 7.1. Состязательный совет на стадии `/board-pack`

**Почему здесь.** Стадия 7 сводит топ-3 перед гейтом 1. После неё основатель тратит деньги на лендинг и рекламу. Пакет пишет один автор — директор, и у одного автора есть риск якорения на первой версии. Документация описывает ровно этот случай: при последовательном разборе возникает якорение, а теория, пережившая атаку нескольких напарников, «much more likely to be the actual root cause» ([agent-teams](https://code.claude.com/docs/en/agent-teams)) [вендор, без замеров]. Задача только на чтение, с ясными границами и без кода — то, с чего документация советует начинать.

**Устройство (предложение).**

| Элемент | Решение | Зачем |
|---|---|---|
| Лид | Отдельное определение `board-chair` (Opus 5.5): копия директора, но `Agent(board-advocate, board-redteam, board-auditor)` и запись только в `objections-A.md` или `objections-B.md` | Основной `director` не меняется |
| Напарники | Три определения на Sonnet 5.5: **advocate** — лучший честный довод за каждую идею топ-3; **redteam** — причины провала, стоп-факторы, юридические риски, слабые допущения; **auditor** — каждая цифра прослеживается до `market.json`, `economics.json`, `ranking.json` | Три разные линзы против дублирования ([agent-teams](https://code.claude.com/docs/en/agent-teams)); документация советует Sonnet для напарников ([costs](https://code.claude.com/docs/en/costs#agent-team-token-costs)) |
| Права | Только чтение данных: `Read`, `Grep`, `Glob` и `Write` лишь в свой файл `cycles/<неделя>/board-debate/<вариант>/<имя>.json`, где вариант — `A` или `B`. Без MCP, веба и Bash. Сессия запускается без токенов в окружении | Пакету совета внешние API не нужны; in-process напарник видит MCP проекта ([agent-teams](https://code.claude.com/docs/en/agent-teams#use-subagent-definitions-for-teammates)) |
| Ход | Раунд 1 — каждый пишет позицию сам, без сообщений. Раунд 2 — читает файлы двух других, шлёт не больше двух возражений через `SendMessage`, обновляет свой файл. Лид ждёт трёх валидных файлов и пишет `objections-B.md` | Независимость первого раунда снимает якорение; лимит раундов сдерживает токены |
| Режим | Без plan mode | Plan mode — главный множитель 7×, а одобрение плана всё равно автоматическое (раздел 4.3) |
| Выход | Раздел «Возражения» рядом с `board-pack.md`; `decision.md` по-прежнему заполняет и коммитит человек | Гейт 1 не меняется |

**Условия запуска (предложение).** Не раньше конца фазы 4 ТЗ: два сухих цикла на субагентах уже дали базовую линию (ТЗ, §13). Флаг — только в отдельном профиле настроек. Task-инструменты включены. Хуки-гейты из раздела 9 протестированы на отказ. Версия Claude Code закреплена. Отдельный workspace с лимитом. Один прогон — не дольше 30 минут: это меньше окна, в котором компакция теряет команду ([#23620](https://github.com/anthropics/claude-code/issues/23620)) [самоотчёт].

**Критерии принятия** — в разделе 10. Коротко: возражения команды заметно лучше возражений субагентов, токены стадии растут не больше чем вдвое, и ни одного сбоя из раздела 5.2.

**Плюсы.** Единственная стадия, где переписка совпадает с задачей. Риск низкий: только чтение, одна стадия, гейт не меняется. Ответ «помогает ли спор» станет фактом, а не мнением. **Минусы.** Доказательства пользы дебатов однотипных моделей слабые (раздел 5.3). Стадия дорожает в 3–7 раз (оценка). Нужен человек у терминала. Экспериментальная функция может сломаться между циклами.

### 7.2. Турнир: судьи независимы, переписка между ними вредна

**Две позиции в заметках — одна в отчёте.** Заметки по механике называли локальной нишей команд «дебаты судей» ([заметки t1](notes/agent-teams/t1-mechanics.md), §6). Заметки по практике требуют независимых судей ([заметки t2](notes/agent-teams/t2-practice.md), §4, §7). **Позиция отчёта (предложение): внутри турнира переписки нет. Спор допустим только после турнира, о топ-3, на стадии пакета совета (раздел 7.1).** То, что в заметках t1 названо «разбором конкурирующих гипотез по финалистам», и есть состязательный совет.

**Почему так.** Первое — протокол ТЗ. Каждая пара судится дважды, A–B и B–A; расхождение даёт 0,5 очка обоим (ТЗ, §6 `/tournament`). Перестановки ловят смещение по позиции. В MT-Bench GPT-4 согласован с собой после перестановки в 65,0% случаев, Claude-v1 — в 23,8% ([arXiv 2306.05685](https://arxiv.org/pdf/2306.05685), по сниппету поиска). Если судьи переписываются, их ошибки коррелируют, и второй порядок перестаёт быть независимым измерением (вывод). Второе — литература. Дебаты часто не лучше Chain-of-Thought и Self-Consistency даже при большем бюджете, а помогает им разнородность моделей ([arXiv 2502.08788](https://arxiv.org/abs/2502.08788v3), по сниппету поиска). Судья же в Claude Code может быть только Claude ([agents](https://code.claude.com/docs/en/agents)). Третье — механика. Турнир — около 50 независимых судейств: 10 финалистов, 5 туров, 25 пар, два порядка ([второй отчёт](claude-code-harness.md)). Это веер без обсуждения. Команда добавит к нему самозахват задач без учёта ролей ([#93668](https://github.com/anthropics/claude-code/issues/93668)), дубли ([#55586](https://github.com/anthropics/claude-code/issues/55586)) и кратную цену.

**Как делать турнир.** Как в ТЗ: судья видит только карточки своей пары (ТЗ, §5). Параллелить — субагентами-судьями с одной рубрикой или воркфлоу `parallel()` с `agent({schema})` и seed через `args`; `Math.random` в скрипте воркфлоу запрещён ([workflows](https://code.claude.com/docs/en/workflows#what-the-saved-script-looks-like)). **Плюсы:** протокол остаётся измеримым и воспроизводимым по seed. **Минусы:** у судей нет «второго мнения» друг друга — его заменяют перестановки, ничьи и Бухгольц.

## 8. Альтернативы: параллельные субагенты и воркфлоу закрывают всё, что обещает команда

Anthropic перечисляет пять способов параллельной работы: субагенты, agent view, agent teams, Projects и dynamic workflows. «In every approach the workers are Claude sessions» ([agents](https://code.claude.com/docs/en/agents)). Рядом — cross-session messaging, `/goal` и advisor. Вердикты в последних двух колонках — предложение.

| Функция | Статус на 2026-10-13 | Годится для недели с гейтами? | Где в MVP | Плюсы / минусы |
|---|---|---|---|---|
| **Параллельные субагенты** | Без ярлыка preview; до 20 одновременно ([sub-agents](https://code.claude.com/docs/en/sub-agents#concurrent-subagent-limit)); работают в `-p` | **Да — основа ТЗ** | Все стадии; веер скаута по сегментам; судьи турнира; оппоненты в пакете совета | + Дёшево: результат сжимается в контекст вызывающего ([workflows](https://code.claude.com/docs/en/workflows#when-to-use-a-workflow)). − Не видят историю диалога; нет переписки друг с другом |
| **Dynamic workflows** | Запуск в v2.1.154 от 2026-05-28 ([changelog](https://code.claude.com/docs/en/changelog#2-1-154)); ярлыка preview на странице нет; все платные планы, API, Bedrock, Vertex, Foundry; на Pro включается в `/config` ([workflows](https://code.claude.com/docs/en/workflows)) | **Да:** одна стадия — один воркфлоу. «No mid-run user input … For sign-off between stages, run each stage as its own workflow» ([workflows](https://code.claude.com/docs/en/workflows#behavior-and-limits)) | Скаут (`pipeline()` по сегментам), турнир (`parallel()` судей), данные по идеям с учётом квот | + План держит скрипт: «Who decides what runs next: The script»; 16 агентов одновременно, до 1 000 на прогон; возобновление в той же сессии, через фон или `claude --resume` ([workflows](https://code.claude.com/docs/en/workflows#resume-after-a-pause)); работает в `-p` и SDK. − Формат только Claude Code; отчётов практиков о надёжности нет |
| **Agent view** (`claude agents`) | Research preview с v2.1.139 от 2026-05-11 ([agent-view](https://code.claude.com/docs/en/agent-view), [changelog](https://code.claude.com/docs/en/changelog#2-1-139)) | **Да, как пульт стадийных сессий** на своей машине | Вторник: `/landing` и `/smoke-plan` в фоне; ежедневный `/monitor` | + Фоновая сессия — «a full Claude Code conversation that keeps running without a terminal attached»; переживает сон машины и рестарт supervisor ([agent-view](https://code.claude.com/docs/en/agent-view)). − Research preview; «stop if the machine shuts down»; десять параллельных агентов тратят квоту примерно в десять раз быстрее ([agent-view](https://code.claude.com/docs/en/agent-view#limitations)); запрос разрешения ждёт в состоянии «Needs input», пока человек не подключится |
| **Cross-session messaging** | `SendMessage` и `ListAgents` между своими сессиями: v2.1.224+ на macOS и Linux, v2.1.234+ на Windows; `-p`-воркер принимает сообщения при `crossSessionInbound: accept` ([cross-session-messaging](https://code.claude.com/docs/en/cross-session-messaging)); с API-ключом — только на одной машине ([feature-availability](https://code.claude.com/docs/en/feature-availability)) | Частично; в MVP не нужен | Передать находку между стадийными сессиями без флага команд | + Работает без agent teams. − В ТЗ обмен уже идёт через файлы; сообщения — второй канал, которого нет в git |
| **Projects** | «Public beta on Pro and Max plans and rolling out gradually»; нет на Team и Enterprise; нет в терминальном CLI, VS Code, JetBrains, Bedrock, Vertex, Foundry; нужен код на github.com и Claude GitHub App ([claude-projects](https://code.claude.com/docs/en/claude-projects#limitations), [claude-projects](https://code.claude.com/docs/en/claude-projects#check-the-prerequisites)) | Частично: единственная функция для работы на дни и недели при выключенной машине, но гейты держатся на инструкциях | Оболочка недели, если стадии остаются файлами в git и скриптами | + Облако; треды на недели. − Бета; предпочтения — «instructions Claude keeps to, not enforced settings» ([claude-projects](https://code.claude.com/docs/en/claude-projects#tune-how-claude-runs-a-project)); одобрение в чате проекта до треда не доходит ([claude-projects](https://code.claude.com/docs/en/claude-projects#unblock-a-thread-waiting-on-approval)); 200 новых тредов в день ([claude-projects](https://code.claude.com/docs/en/claude-projects#what-draws-on-your-plan)); доступ к API Яндекса и ЮKassa из облака — не проверено |
| **`/goal`** | С v2.1.139 ([changelog](https://code.claude.com/docs/en/changelog#2-1-139)); работает в `-p` ([goal](https://code.claude.com/docs/en/goal#run-non-interactively)) | Вспомогательно, внутри стадии | «Продолжай, пока артефакт не пройдёт схему» | + Просто. − Это «a wrapper around a session-scoped prompt-based Stop hook», а оценщик «doesn't run commands or read files independently» ([goal](https://code.claude.com/docs/en/goal#how-evaluation-works)) — суждение модели, не гейт. Детерминированная замена — command-хук Stop с `validate.py` |
| **Advisor** | «Experimental and requires the Anthropic API»; нет на Bedrock, Claude Platform on AWS, Vertex, Foundry; доступен на подписке и по API ([advisor](https://code.claude.com/docs/en/advisor)) | Вспомогательно | Директор или судья на Sonnet с Opus-советником в ключевых точках | + Качество решений при меньшей цене (предложение). − Experimental; это не оркестрация, на выбор между командой и субагентами не влияет |
| **Agent teams** (для сравнения) | «Experimental and disabled by default» ([agent-teams](https://code.claude.com/docs/en/agent-teams)) | **Нет как основа** | Только эксперимент на стадии 7 | См. разделы 4–7 |

**Раскладка по многоагентным частям MVP (предложение).** Скаут — параллельные субагенты или воркфлоу по непересекающимся сегментам; объединение и дедупликация — `registry.py`. Турнир — независимые судьи субагентами или `parallel()`. Пакет совета — штатно оппоненты-субагенты, командой — только эксперимент. Вторник и мониторинг — сессии, которые запускает человек, при желании в agent view. Запуск без человека — `claude -p` с субагентами или воркфлоу; это уже зона гибридного варианта (ТЗ, §18). **Плюсы:** каждая часть на функции, которая работает в `-p` или переживает рестарт. **Минусы:** воркфлоу привязывают стадию к формату Claude Code; при переезде в другой harness их переписывают ([второй отчёт](claude-code-harness.md)).

## 9. Правки ТЗ: флаг в ноль, субагенты без `name`, эксперимент — отдельным профилем

### 9.1. Четырнадцать правок по разделам

Все правки — предложение. Текст в кавычках можно вставлять как есть. Детали эксперимента живут в этом отчёте, в ТЗ — только ссылка: так каждый факт остаётся в одном документе.

| № | Раздел ТЗ | Правка | Плюсы | Минусы |
|---|---|---|---|---|
| 1 | §0 «Коротко» | Новый пункт: «**Без agent teams.** Роли — субагенты, стадии — скиллы. Agent teams экспериментальны, не работают в `-p` и не переживают `/resume`; разбор — [agent-teams.md](agent-teams.md)» | Решение зафиксировано; агент-разработчик не «улучшит» архитектуру командой | Нет |
| 2 | §1.2 «Что не входит» | «Agent teams (флаг `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS`) — кроме опционального эксперимента на стадии 7» | Ясная граница рамки MVP | Нет |
| 3 | §4 «Репозиторий» | Опциональные строки: `.claude/experiments/board-debate-A.settings.json` и `board-debate-B.settings.json` — профили вариантов с одинаковыми хуками и запретами, флаг `0` и `1`; `.claude/agents/board-chair.md`, `board-advocate.md`, `board-redteam.md`, `board-auditor.md`; `.claude/hooks/task-gate.sh`, `idle-gate.sh`, `write-guard.sh`; `schemas/board_debate.schema.json`; `experiments/board-debate/` — оценки и метрики | Видно, что эксперимент изолирован от основного профиля | Больше файлов в репозитории |
| 4 | §5 «Роли» | Новый пункт: «Стадийных субагентов запускать по типу роли, без параметра `name`. При включённом флаге команд именованный субагент становится напарником, и его результат теряется ([#71723](https://github.com/anthropics/claude-code/issues/71723))» | Страховка на случай случайного флага; это обход, который предложил мейнтейнер | Если позже понадобится адресовать субагентов по имени, правило придётся снять |
| 5 | §6, общие правила и `/cycle` | Шаг 0 в `/cycle`: «`scripts/preflight.sh` завершает цикл с ошибкой, если `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS` равен `1`» | Детерминированная проверка до трат | Ещё один скрипт; видит ли Bash значение из `env` настроек — сверить (раздел 11) |
| 6 | §6 `/tournament` | «Судьи независимы: каждая пара и каждый порядок — отдельный вызов судьи только с карточками этой пары. Судьи не видят чужих вердиктов и не переписываются; agent teams не используются. Параллелить — субагентами или воркфлоу `parallel()`» | Фиксирует принцип, на котором стоят перестановки | Нет |
| 7 | §6 `/board-pack` | «Опционально, после фазы 4: раздел „Возражения“ от трёх ролей — „за“, „против“, „аудитор цифр“. Штатно — тремя параллельными субагентами Sonnet 5.5, только чтение. Вариант на agent teams — только A/B-эксперимент по разделу 10 [agent-teams.md](agent-teams.md)» | Проверка топ-3 до гейта 1, без зависимости от экспериментальной функции | Стадия дорожает; нужен шаблон раздела |
| 8 | §9.2 «Права» | В `.claude/settings.json`: `"env": {"CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "0"}`. Если эксперимент на этой машине не планируется — тот же ключ в managed settings (`/etc/claude-code/managed-settings.json`; действие `env` оттуда — сверить вживую). Плюс требования к профилю эксперимента из раздела 9.2 этого отчёта | Флаг не включится случайно из shell или user settings ([agent-teams](https://code.claude.com/docs/en/agent-teams#claude-spawns-teammates-instead-of-subagents)) | Проектный `0` перекрывается local, `--settings` и managed; managed-вариант закрывает и эксперимент |
| 9 | §13 «Фаза 4» | Опциональный пункт: «[ ] A/B состязательного совета на трёх циклах по разделу 10 [agent-teams.md](agent-teams.md); приёмка — критерии оттуда». Подготовка — в фазе 4, прогоны — параллельно с обычными неделями | Эксперимент встаёт после базовой линии | Плюс 1–2 часа человека в неделю на три недели (оценка) |
| 10 | §15 «Стоимость» | «Agent teams — около 7× токенов в plan mode [вендор]. Эксперимент на стадии 7 — оценка +$6–18 в неделю; только в отдельном workspace с лимитом» | Цена эксперимента видна заранее | Нет |
| 11 | §16 «Риски» | Две строки: «Флаг agent teams включён случайно → именованные субагенты становятся напарниками, результаты теряются (#71723) — мера: `0` в проекте, стадии без `name`, preflight» и «Переписка судей → коррелированные ошибки — мера: судьи изолированы, команды в турнире запрещены» | Риски в общем реестре | Нет |
| 12 | §17 «Открытые вопросы» | Пункт 9: «Agent teams на закреплённой версии: `0` в проекте выключает команды; профиль `--settings` включает их только в сессии эксперимента; `CLAUDE_CODE_ENABLE_TODO_TOOLS` через `env` или только export; TaskCompleted и TeammateIdle блокируют при сбое обёртки; какие поля роли (`maxTurns`, `effort`, `permissionMode`) действуют на напарника» | Проверка до первого прогона | Около полудня на проверки (оценка) |
| 13 | §19, стартовый промпт, блок «Правила» | «Agent teams не включай. Субагентов стадий запускай без параметра `name`» | Агент-разработчик не сломает основу | Нет |
| 14 | §20 «Источники» | Добавить [agent-teams](https://code.claude.com/docs/en/agent-teams), [#71723](https://github.com/anthropics/claude-code/issues/71723) и [этот отчёт](agent-teams.md) | Ссылки рядом с ТЗ | Нет |

### 9.2. Требования к профилю эксперимента «состязательный совет»

Профили вариантов — отдельные файлы настроек и отдельные сессии. Основной `/cycle` их не видит. Всё ниже — предложение по фактам разделов 4 и 5.

| № | Требование | Как | Зачем |
|---|---|---|---|
| 1 | Отдельные профили и отдельные сессии | B: `CLAUDE_CODE_ENABLE_TODO_TOOLS=1 claude --agent board-chair --settings .claude/experiments/board-debate-B.settings.json`, в профиле `env.CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS = "1"`. A: тот же запуск с `board-debate-A.settings.json`, где флаг `"0"`. Хуки и запреты у профилей одинаковые | Флаг перечитывается при каждом спавне ([agent-teams](https://code.claude.com/docs/en/agent-teams#claude-spawns-teammates-instead-of-subagents)); основной профиль остаётся в `0` |
| 2 | Включить Task-инструменты | Export `CLAUDE_CODE_ENABLE_TODO_TOOLS=1` до запуска или `--allowedTools` с `TaskCreate`, `TaskGet`, `TaskList`, `TaskUpdate` | Без них на моделях 5.5 нет общего списка задач и не срабатывает TaskCreated ([tools-reference](https://code.claude.com/docs/en/tools-reference#task-tool-availability)) |
| 3 | Обёртка TaskCompleted | Любой исход, кроме явного «пропустить», — exit 2; непустой stdout; «No such file or directory» в stderr заменяется; путь артефакта — в теме задачи (эскиз ниже) | `onFailure: "block"` на TaskCompleted не действует; ловушка ENOENT ([hooks](https://code.claude.com/docs/en/hooks#block-the-action-when-a-hook-fails), [hooks](https://code.claude.com/docs/en/hooks#exit-code-output)) |
| 4 | TeammateIdle как страховка | Та же обёртка; проверка `cycles/<ISO-неделя>/board-debate/B/<teammate_name>.json` по схеме, неделя — `date +%G-W%V`; exit 2, пока артефакт не валиден; после трёх отказов — `{"continue": false, "stopReason": "…"}` | Работает и без Task-инструментов; ловит напарника, который не закрыл задачу ([hooks](https://code.claude.com/docs/en/hooks#teammateidle-decision-control)); свой счётчик нужен, потому что лимит повторов описан только для Stop |
| 5 | Повторная валидация на потребляющей стадии | Лид прогоняет `validate.py` по трём файлам до сводки; PreToolUse-хук `write-guard.sh` с `onFailure: "block"` не даёт записать `objections-A.md` или `objections-B.md`, пока три файла этого варианта не валидны | Статус задач «can lag» ([agent-teams](https://code.claude.com/docs/en/agent-teams#limitations)); хуки команды при сбое пропускают |
| 6 | Только чтение | Напарникам `tools: Read, Grep, Glob, Write`; `write-guard.sh` пускает запись только в `cycles/<неделя>/board-debate/<A или B>/<agent_type>.json`; в профиле `deny` на MCP, `WebSearch`, `WebFetch`; сессия без токенов в окружении | In-process напарник грузит MCP проекта и игнорирует `mcpServers` роли ([agent-teams](https://code.claude.com/docs/en/agent-teams#use-subagent-definitions-for-teammates)); права общие для сессии ([#54898](https://github.com/anthropics/claude-code/issues/54898)) |
| 7 | Лид только координирует | `write-guard.sh` по пустому `agent_id` пускает лида только в `objections-A.md` и `objections-B.md` ([hooks](https://code.claude.com/docs/en/hooks#common-input-fields)); в промпте — «Wait for your teammates to complete their tasks before proceeding» | Delegate mode нет ([agent-teams](https://code.claude.com/docs/en/agent-teams#wait-for-teammates-to-finish)) |
| 8 | Лимит времени | Прогон не дольше 30 минут; по истечении — Esc и `x` на лиде, прогон не засчитывается | Компакция теряет команду ([#23620](https://github.com/anthropics/claude-code/issues/23620)); Esc разрывает idle-петли ([#47930](https://github.com/anthropics/claude-code/issues/47930)) |
| 9 | Лимит токенов | Отдельный workspace Console с месячным лимитом [совет], например $100 на три недели (оценка); plan mode не включать; гасить напарников после ответа | `--max-budget-usd` только в `-p` ([cli-reference](https://code.claude.com/docs/en/cli-reference#cli-flags)); 7× в plan mode ([costs](https://code.claude.com/docs/en/costs#manage-agent-team-costs)) |
| 10 | Явные задачи, спавн по одному, закреплённая версия | У каждой задачи `owner`; напарники порождаются по одному, имя = тип агента; после прогона — подсчёт транскриптов; Claude Code не ниже v2.1.295, перепроверка после обновлений | Самозахват без ролей ([#93668](https://github.com/anthropics/claude-code/issues/93668)); дубли при пачке спавнов ([#32996](https://github.com/anthropics/claude-code/issues/32996)); в v2.1.295 исправлено падение первого `SendMessage` ([changelog](https://code.claude.com/docs/en/changelog#2-1-295)) |

**Эскиз обёртки TaskCompleted** (предложение). Логику кодов выхода я прогнал в оболочке на одиннадцати случаях: задача лида, валидный артефакт, нет пути, путь чужого напарника, путь варианта A, нет файла, схема не пройдена, битый JSON на входе, нет `CLAUDE_PROJECT_DIR`, нет `jq`, нет `python3`. Все сбои дали код 2 и непустой stdout, а строка «No such file or directory» из stderr исчезла. Внутри Claude Code хук не тестировался.

```bash
#!/bin/bash
# .claude/hooks/task-gate.sh — хук TaskCompleted для эксперимента «состязательный совет».
# Любой исход, кроме явного «пропустить», превращается в exit 2 (блок).
set -uo pipefail
trap 'rc=$?; if [ "$rc" -ne 0 ]; then echo "gate: blocked"; echo "гейт: внутренняя ошибка $rc" >&2; exit 2; fi' EXIT
fail() {
  echo "gate: blocked"                                  # непустой stdout обязателен
  printf '%s\n' "$1" | sed 's/No such file or directory/file missing/g' | head -c 4000 >&2
  trap - EXIT; exit 2
}
in=$(cat)
who=$(jq -r '.teammate_name // ""' <<<"$in") || fail "нет jq или битый JSON на входе"
[ -z "$who" ] && exit 0                                 # задачи лида не гейтятся
text=$(jq -r '(.task_subject // "") + " " + (.task_description // "")' <<<"$in") || fail "битый JSON"
art=$(grep -oE "cycles/[0-9]{4}-W[0-9]{2}/board-debate/B/${who}\.json" <<<"$text" | head -n1)
[ -n "$art" ] || fail "в задаче нет пути к артефакту напарника $who"
cd "$CLAUDE_PROJECT_DIR" || fail "нет каталога проекта"
[ -f "$art" ] || fail "артефакт $art отсутствует"
out=$(python3 scripts/validate.py "$art" 2>&1) || fail "схема не пройдена: $out"
exit 0
```

Ловушка `EXIT` ловит и то, что легко пропустить: неизвестную переменную, отсутствие `jq` или `python3` (код 127). Остаётся один класс сбоев, который обёртка не закроет: хук, который вообще не запустился, или таймаут. Тогда решения нет ([hooks](https://code.claude.com/docs/en/hooks#timeouts)). Поэтому нужны пункты 4 и 5. **Плюсы профиля:** эксперимент не задевает основной цикл, худший случай ограничен временем и деньгами. **Минусы:** три хука и четыре определения ролей ради одной стадии; часть поведения не проверена (раздел 11).

## 10. Протокол эксперимента: три недели, слепая оценка, одна переменная — переписка

**Главная идея (предложение).** Сравнивать команду надо не с текущим пакетом совета, а с тем же советом на субагентах. Те же три роли, те же модели, те же два раунда, тот же вход. Единственная разница — переписка напарников. Тогда эксперимент отвечает на вопрос основателя честно: даёт ли agent teams что-то сверх субагентов. Базовый пакет директора остаётся третьей точкой отсчёта: если оба совета ничего не добавляют к нему, совет не нужен вовсе.

| Шаг | Что делать | Результат |
|---|---|---|
| 0 | Предусловия: закончена фаза 4 ТЗ, два сухих цикла на субагентах; версия Claude Code закреплена; отдельный workspace с лимитом | Базовая линия и потолок денег |
| 1 | Подготовка по промпту ниже: два профиля, четыре роли, три хука, схема, скрипты | PR в репозитории компании |
| 2 | Проверки 1–9 из раздела 11 | `experiments/board-debate/checks.md`; без зелёных проверок прогоны не начинать |
| 3 | Каждую неделю: обычный `/cycle` с флагом `0` → артефакты 2–6 и `board-pack.md` закоммичены; записать хеш коммита | Одинаковый вход для A и B |
| 4 | Вариант A: `claude --agent board-chair --settings .claude/experiments/board-debate-A.settings.json` (флаг `0`); `board-chair` порождает трёх субагентов; раунд 1 — позиции, раунд 2 — новые субагенты читают файлы двух других и пишут возражения | `objections-A.md`; токены; время |
| 5 | Вариант B: `CLAUDE_CODE_ENABLE_TODO_TOOLS=1 claude --agent board-chair --settings .claude/experiments/board-debate-B.settings.json`; задачи с `owner`; спавн по одному; не дольше 30 минут | `objections-B.md`; токены; время; транскрипты |
| 6 | Порядок A и B чередовать по неделям | Снимает влияние порядка запуска |
| 7 | Слепая оценка: `ab_board.py` перемешивает файлы; основатель ставит оценки до вскрытия ключа | `rating.md` |
| 8 | После трёх недель — решение по критериям ниже | Запись в `experiments/board-debate/decision.md` и правка ТЗ |

**Метрики сравнения.**

| Метрика | Как мерить | Порог «оставить B» |
|---|---|---|
| Существенные возражения | Возражения, которые меняют GO/NO-GO, цифры пакета или бюджет ступени 1; отмечает основатель | B не хуже A в двух неделях из трёх, и хотя бы одно существенное возражение за три недели есть только у B |
| Ложные тревоги | Возражения, которые опровергают артефакты стадий 2–6 | У B не больше, чем у A |
| Прослеживаемость | Каждое утверждение ссылается на артефакт; файлы проходят `validate.py` | 100% у обоих, иначе прогон не засчитан |
| Оценка по рубрике 1–5 | Полнота рисков, сила возражений, польза для решения GO | Средняя у B выше, чем у A, минимум на 1 балл |
| Токены | Usage отдельного workspace: показания до и после каждого прогона, A и B — в разное время; OpenTelemetry (ТЗ, §13, фаза 4). Статистике напарника в интерфейсе не верить: «0 tokens» ([#42005](https://github.com/anthropics/claude-code/issues/42005)) | B не дороже A вдвое |
| Время | Длительность прогона; минуты человека у терминала | Прогон B не дольше 30 минут; человек тратит на B не больше чем на 15 минут дольше, чем на A |
| Сбои | Дубли (подсчёт транскриптов), потеря команды, потерянный результат, чужая задача, запись вне разрешённых путей, прорыв гейта, число запросов разрешений | Ноль критических сбоев; не больше 5 запросов разрешений за прогон |

Пороги — предложение. Их, как и лимит workspace, утверждает основатель до запуска [совет].

**Решение после трёх недель.** *Оставить B*, если выполнены все пороги. Тогда в ТЗ, §6 `/board-pack`, вариант на командах становится опцией с профилем из раздела 9.2, а проверки раздела 11 повторяются после каждого обновления Claude Code. *Выкинуть B*, если не выполнен любой порог. Профиль B и хуки команд удаляются, флаг остаётся `0`. *Отдельно про A:* если совет на субагентах находит существенные возражения, которых нет в базовом пакете, — он входит в `/board-pack` штатно, без команд. **Ограничение:** три недели — маленькая выборка. Решение получится качественным, а не статистическим. Если пакеты оценивают двое, шум меньше.

**Плюсы протокола.** Одна переменная, слепая оценка, жёсткие потолки времени и денег, решение записано заранее. **Минусы.** Три недели по 1–2 часа человека (оценка). A тоже стоит денег. Результат на одной стадии не переносится на весь цикл.

### Промпт для подготовки эксперимента

Отправить в Claude Code в репозитории компании. Этот отчёт должен лежать как `docs/research/agent-teams.md`, ТЗ — как `docs/mvp-spec.md` (ТЗ, §19).

```markdown
# Задача
Подготовь опциональный A/B-эксперимент «состязательный совет» для стадии `/board-pack`.
Основание — разделы 7.1, 9.2 и 10 отчёта `docs/research/agent-teams.md` и ТЗ `docs/mvp-spec.md`.
Основной конвейер `/cycle` и роль `director` не меняй.

# Дизайн
- Три роли на Sonnet 5.5: `board-advocate` — лучший честный довод за каждую идею топ-3;
  `board-redteam` — причины провала, стоп-факторы, юридические риски, слабые допущения;
  `board-auditor` — каждая цифра пакета прослеживается до `market.json`, `economics.json`, `ranking.json`.
- Лид — `board-chair` на Opus 5.5: копия `director`, но порождает только эти три роли
  и пишет только `objections-A.md` или `objections-B.md`.
- Вариант A: флаг команд выключен; три параллельных субагента; раунд 1 — позиции;
  раунд 2 — новые субагенты читают файлы двух других и пишут возражения.
- Вариант B: тот же лид и те же роли как напарники agent teams;
  раунд 2 — не больше двух адресных возражений через `SendMessage`.
- Единственная разница A и B — переписка. Вход у обоих — один коммит с артефактами стадий 2–6.

# Что сделать
1. `.claude/settings.json`: добавь `"env": {"CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "0"}`. Остальное не трогай.
2. Два профиля: `.claude/experiments/board-debate-A.settings.json` и `board-debate-B.settings.json`.
   Одинаковые хуки из пункта 5 и deny на все MCP-инструменты, `WebSearch` и `WebFetch`.
   Отличие одно: `env.CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS` — `"0"` в A и `"1"` в B.
3. Четыре определения в `.claude/agents/`: `board-chair`, `board-advocate`, `board-redteam`, `board-auditor`.
   У напарников `tools: Read, Grep, Glob, Write` — без MCP и Bash. В теле — роль, формат выхода
   по схеме и правило «каждое утверждение ссылается на артефакт».
4. `schemas/board_debate.schema.json` (позиция по каждой идее топ-3, возражения, ссылки на артефакты)
   и тест для `scripts/validate.py`.
5. Хуки в `.claude/hooks/`:
   - `task-gate.sh` (TaskCompleted) — по эскизу из раздела 9.2 отчёта;
   - `idle-gate.sh` (TeammateIdle) — та же обёртка: проверка
     `cycles/<ISO-неделя>/board-debate/B/<teammate_name>.json`, exit 2, пока файл не валиден;
     после трёх отказов — `{"continue": false, "stopReason": "..."}`;
   - `write-guard.sh` (PreToolUse на Write и Edit, `onFailure: "block"`) — субагент или напарник
     пишет только `cycles/<неделя>/board-debate/<A или B>/<agent_type>.json`; лид (пустой `agent_id`) —
     только `objections-A.md` или `objections-B.md` и только когда три файла этого варианта
     проходят `validate.py`.
6. Тесты хуков на отказ: нет файла, битый JSON, нет `jq`, нет `python3`, нет `CLAUDE_PROJECT_DIR`.
   В каждом случае — exit 2 и непустой stdout.
7. `scripts/ab_board.py`: копирует `objections-A.md` и `objections-B.md` в `blind/objections-1.md`
   и `blind/objections-2.md` в случайном порядке; ключ — в `blind/key.json`.
8. `scripts/count_transcripts.sh`: число транскриптов субагентов и напарников за прогон — для поиска дублей.
9. `scripts/preflight.sh` для `/cycle`: ошибка, если `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS` равен `1`.
10. `experiments/board-debate/README.md`: команды запуска A и B, шаблон `rating.md`,
    таблица метрик и пороги из раздела 10 отчёта.

# Правила
- Не запускай эксперимент сам. Не включай команды в основном профиле. Plan mode не используй.
- Напарникам — никаких MCP, веб-инструментов, Bash и токенов. Денежных действий нет.
- Если документация Claude Code на установленной версии расходится с отчётом — остановись и опиши расхождение.
- Одна ветка, один PR. В конце — список ручных проверок из раздела 11 отчёта.
```

### Промпт лида для варианта B

Запуск: `CLAUDE_CODE_ENABLE_TODO_TOOLS=1 claude --agent board-chair --settings .claude/experiments/board-debate-B.settings.json`. Для варианта A — профиль `board-debate-A.settings.json` и тот же промпт с папкой `A` и файлом `objections-A.md`: шаги 2–4 выполняют субагенты, а раунд 2 — новые субагенты, которые читают файлы двух других. Шаг 1 с задачами в A не нужен.

```markdown
Неделя: <2026-WNN>. Вход — коммит <hash>: артефакты стадий 2–6 и board-pack.md в cycles/<неделя>/.
1. Создай три задачи, по одной на напарника, с owner и путём артефакта в теме:
   «позиция → cycles/<неделя>/board-debate/B/<имя>.json».
2. Породи напарников по одному: board-advocate, board-redteam, board-auditor. Имя = тип агента.
3. Раунд 1: каждый пишет свой файл, не переписываясь.
4. Раунд 2: каждый читает два других файла, шлёт адресно не больше двух возражений и обновляет свой файл.
5. Wait for your teammates to complete their tasks before proceeding. Сам задачи напарников не делай.
6. Когда три файла проходят validate.py, напиши objections-B.md: возражения по каждой идее топ-3
   со ссылками на артефакты. Затем останови напарников.
7. Если прошло 30 минут — остановись и сообщи, на каком шаге.
```

## 11. Риски и проверки вживую

### Риски

| Риск | Что известно | Мера (предложение) |
|---|---|---|
| Флаг команд включён случайно | Именованный субагент становится напарником, результат теряется ([#71723](https://github.com/anthropics/claude-code/issues/71723)); проектный `0` перекрывается local, `--settings` и managed ([agent-teams](https://code.claude.com/docs/en/agent-teams#claude-spawns-teammates-instead-of-subagents)) | `0` в `.claude/settings.json`; стадии без `name`; `preflight.sh` в `/cycle` |
| Команда теряет себя | `/resume` и `/rewind` не восстанавливают напарников ([agent-teams](https://code.claude.com/docs/en/agent-teams#limitations)); компакция ([#23620](https://github.com/anthropics/claude-code/issues/23620)) [самоотчёт] | Прогон не дольше 30 минут; состояние только в файлах; при потере — прогон заново |
| Невидимые дубли и взрыв расхода | До 151 дубликата на вызов, 42,9 млрд токенов в худшей сессии ([#55586](https://github.com/anthropics/claude-code/issues/55586)) [самоотчёт] | Спавн по одному; подсчёт транскриптов; отдельный workspace с лимитом |
| Хуки пропускают при сбое | `onFailure: "block"` не действует на TaskCompleted и TeammateIdle ([hooks](https://code.claude.com/docs/en/hooks#block-the-action-when-a-hook-fails)) | Обёртка с кодом 2; TeammateIdle-страховка; повторная валидация у лида |
| Ложное чувство гейта | План напарника одобряется без лида и человека ([agent-teams](https://code.claude.com/docs/en/agent-teams#have-teammates-plan-before-implementing)) | Гейт 1 — только `decision.md` + `gate_check.py`; plan mode не использовать |
| Роль дотягивается до MCP | In-process напарник грузит MCP проекта ([agent-teams](https://code.claude.com/docs/en/agent-teams#use-subagent-definitions-for-teammates)) | `tools` без MCP; deny в профиле; сессия без токенов |
| Лид делает работу сам или бросает рано | Delegate mode нет; «The lead can stop early too» ([agent-teams](https://code.claude.com/docs/en/agent-teams#agents-stopping-early)) | `write-guard.sh` по пустому `agent_id`; запись итога только после трёх валидных файлов |
| Человек нужен у терминала | Поток запросов ([#26479](https://github.com/anthropics/claude-code/issues/26479)); лид ждёт нажатия клавиши ([#51959](https://github.com/anthropics/claude-code/issues/51959)) [самоотчёт] | Заранее разрешить чтение; прогон только при человеке |
| Платформа ломает поведение | Шесть ломающих изменений с версией за восемь месяцев (раздел 2) | Закрепить версию; повторять проверки ниже после обновлений |
| Спор не помогает | Дебаты однотипных моделей часто не лучше self-consistency ([arXiv 2502.08788](https://arxiv.org/abs/2502.08788v3), по сниппету поиска) | A/B со слепой оценкой; выкинуть B без эффекта |

### Что проверить вживую

Порядок — от дешёвого к дорогому. Проверки 1 и 8 нужны и без эксперимента: они защищают основной цикл.

| № | Что проверить | Почему | Как |
|---|---|---|---|
| 1 | Флаг `0` в `.claude/settings.json` выключает команды, а именованный субагент возвращает результат | Риск #71723 для директора | Сухой `/cycle` на тестовой неделе: в `~/.claude/teams/` нет каталога сессии ([agent-teams](https://code.claude.com/docs/en/agent-teams#enable-agent-teams)); директор получает ответ каждой стадии |
| 2 | Профиль `--settings` включает команды поверх проектного `0`, и только в своей сессии | Порядок приоритета источников настроек в документе команд описан не полностью | Две сессии рядом: в одной напарник порождается, в другой — нет |
| 3 | `CLAUDE_CODE_ENABLE_TODO_TOOLS=1` даёт Task-инструменты in-process напарникам на Sonnet 5.5; работает ли через `env` | Документация говорит только про export ([tools-reference](https://code.claude.com/docs/en/tools-reference#task-tool-availability)) | Ctrl+T показывает задачи; TaskCreated пишет строку в журнал |
| 4 | TaskCompleted блокирует при сбое обёртки внутри Claude Code | Fail-open и ловушка ENOENT ([hooks](https://code.claude.com/docs/en/hooks#exit-code-output)) | Нет файла, битый JSON, нет `python3`, таймаут — задача не закрывается |
| 5 | TeammateIdle-страховка и счётчик | Лимит повторов описан только для Stop ([env-vars](https://code.claude.com/docs/en/env-vars)) | Напарник без артефакта не уходит в простой; после трёх отказов останавливается |
| 6 | Какие поля роли действуют на напарника: `maxTurns`, `permissionMode`, `hooks`, `effort` | Документация молчит; `effort` — [#80569](https://github.com/anthropics/claude-code/issues/80569) | Роли с намеренно жёсткими значениями, например `maxTurns: 2` |
| 7 | Напарник без MCP в `tools` не вызывает MCP ЮKassa | In-process грузит MCP проекта | Тестовый токен ЮKassa; вызов должен быть отклонён |
| 8 | Видит ли `preflight.sh` значение флага из `env` настроек | Иначе проверка в `/cycle` бесполезна | Запуск `/cycle` с флагом `1` в local settings — цикл должен остановиться |
| 9 | `write-guard.sh` по пустому `agent_id` ограничивает только лида | Delegate mode нет ([agent-teams](https://code.claude.com/docs/en/agent-teams#wait-for-teammates-to-finish)) | Лид пишет файл напарника — отказ; напарник пишет свой — успех |
| 10 | Дубли напарников | [#55586](https://github.com/anthropics/claude-code/issues/55586), [#32996](https://github.com/anthropics/claude-code/issues/32996) | После прогона число транскриптов в `subagents/` и `agent-*.meta.json` равно трём |
| 11 | Когда компакция ломает команду на материалах пакета совета | [#23620](https://github.com/anthropics/claude-code/issues/23620) | Длинный прогон на копии недели; засечь время до потери напарников |

## 12. Резюме

Agent teams — интерактивный инструмент для спора под присмотром человека, а не исполнитель недельного цикла. MVP нужны работа без терминала, жизнь между днями, детерминированные гейты и строгие роли. Первых трёх у команд нет, а роли держатся только на списке инструментов. Поэтому ТЗ остаётся на субагентах, флаг в репозитории компании — `0`, турнир — независимые судьи. Это совпадает с границей, которую провела сама Anthropic: последовательные задачи — субагентам, команды — для «share findings, challenge each other».

Новое в этом исследовании: параллелизм MVP закрывают субагенты и воркфлоу, а уникальна у команды только живая переписка. Поэтому вопрос «строить ли на командах» сужается до проверяемого: делает ли переписка возражения к топ-3 лучше, чем тот же совет на субагентах. Ответ даст эксперимент на одной стадии за три недели, с потолками денег и времени. Сейчас достаточно 14 правок ТЗ из раздела 9 — это около получаса работы (оценка). Эксперимент — не раньше конца фазы 4. Отказ от команд целиком пересматривать, когда Anthropic выведет их из experimental, исправит потерю команды, idle-шум и самозахват задач, а независимый замер покажет пользу на аналитике.
