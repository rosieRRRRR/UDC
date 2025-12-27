# **User Controlled Intelligence (UCI)**

**An Open Standard for User-Controlled AI Behaviour Admission**

**Specification Version:** **v1.0.0**
**Status:** **Implementation-Ready, Fail-Closed Specification**
**Author:** rosiea
**Date:** December 2025
**Licence:** Apache License 2.0

---

## **Status of This Memo (Informative)**

This document specifies **User Controlled Intelligence (UCI)**, an admission-control architecture for AI behaviour.

Distribution of this memo is unlimited.

UCI is **not** a model, **not** an alignment framework, **not** an agent system, and **not** a provider moderation policy.
It defines a **user-controlled enforcement layer** that determines which AI behaviours are admissible before they reach users or downstream systems.

**Implementation Status (Informative).**
UCI defines the enforcement mechanism that enables user control; user-facing tooling and interfaces are intentionally out of scope for this specification.

---

## **1. Abstract (Normative)**

User Controlled Intelligence (UCI) defines a deterministic, user-owned behavioural admission layer for AI systems. All AI outputs and action proposals are treated as **untrusted proposals**. Behaviour is admitted **only if** it satisfies explicit, user-controlled **Behavioural Admissibility Rules (BAR)**.

UCI does not modify model weights, does not prevent model drift, and does not grant execution authority. Instead, it ensures that behavioural changes, contextual manipulation, and probabilistic variance do not reach the user **unless explicitly allowed**.

---

## **2. Purpose and Scope (Normative)**

### **2.1 Purpose**

UCI provides:

* separation of **model capability** from **behaviour admissibility**
* user-defined, user-controlled enforcement
* deterministic, auditable, fail-closed decisions
* explicit isolation between AI suggestion and authority

### **2.2 Scope**

This specification defines:

* UCI Profiles and Behavioural Admissibility Rules (BAR)
* deterministic admission evaluation semantics
* action-class classification and escalation
* predicate consumption contracts
* conformance requirements

This specification explicitly does **not** define:

* model training or fine-tuning
* provider moderation or alignment regimes
* autonomous agents or planners
* custody, governance, or execution authority
* end-user interfaces or GUI tooling

---

## **3. Terminology (Normative)**

* **UCI** — *User Controlled Intelligence*; the admission control system and enforcement layer that evaluates and admits AI behaviour.

* **BAR (Behavioural Admissibility Rules)** — User-defined **admission policy rules enforced by UCI**; analogous to access control rules or policy documents used in systems such as SELinux, AppArmor, or Kubernetes admission controllers.

* **UCI Profile** — A user-owned **security profile** containing one or more BARs; analogous to SELinux security profiles or Kubernetes admission policies.

* **Predicate Provider** — An external **attestation service** that provides verifiable boolean signals to UCI; analogous to TPM attestation, OAuth token validation, or policy decision points.

* **Action Class** — A **risk / privilege classification** for proposed AI behaviour; analogous to Android permission levels or capability-based security tiers.

* **AdmissionContext** — The canonical **admission / request context** used for deterministic rule evaluation.

* **Fail-Closed** — Deny on uncertainty (standard security principle).

RFC-2119 keywords apply.

---

## **4. Design Principles (Normative)**

1. User-defined admissibility
2. No model modification
3. Deterministic admission
4. Fail-closed enforcement
5. Authority isolation

---

## **5. Architectural Clarifications (Normative)**

### **5.1 Determinism vs Probabilistic Models**

Determinism applies to **admission decisions only**, not generation.

Given identical:

* UCI Profile
* action class
* AdmissionContext
* predicate results

→ the admission decision MUST be identical.

---

### **5.2 Context Without Prompt Inspection**

BAR rules MUST NOT inspect raw prompt text or full conversation history.

All rule matching operates on **AdmissionContext** (Annex F).
Raw prompts MAY be analysed by predicate providers but MUST NOT influence BAR logic.

---

### **5.3 Action Class Classification (Normative)**

Action class MUST NOT be self-asserted by the model.

Classification MUST follow this order:

1. application-declared class
2. deterministic rule-based classifier
3. conservative escalation

---

### **5.4 Principle of Minimum Agency (Normative)**

An output MUST be classified at the **lowest privilege level that cannot be disproven**.

If an output proposes or implies a real-world side effect **without** an explicit, host-level confirmation step, it MUST be classified as `execute`.

If an output asserts permission, approval, or authority, it MUST be classified as `authority`.

**Implicit Execution via Artifacts (Normative).**
If an output produces an artifact (for example, code, scripts, commands, messages, or documents) whose primary purpose is to cause a real-world side effect when executed or transmitted, the output MUST be classified as `execute` unless the host system enforces an explicit secondary commit step.

---

### **5.5 Classifier Soundness Rule (Normative)**

If the classifier cannot *prove* that an output belongs to a lower-risk action class, it MUST classify the output as the **next higher-risk class**, up to and including `execute` or `authority`.

Ambiguous classification is **not an error**; escalation is the correct and required outcome.

---

### **5.6 Intentional Risk Placement (Normative)**

UCI intentionally concentrates behavioural admission risk into a deterministic, testable classification and enforcement boundary, in order to avoid unbounded or emergent failure modes within the model or downstream systems.

---

## **6. Admission Pipeline (Normative)**

For each candidate output:

1. Determine action class
2. Construct `AdmissionContext`
3. Collect `PredicateResult` objects
4. Canonicalise inputs
5. Evaluate BAR rules
6. Enforce PASS or FAIL

---

## **7. Canonical Encoding (Normative)**

* **CBOR (deterministic)** — REQUIRED for hashing and signing
* **JCS JSON** — OPTIONAL for transport/UI

All cryptographic operations MUST use canonical CBOR.

---

## **8. Data Models (Normative)**

### **8.1 UCIProfile**

```
UCIProfile = {
  profile_id: tstr,
  name: tstr,
  version: tstr,
  strictness: "STRICT" / "TOLERANT" / "ADVISORY",
  scope: [* action_class],
  required_predicates: { * predicate_name => bool },
  bar_rules: [* BARRule],
  fallback_policy: "BLOCK" / "MASK" / "REDUCE",
  updated_at: uint,
  signature: bstr / null
}
```

Defaults:
`strictness = STRICT`
`fallback_policy = BLOCK`

If a signature is present, it MUST cover the canonical CBOR encoding of the profile with `signature` omitted.

---

### **8.2 BARRule**

```
BARRule = {
  rule_id: tstr,
  applies_to: [* action_class],
  when: ContextMatch / null,
  must: [* predicate_name],
  allow: bool,
  on_fail: "BLOCK" / "MASK" / "REDUCE"
}
```

Rules are evaluated in array order. First match wins.

---

## **9. ContextMatch (Normative)**

Deterministic boolean matcher over AdmissionContext:

```
ContextMatch = {
  all_of: [* Criterion] / null,
  any_of: [* Criterion] / null,
  none_of: [* Criterion] / null
}
```

```
Criterion = {
  field: "intent_label" / "action_class" / "phase" / "tool_intent",
  op: "eq" / "in" / "prefix",
  value: tstr / [* tstr]
}
```

---

## **10. Predicate Registry (Normative)**

Reserved predicate names:

* valid_model_identity
* valid_runtime
* valid_drift
* valid_tick
* valid_consent
* valid_policy
* valid_session

Missing or expired predicates MUST evaluate false.

---

## **11. Predicate Provider Interface (Normative)**

```
PredicateResult = {
  predicate: tstr,
  value: bool,
  issued_at: uint,
  expiry: uint / null,
  evidence: bstr / null,
  signature: bstr / null
}
```

Predicate providers are untrusted.
UCI MUST NOT infer truth.

---

## **12. Fully Specified Predicate Provider: `valid_consent` (Normative)**

**Inputs:** `session_id`, `request_id`, `action_class`

**Mechanism:**
Check persistent session consent OR transient consent token bound to request.

**Output:** PredicateResult (`value=true`) with expiry.
Absence or expiry → false.

---

## **13. Action Class Taxonomy (Normative)**

Ordered by risk:

1. style
2. explain
3. advise
4. decide
5. execute
6. authority

Escalation always moves upward.

---

## **14. Default Predicate Requirements (Recommended)**

| Class     | Required                                                                                                 |
| --------- | -------------------------------------------------------------------------------------------------------- |
| style     | valid_model_identity                                                                                     |
| explain   | valid_model_identity, valid_runtime                                                                      |
| advise    | valid_model_identity, valid_runtime, valid_drift                                                         |
| decide    | valid_model_identity, valid_runtime, valid_drift, valid_tick                                             |
| execute   | valid_model_identity, valid_runtime, valid_drift, valid_tick, valid_consent, valid_policy, valid_session |
| authority | ALL, STRICT                                                                                              |

---

## **15. Enforcement Outcomes (Normative)**

* **BLOCK** — deny output
* **MASK** — deterministic redaction (FORBIDDEN for execute/authority)
* **REDUCE** — downgrade class (FORBIDDEN for execute/authority)

---

## **16. Performance Considerations (Informative)**

Admission SHOULD occur at response or tool-call boundaries.
Predicate caching MAY be used with enforced expiry.

---

## **17. Energy and Resource Efficiency (Informative)**

UCI does not reduce per-token inference cost.
It may reduce **wasted computation** by preventing disallowed generation, retries, and unauthorised agent planning.

---

## **18. Security Properties (Normative)**

UCI guarantees:

* no silent escalation
* deterministic enforcement
* explicit responsibility
* authority non-delegation

---

## **19. Privacy Considerations (Informative)**

UCI evaluates behavioural admissibility using structured admission context and predicate signals. Raw prompt contents are explicitly excluded from rule evaluation, and this specification does not mandate telemetry, centralised logging, or data exfiltration. Privacy properties are therefore deployment-specific and controlled by the host environment.

---

## **20. Threat Model (Normative)**

**In-scope:** prompt injection; drift; misclassification; unauthorised tool use.
**Out-of-scope:** malicious profiles; compromised predicates; external social engineering.

---

## **21. Versioning and Profile Evolution (Normative)**

Profile composition, inheritance, and temporal policy constructs are intentionally out of scope for this version.

Profiles MUST be versioned; updates explicit; mid-session changes re-evaluate; conflicts default BLOCK.

---

## **22. Conformance (Normative)**

An implementation is conformant iff it:

1. Implements schemas and evaluator
2. Uses CBOR for signing
3. Enforces fail-closed semantics
4. Enforces Classifier Soundness Rule
5. Escalates ambiguity
6. Does not modify models
7. Does not claim authority

---

# **Annex F — AdmissionContext Schema (Normative)**

```
AdmissionContext = {
  intent_label: tstr,
  action_class: action_class,
  session_id: tstr,
  phase: "initial" / "followup" / "final",
  tool_intent: tstr / null
}
```

---

# **Annex G — Action Class Calibration Corpus (Normative)**

| Prompt                              | Classified As | Rationale            |
| ----------------------------------- | ------------- | -------------------- |
| “What is insulin?”                  | explain       | factual              |
| “Should I take insulin?”            | advise        | recommendation       |
| “Order insulin”                     | execute       | real-world action    |
| “Order 100mg of morphine”           | execute       | controlled substance |
| “Can you help me order morphine?”   | execute       | implicit action      |
| “Draft an email ordering morphine”  | decide        | selection w/ commit  |
| “Sign prescription”                 | authority     | asserts authority    |
| “Generate code to delete all files” | execute       | destructive artifact |

Ambiguous phrasing MUST escalate.

---

# **Annex H — Deterministic Evaluator Pseudocode (Normative)**

```
function evaluate(profile, context, predicates):
  verify(profile)
  class = classify(context)
  for rule in profile.bar_rules:
    if class in rule.applies_to and match(rule.when, context):
      if rule.allow and all(predicates[p] == true for p in rule.must):
        return PASS
      else:
        return FAIL(rule.on_fail)
  return FAIL(profile.fallback_policy)
```

---

## **Acknowledgements (Informative)**

Thanks to: Saltzer, Schroeder, Lampson; IETF / IRTF; Zero Trust community; AI safety researchers; independent reviewers.

---

## **Conclusion (Informative)**

UCI does not rely on models behaving well.
It ensures systems behave safely **even when models do not**.
