---
description: Build tools and framework version compatibility constraints.
---

# Version Compatibility Constraints

## 1. Spring Boot & Gradle Compatibility
**Issue:** Lỗi `java.lang.Integer org.gradle.api.file.CopyProcessingSpec.getDirMode()` khi build `bootJar`.
**Nguyên nhân:** Spring Boot plugin bản 3.3.x (hoặc cũ hơn) không tương thích với Gradle 9.x do Gradle 9 đã thay đổi/xóa hàm `getDirMode()` trong plugin API của họ.
**Quy tắc BẮT BUỘC (Strict Rule):**
- Đối với tất cả các Microservices sử dụng **Spring Boot 3.3.x**.
- **TUYỆT ĐỐI KHÔNG** nâng cấp `gradle-wrapper.properties` lên phiên bản `9.x`.
- **Phiên bản Gradle tối đa được phép sử dụng:** `8.8` (hoặc các bản `8.x` ổn định khác).
- Phải luôn kiểm tra file `gradle/wrapper/gradle-wrapper.properties` trước khi thực hiện build hoặc xử lý lỗi build, đảm bảo `distributionUrl` trỏ về bản `8.x`.

## 2. Spring Boot & JVM Compatibility 
**Issue:** Lỗi `Unrecognized option: -Xquickstart` khiến container bị sập (Exited) ngay lập tức khi khởi động với CPU 0%, RAM 0B.
**Nguyên nhân:** Cờ `-Xquickstart` chỉ dành cho máy ảo **IBM Semeru (OpenJ9)**, không được hỗ trợ trên **Eclipse Temurin (HotSpot)**.
**Quy tắc BẮT BUỘC (Strict Rule):**
- Trừ khi Dockerfile sử dụng Base Image là `ibm-semeru-runtimes` (ví dụ trong quy trình CI/CD mẫu), thì mới được cấu hình cờ `-Xquickstart` trong `JAVA_TOOL_OPTIONS`.
- Nếu Service đang sử dụng Base Image là `eclipse-temurin` (HotSpot), **BẮT BUỘC** phải xóa bỏ cờ `-Xquickstart`.
- Bộ cờ tối ưu RAM tiêu chuẩn cho HotSpot JVM trên VPS nhỏ: `-XX:+UseSerialGC -XX:MaxMetaspaceSize=128m -Xss256k`.

## 3. Spring Boot Plugin Version Mismatch
**Issue:** Lỗi `Spring Boot plugin requires Gradle 8.x (8.14 or later) or 9.x. The current version is Gradle 8.8` khi build bằng CI/CD.
**Nguyên nhân:** Khai báo phiên bản `org.springframework.boot` quá cao (ví dụ `4.1.0`) trong `build.gradle`, dẫn đến việc Spring Boot yêu cầu Gradle đời mới (>= 8.14), nhưng hệ thống lại đang dùng Gradle 8.8 (để tương thích với Spring Boot 3.x).
**Quy tắc BẮT BUỘC (Strict Rule):**
- Luôn kiểm tra phiên bản Spring Boot trong khối `plugins { id 'org.springframework.boot' version '...' }`.
- Phiên bản Spring Boot **BẮT BUỘC** phải đồng bộ với các service khác trong hệ thống (hiện tại là `3.3.2` hoặc `3.3.4`).
- Tuyệt đối không dùng các phiên bản ảo tưởng/chưa phát hành chính thức như `4.1.0`.
