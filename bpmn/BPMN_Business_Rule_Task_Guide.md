# Business Rule Task - الدليل الشامل

## جدول المحتويات
1. [ما هو Business Rule Task؟](#ما-هو-business-rule-task)
2. [لماذا نستخدمه؟](#لماذا-نستخدمه)
3. [مكونات Business Rules](#مكونات-business-rules)
4. [DMN - Decision Model and Notation](#dmn---decision-model-and-notation)
5. [محركات القواعد](#محركات-القواعد)
6. [أمثلة عملية شاملة](#أمثلة-عملية-شاملة)
7. [المقارنة مع Script Task](#المقارنة-مع-script-task)
8. [أفضل الممارسات](#أفضل-الممارسات)

---

## ما هو Business Rule Task؟

### التعريف

**Business Rule Task** هو نشاط في BPMN يقوم بتنفيذ قواعد عمل (Business Rules) معقدة باستخدام محرك قواعد متخصص.

### الشكل في BPMN

```
┌──────────────────────────┐
│ 📋                       │  ← أيقونة جدول
│  حساب خصم العميل         │
│                          │
│ محرك: Drools            │
│ القواعد: DiscountRules   │
└──────────────────────────┘
```

### الفكرة الأساسية

```
بدلاً من:
IF age >= 18 THEN eligible = true
ELSE eligible = false

نستخدم:
┌─────────────────────────┐
│ Business Rule Task      │
│                         │
│ قواعد الأهلية           │
│ (ملف منفصل قابل للتعديل) │
└─────────────────────────┘
```

---

## لماذا نستخدمه؟

### المشكلة التقليدية

#### كود مبعثر (Anti-pattern)

```javascript
// في Service Task أو Script Task
function calculateDiscount(customer, order) {
  let discount = 0;

  // قاعدة 1: العملاء VIP
  if (customer.type === 'VIP' && customer.loyaltyYears >= 5) {
    discount = 20;
  } else if (customer.type === 'VIP' && customer.loyaltyYears >= 2) {
    discount = 15;
  } else if (customer.type === 'VIP') {
    discount = 10;
  }

  // قاعدة 2: الطلبات الكبيرة
  if (order.amount > 10000) {
    discount += 10;
  } else if (order.amount > 5000) {
    discount += 5;
  }

  // قاعدة 3: العروض الموسمية
  const today = new Date();
  if (today.getMonth() === 11) { // ديسمبر
    discount += 5;
  }

  // قاعدة 4: الحد الأقصى
  if (discount > 30) {
    discount = 30;
  }

  return discount;
}
```

**المشاكل**:
- ❌ القواعد مدفونة في الكود
- ❌ صعبة التعديل (تحتاج مبرمج)
- ❌ صعبة الفهم لغير المبرمجين
- ❌ صعبة الاختبار
- ❌ صعبة الصيانة

---

### الحل: Business Rule Task

#### فصل القواعد عن الكود

```
┌─────────────────────────────────────┐
│ Business Process (BPMN)             │
│                                     │
│  [استلام الطلب]                     │
│         ↓                           │
│  ┌─────────────────────┐            │
│  │ 📋 Business Rule    │            │
│  │  حساب الخصم         │ ──→ يقرأ من ملف القواعد
│  └─────────────────────┘            │
│         ↓                           │
│  [تطبيق الخصم]                      │
└─────────────────────────────────────┘
                │
                ↓
    ┌───────────────────────┐
    │ Discount Rules File   │
    │ (DMN, Drools, etc)    │
    │                       │
    │ قواعد يمكن تعديلها    │
    │ بدون برمجة!          │
    └───────────────────────┘
```

---

## مكونات Business Rules

### 1. Business Rule Engine (محرك القواعد)

```
Business Rule Engines
│
├── Drools (JBoss)
│   ├── Java-based
│   ├── Open Source
│   ├── يدعم DRL (Drools Rule Language)
│   └── الأكثر شيوعاً
│
├── DMN (Decision Model and Notation)
│   ├── معيار OMG
│   ├── جداول قرار
│   ├── سهل للمحللين
│   └── يدعمه Camunda, Flowable
│
├── Easy Rules
│   ├── بسيط جداً
│   ├── Java
│   └── للقواعد البسيطة
│
├── OpenRules
│   ├── Excel-based
│   ├── سهل للمستخدمين
│   └── تجاري
│
└── IBM ODM (Operational Decision Manager)
    ├── Enterprise
    ├── قوي جداً
    └── مكلف
```

---

### 2. مكونات القاعدة (Rule Components)

```
Business Rule
│
├── Conditions (الشروط)
│   ├── WHEN
│   ├── IF
│   └── متى تُطبق القاعدة؟
│
├── Actions (الإجراءات)
│   ├── THEN
│   ├── DO
│   └── ماذا تفعل عند تحقق الشرط؟
│
├── Priority (الأولوية)
│   └── ترتيب التنفيذ
│
└── Metadata (البيانات الوصفية)
    ├── الاسم
    ├── الوصف
    └── الفئة
```

---

## DMN - Decision Model and Notation

### ما هو DMN؟

**DMN** = Decision Model and Notation
- معيار من OMG (نفس منظمة BPMN)
- لوصف القرارات بشكل مرئي
- جداول قرار سهلة الفهم
- يدمج مع BPMN بسلاسة

---

### مثال DMN: حساب خصم العميل

#### Decision Table (جدول القرار)

```
╔════════════════════════════════════════════════════════════════════╗
║              جدول قرار: حساب خصم العميل                            ║
╠════════════════════════════════════════════════════════════════════╣
║  Input 1      │  Input 2        │  Input 3      │  Output        ║
║  نوع العميل   │  سنوات الولاء   │  قيمة الطلب   │  نسبة الخصم    ║
╠═══════════════╪═════════════════╪═══════════════╪════════════════╣
║  "VIP"        │  >= 5           │  -            │  20%           ║
║  "VIP"        │  >= 2           │  -            │  15%           ║
║  "VIP"        │  >= 1           │  -            │  10%           ║
║  "Gold"       │  >= 3           │  -            │  12%           ║
║  "Gold"       │  >= 1           │  -            │  8%            ║
║  "Silver"     │  -              │  > 5000       │  7%            ║
║  "Silver"     │  -              │  > 1000       │  5%            ║
║  "Regular"    │  -              │  > 10000      │  10%           ║
║  "Regular"    │  -              │  > 5000       │  5%            ║
║  "Regular"    │  -              │  -            │  0%            ║
╚═══════════════╧═════════════════╧═══════════════╧════════════════╝

Hit Policy: First (أول قاعدة تتطابق)
```

---

#### DMN XML

```xml
<?xml version="1.0" encoding="UTF-8"?>
<definitions xmlns="https://www.omg.org/spec/DMN/20191111/MODEL/"
             id="customer-discount"
             name="Customer Discount Decision"
             namespace="http://camunda.org/schema/1.0/dmn">

  <decision id="discount_decision" name="حساب خصم العميل">

    <decisionTable id="DecisionTable_Discount" hitPolicy="FIRST">

      <!-- المدخلات -->
      <input id="input_customerType" label="نوع العميل">
        <inputExpression id="inputExpression_customerType" typeRef="string">
          <text>customerType</text>
        </inputExpression>
      </input>

      <input id="input_loyaltyYears" label="سنوات الولاء">
        <inputExpression id="inputExpression_loyaltyYears" typeRef="integer">
          <text>loyaltyYears</text>
        </inputExpression>
      </input>

      <input id="input_orderAmount" label="قيمة الطلب">
        <inputExpression id="inputExpression_orderAmount" typeRef="double">
          <text>orderAmount</text>
        </inputExpression>
      </input>

      <!-- المخرجات -->
      <output id="output_discount" label="نسبة الخصم" typeRef="integer"/>

      <!-- القواعد -->
      <rule id="rule_1">
        <inputEntry id="input_1_1">
          <text>"VIP"</text>
        </inputEntry>
        <inputEntry id="input_1_2">
          <text>&gt;= 5</text>
        </inputEntry>
        <inputEntry id="input_1_3">
          <text>-</text>
        </inputEntry>
        <outputEntry id="output_1">
          <text>20</text>
        </outputEntry>
      </rule>

      <rule id="rule_2">
        <inputEntry id="input_2_1">
          <text>"VIP"</text>
        </inputEntry>
        <inputEntry id="input_2_2">
          <text>&gt;= 2</text>
        </inputEntry>
        <inputEntry id="input_2_3">
          <text>-</text>
        </inputEntry>
        <outputEntry id="output_2">
          <text>15</text>
        </outputEntry>
      </rule>

      <!-- ... باقي القواعد -->

      <rule id="rule_default">
        <inputEntry id="input_default_1">
          <text>-</text>
        </inputEntry>
        <inputEntry id="input_default_2">
          <text>-</text>
        </inputEntry>
        <inputEntry id="input_default_3">
          <text>-</text>
        </inputEntry>
        <outputEntry id="output_default">
          <text>0</text>
        </outputEntry>
      </rule>

    </decisionTable>
  </decision>

</definitions>
```

---

### DMN Hit Policies (سياسات التطابق)

```
Hit Policy: كيف نتعامل مع القواعد المتطابقة؟
│
├── UNIQUE (U)
│   └── قاعدة واحدة فقط يمكن أن تتطابق
│
├── FIRST (F)
│   └── أول قاعدة تتطابق تُستخدم
│
├── PRIORITY (P)
│   └── القاعدة ذات الأولوية الأعلى
│
├── ANY (A)
│   └── أي قاعدة (كلهم يعطون نفس النتيجة)
│
├── COLLECT (C)
│   ├── جمع كل النتائج
│   └── اختياري: SUM, MIN, MAX, COUNT
│
├── RULE ORDER (R)
│   └── حسب ترتيب القواعد في الجدول
│
└── OUTPUT ORDER (O)
    └── حسب ترتيب قيم المخرجات
```

**أمثلة على Hit Policies**:

#### UNIQUE
```
يجب أن تكون القواعد متنافية (غير متداخلة)

✅ صحيح:
Rule 1: age < 18  → minor
Rule 2: age >= 18 → adult

❌ خطأ (قاعدتان تتطابقان):
Rule 1: age >= 18 → adult
Rule 2: age >= 21 → adult_plus
(عند age = 25، كلاهما يتطابق!)
```

#### FIRST
```
أول قاعدة تتطابق تُطبق

Rule 1: customerType = "VIP" → discount = 20
Rule 2: orderAmount > 5000   → discount = 10

Input: VIP customer, orderAmount = 6000
Output: discount = 20 (Rule 1 فقط)
```

#### COLLECT + SUM
```
جمع كل النتائج المتطابقة

Rule 1: customerType = "VIP"    → +10
Rule 2: orderAmount > 5000      → +5
Rule 3: seasonalPromo = true    → +3

Input: VIP, amount = 6000, promo = true
Output: 10 + 5 + 3 = 18
```

---

## محركات القواعد

### 1. Drools

#### Drools Rule Language (DRL)

```drools
package com.example.rules;

import com.example.model.Customer;
import com.example.model.Order;
import com.example.model.Discount;

// القاعدة 1: عملاء VIP مع ولاء 5+ سنوات
rule "VIP Customer - 5+ Years Loyalty"
    salience 100  // الأولوية (أعلى رقم = أعلى أولوية)
    when
        customer: Customer(
            type == "VIP",
            loyaltyYears >= 5
        )
        order: Order()
        discount: Discount()
    then
        discount.setPercentage(20);
        discount.setReason("VIP عميل مع ولاء 5+ سنوات");
        update(discount);
end

// القاعدة 2: عملاء VIP مع ولاء 2-4 سنوات
rule "VIP Customer - 2-4 Years Loyalty"
    salience 90
    when
        customer: Customer(
            type == "VIP",
            loyaltyYears >= 2,
            loyaltyYears < 5
        )
        order: Order()
        discount: Discount()
    then
        discount.setPercentage(15);
        discount.setReason("VIP عميل مع ولاء 2-4 سنوات");
        update(discount);
end

// القاعدة 3: طلبات كبيرة (خصم إضافي)
rule "Large Order Bonus"
    salience 50
    when
        order: Order(amount > 10000)
        discount: Discount(percentage > 0)
    then
        int currentDiscount = discount.getPercentage();
        discount.setPercentage(currentDiscount + 10);
        discount.addReason("خصم إضافي للطلبات الكبيرة");
        update(discount);
end

// القاعدة 4: الحد الأقصى للخصم
rule "Maximum Discount Cap"
    salience 10  // أولوية منخفضة (تُطبق آخراً)
    when
        discount: Discount(percentage > 30)
    then
        discount.setPercentage(30);
        discount.addReason("تم تطبيق الحد الأقصى للخصم");
        update(discount);
end

// القاعدة 5: عرض موسمي (ديسمبر)
rule "Seasonal December Promotion"
    date-effective "01-Dec-2025"
    date-expires "31-Dec-2025"
    when
        customer: Customer()
        order: Order()
        discount: Discount()
    then
        int currentDiscount = discount.getPercentage();
        discount.setPercentage(currentDiscount + 5);
        discount.addReason("عرض ديسمبر الموسمي");
        update(discount);
end
```

---

#### استخدام Drools من Java

```java
import org.kie.api.KieServices;
import org.kie.api.runtime.KieContainer;
import org.kie.api.runtime.KieSession;

public class DiscountService {

    private KieContainer kieContainer;

    public DiscountService() {
        KieServices kieServices = KieServices.Factory.get();
        this.kieContainer = kieServices.getKieClasspathContainer();
    }

    public Discount calculateDiscount(Customer customer, Order order) {
        // إنشاء جلسة لتنفيذ القواعد
        KieSession kieSession = kieContainer.newKieSession("discount-rules");

        try {
            // إنشاء كائن الخصم
            Discount discount = new Discount();
            discount.setPercentage(0);

            // إدخال الحقائق (Facts) في محرك القواعد
            kieSession.insert(customer);
            kieSession.insert(order);
            kieSession.insert(discount);

            // تنفيذ جميع القواعد
            int rulesFired = kieSession.fireAllRules();

            System.out.println("تم تنفيذ " + rulesFired + " قاعدة");

            return discount;

        } finally {
            kieSession.dispose();
        }
    }
}
```

---

#### كائنات النموذج (Model Objects)

```java
// Customer.java
public class Customer {
    private Long id;
    private String name;
    private String type;  // VIP, Gold, Silver, Regular
    private int loyaltyYears;

    // Getters and Setters
    public String getType() { return type; }
    public void setType(String type) { this.type = type; }

    public int getLoyaltyYears() { return loyaltyYears; }
    public void setLoyaltyYears(int loyaltyYears) { this.loyaltyYears = loyaltyYears; }
}

// Order.java
public class Order {
    private Long id;
    private Customer customer;
    private double amount;
    private Date orderDate;

    // Getters and Setters
    public double getAmount() { return amount; }
    public void setAmount(double amount) { this.amount = amount; }
}

// Discount.java
public class Discount {
    private int percentage;
    private List<String> reasons = new ArrayList<>();

    public int getPercentage() { return percentage; }
    public void setPercentage(int percentage) { this.percentage = percentage; }

    public void setReason(String reason) {
        this.reasons.clear();
        this.reasons.add(reason);
    }

    public void addReason(String reason) {
        this.reasons.add(reason);
    }

    public List<String> getReasons() { return reasons; }
}
```

---

### 2. Easy Rules

#### قواعد بسيطة بـ Java

```java
import org.jeasy.rules.annotation.*;
import org.jeasy.rules.api.Facts;
import org.jeasy.rules.core.DefaultRulesEngine;
import org.jeasy.rules.core.RulesEngineParameters;

// القاعدة 1: عملاء VIP
@Rule(name = "VIP Customer Rule",
      description = "عملاء VIP يحصلون على 20% خصم",
      priority = 1)
public class VIPCustomerRule {

    @Condition
    public boolean when(@Fact("customer") Customer customer) {
        return "VIP".equals(customer.getType())
            && customer.getLoyaltyYears() >= 5;
    }

    @Action
    public void then(@Fact("discount") Discount discount) {
        discount.setPercentage(20);
        discount.setReason("VIP عميل مع ولاء 5+ سنوات");
    }
}

// القاعدة 2: طلبات كبيرة
@Rule(name = "Large Order Rule",
      description = "طلبات أكثر من 10000 تحصل على خصم إضافي",
      priority = 2)
public class LargeOrderRule {

    @Condition
    public boolean when(@Fact("order") Order order) {
        return order.getAmount() > 10000;
    }

    @Action
    public void then(@Fact("discount") Discount discount) {
        int current = discount.getPercentage();
        discount.setPercentage(current + 10);
        discount.addReason("خصم إضافي للطلبات الكبيرة");
    }
}

// استخدام محرك القواعد
public class DiscountCalculator {

    public static Discount calculate(Customer customer, Order order) {
        // إنشاء محرك القواعد
        RulesEngineParameters parameters = new RulesEngineParameters()
            .skipOnFirstAppliedRule(false)  // تنفيذ جميع القواعد
            .priorityThreshold(Integer.MAX_VALUE);

        DefaultRulesEngine rulesEngine = new DefaultRulesEngine(parameters);

        // تسجيل القواعد
        Rules rules = new Rules();
        rules.register(new VIPCustomerRule());
        rules.register(new LargeOrderRule());

        // إنشاء الحقائق
        Facts facts = new Facts();
        Discount discount = new Discount();

        facts.put("customer", customer);
        facts.put("order", order);
        facts.put("discount", discount);

        // تنفيذ القواعد
        rulesEngine.fire(rules, facts);

        return discount;
    }
}
```

---

### 3. قواعد في Excel (OpenRules)

```
┌──────────────────────────────────────────────────────────────┐
│ ملف Excel: customer_discount_rules.xlsx                      │
└──────────────────────────────────────────────────────────────┘

Sheet: Discount Rules

| Rule ID | Customer Type | Loyalty Years | Order Amount | Discount % | Notes                    |
|---------|---------------|---------------|--------------|------------|--------------------------|
| R001    | VIP           | >= 5          | *            | 20         | VIP 5+ years            |
| R002    | VIP           | >= 2          | *            | 15         | VIP 2-4 years           |
| R003    | VIP           | >= 1          | *            | 10         | VIP new                 |
| R004    | Gold          | >= 3          | *            | 12         | Gold loyal              |
| R005    | Gold          | >= 1          | *            | 8          | Gold regular            |
| R006    | Silver        | *             | > 5000       | 7          | Silver big order        |
| R007    | Silver        | *             | > 1000       | 5          | Silver medium order     |
| R008    | Regular       | *             | > 10000      | 10         | Regular very big order  |
| R009    | Regular       | *             | > 5000       | 5          | Regular big order       |
| R010    | *             | *             | *            | 0          | Default - no discount   |

Sheet: Business Logic

# Additional Bonuses
| Condition              | Bonus % |
|------------------------|---------|
| Order Amount > 20000   | +5      |
| Black Friday           | +10     |
| First Purchase         | +5      |

# Maximum Cap
Maximum Discount: 30%
```

---

## أمثلة عملية شاملة

### مثال 1: نظام الموافقة على القروض

#### متطلبات العمل

```
نظام الموافقة على القروض البنكية

العوامل المؤثرة:
1. الراتب الشهري
2. الدرجة الائتمانية (Credit Score)
3. مدة العمل الحالية
4. الالتزامات الحالية
5. نوع القرض (شخصي، عقاري، سيارة)
6. مبلغ القرض

القرارات الممكنة:
- موافق تلقائياً
- موافق بشروط
- يحتاج مراجعة يدوية
- مرفوض تلقائياً
```

---

#### DMN: جدول قرار الموافقة على القرض

```
╔════════════════════════════════════════════════════════════════════════════════════════╗
║                          جدول قرار: الموافقة على القرض                                  ║
╠════════════════════════════════════════════════════════════════════════════════════════╣
║ Credit │ Monthly  │ Employment │ Debt to  │ Loan    │ Decision         │ Interest  ║
║ Score  │ Salary   │ Duration   │ Income   │ Amount  │                  │ Rate      ║
╠════════╪══════════╪════════════╪══════════╪═════════╪══════════════════╪═══════════╣
║ >= 750 │ >= 15000 │ >= 2 years │ < 30%    │ Any     │ موافق تلقائياً    │ 4.5%      ║
║ >= 700 │ >= 12000 │ >= 1 year  │ < 40%    │ < 500K  │ موافق تلقائياً    │ 5.0%      ║
║ >= 700 │ >= 12000 │ >= 1 year  │ < 40%    │ >= 500K │ موافق بشروط      │ 5.5%      ║
║ >= 650 │ >= 10000 │ >= 1 year  │ < 35%    │ < 300K  │ موافق بشروط      │ 6.0%      ║
║ >= 650 │ >= 10000 │ >= 6 months│ < 40%    │ < 200K  │ يحتاج مراجعة     │ 6.5%      ║
║ >= 600 │ >= 8000  │ >= 1 year  │ < 30%    │ < 200K  │ يحتاج مراجعة     │ 7.0%      ║
║ < 600  │ Any      │ Any        │ Any      │ Any     │ مرفوض تلقائياً    │ -         ║
║ Any    │ < 5000   │ Any        │ Any      │ Any     │ مرفوض تلقائياً    │ -         ║
║ Any    │ Any      │ < 6 months │ Any      │ > 100K  │ مرفوض تلقائياً    │ -         ║
║ Any    │ Any      │ Any        │ >= 50%   │ Any     │ مرفوض تلقائياً    │ -         ║
╚════════╧══════════╧════════════╧══════════╧═════════╧══════════════════╧═══════════╝

Hit Policy: FIRST
```

---

#### Drools Rules للقرض

```drools
package com.bank.loan.rules;

import com.bank.model.LoanApplication;
import com.bank.model.Applicant;
import com.bank.model.LoanDecision;

// قاعدة 1: رفض تلقائي - درجة ائتمان منخفضة
rule "Auto Reject - Low Credit Score"
    salience 1000  // أعلى أولوية
    when
        applicant: Applicant(creditScore < 600)
        decision: LoanDecision()
    then
        decision.setStatus("REJECTED");
        decision.setReason("الدرجة الائتمانية منخفضة جداً (< 600)");
        decision.setAutomatic(true);
end

// قاعدة 2: رفض تلقائي - راتب منخفض
rule "Auto Reject - Low Salary"
    salience 1000
    when
        applicant: Applicant(monthlySalary < 5000)
        decision: LoanDecision()
    then
        decision.setStatus("REJECTED");
        decision.setReason("الراتب الشهري أقل من الحد الأدنى (5000)");
        decision.setAutomatic(true);
end

// قاعدة 3: رفض تلقائي - التزامات عالية
rule "Auto Reject - High Debt Ratio"
    salience 1000
    when
        applicant: Applicant()
        application: LoanApplication()
        eval((applicant.getCurrentDebts() / applicant.getMonthlySalary()) >= 0.5)
        decision: LoanDecision()
    then
        decision.setStatus("REJECTED");
        decision.setReason("نسبة الالتزامات إلى الدخل عالية جداً (>= 50%)");
        decision.setAutomatic(true);
end

// قاعدة 4: موافقة تلقائية - عميل ممتاز
rule "Auto Approve - Excellent Customer"
    salience 900
    when
        applicant: Applicant(
            creditScore >= 750,
            monthlySalary >= 15000,
            employmentDurationMonths >= 24
        )
        application: LoanApplication()
        eval((applicant.getCurrentDebts() / applicant.getMonthlySalary()) < 0.3)
        decision: LoanDecision(status == null)
    then
        decision.setStatus("APPROVED");
        decision.setReason("عميل ممتاز - موافقة تلقائية");
        decision.setInterestRate(4.5);
        decision.setAutomatic(true);
        decision.addCondition("لا شروط إضافية");
end

// قاعدة 5: موافقة تلقائية - عميل جيد جداً
rule "Auto Approve - Very Good Customer"
    salience 850
    when
        applicant: Applicant(
            creditScore >= 700,
            monthlySalary >= 12000,
            employmentDurationMonths >= 12
        )
        application: LoanApplication(amount < 500000)
        eval((applicant.getCurrentDebts() / applicant.getMonthlySalary()) < 0.4)
        decision: LoanDecision(status == null)
    then
        decision.setStatus("APPROVED");
        decision.setReason("عميل جيد جداً - موافقة تلقائية");
        decision.setInterestRate(5.0);
        decision.setAutomatic(true);
end

// قاعدة 6: موافقة بشروط - عميل جيد
rule "Conditional Approve - Good Customer"
    salience 800
    when
        applicant: Applicant(
            creditScore >= 650,
            monthlySalary >= 10000,
            employmentDurationMonths >= 12
        )
        application: LoanApplication(amount < 300000)
        eval((applicant.getCurrentDebts() / applicant.getMonthlySalary()) < 0.35)
        decision: LoanDecision(status == null)
    then
        decision.setStatus("APPROVED");
        decision.setReason("موافقة مشروطة");
        decision.setInterestRate(6.0);
        decision.setAutomatic(false);
        decision.addCondition("يجب تقديم ضامن");
        decision.addCondition("تأمين على الحياة إلزامي");
end

// قاعدة 7: يحتاج مراجعة - حالة متوسطة
rule "Manual Review Required - Medium Risk"
    salience 500
    when
        applicant: Applicant(
            creditScore >= 600,
            creditScore < 650,
            monthlySalary >= 8000
        )
        application: LoanApplication(amount < 200000)
        decision: LoanDecision(status == null)
    then
        decision.setStatus("MANUAL_REVIEW");
        decision.setReason("يحتاج مراجعة يدوية - مخاطر متوسطة");
        decision.setInterestRate(7.0);
        decision.setAutomatic(false);
        decision.setReviewerLevel("SENIOR_MANAGER");
end

// قاعدة 8: خصم على الفائدة - عميل البنك الحالي
rule "Interest Discount - Existing Customer"
    salience 100
    when
        applicant: Applicant(existingCustomer == true, relationshipYears >= 3)
        decision: LoanDecision(status == "APPROVED", interestRate > 0)
    then
        double currentRate = decision.getInterestRate();
        decision.setInterestRate(currentRate - 0.5);
        decision.addReason("خصم 0.5% لعميل البنك الحالي (3+ سنوات)");
end

// قاعدة 9: شرط إضافي - قرض كبير
rule "Additional Condition - Large Loan"
    salience 100
    when
        application: LoanApplication(amount >= 500000)
        decision: LoanDecision(status == "APPROVED")
    then
        decision.addCondition("يجب تقديم رهن عقاري");
        decision.addCondition("تقييم عقاري من مكتب معتمد");
end

// قاعدة 10: رفع للمدير - مبلغ كبير جداً
rule "Escalate - Very Large Amount"
    salience 100
    when
        application: LoanApplication(amount > 1000000)
        decision: LoanDecision(status == "APPROVED")
    then
        decision.setReviewerLevel("DIRECTOR");
        decision.addReason("يحتاج موافقة المدير (مبلغ > 1 مليون)");
end
```

---

#### الربط مع BPMN

```xml
<!-- BPMN Process -->
<bpmn:process id="loan_approval_process">

  <!-- بداية العملية -->
  <bpmn:startEvent id="start" name="تقديم طلب قرض"/>

  <!-- User Task: إدخال البيانات -->
  <bpmn:userTask id="input_data" name="إدخال بيانات الطلب">
    <bpmn:extensionElements>
      <camunda:formData>
        <camunda:formField id="creditScore" label="الدرجة الائتمانية" type="long"/>
        <camunda:formField id="monthlySalary" label="الراتب الشهري" type="long"/>
        <camunda:formField id="employmentMonths" label="مدة العمل (أشهر)" type="long"/>
        <camunda:formField id="currentDebts" label="الالتزامات الحالية" type="long"/>
        <camunda:formField id="loanAmount" label="مبلغ القرض" type="long"/>
      </camunda:formData>
    </bpmn:extensionElements>
  </bpmn:userTask>

  <!-- Business Rule Task: تقييم الطلب -->
  <bpmn:businessRuleTask
      id="evaluate_loan"
      name="تقييم طلب القرض"
      camunda:resultVariable="loanDecision"
      camunda:decisionRef="loan_decision_dmn"
      camunda:mapDecisionResult="singleEntry">

    <bpmn:extensionElements>
      <camunda:inputOutput>
        <camunda:inputParameter name="applicant">${applicant}</camunda:inputParameter>
        <camunda:inputParameter name="application">${application}</camunda:inputParameter>
        <camunda:outputParameter name="decision">${loanDecision}</camunda:outputParameter>
      </camunda:inputOutput>
    </bpmn:extensionElements>
  </bpmn:businessRuleTask>

  <!-- Gateway: نوع القرار؟ -->
  <bpmn:exclusiveGateway id="decision_gateway" name="القرار؟"/>

  <!-- موافق تلقائياً -->
  <bpmn:serviceTask id="auto_approve" name="موافقة تلقائية">
    <bpmn:conditionExpression>${decision.status == 'APPROVED' &amp;&amp; decision.automatic == true}</bpmn:conditionExpression>
  </bpmn:serviceTask>

  <!-- موافق بشروط أو مراجعة -->
  <bpmn:userTask id="manual_review" name="مراجعة يدوية">
    <bpmn:conditionExpression>${decision.status == 'APPROVED' &amp;&amp; decision.automatic == false || decision.status == 'MANUAL_REVIEW'}</bpmn:conditionExpression>
  </bpmn:userTask>

  <!-- مرفوض -->
  <bpmn:serviceTask id="auto_reject" name="رفض تلقائي">
    <bpmn:conditionExpression>${decision.status == 'REJECTED'}</bpmn:conditionExpression>
  </bpmn:serviceTask>

  <!-- الربط -->
  <bpmn:sequenceFlow sourceRef="start" targetRef="input_data"/>
  <bpmn:sequenceFlow sourceRef="input_data" targetRef="evaluate_loan"/>
  <bpmn:sequenceFlow sourceRef="evaluate_loan" targetRef="decision_gateway"/>
  <bpmn:sequenceFlow sourceRef="decision_gateway" targetRef="auto_approve"/>
  <bpmn:sequenceFlow sourceRef="decision_gateway" targetRef="manual_review"/>
  <bpmn:sequenceFlow sourceRef="decision_gateway" targetRef="auto_reject"/>

</bpmn:process>
```

---

### مثال 2: نظام التسعير الديناميكي

#### متطلبات العمل

```
نظام تسعير ديناميكي لمتجر إلكتروني

العوامل المؤثرة:
1. فئة المنتج
2. سعر المنافسين
3. مستوى المخزون
4. وقت الطلب (ساعة، يوم، موسم)
5. موقع العميل
6. سلوك الشراء السابق
7. طريقة الدفع

القرارات:
- السعر النهائي
- الخصومات المطبقة
- العروض الخاصة
```

---

#### DMN: التسعير الديناميكي

```
╔════════════════════════════════════════════════════════════════════════════════╗
║                    جدول قرار: التسعير الديناميكي                                ║
╠════════════════════════════════════════════════════════════════════════════════╣
║ Stock  │ Competitor │ Time of  │ Customer │ Payment │ Price      │ Reasoning ║
║ Level  │ Price      │ Day      │ Type     │ Method  │ Adjustment │           ║
╠════════╪════════════╪══════════╪══════════╪═════════╪════════════╪═══════════╣
║ < 10   │ < Our Price│ Peak     │ VIP      │ Any     │ +15%       │ ندرة+طلب  ║
║ < 10   │ < Our Price│ Off-Peak │ VIP      │ Any     │ +10%       │ ندرة      ║
║ < 10   │ >= Our Price│ Any     │ Any      │ Any     │ +5%        │ ندرة فقط  ║
║ 10-50  │ < Our Price│ Peak     │ VIP      │ Cash    │ +5%        │ طلب عالي  ║
║ 10-50  │ < Our Price│ Peak     │ Regular  │ Any     │ 0%         │ سعر عادي  ║
║ 10-50  │ >= Our Price│ Any     │ VIP      │ Any     │ -5%        │ مطابقة    ║
║ 10-50  │ >= Our Price│ Any     │ Regular  │ Cash    │ -3%        │ منافسة    ║
║ > 50   │ Any        │ Any      │ VIP      │ Cash    │ -10%       │ تصريف     ║
║ > 50   │ Any        │ Any      │ Regular  │ Cash    │ -7%        │ تصريف     ║
║ > 50   │ Any        │ Any      │ Any      │ Credit  │ -5%        │ تصريف     ║
╚════════╧════════════╧══════════╧══════════╧═════════╧════════════╧═══════════╝

Hit Policy: COLLECT + SUM (جمع جميع التعديلات)
```

---

#### Drools: التسعير الديناميكي

```drools
package com.shop.pricing.rules;

import com.shop.model.Product;
import com.shop.model.Customer;
import com.shop.model.Order;
import com.shop.model.PricingResult;
import java.time.LocalDateTime;
import java.time.DayOfWeek;

global PricingService pricingService;

// قاعدة 1: مخزون منخفض = سعر أعلى
rule "Low Stock Premium"
    when
        product: Product(stockLevel < 10, stockLevel > 0)
        result: PricingResult()
    then
        result.addAdjustment("LOW_STOCK", 0.10);
        result.addReason("مخزون منخفض - زيادة 10%");
end

// قاعدة 2: مخزون مرتفع = تخفيض للتصريف
rule "Overstocked Clearance"
    when
        product: Product(stockLevel > 50)
        result: PricingResult()
    then
        result.addAdjustment("CLEARANCE", -0.15);
        result.addReason("تصريف مخزون - خصم 15%");
end

// قاعدة 3: ساعات الذروة
rule "Peak Hours Pricing"
    when
        order: Order()
        eval(pricingService.isPeakHour(order.getOrderTime()))
        customer: Customer(type != "VIP")
        result: PricingResult()
    then
        result.addAdjustment("PEAK_HOUR", 0.05);
        result.addReason("ساعة ذروة - زيادة 5%");
end

// قاعدة 4: منافسة السعر
rule "Price Match Competition"
    when
        product: Product()
        competitorPrice: Double() from pricingService.getCompetitorPrice(product)
        eval(competitorPrice != null && competitorPrice < product.getBasePrice())
        result: PricingResult()
    then
        double discount = (product.getBasePrice() - competitorPrice) / product.getBasePrice();
        result.addAdjustment("PRICE_MATCH", -discount);
        result.addReason("مطابقة سعر المنافس - خصم " + (discount * 100) + "%");
end

// قاعدة 5: عميل VIP
rule "VIP Customer Discount"
    when
        customer: Customer(type == "VIP")
        result: PricingResult()
    then
        result.addAdjustment("VIP_DISCOUNT", -0.08);
        result.addReason("خصم VIP - خصم 8%");
end

// قاعدة 6: الدفع النقدي
rule "Cash Payment Discount"
    when
        order: Order(paymentMethod == "CASH")
        result: PricingResult()
    then
        result.addAdjustment("CASH_DISCOUNT", -0.03);
        result.addReason("دفع نقدي - خصم 3%");
end

// قاعدة 7: موسم التخفيضات (Black Friday)
rule "Black Friday Sale"
    date-effective "25-Nov-2025"
    date-expires "27-Nov-2025"
    when
        result: PricingResult()
    then
        result.addAdjustment("BLACK_FRIDAY", -0.25);
        result.addReason("Black Friday - خصم 25%");
end

// قاعدة 8: عميل جديد - عرض ترحيبي
rule "First Purchase Welcome Offer"
    when
        customer: Customer(totalOrders == 0)
        result: PricingResult()
    then
        result.addAdjustment("WELCOME_OFFER", -0.10);
        result.addReason("عرض ترحيبي للعملاء الجدد - خصم 10%");
end

// قاعدة 9: منتصف الأسبوع - عرض خاص
rule "Midweek Special"
    when
        order: Order()
        eval(order.getOrderTime().getDayOfWeek() == DayOfWeek.WEDNESDAY)
        result: PricingResult()
    then
        result.addAdjustment("MIDWEEK_SPECIAL", -0.05);
        result.addReason("عرض منتصف الأسبوع - خصم 5%");
end

// قاعدة 10: كمية كبيرة
rule "Bulk Purchase Discount"
    when
        order: Order(quantity >= 10)
        result: PricingResult()
    then
        double discount = order.getQuantity() >= 50 ? 0.20 :
                         order.getQuantity() >= 20 ? 0.15 : 0.10;
        result.addAdjustment("BULK_DISCOUNT", -discount);
        result.addReason("خصم الكمية - خصم " + (discount * 100) + "%");
end

// قاعدة 11: الحد الأقصى للخصم
rule "Maximum Discount Cap"
    salience -1000  // أقل أولوية (تُنفذ آخراً)
    when
        result: PricingResult()
        eval(result.getTotalAdjustment() < -0.40)
    then
        result.capAdjustment(-0.40);
        result.addReason("تم تطبيق الحد الأقصى للخصم (40%)");
end

// قاعدة 12: الحد الأدنى للسعر
rule "Minimum Price Floor"
    salience -1001
    when
        product: Product()
        result: PricingResult()
        eval(result.calculateFinalPrice(product.getBasePrice()) < product.getCostPrice() * 1.05)
    then
        double minPrice = product.getCostPrice() * 1.05;
        result.setFinalPrice(minPrice);
        result.addReason("تم تطبيق الحد الأدنى للسعر (تكلفة + 5%)");
end
```

---

#### كائنات النموذج للتسعير

```java
// PricingResult.java
public class PricingResult {
    private Map<String, Double> adjustments = new HashMap<>();
    private List<String> reasons = new ArrayList<>();
    private Double finalPrice;

    public void addAdjustment(String code, double percentage) {
        adjustments.put(code, percentage);
    }

    public double getTotalAdjustment() {
        return adjustments.values().stream()
            .mapToDouble(Double::doubleValue)
            .sum();
    }

    public void addReason(String reason) {
        reasons.add(reason);
    }

    public double calculateFinalPrice(double basePrice) {
        if (finalPrice != null) {
            return finalPrice;
        }
        double adjustment = getTotalAdjustment();
        return basePrice * (1 + adjustment);
    }

    public void setFinalPrice(double price) {
        this.finalPrice = price;
    }

    public void capAdjustment(double maxDiscount) {
        double current = getTotalAdjustment();
        if (current < maxDiscount) {
            double diff = maxDiscount - current;
            adjustments.put("CAP", diff);
        }
    }
}

// PricingService.java
@Service
public class PricingService {

    public boolean isPeakHour(LocalDateTime time) {
        int hour = time.getHour();
        // ساعات الذروة: 12-2 ظهراً و 6-9 مساءً
        return (hour >= 12 && hour <= 14) || (hour >= 18 && hour <= 21);
    }

    public Double getCompetitorPrice(Product product) {
        // استدعاء API لجلب أسعار المنافسين
        // هنا مثال بسيط
        return competitorPriceAPI.getLowestPrice(product.getSku());
    }
}
```

---

## المقارنة مع Script Task

### متى تستخدم أيهما؟

```
                    [قرار منطقي مطلوب]
                            │
                            ↓
                ⬦ القواعد معقدة ومتعددة؟
                ↙                        ↘
              نعم                         لا
               ↓                           ↓
    ⬦ تتغير بشكل متكرر؟          [Script Task كافي]
    ↙                  ↘
  نعم                  لا
   ↓                    ↓
[Business Rule Task] ⬦ يحتاج غير مبرمجين؟
                     ↙              ↘
                   نعم              لا
                    ↓                ↓
          [Business Rule Task]  [Script Task]
```

---

### جدول المقارنة

| المعيار | Script Task | Business Rule Task |
|---------|-------------|-------------------|
| **التعقيد** | بسيط-متوسط | متوسط-معقد جداً |
| **عدد القواعد** | 1-5 | 10+ |
| **التغيير** | نادر | متكرر |
| **من يعدّل** | مبرمج | محلل أعمال |
| **الاختبار** | صعب | سهل (معزول) |
| **إعادة الاستخدام** | صعب | سهل |
| **الأداء** | سريع | أبطأ قليلاً |
| **الصيانة** | صعبة | سهلة |

---

### أمثلة مقارنة

#### حالة 1: التحقق من العمر

```javascript
// Script Task - مناسب
if (age >= 18) {
  eligible = true;
} else {
  eligible = false;
}
```

**السبب**: قاعدة واحدة بسيطة، لا تتغير

---

#### حالة 2: حساب الضريبة

```javascript
// Script Task - مناسب نسبياً
let taxRate;
if (income <= 50000) {
  taxRate = 0.10;
} else if (income <= 100000) {
  taxRate = 0.15;
} else if (income <= 200000) {
  taxRate = 0.20;
} else {
  taxRate = 0.25;
}
tax = income * taxRate;
```

**لكن**: إذا كانت قوانين الضرائب تتغير سنوياً → Business Rule Task أفضل

---

#### حالة 3: خصم العميل

```
❌ Script Task - غير مناسب
function calculateDiscount(customer, order) {
  // 50 سطر من الشروط المعقدة
  // ...
}

✅ Business Rule Task - مناسب
- 20+ قاعدة مختلفة
- تتغير كل شهر
- يحتاج محلل التسويق تعديلها
```

---

## أفضل الممارسات

### 1. تنظيم القواعد

```
✅ جيد: فصل القواعد حسب الموضوع

customer_discount_rules.drl
├── VIP_rules
├── seasonal_rules
├── volume_rules
└── maximum_cap_rules

loan_approval_rules.drl
├── auto_reject_rules
├── auto_approve_rules
├── conditional_approve_rules
└── review_required_rules

❌ سيء: كل القواعد في ملف واحد
all_business_rules.drl (2000 قاعدة!)
```

---

### 2. تسمية واضحة

```
✅ جيد:
rule "VIP Customer With 5+ Years Loyalty - 20% Discount"

❌ سيء:
rule "Rule_123"
rule "Discount Calculation"
```

---

### 3. الأولويات (Salience)

```
✅ جيد: استخدام نطاقات منطقية

1000-999: قواعد التحقق والتهيئة
900-800:  قواعد الرفض التلقائي
700-500:  قواعد الموافقة
400-200:  قواعد الشروط الإضافية
100-1:    قواعد التعديل النهائي
0--999:   قواعد التنظيف والقيود

❌ سيء:
rule "A" salience 75384
rule "B" salience 12
rule "C" salience 999999
```

---

### 4. التوثيق

```drools
✅ جيد:

/**
 * قاعدة: خصم عملاء VIP
 *
 * الهدف: منح خصم للعملاء VIP بناءً على سنوات الولاء
 * الشروط:
 *   - نوع العميل = VIP
 *   - سنوات الولاء >= 5
 * الإجراء:
 *   - خصم 20%
 * التحديث الأخير: 2025-01-04
 * المسؤول: فريق التسويق
 */
rule "VIP Customer - 5+ Years Loyalty"
    salience 100
    when
        customer: Customer(type == "VIP", loyaltyYears >= 5)
        discount: Discount()
    then
        discount.setPercentage(20);
        discount.setReason("VIP عميل مع ولاء 5+ سنوات");
end
```

---

### 5. الاختبار

```java
// اختبارات وحدة للقواعد

@Test
public void testVIPCustomerDiscount() {
    // Arrange
    Customer customer = new Customer();
    customer.setType("VIP");
    customer.setLoyaltyYears(5);

    Order order = new Order();
    order.setAmount(1000);

    // Act
    Discount discount = discountService.calculate(customer, order);

    // Assert
    assertEquals(20, discount.getPercentage());
    assertTrue(discount.getReasons().contains("VIP عميل مع ولاء 5+ سنوات"));
}

@Test
public void testMaximumDiscountCap() {
    // اختبار أن الخصم لا يتجاوز 30%
    Customer customer = new Customer();
    customer.setType("VIP");
    customer.setLoyaltyYears(10);

    Order order = new Order();
    order.setAmount(100000); // طلب كبير جداً

    Discount discount = discountService.calculate(customer, order);

    assertTrue(discount.getPercentage() <= 30,
               "الخصم لا يجب أن يتجاوز 30%");
}
```

---

### 6. المراقبة والتسجيل

```java
// تسجيل تنفيذ القواعد

kieSession.addEventListener(new RuleRuntimeEventListener() {
    @Override
    public void objectInserted(ObjectInsertedEvent event) {
        log.debug("تم إدخال: {}", event.getObject());
    }

    @Override
    public void objectUpdated(ObjectUpdatedEvent event) {
        log.debug("تم تحديث: {}", event.getObject());
    }
});

kieSession.addEventListener(new AgendaEventListener() {
    @Override
    public void afterMatchFired(AfterMatchFiredEvent event) {
        String ruleName = event.getMatch().getRule().getName();
        log.info("تم تنفيذ القاعدة: {}", ruleName);
    }
});
```

---

### 7. إدارة النسخ

```
✅ استخدم Git لتتبع التغييرات في ملفات القواعد

discount_rules.drl
├── v1.0.0 (2024-01-01) - النسخة الأولية
├── v1.1.0 (2024-06-01) - إضافة قواعد الصيف
├── v1.2.0 (2024-11-01) - Black Friday
└── v2.0.0 (2025-01-01) - إعادة هيكلة

كل تغيير:
- Commit message واضح
- Migration guide إذا لزم
- Backward compatibility test
```

---

## الخلاصة

### متى تستخدم Business Rule Task؟

✅ **استخدمه عندما**:
1. القواعد معقدة ومتعددة (10+ قاعدة)
2. تتغير بشكل متكرر (شهرياً أو أكثر)
3. يحتاج غير المبرمجين تعديلها
4. القواعد منطق عمل (ليست منطق تقني)
5. تريد فصل القواعد عن الكود
6. تحتاج إعادة استخدام القواعد
7. التدقيق والمراجعة مهمان

❌ **لا تستخدمه عندما**:
1. القواعد بسيطة (1-3 شروط)
2. لا تتغير أبداً
3. منطق تقني بحت
4. السرعة حرجة جداً (milliseconds matter)

---

### الأدوات المفيدة

```
Business Rule Engines:
1. Drools (JBoss)         - الأكثر شيوعاً
2. Camunda DMN            - مدمج مع Camunda BPM
3. Easy Rules             - للحالات البسيطة
4. OpenRules              - Excel-based
5. IBM ODM                - Enterprise

IDEs و Tools:
1. Drools Workbench       - محرر ويب للقواعد
2. Decision Table Editor  - لجداول DMN
3. IntelliJ IDEA          - دعم Drools
4. Eclipse Drools Plugin  - محرر DRL

Testing:
1. JUnit + Drools         - اختبارات وحدة
2. Drools Test Scenarios  - اختبارات القواعد
3. Decision Table Testing - DMN testing
```

---

**Business Rule Task = فصل منطق العمل عن الكود البرمجي!**

القواعد يديرها محللو الأعمال، الكود يديره المبرمجون. كل واحد يعمل في تخصصه! 🎯

---

**تم إنشاء هذا الدليل كمرجع شامل لـ Business Rule Task في BPMN**
