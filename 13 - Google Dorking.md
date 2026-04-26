---
title: Google Dorking
date: 2026-04-26
tags:
  - google-dorking
  - search-operators
  - recon
  - bug-bounty
  - pentest
---

# Google Dorking


## ما هو Google Dorking؟

Google Dorking هو استخدام search operators في Google للبحث بشكل دقيق عن صفحات، ملفات، مسارات، أو معلومات مفهرسة عن هدف معين.

بدل البحث العادي:

```text
example.com admin
```

تستخدم operators مثل:

```text
site:example.com inurl:admin
```

> [!summary] الفكرة
> Google Dorking يساعدك تستخدم فهرسة جوجل كأداة recon لاكتشاف أشياء منشورة أو مؤرشفة بدون إرسال فحص مباشر كثيف على الهدف.

## لماذا Google Dorking مهم في Bug Bounty و Pentest؟

- اكتشاف صفحات غير ظاهرة في navigation.
- العثور على ملفات مثل PDF, DOCX, XLSX.
- اكتشاف admin panels أو login pages.
- البحث عن error messages أو stack traces.
- اكتشاف subdomains أحيانا.
- العثور على backups أو logs لو كانت مفهرسة.
- معرفة معلومات عن الشركة أو التقنية المستخدمة.
- جمع URLs لاستخدامها لاحقا في testing أو fuzzing.

## أهم Search Operators

| Operator | الاستخدام |
|---|---|
| `site:` | حصر البحث داخل domain معين |
| `inurl:` | البحث عن كلمة داخل URL |
| `intitle:` | البحث داخل عنوان الصفحة |
| `intext:` | البحث داخل محتوى الصفحة |
| `filetype:` | البحث عن نوع ملف معين |
| `ext:` | مثل filetype غالبا |
| `" "` | بحث عن جملة exact |
| `-` | استبعاد كلمة من النتائج |
| `OR` | البحث عن هذا أو ذاك |
| `*` | wildcard بسيط |

## أمثلة أساسية

### كل الصفحات المفهرسة للهدف

```text
site:example.com
```

### البحث عن admin

```text
site:example.com inurl:admin
```

### البحث عن login

```text
site:example.com inurl:login
```

### البحث عن API

```text
site:example.com inurl:api
```

### البحث عن ملفات PDF

```text
site:example.com filetype:pdf
```

## Dorks للملفات المهمة

```text
site:example.com filetype:pdf
site:example.com filetype:docx
site:example.com filetype:xlsx
site:example.com filetype:csv
site:example.com filetype:txt
site:example.com filetype:log
site:example.com filetype:sql
site:example.com filetype:bak
```

## Dorks للمسارات الحساسة

```text
site:example.com inurl:admin
site:example.com inurl:dashboard
site:example.com inurl:panel
site:example.com inurl:backup
site:example.com inurl:debug
site:example.com inurl:test
site:example.com inurl:staging
site:example.com inurl:dev
```

## Dorks للـ Directory Listing

```text
site:example.com "Index of /"
site:example.com intitle:"index of"
site:example.com "Parent Directory"
```

قد تكشف folders تعرض ملفاتها مباشرة مثل:

- uploads.
- backups.
- logs.
- old files.

## Dorks للأخطاء والـ Stack Traces

```text
site:example.com "Fatal error"
site:example.com "Warning: mysql"
site:example.com "Stack trace"
site:example.com "Traceback"
site:example.com "Laravel"
site:example.com "Django"
site:example.com "ASP.NET"
```

الفائدة:

- معرفة framework.
- اكتشاف debug pages.
- كشف internal paths أحيانا.
- معرفة errors مفهرسة.

## Dorks للـ Config و Secrets

> [!warning] تعامل بحذر
> لو وجدت secrets أو بيانات حساسة، لا تستخدمها. وثق الدليل بأقل قدر كاف واتبع قواعد البرنامج.

```text
site:example.com ".env"
site:example.com "DB_PASSWORD"
site:example.com "API_KEY"
site:example.com "secret_key"
site:example.com "BEGIN PRIVATE KEY"
site:example.com "password"
```

## Dorks للـ Subdomains

```text
site:*.example.com
site:*.example.com -www
```

ملاحظة: Google ليس أفضل مصدر للـ subdomains، لكنه مفيد كمصدر إضافي بجانب أدوات مثل `subfinder` و `amass`.

## Dorks للـ JavaScript

```text
site:example.com filetype:js
site:example.com inurl:assets filetype:js
site:example.com inurl:static filetype:js
```

بعد جمع JS files:

- ابحث عن API endpoints.
- ابحث عن old domains.
- ابحث عن feature flags.
- ابحث عن routes.

## استخدام الاستبعاد

أحيانا نتائج كثيرة من `www` أو docs أو صفحات غير مهمة.

مثال:

```text
site:example.com -www
```

أو:

```text
site:example.com inurl:admin -docs
```

## كيف أحول Google Dorking إلى Workflow؟

```text
Define target scope
  -> search indexed pages
  -> find files and paths
  -> collect interesting URLs
  -> verify live status
  -> classify by risk
  -> test only allowed assets
  -> document evidence
```

## كيف يساعد مع باقي الدروس؟

| الدرس | العلاقة |
|---|---|
| DNS Recon | يساعد في العثور على subdomains أو pages مفهرسة |
| Historical Analysis | يكمل Wayback بنتائج من Google |
| Sensitive Files | يساعد في البحث عن files مكشوفة |
| Fingerprinting | يكشف framework/errors/files |
| Fuzzing | يعطي paths وkeywords لبناء wordlist |

## أدوات ومصادر مفيدة

| Tool / Source | الاستخدام |
|---|---|
| Google Search | البحث اليدوي |
| Google Advanced Search | واجهة أسهل لبعض operators |
| GHDB | Google Hacking Database لأمثلة dorks |
| browser bookmarks | حفظ dorks متكررة |
| `httpx` | فحص URLs المجموعة |
| `uro` | تنظيف URLs |

## ملاحظات مهمة في Bug Bounty

- التزم بالـ scope.
- لا تعتمد على النتيجة حتى تتأكد أن URL يعمل.
- لا تستخدم secrets إذا ظهرت.
- لا تدخل حسابات أو لوحات غير مصرح بها.
- صور أو وثق الدليل بدون كشف بيانات أكثر من اللازم.
- بعض النتائج قد تكون قديمة أو cached.

## أخطاء شائعة

- استخدام dorks عامة جدا تسبب noise.
- عدم استخدام `site:` لتحديد الهدف.
- تجاهل الملفات مثل PDF و XLSX.
- اعتبار كل نتيجة Finding.
- نسيان فحص هل الرابط live.
- البحث عن secrets ثم استخدامها بدلا من الإبلاغ عنها.

## أمثلة عملية مفيدة

```text
site:example.com
site:example.com inurl:login OR inurl:admin
site:example.com filetype:pdf OR filetype:xlsx
site:example.com "Index of /"
site:example.com "Stack trace"
site:example.com inurl:api
site:example.com filetype:js
site:example.com inurl:backup OR inurl:old
```

## الخلاصة

> [!summary] Google Dorking
> Google Dorking هو recon ذكي باستخدام محرك البحث. فائدته أنه يكشف صفحات، ملفات، أخطاء، endpoints، وبيانات مفهرسة قد لا تظهر في التصفح العادي. قوته الحقيقية تظهر عندما تجمع النتائج، تنظفها، تتحقق منها، ثم تختبر فقط ما هو داخل scope.
