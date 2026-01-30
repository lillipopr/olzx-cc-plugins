# 索引优化

## 索引类型

### 1. 主键索引（PRIMARY）
```sql
PRIMARY KEY (id)
```
- 唯一标识
- 聚簇索引
- 自动创建

### 2. 唯一索引（UNIQUE）
```sql
UNIQUE KEY uk_user_id (user_id)
```
- 保证唯一性
- 可空字段允许多个 NULL

### 3. 普通索引（INDEX）
```sql
KEY idx_mobile (mobile)
```
- 加速查询
- 可重复

### 4. 联合索引
```sql
KEY idx_user_id_status (user_id, status)
```
- 多字段组合
- 遵循最左前缀

## 索引设计原则

### 1. 选择性优先
```
选择性 = distinct 值数量 / 总行数

高选择性（> 0.1）：
- user_id
- mobile
- order_id

低选择性（< 0.01）：
- gender
- status
- type
```

### 2. 覆盖索引
```sql
-- 表结构
CREATE TABLE user (
    id INT PRIMARY KEY,
    user_id VARCHAR(64),
    name VARCHAR(64),
    age INT,
    KEY idx_user_id_name (user_id, name)
);

-- 查询（覆盖索引）
SELECT name FROM user WHERE user_id = 'xxx';
-- 无需回表，性能最佳
```

### 3. 最左前缀原则
```sql
-- 联合索引 (a, b, c)

-- 可用索引
WHERE a = 1
WHERE a = 1 AND b = 2
WHERE a = 1 AND b = 2 AND c = 3

-- 不可用索引
WHERE b = 2
WHERE c = 3
WHERE b = 2 AND c = 3
```

## 索引失效场景

### 1. 隐式类型转换
```sql
-- 字段类型 VARCHAR
SELECT * FROM user WHERE mobile = 13800138000;  -- 失效
SELECT * FROM user WHERE mobile = '13800138000';  -- 有效
```

### 2. 函数计算
```sql
-- 失效
SELECT * FROM user WHERE YEAR(created_at) = 2024;

-- 优化
SELECT * FROM user WHERE created_at >= '2024-01-01' AND created_at < '2025-01-01';
```

### 3. LIKE 左模糊
```sql
-- 失效
SELECT * FROM user WHERE name LIKE '%张%';

-- 有效（右模糊）
SELECT * FROM user WHERE name LIKE '张%';
```

### 4. OR 条件
```sql
-- 失效（部分情况）
SELECT * FROM user WHERE user_id = 'xxx' OR age = 18;

-- 优化（UNION）
SELECT * FROM user WHERE user_id = 'xxx'
UNION
SELECT * FROM user WHERE age = 18;
```

### 5. 不等于
```sql
-- 失效
SELECT * FROM user WHERE status != 'ACTIVE';

-- 优化
SELECT * FROM user WHERE status IN ('PENDING', 'EXPIRED');
```

## 索引优化示例

### 慢查询优化
```sql
-- 原始查询（慢）
SELECT * FROM order WHERE user_id = 'xxx' AND status = 'PAID';

-- 添加索引
ALTER TABLE `order` ADD KEY idx_user_id_status (user_id, status);
```

### 覆盖索引优化
```sql
-- 原始查询（需回表）
SELECT id FROM user WHERE user_id = 'xxx';

-- 添加覆盖索引
ALTER TABLE user ADD KEY idx_user_id_id (user_id, id);
```

## 常见陷阱

### 1. 冗余索引
```sql
-- 冗余
KEY idx_a_b (a, b)
KEY idx_a (a)

-- 冗余
KEY idx_a (a)
PRIMARY KEY (a)
```

### 2. 过度索引
- 每个字段都加索引
- 写入性能下降
- 存储空间浪费

### 3. 忽略查询条件
- 实际查询条件与索引不匹配
- 索引无法使用
