# Task 3 — Chronic Disease: Hypertension Follow-Up Workflow

Note: Import the [task 3 json file](https://github.com/srujana-egov/n8n_experiments/blob/main/Task3/Task3_%20Chronic%20Disease%20%E2%80%93%20Hypertension%20Follow-Up%20(2).json) in n8n to run the workflow and to see the node logic. Postman collection is also given in [task 3 postman collection](https://github.com/srujana-egov/n8n_experiments/blob/main/Task3/n8n%20-%20task3.postman_collection.json).

<img width="785" height="352" alt="Screenshot 2026-01-20 at 5 51 35 PM" src="https://github.com/user-attachments/assets/29ea8879-cfeb-4568-acbf-be60e6fd3dff" />

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

1. Event-driven orchestration
2. External app participation via webhooks
3. Config-based recurrence handling
4. Portable, machine-readable workflow design
5. Clear separation of logic and execution
