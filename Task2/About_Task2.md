# Task 2: Immunization Follow-up Workflow

## Goal

Automatically identify the **next due vaccination** after an immunization is recorded and create a **CHW outreach task** using configuration from MDMS.

---

## Trigger

**Event:** `immunization-recorded`  
Fired when a vaccine dose is saved in PHP.

Contains:
- Child ID  
- Vaccine code  
- Dose number  
- Date administered  
- Tenant / program info  

---

## Workflow Flow

1. Receive immunization event  
2. Normalize incoming data  
3. Fetch child profile  
4. Build vaccination history  
5. Load vaccine schedule from MDMS  
6. Determine next due vaccine  
7. Create CHW outreach task  

Each step is isolated to keep the workflow readable and easy to debug.

---

## MDMS Usage

MDMS provides the **official vaccine schedule**, including:

- Vaccine sequence  
- Dose order  
- Due offsets (days after birth)  

Example (conceptual):

```json
BCG → Due at birth  
OPV Dose 1 → Due after 42 days  
OPV Dose 2 → Due after 70 days
```
This avoids hardcoding medical rules inside the workflow.

## How Next Vaccine Is Determined
The workflow:

Reads child DOB -> Reads completed vaccines -> Compares against MDMS schedule -> Finds the first missing dose -> Calculates its due date

If the vaccine is due → outreach task is created.
If not due → workflow exits.


## Why This Design
1. Stateless: No workflow state stored in n8n
2. Config-driven: Schedule changes only require MDMS updates
3. Scalable: Same logic works across programs and regions

## Result
This workflow ensures children automatically receive follow-up vaccination outreach based on real-time data and centralized configuration.

