# دليل الأمان الشامل - Spring Boot Security

## جدول المحتويات
1. [JWT (JSON Web Tokens)](#jwt-json-web-tokens)
2. [Spring Security](#spring-security)
3. [OAuth vs OAuth2](#oauth-vs-oauth2)
4. [طرق المصادقة المختلفة](#authentication-methods)
5. [أفضل ممارسات الأمان](#security-best-practices)

---

# JWT (JSON Web Tokens)

## ما هو JWT؟

JWT (JSON Web Token) هو معيار مفتوح (RFC 7519) يُستخدم لنقل المعلومات بشكل آمن بين الأطراف على شكل JSON object. يتم توقيع الـ JWT رقمياً، مما يضمن أن المعلومات موثوقة ولم يتم التلاعب بها.

### الخصائص الرئيسية:
- **Stateless**: الخادم لا يحتاج لحفظ الـ session
- **Self-contained**: يحتوي على جميع المعلومات اللازمة
- **Compact**: حجم صغير، سهل النقل عبر URL, POST parameter, أو HTTP header
- **Secure**: يمكن توقيعه وتشفيره

---

## مكونات JWT

JWT يتكون من ثلاثة أجزاء مفصولة بنقطة (.)

```
xxxxx.yyyyy.zzzzz
```

### 1. Header (الرأس)
يحتوي على نوع الـ token ونوع خوارزمية التشفير

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

### 2. Payload (الحمولة)
يحتوي على الـ Claims (المعلومات المخزنة)

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

#### أنواع الـ Claims:
- **Registered Claims**: معرفة مسبقاً (iss, exp, sub, aud)
- **Public Claims**: يمكن تعريفها حسب الحاجة
- **Private Claims**: claims خاصة بالتطبيق

### 3. Signature (التوقيع)
يُستخدم للتحقق من أن الرسالة لم تتغير

```javascript
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  secret
)
```

---

## كيف يعمل JWT؟

### سير العمل:

```
1. المستخدم يرسل username & password
   ↓
2. الخادم يتحقق من البيانات
   ↓
3. إذا صحيحة، ينشئ JWT ويرسله للمستخدم
   ↓
4. المستخدم يحفظ الـ JWT (localStorage, cookie)
   ↓
5. في كل طلب، يرسل JWT في Authorization header
   ↓
6. الخادم يتحقق من صحة الـ JWT
   ↓
7. إذا صحيح، يسمح بالوصول للموارد
```

### مثال توضيحي:

```
Client                              Server
  |                                   |
  |  POST /login                      |
  |  {username, password}             |
  |---------------------------------->|
  |                                   | يتحقق من البيانات
  |                                   | ينشئ JWT
  |  JWT Token                        |
  |<----------------------------------|
  |                                   |
  | يحفظ الـ Token                     |
  |                                   |
  |  GET /api/users                   |
  |  Authorization: Bearer <JWT>      |
  |---------------------------------->|
  |                                   | يتحقق من الـ JWT
  |                                   | يفك تشفيره
  |  Response Data                    |
  |<----------------------------------|
```

---

## JWT vs Session

### Session-based Authentication:

```
Client                              Server
  |                                   |
  |  POST /login                      |
  |---------------------------------->|
  |                                   | ينشئ Session ID
  |  Session ID (Cookie)              | يحفظ البيانات في الـ Server
  |<----------------------------------|
  |                                   |
  |  GET /api/users                   |
  |  Cookie: session_id=xxx           |
  |---------------------------------->|
  |                                   | يبحث عن الـ Session
  |                                   | يسترجع البيانات من الذاكرة
  |  Response                         |
  |<----------------------------------|
```

### JWT-based Authentication:

```
Client                              Server
  |                                   |
  |  POST /login                      |
  |---------------------------------->|
  |                                   | ينشئ JWT
  |  JWT Token                        | لا يحفظ شيء (Stateless)
  |<----------------------------------|
  |                                   |
  |  GET /api/users                   |
  |  Authorization: Bearer <JWT>      |
  |---------------------------------->|
  |                                   | يتحقق من التوقيع فقط
  |  Response                         | يفك تشفير الـ JWT
  |<----------------------------------|
```

### المقارنة التفصيلية:

| الميزة | Session | JWT |
|-------|---------|-----|
| **التخزين في الخادم** | نعم (يحتاج ذاكرة) | لا (Stateless) |
| **قابلية التوسع** | صعبة (تحتاج Session Store مشترك) | سهلة جداً |
| **الأمان** | أكثر أماناً (يمكن إلغاء Session) | أقل (صعب إلغاء Token قبل انتهائه) |
| **الحجم** | صغير (Session ID فقط) | أكبر (يحمل البيانات) |
| **الأداء** | يحتاج قراءة من Database/Redis | سريع (فقط فك تشفير) |
| **CORS** | معقد | بسيط |
| **Mobile Apps** | صعب | ممتاز |

---

## Access Token vs Refresh Token

### المشكلة:
- **Access Token طويل المدة**: خطر أمني إذا تم سرقته
- **Access Token قصير المدة**: المستخدم يحتاج تسجيل دخول متكرر

### الحل: استخدام Token مزدوج

```
┌─────────────────┐
│  Access Token   │  صلاحية: 15 دقيقة
│  يُستخدم في كل  │  يحمل: user info, permissions
│  طلب API        │  الحجم: كبير نسبياً
└─────────────────┘

┌─────────────────┐
│  Refresh Token  │  صلاحية: 7 أيام / 30 يوم
│  يُستخدم فقط    │  يحمل: user id فقط
│  لتجديد Access  │  الحجم: صغير
└─────────────────┘
```

### سير العمل:

```
1. تسجيل الدخول
   ↓
2. الخادم يرسل: Access Token + Refresh Token
   ↓
3. استخدام Access Token في الطلبات
   ↓
4. عند انتهاء Access Token (401 Unauthorized)
   ↓
5. إرسال Refresh Token لـ /refresh endpoint
   ↓
6. الخادم يرسل Access Token جديد
   ↓
7. استمرار العمل
```

---

## أمثلة Spring Boot كاملة

### 1. Dependencies (pom.xml)

```xml
<dependencies>
    <!-- Spring Boot Starter Web -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <!-- Spring Security -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>

    <!-- JWT Library -->
    <dependency>
        <groupId>io.jsonwebtoken</groupId>
        <artifactId>jjwt-api</artifactId>
        <version>0.11.5</version>
    </dependency>
    <dependency>
        <groupId>io.jsonwebtoken</groupId>
        <artifactId>jjwt-impl</artifactId>
        <version>0.11.5</version>
        <scope>runtime</scope>
    </dependency>
    <dependency>
        <groupId>io.jsonwebtoken</groupId>
        <artifactId>jjwt-jackson</artifactId>
        <version>0.11.5</version>
        <scope>runtime</scope>
    </dependency>

    <!-- Spring Data JPA -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>

    <!-- MySQL Driver -->
    <dependency>
        <groupId>com.mysql</groupId>
        <artifactId>mysql-connector-j</artifactId>
        <scope>runtime</scope>
    </dependency>

    <!-- Lombok -->
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <optional>true</optional>
    </dependency>
</dependencies>
```

### 2. Application Properties

```properties
# application.properties

# Database Configuration
spring.datasource.url=jdbc:mysql://localhost:3306/security_db
spring.datasource.username=root
spring.datasource.password=password
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true

# JWT Configuration
jwt.secret=mySecretKeyForJWTTokenGenerationMustBeLongEnough256Bits
jwt.access-token.expiration=900000
# 15 minutes = 900,000 ms
jwt.refresh-token.expiration=604800000
# 7 days = 604,800,000 ms
```

### 3. User Entity

```java
package com.example.security.entity;

import jakarta.persistence.*;
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

import java.util.Set;

@Entity
@Table(name = "users")
@Data
@NoArgsConstructor
@AllArgsConstructor
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(unique = true, nullable = false)
    private String username;

    @Column(unique = true, nullable = false)
    private String email;

    @Column(nullable = false)
    private String password;

    @ElementCollection(fetch = FetchType.EAGER)
    @CollectionTable(name = "user_roles", joinColumns = @JoinColumn(name = "user_id"))
    @Column(name = "role")
    private Set<String> roles;

    private boolean enabled = true;
    private boolean accountNonExpired = true;
    private boolean accountNonLocked = true;
    private boolean credentialsNonExpired = true;
}
```

### 4. RefreshToken Entity

```java
package com.example.security.entity;

import jakarta.persistence.*;
import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

import java.time.Instant;

@Entity
@Table(name = "refresh_tokens")
@Data
@NoArgsConstructor
@AllArgsConstructor
public class RefreshToken {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, unique = true)
    private String token;

    @Column(nullable = false)
    private Instant expiryDate;

    @OneToOne
    @JoinColumn(name = "user_id", referencedColumnName = "id")
    private User user;

    private boolean revoked = false;
}
```

### 5. JWT Utility Class

```java
package com.example.security.util;

import io.jsonwebtoken.*;
import io.jsonwebtoken.security.Keys;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.stereotype.Component;

import java.security.Key;
import java.util.Date;
import java.util.HashMap;
import java.util.Map;
import java.util.function.Function;

@Component
public class JwtUtil {

    @Value("${jwt.secret}")
    private String secret;

    @Value("${jwt.access-token.expiration}")
    private Long accessTokenExpiration;

    @Value("${jwt.refresh-token.expiration}")
    private Long refreshTokenExpiration;

    // إنشاء Key من الـ secret
    private Key getSigningKey() {
        return Keys.hmacShaKeyFor(secret.getBytes());
    }

    // استخراج Username من Token
    public String extractUsername(String token) {
        return extractClaim(token, Claims::getSubject);
    }

    // استخراج تاريخ الانتهاء
    public Date extractExpiration(String token) {
        return extractClaim(token, Claims::getExpiration);
    }

    // استخراج أي Claim
    public <T> T extractClaim(String token, Function<Claims, T> claimsResolver) {
        final Claims claims = extractAllClaims(token);
        return claimsResolver.apply(claims);
    }

    // استخراج جميع الـ Claims
    private Claims extractAllClaims(String token) {
        return Jwts.parserBuilder()
                .setSigningKey(getSigningKey())
                .build()
                .parseClaimsJws(token)
                .getBody();
    }

    // التحقق من انتهاء صلاحية الـ Token
    private Boolean isTokenExpired(String token) {
        return extractExpiration(token).before(new Date());
    }

    // إنشاء Access Token
    public String generateAccessToken(UserDetails userDetails) {
        Map<String, Object> claims = new HashMap<>();
        claims.put("roles", userDetails.getAuthorities());
        return createToken(claims, userDetails.getUsername(), accessTokenExpiration);
    }

    // إنشاء Access Token مع Claims إضافية
    public String generateAccessToken(UserDetails userDetails, Map<String, Object> extraClaims) {
        Map<String, Object> claims = new HashMap<>(extraClaims);
        claims.put("roles", userDetails.getAuthorities());
        return createToken(claims, userDetails.getUsername(), accessTokenExpiration);
    }

    // إنشاء Refresh Token
    public String generateRefreshToken(String username) {
        Map<String, Object> claims = new HashMap<>();
        claims.put("type", "refresh");
        return createToken(claims, username, refreshTokenExpiration);
    }

    // إنشاء Token
    private String createToken(Map<String, Object> claims, String subject, Long expiration) {
        Date now = new Date();
        Date expiryDate = new Date(now.getTime() + expiration);

        return Jwts.builder()
                .setClaims(claims)
                .setSubject(subject)
                .setIssuedAt(now)
                .setExpiration(expiryDate)
                .signWith(getSigningKey(), SignatureAlgorithm.HS256)
                .compact();
    }

    // التحقق من صحة الـ Token
    public Boolean validateToken(String token, UserDetails userDetails) {
        try {
            final String username = extractUsername(token);
            return (username.equals(userDetails.getUsername()) && !isTokenExpired(token));
        } catch (JwtException | IllegalArgumentException e) {
            return false;
        }
    }

    // التحقق من صحة الـ Token فقط
    public Boolean validateToken(String token) {
        try {
            Jwts.parserBuilder()
                .setSigningKey(getSigningKey())
                .build()
                .parseClaimsJws(token);
            return true;
        } catch (JwtException | IllegalArgumentException e) {
            return false;
        }
    }
}
```

### 6. UserDetailsService Implementation

```java
package com.example.security.service;

import com.example.security.entity.User;
import com.example.security.repository.UserRepository;
import lombok.RequiredArgsConstructor;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.stereotype.Service;

import java.util.stream.Collectors;

@Service
@RequiredArgsConstructor
public class CustomUserDetailsService implements UserDetailsService {

    private final UserRepository userRepository;

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        User user = userRepository.findByUsername(username)
                .orElseThrow(() -> new UsernameNotFoundException("User not found: " + username));

        return org.springframework.security.core.userdetails.User.builder()
                .username(user.getUsername())
                .password(user.getPassword())
                .authorities(user.getRoles().stream()
                        .map(role -> new SimpleGrantedAuthority("ROLE_" + role))
                        .collect(Collectors.toList()))
                .accountExpired(!user.isAccountNonExpired())
                .accountLocked(!user.isAccountNonLocked())
                .credentialsExpired(!user.isCredentialsNonExpired())
                .disabled(!user.isEnabled())
                .build();
    }
}
```

### 7. JWT Authentication Filter

```java
package com.example.security.filter;

import com.example.security.util.JwtUtil;
import jakarta.servlet.FilterChain;
import jakarta.servlet.ServletException;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import lombok.RequiredArgsConstructor;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.web.authentication.WebAuthenticationDetailsSource;
import org.springframework.stereotype.Component;
import org.springframework.web.filter.OncePerRequestFilter;

import java.io.IOException;

@Component
@RequiredArgsConstructor
public class JwtAuthenticationFilter extends OncePerRequestFilter {

    private final JwtUtil jwtUtil;
    private final UserDetailsService userDetailsService;

    @Override
    protected void doFilterInternal(
            HttpServletRequest request,
            HttpServletResponse response,
            FilterChain filterChain
    ) throws ServletException, IOException {

        // استخراج الـ Authorization header
        final String authHeader = request.getHeader("Authorization");
        final String jwt;
        final String username;

        // التحقق من وجود الـ Token
        if (authHeader == null || !authHeader.startsWith("Bearer ")) {
            filterChain.doFilter(request, response);
            return;
        }

        // استخراج الـ Token
        jwt = authHeader.substring(7);

        try {
            // استخراج الـ Username من الـ Token
            username = jwtUtil.extractUsername(jwt);

            // إذا كان الـ username موجود والمستخدم غير مصادق
            if (username != null && SecurityContextHolder.getContext().getAuthentication() == null) {

                // تحميل بيانات المستخدم
                UserDetails userDetails = userDetailsService.loadUserByUsername(username);

                // التحقق من صحة الـ Token
                if (jwtUtil.validateToken(jwt, userDetails)) {

                    // إنشاء Authentication Token
                    UsernamePasswordAuthenticationToken authToken =
                        new UsernamePasswordAuthenticationToken(
                            userDetails,
                            null,
                            userDetails.getAuthorities()
                        );

                    // إضافة تفاصيل الطلب
                    authToken.setDetails(
                        new WebAuthenticationDetailsSource().buildDetails(request)
                    );

                    // تعيين الـ Authentication في الـ SecurityContext
                    SecurityContextHolder.getContext().setAuthentication(authToken);
                }
            }
        } catch (Exception e) {
            // يمكن تسجيل الخطأ هنا
            logger.error("Cannot set user authentication: {}", e);
        }

        filterChain.doFilter(request, response);
    }
}
```

### 8. Refresh Token Service

```java
package com.example.security.service;

import com.example.security.entity.RefreshToken;
import com.example.security.entity.User;
import com.example.security.repository.RefreshTokenRepository;
import com.example.security.repository.UserRepository;
import lombok.RequiredArgsConstructor;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.time.Instant;
import java.util.Optional;
import java.util.UUID;

@Service
@RequiredArgsConstructor
public class RefreshTokenService {

    @Value("${jwt.refresh-token.expiration}")
    private Long refreshTokenDurationMs;

    private final RefreshTokenRepository refreshTokenRepository;
    private final UserRepository userRepository;

    // إنشاء Refresh Token
    public RefreshToken createRefreshToken(Long userId) {
        User user = userRepository.findById(userId)
                .orElseThrow(() -> new RuntimeException("User not found"));

        // حذف الـ Refresh Tokens القديمة للمستخدم
        refreshTokenRepository.deleteByUser(user);

        RefreshToken refreshToken = new RefreshToken();
        refreshToken.setUser(user);
        refreshToken.setToken(UUID.randomUUID().toString());
        refreshToken.setExpiryDate(Instant.now().plusMillis(refreshTokenDurationMs));
        refreshToken.setRevoked(false);

        return refreshTokenRepository.save(refreshToken);
    }

    // البحث عن Refresh Token
    public Optional<RefreshToken> findByToken(String token) {
        return refreshTokenRepository.findByToken(token);
    }

    // التحقق من صلاحية الـ Refresh Token
    public RefreshToken verifyExpiration(RefreshToken token) {
        if (token.getExpiryDate().compareTo(Instant.now()) < 0) {
            refreshTokenRepository.delete(token);
            throw new RuntimeException("Refresh token was expired. Please make a new signin request");
        }
        if (token.isRevoked()) {
            throw new RuntimeException("Refresh token was revoked");
        }
        return token;
    }

    // إلغاء Refresh Token
    @Transactional
    public void revokeToken(String token) {
        refreshTokenRepository.findByToken(token)
                .ifPresent(rt -> {
                    rt.setRevoked(true);
                    refreshTokenRepository.save(rt);
                });
    }

    // حذف جميع Refresh Tokens للمستخدم
    @Transactional
    public void deleteByUserId(Long userId) {
        User user = userRepository.findById(userId)
                .orElseThrow(() -> new RuntimeException("User not found"));
        refreshTokenRepository.deleteByUser(user);
    }
}
```

### 9. Authentication Service

```java
package com.example.security.service;

import com.example.security.dto.*;
import com.example.security.entity.RefreshToken;
import com.example.security.entity.User;
import com.example.security.repository.UserRepository;
import com.example.security.util.JwtUtil;
import lombok.RequiredArgsConstructor;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.authentication.UsernamePasswordAuthenticationToken;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.stereotype.Service;

import java.util.HashMap;
import java.util.Map;
import java.util.Set;

@Service
@RequiredArgsConstructor
public class AuthenticationService {

    private final UserRepository userRepository;
    private final PasswordEncoder passwordEncoder;
    private final JwtUtil jwtUtil;
    private final AuthenticationManager authenticationManager;
    private final UserDetailsService userDetailsService;
    private final RefreshTokenService refreshTokenService;

    // تسجيل مستخدم جديد
    public AuthResponse register(RegisterRequest request) {

        // التحقق من عدم وجود المستخدم
        if (userRepository.existsByUsername(request.getUsername())) {
            throw new RuntimeException("Username already exists");
        }
        if (userRepository.existsByEmail(request.getEmail())) {
            throw new RuntimeException("Email already exists");
        }

        // إنشاء المستخدم
        User user = new User();
        user.setUsername(request.getUsername());
        user.setEmail(request.getEmail());
        user.setPassword(passwordEncoder.encode(request.getPassword()));
        user.setRoles(Set.of("USER")); // صلاحية افتراضية
        user.setEnabled(true);
        user.setAccountNonExpired(true);
        user.setAccountNonLocked(true);
        user.setCredentialsNonExpired(true);

        User savedUser = userRepository.save(user);

        // إنشاء الـ Tokens
        UserDetails userDetails = userDetailsService.loadUserByUsername(savedUser.getUsername());
        String accessToken = jwtUtil.generateAccessToken(userDetails);
        RefreshToken refreshToken = refreshTokenService.createRefreshToken(savedUser.getId());

        return AuthResponse.builder()
                .accessToken(accessToken)
                .refreshToken(refreshToken.getToken())
                .tokenType("Bearer")
                .expiresIn(900000L) // 15 minutes
                .build();
    }

    // تسجيل الدخول
    public AuthResponse login(LoginRequest request) {

        // المصادقة
        authenticationManager.authenticate(
            new UsernamePasswordAuthenticationToken(
                request.getUsername(),
                request.getPassword()
            )
        );

        // تحميل المستخدم
        User user = userRepository.findByUsername(request.getUsername())
                .orElseThrow(() -> new RuntimeException("User not found"));

        UserDetails userDetails = userDetailsService.loadUserByUsername(user.getUsername());

        // إنشاء الـ Tokens
        Map<String, Object> extraClaims = new HashMap<>();
        extraClaims.put("userId", user.getId());
        extraClaims.put("email", user.getEmail());

        String accessToken = jwtUtil.generateAccessToken(userDetails, extraClaims);
        RefreshToken refreshToken = refreshTokenService.createRefreshToken(user.getId());

        return AuthResponse.builder()
                .accessToken(accessToken)
                .refreshToken(refreshToken.getToken())
                .tokenType("Bearer")
                .expiresIn(900000L)
                .build();
    }

    // تجديد Access Token
    public AuthResponse refreshToken(RefreshTokenRequest request) {

        String requestRefreshToken = request.getRefreshToken();

        return refreshTokenService.findByToken(requestRefreshToken)
                .map(refreshTokenService::verifyExpiration)
                .map(RefreshToken::getUser)
                .map(user -> {
                    UserDetails userDetails = userDetailsService.loadUserByUsername(user.getUsername());
                    String accessToken = jwtUtil.generateAccessToken(userDetails);

                    return AuthResponse.builder()
                            .accessToken(accessToken)
                            .refreshToken(requestRefreshToken)
                            .tokenType("Bearer")
                            .expiresIn(900000L)
                            .build();
                })
                .orElseThrow(() -> new RuntimeException("Refresh token is not in database"));
    }

    // تسجيل الخروج
    public void logout(String refreshToken) {
        refreshTokenService.revokeToken(refreshToken);
    }
}
```

### 10. DTOs

```java
// LoginRequest.java
package com.example.security.dto;

import lombok.Data;

@Data
public class LoginRequest {
    private String username;
    private String password;
}

// RegisterRequest.java
package com.example.security.dto;

import lombok.Data;

@Data
public class RegisterRequest {
    private String username;
    private String email;
    private String password;
}

// RefreshTokenRequest.java
package com.example.security.dto;

import lombok.Data;

@Data
public class RefreshTokenRequest {
    private String refreshToken;
}

// AuthResponse.java
package com.example.security.dto;

import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class AuthResponse {
    private String accessToken;
    private String refreshToken;
    private String tokenType;
    private Long expiresIn;
}
```

### 11. Authentication Controller

```java
package com.example.security.controller;

import com.example.security.dto.*;
import com.example.security.service.AuthenticationService;
import lombok.RequiredArgsConstructor;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/auth")
@RequiredArgsConstructor
public class AuthController {

    private final AuthenticationService authenticationService;

    @PostMapping("/register")
    public ResponseEntity<AuthResponse> register(@RequestBody RegisterRequest request) {
        return ResponseEntity.ok(authenticationService.register(request));
    }

    @PostMapping("/login")
    public ResponseEntity<AuthResponse> login(@RequestBody LoginRequest request) {
        return ResponseEntity.ok(authenticationService.login(request));
    }

    @PostMapping("/refresh")
    public ResponseEntity<AuthResponse> refreshToken(@RequestBody RefreshTokenRequest request) {
        return ResponseEntity.ok(authenticationService.refreshToken(request));
    }

    @PostMapping("/logout")
    public ResponseEntity<Void> logout(@RequestBody RefreshTokenRequest request) {
        authenticationService.logout(request.getRefreshToken());
        return ResponseEntity.ok().build();
    }
}
```

### 12. Repositories

```java
// UserRepository.java
package com.example.security.repository;

import com.example.security.entity.User;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

import java.util.Optional;

@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    Optional<User> findByUsername(String username);
    Optional<User> findByEmail(String email);
    Boolean existsByUsername(String username);
    Boolean existsByEmail(String email);
}

// RefreshTokenRepository.java
package com.example.security.repository;

import com.example.security.entity.RefreshToken;
import com.example.security.entity.User;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

import java.util.Optional;

@Repository
public interface RefreshTokenRepository extends JpaRepository<RefreshToken, Long> {
    Optional<RefreshToken> findByToken(String token);
    void deleteByUser(User user);
}
```

### 13. Security Configuration

```java
package com.example.security.config;

import com.example.security.filter.JwtAuthenticationFilter;
import lombok.RequiredArgsConstructor;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.authentication.AuthenticationManager;
import org.springframework.security.authentication.AuthenticationProvider;
import org.springframework.security.authentication.dao.DaoAuthenticationProvider;
import org.springframework.security.config.annotation.authentication.configuration.AuthenticationConfiguration;
import org.springframework.security.config.annotation.method.configuration.EnableMethodSecurity;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.http.SessionCreationPolicy;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.web.SecurityFilterChain;
import org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter;

@Configuration
@EnableWebSecurity
@EnableMethodSecurity
@RequiredArgsConstructor
public class SecurityConfig {

    private final JwtAuthenticationFilter jwtAuthFilter;
    private final UserDetailsService userDetailsService;

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .csrf(csrf -> csrf.disable())
            .authorizeHttpRequests(auth -> auth
                // السماح بالوصول للـ endpoints العامة
                .requestMatchers("/api/auth/**").permitAll()
                .requestMatchers("/api/public/**").permitAll()
                // المستخدمون المصادقون فقط
                .requestMatchers("/api/user/**").hasRole("USER")
                // المدراء فقط
                .requestMatchers("/api/admin/**").hasRole("ADMIN")
                // باقي الطلبات تحتاج مصادقة
                .anyRequest().authenticated()
            )
            .sessionManagement(session -> session
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
            )
            .authenticationProvider(authenticationProvider())
            .addFilterBefore(jwtAuthFilter, UsernamePasswordAuthenticationFilter.class);

        return http.build();
    }

    @Bean
    public AuthenticationProvider authenticationProvider() {
        DaoAuthenticationProvider authProvider = new DaoAuthenticationProvider();
        authProvider.setUserDetailsService(userDetailsService);
        authProvider.setPasswordEncoder(passwordEncoder());
        return authProvider;
    }

    @Bean
    public AuthenticationManager authenticationManager(AuthenticationConfiguration config)
            throws Exception {
        return config.getAuthenticationManager();
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

### 14. Test Controller

```java
package com.example.security.controller;

import org.springframework.http.ResponseEntity;
import org.springframework.security.access.prepost.PreAuthorize;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.HashMap;
import java.util.Map;

@RestController
@RequestMapping("/api")
public class TestController {

    @GetMapping("/public/hello")
    public ResponseEntity<String> publicEndpoint() {
        return ResponseEntity.ok("Hello from public endpoint!");
    }

    @GetMapping("/user/profile")
    @PreAuthorize("hasRole('USER')")
    public ResponseEntity<Map<String, Object>> userProfile() {
        Authentication auth = SecurityContextHolder.getContext().getAuthentication();

        Map<String, Object> profile = new HashMap<>();
        profile.put("username", auth.getName());
        profile.put("authorities", auth.getAuthorities());

        return ResponseEntity.ok(profile);
    }

    @GetMapping("/admin/dashboard")
    @PreAuthorize("hasRole('ADMIN')")
    public ResponseEntity<String> adminDashboard() {
        return ResponseEntity.ok("Welcome to Admin Dashboard!");
    }

    @GetMapping("/user/me")
    public ResponseEntity<Map<String, Object>> getCurrentUser() {
        Authentication auth = SecurityContextHolder.getContext().getAuthentication();

        Map<String, Object> user = new HashMap<>();
        user.put("username", auth.getName());
        user.put("roles", auth.getAuthorities());
        user.put("authenticated", auth.isAuthenticated());

        return ResponseEntity.ok(user);
    }
}
```

### 15. اختبار الـ API

#### تسجيل مستخدم جديد:
```bash
POST http://localhost:8080/api/auth/register
Content-Type: application/json

{
  "username": "ahmed",
  "email": "ahmed@example.com",
  "password": "password123"
}

# Response:
{
  "accessToken": "eyJhbGciOiJIUzI1NiJ9...",
  "refreshToken": "550e8400-e29b-41d4-a716-446655440000",
  "tokenType": "Bearer",
  "expiresIn": 900000
}
```

#### تسجيل الدخول:
```bash
POST http://localhost:8080/api/auth/login
Content-Type: application/json

{
  "username": "ahmed",
  "password": "password123"
}

# Response:
{
  "accessToken": "eyJhbGciOiJIUzI1NiJ9...",
  "refreshToken": "550e8400-e29b-41d4-a716-446655440000",
  "tokenType": "Bearer",
  "expiresIn": 900000
}
```

#### الوصول لـ endpoint محمي:
```bash
GET http://localhost:8080/api/user/profile
Authorization: Bearer eyJhbGciOiJIUzI1NiJ9...

# Response:
{
  "username": "ahmed",
  "authorities": [
    {
      "authority": "ROLE_USER"
    }
  ]
}
```

#### تجديد Access Token:
```bash
POST http://localhost:8080/api/auth/refresh
Content-Type: application/json

{
  "refreshToken": "550e8400-e29b-41d4-a716-446655440000"
}

# Response:
{
  "accessToken": "eyJhbGciOiJIUzI1NiJ9...", // token جديد
  "refreshToken": "550e8400-e29b-41d4-a716-446655440000",
  "tokenType": "Bearer",
  "expiresIn": 900000
}
```

#### تسجيل الخروج:
```bash
POST http://localhost:8080/api/auth/logout
Content-Type: application/json

{
  "refreshToken": "550e8400-e29b-41d4-a716-446655440000"
}
```

---

# Spring Security

## ما هو Spring Security؟

Spring Security هو إطار عمل قوي وقابل للتخصيص يوفر:
- **المصادقة (Authentication)**: التحقق من هوية المستخدم
- **التفويض (Authorization)**: التحقق من صلاحيات المستخدم
- **الحماية من الهجمات**: CSRF, Session Fixation, Clickjacking, XSS

### الميزات الرئيسية:
- دعم شامل للمصادقة والتفويض
- الحماية من الثغرات الأمنية الشائعة
- تكامل مع Servlet API
- تكامل اختياري مع Spring MVC
- دعم OAuth 2.0, SAML, LDAP

---

## Authentication vs Authorization

### Authentication (المصادقة)
**"من أنت؟"**

هي عملية التحقق من هوية المستخدم.

```
المستخدم: أنا أحمد
النظام: أثبت ذلك (أدخل كلمة المرور)
المستخدم: password123
النظام: ✓ تم التحقق، مرحباً أحمد
```

**أمثلة:**
- Username/Password
- Biometric (بصمة، وجه)
- OTP (One-Time Password)
- JWT Token

### Authorization (التفويض)
**"ماذا يمكنك أن تفعل؟"**

هي عملية التحقق من صلاحيات المستخدم.

```
المستخدم (أحمد): أريد حذف هذا المنتج
النظام: لنتحقق من صلاحياتك...
النظام: ✗ أنت USER، لكن تحتاج ADMIN
```

**أمثلة:**
- Role-based (ADMIN, USER, MODERATOR)
- Permission-based (READ, WRITE, DELETE)
- Resource-based (يمكن تعديل منشوراته فقط)

### مثال توضيحي:

```java
@RestController
@RequestMapping("/api/products")
public class ProductController {

    // يحتاج Authentication فقط (أي مستخدم مسجل)
    @GetMapping
    public List<Product> getAllProducts() {
        return productService.findAll();
    }

    // يحتاج Authorization: ROLE_USER أو أعلى
    @PostMapping
    @PreAuthorize("hasRole('USER')")
    public Product createProduct(@RequestBody Product product) {
        return productService.save(product);
    }

    // يحتاج Authorization: ROLE_ADMIN فقط
    @DeleteMapping("/{id}")
    @PreAuthorize("hasRole('ADMIN')")
    public void deleteProduct(@PathVariable Long id) {
        productService.delete(id);
    }

    // يحتاج صلاحية محددة
    @PutMapping("/{id}/approve")
    @PreAuthorize("hasAuthority('APPROVE_PRODUCT')")
    public Product approveProduct(@PathVariable Long id) {
        return productService.approve(id);
    }
}
```

---

## SecurityFilterChain

SecurityFilterChain هو سلسلة من الـ Filters التي تعالج الطلبات الأمنية.

### كيف يعمل؟

```
HTTP Request
    ↓
┌─────────────────────────────────────┐
│   SecurityFilterChain               │
├─────────────────────────────────────┤
│ 1. SecurityContextPersistenceFilter │ ← يحفظ/يسترجع SecurityContext
│ 2. LogoutFilter                     │ ← يعالج تسجيل الخروج
│ 3. UsernamePasswordAuthFilter       │ ← يعالج تسجيل الدخول
│ 4. BasicAuthenticationFilter        │ ← Basic Auth
│ 5. JwtAuthenticationFilter          │ ← JWT (مخصص)
│ 6. ExceptionTranslationFilter       │ ← يعالج الاستثناءات
│ 7. FilterSecurityInterceptor        │ ← يتحقق من الصلاحيات
└─────────────────────────────────────┘
    ↓
Controller
```

### مثال Configuration:

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            // تعطيل CSRF (للـ REST APIs)
            .csrf(csrf -> csrf.disable())

            // تكوين الصلاحيات
            .authorizeHttpRequests(auth -> auth
                // Endpoints عامة
                .requestMatchers("/api/public/**").permitAll()
                .requestMatchers("/api/auth/**").permitAll()

                // Endpoints حسب الصلاحية
                .requestMatchers("/api/admin/**").hasRole("ADMIN")
                .requestMatchers("/api/user/**").hasAnyRole("USER", "ADMIN")

                // باقي الطلبات
                .anyRequest().authenticated()
            )

            // Session Management
            .sessionManagement(session -> session
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
            )

            // إضافة JWT Filter
            .addFilterBefore(
                jwtAuthFilter,
                UsernamePasswordAuthenticationFilter.class
            )

            // معالجة الأخطاء
            .exceptionHandling(ex -> ex
                .authenticationEntryPoint(
                    (request, response, authException) -> {
                        response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
                        response.setContentType("application/json");
                        response.getWriter().write(
                            "{\"error\": \"Unauthorized\"}"
                        );
                    }
                )
                .accessDeniedHandler(
                    (request, response, accessDeniedException) -> {
                        response.setStatus(HttpServletResponse.SC_FORBIDDEN);
                        response.setContentType("application/json");
                        response.getWriter().write(
                            "{\"error\": \"Access Denied\"}"
                        );
                    }
                )
            );

        return http.build();
    }
}
```

### Security Matchers:

```java
// تطابق دقيق
.requestMatchers("/api/login").permitAll()

// تطابق نمط
.requestMatchers("/api/admin/**").hasRole("ADMIN")

// تطابق عدة مسارات
.requestMatchers("/api/public/**", "/api/health").permitAll()

// تطابق حسب HTTP Method
.requestMatchers(HttpMethod.POST, "/api/products").hasRole("ADMIN")
.requestMatchers(HttpMethod.GET, "/api/products").permitAll()

// تطابق حسب regex
.requestMatchers(RegexRequestMatcher.regexMatcher("/api/users/\\d+")).authenticated()
```

---

## UserDetailsService

UserDetailsService هو interface يستخدم لتحميل بيانات المستخدم من قاعدة البيانات.

### Interface Definition:

```java
public interface UserDetailsService {
    UserDetails loadUserByUsername(String username) throws UsernameNotFoundException;
}
```

### UserDetails Interface:

```java
public interface UserDetails extends Serializable {
    Collection<? extends GrantedAuthority> getAuthorities();
    String getPassword();
    String getUsername();
    boolean isAccountNonExpired();
    boolean isAccountNonLocked();
    boolean isCredentialsNonExpired();
    boolean isEnabled();
}
```

### Implementation Example:

```java
@Service
@RequiredArgsConstructor
public class CustomUserDetailsService implements UserDetailsService {

    private final UserRepository userRepository;

    @Override
    public UserDetails loadUserByUsername(String username)
            throws UsernameNotFoundException {

        // 1. البحث عن المستخدم في قاعدة البيانات
        User user = userRepository.findByUsername(username)
                .orElseThrow(() ->
                    new UsernameNotFoundException("User not found: " + username)
                );

        // 2. تحويل الـ Roles إلى GrantedAuthority
        Set<GrantedAuthority> authorities = user.getRoles().stream()
                .map(role -> new SimpleGrantedAuthority("ROLE_" + role))
                .collect(Collectors.toSet());

        // 3. إرجاع UserDetails
        return org.springframework.security.core.userdetails.User.builder()
                .username(user.getUsername())
                .password(user.getPassword())
                .authorities(authorities)
                .accountExpired(!user.isAccountNonExpired())
                .accountLocked(!user.isAccountNonLocked())
                .credentialsExpired(!user.isCredentialsNonExpired())
                .disabled(!user.isEnabled())
                .build();
    }
}
```

### Custom UserDetails Implementation:

```java
@Getter
@AllArgsConstructor
public class CustomUserDetails implements UserDetails {

    private final User user;

    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return user.getRoles().stream()
                .map(role -> new SimpleGrantedAuthority("ROLE_" + role))
                .collect(Collectors.toList());
    }

    @Override
    public String getPassword() {
        return user.getPassword();
    }

    @Override
    public String getUsername() {
        return user.getUsername();
    }

    @Override
    public boolean isAccountNonExpired() {
        return user.isAccountNonExpired();
    }

    @Override
    public boolean isAccountNonLocked() {
        return user.isAccountNonLocked();
    }

    @Override
    public boolean isCredentialsNonExpired() {
        return user.isCredentialsNonExpired();
    }

    @Override
    public boolean isEnabled() {
        return user.isEnabled();
    }

    // يمكن إضافة methods إضافية
    public String getEmail() {
        return user.getEmail();
    }

    public Long getId() {
        return user.getId();
    }
}
```

---

## PasswordEncoder (BCrypt)

PasswordEncoder هو interface لتشفير كلمات المرور.

### لماذا BCrypt؟

1. **Salt تلقائي**: يضيف قيمة عشوائية لكل كلمة مرور
2. **Adaptive**: يمكن زيادة التعقيد مع زيادة قوة المعالجات
3. **آمن**: صعب جداً كسره حتى مع الهجمات المتقدمة

### مثال الاستخدام:

```java
@Configuration
public class PasswordEncoderConfig {

    @Bean
    public PasswordEncoder passwordEncoder() {
        // strength = 10 (default)
        // كلما زاد الرقم، زاد الأمان والوقت
        return new BCryptPasswordEncoder();
    }

    // أو مع strength مخصص
    @Bean
    public PasswordEncoder strongPasswordEncoder() {
        return new BCryptPasswordEncoder(12); // أقوى ولكن أبطأ
    }
}
```

### كيف يعمل BCrypt:

```java
@Service
@RequiredArgsConstructor
public class PasswordService {

    private final PasswordEncoder passwordEncoder;

    // تشفير كلمة المرور
    public String encodePassword(String rawPassword) {
        String encoded = passwordEncoder.encode(rawPassword);
        System.out.println("Raw: " + rawPassword);
        System.out.println("Encoded: " + encoded);
        return encoded;
    }

    // التحقق من كلمة المرور
    public boolean verifyPassword(String rawPassword, String encodedPassword) {
        return passwordEncoder.matches(rawPassword, encodedPassword);
    }
}

// مثال الاستخدام:
String password = "myPassword123";
String encoded1 = passwordEncoder.encode(password);
String encoded2 = passwordEncoder.encode(password);

// نفس كلمة المرور، لكن hash مختلف (بسبب الـ salt)
System.out.println(encoded1);
// $2a$10$N9qo8uLOickgx2ZMRZoMye1J6KJvFvmQqPq3lUJvJnN6gHSJQ6Y.m

System.out.println(encoded2);
// $2a$10$fT7FvDjQF3tIr5YJ9xYOceWmHDUJxVJvGnQQqJQOqJqQqJQOqJqQ.

// لكن كلاهما يطابق كلمة المرور الأصلية
passwordEncoder.matches(password, encoded1); // true
passwordEncoder.matches(password, encoded2); // true
```

### تفسير BCrypt Hash:

```
$2a$10$N9qo8uLOickgx2ZMRZoMye1J6KJvFvmQqPq3lUJvJnN6gHSJQ6Y.m
│  │  │                                                          │
│  │  │                                                          └─ Hash (31 chars)
│  │  └─ Salt (22 chars)
│  └─ Cost/Strength (10 = 2^10 iterations)
└─ Algorithm Version (2a = BCrypt)
```

### مثال كامل - User Registration:

```java
@Service
@RequiredArgsConstructor
public class UserService {

    private final UserRepository userRepository;
    private final PasswordEncoder passwordEncoder;

    public User registerUser(RegisterRequest request) {

        // التحقق من عدم وجود المستخدم
        if (userRepository.existsByUsername(request.getUsername())) {
            throw new RuntimeException("Username already exists");
        }

        // إنشاء المستخدم
        User user = new User();
        user.setUsername(request.getUsername());
        user.setEmail(request.getEmail());

        // تشفير كلمة المرور
        user.setPassword(passwordEncoder.encode(request.getPassword()));

        user.setRoles(Set.of("USER"));
        user.setEnabled(true);

        return userRepository.save(user);
    }

    public void changePassword(Long userId, ChangePasswordRequest request) {
        User user = userRepository.findById(userId)
                .orElseThrow(() -> new RuntimeException("User not found"));

        // التحقق من كلمة المرور القديمة
        if (!passwordEncoder.matches(request.getOldPassword(), user.getPassword())) {
            throw new RuntimeException("Old password is incorrect");
        }

        // التحقق من أن كلمة المرور الجديدة مختلفة
        if (passwordEncoder.matches(request.getNewPassword(), user.getPassword())) {
            throw new RuntimeException("New password must be different");
        }

        // تحديث كلمة المرور
        user.setPassword(passwordEncoder.encode(request.getNewPassword()));
        userRepository.save(user);
    }
}
```

### أنواع أخرى من PasswordEncoders:

```java
@Configuration
public class MultiplePasswordEncodersConfig {

    // للتطبيقات القديمة - NOT RECOMMENDED
    @Bean
    public PasswordEncoder delegatingPasswordEncoder() {
        String encodingId = "bcrypt";
        Map<String, PasswordEncoder> encoders = new HashMap<>();
        encoders.put(encodingId, new BCryptPasswordEncoder());
        encoders.put("pbkdf2", new Pbkdf2PasswordEncoder());
        encoders.put("scrypt", new SCryptPasswordEncoder());
        encoders.put("sha256", new StandardPasswordEncoder()); // Deprecated

        PasswordEncoder passwordEncoder =
            new DelegatingPasswordEncoder(encodingId, encoders);
        return passwordEncoder;
    }
}

// الـ hash سيكون بالشكل:
// {bcrypt}$2a$10$...
// {pbkdf2}$pbkdf2$...
```

### Password Validation:

```java
@Component
public class PasswordValidator {

    public void validate(String password) {
        if (password == null || password.length() < 8) {
            throw new IllegalArgumentException(
                "Password must be at least 8 characters"
            );
        }

        if (!password.matches(".*[A-Z].*")) {
            throw new IllegalArgumentException(
                "Password must contain at least one uppercase letter"
            );
        }

        if (!password.matches(".*[a-z].*")) {
            throw new IllegalArgumentException(
                "Password must contain at least one lowercase letter"
            );
        }

        if (!password.matches(".*\\d.*")) {
            throw new IllegalArgumentException(
                "Password must contain at least one digit"
            );
        }

        if (!password.matches(".*[@#$%^&+=].*")) {
            throw new IllegalArgumentException(
                "Password must contain at least one special character"
            );
        }
    }
}
```

---

# OAuth vs OAuth2

## ما هو OAuth؟

OAuth (Open Authorization) هو بروتوكول يسمح للتطبيقات بالوصول لموارد المستخدم دون الحاجة لمشاركة كلمة المرور.

### مثال من الحياة الواقعية:

```
بدلاً من:
    التطبيق: أعطني username و password لـ Gmail
    أنت: username@gmail.com / mypassword123
    ❌ خطر: التطبيق الآن يملك كلمة مرورك!

مع OAuth:
    التطبيق: يحولك لصفحة Google
    Google: هل تريد السماح لهذا التطبيق بقراءة emails؟
    أنت: نعم، أوافق
    Google: تعطي التطبيق access token
    ✓ آمن: التطبيق لا يعرف كلمة مرورك
```

---

## OAuth 1.0 (القديم)

### الخصائص:
- صدر في 2007
- معقد جداً في التنفيذ
- يتطلب توقيع رقمي لكل طلب
- لا يدعم Mobile Apps بشكل جيد

### سير العمل (مبسط):

```
1. التطبيق يطلب Request Token
2. المستخدم يُحوّل لصفحة المزود
3. المستخدم يوافق
4. المزود يُعيد التوجيه مع Request Token
5. التطبيق يستبدل Request Token بـ Access Token
6. التطبيق يستخدم Access Token للوصول للموارد
```

### مثال توقيع الطلب (معقد):

```
OAuth oauth_consumer_key="xvz1evFS4wEEPTGEFPHBog",
      oauth_nonce="kYjzVBB8Y0ZFabxSWbWovY3uYSQ2pTgmZeNu2VS4cg",
      oauth_signature="tnnArxj06cWHq44gCs1OSKk%2FjLY%3D",
      oauth_signature_method="HMAC-SHA1",
      oauth_timestamp="1318622958",
      oauth_token="370773112-GmHxMAgYyLbNEtIKZeRNFsMKPR9EyMZeS9weJAEb",
      oauth_version="1.0"
```

---

## OAuth 2.0 (الحالي)

### التحسينات على OAuth 1.0:
- أبسط بكثير في التنفيذ
- دعم أفضل للـ Mobile Apps
- لا يحتاج توقيع معقد (يعتمد على HTTPS)
- فصل الـ Roles بوضوح
- عدة Flows لحالات مختلفة

### الفروقات الرئيسية:

| الميزة | OAuth 1.0 | OAuth 2.0 |
|-------|-----------|-----------|
| **التعقيد** | معقد جداً | بسيط نسبياً |
| **التوقيع** | مطلوب لكل طلب | HTTPS فقط |
| **Access Token** | طويل المدى | قصير المدى + Refresh Token |
| **Mobile Support** | ضعيف | ممتاز |
| **Flows** | واحد فقط | 4 أنواع رئيسية |
| **الأمان** | معقد لكن آمن | بسيط وآمن (مع HTTPS) |

---

## OAuth 2.0 Roles

```
┌──────────────────┐
│ Resource Owner   │  ← المستخدم (أنت)
└──────────────────┘

┌──────────────────┐
│ Client           │  ← التطبيق (مثل: تطبيق الهاتف)
└──────────────────┘

┌──────────────────┐
│ Resource Server  │  ← الخادم الذي يحفظ الموارد (مثل: Gmail API)
└──────────────────┘

┌──────────────────┐
│ Authorization    │  ← خادم المصادقة (مثل: Google Accounts)
│ Server           │
└──────────────────┘
```

---

## OAuth 2.0 Flows

### 1. Authorization Code Flow
**الأفضل لـ: Web Applications**

```
User                Client               Auth Server          Resource Server
 |                    |                      |                      |
 |  1. Login          |                      |                      |
 |------------------->|                      |                      |
 |                    |  2. Redirect to Auth |                      |
 |                    |--------------------->|                      |
 |  3. Login & Approve|                      |                      |
 |<--------------------------------------|                      |
 |                    |                      |                      |
 |  4. Auth Code      |                      |                      |
 |------------------------------------->|                      |
 |                    |  5. Auth Code        |                      |
 |                    |--------------------->|                      |
 |                    |  6. Access Token     |                      |
 |                    |<---------------------|                      |
 |                    |  7. Request with Token                      |
 |                    |----------------------------------------->|
 |                    |  8. Protected Resource                      |
 |                    |<-----------------------------------------|
```

#### مثال Spring Boot:

```java
@Configuration
@EnableWebSecurity
public class OAuth2Config {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/", "/login").permitAll()
                .anyRequest().authenticated()
            )
            .oauth2Login(oauth2 -> oauth2
                .loginPage("/login")
                .defaultSuccessUrl("/dashboard")
                .failureUrl("/login?error=true")
            );

        return http.build();
    }
}

// application.yml
spring:
  security:
    oauth2:
      client:
        registration:
          google:
            client-id: your-client-id
            client-secret: your-client-secret
            scope:
              - email
              - profile
            redirect-uri: "{baseUrl}/login/oauth2/code/{registrationId}"
          github:
            client-id: your-github-client-id
            client-secret: your-github-client-secret
            scope:
              - user:email
              - read:user
```

#### Controller Example:

```java
@Controller
public class OAuth2Controller {

    @GetMapping("/")
    public String home() {
        return "home";
    }

    @GetMapping("/dashboard")
    public String dashboard(
            @AuthenticationPrincipal OAuth2User principal,
            Model model
    ) {
        // معلومات المستخدم من OAuth2 Provider
        model.addAttribute("name", principal.getAttribute("name"));
        model.addAttribute("email", principal.getAttribute("email"));
        model.addAttribute("picture", principal.getAttribute("picture"));

        return "dashboard";
    }

    @GetMapping("/user-info")
    @ResponseBody
    public Map<String, Object> userInfo(@AuthenticationPrincipal OAuth2User principal) {
        return principal.getAttributes();
    }
}
```

### 2. Implicit Flow
**⚠️ Not Recommended - Use Authorization Code with PKCE instead**

```
User                Client               Auth Server
 |                    |                      |
 |  1. Login          |                      |
 |------------------->|                      |
 |                    |  2. Redirect         |
 |                    |--------------------->|
 |  3. Login & Approve|                      |
 |<--------------------------------------|
 |  4. Access Token (in URL Fragment)       |
 |------------------------------------->|
 |                    | ❌ Token exposed in URL!
```

**لماذا غير آمن؟**
- الـ Token يظهر في الـ URL
- لا يوجد Refresh Token
- مناسب فقط لـ Single Page Apps القديمة

### 3. Client Credentials Flow
**الأفضل لـ: Server-to-Server Communication**

```
Client Application          Auth Server          Resource Server
       |                         |                      |
       | 1. Client ID + Secret   |                      |
       |------------------------>|                      |
       | 2. Access Token         |                      |
       |<------------------------|                      |
       | 3. Request with Token   |                      |
       |--------------------------------------------->|
       | 4. Protected Resource   |                      |
       |<---------------------------------------------|
```

#### مثال Spring Boot:

```java
@Configuration
public class OAuth2ClientConfig {

    @Bean
    public OAuth2AuthorizedClientManager authorizedClientManager(
            ClientRegistrationRepository clientRegistrationRepository,
            OAuth2AuthorizedClientRepository authorizedClientRepository
    ) {
        OAuth2AuthorizedClientProvider authorizedClientProvider =
            OAuth2AuthorizedClientProviderBuilder.builder()
                .clientCredentials()
                .build();

        DefaultOAuth2AuthorizedClientManager authorizedClientManager =
            new DefaultOAuth2AuthorizedClientManager(
                clientRegistrationRepository,
                authorizedClientRepository
            );

        authorizedClientManager.setAuthorizedClientProvider(authorizedClientProvider);

        return authorizedClientManager;
    }

    @Bean
    public WebClient webClient(OAuth2AuthorizedClientManager authorizedClientManager) {
        ServletOAuth2AuthorizedClientExchangeFilterFunction oauth2Client =
            new ServletOAuth2AuthorizedClientExchangeFilterFunction(
                authorizedClientManager
            );

        return WebClient.builder()
                .apply(oauth2Client.oauth2Configuration())
                .build();
    }
}

// application.yml
spring:
  security:
    oauth2:
      client:
        registration:
          my-api:
            client-id: my-client-id
            client-secret: my-client-secret
            authorization-grant-type: client_credentials
            scope: read, write
        provider:
          my-api:
            token-uri: https://auth-server.com/oauth2/token
```

#### Service Usage:

```java
@Service
@RequiredArgsConstructor
public class ApiService {

    private final WebClient webClient;

    public String callProtectedApi() {
        return webClient
            .get()
            .uri("https://api.example.com/data")
            .attributes(
                ServletOAuth2AuthorizedClientExchangeFilterFunction
                    .clientRegistrationId("my-api")
            )
            .retrieve()
            .bodyToMono(String.class)
            .block();
    }
}
```

### 4. Resource Owner Password Credentials Flow
**⚠️ استخدم فقط للتطبيقات الموثوقة جداً**

```
User                Client               Auth Server
 |                    |                      |
 | Username/Password  |                      |
 |------------------->|                      |
 |                    | Username/Password    |
 |                    |--------------------->|
 |                    | Access Token         |
 |                    |<---------------------|
```

**متى تستخدمه؟**
- تطبيق الهاتف الرسمي للشركة
- تطبيق Desktop الموثوق
- **لا تستخدمه** لتطبيقات third-party

#### مثال Spring Boot:

```java
@Service
@RequiredArgsConstructor
public class PasswordFlowService {

    private final RestTemplate restTemplate;

    public TokenResponse login(String username, String password) {
        String tokenUrl = "https://auth-server.com/oauth2/token";

        HttpHeaders headers = new HttpHeaders();
        headers.setContentType(MediaType.APPLICATION_FORM_URLENCODED);
        headers.setBasicAuth("client-id", "client-secret");

        MultiValueMap<String, String> body = new LinkedMultiValueMap<>();
        body.add("grant_type", "password");
        body.add("username", username);
        body.add("password", password);
        body.add("scope", "read write");

        HttpEntity<MultiValueMap<String, String>> request =
            new HttpEntity<>(body, headers);

        ResponseEntity<TokenResponse> response = restTemplate.postForEntity(
            tokenUrl,
            request,
            TokenResponse.class
        );

        return response.getBody();
    }
}

@Data
class TokenResponse {
    private String access_token;
    private String token_type;
    private int expires_in;
    private String refresh_token;
    private String scope;
}
```

---

## OpenID Connect (OIDC)

OpenID Connect هو طبقة هوية فوق OAuth 2.0.

### OAuth 2.0 vs OIDC:

| | OAuth 2.0 | OpenID Connect |
|---|-----------|----------------|
| **الغرض** | Authorization | Authentication + Authorization |
| **المخرجات** | Access Token | ID Token + Access Token |
| **معلومات المستخدم** | لا توجد | UserInfo endpoint |
| **الاستخدام** | "السماح بالوصول للموارد" | "تسجيل الدخول" |

### ID Token (JWT):

```json
{
  "iss": "https://accounts.google.com",
  "sub": "110169484474386276334",
  "azp": "client-id.apps.googleusercontent.com",
  "aud": "client-id.apps.googleusercontent.com",
  "iat": 1516239022,
  "exp": 1516242622,
  "email": "user@example.com",
  "email_verified": true,
  "name": "Ahmed Ali",
  "picture": "https://...",
  "given_name": "Ahmed",
  "family_name": "Ali",
  "locale": "ar"
}
```

### مثال Spring Boot مع OIDC:

```java
@Configuration
@EnableWebSecurity
public class OIDCConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth
                .anyRequest().authenticated()
            )
            .oauth2Login(oauth2 -> oauth2
                .userInfoEndpoint(userInfo -> userInfo
                    .oidcUserService(oidcUserService())
                )
            );

        return http.build();
    }

    @Bean
    public OidcUserService oidcUserService() {
        OidcUserService delegate = new OidcUserService();

        return new OidcUserService() {
            @Override
            public OidcUser loadUser(OidcUserRequest userRequest)
                    throws OAuth2AuthenticationException {

                // تحميل المستخدم من OIDC Provider
                OidcUser oidcUser = delegate.loadUser(userRequest);

                // يمكن معالجة البيانات هنا
                // مثل: حفظ المستخدم في قاعدة البيانات

                return oidcUser;
            }
        };
    }
}

@RestController
public class UserController {

    @GetMapping("/user")
    public Map<String, Object> user(@AuthenticationPrincipal OidcUser principal) {
        Map<String, Object> user = new HashMap<>();

        // من ID Token
        user.put("sub", principal.getSubject());
        user.put("email", principal.getEmail());
        user.put("name", principal.getFullName());
        user.put("picture", principal.getPicture());

        // ID Token كامل
        OidcIdToken idToken = principal.getIdToken();
        user.put("idToken", idToken.getTokenValue());

        // جميع الـ Claims
        user.put("claims", principal.getClaims());

        return user;
    }
}
```

---

## أمثلة تطبيقية كاملة

### مثال 1: تطبيق يدعم Google و GitHub Login

```java
// SecurityConfig.java
@Configuration
@EnableWebSecurity
@RequiredArgsConstructor
public class MultiProviderSecurityConfig {

    private final CustomOAuth2UserService customOAuth2UserService;

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/", "/login", "/oauth2/**").permitAll()
                .anyRequest().authenticated()
            )
            .oauth2Login(oauth2 -> oauth2
                .loginPage("/login")
                .userInfoEndpoint(userInfo -> userInfo
                    .userService(customOAuth2UserService)
                )
                .successHandler(oauth2SuccessHandler())
            )
            .logout(logout -> logout
                .logoutSuccessUrl("/")
                .deleteCookies("JSESSIONID")
            );

        return http.build();
    }

    @Bean
    public AuthenticationSuccessHandler oauth2SuccessHandler() {
        return (request, response, authentication) -> {
            OAuth2User oauth2User = (OAuth2User) authentication.getPrincipal();

            // يمكن حفظ/تحديث المستخدم في قاعدة البيانات
            String email = oauth2User.getAttribute("email");
            String name = oauth2User.getAttribute("name");

            response.sendRedirect("/dashboard");
        };
    }
}

// CustomOAuth2UserService.java
@Service
@RequiredArgsConstructor
public class CustomOAuth2UserService extends DefaultOAuth2UserService {

    private final UserRepository userRepository;

    @Override
    public OAuth2User loadUser(OAuth2UserRequest userRequest)
            throws OAuth2AuthenticationException {

        OAuth2User oauth2User = super.loadUser(userRequest);

        String registrationId = userRequest.getClientRegistration().getRegistrationId();
        String email = oauth2User.getAttribute("email");
        String name = oauth2User.getAttribute("name");

        // حفظ/تحديث المستخدم في قاعدة البيانات
        User user = userRepository.findByEmail(email)
                .orElse(new User());

        user.setEmail(email);
        user.setName(name);
        user.setProvider(registrationId); // google, github, etc.
        user.setProviderId(oauth2User.getAttribute("sub"));
        user.setEnabled(true);

        userRepository.save(user);

        return oauth2User;
    }
}

// application.yml
spring:
  security:
    oauth2:
      client:
        registration:
          google:
            client-id: ${GOOGLE_CLIENT_ID}
            client-secret: ${GOOGLE_CLIENT_SECRET}
            scope:
              - email
              - profile
          github:
            client-id: ${GITHUB_CLIENT_ID}
            client-secret: ${GITHUB_CLIENT_SECRET}
            scope:
              - user:email
              - read:user
          facebook:
            client-id: ${FACEBOOK_CLIENT_ID}
            client-secret: ${FACEBOOK_CLIENT_SECRET}
            scope:
              - email
              - public_profile
```

### login.html:

```html
<!DOCTYPE html>
<html>
<head>
    <title>Login</title>
</head>
<body>
    <h1>Login</h1>

    <a href="/oauth2/authorization/google">
        <button>Login with Google</button>
    </a>

    <a href="/oauth2/authorization/github">
        <button>Login with GitHub</button>
    </a>

    <a href="/oauth2/authorization/facebook">
        <button>Login with Facebook</button>
    </a>
</body>
</html>
```

---

# Authentication Methods

## مقارنة شاملة

### 1. Basic Authentication

```
Authorization: Basic dXNlcm5hbWU6cGFzc3dvcmQ=
                     └─ Base64(username:password)
```

#### الإيجابيات:
- بسيط جداً
- سهل التنفيذ
- لا يحتاج session أو state

#### السلبيات:
- يُرسل كلمة المرور في كل طلب
- Base64 ليس تشفير (يمكن فك تشفيره بسهولة)
- يجب استخدام HTTPS حتماً
- لا يدعم logout

#### متى تستخدمه:
- APIs داخلية بسيطة
- مع HTTPS فقط
- للاختبار والتطوير

#### مثال Spring Boot:

```java
@Configuration
public class BasicAuthConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth
                .anyRequest().authenticated()
            )
            .httpBasic(Customizer.withDefaults())
            .sessionManagement(session -> session
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
            );

        return http.build();
    }

    @Bean
    public InMemoryUserDetailsManager userDetailsService() {
        UserDetails user = User.builder()
            .username("user")
            .password("{bcrypt}$2a$10$...")
            .roles("USER")
            .build();

        return new InMemoryUserDetailsManager(user);
    }
}

// الاستخدام:
// curl -u username:password http://localhost:8080/api/users
```

---

### 2. JWT Authentication

```
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

#### الإيجابيات:
- Stateless (لا يحتاج تخزين في الخادم)
- قابل للتوسع
- يحمل معلومات المستخدم
- ممتاز للـ Microservices

#### السلبيات:
- لا يمكن إلغاؤه بسهولة
- الحجم أكبر من Session ID
- إذا تسرب، يبقى صالح حتى انتهاء صلاحيته

#### متى تستخدمه:
- REST APIs
- Mobile Applications
- Single Page Applications
- Microservices Architecture

#### مثال (مُغطى بالتفصيل في قسم JWT أعلاه)

---

### 3. OAuth2 Authentication

```
Authorization: Bearer <access_token_from_oauth2_provider>
```

#### الإيجابيات:
- لا تحتاج إدارة كلمات المرور
- الأمان من مسؤولية الـ Provider
- تجربة مستخدم سلسة
- دعم Multi-factor Authentication

#### السلبيات:
- يعتمد على third-party
- معقد قليلاً في الإعداد
- يحتاج اتصال إنترنت

#### متى تستخدمه:
- تطبيقات B2C
- عندما تريد "Login with Google/Facebook"
- عندما لا تريد إدارة المصادقة بنفسك

#### مثال (مُغطى بالتفصيل في قسم OAuth2 أعلاه)

---

### 4. Session-based Authentication

```
Cookie: JSESSIONID=7C8D9E0F1A2B3C4D5E6F7A8B9C0D1E2F
```

#### الإيجابيات:
- بسيط ومباشر
- يمكن إلغاء الـ Session بسهولة
- آمن (الـ Session محفوظ في الخادم)
- دعم قديم وموثوق

#### السلبيات:
- Stateful (يحتاج ذاكرة في الخادم)
- صعوبة التوسع (يحتاج Session Store)
- CORS معقد
- لا يناسب Mobile Apps

#### متى تستخدمه:
- تطبيقات Web تقليدية
- عندما تحتاج logout فوري
- تطبيقات Server-Side Rendering

#### مثال Spring Boot:

```java
@Configuration
@EnableWebSecurity
public class SessionBasedSecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/", "/login", "/register").permitAll()
                .anyRequest().authenticated()
            )
            .formLogin(form -> form
                .loginPage("/login")
                .loginProcessingUrl("/perform-login")
                .defaultSuccessUrl("/dashboard", true)
                .failureUrl("/login?error=true")
                .permitAll()
            )
            .logout(logout -> logout
                .logoutUrl("/perform-logout")
                .logoutSuccessUrl("/")
                .deleteCookies("JSESSIONID")
                .invalidateHttpSession(true)
            )
            .sessionManagement(session -> session
                .sessionCreationPolicy(SessionCreationPolicy.IF_REQUIRED)
                .maximumSessions(1)
                .maxSessionsPreventsLogin(true)
            );

        return http.build();
    }
}

@Controller
public class LoginController {

    @GetMapping("/login")
    public String loginPage() {
        return "login";
    }

    @GetMapping("/dashboard")
    public String dashboard(HttpSession session, Model model) {
        model.addAttribute("sessionId", session.getId());
        return "dashboard";
    }
}
```

---

## جدول المقارنة الشامل

| الميزة | Basic Auth | Session | JWT | OAuth2 |
|-------|-----------|---------|-----|--------|
| **Stateless** | ✅ نعم | ❌ لا | ✅ نعم | ✅ نعم |
| **الأمان** | ⚠️ متوسط | ✅ عالي | ✅ عالي | ✅ عالي جداً |
| **التعقيد** | ✅ بسيط | ✅ بسيط | ⚠️ متوسط | ❌ معقد |
| **Mobile Support** | ⚠️ ممكن | ❌ ضعيف | ✅ ممتاز | ✅ ممتاز |
| **Logout** | ❌ لا يوجد | ✅ فوري | ⚠️ صعب | ✅ ممكن |
| **قابلية التوسع** | ✅ ممتاز | ❌ صعب | ✅ ممتاز | ✅ ممتاز |
| **CORS** | ✅ بسيط | ⚠️ معقد | ✅ بسيط | ✅ بسيط |
| **Third-party** | ❌ | ❌ | ❌ | ✅ نعم |

---

## متى تستخدم كل منها؟

### استخدم Basic Authentication إذا:
```
✓ تطبيق داخلي بسيط
✓ API للاختبار فقط
✓ عدد مستخدمين قليل
✗ لا تستخدمه للإنتاج
```

### استخدم Session-based إذا:
```
✓ تطبيق Web تقليدي (Server-Side Rendering)
✓ تحتاج logout فوري
✓ المستخدمون يستخدمون Browsers فقط
✓ تطبيق صغير/متوسط الحجم
```

### استخدم JWT إذا:
```
✓ REST API
✓ تطبيق Mobile
✓ Single Page Application
✓ Microservices
✓ تحتاج قابلية توسع عالية
```

### استخدم OAuth2 إذا:
```
✓ تريد "Login with Google/Facebook"
✓ لا تريد إدارة كلمات المرور
✓ تطبيق B2C
✓ تحتاج Multi-factor Authentication
```

---

## مثال: Hybrid Approach

يمكن الجمع بين أكثر من طريقة:

```java
@Configuration
@EnableWebSecurity
public class HybridSecurityConfig {

    private final JwtAuthenticationFilter jwtAuthFilter;

    @Bean
    public SecurityFilterChain apiSecurityFilterChain(HttpSecurity http)
            throws Exception {
        http
            .securityMatcher("/api/**")
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/auth/**").permitAll()
                .anyRequest().authenticated()
            )
            .sessionManagement(session -> session
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
            )
            .addFilterBefore(jwtAuthFilter, UsernamePasswordAuthenticationFilter.class);

        return http.build();
    }

    @Bean
    public SecurityFilterChain webSecurityFilterChain(HttpSecurity http)
            throws Exception {
        http
            .securityMatcher("/**")
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/", "/login", "/register").permitAll()
                .anyRequest().authenticated()
            )
            .formLogin(form -> form
                .loginPage("/login")
                .defaultSuccessUrl("/dashboard")
            )
            .oauth2Login(oauth2 -> oauth2
                .loginPage("/login")
            );

        return http.build();
    }
}
```

الآن لديك:
- `/api/**` يستخدم JWT
- `/` يستخدم Session + OAuth2

---

# Security Best Practices

## 1. Password Hashing

### ❌ خطأ:

```java
// لا تفعل هذا أبداً!
user.setPassword(request.getPassword()); // Plain text

// MD5 أو SHA-1 غير آمنة
String hashed = DigestUtils.md5Hex(password);
```

### ✅ صحيح:

```java
@Service
@RequiredArgsConstructor
public class PasswordService {

    private final PasswordEncoder passwordEncoder;

    // استخدم BCrypt مع strength مناسب
    public String hashPassword(String rawPassword) {
        return passwordEncoder.encode(rawPassword);
    }

    // التحقق
    public boolean verify(String rawPassword, String hashedPassword) {
        return passwordEncoder.matches(rawPassword, hashedPassword);
    }
}

// Configuration
@Bean
public PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder(12); // strength 10-12 جيد
}
```

### Password Policy:

```java
@Component
public class PasswordPolicyValidator {

    private static final int MIN_LENGTH = 12;
    private static final String PASSWORD_PATTERN =
        "^(?=.*[0-9])(?=.*[a-z])(?=.*[A-Z])(?=.*[@#$%^&+=])(?=\\S+$).{12,}$";

    public void validate(String password) {
        if (password == null || password.length() < MIN_LENGTH) {
            throw new IllegalArgumentException(
                "Password must be at least " + MIN_LENGTH + " characters"
            );
        }

        if (!password.matches(PASSWORD_PATTERN)) {
            throw new IllegalArgumentException(
                "Password must contain: " +
                "uppercase, lowercase, digit, special character"
            );
        }

        // التحقق من كلمات المرور الشائعة
        if (isCommonPassword(password)) {
            throw new IllegalArgumentException(
                "This password is too common. Please choose another."
            );
        }
    }

    private boolean isCommonPassword(String password) {
        List<String> commonPasswords = Arrays.asList(
            "password", "123456", "password123", "admin123",
            "qwerty", "letmein", "welcome"
        );
        return commonPasswords.contains(password.toLowerCase());
    }
}
```

---

## 2. HTTPS

### لماذا HTTPS؟

```
HTTP (غير آمن):
Browser ----[username:password]----> Server
         ↑ يمكن لأي شخص قراءته!

HTTPS (آمن):
Browser ----[encrypted data]----> Server
         ↑ مشفر، لا يمكن قراءته
```

### Spring Boot Configuration:

```properties
# application.properties

# تفعيل HTTPS
server.port=8443
server.ssl.enabled=true
server.ssl.key-store=classpath:keystore.p12
server.ssl.key-store-password=your-password
server.ssl.key-store-type=PKCS12
server.ssl.key-alias=tomcat

# إجبار HTTPS
server.ssl.enabled-protocols=TLSv1.2,TLSv1.3
```

### إجبار HTTPS في Spring Security:

```java
@Configuration
public class HttpsConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .requiresChannel(channel -> channel
                .anyRequest().requiresSecure() // إجبار HTTPS
            );

        return http.build();
    }

    // Redirect HTTP to HTTPS
    @Bean
    public TomcatServletWebServerFactory servletContainer() {
        TomcatServletWebServerFactory tomcat = new TomcatServletWebServerFactory() {
            @Override
            protected void postProcessContext(Context context) {
                SecurityConstraint securityConstraint = new SecurityConstraint();
                securityConstraint.setUserConstraint("CONFIDENTIAL");
                SecurityCollection collection = new SecurityCollection();
                collection.addPattern("/*");
                securityConstraint.addCollection(collection);
                context.addConstraint(securityConstraint);
            }
        };
        tomcat.addAdditionalTomcatConnectors(redirectConnector());
        return tomcat;
    }

    private Connector redirectConnector() {
        Connector connector = new Connector("org.apache.coyote.http11.Http11NioProtocol");
        connector.setScheme("http");
        connector.setPort(8080);
        connector.setSecure(false);
        connector.setRedirectPort(8443);
        return connector;
    }
}
```

---

## 3. CORS (Cross-Origin Resource Sharing)

### ما هو CORS؟

```
Same Origin (مسموح):
https://example.com/page1
https://example.com/page2
✓ نفس الـ protocol, domain, port

Cross Origin (ممنوع بدون CORS):
https://example.com → https://api.example.com
https://example.com → https://another.com
✗ مختلف الـ domain أو subdomain
```

### Spring Boot Configuration:

```java
@Configuration
public class CorsConfig {

    @Bean
    public WebMvcConfigurer corsConfigurer() {
        return new WebMvcConfigurer() {
            @Override
            public void addCorsMappings(CorsRegistry registry) {
                registry.addMapping("/api/**")
                    .allowedOrigins("https://frontend.com", "https://app.frontend.com")
                    .allowedMethods("GET", "POST", "PUT", "DELETE", "OPTIONS")
                    .allowedHeaders("*")
                    .allowCredentials(true)
                    .maxAge(3600);
            }
        };
    }
}
```

### CORS مع Spring Security:

```java
@Configuration
public class SecurityCorsConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .cors(cors -> cors.configurationSource(corsConfigurationSource()))
            .csrf(csrf -> csrf.disable());

        return http.build();
    }

    @Bean
    public CorsConfigurationSource corsConfigurationSource() {
        CorsConfiguration configuration = new CorsConfiguration();

        // السماح بـ origins محددة
        configuration.setAllowedOrigins(Arrays.asList(
            "https://frontend.com",
            "https://app.frontend.com"
        ));

        // أو السماح بكل شيء (للتطوير فقط!)
        // configuration.setAllowedOriginPatterns(Arrays.asList("*"));

        configuration.setAllowedMethods(Arrays.asList(
            "GET", "POST", "PUT", "DELETE", "OPTIONS"
        ));

        configuration.setAllowedHeaders(Arrays.asList(
            "Authorization", "Content-Type", "X-Requested-With"
        ));

        configuration.setAllowCredentials(true);
        configuration.setMaxAge(3600L);

        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/api/**", configuration);

        return source;
    }
}
```

### Per-Controller CORS:

```java
@RestController
@RequestMapping("/api/products")
@CrossOrigin(
    origins = "https://frontend.com",
    methods = {RequestMethod.GET, RequestMethod.POST},
    maxAge = 3600
)
public class ProductController {
    // ...
}
```

---

## 4. CSRF Protection

### ما هو CSRF؟

```
1. المستخدم يسجل دخول في bank.com
2. يزور موقع شرير evil.com
3. evil.com يحتوي على:
   <form action="https://bank.com/transfer" method="POST">
     <input name="amount" value="1000">
     <input name="to" value="attacker">
   </form>
   <script>document.forms[0].submit();</script>
4. الطلب يُرسل مع cookies الخاصة بـ bank.com
5. البنك ينفذ العملية! ❌
```

### الحماية من CSRF:

```java
@Configuration
public class CsrfSecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .csrf(csrf -> csrf
                // تفعيل CSRF
                .csrfTokenRepository(CookieCsrfTokenRepository.withHttpOnlyFalse())
                .csrfTokenRequestHandler(new CsrfTokenRequestAttributeHandler())
            );

        return http.build();
    }
}
```

### استثناء APIs من CSRF:

```java
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    http
        .csrf(csrf -> csrf
            .ignoringRequestMatchers("/api/**") // APIs لا تحتاج CSRF
        );

    return http.build();
}
```

### استخدام CSRF Token في Frontend:

```javascript
// JavaScript
fetch('/api/data', {
    method: 'POST',
    headers: {
        'Content-Type': 'application/json',
        'X-CSRF-TOKEN': getCsrfToken()
    },
    body: JSON.stringify(data)
});

function getCsrfToken() {
    return document.cookie
        .split('; ')
        .find(row => row.startsWith('XSRF-TOKEN='))
        ?.split('=')[1];
}
```

---

## 5. XSS Protection

### ما هو XSS؟

```
Stored XSS:
1. المهاجم ينشر تعليق: <script>steal_cookies()</script>
2. التعليق يُحفظ في قاعدة البيانات
3. كل من يشاهد التعليق يُنفذ الـ script ❌

Reflected XSS:
https://example.com/search?q=<script>alert('XSS')</script>
```

### الحماية:

#### 1. تفعيل Security Headers:

```java
@Configuration
public class XssSecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .headers(headers -> headers
                .xssProtection(xss -> xss
                    .headerValue(XXssProtectionHeaderWriter.HeaderValue.ENABLED_MODE_BLOCK)
                )
                .contentSecurityPolicy(csp -> csp
                    .policyDirectives(
                        "default-src 'self'; " +
                        "script-src 'self' 'unsafe-inline'; " +
                        "style-src 'self' 'unsafe-inline'; " +
                        "img-src 'self' data: https:;"
                    )
                )
                .frameOptions(frame -> frame.deny())
            );

        return http.build();
    }
}
```

#### 2. Input Validation:

```java
@Component
public class InputSanitizer {

    public String sanitize(String input) {
        if (input == null) {
            return null;
        }

        // إزالة HTML tags
        return input.replaceAll("<[^>]*>", "")
                   .replaceAll("&", "&amp;")
                   .replaceAll("<", "&lt;")
                   .replaceAll(">", "&gt;")
                   .replaceAll("\"", "&quot;")
                   .replaceAll("'", "&#x27;");
    }
}

@RestController
public class CommentController {

    @Autowired
    private InputSanitizer sanitizer;

    @PostMapping("/comments")
    public Comment addComment(@RequestBody CommentRequest request) {
        String safeContent = sanitizer.sanitize(request.getContent());

        Comment comment = new Comment();
        comment.setContent(safeContent);
        return commentRepository.save(comment);
    }
}
```

#### 3. استخدام مكتبات متخصصة:

```xml
<dependency>
    <groupId>org.owasp.encoder</groupId>
    <artifactId>encoder</artifactId>
    <version>1.2.3</version>
</dependency>
```

```java
import org.owasp.encoder.Encode;

@Service
public class SafeOutputService {

    public String prepareForHtml(String input) {
        return Encode.forHtml(input);
    }

    public String prepareForJavaScript(String input) {
        return Encode.forJavaScript(input);
    }

    public String prepareForUrl(String input) {
        return Encode.forUriComponent(input);
    }
}
```

---

## 6. SQL Injection Prevention

### ما هو SQL Injection؟

```java
// ❌ خطأ - SQL Injection vulnerable
String query = "SELECT * FROM users WHERE username = '" + username + "'";

// إذا كان username = "admin' OR '1'='1"
// الـ query يصبح:
// SELECT * FROM users WHERE username = 'admin' OR '1'='1'
// ✗ يسترجع جميع المستخدمين!
```

### الحماية:

#### 1. استخدم Prepared Statements:

```java
// ✅ صحيح - آمن من SQL Injection
@Repository
public interface UserRepository extends JpaRepository<User, Long> {

    // Spring Data JPA تستخدم Prepared Statements تلقائياً
    @Query("SELECT u FROM User u WHERE u.username = :username")
    Optional<User> findByUsername(@Param("username") String username);

    // أو
    Optional<User> findByUsername(String username);
}
```

#### 2. مع Native Queries:

```java
// ✅ صحيح
@Query(value = "SELECT * FROM users WHERE username = :username", nativeQuery = true)
Optional<User> findByUsernameNative(@Param("username") String username);

// ❌ خطأ - لا تفعل هذا
@Query(value = "SELECT * FROM users WHERE username = '" + username + "'", nativeQuery = true)
Optional<User> vulnerableQuery(String username);
```

#### 3. مع JDBC:

```java
@Service
public class UserJdbcService {

    @Autowired
    private JdbcTemplate jdbcTemplate;

    // ✅ صحيح
    public User findByUsername(String username) {
        String sql = "SELECT * FROM users WHERE username = ?";
        return jdbcTemplate.queryForObject(
            sql,
            new Object[]{username},
            new UserRowMapper()
        );
    }

    // ❌ خطأ - vulnerable
    public User vulnerableFind(String username) {
        String sql = "SELECT * FROM users WHERE username = '" + username + "'";
        return jdbcTemplate.queryForObject(sql, new UserRowMapper());
    }
}
```

#### 4. Input Validation:

```java
@Component
public class SqlInputValidator {

    private static final Pattern SQL_INJECTION_PATTERN =
        Pattern.compile(".*([';]|(--)|(\\*/)|(\\*/)).*");

    public void validate(String input) {
        if (input == null) {
            return;
        }

        if (SQL_INJECTION_PATTERN.matcher(input).matches()) {
            throw new IllegalArgumentException("Invalid input detected");
        }

        // التحقق من الكلمات المحجوزة
        String lower = input.toLowerCase();
        if (lower.contains("drop") ||
            lower.contains("delete") ||
            lower.contains("truncate") ||
            lower.contains("union")) {
            throw new IllegalArgumentException("SQL keywords not allowed");
        }
    }
}
```

---

## 7. Additional Security Practices

### Rate Limiting:

```java
@Component
public class RateLimitingFilter extends OncePerRequestFilter {

    private final Map<String, List<Long>> requestCounts = new ConcurrentHashMap<>();
    private static final int MAX_REQUESTS = 100;
    private static final long TIME_WINDOW = 60000; // 1 minute

    @Override
    protected void doFilterInternal(
            HttpServletRequest request,
            HttpServletResponse response,
            FilterChain filterChain
    ) throws ServletException, IOException {

        String clientIp = request.getRemoteAddr();
        long now = System.currentTimeMillis();

        requestCounts.putIfAbsent(clientIp, new ArrayList<>());
        List<Long> timestamps = requestCounts.get(clientIp);

        // إزالة الطلبات القديمة
        timestamps.removeIf(time -> now - time > TIME_WINDOW);

        if (timestamps.size() >= MAX_REQUESTS) {
            response.setStatus(HttpServletResponse.SC_TOO_MANY_REQUESTS);
            response.getWriter().write("Too many requests");
            return;
        }

        timestamps.add(now);
        filterChain.doFilter(request, response);
    }
}
```

### Secure Headers:

```java
@Configuration
public class SecurityHeadersConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .headers(headers -> headers
                // منع Clickjacking
                .frameOptions(frame -> frame.deny())

                // XSS Protection
                .xssProtection(xss -> xss
                    .headerValue(XXssProtectionHeaderWriter.HeaderValue.ENABLED_MODE_BLOCK)
                )

                // Content Security Policy
                .contentSecurityPolicy(csp -> csp
                    .policyDirectives("default-src 'self'")
                )

                // HSTS (HTTP Strict Transport Security)
                .httpStrictTransportSecurity(hsts -> hsts
                    .includeSubDomains(true)
                    .maxAgeInSeconds(31536000)
                )

                // منع Content Sniffing
                .contentTypeOptions(Customizer.withDefaults())

                // Referrer Policy
                .referrerPolicy(referrer -> referrer
                    .policy(ReferrerPolicyHeaderWriter.ReferrerPolicy.STRICT_ORIGIN_WHEN_CROSS_ORIGIN)
                )
            );

        return http.build();
    }
}
```

### Secure Session Configuration:

```java
@Configuration
public class SessionSecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .sessionManagement(session -> session
                .sessionCreationPolicy(SessionCreationPolicy.IF_REQUIRED)
                .invalidSessionUrl("/login?invalid=true")
                .maximumSessions(1)
                .maxSessionsPreventsLogin(true)
                .expiredUrl("/login?expired=true")
            );

        return http.build();
    }

    @Bean
    public HttpSessionEventPublisher httpSessionEventPublisher() {
        return new HttpSessionEventPublisher();
    }
}

// application.properties
server.servlet.session.timeout=30m
server.servlet.session.cookie.http-only=true
server.servlet.session.cookie.secure=true
server.servlet.session.cookie.same-site=strict
```

### Logging & Monitoring:

```java
@Component
@Slf4j
public class SecurityAuditLogger {

    @EventListener
    public void onAuthenticationSuccess(AuthenticationSuccessEvent event) {
        String username = event.getAuthentication().getName();
        log.info("Login successful: {}", username);
    }

    @EventListener
    public void onAuthenticationFailure(AbstractAuthenticationFailureEvent event) {
        String username = event.getAuthentication().getName();
        String error = event.getException().getMessage();
        log.warn("Login failed: {} - {}", username, error);
    }

    @EventListener
    public void onAccessDenied(AuthorizationDeniedEvent event) {
        String username = event.getAuthentication().get().getName();
        log.warn("Access denied: {}", username);
    }
}
```

---

## الخلاصة

### Checklist الأمان:

```
✅ استخدم HTTPS في الإنتاج
✅ استخدم BCrypt لتشفير كلمات المرور
✅ فعّل CSRF protection للـ web applications
✅ نظف الـ input لمنع XSS
✅ استخدم Prepared Statements لمنع SQL Injection
✅ فعّل CORS بشكل صحيح
✅ أضف Security Headers
✅ طبق Rate Limiting
✅ سجل الأحداث الأمنية
✅ حدّث الـ dependencies باستمرار
✅ استخدم JWT أو OAuth2 للـ APIs
✅ لا تخزن معلومات حساسة في الـ Token
✅ استخدم short-lived Access Tokens
✅ طبق Principle of Least Privilege
✅ اختبر الأمان بانتظام
```

---

تم إنشاء هذا الدليل الشامل لتغطية جميع جوانب الأمان في Spring Boot.
للأسئلة والاستفسارات، راجع الأمثلة والكود المرفق.