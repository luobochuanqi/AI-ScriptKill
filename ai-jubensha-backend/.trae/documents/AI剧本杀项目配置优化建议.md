# AI剧本杀项目配置优化建议

## 一、Docker Compose配置优化

### 1. 添加应用服务配置

**问题**：当前docker-compose.yml中缺少应用服务本身的配置，只包含了依赖服务。

**建议**：添加应用服务配置，使用多阶段构建减小镜像体积。

```yaml
# 应用服务
app:
  build:
    context: .
    dockerfile: Dockerfile
  container_name: ai-jubensha-app
  ports:
    - "8080:8080"
  environment:
    - SPRING_PROFILES_ACTIVE=docker
    - SPRING_DATASOURCE_URL=jdbc:mysql://mysql:3306/ai_jubensha
    - SPRING_DATASOURCE_USERNAME=root
    - SPRING_DATASOURCE_PASSWORD=root
    - SPRING_REDIS_HOST=redis
    - SPRING_RABBITMQ_HOST=rabbitmq
    - MILVUS_HOST=milvus
  depends_on:
    - mysql
    - redis
    - rabbitmq
    - milvus
  restart: always
  networks:
    - ai-jubensha-network
```

### 2. 添加Dockerfile

**问题**：缺少应用服务的Dockerfile。

**建议**：创建多阶段构建的Dockerfile。

```dockerfile
# 构建阶段
FROM maven:3.9-eclipse-temurin-17 AS builder
WORKDIR /app
COPY pom.xml .
COPY src ./src
RUN mvn clean package -DskipTests

# 运行阶段
FROM eclipse-temurin:17-jre-alpine
WORKDIR /app
COPY --from=builder /app/target/ai-jubensha-backend-*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

### 3. 优化存储配置

**问题**：当前存储配置使用默认设置，可能存在性能和安全性问题。

**建议**：
- 为MySQL添加配置文件，优化性能
- 为Redis添加密码认证
- 为Elasticsearch添加内存限制

## 二、POM.xml配置优化

### 1. 添加Elasticsearch客户端依赖

**问题**：缺少Elasticsearch的Java客户端依赖。

**建议**：添加Elasticsearch Java客户端依赖。

```xml
<!-- Elasticsearch客户端依赖 -->
<dependency>
    <groupId>co.elastic.clients</groupId>
    <artifactId>elasticsearch-java</artifactId>
    <version>8.6.0</version>
</dependency>
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
</dependency>
```

### 2. 完善Spring State Machine配置

**问题**：Spring State Machine依赖已添加，但可能需要更完整的配置。

**建议**：添加Spring State Machine的扩展依赖（如果需要）。

```xml
<!-- Spring State Machine扩展依赖（可选） -->
<dependency>
    <groupId>org.springframework.statemachine</groupId>
    <artifactId>spring-statemachine-starter</artifactId>
    <version>3.2.0</version>
</dependency>
```

### 3. 添加应用配置管理依赖

**问题**：缺少应用配置管理的依赖。

**建议**：添加Spring Cloud Config或其他配置管理依赖。

```xml
<!-- 配置管理依赖 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

## 三、应用配置文件优化

### 1. 添加环境配置文件

**问题**：缺少应用的环境配置文件。

**建议**：创建以下配置文件：
- application.yml：主配置文件
- application-dev.yml：开发环境配置
- application-docker.yml：Docker环境配置

### 2. 配置文件内容建议

**application-docker.yml**：

```yaml
spring:
  datasource:
    url: jdbc:mysql://mysql:3306/ai_jubensha
    username: root
    password: root
  redis:
    host: redis
  rabbitmq:
    host: rabbitmq
    username: jubensha
    password: jubensha123
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true

milvus:
  host: milvus
  port: 19530

elasticsearch:
  host: elasticsearch
  port: 9200
```

## 四、其他建议

### 1. 添加健康检查

**建议**：在docker-compose.yml中为各服务添加健康检查，确保服务正常运行。

### 2. 添加日志管理

**建议**：添加ELK或其他日志管理方案，便于排查问题。

### 3. 完善CI/CD配置

**建议**：添加GitHub Actions或其他CI/CD配置，实现自动化构建和部署。

### 4. 添加监控系统

**建议**：添加Prometheus和Grafana等监控工具，监控系统运行状态。

## 总结

通过以上优化，可以使项目的配置更加完整和合理，提高系统的可靠性和可维护性。特别是添加应用服务的Docker配置后，整个系统可以通过docker-compose一键启动，方便开发和部署。