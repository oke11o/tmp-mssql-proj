master> create database db1
master> use db1
db1> create schema dbo

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