# goit-de-hw-03. Аналіз даних у PySpark

## Завдання

**Початкові дані**

Вхідні CSV-файли:

1. [users.csv](csv/users.csv) — дані користувачів:

- `user_id` (унікальний ідентифікатор користувача)
- `name` (ім'я користувача)
- `age` (вік користувача)
- `email` (електронна адреса)

2. [purchases.csv](csv/purchases.csv) — дані про покупки:

- `purchase_id` (унікальний ідентифікатор покупки)
- `user_id` (ідентифікатор користувача, який зробив покупку)
- `product_id` (унікальний ідентифікатор продукту)
- `date` (дата покупки)
- `quantity` (кількість одиниць купленого товару)

3. [products.csv)](csv/products.csv) — інформація про продукти:

- `product_id` (унікальний ідентифікатор продукту)
- `product_name` (назва продукту)
- `category` (категорія продукту)
- `price` (ціна одиниці товару)

**Завдання**

1. Завантажте та прочитайте кожен CSV-файл як окремий DataFrame.
2. Очистіть дані, видаляючи будь-які рядки з пропущеними значеннями.
3. Визначте загальну суму покупок за кожною категорією продуктів.
4. Визначте суму покупок за кожною категорією продуктів для вікової категорії від 18 до 25 включно.
5. Визначте частку покупок за кожною категорією товарів від сумарних витрат для вікової категорії від 18 до 25 років.
6. Виберіть 3 категорії продуктів з найвищим відсотком витрат споживачами віком від 18 до 25 років.

   💡 Відсоток потрібно округлити до другого знака після коми.

## Рішення

### 1. Завантажте та прочитайте кожен CSV-файл як окремий DataFrame.

```Py
from pyspark.sql import SparkSession
from pyspark.sql.functions import col, round, sum as _sum

# Створюємо сесію Spark
spark = SparkSession.builder.appName("goit-de-hw-03").getOrCreate()


# Завантажуємо датасет в окремі DataFrame
df_users = spark.read.csv("./csv/users.csv", header=True, inferSchema=True)
df_products = spark.read.csv("./csv/products.csv", header=True, inferSchema=True)
df_purchases = spark.read.csv("./csv/purchases.csv", header=True, inferSchema=True)
```

Перевіримо структуру завантажених даних:

```Py
# Структура даних
df_users.printSchema()
df_products.printSchema()
df_purchases.printSchema()
```

Структура розпізнана правильно:

```yml
root
 |-- user_id: integer (nullable = true)
 |-- name: string (nullable = true)
 |-- age: integer (nullable = true)
 |-- email: string (nullable = true)

root
 |-- product_id: integer (nullable = true)
 |-- product_name: string (nullable = true)
 |-- category: string (nullable = true)
 |-- price: double (nullable = true)

root
 |-- purchase_id: integer (nullable = true)
 |-- user_id: integer (nullable = true)
 |-- product_id: integer (nullable = true)
 |-- date: date (nullable = true)
 |-- quantity: integer (nullable = true)
```

### 2. Очистіть дані, видаляючи будь-які рядки з пропущеними значеннями.

```Py
# Видаляємо будь-які рядки з пропущеними значеннями.
df_users = df_users.dropna()
df_products = df_products.dropna()
df_purchases = df_purchases.dropna()
```

### 3. Визначте загальну суму покупок за кожною категорією продуктів.

```Py
# Об'єднаємо дані і додамо загальну суму покупки
df_joined = (
    df_users.join(df_purchases, "user_id")
    .join(df_products, "product_id")
    .withColumn("total_cost", col("price") * col("quantity"))
)
df_joined.show()
```

Перевіримо об'єднаний датафрейм

<pre style="white-space: pre; overflow-x: auto;">
+----------+-------+--------+---+-------------------+-----------+----------+--------+------------+-----------+-----+------------------+
|product_id|user_id|    name|age|              email|purchase_id|      date|quantity|product_name|   category|price|        total_cost|
+----------+-------+--------+---+-------------------+-----------+----------+--------+------------+-----------+-----+------------------+
|         9|     52| User_52| 39| user52@example.com|          1|2022-01-01|       1|   Product_9|     Beauty|  6.0|               6.0|
|        37|     93| User_93| 25| user93@example.com|          2|2022-01-02|       8|  Product_37|   Clothing|  6.0|              48.0|
|        33|     15| User_15| 30| user15@example.com|          3|2022-01-03|       1|  Product_33|       Home|  9.4|               9.4|
|        42|     72| User_72| 39| user72@example.com|          4|2022-01-04|       9|  Product_42|     Beauty|  9.1| 81.89999999999999|
|        24|     21| User_21| 37| user21@example.com|          6|2022-01-06|       7|  Product_24|Electronics|  2.1|14.700000000000001|
|        32|     87| User_87| 38| user87@example.com|          8|2022-01-08|       3|  Product_32|       Home|  8.8|26.400000000000002|
|        32|     75| User_75| 40| user75@example.com|          9|2022-01-09|       2|  Product_32|       Home|  8.8|              17.6|
|        24|     75| User_75| 40| user75@example.com|         10|2022-01-10|       9|  Product_24|Electronics|  2.1|18.900000000000002|
|        41|     88| User_88| 25| user88@example.com|         11|2022-01-11|       8|  Product_41|     Sports|  5.9|              47.2|
|        49|    100|User_100| 26|user100@example.com|         12|2022-01-12|       7|  Product_49|     Sports|  9.7| 67.89999999999999|
|        49|     24| User_24| 46| user24@example.com|         13|2022-01-13|       9|  Product_49|     Sports|  9.7|              87.3|
|        12|      3|  User_3| 36|  user3@example.com|         14|2022-01-14|       4|  Product_12|       Home|  1.3|               5.2|
|        39|     22| User_22| 45| user22@example.com|         15|2022-01-15|       4|  Product_39|       Home|  7.3|              29.2|
|         2|     53| User_53| 38| user53@example.com|         16|2022-01-16|       1|   Product_2|       Home|  8.3|               8.3|
|         3|      2|  User_2| 48|  user2@example.com|         17|2022-01-17|       8|   Product_3|Electronics|  9.2|              73.6|
|        49|     88| User_88| 25| user88@example.com|         18|2022-01-18|       3|  Product_49|     Sports|  9.7|29.099999999999998|
|        49|     38| User_38| 41| user38@example.com|         20|2022-01-20|       2|  Product_49|     Sports|  9.7|              19.4|
|        17|      2|  User_2| 48|  user2@example.com|         21|2022-01-21|       2|  Product_17|Electronics|  8.1|              16.2|
|        49|     64| User_64| 26| user64@example.com|         22|2022-01-22|       7|  Product_49|     Sports|  9.7| 67.89999999999999|
|         2|     60| User_60| 37| user60@example.com|         23|2022-01-23|       6|   Product_2|       Home|  8.3|49.800000000000004|
+----------+-------+--------+---+-------------------+-----------+----------+--------+------------+-----------+-----+------------------+
only showing top 20 rows
</pre>

Визначимо загальну суму покупок по категоріям за допомогою функцій групування та агрегації

```Py
# Визначимо загальну суму покупок за кожною категорією продуктів

df_joined.groupBy("category").agg(
    round(_sum("total_cost"), 2).alias("total_sum")
).show()
```

Результат:

```v
+-----------+---------+
|   category|total_sum|
+-----------+---------+
|       Home|   1438.9|
|     Sports|   1755.5|
|Electronics|   1141.9|
|   Clothing|    696.1|
|     Beauty|    441.7|
+-----------+---------+
```

### 4. Визначте суму покупок за кожною категорією продуктів для вікової категорії від 18 до 25 включно.

```Py
# Створимо цільовий датафрейм
df_18_25 = df_joined.filter(df_joined.age >= 18).filter(df_joined.age <= 25)

```

```Py
# Визначимо суму покупок за кожною категорією продуктів для вікової категорії від 18 до 25 включно.

df_18_25.groupBy("category").agg(
    round(_sum("total_cost"), 2).alias("total_sum_18_25")
).show()
```

Результат:

```v
+-----------+---------------+
|   category|total_sum_18_25|
+-----------+---------------+
|       Home|          361.1|
|     Sports|          310.5|
|Electronics|          249.6|
|   Clothing|          245.0|
|     Beauty|           41.4|
+-----------+---------------+
```

### 5. Визначте частку покупок за кожною категорією товарів від сумарних витрат для вікової категорії від 18 до 25 років.

```Py
# Сумарні витрати
total_spending_18_25 = df_18_25.agg(
    _sum("total_cost").alias("total_spending")
).collect()[0]["total_spending"]
total_spending_18_25
```

Результат:

```v
1207.6
```

Тепер розрахуємо частку покупок

```Py
# Визначимо частку покупок за кожною категорією товарів від сумарних витрат для вікової категорії від 18 до 25 років
df_18_25.groupBy("category").agg(
    round(_sum("total_cost"), 2).alias("total_sum_18_25"),
    round(_sum("total_cost") / total_spending_18_25 * 100, 2).alias("part_sum_18_25"),
).show()
```

Результат:

```v
+-----------+---------------+--------------+
|   category|total_sum_18_25|part_sum_18_25|
+-----------+---------------+--------------+
|       Home|          361.1|          29.9|
|     Sports|          310.5|         25.71|
|Electronics|          249.6|         20.67|
|   Clothing|          245.0|         20.29|
|     Beauty|           41.4|          3.43|
+-----------+---------------+--------------+
```

### 6. Виберіть 3 категорії продуктів з найвищим відсотком витрат споживачами віком від 18 до 25 років.

💡 Відсоток потрібно округлити до другого знака після коми.

```Py
# 6. Виберіть 3 категорії продуктів з найвищим відсотком витрат споживачами віком від 18 до 25 років.
df_18_25.groupBy("category").agg(
    round(_sum("total_cost"), 2).alias("total_sum_18_25"),
    round(_sum("total_cost") / total_spending_18_25 * 100, 2).alias("part_sum_18_25"),
).sort("part_sum_18_25", ascending=False).limit(3).show()
```
Результат:
```v
+-----------+---------------+--------------+
|   category|total_sum_18_25|part_sum_18_25|
+-----------+---------------+--------------+
|       Home|          361.1|          29.9|
|     Sports|          310.5|         25.71|
|Electronics|          249.6|         20.67|
+-----------+---------------+--------------+
```
