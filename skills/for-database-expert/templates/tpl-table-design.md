# 表设计模板

## 基本信息
- 表名：`{table_name}`
- 说明：{表描述}
- 字符集：utf8mb4
- 引擎：InnoDB

## 字段定义

| 字段名 | 类型 | 空 | 默认值 | 说明 |
|-------|------|------|--------|------|
| id | VARCHAR(64) | NOT NULL | - | 全局唯一ID |
| {field_name} | {type} | {null} | {default} | {comment} |
| deleted | TINYINT | NOT NULL | 0 | 0=未删除 1=已删除 |
| created_at | DATETIME(3) | NOT NULL | CURRENT_TIMESTAMP(3) | 创建时间 |
| updated_at | DATETIME(3) | NOT NULL | CURRENT_TIMESTAMP(3) | 更新时间 |
| deleted_at | DATETIME(3) | NULL | NULL | 删除时间 |

## 索引定义

| 索引名 | 类型 | 字段 | 说明 |
|-------|------|------|------|
| PRIMARY | PRIMARY | id | 主键 |
| uk_{field} | UNIQUE | {field} | 唯一索引 |
| idx_{field} | INDEX | {field} | 普通索引 |

## 约束定义
- 主键：id
- 唯一键：{unique_keys}
- 外键：{无（应用层维护）}

## 建表语句
```sql
CREATE TABLE {table_name} (
    id VARCHAR(64) PRIMARY KEY COMMENT '全局唯一ID',
    {field_definitions}

    deleted TINYINT NOT NULL DEFAULT 0 COMMENT '0=未删除 1=已删除',
    created_at DATETIME(3) NOT NULL DEFAULT CURRENT_TIMESTAMP(3) COMMENT '创建时间',
    updated_at DATETIME(3) NOT NULL DEFAULT CURRENT_TIMESTAMP(3) ON UPDATE CURRENT_TIMESTAMP(3) COMMENT '更新时间',
    deleted_at DATETIME(3) NULL COMMENT '删除时间',

    {indexes}
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='{table_comment}';
```

## 设计要点
- [ ] 符合三范式
- [ ] 字段类型合理
- [ ] 索引设计合理
- [ ] 软删除设计
- [ ] 时间字段精度
