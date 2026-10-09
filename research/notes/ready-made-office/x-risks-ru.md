# Сквозные риски «LLM-компании» из готовых компонентов и ограничения РФ

Дата сбора: 2026-10-09. Контекст: соло-основатель в РФ. LLM-агенты (директор, аналитик, маркетолог) работают недельным циклом: ~50 идей → стоп-факторы → Wordstat / прогноз Директа / ГИР БО → юнит-экономика → турнир → smoke-test (Директ + лендинг + предоплата через ЮKassa, лестница бюджетов). Человек утверждает бюджет, домены, юридические вопросы и финальный GO. Стек — Ruby/Rails или Go, разработка с Claude Code, компоненты подключаются через MCP/HTTP.

Пометки:
- «открыт» — страница открыта через WebFetch, факт сверен с текстом;
- «по сниппету поиска» — факт взят только из выдачи WebSearch, сама страница не открывалась;
- «не проверено» — первоисточник не подтверждён;
- «предложение» — собственная рекомендация исследователя.

Ограничения среды:
- WebFetch не резолвит arxiv.org, conf.researchr.org, modelcontextprotocol.io, yandex.cloud / yandex.ru / aistudio.yandex.ru, developers.sber.ru, docs.litellm.ai, consultant.ru / garant.ru, simonwillison.net, genai.owasp.org и ряд СМИ.
- curl через прокси не работает.
- Общий на все агенты лимит WebSearch исчерпан в середине работы. Поэтому часть вопросов перенесена в Gaps, а многие российские правовые факты помечены «по сниппету поиска».

## 1. LLM-провайдер и доступ из РФ

### Takeaway
Россия не входит в список стран, которые поддерживает Anthropic. Условия Anthropic позволяют приостановить доступ за нарушение региональной политики. В 2026 году СМИ сообщали о волнах блокировок российских аккаунтов Claude. Поэтому Claude Agent SDK или Claude Code в роли production-рантайма «директора» — единая точка отказа, которую основатель не контролирует.

В РФ доступны:
- YandexGPT (Yandex AI Studio): OpenAI-совместимые API и function calling;
- GigaChat: function calling, официальные SDK, пакеты токенов;
- open-weight модели (Qwen, DeepSeek и др.) через Yandex AI Studio и Cloud.ru.

Шлюз LiteLLM снижает lock-in, но сам стал вектором атаки: в марте 2026 его пакет в PyPI был скомпрометирован, а в 2026 году для него опубликованы десятки advisories.

### Cited Findings

**Anthropic / Claude**
- В списке поддерживаемых стран нет России, Беларуси, Китая и Ирана. Формулировка: «Any country or region not listed is unsupported». Отдельно упомянуто использование «while physically located in an unsupported region». — [Anthropic: Supported countries](https://www.anthropic.com/supported-countries) (открыт)
- В сентябре 2025 (на странице — 4 сентября 2025) Anthropic ужесточила ограничения. Теперь запрещены компании, которые более чем на 50% прямо или косвенно принадлежат компаниям со штаб-квартирой в неподдерживаемых регионах, независимо от места их работы. Отдельно упомянут доступ «through subsidiaries incorporated in other countries». Россия в посте поимённо не названа: вместо этого дана ссылка на список стран. — [Anthropic: Updating restrictions of sales to unsupported regions](https://www.anthropic.com/news/updating-restrictions-of-sales-to-unsupported-regions) (открыт)
- Consumer Terms (действуют с 8 октября 2025):
  - раздел 3 требует соблюдать Supported Regions Policy;
  - Anthropic может «suspend or terminate your access to the Services (including any Subscriptions) at any time… without notice to you if we believe that you have breached these Terms»;
  - при прекращении доступа за нарушение возврат за подписку не положен;
  - раздел 12 (экспортный контроль) запрещает доступ из стран под эмбарго США.

  — [Anthropic Consumer Terms](https://www.anthropic.com/legal/consumer-terms) (открыт)
- Commercial Terms (действуют с 17 июня 2025; под ними API и Agent SDK):
  - D.2 — соблюдение Supported Regions Policy;
  - I.3.a — право приостановить доступ при нарушении D.2 или если предоставление услуг запрещено законом;
  - M.8 — экспортный контроль и санкции США;
  - при приостановке Anthropic «use reasonable efforts to provide written notice».

  — [Anthropic Commercial Terms](https://www.anthropic.com/legal/commercial-terms) (открыт)
- Agent SDK:
  - использование регулируется Commercial Terms;
  - «Unless previously approved, Anthropic does not allow third party developers to offer claude.ai login or rate limits for their products, including agents built on the Claude Agent SDK», то есть нужна аутентификация по API-ключу;
  - официально SDK есть только для Python и TypeScript; из других языков (Go/Ruby) CLI запускают подпроцессом с `-p` и `--output-format json`.

  — [Claude Code docs: Agent SDK overview](https://code.claude.com/docs/en/agent-sdk/overview) (открыт)
- LLM-шлюзы для Claude Code:
  - «Anthropic doesn't endorse, maintain, or audit third-party gateway products, and doesn't support routing Claude Code to non-Claude models through any gateway»;
  - трафик через шлюз оплачивается по токенам владельцем ключа (Anthropic Console, Amazon Bedrock, Google Cloud Agent Platform, Microsoft Foundry);
  - шлюз нужно обновлять вслед за релизами Claude Code, иначе соответствующие функции ломаются.

  — [Claude Code docs: Other LLM gateways](https://code.claude.com/docs/en/llm-gateway) (открыт)
- Блокировки (по сниппету поиска; официального заявления Anthropic не найдено):
  - по сообщениям СМИ, в мае 2026 доступ к Claude потеряли «несколько сотен» пользователей из РФ, включая аккаунты Pro и Claude Code. Источник — Telegram-канал Baza. Части пользователей вернули деньги, Anthropic ситуацию не комментировала;
  - 2 октября 2026 появились новые жалобы пользователей из России и Гонконга на блокировки после подключения через VPN;
  - эксперты называют возможными причинами VPN, зарубежные карты и перепродажу подписок посредниками;
  - отдельно отмечается, что Stripe блокирует оплату с выявленных VPN и прокси.

  — [SecurityLab](https://www.securitylab.ru/news/572607.php), [Новая газета Европа, 02.10.2026](https://novayagazeta.eu/articles/2026/10/02/polzovateli-claude-iz-rossii-i-gonkonga-pozhalovalis-na-blokirovki-akkauntov-posle-podkliucheniia-cherez-vpn-news), [Anti-Malware, 02.10.2026](https://www.anti-malware.ru/news/2026-10-02-111332/51615), [BFM](https://www.bfm.ru/news/606080), [Postium](https://postium.ru/claude-massovo-banit-akkaunty-polzovatelej-iz-rossii/), [vc.ru](https://vc.ru/ai/2874009-pochemu-anthropic-banits-akkaunty-iz-rossii)
- Верификация личности. По вторичным источникам, с 8 июля 2026 Anthropic может запрашивать у пользователей Free/Pro/Max физический документ и селфи; провайдер проверки — Persona. Для Team/Enterprise/Developer Platform действуют отдельные условия. — [FourWeekMBA](https://fourweekmba.com/anthropic-claude-id-verification-passport-face-scan/), [Implicator](https://www.implicator.ai/anthropic-adds-passport-checks-to-claude-after-privacy-driven-user-surge/) (по сниппету поиска; по странице Anthropic не проверено)

**OpenAI и агрегаторы**
- В июне 2024 OpenAI уведомила разработчиков, что с 9 июля 2024 будет блокировать API-трафик из неподдерживаемых стран. Текст письма: «Our data shows that your organization has API traffic from a region that OpenAI does not currently support». В публикациях Россия названа среди затронутых стран. Помечаться мог и трафик через edge-сети (Vercel, Cloudflare Workers). — [The Register](https://www.theregister.com/2024/06/25/openai_unsupported_countries/), [The Decoder](https://the-decoder.com/openai-to-restrict-api-access-for-unsupported-countries-in-july/), [OpenAI Community](https://community.openai.com/t/identifying-api-keys-with-traffic-from-unsupported-regions/837837) (по сниппету поиска)
- Официальный список стран OpenAI ([platform.openai.com/docs/supported-countries](https://platform.openai.com/docs/supported-countries)) из среды не открылся. Что РФ в нём нет — в этой сессии «не проверено».
- OpenRouter (по сниппету поиска; данные по датам фактически из одного источника, официального заявления OpenRouter не найдено):
  - по блогу SSD Nodes, в конце мая 2026 аккаунты с российским платёжным адресом получали ошибку «Your billing address is in a region that does not have access to models from OpenAI, Anthropic, and Google»; открытые модели при этом работали;
  - с 27 июня 2026 запросы с российских IP получали HTTP 403 «Not available in your region»;
  - Habr ссылается на письмо OpenRouter от 11.05.2026;
  - ComNews пишет, что OpenRouter «ушёл с российского рынка в июне 2026».

  — [SSD Nodes](https://www.ssdnodes.com/learn/lang/ru/claude-api-from-russia-what-works), [Habr](https://habr.com/ru/articles/1034012/), [ComNews, 03.07.2026](https://www.comnews.ru/content/246176/2026-07-03/2026-w27/1018/cloudru-dobavil-vneshnie-yazykovye-modeli-servis-foundation-models)

**Российские альтернативы**
- В Yandex AI Studio есть группа OpenAI-совместимых API: Models, Chat Completions, Conversations и Responses (агенты). Плюс собственные API Яндекса. Нативные и open-source модели тарифицируются по потреблённым токенам. — [Yandex AI Studio: API](https://aistudio.yandex.ru/docs/en/ai-studio/concepts/api) (по сниппету поиска)
- По странице совместимости поддерживаются Responses API, Realtime API и Vector Store API, Completions API — частично. — [Yandex Cloud: OpenAI compatibility](https://yandex.cloud/en/docs/ai-studio/concepts/openai-compatibility) (по сниппету поиска)
- Function calling в Yandex AI Studio (по сниппету поиска):
  - есть концепт-страница и пошаговое руководство, датированное 18.02.2026;
  - аргументы вызова должны соответствовать JSON Schema;
  - в релиз-нотах февраля 2025 сказано, что function calling в YandexGPT Pro 5-го поколения «значительно улучшен»;
  - есть официальный Python SDK `yandex-ai-studio-sdk` и туториал агента на OpenAI Agents SDK.

  — [Concept](https://yandex.cloud/en/docs/ai-studio/concepts/generation/function-call), [Guide](https://yandex.cloud/en/docs/ai-studio/operations/generation/function-call), [Release notes](https://yandex.cloud/en/docs/ai-studio/release-notes), [SDK](https://github.com/yandex-cloud/yandex-ai-studio-sdk), [туториал](https://yandex.cloud/ru/docs/functions/tutorials/streaming-openai-agent)
- Цены Yandex по таблице документации (по сниппету поиска; таблица могла устареть). За 1000 токенов, с НДС:

  | Модель | Синхронно | Асинхронно |
  |---|---|---|
  | YandexGPT Lite | 0,20 ₽ | 0,10 ₽ |
  | YandexGPT Pro | 1,20 ₽ | 0,60 ₽ |

  Минимальный запуск в пакетном режиме — 200 000 токенов. — [Yandex Foundation Models: тарификация](https://yandex.cloud/ru/docs/foundation-models/pricing)
- Снижение цен по исследованию Nodul, август 2026 (по сниппету поиска):
  - YandexGPT Pro 5.1 — 0,80 ₽ за 1000 токенов (−33% к YandexGPT 5 Pro);
  - YandexGPT Lite — без изменений, 0,20 ₽;
  - GigaChat Lite — с 0,20 до ~0,065–0,07 ₽;
  - GigaChat Pro — с 1,50 до 0,50 ₽;
  - сопоставимые зарубежные модели «до 10 раз дешевле».

  — [AdIndex, 25.08.2026](https://adindex.ru/news/digital/2026/08/25/348304.phtml), [Anti-Malware, 25.08.2026](https://www.anti-malware.ru/news/2026-08-25-111332/51163)
- Тарифы GigaChat API (по сниппету поиска):
  - тарифы для физлиц и юрлиц действуют с 1 февраля 2026;
  - юрлица покупают пакеты или платят по факту (pay-as-you-go). Пример: пакет 300 000 000 токенов GigaChat Lite стоит 19 500 ₽ с НДС, то есть ≈0,065 ₽ за 1000 токенов; пакет действует 12 месяцев;
  - описания функций и история сообщений входят в расход токенов; кэшированные токены не тарифицируются;
  - у физлиц есть Freemium (в сниппете — 365 000 000 бесплатных токенов, период не уточнён).

  — [Тарифы для юрлиц](https://developers.sber.ru/docs/ru/gigachat/tariffs/legal-tariffs), [Тарифы для физлиц](https://developers.sber.ru/docs/ru/gigachat/tariffs/individual-tariffs), [Подсчёт токенов](https://developers.sber.ru/docs/ru/gigachat/guides/counting-tokens)
- Функции и SDK GigaChat (по сниппету поиска):
  - есть раздел «Работа с функциями»: параметр `function_call: "auto"`, встроенная функция text2image;
  - официальные библиотеки — для Python, TypeScript/JavaScript и Java; Python SDK входит в `langchain-gigachat`;
  - перед работой нужно установить сертификаты НУЦ Минцифры.

  — [Function calling](https://developers.sber.ru/docs/ru/gigachat/guides/function-calling), [SDK](https://developers.sber.ru/docs/ru/gigachat/api/python-library), [PyPI gigachat](https://pypi.org/project/gigachat/0.2.3a1/)
- В документации LiteLLM есть страница провайдера GigaChat. — [LiteLLM: GigaChat](https://docs.litellm.ai/docs/providers/gigachat) (наличие страницы — по сниппету поиска; содержимое не проверено)
- Cloud.ru Evolution Foundation Models (по сниппету поиска):
  - каталог моделей (GigaChat, Qwen, DeepSeek и др.) доступен через OpenAI-совместимый API;
  - бесплатный доступ к open-source моделям действовал до 31.10.2025;
  - в ноябре 2025 добавлена GigaChat Lightning;
  - в июле 2026 добавлены внешние модели Alibaba, DeepSeek и Z.ai с оплатой по факту.

  — [Habr Cloud.ru](https://habr.com/ru/companies/cloud_ru/posts/927880), [ComNews, 14.08.2025](https://www.comnews.ru/content/240714/2025-08-14/2025-w33/1018/cloudru-otkryl-besplatnyy-dostup-k-open-source-llm-modelyam-vklyuchaya-120b-openai-i-qwen3-480b), [ComNews, 24.11.2025](https://www.comnews.ru/content/242510/2025-11-24/2025-w48/1018/cloudru-otkryl-besplatnyy-dostup-k-novoy-modeli-lineyki-gigachat), [ComNews, 03.07.2026](https://www.comnews.ru/content/246176/2026-07-03/2026-w27/1018/cloudru-dobavil-vneshnie-yazykovye-modeli-servis-foundation-models)

**Шлюзы (LiteLLM)**
- LiteLLM описан как «Open Source AI Gateway for 100+ LLMs». Enterprise-функции распространяются под отдельной коммерческой лицензией (каталог `enterprise`). У репозитория 60,5k звёзд. — [GitHub BerriAI/litellm](https://github.com/BerriAI/litellm) (открыт)
- 24 марта 2026 в PyPI вышли скомпрометированные версии LiteLLM 1.82.7 и 1.82.8 со стилером учётных данных: SSH-ключи, облачные креды, токены Kubernetes, переменные окружения, секреты CI/CD. Атака приписывается группировке TeamPCP. Версия 1.82.8 добавляла `.pth`-файл, который исполняется при старте интерпретатора. — [BleepingComputer](https://bleepingcomputer.com/news/security/popular-litellm-pypi-package-compromised-in-teampcp-supply-chain-attack), [Help Net Security](https://www.helpnetsecurity.com/2026/03/25/teampcp-supply-chain-attacks/) (по сниппету поиска)
- В GitHub Advisory Database по запросу «litellm» — 58 записей. Среди записей 2026 года:
  - «LiteLLM: MCP Authentication Bypass via OAuth2 Passthrough Fallback» (High, 22.07.2026);
  - «Authentication Bypass via Host Header Injection» (Critical, 16.06.2026);
  - «semantic-router exposed to compromised litellm wheel (CVE-2026-42208)» (Critical, 26.06.2026).

  — [GitHub Advisories: litellm](https://github.com/advisories?query=litellm) (открыт)

### Inferences
- **Влияние на build-vs-buy.**
  - Любой «покупной» компонент, чей рантайм жёстко завязан на Claude/OpenAI (в том числе Claude Agent SDK как ядро «директора»), наследует регуляторный риск. Работа из РФ противоречит Supported Regions Policy.
  - Приостановка возможна без предупреждения (Consumer Terms) или с «разумными усилиями» уведомить (Commercial Terms). Волны блокировок 2026 года показывают, что риск реализуется.
  - Для недельного цикла с реальными деньгами это риск остановки бизнеса, а не просто неудобство.
  - Claude Code как инструмент разработки и Claude Code как production-рантайм — разные классы риска. Потеря первого снижает продуктивность, потеря второго останавливает цикл.
- Агрегаторы вроде OpenRouter проблему не решают: по имеющимся данным, в 2026 году они сами ограничили доступ из РФ.
- Российские провайдеры (Yandex AI Studio, Cloud.ru) дают OpenAI-совместимый интерфейс и function calling. Поэтому архитектура «свой оркестратор + OpenAI-совместимый клиент» позволяет менять модель без переписывания кода. Качество tool use у YandexGPT/GigaChat в многошаговых агентных циклах в этой сессии не измерено.
- Шлюз снижает lock-in, но добавляет поверхность атаки: компрометация в PyPI, десятки advisories в 2026, включая обход аутентификации MCP через OAuth2 passthrough. Держать его в одном окружении с ключами ЮKassa и Директа рискованно.
- **Митигирование (предложение):**
  1. Писать ядро «директора» на своём Go/Ruby: недельный конечный автомат, бюджеты, утверждения человека. LLM вызывать через тонкий интерфейс поверх OpenAI-совместимых Chat Completions/Responses, инструменты описывать в JSON Schema, провайдера задавать конфигурацией.
  2. Выстроить цепочку fallback:
     - основной уровень — модели Yandex AI Studio (YandexGPT, Qwen);
     - резерв — GigaChat (через Сбер или Cloud.ru);
     - аварийный — open-weight модель на своей или арендованной GPU.

     Раз в месяц прогонять eval-набор по каждой роли агента на всех провайдерах.
  3. Если шлюз всё же нужен:
     - фиксировать версию и хэши, устанавливать из проверенного lock-файла;
     - запускать в отдельном контейнере без доступа к платёжным и рекламным секретам;
     - подписаться на GitHub Advisories.

     Альтернатива — обойтись без шлюза и вызывать провайдеров напрямую по HTTP: у всех целевых провайдеров есть OpenAI-совместимый API.
  4. Использовать Claude Code только как инструмент разработки, учитывая риск потери аккаунта, и не строить на нём production-зависимость.
  5. Не передавать персональные данные клиентов в промпты зарубежных моделей (см. риск 2 — локализация и трансграничная передача).

### Gaps
- Официальный список стран OpenAI и позиции OpenAI/Anthropic по российским юрлицам не открыты (DNS). Подтверждения Anthropic о причинах волн блокировок 2026 года нет.
- Доступность Claude/GPT для российских физлиц и юрлиц через Amazon Bedrock, Google Vertex и Azure/Foundry не проверена (лимит поиска).
- Не проверены текущие цены Yandex AI Studio на Qwen, DeepSeek и gpt-oss, а также актуальность таблицы YandexGPT в документации. Данные Nodul вторичные.
- Нет независимого бенчмарка function calling у YandexGPT и GigaChat в многошаговых агентных циклах.
- Не проверено, поддерживает ли LiteLLM YandexGPT как провайдера (для GigaChat страница есть).
- Не проверено, распространяется ли правило «>50% владения» на зарубежную компанию, которой владеет физлицо-резидент РФ (в посте речь о компаниях со штаб-квартирой в неподдерживаемом регионе).

## 2. Юридические ограничения smoke-test в РФ

### Takeaway
Связка «реклама в Директе → лендинг с формой → предоплата» попадает под четыре режима:
- **маркировка интернет-рекламы** — ОРД, erid, ЕРИР, штрафы по ст. 14.3 КоАП, сбор 3%;
- **152-ФЗ** — с 01.07.2025 запрещён первичный сбор персональных данных (ПДн) россиян в зарубежных базах; с 01.09.2025 согласие оформляется отдельным документом; нужно уведомление РКН;
- **54-ФЗ** — чек при получении предоплаты и при исполнении;
- **закон о защите прав потребителей** — отказ от предоплаченного заказа до передачи, возврат в течение 10 дней, неустойка 0,5% в день за просрочку передачи.

Для НПД действуют лимит 2,4 млн ₽ дохода в год и запрет перепродажи. Почти все факты раздела взяты из сниппетов: первоисточники (consultant.ru, garant.ru, справка Директа) из среды не открывались.

### Cited Findings

**Маркировка рекламы**
- Маркировка всей интернет-рекламы обязательна с 1 сентября 2022. Данные о размещении передаются в ЕРИР через оператора рекламных данных (ОРД). Erid выдаётся только после того, как рекламодатель внёс данные: кто размещает, договор, формат. «Яндекс ОРД» входит в число официальных ОРД. — [Битрикс24](https://www.bitrix24.ru/journal/markirovka-reklamy), [Kokoc](https://kokoc.com/blog/markirovka-reklamy), [Точка](https://allo.tochka.com/markirovka-reklamy) (по сниппету поиска)
- Яндекс Директ. Сторонний агрегатор описывает Яндекс ОРД как встроенный автоматический оператор для контекстной рекламы: erid присваивается и отчётность в ЕРИР передаётся автоматически. Но рекламодатель должен заполнить реквизиты: юрлицо или ИП, ИНН, договор, формат. — [Toolfox](https://toolfox.ru/services/ord-markirovka-reklamy/markirovka-yandex-direct) (по сниппету поиска). Официальная справка Директа не открыта — «не проверено».
- Посредникам (агентствам, фрилансерам) нужна отдельная отчётность в ОРД и разаллокация. — [Habr](https://habr.com/en/articles/804685) (по сниппету поиска)
- Штрафы за немаркированную рекламу (по сниппету поиска):
  - ответственность по ст. 14.3 КоАП действует с 1 сентября 2023 (закон от 24.06.2023 № 274-ФЗ);
  - в одном материале приведены суммы: граждане — 30–100 тыс. ₽, должностные лица — 100–200 тыс. ₽, юрлица — 200–500 тыс. ₽. Материал описывает законопроект, поэтому суммы нужно сверить с действующей редакцией;
  - практика 2025: штраф 200 000 ₽ за рекламу без erid оставлен в силе (АС г. Москвы, 21.01.2025, № А40-258209/24-17-1770).

  — [ppt.ru](https://ppt.ru/art/shtrafi/kakie-shtrafy-predusmotreny-za-otsutstvie-markirovki-reklamy), [buh.ru](https://buh.ru/news/uchet_nalogi/169057/), [pravo.ru](https://pravo.ru/news/250157/amp/)
- Сбор 3% с интернет-рекламы с 1 апреля 2025 (по сниппету поиска):
  - основание — ст. 18.2 закона «О рекламе»; порядок установлен постановлением Правительства от 15.08.2025 № 1224;
  - платит сторона, первой получившая деньги рекламодателя по договору: распространитель, оператор рекламной системы или посредник;
  - сумму рассчитывает Роскомнадзор по данным ЕРИР;
  - для Директа по общему правилу плательщик — Яндекс, но расходы могут перекладываться в цену. Прямого подтверждения нет.

  — [Контур](https://kontur.ru/articles/190), [Click.ru](https://blog.click.ru/market-news/mincifry-utverdilo-pravila-rascheta-sbora-3-s-doxodov-ot-internet-reklamy/), [eLama](https://elama.ru/blog/faq-po-sboru-3-za-dohod-ot-reklamy-otvechayut-eksperty-elama/)

**152-ФЗ**
- С 1 июля 2025 ч. 5 ст. 18 152-ФЗ (в редакции закона от 28.02.2025 № 23-ФЗ) прямо запрещает при сборе ПДн граждан РФ использовать базы данных за пределами РФ для записи, систематизации, накопления, хранения, уточнения и извлечения. Акцент — на первичном сборе; исключения узкие. — [Delret, LT in focus (PDF)](https://storage.delret.ru/lt-in-focus/lt-in-focus-lokalizaciya-personalnyh-dannyh.pdf), [Tproger](https://tproger.ru/articles/it-zakonodatelstvo-2025--razbor-izmenenij-dlya-biznesa-i-specialistov) (по сниппету поиска)
- Штрафы за нарушение локализации, ст. 13.11 КоАП, ч. 8–9 (по сниппету поиска; суммы сверить с КоАП):
  - юрлица — 1–6 млн ₽, при повторном нарушении — 6–18 млн ₽;
  - должностные лица — 100–200 тыс. ₽, при повторном — 500–800 тыс. ₽;
  - ИП отвечают как юрлица;
  - самозанятые без статуса ИП — 30–50 тыс. ₽ (по одному источнику).

  — [vitvet.com](https://vitvet.com/articles/koap/sostavy/shtraf-152-fz-2026-tablitsa/), [techora.ru, 31.08.2026](https://techora.ru/news/v-rossii-shtraf-do-18-mln-2026-08-31)
- Мнение юриста: первичный сбор ПДн (например, email) сразу на зарубежные серверы формально нарушает требование локализации, независимо от согласия пользователя. — [Харант, вопрос-ответ](https://harant.ru/questions/q-145519/) (по сниппету поиска; ответ 2024 года)
- С 1 сентября 2025 согласие на обработку ПДн оформляется отдельно от других документов (ст. 9 в редакции закона от 24.06.2025 № 156-ФЗ). Нарушение требований к согласию — ч. 2 ст. 13.11 КоАП: должностные лица — 100–300 тыс. ₽, юрлица — 300–700 тыс. ₽. — [Гарант](https://www.garant.ru/article/1862510/), [Бухэксперт8](https://buhexpert8.ru/1s-buhgalteriya/kadry-i-zarabotnaya-plata/kadrovye-dokumenty/s-1-sentyabrya-2025-oformlyajte-soglasie-na-obrabotku-persdannyh-otdelnym-dokumentom.html) (по сниппету поиска)
- С 30 мая 2025 (закон от 30.11.2024 № 420-ФЗ) наказуемо не подать или подать с опозданием уведомление в Роскомнадзор о намерении обрабатывать ПДн (ч. 10 ст. 13.11 КоАП). По одному источнику, штраф для организаций и ИП — до 300 000 ₽. — [Харант](https://harant.ru/blog/drugoe/kto-i-kogda-obyazan-uvedomit-roskomnadzor-ob-obrabotke-personalnyh-dannyh/), [buh.ru](https://buh.ru/articles/navigator-po-materialam-ob-otvetstvennosti-v-sfere-personalnykh-dannykh.html) (по сниппету поиска)

**54-ФЗ**
- Чеки при предоплате (по сниппету поиска):
  - при 100% предоплате до передачи товара в момент получения денег пробивается чек с признаком «предоплата 100%» (тег 1214);
  - при передаче товара — чек «полный расчёт» с зачётом предоплаты;
  - признак «аванс» используется, когда состав заказа ещё не определён;
  - так разъясняет Минфин (письмо от 13.06.2024 № 30-01-15/54757).

  — [Яндекс Пэй, блог](https://pay.yandex.ru/blog/articles/chek-na-predoplatu), [buh.ru](https://buh.ru/news/kak-primenyat-kkt-i-formirovat-kassovye-cheki-pri-prodazhe-tovara-s-polnoy-predoplatoy.html), [v2b.ru (текст письма)](https://www.v2b.ru/documents/pismo-minfina-rossii-ot-13-06-2024-30-01-15-54757/)
- Самозанятые формируют чек в «Мой налог». Например, 1С автоматически регистрирует доход в «Мой налог» при оплате через СБП или ЮKassa. Пробить чек сверх лимита «Мой налог» не даёт. — [buh.ru (1С)](https://buh.ru/news/samoe-novoe-v-1s-bukhgalterii-8-avtomaticheskaya-registratsiya-dokhodov-v-servise-fns-moy-nalog.html), [Моё дело](https://www.moedelo.org/club/article-knowledge/limit-samozanyatogo) (по сниппету поиска)
- ЮKassa для самозанятых — источники противоречат друг другу. Пресс-релиз времён запуска: подключение бесплатное, 1% (+НДС) за каждый автоматически отправленный чек. Свежий обзор утверждает, что встроенную поддержку чеков самозанятых прекратили. — [Spark (пресс-релиз)](https://spark.ru/user/128977/blog/96764/yukassa-zapustila-servis-avtomaticheskih-chekov-dlya-samozanyatih), [Quasa](https://quasa.io/ru/media/yukassa-ili-robokassa-dlya-samozanyatogo-komissii-cheki-i-usloviya-podklyucheniya) (по сниппету поиска; «не проверено»)

**Защита прав потребителей**
- Ст. 26.1 ЗоЗПП (дистанционная продажа): потребитель вправе отказаться от товара в любой момент до передачи, даже после оплаты, и в течение 7 дней после передачи. Продавец возвращает деньги не позднее 10 дней с момента требования. — [Харант: как вернуть деньги за онлайн-покупку](https://harant.ru/blog/zashchita-prav-potrebitelej/kak-vernut-dengi-za-onlajn-pokupku-instrukcziya-2025/), [Харант: дистанционная продажа](https://harant.ru/blog/zashchita-prav-potrebitelej/prodazha-tovara-distanczionnym-sposobom/) (по сниппету поиска)
- Ст. 23.1 (нарушен срок передачи предоплаченного товара): потребитель вправе потребовать возврат предоплаты и неустойку 0,5% от её суммы за каждый день просрочки. — там же (по сниппету поиска)
- В Госдуме лежит законопроект № 396348-8, ограничивающий плату за возврат при дистанционной торговле (статус не проверен). — [buh.ru](https://buh.ru/news/uchet_nalogi/170038/) (по сниппету поиска)

**НПД vs ИП**
- Лимит НПД — 2,4 млн ₽ дохода за календарный год; учитывается вся выручка. При превышении статус утрачивается с даты поступления суммы сверх лимита. Ставки — 4% с физлиц и 6% с юрлиц и ИП. — [Моё дело](https://www.moedelo.org/club/article-knowledge/limit-samozanyatogo), [Харант](https://harant.ru/blog/nalogi/npd-stavki-naloga-i-limity-kto-mozhet-byt-samozanyatym-strahovye-vznosy/) (по сниппету поиска)
- Самозанятым запрещено перепродавать товары, кроме личного имущества (п. 2 ч. 2 ст. 4 422-ФЗ): продавать можно только товары собственного производства. Законопроект № 396743-8 предлагает отменить запрет (статус не проверен). — [Контур.Эльба](https://kontur.ru/elba/spravka/81251-ogranicheniya_i_zaprety_dlya_samozanyatyh), [КонсультантПлюс: подборка](https://www.consultant.ru/law/podborki/samozanyatye_pereprodazha/) (по сниппету поиска)

### Inferences
- **Влияние на build-vs-buy.** Юридические режимы почти не зависят от того, собрана система или куплена. Но они отсекают целые классы готовых SaaS, которые принимают ПДн россиян на зарубежные серверы: зарубежные конструкторы лендингов и форм, CRM, сервисы рассылок, «AI-компании под ключ» (например, Polsia, см. риск 3). Такие сервисы противоречат ч. 5 ст. 18 152-ФЗ в редакции с 01.07.2025. На практике формы, база лидов и платежи должны жить в РФ: российский хостинг или облако плюс ЮKassa.
- По имеющимся данным, Директ — единственный канал, где маркировку автоматизирует сама площадка, если реквизиты заполнены. Если агент-маркетолог предложит другие каналы (Telegram Ads, VK, посевы у блогеров), их придётся маркировать отдельно через ОРД. Это ручная работа и источник штрафов.
- Предоплата за продукт, которого ещё нет, юридически работает как дистанционная продажа или предзаказ:
  - покупатель может отменить заказ до передачи и получить деньги в течение 10 дней;
  - если обещанный срок пропущен — неустойка 0,5% в день.

  Поэтому в юнит-экономике smoke-test нужен резерв на возвраты и кассовые операции: чек «предоплата 100%», затем «полный расчёт» или чек возврата. Про чек возврата — это мой вывод, см. Gaps.
- НПД подходит для старта, но 2,4 млн ₽ в год — потолок суммарных предоплат по всем тестам. Запрет перепродажи исключает тесты с перепродажей чужих товаров.
- **Митигирование (предложение):**
  1. Ввести «юридический гейт» перед GO; утверждает человек. Чек-лист:
     - уведомление в РКН подано;
     - есть политика обработки ПДн, согласие оформлено отдельным документом или чекбоксом;
     - форма пишет в базу на территории РФ, аналитика и виджеты не отправляют ПДн за рубеж;
     - в Директе заполнены данные рекламодателя и проверено наличие erid;
     - есть оферта предзаказа с датой исполнения и безусловным возвратом;
     - настроены чеки по 54-ФЗ (для НПД — через «Мой налог»);
     - ведётся учёт накопленной выручки относительно лимита НПД.
  2. Ограничить каналы smoke-test Директом, пока не налажен процесс маркировки для остальных.
  3. Формулировать оффер как «предзаказ» с явной датой и условиями возврата. Агенту-маркетологу запретить обещания, которые нельзя выполнить; формулировки проверяет человек на юридическом гейте.
  4. Когда выручка подойдёт к ~70–80% лимита НПД, перейти на ИП (УСН) с онлайн-кассой или облачными чеками.

### Gaps
- Не открыта официальная справка Яндекс Директа: присваивается ли erid автоматически во всех типах кампаний (yandex.ru недоступен, лимит поиска).
- Действующие суммы штрафов по ст. 14.3 и 13.11 КоАП не сверены с consultant.ru или pravo.gov.ru.
- Не найдено, считается ли «fake door»-реклама ещё не существующего продукта недостоверной рекламой (ст. 5 закона «О рекламе»). Не найдена и позиция модерации Директа по предзаказам.
- Ст. 32 ЗоЗПП (отказ от услуг с оплатой фактических расходов — актуально, если продукт является услугой): первоисточник в этой сессии не открыт.
- Не проверены чек при возврате предоплаты (признак «возврат прихода») и момент признания дохода при предоплате на НПД.
- По ЮKassa для самозанятых данные противоречивы: подключение и автоматические чеки не подтверждены.
- Не исследованы роль ЮKassa как оператора или обработчика ПДн и её требования к хранению данных.
- Не найдены пороги УСН и НДС для ИП в 2026 году.

## 3. Накрученная популярность

### Takeaway
Звёзды GitHub — ненадёжный сигнал. Исследование StarScout (CMU, NC State, Socket; препринт на arXiv в декабре 2024, финальная версия на ICSE 2026) нашло 4,5 млн подозрительных звёзд, а в финальной версии — около 6 млн. К июлю 2024 в кампаниях накрутки участвовали ~16% репозиториев с 50+ звёздами. AI/LLM-проекты — среди основных получателей звёзд вне малвари.

Публичных обвинений Paperclip в накрутке не найдено. Соотношение форков к звёздам у него ≈0,17 — в «органическом» диапазоне вторичных эвристик. Главные риски Paperclip другие: релизы держатся на одном мейнтейнере, бэклог PR и issues огромен.

Цифры трекшена Polsia — самоотчёт основателя, и то, что входит в его «ARR», оспаривается.

### Cited Findings
- Препринт «4.5 Million (Suspected) Fake Stars in GitHub: A Growing Spiral of Popularity Contests, Scams, and Malware» (arXiv:2412.13459, декабрь 2024). По сниппету поиска:
  - StarScout нашёл 4,53 млн фейковых звёзд в 22 915 репозиториях от 1,32 млн аккаунтов;
  - консервативная выборка (аномальный всплеск за один месяц, фейки >10% всех звёзд) — 3,1 млн звёзд от 278 тыс. аккаунтов в 15 835 репозиториях;
  - большинство накрученных репозиториев живут считаные дни;
  - звёзды продаются по цене от $0,10 до $2.

  — [arXiv:2412.13459](https://arxiv.org/abs/2412.13459) (из среды не открылся), [heise](https://heise.de/-10223665), [Techzine](https://www.techzine.eu/news/security/127522/fake-stars-undermine-github-4-5-million-fraudulent-stars-discovered/), [Computing](https://www.computing.co.uk/news/2025/security/fake-github-stars-inflating-malicious-repositories)
- Финальная версия статьи (открыт):
  - название — «Six Million (Suspected) Fake Stars on GitHub: A Growing Spiral of Popularity Contests, Spam, and Malware»;
  - авторы — Hao He, Haoqin Yang, Philipp Burckhardt, Alexandros Kapravelos, Bogdan Vasilescu, Christian Kästner;
  - ICSE '26 (Рио-де-Жанейро), DOI 10.1145/3744916.3764531;
  - две эвристики: low-activity и lockstep (алгоритм CopyCatch на полугодовых окнах);
  - датасет выложен на Zenodo (doi:10.5281/zenodo.17009694);
  - авторы предупреждают, что отдельные записи могут быть ложноположительными, и датасет не предназначен для публичного «пристыживания» конкретных репозиториев.

  — [GitHub hehao98/StarScout](https://github.com/hehao98/StarScout)
- Цифры версии ICSE 2026 по вторичным материалам (по сниппету поиска):
  - ~6 млн подозрительных звёзд, 18 617 репозиториев, ~301 тыс. аккаунтов;
  - данные 2019–2024 годов: 20 ТБ, 6,7 млрд событий, 326 млн звёзд;
  - к июлю 2024 в кампаниях участвовали 16,66% репозиториев с ≥50 звёздами (Techzine для препринта даёт ~15,8%);
  - большинство фейковых звёзд продвигают короткоживущие фишинговые и малварные репозитории, остальные — в основном AI/LLM, блокчейн, инструменты и туториалы;
  - эффект продвижения длится меньше двух месяцев, затем накрутка становится «обузой»;
  - на январь 2025 удалено 90,42% помеченных репозиториев и 57,07% аккаунтов;
  - 78 репозиториев с кампаниями попадали в GitHub Trending;
  - цена звезды — $0,03–0,85;
  - по словам инвестора, медианное число звёзд у проектов на стадии seed — 2 850.

  — [ICSE 2026, researchr](https://conf.researchr.org/details/icse-2026/icse-2026-research-track/14/Six-Million-Suspected-Fake-Stars-on-GitHub-A-Growing-Spiral-of-Popularity-Contests) (не открылся), [Peerlist, апрель 2026](https://peerlist.io/saxenashikhil/articles/inside-githubs-fake-star-economy), [Gigazine, 21.04.2026](https://gigazine.net/gsc_news/en/20260421-github-fake-star)
- AI-репозитории 2025–2026: вторичные блоги утверждают, что у Langflow (147k звёзд) 47,9% «подозрительных» звёзд, у Union Labs (74k) — 47,4%. Методика не раскрыта. — [Korben](https://korben.info/en/fake-github-repositories-why-its-a-problem.html), [BuildMVPFast](https://www.buildmvpfast.com/blog/github-fake-stars-agent-seo-open-source-ai-2026) (по сниппету поиска; «не проверено»)
- Эвристики практиков (по сниппету поиска; эвристики не валидированы):
  - соотношение форков к звёздам у органических проектов ~0,10–0,24 (Korben: ~0,16), у накрученных — ниже ~0,05;
  - «25 000 звёзд и 3 коммита за 2 недели» — подозрительно;
  - звёзды учитываются в рейтингах каталогов MCP-серверов, по которым агенты выбирают серверы (мнение автора блога).

  — [Korben](https://korben.info/en/fake-github-repositories-why-its-a-problem.html), [Peerlist](https://peerlist.io/saxenashikhil/articles/inside-githubs-fake-star-economy), [BuildMVPFast](https://www.buildmvpfast.com/blog/github-fake-stars-agent-seo-open-source-ai-2026)
- Инструменты:
  - **Astronomer** оценивает «человечность» старгейзеров по вкладам (коммиты, issues, PR, ревью) и возрасту аккаунтов; сэмплирует ранних и случайных старгейзеров. Репозиторий заархивирован 12.10.2020. — [GitHub Ullaakut/astronomer](https://github.com/Ullaakut/astronomer) (открыт)
  - **Fake Star Audit** (MCP/CLI) ищет всплески, «фермерские» суффиксы логинов, подряд идущие ID аккаунтов, кластеры в одну секунду и машинно-регулярные интервалы. Но сэмплирует только ~100 самых старых и 30 самых новых старгейзеров. — [mcp.so: Fake Star Audit](https://mcp.so/server/fake-star-audit/Armada735) (по сниппету поиска)
  - Ещё есть fake-star-detector (новый, мало используется) и пайплайн на Dagster. — [ecosyste.ms: fake-star-detector](https://awesome.ecosyste.ms/projects/github.com%2Fdidrod205%2Ffake-star-detector), [Dagster blog](https://dagster.io/blog/fake-stars), [frasermarlow/fake-stars](https://github.com/frasermarlow/fake-stars) (по сниппету поиска)
- Paperclip на 2026-10-09 (открыт):

  | Показатель | Значение |
  |---|---|
  | Звёзды | 99,1k |
  | Форки | 16,7k |
  | Watching | 463 |
  | Открытые issues | 2,9k |
  | Открытые PR | 3,4k |
  | Коммиты | 4 930 |

  Лицензия MIT («MIT © 2026 Paperclip Labs, Inc.»). В FAQ: «merged over 2,700 pull requests». Слоган: «The open-source app everyone uses to manage agents at work». — [GitHub paperclipai/paperclip](https://github.com/paperclipai/paperclip)
- История роста Paperclip (по сниппету поиска):
  - запуск — 2 марта 2026, автор @dotta;
  - ~20k звёзд за первую неделю (ThursdAI);
  - 43,9k за месяц по данным GitHub API (OSSInsight, 02.04.2026);
  - на 11.06.2026 — 69 955 звёзд, 12 980 форков, 105 контрибьюторов;
  - «большая часть релизной работы — от одного мейнтейнера»; команда, компания и финансирование не раскрыты (research note);
  - слоган сменился с «zero-human company» на управление агентами.

  — [ThursdAI](https://thursdai.news/companies/paperclip), [OSSInsight](https://ossinsight.io/blog/zero-human-company-2026), [Ry Walker research](https://rywalker.com/research/paperclip), [Fast.io review](https://fast.io/resources/paperclip-ai-review-2026/)
- Поиск не нашёл ни одной публикации, связывающей Paperclip с фейковыми звёздами (на дату поиска). Это отрицательный результат; Hacker News отдельно не проверялся.
- Polsia (по сниппету поиска; самоотчёт, не аудирован).

  Заявления:
  - основатель — Ben Cera (по одному источнику, Victor-Benjamin Broca);
  - он заявлял, что компания приближается к $10 млн годового run rate при единственном сотруднике-человеке;
  - он заявлял о раунде $30 млн при оценке $250 млн; по его словам, data room и due diligence вели агенты;
  - ранее называлась цифра $6,2 млн ARR к апрелю 2026;
  - позже цитировался WSJ: 10 000 платящих клиентов и «на пути» к $10 млн выручки в 2026.

  Критика:
  - ~$10 млн складываются из ~$4,6 млн подписочного ARR, ~$2 млн разовых пакетов задач, ~$2 млн рекламных бюджетов пользователей и прочего;
  - критик в X насчитал ~120 000 созданных проектов при ~8 500 активных;
  - оценка на Trustpilot — ~2,1/5.

  — [RuntimeWire](https://runtimewire.com/article/polsia-ben-broca-10-million-revenue-zero-employees), [RuntimeWire: $30M](https://runtimewire.com/article/polsia-30m-raise-ben-cera-solo-ai-agents), [36Kr](https://www.36kr.com/p/3825813697565316), [AI Weekly](https://aiweekly.co/alerts/polsia-solo-founder-raises-30m-at-250m-valuation), [подкаст «1.5M ARR, Zero (Human) Employees»](https://apx-security-jp.audible.co.jp/podcast/1-5M-ARR-Zero-Human-Employees-Ben-Cera-Polsia/B0GR6P7293)

### Inferences
- **Paperclip.** Расчёты ниже сделаны на основе цифр выше; это не проверка накрутки.
  - Соотношение форков к звёздам: 16,7k / 99,1k ≈ 0,17 (на 11.06.2026: 12 980 / 69 955 ≈ 0,19). С июня по октябрь прибавилось ~3,7k форков на ~29k звёзд, ≈0,13. Всё это в «органическом» диапазоне вторичных эвристик.
  - За ~7 месяцев — около 22 коммитов в день.
  - Watchers составляют ≈0,47% звёзд против ~0,8–0,9% у microsoft/autogen и microsoft/agent-framework. Индикатор не валидирован и годится только как повод проверить.
  - Существеннее другие сигналы: 3,4k открытых PR при ~2,7k смёрженных и релизы, сосредоточенные у одного мейнтейнера (bus factor). См. риск 5.
- **Polsia.** Использовать её цифры как доказательство жизнеспособности модели «автономной компании» некорректно: это самоотчёт, в котором смешаны рекуррентная выручка, разовая выручка и рекламные бюджеты клиентов. Как «buy»-вариант Polsia — зарубежный SaaS, поэтому к ней применимы вопросы оплаты из РФ и 152-ФЗ (риски 1–2).
- **Влияние на build-vs-buy.** При выборе готовых компонентов (агентные фреймворки, MCP-серверы для Директа и Wordstat) звёзды не должны быть критерием. AI/LLM — документированная зона накрутки, а каталоги MCP-серверов учитывают звёзды в ранжировании.
- **Митигирование (предложение):** чек-лист due diligence для каждого OSS-компонента.
  1. Посмотреть кривую star-history. При ступенчатых всплесках взять выборку из 50–100 старгейзеров из окна всплеска и проверить возраст аккаунтов, публичные репозитории, активность, подряд идущие ID.
  2. Посчитать соотношения форков и watchers к звёздам, а также число контрибьюторов с N и более смёрженными PR за 90 дней.
  3. Проверить OpenSSF Scorecard: Maintained, Code-Review, Contributors.
  4. Оценить время первой реакции на issues.
  5. Учитывать зависимые проекты и загрузки — они тоже накручиваются, поэтому только как дополнительный сигнал.
  6. До решения «buy» сделать малый PoC на своих данных.

  Агентам-аналитикам не давать число звёзд как признак при скоринге идей и инструментов.

### Gaps
- Текст статьи (arXiv- и ICSE-версии) напрямую не открыт: arxiv.org и conf.researchr.org не резолвятся. Числа версии ICSE взяты из вторичных обзоров.
- Не найдено первичного рецензируемого анализа накрутки конкретных агентных репозиториев 2025–2026 годов. Цифры по Langflow приведены без методики.
- Для Paperclip не собрана star-history и не сделана выборка старгейзеров: страница stargazers не отдала данные. Обсуждения на Hacker News не проверялись.
- Оригинальная статья WSJ о Polsia и тред основателя в X не открыты.

## 4. Хранение токенов и безопасность MCP

### Takeaway
Спецификация MCP прямо запрещает token passthrough. Она требует проверять, что токен выпущен для этого сервера (audience), а прокси обязаны получать согласие пользователя для каждого динамически зарегистрированного клиента (OAuth 2.1, RFC 9728, RFC 8707). Но авторизация в MCP опциональна, а STDIO-серверы берут креды из окружения.

Реальные инциденты 2025–2026 годов:
- tool poisoning и rug pull;
- первый вредоносный MCP-сервер в npm (postmark-mcp);
- критическая RCE в mcp-remote (CVSS 9,6);
- атака s1ngularity, которая через установленные AI CLI искала секреты;
- вредоносные skills в ClawHub;
- компрометация LiteLLM.

Для токена Директа (права на расход бюджета) и секретного ключа ЮKassa вывод такой: агент, который видит недоверенный контент, не должен держать «пишущие» креды. Тратить деньги можно только через отдельный сервис с лимитами и утверждением человека.

### Cited Findings
- Спецификация авторизации MCP (2025-11-25):
  - «Authorization is OPTIONAL for MCP implementations»;
  - STDIO-реализации «SHOULD NOT follow this specification» и берут креды из окружения;
  - «Authorization servers MUST implement OAuth 2.1…»;
  - «MCP servers MUST implement OAuth 2.0 Protected Resource Metadata» (RFC 9728);
  - клиенты «MUST implement Resource Indicators» (RFC 8707);
  - «MCP servers MUST validate that access tokens were issued specifically for them as the intended audience»;
  - «The MCP server MUST NOT pass through the token it received from the MCP client» и «MUST NOT accept or transit any other tokens»;
  - «MCP proxy servers using static client IDs MUST obtain user consent for each dynamically registered client»;
  - `scopes_supported` — минимальный набор скоупов.

  — [MCP spec 2025-11-25: authorization.mdx на GitHub](https://github.com/modelcontextprotocol/modelcontextprotocol/blob/main/docs/specification/2025-11-25/basic/authorization.mdx) (открыт)
- MCP Security Best Practices (по сниппету поиска):
  - token passthrough описан как источник проблемы confused deputy;
  - он позволяет обойти контроли на стороне downstream-сервисов (rate limiting, валидация, мониторинг), привязанные к audience токена;
  - MCP-прокси MUST реализовать согласие для каждого клиента: реестр одобренных client_id; страница согласия называет клиента и перечисляет скоупы.

  — [MCP: Security Best Practices](https://modelcontextprotocol.io/specification/2025-11-25/basic/security_best_practices)
- Changelog спецификации 2026-07-28 (открыт; статус релиза на странице не указан):
  - протокол стал stateless: удалены сессии, заголовок `Mcp-Session-Id` и handshake `initialize`;
  - добавлена проверка issuer (RFC 9207); креды запрещено переиспользовать с другим authorization server;
  - Dynamic Client Registration объявлена устаревшей в пользу Client ID Metadata Documents;
  - устарели Roots, Sampling, Logging и транспорт HTTP+SSE;
  - есть breaking changes.

  — [MCP 2026-07-28 changelog.mdx](https://github.com/modelcontextprotocol/modelcontextprotocol/blob/main/docs/specification/2026-07-28/changelog.mdx)
- Tool poisoning (Invariant Labs):
  - через описание инструмента `add` сервер заставляет агента слить SSH-ключи и `mcp.json`;
  - «shadowing» перехватывает `send_email` доверенного сервера;
  - «sleeper rug pull»: сервер меняет интерфейс инструмента при второй загрузке и перенаправляет сообщения WhatsApp атакующему;
  - украденные данные прячутся «after many spaces».

  — [GitHub invariantlabs-ai/mcp-injection-experiments](https://github.com/invariantlabs-ai/mcp-injection-experiments) (открыт), [Invariant Labs blog](https://invariantlabs.ai/blog/mcp-security-notification-tool-poisoning-attacks)
- postmark-mcp — первый вредоносный MCP-сервер, обнаруженный в реальном использовании (Koi Security). По сниппету поиска:
  - пакет загружен 15.09.2025;
  - версия 1.0.16 (17.09.2025) добавила одну строку: скрытую копию (BCC) всех писем на домен атакующего;
  - ~1 500 загрузок в неделю;
  - Snyk рекомендует считать скомпрометированными все креды, прошедшие через пакет.

  — [Qualys ThreatPROTECT](https://threatprotect.qualys.com/2025/09/30/malicious-mcp-server-on-npm-postmark-mcp-exploited-in-attack/), [CSO Online](https://csoonline.com/article/4064009/trust-in-mcp-takes-first-in-the-wild-hit-via-squatted-postmark-connector.html), [Snyk](https://snyk.io/de/blog/malicious-mcp-server-on-npm-postmark-mcp-harvests-emails/)
- CVE-2025-6514 (GHSA-6xpm-ggf7-wc3p): в mcp-remote (npm) версий >=0.0.5 и <0.1.16 — внедрение команд ОС при подключении к недоверенному MCP-серверу через подделанный `authorization_endpoint`. CVSS 9,6 (Critical); исправлено в 0.1.16; опубликовано 09.07.2025. — [GitHub Advisory GHSA-6xpm-ggf7-wc3p](https://github.com/advisories/GHSA-6xpm-ggf7-wc3p) (открыт)
- 2026: тайпсквот `mcp-server-sequential-thinking` выдаёт себя за `@modelcontextprotocol/server-sequential-thinking`. Его postinstall-скрипт отправляет hostname, рабочий каталог и платформу на сервер автора. Запись OSV MAL-2026-5484 от 09.06.2026. — [OSV MAL-2026-5484](https://osv.dev/vulnerability/MAL-2026-5484) (по сниппету поиска)
- s1ngularity (Nx, 26.08.2025), по сниппету поиска:
  - вредоносный postinstall собирал переменные окружения, токены GitHub и npm, криптокошельки;
  - если были установлены Claude Code или Gemini CLI (по данным Okta — также Amazon Q), скрипт запускал их с промптом искать секреты в файловой системе;
  - результаты публиковались в публичные репозитории жертвы;
  - не менее 1,4 тыс. пользователей нашли у себя такие репозитории.

  — [Semgrep](https://ai.semgrep.dev/blog/2025/security-alert-nx-compromised-to-steal-wallets-and-credentials), [Okta](https://www.okta.com/blog/threat-intelligence/the-s1ngularity-attack--when-attackers-prompt-your-ai-agents-to/), [StepSecurity](https://www.stepsecurity.io/blog/supply-chain-security-alert-popular-nx-build-system-package-compromised-with-data-stealing-malware)
- ClawHub (маркетплейс skills для OpenClaw), февраль 2026, по сниппету поиска:
  - Koi Security проверила 2 857 skills, 341 оказались вредоносными;
  - 335 из них через фейковые «prerequisites» ставили Atomic Stealer (кампания ClawHavoc);
  - часть skills выносила креды бота из `~/.clawdbot/.env`;
  - загружать skills в ClawHub по умолчанию может кто угодно;
  - CVE-2026-25253 — RCE в OpenClaw до версии 2026.1.29.

  — [The Hacker News](https://thehackernews.com/2026/02/researchers-find-341-malicious-clawhub.html), [Aviatrix](https://aviatrix.ai/threat-research-center/openclaw-2026-clawhub-malicious-skills/)
- Claude Code (открыт):
  - Manual mode стартует только с правами на чтение; есть правила allow/ask/deny;
  - песочница изолирует файловую систему и сеть; `curl` и `wget` не одобряются автоматически;
  - «A `-p` session shows neither prompt»: в режиме `-p` нет ни диалога доверия к папке, ни подтверждения серверов из `.mcp.json`;
  - креды хранятся в macOS Keychain или в файле с правами 0600 на Linux;
  - Anthropic «does not security-audit or manage any MCP server»; рекомендовано писать свои MCP-серверы или брать их у доверенных провайдеров;
  - для работы с недоверенным контентом рекомендованы VM или devcontainer.

  — [Claude Code docs: Security](https://code.claude.com/docs/en/security)
- API Яндекс Директа (по сниппету поиска):
  - приложение действует от имени пользователя Директа: «Приложению доступны только те действия, которые доступны пользователю, для которого получен токен»;
  - доступ к API можно ограничить по IP;
  - заявку на доступ к API должны одобрить;
  - для разработки есть отладочный токен;
  - отдельного OAuth-скоупа «только чтение» на найденных страницах нет.

  — [Директ API: авторизационные токены](https://yandex.ru/dev/direct/doc/ru/concepts/auth-token), [Директ API: доступ и авторизация](https://yandex.ru/dev/direct/doc/ru/concepts/access)
- ЮKassa:
  - официальный Python SDK аутентифицируется парой «идентификатор магазина + секретный ключ» (`Configuration.configure`) или OAuth-токеном (`configure_auth_token`);
  - в примере обработчика webhook IP отправителя проверяется через `SecurityHelper().is_ip_trusted(ip)`.

  — [GitHub yoomoney/yookassa-sdk-python](https://github.com/yoomoney/yookassa-sdk-python), [пример конфигурации и webhook](https://github.com/yoomoney/yookassa-sdk-python/blob/master/docs/examples/01-configuration.md) (открыт)

  Ключ выпускается в разделе «Интеграции → Ключ API»; после перевыпуска старый перестаёт работать. — [Salebot docs](https://docs.salebot.pro/integration/payments/priem-platezhei-v-bote-cherez-yandeks.kassu) (по сниппету поиска)
- LiteLLM как MCP-шлюз: advisories «MCP Authentication Bypass via OAuth2 Passthrough Fallback» (High, 22.07.2026) и «MCP Proxy Has Improper Authentication» (Moderate, 21.06.2026). — [GitHub Advisories: litellm](https://github.com/advisories?query=litellm) (открыт)

### Inferences
- Токен Директа означает право тратить деньги: по документации токен даёт приложению все действия пользователя, поэтому ограничить его можно только правами самого пользователя (вывод из сниппета). Агент-маркетолог, который держит такой токен и одновременно читает недоверенный контент (идеи, страницы конкурентов, ответы инструментов), — классическая мишень для prompt injection, ведущей к нецелевому расходу.
- Готовые community MCP-серверы для Директа и ЮKassa (вариант «buy») несут риски supply chain: npm/PyPI, rug pull, единственный мейнтейнер. Обычно они получают токен через переменные окружения процесса (STDIO), то есть в обход механизмов OAuth из спецификации MCP. Отсюда:
  - для «пишущих» денежных операций безопаснее build — тонкий собственный сервис;
  - для чтения (Wordstat, отчёты, прогноз) buy допустим при фиксации версий и изоляции.
- Headless-режим Claude Code (`-p`) не показывает диалогов доверия. Если использовать его как рантайм, защиту нужно задавать конфигурацией (deny-правила, песочница, managed MCP), а не полагаться на подтверждения.
- **Митигирование (предложение):**
  1. Разделить креды Директа:
     - аналитику — отдельный аккаунт или представитель Директа с правами только на просмотр (если такая роль есть) и свой токен;
     - «пишущий» токен — только у собственного spend-сервиса на Go/Ruby, вне контекста LLM;
     - LLM формирует заявку; сервис проверяет лимиты (на тест, на неделю, дневной бюджет кампании), ставит заявку в очередь на утверждение человеком и исполняет после подтверждения;
     - включить IP-ограничение доступа к API Директа.
  2. ЮKassa:
     - секретный ключ — только на сервере, в хранилище секретов (Yandex Lockbox, Vault, systemd credentials); никогда в промптах, логах и `.mcp.json`;
     - агенту — только эндпоинт «создать ссылку на оплату» с потолком суммы;
     - webhooks: проверка IP плюс повторный запрос статуса платежа по API;
     - ключи идемпотентности;
     - отдельный тестовый магазин для разработки;
     - плановая ротация ключей и внеплановая — при любом supply-chain инциденте в зависимостях.
  3. MCP-гигиена:
     - whitelist серверов (managed MCP);
     - фиксация версий и хэшей, никаких `npx -y …@latest`;
     - сторонние серверы — в контейнере без лишнего egress;
     - ревью описаний инструментов при каждом обновлении (защита от rug pull);
     - никакого token passthrough;
     - для своих HTTP-серверов — токены, привязанные к audience.
  4. Не запускать агента с `--dangerously-skip-permissions` или широкими allow-правилами на машине, где лежат платёжные или рекламные секреты. Отделить dev-окружение основателя от production-секретов.
  5. Журналировать все вызовы, меняющие деньги или кампании, и настроить алерты на аномалии: новая кампания, рост ставок, смена домена.

### Gaps
- Не проверены по первоисточнику: есть ли в Директе роль представителя «только чтение», каков срок жизни OAuth-токенов Яндекса и как их отзывать.
- Официальная документация ЮKassa по ограничениям ключей, списку IP для уведомлений и подписи webhooks не открыта: yookassa.ru не резолвится.
- Существующие community MCP-серверы для Директа, Wordstat и ЮKassa не инвентаризированы.
- Страница MCP Security Best Practices целиком не открыта: modelcontextprotocol.io не резолвится, путь в GitHub-репозитории не найден. Статус версии спецификации 2026-07-28 (релиз или кандидат) не подтверждён.
- OWASP Top 10 for LLM (Excessive Agency) и «lethal trifecta» Саймона Уиллисона не процитированы: страницы не открылись.

## 5. Vendor lock-in и заброшенность

### Takeaway
2024–2026 годы показали, как быстро меняется агентный стек:
- AutoGen перешёл в режим поддержки (maintenance) с переездом на Microsoft Agent Framework; параллельно существует форк AG2;
- OpenAI закрыла Assistants API (план — 26.08.2026);
- LangGraph Platform переименовали в LangSmith Deployment, self-hosted доступен только в Enterprise;
- эталонные MCP-серверы (в том числе GitHub, Postgres, Slack) заархивированы без security-обновлений;
- в спецификации MCP 2026-07-28 есть breaking changes;
- популярный AgentGPT (36k звёзд) заархивирован в январе 2026.

Лицензии смещаются к source-available: n8n Sustainable Use License, коммерческие enterprise-части LiteLLM, ELv2/SSPL у Elastic, RSAL/SSPL у Redis 8 с опцией AGPL.

Для соло-основателя на Go/Ruby это аргумент держать оркестрацию и бизнес-логику у себя, а готовые компоненты подключать через стабильные протоколы и заменяемые адаптеры.

### Cited Findings
- AutoGen: «AutoGen is now in maintenance mode»; дальше проект поддерживает сообщество; «New users should start with Microsoft Agent Framework»; есть гайд миграции. Agent Framework назван «the enterprise‑ready successor to AutoGen». У репозитория 61,3k звёзд. — [GitHub microsoft/autogen](https://github.com/microsoft/autogen) (открыт)
- Microsoft Agent Framework: Python и .NET (Go — в отдельном репозитории microsoft/agent-framework-go), лицензия MIT, ~14k звёзд, гайды миграции с Semantic Kernel и AutoGen. — [GitHub microsoft/agent-framework](https://github.com/microsoft/agent-framework) (открыт)
- AG2: «AG2 (formerly AutoGen): The Open-Source AgentOS». «AG2 Classic» вынесен в ag2ai/ag2-classic. Лицензия Apache-2.0; поддерживается волонтёрами, администраторы — Chi Wang и Qingyun Wu; ~5k звёзд. — [GitHub ag2ai/ag2](https://github.com/ag2ai/ag2) (открыт)
- OpenAI Assistants API: объявлена депрекация с плановым отключением 26 августа 2026; новых функций и моделей не будет. Миграция — на Responses API (+ Conversations), оркестрация вызовов инструментов переносится в код приложения. — [OpenAI Community](https://community.openai.com/t/assistants-api-beta-deprecation-august-26-2026-sunset/1354666), [DEV Community](https://dev.to/mr_manushukla/openai-assistants-api-shuts-down-26-august-2026-a-3-week-migration-sprint-plan-596b) (по сниппету поиска)
- LangGraph: с октября 2025 LangGraph Platform называется «LangSmith Deployment». Библиотека LangGraph — под MIT; self-hosted и гибридное развёртывание — только в Enterprise с индивидуальной ценой; тариф Plus — $39 за место в месяц. — [LangChain blog](https://www.langchain.com/blog/langgraph-platform-announce), [LangChain pricing](https://langchain.com/pricing-langgraph-platform) (по сниппету поиска)
- n8n, Sustainable Use License 1.0: разрешено использовать и модифицировать для внутренних бизнес-целей или некоммерческих и личных целей; распространять — только бесплатно и некоммерчески. Файлы `.ee.` — под n8n Enterprise License. — [GitHub n8n LICENSE.md](https://github.com/n8n-io/n8n/blob/master/LICENSE.md) (открыт)
- Redis: версии 7.2 и ранее — под BSDv3; Redis 8.0+ — на выбор RSALv2, SSPLv1 или AGPLv3. — [GitHub redis LICENSE.txt](https://github.com/redis/redis/blob/unstable/LICENSE.txt) (открыт)
- Elasticsearch: по умолчанию тройная лицензия AGPLv3 / SSPLv1 / Elastic License 2.0; код только под ELv2 лежит в x-pack. — [GitHub elasticsearch LICENSE.txt](https://github.com/elastic/elasticsearch/blob/main/LICENSE.txt) (открыт)
- LiteLLM: enterprise-функции распространяются под отдельной коммерческой лицензией. — [GitHub BerriAI/litellm](https://github.com/BerriAI/litellm) (открыт)
- Эталонные MCP-серверы: репозиторий servers-archived заархивирован 29.05.2025. В нём 14 серверов: AWS KB Retrieval, Brave Search, EverArt, Git, GitHub, GitLab, Google Drive, Google Maps, PostgreSQL, Puppeteer, Redis, Sentry, Slack, SQLite. Предупреждения: «NO SECURITY GUARANTEES ARE PROVIDED FOR THESE ARCHIVED SERVERS», «No security updates or bug fixes will be provided». — [GitHub modelcontextprotocol/servers-archived](https://github.com/modelcontextprotocol/servers-archived) (открыт)
- MCP 2026-07-28: удалены сессии и handshake; устарели Roots, Sampling, Logging, HTTP+SSE и Dynamic Client Registration; удалены методы `ping`, `logging/setLevel` и др. — [changelog.mdx](https://github.com/modelcontextprotocol/modelcontextprotocol/blob/main/docs/specification/2026-07-28/changelog.mdx) (открыт)
- AgentGPT (reworkd) заархивирован 28.01.2026 при 36,3k звёзд и 9,2k форков; в README нет объявления о прекращении. — [GitHub reworkd/AgentGPT](https://github.com/reworkd/AgentGPT) (открыт)
- Astronomer (инструмент поиска фейковых звёзд) заархивирован 12.10.2020. — [GitHub Ullaakut/astronomer](https://github.com/Ullaakut/astronomer) (открыт)
- Claude Code + шлюз: шлюз нужно обновлять вслед за релизами Claude Code, иначе функции ломаются. — [Claude Code docs: LLM gateways](https://code.claude.com/docs/en/llm-gateway) (открыт)
- OpenSSF Scorecard, проверка «Maintained»:
  - заархивированный проект получает минимальный балл;
  - хотя бы 1 коммит в неделю за последние 90 дней — максимальный;
  - активность коллабораторов в issues — частичный балл;
  - проверка работает только для проектов старше 90 дней.

  Проверка «Contributors» смотрит, есть ли недавние контрибьюторы из нескольких компаний. — [OpenSSF Scorecard checks.md](https://github.com/ossf/scorecard/blob/main/docs/checks.md) (открыт)
- Paperclip: «большая часть релизной работы — от одного мейнтейнера»; команда, компания и финансирование не раскрыты (июнь 2026). — [Ry Walker research](https://rywalker.com/research/paperclip) (по сниппету поиска). На 09.10.2026 — 2,9k открытых issues и 3,4k открытых PR. — [GitHub paperclipai/paperclip](https://github.com/paperclipai/paperclip) (открыт)
- OpenRouter как пример lock-in на агрегаторе: в 2026 году он ограничил доступ для российских платёжных адресов и IP. См. риск 1 (по сниппету поиска).

### Inferences
- **Влияние на build-vs-buy.**
  - Купленный агентный фреймворк не избавляет от поддержки: она превращается в отслеживание чужих миграций (AutoGen → Agent Framework, Assistants → Responses, SSE → Streamable HTTP, stateless MCP).
  - Большинство агентных фреймворков написаны на Python, TypeScript или .NET. Для стека Go/Ruby это второй рантайм и вторая цепочка зависимостей со своими supply-chain рисками (см. риск 4).
- Популярность не защищает от заброшенности: AgentGPT с 36k звёзд ушёл в архив. Эталонные MCP-серверы от самих авторов протокола заархивированы без security-гарантий. У community-серверов к API Яндекса риск единственного мейнтейнера ещё выше.
- Source-available-лицензии вроде n8n SUL позволяют использовать продукт во внутреннем бизнесе, но запрещают перепродажу и хостинг для других. Если «LLM-компания» когда-нибудь станет продуктом для третьих лиц, лицензии придётся пересмотреть.
- **Митигирование (предложение):**
  1. Свой тонкий оркестратор на Go/Ruby. Недельный цикл — явный конечный автомат в БД. Промпты, схемы инструментов и правила стоп-факторов — в версионируемых файлах, а не во фреймворк-специфичных конструкциях.
  2. Интеграции — через стабильные внешние протоколы:
     - OpenAI-совместимый HTTP для LLM;
     - MCP только как адаптерный слой, с готовностью к breaking changes спецификации;
     - прямые REST-клиенты к Директу, ЮKassa и ГИР БО.
  3. Критичные сторонние MCP-серверы форкнуть в свою организацию, зафиксировать версию и читать диффы перед каждым обновлением.
  4. Пороговые правила для зависимостей:
     - нет коммитов больше 6 месяцев или проект заархивирован — отказ;
     - Scorecard Maintained ниже заданного уровня — только с планом замены;
     - меньше 2–3 активных мейнтейнеров — допустимо только для заменяемых компонентов;
     - для ядра обязательна OSI-лицензия;
     - пересмотр раз в квартал.
  5. Для каждого внешнего компонента — план выхода: формат экспорта данных, кандидат на замену, оценка миграции в днях.

### Gaps
- Не собраны данные об упадке других агентных библиотек (BabyAGI, SuperAGI и др.) и статистика заброшенности community MCP-серверов (лимит поиска).
- Не подтверждены точная дата перевода AutoGen в режим поддержки и GA-статус Agent Framework: в README нет дат.
- Даты лицензионных переходов Redis (2024, RSAL/SSPL) и Elastic (2024, AGPL) не подтверждены первоисточниками: открыты только текущие файлы лицензий.
- Подтверждения, что Assistants API действительно отключили после 26.08.2026, не найдено.
- Условия и стабильность API Яндекса (Wordstat, прогноз Директа) как источник lock-in не оценены.

## Источники

### 1. LLM-провайдер и доступ из РФ
- https://www.anthropic.com/supported-countries — открыт
- https://www.anthropic.com/news/updating-restrictions-of-sales-to-unsupported-regions — открыт
- https://www.anthropic.com/legal/consumer-terms — открыт
- https://www.anthropic.com/legal/commercial-terms — открыт
- https://code.claude.com/docs/en/agent-sdk/overview — открыт
- https://code.claude.com/docs/en/llm-gateway — открыт
- https://www.securitylab.ru/news/572607.php — по сниппету
- https://novayagazeta.eu/articles/2026/10/02/polzovateli-claude-iz-rossii-i-gonkonga-pozhalovalis-na-blokirovki-akkauntov-posle-podkliucheniia-cherez-vpn-news — по сниппету
- https://www.anti-malware.ru/news/2026-10-02-111332/51615 — по сниппету
- https://www.bfm.ru/news/606080 — по сниппету
- https://postium.ru/claude-massovo-banit-akkaunty-polzovatelej-iz-rossii/ — по сниппету
- https://vc.ru/ai/2874009-pochemu-anthropic-banits-akkaunty-iz-rossii — по сниппету
- https://fourweekmba.com/anthropic-claude-id-verification-passport-face-scan/ — по сниппету
- https://www.implicator.ai/anthropic-adds-passport-checks-to-claude-after-privacy-driven-user-surge/ — по сниппету
- https://www.theregister.com/2024/06/25/openai_unsupported_countries/ — по сниппету
- https://the-decoder.com/openai-to-restrict-api-access-for-unsupported-countries-in-july/ — по сниппету
- https://community.openai.com/t/identifying-api-keys-with-traffic-from-unsupported-regions/837837 — по сниппету
- https://platform.openai.com/docs/supported-countries — не открылся (DNS)
- https://www.ssdnodes.com/learn/lang/ru/claude-api-from-russia-what-works — по сниппету
- https://habr.com/ru/articles/1034012/ — по сниппету
- https://www.comnews.ru/content/246176/2026-07-03/2026-w27/1018/cloudru-dobavil-vneshnie-yazykovye-modeli-servis-foundation-models — по сниппету
- https://aistudio.yandex.ru/docs/en/ai-studio/concepts/api — по сниппету
- https://yandex.cloud/en/docs/ai-studio/concepts/openai-compatibility — по сниппету
- https://yandex.cloud/en/docs/ai-studio/concepts/generation/function-call — по сниппету
- https://yandex.cloud/en/docs/ai-studio/operations/generation/function-call — по сниппету
- https://yandex.cloud/en/docs/ai-studio/release-notes — по сниппету
- https://github.com/yandex-cloud/yandex-ai-studio-sdk — по сниппету
- https://yandex.cloud/ru/docs/functions/tutorials/streaming-openai-agent — по сниппету
- https://yandex.cloud/ru/docs/foundation-models/pricing — по сниппету
- https://adindex.ru/news/digital/2026/08/25/348304.phtml — по сниппету
- https://www.anti-malware.ru/news/2026-08-25-111332/51163 — по сниппету
- https://developers.sber.ru/docs/ru/gigachat/tariffs/legal-tariffs — по сниппету
- https://developers.sber.ru/docs/ru/gigachat/tariffs/individual-tariffs — по сниппету
- https://developers.sber.ru/docs/ru/gigachat/guides/counting-tokens — по сниппету
- https://developers.sber.ru/docs/ru/gigachat/guides/function-calling — по сниппету
- https://developers.sber.ru/docs/ru/gigachat/api/python-library — по сниппету
- https://pypi.org/project/gigachat/0.2.3a1/ — по сниппету
- https://docs.litellm.ai/docs/providers/gigachat — по сниппету
- https://habr.com/ru/companies/cloud_ru/posts/927880 — по сниппету
- https://www.comnews.ru/content/240714/2025-08-14/2025-w33/1018/cloudru-otkryl-besplatnyy-dostup-k-open-source-llm-modelyam-vklyuchaya-120b-openai-i-qwen3-480b — по сниппету
- https://www.comnews.ru/content/242510/2025-11-24/2025-w48/1018/cloudru-otkryl-besplatnyy-dostup-k-novoy-modeli-lineyki-gigachat — по сниппету
- https://github.com/BerriAI/litellm — открыт
- https://bleepingcomputer.com/news/security/popular-litellm-pypi-package-compromised-in-teampcp-supply-chain-attack — по сниппету
- https://www.helpnetsecurity.com/2026/03/25/teampcp-supply-chain-attacks/ — по сниппету
- https://github.com/advisories?query=litellm — открыт

### 2. Юридические ограничения smoke-test в РФ
- https://www.bitrix24.ru/journal/markirovka-reklamy — по сниппету
- https://kokoc.com/blog/markirovka-reklamy — по сниппету
- https://allo.tochka.com/markirovka-reklamy — по сниппету
- https://toolfox.ru/services/ord-markirovka-reklamy/markirovka-yandex-direct — по сниппету
- https://habr.com/en/articles/804685 — по сниппету
- https://ppt.ru/art/shtrafi/kakie-shtrafy-predusmotreny-za-otsutstvie-markirovki-reklamy — по сниппету
- https://buh.ru/news/uchet_nalogi/169057/ — по сниппету
- https://pravo.ru/news/250157/amp/ — по сниппету
- https://kontur.ru/articles/190 — по сниппету
- https://blog.click.ru/market-news/mincifry-utverdilo-pravila-rascheta-sbora-3-s-doxodov-ot-internet-reklamy/ — по сниппету
- https://elama.ru/blog/faq-po-sboru-3-za-dohod-ot-reklamy-otvechayut-eksperty-elama/ — по сниппету
- https://storage.delret.ru/lt-in-focus/lt-in-focus-lokalizaciya-personalnyh-dannyh.pdf — по сниппету
- https://tproger.ru/articles/it-zakonodatelstvo-2025--razbor-izmenenij-dlya-biznesa-i-specialistov — по сниппету
- https://vitvet.com/articles/koap/sostavy/shtraf-152-fz-2026-tablitsa/ — по сниппету
- https://techora.ru/news/v-rossii-shtraf-do-18-mln-2026-08-31 — по сниппету
- https://harant.ru/questions/q-145519/ — по сниппету
- https://www.garant.ru/article/1862510/ — по сниппету
- https://buhexpert8.ru/1s-buhgalteriya/kadry-i-zarabotnaya-plata/kadrovye-dokumenty/s-1-sentyabrya-2025-oformlyajte-soglasie-na-obrabotku-persdannyh-otdelnym-dokumentom.html — по сниппету
- https://harant.ru/blog/drugoe/kto-i-kogda-obyazan-uvedomit-roskomnadzor-ob-obrabotke-personalnyh-dannyh/ — по сниппету
- https://buh.ru/articles/navigator-po-materialam-ob-otvetstvennosti-v-sfere-personalnykh-dannykh.html — по сниппету
- https://pay.yandex.ru/blog/articles/chek-na-predoplatu — по сниппету
- https://buh.ru/news/kak-primenyat-kkt-i-formirovat-kassovye-cheki-pri-prodazhe-tovara-s-polnoy-predoplatoy.html — по сниппету
- https://www.v2b.ru/documents/pismo-minfina-rossii-ot-13-06-2024-30-01-15-54757/ — по сниппету
- https://buh.ru/news/samoe-novoe-v-1s-bukhgalterii-8-avtomaticheskaya-registratsiya-dokhodov-v-servise-fns-moy-nalog.html — по сниппету
- https://www.moedelo.org/club/article-knowledge/limit-samozanyatogo — по сниппету
- https://spark.ru/user/128977/blog/96764/yukassa-zapustila-servis-avtomaticheskih-chekov-dlya-samozanyatih — по сниппету
- https://quasa.io/ru/media/yukassa-ili-robokassa-dlya-samozanyatogo-komissii-cheki-i-usloviya-podklyucheniya — по сниппету
- https://harant.ru/blog/zashchita-prav-potrebitelej/kak-vernut-dengi-za-onlajn-pokupku-instrukcziya-2025/ — по сниппету
- https://harant.ru/blog/zashchita-prav-potrebitelej/prodazha-tovara-distanczionnym-sposobom/ — по сниппету
- https://buh.ru/news/uchet_nalogi/170038/ — по сниппету
- https://harant.ru/blog/nalogi/npd-stavki-naloga-i-limity-kto-mozhet-byt-samozanyatym-strahovye-vznosy/ — по сниппету
- https://kontur.ru/elba/spravka/81251-ogranicheniya_i_zaprety_dlya_samozanyatyh — по сниппету
- https://www.consultant.ru/law/podborki/samozanyatye_pereprodazha/ — по сниппету

### 3. Накрученная популярность
- https://arxiv.org/abs/2412.13459 — не открылся (DNS)
- https://conf.researchr.org/details/icse-2026/icse-2026-research-track/14/Six-Million-Suspected-Fake-Stars-on-GitHub-A-Growing-Spiral-of-Popularity-Contests — не открылся (DNS)
- https://github.com/hehao98/StarScout — открыт
- https://heise.de/-10223665 — по сниппету
- https://www.techzine.eu/news/security/127522/fake-stars-undermine-github-4-5-million-fraudulent-stars-discovered/ — по сниппету
- https://www.computing.co.uk/news/2025/security/fake-github-stars-inflating-malicious-repositories — по сниппету
- https://peerlist.io/saxenashikhil/articles/inside-githubs-fake-star-economy — по сниппету
- https://gigazine.net/gsc_news/en/20260421-github-fake-star — по сниппету
- https://korben.info/en/fake-github-repositories-why-its-a-problem.html — по сниппету
- https://www.buildmvpfast.com/blog/github-fake-stars-agent-seo-open-source-ai-2026 — по сниппету
- https://github.com/Ullaakut/astronomer — открыт
- https://mcp.so/server/fake-star-audit/Armada735 — по сниппету
- https://awesome.ecosyste.ms/projects/github.com%2Fdidrod205%2Ffake-star-detector — по сниппету
- https://dagster.io/blog/fake-stars — по сниппету
- https://github.com/frasermarlow/fake-stars — по сниппету
- https://github.com/paperclipai/paperclip — открыт
- https://thursdai.news/companies/paperclip — по сниппету
- https://ossinsight.io/blog/zero-human-company-2026 — по сниппету
- https://rywalker.com/research/paperclip — по сниппету
- https://fast.io/resources/paperclip-ai-review-2026/ — по сниппету
- https://runtimewire.com/article/polsia-ben-broca-10-million-revenue-zero-employees — по сниппету
- https://runtimewire.com/article/polsia-30m-raise-ben-cera-solo-ai-agents — по сниппету
- https://www.36kr.com/p/3825813697565316 — по сниппету
- https://aiweekly.co/alerts/polsia-solo-founder-raises-30m-at-250m-valuation — по сниппету
- https://apx-security-jp.audible.co.jp/podcast/1-5M-ARR-Zero-Human-Employees-Ben-Cera-Polsia/B0GR6P7293 — по сниппету

### 4. Хранение токенов и безопасность MCP
- https://github.com/modelcontextprotocol/modelcontextprotocol/blob/main/docs/specification/2025-11-25/basic/authorization.mdx — открыт
- https://modelcontextprotocol.io/specification/2025-11-25/basic/security_best_practices — по сниппету
- https://github.com/modelcontextprotocol/modelcontextprotocol/blob/main/docs/specification/2026-07-28/changelog.mdx — открыт
- https://github.com/invariantlabs-ai/mcp-injection-experiments — открыт
- https://invariantlabs.ai/blog/mcp-security-notification-tool-poisoning-attacks — ссылка из репозитория, не открывалась
- https://threatprotect.qualys.com/2025/09/30/malicious-mcp-server-on-npm-postmark-mcp-exploited-in-attack/ — по сниппету
- https://csoonline.com/article/4064009/trust-in-mcp-takes-first-in-the-wild-hit-via-squatted-postmark-connector.html — по сниппету
- https://snyk.io/de/blog/malicious-mcp-server-on-npm-postmark-mcp-harvests-emails/ — по сниппету
- https://github.com/advisories/GHSA-6xpm-ggf7-wc3p — открыт
- https://osv.dev/vulnerability/MAL-2026-5484 — по сниппету
- https://ai.semgrep.dev/blog/2025/security-alert-nx-compromised-to-steal-wallets-and-credentials — по сниппету
- https://www.okta.com/blog/threat-intelligence/the-s1ngularity-attack--when-attackers-prompt-your-ai-agents-to/ — по сниппету
- https://www.stepsecurity.io/blog/supply-chain-security-alert-popular-nx-build-system-package-compromised-with-data-stealing-malware — по сниппету
- https://thehackernews.com/2026/02/researchers-find-341-malicious-clawhub.html — по сниппету
- https://aviatrix.ai/threat-research-center/openclaw-2026-clawhub-malicious-skills/ — по сниппету
- https://code.claude.com/docs/en/security — открыт
- https://yandex.ru/dev/direct/doc/ru/concepts/auth-token — по сниппету
- https://yandex.ru/dev/direct/doc/ru/concepts/access — по сниппету
- https://github.com/yoomoney/yookassa-sdk-python — открыт
- https://github.com/yoomoney/yookassa-sdk-python/blob/master/docs/examples/01-configuration.md — открыт
- https://docs.salebot.pro/integration/payments/priem-platezhei-v-bote-cherez-yandeks.kassu — по сниппету
- https://github.com/advisories?query=litellm — открыт

### 5. Vendor lock-in и заброшенность
- https://github.com/microsoft/autogen — открыт
- https://github.com/microsoft/agent-framework — открыт
- https://github.com/ag2ai/ag2 — открыт
- https://community.openai.com/t/assistants-api-beta-deprecation-august-26-2026-sunset/1354666 — по сниппету
- https://dev.to/mr_manushukla/openai-assistants-api-shuts-down-26-august-2026-a-3-week-migration-sprint-plan-596b — по сниппету
- https://www.langchain.com/blog/langgraph-platform-announce — по сниппету
- https://langchain.com/pricing-langgraph-platform — по сниппету
- https://github.com/n8n-io/n8n/blob/master/LICENSE.md — открыт
- https://github.com/redis/redis/blob/unstable/LICENSE.txt — открыт
- https://github.com/elastic/elasticsearch/blob/main/LICENSE.txt — открыт
- https://github.com/modelcontextprotocol/servers-archived — открыт
- https://github.com/reworkd/AgentGPT — открыт
- https://github.com/ossf/scorecard/blob/main/docs/checks.md — открыт
