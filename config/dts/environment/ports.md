---
name: ports
description: Rules and guidelines for ports
trigger: model_decision
---

This configuration dictates the standardized ports and environment setup for all `dts` and related microservices on the new **2-VPS Architecture**.

## Architecture Overview
- **Infra VPS (`103.20.96.85`) - 4GB RAM**: Databases, Storage, Message Broker, Identity, Media, Frontend.
- **App VPS (`103.20.96.56`) - 2GB RAM**: API Gateway and core application microservices.

## Standardized Ports
### Infra VPS (`103.20.96.85`)
- **Identity Service (`identity-service`)**: 8081
- **Media Service (`media-service`)**: 8080
- **Frontend Website (`dts-frontend`)**: 3000
- **PostgreSQL (`postgres`)**: 5434
- **MinIO (`media-minio`)**: 9000 (API), 9001 (Console)
- **Redis (`redis`)**: 6379
- **Kafka (`kafka`)**: 9092
- **Zookeeper (`media-zookeeper`)**: 2181, 2888, 3888

### App VPS (`103.20.96.56`)
- **API Gateway (`gateway`)**: 8888
- **Practice Service (`practice-service`)**: 8087
- **Examination Service (`examination-service`)**: 8088
- **Progress Service (`progress-service`)**: 8083
- **Result Service (`result-service`)**: 8086
- **Content Builder (`content-builder`)**: 8082

## Rules
- When generating code or deploying, ensure the `server.port` matches the list above.
- In `application.yaml` or `application.properties`, explicitly define the port.
- Any new service must be assigned a port in the `8080+` range that is not currently listed above, and this document must be updated to reflect the new allocation.
