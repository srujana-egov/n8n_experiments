# Task 2: Immunization Follow-up Workflow

Note: Import the [task 2 json file](https://github.com/srujana-egov/n8n_experiments/blob/main/Task2/Task2_%20Immunization%20%E2%80%93%20Cross-System%20Tracking%20(2).json) in n8n to run the workflow and to see the node logic. Postman collection is also given in [task 2 postman collection](https://github.com/srujana-egov/n8n_experiments/blob/main/Task2/n8n%20-%20task2.postman_collection.json).

<img width="1063" height="354" alt="Screenshot 2026-01-20 at 5 38 45 PM" src="https://github.com/user-attachments/assets/584add9a-76d6-445c-a104-9589f29ad4fb" />

The solution is split into **one main orchestration workflow** and **two event sub-workflows** to separate scheduling logic from execution handling.

---

## Main Workflow: Immunization Orchestrator

### Trigger: `immunization-recorded`

Triggered when a new vaccine dose is recorded in PHP.

---

### Processing Steps

#### 1. Sync Immunization to PHP

Persists the immunization update so EMR, dashboards, and downstream services remain consistent.

---

#### 2. Normalize Fields

Converts incoming payloads into a standard structure for workflow processing.

---

#### 3. Fetch Child Profile

Loads child master data (DOB, program, identifiers) required for scheduling.

---

#### 4. Build Vaccine History

Aggregates all completed doses for the child.
This produces a clean vaccine history list used to avoid duplicate scheduling.

---

#### 5. Fetch Vaccine Schedule (MDMS)

Loads vaccination policy from mock MDMS, including:

* Vaccine order
* Minimum age or gap rules
* Due windows
* Program mappings

This removes hardcoded scheduling logic from the workflow.

---

#### 6. Determine Next Vaccine (Core Logic)

This node computes the next due vaccine using configuration and history:

Logic applied:

* Compare **completed vaccines** with **MDMS schedule**
* Identify the **first pending vaccine** in the schedule sequence
* Calculate **due date** using:

  * Child DOB
  * Last vaccination date
  * Minimum interval rules from MDMS
* Validate eligibility (age window / interval)
* Output scheduling parameters:

  * Vaccine name
  * Due date
  * Priority
  * SLA window

Only valid, due vaccines are forwarded for task creation.

---

#### 7. Create CHW Outreach Task (UTM)

Creates a task for the Community Health Worker with:

* Vaccine name
* Due date
* Priority
* Program and tenant context

This task drives field execution.

---

## Sub Workflow 1: Visit Completion Handler

### Trigger: `immunization-visit-completed`

When CHW completes outreach:

* Updates immunization status in PHP
* Marks task as completed
* Keeps registry and reporting systems in sync

---

## Why MDMS Is Used

Vaccination rules are policy-driven and change frequently. Using MDMS:

* Avoids hardcoding logic
* Supports multi-tenant configuration
* Enables dynamic schedule updates

---

## Design Benefits

* Event-driven workflow orchestration
* Config-based vaccination logic
* Clear separation of planning vs execution
* DIGIT-aligned scalable architecture
