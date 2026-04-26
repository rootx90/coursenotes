---
title: Robots Crawlers and Web Logs
date: 2026-04-26
tags:
  - robots-txt
  - crawler
  - googlebot
  - webalizer
  - logs
  - recon
  - bug-bounty
  - pentest
---

# Robots.txt, Crawlers, and Web Logs


## الفكرة العامة

في مواقع الويب يوجد ملفات وخدمات تساعد محركات البحث والـ crawlers تفهم الموقع، وأحيانا تكشف paths أو معلومات مفيدة في recon.

أمثلة مهمة:

- `robots.txt`
- `sitemap.xml`
- Googlebot و search engine crawlers
- Webalizer و AWStats
- Access logs وتحليل الزيارات

> [!important] مهم
> وجود path داخل `robots.txt` لا يعني أنه ثغرة. هو فقط دليل يساعدك تعرف صفحات أو folders قد تكون موجودة. الاختبار لازم يكون داخل الـ scope والتصريح.

## ما هو robots.txt؟

`robots.txt` هو ملف نصي يوضع غالبا في root الموقع.

مثال:

```text
https://example.com/robots.txt
```

وظيفته أنه يعطي تعليمات للـ bots مثل Googlebot عن الصفحات التي يمكن أو لا يمكن الزحف إليها.

## مثال robots.txt

```text
User-agent: *
Disallow: /admin/
Disallow: /backup/
Allow: /public/

Sitemap: https://example.com/sitemap.xml
```

المعنى:

- `User-agent: *` يعني أن القاعدة لكل bots.
- `Disallow: /admin/` يطلب من bots عدم زيارة هذا المسار.
- `Allow: /public/` يسمح بهذا المسار.
- `Sitemap` يحدد مكان خريطة الموقع.

## هل robots.txt يمنع الوصول فعلا؟

لا.

`robots.txt` ليس حماية أمنية. هو مجرد تعليمات للـ bots المحترمة.

لو كتبت في المتصفح:

```text
https://example.com/admin/
```

قد يفتح المسار لو السيرفر لا يمنعه بصلاحيات أو firewall.

> [!warning] خطأ شائع
> لا تستخدم `robots.txt` لإخفاء صفحات حساسة. أي شخص يستطيع فتح الملف وقراءة المسارات الموجودة فيه.

## كيف يفيدني robots.txt في Bug Bounty و Pentest؟

- يكشف paths مخفية أو غير ظاهرة في الموقع.
- قد يحتوي على admin panels أو backup folders.
- قد يشير إلى staging أو old endpoints.
- قد يعطي رابط `sitemap.xml`.
- يساعدك تبدأ content discovery بشكل أذكى.

أمثلة paths مثيرة للاهتمام:

```text
/admin/
/old/
/backup/
/private/
/dev/
/test/
/api/
/uploads/
```

## ما هو Sitemap.xml؟

`sitemap.xml` ملف يحتوي روابط مهمة داخل الموقع حتى تساعد محركات البحث على الفهرسة.

مثال:

```text
https://example.com/sitemap.xml
```

قد يحتوي على:

- صفحات.
- مقالات.
- APIs أحيانا.
- صفحات قديمة.
- صفحات لم تظهر في navigation.

## كيف يفيد sitemap.xml؟

- جمع URLs بسرعة.
- معرفة أقسام الموقع.
- اكتشاف صفحات قديمة.
- تغذية أدوات الفحص بقائمة URLs.

مثال استخدام:

```bash
curl https://example.com/sitemap.xml
```

## ما هو Googlebot؟

Googlebot هو bot تابع لجوجل يزور المواقع ويفهرس صفحاتها حتى تظهر في نتائج البحث.

يوجد bots أخرى مثل:

- Bingbot.
- YandexBot.
- DuckDuckBot.
- Baiduspider.

## كيف تعمل Crawlers؟

الـ crawler يبدأ من URL معين، ثم:

1. يقرأ الصفحة.
2. يستخرج الروابط.
3. يزور روابط جديدة.
4. يحترم `robots.txt` غالبا.
5. يرسل النتائج لمحرك البحث للفهرسة.

## كيف أستفيد من Google Search في Recon؟

باستخدام Google dorks للبحث عن صفحات أو ملفات ظاهرة في نتائج البحث.

أمثلة:

```text
site:example.com
site:example.com filetype:pdf
site:example.com inurl:admin
site:example.com inurl:backup
site:example.com "index of"
```

الفائدة:

- اكتشاف subdomains أو paths.
- العثور على ملفات PDF أو docs.
- اكتشاف صفحات قديمة.
- معرفة معلومات منشورة بالخطأ.

## ما هو Webalizer؟

Webalizer أداة قديمة لتحليل web server logs وتعرض إحصائيات الزيارات في صفحة ويب.

قد تعرض معلومات مثل:

- أكثر الصفحات زيارة.
- User agents.
- Referrers.
- IPs أو hosts.
- عدد الزيارات.
- أخطاء HTTP.

## لماذا Webalizer مهم في Security Recon؟

لو Webalizer مكشوف للعامة بدون حماية، قد يكشف:

- مسارات داخلية أو غير معروفة.
- ملفات تم الوصول لها.
- أسماء folders مهمة.
- User agents وأدوات مستخدمة.
- Referrers فيها معلومات حساسة أحيانا.
- إحصائيات قد تساعد في فهم بنية الموقع.

أمثلة paths قد تظهر:

```text
/webalizer/
/stats/
/usage/
/awstats/
/awstats/awstats.pl
```

## AWStats

AWStats أداة مشابهة لـ Webalizer لتحليل logs.

إذا كانت مكشوفة، قد تكشف معلومات عن:

- URLs.
- Referrers.
- Search keywords.
- User agents.
- Hosts.

## ما هي Access Logs؟

Access logs هي سجلات طلبات الويب على السيرفر.

قد تحتوي على:

```text
IP - time - method - path - status code - user agent
```

مثال:

```text
192.0.2.10 - - [26/Apr/2026:10:00:00] "GET /admin HTTP/1.1" 403 "Mozilla/5.0"
```

## كيف تفيد Logs في Pentest؟

لو عندك تصريح وaccess للبيئة:

- تعرف الطلبات التي وصلت للسيرفر.
- تكتشف paths يتم استخدامها فعلا.
- تراقب أخطاء `404`, `403`, `500`.
- تفهم user agents و bots.
- تراجع محاولات الاستغلال.
- تتأكد هل payload وصل للسيرفر أم لا.

## ملفات شائعة للفحص

| File / Path | الفائدة |
|---|---|
| `/robots.txt` | تعليمات bots وقد يكشف paths |
| `/sitemap.xml` | روابط صفحات الموقع |
| `/security.txt` | طريقة التواصل الأمني أحيانا |
| `/humans.txt` | معلومات عن الفريق أو التقنية أحيانا |
| `/ads.txt` | معلومات إعلانية |
| `/webalizer/` | إحصائيات Webalizer لو مكشوفة |
| `/awstats/` | إحصائيات AWStats لو مكشوفة |
| `/stats/` | إحصائيات عامة قد تكون حساسة |

## أوامر مفيدة

### قراءة robots.txt

```bash
curl -i https://example.com/robots.txt
```

### قراءة sitemap.xml

```bash
curl -i https://example.com/sitemap.xml
```

### فحص ملفات شائعة بـ ffuf

```bash
ffuf -u https://example.com/FUZZ -w wordlist.txt
```

### Google dorks

```text
site:example.com robots.txt
site:example.com sitemap.xml
site:example.com inurl:stats
site:example.com inurl:awstats
site:example.com inurl:webalizer
```

## أخطاء شائعة

- اعتبار `robots.txt` حماية.
- تجاهل `sitemap.xml`.
- عدم فحص paths الموجودة في robots بشكل يدوي.
- الاعتماد على Google فقط وعدم استخدام DNS/subdomain recon.
- فحص صفحات stats خارج الـ scope.
- عدم الانتباه أن Webalizer/AWStats قد تكون معلومات حساسة.

## الخلاصة

> [!summary] Recon Value
> `robots.txt`, `sitemap.xml`, Googlebot, Webalizer, و AWStats ليست ثغرات بحد ذاتها، لكنها مصادر recon مهمة. فائدتها أنها تكشف paths، صفحات، logs، وإشارات تساعدك تفهم بنية الموقع وتختار نقاط اختبار أفضل داخل الـ scope.
