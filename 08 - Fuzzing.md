---
title: Fuzzing
date: 2026-04-26
tags:
  - fuzzing
  - recon
  - bug-bounty
  - pentest
  - wordlist
---

# Fuzzing


## ما هو Fuzzing؟

Fuzzing هو أسلوب اختبار يعتمد على إرسال عدد كبير من القيم أو الطلبات المختلفة إلى تطبيق أو خدمة بهدف اكتشاف أشياء مخفية أو أخطاء أو سلوك غير طبيعي.

في Web Security، الـ fuzzing يستخدم كثيرا لاكتشاف:

- Hidden directories.
- Hidden files.
- Parameters.
- Subdomains.
- API endpoints.
- Backup files.
- Admin panels.
- Inputs تسبب errors أو responses مختلفة.

> [!summary] الفكرة ببساطة
> بدل ما تجرب يدويا مسارات أو باراميترات كثيرة، تستخدم أداة تقرأ wordlist وتجرب القيم واحدة واحدة بسرعة.

## أمثلة بسيطة

### Directory Fuzzing

تجربة مسارات مختلفة على الموقع:

```text
https://example.com/FUZZ
```

الأداة تستبدل `FUZZ` بكلمات من wordlist مثل:

```text
admin
login
backup
uploads
api
```

### Parameter Fuzzing

تجربة أسماء parameters مخفية:

```text
https://example.com/search?FUZZ=test
```

أمثلة:

```text
id
q
page
redirect
url
debug
```

### Subdomain Fuzzing

تجربة subdomains مختلفة:

```text
FUZZ.example.com
```

أمثلة:

```text
admin.example.com
dev.example.com
test.example.com
api.example.com
staging.example.com
```

## فائدة Fuzzing في Bug Bounty و Pentest

- اكتشاف endpoints غير ظاهرة في الموقع.
- الوصول لصفحات admin أو test أو staging.
- العثور على ملفات backup مثل `.zip`, `.bak`, `.old`.
- اكتشاف API routes غير موثقة.
- اكتشاف parameters قد تؤدي إلى ثغرات مثل XSS, SSRF, IDOR, Open Redirect.
- مقارنة responses لمعرفة السلوك المختلف.
- توسيع سطح الهجوم أثناء recon.

## أنواع Fuzzing في الويب

| النوع | الهدف |
|---|---|
| Directory Fuzzing | اكتشاف folders و paths |
| File Fuzzing | اكتشاف ملفات مخفية أو backup |
| Parameter Fuzzing | اكتشاف parameters غير معروفة |
| Subdomain Fuzzing | اكتشاف subdomains |
| Header Fuzzing | تجربة headers مختلفة |
| API Fuzzing | اكتشاف endpoints أو values في APIs |
| Virtual Host Fuzzing | اكتشاف vhosts على نفس IP |

## أدوات Fuzzing مشهورة

| Tool | الاستخدام |
|---|---|
| `ffuf` | من أشهر أدوات fuzzing للويب، سريعة ومرنة |
| `gobuster` | directory, DNS, vhost fuzzing |
| `dirsearch` | اكتشاف directories و files |
| `wfuzz` | fuzzing عام للويب |
| `feroxbuster` | content discovery سريع |
| `Burp Suite Intruder` | fuzzing من داخل Burp Suite |
| `Burp Suite Repeater` | اختبار يدوي وتحليل responses |
| `SecLists` | ليست أداة، لكنه مصدر wordlists مهم جدا |

## أمثلة أوامر

### ffuf - Directory Fuzzing

```bash
ffuf -u https://example.com/FUZZ -w /path/to/wordlist.txt
```

### ffuf - Parameter Fuzzing

```bash
ffuf -u "https://example.com/search?FUZZ=test" -w /path/to/params.txt
```

### ffuf - Subdomain Fuzzing

```bash
ffuf -u https://FUZZ.example.com -w /path/to/subdomains.txt
```

### gobuster - Directory Fuzzing

```bash
gobuster dir -u https://example.com -w /path/to/wordlist.txt
```

### gobuster - DNS Fuzzing

```bash
gobuster dns -d example.com -w /path/to/subdomains.txt
```

## كيف تقرأ النتائج؟

أثناء fuzzing لا تعتمد على status code فقط.

راقب:

- Status code مثل `200`, `301`, `302`, `403`, `500`.
- حجم response.
- عدد الكلمات أو الأسطر.
- اختلاف العنوان أو المحتوى.
- Redirects.
- Errors.
- Response time.

> [!tip] ملاحظة مهمة
> أحيانا `403 Forbidden` يكون مهم لأنه يعني أن المسار موجود لكن الوصول ممنوع.

## Wordlists

الـ wordlist هي قائمة كلمات تستخدمها الأداة في التجربة.

أمثلة على أنواع wordlists:

- Common directories.
- Common files.
- API routes.
- Parameters.
- Subdomains.
- Extensions.

مصدر مشهور:

```text
SecLists
```

## أخطاء شائعة

- استخدام wordlist ضخمة جدا بدون سبب.
- تجاهل rate limits.
- عدم فلترة النتائج المتكررة.
- الاعتماد على status code فقط.
- استخدام fuzzing بدون تصريح.
- عدم تغيير extensions أثناء File Fuzzing.
- تجاهل subdomains و APIs.

## نصائح عملية

- ابدأ بـ wordlist صغيرة ثم كبرها تدريجيا.
- استخدم filters لإخفاء النتائج غير المهمة.
- قارن بين status code و response size.
- جرب extensions مثل `.php`, `.txt`, `.bak`, `.zip`, `.json`.
- افحص النتائج يدويا بعد انتهاء الأداة.
- احترم scope و rate limits في برامج bug bounty.

## الخلاصة

> [!summary] Fuzzing
> الـ fuzzing من أهم خطوات recon واكتشاف المحتوى المخفي. قوته تعتمد على اختيار wordlist مناسبة، قراءة النتائج بذكاء، والالتزام بالتصريح والـ scope.
