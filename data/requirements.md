# Critical Software Risk Register — Map Requirements
**Version:** 0.1  
**Status:** Draft  
**Scope:** MVP map — 21-component seed list, YAML source of truth, no generated output until v0.2

---

## 1. Design Principles

1. **Key and value are separate.** Every entity that is referenced by another entity has a stable machine-readable `id` field distinct from its display `name`. Names change; IDs do not.
2. **Normalise what is shared.** Maintainers referenced by more than one component live in `maintainers.yaml` and are referenced by `id`, not duplicated inline. This correctly models correlated risk.
3. **Unknown is a value, not a gap.** Missing data is represented as `null` in the schema, not as the string `"unknown"`. Tooling and scoring treat `null` according to the missing-data rule (see §6.3).
4. **Source of truth is YAML.** No generated files are canonical. Derived outputs (markdown, JSON, HTML) are regenerated from YAML. Do not hand-edit generated files.
5. **Minimum viable schema.** No fields are added speculatively. Every field must have a defined consumer (scoring, publication gate, display, or methodology).

---

## 2. Repository Structure

```
registry/
  components.yaml        # all component entries (v0.1: 21 entries)
  maintainers.yaml       # named maintainers, referenced by id
  scoring.yaml           # scoring model config — weights, scales, multipliers, thresholds
methodology.md           # scoring rules, bus factor derivation, OT definition
README.md                # plain-language overview and map summary table
CITATION.cff             # citation metadata
LICENSE-data             # CC BY 4.0
LICENSE-code             # MIT
```

One file per component is deferred to v0.2 (warranted at scale; unnecessary for 21 entries).

---

## 3. Schema — Maintainers

File: `registry/maintainers.yaml`  
Each entry is a named maintainer who is referenced by one or more component entries.

```yaml
- id: ""                        # slug, lowercase-hyphenated, stable, unique
  display_name: ""              # full name as publicly known
  pseudonym: false              # true if display_name is a pseudonym or handle
  employer: null                # current employer, null if unknown
  employer_public: false        # true if employer is self-disclosed by individual
  profile_url: null             # public URL (GitHub, personal site), null if none
  notification_status: ""       # one of: pending | sent | confirmed | not_applicable
  notification_date: null       # ISO date of notification, null if not yet sent
```

**Pseudonym rule:** If `pseudonym: true`, the entry may not be published until either (a) the individual's identity is resolved and `pseudonym` set to `false`, or (b) a pseudonym exception is explicitly approved and documented in `methodology.md`. See §7.2.

---

## 4. Schema — Components

File: `registry/components.yaml`  
Each entry is one assessed open source component.

```yaml
- id: ""                        # slug, lowercase-hyphenated, stable, unique
                                # examples: libcurl, samba, xz-utils, libgpg-error
  name: ""                      # display name

  # --- Codebase ---
  primary_language: ""          # C | C++ | other:<name>
  repository_url: null          # canonical repository URL

  # --- Maintainers ---
  maintainer_ids: []            # list of ids from maintainers.yaml; ordered by activity weight
  primary_maintainer_id: null   # single id from maintainers.yaml; null if unknown/distributed

  # --- Related entries ---
  related_entries: []           # list of component ids where a shared maintainer or
                                # direct dependency creates correlated failure risk

  # --- Health indicators ---
  est_active_committers: null   # integer; people with meaningful commit activity in last 12 months
  bus_factor: null              # integer; minimum people whose simultaneous loss halts the project
                                # derivation rule: see methodology.md §2
  last_release_date: null       # ISO date of most recent release
  last_assessed_date: ""        # ISO date this entry was last reviewed (required, always set)

  # --- Funding ---
  funding_status: ""            # unfunded | episodic_grants | commercial_sponsor | foundation_backed
  funding_horizon_months: null  # integer; documented months of current funding remaining
                                # null if unknown; 0 if funding has ended

  # --- Dependency footprint ---
  known_major_dependents: []    # flat list of strings; max 5; manually curated, most significant known users

  # --- OT ---
  ot_confirmed: null            # true | false | null (null = unknown)
  ot_confirmation_source: null  # string; citation for ot_confirmed: true entries; null otherwise

  # --- Scoring (derived, set by scoring tooling or manually with source noted) ---
  scoring_version: null         # string; scoring.yaml version used for this assessment
  score_ecosystem_reach: null   # integer 1–3; see §6.3
  score_bus_factor: null        # integer 1–3; see §6.3
  score_language_risk: null     # integer 1–3; see §6.3
  score_funding_status: null    # integer 1–3; see §6.3
  # score_composite is NOT stored; computed at render time from scoring.yaml

  # --- Publication ---
  status: draft                 # draft | review | published
  notes: null                   # free text; maintainer signals, recent events, data confidence notes
```

---

## 5. Identifiers

Component IDs and maintainer IDs follow the same rules:

- Lowercase ASCII only
- Hyphens as word separators, no underscores, no dots
- No version numbers or years
- Derived from the canonical project name; exceptions documented inline

**Examples:**

| Display name | id |
|---|---|
| libcurl | `libcurl` |
| XZ Utils (liblzma) | `xz-utils` |
| Daniel Stenberg | `stenberg-daniel` |
| D.R. Commander | `dr-commander` *(pseudonym: true)* |
| Werner Koch | `koch-werner` |

Maintainer IDs use `surname-given` order to group by family name in sorted lists.

---

## 6. Scoring Model

### 6.1 Architecture

The scoring model is fully defined in `registry/scoring.yaml`. Component data stores only raw per-criterion scores. All weights, multipliers, tier thresholds, and scale definitions live in `scoring.yaml` and are versioned independently of component data.

Composite scores are computed at render/export time by reading `scoring.yaml` against component raw scores. They are never stored as source-of-truth in `components.yaml`. Changing the model means editing `scoring.yaml` and incrementing its version — no component data changes required.

### 6.2 scoring.yaml structure

```yaml
version: "1"                   # increment on any breaking change

criteria:
  - id: ecosystem_reach
    label: "Ecosystem reach"
    weight: 1.0                # relative weight in composite; change here, not in component data
    scale:
      1: "Significant but bounded footprint"
      2: "Present in most major Linux distributions or platforms at scale"
      3: "Billions of installations; embedded in OT, medical, or automotive at scale"

  - id: bus_factor
    label: "Bus factor"
    weight: 1.0
    scale:
      1: "Bus factor ≥ 3"
      2: "Bus factor = 2"
      3: "Bus factor = 1; single person holds authority, knowledge, or commit rights"

  - id: language_risk
    label: "Language risk"
    weight: 1.0
    scale:
      1: "Memory-safe language; or C/C++ with systematic fuzzing or formal verification"
      2: "C/C++ with reactive security practices"
      3: "C/C++ with no documented security practices or no demonstrated active response"

  - id: funding_status
    label: "Funding status"
    weight: 1.0
    scale:
      1: "Commercial sponsor or foundation-backed, documented horizon ≥ 24 months"
      2: "Episodic grants or arrangement with horizon < 24 months or undocumented"
      3: "No known funding"

multipliers:
  ot_confirmed:
    true: 1.5
    false: 1.0
    null: 1.0                  # unknown OT status: no multiplier applied

tiers:
  - label: "Low"
    min: 0
    max: 6
  - label: "Moderate"
    min: 6.01
    max: 9
  - label: "High"
    min: 9.01
    max: 12
  - label: "Critical"
    min: 12.01
    max: 999

missing_data_rule: worst_case  # null criterion scores are treated as 3
```

The values above are the v1 starting point. They will change.

### 6.3 Component score fields

`components.yaml` stores raw per-criterion scores and a version reference only:

```yaml
scoring_version: "1"           # which scoring.yaml version produced these scores
score_ecosystem_reach: null    # integer 1–3
score_bus_factor: null         # integer 1–3
score_language_risk: null      # integer 1–3
score_funding_status: null     # integer 1–3
```

When `scoring.yaml` version increments, component entries are re-assessed and `scoring_version` updated. Stale entries (where `scoring_version` does not match current `scoring.yaml version`) are flagged in published output.

### 6.4 Missing Data Rule

`null` on any criterion score is treated as 3 (worst case) when computing composite. Applied per the `missing_data_rule` field in `scoring.yaml`. Flagged per-field in published output wherever imputation occurs.

---

## 7. Named Individual Protocol

### 7.1 Standard notification

Before any entry containing a named individual is moved from `draft` to `review`, that individual must be notified directly with:

- The specific data fields about them that will appear
- A 14-day window to submit factual corrections
- The URL where the entry will be published

Corrections are applied where the individual demonstrates a factual error. Disputed analytical judgments (e.g., contested bus factor assessment) are noted in the entry alongside the assessment. Notification is recorded in `maintainers.yaml` via `notification_status` and `notification_date`.

This is a quality control and courtesy measure, not a consent requirement. Public professional activity is factual and publishable.

### 7.2 Pseudonym exception

If `pseudonym: true` on a maintainer entry, the linked component entry may not advance to `review` until one of:

(a) The individual's identity is confirmed and `pseudonym` set to `false`, or  
(b) A documented exception is approved: the entry is published with the pseudonym as display name, `employer` omitted, and a note that identity could not be verified. The exception and its rationale are recorded in `methodology.md`.

The D.R. Commander entry (maintainer id: `dr-commander`) is the current open case requiring resolution before `libjpeg-turbo` can publish.

---

## 8. Bus Factor Derivation Rule

Bus factor is the minimum number of contributors whose simultaneous unavailability would prevent the project from functioning: unable to merge security patches, cut releases, or respond to critical issues.

**Derivation procedure (manual, documented in entry notes):**

1. Check the project's own stated maintainer list or team page.
2. Check git commit share over the last 12 months: if one contributor accounts for > 50% of commits touching security-critical paths, that is a strong bus factor = 1 signal regardless of stated team size.
3. Check mailing list or issue tracker: who actually responds to CVE reports and cuts releases?
4. If maintainers have publicly stated burnout, reduced involvement, or succession concerns, weight those statements heavily.

`est_active_committers` and `bus_factor` are separate fields. A project with 12 contributors but one gatekeeper has `est_active_committers: 12` and `bus_factor: 1`.

---

## 9. OT-Confirmed Definition

`ot_confirmed: true` requires at least one of:

- Appearance in a public SBOM from a product deployed in an OT environment
- Reference in a CISA ICS-CERT advisory or CISA KEV entry in an OT context
- CVE record affecting a named OT product that embeds the component
- Documented vendor disclosure naming the component in an OT product

`ot_confirmed: null` means not yet researched. `ot_confirmed: false` means researched and no OT presence found.

Every `ot_confirmed: true` entry requires a populated `ot_confirmation_source` citing the specific source.

---

## 10. Validation Rules

An entry must pass all of the following to move from `draft` to `review`:

| Rule | Check |
|---|---|
| ID format | `id` matches `^[a-z0-9-]+$` and is unique across `components.yaml` |
| Name populated | `name` is non-empty |
| Language set | `primary_language` is one of the allowed values |
| Assessment date | `last_assessed_date` is a valid ISO date ≤ today |
| Maintainer refs valid | All ids in `maintainer_ids` and `primary_maintainer_id` exist in `maintainers.yaml` |
| Related entries valid | All ids in `related_entries` exist in `components.yaml` |
| OT source present | If `ot_confirmed: true`, `ot_confirmation_source` is non-null |
| Scoring complete | All four `score_*` fields are non-null integers 1–3; `scoring_version` matches current `scoring.yaml` version |
| Notification done | For each maintainer with `pseudonym: false`, `notification_status` is `confirmed` or `not_applicable` |
| Pseudonym gate | No maintainer in `maintainer_ids` has `pseudonym: true` unless exception documented in `methodology.md` |

An entry moves from `review` to `published` on manual approval only. No automated promotion.

---

## 11. Update Triggers

An entry must be re-assessed (new `last_assessed_date`, fields updated, `status` reset to `draft` if substantive change) when any of the following occur:

- A CVE is published affecting the component
- A maintainer departure, burnout signal, or succession event is publicly documented
- A funding change (new grant, grant expiry, commercial arrangement change)
- A new release after a gap of > 6 months (positive signal requiring re-evaluation)
- An OT-relevant advisory references the component for the first time
- 12 months have elapsed since `last_assessed_date` regardless of trigger events

---

## 12. Definition of Done — v0.1

The map is complete for v0.1 when:

- [ ] All 21 `components.yaml` entries have `status: published`
- [ ] All four scoring fields populated on all 21 entries; `scoring_version` set on all entries
- [ ] Composite scores verified by running scoring tool against `scoring.yaml` v1
- [ ] All named maintainers in `maintainers.yaml` with `notification_status: confirmed` or `not_applicable`
- [ ] Pseudonym exception documented for `dr-commander` or entry held at `draft`
- [ ] `methodology.md` written and covers: scoring rules, bus factor derivation, OT-confirmed definition, missing data rule, named individual protocol
- [ ] `README.md` includes a summary table of all 21 entries with id, name, risk tier, and OT status
- [ ] At least 3 entries with `ot_confirmed: true` and `score_composite` in the Critical tier (≥ 13)
- [ ] `CITATION.cff` populated
- [ ] `related_entries` populated for libcurl ↔ libssh2 and any other correlated-maintainer pairs identified during review

---

## Open Items

| # | Status | Item | Blocking |
|---|---|---|---|
| 1 | **RESOLVED → exception** | D.R. Commander: identity unresolvable by design (deliberate 15-year pseudonym). §7.2(b) pseudonym exception applies. Publish with pseudonym as display name; employer as "independent contractor (project-funded)"; note that real identity is not publicly disclosed. Document exception in methodology.md. | libjpeg-turbo can publish once exception is documented |
| 2 | **PARTIAL** | libxml2: Iván Chavero and Daniel Garcia Moreno confirmed as new maintainers. Garcia Moreno employer: SUSE (medium confidence, inferred). Chavero employer: unknown, personal email only. Both require direct employer verification before publication per named-individual protocol. | libxml2 blocked until both employers verified or null accepted |
| 3 | **RESOLVED** | libssh2: Stenberg is release manager only; Viktor Szakats is de facto day-to-day lead. `primary_maintainer_id` should be null; `related_entries` link to libcurl retained (weak — shared Stenberg formal role). Szakats, Fandrich, Cosgrove added as maintainer entries. | Seed list correction applied |
| 4 | **RESOLVED** | PCRE2: two co-maintainers confirmed (Nicholas Wilson + Zoltan Herczeg). Hazel fully retired. Bus factor is 2. Herczeg's PCRE2 work is likely a side project (personal email). Wilson employer unknown. | Seed list correction applied; Wilson employer still open |
| 5 | **RESOLVED** | Samba: STF grant ended February 2026, no follow-on announced. SerNet commercial continuity provides implicit floor. `funding_horizon_months: 0`. Post-grant commit frequency unverified — flag for follow-up after sambaXP 2026 (May–June). | Score updated; sambaXP check deferred |
| 6 | **NEW** | libxml2 ↔ libxslt compounded risk: Chavero maintains both; libxslt depends on libxml2. `related_entries` added to both entries. Highest-density single-point-of-failure in seed list: one person, two interdependent libraries, both freshly transitioned from decade-long experts. | Documented; no additional blocker |
| 7 | **NEW** | Samba post-grant commit frequency: verify github.com/samba-team/samba commit rate before vs. after February 2026. Cannot be inferred from public statements alone. | Score accuracy for funding_status |
| 8 | **NEW** | Nicholas Wilson (PCRE2 co-maintainer) employer unknown. No public profile found. | pcre2 publication |
