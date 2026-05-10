# Appendix C — Decision Templates

> **Implementation Status (2026-03-27):** Decision templates are used by agents when calling the `create_decision` tool. The `options` field accepts a JSON array matching the structure below. The Decisions UI displays "Your AI Team Recommends" prominently based on the `recommendation` field.

## C.1 Architecture Decision

```json
{
  "decision_id": "DEC-XXX",
  "template": "architecture",
  "trigger": "[What architectural question needs answering]",
  "context": "[Background: constraints, requirements, existing patterns]",
  "options": [
    {
      "name": "Option A",
      "description": "[Description]",
      "pros": ["[Pro 1]", "[Pro 2]"],
      "cons": ["[Con 1]", "[Con 2]"],
      "risk": "Low|Medium|High|Critical",
      "effort": "Low|Medium|High"
    },
    {
      "name": "Option B",
      "description": "[Description]",
      "pros": ["[Pro 1]", "[Pro 2]"],
      "cons": ["[Con 1]", "[Con 2]"],
      "risk": "Low|Medium|High|Critical",
      "effort": "Low|Medium|High"
    }
  ],
  "risk_rating": "Medium",
  "recommendation": "[Recommended option with rationale]",
  "owner": "solution-architect",
  "impacted_cards": ["EPIC-XXX", "FEAT-XXX"],
  "impacted_artifacts": ["sdd.md"]
}
```

---

## C.2 Dependency Addition Decision

```json
{
  "decision_id": "DEC-XXX",
  "template": "dependency",
  "trigger": "Need to add [package-name] for [purpose]",
  "context": "Current alternatives: [existing options]. Reason current options insufficient: [reason]",
  "options": [
    {
      "name": "[Package A]",
      "version": "[version]",
      "license": "[license]",
      "size": "[bundle size]",
      "maintenance": "Active|Low|Abandoned",
      "security": "No known CVEs|Has CVEs"
    },
    {
      "name": "[Package B]",
      "version": "[version]",
      "license": "[license]",
      "size": "[bundle size]",
      "maintenance": "Active|Low|Abandoned",
      "security": "No known CVEs|Has CVEs"
    }
  ],
  "risk_rating": "Low",
  "recommendation": "[Recommended package with rationale]",
  "owner": "junior-developer",
  "impacted_cards": ["TASK-XXX"],
  "impacted_artifacts": ["package.json"]
}
```

---

## C.3 Scope Change Decision

```json
{
  "decision_id": "DEC-XXX",
  "template": "scope-change",
  "trigger": "[What scope change is requested]",
  "context": "[Why the change is requested, impact assessment]",
  "options": [
    {
      "name": "Accept scope change",
      "impact_on_timeline": "[estimate]",
      "impact_on_budget": "[estimate]",
      "impacted_features": ["FEAT-XXX"]
    },
    {
      "name": "Defer to next release",
      "impact_on_timeline": "None",
      "impact_on_budget": "None",
      "impacted_features": []
    },
    {
      "name": "Reject scope change",
      "rationale": "[Why]"
    }
  ],
  "risk_rating": "Medium",
  "recommendation": "[Recommended option]",
  "owner": "product-manager",
  "impacted_cards": ["EPIC-XXX"],
  "impacted_artifacts": ["brd.md", "sdd.md"]
}
```

---

## C.4 Security Decision

```json
{
  "decision_id": "DEC-XXX",
  "template": "security",
  "trigger": "[Security concern identified]",
  "context": "[Threat description, affected components, severity]",
  "options": [
    {
      "name": "Remediate immediately",
      "effort": "High",
      "risk_if_deferred": "Critical"
    },
    {
      "name": "Mitigate with workaround",
      "effort": "Medium",
      "risk_if_deferred": "Medium"
    },
    {
      "name": "Accept risk (document rationale)",
      "effort": "Low",
      "risk_if_deferred": "Unchanged"
    }
  ],
  "risk_rating": "High",
  "recommendation": "[Recommended option]",
  "owner": "security-compliance",
  "impacted_cards": ["FEAT-XXX"],
  "impacted_artifacts": ["sdd.md", "security-audit.log"]
}
```
