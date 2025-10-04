# Angular Interview Questions - أسئلة ومقابلات Angular الشاملة

## جدول المحتويات
1. [أسئلة المستوى المبتدئ](#أسئلة-المستوى-المبتدئ)
2. [أسئلة المستوى المتوسط](#أسئلة-المستوى-المتوسط)
3. [أسئلة المستوى المتقدم](#أسئلة-المستوى-المتقدم)
4. [أسئلة RxJS](#أسئلة-rxjs)
5. [أسئلة State Management](#أسئلة-state-management)
6. [أسئلة Performance](#أسئلة-performance)
7. [أسئلة Testing](#أسئلة-testing)
8. [أسئلة عملية وكود](#أسئلة-عملية-وكود)

---

## أسئلة المستوى المبتدئ

### السؤال 1: ما هو Angular؟

**الإجابة:**

Angular هو **framework** مفتوح المصدر من Google لبناء تطبيقات ويب ديناميكية.

**الخصائص:**
- ✅ TypeScript-based
- ✅ Component-based architecture
- ✅ Two-way data binding
- ✅ Dependency Injection
- ✅ Routing
- ✅ RxJS للبرمجة التفاعلية

**الفرق عن AngularJS:**

| المعيار | AngularJS (1.x) | Angular (2+) |
|--------|-----------------|--------------|
| اللغة | JavaScript | TypeScript |
| البنية | MVC | Component-based |
| الأداء | أبطأ | أسرع |
| Mobile | لا | نعم (Ionic) |
| الدعم | انتهى 2022 | مستمر |

---

### السؤال 2: ما هي المكونات الأساسية في Angular؟

**الإجابة:**

```
Angular Building Blocks
│
├── 1. Components
│   └── العرض + المنطق
│
├── 2. Modules (NgModules)
│   └── تنظيم التطبيق
│
├── 3. Services
│   └── منطق العمل + مشاركة البيانات
│
├── 4. Directives
│   ├── Structural (*ngIf, *ngFor)
│   └── Attribute ([ngClass], [ngStyle])
│
├── 5. Pipes
│   └── تحويل البيانات في Templates
│
├── 6. Routing
│   └── التنقل بين الصفحات
│
└── 7. Dependency Injection
    └── حقن التبعيات
```

---

### السؤال 3: ما هو Component في Angular؟

**الإجابة:**

**Component** هو وحدة بناء أساسية في Angular، يتكون من:

**الأجزاء:**
```
Component
│
├── TypeScript Class (.ts)
│   └── المنطق والبيانات
│
├── HTML Template (.html)
│   └── العرض
│
├── CSS Styles (.css/.scss)
│   └── التنسيق
│
└── Metadata (@Component decorator)
    └── التكوين
```

**مثال:**

```typescript
// user.component.ts
import { Component } from '@angular/core';

@Component({
  selector: 'app-user',              // كيف نستخدمه في HTML
  templateUrl: './user.component.html',  // ملف HTML
  styleUrls: ['./user.component.scss']   // ملف CSS
})
export class UserComponent {
  // Properties
  name: string = 'أحمد محمد';
  age: number = 25;
  email: string = 'ahmed@example.com';

  // Methods
  greet(): void {
    console.log(`مرحباً ${this.name}`);
  }

  updateName(newName: string): void {
    this.name = newName;
  }
}
```

```html
<!-- user.component.html -->
<div class="user-card">
  <h2>{{ name }}</h2>
  <p>العمر: {{ age }}</p>
  <p>البريد: {{ email }}</p>
  <button (click)="greet()">تحية</button>
</div>
```

```scss
// user.component.scss
.user-card {
  border: 1px solid #ddd;
  padding: 20px;
  border-radius: 8px;

  h2 {
    color: #333;
    margin-bottom: 10px;
  }

  button {
    background: #007bff;
    color: white;
    padding: 10px 20px;
    border: none;
    border-radius: 4px;
    cursor: pointer;

    &:hover {
      background: #0056b3;
    }
  }
}
```

---

### السؤال 4: ما الفرق بين Component و Directive؟

**الإجابة:**

| المعيار | Component | Directive |
|--------|-----------|-----------|
| **التعريف** | مكون مع Template | تعليمات بدون Template |
| **@Decorator** | @Component | @Directive |
| **Template** | إجباري | لا يوجد |
| **Selector** | Element | Attribute عادةً |
| **الاستخدام** | `<app-user></app-user>` | `<div appHighlight></div>` |

**أنواع Directives:**

**1. Structural Directives (تغير DOM)**
```typescript
// *ngIf
<div *ngIf="isLoggedIn">
  مرحباً بعودتك!
</div>

// *ngFor
<ul>
  <li *ngFor="let user of users">{{ user.name }}</li>
</ul>

// *ngSwitch
<div [ngSwitch]="color">
  <p *ngSwitchCase="'red'">أحمر</p>
  <p *ngSwitchCase="'blue'">أزرق</p>
  <p *ngSwitchDefault>لون آخر</p>
</div>
```

**2. Attribute Directives (تغير مظهر/سلوك)**
```typescript
// ngClass
<div [ngClass]="{'active': isActive, 'disabled': isDisabled}">

// ngStyle
<div [ngStyle]="{'color': textColor, 'font-size': fontSize + 'px'}">

// Custom Directive
@Directive({
  selector: '[appHighlight]'
})
export class HighlightDirective {
  constructor(private el: ElementRef) {
    this.el.nativeElement.style.backgroundColor = 'yellow';
  }
}
```

---

### السؤال 5: ما هو Data Binding؟

**الإجابة:**

**Data Binding** هو الربط بين Component Class والـ Template.

**الأنواع:**

**1. Interpolation (عرض البيانات) - من Component للـ Template**
```typescript
// Component
export class UserComponent {
  name = 'أحمد';
  age = 25;
}

// Template
<h1>{{ name }}</h1>
<p>العمر: {{ age }}</p>
<p>{{ age * 2 }}</p>
```

**2. Property Binding (ربط الخصائص) - من Component للـ Template**
```typescript
// Component
export class AppComponent {
  imageUrl = 'assets/logo.png';
  isDisabled = false;
}

// Template
<img [src]="imageUrl">
<button [disabled]="isDisabled">انقر</button>
```

**3. Event Binding (ربط الأحداث) - من Template للـ Component**
```typescript
// Component
export class AppComponent {
  onClick(): void {
    console.log('تم النقر!');
  }

  onInput(event: Event): void {
    const value = (event.target as HTMLInputElement).value;
    console.log(value);
  }
}

// Template
<button (click)="onClick()">انقر</button>
<input (input)="onInput($event)">
```

**4. Two-Way Binding (اتجاهين) - بين Component والـ Template**
```typescript
// Component
export class AppComponent {
  username = 'أحمد';
}

// Template
<input [(ngModel)]="username">
<p>مرحباً {{ username }}</p>

// Note: يحتاج FormsModule
import { FormsModule } from '@angular/forms';
```

---

### السؤال 6: ما هو Service في Angular؟

**الإجابة:**

**Service** هو class يحتوي على:
- منطق العمل (Business Logic)
- مشاركة البيانات بين Components
- استدعاء APIs
- أي كود قابل لإعادة الاستخدام

**مثال:**

```typescript
// user.service.ts
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';

@Injectable({
  providedIn: 'root'  // Singleton - نسخة واحدة في التطبيق
})
export class UserService {

  private apiUrl = 'https://api.example.com/users';

  constructor(private http: HttpClient) { }

  // جلب جميع المستخدمين
  getUsers(): Observable<User[]> {
    return this.http.get<User[]>(this.apiUrl);
  }

  // جلب مستخدم واحد
  getUserById(id: number): Observable<User> {
    return this.http.get<User>(`${this.apiUrl}/${id}`);
  }

  // إنشاء مستخدم
  createUser(user: User): Observable<User> {
    return this.http.post<User>(this.apiUrl, user);
  }

  // تحديث مستخدم
  updateUser(id: number, user: User): Observable<User> {
    return this.http.put<User>(`${this.apiUrl}/${id}`, user);
  }

  // حذف مستخدم
  deleteUser(id: number): Observable<void> {
    return this.http.delete<void>(`${this.apiUrl}/${id}`);
  }
}

interface User {
  id?: number;
  name: string;
  email: string;
  age: number;
}
```

**الاستخدام:**

```typescript
// user-list.component.ts
import { Component, OnInit } from '@angular/core';
import { UserService } from './user.service';

export class UserListComponent implements OnInit {
  users: User[] = [];
  loading = false;
  error: string | null = null;

  constructor(private userService: UserService) { }

  ngOnInit(): void {
    this.loadUsers();
  }

  loadUsers(): void {
    this.loading = true;
    this.userService.getUsers().subscribe({
      next: (users) => {
        this.users = users;
        this.loading = false;
      },
      error: (error) => {
        this.error = 'حدث خطأ في تحميل البيانات';
        this.loading = false;
        console.error(error);
      }
    });
  }

  deleteUser(id: number): void {
    if (confirm('هل أنت متأكد؟')) {
      this.userService.deleteUser(id).subscribe({
        next: () => {
          this.users = this.users.filter(u => u.id !== id);
        },
        error: (error) => {
          this.error = 'فشل الحذف';
          console.error(error);
        }
      });
    }
  }
}
```

---

### السؤال 7: ما هو Dependency Injection؟

**الإجابة:**

**Dependency Injection (DI)** هو نمط تصميم حيث Angular يحقن التبعيات تلقائياً.

**الفوائد:**
- ✅ Loose Coupling
- ✅ سهولة الاختبار
- ✅ إعادة الاستخدام
- ✅ إدارة أفضل للتبعيات

**مثال:**

```typescript
// بدون DI ❌
export class UserComponent {
  private userService: UserService;

  constructor() {
    this.userService = new UserService(); // Tight coupling
  }
}

// مع DI ✅
export class UserComponent {
  constructor(private userService: UserService) {
    // Angular يحقن UserService تلقائياً
  }
}
```

**مستويات Providers:**

```typescript
// 1. Root Level (نسخة واحدة في التطبيق)
@Injectable({
  providedIn: 'root'
})
export class UserService { }

// 2. Module Level (نسخة لكل Module)
@NgModule({
  providers: [UserService]
})
export class UserModule { }

// 3. Component Level (نسخة لكل Component)
@Component({
  selector: 'app-user',
  providers: [UserService]
})
export class UserComponent { }
```

---

### السؤال 8: ما الفرق بين constructor و ngOnInit؟

**الإجابة:**

| المعيار | constructor | ngOnInit |
|--------|-------------|----------|
| **المعنى** | JavaScript constructor | Angular lifecycle hook |
| **التوقيت** | قبل Angular | بعد تهيئة Component |
| **الاستخدام** | DI فقط | منطق التهيئة |
| **Data Binding** | غير جاهز | جاهز |
| **Input Properties** | غير متاحة | متاحة |

**مثال:**

```typescript
export class UserComponent implements OnInit {
  @Input() userId!: number;
  user: User | null = null;

  constructor(private userService: UserService) {
    // ✅ DI فقط
    console.log('Constructor called');

    // ❌ لا تستخدم هنا
    // this.userId - غير متاح بعد
    // this.loadUser() - ممكن لكن غير مفضل
  }

  ngOnInit(): void {
    // ✅ منطق التهيئة
    console.log('ngOnInit called');
    console.log('User ID:', this.userId); // متاح الآن

    // ✅ استدعاء APIs
    this.loadUser();
  }

  loadUser(): void {
    this.userService.getUserById(this.userId).subscribe(
      user => this.user = user
    );
  }
}
```

---

## أسئلة المستوى المتوسط

### السؤال 9: اشرح Component Lifecycle Hooks

**الإجابة:**

**Angular Component Lifecycle:**

```
Component Lifecycle Hooks (بالترتيب)
│
├── 1. constructor()
│   └── JavaScript constructor
│
├── 2. ngOnChanges()
│   └── عند تغيير @Input properties
│
├── 3. ngOnInit()
│   └── بعد أول ngOnChanges
│
├── 4. ngDoCheck()
│   └── كل دورة change detection
│
├── 5. ngAfterContentInit()
│   └── بعد إدراج ng-content
│
├── 6. ngAfterContentChecked()
│   └── بعد فحص ng-content
│
├── 7. ngAfterViewInit()
│   └── بعد تهيئة View
│
├── 8. ngAfterViewChecked()
│   └── بعد فحص View
│
└── 9. ngOnDestroy()
    └── قبل تدمير Component
```

**الأكثر استخداماً:**

```typescript
import { Component, OnInit, OnDestroy, OnChanges,
         SimpleChanges, Input } from '@angular/core';
import { Subscription } from 'rxjs';

export class UserComponent implements OnInit, OnChanges, OnDestroy {
  @Input() userId!: number;

  private subscription!: Subscription;

  // 1. ngOnChanges - عند تغيير Input
  ngOnChanges(changes: SimpleChanges): void {
    console.log('Changes:', changes);

    if (changes['userId']) {
      const previousValue = changes['userId'].previousValue;
      const currentValue = changes['userId'].currentValue;
      console.log(`userId changed from ${previousValue} to ${currentValue}`);

      if (!changes['userId'].firstChange) {
        // ليس التغيير الأول
        this.loadUser(currentValue);
      }
    }
  }

  // 2. ngOnInit - التهيئة
  ngOnInit(): void {
    console.log('Component initialized');
    this.loadUser(this.userId);

    // Subscribe to observables
    this.subscription = this.userService.getUsers().subscribe(
      users => console.log(users)
    );
  }

  // 3. ngOnDestroy - التنظيف
  ngOnDestroy(): void {
    console.log('Component destroyed');

    // ✅ Unsubscribe لتجنب memory leaks
    if (this.subscription) {
      this.subscription.unsubscribe();
    }

    // إلغاء timers, intervals, etc.
  }

  loadUser(id: number): void {
    // ...
  }
}
```

---

### السؤال 10: ما هو ViewChild و ContentChild؟

**الإجابة:**

**ViewChild:** الوصول لعناصر في Template الخاص بـ Component

**ContentChild:** الوصول لعناصر مُدرجة عبر `<ng-content>`

**مثال:**

```typescript
// ========================================
// ViewChild
// ========================================

// parent.component.ts
import { Component, ViewChild, AfterViewInit, ElementRef } from '@angular/core';
import { ChildComponent } from './child.component';

@Component({
  selector: 'app-parent',
  template: `
    <input #nameInput type="text">
    <app-child></app-child>
    <button (click)="callChild()">Call Child</button>
  `
})
export class ParentComponent implements AfterViewInit {
  // الوصول لـ input element
  @ViewChild('nameInput') nameInput!: ElementRef;

  // الوصول لـ child component
  @ViewChild(ChildComponent) childComponent!: ChildComponent;

  ngAfterViewInit(): void {
    // ✅ متاح بعد View initialization
    console.log(this.nameInput.nativeElement.value);
    this.nameInput.nativeElement.focus();
  }

  callChild(): void {
    this.childComponent.childMethod();
  }
}

// ========================================
// ContentChild
// ========================================

// card.component.ts
@Component({
  selector: 'app-card',
  template: `
    <div class="card">
      <div class="card-header">
        <ng-content select="[header]"></ng-content>
      </div>
      <div class="card-body">
        <ng-content></ng-content>
      </div>
    </div>
  `
})
export class CardComponent implements AfterContentInit {
  @ContentChild('headerContent') headerContent!: ElementRef;

  ngAfterContentInit(): void {
    console.log('Header:', this.headerContent.nativeElement.textContent);
  }
}

// الاستخدام:
<app-card>
  <h2 header #headerContent>العنوان</h2>
  <p>المحتوى هنا</p>
</app-card>
```

---

### السؤال 11: ما الفرق بين Template-driven Forms و Reactive Forms؟

**الإجابة:**

| المعيار | Template-driven | Reactive |
|--------|-----------------|----------|
| **التعريف** | في HTML | في TypeScript |
| **التعقيد** | بسيط | معقد |
| **Validation** | Directives | Code |
| **Testing** | صعب | سهل |
| **الاستخدام** | نماذج بسيطة | نماذج معقدة |

**1. Template-driven Forms:**

```typescript
// app.module.ts
import { FormsModule } from '@angular/forms';

@NgModule({
  imports: [FormsModule]
})

// login.component.ts
export class LoginComponent {
  user = {
    email: '',
    password: ''
  };

  onSubmit(): void {
    console.log(this.user);
  }
}
```

```html
<!-- login.component.html -->
<form #loginForm="ngForm" (ngSubmit)="onSubmit()">

  <div>
    <label>البريد الإلكتروني:</label>
    <input
      type="email"
      name="email"
      [(ngModel)]="user.email"
      required
      email
      #emailField="ngModel">

    <div *ngIf="emailField.invalid && emailField.touched">
      <small *ngIf="emailField.errors?.['required']">
        البريد الإلكتروني مطلوب
      </small>
      <small *ngIf="emailField.errors?.['email']">
        البريد الإلكتروني غير صحيح
      </small>
    </div>
  </div>

  <div>
    <label>كلمة المرور:</label>
    <input
      type="password"
      name="password"
      [(ngModel)]="user.password"
      required
      minlength="8"
      #passwordField="ngModel">

    <div *ngIf="passwordField.invalid && passwordField.touched">
      <small *ngIf="passwordField.errors?.['required']">
        كلمة المرور مطلوبة
      </small>
      <small *ngIf="passwordField.errors?.['minlength']">
        كلمة المرور يجب أن تكون 8 أحرف على الأقل
      </small>
    </div>
  </div>

  <button type="submit" [disabled]="loginForm.invalid">
    تسجيل الدخول
  </button>

</form>
```

**2. Reactive Forms:**

```typescript
// app.module.ts
import { ReactiveFormsModule } from '@angular/forms';

@NgModule({
  imports: [ReactiveFormsModule]
})

// login.component.ts
import { Component, OnInit } from '@angular/core';
import { FormGroup, FormBuilder, Validators } from '@angular/forms';

export class LoginComponent implements OnInit {
  loginForm!: FormGroup;

  constructor(private fb: FormBuilder) { }

  ngOnInit(): void {
    this.loginForm = this.fb.group({
      email: ['', [Validators.required, Validators.email]],
      password: ['', [Validators.required, Validators.minLength(8)]]
    });
  }

  get email() {
    return this.loginForm.get('email');
  }

  get password() {
    return this.loginForm.get('password');
  }

  onSubmit(): void {
    if (this.loginForm.valid) {
      console.log(this.loginForm.value);
    } else {
      this.loginForm.markAllAsTouched();
    }
  }
}
```

```html
<!-- login.component.html -->
<form [formGroup]="loginForm" (ngSubmit)="onSubmit()">

  <div>
    <label>البريد الإلكتروني:</label>
    <input type="email" formControlName="email">

    <div *ngIf="email?.invalid && email?.touched">
      <small *ngIf="email?.errors?.['required']">
        البريد الإلكتروني مطلوب
      </small>
      <small *ngIf="email?.errors?.['email']">
        البريد الإلكتروني غير صحيح
      </small>
    </div>
  </div>

  <div>
    <label>كلمة المرور:</label>
    <input type="password" formControlName="password">

    <div *ngIf="password?.invalid && password?.touched">
      <small *ngIf="password?.errors?.['required']">
        كلمة المرور مطلوبة
      </small>
      <small *ngIf="password?.errors?.['minlength']">
        كلمة المرور يجب أن تكون 8 أحرف على الأقل
      </small>
    </div>
  </div>

  <button type="submit" [disabled]="loginForm.invalid">
    تسجيل الدخول
  </button>

</form>
```

---

### السؤال 12: كيف تنشئ Custom Validator؟

**الإجابة:**

**Custom Validator:**

```typescript
// validators/custom-validators.ts
import { AbstractControl, ValidationErrors, ValidatorFn } from '@angular/forms';

export class CustomValidators {

  // 1. Validator بسيط: رقم هوية سعودي
  static saudiNationalId(): ValidatorFn {
    return (control: AbstractControl): ValidationErrors | null => {
      const value = control.value;

      if (!value) {
        return null; // اترك required validator يتعامل مع الفراغ
      }

      // التحقق من الطول
      if (value.length !== 10) {
        return { saudiNationalId: { message: 'رقم الهوية يجب أن يكون 10 أرقام' } };
      }

      // يجب أن يبدأ بـ 1 أو 2
      if (!value.match(/^[12]\d{9}$/)) {
        return { saudiNationalId: { message: 'رقم الهوية يجب أن يبدأ بـ 1 أو 2' } };
      }

      return null; // صحيح
    };
  }

  // 2. Validator مع parameter: قوة كلمة المرور
  static passwordStrength(minStrength: number): ValidatorFn {
    return (control: AbstractControl): ValidationErrors | null => {
      const value = control.value;

      if (!value) {
        return null;
      }

      let strength = 0;

      // حروف صغيرة
      if (/[a-z]/.test(value)) strength++;

      // حروف كبيرة
      if (/[A-Z]/.test(value)) strength++;

      // أرقام
      if (/\d/.test(value)) strength++;

      // رموز خاصة
      if (/[@$!%*?&]/.test(value)) strength++;

      if (strength < minStrength) {
        return {
          passwordStrength: {
            message: `كلمة المرور ضعيفة (القوة: ${strength}/${minStrength})`
          }
        };
      }

      return null;
    };
  }

  // 3. Cross-field Validator: تطابق كلمة المرور
  static passwordMatch(passwordKey: string, confirmPasswordKey: string): ValidatorFn {
    return (control: AbstractControl): ValidationErrors | null => {
      const password = control.get(passwordKey);
      const confirmPassword = control.get(confirmPasswordKey);

      if (!password || !confirmPassword) {
        return null;
      }

      if (password.value !== confirmPassword.value) {
        confirmPassword.setErrors({ passwordMatch: { message: 'كلمة المرور غير متطابقة' } });
        return { passwordMatch: true };
      } else {
        // إزالة الخطأ إذا كان موجود
        if (confirmPassword.hasError('passwordMatch')) {
          delete confirmPassword.errors?.['passwordMatch'];
          confirmPassword.updateValueAndValidity({ onlySelf: true });
        }
      }

      return null;
    };
  }

  // 4. Async Validator: التحقق من تفرد البريد الإلكتروني
  static emailUnique(userService: UserService): AsyncValidatorFn {
    return (control: AbstractControl): Observable<ValidationErrors | null> => {
      const email = control.value;

      if (!email) {
        return of(null);
      }

      return userService.checkEmailExists(email).pipe(
        map(exists => {
          return exists ? { emailUnique: { message: 'البريد الإلكتروني مستخدم مسبقاً' } } : null;
        }),
        catchError(() => of(null))
      );
    };
  }
}
```

**الاستخدام:**

```typescript
import { CustomValidators } from './validators/custom-validators';

export class RegistrationComponent implements OnInit {
  registrationForm!: FormGroup;

  constructor(
    private fb: FormBuilder,
    private userService: UserService
  ) { }

  ngOnInit(): void {
    this.registrationForm = this.fb.group({
      nationalId: ['', [
        Validators.required,
        CustomValidators.saudiNationalId()
      ]],

      email: ['', [
        Validators.required,
        Validators.email
      ], [
        CustomValidators.emailUnique(this.userService) // Async
      ]],

      password: ['', [
        Validators.required,
        Validators.minLength(8),
        CustomValidators.passwordStrength(3)
      ]],

      confirmPassword: ['', [Validators.required]]

    }, {
      validators: CustomValidators.passwordMatch('password', 'confirmPassword')
    });
  }

  get nationalId() {
    return this.registrationForm.get('nationalId');
  }

  get email() {
    return this.registrationForm.get('email');
  }

  get password() {
    return this.registrationForm.get('password');
  }

  get confirmPassword() {
    return this.registrationForm.get('confirmPassword');
  }
}
```

```html
<!-- registration.component.html -->
<form [formGroup]="registrationForm" (ngSubmit)="onSubmit()">

  <!-- رقم الهوية -->
  <div>
    <label>رقم الهوية:</label>
    <input type="text" formControlName="nationalId">

    <div *ngIf="nationalId?.invalid && nationalId?.touched">
      <small *ngIf="nationalId?.errors?.['required']">
        رقم الهوية مطلوب
      </small>
      <small *ngIf="nationalId?.errors?.['saudiNationalId']">
        {{ nationalId.errors?.['saudiNationalId'].message }}
      </small>
    </div>
  </div>

  <!-- البريد الإلكتروني -->
  <div>
    <label>البريد الإلكتروني:</label>
    <input type="email" formControlName="email">

    <span *ngIf="email?.pending">جاري التحقق...</span>

    <div *ngIf="email?.invalid && email?.touched">
      <small *ngIf="email?.errors?.['required']">
        البريد الإلكتروني مطلوب
      </small>
      <small *ngIf="email?.errors?.['email']">
        البريد الإلكتروني غير صحيح
      </small>
      <small *ngIf="email?.errors?.['emailUnique']">
        {{ email.errors?.['emailUnique'].message }}
      </small>
    </div>
  </div>

  <!-- كلمة المرور -->
  <div>
    <label>كلمة المرور:</label>
    <input type="password" formControlName="password">

    <div *ngIf="password?.invalid && password?.touched">
      <small *ngIf="password?.errors?.['required']">
        كلمة المرور مطلوبة
      </small>
      <small *ngIf="password?.errors?.['minlength']">
        كلمة المرور يجب أن تكون 8 أحرف على الأقل
      </small>
      <small *ngIf="password?.errors?.['passwordStrength']">
        {{ password.errors?.['passwordStrength'].message }}
      </small>
    </div>
  </div>

  <!-- تأكيد كلمة المرور -->
  <div>
    <label>تأكيد كلمة المرور:</label>
    <input type="password" formControlName="confirmPassword">

    <div *ngIf="confirmPassword?.invalid && confirmPassword?.touched">
      <small *ngIf="confirmPassword?.errors?.['required']">
        تأكيد كلمة المرور مطلوب
      </small>
      <small *ngIf="confirmPassword?.errors?.['passwordMatch']">
        {{ confirmPassword.errors?.['passwordMatch'].message }}
      </small>
    </div>
  </div>

  <button type="submit" [disabled]="registrationForm.invalid">
    تسجيل
  </button>

</form>
```

---

### السؤال 13: ما هو Routing في Angular؟

**الإجابة:**

**Routing** يسمح بالتنقل بين الصفحات (Components) بدون إعادة تحميل.

**الإعداد:**

```typescript
// app-routing.module.ts
import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';

import { HomeComponent } from './home/home.component';
import { AboutComponent } from './about/about.component';
import { UsersComponent } from './users/users.component';
import { UserDetailComponent } from './users/user-detail/user-detail.component';
import { NotFoundComponent } from './not-found/not-found.component';
import { LoginComponent } from './login/login.component';
import { AuthGuard } from './guards/auth.guard';

const routes: Routes = [
  { path: '', component: HomeComponent },
  { path: 'about', component: AboutComponent },
  { path: 'login', component: LoginComponent },

  // Route مع parameter
  { path: 'users', component: UsersComponent },
  { path: 'users/:id', component: UserDetailComponent },

  // Route مع Guard
  {
    path: 'dashboard',
    loadChildren: () => import('./dashboard/dashboard.module')
      .then(m => m.DashboardModule),
    canActivate: [AuthGuard]
  },

  // Redirect
  { path: 'home', redirectTo: '', pathMatch: 'full' },

  // Wildcard (404)
  { path: '**', component: NotFoundComponent }
];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule]
})
export class AppRoutingModule { }
```

```html
<!-- app.component.html -->
<nav>
  <a routerLink="/" routerLinkActive="active" [routerLinkActiveOptions]="{exact: true}">
    الرئيسية
  </a>
  <a routerLink="/about" routerLinkActive="active">حول</a>
  <a routerLink="/users" routerLinkActive="active">المستخدمون</a>
</nav>

<router-outlet></router-outlet>
```

**Navigation Programmatically:**

```typescript
export class UserListComponent {
  constructor(private router: Router) { }

  goToUser(id: number): void {
    // طريقة 1
    this.router.navigate(['/users', id]);

    // طريقة 2
    this.router.navigateByUrl(`/users/${id}`);

    // مع query parameters
    this.router.navigate(['/users'], {
      queryParams: { page: 1, sort: 'name' }
    });
    // النتيجة: /users?page=1&sort=name
  }
}
```

**قراءة Route Parameters:**

```typescript
export class UserDetailComponent implements OnInit {
  userId!: number;
  queryPage!: number;

  constructor(
    private route: ActivatedRoute,
    private router: Router
  ) { }

  ngOnInit(): void {
    // Route parameter
    this.route.params.subscribe(params => {
      this.userId = +params['id'];
      this.loadUser(this.userId);
    });

    // Query parameter
    this.route.queryParams.subscribe(params => {
      this.queryPage = params['page'] || 1;
    });

    // Snapshot (للقراءة مرة واحدة)
    this.userId = +this.route.snapshot.params['id'];
  }
}
```

---

### السؤال 14: ما هو Route Guard؟

**الإجابة:**

**Route Guards** تتحكم في الوصول للـ Routes.

**الأنواع:**

```
Route Guards
│
├── 1. CanActivate
│   └── قبل الدخول للـ route
│
├── 2. CanActivateChild
│   └── قبل الدخول لـ child routes
│
├── 3. CanDeactivate
│   └── قبل الخروج من route
│
├── 4. Resolve
│   └── جلب بيانات قبل الدخول
│
└── 5. CanLoad
    └── قبل lazy loading
```

**مثال: Auth Guard**

```typescript
// guards/auth.guard.ts
import { Injectable } from '@angular/core';
import { CanActivate, ActivatedRouteSnapshot, RouterStateSnapshot,
         Router, UrlTree } from '@angular/router';
import { Observable } from 'rxjs';
import { AuthService } from '../services/auth.service';

@Injectable({
  providedIn: 'root'
})
export class AuthGuard implements CanActivate {

  constructor(
    private authService: AuthService,
    private router: Router
  ) { }

  canActivate(
    route: ActivatedRouteSnapshot,
    state: RouterStateSnapshot
  ): Observable<boolean | UrlTree> | Promise<boolean | UrlTree> | boolean | UrlTree {

    if (this.authService.isLoggedIn()) {
      return true; // مسموح
    }

    // إعادة توجيه لصفحة تسجيل الدخول
    return this.router.createUrlTree(['/login'], {
      queryParams: { returnUrl: state.url }
    });
  }
}

// الاستخدام
const routes: Routes = [
  {
    path: 'dashboard',
    component: DashboardComponent,
    canActivate: [AuthGuard]
  }
];
```

**مثال: CanDeactivate (تأكيد الخروج)**

```typescript
// guards/unsaved-changes.guard.ts
import { Injectable } from '@angular/core';
import { CanDeactivate } from '@angular/router';
import { Observable } from 'rxjs';

export interface CanComponentDeactivate {
  canDeactivate: () => Observable<boolean> | Promise<boolean> | boolean;
}

@Injectable({
  providedIn: 'root'
})
export class UnsavedChangesGuard implements CanDeactivate<CanComponentDeactivate> {

  canDeactivate(
    component: CanComponentDeactivate
  ): Observable<boolean> | Promise<boolean> | boolean {
    return component.canDeactivate ? component.canDeactivate() : true;
  }
}

// Component
export class EditUserComponent implements CanComponentDeactivate {
  hasUnsavedChanges = false;

  canDeactivate(): boolean {
    if (this.hasUnsavedChanges) {
      return confirm('لديك تغييرات غير محفوظة. هل تريد المغادرة؟');
    }
    return true;
  }
}

// Route
{
  path: 'edit/:id',
  component: EditUserComponent,
  canDeactivate: [UnsavedChangesGuard]
}
```

---

## أسئلة المستوى المتقدم

### السؤال 15: اشرح Change Detection في Angular

**الإجابة:**

**Change Detection** هي الآلية التي تكتشف التغييرات وتحدث الـ View.

**الاستراتيجيات:**

**1. Default Strategy:**
```typescript
@Component({
  selector: 'app-user',
  changeDetection: ChangeDetectionStrategy.Default // الافتراضي
})
export class UserComponent {
  // Angular يفحص هذا Component وكل أبنائه عند أي حدث
}
```

**2. OnPush Strategy (أداء أفضل):**
```typescript
@Component({
  selector: 'app-user',
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class UserComponent {
  @Input() user!: User;

  // Angular يفحص فقط عند:
  // 1. تغيير @Input reference
  // 2. Event من هذا Component
  // 3. Observable emit + async pipe
  // 4. detectChanges() يدوياً
}
```

**متى يحدث Change Detection:**
- Events (click, input, etc.)
- HTTP Requests
- Timers (setTimeout, setInterval)

**تحسين الأداء:**

```typescript
import { ChangeDetectionStrategy, ChangeDetectorRef } from '@angular/core';

@Component({
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class UserComponent {
  users: User[] = [];

  constructor(private cdr: ChangeDetectorRef) { }

  addUser(user: User): void {
    // ❌ لن يعمل مع OnPush (نفس الـ reference)
    this.users.push(user);

    // ✅ يعمل (reference جديد)
    this.users = [...this.users, user];

    // أو استخدم detectChanges() يدوياً
    // this.cdr.detectChanges();
  }
}
```

---

### السؤال 16: ما الفرق بين Observables و Promises؟

**الإجابة:**

| المعيار | Promise | Observable |
|--------|---------|-----------|
| **القيم** | قيمة واحدة | عدة قيم (stream) |
| **Lazy** | لا (Eager) | نعم |
| **Cancellation** | لا | نعم (unsubscribe) |
| **Operators** | then, catch | map, filter, etc |
| **المكتبة** | JavaScript | RxJS |

**Promise:**
```typescript
// Promise - قيمة واحدة
const promise = new Promise((resolve, reject) => {
  setTimeout(() => resolve('Done'), 1000);
});

promise.then(result => console.log(result));
// لا يمكن إلغاؤه
```

**Observable:**
```typescript
// Observable - عدة قيم
const observable = new Observable(observer => {
  observer.next(1);
  observer.next(2);
  setTimeout(() => observer.next(3), 1000);
});

const subscription = observable.subscribe(value => console.log(value));

// يمكن إلغاؤه
subscription.unsubscribe();
```

**في Angular:**

```typescript
// HTTP مع Promise
async loadUsers(): Promise<void> {
  try {
    this.users = await this.http.get<User[]>('/api/users').toPromise();
  } catch (error) {
    console.error(error);
  }
}

// HTTP مع Observable (الأفضل)
loadUsers(): void {
  this.http.get<User[]>('/api/users')
    .pipe(
      catchError(error => {
        console.error(error);
        return of([]);
      })
    )
    .subscribe(users => this.users = users);
}
```

---

### السؤال 17: اشرح أهم RxJS Operators

**الإجابة:**

**Operators الأكثر استخداماً:**

```typescript
import { of, from, interval } from 'rxjs';
import { map, filter, tap, catchError, switchMap, debounceTime,
         distinctUntilChanged, take, takeUntil } from 'rxjs/operators';

// ========================================
// 1. map - تحويل القيم
// ========================================
of(1, 2, 3).pipe(
  map(x => x * 2)
).subscribe(console.log);
// Output: 2, 4, 6

// ========================================
// 2. filter - تصفية القيم
// ========================================
of(1, 2, 3, 4, 5).pipe(
  filter(x => x % 2 === 0)
).subscribe(console.log);
// Output: 2, 4

// ========================================
// 3. tap - تنفيذ side effects (logging)
// ========================================
of(1, 2, 3).pipe(
  tap(x => console.log('Before:', x)),
  map(x => x * 2),
  tap(x => console.log('After:', x))
).subscribe();

// ========================================
// 4. catchError - معالجة الأخطاء
// ========================================
this.http.get('/api/users').pipe(
  catchError(error => {
    console.error('Error:', error);
    return of([]); // قيمة افتراضية
  })
).subscribe(users => this.users = users);

// ========================================
// 5. switchMap - تحويل لـ Observable آخر
// ========================================
searchControl.valueChanges.pipe(
  debounceTime(300),
  distinctUntilChanged(),
  switchMap(searchTerm => this.userService.search(searchTerm))
).subscribe(results => this.searchResults = results);

// ========================================
// 6. debounceTime - انتظار بين الأحداث
// ========================================
searchInput.valueChanges.pipe(
  debounceTime(500) // انتظر 500ms بعد آخر كتابة
).subscribe(value => this.search(value));

// ========================================
// 7. distinctUntilChanged - تجنب التكرار
// ========================================
searchInput.valueChanges.pipe(
  distinctUntilChanged() // فقط عند التغيير
).subscribe(value => console.log(value));

// ========================================
// 8. take - أخذ عدد محدد
// ========================================
interval(1000).pipe(
  take(5) // أول 5 قيم فقط
).subscribe(console.log);
// Output: 0, 1, 2, 3, 4

// ========================================
// 9. takeUntil - حتى Observable آخر
// ========================================
private destroy$ = new Subject<void>();

ngOnInit(): void {
  this.userService.getUsers().pipe(
    takeUntil(this.destroy$)
  ).subscribe(users => this.users = users);
}

ngOnDestroy(): void {
  this.destroy$.next();
  this.destroy$.complete();
}
```

**مثال عملي: Search مع Auto-complete**

```typescript
export class SearchComponent implements OnInit {
  searchControl = new FormControl();
  searchResults: User[] = [];
  loading = false;

  constructor(private userService: UserService) { }

  ngOnInit(): void {
    this.searchControl.valueChanges.pipe(
      debounceTime(300),           // انتظر 300ms بعد آخر كتابة
      distinctUntilChanged(),      // فقط عند التغيير
      tap(() => this.loading = true),  // أظهر loading
      switchMap(searchTerm =>
        searchTerm.length >= 2
          ? this.userService.search(searchTerm).pipe(
              catchError(() => of([]))
            )
          : of([])
      ),
      tap(() => this.loading = false)  // أخفِ loading
    ).subscribe(results => {
      this.searchResults = results;
    });
  }
}
```

---

### السؤال 18: كيف تدير Memory Leaks في Angular؟

**الإجابة:**

**أسباب Memory Leaks:**
- ❌ عدم عمل unsubscribe للـ Subscriptions
- ❌ Event Listeners غير محذوفة
- ❌ Timers/Intervals غير موقوفة

**الحلول:**

**1. Unsubscribe يدوياً:**
```typescript
export class UserComponent implements OnInit, OnDestroy {
  private subscription!: Subscription;

  ngOnInit(): void {
    this.subscription = this.userService.getUsers()
      .subscribe(users => this.users = users);
  }

  ngOnDestroy(): void {
    this.subscription.unsubscribe(); // ✅
  }
}
```

**2. Subscriptions متعددة:**
```typescript
export class UserComponent implements OnInit, OnDestroy {
  private subscriptions = new Subscription();

  ngOnInit(): void {
    const sub1 = this.service1.getData().subscribe(/*...*/);
    const sub2 = this.service2.getData().subscribe(/*...*/);
    const sub3 = this.service3.getData().subscribe(/*...*/);

    this.subscriptions.add(sub1);
    this.subscriptions.add(sub2);
    this.subscriptions.add(sub3);
  }

  ngOnDestroy(): void {
    this.subscriptions.unsubscribe(); // ✅ كلهم دفعة واحدة
  }
}
```

**3. takeUntil (الأفضل):**
```typescript
export class UserComponent implements OnInit, OnDestroy {
  private destroy$ = new Subject<void>();

  ngOnInit(): void {
    this.service1.getData().pipe(
      takeUntil(this.destroy$)
    ).subscribe(/*...*/);

    this.service2.getData().pipe(
      takeUntil(this.destroy$)
    ).subscribe(/*...*/);

    this.service3.getData().pipe(
      takeUntil(this.destroy$)
    ).subscribe(/*...*/);
  }

  ngOnDestroy(): void {
    this.destroy$.next();
    this.destroy$.complete();
  }
}
```

**4. async pipe (لا يحتاج unsubscribe):**
```typescript
export class UserComponent {
  users$ = this.userService.getUsers(); // Observable

  constructor(private userService: UserService) { }
}
```

```html
<div *ngFor="let user of users$ | async">
  {{ user.name }}
</div>
```

**5. take(1) للـ One-time Subscriptions:**
```typescript
this.userService.getUser(id).pipe(
  take(1) // أول قيمة فقط ثم unsubscribe تلقائياً
).subscribe(user => this.user = user);
```

---

(يتبع في الرد التالي بسبب طول المحتوى...)

---

## أسئلة RxJS

### السؤال 19: ما هو Subject وأنواعه؟

**الإجابة:**

**Subject** هو نوع خاص من Observable يسمح بـ multicasting (بث لعدة مشتركين).

**الأنواع:**

```
Subject Types
│
├── 1. Subject (عادي)
│   └── لا قيمة افتراضية، يبث للمشتركين الحاليين فقط
│
├── 2. BehaviorSubject
│   └── له قيمة افتراضية، يبث آخر قيمة للمشتركين الجدد
│
├── 3. ReplaySubject
│   └── يحفظ عدد معين من القيم ويبثها للمشتركين الجدد
│
└── 4. AsyncSubject
    └── يبث آخر قيمة فقط عند complete()
```

**أمثلة:**

```typescript
// ========================================
// 1. Subject
// ========================================
const subject = new Subject<number>();

subject.subscribe(value => console.log('A:', value));
subject.next(1);
subject.next(2);

subject.subscribe(value => console.log('B:', value));
subject.next(3);

// Output:
// A: 1
// A: 2
// A: 3
// B: 3  ← المشترك B لم يرى 1 و 2

// ========================================
// 2. BehaviorSubject (الأكثر استخداماً)
// ========================================
const behaviorSubject = new BehaviorSubject<number>(0); // قيمة افتراضية

behaviorSubject.subscribe(value => console.log('A:', value));
// A: 0  ← القيمة الافتراضية

behaviorSubject.next(1);
behaviorSubject.next(2);

behaviorSubject.subscribe(value => console.log('B:', value));
// B: 2  ← آخر قيمة

behaviorSubject.next(3);
// A: 3
// B: 3

// الحصول على القيمة الحالية
console.log(behaviorSubject.value); // 3

// ========================================
// 3. ReplaySubject
// ========================================
const replaySubject = new ReplaySubject<number>(2); // آخر 2 قيم

replaySubject.next(1);
replaySubject.next(2);
replaySubject.next(3);

replaySubject.subscribe(value => console.log('A:', value));
// A: 2  ← آخر 2 قيم
// A: 3

// ========================================
// 4. AsyncSubject
// ========================================
const asyncSubject = new AsyncSubject<number>();

asyncSubject.subscribe(value => console.log('A:', value));
asyncSubject.next(1);
asyncSubject.next(2);
asyncSubject.next(3);
asyncSubject.complete();
// A: 3  ← آخر قيمة قبل complete
```

**استخدام عملي: مشاركة البيانات بين Components**

```typescript
// shared-data.service.ts
import { Injectable } from '@angular/core';
import { BehaviorSubject, Observable } from 'rxjs';

@Injectable({
  providedIn: 'root'
})
export class SharedDataService {
  // BehaviorSubject خاص
  private userSubject = new BehaviorSubject<User | null>(null);

  // Observable عام (read-only)
  public user$: Observable<User | null> = this.userSubject.asObservable();

  // تحديث القيمة
  setUser(user: User | null): void {
    this.userSubject.next(user);
  }

  // الحصول على القيمة الحالية
  getCurrentUser(): User | null {
    return this.userSubject.value;
  }
}

// component-a.component.ts
export class ComponentA {
  constructor(private sharedData: SharedDataService) { }

  login(user: User): void {
    this.sharedData.setUser(user);
  }
}

// component-b.component.ts
export class ComponentB implements OnInit {
  user: User | null = null;

  constructor(private sharedData: SharedDataService) { }

  ngOnInit(): void {
    this.sharedData.user$.subscribe(user => {
      this.user = user;
    });
  }
}
```

---

### السؤال 20: اشرح Higher-order Mapping Operators

**الإجابة:**

**Higher-order Observables:** Observable يُصدر Observables أخرى.

**الـ Operators:**

| Operator | السلوك | الاستخدام |
|----------|--------|-----------|
| **mergeMap** | يشترك في كل Observable | طلبات مستقلة |
| **switchMap** | يلغي السابق ويشترك في الجديد | Search, Navigation |
| **concatMap** | ينتظر الانتهاء ثم التالي | ترتيب مهم |
| **exhaustMap** | يتجاهل الجديد حتى ينتهي الحالي | منع Double-click |

**أمثلة:**

```typescript
import { fromEvent, interval } from 'rxjs';
import { mergeMap, switchMap, concatMap, exhaustMap, take } from 'rxjs/operators';

// ========================================
// 1. mergeMap - يشترك في الجميع
// ========================================
fromEvent(button, 'click').pipe(
  mergeMap(() => interval(1000).pipe(take(3)))
).subscribe(console.log);

// Click 1: 0, 1, 2
// Click 2 (أثناء الأول): 0, 1, 2
// النتيجة: 0, 0, 1, 1, 2, 2  ← متداخلة

// Use Case: تحميل عدة users بالتوازي
getUsers(ids: number[]): Observable<User[]> {
  return from(ids).pipe(
    mergeMap(id => this.http.get<User>(`/api/users/${id}`)),
    toArray()
  );
}

// ========================================
// 2. switchMap - يلغي السابق (الأكثر استخداماً)
// ========================================
searchControl.valueChanges.pipe(
  debounceTime(300),
  switchMap(searchTerm => this.http.get(`/api/search?q=${searchTerm}`))
).subscribe(results => this.results = results);

// إذا كتب المستخدم "ang" ثم "angular":
// - يلغي طلب "ang"
// - ينفذ طلب "angular" فقط

// Use Case: Navigation
this.route.params.pipe(
  switchMap(params => this.userService.getUserById(params['id']))
).subscribe(user => this.user = user);

// ========================================
// 3. concatMap - بالترتيب (واحد تلو الآخر)
// ========================================
from([1, 2, 3]).pipe(
  concatMap(num => this.http.post('/api/save', { num }))
).subscribe();

// ينتظر POST 1 ينتهي
// ثم POST 2
// ثم POST 3

// Use Case: عمليات يجب أن تحدث بالترتيب
saveAll(users: User[]): Observable<any> {
  return from(users).pipe(
    concatMap(user => this.http.post('/api/users', user))
  );
}

// ========================================
// 4. exhaustMap - يتجاهل أثناء التنفيذ
// ========================================
fromEvent(saveButton, 'click').pipe(
  exhaustMap(() => this.http.post('/api/save', this.data))
).subscribe();

// Click 1: يبدأ الحفظ
// Click 2, 3, 4 (أثناء الحفظ): تُتجاهل
// عند انتهاء الحفظ، Click 5: يُنفذ

// Use Case: منع double-submit
onSave(): void {
  this.saveButton$.pipe(
    exhaustMap(() => this.userService.save(this.user))
  ).subscribe(result => console.log('Saved'));
}
```

**مقارنة عملية:**

```typescript
// السيناريو: 3 نقرات متتالية، كل نقرة تطلب HTTP request يستغرق 5 ثواني

// mergeMap: 3 طلبات بالتوازي (3 نتائج)
// switchMap: طلب واحد فقط (آخر نقرة) (1 نتيجة)
// concatMap: 3 طلبات بالترتيب (3 نتائج، لكن متأخرة)
// exhaustMap: طلب واحد (أول نقرة) (1 نتيجة)
```

---

### السؤال 21: كيف تتعامل مع Error Handling في RxJS؟

**الإجابة:**

**استراتيجيات معالجة الأخطاء:**

```typescript
import { of, throwError, timer } from 'rxjs';
import { catchError, retry, retryWhen, tap, delayWhen } from 'rxjs/operators';

// ========================================
// 1. catchError - معالجة الخطأ
// ========================================
this.http.get('/api/users').pipe(
  catchError(error => {
    console.error('Error:', error);

    // Option 1: قيمة افتراضية
    return of([]);

    // Option 2: رمي خطأ جديد
    // return throwError(() => new Error('Custom error'));
  })
).subscribe(users => this.users = users);

// ========================================
// 2. retry - إعادة المحاولة
// ========================================
this.http.get('/api/users').pipe(
  retry(3), // 3 محاولات
  catchError(error => {
    console.error('Failed after 3 retries');
    return of([]);
  })
).subscribe();

// ========================================
// 3. retryWhen - إعادة محاولة مخصصة
// ========================================
this.http.get('/api/users').pipe(
  retryWhen(errors =>
    errors.pipe(
      tap(error => console.log('Error occurred, retrying...')),
      delayWhen((error, index) => {
        // Exponential backoff: 1s, 2s, 4s, 8s
        const delay = Math.min(1000 * Math.pow(2, index), 10000);
        console.log(`Retry ${index + 1} after ${delay}ms`);
        return timer(delay);
      }),
      take(5) // حد أقصى 5 محاولات
    )
  ),
  catchError(error => {
    console.error('Failed after retries');
    return of([]);
  })
).subscribe();

// ========================================
// 4. Error Handling Service
// ========================================
@Injectable({
  providedIn: 'root'
})
export class ErrorHandlerService {

  handleError<T>(operation = 'operation', result?: T) {
    return (error: any): Observable<T> => {
      console.error(`${operation} failed:`, error);

      // Log to server
      this.logError(error);

      // Show user notification
      this.notificationService.showError(
        `عملية ${operation} فشلت. يرجى المحاولة لاحقاً.`
      );

      // Return safe result
      return of(result as T);
    };
  }

  private logError(error: any): void {
    // إرسال للسيرفر
    this.http.post('/api/logs/error', {
      message: error.message,
      stack: error.stack,
      timestamp: new Date()
    }).subscribe();
  }
}

// الاستخدام:
export class UserService {
  constructor(
    private http: HttpClient,
    private errorHandler: ErrorHandlerService
  ) { }

  getUsers(): Observable<User[]> {
    return this.http.get<User[]>('/api/users').pipe(
      retry(2),
      catchError(this.errorHandler.handleError<User[]>('getUsers', []))
    );
  }
}

// ========================================
// 5. Global Error Handler
// ========================================
@Injectable()
export class GlobalErrorHandler implements ErrorHandler {

  constructor(private injector: Injector) { }

  handleError(error: Error): void {
    const notificationService = this.injector.get(NotificationService);

    console.error('Global Error:', error);

    // عرض رسالة للمستخدم
    notificationService.showError('حدث خطأ غير متوقع');

    // إرسال للسيرفر
    // ...
  }
}

// app.module.ts
@NgModule({
  providers: [
    { provide: ErrorHandler, useClass: GlobalErrorHandler }
  ]
})
```

---

## أسئلة State Management

### السؤال 22: ما هو State Management ولماذا نحتاجه؟

**الإجابة:**

**State Management** هو إدارة حالة التطبيق (البيانات المشتركة).

**المشكلة بدون State Management:**

```
ComponentA ──┐
              ├──> Service ──> Backend
ComponentB ──┤
              │
ComponentC ──┘

المشاكل:
❌ تكرار الطلبات
❌ بيانات غير متزامنة
❌ صعوبة التتبع
❌ Props drilling
```

**الحلول:**

```
State Management Solutions
│
├── 1. Services + BehaviorSubject (بسيط)
│   └── للتطبيقات الصغيرة
│
├── 2. NgRx (معقد، قوي)
│   └── Redux pattern
│
├── 3. Akita
│   └── أبسط من NgRx
│
└── 4. NGXS
    └── بديل لـ NgRx
```

**مثال: Services + BehaviorSubject**

```typescript
// state/cart.state.ts
export interface CartState {
  items: CartItem[];
  total: number;
  loading: boolean;
}

// services/cart.service.ts
@Injectable({
  providedIn: 'root'
})
export class CartService {
  private initialState: CartState = {
    items: [],
    total: 0,
    loading: false
  };

  private stateSubject = new BehaviorSubject<CartState>(this.initialState);
  public state$ = this.stateSubject.asObservable();

  // Selectors
  get items$(): Observable<CartItem[]> {
    return this.state$.pipe(map(state => state.items));
  }

  get total$(): Observable<number> {
    return this.state$.pipe(map(state => state.total));
  }

  get itemCount$(): Observable<number> {
    return this.state$.pipe(
      map(state => state.items.reduce((sum, item) => sum + item.quantity, 0))
    );
  }

  // Actions
  addItem(product: Product): void {
    const currentState = this.stateSubject.value;
    const existingItem = currentState.items.find(item => item.productId === product.id);

    let newItems: CartItem[];

    if (existingItem) {
      newItems = currentState.items.map(item =>
        item.productId === product.id
          ? { ...item, quantity: item.quantity + 1 }
          : item
      );
    } else {
      newItems = [...currentState.items, {
        productId: product.id,
        name: product.name,
        price: product.price,
        quantity: 1
      }];
    }

    this.updateState({
      items: newItems,
      total: this.calculateTotal(newItems)
    });
  }

  removeItem(productId: number): void {
    const currentState = this.stateSubject.value;
    const newItems = currentState.items.filter(item => item.productId !== productId);

    this.updateState({
      items: newItems,
      total: this.calculateTotal(newItems)
    });
  }

  updateQuantity(productId: number, quantity: number): void {
    const currentState = this.stateSubject.value;
    const newItems = currentState.items.map(item =>
      item.productId === productId
        ? { ...item, quantity }
        : item
    );

    this.updateState({
      items: newItems,
      total: this.calculateTotal(newItems)
    });
  }

  clearCart(): void {
    this.updateState(this.initialState);
  }

  private updateState(partial: Partial<CartState>): void {
    const currentState = this.stateSubject.value;
    this.stateSubject.next({ ...currentState, ...partial });
  }

  private calculateTotal(items: CartItem[]): number {
    return items.reduce((sum, item) => sum + (item.price * item.quantity), 0);
  }
}

// الاستخدام في Component
export class CartComponent implements OnInit {
  items$ = this.cartService.items$;
  total$ = this.cartService.total$;
  itemCount$ = this.cartService.itemCount$;

  constructor(private cartService: CartService) { }

  removeItem(productId: number): void {
    this.cartService.removeItem(productId);
  }
}
```

---

### السؤال 23: ما هو NgRx وكيف يعمل؟

**الإجابة:**

**NgRx** هو State Management library مبني على Redux pattern.

**المكونات:**

```
NgRx Architecture
│
├── Store (مخزن واحد للحالة)
│   └── Single source of truth
│
├── Actions (الإجراءات)
│   └── Events تصف ما حدث
│
├── Reducers (المُختزلات)
│   └── Pure functions تُحدّث الحالة
│
├── Selectors (المُحددات)
│   └── Query الحالة
│
└── Effects (التأثيرات)
    └── Side effects (HTTP, etc)
```

**التدفق:**

```
Component
   ↓ dispatch(action)
Store
   ↓ action
Reducer
   ↓ new state
Store
   ↓ select(selector)
Component
```

**مثال كامل:**

```typescript
// ========================================
// 1. State & Actions
// ========================================

// state/user.state.ts
export interface UserState {
  users: User[];
  selectedUser: User | null;
  loading: boolean;
  error: string | null;
}

export const initialState: UserState = {
  users: [],
  selectedUser: null,
  loading: false,
  error: null
};

// actions/user.actions.ts
import { createAction, props } from '@ngrx/store';

// Load Users
export const loadUsers = createAction('[User List] Load Users');

export const loadUsersSuccess = createAction(
  '[User API] Load Users Success',
  props<{ users: User[] }>()
);

export const loadUsersFailure = createAction(
  '[User API] Load Users Failure',
  props<{ error: string }>()
);

// Select User
export const selectUser = createAction(
  '[User Detail] Select User',
  props<{ userId: number }>()
);

// ========================================
// 2. Reducer
// ========================================

// reducers/user.reducer.ts
import { createReducer, on } from '@ngrx/store';
import * as UserActions from '../actions/user.actions';

export const userReducer = createReducer(
  initialState,

  on(UserActions.loadUsers, state => ({
    ...state,
    loading: true,
    error: null
  })),

  on(UserActions.loadUsersSuccess, (state, { users }) => ({
    ...state,
    users,
    loading: false
  })),

  on(UserActions.loadUsersFailure, (state, { error }) => ({
    ...state,
    loading: false,
    error
  })),

  on(UserActions.selectUser, (state, { userId }) => ({
    ...state,
    selectedUser: state.users.find(u => u.id === userId) || null
  }))
);

// ========================================
// 3. Selectors
// ========================================

// selectors/user.selectors.ts
import { createFeatureSelector, createSelector } from '@ngrx/store';

export const selectUserState = createFeatureSelector<UserState>('users');

export const selectAllUsers = createSelector(
  selectUserState,
  state => state.users
);

export const selectSelectedUser = createSelector(
  selectUserState,
  state => state.selectedUser
);

export const selectUsersLoading = createSelector(
  selectUserState,
  state => state.loading
);

export const selectUsersError = createSelector(
  selectUserState,
  state => state.error
);

export const selectUserById = (userId: number) => createSelector(
  selectAllUsers,
  users => users.find(u => u.id === userId)
);

// ========================================
// 4. Effects
// ========================================

// effects/user.effects.ts
import { Injectable } from '@angular/core';
import { Actions, createEffect, ofType } from '@ngrx/effects';
import { of } from 'rxjs';
import { map, catchError, switchMap } from 'rxjs/operators';
import * as UserActions from '../actions/user.actions';

@Injectable()
export class UserEffects {

  loadUsers$ = createEffect(() =>
    this.actions$.pipe(
      ofType(UserActions.loadUsers),
      switchMap(() =>
        this.userService.getUsers().pipe(
          map(users => UserActions.loadUsersSuccess({ users })),
          catchError(error =>
            of(UserActions.loadUsersFailure({ error: error.message }))
          )
        )
      )
    )
  );

  constructor(
    private actions$: Actions,
    private userService: UserService
  ) { }
}

// ========================================
// 5. Module Setup
// ========================================

// app.module.ts
import { StoreModule } from '@ngrx/store';
import { EffectsModule } from '@ngrx/effects';
import { StoreDevtoolsModule } from '@ngrx/store-devtools';
import { userReducer } from './state/reducers/user.reducer';
import { UserEffects } from './state/effects/user.effects';

@NgModule({
  imports: [
    StoreModule.forRoot({ users: userReducer }),
    EffectsModule.forRoot([UserEffects]),
    StoreDevtoolsModule.instrument({
      maxAge: 25,
      logOnly: environment.production
    })
  ]
})
export class AppModule { }

// ========================================
// 6. Component Usage
// ========================================

export class UserListComponent implements OnInit {
  users$ = this.store.select(selectAllUsers);
  loading$ = this.store.select(selectUsersLoading);
  error$ = this.store.select(selectUsersError);

  constructor(private store: Store) { }

  ngOnInit(): void {
    this.store.dispatch(loadUsers());
  }

  selectUser(userId: number): void {
    this.store.dispatch(selectUser({ userId }));
  }
}
```

---

## أسئلة Performance

### السؤال 24: كيف تحسّن أداء تطبيق Angular؟

**الإجابة:**

**استراتيجيات تحسين الأداء:**

**1. OnPush Change Detection:**
```typescript
@Component({
  selector: 'app-user-card',
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class UserCardComponent {
  @Input() user!: User;
}
```

**2. TrackBy في ngFor:**
```typescript
// ❌ بدون trackBy (يعيد رسم كل شيء)
<div *ngFor="let user of users">
  {{ user.name }}
</div>

// ✅ مع trackBy (يعيد رسم المتغير فقط)
<div *ngFor="let user of users; trackBy: trackByUserId">
  {{ user.name }}
</div>

trackByUserId(index: number, user: User): number {
  return user.id;
}
```

**3. Lazy Loading:**
```typescript
// app-routing.module.ts
const routes: Routes = [
  {
    path: 'admin',
    loadChildren: () => import('./admin/admin.module')
      .then(m => m.AdminModule)
  },
  {
    path: 'dashboard',
    loadChildren: () => import('./dashboard/dashboard.module')
      .then(m => m.DashboardModule)
  }
];
```

**4. Preloading Strategy:**
```typescript
// app-routing.module.ts
import { PreloadAllModules } from '@angular/router';

@NgModule({
  imports: [
    RouterModule.forRoot(routes, {
      preloadingStrategy: PreloadAllModules
    })
  ]
})

// Custom Preloading
export class CustomPreloadingStrategy implements PreloadingStrategy {
  preload(route: Route, load: () => Observable<any>): Observable<any> {
    return route.data && route.data['preload'] ? load() : of(null);
  }
}

// الاستخدام
{
  path: 'dashboard',
  loadChildren: () => import('./dashboard/dashboard.module'),
  data: { preload: true }
}
```

**5. Pure Pipes:**
```typescript
// ❌ غير Pure (يُنفذ كل change detection)
@Pipe({
  name: 'search'
})
export class SearchPipe implements PipeTransform {
  transform(items: any[], searchTerm: string): any[] {
    return items.filter(item =>
      item.name.toLowerCase().includes(searchTerm.toLowerCase())
    );
  }
}

// ✅ Pure Pipe (يُنفذ فقط عند تغيير المدخلات)
@Pipe({
  name: 'search',
  pure: true // الافتراضي
})
```

**6. Unsubscribe:**
```typescript
// استخدم takeUntil أو async pipe
private destroy$ = new Subject<void>();

ngOnInit(): void {
  this.service.getData().pipe(
    takeUntil(this.destroy$)
  ).subscribe();
}

ngOnDestroy(): void {
  this.destroy$.next();
  this.destroy$.complete();
}
```

**7. Virtual Scrolling:**
```typescript
// للقوائم الطويلة
<cdk-virtual-scroll-viewport itemSize="50" style="height: 400px">
  <div *cdkVirtualFor="let user of users">
    {{ user.name }}
  </div>
</cdk-virtual-scroll-viewport>
```

**8. AOT Compilation:**
```bash
# Production build مع AOT
ng build --prod

# angular.json
{
  "configurations": {
    "production": {
      "aot": true,
      "optimization": true,
      "buildOptimizer": true
    }
  }
}
```

**9. Bundle Analysis:**
```bash
# تحليل حجم الـ Bundle
npm install -g webpack-bundle-analyzer
ng build --stats-json
webpack-bundle-analyzer dist/stats.json
```

**10. Web Workers:**
```typescript
// للعمليات الثقيلة
ng generate web-worker app

// app.worker.ts
addEventListener('message', ({ data }) => {
  const result = heavyComputation(data);
  postMessage(result);
});

// component.ts
const worker = new Worker(new URL('./app.worker', import.meta.url));
worker.postMessage(data);
worker.onmessage = ({ data }) => {
  this.result = data;
};
```

---

### السؤال 25: ما هو Tree Shaking؟

**الإجابة:**

**Tree Shaking** هو إزالة الكود غير المستخدم من الـ Bundle النهائي.

**كيف يعمل:**

```typescript
// utils.ts
export function usedFunction() { }
export function unusedFunction() { }  // لن تُضاف للـ Bundle

// component.ts
import { usedFunction } from './utils';

usedFunction(); // فقط هذا سيُضاف للـ Bundle
```

**نصائح لـ Tree Shaking أفضل:**

```typescript
// ✅ استخدم ES6 imports
import { Component } from '@angular/core';

// ❌ تجنب require
const Component = require('@angular/core').Component;

// ✅ استورد ما تحتاج فقط
import { map, filter } from 'rxjs/operators';

// ❌ استيراد كل شيء
import * as operators from 'rxjs/operators';

// ✅ Side-effect free imports
import { MyService } from './my-service';

// ❌ Side effects
import './global-styles.css'; // سيُضاف دائماً
```

---

## أسئلة Testing

### السؤال 26: كيف تختبر Component في Angular؟

**الإجابة:**

**أنواع الاختبارات:**

```
Testing في Angular
│
├── 1. Unit Testing (Jasmine + Karma)
│   └── اختبار Components, Services, Pipes
│
├── 2. Integration Testing
│   └── اختبار تفاعل Components
│
└── 3. E2E Testing (Protractor/Cypress)
    └── اختبار التطبيق كاملاً
```

**مثال: Unit Test لـ Component**

```typescript
// user.component.ts
export class UserComponent {
  @Input() user!: User;
  @Output() userDeleted = new EventEmitter<number>();

  deleteUser(): void {
    this.userDeleted.emit(this.user.id);
  }
}

// user.component.spec.ts
import { ComponentFixture, TestBed } from '@angular/core/testing';
import { UserComponent } from './user.component';

describe('UserComponent', () => {
  let component: UserComponent;
  let fixture: ComponentFixture<UserComponent>;

  beforeEach(async () => {
    await TestBed.configureTestingModule({
      declarations: [ UserComponent ]
    }).compileComponents();

    fixture = TestBed.createComponent(UserComponent);
    component = fixture.componentInstance;
  });

  it('should create', () => {
    expect(component).toBeTruthy();
  });

  it('should display user name', () => {
    const user: User = { id: 1, name: 'أحمد' };
    component.user = user;
    fixture.detectChanges();

    const compiled = fixture.nativeElement;
    expect(compiled.querySelector('h2').textContent).toContain('أحمد');
  });

  it('should emit userDeleted event', () => {
    const user: User = { id: 1, name: 'أحمد' };
    component.user = user;

    let emittedId: number | undefined;
    component.userDeleted.subscribe((id: number) => {
      emittedId = id;
    });

    component.deleteUser();

    expect(emittedId).toBe(1);
  });
});
```

**اختبار Service:**

```typescript
// user.service.spec.ts
import { TestBed } from '@angular/core/testing';
import { HttpClientTestingModule, HttpTestingController } from '@angular/common/http/testing';
import { UserService } from './user.service';

describe('UserService', () => {
  let service: UserService;
  let httpMock: HttpTestingController;

  beforeEach(() => {
    TestBed.configureTestingModule({
      imports: [HttpClientTestingModule],
      providers: [UserService]
    });

    service = TestBed.inject(UserService);
    httpMock = TestBed.inject(HttpTestingController);
  });

  afterEach(() => {
    httpMock.verify();
  });

  it('should fetch users', () => {
    const mockUsers: User[] = [
      { id: 1, name: 'أحمد' },
      { id: 2, name: 'سارة' }
    ];

    service.getUsers().subscribe(users => {
      expect(users.length).toBe(2);
      expect(users).toEqual(mockUsers);
    });

    const req = httpMock.expectOne('/api/users');
    expect(req.request.method).toBe('GET');
    req.flush(mockUsers);
  });

  it('should handle error', () => {
    service.getUsers().subscribe(
      () => fail('should have failed'),
      (error) => {
        expect(error.status).toBe(500);
      }
    );

    const req = httpMock.expectOne('/api/users');
    req.flush('Server error', { status: 500, statusText: 'Server Error' });
  });
});
```

---

## أسئلة عملية وكود

### السؤال 27: اكتب مكون Autocomplete

**الإجابة:**

```typescript
// autocomplete.component.ts
import { Component, OnInit, OnDestroy } from '@angular/core';
import { FormControl } from '@angular/forms';
import { Subject } from 'rxjs';
import { debounceTime, distinctUntilChanged, switchMap,
         takeUntil, filter } from 'rxjs/operators';

@Component({
  selector: 'app-autocomplete',
  template: `
    <div class="autocomplete">
      <input
        [formControl]="searchControl"
        placeholder="ابحث..."
        (focus)="onFocus()"
        (blur)="onBlur()">

      <div class="results" *ngIf="showResults && (results.length > 0 || loading)">
        <div *ngIf="loading" class="loading">جاري البحث...</div>

        <div
          *ngFor="let result of results"
          class="result-item"
          (click)="selectResult(result)">
          {{ result.name }}
        </div>

        <div *ngIf="!loading && results.length === 0" class="no-results">
          لا توجد نتائج
        </div>
      </div>
    </div>
  `,
  styles: [`
    .autocomplete {
      position: relative;
    }

    input {
      width: 100%;
      padding: 10px;
      border: 1px solid #ddd;
      border-radius: 4px;
    }

    .results {
      position: absolute;
      top: 100%;
      left: 0;
      right: 0;
      background: white;
      border: 1px solid #ddd;
      border-top: none;
      max-height: 300px;
      overflow-y: auto;
      z-index: 1000;
    }

    .result-item {
      padding: 10px;
      cursor: pointer;
    }

    .result-item:hover {
      background: #f5f5f5;
    }

    .loading, .no-results {
      padding: 10px;
      color: #999;
    }
  `]
})
export class AutocompleteComponent implements OnInit, OnDestroy {
  searchControl = new FormControl('');
  results: any[] = [];
  loading = false;
  showResults = false;

  private destroy$ = new Subject<void>();

  constructor(private searchService: SearchService) { }

  ngOnInit(): void {
    this.searchControl.valueChanges.pipe(
      debounceTime(300),
      distinctUntilChanged(),
      filter(term => term.length >= 2),
      switchMap(searchTerm => {
        this.loading = true;
        return this.searchService.search(searchTerm);
      }),
      takeUntil(this.destroy$)
    ).subscribe(results => {
      this.results = results;
      this.loading = false;
      this.showResults = true;
    });
  }

  onFocus(): void {
    if (this.results.length > 0) {
      this.showResults = true;
    }
  }

  onBlur(): void {
    // تأخير لإعطاء وقت للنقر على النتيجة
    setTimeout(() => this.showResults = false, 200);
  }

  selectResult(result: any): void {
    this.searchControl.setValue(result.name);
    this.showResults = false;
  }

  ngOnDestroy(): void {
    this.destroy$.next();
    this.destroy$.complete();
  }
}
```

---

### السؤال 28: اكتب Pagination Component

**الإجابة:**

```typescript
// pagination.component.ts
import { Component, Input, Output, EventEmitter, OnChanges } from '@angular/core';

@Component({
  selector: 'app-pagination',
  template: `
    <div class="pagination" *ngIf="totalPages > 1">
      <button
        class="page-btn"
        [disabled]="currentPage === 1"
        (click)="onPageChange(1)">
        الأولى
      </button>

      <button
        class="page-btn"
        [disabled]="currentPage === 1"
        (click)="onPageChange(currentPage - 1)">
        السابقة
      </button>

      <button
        *ngFor="let page of visiblePages"
        class="page-btn"
        [class.active]="page === currentPage"
        (click)="onPageChange(page)">
        {{ page }}
      </button>

      <button
        class="page-btn"
        [disabled]="currentPage === totalPages"
        (click)="onPageChange(currentPage + 1)">
        التالية
      </button>

      <button
        class="page-btn"
        [disabled]="currentPage === totalPages"
        (click)="onPageChange(totalPages)">
        الأخيرة
      </button>

      <div class="page-info">
        صفحة {{ currentPage }} من {{ totalPages }}
        (إجمالي: {{ totalItems }} عنصر)
      </div>
    </div>
  `,
  styles: [`
    .pagination {
      display: flex;
      align-items: center;
      gap: 5px;
      margin: 20px 0;
    }

    .page-btn {
      padding: 8px 12px;
      border: 1px solid #ddd;
      background: white;
      cursor: pointer;
      border-radius: 4px;
    }

    .page-btn:hover:not(:disabled) {
      background: #f5f5f5;
    }

    .page-btn.active {
      background: #007bff;
      color: white;
      border-color: #007bff;
    }

    .page-btn:disabled {
      opacity: 0.5;
      cursor: not-allowed;
    }

    .page-info {
      margin-right: auto;
      color: #666;
      font-size: 14px;
    }
  `]
})
export class PaginationComponent implements OnChanges {
  @Input() currentPage = 1;
  @Input() totalItems = 0;
  @Input() pageSize = 10;
  @Input() maxVisiblePages = 5;

  @Output() pageChange = new EventEmitter<number>();

  totalPages = 0;
  visiblePages: number[] = [];

  ngOnChanges(): void {
    this.totalPages = Math.ceil(this.totalItems / this.pageSize);
    this.updateVisiblePages();
  }

  onPageChange(page: number): void {
    if (page >= 1 && page <= this.totalPages && page !== this.currentPage) {
      this.pageChange.emit(page);
    }
  }

  private updateVisiblePages(): void {
    const half = Math.floor(this.maxVisiblePages / 2);
    let start = Math.max(1, this.currentPage - half);
    let end = Math.min(this.totalPages, start + this.maxVisiblePages - 1);

    if (end - start + 1 < this.maxVisiblePages) {
      start = Math.max(1, end - this.maxVisiblePages + 1);
    }

    this.visiblePages = Array.from(
      { length: end - start + 1 },
      (_, i) => start + i
    );
  }
}

// الاستخدام
export class UserListComponent {
  users: User[] = [];
  currentPage = 1;
  totalItems = 0;
  pageSize = 10;

  constructor(private userService: UserService) { }

  ngOnInit(): void {
    this.loadUsers();
  }

  loadUsers(): void {
    this.userService.getUsers(this.currentPage, this.pageSize)
      .subscribe(response => {
        this.users = response.data;
        this.totalItems = response.total;
      });
  }

  onPageChange(page: number): void {
    this.currentPage = page;
    this.loadUsers();
  }
}
```

```html
<!-- user-list.component.html -->
<div *ngFor="let user of users">
  {{ user.name }}
</div>

<app-pagination
  [currentPage]="currentPage"
  [totalItems]="totalItems"
  [pageSize]="pageSize"
  (pageChange)="onPageChange($event)">
</app-pagination>
```

---

## نصائح للمقابلات

### كيف تستعد؟

**1. أتقن الأساسيات:**
- Components, Services, Modules
- Data Binding
- Dependency Injection
- Lifecycle Hooks

**2. تدرب على الكود:**
- اكتب 10-15 component مختلف
- جرب RxJS operators
- اصنع mini-projects

**3. افهم المفاهيم المتقدمة:**
- Change Detection
- NgRx
- Performance Optimization

**4. جهّز أمثلة:**
- مشاريع عملت عليها
- مشاكل حللتها
- قرارات تقنية اتخذتها

---

## الخلاصة

### النقاط الأساسية للنجاح:

**المعرفة النظرية:**
- ✅ فهم عميق للـ Core concepts
- ✅ معرفة الفروقات
- ✅ متى تستخدم ماذا

**المهارات العملية:**
- ✅ كتابة كود نظيف
- ✅ حل المشاكل
- ✅ التفكير في الأداء

**التواصل:**
- ✅ شرح واضح
- ✅ أمثلة عملية
- ✅ الاستماع جيداً

---

**بالتوفيق في مقابلتك! 🎯**

تذكّر: Angular واسع جداً، ركّز على الأساسيات والمفاهيم الأكثر استخداماً!

---

**تم إنشاء هذا الدليل كمرجع شامل لأسئلة مقابلات Angular**