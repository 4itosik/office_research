# Слой 5. Лендинги для smoke-тестов с предоплатой через ЮKassa (РФ): что взять готовым, что как референс, что писать самому

> Срез на **2026-10-09**. Как собирались данные и как помечены факты:
> - GitHub-страницы открывались напрямую (WebFetch). Реестры опрашивались через curl (rubygems.org, registry.npmjs.org, pypi.org, proxy.golang.org, packagist.org), pkg.go.dev — через WebFetch. Такие факты идут **без пометки** (проверены напрямую).
> - Российские домены (yookassa.ru, yandex.ru, consultant.ru, tilda.cc и др.) из этой среды не открываются (WebFetch → ENOTFOUND). Всё, что взято с них, помечено **«по сниппету поиска»**: факт взят из сводки поисковой выдачи, а URL — официальная страница из этой выдачи.
> - **«не проверено»** — подтверждения первоисточником нет вовсе.

## 1. Кандидаты слоя (≤5): таблица и deep-dive top-3

### Takeaway
ЮKassa обязательна и в конкурсе не участвует: API v3, Checkout Widget, двухстадийные платежи, «Чеки от ЮKassa» и тестовый магазин. Поверх неё из готового стоит взять три вещи:
- официальный MCP-сервер ЮKassa — для агента;
- AstroWind — как основу шаблона лендинга;
- `rvinnie/yookassa-sdk-go` — только если бэкенд на Go.

Для Ruby официального SDK нет, а community-гемы либо заброшены, либо без пользователей. Тонкий клиент проще написать самому. Community-MCP `@theyahia/yookassa-mcp` поддерживает один автор, и инструменты в нём двигают реальные деньги. Это только референс.

### Cited Findings

#### Таблица кандидатов (метрики на 2026-10-09)

| # | Кандидат | URL | Лицензия | Стек | Подключение к Rails / Go | Активность (аномалии) | Покрывает стадии | Работает в РФ | Секреты (shop_id / secret key) | Вердикт |
|---|---|---|---|---|---|---|---|---|---|---|
| 1 | **Официальный MCP-сервер ЮKassa** | [testing](https://yookassa.ru/developers/payment-acceptance/testing-and-going-live/testing), [smart-payment](https://yookassa.ru/developers/payment-acceptance/integration-scenarios/smart-payment) | сервис ЮKassa, код закрыт (не проверено) | удалённый MCP; транспорт и URL — не проверено | Rails/Go он не нужен: бэкенд работает с REST API v3. В Claude Code подключается как MCP по токену | Анонс в СМИ ([ichip.ru](https://ichip.ru/novosti/ii-nauchili-vystavlyat-scheta-pryamo-v-chatah-s-podderzhkoj-972992), дата не установлена). В доках есть раздел «MCP-сервер ЮKassa» (по сниппету поиска) | Создание платежа (одностадийный «Умный платёж», ограниченный набор параметров) и счёта → ссылка на оплату. Чтение платежей, счетов и возвратов, их списки. **Создание возврата не упоминается** | да | Отдельный MCP-токен из ЛК («Интеграция — Доступы к магазину»), у тестового и боевого магазина токены разные. Секретный ключ магазина агенту не передаётся (по сниппету поиска) | **брать** для агента (тестовые ссылки, мониторинг). Основным путём оплаты в проде его не делать |
| 2 | **@theyahia/yookassa-mcp** (community) | [GitHub](https://github.com/theYahia/yookassa-mcp), [npm](https://registry.npmjs.org/@theyahia%2fyookassa-mcp) | MIT | TypeScript/Node; `@modelcontextprotocol/sdk ^1.12.0`, `zod ^3.24.0` | Claude Code: `npx`, stdio. Rails/Go: можно поднять sidecar на Streamable HTTP + Bearer — не рекомендуется | 4★, 0 forks, 27 коммитов, GitHub-релизов нет. На npm с 2026-03-30, 7 версий, 3.0.0 вышла 2026-06-23. Последний коммит 2026-09-03 (README). **Аномалии:** за 3 месяца версии выросли с 0.0.1 до 3.0.0; у автора целая серия «WWmcp» серверов (есть и [tilda-mcp](https://crossaitools.com/mcp/theyahia/tilda-mcp)); один мейнтейнер | 20 инструментов: платежи (в т.ч. СБП, сплит, рекуррент), возвраты, чеки, выплаты, вебхуки, `get_shop_info` | да | `YOOKASSA_SHOP_ID` / `YOOKASSA_SECRET_KEY` в env, то есть полный секрет оказывается у агента. В HTTP-режиме: `MCP_AUTH_TOKEN`, bind 127.0.0.1, проверка Host/Origin | **референс**: набор инструментов и safety-паттерны. В прод не брать |
| 3 | **rvinnie/yookassa-sdk-go** | [GitHub](https://github.com/rvinnie/yookassa-sdk-go), [proxy.golang.org](https://proxy.golang.org/github.com/rvinnie/yookassa-sdk-go/@latest) | MIT | Go (badge 1.25) | Go: `go get github.com/rvinnie/yookassa-sdk-go`. Rails: н/п | 58★, 43 forks, 50 коммитов, 3 открытых issue. Тег v0.2.1 от 2026-05-02, последний коммит тоже 2026-05-02. Импортируют 7 пакетов ([pkg.go.dev](https://pkg.go.dev/search?q=yookassa)). **Аномалия:** на pkg.go.dev ещё ~20 форков с тем же описанием опубликованы отдельными модулями | Платежи (create/capture/cancel/get/list), возвраты (create/get/list), пример вебхуков. **Чеков и счетов в README нет** | да | `yookassa.NewClient(shopID, secretKey)`; значения приходят из env или secret store приложения | **брать** при Go-бэкенде: закрепить коммит, дописать receipt/invoices |
| 4 | **yookassarb** (Ruby) | [GitHub](https://github.com/Wolframko/yookassarb), [rubygems](https://rubygems.org/api/v1/versions/yookassarb.json) | MIT | Ruby ≥ 3.1, Faraday 1.x/2.x, RSpec | Rails: `gem "yookassarb"` + `Yookassa.configure` в initializer. Go: н/п | 0★, 0 forks, 14 коммитов. Версии 0.1.0 и 0.1.1 вышли в один день, 2026-02-06; всего 397 загрузок. Последний коммит 2026-02-06. **Аномалия:** гем новый и без пользователей; 2026-02-05 переименован из `yookassa`. Альтернатива — гем `yookassa` ([PaymentInstruments/yookassa](https://github.com/paderinandrey/yookassa), бывший paderinandrey): 14★, последняя версия 0.2.0 от 2021-11-01, 9 754 загрузки. Возвраты, чеки и вебхуки там значатся только в «Path to 1.0» | Платежи, возвраты, чеки, вебхуки (подписки + парсинг), счета, выплаты, сделки, настройки | да | `shop_id` / `api_key` или OAuth `auth_token`; для нескольких магазинов — отдельные `Yookassa::Client` | **референс**. Можно взять в проект (vendor) после аудита, но лучше написать свой тонкий клиент |
| 5 | **AstroWind** (шаблон лендинга) | [GitHub](https://github.com/arthelokyo/astrowind) | MIT | Astro v7 + Tailwind v4, Node ≥ 22.22.3, `output: 'static'` | Напрямую с Rails/Go не связан: сборка в CI, статика уходит в бакет. Кнопка оплаты делает fetch к API бэкенда, который создаёт платёж | 6.0k★, 1.7k forks, 1 393 коммита, последний коммит 2026-09-12. **Риск:** v1 в режиме поддержки, v2 обещан на октябрь 2026 | Лендинг: 30+ типизированных секций, 6 примеров в `src/pages/landing/`, SEO/OG/sitemap. В репозитории есть `AGENTS.md` и skills для AI-ассистентов | да (хостинг любой, в т.ч. Yandex Object Storage) | Секретов на странице нет: виджет получает только `confirmation_token` конкретного платежа | **брать** как основу шаблона: форк, версию закрепить |

#### Что не вошло в пятёрку (подробнее в разделе 3)
- **Tilda.** API работает только на чтение и экспорт: `getprojectslist`, `getpageslist`, `getpageexport`, `getpagefullexport`. Доступен только на тарифе Business, ключи передаются в query-строке ([help.tilda.cc/api](https://help.tilda.cc/api), по сниппету поиска). Создавать страницы программно нельзя → **нет** для агентной генерации. Годится как референс и ручной запасной вариант.
- **Creatium, Flexbe, LPmotor.** Публичного API для создания страниц не найдено. Для Creatium есть только сторонние коннекторы ApiMonster ([apimonster.ru](https://apimonster.ru/connector/service/creatium/), по сниппету поиска) → **нет**.
- **Puck** (MIT; 13.5k★, 992 forks, 2 139 коммитов, [GitHub](https://github.com/puckeditor/puck)) и **Puck AI** (beta, облако, `PUCK_API_KEY`) → **референс**: страница описывается JSON-данными по схеме компонентов.
- **GrapesJS** (BSD-3; 26.3k★, [GitHub](https://github.com/GrapesJS/grapesjs)) и **Webstudio** (AGPL-3.0-or-later; 9.0k★, [GitHub](https://github.com/webstudio-is/webstudio)) → **нет**: для еженедельных тестов это избыточно, у Webstudio к тому же AGPL.
- **Сайт в Яндекс Бизнесе** (один сайт на компанию, собирается из профиля) и **Турбо-страницы** (поддержка прекращена в 2025) → **нет**.
- **Mixo** → только **референс** UX, деньги через Stripe. **Robokassa MCP** → запасной вариант, если ЮKassa не пропустит магазин.

#### Deep-dive 1. Официальный MCP-сервер ЮKassa
- Через AI-агентов и AI-платформы можно создавать платежи и счета, получать информацию о платежах, счетах и возвратах, а также списки платежей и возвратов по заданным критериям — [История изменений API](https://yookassa.ru/developers/using-api/changelog); [Тестирование](https://yookassa.ru/developers/payment-acceptance/testing-and-going-live/testing), по сниппету поиска.
- Схема работы: агент выбирает инструмент → AI-платформа обращается к MCP-серверу ЮKassa → сервер создаёт платёж или счёт и возвращает ссылку на оплату — [Умный платёж](https://yookassa.ru/developers/payment-acceptance/integration-scenarios/smart-payment), по сниппету поиска.
- Платёж через MCP — это одностадийный «Умный платёж» с ограниченным набором параметров. Покупатель переходит на страницу ЮKassa и вводит платёжные данные там, то есть карты через агента не проходят — [Умный платёж](https://yookassa.ru/developers/payment-acceptance/integration-scenarios/smart-payment), по сниппету поиска.
- Порядок теста:
  1. В тестовом магазине выпустить токен на странице «Интеграция — Доступы к магазину».
  2. Подключить MCP к AI-платформе.
  3. Попросить агента создать платёж на минимальную сумму и оплатить его кошельком ЮMoney или тестовой картой.
  4. После теста удалить тестовый токен, для реальных платежей подключить токен боевого магазина.

  Выставлять счета через MCP в тестовом магазине нельзя — [Тестирование](https://yookassa.ru/developers/payment-acceptance/testing-and-going-live/testing?lang=ru), по сниппету поиска.
- В доках есть разделы «Основы», «Быстрый старт», «Использование MCP-сервера», «Подключение и настройка», «Инструменты». URL этих разделов в выдаче не появились — [Документация API](https://yookassa.ru/developers), по сниппету поиска.
- В СМИ о запуске: агент сам формирует ссылки для оплаты или выставляет счёт, разработчики для этого не нужны — [ichip.ru](https://ichip.ru/novosti/ii-nauchili-vystavlyat-scheta-pryamo-v-chatah-s-podderzhkoj-972992), по сниппету поиска; дата публикации не установлена.
- Конкурент Robokassa заявляет, что «первыми среди российских платежных агрегаторов» вывели платёжный MCP в открытый доступ — [robokassa.com](https://robokassa.com/blog/news/robokassa-zapuskaet-platezhnuyu-integratsiyu-cherez-ii/), по сниппету поиска. Кто был первым, не проверялось.
- Другие community-MCP для ЮKassa есть в каталогах, их код не проверялся:
  - [glama: YooKassa MCP Server](https://glama.ai/mcp/servers/lav2k15fia) — инструментов меньше, сборка локальная;
  - [Belolipetsky/YooKassa-MCP](https://www.remoteopenclaw.com/mcp/Belolipetsky/YooKassa-MCP);
  - Claude-skill [gulivan/yookassa-skill](https://skillsmp.com/creators/gulivan/yookassa-skill/skill).

  Всё — по сниппету поиска.

#### Deep-dive 2. Go/Ruby-клиенты для бэкенда (rvinnie/yookassa-sdk-go; Ruby — для сравнения)
- **Официальные SDK есть только для PHP и Python.** Сообщество поддерживает .NET (ai-iskuzhin, morpher-ru), Java (DeelTer, dynomake, loolzaaa), Go (rvinnie), Node.js, TypeScript и ещё один Python. **Ruby в списке нет.** Если SDK для языка нет, ЮKassa советует сгенерировать клиент по спецификации API — [Использование SDK](https://yookassa.ru/developers/using-api/using-sdks), по сниппету поиска.
- Официальные SDK уехали с GitHub:
  - PHP `yoomoney/yookassa-sdk-php`: 2 049 337 загрузок на Packagist, репозиторий `git.yoomoney.ru/scm/sdk/yookassa-sdk-php.git` — [packagist search](https://packagist.org/search.json?q=yookassa);
  - Python `yookassa`: версия 3.13.0 от 2026-10-06, MIT, homepage на git.yoomoney.ru — [PyPI JSON](https://pypi.org/pypi/yookassa/json);
  - на GitHub `yoomoney/yookassa-sdk-python` архивирован (обновлён 2022-08-01), `yookassa-github-docs` тоже архивирован — [github.com/yoomoney](https://github.com/yoomoney).
- **rvinnie/yookassa-sdk-go:**
  - 58★, 43 forks, 50 коммитов, 3 открытых issue, GitHub-релизов нет. В README есть платежи (create/capture/cancel/get/list), возвраты (create/get/list), обработка вебхуков с гайдом по локальному тесту, получение информации о магазине. Чеков, счетов и выплат в README нет. Авторизация — `yookassa.NewClient(shopID, secretKey)` и опция `yooopts.WithHTTPClient(...)` — [GitHub](https://github.com/rvinnie/yookassa-sdk-go).
  - Последние коммиты: 2026-05-02 (Go 1.19 → 1.25), 2026-03-04 (PR #22: ctx и закрытие body), 2026-02-25 (path escape), 2026-02-24 (у автоплатежа нет проверки Confirmation) — [commits](https://github.com/rvinnie/yookassa-sdk-go/commits/main).
  - Последняя версия по proxy.golang.org — v0.2.1 от 2026-05-02T17:13:35Z — [proxy](https://proxy.golang.org/github.com/rvinnie/yookassa-sdk-go/@latest). Пакет `yookassa/payment` импортируют 7 модулей, остальные ~24 модуля в выдаче — форки с 0 импортёров — [pkg.go.dev](https://pkg.go.dev/search?q=yookassa).
  - Другая Go-библиотека: `github.com/eclipsemode/go-yookassa-sdk` v1.0.1 от 2025-01-31 — [proxy](https://proxy.golang.org/github.com/eclipsemode/go-yookassa-sdk/@latest).
- **Ruby:**
  - `yookassarb`: 0★, 14 коммитов, все в 2026-02-01..06. Покрывает платежи, возвраты, чеки, вебхуки, счета, выплаты, сделки. Ruby ≥ 3.1, Faraday, RSpec/RuboCop. Мульти-тенант через `Yookassa::Client` — [GitHub](https://github.com/Wolframko/yookassarb), [commits](https://github.com/Wolframko/yookassarb/commits/main); 397 загрузок — [rubygems](https://rubygems.org/api/v1/search.json?query=yookassa).
  - `yookassa`: 0.2.0 от 2021-11-01, 9 754 загрузки — [rubygems versions](https://rubygems.org/api/v1/versions/yookassa.json). Репозиторий переехал в PaymentInstruments/yookassa: 14★, 61 коммит. Возвраты, чеки и вебхуки не реализованы, они только в «Path to 1.0» — [GitHub](https://github.com/paderinandrey/yookassa).
  - `yandex_kassa`: 0.3.6 от 2016-07-05, старый протокол Яндекс.Кассы — [rubygems versions](https://rubygems.org/api/v1/versions/yandex_kassa.json).
- Для Node.js выбор есть, хотя Node и не основной стек: `yookassa-sdk-node` 0.7.0 (2026-08-07), `@webzaytsev/yookassa-ts-sdk` 3.5.4 (2026-06-29), `nestjs-yookassa` 2.3.6 (2026-02-19), типы для виджета `types-yoomoneycheckoutwidget` 0.0.2 (2024-06-13) — [npm search](https://registry.npmjs.org/-/v1/search?text=yookassa&size=25).

#### Deep-dive 3. AstroWind как основа генерируемого лендинга и хостинг в РФ
- arthelokyo/astrowind (раньше onwidget/astrowind): 6.0k★, 1.7k forks, MIT. Astro v7, Tailwind v4, Node ≥ 22.22.3.
  - Внутри больше 30 типизированных секций (hero, pricing, FAQ, testimonials и др.) и 6 примеров лендингов в `src/pages/landing/`.
  - Сборка `output: 'static'` в `dist/`; деплой описан для Netlify, Vercel и Cloudflare.
  - В репозитории есть `AGENTS.md` и skills для AI-ассистентов.
  - v1 в режиме поддержки, v2 обещан на октябрь 2026 — [GitHub](https://github.com/arthelokyo/astrowind).
- Последний коммит 2026-09-12 («Adjust github stars shield»), до него серия правок 2026-09-10 — [commits](https://github.com/arthelokyo/astrowind/commits/main).
- Хостинг статики на своём домене в Yandex Object Storage:
  - бакет называется как домен, в нём включается хостинг, домен смотрит на бакет через CNAME;
  - HTTPS через Certificate Manager: свой сертификат или бесплатный Let's Encrypt; после этого HTTP → HTTPS включается автоматически;
  - TLS 1.0/1.1 не поддерживаются с 2025-08-01.

  [Yandex Cloud docs: own domain](https://yandex.cloud/en/docs/storage/operations/hosting/own-domain), по сниппету поиска.
- Виджет ЮKassa встраивается в статическую страницу:
  - скрипт `checkout-widget.js` грузится с yookassa.ru;
  - объект `YooMoneyCheckoutWidget({confirmation_token, return_url, customization})`;
  - `customization.modal=true` даёт всплывающее окно;
  - методы `render` и `destroy`; если заказ изменился — destroy, новый платёж, новый токен;
  - форма показывает способы оплаты, включённые в магазине;
  - тестовая карта 5555 5555 5555 4477, CVC 123, 3-D Secure 123.

  [Виджет: быстрый старт](https://yookassa.ru/developers/payment-acceptance/integration-scenarios/widget/quick-start), [интеграция](https://yookassa.ru/developers/payment-acceptance/integration-scenarios/widget/integration), [модальное окно](https://yookassa.ru/developers/payment-acceptance/integration-scenarios/widget/additional-settings/modal-window), по сниппету поиска.

### Inferences
- **Набор 1+3+5 закрывает слой почти целиком.** Агент собирает статический лендинг из форка AstroWind, а бэкенд на Rails или Go создаёт платёж и отдаёт `confirmation_token` виджету. Официальный MCP нужен агенту для эксплуатации: тестовые платежи, статусы, списки. Секретный ключ магазина остаётся только в бэкенде.
- **Официальный MCP не заменяет бэкенд в воронке лендинга.** Он создаёт одностадийный «Умный платёж» на один платёж, без холда. Передать в нём собственные `metadata` (ClientId Метрики, `yclid`, idea_id), судя по формулировке «ограниченный набор параметров», скорее всего нельзя (не проверено). Поэтому атрибуцию и холд он не даёт.
- **Ruby:** у ЮKassa небольшой REST (платежи, capture/cancel, возвраты, счета, вебхуки). Писать свой клиент (~200–300 строк) дешевле, чем принимать риск гема с 0★ или заброшенного с 2021 года. `yookassarb` годится как карта эндпоинтов и полей.
- **Go:** `rvinnie` — фактический стандарт сообщества (7 импортёров, он же в списке ЮKassa). Но чеков и счетов там нет, их придётся дописать или расширить DTO.

### Gaps
- URL и транспорт официального MCP, точный список инструментов и параметров, лимиты, scope токена, можно ли создавать возвраты. Разделы «Подключение и настройка» и «Инструменты» в выдачу не попали, yookassa.ru из среды не открывается.
- Лицензия и наличие исходников официального MCP — не проверено.
- Код community-MCP (кроме README theYahia) не аудировался. Скачивания npm за месяц получить не удалось: api.npmjs.org отдаёт 403 через прокси.

## 2. ЮKassa: API для smoke-test, тестовый режим, чеки 54-ФЗ, НПД против ИП/ООО, комиссии, онбординг

### Takeaway
В API v3 есть всё нужное:
- встраиваемый виджет и редирект («Умный платёж»);
- счета через `POST /v3/invoices`, ссылку на оплату ЮKassa может сама отправить по email или SMS;
- двухстадийные платежи с холдом от 2 часов до 7 дней, отмена холда без комиссии;
- полные и частичные возвраты: комиссия не возвращается, кроме возврата в день платежа;
- вебхуки по HTTPS на порт 443 или 8443 с повторами 24 часа;
- тестовый магазин.

Чеки закрывают «Чеки от ЮKassa» для ИП и ООО, без своей ККТ. **Критично:** по changelog ЮKassa с **29.12.2025** прекратила услуги для самозанятых, включая чеки и выплаты. Соло-основателю практически нужен статус **ИП**. Сайт проходит модерацию: оферта, реквизиты, цены, контакты, оплата на подключённом сайте.

### Cited Findings

**Сценарии оплаты**
- Виджет: платёж создаётся через API, в ответе приходит токен для инициализации виджета. Виджет встраивает форму на страницу или в модальное окно и сам обрабатывает неуспешные попытки: показывает ошибку и предлагает повторить — [YooMoney Checkout Widget](https://yookassa.ru/developers/payment-forms/widget), [quick-start](https://yookassa.ru/developers/payment-acceptance/integration-scenarios/widget/quick-start), по сниппету поиска. Название типа подтверждения (`embedded`) — по памяти, не проверено.
- «Умный платёж»: покупатель переходит на страницу ЮKassa, сам выбирает способ оплаты и вводит данные там — [Smart payment](https://yookassa.ru/developers/payment-acceptance/integration-scenarios/smart-payment), по сниппету поиска.
- Запросы к API идут только с сервера магазина — [Quick start](https://yookassa.ru/developers/payment-acceptance/getting-started/quick-start), по сниппету поиска. Авторизация — HTTP Basic (shopId + секретный ключ), заголовок `Idempotence-Key` защищает от двойных списаний при повторах — [README theYahia/yookassa-mcp](https://github.com/theYahia/yookassa-mcp).

**Счета и ссылки «без сайта»**
- Запрос: `POST https://api.yookassa.ru/v3/invoices` с `Idempotence-Key`. В теле `payment_data` (сумма, валюта, `capture`), `cart` и `delivery_method_data.type` = `self`, `email` или `sms`. Ссылка возвращается в объекте счёта — [Выставление счетов: приём платежей](https://yookassa.ru/developers/payment-acceptance/scenario-extensions/invoices/payments), по сниппету поиска.
- Ограничения ([invoices/payments](https://yookassa.ru/developers/payment-acceptance/scenario-extensions/invoices/payments), по сниппету поиска):
  - минимальная сумма 1 ₽ для `self` и email, 100 ₽ для SMS;
  - до 100 счетов в минуту на shopId для `self` и email, до 10 для SMS;
  - срок оплаты до 30 дней, по истечении счёт переходит в `canceled`;
  - по одному счёту возможна только одна оплата.
- Данные для чека при выставлении счёта передаются в `payment_data.receipt`, если чеки формирует ЮKassa — [invoices/payments](https://yookassa.ru/developers/payment-acceptance/scenario-extensions/invoices/payments), по сниппету поиска. В другой сводке сказано обратное: при «Чеках от ЮKassa» данные чека в запросе не передаются — [invoices/receipts](https://yookassa.ru/developers/payment-acceptance/scenario-extensions/invoices/receipts), по сниппету поиска. **Противоречие, не разрешено.**
- Магазин можно завести и «на сайте», и «без сайта». Нескольких магазинов в одном аккаунте можно; новый магазин появляется в статусе «Подключается» — [Добавление магазина](https://yookassa.ru/docs/support/merchant/settings/add), [Управление сервисами](https://yookassa.ru/docs/support/merchant/settings/stores), по сниппету поиска.
- В ЛК можно сделать счёт многоразовым: одна ссылка или QR-код на многих покупателей — [sostav.ru](https://www.sostav.ru/blogs/269097/80142), по сниппету поиска, вторичный источник. Есть ли это в API — не проверено.

**Двухстадийная оплата (холд) — ключевой механизм для smoke-теста**
- Платёж создаётся с `capture=false`. После подтверждения покупателем он переходит в статус `waiting_for_capture`: деньги авторизованы, но не списаны. Дальше его можно списать полностью или частично либо отменить — [Payment process](https://yookassa.ru/developers/payment-acceptance/getting-started/payment-process), по сниппету поиска.
- Срок на списание — от 2 часов до 7 дней в зависимости от способа оплаты, точная граница приходит в `expires_at`. Если ничего не сделать, платёж отменяется с причиной `expired_on_capture` — [Payment process](https://yookassa.ru/developers/payment-acceptance/getting-started/payment-process); [Платежи с предавторизацией](https://yookassa.ru/docs/support/payments/extra/pre-auth), по сниппету поиска.
- Отмена из `waiting_for_capture` возвращает деньги покупателю, и ЮKassa не берёт комиссию. Если возвратов много, ЮKassa сама советует двухстадийные платежи: в течение 7-дневного холда отмена бесплатна — [Refunds (support)](https://yookassa.ru/docs/support/payments/refunding), по сниппету поиска.

**Возвраты**
- Возврат через API бывает полным или частичным, для него нужен id платежа — [Входящие уведомления / тест](https://yookassa.ru/developers/using-api/webhooks); [Refunds API](https://yookassa.ru/developers/payments/refunds), по сниппету поиска.
- Комиссия ЮKassa после возврата не возвращается. Возврат в день платежа считается отменой, и комиссии нет; граница в двух справках указана по-разному, 23:59 и 00:00 МСК. Деньги возвращаются только на тот же способ оплаты. Окно возврата — 3 года (SberPay — 13 месяцев; по картам банки могут отказать для заказов старше 15 месяцев) — [Refunds (support)](https://yookassa.ru/docs/support/payments/refunding); [Q&A q212](https://yookassa.ru/questions/q212/), по сниппету поиска.

**Вебхуки**
- Настраиваются в ЛК («Интеграция — HTTP-уведомления») или через API; при OAuth — только через API. Нужен HTTPS на порту 443 или 8443. Сервер должен ответить HTTP 200, иначе ЮKassa повторяет отправку в течение 24 часов — [Входящие уведомления](https://yookassa.ru/developers/using-api/webhooks); [Настройка HTTP-уведомлений](https://yookassa.ru/docs/support/merchant/payments/http-notifications), по сниппету поиска.
- События: `payment.waiting_for_capture`, `refund.succeeded` (по сниппету поиска, [webhooks](https://yookassa.ru/developers/using-api/webhooks)); `payment.succeeded` ([README theYahia](https://github.com/theYahia/yookassa-mcp)). `payment.canceled` — по памяти, не проверено.

**Тестовый режим**
- Тестовый магазин создаётся в ЛК. В нём работают платежи, возвраты, уведомления и отправка чеков; оплатить можно картой или кошельком ЮMoney, деньги не списываются. Перед тестом кошелька нужно выйти из своего аккаунта ЮMoney. Режим проверки чеков включается в «Настройки — Онлайн-касса» — [Тестовый магазин](https://yookassa.ru/docs/support/merchant/payments/implement/test-store); [Тестирование](https://yookassa.ru/developers/payment-acceptance/testing-and-going-live/testing), по сниппету поиска.

**Чеки 54-ФЗ**
- Чек по 54-ФЗ формирует онлайн-касса; ЮKassa только передаёт ей данные. Письма от ЮKassa фискальными чеками не являются — [Оплата по 54-ФЗ](https://yookassa.ru/developers/54fz/basics), по сниппету поиска.
- «Чеки от ЮKassa» ([yookassa.ru/54fz](https://yookassa.ru/54fz/), [fees](https://yookassa.ru/fees/), по сниппету поиска; на какой именно из этих страниц какая формулировка, сводка не уточнила):
  - сервис НКО «ЮМани» с привлечённым платёжным агрегатором ООО «Аванпост»;
  - свою ККТ, фискальный накопитель и ОФД покупать не нужно; подходит ИП и ООО;
  - включается при подключении или по заявке менеджеру; обслуживание бесплатное, платить нужно за пробитые чеки;
  - сниженная комиссия по картам, ЮMoney, T-Pay, SberPay и Alfa Pay действует только с «Чеками от ЮKassa»;
  - акция для новых юрлиц и ИП идёт с 28.01.2026 по 01.12.2026.
- Цена одного чека официально не найдена. Вторичный источник называет «от 0,1% за чек» — [1000bankov.ru](https://1000bankov.ru/rko/tarify/yoomoney/), по сниппету поиска, не проверено.
- Альтернатива — облачная касса партнёров (например, Эвотор), её настройки прописываются в ЮKassa — [Облачная онлайн-касса](https://yookassa.ru/online-kassa/), по сниппету поиска.

**НПД (самозанятые) против ИП/ООО**
- Changelog: «Starting December 29, 2025, YooMoney will no longer support services for self-employed individuals, including receipts for payments and refunds, as well as payouts to the self-employed» — [receipts/self-employed/basics](https://yookassa.ru/developers/payment-acceptance/receipts/self-employed/basics); [changelog](https://yookassa.ru/developers/using-api/changelog); [payouts/self-employed](https://yookassa.ru/developers/payouts/scenario-extensions/self-employed), по сниппету поиска.
- **Противоречие:** маркетинговая страница для самозанятых всё ещё описывает автоматические чеки в налоговую, работу без онлайн-кассы и комиссии: СБП 0,4% (не более 1 500 ₽), карты и кошелёк 3,3–3,8%, зачисление на карту физлица — [Платежи для самозанятых](https://yookassa.ru/platezhi-dlya-samozanyatyh/), по сниппету поиска. Новостного подтверждения прекращения не найдено. Самая техническая формулировка — в changelog.

**Комиссии**
- Официальная справка по тарифам: при обороте до 3 млн ₽ в месяц карты стоят 3,5% (товары с доставкой, услуги, цифровые товары), благотворительность — 2,8%. Абонентской платы нет, комиссия берётся только с успешных платежей. Тариф Premium — от 3 млн ₽ в месяц, Individual — от 5 млн ₽ — [Тарифы ЮKassa](https://yookassa.ru/docs/support/payments/fees), по сниппету поиска.
- В маркетинге: «от 2,8%» по картам ([tarify-dlya-online-kass](https://yookassa.ru/tarify-dlya-online-kass/)), «от 0,4%» в заголовке тарифов ([fees](https://yookassa.ru/fees/)), СБП «от 0%», но только вместе с другими способами ([sbp](https://yookassa.ru/sbp/)) — по сниппету поиска.
- НДС 22% на комиссию с 01.01.2026 упоминался во фрагменте выдачи, полный текст не найден — не проверено.

**Онбординг и требования к сайту** ([Как подготовить сайт](https://yookassa.ru/docs/support/payments/onboarding/arrangement); [Q&A q282](https://yookassa.ru/questions/q282/); [Q&A q21](https://yookassa.ru/questions/q21/), по сниппету поиска)
- Сайт проверяет служба безопасности. Он должен открываться, быть наполнен контентом и иметь рабочие ссылки.
- Нужен хотя бы один товар с реальной ценой в каждом разделе.
- Оферта или пользовательское соглашение должны лежать в открытом доступе; для услуг — тарифы, для товаров — условия доставки.
- Нужны ИНН, ОГРН, телефон и email.
- Оплата должна проходить на подключаемом сайте, без переадресаций. Нужен SSL.
- Из документов обязателен только паспорт руководителя (ИП — свой) — [Документы и договор](https://yookassa.ru/docs/support/payments/onboarding/docs), по сниппету поиска. Подключение «от 1 дня, полностью онлайн» — [yookassa.ru/54fz](https://yookassa.ru/54fz/), по сниппету поиска.
- Под новый сайт обычно заводят отдельный магазин; одну онлайн-кассу можно использовать для нескольких сайтов — [Управление сервисами](https://yookassa.ru/docs/support/merchant/settings/stores); [Q&A q293](https://yookassa.ru/questions/q293/), по сниппету поиска.

### Inferences
- **Холд вместо «списать и вернуть».** Для недельного smoke-теста естественный путь — `capture=false`. Покупатель подтверждает платёж (сильный сигнал намерения), а мы потом отменяем холд без комиссии и без возвратов. Срок до 7 дней совпадает с недельным циклом, но на коротких методах (2 часа) ломается, поэтому холд стоит ограничить картами (не проверено, какие методы его поддерживают).
- **Статус основателя:** из-за прекращения услуг для НПД (changelog) и потребности в автоматических чеках соло-основателю практичнее **ИП на УСН + «Чеки от ЮKassa»**. Самозанятый остаётся только при ручных чеках в «Мой налог». Заодно это проще для модерации Директа: ОГРНИП.
- **Домен:** модерация идёт по сайту, оплата — «на подключаемом сайте». Значит, все идеи лучше держать на одном зонтичном домене, который человек одобряет и ЮKassa проверяет **один раз**: пути вида `/i/<slug>` или поддомены. Без этого каждый новый домен — новый магазин и новая проверка. Пускает ли ЮKassa поддомены и пути без повторной проверки — не проверено.
- «Без переадресаций» — скорее запрет принимать оплату на непроверенном сайте, чем запрет редиректа на страницу ЮKassa в штатном «Умном платеже» (интерпретация, не проверено). Встраиваемый виджет вопрос снимает.
- Подлинность вебхука проверять повторным `GET` платежа по id и не доверять телу. Это стандартная практика; рекомендацию ЮKassa (IP-allowlist или GET) в источниках не нашёл — не проверено.

### Gaps
- Какие способы оплаты поддерживают двухстадийный режим (СБП?), точный холд по картам — нужен `expires_at` с тестового магазина.
- Когда формируется чек «Чеков от ЮKassa» при двухстадийной оплате: при авторизации или при capture? Нужен ли чек, если холд отменён? Не проверено; уточнить у ЮKassa или бухгалтера.
- Действующая цена чека и НДС на комиссию в 2026 году; принимает ли сейчас ЮKassa платежи от НПД без чеков.
- Можно ли использовать один магазин на нескольких доменах или поддоменах и как модерируются новые страницы на уже одобренном домене.
- Многоразовые счета через API (а не ЛК) и передача `metadata` в счёт.

## 3. Варианты генерации и хостинга лендинга: (a) LLM-статика, (b) российские конструкторы, (c) OSS-билдеры, (d) Яндекс; международные референсы

### Takeaway
Программно создавать страницы агентом реально только в варианте (a): Claude Code пишет или заполняет Astro-шаблон, а статика публикуется в РФ-хостинг (Yandex Object Storage). У российских конструкторов либо нет публичного API на создание страниц (Creatium, Flexbe, LPmotor), либо API только экспортирует (Tilda, тариф Business). OSS-билдеры сильнее всего как референс: Puck даёт модель «страница = JSON» и облачную AI-генерацию в beta. Яндекс своих опций под задачу не даёт: Турбо-страницы закрыты, а сайт в Яндекс Бизнесе — один на компанию.

### Cited Findings

**(a) LLM-сгенерированная статика**
- AstroWind: MIT, 6.0k★, Astro v7 + Tailwind v4, 6 примеров лендингов, static output, `AGENTS.md` и skills для AI-ассистентов, последний коммит 2026-09-12 — [GitHub](https://github.com/arthelokyo/astrowind), [commits](https://github.com/arthelokyo/astrowind/commits/main).
- Yandex Object Storage: хостинг статики на своём домене (бакет назван как домен, CNAME) и HTTPS через Certificate Manager, в том числе бесплатный Let's Encrypt — [Yandex Cloud docs](https://yandex.cloud/en/docs/storage/operations/hosting/own-domain), по сниппету поиска.

**(b) Российские конструкторы**
- **Tilda.** API доступен только на Business. Методы: `getprojectslist`, `getpageslist`, `getpageexport`, `getpagefullexport`; базовый адрес `https://api.tildacdn.info`; `publickey` и `secretkey` передаются параметрами запроса; ключи выдаются в «Site Settings → Export → API Integration» — [help.tilda.cc/api](https://help.tilda.cc/api), по сниппету поиска. Лимит 150 запросов в час — [rollout.com](https://rollout.com/integration-guides/tilda-publishing/api-essentials), по сниппету поиска, вторичный источник.
- Tilda Business даёт до 5 сайтов, расширенные тарифы — 10, 15, 20 или 30 — [Tilda FAQ](https://tilda.cc/en/answers/a/subscription-special-en/), по сниппету поиска.
- Все методы Python-обёртки — чтение; репозиторий архивирован 2022-06-04 — [dotzero/tilda-api-python](https://github.com/dotzero/tilda-api-python), по сниппету поиска.
- ЮKassa подключается к Tilda штатно: в ЛК ЮKassa выбирается способ «платёжный модуль» и CMS Tilda Publishing — [Tilda answers](https://tilda.cc/ru/answers/a/yandex-kassa/), по сниппету поиска.
- **Creatium / Flexbe / LPmotor.** Публичного API для программного создания страниц или white label поиск не нашёл. Creatium интегрируется через сторонний ApiMonster (коннекторы и вебхуки) — [apimonster.ru](https://apimonster.ru/connector/service/creatium/), по сниппету поиска. LPmotor (Mottor) публикует материалы про «Mottor AI для лендингов» — [lpmotor.ru](https://lpmotor.ru/articles/flexbe-i-mottor-ai-dlya-lendingov-avtovoronok-i-konversii-2605), по сниппету поиска; содержимое не проверено.

**(c) Open-source билдеры**
- **Puck:** MIT, 13.5k★, 992 forks, 2 139 коммитов. Редактор — React-компонент `<Puck config data onPublish/>`, рендер — `<Render config data/>`, есть рецепт для Next.js — [GitHub](https://github.com/puckeditor/puck).
- **Puck AI (beta):** `generate()` из `@puckeditor/cloud-client` берёт prompt и конфиг компонентов и возвращает Puck Data. Нужен `PUCK_API_KEY`. Можно подставить ключ своего LLM-провайдера (`ai.providerApiKey`). Есть design mode (новые компоненты на лету). Первый headless-релиз — Puck AI 0.3, ноябрь 2025 — [docs generate](https://puckeditor.com/docs/api-reference/ai/cloud-client/generate), [getting started](https://puckeditor.com/docs/ai/getting-started), [blog 0.3](https://puckeditor.com/blog/puck-ai-03), по сниппету поиска.
- **GrapesJS:** BSD-3, 26.3k★, 4.6k forks. Шаблоны описываются как HTML/CSS. Коммерческий «Studio SDK»; в README AI-генерации нет — [GitHub](https://github.com/GrapesJS/grapesjs).
- **Webstudio:** AGPL-3.0-or-later (пакет анимаций проприетарный, EULA), 9.0k★. «Open source website builder and Webflow alternative», работает в облаке или self-host; AI-функций в README нет — [GitHub](https://github.com/webstudio-is/webstudio).

**(d) Яндекс**
- **Сайт в Яндекс Бизнесе:** бесплатный, собирается автоматически из профиля, адрес `<name>.clients.site`. Метрика подключена сразу. **Для одной компании — один сайт** — [Справка Яндекс Бизнеса](https://yandex.ru/support/business-priority/ru/create-site.html); [Конструктор сайтов](https://business.yandex.ru/tools/website-builder/), по сниппету поиска.
- В Директе есть страница «Бесплатный сайт с настроенной рекламой», первую настройку делают специалисты по заявке — [direct.yandex.ru/constructor](https://direct.yandex.ru/constructor), по сниппету поиска.
- **Турбо-страницы:** Яндекс прекратил поддержку, о чём объявил в феврале; страницы доступны только по прямым ссылкам — [click.ru, апрель 2025](https://blog.click.ru/market-news/yandeks-prekratil-podderzhku-turbo-stranic/), по сниппету поиска, вторичный источник. Судьба «турбо-сайтов» в Директе отдельно не подтверждена.

**Международные (только референс)**
- **Mixo:** AI-лендинг за секунды с формой сбора email («launch an MVP and start collecting waitlist signups»). Интеграция со Stripe заявлена сторонними каталогами — [devtoollab.com](https://devtoollab.com/ai-tools/mixo); [mixo.io](https://www.mixo.io/startup-website-builder), по сниппету поиска.

### Inferences
- **(a) — единственный вариант с полным контролем агента.** Детерминированная сборка, бесплатный хостинг в РФ, юридические блоки зашиты в шаблон.
- **Лучше генерировать данные, а не код.** Агент пишет `content.json` по фиксированной схеме секций (hero, проблема, решение, цена, FAQ, предзаказ, юр-футер), Astro рендерит. Это идея Puck, только без облачной зависимости. Так проще валидировать чек-лист модерации до публикации.
- **Tilda** годится, если человек хочет руками собрать красивую страницу. В конвейере «агент генерирует еженедельно» — нет. Export-only API к тому же требует Business и передаёт secret key в query-строке.
- **Puck AI** стоит пересмотреть, когда понадобится визуальное редактирование человеком. Пока это beta-облако с отдельным ключом, вне РФ (не проверено, где хостится) — лишний внешний секрет и зависимость.

### Gaps
- Craftum, Taplink и другие российские конструкторы не исследовались (лимит вызовов). API Flexbe и LPmotor не подтверждены и не опровергнуты по первоисточнику.
- Durable и Framer как референсы не исследовались.
- Где хостится облако Puck AI и есть ли self-host-режим без облака — не проверено.

## 4. Платёжно-юридический минимум для smoke-test (модерация Директа, реклама, предоплата, возвраты, 54-ФЗ)

### Takeaway
Минимум для легального теста:
- статус ИП;
- на лендинге: реквизиты продавца (ФИО, ОГРНИП, контакты), публичная оферта со **сроком передачи** и условиями отмены/возврата, честная формулировка «предзаказ» без обещаний наличия или сроков, которых нет (38-ФЗ ст. 5 ч. 3 п. 3);
- в объявлениях Директа: сведения о продавце (через профиль Яндекс Бизнеса) и маркировка (ERID ставится автоматически);
- деньги: холд и отмена, а если списали — возврат по требованию в 10 дней (ЗоЗПП ст. 23.1) и чеки «предоплата → полный расчёт / возврат прихода».

### Cited Findings

**Модерация Директа**
- Посадочная страница должна соответствовать объявлению, открываться во всех браузерах и не рекламировать запрещённое. Заглушки «сайт в разработке» и «домен припаркован» не допускаются, как и фишинг. Pop-up и pop-under запрещены, кроме контактных форм. Акции и товары из объявления должны быть видны на странице. Ссылки — прямые, без редиректов. Домены, похожие на Яндекс, запрещены — [Как правильно оформить объявление](https://yandex.ru/support/direct/en/moderation/ad-rules), по сниппету поиска.
- В объявлениях о дистанционной продаже нужны сведения о продавце: юрлицо — наименование, ОПФ, ОГРН, юридический адрес; ИП — ФИО и ОГРНИП; самозанятый — ФИО и ИНН. Чтобы они подставлялись автоматически, создают профиль в Яндекс Бизнесе (раздел «Реквизиты») и привязывают его к кампании — [Требования к объявлениям о дистанционной продаже](https://yandex.ru/support/direct/ru/moderation/categories/goods-and-services), по сниппету поиска.
- Маркировка:
  - Директ сам формирует токен (ERID) на каждый креатив;
  - рекламодатель заполняет «Данные рекламодателя» (это согласие на передачу в ЕРИР) и проверяет автоописания креативов;
  - менять зарегистрированный креатив по существу нельзя — нужен новый токен.

  [Маркировка рекламы в Директе](https://yandex.ru/support/direct/en/technologies-and-services/ad-labelingl), [ОРД Яндекса](https://ord.yandex.ru/), по сниппету поиска. Штрафы — от 10 до 500 тыс. ₽ — [partner.market.yandex.ru](https://partner.market.yandex.ru/chtojournal/kak-markirovat-reklamu/), по сниппету поиска, вторичный источник.

**Закон о рекламе (38-ФЗ)**
- Ст. 5 ч. 3 п. 3: реклама недостоверна, если содержит не соответствующие действительности сведения «об ассортименте и о комплектации товаров, а также о возможности их приобретения в определенном месте или в течение определенного срока» — [КонсультантПлюс, ст. 5](https://www.consultant.ru/document/cons_doc_LAW_58968/f67f81c57fdcdacc2643d19d59369f7e185e1156/), по сниппету поиска.
- Ст. 8: в рекламе товаров при дистанционной продаже указываются сведения о продавце — наименование, место нахождения и ОГРН (для ИП — ФИО и ОГРНИП) — [КонсультантПлюс, ст. 8](https://www.consultant.ru/document/cons_doc_LAW_58968/b606ec51a053c12b78e7ab9a8c007fa723e6f321/), по сниппету поиска.

**Защита прав потребителей (предоплата, дистанционная продажа)**
- Ст. 23.1 ЗоЗПП ([КонсультантПлюс, ст. 23.1](https://www.consultant.ru/document/cons_doc_LAW_305/cd5055f2729d0ba6bf1c3724ba16ca46ea92e0f7/), по сниппету поиска; в выдаче есть пометка о редакции от 17.02.2026 с изменениями с 01.04.2026):
  - договор с предоплатой обязан содержать срок передачи товара;
  - при просрочке потребитель выбирает: новый срок или возврат предоплаты, плюс убытки;
  - неустойка 0,5% в день, не больше суммы предоплаты;
  - требования о возврате удовлетворяются в течение 10 дней.
- Правила дистанционной продажи (ПП РФ № 2463 от 31.12.2020): ИП указывает ФИО, ОГРН, email и/или телефон. Информацию покупатель должен получить до заключения договора (п. 2 ст. 26.1 ЗоЗПП) — [КонсультантПлюс, Правила](https://www.consultant.ru/document/cons_doc_LAW_373622/e4ca7557c9ce6273af99d42c900e8579841fe652/), [подборка по ст. 26.1](https://www.consultant.ru/law/podborki/statya_26.1._distancionnyj_sposob_prodazhi_tovara/), по сниппету поиска.

**Оферта и сайт для ЮKassa**
- Оферта или пользовательское соглашение должны быть опубликованы на сайте; нужны цены, реквизиты и контакты (см. раздел 2) — [Как подготовить сайт](https://yookassa.ru/docs/support/payments/onboarding/arrangement), по сниппету поиска.

**54-ФЗ при предоплате и возврате**
- Предоплата тоже считается расчётом. При полной предоплате нужны два чека: при получении денег и после передачи товара или услуги. Признак способа расчёта — тег 1214 («предоплата», «аванс», «полный расчёт») — [buh.ru](https://buh.ru/news/khroniki-54-fz-kassovye-cheki-pri-prodazhakh-s-polnoy-predoplatoy-i-podgotovka-uproshchentsami-kkt-d.html); [ЮKassa 54-ФЗ](https://yookassa.ru/developers/54fz/basics), по сниппету поиска.
- При возврате формируется чек «возврат прихода»; в тестовом магазине это проверяется в режиме проверки чеков — [Тестирование](https://yookassa.ru/developers/payment-acceptance/testing-and-going-live/testing), по сниппету поиска.

### Inferences
Чек-лист smoke-теста — синтез, не юридическая консультация:
1. **ИП (УСН)**. Профиль в Яндекс Бизнесе с реквизитами, привязанный к кампаниям. «Данные рекламодателя» в Директе заполнены.
2. **Зонтичный домен**, один раз одобренный человеком и прошедший модерацию ЮKassa. На каждом лендинге футер с ФИО, ОГРНИП, ИНН, телефоном и email, ссылки на оферту, политику возврата и отмены и (вероятно) политику ПДн.
3. **Текст:**
   - «Предзаказ / бронь. Старт продаж — не позднее ДД.ММ.ГГГГ»;
   - никаких «в наличии», «отправим сегодня», фейковых остатков и отзывов;
   - явно: «если запуск не состоится — бронь снимается автоматически / деньги возвращаются в течение N дней».

   Это прямо закрывает 38-ФЗ ст. 5 ч. 3 п. 3 и ЗоЗПП ст. 23.1 (срок передачи в договоре).
4. **Деньги:** по умолчанию холд (`capture=false`, только карты) с автоотменой в конце теста — без комиссии, без возвратов и, вероятно, без чеков (не проверено). Если решили списать — чек «предоплата 100%» сейчас и «полный расчёт» при передаче. При отказе — возврат через API и чек «возврат прихода»; «Чеки от ЮKassa» делают это без своей ККТ.
5. **Объявления:** ведут прямо на лендинг без редиректов, тексты совпадают со страницей. Каждое изменение креатива — новый ERID.

### Gaps
- Отдельного правила Директа про «fake door» или предзаказ несуществующего товара не найдено. Оценка опирается на общие требования (релевантность, достоверность) и 38-ФЗ.
- Не проверялись: ст. 32 ЗоЗПП (отказ от услуг) для идей-услуг, 152-ФЗ (политика ПДн при сборе email и телефона), cookie-уведомление для Метрики, ограничения НПД на перепродажу товаров.
- Как 54-ФЗ квалифицирует отменённую авторизацию (холд без списания) — не проверено.

## 5. Измерение: Яндекс Метрика, цели «оплата начата / оплачено», офлайн-конверсии

### Takeaway
На фронте — JavaScript-цели `reachGoal` с суммой и валютой (`checkout_open`, `payment_authorized`). На бэкенде — выгрузка подтверждённых платежей и холдов из вебхуков ЮKassa в Метрику как офлайн-конверсий через Management API. Привязка — по `ClientId` (рекомендуемый вариант) и/или `yclid` из клика Директа. Нативной интеграции ЮKassa → Метрика не найдено, есть только платные no-code коннекторы.

### Cited Findings
- `ym(XXXXXXXX, 'reachGoal', 'TARGET_NAME', {order_price: 3000, currency: "RUB"})` для цели типа «JavaScript-событие». Идентификатор в коде должен совпадать с настройкой цели. Одна и та же цель засчитывается не чаще раза в секунду, на счётчик — до 200 целей. Отладка — `?_ym_debug=1` — [Целевое событие](https://yandex.ru/support/metrica/ru/general/goal-js-event); [Клуб Метрики](https://yandex.ru/blog/metrika-club/kak-pravilno-nastroit-reachgoal-v-formakh), по сниппету поиска.
- Офлайн-конверсии ([Uploading offline conversions](https://yandex.ru/dev/metrika/en/management/openapi/offline_conversions/upload_1); [Импорт офлайн-данных](https://yandex.ru/support/metrica/ru/data/offline-params), по сниппету поиска):
  - загрузка — `POST /management/v1/counter/{counterId}/offline_conversions/upload`, multipart CSV до 1 ГБ, UTF-8, OAuth;
  - колонки: `Target` (обязательна), `DateTime` (Unix time, только прошлое), минимум один из `ClientId` / `UserId` / `Yclid` / `PurchaseId`, необязательные `Price` и `Currency`;
  - статус загрузки — `GET .../offline_conversions/uploading/{id}`;
  - `ClientId` берётся методом `getClientID` и даёт самую точную привязку, это рекомендуемый вариант;
  - `Yclid` — ID клика Директа, работает только для офлайн-конверсий и звонков.
- В отчёте «Офлайн-конверсии» видно, привязалась ли конверсия к сессии и почему нет — [Offline conversions report](https://yandex.ru/support/metrica/en/reports/offline-conversions), по сниппету поиска. Яндекс анонсировал «большое обновление» офлайн-конверсий — [новость Яндекс Рекламы](https://b2b.yandex.ru/adv/news/obnovlenie-v-rabote-s-oflajn-konversiyami-v-metrike), по сниппету поиска, детали не проверены.
- ЮKassa → Метрика без кода: платный коннектор ApiMonster (вебхук ЮKassa превращается в офлайн-конверсию или цель), 30 дней пробно — [apimonster.ru](https://apimonster.ru/connector/bundle/yookassa/yandexMetrika/), по сниппету поиска.

### Inferences
Предлагаемая схема:
1. Лендинг кладёт `ClientId` (`getClientID`) и `yclid` (из URL) в запрос к бэкенду `POST /checkout`. Бэкенд сохраняет их в `metadata` платежа ЮKassa.
2. `reachGoal('checkout_open')` срабатывает на клике по CTA, `reachGoal('payment_authorized', {order_price, currency})` — в callback успеха виджета или на странице `return_url`.
3. Вебхук `payment.waiting_for_capture` (или `payment.succeeded`) вызывает выгрузку офлайн-конверсии `paid_intent` с ClientId, Yclid и Price. Это защищает от потерь клиентских событий (блокировщики, закрытая вкладка).
4. Цели создаются один раз через Management API (`POST /management/v1/counter/{id}/goals` — по сниппету поиска в [Импорт офлайн-данных](https://yandex.ru/support/metrica/ru/data/offline-params)). Дальше на них оптимизируются кампании и считаются ступени бюджетной лестницы.

### Gaps
- Окно атрибуции офлайн-конверсий в 2026 году: в сводке расходятся 21 день и правила CRM-формата — не проверено.
- Насколько быстро загруженные офлайн-конверсии становятся доступны для автостратегий Директа — не проверено.

## 6. Вывод по слою: что взять готовым / что как референс / что писать самому

### Takeaway
**Покупаем** платёжную инфраструктуру и сервисы Яндекса: ЮKassa (API, виджет, холды, чеки, тестовый магазин, официальный MCP), Метрику, ОРД и Object Storage. **Берём готовым** AstroWind и (для Go) `rvinnie/yookassa-sdk-go`. **Пишем сами** то, что делает систему «LLM-компанией»: генератор лендинга по схеме с юр-гардрейлами, платёжный сервис (холд → отмена или списание, вебхуки, атрибуция) и мост в Метрику. Ruby-клиент ЮKassa — тоже свой.

### Cited Findings
- Опорные факты: официальный MCP ЮKassa и его ограничения ([testing](https://yookassa.ru/developers/payment-acceptance/testing-and-going-live/testing), по сниппету поиска); холд с отменой без комиссии ([refunding](https://yookassa.ru/docs/support/payments/refunding), по сниппету поиска); прекращение услуг для НПД ([changelog](https://yookassa.ru/developers/using-api/changelog), по сниппету поиска); официальные SDK только PHP и Python, Ruby в списке нет ([using-sdks](https://yookassa.ru/developers/using-api/using-sdks), по сниппету поиска); Tilda API только экспортирует ([help.tilda.cc/api](https://help.tilda.cc/api), по сниппету поиска); активность AstroWind и rvinnie ([astrowind](https://github.com/arthelokyo/astrowind), [rvinnie](https://github.com/rvinnie/yookassa-sdk-go)).

### Inferences

**Взять готовым (брать)**
1. **ЮKassa API v3:** Checkout Widget (встраиваемый), двухстадийные платежи, Refunds API, вебхуки, тестовый магазин; **«Чеки от ЮKassa»** для ИП вместо своей ККТ.
2. **Официальный MCP-сервер ЮKassa** — инструмент агента для тестовых платежей и ссылок, чтения статусов и списков. Токен отдельный, секретный ключ магазина агенту не даём.
3. **AstroWind (MIT)** — форк как база «landing kit», версию закрепить (до выхода v2 в октябре 2026).
4. **Yandex Object Storage + Certificate Manager** — хостинг статики на домене, одобренном человеком.
5. **Яндекс Метрика** (`reachGoal` + офлайн-конверсии), **ОРД Яндекса** (автомаркировка в Директе), **профиль Яндекс Бизнеса** (сведения о продавце в объявлениях).
6. **Go-бэкенд:** `rvinnie/yookassa-sdk-go` (MIT) с закреплённым коммитом и собственной обёрткой.

**Как референс**
- `@theyahia/yookassa-mcp` (MIT): декомпозиция инструментов, destructive-флаги, проверка `get_shop_info → test:true` перед боевым запуском, `Idempotence-Key`, усиление HTTP-транспорта.
- `yookassarb` (MIT): карта эндпоинтов и полей (receipts, invoices, webhooks) для своего Ruby-клиента.
- Puck и Puck AI: «страница = JSON по схеме компонентов, LLM генерирует данные».
- Tilda: эталон ЮKassa-интеграции в конструкторе и ручной запасной вариант.
- Mixo: UX «идея → лендинг за минуты» (Stripe, вне РФ).
- Robokassa MCP: запасной эквайринг.

**Писать самому**
1. **Генератор лендинга** — skill или агент Claude Code. Конвейер: бриф идеи → `content.json` по схеме → Astro-сборка → линтер чек-листа → деплой после аппрува человеком. Линтер проверяет реквизиты, оферту со сроком передачи, слово «предзаказ», отсутствие «в наличии», релевантность объявлению и отсутствие редиректов.
2. **Платёжный сервис (Rails или Go):**
   - `POST /checkout` создаёт платёж: `capture=false`, встраиваемое подтверждение, `Idempotence-Key`, `metadata` = idea_id, variant, ym ClientId, yclid;
   - обработчик вебхуков перепроверяет платёж через `GET`;
   - по итогам теста — массовая отмена холдов или capture;
   - возвраты через API и сверка.

   Пути `/v3/payments`, `/v3/payments/{id}/capture|cancel`, `/v3/refunds` — по памяти, сверить со [Справочником API](https://yookassa.ru/developers/api) (не проверено). `/v3/invoices` подтверждён сниппетом поиска.
3. **Ruby-клиент ЮKassa** (~200–300 строк на Faraday или Net::HTTP), если бэкенд на Rails.
4. **Мост в Метрику:** цели через Management API и выгрузка офлайн-конверсий по вебхукам.
5. **Гардрейлы текста** против недостоверной рекламы и для соответствия Директу — как часть промпта и линтера.

**Нет**
- Tilda, Creatium, Flexbe и LPmotor как генератор страниц (нет API на создание).
- Турбо-страницы (закрыты). Сайт в Яндекс Бизнесе (один на компанию).
- GrapesJS и Webstudio (избыточно, у Webstudio AGPL).
- Community-MCP в проде (один мейнтейнер, полный секретный ключ у агента).
- Гемы `yookassa` (2021, без возвратов и вебхуков) и `yandex_kassa` (2016).
- Схема на НПД (услуги ЮKassa для самозанятых прекращены с 29.12.2025, по changelog).

### Gaps
- Главные риски, которые надо закрыть одним звонком или тикетом в ЮKassa до запуска:
  1. один магазин на несколько лендингов, пути и поддомены одного домена;
  2. какие методы поддерживают холд и какой срок по картам;
  3. когда формируется чек при холде и отмене;
  4. URL и инструменты официального MCP, можно ли через него делать возвраты;
  5. текущая цена «Чеков от ЮKassa».
- Юридический чек-лист стоит один раз проверить у юриста или бухгалтера, особенно холд без списания по 54-ФЗ и формулировку оферты для «предзаказа несуществующего товара».

## Источники

**ЮKassa — документация и справка (все по сниппету поиска)**
- https://yookassa.ru/developers ; https://yookassa.ru/developers/api ; https://yookassa.ru/developers/using-api/changelog
- https://yookassa.ru/developers/using-api/using-sdks
- https://yookassa.ru/developers/payment-forms/widget ; https://yookassa.ru/developers/payment-acceptance/integration-scenarios/widget/quick-start ; https://yookassa.ru/developers/payment-acceptance/integration-scenarios/widget/integration ; https://yookassa.ru/developers/payment-acceptance/integration-scenarios/widget/additional-settings/modal-window
- https://yookassa.ru/developers/payment-acceptance/integration-scenarios/smart-payment
- https://yookassa.ru/developers/payment-acceptance/getting-started/quick-start ; https://yookassa.ru/developers/payment-acceptance/getting-started/payment-process
- https://yookassa.ru/developers/payment-acceptance/scenario-extensions/invoices/payments ; https://yookassa.ru/developers/payment-acceptance/scenario-extensions/invoices/basics ; https://yookassa.ru/developers/payment-acceptance/scenario-extensions/invoices/receipts
- https://yookassa.ru/docs/support/payments/extra/pre-auth
- https://yookassa.ru/docs/support/payments/refunding ; https://yookassa.ru/developers/payments/refunds ; https://yookassa.ru/questions/q212/
- https://yookassa.ru/developers/using-api/webhooks ; https://yookassa.ru/docs/support/merchant/payments/http-notifications
- https://yookassa.ru/docs/support/merchant/payments/implement/test-store ; https://yookassa.ru/developers/payment-acceptance/testing-and-going-live/testing
- https://yookassa.ru/54fz/ ; https://yookassa.ru/online-kassa/ ; https://yookassa.ru/developers/54fz/basics
- https://yookassa.ru/platezhi-dlya-samozanyatyh/ ; https://yookassa.ru/developers/payment-acceptance/receipts/self-employed/basics ; https://yookassa.ru/developers/payouts/scenario-extensions/self-employed ; https://yookassa.ru/questions/q13/
- https://yookassa.ru/docs/support/payments/fees ; https://yookassa.ru/fees/ ; https://yookassa.ru/sbp/ ; https://yookassa.ru/tarify-dlya-online-kass/
- https://yookassa.ru/docs/support/payments/onboarding/arrangement ; https://yookassa.ru/questions/q282/ ; https://yookassa.ru/questions/q21/ ; https://yookassa.ru/docs/support/payments/onboarding/docs
- https://yookassa.ru/docs/support/merchant/settings/stores ; https://yookassa.ru/docs/support/merchant/settings/add ; https://yookassa.ru/questions/q293/

**Реестры и GitHub (проверено напрямую 2026-10-09)**
- https://github.com/yoomoney
- https://packagist.org/search.json?q=yookassa ; https://pypi.org/pypi/yookassa/json
- https://rubygems.org/api/v1/search.json?query=yookassa ; https://rubygems.org/api/v1/versions/yookassa.json ; https://rubygems.org/api/v1/versions/yookassarb.json ; https://rubygems.org/api/v1/versions/yandex_kassa.json
- https://registry.npmjs.org/-/v1/search?text=yookassa&size=25 ; https://registry.npmjs.org/@theyahia%2fyookassa-mcp
- https://proxy.golang.org/github.com/rvinnie/yookassa-sdk-go/@latest ; https://proxy.golang.org/github.com/eclipsemode/go-yookassa-sdk/@latest ; https://pkg.go.dev/search?q=yookassa
- https://github.com/theYahia/yookassa-mcp ; https://github.com/theYahia/yookassa-mcp/commits/main
- https://github.com/rvinnie/yookassa-sdk-go ; https://github.com/rvinnie/yookassa-sdk-go/commits/main
- https://github.com/Wolframko/yookassarb ; https://github.com/Wolframko/yookassarb/commits/main
- https://github.com/paderinandrey/yookassa (→ PaymentInstruments/yookassa)
- https://github.com/arthelokyo/astrowind ; https://github.com/arthelokyo/astrowind/commits/main
- https://github.com/puckeditor/puck ; https://github.com/GrapesJS/grapesjs ; https://github.com/webstudio-is/webstudio

**MCP-каталоги, новости, альтернативы (по сниппету поиска)**
- https://glama.ai/mcp/servers/lav2k15fia ; https://www.remoteopenclaw.com/mcp/Belolipetsky/YooKassa-MCP ; https://crossaitools.com/mcp/theyahia/tilda-mcp ; https://skillsmp.com/creators/gulivan/yookassa-skill/skill
- https://ichip.ru/novosti/ii-nauchili-vystavlyat-scheta-pryamo-v-chatah-s-podderzhkoj-972992
- https://robokassa.com/blog/news/robokassa-zapuskaet-platezhnuyu-integratsiyu-cherez-ii/
- https://www.sostav.ru/blogs/269097/80142 ; https://1000bankov.ru/rko/tarify/yoomoney/

**Конструкторы и билдеры (по сниппету поиска)**
- https://help.tilda.cc/api ; https://tilda.cc/en/answers/a/subscription-special-en/ ; https://tilda.cc/ru/answers/a/yandex-kassa/ ; https://rollout.com/integration-guides/tilda-publishing/api-essentials ; https://github.com/dotzero/tilda-api-python
- https://apimonster.ru/connector/service/creatium/ ; https://lpmotor.ru/articles/flexbe-i-mottor-ai-dlya-lendingov-avtovoronok-i-konversii-2605
- https://puckeditor.com/docs/api-reference/ai/cloud-client/generate ; https://puckeditor.com/docs/ai/getting-started ; https://puckeditor.com/blog/puck-ai-03
- https://devtoollab.com/ai-tools/mixo ; https://www.mixo.io/startup-website-builder

**Яндекс: Директ, Бизнес, Метрика, Cloud (по сниппету поиска)**
- https://yandex.ru/support/direct/en/moderation/ad-rules ; https://yandex.ru/support/direct/ru/moderation/categories/goods-and-services ; https://yandex.ru/support/direct/en/troubleshooting/moderation
- https://yandex.ru/support/direct/en/technologies-and-services/ad-labelingl ; https://ord.yandex.ru/ ; https://partner.market.yandex.ru/chtojournal/kak-markirovat-reklamu/
- https://yandex.ru/support/business-priority/ru/create-site.html ; https://business.yandex.ru/tools/website-builder/ ; https://direct.yandex.ru/constructor
- https://blog.click.ru/market-news/yandeks-prekratil-podderzhku-turbo-stranic/
- https://yandex.ru/support/metrica/ru/general/goal-js-event ; https://yandex.ru/blog/metrika-club/kak-pravilno-nastroit-reachgoal-v-formakh
- https://yandex.ru/dev/metrika/en/management/openapi/offline_conversions/upload_1 ; https://yandex.ru/support/metrica/ru/data/offline-params ; https://yandex.ru/support/metrica/en/reports/offline-conversions ; https://b2b.yandex.ru/adv/news/obnovlenie-v-rabote-s-oflajn-konversiyami-v-metrike
- https://apimonster.ru/connector/bundle/yookassa/yandexMetrika/
- https://yandex.cloud/en/docs/storage/operations/hosting/own-domain

**Право (по сниппету поиска)**
- https://www.consultant.ru/document/cons_doc_LAW_58968/f67f81c57fdcdacc2643d19d59369f7e185e1156/ (38-ФЗ ст. 5)
- https://www.consultant.ru/document/cons_doc_LAW_58968/b606ec51a053c12b78e7ab9a8c007fa723e6f321/ (38-ФЗ ст. 8)
- https://www.consultant.ru/document/cons_doc_LAW_305/cd5055f2729d0ba6bf1c3724ba16ca46ea92e0f7/ (ЗоЗПП ст. 23.1)
- https://www.consultant.ru/document/cons_doc_LAW_373622/e4ca7557c9ce6273af99d42c900e8579841fe652/ (ПП РФ № 2463) ; https://www.consultant.ru/law/podborki/statya_26.1._distancionnyj_sposob_prodazhi_tovara/
- https://buh.ru/news/khroniki-54-fz-kassovye-cheki-pri-prodazhakh-s-polnoy-predoplatoy-i-podgotovka-uproshchentsami-kkt-d.html
