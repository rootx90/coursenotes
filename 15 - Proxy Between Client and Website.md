---
title: Proxy Between Client and Website
date: 2026-04-28
tags:
  - proxy
  - web-security
  - networking
  - bug-bounty
  - pentest
  - burp-suite
---

# Proxy Between Client and Website

[[Index|Back to Index]]

## ما هو Proxy؟

Proxy هو وسيط يقف بين الـ client والـ website أو السيرفر. بدل ما المتصفح يرسل الطلب مباشرة إلى الموقع، يرسل الطلب إلى الـ proxy، والـ proxy يرسله للموقع ثم يرجع response إلى المتصفح.

الشكل العام:

```text
Client / Browser
  -> Proxy
  -> Website / Server
```

> [!summary] الفكرة ببساطة
> الـ proxy هو نقطة مرور للـ traffic. يمكنه مشاهدة، تعديل، تسجيل، فلترة، أو إعادة توجيه requests و responses حسب نوعه ووظيفته.

## كيف يكون بين الـ Website والـ Client؟

عندما تفتح موقعا مثل:

```text
https://example.com
```

بدون proxy:

```text
Browser -> example.com
```

مع proxy:

```text
Browser -> Proxy -> example.com
example.com -> Proxy -> Browser
```

في هذه الحالة، الـ proxy يرى الطلب الخارج من المتصفح والرد القادم من السيرفر.

## مثال عملي بسيط

لو المتصفح أرسل request:

```http
GET /login HTTP/1.1
Host: example.com
User-Agent: Mozilla/5.0
Cookie: session=abc123
```

الـ proxy يستقبل هذا الطلب أولا، ثم يمكنه:

- عرضه للمحلل الأمني.
- تعديله قبل إرساله.
- تسجيله في logs.
- منعه إذا كان يخالف policy.
- إعادة توجيهه إلى destination مختلف.

ثم يرجع response:

```http
HTTP/1.1 200 OK
Content-Type: text/html
```

## لماذا Proxy مهم في Bug Bounty؟

في Bug Bounty و Web Pentest، الـ proxy مهم لأنه يسمح لك تفهم التطبيق من الداخل أثناء التصفح.

فوائده:

- مشاهدة كل HTTP requests و responses.
- تعديل parameters و headers و cookies.
- إعادة إرسال الطلب أكثر من مرة.
- اختبار access control.
- اختبار IDOR.
- اختبار XSS, SQLi, SSRF, Open Redirect.
- تحليل API traffic.
- استخراج endpoints من تطبيقات الويب.
- فهم authentication flow.
- حفظ history كامل للتصفح.

أشهر مثال في الاختبار الأمني:

```text
Browser -> Burp Suite Proxy -> Target Website
```

## أنواع Proxy

## 1. Forward Proxy

Forward Proxy يكون قريب من الـ client ويمثل المستخدم أمام الإنترنت.

الشكل:

```text
Client -> Forward Proxy -> Website
```

استخداماته:

- إخفاء IP الحقيقي للمستخدم.
- التحكم في المواقع المسموحة داخل شركة.
- فلترة traffic.
- تسجيل requests.
- تجاوز بعض القيود الشبكية، حسب السياسة والتصريح.

مثال:

```text
موظف داخل شركة -> Proxy الشركة -> الإنترنت
```

## 2. Reverse Proxy

Reverse Proxy يكون قريب من السيرفر ويمثل الموقع أمام المستخدمين.

الشكل:

```text
Client -> Reverse Proxy -> Web Server
```

استخداماته:

- Load balancing.
- حماية السيرفر الأصلي.
- TLS termination.
- Caching.
- WAF filtering.
- إخفاء origin IP.
- توزيع الطلبات على أكثر من backend.

أمثلة مشهورة:

- Nginx.
- HAProxy.
- Cloudflare.
- AWS ALB.
- Fastly.

في هذه الحالة المستخدم لا يتعامل مباشرة مع السيرفر الأصلي، بل يتعامل مع reverse proxy.

## 3. Transparent Proxy

Transparent Proxy يعمل بدون أن يغير المستخدم إعدادات المتصفح غالبا. الشبكة تمرر traffic من خلاله تلقائيا.

استخداماته:

- مراقبة أو فلترة traffic داخل شبكة.
- Caching في مزودي الإنترنت أو الشركات.
- تطبيق سياسات أمنية.

ملاحظة:

```text
المستخدم قد لا يعرف أن traffic يمر عبر proxy.
```

## 4. Intercepting Proxy

Intercepting Proxy هو proxy يسمح باعتراض request قبل وصوله للموقع، وتعديلها يدويا ثم إرسالها.

أشهر مثال:

```text
Burp Suite
OWASP ZAP
```

استخداماته في bug bounty:

- تعديل parameters.
- تعديل cookies.
- تعديل headers.
- تغيير HTTP method.
- حذف أو إضافة fields.
- إعادة إرسال الطلب إلى Repeater.
- تجربة payloads بشكل يدوي.

مثال:

```text
GET /user?id=1001
```

يمكن تغييره إلى:

```text
GET /user?id=1002
```

لاختبار IDOR إذا كان ذلك داخل scope ومصرحا به.

## 5. SOCKS Proxy

SOCKS Proxy يعمل على مستوى أقل من HTTP، ويمكنه تمرير أنواع مختلفة من traffic، ليس الويب فقط.

أنواعه المشهورة:

```text
SOCKS4
SOCKS5
```

استخداماته:

- تمرير TCP traffic.
- استخدامه مع أدوات command line.
- Pivoting داخل بيئات مصرح بها.
- دعم بروتوكولات مختلفة غير HTTP.

## 6. HTTP Proxy

HTTP Proxy مصمم للتعامل مع HTTP/HTTPS traffic.

استخداماته:

- تصفح الويب عبر proxy.
- اختبار web requests.
- تعديل headers.
- logging و filtering.

مثال إعداد متصفح مع proxy:

```text
HTTP Proxy: 127.0.0.1
Port: 8080
```

هذا الإعداد شائع عند استخدام Burp Suite.

## Proxy و HTTPS

مع HTTPS، الاتصال يكون مشفرا بين المتصفح والموقع. حتى يستطيع proxy مثل Burp Suite قراءة traffic، يجب تثبيت CA certificate الخاص به في المتصفح.

بدون certificate:

```text
Proxy يرى الاتصال لكنه لا يستطيع قراءة المحتوى المشفر بشكل واضح.
```

مع certificate:

```text
Browser يثق في Burp CA
  -> Burp يفك التشفير محليا
  -> يرسل الطلب للموقع عبر HTTPS
```

> [!warning] مهم
> لا تثبت certificate لأي proxy غير موثوق. الـ proxy الذي يملك certificate موثوقا في جهازك يمكنه قراءة HTTPS traffic الخاص بك.

## كيف يتعامل Burp Suite مع Proxy؟

Burp Suite يعمل كـ Intercepting Proxy محلي على جهازك. غالبا يكون مستمعا على:

```text
127.0.0.1:8080
```

معنى ذلك أن Burp يفتح proxy listener على جهازك، ثم تضبط المتصفح حتى يرسل HTTP و HTTPS traffic إلى هذا العنوان بدلا من إرسال الطلب مباشرة إلى الموقع.

الشكل:

```text
Browser
  -> 127.0.0.1:8080
  -> Burp Suite Proxy
  -> Target Website
```

ثم الرد يرجع بنفس الطريق:

```text
Target Website
  -> Burp Suite Proxy
  -> Browser
```

## كيف يصل request إلى Burp؟

الخطوات ببساطة:

1. تفتح Burp Suite.
2. Burp يشغل proxy listener على `127.0.0.1:8080`.
3. تضبط المتصفح على استخدام هذا proxy.
4. تفتح الموقع من المتصفح.
5. المتصفح لا يرسل request مباشرة للموقع.
6. المتصفح يرسل request أولا إلى Burp.
7. Burp يعرض request في تبويب Proxy.
8. إذا كان Intercept مفعلا، Burp يوقف request حتى تضغط Forward.
9. بعد Forward، Burp يرسل request إلى الموقع.
10. الموقع يرد على Burp.
11. Burp يرجع response إلى المتصفح.

مثال request يظهر داخل Burp:

```http
GET /login HTTP/1.1
Host: example.com
Cookie: session=abc123
User-Agent: Mozilla/5.0
```

يمكنك داخل Burp أن تعدل request قبل إرساله:

```http
GET /admin HTTP/1.1
Host: example.com
Cookie: session=abc123
User-Agent: Mozilla/5.0
```

ثم تضغط:

```text
Forward
```

فيرسل Burp النسخة المعدلة إلى الموقع.

## إعداد Burp Suite مع المتصفح

الإعداد المعتاد:

```text
Proxy Host: 127.0.0.1
Proxy Port: 8080
Protocol: HTTP and HTTPS
```

يمكن ضبطه بطريقتين:

- من إعدادات proxy داخل المتصفح.
- باستخدام إضافة مثل FoxyProxy لتبديل proxy بسرعة.

في Burp:

```text
Proxy -> Proxy settings -> Proxy listeners
```

تتأكد أن listener يعمل على:

```text
127.0.0.1:8080
```

## ما معنى Intercept is on؟

عندما يكون:

```text
Intercept is on
```

Burp يوقف requests قبل إرسالها للموقع. هذا يسمح لك بمراجعة الطلب أو تعديله.

مثال:

```text
Browser طلب /profile
  -> Burp يوقف الطلب
  -> أنت تعدل id أو header أو cookie
  -> تضغط Forward
  -> Burp يرسل الطلب للموقع
```

عندما يكون:

```text
Intercept is off
```

Burp لا يوقف الطلبات، لكنه ما زال يسجلها في:

```text
Proxy -> HTTP history
```

## أين تظهر الطلبات داخل Burp؟

أهم أماكن متابعة الطلبات:

| المكان | الاستخدام |
|---|---|
| `Proxy -> Intercept` | عرض الطلب الحالي وإيقافه قبل الإرسال |
| `Proxy -> HTTP history` | سجل كل requests و responses |
| `Repeater` | إعادة إرسال request يدويا مع تعديلات |

أفضل workflow للمبتدئ:

```text
Proxy history
  -> اختر request مهم
  -> Send to Repeater
  -> عدل parameter أو header
  -> Send
  -> قارن response
```

## كيف يتعامل Burp مع HTTPS؟

مع HTTPS، المتصفح يتوقع اتصالا مشفرا وموثوقا مع الموقع. لذلك Burp يحتاج CA certificate حتى يستطيع عرض request و response بشكل واضح.

بدون Burp CA:

```text
Browser -> Burp -> Website
```

لكن المتصفح سيظهر certificate error أو لن يسمح بعرض traffic بشكل صحيح.

بعد تثبيت Burp CA:

```text
Browser يثق في Burp
  -> Burp يستقبل HTTPS traffic
  -> Burp يعرض request/response لك
  -> Burp ينشئ اتصال HTTPS آخر مع الموقع الحقيقي
```

هذا لا يعني أن Burp يكسر HTTPS عشوائيا، بل لأنك أنت جعلت المتصفح يثق في شهادة Burp أثناء الاختبار.

## مثال Bug Bounty باستخدام Burp

طلب عادي:

```http
GET /api/user?id=1001 HTTP/1.1
Host: example.com
Cookie: session=your_session
```

داخل Burp Repeater، تغير `id`:

```http
GET /api/user?id=1002 HTTP/1.1
Host: example.com
Cookie: session=your_session
```

ثم تقارن response.

إذا ظهر لك بيانات مستخدم آخر بدون صلاحية، قد يكون هذا IDOR. لكن يجب أن يكون الاختبار داخل scope وبحسابات اختبار مصرح بها.

## Burp Suite Proxy Setup Checklist

خطوات تشغيل Burp مع المتصفح بشكل كامل:

```text
1. افتح Burp Suite
2. ادخل إلى Proxy -> Proxy settings
3. تأكد أن Proxy listener يعمل على 127.0.0.1:8080
4. افتح المتصفح الخاص بالاختبار
5. اضبط proxy في المتصفح على 127.0.0.1 port 8080
6. افتح http://burp من المتصفح
7. حمل Burp CA certificate
8. ثبت certificate داخل المتصفح
9. افتح Proxy -> Intercept
10. اجعل Intercept is on للتجربة
11. افتح الموقع وشاهد request داخل Burp
```

بعد هذه الخطوات، أي request من المتصفح سيمر أولا على Burp.

## تحميل وتثبيت Burp CA Certificate

بعد ضبط المتصفح على proxy، افتح:

```text
http://burp
```

ثم حمل certificate من صفحة Burp.

الفكرة:

```text
Browser يرسل HTTPS request
  -> Burp يحتاج أن يكون موثوقا داخل المتصفح
  -> تثبت Burp CA certificate
  -> المتصفح يسمح لـ Burp بعرض HTTPS traffic
```

في Firefox غالبا التثبيت يكون من:

```text
Settings
  -> Privacy & Security
  -> Certificates
  -> View Certificates
  -> Authorities
  -> Import
```

ثم تختار Burp certificate وتفعل الثقة لاستخدامه مع websites.

> [!warning] مهم جدا
> ثبت Burp CA فقط في browser profile مخصص للاختبار. لا تستخدم نفس المتصفح لحساباتك الشخصية أثناء تشغيل proxy.

## ماذا يحدث داخل Burp خطوة بخطوة؟

مثال عند فتح صفحة login:

```text
Browser يطلب https://example.com/login
  -> إعدادات المتصفح ترسل الطلب إلى 127.0.0.1:8080
  -> Burp يستقبل الطلب
  -> Burp يعرضه في Proxy -> Intercept إذا كان intercept on
  -> أنت تراجع request
  -> تضغط Forward
  -> Burp يرسل request إلى example.com
  -> example.com يرد على Burp
  -> Burp يعرض response أو يسجله
  -> Burp يرجع response إلى Browser
  -> الصفحة تظهر للمستخدم
```

هذا هو سبب ظهور requests في Burp. المتصفح نفسه هو الذي يرسلها إلى Burp بسبب proxy settings.

## التعامل مع Request داخل Burp

عندما يظهر request في Burp، يمكنك تعديل أجزاء كثيرة:

| الجزء | مثال | لماذا نعدله؟ |
|---|---|---|
| Method | `GET` إلى `POST` | اختبار قبول method مختلف |
| Path | `/profile` إلى `/admin` | اختبار paths أو access |
| Query parameter | `id=1` إلى `id=2` | اختبار IDOR أو logic |
| Header | `X-Forwarded-For` | اختبار rate limit أو IP logic |
| Cookie | `role=user` | اختبار session أو authorization |
| Body | JSON أو form data | اختبار input validation |

مثال request قبل التعديل:

```http
POST /api/profile HTTP/1.1
Host: example.com
Content-Type: application/json
Cookie: session=abc123

{"name":"ahmed","role":"user"}
```

مثال بعد التعديل:

```http
POST /api/profile HTTP/1.1
Host: example.com
Content-Type: application/json
Cookie: session=abc123

{"name":"ahmed","role":"admin"}
```

بعد ذلك تقارن response. لو التطبيق قبل قيمة لا يجب قبولها، قد تكون هناك ثغرة business logic أو access control.

## ملخص حركة Burp Proxy

```text
Browser proxy settings = 127.0.0.1:8080

Browser
  -> يرسل request إلى Burp
  -> Burp يعرض request
  -> Burp يسمح بالتعديل أو Forward
  -> Burp يرسل request إلى Website
  -> Website يرد على Burp
  -> Burp يسجل response
  -> Burp يرجع response إلى Browser
```

> [!summary] Burp Proxy
> Burp لا يحصل على requests وحده. المتصفح يرسلها له لأنك ضبطت proxy settings على `127.0.0.1:8080`. Burp يصبح الوسيط بينك وبين الموقع، لذلك يستطيع عرض الطلبات، تعديلها، إرسالها، وتسجيل الردود.

## Proxy vs VPN

| المقارنة | Proxy | VPN |
|---|---|---|
| المستوى | غالبا تطبيق أو بروتوكول محدد | غالبا كل traffic الجهاز |
| الاستخدام | Web testing, filtering, routing | إخفاء/تغيير مسار اتصال الجهاز |
| التحكم في requests | عالي مع أدوات مثل Burp | عادة أقل |
| مناسب لـ Bug Bounty | نعم، خصوصا intercepting proxy | مفيد أحيانا لكنه ليس بديلا عن Burp |

## Proxy Headers مهمة

أحيانا الـ proxy يضيف headers تكشف معلومات عن IP الأصلي أو مسار الطلب.

أمثلة:

```http
X-Forwarded-For: 192.0.2.10
X-Real-IP: 192.0.2.10
Forwarded: for=192.0.2.10;proto=https
Via: 1.1 proxy
```

في الاختبار الأمني، هذه headers قد تكون مهمة في:

- Rate limit bypass.
- IP-based access control testing.
- Logging analysis.
- معرفة وجود reverse proxy.
- فهم routing داخل التطبيق.

> [!warning] تنبيه
> اختبار bypass للـ rate limit أو access control يجب أن يكون داخل scope وبطريقة لا تسبب ضررا أو ضغطا على الخدمة.

## كيف تعرف أن الموقع خلف Reverse Proxy؟

مؤشرات ممكنة:

- Headers مثل `server: cloudflare`.
- وجود `cf-ray` أو `x-cache`.
- اختلاف IP عن expected origin.
- redirects أو TLS certificate من provider معروف.
- Response موحد لصفحات الخطأ.
- WAF challenge أو CAPTCHA.

أوامر مفيدة:

```bash
curl -I https://example.com
```

```bash
whatweb https://example.com
```

## استخدام Proxy في Workflow bug bounty

```text
اضبط المتصفح على Burp/ZAP
  -> ثبت CA certificate للاختبار
  -> تصفح التطبيق بشكل طبيعي
  -> راقب HTTP history
  -> أرسل الطلبات المهمة إلى Repeater
  -> عدل parameters و headers
  -> اختبر authorization و input validation
  -> وثق request/response كدليل
```

## أخطاء شائعة

- نسيان إيقاف intercept في Burp.
- عدم تثبيت CA certificate ثم الاعتقاد أن HTTPS لا يعمل.
- اختبار مواقع خارج scope لأن المتصفح يمر كله عبر proxy.
- إرسال traffic حساس شخصي عبر proxy اختبار.
- الاعتماد على proxy فقط بدون فهم request/response.
- تعديل headers عشوائيا بدون هدف واضح.
- تجاهل أن reverse proxy أو WAF قد يغير response.

## ملاحظات أمنية

- استخدم proxy فقط على أهداف مصرح بها.
- لا تمرر حساباتك الشخصية أو بياناتك الحساسة عبر proxy غير موثوق.
- افصل browser profile الخاص بالاختبار عن استخدامك اليومي.
- احفظ request/response المهمة كدليل، لكن لا تحفظ أسرار أو بيانات حساسة أكثر من اللازم.
- اقرأ rules الخاصة ببرنامج bug bounty قبل أي اختبار نشط.

## الخلاصة

> [!summary] Proxy
> الـ proxy هو وسيط بين client و website. في الاختبار الأمني يساعدك تشاهد وتعدل requests و responses، وفي البنية الخلفية قد يستخدم كـ reverse proxy للحماية والـ load balancing والـ caching. فهم أنواع proxy مهم جدا لفهم حركة الويب، اختبار التطبيقات، وتحليل سلوك السيرفرات في bug bounty.
