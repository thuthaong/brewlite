# ADR 003: Cơ chế Idempotent Payment và Optimistic Locking cho Tồn kho

- **Trạng thái:** Đã chấp nhận (Accepted)
- **Ngày:** 2026-10

## 1. Bối cảnh (Context)
Trong môi trường mạng di động không ổn định và giờ cao điểm tại quán cà phê, có hai rủi ro lớn:
1. Khách hàng nhấn nút "Xác nhận trả" nhiều lần hoặc mạng retry dẫn đến thanh toán trùng lặp.
2. Nhiều khách hàng đặt cùng lúc một món chỉ còn số lượng ít dẫn đến bán vượt tồn kho (`stock < 0`).

## 2. Quyết định (Decision)
1. **Thanh toán Idempotent:** Bắt buộc sử dụng khóa `Idempotency-Key` (lưu `UNIQUE` trong bảng `Payment`). Nếu nhận lại key đã tồn tại, hệ thống trả về kết quả cũ ngay lập tức.
2. **Kiểm soát đồng thời (Concurrency):** Sử dụng **Optimistic Locking** thông qua cột `version` trên bảng `Product` bên trong Database Transaction khi trừ `stock`.

## 3. Hệ quả & Đánh đổi (Consequences)
- **Tích cực:**
  - Chặn đứng hoàn toàn lỗi trừ tiền 2 lần và lỗi âm kho mà không làm chậm hệ thống bởi cơ chế khóa bảng dài hạn (Pessimistic Locking).
- **Tiêu cực:**
  - Khi xảy ra xung đột `version`, request đến sau sẽ nhận lỗi `409 Conflict` và khách hàng cần xác nhận lại.