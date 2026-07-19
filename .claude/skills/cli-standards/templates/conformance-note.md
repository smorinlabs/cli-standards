<!-- TEMPLATE — render with concrete values; shipped output must contain no {{VARS}}.
     Default destination: CONFORMANCE.md in the target CLI's repo root. -->
# CLI Standard Conformance — {{TOOL_NAME}}

| | |
|---|---|
| **Standard** | CLI Design Standard v{{STANDARD_VERSION}} |
| **Profile** | {{PROFILE}} <!-- Standard (noun-verb) | Small-CLI (Appendix A) --> |
| **Tier** | {{TIER}} <!-- minimal | standard | publishable --> |
| **Owner** | {{OWNER}} |

## Applicability

| Axis | Applies | Reason if N/A |
|---|---|---|
| Config (§5) | {{YES_NO}} | {{REASON_OR_DASH}} |
| Networked (§10) | {{YES_NO}} | {{REASON_OR_DASH}} |
| Destructive ops (§8) | {{YES_NO}} | {{REASON_OR_DASH}} |
| Scripted consumers (R7.2/R7.8) | {{YES_NO}} | {{REASON_OR_DASH}} |
| Async / long-running | {{YES_NO}} | {{REASON_OR_DASH}} |
| Streaming / watch | {{YES_NO}} | {{REASON_OR_DASH}} |
| Plugins (R9.11) | {{YES_NO}} | {{REASON_OR_DASH}} |
| Caching / offline (R5.9) | {{YES_NO}} | {{REASON_OR_DASH}} |
| Secrets handled (R5.5/R5.6) | {{YES_NO}} | {{REASON_OR_DASH}} |

## Waived SHOULDs

<!-- Required by the standard for conformance: rule ID, deviation, rationale,
     owner/date. An empty table is a healthy table. -->

| Rule | Deviation | Rationale | Owner / date |
|---|---|---|---|
| {{RULE_ID}} | {{WHAT_DIFFERS}} | {{WHY}} | {{OWNER_DATE}} |

## Audit history

| Date | Standard version | Mode | Result |
|---|---|---|---|
| {{DATE}} | {{STANDARD_VERSION}} | {{PLAN_REVIEW_OR_AUDIT}} | {{SUMMARY_LINE}} |
