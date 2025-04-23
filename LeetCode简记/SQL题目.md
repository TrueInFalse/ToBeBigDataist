# SQL题目
---
## 1107.每日新用户统计
>新方法！DATE_SUB+CTE公共表表达式详解！
### 一、DATE_SUB()函数用法
当然！下面是关于 `DATE_SUB` 函数和 CTE（公共表表达式）在 SQL 中的详细讲解，适合你理解+背诵：

---

## 🗓️ 一、`DATE_SUB` 函数详解（MySQL）

### 1. 作用：
`DATE_SUB(date, INTERVAL expr unit)` 用于从某个日期中减去指定的时间间隔，常用于时间窗口的查询。

### 2. 基本语法：
```sql
SELECT DATE_SUB('2023-04-10', INTERVAL 7 DAY);
-- 结果：'2023-04-03'
```

### 3. 常用单位：
- `DAY` 天
- `MONTH` 月
- `YEAR` 年
- `HOUR` 小时
- `MINUTE` 分钟

### 4. 实战举例：
```sql
SELECT *
FROM orders
WHERE order_date >= DATE_SUB(CURDATE(), INTERVAL 30 DAY);
-- 查询近30天订单
```

---

## 📌 二、CTE（Common Table Expression）公共表表达式详解

### 1. 作用：
CTE 是一种临时的结果集，可以在主查询中多次引用，语法更清晰、结构更清楚，适用于递归查询、多层嵌套优化、统一复杂子查询。

### 2. 基本语法：
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

### 3. 应用场景举例：

#### 🌟 ① 结构清晰的多层查询
```sql
WITH recent_orders AS (
    SELECT * FROM orders
    WHERE order_date >= DATE_SUB(CURDATE(), INTERVAL 7 DAY)
)
SELECT customer_id, COUNT(*) as order_count
FROM recent_orders
GROUP BY customer_id;
```

#### 🌟 ② 递归查询（例如层级结构）
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

---

## ✅ 总结记忆版（方便背诵）：

> `DATE_SUB` 是 MySQL 中用于“减去时间”的函数，常搭配 `CURDATE()` 或 `NOW()` 用于近 X 天数据查询；CTE 是用 `WITH` 开头的公共表表达式，能临时抽取中间查询结果，使 SQL 更简洁，适用于复杂查询、递归查询或子查询优化场景。

### 二、CTE（公共表表达式）

### 三、Codes

```sql
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
