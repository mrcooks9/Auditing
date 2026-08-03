---
name: frr-auditing-best-of-both-worlds
description: Best-of-both-worlds FRR gate skill combining Jonathan PR gate knowledge with Matt/Cyrus automation pattern rubrics.
---

# Auditing FRR Gate - Best-of-Both-World FRR Skill

**Generated:** 2026-06-15  
**Output folder:** `C:\Users\mghadial\OneDrive - Microsoft\Documents\Copilot - CLI\Skills from Jonathan Prj\FRR-Best-of-Both-World-Skills`  
**Jonathan PR source:** `Auditing\SKILL.md`  
**Matt/Cyrus Automation source:** `Auditing-FRR-Skill.md`  
**Generation mode:** merged skill - preserve both source knowledge bases and use the integrated operating model below.

## How gate owners should use this skill

Use this file as the consolidated gate-owner skill for automated or semi-automated FRR review. The intent is to combine:

1. **Jonathan PR skill strengths:** gate-specific expert workflow, artifact expectations, applicability logic, review depth, evidence-generation patterns, and standard JSON/ADO comment output contract.
2. **Matt/Cyrus automation strengths:** FRR pattern rubric, tiering, hard-blocker logic, explicit evidence scoring, ADO/PR evaluation workflow, self-close rules, and gate-owner process constraints.

## Integrated operating model

1. **Intake** - Start from a PR URL, ADO Feature ID, or FRR gate Task ID. Resolve linked ADO items, feature description, PR diff, child FRR task, comments, linked artifacts, and any owner signoff evidence.
2. **Applicability and self-close check** - Use both sources. If either source requires documented triage before N/A/self-close, require that documentation. Do not treat missing evidence as N/A.
3. **Tier / scope classification** - Apply the automation tier model when present, then calibrate depth using Jonathan's gate-specific applicability and risk/blast-radius guidance.
4. **Artifact gathering** - Gather the union of artifacts required by both sources. Missing required evidence becomes `needs-input`, partial, or a blocker depending on the stricter source rule.
5. **Deep technical review** - Use Jonathan's procedure and references as the primary expert-review workflow.
6. **Pattern scorecard** - Apply the Matt/Cyrus pattern rubrics as the structured scorecard and audit trail.
7. **Verdict aggregation** - Use the stricter outcome when the two sources differ. Any hard blocker in the automation source remains a hard blocker unless the Jonathan source contains a more explicit gate-owner exception path.
8. **Output** - Produce both a human-readable review and a machine-readable verdict envelope. Include evidence citations: ADO ID, PR file/line, artifact link, comment, or test output.
9. **ADO write-back** - Post a discussion comment with the consolidated scorecard, findings, required remediations, owner/action/date, and final verdict. Do not close GC-owned gates unless the source skill explicitly allows self-attestation and evidence is complete.

## Merge guidance for this gate

Use Jonathan audit-event catalog generation as the primary deliverable. Apply Automation rubrics for explicit scope triage, read-only self-close validation, portal review/signoff evidence, and hard-blocker scoring.

## Canonical output envelope

Use this normalized envelope even if one source uses older field names. Findings must be objects, not strings.

```json
{
  "area": "auditing",
  "verdict": "pass|conditional|fail|needs-input",
  "blockingFindings": [
    { "id": "<AREA>-1", "severity": "critical|high|medium|low", "title": "...", "detail": "...", "remediation": "..." }
  ],
  "conditionalFindings": [
    { "id": "<AREA>-2", "severity": "medium|low", "title": "...", "detail": "...", "remediation": "..." }
  ],
  "summary": "One-paragraph verdict summary with evidence basis.",
  "adoCommentMarkdown": "## FRR Auditing FRR Gate - <Verdict>\n\n<scorecard, findings, evidence, next actions>",
  "instrumentation": {
    "skill": "frr-auditing-best-of-both-worlds",
    "jonathanSource": "Auditing\\SKILL.md",
    "automationSource": "Auditing-FRR-Skill.md",
    "applicabilityChecked": true,
    "exitReason": "completed"
  }
}
```

---

# Source A - Jonathan PR skill

> Preserve this source as the expert-review procedure and domain knowledge base.

---
name: frr-auditing
version: 3.0.0
description: >
  Conducts structured, schema-driven FRR Auditing reviews that validate
  CRUD+ coverage, metadata completeness, taxonomy correctness, and onboarding
  hygiene for customer-facing audit events. Invoke when user asks about FRR
  Auditing, audit logging, compliance audit, CRUD operations, audit trail,
  AuditingService wiring, MessageBus audit event emission or onboarding, FSI
  vs FSI Core pipeline choice for audit events, GraphSharedUtil tagging for
  background tasks, audit category classification or taxonomy, audit event
  schema or metadata fields, or onboarding a new resource type for auditing.
tags:
  - frr
  - auditing
  - compliance
  - crud
  - audit-trail
---

# FRR Auditing Review

Conducts first-pass FRR auditing reviews for Intune features. Every applicable
review produces an **audit event catalog** - a structured enumeration of every
state-changing operation in the feature's code, annotated with current
audit-logging coverage - and a review verdict derived from it.

The catalog is the primary deliverable. The verdict is a derivative artifact
used for orchestrator rollup and ADO sign-off.

**Core mental model (apply in this order every time):**
> Does auditing apply? -> What entity changed? -> What category? -> What operation? -> What metadata proves it?

## Primary Deliverable: Audit Event Catalog

The catalog enumerates every state-changing operation in the feature's code,
each row annotated with: CRUD+ verb, affected resource, whether it currently
emits an audit event (Yes / No / Partial / N/A), the emitted or recommended
event name, sensitive-fields handling, and the source `file:line` for
traceability. Followed by summary stats and prioritized recommendations.

See [audit-event-catalog-template.md]($AGENCY_PLUGIN_DIR/skills/Auditing/references/audit-event-catalog-template.md)
for the canonical format and a worked example.

## Output Contract

Every run emits a standard JSON envelope. Finding IDs use the `AUD-N` convention
(e.g., `AUD-1`, `AUD-2`). The full catalog markdown goes into
the `adoCommentMarkdown` field:

```json
{
  "area": "auditing",
  "verdict": "pass | conditional | fail",
  "blockingFindings": [],
  "conditionalFindings": [],
  "summary": "<1-line summary>",
  "adoCommentMarkdown": "<full catalog markdown>",
  "instrumentation": {
    "skill": "frr-auditing",
    "version": "3.0.0",
    "applicabilityChecked": true,
    "applicabilityResult": true,
    "exitReason": "completed",
    "exitStep": 9,
    "operationsTotal": 0,
    "operationsAudited": 0,
    "operationsGap": 0,
    "operationsPartial": 0
  }
}
```

In **orchestrated mode** (`orchestrated: true` in input), return only the JSON
envelope and let the orchestrator post `adoCommentMarkdown` to ADO. In
**standalone mode**, also post the catalog as a discussion comment on the ADO
auditing task.

## When to Use

- A feature is onboarding new audit events into AuditingService / MessageBus
- A PR adds, modifies, or removes auditable actions in FSI, FSI Core, or a workload service
- A new audit category or resource type is being introduced
- Review is needed before FRR sign-off gate for PPv, PubPv, or GA
- Background service attribution, GraphSharedUtil tagging, or MessageBus permissions need validation

## What to Provide

| # | Input | Example |
|---|-------|---------|
| 1 | Repo link or PR | `https://dev.azure.com/msazure/Intune/_git/.../pullrequest/123` |
| 2 | Feature spec or design doc | ADO wiki link or attached markdown |
| 3 | Event list with CRUD+ mapping (**optional** - skill will generate from code) | Table: Operation -> CRUD+ verb -> Target type -> Category |
| 4 | Service / workload identity | FSI, FSI Core, workload service, or background task |
| 5 | Category and resource type mappings | `DeviceConfiguration`, `Compliance`, `Enrollment`, etc. |
| 6 | Output path | Where to save the catalog (standalone mode) |

## Reference Documentation

- **Auditing Onboarding.md** - `doc-Intune` wiki (canonical checklist for pre-review prep)
- [Audit Event Catalog Template]($AGENCY_PLUGIN_DIR/skills/Auditing/references/audit-event-catalog-template.md) - Canonical catalog format + worked example
- [CRUD+ Model]($AGENCY_PLUGIN_DIR/skills/Auditing/references/crud-plus-model.md) - Full taxonomy with verbs, usage rules, and consistency benchmarks
- [5-Gate Pass Model]($AGENCY_PLUGIN_DIR/skills/Auditing/references/five-gate-model.md) - Detailed gate conditions and verification methods
- [Auto-Fail Conditions]($AGENCY_PLUGIN_DIR/skills/Auditing/references/auto-fail-conditions.md) - 5 auto-fail conditions + 3 GA blockers
- [Onboarding Rules]($AGENCY_PLUGIN_DIR/skills/Auditing/references/onboarding-rules.md) - FSI/FSI Core paths, tagging, permissions
- [Code Review Targets]($AGENCY_PLUGIN_DIR/skills/Auditing/references/code-review-targets.md) - What to look for in code with search patterns
- [Metadata Schema]($AGENCY_PLUGIN_DIR/skills/Auditing/references/metadata-schema.md) - 6 mandatory fields with correct/incorrect examples

## Gotchas

- **Applicability is state-change driven** — If the feature does not add or change auditable state transitions, the gate is N/A and deeper review should stop.
- **Misclassification is gating** — Wrong category or resource-type mapping is a fail even if the event technically emits.
- **Every meaningful state change needs a row** — Missing CRUD+ coverage in the audit event catalog is itself a review finding.
- **Background tasks must be tagged correctly** — Background work that looks customer-visible or lacks the right GraphSharedUtil attribution can auto-fail onboarding hygiene.

## Procedure

### Step 1 - Applicability Check

Determine whether the auditing gate applies to this feature:

1. Does the feature add, change, or remove state-changing operations on
   customer-facing resources (create, update, delete)?
2. Does the feature change configuration that an admin can modify?
3. Does the feature affect data that needs CRUD+ audit trail (e.g., for
   compliance)?

**If all three are "no", the gate is N/A.** Emit the fast-path JSON envelope
below and exit. Do not run any further steps.

```json
{
  "area": "auditing",
  "verdict": "pass",
  "blockingFindings": [],
  "conditionalFindings": [],
  "summary": "Not applicable: Feature does not add or change state-changing operations or admin-configurable state",
  "adoCommentMarkdown": "## FRR Auditing - Not Applicable\n\nThis gate does not apply to this feature because there are no new or changed state-changing operations on customer-facing resources. Closing as N/A per the applicability check.",
  "instrumentation": {
    "skill": "frr-auditing",
    "version": "3.0.0",
    "applicabilityChecked": true,
    "applicabilityResult": false,
    "exitReason": "not-applicable",
    "exitStep": 1
  }
}
```

Otherwise the gate applies. Continue to Step 2.

### Step 2 - Identify the Onboarding Context

- What is being onboarded and why?
- **Service classification** (determines onboarding path - see [onboarding-rules.md]($AGENCY_PLUGIN_DIR/skills/Auditing/references/onboarding-rules.md)):
  - **FSI** -> use FSI onboarding steps
  - **FSI Core** -> use FSI Core onboarding steps (different package, permission, and tagging requirements)
  - **Workload service** -> validate correct describing-service ownership and attribution
  - **Background task** -> confirm it is not surfaced as a customer-visible event unless intentional
- Are events user-initiated, system-initiated, or background operations?

### Step 3 - Inventory State-Changing Operations (Build the Catalog)

This is the primary evidence-generation step. Enumerate every state-changing
operation in the feature's code and produce the rows of the audit event
catalog. The format and a worked example live in
[audit-event-catalog-template.md]($AGENCY_PLUGIN_DIR/skills/Auditing/references/audit-event-catalog-template.md).

**Catalog generation approach:**

Prefer **Bluebird MCP** (`bluebird-search_code`) when available - it indexes
the full Intune monorepo. Fall back to direct file reads when scoped to a
single PR. Search for these patterns:

| Layer | Search patterns |
|-------|----------------|
| ASP.NET controllers | `[HttpPost]`, `[HttpPatch]`, `[HttpDelete]`, `[HttpPut]` |
| Method naming | `Create*`, `Update*`, `Delete*`, `Patch*`, `Upsert*`, `*Async` variants |
| Service-layer writes | `DbContext.SaveChanges`, `SaveChangesAsync`, repository setters |
| CosmosDB | `UpsertItemAsync`, `ReplaceItemAsync`, `DeleteItemAsync`, `CreateItemAsync` |
| Azure Storage | `SaveAsync`, blob `UploadAsync`, table `InsertOrReplaceAsync` |
| MessageBus | Publishers that mutate state downstream |

For each operation found, record: Operation (route + verb OR fully-qualified
method name), CRUD+ verb, affected resource, source `file:line`. Coverage
columns are filled in Step 5 after taxonomy validation.

Also cross-check the spec: any documented state change in the feature design
that did not show up in code search is a candidate gap row - add it with
`No` coverage and a recommended event name.

### Step 4 - Determine the Primary Resource and Taxonomy

**Misclassification is a gating issue - fail immediately if wrong.**

- Identify the primary target / resource type for each catalog row
- Validate the selected audit **category** (`DeviceConfiguration`, `Compliance`, `Enrollment`, `RoleAdministration`, etc.)
- Confirm the category matches the **customer's mental model** and existing Intune taxonomy
- If a **new category** is introduced: verify the UX repo is updated (see [onboarding-rules.md]($AGENCY_PLUGIN_DIR/skills/Auditing/references/onboarding-rules.md))
- Consistency benchmark: similar actions across workloads must produce similar naming

### Step 5 - Annotate Catalog with Current Audit Coverage

For each row from Step 3, determine current audit-logging coverage by
searching near the operation for audit emission calls:

- `AuditingService.LogEvent`, `AuditingService.LogAsync`
- `Auditing.LogEvent`, `EmitAuditEvent`, `PublishAuditLog`
- `MessageBus.PublishAsync` to the auditing topic
- Workload-specific audit helpers (e.g., `WorkloadAuditWriter`)

Classify each row:

| Coverage | Definition | Catalog action |
|----------|-----------|----------------|
| **Yes** | Audit event emitted with complete metadata | Record event name in **Emitted event name** column |
| **No** | No audit event found | Suggest event name in **Recommended event name** using `{ResourceType}_{Verb}` per [crud-plus-model.md]($AGENCY_PLUGIN_DIR/skills/Auditing/references/crud-plus-model.md). This is a **gap**. |
| **Partial** | Event emitted but missing mandatory metadata, OR PATCH using the ID/Name-only overload | Note the specific gap. Treat as a blocker for GA. |
| **N/A** | Operation does not change customer-visible state | Record reason briefly |

Flag the **Sensitive fields handled?** column for any operation touching PII
or sensitive config (see [code-review-targets.md]($AGENCY_PLUGIN_DIR/skills/Auditing/references/code-review-targets.md)
for PII detection signals).

Apply the [CRUD+ Auditing Model]($AGENCY_PLUGIN_DIR/skills/Auditing/references/crud-plus-model.md) when validating
verb selection (Patch vs Update, when to use `+` verbs, etc.).

### Step 6 - Validate Event Structure and Mandatory Metadata

All [6 mandatory fields]($AGENCY_PLUGIN_DIR/skills/Auditing/references/metadata-schema.md) are required: Actor,
Timestamp, Target resource info, Action name, Application/service name,
Outcome/result. Any missing, ambiguous, placeholder, or non-customer-readable
field downgrades the row from `Yes` to `Partial` and is an auto-fail at the
gate level.

### Step 7 - Apply Auditing Guardrails and Quality Checks

- **Naming hygiene** - action names must be stable, deterministic, and follow existing patterns. No dynamic strings.
- **String sanitation** - no PII leakage. No unsanitized inputs in audit fields.
- **Noise control** - background polling/heartbeats must NOT surface as customer events.
- **Failure signaling** - failure events must be intentional, not accidentally omitted or misleading.

### Step 8 - Validate Onboarding Completeness

Verify all items per [onboarding-rules.md]($AGENCY_PLUGIN_DIR/skills/Auditing/references/onboarding-rules.md):
correct FSI/FSI Core path, GraphSharedUtil tags, MessageBus publisher
permissions, UX repo update (if new category), and end-to-end event flow
through ingestion -> storage -> Auditing page.

### Step 9 - Emit Catalog + Verdict

Assemble the final catalog with these three sections (template:
[audit-event-catalog-template.md]($AGENCY_PLUGIN_DIR/skills/Auditing/references/audit-event-catalog-template.md)):

1. **Summary stats** - 4 counters: total / audited / gap / partial
2. **Catalog table** - all rows from Step 3 with coverage from Step 5
3. **Prioritized recommendations** - highest-risk gaps first:
   `Delete` > `Update/Patch` > `Create` > sensitive `Read`. Each
   recommendation includes the source `file:line`, recommended event name,
   and the specific gap (missing event, wrong overload, missing metadata,
   etc.). Close with a 2-3 line reminder pointing at
   [crud-plus-model.md]($AGENCY_PLUGIN_DIR/skills/Auditing/references/crud-plus-model.md) for naming conventions.

**Verdict mapping:**

| Catalog state | Verdict |
|---------------|---------|
| All operations `Yes` with complete metadata | `pass` |
| Any `No` or `Partial` rows, no auto-fail | `conditional` |
| Any [auto-fail condition]($AGENCY_PLUGIN_DIR/skills/Auditing/references/auto-fail-conditions.md) triggered | `fail` |

**Orchestrated mode (`orchestrated: true` in input):** return the JSON
envelope with the full catalog markdown in `adoCommentMarkdown`. Do not post
to ADO - the orchestrator handles that.

**Standalone mode:** post the catalog as a discussion comment (not
description) on the ADO auditing task. Then emit the JSON envelope for
instrumentation. Update work item state: Pass -> Resolved; Conditional ->
Active with comment; Fail -> Active and tag the feature DRI.

## Quick Reference Tables

### 5-Gate Pass Model

> Full details: [five-gate-model.md]($AGENCY_PLUGIN_DIR/skills/Auditing/references/five-gate-model.md)

| Gate | Pass Condition |
|------|---------------|
| **1 - CRUD+ Coverage** | Every meaningful action mapped; coverage complete; consistent patterns |
| **2 - Metadata** | All 6 mandatory fields present, human-readable, deterministic |
| **3 - Taxonomy** | Category and resource type match customer-visible action and Intune taxonomy |
| **4 - Onboarding** | Correct FSI/FSI Core path; GraphSharedUtil tags; MessageBus permissions; UX repo |
| **5 - Governance** | Customer-safe; stable names; sanitized strings; intentional failure signaling; no noise |

### Auto-Fail Conditions

> Full details with examples and remediation: [auto-fail-conditions.md]($AGENCY_PLUGIN_DIR/skills/Auditing/references/auto-fail-conditions.md)

1. Missing impactful auditable actions or incomplete CRUD+ coverage
2. Missing, ambiguous, placeholder, or non-customer-readable mandatory metadata
3. Incorrect category or resource type; internal-only event surfaced to customers
4. Guardrail violations: unstable naming, unsanitized strings, PII leakage, noisy background events
5. Incomplete onboarding: missing GraphSharedUtil tags, MessageBus permissions, or UX repo update

**3 GA Non-Negotiable Blockers:** Incomplete CRUD+ on externally visible ops, uncorrelatable events, known-bad audit patterns (PATCH overload misuse, generic Read auditing). See [auto-fail-conditions.md]($AGENCY_PLUGIN_DIR/skills/Auditing/references/auto-fail-conditions.md).

### Release Phase Guidance

| Phase | Quality Bar |
|-------|------------|
| **PPv** | Catalog complete; CRUD+ mapping intentional; metadata schema defined; no blocking gaps |
| **PubPv** | Catalog shows full coverage end-to-end; onboarding checklist completed; naming finalized |
| **GA** | All 5 gates pass; all 3 GA non-negotiables resolved; no `No` or `Partial` rows for customer-visible ops |

## Error Handling

| Scenario | Action |
|----------|--------|
| No repo or PR link provided | Block review; code analysis is required for the catalog |
| Bluebird MCP unavailable | Fall back to direct file reads scoped to PR; note reduced coverage in summary |
| Event list missing CRUD+ mapping | Generate from code search; the catalog IS the mapping |
| FSI vs FSI Core unclear | Ask once; default to FSI Core rules (stricter) |
| New category with no UX repo update | Auto-fail Gate 4 |
| Action names are dynamic strings | Auto-fail Gate 5; provide stable-name pattern |
| Background service events in customer-facing surface | Auto-fail Gate 5 |
| Metadata contains opaque internal IDs | Auto-fail Gate 2 (row drops from `Yes` to `Partial`) |

## Feedback and Iteration

**Version**: 3.0.0
**Last updated**: 2026-05-18
**Feedback**: Post in [Generative AI Discussions](https://teams.microsoft.com/l/channel/19%3Ab6602869391c46539a2c24b712f004d3%40thread.skype/Generative%20AI%20Discussions?groupId=9f67747e-53cb-4d4f-8a60-e849ef90fac0) or file a bug under `Intune\Mgmt\Quality Analytics and Infrastructure\AIOps` with tag `copilot-skill`.

### Instrumentation

After execution, append to `~/.copilot/skill-usage.ndjson`:

```json
{ "skill": "frr-auditing", "version": "3.0.0", "ts": "<ISO-8601>", "feature": "<name>", "adoTaskId": "<id>", "applicabilityResult": true, "verdict": "pass|conditional|fail", "operationsTotal": 0, "operationsAudited": 0, "operationsGap": 0, "operationsPartial": 0, "gateFailures": [], "autoFailConditions": [], "durationMs": 0 }
```

### Known Limitations

- Cannot directly query the `doc-Intune` wiki - users must supply onboarding checklist artifact manually
- GraphSharedUtil tag validation requires a repo link; cannot infer tags from spec alone
- MessageBus namespace verification requires access to the publisher configuration file
- Cross-category consistency benchmarking is heuristic - always cross-reference the Intune auditing wiki
- Catalog completeness depends on code-search coverage; service-layer mutations called through interfaces or dynamic dispatch may be missed - cross-check against the feature spec for documented state changes that did not surface

### Changelog

- 3.0.0 - Evidence-generator pivot. The primary deliverable is now the audit event catalog (a row-per-operation table annotated with current audit-logging coverage and source file:line), not just a verdict. Added explicit Step 1 applicability check with fast-path N/A JSON envelope. Standardized JSON envelope output contract for orchestrated mode with `applicabilityChecked` / `applicabilityResult` / `exitReason` / catalog counters in instrumentation. Added [audit-event-catalog-template.md]($AGENCY_PLUGIN_DIR/skills/Auditing/references/audit-event-catalog-template.md) with worked example. Procedure expanded from 8 to 9 steps: applicability gate added, inventory step now produces the catalog as primary artifact, new annotation step folds current-coverage signal into each row.
- 2.0.0 - Refactored to progressive disclosure with `references/` subdirectory. SKILL.md is lean procedure flow; detailed knowledge in 6 reference files (crud-plus-model, five-gate-model, auto-fail-conditions, onboarding-rules, code-review-targets, metadata-schema).
- 1.0.0 - Initial publication. Grounded in captured FRR area expert review philosophy.


---

# Source B - Matt/Cyrus Automation skill

> Preserve this source as the pattern-rubric, tiering, and ADO/PR scorecard knowledge base.

# Auditing FRR Gate — AI Evaluation Skill

> **Skill ID:** Auditing-FRR-Skill v1.0
> **Gate:** Auditing
> **Stage Gate:** PuP (Public Preview)
> **Gate Type:** Self
> **Gate Coordinator:** Auditing Workstream Owner (see ADO Feature Task AssignedTo field)
> **Pattern Source:** `Auditing-FRR-Pattern.md`
> **Skill Author:** Cyrus (FRR Agent)
> **Last Updated:** 2025-07-14

---

## How to Invoke This Skill

This skill is invoked by the FRR Agent when evaluating a feature's Auditing gate readiness.

**Invocation context required:**
- Feature name and ADO Feature Task ID
- PR URL (if available)
- Tier classification (1 / 2 / 3)
- Access to FRR task comments, linked artifacts, and ADO subtasks
- ADO Feature Task AssignedTo field (identifies the Auditing Workstream owner)

> ⚠️ **Self-Close Exception:** This gate may be marked Not Applicable (➖) **only if** the feature exclusively performs read-only GET operations that do NOT involve secure strings or actions, and this determination is **explicitly documented** in the FRR task. A feature cannot self-close with a blanket "N/A" without a written scope determination. If no scope determination is present, treat as **in scope** and flag P-AUDIT-04.

---

## Section 1 — Gate Scope and Ownership

| Field | Value |
|-------|-------|
| Gate Name | Auditing |
| Stage Required | PuP (Public Preview) |
| Gate Type | Self |
| Gate Coordinator | Auditing Workstream Owner (identified in ADO Feature Task AssignedTo field) |
| GC Contact Method | Joint portal review session (Tenant Administration > Audit Logs) |
| Escalation Path | Auditing Workstream Owner → Intune Auditing Platform Team → FRR Program Manager |
| Self-Close Allowed? | Yes — only for read-only GET features with documented scope determination |
| Pattern Source | `Auditing-FRR-Pattern.md` |

**Gate Purpose:**
Ensure features that create, modify, or delete Intune configuration — or that impact IT admin or end-user experience — emit appropriate audit events in the Intune audit log. Gate close requires a joint portal review session with the Auditing Workstream owner at Tenant Administration > Audit Logs. Deprecating features must maintain audit logging until full retirement with 180+ day post-deprecation retention.

---

## Section 2 — Evaluation Steps

### Step 1 — Gather Context
Retrieve the following before evaluating:
- Feature name and ADO Feature Task ID
- FRR task and all subtasks (look for Auditing-specific subtasks)
- FRR task comments (scope determinations, portal review session notes, Workstream owner sign-off)
- PR description and linked work items (look for audit logging code changes)
- ADO Feature Task AssignedTo field (identifies the Auditing Workstream owner)
- Whether the feature is an active new feature, a modification, or a deprecating feature

### Step 2 — Audit Scope Triage

Before applying any pattern rubrics, determine whether the feature is in scope for audit logging. Use the following criteria:

**In-scope signals (any one = in scope):**
- Creates, reads (secure strings/actions), updates, or deletes Intune configuration resources
- Adds or removes scope tag or group assignments
- Performs Graph API PUT, POST, PATCH, or DELETE operations
- Performs Graph API GET on secure strings or actions (e.g., password retrieval, device wipe actions)
- Impacts IT admin experience (new portal workflows, policy management, enrollment flows)
- Impacts end-user experience through MDM or MAM operations
- Is a deprecating feature that previously had audit logging

**Out-of-scope examples:**
- Internal microservice-to-microservice communication with no customer-facing configuration impact
- Engineer deployment pipelines and build/test infrastructure
- Non-customer-facing data (telemetry, internal diagnostics, logs not visible in Tenant Admin > Audit Logs)
- Read-only GET operations on non-sensitive resources (no secure strings, no action side effects)

**Triage outcome:**
- **If all operations are read-only GET (non-sensitive) and explicitly documented** → Feature is out of scope. Self-close is valid with documented determination. Emit a 🟢 Pass with P-AUDIT-01, P-AUDIT-02, P-AUDIT-03, and P-AUDIT-05 marked ➖. P-AUDIT-04 must still be scored (cannot be ➖).
- **If any in-scope signal is present** → Feature is in scope. Proceed to Step 3 and evaluate all applicable patterns.

**Audit Event Types Reference:**

| Event Type | When to Use |
|------------|-------------|
| `Create` | New resource created (policy, app, profile, enrollment) |
| `Delete` | Resource deleted |
| `Update`/`Patch` | Existing resource modified (any field change) |
| `SetReference` | Scope tag or group assignment added |
| `RemoveReference` | Scope tag or group assignment removed |
| `Action` | Generic action event (e.g., Retire, Wipe, Sync, Lock) |

**Minimum mandatory events:** All Graph API PUT/POST/PATCH/DELETE operations + GET for secure strings/actions must emit corresponding audit events.

**Per-event required fields:**
- Properties (what changed)
- Actor Details: AccountId + applicationId (for app-only calls) or userId (for delegated calls)
- Action Details (what operation was performed)

### Step 3 — Apply Pattern Rubrics
Evaluate each applicable pattern (based on tier and triage from Steps 1–2) using the rubrics in Section 3.
For each pattern, assign one score:
- ✅ Met — evidence clearly demonstrates the requirement is satisfied
- ⚠️ Partial / Unclear — some evidence present but incomplete, ambiguous, or unverifiable
- ❌ Not Met — evidence is absent, requirement is explicitly unmet, or artifact is missing
- ➖ Not Applicable — pattern excluded for this tier or by valid self-close exception (document why)

### Step 4 — Capture Evidence
For each pattern, record:
- What artifact or code element you found (or failed to find)
- Specific reference: ADO task ID, SharePoint link, FRR task comment, PR line reference
- Reason for the score assigned
Do not assign ✅ without citing specific evidence. "No issues found" is not evidence.

### Step 5 — Aggregate and Emit Verdict
Apply verdict logic from Section 6. Emit the final scorecard and gate verdict.

---

## Section 3 — Pattern Rubrics

### P-AUDIT-01 — Required Audit Events Missing

**What the agent looks for:**
All in-scope operations must emit the correct audit event types in Tenant Administration > Audit Logs. The minimum mandatory set includes all CRUD events (Create, Delete, Update/Patch), reference events (SetReference, RemoveReference), and action events (Action) for applicable operations. Look for evidence from the joint portal review session showing all expected event types are present and visible.

**Scoring:**
| Score | Condition |
|-------|-----------|
| ✅ | All required audit event types confirmed present in Tenant Administration > Audit Logs during joint portal review; Workstream owner signed off |
| ⚠️ | Most required events present; 1+ event type missing or incomplete; fix identified and in progress |
| ❌ | Audit events absent or substantially incomplete; gate closed without evidence of portal review |
| ➖ | Feature is legitimately out of scope (all operations are read-only GET on non-sensitive resources, documented in Step 2) |

**Evidence to capture:**
- Is there a portal review session record showing audit events were verified in Tenant Administration > Audit Logs?
- Are all expected event types (Create, Delete, Update, SetReference, RemoveReference, Action) present for applicable operations?
- Did the Workstream owner confirm the events are correct and complete?

**Tier Applicability:** Tier 1 ✅ · Tier 2 ✅ · Tier 3 ✅ (if in scope)

---

### P-AUDIT-02 — Actor Details Incomplete

**What the agent looks for:**
Every audit event must include Actor Details: AccountId plus either applicationId (for service principals/app-only calls) or userId (for delegated user calls). Events that log the action but omit the actor identity are non-compliant. Check both the portal review evidence and any engineering documentation about event schema.

**Scoring:**
| Score | Condition |
|-------|-----------|
| ✅ | All audit events confirmed to include AccountId + applicationId or userId in Actor Details |
| ⚠️ | Actor details present for most event types; gaps identified for specific event types (e.g., Action events missing userId) |
| ❌ | Actor details systematically absent from audit events (AccountId, applicationId, userId all missing or null) |
| ➖ | Feature is out of scope (documented in Step 2) |

**Evidence to capture:**
- During portal review, were Actor Details visible on all logged events?
- Are AccountId and applicationId/userId fields populated on all event types?
- Are there any event types where actor attribution is missing or ambiguous?

**Tier Applicability:** Tier 1 ✅ · Tier 2 ✅ · Tier 3 ✅ (if in scope)

---

### P-AUDIT-03 — Portal Review Session Not Completed

**What the agent looks for:**
Gate close for the Auditing gate requires a joint portal review session between engineering and the Auditing Workstream owner, conducted in Tenant Administration > Audit Logs. Engineering cannot close this gate unilaterally. Look for evidence that the joint session occurred: a comment from the Workstream owner, a session record, or the gate subtask closed by the Workstream owner (not self-closed by engineering).

**Scoring:**
| Score | Condition |
|-------|-----------|
| ✅ | Joint portal review session completed; Auditing Workstream owner commented and closed/signed off on the gate task |
| ⚠️ | Portal review session booked but not yet completed; OR session completed but Workstream owner sign-off pending |
| ❌ | No portal review session found; gate was closed by engineering alone without Workstream owner involvement |
| ➖ | Feature is out of scope (documented in Step 2) |

**Evidence to capture:**
- Is there a comment from the Auditing Workstream owner (identified from ADO AssignedTo field) in the FRR task?
- Was the gate subtask closed by the Workstream owner or by an engineering team member?
- Is there any record of a joint Tenant Administration > Audit Logs review session?

**Tier Applicability:** Tier 1 ✅ · Tier 2 ✅ · Tier 3 ✅ (if in scope)

---

### P-AUDIT-04 — Audit Scope Incorrectly Self-Assessed as Out of Scope

**What the agent looks for:**
Every feature must make an explicit, documented scope determination for the Auditing gate — regardless of whether the feature is actually in scope. This pattern catches features that self-closed the Auditing gate without documenting their reasoning, or features that demonstrably perform in-scope operations but were self-closed as out of scope.

> ⚠️ **This pattern CANNOT be ➖.** Every feature must have a documented scope determination. The absence of a determination is itself a finding.

**Scoring:**
| Score | Condition |
|-------|-----------|
| ✅ | Scope determination is explicitly documented in the FRR task or comments, AND is consistent with observable operations (in-scope operations → full evaluation; read-only GET only → out-of-scope documented) |
| ⚠️ | Scope is ambiguous — possible in-scope operations (PUT/POST/PATCH/DELETE or secure strings) are present but not formally evaluated; scope determination is missing or incomplete |
| ❌ | Feature demonstrably performs in-scope operations (per Step 2 signals) but was self-closed as out of scope without documented rationale |

**Evidence to capture:**
- Is there an explicit written determination in the FRR task or comments stating whether the feature is in scope for audit logging?
- If the gate was self-closed as out of scope, does the reasoning match the Step 2 triage criteria?
- Are there PR changes or feature descriptions indicating PUT/POST/PATCH/DELETE operations that contradict an out-of-scope claim?

**Tier Applicability:** Tier 1 ✅ · Tier 2 ✅ · Tier 3 ✅

---

### P-AUDIT-05 — Deprecating Feature — Audit Logging Removed Prematurely

**What the agent looks for:**
For features being deprecated, audit logging must continue until the feature is fully retired. Additionally, the audit log data must be retained for at least 180 days after the deprecation date. This pattern is only applicable when there are signals that the feature is being deprecated or sunset.

**Scoring:**
| Score | Condition |
|-------|-----------|
| ✅ | Active feature: audit logging complete and verified. Deprecating feature: audit logging continues until full retirement AND retention configured for 180+ days post-deprecation |
| ⚠️ | Deprecation plan documented but retention configuration for 180+ days has not been confirmed; OR logging planned to continue but timeline is unclear |
| ❌ | Audit logging removed or planned to be removed while customers are still active on the feature; OR post-deprecation retention period is less than 180 days |
| ➖ | Active feature with no deprecation signal present |

**Evidence to capture:**
- Is the feature being deprecated or sunset (look for deprecation notices, sunset timelines, or FRR task comments)?
- If deprecating, is audit logging confirmed to continue until the feature is fully retired?
- What is the configured data retention period for audit logs after deprecation?

**Tier Applicability:** Tier 1 ✅ · Tier 2 ✅ · Tier 3 ✅ (if deprecation signal present) · ➖ (active feature with no deprecation)

---

## Section 4 — Tier-Scoping Table

| Pattern ID | Pattern Name | Tier 1 | Tier 2 | Tier 3 |
|-----------|--------------|--------|--------|--------|
| P-AUDIT-01 | Required Audit Events Missing | ✅ | ✅ | ✅ |
| P-AUDIT-02 | Actor Details Incomplete | ✅ | ✅ | ✅ |
| P-AUDIT-03 | Portal Review Session Not Completed | ✅ | ✅ | ✅ |
| P-AUDIT-04 | Audit Scope Incorrectly Self-Assessed as Out of Scope | ✅ | ✅ | ✅ |
| P-AUDIT-05 | Deprecating Feature — Audit Logging Removed Prematurely | ✅ | ✅ | ✅ |

> **Tier 3 minimum set:** P-AUDIT-04 (scope determination required). P-AUDIT-01, P-AUDIT-02, P-AUDIT-03 apply only if in scope. P-AUDIT-05 applies only if a deprecation signal is present.

> **Tier 3 escalation rule:** If Step 2 triage reveals ANY in-scope signal for a Tier 3 feature, escalate to Tier 2.

---

## Section 5 — Scorecard Template

```
# Auditing FRR Gate — Scorecard
Feature: [Feature Name / ADO Feature Task ID]
PR: [PR URL]
Tier: [1 / 2 / 3]
Audit Scope: [In Scope / Out of Scope / Deprecating]
Auditing Workstream Owner: [Name from ADO Feature Task AssignedTo field]
Evaluator: FRR Agent (Cyrus)
Date: [YYYY-MM-DD]

Audit Scope Triage Summary:
  Performs PUT/POST/PATCH/DELETE operations: [Yes / No]
  Performs GET on secure strings or actions: [Yes / No]
  Impacts IT admin portal experience:        [Yes / No]
  Impacts end-user MDM/MAM experience:       [Yes / No]
  Feature is deprecating:                    [Yes / No]

| Pattern ID  | Pattern Name                                              | Score | Evidence Summary |
|------------|-----------------------------------------------------------|-------|-----------------|
| P-AUDIT-01 | Required Audit Events Missing                             |       |                 |
| P-AUDIT-02 | Actor Details Incomplete                                  |       |                 |
| P-AUDIT-03 | Portal Review Session Not Completed                       |       |                 |
| P-AUDIT-04 | Audit Scope Incorrectly Self-Assessed as Out of Scope     |       |                 |
| P-AUDIT-05 | Deprecating Feature — Audit Logging Removed Prematurely   |       |                 |

Totals:
  ✅ Met:           __
  ⚠️ Partial:       __
  ❌ Not Met:        __
  ➖ Not Applicable: __

Gate Verdict: [🟢 / 🟡 / 🔴]
```

---

## Section 6 — Gate Verdict Logic

### Hard Blockers (auto-🔴)
| Pattern | Hard Blocker Rationale |
|---------|----------------------|
| P-AUDIT-01 | Audit events are a customer-facing compliance requirement. An in-scope feature shipping to Public Preview without audit logging puts customers at regulatory and compliance risk. |

> **Special case:** If Step 2 triage confirms the feature is out of scope AND P-AUDIT-04 = ✅ (scope determination documented), emit 🟢 Pass with P-AUDIT-01, P-AUDIT-02, P-AUDIT-03, and P-AUDIT-05 marked ➖.

### Verdict Rules
```
IF feature is out of scope AND P-AUDIT-04 = ✅ → 🟢 Pass (self-close valid)
IF feature is out of scope AND P-AUDIT-04 = ⚠️ or ❌ → 🔴 Blocked (scope determination missing or contradicted)
IF P-AUDIT-01 = ❌ → 🔴 Blocked (state hard blocker + remediation: implement required audit events before milestone)
ELSE IF any remaining pattern = ❌ → 🔴 Blocked
ELSE IF ⚠️ count ≥ 3 → 🟡 Conditional Pass (multiple gaps, Workstream owner review before milestone)
ELSE IF ⚠️ count is 1–2 → 🟡 Conditional Pass (minor gaps, Workstream owner may accept with risk acknowledgment)
ELSE (all ✅ or ➖) → 🟢 Pass
```

### Verdict Summary Format
```
Gate Verdict: 🔴 / 🟡 / 🟢
Hard blockers triggered: [list or "None"]
Patterns failed (❌): [list or "None"]
Patterns partial (⚠️): [list or "None"]
Audit Scope Determination: [In Scope / Out of Scope / Deprecating]
Portal Review Status: [Completed / Booked / Not Scheduled / N/A]
Recommended next steps: [specific actions]
```

---

## Section 7 — ADO Query References

Use these queries to retrieve relevant ADO work items when evaluating this gate.

> **Note:** Tag-based queries in the current ADO environment may return 0 results due to tag propagation quirks. If a query returns empty, check the parent Feature task directly and scan all subtasks and comments manually. Also check the ADO Feature Task AssignedTo field to identify the Auditing Workstream owner.

### Query A — Auditing FRR Gate Tasks

```
Work Item Type: Task
Tags: Management.FRR.Auditing
Parent: [Feature Task ID]
State: Active OR Resolved OR Closed
```

**Purpose:** Retrieves all FRR subtasks tagged for the Auditing gate — used to locate the portal review session task, audit event verification tasks, and scope determination documentation (populates P-AUDIT-01, P-AUDIT-02, P-AUDIT-03).

---

### Query B — Audit Event Implementation and Bugs

```
Work Item Type: Task OR Bug
Tags: Management.FRR.Auditing
Title contains: "Audit" OR "Audit Event" OR "Audit Log"
Parent: [Feature Task ID]
State: Active OR Resolved OR Closed
```

**Purpose:** Narrows to audit event implementation tasks and any bugs filed against audit logging gaps (populates P-AUDIT-01 and P-AUDIT-02).

---

### Query C — Portal Review Sign-Off Tasks

```
Work Item Type: Task
Tags: Management.FRR.Auditing
Title contains: "Portal Review" OR "Audit Review" OR "Tenant Administration"
Parent: [Feature Task ID]
State: Active OR Resolved OR Closed
```

**Purpose:** Locates the joint portal review session task and Workstream owner sign-off evidence (populates P-AUDIT-03).

---

### Query D — Audit Scope Determination and Deprecation Tasks

```
Work Item Type: Task OR Feature
Tags: Management.FRR.Auditing
Title contains: "Auditing" OR "Scope" OR "Deprecat"
Parent: [Feature Task ID]
State: Any
```

**Purpose:** Broad query to surface audit scope determination documentation and deprecation-related audit logging decisions (populates P-AUDIT-04 and P-AUDIT-05).

---

## Section 8 — Regression Test Case (Known-Bad Feature)

This is a **known-bad feature scenario** that should always score 🔴 when this skill is applied correctly. Use it to verify the skill is working as intended.

### Regression Feature: "Intune Policy Bulk Delete" — Auditing Gate Failure

**Description:**
- Engineering team built a bulk delete capability allowing IT admins to delete multiple compliance policies at once
- The feature performs DELETE Graph API operations on multiple resources in a single user action
- No audit events are emitted when policies are deleted via the bulk delete operation (a logging gap introduced by the batch implementation)
- The FRR task Auditing subtask was self-closed by an engineer with the comment "Works same as single delete"
- No Auditing Workstream owner was identified in the ADO task
- No joint portal review session was conducted
- No scope determination was documented in the FRR task
- Actor details are absent from the few existing delete events inherited from the legacy single-delete path

> **Note:** No real ADO bug IDs are available for this scenario. This is a constructed known-bad case. When real Auditing gate bug IDs become available from the Auditing Workstream owner, update this section with actual ADO references. Query tag: `Management.FRR.Auditing`

**Expected Skill Output:**
```
# Auditing FRR Gate — Scorecard
Feature: Intune Policy Bulk Delete / [TBD]
PR: [TBD]
Tier: 1
Audit Scope: In Scope
Auditing Workstream Owner: [Not identified in ADO task]
Evaluator: FRR Agent (Cyrus)
Date: [YYYY-MM-DD]

Audit Scope Triage Summary:
  Performs PUT/POST/PATCH/DELETE operations: Yes — bulk DELETE operations
  Performs GET on secure strings or actions: No
  Impacts IT admin portal experience:        Yes — new portal workflow
  Impacts end-user MDM/MAM experience:       No
  Feature is deprecating:                    No

| Pattern ID  | Pattern Name                                              | Score | Evidence Summary |
|------------|-----------------------------------------------------------|-------|-----------------|
| P-AUDIT-01 | Required Audit Events Missing                             | ❌    | Bulk delete emits no audit events; Delete event type absent |
| P-AUDIT-02 | Actor Details Incomplete                                  | ❌    | Actor details absent on legacy delete events |
| P-AUDIT-03 | Portal Review Session Not Completed                       | ❌    | Self-closed by engineer; no Workstream owner sign-off |
| P-AUDIT-04 | Audit Scope Incorrectly Self-Assessed as Out of Scope     | ❌    | No scope determination documented; self-close without rationale |
| P-AUDIT-05 | Deprecating Feature — Audit Logging Removed Prematurely   | ➖    | Active feature, no deprecation signal |

Totals:
  ✅ Met:           0
  ⚠️ Partial:       0
  ❌ Not Met:        4
  ➖ Not Applicable: 1

Gate Verdict: 🔴 Blocked
Hard blockers triggered: P-AUDIT-01 (required audit events missing — bulk DELETE not logging)
Patterns failed (❌): P-AUDIT-01, P-AUDIT-02, P-AUDIT-03, P-AUDIT-04
Audit Scope Determination: In Scope — DELETE operations confirmed
Portal Review Status: Not Scheduled
Recommended next steps: (1) Implement Delete audit events for bulk delete operation; (2) Add actor details to all event types;
  (3) Identify Auditing Workstream owner in ADO task; (4) Schedule joint portal review session;
  (5) Document scope determination in FRR task
```

**Skill Validation Rule:**
If you apply this skill to this known-bad scenario and the output is NOT 🔴 Blocked with at least P-AUDIT-01 flagged ❌, the skill rubrics need recalibration. Contact the Auditing Workstream owner for review.

---

## Section 9 — Usage Notes

### For the FRR Agent
- **Portal review is non-optional.** The Auditing gate CANNOT be closed by engineering alone. The joint portal review session with the Auditing Workstream owner is a hard requirement. If there is no evidence of the Workstream owner's involvement, flag P-AUDIT-03 as ❌ regardless of how complete the audit events appear to be.
- **Identify the Workstream owner from the ADO AssignedTo field.** The Auditing gate does not have a single named GC like RAI or CELA — the responsible party is whoever is assigned the Auditing gate task in ADO. Always check the AssignedTo field first and name them in your output.
- **Actor details are required on EVERY event.** AccountId + (applicationId or userId) must be present on every audit event. Missing actor details on even a subset of event types is a P-AUDIT-02 finding.
- **Deprecation retention: 180 days minimum.** If any deprecation signal is present (sunset notice, end-of-support date, migration guidance), check that retention is configured for 180+ days after the deprecation date — not just 180 days from feature launch.
- **GET-only features are out of scope — with two exceptions.** Read-only GET operations on non-sensitive resources do not require audit logging. The exceptions are: (1) GET operations that retrieve secure strings (passwords, keys, certificates) and (2) GET operations that trigger action side effects (e.g., getting a device's BitLocker recovery key). These are in scope.
- **Scope determination is always required.** Even if a feature is clearly in scope (e.g., CRUD operations on policies), the FRR task must include a written scope determination. A feature that obviously should be audited but has no documentation is still a P-AUDIT-04 finding.

### For Gate Owners Reviewing Skill Output
This skill is AI-generated from pattern mining of the Auditing gate failure catalogue (`Auditing-FRR-Pattern.md`) and the canonical Security-FRR-Skill.md template. It is a **draft** until the Auditing Workstream owner reviews and signs off on the rubric thresholds and evidence requirements.

Feedback format:
- **Pattern ID** — which rubric needs updating
- **Issue** — what's wrong with the current threshold or condition
- **Correction** — the correct threshold or evidence requirement

Known gaps to address in v1.1:
- Full list of GET operations that qualify as "secure strings or actions" — need definitive list from the Auditing platform team.
- What is the exact portal path for the joint review session? Tenant Administration > Audit Logs confirmed — but is there a specific view or filter that must be used?
- For batch/bulk operations (e.g., bulk delete), does each individual operation need its own audit event, or is one aggregate event sufficient?
- What is the source of truth for the Auditing Workstream owner — always ADO AssignedTo field, or is there a roster document?

---

## Appendix — Pattern-to-Gate Mapping

| Pattern ID | Gate Phase | Required for Milestone? | Evidence Type | Blocker Level |
|-----------|------------|------------------------|---------------|---------------|
| P-AUDIT-01 | PuP | Yes (if in scope) | Portal review session record + Workstream owner sign-off | ⛔ Hard |
| P-AUDIT-02 | PuP | Yes (if in scope) | Actor details confirmed in portal review | ⚠️ Soft |
| P-AUDIT-03 | PuP | Yes (if in scope) | Joint portal review session record | ⚠️ Soft |
| P-AUDIT-04 | PuP | Yes (all features) | Written scope determination in FRR task | ⚠️ Soft |
| P-AUDIT-05 | PuP | Yes (if deprecating) | Retention configuration + deprecation timeline | ⚠️ Soft |

---

*Generated by Cyrus v1.0 | Skill derived from `Auditing-FRR-Pattern.md` | Methodology: Matt Call 3-prompt FRR Automation workflow (documented in `03-Skill-Generation-Documentation.md`)*
