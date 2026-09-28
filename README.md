# Mihomo Runet MRS

Automatically generated **Mihomo MRS rule sets** converted from the GeoIP and GeoSite databases published by [RunetFreedom](https://github.com/runetfreedom/russia-v2ray-rules-dat).

This repository tracks the upstream databases, converts all available GeoIP and GeoSite categories into individual `.mrs` files, verifies the generated output, and publishes the results directly to the `main` branch.

The generated rule sets are intended for use with [Mihomo](https://github.com/MetaCubeX/mihomo).

## Features

- Converts all available GeoSite categories to Mihomo MRS
- Converts all available GeoIP categories to Mihomo MRS
- Automatically detects new upstream categories
- Checks for upstream changes every 12 hours
- Rebuilds only when the source data or converter actually changes
- Tracks the exact upstream source snapshot used for each build
- Tracks the exact `meta-rules-converter` commit used for each build
- Verifies GeoDAT files using upstream SHA-256 checksums
- Generates SHA-256 checksums for all published MRS files
- Detects missing or modified generated MRS files
- Automatically skips empty rule sets produced by the converter
- Generates GeoIP and GeoSite category indexes
- Uses UTC for build timestamps and scheduling
- Publishes directly to the `main` branch without force-pushing

## Update schedule

The workflow checks for updates twice a day:

```text
00:00 UTC
12:00 UTC
```

GitHub Actions cron schedules use UTC, so the update times remain fixed throughout the year and are not affected by daylight saving time.

A scheduled check does not necessarily result in a new commit.

The repository is rebuilt only when one of the following conditions is detected:

- `geoip.dat` has changed
- `geosite.dat` has changed
- `meta-rules-converter` has changed
- a generated MRS file is missing or fails its integrity check
- required generated metadata is missing

If nothing has changed, the workflow exits successfully without rebuilding or creating a new commit.

## Source data

The source databases are provided by:

[RunetFreedom / russia-v2ray-rules-dat](https://github.com/runetfreedom/russia-v2ray-rules-dat)

The workflow tracks the upstream `release` branch.

Before downloading the databases, the workflow resolves the current upstream commit and uses that exact commit as a fixed source snapshot.

Both databases are therefore downloaded from the same upstream revision.

### GeoIP

```text
geoip.dat
```

### GeoSite

```text
geosite.dat
```

The upstream SHA-256 checksum files are also used to verify the downloaded databases before conversion.

## Conversion

Conversion is performed using:

[MetaCubeX / meta-rules-converter](https://github.com/MetaCubeX/meta-rules-converter)

The exact converter commit used for the latest build is stored in:

```text
converter.commit
```

The conversion process is conceptually:

```text
geoip.dat
    ↓
meta-rules-converter
    ↓
geoip/*.mrs

geosite.dat
    ↓
meta-rules-converter
    ↓
geosite/*.mrs
```

No category list is maintained manually.

If a new category appears in the upstream GeoDAT files, it is automatically included during the next rebuild.

## Empty rule sets

Some upstream categories or attribute-specific variants contain no rules that can be emitted as an MRS file.

In these cases, `meta-rules-converter` may create an empty output file.

Empty `.mrs` files are not published.

Instead, the workflow:

1. detects zero-byte MRS files;
2. records their names;
3. removes them from the generated output;
4. continues the build with the remaining valid rule sets.

The list of skipped rule sets is stored in:

```text
skipped-empty-rules.txt
```

An empty rule set does not cause the entire build to fail.

The build fails only if no usable GeoSite or GeoIP MRS files remain after validation.

## Repository structure

```text
.
├── .github/
│   └── workflows/
│       └── build.yml
│
├── geoip/
│   ├── private.mrs
│   ├── ru.mrs
│   ├── ru-blocked.mrs
│   ├── ru-whitelist.mrs
│   └── ...
│
├── geosite/
│   ├── private.mrs
│   ├── category-ru.mrs
│   ├── category-ads.mrs
│   ├── github.mrs
│   ├── apple.mrs
│   ├── ru-blocked.mrs
│   └── ...
│
├── categories.md
├── geoip-categories.txt
├── geosite-categories.txt
├── geoip.dat.sha256
├── geosite.dat.sha256
├── source.commit
├── converter.commit
├── mrs-manifest.sha256
├── skipped-empty-rules.txt
├── build-info.txt
├── LICENSE
└── README.md
```

## Using with Mihomo

MRS files can be used directly through Mihomo `rule-providers`.

### GeoSite example

```yaml
rule-providers:
  category-ru:
    type: http
    behavior: domain
    format: mrs
    url: "https://raw.githubusercontent.com/nofantasysorry/mihomo-runet-mrs/main/geosite/category-ru.mrs"
    path: ./ruleset/category-ru.mrs
    interval: 43200
```

Then reference the provider in your rules:

```yaml
rules:
  - RULE-SET,category-ru,ROUTE-RU
```

### GeoIP example

```yaml
rule-providers:
  ru-ip:
    type: http
    behavior: ipcidr
    format: mrs
    url: "https://raw.githubusercontent.com/nofantasysorry/mihomo-runet-mrs/main/geoip/ru.mrs"
    path: ./ruleset/ru.mrs
    interval: 43200
```

Then:

```yaml
rules:
  - RULE-SET,ru-ip,ROUTE-RU,no-resolve
```

## Example: blocked resources

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

Example rules:

```yaml
rules:
  - RULE-SET,ru-blocked-domain,ROUTE-BLOCKED
  - RULE-SET,ru-blocked-ip,ROUTE-BLOCKED,no-resolve
```

## Category indexes

The current list of generated rule sets is available in:

```text
geosite-categories.txt
geoip-categories.txt
categories.md
```

These files are regenerated automatically from the actual conversion output.

Only successfully generated, non-empty MRS files are included.

## Source tracking

The exact upstream source snapshot used for the latest build is stored in:

```text
source.commit
```

The exact converter revision is stored in:

```text
converter.commit
```

This makes each generated build traceable to a specific source-data revision and converter revision.

## Source checksums

The SHA-256 hashes of the GeoDAT files used for the latest build are stored in:

```text
geoip.dat.sha256
geosite.dat.sha256
```

The downloaded source files are also verified against the checksum information published by the upstream repository before conversion begins.

If checksum verification fails, the build stops before any repository files are replaced.

## MRS integrity manifest

Checksums for all published `.mrs` files are stored in:

```text
mrs-manifest.sha256
```

During scheduled checks, the workflow verifies the existing generated files against this manifest.

If a generated MRS file has been modified, corrupted, or removed, the workflow automatically triggers a rebuild even if the upstream databases have not changed.

## Build information

Information about the latest actual rebuild is stored in:

```text
build-info.txt
```

Example:

```text
Built at: 2026-09-28 12:04:15 UTC
Timezone: UTC

Source repository:
https://github.com/runetfreedom/russia-v2ray-rules-dat

Source snapshot:
0123456789abcdef0123456789abcdef01234567

Converter repository:
https://github.com/MetaCubeX/meta-rules-converter

Converter commit:
89abcdef0123456789abcdef0123456789abcdef

GeoSite MRS files: 1888
GeoIP MRS files:   266
Skipped empty MRS: 9

geoip.dat SHA256:   ...
geosite.dat SHA256: ...
```

`build-info.txt` represents the time of the latest **actual rebuild**, not the time of every scheduled check.

If a scheduled run detects no changes, no new commit is created and `build-info.txt` remains unchanged.

## Build safety

The workflow is designed to avoid replacing working rule sets with incomplete output.

Before the generated directories are updated, the workflow verifies that:

- both source GeoDAT files were downloaded successfully;
- their SHA-256 checksums match the upstream values;
- the converter was built from the expected commit;
- GeoSite conversion produced usable MRS files;
- GeoIP conversion produced usable MRS files;
- empty rule sets were removed;
- the final MRS integrity manifest can be verified successfully.

Repository files such as:

```text
README.md
LICENSE
.github/
```

are not replaced by the build process.

The workflow does not use force pushes.

## Notes

The files in this repository are generated automatically from third-party data.

This project does not independently determine whether a domain, IP address, network, service, or resource should be blocked, proxied, bypassed, or routed in any particular way.

The meaning and contents of individual categories are determined by their respective upstream sources.

Some upstream entries may be ignored or rejected by the converter if they cannot be represented in the target MRS format. Converter warnings remain visible in the GitHub Actions build logs.

## Credits

This repository depends on the work of the following projects and their contributors:

- [RunetFreedom / russia-v2ray-rules-dat](https://github.com/runetfreedom/russia-v2ray-rules-dat)
- [RunetFreedom / russia-blocked-geoip](https://github.com/runetfreedom/russia-blocked-geoip)
- [RunetFreedom / russia-blocked-geosite](https://github.com/runetfreedom/russia-blocked-geosite)
- [V2Fly / domain-list-community](https://github.com/v2fly/domain-list-community)
- [MetaCubeX / meta-rules-converter](https://github.com/MetaCubeX/meta-rules-converter)
- [MetaCubeX / mihomo](https://github.com/MetaCubeX/mihomo)

Additional upstream data sources may be included indirectly through the RunetFreedom databases.

Please refer to the respective upstream repositories for complete source information, attribution, and licensing terms.

## License

This repository is distributed under the **GNU General Public License v3.0 (GPL-3.0)**.

Generated rule sets are derived from upstream datasets and remain subject to the licenses and terms of their respective upstream sources.

See [`LICENSE`](./LICENSE) for details.

---

# Русская версия

Автоматически генерируемые **MRS-наборы правил для Mihomo**, преобразованные из баз GeoIP и GeoSite проекта [RunetFreedom](https://github.com/runetfreedom/russia-v2ray-rules-dat).

Репозиторий отслеживает исходные базы, автоматически преобразует все доступные категории GeoIP и GeoSite в отдельные `.mrs`-файлы, проверяет результат и публикует обновления непосредственно в ветку `main`.

Сгенерированные наборы правил предназначены прежде всего для использования с [Mihomo](https://github.com/MetaCubeX/mihomo).

## Возможности

- Конвертация всех доступных категорий GeoSite в Mihomo MRS
- Конвертация всех доступных категорий GeoIP в Mihomo MRS
- Автоматическое обнаружение новых категорий upstream
- Проверка обновлений каждые 12 часов
- Пересборка только при фактическом изменении исходных данных или конвертера
- Сохранение точного commit исходных данных для каждой сборки
- Сохранение точного commit `meta-rules-converter`
- Проверка GeoDAT-файлов по SHA-256 upstream
- Создание SHA-256 для всех опубликованных MRS-файлов
- Обнаружение удалённых или изменённых MRS-файлов
- Автоматический пропуск пустых rule set, создаваемых конвертером
- Автоматическое создание индексов категорий GeoIP и GeoSite
- Использование UTC для расписания и времени сборки
- Публикация непосредственно в ветку `main` без force push

## Расписание обновлений

Workflow проверяет наличие обновлений два раза в сутки:

```text
00:00 UTC
12:00 UTC
```

Расписание остаётся неизменным в течение всего года и не зависит от перехода на летнее или зимнее время.

Запуск workflow по расписанию не обязательно приводит к созданию нового commit.

Пересборка выполняется только в следующих случаях:

- изменился `geoip.dat`;
- изменился `geosite.dat`;
- изменился `meta-rules-converter`;
- отсутствует один из ранее созданных MRS-файлов;
- MRS-файл не проходит проверку целостности;
- отсутствуют необходимые служебные файлы сборки.

Если изменений нет, workflow успешно завершается без пересборки и без создания нового commit.

## Исходные данные

Исходные базы предоставляются проектом:

[RunetFreedom / russia-v2ray-rules-dat](https://github.com/runetfreedom/russia-v2ray-rules-dat)

Workflow отслеживает upstream-ветку `release`.

Перед скачиванием баз определяется точный commit текущего состояния `release`.

После этого обе базы скачиваются именно из этого commit, поэтому `geoip.dat` и `geosite.dat` всегда относятся к одному и тому же снимку upstream.

### GeoIP

```text
geoip.dat
```

### GeoSite

```text
geosite.dat
```

Перед конвертацией скачанные файлы проверяются по SHA-256, опубликованным upstream.

## Конвертация

Для конвертации используется:

[MetaCubeX / meta-rules-converter](https://github.com/MetaCubeX/meta-rules-converter)

Точный commit конвертера, использованный при последней сборке, сохраняется в:

```text
converter.commit
```

Процесс конвертации выглядит следующим образом:

```text
geoip.dat
    ↓
meta-rules-converter
    ↓
geoip/*.mrs

geosite.dat
    ↓
meta-rules-converter
    ↓
geosite/*.mrs
```

Список категорий вручную не поддерживается.

Если upstream добавляет новую категорию в GeoDAT, она автоматически появится в репозитории после следующей пересборки.

## Пустые наборы правил

Некоторые upstream-категории или варианты категорий с атрибутами могут не содержать правил, которые возможно сохранить в MRS.

В таких случаях `meta-rules-converter` может создать пустой файл.

Пустые `.mrs` в репозиторий не публикуются.

Workflow автоматически:

1. находит MRS-файлы размером 0 байт;
2. записывает их имена в отдельный список;
3. удаляет их из результатов конвертации;
4. продолжает сборку с оставшимися корректными MRS.

Список пропущенных категорий хранится в:

```text
skipped-empty-rules.txt
```

Наличие отдельных пустых rule set не приводит к ошибке всей сборки.

Workflow завершится ошибкой только в случае, если после проверки вообще не останется пригодных файлов GeoSite или GeoIP.

## Структура репозитория

```text
.
├── .github/
│   └── workflows/
│       └── build.yml
│
├── geoip/
│   ├── private.mrs
│   ├── ru.mrs
│   ├── ru-blocked.mrs
│   ├── ru-whitelist.mrs
│   └── ...
│
├── geosite/
│   ├── private.mrs
│   ├── category-ru.mrs
│   ├── category-ads.mrs
│   ├── github.mrs
│   ├── apple.mrs
│   ├── ru-blocked.mrs
│   └── ...
│
├── categories.md
├── geoip-categories.txt
├── geosite-categories.txt
├── geoip.dat.sha256
├── geosite.dat.sha256
├── source.commit
├── converter.commit
├── mrs-manifest.sha256
├── skipped-empty-rules.txt
├── build-info.txt
├── LICENSE
└── README.md
```

## Использование с Mihomo

MRS-файлы можно использовать напрямую через `rule-providers` Mihomo.

### Пример GeoSite

```yaml
rule-providers:
  category-ru:
    type: http
    behavior: domain
    format: mrs
    url: "https://raw.githubusercontent.com/nofantasysorry/mihomo-runet-mrs/main/geosite/category-ru.mrs"
    path: ./ruleset/category-ru.mrs
    interval: 43200
```

Использование в правилах:

```yaml
rules:
  - RULE-SET,category-ru,ROUTE-RU
```

### Пример GeoIP

```yaml
rule-providers:
  ru-ip:
    type: http
    behavior: ipcidr
    format: mrs
    url: "https://raw.githubusercontent.com/nofantasysorry/mihomo-runet-mrs/main/geoip/ru.mrs"
    path: ./ruleset/ru.mrs
    interval: 43200
```

Использование:

```yaml
rules:
  - RULE-SET,ru-ip,ROUTE-RU,no-resolve
```

## Пример: заблокированные ресурсы

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

Пример правил:

```yaml
rules:
  - RULE-SET,ru-blocked-domain,ROUTE-BLOCKED
  - RULE-SET,ru-blocked-ip,ROUTE-BLOCKED,no-resolve
```

## Индексы категорий

Актуальный список созданных наборов правил доступен в:

```text
geosite-categories.txt
geoip-categories.txt
categories.md
```

Эти файлы автоматически создаются на основе фактического результата конвертации.

В списки попадают только успешно созданные непустые MRS-файлы.

## Отслеживание исходных версий

Точный commit upstream, использованный для последней сборки, хранится в:

```text
source.commit
```

Точная версия конвертера хранится в:

```text
converter.commit
```

Это позволяет определить, из какой именно версии исходных данных и какой версии конвертера была создана конкретная сборка.

## Контрольные суммы исходных данных

SHA-256 исходных GeoDAT-файлов последней сборки сохраняются в:

```text
geoip.dat.sha256
geosite.dat.sha256
```

Перед началом конвертации скачанные GeoDAT также сверяются с контрольными суммами, опубликованными upstream.

При несовпадении SHA-256 workflow останавливается до замены каких-либо файлов в репозитории.

## Проверка целостности MRS

SHA-256 всех опубликованных `.mrs` хранятся в:

```text
mrs-manifest.sha256
```

Во время плановых запусков workflow проверяет существующие MRS по этому manifest.

Если один из сгенерированных файлов был изменён, повреждён или удалён, будет автоматически запущена полная пересборка, даже если исходные GeoDAT не изменились.

## Информация о сборке

Информация о последней фактической пересборке хранится в:

```text
build-info.txt
```

Пример:

```text
Built at: 2026-09-28 12:04:15 UTC
Timezone: UTC

Source repository:
https://github.com/runetfreedom/russia-v2ray-rules-dat

Source snapshot:
0123456789abcdef0123456789abcdef01234567

Converter repository:
https://github.com/MetaCubeX/meta-rules-converter

Converter commit:
89abcdef0123456789abcdef0123456789abcdef

GeoSite MRS files: 1888
GeoIP MRS files:   266
Skipped empty MRS: 9

geoip.dat SHA256:   ...
geosite.dat SHA256: ...
```

`build-info.txt` содержит время последней **фактической пересборки**, а не каждого запуска workflow.

Если очередная проверка не обнаруживает изменений, новый commit не создаётся и `build-info.txt` остаётся без изменений.

## Безопасность сборки

Workflow спроектирован так, чтобы не заменять рабочие rule set неполным или повреждённым результатом.

Перед обновлением каталогов с MRS проверяется следующее:

- оба GeoDAT-файла успешно скачаны;
- их SHA-256 соответствуют upstream;
- converter получен из ожидаемого commit;
- GeoSite содержит пригодные MRS-файлы;
- GeoIP содержит пригодные MRS-файлы;
- пустые rule set удалены;
- итоговый manifest MRS успешно проходит проверку.

Ручные файлы репозитория, включая:

```text
README.md
LICENSE
.github/
```

workflow не заменяет и не удаляет.

Force push не используется.

## Примечания

Файлы в этом репозитории автоматически создаются из данных сторонних проектов.

Этот проект самостоятельно не определяет, должен ли конкретный домен, IP-адрес, сервис или ресурс блокироваться, проксироваться, обходиться напрямую или маршрутизироваться каким-либо другим способом.

Содержимое и назначение отдельных категорий определяется соответствующими upstream-источниками.

Некоторые upstream-записи могут быть проигнорированы или отклонены конвертером, если их невозможно представить в целевом формате MRS.

Предупреждения converter остаются доступными в логах GitHub Actions.

## Благодарности

Репозиторий использует результаты работы следующих проектов и их участников:

- [RunetFreedom / russia-v2ray-rules-dat](https://github.com/runetfreedom/russia-v2ray-rules-dat)
- [RunetFreedom / russia-blocked-geoip](https://github.com/runetfreedom/russia-blocked-geoip)
- [RunetFreedom / russia-blocked-geosite](https://github.com/runetfreedom/russia-blocked-geosite)
- [V2Fly / domain-list-community](https://github.com/v2fly/domain-list-community)
- [MetaCubeX / meta-rules-converter](https://github.com/MetaCubeX/meta-rules-converter)
- [MetaCubeX / mihomo](https://github.com/MetaCubeX/mihomo)

Дополнительные источники данных могут использоваться косвенно через базы RunetFreedom.

Полную информацию об источниках, авторах и лицензиях следует смотреть в соответствующих upstream-репозиториях.

## Лицензия

Этот репозиторий распространяется под лицензией **GNU General Public License v3.0 (GPL-3.0)**.

Сгенерированные наборы правил являются производными от upstream-данных и также подчиняются условиям лицензирования соответствующих исходных проектов.

Подробности доступны в файле [`LICENSE`](./LICENSE).
