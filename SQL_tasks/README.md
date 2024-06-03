# SQL-задачи

В нашем распоряжении 2 таблицы.  
В таблице "DEPARTMENT" есть столбцы:
- ID (NUMBER) <pk>;
- NAME (VARCHAR2)

В таблице "EMPLOYEE" следующие столбцы:
- ID (NUMBER) <pk>;
- DEPARTMENT_ID (NUMBER) <fk1>;
- CHIEF_ID (NUMBER) <fk2>;
- NAME (VARCHAR2);
- SALARY (NUMBER).

##### Задания:
1. Вывести список сотрудников в формате : Сотрудник, Отдел сотрудника, Руководитель, Отдел руководителя

SELECT e.NAME as employee_name, d.NAME as employee_department_name, ec.NAME as chief_name, dc.NAME as chief_department_name  
FROM EMPLOYEE e join DEPARTMENT d ON d.ID = e.DEPARTMENT_ID  
join EMPLOYEE ec ON e.CHIEF_ID = ec.ID  
join DEPARTMENT dc ON ec.department_id = dc.ID;


2. Вывести список сотрудников, получающих заработную плату, большую чем у непосредственного руководителя

SELECT e.NAME as employee_name, e.SALARY as employee_salary, c.NAME as chief_name, c.SALARY as chief_salary  
FROM EMPLOYEE e join EMPLOYEE c ON e.CHIEF_ID = c.ID  
WHERE e.SALARY > c.SALARY;


3. Вывести список сотрудников, получающих максимальную заработную плату в своем отделе

with chief_t as  
(SELECT DEPARTMENT_ID, max(SALARY) as max_salary FROM EMPLOYEE GROUP BY DEPARTMENT_ID)  
SELECT e.NAME as name_employee, d.NAME as name_department, ct.max_salary  
FROM EMPLOYEE e join DEPARTMENT d ON d.ID = e.DEPARTMENT_ID join chief_t ct ON e.DEPARTMENT_ID = ct.department_id  
WHERE e.salary = ct.max_salary;


4. Вывести список ID отделов, количество сотрудников в которых не превышает 3 человек

SELECT distinct DEPARTMENT_ID, count(ID) as count_employees  
FROM EMPLOYEE  
GROUP BY DEPARTMENT_ID  
HAVING COUNT(ID) <= 3;


5. Вывести список сотрудников, не имеющих назначенного руководителя, работающего в том же отделе

SELECT distinct NAME  
FROM EMPLOYEE   
WHERE CHIEF_ID is NULL  
GROUP BY NAME;


6. Вывести список наименований отделов с максимальной суммарной зарплатой сотрудников

with tab_salary as (SELECT DISTINCT DEPARTMENT_ID as department_id, SUM(salary) OVER (PARTITION BY department_id) as sum_salary FROM EMPLOYEE)

SELECT d.NAME as name_department, s.sum_salary  
FROM tab_salary s join DEPARTMENT d on d.ID = s.department_id  
GROUP BY d.NAME, s.sum_salary  
ORDER BY s.sum_salary DESC  
LIMIT 5;


7. Вывести ФИО сотрудника(ов), получающего третью по величине зарплату в организации

with tab_rank_salary as  
(SELECT NAME as employee_name, SALARY, RANK() OVER (ORDER BY SALARY DESC) AS rank_each_salary  
FROM EMPLOYEE  
SELECT employee_name  
FROM tab_rank_salary rs  
WHERE rs.rank_each_salary = 3;  


# Задания по поиску информации для SQL
Имеется таблица с номерами заявок, номерами телефона по клиентам в виде числового поля.
| №| MR_REQUEST_ID| MOB_NUM| CREDIT_SUM_REQ|
| :--------------------| :-------------------- | ---------------------: |:---------------------------:|
| 1| S8:1-52JK8J | 9166104609 | 413000 |
| 2| S8:1-52LHR | 9647005152 | 895000|
| 3| S8:1-2ELD0LO | 9686671399 | 140500|
| 4| S8:1-2EPLATF| 9260007533 | 645710|
| 5| S8:1-52Q8GW| 9153635802 | 695000|
| 6| S8:1-ZWZ1KS| 9164718620 | 736000|
| 7| S8:1-2EQMDCC| 9266847957| 900000|
| 8| S8:1-2ESD6HC| 9036155570| 300000|
| 9| S8:1-13A1Z5H| 9031448083| 385000|
| 10| S8:1-292D62YS| 9001031227| 300000|

#### Задания:
1. Поле MR_REQUEST_ID поменять на формат без приставки S8:
     пример: 1-52JK8J
##### Решение:
SELECT CONCAT(SUBSTRING(MR_REQUEST_ID, 3, LENGTH(MR_REQUEST_ID))) as request_id  
FROM tab

2. Поле MOB_NUM - является числом, и необходимо преобразовать его в текстовое и заменить последние 3 цифры на '999'
     пример: 9166104999
##### Решение:
SELECT CAST(CONCAT(SUBSTRING(MOB_NUM, 1, LENGTH(MOB_NUM) - 3), '999') as VARCHAR)  
FROM tab

3. Поле CREDIT_SUM_REQ перевести в тысячи с округлением до целого значения вверх.
     пример: 140500 преобразуется в 141
##### Решение:
SELECT ceil(CREDIT_SUM_REQ/1000.0) as round_credit_sum  
FROM tab


## Задачи (синтаксис MySQL)
![схема3](https://github.com/AyzaOyun/pet-projects/assets/144170277/ed5dbc0f-5438-4483-a622-53ba049a95b4)

#### Задача 1
Требуется написать запрос (SELECT), который вернет результаты по КОЛИЧЕСТВУ (amount) покупок товаров всеми членами семьи в разрезе кварталов (в данной песочнице представлены данные только за 2005 год). 

##### Решение:
with t1 as (SELECT fm.member_name as Name, g.good_name as Good, SUM(p.amount) as total_amount, EXTRACT(QUARTER from p.date) as q
FROM Goods g INNER JOIN Payments p on g.good_id = p.good INNER JOIN FamilyMembers fm on p.family_member = fm.member_id
WHERE p.date BETWEEN '2005-01-01' AND '2006-01-01'
GROUP BY fm.member_name, g.good_name, q
ORDER BY fm.member_name, g.good_name, q)

SELECT Name, Good, 
CASE WHEN q = 1 THEN total_amount ELSE 0 END as q1,
CASE WHEN q = 2 THEN total_amount ELSE 0 END as q2,
CASE WHEN q = 3 THEN total_amount ELSE 0 END as q3,
CASE WHEN q = 4 THEN total_amount ELSE 0 END as q4
FROM t1


#### Задача 2
Измените запрос так, чтобы показать 5 лучших продуктов, которые принесли больше всего денег (имели наивысшую совокупную стоимость) в отчетном году. Отобразите ранг результата по продуктам и расположите их в порядке убывания стоимости так, чтобы ранг 1 имел продукт с максимальной стоимостью и т.д.

##### Решение:
with t1 as (SELECT fm.member_name as Name, g.good_name as Good, SUM(p.unit_price*p.amount) as price, SUM(amount) as total_amount, EXTRACT(QUARTER from p.date) as q  
FROM Goods g INNER JOIN Payments p on g.good_id = p.good INNER JOIN FamilyMembers fm on p.family_member = fm.member_id  
WHERE p.date BETWEEN '2005-01-01' AND '2006-01-01'  
GROUP BY fm.member_name, g.good_name, q  
ORDER BY fm.member_name, g.good_name, q),  

t2 as (SELECT Good,   
CASE WHEN q = 1 THEN price ELSE 0 END as q1,  
CASE WHEN q = 2 THEN price ELSE 0 END as q2,  
CASE WHEN q = 3 THEN price ELSE 0 END as q3,  
CASE WHEN q = 4 THEN price ELSE 0 END as q4,  
SUM(price) OVER (PARTITION BY Good, q) AS total_year  
FROM t1)  

SELECT Good, q1, q2, q3, q4, total_year, RANK() OVER (ORDER BY total_year DESC) as Rank_  
FROM t2  
LIMIT 5  
