# Claude Code как рантайм «LLM-компании» из РФ: условия Anthropic, не-Claude модели, переносимость и план выхода

Дата сбора: 2026-10-09.

**Контекст.**
- Соло-основатель в РФ. Агенты «директор», «аналитик» и «маркетолог» гоняют недельный цикл бизнес-идей. В конце цикла — реальные деньги: Яндекс Директ и предоплаты через ЮKassa. Человек выступает советом директоров.
- Заметки опираются на два документа: отчёт `research/ready-made-office.md` (раздел «Риски» → «Доступ к LLM») и заметки `notes/ready-made-office/x-risks-ru.md` (раздел «LLM-провайдер и доступ из РФ»).
- Что уже есть там и здесь не повторяется, только достраивается:
  - России нет в списке поддерживаемых стран;
  - Commercial и Consumer Terms разрешают приостановку доступа;
  - в мае и октябре 2026 были волны блокировок;
  - OpenRouter и РФ;
  - OpenAI-совместимость Yandex AI Studio;
  - существование gpt2giga.

**Пометки.**
- Без пометки — факт сверен с открытой страницей первоисточника: документацией, юридической страницей, GitHub, реестром npm или PyPI.
- «по сниппету поиска» — факт взят только из выдачи WebSearch, сама страница не открывалась.
- «не проверено» — первоисточник не подтверждён.
- «предложение» — собственный синтез или рекомендация исследователя.

**Ограничения среды.**
- Открывались:
  - через curl: code.claude.com, anthropic.com, support.claude.com, raw.githubusercontent.com, npm, PyPI;
  - через WebFetch: github.com.
- Не открылись (DNS или 403 прокси): agentskills.io, opencode.ai, developers.openai.com, api-docs.deepseek.com, docs.z.ai, platform.minimax.io, platform.moonshot.ai, alibabacloud.com, docs.litellm.ai, VentureBeat, Gigazine, Hacker News, aaif.io.
- Поэтому документация китайских провайдеров, LiteLLM и OpenAI Codex взята по сниппетам или из их GitHub.
- Использовано 19 вызовов WebSearch.

---

## 1. Условия Anthropic для автономного и headless-использования

### Кратко
Подписка Pro/Max разрешает автоматизацию только через собственные инструменты Anthropic: `claude -p`, долгоживущий токен `claude setup-token`, GitHub Actions, routines. И только в рамках «ordinary, individual usage». Подписочный OAuth-токен в сторонних harness запрещён, и в 2026 году этот запрет применяли на практике (OpenCode, OpenClaw). Правила биллинга программного использования за 2026 год менялись минимум четыре раза (часть событий — по сниппету поиска).

Для основателя из РФ главное другое. Сам Claude Code в системных требованиях ограничен «Anthropic supported countries», а его лицензия отсылает к Commercial Terms с политикой регионов. Значит, условия не выполняются при любой модели авторизации. Usage Policy не считает трату собственного рекламного бюджета «high-risk» случаем. Но ответственность за действия агентов лежит на пользователе, и она прописана явно.

### Находки с источниками

**Что Anthropic прямо разрешает подписке (Pro/Max)**
- По Consumer Terms (действуют с 8 октября 2025) запрещено: «Except when you are accessing our Services via an Anthropic API Key or where we otherwise explicitly permit it, to access the Services through automated or non-human means, whether through a bot, script, or otherwise». Там же сказано: «You are responsible for all Inputs you submit to our Services and all Actions» и «Actions may not be error free or operate as you intended… You should not rely on any Outputs or Actions without independently confirming their accuracy». — [Consumer Terms](https://www.anthropic.com/legal/consumer-terms)
- «Явное разрешение» на автоматизацию через подписку есть в документации Claude Code. Пример: «For CI pipelines, scripts, or other environments where interactive browser login isn't available, generate a one-year OAuth token with `claude setup-token`». Токен передаётся через `CLAUDE_CODE_OAUTH_TOKEN`. — [Authentication](https://code.claude.com/docs/en/authentication.md)
- GitHub Actions принимает `CLAUDE_CODE_OAUTH_TOKEN`: это «an OAuth token that authenticates with your Claude subscription, available on Pro, Max, Team, and Enterprise plans». — [GitHub Actions](https://code.claude.com/docs/en/github-actions.md)
- Routines — research preview, тарифы Pro, Max, Team и Enterprise. Они «run autonomously as full Claude Code cloud sessions: there is no permission-mode picker». Routines «belong to your individual claude.ai account», их прогоны расходуют лимиты подписки.
  - Почасовые лимиты:
    - 100 плановых запусков в час на аккаунт;
    - 30 запусков в час на одну routine для Run now и API-вызовов;
    - по 100 в час на аккаунт отдельно для Run now и для API-вызовов.
  - Минимальный интервал расписания — 1 час.
  - Эндпоинт `/fire` работает под beta-заголовком `experimental-cc-routine-2026-04-01`.

  — [Routines](https://code.claude.com/docs/en/routines.md)
- Юридическая страница Claude Code формулирует так:
  - «Advertised usage limits for Pro and Max plans assume ordinary, individual usage of Claude Code and the Agent SDK»;
  - «OAuth authentication is intended exclusively for purchasers of Claude Free, Pro, Max, Team, and Enterprise subscription plans and is designed to support ordinary use of Claude Code and other native Anthropic applications».

  — [Legal and compliance](https://code.claude.com/docs/en/legal-and-compliance.md)
- Справка Anthropic (статья обновлена 7–8 октября 2026):
  - «Update October 7, 2026: Claude Max and Team plans now include monthly API credits, which cover the Claude Agent SDK, claude -p, the Claude API, and Claude Managed Agents. You can still use the Claude Agent SDK, claude -p, and third-party apps with your subscription limits.»
  - «Update June 15, 2026: We've paused the previously-announced changes to Claude Agent SDK usage. For now, nothing has changed…»

  — [Use the Claude Agent SDK with your Claude plan](https://support.claude.com/en/articles/15036540-use-the-claude-agent-sdk-with-your-claude-plan)
- Ежемесячные API-кредиты:
  - размеры: Max 5x — $100, Max 20x — $200, Team — $20 или $100 за место (общий пул до $500); Pro не получает;
  - кредиты зачисляются в привязанную организацию Claude Console и покрывают API, Agent SDK и Managed Agents;
  - кредиты не покрывают интерактивный Claude Code;
  - когда кредиты кончаются, «API requests stop until your next monthly credits arrive», если нет купленных кредитов.

  — [Monthly API credits for Max and Team plans](https://support.claude.com/en/articles/17154008-monthly-api-credits-for-max-and-team-plans)
- В мае 2026 Anthropic анонсировала перевод Agent SDK, `claude -p`, GitHub Actions и сторонних приложений с лимитов подписки на отдельный ежемесячный кредит с 15 июня 2026. Изменение отменили или приостановили (по сниппету поиска). — [gaugr.app](https://gaugr.app/en/blog/claude-billing-overhaul-agent-sdk-credits-june-2026), [env.dev](https://env.dev/updates/anthropic-agent-sdk-credits), [wmedia.es](https://wmedia.es/en/tips/claude-code-agent-sdk-credit)
- Режим `--bare` — «the recommended mode for scripted and SDK calls, and will become the default for `-p` in a future release». В bare-режиме «Claude Code never reads OAuth credentials or the system keychain»: нужен `ANTHROPIC_API_KEY` или `apiKeyHelper`. Кроме того, bare-режим не подгружает хуки, скиллы, команды, субагентов, плагины, MCP и CLAUDE.md. — [Headless](https://code.claude.com/docs/en/headless.md), [Env vars: CLAUDE_CODE_SIMPLE](https://code.claude.com/docs/en/env-vars.md)

**Что запрещено: сторонние harness, чужие токены, перепродажа**
- По юридической странице Claude Code:
  - «Anthropic does not permit third-party developers to offer Claude.ai login into their own applications, or to route requests through Free, Pro, or Max plan credentials on behalf of their users. Moreover, developers may not collect, store, or intermediate Claude.ai credentials or session tokens»;
  - «Anthropic reserves the right to take measures to enforce these restrictions and may do so without prior notice»;
  - разрешено: «an end user… signing in to the unmodified Claude Code binary with their own Claude subscription».

  — [Legal and compliance](https://code.claude.com/docs/en/legal-and-compliance.md)
- Хронология применения в 2026 году:
  - **9 января.** Серверные проверки перестали пускать сторонние инструменты (OpenCode, Cline, RooCode) к подписке Pro/Max через OAuth; по одному источнику, после протестов это частично откатывали (по сниппету поиска). — [VentureBeat](https://venturebeat.com/ai/anthropic-cracks-down-on-unauthorized-claude-usage-by-third-party-harnesses/)
  - **Февраль.** Документация и условия уточнены: OAuth-токены подписки — только для Claude Code и Claude.ai (по сниппету поиска). — [Gigazine, 20.02.2026](https://gigazine.net/gsc_news/en/20260220-anthropic-third-party-block)
  - **19 марта.** В OpenCode влит PR #18186 «anthropic legal requests» (автор thdxr). Он удаляет встроенный плагин `opencode-anthropic-auth`, значение `claude-code-20250219` из заголовка `anthropic-beta`, провайдера «anthropic» из enum, системный промпт `anthropic-20250930.txt` и Pro/Max OAuth из документации. Реакции на PR: 👍 19, 👎 550. — [PR #18186](https://github.com/sst/opencode/pull/18186) (канонический репозиторий — anomalyco/opencode)
  - Документация OpenCode теперь пишет, что Anthropic запрещает плагины, которые используют модели Claude Pro/Max в OpenCode, и что начиная с версии 1.3.0 они не поставляются. — [OpenCode providers.mdx](https://github.com/anomalyco/opencode/blob/dev/packages/web/src/content/docs/providers.mdx)
  - **4 апреля.** OpenClaw и «все сторонние harness» лишились лимитов подписки, нужен «extra usage» по API-тарифу. Причина в письме Anthropic — «outsized strain» на инфраструктуру (по сниппету поиска). — [TechRadar](https://www.techradar.com/pro/bad-news-claude-users-anthropic-says-youll-need-to-pay-to-use-openclaw-now), [зеркало HN](https://hn.nuxt.dev/item/47633396), [lilting.ch](https://lilting.ch/en/articles/anthropic-claude-code-openclaw-third-party-paygo)
  - **Противоречие.** Справка от 7 октября 2026 говорит, что «third-party apps» по-прежнему могут использовать лимиты подписки. Какие именно приложения имеются в виду (вероятно, одобренные Anthropic интеграции), из текста не ясно. — [support 15036540](https://support.claude.com/en/articles/15036540-use-the-claude-agent-sdk-with-your-claude-plan)
- Обзор Agent SDK: «Use of the Claude Agent SDK is governed by Anthropic's Commercial Terms of Service, including when you use it to power products and services that you make available to your own customers». — [Agent SDK overview](https://code.claude.com/docs/en/agent-sdk/overview.md)

**Привязка самого Claude Code к стране, независимо от модели**
- В системных требованиях Claude Code указано «**Location**: Anthropic supported countries». — [Advanced setup](https://code.claude.com/docs/en/setup.md)
- Лицензия Claude Code: «© Anthropic PBC. All rights reserved. Use is subject to Anthropic's Commercial Terms of Service». — [LICENSE.md](https://github.com/anthropics/claude-code/blob/main/LICENSE.md)
- Юридическая страница: «Claude Code remains governed by Anthropic's standard terms… regardless of the platform through which it is accessed». — [Legal and compliance](https://code.claude.com/docs/en/legal-and-compliance.md)
- Commercial Terms (с 17 июня 2025):
  - D.2 — Usage Policy и Supported Regions Policy; «Customer must cooperate with reasonable requests… including to verify Customer's identity»;
  - D.3 — «It is Customer's responsibility to evaluate whether Outputs are appropriate for Customer's use case, including where human review is appropriate».

  — [Commercial Terms](https://www.anthropic.com/legal/commercial-terms)

**Usage Policy и агенты, которые тратят деньги**
- На 09.10.2026 страница Usage Policy показывает редакцию с датой вступления **12 ноября 2026**. Действующая до этой даты редакция не открывалась: она за JS-кнопкой «Previous Version». — [Usage Policy](https://www.anthropic.com/legal/aup)
- Политика прямо распространяется на «individuals using our apps (such as Claude.ai and Claude Code)». Нарушение грозит последствиями: «warn you or throttle, limit, suspend, or terminate your access». — [Usage Policy](https://www.anthropic.com/legal/aup)
- Агенты: «Agentic use cases must comply with the Usage Policy. Users are responsible for ensuring that agents they build or deploy, including actions those agents take through tools, browsers, or connected systems, comply with this Usage Policy». — [Usage Policy](https://www.anthropic.com/legal/aup)
- Требования high-risk (квалифицированный человек в контуре и раскрытие ИИ) действуют для «High-risk AI Recommendations» в 11 областях:
  - «Finance» здесь — это «a personalized recommendation to buy, sell, hold, or allocate among specific investment products… advising an individual on their own taxes»;
  - в исключениях: «Business operations that do not advise or decide about a specific individual» и «Carrying out a decision a person has already made…».

  — [Usage Policy](https://www.anthropic.com/legal/aup)
- Если агент говорит с клиентами напрямую: «All consumer-facing chatbots or AI agents that communicate directly with external users must disclose to users that they are interacting with AI rather than a human». — [Usage Policy](https://www.anthropic.com/legal/aup)
- Пункты о злоупотреблениях, применимые к сценарию:
  - «Promote or facilitate the generation or distribution of spam»;
  - «Engage in actions or behaviors that circumvent the guardrails or terms of other platforms or services»;
  - «Access, facilitate, or provide others… access… in violation of our Supported Regions Policy»;
  - «Circumvent a ban through the use of a different account…»;
  - «Resell, proxy, or otherwise provide access to Claude through unauthorized means, including services that route requests through consumer subscriptions or misrepresent the product or client being used».

  — [Usage Policy](https://www.anthropic.com/legal/aup)
- Классификатор auto mode в Claude Code по умолчанию блокирует production-деплои, отправку чувствительных данных наружу, запись в secret manager и т. п. Отдельного правила про платежи или траты в списке по умолчанию нет. — [Permission modes](https://code.claude.com/docs/en/permission-modes.md)
- Безопасность headless: `claude -p` без `--bare` «runs the hooks in a project's `.claude/settings.json` and connects the servers in its `.mcp.json`, even in a folder you've never trusted». — [Headless](https://code.claude.com/docs/en/headless.md)

### Выводы (предложение)
- **Цикл, который запускает сам основатель для своего бизнеса** через `claude -p`, `setup-token` или routines, формально попадает в «explicitly permit». Но «ordinary, individual usage» нигде не определена в цифрах. Агенты, работающие круглосуточно и по расписанию, — серая зона, и здесь Anthropic оставляет за собой право действовать «without prior notice».
- **Любая схема «свой harness + подписочный токен» запрещена.** Сюда относятся OpenCode, Goose и собственное Agent SDK-приложение, если оно отдаёт claude.ai-логин другим. Остаётся только API-ключ, то есть Commercial Terms, и снова политика регионов.
- **Для РФ условия не выполняются в любой конфигурации.** Требование «supported countries» относится к самому клиенту Claude Code, а не к модели. Поэтому подмена бэкенда на GigaChat юридического статуса не меняет: остаются лицензия и Commercial Terms D.2.
- **Политика по программному использованию нестабильна.** Январь, апрель, май–июнь и октябрь 2026 — четыре изменения. Рантайм, у которого каждые 2–3 месяца меняется экономика и правила, плохо подходит для цикла с реальными деньгами.
- **Трата собственного бюджета на Директ не попадает в high-risk области** Usage Policy. Значит, обязательного «qualified human-in-the-loop» по Usage Policy нет. Но гейт «совет утверждает бюджет» всё равно нужен:
  - Consumer Terms: «Actions may not be error free»;
  - Commercial Terms D.3;
  - auto mode не блокирует траты по умолчанию.

  Если маркетолог будет переписываться с покупателями, нужна плашка «вы общаетесь с ИИ». Правила Директа и ЮKassa об автоматизации тоже нужно соблюдать: пункт про «terms of other platforms».

### Пробелы
- Текст Usage Policy, действующей до 12.11.2026, не получен.
- Точный текст письма Anthropic от 4 апреля 2026 (HN заблокирован) и исходный майский анонс не открыты.
- Количественного определения «ordinary, individual usage» не найдено.
- Нет данных, приводят ли routines, `claude -p` или `setup-token` из РФ к блокировкам чаще, чем интерактивное использование.

---

## 2. Claude Code на не-Claude моделях: что работает и что ломается

### Кратко
Технически Claude Code работает с любым эндпоинтом формата Anthropic Messages через `ANTHROPIC_BASE_URL`:
- DeepSeek, Z.AI (GLM), MiniMax и Alibaba Model Studio документируют такую настройку сами;
- у Moonshot есть Anthropic-совместимый API;
- GigaChat подключается через gpt2giga (руководство проверено на Claude Code 2.1.187);
- для YandexGPT и Cloud.ru Anthropic-эндпоинта не найдено, нужен транслирующий прокси (LiteLLM, claude-code-router).

Anthropic такую схему официально не поддерживает. Каждый релиз Claude Code может добавить поле, которое ломает прокси: так было с `output_config` в марте 2026. Часть функций завязана на api.anthropic.com и теряется: WebSearch, Remote Control, routines, advisor. В сентябре 2026 менялось поведение auto mode на сторонних эндпоинтах. Из РФ китайские провайдеры напрямую оплачиваются плохо.

### Находки с источниками

**Официальная позиция и что требует гайд совместимости**
- «Anthropic doesn't endorse, maintain, or audit third-party gateway products, and doesn't support routing Claude Code to non-Claude models through any gateway». Формулировка повторяется на двух страницах. — [Gateways overview](https://code.claude.com/docs/en/gateways.md), [Other LLM gateways](https://code.claude.com/docs/en/llm-gateway.md)
- Админский чек-лист предполагает, что шлюз «configured to route Claude model names to your provider». Новые модели Claude дают 404, пока их не добавят в маршрутизацию. — [Rollout](https://code.claude.com/docs/en/llm-gateway-rollout.md)
- Гайд совместимости требует:
  - один из трёх форматов: Anthropic Messages (`/v1/messages`, `count_tokens` опционально), Bedrock InvokeModel или Vertex rawPredict;
  - передавать `anthropic-beta` и `anthropic-version` без изменений, не по белому списку;
  - стримить SSE по порядку до `message_delta` и `message_stop`, не буферизовать и не выкидывать `ping`;
  - передавать тела ошибок без изменений: логика автоповторов Claude Code матчится по их тексту.

  — [Gateway compatibility guide](https://code.claude.com/docs/en/llm-gateway-protocol.md)
- Таблица «Feature pass-through» того же гайда (что ломается):

  | Функция | Что отправляет Claude Code | Симптом при поломке |
  |---|---|---|
  | Adaptive reasoning | `thinking: {"type":"adaptive"}`; для неизвестных имён моделей (алиасов шлюза) поле тоже отправляется | `400` |
  | Context management | `context_management` | `400` «Extra inputs are not permitted» |
  | Beta-поля инструментов | `strict`, `defer_loading` | `400` |
  | Effort и structured outputs | `output_config` | `400` |
  | Prompt caching | `cache_control` | без ошибки: каждый ход тарифицируется как некэшированный |
  | `count_tokens` | — | `/context` показывает оценку |

  Частичное лечение — `CLAUDE_CODE_DISABLE_EXPERIMENTAL_BETAS=1`. — [Gateway compatibility guide](https://code.claude.com/docs/en/llm-gateway-protocol.md)
- Для неизвестного ID модели Claude Code считает окно в 200K. Реальное окно задаётся через `CLAUDE_CODE_MAX_CONTEXT_TOKENS`. — [Model configuration](https://code.claude.com/docs/en/model-config.md)
- Обнаружение моделей шлюза через `CLAUDE_CODE_ENABLE_GATEWAY_MODEL_DISCOVERY=1`: Claude Code «keeps an entry when its `id` contains `claude` or `anthropic`… an ID that contains neither substring doesn't». Не-Claude модели в `/model` не видны, если не переименовать их «под Claude». — [Gateway compatibility guide](https://code.claude.com/docs/en/llm-gateway-protocol.md)
- Субагенты: поле `model` принимает `sonnet`, `opus`, `haiku`, `fable`, `inherit` или «a full model ID». Есть `CLAUDE_CODE_SUBAGENT_MODEL` и `CLAUDE_CODE_SUBAGENT_MODEL_FORCE`. — [Subagents](https://code.claude.com/docs/en/sub-agents.md), [Env vars](https://code.claude.com/docs/en/env-vars.md)
  - Раньше было хуже: issue #34821 от 16 марта 2026 жаловался, что `model` у Task tool «hardcoded to an enum of ["sonnet", "opus", "haiku"]» и пользователи шлюзов не могут дать субагентам GLM или DeepSeek. Issue закрыт ботом как stale 13 апреля 2026, ответа Anthropic нет. — [issue #34821](https://github.com/anthropics/claude-code/issues/34821)

**Что Claude Code выключает или по-прежнему тянет с api.anthropic.com**
- Если `ANTHROPIC_BASE_URL` указывает не на api.anthropic.com, выключаются Remote Control и server-managed settings «whatever the gateway forwards». — [Feature availability](https://code.claude.com/docs/en/feature-availability.md)
- Tool search по умолчанию выключен для хостов не первой стороны. Fine-grained tool streaming на подключениях через шлюз выключен по умолчанию. — [Env vars](https://code.claude.com/docs/en/env-vars.md)
- WebSearch ходит в поисковый бэкенд Anthropic: «The search backend is not configurable. To search with a different provider, add an MCP server». — [Tools reference](https://code.claude.com/docs/en/tools-reference.md)
- Предпроверка WebFetch отправляет имя хоста на api.anthropic.com «regardless of which model provider you use». Отключается через `skipWebFetchPreflight: true`. — [Data usage](https://code.claude.com/docs/en/data-usage.md)
- Флаги функций: с токеном шлюза без API-ключа «Claude Code has no credential to fetch the flags with». Часть функций и стартовый режим разрешений тогда другие. — [Env vars](https://code.claude.com/docs/en/env-vars.md)
- Auto mode на сторонних эндпоинтах в сентябре 2026:
  - 2.1.278 (19.09): на шлюзах по умолчанию серверный классификатор (`CLAUDE_CODE_AUTO_MODE_SERVER=0` — отказаться);
  - 2.1.283 (25.09): интерактивные сессии на сторонних провайдерах стартуют в auto mode;
  - 2.1.285 (29.09): `claude -p` и Python SDK на сторонних провайдерах тоже стартуют в auto mode, если режим не задан.

  — [Changelog](https://code.claude.com/docs/en/changelog.md)
  - Сторонний репозиторий с 0 звёзд связывает с этим «зависание» вызовов инструментов на не-Anthropic эндпоинтах и лечит его через `CLAUDE_CODE_AUTO_MODE_SERVER=0`. Причинность автор не доказал. — [NonAnthropicEndpointFix](https://github.com/RichardRamirez123/NonAnthropicEndpointFix)
- Ломка от релизов, пример — issue LiteLLM #22963 от 6 марта 2026: Claude Code через LiteLLM v1.81.14 на gpt-5.1 падал с «Unknown parameter: 'output_config'». Чинилось PR сообщества #22990, #23704 и #23706; issue закрыт 18 марта. — [LiteLLM #22963](https://github.com/BerriAI/litellm/issues/22963)

**Провайдеры с Anthropic-совместимым эндпоинтом и их доступность из РФ**

| Провайдер | Эндпоинт для Claude Code | Источник | Доступ и оплата из РФ |
|---|---|---|---|
| DeepSeek | `https://api.deepseek.com/anthropic` | Официальный гайд DeepSeek на GitHub: `deepseek-v4-pro[1m]` на роли opus и sonnet, `deepseek-v4-flash[1m]` на haiku, плюс `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC=1` ([awesome-deepseek-agent](https://github.com/deepseek-ai/awesome-deepseek-agent/blob/main/docs/claude_code.md)); страница api-docs — по сниппету поиска ([DeepSeek: Claude Code](https://api-docs.deepseek.com/guides/agent_integrations/claude_code)) | Российские карты обычно не проходят, «Мир» не принимается (по сниппету поиска: [vc.ru](https://vc.ru/services/3025455-kak-oplatit-deepseek-iz-rossii-i-popolnit-api)) |
| Z.AI (GLM) | `https://api.z.ai/api/anthropic`; для КНР — `https://open.bigmodel.cn/api/anthropic` (по сниппету поиска) | [docs.z.ai](https://docs.z.ai/devpack/tool/claude) (по сниппету поиска). Документация goose: «The existing Z.AI provider continues to use the Anthropic-compatible endpoint» ([goose providers](https://github.com/aaif-goose/goose/blob/main/documentation/docs/getting-started/providers.md)) | Отказы по российскому BIN, проблемы с 3-D Secure (по сниппету поиска: [vc.ru](https://vc.ru/services/3045731-kak-oplatit-glm-iz-rossii)) |
| MiniMax | `https://api.minimax.io/anthropic`; для КНР — `https://api.minimaxi.com/anthropic` | [MiniMax: Claude Code](https://platform.minimax.io/docs/m-plan/claude-code) (по сниппету поиска) | Данных не найдено |
| Alibaba Model Studio (Qwen) | `https://dashscope.aliyuncs.com/apps/anthropic` (pay-as-you-go); только `/v1/messages`, без списка моделей | [Alibaba Cloud](https://www.alibabacloud.com/help/en/model-studio/claude-code) (по сниппету поиска) | Не проверено |
| Moonshot (Kimi) | В README Kimi-K2: на platform.moonshot.ai «we provide an OpenAI/Anthropic-compatible API»; температура маппится как `request_temperature * 0.6`. Точный URL для Claude Code не проверен | [Kimi-K2](https://github.com/MoonshotAI/Kimi-K2) | Российские карты отклоняются (по сниппету поиска: [vc.ru](https://vc.ru/services/3056116-kak-kupit-kimi-api-i-podpisku-kimi-code-iz-rossii)) |
| GigaChat | Своего нет; через gpt2giga (локально `http://localhost:8090`) | См. блок о прокси ниже | Рубли, провайдер из РФ |
| YandexGPT (AI Studio) | Не найден. Есть только OpenAI-совместимый `https://ai.api.cloud.yandex.net/v1` (по сниппету поиска) | [AI Studio API](https://aistudio.yandex.ru/docs/en/ai-studio/concepts/api) | Рубли; для Claude Code нужен транслирующий прокси (не проверено) |
| Cloud.ru Foundation Models | Anthropic-эндпоинт не найден (по сниппету поиска) | — | Рубли. По заметкам прошлого слоя, в июле 2026 добавлены внешние модели Alibaba, DeepSeek и Z.ai с оплатой по факту, через OpenAI-совместимый API (по сниппету поиска) |

- Совместимость DeepSeek Anthropic API (по сниппету поиска) — [DeepSeek Anthropic API](https://api-docs.deepseek.com/guides/anthropic_api/):
  - `cache_control` игнорируется на tools, text, tool_use и tool_result, хотя сам кэш автоматический;
  - `thinking` поддержан, но `budget_tokens` игнорируется;
  - `anthropic-beta` для `/messages` игнорируется; `mcp_servers` и `container` тоже;
  - блоки `document`, `search_result`, `redacted_thinking` и `mcp_tool_use` не поддерживаются;
  - неизвестное имя модели молча маппится на `deepseek-flash`.
- Сами вендоры подтверждают, что их модели работают в harness Claude Code:
  - MiniMax-M2 считал SWE-bench Multilingual «using the claude-code CLI (300 max steps) as the evaluation scaffold» и требует не вырезать `<think>…</think>` из истории ([MiniMax-M2](https://github.com/MiniMax-AI/MiniMax-M2));
  - README GLM-4.7 говорит об улучшениях «in mainstream agent frameworks such as Claude Code» ([GLM-4.5 repo](https://github.com/zai-org/GLM-4.5));
  - Qwen3-Coder заявляет поддержку «Qwen Code, CLINE, Claude Code» ([Qwen3-Coder](https://github.com/QwenLM/Qwen3-Coder)).

**Прокси и шлюзы**

| Прокси | Что делает | Лицензия и активность | Нюанс для РФ |
|---|---|---|---|
| gpt2giga | FastAPI-прокси: OpenAI-, Anthropic- и Gemini-запросы → GigaChat. Эндпоинты `/messages` и `/messages/count_tokens`. README: «GigaChat не является drop-in заменой OpenAI или Anthropic API». Вне scope — «Anthropic Files beta, Skills beta, Agents beta» | MIT, 137★. 0.3.0 вышел 2026-08-03, 0.3.1a1 — 2026-10-09 ([PyPI](https://pypi.org/pypi/gpt2giga/json), [repo](https://github.com/ai-forever/gpt2giga)) | Есть руководство для Claude Code, «Проверено: 26 июня 2026 — `2.1.187 (Claude Code)`»: `ANTHROPIC_BASE_URL=http://localhost:8090`, `ANTHROPIC_API_KEY=0`, `claude --model GigaChat-2-Max`, headless через `claude -p … --output-format json`. Оговорка руководства: «gpt2giga перенаправляет все запросы в модель GigaChat, заданную через GIGACHAT_MODEL» ([integrations/claude-code](https://github.com/ai-forever/gpt2giga/blob/main/integrations/claude-code/README.md)). Есть также интеграции для qwen-code, codex, aider и openhands ([integrations/](https://github.com/ai-forever/gpt2giga/tree/main/integrations)) |
| claude-code-router (CCR) | «Local model gateway and control plane for coding agents»: профили для Claude Code, Codex, OpenCode, Kilo Code и др. Протоколы: OpenAI Chat/Responses, Anthropic Messages, Gemini. Провайдеры: DeepSeek, Moonshot, Z.AI, Bailian и custom | MIT; npm `@musistudio/claude-code-router` 3.1.3 (2026-10-09), 94 версии с 2025-06-10 ([README](https://github.com/musistudio/claude-code-router/blob/main/README.md), [npm](https://registry.npmjs.org/@musistudio/claude-code-router)) | Custom OpenAI-совместимый провайдер (Yandex, Cloud.ru) теоретически подходит — не проверено. В README есть «local login import» и «Chrome login-state import»; импорт claude.ai-логина нарушил бы правила Anthropic (предложение: не включать) |
| LiteLLM proxy | Официальный туториал «Use Claude Code with Non-Anthropic Models»: `/v1/messages` → любой провайдер, `ANTHROPIC_BASE_URL` и `ANTHROPIC_AUTH_TOKEN` (по сниппету поиска) | MIT; PyPI 1.104.2 (2026-10-08) ([PyPI](https://pypi.org/pypi/litellm/json), [туториал](https://docs.litellm.ai/docs/tutorials/claude_non_anthropic_models)) | Страница провайдера GigaChat есть (из прошлых заметок). Риски цепочки поставок — в прошлом отчёте |

### Выводы (предложение)
- **Ответ на вопрос (b): да, технически.** Самый короткий российский путь — GigaChat через gpt2giga: руководство автора прокси явно покрывает Claude Code, включая headless. YandexGPT и модели Cloud.ru идут через LiteLLM или CCR с трансляцией Anthropic → OpenAI. Это двойная трансляция, и никто её на Claude Code не проверял.
- **Ломается следующее:**
  1. Каждое новое поле в релизах Claude Code: прокси приходится обновлять синхронно (пример — `output_config`).
  2. Окно контекста: по умолчанию 200K, для моделей с меньшим окном нужно задать вручную.
  3. Кэширование: молча выключается, если прокси не передаёт `cache_control`, а провайдер не кэширует сам. Это прямой рост счёта, потому что системный промпт и инструменты Claude Code отправляются каждый ход.
  4. WebSearch: нужен свой MCP-поиск.
  5. Remote Control, routines, облачные сессии: недоступны.
  6. Auto mode и классификатор: меняются от релиза к релизу.
  7. Thinking: у провайдеров семантика отличается (DeepSeek игнорирует `budget_tokens`, MiniMax требует сохранять `<think>`).
  8. Выбор модели: в `/model` не-Claude ID не видны без переименования. Модели по ролям задаются полным ID в субагентах.
- **Юридически** клиент остаётся программой Anthropic под Commercial Terms с требованием «supported countries», и он продолжает ходить на api.anthropic.com (флаги, предпроверка WebFetch), если это не отключить. Значит, «Claude Code + GigaChat из РФ» — это не уход от Anthropic, а та же зависимость от их клиента, его лицензии и темпа релизов.
- **Практика:** зафиксировать версию Claude Code (канал stable или `DISABLE_AUTOUPDATER`) и обновлять её вместе с прокси после прогона eval-набора.

### Пробелы
- Качество вызова инструментов у GigaChat-2-Max и YandexGPT в цикле Claude Code (субагенты, скиллы, длинные сессии) — независимых данных нет. Руководство gpt2giga подтверждает только подключение.
- Есть ли у Moonshot официальная страница про Claude Code и какой у неё URL — не проверено.
- Оплата MiniMax и Alibaba из РФ — данных нет.
- Работает ли связка LiteLLM → Yandex AI Studio с Claude Code (формат tools, стриминг) — не проверено.

---

## 3. Переносимость: стандарт Agent Skills и совместимые harness

### Кратко
Лучше всего переносятся:
- скиллы (SKILL.md): открытый стандарт с 18.12.2025, а OpenCode, Goose и Crush читают даже `.claude/skills` напрямую;
- инструкции: AGENTS.md понимают почти все, Claude Code тоже;
- MCP-серверы: переносимы как протокол, но конфиги переписываются.

Средне переносятся:
- команды: у OpenCode почти идентичный синтаксис;
- субагенты: markdown с YAML, но поля разные;
- хуки: Goose, Qwen Code и Codex используют ту же модель событий `PreToolUse`/`PostToolUse`/`Stop`, а у OpenCode это JS-плагины.

Хуже всего переносятся:
- плагины: в августе 2026 появился стандарт Agent Plugins, но Anthropic в нём нет (по сниппету);
- функции, привязанные к claude.ai.

### Находки с источниками

**Стандарт Agent Skills**
- Agent Skills представлены 16 октября 2025. Обновление: «published Agent Skills as an open standard for cross-platform portability. (December 18, 2025)». — [Anthropic: news/skills](https://www.anthropic.com/news/skills), [Engineering blog](https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills)
- Репозиторий спецификации: «The Agent Skills format was originally developed by Anthropic». Код под Apache-2.0, документация под CC-BY-4.0, 26.0k★. — [agentskills/agentskills](https://github.com/agentskills/agentskills)
- Спецификация SKILL.md — [specification.mdx](https://github.com/agentskills/agentskills/blob/main/docs/specification.mdx):
  - `name`: 1–64 символа, строчные латинские буквы, цифры и дефисы; совпадает с именем каталога;
  - `description`: 1–1024 символа;
  - опционально: `license`, `compatibility` (до 500 символов), `metadata`, `allowed-tools` («Experimental. Support for this field may vary between agent implementations»);
  - каталоги `scripts/`, `references/`, `assets/`;
  - прогрессивная загрузка; рекомендация «Keep your main SKILL.md under 500 lines».
- Витрина клиентов насчитывает 46 записей. В их числе Gemini CLI, OpenCode, Cursor, Goose, GitHub Copilot, VS Code, Claude Code, «ChatGPT & Codex», Roo Code, Kiro, Junie, OpenHands, Amp, Factory, Mistral Vibe, TRAE, Tabnine, Snowflake Cortex Code, OpenClaw и Hermes Agent. Qwen Code, Kilo Code и Crush в списке нет, хотя их собственная документация заявляет поддержку. — [clients.jsx](https://github.com/agentskills/agentskills/blob/main/docs/snippets/clients.jsx)

**Что в скиллах Claude Code не переносится**
- «Claude Code skills follow the Agent Skills open standard… Claude Code extends the standard». Поля только для Claude Code:
  - `when_to_use`, `argument-hint`, `arguments`;
  - `disable-model-invocation`, `user-invocable`, `disallowed-tools`;
  - `model`, `effort`;
  - `context: fork`, `agent`, `background`;
  - `hooks`, `paths`, `shell`.
- «Claude Code-only body features, such as dynamic context injection, don't function in claude.ai chat or through the API».
- При загрузке в claude.ai или Skills API лишний ключ даёт жёсткую ошибку «Unexpected key(s) in SKILL.md frontmatter».
- «Custom commands have been merged into skills»: `.claude/commands/*.md` по-прежнему работает.

  — [Skills](https://code.claude.com/docs/en/skills.md)

**Кто и откуда читает SKILL.md**

| Harness | Пути | Читает `.claude/skills`? | Источник |
|---|---|---|---|
| OpenCode | `.opencode/skills/`, `.claude/skills/`, `.agents/skills/`; глобально — `~/.config/opencode/skills/`, `~/.claude/skills/`, `~/.agents/skills/`. «Unknown frontmatter fields are ignored» | Да | [skills.mdx](https://github.com/anomalyco/opencode/blob/dev/packages/web/src/content/docs/skills.mdx) |
| Goose | `~/.agents/skills/`, `.agents/skills/`; для совместимости — `.goose/skills/`, `.claude/skills/`, `~/.claude/skills/`. «goose skills are compatible with Claude Desktop and other agents that support Agent Skills» | Да | [using-skills.md](https://github.com/aaif-goose/goose/blob/main/documentation/docs/guides/context-engineering/using-skills.md) |
| Crush | Глобально — `~/.agents/skills/`, `~/.claude/skills/`, `~/.config/crush/skills/`; в проекте — `.agents/skills`, `.crush/skills`, `.claude/skills`, `.cursor/skills` | Да | [README](https://github.com/charmbracelet/crush/blob/main/README.md) |
| Qwen Code | `~/.qwen/skills/`, `.qwen/skills/` | Не найдено | [skills.md](https://github.com/QwenLM/qwen-code/blob/main/docs/users/features/skills.md) |
| Codex CLI | `.agents/skills` от текущего каталога до корня репозитория, `$HOME/.agents/skills`, `/etc/codex/skills` | Не найдено | [Codex skills](https://developers.openai.com/codex/skills) (по сниппету поиска) |

**AGENTS.md и инструкции**
- Claude Code читает `AGENTS.md`, если рядом нет `CLAUDE.md`. Если есть оба, читается только `CLAUDE.md`. — [Memory](https://code.claude.com/docs/en/memory.md)
- OpenCode сначала ищет `AGENTS.md`, затем как fallback берёт `CLAUDE.md` и `~/.claude/CLAUDE.md`: «For users migrating from Claude Code, OpenCode supports Claude Code's file conventions as fallbacks». Отключается через `OPENCODE_DISABLE_CLAUDE_CODE=1`. — [rules.mdx](https://github.com/anomalyco/opencode/blob/dev/packages/web/src/content/docs/rules.mdx)
- Codex: «Custom instructions with AGENTS.md» (по сниппету поиска). — [developers.openai.com](https://developers.openai.com/codex/guides/agents-md)
- Репозиторий AGENTS.md: MIT, 24.8k★. — [agentsmd/agents.md](https://github.com/agentsmd/agents.md)
- Agentic AI Foundation (Linux Foundation) объявлена 9 декабря 2025 и хостит MCP (Anthropic), goose (Block) и AGENTS.md (OpenAI) (по сниппету поиска). — [InfoQ](https://infoq.com/news/2025/12/agentic-ai-foundation/), [It's FOSS](https://itsfoss.com/news/agentic-ai-foundation-launch/)
- README goose подтверждает: «goose is part of the Agentic AI Foundation (AAIF) at the Linux Foundation». — [goose](https://github.com/block/goose)

**Субагенты, команды, хуки, плагины, MCP: матрица переносимости**

| Примитив | Формат Claude Code | Куда переносится и как | Источники |
|---|---|---|---|
| Субагенты | `.claude/agents/*.md`: YAML с `name`, `description`, `tools`, `model` (алиас или полный ID) | **OpenCode:** `.opencode/agents/*.md` с полями `description`, `mode`, `model: provider/model-id`, `permission`, `steps`; поле `tools` устарело; `.claude/agents` не упоминается. **Qwen Code:** `.qwen/agents/*.md` с `name`, `description`, `model`, `approvalMode`, `tools`, `disallowedTools` — почти тот же формат. **Codex:** `~/.codex/agents/*.toml` с `name`, `description`, `developer_instructions` (по сниппету поиска). **Goose:** субагенты по просьбе на естественном языке или через `sub_recipes` в YAML-рецептах; по умолчанию 25 ходов и 5 минут; вложенных субагентов нет | [agents.mdx](https://github.com/anomalyco/opencode/blob/dev/packages/web/src/content/docs/agents.mdx), [Qwen sub-agents](https://github.com/QwenLM/qwen-code/blob/main/docs/users/features/sub-agents.md), [Codex subagents](https://developers.openai.com/codex/subagents), [goose subagents](https://github.com/aaif-goose/goose/blob/main/documentation/docs/guides/context-engineering/subagents.mdx) |
| Команды | `.claude/commands/*.md` или скиллы; `$ARGUMENTS` | **OpenCode:** `.opencode/commands/*.md`, `$ARGUMENTS`, `$1…`, `` !`cmd` ``, `@file`, frontmatter `description`, `agent`, `model`, `subtask` — почти копирование с переименованием каталога. **Goose:** документ slash-commands есть, не открывался. **Gemini CLI:** «Custom Commands», формат не проверен | [commands.mdx](https://github.com/anomalyco/opencode/blob/dev/packages/web/src/content/docs/commands.mdx), [goose docs](https://github.com/aaif-goose/goose/tree/main/documentation/docs/guides/context-engineering), [Gemini CLI README](https://github.com/google-gemini/gemini-cli/blob/main/README.md) |
| Хуки | `settings.json`, события `PreToolUse`, `PostToolUse`, `Stop`, `SessionStart`, `UserPromptSubmit`, `SubagentStop` и др.; блокировка — exit code 2 или `permissionDecision: deny` | **Goose:** `hooks/hooks.json` в плагине по «Open Plugins hooks specification», те же события; блокирует `PreToolUse` через exit 2 или `{"decision":"block"}`. **Qwen Code:** `settings.json`, `PreToolUse` и др.; принимает имена инструментов Claude Code (`Bash`, `Write`) как алиасы матчера и выставляет `CLAUDE_PROJECT_DIR`. **Codex:** «Lifecycle hooks», команда `/hooks`; список событий — по комментарию сообщества (по сниппету поиска). **OpenCode:** JS/TS-плагины с событиями `tool.execute.before/after`, `session.idle` и др.; блокировка — исключение в `tool.execute.before`; нужна переписка. **Crush:** «preliminary support for hooks» | [Claude hooks](https://code.claude.com/docs/en/hooks.md), [goose hooks](https://github.com/aaif-goose/goose/blob/main/documentation/docs/guides/context-engineering/hooks.md), [Qwen hooks](https://github.com/QwenLM/qwen-code/blob/main/docs/users/features/hooks.md), [Codex config.md](https://github.com/openai/codex/blob/main/docs/config.md), [OpenCode plugins](https://github.com/anomalyco/opencode/blob/dev/packages/web/src/content/docs/plugins.mdx), [Crush README](https://github.com/charmbracelet/crush/blob/main/README.md) |
| Плагины | `.claude-plugin/plugin.json`, маркетплейсы | **Goose:** форматы «Open Plugins» и Gemini extensions; совместимость с плагинами Claude не заявлена. **Отраслевой стандарт Agent Plugins:** v1.0.0 от 6 августа 2026; TSC — Amazon, Cursor, Microsoft, OpenAI, Vercel, позже Google; Anthropic в списке нет; хуки вне портируемого ядра (по сниппету поиска). В документации плагинов Claude Code упоминаний Agent Plugins не найдено (проверены overview, loading, plugins-reference) | [goose plugins](https://github.com/aaif-goose/goose/blob/main/documentation/docs/guides/context-engineering/plugins.md), [AAIF blog](https://aaif.io/blog/from-skills-and-tools-to-portable-agent-plugins), [Google Developers Blog](https://developers.googleblog.com/agent-plugins-package-your-skills-tools-and-more/), [Plugins overview](https://code.claude.com/docs/en/plugins/overview.md) |
| MCP | `.mcp.json` (`mcpServers`); claude.ai-коннекторы | Серверы переносимы как протокол, конфиги переписываются. **OpenCode:** ключ `mcp`, `"type": "local"`, `"command": [...]`, `"environment"`; `"type": "remote"`, `"url"`, `"headers"`, `"oauth"`. **Crush:** `mcp` (stdio, http, sse). **Goose:** extensions (`--with-extension` и др.). Claude.ai-коннекторы «load only when your claude.ai subscription is the active authentication method» — не переносятся | [OpenCode mcp-servers](https://github.com/anomalyco/opencode/blob/dev/packages/web/src/content/docs/mcp-servers.mdx), [Crush README](https://github.com/charmbracelet/crush/blob/main/README.md), [goose CLI](https://github.com/aaif-goose/goose/blob/main/documentation/docs/guides/goose-cli-commands.md), [Feature availability](https://code.claude.com/docs/en/feature-availability.md) |

### Выводы (предложение)
- **Компания из скиллов, AGENTS.md и своих MCP-серверов переносится за дни, а не недели.** Условие — frontmatter скиллов ограничен шестью полями спецификации, а логика лежит в скриптах и MCP, а не в специфике Claude Code (`context: fork`, `` !`cmd` ``-инъекции, `hooks` во frontmatter).
- **Хуки стоит писать как самостоятельные скрипты** с вводом и выводом JSON и exit code 2. Тогда их без изменений вызывают Claude Code, Goose и Qwen Code, а для OpenCode нужна тонкая JS-обёртка.
- **Субагентов проще всего описывать как «роль = скилл + промпт + модель»** и генерировать файлы агентов под каждый harness из одного источника.
- **Плагины Claude Code и claude.ai-коннекторы — не переносимый слой.** Их не стоит делать носителем бизнес-логики.

### Пробелы
- Полный список событий хуков Codex и формат команд Gemini CLI — не проверены, документация недоступна.
- Читает ли Qwen Code `.claude/skills` и `.claude/agents` — в документации не найдено.
- Текст спецификации Agent Plugins не открывался.

---

## 4. Альтернативные open-source harness

### Кратко
Для российских моделей и недельного цикла по расписанию лучше всего подходят два harness:
- **OpenCode** — MIT, крупнейшее сообщество; читает `.claude/skills` и `CLAUDE.md`; подключает любой OpenAI-совместимый провайдер; есть `run`/`serve`; встроенного планировщика нет.
- **Goose** — Apache-2.0, под Linux Foundation; рецепты, встроенный cron-планировщик `goose schedule`, хуки в модели Claude, `.claude/skills` как fallback; custom-провайдеры в форматах OpenAI, Anthropic и Ollama.

Сильный резерв — **Qwen Code**: ближе всех к семантике Claude Code (субагенты, хуки, скиллы, `qwen serve`), с поддержкой любых OpenAI-совместимых провайдеров. **Codex CLI** привязан к Responses API. **Kilo** — надстройка над OpenCode. **Crush** — не OSI-лицензия и бедные примитивы. **Gemini CLI** работает только с моделями Google.

### Таблица

| Harness | URL | Лицензия | Активность (на 09.10.2026) | Skills / subagents / commands / hooks / MCP / headless / scheduling | Провайдеры, включая РФ | Вердикт (предложение) |
|---|---|---|---|---|---|---|
| OpenCode | [anomalyco/opencode](https://github.com/anomalyco/opencode) (бывший sst/opencode) | MIT | 212.4k★, 15 881 коммит; npm `opencode-ai` 1.18.35 от 2026-10-06 ([npm](https://registry.npmjs.org/opencode-ai)) | Skills ✓, включая `.claude/skills`. Subagents ✓ (`.opencode/agents`). Commands ✓. Hooks ≈ через JS/TS-плагины. MCP ✓. Headless ✓: `opencode run --format json`, `opencode serve` (HTTP), SDK, `opencode github`, `opencode acp`. Scheduling ✗ — в CLI не упоминается ([cli.mdx](https://github.com/anomalyco/opencode/blob/dev/packages/web/src/content/docs/cli.mdx)) | «75+ LLM providers»; custom OpenAI-совместимый через `@ai-sdk/openai-compatible` и `baseURL` — сюда подходят Yandex AI Studio, Cloud.ru, gpt2giga (не проверено). Yandex, GigaChat и Cloud.ru в доке не упомянуты ([providers.mdx](https://github.com/anomalyco/opencode/blob/dev/packages/web/src/content/docs/providers.mdx)) | **№1 для интерактивной и серверной работы** |
| Goose | [aaif-goose/goose](https://github.com/aaif-goose/goose) (бывший block/goose) | Apache-2.0 | 55.1k★, 5 806 коммитов; релизы v1.52.0 (23.09), v1.53.0 (02.10), v1.54.0 (08.10.2026) ([releases](https://github.com/aaif-goose/goose/releases)); AAIF | Skills ✓, включая `.claude/skills`. Subagents ✓ (рецепты; только в autonomous-режиме). Slash-commands ✓. Hooks ✓ (модель событий Claude). MCP ✓ (extensions). Headless ✓: `goose run --recipe … --output-format json --max-turns`, `goose serve`. Scheduling ✓: `goose schedule add --cron` ([CLI](https://github.com/aaif-goose/goose/blob/main/documentation/docs/guides/goose-cli-commands.md)) | 15+ провайдеров; custom-провайдеры «must use OpenAI, Anthropic, or Ollama compatible API formats»; `OPENAI_HOST`/`OPENAI_BASE_PATH`; LiteLLM, OpenRouter, Ollama ([providers](https://github.com/aaif-goose/goose/blob/main/documentation/docs/getting-started/providers.md)). РФ — через custom-провайдер (не проверено) | **№2: лучший для расписания** |
| Qwen Code | [QwenLM/qwen-code](https://github.com/QwenLM/qwen-code) | Apache-2.0 (в npm поле license пустое) | 28.4k★; npm 0.25.0 от 2026-10-05 ([npm](https://registry.npmjs.org/@qwen-code/qwen-code)); форк Gemini CLI v0.8.2 | Skills ✓ (`.qwen/skills`). Subagents ✓ (`.qwen/agents`, формат как у Claude). Commands ✓. Hooks ✓ (события Claude, алиасы `Bash` и `Write`). MCP ✓. Headless ✓ (`qwen -p`, демон `qwen serve`). Scheduling ≈: `/loop` живёт только в сессии, у каналов — постоянный планировщик ([scheduled-tasks](https://github.com/QwenLM/qwen-code/blob/main/docs/users/features/scheduled-tasks.md)) | «Supports OpenAI, Anthropic, Gemini, and Qwen APIs. Any third-party provider or local model» ([README](https://github.com/QwenLM/qwen-code/blob/main/README.md)); у gpt2giga есть `integrations/qwen-code` | **Резерв №3: ближе всех к Claude Code** |
| Codex CLI | [openai/codex](https://github.com/openai/codex) | Apache-2.0 | 128.4k★, 12 064 коммита; npm 0.162.0 от 2026-10-08 ([npm](https://registry.npmjs.org/@openai/codex)) | Skills ✓ (`.agents/skills`). Subagents ✓ (`.codex/agents/*.toml`). Hooks ✓ (lifecycle). MCP ✓. Headless ✓ (non-interactive). Scheduling в CLI — не проверено ([docs](https://github.com/openai/codex/tree/main/docs)) | Custom-провайдеры только через Responses API: `wire_api="chat"` удалён в феврале 2026 (по сниппету поиска, [OpenRouter](https://openrouter.ai/blog/tutorials/codex-cli-openrouter/)). Yandex AI Studio заявляет Responses API (по сниппету, прошлые заметки) — прямое подключение возможно, не проверено | Условно: зависит от Responses API провайдера |
| Kilo Code (CLI) | [Kilo-Org/kilocode](https://github.com/Kilo-Org/kilocode) | MIT | 27.5k★, 33 352 коммита; npm `@kilocode/cli` 7.8.8 от 2026-10-07 ([npm](https://registry.npmjs.org/@kilocode/cli)) | «Kilo CLI is a fork of OpenCode» — примитивы те же; `kilo run --auto` для CI; маркетплейс agents, skills, MCP и plugins ([README](https://github.com/Kilo-Org/kilocode/blob/main/README.md)) | «500+ models» через платформу Kilo; РФ — не проверено | Не нужен поверх upstream-OpenCode |
| Crush | [charmbracelet/crush](https://github.com/charmbracelet/crush) | FSL-1.1-MIT (source-available, не OSI) | 28.6k★, 4 276 коммитов; npm `@charmland/crush` 0.98.1 от 2026-10-09 ([npm](https://registry.npmjs.org/@charmland/crush)) | Skills ✓, включая `.claude/skills`. Subagents — не найдено. Hooks «preliminary». MCP ✓ (stdio, http, sse). Headless и scheduling — не найдено ([README](https://github.com/charmbracelet/crush/blob/main/README.md)) | «OpenAI- or Anthropic-compatible APIs» | Не для ядра |

Вне таблицы — **Gemini CLI** ([google-gemini/gemini-cli](https://github.com/google-gemini/gemini-cli)): Apache-2.0, 107.3k★, npm 0.63.0 от 2026-10-06. Аутентификация только через Google — личный аккаунт, Gemini API key или Vertex AI ([README](https://github.com/google-gemini/gemini-cli/blob/main/README.md)). Российские модели без форка не подключить; доступность Gemini API из РФ не проверена. Исключён.

### Глубже: OpenCode
- **Перенос компании с Claude Code:**
  - `CLAUDE.md` и `.claude/skills` читаются как есть, неизвестные поля frontmatter игнорируются ([rules.mdx](https://github.com/anomalyco/opencode/blob/dev/packages/web/src/content/docs/rules.mdx), [skills.mdx](https://github.com/anomalyco/opencode/blob/dev/packages/web/src/content/docs/skills.mdx));
  - команды копируются в `.opencode/commands` с тем же синтаксисом;
  - агенты переписываются: `tools` → `permission`, `model` → `provider/model-id`;
  - хуки переносятся в плагин с `tool.execute.before`, который можно прерывать исключением ([plugins.mdx](https://github.com/anomalyco/opencode/blob/dev/packages/web/src/content/docs/plugins.mdx));
  - MCP — в ключ `mcp`.
- **Рантайм:**
  - `opencode serve` — headless HTTP-сервер, с паролем через `OPENCODE_SERVER_PASSWORD`;
  - `opencode run --attach` — подключение к нему без холодного старта MCP;
  - `--format json` — поток событий для своего оркестратора.
- **Модели по ролям:** у каждого агента своя `model`; субагенты наследуют модель вызвавшего агента, если не задана своя ([agents.mdx](https://github.com/anomalyco/opencode/blob/dev/packages/web/src/content/docs/agents.mdx)).
- **Риски (предложение):**
  - высокий темп: в npm 12 234 версии `opencode-ai` (видимо, вместе с preview- и snapshot-сборками);
  - история юридического давления Anthropic (PR #18186) — к российским моделям отношения не имеет;
  - встроенного расписания нет, нужны cron или systemd и свой контроль состояния;
  - 212k звёзд — не аргумент: прошлый отчёт показал, что звёзды не валидированы как метрика.

### Глубже: Goose
- **Рецепты.** YAML с `instructions`, `parameters`, `extensions`, `sub_recipes` и `settings.max_turns`. Запуск: `goose run --recipe recipe.yaml --params …`, вывод `--output-format json|stream-json`, ограничения `--max-turns` и `--max-tool-repetitions`. — [CLI](https://github.com/aaif-goose/goose/blob/main/documentation/docs/guides/goose-cli-commands.md), [subagents](https://github.com/aaif-goose/goose/blob/main/documentation/docs/guides/context-engineering/subagents.mdx)
- **Планировщик.** Пример: `goose schedule add --schedule-id daily-report --cron "0 0 9 * * *" --recipe-source …`; cron на 6 полей; есть `sessions` и `run-now`. Для unattended-запусков рекомендован `GOOSE_DISABLE_KEYRING=1`. `goose serve` требует `GOOSE_SERVER__SECRET_KEY` и поддерживает `--enable-scheduler`. — [CLI](https://github.com/aaif-goose/goose/blob/main/documentation/docs/guides/goose-cli-commands.md)
- **Субагенты.**
  - `GOOSE_SUBAGENT_PROVIDER` и `GOOSE_SUBAGENT_MODEL` задают отдельную модель для подзадач.
  - По умолчанию 25 ходов и 5 минут, вложенности нет.
  - «Subagents are disabled in manual approval, smart approval, and chat-only modes».

  — [subagents](https://github.com/aaif-goose/goose/blob/main/documentation/docs/guides/context-engineering/subagents.mdx)
- **Хуки.** События те же, что у Claude Code; блокировка через exit 2 или `{"decision":"block"}`; `on_failure: "block"` для fail-closed. — [hooks](https://github.com/aaif-goose/goose/blob/main/documentation/docs/guides/context-engineering/hooks.md)
- **Провайдеры.** Custom-провайдер — JSON в `~/.config/goose/custom_providers/`. Предупреждение документации: «goose extensively uses tool calling, so models without it can only do chat completion». — [providers](https://github.com/aaif-goose/goose/blob/main/documentation/docs/getting-started/providers.md)
- **Риски (предложение):**
  - субагенты работают только в autonomous-режиме, а это противоречит одобрениям «совета» внутри сессии. Значит, одобрения нужно делать вне harness, через свой шлюз трат;
  - 5 минут на субагента может не хватить для исследовательских задач, а настраиваемость лимита не проверена.

### Выводы (предложение)
- **Для «LLM-компании» на российских моделях:**
  - Goose — исполнитель недельного цикла: рецепт на роль, `goose schedule` или внешний cron, хуки как гейты;
  - OpenCode — интерактивная разработка и серверный режим;
  - Qwen Code — проверочный третий harness с ближайшей к Claude Code семантикой.
- **В любом варианте** состояние цикла, деньги и одобрения остаются в своём ядре или шлюзе (рекомендация прошлого отчёта). Harness — заменяемый исполнитель.

### Пробелы
- Ни один harness не проверен вживую с YandexGPT или GigaChat. Неизвестно:
  - принимает ли Yandex AI Studio `Bearer` и стриминг tools в формате, который ждут `@ai-sdk/openai-compatible` и goose;
  - насколько стабилен tool calling в длинных сессиях.
- Скачиваются ли бинарники и npm-пакеты из РФ без VPN — не проверено (вопрос открыт и в прошлом отчёте).

---

## 5. Функции, привязанные к аккаунту claude.ai

### Кратко
CLI, Agent SDK, субагенты, хуки, команды, скиллы, плагины, MCP и Workflows работают на любом провайдере. Всё «облачное» и «удалённое» требует claude.ai-аккаунта:
- routines;
- облачные сессии и мобильный клиент;
- Remote Control;
- Desktop;
- Slack, Chrome, Computer use, Artifacts, Voice;
- claude.ai-коннекторы.

При шлюзе на не-Claude модель эти функции теряются. Channels требуют claude.ai или Console API-ключ.

Сам Claude Code — проприетарный. С февраля 2025 вышло 537 версий, в последние 90 дней — 78. Регулярно удаляются команды и инструменты, меняются умолчания.

### Находки с источниками
- **Только с claude.ai-подпиской, недоступны с Console API-ключом и у сторонних провайдеров:**
  - Cloud sessions и Claude Code на мобильных;
  - Slack — Pro/Max;
  - Desktop;
  - Routines (`/schedule`);
  - Ultrareview;
  - Code Review — Team/Enterprise;
  - Remote Control;
  - Chrome extension;
  - Computer use — Pro/Max;
  - Artifacts;
  - Voice dictation.

  — [Feature availability](https://code.claude.com/docs/en/feature-availability.md)
- **Работают на любом провайдере:** «CLI and Agent SDK», «Subagents, hooks, commands, and skills», «CLAUDE.md memory, plugins, and MCP servers», «Checkpoints, sandboxing, and Workflows», OpenTelemetry. Исключение — claude.ai-коннекторы MCP, они работают только при подписочной авторизации. — [Feature availability](https://code.claude.com/docs/en/feature-availability.md)
- **Channels** (Telegram, Discord, iMessage; research preview): «require Anthropic authentication through claude.ai or a Console API key, and are not available on Amazon Bedrock, Google Cloud's Agent Platform, or Microsoft Foundry». — [Channels](https://code.claude.com/docs/en/channels.md)
- **Remote Control:** «API keys are not supported». Не работает с Bedrock, Vertex и Foundry, при `ANTHROPIC_BASE_URL` не на api.anthropic.com и через Claude apps gateway. — [Remote Control](https://code.claude.com/docs/en/remote-control.md)
- **Облачные сессии** «always use your subscription credentials. If you set `ANTHROPIC_API_KEY` or `ANTHROPIC_AUTH_TOKEN` in the cloud environment, it doesn't override your subscription credentials». Routines на не-Claude модели не запустить. — [Authentication](https://code.claude.com/docs/en/authentication.md)
- **Чем документация предлагает заменить:**
  - `/loop` вместо `/schedule`. Но задачи `/loop` живут только в сессии, и «Recurring tasks automatically expire 7 days after creation» ([Scheduled tasks](https://code.claude.com/docs/en/scheduled-tasks.md));
  - GitHub Actions или GitLab CI вместо облачных сессий;
  - WebFetch вместо поиска ([Feature availability](https://code.claude.com/docs/en/feature-availability.md)).
  - Локальные задачи Desktop срабатывают, только «while the app is open and your computer is awake», и тоже требуют Desktop, то есть claude.ai ([Desktop scheduled tasks](https://code.claude.com/docs/en/desktop-scheduled-tasks.md)).
- **Сетевые зависимости:** api.anthropic.com (запросы к API, предпроверка WebFetch, флаги функций, телеметрия), claude.ai, platform.claude.com (OAuth, в том числе refresh), downloads.claude.ai (установщик и автообновление). — [Network config](https://code.claude.com/docs/en/network-config.md)

**Сам Claude Code: лицензия, темп релизов, ломающие изменения**
- **Лицензия:**
  - LICENSE.md: «© Anthropic PBC. All rights reserved. Use is subject to Anthropic's Commercial Terms of Service» ([LICENSE.md](https://github.com/anthropics/claude-code/blob/main/LICENSE.md));
  - в npm поле license — «SEE LICENSE IN README.md»; пакет — нативный бинарник через optionalDependencies, Node ≥22 ([npm](https://registry.npmjs.org/@anthropic-ai/claude-code));
  - Python-обёртка `claude-agent-sdk` — MIT, 0.2.165 от 2026-10-08 ([PyPI](https://pypi.org/pypi/claude-agent-sdk/json)), но использование Agent SDK регулируется Commercial Terms ([Agent SDK overview](https://code.claude.com/docs/en/agent-sdk/overview.md)).
- **Темп релизов** (npm `@anthropic-ai/claude-code`):
  - 537 версий с 2025-02-24; v1.0.0 — 2025-05-22, v2.0.0 — 2025-09-29;
  - с сентября 2025 по сентябрь 2026 — 20–30 версий в месяц, за 90 дней — 78;
  - `latest` 2.1.295 от 2026-10-08, `stable` 2.1.286.

  — [npm registry](https://registry.npmjs.org/@anthropic-ai/claude-code)
  - Канал stable «typically about a week behind and skips releases with major regressions». — [Setup](https://code.claude.com/docs/en/setup.md)
- **Удаления и смены умолчаний в 2026** — [Changelog](https://code.claude.com/docs/en/changelog.md):
  - 2.1.68 (04.03): из первого API убраны Opus 4 и 4.1, закреплённые сессии переведены на Opus 4.6;
  - 2.1.73 (11.03): `/output-style` объявлен устаревшим;
  - 2.1.92 (04.04): удалены `/tag` и `/vim`;
  - 2.1.198 (01.07): удалён мастер `/agents`;
  - 2.1.212 (17.07): параметр `mode` у Task tool устарел;
  - 2.1.222 (04.08): «Removed ultraplan feature»;
  - 2.1.225 (08.08): исправлено — временный 401 подменял долгоживущий `CLAUDE_CODE_OAUTH_TOKEN`, «breaking headless sessions until restart»;
  - 2.1.277 (18.09): удалён инструмент TaskOutput;
  - 2.1.278–2.1.285 (19–29.09): смена умолчаний auto mode, см. раздел 2;
  - анонсировано: `--bare` станет умолчанием для `-p`.
- **Утечка кода** (по сниппету поиска). 31 марта 2026 в npm-пакете v2.1.88 оказался source map, открывший около 512 тыс. строк TypeScript клиента. Anthropic подавала DMCA на зеркала и форки; не переписанные форки снимались. — [Kilo blog](https://blog.kilo.ai/p/claude-code-source-leak-a-timeline), [Hackmag](https://hackmag.com/news/claude-code-leak), [liveinthefuture](https://liveinthefuture.org/stories/claude-code-dmca-copyright-paradox.html)

### Выводы (предложение)
- **UX «совета директоров»** (одобрения с телефона, Telegram-канал, routines по расписанию, облачные прогоны) в Claude Code целиком держится на claude.ai-аккаунте. Это ровно та точка, которая в РФ уже ломалась (баны 2026 года). При уходе на шлюз или не-Claude модель эти функции пропадают в любом случае. Значит, их сразу нужно делать своими: Telegram-бот, cron или systemd, headless-прогоны, журнал в своей БД.
- **Форки утёкшего кода** (OpenClaude и подобные) — не путь выхода: это проприетарный код под DMCA.
- **Темп изменений** требует фиксировать версию и обновлять её по графику, после прогона eval-набора.

### Пробелы
- Работают ли Channels с `ANTHROPIC_BASE_URL` на не-Anthropic бэкенд — документация прямо не говорит.
- Официального пост-мортема Anthropic по утечке не найдено.

---

## 6. План выхода (предложение)

**Принцип.** Claude Code — инструмент разработки и, возможно, эталонный harness для сравнения. Рантайм компании собирается из переносимых частей так, чтобы смена harness стоила дни.

**Шаг 0. Сейчас: гигиена артефактов**
1. Инструкции:
   - держать в `AGENTS.md`; Claude Code читает его при отсутствии `CLAUDE.md`, OpenCode — первым;
   - Claude-специфику вынести в отдельный тонкий `CLAUDE.md` или не использовать.
2. Скиллы:
   - frontmatter — только 6 полей спецификации: `name`, `description`, `license`, `compatibility`, `metadata`, `allowed-tools`;
   - без `context: fork`, `` !`cmd` ``-инъекций и `hooks` во frontmatter;
   - логику — в `scripts/` и в свои MCP-серверы;
   - линт в CI: например, упаковка через `package_skill.py` из [anthropics/skills](https://github.com/anthropics/skills) падает с «Unexpected key(s)» на лишних полях.
3. Роли «директор / аналитик / маркетолог» описать в одном YAML-источнике: промпт, скиллы, модель, лимиты, разрешённые инструменты. Из него генерировать `.claude/agents/*.md`, `.opencode/agents/*.md`, рецепты goose и `.qwen/agents/*.md`.
4. Хуки:
   - самостоятельные скрипты: JSON на stdin, exit 2 — блок;
   - регистрировать в Claude Code `settings.json`, goose `hooks.json` и Qwen `settings.json`;
   - для OpenCode — JS-обёртка на `tool.execute.before`.
5. Деньги и одобрения — не в harness. Собственный MCP-шлюз трат с лимитами и очередью одобрений «совета» (прошлый отчёт). Harness получает только read-only ключи и «запрос на трату».
6. Состояние цикла — в своей БД, конечный автомат (прошлый отчёт). Harness вызывается как исполнитель шага: `claude -p`, `opencode run --format json`, `goose run --recipe … --output-format json`.

**Шаг 1. Параллельный прогон (2–4 недели)**
- Тот же eval-набор ролей из прошлого отчёта прогнать на трёх конфигурациях:
  - OpenCode + YandexGPT через OpenAI-совместимый API;
  - goose + GigaChat через gpt2giga;
  - Qwen Code + тот же бэкенд как контроль.
- Плюс эталон — Claude Code + Claude, только как инструмент разработчика.
- Метрики: доля успешных многошаговых вызовов инструментов, стоимость шага в рублях, кэш-хиты, падения из-за формата.

**Шаг 2. Вывод планирования из Anthropic**
- Не использовать routines, Channels и Remote Control как часть цикла.
- Расписание — systemd timer или cron на своём сервере в РФ, либо `goose schedule`.
- Уведомления и одобрения — свой Telegram-бот, который пишет в БД одобрений.

**Шаг 3. Фиксация версий**
- Claude Code — канал stable или `DISABLE_AUTOUPDATER`.
- Версии OpenCode, goose, gpt2giga и LiteLLM — тоже зафиксированы.
- Обновление раз в 2–4 недели после eval; к обновлению Claude Code — синхронная проверка прокси, памятуя про `output_config`.

**Шаг 4. Триггеры переключения** — любой из списка:
- блокировка или запрос верификации аккаунта Anthropic;
- новое изменение правил программного использования;
- релиз Claude Code, ломающий прокси дольше одного цикла;
- рост цены шага выше порога.

Действие: переключить исполнителя шагов в автомате на goose или OpenCode. Скиллы, MCP и хуки уже общие.

**Плюсы:** нет единой точки отказа в Anthropic; переключение — смена исполнителя, а не переписывание компании. **Минусы:** нужен генератор конфигов под 2–3 harness, eval-набор и дисциплина. Часть удобств Claude Code в продакшене не используется: `context: fork`, routines, Remote Control, WebSearch.

---

## 7. Вывод

**(a) Можно ли опираться на Claude Code как на продакшен-рантайм из РФ?** Нет.
- Требование «Location: Anthropic supported countries» относится к самому клиенту. Лицензия отсылает к Commercial Terms с политикой регионов. Это верно при любой авторизации, в том числе при не-Claude модели.
- Подписочная автоматизация разрешена только через инструменты Anthropic и для «ordinary, individual usage».
- Подписочные токены в сторонних harness запрещены, и запрет применялся (OpenCode — март 2026; OpenClaw — апрель 2026, по сниппету поиска).
- Правила программного использования менялись четыре раза за год.
- Функции, удобные для «совета» (routines, Remote Control, Channels, облако), держатся на claude.ai-аккаунте — той самой точке отказа.
- Как инструмент разработки Claude Code остаётся текущей рабочей настройкой основателя на его риске.

**(b) Может ли Claude Code работать на моделях, доступных из РФ?** Технически да:
- GigaChat через gpt2giga, руководство проверено на 2.1.187;
- YandexGPT и Cloud.ru через LiteLLM или CCR — не проверено;
- китайские провайдеры с нативными Anthropic-эндпоинтами, но их трудно оплатить из РФ.

Ограничения:
- Anthropic это не поддерживает, а релизы ломают прокси;
- WebSearch, Remote Control, routines и облако теряются;
- окно контекста, кэш и thinking нужно настраивать вручную;
- юридически это всё равно клиент Anthropic.

Поэтому вариант годится как переходный мост, но не как фундамент.

**(c) Насколько переносима компания из skills, subagents, commands, hooks и MCP?**
- **Высоко:** скиллы (открытый стандарт; OpenCode, Goose и Crush читают `.claude/skills` напрямую), AGENTS.md, MCP-серверы.
- **Средне:** команды (OpenCode — почти копирование), субагенты (перевод полей), хуки (Goose, Qwen Code и Codex — та же модель событий, OpenCode — JS-обёртка).
- **Низко:** плагины Claude Code, claude.ai-коннекторы, routines, Channels, Remote Control, Claude-специфичный frontmatter.

Итог: при дисциплине из шага 0 плана выхода переезд на Goose или OpenCode с российскими моделями — работа на дни. Главная неизвестная — качество вызова инструментов у YandexGPT и GigaChat — проверяется не переносом, а eval-набором.

---

## Источники

**Anthropic: юридические страницы и справка**
- https://www.anthropic.com/legal/consumer-terms
- https://www.anthropic.com/legal/commercial-terms
- https://www.anthropic.com/legal/aup
- https://code.claude.com/docs/en/legal-and-compliance.md
- https://github.com/anthropics/claude-code/blob/main/LICENSE.md
- https://support.claude.com/en/articles/15036540-use-the-claude-agent-sdk-with-your-claude-plan
- https://support.claude.com/en/articles/17154008-monthly-api-credits-for-max-and-team-plans
- https://support.claude.com/en/articles/11145838-using-claude-code-with-your-pro-or-max-plan
- https://www.anthropic.com/news/skills
- https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills

**Документация Claude Code**
- https://code.claude.com/docs/en/llm-gateway.md
- https://code.claude.com/docs/en/gateways.md
- https://code.claude.com/docs/en/llm-gateway-protocol.md
- https://code.claude.com/docs/en/llm-gateway-rollout.md
- https://code.claude.com/docs/en/feature-availability.md
- https://code.claude.com/docs/en/authentication.md
- https://code.claude.com/docs/en/headless.md
- https://code.claude.com/docs/en/routines.md
- https://code.claude.com/docs/en/scheduled-tasks.md
- https://code.claude.com/docs/en/desktop-scheduled-tasks.md
- https://code.claude.com/docs/en/remote-control.md
- https://code.claude.com/docs/en/channels.md
- https://code.claude.com/docs/en/github-actions.md
- https://code.claude.com/docs/en/env-vars.md
- https://code.claude.com/docs/en/model-config.md
- https://code.claude.com/docs/en/data-usage.md
- https://code.claude.com/docs/en/tools-reference.md
- https://code.claude.com/docs/en/network-config.md
- https://code.claude.com/docs/en/setup.md
- https://code.claude.com/docs/en/sub-agents.md
- https://code.claude.com/docs/en/skills.md
- https://code.claude.com/docs/en/memory.md
- https://code.claude.com/docs/en/hooks.md
- https://code.claude.com/docs/en/permission-modes.md
- https://code.claude.com/docs/en/plugins/overview.md
- https://code.claude.com/docs/en/agent-sdk/overview.md
- https://code.claude.com/docs/en/changelog.md

**Реестры**
- https://registry.npmjs.org/@anthropic-ai/claude-code
- https://registry.npmjs.org/opencode-ai
- https://registry.npmjs.org/@google/gemini-cli
- https://registry.npmjs.org/@openai/codex
- https://registry.npmjs.org/@qwen-code/qwen-code
- https://registry.npmjs.org/@charmland/crush
- https://registry.npmjs.org/@kilocode/cli
- https://registry.npmjs.org/@musistudio/claude-code-router
- https://pypi.org/pypi/gpt2giga/json
- https://pypi.org/pypi/litellm/json
- https://pypi.org/pypi/claude-agent-sdk/json

**Стандарты**
- https://github.com/agentskills/agentskills
- https://github.com/agentskills/agentskills/blob/main/docs/specification.mdx
- https://github.com/agentskills/agentskills/blob/main/docs/snippets/clients.jsx
- https://github.com/agentsmd/agents.md
- InfoQ (по сниппету): https://infoq.com/news/2025/12/agentic-ai-foundation/
- It's FOSS (по сниппету): https://itsfoss.com/news/agentic-ai-foundation-launch/
- AAIF blog (по сниппету): https://aaif.io/blog/from-skills-and-tools-to-portable-agent-plugins
- Google Developers Blog (по сниппету): https://developers.googleblog.com/agent-plugins-package-your-skills-tools-and-more/

**OpenCode**
- https://github.com/anomalyco/opencode
- PR #18186: https://github.com/sst/opencode/pull/18186
- Документация в репозитории: https://github.com/anomalyco/opencode/tree/dev/packages/web/src/content/docs (skills.mdx, agents.mdx, commands.mdx, plugins.mdx, rules.mdx, cli.mdx, providers.mdx, mcp-servers.mdx)

**Goose**
- https://github.com/block/goose (aaif-goose/goose)
- https://github.com/aaif-goose/goose/releases
- Документация в репозитории: goose-cli-commands.md, context-engineering/{hooks.md, using-skills.md, plugins.md, subagents.mdx}, getting-started/providers.md

**Остальные harness**
- Qwen Code: https://github.com/QwenLM/qwen-code; документация: docs/users/features/{skills, sub-agents, hooks, scheduled-tasks, headless}.md
- Codex: https://github.com/openai/codex; docs/config.md
- Codex (по сниппету): https://developers.openai.com/codex/subagents, https://developers.openai.com/codex/skills, https://developers.openai.com/codex/guides/agents-md, https://openrouter.ai/blog/tutorials/codex-cli-openrouter/, https://codex.danielvaughan.com/2026/04/23/codex-cli-custom-model-providers-configuration-guide/
- Crush: https://github.com/charmbracelet/crush
- Kilo Code: https://github.com/Kilo-Org/kilocode
- Gemini CLI: https://github.com/google-gemini/gemini-cli

**Прокси**
- gpt2giga: https://github.com/ai-forever/gpt2giga, https://github.com/ai-forever/gpt2giga/blob/main/integrations/claude-code/README.md
- claude-code-router: https://github.com/musistudio/claude-code-router
- LiteLLM: https://github.com/BerriAI/litellm/issues/22963
- LiteLLM (по сниппету): https://docs.litellm.ai/docs/tutorials/claude_non_anthropic_models

**Провайдеры моделей**
- https://github.com/deepseek-ai/awesome-deepseek-agent/blob/main/docs/claude_code.md
- https://github.com/MoonshotAI/Kimi-K2
- https://github.com/zai-org/GLM-4.5
- https://github.com/MiniMax-AI/MiniMax-M2
- https://github.com/QwenLM/Qwen3-Coder
- По сниппету:
  - https://api-docs.deepseek.com/guides/agent_integrations/claude_code
  - https://api-docs.deepseek.com/guides/anthropic_api/
  - https://docs.z.ai/devpack/tool/claude
  - https://platform.minimax.io/docs/m-plan/claude-code
  - https://www.alibabacloud.com/help/en/model-studio/claude-code
  - https://aistudio.yandex.ru/docs/en/ai-studio/concepts/api

**Issues и сообщество**
- https://github.com/anthropics/claude-code/issues/34821
- https://github.com/RichardRamirez123/NonAnthropicEndpointFix

**Новости (по сниппету поиска)**
- https://venturebeat.com/ai/anthropic-cracks-down-on-unauthorized-claude-usage-by-third-party-harnesses/
- https://gigazine.net/gsc_news/en/20260220-anthropic-third-party-block
- https://www.techradar.com/pro/bad-news-claude-users-anthropic-says-youll-need-to-pay-to-use-openclaw-now
- https://hn.nuxt.dev/item/47633396
- https://lilting.ch/en/articles/anthropic-claude-code-openclaw-third-party-paygo
- https://gaugr.app/en/blog/claude-billing-overhaul-agent-sdk-credits-june-2026
- https://env.dev/updates/anthropic-agent-sdk-credits
- https://wmedia.es/en/tips/claude-code-agent-sdk-credit
- https://blog.kilo.ai/p/claude-code-source-leak-a-timeline
- https://hackmag.com/news/claude-code-leak
- https://liveinthefuture.org/stories/claude-code-dmca-copyright-paradox.html

**Оплата из РФ (по сниппету поиска; тексты посредников)**
- https://vc.ru/services/3025455-kak-oplatit-deepseek-iz-rossii-i-popolnit-api
- https://vc.ru/services/3045731-kak-oplatit-glm-iz-rossii
- https://vc.ru/services/3056116-kak-kupit-kimi-api-i-podpisku-kimi-code-iz-rossii
