# 数据库表设计说明文档

> 本文档用于记录数据库表结构设计、字段说明及相关约束信息

---

## 1. 数据库信息

| 项目 | 内容 |
|------|------|
| **数据库名称** | database_name |
| **数据库引擎** | MySQL 8.0 / PostgreSQL 14 / MongoDB 5.0 |
| **字符集** | utf8mb4 |
| **排序规则** | utf8mb4_unicode_ci |
| **所属系统** | 系统名称 |
| **维护人员** | 姓名/团队 |
| **创建日期** | YYYY-MM-DD |
| **最后更新** | YYYY-MM-DD |

---

## 2. 表清单

| 序号 | 表名 | 中文名称 | 说明 | 状态 |
|------|------|----------|------|------|
| 1 | user | 用户表 | 存储系统用户基本信息 | 已实现 |
| 2 | role | 角色表 | 存储用户角色信息 | 已实现 |
| 3 | permission | 权限表 | 存储系统权限信息 | 设计中 |

---

## 3. 表详细设计

### 3.1 表名：`user`（用户表）

#### 3.1.1 表基本信息

| 项目 | 内容 |
|------|------|
| **表名** | user |
| **中文名称** | 用户表 |
| **存储引擎** | InnoDB |
| **字符集** | utf8mb4 |
| **业务说明** | 存储系统用户的基本信息，包括账号、密码、个人资料等 |
| **数据量级** | 预估10万+ |
| **增长速率** | 约1000条/月 |
| **访问频率** | 高频 |

#### 3.1.2 字段说明

| 序号 | 字段名 | 数据类型 | 长度 | 主键 | 非空 | 默认值 | 自增 | 说明 | 备注 |
|------|--------|----------|------|------|------|--------|------|------|------|
| 1 | id | BIGINT | 20 | ✓ | ✓ | - | ✓ | 用户ID | 主键 |
| 2 | username | VARCHAR | 50 | | ✓ | - | | 用户名 | 唯一，用于登录 |
| 3 | password | VARCHAR | 255 | | ✓ | - | | 密码 | 加密存储 |
| 4 | email | VARCHAR | 100 | | | NULL | | 邮箱地址 | 唯一 |
| 5 | phone | VARCHAR | 20 | | | NULL | | 手机号 | 唯一 |
| 6 | nickname | VARCHAR | 50 | | | NULL | | 昵称 | 显示名称 |
| 7 | avatar | VARCHAR | 255 | | | NULL | | 头像URL | 用户头像图片地址 |
| 8 | gender | TINYINT | 1 | | | 0 | | 性别 | 0-未知 1-男 2-女 |
| 9 | birthday | DATE | - | | | NULL | | 生日 | - |
| 10 | status | TINYINT | 1 | | ✓ | 1 | | 账号状态 | 0-禁用 1-正常 2-锁定 |
| 11 | last_login_time | DATETIME | - | | | NULL | | 最后登录时间 | - |
| 12 | last_login_ip | VARCHAR | 50 | | | NULL | | 最后登录IP | - |
| 13 | created_at | DATETIME | - | | ✓ | CURRENT_TIMESTAMP | | 创建时间 | - |
| 14 | updated_at | DATETIME | - | | ✓ | CURRENT_TIMESTAMP ON UPDATE | | 更新时间 | 自动更新 |
| 15 | deleted_at | DATETIME | - | | | NULL | | 删除时间 | 软删除标记 |

#### 3.1.3 索引设计

| 索引名 | 索引类型 | 字段 | 说明 | 备注 |
|--------|----------|------|------|------|
| PRIMARY | PRIMARY KEY | id | 主键索引 | 自动创建 |
| uk_username | UNIQUE | username | 用户名唯一索引 | 保证用户名唯一 |
| uk_email | UNIQUE | email | 邮箱唯一索引 | 允许NULL |
| uk_phone | UNIQUE | phone | 手机号唯一索引 | 允许NULL |
| idx_status | NORMAL | status | 状态普通索引 | 用于状态查询 |
| idx_created_at | NORMAL | created_at | 创建时间索引 | 用于时间范围查询 |
| idx_deleted_at | NORMAL | deleted_at | 删除时间索引 | 用于软删除查询 |

#### 3.1.4 约束说明

| 约束类型 | 约束名 | 字段 | 规则 | 说明 |
|----------|--------|------|------|------|
| CHECK | chk_gender | gender | gender IN (0,1,2) | 性别只能是0/1/2 |
| CHECK | chk_status | status | status IN (0,1,2) | 状态只能是0/1/2 |
| FOREIGN KEY | fk_user_role | role_id | REFERENCES role(id) | 关联角色表 |

#### 3.1.5 触发器

| 触发器名 | 触发时机 | 事件类型 | 说明 |
|----------|----------|----------|------|
| trg_user_before_insert | BEFORE | INSERT | 插入前验证数据格式 |
| trg_user_after_update | AFTER | UPDATE | 更新后记录操作日志 |

#### 3.1.6 关联关系

```
user (1) ----< (*) user_role (*) >---- (1) role
  |
  |----< (1) user_profile (1:1 关系)
  |
  |----< (*) user_login_log (1:N 关系)
```

**说明：**
- 一个用户可以拥有多个角色（多对多关系，通过 user_role 中间表关联）
- 一个用户对应一个用户档案（一对一关系）
- 一个用户可以有多条登录日志（一对多关系）

#### 3.1.7 业务规则

1. **用户名规则**
   - 长度：3-50个字符
   - 格式：字母、数字、下划线组合
   - 不可重复，注册后不可修改

2. **密码规则**
   - 存储方式：使用 bcrypt 加密算法
   - 强度要求：至少8位，包含大小写字母、数字、特殊字符
   - 过期策略：90天强制修改

3. **账号状态**
   - 新注册用户默认为正常状态（1）
   - 连续登录失败5次自动锁定（2）
   - 管理员可手动禁用账号（0）

4. **软删除**
   - 使用 deleted_at 字段标记删除
   - 删除后的账号用户名、邮箱、手机号可以被重新注册
   - 删除后30天可恢复，超过30天后物理删除

#### 3.1.8 示例数据

```sql
INSERT INTO `user` (username, password, email, phone, nickname, gender, status) VALUES
('admin', '$2y$10$...', 'admin@example.com', '13800138000', '管理员', 1, 1),
('user001', '$2y$10$...', 'user001@example.com', '13800138001', '张三', 1, 1),
('user002', '$2y$10$...', 'user002@example.com', '13800138002', '李四', 2, 1);
```

#### 3.1.9 DDL语句

```sql
CREATE TABLE `user` (
  `id` BIGINT(20) NOT NULL AUTO_INCREMENT COMMENT '用户ID',
  `username` VARCHAR(50) NOT NULL COMMENT '用户名',
  `password` VARCHAR(255) NOT NULL COMMENT '密码',
  `email` VARCHAR(100) DEFAULT NULL COMMENT '邮箱地址',
  `phone` VARCHAR(20) DEFAULT NULL COMMENT '手机号',
  `nickname` VARCHAR(50) DEFAULT NULL COMMENT '昵称',
  `avatar` VARCHAR(255) DEFAULT NULL COMMENT '头像URL',
  `gender` TINYINT(1) DEFAULT 0 COMMENT '性别：0-未知 1-男 2-女',
  `birthday` DATE DEFAULT NULL COMMENT '生日',
  `status` TINYINT(1) NOT NULL DEFAULT 1 COMMENT '状态：0-禁用 1-正常 2-锁定',
  `last_login_time` DATETIME DEFAULT NULL COMMENT '最后登录时间',
  `last_login_ip` VARCHAR(50) DEFAULT NULL COMMENT '最后登录IP',
  `created_at` DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `updated_at` DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  `deleted_at` DATETIME DEFAULT NULL COMMENT '删除时间',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_username` (`username`),
  UNIQUE KEY `uk_email` (`email`),
  UNIQUE KEY `uk_phone` (`phone`),
  KEY `idx_status` (`status`),
  KEY `idx_created_at` (`created_at`),
  KEY `idx_deleted_at` (`deleted_at`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='用户表';
```

#### 3.1.10 性能优化建议

1. **索引优化**
   - 根据实际查询场景添加复合索引
   - 定期分析慢查询日志，优化索引策略

2. **分表策略**
   - 当数据量超过1000万时，考虑按 id 进行水平分表
   - 分表规则：id % 10，分为10个表

3. **缓存策略**
   - 用户基本信息使用 Redis 缓存
   - 缓存过期时间：30分钟
   - 缓存更新策略：更新数据库后同步更新缓存

4. **归档策略**
   - 软删除超过90天的数据移至历史表
   - 历史表定期备份后可清理

---

### 3.2 表名：`role`（角色表）

#### 3.2.1 表基本信息

| 项目 | 内容 |
|------|------|
| **表名** | role |
| **中文名称** | 角色表 |
| **存储引擎** | InnoDB |
| **字符集** | utf8mb4 |
| **业务说明** | 存储系统角色信息，用于权限管理 |

#### 3.2.2 字段说明

| 序号 | 字段名 | 数据类型 | 长度 | 主键 | 非空 | 默认值 | 说明 |
|------|--------|----------|------|------|------|--------|------|
| 1 | id | BIGINT | 20 | ✓ | ✓ | - | 角色ID |
| 2 | role_name | VARCHAR | 50 | | ✓ | - | 角色名称 |
| 3 | role_code | VARCHAR | 50 | | ✓ | - | 角色编码 |
| 4 | description | VARCHAR | 255 | | | NULL | 角色描述 |
| 5 | sort | INT | 11 | | | 0 | 排序序号 |
| 6 | status | TINYINT | 1 | | ✓ | 1 | 状态 |
| 7 | created_at | DATETIME | - | | ✓ | CURRENT_TIMESTAMP | 创建时间 |
| 8 | updated_at | DATETIME | - | | ✓ | CURRENT_TIMESTAMP | 更新时间 |

**说明：** 其他字段信息参照用户表格式补充

---

## 4. 数据字典

### 4.1 通用字段说明

| 字段名 | 数据类型 | 说明 | 使用场景 |
|--------|----------|------|----------|
| id | BIGINT | 主键ID | 所有业务表 |
| created_at | DATETIME | 创建时间 | 所有业务表 |
| updated_at | DATETIME | 更新时间 | 所有业务表 |
| deleted_at | DATETIME | 删除时间 | 需要软删除的表 |
| created_by | BIGINT | 创建人ID | 需要追踪创建人的表 |
| updated_by | BIGINT | 更新人ID | 需要追踪修改人的表 |

### 4.2 枚举值字典

#### 4.2.1 用户状态（status）

| 值 | 名称 | 说明 |
|----|------|------|
| 0 | 禁用 | 账号被管理员禁用，无法登录 |
| 1 | 正常 | 账号正常可用 |
| 2 | 锁定 | 账号被系统锁定（如密码错误次数过多） |

#### 4.2.2 性别（gender）

| 值 | 名称 | 说明 |
|----|------|------|
| 0 | 未知 | 未设置或不愿透露 |
| 1 | 男 | 男性 |
| 2 | 女 | 女性 |

---

## 5. ER图

```
┌─────────────┐         ┌──────────────┐         ┌─────────────┐
│    user     │         │  user_role   │         │    role     │
├─────────────┤         ├──────────────┤         ├─────────────┤
│ id (PK)     │────────<│ user_id (FK) │         │ id (PK)     │
│ username    │         │ role_id (FK) │>────────│ role_name   │
│ password    │         │ created_at   │         │ role_code   │
│ email       │         └──────────────┘         │ status      │
│ status      │                                  └─────────────┘
│ created_at  │
└─────────────┘
      │
      │ 1:1
      │
      ▼
┌──────────────┐
│ user_profile │
├──────────────┤
│ id (PK)      │
│ user_id (FK) │
│ address      │
│ company      │
└──────────────┘
```

---

## 6. 变更历史

### 6.1 版本记录

| 版本号 | 变更日期 | 变更人 | 变更类型 | 变更说明 |
|--------|----------|--------|----------|----------|
| v1.0.0 | 2024-01-01 | 张三 | 新建 | 初始版本，创建用户表、角色表 |
| v1.1.0 | 2024-02-15 | 李四 | 修改 | 用户表新增 last_login_time 字段 |
| v1.1.1 | 2024-03-20 | 王五 | 优化 | 优化用户表索引，新增复合索引 |
| v1.2.0 | 2024-04-10 | 赵六 | 新建 | 新增权限表 permission |

### 6.2 详细变更记录

#### v1.1.0 (2024-02-15)

**变更类型：** 字段新增

**影响表：** user

**变更内容：**
```sql
ALTER TABLE `user` ADD COLUMN `last_login_time` DATETIME DEFAULT NULL COMMENT '最后登录时间' AFTER `status`;
ALTER TABLE `user` ADD COLUMN `last_login_ip` VARCHAR(50) DEFAULT NULL COMMENT '最后登录IP' AFTER `last_login_time`;
```

**变更原因：** 业务需求，需要记录用户最后登录时间和IP地址

**影响范围：** 无影响，新增字段允许为空

**回滚方案：**
```sql
ALTER TABLE `user` DROP COLUMN `last_login_time`;
ALTER TABLE `user` DROP COLUMN `last_login_ip`;
```

---

## 7. 规范说明

### 7.1 命名规范

1. **表命名规范**
   - 使用小写字母
   - 单词间使用下划线分隔
   - 使用有意义的英文单词
   - 禁止使用数据库关键字
   - 示例：`user_login_log`

2. **字段命名规范**
   - 使用小写字母
   - 单词间使用下划线分隔
   - 见名知意，避免缩写
   - 布尔类型以 is_ 开头
   - 示例：`is_deleted`, `created_at`

3. **索引命名规范**
   - 普通索引：`idx_字段名`
   - 唯一索引：`uk_字段名`
   - 组合索引：`idx_字段1_字段2`
   - 外键索引：`fk_表名_字段名`

### 7.2 设计规范

1. **字段设计**
   - 尽量使用 NOT NULL + DEFAULT 代替 NULL
   - 禁止使用 TEXT、BLOB 存储大字段，建议使用对象存储
   - 金额字段使用 DECIMAL 类型
   - 时间字段统一使用 DATETIME 类型

2. **索引设计**
   - 单表索引数量不超过5个
   - 单个索引字段数不超过5个
   - 字符串字段索引长度不超过20个字符
   - 区分度低的字段不建索引

3. **性能要求**
   - 单表数据量不超过1000万
   - 单条SQL执行时间不超过1秒
   - 避免使用 SELECT *
   - 避免在 WHERE 子句中使用函数

### 7.3 安全规范

1. **数据安全**
   - 敏感信息必须加密存储（密码、身份证号等）
   - 重要操作需记录操作日志
   - 定期备份数据库

2. **权限控制**
   - 应用程序使用独立数据库账号
   - 最小权限原则，只授予必要的权限
   - 禁止使用 root 账号连接数据库

---

## 8. 附录

### 8.1 常用SQL示例

#### 8.1.1 查询活跃用户
```sql
SELECT id, username, nickname, last_login_time
FROM user
WHERE status = 1 
  AND deleted_at IS NULL
  AND last_login_time > DATE_SUB(NOW(), INTERVAL 30 DAY)
ORDER BY last_login_time DESC;
```

#### 8.1.2 统计用户注册趋势
```sql
SELECT DATE_FORMAT(created_at, '%Y-%m') AS month, COUNT(*) AS user_count
FROM user
WHERE deleted_at IS NULL
GROUP BY DATE_FORMAT(created_at, '%Y-%m')
ORDER BY month DESC;
```

#### 8.1.3 查询用户角色信息
```sql
SELECT u.id, u.username, GROUP_CONCAT(r.role_name) AS roles
FROM user u
LEFT JOIN user_role ur ON u.id = ur.user_id
LEFT JOIN role r ON ur.role_id = r.id
WHERE u.deleted_at IS NULL
GROUP BY u.id, u.username;
```

### 8.2 维护脚本

#### 8.2.1 清理过期的软删除数据
```sql
-- 删除90天前的软删除记录
DELETE FROM user 
WHERE deleted_at IS NOT NULL 
  AND deleted_at < DATE_SUB(NOW(), INTERVAL 90 DAY);
```

#### 8.2.2 重建索引
```sql
-- 分析表
ANALYZE TABLE user;

-- 优化表
OPTIMIZE TABLE user;
```

### 8.3 参考资料

- [MySQL 8.0 官方文档](https://dev.mysql.com/doc/)
- [阿里巴巴Java开发手册-数据库规约](https://developer.aliyun.com/)
- [数据库设计最佳实践](https://example.com)

---

## 9. 联系方式

| 角色 | 姓名 | 邮箱 | 备注 |
|------|------|------|------|
| 数据库管理员 | 张三 | zhangsan@example.com | 负责数据库架构设计 |
| 开发负责人 | 李四 | lisi@example.com | 负责应用层开发 |
| 运维负责人 | 王五 | wangwu@example.com | 负责数据库运维 |

---

**文档版本：** v1.0.0  
**最后更新：** 2024-01-01  
**维护人员：** 数据库团队

