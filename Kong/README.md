# Kong Gateway - دليل شامل

## المحتويات
1. [ما هو Kong](#ما-هو-kong)
2. [المفاهيم الأساسية](#المفاهيم-الأساسية)
3. [المكونات الرئيسية](#المكونات-الرئيسية)
4. [الميزات الرئيسية](#الميزات-الرئيسية)
5. [Plugins](#plugins)
6. [طرق النشر](#طرق-النشر)
7. [حالات الاستخدام](#حالات-الاستخدام)
8. [أمثلة عملية](#أمثلة-عملية)
9. [Kong vs Apigee](#kong-vs-apigee)
10. [أسئلة المقابلات](#أسئلة-المقابلات)

---

## ما هو Kong

**Kong Gateway** هو API Gateway مفتوح المصدر مبني على NGINX وOpenResty (Lua). يعمل كطبقة وسيطة بين العملاء والـ Backend Services لإدارة، تأمين، ومراقبة APIs.

### المزايا الرئيسية
- **Open Source**: مفتوح المصدر مع مجتمع نشط
- **High Performance**: مبني على NGINX، سريع جداً
- **Plugin Architecture**: نظام Plugins قابل للتوسع
- **Scalable**: يدعم Horizontal Scaling
- **Database Options**: دعم PostgreSQL، Cassandra، وDB-less mode
- **Cloud-Native**: متوافق مع Kubernetes وDocker

### الإصدارات
- **Kong Gateway OSS**: مجاني ومفتوح المصدر
- **Kong Gateway Enterprise**: نسخة مدفوعة مع ميزات إضافية
- **Kong Konnect**: SaaS Platform مُدار بالكامل

---

## المفاهيم الأساسية

### 1. Service
يمثل Backend API أو Microservice. يحتوي على:
- **Name**: اسم الخدمة
- **URL**: عنوان Backend Service
- **Protocol**: http, https, tcp, tls, grpc

```json
{
  "name": "user-service",
  "url": "http://backend.example.com:8080/users"
}
```

### 2. Route
مسار يربط الطلبات الواردة بـ Service معين. يمكن تحديد:
- **Paths**: المسارات (`/api/users`)
- **Methods**: HTTP Methods (GET, POST, etc.)
- **Hosts**: أسماء النطاقات
- **Headers**: Headers محددة

```json
{
  "name": "user-route",
  "paths": ["/api/users"],
  "methods": ["GET", "POST"],
  "service": {"id": "service-id"}
}
```

### 3. Consumer
يمثل المستخدم أو التطبيق الذي يستهلك APIs. يُستخدم مع:
- Authentication Plugins
- Rate Limiting
- ACL (Access Control Lists)

```json
{
  "username": "mobile-app",
  "custom_id": "app-001"
}
```

### 4. Plugin
وحدة توفر وظيفة محددة (Authentication, Rate Limiting, Logging, etc.). يمكن تطبيقها على:
- **Global**: على جميع Requests
- **Service**: على Service محدد
- **Route**: على Route محدد
- **Consumer**: على Consumer محدد

### 5. Upstream
يمثل مجموعة من Backend Servers مع Load Balancing.

```json
{
  "name": "user-upstream",
  "algorithm": "round-robin",
  "slots": 1000
}
```

### 6. Target
Backend Server ضمن Upstream.

```json
{
  "target": "backend1.example.com:8080",
  "weight": 100
}
```

---

## المكونات الرئيسية

### 1. Kong Gateway Core
- معالجة Requests/Responses
- توجيه الطلبات
- تطبيق Plugins
- Load Balancing

### 2. Admin API
REST API لإدارة Kong:
- إنشاء Services, Routes
- تفعيل Plugins
- إدارة Consumers
- Port افتراضي: 8001

```bash
# عرض جميع Services
curl -i http://localhost:8001/services

# إنشاء Service جديد
curl -i -X POST http://localhost:8001/services \
  --data name=example-service \
  --data url=http://backend.example.com
```

### 3. Proxy
يستقبل الطلبات من العملاء:
- **HTTP**: Port 8000
- **HTTPS**: Port 8443

### 4. Database (اختياري)
- **PostgreSQL**: الأكثر شيوعاً
- **Cassandra**: للتوزيع الجغرافي
- **DB-less**: تهيئة من ملفات YAML (Declarative Config)

### 5. Kong Manager (Enterprise)
واجهة رسومية لإدارة Kong.

### 6. Dev Portal (Enterprise)
بوابة للمطورين لاستكشاف APIs.

---

## الميزات الرئيسية

### 1. High Performance
- مبني على NGINX
- يدعم آلاف الطلبات في الثانية
- Low Latency

### 2. Extensibility
- Plugin Architecture
- Custom Plugins بـ Lua
- PDK (Plugin Development Kit)

### 3. Scalability
- Horizontal Scaling
- Stateless Architecture
- Clustering Support

### 4. Cloud-Native
- Kubernetes Ingress Controller
- Docker Support
- Service Mesh Integration

### 5. Hybrid Deployment
- Control Plane / Data Plane Architecture
- يسمح بنشر موزع

---

## Plugins

Kong يوفر Plugins لمختلف الوظائف. تُقسم إلى فئات:

### 1. Authentication Plugins

#### Key Authentication
```bash
# تفعيل Plugin على Service
curl -X POST http://localhost:8001/services/user-service/plugins \
  --data name=key-auth

# إنشاء Consumer
curl -X POST http://localhost:8001/consumers \
  --data username=mobile-app

# إنشاء API Key للـ Consumer
curl -X POST http://localhost:8001/consumers/mobile-app/key-auth \
  --data key=my-secret-key
```

الاستخدام:
```bash
curl -i http://localhost:8000/api/users \
  -H "apikey: my-secret-key"
```

#### Basic Authentication
```bash
curl -X POST http://localhost:8001/services/user-service/plugins \
  --data name=basic-auth

curl -X POST http://localhost:8001/consumers/mobile-app/basic-auth \
  --data username=user \
  --data password=password123
```

#### OAuth 2.0
```bash
curl -X POST http://localhost:8001/services/user-service/plugins \
  --data name=oauth2 \
  --data config.enable_authorization_code=true \
  --data config.enable_client_credentials=true
```

#### JWT
```bash
curl -X POST http://localhost:8001/services/user-service/plugins \
  --data name=jwt
```

#### LDAP Authentication
```bash
curl -X POST http://localhost:8001/services/user-service/plugins \
  --data name=ldap-auth \
  --data config.ldap_host=ldap.example.com \
  --data config.ldap_port=389
```

### 2. Security Plugins

#### IP Restriction
```bash
curl -X POST http://localhost:8001/services/user-service/plugins \
  --data name=ip-restriction \
  --data config.whitelist=10.0.0.0/8 \
  --data config.whitelist=192.168.0.0/16
```

#### CORS
```bash
curl -X POST http://localhost:8001/services/user-service/plugins \
  --data name=cors \
  --data config.origins=* \
  --data config.methods=GET \
  --data config.methods=POST \
  --data config.headers=Content-Type \
  --data config.headers=Authorization \
  --data config.max_age=3600
```

#### Bot Detection
```bash
curl -X POST http://localhost:8001/services/user-service/plugins \
  --data name=bot-detection \
  --data config.whitelist=Googlebot \
  --data config.blacklist=BadBot
```

#### ACL (Access Control List)
```bash
# تفعيل ACL Plugin
curl -X POST http://localhost:8001/services/user-service/plugins \
  --data name=acl \
  --data config.whitelist=admin-group

# إضافة Consumer لمجموعة
curl -X POST http://localhost:8001/consumers/john/acls \
  --data group=admin-group
```

### 3. Traffic Control Plugins

#### Rate Limiting
```bash
curl -X POST http://localhost:8001/services/user-service/plugins \
  --data name=rate-limiting \
  --data config.minute=100 \
  --data config.hour=1000 \
  --data config.policy=local
```

#### Request Size Limiting
```bash
curl -X POST http://localhost:8001/services/user-service/plugins \
  --data name=request-size-limiting \
  --data config.allowed_payload_size=10
```

#### Request Termination
```bash
curl -X POST http://localhost:8001/services/user-service/plugins \
  --data name=request-termination \
  --data config.status_code=403 \
  --data config.message="Service temporarily unavailable"
```

#### Response Rate Limiting
```bash
curl -X POST http://localhost:8001/services/user-service/plugins \
  --data name=response-ratelimiting \
  --data config.limits.video.minute=10
```

### 4. Transformation Plugins

#### Request Transformer
```bash
curl -X POST http://localhost:8001/services/user-service/plugins \
  --data name=request-transformer \
  --data config.add.headers=X-Custom-Header:CustomValue \
  --data config.add.querystring=new-param:value \
  --data config.remove.headers=X-Remove-Header \
  --data config.replace.headers=X-Replace:NewValue
```

#### Response Transformer
```bash
curl -X POST http://localhost:8001/services/user-service/plugins \
  --data name=response-transformer \
  --data config.add.headers=X-Response-Header:Value \
  --data config.add.json=new-field:value \
  --data config.remove.json=sensitive-data
```

#### Correlation ID
```bash
curl -X POST http://localhost:8001/services/user-service/plugins \
  --data name=correlation-id \
  --data config.header_name=X-Correlation-ID \
  --data config.generator=uuid
```

### 5. Logging Plugins

#### File Log
```bash
curl -X POST http://localhost:8001/services/user-service/plugins \
  --data name=file-log \
  --data config.path=/tmp/kong-logs.log
```

#### HTTP Log
```bash
curl -X POST http://localhost:8001/services/user-service/plugins \
  --data name=http-log \
  --data config.http_endpoint=http://log-service.example.com/logs
```

#### Syslog
```bash
curl -X POST http://localhost:8001/services/user-service/plugins \
  --data name=syslog \
  --data config.log_level=info \
  --data config.facility=daemon
```

#### TCP Log
```bash
curl -X POST http://localhost:8001/services/user-service/plugins \
  --data name=tcp-log \
  --data config.host=logserver.example.com \
  --data config.port=5000
```

#### UDP Log
```bash
curl -X POST http://localhost:8001/services/user-service/plugins \
  --data name=udp-log \
  --data config.host=logserver.example.com \
  --data config.port=9999
```

### 6. Analytics & Monitoring Plugins

#### Prometheus
```bash
curl -X POST http://localhost:8001/plugins \
  --data name=prometheus

# عرض Metrics
curl http://localhost:8001/metrics
```

#### Zipkin (Tracing)
```bash
curl -X POST http://localhost:8001/services/user-service/plugins \
  --data name=zipkin \
  --data config.http_endpoint=http://zipkin.example.com:9411/api/v2/spans
```

#### Datadog
```bash
curl -X POST http://localhost:8001/plugins \
  --data name=datadog \
  --data config.host=datadog-agent.example.com \
  --data config.port=8125
```

### 7. Caching Plugins

#### Proxy Cache
```bash
curl -X POST http://localhost:8001/services/user-service/plugins \
  --data name=proxy-cache \
  --data config.strategy=memory \
  --data config.cache_ttl=300 \
  --data config.cache_control=true
```

### 8. Serverless Plugins

#### Pre-function (Lua)
```bash
curl -X POST http://localhost:8001/services/user-service/plugins \
  --data name=pre-function \
  --data 'config.access[1]=kong.log.err("Request received")'
```

#### Post-function (Lua)
```bash
curl -X POST http://localhost:8001/services/user-service/plugins \
  --data name=post-function \
  --data 'config.header_filter[1]=kong.response.set_header("X-Custom", "Value")'
```

#### AWS Lambda
```bash
curl -X POST http://localhost:8001/services/user-service/plugins \
  --data name=aws-lambda \
  --data config.aws_key=YOUR_AWS_KEY \
  --data config.aws_secret=YOUR_AWS_SECRET \
  --data config.aws_region=us-east-1 \
  --data config.function_name=my-lambda-function
```

---

## طرق النشر

### 1. Traditional (مع Database)

**Docker Compose مثال:**
```yaml
version: '3.8'

services:
  kong-database:
    image: postgres:13
    environment:
      POSTGRES_USER: kong
      POSTGRES_DB: kong
      POSTGRES_PASSWORD: kongpass
    ports:
      - "5432:5432"
    volumes:
      - kong-db-data:/var/lib/postgresql/data

  kong-migration:
    image: kong:latest
    command: kong migrations bootstrap
    environment:
      KONG_DATABASE: postgres
      KONG_PG_HOST: kong-database
      KONG_PG_USER: kong
      KONG_PG_PASSWORD: kongpass
    depends_on:
      - kong-database

  kong:
    image: kong:latest
    environment:
      KONG_DATABASE: postgres
      KONG_PG_HOST: kong-database
      KONG_PG_USER: kong
      KONG_PG_PASSWORD: kongpass
      KONG_PROXY_ACCESS_LOG: /dev/stdout
      KONG_ADMIN_ACCESS_LOG: /dev/stdout
      KONG_PROXY_ERROR_LOG: /dev/stderr
      KONG_ADMIN_ERROR_LOG: /dev/stderr
      KONG_ADMIN_LISTEN: 0.0.0.0:8001
    ports:
      - "8000:8000"   # Proxy
      - "8443:8443"   # Proxy SSL
      - "8001:8001"   # Admin API
      - "8444:8444"   # Admin API SSL
    depends_on:
      - kong-database
      - kong-migration

volumes:
  kong-db-data:
```

**تشغيل:**
```bash
docker-compose up -d
```

### 2. DB-less Mode

**kong.yml (Declarative Config):**
```yaml
_format_version: "3.0"

services:
  - name: user-service
    url: http://backend.example.com:8080
    routes:
      - name: user-route
        paths:
          - /api/users
        methods:
          - GET
          - POST
    plugins:
      - name: key-auth
      - name: rate-limiting
        config:
          minute: 100
          policy: local

consumers:
  - username: mobile-app
    keyauth_credentials:
      - key: my-secret-key

plugins:
  - name: prometheus
  - name: cors
    config:
      origins:
        - "*"
      methods:
        - GET
        - POST
```

**تشغيل:**
```bash
docker run -d --name kong \
  -v "$(pwd)/kong.yml:/usr/local/kong/declarative/kong.yml" \
  -e "KONG_DATABASE=off" \
  -e "KONG_DECLARATIVE_CONFIG=/usr/local/kong/declarative/kong.yml" \
  -e "KONG_PROXY_ACCESS_LOG=/dev/stdout" \
  -e "KONG_ADMIN_ACCESS_LOG=/dev/stdout" \
  -e "KONG_PROXY_ERROR_LOG=/dev/stderr" \
  -e "KONG_ADMIN_ERROR_LOG=/dev/stderr" \
  -p 8000:8000 \
  -p 8443:8443 \
  kong:latest
```

### 3. Kubernetes (Ingress Controller)

**Installation:**
```bash
# إضافة Kong Helm Repository
helm repo add kong https://charts.konghq.com
helm repo update

# تثبيت Kong
helm install kong kong/kong \
  --set ingressController.enabled=true \
  --set ingressController.installCRDs=false \
  --namespace kong \
  --create-namespace
```

**Kubernetes Ingress Example:**
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: user-api-ingress
  annotations:
    konghq.com/strip-path: "true"
    konghq.com/plugins: rate-limiting, cors
spec:
  ingressClassName: kong
  rules:
    - host: api.example.com
      http:
        paths:
          - path: /users
            pathType: Prefix
            backend:
              service:
                name: user-service
                port:
                  number: 8080
```

**KongPlugin CRD:**
```yaml
apiVersion: configuration.konghq.com/v1
kind: KongPlugin
metadata:
  name: rate-limiting
config:
  minute: 100
  policy: local
plugin: rate-limiting
---
apiVersion: configuration.konghq.com/v1
kind: KongPlugin
metadata:
  name: cors
config:
  origins:
    - "*"
  methods:
    - GET
    - POST
  headers:
    - Content-Type
    - Authorization
plugin: cors
```

### 4. Hybrid Mode (Control Plane / Data Plane)

**Control Plane:**
```bash
docker run -d --name kong-cp \
  -e "KONG_ROLE=control_plane" \
  -e "KONG_CLUSTER_CERT=/path/to/cluster.crt" \
  -e "KONG_CLUSTER_CERT_KEY=/path/to/cluster.key" \
  -p 8005:8005 \
  kong:latest
```

**Data Plane:**
```bash
docker run -d --name kong-dp \
  -e "KONG_ROLE=data_plane" \
  -e "KONG_CLUSTER_CONTROL_PLANE=kong-cp:8005" \
  -e "KONG_CLUSTER_CERT=/path/to/cluster.crt" \
  -e "KONG_CLUSTER_CERT_KEY=/path/to/cluster.key" \
  -p 8000:8000 \
  kong:latest
```

---

## حالات الاستخدام

### 1. API Gateway للـ Microservices
استخدام Kong كنقطة دخول واحدة لجميع Microservices.

### 2. Kubernetes Ingress Controller
إدارة Traffic الداخل لـ Kubernetes Cluster.

### 3. Service Mesh
دمج Kong مع Service Mesh مثل Istio أو Linkerd.

### 4. Authentication & Authorization
توفير طبقة أمان موحدة لجميع Services.

### 5. Rate Limiting & Throttling
التحكم في استهلاك APIs ومنع الإساءة.

### 6. API Transformation
تحويل Requests/Responses بين صيغ مختلفة.

### 7. Legacy Modernization
ربط Legacy Systems مع تطبيقات حديثة.

---

## أمثلة عملية

### مثال 1: إنشاء Service و Route كامل

```bash
# 1. إنشاء Service
curl -i -X POST http://localhost:8001/services \
  --data name=user-service \
  --data url=http://backend.example.com:8080/users

# 2. إنشاء Route
curl -i -X POST http://localhost:8001/services/user-service/routes \
  --data name=user-route \
  --data 'paths[]=/api/users' \
  --data 'methods[]=GET' \
  --data 'methods[]=POST'

# 3. إضافة Key Authentication
curl -i -X POST http://localhost:8001/services/user-service/plugins \
  --data name=key-auth

# 4. إنشاء Consumer
curl -i -X POST http://localhost:8001/consumers \
  --data username=mobile-app \
  --data custom_id=app-001

# 5. إنشاء API Key للـ Consumer
curl -i -X POST http://localhost:8001/consumers/mobile-app/key-auth \
  --data key=abc123xyz456

# 6. اختبار API
curl -i http://localhost:8000/api/users \
  -H "apikey: abc123xyz456"
```

### مثال 2: Load Balancing مع Upstream

```bash
# 1. إنشاء Upstream
curl -i -X POST http://localhost:8001/upstreams \
  --data name=user-upstream \
  --data algorithm=round-robin

# 2. إضافة Targets
curl -i -X POST http://localhost:8001/upstreams/user-upstream/targets \
  --data target=backend1.example.com:8080 \
  --data weight=100

curl -i -X POST http://localhost:8001/upstreams/user-upstream/targets \
  --data target=backend2.example.com:8080 \
  --data weight=100

curl -i -X POST http://localhost:8001/upstreams/user-upstream/targets \
  --data target=backend3.example.com:8080 \
  --data weight=50

# 3. إنشاء Service مع Upstream
curl -i -X POST http://localhost:8001/services \
  --data name=user-service \
  --data host=user-upstream \
  --data path=/users

# 4. إنشاء Route
curl -i -X POST http://localhost:8001/services/user-service/routes \
  --data 'paths[]=/api/users'
```

### مثال 3: Rate Limiting متقدم لـ Consumers مختلفة

```bash
# 1. إنشاء Free Tier Consumer
curl -i -X POST http://localhost:8001/consumers \
  --data username=free-user

curl -i -X POST http://localhost:8001/consumers/free-user/key-auth \
  --data key=free-key-123

# Rate Limiting للـ Free Tier
curl -i -X POST http://localhost:8001/consumers/free-user/plugins \
  --data name=rate-limiting \
  --data config.minute=10 \
  --data config.hour=100

# 2. إنشاء Premium Tier Consumer
curl -i -X POST http://localhost:8001/consumers \
  --data username=premium-user

curl -i -X POST http://localhost:8001/consumers/premium-user/key-auth \
  --data key=premium-key-456

# Rate Limiting للـ Premium Tier
curl -i -X POST http://localhost:8001/consumers/premium-user/plugins \
  --data name=rate-limiting \
  --data config.minute=100 \
  --data config.hour=10000
```

### مثال 4: Custom Plugin بـ Lua

**custom-header-plugin/schema.lua:**
```lua
return {
  name = "custom-header",
  fields = {
    {
      config = {
        type = "record",
        fields = {
          { header_name = { type = "string", required = true } },
          { header_value = { type = "string", required = true } },
        },
      },
    },
  },
}
```

**custom-header-plugin/handler.lua:**
```lua
local CustomHeaderHandler = {
  VERSION = "1.0.0",
  PRIORITY = 1000,
}

function CustomHeaderHandler:access(conf)
  kong.service.request.set_header(conf.header_name, conf.header_value)
end

return CustomHeaderHandler
```

**تثبيت واستخدام:**
```bash
# نسخ Plugin إلى Kong
cp -r custom-header-plugin /usr/local/share/lua/5.1/kong/plugins/

# إضافة إلى kong.conf
# plugins = bundled,custom-header

# تفعيل Plugin
curl -X POST http://localhost:8001/services/user-service/plugins \
  --data name=custom-header \
  --data config.header_name=X-Custom-Header \
  --data config.header_value=MyValue
```

### مثال 5: OAuth 2.0 Implementation

```bash
# 1. تفعيل OAuth2 Plugin
curl -X POST http://localhost:8001/services/user-service/plugins \
  --data name=oauth2 \
  --data config.enable_authorization_code=true \
  --data config.enable_client_credentials=true \
  --data config.enable_password_grant=true \
  --data config.scopes=email \
  --data config.scopes=profile \
  --data config.mandatory_scope=true

# 2. إنشاء Consumer
curl -X POST http://localhost:8001/consumers \
  --data username=oauth-user

# 3. إنشاء OAuth2 Application
curl -X POST http://localhost:8001/consumers/oauth-user/oauth2 \
  --data name=my-app \
  --data client_id=my-client-id \
  --data client_secret=my-client-secret \
  --data redirect_uris=http://localhost:3000/callback

# 4. الحصول على Authorization Code
# Navigate to:
# http://localhost:8000/oauth2/authorize?response_type=code&client_id=my-client-id&scope=email&provision_key=PROVISION_KEY

# 5. تبديل Code بـ Access Token
curl -X POST http://localhost:8000/oauth2/token \
  --data grant_type=authorization_code \
  --data client_id=my-client-id \
  --data client_secret=my-client-secret \
  --data code=AUTHORIZATION_CODE

# 6. استخدام Access Token
curl -H "Authorization: Bearer ACCESS_TOKEN" \
  http://localhost:8000/api/users
```

### مثال 6: Request/Response Transformation

```bash
# إضافة Headers للـ Request
curl -X POST http://localhost:8001/services/user-service/plugins \
  --data name=request-transformer \
  --data config.add.headers=X-Request-ID:$(uuidgen) \
  --data config.add.headers=X-Forwarded-By:Kong \
  --data config.add.querystring=source:kong \
  --data config.remove.headers=X-Sensitive-Header \
  --data config.replace.headers=User-Agent:Kong-Gateway

# تعديل Response
curl -X POST http://localhost:8001/services/user-service/plugins \
  --data name=response-transformer \
  --data config.add.headers=X-Response-Time:$(date +%s) \
  --data config.add.json=server:kong \
  --data config.remove.json=internal_id \
  --data config.remove.headers=X-Internal-Header
```

### مثال 7: Health Checks مع Upstream

```bash
# إنشاء Upstream مع Health Checks
curl -X POST http://localhost:8001/upstreams \
  --data name=user-upstream \
  --data healthchecks.active.healthy.interval=5 \
  --data healthchecks.active.healthy.successes=2 \
  --data healthchecks.active.unhealthy.interval=5 \
  --data healthchecks.active.unhealthy.http_failures=3 \
  --data healthchecks.active.unhealthy.timeouts=3 \
  --data healthchecks.active.http_path=/health \
  --data healthchecks.passive.healthy.successes=5 \
  --data healthchecks.passive.unhealthy.http_failures=5

# عرض حالة Health Checks
curl http://localhost:8001/upstreams/user-upstream/health
```

---

## Kong vs Apigee

| الميزة | Kong | Apigee Edge |
|--------|------|-------------|
| **النوع** | Open Source | Commercial (Google) |
| **السعر** | مجاني (OSS), مدفوع (Enterprise) | مدفوع بالكامل |
| **الأداء** | عالي جداً (NGINX) | عالي |
| **Deployment** | Self-hosted, Cloud | Cloud (Google Cloud) |
| **Plugin System** | قابل للتوسع بسهولة (Lua) | محدود بـ Policies |
| **Kubernetes** | Ingress Controller متكامل | محدود |
| **Database** | PostgreSQL, Cassandra, DB-less | Internal |
| **Developer Portal** | Enterprise فقط | مدمج |
| **Analytics** | محدود، يحتاج Integration | متقدم جداً |
| **Monetization** | غير موجود | مدمج |
| **Learning Curve** | متوسط | صعب |
| **Community** | كبير ونشط | محدود (commercial) |
| **مناسب لـ** | Startups, Kubernetes, Self-hosted | Enterprise, Google Cloud, Complex |

### متى تستخدم Kong؟
- ✅ تريد حل مفتوح المصدر
- ✅ تعمل على Kubernetes
- ✅ تحتاج أداء عالي جداً
- ✅ تريد تخصيص Plugins
- ✅ ميزانية محدودة

### متى تستخدم Apigee؟
- ✅ تعمل على Google Cloud
- ✅ تحتاج Analytics متقدم
- ✅ تريد API Monetization
- ✅ تحتاج Developer Portal شامل
- ✅ Enterprise مع ميزانية كبيرة

---

## أسئلة المقابلات

### أسئلة أساسية

**1. ما هو Kong ولماذا نستخدمه؟**
- **الإجابة**: Kong هو API Gateway مفتوح المصدر مبني على NGINX. يُستخدم لـ:
  - توجيه وإدارة APIs
  - Authentication & Authorization
  - Rate Limiting & Traffic Control
  - Logging & Monitoring
  - Request/Response Transformation

**2. ما الفرق بين Service و Route في Kong؟**
- **الإجابة**:
  - **Service**: يمثل Backend API (URL، Protocol)
  - **Route**: مسار يربط الطلبات الواردة بـ Service (Paths، Methods، Hosts)

**3. ما هي الـ Plugins في Kong؟**
- **الإجابة**: Plugins هي وحدات توفر وظائف محددة:
  - Authentication (Key Auth, OAuth, JWT)
  - Security (IP Restriction, CORS, ACL)
  - Traffic Control (Rate Limiting)
  - Transformation (Request/Response Transformer)
  - Logging (File Log, HTTP Log, Syslog)

**4. كيف يعمل Kong بدون Database (DB-less mode)؟**
- **الإجابة**: في DB-less mode:
  - التهيئة من ملف YAML declarative
  - لا حاجة لـ PostgreSQL أو Cassandra
  - مناسب لـ Kubernetes وContainers
  - التغييرات تتطلب إعادة تحميل التهيئة

**5. ما هو Consumer في Kong؟**
- **الإجابة**: Consumer يمثل المستخدم أو التطبيق الذي يستهلك APIs:
  - يُستخدم مع Authentication Plugins
  - يمكن تطبيق Rate Limiting لكل Consumer
  - يمكن تعيين ACL Permissions

### أسئلة متقدمة

**6. كيف تنفذ Load Balancing في Kong؟**
- **الإجابة**: باستخدام Upstream و Targets:
  - إنشاء Upstream مع Algorithm (round-robin, least-connections, etc.)
  - إضافة Targets (Backend Servers) مع Weights
  - دعم Health Checks (Active & Passive)
  - Automatic failover عند فشل Target

**7. ما الفرق بين Active و Passive Health Checks؟**
- **الإجابة**:
  - **Active**: Kong يرسل طلبات دورية لفحص صحة Targets
  - **Passive**: Kong يراقب Traffic الفعلي ويكتشف الأخطاء

**8. كيف تكتب Custom Plugin في Kong؟**
- **الإجابة**: Custom Plugins تُكتب بـ Lua:
  - إنشاء schema.lua (تعريف Configuration)
  - إنشاء handler.lua (منطق Plugin)
  - تنفيذ Phases (access, header_filter, body_filter, log)
  - تثبيت Plugin وتفعيله

**9. ما هي Plugin Phases في Kong؟**
- **الإجابة**:
```
certificate → rewrite → access → balancer
→ header_filter → body_filter → log
```

**10. كيف تؤمن Kong Admin API؟**
- **الإجابة**:
  - استخدام IP Whitelisting
  - تفعيل RBAC (Role-Based Access Control) - Enterprise
  - استخدام API Key Authentication
  - تشغيل Admin API على Network داخلي فقط
  - استخدام Kong Manager مع Authentication

**11. ما الفرق بين Traditional و Hybrid Mode في Kong؟**
- **الإجابة**:
  - **Traditional**: جميع Kong Nodes متصلة بنفس Database
  - **Hybrid**: Control Plane (يدير التهيئة) منفصل عن Data Planes (تعالج Traffic)
  - Hybrid Mode يوفر أمان وأداء أفضل

**12. كيف تنفذ Rate Limiting لـ Consumers مختلفة؟**
- **الإجابة**: تطبيق Rate Limiting Plugin على مستوى Consumer:
```bash
# Free tier
curl -X POST http://localhost:8001/consumers/free-user/plugins \
  --data name=rate-limiting \
  --data config.minute=10

# Premium tier
curl -X POST http://localhost:8001/consumers/premium-user/plugins \
  --data name=rate-limiting \
  --data config.minute=1000
```

**13. كيف تتعامل مع CORS في Kong؟**
- **الإجابة**: استخدام CORS Plugin:
```bash
curl -X POST http://localhost:8001/plugins \
  --data name=cors \
  --data config.origins=* \
  --data config.methods=GET,POST,PUT,DELETE \
  --data config.headers=Content-Type,Authorization \
  --data config.credentials=true \
  --data config.max_age=3600
```

**14. كيف تراقب Kong Performance؟**
- **الإجابة**:
  - استخدام Prometheus Plugin للـ Metrics
  - تفعيل Logging Plugins (File, HTTP, Syslog)
  - Integration مع Datadog, New Relic
  - استخدام Zipkin للـ Tracing
  - Kong Vitals (Enterprise)

**15. كيف تنشر Kong على Kubernetes؟**
- **الإجابة**:
  - استخدام Kong Ingress Controller
  - تثبيت من Helm Chart
  - إنشاء Kubernetes Ingress Resources
  - استخدام CRDs (KongPlugin, KongConsumer, etc.)
  - Declarative Configuration من ConfigMaps

**16. ما هي Best Practices لاستخدام Kong؟**
- **الإجابة**:
  - استخدام DB-less mode للـ Kubernetes
  - تطبيق Authentication في أقرب نقطة
  - استخدام Caching لتحسين الأداء
  - تفعيل Health Checks للـ Upstreams
  - مراقبة Metrics باستمرار
  - استخدام Declarative Config للـ Version Control

**17. كيف تتعامل مع API Versioning في Kong؟**
- **الإجابة**:
  - استخدام Paths مختلفة: `/v1/users`, `/v2/users`
  - إنشاء Services و Routes منفصلة لكل Version
  - استخدام Header-based routing
  - Request Transformer للتوافق بين Versions

**18. ما هو Kong Gateway PDK؟**
- **الإجابة**: Plugin Development Kit - مجموعة من APIs لتطوير Plugins:
  - kong.request: الوصول للـ Request
  - kong.response: تعديل Response
  - kong.service: التواصل مع Backend
  - kong.log: Logging
  - kong.cache: Caching

**19. كيف تنفذ Circuit Breaker في Kong؟**
- **الإجابة**: لا يوجد Circuit Breaker Plugin built-in، لكن يمكن:
  - استخدام Passive Health Checks للكشف عن Failures
  - كتابة Custom Plugin
  - Integration مع Service Mesh (Istio)

**20. كيف تتعامل مع Authentication متعددة في Kong؟**
- **الإجابة**: يمكن تفعيل Plugins متعددة:
  - استخدام Key Auth للـ Internal APIs
  - استخدام OAuth 2.0 للـ Third-party
  - استخدام JWT للـ Mobile Apps
  - تطبيق Plugins على Routes مختلفة

### أسئلة مقارنة

**21. Kong vs NGINX: ما الفرق؟**
- **الإجابة**:
  - Kong مبني على NGINX لكنه API Gateway كامل
  - Kong يوفر Plugin Architecture
  - Kong يدعم Admin API للإدارة
  - NGINX يحتاج تهيئة يدوية معقدة

**22. Kong vs API Gateway Alternatives؟**
| الميزة | Kong | AWS API Gateway | Apigee |
|--------|------|-----------------|--------|
| Open Source | ✅ | ❌ | ❌ |
| Self-hosted | ✅ | ❌ | ❌ |
| Kubernetes | ✅ | ❌ | محدود |
| Performance | عالي جداً | متوسط | عالي |
| Cost | مجاني/مدفوع | Pay-per-use | مرتفع |

---

## الخلاصة

Kong Gateway هو حل قوي ومرن لإدارة APIs:
- ✅ Open Source مع مجتمع نشط
- ✅ أداء عالي جداً (NGINX-based)
- ✅ Plugin Architecture قابل للتوسع
- ✅ دعم كامل لـ Kubernetes
- ✅ DB-less mode للبيئات الحديثة
- ✅ Hybrid Deployment للمشاريع الكبيرة
- ✅ تكلفة منخفضة (OSS version)

**متى تستخدم Kong:**
- عندما تريد حل مفتوح المصدر
- للمشاريع على Kubernetes
- عندما تحتاج أداء عالي جداً
- للمشاريع بميزانية محدودة
- عندما تريد تخصيص كامل

**الأدوات المكملة:**
- **Konga**: UI لإدارة Kong (Open Source)
- **Kong Manager**: UI رسمي (Enterprise)
- **decK**: CLI لإدارة Kong Declaratively
- **Insomnia**: API Testing من نفس الشركة