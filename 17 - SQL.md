
# SQL

## مقدمة بسيطة

SQL اختصار لـ **Structured Query Language**.

هي اللغة التي نستخدمها للتعامل مع قواعد البيانات العلائقية، أو ما يسمى **Relational Databases**، مثل:

- MySQL
- PostgreSQL
- SQL Server
- SQLite
- Oracle

الفكرة ببساطة أن التطبيق يحتاج يخزن ويقرأ بيانات. بدل ما تكون البيانات مبعثرة داخل ملفات، يتم تنظيمها داخل قاعدة بيانات.

مثال بسيط:

```text
Application -> SQL Query -> Database -> Result -> Application
````

يعني عندما تسجل دخولك في موقع، التطبيق غالبا يرسل query إلى قاعدة البيانات حتى يبحث عن حسابك، ثم يرجع النتيجة للتطبيق.

هذا المقال هدفه أن يشرح SQL من البداية، ثم يربطها بطريقة استخدامها داخل تطبيقات الويب، وبعدها يوضح كيف تظهر مشاكل مثل SQL Injection وكيف يتم إصلاحها.

> [!warning] تنبيه مهم
> أي اختبار أمني أو payloads في هذا المقال يكون فقط داخل lab أو نظام تملك تصريحا واضحا لاختباره. الهدف هنا هو الفهم، التوثيق، والإصلاح، وليس تجربة هذه الأفكار على مواقع حقيقية بدون إذن.

---

## محتويات المقال

* ما هو SQL
* شكل الجداول داخل قاعدة البيانات
* أوامر SQL الأساسية
* تصفية وترتيب النتائج
* التجميع والإحصائيات
* ربط الجداول باستخدام JOIN
* مفاهيم مهمة مثل NULL, DISTINCT, IN, BETWEEN, Aliases
* Subqueries
* تصميم الجداول و Schema Design
* Data Types و Constraints
* Indexes و Views
* Transactions و ACID
* SQL داخل تطبيقات الويب
* SQL Injection وأنواعها
* ORMs و raw queries
* الاختبار الآمن
* الإصلاح والحماية
* كتابة التقرير
* تمارين وأدوات للتعلم

---

## Relational و Non-Relational Databases

قبل الدخول في SQL، مهم نعرف أن قواعد البيانات ليست كلها نوعا واحدا.

| النوع                    | أمثلة                                 | الفكرة                                      |
| ------------------------ | ------------------------------------- | ------------------------------------------- |
| Relational Databases     | MySQL, PostgreSQL, SQL Server, SQLite | البيانات تكون في tables وبينها علاقات       |
| Non-Relational Databases | MongoDB, Redis                        | البيانات لا تكون بالضرورة في tables تقليدية |

SQL تستخدم غالبا مع relational databases، لذلك المقال سيركز على هذا النوع.

---

## شكل الجدول داخل قاعدة البيانات

قاعدة البيانات تكون مقسمة إلى **tables**.

كل table يحتوي:

* **columns**: الأعمدة
* **rows**: الصفوف
* كل row يمثل record أو عنصر من البيانات

مثال table اسمه `users`:

| id | username | email                                         | role  |
| -- | -------- | --------------------------------------------- | ----- |
| 1  | ahmed    | [ahmed@example.com](mailto:ahmed@example.com) | user  |
| 2  | test     | [test@example.com](mailto:test@example.com)   | admin |
| 3  | omar     | [omar@example.com](mailto:omar@example.com)   | user  |

في هذا المثال:

* `users` هو اسم الجدول.
* `id`, `username`, `email`, `role` هي columns.
* كل سطر هو row يمثل مستخدم.

---

## أوامر SQL الأساسية

أشهر أوامر البداية في SQL:

| الأمر    | الاستخدام         |
| -------- | ----------------- |
| SELECT   | قراءة بيانات      |
| INSERT   | إضافة بيانات      |
| UPDATE   | تعديل بيانات      |
| DELETE   | حذف بيانات        |
| WHERE    | تحديد شرط         |
| ORDER BY | ترتيب النتائج     |
| LIMIT    | تحديد عدد النتائج |
| JOIN     | ربط أكثر من table |
| GROUP BY | تجميع النتائج     |
| COUNT    | حساب عدد rows     |

سنشرحهم بهدوء واحدا واحدا.

---

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

لو تريد كل الأعمدة:

```sql
SELECT *
FROM users;
```

> [!tip] ملاحظة
> `*` معناها كل الأعمدة، لكنها ليست دائما أفضل اختيار في التطبيقات الكبيرة، لأنك قد تجلب بيانات أكثر من المطلوب.

---

## DISTINCT

`DISTINCT` تستخدم لإزالة التكرار من النتائج.

مثال:

```sql
SELECT DISTINCT role
FROM users;
```

لو جدول `users` يحتوي أكثر من مستخدم بنفس role، هذه query سترجع الأدوار المختلفة فقط.

مثال نتيجة:

| role  |
| ----- |
| user  |
| admin |

هذا مفيد عندما تريد تعرف القيم الموجودة بدون تكرار، مثل roles أو categories أو statuses.

---

## WHERE

`WHERE` تستخدم لتحديد شرط معين.

مثال:

```sql
SELECT *
FROM users
WHERE id = 2;
```

النتيجة المتوقعة:

| id | username | email                                       | role  |
| -- | -------- | ------------------------------------------- | ----- |
| 2  | test     | [test@example.com](mailto:test@example.com) | admin |

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

---

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
> فهم `AND` و `OR` مهم جدا عند فهم SQL Injection، لأن payload مثل `OR 1=1` قد يغير منطق الشرط ويجعله صحيحا دائما إذا كان التطبيق يبني query بطريقة غير آمنة.

---

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

---

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
> لا تستخدم `UPDATE` بدون `WHERE` إلا لو كنت تقصد تعديل كل rows داخل الجدول.

مثال خطر:

```sql
UPDATE users
SET role = 'admin';
```

هذا قد يجعل كل المستخدمين admins.

---

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
> لا تستخدم `DELETE` بدون `WHERE` إلا لو كنت تقصد حذف كل rows من الجدول.

---

## الفرق بين DELETE و TRUNCATE و DROP

هذه نقطة تسبب لخبطة كثيرا في البداية.

| الأمر    | ماذا يفعل؟                     |
| -------- | ------------------------------ |
| DELETE   | يحذف rows، ويمكن استخدام WHERE |
| TRUNCATE | يفرغ الجدول كله غالبا بسرعة    |
| DROP     | يحذف الجدول نفسه               |

أمثلة:

```sql
DELETE FROM users WHERE id = 5;
TRUNCATE TABLE users;
DROP TABLE users;
```

الفرق مهم:

```text
DELETE يحذف بيانات
TRUNCATE يفرغ الجدول
DROP يحذف الجدول نفسه
```

---

## CREATE TABLE

`CREATE TABLE` تستخدم لإنشاء جدول جديد.

مثال:

```sql
CREATE TABLE users (
  id INTEGER PRIMARY KEY,
  username TEXT NOT NULL,
  email TEXT UNIQUE NOT NULL,
  role TEXT DEFAULT 'user'
);
```

هنا أنشأنا table اسمه `users`.

الجدول يحتوي:

* `id`: رقم فريد لكل مستخدم.
* `username`: لا يمكن أن يكون فارغا.
* `email`: لا يمكن أن يكون فارغا ولا يتكرر.
* `role`: لو لم نرسل قيمة، تكون `user` بشكل افتراضي.

---

## ALTER TABLE

`ALTER TABLE` تستخدم لتعديل شكل الجدول نفسه.

مثال إضافة column جديد:

```sql
ALTER TABLE users
ADD COLUMN phone TEXT;
```

مثال حذف column:

```sql
ALTER TABLE users
DROP COLUMN phone;
```

مثال تعديل نوع column في PostgreSQL:

```sql
ALTER TABLE users
ALTER COLUMN email TYPE VARCHAR(255);
```

> [!tip] ملاحظة
> Syntax الخاص بـ `ALTER TABLE` يختلف قليلا بين قواعد البيانات. لذلك عند التنفيذ العملي راجع documentation الخاصة بقاعدة البيانات التي تستخدمها.

---

## NULL

`NULL` تعني أن القيمة غير موجودة أو غير معروفة.

هي ليست مثل:

* empty string `''`
* الرقم `0`
* كلمة `false`

البحث عن القيم الفارغة:

```sql
SELECT *
FROM users
WHERE email IS NULL;
```

البحث عن القيم غير الفارغة:

```sql
SELECT *
FROM users
WHERE email IS NOT NULL;
```

الصحيح مع `NULL` هو:

```sql
IS NULL
IS NOT NULL
```

وليس:

```sql
WHERE email = NULL;
```

أحيانا تريد عرض قيمة بديلة لو كانت القيمة `NULL`.

مثال:

```sql
SELECT COALESCE(phone, 'غير متوفر') AS phone_number
FROM users;
```

المعنى:

```text
لو phone فارغ، اعرض غير متوفر بدلا منه.
```

---

## IN و NOT IN و BETWEEN

`IN` تستخدم عندما تريد تقارن بقائمة قيم.

مثال:

```sql
SELECT *
FROM users
WHERE role IN ('admin', 'moderator');
```

المعنى:

```text
هات المستخدمين الذين role عندهم admin أو moderator
```

`NOT IN` تعكس الشرط.

مثال:

```sql
SELECT *
FROM users
WHERE role NOT IN ('admin');
```

المعنى:

```text
هات المستخدمين الذين role عندهم ليس admin
```

`BETWEEN` تستخدم مع نطاق.

مثال:

```sql
SELECT *
FROM orders
WHERE total BETWEEN 100 AND 500;
```

المعنى:

```text
هات الطلبات التي total فيها بين 100 و 500
```

> [!tip] ملاحظة
> `BETWEEN` غالبا يشمل البداية والنهاية، يعني في المثال السابق يشمل 100 و 500.

---

## Aliases باستخدام AS

`AS` تستخدم لإعطاء اسم مؤقت لعمود أو نتيجة محسوبة.

مثال:

```sql
SELECT username AS name, email AS contact
FROM users;
```

المعنى:

```text
اعرض username باسم name
واعرض email باسم contact
```

مثال مع `COUNT`:

```sql
SELECT COUNT(*) AS total_users
FROM users;
```

النتيجة قد تكون أوضح:

| total_users |
| ----------- |
| 3           |

Aliases مفيدة جدا في التقارير، وفي queries التي تحتوي على joins أو حسابات.

---

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

| الكلمة | المعنى       |
| ------ | ------------ |
| ASC    | ترتيب تصاعدي |
| DESC   | ترتيب تنازلي |

---

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

> [!tip] ملاحظة
> في بعض قواعد البيانات مثل SQL Server قد تجد syntax مختلفا مثل `TOP` أو `OFFSET FETCH`.

---

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

| الرمز | المعنى           |
| ----- | ---------------- |
| `%`   | أي عدد من الحروف |
| `_`   | حرف واحد فقط     |

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

---

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

| role  | count |
| ----- | ----- |
| admin | 1     |
| user  | 2     |

هذا مفيد في التقارير والإحصائيات داخل التطبيقات.

---

## HAVING

`HAVING` تشبه `WHERE`، لكنها تستخدم بعد `GROUP BY`.

مثال:

```sql
SELECT role, COUNT(*)
FROM users
GROUP BY role
HAVING COUNT(*) > 1;
```

الفرق ببساطة:

| الكلمة | تستخدم متى؟ |
| ------ | ----------- |
| WHERE  | قبل التجميع |
| HAVING | بعد التجميع |

مثال عملي:

```text
WHERE تختار rows قبل GROUP BY
HAVING تصفي النتائج بعد GROUP BY
```

---

## Subqueries

Subquery يعني query داخل query أخرى.

مثال: هات المستخدمين الذين لديهم orders.

```sql
SELECT *
FROM users
WHERE id IN (
  SELECT user_id
  FROM orders
);
```

الفكرة:

```text
الاستعلام الداخلي يرجع user_id من orders
والاستعلام الخارجي يرجع users الذين id عندهم داخل هذه القائمة
```

مثال آخر: عرض عدد الطلبات لكل مستخدم.

```sql
SELECT username,
  (
    SELECT COUNT(*)
    FROM orders
    WHERE orders.user_id = users.id
  ) AS order_count
FROM users;
```

Subqueries مفيدة، لكن في بعض الحالات قد تكون `JOIN` أوضح أو أسرع. الاختيار يعتمد على شكل البيانات وحجمها وطريقة كتابة query.

---

## JOIN

`JOIN` تستخدم لربط أكثر من table مع بعض.

مثال table اسمه `orders`:

| id | user_id | total |
| -- | ------- | ----- |
| 1  | 2       | 150   |
| 2  | 3       | 80    |

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
| -------- | ----- |
| test     | 150   |
| omar     | 80    |

أشهر أنواع JOIN:

| النوع             | المعنى                                             |
| ----------------- | -------------------------------------------------- |
| INNER JOIN / JOIN | يرجع rows التي لها match في الجدولين               |
| LEFT JOIN         | يرجع كل rows من الجدول اليسار حتى لو لا يوجد match |
| RIGHT JOIN        | يرجع كل rows من الجدول اليمين حتى لو لا يوجد match |

مثال `LEFT JOIN`:

```sql
SELECT users.username, orders.total
FROM users
LEFT JOIN orders ON orders.user_id = users.id;
```

المعنى:

```text
هات كل المستخدمين، حتى الذين ليس لديهم orders
```

---

## أنواع JOIN إضافية

بعد فهم `JOIN` و `LEFT JOIN`، توجد أنواع أخرى قد تقابلها.

| النوع           | المعنى                                                 |
| --------------- | ------------------------------------------------------ |
| FULL OUTER JOIN | يرجع كل rows من الجدولين، سواء لها match أو لا         |
| CROSS JOIN      | يربط كل row من الجدول الأول مع كل row من الجدول الثاني |
| SELF JOIN       | يربط الجدول مع نفسه                                    |

مثال `SELF JOIN` شائع في جدول الموظفين والمديرين:

```sql
SELECT e.name AS employee, m.name AS manager
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.id;
```

هنا جدول `employees` استخدمناه مرتين:

* مرة باسم `e` للموظف
* ومرة باسم `m` للمدير

> [!tip] ملاحظة
> دعم بعض أنواع JOIN قد يختلف قليلا حسب قاعدة البيانات. مثلا بعض الإصدارات أو الأنظمة لا تدعم `FULL OUTER JOIN` بنفس الشكل.

---

## Primary Key و Foreign Key

`Primary Key` هو column يميز كل row عن غيره.

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

* تنظيم العلاقة بين الجداول.
* تقليل البيانات المكررة.
* منع وجود order لمستخدم غير موجود.
* تسهيل فهم schema أثناء مراجعة التطبيق.

---

## Data Types الشائعة

كل column في SQL له نوع بيانات.

أمثلة شائعة:

| النوع          | الاستخدام         | مثال            |
| -------------- | ----------------- | --------------- |
| INTEGER        | أرقام صحيحة       | id, age         |
| TEXT / VARCHAR | نصوص              | username, email |
| BOOLEAN        | صح أو خطأ         | is_active       |
| DATE           | تاريخ             | birth_date      |
| TIMESTAMP      | تاريخ ووقت        | login_time      |
| DECIMAL        | أرقام عشرية دقيقة | price, balance  |
| BLOB           | بيانات ثنائية     | صور، ملفات      |

اختيار نوع البيانات مهم.

مثلا:

```text
price الأفضل يكون DECIMAL وليس TEXT
created_at الأفضل يكون TIMESTAMP وليس نصا عاديا
is_active الأفضل يكون BOOLEAN وليس yes/no كنص
```

هذا يجعل البيانات أوضح وأسهل في البحث والترتيب والتحقق.

---

## Constraints مهمة

Constraints هي قواعد تضعها على columns حتى تحافظ على جودة البيانات.

| Constraint  | المعنى                             |
| ----------- | ---------------------------------- |
| NOT NULL    | القيمة لا يمكن أن تكون فارغة       |
| UNIQUE      | القيمة لا تتكرر                    |
| PRIMARY KEY | قيمة فريدة تحدد كل row             |
| FOREIGN KEY | علاقة مع table آخر                 |
| DEFAULT     | قيمة افتراضية لو لم يتم إرسال قيمة |
| CHECK       | شرط لازم يتحقق                     |

مثال:

```sql
CREATE TABLE users (
  id INTEGER PRIMARY KEY,
  email TEXT UNIQUE NOT NULL,
  role TEXT DEFAULT 'user',
  age INTEGER CHECK (age >= 18)
);
```

---

## Normalization

Normalization تعني تنظيم الجداول بطريقة تقلل التكرار وتحافظ على البيانات مرتبة.

مثال غير جيد:

| order_id | username | email | product | total |
| -------- | -------- | ----- | ------- | ----- |

هنا بيانات المستخدم قد تتكرر مع كل order.

الأفضل:

```text
users table
orders table
products table
```

ثم نربط بينهم باستخدام IDs.

مثال:

```text
orders.user_id -> users.id
orders.product_id -> products.id
```

الفكرة ببساطة:

```text
لا تكرر بيانات المستخدم داخل كل order.
خزن المستخدم مرة واحدة، واربط الطلب به عن طريق user_id.
```

### أشكال Normalization بشكل مختصر

| الشكل | المعنى                                                        |
| ----- | ------------------------------------------------------------- |
| 1NF   | كل cell تحتوي قيمة واحدة فقط، لا توجد قوائم داخل cell         |
| 2NF   | كل non-key column يعتمد على key كامل، خصوصا مع composite keys |
| 3NF   | كل non-key column يعتمد على key فقط، وليس على column آخر      |

مثال خطأ قبل Normalization:

```text
| id | username | orders |
| 1  | ahmed    | order1, order2, order3 |
```

مثال أفضل بعد Normalization:

```text
users: id, username
orders: id, user_id, order_details
```

---

## Schema Design مثال عملي

مثال بسيط لتصميم قاعدة بيانات متجر صغير:

```sql
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  username TEXT NOT NULL UNIQUE,
  email TEXT NOT NULL UNIQUE,
  password_hash TEXT NOT NULL,
  role TEXT DEFAULT 'user',
  is_active BOOLEAN DEFAULT true,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE products (
  id SERIAL PRIMARY KEY,
  name TEXT NOT NULL,
  price DECIMAL(10,2) NOT NULL,
  stock INTEGER DEFAULT 0,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE orders (
  id SERIAL PRIMARY KEY,
  user_id INTEGER REFERENCES users(id),
  total DECIMAL(10,2) NOT NULL,
  status TEXT DEFAULT 'pending',
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE order_items (
  id SERIAL PRIMARY KEY,
  order_id INTEGER REFERENCES orders(id),
  product_id INTEGER REFERENCES products(id),
  quantity INTEGER NOT NULL,
  price DECIMAL(10,2) NOT NULL
);
```

الفكرة من التقسيم:

* بيانات المستخدمين في `users`.
* بيانات المنتجات في `products`.
* الطلب الأساسي في `orders`.
* تفاصيل المنتجات داخل كل طلب في `order_items`.

هذا أفضل من وضع كل شيء في جدول واحد كبير، لأنه يقلل التكرار ويسهل الصيانة.

---

## Indexes

Index يساعد قاعدة البيانات تبحث أسرع، خصوصا في columns التي تستخدم كثيرا داخل `WHERE` أو `JOIN`.

مثال:

```sql
CREATE INDEX idx_users_email
ON users(email);
```

مثال index فريد:

```sql
CREATE UNIQUE INDEX idx_users_username
ON users(username);
```

الفكرة:

```text
بدون index، قاعدة البيانات قد تفحص كل rows.
مع index، البحث يكون أسرع غالبا.
```

لكن لا تضع index على كل شيء.

> [!tip] ملاحظة
> كثرة indexes قد تبطئ `INSERT` و `UPDATE` لأن قاعدة البيانات تحتاج تحديث الفهارس أيضا.

---

## Views

`VIEW` هي query محفوظة داخل قاعدة البيانات، ويمكن التعامل معها كأنها table للقراءة.

مثال:

```sql
CREATE VIEW admin_users AS
SELECT username, email
FROM users
WHERE role = 'admin';
```

استخدامها:

```sql
SELECT *
FROM admin_users;
```

الفائدة:

* تبسيط queries متكررة.
* إخفاء بعض التفاصيل عن المستخدم أو التطبيق.
* جعل التقارير أسهل في القراءة.

> [!tip] ملاحظة
> View ليست دائما نسخة مستقلة من البيانات. غالبا هي query محفوظة يتم تنفيذها عند القراءة، حسب نوع قاعدة البيانات وطريقة إعدادها.

---

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

---

## ACID

Transactions تعتمد على فكرة تسمى ACID.

| الحرف       | المعنى      | الفكرة                           |
| ----------- | ----------- | -------------------------------- |
| Atomicity   | الذرية      | كل العملية تنجح أو كلها تفشل     |
| Consistency | الاتساق     | البيانات تبقى في حالة صحيحة      |
| Isolation   | العزل       | العمليات المتزامنة لا تخرب بعضها |
| Durability  | الاستمرارية | بعد COMMIT البيانات تبقى محفوظة  |

مثال بسيط:

```text
لو تحويل الأموال نجح، لازم الخصم والإضافة يحصلان معا.
لو خطوة فشلت، لا نريد نصف عملية.
```

---

# SQL داخل تطبيقات الويب

## مثال Login بسيط

تطبيق login قد يحتاج يبحث عن user باستخدام email.

مثال query:

```sql
SELECT id, username, role
FROM users
WHERE email = 'test@example.com';
```

لو وجد المستخدم، التطبيق يكمل خطوات التحقق من كلمة المرور بطريقة آمنة داخل backend.

لكن هنا نقطة مهمة جدا:

> [!warning] مهم أمنيا
> لا تبني query عن طريق لصق input المستخدم مباشرة داخل SQL. الأفضل استخدام parameterized queries أو prepared statements لتقليل خطر SQL Injection.

مثال فكرة آمنة:

```text
SELECT id, username, role
FROM users
WHERE email = ?
```

علامة `?` تكون مكان قيمة يمررها التطبيق بطريقة آمنة، بدل لصقها كنص داخل query.

---

## Prepared Statements

Prepared statements تفصل بين SQL code والقيم التي يرسلها المستخدم.

يعني قاعدة البيانات تفهم query structure أولا، ثم تتعامل مع input كقيمة فقط.

مثال:

```text
input = "' OR 1=1 --"
```

في الكود غير الآمن، هذا input قد يتحول إلى SQL logic.

أما في prepared statement، يبقى مجرد نص عادي.

مثال Node.js آمن كفكرة:

```js
const q = "SELECT id, username, password_hash, role FROM users WHERE email = ?";
const rows = await db.query(q, [email]);
```

بعدها التطبيق يقارن password مع hash:

```js
const ok = await bcrypt.compare(password, rows[0].password_hash);
```

مثال PHP آمن:

```php
$stmt = $conn->prepare("SELECT id, email, password_hash FROM users WHERE email = ?");
$stmt->bind_param("s", $email);
$stmt->execute();
```

مثال Python آمن:

```py
cursor.execute(
    "SELECT id, email, password_hash FROM users WHERE email = ?",
    (email,)
)
```

القاعدة البسيطة:

```text
SQL text ثابت
User input يمر كقيمة منفصلة
```

---

## حماية كلمات المرور

لا تخزن password كنص واضح، ولا تبحث عنه مباشرة داخل SQL.

خطأ:

```sql
SELECT *
FROM users
WHERE email = 'test@example.com'
AND password = '123456';
```

الأفضل:

```text
1. ابحث عن user بالـ email باستخدام prepared statement.
2. اجلب password_hash من database.
3. قارن password الذي أدخله المستخدم مع hash باستخدام bcrypt أو Argon2.
4. لو المقارنة صحيحة، أنشئ session.
```

مثال شكل data أفضل:

| id | email                                       | password_hash |
| -- | ------------------------------------------- | ------------- |
| 2  | [test@example.com](mailto:test@example.com) | $2b$12$...    |

> [!summary] قاعدة مهمة
> SQL يحضر المستخدم، لكن التحقق من password يكون بمقارنة آمنة مع hash داخل backend.

---

## ORM

كثير من التطبيقات لا تكتب SQL مباشرة، بل تستخدم ORM.

أمثلة:

* Sequelize
* Prisma
* TypeORM
* Django ORM
* Laravel Eloquent
* SQLAlchemy

مثال بفكرة Prisma:

```js
const user = await prisma.user.findUnique({
  where: { email: email }
});
```

ORM يساعد على تقليل الأخطاء، لكنه لا يعني أن التطبيق آمن تلقائيا.

الخطر يظهر غالبا عند استخدام raw queries.

مثال خطر:

```js
const users = await prisma.$queryRawUnsafe(
  "SELECT * FROM users WHERE email = '" + email + "'"
);
```

مثال خطر في Sequelize:

```js
sequelize.query(`SELECT * FROM users WHERE id = ${userId}`);
```

مثال خطر في TypeORM:

```js
getManager().query(`SELECT * FROM users WHERE name = '${name}'`);
```

مثال أفضل كفكرة:

```js
sequelize.query(
  "SELECT * FROM users WHERE id = ?",
  { replacements: [userId] }
);
```

الأفضل استخدام APIs الآمنة أو parameterized raw query حسب الـ ORM المستخدم.

القاعدة هنا:

```text
ORM يساعدك
لكن raw SQL غير الآمن يظل خطرا حتى لو كان داخل ORM
```

---

## مشاكل أمنية ممكن تظهر مع SQL

عندما التطبيق يستخدم SQL بطريقة غير آمنة، قد تظهر مشاكل في:

* login
* search
* profile pages
* admin panels
* APIs
* filters
* sorting
* reports

المشكلة ليست في SQL نفسها، بل في طريقة استخدام التطبيق لها.

---

# SQL Injection

## ما هو SQL Injection؟

SQL Injection تحصل عندما يدخل input المستخدم داخل SQL query بطريقة غير آمنة، فتفهمه قاعدة البيانات كجزء من query وليس كقيمة عادية.

مثال كود غير آمن:

```js
const email = req.body.email;

const query =
  "SELECT id, email FROM users WHERE email = '" + email + "'";

db.query(query);
```

لو المستخدم أدخل قيمة عادية:

```text
test@example.com
```

تصبح query:

```sql
SELECT id, email FROM users WHERE email = 'test@example.com';
```

لكن لو input يحتوي SQL syntax، قد يتغير معنى query بالكامل.

---

## ما المشكلة في كود المطور؟

علامات الكود الضعيف:

* استخدام string concatenation لبناء SQL.
* استخدام template string مع user input مباشرة.
* وجود `req.body`, `req.query`, `params` داخل query بدون parameterization.
* التحقق من password داخل SQL كنص عادي.
* تخزين passwords كنص واضح.
* إرجاع أول row كدليل login success بدون تحقق قوي.

مثال Node.js غير آمن:

```js
const q = `SELECT * FROM users WHERE email = '${email}' AND password = '${password}'`;
const user = await db.query(q);
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

---

## Username Enumeration

Username Enumeration يعني أن التطبيق يكشف هل username أو email موجود أم لا، حتى لو كلمة المرور خطأ.

مثال رسائل غير آمنة:

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

* اختلاف رسالة الخطأ.
* اختلاف status code.
* اختلاف response length.
* اختلاف وقت الرد.
* اختلاف redirect بعد المحاولة.
* وجود endpoint مثل `check-email` أو `forgot-password` يكشف وجود الحساب.

في forgot password، الأفضل:

```text
If account exists, password reset instructions will be sent.
```

بدل:

```text
This email is not registered.
```

---

## Authentication Bypass

Authentication Bypass يعني دخول حساب أو صفحة محمية بدون تحقق صحيح من الهوية.

في SQL Injection، هذا قد يحدث عندما يتغير شرط login.

الفكرة الطبيعية:

```text
username صحيح AND password صحيح -> login success
```

الفكرة الخاطئة:

```text
شرط username أو password تغير بسبب SQL Injection -> login success
```

مثال payload تعليمي داخل lab:

```text
' OR 1=1 -- 
```

قد يحول الشرط إلى true إذا كان التطبيق vulnerable.

مثال آخر مع username معروف داخل lab:

```text
admin' -- 
```

الفكرة أن الجزء الخاص بالpassword قد يتم تجاهله إذا كان الكود يبني query بطريقة غير آمنة.

الخطر:

* الدخول بدون password.
* الدخول كأول user في الجدول.
* أحيانا الوصول لحساب admin.
* فتح panel أو API محمية.

> [!tip] ملاحظة
> نجاح أي payload يعتمد على شكل query، نوع قاعدة البيانات، وطريقة معالجة التطبيق للمدخلات. لو التطبيق يستخدم prepared statements بشكل صحيح، ستتعامل قاعدة البيانات مع payload كنص عادي.

---

## Error-Based SQL Injection

أحيانا التطبيق يعرض error من قاعدة البيانات للمستخدم.

أمثلة رسائل حساسة:

```text
You have an error in your SQL syntax
PostgreSQL query failed
ODBC SQL Server Driver error
SQLite syntax error
```

هذه الرسائل قد تكشف:

* نوع قاعدة البيانات.
* أن input دخل داخل SQL.
* مكان الخطأ في query.
* أحيانا أسماء tables أو columns.

الأفضل:

```text
Something went wrong
```

وتفاصيل الخطأ تكون في server logs فقط.

---

## UNION-Based SQL Injection

`UNION` في SQL تستخدم لدمج نتيجة `SELECT` مع نتيجة `SELECT` أخرى.

مثال طبيعي:

```sql
SELECT username, email
FROM users
UNION
SELECT name, contact_email
FROM customers;
```

في SQL Injection، لو parameter داخل query vulnerable، قد يحاول المختبر داخل lab إضافة `UNION SELECT` حتى يجعل قاعدة البيانات ترجع بيانات من table آخر.

> [!warning] مهم
> أمثلة UNION هنا تعليمية لفهم الفكرة داخل lab أو scope مصرح فقط.

مثال كود vulnerable في صفحة product:

```js
const id = req.query.id;

const query =
  "SELECT name, price, description FROM products " +
  "WHERE id = " + id;

db.query(query);
```

طلب طبيعي:

```text
/product?id=5
```

تصبح query:

```sql
SELECT name, price, description
FROM products
WHERE id = 5;
```

المشكلة أن `id` دخل داخل SQL مباشرة.

### شرط عدد الأعمدة

حتى يعمل `UNION`، لازم عدد columns في أول `SELECT` يساوي عدد columns في `UNION SELECT`.

في المثال السابق:

```sql
SELECT name, price, description
FROM products
WHERE id = 5;
```

عدد الأعمدة = 3.

إذن `UNION SELECT` لازم يرجع 3 values أيضا.

مثال تعليمي:

```text
5 UNION SELECT 'a','b','c'
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

لو `ORDER BY 3` يعمل، و `ORDER BY 4` يعطي error، غالبا عدد الأعمدة هو 3.

طريقة أخرى:

```text
5 UNION SELECT NULL
5 UNION SELECT NULL,NULL
5 UNION SELECT NULL,NULL,NULL
```

لو query الأصلية ترجع 3 columns، فpayload بثلاث `NULL` يكون أقرب للعمل.

### شرط نوع البيانات

ليس عدد الأعمدة فقط مهما. أحيانا نوع البيانات مهم أيضا.

مثال:

```text
name -> text
price -> number
description -> text
```

لذلك يستخدم المختبر `NULL` كثيرا لأن أغلب قواعد البيانات تقبله مكان أنواع كثيرة.

بعدها يمكن معرفة أي column تظهر في الصفحة:

```text
5 UNION SELECT 'test1',NULL,NULL
5 UNION SELECT NULL,'test2',NULL
5 UNION SELECT NULL,NULL,'test3'
```

---

## Blind SQL Injection

Blind SQL Injection يعني أن التطبيق vulnerable، لكنه لا يعرض database error ولا يعرض نتيجة query مباشرة في الصفحة.

بدل أن ترى البيانات، تلاحظ فرق غير مباشر مثل:

* الصفحة رجعت normal أو empty.
* response length تغير.
* status code تغير.
* redirect حصل أو لم يحصل.
* وقت الرد زاد.

الفكرة:

```text
لا أرى نتيجة SQL مباشرة
لكن أقدر أسأل قاعدة البيانات أسئلة true/false
وأفهم الإجابة من سلوك التطبيق
```

أنواع Blind SQLi المهمة:

| النوع             | كيف تعرف النتيجة؟                         |
| ----------------- | ----------------------------------------- |
| Boolean Based     | من اختلاف محتوى الصفحة أو response        |
| Time Based        | من تأخر response                          |
| Conditional Error | من ظهور error عند شرط معين فقط            |
| Out-of-Band       | من اتصال خارجي مثل DNS/HTTP في بيئات خاصة |

---

## Boolean-Based Blind SQLi

Boolean-based يعني ترسل شرطين: واحد true وواحد false، ثم تقارن response.

مثال vulnerable code:

```js
const id = req.query.id;
const query = "SELECT title, body FROM articles WHERE id = " + id;
db.query(query);
```

طلب طبيعي:

```text
/article?id=10
```

اختبار true داخل lab:

```text
10 AND 1=1
```

اختبار false:

```text
10 AND 1=2
```

لو `AND 1=1` يرجع المقال، و `AND 1=2` لا يرجع المقال، فهذا مؤشر أن input دخل داخل SQL.

ماذا تقارن؟

| المقارنة        | مثال                                          |
| --------------- | --------------------------------------------- |
| Status code     | 200 مع true و 404 مع false                    |
| Response length | true = 4820 bytes و false = 1200 bytes        |
| كلمة في الصفحة  | true يظهر Article title و false لا يظهر       |
| Redirect        | true يبقى في الصفحة و false يرجع homepage     |
| JSON value      | true يعطي found:true و false يعطي found:false |

---

## Time-Based Blind SQLi

Time-based يعني لا يوجد فرق واضح في محتوى response، لكن يمكن ملاحظة فرق في وقت الرد.

الفكرة:

```text
لو الشرط صحيح، اجعل قاعدة البيانات تنتظر قليلا
```

أمثلة تعليمية داخل lab:

| Database   | دالة التأخير                                   |
| ---------- | ---------------------------------------------- |
| MySQL      | SLEEP(5)                                       |
| PostgreSQL | pg_sleep(5)                                    |
| SQL Server | WAITFOR DELAY '00:00:05'                       |
| Oracle     | DBMS_LOCK.SLEEP(5) أو طرق مشابهة حسب الصلاحيات |

مثال MySQL داخل lab:

```text
10 AND SLEEP(5)
```

مثال مع شرط:

```text
10 AND IF(1=1,SLEEP(5),0)
10 AND IF(1=2,SLEEP(5),0)
```

> [!tip] ملاحظة
> Time-based يحتاج حذر، لأن بطء الشبكة الطبيعي قد يعطي نتيجة مضللة. قارن أكثر من request، ولا تستخدم delays كبيرة أو requests كثيرة.

---

## Conditional Error-Based Blind SQLi

Conditional Error يعني تجعل قاعدة البيانات تعمل error فقط لو الشرط true.

الفكرة:

```text
true -> يظهر error أو response مختلف
false -> لا يظهر error
```

مثال تعليمي داخل lab:

```text
10 AND (CASE WHEN (1=1) THEN 1/0 ELSE 1 END)=1
```

لو الشرط true، قد يحدث error بسبب `1/0`.

هذا النوع يعتمد جدا على نوع قاعدة البيانات وطريقة معالجة errors داخل التطبيق.

---

## Out-of-Band SQLi

Out-of-Band SQLi يعني أن النتيجة لا تظهر في response ولا في الوقت، بل من اتصال خارجي مثل DNS أو HTTP.

الفكرة:

```text
payload يجعل قاعدة البيانات أو السيرفر يحاول يتصل بدومين تملكه
لو وصل DNS/HTTP request
فهذا دليل أن payload تنفذ
```

هذا النوع أقل شيوعا، ويحتاج lab أو بيئة مصرح بها وأدوات مراقبة DNS/HTTP. لا تضفه في اختبار عشوائي لأنه قد يسبب traffic خارجي غير متوقع.

---

## Second-Order SQL Injection

Second-Order SQL Injection تحدث عندما يتم تخزين input خبيث في قاعدة البيانات أولا، ثم يتم استخدامه لاحقا داخل query غير آمنة.

مثال:

```text
1. المستخدم يسجل username يحتوي SQL syntax.
2. التطبيق يخزن username بدون مشكلة.
3. لاحقا admin panel يستخدم username داخل query بطريقة غير آمنة.
4. هنا تظهر SQL Injection.
```

هذا النوع أخطر من الاختبار العادي، لأن التأثير لا يظهر في نفس اللحظة التي ترسل فيها input.

---

## Stacked Queries

Stacked Queries تعني محاولة تنفيذ أكثر من SQL statement في نفس الطلب.

مثال الفكرة بشكل عام:

```text
statement 1; statement 2
```

الخطورة أن الثغرة لا تكتفي بتغيير نتيجة `SELECT`، بل قد تسمح بتنفيذ أوامر أخرى حسب قاعدة البيانات، driver، والصلاحيات.

لكن دعم هذا السلوك ليس ثابتا:

* بعض drivers تمنعه افتراضيا.
* بعض قواعد البيانات أو الإعدادات تسمح به.
* صلاحيات database user تحدد حجم الضرر.
* prepared statements تقلل هذا الخطر بشكل كبير.

في التقارير، لا تحتاج غالبا إلى تنفيذ أوامر مدمرة لإثبات هذا النوع. يكفي إثبات آمن ومحدود داخل scope.

---

## SQLi وقراءة ملفات السيرفر

في بعض قواعد البيانات، توجد functions أو features قد تسمح بقراءة ملفات من السيرفر إذا توفرت شروط معينة.

هذا لا يحدث دائما. غالبا يحتاج:

* SQL Injection قابلة للاستغلال.
* صلاحيات عالية لحساب قاعدة البيانات.
* إعدادات تسمح بالقراءة من filesystem.
* معرفة مسارات الملفات.
* قاعدة بيانات أو extension يدعم هذا السلوك.

الخطر هنا كبير، لأن قراءة ملفات قد تكشف:

* ملفات configuration.
* credentials.
* مفاتيح أو أسرار.
* معلومات عن النظام.

من ناحية الدفاع:

* لا تشغل التطبيق بحساب database عالي الصلاحيات.
* عطل features غير المطلوبة.
* افصل صلاحيات القراءة والكتابة.
* لا تحفظ secrets داخل ملفات يمكن لحسابات غير لازمة الوصول إليها.
* استخدم prepared statements وأصلح سبب SQLi الأساسي.

> [!warning] مهم
> لا تحاول قراءة ملفات من أي نظام حقيقي لإثبات الثغرة إلا إذا كان ذلك مسموحا بوضوح داخل نطاق الاختبار. غالبا يمكن إثبات SQLi بطرق أقل خطورة.

---

## SQLi في JSON APIs

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

---

## SQLi في ORDER BY

`ORDER BY` حالة خاصة، لأن اسم العمود لا يمكن غالبا تمريره كـ parameter عادي.

مثال غير آمن:

```js
const query = "SELECT * FROM products ORDER BY " + req.query.sort;
```

الإصلاح يكون بـ whitelist:

```text
sort=price  -> ORDER BY price
sort=name   -> ORDER BY name
غير ذلك     -> ORDER BY created_at
```

لا تجعل المستخدم يرسل اسم column الحقيقي مباشرة إلا لو سيتم التحقق منه من قائمة محددة.

---

## فروق مهمة بين قواعد البيانات

ليست كل قواعد البيانات تفهم نفس syntax.

| Database   | Comment شائع | دمج النصوص  | ملاحظة                                             |
| ---------- | ------------ | ----------- | -------------------------------------------------- |
| MySQL      | # أو --      | CONCAT(a,b) | DATABASE() ترجع اسم قاعدة البيانات الحالية         |
| PostgreSQL | --           | a || b      | current_database() ترجع اسم قاعدة البيانات الحالية |
| SQL Server | --           | a + b       | يستخدم WAITFOR DELAY                               |
| SQLite     | --           | a || b      | شائع في التطبيقات الصغيرة والموبايل                |
| Oracle     | --           | a || b      | كثيرا يحتاج FROM dual في بعض queries               |

> [!tip] ملاحظة
> لا تحتاج تحفظ كل syntax من البداية. المهم تفهم أن اختلاف database يغير شكل errors, comments, functions, و time delays.

---

## أمثلة Payloads ومعناها

هذه أمثلة تعليمية لفهم الفكرة داخل lab أو scope مصرح:

| Payload                               | الفكرة                                                |
| ------------------------------------- | ----------------------------------------------------- |
| `' OR 1=1 #`                          | يغلق النص، يجعل الشرط true، ويعلق باقي query في MySQL |
| `' OR '1'='1' -- `                    | نفس الفكرة مع مقارنة نصية و comment                   |
| `admin' -- `                          | اختيار username معروف وتعليق شرط password             |
| `' OR 1=1 LIMIT 1 #`                  | يجعل الشرط true ويرجع نتيجة واحدة                     |
| `') OR ('1'='1`                       | يستخدم عندما تكون القيمة داخل أقواس                   |
| `5 UNION SELECT NULL,NULL,NULL`       | اختبار UNION عندما query الأصلية ترجع 3 columns       |
| `5 UNION SELECT 'test1',NULL,'test3'` | معرفة أي columns تظهر في response                     |
| `10 AND 1=1`                          | اختبار boolean true على parameter رقمي                |
| `10 AND 1=2`                          | اختبار boolean false ومقارنة response                 |
| `10 AND SLEEP(5)`                     | اختبار time-based في MySQL داخل lab                   |
| `10 AND IF(1=1,SLEEP(5),0)`           | time-based مع شرط true                                |
| `10 AND IF(1=2,SLEEP(5),0)`           | time-based مع شرط false                               |

كيف تختار payload؟

* لو التطبيق يستخدم MySQL قد ترى `#` أو `-- `.
* لو input داخل quote، تحتاج تفهم هل quote مفردة `'` أو مزدوجة `"`.
* لو فيه أقواس حول الشرط قد تحتاج تغلق القوس.
* لو التطبيق يستخدم prepared statements، payloads ستتعامل كنص عادي ولن تغير query.
* في blind SQLi لا تعتمد على مرة واحدة. كرر true/false بهدوء للتأكد من أن الفرق ثابت.

علامات أن payload أثرت:

* login success بدون password صحيح.
* تغير واضح في response length.
* تغير status code أو redirect.
* ظهور database error.
* ظهور بيانات أكثر من المتوقع.
* تأخر response بوضوح مع time-based payload.

---

# الاختبار بشكل آمن

## Detection vs Exploitation vs Remediation

من الأفضل تقسيم التفكير إلى ثلاث مراحل:

| المرحلة      | الهدف                             |
| ------------ | --------------------------------- |
| Detection    | إثبات أن input يؤثر على query     |
| Exploitation | فهم التأثير بشكل محدود داخل scope |
| Remediation  | إصلاح السبب الجذري                |

في bug bounty أو pentest، لا تحتاج غالبا إلى استخراج بيانات كثيرة. المطلوب إثبات واضح ومحترم للمشكلة وتأثيرها.

---

## خطوات اختبار هادئة

```text
1. افتح request في Burp Proxy.
2. أرسله إلى Repeater.
3. عدل parameter واحد فقط.
4. قارن response قبل وبعد.
5. ابحث عن اختلاف واضح في status, length, error, redirect, أو login state.
6. وثق النتيجة بدون ضغط على السيرفر.
```

استخدم Repeater أكثر من Intruder في البداية حتى تفهم السلوك بدون إرسال requests كثيرة.

جدول مقارنة بسيط:

| الطلب           | المتوقع          | ماذا تلاحظ؟                             |
| --------------- | ---------------- | --------------------------------------- |
| Normal value    | baseline         | صفحة طبيعية                             |
| True condition  | قريب من baseline | نفس المحتوى أو نفس login state          |
| False condition | مختلف            | محتوى أقل، 404، redirect، أو JSON مختلف |
| Time delay      | تأخير واضح       | فرق ثابت وليس بسبب الشبكة               |

---

## أماكن SQLi الشائعة

لا تختبر login فقط. SQL قد تظهر في أي مكان يستخدم database.

| المكان       | Parameters شائعة                                   |
| ------------ | -------------------------------------------------- |
| Login        | email, username, password                          |
| Search       | q, query, search, keyword                          |
| Filters      | category, type, status, date                       |
| Sorting      | sort, order, direction, column                     |
| Pagination   | page, limit, offset                                |
| Object pages | id, product_id, post_id, user_id                   |
| API JSON     | أي field داخل body                                 |
| Cookies      | tracking IDs أو preferences                        |
| Headers      | أقل شيوعا، لكن أحيانا تدخل في logging أو analytics |

---

## استخدام sqlmap بحذر

`sqlmap` أداة قوية لاختبار SQL Injection، لكنها قد ترسل requests كثيرة وتستخرج بيانات كثيرة إذا استخدمت بدون ضبط.

استخدمها فقط:

* داخل lab.
* أو داخل برنامج يسمح بها بوضوح.
* وبـ rate مناسب.
* وبعد فهم request يدويا في Burp.

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

---

# كيف يكون الإصلاح؟

أفضل دفاع ضد SQL Injection هو استخدام parameterized queries أو prepared statements.

مثال غير آمن:

```text
"SELECT * FROM users WHERE email = '" + email + "'"
```

مثال آمن:

```text
SELECT * FROM users WHERE email = ?
```

مع تمرير `email` كقيمة منفصلة، وليس جزءا من نص query.

إجراءات حماية مهمة:

* استخدام prepared statements.
* عدم لصق user input داخل SQL.
* استخدام password hashing مثل bcrypt أو Argon2.
* جعل رسالة login عامة: `Invalid email or password`.
* إضافة rate limiting على login و forgot password.
* تسجيل المحاولات المشبوهة في logs.
* عدم عرض database errors للمستخدم.
* استخدام أقل صلاحيات ممكنة لحساب قاعدة البيانات.
* استخدام whitelist مع `ORDER BY`, `sort`, `column`, و `direction`.

---

## Least Privilege لحساب قاعدة البيانات

حتى لو حدث SQL Injection، تقليل صلاحيات database user يقلل الضرر.

أمثلة:

| الحساب         | الصلاحيات المناسبة                                    |
| -------------- | ----------------------------------------------------- |
| app_readonly   | SELECT فقط للصفحات التي تقرأ بيانات                   |
| app_writer     | SELECT, INSERT, UPDATE للجداول المطلوبة فقط           |
| migration_user | صلاحيات schema migrations، ولا يستخدمه التطبيق اليومي |

أخطاء خطيرة:

* التطبيق يتصل بقاعدة البيانات بحساب root/admin.
* نفس الحساب يقرأ كل databases.
* الحساب يستطيع DROP, ALTER، أو قراءة ملفات من السيرفر.
* production و staging يستخدمان نفس credentials.

---

## أخطاء شائعة في إصلاح SQL Injection

أخطاء تحدث كثيرا:

* الاعتماد على blacklist فقط.
* حذف single quote فقط.
* استخدام escaping يدوي بدل prepared statements.
* حماية login فقط وترك search/filter/order vulnerable.
* نسيان JSON body أو cookies أو headers.
* استخدام prepared statements للقيم، لكن ترك `ORDER BY` بدون whitelist.
* إخفاء errors بدون إصلاح طريقة بناء query.
* استخدام ORM ثم استخدام raw queries بشكل غير آمن.

---

## Secure Coding Checklist

Checklist مختصرة للمطور أو المراجع الأمني:

* استخدم prepared statements لكل user input.
* تحقق من type مثل `id`, `limit`, `page`.
* استخدم whitelist مع `sort`, `order`, `column`.
* لا تعرض database errors للمستخدم.
* لا تستخدم database admin account داخل التطبيق.
* استخدم bcrypt أو Argon2 لكلمات المرور.
* اجعل رسائل login عامة.
* أضف rate limiting للـ login و forgot password.
* راقب logs لمحاولات SQL syntax غير طبيعية.
* اختبر endpoints: GET, POST, JSON, cookies, headers.
* لا تعتبر ORM حماية تلقائية إذا كان هناك raw SQL.
* افصل صلاحيات database users حسب الحاجة.

---

# كتابة التقرير

## ماذا توثق في Report؟

التوثيق الجيد يثبت المشكلة بدون كشف بيانات أكثر من اللازم.

اكتب:

* endpoint المتأثر.
* parameter المتأثر.
* request طبيعي و request معدل.
* الفرق في response.
* نوع SQLi إذا كان واضحا: error-based, union-based, boolean-based, time-based.
* impact واضح ومحدود.
* recommendation عملية.

لا تكتب:

* dump كامل لقاعدة البيانات.
* passwords أو tokens حقيقية.
* بيانات مستخدمين لا تحتاجها لإثبات الثغرة.
* خطوات تسبب ضغط كبير على السيرفر.

مثال impact جيد:

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

---

## مثال Report: Authentication Bypass

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

---

## مثال Report: Error-Based SQL Injection

```text
Title: Error-Based SQL Injection in product id parameter

Description:
The product id parameter appears to be inserted into a SQL query without proper parameterization.
Changing the parameter value caused a database syntax error to appear in the response.

Impact:
An attacker may be able to manipulate SQL syntax and gather information about the database engine or query structure.

Recommendation:
Use prepared statements for the id parameter, validate that id is numeric server-side,
and avoid exposing database error details to users.
```

---

## مثال Report: Boolean-Based Blind SQL Injection

```text
Title: Boolean-Based Blind SQL Injection in article id parameter

Description:
The article id parameter appears to affect the SQL WHERE condition.
A true condition returned the normal article response, while a false condition returned a different response.

Impact:
An attacker may infer database information by comparing true and false conditions.

Recommendation:
Use parameterized queries, validate numeric parameters, and add tests to ensure user input is never concatenated into SQL queries.
```

---

## مثال Report: UNION-Based SQL Injection

```text
Title: UNION-Based SQL Injection allows reading data from other tables

Description:
The vulnerable parameter allows combining the original query with attacker-controlled SELECT statements in an authorized test environment.

Impact:
This may expose sensitive data from other tables, such as user emails or account metadata.

Recommendation:
Use prepared statements, validate parameter types, restrict database user permissions,
and review similar endpoints for the same pattern.
```

---

## Severity بشكل مبسط

تقييم الخطورة يعتمد على التأثير والسياق.

| الحالة                                | الخطورة غالبا              |
| ------------------------------------- | -------------------------- |
| Error-based فقط بدون استخراج بيانات   | Medium إلى High حسب السياق |
| Authentication bypass                 | Critical غالبا             |
| Reading user data                     | High                       |
| Reading admin/session/password hashes | Critical                   |
| SQLi تحتاج حساب عادي فقط              | تعتمد على التأثير          |
| SQLi في endpoint عام بدون login       | أخطر غالبا                 |

لا تكتب severity من غير توضيح impact. الأفضل دائما تربط الخطورة بما يستطيع المهاجم فعله فعليا.

---

# التعلم العملي

## كيف تبدأ؟

أفضل طريقة لتعلم SQL أن تكتب queries بيدك.

اقتراح بسيط:

```text
1. استخدم DB Browser for SQLite لو تريد بداية سهلة بدون سيرفر.
2. أو استخدم MySQL Workbench مع MySQL.
3. أو استخدم DBeaver لأنه يدعم أكثر من نوع database.
4. لو تعرف Docker، جرب PostgreSQL محليا:
   docker run -d -p 5432:5432 -e POSTGRES_PASSWORD=test postgres
5. نفذ أمثلة المقال واحدة واحدة وعدل عليها.
```

لا تبدأ بالحفظ. ابدأ بسؤال بسيط:

```text
كيف أقرأ؟
كيف أفلتر؟
كيف أرتب؟
كيف أربط جدولين؟
كيف أجمع النتائج؟
```

---

## تمارين مقترحة

| التمرين                                                | الهدف           |
| ------------------------------------------------------ | --------------- |
| أنشئ جدول products وأضف 5 صفوف                         | CREATE و INSERT |
| ابحث عن المنتجات الأغلى من 100                         | WHERE           |
| استخدم DISTINCT لمعرفة categories الموجودة             | DISTINCT        |
| احسب عدد المنتجات لكل category                         | GROUP BY        |
| استخدم HAVING لإظهار categories التي فيها أكثر من منتج | HAVING          |
| اربط users مع orders                                   | JOIN            |
| استخدم subquery لمعرفة مستخدمين لديهم طلبات            | SUBQUERIES      |
| أنشئ transaction لعملية شراء                           | TRANSACTIONS    |
| أضف index على email                                    | INDEX           |
| أنشئ view للمستخدمين admins فقط                        | VIEWS           |

---

## أدوات مفيدة

| الأداة                           | الاستخدام                       |
| -------------------------------- | ------------------------------- |
| DB Browser for SQLite            | تعلم SQL بدون سيرفر             |
| MySQL Workbench                  | واجهة رسومية لـ MySQL           |
| DBeaver                          | يعمل مع عدة أنواع قواعد بيانات  |
| PortSwigger Web Security Academy | تعلم SQLi بأمان                 |
| DVWA                             | تطبيق vulnerable للتدريب المحلي |
| Juice Shop                       | تدريب على ثغرات تطبيقات الويب   |
| WebGoat                          | تدريب تعليمي من OWASP           |
| HackTheBox / TryHackMe           | أهداف تدريبية منظمة             |

---

# Labs للتدريب

بدل تجربة payloads على مواقع حقيقية، استخدم بيئات تدريب.

أمثلة:

* PortSwigger Web Security Academy SQL Injection labs
* DVWA
* Juice Shop
* WebGoat

هذه البيئات مصممة للتعلم، وتعطيك مساحة آمنة تفهم فيها الفكرة بدون إيذاء أي نظام حقيقي.

---

# ماذا تضيف أو تراجع بالترتيب؟

لو كنت تذاكر المقال أو تطوره كمرجع، هذا ترتيب جيد:

| الأولوية | الموضوع                               | السبب                                |
| -------- | ------------------------------------- | ------------------------------------ |
| عالية    | SELECT, WHERE, INSERT, UPDATE, DELETE | أساس SQL                             |
| عالية    | DISTINCT, NULL, IN, BETWEEN           | تستخدم يوميا                         |
| عالية    | JOIN و GROUP BY و HAVING              | مهمة في التطبيقات والتقارير          |
| عالية    | Prepared Statements                   | أهم دفاع ضد SQL Injection            |
| عالية    | Subqueries                            | شائعة في queries الواقعية            |
| متوسطة   | Aliases, Data Types, ALTER TABLE      | تكمل الفهم العملي                    |
| متوسطة   | Indexes و Views                       | مهمة للأداء والتنظيم                 |
| متوسطة   | Normalization و Schema Design         | مهمة لتصميم قاعدة نظيفة              |
| متوسطة   | ORM و raw queries                     | خطأ شائع عند المطورين                |
| متوسطة   | Second-Order SQLi                     | مفهوم أمني مهم                       |
| عادية    | Stacked Queries وقراءة الملفات        | حالات أخطر وأقل شيوعا، تذكرها دفاعيا |
| عادية    | تمارين وأدوات                         | لتثبيت المعلومات                     |

---

# الخلاصة

SQL هي لغة التعامل مع قواعد البيانات العلائقية.

في البداية تحتاج تفهم:

* tables
* rows و columns
* SELECT
* INSERT
* UPDATE
* DELETE
* WHERE
* JOIN
* GROUP BY
* Transactions

بعدها تفهم كيف يستخدمها التطبيق داخل login, search, filters, APIs، وهنا تظهر أهمية الحماية.

أغلب مشاكل SQL Injection تأتي من فكرة واحدة بسيطة:

```text
user input يدخل داخل SQL query مباشرة
```

والإصلاح الأساسي أيضا فكرته بسيطة:

```text
SQL code يبقى منفصل
user input يمر كقيمة منفصلة
```

استخدم prepared statements، تحقق من الأنواع، استخدم whitelist مع القيم التي لا تصلح كـ parameters مثل أسماء الأعمدة، لا تعرض database errors، ولا تستخدم صلاحيات عالية لحساب قاعدة البيانات.

وفي الاختبار الأمني، كن هادئا ومحدودا:

```text
اثبت المشكلة
وضح التأثير
لا تستخرج بيانات أكثر من اللازم
اكتب توصية عملية
```

بهذا الشكل تكون فاهم SQL ليس فقط كأوامر، لكن كجزء حقيقي من بناء وتأمين تطبيقات الويب.

```

