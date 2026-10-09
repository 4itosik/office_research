# Слой 3. Готовые инструменты, промпты и фреймворки для LLM-скаутинга, скоринга, турнира и смоук-теста бизнес-идей (build-vs-buy)

Срез на 2026-10-09. Контекст: соло-основатель в РФ строит «LLM-компанию» (LLM — директор, аналитик, маркетолог; человек — совет: бюджет, домены, юр. вопросы, финальный GO). Недельный цикл: (1) скаут собирает ~50 идей → (2) стоп-фильтр (в т.ч. монополист в нише) → (3) обогащение данными (Wordstat, прогноз бюджета Директа, ГИР БО) → (4) юнит-экономика (max CPC = цена × срок жизни × конверсия ÷ 3) → (5) турнир финалистов → (6) смоук-тест с лестницей бюджета (Директ + лендинг + предоплата через ЮKassa).

**Пометки.** «по сниппету поиска» — факт взят только из выдачи/резюме поисковика, сама страница не открывалась. «не проверено» — первичным источником подтвердить не удалось. «предложение» — мой синтез, источники этого не утверждают. Вердикты: **брать** / **референс** / **нет**.

**Ограничения среды (влияют на доказательность).** В этой сессии WebFetch открывал github.com, но не резолвил arxiv.org, ar5iv.labs.arxiv.org, research.google, forentrepreneurs.com, intercom.com, momtestbook.com, pretotyping.org (ошибка `ENOTFOUND`). curl к ним, а также к api.github.com, huggingface.co, openreview.net и api.semanticscholar.org прокси отклонял (403); работали только pypi.org и registry.npmjs.org. Поэтому факты о репозиториях взяты с первичных страниц GitHub/PyPI/npm, а всё о статьях (co-scientist, Si et al., MT-Bench, Wang et al.) и о фреймворках (Savoia, Skok, RICE, Mom Test, Lean Canvas) — по сниппетам поиска. Число контрибьюторов и дата создания репозиториев на страницах GitHub, которые отдавал WebFetch, не отображались: для всех кандидатов это «не проверено», если не указано иное.

---

## 1. Таблица кандидатов (код/инструменты, максимум 5)

| # | Кандидат | URL | Лицензия | Стек | Как подключить к Rails/Go | Активность на 2026-10-09 | Этапы цикла | Работает в РФ | Вердикт |
|---|---|---|---|---|---|---|---|---|---|
| 1 | **GPT Researcher** + MCP-сервер **gptr-mcp** | [gpt-researcher](https://github.com/assafelovic/gpt-researcher), [gptr-mcp](https://github.com/assafelovic/gptr-mcp) | Apache-2.0 по файлу [LICENSE](https://github.com/assafelovic/gpt-researcher/blob/main/LICENSE). ⚠ На [PyPI](https://pypi.org/pypi/gpt-researcher/json) у пакета классификатор MIT. gptr-mcp — MIT | Python ≥3.12, FastAPI-бэкенд + React-фронт, Docker; LLM через 27 провайдеров, в т.ч. `gigachat` | (а) отдельный сервис в docker-compose, HTTP к FastAPI на :8000; (б) gptr-mcp по SSE или streamable-HTTP, LLM-директор вызывает `deep_research` и `quick_search`; (в) свой поисковый адаптер на Rails/Go в роли custom retriever: `GET ?query=` → JSON `[{url, raw_content}]` | ★30.0k, 4.1k форков, 3 211 коммитов; последний коммит 2026-09-26; PyPI 0.16.1 от 2026-09-26, 111 релизов с 2024-04-12; контрибьюторы и дата создания — не проверено | 1 скаут (desk research), 3 обогащение (текстовая разведка конкурентов и монополиста), частично 2 | **Да, после настройки**: self-hosted; GigaChat есть в списке провайдеров; есть ретриверы SearxNG, DuckDuckGo и custom. Tavily (ретривер по умолчанию) — иностранный SaaS, оплата из РФ не проверена | **брать** |
| 2 | **Open Deep Research** (LangChain) | [open_deep_research](https://github.com/langchain-ai/open_deep_research) | MIT | Python, LangGraph, `init_chat_model()` | LangGraph server (`langgraph dev`, API на 127.0.0.1:2024) — по HTTP | ★12.7k, 1.9k форков, 224 коммита; **архивирован 2026-08-21**; последний коммит 2026-08-10 (dependabot); преемник не указан | 1, 3 | Self-hosted. Tavily по умолчанию; провайдеры OpenAI, Anthropic, OpenRouter, Ollama | **референс** (архив) |
| 3 | **idea-validation-agents** (MaxKmet) | [repo](https://github.com/MaxKmet/idea-validation-agents) | MIT | Только Markdown-промпты и скиллы для Claude Code, Codex и Cursor (`CLAUDE.md`, `AGENTS.md`, `skills/`, `workflows/`, `memory/`) | Рантайма нет: промпты и рубрики переносятся в свои шаблоны | ★478, 57 форков, 9 коммитов с 2026-04-13 по 2026-06-16; минимум 2 автора; ⚠ высокая популярность при 9 коммитах | 1 генерация, 2 насыщенность рынка (в т.ч. incumbent dominance), 4 цена и CAC, 6 RAT (≤2 недель, ≤$100), итоговый скор | Промпты переносимы. Источники данных (TikTok CC, Reddit, X, App Store, Google Trends) без Яндекса | **референс** |
| 4 | **AI-Researcher** (Stanford: Si, Yang, Hashimoto) | [repo](https://github.com/NoviScl/AI-Researcher) | MIT | Python; клиенты OpenAI и Anthropic; sentence-transformers | Портировать алгоритм [`tournament_ranking.py`](https://github.com/NoviScl/AI-Researcher/blob/main/ai_researcher/src/tournament_ranking.py) на Ruby/Go (небольшая логика) | ★410, 40 форков, 192 коммита; последний коммит 2025-08-07, больше года без активности | 1 генерация + дедупликация, 2 фильтр новизны, **5 Swiss-турнир** | Алгоритм от домена не зависит. Промпты заточены под NLP-статьи; лит-обзор через Semantic Scholar для ниш РФ нерелевантен | **референс** |
| 5 | **The AI Scientist** (Sakana AI) | [repo](https://github.com/SakanaAI/AI-Scientist) | ⚠ «The AI Scientist Source Code License» (производная от Responsible AI License). Коммит 2025-12-19: «Update license from Apache 2.0 to AI Scientist License 1.0» | Python | Только паттерн промптов; код не брать | ★14.7k, 2.1k форков, 98 коммитов; последний коммит 2025-12-19 (смена лицензии), предыдущий 2025-04-26 | 1 генерация + самооценка 1–10, проверка новизны | Semantic Scholar и OpenAlex — научный домен | **референс** (паттерн); код — **нет** |

Источники к таблице: страницы репозиториев и коммитов: [gpt-researcher](https://github.com/assafelovic/gpt-researcher), [его коммиты](https://github.com/assafelovic/gpt-researcher/commits), [ретриверы](https://github.com/assafelovic/gpt-researcher/tree/main/gpt_researcher/retrievers), [провайдеры LLM](https://github.com/assafelovic/gpt-researcher/blob/main/gpt_researcher/llm_provider/generic/base.py), [gptr-mcp](https://github.com/assafelovic/gptr-mcp), [open_deep_research](https://github.com/langchain-ai/open_deep_research), [его коммиты](https://github.com/langchain-ai/open_deep_research/commits), [idea-validation-agents, коммиты](https://github.com/MaxKmet/idea-validation-agents/commits), [AI-Researcher, коммиты](https://github.com/NoviScl/AI-Researcher/commits), [AI-Scientist, коммиты](https://github.com/SakanaAI/AI-Scientist/commits).

### Также рассмотрено, в топ-5 не вошло

| Проект | Факты (на 2026-10-09) | Вердикт |
|---|---|---|
| [a-canary/llm-judge](https://github.com/a-canary/llm-judge) | MIT, ★0, 0 форков, 37 коммитов; [npm](https://registry.npmjs.org/llm-judge) 1.0.0 опубликован 2026-05-07. Режим Swiss (Monrad), 3 раунда; старт Elo 1500, K=32; отсечение top-K (`--elo-rank K`, `--elo-class K`); кэш вердиктов на 512 записей; судья — любой OpenAI-совместимый URL или CLI `claude`. Перестановка пары для борьбы с позиционным смещением не описана. ⚠ 0 звёзд | **референс** алгоритма |
| [lmarena/arena-hard-auto](https://github.com/lmarena/arena-hard-auto) | Apache-2.0, ★1.1k, 161 форк, 182 коммита. Судьи GPT-4.1 и Gemini-2.5. «Style control» по длине и markdown (`--control-features`, `-f length markdown`); bootstrap-доверительные интервалы | **референс** (как подавлять смещение к многословию) |
| [The-Swarm-Corporation/AI-CoScientist](https://github.com/The-Swarm-Corporation/AI-CoScientist) | MIT, ★131, 32 форка, 19 коммитов; [PyPI `ai-coscientist`](https://pypi.org/pypi/ai-coscientist/json) — единственный релиз 1.0.0 от 2025-07-11. Построен на Swarms; Elo по попарным сравнениям; в TODO «Improve Elo rating». Оценка здоровья зависимостей 47/100 ([depscope](https://mcp.depscope.dev/pkg/pypi/ai-coscientist), по сниппету поиска) | **нет** |
| [mnemox-ai/idea-reality-mcp](https://github.com/mnemox-ai/idea-reality-mcp) | MIT, ★822, 89 форков, 193 коммита; README: «Maintenance mode», новых функций не планируется. Один MCP-инструмент `idea_check` возвращает «reality score» 0–100. Источники: GitHub, HN, npm, PyPI, Stack Overflow; Product Hunt удалён 2026-07-17. Ключи не нужны. ⚠ README пишет «290+ stars» при 822 на странице | **нет**: оценивает конкуренцию только среди dev-продуктов, не ниши РФ |
| [Contextualist/lone-arena](https://github.com/Contextualist/lone-arena) | Турнир на выбывание по 8 ответам, затем MLE-Elo (по сниппету поиска); лицензия и активность не проверены | **референс** |
| [i-Eval/FairEval](https://github.com/i-Eval/FairEval) | Код к статье Wang et al.: флаги BPC и MEC (по сниппету поиска) | **референс** |
| idea-sieve ([dev.to](https://dev.to/kzeitar/building-an-ai-powered-saas-app-etc-idea-validation-system-bpg)), crewhaus validation MCP ([getdrio](https://www.getdrio.com/mcp/io-github-crewhaus-validation/md)), arkhe startup-validating ([skillselion](https://skillselion.com/skills/joaquimscosta/arkhe-claude-plugins/startup-validating)), VentureForge ([lablab.ai](https://lablab.ai/submissions/sj2ld96jr7uh9lnukrqz16f7)) | Только по сниппету поиска: idea-sieve — монорепо на TypeScript; crewhaus — npm-пакет `crewhaus-mcp-server`, лицензия на странице реестра не указана; arkhe — 6-стадийная воронка, ~21★; VentureForge — хакатонный проект из 6 агентов | не проверено / **нет** |
| Google AI co-scientist | Открытого кода нет; по сниппетам поиска доступ давали через Trusted Tester Program для исследовательских организаций ([itdaily](https://itdaily.com/news/software/google-ai-co-scientist), [eWeek](https://www.eweek.com/fr/news/google-ai-scientist/)) | **референс** по статье |

---

## 2. Deep-dive топ-3

### 2.1 GPT Researcher (+ gptr-mcp): брать как движок «скаута» и desk research

**Факты**
- Масштаб: ★30.0k, 4.1k форков, 3 211 коммитов в `main`. Docker: `docker-compose up --build` поднимает Python-сервер на localhost:8000 и React на localhost:3000 — [README](https://github.com/assafelovic/gpt-researcher).
- Активность: пять последних коммитов датированы 2026-09-26 (правки документации, часть сделана в соавторстве с «claude») — [commits](https://github.com/assafelovic/gpt-researcher/commits). На PyPI версия 0.16.1 загружена 2026-09-26; первая версия на PyPI (0.2.2) — 2024-04-12; всего 111 релизов — [PyPI JSON](https://pypi.org/pypi/gpt-researcher/json).
- Лицензия: файл LICENSE — «Apache License, Version 2.0, January 2004» ([LICENSE](https://github.com/assafelovic/gpt-researcher/blob/main/LICENSE)). В README сказано «sharing codes for academic purposes under the Apache 2 license» ([README](https://github.com/assafelovic/gpt-researcher)). При этом метаданные PyPI указывают «License :: OSI Approved :: MIT License» ([PyPI JSON](https://pypi.org/pypi/gpt-researcher/json)). ⚠ Расхождение; обе лицензии пермиссивные.
- Режим Deep Research в README: «an advanced recursive research workflow» — дерево с настраиваемыми глубиной и шириной, параллельное выполнение, около 5 минут и около $0.4 за прогон на o3-mini с high reasoning — [README](https://github.com/assafelovic/gpt-researcher).
- Локальные документы подключаются через `DOC_PATH`: PDF, txt, CSV, Excel, Markdown, PowerPoint, Word — [README](https://github.com/assafelovic/gpt-researcher).
- Ретриверы (21 каталог): arxiv, bing, bocha, brave, crw, custom, duckduckgo, exa, getxapi, google, groundroute, mcp, openalex, pubmed_central, searchapi, searx, semantic_scholar, serpapi, serper, tavily, xquik. Ретривера для Яндекса нет — [retrievers](https://github.com/assafelovic/gpt-researcher/tree/main/gpt_researcher/retrievers).
- Контракт custom retriever. Эндпоинт задаётся в `RETRIEVER_ENDPOINT`. Переменные окружения с префиксом `RETRIEVER_ARG_` превращаются в параметры запроса. Запрос — GET с `query`, таймаут 20 с. Ответ — JSON-список объектов с `url` (запасной ключ `href`) и `raw_content` (запасной `body`). При ошибке ретривер возвращает пустой список; `max_results` пока не используется — [custom.py](https://github.com/assafelovic/gpt-researcher/blob/main/gpt_researcher/retrievers/custom/custom.py).
- Провайдеры LLM (27): openai, anthropic, azure_openai, cohere, google_vertexai, google_genai, fireworks, ollama, together, mistralai, huggingface, groq, bedrock, dashscope, xai, deepseek, litellm, **gigachat**, openrouter, vllm_openai, aimlapi, netmind, forge, avian, minimax, atlascloud, nebius. Провайдера «yandex» нет — [base.py](https://github.com/assafelovic/gpt-researcher/blob/main/gpt_researcher/llm_provider/generic/base.py). OpenAI-совместимые эндпоинты подключаются через `OPENAI_BASE_URL` — [README](https://github.com/assafelovic/gpt-researcher).
- gptr-mcp: MIT, ★372, 66 форков, 37 коммитов. Инструменты: `deep_research`, `quick_search`, `write_report`, `get_research_sources`, `get_research_context`. Транспорт: stdio, SSE (в Docker включается автоматически), streamable HTTP; переключение через `MCP_TRANSPORT`. Есть Dockerfile и docker-compose, сервер слушает 0.0.0.0:8000. В инструкции для Claude Desktop нужны и `OPENAI_API_KEY`, и `TAVILY_API_KEY` — [gptr-mcp](https://github.com/assafelovic/gptr-mcp).

**Как встроить (предложение)**
- Поднять gpt-researcher и gptr-mcp как sidecar-сервисы в Docker. Rails/Go-приложение и LLM-директор обращаются к ним по MCP (streamable HTTP) или через HTTP к FastAPI. Формальный REST/WebSocket-контракт FastAPI не проверен: README ссылается на внешнюю документацию.
- Написать в Rails/Go эндпоинт-адаптер для custom retriever (`GET /search?query=` → `[{url, raw_content}]`) поверх выбранного RU-поиска. Доступность и тарифы Yandex Search API — тема другого слоя, здесь не проверено. Тогда Tavily не нужен.
- LLM: `gigachat` из коробки, остальные — через `OPENAI_BASE_URL`, `litellm` или `openrouter`. OpenAI-совместимость YandexGPT здесь не проверена.
- Роль в цикле: для каждой из ~50 идей запускать `quick_search`, для финалистов — `deep_research` с вопросами «кто лидер ниши, есть ли монополист, какие цены у конкурентов». Количественные данные (Wordstat, прогноз Директа, ГИР БО) gpt-researcher не даёт: их собирают свои адаптеры из других слоёв.

**Риски.** Качество на русскоязычных запросах и с RU-поиском не проверено. Конфликт лицензий в метаданных (Apache-2.0 против MIT). Высокая скорость изменений: 111 релизов за ~2.5 года на PyPI, поэтому лучше пиновать версию (предложение).

### 2.2 AI-Researcher: Swiss-турнир идей, референс для этапа 5

**Факты о методе из статьи** (Si, Yang, Hashimoto, «Can LLMs Generate Novel Research Ideas?», ICLR 2025; [arXiv 2409.04109](https://arxiv.org/pdf/2409.04109), [ICLR](https://iclr.cc/virtual/2025/poster/29961); всё по сниппету поиска):
- Попарные сравнения организованы как Swiss-турнир: пары составляются из предложений с близким накопленным счётом, победитель получает очко, после N раундов счёт лежит в диапазоне [0, N].
- Валидация на ICLR: разрыв средних оценок ICLR между топ-10 и низ-10 по ранжированию LLM рос с 0.56 при N=1 до 1.73 при N=5 и упал до 1.30 при N=6.
- Выбор ранжировщика: в сравнении GPT-4o показал 61.1%, Claude-3-Opus — 63.5%; few-shot и chain-of-thought значимого прироста не дали. Выбран zero-shot Claude-3.5-Sonnet. Его собственную точность (встречается цифра 71.4%) подтвердить не удалось — не проверено.
- Идеи LLM оценены как более новые (p < 0.05), но чуть слабее по реализуемости. Авторы отмечают «failures of LLM self-evaluation» и «lack of diversity in generation».

**Факты о коде** ([tournament_ranking.py](https://github.com/NoviScl/AI-Researcher/blob/main/ai_researcher/src/tournament_ranking.py)):
- Пары. Функция `single_round()` перемешивает идеи только в раунде 0. Каждый раунд идеи сортируются по счёту и спариваются соседи (0–1, 2–3, …). При нечётном числе последняя идея получает bye (+1).
- Раунды: `max_round=5` по умолчанию; досрочной остановки нет.
- Промпт судьи (`better_idea()`): судья — рецензент NLP/LLM-статей, которому сообщают, что одна из двух статей принята на топ-конференцию, а другая отклонена. Ответ — только «1» или «2».
- **Позиционное смещение не подавляется**: пару судят один раз, без перестановки. Идея с большим счётом после сортировки чаще стоит первой. Любой ответ кроме «1», в том числе мусорный, засчитывается второй идее.
- Счёт хранится по ключу из первых 200 символов текста плана, поэтому идеи с одинаковым началом могут совпасть. Ретраи: 3 попытки с задержкой 2 с. Судья по умолчанию `gpt-4-1106-preview`, temperature 0. Ответы API не кэшируются; топ-10 сохраняется в `top_ideas.json`.
- Остальной конвейер ([README](https://github.com/NoviScl/AI-Researcher)): дедупликация по косинусной близости эмбеддингов с порогом 0.8 (sentence-transformers, без API); генерация батчами по 5; фильтр новизны отсекает предложение, если LLM считает его тем же, что и найденная статья. Цена демо — $0.74 за ранжирование 10 предложений в 5 раундах.

**Что взять (предложение).** Сам Swiss-алгоритм (около 100 строк на Ruby/Go) плюс исправления: два прогона с перестановкой и ничья при расхождении (см. раздел 4), ключ по ID вместо префикса текста, явная обработка невалидного ответа, кэш вердиктов (как в llm-judge), судья из другого семейства моделей, чем генератор.

### 2.3 idea-validation-agents: референс рубрик для стоп-фильтра и скоринга

**Факты** ([README](https://github.com/MaxKmet/idea-validation-agents), [commits](https://github.com/MaxKmet/idea-validation-agents/commits)):
- MIT, ★478, 57 форков, 9 коммитов: первый 2026-04-13, последний 2026-06-16 (мерж PR от kriptoburak, который добавил X/Twitter как источник трендов).
- Итоговый скор 0–100 считается «multiplicative-floor algorithm»: «one catastrophic weakness kills the overall score, just like in a real startup».
- Насыщенность рынка оценивается по 5 факторам: число конкурентов, **incumbent dominance** (доминирование инкумбента), активность финансирования, насыщенность ключевых слов, насыщенность контента.
- Цена: анализ Van Westendorp и множители «desire premium» («survival/status desires command 1.3–2× price premium»).
- Дистрибуция: вирусный k-фактор по 6 типам петель, ASO-скор по 5 факторам, creator-economy fit.
- Бенчмарки SOM, например «niche productivity: 0.5–2.0% year 1».
- Riskiest Assumption Test: «≤2-week, ≤$100 behavioral experiment». Пре-мортем по Klein (2007) с горизонтом 12 месяцев.
- Правила пивота: менять 1–2 переменные. Слабости делятся на structural, situational, knowledge-gap и addressable; пивоты порождают только две последние.
- Kill criteria есть как раздел мемо, но **без числовых порогов**. Вердикты: pursue, test, pivot, drop.
- Источники данных: TikTok Creative Center, Reddit, X/Twitter, App Store, Google Trends.
- ⚠ Аномалия: агрегаторы показывали другие звёзды (427★/47 форков на [sourcepulse](https://www.sourcepulse.org/projects/30847528), 260★ на [gittrend](https://gittrend.io/repo/MaxKmet/idea-validation-agents); по сниппету поиска). Популярность пришла через README-мем при 9 коммитах; это набор промптов, а не протестированная система.

**Что взять (предложение).**
- Мультипликативный «пол» как логику стоп-фильтра: один фатальный фактор обнуляет идею.
- Фактор «incumbent dominance» — прямой аналог стоп-фактора «монополист».
- Формат решения pursue/test/pivot/drop для совета.
- Ограничение RAT (≤2 недель, ≤$100) как шаблон первой ступени лестницы смоук-теста.
- Источники данных заменить на Wordstat, Директ и ГИР БО.

---

## 3. Вопрос 1. Open-source агенты и репозитории для генерации, валидации и скоринга идей; ресёрч-агенты в роли «скаута»

### Takeaway
Готовым стоит брать только **GPT Researcher** (+gptr-mcp). Он активен (коммиты и релиз 2026-09-26), пермиссивно лицензирован, self-hosted, отдаёт MCP и HTTP, принимает свой поисковик через custom retriever и умеет GigaChat. **Open Deep Research архивирован 2026-08-21**. Специализированные «валидаторы стартап-идей» на GitHub — это молодые наборы промптов (idea-validation-agents) или хакатонные проекты, и ни один не работает с данными Яндекса. Поэтому они годятся только как референс рубрик.

### Cited Findings
- GPT Researcher — все факты в разделе 2.1 ([repo](https://github.com/assafelovic/gpt-researcher), [PyPI](https://pypi.org/pypi/gpt-researcher/json), [gptr-mcp](https://github.com/assafelovic/gptr-mcp)).
- Open Deep Research — [repo](https://github.com/langchain-ai/open_deep_research), [commits](https://github.com/langchain-ai/open_deep_research/commits):
  - Баннер «This repository was archived by the owner on Aug 21, 2026. It is now read-only». Последние коммиты — обновления зависимостей от dependabot (2026-08-10, 2026-08-05, 2026-07-25, 2026-07-17). Преемник не назван.
  - Поиск: Tavily по умолчанию; README: «Has full MCP compatibility and work native web search for Anthropic and OpenAI».
  - Провайдеры через `init_chat_model()`: OpenAI (по умолчанию), Anthropic, OpenRouter, Ollama. Модели должны поддерживать structured outputs и tool calling.
  - Legacy-реализации лежат в `src/legacy/`: «Supervisor-Researcher Architecture» и plan-and-execute с human-in-the-loop планированием.
  - Deep Research Bench (RACE): GPT-5 — 0.4943, Claude Sonnet 4 — 0.4401, по умолчанию — 0.4309; конфигурация сабмита — 0.4344, «#6 on the leaderboard (August 2, 2025)».
  - Пакет `open-deep-research` на PyPI: 0.0.1 от 2025-02-19, 0.0.16 от 2025-07-16 ([PyPI JSON](https://pypi.org/pypi/open-deep-research/json)). Ссылки на репозиторий в метаданных нет, связь с LangChain — не проверено.
- The AI Scientist (Sakana) — [README](https://github.com/SakanaAI/AI-Scientist), [generate_ideas.py](https://github.com/SakanaAI/AI-Scientist/blob/main/ai_scientist/generate_ideas.py), [commits](https://github.com/SakanaAI/AI-Scientist/commits):
  - Идея — JSON с полями Name, Title, Experiment, Interestingness, Feasibility, Novelty; три последних — «A rating from 1 to 10 (lowest to highest)», без якорей шкалы. Промпт требует «cautious and realistic on your ratings».
  - Рефлексия: до `num_reflections=5` раундов, досрочный выход по фразе «I am done».
  - Проверка новизны: до `max_num_iterations=10` раундов поиска в Semantic Scholar (по 10 результатов с абстрактами), запасной вариант — OpenAlex. Решение принимается строкой «Decision made: novel.» или «Decision made: not novel.». Если строка не появилась, идея считается не новой.
  - Отдельная функция `perform_review` выдаёт оценку 1–10 и Accept/Reject.
  - Лицензия сменена с Apache 2.0 на «AI Scientist License 1.0» коммитом от 2025-12-19.
- Остальные кандидаты (idea-reality-mcp, crewhaus, idea-sieve, VentureForge, arkhe) — см. таблицу «Также рассмотрено».

### Inferences
- Для этапа (1) «скаут ~50 идей» готовый «генератор бизнес-идей», применимый к РФ, не найден. Разумная схема — генерация промптами в стиле AI Scientist (JSON с полями и рефлексией), обязательная дедупликация (порог 0.8 по эмбеддингам, как в AI-Researcher) и desk research через GPT Researcher (предложение). Si et al. прямо отмечают у LLM «lack of diversity in generation», так что без дедупликации ~50 идей будут сильно повторяться.
- Самооценки 1–10 без якорей (AI Scientist) и «failures of LLM self-evaluation» (Si et al.) говорят о том, что абсолютные баллы от LLM ненадёжны. Для финального выбора лучше попарный турнир, а абсолютные баллы — только для грубого отсева (вывод из источников выше).
- Ruby/Rails-нативных реализаций ни в одной из категорий не найдено (поиск по GitHub через WebSearch выдал только Python и TypeScript).

### Gaps
- Число контрибьюторов и даты создания репозиториев (кроме idea-validation-agents, где первый коммит 2026-04-13) не проверены: на страницах GitHub не отображались, API GitHub недоступен.
- Не проверены REST/WebSocket-контракт FastAPI у GPT Researcher, качество работы на русском и стоимость прогонов на GigaChat.
- CrewAI/LangGraph-примеры «startup idea validator» глубже не исследовались: поиск выдал только перечисленные проекты.

---

## 4. Вопрос 2. Турнир и попарное ранжирование идей LLM-судьями; известные смещения и их подавление

### Takeaway
Есть две проверенные схемы: **Elo-турнир с дебатами для лидеров** (Google co-scientist) и **Swiss-турнир на N раундов** (Si et al., код в AI-Researcher). Обе опираются на попарные сравнения LLM-судьёй. LLM-судьи систематически смещены: к позиции, к многословию, к собственным ответам. Стандартная защита — судить пару дважды с перестановкой и ставить ничью при расхождении (MT-Bench) или агрегировать вердикты по порядкам (BPC, Wang et al.), а также контролировать длину (style control в arena-hard-auto). Готовой open-source библиотеки «турнир идей с защитой от смещений» уровня продакшена нет. Турнир проще написать самому, взяв за образец эти референсы.

### Cited Findings
**Google AI co-scientist** ([arXiv 2502.18864](https://arxiv.org/pdf/2502.18864), [блог Google Research](https://research.google/blog/accelerating-scientific-breakthroughs-with-an-ai-co-scientist/); всё по сниппету поиска):
- Ranking agent: «orchestrates an Elo-based tournament to automatically evaluate and rank all hypotheses, providing supporting rationale».
- Стартовый рейтинг: «We set the initial Elo rating of 1200 for the newly added hypothesis». Второй поиск эту цифру не подтвердил, в целом она подтверждена только сниппетом.
- Лидеры рейтинга сравниваются в «multi-turn scientific debates», остальные — в «single-turn comparisons». По сниппету, формат дебатов помогает против смещения от порядка.
- Похожие гипотезы чаще встречаются друг с другом; новые и лидирующие гипотезы получают приоритет на матчи.
- В промпте дебатов «typically ranging from 3 to 5» ходов, «with a maximum of 10».
- Авторы признают ограничения Elo, но считают его «good proxy» для относительного ранжирования. В блоге предпочтения экспертов названы «concordant» с Elo-метрикой (выборка — 11 исследовательских целей).
- K-фактор и полное расписание матчей найти не удалось. Вторичное описание промптов: [learnprompting](https://learnprompting.org/blog/google-ai-co-scientist-prompts).
- Доступ через Trusted Tester Program, позже — платформа «Gemini for Science» через Google Labs (по сниппету поиска: [itdaily](https://itdaily.com/news/software/google-ai-co-scientist), [educationawards.ie](https://educationawards.ie/news/google-launches-gemini-for-science-platform-to-support-ai-assisted-research-across-academic-and-enterprise-settings)). Открытого кода нет.

**Si et al. (ICLR 2025)** — Swiss-турнир, цифры валидации и код: см. раздел 2.2 ([arXiv](https://arxiv.org/pdf/2409.04109), [код](https://github.com/NoviScl/AI-Researcher/blob/main/ai_researcher/src/tournament_ranking.py)).

**Смещения LLM-судей: MT-Bench** (Zheng et al., «Judging LLM-as-a-Judge with MT-Bench and Chatbot Arena», NeurIPS 2023; [arXiv](https://arxiv.org/pdf/2306.05685), [HTML v4](https://arxiv.org/html/2306.05685v4), [NeurIPS](https://neurips.cc/virtual/2023/poster/73434); по сниппету поиска):
- Изучены position bias, verbosity bias, self-enhancement bias и ограниченность рассуждений. «All of them exhibit strong position bias», большинство судей предпочитает первую позицию.
- Согласованность после перестановки ответов: GPT-4 — 65.0% с промптом по умолчанию и 66.2% с переименованными ассистентами. Claude-v1 согласован лишь в 23.8% случаев и в 75.0% выбирает первый ответ.
- Консервативная защита: судить пару дважды в обоих порядках и засчитывать победу, только если ответ предпочтён оба раза; «If the results are inconsistent after swapping, we can call it a tie». Формулировка подтверждена вторичным разбором ([tailoredai](https://tailoredai.substack.com/p/judging-llm-as-a-judge-with-mt-bench)).
- Атака «repetitive list» на многословие построена на 23 ответах MT-bench с нумерованными списками. По вторичному источнику, GPT-3.5 и Claude-v1 обмануты примерно в 91% случаев. Цифра 8.7% для GPT-4 — не проверено.
- Ошибки судьи на 10 математических вопросах: 14/20 с промптом по умолчанию, 6/20 с chain-of-thought, 3/20 с референсным ответом.

**Wang et al., «Large Language Models are not Fair Evaluators»** (2023, ACL 2024; [arXiv](https://arxiv.org/pdf/2305.17926), [ACL](https://preview.aclanthology.org/setup/2024.acl-long.511), [FairEval](https://github.com/i-Eval/FairEval); по сниппету поиска):
- Одной перестановкой ответов Vicuna-13B «побеждает» ChatGPT в 66 из 80 запросов, когда судьёй выступает ChatGPT.
- Предложены Balanced Position Calibration (агрегация по разным порядкам) и Multiple Evidence Calibration (судья сначала выписывает несколько доводов, потом ставит оценку). В версии v2 добавлен human-in-the-loop с метрикой BPDE.

**Устойчивость рейтингов и расписание матчей** (по сниппету поиска):
- Daynauth et al.: классический Elo для LLM «often produce[s] inconsistent and unstable rankings»; авторы советуют сравнивать с альтернативами вроде Bradley-Terry — [arXiv 2411.14483](https://arxiv.org/html/2411.14483v2).
- Yoon et al.: случайное расписание матчей работает «surprisingly well», Swiss тоже силён в большинстве настроек — [arXiv 2502.15018](https://arxiv.org/html/2502.15018v1).
- Chatbot Arena сначала подбирал пары по предварительному ранжированию, потом перешёл на равномерную выборку ради покрытия — [LMSYS blog, 2023-05-03](https://www.lmsys.org/blog/2023-05-03-arena).

**Open-source реализации** (детали в таблице «Также рассмотрено»):
- arena-hard-auto — style control по длине и markdown, bootstrap-ДИ ([repo](https://github.com/lmarena/arena-hard-auto)).
- llm-judge — Swiss (Monrad) + Elo 1500, K=32, отсечение top-K ([repo](https://github.com/a-canary/llm-judge)).
- AI-CoScientist (Swarms) — сырой ([repo](https://github.com/The-Swarm-Corporation/AI-CoScientist)).
- AI-Researcher — Swiss без перестановок ([код](https://github.com/NoviScl/AI-Researcher/blob/main/ai_researcher/src/tournament_ranking.py)).

### Inferences
- **Схема турнира финалистов (предложение)**:
  - 8–12 финалистов после стоп-фильтра и юнит-экономики; 4–5 раундов Swiss. У Si et al. разделение топа и низа росло до N=5 и падало при N=6.
  - Каждый матч — два вызова судьи с перестановкой: ничья (0.5) при расхождении, иначе 1/0 (MT-Bench, BPC).
  - Судья — модель другого семейства, чем генератор идей, против self-enhancement.
  - Карточки идей одинаковой длины по фиксированному шаблону с числами (частотность, прогнозный CPC, max CPC, маржа). Это защита от многословия: аналог style control.
  - Для топ-2/3 — многоходовые дебаты, как у co-scientist.
  - Итог — сумма очков. Elo со стартом 1200 нужен, только если хранить рейтинг между неделями. При малом N лучше разовая MLE/Bradley-Terry-оценка с bootstrap-интервалом, а не последовательный Elo (Daynauth et al., arena-hard-auto).
- Порядок затрат: демо AI-Researcher — $0.74 за 10 предложений × 5 раундов на моделях того времени ([README](https://github.com/NoviScl/AI-Researcher)). С перестановками вызовов вдвое больше. Это копейки по сравнению с бюджетом смоук-теста (вывод).
- Писать самому на Ruby/Go дешевле, чем тянуть Python-зависимость: логика Swiss + перестановка + кэш — сотни строк, а готовые реализации либо без защиты от смещений (AI-Researcher), либо без пользователей (llm-judge, 0★) (предложение).

### Gaps
- Полные тексты co-scientist, Si et al., MT-Bench и Wang et al. открыть не удалось (arxiv.org недоступен). Не проверены K-фактор и расписание co-scientist, точность Claude-3.5-Sonnet у Si et al. и цифра 8.7% для GPT-4.
- Исследований о турнирах именно бизнес-идей (не научных гипотез) с проверкой на реальных продажах не найдено. Перенос выводов из научной области — допущение.

---

## 5. Фреймворки и формулы (референсы)

### Takeaway
Методическая база смоук-теста — **pretotyping Савойи**: XYZ-гипотеза задаёт порог успеха заранее, «skin in the game» означает, что деньги весомее слов, плюс fake door / painted door. Предоплата через ЮKassa — самый сильный вариант такого теста. Правило **LTV:CAC ≥ 3** (Skok) действительно стоит за «÷ 3» в формуле основателя, но у Skok LTV считается **по валовой марже**, а не по цене. Поэтому формула основателя при марже меньше 100% завышает допустимый CPC. Российская практика проверки ниши через Wordstat и прогноз Директа описана в статьях агентств (ppc.world, kokoc, vc.ru). Формализованного списка стоп-факторов с порогами у практиков не найдено. Единственный «официальный» стоп-сигнал — статус Директа «Мало показов».

### Cited Findings

#### Pretotyping (Alberto Savoia: «Pretotype It», 2011; «The Right It», 2019)
- XYZ-гипотеза: «At least X% of Y will do Z», где X — доля целевого рынка, Y — рынок, Z — действие. Пример: «At least 20% of packaged-sushi eaters will try Second-Day Sushi if it's half the price…». Её локальная версия «xyz» проверяется за часы или дни (по сниппету поиска: [Shortform](https://www.shortform.com/pdf/the-right-it-pdf-alberto-savoia), [Product Compass](https://res.productcompass.pm/top-product-management-books/the-right-it)).
- Техники претотипирования: Fake Door, Mechanical Turk (функцию тайно выполняет человек), Pinocchio (неработающий макет). Цель — проверить спрос, а не технологию (по сниппету поиска: [Stanford eCorner, доклад Savoia](https://stvp.stanford.edu/av/build-right-it-entire-talk), [Helio/ZURB](https://helio.zurb.com/blog/pretotype-your-way-to-product-success-validate-market-demand-fast/), [интервью Savoia в Product Compass](https://www.productcompass.pm/p/how-to-build-the-right-product-with)).
- «Skin in the game»: время, деньги или репутация — единственный надёжный сигнал. Измерять поведение, а не мнения. Предзаказы и листы ожидания — тесты готовности платить. Это вторичные гайды (по сниппету поиска: [skillselion pm-skills](https://skillselion.com/skills/phuryn/pm-skills/brainstorm-experiments-new)).
- Определение pretotyping: «a set of techniques and metrics to help you make sure that you are building "The Right It" before you build "It" right» (по сниппету поиска: [jamasoftware](https://www.jamasoftware.com/?p=8756), [nextview.vc](https://nextview.vc/blog/pretotyping-product-market-fit-google-alberto-savoia/)).
- **«Initial Level of Interest» — не проверено**: два целевых поиска не нашли этого термина ни в одном источнике. Ближайшее найденное — «testing the initial appeal and actual usage».

#### Fake door / painted door / pre-order / smoke test
- Painted door test, он же fake door test: показывают ещё не существующую функцию или продукт и измеряют долю кликнувших; название идёт от «нарисованной двери на стене» (по сниппету поиска: [Optimizely glossary](https://www.optimizely.com/optimization-glossary/painted-door-test/), [Amplitude](https://amplitude.com/explore/experiment/painted-door-testing.md)).
- Более сильные сигналы — оставленный e-mail или проход по симуляции покупки (по сниппету поиска: [Amplitude](https://amplitude.com/explore/experiment/painted-door-testing.md)).
- Первоисточников по «smoke test» и «pre-order validation» в продуктовом смысле поиск не дал (см. Gaps).

#### Lean Canvas (Ash Maurya, «Running Lean»)
- 9 блоков: problem, customer segments, unique value proposition, solution, channels, revenue streams, cost structure, key metrics, unfair advantage. Problem, Solution, Key Metrics и Unfair Advantage заменили «корпоративные» блоки Business Model Canvas (по сниппету поиска: [Umbrex](https://umbrex.com/resources/frameworks/organization-frameworks/lean-canvas/), [Lucid](https://lucid.co/blog/lean-canvas-model)). Сайт leanstack.com не открывался.

#### The Mom Test (Rob Fitzpatrick)
- Три правила: говорить о жизни клиента, а не о своей идее; спрашивать о конкретном прошлом поведении, а не о мнениях и гипотетическом будущем; меньше говорить, больше слушать (по сниппету поиска: [Shortform](https://www.shortform.com/blog/what-is-the-mom-test/), [Unusual Ventures](https://www.unusual.vc/rob-fitzpatricks-mom-test/)). momtestbook.com не открывался.

#### ICE и RICE
- ICE = Impact × Confidence × Ease, каждый фактор обычно по шкале 1–10. Авторство приписывают Sean Ellis (книга «Hacking Growth» с Morgan Brown). Один вторичный источник называет автором McKinsey; это **противоречие**, первоисточника нет (по сниппету поиска: [ProdPad](https://www.prodpad.com/glossary/ice-scoring/), [Savio](https://www.savio.io/product-roadmap/ice-scoring-model/), [LearningLoop](https://learningloop.io/glossary/ice-scoring-model)).
- RICE = (Reach × Impact × Confidence) ÷ Effort (Intercom, Sean McBride, 2016). Шкала Impact: 3 / 2 / 1 / 0.5 / 0.25. Confidence: 100 / 80 / 50%. Effort — в человеко-месяцах (по сниппету поиска: [Tempo](https://www.tempo.io/guides/rice-score-prioritization-framework-product-management), [ClickUp](https://clickup.com/blog/rice-prioritization/), [Early](https://early.app/blog/rice-method/)). Оригинальная статья Intercom не открывалась.

#### LTV:CAC ≥ 3 и окупаемость CAC (David Skok, «SaaS Metrics 2.0»)
- Skok в интервью: LTV к CAC — «at least three times», окупаемость CAC — меньше 12 месяцев. Он же: «you're just too early to calculate LTV to CAC» пока продукт ищет product/market fit (по сниппету поиска: [транскрипт интервью](https://videohighlight.com/v/bCBccKfG9U0), [getAbstract](https://www.getabstract.com/en/free-summaries/direct/34583?af=refind)). Первоисточник forentrepreneurs.com не открылся; в выдаче была его испанская версия [metricas-saas-2](https://www.forentrepreneurs.com/metricas-saas-2/).
- Формула LTV по Skok включает **валовую маржу**: простая версия — ARPA × Gross Margin / Churn; расширенная учитывает дисконт и рост, ставка дисконта для pre-scale бизнеса 20–25%. В примере ChartMogul простая формула даёт $2 550, формула Skok — $1 496.17 (по сниппету поиска: [ChartMogul LTV cheat sheet](https://chartmogul.com/resources/ltv-cheat-sheet.pdf), [ChartMogul LTV](https://chartmogul.com/metrics/ltv/)). Расчёт LTV по выручке вместо маржи «inflates LTV and hides an unviable model» (по сниппету поиска, вторичный гайд: [claudeskills unit-economics](https://claudeskills.info/skills/mohitagw15856/pm-claude-skills/unit-economics/)).

#### Break-even CPC
- Break-even CPC = прибыль на клиента × конверсия (клик → покупка). Пример: $150 × 2% = $3.00. CPA = CPC ÷ CR. Прибыль на продажу = средний чек × валовая маржа (по сниппету поиска: [ppc.io calculator](https://ppc.io/tools/ppc-calculator), [upGrowth calculator](https://upgrowth.in/calculator/break-even-cpa-cpc-calculator/), [ClickZ](https://clickz.com/?p=30106)).
- Целевой CPC ставят ниже break-even, чтобы сохранить плановую маржу; break-even — внешняя граница (по сниппету поиска, источник в той же выдаче).

#### Российская практика: Wordstat + Директ
Всё ниже — по сниппету поиска; привязка каждой цитаты к конкретной странице выдачи не проверена: [ppc.world — прогноз бюджета](https://ppc.world/articles/kak-prognozirovat-byudzhet-na-prodvizhenie-v-direkte/amp/), [ppc.world — поиск с малым бюджетом](https://ppc.world/articles/poiskovaya-reklama-s-nebolshim-byudzhetom-nuzhna-li-i-kogda-nuzhna/), [kokoc.com](https://kokoc.com/blog/yandex-direct-budget-estimation/), [vc.ru](https://vc.ru/id416792/178592-kak-samostoyatelno-poschitat-byudzhet-na-yandeks-direkt-i-google-ads), [qna.habr.com](https://qna.habr.com/q/256633), [habr.com](https://habr.com/en/articles/239773).
- Wordstat: регион выбирается вручную; коммерческие уточнения проверяются одним запросом через `(a|b|c)`, например «приточная вентиляционная установка (оптом|поставщик|производитель|дистрибьютор)».
- «Если люди не ищут ваш продукт в интернете, то запускать рекламу на поиске бесполезно». Кейс: 9 запросов в месяц по России — мало для поиска.
- Минимальный бюджет зависит от ниши: в широкой нише с дешёвым кликом — «от 300 рублей в сутки», в узкой нише с дорогим клиентом — до «10–15 тысяч рублей в сутки».
- Прогнозатор Директа учитывает сезонность: в примере январь — 80 427 ₽, июнь — 66 107 ₽ (шкафы-купе). Мнение с форума: «прогноз бюджета всё равно жутко косой» ([qna.habr](https://qna.habr.com/q/256633)).
- Советы практиков: закладывать 20–40% запаса на колебания аукциона и сезонность; тестовый бюджет — сумма, достаточная для ≥10 целевых конверсий в неделю.
- Минус-слова лучше собирать по ходу работы: после их удаления появляются новые запросы ([habr](https://habr.com/en/articles/239773)).
- Статус «Мало показов» (официальная новость Яндекса): группы с очень низким трафиком «приостанавливаются и не участвуют в аукционе на поиске и в сетях» (по сниппету поиска: [yandex.ru/adv/news](https://yandex.ru/adv/news/malo-pokazov-novyy-status-dlya-grupp-obyavleniy)).
  - Факторы статуса: прогноз показов в месяц по Wordstat, настройки кампании и накопленная статистика ([click.ru](https://blog.click.ru/direct-yandex/status-malo-pokazov-v-yandeks-direkte/), [eLama](https://elama.ru/blog/kak-rabotat-so-statusom-malo-pokazov/); по сниппету поиска).
  - Исследование eLama: при 1–5 показах в месяц отключено 38% фраз, при 5–10 — 17% ([eLama](https://elama.ru/blog/kak-rabotat-so-statusom-malo-pokazov/), по сниппету поиска).
  - Ориентир «<30 запросов в месяц» — ответ ассистента Яндекса со ссылкой на ppc.world, **не официальный порог** ([alice.yandex.ru](https://alice.yandex.ru/neurum/c/drugoe/q/esli_menshe_30_poiskovyh_zaprosov_v_mesyac_02eaf880), по сниппету поиска).
- A/B-тест в Директе: справка рекомендует одновременно тестировать варианты не более чем одного элемента объявления (по сниппету поиска: [справка Директа](https://yandex.ru/support/direct/efficiency/ad-groups.html)). Калькулятор размера выборки — [ppc.world](https://ppc.world/articles/ab-test-v-yandeks-direkte-kak-testirovat-novoe-i-optimizirovat-kampanii-v-period-neopredelennosti/).

#### Стоп-факторы у практиков
- Готового списка «стоп-факторов ниши» с порогами в выдаче нет. Встречаются отдельные сигналы (по сниппету поиска):
  - отсутствие поискового спроса — пустая ниша может значить, что спроса нет ([Альфа-Банк, курс](https://kurs.alfabank.ru/courses/svoyo-delo-s-avito-zapuskaem-biznes-po-prodazhe-tovarov-i-uslug/lesson/2/));
  - барьеры входа: законодательные ограничения, патенты, высокие затраты на производство ([reg.ru о монополии](https://www.reg.ru/blog/monopoliya/));
  - малый объём рынка и соотношение спроса и предложения ([vc.ru](https://vc.ru/1892127-kak-vybrat-nishu-dlya-biznesa), [sales-generator](https://sales-generator.ru/blog/konkurentsiya-v-nishe/)).
- «Монополист» в найденных материалах описан только в общем виде: «одна компания занимает доминирующее положение и контролирует значительную часть рынка» ([reg.ru](https://www.reg.ru/blog/monopoliya/), по сниппету поиска). Количественного критерия у практиков не найдено.

### Inferences
- **Проверка формулы «÷ 3» (предложение, арифметика по источникам выше).** Break-even CPC = маржа на клиента × CR. LTV по Skok = ARPA × GM × срок жизни. Цель LTV:CAC ≥ 3 даёт CPC_max = ARPA × **GM** × срок жизни × CR ÷ 3. Формула основателя — частный случай с GM = 100%, поэтому при марже меньше 100% она завышает допустимый CPC в 1/GM раз.
  - Пример: цена 3 000 ₽/мес, срок жизни 6 мес, CR клик→оплата 2%, GM 40%.
  - Формула основателя: 3 000 × 6 × 0.02 ÷ 3 = **120 ₽**.
  - С маржой: 3 000 × 0.4 × 6 × 0.02 ÷ 3 = **48 ₽**.
  - Break-even по марже: 144 ₽. Значит, при 120 ₽ фактическое LTV:CAC равно 1.2, а не 3.
  - Поправки:
    - (а) вместо цены брать маржу (за вычетом себестоимости и комиссии эквайринга; тарифы ЮKassa — другой слой);
    - (б) CR — именно клик → оплата; если тест меряет заявки, умножать на конверсию заявка → оплата;
    - (в) срок жизни до появления данных брать консервативно (первый платёж или 3 месяца), раз Skok считает LTV:CAC преждевременным до PMF;
    - (г) хранить два порога: break-even (÷1, жёсткий стоп) и целевой (÷3).
- **Лестница смоук-теста (предложение).** Ступень 0 — XYZ-гипотеза с порогами, записанная до запуска (Savoia). Ступень 1 — fake door: объявление, лендинг, кнопка, замер CTR и CR в клик или заявку. Ступень 2 — предзаказ с предоплатой через ЮKassa («skin in the game»). Объём ступени — не меньше ~10 целевых конверсий в неделю или по калькулятору выборки (ppc.world). Каждая ступень повышается, только если фактический CPC ≤ CPC_max по марже. Одновременно меняется не больше одного элемента (справка Директа).
- **Стоп-фильтр (предложение, сведено из источников).** Мультипликативный «пол» (idea-validation-agents) по факторам:
  - частотность ядра в Wordstat ниже порога «Мало показов» (порог калибровать самим: официального числа нет);
  - доминирование инкумбента или монополиста (incumbent dominance; метрику доли задать самим, например по выручке из ГИР БО — другой слой);
  - лицензии, патенты или регуляторные барьеры;
  - прогнозный CPC Директа выше break-even CPC.
- Lean Canvas и Mom Test больше подходят как шаблоны для LLM-аналитика (структура карточки идеи; вопросы для будущих интервью), чем как этапы автоматического конвейера. ICE и RICE — грубая сортировка до турнира, но их баллы субъективны, как и самооценки LLM (вывод).

### Gaps
- Первоисточники Savoia (книги, pretotyping.org), Skok (forentrepreneurs.com/saas-metrics-2), Intercom (RICE), leanstack и momtestbook открыть не удалось. Всё по ним — по сниппетам поиска.
- Термин «Initial Level of Interest» не найден ни в одном источнике.
- Первоисточник по pre-order/smoke test (например, «Testing Business Ideas», Bland и Osterwalder) не найден и не проверен.
- Официального числового порога статуса «Мало показов» Яндекс, по найденным материалам, не публикует.
- Структурированного российского плейбука «проверка ниши» с пороговыми значениями стоп-факторов не найдено. Статьи агентств дают лишь отдельные ориентиры.
- API Wordstat, прогноза Директа и ГИР БО в этом слое не исследовались (это слой данных).

---

## 6. Вопрос 4. SaaS-валидаторы идей (ValidatorAI, DimeADozen, IdeaProof, Exploding Topics)

### Takeaway
Это потребительские генераторы отчётов. Публичного API у ValidatorAI, DimeADozen и IdeaProof не найдено. Exploding Topics (Semrush с августа 2024) API имеет, но дорогое: от $1 000 в месяц за 1 000 запросов. Данных Яндекса ни у кого не найдено. Для цикла в РФ — максимум **референс** формата отчёта; встраивать **нет**.

### Cited Findings
- **ValidatorAI** — источники о цене расходятся: «$49 for 3 sessions» против бесплатной разговорной обратной связи. Даёт мгновенную обратную связь и «viability score» на основе поведенческих данных многих основателей (по сниппету поиска: [stork.ai compare](https://www.stork.ai/compare/ideaproof-vs-validatorai), [preuve.ai](https://preuve.ai/blog/best-business-idea-validators-2026.md)). ⚠ preuve.ai — конкурент.
- **DimeADozen.ai** (по сниппету поиска: [stork.ai](https://www.stork.ai/en/dimeadozen-ai), [toolmage](https://www.toolmage.com/en/tool/dimeadozenai/), [CB Insights](https://www.cbinsights.com/company/dimeadozenai)):
  - Freemium: бесплатный Idea Score; Starter Report $9 (только в одном источнике), Entrepreneur Report $129, 3-Pack $179, Enterprise по запросу, без подписки.
  - Сделка: по одному агрегатору, куплен в октябре 2023 за $150 000 (Felipe Arosemena, Danielle de Corneille); CB Insights пишет об октябре 2023 с нераскрытым покупателем. Детали — не проверено.
- **IdeaProof** — от €19.99 за пакет кредитов. Помимо валидации есть модули брендинга, рекламных текстов и бизнес-плана (по сниппету поиска: [stork.ai](https://www.stork.ai/en/ideaproof)).
- API у трёх валидаторов в найденных источниках не упоминается (по сниппету поиска; вывод «API нет» — по отсутствию упоминаний, не по ответу вендора).
- **Exploding Topics**:
  - Приобретён Semrush: анонс основателя Brian Dean датирован 2024-08-20 ([DesignRush](https://news.designrush.com/semrush-buys-exploding-topics-to-strengthen-market-research-capabilities), [Marketing School](https://marketingschool.io/brian-dean-on-selling-to-semrush-evergreen-seo-strategies-exploding-topics-and-more-2065); по сниппету поиска). ⚠ M&A-трекер [Signalbase](https://www.trysignalbase.com/news/acquisitions/exploding-topics-acquired-by-semrush-acquisition) даёт дату 2026-02-26 — противоречие, вероятно ошибка трекера.
  - API, по FAQ вендора: $1 000 за 1 000 запросов в месяц, $2 000 за 5 000, $4 000 за 25 000; лимит 60 запросов в минуту ([et-api](https://explodingtopics.com/feature/et-api), [блог](https://explodingtopics.com/blog/exploding-topics-api); по сниппету поиска).
  - База знаний Semrush: API-пакеты для внутреннего использования, для коммерческого — индивидуальный план ([Semrush KB](https://de.semrush.com/kb/1490-exploding-topics), по сниппету поиска).
  - Обзор третьей стороны: API — дорогое дополнение к плану Business ($249/мес); планы Entrepreneur $39, Investor $99 по состоянию на июнь 2026 ([toolsurf](https://www.toolsurf.com/exploding-topics-review-2026-trend-discovery-tool-features-pricing-worth-it-2026-plans-features-best-deals-compared/), по сниппету поиска). ⚠ Расхождение с FAQ в том, привязан ли API к плану.
  - Данные: «proprietary datasets» и ML-прогноз роста на 12 месяцев. По обзору третьей стороны — поисковики, соцсети, новости, e-commerce. Есть приложение для Zapier и интеграция через HTTP-ноду n8n (по сниппету поиска: [блог ET](https://explodingtopics.com/blog/exploding-topics-api), [metodoviral](https://metodoviral.com/es/blog/integracion-y-automatizacion-con-ia/api-de-exploding-topics-en-zapier-y-n8n-automatizar-tendencias/)).

### Inferences
- Ни один сервис не раскрывает использование данных Яндекса. Тренды и спрос у них опираются на Google и западные платформы. Для ниш РФ, где спрос надо мерить Wordstat и CPC Директа, их метрики нерелевантны (вывод).
- Без API их нельзя встроить в автономный недельный цикл. Exploding Topics с API стоит минимум $12 000 в год и даёт глобальные тренды, а не данные РФ. Возможность оплаты из РФ иностранных SaaS в рамках слоя не проверялась.

### Gaps
- Сайты вендоров (validatorai.com, dimeadozen.ai, ideaproof.io, документация API Exploding Topics) не открывались. Цены и отсутствие API — по сниппетам и обзорам, часть из которых пишут конкуренты.
- Не проверено, доступны ли Semrush и Exploding Topics пользователям из РФ.

---

## 7. Вывод по слою: что взять готовым / что как референс / что писать самому

**Взять готовым (1 позиция)**
- **GPT Researcher + gptr-mcp** — движок desk research для скаута и для текстовой части обогащения: конкуренты, есть ли монополист, цены, регуляторика.
  - Почему: активен (коммит и релиз 0.16.1 от 2026-09-26), Apache-2.0 (⚠ на PyPI MIT), self-hosted в Docker, MCP поверх SSE и streamable HTTP, GigaChat в провайдерах, свой поисковик подключается через custom retriever.
  - Условия: свой RU-поисковый адаптер вместо Tavily, пиновать версию, проверить качество на русских запросах.
  - Источники: [repo](https://github.com/assafelovic/gpt-researcher), [gptr-mcp](https://github.com/assafelovic/gptr-mcp), [custom.py](https://github.com/assafelovic/gpt-researcher/blob/main/gpt_researcher/retrievers/custom/custom.py), [base.py](https://github.com/assafelovic/gpt-researcher/blob/main/gpt_researcher/llm_provider/generic/base.py).

**Как референс**
- **AI-Researcher** — Swiss-турнир (раунды, bye, счёт 0..N), дедупликация с порогом 0.8, фильтр новизны. Перенести алгоритм, но добавить перестановки ([код](https://github.com/NoviScl/AI-Researcher/blob/main/ai_researcher/src/tournament_ranking.py)).
- **Google co-scientist** — Elo со стартом 1200, многоходовые дебаты для лидеров, приоритет матчам похожих, новых и лидирующих идей ([arXiv](https://arxiv.org/pdf/2502.18864)).
- **MT-Bench и FairEval** — перестановка пары с ничьей, BPC и MEC. **arena-hard-auto** — style control по длине. **llm-judge** — Swiss (Monrad), K=32, отсечение top-K, кэш вердиктов.
- **idea-validation-agents** — мультипликативный «пол» скоринга, рубрика насыщенности с incumbent dominance, RAT (≤2 недель, ≤$100), вердикты pursue/test/pivot/drop, пре-мортем.
- **The AI Scientist** — только паттерн: JSON идеи с баллами 1–10, до 5 рефлексий, цикл проверки новизны до 10 запросов с явной строкой решения. Код не брать: «AI Scientist License 1.0» с 2025-12-19.
- **Open Deep Research** — архитектура supervisor-researcher на LangGraph. В прод не брать: архив с 2026-08-21.
- **Фреймворки**: XYZ-гипотеза и «skin in the game» (Savoia), fake door / painted door, Lean Canvas как шаблон карточки идеи, Mom Test для будущих интервью, ICE/RICE для грубой сортировки, LTV:CAC ≥ 3 и окупаемость < 12 мес (Skok), break-even CPC = маржа × CR.
- **SaaS-валидаторы** (ValidatorAI, DimeADozen, IdeaProof, Exploding Topics) — максимум как образец структуры отчёта. Встраивать **нет**: API не найдено (у ET оно от $1 000 в месяц), данных Яндекса нет.

**Писать самому (на Rails/Go)**
1. **Генератор ~50 идей**: промпт в стиле AI Scientist, дедупликация по эмбеддингам, desk research через GPT Researcher (предложение).
2. **Стоп-фильтр с мультипликативным «полом»**. Факторы: частотность Wordstat ниже порога «Мало показов» (калибровать самим, официального числа нет), доля лидера или монополиста, лицензии и регуляторика, прогнозный CPC выше break-even. Готового RU-списка стоп-факторов с порогами не найдено (предложение).
3. **Калькулятор юнит-экономики** с поправленной формулой: CPC_max = цена × **GM** × срок жизни × CR(клик→оплата) ÷ 3, плюс жёсткий стоп при ÷1. Формула основателя без маржи завышает лимит в 1/GM раз; при GM 40% — в 2.5 раза (предложение).
4. **Турнир финалистов**: Swiss на 4–5 раундов, двойной прогон с перестановкой и ничьей при расхождении, судья другого семейства, карточки одинаковой длины с числами, дебаты для топ-2/3. Elo (старт 1200) только для межнедельной истории; при малом N — MLE/Bradley-Terry с bootstrap (предложение).
5. **Лестница смоук-теста**: XYZ-порог до запуска → fake door → предзаказ с предоплатой через ЮKassa. Объём ступени — не меньше ~10 конверсий в неделю или по калькулятору выборки; одна переменная за раз; переход на следующую ступень только при CPC_факт ≤ CPC_max (предложение).
6. **Адаптеры данных** (Wordstat, прогноз Директа, ГИР БО, ЮKassa) — вне этого слоя: готовых open-source решений в рамках слоя не искали.

---

## 8. Источники

**Репозитории и реестры (первичные, открыты 2026-10-09)**
- https://github.com/assafelovic/gpt-researcher
- https://github.com/assafelovic/gpt-researcher/commits
- https://github.com/assafelovic/gpt-researcher/blob/main/LICENSE
- https://github.com/assafelovic/gpt-researcher/tree/main/gpt_researcher/retrievers
- https://github.com/assafelovic/gpt-researcher/blob/main/gpt_researcher/retrievers/custom/custom.py
- https://github.com/assafelovic/gpt-researcher/blob/main/gpt_researcher/llm_provider/generic/base.py
- https://pypi.org/pypi/gpt-researcher/json
- https://github.com/assafelovic/gptr-mcp
- https://github.com/langchain-ai/open_deep_research
- https://github.com/langchain-ai/open_deep_research/commits
- https://pypi.org/pypi/open-deep-research/json
- https://github.com/MaxKmet/idea-validation-agents
- https://github.com/MaxKmet/idea-validation-agents/commits
- https://github.com/NoviScl/AI-Researcher
- https://github.com/NoviScl/AI-Researcher/commits
- https://github.com/NoviScl/AI-Researcher/tree/main/ai_researcher/src
- https://github.com/NoviScl/AI-Researcher/blob/main/ai_researcher/src/tournament_ranking.py
- https://github.com/SakanaAI/AI-Scientist
- https://github.com/SakanaAI/AI-Scientist/commits
- https://github.com/SakanaAI/AI-Scientist/blob/main/ai_scientist/generate_ideas.py
- https://github.com/a-canary/llm-judge
- https://registry.npmjs.org/llm-judge
- https://github.com/lmarena/arena-hard-auto
- https://github.com/The-Swarm-Corporation/AI-CoScientist
- https://pypi.org/pypi/ai-coscientist/json
- https://github.com/mnemox-ai/idea-reality-mcp

**Статьи и блоги по турнирам и LLM-судьям (по сниппету поиска)**
- https://arxiv.org/pdf/2502.18864 — Google, AI co-scientist
- https://research.google/blog/accelerating-scientific-breakthroughs-with-an-ai-co-scientist/
- https://learnprompting.org/blog/google-ai-co-scientist-prompts
- https://itdaily.com/news/software/google-ai-co-scientist
- https://www.eweek.com/fr/news/google-ai-scientist/
- https://educationawards.ie/news/google-launches-gemini-for-science-platform-to-support-ai-assisted-research-across-academic-and-enterprise-settings
- https://arxiv.org/pdf/2409.04109 ; https://iclr.cc/virtual/2025/poster/29961 — Si, Yang, Hashimoto
- https://arxiv.org/pdf/2306.05685 ; https://arxiv.org/html/2306.05685v4 ; https://neurips.cc/virtual/2023/poster/73434 — MT-Bench
- https://tailoredai.substack.com/p/judging-llm-as-a-judge-with-mt-bench
- https://arxiv.org/pdf/2305.17926 ; https://preview.aclanthology.org/setup/2024.acl-long.511 ; https://github.com/i-Eval/FairEval — Wang et al.
- https://arxiv.org/html/2411.14483v2 — Daynauth et al.
- https://arxiv.org/html/2502.15018v1 — Yoon et al.
- https://www.lmsys.org/blog/2023-05-03-arena
- https://cdn.jsdelivr.net/npm/llm-judge@1.0.0/README.md
- https://github.com/Contextualist/lone-arena
- https://mcp.depscope.dev/pkg/pypi/ai-coscientist

**Прочие open-source и агрегаторы (по сниппету поиска)**
- https://www.sourcepulse.org/projects/30847528 ; https://gittrend.io/repo/MaxKmet/idea-validation-agents
- https://dev.to/kzeitar/building-an-ai-powered-saas-app-etc-idea-validation-system-bpg
- https://lablab.ai/submissions/sj2ld96jr7uh9lnukrqz16f7
- https://www.getdrio.com/mcp/io-github-crewhaus-validation/md
- https://skillselion.com/skills/joaquimscosta/arkhe-claude-plugins/startup-validating

**Фреймворки (по сниппету поиска)**
- Savoia: https://www.shortform.com/pdf/the-right-it-pdf-alberto-savoia ; https://res.productcompass.pm/top-product-management-books/the-right-it ; https://www.productcompass.pm/p/how-to-build-the-right-product-with ; https://stvp.stanford.edu/av/build-right-it-entire-talk ; https://helio.zurb.com/blog/pretotype-your-way-to-product-success-validate-market-demand-fast/ ; https://www.jamasoftware.com/?p=8756 ; https://nextview.vc/blog/pretotyping-product-market-fit-google-alberto-savoia/ ; https://skillselion.com/skills/phuryn/pm-skills/brainstorm-experiments-new
- Painted/fake door: https://www.optimizely.com/optimization-glossary/painted-door-test/ ; https://amplitude.com/explore/experiment/painted-door-testing.md
- Lean Canvas: https://umbrex.com/resources/frameworks/organization-frameworks/lean-canvas/ ; https://lucid.co/blog/lean-canvas-model
- Mom Test: https://www.shortform.com/blog/what-is-the-mom-test/ ; https://www.unusual.vc/rob-fitzpatricks-mom-test/
- ICE: https://www.prodpad.com/glossary/ice-scoring/ ; https://www.savio.io/product-roadmap/ice-scoring-model/ ; https://learningloop.io/glossary/ice-scoring-model
- RICE: https://www.tempo.io/guides/rice-score-prioritization-framework-product-management ; https://clickup.com/blog/rice-prioritization/ ; https://early.app/blog/rice-method/
- Skok / LTV: https://videohighlight.com/v/bCBccKfG9U0 ; https://www.getabstract.com/en/free-summaries/direct/34583?af=refind ; https://www.forentrepreneurs.com/metricas-saas-2/ ; https://chartmogul.com/resources/ltv-cheat-sheet.pdf ; https://chartmogul.com/metrics/ltv/ ; https://claudeskills.info/skills/mohitagw15856/pm-claude-skills/unit-economics/
- Break-even CPC: https://ppc.io/tools/ppc-calculator ; https://upgrowth.in/calculator/break-even-cpa-cpc-calculator/ ; https://clickz.com/?p=30106

**Российская практика (по сниппету поиска)**
- https://ppc.world/articles/kak-prognozirovat-byudzhet-na-prodvizhenie-v-direkte/amp/
- https://ppc.world/articles/poiskovaya-reklama-s-nebolshim-byudzhetom-nuzhna-li-i-kogda-nuzhna/
- https://kokoc.com/blog/yandex-direct-budget-estimation/
- https://vc.ru/id416792/178592-kak-samostoyatelno-poschitat-byudzhet-na-yandeks-direkt-i-google-ads
- https://qna.habr.com/q/256633
- https://habr.com/en/articles/239773
- https://yandex.ru/adv/news/malo-pokazov-novyy-status-dlya-grupp-obyavleniy
- https://blog.click.ru/direct-yandex/status-malo-pokazov-v-yandeks-direkte/
- https://elama.ru/blog/kak-rabotat-so-statusom-malo-pokazov/
- https://alice.yandex.ru/neurum/c/drugoe/q/esli_menshe_30_poiskovyh_zaprosov_v_mesyac_02eaf880
- https://yandex.ru/support/direct/efficiency/ad-groups.html
- https://ppc.world/articles/ab-test-v-yandeks-direkte-kak-testirovat-novoe-i-optimizirovat-kampanii-v-period-neopredelennosti/
- https://www.reg.ru/blog/monopoliya/
- https://kurs.alfabank.ru/courses/svoyo-delo-s-avito-zapuskaem-biznes-po-prodazhe-tovarov-i-uslug/lesson/2/
- https://vc.ru/1892127-kak-vybrat-nishu-dlya-biznesa
- https://sales-generator.ru/blog/konkurentsiya-v-nishe/

**SaaS-валидаторы (по сниппету поиска)**
- https://www.stork.ai/compare/ideaproof-vs-validatorai ; https://www.stork.ai/en/ideaproof ; https://www.stork.ai/en/dimeadozen-ai
- https://preuve.ai/blog/best-business-idea-validators-2026.md (⚠ пишет конкурент)
- https://www.toolmage.com/en/tool/dimeadozenai/ ; https://www.cbinsights.com/company/dimeadozenai
- https://explodingtopics.com/feature/et-api ; https://explodingtopics.com/blog/exploding-topics-api ; https://de.semrush.com/kb/1490-exploding-topics
- https://www.toolsurf.com/exploding-topics-review-2026-trend-discovery-tool-features-pricing-worth-it-2026-plans-features-best-deals-compared/
- https://metodoviral.com/es/blog/integracion-y-automatizacion-con-ia/api-de-exploding-topics-en-zapier-y-n8n-automatizar-tendencias/
- https://news.designrush.com/semrush-buys-exploding-topics-to-strengthen-market-research-capabilities ; https://marketingschool.io/brian-dean-on-selling-to-semrush-evergreen-seo-strategies-exploding-topics-and-more-2065 ; https://www.trysignalbase.com/news/acquisitions/exploding-topics-acquired-by-semrush-acquisition
