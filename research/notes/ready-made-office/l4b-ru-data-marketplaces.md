# Слой 4b. Российские данные и маркетплейсы: ГИР БО (бухотчётность), Битрикс24 Маркет, amoCRM/Kommo, Kwork

Состояние на **2026-10-09**. Анализ только для чтения: ничего не устанавливалось, не запускалось, регистраций и токенов не было. Метрики GitHub и реестров сняты 2026-10-09: страницы github.com через WebFetch, реестры (rubygems.org, pypi.org, registry.npmjs.org, proxy.golang.org) через curl.

Ограничения среды повлияли на полноту. WebFetch не открыл huggingface.co, nature.com, arxiv.org, apidocs.bitrix24.com, developers.kommo.com, amocrm.com, glama.ai, mcpservers.org и kwork.ru (как и прочие .ru). Прокси отклонил curl к huggingface.co и mcpservers.org. К концу работы закончился общий лимит WebSearch, поэтому часть пробелов (раздел «Gaps») осталась открытой.

Пометки:
- «по сниппету поиска» — факт взят только из выдачи поиска (её сводки), саму страницу целиком не открывали;
- «не проверено» — первоисточник не подтверждён, в том числе всё, что известно мне только по памяти.

---

## Сводная таблица кандидатов (не больше 5)

Стадии конвейера «LLM-компании» в таблице: **скаутинг** идей → **фильтр стоп-факторов** (в т. ч. «монополист в нише») → **обогащение** данными → **сигналы спроса** → **канал продаж** → **найм**.

| # | Кандидат | URL | Лицензия / условия | Стек | Как подключить к Rails (и Go), агентам | Активность (на 2026-10-09) | Стадии | Работа в РФ | Хранение токенов | Вердикт |
|---|---|---|---|---|---|---|---|---|---|---|
| 1 | **RFSD**: открытая база бухотчётности всех российских юрлиц, 2011–2025 | [github.com/irlcode/RFSD](https://github.com/irlcode/RFSD); данные на HF `irlspbru/RFSD` и Zenodo DOI 10.5281/zenodo.14622208 | CC BY 4.0, нужна атрибуция и ссылка на статью в Scientific Data | Parquet по годам (hive-партиции `year=`), код на R и Python | Rails: gem `duckdb` ([1.5.5.1, MIT, 2026-08-29](https://rubygems.org/gems/duckdb)) читает Parquet, агрегаты по ОКВЭД пишутся в Postgres. Go: [`duckdb/duckdb-go/v2` v2.10506.0 (2026-09-30)](https://proxy.golang.org/github.com/duckdb/duckdb-go/v2/@latest). Агентам — свой MCP-инструмент поверх витрины | v3.1.0 от 2026-09-02, последний коммит 2026-09-02; 80★, 14 форков; три названных автора; v1.0.1 от 2025-05-13. Аномалий нет | Фильтр «монополист», обогащение (размер ниши), скаутинг (рост отраслей) | Данные ФНС и Росстата. Доступ к HF и Zenodo из РФ не проверен; базу можно зеркалить у себя (CC BY) | Не нужны (публичные данные) | **Брать**: основной бесплатный источник выручки и концентрации по ОКВЭД. Лаг годовой |
| 2 | **Checko API**: ЕГРЮЛ, бухотчётность ГИР БО по ИНН, поиск по ОКВЭД | [checko.ru/integration/api](https://checko.ru/integration/api) | Коммерческий, [публичная оферта](https://checko.ru/offer). 100 запросов в сутки бесплатно, далее 0,15 ₽ или 0,10 ₽ за запрос (по сниппету поиска) | REST/JSON (`api.checko.ru/v2/...`) | Rails: тонкий клиент на Faraday или Net::HTTP. Go: net/http. Агентам — свой MCP-враппер (готового MCP не нашли) | API обновлён до v2.5 24.09.2026 (по сниппету поиска) | Обогащение (свежая отчётность, суды, госзакупки), фильтр «монополист» (проверка топ-игроков) | Российский сервис, оплата картой или со счёта. Доступ из-за рубежа не проверен | API-ключ в `Rails.application.credentials` или ENV; в Go — ENV или секрет-хранилище; суточный лимит трат в кабинете | **Брать** (платно, но дёшево) для точечной свежей проверки. Для массовой выборки дорого и медленно |
| 3 | **Официальный dev-стек Битрикс24**: `b24gosdk` (Go SDK), `mcp-rest-doc` (MCP по документации), `templates-mcp` (шаблон MCP-сервера) | [b24gosdk](https://github.com/bitrix24/b24gosdk), [mcp-rest-doc](https://github.com/bitrix24/mcp-rest-doc), [templates-mcp](https://github.com/bitrix24/templates-mcp) | SDK и шаблон под MIT. MCP по документации — хостинг без авторизации | Go 1.21+ без зависимостей; шаблон на TypeScript (Nuxt 4 + `@bitrix24/b24jssdk`) | Go — SDK напрямую. Rails: официального Ruby SDK нет, писать тонкий REST-клиент (webhook или OAuth) либо вынести в Go-сервис. Агентам-кодерам — `https://mcp-dev.bitrix24.com/mcp` (HTTP) | b24gosdk: v0.1.0 от 2026-08-06, v0.2.0 от 2026-08-07, 68 коммитов, 4★, 0 форков. **Аномалия:** проекту 2 месяца, коммиты сжаты в 3 дня, соавтор коммитов — «claude». templates-mcp: v0.3.0 от 2026-06-16, последний коммит в main 2026-07-03, 138 коммитов, 6★, 6 форков, 79 открытых issues (**аномалия**, см. deep-dive). mcp-rest-doc: 1 коммит, 7★, обновлён 2025-12-12 | Канал продаж (B2B-приложения в Маркете), разработка (документация для агентов-кодеров); скаутинг — нет (API каталога не нашли) | Вендор российский. Доступ к mcp-dev.bitrix24.com из РФ не проверен | Webhook URL — это секрет. OAuth-токены порталов в БД; b24gosdk даёт callback для сохранения обновлённой пары токенов. Шаблон: keychain / `.env` / секрет-хранилище | **Брать**: MCP по документации — сразу, b24gosdk — если бэкенд на Go. templates-mcp — **референс** |
| 4 | **amoCRM/Kommo**: gem `amocrm` (Hexlet) и комьюнити-MCP `cAIborg-ai/amocrm-mcp` | [Hexlet/amocrm-ruby](https://github.com/Hexlet/amocrm-ruby), [cAIborg-ai/amocrm-mcp](https://github.com/cAIborg-ai/amocrm-mcp) | MIT на GitHub; у gem поле licenses в RubyGems пустое | Ruby 3.2+, код сгенерирован Stainless; MCP на Python (FastMCP) | Rails — gem напрямую (`AMOCRM_AUTH_TOKEN` и subdomain). Go — доминирующей библиотеки нет, тонкий клиент под API v4 (7 запросов/с). Агентам — свой узкий MCP | gem 0.6.2 от 2026-10-06, 14 версий с 2026-02-02, 4★, 79 коммитов. MCP: 1 коммит, 2★, 3 форка (**аномалия**: выложен одним коммитом) | Канал продаж (виджеты в маркетплейсе), операционная CRM; скаутинг — данных каталога не нашли | amoCRM работает в РФ, Kommo — международный бренд | Долгоживущий токен или OAuth refresh. MCP сохраняет обновлённые токены на диск, путь не указан (риск) | **Референс**. Gem брать, только если делаем интеграцию с amoCRM из Rails |
| 5 | **Kwork, неофициальная обёртка** `kesha1225/kwork` (бывший `pykwork`) | [github.com/kesha1225/kwork](https://github.com/kesha1225/kwork), PyPI [`kwork`](https://pypi.org/project/kwork/) | MIT, но работает через закрытый API (мобильный `api.kwork.ru` и веб-флоу). Риск нарушить условия Kwork | Python, async (aiohttp, pydantic) | Нативно для Rails и Go — нет. Вариант — Python-сайдкар или порт 2–3 нужных вызовов (`get_projects`, `get_categories`) | PyPI 0.0.1 (2020-02-03) → 0.2.0 (2026-02-10), пауза с 2021-08 по 2025-12; последний коммит 2026-04-14 (зависимости); 70★, 11 форков, 60 коммитов. **Аномалия:** репозиторий переименован, самый ранний коммит на странице — 2022-06-22, хотя первый релиз на PyPI вышел в 2020 | Сигналы спроса (биржа: бюджеты, число откликов, % найма), найм (отклики) | Русскоязычная биржа [kwork.ru](https://kwork.ru/projects), оператор по оферте — RemoteFirst Group Limited (Гонконг; по сниппету поиска). Есть антибот-капча ([guide.md](https://raw.githubusercontent.com/kesha1225/kwork/master/docs/guide.md)) | Логин и пароль аккаунта Kwork — высокий риск | **Референс** (поля и эндпоинты). В прод не брать |

Рассмотрены, но в топ-5 не вошли (детали в разделах ниже):
- прямые скрытые эндпоинты bo.nalog.ru и открытые данные ФНС `revexp` — референс или резерв;
- DaData и её MCP — финансы только на тарифе «Максимальный»;
- Контур.Фокус API — дорого;
- СПАРК и ofdata — нет данных;
- `mcp-egrul` — только ЕГРЮЛ;
- `theYahia/dadata-mcp` и `theYahia/amocrm-mcp`;
- комьюнити-MCP для Битрикс24;
- устаревшие Ruby-гемы Битрикс24 и amoCRM.

---

## В1. ГИР БО: официальный API, скрытые эндпоинты, открытые данные, RFSD, сторонние API, библиотеки и MCP

### Takeaway
Официального публичного API у bo.nalog.ru нет: есть веб-сервис и «абонентское обслуживание» по административному регламенту. Сайт ходит в собственное скрытое JSON-API, но оно не документировано, и мы его не проверили. Для оценки выручки ниши и концентрации лучше всего подходит открытая база **RFSD** (CC BY 4.0, все юрлица за 2011–2025, есть колонки `okved` и `okved_section`, обновление раз в год). Свежие точечные данные дёшево даёт **Checko API**. Готовых MCP-серверов именно для ГИР БО или Checko не нашли, есть только MCP для ЕГРЮЛ и DaData.

### Cited Findings

**Официальный доступ к ГИР БО**
- В выдаче поиска не нашлось официальной документации открытого API bo.nalog.ru. Госорганы и организации получают сведения ГИР БО через интернет-сервис на bo.nalog.ru (по сниппету поиска) — [data.nalog.ru/…/GIRBO.pdf](https://data.nalog.ru/html/sites/www.rn69.nalog.ru/Info/GIRBO.pdf).
- Административный регламент ФНС описывает услугу «предоставление информации из ГИР БО в форме абонентского обслуживания» (по сниппету поиска). Плата и формат не выяснены — [consultant.ru, адм. регламент](https://www.consultant.ru/document/cons_doc_LAW_347267/15655fdbaad6f585e347b871961f6ac8c1cd9559/).
- Сейчас сайт работает на домене **bo.nalog.gov.ru**. Публичные скрипты обращаются к `https://bo.nalog.gov.ru/search?query=<ИНН>` и скачивают архивы отчётности через UI, через Selenium-подобный браузерный сценарий, а не через API — [Vadenoire-ONE/bo.nalog.ru](https://github.com/Vadenoire-ONE/bo.nalog.ru) (1★, Unlicense, 9 коммитов).

**Скрытые (недокументированные) эндпоинты**
- Статья на Infostart (декабрь 2025) описывает, как получать данные bo.nalog.ru по ИНН через скрытый API сайта: эндпоинты находят во вкладке «Сеть» DevTools, затем берут строку выручки 2110 (по сниппету поиска) — [infostart.ru/public/2557058](https://infostart.ru/public/2557058/).
- Поиск GitHub по «bo.nalog.ru» нашёл 6 мелких репозиториев (0–1★). Все — разовые парсеры: `write2art/bo-parser`, `Vadenoire-ONE/bo.nalog.ru`, `Proxor104/ConstructAnalysis`, `surikatio/OKVED_parser` и др. Поддерживаемой библиотеки нет — [поиск GitHub](https://github.com/search?q=bo.nalog.ru&type=repositories).
- `surikatio/OKVED_parser` выгружает компании по 25 кодам ОКВЭД (26.20, 62.01–62.09, 63.11–63.12, 74.90) с полями ИНН, ОГРН, ОКВЭД и выручка «из bo.nalog.gov.ru». ФИО руководителей он дообогащает с datanewton.ru без токенов. Конкретные URL эндпоинтов в README не приведены — [github.com/surikatio/OKVED_parser](https://github.com/surikatio/OKVED_parser).
- По памяти сайт использует пути вида `/advanced-search/organizations/search?query=…`, `/nbo/organizations/{id}/bfo/` и `/nbo/bfo/{id}/details`, а архивы отдаёт через `/download/bfo/{id}` — **не проверено** (ни в одном первоисточнике в этой сессии не подтвердилось).
- Лимиты, капча и гео-ограничения bo.nalog.gov.ru (доступ с зарубежных IP) — **не проверено**: поиск по этой теме не выполнен, лимит WebSearch исчерпан. Скрипт Vadenoire-ONE упоминает лишь «проверку на капчу и блокировки» и экспоненциальный backoff, без деталей — [README](https://github.com/Vadenoire-ONE/bo.nalog.ru).

**Открытые данные ФНС (массовые выгрузки)**
- Набор **7707329152-revexp** — «Сведения о суммах доходов и расходов по данным бухгалтерской (финансовой) отчетности организации за год, предшествующий году размещения». Карточка показывает данные за 2025 год, «Дата актуальности» на разных зеркалах — 25.09.2026 или 25.08.2026, методические рекомендации версии 4.0 (по сниппету поиска) — [nalog.gov.ru/rn07/opendata/7707329152-revexp](https://www.nalog.gov.ru/rn07/opendata/7707329152-revexp/), [rn23](https://www.nalog.gov.ru/rn23/opendata/7707329152-revexp/).
- Правовая основа — пп. 11 п. 1 ст. 102 НК РФ. Данные за 2022 год опубликованы 1 мая 2023 года. ФНС сообщила, что далее сведения будут обновляться 25-го числа каждого месяца до декабря (по сниппету поиска) — [buh.ru](https://buh.ru/news/fns-raskryla-svedeniya-o-dokhodakh-kompaniy-v-2022-godu-i-o-nalogoplatelshchikakh-na-spetsrezhimakh.html), [spmag.ru](https://spmag.ru/news/fns-razmestila-otkrytye-dannye-po-uplate-nalogov).
- Состав полей revexp — по памяти ИНН, наименование, доходы, расходы, **ОКВЭД нет** — **не проверено**.

**RFSD (Russian Financial Statements Database)**
- Описание из README: «открытая гармонизированная коллекция годовой неконсолидированной бухотчётности» российских фирм за **2011–2025**. Лицензия **CC BY 4.0**, авторы Bondarkov, Ledenev, Skougarevskiy. Данные лежат на Hugging Face (`irlspbru/RFSD`) и Zenodo (DOI 10.5281/zenodo.14622208) — [README](https://github.com/irlcode/RFSD).
- Версии: **3.1.0 от 2026-09-02** (исправлена ошибка согласованности строк 2025 г.); 3.0.0 от 2026-08-20 (добавлен 2025 г., около 2,17 млн наблюдений); 2.0.0/2.0.1 от 2025-08-19 (2024 г., около 2,25 млн); 1.0.1 от 2025-05-13 — [README](https://raw.githubusercontent.com/irlcode/RFSD/main/README.md).
- Обновление **раз в год**: данные запрашиваются в начале июня, релиз выходит в конце июля — начале августа. Данные за 2026 год ожидаются к июлю 2027 — [README](https://github.com/irlcode/RFSD).
- Формат — Apache Parquet с партициями по годам. Вся база занимает 7,7 ГБ и больше, срез за 2025 год — около 534 МБ, для загрузки в Polars нужно примерно 8 ГБ RAM. Прямое чтение из HF через Polars на август 2026 «временно сломано» — [README](https://github.com/irlcode/RFSD).
- Выручка — `line_2110`, в словаре `aux_data/descriptive_names_dict.csv` она переименована в `B_revenue`. Префиксы: `B_` — баланс, `PL_` — ОФР, `CFi_`/`CFo_` — ДДС — [дерево репозитория](https://github.com/irlcode/RFSD/tree/main).
- Нефинансовые поля, названные в README: `inn`, `region`, `region_taxcode`, `okopf`, `okfc`, `okpo`, `eligible`, `filed`, `outlier`, `exemption_criteria`, `geocoding_quality` — [README](https://raw.githubusercontent.com/irlcode/RFSD/main/README.md).
- **Колонки отрасли есть.** Пример использования TFP отбирает колонки `"inn", "ogrn", "year", "okved_section", "okved", …`, фильтрует `okved_section == "C"` и берёт двузначную отрасль через `substr(okved, 1, 2)` — [use_cases/tfp.md](https://raw.githubusercontent.com/irlcode/RFSD/main/use_cases/tfp.md).
- Источники данных — Росстат и ФНС (bo.nalog.ru/ГИР БО), панели строятся по ЕГРЮЛ и Статрегистру Росстата — [README](https://github.com/irlcode/RFSD). Версия за 2011–2023 содержала 56,6 млн геокодированных наблюдений «фирма-год» (по сниппету поиска) — [arXiv 2501.05841](https://arxiv.org/html/2501.05841v1), [Zenodo](https://zenodo.org/records/14622209).
- Ограничения, которые README называет сам:
  - отчётность **только неконсолидированная**, о группах компаний выводов делать нельзя;
  - исключены финансовые, религиозные и государственные организации, которые не обязаны сдавать отчётность;
  - есть неподававшие фирмы, часть отчётности импутирована по более поздним (пример — Газпром за 2022 год);
  - поздние корректировки попадают только в следующие релизы;
  - выбросы помечены флагом `outlier`, но не удалены;
  - изменения форм в 2025 году не гармонизированы.
  
  Источник — [README](https://raw.githubusercontent.com/irlcode/RFSD/main/README.md).
- Статья — Scientific Data, 2025, DOI 10.1038/s41597-025-05150-1. О базе писали «Коммерсант» и РБК (январь 2025) и Business FM Санкт-Петербург (февраль 2025) — [README](https://github.com/irlcode/RFSD). Новость ЕУСПб об Институте проблем правоприменения как авторе — [eusp.org](https://eusp.org/en/news/researchers-from-the-institute-for-the-rule-of-law-have-published-a-new-article-on-the-russian-accounting-reporting-database).

**Сторонние API с финансами**
- **Checko API.** Тарифы (по сниппету поиска):
  - «Лайт» — бесплатно, 100 запросов в сутки;
  - «Стандарт» — 0,15 ₽ за запрос после пополнения от 1 000 ₽;
  - «Максимум» — 0,10 ₽ за запрос после пополнения от 50 000 ₽.
  
  Цена одинакова для всех методов, первые 100 запросов в сутки бесплатны на любом тарифе. Ответы 4xx/5xx не оплачиваются, баланс не сгорает, суточный лимит трат настраивается, НДС 5% включён — [checko.ru/integration/api](https://checko.ru/integration/api).
- Checko `search` (по сниппету поиска): `https://api.checko.ru/v2/search?by=okved&obj=org&query=63.11&opf=12300&active=true`, фильтры `region`, `active`, `opf`.
  - Не больше 100 записей на страницу, пагинация через `page`. Результаты по ОКВЭД сортируются **по ОГРН**.
  - Поиск идёт по **основному** ОКВЭД.
  - В ответе: ОГРН, ИНН, КПП, наименование, дата регистрации, статус, адрес, основной ОКВЭД. В `meta` — число запросов за день и баланс.
  - API обновлён до v2.5 24.09.2026.
  
  Источник — [checko.ru/integration/api/search](https://checko.ru/integration/api/search).
- Отдельные методы Checko: `finances` (бухотчётность ФНС по ИНН), `legal-cases`, `contracts`, `person`, `enforcements`, `bankruptcy-messages`, `company`, `entrepreneur` (по сниппету поиска, по заголовкам страниц) — [finances](https://checko.ru/integration/api/finances), [legal-cases](https://checko.ru/integration/api/legal-cases), [contracts](https://checko.ru/integration/api/contracts).
- Сторонний обзор:
  - веб-тариф Checko — 3 500 ₽ в год плюс 100 бесплатных запросов в сутки;
  - конкурент критикует «финансовую отчётность с лагом 12–18 месяцев из ГИРБО» — это мнение, а не факт.
  
  По сниппету поиска — [toolfox.ru](https://toolfox.ru/blog/chekko-obzor-tarify-otzyvy).
- **DaData.**
  - Бесплатный тариф — до 10 000 запросов в день (по сниппету поиска) — [toolfox.ru/services/s/dadata](https://toolfox.ru/services/s/dadata).
  - По README стороннего MCP: в `find_company_by_id` базовые данные бесплатны, а **поля финансов и все коды ОКВЭД требуют тарифа «Максимальный»**. У DaData есть официальный MCP-сервер с 4 инструментами по адресу dadata.ru/mcp. Это утверждение третьей стороны, на dadata.ru **не проверено** — [theYahia/dadata-mcp](https://github.com/theYahia/dadata-mcp). Цена «Максимального» не найдена.
- **Контур.Фокус API** стоит «от 18 000 ₽/мес» (по сниппету поиска, сторонний обзор) — [toolfox.ru](https://toolfox.ru/services/dannye-i-api/obogashchenie-dannyh-kompanii).
- **Saby (СБИС)**: тариф с API отдаёт выписку ЕГРЮЛ/ЕГРИП или бухотчётность ГИР БО (Росстата) в XML (по сниппету поиска) — [saby.ru/help/partner/api/abstract](https://saby.ru/help/partner/api/abstract).
- Сведения по **СПАРК-Интерфакс** и **ofdata.ru** не найдены (см. Gaps).

**Библиотеки и MCP-серверы (ЕГРЮЛ, ФНС, DaData, Checko)**
- `mcp-egrul` (atomno-labs): PyPI 0.1.1 от 2026-04-26, MIT, «EGRUL/EGRIP lookups via ФНС open-data» — [PyPI](https://pypi.org/project/mcp-egrul/). По листингу (по сниппету поиска): 7 инструментов; режим Pro скрапит egrul.nalog.ru и откатывается на DaData; цена $10/мес; бесплатно 30 запросов в день на IP — [mcpservers.org](https://mcpservers.org/de/servers/atomno-labs/mcp-egrul). Финансов нет.
- `theYahia/dadata-mcp`: npm `@theyahia/dadata-mcp` 1.0.7 и `@metarebalance/dadata-mcp` 1.0.6, оба созданы в марте–апреле 2026; 31 инструмент; 3★, 22 коммита; MIT — [GitHub](https://github.com/theYahia/dadata-mcp), [npm](https://registry.npmjs.org/@theyahia/dadata-mcp).
- MCP для Checko и для ГИР БО **не найдены** (поиск по выдаче и реестрам).
- Официальные клиенты DaData — `hflabs/dadata-py` (103★; PyPI `dadata` 25.10.0 от 2025-10-07) и `dadata-csharp`. Официального Ruby- и Go-клиента на странице организации не видно — [github.com/hflabs](https://github.com/hflabs), [PyPI](https://pypi.org/project/dadata/).
- Go-клиент DaData от комьюнити — `ekomobile/dadata/v2` v2.18.0 от 2026-04-20, 23 версии, 29★, 24 форка, MIT; реализует методы Clean и Suggest — [GitHub](https://github.com/ekomobile/dadata), [proxy.golang.org](https://proxy.golang.org/github.com/ekomobile/dadata/v2/@latest).
- Ruby-гемы DaData устарели: `dadata` 0.3.1 (2021-03-30), `dadatas` 0.1.7 (2022-09-28) — [rubygems dadata](https://rubygems.org/gems/dadata), [rubygems dadatas](https://rubygems.org/gems/dadatas).
- Поиск RubyGems по `egrul`, `nalog` и `checko` релевантных гемов не дал: по `checko` нашлись только посторонние `checkout*` — [rubygems.org](https://rubygems.org/). На PyPI пакетов с точными именами `girbo`, `bonalog`, `nalog-bo`, `fns-api`, `egrul`, `rfsd` и `checko` нет (прямой запрос к JSON API) — [pypi.org](https://pypi.org/). Полнотекстовый поиск по PyPI не выполнялся.

### Inferences
- Для стоп-фактора «монополист» основа — RFSD. По последнему году и коду ОКВЭД (4–6 знаков) считаются число фирм, суммарная выручка, CR1/CR3 и HHI. Это бесплатно, офлайн и не нарушает условий ФНС.
- RFSD систематически **недооценивает концентрацию**. Причины: отчётность неконсолидированная (холдинг дробится на несколько юрлиц); ИП бухотчётность не сдают и в базу не попадают; основной ОКВЭД часто не совпадает с реальным бизнесом; маркетплейсы и иностранные игроки не видны. Поэтому топ-5 по RFSD стоит проверить в Checko: аффилированность через учредителей, свежая отчётность.
- Checko не годится для массового расчёта концентрации. Поиск по ОКВЭД возвращает компании без выручки и в порядке ОГРН, поэтому на каждую компанию нужен отдельный вызов `finances` (N+1 запросов). Пример: ниша на 5 000 юрлиц обойдётся примерно в 5 000 вызовов, около 750 ₽ по «Стандарту». Точечная проверка топ-20 игроков по 50 идеям — около 1 050 вызовов в неделю. Из них около 700 закрывает бесплатная квота (100 в сутки × 7), на остальные уйдёт порядка 50 ₽ в неделю.
- Набор revexp дублирует RFSD по свежести: данные за 2025 год есть и там, и там к августу–сентябрю 2026. ОКВЭД в revexp, по-видимому, нет. Нужен только как резерв, если RFSD перестанут обновлять.
- Собственный клиент к скрытому API bo.nalog.gov.ru — резервный вариант. API недокументирован, формально не разрешён, устойчивость, капча и гео-доступ неизвестны.

### Gaps
- Точные пути и JSON-схема скрытого API bo.nalog.gov.ru, его лимиты и капча, доступ с зарубежных IP — не проверены. WebFetch не открывает .ru, лимит WebSearch исчерпан.
- Условия использования bo.nalog.gov.ru (запрет автоматизированного сбора или его отсутствие) и стоимость «абонентского обслуживания» не найдены.
- Структура полей revexp (есть ли ОКВЭД) — не проверена.
- Цены DaData «Максимальный», СПАРК-Интерфакс API и ofdata.ru API, а также наличие у них выборки «выручка по ОКВЭД» — не найдены.
- Карточку RFSD на Hugging Face (число скачиваний, точный список колонок, единицы измерения — вероятно, тыс. руб.) открыть не удалось: huggingface.co недоступен из среды. Доступность HF и Zenodo из РФ не проверена.
- Официальный DaData MCP по адресу dadata.ru/mcp известен только со слов стороннего README.

---

## В2. Битрикс24: REST API, MCP, Маркет (публикация, доля выручки, данные каталога), клиенты Ruby/Go

### Takeaway
У Битрикс24 есть официальные SDK, включая **Go SDK `b24gosdk`** (август 2026), официальный **MCP-сервер по документации REST** (`mcp-dev.bitrix24.com/mcp`, без авторизации) и официальный **шаблон MCP-сервера для порталов** (задачи, CRM в планах). Готового официального MCP для данных портала нет: в Маркете уже не меньше пяти сторонних MCP-коннекторов. В российском Маркете действует подписочная модель: разработчикам распределяется **85%** выручки пула пропорционально использованию. Документированного API к каталогу Маркета (установки, рейтинги) нет.

### Cited Findings

**REST API и лимиты**
- Лимит запросов работает по схеме «дырявого ведра». На тарифе Enterprise счётчик убывает на 5 единиц в секунду при пороге 250, на остальных тарифах — на 2 в секунду при пороге 50. При превышении возвращается 503 `QUERY_LIMIT_EXCEEDED`. Лимит считается на портал с учётом IP: приложения на одном сервере делят лимит. В batch — не больше 50 подзапросов. Один REST-запрос в облаке ограничен 60 секундами (по сниппету поиска) — [apidocs.bitrix24.ru/limits.html](https://apidocs.bitrix24.ru/limits.html), [apidocs.bitrix24.com/limits.html](https://apidocs.bitrix24.com/limits.html).
- Если суммарное время запросов к одному методу превысит 480 секунд за 10 минут, метод блокируется на 10 минут с ошибкой 429 `OPERATION_TIME_LIMIT`. REST «не предназначен для массового обмена», для больших выгрузок есть BI-коннектор (по сниппету поиска) — [helpdesk.bitrix24.ru/open/15959788](https://helpdesk.bitrix24.ru/open/15959788/), [apidocs.bitrix24.com/limits.html](https://apidocs.bitrix24.com/limits.html).

**MCP**
- Официальный `bitrix24/mcp-rest-doc` — «MCP-сервер, дающий ИИ-ассистентам актуальную документацию REST API».
  - Эндпоинт `https://mcp-dev.bitrix24.com/mcp` (тип `http`), «No API key or local setup is required».
  - Инструменты: `bitrix-search`, `bitrix-method-details`, `bitrix-article-details`, `bitrix-event-details`.
  - Даёт только документацию, данных портала нет. 7★, 4 форка, 1 коммит.
  
  Источники — [GitHub](https://github.com/bitrix24/mcp-rest-doc); дата обновления 2025-12-12 по [поиску GitHub в org:bitrix24](https://github.com/search?q=org%3Abitrix24+mcp&type=repositories). Страница официальной документации «MCP-сервер для работы с REST API Битрикс24» (по сниппету поиска, по заголовку) — [apidocs.bitrix24.ru/sdk/mcp.html](https://apidocs.bitrix24.ru/sdk/mcp.html).
- Официальный `bitrix24/templates-mcp` описан как «Reference template for building production-grade MCP servers for Bitrix24».
  - Стек: Nuxt 4 и `@bitrix24/b24jssdk`.
  - 30 инструментов: 27 по задачам, 2 по пользователям и мета-инструмент `bx24mcp_submit_feedback`, который заводит issues на GitHub. CRM — в планах.
  - Авторизация по умолчанию через входящий webhook; мультиарендный OAuth включается флагом `NUXT_BITRIX24_OAUTH_ENABLED`.
  - Транспорты: локальный HTTP, удалённый HTTP в Docker, stdio через `.dxt`.
  - Секреты: OS keychain (DXT), `.env` или секрет-хранилище хоста. Логгер вырезает секреты webhook.
  - MIT, 6★, 6 форков, 138 коммитов, 79 открытых issues.
  
  Источник — [GitHub](https://github.com/bitrix24/templates-mcp). Релиз v0.3.0 от 2026-06-16, последний коммит в main — 2026-07-03 ([коммиты](https://github.com/bitrix24/templates-mcp/commits/main)). Поиск GitHub при этом показывал «updated 18 days ago» — расхождение, вероятно, из-за других веток.
- В выдаче Маркета нашлось не меньше пяти сторонних MCP-приложений: `evrika.mcp`, `zeliboba.mcp`, `kolibbri.mcp`, `intogroup.mcp`, `intelmediasoft.mcp24`. Типичный сценарий: установить приложение, скопировать URL MCP и вставить в клиент; бывают режимы «только чтение» (по сниппету поиска) — [evrika.mcp](https://www.bitrix24.ru/apps/app/evrika.mcp/), [zeliboba.mcp](https://www.bitrix24.ru/apps/app/zeliboba.mcp/), [kolibbri.mcp](https://www.bitrix24.ru/apps/app/kolibbri.mcp/), [intogroup.mcp](https://www.bitrix24.ru/apps/app/intogroup.mcp/), [intelmediasoft.mcp24](https://www.bitrix24.ru/apps/app/intelmediasoft.mcp24/).
- Комьюнити-проекты: PyPI `bitrix24-mcp` 1.0.1 — единственный релиз от 2025-04-20 ([PyPI](https://pypi.org/project/bitrix24-mcp/)); партнёры Битрикс24 открыли свой MCP-сервер (по сниппету поиска) — [habr.com/ru/articles/903190](https://habr.com/ru/articles/903190).

**SDK и клиенты**
- В организации `bitrix24` на GitHub 24 репозитория:
  - `b24phpsdk` — 102★, обновлён 2026-09-30;
  - `b24jssdk` — 18★, 2026-10-05;
  - `b24pysdk` — 17★, 2026-09-25;
  - `b24restdocs` — исходники документации, 36★;
  - `b24ui` — 44★.
  
  Источник — [github.com/bitrix24](https://github.com/bitrix24).
- **`b24gosdk`** (MIT) описан как «A Go SDK for the Bitrix24 REST API».
  - Go 1.21+, без внешних зависимостей.
  - Авторизация: webhooks, OAuth и поток in-portal app (события install/uninstall, проверка токена приложения).
  - Автообновление при `expired_token` с callback для сохранения новой пары токенов.
  - Batch до 50 команд, `CallBatchChunked`, автоповтор при `QUERY_LIMIT_EXCEEDED`, пагинация `Pages`/`Scan`, частичная поддержка REST 3.0, фейковый портал `b24test` для тестов.
  - **Типизированных CRM-сервисов нет**: методы вызываются по имени.
  
  Источник — [GitHub](https://github.com/bitrix24/b24gosdk). Версии v0.1.0 (2026-08-06T08:34Z) и v0.2.0 (2026-08-07T09:25Z) — [proxy.golang.org](https://proxy.golang.org/github.com/bitrix24/b24gosdk/@v/list). На странице коммитов видны соавторы «ExaltedTrou6 and claude» — [коммиты](https://github.com/bitrix24/b24gosdk/commits/main). SDK входит в официальную документацию, папка `sdk/b24gosdk` в `b24restdocs` — [b24restdocs/sdk](https://github.com/bitrix24/b24restdocs/tree/main/sdk).
- Go от комьюнити: 12 модулей на pkg.go.dev, почти без использования (imported by 0–7); среди них `whatcrm/go-bitrix24`, `NickTaporuk/bitrix24`, `murlabrion/bitrix24-go` — [pkg.go.dev](https://pkg.go.dev/search?q=bitrix24&m=package).
- Ruby, только комьюнити и устаревшее:
  - `bitrix24_cloud_api` 0.1.3 (2024-03-03, MIT, около 14 тыс. загрузок);
  - `bitrix_webhook` 0.2.10 (2019-07-24);
  - `omniauth-bitrix24` 0.1.0.
  
  Источники — [bitrix24_cloud_api](https://rubygems.org/gems/bitrix24_cloud_api), [bitrix_webhook](https://rubygems.org/gems/bitrix_webhook), [omniauth-bitrix24](https://rubygems.org/gems/omniauth-bitrix24).

**Маркет: публикация и монетизация**
- Российская документация (по сниппету поиска) — [subscription-details](https://apidocs.bitrix24.ru/market/monetization/subscription-details.html):
  - «85% от распределяемой выручки получают разработчики решений… которые клиенты установили и использовали в рамках подписки»;
  - половина вознаграждения сразу уходит приложениям, которые привели к продаже подписки, вторая половина распределяется в течение срока подписки по ежедневным «баллам использования»;
  - в кабинете есть выгрузка ежедневных выплат.
- Публикация в РФ (по сниппету поиска) — [publication-requirements](https://apidocs.bitrix24.ru/market/preparing-to-publish/publication-requirements.html):
  - платные приложения размещают только ИП и юрлица;
  - после проверки контрагента соглашение приходит в ЭДО за 1–2 рабочих дня;
  - для бесплатного размещения достаточно оферты;
  - в РФ платные решения работают через подписку «Битрикс24.Маркет плюс».
  
  Найдено **противоречие**: «в российском Маркетплейсе можно публиковать только по Подписке (бесплатно нельзя)» против «бесплатные решения для России — только в Маркетплейсе БУС».
- Внешние платежи за доп. функциональность разработчик может принимать сам, «в настоящий момент 1С-Битрикс не взимает комиссию» (по сниппету поиска) — [in-app-purchases](https://apidocs.bitrix24.ru/market/monetization/in-app-purchases.html). Английские исходники это подтверждают: «Currently, Bitrix24 does not charge a fee for such payments». Международный Маркет поддерживает два типа размещения: бесплатно и бесплатно с in-app purchases — [sales-terms.md](https://raw.githubusercontent.com/bitrix24/b24restdocs/main/market/sales-terms.md).
- Модерация: сроки «не регламентированы»; решение «may be denied publication… without explanation»; кабинет разработчика — vendors.bitrix24.com — [publication-requirements.md](https://raw.githubusercontent.com/bitrix24/b24restdocs/main/market/preparing-to-publish/publication-requirements.md).
- По справке Битрикс24, единая подписка «Битрикс24 Маркетплейс» заменяет бесплатный интеграционный пакет и «Маркет Плюс» (по сниппету поиска, может быть устаревшим) — [helpdesk.bitrix24.ru/open/23776074](https://helpdesk.bitrix24.ru/open/23776074/). Продажи по подписке идут через каталог и партнёрскую сеть (по сниппету поиска) — [bitrix24.ru/apps/dev.php](https://www.bitrix24.ru/apps/dev.php).

**Данные каталога для скаутинга**
- Отзывы оставляют в карточке приложения на витрине внутри портала, отзыв могут оставить только аккаунты на коммерческом тарифе. Отзывы публикуются автоматически через 3 дня. Рейтинг влияет на позицию на витрине. **API для чтения рейтингов, отзывов и числа установок не документирован**, о числе установок в статье ничего нет — [users-rating.md](https://raw.githubusercontent.com/bitrix24/b24restdocs/main/market/promoting-and-analytics/users-rating.md).
- Публичный каталог существует: коллекции вроде «Оплаты и платежи», страницы партнёров, карточки приложений (по сниппету поиска) — [bitrix24.ru/apps](https://www.bitrix24.ru/apps/), [коллекция](https://www.bitrix24.ru/apps/collection/16420306/), [страница партнёра](https://www.bitrix24.ru/apps/partner/336901/).

### Inferences
- Как **канал продаж** Битрикс24 подходит для соло-фаундера. Подписочный пул отдаёт 85% разработчикам, внешние платежи пока без комиссии, то есть можно брать деньги напрямую за собственный SaaS-бэкенд. Но выручка из пула зависит от «баллов использования» и непрозрачна для прогноза, а модерация нерегламентирована.
- Как **источник идей** Маркет пригоден только через публичные веб-страницы каталога: категории, коллекции, отзывы, рейтинги. Это ручной разбор или аккуратный скрапинг после проверки условий. Числа установок в документации нет.
- Ниша «MCP-коннектор для Битрикс24» уже насыщена (не меньше пяти приложений), и официальный шаблон Битрикс24 снижает порог входа ещё сильнее. Для стоп-фактора это признак высокой конкуренции, а не монополии.
- 79 открытых issues при 6★ у templates-mcp, скорее всего, заводят агенты через встроенный инструмент обратной связи, а не люди. Это не означает высокой популярности.
- Для Rails официального SDK нет. REST-клиент под webhook и OAuth — несколько сотен строк, бизнес-логику лучше брать из b24gosdk как образец (retry, batch, refresh).

### Gaps
- Не проверено, показываются ли число установок и рейтинг публично на bitrix24.ru/apps и разрешают ли правила Маркета автоматический сбор каталога.
- Не установлено, выпустил ли Битрикс24 официальный MCP для данных порталов (CRM). Пока найдены только шаблон и сторонние приложения Маркета.
- Противоречие о бесплатных решениях в российском Маркете и точные условия оферты (rules.php) не разрешены.
- Доступность `mcp-dev.bitrix24.com` из РФ не проверена.

---

## В3. amoCRM / Kommo: API v4, MCP, маркетплейс, клиенты Ruby/Go

### Takeaway
Официального MCP-сервера у amoCRM/Kommo не нашли, есть несколько небольших комьюнити-серверов. API v4 ограничен **7 запросами в секунду** (по IP); при частых 429 аккаунт или IP блокируются с ответом 403. Публикация в маркетплейсе идёт через «публичную интеграцию», технический аккаунт и модерацию за 1–2 рабочих дня. Комиссия маркетплейса и данные каталога для скаутинга не найдены. Для Rails есть свежий сгенерированный gem `amocrm` (Hexlet, октябрь 2026).

### Cited Findings
- Лимиты: «не более 7 запросов в секунду», лимит применяется к IP. При 429 приходит `retry_after` (в примере — 300). Частые 429 ведут к 403 на любые запросы, то есть к блокировке аккаунта или IP. Повторные запросы одних и тех же данных и «неконтролируемый перебор всех данных» могут ограничиваться (по сниппету поиска) — [developers.kommo.com/docs/limitations](https://developers.kommo.com/docs/limitations), [lead-capture](https://developers.kommo.com/docs/lead-capture), [kommo.com/…/recommendations](https://www.kommo.com/developers/content/api/recommendations/).
- Публикация (Kommo, по сниппету поиска):
  - приватные интеграции модерацию не проходят и в маркетплейсе не публикуются; публичные проходят модерацию;
  - технический аккаунт бесплатный, создаётся через чат с поддержкой и истекает через месяц, если ничего не опубликовано;
  - чек-лист: собирать только нужные данные, быть понятным нетехническим пользователям, не продвигать подписки Kommo через Kommo Partners, устанавливаться только по клику «Install»;
  - модерация: сначала аудит кода, затем функциональный тест, 1–2 рабочих дня.
  
  Источники — [getting-listed](https://developers.kommo.com/docs/getting-listed), [moderation-process](https://developers.kommo.com/docs/moderation-process), [get-started-public](https://developers.kommo.com/docs/get-started-public).
- Партнёрская (реферальная) программа: 35% с покупок привлечённых клиентов, после $10K — 50%. Это сторонний источник и модель для агентств, а не для виджетов (по сниппету поиска) — [uppromote.com](https://uppromote.com/affiliate-directory/amocrm/).
- MCP:
  - `cAIborg-ai/amocrm-mcp` — Python FastMCP, 36 инструментов в 11 доменах; долгоживущий токен или OAuth с автообновлением, обновлённые токены сохраняются на диск; stdio и SSE; MIT; 2★, 3 форка, 1 коммит — [GitHub](https://github.com/cAIborg-ai/amocrm-mcp);
  - `theYahia/amocrm-mcp` — TypeScript, 19 инструментов, **архивирован** (код переехал в монорепо `theYahia/WWmcp`); 0★; npm `@theyahia/amocrm-mcp` 2.0.2 создан 2026-03-30 — [GitHub](https://github.com/theYahia/amocrm-mcp), [npm](https://registry.npmjs.org/@theyahia/amocrm-mcp);
  - PyPI `amocrm-mcp` 0.3.0 (ilyaberdysh, 2026-03-21/22) — [PyPI](https://pypi.org/project/amocrm-mcp/);
  - «Kommo MCP Server» на Glama, около 152 инструментов (по сниппету поиска, не проверено) — [glama.ai](https://glama.ai/mcp/servers/d8tvdmjuy7);
  - статья о создании своего MCP для amoCRM за 2 недели (по сниппету поиска) — [habr.com/ru/articles/964896](https://habr.com/ru/articles/964896/).
- Ruby:
  - `amocrm` 0.6.2 от 2026-10-06, первая версия 2026-02-02, 14 версий; Ruby 3.2+; «generated with Stainless»; токен-авторизация (`AMOCRM_AUTH_TOKEN` и subdomain); MIT на GitHub; 4★, 79 коммитов — [GitHub](https://github.com/Hexlet/amocrm-ruby), [RubyGems](https://rubygems.org/gems/amocrm);
  - устаревшие: `amorail` 0.7.2 (2024-01-08, около 93 тыс. загрузок) — [RubyGems](https://rubygems.org/gems/amorail); `amocrm-rails` 0.0.12 (2022-02-08) — [RubyGems](https://rubygems.org/gems/amocrm-rails).
- Go: 24 модуля на pkg.go.dev, лидера нет. Самые свежие:
  - `git.stit.tech/stit-core/golibinfra/amocrmkit` v1.2.0 (2026-09-29; REST v4, Chat API, OAuth2, лимиты на аккаунт);
  - `alextixru/amocrm-sdk-go` v0.3.3 (2025-12-26);
  - `BadHomer/go-amocrm` v0.1.12 (2026-01-14);
  - `dedomorozoff/amocrm-go-v4` v0.1.1 (2025-12-02).
  
  Источник — [pkg.go.dev](https://pkg.go.dev/search?q=amocrm&m=package).

### Inferences
- Для **канала продаж** amoCRM сопоставим с Битрикс24 по процессу (модерация, технический аккаунт), но модель монетизации виджетов не выяснена. Это блокирует бизнес-оценку канала.
- Gem `amocrm` сгенерирован Stainless, значит, у Hexlet есть OpenAPI-спецификация amoCRM. Для своего Go-клиента или MCP её можно взять как основу, если она опубликована в репозитории (не проверено).
- Комьюнити-MCP молодые и малоактивные. Для агентов «LLM-компании» проще написать свой узкий MCP (лиды, задачи, воронки) на gem `amocrm`, чем доверять токены CRM чужому коду.

### Gaps
- Комиссия и биллинг платных виджетов amoCRM.ru, есть ли «amoМаркет» с оплатой через платформу — не найдены.
- Данные каталога маркетплейса amoCRM (установки, рейтинги) и наличие API к ним — не найдены.
- Отличия условий публикации amoCRM.ru (РФ) от Kommo — не проверены: страницы amocrm.ru и developers.kommo.com недоступны из среды, лимит поиска исчерпан.

---

## В4. Kwork: API, библиотеки, условия использования, сигналы спроса, найм

### Takeaway
Официального публичного API у Kwork не нашли. Единственная живая обёртка — `kesha1225/kwork` (MIT, 70★): она ходит в закрытый мобильный API `api.kwork.ru` и веб-флоу, умеет читать ленту биржи с фильтрами и упирается в антибот-капчу. Пункт о запрете автоматизации в оферте Kwork найти не удалось. Биржа содержит ценные сигналы спроса: желаемый и допустимый бюджет, число предложений, «Нанято, %», категория.

### Cited Findings
- `kesha1225/kwork`: «Асинхронная обёртка над закрытым api для фриланс биржи kwork.ru»; «не является официальным SDK»; часть функций через веб-эндпоинт «может ломаться без предупреждения». MIT, 70★, 11 форков, 60 коммитов, темы kwork-api и kwork-bot — [GitHub](https://github.com/kesha1225/kwork).
- Методы и транспорт по `docs/guide.md` — [guide.md](https://raw.githubusercontent.com/kesha1225/kwork/master/docs/guide.md):
  - мобильный API `api.kwork.ru` и веб `kwork.ru` (`POST /api/offer/createoffer`, `POST /getWebAuthToken`), WebSocket `wss://notice.kwork.ru/ws/public/{channel}`;
  - авторизация логином и паролем;
  - `get_projects(categories_ids, price_from, price_to, hiring_from, page)` возвращает `list[WantWorker]`, `get_categories()` — `list[ParentCategory]`;
  - отклик на проект — `api.web.submit_exchange_offer(...)`;
  - предупреждение: сайт может ответить «Подтвердите, что вы не робот», советы — повтор или смена IP через socks5-прокси.
- В папке `docs/` есть `openapi.json` (содержимое не проверено) — [docs](https://github.com/kesha1225/kwork/tree/master/docs).
- Активность:
  - PyPI `kwork`: 0.0.1–0.0.2 (2020-02-03), 0.0.3–0.0.5 (2021-08-17), 0.1.0 (2025-12-01), 0.1.1 (2025-12-05), 0.2.0 (2026-02-10) — [PyPI JSON](https://pypi.org/pypi/kwork/json);
  - последние коммиты — 2026-04-14 (зависимости), 2026-02-10 (документация, тесты); самая ранняя дата на странице коммитов — 2022-06-22 — [коммиты](https://github.com/kesha1225/kwork/commits/master).
- Оферта kwork.ru/terms (по сниппету поиска):
  - оператор — RemoteFirst Group Limited (Гонконг);
  - соглашение принимается регистрацией;
  - платежи идут через сервис Paymore;
  - оператор вправе контролировать личную переписку и расторгнуть соглашение в одностороннем порядке.
  
  Пункт об автоматизированном сборе или ботах в сниппетах не встретился — [kwork.ru/terms](https://kwork.ru/terms).
- С 1 сентября 2025 года запрещены услуги для заблокированных соцсетей и продажа аккаунтов (по сниппету поиска) — [blog.kwork.ru](https://blog.kwork.ru/updates/izmenenie-pravil-s-1-sentyabrya-2025-zapret-uslug-dlya-zablokirovannyx-socsetej-i-prodazhi-akkauntov).
- Карточки биржи показывают «Желаемый бюджет: до 5 000 ₽», «Допустимый: до 15 000 ₽», «Предложений: 18», «Нанято: 47%» (по сниппету поиска) — [kwork.ru/projects](https://kwork.ru/projects). Фильтр «Количество предложений» обновляется в реальном времени. Покупатель задаёт максимальный бюджет как верхнюю границу откликов (по сниппету поиска; из какой именно статьи блога — не установлено) — [blog.kwork.ru: обновления Биржи](https://blog.kwork.ru/updates/dolgozhdannye-obnovleniya-birzhi), [blog.kwork.ru: декабрь 2021](https://blog.kwork.ru/updates/dekabr-2021-poslednie-obnovleniya-uxodyashhego-goda).
- Существует коммерческий скрапер Kwork (по сниппету поиска, по заголовку, не оценивался) — [spider.cloud](https://spider.cloud/scrapers/kwork-com-scraper).
- Гемов для Kwork нет: поиск RubyGems по `kwork` пуст — [rubygems.org](https://rubygems.org/). Go-модули для Kwork не проверялись.

### Inferences
- Из ленты биржи можно собрать **индекс спроса и конкуренции** по категориям. Еженедельный снимок даёт: число новых заявок, медианный желаемый и допустимый бюджет, среднее число предложений на заявку (насыщенность исполнителями) и долю «Нанято» (платёжеспособность покупателей). Для идей вида «сервис вместо фриланса» это прямой сигнал.
- Риск автоматизации высокий: закрытый API, авторизация логином и паролем, капча и советы менять IP, право оператора расторгнуть соглашение. Для MVP разумен ручной или полуручной просмотр ленты по 5–10 категориям раз в неделю. Read-only сборщик стоит писать только после чтения полной оферты; автоотклики не делать.
- **Найм** фрилансеров через Kwork — вручную через сайт и приложение: оплата через Paymore, переписка модерируется. API для размещения заказов нет.

### Gaps
- Полный текст оферты (пункты об автоматизированном доступе и парсинге) не получен: kwork.ru не открывается из среды, лимит поиска исчерпан.
- Есть ли у Kwork официальная OpenAPI или партнёрский API (упоминание «OpenAPI api.kwork.ru» в README обёртки) — не проверено.
- Метрики масштаба биржи (число заявок в неделю по категориям) — не найдены.

---

## Deep-dive 1. RFSD — основа для оценки ниши и концентрации

**Что это.** Академическая открытая база неконсолидированной бухотчётности всех юрлиц РФ за 2011–2025. Её делает команда из ЕУСПб (Институт проблем правоприменения) с рецензируемой статьёй в Scientific Data (2025) — [README](https://github.com/irlcode/RFSD), [eusp.org](https://eusp.org/en/news/researchers-from-the-institute-for-the-rule-of-law-have-published-a-new-article-on-the-russian-accounting-reporting-database).

**Почему брать.**
1. Лицензия CC BY 4.0 разрешает коммерческое использование и зеркалирование при атрибуции — [README](https://raw.githubusercontent.com/irlcode/RFSD/main/README.md).
2. Есть отраслевые поля `okved` и `okved_section`, ИНН и ОГРН, регион — [tfp.md](https://raw.githubusercontent.com/irlcode/RFSD/main/use_cases/tfp.md), [README](https://github.com/irlcode/RFSD).
3. Объём посильный: 2025 год — около 534 МБ Parquet — [README](https://github.com/irlcode/RFSD).
4. Проект поддерживается: релизы 2025-05, 2025-08/09, 2026-08, 2026-09, последний коммит 2026-09-02 — [коммиты](https://github.com/irlcode/RFSD/commits/main).
5. База снимает необходимость скрапить bo.nalog.gov.ru, а у того неизвестны условия и устойчивость.

**Как встроить (вывод, проверить на практике).**
- Раз в год, в августе–сентябре после релиза, скачивать срез `year=<последний>` с HF или Zenodo. Пути вида `hf://datasets/irlspbru/RFSD/RFSD/year=2025/*.parquet` (прямое чтение через Polars на август 2026 сломано) — [README](https://raw.githubusercontent.com/irlcode/RFSD/main/README.md).
- В Rails: rake-задача на gem `duckdb` ([RubyGems](https://rubygems.org/gems/duckdb)) или в Go через `duckdb-go/v2` ([proxy.golang.org](https://proxy.golang.org/github.com/duckdb/duckdb-go/v2/@latest)). Задача агрегирует по `okved` на уровнях 2, 4 и 6 знаков (и при необходимости по `region`) и записывает в Postgres витрину `niche_stats`: `n_firms`, `revenue_total`, `cr1`, `cr3`, `hhi`, `top10` с ИНН и долями, рост к прошлому году.
- Для агентов сделать один MCP-инструмент `niche_concentration(okved, region?, year?)`, который читает витрину. Стоп-фактор «монополист» — порог вроде CR1 ≥ 50% или HHI ≥ 2500 (порог выбрать самим).
- Перед отсевом идеи проверять топ-5 через Checko (deep-dive 2): консолидация групп, свежие данные.

**Риски и ограничения.**
- Неконсолидированность и отсутствие ИП ведут к недооценке концентрации и размера ниши (вывод, см. В1).
- Годовой лаг: данные за 2026 год появятся к июлю 2027 — [README](https://github.com/irlcode/RFSD).
- Выбросы и импутация помечены флагами; их нужно фильтровать (`outlier`, `filed`) — [README](https://raw.githubusercontent.com/irlcode/RFSD/main/README.md).
- В 2025 году менялись формы — [README](https://raw.githubusercontent.com/irlcode/RFSD/main/README.md).
- Единицы измерения и точная семантика `okved` (основной код, источник — Статрегистр или ЕГРЮЛ) — **не проверено**.

**Вердикт: брать**, это ядро слоя 4b.

---

## Deep-dive 2. Checko API — точечное свежее обогащение

**Что даёт.** ЕГРЮЛ/ЕГРИП, бухотчётность ФНС по ИНН (`/finances`), суды, госзакупки, исполнительные производства, ЕФРСБ, поиск по основному ОКВЭД с фильтрами по региону, ОПФ и статусу (по сниппету поиска) — [api](https://checko.ru/integration/api), [search](https://checko.ru/integration/api/search), [finances](https://checko.ru/integration/api/finances).

**Цена** (по сниппету поиска): 100 запросов в сутки бесплатно; дальше 0,15 ₽ за запрос (пополнение от 1 000 ₽) или 0,10 ₽ (от 50 000 ₽). Ответы 4xx/5xx не оплачиваются, есть суточный лимит трат — [checko.ru/integration/api](https://checko.ru/integration/api).

**Как встроить (вывод).**
- Тонкий HTTP-клиент: Rails-сервис на Faraday или Go-пакет. Ключ хранить в `credentials` или ENV, в кабинете выставить суточный лимит.
- Кэш ответов в Postgres с TTL около 30 дней для `/finances`, потому что отчётность годовая.
- MCP-инструменты для агентов: `company_card(inn)`, `company_finances(inn)`, `okved_players(okved, region)` с жёстким бюджетом вызовов на одну идею.
- Типичный сценарий для 50 идей в неделю: RFSD даёт топ-N игроков ниши, Checko даёт по ним свежие карточки, учредителей и аффилированность, а также суды и госзакупки как дополнительные стоп-факторы. Это порядка 1 000 вызовов в неделю, большая часть укладывается в бесплатную квоту (расчёт в В1).

**Ограничения.**
- Поиск по ОКВЭД возвращает компании без выручки, отсортированные по ОГРН, а не по выручке. Ранжировать нишу через Checko дорого: N+1 вызовов (по сниппету поиска) — [search](https://checko.ru/integration/api/search).
- По мнению конкурента, отчётность в Checko отстаёт на 12–18 месяцев — то есть тот же источник ГИР БО, что и в RFSD (по сниппету поиска) — [toolfox.ru](https://toolfox.ru/blog/chekko-obzor-tarify-otzyvy).
- Готового MCP нет, условия оферты по автоматизированному использованию API не читались — [checko.ru/offer](https://checko.ru/offer).

**Альтернативы.**
- DaData: финансы только на «Максимальном», цена не найдена; официальный MCP — 4 инструмента, не проверено — [theYahia/dadata-mcp](https://github.com/theYahia/dadata-mcp).
- Контур.Фокус API — от 18 000 ₽/мес (по сниппету поиска) — [toolfox.ru](https://toolfox.ru/services/dannye-i-api/obogashchenie-dannyh-kompanii).

**Вердикт: брать** как недорогой платный слой свежести и проверки аффилированности поверх RFSD.

---

## Deep-dive 3. Официальный dev-стек Битрикс24 (b24gosdk + mcp-rest-doc + templates-mcp)

**Роль в конвейере.** Канал продаж B2B-приложений через Маркет и инструменты агентов-кодеров «LLM-компании», которые будут писать такие приложения.

**Компоненты.**
1. **`mcp-rest-doc`** — подключается к Claude Code как HTTP MCP `https://mcp-dev.bitrix24.com/mcp` без ключа. Даёт поиск по методам, событиям и статьям документации REST и снижает число галлюцинаций в именах методов — [GitHub](https://github.com/bitrix24/mcp-rest-doc). Брать сразу: бесплатно, без токенов, данных портала не касается.
2. **`b24gosdk`** — Go SDK из официальной организации, указан в официальной документации. Закрывает webhooks, OAuth и жизненный цикл установки приложения, refresh с callback для сохранения токенов, batch и chunked batch, retry при `QUERY_LIMIT_EXCEEDED`, пагинацию, тестовый фейковый портал — [GitHub](https://github.com/bitrix24/b24gosdk), [b24restdocs/sdk](https://github.com/bitrix24/b24restdocs/tree/main/sdk). Минусы: проекту около 2 месяцев (v0.1.0 — 2026-08-06), 4★ и 0 форков, код писался при участии ИИ (соавтор «claude»), типизированных CRM-сервисов нет — [proxy.golang.org](https://proxy.golang.org/github.com/bitrix24/b24gosdk/@v/list), [коммиты](https://github.com/bitrix24/b24gosdk/commits/main). Брать, если бэкенд на Go: официальность и покрытие сложных мест (refresh, лимиты) важнее молодости. Версию закреплять.
3. **`templates-mcp`** — референсная архитектура MCP-сервера под портал Битрикс24 — [GitHub](https://github.com/bitrix24/templates-mcp):
   - авторизация через webhook или мультиарендный OAuth;
   - три транспорта;
   - редактирование секретов в логах;
   - DXT для Claude Desktop;
   - 27 инструментов по задачам.
   
   Это **референс**: стек Nuxt/TypeScript не совпадает с Rails и Go, а сама ниша «MCP-коннектор для Б24» в Маркете уже занята не меньше чем пятью приложениями (по сниппету поиска) — [пример](https://www.bitrix24.ru/apps/app/evrika.mcp/).

**Экономика канала** (РФ, по сниппету поиска).
- Подписочный пул отдаёт 85% разработчикам по «баллам использования», половина — сразу приложению, которое привело к продаже подписки — [subscription-details](https://apidocs.bitrix24.ru/market/monetization/subscription-details.html).
- Внешние платежи за функции сейчас без комиссии — [in-app-purchases](https://apidocs.bitrix24.ru/market/monetization/in-app-purchases.html), [sales-terms.md](https://raw.githubusercontent.com/bitrix24/b24restdocs/main/market/sales-terms.md).
- Платные приложения публикуют только ИП и юрлица, договор оформляется через ЭДО — [publication-requirements](https://apidocs.bitrix24.ru/market/preparing-to-publish/publication-requirements.html).
- Модерация может отказать без объяснения — [publication-requirements.md](https://raw.githubusercontent.com/bitrix24/b24restdocs/main/market/preparing-to-publish/publication-requirements.md).

**Технические рамки.** Около 2 запросов в секунду на портал (ведро на 50), batch до 50 вызовов, 480 секунд времени метода за 10 минут. REST не предназначен для массовых выгрузок (по сниппету поиска) — [limits](https://apidocs.bitrix24.ru/limits.html), [helpdesk](https://helpdesk.bitrix24.ru/open/15959788/).

**Хранение токенов (вывод).**
- Webhook URL — это секрет, хранить в credentials или ENV.
- OAuth-токены каждого портала хранить в БД с шифрованием атрибутов (Rails `encrypts`) и обновлять через refresh-callback (в b24gosdk он есть).
- В MCP-сценарии по примеру templates-mcp — keychain или секрет-хранилище.

**Вердикт: брать** mcp-rest-doc сразу и b24gosdk для Go-бэкенда; templates-mcp — **референс**; для Rails — **писать** тонкий REST-клиент.

---

## Вывод по слою: что взять готовым / что как референс / что писать самому

**Взять готовым**
1. **RFSD** (CC BY 4.0) — ежегодный бесплатный источник выручки всех юрлиц с `okved` и `okved_section`. На нём строится фильтр «монополист в нише» (CR1/CR3/HHI) и оценка размера ниши — [github.com/irlcode/RFSD](https://github.com/irlcode/RFSD).
2. **Checko API** — платный, но копеечный слой свежести: карточки, отчётность, суды и госзакупки по ИНН для топ-N игроков, 100 запросов в сутки бесплатно — [checko.ru/integration/api](https://checko.ru/integration/api).
3. **MCP по документации Битрикс24** (`mcp-dev.bitrix24.com/mcp`) — для агентов-кодеров, без токенов — [mcp-rest-doc](https://github.com/bitrix24/mcp-rest-doc).
4. **`b24gosdk`** — если бэкенд интеграций на Go — [GitHub](https://github.com/bitrix24/b24gosdk).
5. Gem **`amocrm`** (Hexlet) — если понадобится интеграция с amoCRM из Rails, версию закрепить — [GitHub](https://github.com/Hexlet/amocrm-ruby).

**Использовать как референс**
- Скрытое JSON-API bo.nalog.gov.ru (Infostart, мелкие парсеры) — только как резерв для точечной проверки, если Checko недоступен: условия и устойчивость неизвестны — [infostart.ru](https://infostart.ru/public/2557058/).
- Открытые данные ФНС `revexp` (доходы и расходы всех организаций, обновление ежемесячно) — резерв на случай остановки RFSD — [nalog.gov.ru](https://www.nalog.gov.ru/rn07/opendata/7707329152-revexp/).
- `templates-mcp` Битрикс24 — архитектура своего MCP или приложения: OAuth, транспорты, секреты — [GitHub](https://github.com/bitrix24/templates-mcp).
- Комьюнити-MCP для amoCRM (`cAIborg-ai/amocrm-mcp`, `theYahia/amocrm-mcp`) и DaData (`theYahia/dadata-mcp`) — как наборы инструментов и схемы авторизации — [cAIborg](https://github.com/cAIborg-ai/amocrm-mcp), [theYahia](https://github.com/theYahia/dadata-mcp).
- `kesha1225/kwork` — модель данных биржи (`WantWorker`, категории, фильтры `price_from/to`, `hiring_from`) и картина антибот-защиты — [GitHub](https://github.com/kesha1225/kwork).

**Писать самому**
1. **ETL «RFSD → DuckDB → Postgres-витрина ниш»** и MCP-инструмент `niche_concentration(okved, region, year)`, запуск раз в год (август–сентябрь).
2. **Тонкий клиент Checko** с кэшем, бюджетом вызовов и MCP-инструментами `company_card`, `company_finances`, `okved_players`. Готового MCP нет.
3. **Скаутинг каталогов Битрикс24 Маркета и маркетплейса amoCRM.** Документированного API нет, поэтому еженедельные ручные или полуручные снимки публичных страниц: категории, новые приложения, рейтинги и отзывы. Автоматический сбор — только после проверки правил.
4. **Сигналы спроса Kwork** — ручной или полуручной недельный снимок биржи по 5–10 категориям: заявки, бюджеты, число предложений, «Нанято %». Read-only сборщик — только после изучения полной оферты; автоотклики и бот-аккаунты не делать. Найм — вручную.
5. **REST-клиент Битрикс24 для Rails** (webhook и OAuth, refresh, batch, retry по образцу b24gosdk), если бэкенд на Rails. Ruby-гемы устарели — [bitrix24_cloud_api](https://rubygems.org/gems/bitrix24_cloud_api).

**Главные риски слоя:** недооценка концентрации из-за неконсолидированности и отсутствия ИП в RFSD; неизвестные условия автоматизации у ФНС, Kwork и каталогов маркетплейсов; непрозрачная экономика подписочного пула Битрикс24; неизвестная комиссия маркетплейса amoCRM.

---

## Источники

**ГИР БО, ФНС, RFSD**
- https://github.com/irlcode/RFSD
- https://raw.githubusercontent.com/irlcode/RFSD/main/README.md
- https://github.com/irlcode/RFSD/tree/main
- https://raw.githubusercontent.com/irlcode/RFSD/main/use_cases/tfp.md
- https://github.com/irlcode/RFSD/commits/main
- https://huggingface.co/datasets/irlspbru/RFSD (ссылка из README, из среды недоступна)
- https://zenodo.org/records/14622209 (по сниппету поиска)
- https://arxiv.org/html/2501.05841v1 (по сниппету поиска)
- https://eusp.org/en/news/researchers-from-the-institute-for-the-rule-of-law-have-published-a-new-article-on-the-russian-accounting-reporting-database (по сниппету поиска)
- https://www.nalog.gov.ru/rn07/opendata/7707329152-revexp/ (по сниппету поиска)
- https://www.nalog.gov.ru/rn23/opendata/7707329152-revexp/ (по сниппету поиска)
- https://buh.ru/news/fns-raskryla-svedeniya-o-dokhodakh-kompaniy-v-2022-godu-i-o-nalogoplatelshchikakh-na-spetsrezhimakh.html (по сниппету поиска)
- https://spmag.ru/news/fns-razmestila-otkrytye-dannye-po-uplate-nalogov (по сниппету поиска)
- https://www.consultant.ru/document/cons_doc_LAW_347267/15655fdbaad6f585e347b871961f6ac8c1cd9559/ (по сниппету поиска)
- https://data.nalog.ru/html/sites/www.rn69.nalog.ru/Info/GIRBO.pdf (по сниппету поиска)
- https://infostart.ru/public/2557058/ (по сниппету поиска)
- https://github.com/search?q=bo.nalog.ru&type=repositories
- https://github.com/Vadenoire-ONE/bo.nalog.ru
- https://github.com/surikatio/OKVED_parser

**Сторонние API и MCP по компаниям**
- https://checko.ru/integration/api (по сниппету поиска)
- https://checko.ru/integration/api/search (по сниппету поиска)
- https://checko.ru/integration/api/finances (по сниппету поиска)
- https://checko.ru/offer (по сниппету поиска, по заголовку)
- https://toolfox.ru/blog/chekko-obzor-tarify-otzyvy (по сниппету поиска)
- https://toolfox.ru/services/s/dadata (по сниппету поиска)
- https://toolfox.ru/services/dannye-i-api/obogashchenie-dannyh-kompanii (по сниппету поиска)
- https://saby.ru/help/partner/api/abstract (по сниппету поиска)
- https://github.com/theYahia/dadata-mcp
- https://registry.npmjs.org/@theyahia/dadata-mcp
- https://registry.npmjs.org/@metarebalance/dadata-mcp
- https://mcp.so/servers/dadata-mcp (по сниппету поиска)
- https://pypi.org/project/mcp-egrul/
- https://mcpservers.org/de/servers/atomno-labs/mcp-egrul (по сниппету поиска)
- https://github.com/hflabs
- https://pypi.org/project/dadata/
- https://github.com/ekomobile/dadata
- https://proxy.golang.org/github.com/ekomobile/dadata/v2/@latest
- https://rubygems.org/gems/dadata
- https://rubygems.org/gems/dadatas
- https://rubygems.org/gems/duckdb
- https://proxy.golang.org/github.com/duckdb/duckdb-go/v2/@latest

**Битрикс24**
- https://apidocs.bitrix24.ru/limits.html (по сниппету поиска)
- https://apidocs.bitrix24.com/limits.html (по сниппету поиска)
- https://helpdesk.bitrix24.ru/open/15959788/ (по сниппету поиска)
- https://apidocs.bitrix24.ru/sdk/mcp.html (по сниппету поиска, по заголовку)
- https://github.com/bitrix24
- https://github.com/search?q=org%3Abitrix24+mcp&type=repositories
- https://github.com/bitrix24/mcp-rest-doc
- https://github.com/bitrix24/templates-mcp
- https://github.com/bitrix24/templates-mcp/commits/main
- https://github.com/bitrix24/b24gosdk
- https://github.com/bitrix24/b24gosdk/commits/main
- https://proxy.golang.org/github.com/bitrix24/b24gosdk/@v/list
- https://github.com/bitrix24/b24restdocs/tree/main/sdk
- https://raw.githubusercontent.com/bitrix24/b24restdocs/main/market/preparing-to-publish/publication-requirements.md
- https://raw.githubusercontent.com/bitrix24/b24restdocs/main/market/sales-terms.md
- https://raw.githubusercontent.com/bitrix24/b24restdocs/main/market/promoting-and-analytics/users-rating.md
- https://apidocs.bitrix24.ru/market/monetization/subscription-details.html (по сниппету поиска)
- https://apidocs.bitrix24.ru/market/preparing-to-publish/publication-requirements.html (по сниппету поиска)
- https://apidocs.bitrix24.ru/market/monetization/in-app-purchases.html (по сниппету поиска)
- https://www.bitrix24.ru/apps/dev.php (по сниппету поиска)
- https://helpdesk.bitrix24.ru/open/23776074/ (по сниппету поиска)
- https://www.bitrix24.ru/apps/ (по сниппету поиска)
- https://www.bitrix24.ru/apps/collection/16420306/ (по сниппету поиска)
- https://www.bitrix24.ru/apps/partner/336901/ (по сниппету поиска)
- https://www.bitrix24.ru/apps/app/evrika.mcp/ (по сниппету поиска)
- https://www.bitrix24.ru/apps/app/zeliboba.mcp/ (по сниппету поиска)
- https://www.bitrix24.ru/apps/app/kolibbri.mcp/ (по сниппету поиска)
- https://www.bitrix24.ru/apps/app/intogroup.mcp/ (по сниппету поиска)
- https://www.bitrix24.ru/apps/app/intelmediasoft.mcp24/ (по сниппету поиска)
- https://habr.com/ru/articles/903190 (по сниппету поиска)
- https://pypi.org/project/bitrix24-mcp/
- https://pkg.go.dev/search?q=bitrix24&m=package
- https://rubygems.org/gems/bitrix24_cloud_api
- https://rubygems.org/gems/bitrix_webhook
- https://rubygems.org/gems/omniauth-bitrix24

**amoCRM / Kommo**
- https://developers.kommo.com/docs/limitations (по сниппету поиска)
- https://developers.kommo.com/docs/lead-capture (по сниппету поиска)
- https://www.kommo.com/developers/content/api/recommendations/ (по сниппету поиска)
- https://developers.kommo.com/docs/getting-listed (по сниппету поиска)
- https://developers.kommo.com/docs/moderation-process (по сниппету поиска)
- https://developers.kommo.com/docs/get-started-public (по сниппету поиска)
- https://uppromote.com/affiliate-directory/amocrm/ (по сниппету поиска)
- https://github.com/cAIborg-ai/amocrm-mcp
- https://github.com/theYahia/amocrm-mcp
- https://registry.npmjs.org/@theyahia/amocrm-mcp
- https://pypi.org/project/amocrm-mcp/
- https://glama.ai/mcp/servers/d8tvdmjuy7 (по сниппету поиска)
- https://habr.com/ru/articles/964896/ (по сниппету поиска)
- https://github.com/Hexlet/amocrm-ruby
- https://rubygems.org/gems/amocrm
- https://rubygems.org/gems/amorail
- https://rubygems.org/gems/amocrm-rails
- https://pkg.go.dev/search?q=amocrm&m=package

**Kwork**
- https://github.com/kesha1225/kwork
- https://github.com/kesha1225/kwork/commits/master
- https://github.com/kesha1225/kwork/tree/master/docs
- https://raw.githubusercontent.com/kesha1225/kwork/master/docs/guide.md
- https://pypi.org/pypi/kwork/json
- https://kwork.ru/terms (по сниппету поиска)
- https://kwork.ru/projects (по сниппету поиска)
- https://blog.kwork.ru/updates/dolgozhdannye-obnovleniya-birzhi (по сниппету поиска)
- https://blog.kwork.ru/updates/dekabr-2021-poslednie-obnovleniya-uxodyashhego-goda (по сниппету поиска)
- https://blog.kwork.ru/updates/izmenenie-pravil-s-1-sentyabrya-2025-zapret-uslug-dlya-zablokirovannyx-socsetej-i-prodazhi-akkauntov (по сниппету поиска)
- https://spider.cloud/scrapers/kwork-com-scraper (по сниппету поиска, по заголовку)

**Не прочитано, но может пригодиться автору отчёта**
- https://www.sostav.ru/blogs/289407/104725 — «MCP-серверы для российского стека: с чем Claude Code уже работает, а где придётся собирать самому» (виден только заголовок в выдаче).
