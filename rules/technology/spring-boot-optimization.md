---
description: Rules for optimizing Spring Boot microservices for low-resource environments (VPS).
---

# Spring Boot Resource Optimization

All new microservices in the DTS system MUST adhere to these resource optimization guidelines to ensure they can run effectively on a memory-constrained VPS.

## 1. Application Configuration (`application.yaml`)

Add the following to every `application.yaml` by default:

```yaml
server:
  tomcat:
    threads:
      max: 50 # Giới hạn luồng Tomcat, mặc định là 200, giảm xuống 50 để tiết kiệm RAM.

spring:
  main:
    lazy-initialization: true # Tải lười, chỉ khởi tạo Bean khi cần thiết, giúp khởi động nhanh và ít tốn RAM.
  jmx:
    enabled: false # Tắt JMX vì không sử dụng trên môi trường VPS nhỏ.
```

## 2. Docker & JVM Environment (`docker-compose.prod.yml`)

When writing `docker-compose.prod.yml`:
- **mem_limit:** Set to `200m` (or maximum `320m` for Kafka-heavy services like media).
- **JAVA_TOOL_OPTIONS:** Must include `-Xms64m -Xmx128m -Xquickstart`.
  > [!WARNING]
  > **Lưu ý (Trường hợp bất khả kháng):** Hiện tại cờ `-Xquickstart` trên OpenJ9 có thể gây lỗi tăng vọt CPU (>80%) do JIT compiler bị quá tải trong môi trường VPS cực nhỏ. Khi gặp tình trạng này, giải pháp chữa cháy bắt buộc là đổi sang: `-Xms64m -Xmx128m -XX:+UseSerialGC -XX:MaxMetaspaceSize=128m -Xss512k`. Đây chỉ là phương án bất khả kháng, không phải chủ đích thiết kế kiến trúc ban đầu.
- **Environment Variables:** Use underscores `_`, NEVER hyphens `-` (e.g. use `SPRING_KAFKA_BOOTSTRAP_SERVERS`, not `SPRING_KAFKA_BOOTSTRAP-SERVERS`).

## 3. Dockerfile
- Always use `ibm-semeru-runtimes:open-21-jre AS runtime` instead of `eclipse-temurin` or `alpine` to utilize OpenJ9's aggressive memory management.
- Always use standard Linux user commands: `RUN groupadd -r dts && useradd -r -g dts -s /bin/false dts`.
