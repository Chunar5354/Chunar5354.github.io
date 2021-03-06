---
layout: article
tags: Leetcode SQL
title: Leetcode problems of sql
article_header:
  type: overlay
  theme: dark
  background_color: '#123'
  background_image: false
---

SQL problems in leetcode.

<!--more-->

There are questions of database.

## 175.Combine Two Tables

[Problem link](https://leetcode.com/problems/combine-two-tables/)

- My approach

To conmbine two tables, MySQL provides a method `join`, and in this problem, we should use `left join`.

For more details about `join`, please see [here](https://blog.csdn.net/Jintao_Ma/article/details/51260458)

```sql
SELECT FirstName, LastName, City, State
FROM Person
LEFT JOIN Address
on Person.PersonId = Address.PersonId;
```

And if we use a parameter to represent the table name, it will run faster.

```sql
select p.FirstName, p.LastName, a.City, a.State from Person p left join Address a on p.PersonId = a.PersonId;
```


## 176.Second Highest Salary


[Problem link](https://leetcode.com/problems/second-highest-salary/)

- My approach

To search the second highest data, we can search the maximum data from the collection which is smaller than the highest data.

```sql
select max(Salary) as SecondHighestSalary from Employee where Salary < (select max(Salary) from Employee);
```

But if we want to search the third highest or more smaller data, it will be very complex.

There is a more universal method by using `limit`.

```sql
select ifnull((select distinct Salary from Employee order by Salary desc limit 1,1), null) as SecondHighestSalary;
```

There are three points:

  - `limit 1, 1` means we select the result from the second line to the second line
  - `ifnull((result), other)` means if the result in bracket is null, we use other to replace the result
  - `distinct` means avoid the duplicated values while selecting


## 177.Nth Highest Salary

[Problem link](https://leetcode.com/problems/nth-highest-salary/)

- My approach

This problem creates a function in sql.

There is a point that in sql query, there can't be expression such as `N-1`. So if we want to calculate some values, we should declare 
a parament before.

```sql
CREATE FUNCTION getNthHighestSalary(N INT) RETURNS INT
BEGIN
# Create a new parament M = N-1
DECLARE M INT;
SET M=N-1;
RETURN (
      # Write your MySQL query statement below.
      SELECT DISTINCT salary FROM Employee ORDER BY salary DESC LIMIT M, 1
  );
END
```


## 178.Rank Scores

[Problem link](https://leetcode.com/problems/rank-scores/)

To solve the rank problem, we can use MySQL function `DENSE_RANK`. This function can rank the values order by certain data.

```sql
select Score, dense_rank() over (order by Score desc) as 'Rank' from Scores;
```

Pay attention to the usage `dense_rank() over()`. And the new name after `as` should be in quotes.

For more details of `DENSE_RANK` please see [here](https://www.begtut.com/mysql/mysql-dense_rank-function.html).


## 180.Consecutive Numbers

[Problem link](https://leetcode.com/problems/consecutive-numbers/)

- My approach

To solve this problem, we can use `and` method to select three consecutive numbers.

```sql
# Write your MySQL query statement below
SELECT DISTINCT
    l1.Num AS ConsecutiveNums
FROM
    Logs l1,
    Logs l2,
    Logs l3
WHERE
    l1.Id = l2.Id - 1
    AND l2.Id = l3.Id - 1
    AND l1.Num = l2.Num
    AND l2.Num = l3.Num
;
```

And there is another method.

```sql
select distinct Num as ConsecutiveNums from Logs
where (Id+1,Num) in (select Id, Num from Logs) 
and (Id+2,Num) in (select Id, Num from Logs)
```


## 181.Employees Earning More Than Their Managers

[Problem link](https://leetcode.com/problems/employees-earning-more-than-their-managers/)

- My approach

To solve this problem, we need to search the table twice. So we can use `join` to connect the two searching.

```sql
# Write your MySQL query statement below
SELECT
     a.NAME AS Employee
FROM Employee AS a JOIN Employee AS b
     ON a.ManagerId = b.Id
     AND a.Salary > b.Salary
;
```


## 182.Duplicate Emails

[Problem link](https://leetcode.com/problems/duplicate-emails/)

- My approach

We can use `join` to select the table twice.

```sql
# Write your MySQL query statement below
select distinct a.Email from Person as a join Person as b where a.Email = b.Email and a.Id != b.Id;
```


## 183.Customers Who Never Order

[Problem link](https://leetcode.com/problems/customers-who-never-order/)

- My approach

Check if data from table 1 is in the select result of table 2.

```sql
# Write your MySQL query statement below
select c.name as Customers from Customers as c where c.Id not in (select CustomerId from Orders);
```


## 184.Department Heightest Salary

[Problem link](https://leetcode.com/problems/department-highest-salary/)

- My approach

We can use `join...on` to connect two tables.

```sql
# Write your MySQL query statement below
select d.Name as 'Department', e.Name as 'Employee', e.Salary
from Employee as e join Department as d on e.DepartmentId = d.Id
where (e.DepartmentId, e.Salary) in (select departmentId, max(Salary) from Employee group by DepartmentId);
```


## 185.Department Top Three Salaries

[Problem link](https://leetcode.com/problems/department-top-three-salaries/)

- My approach

Because we need to select more than one row and divide groups, we can't use `limit` here.

To do group, we can select the table twice, and set a condition that `e1.Department = e2.Department`.

And to select the top three results, we can use `count`.

```sql
SELECT
    d.Name AS 'Department', e1.Name AS 'Employee', e1.Salary
FROM
    Employee e1
        JOIN
    Department d ON e1.DepartmentId = d.Id
WHERE
    3 > (SELECT
            COUNT(DISTINCT e2.Salary)
        FROM
            Employee e2
        WHERE
            e2.Salary > e1.Salary
                AND e1.DepartmentId = e2.DepartmentId
        )
;
```


## 196.Delete Duplicated Emails

[Problem link](https://leetcode.com/problems/delete-duplicate-emails/)

- My approach

We can solve this problem by making the table to two same tables.

```sql
delete p1 from Person as p1, Person as p2 where p1.Email = p2.Email and p1.Id > p2.Id;
```

And there is a method to create a derived table.

```sql
DELETE
FROM Person 
WHERE Id
NOT IN (select id from (SELECT MIN(ID) as Id
       FROM Person
       GROUP BY Email) t1)
```

Pay attention to the `t1`, when create a dreived table, we must give it a alia.


## 197.Rising Temperature

[Problem link](https://leetcode.com/problems/rising-temperature/)

- My approach

We can solve this problem by copying the table into two tables.

```sql
select w1.Id from Weather as w1, Weather as w2 
where w1.Temperature > w2.Temperature and datediff(w1.RecordDate, w2.RecordDate) = 1;
```

Pay attention to the function `datediff`, it can calculate the different of two dates, including the situation that the two days are in 
two months or two years.

We can also use `join` method to solve this problem.

```sql
select w1.Id from Weather as w1 
    join Weather as w2 on w1.Temperature > w2.Temperature 
                       and datediff(w1.RecordDate, w2.RecordDate) = 1;
```


## 262. Trips and Users

[Problem link](https://leetcode.com/problems/trips-and-users/)

- Other's approach

```sql
# Write your MySQL query statement below
WITH T1 AS 
(
SELECT *
FROM TRIPS AS T
LEFT JOIN USERS AS U 
ON T.CLIENT_ID = U.USERS_ID OR T.DRIVER_ID = U.USERS_ID
WHERE  CLIENT_ID IN (SELECT DISTINCT USERS_ID FROM USERS WHERE BANNED = 'NO')
AND DRIVER_ID IN (SELECT DISTINCT USERS_ID FROM USERS WHERE BANNED = 'NO')
AND REQUEST_AT BETWEEN DATE('2013-10-01') AND DATE('2013-10-03')
)

SELECT REQUEST_AT AS DAY,
ROUND((COUNT(CASE WHEN STATUS ='cancelled_by_driver' OR STATUS = 'cancelled_by_client' THEN ID END)/COUNT(*)),2) AS 'CANCELLATION RATE'
FROM T1 
GROUP BY REQUEST_AT
```

There are some points:

- 1.Using `with... as` to create a new combination table with the given conditions.

- 2.Using `case... then` to divide groups
