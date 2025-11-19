# WIPTRACK OPERATIONS MANUAL

**TAIMEX Industrial - Work In Progress Tracking System**
**Version:** 1.0
**Last Updated:** November 19, 2025
**Module:** WipTrack

---

## Table of Contents

1. [Overview](#overview)
2. [System Architecture](#system-architecture)
3. [Core Files Reference](#core-files-reference)
4. [Database Tables](#database-tables)
5. [Main Workflows](#main-workflows)
6. [Process Descriptions](#process-descriptions)
7. [State Management](#state-management)
8. [Session Variables](#session-variables)
9. [Common Patterns](#common-patterns)
10. [Error Handling](#error-handling)
11. [Best Practices](#best-practices)
12. [Troubleshooting](#troubleshooting)

---

## Overview

### Purpose

The WipTrack module provides comprehensive tracking of Work In Progress (WIP) for manufacturing operations. It allows operators to:

- Clock in/out of work orders using badge scanning
- Track production progress in real-time
- Record scrap and quality issues
- Monitor work order status and location
- Generate activity logs and reports

### Key Features

- **Badge-based Authentication**: Operators authenticate using physical badges
- **Work Order Tracking**: Real-time tracking of WIP status
- **Location Management**: Track WIP location through production floor
- **Scrap Recording**: Document waste and quality issues
- **Status Management**: Start, pause, resume, and complete work orders
- **Audit Trail**: Complete log of all WIP activities

---

## System Architecture

### Flow Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                      WIPTRACK SYSTEM FLOW                        │
└─────────────────────────────────────────────────────────────────┘

   ┌────────────────┐
   │  WIPMENU_EDIT  │  WIP Menu
   │   (Main Menu)  │
   └───────┬────────┘
           │
           ▼
   ┌────────────────┐
   │ WIPTRACK_EDIT  │  Step 1: Scan Badge
   │                │
   └───────┬────────┘
           │
           ▼
   ┌────────────────┐
   │WIPTRACK2_EDIT  │  Step 2: Scan Work Order
   │                │
   └───────┬────────┘
           │
           ├──────────────┬──────────────┐
           │              │              │
      [Status='']    [Status='P']   [Status='O']
      [Status='P']   (Paused)       (Open/Active)
           │              │              │
           ▼              │              ▼
   ┌────────────────┐    │      ┌────────────────┐
   │WIPTRACK3_EDIT  │    │      │WIPTRACK4_EDIT  │
   │ Start New WO   │    │      │ Resume/Manage  │
   └───────┬────────┘    │      └───────┬────────┘
           │              │              │
           └──────────────┴──────────────┘
                          │
                          ▼
                 ┌────────────────┐
                 │WIPTRACKING_EDIT│  Main Tracking Interface
                 │                │
                 └───────┬────────┘
                          │
        ┌─────────────────┼─────────────────┐
        │                 │                 │
        ▼                 ▼                 ▼
┌────────────────┐ ┌────────────────┐ ┌────────────────┐
│WIPTRACKSTART2  │ │WIPTRACKSUSPEND2│ │WIPTRACKSCRAP2  │
│ Start Work     │ │ Pause Work     │ │ Record Scrap   │
└────────────────┘ └────────────────┘ └────────────────┘

        │                 │                 │
        └─────────────────┼─────────────────┘
                          │
                          ▼
                 ┌────────────────┐
                 │WIPTRACKCOMPLETE│  Complete Work
                 │     _EDIT      │
                 └───────┬────────┘
                          │
                          ▼
                 ┌────────────────┐
                 │WIPTRACKFINAL   │  Finalize
                 │     _EDIT      │
                 └────────────────┘
```

### Supporting Functions

```
┌────────────────┐         ┌────────────────┐
│WIPTRACKLOG     │         │WIPTRACKREVIEW  │
│ View WO Log    │         │ Review Status  │
└────────────────┘         └────────────────┘

┌────────────────┐         ┌────────────────┐
│WIPTRACKVAL     │         │WIPTRACKAUTH    │
│ Validation     │         │ Authorization  │
└────────────────┘         └────────────────┘
```

---

## Core Files Reference

### Main Interface Files (22 files)

| File Name | Purpose | Entry Point | Flow Position |
|-----------|---------|-------------|---------------|
| **WIPMENU_EDIT.HTM** | WIP main menu | WIP Module | Entry |
| **WIPTRACK_EDIT.HTM** | Badge scan entry | Menu > WIP Tracking | Step 1 |
| **WIPTRACK2_EDIT.HTM** | Work order scan | After badge scan | Step 2 |
| **WIPTRACK3_EDIT.HTM** | Start new WO | WO Status = '' or 'P' | Step 3a |
| **WIPTRACK4_EDIT.HTM** | Resume active WO | WO Status = 'O' | Step 3b |
| **WIPTRACKING_EDIT.HTM** | Main tracking interface | After WO selection | Main |
| **WIPTRACKING_EDIT_BACKUP.HTM** | Backup of main interface | N/A | Backup |
| **WIPTRACKSTART2.HTM** | Start work session | WIPTRACKING > Start | Action |
| **WIPTRACKSUSPEND2_EDIT.HTM** | Suspend/pause work | WIPTRACKING > Pause | Action |
| **WIPTRACKSCRAP2_EDIT.HTM** | Record scrap | WIPTRACKING > Scrap | Action |
| **WIPTRACKCOMPLETE_EDIT.HTM** | Complete work order | WIPTRACKING > Complete | Action |
| **WIPTRACKFINAL_EDIT.HTM** | Finalize completion | After complete | Final |
| **WIPTRACKLOG_EDIT.HTM** | View WO activity log | WIPTRACKING > View Log | View |
| **WIPTRACKREVIEW_EDIT.HTM** | Review WO status | WIPTRACKING > Review | View |
| **WIPTRACKVAL_EDIT.HTM** | Data validation | Various points | Utility |
| **WIPTRACKAUTH_EDIT.HTM** | Authorization check | Protected actions | Utility |

### Template and Tool Files

| File Name | Purpose | Type |
|-----------|---------|------|
| **WIP_TEMPLATE_MASTER.HTM** | Master template for WIP pages | Template |
| **WIP_GENERATOR_TOOL.HTM** | Page generation tool | Tool |
| **WIP_GENERATOR.HTM** | Generator interface | Tool |

### XML/Service Files

| File Name | Purpose | Integration Point |
|-----------|---------|-------------------|
| **CDAWIPTRACK.HTM** | WIP tracking web service | External API |
| **CDA_WIPTRACKLOG.HTM** | WIP log web service | External API |
| **CDA_WIPTRACKSCRAP2.HTM** | Scrap recording service | External API |
| **CDA_WIPTRACK4.HTM** | WIP status service | External API (misc folder) |

---

## Database Tables

### Core Tables

#### OPERS (Operators)
- **Purpose**: Operator/user information
- **Key Field**: Operator ID (INITIALS)
- **Fields Used**:
  - `<1>` - Operator Name
  - `<9>` - Operator Badge ID
  - `<18>` - Menu Access Permissions
- **Access**: READ for authentication and permissions

#### BADGES
- **Purpose**: Badge to operator mapping
- **Key Field**: Badge ID
- **Fields Used**:
  - `<4>` - Operator Initials/ID
- **Access**: READ for badge scanning

#### WO (Work Orders)
- **Purpose**: Work order master records
- **Key Field**: Work Order Number (WOID)
- **Fields Used**:
  - `<32>` - Assembly Number (ASSY)
  - `<90>` - WIP Location Code
  - `<90,1>` - Current WIP Location
  - `<91,1>` - WIP Balance/Quantity
- **Access**: READ for WO information, WRITE for status updates

#### WIPSTATUS
- **Purpose**: Current WIP status tracking
- **Key Field**: Work Order Number
- **Fields Used**:
  - `<1>` - Status Code:
    - `''` (Empty) - Not started
    - `'P'` - Paused/Suspended
    - `'O'` - Open/Active
    - `'C'` - Completed
- **Access**: READ/WRITE for status management

#### WIPLOC (WIP Locations)
- **Purpose**: Location master data
- **Key Field**: Location Code
- **Fields Used**:
  - `<1>` - Location Description (WIPD)
- **Access**: READ for location information

---

## Main Workflows

### 1. Start New Work Order

**Entry Point**: WIPTRACK_EDIT.HTM

```
┌─────────────────────────────────────────────────────────────┐
│ WORKFLOW: Start New Work Order                              │
└─────────────────────────────────────────────────────────────┘

Step 1: WIPTRACK_EDIT.HTM (Scan Badge)
├─ Input: BADGEID (scanned or typed)
├─ Validation:
│  ├─ Check OPERS table for operator ID
│  ├─ Check BADGES table for badge ID
│  └─ Verify menu access permissions
├─ Processing:
│  ├─ Clear session cookies: SONO, QTNO
│  ├─ Create BADGEID session: "INIT|BADGEID" format
│  └─ Set OPERNAME from OPERS<1>
└─ Next: WIPTRACK2_EDIT.HTM

Step 2: WIPTRACK2_EDIT.HTM (Scan Work Order)
├─ Input: WOID (Work Order Number)
├─ Session: BADGEID, OPERNAME
├─ Validation:
│  ├─ Verify WOID exists in WO table
│  ├─ Check WO activation status (WO<90>)
│  └─ Read WIPSTATUS for current state
├─ Processing:
│  ├─ Parse operator initials from BADGEID
│  ├─ Validate WO is activated
│  └─ Check WIPSTATUS<1> value
├─ Routing:
│  ├─ If WIPSTATUS = '' or 'P' → WIPTRACK3_EDIT.HTM (Start)
│  └─ If WIPSTATUS = 'O' → WIPTRACK4_EDIT.HTM (Resume)
└─ Error: "INVALID WORK ORDER" or "WO NOT ACTIVATED"

Step 3a: WIPTRACK3_EDIT.HTM (Start New/Paused WO)
├─ Display: WO information, operator details
├─ Form Fields:
│  ├─ Location selection
│  ├─ Quantity to start
│  └─ Notes/comments
├─ Actions:
│  ├─ START WORK - Begin tracking
│  └─ BACK - Return to WO scan
└─ Next: WIPTRACKSTART2.HTM (on submit)

Step 3b: WIPTRACK4_EDIT.HTM (Resume Active WO)
├─ Display: Current WO status
├─ Form Fields:
│  ├─ Current location
│  ├─ Current quantity
│  └─ Time elapsed
├─ Actions:
│  ├─ CONTINUE WORK - Resume tracking
│  ├─ PAUSE - Suspend work
│  └─ VIEW LOG - Check history
└─ Next: WIPTRACKING_EDIT.HTM

Step 4: WIPTRACKSTART2.HTM (Initialize Work)
├─ Processing:
│  ├─ Update WIPSTATUS to 'O' (Open)
│  ├─ Record start time
│  ├─ Set initial location
│  └─ Log activity
└─ Next: WIPTRACKING_EDIT.HTM

Step 5: WIPTRACKING_EDIT.HTM (Main Interface)
├─ Display:
│  ├─ WO summary information
│  ├─ Current progress
│  ├─ Active operator
│  └─ Location and quantity
├─ Available Actions:
│  ├─ Record Production
│  ├─ Record Scrap
│  ├─ Pause Work
│  ├─ Complete WO
│  └─ View Log
└─ Workflow continues based on action selected
```

### 2. Record Scrap

**Entry Point**: WIPTRACKING_EDIT.HTM > Scrap Button

```
┌─────────────────────────────────────────────────────────────┐
│ WORKFLOW: Record Scrap                                       │
└─────────────────────────────────────────────────────────────┘

Step 1: Navigate to Scrap Entry
├─ From: WIPTRACKING_EDIT.HTM
├─ Click: "Record Scrap" button
└─ Redirect: WIPTRACKSCRAP2_EDIT.HTM

Step 2: WIPTRACKSCRAP2_EDIT.HTM (Scrap Entry Form)
├─ Display:
│  ├─ WO Number and Assembly
│  ├─ Current operator
│  └─ Scrap entry form
├─ Form Fields:
│  ├─ Scrap Quantity (required)
│  ├─ Scrap Reason Code (dropdown)
│  ├─ Scrap Location
│  ├─ Notes/Comments
│  └─ Date/Time (auto-filled)
├─ Validation:
│  ├─ Quantity > 0
│  ├─ Reason code selected
│  └─ Quantity ≤ WIP Balance
└─ Actions:
   ├─ SUBMIT - Record scrap
   ├─ CANCEL - Return without saving
   └─ CLEAR - Reset form

Step 3: Process Scrap Record
├─ Processing:
│  ├─ Reduce WIP balance
│  ├─ Create scrap transaction
│  ├─ Update WO scrap total
│  ├─ Log activity with reason
│  └─ Trigger alerts if threshold exceeded
├─ Database Updates:
│  ├─ WO<91,1> -= ScrapQty (reduce balance)
│  ├─ Create SCRAP record
│  └─ Update WIPSTATUS log
└─ Result:
   ├─ Success: Return to WIPTRACKING_EDIT.HTM
   ├─ Show confirmation message
   └─ Error: Display error and stay on form
```

### 3. Pause/Suspend Work

**Entry Point**: WIPTRACKING_EDIT.HTM > Pause Button

```
┌─────────────────────────────────────────────────────────────┐
│ WORKFLOW: Pause/Suspend Work                                 │
└─────────────────────────────────────────────────────────────┘

Step 1: WIPTRACKSUSPEND2_EDIT.HTM (Suspend Form)
├─ Display:
│  ├─ WO summary
│  ├─ Current progress
│  └─ Suspend reason form
├─ Form Fields:
│  ├─ Suspend Reason (required)
│  ├─ Expected Resume Time
│  ├─ Notes
│  └─ Current operator
├─ Validation:
│  └─ Suspend reason must be provided
└─ Actions:
   ├─ CONFIRM SUSPEND
   └─ CANCEL

Step 2: Process Suspension
├─ Processing:
│  ├─ Update WIPSTATUS<1> = 'P' (Paused)
│  ├─ Record pause time
│  ├─ Calculate elapsed time
│  ├─ Save current state
│  └─ Log suspend activity
├─ Database Updates:
│  ├─ WIPSTATUS<1> = 'P'
│  ├─ Add suspend record to log
│  └─ Update WO timestamp
└─ Result:
   ├─ Return to WIPTRACK_EDIT.HTM
   └─ Display "Work Paused" message
```

### 4. Complete Work Order

**Entry Point**: WIPTRACKING_EDIT.HTM > Complete Button

```
┌─────────────────────────────────────────────────────────────┐
│ WORKFLOW: Complete Work Order                                │
└─────────────────────────────────────────────────────────────┘

Step 1: WIPTRACKCOMPLETE_EDIT.HTM (Complete Form)
├─ Display:
│  ├─ WO summary
│  ├─ Total production
│  ├─ Total scrap
│  └─ Completion form
├─ Form Fields:
│  ├─ Final quantity completed
│  ├─ Final location
│  ├─ Quality check (pass/fail)
│  ├─ Completion notes
│  └─ Supervisor approval (if required)
├─ Validation:
│  ├─ All quantities reconciled
│  ├─ Quality check completed
│  └─ Required approvals obtained
└─ Actions:
   ├─ COMPLETE WO
   ├─ SAVE DRAFT
   └─ CANCEL

Step 2: WIPTRACKFINAL_EDIT.HTM (Finalization)
├─ Processing:
│  ├─ Final quantity validation
│  ├─ Close all open transactions
│  ├─ Generate completion report
│  └─ Update WO master status
├─ Database Updates:
│  ├─ WIPSTATUS<1> = 'C' (Completed)
│  ├─ WO final quantities
│  ├─ Close WIP location
│  └─ Generate audit record
├─ Reporting:
│  ├─ Print completion document
│  ├─ Email notifications
│  └─ Update dashboards
└─ Result:
   ├─ Display completion summary
   ├─ Option to print report
   └─ Return to WIPMENU_EDIT.HTM
```

### 5. View Work Order Log

**Entry Point**: WIPTRACKING_EDIT.HTM > View Log

```
┌─────────────────────────────────────────────────────────────┐
│ WORKFLOW: View Work Order Log                                │
└─────────────────────────────────────────────────────────────┘

Step 1: WIPTRACKLOG_EDIT.HTM (Log Display)
├─ Display:
│  ├─ WO header information
│  ├─ Assembly: WOREC<32>
│  ├─ WIP Balance: WOREC<91,1>
│  └─ Current Location: WOREC<90,1> + WIPD
├─ Log Content:
│  ├─ Call BUILDWIPSTATUS(WOID, RESULT, PROD.IDS)
│  ├─ Convert to CRLF format
│  └─ Display in scrollable text area
├─ Log Entries Include:
│  ├─ Timestamp
│  ├─ Operator
│  ├─ Action (Start/Pause/Resume/Complete)
│  ├─ Quantity changes
│  ├─ Location changes
│  ├─ Scrap records
│  └─ Status changes
└─ Actions:
   ├─ REFRESH - Reload log
   ├─ EXPORT - Download log file
   ├─ PRINT - Print log
   └─ BACK - Return to main interface
```

---

## Process Descriptions

### Badge Scanning Process

**File**: WIPTRACK_EDIT.HTM

**Purpose**: Authenticate operator via badge scan

**Logic**:
```basic
1. Scan/Enter BADGEID
2. Trim and convert to uppercase
3. Attempt to read from OPERS table using BADGEID
4. If not found, attempt to read from BADGES table
5. If found in BADGES, extract operator ID from BADGES<4>
6. If found in OPERS, extract operator name from OPERS<9>
7. Create session variable: "OPERID|BADGEID"
8. Check menu permissions in OPERS<18>
9. Verify 'SCHEDMENU.HTM' access
10. Proceed to WIPTRACK2_EDIT.HTM
```

**Error Conditions**:
- Badge not found in OPERS or BADGES → "INVALID OPERATOR BADGEID OR SYSTEM OPER ID"
- No menu access permissions → "YOU ARE NOT ALLOWED TO ACCESS THIS MENU"

### Work Order Status Determination

**File**: WIPTRACK2_EDIT.HTM

**Purpose**: Route operator to appropriate screen based on WO status

**Logic**:
```basic
1. Receive WOID from scan
2. Validate WOID exists in WO table
3. Check WO<90> for activation status
   - If empty → Error: "WORK ORDER HAS NOT BEEN ACTIVATED"
4. Read WIPSTATUS record for WOID
5. Check WIPSTATUS<1>:
   - '' or 'P' → Route to WIPTRACK3_EDIT.HTM (Start/Resume Paused)
   - 'O' → Route to WIPTRACK4_EDIT.HTM (Continue Active)
   - 'C' → Error: "WORK ORDER IS COMPLETED"
```

**Status Codes**:
- **Empty String ('')**: WO never started
- **'P'**: Paused/Suspended
- **'O'**: Open/Active (work in progress)
- **'C'**: Completed

### Location Tracking

**Purpose**: Track physical location of WIP through production

**Tables**: WO, WIPLOC

**Fields**:
- `WO<90,1>`: Current WIP location code
- `WIPLOC<1>`: Location description

**Process**:
1. Location set when work starts (WIPTRACK3_EDIT)
2. Can be updated during work (WIPTRACKING_EDIT)
3. Changes logged with timestamp
4. Final location recorded at completion

### Quantity Management

**Fields**:
- `WO<91,1>`: Current WIP balance

**Operations**:
- **Start Work**: Initialize WIP balance
- **Record Production**: Reduce WIP, increase finished goods
- **Record Scrap**: Reduce WIP balance
- **Complete**: Finalize and close balance

**Validation**:
- Quantity changes must not result in negative balance
- Scrap quantity ≤ current WIP balance
- Final completion must reconcile all quantities

---

## State Management

### WIP Status States

```
┌─────────────────────────────────────────────────────────────┐
│ WIP STATUS STATE MACHINE                                     │
└─────────────────────────────────────────────────────────────┘

        [NOT STARTED]
        Status = ''
             │
             │ Start Work
             ▼
        [OPEN/ACTIVE]
        Status = 'O'
             │
             ├──────────────┬──────────────┐
             │              │              │
         Continue      Pause/Suspend    Complete
             │              │              │
             ▼              ▼              ▼
        [OPEN/ACTIVE]  [PAUSED]      [COMPLETED]
        Status = 'O'   Status = 'P'  Status = 'C'
             ▲              │              │
             │              │              │
             └──── Resume ──┘              │
                                           ▼
                                      [CLOSED]
                                     (Terminal)
```

### State Transitions

| From State | Action | To State | File |
|------------|--------|----------|------|
| '' (Not Started) | Start Work | 'O' (Open) | WIPTRACKSTART2.HTM |
| 'O' (Open) | Pause Work | 'P' (Paused) | WIPTRACKSUSPEND2_EDIT.HTM |
| 'P' (Paused) | Resume Work | 'O' (Open) | WIPTRACK3_EDIT.HTM |
| 'O' (Open) | Complete Work | 'C' (Completed) | WIPTRACKCOMPLETE_EDIT.HTM |
| 'C' (Completed) | - | - | Terminal State |

---

## Session Variables

### Cookie-based Session Data

| Variable | Set By | Used By | Purpose | Format |
|----------|--------|---------|---------|--------|
| **INIT** | Login | All pages | User initials/ID | String |
| **INITIALS** | Login | All pages | Operator ID | String |
| **BADGEID** | WIPTRACK_EDIT | All WIP pages | Badge + Operator | "INIT\|BADGE" |
| **WOID** | WIPTRACK2_EDIT | All WIP pages | Current work order | WO Number |
| **OPERNAME** | WIPTRACK2_EDIT | Display | Operator name | String |
| **SONO** | Various | - | Sales order (cleared) | - |
| **QTNO** | Various | - | Quote (cleared) | - |
| **SESSIONID** | WIPMENU | - | Session ID | String |
| **VKEY** | WIPMENU | - | Validation key | String |

### Session Lifecycle

```
Login → Set INIT, INITIALS
  ↓
WIPMENU_EDIT → Clear SONO, QTNO, SESSIONID, VKEY
  ↓
WIPTRACK_EDIT → Set BADGEID
  ↓
WIPTRACK2_EDIT → Set WOID, OPERNAME
  ↓
WIPTRACKING_EDIT → Use all session variables
  ↓
Logout/Complete → Clear all variables
```

---

## Common Patterns

### 1. Badge Authentication Pattern

**Used in**: WIPTRACK_EDIT.HTM

```basic
PL_GETVAR BADGEID FROM 'BADGEID' ELSE BADGEID = ''
PL_GETVAR UPDATE FROM 'UPDATE' ELSE UPDATE = ''

BADGEID = TRIM(BADGEID)
BADGEID = OCONV(BADGEID,'MCU')

IF UPDATE NE '' THEN
    READ OPER FROM OPERSF,BADGEID ELSE
        READ BREC FROM BADGEF,BADGEID ELSE
            MSG = 'INVALID OPERATOR BADGEID OR SYSTEM OPER ID'
            BADGEID = ''
            GOTO 99
        END
    END

    IF OPER NE '' THEN
        BADGEID = BADGEID:'|':OPER<9>
    END
    IF BREC NE '' THEN
        BADGEID = BREC<4>:'|':BADGEID
    END

    PL_PUTVAR BADGEID IN 'BADGEID' ELSE NULL
    CALL PLW.PAGE('WIPTRACK2_EDIT.HTM','',ERR)
END
```

### 2. Work Order Validation Pattern

**Used in**: WIPTRACK2_EDIT.HTM

```basic
PL_GETVAR WOID FROM 'WOID' ELSE WOID = ''
PL_GETVAR BADGEID FROM 'BADGEID' ELSE BADGEID = ''

WOID = TRIM(WOID)
WOID = OCONV(WOID,'MCU')

READ WOREC FROM WOF,WOID ELSE WOREC = ''
IF WOREC = '' THEN
    MSG = 'INVALID WORK ORDER NUMBER:':WOID
    WOID = ''
    GOTO 99
END

IF WOREC<90> = '' THEN
    MSG = 'WORK ORDER HAS NOT BEEN ACTIVATED.':WOID
    WOID = ''
    GOTO 99
END

READ WIPSTATUS FROM WIPSTATUSF,WOID ELSE WIPSTATUS = ''
IF WIPSTATUS<1> = '' OR WIPSTATUS<1> = 'P' THEN
    CALL PLW.PAGE('WIPTRACK3_EDIT.HTM','',ERR)
END
IF WIPSTATUS<1> = 'O' THEN
    CALL PLW.PAGE('WIPTRACK4_EDIT.HTM','',ERR)
END
```

### 3. Menu Access Control Pattern

**Used in**: WIPMENU_EDIT.HTM

```basic
READ OPER FROM OPERSF,INITIALS<1> ELSE
    ERR = ''
    CALL PLW.PAGE('LOGIN.HTM','',ERR)
    RETURN
END

IF OPER<18> NE '' THEN
    LOCATE('SCHEDMENU.HTM',OPER<18>,1;HIT) ELSE
        ERRVAL = 'YOU ARE NOT ALLOWED TO ACCESS THIS MENU'
        PL_PUTVAR ERRVAL IN 'ERRVAL' ELSE NULL
        CALL PLW.PAGE('ERRORMSG.HTM','',ERR)
        RETURN
    END
END
```

### 4. Log Building Pattern

**Used in**: WIPTRACKLOG_EDIT.HTM

```basic
PL_GETVAR WOID FROM 'WOID' ELSE WOID = ''

WOLOG = '' ; RESULT = '' ; PROD.IDS = ''
CALL BUILDWIPSTATUS(WOID,RESULT,PROD.IDS)
WOLOG = PL_AMTOCRLF(RESULT)

READ WOREC FROM WOF,WOID ELSE WOREC = ''
ASSY = WOREC<32>
WIPLOC = WOREC<90,1>
WIPBAL = WOREC<91,1>

READ WIPLOCREC FROM WIPLOCF,WIPLOC ELSE WIPLOCREC = ''
WIPD = WIPLOCREC<1>
```

---

## Error Handling

### Common Error Messages

| Error Message | Cause | Resolution | File |
|---------------|-------|------------|------|
| INVALID OPERATOR BADGEID OR SYSTEM OPER ID | Badge not found in OPERS or BADGES | Verify badge is registered | WIPTRACK_EDIT.HTM |
| YOU ARE NOT ALLOWED TO ACCESS THIS MENU | Missing menu permissions | Add SCHEDMENU.HTM to OPERS<18> | WIPMENU_EDIT.HTM |
| INVALID WORK ORDER NUMBER | WO doesn't exist | Verify WO number | WIPTRACK2_EDIT.HTM |
| WORK ORDER HAS NOT BEEN ACTIVATED | WO<90> is empty | Activate WO first | WIPTRACK2_EDIT.HTM |
| WORK ORDER IS COMPLETED | WIPSTATUS<1> = 'C' | Cannot modify completed WO | WIPTRACK2_EDIT.HTM |

### Error Handling Pattern

```basic
* Initialize error variables
MSG = ''

* Validate input
IF <validation fails> THEN
    MSG = '<error message>'
    GOTO 99
END

* Process normally
...

* Error exit point
99 *
PL_PUTVAR MSG IN 'MSG' ELSE NULL
* Display error or return to previous page
```

### Database Error Handling

```basic
OPEN 'OPERS' TO OPERSF ELSE STOP 201,'OPERS'
OPEN 'BADGES' TO BADGEF ELSE STOP 201,'BADGES'
OPEN 'WO' TO WOF ELSE STOP 201,'WO'
OPEN 'WIPSTATUS' TO WIPSTATUSF ELSE STOP 201,'WIPSTATUS'
OPEN 'WIPLOC' TO WIPLOCF ELSE STOP 201,'WIPLOC'

READ <record> FROM <file>,<key> ELSE <record> = ''
IF <record> = '' THEN
    * Handle missing record
END
```

---

## Best Practices

### For Operators

1. **Badge Scanning**
   - Always scan badge clearly
   - Verify operator name displays correctly
   - Report badge scan issues immediately

2. **Work Order Selection**
   - Double-check WO number before proceeding
   - Verify assembly matches physical work
   - Check WO status before starting

3. **Scrap Recording**
   - Record scrap immediately when it occurs
   - Provide detailed reason codes
   - Include notes for unusual situations
   - Never batch scrap entries

4. **Status Updates**
   - Pause work when leaving station
   - Resume promptly when returning
   - Update location if WIP moves
   - Complete WO only when fully finished

### For Developers

1. **Session Management**
   - Always clear sensitive cookies (SONO, QTNO)
   - Validate session variables on each page
   - Use PL_GET_COOKIE and PL_SET_COOKIE consistently

2. **Data Validation**
   - Trim and uppercase all input (TRIM, OCONV MCU)
   - Validate before database operations
   - Provide clear error messages
   - Use GOTO 99 pattern for error exits

3. **Database Operations**
   - Always use ELSE clauses on OPEN statements
   - Check for empty records after READ
   - Use multi-valued fields correctly (field<attr,value>)
   - Lock records when updating

4. **UI/UX**
   - Set autofocus on primary input fields
   - Show clear visual feedback for actions
   - Display current status prominently
   - Use consistent button styling

### For System Administrators

1. **Badge Management**
   - Keep BADGES table synchronized with OPERS
   - Deactivate badges for terminated employees
   - Audit badge usage regularly

2. **Work Order Activation**
   - Ensure WO<90> is set before production starts
   - Verify location codes are valid
   - Monitor for stale open work orders

3. **Performance Monitoring**
   - Watch for slow WIPSTATUS queries
   - Monitor log file growth
   - Optimize BUILDWIPSTATUS function
   - Index frequently queried fields

4. **Security**
   - Review OPERS<18> permissions regularly
   - Audit log access for sensitive WOs
   - Monitor unauthorized access attempts
   - Implement badge access logging

---

## Troubleshooting

### Issue: Badge Won't Scan

**Symptoms**: Error "INVALID OPERATOR BADGEID"

**Possible Causes**:
1. Badge not registered in BADGES table
2. Operator not in OPERS table
3. Badge ID mismatch between tables
4. Special characters in badge ID

**Resolution**:
```sql
* Check BADGES table
LIST BADGES WITH @ID = "BADGEID"

* Check OPERS table
LIST OPERS WITH OPERS<9> = "BADGEID"

* Verify link
LIST BADGES BADGES<4> OPERS OPERS<9>
```

### Issue: Cannot Access WIP Menu

**Symptoms**: Error "YOU ARE NOT ALLOWED TO ACCESS THIS MENU"

**Resolution**:
```sql
* Check operator permissions
LIST OPERS WITH @ID = "OPERID" OPERS<18>

* Add SCHEDMENU.HTM permission
ED OPERS OPERID
<18> = append "SCHEDMENU.HTM"
```

### Issue: Work Order Shows Wrong Status

**Symptoms**: WO status doesn't match actual state

**Resolution**:
```sql
* Check WIPSTATUS
LIST WIPSTATUS WITH @ID = "WOID"

* Verify WO activation
LIST WO WITH @ID = "WOID" WO<90>

* Check for orphaned status
* Manually correct if needed
ED WIPSTATUS WOID
<1> = "O" or "P" or ""
```

### Issue: Scrap Entry Rejected

**Symptoms**: Cannot save scrap record

**Possible Causes**:
1. Scrap quantity > WIP balance
2. Invalid reason code
3. WO not in 'O' status

**Resolution**:
1. Verify WIP balance: `LIST WO WITH @ID = "WOID" WO<91,1>`
2. Check status: `LIST WIPSTATUS WITH @ID = "WOID"`
3. Adjust scrap quantity or WO status

### Issue: Log Not Displaying

**Symptoms**: WIPTRACKLOG_EDIT shows blank or error

**Possible Causes**:
1. BUILDWIPSTATUS function error
2. Missing WO record
3. Corrupted log data

**Resolution**:
```sql
* Verify WO exists
LIST WO WITH @ID = "WOID"

* Check log function
* Call manually: CALL BUILDWIPSTATUS("WOID", RESULT, PROD.IDS)
* Inspect RESULT variable

* Rebuild log if necessary
```

### Issue: Cannot Complete Work Order

**Symptoms**: Complete button disabled or errors

**Possible Causes**:
1. Outstanding scrap not recorded
2. Quantities don't reconcile
3. Missing required approvals
4. WO not in 'O' status

**Resolution**:
1. Reconcile all quantities
2. Record all scrap
3. Obtain necessary approvals
4. Verify WIPSTATUS = 'O'

---

## Appendices

### A. File Categories

**Entry/Menu Files**:
- WIPMENU_EDIT.HTM

**Workflow Files**:
- WIPTRACK_EDIT.HTM
- WIPTRACK2_EDIT.HTM
- WIPTRACK3_EDIT.HTM
- WIPTRACK4_EDIT.HTM

**Main Interface**:
- WIPTRACKING_EDIT.HTM

**Action Files**:
- WIPTRACKSTART2.HTM
- WIPTRACKSUSPEND2_EDIT.HTM
- WIPTRACKSCRAP2_EDIT.HTM
- WIPTRACKCOMPLETE_EDIT.HTM
- WIPTRACKFINAL_EDIT.HTM

**Utility Files**:
- WIPTRACKLOG_EDIT.HTM
- WIPTRACKREVIEW_EDIT.HTM
- WIPTRACKVAL_EDIT.HTM
- WIPTRACKAUTH_EDIT.HTM

**Services**:
- CDAWIPTRACK.HTM
- CDA_WIPTRACKLOG.HTM
- CDA_WIPTRACKSCRAP2.HTM
- CDA_WIPTRACK4.HTM

**Tools**:
- WIP_TEMPLATE_MASTER.HTM
- WIP_GENERATOR_TOOL.HTM
- WIP_GENERATOR.HTM

### B. Quick Reference Card

```
┌─────────────────────────────────────────────────────────────┐
│ WIPTRACK QUICK REFERENCE                                     │
├─────────────────────────────────────────────────────────────┤
│ START WORK                                                   │
│ 1. Scan badge → 2. Scan WO → 3. Start                      │
│                                                              │
│ PAUSE WORK                                                   │
│ Main Interface → Pause → Enter reason → Confirm             │
│                                                              │
│ RECORD SCRAP                                                 │
│ Main Interface → Scrap → Enter qty & reason → Submit        │
│                                                              │
│ COMPLETE WORK                                                │
│ Main Interface → Complete → Final checks → Finalize         │
│                                                              │
│ VIEW LOG                                                     │
│ Main Interface → View Log                                   │
├─────────────────────────────────────────────────────────────┤
│ STATUS CODES                                                 │
│ '' = Not Started  | 'P' = Paused | 'O' = Open | 'C' = Done │
├─────────────────────────────────────────────────────────────┤
│ KEY TABLES                                                   │
│ OPERS | BADGES | WO | WIPSTATUS | WIPLOC                   │
└─────────────────────────────────────────────────────────────┘
```

### C. Navigation Matrix

| From | To | Via | Purpose |
|------|----|----|---------|
| WIPMENU | WIPTRACK | Menu button | Start WIP tracking |
| WIPTRACK | WIPTRACK2 | Badge scan | Operator authenticated |
| WIPTRACK2 | WIPTRACK3 | Status='' or 'P' | Start new/paused WO |
| WIPTRACK2 | WIPTRACK4 | Status='O' | Resume active WO |
| WIPTRACK3 | WIPTRACKSTART2 | Start button | Initialize work |
| WIPTRACK4 | WIPTRACKING | Continue button | Resume work |
| WIPTRACKSTART2 | WIPTRACKING | Auto | Work started |
| WIPTRACKING | WIPTRACKSCRAP2 | Scrap button | Record scrap |
| WIPTRACKING | WIPTRACKSUSPEND2 | Pause button | Pause work |
| WIPTRACKING | WIPTRACKCOMPLETE | Complete button | Complete WO |
| WIPTRACKING | WIPTRACKLOG | View Log button | View activity |
| WIPTRACKCOMPLETE | WIPTRACKFINAL | Submit | Finalize completion |
| WIPTRACKFINAL | WIPMENU | Done | Return to menu |

---

## Document History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2025-11-19 | System Documentation | Initial creation of comprehensive WipTrack operations manual |

---

## Contact & Support

For questions or issues with the WipTrack module:

1. **Technical Support**: Contact IT Help Desk
2. **Operator Training**: See shift supervisor
3. **System Issues**: Open ticket with system administrator
4. **Documentation Updates**: Submit via change request process

---

*End of WipTrack Operations Manual*
