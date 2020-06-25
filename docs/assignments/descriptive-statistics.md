В этой работе вам предстоит выполнить два домашних задания с [открытого курса по машинному обучению](https://habrahabr.ru/company/ods/blog/344044/) от OpenDataScience. Каждое задание нужно выполнить отдельно на Pandas, R и SQL. [Первая тема курса](https://habrahabr.ru/company/ods/blog/322626/) посвящена первичному анализу данных с Pandas. Мы рассмотрим этот же пример с использованием SQL и R (решение на R можно найти в нашем репозитории). Чтобы не дублировать исходный текст статьи я оставил только ключевые фразы, поэтому за выводами по полученным результатам следует обращаться именно к нему.

## Запускаем PostgreSQL в Docker

В качестве СУБД будем использовать PostgreSQL, которая будет запущена в докер-контейнере. Все данные будут храниться в отдельном дата-контейнере. Подробное описание команд можно найти в статье [«Dockerized Postgresql Development Environment»](https://ryaneschinger.com/blog/dockerized-postgresql-development-environment/).

```
# Создание дата-контейнера, в котором будут хранится все базы данных постгреса
$ docker create -v /var/lib/postgresql/data --name mypostgres-data busybox

# Создание контейнера с PostgreSQL
$ docker run --name local-mypostgres -e POSTGRES_PASSWORD=secret -d --volumes-from mypostgres-data postgres:latest
```

Подключимся к контейнеру и создадим новую БД:

```
$ docker exec -it local-mypostgres bash
root@9ab15e9feb4d:/# psql -U postgres
psql (9.6.5)
Type "help" for help.

postgres=# CREATE DATABASE odscourse;
postgres=# \l
                                 List of databases
   Name    |  Owner   | Encoding |  Collate   |   Ctype    |   Access privileges
-----------+----------+----------+------------+------------+-----------------------
 odscourse | postgres | UTF8     | en_US.utf8 | en_US.utf8 |
 postgres  | postgres | UTF8     | en_US.utf8 | en_US.utf8 |
 template0 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
           |          |          |            |            | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
           |          |          |            |            | postgres=CTc/postgres
postgres=# \q
root@9ab15e9feb4d:/# exit
```

## Создание таблицы с данными по оттоку клиентов 

Для взаимодействия с PostgreSQL из Python мы будем использовать драйвер `psycopg2`, который можно установить следующей командой:

!!! note
    Перед установкой новых пакетов не забудьте активировать виртуальное окружение.

```
(cs102) $ python -m pip install psycopg2
```

Теперь скачаем файл с данными, которые потребуются для выполнения примеров:

```
(cs102) $ wget https://raw.githubusercontent.com/Yorko/mlcourse_open/master/data/telecom_churn.csv
```

Создадим таблицу, содержащую данные по оттоку клиентов:

```python
import psycopg2
import csv

conn = psycopg2.connect("host=localhost port=5433 dbname=odscourse user=postgres password=secret")
cursor = conn.cursor()

query = """
CREATE TABLE IF NOT EXISTS telecom_churn (
    id SERIAL PRIMARY KEY,
    state VARCHAR,
    account_length INTEGER,
    area_code INTEGER,
    international_plan VARCHAR,
    voice_mail_plan VARCHAR,
    number_vmail_messages INTEGER,
    total_day_minutes REAL,
    total_day_calls INTEGER,
    total_day_charge REAL,
    total_eve_minutes REAL,
    total_eve_calls INTEGER,
    total_eve_charge REAL,
    total_night_minutes REAL,
    total_night_calls INTEGER,
    total_night_charge REAL,
    total_intl_minutes REAL,
    total_intl_calls INTEGER,
    total_intl_charge REAL,
    customer_service_calls INTEGER,
    churn BOOLEAN
)
"""
cursor.execute(query)
conn.commit()

with open('telecom_churn.csv', 'r') as f:
    reader = csv.reader(f)
    # Skip the header row
    next(reader)
    for Id, row in enumerate(reader):
        cursor.execute(
            "INSERT INTO telecom_churn VALUES (%s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s)",
            [Id] + row
        )
conn.commit()
```

Посмотрим на первые 5 строк:

```python
import psycopg2

conn = psycopg2.connect("host=localhost port=5433 dbname=odscourse user=postgres password=secret")
cursor = conn.cursor()

cursor.execute("SELECT * FROM telecom_churn LIMIT 5")
records = cursor.fetchall()
print(records)
```

```
[(0, 'KS', 128, 415, 'No', 'Yes', 25, 265.1, 110, 45.07, 197.4, 99, 16.78, 244.7, 91, 11.01, 10.0, 3, 2.7, 1, False),
(1, 'OH', 107, 415, 'No', 'Yes', 26, 161.6, 123, 27.47, 195.5, 103, 16.62, 254.4, 103, 11.45, 13.7, 3, 3.7, 1, False),
(2, 'NJ', 137, 415, 'No', 'No', 0, 243.4, 114, 41.38, 121.2, 110, 10.3, 162.6, 104, 7.32, 12.2, 5, 3.29, 0, False),
(3, 'OH', 84, 408, 'Yes', 'No', 0, 299.4, 71, 50.9, 61.9, 88, 5.26, 196.9, 89, 8.86, 6.6, 7, 1.78, 2, False),
(4, 'OK', 75, 415, 'Yes', 'No', 0, 166.7, 113, 28.34, 148.3, 122, 12.61, 186.9, 121, 8.41, 10.1, 3, 2.73, 3, False)]
```

Посмотрим на распределение данных по целевой переменной `churn`:

!!! note
    Для более красивого вывода табличных данных можно воспользоваться модулем `tabulate`. Чтобы установить модуль воспользуйтесь командой `pip install tabulate`.

```python
from tabulate import tabulate

def fetch_all(cursor):
    colnames = [desc[0] for desc in cursor.description]
    records = cursor.fetchall()
    return [{colname:value for colname, value in zip(colnames, record)} for record in records]


cursor.execute(
    """
    SELECT churn, COUNT(*)
        FROM telecom_churn
        GROUP BY churn
    """
)
print(tabulate(fetch_all(cursor), "keys", "psql"))
```

```
+---------+---------+
| churn   |   count |
|---------+---------|
| False   |    2850 |
| True    |     483 |
+---------+---------+
```

Посмотрим на распределение пользователей по переменной `area_code`. Нормализуем значения, чтобы посмотреть не абсолютные частоты, а относительные:

```python
cursor.execute(
    """
    SELECT area_code, ROUND((COUNT(*) / (SELECT COUNT(*) FROM telecom_churn)::numeric), 6)
        FROM telecom_churn
        GROUP BY area_code;
    """
)
print(tabulate(fetch_all(cursor), "keys", "psql"))
```

```
+-------------+----------+
|   area_code |    round |
|-------------+----------|
|         408 | 0.251425 |
|         510 | 0.252025 |
|         415 | 0.49655  |
+-------------+----------+
```

## Сортировка

Упорядочим значения в порядке убывания по столбцу `total_day_charge`:

```python
cursor.execute("SELECT * FROM telecom_churn ORDER BY total_day_charge DESC LIMIT 5")
records = cursor.fetchall()
print(records)
```

```
[(365, 'CO', 154, 415, 'No', 'No', 0, 350.8, 75, 59.64, 216.5, 94, 18.4, 253.9, 100, 11.43, 10.1, 9, 2.73, 1, True),
 (985, 'NY', 64, 415, 'Yes', 'No', 0, 346.8, 55, 58.96, 249.5, 79, 21.21, 275.4, 102, 12.39, 13.3, 9, 3.59, 1, True),
 (2594, 'OH', 115, 510, 'Yes', 'No', 0, 345.3, 81, 58.7, 203.4, 106, 17.29, 217.5, 107, 9.79, 11.8, 8, 3.19, 1, True),
 (156, 'OH', 83, 415, 'No', 'No', 0, 337.4, 120, 57.36, 227.4, 116, 19.33, 153.9, 114, 6.93, 15.8, 7, 4.27, 0, True),
 (605, 'MO', 112, 415, 'No', 'No', 0, 335.5, 77, 57.04, 212.5, 109, 18.06, 265.0, 132, 11.93, 12.7, 8, 3.43, 2, True)]
```

Упорядочивать можно по нескольким столбцам:

```python
cursor.execute("SELECT * FROM telecom_churn ORDER BY churn ASC, total_day_charge DESC LIMIT 5")
records = cursor.fetchall()
print(records)
```

```
[(688, 'MN', 13, 510, 'No', 'Yes', 21, 315.6, 105, 53.65, 208.9, 71, 17.76, 260.1, 123, 11.7, 12.1, 3, 3.27, 3, False),
 (2259, 'NC', 210, 415, 'No', 'Yes', 31, 313.8, 87, 53.35, 147.7, 103, 12.55, 192.7, 97, 8.67, 10.1, 7, 2.73, 3, False),
 (534, 'LA', 67, 510, 'No', 'No', 0, 310.4, 97, 52.77, 66.5, 123, 5.65, 246.5, 99, 11.09, 9.2, 10, 2.48, 4, False),
 (575, 'SD', 114, 415, 'No', 'Yes', 36, 309.9, 90, 52.68, 200.3, 89, 17.03, 183.5, 105, 8.26, 14.2, 2, 3.83, 1, False),
 (2858, 'AL', 141, 510, 'No', 'Yes', 28, 308.0, 123, 52.36, 247.8, 128, 21.06, 152.9, 103, 6.88, 7.4, 3, 2.0, 1, False)]
```

## Извлечение данных

Ответим на вопрос: какова доля людей нелояльных пользователей в нашем датафрейме?

```python
cursor.execute("SELECT AVG(churn::int) FROM telecom_churn")
print(tabulate(fetch_all(cursor), "keys", "psql"))
```

```
+----------+
|      avg |
|----------|
| 0.144914 |
+----------+
```

Ответим на вопрос: каковы средние значения числовых признаков среди нелояльных пользователей?

```python
from pprint import pprint as pp

cursor.execute("""
    SELECT AVG(account_length), AVG(number_vmail_messages), AVG(total_day_minutes), AVG(total_day_calls),
              AVG(total_day_charge), AVG(total_eve_minutes), AVG(total_eve_calls), AVG(total_eve_charge),
              AVG(total_night_minutes), AVG(total_night_calls), AVG(total_night_charge), AVG(total_intl_minutes),
              AVG(total_intl_calls), AVG(total_intl_charge), AVG(customer_service_calls), AVG(churn::int)
        FROM telecom_churn WHERE churn = TRUE
""")
records = cursor.fetchall()
pp(records)
```

```
[(Decimal('102.6645962732919255'),
  Decimal('5.1159420289855072'),
  206.9140780984,
  Decimal('101.3354037267080745'),
  35.1759213532473,
  212.410144829602,
  Decimal('100.5610766045548654'),
  18.0549689119153,
  205.231677321914,
  Decimal('100.3995859213250518'),
  9.23552795029081,
  10.6999999869684,
  Decimal('4.1635610766045549'),
  2.88954451525927,
  Decimal('2.2298136645962733'),
  Decimal('1.00000000000000000000'))]
```

Ответим на вопрос: сколько в среднем в течение дня разговаривают по телефону нелояльные пользователи?

```python
cursor.execute("""
    SELECT AVG(total_day_minutes) FROM telecom_churn WHERE churn = TRUE
""")
print(tabulate(fetch_all(cursor), "keys", "psql"))
```

```
+---------+
|     avg |
|---------|
| 206.914 |
+---------+
```

Какова максимальная длина международных звонков среди лояльных пользователей (`churn = FALSE`), не пользующихся услугой международного роуминга (`international_plan = No`)?

```python
cursor.execute("""
    SELECT MAX(total_intl_minutes) FROM telecom_churn
    WHERE churn = FALSE AND international_plan = 'No'
""")
print(tabulate(fetch_all(cursor), "keys", "psql"))
```

```
+-------+
|   max |
|-------|
|  18.9 |
+-------+
```

Для замены значений в колонке можно воспользоваться `CASE`, например:

```python
cursor.execute("""
    SELECT (CASE WHEN international_plan = 'No' THEN False ELSE True END) as international_plan
    FROM telecom_churn
    LIMIT 5
""")
print(tabulate(fetch_all(cursor), "keys", "psql"))
```

```
+----------------------+
| international_plan   |
|----------------------|
| False                |
| False                |
| False                |
| True                 |
| True                 |
+----------------------+
```

## Группировка данных

Группирование данных в зависимости от значения признака `churn` и вывод статистик по трём столбцам в каждой группе:

```python
cursor.execute("""
    SELECT COUNT(*),
           AVG(total_day_minutes), STDDEV(total_day_minutes), MIN(total_day_minutes),
           PERCENTILE_DISC(0.5) WITHIN GROUP (ORDER BY total_day_minutes) as "50%", MAX(total_day_minutes)
    FROM telecom_churn
    GROUP BY churn
""")
print(tabulate(fetch_all(cursor), "keys", "psql"))
```

```
+---------+---------+----------+-------+-------+-------+
|   count |     avg |   stddev |   min |   50% |   max |
|---------+---------+----------+-------+-------+-------|
|    2850 | 175.176 |  50.1817 |     0 | 177.2 | 315.6 |
|     483 | 206.914 |  68.9978 |     0 | 217.6 | 350.8 |
+---------+---------+----------+-------+-------+-------+
```

!!! note
    Примеры с таблицами сопряженности, аналогичными тем, которые рассмотрены в статье, будут показаны далее.

Допустим, мы хотим посмотреть, как наблюдения в нашей выборке распределены в контексте двух признаков: `churn` и `international_plan`:

```python
cursor.execute("""
    SELECT churn, international_plan, COUNT(*) FROM telecom_churn
    GROUP BY churn, international_plan
""")
print(tabulate(fetch_all(cursor), "keys", "psql"))
```

```
+---------+----------------------+---------+
| churn   | international_plan   |   count |
|---------+----------------------+---------|
| True    | No                   |     346 |
| False   | Yes                  |     186 |
| False   | No                   |    2664 |
| True    | Yes                  |     137 |
+---------+----------------------+---------+
```

Давайте посмотрим среднее число дневных, вечерних и ночных звонков для разных `area_code`:

```python
cursor.execute("""
    SELECT area_code,
           AVG(total_day_calls) as avg_total_day_calls,
           AVG(total_eve_calls) as avg_total_eve_calls,
           AVG(total_night_calls) as avg_total_night_calls
    FROM telecom_churn
    GROUP BY area_code
    ORDER BY area_code
""")
print(tabulate(fetch_all(cursor), "keys", "psql"))
```

```
+-------------+-----------------------+-----------------------+-------------------------+
|   area_code |   avg_total_day_calls |   avg_total_eve_calls |   avg_total_night_calls |
|-------------+-----------------------+-----------------------+-------------------------|
|         408 |               100.496 |               99.7888 |                 99.0394 |
|         415 |               100.576 |              100.504  |                100.398  |
|         510 |               100.098 |               99.6714 |                100.601  |
+-------------+-----------------------+-----------------------+-------------------------+
```

Хотим посчитать общее количество звонков для всех пользователей. Создадим временную таблицу и добавим в нее столбец `total_calls`:

```python
cursor.execute("""
    CREATE TABLE telecom_churn_temp  AS
        SELECT *, (total_day_calls + total_eve_calls + total_night_calls + total_intl_calls) as total_calls
        FROM telecom_churn
        LIMIT 5;
    SELECT total_calls FROM telecom_churn_temp
""")
print(tabulate(fetch_all(cursor), "keys", "psql"))
```

```
+---------------+
|   total_calls |
|---------------|
|           303 |
|           332 |
|           333 |
|           255 |
|           359 |
+---------------+
```

## Первые попытки прогнозирования оттока

Посмотрим, как отток связан с признаком «Подключение международного роуминга» (`international_plan`). Сделаем это с помощью сводной таблицы crosstab:

```python
cursor.execute("""
    CREATE EXTENSION tablefunc;
    SELECT Churn, SUM(No) as No, SUM(Yes) as Yes, SUM(No+Yes) as "All" FROM (
        SELECT *
            FROM crosstab('SELECT churn, international_plan, COUNT(*)::int FROM telecom_churn GROUP BY churn, international_plan ORDER BY 1,2')
                AS (Churn BOOLEAN, No INTEGER, Yes INTEGER)
        ) results_tbl
        GROUP BY rollup(Churn)
""")
print(tabulate(fetch_all(cursor), "keys", "psql"))
```

```
+---------+------+-------+-------+
| churn   |   no |   yes |   All |
|---------+------+-------+-------|
| False   | 2664 |   186 |  2850 |
| True    |  346 |   137 |   483 |
|         | 3010 |   323 |  3333 |
+---------+------+-------+-------+
```

Далее посмотрим на еще один важный признак – «Число обращений в сервисный центр» (`customer_service_calls`):

```python
cursor.execute("""
    SELECT Churn,
           SUM("0") as "0", SUM("1") as "1", SUM("2") as "2", SUM("3") as "3",
           SUM("4") as "4", SUM("5") as "5", SUM("6") as "6", SUM("7") as "7",
           SUM("8") as "8", (CASE WHEN SUM("9") IS NULL THEN 0 ELSE SUM("9") END) as "9",
           SUM("0"+"1"+"2"+"3"+"4"+"5"+"6"+"7"+"8"+(CASE WHEN "9" IS NULL THEN 0 ELSE "9" END)) as "ALL"
    FROM (
        SELECT * FROM crosstab(
            'SELECT churn, customer_service_calls, COUNT(*)::int
             FROM telecom_churn GROUP BY churn, customer_service_calls ORDER BY 1,2
        ') AS (
            Churn BOOLEAN, "0" INTEGER, "1" INTEGER, "2" INTEGER, "3" INTEGER,
            "4" INTEGER, "5" INTEGER, "6" INTEGER, "7" INTEGER, "8" INTEGER, "9" INTEGER)
    ) results
    GROUP BY rollup(Churn)
""")
print(tabulate(fetch_all(cursor), "keys", "psql"))
```

```
+---------+-----+------+-----+-----+-----+-----+-----+-----+-----+-----+-------+
| churn   |   0 |    1 |   2 |   3 |   4 |   5 |   6 |   7 |   8 |   9 |   ALL |
|---------+-----+------+-----+-----+-----+-----+-----+-----+-----+-----+-------|
| False   | 605 | 1059 | 672 | 385 |  90 |  26 |   8 |   4 |   1 |   0 |  2850 |
| True    |  92 |  122 |  87 |  44 |  76 |  40 |  14 |   5 |   1 |   2 |   483 |
|         | 697 | 1181 | 759 | 429 | 166 |  66 |  22 |   9 |   2 |   2 |  3333 |
+---------+-----+------+-----+-----+-----+-----+-----+-----+-----+-----+-------+
```

Добавим бинарный признак — результат сравнения `customer_service_calls > 3`. И еще раз посмотрим, как он связан с оттоком:

```python
cursor.execute("""
    SELECT Churn, SUM("0") as "0", SUM("1") as "1", SUM("0"+"1") as "ALL"
    FROM (
        SELECT * FROM crosstab('
            SELECT churn, (CASE WHEN customer_service_calls > 3 THEN 1 ELSE 0 END) as many_service_calls, COUNT(*)::int
            FROM telecom_churn GROUP BY churn, many_service_calls ORDER BY 1,2
        ') AS (
            Churn BOOLEAN, "0" INTEGER, "1" INTEGER
        )
    ) results
    GROUP BY rollup(Churn)
""")
print(tabulate(fetch_all(cursor), "keys", "psql"))
```

```
+---------+------+-----+-------+
| churn   |    0 |   1 |   ALL |
|---------+------+-----+-------|
| False   | 2721 | 129 |  2850 |
| True    |  345 | 138 |   483 |
|         | 3066 | 267 |  3333 |
+---------+------+-----+-------+
```

Объединим рассмотренные выше условия и построим сводную таблицу для этого объединения и оттока:

```python
cursor.execute("""
    SELECT Churn, SUM("0") as "0", SUM("1") as "1", SUM("0"+"1") as "ALL"
    FROM (
        SELECT * FROM crosstab('
            SELECT churn, (
                CASE
                    WHEN customer_service_calls > 3 AND international_plan LIKE $$Yes$$
                    THEN 1
                    ELSE 0
                END) as many_calls_and_plan, COUNT(*)::int
            FROM telecom_churn GROUP BY churn, many_calls_and_plan ORDER BY 1,2
        ') AS (
            Churn BOOLEAN, "0" INTEGER, "1" INTEGER
        )
    ) results
    GROUP BY rollup(Churn)
""")
print(tabulate(fetch_all(cursor), "keys", "psql"))
```

```
+---------+------+-----+-------+
| churn   |    0 |   1 |   ALL |
|---------+------+-----+-------|
| False   | 2841 |   9 |  2850 |
| True    |  464 |  19 |   483 |
|         | 3305 |  28 |  3333 |
+---------+------+-----+-------+
```
