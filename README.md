# PySpark_project
# Тема работы

Сбор, предобработка и анализ данных о работе сервиса по доставке продуктов.

## 1.1 Информация о выбранном наборе данных

Данные о доставке продуктов различных компаний были взяты с сайта [karpov courses](https://lab.karpov.courses)

![image](https://github.com/Fourzeroo/PySpark_project/assets/92236009/028ea826-a425-496b-b208-a0e2e16198c1)

## 1.2 Архитектура конвейера для получения и предобработки данных 

Схема конвейера представлена ниже:

![image](https://github.com/Fourzeroo/PySpark_project/assets/92236009/05398d68-89ed-4b84-a4e8-019d73514133)

Для начала мы очищаем немного данные и приводим их к нормальному виду для того, чтобы в дальнейшем данные могли хранится в таблицах Hive. В очистку данных входит преобразование времени к стандарту ISO 8601 и удаление квадратных скобок в колонке product_ids и т.д.

![image](https://github.com/Fourzeroo/PySpark_project/assets/92236009/0645291d-99d7-4616-ac4e-88976ea4f65e)


После предобработки данные можно помещать в локальное хранилище HDFS

```bash
[student@localhost ~]$ hdfs dfs -mkdir /user/student/kursovaya
[student@localhost ~]$ hdfs dfs -put /home/student/Data/orders.csv /user/student/kursovaya
[student@localhost ~]$ hdfs dfs -put /home/student/Data/couriers.csv /user/student/kursovaya
[student@localhost ~]$ hdfs dfs -put /home/student/Data/courier_actions.csv /user/student/kursovaya
[student@localhost ~]$ hdfs dfs -put /home/student/Data/users.csv /user/student/kursovaya
[student@localhost ~]$ hdfs dfs -put /home/student/Data/user_actions.csv /user/student/kursovaya
[student@localhost ~]$ hdfs dfs -put /home/student/Data/products.csv /user/student/kursovaya
```
Далее создадим таблицы в Hive для дальнейшего транспортирования наших файлов в HDFS в таблицы Hive. Все это будет выполнено в веб-приложении HUE для просмотра хранилища, связанного с Hadoop кластером, запуска Hive заданий, Pig скриптов и т.д.

```sql

-- создание таблицы courier_actions 
CREATE TABLE courier_actions (
  courier_id INT,
  order_id INT,
  action VARCHAR(50),
  deliver_time TIMESTAMP
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
STORED AS TEXTFILE
TBLPROPERTIES ("skip.header.line.count"="1");


-- создание таблицы couriers 
CREATE TABLE couriers (
  courier_id INT,
  birth_date DATE,
  sex VARCHAR(50)
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
STORED AS TEXTFILE
TBLPROPERTIES ("skip.header.line.count"="1");


-- создание таблицы orders 
CREATE TABLE orders (
  order_id INT,
  creation_time TIMESTAMP,
  product_ids ARRAY<INT>
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
COLLECTION ITEMS TERMINATED BY ' '
STORED AS TEXTFILE
TBLPROPERTIES ("skip.header.line.count"="1");


-- создание таблицы products 
CREATE TABLE products (
  product_id INT,
  name VARCHAR(70),
  price DECIMAL(10, 2)
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
STORED AS TEXTFILE
TBLPROPERTIES ("skip.header.line.count"="1");


-- создание таблицы user_actions 
CREATE TABLE user_actions (
  user_id INT,
  order_id INT,
  action VARCHAR(50),
  deliver_time TIMESTAMP
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
STORED AS TEXTFILE
TBLPROPERTIES ("skip.header.line.count"="1");


-- создание таблицы users 
CREATE TABLE users (
  user_id INT,
  birth_date DATE,
  sex VARCHAR(50)
)
ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
STORED AS TEXTFILE
TBLPROPERTIES ("skip.header.line.count"="1");
```

После создания таблиц мы переместим туда наши данные из HDFS:
```sql
LOAD DATA INPATH '/user/student/kursovaya/orders.csv' INTO TABLE kursdb.orders;
LOAD DATA INPATH '/user/student/kursovaya/couriers.csv' INTO TABLE kursdb.orders;
LOAD DATA INPATH '/user/student/kursovaya/courier_actions.csv' INTO TABLE kursdb.orders;
LOAD DATA INPATH '/user/student/kursovaya/users.csv' INTO TABLE kursdb.orders;
LOAD DATA INPATH '/user/student/kursovaya/user_actions.csv' INTO TABLE kursdb.orders;
LOAD DATA INPATH '/user/student/kursovaya/products.csv' INTO TABLE kursdb.orders;
```

**После этого данные идут по двум путям:**

1.	Данные идут в PySpark с помощью SparkSQL для дальнейшего анализа
2.	Рассчитанные метрики и другие важные показатели при помощи языка запросов HiveQL и с помощью Sqoop поступают в созданные таблицы в MariaDB
