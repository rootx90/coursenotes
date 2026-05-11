---
title: Authentication and Authorization
date: 2026-05-10
tags:
  - authentication
  - authorization
  - access-control
  - broken-access-control
  - session-management
  - jwt
  - oauth
  - mfa
  - bug-bounty
  - pentest
  - web-security
---
#  Authentication 


---

## تنبيه مهم قبل ما نبدأ

المقال ده للتعلم فقط داخل:

- Labs.
- CTFs.
- بيئات تدريب.
- Bug Bounty programs مصرح لك تختبرها.
- Test accounts أنت عاملها بنفسك.

يعني ببساطة:

```text
اختبر بأمان، ومتلمسش بيانات ناس حقيقية، ومتعملش تعديل أو حذف في production.
```

في Bug Bounty، إثبات الثغرة الصح بيكون بأقل خطوة آمنة توضح المشكلة، مش بأكبر ضرر ممكن.

---

# 1. Introduction

خلينا نبدأها ببساطة يا صاحبي.

لو عندك تطبيق ويب، فأول سؤال مهم جدًا التطبيق لازم يجاوب عليه هو:

```text
الشخص اللي بيحاول يدخل ده مين؟
```

السؤال ده هو جوهر الـ **Authentication**.

يعني التطبيق بيتأكد من هوية المستخدم. هل هو فعلًا صاحب الحساب؟ هل الباسورد صح؟ هل الـ OTP صح؟ هل الـ session أو token اللي معاه صالح؟

تخيلها كأنك داخل مبنى مهم. أول حاجة عند الباب، الأمن يطلب منك إثبات هوية. البطاقة أو الكارنيه أو تصريح الدخول. لو إثبات الهوية صحيح، تدخل. لو غلط، تقف عند الباب.

في الويب، إثبات الهوية ممكن يكون بأكثر من طريقة:

```text
Email + Password
Phone + OTP
Magic Link
Login with Google
API Key
Session Cookie
Bearer Token
Certificate
Biometric login في تطبيق موبايل
```

ومن الطرق الشائعة كمان:

```text
OAuth 2.0 / Social Login
زي: Login with Google أو GitHub أو Facebook
```

هنا التطبيق بيخلي مزود خارجي موثوق يساعده في إثبات الهوية. مش هنغوص فيه في الجزء ده، لكن مهم تعرف اسمه لأنك هتشوفه كتير في التطبيقات الحديثة.

الهدف الأساسي من الـ Authentication إن التطبيق ما يسمحش لأي شخص مجهول يدخل كأنه مستخدم حقيقي.

## ليه الموضوع مهم في Web Security؟

أي خطأ في المصادقة ممكن يتحول لحاجة خطيرة جدًا، زي:

- Account Takeover.
- سرقة Session.
- دخول بدون كلمة مرور صحيحة.
- تخمين OTP.
- استخدام Reset Password بطريقة غلط.
- Tokens شغالة للأبد.
- Brute force على login.

المشكلة هنا إن المصادقة هي الباب الأول. لو الباب الأول اتكسر، كل حاجة بعده ممكن تتفتح.

## مثال بسيط

المستخدم يكتب:

```text
email = ahmed@example.com
password = Ahmed123!
```

السيرفر يعمل check:

```text
هل email موجود؟
هل password صح؟
هل الحساب مش مقفول؟
هل محتاج OTP؟
```

لو كله تمام، السيرفر يدي المستخدم إثبات إنه دخل بنجاح، زي:

```text
Session Cookie
أو
Access Token
```

ومن هنا يبدأ التطبيق يفتكر المستخدم في الطلبات الجاية.

## نصائح اختبار في Bug Bounty

- راجع كل مكان فيه إثبات هوية، مش صفحة login بس. شوف كمان: register, forgot password, reset password, verify OTP, resend OTP, change email, change password.
- استخدم test accounts فقط، واعمل حسابين أو أكثر عشان تقارن السلوك بينهم.
- لاحظ رسائل الخطأ. هل بتكشف إن الإيميل موجود؟ هل بتفرق بين “الباسورد غلط” و “المستخدم غير موجود”؟
- اختبر هل فيه Rate Limiting على المحاولات الكثيرة.
- جرب logout وبعدها استخدم نفس session أو token في طلب آمن. لو لسه شغال، دي ملاحظة مهمة.
- راجع هل التطبيق بيطلب إعادة إدخال الباسورد قبل العمليات الحساسة زي تغيير الإيميل أو تغيير الباسورد.

---

# 2. Authentication vs Authorization

رغم إن النسخة دي مركزة على الـ Authentication، لازم تعرف الفرق بسرعة عشان ما تخلطش بينهم وانت بتكتب report.

ببساطة:

```text
Authentication = أنت مين؟
Authorization  = مسموح لك تعمل إيه؟
```

مثال من الحياة:

```text
الأمن شاف الكارنيه وقال: أنت فعلًا أحمد.
ده Authentication.

بعدها أحمد حاول يدخل غرفة السيرفرات.
السيستم قال: أنت مش مسموح لك تدخل هنا.
ده Authorization.
```

في الويب:

```text
Login بالباسورد والـ OTP = Authentication
فتح صفحة admin أو مشاهدة بيانات مستخدم آخر = Authorization
```

رسم سريع يثبت الفرق في دماغك:

```text
┌─────────────────────────────────────────────────────────────────┐
│                    الفرق بين الاتنين                            │
├─────────────────────────────────────────────────────────────────┤
│  Authentication (المصادقة)    → من أنت؟                         │
│  Authorization  (التفويض)     → ماذا يسمح لك بفعله؟             │
│                                                                 │
│  مثال عملي:                                                     │
│  - Authentication: تسجيل الدخول باسم مستخدم وكلمة مرور          │
│  - Authorization: هل تستطيع الوصول لصفحة الإعدادات أم لا        │
└─────────────────────────────────────────────────────────────────┘
```

## جدول سريع للتثبيت

| النقطة         | Authentication                        | Authorization                          |
| -------------- | ------------------------------------- | -------------------------------------- |
| السؤال الأساسي | أنت مين؟                              | مسموح لك تعمل إيه؟                     |
| مثال           | Login, OTP, Session                   | Admin page, user roles, resource owner |
| فشلها غالبًا   | 401 أو login failed                   | 403 Forbidden                          |
| نوع الأخطاء    | Brute force, weak reset, token issues | Access control, IDOR, privilege issues |

> في الملف ده مش هنوسع في Authorization دلوقتي، بس لازم تكون فاهم الفرق عشان لما تلاقي bug تعرف تصنفه صح.

## مثال صغير جدًا بدون توسع

لو مستخدم مش عامل login حاول يفتح:

```http
GET /account HTTP/1.1
Host: example.com
```

المفروض الرد يكون حاجة زي:

```http
HTTP/1.1 401 Unauthorized
```

يعني التطبيق بيقول: أنا مش عارف أنت مين.

لكن لو مستخدم عامل login فعلًا، بس حاول يدخل مكان مش مسموح له، غالبًا الرد يكون:

```http
HTTP/1.1 403 Forbidden
```

يعني التطبيق عارفه، بس مش مديله الصلاحية.

## نصائح اختبار في Bug Bounty

- وانت بتكتب التقرير، فرق كويس بين المشكلة: هل التطبيق مش بيتأكد من هوية المستخدم؟ ولا بيتأكد من الهوية لكن مش بيتحقق من الصلاحية؟
- لو لقيت endpoint شغال من غير login، غالبًا دي Authentication issue.
- لو endpoint محتاج login لكن أي مستخدم يقدر يعمل action مش المفروض يعمله، دي Authorization issue، لكن مش موضوعنا هنا بالتفصيل.
- في اختبار المصادقة، ركز على bypass login, weak sessions, weak tokens, password reset, OTP, rate limits.
- استخدم status codes كإشارة، لكن متعتمدش عليها وحدها. أحيانًا التطبيق يرجع 200 وفي body يقول error.

---

# 3. Login Flow

الـ Login Flow هو الرحلة اللي بتحصل من أول ما المستخدم يكتب الإيميل والباسورد لحد ما التطبيق يقول له: أهلًا، أنت دخلت.

خلينا نمشيها خطوة خطوة:

```text
1. المستخدم يكتب email/password.
2. المتصفح يبعتهم للسيرفر.
3. السيرفر يتأكد من البيانات.
4. لو البيانات صح، السيرفر ينشئ Session أو Token.
5. المتصفح يخزن Session Cookie أو Token.
6. أي request بعد كده يبعت الإثبات ده.
7. السيرفر يعرف المستخدم من غير ما يطلب الباسورد كل مرة.
```

شكلها كـ Flow بسيط:

```text
User
  │ يكتب email/password
  ▼
Browser
  │ POST /login
  ▼
Server
  │ يتحقق من البيانات
  ▼
Session أو Token
  │ يرجع للمتصفح
  ▼
Next Requests
  │ Cookie أو Authorization Header
  ▼
Server يعرف المستخدم
```

## HTTP بطبيعته Stateless

نقطة مهمة جدًا:

```text
HTTP مش بيفتكر المستخدم لوحده.
```

يعني كل request مستقل. السيرفر محتاج وسيلة يعرف بيها إن الطلب الجديد ده جاي من نفس المستخدم اللي عمل login من شوية.

عشان كده بنستخدم:

```text
Session Cookie
أو
Bearer Token
```

من غيرهم، السيرفر هيشوف كل request كأنه جاي من شخص جديد.

## مثال Login Request

```http
POST /login HTTP/1.1
Host: example.com
Content-Type: application/json

{
  "email": "ahmed@example.com",
  "password": "Ahmed123!"
}
```

## مثال Response باستخدام Session

```http
HTTP/1.1 200 OK
Set-Cookie: session=abc123; HttpOnly; Secure; SameSite=Lax
Content-Type: application/json

{
  "message": "Logged in successfully"
}
```

بعد كده المتصفح يبعت الـ cookie تلقائيًا:

```http
GET /profile HTTP/1.1
Host: example.com
Cookie: session=abc123
```

## مثال Response باستخدام Token

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "access_token": "eyJhbGciOi...",
  "token_type": "Bearer"
}
```

بعدها العميل يبعت:

```http
GET /api/me HTTP/1.1
Host: example.com
Authorization: Bearer eyJhbGciOi...
```

## أخطاء شائعة في Login Flow

- رسائل خطأ تكشف هل الإيميل موجود.
- مفيش rate limit على محاولات login.
- قبول password ضعيف جدًا.
- session لا تتغير بعد login.
- login لا يحتاج MFA رغم إن العملية حساسة.
- token يرجع في URL بدل body أو header.
- التطبيق يسمح بتسجيل الدخول من حساب مقفول أو غير مؤكد.

## مثال User Enumeration

رد غير آمن:

```json
{
  "error": "User does not exist"
}
```

ورد آخر:

```json
{
  "error": "Wrong password"
}
```

كده التطبيق بيقول للمهاجم مين الإيميلات الموجودة.

الأفضل:

```json
{
  "error": "Invalid email or password"
}
```

## نصائح اختبار في Bug Bounty

- جرب login بإيميل موجود وباسورد غلط، وبإيميل غير موجود. قارن status code, response body, response time, headers.
- اختبر هل فيه rate limit بعد عدد محاولات كبير، لكن بدون إزعاج أو تخطي حدود البرنامج.
- شوف هل الرسائل بتكشف معلومات زي: user exists, account disabled, email not verified.
- جرّب بعد login هل session id بيتغير ولا هو نفسه قبل login. لو ما بيتغيرش، ممكن يكون فيه Session Fixation.
- جرب login ثم logout ثم أعد استخدام نفس cookie أو token في طلب بسيط مثل `/me`.
- راقب هل التطبيق يرسل credentials أو tokens في URL، لأن URL ممكن يتخزن في logs/history/referrer.
- راجع هل endpoint login يرجع معلومات زيادة عن المستخدم مثل role أو internal_id أو flags حساسة.

---

# 4. Session-Based Authentication

في النظام المعتمد على Sessions، السيرفر بيعمل session للمستخدم بعد login.

الفكرة ببساطة:

```text
المتصفح معاه Session ID صغير.
السيرفر عنده البيانات الحقيقية المرتبطة بالـ Session ID.
```

مثال:

```text
Browser Cookie:
session=abc123

Server Storage:
abc123 -> user_id=50, role=user, created_at=...
```

لما المتصفح يبعت cookie، السيرفر يدور على session id ويعرف المستخدم.

## تشبيه بسيط

تخيل إنك في مكان كبير واخدت كارت عليه رقم. الكارت نفسه مش فيه كل بياناتك، لكنه بيربطك بملف موجود عند الإدارة.

الـ session id هو الكارت.

والبيانات الحقيقية موجودة عند السيرفر.

## مميزات Sessions

- السيرفر يقدر يلغي session بسهولة.
- logout الحقيقي أسهل.
- مناسب للمواقع التقليدية.
- البيانات الحساسة مش لازم تتحط في المتصفح.
- ممكن تعمل device/session management.

## عيوب Sessions

- السيرفر محتاج يخزن sessions.
- في الأنظمة الكبيرة لازم حلول زي Redis أو shared session storage.
- لو session id اتسرق، المهاجم ممكن يستخدمه لحد ما الجلسة تنتهي أو تتلغي.

## Session ID لازم يكون قوي

مينفعش session id يكون:

```text
user-50
admin-1
12345
email-base64
timestamp
```

لازم يكون:

```text
طويل
عشوائي
غير قابل للتخمين
يتولد من secure random generator
```

## Session Rotation

بعد login، الأفضل إن السيرفر يغير session id.

ليه؟

عشان يمنع Session Fixation.

مثال المشكلة:

```text
1. المهاجم يثبت session id معين للضحية.
2. الضحية تعمل login بنفس session.
3. المهاجم يستخدم نفس session id ويدخل.
```

الحل:

```text
غيّر session id بعد login وبعد العمليات الحساسة.
```

## Logout الصحيح

Logout مش بس إنك تمسح cookie من المتصفح.

الصح:

```text
1. حذف session من السيرفر.
2. حذف cookie من المتصفح.
3. منع استخدام session القديمة مرة تانية.
```

## Timeout

لازم يكون فيه نوعين من timeout:

```text
Idle Timeout:
لو المستخدم سايب الحساب فترة بدون نشاط.

Absolute Timeout:
حتى لو نشط، الجلسة لازم تنتهي بعد مدة معينة.
```

## نصائح اختبار في Bug Bounty

- بعد logout، أعد إرسال request بنفس session cookie. لو اشتغل، logout مش بيلغي الجلسة فعليًا.
- سجّل قيمة session قبل login وبعد login. لو لم تتغير، دوّن احتمال Session Fixation.
- جرّب تستخدم نفس session من متصفح أو جهاز آخر. هل التطبيق يكتشف device change؟ مش دائمًا bug، لكنها ملاحظة حسب حساسية التطبيق.
- جرّب بعد تغيير الباسورد: هل sessions القديمة تفضل شغالة؟ في التطبيقات الحساسة، الأفضل يتم إبطالها.
- اختبر انتهاء الجلسة: هل session بتفضل شغالة لفترة طويلة جدًا؟
- راجع خصائص cookie المرتبطة بالـ session: HttpOnly, Secure, SameSite, Max-Age.
- تأكد إن session id مش ظاهر في URL أو response body أو logs واضحة.

---

# 5. Cookie Security

الـ Cookie هو مكان شائع لتخزين Session ID في المتصفح.

لو cookie بتاعة session اتسرقت، الشخص اللي سرقها ممكن يتصرف كأنه المستخدم، حسب حماية التطبيق.

عشان كده إعدادات الـ Cookie مهمة جدًا.

## أنواع Cookies بسرعة

```text
┌─────────────────────────────────────────────────────────────┐
│                    أنواع الـ Cookies                         │
├─────────────────────────────────────────────────────────────┤
│  Session Cookies                                            │
│  - مؤقتة غالبًا                                             │
│  - ممكن تتحذف لما تقفل المتصفح                              │
│                                                             │
│  Persistent Cookies                                         │
│  - بتفضل محفوظة لفترة محددة                                │
│  - لها Expires أو Max-Age                                  │
└─────────────────────────────────────────────────────────────┘
```

المهم هنا مش الاسم بس، المهم هل الكوكي شايلة session حساسة؟ لو آه، لازم تتأمن كويس.

## أهم خصائص Cookie

| الخاصية | معناها | أهميتها |
|---|---|---|
| HttpOnly | JavaScript ما يقرأش الـ cookie | يقلل سرقة session عبر XSS |
| Secure | تتبعت عبر HTTPS فقط | يمنع إرسالها على HTTP |
| SameSite | يقلل إرسالها من مواقع خارجية | يساعد ضد CSRF |
| Max-Age / Expires | مدة صلاحية cookie | يمنع sessions طويلة جدًا |
| Path | يحدد المسارات التي ترسل معها cookie | يقلل exposure |
| Domain | يحدد الدومين المسموح له يستقبل cookie | يمنع انتشارها على نطاقات غير لازمة |

## مثال Cookie جيدة

```http
Set-Cookie: session=abc123; HttpOnly; Secure; SameSite=Lax; Path=/; Max-Age=3600
```

مثال كود آمن للتوضيح:

```python
response.set_cookie(
    'session_id',
    session_id,
    httponly=True,      # يمنع JavaScript من قراءة الـ cookie
    secure=True,        # يرسل فقط عبر HTTPS
    samesite='Strict',  # يساعد ضد CSRF
    max_age=3600        # انتهاء بعد ساعة
)
```

مثال كود غير آمن:

```python
# ❌ غير آمن
response.set_cookie(
    'user_id',           # قيمة سهلة التوقع أو التلاعب
    str(user.id),
    httponly=False       # JavaScript يقدر يقرأها
)
```

## HttpOnly

دي بتمنع JavaScript من قراءة الـ cookie مباشرة.

مثال خطر لو مش موجودة:

```javascript
document.cookie
```

لو فيه XSS وcookie مش HttpOnly، ممكن تتسرق بسهولة.

لكن مهم تفهم:

```text
HttpOnly لا يمنع XSS نفسه.
```

هو فقط يقلل أثره في سرقة cookie.

## Secure

لو Secure مش موجودة، ممكن cookie تتبعت على HTTP لو الموقع أو جزء منه يسمح بده.

الأفضل:

```text
أي session cookie لازم Secure.
```

## SameSite

SameSite يساعد ضد CSRF.

القيم المشهورة:

```text
Strict
Lax
None
```

- `Strict`: أقوى، لكن ممكن يكسر بعض flows.
- `Lax`: مناسب لكثير من المواقع.
- `None`: لازم معها Secure، وغالبًا تستخدم مع cross-site flows.

## مثال Cookie غير آمنة

```http
Set-Cookie: session=abc123
```

المشكلة:

```text
لا HttpOnly
لا Secure
لا SameSite
لا Max-Age واضح
```

## نصائح اختبار في Bug Bounty

- افتح DevTools أو Proxy وشوف كل Set-Cookie headers.
- راجع هل session cookie فيها HttpOnly وSecure وSameSite.
- لو الموقع HTTPS لكن cookie بدون Secure، دي ملاحظة مهمة.
- لو session cookie بدون HttpOnly، ومع وجود XSS في نفس النطاق، impact يزيد جدًا.
- راجع هل cookie scope واسع جدًا مثل Domain=.example.com بدون داعي.
- شوف هل فيه cookies حساسة لها Max-Age طويل جدًا.
- راجع هل التطبيق يخزن user_id أو role أو email داخل cookie بشكل قابل للتعديل.
- لو cookie فيها بيانات، جرّب تفكها فقط لو كانت base64 أو JSON واضح، بدون تعديل مؤذي على production.

---

# 6. Token-Based Authentication

في تطبيقات APIs والموبايل، كثير بنشوف Token-Based Authentication.

بدل ما السيرفر يخزن session ويرسل session id في cookie، ممكن يرسل token للعميل.

التشبيه البسيط:

```text
الـ Token زي جواز سفر أو تذكرة مختومة.
كل مرة تروح للسيرفر، توريه التذكرة.
لو التذكرة صحيحة ولسه صالحة، يسمح لك تكمل.
```

## شكل request باستخدام Bearer Token

```http
GET /api/me HTTP/1.1
Host: example.com
Authorization: Bearer eyJhbGciOi...
```

## أنواع Tokens

### 1. Opaque Token

ده token عشوائي ملوش معنى ظاهر.

مثال:

```text
8f3a9c0b2d...
```

السيرفر لازم يرجع لقاعدة البيانات أو storage يعرف token ده تابع لمين.

### 2. JWT

Token فيه بيانات جواه وموقّع.

مثال شكلي:

```text
header.payload.signature
```

### 3. API Key

مفتاح يستخدمه مطور أو system عشان يدخل API.

مثال:

```http
Authorization: ApiKey abc123xyz
```

أو:

```http
X-API-Key: abc123xyz
```

## Access Token و Refresh Token

غالبًا الأنظمة الحديثة تستخدم:

```text
Access Token:
قصير العمر، يستخدم للوصول للـ API.

Refresh Token:
أطول عمرًا، يستخدم لإصدار access token جديد.
```

الفكرة:

```text
لو access token اتسرق، الضرر يكون محدود لأنه ينتهي بسرعة.
```

لكن refresh token لازم يتخزن بحذر شديد.

## أين نخزن الـ Token في الـ Frontend؟

دي نقطة صغيرة بس مهمة جدًا. تخزين الـ Token غلط ممكن يخلي XSS بسيط يتحول لـ Account Takeover.

| المكان | التقييم | السبب |
|---|---|---|
| localStorage | خطر مع XSS | JavaScript تقدر تقرأه وتسرقه |
| sessionStorage | أقل بقاءً، لكن برضه خطر مع XSS | JavaScript تقدر تقرأه |
| Memory / State | أفضل من التخزين الدائم | يضيع عند refresh أو إغلاق الصفحة |
| HttpOnly Cookie | قوي ضد سرقة JavaScript | يحتاج حماية CSRF وSameSite مضبوط |

الخلاصة:

```text
لو التوكن حساس، بلاش localStorage قدر الإمكان.
الأفضل حسب تصميم التطبيق: HttpOnly Cookie مضبوط أو تخزين مؤقت في Memory.
```

## مشاكل Token-Based Authentication

- token لا ينتهي.
- token يتخزن في localStorage مع وجود XSS.
- token يظهر في URL.
- refresh token لا يتم تدويره أو إبطاله.
- نفس token يفضل شغال بعد logout أو تغيير password.
- API keys بدون scopes أو expiry.

## لا تضع Token في URL

خطر:

```http
GET /api/me?token=abc123
```

ليه؟

لأن URL ممكن يظهر في:

```text
Browser history
Server logs
Proxy logs
Referer header
Screenshots
```

الأفضل:

```http
Authorization: Bearer abc123
```

## نصائح اختبار في Bug Bounty

- بعد logout، جرّب نفس access token في endpoint آمن مثل `/api/me`.
- بعد تغيير password، جرّب token القديم.
- راجع هل access token له expiry واضح، خصوصًا لو JWT.
- راقب هل token يظهر في URL أو logs أو response غير لازم.
- لو فيه refresh token، شوف هل بيتغير عند استخدامه ولا نفس الواحد يفضل شغال للأبد.
- اختبر هل refresh token القديم يفضل صالح بعد إصدار واحد جديد.
- راجع API keys: هل لها scopes؟ هل يمكن إلغاؤها؟ هل تظهر مرة واحدة فقط؟
- شوف مكان تخزين التوكن في المتصفح: localStorage, sessionStorage, memory, أو HttpOnly Cookie.
- لو التوكن في localStorage ومعاك XSS مصرح باختباره، اشرح إن XSS ممكن يسرق التوكن ويرفع الـ impact.
- لو التوكن في Cookie، راجع SameSite وCSRF protection لأن JavaScript مش لازم تقرأ الكوكي عشان تعمل requests باسم المستخدم.
- لا تعمل brute force على tokens في برامج حقيقية إلا لو البرنامج سامح بوضوح؛ غالبًا ده مرفوض.

---

# 7. JWT Security

JWT اختصار لـ:

```text
JSON Web Token
```

وشكله غالبًا:

```text
header.payload.signature
```

مثال شكلي:

```text
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiI1MCIsInJvbGUiOiJ1c2VyIn0.signature
```

## مكونات JWT

### Header

يحتوي معلومات عن نوع التوكن والخوارزمية:

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

### Payload

يحتوي claims أو بيانات:

```json
{
  "sub": "50",
  "email": "user@example.com",
  "role": "user",
  "exp": 1770000000
}
```

### Signature

التوقيع يثبت إن header و payload لم يتم تعديلهم.

## نقطة مهمة جدًا

```text
JWT payload غالبًا مش مشفر.
```

يعني أي شخص معه token يقدر يفك payload ويقرأ البيانات.

السرية مش في إن payload مخفي، السرية في إن التوقيع يمنع التعديل.

عشان كده ممنوع تحط بيانات حساسة في payload، مثل:

```text
password
password_hash
credit card
secret answers
private tokens
```

## Claims مهمة في JWT

| Claim | معناها |
|---|---|
| sub | هوية المستخدم |
| exp | وقت انتهاء التوكن |
| iat | وقت إصدار التوكن |
| nbf | لا يعمل قبل وقت معين |
| iss | الجهة التي أصدرت التوكن |
| aud | الجمهور أو الخدمة المقصودة |
| jti | معرف فريد للتوكن |

## أخطاء JWT شائعة

### 1. عدم التحقق من التوقيع

لو السيرفر يقرأ payload ويثق فيها بدون التحقق من signature، أي حد يقدر يغير البيانات.

### 2. قبول alg=none

بعض التطبيقات القديمة أو المكتبات المضبوطة غلط ممكن تقبل token بدون توقيع.

### 3. Secret ضعيف

لو HS256 مستخدم مع secret ضعيف مثل:

```text
secret
password
123456
jwtsecret
```

ده خطر جدًا.

### 4. عدم التحقق من exp

لو التوكن مفيهوش expiry أو السيرفر لا يتحقق منه، التوكن ممكن يفضل شغال للأبد.

### 5. تخزين بيانات حساسة في payload

لأن payload قابل للقراءة.

### 6. الاعتماد الكامل على role داخل JWT

لو role داخل JWT والتطبيق متحققش من التوقيع أو لا يراجع الصلاحيات من مصدر موثوق عند الحاجة، ممكن يحصل مشاكل خطيرة.

### 7. JWK / Key Injection كمعلومة متقدمة

أحيانًا بعض التطبيقات تسمح بوجود مفاتيح أو روابط مفاتيح داخل JWT header مثل:

```text
jwk
jku
kid
```

لو السيرفر بيتعامل مع القيم دي بدون قيود قوية، ممكن يفتح باب لتوقيع token مزيف أو استخدام مفتاح غير موثوق.

دي نقطة متقدمة، ومش كل التطبيقات معرضة ليها، لكنها مهمة تعرفها.

## مثال JWT جيد نسبيًا

```json
{
  "sub": "50",
  "iat": 1770000000,
  "exp": 1770000900,
  "iss": "https://example.com",
  "aud": "example-api",
  "jti": "random-unique-id"
}
```

مميزاته:

```text
فيه exp
فيه iss و aud
فيه jti
مفيهوش بيانات حساسة
```

## نصائح اختبار في Bug Bounty

- فك JWT واقرأ payload. هل فيه بيانات حساسة؟
- غيّر claim غير حساس في بيئة test وشوف هل السيرفر يرفض token بعد التعديل. المفروض يرفضه.
- راجع هل token فيه exp، وهل السيرفر يحترمه فعلًا.
- جرّب token بعد انتهاء وقته في test environment.
- راجع header: هل فيه alg=none؟ هل فيه kid أو jku أو jwk؟ دوّنها وافهم هل التطبيق يستخدمها بأمان.
- اختبر هل token القديم يفضل صالح بعد logout أو password change.
- لا تكسر secrets أو تعمل brute force على production إلا لو البرنامج سامح صراحة، وده نادر جدًا.
- في التقرير، ركز على الأثر: هل أدى ده لدخول غير مصرح؟ استمرار جلسة؟ كشف بيانات داخل payload؟

---

# 8. Password Security

الباسوردات من أهم أجزاء الـ Authentication.

الغلط الكبير جدًا:

```text
تخزين password كنص واضح.
```

يعني لو database اتسربت، كل كلمات المرور تتقرأ فورًا.

غلط برضه:

```text
تشفير password بطريقة قابلة للفك.
```

لأن لو مفتاح التشفير اتسرق، الباسوردات كلها تتفك.

الأفضل:

```text
Password Hashing + Salt
```

## Hashing vs Encryption

### Encryption

قابل للفك بمفتاح.

```text
password -> encryption -> encrypted password
encrypted password + key -> password
```

### Hashing

اتجاه واحد.

```text
password -> hash
```

المفروض ما ينفعش ترجع من hash للباسورد الأصلي.

## يعني إيه Salt؟

Salt قيمة عشوائية بتتضاف لكل باسورد قبل hashing.

ليه مهمة؟

لو شخصين عندهم نفس الباسورد:

```text
123456
```

من غير salt ممكن hash يكون نفسه.

لكن مع salt مختلف لكل مستخدم، hash يطلع مختلف.

ده يصعب هجمات زي Rainbow Tables.

## يعني إيه Pepper؟

Pepper قيمة سرية عامة للتطبيق، تتحفظ خارج قاعدة البيانات، مثل environment variable.

الفكرة:

```text
لو database اتسربت، المهاجم لسه ناقصه pepper.
```

لكن pepper لازم يتدار بحذر، ومش بديل عن bcrypt أو Argon2.

## خوارزميات مناسبة للباسورد

استخدم:

```text
Argon2
bcrypt
scrypt
```

تجنب:

```text
MD5
SHA1
SHA256 لوحده
```

## ليه SHA256 وحده مش كفاية؟

لأنه سريع جدًا.

والسرعة هنا مش ميزة، دي مشكلة.

المهاجم لو حصل على hashes، يقدر يجرب عدد ضخم من الباسوردات بسرعة عالية جدًا.

لكن bcrypt و Argon2 مصممين يكونوا أبطأ ومكلفين أكتر، فالهجوم يبقى أصعب.

## مثال غير آمن

```python
import hashlib

password = "Ahmed123!"
hash_value = hashlib.sha256(password.encode()).hexdigest()
```

ده أفضل من plain text، لكنه غير مناسب لتخزين كلمات المرور.

## مثال أفضل باستخدام bcrypt

```python
import bcrypt

password = b"Ahmed123!"

# إنشاء hash
hashed = bcrypt.hashpw(password, bcrypt.gensalt())

# التحقق عند login
if bcrypt.checkpw(password, hashed):
    print("Login successful")
else:
    print("Wrong password")
```

## Password Policy

سياسة كلمة المرور لازم تكون متوازنة.

مش لازم تعذب المستخدم بقواعد غريبة، لكن لازم تمنع الكوارث.

أمثلة سيئة لازم تتمنع:

```text
123456
password
qwerty
admin123
اسم المستخدم نفسه
الإيميل نفسه
```

الأفضل:

```text
طول مناسب
منع الكلمات الشائعة
السماح بالمسافات والجمل الطويلة
عدم وضع حد أقصى قصير جدًا
استخدام MFA للحسابات الحساسة
```

## Rate Limiting مهم مع Password

حتى لو التخزين ممتاز، login endpoint لازم يكون عليه حماية من التخمين.

أمثلة:

```text
تحديد عدد المحاولات لكل IP
تحديد عدد المحاولات لكل account
تأخير تدريجي بعد المحاولات الفاشلة
Captcha عند الاشتباه فقط
تنبيه المستخدم عند login غريب
```

## Password Reset

Password reset جزء حساس جدًا من Authentication.

Flow آمن بشكل مبسط:

```text
┌─────────────────────────────────────────────────────────────────┐
│              عملية إعادة تعيين كلمة المرور                     │
├─────────────────────────────────────────────────────────────────┤
│  1. المستخدم يطلب reset                                       │
│  2. السيرفر ينشئ token فريد وغير قابل للتخمين                 │
│  3. السيرفر يخزن hash للـ token وليس token الحقيقي             │
│  4. يرسل رابط reset على البريد الإلكتروني                      │
│  5. الرابط صالح لمدة قصيرة مثل 15 دقيقة                       │
│  6. المستخدم يختار كلمة مرور جديدة                            │
│  7. الـ token يُستخدم مرة واحدة فقط                           │
│  8. كل sessions/tokens القديمة تُلغى                           │
└─────────────────────────────────────────────────────────────────┘
```

مشاكل شائعة:

- reset token قصير أو قابل للتخمين.
- token لا ينتهي.
- token يستخدم أكثر من مرة.
- reset link غير مربوط بالمستخدم.
- تغيير الباسورد لا يبطل sessions القديمة.
- response يكشف هل الإيميل موجود.

الأفضل:

```text
Token طويل وعشوائي
ينتهي خلال وقت قصير
يستخدم مرة واحدة فقط
مرتبط بحساب محدد
تغيير الباسورد يبطل sessions/tokens القديمة
```

## Host Header Injection في Password Reset

دي ثغرة صغيرة لكن مهمة. بتحصل لما السيرفر يبني رابط إعادة تعيين كلمة المرور اعتمادًا على قيمة `Host` اللي جاية من المستخدم في الطلب.

مثال خطر:

```http
POST /forgot-password HTTP/1.1
Host: attacker.example
Content-Type: application/json

{
  "email": "victim@example.com"
}
```

لو السيرفر استخدم الـ Host ده في بناء اللينك، ممكن يبعت للضحية رابط زي:

```text
https://attacker.example/reset?token=REAL_RESET_TOKEN
```

وساعتها لو الضحية ضغطت، الـ token ممكن يوصل للمهاجم.

الأفضل إن التطبيق يستخدم دومين ثابت من إعدادات السيرفر، مش من Header المستخدم:

```text
APP_BASE_URL=https://app.example.com
```

## لا تكشف هل الإيميل موجود في Forgot Password

رد سيئ:

```json
{
  "message": "Email not found"
}
```

رد أفضل:

```json
{
  "message": "If this email exists, we sent reset instructions"
}
```

## نصائح اختبار في Bug Bounty

- في forgot password، جرّب إيميل موجود وغير موجود وقارن response والوقت والرسائل.
- افحص reset token: هل طويل وعشوائي؟ هل ينتهي؟ هل يستخدم مرة واحدة؟
- بعد استخدام reset link مرة، جرّبه مرة ثانية. المفروض يترفض.
- بعد تغيير الباسورد، جرّب session أو token قديم. الأفضل يتبطل خصوصًا في التطبيقات الحساسة.
- جرّب هل تغيير الباسورد يحتاج الباسورد القديم عندما المستخدم داخل بالفعل.
- راجع password policy: هل يسمح بكلمات شائعة جدًا؟ هل فيه حد أقصى قصير يمنع passphrases؟
- اختبر rate limit على login وforgot password وresend email بدون تجاوز قواعد البرنامج.
- راقب هل التطبيق يرسل password أو reset token في email أو response بطريقة مكشوفة.
- اختبر Host Header Injection بحذر: غيّر `Host` أو أضف `X-Forwarded-Host` وشوف هل رابط reset في الإيميل اتبنى على الدومين الغلط.
- لو البرنامج لا يسمح باستقبال إيميلات حقيقية، استخدم حسابك أنت فقط ولا تستخدم بريد أي شخص آخر.

---

# Summary

في الجزء ده ركزنا على الـ Authentication فقط، ومعاه الحاجات المرتبطة بيه:

```text
Login Flow
Sessions
Cookies
Tokens
JWT
Password Security
```

الخلاصة:

```text
Authentication = التطبيق يتأكد أنت مين.
```

وأهم قواعد الأمان:

- لا تكشف هل المستخدم موجود من رسائل الخطأ.
- استخدم sessions أو tokens بشكل آمن.
- session لازم تتلغي بعد logout.
- cookies لازم تكون HttpOnly وSecure وSameSite.
- tokens لازم يكون لها expiry.
- JWT payload مش مكان للبيانات الحساسة.
- passwords لازم تتخزن بـ Argon2 أو bcrypt أو scrypt.
- reset password لازم يكون قوي ومحدود ومرة واحدة.
- كل نقطة اختبار لازم تتم فقط في بيئة مصرح بها.

---

# Quick Practice Questions

جاوب على دول عشان تثبت المعلومة:

1. ليه HTTP محتاج Session أو Token؟
2. إيه الفرق بين 401 و 403؟
3. ليه HttpOnly مهم؟
4. ليه Secure مهم مع Cookie؟
5. هل JWT payload مشفر؟
6. إيه خطر token بدون exp؟
7. ليه SHA256 لوحده مش مناسب للباسورد؟
8. إيه مشكلة reset token يتستخدم أكثر من مرة؟
9. بعد logout، إيه الاختبار المهم تعمله؟
10. إيه الفرق بين Access Token و Refresh Token؟
11. ليه localStorage خطر مع وجود XSS؟
12. إيه فكرة Host Header Injection في Password Reset؟
