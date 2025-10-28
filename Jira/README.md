# Jira - دليل شامل

## المحتويات
1. [ما هو Jira](#ما-هو-jira)
2. [أنواع المشاريع](#أنواع-المشاريع)
3. [Issue Types](#issue-types)
4. [Workflows](#workflows)
5. [Agile Boards](#agile-boards)
6. [JQL - Jira Query Language](#jql---jira-query-language)
7. [Automation](#automation)
8. [Reports & Dashboards](#reports--dashboards)
9. [Best Practices](#best-practices)
10. [أسئلة المقابلات](#أسئلة-المقابلات)

---

## ما هو Jira

**Jira** هو أداة لإدارة المشاريع وتتبع المهام من Atlassian، تُستخدم على نطاق واسع في فرق تطوير البرمجيات.

### المزايا الرئيسية
- **Issue Tracking**: تتبع المهام والأخطاء
- **Agile Support**: دعم Scrum و Kanban
- **Customizable Workflows**: سير عمل قابل للتخصيص
- **Integration**: تكامل مع Git، Confluence، Slack
- **Reporting**: تقارير تفصيلية وDashboards
- **Automation**: أتمتة المهام المتكررة

### النسخ المتاحة
- **Jira Work Management**: لفرق Business
- **Jira Software**: لفرق Development (الأشهر)
- **Jira Service Management**: لفرق IT Support

---

## أنواع المشاريع

### 1. Team-managed Project
- بسيط وسريع التهيئة
- لفرق صغيرة
- إدارة مستقلة
- Workflow مبسط

### 2. Company-managed Project
- متقدم وقابل للتخصيص الكامل
- لفرق كبيرة ومؤسسات
- Workflows معقدة
- Permissions متقدمة

### Project Types حسب Methodology

#### Scrum Project
```
Sprint Planning → Sprint → Daily Standup → Sprint Review → Retrospective
```

**الميزات:**
- Sprints (1-4 أسابيع)
- Sprint Planning
- Backlog Management
- Burndown Charts
- Velocity Tracking

#### Kanban Project
```
Backlog → Selected for Development → In Progress → Done
```

**الميزات:**
- Continuous Flow
- WIP Limits
- Cumulative Flow Diagram
- Cycle Time
- No Sprints

---

## Issue Types

### 1. Epic
**أكبر وحدة عمل** - ميزة كبيرة تحتوي على Stories متعددة.

```
Epic: User Authentication System
├── Story: Login Page
├── Story: Registration Page
├── Story: Password Reset
└── Story: OAuth Integration
```

**مثال:**
- **Name**: E-commerce Checkout System
- **Description**: تطوير نظام الدفع الكامل
- **Time**: 2-3 Sprints

### 2. Story (User Story)
**وحدة عمل من منظور المستخدم**.

**Format:**
```
As a [user type],
I want to [action],
So that [benefit].
```

**مثال:**
```
Title: User Login
Description: As a registered user, I want to login to my account,
             so that I can access my personalized dashboard.

Acceptance Criteria:
- ✅ Email and password fields
- ✅ Remember me checkbox
- ✅ Forgot password link
- ✅ Error messages for invalid credentials
```

**Story Points**: 1, 2, 3, 5, 8, 13 (Fibonacci)

### 3. Task
**مهمة تقنية** لا تتعلق مباشرة بالمستخدم.

**أمثلة:**
- إعداد Database Schema
- تحديث Dependencies
- Code Refactoring
- Writing Documentation

### 4. Bug
**خطأ في النظام** يحتاج إصلاح.

**الحقول:**
- **Priority**: Highest, High, Medium, Low, Lowest
- **Severity**: Critical, Major, Minor, Trivial
- **Environment**: Production, Staging, Dev
- **Steps to Reproduce**: خطوات إعادة الخطأ
- **Expected Result**: النتيجة المتوقعة
- **Actual Result**: النتيجة الفعلية

**مثال:**
```
Title: Login fails with valid credentials
Priority: High
Severity: Critical
Environment: Production

Steps to Reproduce:
1. Navigate to /login
2. Enter valid email: user@example.com
3. Enter valid password
4. Click Login button

Expected: User logged in successfully
Actual: Error 500 - Internal Server Error
```

### 5. Subtask
**مهمة فرعية** تحت Story أو Task.

**مثال تحت Story "User Login":**
- Design Login UI
- Implement Frontend Form
- Create Login API Endpoint
- Add Validation
- Write Unit Tests

---

## Workflows

### Basic Workflow

```
TO DO → IN PROGRESS → DONE
```

### Advanced Workflow

```
                    ┌─────────────┐
                    │   BACKLOG   │
                    └──────┬──────┘
                           │
                    ┌──────▼──────┐
                    │   TO DO     │
                    └──────┬──────┘
                           │
                    ┌──────▼──────┐
                    │ IN PROGRESS │
                    └──┬───────┬──┘
                       │       │
            ┌──────────▼──┐ ┌──▼──────────┐
            │ CODE REVIEW │ │  IN TESTING │
            └──────┬──────┘ └──┬──────────┘
                   │            │
                   └─────┬──────┘
                         │
                   ┌─────▼─────┐
                   │   DONE    │
                   └───────────┘
```

### Custom Workflow Example (Bug Workflow)

```
NEW → OPEN → IN PROGRESS → RESOLVED → CLOSED
                    ↓
               REOPENED
```

### Workflow Transitions

**Transition**: الانتقال من حالة لأخرى

**Conditions**: شروط الانتقال
- User has permission
- Issue has required fields
- Specific status

**Validators**: التحقق قبل الانتقال
- All subtasks closed
- Required fields filled
- Resolution set

**Post Functions**: إجراءات بعد الانتقال
- Assign to user
- Send email
- Update field
- Create subtask

---

## Agile Boards

### Scrum Board

#### 1. Backlog
قائمة جميع Stories والTasks المطلوبة.

**Backlog Refinement:**
- تحديد الأولويات
- تقدير Story Points
- تقسيم Stories الكبيرة
- كتابة Acceptance Criteria

#### 2. Sprint
فترة زمنية محددة (1-4 أسابيع) لإنجاز Stories.

**Sprint Planning:**
```
Sprint Goal: Complete User Profile Module
Capacity: 40 Story Points
Duration: 2 weeks

Selected Stories:
- [5 points] User Profile Page
- [8 points] Edit Profile
- [5 points] Profile Picture Upload
- [3 points] Change Password
- [8 points] Account Settings
Total: 29 points
```

**Sprint Ceremonies:**
1. **Sprint Planning** (بداية Sprint)
2. **Daily Standup** (يومي، 15 دقيقة)
   - What did I do yesterday?
   - What will I do today?
   - Any blockers?
3. **Sprint Review** (نهاية Sprint)
   - Demo للميزات الجديدة
4. **Sprint Retrospective** (بعد Review)
   - What went well?
   - What didn't go well?
   - What can we improve?

#### 3. Sprint Reports

**Burndown Chart:**
```
Story Points
    │
 40 │●
 35 │ ●
 30 │  ●●
 25 │    ●●
 20 │      ●
 15 │       ●●
 10 │         ●
  5 │          ●
  0 └──────────────────► Days
    1  2  3  4  5  6  7  8  9  10
```

**Velocity Chart:**
```
Story Points
    │
 40 │    ███
 35 │    ███ ███
 30 │    ███ ███
 25 │ ███ ███ ███
 20 │ ███ ███ ███ ███
    └──────────────────────
     S1  S2  S3  S4
```

### Kanban Board

#### Columns
```
┌─────────┬──────────────┬─────────────┬─────────┐
│ BACKLOG │  TO DO (5)   │ IN PROG (3) │  DONE   │
├─────────┼──────────────┼─────────────┼─────────┤
│ STORY-1 │ STORY-5      │ STORY-8     │ STORY-2 │
│ STORY-3 │ STORY-6      │ STORY-9     │ STORY-4 │
│ STORY-7 │ STORY-10     │ STORY-11    │ STORY-12│
│ ...     │ STORY-14     │             │ ...     │
│         │ STORY-15     │             │         │
└─────────┴──────────────┴─────────────┴─────────┘
         WIP Limit: 5    WIP Limit: 3
```

**WIP Limits (Work in Progress):**
- تحديد عدد Issues في كل عمود
- تحسين التدفق
- تقليل Multitasking

**Kanban Metrics:**
- **Cycle Time**: الوقت من "In Progress" إلى "Done"
- **Lead Time**: الوقت من "To Do" إلى "Done"
- **Throughput**: عدد Issues المكتملة في فترة

---

## JQL - Jira Query Language

### Basic Syntax

```jql
field operator value
```

### Common Fields

```jql
project = "PROJECT_KEY"
assignee = currentUser()
status = "In Progress"
priority = High
created >= -7d
updated <= -1w
sprint = "Sprint 23"
labels = "bug"
fixVersion = "1.0"
```

### Operators

| Operator | الاستخدام |
|----------|-----------|
| `=` | يساوي |
| `!=` | لا يساوي |
| `>` | أكبر من |
| `<` | أصغر من |
| `>=` | أكبر من أو يساوي |
| `<=` | أصغر من أو يساوي |
| `IN` | ضمن قائمة |
| `NOT IN` | ليس ضمن قائمة |
| `~` | يحتوي على |
| `IS EMPTY` | فارغ |
| `IS NOT EMPTY` | غير فارغ |

### أمثلة عملية

#### 1. جميع Bugs في المشروع
```jql
project = PROJ AND type = Bug
```

#### 2. المهام المفتوحة المُسندة لي
```jql
assignee = currentUser() AND status NOT IN (Done, Closed)
```

#### 3. Stories بأولوية عالية في Sprint الحالي
```jql
type = Story AND priority IN (High, Highest) AND sprint IN openSprints()
```

#### 4. Bugs تم إنشاؤها آخر 7 أيام
```jql
type = Bug AND created >= -7d ORDER BY priority DESC
```

#### 5. Issues بدون Assignee
```jql
assignee IS EMPTY AND status != Done
```

#### 6. Tasks المتأخرة
```jql
duedate < now() AND status NOT IN (Done, Closed)
```

#### 7. Stories في Sprint معين
```jql
type = Story AND sprint = "Sprint 23"
```

#### 8. جميع Issues التي تم تحديثها الأسبوع الماضي
```jql
updated >= -1w AND updated < now()
```

#### 9. High Priority Bugs في Production
```jql
type = Bug
AND priority IN (High, Highest)
AND labels = "production"
AND status NOT IN (Resolved, Closed)
ORDER BY created ASC
```

#### 10. Stories التي أنشأتها أنا ولم تكتمل بعد
```jql
type = Story
AND creator = currentUser()
AND status != Done
```

### Functions

```jql
currentUser()           # المستخدم الحالي
now()                   # الوقت الحالي
startOfDay()            # بداية اليوم
endOfDay()              # نهاية اليوم
startOfWeek()           # بداية الأسبوع
openSprints()           # Sprints المفتوحة
closedSprints()         # Sprints المغلقة
membersOf("group")      # أعضاء مجموعة
```

### Advanced JQL

```jql
# Issues في مشاريع متعددة
project IN (PROJ1, PROJ2, PROJ3) AND status = "In Progress"

# Issues بـ Label محدد أو Priority عالية
(labels = urgent OR priority = Highest) AND status != Done

# Stories في Sprint الحالي للفريق
type = Story
AND sprint IN openSprints()
AND assignee IN membersOf("dev-team")

# Bugs التي لم يتم حلها لأكثر من 30 يوم
type = Bug
AND status NOT IN (Resolved, Closed)
AND created <= -30d
ORDER BY priority DESC, created ASC
```

---

## Automation

### Automation Rules

**البنية:**
```
WHEN: Trigger (متى تنفذ)
IF: Conditions (شروط)
THEN: Actions (إجراءات)
```

### أمثلة Automation Rules

#### 1. Auto-assign Bugs
```
WHEN: Issue created
IF: Issue Type = Bug
THEN: Assign to QA Lead
```

#### 2. Notify on High Priority
```
WHEN: Issue updated
IF: Priority changed to High or Highest
THEN: Send email to Project Manager
```

#### 3. Auto-close Resolved Issues
```
WHEN: Issue moved to Resolved
IF: Status = Resolved for 7 days
THEN: Transition to Closed
```

#### 4. Label based on Priority
```
WHEN: Issue created
IF: Priority = Highest
THEN: Add label "urgent"
```

#### 5. Comment on Status Change
```
WHEN: Issue transitioned
IF: Status changed to In Progress
THEN: Add comment "Work started by {{assignee}}"
```

#### 6. Auto-update Sprint
```
WHEN: Issue transitioned to Done
IF: Sprint is active
THEN: Log work and send Slack notification
```

### Smart Values

```
{{issue.key}}           # PROJ-123
{{issue.summary}}       # عنوان Issue
{{issue.assignee}}      # المُسند له
{{issue.priority}}      # الأولوية
{{now}}                 # الوقت الحالي
{{reporter}}            # من أنشأ Issue
{{created}}             # تاريخ الإنشاء
{{sprint.name}}         # اسم Sprint
```

---

## Reports & Dashboards

### Reports

#### 1. Burndown Chart
يوضح تقدم Sprint يومياً.

**الاستخدام:**
- متابعة Sprint Progress
- كشف إذا كان Sprint سيكتمل في الوقت

#### 2. Burnup Chart
يوضح العمل المكتمل مقابل Total Scope.

#### 3. Velocity Chart
سرعة الفريق (Story Points per Sprint).

**الفائدة:**
- تخطيط Sprint القادم
- تحديد Capacity

#### 4. Cumulative Flow Diagram
توزيع Issues عبر Status.

**يكشف:**
- Bottlenecks
- WIP Limits
- Flow Efficiency

#### 5. Sprint Report
ملخص Sprint:
- Committed vs Completed
- Added/Removed Stories
- Incomplete Issues

#### 6. Epic Report
تقدم Epic عبر Sprints.

#### 7. Control Chart
Cycle Time و Lead Time لكل Issue.

#### 8. Version Report
Issues في Version/Release معين.

### Dashboards

**Dashboard**: صفحة تحتوي على Gadgets متعددة.

#### Gadgets شائعة

1. **Filter Results**
   - عرض نتائج JQL Filter

2. **Pie Chart**
   - توزيع Issues حسب Status، Priority، Assignee

3. **Activity Stream**
   - آخر التحديثات

4. **Sprint Burndown**
   - Burndown للـ Sprint الحالي

5. **Created vs Resolved**
   - مقارنة Issues الجديدة والمحلولة

6. **Average Age**
   - متوسط عمر Open Issues

#### Dashboard مثالي لـ Scrum Team

```
┌───────────────────────┬───────────────────────┐
│ Sprint Burndown       │ Sprint Velocity       │
├───────────────────────┼───────────────────────┤
│ Issues by Status (Pie)│ Assigned to Me        │
├───────────────────────┴───────────────────────┤
│ Activity Stream                               │
└───────────────────────────────────────────────┘
```

---

## Best Practices

### 1. Issue Creation

#### Story Best Practices
```
✅ GOOD:
Title: User can reset password via email
Description:
  As a registered user,
  I want to receive a password reset link via email,
  So that I can regain access to my account if I forget my password.

  Acceptance Criteria:
  - User clicks "Forgot Password"
  - User enters email
  - System sends reset link (valid 1 hour)
  - User clicks link and sets new password

  Story Points: 5

❌ BAD:
Title: Password thing
Description: Make password reset work
```

#### Bug Best Practices
```
✅ GOOD:
Title: Login fails with 500 error for Gmail users
Priority: High
Severity: Critical
Environment: Production

Steps to Reproduce:
1. Go to /login
2. Enter email: user@gmail.com
3. Enter password
4. Click Login

Expected: User logs in
Actual: 500 Internal Server Error

Logs:
[ERROR] NullPointerException at UserService.java:42

❌ BAD:
Title: Login broken
Description: It doesn't work
```

### 2. Workflow Design

**قواعد:**
- ✅ Keep it simple (3-5 statuses)
- ✅ واضح ومفهوم للجميع
- ✅ يعكس عملية الفريق الفعلية
- ❌ لا تعقده بدون سبب

**Minimal Workflow:**
```
TO DO → IN PROGRESS → DONE
```

**Standard Workflow:**
```
BACKLOG → TO DO → IN PROGRESS → CODE REVIEW → TESTING → DONE
```

### 3. Sprint Planning

**قبل Planning:**
- Backlog Refinement
- Stories جاهزة (Definition of Ready)
- Story Points محددة

**أثناء Planning:**
- Sprint Goal واضح
- Team Capacity معروف
- لا تفرط في Commitment

**Definition of Ready:**
- ✅ Story واضحة ومفهومة
- ✅ Acceptance Criteria محددة
- ✅ Story Points محددة
- ✅ Dependencies معروفة
- ✅ لا يوجد Blockers

**Definition of Done:**
- ✅ Code مكتوب ومراجع
- ✅ Tests مكتوبة وناجحة
- ✅ Documentation محدثة
- ✅ Deployed على Staging
- ✅ Acceptance Criteria مستوفاة

### 4. JQL Filters

**أنشئ Filters مفيدة:**
```jql
# My Open Issues
assignee = currentUser() AND status NOT IN (Done, Closed)

# High Priority This Week
priority IN (High, Highest) AND created >= -7d

# Blocked Issues
status = Blocked

# Overdue Tasks
duedate < now() AND status != Done

# Bugs in Production
type = Bug AND labels = "production" AND status != Closed
```

**احفظها كـ Favorites** للوصول السريع.

### 5. Board Management

**Scrum Board:**
- ✅ Sprint Goal واضح
- ✅ Daily cleanup
- ✅ Update progress يومياً
- ✅ Blockers واضحة

**Kanban Board:**
- ✅ WIP Limits محددة
- ✅ Column Definitions واضحة
- ✅ Review Board يومياً
- ✅ Optimize Flow

### 6. Naming Conventions

**Projects:**
```
PROJ  - Project Name
ECOM  - E-commerce
MOB   - Mobile App
```

**Epics:**
```
[Module] Feature Name
[Auth] User Authentication
[Payment] Checkout System
```

**Stories:**
```
User can [action]
Admin can [action]
System should [action]
```

**Bugs:**
```
[Component] Brief description
[Login] 500 error on submit
[API] Timeout on user list
```

### 7. Labels

**استخدم Labels للتصنيف:**
- `frontend`, `backend`, `mobile`
- `urgent`, `technical-debt`
- `production`, `staging`
- `bug`, `enhancement`

### 8. Components

**للمشاريع الكبيرة:**
- User Interface
- Backend API
- Database
- Mobile App
- Documentation

### 9. Communication

**Comments:**
- ✅ استخدم @mention للإشعارات
- ✅ وضح السياق
- ✅ أرفق Screenshots عند الحاجة
- ❌ لا تستخدم Comments للمناقشات الطويلة (استخدم Confluence)

**Status Updates:**
- حدث Status بانتظام
- أضف تعليق عند Status Change مهم
- Tag الأشخاص المعنيين

---

## أسئلة المقابلات

### أسئلة أساسية

**1. ما هو Jira ولماذا نستخدمه؟**
- **الإجابة**: Jira هو أداة لإدارة المشاريع وتتبع المهام من Atlassian. نستخدمه لـ:
  - Issue Tracking
  - Agile Project Management (Scrum/Kanban)
  - Sprint Planning
  - Bug Tracking
  - Reporting & Analytics

**2. ما الفرق بين Epic و Story و Task؟**
- **الإجابة**:
  - **Epic**: أكبر وحدة، ميزة كبيرة تحتوي على Stories (مثل: نظام الدفع الكامل)
  - **Story**: وحدة عمل من منظور المستخدم (مثل: User can login)
  - **Task**: مهمة تقنية غير مرتبطة بالمستخدم (مثل: إعداد Database)

**3. ما هو Workflow في Jira؟**
- **الإجابة**: Workflow هو سير العمل الذي يمر به Issue من البداية للنهاية. مثل:
  ```
  TO DO → IN PROGRESS → CODE REVIEW → TESTING → DONE
  ```

**4. ما الفرق بين Scrum و Kanban في Jira؟**
- **الإجابة**:
  - **Scrum**: يعمل بـ Sprints (فترات ثابتة)، Sprint Planning، Retrospectives
  - **Kanban**: Continuous Flow، WIP Limits، بدون Sprints

**5. ما هو JQL؟**
- **الإجابة**: JQL (Jira Query Language) لغة استعلام للبحث في Jira. مثال:
  ```jql
  assignee = currentUser() AND status = "In Progress"
  ```

**6. ما هو Sprint في Jira؟**
- **الإجابة**: Sprint هو فترة زمنية محددة (1-4 أسابيع) في Scrum يتم فيها إنجاز مجموعة من Stories.

**7. ما هو Backlog؟**
- **الإجابة**: Backlog هو قائمة جميع Stories والTasks المطلوبة والتي لم تُنجز بعد، مرتبة حسب الأولوية.

**8. ما هي Story Points؟**
- **الإجابة**: Story Points هي وحدة قياس نسبية لتقدير الجهد المطلوب لإنجاز Story، غالباً تستخدم Fibonacci (1, 2, 3, 5, 8, 13).

**9. ما هو Burndown Chart؟**
- **الإجابة**: Burndown Chart يوضح تقدم Sprint يومياً، يظهر Story Points المتبقية مقابل الأيام.

**10. ما هو Subtask؟**
- **الإجابة**: Subtask هو مهمة فرعية تحت Story أو Task، لتقسيم العمل إلى أجزاء أصغر.

### أسئلة متقدمة

**11. كيف تكتب User Story جيدة؟**
- **الإجابة**: User Story جيدة تتبع Format:
  ```
  As a [user type],
  I want to [action],
  So that [benefit].
  ```
  مع Acceptance Criteria واضحة و Story Points محددة.

**12. ما هو Definition of Done (DoD)؟**
- **الإجابة**: DoD هي معايير يجب استيفاؤها قبل اعتبار Story مكتملة:
  - Code مكتوب ومراجع
  - Tests ناجحة
  - Documentation محدثة
  - Deployed على Staging
  - Acceptance Criteria مستوفاة

**13. كيف تدير Bugs في Jira؟**
- **الإجابة**:
  - إنشاء Bug issue type
  - تحديد Priority و Severity
  - كتابة Steps to Reproduce
  - تعيين للمطور المناسب
  - تتبع عبر Workflow: New → Open → In Progress → Resolved → Closed

**14. ما هي WIP Limits في Kanban؟**
- **الإجابة**: WIP Limits (Work in Progress) تحدد العدد الأقصى من Issues في كل عمود لـ:
  - تقليل Multitasking
  - تحسين Flow
  - كشف Bottlenecks

**15. كيف تستخدم JQL للبحث المتقدم؟**
- **الإجابة**: أمثلة:
  ```jql
  # High Priority Bugs في Production
  type = Bug AND priority = High AND labels = "production"

  # Issues المتأخرة
  duedate < now() AND status != Done

  # My Open Issues في Sprint الحالي
  assignee = currentUser() AND sprint IN openSprints()
  ```

**16. ما هو Automation في Jira؟**
- **الإجابة**: Automation تسمح بأتمتة المهام المتكررة:
  ```
  WHEN: Issue created
  IF: Priority = Highest
  THEN: Assign to Manager AND Send notification
  ```

**17. كيف تقيس أداء الفريق في Jira؟**
- **الإجابة**: من خلال:
  - **Velocity**: Story Points per Sprint
  - **Cycle Time**: وقت إنجاز Issue
  - **Burndown Chart**: تقدم Sprint
  - **Cumulative Flow**: توزيع Issues

**18. ما هو Sprint Retrospective؟**
- **الإجابة**: اجتماع في نهاية Sprint لمراجعة:
  - What went well?
  - What didn't go well?
  - What can we improve?

  والتخطيط لتحسينات في Sprint القادم.

**19. كيف تتعامل مع Blocked Issues؟**
- **الإجابة**:
  - تحديد Status = Blocked
  - إضافة Comment يوضح السبب
  - Tag الأشخاص المعنيين
  - متابعة في Daily Standup
  - حل Blocker بأسرع وقت

**20. ما الفرق بين Component و Label؟**
- **الإجابة**:
  - **Component**: تصنيف رئيسي ثابت (Frontend, Backend, Mobile)
  - **Label**: تصنيف مرن يمكن إضافته بحرية (urgent, bug, technical-debt)

**21. كيف تخطط Sprint بفعالية؟**
- **الإجابة**:
  1. Backlog Refinement قبل Planning
  2. تحديد Sprint Goal واضح
  3. حساب Team Capacity
  4. اختيار Stories بناءً على Priority و Capacity
  5. تقسيم Stories إلى Subtasks
  6. Commitment من الفريق

**22. ما هو Velocity Chart ولماذا مهم؟**
- **الإجابة**: Velocity Chart يوضح Story Points المكتملة في كل Sprint. مهم لـ:
  - تخطيط Sprint القادم
  - تحديد Team Capacity
  - كشف التحسن أو التراجع في الأداء

**23. كيف تكتب Bug Report احترافي؟**
- **الإجابة**:
  ```
  Title: [Component] واضح ومختصر
  Priority: High/Medium/Low
  Severity: Critical/Major/Minor
  Environment: Production/Staging

  Steps to Reproduce:
  1. ...
  2. ...
  3. ...

  Expected Result: ...
  Actual Result: ...

  Screenshots/Logs: ...
  ```

**24. ما هو Epic Report؟**
- **الإجابة**: Epic Report يوضح:
  - تقدم Epic عبر Sprints
  - Stories المكتملة vs المتبقية
  - Story Points المكتملة
  - Expected completion date

**25. كيف تدمج Jira مع Git؟**
- **الإجابة**:
  - ربط Jira مع GitHub/GitLab/Bitbucket
  - استخدام Issue Key في Commit Messages:
    ```
    PROJ-123: Fix login bug
    ```
  - يظهر Commits في Jira Issue
  - يمكن Transition من Git (Smart Commits):
    ```
    PROJ-123 #done Fix applied
    ```

**26. ما هي Smart Values في Automation؟**
- **الإجابة**: Smart Values هي متغيرات ديناميكية:
  ```
  {{issue.key}}           # PROJ-123
  {{issue.summary}}       # عنوان Issue
  {{assignee.displayName}}# اسم المُسند له
  {{now}}                 # الوقت الحالي
  ```

**27. كيف تحسن Workflow لفريقك؟**
- **الإجابة**:
  1. فهم عملية الفريق الحالية
  2. تحديد Bottlenecks
  3. تبسيط Statuses (إزالة غير الضرورية)
  4. إضافة Validations و Post Functions
  5. مراجعة دورية وتحسين مستمر

**28. ما هو Cumulative Flow Diagram؟**
- **الإجابة**: CFD يوضح توزيع Issues عبر Statuses مع الوقت. يكشف:
  - Bottlenecks (تراكم في Status معين)
  - WIP (Work in Progress)
  - Throughput (معدل الإنجاز)

**29. كيف تتعامل مع Scope Creep في Sprint؟**
- **الإجابة**:
  - **Scope Creep**: إضافة Stories جديدة لـ Sprint بعد بدايته
  - **الحل**:
    - رفض إضافة Stories إلا للضرورة القصوى
    - إذا أُضيفت Story، يجب إزالة Story أخرى
    - مناقشة في Daily Standup
    - حماية Sprint Goal

**30. ما الفرق بين Filter و Dashboard؟**
- **الإجابة**:
  - **Filter**: JQL Query محفوظ لعرض Issues معينة
  - **Dashboard**: صفحة تحتوي على Gadgets متعددة (Charts, Filters, Reports)

---

## الخلاصة

Jira هو أداة قوية لإدارة المشاريع Agile:
- ✅ Issue Tracking شامل
- ✅ Scrum و Kanban Support
- ✅ Customizable Workflows
- ✅ JQL للبحث المتقدم
- ✅ Automation للمهام المتكررة
- ✅ Reports و Dashboards تفصيلية
- ✅ Integration مع أدوات أخرى

**Best Practices:**
- Write clear User Stories
- Keep Workflows simple
- Use JQL Filters
- Automate repetitive tasks
- Review Reports regularly
- Update Issues daily
- Communicate effectively

**الأدوات المكملة:**
- **Confluence**: Documentation
- **Bitbucket/GitHub**: Code Repository
- **Slack**: Team Communication
- **Zephyr**: Test Management