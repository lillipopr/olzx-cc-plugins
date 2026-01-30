# 表结构设计方法论

## 核心原则

### 1. 三范式
- **1NF**：原子性，字段不可再分
- **2NF**：完全依赖，非主键字段完全依赖于主键
- **3NF**：传递依赖，消除传递依赖

### 2. 反范式
- 适度冗余提升性能
- 冗余字段同步更新
- 权衡一致性与性能

### 3. 字段规范

#### 主键设计
```sql
id VARCHAR(64) PRIMARY KEY COMMENT '全局唯一ID'
```
- 使用 VARCHAR(64) 存储全局唯一 ID
- 不使用自增 ID（分布式场景）

#### 软删除设计
```sql
deleted TINYINT NOT NULL DEFAULT 0 COMMENT '0=未删除 1=已删除'
deleted_at DATETIME(3) NULL COMMENT '删除时间'
```
- 双字段软删除
- 查询时带上 `deleted = 0`

#### 时间字段
```sql
created_at DATETIME(3) NOT NULL DEFAULT CURRENT_TIMESTAMP(3) COMMENT '创建时间'
updated_at DATETIME(3) NOT NULL DEFAULT CURRENT_TIMESTAMP(3) ON UPDATE CURRENT_TIMESTAMP(3) COMMENT '更新时间'
```
- 毫秒级精度 DATETIME(3)
- 自动更新 updated_at

### 4. 索引规范

#### 命名规范
```sql
-- 主键索引
PRIMARY KEY (id)

-- 唯一索引
UNIQUE KEY uk_user_id (user_id)

-- 普通索引
KEY idx_mobile (mobile)

-- 联合索引
KEY idx_user_id_status (user_id, status)
```

#### 索引设计原则
1. **选择性高的字段优先**：distinct 值多的字段
2. **覆盖索引**：索引包含查询所有字段
3. **最左前缀**：联合索引遵循最左前缀原则
4. **避免冗余**：(a,b) 和 (a) 冗余

### 5. 常见模式

#### 会员表
```sql
CREATE TABLE membership (
    id VARCHAR(64) PRIMARY KEY COMMENT '会员ID',
    user_id VARCHAR(64) NOT NULL COMMENT '用户ID',
    level VARCHAR(32) NOT NULL COMMENT '会员等级',
    status VARCHAR(32) NOT NULL COMMENT '状态',
    expire_at DATETIME(3) NOT NULL COMMENT '过期时间',
    deleted TINYINT NOT NULL DEFAULT 0 COMMENT '0=未删除 1=已删除',
    created_at DATETIME(3) NOT NULL DEFAULT CURRENT_TIMESTAMP(3) COMMENT '创建时间',
    updated_at DATETIME(3) NOT NULL DEFAULT CURRENT_TIMESTAMP(3) ON UPDATE CURRENT_TIMESTAMP(3) COMMENT '更新时间',
    deleted_at DATETIME(3) NULL COMMENT '删除时间',

    UNIQUE KEY uk_user_id (user_id),
    KEY idx_status_expire_at (status, expire_at)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='会员表';
```

#### 点券表
```sql
CREATE TABLE coupon (
    id VARCHAR(64) PRIMARY KEY COMMENT '点券ID',
    user_id VARCHAR(64) NOT NULL COMMENT '用户ID',
    amount INT NOT NULL COMMENT '点券数量',
    balance INT NOT NULL COMMENT '点券余额',
    deleted TINYINT NOT NULL DEFAULT 0 COMMENT '0=未删除 1=已删除',
    created_at DATETIME(3) NOT NULL DEFAULT CURRENT_TIMESTAMP(3) COMMENT '创建时间',
    updated_at DATETIME(3) NOT NULL DEFAULT CURRENT_TIMESTAMP(3) ON UPDATE CURRENT_TIMESTAMP(3) COMMENT '更新时间',
    deleted_at DATETIME(3) NULL COMMENT '删除时间',

    KEY idx_user_id (user_id),
    KEY idx_balance (balance)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='点券表';
```

## 常见陷阱

### 1. 过度反范式
- 冗余过多导致同步复杂
- 数据不一致风险

### 2. 索引过多
- 写入性能下降
- 存储空间浪费

### 3. 字段类型不当
- VARCHAR过长
- 精度不足
- 时区问题

### 4. 无软删除
- 数据无法恢复
- 审计追踪缺失
