---
title: DNS Records
date: 2026-04-26
tags:
  - dns
  - records
  - recon
  - bug-bounty
  - pentest
---

# DNS Records

[[Index|Back to Index]]

## ما هو DNS؟

DNS اختصار لـ Domain Name System.
هو النظام الذي يحول اسم الدومين مثل `example.com` إلى IP address يستطيع الجهاز الاتصال به.

بدون DNS المستخدم سيحتاج أن يحفظ IP لكل موقع بدلا من اسم الموقع.

## فائدة DNS

- تحويل الدومين إلى IP.
- تحديد سيرفرات البريد الإلكتروني.
- ربط subdomains بخدمات مختلفة.
- إثبات ملكية الدومين لبعض الخدمات.
- التحكم في توجيه الترافيك بين أكثر من سيرفر أو خدمة.

## أشهر أنواع DNS Records

| Record | المعنى | الاستخدام |
|---|---|---|
| A | Address Record | يربط الدومين أو subdomain بـ IPv4 |
| AAAA | IPv6 Address Record | يربط الدومين أو subdomain بـ IPv6 |
| CNAME | Canonical Name | يجعل subdomain يشير إلى دومين آخر |
| MX | Mail Exchange | يحدد سيرفرات البريد للدومين |
| TXT | Text Record | يستخدم للتحقق، SPF, DKIM, DMARC، ومعلومات نصية |
| NS | Name Server | يحدد DNS servers المسؤولة عن الدومين |
| SOA | Start of Authority | يحتوي معلومات إدارية عن DNS zone |
| PTR | Pointer Record | يحول IP إلى domain name، عكس A record |
| SRV | Service Record | يحدد مكان خدمة معينة مثل SIP أو LDAP |
| CAA | Certification Authority Authorization | يحدد أي CA مسموح لها بإصدار شهادات للدومين |

## A Record

يربط الدومين أو subdomain بعنوان IPv4.

```text
example.com -> 93.184.216.34
```

الفائدة:

- معرفة السيرفر الذي يستضيف الموقع.
- اكتشاف origin IP أحيانا إذا لم يكن مخفيا خلف WAF/CDN.

## AAAA Record

مثل A record لكن خاص بـ IPv6.

```text
example.com -> 2606:2800:220:1:248:1893:25c8:1946
```

الفائدة:

- معرفة إن كان الموقع يدعم IPv6.
- أحيانا يتم حماية IPv4 خلف WAF لكن IPv6 يكون مكشوفا بسبب خطأ في الإعدادات.

## CNAME Record

يجعل subdomain يشير إلى دومين آخر.

```text
blog.example.com -> example.github.io
```

الفائدة:

- معرفة الخدمات الخارجية المستخدمة.
- اكتشاف احتمالية Subdomain Takeover إذا كان الـ CNAME يشير لخدمة غير مفعلة أو محذوفة.

## MX Record

يحدد سيرفرات البريد الإلكتروني للدومين.

```text
example.com -> mail.example.com
```

الفائدة:

- معرفة مزود البريد المستخدم مثل Google Workspace أو Microsoft 365.
- تحليل سطح الهجوم الخاص بالإيميل.
- فحص إعدادات SPF, DKIM, DMARC المرتبطة بالحماية من spoofing.

## TXT Record

يحتوي نصوصا تستخدم غالبا للتحقق أو إعدادات الحماية.

```text
v=spf1 include:_spf.google.com ~all
v=DMARC1; p=reject
```

الفائدة:

- فحص SPF لمعرفة السيرفرات المسموح لها بإرسال بريد باسم الدومين.
- فحص DMARC لمعرفة سياسة التعامل مع الرسائل المزيفة.
- اكتشاف خدمات مرتبطة بالدومين من records التحقق.

## NS Record

يحدد name servers المسؤولة عن إدارة DNS zone.

```text
example.com -> ns1.cloudflare.com
```

الفائدة:

- معرفة مزود DNS.
- أحيانا يكشف استخدام Cloudflare أو Route53 أو غيرها.
- يساعد في فهم بنية الدومين.

## SOA Record

يحتوي معلومات إدارية عن DNS zone مثل:

- Primary name server.
- Email المسؤول.
- Serial number.
- Refresh و retry values.

الفائدة:

- يعطي معلومات عن إدارة DNS.
- مفيد في التحليل العام وليس غالبا نقطة ضعف مباشرة.

## PTR Record

يعمل reverse DNS، يعني يحول IP إلى domain name.

```text
93.184.216.34 -> example.com
```

الفائدة:

- معرفة الدومينات المرتبطة بـ IP معين.
- مفيد في التحقيقات وتحليل البنية التحتية.

## SRV Record

يحدد مكان خدمة معينة على الدومين.

```text
_sip._tcp.example.com
```

الفائدة:

- اكتشاف خدمات غير ظاهرة في الموقع الرئيسي.
- قد يكشف أنظمة داخلية أو خدمات enterprise.

## CAA Record

يحدد الجهات المسموح لها بإصدار SSL/TLS certificates للدومين.

```text
example.com CAA 0 issue "letsencrypt.org"
```

الفائدة:

- يحسن أمان الشهادات.
- يساعد في معرفة الجهات المستخدمة لإصدار certificates.

## كيف يفيد DNS في Bug Bounty و Pentest؟

DNS مهم جدا في مرحلة Reconnaissance لأنه يساعد في فهم سطح الهجوم.

أمثلة على الفوائد:

- اكتشاف subdomains.
- معرفة IPs الخاصة بالخدمات.
- تحديد الخدمات الخارجية المستخدمة.
- البحث عن origin IP خلف WAF/CDN.
- اكتشاف misconfigurations.
- اكتشاف Subdomain Takeover من CNAME records.
- فحص email security من SPF, DKIM, DMARC.
- معرفة إن كان IPv6 مكشوفا بدون حماية.
- ربط assets مختلفة بنفس الشركة.
- تحديد cloud providers مثل AWS, Azure, GCP, Cloudflare.

## أمثلة أوامر مفيدة

```bash
dig example.com A
dig example.com AAAA
dig example.com MX
dig example.com TXT
dig example.com NS
dig CNAME blog.example.com
whois example.com
```

## نقاط مهمة أثناء الاختبار

- وجود record لا يعني وجود ثغرة مباشرة.
- لا تجرب هجمات بدون تصريح واضح.
- ركز على misconfigurations مثل CNAME لخدمة محذوفة أو origin IP مكشوف.
- قارن DNS الحالي مع DNS history إذا كان مسموحا في نطاق البرنامج.
- افحص subdomains لأن الأخطاء غالبا تظهر فيها أكثر من الدومين الرئيسي.

## الخلاصة

> [!summary] DNS في الاختبار الأمني
> DNS هو خريطة مهمة للبنية التحتية. في bug bounty و pentest، تحليل DNS يساعدك تعرف الأصول، الخدمات، مزودي التقنية، ونقاط الضعف المحتملة في الإعدادات.
