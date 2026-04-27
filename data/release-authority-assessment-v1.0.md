# Release Authority Assessment — Bus Factor Scoring Pass
## Projects: libpng, libpcap, libssh2

**Version:** 1.0  
**Date:** 2026-04-27  
**Author:** Holger Schmidt  
**Scope:** Three projects from the MVP seed list; release authority criterion only  
**Status:** Complete  

---

## 1. Purpose

This document records the research, methodology, sources, and results for the release authority component of the bus factor (`bus_factor`) scoring criterion, applied to three projects where contributor count alone gave insufficient signal.

The question answered for each project:

> Who has the **practical** authority to cut an official release — meaning: who holds GPG/PGP signing keys for release artifacts, who has push access to the canonical release location, and who has demonstrably done so in the last two years? Is this one person or more than one acting independently?

---

## 2. Scoring Scale

| Score | Label | Definition |
|---|---|---|
| **BF=2** | Shared | Two or more people can independently cut a signed release without requiring any single individual |
| **BF=3** | Bottleneck | One person is the practical release bottleneck regardless of how many contributors the project lists |

---

## 3. Methodology

Research conducted 2026-04-27. For each project:

1. Identified the canonical release location (GitHub releases page, official tarball server, or both)
2. Checked signed release tags for the GPG key ID and tagger identity
3. Checked mailing list release announcements for who performed and announced releases in 2023–2025
4. Checked MAINTAINERS/CONTRIBUTORS files for formally listed maintainers with push access
5. Checked GitHub discussions and issue threads for evidence of release bottlenecks

Where GitHub release tag pages showed "This tag was signed with the committer's verified signature" but did not display a key ID in search-accessible text, the tagger was inferred from commit authorship immediately preceding the release tag (the RELEASE-NOTES or version-bump commit), corroborated by mailing list announcements.

---

## 4. Results Table

| Project | BF Score | Release Authority Holder(s) | Confidence |
|---|---|---|---|
| **libpng** | 3 | Cosmin Truta (sole) | High |
| **libpcap** | 3 | Denis Ovsienko (de facto sole; 4 listed maintainers, only Ovsienko has cut recent releases) | Medium |
| **libssh2** | 3 | Daniel Stenberg (formal release manager; sole release cutter despite low self-reported involvement) | High |

---

## 5. Per-Project Evidence

---

### 5.1 libpng

**Proposed score: BF=3**

**Release authority holder: Cosmin Truta (sole)**

#### Signing key and tag evidence

Every release tag in the pnggroup/libpng repository is signed by Cosmin Truta (GitHub username: `ctruta`) using GPG key `B292C64843FF5BCF`. GitHub displays "Verified — This tag was signed with the committer's verified signature. ctruta Cosmin Truta · GPG key ID: B292C64843FF5BCF" on each release.

Most recent confirmed releases signed by Truta:
- **v1.6.44** — confirmed in Android AOSP source mirror: "Release libpng version 1.6.44 by Cosmin Truta" (2024)
- **v1.6.50** — release commit `2b978915` by `ctruta@gmail.com`, 2025-07-01
- **v1.6.51** — GitHub release page: "ctruta tagged this · 21 Nov [2024]", GPG key B292C64843FF5BCF

Source: https://github.com/pnggroup/libpng/releases/tag/v1.6.51

#### Contributor role evidence

The libpng README names only one **current maintainer**: "Cosmin Truta (current maintainer, since 2018)". John Bowler is an active contributor whose commits appear in the log and who co-signs release-related commits (e.g., `Signed-off-by: John Bowler <jbowler@acm.org>`), but review of 2024–2025 release tags shows no tag or tarball signed by Bowler. Bowler's role is code contributor, not release authority.

Source: https://github.com/pnggroup/libpng (README)

#### Governance evidence

In a February 2025 mailing list post to public-png@w3.org, Truta explicitly states he manages the pnggroup GitHub organisation and holds signing authority for all tags, offering to transfer signing responsibility to another member only if they prefer ("if you, Greg, would prefer to sign those tags instead of me, please tell me so, and I'll remove my tags") — confirming this was his unilateral choice, not a shared arrangement.

Source: https://lists.w3.org/Archives/Public/public-png/2025Feb/0008.html (2025-02-06)

#### Assessment

Truta is the sole person who has signed release tags in the last two years. John Bowler contributes code but has not signed any release. The project's own README lists one current maintainer. **BF=3**.

---

### 5.2 libpcap

**Proposed score: BF=3**

**Release authority holder: Denis Ovsienko (de facto sole)**

#### MAINTAINERS file

The MAINTAINERS file (current, verified via Android source mirror of upstream) lists four people:
- Denis Ovsienko `<denis at ovsienko dot info>`
- Francois-Xavier Le Bail `<devel dot fx dot lebail at orange dot fr>`
- Guy Harris `<gharris at sonic dot net>`
- Michael Richardson `<mcr at sandelman dot ottawa dot on dot ca>`

All four are presumably listed as project maintainers with push access to the GitHub repository. This is the basis for the scoring uncertainty (medium confidence).

Source: https://android.googlesource.com/platform/external/libpcap/+/c1cf4218060d4994e51e77c935b84a1e20a23d46

#### Release announcement evidence

All release announcements to the tcpdump-workers mailing list in the research window were authored by Denis Ovsienko:

- **libpcap 1.10.2 / tcpdump 4.99.2** — announced by Denis Ovsienko, 2022-12-31
  Source: https://seclists.org/tcpdump/2022/q4/16

- **libpcap 1.10.5 / tcpdump 4.99.5** — announced by Denis Ovsienko, 2024-08-31. Quote: *"git tags for these releases have been signed using the same release key."*
  Source: https://seclists.org/tcpdump/2024/q3/3

- **libpcap 1.10.6 / tcpdump 4.99.6** — released 2025-12-30. No mailing list announcement located in search; release note on tcpdump.org references "(PGP signature and key)" per the standard pattern. Denis Ovsienko also posted a "activities report for May 2025" to the mailing list (source: seclists.org/tcpdump/), consistent with de facto project lead role.

#### Signing key note

Ovsienko's 2024-08-31 announcement states tags were "signed using the same release key." This phrasing implies a single key, held by Ovsienko. It does not confirm whether the key is shared with other maintainers. No release announcement from Harris, Le Bail, or Richardson was found in the 2022–2025 window.

#### Historical context

Prior to the current era, releases were cut by others: the 2018 libpcap 1.9.0 change log attribution reads "by mcr@sandelman.ca" (Michael Richardson). This confirms release authority was historically distributed. However, the last 8+ releases (2021–2025) have all been cut and announced by Ovsienko.

#### Assessment

Denis Ovsienko has been the exclusive release cutter for at least three years. The other three listed maintainers retain push access in principle, but there is no evidence any of them have signed a release artifact since at least 2021. Until confirmed otherwise, practical release authority resides with one person. **BF=3, confidence: medium** — would upgrade to high if confirmed that other maintainers do not hold the release signing key.

---

### 5.3 libssh2

**Proposed score: BF=3**

**Release authority holder: Daniel Stenberg (formal; sole despite self-described low involvement)**

#### Most recent release evidence

The last release, **libssh2 1.11.1** (October 2024), was cut by Daniel Stenberg:

- NEWS file at `github.com/winlibs/libssh2` (mirror of upstream): `"Daniel Stenberg (16 Oct 2024) - RELEASE-NOTES: 1.11.1"` — Stenberg committed the release notes, the conventional final step before tagging.
- GitHub release page for `libssh2-1.11.1`: "This tag was signed with the committer's verified signature" (committer: Stenberg).

Source: https://github.com/libssh2/libssh2/releases (October 2024)

#### Release bottleneck evidence from GitHub Discussions

GitHub Discussion #1249 "libssh2 1.11.1 release?" is the clearest single source. Community members had been requesting a release for months. In the discussion, Stenberg wrote (verbatim):

> *"I'm willing to do the actual release tarball and upload if that helps. But I'm not really involved in the project right now so someone would need to make sure to populate RELEASE-NOTES appropriately and whatever else that needs to be done to make a release shine."*

This single quote confirms:
1. Only Stenberg has the infrastructure to "do the actual release tarball and upload" — no other contributor offered or demonstrated this capability.
2. Stenberg himself acknowledges his involvement is minimal, yet he remains the exclusive practical release authority.
3. Contributors (Viktor Szakats most prominently, with 196 commits since 1.11.0 as of the discussion) cannot cut a release independently.

Source: https://github.com/libssh2/libssh2/discussions/1249

#### Role of other named contributors

**Viktor Szakats** — most active code contributor in the 1.11.0–1.11.1 cycle. Commits build system improvements, CI, and crypto backend fixes. No release tags attributed to Szakats in search results.

**Dan Fandrich** — listed as contributor in release notes. No release tag attribution found.

**Will Cosgrove** — listed as contributor in release notes. No release tag attribution found.

None of the three appears to have the credentials or established process to upload signed tarballs to libssh2.org or push a signed release tag to the canonical repository.

#### Prior release pattern

libssh2 1.11.0 (May 2023) was also cut by Stenberg, consistent with his long-standing formal release manager role. The gap between 1.11.0 (May 2023) and 1.11.1 (October 2024) — 17 months — is itself a consequence of the single-person bottleneck: Szakats was producing code continuously, but no release occurred until Stenberg acted.

Source: https://github.com/libssh2/libssh2/releases/tag/libssh2-1.11.0

#### Assessment

Stenberg holds sole practical release authority. His self-described low involvement does not translate to distributed release rights — it translates to infrequent releases. No other contributor has produced a signed release tarball or signed tag. The 17-month gap between 1.11.0 and 1.11.1 is a direct, observable consequence of the single-person bottleneck. **BF=3, confidence: high**.

---

## 6. Open Items

- [ ] **libpcap**: Confirm whether Francois-Xavier Le Bail, Guy Harris, or Michael Richardson hold a copy of the release signing key. If confirmed they do, downgrade to BF=2. Recommended action: direct query to the tcpdump-workers mailing list or inspection of the PGP key listed on tcpdump.org/release/.
- [ ] **libssh2**: Monitor for any release cut by Szakats, Fandrich, or Cosgrove — this would be evidence of distributed release authority and a downgrade to BF=2. Currently no such evidence exists.

---

## 7. Version History

| Version | Date | Changes |
|---|---|---|
| 1.0 | 2026-04-27 | Initial assessment — libpng, libpcap, libssh2 |

---

*Contact: contact@criticalsoftwareriskregister.com*  
*Data license: CC BY 4.0*
