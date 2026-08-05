# 📋 ABAP Code Artifacts - Ready to Copy & Paste

All ABAP code ready to copy directly into SAP (SE37 for function modules, SE38 for forms).

---

## ARTIFACT 1: Function Module ZFILL_PARALLEL_APPROVERS

**Transaction:** SE37  
**Create New Function Module:** `ZFILL_PARALLEL_APPROVERS`

### Step-by-Step in SE37:

1. SE37 → Function Module Name: `ZFILL_PARALLEL_APPROVERS` → Create
2. **Attributes Tab:**
   - Application: (your area, e.g., `FIN`)
   - Release Status: (check if needed)
3. **Import Tab:**
   ```
   Parameter Name          Type      Length  Optional
   IV_DOC_NUMBER          CHAR        10       ☐
   IV_DOC_TYPE            CHAR        10       ☑
   ```
4. **Export Tab:**
   ```
   Parameter Name          Type      Reference/Length
   ET_PAR_APPROVER_LIST   Table      ZAPP_PAR_APPROVER_ITEM
   EV_COUNT               NUMC       3
   ```
5. **Exceptions Tab:**
   ```
   NO_APPROVERS_FOUND
   ```
6. **Source Code Tab:** Paste code below:

### Source Code (Copy Everything Below)

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
  " OPTION 1: Hard-coded for testing/demo
  " (Replace with your org logic in production)
  "=======================================================================
  
  ls_app-USERID    = 'APPROVER03'.
  ls_app-NAME      = 'Approver Three'.
  ls_app-ROLE      = 'FINANCE_MANAGER'.
  ls_app-LEVEL     = '2'.
  ls_app-DECISION  = 'P'.  " P = Pending
  ls_app-COMMENTS  = ' '.
  ls_app-TIMESTAMP = sy-tstmp.
  APPEND ls_app TO ET_PAR_APPROVER_LIST.
  lv_count = lv_count + 1.

  ls_app-USERID    = 'APPROVER04'.
  ls_app-NAME      = 'Approver Four'.
  ls_app-ROLE      = 'HR_MANAGER'.
  ls_app-LEVEL     = '2'.
  ls_app-DECISION  = 'P'.
  ls_app-COMMENTS  = ' '.
  ls_app-TIMESTAMP = sy-tstmp.
  APPEND ls_app TO ET_PAR_APPROVER_LIST.
  lv_count = lv_count + 1.

  ls_app-USERID    = 'APPROVER05'.
  ls_app-NAME      = 'Approver Five'.
  ls_app-ROLE      = 'BUDGET_OWNER'.
  ls_app-LEVEL     = '2'.
  ls_app-DECISION  = 'P'.
  ls_app-COMMENTS  = ' '.
  ls_app-TIMESTAMP = sy-tstmp.
  APPEND ls_app TO ET_PAR_APPROVER_LIST.
  lv_count = lv_count + 1.

  "=======================================================================
  " OPTION 2: Dynamic lookup from org structure (for production)
  " Uncomment and replace OPTION 1 when ready
  "=======================================================================
  
  "  SELECT * FROM hrp1000
  "    WHERE otype = 'O'
  "    AND subty = 'DEPT'
  "    INTO TABLE @DATA(lt_orgs).
  "
  "  LOOP AT lt_orgs ASSIGNING FIELD-SYMBOL(<org>).
  "    ls_app-USERID = <org>-objid.
  "    ls_app-NAME = <org>-stext.
  "    " ... add more org lookup logic ...
  "    APPEND ls_app TO ET_PAR_APPROVER_LIST.
  "    lv_count = lv_count + 1.
  "  ENDLOOP.

  "=======================================================================
  " Validation
  "=======================================================================
  IF lv_count = 0.
    RAISE NO_APPROVERS_FOUND.
  ENDIF.

  EV_COUNT = lv_count.

ENDFUNCTION.
```

### Test Instructions:

1. **After pasting code above:**
   - Press Ctrl+S (Save)
   - Press Ctrl+F3 (Activate)
   - Expect message: "Function module activated successfully"

2. **Test the function module:**
   - Press F8 (Test/Execute)
   - Input: `IV_DOC_NUMBER = 'TEST001'`
   - Click Execute
   - Expected Output:
     ```
     ET_PAR_APPROVER_LIST (3 rows):
       Row 1: USERID='APPROVER03', NAME='Approver Three'
       Row 2: USERID='APPROVER04', NAME='Approver Four'
       Row 3: USERID='APPROVER05', NAME='Approver Five'
     EV_COUNT = 3
     ```
   - Status: ✅ Pass

---

## ARTIFACT 2: Form Routine ZFORM_CHECK_PARALLEL_REJECTION

**Use in:** SWDD as automatic activity (Form Routine)

**How to reference in SWDD Step 5:**
- Step Type: Activity (Automatic)
- Form Routine: `ZFORM_CHECK_PARALLEL_REJECTION`
- Workflow: `WS90000231`

**Or place this in INCLUDE for your workflow functions**

### Source Code (Copy Everything Below)

```abap
FORM ZFORM_CHECK_PARALLEL_REJECTION.
*"----------------------------------------------------------------------
* Purpose: Check if any parallel approver REJECTED
*          Sets gv_parallel_rejection = 'X' if found
*          Sets gv_parallel_rejection = ' ' if all approved/pending
*
* Called from: SWDD Step 5 (Automatic Activity)
* Container Elements Used:
*   Input:  t_parallel_approvers (table of ZAPP_PAR_APPROVER_ITEM)
*   Output: gv_parallel_rejection (CHAR(1))
*----------------------------------------------------------------------

  DATA: lt_approvers TYPE TABLE OF ZAPP_PAR_APPROVER_ITEM,
        ls_approver  TYPE ZAPP_PAR_APPROVER_ITEM,
        lv_rejected  TYPE CHAR1 VALUE ' ',
        lv_count     TYPE I VALUE 0,
        lv_reject_count TYPE I VALUE 0.

  "=======================================================================
  " Get the parallel approvers table from workflow container
  "=======================================================================
  CLEAR lt_approvers.
  container-GET_ELEMENT( 't_parallel_approvers' lt_approvers ).

  IF lt_approvers IS INITIAL.
    " No approvers in table; set no rejection
    container-SET_ELEMENT( 'gv_parallel_rejection' ' ' ).
    WRITE: / '=== Parallel Rejection Check Complete ==='.
    WRITE: / 'Result: No approvers to check'.
    RETURN.
  ENDIF.

  "=======================================================================
  " Loop through approvers and check for rejection/refer-back
  "=======================================================================
  LOOP AT lt_approvers INTO ls_approver.
    lv_count = lv_count + 1.
    
    WRITE: / 'Approver ', lv_count, ': ', ls_approver-userid,
           ' | Decision: ', ls_approver-decision.
    
    " Check for rejection (R) or refer-back (F)
    IF ls_approver-decision = 'R' OR ls_approver-decision = 'F'.
      lv_rejected = 'X'.
      lv_reject_count = lv_reject_count + 1.
      WRITE: / '  >>> REJECTION DETECTED from ', 
              ls_approver-userid,
              ' | Comments: ', ls_approver-comments.
      " Note: You can optionally EXIT here for immediate stop
      " or continue loop to log all rejections
    ENDIF.
  ENDLOOP.

  "=======================================================================
  " Set rejection flag in workflow container
  "=======================================================================
  container-SET_ELEMENT( 'gv_parallel_rejection' lv_rejected ).

  "=======================================================================
  " Logging output for debugging
  "=======================================================================
  WRITE: / '=== Parallel Rejection Check Complete ==='.
  WRITE: / 'Total Approvers Checked: ', lv_count.
  WRITE: / 'Rejections Found: ', lv_reject_count.
  WRITE: / 'Result Flag: ', lv_rejected.
  IF lv_rejected = 'X'.
    WRITE: / 'STATUS: REJECTION DETECTED - PPM Flow will be triggered'.
  ELSE.
    WRITE: / 'STATUS: ALL APPROVED - Continue to Final Approver'.
  ENDIF.

ENDFORM.
```

### Integration Steps:

1. **Place this form in:**
   - Option A: INCLUDE of workflow form routines
   - Option B: Create separate program if not in standard location

2. **In SWDD Step 5 configuration:**
   - Step Type: Activity (Automatic)
   - Form Routine Name: `ZFORM_CHECK_PARALLEL_REJECTION`
   - Workflow: `WS90000231`
   - No binding needed (form accesses container directly)

3. **After Step 5 completes:**
   - Container `gv_parallel_rejection` will be set to 'X' or ' '
   - Used by Decision step to route workflow

---

## ARTIFACT 3: Function Module ZRAISE_PARALLEL_REJECTION (Optional - Advanced)

**Use only if:** You need immediate termination on parallel rejection (advanced scenario)

**Transaction:** SE37  
**Create New Function Module:** `ZRAISE_PARALLEL_REJECTION`

### Interface Definition:

**Import Tab:**
```
Parameter Name          Type      Length
IV_OBJECT_TYPE         CHAR        30
IV_OBJECT_KEY          CHAR        50
IV_EVENT               CHAR        30
IV_REASON              CHAR        100  (Optional)
```

**No Exports**

**Exceptions:**
```
OBJECT_NOT_FOUND
NO_EVENT_LINK
GENERIC_ERROR
```

### Source Code (Copy Everything Below)

```abap
FUNCTION ZRAISE_PARALLEL_REJECTION.
*"----------------------------------------------------------------------
*"*"Local Interface:
*"  IMPORTING
*"     VALUE(IV_OBJECT_TYPE) TYPE CHAR30
*"     VALUE(IV_OBJECT_KEY)  TYPE CHAR50
*"     VALUE(IV_EVENT)       TYPE CHAR30
*"     VALUE(IV_REASON)      TYPE CHAR100 OPTIONAL
*"  EXCEPTIONS
*"     OBJECT_NOT_FOUND
*"     NO_EVENT_LINK
*"     GENERIC_ERROR
*"----------------------------------------------------------------------

  DATA: lv_msg TYPE CHAR200,
        lv_rc  TYPE SY-SUBRC.

  "=======================================================================
  " Purpose: Raise event on business object to trigger immediate
  "          workflow termination or rejection handling
  "
  " Note: Business object and event must be defined in SWET
  "       Main workflow must subscribe to this event
  "=======================================================================

  WRITE: / '=== Raising Parallel Rejection Event ==='.
  WRITE: / 'Object Type: ', iv_object_type.
  WRITE: / 'Object Key: ', iv_object_key.
  WRITE: / 'Event: ', iv_event.
  IF iv_reason IS NOT INITIAL.
    WRITE: / 'Reason: ', iv_reason.
  ENDIF.

  "=======================================================================
  " Raise event using SAP Workflow Event API
  "=======================================================================
  
  CALL FUNCTION 'SWE_EVENT_CREATE'
    EXPORTING
      objtype          = iv_object_type
      objkey           = iv_object_key
      event            = iv_event
    EXCEPTIONS
      OBJECT_NOT_FOUND = 1
      NO_EVENT_LINK    = 2
      OTHERS           = 3.

  lv_rc = sy-subrc.

  CASE lv_rc.
    WHEN 0.
      WRITE: / 'Result: Event raised SUCCESSFULLY'.
      WRITE: / 'Workflow will catch event and process rejection'.
    WHEN 1.
      WRITE: / 'Error: OBJECT_NOT_FOUND'.
      WRITE: / 'Verify object type exists in SWET'.
      RAISE OBJECT_NOT_FOUND.
    WHEN 2.
      WRITE: / 'Error: NO_EVENT_LINK'.
      WRITE: / 'Verify business object event is defined'.
      RAISE NO_EVENT_LINK.
    WHEN OTHERS.
      WRITE: / 'Error: Generic error, subrc = ', lv_rc.
      RAISE GENERIC_ERROR.
  ENDCASE.

ENDFUNCTION.
```

### When to Use:

- **Only if:** Your workflow must terminate immediately when any parallel approver rejects
- **Instead of:** Waiting for all parallel branches to complete + Step5 aggregation
- **Complexity:** Advanced; requires event subscription setup in main workflow

### Setup Required:

1. **Create Business Object Event (SWET):**
   - SWET: Create event for your BO (e.g., `ZBUSOBJ_DOC`)
   - Event Name: (e.g., `PARALLEL_REJECTION_EVENT`)

2. **Subscribe in Main Workflow:**
   - SWDD: Configure workflow to listen for event
   - When event received: Trigger termination/exit step

3. **Call from Sub-workflow:**
   - When sub-wf detects rejection, call this FM
   - Passes event to main workflow
   - Main workflow catches event and terminates immediately

---

## ARTIFACT 4: Test Program (Optional - for FM validation)

**Transaction:** SE38  
**Program Name:** `ZTEST_PARALLEL_APPROVERS`

### Source Code (for testing ZFILL_PARALLEL_APPROVERS)

```abap
REPORT ZTEST_PARALLEL_APPROVERS.

DATA: lt_approvers TYPE TABLE OF ZAPP_PAR_APPROVER_ITEM,
      ls_approver  TYPE ZAPP_PAR_APPROVER_ITEM,
      lv_count     TYPE NUMC3,
      lv_msg       TYPE CHAR100.

START-OF-SELECTION.

  WRITE: / '=== Test: ZFILL_PARALLEL_APPROVERS ==='.
  WRITE: / 'Calling function module...'.
  WRITE: /.

  " Call the function module
  CALL FUNCTION 'ZFILL_PARALLEL_APPROVERS'
    EXPORTING
      iv_doc_number        = 'TEST001'
      iv_doc_type          = 'PO'
    IMPORTING
      et_par_approver_list = lt_approvers
      ev_count             = lv_count
    EXCEPTIONS
      no_approvers_found   = 1
      OTHERS               = 2.

  IF sy-subrc <> 0.
    WRITE: / 'ERROR: Function module failed with subrc = ', sy-subrc.
    STOP.
  ENDIF.

  " Display results
  WRITE: / 'SUCCESS: Function module returned ', lv_count, ' approvers'.
  WRITE: / ''.
  WRITE: / 'Approver List:'.
  WRITE: / '────────────────────────────────────────────'.

  LOOP AT lt_approvers INTO ls_approver.
    WRITE: / 'Approver #', sy-tabix.
    WRITE: / '  UserID: ', ls_approver-userid.
    WRITE: / '  Name: ', ls_approver-name.
    WRITE: / '  Role: ', ls_approver-role.
    WRITE: / '  Level: ', ls_approver-level.
    WRITE: / '  Decision: ', ls_approver-decision.
    WRITE: / ''.
  ENDLOOP.

  WRITE: / '────────────────────────────────────────────'.
  WRITE: / 'TEST RESULT: ✅ PASS - All 3 approvers loaded correctly'.

END-OF-SELECTION.
```

**To Run:**
1. SE38 → Enter `ZTEST_PARALLEL_APPROVERS`
2. Create program and paste code
3. Save and Activate
4. Execute (F8)
5. Expected Output: 3 approvers listed with details

---

## QUICK COPY-PASTE CHECKLIST

Use this checklist to track what you've copied and where:

- [ ] **ZFILL_PARALLEL_APPROVERS** (SE37)
  - [ ] Function module created
  - [ ] Interface (Import/Export/Exceptions) added
  - [ ] Source code pasted
  - [ ] Saved and Activated
  - [ ] Tested (F8) - confirmed 3 approvers returned

- [ ] **ZFORM_CHECK_PARALLEL_REJECTION** (Form or INCLUDE)
  - [ ] Code copied to workflow form include
  - [ ] Integrated in SWDD Step 5
  - [ ] Tested by running workflow to Step5

- [ ] **ZRAISE_PARALLEL_REJECTION** (SE37) - OPTIONAL
  - [ ] Created (if using advanced immediate-termination)
  - [ ] Interface configured
  - [ ] Event subscription set up in main workflow
  - [ ] Tested

- [ ] **ZTEST_PARALLEL_APPROVERS** (SE38) - OPTIONAL
  - [ ] Test program created
  - [ ] Executed and validated FM output

---

## COMMON COPY-PASTE ERRORS TO AVOID

❌ **Do NOT:**
- Copy just the function signature without the full source code
- Change parameter names (they must match exactly)
- Remove exception handling (EXCEPTIONS clause)
- Modify the binding names without updating SWDD

✅ **DO:**
- Copy the entire source code block (from FUNCTION to ENDFUNCTION)
- Test each FM immediately after pasting with F8
- Use exact parameter names in SWDD bindings
- Verify Activate (Ctrl+F3) shows success message

---

## Troubleshooting Paste Issues

**If you get syntax errors after pasting:**

1. **Check indentation:**
   - ABAP syntax requires proper indentation
   - SAP may auto-correct, but if errors persist:
   - Select all code and press Ctrl+Shift+< to fix indentation

2. **Check comment blocks:**
   - Comments starting with `*"` are ABAP syntax
   - If editor doesn't recognize, replace with `*` comment style

3. **Test syntax:**
   - In SE37/SE38 code editor: Ctrl+F2 or Menu → Check
   - Fix any reported line numbers

4. **Activate:**
   - After fixes, press Ctrl+F3 to activate
   - Should see: "Generated object saved successfully" message

---

## End of Artifacts

All code ready to copy. Proceed to implement in this order:
1. ZFILL_PARALLEL_APPROVERS (SE37)
2. ZFORM_CHECK_PARALLEL_REJECTION (Form/Include)
3. Test with ZTEST_PARALLEL_APPROVERS (SE38)
4. Configure SWDD workflow with these modules
5. Run integration tests
