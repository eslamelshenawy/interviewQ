# أسئلة مقابلات Docker - دليل شامل

## جدول المحتويات
1. [أساسيات Docker](#أساسيات-docker)
2. [أوامر Docker](#أوامر-docker)
3. [شبكات Docker](#شبكات-docker)
4. [Docker Volumes](#docker-volumes)
5. [Docker Compose](#docker-compose)
6. [أفضل ممارسات Dockerfile](#أفضل-ممارسات-dockerfile)
7. [Multi-stage Builds](#multi-stage-builds)
8. [أمان Docker](#أمان-docker)
9. [استكشاف الأخطاء وإصلاحها](#استكشاف-الأخطاء-وإصلاحها)
10. [سيناريوهات واقعية](#سيناريوهات-واقعية)

---

## أساسيات Docker

### 1. ما هو Docker؟
**الإجابة:**
Docker هو منصة مفتوحة المصدر لتطوير ونشر وتشغيل التطبيقات في حاويات (Containers). يسمح بتغليف التطبيق مع جميع تبعياته في حاوية واحدة يمكن تشغيلها على أي نظام.

**المميزات الرئيسية:**
- عزل التطبيقات (Isolation)
- قابلية النقل (Portability)
- كفاءة الموارد (Resource Efficiency)
- سرعة النشر (Fast Deployment)

**مثال:**
```bash
# تشغيل أول حاوية Docker
docker run hello-world
```

### 2. ما الفرق بين Container و Image؟
**الإجابة:**

**Image (الصورة):**
- ملف للقراءة فقط (Read-only)
- تحتوي على التطبيق وجميع تبعياته
- قالب لإنشاء الحاويات
- يتم بناؤها من Dockerfile

**Container (الحاوية):**
- نسخة قابلة للتشغيل من الصورة
- حالة تشغيل (Runtime instance)
- يمكن إيقافها وبدءها وحذفها
- لها طبقة قابلة للكتابة

**مثال:**
```bash
# بناء صورة
docker build -t myapp:1.0 .

# إنشاء حاوية من الصورة
docker run -d --name mycontainer myapp:1.0

# الصورة واحدة، لكن يمكن إنشاء عدة حاويات منها
docker run -d --name mycontainer2 myapp:1.0
```

### 3. ما هو Dockerfile؟
**الإجابة:**
Dockerfile هو ملف نصي يحتوي على تعليمات لبناء صورة Docker. كل تعليمة تنشئ طبقة (layer) جديدة في الصورة.

**مثال - Dockerfile لتطبيق Node.js:**
```dockerfile
# الصورة الأساسية
FROM node:18-alpine

# تحديد مجلد العمل
WORKDIR /app

# نسخ ملفات package.json
COPY package*.json ./

# تثبيت التبعيات
RUN npm install --production

# نسخ بقية الملفات
COPY . .

# تحديد المنفذ
EXPOSE 3000

# الأمر الافتراضي للتشغيل
CMD ["node", "server.js"]
```

### 4. ما هي طبقات Docker Layers؟
**الإجابة:**
كل تعليمة في Dockerfile تنشئ طبقة جديدة. الطبقات قابلة للقراءة فقط ويتم مشاركتها بين الصور لتوفير المساحة.

**فوائد الطبقات:**
- إعادة استخدام الطبقات المشتركة
- تسريع بناء الصور
- توفير مساحة التخزين
- كفاءة التخزين المؤقت (Caching)

**مثال:**
```dockerfile
# كل سطر ينشئ طبقة جديدة
FROM ubuntu:20.04          # Layer 1
RUN apt-get update         # Layer 2
RUN apt-get install -y vim # Layer 3
COPY app.py /app/          # Layer 4
CMD ["python", "/app/app.py"] # Layer 5
```

### 5. ما الفرق بين CMD و ENTRYPOINT؟
**الإجابة:**

**CMD:**
- يحدد الأمر الافتراضي للحاوية
- يمكن استبداله عند تشغيل الحاوية
- يستخدم للأوامر الافتراضية القابلة للتغيير

**ENTRYPOINT:**
- يحدد الأمر الرئيسي الذي سيتم تنفيذه دائماً
- لا يمكن استبداله بسهولة (يحتاج --entrypoint)
- يستخدم عندما تريد الحاوية أن تعمل كأداة محددة

**مثال:**
```dockerfile
# استخدام CMD فقط
FROM ubuntu
CMD ["echo", "Hello World"]
# يمكن استبداله: docker run myimage echo "Different Message"

# استخدام ENTRYPOINT فقط
FROM ubuntu
ENTRYPOINT ["echo"]
# الاستخدام: docker run myimage "Hello World"

# استخدام كليهما معاً
FROM ubuntu
ENTRYPOINT ["echo"]
CMD ["Hello World"]
# الافتراضي: echo "Hello World"
# تخصيص: docker run myimage "Custom Message" => echo "Custom Message"
```

---

## أوامر Docker

### 6. ما هي أهم أوامر Docker الأساسية؟
**الإجابة:**

```bash
# === إدارة الصور ===
docker images                    # عرض جميع الصور
docker pull nginx:latest         # تحميل صورة
docker build -t myapp:1.0 .     # بناء صورة
docker rmi image_id             # حذف صورة
docker tag myapp:1.0 myapp:latest # إضافة tag للصورة

# === إدارة الحاويات ===
docker ps                       # عرض الحاويات العاملة
docker ps -a                    # عرض جميع الحاويات
docker run -d --name web nginx  # تشغيل حاوية
docker start container_id       # بدء حاوية متوقفة
docker stop container_id        # إيقاف حاوية
docker restart container_id     # إعادة تشغيل حاوية
docker rm container_id          # حذف حاوية

# === الوصول للحاويات ===
docker exec -it container_id bash  # الدخول لحاوية قيد التشغيل
docker logs container_id           # عرض السجلات
docker logs -f container_id        # متابعة السجلات

# === معلومات النظام ===
docker info                     # معلومات Docker
docker version                  # إصدار Docker
docker system df                # استخدام المساحة
docker system prune             # تنظيف الموارد غير المستخدمة
```

### 7. كيف تنسخ الملفات من وإلى الحاوية؟
**الإجابة:**

```bash
# نسخ ملف من الحاوية إلى المضيف
docker cp container_id:/path/in/container /path/on/host

# نسخ ملف من المضيف إلى الحاوية
docker cp /path/on/host container_id:/path/in/container

# مثال عملي: نسخ ملف تكوين
docker cp mycontainer:/etc/nginx/nginx.conf ./nginx.conf

# نسخ مجلد كامل
docker cp mycontainer:/var/www/html ./html_backup
```

### 8. كيف تحد من موارد الحاوية؟
**الإجابة:**

```bash
# تحديد الذاكرة والمعالج
docker run -d \
  --name myapp \
  --memory="512m" \          # حد أقصى 512 ميجابايت
  --memory-swap="1g" \       # ذاكرة + swap
  --cpus="1.5" \            # 1.5 نواة معالج
  --cpu-shares="1024" \     # الأولوية النسبية
  nginx

# مراقبة استخدام الموارد
docker stats myapp

# تحديث موارد حاوية قيد التشغيل
docker update --memory="1g" --cpus="2" myapp
```

### 9. كيف تعرض معلومات تفصيلية عن حاوية أو صورة؟
**الإجابة:**

```bash
# معلومات تفصيلية عن حاوية
docker inspect container_id

# معلومات محددة باستخدام --format
docker inspect --format='{{.NetworkSettings.IPAddress}}' container_id

# معلومات عن صورة
docker inspect nginx:latest

# عرض التاريخ والطبقات
docker history nginx:latest

# عرض العمليات داخل الحاوية
docker top container_id
```

---

## شبكات Docker

### 10. ما هي أنواع الشبكات في Docker؟
**الإجابة:**

**1. Bridge (الافتراضي):**
- شبكة خاصة داخلية
- تستخدم للحاويات على نفس المضيف
- عزل بين الحاويات

**2. Host:**
- الحاوية تشارك شبكة المضيف مباشرة
- أداء عالي
- لا يوجد عزل شبكي

**3. None:**
- لا توجد شبكة
- عزل كامل
- يستخدم لحاويات معزولة تماماً

**4. Overlay:**
- للشبكات متعددة المضيفات
- يستخدم مع Docker Swarm
- اتصال بين حاويات على أجهزة مختلفة

**مثال:**
```bash
# عرض الشبكات
docker network ls

# إنشاء شبكة bridge مخصصة
docker network create --driver bridge my_network

# تشغيل حاوية على شبكة محددة
docker run -d --name web --network my_network nginx

# ربط حاوية موجودة بشبكة
docker network connect my_network container_id

# فصل حاوية من شبكة
docker network disconnect my_network container_id
```

### 11. كيف تتواصل الحاويات مع بعضها؟
**الإجابة:**

```bash
# إنشاء شبكة مخصصة
docker network create app_network

# تشغيل قاعدة بيانات
docker run -d \
  --name database \
  --network app_network \
  -e MYSQL_ROOT_PASSWORD=secret \
  mysql:8.0

# تشغيل تطبيق يتصل بقاعدة البيانات
docker run -d \
  --name webapp \
  --network app_network \
  -e DB_HOST=database \
  -e DB_USER=root \
  -e DB_PASSWORD=secret \
  myapp:1.0

# الآن التطبيق يمكنه الاتصال بـ database باستخدام اسم الحاوية
```

**في التطبيق:**
```javascript
const mysql = require('mysql');
const connection = mysql.createConnection({
  host: 'database',  // اسم حاوية قاعدة البيانات
  user: 'root',
  password: 'secret',
  database: 'mydb'
});
```

### 12. ما هو Port Mapping؟
**الإجابة:**
Port Mapping هو ربط منفذ الحاوية بمنفذ على المضيف لجعل الخدمة متاحة من الخارج.

```bash
# صيغة: -p HOST_PORT:CONTAINER_PORT

# ربط منفذ 80 في الحاوية بـ 8080 على المضيف
docker run -d -p 8080:80 nginx
# الوصول: http://localhost:8080

# ربط منفذ على واجهة محددة
docker run -d -p 127.0.0.1:8080:80 nginx

# ربط جميع المنافذ المعرفة بـ EXPOSE
docker run -d -P nginx

# ربط عدة منافذ
docker run -d \
  -p 8080:80 \
  -p 8443:443 \
  nginx

# عرض المنافذ المربوطة
docker port container_id
```

---

## Docker Volumes

### 13. ما هي Docker Volumes ولماذا نستخدمها؟
**الإجابة:**
Volumes هي الطريقة المفضلة لتخزين البيانات الدائمة (Persistent Data) في Docker. البيانات في الحاوية تُفقد عند حذفها، لكن Volumes تبقى.

**فوائد Volumes:**
- استمرارية البيانات
- مشاركة البيانات بين الحاويات
- إدارة سهلة
- أداء أفضل من bind mounts
- نسخ احتياطي وترحيل

**مثال:**
```bash
# إنشاء volume
docker volume create my_data

# عرض جميع volumes
docker volume ls

# معلومات تفصيلية
docker volume inspect my_data

# استخدام volume مع حاوية
docker run -d \
  --name mysql_db \
  -v my_data:/var/lib/mysql \
  -e MYSQL_ROOT_PASSWORD=secret \
  mysql:8.0

# حذف volume
docker volume rm my_data

# حذف جميع volumes غير المستخدمة
docker volume prune
```

### 14. ما الفرق بين Volumes و Bind Mounts؟
**الإجابة:**

**Volumes:**
- تدار بواسطة Docker
- موقعها: `/var/lib/docker/volumes/`
- الطريقة المفضلة للبيانات الدائمة
- تعمل على جميع أنظمة التشغيل

**Bind Mounts:**
- تربط مجلد من المضيف مباشرة
- مسار كامل محدد
- مفيدة للتطوير
- تعتمد على هيكل المضيف

**مثال:**
```bash
# Volume (يدار بواسطة Docker)
docker run -d -v my_volume:/data nginx

# Bind Mount (مسار كامل)
docker run -d -v /home/user/data:/data nginx
docker run -d -v $(pwd)/data:/data nginx  # المجلد الحالي

# مثال للتطوير - ربط الكود
docker run -d \
  --name dev_app \
  -v $(pwd)/src:/app/src \
  -v $(pwd)/public:/app/public \
  -p 3000:3000 \
  node:18
```

### 15. كيف تشارك البيانات بين حاويات متعددة؟
**الإجابة:**

```bash
# إنشاء volume مشترك
docker volume create shared_data

# حاوية تكتب البيانات
docker run -d \
  --name writer \
  -v shared_data:/data \
  alpine sh -c "echo 'Hello from writer' > /data/message.txt && sleep infinity"

# حاوية تقرأ البيانات
docker run -d \
  --name reader \
  -v shared_data:/data:ro \  # read-only
  alpine sh -c "cat /data/message.txt && sleep infinity"

# التحقق
docker exec reader cat /data/message.txt

# مثال عملي: Nginx + Logs Processor
docker run -d \
  --name nginx \
  -v logs_volume:/var/log/nginx \
  -p 80:80 \
  nginx

docker run -d \
  --name log_analyzer \
  -v logs_volume:/logs:ro \
  log-analysis-tool
```

---

## Docker Compose

### 16. ما هو Docker Compose ومتى نستخدمه؟
**الإجابة:**
Docker Compose هو أداة لتعريف وتشغيل تطبيقات Docker متعددة الحاويات باستخدام ملف YAML واحد.

**متى نستخدمه:**
- تطبيقات متعددة الخدمات
- بيئات التطوير
- الاختبار الآلي
- نشر بسيط على مضيف واحد

**مثال - docker-compose.yml:**
```yaml
version: '3.8'

services:
  # قاعدة البيانات
  database:
    image: postgres:15
    environment:
      POSTGRES_DB: myapp
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: secret
    volumes:
      - db_data:/var/lib/postgresql/data
    networks:
      - backend

  # تطبيق الويب
  web:
    build: ./web
    ports:
      - "3000:3000"
    environment:
      DB_HOST: database
      DB_PORT: 5432
      DB_NAME: myapp
    depends_on:
      - database
    networks:
      - backend
      - frontend

  # Nginx Reverse Proxy
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
    depends_on:
      - web
    networks:
      - frontend

volumes:
  db_data:

networks:
  frontend:
  backend:
```

**الأوامر:**
```bash
# تشغيل جميع الخدمات
docker-compose up -d

# إيقاف جميع الخدمات
docker-compose down

# عرض الحالة
docker-compose ps

# عرض السجلات
docker-compose logs -f

# إعادة بناء وتشغيل
docker-compose up -d --build
```

### 17. كيف تتعامل مع المتغيرات البيئية في Docker Compose؟
**الإجابة:**

**1. ملف .env:**
```bash
# .env
DB_PASSWORD=secret123
APP_PORT=3000
NODE_ENV=production
```

**2. docker-compose.yml:**
```yaml
version: '3.8'

services:
  web:
    image: myapp:latest
    ports:
      - "${APP_PORT}:3000"
    environment:
      - NODE_ENV=${NODE_ENV}
      - DB_PASSWORD=${DB_PASSWORD}
    # أو استخدام env_file
    env_file:
      - .env
      - .env.production

  database:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD: ${DB_PASSWORD}
```

**3. تمرير متغيرات من الشل:**
```bash
# تعيين متغير
export DB_PASSWORD=mysecret

# تشغيل Compose
docker-compose up -d

# أو تمرير مباشرة
DB_PASSWORD=secret docker-compose up -d
```

### 18. كيف تستخدم depends_on بشكل صحيح؟
**الإجابة:**
`depends_on` يحدد ترتيب بدء الخدمات، لكنه لا ينتظر جاهزية الخدمة كاملاً.

**المشكلة:**
```yaml
services:
  web:
    image: myapp
    depends_on:
      - database  # ينتظر بدء الحاوية فقط، ليس جاهزية DB

  database:
    image: postgres:15
```

**الحل 1 - استخدام healthcheck:**
```yaml
services:
  database:
    image: postgres:15
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s
      timeout: 5s
      retries: 5

  web:
    image: myapp
    depends_on:
      database:
        condition: service_healthy
```

**الحل 2 - استخدام wait-for-it script:**
```dockerfile
# في Dockerfile
COPY wait-for-it.sh /wait-for-it.sh
RUN chmod +x /wait-for-it.sh

CMD ["/wait-for-it.sh", "database:5432", "--", "node", "server.js"]
```

```yaml
services:
  web:
    build: .
    command: /wait-for-it.sh database:5432 -- node server.js
    depends_on:
      - database
```

---

## أفضل ممارسات Dockerfile

### 19. ما هي أفضل الممارسات لكتابة Dockerfile فعال؟
**الإجابة:**

**1. استخدام صور أساسية صغيرة:**
```dockerfile
# ❌ سيء
FROM ubuntu:latest

# ✅ جيد
FROM node:18-alpine
FROM python:3.11-slim
```

**2. ترتيب الطبقات حسب تغيرها:**
```dockerfile
# ✅ الأفضل - الأقل تغيراً أولاً
FROM node:18-alpine

WORKDIR /app

# التبعيات أولاً (تتغير نادراً)
COPY package*.json ./
RUN npm ci --production

# الكود أخيراً (يتغير كثيراً)
COPY . .

CMD ["node", "server.js"]
```

**3. دمج أوامر RUN لتقليل الطبقات:**
```dockerfile
# ❌ سيء - طبقات كثيرة
RUN apt-get update
RUN apt-get install -y curl
RUN apt-get install -y vim
RUN apt-get clean

# ✅ جيد - طبقة واحدة
RUN apt-get update && \
    apt-get install -y curl vim && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
```

**4. استخدام .dockerignore:**
```
# .dockerignore
node_modules
npm-debug.log
.git
.env
*.md
.DS_Store
```

**5. تجنب تشغيل كـ root:**
```dockerfile
FROM node:18-alpine

WORKDIR /app

# إنشاء مستخدم غير root
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

COPY package*.json ./
RUN npm ci --production

COPY --chown=nodejs:nodejs . .

USER nodejs

CMD ["node", "server.js"]
```

### 20. كيف تصغر حجم صورة Docker؟
**الإجابة:**

**1. استخدام Alpine Linux:**
```dockerfile
# قبل: ~900MB
FROM node:18
WORKDIR /app
COPY . .
RUN npm install
CMD ["node", "server.js"]

# بعد: ~150MB
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --production
COPY . .
CMD ["node", "server.js"]
```

**2. Multi-stage Build (سنشرحها بالتفصيل):**
```dockerfile
# مرحلة البناء
FROM node:18 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# مرحلة الإنتاج
FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
CMD ["node", "dist/server.js"]
```

**3. تنظيف الملفات غير الضرورية:**
```dockerfile
RUN apt-get update && \
    apt-get install -y --no-install-recommends build-essential && \
    # ... بناء التطبيق ... && \
    apt-get purge -y build-essential && \
    apt-get autoremove -y && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*
```

**4. مقارنة الأحجام:**
```bash
# عرض حجم الصور
docker images

# تحليل طبقات الصورة
docker history myapp:latest

# استخدام dive لتحليل الصورة
dive myapp:latest
```

---

## Multi-stage Builds

### 21. ما هو Multi-stage Build ولماذا نستخدمه؟
**الإجابة:**
Multi-stage Build يسمح باستخدام عدة مراحل بناء في Dockerfile واحد، مما يقلل حجم الصورة النهائية بشكل كبير.

**الفوائد:**
- تقليل حجم الصورة النهائية
- فصل بيئة البناء عن بيئة الإنتاج
- أمان أفضل (عدم تضمين أدوات البناء)
- صيانة أسهل

**مثال - تطبيق Go:**
```dockerfile
# المرحلة 1: البناء
FROM golang:1.21 AS builder

WORKDIR /app

# نسخ ملفات Go modules
COPY go.mod go.sum ./
RUN go mod download

# نسخ الكود المصدري
COPY . .

# بناء التطبيق
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o main .

# المرحلة 2: الإنتاج
FROM alpine:latest

RUN apk --no-cache add ca-certificates

WORKDIR /root/

# نسخ الملف التنفيذي فقط من مرحلة البناء
COPY --from=builder /app/main .

EXPOSE 8080

CMD ["./main"]
```

**النتيجة:**
- صورة البناء: ~800MB
- الصورة النهائية: ~10MB

### 22. مثال Multi-stage Build لتطبيق React + Node.js
**الإجابة:**

```dockerfile
# المرحلة 1: بناء Frontend
FROM node:18 AS frontend-builder

WORKDIR /app/frontend

COPY frontend/package*.json ./
RUN npm ci

COPY frontend/ ./
RUN npm run build

# المرحلة 2: بناء Backend
FROM node:18 AS backend-builder

WORKDIR /app/backend

COPY backend/package*.json ./
RUN npm ci --production

COPY backend/ ./

# المرحلة 3: الإنتاج النهائي
FROM node:18-alpine

WORKDIR /app

# نسخ Backend
COPY --from=backend-builder /app/backend ./

# نسخ Frontend المبني
COPY --from=frontend-builder /app/frontend/build ./public

# إنشاء مستخدم غير root
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001 && \
    chown -R nodejs:nodejs /app

USER nodejs

EXPOSE 3000

CMD ["node", "server.js"]
```

### 23. كيف تستخدم مراحل البناء المسماة بشكل متقدم؟
**الإجابة:**

```dockerfile
# قاعدة مشتركة
FROM node:18-alpine AS base
WORKDIR /app
COPY package*.json ./

# مرحلة التبعيات
FROM base AS dependencies
RUN npm ci

# مرحلة التطوير
FROM dependencies AS development
COPY . .
EXPOSE 3000
CMD ["npm", "run", "dev"]

# مرحلة البناء
FROM dependencies AS build
COPY . .
RUN npm run build

# مرحلة الاختبار
FROM dependencies AS test
COPY . .
RUN npm run test

# مرحلة الإنتاج
FROM base AS production
COPY --from=dependencies /app/node_modules ./node_modules
COPY --from=build /app/dist ./dist
CMD ["node", "dist/server.js"]
```

**الاستخدام:**
```bash
# بناء للتطوير
docker build --target development -t myapp:dev .

# بناء للاختبار
docker build --target test -t myapp:test .

# بناء للإنتاج
docker build --target production -t myapp:prod .
```

---

## أمان Docker

### 24. ما هي أفضل ممارسات الأمان في Docker؟
**الإجابة:**

**1. عدم استخدام root:**
```dockerfile
FROM node:18-alpine

# إنشاء مستخدم
RUN addgroup -g 1001 appgroup && \
    adduser -D -u 1001 -G appgroup appuser

WORKDIR /app

COPY --chown=appuser:appgroup . .

USER appuser

CMD ["node", "server.js"]
```

**2. مسح الصور للثغرات الأمنية:**
```bash
# استخدام Docker Scout
docker scout cves myapp:latest

# استخدام Trivy
trivy image myapp:latest

# استخدام Snyk
snyk container test myapp:latest
```

**3. استخدام صور موثوقة:**
```dockerfile
# ✅ من المصادر الرسمية
FROM node:18-alpine

# ✅ التحقق من الصورة
FROM nginx:1.25@sha256:abc123...

# ❌ تجنب latest في الإنتاج
FROM ubuntu:latest
```

**4. تشغيل الحاويات بصلاحيات محدودة:**
```bash
# Read-only filesystem
docker run -d --read-only --tmpfs /tmp myapp

# إسقاط الصلاحيات
docker run -d \
  --cap-drop=ALL \
  --cap-add=NET_BIND_SERVICE \
  myapp

# استخدام security profiles
docker run -d \
  --security-opt=no-new-privileges \
  --security-opt apparmor=docker-default \
  myapp
```

**5. إدارة الأسرار بشكل آمن:**
```bash
# ❌ لا تضع الأسرار في Dockerfile
ENV DB_PASSWORD=secret123

# ✅ استخدام Docker secrets (في Swarm)
echo "mysecret" | docker secret create db_password -

# ✅ استخدام متغيرات بيئة من ملفات
docker run -d --env-file .env.production myapp

# ✅ استخدام HashiCorp Vault أو AWS Secrets Manager
```

### 25. كيف تحد من موارد الحاوية لمنع DoS؟
**الإجابة:**

```bash
# تحديد الموارد
docker run -d \
  --name secure_app \
  --memory="512m" \
  --memory-reservation="256m" \
  --memory-swap="1g" \
  --cpus="1.0" \
  --pids-limit=200 \
  --ulimit nofile=1024:2048 \
  myapp

# في Docker Compose
services:
  web:
    image: myapp
    deploy:
      resources:
        limits:
          cpus: '1.0'
          memory: 512M
        reservations:
          cpus: '0.5'
          memory: 256M
    ulimits:
      nofile:
        soft: 1024
        hard: 2048
```

**مراقبة الموارد:**
```bash
# مراقبة مباشرة
docker stats

# تنبيهات عند تجاوز الحدود
docker events --filter 'event=oom'
```

### 26. كيف تؤمن شبكة Docker؟
**الإجابة:**

```bash
# 1. استخدام شبكات مخصصة (عزل أفضل)
docker network create \
  --driver bridge \
  --subnet=172.20.0.0/16 \
  --opt encrypted \
  secure_network

# 2. تفعيل التشفير في Overlay networks
docker network create \
  --driver overlay \
  --opt encrypted \
  swarm_secure_network

# 3. عزل الخدمات
docker-compose.yml:
```

```yaml
version: '3.8'

services:
  frontend:
    image: frontend:latest
    networks:
      - frontend_net

  backend:
    image: backend:latest
    networks:
      - frontend_net
      - backend_net

  database:
    image: postgres:15
    networks:
      - backend_net  # لا يمكن الوصول من frontend مباشرة

networks:
  frontend_net:
    driver: bridge
  backend_net:
    driver: bridge
    internal: true  # لا وصول للإنترنت
```

**4. استخدام Firewall:**
```bash
# تحديد الوصول بـ iptables
iptables -A DOCKER-USER -i ext_if ! -s 192.168.1.0/24 -j DROP

# السماح فقط بـ IPs محددة
docker run -d \
  --add-host=allowed-host:192.168.1.100 \
  myapp
```

---

## استكشاف الأخطاء وإصلاحها

### 27. كيف تحل مشكلة حاوية تتوقف فوراً بعد التشغيل؟
**الإجابة:**

**التشخيص:**
```bash
# 1. عرض السجلات
docker logs container_id

# 2. عرض حالة الخروج
docker ps -a
# انظر لـ STATUS: Exited (1) أو Exited (0)

# 3. فحص الحاوية
docker inspect container_id | grep -A 10 "State"
```

**الأسباب الشائعة:**

**1. الأمر الرئيسي ينتهي:**
```dockerfile
# ❌ المشكلة
FROM ubuntu
RUN echo "Hello"
# الحاوية تتوقف لأن ليس هناك عملية مستمرة

# ✅ الحل
FROM ubuntu
CMD ["tail", "-f", "/dev/null"]
# أو
CMD ["sleep", "infinity"]
```

**2. خطأ في التطبيق:**
```bash
# تشغيل تفاعلي لرؤية الخطأ
docker run -it myapp /bin/sh

# أو تجاوز CMD
docker run -it myapp sh
```

**3. متغيرات بيئية مفقودة:**
```bash
# التحقق من المتغيرات
docker exec container_id env

# تشغيل مع متغيرات صحيحة
docker run -d \
  -e DB_HOST=localhost \
  -e DB_PORT=5432 \
  myapp
```

### 28. كيف تحل مشاكل الشبكة في Docker؟
**الإجابة:**

**1. حاوية لا يمكن الوصول إليها:**
```bash
# التحقق من IP الحاوية
docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' container_id

# التحقق من المنفذ
docker port container_id

# اختبار الاتصال
docker exec container_id ping -c 3 8.8.8.8

# اختبار DNS
docker exec container_id nslookup google.com
```

**2. حاويات لا تتواصل:**
```bash
# التحقق من الشبكات
docker network ls
docker network inspect network_name

# اختبار الاتصال بين حاويات
docker exec container1 ping container2

# التحقق من firewall
sudo iptables -L DOCKER-USER
```

**3. تضارب المنافذ:**
```bash
# ❌ خطأ: Port already allocated
docker run -d -p 80:80 nginx

# ✅ الحل: استخدام منفذ آخر
docker run -d -p 8080:80 nginx

# التحقق من المنافذ المستخدمة
netstat -tuln | grep 80
# أو
lsof -i :80
```

**4. مشاكل DNS:**
```dockerfile
# في Dockerfile
RUN echo "nameserver 8.8.8.8" > /etc/resolv.conf
```

```bash
# عند التشغيل
docker run -d \
  --dns 8.8.8.8 \
  --dns 8.8.4.4 \
  myapp
```

### 29. كيف تحل مشاكل الأداء في Docker؟
**الإجابة:**

**1. تشخيص الأداء:**
```bash
# مراقبة الموارد
docker stats

# معلومات تفصيلية
docker stats --no-stream --format "table {{.Container}}\t{{.CPUPerc}}\t{{.MemUsage}}"

# أحداث النظام
docker events

# معلومات النظام
docker system df
docker system df -v
```

**2. بطء بناء الصورة:**
```dockerfile
# ✅ استخدام cache بشكل فعال
FROM node:18-alpine

WORKDIR /app

# الملفات الأقل تغيراً أولاً
COPY package*.json ./
RUN npm ci

# الملفات المتغيرة أخيراً
COPY . .

# استخدام buildkit
# export DOCKER_BUILDKIT=1
```

**3. بطء تشغيل الحاوية:**
```bash
# تحليل الصورة
docker history myapp:latest

# تقليل الطبقات
# استخدام multi-stage build
# حذف الملفات غير الضرورية

# استخدام volumes بدل bind mounts للأداء
docker run -d -v myvolume:/data myapp
# أسرع من
docker run -d -v /host/path:/data myapp
```

**4. مشاكل I/O:**
```bash
# استخدام storage driver مناسب
docker info | grep "Storage Driver"

# تحسين overlay2
# في /etc/docker/daemon.json
{
  "storage-driver": "overlay2",
  "storage-opts": [
    "overlay2.override_kernel_check=true"
  ]
}
```

### 30. كيف تصلح صورة Docker تالفة؟
**الإجابة:**

**1. إعادة بناء من الصفر:**
```bash
# حذف cache وإعادة البناء
docker build --no-cache -t myapp:latest .

# حذف الصورة التالفة
docker rmi -f image_id
```

**2. استرجاع من layer محدد:**
```bash
# عرض الطبقات
docker history myapp:latest

# إنشاء صورة من layer محدد
docker commit sha256:layer_id myapp:recovered
```

**3. تنظيف النظام:**
```bash
# حذف كل شيء غير مستخدم
docker system prune -a

# حذف volumes غير مستخدمة
docker volume prune

# حذف شبكات غير مستخدمة
docker network prune

# إعادة تشغيل Docker daemon
sudo systemctl restart docker
```

---

## سيناريوهات واقعية

### 31. سيناريو: نشر تطبيق Full-Stack مع قاعدة بيانات
**الإجابة:**

**البنية:**
- Frontend: React
- Backend: Node.js API
- Database: PostgreSQL
- Reverse Proxy: Nginx

**docker-compose.yml:**
```yaml
version: '3.8'

services:
  # قاعدة البيانات
  postgres:
    image: postgres:15-alpine
    container_name: app_db
    environment:
      POSTGRES_DB: myapp
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql
    networks:
      - backend
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U admin"]
      interval: 10s
      timeout: 5s
      retries: 5

  # Backend API
  api:
    build:
      context: ./backend
      dockerfile: Dockerfile
    container_name: app_api
    environment:
      NODE_ENV: production
      DATABASE_URL: postgresql://admin:${DB_PASSWORD}@postgres:5432/myapp
      JWT_SECRET: ${JWT_SECRET}
    depends_on:
      postgres:
        condition: service_healthy
    networks:
      - backend
      - frontend
    restart: unless-stopped

  # Frontend
  web:
    build:
      context: ./frontend
      dockerfile: Dockerfile
    container_name: app_web
    environment:
      REACT_APP_API_URL: http://api:3000
    depends_on:
      - api
    networks:
      - frontend

  # Nginx Reverse Proxy
  nginx:
    image: nginx:alpine
    container_name: app_nginx
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./nginx/ssl:/etc/nginx/ssl:ro
    depends_on:
      - web
      - api
    networks:
      - frontend
    restart: unless-stopped

volumes:
  postgres_data:

networks:
  frontend:
  backend:
```

**Backend Dockerfile:**
```dockerfile
FROM node:18-alpine AS builder

WORKDIR /app

COPY package*.json ./
RUN npm ci

COPY . .
RUN npm run build

FROM node:18-alpine

WORKDIR /app

RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

COPY --from=builder --chown=nodejs:nodejs /app/dist ./dist
COPY --from=builder --chown=nodejs:nodejs /app/node_modules ./node_modules
COPY --chown=nodejs:nodejs package*.json ./

USER nodejs

EXPOSE 3000

CMD ["node", "dist/server.js"]
```

**Frontend Dockerfile:**
```dockerfile
FROM node:18-alpine AS builder

WORKDIR /app

COPY package*.json ./
RUN npm ci

COPY . .
RUN npm run build

FROM nginx:alpine

COPY --from=builder /app/build /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

**Nginx Configuration:**
```nginx
upstream api_backend {
    server api:3000;
}

server {
    listen 80;
    server_name example.com;

    # Frontend
    location / {
        root /usr/share/nginx/html;
        try_files $uri $uri/ /index.html;
    }

    # API Proxy
    location /api {
        proxy_pass http://api_backend;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

### 32. سيناريو: إعداد بيئة Microservices
**الإجابة:**

**البنية:**
- User Service (Python Flask)
- Order Service (Node.js)
- Product Service (Go)
- Message Queue (RabbitMQ)
- API Gateway (Kong)
- Service Discovery (Consul)

**docker-compose.yml:**
```yaml
version: '3.8'

services:
  # Message Queue
  rabbitmq:
    image: rabbitmq:3-management-alpine
    ports:
      - "5672:5672"
      - "15672:15672"
    environment:
      RABBITMQ_DEFAULT_USER: admin
      RABBITMQ_DEFAULT_PASS: ${RABBITMQ_PASSWORD}
    networks:
      - microservices
    healthcheck:
      test: rabbitmq-diagnostics -q ping
      interval: 30s
      timeout: 10s
      retries: 5

  # Service Discovery
  consul:
    image: consul:latest
    ports:
      - "8500:8500"
    networks:
      - microservices
    command: agent -server -ui -bootstrap-expect=1 -client=0.0.0.0

  # User Service
  user-service:
    build: ./services/user-service
    environment:
      RABBITMQ_URL: amqp://admin:${RABBITMQ_PASSWORD}@rabbitmq:5672
      CONSUL_HOST: consul
      SERVICE_PORT: 5000
    depends_on:
      - rabbitmq
      - consul
    networks:
      - microservices
    deploy:
      replicas: 2
      restart_policy:
        condition: on-failure

  # Order Service
  order-service:
    build: ./services/order-service
    environment:
      RABBITMQ_URL: amqp://admin:${RABBITMQ_PASSWORD}@rabbitmq:5672
      CONSUL_HOST: consul
      SERVICE_PORT: 3000
    depends_on:
      - rabbitmq
      - consul
    networks:
      - microservices
    deploy:
      replicas: 2

  # Product Service
  product-service:
    build: ./services/product-service
    environment:
      RABBITMQ_URL: amqp://admin:${RABBITMQ_PASSWORD}@rabbitmq:5672
      CONSUL_HOST: consul
      SERVICE_PORT: 8080
    depends_on:
      - rabbitmq
      - consul
    networks:
      - microservices
    deploy:
      replicas: 2

  # API Gateway
  kong:
    image: kong:latest
    environment:
      KONG_DATABASE: "off"
      KONG_DECLARATIVE_CONFIG: /kong/kong.yml
      KONG_PROXY_ACCESS_LOG: /dev/stdout
      KONG_ADMIN_ACCESS_LOG: /dev/stdout
      KONG_PROXY_ERROR_LOG: /dev/stderr
      KONG_ADMIN_ERROR_LOG: /dev/stderr
    ports:
      - "8000:8000"
      - "8443:8443"
      - "8001:8001"
    volumes:
      - ./kong.yml:/kong/kong.yml
    networks:
      - microservices

networks:
  microservices:
    driver: bridge
```

**User Service Dockerfile (Python):**
```dockerfile
FROM python:3.11-slim AS builder

WORKDIR /app

COPY requirements.txt .
RUN pip install --user --no-cache-dir -r requirements.txt

FROM python:3.11-slim

WORKDIR /app

RUN useradd -m -u 1001 appuser

COPY --from=builder --chown=appuser:appuser /root/.local /home/appuser/.local
COPY --chown=appuser:appuser . .

USER appuser

ENV PATH=/home/appuser/.local/bin:$PATH

EXPOSE 5000

CMD ["python", "app.py"]
```

### 33. سيناريو: CI/CD Pipeline مع Docker
**الإجابة:**

**.gitlab-ci.yml:**
```yaml
stages:
  - test
  - build
  - deploy

variables:
  DOCKER_DRIVER: overlay2
  DOCKER_TLS_CERTDIR: "/certs"
  IMAGE_TAG: $CI_REGISTRY_IMAGE:$CI_COMMIT_SHORT_SHA

# المرحلة 1: الاختبار
test:
  stage: test
  image: node:18
  script:
    - npm ci
    - npm run lint
    - npm run test
    - npm run test:coverage
  coverage: '/Statements\s*:\s*(\d+\.?\d*)%/'
  artifacts:
    reports:
      coverage_report:
        coverage_format: cobertura
        path: coverage/cobertura-coverage.xml

# المرحلة 2: بناء الصورة
build:
  stage: build
  image: docker:latest
  services:
    - docker:dind
  before_script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
  script:
    # بناء الصورة
    - docker build -t $IMAGE_TAG .
    - docker tag $IMAGE_TAG $CI_REGISTRY_IMAGE:latest

    # فحص الثغرات الأمنية
    - docker run --rm -v /var/run/docker.sock:/var/run/docker.sock
        aquasec/trivy image --severity HIGH,CRITICAL $IMAGE_TAG

    # رفع الصورة
    - docker push $IMAGE_TAG
    - docker push $CI_REGISTRY_IMAGE:latest
  only:
    - main
    - develop

# المرحلة 3: النشر
deploy_staging:
  stage: deploy
  image: alpine:latest
  before_script:
    - apk add --no-cache curl
    - curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
    - chmod +x kubectl
    - mv kubectl /usr/local/bin/
  script:
    - kubectl config use-context staging
    - kubectl set image deployment/myapp myapp=$IMAGE_TAG
    - kubectl rollout status deployment/myapp
  environment:
    name: staging
    url: https://staging.example.com
  only:
    - develop

deploy_production:
  stage: deploy
  image: alpine:latest
  before_script:
    - apk add --no-cache curl
    - curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
    - chmod +x kubectl
    - mv kubectl /usr/local/bin/
  script:
    - kubectl config use-context production
    - kubectl set image deployment/myapp myapp=$IMAGE_TAG
    - kubectl rollout status deployment/myapp
  environment:
    name: production
    url: https://example.com
  when: manual
  only:
    - main
```

**Dockerfile محسن للـ CI/CD:**
```dockerfile
# مرحلة التبعيات
FROM node:18-alpine AS dependencies
WORKDIR /app
COPY package*.json ./
RUN npm ci --production

# مرحلة البناء
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# مرحلة الاختبار (اختيارية في CI)
FROM builder AS test
RUN npm run test

# مرحلة الإنتاج
FROM node:18-alpine AS production

WORKDIR /app

RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

COPY --from=dependencies --chown=nodejs:nodejs /app/node_modules ./node_modules
COPY --from=builder --chown=nodejs:nodejs /app/dist ./dist
COPY --chown=nodejs:nodejs package*.json ./

USER nodejs

EXPOSE 3000

HEALTHCHECK --interval=30s --timeout=3s --start-period=40s --retries=3 \
  CMD node healthcheck.js

CMD ["node", "dist/server.js"]
```

### 34. سيناريو: Monitoring و Logging
**الإجابة:**

**docker-compose.monitoring.yml:**
```yaml
version: '3.8'

services:
  # Prometheus - Metrics Collection
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus/prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus_data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
    networks:
      - monitoring

  # Grafana - Visualization
  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    ports:
      - "3000:3000"
    environment:
      GF_SECURITY_ADMIN_USER: admin
      GF_SECURITY_ADMIN_PASSWORD: ${GRAFANA_PASSWORD}
    volumes:
      - grafana_data:/var/lib/grafana
      - ./grafana/dashboards:/etc/grafana/provisioning/dashboards
      - ./grafana/datasources:/etc/grafana/provisioning/datasources
    networks:
      - monitoring

  # Elasticsearch - Log Storage
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.11.0
    container_name: elasticsearch
    environment:
      - discovery.type=single-node
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      - xpack.security.enabled=false
    volumes:
      - elasticsearch_data:/usr/share/elasticsearch/data
    ports:
      - "9200:9200"
    networks:
      - monitoring

  # Logstash - Log Processing
  logstash:
    image: docker.elastic.co/logstash/logstash:8.11.0
    container_name: logstash
    volumes:
      - ./logstash/logstash.conf:/usr/share/logstash/pipeline/logstash.conf
    ports:
      - "5000:5000"
    depends_on:
      - elasticsearch
    networks:
      - monitoring

  # Kibana - Log Visualization
  kibana:
    image: docker.elastic.co/kibana/kibana:8.11.0
    container_name: kibana
    ports:
      - "5601:5601"
    environment:
      ELASTICSEARCH_HOSTS: http://elasticsearch:9200
    depends_on:
      - elasticsearch
    networks:
      - monitoring

  # cAdvisor - Container Metrics
  cadvisor:
    image: gcr.io/cadvisor/cadvisor:latest
    container_name: cadvisor
    ports:
      - "8080:8080"
    volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:ro
      - /sys:/sys:ro
      - /var/lib/docker/:/var/lib/docker:ro
    networks:
      - monitoring

  # Node Exporter - Host Metrics
  node-exporter:
    image: prom/node-exporter:latest
    container_name: node-exporter
    ports:
      - "9100:9100"
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /:/rootfs:ro
    command:
      - '--path.procfs=/host/proc'
      - '--path.sysfs=/host/sys'
      - '--collector.filesystem.mount-points-exclude=^/(sys|proc|dev|host|etc)($$|/)'
    networks:
      - monitoring

volumes:
  prometheus_data:
  grafana_data:
  elasticsearch_data:

networks:
  monitoring:
    driver: bridge
```

**prometheus.yml:**
```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'cadvisor'
    static_configs:
      - targets: ['cadvisor:8080']

  - job_name: 'node-exporter'
    static_configs:
      - targets: ['node-exporter:9100']

  - job_name: 'docker'
    static_configs:
      - targets: ['172.17.0.1:9323']
```

**إضافة logging للتطبيق:**
```yaml
# في docker-compose.yml الرئيسي
services:
  myapp:
    image: myapp:latest
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
        labels: "app,environment"
        env: "ENV,APP_VERSION"
    # أو استخدام Fluentd
    logging:
      driver: fluentd
      options:
        fluentd-address: localhost:24224
        tag: myapp.logs
```

### 35. سيناريو: Backup و Recovery
**الإجابة:**

**استراتيجية النسخ الاحتياطي:**

**1. نسخ احتياطي لـ Volumes:**
```bash
#!/bin/bash
# backup-volumes.sh

BACKUP_DIR="/backup/docker-volumes"
DATE=$(date +%Y%m%d_%H%M%S)

# إنشاء مجلد النسخ الاحتياطي
mkdir -p $BACKUP_DIR

# نسخ احتياطي لكل volume
for volume in $(docker volume ls -q); do
    echo "Backing up volume: $volume"

    docker run --rm \
        -v $volume:/data \
        -v $BACKUP_DIR:/backup \
        alpine tar czf /backup/${volume}_${DATE}.tar.gz -C /data .
done

# حذف النسخ القديمة (أكثر من 30 يوم)
find $BACKUP_DIR -name "*.tar.gz" -mtime +30 -delete

echo "Backup completed: $DATE"
```

**2. نسخ احتياطي لقاعدة البيانات:**
```bash
#!/bin/bash
# backup-database.sh

BACKUP_DIR="/backup/database"
DATE=$(date +%Y%m%d_%H%M%S)
CONTAINER_NAME="postgres_db"

mkdir -p $BACKUP_DIR

# PostgreSQL Backup
docker exec $CONTAINER_NAME pg_dump -U postgres mydb > \
    $BACKUP_DIR/mydb_${DATE}.sql

# ضغط النسخة الاحتياطية
gzip $BACKUP_DIR/mydb_${DATE}.sql

# رفع للـ Cloud (S3)
aws s3 cp $BACKUP_DIR/mydb_${DATE}.sql.gz \
    s3://my-backups/database/

echo "Database backup completed"
```

**3. استرجاع Volume:**
```bash
#!/bin/bash
# restore-volume.sh

VOLUME_NAME=$1
BACKUP_FILE=$2

if [ -z "$VOLUME_NAME" ] || [ -z "$BACKUP_FILE" ]; then
    echo "Usage: $0 <volume_name> <backup_file>"
    exit 1
fi

# إنشاء volume جديد
docker volume create $VOLUME_NAME

# استرجاع البيانات
docker run --rm \
    -v $VOLUME_NAME:/data \
    -v $(dirname $BACKUP_FILE):/backup \
    alpine tar xzf /backup/$(basename $BACKUP_FILE) -C /data

echo "Volume $VOLUME_NAME restored from $BACKUP_FILE"
```

**4. استرجاع قاعدة البيانات:**
```bash
#!/bin/bash
# restore-database.sh

BACKUP_FILE=$1
CONTAINER_NAME="postgres_db"

if [ -z "$BACKUP_FILE" ]; then
    echo "Usage: $0 <backup_file>"
    exit 1
fi

# فك الضغط إذا كان مضغوطاً
if [[ $BACKUP_FILE == *.gz ]]; then
    gunzip -c $BACKUP_FILE > /tmp/restore.sql
    RESTORE_FILE="/tmp/restore.sql"
else
    RESTORE_FILE=$BACKUP_FILE
fi

# استرجاع البيانات
docker exec -i $CONTAINER_NAME psql -U postgres mydb < $RESTORE_FILE

rm -f /tmp/restore.sql

echo "Database restored from $BACKUP_FILE"
```

**5. جدولة النسخ الاحتياطي (Cron):**
```bash
# crontab -e

# نسخ احتياطي يومي للـ volumes الساعة 2 صباحاً
0 2 * * * /opt/scripts/backup-volumes.sh >> /var/log/docker-backup.log 2>&1

# نسخ احتياطي لقاعدة البيانات كل 6 ساعات
0 */6 * * * /opt/scripts/backup-database.sh >> /var/log/db-backup.log 2>&1
```

**docker-compose مع backup:**
```yaml
version: '3.8'

services:
  app:
    image: myapp:latest
    volumes:
      - app_data:/data

  backup:
    image: alpine:latest
    volumes:
      - app_data:/source:ro
      - ./backups:/backup
    command: >
      sh -c "
        while true; do
          tar czf /backup/app_data_$$(date +%Y%m%d_%H%M%S).tar.gz -C /source .
          find /backup -name '*.tar.gz' -mtime +7 -delete
          sleep 86400
        done
      "

volumes:
  app_data:
```

---

## أسئلة متقدمة إضافية

### 36. كيف تدير Secrets بشكل آمن في Docker؟
**الإجابة:**

**1. Docker Secrets (Swarm Mode):**
```bash
# إنشاء secret من ملف
docker secret create db_password ./password.txt

# إنشاء secret من stdin
echo "mysecretpassword" | docker secret create db_password -

# عرض secrets
docker secret ls

# استخدام في service
docker service create \
    --name myapp \
    --secret db_password \
    myapp:latest
```

**في التطبيق:**
```javascript
const fs = require('fs');
// Secret متاح في /run/secrets/
const password = fs.readFileSync('/run/secrets/db_password', 'utf8');
```

**2. مع Docker Compose (Swarm):**
```yaml
version: '3.8'

services:
  app:
    image: myapp:latest
    secrets:
      - db_password
      - api_key

secrets:
  db_password:
    file: ./secrets/db_password.txt
  api_key:
    external: true
```

**3. استخدام Environment Variables بأمان:**
```bash
# ملف .env (لا ترفعه لـ Git!)
DB_PASSWORD=secret123
API_KEY=xyz789

# .gitignore
.env
secrets/
```

**4. استخدام HashiCorp Vault:**
```yaml
services:
  vault:
    image: vault:latest
    cap_add:
      - IPC_LOCK
    environment:
      VAULT_DEV_ROOT_TOKEN_ID: myroot
    ports:
      - "8200:8200"

  app:
    image: myapp:latest
    environment:
      VAULT_ADDR: http://vault:8200
      VAULT_TOKEN: myroot
```

### 37. كيف تنفذ Health Checks فعالة؟
**الإجابة:**

**1. في Dockerfile:**
```dockerfile
FROM node:18-alpine

WORKDIR /app

COPY package*.json ./
RUN npm ci --production

COPY . .

# Simple HTTP healthcheck
HEALTHCHECK --interval=30s --timeout=3s --start-period=40s --retries=3 \
  CMD wget --no-verbose --tries=1 --spider http://localhost:3000/health || exit 1

# أو باستخدام curl
HEALTHCHECK --interval=30s --timeout=3s \
  CMD curl -f http://localhost:3000/health || exit 1

EXPOSE 3000
CMD ["node", "server.js"]
```

**2. Health Check Endpoint:**
```javascript
// server.js
const express = require('express');
const app = express();

app.get('/health', async (req, res) => {
  // فحص الاتصال بقاعدة البيانات
  try {
    await db.ping();
    res.status(200).json({
      status: 'healthy',
      uptime: process.uptime(),
      timestamp: Date.now()
    });
  } catch (error) {
    res.status(503).json({
      status: 'unhealthy',
      error: error.message
    });
  }
});

app.listen(3000);
```

**3. في Docker Compose:**
```yaml
version: '3.8'

services:
  web:
    image: myapp:latest
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s

  database:
    image: postgres:15
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5

  api:
    image: api:latest
    depends_on:
      database:
        condition: service_healthy
```

**4. مراقبة Health Status:**
```bash
# عرض حالة الصحة
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"

# فحص صحة حاوية محددة
docker inspect --format='{{.State.Health.Status}}' container_name

# عرض سجل فحوصات الصحة
docker inspect --format='{{json .State.Health}}' container_name | jq
```

### 38. كيف تحسن وقت بناء الصورة (Build Time)?
**الإجابة:**

**1. استخدام BuildKit:**
```bash
# تفعيل BuildKit
export DOCKER_BUILDKIT=1

# أو بشكل دائم في /etc/docker/daemon.json
{
  "features": {
    "buildkit": true
  }
}
```

**2. Cache Optimization:**
```dockerfile
# ❌ سيء - يعيد بناء كل شيء عند تغيير أي ملف
FROM node:18-alpine
WORKDIR /app
COPY . .
RUN npm install

# ✅ جيد - يستخدم cache للتبعيات
FROM node:18-alpine
WORKDIR /app

# نسخ ملفات package أولاً (نادراً ما تتغير)
COPY package*.json ./
RUN npm ci --production

# نسخ الكود (يتغير كثيراً)
COPY . .
```

**3. Parallel Builds:**
```dockerfile
# باستخدام BuildKit
FROM node:18 AS base
WORKDIR /app

FROM base AS dependencies
COPY package*.json ./
RUN npm ci

FROM base AS build
COPY --from=dependencies /app/node_modules ./node_modules
COPY . .
RUN npm run build

FROM base AS test
COPY --from=dependencies /app/node_modules ./node_modules
COPY . .
RUN npm test

# المراحل dependencies و test تُبنى بالتوازي
```

**4. استخدام Cache Mounts:**
```dockerfile
# syntax=docker/dockerfile:1.4

FROM node:18-alpine

WORKDIR /app

# استخدام cache mount لـ npm
RUN --mount=type=cache,target=/root/.npm \
    npm install -g npm@latest

COPY package*.json ./

RUN --mount=type=cache,target=/root/.npm \
    npm ci --production

COPY . .
```

**5. External Cache:**
```bash
# بناء مع تصدير cache
docker buildx build \
  --cache-to type=local,dest=/tmp/cache \
  --cache-from type=local,src=/tmp/cache \
  -t myapp:latest .

# استخدام registry كـ cache
docker buildx build \
  --cache-to type=registry,ref=myregistry/myapp:buildcache \
  --cache-from type=registry,ref=myregistry/myapp:buildcache \
  -t myapp:latest .
```

### 39. ما الفرق بين Docker و Podman؟
**الإجابة:**

| الميزة | Docker | Podman |
|--------|--------|--------|
| **Architecture** | Client-Server (daemon) | Daemonless (fork-exec) |
| **Root** | يحتاج root للـ daemon | Rootless by default |
| **أمان** | يعمل كـ root | أكثر أماناً (rootless) |
| **Docker Compose** | مدعوم أصلياً | يستخدم podman-compose |
| **Kubernetes** | يحتاج أدوات إضافية | يولد YAML مباشرة |
| **Systemd** | لا يدعم مباشرة | مدعوم أصلياً |

**استخدام Podman:**
```bash
# نفس أوامر Docker
alias docker=podman

# تشغيل rootless
podman run -d --name nginx nginx:alpine

# توليد Kubernetes YAML
podman generate kube nginx > nginx-pod.yaml

# تشغيل مع systemd
podman generate systemd nginx > nginx.service
sudo mv nginx.service /etc/systemd/system/
sudo systemctl enable nginx.service
```

### 40. كيف تتعامل مع Container Orchestration؟
**الإجابة:**

**Docker Swarm:**
```bash
# تهيئة Swarm
docker swarm init

# إضافة worker node
docker swarm join --token TOKEN MANAGER-IP:2377

# نشر stack
docker stack deploy -c docker-compose.yml myapp

# توسيع service
docker service scale myapp_web=5

# تحديث service (zero-downtime)
docker service update --image myapp:v2 myapp_web
```

**docker-compose لـ Swarm:**
```yaml
version: '3.8'

services:
  web:
    image: myapp:latest
    deploy:
      replicas: 3
      update_config:
        parallelism: 1
        delay: 10s
        order: start-first
      restart_policy:
        condition: on-failure
        delay: 5s
        max_attempts: 3
      resources:
        limits:
          cpus: '0.5'
          memory: 512M
        reservations:
          cpus: '0.25'
          memory: 256M
      placement:
        constraints:
          - node.role == worker
          - node.labels.type == web
    networks:
      - webnet
    ports:
      - "80:80"

networks:
  webnet:
    driver: overlay
```

---

## نصائح نهائية للمقابلة

### نصائح عامة:
1. **فهم المفاهيم الأساسية جيداً** (Container vs Image, Volumes, Networks)
2. **تدرب على الأوامر العملية** - لا تحفظ فقط، طبق
3. **اشرح التفكير** - عند حل مشكلة، اشرح خطوات التفكير
4. **اذكر Best Practices** - الأمان، الأداء، الصيانة
5. **استعد لأسئلة Troubleshooting** - كيف تحل المشاكل الشائعة
6. **اربط بالواقع** - اذكر تجاربك العملية

### أسئلة قد يطرحها المقابل:
- لماذا تستخدم Docker في مشاريعك؟
- ما هي التحديات التي واجهتها مع Docker؟
- كيف تضمن أمان containers في الإنتاج؟
- ما الفرق بين Docker و VM؟
- كيف تتعامل مع Docker في CI/CD؟
- ما هي تجربتك مع Kubernetes أو Swarm؟

### موارد مفيدة:
- Docker Documentation: https://docs.docker.com
- Docker Hub: https://hub.docker.com
- Play with Docker: https://labs.play-with-docker.com
- Docker Best Practices: https://docs.docker.com/develop/dev-best-practices

---

**بالتوفيق في مقابلتك! 🚀**
