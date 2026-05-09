---
title: Common Attacks 4 - LFI Path Traversal and Log Poisoning
date: 2026-05-05
tags:
  - common-attacks
  - lfi
  - local-file-inclusion
  - path-traversal
  - directory-traversal
  - log-poisoning
  - web-security
  - bug-bounty
  - pentest
---


# Common Attacks 4 - LFI Path Traversal and Log Poisoning

[[Index|Back to Index]]

## مقدمة

المقال ده بيشرح 4 مفاهيم مهمين جدا في Web Security و Bug Bounty:

```text
Path Traversal
LFI
RFI
Log Poisoning
````

المواضيع دي مرتبطة ببعض جدا، وكتير بتظهر في نفس الـ attack chain.

الفكرة العامة:

```text
Parameter بيتحكم في file path
  -> Path Traversal يخليك تخرج من folder المسموح
  -> LFI يخليك تقرأ أو تعمل include لملف local
  -> Log Poisoning ممكن يصعد الخطر لو قدرت تكتب داخل log وبعدين التطبيق يقرأه أو يعمله include
```

بمعنى أبسط:

```text
Path Traversal = حيلة المسار
LFI = الثغرة اللي بتقرأ أو بتعمل include لملف local
RFI = include لملف remote من سيرفر خارجي
Log Poisoning = تسميم ملف log بمدخلات تتحكم فيها ثم قراءتها أو تضمينها
```

> [!warning] تنبيه مهم  
> كل الأمثلة هنا للتعلم فقط داخل labs أو أهداف مصرح باختبارها.  
> في Bug Bounty الحقيقي، التزم بالـ scope والـ policy.  
> لا تقرأ ملفات حساسة إلا بأقل قدر يثبت الثغرة.  
> لا تحاول تنفيذ أوامر أو Web Shell أو Reverse Shell على production.  
> استخدم markers نصية آمنة لإثبات المشكلة.

---

## هل LFI و Path Traversal و Log Poisoning في نفس الموضوع؟

آه، الأفضل في البداية يكونوا في نفس الموضوع لأنهم غالبا بيظهروا كسلسلة واحدة.

مثال:

```text
/download?file=report.pdf
```

لو التطبيق بيستخدم قيمة `file` مباشرة في قراءة الملفات، ممكن تجرب:

```text
../../../../etc/hosts
```

لو نجحت القراءة، هنا عندك Path Traversal.

لو التطبيق مش بس بيقرأ الملف، لكنه بيعمل include له داخل التطبيق، يبقى عندك LFI.

ولو قدرت تكتب marker داخل log وبعدين تقرأ الـ log من خلال LFI، يبقى عندك Log Poisoning risk.

بعد ما تفهم الأساس، ممكن تفصلهم لاحقا:

```text
Path Traversal topic
LFI / RFI topic
Log Poisoning topic
LFI to RCE topic
```

لكن كتعلم عملي، وجودهم مع بعض مفيد جدا.

---

## ما هو Path Traversal؟

Path Traversal أو Directory Traversal بتحصل لما المستخدم يقدر يتحكم في اسم ملف أو مسار، والتطبيق ما يمنعوش من الخروج بره المجلد المسموح.

مثال طبيعي:

```http
GET /download?file=report.pdf
```

التطبيق المفروض يقرأ من:

```text
/var/www/app/files/report.pdf
```

لكن لو المستخدم بعت:

```http
GET /download?file=../../../../etc/hosts
```

ممكن التطبيق يقرأ:

```text
/etc/hosts
```

وده معناه إن المستخدم خرج من:

```text
/var/www/app/files/
```

ووصل لمسار تاني على السيرفر.

---

## لماذا نستخدم ../ ؟

`../` معناها ارجع folder واحد للخلف.

مثال:

```text
/var/www/app/files/report.pdf
```

لو استخدمت:

```text
../
```

هترجع من:

```text
/files/
```

إلى:

```text
/app/
```

ولو استخدمت:

```text
../../
```

هترجع مستويين.

مثال:

```text
../../../../etc/hosts
```

ممكن يتحول في النهاية إلى:

```text
/etc/hosts
```

حسب مكان التطبيق وعدد المجلدات.

---

## ما هو LFI؟

LFI اختصار لـ:

```text
Local File Inclusion
```

يعني التطبيق بيسمح بإدخال اسم ملف، وبعدين بيقرأه أو بيعمله include من نفس السيرفر.

مثال PHP خطر:

```php
<?php
include($_GET["page"]);
?>
```

لو المستخدم طلب:

```http
GET /index.php?page=home.php
```

التطبيق هيعمل include لملف:

```text
home.php
```

لكن لو المستخدم طلب:

```http
GET /index.php?page=../../../../etc/hosts
```

ممكن التطبيق يحاول يقرأ أو يضم ملف من النظام.

---

## الفرق بين File Read و File Include

مش كل LFI معناها تنفيذ كود.

فيه فرق مهم:

|النوع|المعنى|الخطورة|
|---|---|---|
|File Read|التطبيق يقرأ الملف ويعرضه كنص|ممكن يكشف ملفات حساسة|
|File Include|التطبيق يدخل الملف جوه execution flow|ممكن يزود الخطورة حسب اللغة والسياق|

مثال File Read:

```php
readfile($_GET["file"]);
```

مثال File Include:

```php
include($_GET["page"]);
```

`readfile` غالبا بيعرض المحتوى فقط.

لكن `include` في PHP ممكن يتعامل مع الملف كجزء من التطبيق، وده بيخلي الخطر أكبر.

---

## الفرق بين Path Traversal و LFI

|النقطة|Path Traversal|LFI|
|---|---|---|
|الفكرة|الخروج من folder باستخدام مسارات مثل `../`|قراءة أو include ملف local|
|المكان الشائع|download, view, image|page, template, language|
|النتيجة|قراءة ملف غالبا|قراءة ملف أو احتمال تصعيد حسب السياق|
|مثال|`/download?file=../../etc/hosts`|`/index.php?page=../../etc/hosts`|

ببساطة:

```text
Path Traversal = الطريقة اللي بتتحرك بيها في المسارات
LFI = الثغرة اللي بتسمح بالقراءة أو include
```

---

## ما هو RFI؟

RFI اختصار لـ:

```text
Remote File Inclusion
```

ودي شبيهة بـ LFI، لكن بدل ما التطبيق يعمل include لملف local من نفس السيرفر، بيعمل include لملف remote من سيرفر خارجي.

مثال خطر:

```php
<?php
include($_GET["page"]);
?>
```

لو إعدادات PHP تسمح، المهاجم ممكن يحاول يمرر رابط خارجي.

### الفرق بين LFI و RFI

|النوع|مصدر الملف|مثال الفكرة|
|---|---|---|
|LFI|ملف موجود على نفس السيرفر|ملف داخل النظام|
|RFI|ملف موجود على سيرفر خارجي|ملف من URL خارجي|

> [!warning] ملاحظة مهمة  
> RFI غالبا محتاج إعدادات PHP معينة مثل `allow_url_include = On`.  
> في البيئات الحديثة غالبا بتكون مقفولة، لكن لازم تفهم الفكرة.

---

## الفرق بين LFI و SSRF

فيه ناس بتخلط بين LFI و SSRF.

|النقطة|LFI|SSRF|
|---|---|---|
|الهدف|قراءة أو include ملفات local|إجبار السيرفر يبعت requests|
|المصدر|File system|Network / HTTP services|
|مثال الفكرة|قراءة ملف من السيرفر|طلب خدمة داخلية من السيرفر|
|الخطر|كشف ملفات أو تصعيد في بعض الحالات|الوصول لخدمات داخلية أو metadata|

مثال LFI:

```text
?page=../../../../etc/hosts
```

مثال SSRF:

```text
?url=http://127.0.0.1:8080/admin
```

الاتنين server-side، لكن واحد بيتعامل مع ملفات، والتاني بيتعامل مع network requests.

---

## أين تجد Path Traversal و LFI؟

دور على أي parameter له علاقة بملف، صفحة، صورة، لغة، template، أو download.

### Parameters شائعة

```text
file
path
page
template
view
lang
locale
theme
document
download
image
img
css
js
include
module
folder
name
dir
load
content
resource
```

### Endpoints شائعة

```text
/download?file=report.pdf
/view?file=invoice.pdf
/image?name=avatar.png
/index.php?page=home
/template?name=main
/lang?file=en.php
/api/files?path=uploads/a.png
/export?document=report
/load?template=main
/render?view=profile
```

---

## مؤشرات تدل إن فيه LFI أو Path Traversal

ممكن تشك في الثغرة لو شفت:

```text
اسم ملف في URL
امتداد ملف مثل .php أو .html أو .pdf
رسالة error فيها full path
parameter اسمه page أو file أو path
اختلاف response لما تغير اسم الملف
ظهور warning من PHP include أو require
```

أمثلة errors مفيدة:

```text
Warning: include(home.php): failed to open stream
No such file or directory
failed to open stream
Permission denied
open_basedir restriction in effect
```

وجود errors دي ممكن يساعدك تعرف:

```text
مسار التطبيق
لغة البرمجة
هل فيه include ولا read فقط
هل فيه قيود open_basedir
```

---

## ملفات تستخدم كإثبات آمن

في Bug Bounty ما تبدأش بملفات حساسة جدا.

استخدم ملفات آمنة نسبيا لإثبات الثغرة.

### Linux

```text
/etc/hostname
/etc/hosts
/proc/version
/proc/self/cmdline
```

### Windows

```text
C:\Windows\win.ini
C:\Windows\System32\drivers\etc\hosts
```

> [!tip] ملاحظة عن /etc/passwd  
> `/etc/passwd` غالبا لا يحتوي passwords في الأنظمة الحديثة، لكنه ما زال ملف system حساس نسبيا.  
> لو استخدمته كإثبات، اعرض أقل سطور ممكنة فقط.

ملفات لا تبدأ بها في production:

```text
/etc/shadow
.env
config.php
database.yml
private keys
backup files
user documents
```

إلا لو البرنامج يسمح، وبرضه خليك في أقل دليل ممكن.

---

## Payloads أساسية لـ Path Traversal

### Linux

```text
../../../../etc/hosts
../../../etc/hosts
../../../../etc/passwd
/etc/hosts
/etc/hostname
```

### Windows

```text
..\..\..\windows\win.ini
..\..\..\Windows\win.ini
..%5c..%5c..%5cwindows%5cwin.ini
C:\Windows\win.ini
C:\Windows\System32\drivers\etc\hosts
```

### Payloads مع URL Encoding

```text
..%2f..%2f..%2f..%2fetc%2fhosts
..%2F..%2F..%2F..%2Fetc%2Fhosts
%2e%2e%2f%2e%2e%2fetc%2fhosts
```

---

## Bypasses متقدمة

لو payload عادي ما اشتغلش، ممكن يكون فيه filter أو WAF أو normalization.

### 1. Double Encoding

لو التطبيق بيفك الترميز مرتين:

```text
..%252f..%252f..%252fetc%252fhosts
```

الفكرة:

```text
%252f -> بعد أول decode تبقى %2f
%2f -> بعد ثاني decode تبقى /
```

---

### 2. Backslash على Windows

بعض التطبيقات تقبل `\` بدل `/`.

```text
..\..\..\windows\win.ini
..%5c..%5c..%5cwindows%5cwin.ini
```

---

### 3. Dot-Dot-Slash Variations

بعض الفلاتر الضعيفة بتحذف `../` مرة واحدة فقط.

جرب:

```text
....//....//....//etc/hosts
..../..../..../etc/hosts
..././..././..././etc/hosts
```

مثال:

```text
....//
```

لو الفلتر حذف `../` من النص، ممكن تفضل:

```text
../
```

---

### 4. Absolute Path

أحيانا التطبيق يمنع `../` لكن يسمح بمسار كامل.

Linux:

```text
/etc/hosts
/var/log/nginx/access.log
/var/www/html/index.php
```

Windows:

```text
C:\Windows\win.ini
C:\Windows\System32\drivers\etc\hosts
```

---

### 5. Null Byte

قديم جدا، وكان يستخدم لقطع الامتداد الإجباري.

```text
../../../../etc/passwd%00
../../../../etc/passwd%00.jpg
```

مثال vulnerable قديم:

```php
include($_GET["page"] . ".php");
```

في إصدارات قديمة جدا من PHP، كان ممكن null byte يقطع `.php`.

> [!note]  
> Null byte غالبا لا يعمل في الإصدارات الحديثة، لكنه مهم تاريخيا.

---

### 6. Forced Extension Bypass

لو التطبيق بيضيف extension تلقائيا:

```php
include($_GET["page"] . ".php");
```

في بيئات الاختبار، ممكن تراجع هل فيه normalization أو parsing غريب، لكن في الأنظمة الحديثة غالبا الطرق القديمة مش هتنفع.

أمثلة أفكار اختبار آمنة:

```text
هل الامتداد بيتضاف فعلا؟
هل response بيختلف؟
هل error بيكشف المسار النهائي؟
هل فيه allowlist ولا blocklist؟
```

---

### 7. Unicode / UTF-8 Encoding

بعض الخوادم القديمة أو misconfigured ممكن تفسر encoding غريب كـ `/`.

```text
..%c0%af..%c0%af..%c0%afetc/passwd
```

> [!warning]  
> دي bypass تاريخية وغالبا مش هتشتغل في أنظمة حديثة، لكنها بتظهر أحيانا في labs أو أنظمة قديمة.

---

## مثال Vulnerable Code - PHP LFI

الكود ده خطر لأنه بيعمل include من user input مباشرة:

```php
<?php
$page = $_GET["page"] ?? "home.php";
include($page);
?>
```

Request طبيعي:

```http
GET /index.php?page=home.php
```

Request خطر:

```http
GET /index.php?page=../../../../etc/hosts
```

سبب الثغرة:

```text
User controls include path directly.
```

---

## Secure Code - PHP LFI

الحل الأفضل إنك تستخدم allowlist.

```php
<?php
$pages = [
    "home" => __DIR__ . "/pages/home.php",
    "about" => __DIR__ . "/pages/about.php",
    "contact" => __DIR__ . "/pages/contact.php",
];

$page = $_GET["page"] ?? "home";

if (!array_key_exists($page, $pages)) {
    http_response_code(404);
    exit("Page not found");
}

include($pages[$page]);
?>
```

هنا المستخدم لا يرسل path.

هو يرسل key فقط:

```text
home
about
contact
```

والتطبيق هو اللي يحدد الملف الحقيقي.

---

## مثال Vulnerable Code - File Download

```php
<?php
$file = $_GET["file"] ?? "";
$path = __DIR__ . "/files/" . $file;

if (file_exists($path)) {
    readfile($path);
}
?>
```

Request خطر:

```http
GET /download.php?file=../../../../etc/hosts
```

المشكلة:

```text
Direct path concatenation
```

يعني التطبيق بيركب المسار من input مباشرة.

---

## Secure Code - File Download

استخدم `realpath` وتأكد إن المسار النهائي داخل folder المسموح.

```php
<?php
$baseDir = realpath(__DIR__ . "/files");
$file = $_GET["file"] ?? "";
$target = realpath($baseDir . "/" . $file);

if ($target === false || !str_starts_with($target, $baseDir . DIRECTORY_SEPARATOR)) {
    http_response_code(400);
    exit("Invalid file");
}

if (!is_file($target)) {
    http_response_code(404);
    exit("File not found");
}

header("Content-Type: application/octet-stream");
header("Content-Disposition: attachment; filename=\"" . basename($target) . "\"");
readfile($target);
?>
```

المهم هنا:

```text
realpath -> يحول المسار لشكل نهائي حقيقي
starts_with -> يتأكد إن الملف داخل baseDir
basename -> يمنع تسريب path داخلي في اسم التحميل
```

---

## مثال Vulnerable Code - Node.js

```js
const express = require("express");
const path = require("path");

const app = express();

app.get("/download", (req, res) => {
  const file = req.query.file || "";
  res.sendFile(path.join(__dirname, "files", file));
});

app.listen(3000);
```

المشكلة:

```text
file يأتي من المستخدم
وممكن يحتوي ../
```

---

## Secure Code - Node.js

```js
const express = require("express");
const path = require("path");

const app = express();
const baseDir = path.resolve(__dirname, "files");

app.get("/download", (req, res) => {
  const file = req.query.file || "";
  const target = path.resolve(baseDir, file);

  if (!target.startsWith(baseDir + path.sep)) {
    return res.status(400).send("Invalid file");
  }

  res.download(target);
});

app.listen(3000);
```

الأفضل كمان:

```text
استخدم file IDs بدل أسماء ملفات
اربط كل ملف بمالك في database
اعمل authorization قبل التحميل
افصل uploads عن web root
```

---

## PHP Wrappers في LFI

PHP فيه wrappers ممكن تظهر في LFI.

أشهرهم:

```text
php://filter
php://input
data://
zip://
phar://
file://
```

---

## php://filter

ده أشهر wrapper مفيد في LFI.

بيستخدم غالبا لقراءة source code بدون تنفيذه.

مثال:

```text
?page=php://filter/convert.base64-encode/resource=index.php
```

الفكرة:

```text
بدل ما PHP ينفذ index.php
هيعرضه Base64
وبعدين تفكه وتقرأ الكود
```

ده ممكن يكشف:

```text
source code
database credentials
API keys
secret keys
internal paths
```

> [!warning]  
> قراءة source code ممكن تكشف أسرار.  
> في Bug Bounty، اقرأ أقل ملف يثبت المشكلة فقط.

---

## php://input

في بعض الإعدادات القديمة أو الخطرة، ممكن التطبيق يعمل include لمحتوى request body.

الفكرة العامة:

```text
التطبيق يعمل include لمصدر بيانات المستخدم مباشرة
```

> [!warning]  
> استخدام `php://input` لتشغيل كود يعتبر RCE.  
> لا تستخدمه على production إلا لو البرنامج سامح صراحة.  
> في التقارير الحقيقية، اشرح الـ risk بدون تنفيذ أوامر.

---

## data://

ده wrapper يسمح بإدخال data مباشرة في المسار.

غالبا يحتاج:

```text
allow_url_include = On
```

> [!warning]  
> ده ممكن يتحول لـ RCE في بيئات معينة.  
> استخدمه فقط في labs أو بإذن صريح.  
> في المقال ده هنركز على الفهم والإثبات الآمن.

---

## zip:// و phar://

دول wrappers متقدمة.

### zip://

ممكن يستخدم لقراءة أو include ملف داخل archive.

الفكرة:

```text
include ملف موجود داخل archive
```

### phar://

`phar://` خطير في بعض الحالات لأنه ممكن يدخل في مشاكل deserialization لو التطبيق بيتعامل مع metadata بطريقة غلط.

> [!note]  
> دي مواضيع متقدمة.  
> في Bug Bounty، اشرح الـ risk لو ظهر، لكن ما تعملش استغلال مؤذي.

---

## Attack Chain عملية لاختبار LFI

دي سلسلة منظمة تفكر بيها:

```text
1. دور على parameter له علاقة بملف
2. جرب قيمة طبيعية وشوف response
3. جرب traversal بسيط
4. لو فشل، جرب encoding و bypasses
5. أثبت القراءة بملف آمن
6. حدد هل endpoint read ولا include
7. شوف هل تقدر تقرأ source code بشكل محدود
8. شوف هل تقدر تقرأ logs
9. جرب marker آمن في User-Agent
10. اقرأ log وشوف هل marker ظهر
11. اكتب التقرير بأقل دليل ممكن
```

شكل أبسط:

```text
Parameter
  -> Traversal
  -> Safe file read
  -> Source code read?
  -> Logs readable?
  -> Marker in log?
  -> Report impact safely
```

---

## ما هو Log Poisoning؟

Log Poisoning يعني إنك تكتب نص تتحكم فيه داخل log file، وبعدين تستخدم LFI لقراءة أو include الـ log.

السلسلة:

```text
Send controlled text in request header
  -> Server writes it into access.log
  -> LFI reads/includes access.log
  -> لو التطبيق بيتعامل مع الملف بطريقة خطرة، ممكن الخطر يزيد
```

أشهر أماكن logs:

```text
/var/log/apache2/access.log
/var/log/apache2/error.log
/var/log/httpd/access_log
/var/log/httpd/error_log
/var/log/nginx/access.log
/var/log/nginx/error.log
/var/log/auth.log
/var/log/syslog
```

---

## Log Poisoning إثبات آمن

بدل ما تكتب كود، اكتب marker نصي.

Request:

```http
GET / HTTP/1.1
Host: lab.local
User-Agent: LFI-POC-12345
```

بعدها جرب تقرأ access log:

```text
/index.php?page=../../../../var/log/apache2/access.log
```

لو ظهر:

```text
LFI-POC-12345
```

يبقى أثبت إنك:

```text
تقدر تكتب input داخل log
وتقدر تقرأ log من خلال LFI
```

ده كفاية غالبا في Bug Bounty لإثبات risk.

---

## Log Poisoning - الدليل العملي الآمن

الفكرة العملية مش بس إنك تعرف إن فيه logs، لكن تعرف تختبرها بطريقة منظمة وآمنة.

### الخطوة 1: اختار marker واضح

استخدم marker فريد عشان تعرفه بسهولة في الـ response.

مثال:

```text
LFI-POC-2026-UNIQUE-MARKER
```

حط الـ marker في headers ممكن تتسجل:

```text
User-Agent
Referer
X-Forwarded-For
Cookie
Request path
Host
```

أكثر مكان شائع:

```http
User-Agent: LFI-POC-2026-UNIQUE-MARKER
```

---

### الخطوة 2: ابعت request يسجل الـ marker

مثال آمن:

```http
GET / HTTP/1.1
Host: lab.local
User-Agent: LFI-POC-2026-UNIQUE-MARKER
Referer: https://example.com/LFI-POC-2026-UNIQUE-MARKER
```

بعد كده دور على الـ marker في logs عبر LFI.

---

### الخطوة 3: جرب مسارات logs الشائعة

Apache:

```text
/var/log/apache2/access.log
/var/log/apache2/error.log
/var/log/httpd/access_log
/var/log/httpd/error_log
```

Nginx:

```text
/var/log/nginx/access.log
/var/log/nginx/error.log
```

Linux system logs:

```text
/var/log/syslog
/var/log/messages
/var/log/auth.log
```

Windows IIS:

```text
C:\inetpub\logs\LogFiles\
```

> [!warning]  
> قراءة system logs قد تكشف بيانات حساسة.  
> استخدم أقل دليل ممكن، ولا تعرض محتوى كبير في التقرير.

---

### الخطوة 4: افهم هل الـ log مقروء ولا لا

لو القراءة فشلت، الأسباب الشائعة:

```text
المسار غلط
صلاحيات القراءة ممنوعة
التطبيق داخل container ومش شايف logs
open_basedir مانع الوصول
الـ log format لا يسجل الـ header اللي استخدمته
الـ log كبير جدا وبيسبب timeout أو memory issue
```

---

## مشاكل Log Poisoning الشائعة وحلولها الآمنة

### المشكلة 1: الـ marker لا يظهر في الـ log

الأسباب المحتملة:

```text
الـ User-Agent مش بيتسجل
الـ Referer مش بيتسجل
فيه proxy بيغير headers
الـ request ما وصلش لنفس السيرفر
الـ log path مختلف
```

حلول آمنة:

```text
جرب Header مختلف مثل Referer أو X-Forwarded-For
جرب request path فيه marker
جرب error.log بدل access.log
جرب تعمل request يسبب 404 آمن عشان يظهر في error/access logs
```

مثال request path آمن:

```http
GET /LFI-POC-2026-UNIQUE-MARKER HTTP/1.1
Host: lab.local
User-Agent: NormalBrowser
```

---

### المشكلة 2: الـ log كبير جدا

أحيانا access.log بيكون كبير جدا، ولما التطبيق يحاول يقرأه الصفحة تقع أو تعمل timeout.

حلول آمنة:

```text
جرب error.log لأنه أحيانا أصغر
جرب log خاص بالتطبيق لو معروف
استخدم marker فريد وسجل وقت الاختبار في التقرير
لا تسحب محتوى log كامل
اكتفي بصورة أو جزء صغير يظهر marker
```

في التقرير، اكتب:

```text
The response contains the unique marker sent in the User-Agent.
Only a small excerpt was captured as evidence.
```

---

### المشكلة 3: Permission denied

لو ظهر:

```text
Permission denied
```

معناه إن مستخدم التطبيق مش قادر يقرأ ملف الـ log.

ده في حد ذاته يقلل احتمال التصعيد، لكنه لا يلغي أصل LFI لو تقدر تقرأ ملفات أخرى.

اكتب في الملاحظات:

```text
Access to web server logs appears restricted by file permissions.
However, the LFI still allows reading other accessible local files.
```

---

### المشكلة 4: الـ marker يظهر encoded أو متغير

أحيانا الـ header يتسجل بشكل مختلف:

```text
spaces تتحول
quotes تتغير
characters تتعمل escaping
proxy يضيف أو يحذف أجزاء
```

عشان كده استخدم marker بسيط:

```text
LFI-POC-ABC123
```

وتجنب في الإثبات الآمن:

```text
رموز معقدة
أسطر جديدة
payloads تنفيذية
characters ممكن تعمل parsing غريب
```

---

### المشكلة 5: التطبيق يقرأ log لكن لا يعمل include فعلي

ممكن يكون endpoint بيقرأ الملف كنص فقط.

في الحالة دي عندك:

```text
File Read + Log Exposure
```

مش بالضرورة RCE.

اكتب في التقرير:

```text
The endpoint reads local log files and exposes attacker-controlled log entries.
No code execution was attempted.
```

---

## لماذا Log Poisoning خطر؟

لأن بعض التطبيقات بتستخدم include بشكل مباشر.

لو التطبيق عمل include لملف log يحتوي على content يتحكم فيه المستخدم، الخطر ممكن يزيد حسب اللغة والإعدادات.

لكن في التقارير الحقيقية، الأفضل تقف عند:

```text
Attacker-controlled content is written to logs and readable/includable via LFI.
```

وتشرح إن ده ممكن يتصعد حسب البيئة، بدون تنفيذ أوامر.

> [!warning]  
> لا تكتب Web Shell داخل logs على production.  
> لا تستخدم reverse shell.  
> لا تنفذ أوامر.  
> استخدم marker نصي فقط.

---

## Log Poisoning في خدمات مختلفة

Log Poisoning مش مرتبط بس Apache و Nginx.

أي خدمة بتكتب input من المستخدم في ملف log، والملف ده مقروء عبر LFI، ممكن يكون فيه risk.

### Apache

مسارات شائعة:

```text
/var/log/apache2/access.log
/var/log/apache2/error.log
/var/log/httpd/access_log
/var/log/httpd/error_log
```

Headers ممكن تظهر:

```text
User-Agent
Referer
Host
Request path
```

### Nginx

مسارات شائعة:

```text
/var/log/nginx/access.log
/var/log/nginx/error.log
```

مش كل header بيظهر حسب log format.

### IIS على Windows

مسارات شائعة:

```text
C:\inetpub\logs\LogFiles\
```

ملفات IIS logs غالبا بتكون بصيغ مختلفة وداخل folders حسب site id.

### Auth Logs

أحيانا محاولات تسجيل الدخول الفاشلة بتظهر في:

```text
/var/log/auth.log
```

لكن اختبار ده على production حساس جدا وممكن يكون خارج scope.

الطريقة الآمنة:

```text
لا تختبر auth.log إلا لو lab أو تصريح واضح
استخدم marker فقط
لا تحاول تشغيل أوامر
```

### FTP Logs

خدمات FTP ممكن تسجل usernames أو محاولات الدخول.

ده مفيد كفكرة في labs، لكن في Bug Bounty الحقيقي لازم تتأكد إن الخدمة داخل scope.

### Application Logs

أحيانا التطبيق نفسه عنده logs داخل:

```text
/var/www/app/logs/
/app/storage/logs/
/var/log/app/
```

أمثلة:

```text
Laravel: storage/logs/laravel.log
Django: app-specific logs حسب الإعداد
Node.js: logs/app.log حسب المشروع
```

لو قدرت تكتب marker في application log وتقرأه بالـ LFI، ده دليل قوي.

---

## الفرق بين Log Injection و Log Poisoning

|النوع|الشرح|
|---|---|
|Log Injection|إدخال نص يغير شكل الـ log أو يضيف سطور مزيفة|
|Log Poisoning|إدخال محتوى داخل log لاستخدامه لاحقا في استغلال مثل LFI|

مثال Log Injection:

```text
User-Agent: normal-user
```

لو التطبيق لا ينظف new lines، المهاجم ممكن يضيف سطور مزيفة.

أما Log Poisoning فهدفه:

```text
كتابة محتوى داخل log
ثم قراءته أو include له
```

---

## LFI to RCE - الفكرة فقط

LFI ممكن تتحول إلى RCE في حالات معينة.

|الطريقة|الفكرة|
|---|---|
|Log Poisoning|تكتب محتوى في log ثم التطبيق يعمل include للـ log|
|Upload + LFI|ترفع ملف ثم التطبيق يقرأه أو يضمه|
|Session Files|تتحكم في session data ثم التطبيق يقرأ session file|
|php://input|include لمحتوى request body في إعدادات معينة|
|data://|include مباشر لمحتوى data في إعدادات معينة|
|/proc/self/environ|headers قد تظهر في environment في بيئات قديمة|
|phar://|ممكن يدخل في deserialization في سيناريوهات معينة|

> [!warning]  
> في Bug Bounty الحقيقي، لا تصعد إلى RCE إلا بإذن صريح.  
> غالبا إثبات القراءة أو marker كفاية.

---

## LFI مع File Upload

لو فيه upload feature ومعاك LFI، ممكن تحصل سلسلة:

```text
Upload harmless proof file
  -> Get storage path
  -> Include or read file using LFI
```

مثال آمن:

```text
Upload file: proof.txt
Content: LFI-UPLOAD-POC
```

ثم:

```text
/index.php?page=uploads/proof.txt
```

لو ظهر:

```text
LFI-UPLOAD-POC
```

فده يثبت إن LFI يقدر يوصل لملفات مرفوعة.

لو التطبيق بيتعامل مع الملفات المرفوعة بطريقة تنفيذية، ده أخطر، لكن ما تستخدمش payload تنفيذي في production.

---

## Session Files

في PHP، الـ session ممكن تتخزن في ملفات.

مسارات شائعة:

```text
/var/lib/php/sessions/sess_<PHPSESSID>
/tmp/sess_<PHPSESSID>
```

الفكرة:

```text
لو تقدر تتحكم في قيمة داخل session
والتطبيق يقدر يقرأ أو يعمل include لملف session
ممكن يحصل escalation في بيئات vulnerable
```

مثال آمن للتفكير:

```text
Set username = LFI-SESSION-POC
Read session file via LFI
Check if marker appears
```

> [!warning]  
> لا تحقن كود تنفيذي في session على production.  
> استخدم marker نصي فقط.

---

## /proc/self/environ

في Linux، الملف ده ممكن يحتوي environment variables للعملية.

```text
/proc/self/environ
```

في بيئات قديمة، بعض headers ممكن تظهر فيه.

اختبار آمن:

```text
User-Agent: LFI-ENV-POC
```

ثم:

```text
?page=../../../../proc/self/environ
```

لو marker ظهر، يبقى فيه exposure.

> [!note]  
> السيناريو ده أقل شيوعا في الأنظمة الحديثة.

---

## قراءة Source Code عبر LFI

لو عندك LFI في PHP، ممكن تجرب `php://filter` لقراءة source code.

مثال:

```text
?page=php://filter/convert.base64-encode/resource=index.php
```

بعد ما يظهر Base64، تفكه محليا وتشوف الكود.

لكن في Bug Bounty:

```text
اقرأ أقل ملف ممكن
لا تسحب المشروع كله
لا تقرأ secrets إلا لو محتاج تثبت impact وبأقل قدر
```

---

## LFI في Frameworks

LFI ممكن تظهر بأشكال مختلفة حسب framework.

### Laravel

أماكن ممكن تراجعها:

```text
view/template selection
file download routes
storage file access
language files
theme switching
logs inside storage/logs
```

أخطاء شائعة:

```text
return view($request->page)
Storage::download($request->file)
include resource_path($request->template)
```

الحماية:

```text
استخدم named views فقط
استخدم Storage facade مع authorization
لا تسمح للمستخدم يمرر path مباشر
```

---

### Django

Django مش بيعمل include PHP، لكن ممكن يحصل path traversal في:

```text
file download views
media files
template selection
static file serving misconfigurations
```

مثال خطر:

```python
path = request.GET.get("file")
return FileResponse(open(path, "rb"))
```

الحماية:

```text
استخدم storage backends
استخدم allowlist
تأكد من ownership والـ permissions
```

---

### Ruby on Rails

ممكن تظهر في:

```text
send_file
render template
file attachments
theme/template selection
```

مثال خطر:

```ruby
send_file params[:path]
```

الحماية:

```text
استخدم IDs
استخدم ActiveStorage
تحقق من صلاحيات المستخدم
لا تستخدم params كمسار مباشر
```

---

### Node.js / Express

أماكن شائعة:

```text
res.sendFile
res.download
static file serving
template rendering
```

الحماية:

```text
path.resolve
base directory check
file IDs
authorization
```

---

## أدوات الاختبار

الاختبار اليدوي مهم، لكن الأدوات بتسرع.

> [!warning]  
> استخدم الأدوات فقط على targets داخل scope.  
> قلل rate.  
> لا تعمل scanning عنيف.

---

### Burp Suite

مفيد في:

```text
تعديل parameters
تجربة payloads
Repeater
Intruder
Payload encoding
Comparing responses
```

خطوات بسيطة:

```text
1. افتح request في Repeater
2. غير parameter مثل file أو page
3. جرب payload آمن
4. راقب status code و response length
5. جرب encoding لو فيه filtering
```

---

### ffuf

مثال لاختبار payloads:

```bash
ffuf -u "https://target.example/index.php?page=FUZZ" -w lfi-payloads.txt
```

مع فلترة size متكرر:

```bash
ffuf -u "https://target.example/index.php?page=FUZZ" -w lfi-payloads.txt -fs 1234
```

---

### nuclei

ممكن يستخدم templates جاهزة أو مخصصة.

مثال عام:

```bash
nuclei -u https://target.example -tags lfi
```

استخدمه بحذر، وراجع النتائج يدويا.

---

### feroxbuster

مفيد لاكتشاف paths و files.

```bash
feroxbuster -u https://target.example
```

---

### gowitness

مفيد لتوثيق screenshots للنتائج.

```bash
gowitness single https://target.example
```

لكن ما تصورش بيانات حساسة زيادة عن المطلوب.

---

## Wordlists مفيدة

ممكن تستخدم wordlists فيها:

```text
LFI payloads
common Linux files
common Windows files
log paths
framework config paths
backup file names
```

أمثلة categories:

```text
/etc/hosts
/etc/hostname
C:\Windows\win.ini
/var/log/apache2/access.log
/var/log/nginx/access.log
.env
config.php
settings.py
database.yml
```

> [!warning]  
> لا تسحب ملفات secrets لمجرد الفضول.  
> استخدم أقل دليل يثبت impact.

---

## Case Study تعليمية - LFI في تغيير اللغة

الهدف:

```text
تطبيق يسمح بتغيير اللغة عبر:
?lang=en.php
```

### الاكتشاف

لاحظ الباحث إن parameter اسمه:

```text
lang
```

وده مؤشر قوي لأنه ممكن يدخل في include لملفات اللغة.

### الاختبار الأول

جرب:

```text
?lang=../../../../etc/hostname
```

النتيجة فشل.

ممكن السبب:

```text
فلتر يمنع ../
أو التطبيق يضيف .php
أو فيه normalization
```

### تجربة bypass

جرب double encoding:

```text
?lang=..%252f..%252f..%252fetc%252fhostname
```

النتيجة نجح وقرأ hostname.

### إثبات آمن

استخدم ملف بسيط:

```text
/etc/hostname
```

وعرض جزء صغير فقط.

### فحص Log Poisoning بشكل آمن

جرب marker في User-Agent:

```http
User-Agent: LFI-POC-LANG-123
```

ثم حاول قراءة:

```text
/var/log/apache2/access.log
```

لو ظهر marker، يبقى فيه Log Poisoning risk.

### التقرير

التقرير يوضح:

```text
parameter: lang
نوع الثغرة: LFI / Path Traversal
الدليل: قراءة /etc/hostname
إثبات إضافي: ظهور marker داخل log لو متاح
التأثير: إمكانية قراءة ملفات داخلية وربما source code
الإصلاح: allowlist للغات مثل en/ar فقط
```

---

## Impact في Bug Bounty

الـ impact يعتمد على:

```text
نوع الملف المقروء
هل الملف حساس؟
هل يمكن قراءة source code؟
هل يوجد credentials؟
هل يمكن التصعيد؟
هل البيانات تخص users؟
```

|النتيجة|Impact المتوقع|
|---|---|
|قراءة `/etc/hosts` أو hostname فقط|Low غالبا|
|قراءة source code|Medium أو High حسب المحتوى|
|قراءة config فيه credentials|High|
|قراءة private user files|High|
|قراءة secrets أو API keys|High إلى Critical|
|LFI + Log Poisoning قابل للتصعيد في بيئة معينة|High إلى Critical حسب الإثبات والسياسة|
|Path Traversal في download يكشف documents|High حسب البيانات|
|RFI يؤدي لتنفيذ كود|Critical غالبا|

---

## كيف تكتب Report جيد؟

لا تكتب فقط:

```text
LFI found
```

اكتب التفاصيل.

لازم التقرير يحتوي:

```text
Title
Endpoint
Parameter
Payload
Steps to reproduce
Evidence
Impact
Risk of escalation
Suggested fix
```

---

## Report مثال - Path Traversal

```text
Title:
Path Traversal in file download allows reading local server files

Endpoint:
GET /download?file=

Affected Parameter:
file

Steps:
1. Login as a normal user.
2. Open /download?file=report.pdf and confirm normal file download.
3. Change file parameter to ../../../../etc/hosts.
4. The response returns content from the server hosts file.

Evidence:
Only a small non-sensitive part of /etc/hosts was used to confirm the issue.

Impact:
An attacker can read local files outside the intended download directory. Depending on readable files, this may expose source code, configuration files, credentials, or user data.

Suggested Fix:
Use file IDs or an allowlist, canonicalize paths with realpath, and ensure the final path stays inside the allowed base directory.
```

---

## Report مثال - LFI with Log Poisoning Risk

```text
Title:
LFI allows reading web server access logs containing attacker-controlled input

Endpoint:
GET /index.php?page=

Affected Parameter:
page

Steps:
1. Send a request with User-Agent: LFI-POC-12345.
2. Request /index.php?page=../../../../var/log/apache2/access.log.
3. The response contains LFI-POC-12345 from the access log.

Evidence:
The marker LFI-POC-12345 appears in the response after being written to the access log.

Impact:
The application can read or include local log files that contain attacker-controlled input. In vulnerable configurations, this pattern may increase the risk of further exploitation. At minimum, it exposes internal logs.

Suggested Fix:
Do not include files based on user-controlled paths. Use an allowlist of page names, block access to log paths, and run the application with least-privilege filesystem permissions.
```

---

## Report مثال - php://filter Source Code Disclosure

```text
Title:
LFI allows source code disclosure via php://filter wrapper

Endpoint:
GET /index.php?page=

Affected Parameter:
page

Steps:
1. Request /index.php?page=php://filter/convert.base64-encode/resource=index.php.
2. The response returns Base64 encoded source code.
3. Decode the response locally and confirm it contains PHP source code.

Impact:
An attacker can read application source code. This may reveal business logic, credentials, API keys, internal endpoints, or additional vulnerabilities.

Suggested Fix:
Disable unsafe file inclusion, use allowlisted page names only, and restrict PHP wrappers where possible.
```

---

## Report مثال - Log File Exposure with Marker

```text
Title:
Local File Inclusion allows reading application logs containing user-controlled input

Endpoint:
GET /view?page=

Affected Parameter:
page

Steps:
1. Send a request with a unique marker in the User-Agent:
   LFI-POC-2026-UNIQUE-MARKER
2. Use the LFI endpoint to read the web server log.
3. Confirm that the response contains the same marker.

Evidence:
Only the line containing the unique marker was captured.

Impact:
An attacker can access internal log files and confirm that user-controlled input is stored there. This may expose internal paths, requests, headers, user identifiers, tokens, or other operational data depending on the log contents.

Suggested Fix:
Prevent user-controlled file inclusion, block access to log paths, reduce filesystem permissions, and avoid logging sensitive data.
```

---

## أفضل طرق الحماية

### 1. لا تستخدم user input كمسار مباشر

خطأ:

```php
include($_GET["page"]);
```

صح:

```php
include($pages[$page]);
```

---

### 2. استخدم Allowlist

مثال:

```text
home -> /pages/home.php
about -> /pages/about.php
contact -> /pages/contact.php
```

المستخدم يختار:

```text
home
```

مش:

```text
/pages/home.php
```

---

### 3. استخدم File IDs بدل paths

بدل:

```text
/download?file=invoice.pdf
```

استخدم:

```text
/download?file_id=123
```

والـ backend يجيب المسار من database بعد authorization.

---

### 4. تحقق من صلاحيات المستخدم

حتى لو المسار آمن، لازم تسأل:

```text
هل المستخدم ده يملك الملف؟
هل عنده permission يشوفه؟
هل الملف تابع لنفس account أو organization؟
```

---

### 5. استخدم realpath أو path.resolve

لازم تتحقق إن final path داخل base directory.

```text
baseDir = /app/files
target = /app/files/report.pdf
```

ممنوع target يبقى:

```text
/etc/hosts
/var/www/config.php
```

---

### 6. افصل uploads عن web root

الأفضل:

```text
/app/storage/uploads
```

وليس:

```text
/var/www/html/uploads
```

كده الملفات المرفوعة ما تتنفذش مباشرة من الويب.

---

### 7. لا تعمل include لملفات uploads أو logs

دي قاعدة مهمة:

```text
Never include logs
Never include uploads
Never include user-controlled files
```

---

### 8. قلل صلاحيات السيرفر

شغل التطبيق بمستخدم محدود.

المستخدم اللي مشغل web server ما يكونش قادر يقرأ:

```text
/etc/shadow
private keys
other users files
backup files
```

---

### 9. إعدادات PHP

اقفل الإعدادات الخطرة لو مش محتاجها:

```ini
allow_url_include = Off
allow_url_fopen = Off
display_errors = Off
```

ممكن كمان تقيد التطبيق بـ:

```ini
open_basedir
```

لكن ما تعتمدش عليها وحدها.

---

### 10. لا تعرض full paths في errors

بدل ما تعرض:

```text
Warning: include(/var/www/app/pages/test.php)
```

اعرض رسالة عامة:

```text
Page not found
```

وسجل التفاصيل داخليا فقط.

---

### 11. راقب وسجل محاولات Traversal

سجل requests اللي فيها:

```text
../
..\
%2e%2e
%2f
%5c
php://
data://
```

واستخدم rate limiting و alerts لو فيه scanning واضح.

---

## Defense in Depth

الحماية المتعمقة معناها ما تعتمدش على حل واحد فقط.

### طبقة التصميم

```text
استخدم IDs بدل paths
افصل uploads عن web root
صمم routes واضحة بدل dynamic includes
```

### طبقة التحقق

```text
Allowlist
رفض ../ و / و \ و null bytes
التحقق من extension
التحقق من filename
```

### طبقة معالجة الملفات

```text
realpath
canonicalization
base directory check
basename عند الحاجة
```

### طبقة الصلاحيات

```text
Authorization قبل file access
Least privilege
منع قراءة ملفات خارج نطاق التطبيق
```

### طبقة إعدادات السيرفر

```text
disable unsafe PHP settings
containerization
chroot/jail عند الحاجة
separate users لكل تطبيق
```

### طبقة المراقبة

```text
logging
alerting
rate limiting
WAF rules
monitor traversal attempts
```

---

## أخطاء شائعة في الاختبار

```text
قراءة ملفات حساسة أكثر من اللازم
استخدام RCE payload على production
نسيان authorization impact
الاعتماد على /etc/passwd فقط
عدم توضيح هل الثغرة read ولا include
عدم ذكر endpoint والparameter بدقة
عدم تقديم fix عملي
عدم تجربة logs بأمان باستخدام marker
عدم الانتباه لمشكلة حجم logs أو permissions
```

---

## Checklist للاختبار

```text
[ ] هل يوجد parameter اسمه file, page, path, template, lang؟
[ ] هل input يظهر كاسم ملف أو صفحة؟
[ ] هل response يتغير عند تغيير اسم الملف؟
[ ] هل تظهر errors فيها full path؟
[ ] هل يمكن استخدام ../؟
[ ] هل يمكن استخدام absolute path؟
[ ] هل Windows paths محتملة؟
[ ] هل يوجد extension يضاف تلقائيا؟
[ ] هل يعمل URL encoding؟
[ ] هل يعمل double encoding؟
[ ] هل تنجح dot-dot-slash variations؟
[ ] هل يمكن قراءة ملف آمن مثل /etc/hosts أو win.ini؟
[ ] هل endpoint يعمل read فقط أم include؟
[ ] هل يمكن استخدام php://filter؟
[ ] هل يمكن قراءة source code؟
[ ] هل يمكن الوصول إلى logs؟
[ ] هل يمكن كتابة marker في User-Agent ثم قراءته؟
[ ] هل marker لا يظهر بسبب header مختلف أو log format؟
[ ] هل access.log كبير جدا؟ هل error.log أصغر؟
[ ] هل هناك Permission denied على logs؟
[ ] هل يوجد application log يمكن اختباره بmarker آمن؟
[ ] هل يوجد upload feature يمكن ربطه مع LFI؟
[ ] هل session files قابلة للقراءة؟
[ ] هل /proc/self/environ مفيد؟
[ ] هل يوجد RFI محتمل؟
[ ] هل فيه SSRF confusion ولا الثغرة file-based فعلا؟
[ ] هل توجد صلاحيات تمنع قراءة ملفات حساسة؟
[ ] هل يمكن الوصول لملفات تخص مستخدمين آخرين؟
[ ] هل التقرير يحتوي endpoint و payload و impact و fix؟
```

---

## ملخص سريع

```text
Path Traversal:
حيلة استخدام مسارات مثل ../ للخروج من folder المسموح.

LFI:
ثغرة تسمح بقراءة أو include ملف local من السيرفر.

RFI:
ثغرة تسمح بعمل include لملف remote من سيرفر خارجي.

Log Poisoning:
إدخال نص داخل log ثم قراءته أو include له من خلال LFI.

LFI to RCE:
تصعيد ممكن في بيئات معينة، لكن لا تختبره على production إلا بإذن صريح.
```

أفضل طريقة تفهم الموضوع:

```text
Parameter يتحكم في path
  -> جرب traversal
  -> أثبت بملف آمن
  -> حدد read ولا include
  -> افحص source/logs بشكل محدود
  -> استخدم marker لإثبات Log Poisoning
  -> اكتب report واضح
```

وافتكر دايما:

```text
Safe proof أفضل من استغلال عدواني.
أقل دليل كافي أفضل من قراءة ملفات كثيرة.
التزامك بالـ scope أهم من إثبات impact بطريقة خطرة.
```
