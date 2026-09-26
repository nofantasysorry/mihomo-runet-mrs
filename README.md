# Mihomo Runet MRS

Automatically updated **Mihomo MRS rule sets** converted from the GeoIP and GeoSite databases published by [RunetFreedom](https://github.com/runetfreedom/russia-v2ray-rules-dat).

The repository automatically downloads the latest `geoip.dat` and `geosite.dat`, converts every available category into an individual `.mrs` rule set, and publishes the results directly to the `release` branch.

The generated files are intended primarily for use with [Mihomo](https://github.com/MetaCubeX/mihomo).

## Features

- Automatic conversion of all GeoIP categories to Mihomo MRS
- Automatic conversion of all GeoSite categories to Mihomo MRS
- No manually maintained category list
- Automatically picks up newly added upstream categories
- Updated every 6 hours
- Build timestamps use the `America/New_York` timezone
- SHA-256 hashes of the source GeoDAT files are published with every build
- Separate GeoIP and GeoSite category indexes
- Direct raw GitHub URLs suitable for Mihomo `rule-providers`

## Repository structure

```text
.
├── .github/
│   └── workflows/
│       └── build.yml
├── geoip/
│   ├── private.mrs
│   ├── ru.mrs
│   ├── ru-blocked.mrs
│   ├── ru-whitelist.mrs
│   └── ...
├── geosite/
│   ├── private.mrs
│   ├── category-ru.mrs
│   ├── category-ads.mrs
│   ├── github.mrs
│   ├── apple.mrs
│   ├── ru-blocked.mrs
│   └── ...
├── geoip-categories.txt
├── geosite-categories.txt
├── categories.md
├── geoip.dat.sha256
├── geosite.dat.sha256
└── build-info.txt
```

## Source data

The source databases are downloaded from:

### GeoIP

```text
https://raw.githubusercontent.com/runetfreedom/russia-v2ray-rules-dat/release/geoip.dat
```

### GeoSite

```text
https://raw.githubusercontent.com/runetfreedom/russia-v2ray-rules-dat/release/geosite.dat
```

Upstream repository:

- [runetfreedom/russia-v2ray-rules-dat](https://github.com/runetfreedom/russia-v2ray-rules-dat)

The RunetFreedom databases include additional Russian routing categories as well as data originating from other upstream projects.

## Conversion

Conversion is performed using:

- [MetaCubeX/meta-rules-converter](https://github.com/MetaCubeX/meta-rules-converter)

The workflow automatically processes the complete GeoDAT files. Categories do not need to be listed manually.

Conceptually:

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

If a new category is added to the upstream GeoDAT files, it will automatically appear in this repository after the next successful build.

## Using with Mihomo

MRS files can be used through Mihomo `rule-providers`.

### GeoSite example

```yaml
rule-providers:
  category-ru:
    type: http
    behavior: domain
    format: mrs
    url: "https://raw.githubusercontent.com/nofantasysorry/mihomo-runet-mrs/release/geosite/category-ru.mrs"
    path: ./ruleset/category-ru.mrs
    interval: 21600
```

Then use it in `rules`:

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
    url: "https://raw.githubusercontent.com/nofantasysorry/mihomo-runet-mrs/release/geoip/ru.mrs"
    path: ./ruleset/ru.mrs
    interval: 21600
```

And:

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
    url: "https://raw.githubusercontent.com/nofantasysorry/mihomo-runet-mrs/release/geosite/ru-blocked.mrs"
    path: ./ruleset/ru-blocked-domain.mrs
    interval: 21600

  ru-blocked-ip:
    type: http
    behavior: ipcidr
    format: mrs
    url: "https://raw.githubusercontent.com/nofantasysorry/mihomo-runet-mrs/release/geoip/ru-blocked.mrs"
    path: ./ruleset/ru-blocked-ip.mrs
    interval: 21600
```

```yaml
rules:
  - RULE-SET,ru-blocked-domain,ROUTE-BLOCKED
  - RULE-SET,ru-blocked-ip,ROUTE-BLOCKED,no-resolve
```

## Category lists

The repository automatically generates:

```text
geosite-categories.txt
geoip-categories.txt
categories.md
```

These files contain the categories available in the current build.

They are generated directly from the converter output, so they always reflect the actual contents of the repository.

## Updates

The GitHub Actions workflow runs every 6 hours.

Build information is written to:

```text
build-info.txt
```

Example:

```text
Built at: 2026-09-26 09:23:00 EDT
Timezone: America/New_York
```

The `America/New_York` timezone is used, so timestamps automatically switch between `EDT` and `EST` when daylight saving time changes.

## Integrity

SHA-256 hashes of the source GeoDAT files used for each build are available in:

```text
geoip.dat.sha256
geosite.dat.sha256
```

These hashes identify the exact upstream databases used to generate the current MRS files.

## Important notes

The files in this repository are automatically generated from third-party data.

This project does not independently determine whether a domain, IP address, network, service, or resource should be blocked, proxied, or routed in any particular way.

The meaning and contents of individual categories are determined by their respective upstream data sources.

GeoSite data may contain different domain matching types and attributes. Conversion to Mihomo MRS is performed by `meta-rules-converter`; therefore the generated files follow the behavior and limitations of that converter and the Mihomo MRS format.

## Credits

This repository would not be possible without the following projects and their contributors:

- [RunetFreedom / russia-v2ray-rules-dat](https://github.com/runetfreedom/russia-v2ray-rules-dat)
- [RunetFreedom / russia-blocked-geoip](https://github.com/runetfreedom/russia-blocked-geoip)
- [RunetFreedom / russia-blocked-geosite](https://github.com/runetfreedom/russia-blocked-geosite)
- [V2Fly / domain-list-community](https://github.com/v2fly/domain-list-community)
- [MetaCubeX / meta-rules-converter](https://github.com/MetaCubeX/meta-rules-converter)
- [MetaCubeX / mihomo](https://github.com/MetaCubeX/mihomo)

Additional upstream data sources may be included indirectly through the RunetFreedom databases.

Please refer to the upstream repositories for their full attribution, source information, and licensing terms.

## License

This repository is distributed under the **GNU General Public License v3.0 (GPL-3.0)**.

The generated rule sets are derived from upstream datasets and remain subject to the licenses and terms of their respective upstream sources.

See [`LICENSE`](./LICENSE) for details.
