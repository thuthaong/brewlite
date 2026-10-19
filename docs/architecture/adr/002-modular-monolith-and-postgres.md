# ADR 002: Kiến trúc Backend Modular Monolith với NestJS và PostgreSQL

- **Trạng thái:** Đã chấp nhận (Accepted)
- **Ngày:** 2026-10

## 1. Bối cảnh (Context)
Hệ thống cần xử lý các nghiệp vụ có tính ràng buộc dữ liệu chặt chẽ: tạo đơn hàng, kiểm tra và trừ tồn kho, thanh toán và tích điểm thưởng với thời gian phản hồi mục tiêu nhỏ hơn 500ms.

## 2. Quyết định (Decision)
Xây dựng Backend theo kiến trúc **Modular Monolith** bằng framework **NestJS** kết nối tới một cơ sở dữ liệu quan hệ **PostgreSQL** duy nhất, quản lý cấu trúc bảng thông qua cơ chế **Migration**.

## 3. Hệ quả & Đánh đổi (Consequences)
- **Tích cực:**
  - Tận dụng trực tiếp ACID Transaction của PostgreSQL để đảm bảo tính toàn vẹn dữ liệu giữa `Order`, `OrderItem`, `Product` và `Payment`.
  - Phân tách rõ các module (`Products`, `Auth`, `Orders`, `Payments`) giúp dễ dàng viết Unit Test và Integration Test độc lập.
- **Tiêu cực:**
  - Không thể mở rộng (scale) riêng lẻ từng module ở mức hạ tầng container như Microservices, nhưng hoàn toàn đáp ứng vượt mức tải của bài toán MVP.