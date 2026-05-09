---
title: Common Attacks 2 - XSS
date: 2026-05-05
tags:
  - common-attacks
  - xss
  - cross-site-scripting
  - web-security
  - bug-bounty
  - pentest
---


# Common Attacks 2 - XSS


## يعني إيه XSS؟

**XSS** اختصار لـ **Cross-Site Scripting**.

ببساطة، دي ثغرة بتحصل لما الموقع ياخد **input** من المستخدم، وبعدين يعرضه جوّه الصفحة من غير حماية كفاية. ساعتها الـ input ممكن يتحوّل لـ **JavaScript** يشتغل جوّه متصفح المستخدم.

الفكرة بشكل بسيط:

```text
User Input -> Website -> HTML Response -> Browser executes JavaScript
```

مثال صغير:

```text
https://example.com/search?q=ahmed
```

لو الموقع كتب `ahmed` جوّه الصفحة بشكل مباشر، ممكن نجرّب:

```html
<script>alert(1)</script>
```

لو ظهر **alert** في المتصفح، فده معناه إن الـ input اتحوّل لكود JavaScript واتنفّذ.

> [!warning] تنبيه مهم
> اختبر XSS بس على أهداف جوّه الـ **scope** أو في **labs** تدريبية. استخدم payloads آمنة زي `alert(1)` أو `print()` أو تغيير نص بسيط في الصفحة. ما تسرقش cookies أو tokens، وما تختبرش على مستخدمين حقيقيين من غير تصريح.

## ليه XSS مهمة في Bug Bounty؟

XSS ممكن تبان بسيطة لما الإثبات يكون `alert(1)`، لكن الخطورة الحقيقية إن JavaScript بيشتغل على نفس **domain** بتاع الموقع.

تأثيرها ممكن يكون:

- تنفيذ **actions** باسم الضحية.
- قراءة بيانات ظاهرة جوّه الصفحة.
- تغيير محتوى الصفحة.
- عمل **phishing** جوّه نفس الـ domain.
- سرقة tokens لو مكشوفة في JavaScript أو `localStorage`.
- الوصول لـ **Account Takeover** في بعض الحالات.
- لو كانت **Stored XSS** عند **admin** ممكن توصل لـ Critical.

> [!tip] ملاحظة لطيفة
> وجود `HttpOnly` على الـ cookies بيقلل خطر سرقة الكوكي مباشرة، لكنه مش بيلغي XSS؛ لأن JavaScript ممكن يفضل قادر ينفّذ actions باسم المستخدم.

## فين ممكن تلاقي XSS؟

دور في أي مكان الموقع بياخد فيه input من المستخدم وبعدين يعرضه في HTML أو JavaScript.

### أماكن شائعة

| المكان | مثال على الـ input |
|---|---|
| البحث | `?q=test` |
| التعليقات | `comment body` |
| البروفايل | `name`, `bio`, `website` |
| فورم التواصل | `subject`, `message` |
| التذاكر | `title`, `description` |
| الشات | `message` |
| التقييمات | `product review` |
| صفحات الأخطاء | `invalid parameter` |
| صفحات التحويل | `next`, `returnUrl` |
| أسماء الملفات | `uploaded file name` |
| بيانات الصور | `title`, `description` |
| لوحة الأدمن | بيانات المستخدمين المعروضة للأدمن |
| API responses | JSON بيتعرض بعدين في الـ frontend |
| Markdown editor | Markdown بعد ما يتعمله render |
| WYSIWYG editor | rich text content |

### Parameters مهمة وقت الاختبار

```text
q
search
query
keyword
name
username
email
message
comment
title
description
url
next
return
redirect
callback
lang
page
file
path
```

## الأنواع الأساسية لـ XSS

| النوع | الـ payload بيعيش فين؟ | بيشتغل إمتى؟ |
|---|---|---|
| Reflected XSS | جوّه الـ request | لما الرابط يتفتح أو request يتبعت |
| Stored XSS | جوّه database أو storage | لما البيانات تتعرض بعدين |
| DOM XSS | جوّه JavaScript في المتصفح | لما JS يقرأ input ويحطه في DOM بطريقة خطرة |

## 1. Reflected XSS

**Reflected XSS** بتحصل لما المستخدم يبعت input في request، والسيرفر يرجعه مباشرة في الـ response من غير escaping.

### مثال

Request:

```text
GET /search?q=ahmed HTTP/1.1
Host: example.com
```

Response:

```html
<h1>Search result for ahmed</h1>
```

لو غيرنا `q` لـ payload:

```text
/search?q=<script>alert(1)</script>
```

الـ response ممكن يبقى كده:

```html
<h1>Search result for <script>alert(1)</script></h1>
```

هنا المتصفح ممكن يشغّل JavaScript.

### فين تلاقي Reflected XSS؟

- البحث.
- رسائل الأخطاء.
- رسائل تسجيل الدخول.
- Parameters في الـ URL.
- الفلاتر.
- Tracking parameters.
- صفحات التحويل.

### الـ Impact

المهاجم غالبًا محتاج يخلي الضحية تفتح رابط معيّن. عشان كده التأثير بيعتمد على حساسية الصفحة وسياق المستخدم.

## 2. Stored XSS

**Stored XSS** بتحصل لما الـ payload يتخزن في database أو file أو cache، وبعدين يظهر لمستخدمين تانيين.

### مثال

مستخدم كتب comment:

```html
<img src=x onerror=alert(1)>
```

والتطبيق خزّنه زي ما هو.

لما أي مستخدم يفتح صفحة التعليقات، يظهر:

```html
<div class="comment">
  <img src=x onerror=alert(1)>
</div>
```

وساعتها JavaScript يشتغل عند كل شخص يفتح الصفحة.

### فين تلاقي Stored XSS؟

- التعليقات.
- التقييمات.
- تذاكر الدعم.
- رسائل الشات.
- اسم البروفايل أو الـ bio.
- أسماء المنتجات في dashboard.
- الملاحظات.
- اسم الملف المرفوع.
- لوحة الأدمن اللي بتعرض بيانات users.

### ليه Stored XSS أخطر؟

لأن الضحية مش محتاجة تفتح رابط غريب. يكفي تدخل صفحة طبيعية جوّه الموقع.

Stored XSS ضد admin ممكن يؤدي لـ:

- تنفيذ actions بصلاحيات الأدمن.
- تغيير settings.
- قراءة بيانات جوّه لوحة التحكم.
- takeover لو في tokens أو sensitive actions.

## 3. DOM XSS

**DOM XSS** بتحصل جوّه المتصفح، ومش لازم تكون جاية من السيرفر مباشرة.

المشكلة غالبًا بتكون في JavaScript بيقرأ قيمة من URL أو hash أو storage، وبعدين يحطها في الصفحة باستخدام API خطر زي `innerHTML`.

### مثال JavaScript vulnerable

URL:

```text
https://example.com/#<img src=x onerror=alert(1)>
```

Code:

```html
<div id="result"></div>

<script>
  const value = location.hash.substring(1);
  document.getElementById("result").innerHTML = value;
</script>
```

هنا JavaScript أخد القيمة من `location.hash` وحطها في `innerHTML`، فالمتصفح ممكن يفسّرها كـ HTML وينفّذها.

### Sources و Sinks

في DOM XSS بندوّر على حاجتين:

| المصطلح | معناه                                 | أمثلة                                                                               |
| ------- | ------------------------------------- | ----------------------------------------------------------------------------------- |
| Source  | المكان اللي الـ input بييجي منه       | `location`, `location.hash`, `location.search`, `document.referrer`, `localStorage` |
| Sink    | المكان الخطر اللي الـ input بيتحط فيه | `innerHTML`, `outerHTML`, `document.write`, `insertAdjacentHTML`, `eval`            |

### فين تلاقي DOM XSS؟

- Single Page Applications.
- بحث بيشتغل من غير reload.
- صفحات بتستخدم hash routing.
- Client-side templates.
- JavaScript بياخد parameters من URL.
- Widgets.

## أنواع تانية مهمة من XSS

### Blind XSS

**Blind XSS** هي Stored XSS مش بتظهرلك أنت مباشرة، لكنها بتشتغل عند شخص تاني، وغالبًا يكون admin أو support employee.

مثال:

```text
Contact form -> Admin panel
```

أنت تبعت payload في contact form، ومتشوفش النتيجة. لكن لو الأدمن فتح الرسالة في لوحة التحكم، الـ payload يشتغل عنده.

اختبارها يكون بأدوات callback آمنة زي **Burp Collaborator** أو **Interactsh**، وده لازم يكون جوّه الـ scope وطبقًا لسياسة البرنامج.

### Self XSS

**Self XSS** معناها إن الـ payload بيشتغل في حسابك أنت بس، أو محتاج الضحية تكتب كود في الـ console بنفسها.

غالبًا برامج bug bounty مش بتقبلها؛ لأنها محتاجة social engineering واضح ومش بتأثر على مستخدمين تانيين بشكل مباشر.

### Mutation XSS - mXSS

**Mutation XSS** بتحصل لما المتصفح أو الـ sanitizer يغيّر HTML بطريقة تخلي الـ payload يبان آمن في الأول، وبعد parsing يبقى خطر.

النوع ده بيظهر أكتر مع:

- HTML sanitizers ضعيفة.
- SVG/MathML.
- Rich text editors.
- Browser parsing quirks.

### Universal XSS - UXSS

**UXSS** بتكون ثغرة في المتصفح أو extension بتسمح بتشغيل JavaScript عبر مواقع مختلفة. النوع ده نادر، ومش هو الهدف المعتاد في bug bounty لتطبيق ويب.

## XSS حسب مكان ظهور الـ input

طريقة الحماية بتختلف حسب مكان ظهور الـ input جوّه HTML. نفس الـ payload مش لازم يشتغل في كل مكان.

### 1. HTML Body Context

مثال:

```html
<p>Hello USER_INPUT</p>
```

Payload:

```html
<script>alert(1)</script>
```

أو:

```html
<img src=x onerror=alert(1)>
```

### 2. HTML Attribute Context

مثال:

```html
<input value="USER_INPUT">
```

لو الـ input داخل attribute، غالبًا هتحتاج تخرج من الـ quotes:

```html
" autofocus onfocus=alert(1) x="
```

النتيجة الخطرة ممكن تبقى:

```html
<input value="" autofocus onfocus=alert(1) x="">
```

### 3. JavaScript Context

مثال:

```html
<script>
  const name = "USER_INPUT";
</script>
```

هنا الـ input جوّه JavaScript string.

Payload تعليمي:

```text
";alert(1);//
```

النتيجة:

```html
<script>
  const name = "";alert(1);//";
</script>
```

### 4. URL Context

مثال:

```html
<a href="USER_INPUT">open</a>
```

لو التطبيق بيسمح بـ `javascript:` جوّه `href`:

```text
javascript:alert(1)
```

ممكن يشتغل لما المستخدم يضغط على الرابط.

### 5. CSS Context

مثال:

```html
<div style="background: USER_INPUT">
```

CSS context أقل شيوعًا في المتصفحات الحديثة، لكنه مهم لو الـ input بيتحط جوّه `style` أو CSS file.

## Payloads آمنة للتعلم

استخدم payloads بسيطة، ما تسرقش بيانات وما تغيّرش حسابات:

```html
<script>alert(1)</script>
```

```html
<img src=x onerror=alert(1)>
```

```html
<svg onload=alert(1)>
```

```text
"><img src=x onerror=alert(1)>
```

```text
'"><svg onload=alert(1)>
```

```text
javascript:alert(1)
```

> [!tip] ملاحظة
> لو payload مشتغلش، ده مش معناه إن مفيش XSS. ممكن تكون في context مختلف، أو في encoding، أو sanitizer، أو CSP.

## تختبر XSS إزاي في Bug Bounty؟

Workflow عملي:

```text
افتح الموقع جوّه الـ scope
  -> شغّل Burp Proxy
  -> استخدم التطبيق بشكل طبيعي
  -> راقب parameters و forms
  -> جرّب input واضح زي xsstest123
  -> دور عليه في response
  -> حدد الـ context: HTML, attribute, JS, URL
  -> جرّب payload آمن مناسب للـ context
  -> اتأكد هل بيشتغل في حساب تاني ولا لأ
  -> حدد النوع: Reflected, Stored, DOM, Blind
  -> وثّق request, response, screenshot, impact
```

## تعرف إزاي إن الـ input ظهر في الـ response؟

استخدم قيمة فريدة:

```text
xsstest12345
```

وبعدين دور عليها في الـ response:

```html
<h1>xsstest12345</h1>
```

لو ظهرت، اسأل نفسك:

- ظهرت جوّه HTML body؟
- ظهرت جوّه attribute؟
- ظهرت جوّه JavaScript؟
- هل الحروف زي `< > " ' /` اتعملها encoding؟
- هل ينفع تكسر الـ context؟

## Vulnerable Code - Node.js Express

المثال ده vulnerable لأنه بيحط input مباشرة جوّه HTML.

```js
const express = require("express");
const app = express();

app.get("/search", (req, res) => {
  const q = req.query.q || "";
  res.send(`<h1>Search result for ${q}</h1>`);
});

app.listen(3000);
```

Request:

```text
GET /search?q=<script>alert(1)</script>
```

Response:

```html
<h1>Search result for <script>alert(1)</script></h1>
```

## Secure Code - Node.js Express

الحل: استخدم escaping قبل ما تحط input جوّه HTML.

```js
const express = require("express");
const escapeHtml = require("escape-html");
const app = express();

app.get("/search", (req, res) => {
  const q = req.query.q || "";
  res.send(`<h1>Search result for ${escapeHtml(q)}</h1>`);
});

app.listen(3000);
```

لو المستخدم بعت:

```html
<script>alert(1)</script>
```

هيظهر كنص:

```html
&lt;script&gt;alert(1)&lt;/script&gt;
```

ومش هيتنفّذ كـ JavaScript.

## Vulnerable Code - PHP

مثال vulnerable:

```php
<?php
$name = $_GET["name"] ?? "";
echo "<h1>Hello $name</h1>";
?>
```

Request:

```text
/hello.php?name=<img src=x onerror=alert(1)>
```

النتيجة:

```html
<h1>Hello <img src=x onerror=alert(1)></h1>
```

## Secure Code - PHP

استخدم `htmlspecialchars`:

```php
<?php
$name = $_GET["name"] ?? "";
$safeName = htmlspecialchars($name, ENT_QUOTES, "UTF-8");
echo "<h1>Hello $safeName</h1>";
?>
```

`ENT_QUOTES` مهمة لأنها بتعمل escaping للـ single quote والـ double quote.

## Vulnerable Code - DOM XSS

مثال خطر:

```html
<!doctype html>
<html>
  <body>
    <div id="message"></div>

    <script>
      const params = new URLSearchParams(location.search);
      const msg = params.get("msg") || "";
      document.getElementById("message").innerHTML = msg;
    </script>
  </body>
</html>
```

Payload:

```text
?msg=<img src=x onerror=alert(1)>
```

سبب الثغرة:

```js
innerHTML = msg
```

لأن `innerHTML` بتفسّر النص كـ HTML.

## Secure Code - DOM XSS

استخدم `textContent` لما تكون عايز تعرض نص بس:

```html
<!doctype html>
<html>
  <body>
    <div id="message"></div>

    <script>
      const params = new URLSearchParams(location.search);
      const msg = params.get("msg") || "";
      document.getElementById("message").textContent = msg;
    </script>
  </body>
</html>
```

هنا الـ payload هيظهر كنص ومش هيتحوّل لـ HTML.

## Vulnerable Code - React

React بيعمل escaping تلقائيًا لما تعرض text:

```jsx
function Profile({ name }) {
  return <h1>Hello {name}</h1>;
}
```

ده غالبًا آمن لأن React مش بيفسّر `name` كـ HTML.

لكن المثال ده vulnerable:

```jsx
function Bio({ html }) {
  return <div dangerouslySetInnerHTML={{ __html: html }} />;
}
```

لو `html` جاي من المستخدم من غير sanitizer قوي، ممكن تحصل XSS.

## Secure Code - React

الأفضل تعرض user input كنص:

```jsx
function Bio({ bio }) {
  return <p>{bio}</p>;
}
```

لو لازم تسمح بـ HTML، استخدم sanitizer موثوق زي DOMPurify مع allowlist محدودة:

```jsx
import DOMPurify from "dompurify";

function Bio({ html }) {
  const clean = DOMPurify.sanitize(html);
  return <div dangerouslySetInnerHTML={{ __html: clean }} />;
}
```

## أسباب XSS الشائعة

- عرض input من غير escaping.
- استخدام `innerHTML` مع user input.
- استخدام `document.write`.
- استخدام `eval`.
- Sanitizer ضعيف أو معمول custom.
- السماح بـ HTML في comments أو profiles.
- عدم فصل data عن code.
- إدخال user input جوّه JavaScript string.
- السماح بروابط `javascript:`.
- رفع SVG أو HTML من غير قيود.

## طرق الحماية

### 1. Output Encoding

اعمل encoding حسب المكان اللي بتعرض فيه البيانات:

| Context | الحماية |
|---|---|
| HTML body | HTML encoding |
| HTML attribute | Attribute encoding |
| JavaScript string | JavaScript encoding أو الأفضل تتجنب الإدخال جوّه script |
| URL | URL validation و URL encoding |
| CSS | تجنب user input جوّه CSS |

### 2. Sanitization

لو التطبيق بيسمح للمستخدم يكتب HTML، فالـ encoding لوحده مش كفاية؛ لأنك عايز تسمح ببعض tags.

استخدم sanitizer موثوق زي:

- DOMPurify في JavaScript.
- HTML Purifier في PHP.
- Bleach في Python.

الفكرة إنك تسمح بس بـ tags و attributes آمنة.

### 3. Safe DOM APIs

استخدم:

```js
element.textContent = userInput;
```

بدل:

```js
element.innerHTML = userInput;
```

### 4. Validate URLs

ما تسمحش بروابط تبدأ بـ:

```text
javascript:
data:
vbscript:
```

واسمح بس بـ `http` و `https` وقت الحاجة:

```js
function isSafeUrl(value) {
  const url = new URL(value);
  return url.protocol === "http:" || url.protocol === "https:";
}
```

### 5. Content Security Policy - CSP

CSP طبقة حماية إضافية بتقلل تأثير XSS.

مثال:

```text
Content-Security-Policy: default-src 'self'; script-src 'self'; object-src 'none'; base-uri 'self'
```

> [!tip] مهم
> CSP مش بتصلّح الكود vulnerable لوحدها. الأساس هو encoding و sanitization بشكل صحيح.

### 6. HttpOnly Cookies

استخدم:

```text
Set-Cookie: session=abc; HttpOnly; Secure; SameSite=Lax
```

`HttpOnly` يمنع JavaScript من قراءة الـ cookie مباشرة، لكنه ما يمنعش JavaScript من إرسال requests باسم المستخدم.

## الفرق بين Encoding و Sanitization

| الطريقة | تستخدمها إمتى؟ | مثال |
|---|---|---|
| Encoding | لما تحب تعرض input كنص فقط | اسم مستخدم، عنوان، رسالة نصية |
| Sanitization | لما تحب تسمح ببعض HTML | rich text editor, markdown preview |

مثال encoding:

```text
<b>hello</b>
```

يظهر للمستخدم كنص:

```html
&lt;b&gt;hello&lt;/b&gt;
```

مثال sanitization:

```html
<b>hello</b> <script>alert(1)</script>
```

ممكن يتحول لـ:

```html
<b>hello</b>
```

## علامات تساعدك أثناء الفحص

وقت الاختبار، لاحظ:

- هل الـ input ظهر في response؟
- هل الحروف `< > " ' /` اتعملها encoding؟
- هل الـ input ظهر جوّه script؟
- هل التطبيق بيستخدم frontend framework؟
- هل في CSP؟
- هل الـ payload بيشتغل في حساب تاني؟
- هل الـ payload محفوظ ولا reflected بس؟
- هل بيشتغل بعد reload؟
- هل بيظهر في admin panel؟

## أخطاء شائعة وقت تعلم XSS

- تجربة payload واحد بس وبعدين الحكم إن الموقع آمن.
- عدم تحديد الـ context.
- الخلط بين HTML encoding و URL encoding.
- اعتبار Self XSS finding قوي.
- تجاهل DOM XSS.
- تجاهل admin panels و Blind XSS.
- كتابة report من غير impact واضح.
- استخدام payloads مؤذية في bug bounty.

## مثال Report مختصر

```text
Title:
Stored XSS in support ticket title allows JavaScript execution in admin panel

Endpoint:
POST /support/tickets

Steps:
1. Login as a normal user.
2. Create a new ticket.
3. Set the title to: <img src=x onerror=alert(1)>
4. Open the ticket list from another account or admin test account.
5. The payload executes when the ticket title is rendered.

Impact:
An attacker can execute JavaScript in the context of staff users who view support tickets. This may allow actions to be performed using the staff user's active session.

Fix:
HTML-encode ticket titles before rendering them. If rich text is required, sanitize with a strict allowlist.
```

## ملخص سريع

XSS بتحصل لما input من المستخدم يتحول لكود يشتغل جوّه المتصفح.

أهم الأنواع:

- Reflected XSS.
- Stored XSS.
- DOM XSS.
- Blind XSS.
- Self XSS.
- Mutation XSS.

أهم أماكن البحث:

- البحث.
- التعليقات.
- البروفايلات.
- التذاكر.
- لوحات الأدمن.
- URL parameters.
- JavaScript code بيستخدم `innerHTML`.

أهم طرق الحماية:

- Output encoding حسب الـ context.
- Sanitization عند السماح بـ HTML.
- استخدام `textContent` بدل `innerHTML`.
- منع `javascript:` URLs.
- CSP كطبقة حماية إضافية.
- HttpOnly cookies لتقليل التأثير.



