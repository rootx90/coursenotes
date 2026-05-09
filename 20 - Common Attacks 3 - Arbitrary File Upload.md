---
title: Common Attacks 3 - Arbitrary File Upload
date: 2026-05-05
tags:
  - common-attacks
  - file-upload
  - arbitrary-file-upload
  - unrestricted-file-upload
  - web-security
  - bug-bounty
  - pentest
---
# Common Attacks 3 - Arbitrary File Upload

[[Index|Back to Index]]

## يعني إيه Arbitrary File Upload؟

**Arbitrary File Upload** أو **Unrestricted File Upload** هي ثغرة بتحصل لما التطبيق يسمح برفع ملفات من غير تحقق كافي من النوع، الامتداد، المحتوى، الحجم، مكان التخزين، أو طريقة عرض الملف بعد الرفع.

الفكرة ببساطة:

```text
User uploads file -> Server accepts file -> File becomes accessible or executable
```

لو التطبيق بيقبل أي ملف ويرفعه في مكان public، الموضوع ممكن يتحول إلى:

- Stored XSS.
- Information disclosure.
- File overwrite.
- Malware hosting.
- Remote Code Execution في أسوأ الحالات.

> [!warning] تنبيه مهم
> اختبر بس على أهداف جوّه الـ scope أو labs تدريبية. ما ترفعش ملفات مؤذية، وما تستخدمش reverse shell، وما تحاولش تنفّذ أوامر على سيرفر حقيقي إلا لو قواعد البرنامج سامحة بده صراحة وبوضوح. في bug bounty غالبًا كفاية تثبت الخطر بأقل دليل آمن.

## ليه File Upload خطيرة؟

رفع الملفات بيبان feature عادي: avatar, CV, image, invoice, attachment.

لكن لو الحماية ضعيفة، المستخدم ممكن يرفع ملف التطبيق مش متوقعه.

أمثلة impact:

- رفع HTML يؤدي إلى Stored XSS.
- رفع SVG يحتوي JavaScript.
- رفع ملف server-side زي PHP جوّه مجلد بيتنفذ.
- رفع ملف باسمه Path Traversal.
- استبدال ملف موجود.
- رفع ملفات كبيرة تسبب استهلاك storage.
- رفع ملف يحتوي secrets أو malware واستضافة الرابط على domain موثوق.
- تجاوز قيود نوع الملف باستخدام extension أو Content-Type مزيف.

## فين تلاقي File Upload؟

دور في أي feature بتسمح بإرسال file أو attachment.

| المكان | أمثلة |
|---|---|
| Profile avatar | صورة الحساب |
| Support tickets | attachments |
| Chat | إرسال ملفات |
| CV upload | PDF أو DOCX |
| KYC | passport, national ID |
| Product images | صور منتجات |
| Blog CMS | media library |
| Import feature | CSV, XML, JSON |
| Report upload | PDF, Excel |
| Logo upload | company logo |
| Markdown editor | drag and drop images |
| Admin panel | file manager |
| API upload endpoint | `/api/upload`, `/files`, `/media` |

## أسماء Endpoints شائعة

```text
/upload
/api/upload
/api/files
/api/media
/avatar
/profile/photo
/attachments
/support/attachments
/admin/media
/files/upload
/documents
/import
```

## شكل Request رفع ملف

غالبًا بيكون `multipart/form-data`.

مثال:

```http
POST /upload HTTP/1.1
Host: example.com
Content-Type: multipart/form-data; boundary=----abc

------abc
Content-Disposition: form-data; name="file"; filename="avatar.png"
Content-Type: image/png

PNG_FILE_CONTENT
------abc--
```

الأجزاء المهمة أثناء الاختبار:

- `filename`.
- `Content-Type`.
- محتوى الملف نفسه.
- response بعد الرفع.
- رابط الملف بعد الرفع.
- هل الملف public ولا private؟
- هل بيتغير اسمه؟
- هل بيتحفظ في CDN ولا على نفس server؟

## أنواع File Upload Vulnerabilities

| النوع | الشرح | Impact |
|---|---|---|
| Unrestricted file type | بيقبل أي extension | XSS, RCE, hosting |
| Content-Type bypass | بيعتمد بس على header | رفع ملف غير مسموح |
| Extension bypass | فلتر extension ضعيف | تنفيذ أو عرض خطر |
| Magic bytes bypass | تحقق سطحي من بداية الملف فقط | Polyglot files |
| SVG upload XSS | بيسمح بـ SVG active content | Stored XSS |
| HTML upload | بيسمح بـ HTML public | Stored XSS أو phishing |
| Path traversal filename | بيستخدم filename زي ما هو | حفظ خارج المسار |
| File overwrite | نفس الاسم بيستبدل ملف قديم | تغيير محتوى أو data loss |
| Public sensitive upload | ملفات private بتبقى public | تسريب بيانات |
| Large file upload | مفيش size limit | DoS أو storage abuse |
| Archive extraction | بيفك ZIP من غير حماية | Zip Slip أو overwrite |
| Metadata leakage | مش بيمسح EXIF أو metadata | تسريب location أو device |

## تختبر File Upload إزاي؟

Workflow عملي:

```text
حدد upload feature
  -> ارفع ملف طبيعي مسموح
  -> راقب request في Burp
  -> افهم validation: extension, MIME, magic bytes, size
  -> افهم مكان التخزين والرابط الناتج
  -> جرب أنواع ملفات آمنة مختلفة
  -> جرب تغيير filename و Content-Type
  -> افحص هل الملف بيتعرض ولا بيتحمل
  -> افحص response headers
  -> اربط النتيجة بـ impact واضح
  -> وثق بأقل دليل كافي
```

## 1. Unrestricted File Type

بتحصل لما التطبيق يسمح برفع أي امتداد.

مثال:

```text
avatar.png -> accepted
avatar.txt -> accepted
avatar.html -> accepted
avatar.php -> accepted
```

مش كل حالة قبول ملف غير صورة تعتبر ثغرة عالية. لازم تسأل:

- هل الملف public؟
- هل المتصفح بيعرضه ولا بيحمّله؟
- هل JavaScript جواه بيشتغل؟
- هل السيرفر بينفذ ملفات server-side؟
- هل ممكن استخدامه ضد مستخدم تاني؟

## 2. Content-Type Bypass

بعض التطبيقات بتعتمد بس على `Content-Type`.

Request طبيعي:

```http
Content-Disposition: form-data; name="file"; filename="avatar.png"
Content-Type: image/png
```

Bypass تعليمي:

```http
Content-Disposition: form-data; name="file"; filename="page.html"
Content-Type: image/png
```

لو التطبيق قبل `page.html` لأنه شاف `image/png` بس، يبقى التحقق ضعيف.

> [!tip] ملاحظة
> `Content-Type` بييجي من client وممكن يتغير بسهولة من Burp. ما تعتمدش عليه لوحده في الحماية.

## 3. Extension Bypass

فلتر الامتدادات ممكن يكون ضعيف.

أمثلة تعليمية لأسماء ملفات المفروض التطبيق يرفضها لو هو بيسمح بالصور بس:

```text
shell.php
shell.php.jpg
shell.phtml
shell.phar
image.svg
image.html
image.php%00.jpg
image.jpg.php
```

في bug bounty، مش محتاج ترفع web shell مؤذية. كفاية ترفع ملف proof آمن يثبت إن التطبيق قبل امتداد غير مسموح أو نفذ ملف بسيط جوّه lab.

## 4. Magic Bytes Bypass

**Magic bytes** هي أول bytes في الملف وبتدل على نوعه.

مثال PNG بيبدأ غالبًا بـ:

```text
89 50 4E 47
```

بعض التطبيقات بتفحص بداية الملف بس، فممكن يتعمل ملف يبدأ كبنية صورة وبعدين يحتوي HTML أو code بعد كده. ده بيتسمى أحيانًا polyglot file.

الاختبار الآمن:

- ارفع صورة صحيحة.
- غير الامتداد.
- افحص هل التطبيق بيتحقق من المحتوى الحقيقي ولا extension بس.
- ما تستخدمش polyglot لتنفيذ كود على هدف حقيقي من غير تصريح.

## 5. SVG Upload XSS

SVG ملف صورة، لكنه مبني على XML وممكن يحتوي scripts أو event handlers في بعض السياقات.

مثال تعليمي:

```xml
<svg xmlns="http://www.w3.org/2000/svg" onload="alert(1)">
  <text x="10" y="20">XSS Test</text>
</svg>
```

لو التطبيق بيسمح برفع SVG وبيعرضه جوّه نفس domain، ممكن يحصل Stored XSS.

الحماية:

- منع SVG لو مش ضروري.
- أو sanitize SVG بصرامة.
- أو تقديمه من domain منفصل من غير cookies.
- أو إجبار `Content-Disposition: attachment`.

## 6. HTML Upload

لو التطبيق بيسمح برفع `.html` وبيعرضه public، ممكن يستخدم في Stored XSS أو phishing.

مثال آمن للتجربة:

```html
<!doctype html>
<html>
  <body>
    <h1>Upload proof</h1>
    <script>alert(1)</script>
  </body>
</html>
```

الـ Impact بيعتمد على:

- هل الملف على نفس domain؟
- هل cookies بتتبعت معاه؟
- هل ممكن تبعت الرابط للضحية؟
- هل في CSP؟
- هل ممكن توصل لبيانات التطبيق من نفس origin؟

## 7. Server-Side File Execution

دي أخطر حالة: ترفع ملف server-side وبعدين السيرفر ينفذه.

مثال في بيئة PHP:

```text
proof.php
```

محتوى proof آمن في lab:

```php
<?php echo "upload-proof"; ?>
```

لو فتحت:

```text
https://example.com/uploads/proof.php
```

وظهر:

```text
upload-proof
```

فده معناه إن السيرفر نفذ PHP جوّه مجلد uploads.

> [!warning] مهم
> ما تستخدمش reverse shell أو web shell على أهداف حقيقية. إثبات تنفيذ code بسيط زي طباعة نص ثابت جوّه lab أو حسب قواعد البرنامج غالبًا كفاية لإثبات الخطورة.

### إيه علاقة Reverse Shell بالموضوع؟

Reverse shell هو سيناريو بعد RCE، بيخلي السيرفر يفتح اتصال راجع لجهاز المهاجم للتحكم بالأوامر.

في سياق File Upload:

```text
File Upload -> Server executes uploaded file -> RCE -> Reverse shell ممكن
```

لكن في bug bounty، غالبًا مش محتاج توصل للمرحلة دي. اذكر في report إن الثغرة قد تؤدي إلى RCE، واثبت ده بطريقة آمنة ومحدودة حسب الـ policy.

## 8. Path Traversal in Filename

بعض التطبيقات بتستخدم اسم الملف زي ما المستخدم بعته.

Request:

```http
Content-Disposition: form-data; name="file"; filename="../../avatar.png"
```

لو السيرفر ما بينضفش الاسم، ممكن يحفظ الملف بره مجلد uploads.

Impact:

- overwrite لملفات مهمة.
- حفظ ملف في path public مختلف.
- تجاوز قيود التخزين.

الحماية:

- تجاهل اسم الملف الأصلي في التخزين.
- توليد اسم random.
- استخدام path ثابت.
- منع `../` و separators.

## 9. File Overwrite

بتحصل لما المستخدم يرفع ملف بنفس اسم ملف موجود ويتم استبداله.

مثال:

```text
logo.png
```

لو attacker يقدر يرفع ملف بنفس الاسم ويستبدل logo أو ملف مستخدم تاني، فده خطر.

Impact:

- تغيير محتوى ظاهر للمستخدمين.
- حذف أو تخريب ملفات.
- استبدال invoice أو document.
- Stored XSS لو استبدل ملف بيتعرض في browser.

الحماية:

- أسماء عشوائية.
- منع overwrite.
- ربط كل ملف بمالكه.
- authorization قبل التعديل أو الحذف.

## 10. Public Access to Private Files

أحيانًا الرفع نفسه يكون آمن، لكن الملفات تبقى public من غير authorization.

مثال:

```text
https://example.com/uploads/2026/05/passport-ahmed.png
```

لو أي شخص يقدر يفتح الرابط من غير login أو من غير ownership check، فدي مشكلة authorization.

اختبر بحسابين:

```text
User A uploads file
User B opens file URL
```

لو User B يقدر يشوف file خاص بـ User A، فده finding قوي.

## 11. Archive Upload and Zip Slip

بعض التطبيقات بتسمح برفع ZIP وبعدين تفك الملفات على السيرفر.

الخطر:

- ملفات جوّه ZIP بأسماء فيها `../`.
- overwrite لملفات بره folder.
- استخراج ملفات كتير جدًا.
- zip bombs تستهلك disk أو CPU.

مثال أسماء خطرة جوّه archive:

```text
../../public/index.html
../../../config.php
```

الحماية:

- منع path traversal وقت extraction.
- تحديد عدد وحجم الملفات بعد الفك.
- رفض symlinks.
- فك archive جوّه sandbox directory.

## 12. Image Processing Bugs

بعض التطبيقات بتعالج الصور بعد الرفع:

- resize.
- crop.
- convert.
- read metadata.
- generate thumbnail.

المخاطر:

- parser vulnerable.
- image library قديمة.
- metadata leakage.
- crash بسبب malformed image.

في bug bounty، ما تستخدمش ملفات exploit مؤذية. اختبر بشكل آمن:

- هل EXIF بيتم حذفه؟
- هل الحجم محدود؟
- هل نوع الصورة بيتحقق فعلًا؟
- هل الصورة بيتعاد تحويلها لصيغة آمنة؟

## Reverse Engineering للـ Upload Filter

المقصود هنا فهم التطبيق بيتحقق من الملف إزاي، مش كسر نظام خارج الـ scope.

### تراقب إيه؟

- هل الرفض بيحصل في browser ولا server؟
- هل في JavaScript بيمنع الامتداد بس؟
- هل server بيرجع error مختلف لكل سبب؟
- هل بيعتمد على `filename`؟
- هل بيعتمد على `Content-Type`؟
- هل بيفحص magic bytes؟
- هل بيعيد ضغط الصور؟
- هل بيغير اسم الملف؟
- هل بيخزن في S3 أو CDN؟
- هل الرابط الناتج signed URL ولا public URL؟

### Client-Side Validation

مثال JavaScript بيمنع غير الصور:

```js
const file = input.files[0];

if (!file.name.endsWith(".png")) {
  alert("Only PNG allowed");
}
```

ده مش حماية كافية، لأن attacker يقدر يعدل request من Burp ويبعت ملف مختلف.

### Server-Side Validation

الحماية الحقيقية لازم تكون في السيرفر:

```text
Browser validation = User experience
Server validation = Security
```

## Vulnerable Code - PHP

الكود ده vulnerable لأنه:

- بيقبل أي file.
- بيستخدم اسم الملف الأصلي.
- بيحفظ جوّه public uploads.
- مش بيفحص extension أو content.

```php
<?php
$target = "uploads/" . $_FILES["file"]["name"];
move_uploaded_file($_FILES["file"]["tmp_name"], $target);
echo "Uploaded to " . $target;
?>
```

مشاكل:

```text
filename="proof.php"
filename="../../proof.php"
filename="xss.html"
filename="avatar.php.jpg"
```

## Secure Code - PHP

مثال أبسط وأكثر أمانًا للصور بس:

```php
<?php
$allowedExtensions = ["jpg", "jpeg", "png"];
$allowedMimeTypes = ["image/jpeg", "image/png"];
$maxSize = 2 * 1024 * 1024;

if (!isset($_FILES["file"]) || $_FILES["file"]["error"] !== UPLOAD_ERR_OK) {
    http_response_code(400);
    exit("Upload failed");
}

if ($_FILES["file"]["size"] > $maxSize) {
    http_response_code(400);
    exit("File too large");
}

$originalName = $_FILES["file"]["name"];
$extension = strtolower(pathinfo($originalName, PATHINFO_EXTENSION));

if (!in_array($extension, $allowedExtensions, true)) {
    http_response_code(400);
    exit("Invalid extension");
}

$finfo = new finfo(FILEINFO_MIME_TYPE);
$mime = $finfo->file($_FILES["file"]["tmp_name"]);

if (!in_array($mime, $allowedMimeTypes, true)) {
    http_response_code(400);
    exit("Invalid file type");
}

$safeName = bin2hex(random_bytes(16)) . "." . $extension;
$uploadDir = __DIR__ . "/uploads/";
$target = $uploadDir . $safeName;

if (!move_uploaded_file($_FILES["file"]["tmp_name"], $target)) {
    http_response_code(500);
    exit("Could not save file");
}

echo "Uploaded";
?>
```

تحسينات إضافية:

- تخزين uploads خارج web root.
- إعادة تحويل الصور باستخدام image library.
- تقديم الملفات عبر endpoint بيفحص الصلاحيات.
- ضبط السيرفر عشان ما ينفذش scripts جوّه uploads.

## Vulnerable Code - Node.js Express

مثال vulnerable باستخدام `multer`:

```js
const express = require("express");
const multer = require("multer");

const app = express();
const upload = multer({ dest: "public/uploads/" });

app.post("/upload", upload.single("file"), (req, res) => {
  res.json({
    url: `/uploads/${req.file.filename}`,
  });
});

app.use(express.static("public"));
app.listen(3000);
```

المشكلة:

- مفيش file filter.
- مفيش size limit.
- الملفات public.
- مفيش authorization حول الوصول للملفات.

## Secure Code - Node.js Express

مثال أفضل للصور:

```js
const crypto = require("crypto");
const path = require("path");
const express = require("express");
const multer = require("multer");

const app = express();

const storage = multer.diskStorage({
  destination: "uploads-private/",
  filename: (req, file, cb) => {
    const ext = path.extname(file.originalname).toLowerCase();
    cb(null, `${crypto.randomBytes(16).toString("hex")}${ext}`);
  },
});

const upload = multer({
  storage,
  limits: {
    fileSize: 2 * 1024 * 1024,
  },
  fileFilter: (req, file, cb) => {
    const allowed = new Set(["image/png", "image/jpeg"]);
    cb(null, allowed.has(file.mimetype));
  },
});

app.post("/upload", upload.single("file"), (req, res) => {
  res.json({ id: req.file.filename });
});

app.listen(3000);
```

ملاحظات:

- ده مثال تعليمي جيد كبداية، لكنه مش كفاية لوحده في production.
- الأفضل تفحص magic bytes كمان.
- ما تقدمش `uploads-private` كـ static public folder.
- اعمل endpoint منفصل لقراءة الملف بعد authorization.

## إعدادات سيرفر مهمة

### Apache

جوّه مجلد uploads، امنع تنفيذ PHP:

```apache
php_flag engine off
RemoveHandler .php .phtml .phar
Options -ExecCGI
```

### Nginx

ما تمررش ملفات uploads إلى PHP-FPM:

```nginx
location /uploads/ {
    default_type application/octet-stream;
    add_header Content-Disposition "attachment";
}
```

الفكرة:

```text
uploads folder should store files, not execute code
```

## Headers مهمة عند عرض الملفات

لتقليل XSS من الملفات المرفوعة:

```text
Content-Type: application/octet-stream
Content-Disposition: attachment
X-Content-Type-Options: nosniff
```

لو محتاج تعرض الصور:

```text
Content-Type: image/png
X-Content-Type-Options: nosniff
```

## أفضل طرق الحماية

- Allowlist للامتدادات، مش blocklist.
- تحقق من MIME الحقيقي، مش header بس.
- تحقق من magic bytes.
- حدد maximum file size.
- غير اسم الملف إلى random name.
- ما تستخدمش filename الأصلي في path.
- خزن الملفات خارج web root.
- ما تنفذش scripts جوّه uploads.
- قدم الملفات عبر endpoint فيه authorization.
- استخدم separate domain للملفات وقت الحاجة.
- أعد تحويل الصور إلى صيغة آمنة.
- احذف metadata الحساسة من الصور.
- افحص الملفات بـ antivirus وقت الحاجة.
- امنع SVG أو sanitize بصرامة.
- استخدم `Content-Disposition: attachment` للملفات غير الموثوقة.
- استخدم `X-Content-Type-Options: nosniff`.
- سجل عمليات الرفع والتحميل.

## Checklist للاختبار

```text
[ ] هل upload endpoint جوّه scope؟
[ ] هل يوجد client-side validation فقط؟
[ ] هل يمكن تغيير Content-Type؟
[ ] هل يمكن تغيير extension؟
[ ] هل يقبل SVG؟
[ ] هل يقبل HTML؟
[ ] هل يقبل server-side extensions زي php, phtml, aspx, jsp؟
[ ] هل الملف public؟
[ ] هل الملف على نفس domain؟
[ ] هل المتصفح بيعرض الملف ولا بيحمّله؟
[ ] هل يوجد nosniff؟
[ ] هل يوجد Content-Disposition attachment؟
[ ] هل يمكن استبدال ملف موجود؟
[ ] هل filename يسمح بـ ../؟
[ ] هل يمكن الوصول لملفات مستخدم آخر؟
[ ] هل يوجد size limit؟
[ ] هل يتم حذف metadata؟
[ ] هل يتم استخراج ZIP أو archives؟
```

## إمتى تكون Finding؟

مش كل upload issue ثغرة عالية. قيّم حسب الـ impact.

| الحالة | Severity تقريبي |
|---|---|
| رفع PHP وتنفيذه | Critical غالبًا |
| Stored XSS عبر HTML/SVG على نفس domain | High أو Medium حسب السياق |
| الوصول لملفات private لمستخدم آخر | High غالبًا |
| رفع أي ملف لكن يتم تحميله attachment فقط | Low أو Informational غالبًا |
| bypass client-side validation فقط والسيرفر يمنع | غالبًا Not a valid finding |
| رفع ملف كبير يستهلك storage | Medium أو High حسب التأثير |
| EXIF metadata leakage | Low أو Medium حسب البيانات |

## مثال Report مختصر

```text
Title:
Arbitrary HTML file upload leads to stored XSS on main application domain

Endpoint:
POST /api/upload

Steps:
1. Login as a normal user.
2. Upload a file named proof.html.
3. Set Content-Type to text/html.
4. The application accepts the file and returns:
   https://example.com/uploads/proof.html
5. Open the returned URL.
6. JavaScript executes in the context of example.com.

Impact:
An attacker can host JavaScript on the main application domain. This can be used to perform phishing or execute actions in the context of users who open the uploaded file.

Fix:
Restrict uploads to a strict allowlist, store files outside web root, serve untrusted files with Content-Disposition: attachment, and use X-Content-Type-Options: nosniff.
```

## ملخص سريع

**Arbitrary File Upload** معناها إن التطبيق بيسمح برفع ملفات بطريقة غير مقيدة أو غير آمنة.

أهم حاجة تفحصها:

- Extension.
- Content-Type.
- Magic bytes.
- Filename.
- Storage path.
- Public access.
- Execution.
- Authorization.
- Response headers.

أخطر النتائج:

- تنفيذ server-side code.
- Stored XSS من HTML أو SVG.
- الوصول لملفات خاصة.
- overwrite أو path traversal.

أفضل حماية:

```text
Strict allowlist
  -> Validate real content
  -> Random filename
  -> Store outside web root
  -> No execution in uploads
  -> Authorization before access
  -> Safe response headers
```