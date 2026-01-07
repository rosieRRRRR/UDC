# User-Defined Control (UDC)

**Status:** DEPRECATED / RETIRED
**Effective Date:** January 2026
**Author:** rosiea
**Contact:** [PQRosie@proton.me](mailto:PQRosie@proton.me)
**Licence:** Apache License 2.0 — Copyright 2025 rosiea

---

## 1. Status

This document is a **tombstone specification**.

User-Defined Control (UDC) is **deprecated** and **MUST NOT** be used as an active or authoritative specification.

All normative functionality previously defined by UDC has been **fully absorbed into PQAI** and is now enforced exclusively via **PQSEC**.

This document is retained **for historical reference only**.

---

## 2. Reason for Deprecation

UDC originally defined a deterministic behavioural admission layer for AI systems.

That functionality has been:

* structurally integrated into **PQAI**
* formally enforced by **PQSEC**
* aligned with canonical encoding, replay safety, and fail-closed semantics across the PQ ecosystem

Maintaining UDC as a standalone specification would duplicate logic, fragment authority boundaries, and violate single-source-of-truth principles.

---

## 3. Supersession

UDC is superseded by:

* **PQAI — Post-Quantum Artificial Intelligence**
  Defines artefact schemas for:

  * action class classification
  * behavioural fingerprinting
  * drift classification
  * SafePrompt and consent artefacts

* **PQSEC — Post-Quantum Security Ecosystem**
  Defines:

  * enforcement semantics
  * refusal and escalation
  * lockout and replay protection
  * authoritative decision handling

No normative behaviour defined in UDC remains unique or authoritative.

---

## 4. Prohibited Usage

Implementations MUST NOT:

* claim conformance to UDC
* implement UDC as an enforcement layer
* reference UDC for behavioural authority decisions
* derive security, safety, or admission guarantees from UDC alone

Any such implementation is non conformant with the current PQ specification set.

---

## 5. Historical Notes

UDC introduced several concepts that directly informed later specifications, including:

* explicit separation of behaviour admissibility from model capability
* deterministic action class escalation
* fail-closed behavioural gating
* isolation of authority from AI output

These concepts are now normatively specified and enforced within PQAI and PQSEC.

---

## 6. Conformance

There is **no conformance target** for this document.

This specification defines no requirements, no schemas, and no enforcement semantics.

---

## 7. Explicit Dependencies

This document has **no dependencies**.

Active systems MUST depend on PQAI and PQSEC instead.

---

## Annexes

None.

---

**End of Document**
