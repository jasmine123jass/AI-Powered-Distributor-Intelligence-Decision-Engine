# Agent 0 — Tata Steel Distributor Discovery

An evidence-driven AI decision-support engine that transforms GST registry exports into prioritized distributor discovery and verification queues for Tata Steel.

Agent 0 combines deterministic business rules, web research, structured LLM reasoning, and citation validation to identify commercially relevant distributor candidates.

It is designed to support human decision-making, not autonomous distributor onboarding.

## Key Features

Agent 0 converts distributor registry data into four explicit commercial queues:

### 1. Verified Recommendations
The only candidates eligible for Grade A/B recommendations. Nested Tree-of-Thoughts (ToT) and Stage 2 Thinking outputs contain citations that deterministic code validates against retained web-source content.

### 2. Prioritized Leads
Plausible businesses ranked for field verification. Incomplete or weak evidence cannot promote candidates to A/B recommendations.

### 3. Needs Data
Candidates with uncertain turnover, identity, or location that require additional data collection instead of assumption-based decisions.

### 4. Rejected
Records excluded by deterministic business rules, with explicit rejection reasons.

**Core principle:** Discovery remains optimistic, but missing or weak evidence is treated as uncertainty, never as a positive signal.

## Quickstart

### Installation

```bash
pip install -e ".[dev]"
```

Configure environment variables:

```bash
cp .env.example .env
```

Fill in only the credentials for services you intend to use.

Check the CLI:

```bash
agent-0 --help
```

### Offline Execution

Run the pipeline without Firecrawl, LLM, Nominatim, or Google Maps API calls.

```bash
agent-0 run --distributor-csv data/distributors.csv \
  --depot-name "Jamshedpur" --depot-lat 22.80 --depot-lon 86.20 \
  --offline --format all --output out/
```

### Connected Verification Run

```bash
agent-0 run --distributor-csv data/distributors.csv \
  --depot-name "Jamshedpur" --depot-lat 22.80 --depot-lon 86.20 \
  --tata-csv existing_tata.csv --nominatim \
  --format all --output out/
```

Use `--gmaps-top 0` to disable route refinement in connected execution.

Explicit `--config`, `--env`, and `--tata-csv` paths must exist. Agent 0 never silently falls back when an operator supplies one.

## Evidence and Reasoning Workflow

For candidates within the configured enrichment budget, Firecrawl search results are individually retained with:

- Source IDs
- URLs
- Page titles
- Bounded source content

The LLM generates a structured response containing:

### Three Nested Tree-of-Thoughts Branches

1. **Brand Alignment** — evaluates alignment with the Tata Steel distributor ecosystem.
2. **Infrastructure** — assesses business infrastructure and operational capabilities.
3. **Onboarding Risk** — identifies potential risks and unresolved concerns.

Each branch includes exact source-ID citations.

### Stage 2 Thinking

An adversarial reconciliation stage that includes:

- Challenged claims
- Conflicting evidence
- Unresolved questions
- Final decision rationale

### Citation Validation

Every cited excerpt is validated against retained source content using deterministic code.

Without citation-validated evidence, a candidate cannot receive an A/B recommendation.

## Output Artifacts

The CLI supports CSV, JSON, or combined exports using `--format csv`, `--format json`, or `--format all`.

| Artifact | Description |
|---|---|
| `agent0_results.*` | All scored candidates |
| `agent0_verified_recommendations.*` | Evidence-verified recommendations |
| `agent0_prioritized_leads.*` | Leads prioritized for field verification |
| `agent0_needs_data.*` | Candidates requiring additional information |
| `agent0_prefilter_rejected.*` | Records rejected during deterministic prefiltering |
| `agent0_post_verification_rejected.*` | Candidates rejected after verification |
| `agent0_run_manifest.json` | Execution metadata and decision audit information |

The run manifest captures:

- Input hashes
- Decision configuration
- Execution options
- Depot information
- Code version
- Queue counts

CSV exports neutralize spreadsheet formulas in source-derived text to reduce formula-injection risks.

## Location Verification Policy

Coordinates are classified using the following labels:

| Coordinate Type | Usage |
|---|---|
| `MANUAL` | Verified manually supplied coordinates |
| `PIN_EXACT` | Exact PIN-level coordinates |
| `PIN_DISTRICT` | District-level PIN coordinates |
| `NOMINATIM` | Coordinates obtained through Nominatim |
| `STATE_CENTROID` | State centroid estimates |
| `NONE` | No usable coordinates |

Only `MANUAL`, `PIN_EXACT`, and `NOMINATIM` coordinates can drive hard distance and anti-cannibalization gates.

District-level and state-centroid coordinates remain useful for discovery but are classified under Needs Data until location verification is available.

## Validation and Testing

Run the test suite:

```bash
pytest -q
```

Check formatting:

```bash
ruff format --check src tests
```

Run linting:

```bash
ruff check src tests
```

Build the Python wheel:

```bash
python -m pip wheel --no-deps .
```

The current test suite is mocked and network-free.

Run controlled connected smoke tests only with approved credentials and a bounded enrichment budget.

## Design Principles

- **Evidence over assumptions:** Weak or missing evidence never becomes a positive recommendation.
- **Deterministic decision gates:** Critical business rules are enforced through code.
- **Auditable reasoning:** Source references and decision rationales are retained.
- **Human-in-the-loop:** Final commercial decisions remain with human operators.
- **Controlled enrichment:** External API usage is constrained by configured budgets.
- **Location integrity:** Unverified geographic estimates cannot drive hard eligibility decisions.

## Project Objective

Agent 0 aims to make distributor discovery more systematic, explainable, and auditable by combining deterministic screening with AI-assisted commercial intelligence.

It transforms raw GST registry records into structured, evidence-backed business intelligence that supports better distributor identification and field verification.
