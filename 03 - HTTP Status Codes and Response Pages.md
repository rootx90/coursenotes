---
title: HTTP Status Codes and Response Pages
date: 2026-04-26
tags:
  - http
  - status-codes
  - response
  - bug-bounty
  - pentest
---

# HTTP Status Codes and Response Pages

[[Index|Back to Index]]

## ما هو HTTP Response؟

عندما يرسل المتصفح أو الأداة Request للسيرفر، السيرفر يرد بـ Response.

الـ response غالبا يحتوي على:

- Status code.
- Headers.
- Body أو page content.
- Cookies أحيانا.
- Redirect location أحيانا.

مثال بسيط:

```text
HTTP/1.1 200 OK
Content-Type: text/html
Set-Cookie: session=abc123

<html>Page content</html>
```

## ما هو Status Code؟

Status code هو رقم يوضح نتيجة الطلب.

أمثلة:

- `200`: الطلب نجح.
- `301`: تحويل دائم.
- `403`: ممنوع.
- `404`: غير موجود.
- `500`: خطأ في السيرفر.

> [!summary] الفكرة الأساسية
> الـ status code يعطيك إشارة سريعة، لكن لا تعتمد عليه وحده. لازم تقارن معه حجم الصفحة، المحتوى، headers، وredirects.

## عائلات Status Codes

| العائلة | المعنى | أمثلة |
|---|---|---|
| `1xx` | Informational | `100`, `101` |
| `2xx` | Success | `200`, `201`, `204` |
| `3xx` | Redirection | `301`, `302`, `304` |
| `4xx` | Client Error | `400`, `401`, `403`, `404`, `429` |
| `5xx` | Server Error | `500`, `502`, `503`, `504` |

## 2xx - Success

أكواد `2xx` تعني أن الطلب نجح أو تم قبوله.

### 200 OK

يعني أن الطلب نجح والسيرفر رجع response عادي.

في bug bounty:

- قد يعني أن path موجود.
- قد يعني أن endpoint يعمل.
- لا يعني بالضرورة وجود ثغرة.
- مهم أثناء fuzzing لأنه قد يكشف pages أو files.

### 201 Created

يعني أنه تم إنشاء resource جديد.

مثال:

- إنشاء user.
- رفع ملف.
- إنشاء object عبر API.

في pentest:

- مهم في APIs.
- قد يساعدك تفهم صلاحيات الإنشاء.

### 204 No Content

يعني أن الطلب نجح لكن لا يوجد body في الرد.

مثال:

- حذف resource.
- تحديث بدون إرجاع محتوى.

## 3xx - Redirection

أكواد `3xx` تعني أن السيرفر يحولك لمكان آخر.

### 301 Moved Permanently

تحويل دائم.

مثال:

```text
http://example.com -> https://example.com
```

### 302 Found

تحويل مؤقت.

يستخدم كثيرا في:

- Login redirects.
- Logout.
- تحويل المستخدم بعد action معين.

### 304 Not Modified

يعني أن النسخة الموجودة عند المتصفح لم تتغير، ويستخدم مع caching.

## 4xx - Client Error

أكواد `4xx` تعني غالبا أن المشكلة من الطلب أو صلاحيات المستخدم.

### 400 Bad Request

الطلب غير صحيح أو ناقص.

أمثلة:

- JSON غير صحيح.
- parameter مفقود.
- header مطلوب غير موجود.

### 401 Unauthorized

يعني أنك تحتاج authentication.

مهم لأنه يدل أن endpoint موجود لكنه يحتاج تسجيل دخول أو token.

### 403 Forbidden

يعني أن السيرفر فهم الطلب لكن يمنع الوصول.

في bug bounty:

- مهم جدا أثناء fuzzing.
- قد يعني أن path موجود لكنه محمي.
- يمكن أن يشير إلى admin panel أو file موجود.

> [!tip] ملاحظة
> `403` ليس فشل دائما. أحيانا هو دليل أن المسار موجود ويستحق تحليل أكثر داخل التصريح.

### 404 Not Found

يعني أن الصفحة أو resource غير موجود.

في fuzzing:

- غالبا تستخدمه كـ baseline للنتائج غير المهمة.
- لكن أحيانا بعض المواقع ترجع `200` مع صفحة تقول Not Found، وهذا يسمى soft 404.

### 405 Method Not Allowed

يعني أن المسار موجود لكن HTTP method غير مسموح.

مثال:

- `GET` غير مسموح.
- `POST` مسموح.

في pentest:

- جرب methods مختلفة مثل `GET`, `POST`, `PUT`, `DELETE`, `OPTIONS` إذا كان مسموحا.

### 409 Conflict

يعني أن الطلب فيه conflict مع حالة النظام.

مثال:

- إنشاء user موجود بالفعل.
- تعديل resource تم تغييره.

### 429 Too Many Requests

يعني أنك أرسلت طلبات كثيرة وتم تطبيق rate limit.

في bug bounty:

- قلل السرعة.
- احترم rules البرنامج.
- لا تحاول تجاوز rate limit إلا لو مسموح ومطلوب.

## 5xx - Server Error

أكواد `5xx` تعني أن الخطأ من السيرفر أو خدمة خلفية.

### 500 Internal Server Error

خطأ عام في السيرفر.

في pentest:

- قد يدل على input تسبب crash أو exception.
- راقب هل الخطأ يكشف stack trace أو معلومات حساسة.

### 502 Bad Gateway

السيرفر الوسيط لم يحصل على رد صحيح من backend.

قد يظهر مع:

- Reverse proxy.
- Load balancer.
- CDN.

### 503 Service Unavailable

الخدمة غير متاحة مؤقتا.

أسباب:

- ضغط على السيرفر.
- Maintenance.
- Rate limiting أو protection.

### 504 Gateway Timeout

السيرفر الوسيط انتظر backend فترة طويلة ولم يحصل على رد.

قد يظهر مع requests ثقيلة أو backend بطيء.

## ما هي Response Page؟

Response page هي محتوى الصفحة الذي يرجع من السيرفر.

مثال:

- صفحة login.
- صفحة error.
- JSON response.
- HTML page.
- Empty response.
- Redirect page.

## كيف تحلل Response Page؟

راقب هذه النقاط:

- Status code.
- Page title.
- Response size.
- Words/lines count.
- Headers.
- Cookies.
- Redirect location.
- Error messages.
- Technology fingerprints.
- Differences بين response و baseline.

## Response Headers مهمة

| Header | الفائدة |
|---|---|
| `Server` | قد يكشف نوع السيرفر |
| `Set-Cookie` | يفيد في تحليل الجلسات |
| `Location` | يظهر مكان redirect |
| `Content-Type` | يوضح نوع الرد HTML, JSON, etc |
| `Content-Length` | حجم الرد |
| `X-Powered-By` | قد يكشف framework أو language |
| `WWW-Authenticate` | يظهر نوع authentication |

## أمثلة أثناء Fuzzing

| Code | كيف تفكر فيه؟ |
|---|---|
| `200` | قد يكون path موجود، افحص المحتوى |
| `301/302` | اتبع redirect وشوف الوجهة |
| `401` | endpoint موجود ويحتاج auth |
| `403` | path موجود غالبا لكن ممنوع |
| `404` | غالبا غير موجود، استخدمه baseline |
| `405` | path موجود لكن method خطأ |
| `500` | input أو request سبب خطأ في السيرفر |

## Soft 404

Soft 404 يعني أن السيرفر يرجع status code مثل `200` لكن الصفحة نفسها تقول إن المحتوى غير موجود.

مثال:

```text
HTTP/1.1 200 OK

Page Not Found
```

هذا يسبب noise أثناء fuzzing.

الحل:

- قارن response size.
- قارن page title.
- استخدم filters في الأداة.
- جرّب path عشوائي لمعرفة baseline.

## أوامر مفيدة

### curl لعرض headers

```bash
curl -I https://example.com
```

### curl لعرض response كامل

```bash
curl -i https://example.com/admin
```

### httpx لفحص status/title/tech

```bash
httpx -l urls.txt -status-code -title -tech-detect
```

### ffuf مع status codes

```bash
ffuf -u https://example.com/FUZZ -w wordlist.txt -mc all
```

## أخطاء شائعة

- اعتبار `404` دائما غير مهم بدون مقارنة المحتوى.
- تجاهل `403` رغم أنه قد يدل على path موجود.
- الاعتماد على status code فقط.
- عدم متابعة redirects.
- تجاهل response headers.
- عدم عمل baseline قبل fuzzing.

## الخلاصة

> [!summary] Response Analysis
> Status codes تعطيك أول إشارة عن نتيجة الطلب، لكن التحليل الصحيح يحتاج مقارنة الكود مع الصفحة، الحجم، headers، redirects، والاختلاف عن baseline.
