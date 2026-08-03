# API Gateway Configuration & Patterns

Tài liệu này ghi nhận lại các thiết kế cốt lõi của API Gateway trong hệ sinh thái DTS nhằm làm kim chỉ nam cho các lần nâng cấp hoặc phát triển service mới.

## 1. Static Routing (Định tuyến Tĩnh)
Để tránh việc phải sửa đổi mã nguồn hoặc cấu hình của các service cũ, Gateway không sử dụng Service Discovery (như Eureka). Thay vào đó, Gateway đóng vai trò Reverse Proxy tĩnh.
- **Port Gateway:** Toàn bộ hệ thống giao tiếp qua một port duy nhất của Gateway là `8888`.
- **Cấu hình:** Ánh xạ các service trực tiếp vào file `application.yaml` của Gateway thông qua URI `localhost:<port_cua_service>`.

## 2. Authentication Pass-Through (Xác thực Phân tán)
Nhằm bảo toàn 100% logic của các microservice cũ (đã được cấu hình tự giải mã token):
- **Gateway:** Vô hiệu hóa (comment `@Component`) bộ lọc `AuthenticationFilter`. Gateway KHÔNG trực tiếp đọc hay giải mã JWT token.
- **Microservices:** Các service con (như `identity-service`, `content-builder`, v.v.) sẽ tự tiếp nhận Header `Authorization: Bearer <token>` từ Gateway chuyển xuống và dùng `JwtAuthenticationFilter` cục bộ của chính nó để giải mã.
- Lợi ích: Khắc phục lỗi **Double-Decoding** (Gateway giải mã một lần, Service giải mã lần hai gây lãng phí tài nguyên) và không làm hỏng code của các service cũ.

## 3. Traceability (Theo dấu Request)
Để tuân thủ tiêu chuẩn `OBS-018` (Không làm đứt gãy vết của TraceID):
- **Tại Gateway:** `TraceIdFilter` sẽ sinh ra một Trace ID duy nhất cho mỗi Request mới và đưa vào Header mang tên **`X-Request-ID`**.
- **Tại Microservice:** Các service bên dưới hứng header `X-Request-ID` này thông qua `MdcTraceIdFilter` và lưu vào `MDC` để in ra trong mọi dòng log `%X{traceId}`.
- Lợi ích: Dù request đi lạc qua nhiều service khác nhau, bạn luôn có thể lọc theo mã `X-Request-ID` trên Kibana hoặc Console để đọc luồng xử lý hoàn chỉnh.
