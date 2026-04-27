# Critical Software Risk Register

A public, maintained dependency risk map for critical open source software infrastructure.

**Version:** v0.1 (seed list — work in progress)  
**Maintained by:** [Critical Software Risk Register](https://criticalsoftwareriskregister.com)  
**Licence:** Data: CC BY 4.0 · Code: MIT

## What this is

This registry tracks the organisational health and dependency risk of open source software components embedded in critical infrastructure: automotive, medical devices, industrial control systems, rail, and enterprise IT.

The primary risk tracked is **availability**: what happens when the maintainer of a component embedded in safety-critical systems stops working.

## How to use it

The canonical data is in `registry/components.yaml`. Maintainer records are in `registry/maintainers.yaml`. The scoring model is in `registry/scoring.yaml`.

All 21 v0.1 entries are in draft status. Scores are set; entries advance to review after named-maintainer notification is complete.

## v0.1 component map

Composite score = sum of four criteria × OT multiplier (1.5 if OT-confirmed).  
Tiers: Critical ≥ 12.01 · High 9.01–12 · Moderate 6.01–9 · Low 0–6.  
See [METHODOLOGY.md](./METHODOLOGY.md) for scoring rules.

| id | name | tier | OT confirmed |
|---|---|---|---|
| libcurl | libcurl | Critical | ✓ |
| samba | Samba | Moderate | ✓ |
| zlib | zlib | Critical | ✓ |
| libxml2 | libxml2 | Critical | ✓ |
| libxslt | libxslt | Moderate | — |
| expat | expat | Critical | ✓ |
| freetype | FreeType | Critical | ✓ |
| ntpsec | NTPsec | High | ✓ |
| libpng | libpng | Critical | ✓ |
| libjpeg-turbo | libjpeg-turbo | Critical | ✓ |
| libmodbus | libmodbus | Critical | ✓ |
| open62541 | open62541 | Moderate | ✓ |
| xz-utils | XZ Utils (liblzma) | Critical | ✓ |
| openssh | OpenSSH (portable) | High | ✓ |
| libpcap | libpcap | Critical | ✓ |
| ncurses | ncurses | Critical | ✓ |
| stunnel | stunnel | Critical | ✓ |
| libgpg-error | libgpg-error | Moderate | — |
| pcre2 | PCRE2 | Moderate | — |
| openldap | OpenLDAP | Critical | ✓ |
| libssh2 | libssh2 | Critical | ✓ |

## Methodology

See [METHODOLOGY.md](./METHODOLOGY.md) for how risk scores are calculated.

## Contributing

See [CONTRIBUTING.md](./CONTRIBUTING.md) for how to submit corrections or additions.

## Citation

See [CITATION.cff](./CITATION.cff) or use:  
Schmidt, Holger. Critical Software Risk Register. criticalsoftwareriskregister.com. 2026.
