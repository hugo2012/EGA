# 🔧 SWDD Configuration Reference - Exact Field Values

Complete binding tables and step configurations ready to copy into SWDD.

---

## SE11 STRUCTURE DEFINITION: ZAPP_PAR_APPROVER_ITEM

**Transaction:** SE11

### Steps to Create:

1. SE11 → Data Type → Create
2. Name: `ZAPP_PAR_APPROVER_ITEM`
3. Select: Structure
4. Copy the component table below:

### Component Table (Copy Each Row)

```
╔══════════════════╦═══════════════╦════════╦═════════════════════════════════╗
║ Component Name   ║ Data Type     ║ Length ║ Description                     ║
╠══════════════════╬═══════════════╬════════╬═════════════════════════════════╣
║ USERID           ║ CHAR          ║ 12     ║ SAP User ID / Agent             ║
╠══════════════════╬═══════════════╬════════╬═════════════════════════════════╣
║ NAME             ║ CHAR          ║ 40     ║ Approver Full Name              ║
╠══════════════════╬═══════════════╬════════╬═════════════════════════════════╣
║ ROLE             ║ CHAR          ║ 20     ║ Organizational Role             ║
╠══════════════════╬═══════════════╬════════╬═════════════════════════════════╣
║ LEVEL            ║ NUMC          ║ 2      ║ Approval Priority (1-9)         ║
╠══════════════════╬═══════════════╬════════╬═════════════════════════════════╣
║ DECISION         ║ CHAR          ║ 1      ║ A=Approved, R=Rejected, P=Pend  ║
╠══════════════════╬═══════════════╬════════╬═════════════════════════════════╣
║ COMMENTS         ║ CHAR          ║ 255    ║ Approver Comments / Feedback    ║
╠══════════════════╬═══════════════╬════════╬═════════════════════════════════╣
║ TIMESTAMP        ║ TIMESTAMP     ║        ║ Decision Timestamp              ║
╠══════════════════╬═══════════════╬════════╬═════════════════════════════════╣
║ REJECT_FLAG      ║ CHAR          ║ 1      ║ Quick Rejection Flag (X/blank)  ║
╚══════════════════╩═══════════════╩════════╩═════════════════════════════════╝
```

**After entering all components:**
- Click [Save]
- Click [Activate]
- Verify: "Structure ZAPP_PAR_APPROVER_ITEM created/updated successfully"

---

## SWDD CONTAINER ELEMENTS TABLE

**Transaction:** SWDD  
**Workflow:** WS90000231  
**Tab:** Container Elements

### Complete Container Elements List (Copy Each Row)

```
╔══════════════════════════════╦═════════════╦═══════════════╦════════════════════════════════╗
║ Element Name                 ║ Category    ║ Type          ║ Reference / Description        ║
╠══════════════════════════════╬═════════════╬═══════════════╬════════════════════════════════╣
║ t_parallel_approvers         ║ PARAMETER   ║ Table         ║ ZAPP_PAR_APPROVER_ITEM         ║
║                              ║             ║               ║ (for PARAFOREACH loop)         ║
╠══════════════════════════════╬═════════════╬═══════════════╬════════════════════════════════╣
║ current_approver             ║ PARAMETER   ║ Structure     ║ ZAPP_PAR_APPROVER_ITEM         ║
║                              ║             ║               ║ (loop element)                 ║
╠══════════════════════════════╬═════════════╬═══════════════╬════════════════════════════════╣
║ gv_parallel_rejection        ║ PARAMETER   ║ CHAR(1)       ║ Rejection flag (X/blank)       ║
║                              ║             ║               ║ Initial value: blank           ║
╠══════════════════════════════╬═════════════╬═══════════════╬════════════════════════════════╣
║ DOC_NUMBER                   ║ PARAMETER   ║ CHAR(10)      ║ Document ID (PO, Req, etc.)   ║
╠══════════════════════════════╬═════════════╬═══════════════╬════════════════════════════════╣
║ PROCESS_ID                   ║ PARAMETER   ║ CHAR(20)      ║ Unique Process ID              ║
║                              ║             ║               ║ (for tracking)                 ║
╠══════════════════════════════╬═════════════╬═══════════════╬════════════════════════════════╣
║ APPROVER1_ID                 ║ PARAMETER   ║ CHAR(12)      ║ Approver 1 User ID             ║
╠══════════════════════════════╬═════════════╬═══════════════╬════════════════════════════════╣
║ APPROVER2_ID                 ║ PARAMETER   ║ CHAR(12)      ║ Approver 2 User ID             ║
╠══════════════════════════════╬═════════════╬═══════════════╬════════════════════════════════╣
║ APPROVER6_ID                 ║ PARAMETER   ║ CHAR(12)      ║ Final Approver (6) User ID     ║
╠══════════════════════════════╬═════════════╬═══════════════╬════════════════════════════════╣
║ t_parallel_results           ║ PARAMETER   ║ Table         ║ ZAPP_PAR_APPROVER_ITEM         ║
║                              ║             ║               ║ (optional: aggregated results) ║
╚══════════════════════════════╩═════════════╩═══════════════╩════════════════════════════════╝
```

**Steps:**
1. Tab: "Container Elements"
2. For each row: Click [New Row]
3. Fill Element Name, Category, Type, Reference exactly as shown
4. Click [Save]

---

## STEP 1 CONFIGURATION: STEP1_APPROVER1_CALL

**Step Type:** Work Item → Call Sub-workflow  
**Workflow:** WS900000232

### Basic Data

```
Field Name                  Value
─────────────────────────────────────────────────────
Step Name                   STEP1_APPROVER1_CALL
Step Type                   Work Item
Sub-workflow                WS900000232
Processing Deadline         2 Days
Escalation Deadline         1 Day
Escalation Recipient        (optional: manager)
```

### Agent

```
Field Name                  Value
─────────────────────────────────────────────────────
Recipient Type              User
Recipient                   APPROVER1_ID
Role/User/Mail Selection    USER
```

### Binding Tab: IMPORT (TO Sub-workflow)

```
┌────────────────────────────────────┬────────────────────────────────┐
│ Sub-workflow Parameter             │ Container Element              │
├────────────────────────────────────┼────────────────────────────────┤
│ IV_APPROVER_ID                     │ ← &APPROVER1_ID&               │
│ IV_DOC_NUMBER                      │ ← &DOC_NUMBER&                 │
│ IV_PROCESS_ID                      │ ← &PROCESS_ID&                 │
└────────────────────────────────────┴────────────────────────────────┘
```

### Binding Tab: EXPORT (FROM Sub-workflow)

```
┌────────────────────────────────────┬────────────────────────────────┐
│ Container Element                  │ Sub-workflow Parameter         │
├────────────────────────────────────┼────────────────────────────────┤
│ (local decision variable)           │ ← EV_DECISION                  │
└────────────────────────────────────┴────────────────────────────────┘
```

### Decision Routing

```
After Step 1 Completion:
├─ IF EV_DECISION = 'R' (REJECTED)
│  └─→ Route to: EXIT_REJECTION_STEP
│
└─ IF EV_DECISION = 'A' (APPROVED)
   └─→ Route to: STEP2_APPROVER2_CALL
```

---

## STEP 2 CONFIGURATION: STEP2_APPROVER2_CALL

**Step Type:** Work Item → Call Sub-workflow  
**Workflow:** WS900000232

### Basic Data

```
Field Name                  Value
─────────────────────────────────────────────────────
Step Name                   STEP2_APPROVER2_CALL
Step Type                   Work Item
Sub-workflow                WS900000232
Processing Deadline         2 Days
Escalation Deadline         1 Day
```

### Agent

```
Field Name                  Value
─────────────────────────────────────────────────────
Recipient Type              User
Recipient                   APPROVER2_ID
Role/User/Mail Selection    USER
```

### Binding Tab: IMPORT

```
┌────────────────────────────────────┬────────────────────────────────┐
│ Sub-workflow Parameter             │ Container Element              │
├────────────────────────────────────┼────────────────────────────────┤
│ IV_APPROVER_ID                     │ ← &APPROVER2_ID&               │
│ IV_DOC_NUMBER                      │ ← &DOC_NUMBER&                 │
│ IV_PROCESS_ID                      │ ← &PROCESS_ID&                 │
└────────────────────────────────────┴────────────────────────────────┘
```

### Binding Tab: EXPORT

```
┌────────────────────────────────────┬────────────────────────────────┐
│ Container Element                  │ Sub-workflow Parameter         │
├────────────────────────────────────┼────────────────────────────────┤
│ (local decision variable)           │ ← EV_DECISION                  │
└────────────────────────────────────┴────────────────────────────────┘
```

### Decision Routing

```
├─ IF EV_DECISION = 'R' (REJECTED)
│  └─→ Route to: EXIT_REJECTION_STEP
│
└─ IF EV_DECISION = 'A' (APPROVED)
   └─→ Route to: STEP3_FETCH_PARALLEL_APPROVERS
```

---

## STEP 3 CONFIGURATION: STEP3_FETCH_PARALLEL_APPROVERS

**Step Type:** Activity → Function Module (Automatic)  
**Function Module:** ZFILL_PARALLEL_APPROVERS

### Basic Data

```
Field Name                  Value
─────────────────────────────────────────────────────
Step Name                   STEP3_FETCH_PARALLEL_APPROVERS
Step Type                   Activity
Processing Type             Background Task
Function Module             ZFILL_PARALLEL_APPROVERS
Processing                  Automatic
Deadline                    (none)
```

### Binding Tab: IMPORT (TO Function)

```
┌────────────────────────────────────┬────────────────────────────────┐
│ FM Parameter                       │ Container Element              │
├────────────────────────────────────┼────────────────────────────────┤
│ IV_DOC_NUMBER                      │ ← &DOC_NUMBER&                 │
│ IV_DOC_TYPE                        │ ← 'PO' (or your doc type)      │
└────────────────────────────────────┴────────────────────────────────┘
```

### Binding Tab: EXPORT (FROM Function)

```
┌────────────────────────────────────┬────────────────────────────────┐
│ Container Element                  │ FM Parameter                   │
├────────────────────────────────────┼────────────────────────────────┤
│ &t_parallel_approvers&             │ ← ET_PAR_APPROVER_LIST         │
└────────────────────────────────────┴────────────────────────────────┘
```

### Decision Routing

```
After Step 3 Completion:
└─→ Continue to: PARAFOREACH_BLOCK (Step 4)
```

---

## STEP 4 CONFIGURATION: PARAFOREACH BLOCK

**Block Type:** PARAFOREACH  
**Loop Container:** t_parallel_approvers  
**Loop Element:** current_approver

### PARAFOREACH Block Settings

```
┌────────────────────────────────────┬────────────────────────────────┐
│ Field Name                         │ Value                          │
├────────────────────────────────────┼────────────────────────────────┤
│ Block Name                         │ PARALLEL_APPROVERS_BLOCK       │
├────────────────────────────────────┼────────────────────────────────┤
│ Loop Container                     │ t_parallel_approvers           │
├────────────────────────────────────┼────────────────────────────────┤
│ Loop Element                       │ current_approver               │
├────────────────────────────────────┼────────────────────────────────┤
│ Create New Context                 │ ☑ YES                         │
├────────────────────────────────────┼────────────────────────────────┤
│ Max Parallel Branches              │ 5                              │
├────────────────────────────────────┼────────────────────────────────┤
│ Allow Re-blocking                  │ ☐ NO                          │
├────────────────────────────────────┼────────────────────────────────┤
│ Collect Results                    │ ☑ YES                         │
├────────────────────────────────────┼────────────────────────────────┤
│ Result Container                   │ t_parallel_approvers           │
├────────────────────────────────────┼────────────────────────────────┤
│ Skip Empty Loops                   │ ☑ YES                         │
└────────────────────────────────────┴────────────────────────────────┘
```

---

## STEP 4 INNER: STEP4_PARALLEL_APPROVER_CALL (Inside PARAFOREACH)

**Step Type:** Work Item → Call Sub-workflow  
**Workflow:** WS900000232

### Basic Data

```
Field Name                  Value
─────────────────────────────────────────────────────
Step Name                   STEP4_PARALLEL_APPROVER_CALL
Step Type                   Work Item
Sub-workflow                WS900000232
Processing Deadline         2 Days
Escalation Deadline         1 Day
```

### Agent (CRITICAL)

```
Field Name                  Value
─────────────────────────────────────────────────────
Recipient Type              User
Recipient                   &current_approver-USERID&
Role/User/Mail Selection    USER
```

⚠️ **IMPORTANT:** Use loop element `&current_approver-USERID&` not hardcoded user!

### Binding Tab: IMPORT (TO Sub-workflow)

```
┌────────────────────────────────────┬────────────────────────────────┐
│ Sub-workflow Parameter             │ Container Element              │
├────────────────────────────────────┼────────────────────────────────┤
│ IV_APPROVER_ID                     │ ← &current_approver-USERID&    │
│ IV_APPROVER_NAME                   │ ← &current_approver-NAME&      │
│ IV_DOC_NUMBER                      │ ← &DOC_NUMBER&                 │
│ IV_PROCESS_ID                      │ ← &PROCESS_ID&                 │
│ IV_APPROVAL_LEVEL                  │ ← &current_approver-LEVEL&     │
└────────────────────────────────────┴────────────────────────────────┘
```

### Binding Tab: EXPORT (FROM Sub-workflow)

```
┌────────────────────────────────────┬────────────────────────────────┐
│ Container Element                  │ Sub-workflow Parameter         │
├────────────────────────────────────┼────────────────────────────────┤
│ &current_approver-DECISION&        │ ← EV_DECISION                  │
│ &current_approver-COMMENTS&        │ ← EV_COMMENTS                  │
│ &current_approver-TIMESTAMP&       │ ← EV_DECISION_TIME             │
└────────────────────────────────────┴────────────────────────────────┘
```

### Decision Routing

```
Inside PARAFOREACH (per iteration):
└─→ Continue (no decision needed - all iterations run)

After PARAFOREACH completes:
└─→ Auto-sync point (workflow waits for all branches)
    then continues to: STEP5_CHECK_PARALLEL_REJECTION
```

---

## STEP 5 CONFIGURATION: STEP5_CHECK_PARALLEL_REJECTION

**Step Type:** Activity (Automatic)  
**Form Routine:** ZFORM_CHECK_PARALLEL_REJECTION

### Basic Data

```
Field Name                  Value
─────────────────────────────────────────────────────
Step Name                   STEP5_CHECK_PARALLEL_REJECTION
Step Type                   Activity
Processing Type             Automatic
Form Routine                ZFORM_CHECK_PARALLEL_REJECTION
Workflow                    WS90000231
Deadline                    (none)
```

### Binding Tab

```
NO BINDING NEEDED
(Form accesses container directly via container-GET_ELEMENT)

After Form Execution:
└─→ gv_parallel_rejection will be set to 'X' or ' ' (blank)
```

### Decision Routing

```
After Step 5 Completion:
└─→ Continue to: DECISION_GATEWAY
```

---

## STEP 6 CONFIGURATION: DECISION_GATEWAY

**Step Type:** Decision (Automatic)

### Decision Logic

```
┌────────────────────────────────────────────────────────────────────┐
│ Condition                          │ Route To                       │
├────────────────────────────────────┼────────────────────────────────┤
│ &gv_parallel_rejection& = 'X'     │ EXIT_PPM_REJECTION             │
│                                    │ (Rejection detected)           │
├────────────────────────────────────┼────────────────────────────────┤
│ &gv_parallel_rejection& != 'X'    │ STEP6_FINAL_APPROVER_CALL      │
│                                    │ (No rejection, continue)       │
└────────────────────────────────────┴────────────────────────────────┘
```

---

## STEP 7 CONFIGURATION: STEP6_FINAL_APPROVER_CALL

**Step Type:** Work Item → Call Sub-workflow  
**Workflow:** WS900000232

### Basic Data

```
Field Name                  Value
─────────────────────────────────────────────────────
Step Name                   STEP6_FINAL_APPROVER_CALL
Step Type                   Work Item
Sub-workflow                WS900000232
Processing Deadline         2 Days
Escalation Deadline         1 Day
```

### Agent

```
Field Name                  Value
─────────────────────────────────────────────────────
Recipient Type              User
Recipient                   APPROVER6_ID
Role/User/Mail Selection    USER
```

### Binding Tab: IMPORT

```
┌────────────────────────────────────┬────────────────────────────────┐
│ Sub-workflow Parameter             │ Container Element              │
├────────────────────────────────────┼────────────────────────────────┤
│ IV_APPROVER_ID                     │ ← &APPROVER6_ID&               │
│ IV_DOC_NUMBER                      │ ← &DOC_NUMBER&                 │
│ IV_PROCESS_ID                      │ ← &PROCESS_ID&                 │
└────────────────────────────────────┴────────────────────────────────┘
```

### Binding Tab: EXPORT

```
┌────────────────────────────────────┬────────────────────────────────┐
│ Container Element                  │ Sub-workflow Parameter         │
├────────────────────────────────────┼────────────────────────────────┤
│ (optional - local variable)         │ ← EV_DECISION                  │
└────────────────────────────────────┴────────────────────────────────┘
```

### Decision Routing

```
After Step 6 Completion:
└─→ Complete workflow
    (Route to END)
```

---

## EXIT STEP CONFIGURATION: EXIT_REJECTION_STEP

**Step Type:** End (Termination)

### Basic Data

```
Field Name                  Value
─────────────────────────────────────────────────────
Step Name                   EXIT_REJECTION_STEP
Step Type                   Step (Termination)
Status on Exit              Rejected
Message                     (optional) Approval rejected by approver
```

### Binding (Optional)

```
Update Document Status = REJECTED
(if your system has status update logic)
```

---

## EXIT STEP CONFIGURATION: EXIT_PPM_REJECTION

**Step Type:** End (Termination)

### Basic Data

```
Field Name                  Value
─────────────────────────────────────────────────────
Step Name                   EXIT_PPM_REJECTION
Step Type                   Step (Termination)
Status on Exit              Rejected
Message                     Parallel approval rejection detected
                           Trigger PPM Flow Status Rejection
```

### Optional: Trigger PPM Status Update

```
(If PPM integration exists)
Before termination:
└─→ Call Function Module to update PPM status to "REJECTED"
    (e.g., ZUPDATE_PPM_STATUS with parameters IV_DOC_ID, IV_STATUS='REJ')
```

---

## SUMMARY: Complete Step Sequence

```
┌─────────────┐
│   START     │
└──────┬──────┘
       │
       ├─→ [STEP1_APPROVER1_CALL] ──┐
       │                            │
       │                      ┌─────▼─────┐
       │                      │ Decision  │
       │                      │ R? → EXIT │
       │                      │ A? → cont │
       │                      └─────┬─────┘
       │                            │
       ├─→ [STEP2_APPROVER2_CALL] ──┤
       │                            │
       │                      ┌─────▼─────┐
       │                      │ Decision  │
       │                      │ R? → EXIT │
       │                      │ A? → cont │
       │                      └─────┬─────┘
       │                            │
       ├─→ [STEP3_FETCH_PARALLEL_APPROVERS] ──┐
       │   (Populates t_parallel_approvers)   │
       │                                      │
       ├─→ [PARAFOREACH: STEP4_PARALLEL_APPROVER_CALL] ──┐
       │   • Loop: t_parallel_approvers                  │
       │   • Element: current_approver                   │
       │   • Max: 5 concurrent                           │
       │   • Collect Results: YES                        │
       │                                                  │
       ├─→ [STEP5_CHECK_PARALLEL_REJECTION] ──┐
       │   (Sets gv_parallel_rejection)       │
       │                                      │
       ├─→ [DECISION_GATEWAY] ──┬──────────────┐
       │                        │              │
       │                  X='X' │         X=' '│
       │                        │              │
       └─→ [EXIT_PPM_REJECTION] │   [STEP6_FINAL_APPROVER_CALL]
           (Reject)             │
                                └──→ [END] (Complete)
```

---

## Quick Reference: All Binding Expressions

**Use these exact expressions (copy-paste) in SWDD binding tabs:**

```
SINGLE APPROVERS:
  Agent: &APPROVER1_ID&, &APPROVER2_ID&, &APPROVER6_ID&
  Document: &DOC_NUMBER&
  Process: &PROCESS_ID&

PARALLEL BLOCK (PARAFOREACH):
  Agent: &current_approver-USERID&
  Decision IN: &current_approver-DECISION&
  Decision OUT: EV_DECISION
  Comments: &current_approver-COMMENTS&, EV_COMMENTS

REJECTION FLAG:
  Check: &gv_parallel_rejection& = 'X'
  Set by: ZFORM_CHECK_PARALLEL_REJECTION
```

---

## Final Checklist Before Saving SWDD

- [ ] All 8 container elements defined (Container Elements tab)
- [ ] Step 1: Recipient = APPROVER1_ID, bindings correct
- [ ] Step 2: Recipient = APPROVER2_ID, bindings correct
- [ ] Step 3: FM ZFILL_PARALLEL_APPROVERS, binding returns to t_parallel_approvers
- [ ] Step 4 (PARAFOREACH):
  - [ ] Loop Container = t_parallel_approvers
  - [ ] Loop Element = current_approver
  - [ ] Max Parallel = 5
  - [ ] Collect Results = YES
- [ ] Step 4 Inner:
  - [ ] Agent = &current_approver-USERID& (NOT hardcoded!)
  - [ ] Binding: &current_approver-DECISION& ← EV_DECISION
- [ ] Step 5: Form ZFORM_CHECK_PARALLEL_REJECTION
- [ ] Step 6 (Decision): IF gv_parallel_rejection='X' → EXIT_PPM_REJECTION
- [ ] Step 7: Final Approver = APPROVER6_ID
- [ ] All steps connected (no broken flows)
- [ ] Save: Ctrl+S
- [ ] Check: Ctrl+F2 (Syntax check - no errors)
- [ ] Activate: Ctrl+F3

---

**After completing all configurations, proceed to TESTING section in PARALLEL_WORKFLOW_IMPLEMENTATION_FINAL.md**
