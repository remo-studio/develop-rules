# Java MCP 共享库发布方案

> 适用场景：将 MCP 公共能力（认证、签名、模型定义、工具注册等）抽取为独立 Java 库，供团队多个项目复用。
>
> 核心问题：发布到 **AWS CodeArtifact** 还是 **GitHub Packages**？

---

## 1. 可行性结论

**完全可行**。两种方案均支持私有 Maven 仓库，可供多个 Java/Spring Boot 项目通过标准 `<dependency>` 引入。

| 对比项 | AWS CodeArtifact | GitHub Packages |
|--------|-----------------|----------------|
| 适用场景 | 已重度使用 AWS 的团队 | 代码已托管在 GitHub 的团队 |
| 认证方式 | AWS IAM / OIDC | GitHub PAT / GITHUB_TOKEN |
| CI/CD 集成 | CodeBuild / GitHub Actions via OIDC | GitHub Actions 原生支持 |
| 私有性 | 账号内私有，精细 IAM 控制 | 仓库/组织级私有 |
| 费用 | 存储+请求费用（有免费额度） | 免费额度内免费 |
| 推荐指数 | ⭐⭐⭐（AWS 深度用户首选） | ⭐⭐⭐⭐（GitHub 团队首选） |

> **推荐**：团队代码已托管在 GitHub，优先选择 **GitHub Packages**，流程更简单，CI/CD 零额外配置。
> 若团队有 AWS 统一制品管理要求，选择 **AWS CodeArtifact**。

---

## 2. 库结构建议（good7ob-mcp-java）

```
good7ob-mcp-java/
├── pom.xml                          # 父 POM，groupId: com.good7ob, artifactId: good7ob-mcp
├── good7ob-mcp-core/                # 核心模型与接口
│   └── src/main/java/com/good7ob/mcp/
│       ├── model/
│       │   ├── McpToolDefinition.java
│       │   ├── McpToolCallRequest.java
│       │   ├── McpToolCallResponse.java
│       │   └── McpContent.java
│       └── spi/
│           └── McpToolHandler.java  # 工具实现接口（SPI）
├── good7ob-mcp-security/            # 认证与签名
│   └── src/main/java/com/good7ob/mcp/security/
│       ├── McpSignatureUtil.java    # HMAC-SHA256 签名工具
│       ├── McpAuthFilter.java       # Spring 过滤器
│       └── McpSecurityProperties.java
├── good7ob-mcp-spring-boot-starter/ # Spring Boot 自动装配
│   └── src/main/java/com/good7ob/mcp/autoconfigure/
│       └── McpAutoConfiguration.java
└── README.md
```

### 2.1 版本号规范

遵循语义化版本 `MAJOR.MINOR.PATCH`：

```xml
<groupId>com.good7ob</groupId>
<artifactId>good7ob-mcp-spring-boot-starter</artifactId>
<version>1.0.0</version>
```

---

## 3. 方案 A：发布到 GitHub Packages

### 3.1 库项目 `pom.xml` 配置

```xml
<distributionManagement>
  <repository>
    <id>github</id>
    <name>GitHub good7ob Apache Maven Packages</name>
    <url>https://maven.pkg.github.com/good7ob/good7ob-mcp-java</url>
  </repository>
</distributionManagement>
```

### 3.2 GitHub Actions 发布流程（`.github/workflows/publish.yml`）

```yaml
name: Publish to GitHub Packages

on:
  release:
    types: [created]

jobs:
  publish:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with:
          java-version: '17'
          distribution: 'temurin'
          cache: maven
          server-id: github
          server-username: GITHUB_ACTOR
          server-password: GITHUB_TOKEN
      - name: Publish package
        run: mvn --batch-mode deploy
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

### 3.3 消费方项目配置

**`~/.m2/settings.xml`（本地开发）**

```xml
<settings>
  <servers>
    <server>
      <id>github</id>
      <username>YOUR_GITHUB_USERNAME</username>
      <password>YOUR_PAT_WITH_READ_PACKAGES_SCOPE</password>
    </server>
  </servers>
</settings>
```

**`pom.xml`（引入依赖）**

```xml
<repositories>
  <repository>
    <id>github</id>
    <url>https://maven.pkg.github.com/good7ob/good7ob-mcp-java</url>
    <snapshots><enabled>true</enabled></snapshots>
  </repository>
</repositories>

<dependencies>
  <dependency>
    <groupId>com.good7ob</groupId>
    <artifactId>good7ob-mcp-spring-boot-starter</artifactId>
    <version>1.0.0</version>
  </dependency>
</dependencies>
```

**GitHub Actions 消费方（无需 PAT，用 GITHUB_TOKEN 即可，同 org/user 内）**

```yaml
- uses: actions/setup-java@v4
  with:
    java-version: '17'
    distribution: 'temurin'
    server-id: github
    server-username: GITHUB_ACTOR
    server-password: GITHUB_TOKEN
```

---

## 4. 方案 B：发布到 AWS CodeArtifact

### 4.1 创建 CodeArtifact 域和仓库

```bash
# 创建域（整个组织共享一个域）
aws codeartifact create-domain --domain good7ob

# 创建仓库
aws codeartifact create-repository \
  --domain good7ob \
  --domain-owner <AWS_ACCOUNT_ID> \
  --repository good7ob-mcp \
  --description "good7ob MCP shared library"

# 关联上游 Maven Central（可选，用于传递依赖解析）
aws codeartifact associate-external-connection \
  --domain good7ob \
  --repository good7ob-mcp \
  --external-connection public:maven-central
```

### 4.2 获取临时 Token 并配置 `settings.xml`

```bash
export CODEARTIFACT_TOKEN=$(aws codeartifact get-authorization-token \
  --domain good7ob \
  --domain-owner <AWS_ACCOUNT_ID> \
  --query authorizationToken --output text)

export CODEARTIFACT_URL=$(aws codeartifact get-repository-endpoint \
  --domain good7ob \
  --domain-owner <AWS_ACCOUNT_ID> \
  --repository good7ob-mcp \
  --format maven \
  --query repositoryEndpoint --output text)
```

**`~/.m2/settings.xml`**

```xml
<settings>
  <servers>
    <server>
      <id>codeartifact</id>
      <username>aws</username>
      <password>${env.CODEARTIFACT_TOKEN}</password>
    </server>
  </servers>
  <mirrors>
    <mirror>
      <id>codeartifact</id>
      <mirrorOf>*</mirrorOf>
      <url>${env.CODEARTIFACT_URL}</url>
    </mirror>
  </mirrors>
</settings>
```

### 4.3 GitHub Actions 发布流程（OIDC 方式，无需硬编码密钥）

```yaml
name: Publish to AWS CodeArtifact

on:
  release:
    types: [created]

permissions:
  id-token: write   # 用于 OIDC
  contents: read

jobs:
  publish:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::<ACCOUNT_ID>:role/github-actions-codeartifact
          aws-region: ap-northeast-1
      - name: Get CodeArtifact Token
        run: |
          CODEARTIFACT_TOKEN=$(aws codeartifact get-authorization-token \
            --domain good7ob --domain-owner <ACCOUNT_ID> \
            --query authorizationToken --output text)
          echo "CODEARTIFACT_TOKEN=$CODEARTIFACT_TOKEN" >> $GITHUB_ENV
      - uses: actions/setup-java@v4
        with:
          java-version: '17'
          distribution: 'temurin'
          cache: maven
      - name: Publish
        run: mvn --batch-mode deploy -s .mvn/settings.xml
```

### 4.4 `pom.xml` 发布配置

```xml
<distributionManagement>
  <repository>
    <id>codeartifact</id>
    <name>good7ob--good7ob-mcp</name>
    <url>https://good7ob-<ACCOUNT_ID>.d.codeartifact.ap-northeast-1.amazonaws.com/maven/good7ob-mcp/</url>
  </repository>
</distributionManagement>
```

---

## 5. 消费方快速接入（以 Spring Boot 项目为例）

无论使用哪种发布方案，消费方接入步骤相同：

### 5.1 引入依赖

```xml
<dependency>
  <groupId>com.good7ob</groupId>
  <artifactId>good7ob-mcp-spring-boot-starter</artifactId>
  <version>1.0.0</version>
</dependency>
```

### 5.2 `application.yml` 配置

```yaml
good7ob:
  mcp:
    security:
      enabled: true
      signature-enabled: true
      signature-expire-seconds: 300
      api-keys:
        - key-id: "client-a"
          key-secret: "${MCP_SECRET_KEY_A}"
          active: true
    rate-limit:
      enabled: true
      per-key-rps: 10
```

### 5.3 实现自定义工具

```java
@Component
public class StationStatusTool implements McpToolHandler {

    @Override
    public McpToolDefinition definition() {
        return McpToolDefinition.builder()
            .name("get_station_status")
            .description("获取站点在线状态统计")
            .inputSchema(InputSchema.empty())
            .build();
    }

    @Override
    public McpToolCallResponse call(Map<String, Object> arguments) {
        // 业务逻辑
        return McpToolCallResponse.text("## 站点状态\n- 在线: 3096");
    }
}
```

Spring Boot Starter 会自动扫描所有 `McpToolHandler` 实现并注册到 `/mcp/tools/call`。

---

## 6. 选型决策树

```
是否已重度使用 AWS？
├── 是 → 是否有统一 IAM 权限管理需求？
│         ├── 是 → ✅ AWS CodeArtifact
│         └── 否 → GitHub Packages 也可（更简单）
└── 否 → 代码是否托管在 GitHub？
          ├── 是 → ✅ GitHub Packages（推荐）
          └── 否 → 考虑自建 Nexus / JFrog Artifactory
```

---

## 7. 注意事项

1. **语义化版本**：稳定版用 `1.x.x`，开发中用 `1.x.x-SNAPSHOT`，SNAPSHOT 在 GitHub Packages 需显式启用。
2. **依赖传递**：库的 `pom.xml` 尽量将 Spring Boot 相关依赖设置为 `<scope>provided</scope>`，避免版本冲突。
3. **Breaking Change**：修改 `McpToolHandler` 接口等公共 API 时，遵循 Major 版本升级。
4. **密钥安全**：所有 `key-secret` 通过环境变量或 AWS Secrets Manager / GitHub Secrets 注入，禁止硬编码。
5. **Token 有效期（CodeArtifact）**：CodeArtifact Token 默认 12 小时有效，CI/CD 每次构建重新获取。

---

## 8. 参考资源

- [GitHub Packages - Maven 官方文档](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-apache-maven-registry)
- [AWS CodeArtifact - Maven 官方文档](https://docs.aws.amazon.com/codeartifact/latest/ug/maven-mvn.html)
- [AWS CodeArtifact + GitHub Actions OIDC](https://docs.aws.amazon.com/codeartifact/latest/ug/getting-started-github-actions.html)
- [MCP 通用设计方案](MCP通用设计方案.md)

---

**最后更新**: 2026-02-22
