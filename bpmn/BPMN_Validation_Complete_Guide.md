# BPMN Validation - الدليل الشامل للتحقق من صحة البيانات في BPMN

## جدول المحتويات
1. [مقدمة عن Validation في BPMN](#مقدمة-عن-validation-في-bpmn)
2. [مستويات Validation](#مستويات-validation)
3. [Validation في User Tasks](#validation-في-user-tasks)
4. [Validation باستخدام Script Tasks](#validation-باستخدام-script-tasks)
5. [Validation باستخدام Business Rule Tasks](#validation-باستخدام-business-rule-tasks)
6. [Validation باستخدام Service Tasks](#validation-باستخدام-service-tasks)
7. [Validation في Gateways](#validation-في-gateways)
8. [معالجة أخطاء Validation](#معالجة-أخطاء-validation)
9. [أمثلة عملية شاملة](#أمثلة-عملية-شاملة)
10. [أفضل الممارسات](#أفضل-الممارسات)

---

## مقدمة عن Validation في BPMN

### لماذا Validation في BPMN؟

**BPMN** يمثل عمليات تجارية تتعامل مع بيانات، وهذه البيانات يجب أن تكون:
- ✅ صحيحة
- ✅ كاملة
- ✅ متوافقة مع قواعد العمل
- ✅ آمنة

### أين يحدث Validation في BPMN؟

```
BPMN Validation Points
│
├── 1. في User Tasks (نماذج المستخدم)
│   └── Client-side + Server-side
│
├── 2. في Service Tasks (استدعاء APIs)
│   └── التحقق من البيانات قبل/بعد الاستدعاء
│
├── 3. في Script Tasks (تنفيذ كود)
│   └── Validation Logic مباشر
│
├── 4. في Business Rule Tasks (قواعد العمل)
│   └── DMN, Drools
│
├── 5. في Gateways (نقاط القرار)
│   └── التحقق من الشروط
│
└── 6. في Event Handlers (معالجة الأحداث)
    └── Error Events, Boundary Events
```

---

## مستويات Validation

### الطبقات الثلاث

```
┌──────────────────────────────────────┐
│ 1. Form Validation (نموذج المستخدم)  │  ← UX
│    - HTML5 validation                │
│    - JavaScript validation           │
│    - Camunda Form validation         │
├──────────────────────────────────────┤
│ 2. Process Validation (في BPMN)     │  ← Logic
│    - Script Tasks                    │
│    - Business Rule Tasks             │
│    - Service Tasks                   │
├──────────────────────────────────────┤
│ 3. Backend Validation (السيرفر)      │  ← Security
│    - Java/Spring Validators          │
│    - Database Constraints            │
└──────────────────────────────────────┘
```

---

## Validation في User Tasks

### 1. HTML5 Form Validation (Camunda Forms)

**مثال: نموذج طلب قرض**

```xml
<!-- BPMN User Task -->
<bpmn:userTask id="submitLoanApplication" name="تقديم طلب القرض">
  <bpmn:extensionElements>
    <camunda:formData>

      <!-- الاسم الكامل -->
      <camunda:formField
        id="fullName"
        label="الاسم الكامل"
        type="string">
        <camunda:validation>
          <camunda:constraint name="required"/>
          <camunda:constraint name="minlength" config="5"/>
          <camunda:constraint name="maxlength" config="100"/>
        </camunda:validation>
      </camunda:formField>

      <!-- البريد الإلكتروني -->
      <camunda:formField
        id="email"
        label="البريد الإلكتروني"
        type="string">
        <camunda:validation>
          <camunda:constraint name="required"/>
          <camunda:constraint name="validator" config="email"/>
        </camunda:validation>
      </camunda:formField>

      <!-- رقم الهوية -->
      <camunda:formField
        id="nationalId"
        label="رقم الهوية"
        type="string">
        <camunda:validation>
          <camunda:constraint name="required"/>
          <camunda:constraint name="pattern" config="^[12]\d{9}$"/>
        </camunda:validation>
      </camunda:formField>

      <!-- العمر -->
      <camunda:formField
        id="age"
        label="العمر"
        type="long">
        <camunda:validation>
          <camunda:constraint name="required"/>
          <camunda:constraint name="min" config="21"/>
          <camunda:constraint name="max" config="65"/>
        </camunda:validation>
      </camunda:formField>

      <!-- الراتب الشهري -->
      <camunda:formField
        id="monthlySalary"
        label="الراتب الشهري"
        type="long">
        <camunda:validation>
          <camunda:constraint name="required"/>
          <camunda:constraint name="min" config="5000"/>
        </camunda:validation>
      </camunda:formField>

      <!-- مبلغ القرض -->
      <camunda:formField
        id="loanAmount"
        label="مبلغ القرض المطلوب"
        type="long">
        <camunda:validation>
          <camunda:constraint name="required"/>
          <camunda:constraint name="min" config="10000"/>
          <camunda:constraint name="max" config="2000000"/>
        </camunda:validation>
      </camunda:formField>

      <!-- مدة القرض -->
      <camunda:formField
        id="loanDuration"
        label="مدة القرض (أشهر)"
        type="long">
        <camunda:validation>
          <camunda:constraint name="required"/>
          <camunda:constraint name="min" config="6"/>
          <camunda:constraint name="max" config="240"/>
        </camunda:validation>
      </camunda:formField>

      <!-- نوع الوظيفة -->
      <camunda:formField
        id="employmentType"
        label="نوع الوظيفة"
        type="enum">
        <camunda:validation>
          <camunda:constraint name="required"/>
        </camunda:validation>
        <camunda:value id="government" name="حكومي"/>
        <camunda:value id="private" name="قطاع خاص"/>
        <camunda:value id="self-employed" name="عمل حر"/>
      </camunda:formField>

      <!-- الموافقة على الشروط -->
      <camunda:formField
        id="termsAccepted"
        label="أوافق على الشروط والأحكام"
        type="boolean">
        <camunda:validation>
          <camunda:constraint name="required"/>
        </camunda:validation>
      </camunda:formField>

    </camunda:formData>
  </bpmn:extensionElements>
</bpmn:userTask>
```

---

### 2. Custom Validators في Camunda

**Java Custom Validator:**

```java
// ========================================
// Custom Validator: Saudi National ID
// ========================================

package com.example.validators;

import org.camunda.bpm.engine.impl.form.validator.FormFieldValidator;
import org.camunda.bpm.engine.impl.form.validator.FormFieldValidatorContext;

public class SaudiNationalIdValidator implements FormFieldValidator {

    @Override
    public boolean validate(Object submittedValue,
                           FormFieldValidatorContext validatorContext) {

        if (submittedValue == null) {
            return false;
        }

        String nationalId = submittedValue.toString();

        // التحقق من الطول
        if (nationalId.length() != 10) {
            return false;
        }

        // يجب أن يبدأ بـ 1 أو 2
        if (!nationalId.matches("^[12]\\d{9}$")) {
            return false;
        }

        // التحقق من checksum (إن وجد)
        // ... منطق إضافي

        return true;
    }
}

// ========================================
// تسجيل الـ Validator
// ========================================

@Configuration
public class ValidatorConfiguration {

    @Bean
    public static ProcessEnginePlugin customValidatorsPlugin() {
        return new ProcessEnginePlugin() {
            @Override
            public void preInit(ProcessEngineConfigurationImpl configuration) {
                Map<String, FormFieldValidator> validators = new HashMap<>();
                validators.put("saudiNationalId", new SaudiNationalIdValidator());
                validators.put("saudiPhone", new SaudiPhoneValidator());
                validators.put("loanAmountValidator", new LoanAmountValidator());

                configuration.setCustomFormFieldValidators(validators);
            }
        };
    }
}
```

**الاستخدام في BPMN:**
```xml
<camunda:formField id="nationalId" type="string">
  <camunda:validation>
    <camunda:constraint name="validator" config="saudiNationalId"/>
  </camunda:validation>
</camunda:formField>
```

---

### 3. Cross-field Validation (التحقق بين الحقول)

**مثال: التحقق من أن مبلغ القرض لا يتجاوز 30 ضعف الراتب**

```java
// ========================================
// Loan Amount Validator
// ========================================

public class LoanAmountValidator implements FormFieldValidator {

    @Override
    public boolean validate(Object submittedValue,
                           FormFieldValidatorContext validatorContext) {

        Long loanAmount = (Long) submittedValue;

        // الحصول على الراتب من الحقول الأخرى
        Map<String, Object> allFields = validatorContext.getSubmittedValues();
        Long monthlySalary = (Long) allFields.get("monthlySalary");

        if (monthlySalary == null) {
            return false;
        }

        // الحد الأقصى: 30 ضعف الراتب
        Long maxLoanAmount = monthlySalary * 30;

        if (loanAmount > maxLoanAmount) {
            validatorContext.addError(
                "loanAmount",
                "مبلغ القرض يجب أن لا يتجاوز " + maxLoanAmount + " ريال (30 ضعف راتبك)"
            );
            return false;
        }

        return true;
    }
}
```

---

## Validation باستخدام Script Tasks

### 1. JavaScript Validation

**مثال: التحقق من البيانات بعد إدخالها**

```xml
<bpmn:scriptTask id="validateLoanData"
                 name="التحقق من بيانات القرض"
                 scriptFormat="javascript">
  <bpmn:script><![CDATA[

    // جلب المتغيرات
    var fullName = execution.getVariable("fullName");
    var email = execution.getVariable("email");
    var age = execution.getVariable("age");
    var monthlySalary = execution.getVariable("monthlySalary");
    var loanAmount = execution.getVariable("loanAmount");
    var loanDuration = execution.getVariable("loanDuration");

    // متغير الأخطاء
    var errors = [];
    var isValid = true;

    // التحقق من الاسم
    if (!fullName || fullName.trim().length < 5) {
      errors.push("الاسم الكامل يجب أن يكون 5 أحرف على الأقل");
      isValid = false;
    }

    // التحقق من البريد الإلكتروني
    var emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    if (!email || !emailRegex.test(email)) {
      errors.push("البريد الإلكتروني غير صحيح");
      isValid = false;
    }

    // التحقق من العمر
    if (!age || age < 21 || age > 65) {
      errors.push("العمر يجب أن يكون بين 21 و 65 سنة");
      isValid = false;
    }

    // التحقق من الراتب
    if (!monthlySalary || monthlySalary < 5000) {
      errors.push("الراتب الشهري يجب أن يكون 5000 ريال على الأقل");
      isValid = false;
    }

    // التحقق من مبلغ القرض
    if (!loanAmount || loanAmount < 10000 || loanAmount > 2000000) {
      errors.push("مبلغ القرض يجب أن يكون بين 10,000 و 2,000,000 ريال");
      isValid = false;
    }

    // التحقق من أن القرض لا يتجاوز 30 ضعف الراتب
    if (loanAmount > (monthlySalary * 30)) {
      errors.push("مبلغ القرض يجب أن لا يتجاوز " + (monthlySalary * 30) + " ريال");
      isValid = false;
    }

    // التحقق من مدة القرض
    if (!loanDuration || loanDuration < 6 || loanDuration > 240) {
      errors.push("مدة القرض يجب أن تكون بين 6 و 240 شهر");
      isValid = false;
    }

    // حفظ النتائج
    execution.setVariable("validationErrors", errors);
    execution.setVariable("isDataValid", isValid);

    // Log
    print("Validation Result: " + isValid);
    if (!isValid) {
      print("Errors: " + errors.join(", "));
    }

  ]]></bpmn:script>
</bpmn:scriptTask>
```

**الاستخدام في العملية:**
```xml
[User Task: إدخال البيانات]
        ↓
[Script Task: التحقق من البيانات]
        ↓
     ⬦ X ⬦  البيانات صحيحة؟
     ↙         ↘
   نعم         لا
    ↓           ↓
[المتابعة]  [إعادة الإدخال]
```

---

### 2. Groovy Validation

```xml
<bpmn:scriptTask id="validateData" scriptFormat="groovy">
  <bpmn:script><![CDATA[

    def errors = []

    // التحقق من رقم الهوية السعودي
    def nationalId = execution.getVariable("nationalId")
    if (nationalId == null || !nationalId.matches(/^[12]\d{9}$/)) {
      errors << "رقم الهوية غير صحيح"
    }

    // التحقق من رقم الجوال السعودي
    def phone = execution.getVariable("phone")
    if (phone == null || !phone.matches(/^(\+966|00966|0)?5[0-9]{8}$/)) {
      errors << "رقم الجوال غير صحيح"
    }

    // التحقق من تاريخ الميلاد
    def birthdate = execution.getVariable("birthdate")
    if (birthdate != null) {
      def today = new Date()
      def age = today.year - birthdate.year

      if (age < 21) {
        errors << "يجب أن يكون عمرك 21 سنة على الأقل"
      }
      if (age > 65) {
        errors << "لا يمكن منح قرض لمن هم أكبر من 65 سنة"
      }
    }

    // حساب نسبة الالتزامات
    def monthlySalary = execution.getVariable("monthlySalary") ?: 0
    def currentDebts = execution.getVariable("currentDebts") ?: 0
    def loanAmount = execution.getVariable("loanAmount") ?: 0
    def loanDuration = execution.getVariable("loanDuration") ?: 1

    // حساب القسط الشهري (تقريبي)
    def interestRate = 0.05 / 12
    def monthlyPayment = loanAmount * (interestRate * Math.pow(1 + interestRate, loanDuration)) /
                        (Math.pow(1 + interestRate, loanDuration) - 1)

    def totalDebts = currentDebts + monthlyPayment
    def debtRatio = monthlySalary > 0 ? (totalDebts / monthlySalary) * 100 : 0

    execution.setVariable("monthlyPayment", monthlyPayment)
    execution.setVariable("debtRatio", debtRatio)

    // التحقق من نسبة الالتزامات
    if (debtRatio > 50) {
      errors << "نسبة الالتزامات عالية جداً (${debtRatio.round(2)}%)"
    }

    // حفظ النتائج
    execution.setVariable("validationErrors", errors)
    execution.setVariable("isValid", errors.isEmpty())

  ]]></bpmn:script>
</bpmn:scriptTask>
```

---

## Validation باستخدام Business Rule Tasks

### 1. DMN Decision Table

**ملف DMN: loan-validation.dmn**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<definitions xmlns="https://www.omg.org/spec/DMN/20191111/MODEL/"
             id="loan-validation"
             name="Loan Validation Rules">

  <decision id="validateLoan" name="التحقق من أهلية القرض">

    <decisionTable id="LoanValidationTable" hitPolicy="COLLECT">

      <!-- المدخلات -->
      <input id="input_age" label="العمر">
        <inputExpression typeRef="integer">
          <text>age</text>
        </inputExpression>
      </input>

      <input id="input_salary" label="الراتب الشهري">
        <inputExpression typeRef="integer">
          <text>monthlySalary</text>
        </inputExpression>
      </input>

      <input id="input_loanAmount" label="مبلغ القرض">
        <inputExpression typeRef="integer">
          <text>loanAmount</text>
        </inputExpression>
      </input>

      <input id="input_debtRatio" label="نسبة الالتزامات">
        <inputExpression typeRef="double">
          <text>debtRatio</text>
        </inputExpression>
      </input>

      <!-- المخرجات -->
      <output id="output_error" label="رسالة الخطأ" typeRef="string"/>

      <!-- القواعد -->

      <!-- قاعدة 1: العمر أقل من 21 -->
      <rule id="rule_age_min">
        <inputEntry>
          <text>&lt; 21</text>
        </inputEntry>
        <inputEntry>
          <text>-</text>
        </inputEntry>
        <inputEntry>
          <text>-</text>
        </inputEntry>
        <inputEntry>
          <text>-</text>
        </inputEntry>
        <outputEntry>
          <text>"يجب أن يكون عمرك 21 سنة على الأقل"</text>
        </outputEntry>
      </rule>

      <!-- قاعدة 2: العمر أكبر من 65 -->
      <rule id="rule_age_max">
        <inputEntry>
          <text>&gt; 65</text>
        </inputEntry>
        <inputEntry>
          <text>-</text>
        </inputEntry>
        <inputEntry>
          <text>-</text>
        </inputEntry>
        <inputEntry>
          <text>-</text>
        </inputEntry>
        <outputEntry>
          <text>"لا يمكن منح قرض لمن هم أكبر من 65 سنة"</text>
        </outputEntry>
      </rule>

      <!-- قاعدة 3: الراتب أقل من 5000 -->
      <rule id="rule_salary_min">
        <inputEntry>
          <text>-</text>
        </inputEntry>
        <inputEntry>
          <text>&lt; 5000</text>
        </inputEntry>
        <inputEntry>
          <text>-</text>
        </inputEntry>
        <inputEntry>
          <text>-</text>
        </inputEntry>
        <outputEntry>
          <text>"الراتب الشهري يجب أن يكون 5000 ريال على الأقل"</text>
        </outputEntry>
      </rule>

      <!-- قاعدة 4: مبلغ القرض أقل من الحد الأدنى -->
      <rule id="rule_loan_min">
        <inputEntry>
          <text>-</text>
        </inputEntry>
        <inputEntry>
          <text>-</text>
        </inputEntry>
        <inputEntry>
          <text>&lt; 10000</text>
        </inputEntry>
        <inputEntry>
          <text>-</text>
        </inputEntry>
        <outputEntry>
          <text>"الحد الأدنى للقرض 10,000 ريال"</text>
        </outputEntry>
      </rule>

      <!-- قاعدة 5: مبلغ القرض أكبر من الحد الأقصى -->
      <rule id="rule_loan_max">
        <inputEntry>
          <text>-</text>
        </inputEntry>
        <inputEntry>
          <text>-</text>
        </inputEntry>
        <inputEntry>
          <text>&gt; 2000000</text>
        </inputEntry>
        <inputEntry>
          <text>-</text>
        </inputEntry>
        <outputEntry>
          <text>"الحد الأقصى للقرض 2,000,000 ريال"</text>
        </outputEntry>
      </rule>

      <!-- قاعدة 6: نسبة الالتزامات عالية جداً -->
      <rule id="rule_debt_ratio_high">
        <inputEntry>
          <text>-</text>
        </inputEntry>
        <inputEntry>
          <text>-</text>
        </inputEntry>
        <inputEntry>
          <text>-</text>
        </inputEntry>
        <inputEntry>
          <text>&gt; 50</text>
        </inputEntry>
        <outputEntry>
          <text>"نسبة الالتزامات عالية جداً (أكثر من 50%)"</text>
        </outputEntry>
      </rule>

    </decisionTable>
  </decision>

</definitions>
```

---

**استخدام DMN في BPMN:**

```xml
<bpmn:businessRuleTask
    id="validateWithDMN"
    name="التحقق باستخدام DMN"
    camunda:resultVariable="validationErrors"
    camunda:decisionRef="validateLoan"
    camunda:mapDecisionResult="collectEntries">

  <bpmn:extensionElements>
    <camunda:inputOutput>
      <camunda:inputParameter name="age">${age}</camunda:inputParameter>
      <camunda:inputParameter name="monthlySalary">${monthlySalary}</camunda:inputParameter>
      <camunda:inputParameter name="loanAmount">${loanAmount}</camunda:inputParameter>
      <camunda:inputParameter name="debtRatio">${debtRatio}</camunda:inputParameter>
    </camunda:inputOutput>
  </bpmn:extensionElements>

</bpmn:businessRuleTask>
```

**معالجة النتيجة:**
```xml
[Business Rule Task: Validation]
        ↓
[Script Task: معالجة الأخطاء]
        ↓
     ⬦ X ⬦  يوجد أخطاء؟
     ↙         ↘
   نعم         لا
    ↓           ↓
[عرض الأخطاء] [المتابعة]
```

```javascript
// Script لمعالجة نتيجة DMN
var validationErrors = execution.getVariable("validationErrors");
var isValid = validationErrors == null || validationErrors.isEmpty();

execution.setVariable("isValid", isValid);

if (!isValid) {
  var errorMessages = [];
  for (var i = 0; i < validationErrors.size(); i++) {
    errorMessages.push(validationErrors.get(i));
  }
  execution.setVariable("errorMessages", errorMessages.join(", "));
}
```

---

### 2. Drools Validation

**ملف القواعد: loan-validation.drl**

```drools
package com.example.validation;

import com.example.model.LoanApplication;
import java.util.List;

global List<String> errors;

// قاعدة 1: التحقق من العمر
rule "Age Validation"
    when
        app : LoanApplication(age < 21 || age > 65)
    then
        errors.add("العمر يجب أن يكون بين 21 و 65 سنة");
end

// قاعدة 2: التحقق من الراتب
rule "Salary Validation"
    when
        app : LoanApplication(monthlySalary < 5000)
    then
        errors.add("الراتب الشهري يجب أن يكون 5000 ريال على الأقل");
end

// قاعدة 3: التحقق من مبلغ القرض
rule "Loan Amount Range Validation"
    when
        app : LoanApplication(loanAmount < 10000 || loanAmount > 2000000)
    then
        errors.add("مبلغ القرض يجب أن يكون بين 10,000 و 2,000,000 ريال");
end

// قاعدة 4: التحقق من نسبة القرض للراتب
rule "Loan to Salary Ratio Validation"
    when
        app : LoanApplication(loanAmount > (monthlySalary * 30))
    then
        errors.add("مبلغ القرض يجب أن لا يتجاوز 30 ضعف راتبك الشهري");
end

// قاعدة 5: التحقق من نسبة الالتزامات
rule "Debt Ratio Validation"
    when
        app : LoanApplication()
        eval(app.calculateDebtRatio() > 50)
    then
        errors.add("نسبة الالتزامات عالية جداً (أكثر من 50%)");
end

// قاعدة 6: التحقق من مدة العمل
rule "Employment Duration Validation"
    when
        app : LoanApplication(employmentDurationMonths < 6)
    then
        errors.add("مدة العمل يجب أن تكون 6 أشهر على الأقل");
end

// قاعدة 7: التحقق من البريد الإلكتروني
rule "Email Format Validation"
    when
        app : LoanApplication()
        eval(!app.getEmail().matches("^[^\\s@]+@[^\\s@]+\\.[^\\s@]+$"))
    then
        errors.add("البريد الإلكتروني غير صحيح");
end

// قاعدة 8: التحقق من رقم الهوية السعودي
rule "National ID Validation"
    when
        app : LoanApplication()
        eval(!app.getNationalId().matches("^[12]\\d{9}$"))
    then
        errors.add("رقم الهوية غير صحيح");
end
```

---

## Validation باستخدام Service Tasks

### 1. Java Validation Service

**Java Service:**

```java
// ========================================
// Validation Service
// ========================================

@Service
public class LoanValidationService implements JavaDelegate {

    @Autowired
    private UserRepository userRepository;

    @Autowired
    private CreditScoreService creditScoreService;

    @Override
    public void execute(DelegateExecution execution) throws Exception {

        List<String> errors = new ArrayList<>();

        // جلب البيانات
        String email = (String) execution.getVariable("email");
        String nationalId = (String) execution.getVariable("nationalId");
        Integer age = (Integer) execution.getVariable("age");
        Long monthlySalary = (Long) execution.getVariable("monthlySalary");
        Long loanAmount = (Long) execution.getVariable("loanAmount");

        // 1. التحقق من التفرد (Uniqueness)
        if (userRepository.existsByEmail(email)) {
            errors.add("البريد الإلكتروني مستخدم مسبقاً");
        }

        if (userRepository.existsByNationalId(nationalId)) {
            errors.add("رقم الهوية مسجل مسبقاً");
        }

        // 2. التحقق من الدرجة الائتمانية
        Integer creditScore = creditScoreService.getCreditScore(nationalId);
        execution.setVariable("creditScore", creditScore);

        if (creditScore == null) {
            errors.add("لا يمكن جلب السجل الائتماني");
        } else if (creditScore < 600) {
            errors.add("الدرجة الائتمانية منخفضة جداً (" + creditScore + ")");
        }

        // 3. التحقق من القائمة السوداء
        if (isBlacklisted(nationalId)) {
            errors.add("أنت في القائمة السوداء");
        }

        // 4. التحقق من القروض الحالية
        int currentLoansCount = getCurrentLoansCount(nationalId);
        if (currentLoansCount >= 3) {
            errors.add("لديك " + currentLoansCount + " قروض حالية (الحد الأقصى 3)");
        }

        // 5. Business Logic Validation
        if (age < 21) {
            errors.add("يجب أن يكون عمرك 21 سنة على الأقل");
        }

        if (loanAmount > monthlySalary * 30) {
            errors.add("مبلغ القرض يجب أن لا يتجاوز " + (monthlySalary * 30) + " ريال");
        }

        // حفظ النتائج
        execution.setVariable("validationErrors", errors);
        execution.setVariable("isValid", errors.isEmpty());

        if (!errors.isEmpty()) {
            execution.setVariable("errorMessage", String.join(", ", errors));
        }
    }

    private boolean isBlacklisted(String nationalId) {
        // استدعاء API للتحقق من القائمة السوداء
        return false; // مثال
    }

    private int getCurrentLoansCount(String nationalId) {
        // استدعاء API لجلب عدد القروض الحالية
        return 0; // مثال
    }
}
```

**استخدام في BPMN:**

```xml
<bpmn:serviceTask
    id="validateWithService"
    name="التحقق عبر Service"
    camunda:class="com.example.service.LoanValidationService"/>
```

---

### 2. REST API Validation

**استدعاء API خارجي للتحقق:**

```java
@Service
public class ExternalValidationService implements JavaDelegate {

    @Autowired
    private RestTemplate restTemplate;

    @Override
    public void execute(DelegateExecution execution) throws Exception {

        String nationalId = (String) execution.getVariable("nationalId");

        // استدعاء API خارجي للتحقق من الهوية
        try {
            String url = "https://api.absher.sa/validate-national-id";

            HttpHeaders headers = new HttpHeaders();
            headers.set("Authorization", "Bearer " + getApiToken());
            headers.setContentType(MediaType.APPLICATION_JSON);

            Map<String, String> requestBody = new HashMap<>();
            requestBody.put("nationalId", nationalId);

            HttpEntity<Map<String, String>> entity = new HttpEntity<>(requestBody, headers);

            ResponseEntity<ValidationResponse> response =
                restTemplate.postForEntity(url, entity, ValidationResponse.class);

            if (response.getStatusCode() == HttpStatus.OK) {
                ValidationResponse validationResult = response.getBody();

                execution.setVariable("idValidated", validationResult.isValid());
                execution.setVariable("fullNameFromAPI", validationResult.getFullName());

                if (!validationResult.isValid()) {
                    execution.setVariable("validationError", "رقم الهوية غير صحيح");
                }
            } else {
                execution.setVariable("idValidated", false);
                execution.setVariable("validationError", "خطأ في التحقق من الهوية");
            }

        } catch (Exception e) {
            execution.setVariable("idValidated", false);
            execution.setVariable("validationError", "فشل الاتصال بخدمة التحقق");
        }
    }

    private String getApiToken() {
        // جلب API Token
        return "your-api-token";
    }
}
```

---

## Validation في Gateways

### 1. Conditional Sequence Flow

**التحقق من الشروط في التدفق:**

```xml
<bpmn:sequenceFlow
    id="flow_approved"
    sourceRef="decision_gateway"
    targetRef="approve_task">
  <bpmn:conditionExpression xsi:type="tFormalExpression">
    ${isValid == true &amp;&amp; creditScore >= 700}
  </bpmn:conditionExpression>
</bpmn:sequenceFlow>

<bpmn:sequenceFlow
    id="flow_manual_review"
    sourceRef="decision_gateway"
    targetRef="manual_review_task">
  <bpmn:conditionExpression xsi:type="tFormalExpression">
    ${isValid == true &amp;&amp; creditScore >= 600 &amp;&amp; creditScore &lt; 700}
  </bpmn:conditionExpression>
</bpmn:sequenceFlow>

<bpmn:sequenceFlow
    id="flow_rejected"
    sourceRef="decision_gateway"
    targetRef="reject_task">
  <bpmn:conditionExpression xsi:type="tFormalExpression">
    ${isValid == false || creditScore &lt; 600}
  </bpmn:conditionExpression>
</bpmn:sequenceFlow>
```

---

### 2. Exclusive Gateway مع Validation

```xml
[User Task: إدخال البيانات]
        ↓
[Service Task: التحقق من البيانات]
        ↓
     ⬦ X ⬦  isValid?
     ↙ ↓ ↘
   false  true
    ↓      ↓
[إعادة  [متابعة
 الإدخال] المعالجة]
    ↓
    ⬦ X ⬦  عدد المحاولات < 3?
    ↙         ↘
  نعم         لا
   ↓           ↓
[العودة]    [رفض تلقائي]
```

**BPMN XML:**

```xml
<!-- Gateway: التحقق من صحة البيانات -->
<bpmn:exclusiveGateway id="validation_gateway" name="البيانات صحيحة؟"/>

<!-- Flow: بيانات صحيحة -->
<bpmn:sequenceFlow
    id="flow_valid"
    sourceRef="validation_gateway"
    targetRef="continue_process">
  <bpmn:conditionExpression xsi:type="tFormalExpression">
    ${isValid == true}
  </bpmn:conditionExpression>
</bpmn:sequenceFlow>

<!-- Flow: بيانات خاطئة -->
<bpmn:sequenceFlow
    id="flow_invalid"
    sourceRef="validation_gateway"
    targetRef="retry_input">
  <bpmn:conditionExpression xsi:type="tFormalExpression">
    ${isValid == false &amp;&amp; retryCount &lt; 3}
  </bpmn:conditionExpression>
</bpmn:sequenceFlow>

<!-- Flow: تجاوز الحد الأقصى للمحاولات -->
<bpmn:sequenceFlow
    id="flow_max_retries"
    sourceRef="validation_gateway"
    targetRef="auto_reject">
  <bpmn:conditionExpression xsi:type="tFormalExpression">
    ${isValid == false &amp;&amp; retryCount >= 3}
  </bpmn:conditionExpression>
</bpmn:sequenceFlow>
```

---

## معالجة أخطاء Validation

### 1. Error Boundary Event

**معالجة أخطاء Validation باستخدام Error Events:**

```xml
<!-- Service Task مع Error Boundary -->
<bpmn:serviceTask
    id="validateData"
    name="التحقق من البيانات"
    camunda:class="com.example.ValidationService">

  <!-- Error Boundary Event -->
  <bpmn:boundaryEvent
      id="validation_error"
      name="خطأ في التحقق"
      attachedToRef="validateData">
    <bpmn:errorEventDefinition errorRef="ValidationError"/>
  </bpmn:boundaryEvent>

</bpmn:serviceTask>

<!-- معالجة الخطأ -->
<bpmn:sequenceFlow sourceRef="validation_error" targetRef="handle_error"/>

<bpmn:userTask id="handle_error" name="عرض أخطاء التحقق"/>
```

**Java Service يرمي Error:**

```java
@Service
public class ValidationService implements JavaDelegate {

    @Override
    public void execute(DelegateExecution execution) throws Exception {

        List<String> errors = validateData(execution);

        if (!errors.isEmpty()) {
            // حفظ الأخطاء
            execution.setVariable("validationErrors", errors);

            // رمي Error Event
            throw new BpmnError("ValidationError", String.join(", ", errors));
        }

        // البيانات صحيحة
        execution.setVariable("isValid", true);
    }

    private List<String> validateData(DelegateExecution execution) {
        // منطق التحقق
        List<String> errors = new ArrayList<>();
        // ...
        return errors;
    }
}
```

---

### 2. Event Sub-Process للتحقق

```xml
<!-- العملية الرئيسية -->
<bpmn:process id="loanProcess">

  <bpmn:startEvent id="start"/>

  <bpmn:userTask id="inputData" name="إدخال البيانات"/>

  <bpmn:serviceTask id="processData" name="معالجة البيانات"/>

  <bpmn:endEvent id="end"/>

  <!-- Event Sub-Process للتحقق -->
  <bpmn:subProcess
      id="validationSubProcess"
      triggeredByEvent="true">

    <!-- Error Start Event -->
    <bpmn:startEvent
        id="validationErrorStart"
        name="خطأ في التحقق">
      <bpmn:errorEventDefinition errorRef="ValidationError"/>
    </bpmn:startEvent>

    <!-- معالجة الخطأ -->
    <bpmn:userTask
        id="showErrors"
        name="عرض الأخطاء"
        camunda:formKey="error-form"/>

    <bpmn:exclusiveGateway id="retry_gateway" name="إعادة المحاولة؟"/>

    <!-- إعادة المحاولة -->
    <bpmn:sequenceFlow
        sourceRef="retry_gateway"
        targetRef="compensate_back">
      <bpmn:conditionExpression>
        ${retryRequested == true}
      </bpmn:conditionExpression>
    </bpmn:sequenceFlow>

    <!-- إلغاء العملية -->
    <bpmn:endEvent
        id="cancel_end"
        name="إلغاء">
      <bpmn:terminateEventDefinition/>
    </bpmn:endEvent>

  </bpmn:subProcess>

</bpmn:process>
```

---

## أمثلة عملية شاملة

### مثال كامل: عملية طلب قرض مع Validation شامل

```xml
<?xml version="1.0" encoding="UTF-8"?>
<bpmn:definitions xmlns:bpmn="http://www.omg.org/spec/BPMN/20100524/MODEL"
                  xmlns:camunda="http://camunda.org/schema/1.0/bpmn">

  <bpmn:process id="loan_application_with_validation" isExecutable="true">

    <!-- ============================================ -->
    <!-- البداية -->
    <!-- ============================================ -->
    <bpmn:startEvent id="start" name="بدء طلب قرض"/>

    <bpmn:sequenceFlow sourceRef="start" targetRef="input_data"/>

    <!-- ============================================ -->
    <!-- المرحلة 1: إدخال البيانات مع Validation -->
    <!-- ============================================ -->
    <bpmn:userTask id="input_data" name="إدخال بيانات الطلب">
      <bpmn:extensionElements>
        <camunda:formData>

          <!-- البيانات الشخصية -->
          <camunda:formField id="fullName" label="الاسم الكامل" type="string">
            <camunda:validation>
              <camunda:constraint name="required"/>
              <camunda:constraint name="minlength" config="5"/>
            </camunda:validation>
          </camunda:formField>

          <camunda:formField id="email" label="البريد الإلكتروني" type="string">
            <camunda:validation>
              <camunda:constraint name="required"/>
              <camunda:constraint name="validator" config="email"/>
            </camunda:validation>
          </camunda:formField>

          <camunda:formField id="nationalId" label="رقم الهوية" type="string">
            <camunda:validation>
              <camunda:constraint name="required"/>
              <camunda:constraint name="validator" config="saudiNationalId"/>
            </camunda:validation>
          </camunda:formField>

          <camunda:formField id="phone" label="رقم الجوال" type="string">
            <camunda:validation>
              <camunda:constraint name="required"/>
              <camunda:constraint name="validator" config="saudiPhone"/>
            </camunda:validation>
          </camunda:formField>

          <camunda:formField id="age" label="العمر" type="long">
            <camunda:validation>
              <camunda:constraint name="required"/>
              <camunda:constraint name="min" config="21"/>
              <camunda:constraint name="max" config="65"/>
            </camunda:validation>
          </camunda:formField>

          <!-- البيانات المالية -->
          <camunda:formField id="monthlySalary" label="الراتب الشهري" type="long">
            <camunda:validation>
              <camunda:constraint name="required"/>
              <camunda:constraint name="min" config="5000"/>
            </camunda:validation>
          </camunda:formField>

          <camunda:formField id="currentDebts" label="الالتزامات الحالية" type="long">
            <camunda:validation>
              <camunda:constraint name="required"/>
              <camunda:constraint name="min" config="0"/>
            </camunda:validation>
          </camunda:formField>

          <!-- بيانات القرض -->
          <camunda:formField id="loanAmount" label="مبلغ القرض" type="long">
            <camunda:validation>
              <camunda:constraint name="required"/>
              <camunda:constraint name="min" config="10000"/>
              <camunda:constraint name="max" config="2000000"/>
              <camunda:constraint name="validator" config="loanAmountValidator"/>
            </camunda:validation>
          </camunda:formField>

          <camunda:formField id="loanDuration" label="مدة القرض (أشهر)" type="long">
            <camunda:validation>
              <camunda:constraint name="required"/>
              <camunda:constraint name="min" config="6"/>
              <camunda:constraint name="max" config="240"/>
            </camunda:validation>
          </camunda:formField>

        </camunda:formData>
      </bpmn:extensionElements>
    </bpmn:userTask>

    <bpmn:sequenceFlow sourceRef="input_data" targetRef="parallel_validation"/>

    <!-- ============================================ -->
    <!-- المرحلة 2: Validations متوازية -->
    <!-- ============================================ -->
    <bpmn:parallelGateway id="parallel_validation" name="فحوصات متوازية"/>

    <!-- الفحص 1: Script Validation -->
    <bpmn:sequenceFlow sourceRef="parallel_validation" targetRef="script_validation"/>

    <bpmn:scriptTask
        id="script_validation"
        name="التحقق الأساسي (JavaScript)"
        scriptFormat="javascript">
      <bpmn:script><![CDATA[
        var errors = [];

        var monthlySalary = execution.getVariable("monthlySalary");
        var loanAmount = execution.getVariable("loanAmount");
        var currentDebts = execution.getVariable("currentDebts");

        // التحقق من نسبة القرض للراتب
        if (loanAmount > monthlySalary * 30) {
          errors.push("مبلغ القرض يجب أن لا يتجاوز " + (monthlySalary * 30) + " ريال");
        }

        execution.setVariable("scriptErrors", errors);
      ]]></bpmn:script>
    </bpmn:scriptTask>

    <!-- الفحص 2: Business Rule Validation -->
    <bpmn:sequenceFlow sourceRef="parallel_validation" targetRef="dmn_validation"/>

    <bpmn:businessRuleTask
        id="dmn_validation"
        name="التحقق بالقواعد (DMN)"
        camunda:resultVariable="dmnErrors"
        camunda:decisionRef="loan_validation_dmn"
        camunda:mapDecisionResult="collectEntries"/>

    <!-- الفحص 3: Service Validation -->
    <bpmn:sequenceFlow sourceRef="parallel_validation" targetRef="service_validation"/>

    <bpmn:serviceTask
        id="service_validation"
        name="التحقق الخارجي (Service)"
        camunda:class="com.example.LoanValidationService"/>

    <!-- ============================================ -->
    <!-- دمج نتائج الفحوصات -->
    <!-- ============================================ -->
    <bpmn:sequenceFlow sourceRef="script_validation" targetRef="merge_validations"/>
    <bpmn:sequenceFlow sourceRef="dmn_validation" targetRef="merge_validations"/>
    <bpmn:sequenceFlow sourceRef="service_validation" targetRef="merge_validations"/>

    <bpmn:parallelGateway id="merge_validations" name="دمج النتائج"/>

    <bpmn:sequenceFlow sourceRef="merge_validations" targetRef="aggregate_errors"/>

    <!-- ============================================ -->
    <!-- تجميع جميع الأخطاء -->
    <!-- ============================================ -->
    <bpmn:scriptTask
        id="aggregate_errors"
        name="تجميع الأخطاء"
        scriptFormat="javascript">
      <bpmn:script><![CDATA[
        var allErrors = [];

        var scriptErrors = execution.getVariable("scriptErrors") || [];
        var dmnErrors = execution.getVariable("dmnErrors") || [];
        var serviceErrors = execution.getVariable("serviceErrors") || [];

        allErrors = allErrors.concat(scriptErrors);
        allErrors = allErrors.concat(dmnErrors);
        allErrors = allErrors.concat(serviceErrors);

        execution.setVariable("allValidationErrors", allErrors);
        execution.setVariable("isValid", allErrors.length === 0);
      ]]></bpmn:script>
    </bpmn:scriptTask>

    <bpmn:sequenceFlow sourceRef="aggregate_errors" targetRef="validation_gateway"/>

    <!-- ============================================ -->
    <!-- Gateway: التحقق من النتيجة -->
    <!-- ============================================ -->
    <bpmn:exclusiveGateway id="validation_gateway" name="البيانات صحيحة؟"/>

    <!-- المسار 1: بيانات صحيحة -->
    <bpmn:sequenceFlow sourceRef="validation_gateway" targetRef="calculate_eligibility">
      <bpmn:conditionExpression xsi:type="tFormalExpression">
        ${isValid == true}
      </bpmn:conditionExpression>
    </bpmn:sequenceFlow>

    <!-- المسار 2: بيانات خاطئة -->
    <bpmn:sequenceFlow sourceRef="validation_gateway" targetRef="show_errors">
      <bpmn:conditionExpression xsi:type="tFormalExpression">
        ${isValid == false}
      </bpmn:conditionExpression>
    </bpmn:sequenceFlow>

    <!-- ============================================ -->
    <!-- عرض الأخطاء -->
    <!-- ============================================ -->
    <bpmn:userTask id="show_errors" name="عرض الأخطاء">
      <bpmn:extensionElements>
        <camunda:inputOutput>
          <camunda:inputParameter name="errors">${allValidationErrors}</camunda:inputParameter>
        </camunda:inputOutput>
      </bpmn:extensionElements>
    </bpmn:userTask>

    <bpmn:sequenceFlow sourceRef="show_errors" targetRef="retry_gateway"/>

    <bpmn:exclusiveGateway id="retry_gateway" name="إعادة المحاولة؟"/>

    <!-- إعادة المحاولة -->
    <bpmn:sequenceFlow sourceRef="retry_gateway" targetRef="input_data">
      <bpmn:conditionExpression xsi:type="tFormalExpression">
        ${retryRequested == true}
      </bpmn:conditionExpression>
    </bpmn:sequenceFlow>

    <!-- إلغاء -->
    <bpmn:sequenceFlow sourceRef="retry_gateway" targetRef="end_cancelled">
      <bpmn:conditionExpression xsi:type="tFormalExpression">
        ${retryRequested == false}
      </bpmn:conditionExpression>
    </bpmn:sequenceFlow>

    <bpmn:endEvent id="end_cancelled" name="ملغي"/>

    <!-- ============================================ -->
    <!-- المتابعة: حساب الأهلية -->
    <!-- ============================================ -->
    <bpmn:businessRuleTask
        id="calculate_eligibility"
        name="حساب الأهلية"
        camunda:resultVariable="eligibilityResult"
        camunda:decisionRef="loan_eligibility_dmn"/>

    <bpmn:sequenceFlow sourceRef="calculate_eligibility" targetRef="end_success"/>

    <bpmn:endEvent id="end_success" name="تم بنجاح"/>

  </bpmn:process>

</bpmn:definitions>
```

---

## أفضل الممارسات

### 1. Validate على كل المستويات

```
✅ الطبقات الثلاث:

1. Form Validation (UX)
   → ردود فعل فورية

2. Process Validation (Logic)
   → قواعد عمل معقدة

3. Backend Validation (Security)
   → التحقق النهائي
```

---

### 2. استخدام النوع المناسب من Validation

```
┌──────────────────────────────────────┐
│ Form Validation                      │  → بسيط، سريع
│ ✅ Required, Min, Max, Pattern       │
├──────────────────────────────────────┤
│ Script Task                          │  → منطق بسيط
│ ✅ حسابات، تحويلات                   │
├──────────────────────────────────────┤
│ Business Rule Task                   │  → قواعد معقدة
│ ✅ DMN, Drools                       │
├──────────────────────────────────────┤
│ Service Task                         │  → تحقق خارجي
│ ✅ Database, APIs                    │
└──────────────────────────────────────┘
```

---

### 3. معالجة الأخطاء بشكل واضح

```javascript
// ❌ سيء
throw new Error("Invalid data");

// ✅ جيد
var errors = [];
errors.push("الاسم يجب أن يكون 5 أحرف على الأقل");
errors.push("البريد الإلكتروني غير صحيح");
execution.setVariable("validationErrors", errors);
```

---

### 4. Validation متوازي للسرعة

```
❌ بطيء (تسلسلي):
[Validation 1] → [Validation 2] → [Validation 3]
(30 ثانية)

✅ سريع (متوازي):
         ⬦ + ⬦
         ↙ ↓ ↘
[Validation 1] [2] [3]
         ↓ ↓ ↓
         ⬦ + ⬦
(10 ثواني)
```

---

### 5. إعادة استخدام القواعد

```
✅ استخدم:
- DMN Tables → يمكن إعادة استخدامها
- Drools Rules → مشتركة بين العمليات
- Custom Validators → في أكثر من Form

❌ تجنب:
- تكرار نفس القواعد في كل عملية
```

---

### 6. Logging و Auditing

```java
@Override
public void execute(DelegateExecution execution) {
    logger.info("Starting validation for process: {}",
                execution.getProcessInstanceId());

    List<String> errors = validate(execution);

    if (!errors.isEmpty()) {
        logger.warn("Validation failed: {}", errors);
        auditService.log("VALIDATION_FAILED",
                        execution.getProcessInstanceId(),
                        errors);
    } else {
        logger.info("Validation passed");
        auditService.log("VALIDATION_PASSED",
                        execution.getProcessInstanceId());
    }
}
```

---

## الخلاصة

### استراتيجية Validation الشاملة في BPMN:

```
1. Form Validation (في User Tasks)
   ✅ HTML5 constraints
   ✅ Custom validators
   ✅ Cross-field validation

2. Process Validation (في BPMN)
   ✅ Script Tasks → منطق بسيط
   ✅ Business Rule Tasks → قواعد معقدة
   ✅ Service Tasks → تحقق خارجي

3. Error Handling
   ✅ Error Boundary Events
   ✅ Event Sub-Processes
   ✅ Retry mechanisms

4. Best Practices
   ✅ Validate على كل المستويات
   ✅ معالجة أخطاء واضحة
   ✅ Parallel validations
   ✅ إعادة استخدام القواعد
```

---

**Validation في BPMN = أمان + جودة بيانات + تجربة مستخدم ممتازة!**

التحقق الجيد من البيانات في كل مرحلة من العملية يضمن:
- 🛡️ أمان النظام
- ✅ صحة البيانات
- 📊 قرارات صحيحة
- 😊 رضا المستخدمين

---

**تم إنشاء هذا الدليل كمرجع شامل لـ Validation في BPMN**
