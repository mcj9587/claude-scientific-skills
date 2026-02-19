# Aegis MVP Blueprint (14-Day Launch)

## 1) Initial Customer Choice

**Pick first:** security-conscious teams already piloting agents.

Why this segment first:
- Strong, immediate pain: they need approvals, auditability, and policy controls before broader rollout.
- Faster willingness-to-pay than individual power users.
- Better references for enterprise expansion once the trust layer proves value.

Wedge distribution still comes from OpenClaw compatibility, but pricing and roadmap are shaped for team deployment from day one.

## 2) Ruthless Promise

> Install Aegis and your agent cannot silently do catastrophic things.

This is enforced through three controls:
1. Pre-install skill trust checks (scan + verify + signatures).
2. Runtime decision point for every high-risk action (allow/deny/require approval).
3. Tamper-evident audit records for every tool invocation.

## 3) Proposed Repository Structure

```text
aegis/
  cmd/
    aegis/                      # CLI entrypoint (scan/verify/install-verified/policy test)
    aegisd/                     # Local firewall daemon entrypoint
  internal/
    policy/
      engine/                   # Decision engine: allow/deny/approve
      parser/                   # Policy schema + validation
      simulator/                # "what would happen" mode
    skills/
      scanner/                  # SKILL.md + referenced file scanner
      manifest/                 # Deterministic manifest builder
      signer/                   # Signature + verification logic
      registry/                 # Verified registry client
    runtime/
      interceptors/             # Tool call hooks (exec/fs/network/connectors)
      approvals/                # Approval queue + UI adapters
      sandbox/                  # Docker/container profile orchestration
      secrets/                  # Scoped, short-lived broker tokens
    audit/
      logstore/                 # SQLite append-only event log
      hashchain/                # Optional cryptographic linkage
      exporters/                # JSONL/SIEM exports
    integrations/
      openclaw/                 # OpenClaw install + runtime adapters
      nanoclaw/                 # NanoClaw runtime adapter
  policies/
    baseline/                   # Secure-by-default policy packs
    examples/                   # Team/role-specific examples
  schemas/
    policy.schema.json
    manifest.schema.json
    risk-report.schema.json
  scripts/
    ci/
      scan_and_sign.sh
      publish_registry.sh
  web/
    registry-site/              # Static verified-skill index
  docs/
    architecture.md
    policy-language.md
    threat-model.md
    onboarding.md
```

## 4) First Policies to Ship (Secure-by-Default, Not Useless)

### 4.1 Baseline policy profile

- Default decision: `deny`.
- Read access: allow within explicit workspace roots only.
- Write access: allow in workspace; deny dotfiles and system paths unless approved.
- Shell execution: allow low-risk commands; require approval for mutating commands.
- Network egress: deny by default; allowlist approved domains per workflow.
- Secret access: brokered only; never return long-lived plaintext keys to model context.

### 4.2 High-risk action gates (require human approval)

- Sending email or external messages.
- Any command containing package install + execution in one step.
- Git operations that rewrite history (`push --force`, rebase on protected branches).
- File writes outside declared workspace.
- Access to production credentials, cloud consoles, or deployment tools.
- Browser automation on non-allowlisted domains.

### 4.3 Hard denies (never allow in MVP)

- Direct writes to SSH keys and shell startup files.
- Unscoped recursive exfil patterns (bulk read + network post in same flow).
- Disabling firewall hooks or tampering with audit logs.
- Persisting secrets to plaintext files in project tree.

## 5) Skill Risk Spec v0 (Scanner Output)

Each scanned skill returns:
- `risk_score` (0-100)
- `risk_level` (`low|medium|high|critical`)
- `requested_permissions`:
  - filesystem (`read`, `write`, paths)
  - execution (`shell`, `python`, external binaries)
  - network (`domains`, `wildcards`)
  - secrets (`which providers/scopes`)
- `findings[]` with machine-readable rule IDs + plain-language explanations
- `recommended_policy_overrides` (minimal permissions to run safely)
- `integrity` block (file hashes + manifest digest)

## 6) First 10 Verified Skills to Curate

Prioritize high-demand, bounded-risk workflows:
1. Gmail triage + draft responses (send action approval required)
2. Calendar scheduling assistant (external invite approval required)
3. GitHub issues triage + labeling (repo-scoped token)
4. Pull request summarizer + changelog generator (read-only default)
5. Local repo maintenance (format/lint/test only, no network)
6. Documentation QA + link checker (allowlisted docs domains)
7. Research web summarizer (strict domain allowlist + citation output)
8. Competitive intel monitor (RSS/news APIs only)
9. Incident note compiler (reads logs, no outbound post)
10. Meeting notes to action items (writes local markdown, optional Jira approval)

## 7) Success Criteria for the First 14 Days

- `aegis scan` produces deterministic risk reports and manifests for OpenClaw-style skills.
- `aegis verify` validates signatures and provenance.
- `aegisd` returns policy decisions under 100ms for local requests.
- Approval flow works for at least three high-risk actions.
- Audit logs capture actor, action, decision, and redaction metadata.
- At least 25 verified skills published in registry alpha.

## 8) Day-0 GTM Messaging

- Primary message: **"Untrusted skills are software supply chain risk. Aegis verifies before install and governs at runtime."**
- Buyer message: **"Ship agents without violating your security team’s non-negotiables."**
- User message: **"You can still automate real work—just with guardrails that prevent irreversible mistakes."**
