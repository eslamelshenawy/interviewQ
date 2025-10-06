# Cisco Customer Voice Portal (CVP) - Comprehensive Guide

## Table of Contents
1. [Introduction to CVP](#introduction-to-cvp)
2. [Architecture](#architecture)
3. [Core Components](#core-components)
4. [CVP Deployment Types](#cvp-deployment-types)
5. [CVP Studio](#cvp-studio)
6. [Call Flow in CVP](#call-flow-in-cvp)
7. [ICM Integration](#icm-integration)
8. [VoiceXML](#voicexml)
9. [Best Practices](#best-practices)
10. [Troubleshooting](#troubleshooting)

---

## Introduction to CVP

### What is Cisco Customer Voice Portal (CVP)?

**Cisco CVP** is an advanced IVR (Interactive Voice Response) platform that enables organizations to:
- Build interactive self-service voice applications
- Provide sophisticated automated customer service
- Integrate with backend systems (CRM, Databases, APIs)
- Intelligently route calls to appropriate agents

### Key Benefits

✅ **Cost Reduction**: Minimize need for human agents
✅ **24/7 Service**: Always available customer service
✅ **Scalability**: Supports thousands of concurrent calls
✅ **Customization**: Build tailored voice experiences
✅ **Integration**: Easy integration with enterprise systems

---

## Architecture

### Core Components in CVP System

```
┌─────────────────────────────────────────────────────┐
│                    PSTN/SIP                         │
└──────────────────┬──────────────────────────────────┘
                   │
          ┌────────▼────────┐
          │  Cisco CUBE/    │
          │  SIP Gateway    │
          └────────┬────────┘
                   │
          ┌────────▼────────┐
          │  CVP Server     │◄──────┐
          │  (Call Server)  │       │
          └────────┬────────┘       │
                   │                │
          ┌────────▼────────┐       │
          │  VXML Server    │       │
          │  (CVP Studio)   │       │
          └────────┬────────┘       │
                   │                │
          ┌────────▼────────┐       │
          │  ICM/UCCE       │───────┘
          │  (Routing)      │
          └────────┬────────┘
                   │
          ┌────────▼────────┐
          │  Backend APIs   │
          │  (Spring Boot)  │
          └─────────────────┘
```

---

## Core Components

### 1. CVP Call Server

**Role:**
- Manage call sessions
- Communicate with ICM for routing decisions
- Process SIP signaling
- Manage media resources

**Key Functions:**
- Call admission control
- SIP call handling
- Session management
- Reporting and logging

### 2. CVP VXML Server

**Role:**
- Serve VoiceXML pages to caller
- Process user inputs (DTMF/Speech)
- Execute application logic
- Integrate with backend APIs

**Capabilities:**
- VoiceXML interpretation
- TTS (Text-to-Speech)
- ASR (Automatic Speech Recognition)
- Audio playback
- Data collection

### 3. CVP Reporting Server

**Role:**
- Collect call data
- Generate detailed reports
- Performance analysis
- Historical data storage

### 4. CVP Operations Console

**Role:**
- CVP management interface
- System monitoring
- Application management
- Troubleshooting

---

## CVP Deployment Types

### 1. Comprehensive Deployment

```
Caller → Gateway → CVP → ICM → Agent
                    ↓
              VXML Server
                    ↓
            Backend Systems
```

**Features:**
- Full self-service support
- Agent queuing and routing
- Complete ICM/UCCE integration
- Most flexible and powerful

**Use Cases:**
- Large contact centers
- Complex IVR requirements
- Multiple applications

### 2. Call Studio Deployment

```
Caller → Gateway → CVP VXML Server → Backend APIs
```

**Features:**
- Self-service only (no agent routing)
- Simpler implementation
- Lower cost
- No ICM required

**Use Cases:**
- Simple self-service applications
- Account inquiries
- Bill payment
- Information services

### 3. Standalone Deployment

**Features:**
- CVP without ICM
- Limited routing capabilities
- Suitable for small applications

---

## CVP Studio

### What is CVP Studio?

**CVP Studio** is a GUI-based development tool for building IVR applications without manually writing VoiceXML.

### Main Components

#### 1. Elements

**Call Control Elements:**
- `Start` - Entry point
- `End` - Exit point
- `Decision` - Make decisions
- `Subdialog` - Call other dialogs

**Voice Elements:**
- `Menu` - Voice menus
- `Play` - Play audio messages
- `Get Digits` - Collect DTMF digits
- `Get Speech` - Speech recognition

**Data Elements:**
- `Set` - Set variables
- `Database` - Database queries
- `HTTP Request` - REST API calls
- `Decision` - Conditional logic

**Integration Elements:**
- `Send To ICM` - Send to ICM
- `Web Service` - SOAP/REST calls
- `Java Class` - Execute Java code

#### 2. Variables

**Variable Types:**

```java
// Session Variables - Available throughout session
session.callerID
session.language
session.accountNumber

// Application Variables - Application specific
app.maxRetries = 3
app.timeout = 5

// Element Variables - Local to element
digit_input
menu_choice
```

#### 3. Audio Management

**Audio Types:**
- **Pre-recorded Audio**: Pre-saved WAV files
- **TTS (Text-to-Speech)**: Text to speech conversion
- **Dynamic Audio**: Dynamically built audio

**Example:**
```xml
<!-- Pre-recorded -->
<audio src="welcome.wav"/>

<!-- TTS -->
<audio>Welcome to customer service</audio>

<!-- Dynamic -->
<audio>Your balance is <value expr="session.balance"/> dollars</audio>
```

---

## Call Flow in CVP

### Example: Typical Call Flow

```
1. Customer calls → PSTN/SIP
          ↓
2. Gateway routes to CVP Call Server
          ↓
3. CVP queries ICM for routing decision
          ↓
4. ICM returns VoiceXML URL
          ↓
5. CVP fetches VXML from VXML Server
          ↓
6. Customer interacts with IVR
   - Language selection
   - Authentication
   - Service selection
          ↓
7. Two options:
   a) Self-service completion → End call
   b) Transfer to agent → CVP+ICM routing
```

### Detailed Example: Banking IVR

```
Start
  ↓
Welcome Message
"Welcome to ABC Bank"
  ↓
Language Selection
"Press 1 for English, 2 for Arabic"
  ↓
Main Menu
"Press 1 for Balance, 2 for Transfers, 3 for Agent"
  ↓
┌─────────┬─────────┬─────────┐
│ Balance │Transfer │  Agent  │
└────┬────┴────┬────┴────┬────┘
     ↓         ↓         ↓
  Get PIN   Get PIN   Route to
     ↓         ↓       Agent
  Validate  Validate
     ↓         ↓
  Call API  Call API
     ↓         ↓
  Play      Execute
  Balance   Transfer
     ↓         ↓
   End       End
```

---

## ICM Integration

### Why Integrate with ICM?

**ICM (Intelligent Contact Management)** provides:
- **Intelligent Routing**: Smart call routing
- **Skills-based Routing**: Route by agent skills
- **Queue Management**: Manage waiting queues
- **Reporting**: Comprehensive reports

### How Integration Works

#### 1. Call Flow with ICM

```
CVP Call Server
    ↓ (Send routing request)
ICM Peripheral Gateway (PG)
    ↓ (Execute routing script)
ICM Router
    ↓ (Return routing decision)
CVP
    ↓ (Execute VXML or transfer to agent)
```

#### 2. ICM Routing Scripts

**Example: Call routing script**

```
IF (Time of Day = Business Hours) THEN
    IF (Caller Language = Arabic) THEN
        Queue to Arabic_Support_Team
    ELSE
        Queue to English_Support_Team
    END IF
ELSE
    Play "We are closed" message
    Send to Voicemail
END IF
```

#### 3. CVP-ICM Variables

**Passing data between CVP and ICM:**

```java
// From CVP to ICM
callVariables.accountNumber = "123456"
callVariables.serviceType = "Technical Support"
callVariables.priority = "High"

// From ICM to CVP
agent.skillGroup = "Billing"
agent.extension = "1234"
queue.position = "5"
```

---

## VoiceXML

### What is VoiceXML?

**VoiceXML** is an XML language for building interactive voice applications.

### Basic Structure

```xml
<?xml version="1.0"?>
<vxml version="2.1">
    <form id="mainMenu">
        <!-- Play welcome message -->
        <block>
            <prompt>
                Welcome to customer service.
            </prompt>
        </block>

        <!-- Get user input -->
        <field name="menuChoice">
            <prompt>
                Press 1 for account balance.
                Press 2 for bill payment.
                Press 0 for operator.
            </prompt>

            <grammar type="application/x-gsl">
                [
                    (one 1)    {<menuChoice "1">}
                    (two 2)    {<menuChoice "2">}
                    (zero 0)   {<menuChoice "0">}
                ]
            </grammar>

            <filled>
                <if cond="menuChoice == '1'">
                    <goto next="#checkBalance"/>
                <elseif cond="menuChoice == '2'"/>
                    <goto next="#payBill"/>
                <elseif cond="menuChoice == '0'"/>
                    <goto next="#transferAgent"/>
                </if>
            </filled>
        </field>
    </form>

    <form id="checkBalance">
        <!-- Balance inquiry logic -->
        <block>
            <prompt>Your current balance is 1000 dollars.</prompt>
            <goto next="#mainMenu"/>
        </block>
    </form>
</vxml>
```

### Core Elements

#### 1. `<form>` - Form

```xml
<form id="getAccountNumber">
    <field name="accountNum" type="digits">
        <prompt>Please enter your 10 digit account number</prompt>
    </field>
</form>
```

#### 2. `<field>` - Input Field

```xml
<field name="language">
    <prompt>Press 1 for English, 2 for Arabic</prompt>
    <option dtmf="1" value="en">English</option>
    <option dtmf="2" value="ar">Arabic</option>
</field>
```

#### 3. `<menu>` - Menu

```xml
<menu id="mainMenu">
    <prompt>
        Main menu.
        Say or press 1 for sales.
        Say or press 2 for support.
    </prompt>
    <choice dtmf="1" next="#sales">Sales</choice>
    <choice dtmf="2" next="#support">Support</choice>
</menu>
```

#### 4. `<block>` - Execution Block

```xml
<block>
    <prompt>Please wait while we retrieve your information</prompt>
    <data name="customerData" src="http://api.example.com/customer"/>
</block>
```

#### 5. `<subdialog>` - Subdialog

```xml
<subdialog name="auth" src="authentication.vxml">
    <param name="custId" expr="session.customerId"/>
    <filled>
        <if cond="auth.status == 'success'">
            <goto next="#authenticatedMenu"/>
        <else/>
            <goto next="#authFailed"/>
        </if>
    </filled>
</subdialog>
```

---

## Best Practices

### 1. User Experience Design

✅ **Keep menus short**
- No more than 5 options
- List most used options first

✅ **Use clear language**
```
❌ Bad: "For account-related options press 1"
✅ Good: "To check your balance press 1"
```

✅ **Always provide exit option**
```xml
<field name="choice">
    <prompt>
        Press 1 for balance, 2 for transfer.
        Press star to return to main menu.
        Press 0 for operator.
    </prompt>
</field>
```

### 2. Error Handling

✅ **Use limited retries**
```xml
<field name="accountNum">
    <prompt>Enter your account number</prompt>

    <noinput count="1">
        I didn't hear anything. Please enter your account number.
    </noinput>

    <noinput count="2">
        I still didn't hear you. Let me transfer you to an agent.
        <goto next="#transferAgent"/>
    </noinput>

    <nomatch count="1">
        I didn't understand. Please enter digits only.
    </nomatch>
</field>
```

### 3. Performance Optimization

✅ **Cache audio files**
- Use CDN for audio files
- Pre-load frequently used files

✅ **Optimize API calls**
```xml
<!-- Bad: Multiple API calls -->
<data name="name" src="api/getName"/>
<data name="balance" src="api/getBalance"/>

<!-- Good: Single API call -->
<data name="customer" src="api/getCustomerInfo"/>
```

✅ **Use session variables**
```xml
<!-- Store data once -->
<var name="session.customerId" expr="'123456'"/>
<!-- Reuse it -->
<value expr="session.customerId"/>
```

### 4. Security

✅ **Protect sensitive data**
```xml
<!-- Mask sensitive input -->
<field name="pin" type="digits">
    <property name="inputmodes" value="dtmf"/>
    <property name="confidencelevel" value="0.5"/>
    <prompt>Enter your 4 digit PIN</prompt>
    <!-- Don't log PIN -->
</field>
```

✅ **Use HTTPS**
```xml
<data name="account" src="https://secure.api.com/account"/>
```

✅ **Timeout sessions**
```xml
<property name="timeout" value="10s"/>
```

---

## Troubleshooting

### 1. Common Issues

#### Issue: No Audio Playback

**Possible Causes:**
- Incorrect audio file path
- Unsupported audio format
- Network issues

**Solution:**
```xml
<!-- Verify path -->
<audio src="http://cvp-server/audio/welcome.wav">
    <!-- Fallback TTS -->
    Welcome to our service
</audio>
```

#### Issue: DTMF Not Recognized

**Causes:**
- Incorrect grammar settings
- Gateway issues

**Solution:**
```xml
<field name="choice">
    <grammar type="application/x-gsl">
        [
            (1) {<choice "1">}
            (2) {<choice "2">}
        ]
    </grammar>
</field>
```

### 2. Debugging Tools

#### CVP Call Studio Debugger
```
- Breakpoints in call flow
- Variable inspection
- Step-by-step execution
```

#### CVP Logs
```bash
# Call Server logs
/cvp/logs/callserver.log

# VXML Server logs
/cvp/logs/vxmlserver.log

# Application logs
/cvp/logs/applications/myapp.log
```

### 3. Log Analysis

**Example log entry:**
```
2024-01-15 10:30:45 INFO [CallID: 123456]
Session started from 966501234567
Application: CustomerService
Entry point: MainMenu
```

**What to look for:**
- Call ID for tracking
- Error messages
- Session duration
- User inputs
- API response times

---

## Comprehensive Example: Banking IVR Application

### Call Flow Design

```
Start
  ↓
Welcome + Language Selection
  ↓
Main Menu
  ├─ 1: Check Balance
  │   ├─ Get Account Number
  │   ├─ Authenticate PIN
  │   ├─ Call Balance API
  │   └─ Play Balance
  │
  ├─ 2: Recent Transactions
  │   ├─ Get Account Number
  │   ├─ Authenticate PIN
  │   ├─ Call Transactions API
  │   └─ Play Transactions
  │
  ├─ 3: Transfer Money
  │   ├─ Authenticate
  │   ├─ Get Source Account
  │   ├─ Get Target Account
  │   ├─ Get Amount
  │   ├─ Confirm
  │   ├─ Call Transfer API
  │   └─ Confirmation
  │
  └─ 0: Speak to Agent
      └─ Transfer to ICM
```

### CVP Studio Implementation

This will be built using:
- **10+ Elements**: Start, Menu, Get Digits, Decision, HTTP Request, Play, End
- **15+ Variables**: language, accountNum, pin, balance, etc.
- **5+ Audio files**: Welcome, Menu, Error, Success, Goodbye
- **3+ API Integrations**: Balance, Transactions, Transfer APIs

---

## Summary

### Key Points

1. **CVP** is a powerful IVR platform for building voice applications
2. **CVP Studio** simplifies development without writing VoiceXML
3. **ICM integration** provides intelligent routing
4. **VoiceXML** is the core language for voice applications
5. **Best practices** are essential for good user experience

### Required Skills

- Understanding IVR concepts
- CVP Studio development
- VoiceXML programming
- ICM integration
- API integration
- Debugging and troubleshooting

### Next Steps

1. Learn ICM scripting
2. Understand call routing
3. Integrate with Spring Boot APIs
4. UCCE administration
5. Advanced troubleshooting

---

## Additional Resources

### Documentation
- Cisco CVP Official Documentation
- VoiceXML 2.1 Specification
- CVP Studio User Guide

### Training
- Cisco CVP Administration
- CVP Application Development
- ICM/UCCE Fundamentals

### Community
- Cisco Community Forums
- Stack Overflow - CVP tags
- GitHub - CVP samples
