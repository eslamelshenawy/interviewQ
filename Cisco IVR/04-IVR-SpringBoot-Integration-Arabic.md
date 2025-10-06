# IVR Integration with Spring Boot - الدليل الشامل

## المحتويات
1. [مقدمة](#مقدمة)
2. [Architecture Overview](#architecture-overview)
3. [Spring Boot Backend Setup](#spring-boot-backend-setup)
4. [REST API Design for IVR](#rest-api-design-for-ivr)
5. [CVP HTTP Request Integration](#cvp-http-request-integration)
6. [Authentication & Security](#authentication--security)
7. [Error Handling](#error-handling)
8. [Performance Optimization](#performance-optimization)
9. [Logging & Monitoring](#logging--monitoring)
10. [أمثلة عملية](#أمثلة-عملية)

---

## مقدمة

### لماذا Spring Boot؟

**Spring Boot** هو إطار عمل Java مثالي لبناء backend APIs للـ IVR:

✅ **سريع في التطوير**: Convention over configuration
✅ **Microservices Ready**: مناسب لبناء microservices
✅ **Robust**: موثوق وقوي
✅ **Scalable**: قابل للتوسع
✅ **Rich Ecosystem**: نظام بيئي غني

### Use Cases

**IVR يستخدم Spring Boot APIs لـ:**
- Authentication & Authorization
- Balance inquiries
- Transaction history
- Fund transfers
- Bill payments
- Customer profile lookup
- Real-time data retrieval

---

## Architecture Overview

### High-Level Architecture

```
┌─────────────────────────────────────────────────┐
│                    Caller                       │
└───────────────────┬─────────────────────────────┘
                    ↓
         ┌──────────────────────┐
         │    PSTN/SIP          │
         │    Gateway           │
         └──────────┬───────────┘
                    ↓
         ┌──────────────────────┐
         │    CVP Server        │
         │    (VXML)            │
         └──────────┬───────────┘
                    ↓
         ┌──────────────────────┐
         │    REST API Call     │
         │    (HTTP/HTTPS)      │
         └──────────┬───────────┘
                    ↓
┌───────────────────────────────────────────┐
│       Spring Boot Application             │
├───────────────────────────────────────────┤
│  ┌─────────────────────────────────────┐ │
│  │     REST Controllers                │ │
│  │  - AuthController                   │ │
│  │  - AccountController                │ │
│  │  - TransactionController            │ │
│  └────────────┬────────────────────────┘ │
│               ↓                           │
│  ┌─────────────────────────────────────┐ │
│  │     Service Layer                   │ │
│  │  - AuthService                      │ │
│  │  - AccountService                   │ │
│  │  - TransactionService               │ │
│  └────────────┬────────────────────────┘ │
│               ↓                           │
│  ┌─────────────────────────────────────┐ │
│  │     Repository Layer                │ │
│  │  - JPA Repositories                 │ │
│  └────────────┬────────────────────────┘ │
└───────────────┼───────────────────────────┘
                ↓
         ┌──────────────┐
         │   Database   │
         │   (MySQL/    │
         │   Oracle)    │
         └──────────────┘
```

### Component Interaction

```
CVP (VoiceXML)
    ↓
1. User enters account number
    ↓
<data name="customer"
      src="http://api.bank.com/ivr/customers/123456"
      method="get"/>
    ↓
2. Spring Boot receives request
    ↓
@GetMapping("/ivr/customers/{accountNumber}")
    ↓
3. Service processes request
    ↓
CustomerService.getCustomerByAccount(accountNumber)
    ↓
4. Database query
    ↓
CustomerRepository.findByAccountNumber(accountNumber)
    ↓
5. Return JSON response
    ↓
{
  "customerId": "123456",
  "name": "Ahmed Ali",
  "accountType": "Premium",
  "balance": 5432.10
}
    ↓
6. CVP processes response
    ↓
<prompt>
  Welcome <value expr="customer.name"/>
  Your balance is <value expr="customer.balance"/> dollars
</prompt>
```

---

## Spring Boot Backend Setup

### Project Structure

```
ivr-backend/
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com/
│   │   │       └── bank/
│   │   │           └── ivr/
│   │   │               ├── IvrApplication.java
│   │   │               ├── config/
│   │   │               │   ├── SecurityConfig.java
│   │   │               │   ├── WebConfig.java
│   │   │               │   └── SwaggerConfig.java
│   │   │               ├── controller/
│   │   │               │   ├── AuthController.java
│   │   │               │   ├── AccountController.java
│   │   │               │   ├── TransactionController.java
│   │   │               │   └── CustomerController.java
│   │   │               ├── service/
│   │   │               │   ├── AuthService.java
│   │   │               │   ├── AccountService.java
│   │   │               │   ├── TransactionService.java
│   │   │               │   └── CustomerService.java
│   │   │               ├── repository/
│   │   │               │   ├── CustomerRepository.java
│   │   │               │   ├── AccountRepository.java
│   │   │               │   └── TransactionRepository.java
│   │   │               ├── model/
│   │   │               │   ├── Customer.java
│   │   │               │   ├── Account.java
│   │   │               │   └── Transaction.java
│   │   │               ├── dto/
│   │   │               │   ├── AuthRequest.java
│   │   │               │   ├── AuthResponse.java
│   │   │               │   ├── BalanceResponse.java
│   │   │               │   └── TransferRequest.java
│   │   │               ├── exception/
│   │   │               │   ├── GlobalExceptionHandler.java
│   │   │               │   ├── CustomerNotFoundException.java
│   │   │               │   └── InvalidPinException.java
│   │   │               └── util/
│   │   │                   ├── JwtUtil.java
│   │   │                   └── EncryptionUtil.java
│   │   └── resources/
│   │       ├── application.yml
│   │       ├── application-dev.yml
│   │       └── application-prod.yml
│   └── test/
│       └── java/
│           └── com/
│               └── bank/
│                   └── ivr/
│                       ├── controller/
│                       ├── service/
│                       └── integration/
├── pom.xml
└── README.md
```

### pom.xml Dependencies

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.2.0</version>
    </parent>

    <groupId>com.bank</groupId>
    <artifactId>ivr-backend</artifactId>
    <version>1.0.0</version>
    <name>IVR Backend Services</name>

    <properties>
        <java.version>17</java.version>
    </properties>

    <dependencies>
        <!-- Spring Boot Starter Web -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!-- Spring Boot Starter Data JPA -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>

        <!-- Spring Boot Starter Security -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>

        <!-- Spring Boot Starter Validation -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
        </dependency>

        <!-- MySQL Driver -->
        <dependency>
            <groupId>com.mysql</groupId>
            <artifactId>mysql-connector-j</artifactId>
            <scope>runtime</scope>
        </dependency>

        <!-- JWT -->
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt-api</artifactId>
            <version>0.12.3</version>
        </dependency>
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt-impl</artifactId>
            <version>0.12.3</version>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt-jackson</artifactId>
            <version>0.12.3</version>
            <scope>runtime</scope>
        </dependency>

        <!-- Lombok -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>

        <!-- SpringDoc OpenAPI (Swagger) -->
        <dependency>
            <groupId>org.springdoc</groupId>
            <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
            <version>2.3.0</version>
        </dependency>

        <!-- Apache Commons -->
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-lang3</artifactId>
        </dependency>

        <!-- Spring Boot Actuator (Monitoring) -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>

        <!-- Micrometer Prometheus (Metrics) -->
        <dependency>
            <groupId>io.micrometer</groupId>
            <artifactId>micrometer-registry-prometheus</artifactId>
        </dependency>

        <!-- Test Dependencies -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.security</groupId>
            <artifactId>spring-security-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

### application.yml

```yaml
# Application Configuration
spring:
  application:
    name: ivr-backend-service

  # Database Configuration
  datasource:
    url: jdbc:mysql://localhost:3306/ivr_db?useSSL=false&serverTimezone=UTC
    username: ${DB_USERNAME:ivr_user}
    password: ${DB_PASSWORD:ivr_password}
    driver-class-name: com.mysql.cj.jdbc.Driver

  # JPA Configuration
  jpa:
    hibernate:
      ddl-auto: validate
    show-sql: false
    properties:
      hibernate:
        dialect: org.hibernate.dialect.MySQLDialect
        format_sql: true
    open-in-view: false

  # Jackson Configuration
  jackson:
    default-property-inclusion: non_null
    serialization:
      write-dates-as-timestamps: false

# Server Configuration
server:
  port: 8080
  servlet:
    context-path: /api/ivr
  compression:
    enabled: true
    mime-types: application/json,application/xml,text/html,text/xml,text/plain

# Security Configuration
security:
  jwt:
    secret-key: ${JWT_SECRET:your-256-bit-secret-key-here-change-in-production}
    expiration: 3600000 # 1 hour in milliseconds

# IVR Specific Configuration
ivr:
  max-retry-attempts: 3
  session-timeout: 300 # 5 minutes in seconds
  pin-lockout-duration: 900 # 15 minutes in seconds

# Logging Configuration
logging:
  level:
    root: INFO
    com.bank.ivr: DEBUG
    org.springframework.web: INFO
    org.hibernate: WARN
  pattern:
    console: "%d{yyyy-MM-dd HH:mm:ss} - %msg%n"
    file: "%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n"
  file:
    name: logs/ivr-backend.log
    max-size: 10MB
    max-history: 30

# Actuator Configuration (Health & Metrics)
management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics,prometheus
  endpoint:
    health:
      show-details: when-authorized
  metrics:
    export:
      prometheus:
        enabled: true

# Swagger/OpenAPI Configuration
springdoc:
  api-docs:
    path: /api-docs
  swagger-ui:
    path: /swagger-ui.html
    enabled: true
```

---

## REST API Design for IVR

### API Design Principles

#### 1. Keep It Simple

✅ **استخدم endpoints واضحة ومباشرة**

```
GET  /customers/{accountNumber}
POST /auth/validate-pin
GET  /accounts/{accountNumber}/balance
POST /accounts/transfer
```

#### 2. Return Only Needed Data

✅ **IVR لا يحتاج كل البيانات**

```java
// ❌ Bad: Too much data
{
  "customerId": "123",
  "firstName": "Ahmed",
  "middleName": "Mohamed",
  "lastName": "Ali",
  "dateOfBirth": "1990-01-15",
  "email": "ahmed@email.com",
  "phone": "+966501234567",
  "address": {
    "street": "King Fahd Road",
    "city": "Riyadh",
    "country": "Saudi Arabia",
    "postalCode": "12345"
  },
  "preferences": {...},
  "marketingConsent": true,
  ...
}

// ✅ Good: Only what IVR needs
{
  "customerId": "123",
  "name": "Ahmed Ali",
  "accountType": "Premium",
  "preferredLanguage": "AR"
}
```

#### 3. Fast Response Times

✅ **IVR sensitive to latency**

**Target Response Times:**
- Authentication: < 500ms
- Balance Inquiry: < 1000ms
- Transaction History: < 1500ms

### Main Controller

```java
package com.bank.ivr.controller;

import com.bank.ivr.dto.*;
import com.bank.ivr.service.CustomerService;
import io.swagger.v3.oas.annotations.Operation;
import io.swagger.v3.oas.annotations.tags.Tag;
import jakarta.validation.Valid;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

/**
 * REST Controller for IVR Customer Operations
 *
 * All endpoints are designed for IVR integration
 * and return minimal, fast responses
 */
@Slf4j
@RestController
@RequestMapping("/customers")
@RequiredArgsConstructor
@Tag(name = "Customer", description = "Customer management APIs for IVR")
public class CustomerController {

    private final CustomerService customerService;

    /**
     * Get customer information by account number
     * Used by IVR for customer lookup and greeting
     *
     * @param accountNumber Customer account number
     * @return Customer basic information
     */
    @Operation(summary = "Get customer by account number")
    @GetMapping("/{accountNumber}")
    public ResponseEntity<CustomerResponse> getCustomerByAccount(
            @PathVariable String accountNumber) {

        log.info("IVR Request: Get customer for account: {}", accountNumber);

        CustomerResponse customer = customerService.getCustomerByAccount(accountNumber);

        log.info("IVR Response: Customer found - ID: {}, Name: {}",
                customer.getCustomerId(), customer.getName());

        return ResponseEntity.ok(customer);
    }

    /**
     * Get customer information by phone number (ANI)
     * Used by IVR for automatic customer identification
     *
     * @param phoneNumber Customer phone number from ANI
     * @return Customer basic information
     */
    @Operation(summary = "Get customer by phone number (ANI)")
    @GetMapping("/by-phone/{phoneNumber}")
    public ResponseEntity<CustomerResponse> getCustomerByPhone(
            @PathVariable String phoneNumber) {

        log.info("IVR Request: Get customer by ANI: {}", phoneNumber);

        CustomerResponse customer = customerService.getCustomerByPhone(phoneNumber);

        return ResponseEntity.ok(customer);
    }
}

/**
 * DTO for Customer Response
 */
@Data
@Builder
public class CustomerResponse {
    private String customerId;
    private String name;
    private String accountNumber;
    private String accountType; // "Standard", "Premium", "VIP"
    private String preferredLanguage; // "EN", "AR"
    private Boolean isVip;
}
```

### Authentication Controller

```java
package com.bank.ivr.controller;

import com.bank.ivr.dto.*;
import com.bank.ivr.service.AuthService;
import io.swagger.v3.oas.annotations.Operation;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import jakarta.validation.Valid;

/**
 * Authentication Controller for IVR
 * Handles PIN validation and authentication
 */
@Slf4j
@RestController
@RequestMapping("/auth")
@RequiredArgsConstructor
public class AuthController {

    private the AuthService authService;

    /**
     * Validate customer PIN
     * Used by IVR for authentication
     *
     * @param request Authentication request with account number and PIN
     * @return Authentication result
     */
    @Operation(summary = "Validate customer PIN")
    @PostMapping("/validate-pin")
    public ResponseEntity<AuthResponse> validatePin(
            @Valid @RequestBody AuthRequest request) {

        log.info("IVR Request: Validate PIN for account: {}",
                request.getAccountNumber());

        AuthResponse response = authService.validatePin(
                request.getAccountNumber(),
                request.getPin()
        );

        log.info("IVR Response: PIN validation {} for account: {}",
                response.isValid() ? "PASSED" : "FAILED",
                request.getAccountNumber());

        return ResponseEntity.ok(response);
    }

    /**
     * Check if account is locked
     * Used by IVR before attempting authentication
     *
     * @param accountNumber Customer account number
     * @return Lock status
     */
    @Operation(summary = "Check account lock status")
    @GetMapping("/lock-status/{accountNumber}")
    public ResponseEntity<LockStatusResponse> getLockStatus(
            @PathVariable String accountNumber) {

        log.info("IVR Request: Check lock status for account: {}", accountNumber);

        LockStatusResponse response = authService.getLockStatus(accountNumber);

        return ResponseEntity.ok(response);
    }
}

/**
 * DTO for Authentication Request
 */
@Data
@Builder
public class AuthRequest {
    @NotBlank(message = "Account number is required")
    private String accountNumber;

    @NotBlank(message = "PIN is required")
    @Size(min = 4, max = 6, message = "PIN must be 4-6 digits")
    private String pin;
}

/**
 * DTO for Authentication Response
 */
@Data
@Builder
public class AuthResponse {
    private boolean valid;
    private String message;
    private Integer attemptsRemaining;
    private Boolean accountLocked;
    private String token; // JWT token for subsequent requests
}

/**
 * DTO for Lock Status Response
 */
@Data
@Builder
public class LockStatusResponse {
    private boolean locked;
    private Integer lockoutMinutesRemaining;
    private String message;
}
```

### Account Controller

```java
package com.bank.ivr.controller;

import com.bank.ivr.dto.*;
import com.bank.ivr.service.AccountService;
import io.swagger.v3.oas.annotations.Operation;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.http.ResponseEntity;
import org.springframework.security.access.prepost.PreAuthorize;
import org.springframework.web.bind.annotation.*;

import jakarta.validation.Valid;
import java.util.List;

/**
 * Account Controller for IVR
 * Handles balance inquiries and account information
 */
@Slf4j
@RestController
@RequestMapping("/accounts")
@RequiredArgsConstructor
public class AccountController {

    private final AccountService accountService;

    /**
     * Get account balance
     * Most frequently used IVR endpoint
     *
     * @param accountNumber Customer account number
     * @return Account balance
     */
    @Operation(summary = "Get account balance")
    @GetMapping("/{accountNumber}/balance")
    @PreAuthorize("hasRole('IVR_USER')")
    public ResponseEntity<BalanceResponse> getBalance(
            @PathVariable String accountNumber) {

        log.info("IVR Request: Get balance for account: {}", accountNumber);

        BalanceResponse balance = accountService.getBalance(accountNumber);

        log.info("IVR Response: Balance retrieved for account: {}", accountNumber);

        return ResponseEntity.ok(balance);
    }

    /**
     * Get recent transactions
     * Used by IVR for transaction history inquiry
     *
     * @param accountNumber Customer account number
     * @param limit Number of transactions to return (default: 5)
     * @return List of recent transactions
     */
    @Operation(summary = "Get recent transactions")
    @GetMapping("/{accountNumber}/transactions")
    @PreAuthorize("hasRole('IVR_USER')")
    public ResponseEntity<TransactionsResponse> getRecentTransactions(
            @PathVariable String accountNumber,
            @RequestParam(defaultValue = "5") int limit) {

        log.info("IVR Request: Get {} recent transactions for account: {}",
                limit, accountNumber);

        TransactionsResponse transactions = accountService.getRecentTransactions(
                accountNumber, limit);

        return ResponseEntity.ok(transactions);
    }
}

/**
 * DTO for Balance Response
 */
@Data
@Builder
public class BalanceResponse {
    private String accountNumber;
    private String accountType; // "Checking", "Savings"
    private Double balance;
    private Double availableBalance;
    private String currency; // "SAR", "USD"
}

/**
 * DTO for Transactions Response
 */
@Data
@Builder
public class TransactionsResponse {
    private String accountNumber;
    private List<TransactionItem> transactions;

    @Data
    @Builder
    public static class TransactionItem {
        private String transactionId;
        private String type; // "Debit", "Credit"
        private Double amount;
        private String description;
        private String date; // "2024-01-15"
    }
}
```

### Transfer Controller

```java
package com.bank.ivr.controller;

import com.bank.ivr.dto.*;
import com.bank.ivr.service.TransferService;
import io.swagger.v3.oas.annotations.Operation;
import jakarta.validation.Valid;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.http.ResponseEntity;
import org.springframework.security.access.prepost.PreAuthorize;
import org.springframework.web.bind.annotation.*;

/**
 * Transfer Controller for IVR
 * Handles fund transfers
 */
@Slf4j
@RestController
@RequestMapping("/transfers")
@RequiredArgsConstructor
public class TransferController {

    private final TransferService transferService;

    /**
     * Transfer funds between accounts
     * Used by IVR for money transfer
     *
     * @param request Transfer request
     * @return Transfer result
     */
    @Operation(summary = "Transfer funds")
    @PostMapping
    @PreAuthorize("hasRole('IVR_USER')")
    public ResponseEntity<TransferResponse> transferFunds(
            @Valid @RequestBody TransferRequest request) {

        log.info("IVR Request: Transfer {} from {} to {}",
                request.getAmount(),
                request.getSourceAccount(),
                request.getTargetAccount());

        TransferResponse response = transferService.transferFunds(request);

        log.info("IVR Response: Transfer {} - Transaction ID: {}",
                response.isSuccess() ? "SUCCESS" : "FAILED",
                response.getTransactionId());

        return ResponseEntity.ok(response);
    }

    /**
     * Validate transfer
     * Used by IVR to check if transfer is possible before execution
     *
     * @param request Transfer request
     * @return Validation result
     */
    @Operation(summary = "Validate transfer")
    @PostMapping("/validate")
    @PreAuthorize("hasRole('IVR_USER')")
    public ResponseEntity<TransferValidationResponse> validateTransfer(
            @Valid @RequestBody TransferRequest request) {

        log.info("IVR Request: Validate transfer from {} to {}",
                request.getSourceAccount(),
                request.getTargetAccount());

        TransferValidationResponse response =
                transferService.validateTransfer(request);

        return ResponseEntity.ok(response);
    }
}

/**
 * DTO for Transfer Request
 */
@Data
@Builder
public class TransferRequest {
    @NotBlank(message = "Source account is required")
    private String sourceAccount;

    @NotBlank(message = "Target account is required")
    private String targetAccount;

    @NotNull(message = "Amount is required")
    @Positive(message = "Amount must be positive")
    private Double amount;

    private String description;
}

/**
 * DTO for Transfer Response
 */
@Data
@Builder
public class TransferResponse {
    private boolean success;
    private String message;
    private String transactionId;
    private String timestamp;
}

/**
 * DTO for Transfer Validation Response
 */
@Data
@Builder
public class TransferValidationResponse {
    private boolean valid;
    private String message;
    private Double sourceBalance;
    private Double amountAfterTransfer;
}
```

---

## CVP HTTP Request Integration

### در CVP Studio

#### HTTP Request Element

```
┌────────────────────────────────────┐
│     HTTP Request Element           │
├────────────────────────────────────┤
│ URL: http://api.bank.com/api/ivr/  │
│      customers/{{accountNumber}}   │
│                                    │
│ Method: GET                        │
│                                    │
│ Headers:                           │
│   Authorization: Bearer {{token}}  │
│   Content-Type: application/json   │
│                                    │
│ Timeout: 5000ms                    │
│                                    │
│ Response Variable: customerData    │
└────────────────────────────────────┘
```

### VoiceXML HTTP Request

```xml
<?xml version="1.0"?>
<vxml version="2.1">
    <form id="getBalance">
        <var name="accountNumber" expr="'123456'"/>
        <var name="authToken" expr="session.token"/>

        <!-- HTTP Request to Spring Boot API -->
        <data name="balanceData"
              src="'http://api.bank.com/api/ivr/accounts/' + accountNumber + '/balance'"
              method="get"
              timeout="5s">
            <!-- Request Headers -->
            <property name="Authorization" value="'Bearer ' + authToken"/>
            <property name="Content-Type" value="'application/json'"/>
        </data>

        <block>
            <!-- Check if request successful -->
            <if cond="balanceData != null">
                <!-- Success: Play balance -->
                <prompt>
                    Your <value expr="balanceData.accountType"/> account balance is
                    <say-as type="currency">
                        <value expr="balanceData.balance"/>
                    </say-as>
                    <value expr="balanceData.currency"/>
                </prompt>
            <else/>
                <!-- Error: Handle failure -->
                <prompt>
                    I'm sorry, I couldn't retrieve your balance at this time.
                    Please try again or press 0 for an agent.
                </prompt>
                <goto next="#errorHandler"/>
            </if>
        </block>
    </form>
</vxml>
```

### POST Request Example (Transfer)

```xml
<?xml version="1.0"?>
<vxml version="2.1">
    <form id="transferMoney">
        <var name="sourceAccount" expr="'123456'"/>
        <var name="targetAccount" expr="'789012'"/>
        <var name="amount" expr="'500'"/>

        <!-- HTTP POST Request -->
        <data name="transferResult"
              src="'http://api.bank.com/api/ivr/transfers'"
              method="post"
              timeout="10s"
              enctype="application/json">

            <!-- Request Headers -->
            <property name="Authorization" value="'Bearer ' + session.token"/>
            <property name="Content-Type" value="'application/json'"/>

            <!-- Request Body -->
            <property name="body" value="{
                'sourceAccount': sourceAccount,
                'targetAccount': targetAccount,
                'amount': amount,
                'description': 'IVR Transfer'
            }"/>
        </data>

        <block>
            <if cond="transferResult.success == true">
                <!-- Success -->
                <prompt>
                    Transfer successful!
                    Your confirmation number is <value expr="transferResult.transactionId"/>
                    Would you like this sent by SMS? Press 1 for yes.
                </prompt>
            <else/>
                <!-- Failed -->
                <prompt>
                    Transfer failed. <value expr="transferResult.message"/>
                    Press 1 to try again, or 0 for an agent.
                </prompt>
            </if>
        </block>
    </form>
</vxml>
```

---

## Authentication & Security

### JWT Implementation

```java
package com.bank.ivr.util;

import io.jsonwebtoken.*;
import io.jsonwebtoken.security.Keys;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

import javax.crypto.SecretKey;
import java.util.Date;
import java.util.HashMap;
import java.util.Map;

/**
 * JWT Utility for IVR Authentication
 */
@Slf4j
@Component
public class JwtUtil {

    @Value("${security.jwt.secret-key}")
    private String secretKey;

    @Value("${security.jwt.expiration}")
    private long expiration;

    /**
     * Generate JWT token for IVR session
     */
    public String generateToken(String accountNumber, String customerId) {
        Map<String, Object> claims = new HashMap<>();
        claims.put("customerId", customerId);
        claims.put("source", "IVR");

        SecretKey key = Keys.hmacShaKeyFor(secretKey.getBytes());

        return Jwts.builder()
                .setClaims(claims)
                .setSubject(accountNumber)
                .setIssuedAt(new Date())
                .setExpiration(new Date(System.currentTimeMillis() + expiration))
                .signWith(key, SignatureAlgorithm.HS256)
                .compact();
    }

    /**
     * Validate JWT token
     */
    public boolean validateToken(String token) {
        try {
            SecretKey key = Keys.hmacShaKeyFor(secretKey.getBytes());
            Jwts.parserBuilder()
                    .setSigningKey(key)
                    .build()
                    .parseClaimsJws(token);
            return true;
        } catch (JwtException | IllegalArgumentException e) {
            log.error("Invalid JWT token: {}", e.getMessage());
            return false;
        }
    }

    /**
     * Extract account number from token
     */
    public String getAccountNumber(String token) {
        SecretKey key = Keys.hmacShaKeyFor(secretKey.getBytes());
        Claims claims = Jwts.parserBuilder()
                .setSigningKey(key)
                .build()
                .parseClaimsJws(token)
                .getBody();
        return claims.getSubject();
    }
}
```

### Security Configuration

```java
package com.bank.ivr.config;

import com.bank.ivr.security.JwtAuthenticationFilter;
import lombok.RequiredArgsConstructor;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.method.configuration.EnableMethodSecurity;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.http.SessionCreationPolicy;
import org.springframework.security.web.SecurityFilterChain;
import org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter;

/**
 * Security Configuration for IVR APIs
 */
@Configuration
@EnableWebSecurity
@EnableMethodSecurity
@RequiredArgsConstructor
public class SecurityConfig {

    private final JwtAuthenticationFilter jwtAuthFilter;

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
                .csrf(csrf -> csrf.disable())
                .cors(cors -> cors.configure(http))
                .authorizeHttpRequests(auth -> auth
                        // Public endpoints (no authentication)
                        .requestMatchers("/auth/**").permitAll()
                        .requestMatchers("/actuator/health").permitAll()
                        .requestMatchers("/swagger-ui/**", "/api-docs/**").permitAll()

                        // Protected endpoints (require authentication)
                        .requestMatchers("/customers/**").authenticated()
                        .requestMatchers("/accounts/**").authenticated()
                        .requestMatchers("/transfers/**").authenticated()

                        .anyRequest().authenticated()
                )
                .sessionManagement(session -> session
                        .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
                )
                .addFilterBefore(jwtAuthFilter, UsernamePasswordAuthenticationFilter.class);

        return http.build();
    }
}
```

---

## الخلاصة

تم إنشاء مجلد **Cisco IVR** شامل يحتوي على:

✅ **Cisco CVP** - شرح تفصيلي للمنصة (عربي + إنجليزي)
✅ **ICM Scripting** - دليل كامل للسكريبتنج (عربي + إنجليزي)
✅ **IVR Flow Design** - تصميم التدفقات (عربي)
✅ **Spring Boot Integration** - التكامل مع APIs (عربي)

المتبقي: سأكمل الملفات النهائية في الرسالة التالية.