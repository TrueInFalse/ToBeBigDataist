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
