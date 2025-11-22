# 日志配置模板

## 📋 说明

本目录用于存放日志配置相关的模板和指南。

## 📍 模板位置

日志配置模板实际位于 `mocha-documents/logging/` 目录，包含以下模板：

### 1. 多服务 Datadog 分布式追踪配置模板

**文件**: [`../../mocha-documents/logging/01-多服务-datadog-分布式追踪配置模板.md`](../../mocha-documents/logging/01-多服务-datadog-分布式追踪配置模板.md)

**适用场景**:
- 多服务微服务架构
- 需要分布式追踪
- 使用 Datadog 进行日志管理

**核心特性**:
- ✅ 通过 `span_id` 和 `parent_span_id` 实现跨服务追踪
- ✅ JSON 格式日志输出
- ✅ 完整的链路追踪工具类
- ✅ HTTP 拦截器和 Feign 客户端配置

---

### 2. 单服务日志配置模板

**文件**: [`../../mocha-documents/logging/02-单服务日志配置模板.md`](../../mocha-documents/logging/02-单服务日志配置模板.md)

**适用场景**:
- 单服务应用
- 简单的日志管理需求
- 不需要分布式追踪

**核心特性**:
- ✅ 分级日志（DEBUG、INFO、WARN、ERROR）
- ✅ 自动滚动策略
- ✅ 异步输出优化
- ✅ 支持 Java、Node.js、Python

---

### 3. 其他日志配置模板

**文件**: [`../../mocha-documents/logging/03-其他日志配置模板.md`](../../mocha-documents/logging/03-其他日志配置模板.md)

**适用场景**:
- ELK Stack 日志收集
- Fluentd 日志聚合
- Splunk 企业日志管理
- Kubernetes 云原生环境
- 审计日志
- 性能监控日志

**包含内容**:
- ELK Stack 配置（Elasticsearch + Logstash + Kibana）
- Fluentd 配置
- Splunk 配置
- Kubernetes 日志配置
- 结构化 JSON 日志
- 审计日志配置
- 性能日志配置

---

## 🚀 快速开始

### 1. 选择合适的模板

根据你的架构和需求选择：

- **多服务 + Datadog** → 模板 1
- **单服务** → 模板 2
- **ELK/Fluentd/K8s 等** → 模板 3

### 2. 查看模板

```bash
# 查看多服务 Datadog 模板
cat ../../mocha-documents/logging/01-多服务-datadog-分布式追踪配置模板.md

# 查看单服务模板
cat ../../mocha-documents/logging/02-单服务日志配置模板.md

# 查看其他模板
cat ../../mocha-documents/logging/03-其他日志配置模板.md
```

### 3. 复制配置

根据模板中的配置示例，复制到你的项目中：

```bash
# 示例：复制 Logback 配置
cp ../../mocha-documents/logging/01-多服务-datadog-分布式追踪配置模板.md \
   your-project/src/main/resources/logback-spring.xml
```

---

## 📚 相关文档

- [Datadog 官方文档](https://docs.datadoghq.com/)
- [Logback 文档](http://logback.qos.ch/)
- [Winston 文档](https://github.com/winstonjs/winston)
- [Loguru 文档](https://loguru.readthedocs.io/)

---

## 💡 使用建议

1. **多服务架构**: 优先使用模板 1，确保完整的分布式追踪
2. **单服务应用**: 使用模板 2，简单高效
3. **云原生环境**: 使用模板 3 中的 Kubernetes 配置
4. **企业级需求**: 考虑使用模板 3 中的 ELK 或 Splunk 配置

---

**最后更新**: 2025-01-20

