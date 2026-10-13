# Agent teams в Claude Code на практике: кейсы, стоимость, сбои, применимость (на 2026-10-13)

Как читать. Метки: **[вендор]** — заявление Anthropic или продавца инструмента; **[самоотчёт]** — цифры автора issue или поста, независимо не проверены; **[независимо]** — внешнее исследование. «По сниппету поиска» — факт виден только в сводке поисковой выдачи, страница не открыта. «Не проверено» — нет подтверждения первичным источником. «Предложение» — мой синтез. «Расчёт» — моя арифметика по цифрам источника.

Ограничение среды. WebFetch открывал только github.com, anthropic.com, claude.com, code.claude.com и platform.claude.com. Не открывались (DNS): news.ycombinator.com, hn.algolia.com, arxiv.org, huggingface.co, research.google, cognition.ai/.com, cursor.com, addyosmani.com, paddo.dev, alexop.dev, growwstacks.com, codex.danielvaughan.com, зеркала Reddit, российские сайты. Всё о них ниже — по сниппетам. Израсходовано 24 вызова WebSearch.

## 1. Кейсы Anthropic

### Главное
Единственный крупный кейс Anthropic с ярлыком «agent teams» — C-компилятор Карлини: 16 агентов, около двух недель, чуть меньше $20 000. Механика там не та, что у вышедшей функции: нет лида, нет обмена сообщениями, задачи захватываются через git-блокировки. Сработала опора на тестовый оракул, а не общение агентов. Официально Anthropic сужает область команд до «независимой работы с преобладанием чтения» (ревью, исследование, конкурирующие гипотезы). В API-платформе (Managed Agents) одноранговых команд нет вовсе, а для большинства многоагентной работы рекомендованы workflows.

### Факты (с источниками)

#### 1.1. «Building a C compiler with a team of parallel Claudes»
- Заголовок, автор, дата, модель: «Building a C compiler with a team of parallel Claudes», Nicholas Carlini (команда Safeguards), 5 февраля 2026, Opus 4.6 — [Anthropic Engineering](https://www.anthropic.com/engineering/building-c-compiler) [вендор].
- Термин в посте есть: «We tasked Opus 4.6 using agent teams to build a C Compiler, and then (mostly) walked away» и «a new approach to supervising language models that we're calling "agent teams."» — [там же](https://www.anthropic.com/engineering/building-c-compiler).
- Харнесс самодельный, это не функция Claude Code с лидом и почтовыми ящиками:
  - bash-цикл `while true` запускает `claude --dangerously-skip-permissions`;
  - каждый агент в своём Docker-контейнере, голый git-репозиторий `/upstream`;
  - блокировка задачи — текстовый файл в `current_tasks/`;
  - «I don't use an orchestration agent»;
  - «I haven't yet implemented any other method for communication between agents»;
  - обычно Claude берёт «next most obvious» проблему.
  
  [Там же](https://www.anthropic.com/engineering/building-c-compiler). Сам харнесс не опубликован, выложен только исходный код компилятора — [там же](https://www.anthropic.com/engineering/building-c-compiler).
- Масштаб и стоимость:
  - 16 агентов, около двух недель;
  - «Over nearly 2,000 Claude Code sessions and $20,000 in API costs», «a total cost just under $20,000»;
  - 2 млрд входных и 140 млн выходных токенов;
  - компилятор на Rust, 100 000 строк.
  
  [Там же](https://www.anthropic.com/engineering/building-c-compiler). Расчёт: около $10 на сессию, около $625 на агента в неделю.
- Результат:
  - собирает загружаемый Linux 6.9 на x86, ARM и RISC-V;
  - компилирует QEMU, FFmpeg, SQLite, postgres, redis и Doom;
  - проходит 99% большинства тестовых наборов компиляторов, включая GCC torture.
  
  [Там же](https://www.anthropic.com/engineering/building-c-compiler).
- Что сработало:
  - пока падающих тестов много, параллелизм тривиален: «each agent picks a different failing test to work on»;
  - на ядре Linux GCC служил известным правильным оракулом: случайные подмножества файлов собирались GCC, и агенты чинили разные баги в разных файлах;
  - специализация ролей: дедупликация, производительность, эффективность кодогенерации, критик кода на Rust, документация;
  - строгие тесты и CI против регрессий.
  
  [Там же](https://www.anthropic.com/engineering/building-c-compiler).
- Что сломалось:
  - «Having 16 agents running didn't help because each was stuck solving the same task»;
  - «Every agent would hit the same bug, fix that bug, and then overwrite each other's changes»;
  - поздние регрессии при добавлении функций;
  - засорение контекста выводом харнесса;
  - «слепота ко времени»: Claude «will happily spend hours running tests instead of making progress», понадобилась опция `--fast` с выборкой тестов;
  - однажды агент выполнил `pkill -9 bash` и убил собственный цикл.
  
  [Там же](https://www.anthropic.com/engineering/building-c-compiler).
- Ограничения результата:
  - нет 16-битного x86 (вызывает GCC);
  - нет своего ассемблера и линкера;
  - код менее эффективен, чем у GCC без оптимизаций;
  - Rust «не экспертного» уровня.
  
  [Там же](https://www.anthropic.com/engineering/building-c-compiler).
- Роль человека:
  - «Most of my effort went into designing the environment around Claude»;
  - «Existing agent scaffolds like Claude Code require an operator to be online and available to work jointly»;
  - «it is easy to see tests pass and assume the job is done, when this is rarely the case».
  
  [Там же](https://www.anthropic.com/engineering/building-c-compiler).
- Прогресс моделей: «Previous Opus 4 models were barely capable of producing a functional compiler»; Opus 4.5 первой перешла порог — [там же](https://www.anthropic.com/engineering/building-c-compiler).
- Реакция сообщества в репозитории компилятора (на 2026-10-13):
  - всего 165 issues, около 49 открыты;
  - самый обсуждаемый — [#1 «Hello world does not compile»](https://github.com/anthropics/claudes-c-compiler/issues/1): открыт с 5 февраля 2026, 337 комментариев, ответа мейнтейнеров нет;
  - причина #1 — драйвер не находит системные `stddef.h` и `stdarg.h`; с ручным `-I` пример собирается;
  - тон обсуждения — в основном насмешки;
  - другие темы: фаззинг csmith/yarpgen ([#227](https://github.com/anthropics/claudes-c-compiler/issues/227), 29 комментариев); утверждения о «следах» LLVM и chibicc (#231, #232 — не проверено).
  
  Список — [issues](https://github.com/anthropics/claudes-c-compiler/issues?q=is%3Aissue+sort%3Acomments-desc).
- Внешняя критика (по сниппету поиска):
  - в зеркале HN спрашивали, загружается ли ядро, а не только собирается, и критиковали качество кода — [brianlovin.com/hn/46903616](https://brianlovin.com/hn/46903616);
  - фактчек Lenz оценил заявление в 7/10 «mostly true»: зависимость от ассемблера и линкера GCC, проблемы надёжности — [lenz.io](https://lenz.io/c/e37520cc).
- Вторичные СМИ путают кейс с функцией: например, «Anthropic Researcher Builds Compiler Using Agent Teams» — [letsdatascience](https://www.letsdatascience.com/news/anthropic-researcher-builds-compiler-using-agent-teams-3ab2ffc4) (по сниппету поиска).

#### 1.2. Анонс функции (вместе с Opus 4.6)
- Анонс — 5 февраля 2026: «We've introduced agent teams in Claude Code as a research preview»; «best for tasks that split into independent, read-heavy work like codebase reviews» — [Introducing Claude Opus 4.6](https://www.anthropic.com/news/claude-opus-4-6) [вендор]. Заявлений о качестве или скорости команд в анонсе нет — [там же](https://www.anthropic.com/news/claude-opus-4-6).
- Цитаты клиентов в анонсе — про субагентов, не про команды:
  - NBIM: Opus 4.6 дал лучший результат в 38 из 40 расследований по кибербезопасности при «up to 9 subagents and 100+ tool calls»;
  - Rakuten: «closed 13 issues and assigned 12 issues» за день.
  
  [Там же](https://www.anthropic.com/news/claude-opus-4-6) [вендор].
- Статус на 2026-10-13: «Agent teams are experimental and disabled by default» — по-прежнему, через восемь месяцев после анонса. Включение: `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`. Тиммейты не создаются в `-p` и Agent SDK — [docs: agent-teams](https://code.claude.com/docs/en/agent-teams).
- Требование версии v2.1.32+ видно только по сниппету поиска; на текущей странице документации его нет — [docs: agent-teams](https://code.claude.com/docs/en/agent-teams).

#### 1.3. Руководства Anthropic: когда команда, а когда субагенты
- Документация (все пункты — [docs: agent-teams](https://code.claude.com/docs/en/agent-teams) [вендор]):
  - сильнейшие сценарии: «Research and review», «New modules or features», «Debugging with competing hypotheses», «Cross-layer coordination»;
  - «For sequential tasks, same-file edits, or work with many dependencies, a single session or subagents are more effective»;
  - «Before you set up a team, check whether a lighter option does the job»;
  - «Use subagents when you need quick, focused workers that report back. Use agent teams when teammates need to share findings, challenge each other, and coordinate on their own».
- Стоимость: «Agent teams use approximately 7x more tokens than standard sessions when teammates run in plan mode». Советы: Sonnet для тиммейтов, маленькие команды, короткие промпты, своевременно выключать тиммейтов — [docs: costs](https://code.claude.com/docs/en/costs) [вендор].
- Блог Claude «Building multi-agent systems: When and how to use them» (23 января 2026, Cara Phillips и др.; [claude.com](https://claude.com/blog/building-multi-agent-systems-when-and-how-to-use-them)) [вендор]:
  - «multi-agent implementations typically use 3-10x more tokens than single-agent approaches for equivalent tasks»;
  - параллелизм даёт «thoroughness, not speed»; такие системы «often take longer overall»;
  - в ролевом эксперименте субагенты «spent more tokens on coordination than on actual work»;
  - делить работу нужно по нужному контексту, а не по типу работы, иначе выходит «telephone game»;
  - «Start with the simplest approach that works, and add complexity only when evidence supports it».
- Блог «How and when to use subagents in Claude Code» (7 апреля 2026): команды — «heavier and more expensive»; «Two subagents editing the same file in parallel is a recipe for conflict» — [claude.com](https://claude.com/blog/subagents-in-claude-code) [вендор].
- Сравнительные таблицы:
  - [docs: workflows](https://code.claude.com/docs/en/workflows): у команды масштаб «A handful of long-running peers», у workflows — «Dozens to hundreds of agents per run»;
  - [docs: agents](https://code.claude.com/docs/en/agents): «Agent teams don't isolate teammates in worktrees».
- Managed Agents (API-платформа Anthropic, beta) предлагает только субагентов, dynamic workflows и advisor — одноранговых команд нет [вендор]:
  - субагенты — «one level deep», не больше 25 дочерних потоков;
  - workflows — для «Most work that needs more than one agent».
  
  [platform docs: multiagent orchestration](https://platform.claude.com/docs/en/managed-agents/multiagent-orchestration).

#### 1.4. Смежные кейсы Anthropic с несколькими агентами (не agent teams)
- Исследовательская система (13 июня 2025; [Anthropic Engineering](https://www.anthropic.com/engineering/multi-agent-research-system)) [вендор, внутренняя оценка]:
  - лид Opus 4 с субагентами Sonnet 4 превзошёл одиночный Opus 4 на 90,2% во внутреннем исследовательском eval;
  - агенты тратят около 4× токенов чата, многоагентные системы — около 15×;
  - на BrowseComp объём токенов объясняет 80% разброса качества;
  - «Multi-agent systems work mainly because they help spend enough tokens to solve the problem»;
  - нужны задачи, где «the value of the task is high enough to pay for the increased performance»;
  - наблюдавшиеся сбои: 50 субагентов на простой запрос, бесконечный поиск несуществующих источников, отвлечение друг друга, дублирование работы;
  - плохо подходят домены с общим контекстом и многими зависимостями, а также большинство задач кодинга.
- Code Review (9 марта 2026; [claude.com](https://claude.com/blog/code-review), [docs: code-review](https://code.claude.com/docs/en/code-review)) [вендор, внутренняя статистика]:
  - механика: команда агентов ищет баги параллельно, отдельный шаг верификации отсеивает ложные срабатывания, находки ранжируются;
  - доля PR с содержательными комментариями выросла с 16% до 54%;
  - на PR больше 1000 строк находки у 84%, в среднем 7,5 проблемы; на PR меньше 50 строк — у 31%, в среднем 0,5;
  - ошибочными инженеры отметили «less than 1%» находок;
  - цена $15–25 за ревью, в среднем около 20 минут.

### Интерпретация
- C-компилятор — не доказательство работоспособности механики лида и почтовых ящиков. Это доказательство другого: много автономных параллельных сессий плюс жёсткий детерминированный оракул (тесты, GCC). Параллелизм работал, пока оракул сам делил работу на независимые куски, и ломался на монолитной задаче. Предложение: в LLM-компании роль оракула играют детерминированные скрипты MVP (`validate.py`, `max_cpc.py`, `swiss.py`), а не переписка агентов.
- Во всех измеренных кейсах Anthropic (исследование, Code Review) — оркестратор с изолированными исполнителями и верификацией, а не одноранговая команда. Количественных публикаций Anthropic о выигрыше agent teams над субагентами я не нашёл.
- Анонс, документация и Managed Agents сужают область команд до независимой работы с преобладанием чтения. Предложение: это сигнал, что основная ставка Anthropic для промышленной многоагентности — workflows и субагенты, а не команды.

### Пробелы
- Нет официального бенчмарка agent teams против субагентов или одиночной сессии. Множитель 7× — единственная цифра Anthropic о стоимости команд, и он дан без методики.
- Харнесс и промпты C-компилятора не опубликованы. Ветку HN (id 46903616 по зеркалу) открыть не удалось.

## 2. Опыт сообщества: цифры и сбои

### Главное
Измеренные данные в основном из GitHub issues, это самоотчёты. Типичная картина: токены растут в 2–4 раза против субагентов или одиночной сессии. Сверху ложится шум уведомлений о простое (13–22% входных токенов лида). Бывают тяжёлые дубликаты тиммейтов: в самоотчётах от 2× до 12× токенов, в крайнем случае — десятки тысяч долларов. Типовые сбои:
- лид теряет команду после компакции;
- тиммейты берут не свои задачи;
- поток запросов разрешений;
- лид делает работу сам;
- результаты «именованных» субагентов теряются, когда флаг команд включён.

Контролируемых сравнений нет. Блоги дают единичные противоречивые замеры.

### Факты (с источниками)

| Источник | Что делали | Размер команды | Стоимость/токены | Что сломалось | URL |
|---|---|---|---|---|---|
| Anthropic, Carlini [вендор] | C-компилятор на Rust, около 2 недель | 16 параллельных агентов (свой харнесс) | чуть меньше $20 000; 2 млрд вход + 140 млн выход; около 2000 сессий | все агенты чинили один баг и затирали правки друг друга; регрессии; часы на тесты; `pkill -9 bash` | [anthropic.com](https://www.anthropic.com/engineering/building-c-compiler) |
| GitHub #47930 [самоотчёт] | Opus 4.6, v2.1.107: 5 + 3 задачи, все выполнены без переделок | 8 тиммейтов | командная часть: 138 ходов лида, 23,3 млн входных токенов; та же работа фоновыми `Agent` — около 30–40 ходов (раздувание в 3–4 раза) | 13,0% (строго) — 22,2% (широко) входных токенов на пустые «ack» по idle-уведомлениям и эхо `task_assignment`; уведомления идут минутами после завершения | [issue](https://github.com/anthropics/claude-code/issues/47930) |
| Комментарии к #47930 [самоотчёт] | парк агентов за неделю; ночная сессия | 4 агента | около 18% потерь за неделю; около 20 одинаковых пингов за ночь ≈ $8–10 накладных | idle-пинги не отключаются: конструктор вызывается без флага (анализ бинаря v2.1.204) | [issue](https://github.com/anthropics/claude-code/issues/47930) |
| GitHub #32996 [самоотчёт] | отчёт тремя ролями (Restorer, Auditor, Evolver), v2.1.72 | 3 | 2,2× по объёму транскриптов: 1008 КБ субагентами против 2220 КБ командой; позже 2,5× за 90 с (6 тиммейтов) и 12× в 30-минутном конвейере; «5-12x token waste» | каждый тиммейт создан дважды, поздняя копия перезаписала отчёт ранней; гипотеза — гонка при параллельном спавне | [issue](https://github.com/anthropics/claude-code/issues/32996) |
| GitHub #55586 [самоотчёт] | реальные проекты 14 недель, v2.1.90–2.1.126, Windows/WSL | 1 вызов `Agent()` → 10–151 дубликат; пик — 389 агентов одновременно | худшая сессия — 42,9 млрд токенов (около $43 000 по API); за 14 недель — 90,8 млрд (около $91 000), из них около $86 000 на дубликаты | невидимые дубликаты правят файлы: 12 974 правки и 8 836 записей у одной роли; независимое воспроизведение на macOS дало 30 вызовов → 107 транскриптов | [issue](https://github.com/anthropics/claude-code/issues/55586) |
| GitHub #23620 и комментарии [самоотчёт] | 16 страниц Astro (написание и ревью); команда из 4 агентов на кодовой базе около 150 тыс. токенов | несколько; 4 | — | после компакции лид «completely loses awareness of the team»; воспроизводится примерно за 45 минут; «the number 1 most frequent bug I hit with Agent Teams» | [issue](https://github.com/anthropics/claude-code/issues/23620) |
| GitHub #93668 [самоотчёт] | внутренний прогон: лид, 3 исполнителя, 1 ревьюер только на чтение | лид + 4 | — | ревьюер раз за разом брал задачи реализации, пока лид вручную не переназначил все 14 | [issue](https://github.com/anthropics/claude-code/issues/93668) |
| GitHub #71723 [самоотчёт + подтверждение мейнтейнера] | проект со 138 именованными и 191 безымянным запуском `Agent` | — | idle-уведомление около 200 токенов | с включённым флагом именованный субагент уходит в протокол тиммейта, результат теряется; около 20 минут на случай; один пользователь потерял трёх воркеров за день | [issue](https://github.com/anthropics/claude-code/issues/71723) |
| GitHub #26479 [самоотчёт] | тиммейты при 100+ allow-правилах в `settings.local.json` | — | — | «dozens of prompts per session»; `bypassPermissions` не действует на Bash; тиммейты блокируются до одобрения лидом | [issue](https://github.com/anthropics/claude-code/issues/26479) |
| GitHub #24414 [самоотчёт] | запуск тиммейта без задачи | N | «~4 turns of token burn» на тиммейта; «N teammates = ~2N wasted lead turns» (оценка автора) | пустая переписка при старте | [issue](https://github.com/anthropics/claude-code/issues/24414) |
| mejba.me [самоотчёт; по сниппету поиска] | одно и то же to-do-приложение двумя способами | 1 агент Opus против команды: лид Opus + 2 Sonnet + 1 Haiku | около 45 тыс. токенов и 8 минут против около 180 тыс. и 22 минут (в версии команды больше функций) | команда медленнее и дороже; по пересказу в выдаче, автор считает, что команды не всегда лучше и бывают намного хуже | [mejba.me](https://www.mejba.me/claude-agent-teams-guide) |
| growwstacks.com [вендорский блог; по сниппету поиска] | «coordination-heavy workflows» | — | 2–3 минуты командой против 5–10 минут субагентами; методики нет | — | [growwstacks](https://growwstacks.com/blog/claude-agent-teams-vs-sub-agents/) |
| Неуточнённый блог из выдачи [по сниппету поиска; атрибуция не проверена, вероятно growwstacks] | пример | — | около 12 тыс. токенов субагентами против 36–48 тыс. командой | — | [growwstacks (вероятно)](https://growwstacks.com/blog/claude-agent-team-mistakes/) |
| Гайды о «delegate mode» [по сниппету поиска] | лид в режиме только координации (Shift+Tab) | — | — | у одного автора тиммейты унаследовали режим и застряли на запросах доступа к файлам, а в обычном режиме всё заработало. Связанный issue «#23447 Delegate Mode strips tools» упомянут в #26479 | [claudefa.st](https://claudefa.st/blog/guide/agents/agent-teams-controls); [#26479](https://github.com/anthropics/claude-code/issues/26479) |
| Steve Yegge, Gas Town (сторонний оркестратор) [по сниппету поиска] | 20–30 параллельных агентов | 20–30 | около €10 400 в месяц на 21 аккаунт; квота одного агента кончалась за 2–4 часа | по одному источнику проект закрыт (не проверено) | [drweb.de](https://www.drweb.de/ki-agenten-token-budget-feste-rollen/); [ecosistemastartup](https://ecosistemastartup.com/?p=108516) |
| Cursor, «Scaling long-running autonomous coding» (не Claude Code) [вендор; по сниппету поиска] | плоская самокоординация через общий файл и блокировки, затем иерархия «планировщики → воркеры → судья» | 20 → сотни | — | блокировки держались слишком долго или не снимались; 20 агентов работали с эффективной пропускной способностью 2–3 агентов; иерархия масштабировалась лучше | [cursor.com](https://cursor.com/blog/scaling-agents); [zenml](https://www.zenml.io/llmops-database/scaling-multi-agent-autonomous-coding-systems) |

Сводка множителей стоимости:
- Около 7× токенов против обычной сессии, когда тиммейты в plan mode — [docs: costs](https://code.claude.com/docs/en/costs) [вендор].
- 3–10× для многоагентных систем против одного агента — [claude.com](https://claude.com/blog/building-multi-agent-systems-when-and-how-to-use-them) [вендор].
- Около 15× против чата для исследовательской системы на субагентах — [anthropic.com](https://www.anthropic.com/engineering/multi-agent-research-system) [вендор].
- Самоотчёты:
  - около 4× токенов и 2,75× времени ([mejba.me](https://www.mejba.me/claude-agent-teams-guide), по сниппету поиска);
  - 3–4× ходов лида и 13–22% шума ([#47930](https://github.com/anthropics/claude-code/issues/47930));
  - 2,2× из-за дубликатов ([#32996](https://github.com/anthropics/claude-code/issues/32996)).

Русскоязычные источники:
- Практических отчётов об agent teams на Хабре и vc.ru не нашлось.
- Пост [habr.com/ru/posts/1010120](https://habr.com/ru/posts/1010120) по одному сниппету — обзор Agent Teams, по другому — про контекст 1M, `/btw` и `/loop`. Сниппеты противоречат друг другу, страница не открывалась.
- Пост [habr.com/ru/posts/986080](https://habr.com/ru/posts/986080): Борис Черный запускает пять Claude параллельно в терминале — это не agent teams (по сниппету поиска).

### Интерпретация
- Главные сбои в практике — не качество рассуждений, а инфраструктура координации:
  - доставка и дублирование сообщений;
  - жизненный цикл тиммейтов;
  - состояние команды после компакции;
  - разрешения.
  
  Именно эти ошибки делают прогоны недетерминированными и дорогими.
- Рост токенов в 2–4 раза — нижняя граница при исправной работе. Шум idle-уведомлений и редкие катастрофы с дубликатами добавляют сверху, причём пользователь без специальной проверки их не видит. Как проверять: считать `agent-*.meta.json` и транскрипты в `subagents/` ([#55586](https://github.com/anthropics/claude-code/issues/55586)).
- Выигрыш по скорости не подтверждён. Единичные замеры противоречат друг другу (mejba.me против growwstacks), и это совпадает с позицией Anthropic: «thoroughness, not speed».

### Пробелы
- Треды HN и Reddit об agent teams открыть не удалось; цифр оттуда нет.
- Посты Osmani («Claude Code Swarms»), paddo.dev («Agent Teams: The Switch Got Flipped», «Claude Code's Hidden Multi-Agent System») и alexop.dev видны только как заголовки в выдаче, содержание не проверено — [addyosmani.com](https://addyosmani.com/blog/claude-code-agent-teams/), [paddo.dev](https://paddo.dev/blog/agent-teams-the-switch-got-flipped/), [alexop.dev](https://alexop.dev/posts/from-tasks-to-swarms-agent-teams-in-claude-code/).
- Нет ни одного контролируемого замера «команда против субагентов» на одной задаче с повторами.

## 3. Топ-проблемы из GitHub issues

### Главное
Объём на 2026-10-13 (суммы открытых и закрытых, пересекаются):
- 348 issues со словом «teammate» в заголовке — [поиск](https://github.com/anthropics/claude-code/issues?q=is%3Aissue+in%3Atitle+teammate+sort%3Acomments-desc);
- 359 — с «agent team» — [поиск](https://github.com/anthropics/claude-code/issues?q=is%3Aissue+in%3Atitle+%22agent+team%22+sort%3Acomments-desc);
- 54 открытых — с «agent teams» — [поиск](https://github.com/anthropics/claude-code/issues?q=is%3Aissue+is%3Aopen+in%3Atitle+%22agent+teams%22+sort%3Areactions-%2B1-desc).

Многие тяжёлые баги закрыты ботом как «not planned» за неактивность, без исправления: дубликаты тиммейтов (#55586, #32996) и пустые ходы при старте (#24414). Ключевые баги координации открыты месяцами: компакция (#23620, с февраля), шум idle (#47930, с апреля), разрешения (#26479, с февраля). Ответы мейнтейнеров редки.

### Факты (с источниками)

| Issue | Симптом | Статус на 2026-10-13 | URL |
|---|---|---|---|
| #23620 | После компакции контекста лид теряет команду: не может писать тиммейтам и заново раздаёт уже идущую работу; ошибок нет | Открыт с 6 февраля 2026; 18 комментариев; со стороны мейнтейнеров только метка (19 августа) | [#23620](https://github.com/anthropics/claude-code/issues/23620) |
| Ограничение в документации | `/resume` и `/rewind` не восстанавливают in-process тиммейтов; лид пишет несуществующим тиммейтам | Задокументированное ограничение; запрос на «graceful restart» — [#90247](https://github.com/anthropics/claude-code/issues/90247), открыт | [docs](https://code.claude.com/docs/en/agent-teams#limitations) |
| #55586 | Один спавн порождает 10–151 невидимый дубликат, все правят файлы и тратят полный контекст | Закрыт 25 августа 2026 как «not planned» (бот за неактивность), исправления нет | [#55586](https://github.com/anthropics/claude-code/issues/55586) |
| #32996 | В in-process режиме тиммейты систематически дублируются, токены удваиваются | Закрыт 11 мая 2026 как «not planned» (бот), исправления нет | [#32996](https://github.com/anthropics/claude-code/issues/32996) |
| #92244 | Сообщение или idle-уведомление тиммейта приходит лиду до 4 раз с одной меткой времени (наследник #74112, закрытого за неактивность) | Открыт с 5 сентября 2026 | [#92244](https://github.com/anthropics/claude-code/issues/92244) |
| #47930 | Лид зацикливается на idle-уведомлениях и эхо-назначениях: 13–22% входных токенов | Открыт с 14 апреля 2026, метка `area:cost`; ответа мейнтейнеров нет | [#47930](https://github.com/anthropics/claude-code/issues/47930) |
| #85047 | «Пинг-понг» idle: подтверждение будит тиммейта, он шлёт новое уведомление | Открыт с 8 августа 2026 | [#85047](https://github.com/anthropics/claude-code/issues/85047) |
| #24414 | Запуск тиммейта без задачи сжигает около 4 ходов | Закрыт 10 марта 2026 как «not planned» (бот) | [#24414](https://github.com/anthropics/claude-code/issues/24414) |
| #77300 | События Monitor и фоновых задач не будят простаивающего тиммейта и теряются; будит только `SendMessage` | Открыт с 13 июля 2026 | [#77300](https://github.com/anthropics/claude-code/issues/77300) |
| #38932, #51959 | Лид не обрабатывает ответы тиммейтов до ручного нажатия клавиши: «breaks unattended orchestration» | Оба закрыты ботом (25 июня и 1 сентября 2026) | [#38932](https://github.com/anthropics/claude-code/issues/38932), [#51959](https://github.com/anthropics/claude-code/issues/51959) |
| #50779 | Входящие лиду откладываются до `end_turn` и теряются в цепочках tool_use (регрессия) | Закрыт ботом 26 июля 2026 | [#50779](https://github.com/anthropics/claude-code/issues/50779) |
| #93668 | Самозахват задач без фильтра по ролям: тиммейт только на чтение берёт задачи, которые не может сделать | Открыт с 11 сентября 2026; ответа нет | [#93668](https://github.com/anthropics/claude-code/issues/93668) |
| #71723 | Параметр `name` у Agent тихо переводит субагента в протокол тиммейта, результат теряется | Открыт; 17 августа 2026 мейнтейнер (bcherny) воспроизвёл: «Confirmed as a bug», исправление «already in progress»; обход — не передавать `name` | [#71723](https://github.com/anthropics/claude-code/issues/71723) |
| #34476 | Нельзя остановить команду, не убив сессию | Закрыт 9 июля 2026; сейчас в документации тиммейта останавливают клавишей `x` | [#34476](https://github.com/anthropics/claude-code/issues/34476), [docs](https://code.claude.com/docs/en/agent-teams) |
| #60199 | Одобренный `shutdown_request` не завершает тиммейта | Закрыт ботом 30 июля 2026 | [#60199](https://github.com/anthropics/claude-code/issues/60199) |
| #26479 | `bypassPermissions` не действует на Bash у тиммейтов, `settings.local.json` не наследуется, «dozens of prompts per session» | Открыт с 18 февраля 2026; подтверждён на 2.1.283 (30 сентября); со стороны мейнтейнеров только метка | [#26479](https://github.com/anthropics/claude-code/issues/26479) |
| #23983 | Хуки PermissionRequest не срабатывают на запросы тиммейтов, автоматизировать одобрения нельзя | Открыт с 7 февраля 2026 | [#23983](https://github.com/anthropics/claude-code/issues/23983) |
| #49865 | Лид падает с «getAppState is not a function», когда тиммейт просит разрешение | Закрыт 17 апреля 2026 | [#49865](https://github.com/anthropics/claude-code/issues/49865) |
| #32368, #80569, #78234 | Ролевые настройки применяются частично: модель не наследовалась (закрыт ботом 18 июля); `effort` из определения игнорируется (открыт); определения из плагинов игнорируются (открыт) | См. слева | [#32368](https://github.com/anthropics/claude-code/issues/32368), [#80569](https://github.com/anthropics/claude-code/issues/80569), [#78234](https://github.com/anthropics/claude-code/issues/78234) |
| #23669 | Нет у каждого тиммейта своего рабочего каталога, CLAUDE.md и MCP | Открыт с 6 февраля 2026; 26 комментариев | [#23669](https://github.com/anthropics/claude-code/issues/23669) |
| #29207, #23572, #24301, #23815 | Сплит-панели tmux или iTerm2 тихо откатываются в in-process | Закрыты (март — май 2026), часть ботом | [#29207](https://github.com/anthropics/claude-code/issues/29207), [#23572](https://github.com/anthropics/claude-code/issues/23572), [#24301](https://github.com/anthropics/claude-code/issues/24301), [#23815](https://github.com/anthropics/claude-code/issues/23815) |
| #23615, #24385, #27882 | Панели делят текущее окно tmux; панели iTerm2 не закрываются после выключения; «зомби-тиммейты» в UI | #23615 и #24385 открыты; #27882 закрыт ботом | [#23615](https://github.com/anthropics/claude-code/issues/23615), [#24385](https://github.com/anthropics/claude-code/issues/24385), [#27882](https://github.com/anthropics/claude-code/issues/27882) |
| #24189, #24384, #24122, #23574, #26572 | Нет бэкендов панелей для Ghostty, Windows Terminal, zellij, WezTerm; предложен протокол CustomPaneBackend. В документации: панели не поддерживаются в терминале VS Code, Windows Terminal и Ghostty | Все открыты с февраля 2026 | [#24189](https://github.com/anthropics/claude-code/issues/24189), [#24384](https://github.com/anthropics/claude-code/issues/24384), [#26572](https://github.com/anthropics/claude-code/issues/26572), [docs](https://code.claude.com/docs/en/agent-teams#limitations) |
| #98550, #98657 | Вид in-process тиммейта хранит только последние 50 сообщений и обрезается до одного при завершении; у тиммейта в своей панели висит «manual mode» и бесконечный таймер | Открыты (30 сентября и 1 октября 2026) | [#98550](https://github.com/anthropics/claude-code/issues/98550), [#98657](https://github.com/anthropics/claude-code/issues/98657) |
| #42005, #60591 | Статистика тиммейта всегда «0 tool uses / 0 tokens»; нет агрегированной стоимости по командам | Оба закрыты ботом (30 мая и 25 июня 2026) | [#42005](https://github.com/anthropics/claude-code/issues/42005), [#60591](https://github.com/anthropics/claude-code/issues/60591) |

- Часть проблем исправлена, судя по версионным заметкам в документации:
  - битая запись в mailbox блокировала доставку до v2.1.207;
  - тиммейты в панелях загружали все источники настроек до v2.1.281;
  - не наследовали `effort` лида до v2.1.186;
  - `teammateDefaultModel` удалён в v2.1.234;
  - порядок выбора модели изменён в v2.1.251.
  
  [Docs: agent-teams](https://code.claude.com/docs/en/agent-teams). Полный CHANGELOG (8789 строк) через WebFetch прочитать не удалось — [CHANGELOG](https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md).
- Запрос #24316 на определения из `.claude/agents/` в роли тиммейтов (43 комментария) закрыт 17 августа 2026. Сейчас документация это поддерживает, но частично:
  - `skills` из определения не применяются;
  - in-process тиммейт игнорирует `mcpServers` из определения и грузит MCP-серверы проекта и пользователя.
  
  [#24316](https://github.com/anthropics/claude-code/issues/24316), [docs](https://code.claude.com/docs/en/agent-teams#use-subagent-definitions-for-teammates).

### Интерпретация
- Статус «closed» у issues по командам часто не значит «fixed». Бот закрывает за неактивность, и пользователи открывают «наследников» (#92244 после #74112).
- Классы проблем, опасные для конвейера, который должен быть воспроизводимым:
  - недетерминированный самозахват задач и дубликаты;
  - потеря команды при компакции или resume;
  - лид не обрабатывает сообщения без участия человека;
  - запросы разрешений.
  
  Предложение: команду нельзя считать «надёжным исполнителем стадии». Это интерактивный инструмент под присмотром человека.

### Пробелы
- Не проверено, какие из закрытых ботом багов исправлены в коде: CHANGELOG не прочитан, а в issues нет ссылок на фиксы.
- Цифры в issues — самоотчёты, Anthropic их не подтверждала, кроме воспроизведения #71723.

## 4. Некодовые применения

### Главное
Anthropic сама советует начинать с команд на задачах без кода: исследование, ревью, конкурирующие гипотезы. Но измеренных результатов именно agent teams на некодовых задачах я не нашёл ни у Anthropic, ни у сообщества. Есть два вида косвенных данных:
- **За:** оркестратор с независимыми исполнителями и верификацией работает на задачах, которые параллелятся, — исследовательская система на субагентах (+90,2%) и Code Review (меньше 1% ложных находок).
- **Против:** дебаты однотипных моделей при равном бюджете вычислений часто не лучше self-consistency, а на последовательных задачах многоагентность ухудшает результат на 39–70%.

### Факты (с источниками)
- Из документации ([docs: agent-teams](https://code.claude.com/docs/en/agent-teams)) [вендор, без замеров]:
  - «Start with research and review… tasks that have clear boundaries and don't require writing code»;
  - пример: три тиммейта по UX, архитектуре и «devil's advocate»;
  - пример: 5 тиммейтов опровергают гипотезы друг друга «like a scientific debate». Обоснование: при последовательном расследовании возникает якорение, и выжившая теория «much more likely to be the actual root cause».
- Встроенный workflow `/deep-research` веером ищет, перекрёстно проверяет источники и «votes on each claim». Утверждения, не пережившие проверку, отбрасываются. Это нынешний путь Anthropic для «cross-checked research» — workflow, а не команда — [docs: workflows](https://code.claude.com/docs/en/workflows) [вендор].
- Исследовательская система Anthropic (субагенты): +90,2% против одиночного агента; около 15× токенов; объём токенов объясняет 80% разброса; проблемы — дублирование и отвлечение — [anthropic.com](https://www.anthropic.com/engineering/multi-agent-research-system) [вендор].
- «Towards a Science of Scaling Agent Systems» (arXiv 2512.08296; Google Research и MIT; по сниппету поиска) [независимо]:
  - централизованная координация даёт +80,9% на задачах, которые параллелятся (финансовые рассуждения);
  - на последовательных задачах все многоагентные варианты ухудшили результат на 39–70% (по пересказу аннотации);
  - независимые агенты усиливают ошибки в 17,2 раза, централизованные — в 4,4 раза;
  - 180 конфигураций, предсказательная модель точна на 87% отложенных конфигураций.
  
  [Google Research](https://research.google/blog/towards-a-science-of-scaling-agent-systems-when-and-why-agent-systems-work/), [arXiv](https://arxiv.org/abs/2512.08296v2).
- Дебаты агентов [независимо; по сниппету поиска]:
  - «Stop Overvaluing Multi-Agent Debate» (arXiv 2502.08788): 5 методов, 9 бенчмарков, 4 модели; дебаты часто не превосходят простые одиночные базовые линии (Chain-of-Thought, Self-Consistency) даже при заметно большем бюджете вычислений; помогает разнородность моделей — [arXiv](https://arxiv.org/abs/2502.08788v3);
  - «Should we be going MAD?»: дебаты надёжно не превосходят self-consistency и ансамбли, хотя после настройки гиперпараметров некоторые (Multi-Persona) лучше — [arXiv](https://arxiv.org/html/2311.17371v3).
- MAST, «Why Do Multi-Agent LLM Systems Fail?» (NeurIPS 2025; по сниппету поиска) [независимо]:
  - 14 режимов отказа в трёх группах: проектирование системы, рассогласование агентов, верификация задачи;
  - 7 фреймворков, больше 1600 трасс, каппа 0,88;
  - сбои в основном от проектирования и рассогласования, а не от слабости моделей.
  
  [arXiv](https://arxiv.org/html/2503.13657v3).
- Сообщество, бизнес и маркетинг (только по сниппетам поиска):
  - отчёт о рынке и конкурентах командой за «about eight minutes», цифры не подтверждены — [growwstacks](https://growwstacks.com/blog/build-ai-agent-teams-with-claude-code-for-deep-market-research);
  - персоны-оппоненты для стратегических решений, без данных — [MindStudio](https://www.mindstudio.ai/blog/claude-code-agent-teams-brainstorming);
  - «Two AI agents whose only job is to kill my ideas», виден только заголовок — [calevans.com](https://calevans.com/blog/two-ai-agents-whose-only-job-is-to-kill-my-ideas.html);
  - скилл «the-fool» для pre-mortem и red-team бизнес-предложений — [claudeskills.info](https://claudeskills.info/skills/jeffallan/claude-skills/the-fool/);
  - промпт-подход «adversarial PM» — [substack](https://lertechnotes.substack.com/p/validate-before-you-vibe-code-how).
  
  Ни в одном из этих источников нет сравнения с одиночным агентом.

### Интерпретация
- Предложение: качество некодовой аналитики растёт от двух вещей — покрытия (больше независимых читателей и токенов) и независимой верификации. Оба механизма есть у субагентов и workflows без переписки агентов.
- Переписка тиммейтов полезна в одном сценарии — состязательном. Но доказательная база дебатов однотипных моделей слабая, а все тиммейты — Claude.
- Для оценок вроде турнира независимость суждений — ценность. Переписка судей её разрушает.

### Пробелы
- Не найдено ни одного измерения качества agent teams на бизнес-аналитике, маркетинге или исследованиях против одиночного агента или субагентов.
- Первичные тексты arXiv, Google и Cognition открыть не удалось.

## 5. Сравнение с субагентами и воркфлоу

### Главное
Anthropic позиционирует инструменты так:
- **Команды** — «горстка долгоживущих пиров», которые обсуждают и спорят. Только в интерактивной сессии, без возобновления.
- **Субагенты** — сфокусированные исполнители с отчётом вызывающему.
- **Workflows** — код держит план; масштаб — десятки и сотни агентов; можно возобновить; есть лимиты и предупреждения о стоимости; работают в `-p` и SDK.

Внешние оркестраторы (Ruflo/claude-flow, Gas Town) — надстройки с собственными заявлениями без независимых бенчмарков против agent teams. Независимые разборы (Cursor, Cognition, Google) сходятся: иерархия и единственный писатель лучше плоского роя пишущих агентов.

### Факты (с источниками)

| Подход | Кто держит план | Масштаб | Прерывание и возобновление | Стоимость и лимиты | Когда, по Anthropic |
|---|---|---|---|---|---|
| Субагенты | Claude по ходу диалога | «A few delegated tasks per turn» | «Restarts the turn» | Дешевле: результат сжимается в контекст вызывающего | «quick, focused workers that report back» |
| Agent teams | Лид по ходу диалога; общий список задач и mailbox | «A handful of long-running peers»; 3–5 тиммейтов | «Teammates keep running»; `/resume` их не восстанавливает | Около 7× в plan mode; растёт линейно с числом тиммейтов; лимита размера команды нет | «share findings, challenge each other, and coordinate on their own» |
| Dynamic workflows | Скрипт | «Dozens to hundreds of agents per run»; по умолчанию до 16 одновременно, до 1000 на прогон | «Resumable in the same session»; пауза при лимите подписки (кроме тиммейтов) | Предупреждение «Large workflow» при больше 25 агентах или больше 1,5 млн токенов; ориентир размера small (<5), medium (<10), large (<50) | «A job outgrows a handful of subagents, or you want findings verified against each other» |

Источники таблицы: [docs: agent-teams](https://code.claude.com/docs/en/agent-teams), [docs: workflows](https://code.claude.com/docs/en/workflows), [docs: agents](https://code.claude.com/docs/en/agents), [docs: costs](https://code.claude.com/docs/en/costs) [вендор].

- У workflows «No mid-run user input». Совет: «For sign-off between stages, run each stage as its own workflow» — [docs: workflows](https://code.claude.com/docs/en/workflows).
- Команды не работают в `-p` и Agent SDK — [docs: agent-teams](https://code.claude.com/docs/en/agent-teams). Workflows доступны в CLI, `-p` и SDK — [docs: workflows](https://code.claude.com/docs/en/workflows).
- «Agent teams don't isolate teammates in worktrees». Субагенты и сессии могут получать свой worktree — [docs: agents](https://code.claude.com/docs/en/agents).
- Ruflo (бывший claude-flow) [вендор]:
  - MIT, «agent meta-harness for Claude Code and Codex»;
  - около 74,5 тыс. звёзд;
  - заявляет 100+ (или 98) агентов, 314 MCP-инструментов, топологии роя с консенсусом;
  - требует Claude Code 2.1.287+;
  - плагины не изолированы и работают с правами аккаунта;
  - прямого сравнения со встроенными командами в README нет.
  
  [GitHub ruvnet/ruflo](https://github.com/ruvnet/ruflo).
- Сравнительные обзоры Ruflo и agent teams видны только в выдаче. Вывод там такой: это разные уровни; у Ruflo расход API выше. Один из обзоров написал конкурент (Bernstein) — [danielvaughan](https://codex.danielvaughan.com/2026/04/09/claude-multi-agent-ecosystem/), [braindetox](https://braindetox.kr/en/posts/ruflo_claude_code_multi_agent_2026.html), [bernstein](https://bernstein.readthedocs.io/en/latest/compare/bernstein-vs-claude-flow/) (по сниппету поиска).
- Cognition (по сниппету поиска):
  - «Don't Build Multi-Agents» (июнь 2025): главная причина провалов — потеря общего контекста и конфликт неявных решений (пример с Flappy Bird);
  - продолжение «Multi-Agents: What's Actually Working» (апрель 2026): прежние выводы верны для роёв параллельных писателей. Работает класс схем, где интеллект вносят несколько агентов, а запись остаётся однопоточной.
  
  [cognition.ai](https://cognition.ai/blog/multi-agents-working), [пересказ принципов](https://agentic-ai.readthedocs.io/en/latest/ContextEngineering/devin/).
- Cursor (по сниппету поиска): плоская координация через блокировки провалилась, иерархия «планировщики, воркеры, судьи» сработала — [cursor.com](https://cursor.com/blog/scaling-agents).
- Бенчмарки: контролируемых сравнений «agent teams против субагентов против workflows» не найдено. Есть только единичные замеры из раздела 2 (mejba.me, growwstacks), и они противоречат друг другу.

### Интерпретация
- Предложение: для конвейера с фиксированными стадиями и обменом через файлы подходит модель «план держит код или скилл». Это субагенты со стадийными скиллами или workflows.
- Уникальная ценность команд — переписка пиров. Она нужна только для состязательного обсуждения, а для него есть более управляемые замены: workflow со взаимной проверкой агентов, отдельный субагент-оппонент.
- Независимые разборы (Cognition, Cursor, Google) поддерживают схему «один писатель, много читателей и проверяющих» и иерархию. Плоская самокоординация с общим списком задач — схема, которую Cursor отбросил.

### Пробелы
- Нет независимого бенчмарка Ruflo или Gas Town против встроенных команд.
- Workflows — сравнительно новая функция. Отчётов практиков о её надёжности я не искал: вне рамки и без бюджета поиска.

## 6. Практики

### Главное
Рабочие практики из документации и issues:
- начинать с задач только на чтение;
- 3–5 тиммейтов, 5–6 задач на тиммейта;
- самодостаточный промпт спавна с целью, форматом выхода и границами;
- владение файлами без пересечений;
- назначать задачи явно, не полагаясь на самозахват;
- держать состояние команды вне контекста лида;
- гейты через хуки;
- заранее одобрить разрешения;
- выключать тиммейтов.

Встроенного потолка размера команды и бюджета нет. Управлять стоимостью приходится дисциплиной и ручным контролем.

### Факты (с источниками)
1. **С чего начинать.** С ревью PR, изучения библиотеки или расследования бага. Это показывает ценность параллельного исследования без проблем параллельной реализации — [docs: agent-teams](https://code.claude.com/docs/en/agent-teams).
2. **Размер команды.**
   - «Start with 3-5 teammates for most workflows»;
   - на 15 независимых задач — 3 тиммейта;
   - «Three focused teammates often outperform five scattered ones»;
   - «Having 5-6 tasks per teammate keeps everyone productive».
   
   [Docs](https://code.claude.com/docs/en/agent-teams).
3. **Грануляция задач.** Слишком мелкие — координация дороже пользы. Слишком крупные — долгая работа без сверки. В самый раз — самодостаточная единица с понятным результатом — [docs](https://code.claude.com/docs/en/agent-teams).
4. **Промпт спавна.**
   - Тиммейт не наследует историю лида, поэтому все детали задачи — в промпте спавна. Тиммейтам задают предсказуемые имена — [docs](https://code.claude.com/docs/en/agent-teams).
   - В промпт входят цель, формат выхода, указания по инструментам и источникам, границы задачи — [Anthropic, research system](https://www.anthropic.com/engineering/multi-agent-research-system).
   - Промпт короткий: всё в нём ложится в контекст каждого тиммейта — [docs: costs](https://code.claude.com/docs/en/costs).
   - Работу делят по нужному контексту, а не по типу работы — [claude.com](https://claude.com/blog/building-multi-agent-systems-when-and-how-to-use-them).
5. **Против дублирования.**
   - Каждому тиммейту — своя «линза» (пример ревью: безопасность, производительность, тесты) — [docs](https://code.claude.com/docs/en/agent-teams).
   - «Break the work so each teammate owns a different set of files» — [docs](https://code.claude.com/docs/en/agent-teams).
   - Задачи заранее назначаются владельцу через `TaskUpdate({owner})`, на ревью ставится `blockedBy`. Самозахват не учитывает роли — [#93668](https://github.com/anthropics/claude-code/issues/93668).
   - Проверка на дубликаты — считать транскрипты в `subagents/`. Тиммейтов лучше не спавнить пачкой: гипотеза о гонке — [#55586](https://github.com/anthropics/claude-code/issues/55586), [#32996](https://github.com/anthropics/claude-code/issues/32996).
   - Захват задач файлами-блокировками и разделение работы оракулом — [C compiler](https://www.anthropic.com/engineering/building-c-compiler).
6. **Лид не должен работать сам.**
   - Промпт «Wait for your teammates to complete their tasks before proceeding». Лид также может объявить конец работы раньше времени — тогда велеть продолжать — [docs](https://code.claude.com/docs/en/agent-teams).
   - В гайдах сообщества есть «delegate mode» по Shift+Tab, но с жалобами, что тиммейты теряют инструменты (по сниппету поиска) — [claudefa.st](https://claudefa.st/blog/guide/agents/agent-teams-controls), #23447 в [#26479](https://github.com/anthropics/claude-code/issues/26479). В текущей странице документации delegate mode не упоминается — [docs](https://code.claude.com/docs/en/agent-teams).
7. **Гейты качества.**
   - Хуки `TeammateIdle`, `TaskCreated`, `TaskCompleted`: код выхода 2 возвращает тиммейта к работе или запрещает создать либо закрыть задачу — [docs](https://code.claude.com/docs/en/agent-teams).
   - Plan approval — не гейт: «Claude Code approves the plan in the lead's session as soon as the request arrives, without the lead reviewing it» — [docs](https://code.claude.com/docs/en/agent-teams).
   - Против «early victory problem» — явное требование прогнать полный набор проверок — [claude.com](https://claude.com/blog/building-multi-agent-systems-when-and-how-to-use-them).
8. **Разрешения.**
   - Запросы тиммейтов всплывают у лида; общие операции одобряют заранее — [docs](https://code.claude.com/docs/en/agent-teams).
   - `bypassPermissions` у тиммейтов на Bash может не действовать, хуки PermissionRequest на тиммейтов не срабатывают — [#26479](https://github.com/anthropics/claude-code/issues/26479), [#23983](https://github.com/anthropics/claude-code/issues/23983).
9. **Длинные сессии.**
   - Состояние команды держать вне контекста (`team-state.md` или протокол в CLAUDE.md) и перечитывать после компакции. Но хук SessionStart(compact), по отзыву пользователя, «Claude ignores it» — [#23620](https://github.com/anthropics/claude-code/issues/23620).
   - Не оставлять команду без присмотра надолго — [docs](https://code.claude.com/docs/en/agent-teams).
10. **Стоимость.**
    - Sonnet для тиммейтов, маленькие команды, выключать тиммейтов после работы — [docs: costs](https://code.claude.com/docs/en/costs).
    - Встроенного `maxTeamSize` нет: запрос в [#55586](https://github.com/anthropics/claude-code/issues/55586); `maxParallelAgents` (#15487) закрыт как not planned (по ссылке из #55586).
    - Esc на лиде разрывает цикл idle-уведомлений — [#47930](https://github.com/anthropics/claude-code/issues/47930).
    - Жёсткие бюджеты есть у других механизмов: ориентир размера workflows и предупреждение «Large workflow» ([docs: workflows](https://code.claude.com/docs/en/workflows)); бюджеты сессий в Managed Agents ([platform docs](https://platform.claude.com/docs/en/managed-agents/multiagent-orchestration)).
11. **Не включать флаг без нужды.**
    - С включённым флагом именованный субагент становится тиммейтом, команды «can form even when you didn't ask for one» — [docs](https://code.claude.com/docs/en/agent-teams).
    - Обход от мейнтейнера: не передавать `name` для фоновых задач по принципу «запустил и забыл» — [#71723](https://github.com/anthropics/claude-code/issues/71723).
12. **Масштаб усилия под сложность** (для исследований): простой поиск фактов — 1 агент и 3–10 вызовов инструментов; сравнение — 2–4 субагента по 10–15 вызовов; сложное исследование — больше 10 субагентов — [anthropic.com](https://www.anthropic.com/engineering/multi-agent-research-system).

### Интерпретация
- Почти все практики сводятся к одному: убрать то, чем команды отличаются от субагентов. Это самозахват задач, свободная переписка и общее редактирование. Предложение: если после этого остаётся только параллельный веер с отчётом, дешевле и надёжнее взять субагентов или workflow.

### Пробелы
- Не проверено, работает ли `--max-budget-usd` для интерактивной сессии с командой. В [docs: costs](https://code.claude.com/docs/en/costs) он упомянут без привязки к командам.

## 7. Вывод для недельного цикла LLM-компании

### Главное
Предложение: недельный цикл MVP на agent teams не переводить. Оставить субагентов и стадийные скиллы, как в ТЗ.

Из стадий выигрыш правдоподобен в одной — пакет совета, как опциональный ручной эксперимент «состязательного совета» перед гейтом 1 (3 тиммейта Sonnet, только чтение). Стадии-веера (скаут, турнир) лучше перевести на dynamic workflows или параллельных субагентов, а не на команду. Остальные стадии последовательные или детерминированные; команда там вредна.

### Факты (с источниками), значимые для ТЗ
- В ТЗ `/cycle` последовательно запускает стадии 2–7, каждая стадия — свой субагент, обмен только через файлы, состояние в git, у агентов нет токенов на запись, все денежные действия делает человек — [ТЗ MVP](/home/user/office_research/research/mvp-claude-code.md), разделы 0, 2, 5.
- Свойства команд против этих требований:
  - **Только интерактивная сессия.** Тиммейты не создаются в `-p` и SDK — [docs](https://code.claude.com/docs/en/agent-teams).
  - **Нет возобновления.** `/resume` не восстанавливает тиммейтов, одна команда на сессию — [docs](https://code.claude.com/docs/en/agent-teams#limitations).
  - **Компакция стирает команду.** Длинная сессия `/cycle` теряет её (#23620, воспроизведение примерно за 45 минут) — [#23620](https://github.com/anthropics/claude-code/issues/23620).
  - **Недетерминизм.** Самозахват без учёта ролей, дубликаты, повторная доставка — [#93668](https://github.com/anthropics/claude-code/issues/93668), [#55586](https://github.com/anthropics/claude-code/issues/55586), [#92244](https://github.com/anthropics/claude-code/issues/92244).
  - **Нагрузка на человека.** Запросы разрешений всплывают у лида; лид бывает «зависшим» до нажатия клавиши — [docs](https://code.claude.com/docs/en/agent-teams), [#26479](https://github.com/anthropics/claude-code/issues/26479), [#51959](https://github.com/anthropics/claude-code/issues/51959).
  - **Ролевая изоляция слабее, чем у субагентов.** In-process тиммейт игнорирует `mcpServers` и `skills` из определения и грузит MCP и скиллы проекта; ограничение держится только на списке `tools` — [docs](https://code.claude.com/docs/en/agent-teams#use-subagent-definitions-for-teammates). `effort` и определения из плагинов игнорируются — [#80569](https://github.com/anthropics/claude-code/issues/80569), [#78234](https://github.com/anthropics/claude-code/issues/78234).
  - **Флаг ломает субагентов.** Включённый флаг меняет поведение именованных субагентов директора — [docs](https://code.claude.com/docs/en/agent-teams), [#71723](https://github.com/anthropics/claude-code/issues/71723).
- Бюджет MVP — $13–27 в неделю (оценка) — [ТЗ MVP](/home/user/office_research/research/mvp-claude-code.md). Команда умножает токены затронутой стадии примерно в 3–7 раз по оценкам вендора и самоотчётам — [docs: costs](https://code.claude.com/docs/en/costs), [claude.com](https://claude.com/blog/building-multi-agent-systems-when-and-how-to-use-them), раздел 2. Сверху — 13–22% шума на лиде ([#47930](https://github.com/anthropics/claude-code/issues/47930)).

Оценка по стадиям (вся таблица — предложение):

| # | Стадия | Характер работы | Нужна ли переписка агентов | Команда? | Лучше | Почему |
|---|---|---|---|---|---|---|
| 2 | `/scout` (около 50 идей) | Веер с преобладанием чтения | Нет: дедупликация детерминированная (`registry.py`) | Нет | Параллельные субагенты или `pipeline()` в workflow по непересекающимся секторам поиска; объединение и дедупликация скриптом | Схема «читатели плюс один писатель» ([Cognition, по сниппету](https://cognition.ai/blog/multi-agents-working)); команды не изолируют файлы ([docs](https://code.claude.com/docs/en/agents)) |
| 3 | `/filter` | Правила плюс скрипт, Haiku | Нет | Нет | Как в ТЗ | Последовательная детерминированная логика: −39–70% у многоагентности на последовательных задачах ([Google, по сниппету](https://research.google/blog/towards-a-science-of-scaling-agent-systems-when-and-why-agent-systems-work/)) |
| 4 | `/data` | Вызовы Вордстата и прогнозов Директа с квотами | Нет | Нет | Как в ТЗ (последовательно, с учётом квот) | Параллельные агенты быстрее выбирают квоты; пользы от обсуждения нет |
| 5 | `/economics` | Скрипт `max_cpc.py` | Нет | Нет | Как в ТЗ | Числа считает скрипт |
| 6 | `/tournament` | Много независимых суждений по парам, A–B и B–A | Вредна: судьи должны быть независимы | Нет | `parallel()` в workflow или параллельные субагенты-судьи с одинаковой рубрикой | Переписка судей ломает независимость; дебаты однотипных моделей при равном бюджете не лучше self-consistency ([arXiv 2502.08788, по сниппету](https://arxiv.org/abs/2502.08788v3)) |
| 7 | `/board-pack` | Синтез плюс критика топ-3 перед гейтом 1 | Полезна в состязательном режиме | Опционально, как эксперимент | (а) один субагент red-team, (б) workflow со взаимной проверкой находок, (в) команда из 3 тиммейтов Sonnet: «за», «против», аудитор цифр; только чтение; финальный текст пишет лид | Единственная стадия, где сценарий «challenge each other» из документации совпадает с задачей. Доказательства пользы дебатов слабые, поэтому только A/B-эксперимент |
| 8–9 | `/landing`, `/smoke-plan` | Один писатель, последовательно | Нет | Нет | Как в ТЗ | Правка общих файлов и зависимые шаги — анти-сценарий по документации |
| 10 | `/monitor` (ежедневно) | Дешёвое чтение метрик | Нет | Нет | Как в ТЗ (Haiku, позже `-p`) | Команды не работают в `-p`, тиммейты тратят токены, пока живы |
| 11 | `/retro` | Один синтез | Нет | Нет | Как в ТЗ | — |

Условия эксперимента на стадии 7 (предложение):
- Отдельный профиль настроек с `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` и отдельная сессия. В основном репозитории флаг выключен, чтобы не ломать субагентов `/cycle`.
- Тиммейты только на чтение; каждый пишет свой файл `board-debate/<роль>.md`; `board-pack.md` пишет только лид.
- Хук `TaskCompleted` прогоняет `validate.py`.
- Задачи назначаются явно, не самозахватом.
- Потолок времени — около 30 минут, меньше окна, в котором компакция ломает команду.
- Сравнение с вариантом (а) на 2–3 циклах. Критерии принятия: человек оценивает пакет совета заметно выше; токены стадии растут не больше чем в 2 раза; ни одного сбоя из раздела 3.
- Оценка затрат: прирост недельного бюджета ≈ доля стадии в токенах × (k − 1), где k ≈ 3–7. Например, при доле 15% и k = 5 это около +60% к недельному бюджету (оценка; разбивки токенов по стадиям в ТЗ нет).

### Интерпретация
- Команды не дают MVP ничего, чего не дают субагенты или workflows, кроме свободной переписки. Зато ломают три опоры ТЗ: воспроизводимость стадий в git, возможность паузы и возобновления, строгую ролевую изоляцию.
- Пересмотреть вывод стоит, если выполнены все условия:
  - Anthropic выведет команды из experimental;
  - закрыты исправлением #23620, #47930 и #93668;
  - появится независимый замер качества на аналитических задачах.

### Пробелы
- Нет данных о доле токенов по стадиям MVP, поэтому прирост стоимости — только формула.
- Не проверена надёжность dynamic workflows на практике (только документация).
- Не проверено, как команды взаимодействуют с MCP Вордстата, Метрики и ЮKassa: in-process тиммейт грузит MCP проекта ([docs](https://code.claude.com/docs/en/agent-teams#use-subagent-definitions-for-teammates)), но сбои на практике не искались.

## 8. Источники

Anthropic (первичные, [вендор]):
- Building a C compiler with a team of parallel Claudes (5 февраля 2026) — https://www.anthropic.com/engineering/building-c-compiler
- Introducing Claude Opus 4.6 (5 февраля 2026) — https://www.anthropic.com/news/claude-opus-4-6
- How we built our multi-agent research system (13 июня 2025) — https://www.anthropic.com/engineering/multi-agent-research-system
- Building multi-agent systems: When and how to use them (23 января 2026) — https://claude.com/blog/building-multi-agent-systems-when-and-how-to-use-them
- How and when to use subagents in Claude Code (7 апреля 2026) — https://claude.com/blog/subagents-in-claude-code
- Bringing Code Review to Claude Code (9 марта 2026) — https://claude.com/blog/code-review
- Документация Claude Code:
  - agent teams — https://code.claude.com/docs/en/agent-teams
  - costs — https://code.claude.com/docs/en/costs
  - workflows — https://code.claude.com/docs/en/workflows
  - agents — https://code.claude.com/docs/en/agents
  - code review — https://code.claude.com/docs/en/code-review
- Managed Agents, multiagent orchestration (beta) — https://platform.claude.com/docs/en/managed-agents/multiagent-orchestration

GitHub (первичные; цифры в issues — [самоотчёт]):
- claudes-c-compiler:
  - список issues — https://github.com/anthropics/claudes-c-compiler/issues?q=is%3Aissue+sort%3Acomments-desc
  - #1 — https://github.com/anthropics/claudes-c-compiler/issues/1
  - #227 — https://github.com/anthropics/claudes-c-compiler/issues/227
- claude-code, поиски:
  - «teammate» в заголовке — https://github.com/anthropics/claude-code/issues?q=is%3Aissue+in%3Atitle+teammate+sort%3Acomments-desc
  - «agent team» в заголовке — https://github.com/anthropics/claude-code/issues?q=is%3Aissue+in%3Atitle+%22agent+team%22+sort%3Acomments-desc
  - открытые «agent teams» — https://github.com/anthropics/claude-code/issues?q=is%3Aissue+is%3Aopen+in%3Atitle+%22agent+teams%22+sort%3Areactions-%2B1-desc
- claude-code, issues (https://github.com/anthropics/claude-code/issues/<номер>):
  - #23620, #47930, #55586, #32996, #24414, #71723, #93668, #26479, #23983, #49865;
  - #38932, #51959, #50779, #34476, #60199, #85047, #92244, #77300, #90247;
  - #32368, #80569, #78234, #23669, #24316, #29207, #23572, #24301, #23815;
  - #23615, #24385, #27882, #24189, #24384, #26572, #98550, #98657, #42005, #60591.
- CHANGELOG (не прочитан, слишком большой) — https://github.com/anthropics/claude-code/blob/main/CHANGELOG.md
- Ruflo (бывший claude-flow) — https://github.com/ruvnet/ruflo

Независимые исследования (по сниппетам поиска):
- Towards a Science of Scaling Agent Systems:
  - https://research.google/blog/towards-a-science-of-scaling-agent-systems-when-and-why-agent-systems-work/
  - https://arxiv.org/abs/2512.08296v2
- Why Do Multi-Agent LLM Systems Fail? (MAST) — https://arxiv.org/html/2503.13657v3
- Stop Overvaluing Multi-Agent Debate — https://arxiv.org/abs/2502.08788v3
- Should we be going MAD? — https://arxiv.org/html/2311.17371v3

Практики, вендоры, СМИ (по сниппетам поиска):
- Cursor, Scaling agents:
  - https://cursor.com/blog/scaling-agents
  - https://www.zenml.io/llmops-database/scaling-multi-agent-autonomous-coding-systems
- Cognition:
  - https://cognition.ai/blog/multi-agents-working
  - https://agentic-ai.readthedocs.io/en/latest/ContextEngineering/devin/
- mejba.me (замер «один агент против команды») — https://www.mejba.me/claude-agent-teams-guide
- growwstacks:
  - https://growwstacks.com/blog/claude-agent-teams-vs-sub-agents/
  - https://growwstacks.com/blog/build-ai-agent-teams-with-claude-code-for-deep-market-research
  - https://growwstacks.com/blog/claude-agent-team-mistakes/
- Delegate mode — https://claudefa.st/blog/guide/agents/agent-teams-controls
- Gas Town:
  - https://www.drweb.de/ki-agenten-token-budget-feste-rollen/
  - https://ecosistemastartup.com/?p=108516
- Критика C-компилятора:
  - https://brianlovin.com/hn/46903616
  - https://lenz.io/c/e37520cc
  - https://www.letsdatascience.com/news/anthropic-researcher-builds-compiler-using-agent-teams-3ab2ffc4
- Ruflo в сравнениях:
  - https://codex.danielvaughan.com/2026/04/09/claude-multi-agent-ecosystem/
  - https://braindetox.kr/en/posts/ruflo_claude_code_multi_agent_2026.html
  - https://bernstein.readthedocs.io/en/latest/compare/bernstein-vs-claude-flow/
- Некодовые применения:
  - https://www.mindstudio.ai/blog/claude-code-agent-teams-brainstorming
  - https://calevans.com/blog/two-ai-agents-whose-only-job-is-to-kill-my-ideas.html
  - https://claudeskills.info/skills/jeffallan/claude-skills/the-fool/
  - https://lertechnotes.substack.com/p/validate-before-you-vibe-code-how
- Посты, видимые только по заголовкам:
  - https://addyosmani.com/blog/claude-code-agent-teams/
  - https://paddo.dev/blog/agent-teams-the-switch-got-flipped/
  - https://paddo.dev/blog/claude-code-hidden-swarm/
  - https://alexop.dev/posts/from-tasks-to-swarms-agent-teams-in-claude-code/
- Хабр:
  - https://habr.com/ru/posts/1010120
  - https://habr.com/ru/posts/986080

Внутренний контекст — ТЗ MVP, разделы 0, 2, 5, 6: /home/user/office_research/research/mvp-claude-code.md
