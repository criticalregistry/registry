# Methodology
## Critical Software Risk Register — v0.1

---

### 1. Purpose

The Critical Software Risk Register is a public, maintained, machine-readable map of open source components whose organisational health presents risk to the infrastructure depending on them. The primary risk the registry tracks is **availability**: what happens when the person or small team maintaining a component stops — through burnout, illness, funding loss, or death — before a successor is in place. Vulnerability counts are a secondary concern and are tracked by other initiatives. This registry addresses the question those initiatives do not: is there anyone left to fix it, and for how long?

The registry is not a vulnerability scanner, a compliance checklist, a certification body, or a shaming exercise. It is a risk data asset designed to make invisible supplier relationships visible to the organisations that carry them.

---

### 2. Scope

The v0.1 registry covers 21 manually curated components. A component is included when it meets all three of the following criteria:

- It is widely deployed in production infrastructure at a scale where failure would have broad systemic consequences.
- Its maintainer base is small relative to its deployment footprint, creating meaningful bus factor risk.
- Its source code is publicly available and its maintenance activity is publicly observable.

Components with well-resourced institutional backing are excluded — not because they are risk-free, but because the marginal value of this registry's scrutiny is lower where funding and succession are already managed. Excluded from v0.1 on this basis: BIND9 (ISC commercial model), SQLite (small professional team, structured governance), Mosquitto (Cedalo GmbH commercial backer), OpenBSD LibreSSL (Foundation-backed, distributed team). A component may be re-evaluated for inclusion if its funding or maintainer situation changes materially.

The scope is limited to software components. Firmware and hardware supply chains, where SBOM data is not yet available at scale, are not assessed directly. Where known hardware or OT embeddings are documented in public sources, they are recorded in the `ot_confirmation_source` field.

---

### 3. Data Sources

**Layer 1 — Algorithmic baseline.** The OpenSSF Criticality Score provides a quantitative starting point, ranking projects by dependency centrality based on contributor count, commit frequency, dependent project count, and release history. This layer is largely automatable and updates continuously from public repository data. Its limitation is that it is GitHub-centric: projects maintained via mailing lists, hosted on kernel.org, or developed outside the pull-request workflow may score poorly regardless of actual criticality. For such projects, the human intelligence layer carries proportionally more weight in the overall assessment.

**Layer 2 — Dependency graph.** The dependency footprint of each component is constructed from package manager dependency data (npm, PyPI, Maven, cargo, apt, and others), published SBOM data in SPDX and CycloneDX formats from manufacturers and system integrators, and known hardware and firmware embeddings documented in vendor disclosures, CVE records from NVD, and CISA ICS-CERT advisories. The OT layer — what is embedded in vehicle firmware, medical device software, and industrial controllers — is the least complete and the most important. It is built incrementally from public disclosure data, CISA KEV entries, and direct engagement with manufacturers where possible.

**Layer 3 — Human intelligence.** Maintainer data cannot be derived algorithmically. Who actually maintains a project in practice, what their bus factor is, what their employment and financial situation is, and what succession plans exist requires human research: reading mailing lists, reviewing financial disclosures where available, and in some cases direct engagement with the maintainers themselves. This layer is the differentiator that makes the map credible rather than merely large. It is also the layer most subject to staleness; update triggers are defined in Section 8.

---

### 4. Scoring Model

Each component is assessed on four criteria, each scored on a scale of 1 to 3 where 3 is highest risk. The criteria, their scales, and the current weights are defined in `registry/scoring.yaml`. That file is the authoritative source; what follows is a plain-language explanation of the v1 model.

**Ecosystem reach** measures how broadly the component is deployed. A score of 1 indicates a significant but bounded footprint. A score of 2 means the component is present in most major Linux distributions or platforms at scale. A score of 3 means billions of installations, or confirmed embedding in OT, medical, or automotive systems at scale.

**Bus factor** measures how many people must be simultaneously unavailable to halt the project — unable to merge security patches, cut releases, or respond to critical issues. A score of 1 means bus factor of 3 or more. A score of 2 means bus factor of 2. A score of 3 means a single person holds the authority, institutional knowledge, or commit rights that keep the project functional.

**Language risk** measures the security properties of the implementation language and the project's security practices. A score of 1 means a memory-safe language, or C/C++ with systematic fuzzing or formal verification. A score of 2 means C/C++ with reactive security practices — responding to reported vulnerabilities without systematic proactive tooling. A score of 3 means C/C++ with no documented security practices or no demonstrated active response to vulnerability reports.

**Funding status** measures the sustainability of the project's current resource base. A score of 1 means a commercial sponsor or foundation-backed arrangement with a documented horizon of 24 months or more. A score of 2 means episodic grants or a commercial arrangement with a horizon shorter than 24 months or not publicly documented. A score of 3 means no known funding.

The composite score is computed at render time as the sum of the four criterion scores multiplied by an OT presence multiplier. With all weights at 1.0 (the v1 default), the formula is:

```
composite = (score_ecosystem_reach + score_bus_factor + score_language_risk + score_funding_status) × ot_multiplier
```

The OT multiplier is 1.5 where `ot_confirmed: true`, and 1.0 otherwise. A component with unknown OT status receives no multiplier — the uncertainty is not treated as an absence of risk, but it is not amplified either.

Composite scores map to four risk tiers: Low (0–6), Moderate (6.01–9), High (9.01–12), Critical (12.01 and above). A component with all criteria at maximum risk and confirmed OT presence scores 18 and lands well into the Critical tier. A component with no OT presence can score at most 12, placing it at the top of the High tier. Reaching Critical requires confirmed OT presence. This is a deliberate property of the model: the multiplier encodes the judgement that the consequences of failure are qualitatively different in operational technology environments.

Composite scores are never stored in `components.yaml`. They are computed at render/export time from the raw per-criterion scores and the current `scoring.yaml`. Changing the scoring model means incrementing the version in `scoring.yaml`; component data does not change. Entries where `scoring_version` does not match the current `scoring.yaml` version are flagged as stale in published output.

**Missing data rule** (`missing_data_rule: worst_case` in `scoring.yaml`). A null score on any criterion is treated as 3 (worst case) when computing the composite. This reflects the epistemic position that unknown is not safe: a component whose funding situation cannot be established from public sources is treated as unfunded for scoring purposes. Where imputation has occurred, it is flagged per field in published output.

---

### 5. Bus Factor Derivation

Bus factor is derived manually for each component and documented in the entry notes. It is not computed algorithmically. The derivation follows four steps.

First, check the project's own stated maintainer list or team page. Some projects maintain explicit documentation of who holds commit rights and decision authority. Where this exists, it is the starting point — not the conclusion, because stated team size and actual active capacity frequently diverge.

Second, check git commit share over the last 12 months, focusing on commits touching security-critical paths: cryptographic code, parsing logic, memory management, and authentication. If one contributor accounts for more than 50% of commits in these areas, that is a strong signal of bus factor 1 regardless of stated team size. Commit count alone is insufficient; the distribution of commits across security-sensitive code is what matters.

Third, check mailing lists, issue trackers, and release histories to identify who actually responds to CVE reports and cuts releases. Release authority and CVE response authority are the two functions that cannot be absent when a security incident occurs. A project with ten contributors but one person who handles all releases and CVE coordination has a functional bus factor of 1.

Fourth, weight any public statements by maintainers about burnout, reduced involvement, or succession concerns heavily. A maintainer who has publicly stated they are struggling is providing direct evidence about availability risk. Such statements are documented in the entry notes and factored into the bus factor assessment even where the git history does not yet reflect them.

The `est_active_committers` and `bus_factor` fields are separate and may differ significantly. A project with 12 contributors but one gatekeeper records `est_active_committers: 12` and `bus_factor: 1`.

---

### 6. OT-Confirmed Definition

`ot_confirmed: true` requires at least one of the following:

- Appearance in a public SBOM from a product deployed in an OT environment.
- Reference in a CISA ICS-CERT advisory or CISA Known Exploited Vulnerabilities (KEV) entry in an OT context.
- A CVE record affecting a named OT product that embeds the component.
- A documented vendor disclosure naming the component in an OT product.

`ot_confirmed: null` means the question has not yet been researched for this entry. It does not mean the component is absent from OT environments. `ot_confirmed: false` means the question was researched and no OT presence was found.

Every entry with `ot_confirmed: true` requires a populated `ot_confirmation_source` field citing the specific source. General ecosystem reasoning — the component is present in Linux, therefore it is probably in some OT systems — does not qualify. The source must be specific.

---

### 7. Named Individual Protocol

Before any entry containing a named individual moves from `draft` to `review`, that individual is notified directly with the specific data fields about them that will appear in the published entry, a 14-day window to submit factual corrections, and the URL where the entry will be published.

Corrections are applied where the individual demonstrates a factual error. Disputed analytical judgements — for example, a maintainer who contests the bus factor assessment — are noted in the entry alongside the assessment. The individual's position and the registry's assessment both appear; the registry does not defer to the subject on questions of methodology.

This process is a quality control and courtesy measure, not a consent requirement. Public professional activity is factual and publishable. The notification exists because a named individual has the most reliable information about their own situation, and direct contact produces more accurate data than public-source research alone.

Notification status is tracked in `maintainers.yaml` via the `notification_status` field: `pending` (not yet contacted), `sent` (notified, window open), `confirmed` (window closed, corrections applied or none received), `not_applicable` (pseudonym exception applies; see Section 10). The `notification_date` field records when notification was sent.

---

### 8. Update Triggers

An entry is re-assessed — `last_assessed_date` updated, fields revised, `status` reset to `draft` if any substantive change is made — when any of the following occur:

A CVE is published affecting the component. A maintainer departure, burnout signal, or succession event is publicly documented. A funding change occurs: a new grant, grant expiry, or change in commercial arrangement. A new release is made after a gap of more than six months, which is a positive signal requiring re-evaluation of health indicators. An OT-relevant advisory references the component for the first time. Twelve months have elapsed since `last_assessed_date` regardless of whether any of the above triggers have fired.

The 12-month maximum staleness rule is unconditional. An entry that has not been re-assessed within 12 months is out of date by definition, regardless of how quiet the project has been.

---

### 9. Limitations

**Proprietary firmware.** The most significant gap in the registry is OT firmware that has not been disclosed via SBOM, CVE record, or vendor advisory. The components embedded in vehicle ECUs, medical device firmware, and industrial controllers are only partially visible from public sources. SBOM mandates under the EU Cyber Resilience Act and FDA guidance will improve this over time. Until then, `ot_confirmed: null` on a given entry should not be read as confirming absence.

**Assessment coverage.** The v0.1 registry covers 21 components. This is a seed list, not a comprehensive map. Many critical components are not yet assessed. Absence from the registry is not a risk signal in either direction.

**Scoring model subjectivity.** The four criteria are assessed by human researchers applying defined scales. Different researchers may reach different scores on the same component, particularly on `language_risk` and `funding_status` where the evidence is ambiguous. The methodology aims for consistency but does not claim objectivity. The criterion scales and weights in `scoring.yaml` are explicitly versioned and will change as the model is refined.

**Assessment lag.** The human intelligence layer updates on annual cycles or upon trigger events. Between assessments, the data may not reflect recent developments. The `last_assessed_date` field makes the assessment date visible; users should treat entries approaching 12 months of age with additional caution.

**Bus factor derivation constraints.** Bus factor is derived from publicly observable signals. Private arrangements — a company quietly funding a maintainer, an informal agreement between contributors, a succession plan that has not been publicly documented — are invisible to this methodology. The registry may undercount both risk and resilience where relevant information is not public.

---

### 10. Pseudonym Exception: D.R. Commander

The libjpeg-turbo entry references a maintainer identified as D.R. Commander (maintainer id: `dr-commander`). This is a deliberate pseudonym maintained consistently for approximately 15 years. The primary maintainer's public GitHub handle is `dcommander`. Their legal identity is not publicly disclosed.

After exhaustive research — including review of the project's published GPG key (which is not linked to a name), sponsorship records, conference appearances, and commit history — no reliable identification was possible. This is not a gap in research effort. The pseudonym is actively maintained by the individual as a matter of deliberate practice.

The §7.2(b) pseudonym exception applies on the following grounds: the pseudonym has been used consistently for approximately 15 years; the practice is deliberate and actively maintained; named-individual notification is not possible without a verified identity; and the project's funding situation and operational continuity are sufficiently documented through public sponsor records to allow a meaningful risk assessment without requiring the maintainer's legal name.

Under this exception, the libjpeg-turbo entry is published as follows:

- `display_name: "D.R. Commander"` — the pseudonym as display name
- `employer: "independent contractor (project-funded)"` — role without legal name
- A note in the entry that real identity is not publicly disclosed and that this reflects deliberate, long-standing practice
- `notification_status` set to `not_applicable` on formal approval of this exception

The exception is specific to this entry. Any future entry where a pseudonym situation arises requires a separate exception decision documented in this section.
