---
title: Directory and File Brute Force
date: 2026-04-28
tags:
  - directory-bruteforce
  - file-bruteforce
  - content-discovery
  - recon
  - bug-bounty
  - pentest
---

# Directory and File Brute Force

[[Index|Back to Index]]

## ما هو Directory & File Brute Force؟

Directory & File Brute Force هو أسلوب في مرحلة recon يعتمد على تجربة أسماء directories و files كثيرة على موقع معين لاكتشاف محتوى مخفي أو غير ظاهر في واجهة الموقع.

الفكرة أن التطبيق قد يحتوي على صفحات أو ملفات موجودة فعلا، لكنها غير مرتبطة من الصفحة الرئيسية أو غير ظاهرة للمستخدم العادي.

مثال:

```text
https://example.com/admin
https://example.com/backup.zip
https://example.com/api/users
https://example.com/uploads/
```

> [!summary] الفكرة ببساطة
> الأداة تأخذ wordlist وتجرب كل كلمة كمسار أو ملف، ثم تعرض النتائج التي تعطي response مختلف أو مهم.

## الفرق بين Directory Brute Force و File Brute Force

| النوع | الهدف | مثال |
|---|---|---|
| Directory Brute Force | اكتشاف folders أو paths | `/admin`, `/uploads`, `/api` |
| File Brute Force | اكتشاف ملفات محددة | `/backup.zip`, `/config.php`, `/robots.txt` |

## لماذا مهم في Bug Bounty؟

في Bug Bounty، هذه الخطوة مهمة لأنها قد تكشف أشياء لا تظهر بالتصفح العادي، مثل:

- Admin panels.
- Backup files.
- Old versions.
- Test أو staging paths.
- API endpoints غير موثقة.
- Upload directories.
- Logs أو debug files.
- Config files مكشوفة.
- Directory listing.
- ملفات تحتوي metadata أو معلومات داخلية.

هذه النتائج قد تتحول إلى findings إذا كانت داخل scope وتسبب impact واضح.

## أمثلة على مسارات مهمة

```text
/admin
/login
/dashboard
/api
/api/v1
/uploads
/backup
/backups
/old
/test
/dev
/staging
/debug
/logs
```

## أمثلة على ملفات مهمة

```text
/robots.txt
/sitemap.xml
/.git/config
/.env
/config.php
/backup.zip
/backup.tar.gz
/db.sql
/error.log
/debug.log
/phpinfo.php
```

> [!warning] تنبيه مهم
> لو وجدت ملفات حساسة مثل `.env`, database dump, private keys, أو credentials، لا تستخدم البيانات. وثق أقل دليل كاف واتبع قواعد برنامج الـ bug bounty.

## أدوات مشهورة

| Tool | الاستخدام |
|---|---|
| `ffuf` | سريع ومرن لاكتشاف directories و files |
| `gobuster` | مناسب للـ dir, dns, vhost |
| `dirsearch` | مشهور لاكتشاف web paths و extensions |
| `feroxbuster` | سريع ويدعم recursive scanning |
| `Burp Suite Intruder` | اختبار يدوي وتحكم عالي |
| `SecLists` | مصدر مهم للـ wordlists |

## أوامر عملية

### ffuf - Directory Brute Force

```bash
ffuf -u https://example.com/FUZZ -w /path/to/wordlist.txt
```

### ffuf - File Brute Force مع Extensions

```bash
ffuf -u https://example.com/FUZZ -w /path/to/wordlist.txt -e .php,.txt,.bak,.zip,.json
```

### gobuster - Directories and Files

```bash
gobuster dir -u https://example.com -w /path/to/wordlist.txt -x php,txt,bak,zip,json
```

### dirsearch

```bash
dirsearch -u https://example.com -w /path/to/wordlist.txt -e php,txt,bak,zip,json
```

## كيف تقرأ النتائج؟

لا تعتمد على status code فقط. اقرأ النتائج بناء على أكثر من عامل:

- `200 OK`: المسار موجود وغالبا مهم.
- `301` أو `302`: يوجد redirect وقد يدل على directory حقيقي.
- `403 Forbidden`: المسار موجود لكن الوصول ممنوع، وقد يكون مهم.
- `401 Unauthorized`: يحتاج login أو auth.
- `500 Internal Server Error`: قد يدل على bug أو endpoint حساس.
- Response size مختلف.
- عدد الكلمات أو الأسطر مختلف.
- عنوان الصفحة أو نوع المحتوى مختلف.
- Response time غير طبيعي.

مثال:

```text
/admin        301
/backup.zip   200
/.git/config  403
/debug        500
```

كل نتيجة من هذه النتائج تحتاج فحص يدوي قبل الحكم عليها.

## اختيار Wordlist مناسبة

اختيار wordlist مهم جدا. لا تبدأ دائما بأكبر قائمة.

ابدأ بقائمة صغيرة ثم كبرها حسب الهدف:

- Common directories.
- Common files.
- Technology-specific wordlist.
- API routes.
- Backup names.
- Extensions حسب التقنية.

أمثلة extensions:

```text
.php
.aspx
.jsp
.txt
.json
.bak
.old
.zip
.tar.gz
.sql
```

## متى تكون النتيجة Finding؟

ليست كل نتيجة تعتبر ثغرة. النتيجة تصبح مهمة عندما يوجد impact.

أمثلة impact:

- ملف backup يحتوي source code أو database.
- `.git` مكشوف ويمكن استخراج code منه.
- `.env` يحتوي secrets.
- صفحة admin بدون حماية كافية.
- Directory listing يعرض ملفات حساسة.
- Debug page تعرض stack trace أو internal paths.
- API endpoint يكشف بيانات بدون صلاحيات.

## Workflow عملي في Bug Bounty

```text
حدد scope
  -> اجمع subdomains
  -> شغل live probing
  -> اختر wordlist مناسبة
  -> نفذ directory/file brute force
  -> فلتر النتائج المتكررة
  -> افحص النتائج يدويا
  -> اربط النتيجة ب impact
  -> وثق الدليل بأقل قدر كاف
```

## نصائح مهمة

- التزم بالـ scope فقط.
- راعي rate limits حتى لا تسبب ضغط على السيرفر.
- استخدم wordlist صغيرة في البداية.
- لا تعتمد على `200` فقط، فـ `403` و `401` قد تكون مهمة.
- جرب extensions مناسبة للتقنية المستخدمة.
- افحص النتائج يدويا قبل كتابة report.
- استخدم findings من Google Dorking و Wayback لبناء wordlist مخصصة.
- لا تحاول bypass أو استغلال شيء خارج تصريح البرنامج.

## أخطاء شائعة

- استخدام wordlist ضخمة جدا بدون فلترة.
- تجاهل redirects.
- تجاهل `403 Forbidden`.
- عدم تجربة extensions.
- اعتبار كل path موجود vulnerability.
- عدم توثيق impact.
- تشغيل brute force بسرعة عالية ضد هدف حساس.
- اختبار assets خارج scope.

## العلاقة مع باقي الدروس

| الدرس | العلاقة |
|---|---|
| Fuzzing | Directory brute force هو نوع عملي من fuzzing |
| Google Dorking | يعطيك كلمات ومسارات لبناء wordlist |
| Wayback Machine | يكشف URLs قديمة يمكن إعادة اختبارها |
| Sensitive Files | يساعدك تعرف الملفات التي تبحث عنها |
| Fingerprinting | يساعدك تختار extensions حسب التقنية |

## الخلاصة

> [!summary] Directory & File Brute Force
> Directory & File Brute Force من أهم خطوات content discovery في bug bounty. فائدته أنه يكشف صفحات، ملفات، APIs، ونسخ احتياطية غير ظاهرة. القيمة الحقيقية ليست في كثرة النتائج، بل في فلترتها وفهم الـ impact والالتزام بالـ scope.
