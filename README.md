# PPL-Library

The **Process Protocol Layer (PPL)** is a universal schema for describing any process — from brewing coffee to validating a vaccine.  
This repository serves as the **core library**, documenting the schema, example processes, and applications across industries.

---

## What is PPL?

Like DNA’s four bases, PPL reduces any process into a simple but universal structure.  
Every process, no matter how complex, can be expressed with five core elements:

1. **Inputs** – The data, documents, or materials required to begin.  
   *Examples: lab test results, a customer application form, a meeting agenda.*

2. **Preconditions** – The conditions that must be true before starting.  
   *Examples: "approval granted", "data verified", "quorum present".*  
   - In v2.3, a precondition can also **depend on another process**. The execution engine can auto-check that dependency.

3. **Activity** – The description of the action or steps to be performed.  
   *Examples: "mix ingredients", "conduct meeting", "validate data".*  
   - Activities can now be broken into **steps** with explicit `sequence` or `depends_on` relationships.  
   - This allows ordered flows, parallelism, or full DAGs (directed acyclic graphs).

4. **Outputs** – The expected result(s) of the activity.  
   *Examples: "final report generated", "batch compounded", "decision recorded".*  
   - Outputs may include **verification rules** to check correctness (e.g., hash match, schema validation).

5. **Approvals** – Verification or sign-off steps, human or automated.  
   *Examples: supervisor approval, e-signature, regulatory sign-off.*  
   - Approvals now support `method` (e.g., e-signature, click-to-approve) and optional `recordkeeping` for signed evidence.  
   - At runtime, an `evidence` block is populated (signer, timestamp, certificate chain, etc.).

---

## Attachments and Extensions

Each element (and the process itself) can be extended with optional attachments:

- **Overlays** – Values, Metrics, Bias Checks, Controls  
- **Layers** – Decision Artifacts, Verification, Deployment Mapping  
- **Recordkeeping** – Optional overrides for routing/retention/policy (engine always logs by default)  
- **Dispatch** – Send records or evidence packs (PDF, JSON, etc.) to external systems  
- **Tools/Integrations** – Attach calls to automation frameworks (e.g., n8n workflows)  

---

## Dependencies vs Transitions

PPL v2.3 distinguishes **dependencies** and **transitions**:

- **Dependencies** – Define other processes that must reach a state before this process can start or proceed.  
  *Example: "Batch Record Review must succeed before Release to Warehouse can begin."*

- **Transitions** – Define routing after a process completes.  
  *Example: "If QA Approved → go to Release to Warehouse; if QA Rejected → go to Rework Batch."*

This allows the execution engine to act like both a **project manager** (tracking prerequisites and critical paths) and a **workflow engine** (routing after completion).

---

## Audit Logging

The Execution Engine always maintains an immutable log.

- Inputs are logged with hash, size, mime type, and pointer to storage  
- Configurable to embed small, safe inputs inline  
- Evidence packs can bundle inputs, outputs, and approvals into a signed archive  

---

## Example: Coffee with Dependency

```yaml
process: Make Coffee
process_id: urn:ppl:process:make_coffee
version: 2.3

dependencies:
  - process_id: urn:ppl:process:fill_water_tank
    required_state: succeeded

inputs:
  - Coffee grounds
  - Water
preconditions:
  - Coffee machine is plugged in
  - depends_on_process:
      process_id: urn:ppl:process:fill_water_tank
      required_state: succeeded

activity:
  sequence_model: sequential
  steps:
    - id: grind
      name: Grind beans
      instruction: Use grinder until coarse texture
    - id: brew
      name: Brew coffee
      instruction: Use machine to brew
      depends_on: [grind]

outputs:
  - name: Cup of brewed coffee
    verification:
      method: taste_test
      required: true

approvals:
  - type: Human taste approval
    actor: Coffee drinker
    method: click_to_approve

transitions:
  - to_process: Clean Machine
    condition: Always
```

---

## Repository Contents

- **schemas/ppl.schema.json** – the current schema (v2.3)  
- **examples/** – simple and complex process examples  
- **docs/** – design notes, white papers, and application guides  
or data sets used to guide decisions.  
- **Transitions** – State changes that connect one PPL to another.  
- **Verification** – Defines if a human check or automated validation is required.  
- **Deployment Profile** – Links the PPL definition to actual tools, systems, or environments.  

---

## ☕ Example: Making Coffee (Simple PPL)

```yaml
Process: Make Coffee
Inputs:
  - Coffee grounds
  - Water
Preconditions:
  - Coffee machine is plugged in
  - Water reservoir filled
Activity: Brew coffee using the machine
Outputs:
  - Cup of brewed coffee
Approvals:
  - Taste test by human drinker
