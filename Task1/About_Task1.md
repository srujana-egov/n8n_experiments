### Task 1 — High-Risk Pregnancy + Diabetes Screening
Note: Import https://github.com/srujana-egov/n8n_experiments/blob/main/Task1/Task1_%20High-Risk%20Pregnancy%20%2B%20Diabetes%20Screening%20(3).json to see the node logic
<img width="914" height="488" alt="Screenshot 2026-01-20 at 5 14 56 PM" src="https://github.com/user-attachments/assets/3dfacc5f-85ca-4c64-ac0e-fe490519ad3b" />

This task was implemented as three event-driven workflows to reflect real healthcare process boundaries and human-in-the-loop interactions.

1. **Pregnancy Registration Workflow**
   Triggered by a pregnancy registration event from the Maternal App. The workflow creates a diabetes screening task for CHWs in the task system (UTM). This isolates intake logic and ensures automatic screening initiation without coupling to downstream processes.

2. **Screening Evaluation Workflow**
   Triggered when screening results are submitted by the NCD App. A decision node evaluates blood sugar thresholds. For high-risk cases, the workflow creates a doctor referral task and sends patient notifications via SMS/IVR. This makes branching logic explicit and readable as a shared artifact.

3. **Follow-Up Care Workflow**
   Triggered when the doctor updates the care plan in the facility system. The workflow normalizes the payload, loads scheduling policy from a mocked MDMS configuration, computes recurrence rules, and creates recurring CHW follow-up tasks. This separates clinical intent from operational scheduling and avoids hard-coded logic.

Notes:
1. Mock MDMS was used to externalize policy such as allowed frequencies and SLA rules, demonstrating how workflows remain configurable across programs and tenants.
2. Sub-workflow invocation is implemented using event chaining rather than direct workflow calls, enabling loose coupling, asynchronous human actions, and better portability.
3. Dashboard updates are achieved indirectly through task and profile updates, which downstream analytics systems consume, avoiding direct dashboard dependencies inside workflows.



