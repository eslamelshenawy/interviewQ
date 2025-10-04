# Data Validation - الدليل الشامل لمعالجة التحقق من البيانات

## جدول المحتويات
1. [ما هو Validation؟](#ما-هو-validation)
2. [أنواع Validation](#أنواع-validation)
3. [مستويات Validation](#مستويات-validation)
4. [Validation في الواجهات](#validation-في-الواجهات)
5. [Validation في Backend](#validation-في-backend)
6. [Validation في قواعد البيانات](#validation-في-قواعد-البيانات)
7. [معالجة أخطاء Validation](#معالجة-أخطاء-validation)
8. [أمثلة عملية شاملة](#أمثلة-عملية-شاملة)
9. [أفضل الممارسات](#أفضل-الممارسات)

---

## ما هو Validation؟

### التعريف

**Validation (التحقق من صحة البيانات)** هو عملية التأكد من أن البيانات المُدخلة:
- صحيحة
- كاملة
- آمنة
- تتوافق مع قواعد العمل

### لماذا Validation مهم؟

```
بدون Validation:
┌─────────────────────────┐
│ مستخدم يدخل:            │
│ Email: "abc"            │  ← ليس بريد إلكتروني!
│ Age: -5                 │  ← عمر سالب!
│ Phone: "hello"          │  ← ليس رقم!
└─────────────────────────┘
        ↓
┌─────────────────────────┐
│ قاعدة البيانات:         │
│ ❌ بيانات فاسدة         │
│ ❌ أخطاء في المعالجة    │
│ ❌ ثغرات أمنية          │
└─────────────────────────┘

مع Validation:
┌─────────────────────────┐
│ مستخدم يدخل بيانات خاطئة │
└─────────────────────────┘
        ↓
┌─────────────────────────┐
│ ✅ Validation يرفضها    │
│ ✅ رسائل خطأ واضحة      │
│ ✅ إرشادات للتصحيح      │
└─────────────────────────┘
```

---

## أنواع Validation

### شجرة التصنيف

```
Data Validation
│
├── 1. Syntax Validation (التحقق من الصيغة)
│   ├── Format Validation
│   │   ├── Email format
│   │   ├── Phone format
│   │   ├── Date format
│   │   ├── URL format
│   │   └── Credit card format
│   │
│   ├── Type Validation
│   │   ├── String
│   │   ├── Number
│   │   ├── Boolean
│   │   ├── Date
│   │   └── Object
│   │
│   └── Pattern Validation (Regex)
│       ├── Alphanumeric
│       ├── Custom patterns
│       └── Special characters
│
├── 2. Semantic Validation (التحقق من المعنى)
│   ├── Range Validation
│   │   ├── Min/Max value
│   │   ├── Min/Max length
│   │   └── Between ranges
│   │
│   ├── Business Rules
│   │   ├── Age >= 18 for registration
│   │   ├── Order amount > 0
│   │   ├── Start date < End date
│   │   └── Custom business logic
│   │
│   └── Cross-field Validation
│       ├── Password = Confirm Password
│       ├── Departure < Arrival
│       └── Related fields consistency
│
├── 3. Database Validation (التحقق من قاعدة البيانات)
│   ├── Uniqueness
│   │   ├── Unique email
│   │   ├── Unique username
│   │   └── Unique IDs
│   │
│   ├── Existence
│   │   ├── Foreign key exists
│   │   ├── Reference data exists
│   │   └── Related records exist
│   │
│   └── Constraints
│       ├── NOT NULL
│       ├── UNIQUE
│       ├── CHECK
│       └── FOREIGN KEY
│
├── 4. Security Validation (التحقق الأمني)
│   ├── SQL Injection prevention
│   ├── XSS prevention
│   ├── CSRF protection
│   ├── File upload validation
│   └── Input sanitization
│
└── 5. Business Logic Validation (منطق العمل)
    ├── Inventory check (stock available?)
    ├── Credit limit check
    ├── Permission check
    ├── Workflow state validation
    └── Complex business rules
```

---

## مستويات Validation

### 1. Client-Side Validation (في المتصفح)

```
المميزات:
✅ سريع - فوري بدون طلب للسيرفر
✅ تجربة مستخدم أفضل
✅ توفير موارد السيرفر

العيوب:
❌ يمكن تجاوزه (غير آمن)
❌ يعتمد على JavaScript
❌ غير كافٍ وحده

الاستخدام:
→ تحسين تجربة المستخدم
→ ردود فعل فورية
→ ليس للأمان!
```

---

### 2. Server-Side Validation (في السيرفر)

```
المميزات:
✅ آمن - لا يمكن تجاوزه
✅ موثوق
✅ يصل لقاعدة البيانات

العيوب:
❌ أبطأ (يحتاج طلب HTTP)
❌ استهلاك موارد السيرفر

الاستخدام:
→ إجباري للأمان
→ التحقق النهائي
→ قواعد العمل المعقدة
```

---

### 3. Database Validation (في قاعدة البيانات)

```
المميزات:
✅ ضمان تكامل البيانات
✅ لا يمكن تجاوزه أبداً
✅ آخر خط دفاع

العيوب:
❌ رسائل خطأ غير ودية
❌ صعب التخصيص
❌ أداء أقل

الاستخدام:
→ قيود حرجة (UNIQUE, NOT NULL)
→ العلاقات بين الجداول
→ تكامل البيانات
```

---

### الاستراتيجية المثلى: التحقق متعدد المستويات

```
طبقات الحماية (Defense in Depth)

┌──────────────────────────────────────┐
│ 1. Client-Side Validation            │  ← UX
│    (تجربة مستخدم)                    │
├──────────────────────────────────────┤
│ 2. Server-Side Validation            │  ← Security
│    (الأمان الحقيقي)                  │
├──────────────────────────────────────┤
│ 3. Database Constraints               │  ← Integrity
│    (تكامل البيانات)                  │
└──────────────────────────────────────┘

كل طبقة تحمي من نوع مختلف من المشاكل
```

---

## Validation في الواجهات

### 1. HTML5 Native Validation

```html
<!-- مثال: نموذج تسجيل -->
<form id="registrationForm" novalidate>

  <!-- الاسم - مطلوب، طول معين -->
  <div>
    <label for="username">اسم المستخدم:</label>
    <input
      type="text"
      id="username"
      name="username"
      required
      minlength="3"
      maxlength="20"
      pattern="^[a-zA-Z0-9_]+$"
      title="حروف وأرقام و _ فقط"
      placeholder="اسم المستخدم"
    >
    <span class="error" id="username-error"></span>
  </div>

  <!-- البريد الإلكتروني -->
  <div>
    <label for="email">البريد الإلكتروني:</label>
    <input
      type="email"
      id="email"
      name="email"
      required
      placeholder="example@email.com"
    >
    <span class="error" id="email-error"></span>
  </div>

  <!-- كلمة المرور -->
  <div>
    <label for="password">كلمة المرور:</label>
    <input
      type="password"
      id="password"
      name="password"
      required
      minlength="8"
      pattern="^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]{8,}$"
      title="8 أحرف على الأقل، حرف كبير، حرف صغير، رقم، رمز خاص"
      placeholder="********"
    >
    <span class="error" id="password-error"></span>
  </div>

  <!-- تأكيد كلمة المرور -->
  <div>
    <label for="confirmPassword">تأكيد كلمة المرور:</label>
    <input
      type="password"
      id="confirmPassword"
      name="confirmPassword"
      required
      placeholder="********"
    >
    <span class="error" id="confirmPassword-error"></span>
  </div>

  <!-- العمر -->
  <div>
    <label for="age">العمر:</label>
    <input
      type="number"
      id="age"
      name="age"
      required
      min="18"
      max="120"
      placeholder="18"
    >
    <span class="error" id="age-error"></span>
  </div>

  <!-- رقم الهاتف -->
  <div>
    <label for="phone">رقم الهاتف:</label>
    <input
      type="tel"
      id="phone"
      name="phone"
      required
      pattern="^(\+966|00966|0)?5[0-9]{8}$"
      title="رقم سعودي صحيح"
      placeholder="+966501234567"
    >
    <span class="error" id="phone-error"></span>
  </div>

  <!-- تاريخ الميلاد -->
  <div>
    <label for="birthdate">تاريخ الميلاد:</label>
    <input
      type="date"
      id="birthdate"
      name="birthdate"
      required
      max="2007-01-01"
    >
    <span class="error" id="birthdate-error"></span>
  </div>

  <!-- الموافقة على الشروط -->
  <div>
    <label>
      <input
        type="checkbox"
        id="terms"
        name="terms"
        required
      >
      أوافق على الشروط والأحكام
    </label>
    <span class="error" id="terms-error"></span>
  </div>

  <button type="submit">تسجيل</button>
</form>
```

---

### 2. JavaScript Validation (متقدم)

```javascript
// ========================================
// Validation Rules
// ========================================

const validationRules = {
  username: {
    required: true,
    minLength: 3,
    maxLength: 20,
    pattern: /^[a-zA-Z0-9_]+$/,
    messages: {
      required: 'اسم المستخدم مطلوب',
      minLength: 'اسم المستخدم يجب أن يكون 3 أحرف على الأقل',
      maxLength: 'اسم المستخدم يجب أن لا يتجاوز 20 حرف',
      pattern: 'اسم المستخدم يجب أن يحتوي على حروف وأرقام و _ فقط'
    }
  },

  email: {
    required: true,
    pattern: /^[^\s@]+@[^\s@]+\.[^\s@]+$/,
    messages: {
      required: 'البريد الإلكتروني مطلوب',
      pattern: 'البريد الإلكتروني غير صحيح'
    }
  },

  password: {
    required: true,
    minLength: 8,
    pattern: /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]{8,}$/,
    messages: {
      required: 'كلمة المرور مطلوبة',
      minLength: 'كلمة المرور يجب أن تكون 8 أحرف على الأقل',
      pattern: 'كلمة المرور يجب أن تحتوي على: حرف كبير، حرف صغير، رقم، رمز خاص'
    }
  },

  confirmPassword: {
    required: true,
    matchField: 'password',
    messages: {
      required: 'تأكيد كلمة المرور مطلوب',
      matchField: 'كلمة المرور غير متطابقة'
    }
  },

  age: {
    required: true,
    min: 18,
    max: 120,
    messages: {
      required: 'العمر مطلوب',
      min: 'يجب أن يكون عمرك 18 سنة على الأقل',
      max: 'العمر غير صحيح'
    }
  },

  phone: {
    required: true,
    pattern: /^(\+966|00966|0)?5[0-9]{8}$/,
    messages: {
      required: 'رقم الهاتف مطلوب',
      pattern: 'رقم الهاتف غير صحيح (مثال: +966501234567)'
    }
  },

  terms: {
    required: true,
    checked: true,
    messages: {
      required: 'يجب الموافقة على الشروط والأحكام'
    }
  }
};

// ========================================
// Validation Functions
// ========================================

class FormValidator {
  constructor(formId, rules) {
    this.form = document.getElementById(formId);
    this.rules = rules;
    this.errors = {};
    this.init();
  }

  init() {
    // منع الإرسال الافتراضي
    this.form.addEventListener('submit', (e) => {
      e.preventDefault();
      this.validateForm();
    });

    // Validation فوري عند الكتابة
    Object.keys(this.rules).forEach(fieldName => {
      const field = this.form.elements[fieldName];
      if (field) {
        // عند فقدان التركيز
        field.addEventListener('blur', () => {
          this.validateField(fieldName);
        });

        // عند الكتابة (للحقول النصية)
        if (field.type === 'text' || field.type === 'email' ||
            field.type === 'password' || field.type === 'tel') {
          field.addEventListener('input', () => {
            // امسح الخطأ عند الكتابة
            this.clearFieldError(fieldName);
          });
        }
      }
    });
  }

  validateField(fieldName) {
    const field = this.form.elements[fieldName];
    const rules = this.rules[fieldName];
    const value = field.type === 'checkbox' ? field.checked : field.value.trim();

    // Required
    if (rules.required && !value) {
      this.setFieldError(fieldName, rules.messages.required);
      return false;
    }

    // Checked (for checkboxes)
    if (rules.checked && !field.checked) {
      this.setFieldError(fieldName, rules.messages.required);
      return false;
    }

    // MinLength
    if (rules.minLength && value.length < rules.minLength) {
      this.setFieldError(fieldName, rules.messages.minLength);
      return false;
    }

    // MaxLength
    if (rules.maxLength && value.length > rules.maxLength) {
      this.setFieldError(fieldName, rules.messages.maxLength);
      return false;
    }

    // Min (for numbers)
    if (rules.min !== undefined && parseFloat(value) < rules.min) {
      this.setFieldError(fieldName, rules.messages.min);
      return false;
    }

    // Max (for numbers)
    if (rules.max !== undefined && parseFloat(value) > rules.max) {
      this.setFieldError(fieldName, rules.messages.max);
      return false;
    }

    // Pattern (Regex)
    if (rules.pattern && !rules.pattern.test(value)) {
      this.setFieldError(fieldName, rules.messages.pattern);
      return false;
    }

    // Match Field (مقارنة مع حقل آخر)
    if (rules.matchField) {
      const matchFieldValue = this.form.elements[rules.matchField].value;
      if (value !== matchFieldValue) {
        this.setFieldError(fieldName, rules.messages.matchField);
        return false;
      }
    }

    // Custom Validator
    if (rules.customValidator) {
      const customError = rules.customValidator(value, this.form);
      if (customError) {
        this.setFieldError(fieldName, customError);
        return false;
      }
    }

    // كل شيء صحيح
    this.clearFieldError(fieldName);
    return true;
  }

  validateForm() {
    this.errors = {};
    let isValid = true;

    // Validate كل الحقول
    Object.keys(this.rules).forEach(fieldName => {
      if (!this.validateField(fieldName)) {
        isValid = false;
      }
    });

    if (isValid) {
      console.log('✅ النموذج صحيح');
      this.submitForm();
    } else {
      console.log('❌ النموذج يحتوي على أخطاء');
      // التركيز على أول حقل خاطئ
      const firstErrorField = Object.keys(this.errors)[0];
      if (firstErrorField) {
        this.form.elements[firstErrorField].focus();
      }
    }

    return isValid;
  }

  setFieldError(fieldName, message) {
    this.errors[fieldName] = message;
    const field = this.form.elements[fieldName];
    const errorElement = document.getElementById(`${fieldName}-error`);

    // إضافة class للحقل
    field.classList.add('invalid');
    field.classList.remove('valid');

    // عرض رسالة الخطأ
    if (errorElement) {
      errorElement.textContent = message;
      errorElement.style.display = 'block';
    }
  }

  clearFieldError(fieldName) {
    delete this.errors[fieldName];
    const field = this.form.elements[fieldName];
    const errorElement = document.getElementById(`${fieldName}-error`);

    // إزالة class
    field.classList.remove('invalid');
    field.classList.add('valid');

    // إخفاء رسالة الخطأ
    if (errorElement) {
      errorElement.textContent = '';
      errorElement.style.display = 'none';
    }
  }

  async submitForm() {
    const formData = new FormData(this.form);
    const data = Object.fromEntries(formData.entries());

    try {
      const response = await fetch('/api/register', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify(data)
      });

      const result = await response.json();

      if (response.ok) {
        console.log('✅ تم التسجيل بنجاح');
        // إعادة توجيه أو عرض رسالة نجاح
        window.location.href = '/success';
      } else {
        // معالجة أخطاء السيرفر
        this.handleServerErrors(result.errors);
      }
    } catch (error) {
      console.error('❌ خطأ في الإرسال:', error);
      alert('حدث خطأ، يرجى المحاولة لاحقاً');
    }
  }

  handleServerErrors(serverErrors) {
    // عرض أخطاء السيرفر
    Object.keys(serverErrors).forEach(fieldName => {
      this.setFieldError(fieldName, serverErrors[fieldName]);
    });
  }
}

// ========================================
// تهيئة الـ Validator
// ========================================

document.addEventListener('DOMContentLoaded', () => {
  const validator = new FormValidator('registrationForm', validationRules);
});
```

---

### 3. CSS للتصميم

```css
/* تنسيق الحقول */
input,
select,
textarea {
  width: 100%;
  padding: 10px;
  border: 2px solid #ddd;
  border-radius: 4px;
  font-size: 16px;
  transition: border-color 0.3s;
}

input:focus {
  outline: none;
  border-color: #4CAF50;
}

/* الحقول الصحيحة */
input.valid {
  border-color: #4CAF50;
  background-image: url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 20 20"><path fill="%234CAF50" d="M7 10l2 2 4-4"/></svg>');
  background-repeat: no-repeat;
  background-position: right 10px center;
  padding-right: 40px;
}

/* الحقول الخاطئة */
input.invalid {
  border-color: #f44336;
  background-image: url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 20 20"><path fill="%23f44336" d="M10 0C4.5 0 0 4.5 0 10s4.5 10 10 10 10-4.5 10-10S15.5 0 10 0zm1 15H9v-2h2v2zm0-4H9V5h2v6z"/></svg>');
  background-repeat: no-repeat;
  background-position: right 10px center;
  padding-right: 40px;
}

/* رسائل الخطأ */
.error {
  display: none;
  color: #f44336;
  font-size: 14px;
  margin-top: 5px;
  min-height: 20px;
}

/* Animation */
@keyframes shake {
  0%, 100% { transform: translateX(0); }
  10%, 30%, 50%, 70%, 90% { transform: translateX(-5px); }
  20%, 40%, 60%, 80% { transform: translateX(5px); }
}

input.invalid {
  animation: shake 0.5s;
}
```

---

## Validation في Backend

### 1. Node.js + Express (Validator Library)

```javascript
// ========================================
// تثبيت المكتبات
// ========================================
// npm install express-validator

const express = require('express');
const { body, validationResult } = require('express-validator');

const app = express();
app.use(express.json());

// ========================================
// Validation Rules
// ========================================

const registerValidationRules = [
  // اسم المستخدم
  body('username')
    .trim()
    .notEmpty().withMessage('اسم المستخدم مطلوب')
    .isLength({ min: 3, max: 20 }).withMessage('اسم المستخدم يجب أن يكون بين 3 و 20 حرف')
    .matches(/^[a-zA-Z0-9_]+$/).withMessage('اسم المستخدم يجب أن يحتوي على حروف وأرقام و _ فقط')
    .custom(async (value) => {
      // تحقق من التفرد في قاعدة البيانات
      const user = await User.findOne({ username: value });
      if (user) {
        throw new Error('اسم المستخدم موجود مسبقاً');
      }
      return true;
    }),

  // البريد الإلكتروني
  body('email')
    .trim()
    .notEmpty().withMessage('البريد الإلكتروني مطلوب')
    .isEmail().withMessage('البريد الإلكتروني غير صحيح')
    .normalizeEmail()
    .custom(async (value) => {
      const user = await User.findOne({ email: value });
      if (user) {
        throw new Error('البريد الإلكتروني مستخدم مسبقاً');
      }
      return true;
    }),

  // كلمة المرور
  body('password')
    .notEmpty().withMessage('كلمة المرور مطلوبة')
    .isLength({ min: 8 }).withMessage('كلمة المرور يجب أن تكون 8 أحرف على الأقل')
    .matches(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]{8,}$/)
    .withMessage('كلمة المرور يجب أن تحتوي على: حرف كبير، حرف صغير، رقم، رمز خاص'),

  // تأكيد كلمة المرور
  body('confirmPassword')
    .notEmpty().withMessage('تأكيد كلمة المرور مطلوب')
    .custom((value, { req }) => {
      if (value !== req.body.password) {
        throw new Error('كلمة المرور غير متطابقة');
      }
      return true;
    }),

  // العمر
  body('age')
    .notEmpty().withMessage('العمر مطلوب')
    .isInt({ min: 18, max: 120 }).withMessage('يجب أن يكون عمرك بين 18 و 120 سنة'),

  // رقم الهاتف
  body('phone')
    .notEmpty().withMessage('رقم الهاتف مطلوب')
    .matches(/^(\+966|00966|0)?5[0-9]{8}$/).withMessage('رقم الهاتف غير صحيح'),

  // تاريخ الميلاد
  body('birthdate')
    .notEmpty().withMessage('تاريخ الميلاد مطلوب')
    .isISO8601().withMessage('تاريخ غير صحيح')
    .custom((value) => {
      const birthDate = new Date(value);
      const today = new Date();
      const age = today.getFullYear() - birthDate.getFullYear();
      if (age < 18) {
        throw new Error('يجب أن يكون عمرك 18 سنة على الأقل');
      }
      return true;
    })
];

// ========================================
// Validation Middleware
// ========================================

const validate = (req, res, next) => {
  const errors = validationResult(req);

  if (!errors.isEmpty()) {
    // تنسيق الأخطاء
    const formattedErrors = {};
    errors.array().forEach(error => {
      formattedErrors[error.path] = error.msg;
    });

    return res.status(422).json({
      success: false,
      message: 'فشل التحقق من البيانات',
      errors: formattedErrors
    });
  }

  next();
};

// ========================================
// Routes
// ========================================

app.post('/api/register',
  registerValidationRules,
  validate,
  async (req, res) => {
    try {
      // البيانات صحيحة، يمكن المتابعة
      const { username, email, password, age, phone, birthdate } = req.body;

      // Hash password
      const hashedPassword = await bcrypt.hash(password, 10);

      // إنشاء المستخدم
      const user = await User.create({
        username,
        email,
        password: hashedPassword,
        age,
        phone,
        birthdate
      });

      res.status(201).json({
        success: true,
        message: 'تم التسجيل بنجاح',
        user: {
          id: user._id,
          username: user.username,
          email: user.email
        }
      });

    } catch (error) {
      console.error(error);
      res.status(500).json({
        success: false,
        message: 'حدث خطأ في السيرفر'
      });
    }
  }
);
```

---

### 2. Java + Spring Boot (Bean Validation)

```java
// ========================================
// Dependencies (pom.xml)
// ========================================
/*
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
*/

// ========================================
// DTO with Validation Annotations
// ========================================

import javax.validation.constraints.*;
import lombok.Data;

@Data
public class UserRegistrationDTO {

    @NotBlank(message = "اسم المستخدم مطلوب")
    @Size(min = 3, max = 20, message = "اسم المستخدم يجب أن يكون بين 3 و 20 حرف")
    @Pattern(
        regexp = "^[a-zA-Z0-9_]+$",
        message = "اسم المستخدم يجب أن يحتوي على حروف وأرقام و _ فقط"
    )
    private String username;

    @NotBlank(message = "البريد الإلكتروني مطلوب")
    @Email(message = "البريد الإلكتروني غير صحيح")
    private String email;

    @NotBlank(message = "كلمة المرور مطلوبة")
    @Size(min = 8, message = "كلمة المرور يجب أن تكون 8 أحرف على الأقل")
    @Pattern(
        regexp = "^(?=.*[a-z])(?=.*[A-Z])(?=.*\\d)(?=.*[@$!%*?&])[A-Za-z\\d@$!%*?&]{8,}$",
        message = "كلمة المرور يجب أن تحتوي على: حرف كبير، حرف صغير، رقم، رمز خاص"
    )
    private String password;

    @NotBlank(message = "تأكيد كلمة المرور مطلوب")
    private String confirmPassword;

    @NotNull(message = "العمر مطلوب")
    @Min(value = 18, message = "يجب أن يكون عمرك 18 سنة على الأقل")
    @Max(value = 120, message = "العمر غير صحيح")
    private Integer age;

    @NotBlank(message = "رقم الهاتف مطلوب")
    @Pattern(
        regexp = "^(\\+966|00966|0)?5[0-9]{8}$",
        message = "رقم الهاتف غير صحيح"
    )
    private String phone;

    @NotNull(message = "تاريخ الميلاد مطلوب")
    @Past(message = "تاريخ الميلاد يجب أن يكون في الماضي")
    private LocalDate birthdate;

    @AssertTrue(message = "يجب الموافقة على الشروط والأحكام")
    private Boolean termsAccepted;

    // Custom Validation: Password Match
    @AssertTrue(message = "كلمة المرور غير متطابقة")
    private boolean isPasswordMatching() {
        return password != null && password.equals(confirmPassword);
    }
}

// ========================================
// Custom Validators
// ========================================

// UniqueUsername Annotation
@Documented
@Constraint(validatedBy = UniqueUsernameValidator.class)
@Target({ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
public @interface UniqueUsername {
    String message() default "اسم المستخدم موجود مسبقاً";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}

// UniqueUsername Validator
@Component
public class UniqueUsernameValidator implements ConstraintValidator<UniqueUsername, String> {

    @Autowired
    private UserRepository userRepository;

    @Override
    public boolean isValid(String username, ConstraintValidatorContext context) {
        if (username == null) {
            return true; // @NotBlank will handle this
        }
        return !userRepository.existsByUsername(username);
    }
}

// استخدام Custom Validator
@Data
public class UserRegistrationDTO {
    @NotBlank
    @UniqueUsername
    private String username;

    @NotBlank
    @UniqueEmail
    private String email;

    // ... باقي الحقول
}

// ========================================
// Controller
// ========================================

@RestController
@RequestMapping("/api")
public class UserController {

    @Autowired
    private UserService userService;

    @PostMapping("/register")
    public ResponseEntity<?> register(
        @Valid @RequestBody UserRegistrationDTO dto,
        BindingResult bindingResult
    ) {

        // التحقق من الأخطاء
        if (bindingResult.hasErrors()) {
            Map<String, String> errors = new HashMap<>();

            bindingResult.getFieldErrors().forEach(error -> {
                errors.put(error.getField(), error.getDefaultMessage());
            });

            return ResponseEntity
                .status(HttpStatus.UNPROCESSABLE_ENTITY)
                .body(new ErrorResponse(
                    "فشل التحقق من البيانات",
                    errors
                ));
        }

        // البيانات صحيحة
        try {
            User user = userService.registerUser(dto);

            return ResponseEntity
                .status(HttpStatus.CREATED)
                .body(new SuccessResponse(
                    "تم التسجيل بنجاح",
                    user
                ));

        } catch (Exception e) {
            return ResponseEntity
                .status(HttpStatus.INTERNAL_SERVER_ERROR)
                .body(new ErrorResponse("حدث خطأ في السيرفر"));
        }
    }
}

// ========================================
// Global Exception Handler
// ========================================

@ControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<?> handleValidationExceptions(
        MethodArgumentNotValidException ex
    ) {
        Map<String, String> errors = new HashMap<>();

        ex.getBindingResult().getFieldErrors().forEach(error -> {
            errors.put(error.getField(), error.getDefaultMessage());
        });

        return ResponseEntity
            .status(HttpStatus.UNPROCESSABLE_ENTITY)
            .body(new ErrorResponse(
                "فشل التحقق من البيانات",
                errors
            ));
    }

    @ExceptionHandler(ConstraintViolationException.class)
    public ResponseEntity<?> handleConstraintViolation(
        ConstraintViolationException ex
    ) {
        Map<String, String> errors = new HashMap<>();

        ex.getConstraintViolations().forEach(violation -> {
            String fieldName = violation.getPropertyPath().toString();
            String message = violation.getMessage();
            errors.put(fieldName, message);
        });

        return ResponseEntity
            .status(HttpStatus.UNPROCESSABLE_ENTITY)
            .body(new ErrorResponse(
                "فشل التحقق من البيانات",
                errors
            ));
    }
}

// ========================================
// Response DTOs
// ========================================

@Data
@AllArgsConstructor
class ErrorResponse {
    private String message;
    private Map<String, String> errors;

    public ErrorResponse(String message) {
        this.message = message;
    }
}

@Data
@AllArgsConstructor
class SuccessResponse {
    private String message;
    private Object data;
}
```

---

### 3. PHP + Laravel (Validation)

```php
// ========================================
// Form Request Validation
// ========================================

<?php

namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;
use Illuminate\Validation\Rules\Password;

class UserRegistrationRequest extends FormRequest
{
    public function authorize()
    {
        return true;
    }

    public function rules()
    {
        return [
            'username' => [
                'required',
                'string',
                'min:3',
                'max:20',
                'regex:/^[a-zA-Z0-9_]+$/',
                'unique:users,username'
            ],

            'email' => [
                'required',
                'email',
                'unique:users,email'
            ],

            'password' => [
                'required',
                'confirmed',
                Password::min(8)
                    ->letters()
                    ->mixedCase()
                    ->numbers()
                    ->symbols()
            ],

            'age' => [
                'required',
                'integer',
                'min:18',
                'max:120'
            ],

            'phone' => [
                'required',
                'regex:/^(\+966|00966|0)?5[0-9]{8}$/'
            ],

            'birthdate' => [
                'required',
                'date',
                'before:18 years ago'
            ],

            'terms' => [
                'required',
                'accepted'
            ]
        ];
    }

    public function messages()
    {
        return [
            'username.required' => 'اسم المستخدم مطلوب',
            'username.min' => 'اسم المستخدم يجب أن يكون 3 أحرف على الأقل',
            'username.max' => 'اسم المستخدم يجب أن لا يتجاوز 20 حرف',
            'username.regex' => 'اسم المستخدم يجب أن يحتوي على حروف وأرقام و _ فقط',
            'username.unique' => 'اسم المستخدم موجود مسبقاً',

            'email.required' => 'البريد الإلكتروني مطلوب',
            'email.email' => 'البريد الإلكتروني غير صحيح',
            'email.unique' => 'البريد الإلكتروني مستخدم مسبقاً',

            'password.required' => 'كلمة المرور مطلوبة',
            'password.confirmed' => 'كلمة المرور غير متطابقة',

            'age.required' => 'العمر مطلوب',
            'age.min' => 'يجب أن يكون عمرك 18 سنة على الأقل',
            'age.max' => 'العمر غير صحيح',

            'phone.required' => 'رقم الهاتف مطلوب',
            'phone.regex' => 'رقم الهاتف غير صحيح',

            'birthdate.required' => 'تاريخ الميلاد مطلوب',
            'birthdate.date' => 'تاريخ غير صحيح',
            'birthdate.before' => 'يجب أن يكون عمرك 18 سنة على الأقل',

            'terms.required' => 'يجب الموافقة على الشروط والأحكام',
            'terms.accepted' => 'يجب الموافقة على الشروط والأحكام'
        ];
    }

    // Custom validation logic
    public function withValidator($validator)
    {
        $validator->after(function ($validator) {
            // مثال: التحقق من أن تاريخ الميلاد يطابق العمر
            if ($this->birthdate && $this->age) {
                $calculatedAge = now()->diffInYears($this->birthdate);
                if ($calculatedAge != $this->age) {
                    $validator->errors()->add(
                        'age',
                        'العمر لا يطابق تاريخ الميلاد'
                    );
                }
            }
        });
    }
}

// ========================================
// Controller
// ========================================

<?php

namespace App\Http\Controllers;

use App\Http\Requests\UserRegistrationRequest;
use App\Models\User;
use Illuminate\Support\Facades\Hash;

class UserController extends Controller
{
    public function register(UserRegistrationRequest $request)
    {
        // البيانات صحيحة تلقائياً (Laravel يتحقق قبل الوصول هنا)

        $validated = $request->validated();

        // إنشاء المستخدم
        $user = User::create([
            'username' => $validated['username'],
            'email' => $validated['email'],
            'password' => Hash::make($validated['password']),
            'age' => $validated['age'],
            'phone' => $validated['phone'],
            'birthdate' => $validated['birthdate']
        ]);

        return response()->json([
            'success' => true,
            'message' => 'تم التسجيل بنجاح',
            'user' => $user
        ], 201);
    }
}

// ========================================
// Custom Validation Rule
// ========================================

<?php

namespace App\Rules;

use Illuminate\Contracts\Validation\Rule;

class SaudiPhoneNumber implements Rule
{
    public function passes($attribute, $value)
    {
        // رقم سعودي: يبدأ بـ +966 أو 00966 أو 0
        // ثم 5 (للجوال)
        // ثم 8 أرقام
        return preg_match('/^(\+966|00966|0)?5[0-9]{8}$/', $value);
    }

    public function message()
    {
        return 'رقم الهاتف يجب أن يكون رقم سعودي صحيح';
    }
}

// الاستخدام:
'phone' => ['required', new SaudiPhoneNumber]
```

---

## Validation في قواعد البيانات

### 1. SQL Constraints

```sql
-- ========================================
-- جدول المستخدمين مع Constraints
-- ========================================

CREATE TABLE users (
    id INT PRIMARY KEY AUTO_INCREMENT,

    -- اسم المستخدم
    username VARCHAR(20) NOT NULL UNIQUE,

    -- البريد الإلكتروني
    email VARCHAR(255) NOT NULL UNIQUE,

    -- كلمة المرور (hashed)
    password VARCHAR(255) NOT NULL,

    -- العمر
    age INT NOT NULL,
    CHECK (age >= 18 AND age <= 120),

    -- رقم الهاتف
    phone VARCHAR(20) NOT NULL,

    -- تاريخ الميلاد
    birthdate DATE NOT NULL,
    CHECK (birthdate <= DATE_SUB(CURDATE(), INTERVAL 18 YEAR)),

    -- الحالة
    status ENUM('active', 'inactive', 'suspended') NOT NULL DEFAULT 'active',

    -- التاريخ
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

    -- Indexes
    INDEX idx_username (username),
    INDEX idx_email (email)
);

-- ========================================
-- جدول الطلبات مع Constraints
-- ========================================

CREATE TABLE orders (
    id INT PRIMARY KEY AUTO_INCREMENT,

    user_id INT NOT NULL,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,

    -- المبلغ
    amount DECIMAL(10, 2) NOT NULL,
    CHECK (amount > 0),

    -- الكمية
    quantity INT NOT NULL,
    CHECK (quantity > 0),

    -- حالة الطلب
    status ENUM('pending', 'processing', 'completed', 'cancelled') NOT NULL DEFAULT 'pending',

    -- التواريخ
    order_date TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    delivery_date TIMESTAMP NULL,
    CHECK (delivery_date IS NULL OR delivery_date > order_date),

    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

-- ========================================
-- Triggers للتحقق المعقد
-- ========================================

-- Trigger: التحقق من العمر قبل الإدخال
DELIMITER $$

CREATE TRIGGER validate_user_age_before_insert
BEFORE INSERT ON users
FOR EACH ROW
BEGIN
    DECLARE calculated_age INT;

    SET calculated_age = TIMESTAMPDIFF(YEAR, NEW.birthdate, CURDATE());

    IF calculated_age != NEW.age THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'العمر لا يطابق تاريخ الميلاد';
    END IF;

    IF calculated_age < 18 THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'يجب أن يكون عمرك 18 سنة على الأقل';
    END IF;
END$$

DELIMITER ;

-- Trigger: التحقق من تاريخ التسليم
DELIMITER $$

CREATE TRIGGER validate_delivery_date_before_insert
BEFORE INSERT ON orders
FOR EACH ROW
BEGIN
    IF NEW.delivery_date IS NOT NULL AND NEW.delivery_date <= NEW.order_date THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'تاريخ التسليم يجب أن يكون بعد تاريخ الطلب';
    END IF;
END$$

DELIMITER ;
```

---

### 2. MongoDB Validation

```javascript
// ========================================
// MongoDB Schema Validation
// ========================================

db.createCollection("users", {
  validator: {
    $jsonSchema: {
      bsonType: "object",
      required: ["username", "email", "password", "age", "phone", "birthdate"],
      properties: {
        username: {
          bsonType: "string",
          minLength: 3,
          maxLength: 20,
          pattern: "^[a-zA-Z0-9_]+$",
          description: "اسم المستخدم مطلوب ويجب أن يكون بين 3-20 حرف"
        },
        email: {
          bsonType: "string",
          pattern: "^[^\\s@]+@[^\\s@]+\\.[^\\s@]+$",
          description: "البريد الإلكتروني مطلوب ويجب أن يكون صحيح"
        },
        password: {
          bsonType: "string",
          minLength: 8,
          description: "كلمة المرور مطلوبة ويجب أن تكون 8 أحرف على الأقل"
        },
        age: {
          bsonType: "int",
          minimum: 18,
          maximum: 120,
          description: "العمر مطلوب ويجب أن يكون بين 18-120"
        },
        phone: {
          bsonType: "string",
          pattern: "^(\\+966|00966|0)?5[0-9]{8}$",
          description: "رقم الهاتف مطلوب ويجب أن يكون رقم سعودي صحيح"
        },
        birthdate: {
          bsonType: "date",
          description: "تاريخ الميلاد مطلوب"
        },
        status: {
          enum: ["active", "inactive", "suspended"],
          description: "الحالة يجب أن تكون active أو inactive أو suspended"
        }
      }
    }
  },
  validationLevel: "strict",
  validationAction: "error"
});

// Unique Indexes
db.users.createIndex({ username: 1 }, { unique: true });
db.users.createIndex({ email: 1 }, { unique: true });
```

---

## معالجة أخطاء Validation

### 1. تنسيق موحد للأخطاء

```javascript
// ========================================
// Error Response Format
// ========================================

{
  "success": false,
  "message": "فشل التحقق من البيانات",
  "errors": {
    "username": "اسم المستخدم موجود مسبقاً",
    "email": "البريد الإلكتروني غير صحيح",
    "password": "كلمة المرور يجب أن تكون 8 أحرف على الأقل"
  },
  "timestamp": "2025-01-04T18:30:00Z",
  "path": "/api/register"
}
```

---

### 2. عرض الأخطاء في الواجهة

```javascript
// ========================================
// عرض الأخطاء من السيرفر
// ========================================

async function handleFormSubmit(formData) {
  try {
    const response = await fetch('/api/register', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(formData)
    });

    const result = await response.json();

    if (!response.ok) {
      // عرض الأخطاء
      if (result.errors) {
        Object.keys(result.errors).forEach(fieldName => {
          showFieldError(fieldName, result.errors[fieldName]);
        });
      } else {
        showGlobalError(result.message);
      }
    } else {
      // نجح
      showSuccessMessage(result.message);
      redirectToSuccess();
    }

  } catch (error) {
    showGlobalError('حدث خطأ، يرجى المحاولة لاحقاً');
  }
}

function showFieldError(fieldName, message) {
  const field = document.getElementById(fieldName);
  const errorElement = document.getElementById(`${fieldName}-error`);

  field.classList.add('invalid');
  errorElement.textContent = message;
  errorElement.style.display = 'block';
}

function showGlobalError(message) {
  const toast = document.createElement('div');
  toast.className = 'toast error';
  toast.textContent = message;
  document.body.appendChild(toast);

  setTimeout(() => {
    toast.remove();
  }, 5000);
}
```

---

## أمثلة عملية شاملة

### مثال كامل: نموذج طلب قرض

```html
<!-- ======================================== -->
<!-- HTML Form -->
<!-- ======================================== -->

<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <title>طلب قرض</title>
    <link rel="stylesheet" href="loan-form.css">
</head>
<body>

<div class="container">
    <h1>طلب قرض بنكي</h1>

    <form id="loanForm" novalidate>

        <!-- معلومات شخصية -->
        <fieldset>
            <legend>معلومات شخصية</legend>

            <div class="form-group">
                <label for="fullName">الاسم الكامل *</label>
                <input type="text" id="fullName" name="fullName" required>
                <span class="error" id="fullName-error"></span>
            </div>

            <div class="form-group">
                <label for="nationalId">رقم الهوية *</label>
                <input type="text" id="nationalId" name="nationalId" required>
                <span class="error" id="nationalId-error"></span>
                <span class="help-text">10 أرقام (مثال: 1234567890)</span>
            </div>

            <div class="form-group">
                <label for="birthdate">تاريخ الميلاد *</label>
                <input type="date" id="birthdate" name="birthdate" required>
                <span class="error" id="birthdate-error"></span>
            </div>

            <div class="form-group">
                <label for="email">البريد الإلكتروني *</label>
                <input type="email" id="email" name="email" required>
                <span class="error" id="email-error"></span>
            </div>

            <div class="form-group">
                <label for="phone">رقم الجوال *</label>
                <input type="tel" id="phone" name="phone" required>
                <span class="error" id="phone-error"></span>
            </div>
        </fieldset>

        <!-- معلومات مالية -->
        <fieldset>
            <legend>معلومات مالية</legend>

            <div class="form-group">
                <label for="monthlySalary">الراتب الشهري (ريال) *</label>
                <input type="number" id="monthlySalary" name="monthlySalary" required>
                <span class="error" id="monthlySalary-error"></span>
            </div>

            <div class="form-group">
                <label for="employmentType">نوع الوظيفة *</label>
                <select id="employmentType" name="employmentType" required>
                    <option value="">-- اختر --</option>
                    <option value="government">حكومي</option>
                    <option value="private">قطاع خاص</option>
                    <option value="self-employed">عمل حر</option>
                    <option value="retired">متقاعد</option>
                </select>
                <span class="error" id="employmentType-error"></span>
            </div>

            <div class="form-group">
                <label for="employmentDuration">مدة العمل الحالية (أشهر) *</label>
                <input type="number" id="employmentDuration" name="employmentDuration" required>
                <span class="error" id="employmentDuration-error"></span>
            </div>

            <div class="form-group">
                <label for="currentDebts">الالتزامات الشهرية الحالية (ريال) *</label>
                <input type="number" id="currentDebts" name="currentDebts" required>
                <span class="error" id="currentDebts-error"></span>
            </div>
        </fieldset>

        <!-- معلومات القرض -->
        <fieldset>
            <legend>معلومات القرض</legend>

            <div class="form-group">
                <label for="loanType">نوع القرض *</label>
                <select id="loanType" name="loanType" required>
                    <option value="">-- اختر --</option>
                    <option value="personal">قرض شخصي</option>
                    <option value="car">قرض سيارة</option>
                    <option value="real-estate">قرض عقاري</option>
                </select>
                <span class="error" id="loanType-error"></span>
            </div>

            <div class="form-group">
                <label for="loanAmount">مبلغ القرض المطلوب (ريال) *</label>
                <input type="number" id="loanAmount" name="loanAmount" required>
                <span class="error" id="loanAmount-error"></span>
            </div>

            <div class="form-group">
                <label for="loanDuration">مدة القرض (أشهر) *</label>
                <input type="number" id="loanDuration" name="loanDuration" required>
                <span class="error" id="loanDuration-error"></span>
            </div>
        </fieldset>

        <!-- معلومات حسابية تلقائية -->
        <div class="info-box">
            <h3>معلومات القرض المحسوبة</h3>
            <div class="info-row">
                <span>القسط الشهري التقريبي:</span>
                <strong id="monthlyPayment">---</strong>
            </div>
            <div class="info-row">
                <span>نسبة الالتزامات من الدخل:</span>
                <strong id="debtRatio">---</strong>
            </div>
            <div class="info-row">
                <span>الحالة:</span>
                <strong id="eligibilityStatus">---</strong>
            </div>
        </div>

        <div class="form-group">
            <label>
                <input type="checkbox" id="terms" name="terms" required>
                أوافق على الشروط والأحكام *
            </label>
            <span class="error" id="terms-error"></span>
        </div>

        <button type="submit" class="submit-btn">تقديم الطلب</button>

    </form>
</div>

<script src="loan-form.js"></script>
</body>
</html>
```

---

```javascript
// ======================================== //
// JavaScript Validation
// ======================================== //

// Validation Rules للقرض
const loanValidationRules = {
  fullName: {
    required: true,
    minLength: 5,
    pattern: /^[\u0600-\u06FF\s]+$/,  // عربي فقط
    messages: {
      required: 'الاسم الكامل مطلوب',
      minLength: 'الاسم يجب أن يكون 5 أحرف على الأقل',
      pattern: 'الاسم يجب أن يكون بالعربي فقط'
    }
  },

  nationalId: {
    required: true,
    pattern: /^[12]\d{9}$/,  // رقم هوية سعودي
    messages: {
      required: 'رقم الهوية مطلوب',
      pattern: 'رقم الهوية يجب أن يكون 10 أرقام ويبدأ بـ 1 أو 2'
    }
  },

  birthdate: {
    required: true,
    customValidator: (value) => {
      const birthDate = new Date(value);
      const today = new Date();
      const age = today.getFullYear() - birthDate.getFullYear();

      if (age < 21) {
        return 'يجب أن يكون عمرك 21 سنة على الأقل';
      }
      if (age > 65) {
        return 'لا يمكن منح قرض لمن هم أكبر من 65 سنة';
      }
      return null;
    },
    messages: {
      required: 'تاريخ الميلاد مطلوب'
    }
  },

  email: {
    required: true,
    pattern: /^[^\s@]+@[^\s@]+\.[^\s@]+$/,
    messages: {
      required: 'البريد الإلكتروني مطلوب',
      pattern: 'البريد الإلكتروني غير صحيح'
    }
  },

  phone: {
    required: true,
    pattern: /^(\+966|00966|0)?5[0-9]{8}$/,
    messages: {
      required: 'رقم الجوال مطلوب',
      pattern: 'رقم الجوال غير صحيح (مثال: 0501234567)'
    }
  },

  monthlySalary: {
    required: true,
    min: 5000,
    messages: {
      required: 'الراتب الشهري مطلوب',
      min: 'الراتب الشهري يجب أن يكون 5000 ريال على الأقل'
    }
  },

  employmentType: {
    required: true,
    messages: {
      required: 'نوع الوظيفة مطلوب'
    }
  },

  employmentDuration: {
    required: true,
    min: 6,
    messages: {
      required: 'مدة العمل الحالية مطلوبة',
      min: 'مدة العمل يجب أن تكون 6 أشهر على الأقل'
    }
  },

  currentDebts: {
    required: true,
    min: 0,
    customValidator: (value, form) => {
      const salary = parseFloat(form.elements.monthlySalary.value);
      const debts = parseFloat(value);

      if (debts > salary * 0.5) {
        return 'الالتزامات الحالية عالية جداً (أكثر من 50% من الراتب)';
      }
      return null;
    },
    messages: {
      required: 'الالتزامات الشهرية الحالية مطلوبة',
      min: 'القيمة يجب أن تكون صفر أو أكثر'
    }
  },

  loanType: {
    required: true,
    messages: {
      required: 'نوع القرض مطلوب'
    }
  },

  loanAmount: {
    required: true,
    min: 10000,
    max: 2000000,
    customValidator: (value, form) => {
      const salary = parseFloat(form.elements.monthlySalary.value);
      const amount = parseFloat(value);

      // الحد الأقصى للقرض: 30 ضعف الراتب
      if (amount > salary * 30) {
        return `مبلغ القرض يجب أن لا يتجاوز ${salary * 30} ريال (30 ضعف راتبك)`;
      }
      return null;
    },
    messages: {
      required: 'مبلغ القرض مطلوب',
      min: 'الحد الأدنى للقرض 10,000 ريال',
      max: 'الحد الأقصى للقرض 2,000,000 ريال'
    }
  },

  loanDuration: {
    required: true,
    min: 6,
    max: 240,
    messages: {
      required: 'مدة القرض مطلوبة',
      min: 'الحد الأدنى للمدة 6 أشهر',
      max: 'الحد الأقصى للمدة 240 شهر (20 سنة)'
    }
  },

  terms: {
    required: true,
    checked: true,
    messages: {
      required: 'يجب الموافقة على الشروط والأحكام'
    }
  }
};

// الحسابات التلقائية
function calculateLoanInfo() {
  const salary = parseFloat(document.getElementById('monthlySalary').value) || 0;
  const currentDebts = parseFloat(document.getElementById('currentDebts').value) || 0;
  const loanAmount = parseFloat(document.getElementById('loanAmount').value) || 0;
  const loanDuration = parseFloat(document.getElementById('loanDuration').value) || 0;

  // حساب القسط الشهري (تقريبي - فائدة 5%)
  const interestRate = 0.05 / 12;  // 5% سنوياً
  let monthlyPayment = 0;

  if (loanAmount > 0 && loanDuration > 0) {
    monthlyPayment = loanAmount * (interestRate * Math.pow(1 + interestRate, loanDuration)) /
                     (Math.pow(1 + interestRate, loanDuration) - 1);
  }

  // حساب نسبة الالتزامات
  const totalDebts = currentDebts + monthlyPayment;
  const debtRatio = salary > 0 ? (totalDebts / salary) * 100 : 0;

  // تحديد الأهلية
  let eligibility = '';
  let eligibilityClass = '';

  if (salary === 0 || loanAmount === 0) {
    eligibility = '---';
  } else if (debtRatio <= 33) {
    eligibility = '✅ مؤهل - فرصة موافقة عالية';
    eligibilityClass = 'eligible';
  } else if (debtRatio <= 40) {
    eligibility = '⚠️ مؤهل مشروط - يحتاج مراجعة';
    eligibilityClass = 'conditional';
  } else {
    eligibility = '❌ غير مؤهل - نسبة الالتزامات عالية';
    eligibilityClass = 'not-eligible';
  }

  // عرض النتائج
  document.getElementById('monthlyPayment').textContent =
    monthlyPayment > 0 ? monthlyPayment.toFixed(2) + ' ريال' : '---';

  document.getElementById('debtRatio').textContent =
    salary > 0 ? debtRatio.toFixed(1) + '%' : '---';

  const statusElement = document.getElementById('eligibilityStatus');
  statusElement.textContent = eligibility;
  statusElement.className = eligibilityClass;
}

// ربط الحسابات بالحقول
['monthlySalary', 'currentDebts', 'loanAmount', 'loanDuration'].forEach(fieldName => {
  document.getElementById(fieldName).addEventListener('input', calculateLoanInfo);
});

// تهيئة الـ Validator
document.addEventListener('DOMContentLoaded', () => {
  const validator = new FormValidator('loanForm', loanValidationRules);
});
```

---

## أفضل الممارسات

### 1. Validate على كل المستويات

```
✅ Client + Server + Database

❌ Client فقط (غير آمن)
❌ Server فقط (تجربة مستخدم سيئة)
```

---

### 2. رسائل خطأ واضحة

```
✅ جيد:
"اسم المستخدم يجب أن يكون بين 3 و 20 حرف"
"البريد الإلكتروني غير صحيح (مثال: user@example.com)"

❌ سيء:
"خطأ في الحقل"
"Invalid input"
"Error 422"
```

---

### 3. Validation فوري

```javascript
// ✅ Validate عند الكتابة أو فقدان التركيز
field.addEventListener('blur', () => validate());

// ❌ Validate فقط عند الإرسال
```

---

### 4. Sanitization + Validation

```javascript
// ✅ نظّف ثم تحقق
const cleanInput = input.trim().toLowerCase();
validate(cleanInput);

// ❌ تحقق بدون تنظيف
validate(input);
```

---

### 5. توحيد القواعد

```
✅ استخدم نفس القواعد في:
- Client-side
- Server-side
- Database

❌ قواعد مختلفة في كل مكان
```

---

## الخلاصة

### الخطوات الأساسية لـ Validation صحيح:

```
1. Client-Side Validation
   ↓ (تجربة مستخدم)

2. Server-Side Validation
   ↓ (الأمان الحقيقي)

3. Database Constraints
   ↓ (تكامل البيانات)

4. رسائل خطأ واضحة
   ↓ (تجربة مستخدم)

5. Logging & Monitoring
   ↓ (تتبع المشاكل)
```

---

**Validation = أمان + تكامل بيانات + تجربة مستخدم جيدة!**

لا تستهن أبداً بأهمية التحقق من البيانات - هو خط الدفاع الأول ضد البيانات الفاسدة والثغرات الأمنية! 🛡️

---

**تم إنشاء هذا الدليل كمرجع شامل لمعالجة Validation في جميع الطبقات**
