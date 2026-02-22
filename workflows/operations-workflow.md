# 运营流程规范

本文档定义生产环境的监控、告警、日志分析、性能优化和用户反馈收集流程。

## 监控体系

### 监控层次

```
┌─────────────────────────────────────┐
│  Business Metrics (业务指标)        │
│  - DAU/MAU, 转化率, 收入            │
└─────────────────────────────────────┘
              ↑
┌─────────────────────────────────────┐
│  Application Metrics (应用指标)     │
│  - 请求量, 响应时间, 错误率          │
└─────────────────────────────────────┘
              ↑
┌─────────────────────────────────────┐
│  System Metrics (系统指标)          │
│  - CPU, 内存, 磁盘, 网络             │
└─────────────────────────────────────┘
              ↑
┌─────────────────────────────────────┐
│  Infrastructure (基础设施)          │
│  - 服务器, 数据库, 负载均衡器        │
└─────────────────────────────────────┘
```

### 关键指标 (Golden Signals)

#### 1. 延迟 (Latency)
```javascript
// 使用 Prometheus 客户端记录延迟
const httpRequestDuration = new promClient.Histogram({
  name: 'http_request_duration_seconds',
  help: 'HTTP request latency',
  labelNames: ['method', 'route', 'status_code'],
  buckets: [0.1, 0.3, 0.5, 0.7, 1, 3, 5, 7, 10],
});

// 中间件
app.use((req, res, next) => {
  const start = Date.now();
  
  res.on('finish', () => {
    const duration = (Date.now() - start) / 1000;
    httpRequestDuration
      .labels(req.method, req.route?.path || req.path, res.statusCode)
      .observe(duration);
  });
  
  next();
});
```

**告警规则**：
- P95 延迟 > 1s：警告
- P95 延迟 > 3s：严重
- P99 延迟 > 5s：紧急

#### 2. 流量 (Traffic)
```javascript
const httpRequestTotal = new promClient.Counter({
  name: 'http_requests_total',
  help: 'Total HTTP requests',
  labelNames: ['method', 'route', 'status_code'],
});

app.use((req, res, next) => {
  res.on('finish', () => {
    httpRequestTotal
      .labels(req.method, req.route?.path || req.path, res.statusCode)
      .inc();
  });
  
  next();
});
```

**告警规则**：
- QPS 突降 > 50%：警告
- QPS 突增 > 200%：警告（可能攻击）

#### 3. 错误率 (Errors)
```javascript
const httpRequestErrors = new promClient.Counter({
  name: 'http_request_errors_total',
  help: 'Total HTTP errors',
  labelNames: ['method', 'route', 'error_type'],
});

app.use((err, req, res, next) => {
  httpRequestErrors
    .labels(req.method, req.route?.path || req.path, err.name)
    .inc();
  
  next(err);
});
```

**告警规则**：
- 5xx 错误率 > 1%：警告
- 5xx 错误率 > 5%：严重
- 4xx 错误率 > 10%：警告

#### 4. 饱和度 (Saturation)
```javascript
// 系统资源使用率
const cpuUsage = new promClient.Gauge({
  name: 'process_cpu_usage_percent',
  help: 'CPU usage percentage',
});

const memoryUsage = new promClient.Gauge({
  name: 'process_memory_usage_bytes',
  help: 'Memory usage in bytes',
});

setInterval(() => {
  const usage = process.cpuUsage();
  cpuUsage.set(usage.user / 1000000);
  
  const mem = process.memoryUsage();
  memoryUsage.set(mem.heapUsed);
}, 5000);
```

**告警规则**：
- CPU 使用率 > 70%：警告
- CPU 使用率 > 90%：严重
- 内存使用率 > 80%：警告
- 磁盘使用率 > 85%：警告

### 业务指标监控

#### 用户指标
```javascript
// 用户注册
const userRegistrations = new promClient.Counter({
  name: 'user_registrations_total',
  help: 'Total user registrations',
  labelNames: ['source'],
});

// 用户登录
const userLogins = new promClient.Counter({
  name: 'user_logins_total',
  help: 'Total user logins',
  labelNames: ['method'],
});

// 活跃用户（使用 Redis）
async function trackActiveUsers(userId) {
  const today = moment().format('YYYY-MM-DD');
  await redis.sadd(`active_users:${today}`, userId);
  await redis.expire(`active_users:${today}`, 86400 * 30); // 保留 30 天
}

// 定期报告 DAU/MAU
setInterval(async () => {
  const today = moment().format('YYYY-MM-DD');
  const dau = await redis.scard(`active_users:${today}`);
  
  // 计算 MAU
  const last30Days = Array.from({ length: 30 }, (_, i) => 
    moment().subtract(i, 'days').format('YYYY-MM-DD')
  );
  const mau = await redis.sunionstore('temp_mau', 
    ...last30Days.map(d => `active_users:${d}`)
  );
  
  logger.info('User metrics', { dau, mau });
}, 300000); // 每 5 分钟
```

#### 转化指标
```javascript
// 转化漏斗
const conversionFunnel = new promClient.Counter({
  name: 'conversion_funnel_total',
  help: 'Conversion funnel steps',
  labelNames: ['step'],
});

// 记录转化步骤
function trackConversion(step) {
  conversionFunnel.labels(step).inc();
}

// 使用示例
trackConversion('view_landing');
trackConversion('click_signup');
trackConversion('complete_registration');
trackConversion('first_purchase');
```

### 监控工具集成

#### Prometheus + Grafana
```yaml
# prometheus.yml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: 'myapp'
    static_configs:
      - targets: ['localhost:3000']
    metrics_path: '/metrics'

  - job_name: 'postgres'
    static_configs:
      - targets: ['localhost:9187']

  - job_name: 'redis'
    static_configs:
      - targets: ['localhost:9121']
```

#### Datadog 集成
```javascript
// datadog.js
import StatsD from 'hot-shots';

const dogstatsd = new StatsD({
  host: process.env.DD_AGENT_HOST,
  port: 8125,
  prefix: 'myapp.',
  globalTags: {
    env: process.env.NODE_ENV,
    service: 'api',
  },
});

// 记录指标
dogstatsd.increment('user.login', 1, ['method:oauth']);
dogstatsd.histogram('request.duration', 245, ['route:/api/users']);
dogstatsd.gauge('database.connections', 20);

export default dogstatsd;
```

## 日志管理

### 日志级别
- **ERROR**：错误，需要立即关注
- **WARN**：警告，可能有问题但不影响核心功能
- **INFO**：重要业务事件
- **DEBUG**：调试信息（仅开发/测试环境）
- **TRACE**：详细追踪信息（仅开发环境）

### 结构化日志

#### Winston 配置
```javascript
// logger.js
import winston from 'winston';

const logger = winston.createLogger({
  level: process.env.LOG_LEVEL || 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.errors({ stack: true }),
    winston.format.json()
  ),
  defaultMeta: {
    service: 'myapp-api',
    environment: process.env.NODE_ENV,
  },
  transports: [
    // 输出到控制台
    new winston.transports.Console({
      format: winston.format.combine(
        winston.format.colorize(),
        winston.format.simple()
      ),
    }),
    // 输出到文件
    new winston.transports.File({
      filename: 'logs/error.log',
      level: 'error',
    }),
    new winston.transports.File({
      filename: 'logs/combined.log',
    }),
  ],
});

// 生产环境发送到日志收集服务
if (process.env.NODE_ENV === 'production') {
  logger.add(new winston.transports.Http({
    host: process.env.LOG_COLLECTOR_HOST,
    port: process.env.LOG_COLLECTOR_PORT,
    path: '/logs',
  }));
}

export default logger;
```

#### 使用示例
```javascript
// 好的日志实践 ✅
logger.info('User login successful', {
  userId: user.id,
  username: user.username,
  ip: req.ip,
  userAgent: req.get('user-agent'),
  timestamp: new Date().toISOString(),
});

logger.error('Database query failed', {
  error: error.message,
  stack: error.stack,
  query: 'SELECT * FROM users WHERE id = $1',
  params: [userId],
  duration: 1250,
});

// 不好的日志实践 ❌
console.log('User logged in');  // 缺少上下文
logger.info('Error: ' + error);  // 丢失堆栈信息
logger.info(user);  // 可能包含敏感信息
```

### 日志收集与分析

#### ELK Stack (Elasticsearch + Logstash + Kibana)
```yaml
# docker-compose.yml
version: '3.8'

services:
  elasticsearch:
    image: elasticsearch:8.11.0
    environment:
      - discovery.type=single-node
      - xpack.security.enabled=false
    ports:
      - "9200:9200"
    volumes:
      - es-data:/usr/share/elasticsearch/data

  logstash:
    image: logstash:8.11.0
    volumes:
      - ./logstash.conf:/usr/share/logstash/pipeline/logstash.conf
    ports:
      - "5044:5044"
    depends_on:
      - elasticsearch

  kibana:
    image: kibana:8.11.0
    ports:
      - "5601:5601"
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
    depends_on:
      - elasticsearch

volumes:
  es-data:
```

#### 日志查询示例
```
# Kibana Query Language (KQL)

# 查找所有错误日志
level: "error"

# 查找特定用户的日志
userId: "123" AND level: "info"

# 查找慢查询
duration > 1000

# 查找 API 错误
service: "myapp-api" AND statusCode >= 500

# 时间范围查询
@timestamp >= "2025-01-01" AND @timestamp <= "2025-01-31"
```

### 日志保留策略
- **ERROR 日志**：保留 90 天
- **WARN 日志**：保留 60 天
- **INFO 日志**：保留 30 天
- **DEBUG 日志**：保留 7 天（仅非生产环境）

## 告警系统

### 告警级别
- **P0 (Critical)**：核心功能不可用，立即处理
- **P1 (High)**：重要功能受影响，1 小时内处理
- **P2 (Medium)**：次要功能问题，当天处理
- **P3 (Low)**：优化建议，计划处理

### 告警规则

#### Prometheus 告警规则
```yaml
# alerts.yml
groups:
  - name: api_alerts
    interval: 30s
    rules:
      # 高错误率告警
      - alert: HighErrorRate
        expr: |
          rate(http_request_errors_total[5m]) 
          / rate(http_requests_total[5m]) > 0.05
        for: 5m
        labels:
          severity: critical
          team: backend
        annotations:
          summary: "High error rate detected"
          description: "Error rate is {{ $value | humanizePercentage }} (threshold: 5%)"

      # 高延迟告警
      - alert: HighLatency
        expr: |
          histogram_quantile(0.95, 
            rate(http_request_duration_seconds_bucket[5m])
          ) > 1
        for: 5m
        labels:
          severity: warning
          team: backend
        annotations:
          summary: "High latency detected"
          description: "P95 latency is {{ $value }}s (threshold: 1s)"

      # 服务宕机告警
      - alert: ServiceDown
        expr: up == 0
        for: 1m
        labels:
          severity: critical
          team: sre
        annotations:
          summary: "Service is down"
          description: "{{ $labels.instance }} has been down for more than 1 minute"

      # 数据库连接池耗尽
      - alert: DatabaseConnectionPoolExhausted
        expr: |
          database_connections_active 
          / database_connections_max > 0.9
        for: 2m
        labels:
          severity: critical
          team: backend
        annotations:
          summary: "Database connection pool nearly exhausted"
          description: "{{ $value | humanizePercentage }} of connections in use"

      # 磁盘空间不足
      - alert: DiskSpaceLow
        expr: |
          (node_filesystem_avail_bytes 
          / node_filesystem_size_bytes) < 0.15
        for: 5m
        labels:
          severity: warning
          team: sre
        annotations:
          summary: "Disk space is running low"
          description: "Only {{ $value | humanizePercentage }} space remaining"
```

### 告警通知

#### Slack 集成
```javascript
// alert-manager.js
import axios from 'axios';

async function sendSlackAlert(alert) {
  const color = {
    critical: '#FF0000',
    warning: '#FFA500',
    info: '#0000FF',
  }[alert.severity] || '#808080';

  const message = {
    attachments: [
      {
        color,
        title: alert.summary,
        text: alert.description,
        fields: [
          {
            title: 'Severity',
            value: alert.severity,
            short: true,
          },
          {
            title: 'Team',
            value: alert.team,
            short: true,
          },
          {
            title: 'Time',
            value: new Date().toISOString(),
            short: false,
          },
        ],
      },
    ],
  };

  await axios.post(process.env.SLACK_WEBHOOK_URL, message);
}
```

#### PagerDuty 集成
```javascript
// pagerduty.js
import axios from 'axios';

async function triggerPagerDutyIncident(alert) {
  const event = {
    routing_key: process.env.PAGERDUTY_ROUTING_KEY,
    event_action: 'trigger',
    payload: {
      summary: alert.summary,
      severity: alert.severity,
      source: 'myapp-monitoring',
      custom_details: alert.details,
    },
  };

  await axios.post(
    'https://events.pagerduty.com/v2/enqueue',
    event
  );
}
```

### 告警疲劳防护
1. **聚合**：5 分钟内相同告警只发送一次
2. **静默**：维护期间暂停告警
3. **抑制**：上游告警触发时，抑制下游告警
4. **升级**：P0 告警 5 分钟未响应，自动升级并通知更多人

## 性能优化

### 性能分析

#### Node.js 性能分析
```bash
# 生成火焰图
node --prof app.js
node --prof-process isolate-*.log > profile.txt

# 使用 clinic.js
npx clinic doctor -- node app.js
npx clinic flame -- node app.js
npx clinic bubbleprof -- node app.js
```

#### 数据库查询优化
```sql
-- 分析查询计划
EXPLAIN ANALYZE
SELECT * FROM users WHERE email = 'user@example.com';

-- 查找慢查询
SELECT query, calls, total_time, mean_time
FROM pg_stat_statements
ORDER BY mean_time DESC
LIMIT 10;

-- 查找缺失的索引
SELECT schemaname, tablename, attname
FROM pg_stats
WHERE correlation < 0.1
  AND n_distinct > 100;
```

### 缓存策略

#### Redis 缓存
```javascript
// cache.js
import Redis from 'ioredis';

const redis = new Redis(process.env.REDIS_URL);

// 缓存装饰器
function cached(ttl = 300) {
  return function (target, propertyKey, descriptor) {
    const originalMethod = descriptor.value;
    
    descriptor.value = async function (...args) {
      const key = `cache:${propertyKey}:${JSON.stringify(args)}`;
      
      // 尝试从缓存获取
      const cached = await redis.get(key);
      if (cached) {
        logger.debug('Cache hit', { key });
        return JSON.parse(cached);
      }
      
      // 缓存未命中，执行原方法
      logger.debug('Cache miss', { key });
      const result = await originalMethod.apply(this, args);
      
      // 存入缓存
      await redis.setex(key, ttl, JSON.stringify(result));
      
      return result;
    };
    
    return descriptor;
  };
}

// 使用示例
class UserService {
  @cached(600) // 缓存 10 分钟
  async getUserById(userId) {
    return await db.users.findOne({ id: userId });
  }
}
```

#### CDN 缓存
```nginx
# nginx.conf
location /static/ {
    alias /var/www/static/;
    expires 1y;
    add_header Cache-Control "public, immutable";
}

location /api/ {
    proxy_pass http://backend;
    add_header Cache-Control "no-cache, no-store, must-revalidate";
}
```

### 性能指标目标
- **首页加载时间**：< 2s
- **API 响应时间（P95）**：< 500ms
- **数据库查询时间（P95）**：< 100ms
- **缓存命中率**：> 80%
- **错误率**：< 0.1%

## 用户反馈收集

### 反馈渠道

#### 1. 应用内反馈
```javascript
// feedback API
app.post('/api/v1/feedback', async (req, res) => {
  const { type, message, metadata } = req.body;
  
  const feedback = await db.feedback.create({
    userId: req.user?.id,
    type, // bug | feature | improvement | other
    message,
    metadata: {
      ...metadata,
      userAgent: req.get('user-agent'),
      url: req.get('referer'),
      timestamp: new Date(),
    },
  });
  
  // 发送到 Slack
  await sendSlackMessage(`New ${type} feedback: ${message}`);
  
  res.json({ id: feedback.id });
});
```

#### 2. 用户访谈
- 定期组织用户访谈（1-2 次/月）
- 了解用户痛点和需求
- 验证产品假设

#### 3. NPS 调查
```javascript
// NPS (Net Promoter Score) 调查
app.get('/api/v1/nps', async (req, res) => {
  const { score, reason } = req.query;
  
  await db.nps.create({
    userId: req.user.id,
    score: parseInt(score), // 0-10
    reason,
    timestamp: new Date(),
  });
  
  res.json({ success: true });
});

// 计算 NPS
async function calculateNPS() {
  const responses = await db.nps.find({
    timestamp: { $gte: moment().subtract(30, 'days') },
  });
  
  const promoters = responses.filter(r => r.score >= 9).length;
  const detractors = responses.filter(r => r.score <= 6).length;
  const total = responses.length;
  
  const nps = ((promoters - detractors) / total) * 100;
  
  logger.info('NPS calculated', { nps, promoters, detractors, total });
  
  return nps;
}
```

### 数据分析

#### Google Analytics / Mixpanel
```javascript
// analytics.js
import mixpanel from 'mixpanel';

const mp = mixpanel.init(process.env.MIXPANEL_TOKEN);

// 追踪事件
function trackEvent(userId, event, properties) {
  mp.track(event, {
    distinct_id: userId,
    ...properties,
    timestamp: new Date(),
  });
}

// 使用示例
trackEvent(user.id, 'User Login', {
  method: 'oauth',
  provider: 'google',
});

trackEvent(user.id, 'Purchase Completed', {
  amount: 99.99,
  currency: 'USD',
  product: 'Premium Plan',
});
```

### A/B 测试
```javascript
// ab-test.js
function assignVariant(userId, experimentName) {
  const hash = crypto
    .createHash('md5')
    .update(userId + experimentName)
    .digest('hex');
  
  const value = parseInt(hash.substring(0, 8), 16);
  const variant = value % 2 === 0 ? 'A' : 'B';
  
  return variant;
}

// 使用示例
app.get('/api/v1/feature', (req, res) => {
  const variant = assignVariant(req.user.id, 'new_checkout_flow');
  
  if (variant === 'A') {
    // 原有流程
    return res.json({ flow: 'original' });
  } else {
    // 新流程
    return res.json({ flow: 'new' });
  }
});
```

## 事故管理

### 事故响应流程

#### 1. 发现问题
- 监控告警触发
- 用户报告
- 团队成员发现

#### 2. 确认严重程度
- **SEV-1**：核心功能完全不可用，影响所有用户
- **SEV-2**：重要功能受影响，影响部分用户
- **SEV-3**：次要功能问题，影响较小
- **SEV-4**：优化建议，不影响用户

#### 3. 启动应急响应
```markdown
# 事故响应清单

- [ ] 创建事故频道（Slack: #incident-YYYY-MM-DD）
- [ ] 指定事故指挥官（Incident Commander）
- [ ] 更新状态页面
- [ ] 通知相关团队
- [ ] 开始记录时间线
- [ ] 启动问题诊断
- [ ] 实施临时修复或回滚
- [ ] 验证修复效果
- [ ] 发布事故报告
- [ ] 安排事后复盘
```

#### 4. 事后复盘 (Post-mortem)
```markdown
# 事故复盘报告模板

## 事故概要
- **事故编号**：INC-2025-001
- **发生时间**：2025-01-10 14:30 UTC
- **持续时间**：45 分钟
- **严重程度**：SEV-2
- **影响范围**：约 30% 用户无法登录

## 时间线
- 14:30 - 监控告警：登录 API 错误率飙升
- 14:32 - 确认问题，启动应急响应
- 14:35 - 诊断发现数据库连接池耗尽
- 14:40 - 临时增加连接池大小
- 14:45 - 问题解决，错误率恢复正常
- 15:15 - 发布事故通告

## 根本原因
数据库连接未正确释放，导致连接池耗尽。

## 解决方案
1. 修复连接泄漏问题
2. 增加连接池监控告警
3. 添加连接自动回收机制

## 改进措施
- [ ] 代码审查流程增加资源管理检查
- [ ] 添加连接池使用率监控
- [ ] 编写连接管理最佳实践文档
- [ ] 进行团队培训

## 经验教训
1. 监控覆盖不全面，未监控连接池使用率
2. 缺少自动恢复机制
3. 需要更完善的测试覆盖
```

## 最佳实践

1. **主动监控**：不等问题发生才发现
2. **快速响应**：建立清晰的值班和升级机制
3. **持续改进**：每次事故后进行复盘
4. **自动化优先**：能自动化的运维任务都自动化
5. **文档完善**：记录所有运维流程和决策
6. **用户优先**：始终以用户体验为中心
7. **数据驱动**：基于数据做决策，不凭感觉
8. **安全第一**：在性能和安全间，优先选择安全

## 相关文档

- [部署流程](./deployment-workflow.md)
- [测试流程](./testing-workflow.md)
- [需求管理规范](./requirements-management.md)
- [AI 开发工作流](./ai-development-workflow.md)
