# Mihomo Runet MRS

[![Build MRS rule sets](https://github.com/nofantasysorry/mihomo-runet-mrs/actions/workflows/build.yml/badge.svg)](https://github.com/nofantasysorry/mihomo-runet-mrs/actions/workflows/build.yml)

Automatically generated **MRS rule sets** (`.mrs`) for [Mihomo](https://github.com/MetaCubeX/mihomo), converted from the GeoIP and GeoSite databases of [RunetFreedom / russia-v2ray-rules-dat](https://github.com/runetfreedom/russia-v2ray-rules-dat).

A GitHub Actions workflow checks the upstream databases and the converter twice a day. When something changes, every GeoIP and GeoSite category is converted into an individual `.mrs` file, verified with SHA-256 checksums, and published to the `main` branch. Empty rule sets are skipped automatically, force push is never used, and hand-maintained files (`README.md`, `LICENSE`, `.github/`) are never touched.

**[Русская версия](#русская-версия)** — описание проекта на русском языке находится ниже.

## Contents

- [Quick start](#quick-start)
- [How it works](#how-it-works)
- [Update schedule](#update-schedule)
- [When a new commit is created](#when-a-new-commit-is-created)
- [Configuration examples](#configuration-examples)
- [Category lists](#category-lists)
- [Files in this repository](#files-in-this-repository)
- [FAQ](#faq)
- [Acknowledgements](#acknowledgements)
- [License](#license)

## Quick start

Add these rule providers to your Mihomo configuration:

```yaml
rule-providers:
  geosite-ru:
    type: http
    behavior: domain
    format: mrs
    url: "https://raw.githubusercontent.com/nofantasysorry/mihomo-runet-mrs/main/geosite/category-ru.mrs"
    path: ./ruleset/category-ru.mrs
    interval: 43200

  geoip-ru:
    type: http
    behavior: ipcidr
    format: mrs
    url: "https://raw.githubusercontent.com/nofantasysorry/mihomo-runet-mrs/main/geoip/ru.mrs"
    path: ./ruleset/ru.mrs
    interval: 43200
```

Then reference them in the rules section (replace `PROXY` and `DIRECT` with your own policies):

```yaml
rules:
  - RULE-SET,geosite-ru,PROXY
  - RULE-SET,geoip-ru,DIRECT,no-resolve
```

`interval: 43200` is the refresh period in seconds: 43200 = 12 hours. Mihomo downloads updated rule sets by itself, nothing needs to be restarted.

## How it works

```text
runetfreedom/russia-v2ray-rules-dat (release branch)
        |
        |  1. the current upstream commit is pinned
        |  2. geoip.dat and geosite.dat are downloaded from it
        |  3. both files are verified against upstream SHA-256
        v
MetaCubeX/meta-rules-converter (pinned commit)
        |
        |  4. the converter is built from source
        |  5. both databases are converted to individual .mrs files
        |  6. empty rule sets are removed and recorded
        v
geosite/*.mrs + geoip/*.mrs  ->  the main branch of this repository
        |
        +-- 7. SHA-256 of every .mrs file goes to mrs-manifest.sha256
```

The category list is not maintained manually: when a new category appears upstream, it is included automatically in the next rebuild.

## Update schedule

The workflow runs twice a day, at **12:00** and **00:00** Moscow time (MSK, UTC+3).

```cron
0 9,21 * * *
```

GitHub Actions evaluates cron expressions in UTC only, so the workflow file contains the UTC equivalent: 09:00 UTC = 12:00 MSK, 21:00 UTC = 00:00 MSK. The schedule trigger has no timezone option; a `timezone` key there would simply be ignored.

Note: scheduled runs may start a few minutes late. GitHub queues scheduled workflows on shared infrastructure, and during busy periods the delay can reach tens of minutes. This is normal and does not affect the result: the check itself is fast, and a rebuild happens only when the data actually changed.

## When a new commit is created

A scheduled run does not necessarily produce a new commit. The repository is rebuilt only when one of the following is detected:

- `geoip.dat` changed upstream;
- `geosite.dat` changed upstream;
- `meta-rules-converter` got a new commit;
- a published `.mrs` file is missing or fails its integrity check;
- a required service file (`mrs-manifest.sha256`, `build-info.txt`, etc.) is missing.

If nothing changed, the workflow finishes successfully without rebuilding and without a commit.

## Configuration examples

### Basic: route Russian sites directly, everything else via proxy

```yaml
rule-providers:
  geosite-ru:
    type: http
    behavior: domain
    format: mrs
    url: "https://raw.githubusercontent.com/nofantasysorry/mihomo-runet-mrs/main/geosite/category-ru.mrs"
    path: ./ruleset/category-ru.mrs
    interval: 43200

  geoip-ru:
    type: http
    behavior: ipcidr
    format: mrs
    url: "https://raw.githubusercontent.com/nofantasysorry/mihomo-runet-mrs/main/geoip/ru.mrs"
    path: ./ruleset/ru.mrs
    interval: 43200
```

```yaml
rules:
  - RULE-SET,geosite-ru,DIRECT
  - RULE-SET,geoip-ru,DIRECT,no-resolve
```

### Blocked resources

```yaml
rule-providers:
  ru-blocked-domain:
    type: http
    behavior: domain
    format: mrs
    url: "https://raw.githubusercontent.com/nofantasysorry/mihomo-runet-mrs/main/geosite/ru-blocked.mrs"
    path: ./ruleset/ru-blocked-domain.mrs
    interval: 43200

  ru-blocked-ip:
    type: http
    behavior: ipcidr
    format: mrs
    url: "https://raw.githubusercontent.com/nofantasysorry/mihomo-runet-mrs/main/geoip/ru-blocked.mrs"
    path: ./ruleset/ru-blocked-ip.mrs
    interval: 43200
```

```yaml
rules:
  - RULE-SET,ru-blocked-domain,PROXY
  - RULE-SET,ru-blocked-ip,PROXY,no-resolve
```

### Full list of categories

See [`categories.md`](categories.md) — it is regenerated on every rebuild and contains every available rule set. You can also browse the [`geosite/`](geosite/) and [`geoip/`](geoip/) directories directly.

## Category lists

The current list of generated rule sets is available in:

| File | Contents |
|---|---|
| [`categories.md`](categories.md) | Human-readable index of all rule sets |
| [`geosite-categories.txt`](geosite-categories.txt) | Plain list of GeoSite `.mrs` files |
| [`geoip-categories.txt`](geoip-categories.txt) | Plain list of GeoIP `.mrs` files |

These files are regenerated automatically from the actual conversion output. Only successfully generated, non-empty MRS files are included.

## Files in this repository

| Path | Description |
|---|---|
| `geosite/*.mrs` | Generated GeoSite rule sets (`behavior: domain`) |
| `geoip/*.mrs` | Generated GeoIP rule sets (`behavior: ipcidr`) |
| `categories.md` | Index of all available rule sets |
| `geosite-categories.txt`, `geoip-categories.txt` | Plain category lists |
| `source.commit` | Upstream snapshot used for the last rebuild |
| `converter.commit` | `meta-rules-converter` commit used for the last rebuild |
| `geoip.dat.sha256`, `geosite.dat.sha256` | SHA-256 of the source databases |
| `mrs-manifest.sha256` | SHA-256 of every published `.mrs` file |
| `skipped-empty-rules.txt` | Rule sets skipped because they are empty |
| `build-info.txt` | Summary of the last actual rebuild |

`build-info.txt` stores the time of the last **actual rebuild**, not of every workflow run. If a check finds no changes, no commit is created and `build-info.txt` stays unchanged.

## FAQ

### Why does the scheduled run start at 00:07 instead of 00:00?

GitHub Actions runs scheduled workflows from a shared queue. Under load, runs start a few minutes (sometimes up to half an hour) later than the cron time. The data itself is unaffected: the check compares checksums, so a late run behaves exactly like a punctual one.

### Why is the cron in the workflow written in UTC and not in Moscow time?

GitHub Actions evaluates cron expressions in UTC only — the `schedule` trigger has no timezone option. A `timezone` key there (GitLab CI syntax) is silently ignored. The workflow file therefore stores the UTC equivalent of the Moscow schedule: `0 9,21 * * *` = 12:00 and 00:00 MSK.

### Some category has no `.mrs` file. Where is it?

Some upstream categories contain no rules that can be represented in the MRS format, so the converter produces a zero-byte file. Such files are not published; their names are recorded in [`skipped-empty-rules.txt`](skipped-empty-rules.txt). The build fails only if no usable GeoSite or GeoIP files remain at all.

### How do I trigger a rebuild manually?

Open the **Actions** tab, select **Build MRS rule sets**, and click **Run workflow**. The same change detection applies: if nothing changed upstream, no commit is created.

### How do I know the downloaded rule sets are intact?

Every published `.mrs` file has its SHA-256 recorded in `mrs-manifest.sha256`, and this manifest is verified on every run. If any file is missing or modified, the workflow triggers a full rebuild.

## Acknowledgements

This repository is built on the work of these projects and their contributors:

- [RunetFreedom / russia-v2ray-rules-dat](https://github.com/runetfreedom/russia-v2ray-rules-dat)
- [RunetFreedom / russia-blocked-geoip](https://github.com/runetfreedom/russia-blocked-geoip)
- [RunetFreedom / russia-blocked-geosite](https://github.com/runetfreedom/russia-blocked-geosite)
- [V2Fly / domain-list-community](https://github.com/v2fly/domain-list-community)
- [MetaCubeX / meta-rules-converter](https://github.com/MetaCubeX/meta-rules-converter)
- [MetaCubeX / mihomo](https://github.com/MetaCubeX/mihomo)

Additional upstream data sources may be included indirectly through the RunetFreedom databases. Please refer to the respective upstream repositories for complete source information, attribution, and licensing terms.

## License

This repository is distributed under the **GNU General Public License v3.0 (GPL-3.0)**.

Generated rule sets are derived from upstream datasets and remain subject to the licenses and terms of their respective upstream sources.

See [`LICENSE`](./LICENSE) for details.

---

# Русская версия

Автоматически генерируемые **MRS-наборы правил** (`.mrs`) для [Mihomo](https://github.com/MetaCubeX/mihomo), преобразованные из баз GeoIP и GeoSite проекта [RunetFreedom / russia-v2ray-rules-dat](https://github.com/runetfreedom/russia-v2ray-rules-dat).

Workflow GitHub Actions дважды в сутки проверяет исходные базы и конвертер. Если что-то изменилось, все категории GeoIP и GeoSite конвертируются в отдельные `.mrs`-файлы, проверяются по SHA-256 и публикуются в ветку `main`. Пустые наборы пропускаются автоматически, force-push не используется, ручные файлы репозитория (`README.md`, `LICENSE`, `.github/`) не изменяются.

**[English version](#mihomo-runet-mrs)** — описание проекта на английском языке находится выше.

## Содержание

- [Быстрый старт](#быстрый-старт)
- [Как это работает](#как-это-работает)
- [Расписание обновлений](#расписание-обновлений)
- [Когда создаётся новый коммит](#когда-создаётся-новый-коммит)
- [Примеры конфигурации](#примеры-конфигурации)
- [Список категорий](#список-категорий)
- [Служебные файлы](#служебные-файлы)
- [Частые вопросы](#частые-вопросы)
- [Благодарности](#благодарности)
- [Примечания](#примечания)
- [Лицензия](#лицензия)

## Быстрый старт

Добавьте провайдеры правил в конфиг Mihomo:

```yaml
rule-providers:
  geosite-ru:
    type: http
    behavior: domain
    format: mrs
    url: "https://raw.githubusercontent.com/nofantasysorry/mihomo-runet-mrs/main/geosite/category-ru.mrs"
    path: ./ruleset/category-ru.mrs
    interval: 43200

  geoip-ru:
    type: http
    behavior: ipcidr
    format: mrs
    url: "https://raw.githubusercontent.com/nofantasysorry/mihomo-runet-mrs/main/geoip/ru.mrs"
    path: ./ruleset/ru.mrs
    interval: 43200
```

И подключите их в правилах (замените `PROXY` / `DIRECT` на свои политики):

```yaml
rules:
  - RULE-SET,geosite-ru,PROXY
  - RULE-SET,geoip-ru,DIRECT,no-resolve
```

`interval: 43200` — период обновления в секундах: 43200 = 12 часов. Mihomo скачивает обновлённые наборы правил сам, ничего перезапускать не нужно.

## Как это работает

```text
runetfreedom/russia-v2ray-rules-dat (ветка release)
        |
        |  1. фиксируется текущий upstream-коммит
        |  2. из него скачиваются geoip.dat и geosite.dat
        |  3. оба файла сверяются с SHA-256 из upstream
        v
MetaCubeX/meta-rules-converter (фиксированный коммит)
        |
        |  4. конвертер собирается из исходников
        |  5. базы конвертируются в отдельные .mrs-файлы
        |  6. пустые наборы удаляются и учитываются
        v
geosite/*.mrs + geoip/*.mrs  ->  ветка main этого репозитория
        |
        +-- 7. SHA-256 каждого .mrs попадает в mrs-manifest.sha256
```

Список категорий не поддерживается вручную: если в upstream появится новая категория, она автоматически попадёт в следующую сборку.

## Расписание обновлений

Workflow запускается дважды в сутки: в **12:00** и **00:00** по московскому времени (MSK, UTC+3).

```cron
0 9,21 * * *
```

GitHub Actions вычисляет cron-выражения только по UTC, поэтому в файле workflow хранится UTC-эквивалент расписания: 09:00 UTC = 12:00 MSK, 21:00 UTC = 00:00 MSK. У триггера `schedule` нет опции часового пояса — ключ `timezone` там (синтаксис GitLab CI) просто игнорируется.

Обратите внимание: плановые запуски могут начинаться с задержкой в несколько минут. GitHub ставит запланированные workflow в общую очередь, и в часы пиковой нагрузки задержка может достигать десятков минут. Это нормально и не влияет на результат: сама проверка быстрая, а пересборка выполняется только при фактическом изменении данных.

## Когда создаётся новый коммит

Запуск по расписанию не обязательно приводит к созданию нового коммита. Пересборка выполняется только в следующих случаях:

- изменился `geoip.dat` в upstream;
- изменился `geosite.dat` в upstream;
- в `meta-rules-converter` появился новый коммит;
- опубликованный `.mrs`-файл отсутствует или не проходит проверку целостности;
- отсутствует один из служебных файлов (`mrs-manifest.sha256`, `build-info.txt` и т.д.).

Если изменений нет, workflow успешно завершается без пересборки и без коммита.

## Примеры конфигурации

### Базовый: русские сайты напрямую, остальное через прокси

```yaml
rule-providers:
  geosite-ru:
    type: http
    behavior: domain
    format: mrs
    url: "https://raw.githubusercontent.com/nofantasysorry/mihomo-runet-mrs/main/geosite/category-ru.mrs"
    path: ./ruleset/category-ru.mrs
    interval: 43200

  geoip-ru:
    type: http
    behavior: ipcidr
    format: mrs
    url: "https://raw.githubusercontent.com/nofantasysorry/mihomo-runet-mrs/main/geoip/ru.mrs"
    path: ./ruleset/ru.mrs
    interval: 43200
```

```yaml
rules:
  - RULE-SET,geosite-ru,DIRECT
  - RULE-SET,geoip-ru,DIRECT,no-resolve
```

### Заблокированные ресурсы

```yaml
rule-providers:
  ru-blocked-domain:
    type: http
    behavior: domain
    format: mrs
    url: "https://raw.githubusercontent.com/nofantasysorry/mihomo-runet-mrs/main/geosite/ru-blocked.mrs"
    path: ./ruleset/ru-blocked-domain.mrs
    interval: 43200

  ru-blocked-ip:
    type: http
    behavior: ipcidr
    format: mrs
    url: "https://raw.githubusercontent.com/nofantasysorry/mihomo-runet-mrs/main/geoip/ru-blocked.mrs"
    path: ./ruleset/ru-blocked-ip.mrs
    interval: 43200
```

```yaml
rules:
  - RULE-SET,ru-blocked-domain,PROXY
  - RULE-SET,ru-blocked-ip,PROXY,no-resolve
```

### Полный список категорий

Смотрите [`categories.md`](categories.md) — он перегенерируется при каждой пересборке и содержит все доступные наборы правил. Также можно просматривать каталоги [`geosite/`](geosite/) и [`geoip/`](geoip/) напрямую.

## Список категорий

Актуальный список сгенерированных наборов правил доступен в:

| Файл | Содержимое |
|---|---|
| [`categories.md`](categories.md) | Читаемый индекс всех наборов правил |
| [`geosite-categories.txt`](geosite-categories.txt) | Простой список GeoSite `.mrs` |
| [`geoip-categories.txt`](geoip-categories.txt) | Простой список GeoIP `.mrs` |

Эти файлы перегенерируются автоматически из фактического результата конвертации. Включаются только успешно созданные непустые MRS-файлы.

## Служебные файлы

| Путь | Описание |
|---|---|
| `geosite/*.mrs` | Сгенерированные GeoSite-наборы (`behavior: domain`) |
| `geoip/*.mrs` | Сгенерированные GeoIP-наборы (`behavior: ipcidr`) |
| `categories.md` | Индекс всех доступных наборов правил |
| `geosite-categories.txt`, `geoip-categories.txt` | Простые списки категорий |
| `source.commit` | Upstream-снимок последней пересборки |
| `converter.commit` | Коммит `meta-rules-converter` последней пересборки |
| `geoip.dat.sha256`, `geosite.dat.sha256` | SHA-256 исходных баз |
| `mrs-manifest.sha256` | SHA-256 каждого опубликованного `.mrs` |
| `skipped-empty-rules.txt` | Наборы, пропущенные из-за пустоты |
| `build-info.txt` | Сводка последней фактической пересборки |

`build-info.txt` хранит время последней **фактической пересборки**, а не каждого запуска workflow. Если проверка не находит изменений, новый коммит не создаётся и `build-info.txt` не меняется.

## Частые вопросы

### Почему плановый запуск начинается в 00:07, а не в 00:00?

GitHub Actions выполняет запланированные workflow из общей очереди. Под нагрузкой запуски начинаются на несколько минут (иногда до получаса) позже времени из cron. На данные это не влияет: проверка сравнивает контрольные суммы, поэтому поздний запуск ведёт себя точно так же, как вовремя начатый.

### Почему в workflow cron записан по UTC, а не по Москве?

GitHub Actions вычисляет cron только по UTC — у триггера `schedule` нет опции часового пояса. Ключ `timezone` там (синтаксис GitLab CI) молча игнорируется. Поэтому в файле workflow хранится UTC-эквивалент московского расписания: `0 9,21 * * *` = 12:00 и 00:00 MSK.

### У некоторой категории нет `.mrs`-файла. Где он?

Некоторые upstream-категории не содержат правил, которые можно представить в формате MRS, и конвертер создаёт пустой файл. Такие файлы не публикуются; их имена записываются в [`skipped-empty-rules.txt`](skipped-empty-rules.txt). Сборка падает, только если не осталось ни одного пригодного GeoSite- или GeoIP-файла.

### Как запустить пересборку вручную?

Откройте вкладку **Actions**, выберите **Build MRS rule sets** и нажмите **Run workflow**. Действует та же логика обнаружения изменений: если upstream не менялся, коммит не создаётся.

### Как убедиться, что скачанные наборы правил целы?

Для каждого опубликованного `.mrs`-файла его SHA-256 записан в `mrs-manifest.sha256`, и этот manifest проверяется при каждом запуске. Если какой-то файл отсутствует или изменён, workflow запускает полную пересборку.

## Благодарности

Репозиторий построен на работе следующих проектов и их участников:

- [RunetFreedom / russia-v2ray-rules-dat](https://github.com/runetfreedom/russia-v2ray-rules-dat)
- [RunetFreedom / russia-blocked-geoip](https://github.com/runetfreedom/russia-blocked-geoip)
- [RunetFreedom / russia-blocked-geosite](https://github.com/runetfreedom/russia-blocked-geosite)
- [V2Fly / domain-list-community](https://github.com/v2fly/domain-list-community)
- [MetaCubeX / meta-rules-converter](https://github.com/MetaCubeX/meta-rules-converter)
- [MetaCubeX / mihomo](https://github.com/MetaCubeX/mihomo)

Дополнительные upstream-источники данных могут использоваться косвенно через базы RunetFreedom. Полную информацию об источниках, авторах и лицензиях смотрите в соответствующих upstream-репозиториях.

## Примечания

Файлы в этом репозитории создаются автоматически из данных сторонних проектов.

Этот проект самостоятельно не определяет, должен ли конкретный домен, IP-адрес, сервис или ресурс блокироваться, проксироваться, обходиться напрямую или маршрутизироваться иным образом. Содержимое и назначение отдельных категорий определяется соответствующими upstream-источниками.

Некоторые upstream-записи могут быть проигнорированы конвертером, если их невозможно представить в целевом формате MRS. Предупреждения конвертера доступны в логах GitHub Actions.

## Лицензия

Этот репозиторий распространяется под лицензией **GNU General Public License v3.0 (GPL-3.0)**.

Сгенерированные наборы правил являются производными от upstream-данных и подчиняются условиям лицензирования соответствующих исходных проектов.

Подробности — в файле [`LICENSE`](./LICENSE).
