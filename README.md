master> create database db1
master> use db1
db1> create schema dbo

# Практическое задание №1

```sql
-- 1. создание таблицы dept
create table dept
(
    dept_id int
        constraint dept_pk
            primary key nonclustered,
    dname   varchar(20)
);

-- 1. создание таблицы emp
create table emp
(
    emt_id  int
        constraint emp_pk
            primary key nonclustered,
    dept_id int
        constraint emp_dept_dept_id_fk
            references dept,
    ename   varchar(15),
    salary  numeric(6, 2)
);

-- 2. вставляем данные в таблицу dept
INSERT INTO dept (dept_id, dname)
VALUES (1, 'Marketing'),
       (2, 'RD');

-- 2. вставляем данные в таблицу emp
INSERT INTO emp (emt_id, dept_id, ename, salary)
VALUES (1, 1, 'James', 1000),
       (2, 2, 'Smith', 2000);

-- 3. создание таблицы dept_arch
create table dept_arch
(
    dept_id int
        constraint dept_arch_pk
            primary key nonclustered,
    dname   varchar(20)
);

-- 4. вставить в таблицу dept_arch все данные из таблицы dept
INSERT INTO dept_arch SELECT * FROM dept;

-- 5. Увеличьте на 15% зарплату сотруднику Smith
UPDATE emp SET salary = salary * 1.15 WHERE ename='Smith';

-- 6. Убедитесь, что в таблицу dept нельзя вставить такую запись: (2, 'Sales'). Почему?
-- INSERT INTO dept (dept_id, dname) VALUES (2, 'Sales');
-- [23000][2627] Violation of PRIMARY KEY constraint 'dept_pk'. Cannot insert duplicate key in object 'dbo.dept'. The duplicate key value is (2).
-- Получили ошибку Cannot insert duplicate key. Так как уже есть запись, где dept_id=2

-- 7. Убедитесь, что в таблицу emp нельзя вставить такую запись: (3, 4, 'Black', 3000, 'Active'). Почему?
-- INSERT INTO emp (emt_id, dept_id, ename, salary) VALUES (3, 4, 'Black', 3000, 'Active');
-- [S0001][110] There are fewer columns in the INSERT statement than values specified in the VALUES clause. The number of values in the VALUES clause must match the number of columns specified in the INSERT statement.
-- Получили ошибку There are fewer columns. То есть у нас в VALUES указаны лишние значения, и база не знает, куда вставлять значение 'Active'

-- 8. Измените название отдела RD на RandD (таблица dept)
UPDATE dept SET dname='RandD' WHERE dname='RD';

-- 9. Удалите из таблицы emp запись с emp_id = 1.
DELETE FROM emp WHERE emt_id=1;

-- 10. Удалите из таблицы emp все записи
-- DELETE FROM emp WHERE 1;
TRUNCATE TABLE emp;

-- 11. Удалите таблицу emp
DROP TABLE emp;
```


# Практическое задание №2

```sql
-- Создать схему
create table person
(
    person_code varchar(3)
        constraint person_pk
            primary key nonclustered,
    first_name  varchar(15),
    last_name   varchar(20),
    hiredate    DATE
);

create table product
(
    product_name     varchar(25)
        constraint product_pk
            primary key nonclustered,
    product_price    numeric(8, 2),
    quantity_on_hand numeric(5),
    laststockdate    date
);

create table purchase
(
    product_name  varchar(25)
        constraint purchase_product_product_name_fk
            references product,
    salesperson   varchar(3)
        constraint purchase_person_person_code_fk
            references person,
    purchase_date date,
    quantity      numeric(4, 2)
);

create table purchase_archive
(
    product_name  varchar(25),
    salesperson   varchar(3),
    purchase_date date,
    quantity      numeric(4, 2)
);

create table old_item
(
    item_id   char(20),
    item_desc char(25)
);

-- Заполнить таблицы
INSERT INTO person (person_code, first_name, last_name, hiredate)
VALUES ('VAN', 'Ivan', 'Ivanov', '2010-02-01'),
       ('SER', 'Sergey', 'Serov', '2020-08-07'),
       ('BAL', 'Alex', 'Balabanov', '2020-08-07'),
       ('BOB', 'Vasiliy', 'Bobrov', '2010-01-01'),
       ('SOL', 'Sokol', 'Soloviov', '2010-01-01');

INSERT INTO product (product_name, product_price, quantity_on_hand, laststockdate)
VALUES ('milk', 12.50, 2.5, '2020-08-05'),
       ('beaf', 35.50, 5.50, '2020-08-23'),
       ('cheese', null, null, null),
       ('tea', 10.50, 5.50, '2020-08-10'),
       ('Small Widget', 35.50, 5.50, '2020-08-23'),
       ('Medium Widget', 35.50, 5.50, '2020-08-13'),
       ('Large Widget', 35.50, 5.50, '2020-08-03');

INSERT INTO purchase (product_name, salesperson, purchase_date, quantity)
VALUES ('milk', 'VAN', '2020-08-05', 3),
       ('beaf', 'VAN', '2020-08-06', 2.5),
       ('milk', 'SER', '2020-08-07', 24),
       ('milk', 'VAN', '2020-08-07', 20),
       ('beaf', 'SER', '2020-08-08', 2.5),
       ('tea', 'BAL', '2020-08-23', 20),
       ('cheese', 'SER', '2020-08-08', 23);

INSERT INTO purchase_archive
SELECT *
FROM purchase;
INSERT INTO purchase_archive (product_name, salesperson)
VALUES ('milk', 'SOL');

-- 1. Напишите запрос, полностью показывающий таблицу purchase.
SELECT *
FROM purchase;

-- 2. Напишите запрос, выбирающий столбцы product_name и quantity из таблицы Purchase.
SELECT product_name, quantity
FROM purchase;

-- 3. Напишите запрос, выбирающий эти столбцы в обратном порядке.
SELECT quantity, product_name
FROM purchase;

-- 4. Напишите запрос, выводящий для каждой строки таблицы person следующий текст:
-- <first_name> <last_name> started work <hiredate>*. Получаемому столбцу присвоить псевдоним "Started Work”.
SELECT CONCAT(first_name, ' ', last_name, ' started work ', hiredate) as 'Started Work'
FROM person;

-- 5. Напишите запрос,выводящий наименование продуктов product_name (таблица product), для которых цена не определена (NULL).
SELECT product_name
FROM product
WHERE product_price IS NULL;

-- 6. Напишите запрос, выводящий наименование продуктов product_name (таблица purchase), которых продали от 3 до 23 штук.
-- * MSSQL не поддерживает объединение столбцов с типами данных varchar и date. Используйте оператор конветации:  CONVERT(VARCHAR, hiredate)
SELECT product_name
FROM purchase pur
WHERE quantity BETWEEN 3 AND 23
GROUP BY product_name;

-- 7. Напишите запрос, выводящий фамилии сотрудников, которых приняли на работу 1го, 15го и 28го февраля 2010 года.
SELECT last_name
FROM person
WHERE hiredate IN ('2010-02-01', '2010-02-15', '2010-02-28')

-- 8. Напишите запрос, выводящий наименование продуктов product_name (таблица purchase), проданных сотрудниками, фамилии которых начинаются на "B”.
SELECT pur.product_name
FROM purchase pur
         LEFT JOIN person p on pur.salesperson = p.person_code
WHERE p.last_name LIKE 'B%'
GROUP BY pur.product_name;

-- 9. Напишите запрос, выводящий наименование продуктов product_name (таблица purchase), проданных сотрудниками, фамилии которых НЕ начинаются на "B”.
SELECT pur.product_name
FROM purchase pur
         LEFT JOIN person p on pur.salesperson = p.person_code
WHERE p.last_name NOT LIKE 'B%'
GROUP BY pur.product_name;

-- 10. Напишите запрос, выводящий фамилии и дату приема на работу сотрудников, фамилии которых начинаются на "B” и которых приняли на работу раньше 1 марта 2010 года.
SELECT last_name, hiredate
FROM person
WHERE last_name LIKE 'B%'
  AND hiredate < '2010-03-01';

-- 11. Напишите запрос, выводящий наименование продуктов product_name и дату последней поставки laststockdate (таблица product),
-- наименование которых Small Widget, Medium Widget и Large Widget или те, для которых не указана дата последней поставки.
-- Отсортируйте по убыванию даты последней поставки.
SELECT product_name, laststockdate FROM product WHERE product_name IN ('Small Widget', 'Medium Widget', 'Large Widget') OR laststockdate IS NULL ORDER BY laststockdate;


-- drop table purchase;
-- drop table old_item;
-- drop table person;
-- drop table product;
-- drop table purchase_archive;
```


# Практическое задание №3

```sql
-- 1. Напишите запрос, выводящий декартово произведение таблиц product и purchase.
SELECT product.*, purchase.*
FROM product
         CROSS JOIN purchase;

-- 2. Напишите запрос, выводящий наименование проданного товара product_name,
-- количество quantity (таблица purchase) и quantity_on_hand (таблица product).
SELECT p.product_name, sum(quantity) qty, sum(quantity_on_hand) on_hand
FROM product p
         LEFT JOIN purchase pur on p.product_name = p.product_name
GROUP BY p.product_name;

-- 3.Напишите запрос, выводящий наименование товара product_name (таблица purchase),
-- дату последней поставки laststockdate (таблица product) и фамилию продавца last_name (таблица person).
SELECT pur.product_name, p.laststockdate, per.last_name
FROM purchase pur
         LEFT JOIN product p on pur.product_name = p.product_name
         LEFT JOIN person per on pur.salesperson = per.person_code;

-- 4. Напишите запрос, выводящий столбцы product_name, first_name, last_name внешнего объединения таблиц purchase и person.
-- Используйте для таблиц короткие псевдонимы.
SELECT product_name
FROM purchase pur
         FULL OUTER JOIN person p on pur.salesperson = p.person_code;
SELECT product_name
FROM purchase pur
         LEFT OUTER JOIN person p on pur.salesperson = p.person_code;

-- 5.Напишите запрос, который выводит коды продавцов salesperson из таблицы purchase_archive , которые не повторяются в таблице purchase.
SELECT a.salesperson
FROM purchase_archive a
EXCEPT
SELECT p.salesperson
FROM purchase p;


-- 6. Напишите запрос, который выводит коды только тех продавцов salesperson из таблицы purchase,
-- которые так же содержаться в таблице purchase_archive.
SELECT a.salesperson
FROM purchase_archive a
INTERSECT
SELECT p.salesperson
FROM purchase p;

-- 7. Напишите запрос, который выводит все (в том числе повторяющиеся) коды продавцов salesperson
-- из таблиц purchase и purchase_archive.
SELECT a.salesperson
FROM purchase_archive a
UNION
SELECT p.salesperson
FROM purchase p;
```

# Практическое задание №4

```sql
-- 1. Напишите запрос, показывающий, какой будет цена продукта product_price после увеличения на 15%.
SELECT product_name, product_price * 1.15 as new_price
FROM product
WHERE product_price IS NOT NULL;

-- 2. Напишите запрос, показывающий, сколько всего имеется товаров в таблице product.
SELECT count(*)
FROM product;

-- 3.Напишите запрос, показывающий, для какого количества товаров (таблица product) не указана цена.
SELECT product_name
FROM product
WHERE product_price IS NULL;

-- 4. Напишите запрос, выводящий минимальную и максимальную цену товаров product_price.
SELECT min(product_price), max(product_price)
FROM product;

-- 5. Напишите запрос, показывающий, какая сумма была выручена с продаж товаров каждого наименования.
SELECT pur.product_name, sum(quantity * p.product_price) as sum
FROM purchase pur
         LEFT JOIN product p on pur.product_name = p.product_name
GROUP BY pur.product_name;

-- 6. Напишите запрос, показывающий, какая сумма была выручена с продаж товаров каждого наименования. Вывести только те записи, для которых сумма продаж больше 125.
SELECT pur.product_name, sum(quantity * p.product_price) as sum
FROM purchase pur
         LEFT JOIN product p on pur.product_name = p.product_name
GROUP BY pur.product_name
HAVING sum(quantity * p.product_price) > 125;
```