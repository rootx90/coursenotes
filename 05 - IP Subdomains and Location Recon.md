---
title: IP Subdomains and Location Recon
date: 2026-04-26
tags:
  - ip
  - subdomains
  - geolocation
  - asn
  - recon
  - bug-bounty
  - pentest
---

# IP, Subdomains, and Location Recon

[[Index|Back to Index]]

## الفكرة العامة

في bug bounty و pentest، مرحلة recon هدفها معرفة الأصول التابعة للهدف:

- Domains.
- Subdomains.
- IP addresses.
- Hosting providers.
- Cloud services.
- ASN.
- Open ports.
- Technologies.
- Locations تقريبية للـ IPs.

> [!important] مهم
> المعلومات دي لا تعني وجود ثغرة مباشرة. هي تساعدك تبني خريطة للهدف وتحدد أين تبدأ الاختبار داخل الـ scope المسموح.

## ما هو IP Address؟

الـ IP هو عنوان الجهاز أو السيرفر على الشبكة.

أنواعه الأساسية:

| النوع | مثال | ملاحظات |
|---|---|---|
| IPv4 | `192.0.2.10` | الأكثر استخداما |
| IPv6 | `2001:db8::1` | أحدث وأطول |
| Public IP | يظهر على الإنترنت | يمكن الوصول إليه من الإنترنت حسب الإعدادات |
| Private IP | مثل `10.0.0.1` | يستخدم داخل الشبكات الداخلية |

## فائدة الـ IP في Bug Bounty و Pentest

- معرفة السيرفرات التي يستضيف عليها الهدف خدماته.
- فحص open ports لو كان مسموحا في الـ scope.
- معرفة هل الموقع خلف CDN/WAF أم لا.
- ربط أكثر من subdomain بنفس السيرفر.
- اكتشاف origin IP أحيانا.
- معرفة مزود الاستضافة أو cloud provider.
- البحث عن services غير ويب مثل SSH, FTP, SMTP, VPN.

## ما هو Subdomain؟

الـ subdomain هو جزء فرعي من الدومين الرئيسي.

مثال:

```text
example.com
api.example.com
admin.example.com
dev.example.com
staging.example.com
```

## فائدة Subdomains في Bug Bounty و Pentest

Subdomains مهمة جدا لأن الأخطاء غالبا تظهر فيها أكثر من الدومين الرئيسي.

أمثلة:

- `dev.example.com`: بيئة تطوير قد تكون ضعيفة.
- `staging.example.com`: نسخة اختبار قد تحتوي بيانات أو إعدادات ناقصة.
- `admin.example.com`: لوحة إدارة.
- `api.example.com`: API endpoints.
- `old.example.com`: نظام قديم غير محدث.
- `backup.example.com`: ملفات أو خدمات backup.

## طرق جمع Subdomains

### Passive Recon

جمع معلومات بدون إرسال طلبات مباشرة كثيرة للهدف.

أمثلة مصادر:

- Certificate Transparency.
- Search engines.
- DNS datasets.
- Public archives.
- SecurityTrails.
- VirusTotal.
- Shodan.
- Censys.

### Active Recon

إرسال طلبات مباشرة أو DNS queries لاكتشاف subdomains.

أمثلة:

- DNS brute force.
- Zone transfer test.
- Vhost fuzzing.
- Resolving subdomains.

> [!warning] التصريح
> Active recon قد يكون خارج المسموح في بعض برامج bug bounty. لازم تراجع scope و rules قبل التشغيل.

## أدوات مفيدة للـ Subdomain Recon

| Tool | الاستخدام |
|---|---|
| `subfinder` | Passive subdomain enumeration |
| `amass` | Passive و active recon متقدم |
| `assetfinder` | جمع subdomains بسرعة |
| `findomain` | subdomain discovery |
| `dnsx` | DNS resolving وفحص records |
| `puredns` | DNS brute force و resolving |
| `httpx` | فحص subdomains التي عليها web service |
| `ffuf` | vhost fuzzing أو subdomain fuzzing |

## أمثلة أوامر

### جمع Subdomains

```bash
subfinder -d example.com -all -o subs.txt
```

### Resolve للـ Subdomains

```bash
dnsx -l subs.txt -a -aaaa -resp -o resolved.txt
```

### معرفة الخدمات التي تعمل HTTP/HTTPS

```bash
httpx -l subs.txt -title -tech-detect -status-code -o live-web.txt
```

### Vhost Fuzzing

```bash
ffuf -u https://example.com -H "Host: FUZZ.example.com" -w subdomains.txt
```

## ما هو IP Geolocation؟

IP Geolocation يعني محاولة معرفة الموقع الجغرافي التقريبي للـ IP.

قد يظهر:

- Country.
- City.
- ISP.
- Hosting provider.
- ASN.
- Organization.

> [!note] الدقة
> Location by IP ليست دقيقة دائما. أحيانا تعطي مكان مزود الخدمة أو datacenter وليس مكان الشركة الحقيقي.

## فائدة Location by IP

- معرفة الدولة أو المنطقة التي يوجد بها السيرفر أو الـ datacenter.
- معرفة مزود الاستضافة أو ISP.
- التمييز بين IP تابع للشركة و IP تابع لـ CDN.
- فهم توزيع البنية التحتية عالميا.
- مساعدة في كتابة report أو وصف asset.

## أدوات ومواقع IP Lookup

| Tool / Site | الاستخدام |
|---|---|
| `whois` | معلومات تسجيل IP أو domain |
| `ipinfo.io` | IP lookup و ASN و location |
| `bgp.he.net` | ASN و BGP information |
| `Shodan` | services مكشوفة على IPs |
| `Censys` | certificates و exposed services |
| `SecurityTrails` | DNS history و asset discovery |
| `ViewDNS` | DNS/IP tools |

## أمثلة أوامر IP

### Whois للـ IP

```bash
whois 8.8.8.8
```

### Reverse DNS

```bash
dig -x 8.8.8.8
```

### معرفة IP للدومين

```bash
dig example.com A
dig example.com AAAA
```

### فحص ports بشكل بسيط

```bash
nmap -sV 192.0.2.10
```

> [!warning] فحص المنافذ
> استخدم `nmap` فقط عندما يكون port scanning مسموحا داخل الـ scope.

## ASN Recon

ASN اختصار لـ Autonomous System Number.
هو رقم يستخدم لتعريف شبكة كبيرة تملكها شركة أو مزود خدمة.

مثال:

```text
AS15169 - Google
```

## فائدة ASN في Recon

- معرفة IP ranges تابعة للشركة.
- اكتشاف assets غير معروفة.
- ربط بنية تحتية كبيرة بنفس المؤسسة.
- فهم هل الـ IP تابع للشركة نفسها أم cloud provider.

## أدوات ASN

| Tool / Site | الاستخدام |
|---|---|
| `amass intel` | ASN و organization recon |
| `bgp.he.net` | بحث ASN و IP ranges |
| `whois` | معلومات ASN و netblocks |
| `asnmap` | استخراج IP ranges من ASN |

## ما الذي نبحث عنه فعليا؟

في bug bounty و pentest، ابحث عن:

- Subdomains live.
- Admin panels.
- Dev, staging, test environments.
- Old applications.
- APIs.
- Exposed storage أو buckets.
- Origin IP خلف WAF/CDN.
- IPs عليها ports غير متوقعة.
- Misconfigured DNS records.
- CNAME takeover.
- Services عليها default pages أو old versions.

## Workflow عملي مختصر

```text
Domain
  -> collect subdomains
  -> resolve DNS
  -> find live web services
  -> identify technologies
  -> check interesting paths
  -> review IPs and hosting
  -> stay inside scope
```

## أخطاء شائعة

- اعتبار IP geolocation حقيقة مؤكدة.
- فحص IPs خارج scope.
- الاعتماد على أداة واحدة فقط.
- تجاهل IPv6.
- تجاهل subdomains القديمة.
- تجاهل CNAME records.
- عدم حفظ النتائج وتنظيمها.

## الخلاصة

> [!summary] Recon
> IPs, subdomains, ASN, و location by IP تساعدك تفهم البنية التحتية وتوسع خريطة الأصول. أهم شيء هو الالتزام بالـ scope، ثم تحليل النتائج لاختيار أهداف اختبار منطقية ومسموحة.
