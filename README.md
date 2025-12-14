# VeriRuntime

**Verify what is actually running.**

VeriRuntime is a Runtime Binary Verification (RBV) engine that continuously verifies whether *running processes* match their **expected build artifacts, signatures, and attestations**.

Unlike EDRs or behavior-based monitoring tools, VeriRuntime focuses on detecting **runtime drift and binary tampering after initial compromise**. It is designed to be minimal, read-only, and composable with existing security stacks.

---

## What VeriRuntime Is

* A **runtime verifier** for executables and processes
* Focused on **post-compromise detection**
* Read-only, low-noise, least-privilege by design
* Built for Linux-first environments

## What VeriRuntime Is Not

* ❌ An EDR or behavior analytics engine
* ❌ A prevention or auto-remediation tool
* ❌ A network or syscall monitoring system

VeriRuntime complements EDR, Falco, and CNAPP tools by validating a missing invariant:

> *"Is the binary that is actually running the one we intended to run?"*

## Verification Coverage and Guarantees

VeriRuntime explicitly reports **what was verified and what was not**.

A process is **never reported as fully verified** unless all intended
verification scopes succeed.

If verification is partially degraded due to permission, platform,
or environmental limitations, VeriRuntime emits an explicit
`VERIFICATION_DEGRADED` event instead of silently reporting success.

---

## Core Concepts

* **Runtime Binary Verification (RBV)**
  Continuous verification of running binaries against trusted references

* **Runtime Drift**
  Any deviation between the expected binary state and the observed runtime state

* **Read-only Verifier**
  VeriRuntime detects and reports — it does not mutate system state

---

## High-Level Architecture

* **Core Engine (Rust)**
  Performs runtime verification with minimal attack surface

* **Control Plane (Go)**
  Handles policy, notifications, and external integrations

* **Hybrid IPC Model**

  * Internal: Unix Domain Socket (UDS)
  * External: gRPC (optional)

---

## Initial Scope

* Verify running processes against:

  * Binary hashes
  * ELF headers and sections
  * Memory mapping consistency
* Emit high-signal security events on mismatch
* Integrate with existing security tooling (optional)

---

## Project Status

🚧 **Early development / PoC stage**

---

# VeriRuntime – Technical Specification (v1 Draft)

## 1. Threat Model

VeriRuntime assumes:

* The attacker may already have initial access
* Traditional EDR or monitoring may be partially bypassed
* The filesystem, runtime memory, or execution chain may be tampered with

The goal is **detection**, not prevention.

---

## 2. Responsibilities

### Core Engine (Rust)

**Responsibilities**

* Verify runtime state
* Compare observed state with expected references
* Emit verification results and events

**Non-Responsibilities**

* No remediation
* No policy decisions
* No external network communication

---

### Control Plane (Go)

**Responsibilities**

* Receive verification results
* Apply policy
* Forward events to external systems (EDR, SIEM, webhook)

---

## 3. Core API (v1 Minimal)

```text
VerifyProcess(pid)
VerifyBinary(path | inode)
GetEvidence(ref)
ListRuntimeState()
Health()
```

All APIs are read-only and side-effect free.

---

## 4. Verification Techniques (v1)

* SHA-256 hash of on-disk executable
* ELF header validation
* ELF section table integrity
* Runtime memory map comparison (basic)

---

## 5. Event Model

VeriRuntime emits events only on anomalies.

### Event Types

* VERIFY_FAILED
* RUNTIME_DRIFT
* UNKNOWN_EXECUTION
* SELF_INTEGRITY_FAILURE

Events are rate-limited and designed to be low-volume, high-signal.

---

## 6. Security Principles

* Least privilege
* Explicit trust boundaries
* Minimal dependencies
* Crash-safe and fail-closed reporting

---

# Roadmap

## Phase 0 – Foundation (Week 0–1)

* Project structure
* API definition freeze (v1)
* Threat model documentation

---

## Phase 1 – Proof of Concept (Not implemented)

**Core Engine**

* Implement `VerifyProcess(pid)`
* ELF header parsing
* Binary hashing
* UDS server

**Control Plane**

* UDS client
* Console-based event output

**Goal:** Demonstrate runtime drift detection

---

## Phase 2 – MVP (Not implemented)

* ELF section validation
* Memory map comparison
* Self-integrity checks
* Basic policy handling in Control Plane
* Webhook / Slack notification support

**Goal:** Usable in real environments

---

## Phase 3 – Integration  (Not implemented)

* gRPC interface for external tools
* Falco / EDR event consumption
* Improved performance and rate limiting

**Goal:** Enterprise PoC readiness

---

## Phase 4 – OSS Hardening (Not implemented)

* Documentation
* Security guidelines
* systemd / container deployment examples
* Governance and contribution model

**Goal:** Public OSS adoption

---

## Long-Term Ideas

* Attestation-based reference sources
* Trust-chain visualization
* Cross-platform research (macOS / Windows)

---

## License

TBD (Apache-2.0 or MIT recommended)
