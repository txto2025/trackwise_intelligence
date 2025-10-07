# 🧠 Fryt.ai Core Components Reference

**Document Version:** v1.0  
**Author:** Architecture Team  
**Last Updated:** October 2025  
**Status:** Core Reference Document  

---

## 🏗️ 1. Overview

The Fryt.ai Reasoning Engine consists of **three architectural layers**, each composed of modular, versioned components. Together, they transform **raw carrier data** into **explainable, auditable claim decisions**.

| Layer | Purpose | Components |
|-------|----------|-------------|
| **Contract & Context Layer** | Converts raw tracking data and contracts into structured, contextual facts. | `SLA_RULES`, `CANONICAL_EVENTS`, `CONTRACTUAL_WINDOWS`, `EXCEPTIONS` |
| **Reasoning Engine** | Applies declarative rules to facts to determine claim eligibility. | `RULE_REGISTRY`, `EVALUATOR`, `CONTEXT_FACTS` |
| **Intelligence Layer (Outcome)** | Generates explainable, confidence-scored, auditable outcomes. | `LLM_SUMMARIZER`, `CONFIDENCE_SCORER`, `HITL_MODULE` |

---

## 🧩 2. Contract & Context Layer

This layer defines the **factual baseline** of every shipment — what was promised, what happened, and what exceptions apply.

### **2.1 SLA_RULES**
Encodes carrier contractual terms into structured, machine-readable form.

```json
{
  "carrier": "UPS",
  "service_level": "2DAY_AIR",
  "sla": {
    "promise_window": {
      "basis": "pickup",
      "business_days": 2,
      "end_of_day": true
    },
    "exemptions": ["WEATHER", "CUSTOMS"]
  },
  "lost": {"inactivity_threshold_days": 7},
  "damage": {"evidence_window_hours": 72},
  "filing": {"deadline_days": 15}
}
```

---

### **2.2 CANONICAL_EVENTS**
Normalizes raw carrier events into a consistent schema.

```json
{
  "tracking": "1Z12345",
  "canonical_events": [
    {"event_code": "PICKUP", "ts_local": "2025-10-02T09:15-04:00"},
    {"event_code": "IN_TRANSIT", "ts_local": "2025-10-03T14:45-04:00"},
    {"event_code": "DELIVERED", "ts_local": "2025-10-04T15:02-04:00"}
  ]
}
```

---

### **2.3 CONTRACTUAL_WINDOWS**
Defines time and condition windows derived from contracts.

```json
{
  "sla_window": {"start": "2025-10-02T09:15-04:00", "end": "2025-10-04T23:59-04:00"},
  "lost_window": {"threshold_days": 7},
  "damage_window": {"evidence_hours": 72},
  "filing_window": {"deadline_days": 15}
}
```

---

### **2.4 EXCEPTIONS**
Identifies and classifies exceptions affecting claim eligibility.

```json
{
  "tracking": "1Z12345",
  "exceptions_applied": ["WEATHER"],
  "exempted": true
}
```

---

## ⚙️ 3. Reasoning Engine

This layer transforms structured facts into **deterministic eligibility decisions**.

### **3.1 RULE_REGISTRY**
Central catalog of declarative rules defining how to evaluate claims.

```json
{
  "rule_id": "SLA_LATE_DELIVERY",
  "category": "SLA",
  "description": "Mark shipment eligible if delivered late and not exempted.",
  "when": [
    {"fact": "sla_promise.late_business_days", "op": "gt", "value": 0},
    {"fact": "sla_promise.exempted", "op": "eq", "value": false}
  ],
  "then": [
    {"set": "eligibility.sla_refund", "value": true},
    {"append": "reasons", "value": "Delivered after promised window"}
  ],
  "priority": 10
}
```

---

### **3.2 EVALUATOR**
Executes rules from the registry against contextual data.

```json
{
  "tracking": "1Z12345",
  "eligibility": {"sla_refund": true, "lost": false, "damage": false},
  "rules_fired": ["SLA_LATE_DELIVERY"],
  "trace": {
    "SLA_LATE_DELIVERY": {
      "sla_promise.late_business_days": 1,
      "sla_promise.exempted": false
    }
  }
}
```

---

### **3.3 CONTEXT_FACTS**
Shipment-level facts produced by combining contracts and events.

```json
{
  "tracking": "1Z12345",
  "carrier": "UPS",
  "sla_promise": {
    "late_business_days": 1,
    "exempted": false
  },
  "lost_facts": {
    "days_since_last_scan": 8
  },
  "damage_facts": {
    "has_damage_event": true,
    "reported_within_hours": 48
  },
  "exemptions_applied": []
}
```

---

## 🧠 4. Intelligence Layer (Outcome)

Synthesizes reasoning output into human-readable, explainable outcomes.

### **4.1 LLM_SUMMARIZER**
Converts traces into natural language.

> “Shipment was delivered one business day late with no valid exemptions. Eligible for SLA refund under UPS 2-Day Air terms.”

---

### **4.2 CONFIDENCE_SCORER**
Quantifies reliability of the decision.

```json
{"confidence": 0.88}
```

---

### **4.3 HITL_MODULE**
Manages human overrides and audit trails.

```json
{
  "tracking": "1Z12345",
  "override": {
    "field": "sla_promise.exempted",
    "old_value": false,
    "new_value": true,
    "user_id": "ops_user_17",
    "rationale": "Weather delay confirmed by customer",
    "timestamp": "2025-10-06T21:00:00Z"
  }
}
```

---

## ✅ 5. Summary Matrix

| Layer | Component | Function | Output Artifact |
|--------|------------|-----------|-----------------|
| **Contract & Context** | SLA_RULES | Contract → Structured JSON | Structured rule JSON |
|  | CANONICAL_EVENTS | Raw → Canonical timeline | Unified event log |
|  | CONTRACTUAL_WINDOWS | Time constraints | Promise & filing windows |
|  | EXCEPTIONS | Identify disruptions | Exception flags |
| **Reasoning Engine** | RULE_REGISTRY | Logical rule definitions | Declarative logic set |
|  | EVALUATOR | Executes rules | Reasoning Results |
|  | CONTEXT_FACTS | Shipment facts | Facts JSON/table |
| **Intelligence Layer** | LLM_SUMMARIZER | Generates explanations | Narrative text |
|  | CONFIDENCE_SCORER | Scores reliability | Confidence score |
|  | HITL_MODULE | Manages overrides | Audit trail |
