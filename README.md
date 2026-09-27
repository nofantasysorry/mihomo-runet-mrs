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
- Uses New York local time with automatic EDT/EST handling
- Publishes directly to the `main` branch without force-pushing

## Update schedule

The workflow checks for updates twice a day:

```text
00:00 America/New_York
12:00 America/New_York
```

The `America/New_York` timezone is used, so GitHub Actions automatically handles the transition between **EDT** and **EST**.

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

This makes each generated build reproducible against a specific source-data revision and converter revision.

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
Built at: 2026-09-27 12:04:15 EDT
Timezone: America/New_York

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
