---
title: HTTP Requests Responses and Methods
date: 2026-04-26
tags:
  - http
  - request
  - response
  - methods
  - bug-bounty
  - pentest
---

# HTTP Requests, Responses, and Methods

[[Index|Back to Index]]

## الفكرة العامة

أي تواصل بين المتصفح والسيرفر في الويب غالبا يكون على شكل:

```text
Client -> HTTP Request -> Server
Client <- HTTP Response <- Server
```

المتصفح أو الأداة مثل `curl` أو Burp Suite ترسل request، والسيرفر يرد response.

> [!summary] المهم في الاختبار
> فهم شكل request و response هو أساس تحليل أي ثغرة Web، لأنك تحتاج تعرف ماذا أرسلت، وماذا رد السيرفر، وهل السلوك طبيعي أم لا.

## ما هو HTTP Request؟

HTTP Request هو الطلب الذي يرسله العميل للسيرفر.

يتكون غالبا من:

- Method مثل `GET` أو `POST`.
- Path أو URL.
- Headers.
- Body أحيانا.
- Cookies أحيانا.
- Query parameters أحيانا.

مثال:

```http
GET /profile?id=10 HTTP/1.1
Host: example.com
User-Agent: Mozilla/5.0
Cookie: session=abc123
```

## ما هو HTTP Response؟

HTTP Response هو رد السيرفر على الطلب.

يتكون غالبا من:

- Status code.
- Headers.
- Body أو page content.
- Cookies جديدة أحيانا.
- Redirect location أحيانا.

مثال:

```http
HTTP/1.1 200 OK
Content-Type: text/html
Set-Cookie: session=abc123; HttpOnly

<html>Profile page</html>
```

## أجزاء Request المهمة

| الجزء | المعنى | مثال |
|---|---|---|
| Method | نوع العملية المطلوبة | `GET`, `POST` |
| Path | المسار المطلوب | `/admin` |
| Query String | parameters داخل URL | `?id=10` |
| Headers | معلومات إضافية | `User-Agent`, `Cookie` |
| Body | بيانات الطلب | JSON أو form data |
| Cookies | بيانات الجلسة | `session=abc123` |

## Query Parameters

Query parameters تظهر بعد `?` في الرابط.

مثال:

```text
https://example.com/product?id=5&lang=en
```

هنا:

- `id=5`
- `lang=en`

في bug bounty، parameters مهمة لاختبار:

- IDOR.
- XSS.
- SQL Injection.
- Open Redirect.
- SSRF.
- LFI.

## Headers

Headers تحمل معلومات عن الطلب أو الرد.

أمثلة request headers:

```http
Host: example.com
User-Agent: Mozilla/5.0
Authorization: Bearer token
Content-Type: application/json
Cookie: session=abc123
```

أمثلة response headers:

```http
Server: nginx
Set-Cookie: session=abc123; HttpOnly; Secure
Location: /login
Content-Type: application/json
```

## Body

Body هو جزء البيانات في الطلب، ويظهر غالبا مع `POST`, `PUT`, `PATCH`.

مثال JSON:

```http
POST /api/login HTTP/1.1
Host: example.com
Content-Type: application/json

{
  "email": "test@example.com",
  "password": "123456"
}
```

مثال form data:

```http
POST /login HTTP/1.1
Host: example.com
Content-Type: application/x-www-form-urlencoded

username=admin&password=123456
```

## HTTP Methods

HTTP Method يوضح نوع العملية التي يريدها العميل.

| Method | الاستخدام الشائع |
|---|---|
| `GET` | قراءة بيانات |
| `POST` | إرسال أو إنشاء بيانات |
| `PUT` | استبدال resource كامل |
| `PATCH` | تعديل جزء من resource |
| `DELETE` | حذف resource |
| `HEAD` | مثل GET لكن بدون body |
| `OPTIONS` | معرفة methods المسموحة |
| `TRACE` | اختبار echo للطلب، غالبا يجب تعطيله |

## GET

`GET` يستخدم لجلب أو قراءة بيانات.

مثال:

```http
GET /products?id=10 HTTP/1.1
Host: example.com
```

ملاحظات:

- البيانات تظهر غالبا في URL.
- مناسب للقراءة وليس لتغيير البيانات.
- URLs قد تظهر في logs و history.

في الاختبار:

- راقب parameters.
- جرّب access control على IDs.
- افحص caching للبيانات الحساسة.

## POST

`POST` يستخدم لإرسال بيانات للسيرفر.

مثال:

```http
POST /login HTTP/1.1
Host: example.com
Content-Type: application/json

{"email":"test@example.com","password":"123456"}
```

في الاختبار:

- افحص validation.
- راقب authentication.
- افحص CSRF إذا مناسب.
- افحص rate limiting.
- افحص هل response يكشف معلومات حساسة.

## PUT

`PUT` يستخدم غالبا لاستبدال resource كامل.

مثال:

```http
PUT /api/users/10 HTTP/1.1
Host: example.com
Content-Type: application/json

{"name":"Ali","role":"user"}
```

في الاختبار:

- هل المستخدم يستطيع تعديل resource لا يملكه؟
- هل يمكن تغيير fields حساسة مثل `role`؟
- هل method مسموح بدون سبب؟

## PATCH

`PATCH` يستخدم لتعديل جزء من resource.

مثال:

```http
PATCH /api/users/10 HTTP/1.1
Host: example.com
Content-Type: application/json

{"name":"Ali"}
```

في الاختبار:

- افحص mass assignment.
- افحص authorization.
- جرّب fields غير ظاهرة في الواجهة إذا كان مسموحا.

## DELETE

`DELETE` يستخدم لحذف resource.

مثال:

```http
DELETE /api/users/10 HTTP/1.1
Host: example.com
```

في الاختبار:

- هل يمكن حذف resource يخص مستخدم آخر؟
- هل يوجد confirmation أو authorization؟
- هل الحذف فعلي أم soft delete؟

## HEAD

`HEAD` يشبه `GET` لكن السيرفر يرد headers فقط بدون body.

يفيد في:

- فحص وجود resource بسرعة.
- معرفة headers.
- تقليل حجم الرد.

مثال:

```bash
curl -I https://example.com/admin
```

## OPTIONS

`OPTIONS` يعرض أحيانا الطرق المسموحة على endpoint.

مثال:

```bash
curl -i -X OPTIONS https://example.com/api/users
```

قد يظهر:

```http
Allow: GET, POST, PUT, DELETE, OPTIONS
```

في الاختبار:

- يساعدك تعرف methods المتاحة.
- مهم مع APIs.
- قد يكشف methods خطيرة مفعلة بالخطأ.

## TRACE

`TRACE` يعيد الطلب كما وصل للسيرفر.

غالبا يجب تعطيله لأسباب أمنية.

في الاختبار:

- لو كان مفعلا، قد يكون misconfiguration.
- لا يعتبر دائما ثغرة عالية بمفرده، لكن يذكر حسب السياق.

## الفرق بين GET و POST

| النقطة | GET | POST |
|---|---|---|
| الاستخدام | قراءة بيانات | إرسال بيانات |
| مكان البيانات | URL غالبا | Body غالبا |
| الظهور في history/logs | أعلى | أقل في URL |
| مناسب للـ forms الحساسة | لا | نعم غالبا |
| قابلية caching | أعلى | أقل |

## Content-Type

`Content-Type` يوضح نوع البيانات في body.

أمثلة:

| Content-Type | الاستخدام |
|---|---|
| `application/json` | APIs |
| `application/x-www-form-urlencoded` | forms عادية |
| `multipart/form-data` | upload files |
| `text/plain` | نص عادي |
| `application/xml` | XML APIs |

## كيف يساعدني هذا في Bug Bounty و Pentest؟

- تفهم أين توجد البيانات: URL, body, headers, cookies.
- تعرف أي parameters تختبرها.
- تجرب method tampering مثل تغيير `GET` إلى `POST` أو `POST` إلى `PUT` إذا كان مسموحا.
- تحلل authorization على resources.
- تكتشف APIs غير محمية.
- تفهم أسباب status codes مثل `405 Method Not Allowed`.
- تستخدم Burp Suite بشكل أفضل.

## أمثلة أدوات

### curl GET

```bash
curl -i "https://example.com/product?id=10"
```

### curl POST JSON

```bash
curl -i -X POST https://example.com/api/login \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com","password":"123456"}'
```

### curl OPTIONS

```bash
curl -i -X OPTIONS https://example.com/api/users
```

### Burp Suite

في Burp Suite تقدر:

- تشوف request و response.
- تعدل method.
- تعدل headers.
- تعدل body.
- تعيد إرسال الطلب في Repeater.
- تقارن responses.

## أخطاء شائعة

- اعتبار GET و POST فرق أمان بحد ذاته.
- تجاهل headers و cookies.
- عدم فحص authorization عند تغيير IDs.
- تجاهل methods مثل PUT و DELETE.
- عدم ملاحظة `Content-Type`.
- اختبار endpoint بدون فهم وظيفته.

## الخلاصة

> [!summary] Request/Response
> أي اختبار ويب يبدأ من فهم request و response. راقب method, path, parameters, headers, cookies, body، ثم حلل response. هذا الأساس يساعدك تفهم APIs، access control، validation، ونتائج الفحص بشكل صحيح.
