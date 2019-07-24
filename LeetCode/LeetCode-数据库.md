#### 查找重复的电子邮箱-182

编写一个 SQL 查询，查找 `Person` 表中所有重复的电子邮箱。

```html
输入：
+----+---------+
| Id | Email   |
+----+---------+
| 1  | a@b.com |
| 2  | c@d.com |
| 3  | a@b.com |
+----+---------+
输出：
+---------+
| Email   |
+---------+
| a@b.com |
+---------+
```

方法一：使用 `GROUP BY` 和临时表

```sql
select Email from
(
  select Email, count(Email) as num
  from Person
  group by Email
) as t
where num > 1;

```

方法二：使用 `GROUP BY` 和 `HAVING` 条件

```sql
select Email from Person group by Email having count(Email) > 1;
```



#### 有趣的电影-620

某城市开了一家新的电影院，吸引了很多人过来看电影。该电影院特别注意用户体验，专门有个 LED显示板做电影推荐，上面公布着影评和相关电影描述。

作为该电影院的信息部主管，您需要编写一个 SQL查询，找出所有影片描述为非 boring (不无聊) 的并且 id 为奇数 的影片，结果请按等级 rating 排列。

```html
输入：
+---------+-----------+--------------+-----------+
|   id    | movie     |  description |  rating   |
+---------+-----------+--------------+-----------+
|   1     | War       |   great 3D   |   8.9     |
|   2     | Science   |   fiction    |   8.5     |
|   3     | irish     |   boring     |   6.2     |
|   4     | Ice song  |   Fantacy    |   8.6     |
|   5     | House card|   Interesting|   9.1     |
+---------+-----------+--------------+-----------+
输出：
+---------+-----------+--------------+-----------+
|   id    | movie     |  description |  rating   |
+---------+-----------+--------------+-----------+
|   5     | House card|   Interesting|   9.1     |
|   1     | War       |   great 3D   |   8.9     |
+---------+-----------+--------------+-----------+
```

```sql
select * from cinema where description <> 'boring' and mod(id,2) = 1 order by rating desc;
select * from cinema where description != 'boring' and i%2 = 1 order by rating desc;
```



#### 组合两个表-175

```html
表1：Person
+-------------+---------+
| 列名         | 类型     |
+-------------+---------+
| PersonId    | int     |
| FirstName   | varchar |
| LastName    | varchar |
+-------------+---------+
PersonId 是上表主键

表2：Address
+-------------+---------+
| 列名         | 类型    |
+-------------+---------+
| AddressId   | int     |
| PersonId    | int     |
| City        | varchar |
| State       | varchar |
+-------------+---------+
AddressId 是上表主键
```

思路：使用 left join 外连接，返回左表所有的信息，右表匹配的信息。

```sql
select FirstName, LastName, City, State from Person p left join Address a on a.PersonId = p.PersonId;
```



#### 交换工资-627

给定一个 salary 表，如下所示，有 m = 男性 和 f = 女性 的值。交换所有的 f 和 m 值（例如，将所有 f 值更改为 m，反之亦然）。要求只使用一个更新（Update）语句，并且没有中间的临时表。

注意，您必只能写一个 Update 语句，请不要编写任何 Select 语句。

```html
输入：
| id | name | sex | salary |
|----|------|-----|--------|
| 1  | A    | m   | 2500   |
| 2  | B    | f   | 1500   |
| 3  | C    | m   | 5500   |
| 4  | D    | f   | 500    |
输出：
| id | name | sex | salary |
|----|------|-----|--------|
| 1  | A    | f   | 2500   |
| 2  | B    | m   | 1500   |
| 3  | C    | f   | 5500   |
| 4  | D    | m   | 500    |
```

两种方法：1.使用 CASE..WHEN..THEN..ELSE..END；2.使用 if 判断；

```sql
update salary set sex = case sex when 'm' then 'f' else 'm' end;
update salary set sex = if(sex = 'f', 'm', 'f');
```



#### 超过经理收入的员工-181

`Employee` 表包含所有员工，他们的经理也属于员工。每个员工都有一个 Id，此外还有一列对应员工的经理的 Id。

```
输入：
+----+-------+--------+-----------+
| Id | Name  | Salary | ManagerId |
+----+-------+--------+-----------+
| 1  | Joe   | 70000  | 3         |
| 2  | Henry | 80000  | 4         |
| 3  | Sam   | 60000  | NULL      |
| 4  | Max   | 90000  | NULL      |
+----+-------+--------+-----------+
输出：
+----------+
| Employee |
+----------+
| Joe      |
+----------+
```

写法1：

从两个表里使用 Select 语句可能会导致产生 笛卡尔乘积 。在这种情况下，输出会产生 4*4=16 个记录。然而我们只对雇员工资高于经理的人感兴趣。所以我们应该用 WHERE 语句加 2 个判断条件。

```sql
SELECT
    a.Name AS 'Employee'
FROM
    Employee AS a,
    Employee AS b
WHERE
    a.ManagerId = b.Id
        AND a.Salary > b.Salary;
```

写法2：使用 join 表连接

```sql
select a.name as Employee
from employee a inner join employee b on a.managerid = b.id 
and a.salary > b.salary;
```

写法3：使用子查询

```sql
select name as Employee from employee a where salary >
 (select salary from employee where a.managerid = id  ) 
```



#### 从不订购的客户-183

某网站包含两个表，`Customers` 表和 `Orders` 表。编写一个 SQL 查询，找出所有从不订购任何东西的客户。

```html
Customers:
+----+-------+
| Id | Name  |
+----+-------+
| 1  | Joe   |
| 2  | Henry |
| 3  | Sam   |
| 4  | Max   |
+----+-------+
Orders:
+----+------------+
| Id | CustomerId |
+----+------------+
| 1  | 3          |
| 2  | 1          |
+----+------------+

输出：
+-----------+
| Customers |
+-----------+
| Henry     |
| Max       |
+-----------+
```

两种方式：1.使用左连接；2.使用子查询

```sql
select c.Name as Customers form customers c left join orders o on c.Id = o.customersId where o.customersId is null;

select c.Name as Customers from customers c where c.Id not in (select customerId from orders);
```



#### 删除重复的电子邮箱-196

编写一个 SQL 查询，来删除 `Person` 表中所有重复的电子邮箱，重复的邮箱里只保留 **Id** *最小* 的那个。

```
+----+------------------+
| Id | Email            |
+----+------------------+
| 1  | john@example.com |
| 2  | bob@example.com  |
| 3  | john@example.com |
+----+------------------+
Id 是这个表的主键。
```

例如，在运行你的查询语句之后，上面的 `Person` 表应返回以下几行：

```
+----+------------------+
| Id | Email            |
+----+------------------+
| 1  | john@example.com |
| 2  | bob@example.com  |
+----+------------------+
```

先把表自连接，然后从表1中删除 id 较小的记录

```sql
DELETE p1 FROM Person p1,
    Person p2
WHERE
    p1.Email = p2.Email AND p1.Id > p2.Id;
```

