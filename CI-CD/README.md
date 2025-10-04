# CI/CD - Continuous Integration & Continuous Deployment

## المحتويات
- [ما هو CI/CD؟](#ما-هو-cicd)
- [أسئلة المقابلات](CI-CD-Interview-Questions.md)
- [الأدوات الشائعة](#الأدوات-الشائعة)

---

## ما هو CI/CD؟

### CI - Continuous Integration (التكامل المستمر)
عملية دمج الكود تلقائياً واختباره عند كل commit.

```
Developer Push Code → Build → Test → Report
```

**الفوائد:**
✅ اكتشاف الأخطاء مبكراً
✅ جودة كود أفضل
✅ تكامل أسرع

### CD - Continuous Deployment (النشر المستمر)
عملية نشر الكود تلقائياً للـ Production بعد نجاح الاختبارات.

```
Code → Build → Test → Deploy to Staging → Deploy to Production
```

**الفوائد:**
✅ نشر أسرع
✅ تقليل الأخطاء البشرية
✅ Rollback سريع

---

## CI/CD Pipeline

```
┌─────────────┐
│  Git Push   │
└──────┬──────┘
       │
┌──────▼──────┐
│   Checkout  │ ← جلب الكود
└──────┬──────┘
       │
┌──────▼──────┐
│    Build    │ ← بناء التطبيق
└──────┬──────┘
       │
┌──────▼──────┐
│    Test     │ ← تشغيل الاختبارات
└──────┬──────┘
       │
┌──────▼──────┐
│  Code Scan  │ ← فحص الكود (SonarQube)
└──────┬──────┘
       │
┌──────▼──────┐
│Build Docker │ ← بناء Image
└──────┬──────┘
       │
┌──────▼──────┐
│Push Registry│ ← رفع للـ Registry
└──────┬──────┘
       │
┌──────▼──────┐
│Deploy Staging│ ← نشر للاختبار
└──────┬──────┘
       │
┌──────▼──────┐
│ Manual Approve│ ← موافقة يدوية
└──────┬──────┘
       │
┌──────▼──────┐
│Deploy Prod  │ ← نشر Production
└─────────────┘
```

---

## الأدوات الشائعة

### CI/CD Platforms:
- **Jenkins** (الأكثر شيوعاً)
- **GitLab CI/CD**
- **GitHub Actions**
- **CircleCI**
- **Travis CI**
- **Azure DevOps**
- **AWS CodePipeline**

### Container & Orchestration:
- **Docker**
- **Kubernetes**
- **OpenShift Pipelines (Tekton)**

### Code Quality:
- **SonarQube**
- **ESLint**
- **CheckStyle**

### Testing:
- **JUnit** (Java)
- **Jest** (JavaScript)
- **PyTest** (Python)
- **Selenium** (E2E)

---

**للمزيد، راجع [أسئلة المقابلات](CI-CD-Interview-Questions.md)**
