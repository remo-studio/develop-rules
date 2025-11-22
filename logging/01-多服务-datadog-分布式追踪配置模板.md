# 多服务 Datadog 分布式追踪日志配置模板

## 📋 概述

本模板适用于多服务架构，使用 Datadog 进行日志管理和分布式追踪。通过 `span_id` 和 `parent_span_id` 实现跨服务的请求链路追踪。

## 🎯 核心特性

- ✅ **分布式追踪**: 通过 `trace_id`、`span_id`、`parent_span_id` 实现跨服务追踪
- ✅ **Datadog 集成**: JSON 格式日志，便于 Datadog 解析和查询
- ✅ **结构化日志**: 统一的日志格式，包含完整的上下文信息
- ✅ **服务标识**: 每个服务自动添加服务名称、环境等信息
- ✅ **链路追踪**: 支持服务间调用链路的完整追踪

---

## 📦 依赖配置

### Maven (Java/Spring Boot)

```xml
<dependencies>
    <!-- Logback -->
    <dependency>
        <groupId>ch.qos.logback</groupId>
        <artifactId>logback-classic</artifactId>
        <version>1.4.11</version>
    </dependency>
    
    <!-- Logstash Encoder for JSON -->
    <dependency>
        <groupId>net.logstash.logback</groupId>
        <artifactId>logstash-logback-encoder</artifactId>
        <version>7.4</version>
    </dependency>
    
    <!-- Datadog Agent (可选，用于 APM) -->
    <dependency>
        <groupId>com.datadoghq</groupId>
        <artifactId>dd-java-agent</artifactId>
        <version>1.20.0</version>
        <scope>provided</scope>
    </dependency>
</dependencies>
```

### Node.js/TypeScript

```json
{
  "dependencies": {
    "winston": "^3.11.0",
    "winston-datadog-logs": "^1.0.0",
    "dd-trace": "^3.20.0"
  }
}
```

---

## 🔧 Logback 配置 (Java)

### `logback-spring.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <!-- 环境变量配置 -->
    <springProperty scope="context" name="SERVICE_NAME" source="spring.application.name" defaultValue="unknown-service"/>
    <springProperty scope="context" name="ENV" source="spring.profiles.active" defaultValue="dev"/>
    <springProperty scope="context" name="LOG_DIR" source="logging.file.path" defaultValue="/logs"/>
    
    <!-- 日志输出格式 -->
    <property name="LOG_PATTERN" value="%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n"/>
    
    <!-- JSON 编码器配置 -->
    <include resource="org/springframework/boot/logging/logback/defaults.xml"/>
    
    <!-- 控制台输出 (开发环境) -->
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder class="net.logstash.logback.encoder.LoggingEventCompositeJsonEncoder">
            <providers>
                <timestamp>
                    <timeZone>UTC</timeZone>
                </timestamp>
                <version/>
                <logLevel/>
                <message/>
                <mdc/>
                <stackTrace>
                    <throwableConverter class="net.logstash.logback.stacktrace.ShortenedThrowableConverter">
                        <maxDepthPerThrowable>30</maxDepthPerThrowable>
                        <maxLength>2048</maxLength>
                    </throwableConverter>
                </stackTrace>
                <pattern>
                    <pattern>
                        {
                            "service": "${SERVICE_NAME}",
                            "env": "${ENV}",
                            "thread": "%thread",
                            "logger": "%logger{50}"
                        }
                    </pattern>
                </pattern>
            </providers>
        </encoder>
    </appender>
    
    <!-- JSON 文件输出 (生产环境) -->
    <appender name="JSON_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${LOG_DIR}/${SERVICE_NAME}/app.json</file>
        <encoder class="net.logstash.logback.encoder.LoggingEventCompositeJsonEncoder">
            <providers>
                <timestamp>
                    <timeZone>UTC</timeZone>
                </timestamp>
                <version/>
                <logLevel/>
                <message/>
                <mdc/>
                <stackTrace>
                    <throwableConverter class="net.logstash.logback.stacktrace.ShortenedThrowableConverter">
                        <maxDepthPerThrowable>30</maxDepthPerThrowable>
                        <maxLength>2048</maxLength>
                    </throwableConverter>
                </stackTrace>
                <pattern>
                    <pattern>
                        {
                            "service": "${SERVICE_NAME}",
                            "env": "${ENV}",
                            "host": "${HOSTNAME:-unknown}",
                            "thread": "%thread",
                            "logger": "%logger{50}"
                        }
                    </pattern>
                </pattern>
            </providers>
        </encoder>
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <fileNamePattern>${LOG_DIR}/${SERVICE_NAME}/app-%d{yyyy-MM-dd}.%i.json</fileNamePattern>
            <maxHistory>30</maxHistory>
            <maxFileSize>100MB</maxFileSize>
            <totalSizeCap>10GB</totalSizeCap>
        </rollingPolicy>
    </appender>
    
    <!-- 错误日志单独输出 -->
    <appender name="ERROR_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${LOG_DIR}/${SERVICE_NAME}/error.json</file>
        <encoder class="net.logstash.logback.encoder.LoggingEventCompositeJsonEncoder">
            <providers>
                <timestamp>
                    <timeZone>UTC</timeZone>
                </timestamp>
                <version/>
                <logLevel/>
                <message/>
                <mdc/>
                <stackTrace>
                    <throwableConverter class="net.logstash.logback.stacktrace.ShortenedThrowableConverter">
                        <maxDepthPerThrowable>50</maxDepthPerThrowable>
                        <maxLength>4096</maxLength>
                    </throwableConverter>
                </stackTrace>
                <pattern>
                    <pattern>
                        {
                            "service": "${SERVICE_NAME}",
                            "env": "${ENV}",
                            "host": "${HOSTNAME:-unknown}",
                            "thread": "%thread",
                            "logger": "%logger{50}"
                        }
                    </pattern>
                </pattern>
            </providers>
        </encoder>
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>ERROR</level>
        </filter>
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <fileNamePattern>${LOG_DIR}/${SERVICE_NAME}/error-%d{yyyy-MM-dd}.%i.json</fileNamePattern>
            <maxHistory>90</maxHistory>
            <maxFileSize>100MB</maxFileSize>
            <totalSizeCap>20GB</totalSizeCap>
        </rollingPolicy>
    </appender>
    
    <!-- 异步输出提升性能 -->
    <appender name="ASYNC_JSON_FILE" class="ch.qos.logback.classic.AsyncAppender">
        <queueSize>512</queueSize>
        <discardingThreshold>0</discardingThreshold>
        <appender-ref ref="JSON_FILE"/>
    </appender>
    
    <appender name="ASYNC_ERROR_FILE" class="ch.qos.logback.classic.AsyncAppender">
        <queueSize>256</queueSize>
        <discardingThreshold>0</discardingThreshold>
        <appender-ref ref="ERROR_FILE"/>
    </appender>
    
    <!-- 根日志级别 -->
    <root level="INFO">
        <appender-ref ref="CONSOLE"/>
        <appender-ref ref="ASYNC_JSON_FILE"/>
        <appender-ref ref="ASYNC_ERROR_FILE"/>
    </root>
    
    <!-- 业务日志级别 -->
    <logger name="com.greenutility" level="DEBUG" additivity="false">
        <appender-ref ref="CONSOLE"/>
        <appender-ref ref="ASYNC_JSON_FILE"/>
    </logger>
</configuration>
```

---

## 🔧 链路追踪工具类 (Java)

### `TraceUtils.java`

```java
package com.greenutility.common.util;

import org.slf4j.MDC;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import java.util.UUID;

/**
 * 分布式追踪工具类
 * 用于管理 trace_id、span_id、parent_span_id
 */
public class TraceUtils {
    private static final Logger log = LoggerFactory.getLogger(TraceUtils.class);
    
    private static final String TRACE_ID = "trace_id";
    private static final String SPAN_ID = "span_id";
    private static final String PARENT_SPAN_ID = "parent_span_id";
    private static final String REQUEST_ID = "request_id";
    private static final String USER_ID = "user_id";
    private static final String IP = "ip";
    
    /**
     * 开始新的追踪链路
     * 生成新的 trace_id 和 span_id
     */
    public static void startTrace() {
        String traceId = generateId();
        String spanId = generateId();
        
        MDC.put(TRACE_ID, traceId);
        MDC.put(SPAN_ID, spanId);
        MDC.remove(PARENT_SPAN_ID);
        
        log.debug("Started new trace: trace_id={}, span_id={}", traceId, spanId);
    }
    
    /**
     * 从上游服务继承追踪信息
     * @param traceId 上游服务的 trace_id
     * @param parentSpanId 上游服务的 span_id（作为当前服务的 parent_span_id）
     */
    public static void inheritTrace(String traceId, String parentSpanId) {
        String spanId = generateId();
        
        MDC.put(TRACE_ID, traceId);
        MDC.put(SPAN_ID, spanId);
        MDC.put(PARENT_SPAN_ID, parentSpanId);
        
        log.debug("Inherited trace: trace_id={}, parent_span_id={}, span_id={}", 
                  traceId, parentSpanId, spanId);
    }
    
    /**
     * 创建子 span（用于服务内部嵌套调用）
     * @return 新的 span_id
     */
    public static String createChildSpan() {
        String currentSpanId = MDC.get(SPAN_ID);
        String newSpanId = generateId();
        
        MDC.put(PARENT_SPAN_ID, currentSpanId);
        MDC.put(SPAN_ID, newSpanId);
        
        log.debug("Created child span: parent_span_id={}, span_id={}", 
                  currentSpanId, newSpanId);
        
        return newSpanId;
    }
    
    /**
     * 恢复父 span
     */
    public static void restoreParentSpan() {
        String parentSpanId = MDC.get(PARENT_SPAN_ID);
        if (parentSpanId != null) {
            MDC.put(SPAN_ID, parentSpanId);
            MDC.remove(PARENT_SPAN_ID);
        }
    }
    
    /**
     * 设置请求ID
     */
    public static void setRequestId(String requestId) {
        MDC.put(REQUEST_ID, requestId);
    }
    
    /**
     * 设置用户ID
     */
    public static void setUserId(String userId) {
        MDC.put(USER_ID, userId);
    }
    
    /**
     * 设置IP地址
     */
    public static void setIp(String ip) {
        MDC.put(IP, ip);
    }
    
    /**
     * 获取当前 trace_id（用于传递给下游服务）
     */
    public static String getTraceId() {
        return MDC.get(TRACE_ID);
    }
    
    /**
     * 获取当前 span_id（用于传递给下游服务）
     */
    public static String getSpanId() {
        return MDC.get(SPAN_ID);
    }
    
    /**
     * 获取追踪头信息（用于 HTTP 调用）
     */
    public static String getTraceHeader() {
        return String.format("X-Trace-Id: %s, X-Span-Id: %s", 
                            getTraceId(), getSpanId());
    }
    
    /**
     * 清除追踪信息
     */
    public static void clearTrace() {
        MDC.remove(TRACE_ID);
        MDC.remove(SPAN_ID);
        MDC.remove(PARENT_SPAN_ID);
        MDC.remove(REQUEST_ID);
        MDC.remove(USER_ID);
        MDC.remove(IP);
    }
    
    /**
     * 生成唯一ID
     */
    private static String generateId() {
        return UUID.randomUUID().toString().replace("-", "").substring(0, 16);
    }
}
```

---

## 🔧 HTTP 拦截器 (Spring Boot)

### `TraceInterceptor.java`

```java
package com.greenutility.common.config;

import com.greenutility.common.util.TraceUtils;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Component;
import org.springframework.web.servlet.HandlerInterceptor;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

@Component
public class TraceInterceptor implements HandlerInterceptor {
    private static final Logger log = LoggerFactory.getLogger(TraceInterceptor.class);
    
    private static final String TRACE_ID_HEADER = "X-Trace-Id";
    private static final String SPAN_ID_HEADER = "X-Span-Id";
    
    @Override
    public boolean preHandle(HttpServletRequest request, 
                           HttpServletResponse response, 
                           Object handler) {
        // 从请求头获取追踪信息
        String traceId = request.getHeader(TRACE_ID_HEADER);
        String spanId = request.getHeader(SPAN_ID_HEADER);
        
        if (traceId != null && spanId != null) {
            // 继承上游服务的追踪信息
            TraceUtils.inheritTrace(traceId, spanId);
        } else {
            // 创建新的追踪链路
            TraceUtils.startTrace();
        }
        
        // 设置请求相关信息
        TraceUtils.setRequestId(request.getRequestId());
        TraceUtils.setIp(getClientIp(request));
        
        // 将追踪信息添加到响应头，供下游服务使用
        response.setHeader(TRACE_ID_HEADER, TraceUtils.getTraceId());
        response.setHeader(SPAN_ID_HEADER, TraceUtils.getSpanId());
        
        log.info("Request: {} {}, trace_id={}, span_id={}", 
                 request.getMethod(), 
                 request.getRequestURI(),
                 TraceUtils.getTraceId(),
                 TraceUtils.getSpanId());
        
        return true;
    }
    
    @Override
    public void afterCompletion(HttpServletRequest request, 
                               HttpServletResponse response, 
                               Object handler, 
                               Exception ex) {
        if (ex != null) {
            log.error("Request failed: {} {}, error={}", 
                     request.getMethod(), 
                     request.getRequestURI(),
                     ex.getMessage(), 
                     ex);
        }
        
        // 清除追踪信息
        TraceUtils.clearTrace();
    }
    
    private String getClientIp(HttpServletRequest request) {
        String ip = request.getHeader("X-Forwarded-For");
        if (ip == null || ip.isEmpty() || "unknown".equalsIgnoreCase(ip)) {
            ip = request.getHeader("X-Real-IP");
        }
        if (ip == null || ip.isEmpty() || "unknown".equalsIgnoreCase(ip)) {
            ip = request.getRemoteAddr();
        }
        return ip;
    }
}
```

---

## 🔧 Feign 客户端配置 (服务间调用)

### `FeignTraceConfig.java`

```java
package com.greenutility.common.config;

import com.greenutility.common.util.TraceUtils;
import feign.RequestInterceptor;
import feign.RequestTemplate;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class FeignTraceConfig {
    
    @Bean
    public RequestInterceptor traceRequestInterceptor() {
        return new RequestInterceptor() {
            @Override
            public void apply(RequestTemplate template) {
                // 将追踪信息添加到请求头
                String traceId = TraceUtils.getTraceId();
                String spanId = TraceUtils.getSpanId();
                
                if (traceId != null) {
                    template.header("X-Trace-Id", traceId);
                }
                if (spanId != null) {
                    template.header("X-Span-Id", spanId);
                }
            }
        };
    }
}
```

---

## 🔧 Node.js/TypeScript 配置

### `logger.ts`

```typescript
import winston from 'winston';
import { createLogger, format, transports } from 'winston';
import { v4 as uuidv4 } from 'uuid';

// 追踪信息存储
const traceContext = {
  traceId: '',
  spanId: '',
  parentSpanId: '',
  requestId: '',
  userId: '',
  ip: '',
};

// 追踪工具函数
export const TraceUtils = {
  startTrace() {
    traceContext.traceId = uuidv4().replace(/-/g, '').substring(0, 16);
    traceContext.spanId = uuidv4().replace(/-/g, '').substring(0, 16);
    traceContext.parentSpanId = '';
  },
  
  inheritTrace(traceId: string, parentSpanId: string) {
    traceContext.traceId = traceId;
    traceContext.spanId = uuidv4().replace(/-/g, '').substring(0, 16);
    traceContext.parentSpanId = parentSpanId;
  },
  
  createChildSpan(): string {
    const currentSpanId = traceContext.spanId;
    traceContext.parentSpanId = currentSpanId;
    traceContext.spanId = uuidv4().replace(/-/g, '').substring(0, 16);
    return traceContext.spanId;
  },
  
  setRequestId(requestId: string) {
    traceContext.requestId = requestId;
  },
  
  setUserId(userId: string) {
    traceContext.userId = userId;
  },
  
  setIp(ip: string) {
    traceContext.ip = ip;
  },
  
  getTraceId(): string {
    return traceContext.traceId;
  },
  
  getSpanId(): string {
    return traceContext.spanId;
  },
  
  clearTrace() {
    traceContext.traceId = '';
    traceContext.spanId = '';
    traceContext.parentSpanId = '';
    traceContext.requestId = '';
    traceContext.userId = '';
    traceContext.ip = '';
  },
};

// 自定义格式，包含追踪信息
const traceFormat = format((info) => {
  info.trace_id = traceContext.traceId;
  info.span_id = traceContext.spanId;
  info.parent_span_id = traceContext.parentSpanId || undefined;
  info.request_id = traceContext.requestId || undefined;
  info.user_id = traceContext.userId || undefined;
  info.ip = traceContext.ip || undefined;
  info.service = process.env.SERVICE_NAME || 'unknown-service';
  info.env = process.env.NODE_ENV || 'development';
  info.host = require('os').hostname();
  return info;
})();

// 创建 Logger
export const logger = createLogger({
  level: process.env.LOG_LEVEL || 'info',
  format: format.combine(
    format.timestamp(),
    traceFormat,
    format.json()
  ),
  transports: [
    new transports.Console({
      format: format.combine(
        format.colorize(),
        format.simple()
      ),
    }),
    new transports.File({
      filename: `${process.env.LOG_DIR || '/logs'}/app.json`,
      maxsize: 104857600, // 100MB
      maxFiles: 30,
    }),
    new transports.File({
      filename: `${process.env.LOG_DIR || '/logs'}/error.json`,
      level: 'error',
      maxsize: 104857600, // 100MB
      maxFiles: 90,
    }),
  ],
});
```

---

## 📊 日志格式示例

### JSON 日志输出

```json
{
  "timestamp": "2025-01-20T10:30:45.123Z",
  "level": "INFO",
  "message": "Processing payment request",
  "service": "payment-service",
  "env": "production",
  "host": "payment-service-01",
  "thread": "http-nio-8080-exec-1",
  "logger": "com.greenutility.pay.service.PayService",
  "trace_id": "a1b2c3d4e5f6g7h8",
  "span_id": "i9j0k1l2m3n4o5p6",
  "parent_span_id": "q7r8s9t0u1v2w3x4",
  "request_id": "req-123456",
  "user_id": "user-789",
  "ip": "192.168.1.100",
  "order_id": "order-12345",
  "amount": 100.00,
  "payment_method": "alipay"
}
```

---

## 🚀 部署配置

### Docker Compose 环境变量

```yaml
services:
  payment-service:
    environment:
      - SERVICE_NAME=payment-service
      - ENV=production
      - LOG_DIR=/logs
      - DD_AGENT_HOST=datadog-agent
      - DD_TRACE_ENABLED=true
      - DD_SERVICE=payment-service
      - DD_ENV=production
```

### Datadog Agent 配置

```yaml
# datadog.yaml
logs_enabled: true
logs_config:
  container_collect_all: true
apm_config:
  enabled: true
  non_local_traffic: true
```

---

## 📝 使用示例

### Java 服务间调用

```java
@Service
public class OrderService {
    private static final Logger log = LoggerFactory.getLogger(OrderService.class);
    
    @Autowired
    private PaymentServiceClient paymentServiceClient;
    
    public void createOrder(OrderRequest request) {
        // 创建子 span
        String childSpanId = TraceUtils.createChildSpan();
        
        try {
            log.info("Creating order: order_id={}, user_id={}", 
                    request.getOrderId(), 
                    request.getUserId());
            
            // 调用支付服务（自动传递 trace_id 和 span_id）
            PaymentResponse response = paymentServiceClient.processPayment(
                request.getPaymentInfo()
            );
            
            log.info("Order created successfully: order_id={}, payment_id={}", 
                    request.getOrderId(), 
                    response.getPaymentId());
        } finally {
            // 恢复父 span
            TraceUtils.restoreParentSpan();
        }
    }
}
```

### Node.js 服务间调用

```typescript
import axios from 'axios';
import { logger, TraceUtils } from './logger';

export async function callPaymentService(paymentData: any) {
  const childSpanId = TraceUtils.createChildSpan();
  
  try {
    logger.info('Calling payment service', { payment_data: paymentData });
    
    const response = await axios.post('http://payment-service/api/pay', paymentData, {
      headers: {
        'X-Trace-Id': TraceUtils.getTraceId(),
        'X-Span-Id': TraceUtils.getSpanId(),
      },
    });
    
    logger.info('Payment service response', { 
      payment_id: response.data.paymentId 
    });
    
    return response.data;
  } finally {
    TraceUtils.restoreParentSpan();
  }
}
```

---

## 🔍 Datadog 查询示例

### 查询特定 trace_id 的所有日志

```
trace_id:a1b2c3d4e5f6g7h8
```

### 查询特定服务的错误日志

```
service:payment-service level:error
```

### 查询包含特定订单的日志

```
service:payment-service order_id:order-12345
```

### 查询跨服务调用链路

```
trace_id:a1b2c3d4e5f6g7h8
```

---

## ✅ 最佳实践

1. **统一追踪头**: 使用 `X-Trace-Id` 和 `X-Span-Id` 作为标准 HTTP 头
2. **自动继承**: 在网关/拦截器中自动处理追踪信息的继承
3. **子 span 管理**: 服务内部嵌套调用时创建子 span
4. **清理资源**: 请求结束后及时清理 MDC/上下文信息
5. **错误追踪**: 确保错误日志也包含完整的追踪信息
6. **性能考虑**: 使用异步日志输出，避免影响业务性能

---

## 📚 相关文档

- [Datadog Log Management](https://docs.datadoghq.com/logs/)
- [Datadog APM](https://docs.datadoghq.com/tracing/)
- [Logback Documentation](http://logback.qos.ch/)
- [Winston Documentation](https://github.com/winstonjs/winston)

