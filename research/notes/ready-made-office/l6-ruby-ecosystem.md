# Слой 6. Ruby/Rails-экосистема для LLM-агентов (LLM-клиенты, агентные фреймворки, MCP SDK) + краткое сравнение с Go

Все метрики — **по состоянию на 2026-10-09**.

Как собирались данные. Версии, даты релизов, загрузки и лицензии брались из rubygems.org API через curl. Звёзды, форки и коммиты — со страниц GitHub через WebFetch. README, документацию и исходники читал через raw.githubusercontent.com (curl). Для Go — proxy.golang.org и pkg.go.dev.

Ограничения среды:
- **WebSearch-бюджет исчерпан в начале сбора**: поиск не выполнялся, поэтому пометок «по сниппету поиска» в заметках нет.
- Не открылись yandex.*, sber.ru, rubyllm.com, platform.openai.com и ruby-toolbox.com. Факты по YandexGPT и GigaChat взяты из официальных репозиториев Yandex (`yandex-cloud/yandex-cloud-ml-sdk`) и Сбера (`ai-forever/*`) на GitHub. Документацию RubyLLM читал из исходников `docs/` в репозитории.
- api.github.com заблокирован, а в HTML GitHub не отображаются ни число контрибьюторов, ни дата создания репозитория. Поэтому контрибьюторы помечены «не проверено», а вместо даты создания дан **первый релиз гема** (это прокси).

---

## 1. Кандидаты: сводная таблица, метрики, шорт-лист

### Takeaway

- **Ядро — RubyLLM 2.1.** Это единственный Ruby-фреймворк, который «из коробки» закрывает агентов, tools, structured output, durable-агентов на ActiveJob, approvals человека, MCP-клиент и **поатомный учёт токенов и стоимости в БД**.
- **MCP-сервер — официальный SDK `mcp` 1.7.** Своих MCP-серверов RubyLLM не строит.
- **Референсы**: ActiveAgent (альтернативная архитектура «агенты = контроллеры»), ruby_llm-agents (бюджеты), DSPy.rb (типизированный скоринг).
- **Отсев**:
  - langchainrb — релизов нет 17 месяцев;
  - fast-mcp — заглох в 2025-09;
  - ruby_llm-mcp — несовместим с RubyLLM 2.x, его заменил нативный MCP-клиент;
  - ruby-openai — релизов нет больше года;
  - anthropic SDK — Anthropic не поддерживает РФ.

### Cited Findings

#### 1.1 Сводная таблица (все проверенные Ruby-кандидаты; первые 6 — финальный шорт-лист)

| # | Кандидат | URL | Лицензия | Стек | Подключение к Rails | Активность | Роли/этапы цикла | РФ: провайдеры | Ключи/секреты | Вердикт |
|---|---|---|---|---|---|---|---|---|---|---|
| 1 | **RubyLLM** (`ruby_llm`) | [github.com/crmne/ruby_llm](https://github.com/crmne/ruby_llm) | MIT ([rg](https://rubygems.org/api/v1/gems/ruby_llm.json)) | Ruby ≥3.2, Faraday, Zeitwerk, Schematist ([rg](https://rubygems.org/api/v1/gems/ruby_llm.json)) | `bundle add ruby_llm` → `bin/rails generate ruby_llm:install` → `db:migrate` → `ruby_llm:load_models`; `acts_as_chat`; опционально `ruby_llm:chat_ui` ([README](https://github.com/crmne/ruby_llm/blob/main/README.md)) | 2.1.0 от 2026-10-08; 20 релизов за 365 дн. ([rg](https://rubygems.org/api/v1/versions/ruby_llm.json)); последний коммит 2026-10-08 ([commits](https://github.com/crmne/ruby_llm/commits/main)) | Все роли через `RubyLLM::Agent`. Tools для обогащения, `with_schema` для скоринга и юнит-экономики, workflow-паттерны (fan-out/fan-in, evaluator-optimizer), `requires_approval` для гейтов человека, учёт затрат ([README](https://github.com/crmne/ruby_llm/blob/main/README.md), [workflows](https://github.com/crmne/ruby_llm/blob/main/docs/_advanced/agentic-workflows.md)) | Любой OpenAI-совместимый эндпоинт через `openai_api_base` + `assume_model_exists` ([docs](https://github.com/crmne/ruby_llm/blob/main/docs/_getting_started/configuration-providers.md)). Нативно — DeepSeek, OpenRouter, Ollama, GPUStack ([README](https://github.com/crmne/ruby_llm/blob/main/README.md)). Нативных YandexGPT/GigaChat нет | Ключи задаются в `RubyLLM.configure`/ENV; `RubyLLM.context` даёт изолированную копию на тенант ([config.rb](https://github.com/crmne/ruby_llm/blob/main/lib/ruby_llm/configuration.rb)). OAuth-креды MCP шифруются в таблице `ruby_llm_mcp_credentials` (AR encryption) ([upgrading](https://github.com/crmne/ruby_llm/blob/main/docs/_reference/upgrading.md)) | **брать** (ядро) |
| 2 | **MCP Ruby SDK** (`mcp`) | [github.com/modelcontextprotocol/ruby-sdk](https://github.com/modelcontextprotocol/ruby-sdk) | Apache-2.0 для нового кода, MIT для существующего ([README](https://github.com/modelcontextprotocol/ruby-sdk)) | Ruby ≥2.7, единственная runtime-зависимость `json_schemer` ([rg](https://rubygems.org/api/v1/gems/mcp.json)) | `StreamableHTTPTransport` — это Rack-приложение: монтируется в routes.rb или вызывается из контроллера в stateless-режиме ([docs](https://ruby.sdk.modelcontextprotocol.io/server/transports/)) | 1.7.0 от 2026-10-05; 35 релизов за 365 дн. ([rg](https://rubygems.org/api/v1/versions/mcp.json)); коммит 2026-10-08 ([commits](https://github.com/modelcontextprotocol/ruby-sdk/commits/main)) | Слой инструментов: Wordstat/Direct/ГИР БО/ЮKassa как MCP-tools для агентов и для Claude Code | Не зависит от LLM-провайдера | Транспорт сам не аутентифицирует: identity передаётся через `server_context`, сессии проверяет `session_request_validator`, есть `allowed_hosts/origins` ([docs](https://ruby.sdk.modelcontextprotocol.io/server/transports/)). OAuth 2.1 в клиенте ([README](https://github.com/modelcontextprotocol/ruby-sdk)) | **брать** (MCP-сервер) |
| 3 | **ActiveAgent** (`activeagent`) | [github.com/activeagents/activeagent](https://github.com/activeagents/activeagent) | MIT ([rg](https://rubygems.org/api/v1/gems/activeagent.json)) | Rails-компоненты ≥7.2 и ≤9.0 + гем провайдера (`openai`/`anthropic`/`ruby_llm`) ([rg](https://rubygems.org/api/v1/gems/activeagent.json), [README](https://github.com/activeagents/activeagent/blob/main/README.md)) | `rails generate active_agent:install` создаёт `config/active_agent.yml` и `ApplicationAgent`; есть `active_agent:agent`. Хранение — отдельный гем `solid_agent`, дашборд — `actionagent` ([README](https://github.com/activeagents/activeagent/blob/main/README.md)) | 1.8.1 от 2026-10-01; 17 релизов за 90 дн. ([rg](https://rubygems.org/api/v1/versions/activeagent.json)); коммит 2026-10-08 ([commits](https://github.com/activeagents/activeagent/commits/main)) | Роли — агенты-контроллеры с action'ами и ERB-промптами; делегирование с бюджетами; `generate_later` | OpenAI `host` + `api_version: :chat` ([docs](https://github.com/activeagents/activeagent/blob/main/docs/providers/open_ai.md)); DeepSeek, Ollama (`host`), OpenRouter (`uri_base`); провайдер RubyLLM с `protocol:` ([docs](https://github.com/activeagents/activeagent/blob/main/docs/providers/ruby_llm.md)) | `access_token` в `config/active_agent.yml` ([README](https://github.com/activeagents/activeagent/blob/main/README.md)); MCP `require_approval` ([docs](https://github.com/activeagents/activeagent/blob/main/docs/actions/mcps.md)) | **референс** (альтернатива RubyLLM; два фреймворка сразу не тянуть) |
| 4 | **ruby_llm-agents** | [github.com/adham90/ruby_llm-agents](https://github.com/adham90/ruby_llm-agents) | MIT ([rg](https://rubygems.org/api/v1/gems/ruby_llm-agents.json)) | Rails engine (rails ≥7.0, ruby_llm ≥1.16.0) ([rg](https://rubygems.org/api/v1/gems/ruby_llm-agents.json)) | Engine + монтируемый дашборд; генератор `ruby_llm_agents:upgrade` ([CHANGELOG](https://github.com/adham90/ruby_llm-agents/blob/main/CHANGELOG.md)) | 3.16.0 от 2026-10-09; 55 релизов с 2025-11-25 ([rg](https://rubygems.org/api/v1/versions/ruby_llm-agents.json)) | Бюджеты (daily/monthly, hard/soft), cost analytics по агенту/модели/тенанту, circuit breakers, алерты ([README](https://github.com/adham90/ruby_llm-agents/blob/main/README.md)) | Через RubyLLM | API-ключи на тенант ([README](https://github.com/adham90/ruby_llm-agents/blob/main/README.md)) | **референс** (дизайн бюджетов) |
| 5 | **DSPy.rb** (`dspy`) | [github.com/vicentereig/dspy.rb](https://github.com/vicentereig/dspy.rb) | MIT ([rg](https://rubygems.org/api/v1/gems/dspy.json)) | Ruby ≥3.3, sorbet-runtime, async ([rg](https://rubygems.org/api/v1/gems/dspy.json)) | Обычный гем + адаптер-гем провайдера, без Rails-генераторов ([README](https://github.com/vicentereig/dspy.rb/blob/main/README.md)) | 1.0.2 от 2026-07-18; 17 релизов за 365 дн. ([rg](https://rubygems.org/api/v1/versions/dspy.json)) | Типизированные сигнатуры для фильтра, скоринга и турнира; ReAct с лимитом итераций; оптимизаторы ([README](https://github.com/vicentereig/dspy.rb/blob/main/README.md)) | Ollama-адаптер принимает `base_url:` (OpenAI-совместимый, [src](https://github.com/vicentereig/dspy.rb/blob/main/lib/dspy/openai/lm/adapters/ollama_adapter.rb)). OpenAI-адаптер создаёт `OpenAI::Client.new(api_key:)` ([src](https://github.com/vicentereig/dspy.rb/blob/main/lib/dspy/openai/lm/adapters/openai_adapter.rb)). `dspy-ruby_llm` требует ruby_llm <2.0 ([rg](https://rubygems.org/api/v1/gems/dspy-ruby_llm.json)) | `api_key` передаётся в адаптер ([src](https://github.com/vicentereig/dspy.rb/blob/main/lib/dspy/openai/lm/adapters/openai_adapter.rb)) | **референс** (опционально для скоринга) |
| 6 | **openai-ruby** (`openai`, официальный) | [github.com/openai/openai-ruby](https://github.com/openai/openai-ruby) | Apache-2.0 ([rg](https://rubygems.org/api/v1/gems/openai.json)) | Ruby ≥3.3 ([rg](https://rubygems.org/api/v1/versions/openai.json)) | Обычный клиент | 0.102.0 от 2026-10-09; 79 релизов за 365 дн.; всё ещё 0.x ([rg](https://rubygems.org/api/v1/versions/openai.json)) | Низкоуровневый клиент | `base_url:` (по умолчанию `ENV["OPENAI_BASE_URL"]`), `project:`, `default_headers:` ([client.rb](https://github.com/openai/openai-ruby/blob/main/lib/openai/client.rb)) | Ключи из ENV по умолчанию ([client.rb](https://github.com/openai/openai-ruby/blob/main/lib/openai/client.rb)) | **резерв** (как зависимость ActiveAgent/DSPy) |
| 7 | ruby_llm-mcp | [github.com/patvice/ruby_llm-mcp](https://github.com/patvice/ruby_llm-mcp) | MIT | Требует `ruby_llm ~> 1.9`, `httpx` ([rg](https://rubygems.org/api/v1/gems/ruby_llm-mcp.json)) | `rails generate ruby_llm:mcp:install` ([GitHub](https://github.com/patvice/ruby_llm-mcp)) | 1.0.1 от 2026-07-21; коммиты до 2026-10-08 ([commits](https://github.com/patvice/ruby_llm-mcp/commits/main)) | MCP-клиент | — | OAuth 2.1 + PKCE ([GitHub](https://github.com/patvice/ruby_llm-mcp)) | **нет** (несовместим с RubyLLM 2.x; в 2.1 есть нативный MCP-клиент) |
| 8 | langchainrb | [github.com/patterns-ai-core/langchainrb](https://github.com/patterns-ai-core/langchainrb) | MIT | Ruby ≥3.1 | Через `langchainrb_rails` 0.1.12 (2024-09-20), который пинит `langchainrb < 0.17` ([rg](https://rubygems.org/api/v1/gems/langchainrb_rails.json)) | Последний релиз 0.19.5 от 2025-05-01, за 365 дн. релизов нет ([rg](https://rubygems.org/api/v1/versions/langchainrb.json)); коммиты до 2026-09-09 ([commits](https://github.com/patterns-ai-core/langchainrb/commits/main)) | Assistant, RAG | OpenAI, Gemini, Anthropic, Mistral, Ollama ([README](https://github.com/patterns-ai-core/langchainrb/blob/main/README.md)) | — | **нет** |
| 9 | fast-mcp | [github.com/yjacquin/fast-mcp](https://github.com/yjacquin/fast-mcp) | MIT | Rack, dry-schema ([rg](https://rubygems.org/api/v1/gems/fast-mcp.json)) | `bin/rails generate fast_mcp:install`, `FastMcp.mount_in_rails` ([README](https://github.com/yjacquin/fast_mcp/blob/main/README.md)) | 1.6.0 от 2025-09-28; последний коммит 2025-09-28 ([commits](https://github.com/yjacquin/fast-mcp/commits/main)) | MCP-сервер | — | Опция `authenticate:` ([README](https://github.com/yjacquin/fast_mcp/blob/main/README.md)) | **нет** (заглох; есть официальный SDK) |
| 10 | Raix | [github.com/OlympiaAI/raix](https://github.com/OlympiaAI/raix) | MIT | `ruby_llm ~> 2.0`, activesupport ([rg](https://rubygems.org/api/v1/gems/raix.json)) | Модули подключаются в Ruby-классы ([README](https://github.com/OlympiaAI/raix/blob/main/README.md)) | 3.0.0 от 2026-09-22 ([rg](https://rubygems.org/api/v1/versions/raix.json)) | Дискретные AI-компоненты (ChatCompletion, хуки) | Через RubyLLM/OpenRouter ([README](https://github.com/OlympiaAI/raix/blob/main/README.md)) | — | **референс** |
| 11 | Roast (`roast-ai`, Shopify) | [github.com/Shopify/roast](https://github.com/Shopify/roast) | MIT | activesupport ~>8.0, async, ruby_llm ≥1.13; Ruby ≥3.3 ([rg](https://rubygems.org/api/v1/gems/roast-ai.json)) | Не Rails: CLI `bin/roast execute` ([README](https://github.com/Shopify/roast/blob/main/README.md)) | 1.3.0 от 2026-10-01 ([rg](https://rubygems.org/api/v1/versions/roast-ai.json)) | Workflow DSL: когсы `chat`/`agent` (Claude Code CLI)/`cmd`/`map`/`repeat` ([README](https://github.com/Shopify/roast/blob/main/README.md)) | `OPENAI_API_BASE` для chat-кога ([README](https://github.com/Shopify/roast/blob/main/README.md)) | Ключи в ENV | **референс** (dev-автоматизация) |
| 12 | anthropic-sdk-ruby (`anthropic`) | [github.com/anthropics/anthropic-sdk-ruby](https://github.com/anthropics/anthropic-sdk-ruby) | MIT | Ruby ≥3.2 ([rg](https://rubygems.org/api/v1/versions/anthropic.json)) | Обычный клиент | 1.77.1 от 2026-10-08; 78 релизов за 365 дн. ([rg](https://rubygems.org/api/v1/versions/anthropic.json)) | Клиент Claude | Россия не входит в список поддерживаемых стран ([anthropic.com](https://www.anthropic.com/supported-countries)) | `base_url` (по умолчанию `ENV["ANTHROPIC_BASE_URL"]`) ([client.rb](https://github.com/anthropics/anthropic-sdk-ruby/blob/main/lib/anthropic/client.rb)) | **нет** для прод-контура в РФ |
| 13 | ruby-openai (community) | [github.com/alexrudall/ruby-openai](https://github.com/alexrudall/ruby-openai) | MIT | Faraday | — | 8.3.0 от 2025-08-29, за 365 дн. релизов нет ([rg](https://rubygems.org/api/v1/versions/ruby-openai.json)) | — | — | — | **нет** |
| 14 | gigachat-ruby | [github.com/amdest/gigachat-ruby](https://github.com/amdest/gigachat-ruby) | MIT ([rg](https://rubygems.org/api/v1/gems/gigachat-ruby.json)) | faraday, httpx ([rg](https://rubygems.org/api/v1/gems/gigachat-ruby.json)) | Обычный клиент | 0.1.0–0.1.2 от 2026-10-02…04, 526 загрузок ([rg](https://rubygems.org/api/v1/versions/gigachat-ruby.json)) | Клиент GigaChat | Нативный GigaChat (см. §3) | Сам управляет OAuth-токеном, вшит Russian Trusted Root CA ([README](https://github.com/amdest/gigachat-ruby/blob/main/README.md)) | **референс** для своего провайдера |

#### 1.2 Метрики зрелости и аномалии

Источники по каждой строке: rubygems — `https://rubygems.org/api/v1/versions/<gem>.json` и `https://rubygems.org/api/v1/gems/<gem>.json`; GitHub — страница репо и `/commits/main`.

| Гем | Посл. версия (дата) | 1.0 (дата) | Релизов 90 / 365 дн. | Загрузки (всего) | ★ / forks / watchers / коммиты | Последний коммит | Первый релиз гема | Аномалии |
|---|---|---|---|---|---|---|---|---|
| ruby_llm | 2.1.0 (2026-10-08) | 1.0.0 (2025-03-11); 2.0.0 (2026-09-18) | 6 / 20 | 13 646 258 | 4.4k / 510 / 33 / 1 804 ([GitHub](https://github.com/crmne/ruby_llm)) | 2026-10-08, crmne ([commits](https://github.com/crmne/ruby_llm/commits/main)) | 0.1.0.pre (2025-01-30) | Мажор 2.0 вышел три недели назад, ломающий: OpenAI по умолчанию на Responses API, Rails <8.1.4 требует `json < 3` ([releases](https://github.com/crmne/ruby_llm/releases)). Плагины отстают — см. Inferences. Все последние коммиты от одного автора ([commits](https://github.com/crmne/ruby_llm/commits/main)) |
| mcp | 1.7.0 (2026-10-05) | 1.0.0 (2026-07-24) | 12 / 35 | 11 175 616 | 923 / 136 / 16 / 1 001 ([GitHub](https://github.com/modelcontextprotocol/ruby-sdk)) | 2026-10-08, koic ([commits](https://github.com/modelcontextprotocol/ruby-sdk/commits/main)) | 0.1.0 (2025-05-30) | Лицензия сменилась MIT → Apache-2.0 с 0.6.0 (2026-01-16) ([rg](https://rubygems.org/api/v1/versions/mcp.json)). Загрузок очень много при 923★ |
| activeagent | 1.8.1 (2026-10-01) | 1.0.0 (2025-11-21) | 17 / 22 | 180 172 | 973 / 89 / 14 / 1 242 ([GitHub](https://github.com/activeagents/activeagent)) | 2026-10-08; среди 8 последних коммитов 3 автора: TonsOfFun, claude, codex ([commits](https://github.com/activeagents/activeagent/commits/main)) | 0.0.0 (2024-03-26) | Темп 1.0.2 → 1.8.1 за 2026-06…10. Коммиты массово пишут ИИ-агенты, человек-мейнтейнер один. Рядом коммерческая платформа activeagents.ai ([README](https://github.com/activeagents/activeagent/blob/main/README.md)) |
| ruby_llm-agents | 3.16.0 (2026-10-09) | 1.0.0 (2026-01-22) | 4 / 55 | 26 351 | 140 / 10 / 3 / 599 ([GitHub](https://github.com/adham90/ruby_llm-agents)) | не проверено | 0.1.0 (2025-11-25) | Три мажора меньше чем за год. В Unreleased чинят недоучёт токенов и стоимости в tool loops ([CHANGELOG](https://github.com/adham90/ruby_llm-agents/blob/main/CHANGELOG.md)) |
| dspy | 1.0.2 (2026-07-18) | 1.0.0 (2026-04-11) | 1 / 17 | 141 933 | 237 / 24 / 4 / 1 229 ([GitHub](https://github.com/vicentereig/dspy.rb)) | не проверено | 0.1.0 (2025-03-15) | В метаданных гема нет `source_code_uri` ([rg](https://rubygems.org/api/v1/gems/dspy.json)) |
| openai | 0.102.0 (2026-10-09) | нет (0.x) | 35 / 79 | 2 373 857 | 466 / 67 / 9 / 1 116 ([GitHub](https://github.com/openai/openai-ruby)) | не проверено | 0.1.0 (2020-07-20) — тогда это был другой автор | Имя гема до 2025 принадлежало Nilesh Trivedi (0.1–0.3, 2020–2023), официальные версии идут с 2025-05-22. Поэтому загрузки смешанные ([rg](https://rubygems.org/api/v1/versions/openai.json)) |
| ruby_llm-mcp | 1.0.1 (2026-07-21) | 1.0.0 (2026-02-23) | 1 / 8 | 329 317 | 269 / 42 / 3 / 395 ([GitHub](https://github.com/patvice/ruby_llm-mcp)) | 2026-10-08 ([commits](https://github.com/patvice/ruby_llm-mcp/commits/main)) | 0.0.1 (2025-05-22) | Пинит `ruby_llm ~> 1.9`, то есть не ставится с RubyLLM 2.x ([rg](https://rubygems.org/api/v1/gems/ruby_llm-mcp.json)) |
| langchainrb | 0.19.5 (2025-05-01) | нет | 0 / 0 | 1 663 472 | 2.0k / 265 / 38 / 931 ([GitHub](https://github.com/patterns-ai-core/langchainrb)) | 2026-09-09 ([commits](https://github.com/patterns-ai-core/langchainrb/commits/main)) | 0.1.3 (2023-05-01) | Коммиты идут, релизов 17 мес. нет. В Unreleased лежат BREAKING-изменения ([CHANGELOG](https://github.com/patterns-ai-core/langchainrb/blob/main/CHANGELOG.md)) |
| fast-mcp | 1.6.0 (2025-09-28) | 1.0.0 (2025-03-30) | 0 / 0 | 2 793 774 | 1.2k / 107 / 16 / 73 ([GitHub](https://github.com/yjacquin/fast-mcp)) | 2025-09-28 ([commits](https://github.com/yjacquin/fast-mcp/commits/main)) | 0.1.0 (2025-03-23) | Больше 12 мес. без коммитов |
| raix | 3.0.0 (2026-09-22) | 1.0.0 (2025-06-04) | 2 / 8 | 346 400 | 330 / 30 / 4 / 147 ([GitHub](https://github.com/OlympiaAI/raix)) | не проверено | 0.1.0 (2024-04-03) | Начиная с 3.0 — надстройка над RubyLLM ([README](https://github.com/OlympiaAI/raix/blob/main/README.md)) |
| roast-ai | 1.3.0 (2026-10-01) | 1.0.0 (2026-02-23) | 1 / 14 | 303 982 | 1.3k / 78 / 62 / 896 ([GitHub](https://github.com/Shopify/roast)) | не проверено | 0.1.0 (2025-05-06) | Требует `activesupport ~> 8.0` ([rg](https://rubygems.org/api/v1/gems/roast-ai.json)) |
| anthropic | 1.77.1 (2026-10-08) | 1.0.0 (2025-05-21) | 24 / 78 | 3 835 640 | 372 / 74 / 11 / 1 077 ([GitHub](https://github.com/anthropics/anthropic-sdk-ruby)) | не проверено | 0.0.0 (2023-07-12) — тогда это был Alex Rudall | Имя гема передал @alexrudall ([README](https://github.com/anthropics/anthropic-sdk-ruby/blob/main/README.md)); загрузки смешанные |
| ruby-openai | 8.3.0 (2025-08-29) | 1.0.0 (2021-02-01) | 0 / 0 | 48 015 650 | — | — | 0.1.0 (2020-09-06) | Год без релизов |

Контрибьюторы: на всех страницах GitHub счётчик пуст без JS — **не проверено** ([пример](https://github.com/crmne/ruby_llm)). Дата создания репозитория — **не проверено** (api.github.com недоступен).

#### 1.3 Прочие факты по кандидатам

- **Экосистема RubyLLM**: на первых 4 страницах поиска rubygems по `ruby_llm` нашлось не менее 78 гемов ([поиск](https://rubygems.org/api/v1/search.json?query=ruby_llm)). Среди них:
  - `opentelemetry-instrumentation-ruby_llm` от thoughtbot, 0.7.1 (2026-07-31) ([rg](https://rubygems.org/api/v1/gems/opentelemetry-instrumentation-ruby_llm.json));
  - `ruby_llm-monitoring`, 0.4.0 (2026-06-09), лицензия в метаданных не указана ([rg](https://rubygems.org/api/v1/gems/ruby_llm-monitoring.json));
  - `ruby_llm-schema` 1.0.0 — теперь лишь пробрасывает `RubyLLM::Schema` в `Schematist::Schema` ([поиск](https://rubygems.org/api/v1/search.json?query=ruby_llm)).
- **Совместимость плагинов с RubyLLM 2.x**:
  - `dspy-ruby_llm` 0.1.2 требует `ruby_llm >= 1.14.1, < 2.0` ([rg](https://rubygems.org/api/v1/gems/dspy-ruby_llm.json));
  - `ruby_llm-mcp` требует `~> 1.9` ([rg](https://rubygems.org/api/v1/gems/ruby_llm-mcp.json));
  - `raix` 3.0 уже на `~> 2.0` ([rg](https://rubygems.org/api/v1/gems/raix.json));
  - `ruby_llm-agents` (≥1.16.0) и `roast-ai` (≥1.13) задают открытые ограничения ([rg](https://rubygems.org/api/v1/gems/ruby_llm-agents.json), [rg](https://rubygems.org/api/v1/gems/roast-ai.json)). README ruby_llm-agents про 2.x молчит ([GitHub](https://github.com/adham90/ruby_llm-agents)).
- **Мультиагентность (SwarmSDK)**: `swarm_sdk` 2.7.15 (2026-02-12) — «multi-agent orchestration» на RubyLLM, зависит от `ruby_llm ~> 1.11` ([rg](https://rubygems.org/api/v1/gems/swarm_sdk.json)). Релизов нет с 2026-02.
- **Raix**:
  - извлечён из продукта Olympia;
  - в 3.0 работает поверх RubyLLM, по умолчанию маршрутизирует через OpenRouter ([README](https://github.com/OlympiaAI/raix/blob/main/README.md));
  - есть экспериментальный раздел про MCP-клиент ([GitHub](https://github.com/OlympiaAI/raix));
  - учёт стоимости в README показан как пример `CostTracker` в хуке `before_completion`, считающий оценку «based on message length» ([README](https://github.com/OlympiaAI/raix/blob/main/README.md)).
- **Roast** — это Ruby DSL «structured AI workflows» ([README](https://github.com/Shopify/roast/blob/main/README.md)):
  - когс `agent` запускает локальных кодинг-агентов (Pi CLI, Claude Code CLI);
  - `chat` поддерживает OpenAI, Anthropic, Perplexity, Gemini и Bedrock;
  - базовый URL меняется через `OPENAI_API_BASE`, `ANTHROPIC_API_BASE`, `GEMINI_API_BASE`.
- **DSPy.rb** ([README](https://github.com/vicentereig/dspy.rb/blob/main/README.md)):
  - «1.x — текущая стабильная линия»;
  - про ReAct: «The application still owns tool authorization, side effects, budgets, and errors»;
  - адаптеры провайдеров, оптимизаторы и observability-экспортёры вынесены в отдельные пакеты;
  - `dspy-openai` 1.0.3 требует `openai >= 0.57.0, < 1.0`, `dspy-anthropic` 1.0.6 — `anthropic < 2.0` ([rg](https://rubygems.org/api/v1/gems/dspy-openai.json), [rg](https://rubygems.org/api/v1/gems/dspy-anthropic.json)).
- **openai-ruby**:
  - README: «This package follows Semantic Versioning»;
  - пользователям Ruby 3.2 предлагают оставаться на v0.75.x — «the final compatible release line» ([GitHub](https://github.com/openai/openai-ruby));
  - в клиенте есть `base_url` (по умолчанию `ENV["OPENAI_BASE_URL"]`), `project` (`ENV["OPENAI_PROJECT_ID"]`) и `default_headers` ([client.rb](https://github.com/openai/openai-ruby/blob/main/lib/openai/client.rb)).
- **anthropic-sdk-ruby**:
  - `base_url` по умолчанию берётся из `ENV["ANTHROPIC_BASE_URL"]` ([client.rb](https://github.com/anthropics/anthropic-sdk-ruby/blob/main/lib/anthropic/client.rb));
  - есть хелпер tool runner ([src](https://github.com/anthropics/anthropic-sdk-ruby/blob/main/lib/anthropic/helpers/tools/runner.rb)).
- **langchainrb** ([CHANGELOG](https://github.com/patterns-ai-core/langchainrb/blob/main/CHANGELOG.md)):
  - в Unreleased — фикс «Cap automatic assistant turns to prevent infinite tool execution loops» и BREAKING «Response classes … converted to Rails engine»;
  - README приглашает на платный консалтинг автора ([README](https://github.com/patterns-ai-core/langchainrb/blob/main/README.md)).
- **fast-mcp**: транспорты STDIO, HTTP и SSE; Rails-генератор и `FastMcp.mount_in_rails` ([README](https://github.com/yjacquin/fast_mcp/blob/main/README.md)). Последняя запись CHANGELOG — 1.6.0 от 2025-09-28 ([CHANGELOG](https://github.com/yjacquin/fast_mcp/blob/main/CHANGELOG.md)).

### Inferences

- **Финальный шорт-лист (6):**
  1. RubyLLM — брать;
  2. `mcp` — брать;
  3. ActiveAgent — референс;
  4. ruby_llm-agents — референс;
  5. DSPy.rb — референс/опционально;
  6. официальный `openai` — резерв.
- **Плагины RubyLLM отстают от 2.0.** Мажор 2.0 вышел 2026-09-18, и часть экосистемы ещё на 1.x: `ruby_llm-mcp`, `dspy-ruby_llm`, `swarm_sdk` жёстко пинят <2.0, а у ruby_llm-agents и ruby_llm-monitoring совместимость не заявлена. Поэтому стартовать надо сразу на 2.x и брать только ядро. Плагины подключать после проверки.
- **Загрузки в rubygems — слабый сигнал.** У `mcp` 11.2M загрузок при 923★ — вероятно, это транзитивные зависимости (гипотеза, не проверено). У `openai` и `anthropic` загрузки смешаны с историей чужих гемов под тем же именем.
- **Bus factor.** Ядро RubyLLM и ActiveAgent держатся на одном мейнтейнере каждый (по коммитам). Для соло-фаундера это приемлемо: код открытый (MIT), его можно форкнуть. Но версии нужно пинить (`~> 2.1`) и обновляться по одному релизу.

### Gaps

- Число контрибьюторов и даты создания репозиториев — не проверено: API GitHub заблокирован, а HTML не показывает счётчик.
- Совместимость ruby_llm-agents, ruby_llm-monitoring и roast-ai с RubyLLM 2.x — не проверено, нужно смотреть их Gemfile.lock и CI.
- Последний коммит у raix, roast, dspy.rb, openai-ruby, anthropic-sdk-ruby и ruby_llm-agents — не проверено (страницы коммитов не открывал). Активность оценена по датам релизов, и у всех кроме dspy она не старше 2026-09.

---

## 2. Глубокий разбор топ-3

### 2.1 RubyLLM (`ruby_llm`, crmne/ruby_llm)

#### Takeaway

RubyLLM 2.1 покрывает почти весь «агентный рантайм» под Rails:
- агенты, tools с approval человека, structured output;
- workflow-паттерны и durable-исполнение на ActiveJob;
- MCP-клиент;
- журнал использования `ruby_llm_usages` с polymorphic `owner` — основа для бюджетов по агентам;
- OpenTelemetry.

Для РФ подходит через OpenAI-совместимые эндпоинты, но с оговорками по протоколу, авторизации и ценам — см. §3.

#### Cited Findings

**Позиционирование и провайдеры.** RubyLLM называет себя «Ruby-native AI framework» ([README](https://github.com/crmne/ruby_llm/blob/main/README.md)). Заявлены 19 провайдеров: OpenAI, Azure, xAI, Anthropic, Gemini, VertexAI, Bedrock, Cohere, DeepSeek, Hetzner, Mistral, Ollama, Ollama Cloud, OpenRouter, Perplexity, GPUStack, ElevenLabs, Deepgram, TypeSafe «and any OpenAI-compatible API» ([README](https://github.com/crmne/ruby_llm/blob/main/README.md)).

**Tools и approvals.**
- Инструменты описываются классами `RubyLLM::Tool` и подключаются через `chat.with_tools(...)` ([README](https://github.com/crmne/ruby_llm/blob/main/README.md)).
- «Tool approval: Park a run until a human approves with `requires_approval`» ([README](https://github.com/crmne/ruby_llm/blob/main/README.md)).
- Решение о вызове хранится в БД: `chat.approve(id)`, `chat.deny(id)`, `chat.pending_approvals` ([durable-agents](https://github.com/crmne/ruby_llm/blob/main/docs/_advanced/durable-agents.md)).

**Structured output.** `chat.with_schema(ProductSchema)`, затем `response.parsed`; схемы описываются на `Schematist::Schema` ([README](https://github.com/crmne/ruby_llm/blob/main/README.md)).

**Streaming.** Ответ стримится блоком в `ask`. В Rails — через Hotwire ([README](https://github.com/crmne/ruby_llm/blob/main/README.md), [rails](https://github.com/crmne/ruby_llm/blob/main/docs/_advanced/rails.md)).

**Агенты.**
- `class X < RubyLLM::Agent` с DSL `model`, `instructions`, `tools` ([README](https://github.com/crmne/ruby_llm/blob/main/README.md)).
- Агент поверх записи в БД объявляется через `chat_model Chat`, а `StudyAgent.find(chat_id).ask(...)` можно звать из ActiveJob ([rails](https://github.com/crmne/ruby_llm/blob/main/docs/_advanced/rails.md)).

**Цикл агента.**
- Цикл можно вести вручную глаголами `generate`, `run_tools`, `step`, `complete`, `complete?` и `ask_later`. Так делают «iteration budget», «one turn per job» и остановку с возобновлением на другой машине.
- Пока цикл ждёт, проверяются `waiting?`, `awaiting_approval?`, `awaiting_input?` и `awaiting_tasks?`.
- Источник: [agentic-workflows](https://github.com/crmne/ruby_llm/blob/main/docs/_advanced/agentic-workflows.md).

**Мультиагентные паттерны** в документации: Sequential, Routing, Agent Handoffs, Parallel, Fan-Out/Fan-In, Evaluation Loop (Evaluator-Optimizer) ([agentic-workflows](https://github.com/crmne/ruby_llm/blob/main/docs/_advanced/agentic-workflows.md)). `RubyLLM.workflow` связывает мультиагентные прогоны в телеметрии ([README](https://github.com/crmne/ruby_llm/blob/main/README.md)).

**Durable-агенты** ([durable-agents](https://github.com/crmne/ruby_llm/blob/main/docs/_advanced/durable-agents.md)):
- «The Transcript Is the State»: сохранённый транскрипт сам является состоянием;
- можно исполнять «One Turn per Job» или весь цикл одной job на `ActiveJob::Continuable` (Rails 8.1+) с `checkpoint!` после каждого шага, и тогда job переживает деплой;
- `chat.cancel` пишет флаг отмены в БД;
- гарантия — «At-Least-Once, Not Exactly-Once»: инструменты должны быть идемпотентны (`find_or_create_by!`, idempotency keys).

**MCP-клиент (2.1).** «RubyLLM is an MCP client… It does not build MCP servers» ([mcp](https://github.com/crmne/ruby_llm/blob/main/docs/_core_features/mcp.md)). Что умеет, по тому же документу:
- сервер описывается подклассом `RubyLLM::MCP`;
- транспорты: `url` (Streamable HTTP), `command` (stdio) и собственные;
- фильтр, переименование и обёртка инструментов, `requires_approval`;
- ресурсы, промпты, input requests, прогресс и отмена, подписка на изменения;
- расширения MCP Apps и Tasks;
- OAuth, включая enterprise SSO;
- поддержка ревизии протокола 2026-07-28.

Release notes 2.1.0 описывают MCP-клиент с «OAuth, approvals, and resumable background tasks» ([releases](https://github.com/crmne/ruby_llm/releases)).

**Rails-интеграция.**
- Генераторы `ruby_llm:install`, `ruby_llm:chat_ui`, `ruby_llm:load_models` ([README](https://github.com/crmne/ruby_llm/blob/main/README.md)).
- Таблицы ([rails-persistence](https://github.com/crmne/ruby_llm/blob/main/docs/_advanced/rails-persistence.md)):
  - `ruby_llm_models`;
  - `ruby_llm_tool_calls`;
  - `ruby_llm_usages`;
  - `ruby_llm_batches`;
  - `ruby_llm_mcp_credentials`;
  - `ruby_llm_provider_files`.
- Через `acts_as_chat` / `acts_as_message` можно задать отдельную ассоциацию для «LLM-транскрипта» и пользовательского ([rails-persistence](https://github.com/crmne/ruby_llm/blob/main/docs/_advanced/rails-persistence.md)).
- В 2.1 появились «A Secondary Database for RubyLLM» и «Agent Configuration» ([whats-new-2.1](https://github.com/crmne/ruby_llm/blob/main/docs/_getting_started/whats-new-in-2-1.md)).

**Учёт токенов и стоимости** ([cost](https://github.com/crmne/ruby_llm/blob/main/docs/_core_features/cost-and-usage-tracking.md)):
- «per-attempt usage ledger»: каждая физическая попытка (ретраи, fallback'и, отменённые стримы) пишется в `ruby_llm_usages` сразу, до колбэка сообщения;
- в строке нормализованные числовые колонки: operation, provider, model, status, token buckets, cost components, timestamps;
- цена «замораживается» в момент завершения попытки;
- если usage неизвестен, токены и `cost.total` равны `nil`, а не 0;
- если для использованных токенов неполны цены, `cost.total` возвращает `nil`.

**Owner и события.** Ledger имеет polymorphic `owner`: `has_many :ruby_llm_usages, as: :owner` и `current_user.ruby_llm_usages.sum(:total_cost)`. Каждая попытка шлёт событие `usage.ruby_llm` ([rails-persistence](https://github.com/crmne/ruby_llm/blob/main/docs/_advanced/rails-persistence.md), [upgrading](https://github.com/crmne/ruby_llm/blob/main/docs/_reference/upgrading.md)).

**Наблюдаемость.**
- События ActiveSupport::Notifications ([instrumentation](https://github.com/crmne/ruby_llm/blob/main/docs/_advanced/instrumentation.md)):
  - `chat.ruby_llm`, `tool_call.ruby_llm`, `usage.ruby_llm` (статус, токены, стоимость, owner);
  - `workflow.ruby_llm`, `workflow_step.ruby_llm`;
  - `evaluation.ruby_llm`, `request.ruby_llm` и другие.
- OpenTelemetry в 2.1 опционален, «Spans omit prompts and responses» ([releases](https://github.com/crmne/ruby_llm/releases)).

**Evaluations и Judge.**
- `RubyLLM::Evaluation` (датасеты, RSpec/Minitest, `report.cost.total`) и `RubyLLM::Judge` (вероятности, выбор, оценки) входят в 2.1 по [What's New in 2.1](https://github.com/crmne/ruby_llm/blob/main/docs/_getting_started/whats-new-in-2-1.md) и [release notes](https://github.com/crmne/ruby_llm/releases). **Противоречие:** README на main помечает Evaluations как «available on `main`» ([README](https://github.com/crmne/ruby_llm/blob/main/README.md)).
- Модель для judgments по умолчанию — `jev-latest` от провайдера TypeSafe ([configuration.rb](https://github.com/crmne/ruby_llm/blob/main/lib/ruby_llm/configuration.rb)). Альтернативы — OpenAI Decisions или локальный Clef через Ollama ([whats-new-2.1](https://github.com/crmne/ruby_llm/blob/main/docs/_getting_started/whats-new-in-2-1.md)).

**Надёжность.** Конфиг по умолчанию ([configuration.rb](https://github.com/crmne/ruby_llm/blob/main/lib/ruby_llm/configuration.rb)):
- `request_timeout` 300;
- `max_retries` 3 с backoff;
- `http_proxy`;
- `tool_concurrency`.

Кроме того, есть `with_fallbacks` (fallback-модели) и `cancel` ([README](https://github.com/crmne/ruby_llm/blob/main/README.md)).

**Кастомные провайдеры** ([custom-providers](https://github.com/crmne/ruby_llm/blob/main/docs/_reference/custom-providers.md)):
- провайдер (хост, auth-заголовки, конфиг, каталог) отделён от протокола (wire format);
- для сервиса на OpenAI Chat Completions пишется только провайдер: «This is the common case, and it is a few dozen lines»;
- генератор отдельного гема — `bundle exec ruby_llm provider-gem Acme --api-base …`;
- приоритет выбора протокола: явный `protocol:` у чата, затем конфиг `<provider>_protocol`, затем хук `protocol_for`.

**Разработка с Claude Code.** Гем содержит Agent Skill для кодинг-ассистентов: `npx skills add "$(bundle show ruby_llm)" --skill rubyllm` ([ai-coding-assistants](https://github.com/crmne/ruby_llm/blob/main/docs/_getting_started/ai-coding-assistants.md)).

**Зрелость.**
- 1.0.0 вышел 2025-03-11, 2.0.0 — 2026-09-18, 2.1.0 — 2026-10-08 ([rg](https://rubygems.org/api/v1/versions/ruby_llm.json)).
- 2.1 требует Ruby 3.2+ и предписывает «Upgrades, One Release at a Time»: приложения на 1.x сначала переходят на 2.0 ([releases](https://github.com/crmne/ruby_llm/releases), [whats-new-2.1](https://github.com/crmne/ruby_llm/blob/main/docs/_getting_started/whats-new-in-2-1.md)).
- 1.16.0 (2026-06-09) добавил кастомные base URL для всех нативных провайдеров ([releases](https://github.com/crmne/ruby_llm/releases)).

**Мейнтейнер.** Carmine Paolino ([rg](https://rubygems.org/api/v1/gems/ruby_llm.json)). README: «Battle tested at Chat with Work» ([README](https://github.com/crmne/ruby_llm/blob/main/README.md)).

#### Inferences

**Соответствие этапам цикла:**

| Этап | Чем закрывается |
|---|---|
| Скаутинг ~50 идей | Fan-Out/Fan-In через `RubyLLM::Agent`; каждую идею — своей job |
| Фильтрация и юнит-экономика | `with_schema` (типизированные оценки) |
| Обогащение | Свои `RubyLLM::Tool` или MCP-tools поверх Wordstat/Direct/ГИР БО |
| Турнир | Evaluator-Optimizer или попарные сравнения через structured output. `Judge` по умолчанию завязан на TypeSafe/OpenAI Decisions, в РФ его надо переопределять |
| Гейты человека (бюджет, домены, legal, GO) | `requires_approval` + `pending_approvals` в Rails-UI |
| Бюджеты агентов | `ruby_llm_usages.owner` = агент или прогон недели, суммы через SQL, контроль по событию `usage.ruby_llm` |

- **Бюджет надо принуждать самому.** RubyLLM учитывает расходы, но лимит сам не применяет: в просмотренных доках нет механизма «стоп при превышении». Нужен свой guard перед `step`.
- **Durable-модель «one turn per job» хорошо ложится на Solid Queue.** Недельный цикл можно резюмировать после сбоя или деплоя. Обязательное условие — идемпотентные инструменты, иначе возможны двойные вызовы Direct/ЮKassa.

#### Gaps

- Работа `RubyLLM::Judge` с YandexGPT или GigaChat — не проверено.
- Есть ли в RubyLLM встроенное принудительное ограничение бюджета — не найдено в просмотренных доках (cost-and-usage-tracking, agents, durable-agents).
- Можно ли задать собственные цены (в рублях) для моделей вне реестра — не проверено. Документ гарантирует только `nil` для неизвестных цен.

### 2.2 MCP Ruby SDK (`mcp`, modelcontextprotocol/ruby-sdk)

#### Takeaway

Это официальный SDK: и сервер, и клиент, версия 1.x с июля 2026 и активной разработкой. Он нужен, чтобы публиковать собственные инструменты (Wordstat, Direct, ГИР БО, ЮKassa, выборки из БД) как MCP-сервер прямо из Rails. Тогда их могут вызывать Claude Code при разработке и эксплуатации, агенты RubyLLM через MCP-клиент и любые другие хосты.

#### Cited Findings

**Возможности** ([README](https://github.com/modelcontextprotocol/ruby-sdk)):
- описание: «The official Ruby SDK for Model Context Protocol servers and clients»;
- серверы с tools, prompts и resources;
- клиенты с «automatic lifecycle negotiation and OAuth 2.1 authorization»;
- транспорты: stdio и Streamable HTTP (включая SSE) и «a Rails integration»;
- поверхность протокола: server-to-client requests, notifications, progress, logging, cancellation, completions, pagination.

**API.**
- Сервер: `MCP::Tool` с `input_schema`, `MCP::Server.new(name:, tools:)`, `MCP::Server::Transports::StdioTransport`.
- Клиент: `MCP::Client` с транспортами `MCP::Client::Stdio` и `MCP::Client::HTTP` ([README](https://github.com/modelcontextprotocol/ruby-sdk)).

**Rails** ([docs](https://ruby.sdk.modelcontextprotocol.io/server/transports/)):
- `MCP::Server::Transports::StreamableHTTPTransport` — это Rack-приложение. Его можно смонтировать в `config/routes.rb` (например, `/mcp`; только так обслуживаются SSE-потоки `subscriptions/listen`) или вызывать из контроллера через `handle_request` с `stateless: true` и `serve_subscriptions_listen: false`;
- stateless-режим не выдаёт `Mcp-Session-Id`, подходит для нескольких нод, но не умеет server-to-client запросы (sampling, roots, elicitation);
- сессии по умолчанию живут в памяти: `session_idle_timeout` 1800 с, `max_sessions` 10000; при нескольких инстансах нужны sticky sessions;
- аутентификацию делает приложение: identity передаётся через `server_context`, сессию проверяет `session_request_validator`;
- по умолчанию принимаются только loopback-хосты; для своего домена задаются `allowed_hosts:` и `allowed_origins:`;
- упоминаются ревизии MCP 2025-11-25 и 2026-07-28 (sessionless lifecycle).

**Активность и мейнтейнеры.**
- Последние коммиты (2026-10-05…08) — релиз 1.7.0, OAuth scope selector, редакция заголовка Authorization в сохраняемых ошибках транспорта, отбрасывание не-JSON-RPC 2.0 сообщений ([commits](https://github.com/modelcontextprotocol/ruby-sdk/commits/main)).
- Авторы последних коммитов — koic и atesgoral ([commits](https://github.com/modelcontextprotocol/ruby-sdk/commits/main)). Аффилиация не проверена; README Shopify не упоминает ([GitHub](https://github.com/modelcontextprotocol/ruby-sdk)).
- На rubygems автор указан как «Model Context Protocol» ([rg](https://rubygems.org/api/v1/gems/mcp.json)).

**Зрелость.** 0.1.0 вышел 2025-05-30, 1.0.0 — 2026-07-24, 1.7.0 — 2026-10-05; за 365 дней 35 релизов. Лицензия была MIT до 0.6.0 (2026-01-16), затем Apache-2.0 ([rg](https://rubygems.org/api/v1/versions/mcp.json)).

**Кто ещё опирается на `mcp`.**
- ActiveAgent для client-side MCP-моста требует `gem "mcp"` ([ActiveAgent docs](https://github.com/activeagents/activeagent/blob/main/docs/actions/mcps.md)).
- Адаптер `:mcp_sdk` в ruby_llm-mcp требует `mcp ~> 0.7` ([GitHub](https://github.com/patvice/ruby_llm-mcp)).

#### Inferences

- **Где MCP-сервер не нужен.** Для внутренних агентов в том же Rails-процессе проще писать инструменты как `RubyLLM::Tool`.
- **Где он оправдан:**
  - (а) отдать те же инструменты Claude Code для разработки и ручного ops;
  - (б) переиспользовать их из других процессов или языков, если часть системы будет на Go;
  - (в) централизовать аутентификацию и аудит внешних API.

  Практичный паттерн: одна бизнес-логика (сервис-объекты) и два тонких адаптера — `RubyLLM::Tool` и `MCP::Tool`.
- **Ограничения в проде.** Stateless-режим плюс аутентификация в контроллере — естественный выбор для деплоя на несколько процессов Puma. Но тогда недоступны sampling и elicitation.

#### Gaps

- Полный список поддерживаемых версий протокола — на странице «Protocol Versions»; не открывал.
- Аффилиация мейнтейнеров (Shopify или другая) — не проверено.

### 2.3 ActiveAgent (`activeagent`, activeagents/activeagent)

#### Takeaway

Самая «Rails-образная» архитектура. Агенты здесь — контроллеры с action'ами, промпты — ERB-вьюхи, есть `generate_later` через ActiveJob и стриминг через ActionCable. Но само хранение (`solid_agent` 0.2.x) и дашборд вынесены в отдельные, менее зрелые гемы, а темп релизов и доля ИИ-коммитов очень высоки. Для проекта на RubyLLM это референс по архитектуре, а не второй рантайм.

#### Cited Findings

**Концепция и установка** ([README](https://github.com/activeagents/activeagent/blob/main/README.md)):
- «The only agent-oriented AI framework designed for Rails, where Agents are Controllers» ([rg](https://rubygems.org/api/v1/gems/activeagent.json));
- `ApplicationAgent < ActiveAgent::Base`, `generate_with :openai, model: …`;
- `rails generate active_agent:agent TravelAgent search book confirm`;
- конфиг — `config/active_agent.yml`;
- провайдеры ставятся отдельными гемами (`openai`, `anthropic`, `ruby_llm`).

**Цикл генерации.** Колбэки `before_generation` и `after_generation`, `embed_with` ([framework](https://github.com/activeagents/activeagent/blob/main/docs/framework.md)).

**Фоновое исполнение.** `prompt_later` (алиас `generate_later`) и `embed_later` работают через `ActiveAgent::GenerationJob` с любым адаптером Active Job ([generation](https://github.com/activeagents/activeagent/blob/main/docs/agents/generation.md), [rails](https://github.com/activeagents/activeagent/blob/main/docs/framework/rails.md)).

**Стриминг и structured output.** «Real-time response streaming with ActionCable», JSON-схемы для structured output, tool calling ([README](https://github.com/activeagents/activeagent/blob/main/README.md)).

**MCP** ([mcps](https://github.com/activeagents/activeagent/blob/main/docs/actions/mcps.md)):
- все провайдеры принимают `mcps:` с `url:` (HTTP) или `command:` (stdio);
- у Anthropic и OpenAI Responses remote-сервер исполняет сам провайдер, у остальных провайдеров ActiveAgent запускает сервер сам;
- `require_approval:` переводит сервер в client-side;
- для client-side нужен `gem "mcp"`.

**Мультиагентность и бюджеты** ([delegation](https://github.com/activeagents/activeagent/blob/main/docs/actions/delegation.md)):
- `delegate_to Agent, budget: { max_calls:, max_tokens: }`;
- `delegation_budget max_calls:, max_duration:`;
- `max_cost` в USD требует задать `rates`;
- «Budgets are scoped to one generation».

**Учёт.** Нормализованный `response.usage` (input/output/total tokens, `provider_details`); мониторинг — через ActiveSupport::Notifications ([usage](https://github.com/activeagents/activeagent/blob/main/docs/actions/usage.md)).

**Хранение.**
- «ActiveAgent runs agents; it deliberately doesn't store anything».
- Хранение вынесено в `solid_agent`: разговоры, обмен tools/MCP, генерации с токенами, долговременная память, reasoning traces, «durable run records», оценки стоимости ([solid_agent](https://github.com/activeagents/activeagent/blob/main/docs/solid_agent.md)).
- Версия `solid_agent` — 0.2.1 (2026-10-08), 3 709 загрузок ([rg](https://rubygems.org/api/v1/gems/solid_agent.json)).

**Дашборд и платформа.**
- `actionagent` 1.8.1 — монтируемый engine на `/activeagents`: трейсы, токены, метрики по агентам, оценки стоимости, evaluations ([README](https://github.com/activeagents/activeagent/blob/main/README.md), [rg](https://rubygems.org/api/v1/gems/actionagent.json)).
- Хостинг-платформа activeagents.ai с биллингом и квотами ([README](https://github.com/activeagents/activeagent/blob/main/README.md)).
- Зависимость `activeagents-telemetry` 0.3.2 появилась 2026-08-10 ([rg](https://rubygems.org/api/v1/gems/activeagents-telemetry.json)).

**Провайдеры для РФ.**
- OpenAI ([open_ai](https://github.com/activeagents/activeagent/blob/main/docs/providers/open_ai.md)):
  - `host` — «Custom API endpoint URL (for Azure OpenAI)»;
  - по умолчанию используется Responses API;
  - `api_version: :chat` переключает на Chat Completions.
- Другие: Ollama `host`, OpenRouter `uri_base`, DeepSeek через OpenAI-совместимый API ([ollama](https://github.com/activeagents/activeagent/blob/main/docs/providers/ollama.md), [open_router](https://github.com/activeagents/activeagent/blob/main/docs/providers/open_router.md), [deepseek](https://github.com/activeagents/activeagent/blob/main/docs/providers/deepseek.md)).
- Провайдер RubyLLM: «Set `protocol:` to keep an agent on Chat Completions» для серверов, которые умеют только Chat Completions ([ruby_llm](https://github.com/activeagents/activeagent/blob/main/docs/providers/ruby_llm.md)).

**Зрелость и активность** ([rg](https://rubygems.org/api/v1/versions/activeagent.json), [commits](https://github.com/activeagents/activeagent/commits/main)):
- 1.0.0 вышел 2025-11-21; 1.8.1 — 2026-10-01; 17 релизов за 90 дней;
- последние коммиты помечены авторами TonsOfFun, claude, codex;
- коммит «Fold #578 into 1.9.0», то есть 1.9.0 готовится.

**Мейнтейнер.** Justin Bowen ([rg](https://rubygems.org/api/v1/gems/activeagent.json)).

#### Inferences

- **Сильные стороны.** Привычная Rails-ментальная модель: агент-роль = класс, этап = action, промпт = view. Бюджеты на делегирование сформулированы явно. Через провайдер `ruby_llm` ActiveAgent может работать поверх RubyLLM.
- **Слабые стороны.**
  - Бюджеты живут в рамках одной генерации, недельный бюджет агента придётся писать самому.
  - Хранение в 0.x-геме.
  - Очень быстрые релизы: 1.x меняется еженедельно.
  - Зависимость от коммерческой платформы как направления развития — это вывод, а не факт.
- **Вывод.** Если ядро — RubyLLM 2.1, ActiveAgent дублирует слой. Брать идеи: ERB-промпты, delegation budget, схему `solid_agent` для памяти и прогонов.

#### Gaps

- Точное поведение `host` у OpenAI-провайдера с не-Azure эндпоинтом (Yandex) — не проверено: документация описывает его «for Azure OpenAI».
- Есть ли в ActiveAgent MCP-сервер — не проверено. Найдено только клиентское подключение через `mcps:`.

---

## 3. Применимость в РФ: провайдеры и как их подключать

### Takeaway

- **Нативных Ruby-гемов для YandexGPT нет.** Для GigaChat есть два: свежий `gigachat-ruby` и брошенный `gigachat`.
- **YandexGPT.** Yandex AI Studio даёт OpenAI-совместимый Chat Completions на `https://llm.api.cloud.yandex.net/v1/`. RubyLLM, ActiveAgent, `openai`-gem и DSPy.rb могут к нему ходить. Для RubyLLM надежнее короткий собственный провайдер: заголовок `Api-Key`, URI модели `gpt://<folder>/…`, принудительный `chat_completions`.
- **GigaChat — не drop-in OpenAI.** Нужен OAuth-токен на 30 минут и корневой сертификат Минцифры. Решения: либо sidecar-прокси `gpt2giga`, либо свой провайдер RubyLLM поверх `gigachat-ruby`.
- **Anthropic РФ не поддерживает.**

### Cited Findings

**YandexGPT / Yandex AI Studio** (официальный Python SDK Yandex на GitHub):
- Эндпоинт: в `CLOUD_ENDPOINTS` сервис `http_completions` указывает на `https://llm.api.cloud.yandex.net/v1/` ([_utils/http.py](https://github.com/yandex-cloud/yandex-cloud-ml-sdk/blob/master/src/yandex_ai_studio_sdk/_utils/http.py)).
- Запрос и модель: чат шлёт POST на `/chat/completions` с полем `'model': self._uri` ([chat/completions/model.py](https://github.com/yandex-cloud/yandex-cloud-ml-sdk/blob/master/src/yandex_ai_studio_sdk/_chat/completions/model.py)). URI модели имеет вид `gpt://<folder_id>/<model>/<version>` ([models/completions/function.py](https://github.com/yandex-cloud/yandex-cloud-ml-sdk/blob/master/src/yandex_ai_studio_sdk/_models/completions/function.py)).
- Авторизация: для API-ключа SDK шлёт `authorization: Api-Key <key>`, для IAM-токена — `Bearer <token>` ([_auth.py](https://github.com/yandex-cloud/yandex-cloud-ml-sdk/blob/master/src/yandex_ai_studio_sdk/_auth.py)).
- Возможности API по README SDK ([README](https://github.com/yandex-cloud/yandex-cloud-ml-sdk/blob/master/README.md)):
  - «OpenAI‑compatible chat API (`sdk.chat`)» — stream, tool calls и OpenAI-совместимые embeddings;
  - Assistants/Threads/Runs удалены из SDK со ссылкой на «migration guide to Responses API»;
  - Search API включает **Wordstat** (динамика запросов, топ-запросы, распределение по регионам и устройствам).
- Репозиторий SDK: 177★, 33 форка ([GitHub](https://github.com/yandex-cloud/yandex-cloud-ml-sdk)); лицензия Apache 2.0, «Copyright 2024 YANDEX LLC» ([LICENSE](https://github.com/yandex-cloud/yandex-cloud-ml-sdk/blob/master/LICENSE)).

**GigaChat (Сбер):**
- README `gpt2giga`: «GigaChat не является drop-in заменой OpenAI или Anthropic API. Прямое подключение существующих SDK часто ломается на формате запросов, streaming-событиях, tool schemas, model discovery, авторизации и optional-параметрах клиентов» ([gpt2giga README](https://github.com/ai-forever/gpt2giga/blob/main/README.md)).
- `gpt2giga` — это FastAPI-прокси ([gpt2giga README](https://github.com/ai-forever/gpt2giga/blob/main/README.md)):
  - эндпоинты: OpenAI-совместимые `/chat/completions`, `/responses`, `/embeddings`, `/models`; Anthropic `/messages`; Gemini;
  - маппинг tools/function calling, structured output и SSE «там, где GigaChat поддерживает базовую возможность»;
  - Files и Batches отключены;
  - локальный адрес — `http://localhost:8090`, клиент OpenAI SDK подключается с `base_url="http://localhost:8090/v1"`.
- Метрики репо: MIT, 137★, 49 форков ([GitHub](https://github.com/ai-forever/gpt2giga)).
- Официальный Python SDK GigaChat ([gigachat README](https://github.com/ai-forever/gigachat/blob/master/README.md)):
  - `base_url` по умолчанию `https://api.giga.chat/v1`;
  - `auth_url` — `https://ngw.devices.sberbank.ru:9443/api/v2/oauth`;
  - scopes: `GIGACHAT_API_PERS` (физлица), `GIGACHAT_API_B2B` (бизнес, предоплата), `GIGACHAT_API_CORP` (бизнес, постоплата);
  - «Access tokens expire after 30 minutes»;
  - TLS-сертификат ставится по инструкции Госуслуг, опции `verify_ssl_certs` и `ca_bundle_file`.
- `gigachat-ruby` ([README](https://github.com/amdest/gigachat-ruby/blob/main/README.md)):
  - чат v1/v2 со стримингом, embeddings, files, batches, подсчёт токенов, проверка функций;
  - «Automatic OAuth token management»: 30-минутный токен кешируется и обновляется заранее;
  - вшит «Russian Trusted Root CA (Ministry of Digital Development)», действует до 2032-02-27, добавляется в per-client store;
  - стриминг идёт по HTTP/2 через httpx: «over HTTP/1.1 the whole answer arrives at once»;
  - прокси из `HTTP(S)_PROXY` к стримам не применяются;
  - версии 0.1.0–0.1.2 вышли 2026-10-02…04, автор Aleksandr Dryzhuk ([rg](https://rubygems.org/api/v1/gems/gigachat-ruby.json)).
- Гем `gigachat` (Denis Smolev): единственная версия 0.1.0 от 2024-09-12, 705 загрузок ([rg](https://rubygems.org/api/v1/versions/gigachat.json)).

**Нативные гемы YandexGPT** не найдены. Пустые выдачи rubygems по запросам `yandexgpt`, `yandex_gpt`, `yandex-gpt`, `ruby_llm yandex`, `llm yandex`, `yandex foundation` ([пример](https://rubygems.org/api/v1/search.json?query=yandexgpt)). По запросу `yandex` LLM-гемов тоже нет ([поиск](https://rubygems.org/api/v1/search.json?query=yandex)).

**OpenAI-совместимые эндпоинты в RubyLLM.**
- `config.openai_api_base = "http://localhost:8080/v1"  # vLLM, LiteLLM, etc.` плюс `RubyLLM.chat(model:, provider: :openai, assume_model_exists: true)` ([configuration-providers](https://github.com/crmne/ruby_llm/blob/main/docs/_getting_started/configuration-providers.md)).
- `config.openai_use_system_role = true` — для серверов, которые требуют роль `system` вместо `developer`, например старых vLLM ([configuration-providers](https://github.com/crmne/ruby_llm/blob/main/docs/_getting_started/configuration-providers.md)).
- `config.openai_protocol = Symbol` — «Every provider exposes <provider>_protocol» ([configuration](https://github.com/crmne/ruby_llm/blob/main/docs/_getting_started/configuration.md)).
- Провайдер OpenAI по умолчанию берёт протокол `:responses`; доступен и `:chat_completions` ([openai.rb](https://github.com/crmne/ruby_llm/blob/main/lib/ruby_llm/providers/openai.rb)).
- Заголовки OpenAI-провайдера: `Authorization: Bearer <openai_api_key>`, `OpenAI-Organization`, `OpenAI-Project` (из `openai_project_id`) ([openai.rb](https://github.com/crmne/ruby_llm/blob/main/lib/ruby_llm/providers/openai.rb)).

**Нативные провайдеры RubyLLM, полезные в РФ:**
- DeepSeek: `deepseek_api_base`, по умолчанию `https://api.deepseek.com` ([deepseek.rb](https://github.com/crmne/ruby_llm/blob/main/lib/ruby_llm/providers/deepseek.rb));
- OpenRouter: по умолчанию `https://openrouter.ai/api/v1` ([openrouter.rb](https://github.com/crmne/ruby_llm/blob/main/lib/ruby_llm/providers/openrouter.rb));
- Ollama: `ollama_api_base` ([ollama.rb](https://github.com/crmne/ruby_llm/blob/main/lib/ruby_llm/providers/ollama.rb));
- GPUStack (self-hosted): `gpustack_api_base` ([gpustack.rb](https://github.com/crmne/ruby_llm/blob/main/lib/ruby_llm/providers/gpustack.rb)).

**Сторонние провайдер-плагины RubyLLM.** Например, `ruby_llm-providers-lms` (LM Studio) и `ruby_llm-providers-infomaniak` ([поиск](https://rubygems.org/api/v1/search.json?query=ruby_llm)).

**Anthropic.** Россия отсутствует в списке, «Any country or region not listed is unsupported» ([anthropic.com/supported-countries](https://www.anthropic.com/supported-countries)).

### Inferences

**Рецепт YandexGPT в RubyLLM (гипотеза, вживую не проверено).** Вариант 1 — свой провайдер `Yandex`: подкласс со встроенным протоколом `Protocols::ChatCompletions`. Он задаёт:
- `api_base "https://llm.api.cloud.yandex.net/v1"`;
- `headers { "Authorization" => "Api-Key #{key}" }`;
- маппинг имени модели в `gpt://<folder>/<model>/<version>`.

Это укладывается в «a few dozen lines» по [custom-providers](https://github.com/crmne/ruby_llm/blob/main/docs/_reference/custom-providers.md). Вариант 2 (быстрее): провайдер `:openai` с `openai_api_base`, `openai_protocol = :chat_completions`, `assume_model_exists: true` и IAM-токеном в `openai_api_key`. Заголовок `Bearer` для IAM подтверждён в SDK Yandex.

**Учёт стоимости для YandexGPT/GigaChat.** Этих моделей нет в реестре цен RubyLLM, поэтому `cost.total` будет `nil` ([cost](https://github.com/crmne/ruby_llm/blob/main/docs/_core_features/cost-and-usage-tracking.md)). Токены пишутся, если эндпоинт возвращает usage. Значит, рублёвую стоимость и бюджеты считать своей таблицей тарифов поверх `ruby_llm_usages`.

**GigaChat.** Минимум кода — sidecar `gpt2giga` (Python) и RubyLLM `openai_api_base = http://gpt2giga:8090/v1` с `chat_completions`. Более «чистый Ruby» — свой провайдер RubyLLM, где авторизацию и CA берёт `gigachat-ruby` или код из него. Риск: гему одна неделя.

**Qwen, DeepSeek и другие open-weight модели — self-hosted.** vLLM или Ollama подключаются через `openai_api_base` или нативный `ollama`. Это работает во всех трёх фреймворках из топ-3, MCP SDK от провайдера не зависит.

**Wordstat.** Для этапа обогащения Wordstat доступен через Search API в Yandex AI Studio (по README Python SDK). Ruby-клиента нет, REST-клиент придётся писать самому.

### Gaps

- **Авторизация Yandex.** Принимает ли OpenAI-совместимый эндпоинт `Authorization: Bearer <API-ключ>` (так шлют OpenAI SDK и RubyLLM) и нужен ли заголовок `OpenAI-Project`/`x-folder-id` — **не проверено**. Документация aistudio.yandex.ru недоступна из среды, WebSearch исчерпан.
- **Responses API у Yandex.** Поддерживает ли AI Studio Responses API на том же хосте, то есть можно ли оставить дефолтный протокол RubyLLM 2.x, — не проверено. Есть только ссылка на «migration guide to Responses API» в README SDK.
- Срок жизни IAM-токена Yandex — не проверено.
- Каталог моделей Yandex AI Studio (Qwen, DeepSeek, gpt-oss и т.п.) и Cloud.ru Foundation Models — не проверено.
- Доступность из РФ и способы оплаты DeepSeek API и OpenRouter — не проверено.
- Поддержка РФ в OpenAI — не проверено: platform.openai.com не открылся.

---

## 4. Rails-планировщики (для недельного цикла; не входят в число кандидатов)

- **Solid Queue — recurring tasks.**
  - Задачи описываются в `config/recurring.yml` (например, `schedule: every …`) и исполняются отдельным процессом-шедулером.
  - Есть динамическое планирование `SolidQueue.schedule_recurring_task` с `dynamic_tasks_enabled`.
  - Установщик настраивает Solid Queue как production-бэкенд Active Job.
  - Версия 1.7.0 от 2026-08-21, MIT.
  - Источники: [README#recurring-tasks](https://github.com/rails/solid_queue#recurring-tasks), [rg](https://rubygems.org/api/v1/gems/solid_queue.json).
- **GoodJob — cron.**
  - Включается `config.good_job.enable_cron = true` и задаётся `config.good_job.cron = {…}`.
  - «Enabling cron on multiple processes will not enqueue duplicate jobs» — дубли отсекают unique indexes.
  - Только Postgres; есть batches, concurrency controls и веб-дашборд.
  - Версия 4.21.1 от 2026-10-07, MIT.
  - Источники: [README#cron](https://github.com/bensheldon/good_job#cron-style-repeatingrecurring-jobs), [rg](https://rubygems.org/api/v1/gems/good_job.json).
- **(+) ActiveJob Continuations (Rails 8.1+).** Долгая job с `checkpoint!` переживает деплой. На этом RubyLLM строит durable-агентов ([RubyLLM durable-agents](https://github.com/crmne/ruby_llm/blob/main/docs/_advanced/durable-agents.md), [API](https://api.rubyonrails.org/classes/ActiveJob/Continuable.html)).

---

## 5. Go-эквиваленты (кратко; не входят в 6)

### Takeaway

У Go сильные **официальные SDK**: MCP v1.8, Anthropic v1.79, OpenAI v3.74. Есть два зрелых агентных фреймворка — Genkit Go 1.x (production-ready по README) и Eino (ещё 0.x). Но в Go нет аналога связки RubyLLM + Rails: ORM-персистентности, durable-агентов на очереди задач, approvals и ledger затрат «из коробки».

### Cited Findings

| Модуль | Зрелость | Активность | Лицензия |
|---|---|---|---|
| [modelcontextprotocol/go-sdk](https://github.com/modelcontextprotocol/go-sdk) | Официальный SDK; v1.0.0 от 2025-09-30 ([proxy](https://proxy.golang.org/github.com/modelcontextprotocol/go-sdk/@v/v1.0.0.info)); v1.7.0+ поддерживает спецификацию 2026-07-28, client-side OAuth для 2025-11-25 экспериментальный ([README](https://github.com/modelcontextprotocol/go-sdk/blob/main/README.md)) | v1.8.0 от 2026-09-04, 33 версии ([proxy](https://proxy.golang.org/github.com/modelcontextprotocol/go-sdk/@latest)) | Apache-2.0 для нового кода, MIT для существующего; на pkg.go.dev: Apache-2.0, CC-BY-4.0, MIT ([pkg.go.dev](https://pkg.go.dev/github.com/modelcontextprotocol/go-sdk?tab=licenses)) |
| [mark3labs/mcp-go](https://github.com/mark3labs/mcp-go) | Комьюнити; v1.0.0 только от 2026-09-02 ([proxy](https://proxy.golang.org/github.com/mark3labs/mcp-go/@v/v1.0.0.info)) | v1.2.1 от 2026-10-09, 124 версии ([proxy](https://proxy.golang.org/github.com/mark3labs/mcp-go/@latest)) | MIT ([pkg.go.dev](https://pkg.go.dev/github.com/mark3labs/mcp-go?tab=licenses)) |
| [anthropics/anthropic-sdk-go](https://github.com/anthropics/anthropic-sdk-go) | v1.0.0 от 2025-05-22 ([proxy](https://proxy.golang.org/github.com/anthropics/anthropic-sdk-go/@v/v1.0.0.info)); imported by 372 ([pkg.go.dev](https://pkg.go.dev/github.com/anthropics/anthropic-sdk-go)) | v1.79.1 от 2026-10-08 ([proxy](https://proxy.golang.org/github.com/anthropics/anthropic-sdk-go/@latest)) | MIT ([pkg.go.dev](https://pkg.go.dev/github.com/anthropics/anthropic-sdk-go?tab=licenses)) |
| [openai/openai-go](https://github.com/openai/openai-go) | v1.0.0 от 2025-05-19; мажор v3 с 2025-09-30 ([proxy](https://proxy.golang.org/github.com/openai/openai-go/v3/@v/v3.0.0.info)); /v3 imported by 697 ([pkg.go.dev](https://pkg.go.dev/github.com/openai/openai-go/v3)) | v3.74.0 от 2026-10-08 ([proxy](https://proxy.golang.org/github.com/openai/openai-go/v3/@latest)) | Apache-2.0 ([pkg.go.dev](https://pkg.go.dev/github.com/openai/openai-go/v3?tab=licenses)) |
| [cloudwego/eino](https://github.com/cloudwego/eino) | Нет 1.0: стабильная v0.9.21, параллельно v0.10.0-alpha.35; первая версия в proxy — v0.3.0 от 2024-12-11 ([proxy list](https://proxy.golang.org/github.com/cloudwego/eino/@v/list)) | v0.9.21 от 2026-09-23, 250 версий ([proxy](https://proxy.golang.org/github.com/cloudwego/eino/@latest)) | Apache-2.0 ([pkg.go.dev](https://pkg.go.dev/github.com/cloudwego/eino?tab=licenses)) |
| [tmc/langchaingo](https://github.com/tmc/langchaingo) | 0.1.x; README зовёт в мейнтейнеры: «momentum for moving the development… to a more community effort» ([README](https://github.com/tmc/langchaingo/blob/main/README.md)) | v0.1.13 (2025-02-09) → v0.1.14 (2025-10-20) → v0.1.15 (2026-10-05) ([proxy](https://proxy.golang.org/github.com/tmc/langchaingo/@latest)) — редкие релизы | MIT ([pkg.go.dev](https://pkg.go.dev/github.com/tmc/langchaingo?tab=licenses)) |
| [firebase/genkit (Go)](https://github.com/firebase/genkit) | v1.0.0 от 2025-09-09 ([proxy](https://proxy.golang.org/github.com/firebase/genkit/go/@v/v1.0.0.info)); «Go: Production-ready with full feature support» ([README](https://github.com/firebase/genkit/blob/main/README.md)) | v1.13.1 от 2026-09-03, 42 версии ([proxy](https://proxy.golang.org/github.com/firebase/genkit/go/@latest)) | Apache-2.0 ([pkg.go.dev](https://pkg.go.dev/github.com/firebase/genkit/go?tab=licenses)) |

### Inferences

- **MCP.** Оба стека имеют официальный MCP SDK 1.x. Go дошёл до 1.0 примерно на 10 месяцев раньше (2025-09-30 против 2026-07-24). Для MCP-сервера стек не критичен.
- **Ruby/Rails выигрывает объёмом «не писать самому».** В Ruby durable-агенты, approvals, хранение транскриптов, ledger затрат, генераторы UI и Agent Skill для Claude Code приходят одним гемом поверх ActiveRecord и ActiveJob. На Go это собирается из SDK и фреймворка (Genkit или Eino) плюс собственные БД-схема, очередь задач и админка.
- **Go выигрывает** официальностью и стабильностью клиентских SDK (OpenAI v3 против Ruby 0.x) и, вероятно, операционной простотой: один бинарник. Это вывод, в этом слое не проверялся.
- **Оценка для соло-фаундера с Claude Code.** Главные риски Ruby — один мейнтейнер у RubyLLM и свежий мажор 2.0. Главный риск Go — больше собственного кода на оркестрацию, хранение и UI.

### Gaps

- Звёзды и форки Go-репозиториев не собирались: в задаче их не требовали.
- Есть ли в Genkit или Eino durable-исполнение и approvals — не проверено.
- ByteDance как владелец CloudWeGo не подтверждён в README — не проверено.
- Счётчик «Imported by» для go-sdk, mcp-go, eino, langchaingo и genkit — не извлечён или ненадёжен (у модульных корней 0).

---

## 6. Вывод по слою: что взять готовым / что как референс / что писать самому

### Takeaway

Готовым брать RubyLLM 2.1 (ядро агентов и учёта), официальный `mcp` (MCP-сервер) и Solid Queue (расписание и durable-джобы). Как референс использовать ActiveAgent, ruby_llm-agents, DSPy.rb, Roast, Raix и gigachat-ruby/gpt2giga. Самому писать провайдеры YandexGPT и GigaChat, рублёвый прайсинг и принудительные бюджеты, UI одобрений, оркестрацию недельного цикла и REST-клиенты к Wordstat, Direct, ГИР БО и ЮKassa.

### Cited Findings

**Брать готовым:**
- **RubyLLM `~> 2.1`** ([rg](https://rubygems.org/api/v1/versions/ruby_llm.json), [README](https://github.com/crmne/ruby_llm/blob/main/README.md)):
  - агенты (`RubyLLM::Agent`) и tools с `requires_approval`;
  - `with_schema`;
  - durable-агенты на ActiveJob и Continuations ([durable-agents](https://github.com/crmne/ruby_llm/blob/main/docs/_advanced/durable-agents.md));
  - MCP-клиент ([mcp](https://github.com/crmne/ruby_llm/blob/main/docs/_core_features/mcp.md));
  - ledger `ruby_llm_usages` с `owner` ([rails-persistence](https://github.com/crmne/ruby_llm/blob/main/docs/_advanced/rails-persistence.md));
  - события и OpenTelemetry ([instrumentation](https://github.com/crmne/ruby_llm/blob/main/docs/_advanced/instrumentation.md));
  - Agent Skill для Claude Code ([ai-coding-assistants](https://github.com/crmne/ruby_llm/blob/main/docs/_getting_started/ai-coding-assistants.md)).
- **`mcp` `~> 1.7`** — MCP-сервер внутри Rails на Streamable HTTP ([docs](https://ruby.sdk.modelcontextprotocol.io/server/transports/), [rg](https://rubygems.org/api/v1/versions/mcp.json)).
- **Solid Queue** (recurring) или **GoodJob** (cron) — запуск недельного цикла ([solid_queue](https://github.com/rails/solid_queue#recurring-tasks), [good_job](https://github.com/bensheldon/good_job#cron-style-repeatingrecurring-jobs)).

**Как референс:**

| Что | Что взять |
|---|---|
| ActiveAgent | Агенты-контроллеры, ERB-промпты, delegation budgets ([delegation](https://github.com/activeagents/activeagent/blob/main/docs/actions/delegation.md)), схема `solid_agent` для памяти и прогонов ([solid_agent](https://github.com/activeagents/activeagent/blob/main/docs/solid_agent.md)) |
| ruby_llm-agents | Бюджеты daily/monthly hard/soft, circuit breakers, алерты, cost analytics по агентам ([README](https://github.com/adham90/ruby_llm-agents/blob/main/README.md)). Учесть, что там недавно чинили недоучёт токенов в tool loops ([CHANGELOG](https://github.com/adham90/ruby_llm-agents/blob/main/CHANGELOG.md)) |
| DSPy.rb | Типизированные сигнатуры и оптимизаторы для скоринга, фильтра и турнира ([README](https://github.com/vicentereig/dspy.rb/blob/main/README.md)). Если брать, то через `dspy-openai`/Ollama-адаптер, а не `dspy-ruby_llm` (<2.0) ([rg](https://rubygems.org/api/v1/gems/dspy-ruby_llm.json)) |
| Roast (Shopify) | Workflow DSL для dev-автоматизации с Claude Code CLI ([README](https://github.com/Shopify/roast/blob/main/README.md)) |
| Raix | Паттерны «дискретных AI-компонентов» ([README](https://github.com/OlympiaAI/raix/blob/main/README.md)) |
| gigachat-ruby, gpt2giga | Ввод GigaChat ([gigachat-ruby](https://github.com/amdest/gigachat-ruby/blob/main/README.md), [gpt2giga](https://github.com/ai-forever/gpt2giga/blob/main/README.md)) |

**Не брать:**

| Что | Почему |
|---|---|
| langchainrb | Релизов нет 17 мес. ([rg](https://rubygems.org/api/v1/versions/langchainrb.json)) |
| fast-mcp | Заглох с 2025-09-28 ([commits](https://github.com/yjacquin/fast-mcp/commits/main)) |
| ruby_llm-mcp | Пинит `ruby_llm ~> 1.9` ([rg](https://rubygems.org/api/v1/gems/ruby_llm-mcp.json)) |
| ruby-openai | Релизов нет год ([rg](https://rubygems.org/api/v1/versions/ruby-openai.json)) |
| anthropic | РФ не поддерживается ([anthropic.com](https://www.anthropic.com/supported-countries)) |
| gigachat (2024) | Одна версия 0.1.0 от 2024-09-12 ([rg](https://rubygems.org/api/v1/versions/gigachat.json)) |

### Inferences

**Писать самому** (ориентировочный объём — оценка, не факт):
1. **Провайдер YandexGPT для RubyLLM** — десятки строк: `Api-Key`, `gpt://folder/...`, `chat_completions` ([custom-providers](https://github.com/crmne/ruby_llm/blob/main/docs/_reference/custom-providers.md)). Плюс провайдер GigaChat (OAuth 30 мин, CA) или sidecar `gpt2giga`.
2. **Рублёвый прайсинг и бюджет-guard на роль и неделю** поверх `ruby_llm_usages` и `usage.ruby_llm`. RubyLLM учитывает, но не ограничивает; для моделей вне реестра стоимость равна `nil`.
3. **UI одобрений.** Очередь `pending_approvals` с решениями «бюджет / домен / legal / GO» и авторизация решений средствами приложения ([durable-agents](https://github.com/crmne/ruby_llm/blob/main/docs/_advanced/durable-agents.md)).
4. **Оркестрация недельного цикла.** Recurring-job, затем fan-out 50 идей по job'ам, фильтр, обогащение, юнит-экономика, турнир и смоук-тест. Опора — паттерны RubyLLM, идемпотентные инструменты.
5. **REST-клиенты** Wordstat/Direct/ГИР БО/ЮKassa как сервис-объекты с двумя адаптерами: `RubyLLM::Tool` для агентов и `MCP::Tool` для Claude Code. Ruby-гемы для них в этом слое не проверялись.
6. **Свой «турнир».** Попарные сравнения через `with_schema` на доступных в РФ моделях, а не через `RubyLLM::Judge`, который по умолчанию завязан на TypeSafe/OpenAI Decisions.

**Аргумент для выбора стека.** В Ruby большая часть агентного рантайма готова. Остаётся интеграционный код под российских провайдеров и бизнес-API — его придётся писать в любом стеке.

### Gaps

- Ничего из рецептов не проверено запуском: анализ только read-only, без установки и токенов.
- Нужна ранняя проверка вживую: (1) авторизация и протокол Yandex AI Studio через RubyLLM; (2) возвращает ли Yandex usage в ответах Chat Completions, без этого ledger не заполнится токенами.

---

## 7. Источники

**RubyGems (API):**
- [ruby_llm gem](https://rubygems.org/api/v1/gems/ruby_llm.json), [versions](https://rubygems.org/api/v1/versions/ruby_llm.json)
- [ruby_llm-mcp gem](https://rubygems.org/api/v1/gems/ruby_llm-mcp.json), [versions](https://rubygems.org/api/v1/versions/ruby_llm-mcp.json)
- [activeagent gem](https://rubygems.org/api/v1/gems/activeagent.json), [versions](https://rubygems.org/api/v1/versions/activeagent.json)
- [solid_agent](https://rubygems.org/api/v1/gems/solid_agent.json), [actionagent](https://rubygems.org/api/v1/gems/actionagent.json), [activeagents-telemetry](https://rubygems.org/api/v1/gems/activeagents-telemetry.json)
- [langchainrb versions](https://rubygems.org/api/v1/versions/langchainrb.json), [langchainrb_rails](https://rubygems.org/api/v1/gems/langchainrb_rails.json)
- [mcp gem](https://rubygems.org/api/v1/gems/mcp.json), [versions](https://rubygems.org/api/v1/versions/mcp.json)
- [fast-mcp gem](https://rubygems.org/api/v1/gems/fast-mcp.json)
- [raix gem](https://rubygems.org/api/v1/gems/raix.json), [versions](https://rubygems.org/api/v1/versions/raix.json)
- [roast-ai gem](https://rubygems.org/api/v1/gems/roast-ai.json), [versions](https://rubygems.org/api/v1/versions/roast-ai.json)
- [anthropic versions](https://rubygems.org/api/v1/versions/anthropic.json), [openai gem](https://rubygems.org/api/v1/gems/openai.json), [openai versions](https://rubygems.org/api/v1/versions/openai.json)
- [dspy gem](https://rubygems.org/api/v1/gems/dspy.json), [versions](https://rubygems.org/api/v1/versions/dspy.json), [dspy-ruby_llm](https://rubygems.org/api/v1/gems/dspy-ruby_llm.json), [dspy-openai](https://rubygems.org/api/v1/gems/dspy-openai.json), [dspy-anthropic](https://rubygems.org/api/v1/gems/dspy-anthropic.json)
- [ruby-openai versions](https://rubygems.org/api/v1/versions/ruby-openai.json)
- [ruby_llm-agents gem](https://rubygems.org/api/v1/gems/ruby_llm-agents.json), [versions](https://rubygems.org/api/v1/versions/ruby_llm-agents.json)
- [ruby_llm-monitoring](https://rubygems.org/api/v1/gems/ruby_llm-monitoring.json), [opentelemetry-instrumentation-ruby_llm](https://rubygems.org/api/v1/gems/opentelemetry-instrumentation-ruby_llm.json), [swarm_sdk](https://rubygems.org/api/v1/gems/swarm_sdk.json)
- [gigachat-ruby gem](https://rubygems.org/api/v1/gems/gigachat-ruby.json), [versions](https://rubygems.org/api/v1/versions/gigachat-ruby.json), [gigachat versions](https://rubygems.org/api/v1/versions/gigachat.json)
- [solid_queue](https://rubygems.org/api/v1/gems/solid_queue.json), [good_job](https://rubygems.org/api/v1/gems/good_job.json)
- Поиск: [ruby_llm](https://rubygems.org/api/v1/search.json?query=ruby_llm), [yandexgpt](https://rubygems.org/api/v1/search.json?query=yandexgpt), [yandex](https://rubygems.org/api/v1/search.json?query=yandex), [gigachat](https://rubygems.org/api/v1/search.json?query=gigachat)

**RubyLLM:**
- [repo](https://github.com/crmne/ruby_llm), [commits](https://github.com/crmne/ruby_llm/commits/main), [releases](https://github.com/crmne/ruby_llm/releases), [README](https://github.com/crmne/ruby_llm/blob/main/README.md)
- Исходники: [configuration.rb](https://github.com/crmne/ruby_llm/blob/main/lib/ruby_llm/configuration.rb), [providers/openai.rb](https://github.com/crmne/ruby_llm/blob/main/lib/ruby_llm/providers/openai.rb), [deepseek.rb](https://github.com/crmne/ruby_llm/blob/main/lib/ruby_llm/providers/deepseek.rb), [openrouter.rb](https://github.com/crmne/ruby_llm/blob/main/lib/ruby_llm/providers/openrouter.rb), [ollama.rb](https://github.com/crmne/ruby_llm/blob/main/lib/ruby_llm/providers/ollama.rb), [gpustack.rb](https://github.com/crmne/ruby_llm/blob/main/lib/ruby_llm/providers/gpustack.rb)
- Документация, настройка: [configuration-providers](https://github.com/crmne/ruby_llm/blob/main/docs/_getting_started/configuration-providers.md), [configuration](https://github.com/crmne/ruby_llm/blob/main/docs/_getting_started/configuration.md), [custom-providers](https://github.com/crmne/ruby_llm/blob/main/docs/_reference/custom-providers.md)
- Документация, агенты и Rails: [agents](https://github.com/crmne/ruby_llm/blob/main/docs/_advanced/agents.md), [agentic-workflows](https://github.com/crmne/ruby_llm/blob/main/docs/_advanced/agentic-workflows.md), [durable-agents](https://github.com/crmne/ruby_llm/blob/main/docs/_advanced/durable-agents.md), [rails](https://github.com/crmne/ruby_llm/blob/main/docs/_advanced/rails.md), [rails-persistence](https://github.com/crmne/ruby_llm/blob/main/docs/_advanced/rails-persistence.md)
- Документация, MCP, учёт, наблюдаемость: [mcp](https://github.com/crmne/ruby_llm/blob/main/docs/_core_features/mcp.md), [cost-and-usage-tracking](https://github.com/crmne/ruby_llm/blob/main/docs/_core_features/cost-and-usage-tracking.md), [instrumentation](https://github.com/crmne/ruby_llm/blob/main/docs/_advanced/instrumentation.md)
- Документация, версии: [upgrading](https://github.com/crmne/ruby_llm/blob/main/docs/_reference/upgrading.md), [whats-new-2.0](https://github.com/crmne/ruby_llm/blob/main/docs/_getting_started/whats-new-in-2-0.md), [whats-new-2.1](https://github.com/crmne/ruby_llm/blob/main/docs/_getting_started/whats-new-in-2-1.md), [ai-coding-assistants](https://github.com/crmne/ruby_llm/blob/main/docs/_getting_started/ai-coding-assistants.md)

**MCP Ruby SDK:**
- [repo/README](https://github.com/modelcontextprotocol/ruby-sdk), [commits](https://github.com/modelcontextprotocol/ruby-sdk/commits/main), [server transports](https://ruby.sdk.modelcontextprotocol.io/server/transports/)

**ActiveAgent:**
- [repo](https://github.com/activeagents/activeagent), [README](https://github.com/activeagents/activeagent/blob/main/README.md), [commits](https://github.com/activeagents/activeagent/commits/main)
- Документация: [framework](https://github.com/activeagents/activeagent/blob/main/docs/framework.md), [generation](https://github.com/activeagents/activeagent/blob/main/docs/agents/generation.md), [rails](https://github.com/activeagents/activeagent/blob/main/docs/framework/rails.md), [mcps](https://github.com/activeagents/activeagent/blob/main/docs/actions/mcps.md), [delegation](https://github.com/activeagents/activeagent/blob/main/docs/actions/delegation.md), [usage](https://github.com/activeagents/activeagent/blob/main/docs/actions/usage.md), [solid_agent](https://github.com/activeagents/activeagent/blob/main/docs/solid_agent.md)
- Провайдеры: [open_ai](https://github.com/activeagents/activeagent/blob/main/docs/providers/open_ai.md), [ollama](https://github.com/activeagents/activeagent/blob/main/docs/providers/ollama.md), [open_router](https://github.com/activeagents/activeagent/blob/main/docs/providers/open_router.md), [deepseek](https://github.com/activeagents/activeagent/blob/main/docs/providers/deepseek.md), [ruby_llm](https://github.com/activeagents/activeagent/blob/main/docs/providers/ruby_llm.md)

**Прочие Ruby:**
- ruby_llm-mcp: [repo](https://github.com/patvice/ruby_llm-mcp), [commits](https://github.com/patvice/ruby_llm-mcp/commits/main)
- langchainrb: [repo](https://github.com/patterns-ai-core/langchainrb), [README](https://github.com/patterns-ai-core/langchainrb/blob/main/README.md), [CHANGELOG](https://github.com/patterns-ai-core/langchainrb/blob/main/CHANGELOG.md), [commits](https://github.com/patterns-ai-core/langchainrb/commits/main)
- fast-mcp: [repo](https://github.com/yjacquin/fast-mcp), [README](https://github.com/yjacquin/fast_mcp/blob/main/README.md), [CHANGELOG](https://github.com/yjacquin/fast_mcp/blob/main/CHANGELOG.md), [commits](https://github.com/yjacquin/fast-mcp/commits/main)
- Raix: [repo](https://github.com/OlympiaAI/raix), [README](https://github.com/OlympiaAI/raix/blob/main/README.md)
- Roast: [repo](https://github.com/Shopify/roast), [README](https://github.com/Shopify/roast/blob/main/README.md)
- anthropic-sdk-ruby: [repo](https://github.com/anthropics/anthropic-sdk-ruby), [README](https://github.com/anthropics/anthropic-sdk-ruby/blob/main/README.md), [client.rb](https://github.com/anthropics/anthropic-sdk-ruby/blob/main/lib/anthropic/client.rb), [tools runner](https://github.com/anthropics/anthropic-sdk-ruby/blob/main/lib/anthropic/helpers/tools/runner.rb)
- openai-ruby: [repo](https://github.com/openai/openai-ruby), [client.rb](https://github.com/openai/openai-ruby/blob/main/lib/openai/client.rb)
- DSPy.rb: [repo](https://github.com/vicentereig/dspy.rb), [README](https://github.com/vicentereig/dspy.rb/blob/main/README.md), [openai_adapter.rb](https://github.com/vicentereig/dspy.rb/blob/main/lib/dspy/openai/lm/adapters/openai_adapter.rb), [ollama_adapter.rb](https://github.com/vicentereig/dspy.rb/blob/main/lib/dspy/openai/lm/adapters/ollama_adapter.rb)
- ruby_llm-agents: [repo](https://github.com/adham90/ruby_llm-agents), [README](https://github.com/adham90/ruby_llm-agents/blob/main/README.md), [CHANGELOG](https://github.com/adham90/ruby_llm-agents/blob/main/CHANGELOG.md)
- ruby-openai: [repo](https://github.com/alexrudall/ruby-openai)

**РФ-провайдеры:**
- Yandex AI Studio SDK: [repo](https://github.com/yandex-cloud/yandex-cloud-ml-sdk), [README](https://github.com/yandex-cloud/yandex-cloud-ml-sdk/blob/master/README.md), [LICENSE](https://github.com/yandex-cloud/yandex-cloud-ml-sdk/blob/master/LICENSE)
- Yandex AI Studio SDK, исходники: [_utils/http.py](https://github.com/yandex-cloud/yandex-cloud-ml-sdk/blob/master/src/yandex_ai_studio_sdk/_utils/http.py), [_auth.py](https://github.com/yandex-cloud/yandex-cloud-ml-sdk/blob/master/src/yandex_ai_studio_sdk/_auth.py), [chat/completions/model.py](https://github.com/yandex-cloud/yandex-cloud-ml-sdk/blob/master/src/yandex_ai_studio_sdk/_chat/completions/model.py), [models/completions/function.py](https://github.com/yandex-cloud/yandex-cloud-ml-sdk/blob/master/src/yandex_ai_studio_sdk/_models/completions/function.py)
- GigaChat: [gpt2giga repo](https://github.com/ai-forever/gpt2giga), [gpt2giga README](https://github.com/ai-forever/gpt2giga/blob/main/README.md), [gigachat (Python SDK) README](https://github.com/ai-forever/gigachat/blob/master/README.md), [gigachat-ruby README](https://github.com/amdest/gigachat-ruby/blob/main/README.md)
- Anthropic: [Anthropic supported countries](https://www.anthropic.com/supported-countries)

**Rails-планировщики:**
- [Solid Queue README (recurring tasks)](https://github.com/rails/solid_queue#recurring-tasks), [GoodJob README (cron)](https://github.com/bensheldon/good_job#cron-style-repeatingrecurring-jobs), [ActiveJob::Continuable](https://api.rubyonrails.org/classes/ActiveJob/Continuable.html)

**Go:**
- modelcontextprotocol/go-sdk: [README](https://github.com/modelcontextprotocol/go-sdk/blob/main/README.md), [proxy @latest](https://proxy.golang.org/github.com/modelcontextprotocol/go-sdk/@latest), [v1.0.0](https://proxy.golang.org/github.com/modelcontextprotocol/go-sdk/@v/v1.0.0.info), [licenses](https://pkg.go.dev/github.com/modelcontextprotocol/go-sdk?tab=licenses)
- mark3labs/mcp-go: [proxy @latest](https://proxy.golang.org/github.com/mark3labs/mcp-go/@latest), [v1.0.0](https://proxy.golang.org/github.com/mark3labs/mcp-go/@v/v1.0.0.info), [licenses](https://pkg.go.dev/github.com/mark3labs/mcp-go?tab=licenses)
- anthropic-sdk-go: [proxy @latest](https://proxy.golang.org/github.com/anthropics/anthropic-sdk-go/@latest), [v1.0.0](https://proxy.golang.org/github.com/anthropics/anthropic-sdk-go/@v/v1.0.0.info), [pkg.go.dev](https://pkg.go.dev/github.com/anthropics/anthropic-sdk-go)
- openai-go: [v3 @latest](https://proxy.golang.org/github.com/openai/openai-go/v3/@latest), [v3.0.0](https://proxy.golang.org/github.com/openai/openai-go/v3/@v/v3.0.0.info), [pkg.go.dev v3](https://pkg.go.dev/github.com/openai/openai-go/v3)
- eino: [proxy list](https://proxy.golang.org/github.com/cloudwego/eino/@v/list), [@latest](https://proxy.golang.org/github.com/cloudwego/eino/@latest), [licenses](https://pkg.go.dev/github.com/cloudwego/eino?tab=licenses)
- langchaingo: [README](https://github.com/tmc/langchaingo/blob/main/README.md), [@latest](https://proxy.golang.org/github.com/tmc/langchaingo/@latest), [licenses](https://pkg.go.dev/github.com/tmc/langchaingo?tab=licenses)
- genkit: [README](https://github.com/firebase/genkit/blob/main/README.md), [go @latest](https://proxy.golang.org/github.com/firebase/genkit/go/@latest), [go v1.0.0](https://proxy.golang.org/github.com/firebase/genkit/go/@v/v1.0.0.info), [licenses](https://pkg.go.dev/github.com/firebase/genkit/go?tab=licenses)
