---
description: 如何重构实体类并更新所有引用
---

# 实体类重构 Workflow

本 workflow 描述如何安全地重构实体类，包括重命名、合并和移动。

## 场景 1: 重命名实体类

### 示例: 将 `TUser` 重命名为 `User`

#### 步骤 1: 规划重构

1. **确定影响范围**
```bash
# 搜索所有引用
grep -r "TUser" src/main/java --include="*.java" | wc -l
```

2. **检查相关类**
- Mapper: `TUserMapper`
- Service: 使用 `TUser` 的服务
- Controller: 使用 `TUser` 的控制器
- DTO/VO: 继承或引用 `TUser` 的类

#### 步骤 2: 创建新实体类

```java
// src/main/java/com/remostudio/user/entity/User.java
@Data
@TableName("auth_user")
public class User {
    @TableId(type = IdType.AUTO)
    private Integer id;
    
    private String awsSub;
    private String email;
    private String nickName;
    // ... 其他字段
}
```

#### 步骤 3: 更新 Mapper

```java
// 重命名 TUserMapper.java 为 UserMapper.java
@Mapper
public interface UserMapper extends BaseMapper<User> {
    // 保持方法不变
}
```

#### 步骤 4: 更新 Service

```java
@Service
public class UserService extends ServiceImpl<UserMapper, User> {
    // 替换所有 TUser 为 User
    public User findOne(Integer id) {
        return this.getById(id);
    }
}
```

#### 步骤 5: 批量更新引用

```bash
# 替换类名
find src/main/java -name "*.java" -type f -exec sed -i '' 's/\bTUser\b/User/g' {} \;

# 替换导入
find src/main/java -name "*.java" -type f -exec sed -i '' 's/import com.remostudio.user.model.entity.TUser/import com.remostudio.user.entity.User/g' {} \;
```

#### 步骤 6: 手动检查特殊情况

检查以下文件：
- 继承 `TUser` 的 VO 类
- 使用 `TUser::` 方法引用的代码
- XML mapper 文件中的类型引用

#### 步骤 7: 验证

```bash
mvn clean compile -DskipTests
```

## 场景 2: 合并实体类

### 示例: 合并 DDD `User` 和 MyBatis `TUser`

#### 步骤 1: 分析两个类

```bash
# 比较字段
diff <(grep "private" src/main/java/com/remostudio/user/domain/entity/User.java) \
     <(grep "private" src/main/java/com/remostudio/user/model/entity/TUser.java)
```

#### 步骤 2: 创建统一实体

```java
@Data
@TableName("auth_user")
public class User {
    // MyBatis-Plus 字段
    @TableId(type = IdType.AUTO)
    private Integer id;
    
    // DDD 字段
    private String awsSub;
    private String email;
    
    // 合并后的字段
    private String nickName;
    private String mobile;
    // ...
}
```

#### 步骤 3: 删除旧类

```bash
# 备份
git add .
git commit -m "backup: 合并实体类前备份"

# 删除 DDD 层
rm -rf src/main/java/com/remostudio/user/domain
rm -rf src/main/java/com/remostudio/user/application
rm -rf src/main/java/com/remostudio/user/infrastructure
```

#### 步骤 4: 更新所有引用

使用场景 1 的批量替换方法。

## 场景 3: 移动实体类到新包

### 示例: 将 `User` 从 `model.entity` 移动到 `entity`

#### 步骤 1: 创建新包

```bash
mkdir -p src/main/java/com/remostudio/user/entity
```

#### 步骤 2: 移动文件

```bash
mv src/main/java/com/remostudio/user/model/entity/User.java \
   src/main/java/com/remostudio/user/entity/User.java
```

#### 步骤 3: 更新包声明

```java
// 修改 User.java 第一行
package com.remostudio.user.entity;
```

#### 步骤 4: 更新所有导入

```bash
find src/main/java -name "*.java" -type f -exec sed -i '' \
  's/import com.remostudio.user.model.entity.User/import com.remostudio.user.entity.User/g' {} \;
```

## 最佳实践

### 1. 使用版本控制

每个步骤都提交：
```bash
git add .
git commit -m "refactor: 创建新 User 实体"
git commit -m "refactor: 更新 UserMapper"
git commit -m "refactor: 批量替换 TUser 引用"
```

### 2. 增量验证

每次修改后编译：
```bash
mvn compile -DskipTests
```

### 3. 保留相关实体

如果有关联实体（如 `TUserWork`），先不要重命名：
```java
// 保持不变
import com.remostudio.user.model.entity.TUserWork;
import com.remostudio.user.model.entity.TUserComment;
```

### 4. 处理字段名差异

如果字段名不同，创建兼容方法：
```java
@Deprecated
public void setState(Integer state) {
    this.setStatus(state);
}
```

## 常见问题

### 问题 1: 方法引用失效

**症状**: `User::getUserName` 报错

**原因**: 新实体使用 `getNickName()` 而不是 `getUserName()`

**解决**:
```java
// 添加兼容方法
public String getUserName() {
    return this.nickName;
}
```

### 问题 2: XML Mapper 引用错误

**症状**: MyBatis 报 `Type not found`

**解决**: 更新 XML 中的类型引用
```xml
<!-- 旧 -->
<resultMap type="com.remostudio.user.model.entity.TUser">

<!-- 新 -->
<resultMap type="com.remostudio.user.entity.User">
```

### 问题 3: 测试代码失败

**症状**: 测试编译失败

**解决**: 
1. 更新测试代码中的导入
2. 或者跳过测试: `mvn package -Dmaven.test.skip=true`

## 验证清单

- [ ] 新实体类创建完成
- [ ] Mapper 已更新
- [ ] Service 已更新
- [ ] Controller 已更新
- [ ] 所有导入已更新
- [ ] 编译成功
- [ ] 测试通过（或已跳过）
- [ ] 代码已提交

## 回滚方案

如果重构出现问题：

```bash
# 回滚到上一次提交
git reset --hard HEAD~1

# 或者回滚到特定提交
git log --oneline
git reset --hard <commit-hash>
```

## 相关工具

### IDE 重构工具

- **IntelliJ IDEA**: Refactor > Rename (Shift+F6)
- **Eclipse**: Refactor > Rename
- **VS Code**: F2 (需要 Java 扩展)

### 命令行工具

```bash
# 搜索
grep -r "ClassName" src/main/java

# 替换
sed -i '' 's/OldClass/NewClass/g' file.java

# 批量查找
find . -name "*.java" -exec grep "ClassName" {} \;
```
