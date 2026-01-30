# 分库分表策略

## 分表时机

### 判断标准
- **单表数据量**：> 1000 万行
- **单库数据量**：> 500GB
- **QPS**：> 单库承载上限（~5000）
- **TPS**：> 单库写入上限（~1000）

## 分表策略

### 1. 水平分表（Sharding）

#### 按时间分表
```sql
-- 按月分表
membership_202401
membership_202402
membership_202403

-- 适用场景
- 数据按时间查询
- 历史数据归档
- 日志类数据
```

#### 按 ID 取模分表
```sql
-- 按 user_id 取模
membership_0  -- user_id % 4 = 0
membership_1  -- user_id % 4 = 1
membership_2  -- user_id % 4 = 2
membership_3  -- user_id % 4 = 3

-- 适用场景
- 数据分布均匀
- 按 ID 查询为主
- 避免热点
```

#### 按范围分表
```sql
-- 按用户 ID 范围
membership_0   -- user_id: 0 ~ 999999
membership_1   -- user_id: 1000000 ~ 1999999
membership_2   -- user_id: 2000000 ~ 2999999

-- 适用场景
- ID 范围可预测
- 范围查询多
```

### 2. 垂直分表

#### 大字段分离
```sql
-- 主表
CREATE TABLE membership (
    id VARCHAR(64) PRIMARY KEY,
    user_id VARCHAR(64) NOT NULL,
    status VARCHAR(32) NOT NULL,
    -- ...
)

-- 扩展表
CREATE TABLE membership_ext (
    membership_id VARCHAR(64) PRIMARY KEY,
    description TEXT,
    metadata JSON,
    -- ...
)
```

### 3. 水平分库

#### 按业务分库
```
user_db        -- 用户库
order_db       -- 订单库
product_db     -- 商品库
payment_db     -- 支付库
```

#### 按地域分库
```
user_db_cn     -- 中国用户
user_db_us     -- 美国用户
user_db_eu     -- 欧洲用户
```

## 分片键选择

### 原则
1. **查询频率高**：常用查询条件
2. **数据分布均匀**：避免热点
3. **避免跨片**：减少跨库查询

### 示例
```sql
-- 好的分片键
user_id       -- 查询频率高、分布均匀
order_id      -- 唯一键

-- 差的分片键
gender        -- 数据倾斜严重
create_time   -- 范围查询需跨片
```

## 常见陷阱

### 1. 跨库查询
- 避免 JOIN 操作
- 使用应用层组装
- 冗余字段减少跨库

### 2. 数据倾斜
- 某些分片数据过多
- 热点问题
- 需要重新分片

### 3. 分片事务
- 分布式事务复杂
- 尽量避免跨片事务
- 使用最终一致性
