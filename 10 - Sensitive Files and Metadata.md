---
title: Sensitive Files and Metadata
date: 2026-04-26
tags:
  - sensitive-files
  - metadata
  - recon
  - bug-bounty
  - pentest
  - exposure
---

# Sensitive Files and Metadata


## الفكرة العامة

أثناء bug bounty أو pentest، ممكن تلاقي ملفات أو folders مكشوفة على الويب بالخطأ.
هذه الملفات قد تكشف معلومات مهمة عن التطبيق أو السيرفر أو الكود أو إعدادات الحماية.

أمثلة:

- `.git/`
- `.htaccess`
- `/admin/`
- backup files
- config files
- log files
- environment files
- metadata files

> [!important] مهم
> وجود ملف أو path لا يعني ثغرة تلقائيا. الخطورة تعتمد على: هل الملف قابل للقراءة؟ هل يكشف secrets؟ هل يعطي وصول غير مصرح؟ هل داخل scope؟

## .htaccess

ملف `.htaccess` يستخدم غالبا مع Apache للتحكم في إعدادات على مستوى folder.

قد يحتوي على:

- Rewrite rules.
- Access control.
- Redirects.
- Blocking rules.
- Authentication settings.
- Security headers.

مثال:

```apache
RewriteEngine On
RewriteRule ^admin$ /admin/ [R=301,L]
Options -Indexes
```

## كيف يفيد .htaccess في الاختبار؟

لو كان `.htaccess` مكشوفا للقراءة، قد يكشف:

- مسارات داخلية.
- قواعد redirect.
- folders محمية.
- طرق منع access.
- إعدادات security headers.
- معلومات عن بنية التطبيق.

> [!warning] ملاحظة
> في الإعداد الطبيعي، السيرفر لا يجب أن يعرض `.htaccess` للعامة.

## .git Directory

`.git/` هو folder الخاص بنظام Git داخل المشروع.

لو كان مكشوفا على الويب، قد يكون خطر جدا لأنه قد يكشف:

- Source code.
- Commit history.
- Config files.
- أسماء branches.
- Emails أو usernames.
- Secrets لو كانت موجودة في history.

أمثلة paths:

```text
/.git/
/.git/config
/.git/HEAD
/.git/logs/HEAD
```

## لماذا .git Exposure خطير؟

لأنه ممكن يسمح بفهم الكود والمنطق الداخلي للتطبيق.
أحيانا يكشف:

- Database credentials.
- API keys.
- Hidden endpoints.
- Internal comments.
- Old vulnerable code.

> [!danger] High Impact
> كشف `.git` غالبا يعتبر finding مهم إذا كان يمكن قراءة محتواه واسترجاع معلومات حساسة أو source code.

## /admin و Admin Panels

`/admin/` أو paths مشابهة قد تشير إلى لوحة إدارة.

أمثلة:

```text
/admin/
/administrator/
/panel/
/dashboard/
/manage/
/controlpanel/
/wp-admin/
```

## كيف تفيد Admin Paths؟

- معرفة وجود لوحة إدارة.
- تحديد technology أو CMS.
- اختبار access control إذا كان مسموحا.
- التأكد من وجود rate limiting و MFA.
- اكتشاف panels قديمة أو غير محمية.

> [!important] حدود الاختبار
> لا تجرب brute force أو login attacks إلا لو مسموح بوضوح في قواعد البرنامج.

## Backup Files

Backup files هي نسخ احتياطية قد يتركها المطورون على السيرفر بالخطأ.

أمثلة:

```text
/backup.zip
/backup.tar.gz
/site.zip
/www.zip
/db.sql
/database.sql
/old.zip
/admin.bak
/config.php.bak
```

## لماذا Backup Files خطيرة؟

قد تحتوي على:

- Source code.
- Database dumps.
- Credentials.
- API keys.
- User data.
- Internal paths.

## Config and Environment Files

ملفات الإعدادات قد تحتوي على secrets.

أمثلة:

```text
/.env
/config.php
/config.json
/settings.py
/web.config
/appsettings.json
/database.yml
```

معلومات قد تظهر:

- Database host.
- Database username/password.
- API tokens.
- Secret keys.
- Debug mode.
- Mail credentials.

## Log Files

Logs قد تكشف معلومات تشغيلية أو أخطاء.

أمثلة:

```text
/error.log
/access.log
/debug.log
/laravel.log
/logs/
```

قد تحتوي على:

- Stack traces.
- Internal paths.
- User IPs.
- Request URLs.
- Tokens في URLs.
- Errors تكشف framework.

## Metadata Files

Metadata files هي ملفات صغيرة تعطي معلومات عن الموقع أو التطبيق.

أمثلة:

| File | الفائدة |
|---|---|
| `/security.txt` | طريقة التواصل مع فريق الأمن |
| `/humans.txt` | معلومات عن الفريق أو التقنية أحيانا |
| `/manifest.json` | معلومات عن web app |
| `/package.json` | dependencies لو مكشوف |
| `/composer.json` | PHP dependencies |
| `/phpinfo.php` | معلومات PHP خطيرة إذا مكشوفة |
| `/server-status` | Apache status إذا مكشوف |
| `/actuator/` | Spring Boot endpoints |

## Directory Listing

Directory listing يحدث عندما يعرض السيرفر محتوى folder بدلا من منع الوصول.

مثال:

```text
Index of /uploads/
Index of /backup/
Index of /files/
```

الخطورة:

- عرض ملفات حساسة.
- تحميل backups.
- كشف أسماء ملفات داخلية.
- كشف uploads خاصة بالمستخدمين.

## طرق البحث أثناء Recon

### فحص يدوي سريع

```bash
curl -i https://example.com/.git/HEAD
curl -i https://example.com/.env
curl -i https://example.com/backup.zip
curl -i https://example.com/admin/
```

### باستخدام ffuf

```bash
ffuf -u https://example.com/FUZZ -w sensitive-files.txt -mc all
```

### باستخدام Google Dorks

```text
site:example.com ext:sql
site:example.com ext:log
site:example.com ext:bak
site:example.com inurl:admin
site:example.com "Index of /"
site:example.com ".env"
site:example.com "phpinfo()"
```

## Status Codes مهمة هنا

| Code | المعنى أثناء الفحص |
|---|---|
| `200` | الملف أو الصفحة قد تكون موجودة وقابلة للقراءة |
| `301/302` | يوجد redirect، افحص الوجهة |
| `401` | موجود لكن يحتاج authentication |
| `403` | موجود غالبا لكن ممنوع |
| `404` | غير موجود أو مخفي |
| `500` | قد يكون خطأ في السيرفر أو misconfiguration |

## كيف تقيّم الخطورة؟

اسأل نفسك:

- هل الملف قابل للقراءة؟
- هل يحتوي على secrets؟
- هل يكشف source code؟
- هل يكشف بيانات مستخدمين؟
- هل يكشف internal paths أو stack traces؟
- هل يمكن استخدامه للوصول لشيء آخر؟
- هل الـ asset داخل scope؟

## أمثلة Findings قوية

- `.git` مكشوف ويمكن استخراج source code.
- `.env` يكشف database credentials.
- `backup.zip` يحتوي source code أو database dump.
- `phpinfo.php` يكشف environment variables أو server configuration.
- Directory listing يعرض private uploads.
- `/server-status` مكشوف ويعرض requests أو paths.

## أخطاء شائعة

- اعتبار أي `/admin/` ثغرة بدون bypass أو exposure.
- تجاهل `403` رغم أنه قد يدل على path موجود.
- تحميل أو استخدام بيانات حساسة خارج حدود التصريح.
- عدم توثيق URL, status code, evidence بشكل واضح.
- الاعتماد على wordlist واحدة فقط.

## الخلاصة

> [!summary] Sensitive Exposure
> الملفات والمسارات مثل `.git`, `.htaccess`, `.env`, backups, logs, و admin panels مهمة جدا في recon. قيمتها الحقيقية تظهر عندما تكون قابلة للقراءة أو تكشف secrets أو source code أو معلومات تساعد على استغلال مسموح داخل الـ scope.
