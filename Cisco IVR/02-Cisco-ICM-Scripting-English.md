# Cisco ICM Scripting - Comprehensive Guide

## Table of Contents
1. [Introduction to ICM](#introduction-to-icm)
2. [ICM Architecture](#icm-architecture)
3. [Script Editor](#script-editor)
4. [Script Types](#script-types)
5. [Nodes and Types](#nodes-and-types)
6. [Variables and Types](#variables-and-types)
7. [Call Routing Logic](#call-routing-logic)
8. [Skills-Based Routing](#skills-based-routing)
9. [Practical Examples](#practical-examples)
10. [Best Practices](#best-practices)
11. [Debugging](#debugging)

---

## Introduction to ICM

### What is Cisco ICM?

**Cisco ICM (Intelligent Contact Management)** is an advanced system for managing and routing calls in contact centers.

### Key Functions

✅ **Intelligent Call Routing**: Smart call distribution
✅ **Skills-Based Routing**: Route by agent skills
✅ **Queue Management**: Manage waiting queues
✅ **Real-Time Reporting**: Live reporting
✅ **Multi-Channel Support**: Voice, chat, email support

### ICM vs UCCE

| Feature | ICM | UCCE |
|---------|-----|------|
| **Scope** | Enterprise-wide | Unified Contact Center |
| **Complexity** | More complex | Easier to manage |
| **Cost** | Higher | Lower |
| **Deployment** | Multi-site | Single/Multi-site |

---

## ICM Architecture

### Core Components

```
┌─────────────────────────────────────────────────┐
│              Unified CCE Architecture           │
└─────────────────────────────────────────────────┘

┌──────────────┐         ┌──────────────┐
│   ICM Logger │◄───────►│   ICM AW     │
│   (Database) │         │ (Admin & Data)│
└──────┬───────┘         └──────┬───────┘
       │                        │
       ├────────────────────────┤
       │                        │
┌──────▼───────┐         ┌──────▼───────┐
│ ICM Router   │◄───────►│   ICM PG     │
│ (Routing)    │         │ (Peripheral) │
└──────┬───────┘         └──────┬───────┘
       │                        │
       │                        ↓
       │                 ┌──────────────┐
       │                 │     CVP      │
       │                 └──────────────┘
       │                        │
       ↓                        ↓
┌──────────────┐         ┌──────────────┐
│    Agent     │         │   Caller     │
│  (Finesse)   │         │    (IVR)     │
└──────────────┘         └──────────────┘
```

### Component Details

#### 1. ICM Router
- Execute routing scripts
- Make routing decisions
- Manage queues
- Real-time call processing

#### 2. ICM Logger
- Store historical data
- Configuration data
- Reporting data
- Call detail records (CDRs)

#### 3. ICM Peripheral Gateway (PG)
- Mediator between ICM and external systems
- Communicate with CVP
- Monitor agent status
- Call management

#### 4. ICM Admin Workstation (AW)
- ICM management tool
- Script Editor
- Configuration tools
- Monitoring tools

---

## Script Editor

### Main Interface

**Script Editor** is the graphical tool for building routing scripts in ICM.

### Interface Components

```
┌─────────────────────────────────────────────┐
│  File  Edit  View  Tools  Debug  Help      │
├─────────────────────────────────────────────┤
│  [Palette]                    [Properties]  │
│  ┌─────────┐                  ┌──────────┐ │
│  │ Nodes   │    [Canvas]      │ Node     │ │
│  │ ├─Start │   ┌───────────┐  │ Settings │ │
│  │ ├─Queue │   │           │  │          │ │
│  │ ├─Select│   │  Script   │  │ Name:    │ │
│  │ ├─If    │   │  Flow     │  │ ______   │ │
│  │ ├─End   │   │           │  │          │ │
│  │ └─...   │   └───────────┘  └──────────┘ │
│  └─────────┘                                │
└─────────────────────────────────────────────┘
```

### Palette (Toolbox)

Contains all available **Nodes**:
- **Queue Nodes**: Queue management
- **Decision Nodes**: Decision making
- **Data Nodes**: Data handling
- **Routing Nodes**: Call routing

---

## Script Types

### 1. Routing Scripts

**Usage:** Route calls to appropriate agents or queues

**Example:**
```
Start
  ↓
Check Time of Day
  ↓
IF Business Hours
  ↓ Yes
  Queue to Available Agent
ELSE
  ↓ No
  Play After Hours Message
  ↓
End
```

### 2. Admin Scripts

**Usage:** Administrative and maintenance tasks

**Examples:**
- Agent re-skilling scripts
- Emergency routing scripts
- Maintenance mode scripts

### 3. Network Scripts

**Usage:** Route calls between multiple sites

**Example:**
```
Start
  ↓
Check Caller Location
  ↓
Route to Nearest Site
  ↓
IF Site Unavailable
  ↓
Route to Backup Site
```

### 4. Translation Scripts

**Usage:** Translate DNIS/ANI numbers

---

## Nodes and Types

### 1. Queue Nodes

#### Queue to Skill Group
```
┌─────────────────────────┐
│  Queue to Skill Group   │
│  ─────────────────────  │
│  Skill Group: Technical │
│  Priority: 5            │
│  Timeout: 120 sec       │
└─────────────────────────┘
```

**Usage:**
- Place call in queue
- Route to specific skill group
- Set priority

**Example:**
```javascript
// Queue to Technical Support with high priority
node.skillGroup = "Technical_Support"
node.priority = 10
node.timeout = 180
```

#### Queue to Agent
```
┌─────────────────────────┐
│    Queue to Agent       │
│  ─────────────────────  │
│  Agent ID: 12345        │
│  Priority: 10           │
└─────────────────────────┘
```

### 2. Select Nodes

#### Select Node
```
┌─────────────────────────┐
│      Select             │
│  ─────────────────────  │
│  Select Best Agent      │
│  Based on:              │
│  □ Longest Idle         │
│  ☑ Skill Level          │
│  ☑ Service Level        │
└─────────────────────────┘
```

**Usage:**
- Select best available agent
- Based on specific criteria

### 3. Decision Nodes

#### If Node
```
┌─────────────────────────┐
│         IF              │
│  ─────────────────────  │
│  Condition:             │
│  Call.Language == "AR"  │
│                         │
│  True ──► [Arabic Team] │
│  False ──► [Eng Team]   │
└─────────────────────────┘
```

**Example:**
```javascript
IF (Call.CallerEnteredDigits == "1")
    GOTO Sales_Menu
ELSE IF (Call.CallerEnteredDigits == "2")
    GOTO Support_Menu
ELSE
    GOTO Invalid_Input
END IF
```

#### Case Node
```
┌─────────────────────────┐
│        CASE             │
│  ─────────────────────  │
│  Variable: ServiceType  │
│                         │
│  1 ──► Sales            │
│  2 ──► Support          │
│  3 ──► Billing          │
│  Default ──► Main Menu  │
└─────────────────────────┘
```

### 4. Data Nodes

#### Set Variable
```
┌─────────────────────────┐
│    Set Variable         │
│  ─────────────────────  │
│  Variable: Priority     │
│  Value: 10              │
└─────────────────────────┘
```

#### Database Lookup (DBL)
```
┌─────────────────────────┐
│   Database Lookup       │
│  ─────────────────────  │
│  Query: GetCustomer     │
│  Input: ANI             │
│  Output: CustomerInfo   │
└─────────────────────────┘
```

**Example:**
```sql
-- Query definition
SELECT customer_id, customer_name, vip_status
FROM customers
WHERE phone_number = :Call.ANI
```

#### Run External Script
```
┌─────────────────────────┐
│  Run External Script    │
│  ─────────────────────  │
│  Script: GetBalance.jar │
│  Input: AccountNumber   │
│  Output: Balance        │
└─────────────────────────┘
```

### 5. Control Nodes

#### Label
```
┌─────────────────────────┐
│       Label             │
│  ─────────────────────  │
│  Name: Main_Menu        │
└─────────────────────────┘
```

#### Goto
```
┌─────────────────────────┐
│       GOTO              │
│  ─────────────────────  │
│  Target: Main_Menu      │
└─────────────────────────┘
```

#### Requalify
```
┌─────────────────────────┐
│     Requalify           │
│  ─────────────────────  │
│  Recalculate routing    │
└─────────────────────────┘
```

---

## Variables and Types

### 1. Call Variables

**Description:** Data specific to current call

```javascript
// System Call Variables
Call.ANI              // Caller number
Call.DNIS             // Dialed number
Call.CallerEnteredDigits  // Digits entered in IVR
Call.PeripheralVariable1  // Customizable variables
Call.PeripheralVariable10

// Custom Call Variables
Call.CustomerID = "123456"
Call.AccountType = "Premium"
Call.Language = "Arabic"
Call.Priority = 10
```

### 2. ECC Variables (Extended Call Context)

**Description:** Extended variables for additional data

```javascript
// ECC Array Variables
Call.ECC.Array[0] = "CustomerID"
Call.ECC.Array[1] = "AccountNumber"
Call.ECC.Array[2] = "ServiceType"

// Named ECC Variables
Call.ECC.CustomerName = "Ahmed Ali"
Call.ECC.AccountBalance = "5000"
Call.ECC.LastCallDate = "2024-01-15"
```

**Use Cases:**
- Pass data from IVR to ICM
- Pass data from ICM to Agent Desktop
- Store custom data

### 3. System Variables

```javascript
// Time Variables
System.DateTime        // Current date and time
System.DayOfWeek       // Day of week (0-6)
System.Hour            // Hour (0-23)

// System Status
System.SkillGroupAvailable
System.AgentsReady
System.CallsInQueue
```

### 4. Formula Variables

**Description:** Calculated variables based on formulas

```javascript
// Formula example
VIP_Priority = IF(Customer.Type == "VIP", 10, 5)

// Another example
Working_Hours = (System.Hour >= 8 AND System.Hour < 17)
```

---

## Call Routing Logic

### Comprehensive Example: Banking Contact Center

```
┌─────────────────────────────────────────────────┐
│                   START                         │
└────────────────────┬────────────────────────────┘
                     ↓
┌─────────────────────────────────────────────────┐
│           Set Call.ReceivedTime                 │
└────────────────────┬────────────────────────────┘
                     ↓
┌─────────────────────────────────────────────────┐
│      Database Lookup: Get Customer Info         │
│      Input: Call.ANI                            │
│      Output: CustomerInfo                       │
└────────────────────┬────────────────────────────┘
                     ↓
┌─────────────────────────────────────────────────┐
│         IF CustomerInfo.Found                   │
└────┬────────────────────────────────────┬───────┘
     ↓ Yes                                 ↓ No
┌─────────────────┐              ┌─────────────────┐
│ Set VIP Status  │              │ Set as New      │
│ from Database   │              │ Customer        │
└────────┬────────┘              └────────┬────────┘
         ↓                                 ↓
         └────────────┬────────────────────┘
                      ↓
┌─────────────────────────────────────────────────┐
│        Check Business Hours                     │
│        IF (Hour >= 8 AND Hour < 17)             │
└────┬─────────────────────────────────────┬──────┘
     ↓ Business Hours                      ↓ After Hours
┌─────────────────┐              ┌─────────────────────┐
│ Check Service   │              │ Play After Hours    │
│ Type from IVR   │              │ Message             │
└────────┬────────┘              └─────────┬───────────┘
         ↓                                  ↓
    ┌────┴────┬────────┐                  END
    ↓         ↓        ↓
┌───────┐ ┌───────┐ ┌───────┐
│ Sales │ │Support│ │Billing│
└───┬───┘ └───┬───┘ └───┬───┘
    ↓         ↓         ↓
┌───────────────────────────┐
│   Check Agent Availability│
└───────┬───────────────────┘
        ↓
┌───────┴────────┐
│ Agents Ready?  │
└───┬────────┬───┘
    ↓ Yes    ↓ No
┌───────┐ ┌────────┐
│Queue  │ │Play    │
│to     │ │Busy    │
│Agent  │ │Message │
└───┬───┘ └────┬───┘
    ↓          ↓
   END       Callback?
              ↓
           ┌──┴──┐
           │Queue│
           │for  │
           │Call │
           │Back │
           └──┬──┘
              ↓
             END
```

### Script Code Example

```javascript
/************************************************
 * Banking IVR Routing Script
 * Description: Routes calls based on customer
 *              type, time, and service
 ************************************************/

// Start Node
START:
    // Log call receipt
    Set Call.StartTime = System.DateTime
    Set Call.Application = "Banking_IVR"

// Get Customer Info
DATABASE_LOOKUP:
    Query: "GetCustomerInfo"
    Input: Call.ANI
    Output: Customer.ID, Customer.Name, Customer.VIP, Customer.Language

    IF (Customer.ID EXISTS) THEN
        Set Call.CustomerID = Customer.ID
        Set Call.CustomerName = Customer.Name
        Set Call.VIPStatus = Customer.VIP
        Set Call.Language = Customer.Language
    ELSE
        Set Call.VIPStatus = FALSE
        Set Call.Language = "EN"  // Default
    END IF

// Check Business Hours
CHECK_HOURS:
    IF (System.Hour >= 8 AND System.Hour < 17 AND System.DayOfWeek >= 1 AND System.DayOfWeek <= 5) THEN
        GOTO BUSINESS_HOURS
    ELSE
        GOTO AFTER_HOURS
    END IF

// Business Hours Routing
BUSINESS_HOURS:
    // Get service type from IVR
    Set ServiceType = Call.CallerEnteredDigits

    CASE ServiceType:
        CASE "1":  // Sales
            Set Call.SkillGroup = "Sales_Team"
            Set Call.Priority = 5
            GOTO QUEUE_CALL

        CASE "2":  // Technical Support
            Set Call.SkillGroup = "Tech_Support"
            IF (Call.VIPStatus == TRUE) THEN
                Set Call.Priority = 10
            ELSE
                Set Call.Priority = 5
            END IF
            GOTO QUEUE_CALL

        CASE "3":  // Billing
            Set Call.SkillGroup = "Billing_Team"
            Set Call.Priority = 7
            GOTO QUEUE_CALL

        DEFAULT:
            GOTO INVALID_INPUT
    END CASE

// Queue to Agent
QUEUE_CALL:
    // Check if agents available
    IF (SkillGroup[Call.SkillGroup].AgentsReady > 0) THEN
        Queue to Skill Group = Call.SkillGroup
        Priority = Call.Priority
        Timeout = 180 seconds
    ELSE
        // No agents available
        Play "All agents are busy"

        // Offer callback
        Get Digits "Press 1 for callback, 2 to hold"

        IF (Call.CallerEnteredDigits == "1") THEN
            Set Call.CallbackNumber = Call.ANI
            Insert into CallbackQueue
            Play "We will call you back"
            GOTO END
        ELSE
            Queue to Skill Group = Call.SkillGroup
            Priority = Call.Priority
            Timeout = 300 seconds
        END IF
    END IF

    GOTO END

// After Hours
AFTER_HOURS:
    Play "Our office is currently closed"
    Play "Business hours are 8 AM to 5 PM"

    // Offer voicemail
    Get Digits "Press 1 to leave a message"

    IF (Call.CallerEnteredDigits == "1") THEN
        Transfer to Voicemail
    END IF

    GOTO END

// Invalid Input
INVALID_INPUT:
    Play "Invalid selection"
    GOTO BUSINESS_HOURS

// End
END:
    Set Call.EndTime = System.DateTime
    Terminate
```

---

## Skills-Based Routing

### Skills-Based Routing Concept

**Definition:** Route calls based on agent skills that match caller needs.

### Skill Groups

```javascript
// Define Skill Groups
SkillGroup: "English_Technical_Support"
    Skills Required:
        - Language: English (Level 5)
        - Technical Knowledge: Level 4
        - Customer Service: Level 3

SkillGroup: "Arabic_Billing"
    Skills Required:
        - Language: Arabic (Level 5)
        - Billing Systems: Level 4
        - Customer Service: Level 3

SkillGroup: "VIP_Support"
    Skills Required:
        - Language: English/Arabic (Level 5)
        - Technical: Level 5
        - VIP Handling: Level 5
```

### Agent Skills

```javascript
// Agent Profile Example
Agent: "Ahmed Ali" (ID: 12345)
    Skills:
        - Arabic: 5/5
        - English: 3/5
        - Technical Support: 4/5
        - Billing: 2/5
        - VIP Handling: 4/5

    Assigned Skill Groups:
        - Arabic_Technical_Support (Primary)
        - VIP_Support (Secondary)
```

### Routing Logic

```javascript
// Skills-Based Routing Script
START:
    Get Call.Language from IVR
    Get Call.ServiceType from IVR
    Get Call.VIPStatus from Database

DETERMINE_SKILL_GROUP:
    IF (Call.VIPStatus == TRUE) THEN
        Set RequiredSkillGroup = "VIP_Support"
        Set Call.Priority = 10
    ELSE IF (Call.ServiceType == "Technical" AND Call.Language == "Arabic") THEN
        Set RequiredSkillGroup = "Arabic_Technical_Support"
        Set Call.Priority = 5
    ELSE IF (Call.ServiceType == "Billing" AND Call.Language == "Arabic") THEN
        Set RequiredSkillGroup = "Arabic_Billing"
        Set Call.Priority = 5
    END IF

QUEUE:
    Queue to Skill Group = RequiredSkillGroup
    Priority = Call.Priority

    // Select best agent based on:
    // 1. Longest Idle Time
    // 2. Highest Skill Level
    // 3. Service Level Agreement (SLA)
```

---

## Practical Examples

### Example 1: Multi-Language Support

```javascript
START:
    Play "Welcome. Press 1 for English, 2 for Arabic"
    Get Digits

    Set Call.Language = Call.CallerEnteredDigits

    IF (Call.Language == "1") THEN
        Set Call.LanguageSkill = "English"
        Set Call.Prompt = "EN"
    ELSE IF (Call.Language == "2") THEN
        Set Call.LanguageSkill = "Arabic"
        Set Call.Prompt = "AR"
    ELSE
        GOTO START  // Invalid input, retry
    END IF

    // Continue to main menu
    GOTO MAIN_MENU
```

### Example 2: Priority Routing for VIP

```javascript
DATABASE_LOOKUP:
    Query: "GetCustomerTier"
    Input: Call.ANI
    Output: Customer.Tier

ASSIGN_PRIORITY:
    CASE Customer.Tier:
        CASE "Platinum":
            Set Call.Priority = 10
            Set Call.SkillGroup = "VIP_Platinum"
            Play "Welcome Platinum Member"

        CASE "Gold":
            Set Call.Priority = 7
            Set Call.SkillGroup = "VIP_Gold"
            Play "Welcome Gold Member"

        CASE "Silver":
            Set Call.Priority = 5
            Set Call.SkillGroup = "Regular_Support"

        DEFAULT:
            Set Call.Priority = 3
            Set Call.SkillGroup = "Regular_Support"
    END CASE
```

### Example 3: Overflow Routing

```javascript
QUEUE_PRIMARY:
    Queue to Skill Group = "Primary_Team"
    Priority = 5
    Timeout = 60 seconds

    // Check queue result
    IF (Result == "Timeout") THEN
        GOTO OVERFLOW
    ELSE IF (Result == "Queued") THEN
        GOTO AGENT_CONNECTED
    END IF

OVERFLOW:
    // Check secondary team
    IF (SkillGroup["Backup_Team"].AgentsReady > 0) THEN
        Queue to Skill Group = "Backup_Team"
        Priority = 7  // Increase priority
        Timeout = 120 seconds
    ELSE
        // Check third level
        GOTO OVERFLOW_LEVEL_2
    END IF
```

### Example 4: Time-Based Routing

```javascript
TIME_ROUTING:
    Get System.Hour
    Get System.DayOfWeek

    // Weekend
    IF (System.DayOfWeek == 0 OR System.DayOfWeek == 6) THEN
        Set Call.SkillGroup = "Weekend_Team"
        GOTO QUEUE_CALL
    END IF

    // Weekday routing
    IF (System.Hour >= 8 AND System.Hour < 12) THEN
        // Morning shift
        Set Call.SkillGroup = "Morning_Team"
    ELSE IF (System.Hour >= 12 AND System.Hour < 17) THEN
        // Afternoon shift
        Set Call.SkillGroup = "Afternoon_Team"
    ELSE IF (System.Hour >= 17 AND System.Hour < 24) THEN
        // Evening shift
        Set Call.SkillGroup = "Evening_Team"
    ELSE
        // After hours
        GOTO AFTER_HOURS
    END IF
```

---

## Best Practices

### 1. Script Organization

✅ **Use clear labels**
```javascript
// Bad
Label: L1

// Good
Label: CHECK_BUSINESS_HOURS
```

✅ **Add comments**
```javascript
/************************************************
 * Section: Customer Authentication
 * Purpose: Verify customer identity using PIN
 * Author: Ahmed Ali
 * Date: 2024-01-15
 ************************************************/
```

✅ **Break down large scripts**
```
Main_Script
├── Authentication_Module
├── Routing_Module
├── Queue_Management_Module
└── Reporting_Module
```

### 2. Error Handling

✅ **Handle all cases**
```javascript
DATABASE_LOOKUP:
    Query: "GetCustomer"

    IF (Result == "Success") THEN
        GOTO PROCESS_CUSTOMER
    ELSE IF (Result == "NotFound") THEN
        GOTO NEW_CUSTOMER
    ELSE IF (Result == "DatabaseError") THEN
        Log "Database connection failed"
        GOTO ERROR_HANDLER
    ELSE
        // Unexpected error
        GOTO GENERAL_ERROR
    END IF
```

### 3. Performance

✅ **Avoid unnecessary loops**
```javascript
// Bad: Can cause infinite loop
RETRY:
    Queue to Agent
    IF (Result == "Failed") THEN
        GOTO RETRY
    END IF

// Good: With retry limit
Set RetryCount = 0
RETRY:
    Queue to Agent
    IF (Result == "Failed") THEN
        Set RetryCount = RetryCount + 1
        IF (RetryCount < 3) THEN
            GOTO RETRY
        ELSE
            GOTO TRANSFER_TO_SUPERVISOR
        END IF
    END IF
```

✅ **Use caching for data**
```javascript
// Cache customer data for session
IF (Session.CustomerData == NULL) THEN
    Database Lookup: GetCustomer
    Set Session.CustomerData = Result
ELSE
    // Use cached data
    Set CustomerInfo = Session.CustomerData
END IF
```

### 4. Reporting Variables

✅ **Collect useful reporting data**
```javascript
// At call start
Set Call.StartTime = System.DateTime
Set Call.Application = "CustomerService"
Set Call.EntryPoint = "MainIVR"

// During routing
Set Call.SkillGroupSelected = "Technical_Support"
Set Call.QueueTime = System.DateTime - Call.StartTime

// At call end
Set Call.EndTime = System.DateTime
Set Call.TotalDuration = Call.EndTime - Call.StartTime
Set Call.Resolution = "Resolved"  // or "Transferred", "Abandoned"
```

---

## Debugging

### 1. Script Editor Debugger

**Debugging Steps:**

```
1. Open Script in Script Editor
   ↓
2. Set Breakpoints at nodes
   ↓
3. Click "Debug" button
   ↓
4. Enter test call data
   ↓
5. Step through script
   ↓
6. Inspect variables
```

**Debugging Tools:**
- **Breakpoints**: Pause execution at specific node
- **Step Into**: Execute one line
- **Step Over**: Skip subroutine
- **Watch Variables**: Monitor variable values
- **Call Stack**: View call sequence

### 2. Log Analysis

**Log Types:**

```bash
# Router logs
C:\icm\logs\router.log

# PG logs
C:\icm\logs\pg.log

# Script logs
C:\icm\logs\script_execution.log
```

**Example log entry:**
```
2024-01-15 10:30:45.123 [CallID: 123456789]
[Script: CustomerService_Routing]
[Node: QUEUE_TO_AGENT]
[Result: Queued]
[SkillGroup: Technical_Support]
[Priority: 5]
[Agent: 12345 - Ahmed Ali]
```

### 3. Common Issues

#### Issue: Calls not routing

**Possible Cause:**
- Skill Group doesn't exist
- No agents available
- Script logic error

**Solution:**
```javascript
// Add validation
IF (SkillGroupExists(Call.SkillGroup)) THEN
    IF (SkillGroup[Call.SkillGroup].AgentsReady > 0) THEN
        Queue to Skill Group
    ELSE
        Log "No agents available"
        GOTO OVERFLOW
    END IF
ELSE
    Log "Skill Group not found: " + Call.SkillGroup
    GOTO ERROR_HANDLER
END IF
```

#### Issue: Variables not passing

**Cause:**
- Variable name mismatch
- Scope issues
- Timing problems

**Solution:**
```javascript
// Verify variable names match
// In CVP:
Call.ECC.AccountNumber = "123456"

// In ICM Script:
// Use exact same name
IF (Call.ECC.AccountNumber != NULL) THEN
    Set CustomerAccount = Call.ECC.AccountNumber
ELSE
    Log "Account number not received from IVR"
END IF
```

---

## Summary

### Key Points

1. **ICM Scripts** are the heart of routing system
2. **Nodes** build routing logic
3. **Variables** pass data through system
4. **Skills-Based Routing** routes calls to right agents
5. **Debugging** is essential for fixing issues

### Required Skills

- Understanding ICM architecture
- Proficiency in Script Editor
- Knowledge of nodes and variables
- Debugging skills
- Understanding Contact Center workflows

---

## Next Step

After mastering ICM Scripting, move to:
1. IVR Flow Design
2. Call Routing strategies
3. CVP-ICM Integration
4. UCCE Administration
