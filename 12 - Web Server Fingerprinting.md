---
title: Web Server Fingerprinting
date: 2026-04-26
tags:
  - fingerprinting
  - web-server
  - nmap
  - recon
  - bug-bounty
  - pentest
---

# Web Server Fingerprinting


## ما هو Web Server Fingerprinting؟

Web Server Fingerprinting يعني جمع إشارات من السيرفر لمعرفة نوعه والتقنيات المستخدمة خلف الموقع.

أمثلة معلومات نبحث عنها:

- نوع السيرفر: Apache, Nginx, IIS, LiteSpeed.
- نظام التشغيل أحيانا: Linux, Windows.
- Framework: Laravel, Django, Express, ASP.NET.
- CMS: WordPress, Drupal, Joomla.
- CDN/WAF: Cloudflare, Akamai, Fastly.
- إصدار الخدمة إذا ظهر.
- Headers وسياسات الأمان.
- TLS/SSL configuration.
- Open ports والخدمات التي تعمل.

> [!summary] الفكرة
> كلما فهمت التقنية التي أمامك، تختار اختبارات أدق بدل الفحص العشوائي.

## كيف يساعدني في Bug Bounty و Pentest؟

- تحديد نوع الاختبارات المناسبة.
- معرفة CMS أو framework لاستهداف misconfigurations الشائعة.
- اكتشاف نسخ قديمة أو خدمات vulnerable.
- فهم هل الموقع خلف WAF/CDN.
- معرفة exposed services غير HTTP.
- تحسين wordlists حسب التقنية.
- كتابة report أوضح بأدلة تقنية.

مثال:

| النتيجة | كيف تفيدك |
|---|---|
| WordPress | تفحص `/wp-admin/`, plugins, users, XML-RPC |
| IIS / ASP.NET | تبحث عن `.aspx`, `web.config`, Windows paths |
| Apache | تفهم `.htaccess`, directory listing, server-status |
| Nginx | تفحص reverse proxy issues و misconfig |
| Laravel | تبحث عن `.env`, debug pages, common routes |

## مصادر Fingerprinting

| المصدر | ماذا يكشف؟ |
|---|---|
| HTTP Headers | server, framework, cookies, security headers |
| HTML Source | generators, comments, scripts |
| JavaScript Files | API endpoints, framework, build info |
| Cookies | framework/session names |
| Error Pages | stack trace, server type, framework |
| TLS Certificate | domains, organization, SANs |
| Open Ports | services مثل SSH, SMTP, FTP |
| Response Behavior | redirects, status codes, custom pages |

## Headers مهمة

| Header | ماذا قد يكشف؟ |
|---|---|
| `Server` | Apache, Nginx, IIS |
| `X-Powered-By` | PHP, Express, ASP.NET |
| `Set-Cookie` | framework أو session technology |
| `Via` | proxy أو CDN |
| `X-Cache` | cache layer |
| `CF-Cache-Status` | Cloudflare |
| `Location` | redirect logic |
| `Content-Type` | HTML, JSON, XML |

مثال:

```text
Server: nginx
X-Powered-By: PHP/8.1
Set-Cookie: laravel_session=...
```

## أدوات Fingerprinting مهمة

| Tool | الاستخدام |
|---|---|
| `nmap` | فحص ports و service versions |
| `httpx` | status, title, tech detect, headers |
| `whatweb` | معرفة technologies من الموقع |
| `wappalyzer` | إضافة متصفح أو CLI لمعرفة التقنيات |
| `curl` | قراءة headers والردود يدويا |
| `nikto` | فحص misconfigurations شائعة |
| `testssl.sh` | تحليل TLS/SSL |
| `sslscan` | فحص SSL/TLS سريع |
| `builtwith` | تحليل تقنيات الموقع من الويب |

## استخدام curl

### عرض headers فقط

```bash
curl -I https://example.com
```

### عرض response مع headers

```bash
curl -i https://example.com
```

### تغيير User-Agent

```bash
curl -i https://example.com -A "Mozilla/5.0"
```

## استخدام httpx

`httpx` مفيد جدا مع قائمة subdomains.

```bash
httpx -l subs.txt -status-code -title -tech-detect -server -cdn -o fingerprint.txt
```

الفائدة:

- يعرف live hosts.
- يعرض status code.
- يعرض page title.
- يكتشف technologies.
- يوضح CDN أحيانا.
- مناسب للفرز السريع.

## استخدام WhatWeb

```bash
whatweb https://example.com
```

مع تفاصيل أكثر:

```bash
whatweb -v https://example.com
```

يفيد في كشف:

- CMS.
- Framework.
- JavaScript libraries.
- Server.
- Analytics tools.

## استخدام Nmap

`nmap` يستخدم لفحص ports والخدمات.

> [!warning] Scope
> استخدم `nmap` فقط إذا كان port scanning مسموحا في برنامج bug bounty أو ضمن تصريح pentest.

### فحص خدمات وإصدارات

```bash
nmap -sV example.com
```

### فحص ports شائعة

```bash
nmap -sV -p 80,443,8080,8443 example.com
```

### فحص scripts آمنة نسبيا للويب

```bash
nmap -sV --script=http-title,http-headers -p 80,443 example.com
```

### فحص TLS certificate

```bash
nmap --script ssl-cert -p 443 example.com
```

## ماذا أستفيد من Nmap؟

- معرفة open ports.
- معرفة service versions.
- اكتشاف services غير متوقعة.
- معرفة هل هناك admin panels على ports أخرى.
- تحديد أولويات الاختبار.

أمثلة ports مهمة:

| Port | الخدمة غالبا |
|---|---|
| 80 | HTTP |
| 443 | HTTPS |
| 8080 | Web app / proxy |
| 8443 | HTTPS alternative |
| 22 | SSH |
| 21 | FTP |
| 25 | SMTP |
| 3306 | MySQL |
| 5432 | PostgreSQL |
| 6379 | Redis |
| 9200 | Elasticsearch |

## TLS/SSL Fingerprinting

TLS configuration قد تكشف معلومات مهمة:

- Certificate SANs فيها subdomains.
- Issuer مثل Let's Encrypt أو DigiCert.
- Expiration date.
- Supported protocols.
- Weak ciphers.
- Misconfiguration.

أدوات:

```bash
sslscan example.com
testssl.sh https://example.com
```

## Error Page Fingerprinting

أحيانا صفحات الخطأ تكشف التقنية.

أمثلة:

- صفحة Apache default.
- صفحة Nginx default.
- ASP.NET error page.
- Laravel debug page.
- Django traceback.
- Express error.

طرق ملاحظة ذلك:

- افتح path غير موجود.
- جرّب request ناقص أو غير صحيح.
- راقب headers وbody.

> [!danger] Debug Pages
> صفحات debug التي تكشف stack trace أو environment variables قد تكون finding قوي.

## Fingerprinting من Cookies

أسماء cookies قد تكشف التقنية:

| Cookie | إشارة محتملة |
|---|---|
| `PHPSESSID` | PHP |
| `JSESSIONID` | Java |
| `ASP.NET_SessionId` | ASP.NET |
| `laravel_session` | Laravel |
| `connect.sid` | Express/Node.js |
| `wordpress_logged_in` | WordPress |

## Fingerprinting من ملفات ومسارات

بعض paths تدل على تقنيات:

| Path | احتمال |
|---|---|
| `/wp-admin/` | WordPress |
| `/administrator/` | Joomla |
| `/user/login` | Drupal |
| `/phpmyadmin/` | phpMyAdmin |
| `/actuator/` | Spring Boot |
| `/server-status` | Apache |
| `/swagger-ui/` | Swagger/OpenAPI |

## كيف أحول Fingerprinting لاختبار عملي؟

بعد معرفة التقنية:

- ابحث عن misconfigurations الخاصة بها.
- استخدم wordlists مناسبة.
- افحص docs أو endpoints الشائعة.
- راجع CVEs فقط إذا الإصدار مؤكد وداخل scope.
- اختبر access control و exposed files.
- افحص security headers.
- راقب debug/error behavior.

مثال workflow:

```text
Subdomains
  -> httpx tech detect
  -> group by technology
  -> check interesting ports
  -> review headers/cookies
  -> run focused checks
  -> document evidence
```

## أخطاء شائعة

- الاعتماد على Header واحد فقط.
- اعتبار version ظاهر صحيح دائما.
- تشغيل nmap بشكل واسع بدون تصريح.
- تجاهل CDN/WAF لأنه قد يخفي السيرفر الحقيقي.
- عدم مراجعة JavaScript وcookies.
- استخدام أدوات كثيرة بدون تحليل النتائج.

## الخلاصة

> [!summary] Fingerprinting
> Web Server Fingerprinting يساعدك تفهم البيئة قبل الاختبار: السيرفر، framework، CMS، ports، TLS، وheaders. قوته في أنه يحول recon من تخمين عشوائي إلى اختبار مركز ومناسب للتقنية الموجودة.
