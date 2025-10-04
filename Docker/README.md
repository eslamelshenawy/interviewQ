# Docker

## المحتويات
- [ما هو Docker؟](#ما-هو-docker)
- [أسئلة المقابلات](Docker-Interview-Questions.md)
- [المقارنة مع Kubernetes و OpenShift](../Comparison.md)

---

## ما هو Docker؟

**Docker** هو منصة لبناء وتشغيل ونشر التطبيقات باستخدام **Containers**.

### المفاهيم الأساسية:

#### 1. **Container**
بيئة معزولة تحتوي على كل ما يحتاجه التطبيق للعمل.

```bash
# تشغيل container
docker run nginx
```

#### 2. **Image**
قالب للـ container (مثل Class والـ Container مثل Object).

```bash
# بناء image
docker build -t myapp:1.0 .

# قائمة الـ images
docker images
```

#### 3. **Dockerfile**
ملف نصي يحتوي على تعليمات بناء الـ image.

```dockerfile
FROM node:14
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
CMD ["npm", "start"]
```

#### 4. **Docker Hub**
Registry عام للـ Docker images.

```bash
# رفع image
docker push myusername/myapp:1.0

# تحميل image
docker pull nginx
```

---

## لماذا Docker؟

### قبل Docker:
```
المشاكل:
❌ "Works on my machine"
❌ اختلاف البيئات (Development vs Production)
❌ صعوبة Setup
❌ تعارض Dependencies
```

### مع Docker:
```
الحلول:
✅ بيئة موحدة
✅ سهولة النشر
✅ عزل التطبيقات
✅ استخدام أمثل للموارد
```

---

## المكونات الأساسية:

```
Docker Architecture:
├── Docker Client (CLI)
├── Docker Daemon (Server)
├── Docker Registry (Docker Hub)
└── Docker Objects
    ├── Images
    ├── Containers
    ├── Networks
    └── Volumes
```

---

## الأوامر الأساسية:

```bash
# Container Management
docker run [image]
docker ps
docker stop [container]
docker rm [container]

# Image Management
docker build -t [name] .
docker images
docker rmi [image]

# Logs and Debugging
docker logs [container]
docker exec -it [container] bash
```

---

## متى تستخدم Docker؟

✅ **استخدم Docker عندما:**
- تريد بيئة موحدة
- تطوير Microservices
- CI/CD Pipelines
- نشر سريع وسهل

**أمثلة:**
- تطبيقات Web
- APIs
- Databases في Development
- Testing Environments

---

**للمزيد من التفاصيل، راجع [أسئلة المقابلات](Docker-Interview-Questions.md)**
