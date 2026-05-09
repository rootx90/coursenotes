---
title: Common Attacks
date: 2026-05-05
tags:
  - common-attacks
  - web-security
  - bug-bounty
  - pentest
  - owasp
---

# Common Attacks

[[Index|Back to Index]]

## ما معنى Common Attacks؟

Common Attacks هي الثغرات والهجمات المتكررة في تطبيقات الويب والتي تظهر كثيرا في bug bounty و pentest.

الفكرة ليست حفظ أسماء الثغرات فقط، بل فهم:

- أين تظهر الثغرة؟
- ما سببها؟
- كيف تختبرها بشكل آمن؟
- ما هو الـ impact؟
- كيف تكتب report واضح؟

> [!warning] تنبيه مهم
> طبق هذه الأمثلة فقط على أهداف داخل scope أو labs تدريبية مثل PortSwigger Academy, OWASP Juice Shop, DVWA, أو تطبيق تملكه. لا تستخرج بيانات حساسة ولا تغير بيانات حقيقية إلا إذا كانت قواعد البرنامج تسمح بذلك.

## خريطة سريعة لأشهر الثغرات

| الثغرة | الفكرة | Impact محتمل |
|---|---|---|
| XSS | تشغيل JavaScript داخل صفحة الضحية | سرقة session, account takeover, phishing |
| SQL Injection | إدخال يتحول إلى SQL query غير آمن | قراءة أو تعديل database |
| IDOR | الوصول إلى object لا يخصك بتغيير id | كشف بيانات مستخدمين آخرين |
| Broken Access Control | صلاحيات غير مطبقة بشكل صحيح | تنفيذ أفعال admin أو الوصول لبيانات ممنوعة |
| Authentication Bugs | مشاكل login, reset password, session | takeover أو bypass |
| CSRF | تنفيذ action باسم المستخدم بدون قصده | تغيير email, password, settings |
| SSRF | جعل السيرفر يرسل request لمكان داخلي أو خارجي | كشف internal services أو metadata |
| File Upload | رفع ملف خطر أو غير متوقع | RCE, stored XSS, overwrite |
| Path Traversal | قراءة ملفات خارج المسار المسموح | قراءة config أو secrets |
| Open Redirect | تحويل المستخدم إلى رابط خارجي | phishing أو bypass لبعض الحمايات |
| CORS Misconfiguration | السماح لمواقع خارجية بقراءة responses | تسريب بيانات من حساب الضحية |
| Information Disclosure | ظهور أسرار أو معلومات داخلية | تسهيل هجمات أخرى |
| Command Injection | إدخال يتحول إلى system command | تنفيذ أوامر على السيرفر |
| SSTI | إدخال يتحول إلى server-side template | قراءة secrets أو RCE |
| XXE | XML parser يقرأ external entities | قراءة ملفات أو SSRF |
| Rate Limit Bugs | عدم وجود حدود للتجارب المتكررة | brute force, OTP bypass, scraping |

## 1. XSS - Cross Site Scripting

XSS تحدث عندما يعرض الموقع input من المستخدم داخل الصفحة بدون تنظيف أو escaping كافي، فيتحول input إلى JavaScript يعمل في متصفح الضحية.

### أين تبحث عنها؟

- Search boxes.
- Comments.
- Profile name أو bio.
- Contact forms.
- URL parameters.
- Error messages.
- Admin panels التي تعرض بيانات المستخدمين.

### مثال بسيط

رابط بحث:

```text
https://example.com/search?q=ahmed
```

لو الموقع يعرض قيمة `q` داخل الصفحة كما هي، جرب payload آمن للتأكد:

```html
<script>alert(1)</script>
```

أو payload أقل إزعاجا:

```html
<img src=x onerror=alert(1)>
```

### أنواع XSS

| النوع | الشرح |
|---|---|
| Reflected XSS | payload في request ويرجع مباشرة في response |
| Stored XSS | payload يتم حفظه في database ويظهر لاحقا للمستخدمين |
| DOM XSS | JavaScript في المتصفح يعالج input بطريقة غير آمنة |

### متى تكون مهمة؟

تكون قوية عندما تستطيع تنفيذ JavaScript في سياق مستخدم آخر، خصوصا إذا كان admin أو موظف داخلي.

Impact أمثلة:

- سرقة tokens غير محمية.
- تنفيذ actions باسم الضحية.
- تغيير بيانات الحساب.
- phishing داخل نفس domain.

## 2. SQL Injection

SQL Injection تحدث عندما يدخل المستخدم قيمة يتم دمجها داخل SQL query بدون prepared statements أو validation صحيح.

### أين تبحث عنها؟

- Login forms.
- Search.
- Product filters.
- User id في URL.
- Sorting parameters.
- API endpoints التي تقبل أرقام أو strings.

### مثال

Request:

```text
GET /product?id=5 HTTP/1.1
Host: example.com
```

جرب علامات بسيطة:

```text
id=5'
id=5"
id=5 OR 1=1
```

لو ظهر error غريب أو تغيرت النتائج، قد يكون هناك SQL Injection.

### مثال Authentication Bypass في lab

```text
username=admin'--&password=test
```

الفكرة أن `'--` قد تغلق الشرط وتعلق باقي query في بعض قواعد البيانات.

> [!warning] مهم
> في bug bounty الحقيقي لا تسحب database كاملة ولا تستخدم automated dumping بدون تصريح. أثبت الثغرة بأقل دليل كاف مثل error واضح، اختلاف response، أو استخراج قيمة غير حساسة.

## 3. IDOR - Insecure Direct Object Reference

IDOR تحدث عندما يعتمد التطبيق على id من المستخدم بدون التأكد أن هذا المستخدم يملك الحق للوصول لهذا object.

### مثال

أنت داخل حسابك ورأيت request:

```text
GET /api/invoices/1001 HTTP/1.1
Host: example.com
Cookie: session=your_session
```

جرب تغيير الرقم:

```text
GET /api/invoices/1002 HTTP/1.1
```

لو ظهرت فاتورة مستخدم آخر، فهذا IDOR.

### أماكن شائعة

- Invoices.
- Orders.
- Tickets.
- Messages.
- User profiles.
- Uploaded files.
- API endpoints.

### Impact

- كشف بيانات شخصية.
- تحميل ملفات لا تخصك.
- تعديل أو حذف objects لمستخدمين آخرين.
- Account takeover أحيانا إذا كانت reset tokens أو email change flows متأثرة.

## 4. Broken Access Control

Broken Access Control أوسع من IDOR. معناها أن التطبيق لا يطبق الصلاحيات بشكل صحيح.

### أمثلة

مستخدم عادي يفتح صفحة admin:

```text
GET /admin/users HTTP/1.1
```

مستخدم عادي ينفذ action مخصص للـ admin:

```text
POST /api/admin/ban-user HTTP/1.1
Content-Type: application/json

{"user_id":123}
```

تغيير role داخل request:

```json
{
  "name": "ahmed",
  "role": "admin"
}
```

### كيف تختبر؟

- استخدم حسابين بصلاحيات مختلفة.
- جرب نفس request من حساب user وحساب admin.
- غير method من `GET` إلى `POST` أو العكس.
- احذف headers أو parameters خاصة بالصلاحيات.
- جرب direct API call بدل واجهة الموقع.

## 5. Authentication Bugs

Authentication Bugs هي مشاكل في login, registration, password reset, OTP, session management.

### أمثلة شائعة

| المشكلة | مثال |
|---|---|
| User enumeration | رسالة مختلفة عند email موجود وغير موجود |
| Weak password reset | token قصير أو قابل للتوقع |
| OTP بدون rate limit | تجربة أرقام كثيرة |
| Session لا تنتهي | session تبقى صالحة بعد logout أو تغيير password |
| 2FA bypass | endpoint حساس لا يطلب 2FA |

### مثال User Enumeration

```text
Forgot password:

test@example.com -> "Email sent"
notfound@example.com -> "User not found"
```

الفرق في الرسالة قد يكشف وجود الحساب.

### مثال OTP Rate Limit

```text
POST /verify-otp

{"otp":"000000"}
{"otp":"000001"}
{"otp":"000002"}
```

لو يمكن تجربة آلاف الأكواد بدون lockout أو rate limit، هذه مشكلة.

## 6. CSRF - Cross Site Request Forgery

CSRF تحدث عندما يستطيع موقع خارجي إرسال request باسم المستخدم لأن المتصفح يرسل cookies تلقائيا.

### متى تكون ممكنة؟

- التطبيق يعتمد على cookie session.
- action مهم لا يحتاج CSRF token.
- لا يوجد تحقق من `Origin` أو `Referer`.
- SameSite cookie غير مضبوط أو يسمح بالسيناريو.

### مثال action حساس

```text
POST /account/change-email
Content-Type: application/x-www-form-urlencoded

email=attacker@example.com
```

لو يمكن تنفيذ هذا الطلب من صفحة خارجية والضحية logged in، قد تكون CSRF.

### Impact

- تغيير email.
- تغيير password إذا لا يطلب password الحالي.
- إضافة payment method.
- تغيير settings.
- تنفيذ action إداري.

## 7. SSRF - Server Side Request Forgery

SSRF تحدث عندما تجعل السيرفر يرسل request إلى URL أنت تتحكم فيه.

### أين تظهر؟

- Import from URL.
- Webhook testing.
- Image fetcher.
- PDF generator.
- URL preview.
- File upload by URL.

### مثال

```json
{
  "image_url": "https://example.com/avatar.png"
}
```

جرب URL تابع لك أو lab:

```json
{
  "image_url": "https://your-collaborator.example/test"
}
```

لو وصلك request من سيرفر الموقع، فهذا دليل أن السيرفر يجلب الرابط.

### أهداف اختبار آمنة

- Domain تملكه.
- Burp Collaborator.
- Interactsh.
- Endpoint داخل lab.

> [!warning] مهم
> لا تفحص internal IP ranges أو cloud metadata على أهداف حقيقية إلا لو قواعد البرنامج تسمح بوضوح، لأن SSRF قد يكون عالي الخطورة.

## 8. File Upload Vulnerabilities

تحدث عندما يسمح الموقع برفع ملفات بدون تحقق كاف من النوع، الامتداد، المحتوى، أو مكان التخزين.

### أمثلة

رفع صورة لكن السيرفر يقبل HTML:

```html
<script>alert(1)</script>
```

رفع ملف باسم خطر:

```text
avatar.php
avatar.php.jpg
../../avatar.jpg
```

### نقاط فحص

- هل يمكن رفع امتداد غير مسموح؟
- هل يتم تنفيذ الملف أم تحميله فقط؟
- هل يظهر الملف داخل نفس domain؟
- هل توجد Content-Type صحيحة؟
- هل يمكن overwrite لملف موجود؟
- هل يمكن رفع SVG يحتوي JavaScript؟

### Impact

- Stored XSS.
- Remote Code Execution إذا تم تنفيذ server-side file.
- تسريب ملفات users.
- تغيير محتوى الموقع.

## 9. Path Traversal

Path Traversal تحدث عندما يستخدم التطبيق اسم ملف أو مسار من المستخدم بدون منع الرجوع خارج directory المسموح.

### مثال

```text
GET /download?file=report.pdf
```

تجربة:

```text
GET /download?file=../../../../etc/passwd
```

على Windows:

```text
GET /download?file=..\..\..\windows\win.ini
```

### أين تظهر؟

- Download endpoints.
- Image preview.
- Template loading.
- Language files.
- Backup download.

### Impact

- قراءة config files.
- قراءة source code.
- قراءة logs.
- كشف secrets أو database credentials.

## 10. Open Redirect

Open Redirect تحدث عندما يسمح الموقع بتحويل المستخدم إلى URL خارجي بدون validation صحيح.

### مثال

```text
https://example.com/redirect?next=https://evil.example
```

لو الموقع حولك مباشرة إلى `evil.example`، فهذا Open Redirect.

### أماكن شائعة

- Login redirect.
- Logout redirect.
- OAuth flows.
- Email links.
- SSO.

### Impact

Open Redirect وحدها غالبا impact ضعيف، لكنها تصبح أقوى عندما تستخدم في:

- phishing بسبب ثقة المستخدم في domain.
- OAuth token theft في بعض flows.
- bypass لقوائم allowlist غير قوية.

## 11. CORS Misconfiguration

CORS تتحكم في هل موقع خارجي يستطيع قراءة response من API أم لا.

### مثال Response خطر

```text
Access-Control-Allow-Origin: https://attacker.example
Access-Control-Allow-Credentials: true
```

لو API تحتوي بيانات حساسة وتقبل origin من attacker مع credentials، يمكن لموقع خارجي قراءة بيانات الضحية.

### كيف تختبر؟

أرسل header:

```text
Origin: https://attacker.example
```

راقب response headers:

```text
Access-Control-Allow-Origin
Access-Control-Allow-Credentials
```

### Impact

- قراءة بيانات حساب الضحية من API.
- تسريب معلومات شخصية.
- تسريب tokens إذا كانت تظهر في response.

## 12. Information Disclosure

Information Disclosure تعني ظهور معلومات لا يجب أن تظهر للمستخدم.

### أمثلة

- Stack trace.
- Debug mode.
- `.env`.
- API keys.
- Internal IPs.
- Source maps.
- Backup files.
- Git metadata.
- Error messages تكشف database أو framework.

### مثال

```text
GET /.env
GET /config.php.bak
GET /app.js.map
GET /.git/config
```

### متى تكون Finding؟

تكون finding عندما توجد معلومات لها impact واضح، مثل:

- credentials.
- private keys.
- tokens.
- internal endpoints تساعد على هجوم آخر.
- PII.

## 13. Command Injection

Command Injection تحدث عندما يدخل المستخدم قيمة يتم تمريرها إلى system command.

### أين تظهر؟

- Ping tools.
- DNS lookup tools.
- PDF أو image processing.
- Backup tools.
- Admin panels.

### مثال lab

```text
POST /ping

host=example.com
```

Payload اختبار:

```text
host=example.com; whoami
```

أو time-based:

```text
host=example.com; sleep 5
```

### Impact

- تنفيذ أوامر على السيرفر.
- قراءة ملفات.
- pivot داخل الشبكة.
- takeover حسب صلاحيات التطبيق.

> [!warning] مهم
> لا تنفذ أوامر تغير النظام أو تقرأ أسرار حقيقية. استخدم أوامر إثبات آمنة مثل `whoami` أو delay بسيط إذا كان مسموحا.

## 14. SSTI - Server Side Template Injection

SSTI تحدث عندما يدخل المستخدم قيمة يتم تفسيرها داخل template engine على السيرفر.

### مثال

لو صفحة تعرض:

```text
Hello Ahmed
```

وجربت:

```text
{{7*7}}
```

والنتيجة أصبحت:

```text
49
```

فقد يكون هناك SSTI.

### أين تظهر؟

- Email templates.
- Invoice templates.
- PDF generation.
- Profile fields.
- Custom pages.

### Impact

- قراءة variables داخل التطبيق.
- كشف secrets.
- تنفيذ أوامر في بعض template engines.

## 15. XXE - XML External Entity

XXE تحدث عندما يعالج السيرفر XML ويسمح بتحميل external entities.

### أين تظهر؟

- APIs تقبل XML.
- SOAP services.
- File import.
- SAML.
- Office أو SVG processing.

### مثال تعليمي

```xml
<?xml version="1.0"?>
<!DOCTYPE data [
  <!ENTITY test "hello">
]>
<data>&test;</data>
```

لو ظهر `hello` في response، فهذا يدل أن parser يفسر entities. الخطوة التالية في lab تكون اختبار قراءة ملف أو SSRF، لكن في bug bounty الحقيقي يجب الالتزام بقواعد البرنامج.

### Impact

- قراءة local files.
- SSRF.
- denial of service في بعض الحالات.

## 16. Rate Limiting Issues

Rate Limiting Issues تظهر عندما يسمح التطبيق بمحاولات كثيرة بدون حدود.

### أماكن شائعة

- Login.
- Forgot password.
- OTP.
- Coupon codes.
- Gift cards.
- Username search.
- Email verification.
- Invite links.

### أمثلة

تجربة passwords كثيرة:

```text
POST /login

email=victim@example.com&password=Password123
```

تجربة OTP:

```text
POST /verify

code=123456
```

### Impact

- brute force password أو OTP.
- credential stuffing.
- enumeration.
- abuse للخصومات أو coupons.

## Workflow عملي لاختبار Common Attacks

```text
حدد scope
  -> افهم الوظائف المهمة في التطبيق
  -> جهز حسابين أو أكثر بصلاحيات مختلفة
  -> راقب requests في Burp
  -> صنف endpoints حسب الوظيفة
  -> اختبر input validation
  -> اختبر authorization بين الحسابات
  -> اختبر auth/session flows
  -> اختبر الملفات والروابط والـ APIs
  -> اربط كل نتيجة ب impact واضح
  -> وثق بأقل دليل كاف
```

## كيف تكتب Report جيد؟

Report الجيد لا يقول فقط "وجدت XSS" أو "وجدت IDOR". يجب أن يوضح الطريق والضرر.

اكتب:

- Summary مختصر.
- Affected endpoint.
- Steps to reproduce.
- Request و response مهمين.
- Impact واضح.
- الحسابات المستخدمة في الاختبار.
- Evidence بدون كشف بيانات حساسة.
- Suggested fix.

### قالب بسيط

```text
Title:
IDOR allows user A to read invoices of user B

Endpoint:
GET /api/invoices/{id}

Steps:
1. Login as user A.
2. Open your invoice /api/invoices/1001.
3. Change id to 1002.
4. The response returns invoice data for user B.

Impact:
An attacker can access invoices belonging to other users, including names, billing addresses, and payment metadata.

Fix:
Validate object ownership on the server side before returning invoice data.
```

## نصائح مهمة في Bug Bounty

- لا تختبر خارج scope.
- استخدم حساباتك أنت قدر الإمكان.
- لا تسبب ضرر أو ضغط على الخدمة.
- لا تسحب بيانات أكثر من اللازم.
- افصل بين bug بدون impact و bug له impact حقيقي.
- لا تعتمد على scanner فقط.
- اقرأ program policy قبل أي اختبار حساس.
- وثق requests المهمة من Burp Repeater.
- اربط الثغرة بوظيفة business مهمة كلما أمكن.

## ملخص سريع

Common Attacks هي الأساس العملي لاختبار تطبيقات الويب. أهم شيء ليس اسم الثغرة، بل فهم أين تظهر، كيف تثبتها بأمان، وما الضرر الحقيقي منها.

أفضل بداية في bug bounty:

```text
XSS
SQL Injection
IDOR
Broken Access Control
Authentication Bugs
File Upload
Information Disclosure
Rate Limiting
```

هذه الثغرات تتكرر كثيرا، ومع التدريب على Burp وقراءة requests ستبدأ تلاحظ patterns بسرعة.
