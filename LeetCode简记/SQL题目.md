# SQL题目
---
## 1107.每日新用户统计
>新方法！DATE_SUB+CTE公共表表达式详解！
> `DATE_SUB` 是 MySQL 中用于“减去时间”的函数，常搭配 `CURDATE()` 或 `NOW()` 用于近 X 天数据查询；CTE 是用 `WITH` 开头的公共表表达式，能临时抽取中间查询结果，使 SQL 更简洁，适用于复杂查询、递归查询或子查询优化场景。
### 一、DATE_SUB()函数用法

#### 1. 作用：
`DATE_SUB(date, INTERVAL expr unit)` 用于从某个日期中减去指定的时间间隔，常用于时间窗口的查询。

#### 2. 基本语法：
```sql
SELECT DATE_SUB('2023-04-10', INTERVAL 7 DAY);
-- 结果：'2023-04-03'
```
常用单位：`DAY` 天、 `MONTH` 月、`YEAR` 年、`HOUR` 小时、`MINUTE` 分钟

#### 3. 实战举例：
```sql
SELECT *
FROM orders
WHERE order_date >= DATE_SUB(CURDATE(), INTERVAL 30 DAY);
-- 查询近30天订单
```

### 二、CTE（公共表表达式）

#### 1. 作用：
CTE 是一种临时的结果集，可以在主查询中多次引用，语法更清晰、结构更清楚，适用于递归查询、多层嵌套优化、统一复杂子查询。

#### 2. 基本语法：
```sql
WITH cte_name AS (
    SELECT column1, column2
    FROM table_name
    WHERE condition
)
SELECT *
FROM cte_name
WHERE column2 > 100;
```

#### 3. 应用场景举例：

##### ① 结构清晰的多层查询
```sql
WITH recent_orders AS (
    SELECT * FROM orders
    WHERE order_date >= DATE_SUB(CURDATE(), INTERVAL 7 DAY)
)
SELECT customer_id, COUNT(*) as order_count
FROM recent_orders
GROUP BY customer_id;
```

##### ② 递归查询（例如层级结构）
```sql
WITH RECURSIVE org_chart AS (
    SELECT id, name, manager_id
    FROM employees
    WHERE manager_id IS NULL
    UNION ALL
    SELECT e.id, e.name, e.manager_id
    FROM employees e
    INNER JOIN org_chart o ON e.manager_id = o.id
)
SELECT * FROM org_chart;
```

### 三、Codes

```mysql
WITH temp AS(
    SELECT user_id,MIN(activity_date) AS activity_date
    FROM Traffic
    WHERE activity='login'
    GROUP BY user_id
)

SELECT activity_date AS login_date,COUNT(*) AS user_count
FROM temp
WHERE activity_date BETWEEN DATE_SUB('2019-06-30',INTERVAL 90 DAY) AND '2019-06-30'
GROUP BY login_date
ORDER BY login_date;
```
---

好的，我们换一种方法来理解这道题，并构建 SQL 查询。这次我们将从**事件类型**的角度出发，将不同类型的交易事件分开处理，然后汇总。

------

## 1205.每日交易Ⅱ

### 一、解题目的核心挑战

这道题最容易让人困惑的地方是：

1. **日期依赖不同：** `批准交易` 的日期是 `Transactions.trans_date`，而 `退单` 的日期是 `Chargebacks.trans_date`。
2. **金额归属：** `退单金额` 需要从 `Transactions` 表中获取，而不是 `Chargebacks` 表。
3. **统计类别互斥：** `approved_count/amount` 和 `chargeback_count/amount` 是两个独立的统计维度，它们不会互相包含。也就是说，一笔交易如果被计入 `approved`，就不会再计入 `chargeback`（除非它同时是一笔批准的交易又发生了退单，但根据题目设计，这两个统计口径的日期基准是分开的）。

------

### 二、“分而治之”的策略

我们可以把问题拆解成两个独立的子问题，然后把它们的结果合并起来：

#### 子问题 1: 统计“批准的交易”

这部分比较简单，我们只需要关注 `Transactions` 表中 `state = 'approved'` 的记录。

- **数据来源：** `Transactions` 表。
- **筛选条件：** `state = 'approved'`。
- **日期维度：** `Transactions.trans_date` 的年-月。
- **统计内容：** `approved_count` 和 `approved_amount`。

#### 子问题 2: 统计“退单”

这部分稍微复杂一点，因为需要连接两个表，并且日期以 `Chargebacks` 表为准。

- **数据来源：** `Chargebacks` 表 **和** `Transactions` 表（通过 `trans_id` 连接）。
- **筛选条件：** `Chargebacks` 表中存在的记录。
- **日期维度：** `Chargebacks.trans_date` 的年-月。
- **统计内容：** `chargeback_count` 和 `chargeback_amount`。

------

### 三、构建 SQL 查询

#### 步骤 1: 准备“批准交易”的数据

我们从 `Transactions` 表中选出批准的交易，并计算它们的数量和金额。为了和后续的退单数据合并，我们把 `chargeback` 相关的字段设为 0。

SQL

```mysql
SELECT
    DATE_FORMAT(T.trans_date, '%Y-%m') AS month, -- 提取年-月
    T.country,
    1 AS approved_count,                       -- 每一行代表一笔批准交易，所以计数为 1
    T.amount AS approved_amount,               -- 批准交易的金额
    0 AS chargeback_count,                     -- 不计入退单计数
    0 AS chargeback_amount                     -- 不计入退单金额
FROM
    Transactions AS T
WHERE
    T.state = 'approved'
```

- **小贴士：** `DATE_FORMAT(T.trans_date, '%Y-%m')` 是 MySQL 语法。如果你的数据库是 PostgreSQL/Oracle，用 `TO_CHAR(T.trans_date, 'YYYY-MM')`；如果是 SQL Server，用 `FORMAT(T.trans_date, 'yyyy-MM')`。

------

#### 步骤 2: 准备“退单”的数据

接下来，我们处理退单。退单的日期是 `Chargebacks` 表的日期，但金额要从 `Transactions` 表中找。

SQL

```sql
SELECT
    DATE_FORMAT(C.trans_date, '%Y-%m') AS month, -- 退单的年-月
    T.country,                                 -- 通过连接获取国家
    0 AS approved_count,                       -- 退单不计入批准计数
    0 AS approved_amount,                      -- 退单不计入批准金额
    1 AS chargeback_count,                     -- 每一行代表一个退单，所以计数为 1
    T.amount AS chargeback_amount              -- 退单的金额是原始交易的金额
FROM
    Chargebacks AS C
JOIN
    Transactions AS T ON C.trans_id = T.id -- 通过 trans_id 连接原始交易表，获取国家和金额
```

------

#### 步骤 3: 合并两种数据流

现在我们有了两份独立的“事件”数据，它们都有 `month`, `country`, `approved_count`, `approved_amount`, `chargeback_count`, `chargeback_amount` 这些列。我们可以使用 `UNION ALL` 把它们合并起来。

SQL

```sql
WITH CombinedEvents AS (
    -- 批准的交易
    SELECT
        DATE_FORMAT(T.trans_date, '%Y-%m') AS month,
        T.country,
        1 AS approved_count,
        T.amount AS approved_amount,
        0 AS chargeback_count,
        0 AS chargeback_amount
    FROM
        Transactions AS T
    WHERE
        T.state = 'approved'

    UNION ALL -- 合并两组结果

    -- 退单
    SELECT
        DATE_FORMAT(C.trans_date, '%Y-%m') AS month,
        T.country,
        0 AS approved_count,
        0 AS approved_amount,
        1 AS chargeback_count,
        T.amount AS chargeback_amount
    FROM
        Chargebacks AS C
    JOIN
        Transactions AS T ON C.trans_id = T.id
)
-- 这一步只是合并，还没聚合
SELECT * FROM CombinedEvents;
```

**`CombinedEvents` 这个临时表里会是这样的数据（以示例数据为例）：**

| **month** | **country** | **approved_count** | **approved_amount** | **chargeback_count** | **chargeback_amount** |
| --------- | ----------- | ------------------ | ------------------- | -------------------- | --------------------- |
| 2019-05   | US          | 1                  | 1000                | 0                    | 0                     |
| 2019-06   | US          | 1                  | 3000                | 0                    | 0                     |
| 2019-06   | US          | 1                  | 5000                | 0                    | 0                     |
| 2019-05   | US          | 0                  | 0                   | 1                    | 2000                  |
| 2019-06   | US          | 0                  | 0                   | 1                    | 1000                  |
| 2019-09   | US          | 0                  | 0                   | 1                    | 5000                  |

------

#### 步骤 4: 分组聚合和过滤

现在有了 `CombinedEvents` 这个“事件流”，我们就可以按 `month` 和 `country` 进行分组，然后对相应的计数和金额进行求和。最后，根据题目要求过滤掉所有计数和金额都为零的行。

SQL

```mysql
WITH CombinedEvents AS (
    SELECT
        DATE_FORMAT(T.trans_date, '%Y-%m') AS month,
        T.country,
        1 AS approved_count,
        T.amount AS approved_amount,
        0 AS chargeback_count,
        0 AS chargeback_amount
    FROM
        Transactions AS T
    WHERE
        T.state = 'approved'

    UNION ALL

    SELECT
        DATE_FORMAT(C.trans_date, '%Y-%m') AS month,
        T.country,
        0 AS approved_count,
        0 AS approved_amount,
        1 AS chargeback_count,
        T.amount AS chargeback_amount
    FROM
        Chargebacks AS C
    JOIN
        Transactions AS T ON C.trans_id = T.id
)
SELECT
    month,
    country,
    SUM(approved_count) AS approved_count,
    SUM(approved_amount) AS approved_amount,
    SUM(chargeback_count) AS chargeback_count,
    SUM(chargeback_amount) AS chargeback_amount
FROM
    CombinedEvents
GROUP BY
    month,
    country
HAVING
    SUM(approved_count) > 0 OR         -- 如果批准计数大于 0
    SUM(approved_amount) > 0 OR        -- 或者批准金额大于 0
    SUM(chargeback_count) > 0 OR       -- 或者退单计数大于 0
    SUM(chargeback_amount) > 0         -- 或者退单金额大于 0
ORDER BY
    month, country; -- 排序让结果更清晰
```

------

### 总结

这种“分而治之，再合并聚合”的方法，通常在处理多个不同来源、不同日期维度但最终需要汇总到一起的统计问题时非常有效。它的优点是：

- **逻辑清晰：** 每一步只关注一个特定的事件类型。
- **易于调试：** 如果结果不对，你可以单独检查 `UNION ALL` 前的两个子查询，看看它们的数据是否正确。
- **可扩展性：** 如果未来需要添加更多事件类型（比如“退款”），你只需要再增加一个 `UNION ALL` 分支即可。



## 1308.不同性别每日分数总计

> 非常经典的“开窗函数”（窗口函数）入门题目，可借助该题目详细理解掌握开窗函数。

### Code

```mysql
SELECT 
    gender,
    day,
    SUM(score_points) OVER(PARTITION BY gender ORDER BY day) AS total
FROM Scores
ORDER BY gender,day
```

**简介**：传统聚合函数（如 `SUM()`、`COUNT()`、`AVG()` 等）在使用 `GROUP BY` 时，会将多行数据合并成一行，丢失了原始行的细节。而窗口函数则能让你在保留原始行结构的同时，完成复杂的聚合和分析。

**基本语法结构**：

```mysql
<窗口函数名> ( [参数列表] ) OVER (
    [ PARTITION BY <列名1>, <列名2>, ... ]
    [ ORDER BY <列名3> [ASC|DESC], <列名4> [ASC|DESC], ... ]
    [ <窗口框架> ]
)
```

### 窗口函数

窗口函数是 SQL 中一个非常强大的功能，它允许你在**与当前行相关的某个“窗口”内执行计算，而不会像 `GROUP BY` 那样将行折叠成单个输出行**。简单来说，它既能像聚合函数一样进行分组计算，又能保留原始数据的每一行，这使得它在数据分析中异常灵活。

你可以把它想象成一个“滑动窗口”：对于结果集中的每一行，这个窗口都会定义一个或多组相关的行，然后在这个窗口内执行计算。


在没有窗口函数之前，如果你想解决以下问题，可能需要复杂的子查询或自连接：

- 计算每一行在总销售额中的占比。
- 找出每个部门薪水最高的人。
- 计算某个员工过去 N 个月的平均销售额。
- 对数据进行排名。
- 比较当前行与前一行或后一行的数据。

传统聚合函数（如 `SUM()`、`COUNT()`、`AVG()` 等）在使用 `GROUP BY` 时，会将多行数据合并成一行，丢失了原始行的细节。而窗口函数则能让你在保留原始行结构的同时，完成复杂的聚合和分析。


### 窗口函数示例

假设我们有一个 `Employees` 表：

| **employee_id** | **department** | **salary** | **hire_date** |
| --------------- | -------------- | ---------- | ------------- |
| 1               | HR             | 60000      | 2020-01-01    |
| 2               | Sales          | 75000      | 2019-03-15    |
| 3               | HR             | 65000      | 2021-06-01    |
| 4               | Sales          | 75000      | 2018-09-01    |
| 5               | Marketing      | 80000      | 2022-02-10    |
| 6               | Marketing      | 80000      | 2022-02-10    |
| 7               | HR             | 58000      | 2020-05-20    |
| 8               | Sales          | 90000      | 2017-11-11    |

#### eg1: 计算各部门平均薪水

```mysql
SELECT
    employee_id,
    department,
    salary,
    -- 在每个部门内部计算平均薪水，保留所有行
    AVG(salary) OVER (PARTITION BY department) AS avg_department_salary
FROM
    Employees;
```

**结果：**

| **employee_id** | **department** | **salary** | **avg_department_salary** |
| --------------- | -------------- | ---------- | ------------------------- |
| 1               | HR             | 60000      | 61000.00                  |
| 3               | HR             | 65000      | 61000.00                  |
| 7               | HR             | 58000      | 61000.00                  |
| 5               | Marketing      | 80000      | 80000.00                  |
| 6               | Marketing      | 80000      | 80000.00                  |
| 2               | Sales          | 75000      | 80000.00                  |
| 4               | Sales          | 75000      | 80000.00                  |
| 8               | Sales          | 90000      | 80000.00                  |

**解释**：`PARTITION BY department` 将数据按部门分成三个分区（HR, Sales, Marketing）。`AVG(salary)` 会在每个分区内部计算平均值，并将这个平均值填充到该分区的所有行中。

#### eg2: 各部门内部按薪水排名

```MySQL
SELECT
    employee_id,
    department,
    salary,
    -- 排名函数：ROW_NUMBER, RANK, DENSE_RANK
    ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC) AS rn,
    RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS rk,
    DENSE_RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS drk
FROM
    Employees;
```

**结果（排序可能不同，取决于数据库）：**

| **employee_id** | **department** | **salary** | **rn** | **rk** | **drk** |
| --------------- | -------------- | ---------- | ------ | ------ | ------- |
| 3               | HR             | 65000      | 1      | 1      | 1       |
| 1               | HR             | 60000      | 2      | 2      | 2       |
| 7               | HR             | 58000      | 3      | 3      | 3       |
| 8               | Sales          | 90000      | 1      | 1      | 1       |
| 2               | Sales          | 75000      | 2      | 2      | 2       |
| 4               | Sales          | 75000      | 3      | 2      | 2       |
| 5               | Marketing      | 80000      | 1      | 1      | 1       |
| 6               | Marketing      | 80000      | 2      | 1      | 1       |

- 解释

  ：

  - `PARTITION BY department`：首先按部门分组。
  - `ORDER BY salary DESC`：在每个部门内部，按薪水降序排列。
  - **`ROW_NUMBER()`**：为每个部门内的每行分配一个唯一的连续数字。
  - **`RANK()`**：如果薪水相同，排名也相同，但下一个不同的薪水会跳过排名（例如 Sales 部门，75000 排名都是 2，但下一个 90000 变成 4）。
  - **`DENSE_RANK()`**：如果薪水相同，排名也相同，但下一个不同的薪水不会跳过排名（例如 Sales 部门，75000 排名都是 2，下一个 90000 变成 3）。

#### eg3: 使用 `LAG` 和 `LEAD` 比较前后行

```mysql
SELECT
    employee_id,
    department,
    salary,
    hire_date,
    -- 获取当前员工之前入职的员工的薪水 (按部门和入职日期排序)
    LAG(salary, 1, 0) OVER (PARTITION BY department ORDER BY hire_date ASC) AS prev_employee_salary,
    -- 获取当前员工之后入职的员工的薪水
    LEAD(salary, 1, 0) OVER (PARTITION BY department ORDER BY hire_date ASC) AS next_employee_salary
FROM
    Employees;
```

**结果（部分）：**

| **employee_id** | **department** | **salary** | **hire_date** | **prev_employee_salary** | **next_employee_salary** |
| --------------- | -------------- | ---------- | ------------- | ------------------------ | ------------------------ |
| 1               | HR             | 60000      | 2020-01-01    | 0                        | 58000                    |
| 7               | HR             | 58000      | 2020-05-20    | 60000                    | 65000                    |
| 3               | HR             | 65000      | 2021-06-01    | 58000                    | 0                        |
| ...             | ...            | ...        | ...           | ...                      | ...                      |

解释：

- `PARTITION BY department ORDER BY hire_date ASC`：在每个部门内部，按入职日期升序排列。
- `LAG(salary, 1, 0)`：获取前一行的 `salary` 值，如果前一行不存在（即当前行是分区的第一行），则返回 `0`。
- `LEAD(salary, 1, 0)`：获取后一行的 `salary` 值，如果后一行不存在（即当前行是分区的最后一行），则返回 `0`。

#### eg4: 使用窗口框架进行“滚动求和”

计算每个员工入职当月以及之前所有月份的销售额总和（假设有一个 `Sales` 表）。这里我们用 `Employees` 表的 `salary` 和 `hire_date` 演示“滚动求和”的概念。

```mysql
SELECT
    employee_id,
    department,
    salary,
    hire_date,
    -- 计算从分区开始到当前行（按入职日期排序）的累计薪水
    SUM(salary) OVER (PARTITION BY department ORDER BY hire_date ASC ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS running_total_salary
FROM
    Employees;
```

**结果（部分）：**

| **employee_id** | **department** | **salary** | **hire_date** | **running_total_salary** |
| --------------- | -------------- | ---------- | ------------- | ------------------------ |
| 1               | HR             | 60000      | 2020-01-01    | 60000                    |
| 7               | HR             | 58000      | 2020-05-20    | 118000                   |
| 3               | HR             | 65000      | 2021-06-01    | 183000                   |
| ...             | ...            | ...        | ...           | ...                      |

解释：

- `PARTITION BY department ORDER BY hire_date ASC`：按部门分区，在每个部门内按入职日期排序。
- `ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`：这个窗口框架告诉 `SUM()` 函数，对于每一行，只对其分区内从第一行到当前行的数据进行求和。这就实现了“累计求和”或“滚动求和”。

### 与 `GROUP BY` 的区别

这是理解窗口函数最关键的一点：

| **特性**     | **窗口函数**                                 | **GROUP BY 聚合函数**                             |
| ------------ | -------------------------------------------- | ------------------------------------------------- |
| **行数变化** | **不改变原始行数**，为每一行都返回一个结果。 | **减少行数**，将多行数据合并成一行。              |
| **数据粒度** | 可以在保持原始行粒度的同时，进行分组聚合。   | 只能在聚合后的粒度上进行分析，丢失原始细节。      |
| **`SELECT`** | 可以和非聚合列、其他普通列一起选择。         | `SELECT` 列表中只能包含 `GROUP BY` 列和聚合函数。 |
| **应用场景** | 排名、累计计算、前后行比较、分组内比例等。   | 总计、平均、最大/最小、计数等。                   |

------

### 总结

窗口函数是 SQL 数据分析的利器，它弥补了传统聚合函数在保留原始数据细节方面的不足。通过灵活运用 `PARTITION BY`、`ORDER BY` 和窗口框架，你可以完成非常复杂的分析任务，而无需编写冗长且低效的子查询。掌握窗口函数，能大大提升你的 SQL 查询能力。
