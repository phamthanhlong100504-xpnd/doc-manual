# Agent Customizations (`.agents/`)

Thư mục này chứa các thiết lập tùy chỉnh (Customizations) dành cho AI Coding Assistant nhằm hướng dẫn Agent tuân thủ nghiêm ngặt các quy chuẩn thiết kế, kiến trúc và quy trình nghiệp vụ của dự án.

---

## 1. Cấu trúc thư mục

Thư mục `.agents/` được tổ chức thành 2 phần chính:

### `/rules/` (Bộ Quy Tắc)
Chứa các quy tắc chuẩn hóa code và thiết kế hệ thống, phân chia theo các nhóm công nghệ và kiến trúc:
*   **`architecture/`**: Định nghĩa chuẩn kiến trúc (Clean Architecture, DDD, Event-driven, Microservices, Modular Monolith).
*   **`docx/`**: Quy chuẩn sinh tài liệu thiết kế API Blueprint (`docx/java/api-blueprint-generator.md`).
*   **`global/`**: Bộ quy tắc nền tảng áp dụng chung (Đặt tên `NAME-xxx`, API `API-xxx`, Database, Security, Error Handling, Logging, Git...).
*   **`infrastructure/`**: Tiêu chuẩn cấu hình và kết nối hạ tầng (PostgreSQL, Docker, Kubernetes, Kafka, Redis...).
*   **`technology/`**: Tiêu chuẩn viết code của ngôn ngữ và framework (Java Core `JAVA-xxx`, JPA/Hibernate `PERSIST-xxx`, Spring Boot, Testing).
*   **`templates/`**: Bản mẫu (Template) cấu trúc package và các class chuẩn (Controller, Service, Entity, Mapper...).

### `/workflows/` (Quy Trình Làm Việc)
Chứa 13 quy trình hướng dẫn chi tiết từng bước cho Agent khi thực hiện các tác vụ phát triển phần mềm cụ thể.

---

## 2. Các Tính Năng & Workflows được hỗ trợ

Bộ quy trình này giúp Agent phối hợp nhịp nhàng với bộ quy tắc thông qua các workflow từ đầu đến cuối:

1.  **Quản lý Quy Tắc (`00_select_rules.md`)**: Tự động xác định và tải đúng bộ quy tắc cần thiết cho từng tác vụ nhằm tối ưu ngữ cảnh.
2.  **Đọc Hiểu Mã Nguồn (`01_read_code.md`)**: Hướng dẫn phân tích cấu trúc dự án hiện tại dựa trên các quy tắc kiến trúc và đặt tên.
3.  **Làm Rõ Yêu Cầu (`02_understand_requirement.md`)**: Phân tích yêu cầu nghiệp vụ từ khách hàng mà không tự ý phỏng đoán, kiểm tra tính hợp lệ của thiết kế API/Database.
4.  **Thiết Kế API Blueprint (`03_generate_blueprint.md`)**: Sinh tài liệu thiết kế API độc lập với ngôn ngữ (tuân thủ `API-BLUEPRINT-xxx`).
5.  **Sinh Code Tự Động (`04_generate_code.md`)**: Sinh code Java Spring Boot chuẩn chỉ theo các template, áp dụng quy tắc Lombok, JPA, clean architecture.
6.  **Đánh Giá Mã Nguồn (`05_review_code.md`)**: Review code và phát hiện các lỗi vi phạm, bắt buộc chỉ ra mã quy tắc cụ thể (ví dụ: `PERSIST-028a`).
7.  **Tái Cấu Trúc Code (`06_refactor_code.md`)**: Cải tiến chất lượng mã nguồn một cách an toàn mà không làm thay đổi hành vi nghiệp vụ.
8.  **Tìm và Sửa Lỗi (`07_debug_issue.md` & `12_bug_fix.md`)**: Định vị nhanh root cause, kiểm tra xem có vi phạm quy tắc không và sửa lỗi an toàn.
9.  **Viết Test Tự Động (`08_generate_test.md`)**: Sinh các test case (Unit, Integration, Controller) toàn diện theo chuẩn JUnit 5.
10. **Tài Liệu Hóa (`09_generate_documentation.md`)**: Tạo tài liệu kỹ thuật khớp 100% với code thực tế.
11. **Thiết Kế Kiến Trúc (`10_design_architecture.md`)**: Thiết kế module/service và đánh giá rủi ro kiến trúc.
12. **Phát Triển Tính Năng Khép Kín (`11_feature_development.md`)**: Bộ điều phối (Coordinator) chạy xuyên suốt từ Phân tích yêu cầu → Viết Blueprint → Sinh code → Viết Test → Review.

---

## 3. Cách Sử Dụng Dành Cho Lập Trình Viên

Khi bạn trò chuyện với AI Coding Assistant trong workspace này:
1.  **Tự động nhận diện:** Agent sẽ tự động phát hiện thư mục `.agents/` này làm Workspace Customizations Root và tải các quy tắc.
2.  **Kích hoạt Workflow:** Bạn có thể yêu cầu Agent chạy một workflow cụ thể bằng cách gõ tên hoặc gọi lệnh (ví dụ: *"Hãy dùng workflow 11_feature_development để phát triển tính năng X"*).
3.  **Tuân thủ quy chuẩn:** Agent sẽ luôn thực hiện **Step 0 — Load Rules** của mỗi workflow để đọc các file quy tắc trước khi tạo ra bất kỳ thay đổi nào trên code của bạn.
