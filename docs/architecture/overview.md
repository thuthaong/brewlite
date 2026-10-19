# Thiết kế Kiến trúc Hệ thống (System Architecture) – BrewLite

## 1. Phương pháp Tiếp cận Kiến trúc
Hệ thống **BrewLite** được thiết kế kết hợp giữa mô hình **C4 (System Context & Container)** ở cấp độ tổng thể và **Kiến trúc Phân lớp theo Module (Modular Layered Architecture)** ở cấp độ ứng dụng:
- **Tổ chức mã nguồn:** Monorepo quản lý tập trung Frontend (`apps/web`), Backend (`apps/api`), Tài liệu (`docs`) và Kiểm thử (`tests`).
- **Kiến trúc Backend:** Modular Monolith trên nền tảng **NestJS**, chia tách rõ ràng theo từng miền nghiệp vụ (Domain Modules) và phân lớp (`Controller` -> `Service / Domain Logic` -> `Repository / ORM`).
- **Kiến trúc Frontend:** Ứng dụng Web theo hướng **Mobile-first / PWA** sử dụng **Next.js (App Router)**, kết hợp Server Components (tải danh mục sản phẩm nhanh) và Client State (**Zustand**) cho giỏ hàng.

---

## 2. Chi tiết các Thành phần Kiến trúc (Containers & Modules)

### 2.1. Frontend – Next.js Mobile-first / PWA (`apps/web`)
- **Vai trò:** Cung cấp giao diện người dùng tối ưu cho màn hình di động, cho phép khách hàng thực hiện trọn vẹn luồng: *Xem menu -> Chọn tùy chọn -> Giỏ hàng -> Đặt đơn -> Thanh toán -> Theo dõi đơn*.
- **Cấu trúc chính:**
  - `app/`: Các route trang chính (`/`, `/products/[id]`, `/cart`, `/login`, `/register`, `/checkout`, `/orders/[id]`, `/orders/me`).
  - `components/`: Các UI component tái sử dụng kèm xử lý 3 trạng thái bắt buộc: *Loading*, *Empty*, *Error*.
  - `store/`: Quản lý trạng thái Giỏ hàng (`cartStore` bằng Zustand có lưu trữ `localStorage`) và trạng thái phiên đăng nhập (`authStore`).
  - `lib/api.ts`: HTTP Client tập trung tự động đính kèm header `Authorization: Bearer <JWT>` và sinh `Idempotency-Key` (UUID v4) khi thanh toán.

### 2.2. Backend – NestJS Modular API (`apps/api`)
Backend được chia thành các Module độc lập, dễ kiểm thử (Unit/Integration test):
1. **`ProductsModule`:** Cung cấp `GET /products` và `GET /products/:id`, quản lý thông tin đồ uống và tồn kho (`stock`, `version`).
2. **`AuthModule`:** Xử lý `POST /auth/register`, `POST /auth/login`, băm mật khẩu bằng `bcrypt`, phát hành JWT và cung cấp `JwtAuthGuard`.
3. **`OrdersModule`:** Xử lý `POST /orders`, `GET /orders/me`, tích hợp bộ máy trạng thái **Order State Machine** (`assertTransition`) và kiểm soát trừ tồn kho đồng thời bằng **Optimistic Locking** trong Database Transaction.
4. **`PaymentsModule`:** Xử lý `POST /payments`, kiểm tra khóa chống trùng lặp **`Idempotency-Key`**, giao tiếp với `MockPaymentService` và kích hoạt cộng điểm thưởng.
5. **`PromotionsModule` & `LoyaltyModule`:** Kiểm tra tính hợp lệ của mã giảm giá (`Promotion`) và ghi nhận giao dịch điểm tích lũy (`LoyaltyTransaction`) khi đơn hàng chuyển sang `PAID`.
6. **`HealthModule`:** Cung cấp endpoint `GET /health` để kiểm tra trạng thái sống của API và kết nối PostgreSQL (phục vụ Docker healthcheck và Smoke Test trong CI/CD).

### 2.3. Cơ sở dữ liệu – PostgreSQL & Cơ chế Migration
- **Hệ quản trị CSDL:** PostgreSQL 16 (lưu trữ bền vững qua Persistent Volume).
- **ORM & Migration:** Sử dụng Prisma / TypeORM quản lý schema chặt chẽ.
  - Mọi thay đổi cấu trúc bảng (`users`, `products`, `orders`, `order_items`, `payments`, `promotions`, `loyalty_transactions`) đều được quản lý qua các file **Migration** có kiểm soát phiên bản trên Git.
  - Cung cấp kịch bản **Seed Data** tự động khởi tạo danh mục đồ uống mẫu và mã khuyến mãi để môi trường Local/CI/Production luôn sẵn sàng hoạt động.

### 2.4. Dịch vụ Thanh toán Mô phỏng (Mock Payment Service)
- Được đóng gói thành một Adapter/Service độc lập bên trong Backend (`MockPaymentService`), mô phỏng cổng thanh toán Ví điện tử (`WALLET`) và Thẻ ngân hàng (`CARD`).
- Hỗ trợ truyền tham số/kịch bản giả lập cả kết quả thành công (`SUCCESS` -> đơn chuyển sang `PAID`) và thất bại (`FAILED` -> đơn chuyển sang `PAYMENT_FAILED`, giữ nguyên giỏ hàng) để kiểm thử đầy đủ các nhánh nghiệp vụ.

### 2.5. Logging, Health Check và Quản lý Cấu hình
- **Quản lý cấu hình (Configuration):** Sử dụng `@nestjs/config` đọc biến môi trường từ `.env` (ở local) hoặc CI/Cloud Secrets (ở production). Tuyệt đối không hardcode secret trong mã nguồn.
- **Logging:** Sử dụng Logger tích hợp của NestJS ghi nhận request/response, lỗi ngoại lệ (`HttpExceptionFilter`) và lịch sử chuyển trạng thái đơn hàng.
- **Health Check:** Endpoint `GET /health` thực hiện truy vấn nhẹ (`SELECT 1`) xuống PostgreSQL và trả về HTTP `200 OK` `{ status: "ok", db: "up" }`.

---

## 3. Lý do Chọn Kiến trúc, Ưu điểm, Hạn chế và Đánh đổi (Trade-offs)

| Tiêu chí | Đánh giá chi tiết |
| :--- | :--- |
| **Lý do lựa chọn** | Phù hợp tối đa với quy mô nhóm sinh viên thực hiện qua 2 Sprint: vừa đảm bảo tính chặt chẽ về mặt kỹ thuật (TypeScript toàn trình từ Frontend xuống Backend), vừa đơn giản hóa việc đóng gói Docker và triển khai CI/CD. |
| **Ưu điểm (Pros)** | 1. **Đồng nhất ngôn ngữ (TypeScript):** Dễ dàng chia sẻ kiểu dữ liệu (DTO/Interfaces) giữa Next.js và NestJS.<br>2. **Độ tin cậy giao dịch cao:** Nhờ chung một CSDL PostgreSQL, việc kết hợp tạo đơn, trừ kho Optimistic Locking và cộng điểm Loyalty có thể thực hiện gọn trong ACID Database Transaction mà không cần Distributed Transaction (Saga/2PC) phức tạp.<br>3. **Triển khai & Vận hành gọn nhẹ:** Chỉ cần quản lý 2 container ứng dụng (`web`, `api`) và 1 CSDL (`postgres`). |
| **Hạn chế (Cons)** | 1. Khi toàn bộ các module nằm chung trong một tiến trình NestJS (Modular Monolith), nếu một module tiêu tốn quá nhiều CPU sẽ ảnh hưởng đến các module còn lại.<br>2. Build time của Monorepo có thể tăng lên nếu không cấu hình cache hợp lý trong CI. |
| **Đánh đổi (Trade-offs)** | **Chọn Modular Monolith thay vì Microservices:** Nhóm chấp nhận đánh đổi khả năng scale độc lập từng dịch vụ nhỏ để đổi lấy **tốc độ phát triển nhanh, tính toàn vẹn giao dịch ACID tuyệt đối cho tồn kho/thanh toán** và chi phí vận hành/triển khai bằng 0 trên các nền tảng Cloud PaaS (Render/Vercel). |