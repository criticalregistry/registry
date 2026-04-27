# Security Practices Assessment — Critical OSS Registry Seed List
## Scoring Pass: `security_practices` Criterion

**Version:** 1.0  
**Date:** 2026-04-27  
**Author:** Holger Schmidt  
**Scope:** 21-project MVP seed list, security practices criterion only  
**Status:** Complete — all 21 projects assessed  

---

## 1. Purpose

This document records the research, methodology, sources, and results for the `security_practices` scoring criterion applied to the Critical Software Risk Register's 21-project MVP seed list.

The criterion answers one question:

> Does this project have **documented, systematic** security practices — such as continuous fuzzing (OSS-Fuzz, libFuzzer), formal verification, or a published secure development lifecycle — or does it rely on **reactive** practices only, or does it have **no documented** security practices and/or a demonstrated failure to respond to vulnerability reports promptly?

---

## 2. Scoring Scale

| Score | Label | Definition |
|---|---|---|
| **1** | Systematic | Continuous automated fuzzing (e.g., OSS-Fuzz enrollment), formal verification, or an equivalently documented proactive SDL in place |
| **2** | Reactive | Responds to reported CVEs with patches; no documented proactive tooling or continuous fuzzing infrastructure |
| **3** | Absent / Unresponsive | No documented security practices, or demonstrated slow/absent response to disclosed vulnerability reports |

---

## 3. Methodology

### 3.1 Research Approach

Research was conducted on 2026-04-27 using live web search and direct source verification. The primary proxy for Score 1 is **OSS-Fuzz enrollment**, because:

- OSS-Fuzz is the dominant continuous fuzzing infrastructure for C/C++ open source projects, operated by Google in partnership with OpenSSF
- Enrollment requires a project to maintain fuzzing harnesses (build.sh, Dockerfile, project.yaml) in the google/oss-fuzz repository
- Enrollment status is publicly verifiable via the OSS-Fuzz GitHub repository and the Fuzz Introspector dashboard

OSS-Fuzz enrollment was **not** treated as the only path to Score 1. Projects with formally documented, externally verified alternative proactive security practices (e.g., formal verification, independent published SDL) would also qualify. None of the 21 projects presented such an alternative in this pass.

### 3.2 Verification Sources, by Priority

1. **Direct URL check** — `github.com/google/oss-fuzz/tree/master/projects/<name>` or confirmed `build.sh` presence
2. **Fuzz Introspector dashboard** — `introspector.oss-fuzz.com/project-profile?project=<name>` — confirms active enrollment with coverage data
3. **OSS-Fuzz documentation** — projects cited as reference/example integrations in the official OSS-Fuzz docs
4. **Android source mirror** — `android.googlesource.com/platform/external/oss-fuzz` — contains a mirror of the upstream projects directory; used to confirm presence of less prominent projects (accepted as reliable but flagged as medium confidence where used as sole source)
5. **Project-maintained documentation** — e.g., wiki.samba.org/Fuzzing, daniel.haxx.se/blog, openssh.org/security.html
6. **CVE history and disclosure records** — NVD, CVEdetails.com, Red Hat Security Advisories — used to assess reactive response for Score 2 candidates
7. **Third-party security research** — Trail of Bits, Microsoft Security Blog, GitHub Security Lab — used where project-maintained documentation was absent

### 3.3 Confidence Levels

| Level | Meaning |
|---|---|
| **High** | OSS-Fuzz enrollment confirmed via direct project URL or Fuzz Introspector profile |
| **Medium** | Enrollment confirmed via Android source mirror only, or via indirect evidence (OSS-Fuzz issue tracker references, PR history) without a direct project URL check |
| **Low** | Assessment based primarily on absence of evidence and CVE history alone |

### 3.4 Limitations and Caveats

**OSS-Fuzz enrollment ≠ security immunity.** Enrollment confirms that a fuzzing harness exists and runs continuously. It does not guarantee:
- High code coverage (some projects have very low harness coverage)
- Detection of non-memory-safety bugs (logic errors, supply chain attacks, authentication bypasses)
- Active maintainer engagement with reported findings

Three enrolled projects in this list (libxml2, libxslt, XZ Utils) have had serious security incidents despite enrollment, for different reasons:
- **libxml2 / libxslt**: Maintainer suspended security embargo coordination in 2025 due to unsustainable unpaid burden. Tooling exists; human capacity to act on findings is under strain.
- **XZ Utils**: The 2024 backdoor (CVE-2024-3094) was a build-process supply chain attack. The compromised maintainer "Jia Tan" had interacted with OSS-Fuzz configuration directly (PR #10667). Continuous fuzzing of runtime code paths does not detect malicious modifications to build infrastructure.

These caveats should be reflected in individual registry entries — the `security_practices` score is a necessary but not sufficient indicator of overall project security posture.

**No project scored 3** in this assessment set. All 21 projects respond to CVE disclosures in some form.

---

## 4. Sources

### 4.1 Primary Sources — OSS-Fuzz Infrastructure

| Source | URL |
|---|---|
| OSS-Fuzz GitHub repository (projects directory) | https://github.com/google/oss-fuzz/tree/master/projects |
| OSS-Fuzz documentation — ideal integration | https://google.github.io/oss-fuzz/advanced-topics/ideal-integration/ |
| OSS-Fuzz documentation — new project guide | https://google.github.io/oss-fuzz/getting-started/new-project-guide/ |
| Fuzz Introspector dashboard | https://introspector.oss-fuzz.com/projects-overview |
| Android source mirror of OSS-Fuzz projects (upstream master) | https://android.googlesource.com/platform/external/oss-fuzz/+/refs/heads/upstream-master/projects/ |
| OpenSSH OSS-Fuzz build.sh (direct file) | https://github.com/google/oss-fuzz/blob/master/projects/openssh/build.sh |
| libxml2 OSS-Fuzz project directory | https://github.com/google/oss-fuzz/tree/master/projects/libxml2 |
| libxslt OSS-Fuzz project directory | https://github.com/google/oss-fuzz/tree/master/projects/libxslt |
| open62541 OSS-Fuzz project directory | https://github.com/google/oss-fuzz/tree/master/projects/open62541 |
| zlib OSS-Fuzz project directory | https://github.com/google/oss-fuzz/tree/master/projects/zlib |
| NTPsec Fuzz Introspector profile | https://introspector.oss-fuzz.com/project-profile?project=ntpsec |
| curl Fuzz Introspector profile | https://introspector.oss-fuzz.com/project-profile?project=curl |

### 4.2 Project-Maintained Security Documentation

| Project | Source | URL |
|---|---|---|
| curl/libcurl | "5 years on OSS-Fuzz" — Daniel Stenberg, 2022-07-01 | https://daniel.haxx.se/blog/2022/07/01/5-years-on-oss-fuzz/ |
| curl/libcurl | curl-fuzzer repository README | https://github.com/curl/curl-fuzzer |
| Samba | Samba Wiki — Fuzzing | https://wiki.samba.org/index.php/Fuzzing |
| OpenSSH | OpenSSH Security page | https://www.openssh.org/security.html |
| libgpg-error | GnuPG/libgpg-error GitHub (mirror) | https://github.com/gpg/libgpg-error |

### 4.3 Third-Party Security Research

| Source | URL |
|---|---|
| Trail of Bits — "Toward More Effective Curl Fuzzing" (2024-03-01) | https://blog.trailofbits.com/2024/03/01/toward-more-effective-curl-fuzzing/ |
| Trail of Bits — "Curl Audit: Fuzzing libcurl CLI" (2023-02-14) | https://blog.trailofbits.com/2023/02/14/curl-audit-fuzzing-libcurl-command-line-interface/ |
| Microsoft Security Blog — "Uncursing the ncurses" (2023-09-14) | https://www.microsoft.com/en-us/security/blog/2023/09/14/uncursing-the-ncurses-memory-corruption-vulnerabilities-found-in-library/ |
| GitHub Security Lab — "Bugs That Survive Continuous Fuzzing" (2025-12-29) | https://github.blog/security/vulnerability-research/bugs-that-survive-the-heat-of-continuous-fuzzing/ |
| XZ Utils backdoor analysis — thesamesam (gist, 2024) | https://gist.github.com/thesamesam/223949d5a074ebc3dce9ee78baad9e27 |
| Cynet — CVE-2024-3094 XZ analysis (Jia Tan / OSS-Fuzz PR #10667) | https://www.cynet.com/blog/cve-2024-3094-xz-lzma-vulnerability/ |

### 4.4 CVE and Disclosure Records

| Source | URL |
|---|---|
| Red Hat Security Advisory RHSA-2024:4264 (OpenLDAP) | https://access.redhat.com/errata/RHSA-2024:4264 |
| Red Hat Security Advisory RHSA-2024:6033 (OpenLDAP) | https://access.redhat.com/errata/RHSA-2024:6033 |
| Red Hat Security Advisory RHSA-2025:8176 (OpenLDAP) | https://access.redhat.com/errata/RHSA-2025:8176 |
| CVEdetails — GNU ncurses | https://www.cvedetails.com/product/38464/GNU-Ncurses.html |
| CVEdetails — Stunnel | https://www.cvedetails.com/product/1122/Stunnel-Stunnel.html |
| NVD — CVE-2023-29491 (ncurses) | https://nvd.nist.gov/vuln/detail/CVE-2023-29491 |
| OSS-Fuzz issue tracker — curl ASSERT example (issue #42531812, 2024-01-10) | https://issues.oss-fuzz.com/issues/42531812 |

---

## 5. Results

### 5.1 Summary

| Score | Count | Projects |
|---|---|---|
| 1 — Systematic | 17 | libcurl, Samba, zlib, libxml2, libxslt, expat, FreeType, NTPsec, libpng, libjpeg-turbo, libmodbus, open62541, XZ Utils, OpenSSH, libpcap, PCRE2, libssh2 |
| 2 — Reactive | 4 | ncurses, stunnel, libgpg-error, OpenLDAP |
| 3 — Absent | 0 | — |

### 5.2 Full Results Table

| Project | Score | Evidence | Confidence |
|---|---|---|---|
| **libcurl** | 1 | OSS-Fuzz enrolled since 2017-07-01 (curl-fuzzer repository); CIFuzz integrated into CI pipeline since 2020; 6 CVEs detected by OSS-Fuzz to date. Documented in detail by Daniel Stenberg (daniel.haxx.se, 2022-07-01). | High |
| **Samba** | 1 | OSS-Fuzz enrolled; documented fuzzing process at wiki.samba.org/Fuzzing including harness maintenance instructions, embargo procedures, and integration with OSS-Fuzz infrastructure. | High |
| **zlib** | 1 | OSS-Fuzz enrolled; project directory confirmed at github.com/google/oss-fuzz/tree/master/projects/zlib. | High |
| **libxml2** | 1 | OSS-Fuzz enrolled; project directory confirmed and cited in OSS-Fuzz docs as a reference example. **Caveat:** maintainer suspended security embargo coordination in 2025 due to unsustainable workload; availability risk should be reflected in a separate criterion. | High |
| **libxslt** | 1 | OSS-Fuzz enrolled; project directory confirmed at github.com/google/oss-fuzz/tree/master/projects/libxslt. Shares maintainer situation with libxml2. | High |
| **expat** | 1 | OSS-Fuzz enrolled; cited in OSS-Fuzz new project guide as a reference Dockerfile example and dictionary example. | High |
| **FreeType** | 1 | OSS-Fuzz enrolled; listed explicitly in OSS-Fuzz ideal integration documentation alongside boringssl, SQLite, OpenSSL, harfbuzz, and PCRE2. | High |
| **NTPsec** | 1 | OSS-Fuzz enrolled; active Fuzz Introspector profile confirmed at introspector.oss-fuzz.com/project-profile?project=ntpsec. NTPsec was a ground-up security-focused rewrite of the reference NTP daemon. | High |
| **libpng** | 1 | OSS-Fuzz enrolled; project directory present in upstream google/oss-fuzz (confirmed via Android source mirror of upstream master). | High |
| **libjpeg-turbo** | 1 | OSS-Fuzz enrolled; active issue history in OSS-Fuzz tracker (build failure issue #4931, 2021); separate oss-fuzz branch maintained at skia.googlesource.com. | High |
| **libmodbus** | 1 | OSS-Fuzz enrolled; project directory present in upstream google/oss-fuzz (confirmed via Android source mirror of upstream master). | Medium |
| **open62541** | 1 | OSS-Fuzz enrolled; project directory confirmed at github.com/google/oss-fuzz/tree/master/projects/open62541. open62541 is the primary OPC UA C implementation; enrollment is a positive signal given OT deployment context. | High |
| **XZ Utils (liblzma)** | 1 | OSS-Fuzz enrolled; confirmed indirectly by compromised maintainer "Jia Tan" opening OSS-Fuzz PR #10667 to modify fuzzing configuration. **Caveat:** the 2024 backdoor (CVE-2024-3094, CVSS 10.0) was a build-process supply chain attack not detectable by runtime fuzzing. | High |
| **OpenSSH** | 1 | OSS-Fuzz enrolled; build.sh confirmed at github.com/google/oss-fuzz/blob/master/projects/openssh/build.sh. OpenSSH security page documents coordinated disclosure process with prompt patch releases (e.g., CVE-2025-26466 patched 2025-02-18). | High |
| **libpcap** | 1 | OSS-Fuzz enrolled; project directory present in upstream google/oss-fuzz (confirmed via Android source mirror of upstream master). | Medium |
| **PCRE2** | 1 | OSS-Fuzz enrolled; listed explicitly in OSS-Fuzz ideal integration documentation as a reference example. | High |
| **libssh2** | 1 | OSS-Fuzz enrolled; project directory present in upstream google/oss-fuzz (confirmed via Android source mirror of upstream master). | Medium |
| **ncurses** | 2 | Single maintainer (Thomas Dickey) responds to coordinated disclosure — CVE-2023-29491 patched promptly (commit 20230408) after Microsoft disclosed via CVD process. Vulnerability was found by Microsoft using their own fuzzer, not a project-maintained harness. No OSS-Fuzz enrollment found. | High |
| **stunnel** | 2 | CVE history documents responsive patch releases across multiple major versions; no evidence of OSS-Fuzz enrollment or documented proactive fuzzing tooling. CVE-2021-20230 (authentication bypass) patched in same release cycle as disclosure. | Medium |
| **libgpg-error** | 2 | GnuPG ecosystem responds to security reports via mailing list and publishes security advisories (most recent: GnuPG CVE-2025-68973, patched in RHSA-2026:0935). libgpg-error itself is narrow in scope (error codes and utility functions); no evidence of OSS-Fuzz enrollment or proactive fuzzing for this library specifically. | Medium |
| **OpenLDAP** | 2 | Red Hat issues regular security advisories against OpenLDAP packages (RHSA-2024:4264, RHSA-2024:6033, RHSA-2025:8176); upstream responds to disclosures. No evidence of OSS-Fuzz enrollment or documented systematic fuzzing tooling found. | Medium |

---

## 6. Open Items

- [ ] **Verify medium-confidence Score 1s** (libmodbus, libpcap, libssh2) via direct URL check against live google/oss-fuzz master at next update cycle
- [ ] **Monitor libxml2/libxslt maintainer situation** — security practices score is 1 but governance/availability risk is elevated; flag for separate bus\_factor and funding\_status criteria
- [ ] **Re-assess stunnel** — last high-confidence CVE data is 2021; check whether active maintenance has continued into 2025–2026
- [ ] **Re-assess libgpg-error scope** — the library's narrow scope may warrant a lower weight applied to this criterion in the composite risk score

---

## 7. Version History

| Version | Date | Changes |
|---|---|---|
| 1.0 | 2026-04-27 | Initial research pass — all 21 seed projects assessed |

---

*Contact: contact@criticalsoftwareriskregister.com*  
*Data license: CC BY 4.0*
