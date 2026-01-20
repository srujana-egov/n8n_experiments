# Task 3 — Chronic Disease: Hypertension Follow-Up Workflow

This implementation uses **two workflows**:

### 1) Diagnosis → Recurring Task Creation (Primary Workflow)

Triggered when hypertension is recorded in the Facility EMR.

**Flow:**

1. **hypertension-diagnosed (Webhook)**
   Receives diagnosis event from EMR.

2. **Sync Diagnosis to PHP**
   Updates the unified Personal Health Profile (PHP).

3. **Normalize Fields**
   Converts EMR payload into a standard internal format.

4. **Fetch Recurrence Policy (Mock MDMS)**
   Loads follow-up rules (frequency, SLA, duration) from config instead of hardcoding.

5. **Merge**
   Combines patient data with recurrence policy.

6. **Determine Recurrence (Logic Node)**
   Applies rules:

   * Uses program type + risk level
   * Calculates frequency, interval, SLA
   * Computes max occurrences from duration
     Output = clean recurring task definition.

7. **Create Recurring CHW Task (UTM)**
   Publishes monthly/weekly follow-up tasks to the Unified Task Manager.

---

### 2) BP Reading Capture (Update Workflow)

Handles field data updates.

**Flow:**

1. **bp-reading-recorded (Webhook)**
   Triggered by CHW app after home visit.

2. **Update PHP with BP Reading**
   Stores measurement centrally so doctors can view latest values.

---

## Determine Recurrence Logic (Key Node)

The recurrence engine follows this logic:

* Read program type (`MATERNAL_FOLLOWUP` / `NCD_FOLLOWUP`)
* Load matching policy from MDMS
* If patient is **HIGH risk** → override SLA and priority
* Use:

  * `frequency` (eg: WEEKLY)
  * `interval` (eg: every 1 week)
  * `durationWeeks` → converted to `maxOccurrences`
* Output is a normalized recurring schedule for UTM

This keeps scheduling **config-driven**, not hardcoded.

---

## Why Mock MDMS Is Used

MDMS simulates a central policy service that controls:

* Follow-up frequency
* Allowed schedules
* SLA rules
* Risk-based overrides

This makes the workflow reusable across programs and geographies.

---

## Alignment With Experiment Goals

✔ Event-driven orchestration
✔ External app participation via webhooks
✔ Config-based recurrence handling
✔ Portable, machine-readable workflow design
✔ Clear separation of logic and execution
