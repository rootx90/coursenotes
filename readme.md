# Notes 

* هنا كل ملاحظات المحاضرات، مع التحديث المستمر للمحتوى. 
* كل مقالة محفوظة في ملف منفصل .

## المقالات

| # | المقالة | الموضوع |
|---|---|---|
| 1 | [HTTP vs HTTPS](./01%20-%20HTTP%20vs%20HTTPS.md) | HTTP, HTTPS, TLS, وخطوات التشفير |
| 2 | [HTTP Requests Responses and Methods](./02%20-%20HTTP%20Requests%20Responses%20and%20Methods.md) | شرح request, response, GET, POST, PUT, DELETE |
| 3 | [HTTP Status Codes and Response Pages](./03%20-%20HTTP%20Status%20Codes%20and%20Response%20Pages.md) | شرح 200, 300, 404 وباقي response codes |
| 4 | [DNS Records](./04%20-%20DNS%20Records.md) | DNS records وفائدتها في bug bounty و pentest |
| 5 | [IP Subdomains and Location Recon](./05%20-%20IP%20Subdomains%20and%20Location%20Recon.md) | IP, subdomains, geolocation, وطرق recon مفيدة |
| 6 | [WAF - Origin IP](./06%20-%20WAF%20-%20Origin%20IP.md) | حماية الـ origin IP خلف WAF |
| 7 | [Online Tools vs Command Line Tools](./07%20-%20Online%20Tools%20vs%20Command%20Line%20Tools.md) | الفرق بين الأدوات الأونلاين وأدوات سطر الأوامر |
| 8 | [Fuzzing](./08%20-%20Fuzzing.md) | ما هو الـ fuzzing وأدواته وفائدته |
| 9 | [Robots Crawlers and Web Logs](./09%20-%20Robots%20Crawlers%20and%20Web%20Logs.md) | robots.txt, Google bots, Webalizer, وملفات الويب المفيدة |
| 10 | [Sensitive Files and Metadata](./10%20-%20Sensitive%20Files%20and%20Metadata.md) | ملفات ومسارات حساسة مثل .htaccess, .git, admin, backups |
| 11 | [Historical Analysis and Wayback Machine](./11%20-%20Historical%20Analysis%20and%20Wayback%20Machine.md) | تحليل تاريخي للموقع باستخدام Wayback Machine و URL archives |
| 12 | [Web Server Fingerprinting](./12%20-%20Web%20Server%20Fingerprinting.md) | معرفة نوع السيرفر والتقنيات باستخدام nmap وأدوات fingerprinting |
| 13 | [Google Dorking](./13%20-%20Google%20Dorking.md) | استخدام Google search operators في recon |
| 14 | [Directory and File Brute Force](./14%20-%20Directory%20and%20File%20Brute%20Force.md) | اكتشاف الملفات والمجلدات المخفية باستخدام brute force و wordlists |
| 15 | [Proxy Between Client and Website](./15%20-%20Proxy%20Between%20Client%20and%20Website.md) | فهم دور الـ proxy بين المتصفح والموقع أثناء تحليل الطلبات |
| 16 | [3 Tabs in Burp](./16%20-%203%20Tabs%20in%20Burp.md) | شرح أهم 3 تبويبات في Burp Suite واستخدامها في تحليل الويب |
| 17 | [SQL](./17%20-%20SQL.md) | شرح أساسيات SQL وعلاقتها بتطبيقات الويب وقواعد البيانات |
| 18 | [Common Attacks 2 - XSS](./18%20-%20Common%20Attacks%202%20-%20XSS.md) | شرح ثغرة XSS وأنواعها وفكرة حقن JavaScript داخل صفحات الويب |
| 19 | [Common Attacks 3 - Arbitrary File Upload](./19%20-%20Common%20Attacks%203%20-%20Arbitrary%20File%20Upload.md) | شرح مخاطر رفع الملفات غير الآمن وكيفية التعامل معه بشكل دفاعي |
| 20 | [Common Attacks 4 - LFI Path Traversal and Log Poisoning](./20%20-%20Common%20Attacks%204%20-%20LFI%20Path%20Traversal%20and%20Log%20Poisoning.md) | شرح LFI و Path Traversal وفكرة Log Poisoning في اختبار تطبيقات الويب |

---

## خريطة التعلم السريعة

1. [HTTP vs HTTPS](./01%20-%20HTTP%20vs%20HTTPS.md)
2. [HTTP Requests Responses and Methods](./02%20-%20HTTP%20Requests%20Responses%20and%20Methods.md)
3. [HTTP Status Codes and Response Pages](./03%20-%20HTTP%20Status%20Codes%20and%20Response%20Pages.md)
4. [DNS Records](./04%20-%20DNS%20Records.md)
5. [IP Subdomains and Location Recon](./05%20-%20IP%20Subdomains%20and%20Location%20Recon.md)
6. [WAF - Origin IP](./06%20-%20WAF%20-%20Origin%20IP.md)
7. [Online Tools vs Command Line Tools](./07%20-%20Online%20Tools%20vs%20Command%20Line%20Tools.md)
8. [Fuzzing](./08%20-%20Fuzzing.md)
9. [Robots Crawlers and Web Logs](./09%20-%20Robots%20Crawlers%20and%20Web%20Logs.md)
10. [Sensitive Files and Metadata](./10%20-%20Sensitive%20Files%20and%20Metadata.md)
11. [Historical Analysis and Wayback Machine](./11%20-%20Historical%20Analysis%20and%20Wayback%20Machine.md)
12. [Web Server Fingerprinting](./12%20-%20Web%20Server%20Fingerprinting.md)
13. [Google Dorking](./13%20-%20Google%20Dorking.md)
14. [Directory and File Brute Force](./14%20-%20Directory%20and%20File%20Brute%20Force.md)
15. [Proxy Between Client and Website](./15%20-%20Proxy%20Between%20Client%20and%20Website.md)
16. [3 Tabs in Burp](./16%20-%203%20Tabs%20in%20Burp.md)
17. [SQL](./17%20-%20SQL.md)
18. [Common Attacks 2 - XSS](./18%20-%20Common%20Attacks%202%20-%20XSS.md)
19. [Common Attacks 3 - Arbitrary File Upload](./19%20-%20Common%20Attacks%203%20-%20Arbitrary%20File%20Upload.md)
20. [Common Attacks 4 - LFI Path Traversal and Log Poisoning](./20%20-%20Common%20Attacks%204%20-%20LFI%20Path%20Traversal%20and%20Log%20Poisoning.md)
---


