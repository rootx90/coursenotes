---
title: 3 Tabs in Burp
date: 2026-04-28
tags:
  - burp-suite
  - proxy
  - repeater
  - intruder
  - bug-bounty
  - pentest
---

# 3 Tabs في Burp


## الفكرة العامة

بعد ما تضبط المتصفح على Burp Proxy، ستستخدم غالبا 3 tabs أساسية:

```text
Proxy -> Repeater -> Intruder
```

| Tab | الدور |
|---|---|
| Proxy | استقبال request من المتصفح وعرضه قبل وصوله للموقع |
| Repeater | تعديل request يدويا وإرساله أكثر من مرة |
| Intruder | تجربة payloads متعددة على جزء محدد من request |

الشكل العام:

```text
Proxy
  -> يشاهد request القادم من المتصفح
  -> ترسل request المهم إلى Repeater أو Intruder

Repeater
  -> تعديل request يدويا
  -> إرسال الطلب أكثر من مرة
  -> مقارنة response

Intruder
  -> تجربة أكثر من payload على نفس request
  -> مناسب للاختبارات المتكررة بشرط الالتزام بالـ scope والـ rate limits
```

## 1. Proxy Tab

Proxy هو أول مكان يصل إليه request من المتصفح بعد ضبط المتصفح على Burp.

استخدامه:

- مشاهدة request قبل وصوله للموقع.
- إيقاف request عند `Intercept is on`.
- تعديل request ثم الضغط على `Forward`.
- رؤية كل الطلبات في `HTTP history`.
- اختيار request مهم وإرساله إلى Repeater أو Intruder.

مثال flow:

```text
Browser
  -> Burp Proxy
  -> Proxy tab shows request
  -> Forward
  -> Website
```

مثال request يظهر في Proxy:

```http
GET /login HTTP/1.1
Host: example.com
Cookie: session=abc123
User-Agent: Mozilla/5.0
```

## 2. Repeater Tab

Repeater يستخدم للاختبار اليدوي الهادئ. تأخذ request من `Proxy -> HTTP history` وترسله إلى Repeater.

الخطوات:

```text
Proxy -> HTTP history
  -> اختر request
  -> Send to Repeater
  -> عدل parameter أو cookie أو header
  -> Send
  -> قارن response
```

مثال request:

```http
GET /api/user?id=1001 HTTP/1.1
Host: example.com
Cookie: session=abc123
```

تعدله إلى:

```http
GET /api/user?id=1002 HTTP/1.1
Host: example.com
Cookie: session=abc123
```

Repeater مناسب لاختبار:

- IDOR.
- Access control.
- تغيير parameters.
- تغيير cookies.
- اختبار headers.
- فهم response.
- مقارنة response قبل وبعد التعديل.

> [!tip] ملاحظة
> Repeater مناسب عندما تريد تفهم request واحد بتركيز وتعدل عليه يدويا أكثر من مرة.

## 3. Intruder Tab

Intruder يستخدم عندما تريد تجربة أكثر من قيمة على نفس المكان داخل request.

مثال:

```http
GET /api/user?id=§1001§ HTTP/1.1
Host: example.com
Cookie: session=abc123
```

علامة `§` تحدد المكان الذي ستتغير فيه القيم.

مثال payloads:

```text
1001
1002
1003
1004
```

الفكرة:

```text
Intruder يجرب id=1001
Intruder يجرب id=1002
Intruder يجرب id=1003
Intruder يجرب id=1004
```

ثم تقارن النتائج:

- Status code.
- Response length.
- Response body.
- Error messages.
- اختلاف البيانات الراجعة.

Intruder مناسب لاختبار:

- قيم كثيرة لنفس parameter.
- أسماء usernames في login flow مصرح به.
- IDs متتابعة داخل scope.
- Headers أو cookies بقيم مختلفة.
- Parameters تحتاج أكثر من payload.

> [!warning] مهم
> Intruder قد يرسل عدد كبير من requests. استخدمه فقط داخل scope، وبسرعة مناسبة، وبدون ضغط على السيرفر.

## الفرق بين Repeater و Intruder

| المقارنة | Repeater | Intruder |
|---|---|---|
| طريقة الاستخدام | يدوي | شبه آلي |
| عدد القيم | قيمة أو تعديلات قليلة | payloads كثيرة |
| الهدف | فهم request وتحليل response | تجربة قيم متعددة ومقارنة النتائج |
| الأفضل لـ | اختبار يدوي مركز | اختبار متكرر على نفس المكان |

## Workflow عملي

```text
Browser يرسل request
  -> يظهر في Proxy
  -> لو request مهم، أرسله إلى Repeater
  -> عدل واختبر يدويا
  -> لو احتجت تجربة قيم كثيرة، أرسله إلى Intruder
  -> قارن النتائج
  -> وثق فقط النتيجة التي لها impact
```

## الخلاصة

> [!summary] 3 Tabs في Burp
> Proxy يستقبل ويعرض الطلبات، Repeater يسمح بتعديل request يدويا وإرساله أكثر من مرة، و Intruder يستخدم لتجربة payloads متعددة على جزء محدد من request. الثلاثة مع بعض يشكلون workflow أساسي في اختبار تطبيقات الويب باستخدام Burp.
