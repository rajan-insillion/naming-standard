# Insillion JSON Schema Naming Standard
## Architecture, Conventions & Governance Guide

**Version:** 1.0  
**Applies to:** USA · India · Singapore · Middle East  
**Audience:** Business Analysts, Developers, SI Partners, LLM Pipeline Engineers

---

## Table of Contents

1. [Design Dimensions to Consider](#1-design-dimensions-to-consider)
2. [Field Naming Conventions](#2-field-naming-conventions)
3. [Structural Patterns](#3-structural-patterns)
4. [Schema Taxonomy & File Organisation](#4-schema-taxonomy--file-organisation)
5. [Type System](#5-type-system)
6. [Geography & Jurisdiction Handling](#6-geography--jurisdiction-handling)
7. [Use-Case Layers](#7-use-case-layers)
8. [Git Repository Architecture](#8-git-repository-architecture)
9. [Tooling & Automation](#9-tooling--automation)
10. [Governance & Process](#10-governance--process)
11. [Quick Reference Card](#11-quick-reference-card)

---

## 1. Design Dimensions to Consider

Before writing any naming rule, the following 8 dimensions must be resolved. Every downstream decision flows from these.

| # | Dimension | Question to Resolve | Example Decision |
|---|-----------|--------------------|--------------------|
| 1 | **Lexicon** | What words are allowed for field names? | snake_case, from an approved glossary |
| 2 | **Suffix Sigils** | How are structural roles encoded in the name? | `_^` = array, `_@` = mutually exclusive group |
| 3 | **Type System** | What scalar types exist, and are they shared? | `currency`, `year`, `date`, `naic` etc. |
| 4 | **Hierarchy Depth** | How deep may nesting go before a ref/anchor is required? | Max 4 levels; beyond that extract as `$ref` |
| 5 | **Repeatability** | How are one-to-many relationships expressed? | `_^` suffix, consistent across all schemas |
| 6 | **Geography Variance** | How do jurisdiction-specific fields sit alongside universal fields? | `_geo` sub-object or override layer |
| 7 | **Versioning** | How is schema evolution tracked without breaking consumers? | Semantic version in file name + `schema_version` field |
| 8 | **Use-Case Binding** | Does one schema serve all 4 use cases, or are there derivative views? | Canonical schema + thin view projections |

Resolving all 8 dimensions produces a **Schema Constitution** — a single authoritative document every creator must sign off on before creating schemas.

---

## 2. Field Naming Conventions

### 2.1 Casing Rule

All field names use **snake_case**. 
No camelCase. No kebab-case. No PascalCase.
Avoid prepositions such as 'of', 'and', 'or'.

```
✅  policy_start_date
✅  registration_date
✅  general_aggregate
✅  naic_code
❌  policyStartDate
❌  policy-start-date
❌  PolicyStartDate
❌  date_of_registration
```

### 2.2 Approved Sigil Suffixes

These suffixes are already established in the existing schemas and must be used consistently:

| Sigil | Meaning | Usage |
|-------|---------|-------|
| `_^` | Repeatable / array | `product_^`, `location_^`, `history_^` |
| `_@` | Mutually exclusive option group | `policy_type_@`, `valuation_@`, `legal_entity_@` |

No other structural sigils may be introduced without a governance review.

### 2.3 Approved Abbreviations

Maintain a central glossary. Do not abbreviate freely. Approved short forms:

| Short Form | Full Meaning |
|------------|-------------|
| `no` | number (policy_no, phone_no) |
| `pct` | percent |
| `amt` | amount — use only when `_amount` is too verbose |
| `bi` | business income |
| `emp` | employee |
| `sqft` | square feet |
| `rc` | replacement cost |
| `acv` | actual cash value |
| `lob` | line of business |
| `naic` | National Association of Insurance Commissioners |
| `npn` | National Producer Number |
| `sic` | Standard Industrial Classification |
| `qa` | questions & answers (underwriting Q&A block) |
| `coi` | certificate of insurance |
| `gl` | general liability |
| `um` / `uim` | uninsured / underinsured motorist |

Any new abbreviation must be added to the glossary via Pull Request before use.

### 2.4 Name Reuse Rule

**If a field name already exists in any ACORD schema in the repository, reuse that exact name and structure.**  
This is already the stated rule in `objective.md` and is the single most important convention to enforce.

Examples of names that are already canonical and must not be varied:

```
agency, carrier, insured, location_^, additional_interest_^,
policy_start_date, policy_start_time, application_date,
cust_ref_id, remarks, signature, prior_carrier_^, loss_run
```

---

## 3. Structural Patterns

### 3.1 Standard Header Block

Every schema — regardless of line of business or geography — must open with this identical header block:

```json
{
  "app_form_name":    {"type": "string"},
  "app_form_version": {"type": "string"},
  "application_date": {"type": "date"},
  "cust_ref_id":      {"type": "string"},
  "policy_start_date":{"type": "date"},
  "policy_start_time":{"type": "string"},

  "producer":  { ... },
  "carrier": { ... },
  "insured": { ... }
}
```

### 3.2 Standard Producer / Carrier / Insured Blocks

These 3 sub-objects are identical across all 3 existing ACORD schemas. They are canonical. Copy verbatim, do not modify field names.

```json
"producer": {
  "name":      {"type": "agency"},
  "address_1": {"type": "string"},
  "address_2": {"type": "string"},
  "city":      {"type": "string"},
  "state":     {"type": "state"},
  "zipcode":   {"type": "zip"}
}
```

### 3.3 Standard Signature Block

Every schema must close with this block, unchanged:

```json
"signature": {
  "applicant_signature_date":  {"type": "date"},
  "producer_name":             {"type": "string"},
  "producer_state_license_no": {"type": "string"},
  "national_producer_number":  {"type": "string", "description": "NPN, required in Florida"}
}
```

### 3.4 Repeatable Section Pattern (`_^`)

A repeatable section carries all its child fields inline. Do not hoist fields out of the array object.

```json
"prior_carrier_^": {
  "year":       {"type": "year"},
  "carrier":    {"type": "carrier"},
  "policy_no":  {"type": "policy_no"},
  "premium":    {"type": "number"},
  "start_date": {"type": "date"},
  "end_date":   {"type": "date"}
}
```

### 3.5 Mutually Exclusive Options Pattern (`_@`)

An `_@` group contains only `bool` fields. No other types belong inside it.

```json
"legal_entity_@": {
  "corporation":      {"type": "bool"},
  "llc":              {"type": "bool"},
  "partnership":      {"type": "bool"},
  "individual":       {"type": "bool"},
  "not_for_profit":   {"type": "bool"},
  "other":            {"type": "bool"}
}
```

### 3.6 Q&A Block Pattern

Underwriting questions use a consistent `opted + description` pattern. Group them under a `qa` object:

```json
"qa": {
  "blasting_or_explosives": {
    "opted":       {"type": "boolean"},
    "description": {"type": "string"}
  },
  "subcontractors_without_coi": {
    "opted":       {"type": "boolean"},
    "description": {"type": "string"}
  }
}
```

---

## 4. Schema Taxonomy & File Organisation

### 4.1 Schema Categories

| Category | Prefix | Description | Examples |
|----------|--------|-------------|---------|
| ACORD forms | `acord_` | Standard ACORD application forms | `acord_125`, `acord_126`, `acord_140` |
| Line of business | `lob_` | LOB-specific supplement schemas | `lob_cyber`, `lob_marine`, `lob_wc` |
| Geography supplement | `geo_` | Jurisdiction-specific field overlays | `geo_india_fire`, `geo_uae_motor` |
| Product schema | `product_` | Assembled schema for a specific product | `product_bop_usa`, `product_fire_india` |
| Report / view | `view_` | Projected subset for dashboards/reports | `view_premium_summary`, `view_loss_run` |

### 4.2 File Naming

```
{category_prefix}{name}_v{major}.{minor}.json

Examples:
  acord_125_v2025.03.json
  lob_cyber_v1.0.json
  geo_india_fire_v1.2.json
  product_bop_usa_v3.1.json
  view_premium_summary_v1.0.json
```

### 4.3 Schema Versioning Fields

Every schema file must include at its root:

```json
{
  "schema_version": "1.0",
  "schema_category": "acord",
  "jurisdiction": ["USA"],
  "lines_of_business": ["CGL", "Property"],
  ...
}
```

---

## 5. Type System

### 5.1 Canonical Scalar Types

All `{"type": "..."}` values must come from this registry. No ad-hoc type names.

| Type Token | Meaning | Format / Validation |
|------------|---------|---------------------|
| `string` | Free text | Any UTF-8 string |
| `number` | Numeric (int or float) | JSON number |
| `boolean` | Boolean | `true` / `false` |
| `date` | Calendar date | ISO 8601: `YYYY-MM-DD` |
| `year` | 4-digit year | `YYYY` |
| `currency` | Monetary amount | Numeric, 2 decimal places |
| `phone_no` | Phone number | String |
| `email` | Email address | String, valid email format |
| `website` | URL | String |
| `naic` | NAIC code | String |

### 5.2 Adding New Types

New types require a PR to `common/types_registry.json` with:
- Token name
- Meaning
- Validation rule
- At least 1 example value

---

## 6. Geography & Jurisdiction Handling

### 6.1 Strategy: Canonical + Overlay

Do not create entirely separate schemas per country. Use a 2-layer approach:

```
canonical schema (universal fields)
        +
geo overlay schema (jurisdiction-specific additions/overrides)
        =
assembled product schema (merged at runtime or build time)
```

### 6.2 Geo Overlay Structure

A geo overlay file contains only the delta — fields that are added, modified, or required for that jurisdiction.

```json
// geo_india_fire_v1.0.json
{
  "schema_category": "geo_overlay",
  "jurisdiction": ["India"],
  "base_schema": "acord_140_v2016.03.json",
  "overrides": {
    "insured": {
      "gstin":    {"type": "string", "description": "Goods & Services Tax Identification Number"},
      "pan_no":   {"type": "string", "description": "Permanent Account Number"}
    }
  },
  "additions": {
    "tariff_class": {"type": "string"},
    "iib_code":     {"type": "string", "description": "Insurance Information Bureau class code"}
  }
}
```

### 6.3 Jurisdiction Codes

Use ISO standards:

| Region | Code |
|--------|------|
| USA | `US` |
| India | `IN` |
| Singapore | `SG` |
| UAE | `AE` |
| Saudi Arabia | `SA` |
| Qatar | `QA` |

---

## 7. Use-Case Layers

The same canonical schema is consumed differently across the 4 use cases. Do not create 4 separate schemas. Instead, use **view projections** and **metadata annotations**.

### 7.1 Use Case 1 — LLM Document Extraction

The canonical schema is used directly as the LLM prompt input.

Requirements:
- Every field must have a `description` where the field name alone is ambiguous
- Fields needed by the LLM for context should include an example in the description
- Mark mandatory extraction targets: `"required": true`

```json
"policy_start_date": {
  "type": "date",
  "description": "Proposed effective date of the policy. Look for phrases like 'effective date', 'inception date', 'from date'.",
  "required": true
}
```

### 7.2 Use Case 2 — Business Analyst / Product Creation

BAs need human-readable labels and grouping hints.

Add a `_ui` annotation block per schema (stripped at runtime):

```json
"_ui": {
  "label": "Policy Start Date",
  "section": "Policy Details",
  "display_order": 3,
  "tooltip": "The date from which coverage begins"
}
```

### 7.3 Use Case 3 — Backend Integration

Developers need type constraints, API field mappings, and required/optional flags.

Add a `_api` annotation block:

```json
"_api": {
  "insillion_field": "policyInceptionDate",
  "required": true,
  "nullable": false,
  "source": "ACORD 125 § 2.1"
}
```

### 7.4 Use Case 4 — Reports & Dashboards

Expose a `view_` schema that is a flattened projection of the canonical schema, safe for BI tool consumption.

```json
// view_premium_summary_v1.0.json
{
  "schema_category": "view",
  "source_schemas": ["acord_125", "acord_126"],
  "fields": {
    "insured_name":           {"type": "string", "source_path": "insured.name"},
    "policy_no":              {"type": "policy_no", "source_path": "carrier.policy_no"},
    "policy_start_date":      {"type": "date", "source_path": "policy_start_date"},
    "total_premium":          {"type": "currency", "source_path": "coverage.premiums.total"},
    "general_aggregate_limit":{"type": "currency", "source_path": "coverage.limits.general_aggregate"}
  }
}
```

---

## 8. Git Repository Architecture

### 8.1 Recommended Repository Structure

```
insillion-schemas/
│
├── README.md
├── CONTRIBUTING.md
├── SCHEMA_CONSTITUTION.md          ← the master rules document
│
├── common/
│   ├── types_registry.json         ← all scalar type definitions
│   │── agency.json             ← reusable canonical blocks
│   │── carrier.json
│   │── insured.json
│   │── address.json
│   │── additional_interest.json
│   │── signature.json
│   └── glossary.json               ← approved abbreviations & terms
│
├── acord/
│   ├── acord_125_v2025.03.json
│   ├── acord_126_v2025.03.json
│   └── acord_140_v2016.03.json
│
├── lob/
│   ├── liability/lob_liability.json
│   ├── liability/lob_general_liability.json
│   └── property/lob_property.json
│
├── geo/
│   ├── geo_india_fire_v1.0.json
│   ├── geo_singapore_motor_v1.0.json
│   └── geo_uae_liability_v1.0.json
│
├── products/
│   ├── product_bop_usa_v3.1.json
│   ├── product_fire_india_v2.0.json
│   └── product_pkg_singapore_v1.0.json
│
├── views/
│   ├── view_premium_summary_v1.0.json
│   ├── view_loss_run_v1.0.json
│   └── view_underwriting_dashboard_v1.0.json
│
├── tools/
│   ├── validator/                  ← JSON schema linter CLI
│   ├── merger/                     ← canonical + geo overlay assembler
│   └── doc_generator/              ← auto-generates BA-readable docs
│
└── tests/
    ├── fixtures/                   ← sample PDFs/DOCXs for extraction tests
    └── expected_outputs/           ← expected LLM extraction results
```

### 8.2 Branch Strategy

| Branch | Purpose | Who merges |
|--------|---------|------------|
| `main` | Production-stable schemas | Schema Governance Board only |
| `develop` | Integration of reviewed PRs | Tech Lead |
| `feat/schema-{name}` | New schema development | Any contributor |
| `fix/schema-{name}` | Bug fix in existing schema | Any contributor |
| `release/v{x.y}` | Release preparation | Tech Lead |

### 8.3 Access Control

| Role | Access Level | Who |
|------|-------------|-----|
| Schema Owner | Write to `main` after review | Insillion core team |
| Contributor | Write to `feat/*`, `fix/*` | Employees + approved SI partners |
| Reviewer | PR review required | Minimum 2 reviewers per PR to `develop` |
| Reader | Read-only | All other partners |

---

## 9. Tooling & Automation

### 9.1 CLI Validator (`tools/validator`)

A command-line tool that checks every schema against the constitution. Must be run before any PR is raised.

Rules it enforces:
- All field names are snake_case
- All `type` values exist in `types_registry.json`
- All `_@` groups contain only `bool` fields
- Canonical blocks (agency, carrier, insured, signature) match the reference exactly
- No field name that exists in a canonical block is redefined differently
- `schema_version`, `schema_category`, `jurisdiction` are present at root
- Nesting depth does not exceed 4 levels

```bash
# Usage
npx insillion-validator validate ./acord/acord_126_v2025.03.json
npx insillion-validator validate-all ./acord/
npx insillion-validator diff acord_125_v2025.03.json acord_125_v2024.01.json
```

### 9.2 GitHub Actions CI Pipeline

Every PR triggers:

```yaml
# .github/workflows/schema_ci.yml
steps:
  - name: Validate schema naming conventions
    run: npx insillion-validator validate-all

  - name: Check for canonical block drift
    run: npx insillion-validator check-canonical-blocks

  - name: Run extraction regression tests
    run: npm test -- --suite=extraction

  - name: Generate schema documentation
    run: npx insillion-doc-gen --output ./docs/

  - name: Post summary to PR
    run: npx insillion-validator report --format=github-pr
```

PRs to `develop` cannot be merged if any validation step fails.

### 9.3 Schema Linter (IDE Plugin)

Provide a VS Code extension that:
- Flags non-snake_case field names inline
- Warns when a field name matches an existing canonical name but has a different type
- Autocompletes approved type tokens from `types_registry.json`
- Autocompletes approved abbreviations from `glossary.json`

Distribute via internal VS Code marketplace or as a `.vsix` package in the repository.

### 9.4 Doc Generator (`tools/doc_generator`)

Auto-generates a human-readable HTML or PDF reference from each schema, targeted at Use Case 2 (Business Analysts).

Output per schema:
- Field name
- Type
- Description
- Required / Optional
- Example value
- Which use cases it applies to

```bash
npx insillion-doc-gen --schema ./acord/acord_126_v2025.03.json --output ./docs/acord_126.html
```

### 9.5 Schema Merger (`tools/merger`)

Assembles a final product schema from a canonical base + geo overlay:

```bash
npx insillion-merger \
  --base ./acord/acord_140_v2016.03.json \
  --overlay ./geo/geo_india_fire_v1.0.json \
  --output ./products/product_fire_india_v2.0.json
```

### 9.6 LLM Regression Test Harness

For Use Case 1, maintain a regression suite:
- Input: sample PDF/DOCX fixture
- Schema: the extraction schema
- Expected output: `tests/expected_outputs/{test_name}.json`
- Tolerance: defined per field (exact match for dates/numbers; semantic match for strings)

Run on every PR that modifies a schema used in LLM extraction.

---

## 10. Governance & Process

### 10.1 Schema Governance Board (SGB)

A standing group of 3–5 people with authority over the Schema Constitution and canonical blocks.

| Role | Responsibility |
|------|---------------|
| Schema Architect (1) | Owns constitution, approves structural changes |
| BA Lead (1) | Ensures field semantics reflect business intent |
| Integration Lead (1) | Ensures alignment with Insillion API contracts |
| Geo Representatives (1 per active region) | Review geo overlay PRs |

SGB reviews are required for:
- Any new sigil or structural pattern
- Any new type token added to the registry
- Any change to a canonical block (agency, carrier, insured, signature)
- Any new schema category

### 10.2 Contributor Workflow

```
1. Branch from develop:  git checkout -b feat/schema-lob-cyber
2. Create / edit schema
3. Run validator locally: npx insillion-validator validate
4. Raise PR to develop
5. CI pipeline runs automatically
6. 2 peer reviews required (at least 1 from SGB for structural changes)
7. Merge to develop
8. SGB batch-reviews develop → main monthly
```

### 10.3 Onboarding Checklist for New Contributors

Every new employee or SI partner must complete before creating schemas:

- [ ] Read `SCHEMA_CONSTITUTION.md`
- [ ] Read `common/glossary.json`
- [ ] Study the 3 reference schemas: `acord_125`, `acord_126`, `acord_140`
- [ ] Install VS Code schema linter extension
- [ ] Run the validator on all 3 reference schemas to confirm local tooling works
- [ ] Submit a trivial PR (description fix) to demonstrate understanding of the PR workflow
- [ ] Receive approval from SGB member before being granted contributor access

### 10.4 Schema Change Classification

| Change Type | Backward Compatible | Approval Required | Version Bump |
|-------------|--------------------|--------------------|--------------|
| Add optional field | Yes | Peer review | Minor (`1.0` → `1.1`) |
| Add required field | No | SGB review | Major (`1.0` → `2.0`) |
| Rename field | No | SGB review | Major |
| Change field type | No | SGB review | Major |
| Add new `_@` option | Yes | Peer review | Minor |
| Remove field | No | SGB review | Major |
| Add geo overlay | Yes | Geo rep review | Minor |

### 10.5 Deprecation Policy

1. Mark field with `"deprecated": true, "replaced_by": "new_field_name"` in schema
2. Announce in `CHANGELOG.md` with migration guide
3. Maintain deprecated field for a minimum of 2 major versions
4. Remove only after all downstream consumers confirm migration

---

## 11. Quick Reference Card

### Naming Rules at a Glance

```
snake_case always
_^   → repeatable/array section
_@   → mutually exclusive option group
_ui  → UI annotation (stripped at runtime)
_api → API binding annotation (stripped at runtime)
```

### Type Tokens at a Glance

```
string · number · bool · date · year · currency
YN · state · zip · phone_no · email · website
agency · carrier · company · naic · policy_no
```

### Field Decision Tree

```
Is this field already in acord_125 / acord_126 / acord_140?
  YES → copy the exact name and type. Stop.
  NO  →
    Is it a yes/no with a narrative?  → opted + description pattern
    Is it one of many exclusive choices? → put inside _@ group as bool
    Is it a repeated record?  → suffix with _^
    Is it a scalar?  → pick from types_registry.json
    Is it jurisdiction-specific? → put in geo overlay, not canonical schema
```

### File Naming

```
{category}_{name}_v{major}.{minor}.json
acord_125_v2025.03.json
product_bop_usa_v3.1.json
geo_india_fire_v1.0.json
view_premium_summary_v1.0.json
```

---

*Questions or proposed changes to this standard: raise a PR to `SCHEMA_CONSTITUTION.md` or contact the Schema Governance Board.*
