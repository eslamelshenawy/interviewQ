# أسئلة الانترفيو - Cisco IVR & Contact Center

## المحتويات
1. [Cisco CVP](#cisco-cvp)
2. [Cisco ICM/UCCE](#cisco-icmucce)
3. [IVR Flow Design](#ivr-flow-design)
4. [VoiceXML](#voicexml)
5. [Integration & APIs](#integration--apis)
6. [Troubleshooting](#troubleshooting)
7. [Scenario-Based Questions](#scenario-based-questions)

---

## Cisco CVP

### أسئلة أساسية

**Q1: What is Cisco CVP and what are its main components?**

**الإجابة:**
Cisco CVP (Customer Voice Portal) is an IVR platform for building self-service voice applications.

**Main Components:**
1. **CVP Call Server**: Manages SIP signaling and call sessions
2. **CVP VXML Server**: Serves VoiceXML applications and executes IVR logic
3. **CVP Reporting Server**: Collects call data and generates reports
4. **CVP Operations Console**: Management interface

**Example:**
```
Caller → SIP Gateway → CVP Call Server → VXML Server
                            ↓
                       ICM Router
```

---

**Q2: What's the difference between Comprehensive and Standalone CVP deployments?**

**الإجابة:**

| Feature | Comprehensive | Standalone |
|---------|--------------|------------|
| **ICM Integration** | Yes | No |
| **Agent Routing** | Yes | Limited |
| **Use Case** | Full contact center | Self-service only |
| **Complexity** | High | Low |
| **Cost** | Higher | Lower |

**Example:**
```
Comprehensive: IVR → Agent Transfer → ICM → Agent Desktop
Standalone: IVR → Self-service → End Call
```

---

**Q3: Explain the CVP call flow from start to end.**

**الإجابة:**
```
1. Customer dials number
   ↓
2. Gateway routes call to CVP Call Server (SIP INVITE)
   ↓
3. CVP queries ICM for routing decision
   ↓
4. ICM returns VXML application URL
   ↓
5. CVP fetches VXML from VXML Server
   ↓
6. Customer interacts with IVR (DTMF/Speech)
   ↓
7. Two paths:
   a) Self-service completed → End call
   b) Transfer to agent → CVP+ICM routing
   ↓
8. Call ends / Agent answers
```

---

**Q4: What are ECC variables and how are they used?**

**الإجابة:**
**ECC (Extended Call Context)** variables pass data between CVP, ICM, and Agent Desktop.

**Example:**
```javascript
// In CVP VXML - Collecting data
Call.ECC.CustomerID = "123456"
Call.ECC.AccountNumber = "789012"
Call.ECC.ServiceType = "Balance Inquiry"
Call.ECC.Language = "Arabic"

// Sent to ICM for routing decisions

// Displayed on Agent Desktop when call arrives
Agent sees:
- Customer ID: 123456
- Account: 789012
- Service Requested: Balance Inquiry
- Preferred Language: Arabic
```

**Types:**
- **ECC Arrays**: `Call.ECC.Array[0]` to `Call.ECC.Array[9]`
- **Named ECC**: Custom variable names

---

**Q5: How do you handle timeout and no-input in CVP?**

**الإجابة:**
```xml
<field name="accountNumber">
    <prompt>Please enter your account number</prompt>

    <!-- No input handler -->
    <noinput count="1">
        <prompt>I didn't hear anything. Please enter your account number.</prompt>
        <reprompt/>
    </noinput>

    <noinput count="2">
        <prompt>I still can't hear you. Let me transfer you to an agent.</prompt>
        <goto next="#transferAgent"/>
    </noinput>

    <!-- Timeout property -->
    <property name="timeout" value="5s"/>
</field>
```

**Best Practices:**
- Max 2-3 retries before transferring to agent
- Clear error messages
- Always provide alternative (press 0 for agent)

---

### أسئلة متقدمة

**Q6: How would you optimize CVP performance?**

**الإجابة:**

**1. Audio Caching:**
```xml
<!-- Use CDN for audio files -->
<audio src="https://cdn.bank.com/ivr/welcome.wav"/>

<!-- Cache frequently used prompts -->
<property name="audiocaching" value="true"/>
```

**2. API Call Optimization:**
```xml
<!-- Bad: Multiple calls -->
<data name="name" src="api/getName"/>
<data name="balance" src="api/getBalance"/>
<data name="status" src="api/getStatus"/>

<!-- Good: Single call -->
<data name="customer" src="api/getCustomerProfile"/>
<!-- Returns: {name, balance, status, ...} -->
```

**3. Session Variables:**
```xml
<!-- Store once, reuse multiple times -->
<var name="session.customerId" expr="'123456'"/>
<value expr="session.customerId"/> <!-- Reuse -->
```

**4. Connection Pooling:**
```java
// In Spring Boot API
spring.datasource.hikari.maximum-pool-size=20
spring.datasource.hikari.minimum-idle=5
```

---

**Q7: Explain CVP high availability and failover.**

**الإجابة:**

**Architecture:**
```
        Load Balancer
              ↓
    ┌─────────┴─────────┐
    ↓                   ↓
CVP Server 1      CVP Server 2
(Primary)         (Standby)
    ↓                   ↓
    └─────────┬─────────┘
              ↓
         ICM Cluster
```

**Failover Scenarios:**

1. **CVP Call Server Failure:**
   - Load balancer detects failure (health check)
   - Routes new calls to standby server
   - Active calls may drop (depends on configuration)

2. **VXML Server Failure:**
   - Use multiple VXML servers behind load balancer
   - Stateless design allows any server to handle request

3. **ICM Router Failure:**
   - ICM has Side A and Side B routers
   - Automatic failover within seconds
   - No call loss

**Configuration:**
```xml
<!-- Multiple VXML servers -->
<property name="vxml.server.primary" value="http://vxml1.bank.com"/>
<property name="vxml.server.backup" value="http://vxml2.bank.com"/>
```

---

**Q8: How do you secure sensitive data in CVP?**

**الإجابة:**

**1. Mask PIN Input:**
```xml
<field name="pin" type="digits">
    <property name="inputmodes" value="dtmf"/>
    <!-- Don't log PIN -->
    <property name="secure" value="true"/>
    <prompt>Enter your 4-digit PIN</prompt>
</field>
```

**2. Use HTTPS:**
```xml
<data name="account"
      src="https://secure-api.bank.com/accounts"
      method="post"
      enctype="application/json"/>
```

**3. Encrypt API Payloads:**
```java
// Spring Boot API
@PostMapping("/auth/validate-pin")
public ResponseEntity<AuthResponse> validatePin(
        @RequestBody @Encrypted PinRequest request) {
    // PIN is encrypted in transit
    String decryptedPin = encryptionService.decrypt(request.getPin());
    // Validate against hashed PIN in database
}
```

**4. Session Timeout:**
```xml
<property name="timeout" value="10s"/>
<property name="sessiontimeout" value="300s"/> <!-- 5 minutes -->
```

**5. Don't Log Sensitive Data:**
```java
// Bad
log.info("PIN entered: {}", pin); // ❌ Never log PINs!

// Good
log.info("PIN validation attempted for account: {}", accountNumber); // ✅
```

---

## Cisco ICM/UCCE

### أسئلة أساسية

**Q9: What is Cisco ICM and how does it differ from UCCE?**

**الإجابة:**

**Cisco ICM (Intelligent Contact Management):**
- Enterprise-wide contact routing solution
- Multiple sites, complex routing
- Used by large organizations

**Cisco UCCE (Unified Contact Center Enterprise):**
- Packaged version of ICM
- Includes CVP, Finesse, Unified CM
- Easier deployment and management
- Single or multi-site

**Key Difference:**
```
ICM = Routing Engine only
UCCE = ICM + CVP + Finesse + CUCM (Complete package)
```

---

**Q10: Explain ICM architecture components.**

**الإجابة:**

```
┌──────────────────────────────────────┐
│         ICM Architecture             │
└──────────────────────────────────────┘

1. ICM Router
   - Executes routing scripts
   - Makes routing decisions
   - Real-time call processing

2. ICM Logger
   - Stores historical data
   - Configuration database
   - Reporting data
   - Call detail records (CDRs)

3. ICM Peripheral Gateway (PG)
   - Interface between ICM and peripherals
   - Communicates with CVP, ACD, etc.
   - Monitors agent status

4. ICM Admin Workstation (AW)
   - Management tool
   - Script Editor
   - Configuration
   - Monitoring

Communication:
Router ↔ PG ↔ CVP
   ↓       ↓
Logger   Logger
```

---

**Q11: What are ICM routing scripts? Provide an example.**

**الإجابة:**

**ICM Routing Scripts** define how calls are routed.

**Example: Business Hours Routing**
```javascript
START:
    // Get current time
    Get System.Hour
    Get System.DayOfWeek

CHECK_HOURS:
    IF (System.DayOfWeek >= 1 AND System.DayOfWeek <= 5) THEN
        // Weekday
        IF (System.Hour >= 8 AND System.Hour < 17) THEN
            // Business hours
            GOTO ROUTE_TO_AGENT
        ELSE
            // After hours
            GOTO AFTER_HOURS
        END IF
    ELSE
        // Weekend
        GOTO AFTER_HOURS
    END IF

ROUTE_TO_AGENT:
    // Get service type from IVR
    Set ServiceType = Call.CallerEnteredDigits

    IF (ServiceType == "1") THEN
        // Sales
        Queue to Skill Group = "Sales_Team"
        Priority = 5
    ELSE IF (ServiceType == "2") THEN
        // Support
        Queue to Skill Group = "Support_Team"
        Priority = 5
    END IF

    GOTO END

AFTER_HOURS:
    Play "Our office is closed"
    Play "Business hours are 8 AM to 5 PM"
    Send to Voicemail

END:
    Terminate
```

---

**Q12: Explain skills-based routing in ICM.**

**الإجابة:**

**Skills-Based Routing** matches calls to agents based on required skills.

**Example Setup:**

**1. Define Skills:**
```
Skills:
- Language (English: 1-5, Arabic: 1-5)
- Technical Knowledge (1-5)
- Product Expertise (Billing, Loans, Cards)
- Customer Service (1-5)
```

**2. Create Skill Groups:**
```
Skill Group: "Arabic_Technical_Support"
Required Skills:
  - Arabic: Level 5
  - Technical: Level 4+
  - Customer Service: Level 3+
```

**3. Assign Agents:**
```
Agent: Ahmed Ali (ID: 12345)
Skills:
  - Arabic: 5/5
  - English: 3/5
  - Technical: 4/5
  - Customer Service: 4/5

Assigned to:
  - Arabic_Technical_Support (Primary)
  - General_Support (Secondary)
```

**4. Routing Logic:**
```javascript
START:
    Get Call.Language from IVR
    Get Call.Issue from IVR

DETERMINE_SKILL:
    IF (Call.Language == "Arabic" AND Call.Issue == "Technical") THEN
        Set SkillGroup = "Arabic_Technical_Support"
    ELSE IF (Call.Language == "English" AND Call.Issue == "Technical") THEN
        Set SkillGroup = "English_Technical_Support"
    END IF

QUEUE:
    Queue to Skill Group = SkillGroup
    Priority = Call.Priority

    // ICM automatically selects best agent based on:
    // 1. Skill level match
    // 2. Longest idle time
    // 3. Service level agreements
```

---

**Q13: What are ICM call variables and how do you use them?**

**الإجابة:**

**Call Variables** store information about the call for routing decisions.

**Types:**

**1. System Call Variables:**
```javascript
Call.ANI              // Caller's phone number
Call.DNIS             // Dialed number
Call.CallerEnteredDigits  // IVR input
Call.PeripheralVariable1-10  // Custom variables
```

**2. Extended Call Context (ECC):**
```javascript
Call.ECC.CustomerID = "123456"
Call.ECC.AccountNumber = "789012"
Call.ECC.VIPStatus = "TRUE"
Call.ECC.Language = "AR"
```

**3. Custom Variables:**
```javascript
Call.Priority = 10
Call.ServiceType = "Technical"
Call.WaitTime = 120
```

**Example Usage:**
```javascript
// In ICM Script
START:
    // Get customer info from database
    Database Lookup: "SELECT * FROM customers WHERE phone = Call.ANI"

    IF (Customer.VIP == TRUE) THEN
        Set Call.Priority = 10
        Set Call.SkillGroup = "VIP_Support"
    ELSE
        Set Call.Priority = 5
        Set Call.SkillGroup = "General_Support"
    END IF

    Queue to Skill Group = Call.SkillGroup
    Priority = Call.Priority
```

---

### أسئلة متقدمة

**Q14: How would you implement overflow routing in ICM?**

**الإجابة:**

**Overflow Routing** redirects calls when primary queue is full or wait time is too long.

**Strategy:**
```javascript
QUEUE_PRIMARY:
    Queue to Skill Group = "Primary_Team"
    Priority = 5
    Timeout = 60 seconds

    // Check result
    IF (Result == "Timeout") THEN
        // No agent available in 60 seconds
        GOTO OVERFLOW_LEVEL_1
    ELSE IF (Result == "Queued") THEN
        // Success
        GOTO END
    END IF

OVERFLOW_LEVEL_1:
    // Try secondary team
    IF (SkillGroup["Secondary_Team"].AgentsReady > 0) THEN
        Queue to Skill Group = "Secondary_Team"
        Priority = 7  // Higher priority in overflow
        Timeout = 90 seconds

        IF (Result == "Timeout") THEN
            GOTO OVERFLOW_LEVEL_2
        END IF
    ELSE
        GOTO OVERFLOW_LEVEL_2
    END IF

OVERFLOW_LEVEL_2:
    // Try another site
    IF (SkillGroup["RemoteSite_Team"].AgentsReady > 0) THEN
        Queue to Skill Group = "RemoteSite_Team"
        Priority = 10
        Timeout = 120 seconds

        IF (Result == "Timeout") THEN
            GOTO OFFER_CALLBACK
        END IF
    ELSE
        GOTO OFFER_CALLBACK
    END IF

OFFER_CALLBACK:
    // No agents available anywhere
    Play "All agents are busy"
    Play "Press 1 for callback, 2 for voicemail"

    Get Digits

    IF (Call.EnteredDigits == "1") THEN
        Insert into CallbackQueue
        Play "We will call you back"
    ELSE
        Transfer to Voicemail
    END IF

END:
    Terminate
```

---

**Q15: Explain ICM script debugging techniques.**

**الإجابة:**

**Debugging Techniques:**

**1. Script Editor Debugger:**
```
Steps:
1. Open script in Script Editor
2. Set breakpoints at nodes
3. Click "Debug" button
4. Enter test call data:
   - ANI: +966501234567
   - DNIS: 800123456
   - Caller Entered Digits: 1
5. Step through script
6. Inspect variables at each node
7. Verify routing decisions
```

**2. Logging:**
```javascript
// Add logging nodes in script
LOG_START:
    Log "Call received from: " + Call.ANI
    Log "Service requested: " + Call.ServiceType
    Log "Routing to: " + Call.SkillGroup

// Check logs
// C:\icm\logs\router.log
```

**3. Real-time Monitoring:**
```
Tools:
- RTTEST (Real-Time Test tool)
- ICM Web Tools
- Finesse Desktop (agent view)

Monitor:
- Call arrival
- Routing decisions
- Queue status
- Agent status
```

**4. Historical Reporting:**
```sql
-- Query ICM database
SELECT
    DateTime,
    RouterCallKeyDay,
    OriginatorDN,  -- ANI
    DestinationDN, -- DNIS
    SkillTargetID,
    Duration
FROM Termination_Call_Detail
WHERE DateTime >= DATEADD(hour, -1, GETDATE())
ORDER BY DateTime DESC
```

**5. Common Issues:**

**Issue: Calls not routing**
```javascript
// Check:
1. Skill Group exists?
   - Verify in ICM configuration

2. Agents logged in?
   - Check agent availability

3. Script logic correct?
   - Add logs at each decision point

// Solution:
IF (SkillGroupExists(Call.SkillGroup)) THEN
    IF (SkillGroup[Call.SkillGroup].AgentsReady > 0) THEN
        Queue to Skill Group
    ELSE
        Log "No agents available"
        GOTO BACKUP_QUEUE
    END IF
ELSE
    Log "Skill Group not found: " + Call.SkillGroup
    GOTO ERROR_HANDLER
END IF
```

---

## IVR Flow Design

**Q16: What are the key principles of good IVR design?**

**الإجابة:**

**1. Simplicity:**
```
✅ Good: 3-5 menu options
❌ Bad: 10+ menu options (caller forgets first options)
```

**2. Efficiency:**
```
✅ Good: 3 steps to reach service
Main Menu → Service Selection → Execute
❌ Bad: 7 steps with unnecessary confirmations
```

**3. Clarity:**
```
✅ Good: "Press 1 for account balance"
❌ Bad: "Press 1 for account-related financial information services"
```

**4. Escape Options:**
```
✅ Always provide:
- Press 0 for agent (in every menu)
- Press * to go back
- Press # to replay menu
```

**5. Error Tolerance:**
```xml
<!-- Allow retries -->
<noinput count="1">
    I didn't hear anything. Please try again.
</noinput>
<noinput count="2">
    Let me transfer you to an agent.
</noinput>
```

---

**Q17: How would you design an IVR for a banking application?**

**الإجابة:**

**Complete Banking IVR Design:**

```
┌─────────────────────────────────────────┐
│            CALL ARRIVES                 │
└────────────────┬────────────────────────┘
                 ↓
┌─────────────────────────────────────────┐
│ Welcome Message                         │
│ "Welcome to ABC Bank"                   │
└────────────────┬────────────────────────┘
                 ↓
┌─────────────────────────────────────────┐
│ Language Selection                      │
│ "Press 1 for English, 2 for Arabic"    │
└────────┬───────────────┬────────────────┘
         ↓ 1             ↓ 2
      English         Arabic
         │               │
         └───────┬───────┘
                 ↓
┌─────────────────────────────────────────┐
│ ANI Recognition                         │
│ Check if caller is known customer       │
└────┬─────────────────────────┬──────────┘
     ↓ Known                   ↓ Unknown
┌──────────────┐        ┌──────────────┐
│ "Welcome     │        │ "Please enter│
│  Ahmed"      │        │  account #"  │
└──────┬───────┘        └──────┬───────┘
       │                       │
       └───────────┬───────────┘
                   ↓
         ┌──────────────────┐
         │ Authentication   │
         │ "Enter 4-digit   │
         │  PIN"            │
         └────────┬─────────┘
                  ↓
         ┌──────────────────┐
         │ Main Menu        │
         ├──────────────────┤
         │ 1. Balance       │
         │ 2. Transactions  │
         │ 3. Transfer      │
         │ 4. Bill Payment  │
         │ 0. Agent         │
         └────────┬─────────┘
                  ↓
    ┌─────────────┼─────────────┬─────────────┐
    ↓             ↓             ↓             ↓
┌────────┐  ┌────────────┐  ┌────────┐  ┌────────┐
│Balance │  │Transactions│  │Transfer│  │  Bill  │
│Inquiry │  │  History   │  │  Money │  │Payment │
└───┬────┘  └─────┬──────┘  └────┬───┘  └────┬───┘
    │             │              │           │
    ↓             ↓              ↓           ↓
[Execute]     [Execute]      [Execute]   [Execute]
    │             │              │           │
    └─────────────┴──────────────┴───────────┘
                  ↓
         ┌──────────────────┐
         │ More Services?   │
         │ 1. Yes           │
         │ 2. No, Goodbye   │
         └────────┬─────────┘
                  ↓
              [Loop or Exit]
```

**Key Features:**
1. **Quick access**: Balance inquiry in 3 steps
2. **Authentication**: PIN required for sensitive operations
3. **Personalization**: Welcome by name for known customers
4. **Multiple languages**: English & Arabic
5. **Always escape**: Press 0 for agent anytime

---

**Q18: How do you handle errors and edge cases in IVR?**

**الإجابة:**

**Error Handling Strategy:**

**1. No Input:**
```xml
<field name="choice">
    <prompt>Press 1 for Sales, 2 for Support</prompt>

    <noinput count="1">
        <prompt>I didn't hear anything. Please press 1 or 2.</prompt>
        <reprompt/>
    </noinput>

    <noinput count="2">
        <prompt>
            I'm having trouble hearing you.
            Let me connect you to an agent.
        </prompt>
        <goto next="#transferAgent"/>
    </noinput>
</field>
```

**2. Invalid Input:**
```xml
<field name="accountNumber" type="digits">
    <prompt>Enter your 10-digit account number</prompt>

    <nomatch count="1">
        <prompt>
            That doesn't seem right.
            Please enter exactly 10 digits.
        </prompt>
        <clear field="accountNumber"/>
        <reprompt/>
    </nomatch>

    <nomatch count="2">
        <prompt>
            I'm still having trouble.
            Press 0 to speak with an agent.
        </prompt>
        <goto next="#transferAgent"/>
    </nomatch>

    <!-- Validate length -->
    <filled>
        <if cond="accountNumber.length != 10">
            <clear field="accountNumber"/>
            <prompt>
                Account number must be exactly 10 digits.
                Please try again.
            </prompt>
            <reprompt/>
        </if>
    </filled>
</field>
```

**3. System Errors:**
```xml
<catch event="error.badfetch">
    <prompt>
        I'm sorry, I'm having trouble connecting to our system.
        Let me transfer you to an agent who can help.
    </prompt>
    <goto next="#transferAgent"/>
</catch>

<catch event="error.timeout">
    <prompt>
        Our system is taking longer than expected.
        Press 1 to continue waiting, or 0 for an agent.
    </prompt>
    <field name="timeoutChoice">
        <option dtmf="1" value="wait">Continue waiting</option>
        <option dtmf="0" value="agent">Speak to agent</option>

        <filled>
            <if cond="timeoutChoice == 'agent'">
                <goto next="#transferAgent"/>
            <else/>
                <goto next="#retryOperation"/>
            </if>
        </filled>
    </field>
</catch>
```

**4. Business Logic Errors:**
```xml
<block>
    <!-- Call balance API -->
    <data name="balance" src="http://api.bank.com/balance"/>

    <if cond="balance.error != null">
        <!-- Handle specific errors -->
        <if cond="balance.error.code == 'ACCOUNT_LOCKED'">
            <prompt>
                Your account is temporarily locked.
                Please call customer service at 1-800-BANK.
            </prompt>
            <goto next="#end"/>

        <elseif cond="balance.error.code == 'ACCOUNT_NOT_FOUND'"/>
            <prompt>
                I couldn't find that account number.
                Press 1 to try again, or 0 for an agent.
            </prompt>
            <goto next="#accountRetry"/>

        <else/>
            <!-- Unknown error -->
            <prompt>
                I encountered an unexpected error.
                Transferring you to an agent.
            </prompt>
            <goto next="#transferAgent"/>
        </if>
    </if>
</block>
```

---

## Integration & APIs

**Q19: How do you integrate IVR with Spring Boot backend?**

**الإجابة:**

**Complete Integration Example:**

**1. Spring Boot REST Controller:**
```java
@RestController
@RequestMapping("/api/ivr")
public class IVRController {

    @GetMapping("/customers/{accountNumber}")
    public ResponseEntity<CustomerResponse> getCustomer(
            @PathVariable String accountNumber) {

        CustomerResponse customer = CustomerResponse.builder()
                .customerId("123456")
                .name("Ahmed Ali")
                .accountType("Premium")
                .balance(5432.10)
                .build();

        return ResponseEntity.ok(customer);
    }

    @PostMapping("/auth/validate-pin")
    public ResponseEntity<AuthResponse> validatePin(
            @RequestBody AuthRequest request) {

        boolean valid = authService.validatePin(
                request.getAccountNumber(),
                request.getPin()
        );

        AuthResponse response = AuthResponse.builder()
                .valid(valid)
                .token(valid ? jwtUtil.generateToken(request.getAccountNumber()) : null)
                .attemptsRemaining(valid ? null : 2)
                .build();

        return ResponseEntity.ok(response);
    }
}
```

**2. CVP VXML Integration:**
```xml
<?xml version="1.0"?>
<vxml version="2.1">
    <form id="getCustomer">
        <var name="accountNumber" expr="'123456'"/>

        <!-- HTTP GET Request -->
        <data name="customer"
              src="'http://api.bank.com/api/ivr/customers/' + accountNumber"
              method="get"
              timeout="5s">

            <property name="Content-Type" value="'application/json'"/>
        </data>

        <block>
            <if cond="customer != null">
                <!-- Success -->
                <prompt>
                    Welcome <value expr="customer.name"/>.
                    Your <value expr="customer.accountType"/> account
                    balance is <value expr="customer.balance"/> dollars.
                </prompt>
            <else/>
                <!-- Error -->
                <prompt>
                    I'm sorry, I couldn't retrieve your information.
                    Press 0 for an agent.
                </prompt>
            </if>
        </block>
    </form>

    <form id="validatePin">
        <var name="accountNumber" expr="'123456'"/>
        <var name="pin" expr="'1234'"/>

        <!-- HTTP POST Request -->
        <data name="authResult"
              src="'http://api.bank.com/api/ivr/auth/validate-pin'"
              method="post"
              timeout="5s"
              enctype="application/json">

            <property name="Content-Type" value="'application/json'"/>
            <property name="body" value="{
                'accountNumber': accountNumber,
                'pin': pin
            }"/>
        </data>

        <block>
            <if cond="authResult.valid == true">
                <!-- Success -->
                <var name="session.token" expr="authResult.token"/>
                <prompt>PIN accepted.</prompt>
                <goto next="#mainMenu"/>
            <else/>
                <!-- Failed -->
                <prompt>
                    Invalid PIN.
                    You have <value expr="authResult.attemptsRemaining"/> attempts remaining.
                </prompt>
                <goto next="#retryPin"/>
            </if>
        </block>
    </form>
</vxml>
```

**3. Error Handling:**
```java
// Spring Boot Exception Handler
@ControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(CustomerNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleCustomerNotFound(
            CustomerNotFoundException ex) {

        ErrorResponse error = ErrorResponse.builder()
                .code("CUSTOMER_NOT_FOUND")
                .message("Customer not found")
                .timestamp(LocalDateTime.now())
                .build();

        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(error);
    }

    @ExceptionHandler(InvalidPinException.class)
    public ResponseEntity<ErrorResponse> handleInvalidPin(
            InvalidPinException ex) {

        ErrorResponse error = ErrorResponse.builder()
                .code("INVALID_PIN")
                .message("Invalid PIN")
                .attemptsRemaining(ex.getAttemptsRemaining())
                .build();

        return ResponseEntity.status(HttpStatus.UNAUTHORIZED).body(error);
    }
}
```

---

## Scenario-Based Questions

**Q20: A customer reports that the IVR is not accepting their PIN even though it's correct. How would you troubleshoot?**

**الإجابة:**

**Troubleshooting Steps:**

**1. Check IVR Logs:**
```bash
# CVP logs
tail -f /cvp/logs/application.log

# Look for:
- Call ID
- Account number entered
- PIN validation API call
- API response
```

**2. Verify API is Working:**
```bash
# Test API directly
curl -X POST http://api.bank.com/api/ivr/auth/validate-pin \
  -H "Content-Type: application/json" \
  -d '{"accountNumber":"123456","pin":"1234"}'

# Expected response:
{
  "valid": true,
  "token": "eyJhbGc...",
  "attemptsRemaining": null
}
```

**3. Check Database:**
```sql
-- Verify customer PIN
SELECT account_number, pin_hash, is_locked, failed_attempts
FROM customers
WHERE account_number = '123456';

-- Check if account is locked
-- Check failed attempts counter
```

**4. Possible Issues:**

**Issue 1: DTMF not recognized**
```xml
<!-- Solution: Add grammar -->
<field name="pin" type="digits">
    <grammar type="application/x-gsl">
        [(0-9){4}]
    </grammar>
    <property name="inputmodes" value="dtmf"/>
</field>
```

**Issue 2: PIN comparison case-sensitive**
```java
// Bug:
if (enteredPin.equals(storedPin)) // May fail if stored as hash

// Fix:
if (passwordEncoder.matches(enteredPin, storedPinHash))
```

**Issue 3: Network timeout**
```xml
<!-- Increase timeout -->
<data name="authResult"
      src="..."
      timeout="10s"/> <!-- Increased from 5s -->
```

**Issue 4: Session/token expired**
```java
// Check token expiration
if (jwtUtil.isTokenExpired(token)) {
    return "EXPIRED_SESSION";
}
```

**5. Resolution Steps:**
```
1. Identify root cause from logs
2. Fix the issue (code, config, or network)
3. Test with customer's account
4. Verify fix in production
5. Monitor for similar issues
6. Document the issue and resolution
```

---

تم إنشاء ملف شامل لأسئلة الانترفيو! الآن سأنشئ الملف الإنجليزي للأسئلة: