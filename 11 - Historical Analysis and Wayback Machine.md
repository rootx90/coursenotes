---
title: Historical Analysis and Wayback Machine
date: 2026-04-26
tags:
  - historical-analysis
  - wayback-machine
  - archive
  - urls
  - recon
  - bug-bounty
  - pentest
---

# Historical Analysis and Wayback Machine


## ما هو Historical Analysis؟

Historical Analysis يعني تحليل النسخ القديمة من الموقع والروابط التي ظهرت في الماضي.

الفكرة أن الموقع قد يكون تغير، لكن الأرشيفات ومحركات البحث قد تحتفظ بـ:

- URLs قديمة.
- API endpoints.
- Parameters.
- Subdomains.
- JavaScript files.
- Admin paths.
- Backup files.
- صفحات تم حذفها لاحقا.

> [!summary] الفكرة ببساطة
> حتى لو الصفحة غير موجودة حاليا في navigation، قد تكون ظهرت قديما وتم حفظها في archive. هذا يساعدك في recon واكتشاف attack surface أوسع.

## ما هي Wayback Machine؟

Wayback Machine هي خدمة من Internet Archive تحفظ نسخا تاريخية من مواقع الويب.

الرابط:

```text
https://web.archive.org
```

يمكنك إدخال دومين أو URL ومشاهدة snapshots قديمة للموقع.

## لماذا Wayback Machine مهمة في Bug Bounty و Pentest؟

تفيد في:

- اكتشاف URLs قديمة.
- العثور على parameters مخفية.
- اكتشاف endpoints تم حذفها من الواجهة.
- العثور على JavaScript files قديمة.
- معرفة تغييرات الموقع عبر الوقت.
- اكتشاف صفحات test أو staging ظهرت سابقا.
- العثور على docs أو files قديمة.
- جمع URLs لاستخدامها في fuzzing أو testing.

## أمثلة أشياء ممكن تلاقيها

```text
/old-login
/admin
/api/v1/users
/api/debug
/backup.zip
/config.json
/test.php
/dev/
/staging/
/assets/app-old.js
```

## الاستخدام من الموقع

### الطريقة اليدوية

1. افتح:

```text
https://web.archive.org
```

2. اكتب الدومين:

```text
example.com
```

3. اختار snapshot من تاريخ قديم.

4. راجع الصفحات والروابط.

5. افتح JavaScript files وابحث عن endpoints.

## استخدام CDX API

Wayback Machine توفر API اسمه CDX API لاستخراج URLs المؤرشفة.

مثال:

```text
https://web.archive.org/cdx?url=example.com/*&output=text&fl=original&collapse=urlkey
```

هذا يرجع URLs قديمة للدومين.

## أدوات CLI مهمة

| Tool | الاستخدام |
|---|---|
| `waybackurls` | استخراج URLs من Wayback Machine |
| `gau` | GetAllURLs من أكثر من source |
| `gauplus` | نسخة محسنة من gau |
| `hakrawler` | crawling لاستخراج URLs |
| `katana` | crawler حديث من ProjectDiscovery |
| `uro` | تنظيف وترتيب URLs |
| `unfurl` | استخراج parts من URLs مثل parameters |
| `httpx` | فحص URLs live ومعرفة status/title/tech |

## أمثلة أوامر

### waybackurls

```bash
echo example.com | waybackurls > wayback.txt
```

### gau

```bash
gau example.com > gau.txt
```

### دمج النتائج وإزالة التكرار

```bash
cat wayback.txt gau.txt | sort -u > urls.txt
```

### فحص URLs التي تعمل حاليا

```bash
httpx -l urls.txt -status-code -title -tech-detect -o live-urls.txt
```

### استخراج parameters

```bash
cat urls.txt | grep "=" | sort -u > urls-with-params.txt
```

### تنظيف URLs باستخدام uro

```bash
cat urls.txt | uro > clean-urls.txt
```

## كيف أستخدم النتائج عملياً؟

بعد جمع URLs:

1. احذف التكرار.
2. افصل URLs التي تحتوي parameters.
3. افحص status codes.
4. ابحث عن endpoints مهمة مثل `/api/`, `/admin/`, `/debug/`.
5. افحص JavaScript files.
6. استخدم URLs كمدخلات لأدوات الفحص.
7. قارن القديم بالجديد.

## البحث عن Parameters

Historical URLs ممتازة لاكتشاف parameters.

أمثلة:

```text
?id=
?user=
?redirect=
?url=
?next=
?file=
?path=
?debug=
```

هذه parameters قد تساعد في اختبار:

- IDOR.
- Open Redirect.
- SSRF.
- LFI/RFI.
- XSS.
- SQL Injection.
- Access control issues.

> [!warning] التصريح
> وجود parameter لا يعني أنك تجرب استغلال خارج الـ scope. التزم بقواعد البرنامج وحدود الاختبار.

## البحث في JavaScript القديم

JavaScript files القديمة قد تحتوي على:

- API endpoints.
- Feature flags.
- Internal routes.
- Old domains.
- Comments.
- Keys غير حساسة أو أحيانا secrets بالخطأ.

أمثلة بحث:

```text
api
token
key
secret
debug
admin
staging
internal
```

## فائدة Historical Analysis مع Fuzzing

بدل استخدام wordlist عامة فقط، يمكنك بناء wordlist من URLs القديمة.

مثال:

- استخرج paths من Wayback.
- نظفها.
- استخدمها في fuzzing.
- قارن responses الحالية.

هذا يجعل الفحص أذكى لأنه مبني على تاريخ الموقع الحقيقي.

## مصادر Historical URLs أخرى

| Source | الفائدة |
|---|---|
| Wayback Machine | snapshots و URLs قديمة |
| Common Crawl | أرشيف كبير للويب |
| AlienVault OTX | URLs و domains مرتبطة |
| URLScan.io | صفحات و requests تم تحليلها |
| VirusTotal | URLs و subdomains أحيانا |
| GitHub search | URLs أو domains داخل repos |
| Google Cache/Search | نتائج مفهرسة |

## Google Dorks مفيدة

```text
site:example.com
site:example.com inurl:api
site:example.com inurl:admin
site:example.com inurl:debug
site:example.com filetype:js
site:example.com filetype:json
site:example.com "backup"
```

## ما الذي يعتبر Finding؟

Historical URL وحده ليس ثغرة.

لكن قد يتحول إلى finding لو أدى إلى:

- Endpoint قديم مازال يعمل بدون حماية.
- ملف حساس متاح.
- Parameter يؤدي لثغرة.
- JavaScript قديم يكشف endpoint حساس.
- صفحة admin أو debug تعمل.
- Backup أو config file متاح.

## أخطاء شائعة

- اعتبار كل URL قديم مهم.
- عدم فلترة النتائج.
- عدم فحص هل URL يعمل حاليا.
- تجاهل JavaScript files.
- تجربة payloads بدون فهم endpoint.
- عدم احترام rate limits.
- الاعتماد على Wayback فقط وعدم استخدام مصادر أخرى.

## Workflow عملي مختصر

```text
Domain
  -> collect historical URLs
  -> clean and deduplicate
  -> extract parameters
  -> check live URLs
  -> review interesting paths
  -> analyze old JS files
  -> use findings in fuzzing/testing
```

## الخلاصة

> [!summary] Historical Recon
> Wayback Machine و URL archives تساعدك ترى تاريخ الموقع، وتكتشف endpoints, parameters, files, و JavaScript قديم. أهميتها أنها تعطيك attack surface مبني على بيانات حقيقية من الموقع، وليس wordlists عامة فقط.
