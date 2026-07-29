# Environment Variables and Port Allocations

This configuration dictates the standardized ports and environment setup for all `dts` and related microservices on the VPS (`103.75.182.249`). To prevent port collisions, all services must adhere to this port configuration.

## Standardized Ports
- **Media Service (`media-service`)**: 8080
- **Identity Service (`identity-service`)**: 8081
- **Content Builder (`content-builder`)**: 8082
- **API Gateway (`gateway`)**: 8083 (Dự kiến)
- **LMS Core (`lms-core`)**: 8084 (Dự kiến)
- **CMS Request Service (`cms-request-service`)**: 8085 (Dự kiến)
- **Common Info Service (`common-info-service`)**: 8086 (Dự kiến)

## External / Infrastructure Services
- **PostgreSQL (`postgres`)**: 5434
- **MinIO (`media-minio`)**: 9000 (API), 9001 (Console)
- **Redis (`redis`)**: 6379
- **Kafka (`media-kafka`)**: 9092
- **Zookeeper (`media-zookeeper`)**: 2181, 2888, 3888

## Rules
- When generating code or deploying, ensure the `server.port` matches the list above.
- In `application.yaml` or `application.properties`, explicitly define the port.
- Any new service must be assigned a port in the `8080+` range that is not currently listed above, and this document must be updated to reflect the new allocation.
