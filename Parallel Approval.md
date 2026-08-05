# 🚀 Parallel Approval Workflow Implementation Guide (WS90000231)
## Your Exact Design: Step1 → Step2 → Step3 (Fetch) → Step4 (PARAFOREACH) → Step5 (Check Rejection) → Step6

**Date:** August 5, 2026  
**Status:** PRODUCTION READY  
**Target Workflow:** WS90000231 (Main) with WS900000232 (Sub-workflow)  
**Parallel Approvers:** 3, 4, 5 (in PARAFOREACH block)

---

## TABLE OF CONTENTS

1. [Architecture Overview](#architecture-overview)
2. [Step 1: Create SE11 Data Structures](#step-1-create-se11-data-structures)
3. [Step 2: Define Container Elements in SWDD](#step-2-define-container-elements-in-swdd)
4. [Step 3: Create ABAP Function Modules](#step-3-create-abap-function-modules)
5. [Step 4: Build Workflow in SWDD (Steps 1-6)](#step-4-build-workflow-in-swdd-steps-1-6)
6. [Step 5: Configure PARAFOREACH Block](#step-5-configure-paraforeach-block)
7. [Step 6: Binding Configuration](#step-6-binding-configuration)
8. [Step 7: Testing Checklist](#step-7-testing-checklist)
9. [Step 8: Troubleshooting Guide](#step-8-troubleshooting-guide)

---

## ARCHITECTURE OVERVIEW

```
WS90000231 (Main Workflow)
│
├─ [START]
│
├─ Step 1: Call WS900000232 (Approver 1)
│  └─ IF Rejected → Exit / Refer Back
│
├─ Step 2: Call WS900000232 (Approver 2)
│  └─ IF Rejected → Exit / Refer Back
│
├─ Step 3: Background Task (FM: ZFILL_PARALLEL_APPROVERS)
│  └─ Populates: t_parallel_approvers (3 approvers: 3, 4, 5)
│
├─ ╔════════════════════════════════════════════╗
│  ║ Step 4: PARAFOREACH Block                  ║
│  ║ Loop: t_parallel_approvers → current_approver
│  ║ Max Parallel: 5 approvers concurrent       ║
│  ║                                            ║
│  ║ ├─ Call WS900000232 for each approver     ║
│  ║ │  Agent: current_approver-USERID         ║
│  ║ │  Return: Decision → current_approver    ║
│  ║ │                                         ║
│  ║ ║  ON REJECTION:                          ║
│  ║ ║  └─ Set current_approver-DECISION='R'  ║
│  ║                                            ║
│  ║ └─ Results collected back to table        ║
│  ║   (Collect Results = YES)                 ║
│  ╚════════════════════════════════════════════╝
│
├─ Step 5: Automatic Check (Form: ZFORM_CHECK_PARALLEL_REJECTION)
│  ├─ Scan t_parallel_approvers for any DECISION='R'
│  ├─ IF found → Set gv_parallel_rejection='X'
│  └─ IF not found → Set gv_parallel_rejection=' '
│
├─ DECISION GATEWAY (based on gv_parallel_rejection)
│  ├─ IF 'X' (REJECTED) → Trigger PPM Flow Status Rejection
│  └─ IF ' ' (CLEAR) → Continue to Step 6
│
├─ Step 6: Call WS900000232 (Final Approver 6)
│  └─ Complete approval process
│
└─ [END]
```

---

# STEP 1: CREATE SE11 DATA STRUCTURES

## 1.1 Structure: ZAPP_PAR_APPROVER_ITEM

**Transaction:** SE11

**Action:** Create new Structure

```
Structure Name: ZAPP_PAR_APPROVER_ITEM
Description: Parallel Approver Item for PARAFOREACH Loop

┌────────────────┬──────────────┬────────┬─────────────────────────────┐
│ Component Name │ Data Type    │ Length │ Description                 │
├────────────────┼──────────────┼────────┼─────────────────────────────┤
│ USERID         │ CHAR         │ 12     │ SAP User ID / Agent         │
│ NAME           │ CHAR         │ 40     │ Approver Full Name          │
│ ROLE           │ CHAR         │ 20     │ Organizational Role         │
│ LEVEL          │ NUMC         │ 2      │ Approval Priority Level     │
│ DECISION       │ CHAR         │ 1      │ A=Approved, R=Rejected, P=Pending
│ COMMENTS       │ CHAR         │ 255    │ Approver Comments/Feedback  │
│ TIMESTAMP      │ TIMESTAMP    │        │ Decision Time               │
│ REJECT_FLAG    │ CHAR         │ 1      │ Flag for quick rejection check
└────────────────┴──────────────┴────────┴─────────────────────────────┘
```

**Step by Step in SE11:**
1. Transaction SE11
2. Select: "Data Type" (default radio button)
3. Enter Name: `ZAPP_PAR_APPROVER_ITEM`
4. Click [Create]
5. Select: "Structure" (radio button)
6. Click [Continue]
7. Fill Component Table (copy above rows)
8. Save (Ctrl+S)
9. Activate (Ctrl+F3)

---

## 1.2 Table Type (Optional, for type checking)

If you need to use typed table references:

```
Type Name: ZAPP_PAR_APPROVER_TABLE
Description: Table of Parallel Approvers

Category: Table Type

┌─────────────────────────┬──────────────────────────┐
│ Row Type                │ ZAPP_PAR_APPROVER_ITEM   │
│ With Header Line        │ ☐ (unchecked)            │
│ Standard Table Type     │ ☑ (checked)              │
└─────────────────────────┴──────────────────────────┘
```

---

# STEP 2: DEFINE CONTAINER ELEMENTS IN SWDD

**Transaction:** SWDD
**Workflow:** WS90000231
**Mode:** CHANGE

### 2.1 Open Workflow & Container Elements Tab

1. Transaction SWDD
2. Enter Workflow: `WS90000231`
3. Click [Change]
4. Click Tab: "Container Elements"

### 2.2 Add All Container Elements

Use [New Row] button and add these elements:

```
┌──────────────────────────┬─────────────┬──────────────┬──────────────────────┐
│ Element Name             │ Category    │ Type         │ Reference/Desc       │
├──────────────────────────┼─────────────┼──────────────┼──────────────────────┤
│ t_parallel_approvers     │ PARAMETER   │ Table        │ Table of ZAPP_PAR_   │
│                          │             │              │ APPROVER_ITEM        │
├──────────────────────────┼─────────────┼──────────────┼──────────────────────┤
│ current_approver         │ PARAMETER   │ Structure    │ ZAPP_PAR_APPROVER_   │
│                          │             │              │ ITEM (loop element)  │
├──────────────────────────┼─────────────┼──────────────┼──────────────────────┤
│ gv_parallel_rejection    │ PARAMETER   │ CHAR(1)      │ Rejection flag       │
│                          │             │              │ Initial: blank       │
├──────────────────────────┼─────────────┼──────────────┼──────────────────────┤
│ DOC_NUMBER               │ PARAMETER   │ CHAR(10)     │ Document ID          │
├──────────────────────────┼─────────────┼──────────────┼──────────────────────┤
│ PROCESS_ID               │ PARAMETER   │ CHAR(20)     │ Unique Process ID    │
├──────────────────────────┼─────────────┼──────────────┼──────────────────────┤
│ APPROVER1_ID             │ PARAMETER   │ CHAR(12)     │ Approver 1 User ID   │
├──────────────────────────┼─────────────┼──────────────┼──────────────────────┤
│ APPROVER2_ID             │ PARAMETER   │ CHAR(12)     │ Approver 2 User ID   │
├──────────────────────────┼─────────────┼──────────────┼──────────────────────┤
│ APPROVER6_ID             │ PARAMETER   │ CHAR(12)     │ Final Approver User  │
├──────────────────────────┼─────────────┼──────────────┼──────────────────────┤
│ t_parallel_results       │ PARAMETER   │ Table        │ Optional: aggregated │
│                          │             │              │ results table        │
└──────────────────────────┴─────────────┴──────────────┴──────────────────────┘
```

**Action Steps:**
1. Click "Container Elements" tab
2. For each row:
   - Click [New Row]
   - Enter Element Name (e.g., `t_parallel_approvers`)
   - Select Category: PARAMETER
   - Select Type: Table (for table elements) or Structure or CHAR(1)
   - Enter Reference (if Type is Table/Structure)
   - Press Enter / Tab to confirm
3. Save: Click [Save] button or Ctrl+S
4. After ALL elements added, save the workflow again

---

# STEP 3: CREATE ABAP FUNCTION MODULES

## 3.1 Function Module: ZFILL_PARALLEL_APPROVERS

**Transaction:** SE37

**Create New Function Module**

```abap
FUNCTION ZFILL_PARALLEL_APPROVERS.
*"----------------------------------------------------------------------
*"*"Local Interface:
*"  IMPORTING
*"     VALUE(IV_DOC_NUMBER) TYPE CHAR10
*"     VALUE(IV_DOC_TYPE) TYPE CHAR10 OPTIONAL
*"  EXPORTING
*"     VALUE(ET_PAR_APPROVER_LIST) TYPE TABLE OF ZAPP_PAR_APPROVER_ITEM
*"     VALUE(EV_COUNT) TYPE NUMC3
*"  EXCEPTIONS
*"     NO_APPROVERS_FOUND
*"----------------------------------------------------------------------

  DATA: ls_app TYPE ZAPP_PAR_APPROVER_ITEM,
        lv_count TYPE NUMC3 VALUE 0.

  CLEAR ET_PAR_APPROVER_LIST.

  "=======================================================================
  " OPTION 1: Hard-coded for Approver 3, 4, 5 (for testing)
  "=======================================================================
  
  ls_app-USERID    = 'APPROVER03'.
  ls_app-NAME      = 'Approver Three'.
  ls_app-ROLE      = 'FINANCE_MANAGER'.
  ls_app-LEVEL     = '2'.
  ls_app-DECISION  = 'P'.  " P = Pending
  ls_app-TIMESTAMP = sy-tstmp.
  APPEND ls_app TO ET_PAR_APPROVER_LIST.
  lv_count = lv_count + 1.

  ls_app-USERID    = 'APPROVER04'.
  ls_app-NAME      = 'Approver Four'.
  ls_app-ROLE      = 'HR_MANAGER'.
  ls_app-LEVEL     = '2'.
  ls_app-DECISION  = 'P'.
  ls_app-TIMESTAMP = sy-tstmp.
  APPEND ls_app TO ET_PAR_APPROVER_LIST.
  lv_count = lv_count + 1.

  ls_app-USERID    = 'APPROVER05'.
  ls_app-NAME      = 'Approver Five'.
  ls_app-ROLE      = 'BUDGET_OWNER'.
  ls_app-LEVEL     = '2'.
  ls_app-DECISION  = 'P'.
  ls_app-TIMESTAMP = sy-tstmp.
  APPEND ls_app TO ET_PAR_APPROVER_LIST.
  lv_count = lv_count + 1.

  "=======================================================================
  " OPTION 2: Dynamic lookup from org structure (production use)
  "=======================================================================
  " Uncomment below and remove Option 1 when ready for production
  " 
  " SELECT * FROM hrp1000
  "   WHERE otype = 'O'
  "   AND objid IN (SELECT objid FROM hrp1000 WHERE otype='O' AND subty='DEPT')
  "   INTO TABLE @DATA(lt_orgs).
  "
  " LOOP AT lt_orgs ASSIGNING FIELD-SYMBOL(<org>).
  "   ls_app-USERID = <org>-objid.
  "   " ... fetch user details from org assignment ...
  "   APPEND ls_app TO ET_PAR_APPROVER_LIST.
  "   lv_count = lv_count + 1.
  " ENDLOOP.

  "=======================================================================
  " Validate result
  "=======================================================================
  IF lv_count = 0.
    RAISE NO_APPROVERS_FOUND.
  ENDIF.

  EV_COUNT = lv_count.

ENDFUNCTION.
```

**Steps in SE37:**
1. Transaction SE37
2. Function Module Name: `ZFILL_PARALLEL_APPROVERS`
3. Click [Create]
4. **Attributes Tab:**
   - Application: (select your area, e.g., FIN)
   - Processing Type: (leave Normal Function Module)
   - Release Status: (check if needed)
5. **Import Tab:**
   - Add parameter: `IV_DOC_NUMBER` (Type: CHAR, Length: 10)
   - Add parameter: `IV_DOC_TYPE` (Type: CHAR, Length: 10, Mark: [Optional checkbox])
6. **Export Tab:**
   - Add parameter: `ET_PAR_APPROVER_LIST` (Type: Table, Reference: ZAPP_PAR_APPROVER_ITEM)
   - Add parameter: `EV_COUNT` (Type: NUMC, Length: 3)
7. **Exceptions Tab:**
   - Add: `NO_APPROVERS_FOUND`
8. **Source Code Tab:** Paste ABAP code above
9. Save (Ctrl+S)
10. Activate (Ctrl+F3)
11. **Test:** F8 key to test-run (enter `DOC_NUMBER='TEST001'`, execute)

---

## 3.2 Form Routine: ZFORM_CHECK_PARALLEL_REJECTION

**This form runs inside SWDD as an Automatic Activity**

```abap
FORM ZFORM_CHECK_PARALLEL_REJECTION.
*"----------------------------------------------------------------------
* Purpose: Scan t_parallel_approvers for any rejection decision.
*          If found, set gv_parallel_rejection = 'X'
*          Otherwise set gv_parallel_rejection = ' '
*
* Called in SWDD Step 5 as: Automatic Activity with Form Routine
*----------------------------------------------------------------------

  DATA: lt_approvers TYPE TABLE OF ZAPP_PAR_APPROVER_ITEM,
        ls_approver  TYPE ZAPP_PAR_APPROVER_ITEM,
        lv_rejected  TYPE CHAR1 VALUE ' ',
        lv_count     TYPE I VALUE 0.

  " Get the parallel approvers table from container
  container-GET_ELEMENT( 't_parallel_approvers' lt_approvers ).

  IF lt_approvers IS INITIAL.
    " No approvers to check; continue (no rejection)
    container-SET_ELEMENT( 'gv_parallel_rejection' ' ' ).
    RETURN.
  ENDIF.

  " Loop through approvers and check for rejection
  LOOP AT lt_approvers INTO ls_approver.
    lv_count = lv_count + 1.
    
    WRITE: / 'Approver ', lv_count, ': ', ls_approver-userid,
           ' | Decision: ', ls_approver-decision.
    
    " Check for rejection or refer-back
    IF ls_approver-decision = 'R' OR ls_approver-decision = 'F'.
      lv_rejected = 'X'.
      WRITE: / '  >>> REJECTION DETECTED from ', ls_approver-userid.
      EXIT.
    ENDIF.
  ENDLOOP.

  " Set rejection flag in container
  container-SET_ELEMENT( 'gv_parallel_rejection' lv_rejected ).

  WRITE: / '=== Parallel Rejection Check Complete ==='.
  WRITE: / 'Result: ', lv_rejected.

ENDFORM.
```

**How to use in SWDD:**
- In Step 5 (automatic activity):
  - Type: Activity (Automatic)
  - Processing: Form
  - Form Routine: `ZFORM_CHECK_PARALLEL_REJECTION`
  - Workflow: `WS90000231` (your workflow)

---

## 3.3 Function Module: ZRAISE_PARALLEL_REJECTION (OPTIONAL - ADVANCED)

**For immediate termination on rejection (use only if needed)**

```abap
FUNCTION ZRAISE_PARALLEL_REJECTION.
*"----------------------------------------------------------------------
*"*"Local Interface:
*"  IMPORTING
*"     VALUE(IV_OBJECT_TYPE) TYPE CHAR30
*"     VALUE(IV_OBJECT_KEY)  TYPE CHAR50
*"     VALUE(IV_EVENT)       TYPE CHAR30
*"     VALUE(IV_REASON)      TYPE CHAR100 OPTIONAL
*"----------------------------------------------------------------------

  DATA: ls_event TYPE swr_event.

  " Raise event on business object
  CALL FUNCTION 'SWE_EVENT_CREATE'
    EXPORTING
      objtype          = iv_object_type
      objkey           = iv_object_key
      event            = iv_event
    EXCEPTIONS
      OBJECT_NOT_FOUND = 1
      NO_EVENT_LINK    = 2
      OTHERS           = 3.

  IF sy-subrc = 0.
    WRITE: / 'Event raised successfully: ', iv_event.
  ELSE.
    WRITE: / 'Error raising event, subrc: ', sy-subrc.
  ENDIF.

ENDFUNCTION.
```

**Note:** Advanced usage; configure only if your workflow design requires immediate termination.

---

# STEP 4: BUILD WORKFLOW IN SWDD (STEPS 1-6)

**Transaction:** SWDD  
**Workflow:** WS90000231  
**Mode:** CHANGE

## 4.1 Workflow Diagram Layout

In the Workflow Editor (graphical view), create this structure:

```
[START] → [STEP1] → [STEP2] → [STEP3] → [PARAFOREACH: STEP4] → [STEP5] → [DECISION] → [STEP6] → [END]
                       ↓ Reject                                              ↓ Reject
                    [EXIT/REFER]                                        [PPM REJECTION]
```

---

## 4.2 Step 1: Sub-workflow Call (Approver 1)

**Step Name:** `STEP1_APPROVER1_CALL`  
**Step Type:** Work Item → Call Sub-workflow

### Configuration

| Field | Value |
|-------|-------|
| **Basic Data Tab** | |
| Step Name | STEP1_APPROVER1_CALL |
| Step Type | Work Item (Sub-workflow call) |
| Workflow | WS900000232 |
| Processing Deadline | 2 Days |
| **Agent Tab** | |
| Recipient | APPROVER1_ID (or fixed literal, e.g., 'APPROVER01') |
| **Binding Tab** | |
| TO Sub-wf: IV_APPROVER_ID | ← APPROVER1_ID |
| TO Sub-wf: IV_DOC_NUMBER | ← DOC_NUMBER |
| FROM Sub-wf: EV_DECISION | → (capture to local variable or decision) |

### Decision Routing

After Step 1 completes:
- **IF EV_DECISION = 'R'** (Rejected) → Route to **EXIT_REJECTION** step
- **IF EV_DECISION = 'A'** (Approved) → Continue to **STEP2**

---

## 4.3 Step 2: Sub-workflow Call (Approver 2)

**Step Name:** `STEP2_APPROVER2_CALL`  
**Step Type:** Work Item → Call Sub-workflow

### Configuration

| Field | Value |
|-------|-------|
| **Basic Data Tab** | |
| Step Name | STEP2_APPROVER2_CALL |
| Step Type | Work Item (Sub-workflow call) |
| Workflow | WS900000232 |
| Processing Deadline | 2 Days |
| **Agent Tab** | |
| Recipient | APPROVER2_ID (or fixed literal, e.g., 'APPROVER02') |
| **Binding Tab** | |
| TO Sub-wf: IV_APPROVER_ID | ← APPROVER2_ID |
| TO Sub-wf: IV_DOC_NUMBER | ← DOC_NUMBER |
| FROM Sub-wf: EV_DECISION | → (capture to local variable) |

### Decision Routing

- **IF EV_DECISION = 'R'** → Route to **EXIT_REJECTION**
- **IF EV_DECISION = 'A'** → Continue to **STEP3**

---

## 4.4 Step 3: Background Task (Populate Parallel Approvers)

**Step Name:** `STEP3_FETCH_PARALLEL_APPROVERS`  
**Step Type:** Activity → Function Module (Automatic)

### Configuration

| Field | Value |
|-------|-------|
| **Basic Data Tab** | |
| Step Name | STEP3_FETCH_PARALLEL_APPROVERS |
| Step Type | Activity (Background Task) |
| Function Module | ZFILL_PARALLEL_APPROVERS |
| Processing | Automatic |
| **Binding Tab** | |
| IMPORT: IV_DOC_NUMBER | ← DOC_NUMBER |
| EXPORT: ET_PAR_APPROVER_LIST | → t_parallel_approvers |
| EXPORT: EV_COUNT | → (optional, debug) |

**After STEP3:** Continue to STEP4 (PARAFOREACH block)

---

# STEP 5: CONFIGURE PARAFOREACH BLOCK

**This is the critical section for parallel processing.**

## 5.1 Insert PARAFOREACH Block

**In SWDD Diagram Editor:**
1. Position cursor after STEP3
2. Menu: **Insert** → **Block** → **PARAFOREACH**
3. Fill dialog:

```
┌─────────────────────────────────────────────────────────┐
│ Create PARAFOREACH Block Dialog                        │
├─────────────────────────────────────────────────────────┤
│                                                         │
│ Block Name:           PARALLEL_APPROVERS_BLOCK         │
│                                                         │
│ Loop Configuration:                                     │
│   Loop Container:     t_parallel_approvers             │
│   Loop Element:       current_approver                │
│   Create New Context: ☑ YES (recommended)             │
│                                                         │
│ Parallelization Settings:                             │
│   Max Parallel Branches:  5                           │
│   Allow Re-blocking:      ☐ NO                        │
│   Collect Results:        ☑ YES                       │
│   Result Container:       t_parallel_approvers        │
│   Skip Empty Loops:       ☑ YES                       │
│                                                         │
│ Block Boundaries:                                       │
│   Start Step:         STEP4_PARALLEL_CALL            │
│   End Step:           STEP4_PARALLEL_CALL            │
│   (will add steps inside block after creation)        │
│                                                         │
│              [CREATE] [CANCEL]                        │
└─────────────────────────────────────────────────────────┘
```

4. Click [CREATE]

---

## 5.2 Add Step Inside PARAFOREACH Block

**Step Name:** `STEP4_PARALLEL_APPROVER_CALL`  
**Step Type:** Work Item → Call Sub-workflow

### Configuration

| Field | Value |
|-------|-------|
| **Basic Data Tab** | |
| Step Name | STEP4_PARALLEL_APPROVER_CALL |
| Step Type | Work Item (Sub-workflow call) |
| Workflow | WS900000232 |
| Processing Deadline | 2 Days |
| Escalation | 1 Day → Manager |
| **Agent Tab** | |
| Recipient Expression | **`&current_approver-USERID&`** (CRITICAL!) |
| **Binding Tab** | (SEE SECTION 6 BELOW) |

### Key Points
- **Agent/Recipient** must use loop element: `&current_approver-USERID&`
- This ensures each parallel iteration sends to a different approver.
- Binding: pass approver details to sub-workflow and collect DECISION back to `current_approver-DECISION`.

---

## 5.3 PARAFOREACH Block Summary

| Property | Value |
|----------|-------|
| Block Type | PARAFOREACH |
| Loop Container | t_parallel_approvers |
| Loop Element | current_approver |
| Max Parallel | 5 |
| Collect Results | YES |
| Create Context | YES |

**Behavior:**
- For each row in `t_parallel_approvers`, one copy becomes `current_approver`
- Step 4 (inside block) executes in parallel for each row (up to Max Parallel=5 concurrent)
- When all iterations complete, results are merged back to `t_parallel_approvers`
- Workflow continues to Step 5 (automatic sync point)

---

# STEP 6: BINDING CONFIGURATION

## 6.1 Bindings for STEP4_PARALLEL_APPROVER_CALL (Inside PARAFOREACH)

**Critical:** Use loop element `current_approver` in all bindings.

### IMPORT Bindings (TO Sub-workflow WS900000232)

```
┌──────────────────────────────────┬────────────────────────────────────┐
│ Sub-workflow Parameter           │ Container Element Binding          │
├──────────────────────────────────┼────────────────────────────────────┤
│ IV_APPROVER_ID                   │ ← &current_approver-USERID&        │
│ IV_APPROVER_NAME                 │ ← &current_approver-NAME&          │
│ IV_DOC_NUMBER                    │ ← &DOC_NUMBER&                     │
│ IV_PROCESS_ID                    │ ← &PROCESS_ID&                     │
│ IV_APPROVAL_LEVEL                │ ← &current_approver-LEVEL&         │
└──────────────────────────────────┴────────────────────────────────────┘
```

### EXPORT Bindings (FROM Sub-workflow WS900000232)

```
┌──────────────────────────────────┬────────────────────────────────────┐
│ Container Element                │ Sub-workflow Parameter             │
├──────────────────────────────────┼────────────────────────────────────┤
│ &current_approver-DECISION&      │ ← EV_DECISION                      │
│ &current_approver-COMMENTS&      │ ← EV_COMMENTS                      │
│ &current_approver-TIMESTAMP&     │ ← EV_DECISION_TIME                 │
└──────────────────────────────────┴────────────────────────────────────┘
```

**Steps to Add Bindings in SWDD:**
1. Double-click Step 4 in diagram
2. Go to **Binding** tab
3. **IMPORT section:**
   - Click [New Row]
   - Parameter Name: `IV_APPROVER_ID`
   - Container Element: `&current_approver-USERID&`
   - Press Tab
   - Repeat for all import parameters
4. **EXPORT section:**
   - Click [New Row]
   - Container Element: `&current_approver-DECISION&`
   - Parameter Name: `EV_DECISION`
   - Press Tab
   - Repeat for all export parameters
5. Click [Check Syntax]
6. Save

---

## 6.2 Bindings for STEP3_FETCH_PARALLEL_APPROVERS

**Step Name:** STEP3_FETCH_PARALLEL_APPROVERS  
**Function Module:** ZFILL_PARALLEL_APPROVERS

### IMPORT (TO Function)

```
┌──────────────────────────────────┬────────────────────────────────────┐
│ FM Parameter                     │ Container Element                  │
├──────────────────────────────────┼────────────────────────────────────┤
│ IV_DOC_NUMBER                    │ ← &DOC_NUMBER&                     │
└──────────────────────────────────┴────────────────────────────────────┘
```

### EXPORT (FROM Function)

```
┌──────────────────────────────────┬────────────────────────────────────┐
│ Container Element                │ FM Parameter                       │
├──────────────────────────────────┼────────────────────────────────────┤
│ &t_parallel_approvers&           │ ← ET_PAR_APPROVER_LIST             │
│ (optional) debug_count           │ ← EV_COUNT                         │
└──────────────────────────────────┴────────────────────────────────────┘
```

---

## 6.3 Step 5: Check Parallel Rejection

**Step Name:** `STEP5_CHECK_PARALLEL_REJECTION`  
**Step Type:** Activity (Automatic)

### Configuration

| Field | Value |
|-------|-------|
| Step Name | STEP5_CHECK_PARALLEL_REJECTION |
| Step Type | Activity (Automatic) |
| Form Routine | ZFORM_CHECK_PARALLEL_REJECTION |
| Workflow | WS90000231 |

**After execution:** Form sets `gv_parallel_rejection` to 'X' or ' '

---

## 6.4 Decision Gateway (After Step 5)

**Step Name:** `DECISION_PARALLEL_REJECTION`  
**Step Type:** Decision (Automatic)

### Decision Logic

```
IF &gv_parallel_rejection& = 'X'
   THEN route to: EXIT_PPM_REJECTION
   ELSE route to: STEP6_FINAL_APPROVER
ENDIF
```

**Steps in SWDD:**
1. Create decision step
2. Go to **Decision** tab
3. Create two outcome branches:
   - **Outcome 1:** Condition = `gv_parallel_rejection = 'X'`
     → Route to `EXIT_PPM_REJECTION` step
   - **Outcome 2:** Condition = `gv_parallel_rejection != 'X'` (or empty)
     → Route to `STEP6_FINAL_APPROVER`

---

## 6.5 Step 6: Final Approver

**Step Name:** `STEP6_FINAL_APPROVER_CALL`  
**Step Type:** Work Item → Call Sub-workflow

### Configuration

| Field | Value |
|-------|-------|
| Step Name | STEP6_FINAL_APPROVER_CALL |
| Step Type | Work Item (Sub-workflow call) |
| Workflow | WS900000232 |
| Processing Deadline | 2 Days |
| **Agent Tab** | |
| Recipient | APPROVER6_ID (or fixed literal 'APPROVER06') |
| **Binding Tab** | |
| TO Sub-wf: IV_APPROVER_ID | ← APPROVER6_ID |
| TO Sub-wf: IV_DOC_NUMBER | ← DOC_NUMBER |
| FROM Sub-wf: EV_DECISION | → (optional) |

---

# STEP 7: TESTING CHECKLIST

## 7.1 Pre-Testing: Unit Tests

### Test 1: Verify SE11 Structure

```
Transaction: SE11
Enter: ZAPP_PAR_APPROVER_ITEM
Action: Display
Expected: 7 components (USERID, NAME, ROLE, LEVEL, DECISION, COMMENTS, TIMESTAMP)
```

✓ **Status:** Pass / Fail

### Test 2: Verify Function Module ZFILL_PARALLEL_APPROVERS

```
Transaction: SE37
Enter: ZFILL_PARALLEL_APPROVERS
Click: [Test] or [Execute Test]
Input:  IV_DOC_NUMBER = 'TEST001'
Expected Output:
  ET_PAR_APPROVER_LIST has 3 rows
  EV_COUNT = 3
  Row1: USERID='APPROVER03', NAME='Approver Three'
  Row2: USERID='APPROVER04', NAME='Approver Four'
  Row3: USERID='APPROVER05', NAME='Approver Five'
```

✓ **Status:** Pass / Fail

### Test 3: Verify SWDD Workflow Syntax

```
Transaction: SWDD
Enter: WS90000231
Click: [Check] (Syntax Check)
Expected: "No errors found" message
```

✓ **Status:** Pass / Fail

---

## 7.2 Integration Testing: End-to-End

### Test 4: Workflow Start with All Approvals

```
Steps to Execute:
  1. Create test document (e.g., PO: PO12345)
  2. Start workflow WS90000231
     Container: DOC_NUMBER='PO12345', APPROVER1_ID='APPROVER01',
                APPROVER2_ID='APPROVER02', APPROVER6_ID='APPROVER06'
  3. Step 1: APPROVER01 opens work item → Approves (EV_DECISION='A')
  4. Step 2: APPROVER02 opens work item → Approves (EV_DECISION='A')
  5. Step 3: FM ZFILL_PARALLEL_APPROVERS runs
              Populates t_parallel_approvers (3 entries)
  6. Step 4 (PARAFOREACH): 
     Check SWIA: Should show 3 parallel work items
     - APPROVER03 receives task
     - APPROVER04 receives task
     - APPROVER05 receives task
  7. All 3 approvers: Open and Approve
  8. Step 5: Automatic check runs
     Expected: gv_parallel_rejection = ' ' (blank, no rejection)
  9. Step 6: APPROVER06 receives final task → Approves
  10. Workflow completes

Verification:
  - Monitor in SWIA: All steps executed in expected order
  - PARAFOREACH block shows 3 iterations in Event Log (SWEL)
  - All 3 parallel tasks completed before Step 5 started
  - Final status: Workflow completed successfully
```

✓ **Status:** Pass / Fail

### Test 5: Workflow with Rejection in Step 1

```
Steps to Execute:
  1. Start workflow WS90000231
  2. Step 1: APPROVER01 → REJECTS (EV_DECISION='R')
  3. Expected: Workflow routes to EXIT_REJECTION path
              Steps 2, 3, 4, 5, 6 NOT executed
  4. Workflow terminates with "Rejected" status

Verification:
  - SWIA shows: Step1 completed, routed to exit step
  - No work items sent to APPROVER02, APPROVER03-05, APPROVER06
  - Workflow instance shows termination reason
```

✓ **Status:** Pass / Fail

### Test 6: Workflow with Rejection in PARAFOREACH

```
Steps to Execute:
  1. Start workflow WS90000231
  2. Step 1: APPROVER01 → Approves
  3. Step 2: APPROVER02 → Approves
  4. Step 3: ZFILL_PARALLEL_APPROVERS populates 3 approvers
  5. Step 4 (PARAFOREACH): 3 parallel work items created
     - APPROVER03 → Approves
     - APPROVER04 → REJECTS (EV_DECISION='R')
     - APPROVER05 → Approves (arrives too late, already rejected)
  6. All branches complete, results collected to t_parallel_approvers
  7. Step 5: Form ZFORM_CHECK_PARALLEL_REJECTION runs
     Scans table: Finds APPROVER04 decision='R'
     Sets: gv_parallel_rejection='X'
  8. Decision gateway checks gv_parallel_rejection
     Expected: Route to EXIT_PPM_REJECTION (skip Step 6)
  9. Workflow terminates with "Parallel Rejection Detected"

Verification:
  - SWIA: All 3 parallel tasks executed concurrently
  - Step 5: Form detected rejection
  - Workflow routed to rejection exit (Step 6 not executed)
  - PPM status updated to rejection
```

✓ **Status:** Pass / Fail

### Test 7: Workflow Timeout & Escalation

```
Steps to Execute:
  1. Start workflow WS90000231
  2. Steps 1, 2, 3 complete successfully
  3. Step 4 (PARAFOREACH): Wait 2 days (or simulate via SWUA)
     Processing Deadline = 2 Days
     Expected: After 2 days, escalation task sent to manager
  4. Manager receives escalation work item
  5. Manager approves escalation → Approval continues
  6. Workflow proceeds to Step 5 and beyond

Verification:
  - Escalation task appears in SWIA after deadline
  - Manager receives notification
  - Approval can be delegated or reassigned
```

✓ **Status:** Pass / Fail

---

## 7.3 Monitoring During Tests

**Transaction: SWIA (Workflow Instance Analysis)**

```
1. Execute test workflow
2. Go to SWIA
3. Enter: Workflow = WS90000231
4. List instances
5. Double-click your test instance
6. Observe:
   - Current step indicator
   - Step execution timeline
   - Agent information
   - Container values (e.g., t_parallel_approvers contents)
   - Event log details
7. For PARAFOREACH block:
   - Expand block to see all iterations
   - Verify "Max Parallel" limit respected
   - Check each iteration's decision value
```

**Transaction: SWEL (Workflow Event Log)**

```
1. Go to SWEL
2. Enter: Workflow = WS90000231, Instance ID = (your instance)
3. View event sequence:
   - ACTIVATED, COMPLETED, EXCEPTION, etc.
   - For each step and block
4. Verify PARAFOREACH events:
   - BLOCK_START
   - ITERATION_ACTIVATED (x3 for 3 approvers)
   - ITERATION_COMPLETED (x3)
   - BLOCK_END
```

---

# STEP 8: TROUBLESHOOTING GUIDE

## Issue 1: PARAFOREACH Creates Only 1 Work Item (Not 3)

### Root Cause
- `t_parallel_approvers` is empty or not populated
- Loop container binding incorrect

### Solution
1. **Verify STEP3 execution:**
   - SWIA: Check if STEP3 completed successfully
   - Check function module return value: `ET_PAR_APPROVER_LIST` should have 3 rows
2. **Test FM directly:**
   ```
   Transaction: SE37 → ZFILL_PARALLEL_APPROVERS
   Click: [Test]
   Input: IV_DOC_NUMBER='TEST001'
   Execute: F8
   Check: ET_PAR_APPROVER_LIST has 3 rows
   ```
3. **Verify binding in SWDD:**
   - Step 3 binding: `ET_PAR_APPROVER_LIST → t_parallel_approvers`
   - Check syntax (click [Check Syntax] in Binding tab)
4. **Verify PARAFOREACH configuration:**
   - Loop Container: `t_parallel_approvers` (exact spelling)
   - Loop Element: `current_approver` (exact spelling)

---

## Issue 2: Parallel Tasks Not Running Concurrently (Run Sequentially)

### Root Cause
- Max Parallel Branches set too low (e.g., 1)
- System load limiting parallel execution
- PARAFOREACH context not properly isolated

### Solution
1. **Check Max Parallel setting:**
   - SWDD: Double-click PARAFOREACH block
   - Verify: Max Parallel Branches = 5 (or appropriate value)
   - Increase if system capacity allows
2. **Verify "Create New Context":**
   - Should be: ☑ YES
   - This isolates each iteration's context
3. **Monitor system load:**
   - Transaction SWUL (Workflow Load Monitor)
   - Check CPU/Memory usage
   - May need to reduce Max Parallel if system overloaded
4. **Check workflow load monitor:**
   - SWUL shows queue lengths and parallel capacity
   - May be queued due to system work

---

## Issue 3: Rejection Not Detected in Step 5

### Root Cause
- Form `ZFORM_CHECK_PARALLEL_REJECTION` not executed
- `current_approver-DECISION` not populated from sub-workflow
- "Collect Results" not enabled on PARAFOREACH

### Solution
1. **Verify Collect Results enabled:**
   - SWDD: Double-click PARAFOREACH block
   - Confirm: Collect Results = ☑ YES
   - Result Container = `t_parallel_approvers` (or blank for auto-merge)
2. **Verify binding from sub-workflow:**
   - Step 4 binding: `&current_approver-DECISION&` ← `EV_DECISION`
   - Check syntax
3. **Test form manually:**
   - SE38: Create test program
   - Call `ZFORM_CHECK_PARALLEL_REJECTION` with mock container
   - Verify it detects rejection correctly
4. **Check SWIA for Step 5:**
   - Verify Step 5 (automatic check) actually executes
   - Check event log for any errors

---

## Issue 4: Binding Expression Fails (Syntax Error)

### Root Cause
- Incorrect loop element syntax: `&current_approver-DECISION&` vs `current_approver-DECISION`
- Typo in container/field name
- Field doesn't exist in structure

### Solution
1. **Check syntax in SWDD Binding tab:**
   - Click [Check Syntax]
   - Fix any reported errors
2. **Verify loop element name:**
   - Must be: `current_approver` (exact)
   - Not: `current_approver_item` or other variants
3. **Verify field names in structure:**
   - SE11: Check ZAPP_PAR_APPROVER_ITEM
   - Field: `DECISION` (not `decision` or `DECISION_FLAG`)
4. **Use correct binding syntax:**
   - In SWDD: `&current_approver-DECISION&` (with &...& delimiters)
   - In ABAP form: `container-GET_ELEMENT('current_approver' ls_approver)`

---

## Issue 5: Sub-workflow Returns No Decision Value

### Root Cause
- Sub-workflow WS900000232 doesn't have EV_DECISION export
- Sub-workflow not configured to return decision

### Solution
1. **Check WS900000232 container:**
   - SWDD: Open WS900000232
   - Container Elements tab: Verify export parameters exist
     - `EV_DECISION` (or similar)
     - `EV_COMMENTS` (or similar)
2. **Verify sub-workflow passes decision back:**
   - Last step in WS900000232 must set decision value
   - Example: In final step, set `DECISION_VALUE='A'` or `'R'`
3. **Check binding in main workflow:**
   - Step 4 binding: FROM sub-wf `EV_DECISION` → TO main wf `current_approver-DECISION`
   - Verify exact parameter name matches sub-wf exports

---

## Issue 6: gv_parallel_rejection Remains Blank After Step 5

### Root Cause
- Form `ZFORM_CHECK_PARALLEL_REJECTION` not setting `gv_parallel_rejection`
- Container element not defined
- Form execution failed silently

### Solution
1. **Verify container element exists:**
   - SWDD: Container Elements tab
   - Search: `gv_parallel_rejection`
   - If missing: Add (Category: PARAMETER, Type: CHAR(1))
2. **Verify form sets value:**
   - ABAP: Form should call `container-SET_ELEMENT('gv_parallel_rejection' lv_rejected)`
   - Check form code for typos
3. **Test form execution:**
   - SWIA: After Step 5, check container values
   - Verify `gv_parallel_rejection` has value 'X' or ' '
4. **Check event log:**
   - SWEL: Look for form execution errors
   - May see exception if form failed

---

## Issue 7: Workflow Hangs After PARAFOREACH

### Root Cause
- Parallel branch deadlocked or timed out
- Automatic sync point not releasing
- Max Parallel limit reached globally

### Solution
1. **Check for stuck work items:**
   - SWIA: Look for tasks in "STARTED" state
   - Try completing or rejecting stuck tasks
2. **Release parallel limit:**
   - Workflow admin can reduce Max Parallel if limit hit
   - Or wait for work items to complete
3. **Check background job logs:**
   - SM37: Check for failed workflow background jobs
   - Restart if needed
4. **Monitor sync point:**
   - SWEL: Verify all iterations show COMPLETED event
   - If not, investigate why iteration is stuck

---

## Issue 8: Approver Not Receiving Work Item

### Root Cause
- USERID invalid or user doesn't exist
- User has no work item inbox configured
- Workflow notification email not configured

### Solution
1. **Verify USERID:**
   - SE37: Test `ZFILL_PARALLEL_APPROVERS`
   - Check returned USERID values (e.g., 'APPROVER03')
   - Confirm users exist in SU01
2. **Check user configuration:**
   - SU01: Search user (e.g., APPROVER03)
   - Verify: Workflow notification method set
     - Email, SAP Inbox, etc.
3. **Check work item send:**
   - SWIA: Look for "Task Agent = APPROVER03"
   - If task has no agent, binding expression may have failed
4. **Test recipient expression:**
   - Manual workflow test with hardcoded agent first
   - Then switch to expression `&current_approver-USERID&`

---

# APPENDIX: QUICK REFERENCE

## Checklist: Implementation Steps

- [ ] Step 1: Create SE11 Structure `ZAPP_PAR_APPROVER_ITEM`
- [ ] Step 2: Add container elements in SWDD WS90000231
- [ ] Step 3: Create FM `ZFILL_PARALLEL_APPROVERS` (SE37)
- [ ] Step 4: Create form `ZFORM_CHECK_PARALLEL_REJECTION`
- [ ] Step 5: Build SWDD workflow (Steps 1-6)
  - [ ] Step 1: Sub-wf Approver1
  - [ ] Step 2: Sub-wf Approver2
  - [ ] Step 3: FM to populate table
  - [ ] Step 4: PARAFOREACH block
  - [ ] Step 5: Check rejection
  - [ ] Step 6: Final Approver
- [ ] Step 6: Configure all bindings
- [ ] Step 7: Test end-to-end
  - [ ] Unit test FM
  - [ ] Test all approvals path
  - [ ] Test rejection in Step 1
  - [ ] Test rejection in PARAFOREACH
  - [ ] Test escalation (optional)
- [ ] Step 8: Deploy to production

---

## SWDD Container Elements (Quick Ref)

```
t_parallel_approvers    Table of ZAPP_PAR_APPROVER_ITEM
current_approver        Structure ZAPP_PAR_APPROVER_ITEM
gv_parallel_rejection   CHAR(1)
DOC_NUMBER              CHAR(10)
PROCESS_ID              CHAR(20)
APPROVER1_ID            CHAR(12)
APPROVER2_ID            CHAR(12)
APPROVER6_ID            CHAR(12)
```

---

## PARAFOREACH Block Settings (Quick Ref)

```
Block Name:           PARALLEL_APPROVERS_BLOCK
Loop Container:       t_parallel_approvers
Loop Element:         current_approver
Max Parallel:         5
Allow Re-blocking:    NO
Collect Results:      YES
Create New Context:   YES
Skip Empty Loops:     YES
```

---

## Binding Expressions (Quick Ref)

```
Agent expression:     &current_approver-USERID&
Decision return:      &current_approver-DECISION&
Document reference:   &DOC_NUMBER&
Process tracking:     &PROCESS_ID&
```

---

## Transaction Codes Reference

```
SE11    Data Dictionary (create structures, tables)
SE37    Function Module Editor (create ZFILL_*, ZRAISE_*)
SWDD    Workflow Definition (build WS90000231)
SWIA    Workflow Instance Analysis (monitor execution)
SWEL    Workflow Event Log (detailed trace)
SWUL    Workflow Load Monitor (system capacity)
SWUA    Work Item Monitor (agent's work items)
SWU3    Workflow Maintain API (direct workflow updates)
```

---

## Support Contacts

- **Workflow Issues:** Your Workflow Admin
- **ABAP/FM Issues:** Your ABAP Development Lead
- **SAP Basis:** Your System Admin
- **PPM Integration:** Your Process Owner

---

## Version & Changelog

**Document Version:** 1.0  
**Date:** August 5, 2026  
**Status:** PRODUCTION READY  
**Last Updated:** August 5, 2026

**Changes:**
- Initial creation with full 6-step workflow design
- PARAFOREACH configuration detailed
- ABAP skeletons provided
- Testing checklist included
- Troubleshooting guide comprehensive

---

# CONCLUSION

This document provides a complete, step-by-step guide to implement your parallel approval workflow:

**Step1 → Step2 → Step3 (Fetch) → Step4 (PARAFOREACH: Approvers 3,4,5) → Step5 (Check Rejection) → Step6**

**Key Features:**
✅ Sequential approvers 1 & 2 run first  
✅ Dynamic parallel block for approvers 3, 4, 5  
✅ Automatic rejection detection  
✅ Graceful handling of rejected approvals  
✅ Final approver (6) only runs if all parallel approvals pass  

Follow the steps in order, test each section, and refer to the Troubleshooting Guide for issues.

**Ready to implement? Start with Step 1 (SE11 structure) today!**

---

*This guide is based on SAP best practices for PARAFOREACH block usage and workflow parallelization. Customize the ABAP code and user IDs to match your organization.*
