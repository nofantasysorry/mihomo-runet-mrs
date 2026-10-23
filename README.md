# mihomo-runet-mrs — MRS-правила для Mihomo из баз RunetFreedom

[![Convert Runet dat to MRS](https://github.com/nofantasysorry/mihomo-runet-mrs/actions/workflows/build.yml/badge.svg)](https://github.com/nofantasysorry/mihomo-runet-mrs/actions/workflows/build.yml)

[English version](#english-version)

---

## Что это

Автоматически обновляемый репозиторий **MRS-наборов правил** для [Mihomo](https://github.com/MetaCubeX/mihomo) (Clash Meta) и построенных на нём клиентов: Clash Verge Rev, FlClash, Mihomo Party, Hakuzzi, Box для Android и другие.

Дважды в сутки workflow GitHub Actions скачивает актуальные `geoip.dat` и `geosite.dat` из ветки `release` проекта [RunetFreedom / russia-v2ray-rules-dat](https://github.com/runetfreedom/russia-v2ray-rules-dat), конвертирует каждую категорию в отдельный `.mrs`-файл и публикует результат прямо в этот репозиторий. От вас не требуется ничего — просто укажите нужные URL в конфиге.

**Кому полезно:**

- вы хотите, чтобы российские сайты и сервисы шли **напрямую**, а всё остальное — через прокси (или наоборот);
- вы не хотите собирать и обновлять наборы правил вручную из больших dat-баз;
- ваш клиент поддерживает формат `.mrs` (любое ядро Mihomo). Старый Clash без MRS-поддержки не сможет использовать эти файлы.

## Как это работает

1. **01:37 и 13:37 UTC** срабатывает расписание (два запуска в сутки; минута 37 выбрана для обхода пиковых задержек очереди GitHub Actions в начале часа).
2. Workflow скачивает свежие `geoip.dat` / `geosite.dat` с ветки `release` RunetFreedom, собирает конвертер [MetaCubeX / meta-rules-converter](https://github.com/MetaCubeX/meta-rules-converter) и конвертирует все категории в `.mrs`.
3. Пустые категории (те, что не содержат правил) отбрасываются — их список виден в [`skipped-empty-rules.txt`](skipped-empty-rules.txt).
4. Результат публикуется коммитом в `main` — **только если содержимое действительно изменилось**; если upstream не обновлялся, пустой коммит не создаётся.
5. Ваш Mihomo сам подхватывает обновления по `interval` из конфига провайдеров.

Последний прогон: время сборки, коммиты upstream и конвертера, хэши исходных dat-файлов — всё в [`build-info.txt`](build-info.txt). Список всех доступных наборов — в [`categories.md`](categories.md) и [`geosite-categories.txt`](geosite-categories.txt) / [`geoip-categories.txt`](geoip-categories.txt).

## Быстрый старт

В конфиг Mihomo (`config.yaml`) добавьте rule-providers и подключите их в правила:

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

rules:
  - RULE-SET,geosite-ru,DIRECT
  - RULE-SET,geoip-ru,DIRECT,no-resolve
  - MATCH,PROXY
```

- `interval: 43200` — перепроверка каждые 12 часов, обновления применяются без перезапуска.
- `MATCH,PROXY` в конце отправляет всё, что не подошло выше, в вашу прокси-группу.
- Хотите наоборот — замените `DIRECT` на имя своей группы.

## Примеры

### Заблокированные ресурсы через прокси, остальное не трогаем

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

rules:
  - RULE-SET,ru-blocked-domain,PROXY
  - RULE-SET,ru-blocked-ip,PROXY,no-resolve
  - MATCH,DIRECT
```

### Российские сайты напрямую + заблокированные через прокси

```yaml
rules:
  - RULE-SET,ru-blocked-domain,PROXY
  - RULE-SET,ru-blocked-ip,PROXY,no-resolve
  - RULE-SET,geosite-ru,DIRECT
  - RULE-SET,geoip-ru,DIRECT,no-resolve
  - MATCH,PROXY
```

Порядок важен: правила проверяются сверху вниз, поэтому узкие правила (заблокированные) должны стоять выше широких («всё российское — напрямую»).

### Все доступные категории

В репозитории около двух тысяч наборов — не только про Россию. Любой файл из [`geosite/`](geosite/) и [`geoip/`](geoip/) подключается так же: поменяйте URL, `behavior` (`domain` для geosite, `ipcidr` для geoip) и имя файла. Полный список — [`categories.md`](categories.md).

## Надёжность

- Каждый опубликованный файл имеет SHA-256 в [`mrs-manifest.sha256`](mrs-manifest.sha256) — манифест пересоздаётся и самопроверяется при каждой сборке.
- Коммит создаётся только при реальном изменении содержимого; в неизменённых запусках ничего не пушится.
- Хэши использованных `geoip.dat` / `geosite.dat` и коммит upstream-снимка хранятся в корне репозитория (`geoip.dat.sha256`, `geosite.dat.sha256`, `source.commit`, `converter.commit`) — как информация о происхождении данных, а не как условие сборки.
- Принудительная проверка обновлений: вкладка **Actions** → **Convert Runet dat to MRS** → **Run workflow**.

## Источники

- [RunetFreedom / russia-v2ray-rules-dat](https://github.com/runetfreedom/russia-v2ray-rules-dat) — исходные базы GeoIP/GeoSite
- [MetaCubeX / meta-rules-converter](https://github.com/MetaCubeX/meta-rules-converter) — инструмент конвертации
- [MetaCubeX / mihomo](https://github.com/MetaCubeX/mihomo) — ядро прокси и формат MRS
- [V2Fly / domain-list-community](https://github.com/v2fly/domain-list-community) и [RunetFreedom / russia-blocked-geosite](https://github.com/runetfreedom/russia-blocked-geosite), [RunetFreedom / russia-blocked-geoip](https://github.com/runetfreedom/russia-blocked-geoip) — данные, входящие в базы RunetFreedom опосредованно

Содержимое категорий полностью определяется исходными проектами; этот репозиторий не решает, что блокировать, а что проксировать.

## Лицензия

Распространяется по **GNU General Public License v3.0 (GPL-3.0)** — см. [`LICENSE`](LICENSE). Сгенерированные наборы являются производными от upstream-данных и подчиняются лицензиям их источников.

## Авторские права

- © 2026 [nofantasysorry](https://github.com/nofantasysorry) — владелец и мейнтейнер репозитория
- Автоматизация, переработка CI и документация — [GPT-6 Astra](https://openai.com) (OpenAI)

---

# English version

Auto-updated **MRS rule sets** for [Mihomo](https://github.com/MetaCubeX/mihomo) (Clash Meta) and clients built on it. Twice a day — at 01:37 and 13:37 UTC — a GitHub Actions workflow downloads the current `geoip.dat` and `geosite.dat` from the `release` branch of [RunetFreedom / russia-v2ray-rules-dat](https://github.com/runetfreedom/russia-v2ray-rules-dat), converts every category into a separate `.mrs` file with [MetaCubeX / meta-rules-converter](https://github.com/MetaCubeX/meta-rules-converter), and publishes the result to this repository. A commit is created only when the generated content actually changes.

Point your `rule-providers` at this repository (see the examples above; swap `DIRECT`/`PROXY` to match your routing policy). Each client re-checks the files on its own `interval` — no restarts, no manual work. The full catalogue of available sets is in [`categories.md`](categories.md); hashes of every published file are in [`mrs-manifest.sha256`](mrs-manifest.sha256); build provenance is in [`build-info.txt`](build-info.txt).

Licensed under **GPL-3.0**. © 2026 [nofantasysorry](https://github.com/nofantasysorry) — owner and maintainer; automation, CI redesign and documentation by [GPT-6 Astra](https://openai.com) (OpenAI).
