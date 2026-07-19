# ADR-0005: Adopt the Terraform template as the direct inheritance parent

| Field | Value |
|-------|-------|
| Status | proposed |
| Date | 2026-07-19 |
| Deciders | repository owner |
| Author | Codex (AI agent) |
| Supersedes / Superseded by | Applies ADR-0004 to this repository; supersedes the direct `ai-dev-foundation` template-sync source |

## Context

`secure-ai-controls` was created directly from `ai-dev-foundation` at commit
`0f7da7f` on 2026-07-16 18:52 JST. It has no inheritance manifest or accepted-parent
lock, and its scheduled sync workflow still names the foundation directly. The intended
topology is instead:

`ai-dev-foundation -> terraform-gcp-template -> secure-ai-controls`

Bypassing the intermediate template prevents this repository from receiving the
Terraform-family governance profile, always-reported IaC checks, workflow runtime
updates, and later foundation-document namespace through the direct-parent process
accepted in ADR-0004. The current tree also predates the documentation ownership rule:
foundation-authored guides occupy `docs/`, where repository-authored Japanese product
documents should live.

The migration must preserve local security-control and product work, keep each review
under the repository size limit, avoid workflow-writing credentials in Actions, and
perform no live GitHub, Terraform, or cloud mutation. Unknown or overlapping ownership
must fail closed.

## Options considered

### Option 1: Do nothing

Keep `ai-dev-foundation` as the effective direct source and continue denylist-based
sync. This has no immediate migration cost, but violates the intended topology and
permanently skips Terraform-family contracts. Rejected.

### Option 2: Declare both foundation and Terraform as direct parents

Merge two independent parent streams into this repository. This appears flexible, but
creates ambiguous ownership and ordering for shared files and governance policy, and
contradicts ADR-0004's direct-parent-only topology. Rejected.

### Option 3: Adopt `terraform-gcp-template` as the sole direct parent

Add a child-owned manifest and lock, protect local files, inherit reviewed shared files
only from Terraform, and advance one Terraform first-parent commit per PR. The
intermediate template remains responsible for its own foundation lock. This adds
sequential migration work but makes provenance and ownership deterministic.

## Decision

Adopt Option 3. `secure-ai-controls` MUST name
`Yukihide-Mitsuoka/terraform-gcp-template` on `main` as its only direct parent. The
initial lock MUST be `10e4a1a13f0cd7db973e986f7aafb8e5a5fb0174`, the latest Terraform
first-parent commit before this repository's initial commit, and every subsequent lock
advance MUST cover exactly one direct-parent first-parent commit in a green PR.

The child-owned manifest MUST define non-overlapping inherited and protected paths.
Repository-specific source, tests, configuration, security-control policy, project
documents, the manifest, lock, sync workflow, and ignore files remain protected unless
a later reviewed change says otherwise. Shared foundation and Terraform files are
inherited only when byte-identical materialization is safe; protected differences need
explicit child adaptations before the next checkpoint.

Inherited documentation MUST live under `docs/foundation/`. Repository-authored
documents MUST live directly under `docs/` or its project-owned subdirectories and be
written in Japanese. Existing foundation-authored root copies will be removed only in a
reviewed migration PR after their namespaced replacements are present. The
write-capable scheduled Template Sync path MUST remain disabled; local authenticated
branches and reviewed PRs are the propagation transport.

## Consequences

**Positive:** The repository receives the complete Terraform-family chain without
bypassing its parent, provenance becomes auditable, local ownership fails closed, and
foundation documentation no longer competes with project documentation.

**Negative:** Catch-up requires many small PRs and protected adaptations. The leaf can
lag while an intermediate checkpoint is under review, and maintainers must run the
local planner instead of relying on unattended writes.

Rollback is a reviewed PR reverting a materialization or accepted lock checkpoint. It
does not mutate live GitHub or cloud state.

**Follow-ups:**

1. Add and validate the direct-parent manifest and bootstrap lock without copying
   unreviewed paths.
2. Advance and adapt one Terraform first-parent commit at a time.
3. Materialize `docs/foundation/`, remove only confirmed legacy foundation copies, and
   add Japanese project-document placement guidance.
4. Replace or disable the legacy scheduled sync source and verify all required CI before
   each merge.
