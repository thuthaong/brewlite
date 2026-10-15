# Đặc tả Yêu cầu Phần mềm Rút gọn (Lightweight SRS) – BrewLite

## 1. Quy ước Mức độ Ưu tiên (MoSCoW)
- **Must (M):** Yêu cầu bắt buộc phải có để hệ thống hoạt động và đạt tiêu chuẩn MVP / nghiệm thu.
- **Should (S):** Yêu cầu quan trọng, nâng cao tính chặt chẽ của nghiệp vụ thực tế (Task 10).
- **Could (C):** Yêu cầu cải thiện trải nghiệm người dùng nếu còn thời gian.
- **Won't (W):** Yêu cầu không thực hiện trong phạm vi phiên bản hiện tại.

---

## 2. Yêu cầu Chức năng (Functional Requirements - FR)

| Mã YC | Nhóm chức năng | Mô tả yêu cầu | MoSCoW | Tiêu chí chấp nhận có thể kiểm chứng (Testable Acceptance Criteria) |
| :-: | :--- | :--- | :-: | :--- |
| **FR-01** | **Menu** | Hệ thống hiển thị danh sách (menu) đồ uống kèm hình ảnh, tên, giá cơ bản và trạng thái còn hàng. | **Must** | - `GET /products` trả về HTTP 200 và mảng JSON các sản phẩm (`id`, `name`, `price`, `imageUrl`, `stock`).<br>- Giao diện Next.js hiển thị lưới sản phẩm chuẩn mobile-first.<br>- Hiển thị đúng các trạng thái UI: *Loading* (khi đang tải), *Empty* (khi danh sách rỗng) và *Error* (khi API lỗi). |
| **FR-02** | **Chi tiết sản phẩm** | Khách hàng xem chi tiết một đồ uống, chọn kích cỡ (Size S/M/L) và Topping kèm theo, hệ thống tự động tính lại đơn giá. | **Must** | - `GET /products/:id` trả về HTTP 200 với chi tiết sản phẩm (hoặc HTTP 404 nếu không tồn tại).<br>- Khi chọn Size (ví dụ: S +0đ, M +5.000đ, L +10.000đ) hoặc Topping (Trân châu, Kem...), giá hiển thị trên màn hình cập nhật tức thì theo công thức: `Giá cơ bản + Phụ thu Size + Tổng giá Topping`. |
| **FR-03** | **Giỏ hàng** | Khách hàng có thể thêm món (kèm tùy chọn), tăng/giảm số lượng, xóa món khỏi giỏ hàng và xem tổng tiền tạm tính. | **Must** | - Chọn cùng sản phẩm và cùng tổ hợp (Size + Topping) sẽ cộng dồn số lượng; khác tổ hợp sẽ tách thành dòng (line item) riêng.<br>- Thay đổi số lượng hoặc xóa món sẽ cập nhật ngay `Tổng tiền` và `Badge số lượng` trên thanh điều hướng.<br>- Trạng thái giỏ hàng được lưu giữ qua Zustand/Context (không bị mất khi chuyển trang). |
| **FR-04** | **Xác thực (Auth)** | Khách hàng có thể đăng ký tài khoản mới và đăng nhập để nhận JWT phục vụ đặt đơn và tích điểm. | **Must** | - `POST /auth/register` kiểm tra email hợp lệ, chưa tồn tại, băm mật khẩu bằng `bcrypt` và lưu `User` mới.<br>- `POST /auth/login` trả về HTTP 200 kèm `accessToken` (JWT) khi đúng thông tin; trả về HTTP 401 khi sai.<br>- Các API đặt đơn/thanh toán/lịch sử đơn không có JWT hợp lệ sẽ bị `JwtAuthGuard` chặn với mã HTTP 401. |
| **FR-05** | **Đặt đơn hàng** | Khách hàng đã đăng nhập có thể gửi giỏ hàng để tạo đơn hàng mới ở trạng thái `PENDING`. | **Must** | - `POST /orders` validate dữ liệu đầu vào bằng `class-validator` (danh sách món không rỗng, `qty >= 1`).<br>- Tính toán chính xác `lineTotal` cho từng `OrderItem` và `total` của `Order` trên server (không tin tưởng giá gửi từ client).<br>- Lưu đơn hàng với trạng thái khởi tạo là `PENDING` và trả về mã đơn hàng (`orderId`). |
| **FR-06** | **Kiểm soát tồn kho đồng thời** | Hệ thống kiểm tra và trừ tồn kho (`stock`) an toàn khi có nhiều khách hàng đặt mua cùng một thời điểm. | **Should** | - Khi tạo đơn, hệ thống kiểm tra `product.stock >= qty`. Nếu không đủ, trả về HTTP 400/409 và không tạo đơn.<br>- Sử dụng Database Transaction kết hợp **Optimistic Locking** (`version`): nếu 2 request cùng mua món chỉ còn `stock = 1`, chỉ 1 request thành công, request còn lại bị từ chối và rollback an toàn (không bao giờ để `stock < 0`). |
| **FR-07** | **Thanh toán không tiền mặt** | Khách hàng chọn phương thức thanh toán (Ví điện tử hoặc Thẻ ngân hàng) và thực hiện thanh toán cho đơn hàng. | **Must** | - Khi giỏ hàng có ít nhất 1 sản phẩm, nút "Thanh toán" hiển thị tổng tiền.<br>- `POST /payments` nhận `orderId`, `method` (`WALLET` / `CARD`).<br>- Nếu thanh toán thành công: trạng thái đơn chuyển sang `PAID`, hiển thị màn hình xác nhận kèm mã đơn.<br>- Nếu thanh toán thất bại: trạng thái đơn chuyển sang `PAYMENT_FAILED`, giữ nguyên giỏ hàng để khách thử lại. |
| **FR-08** | **Thanh toán Idempotent** | Hệ thống đảm bảo các yêu cầu thanh toán gửi lặp lại với cùng một `Idempotency-Key` không làm trừ tiền hoặc xử lý thanh toán 2 lần. | **Should** | - `POST /payments` yêu cầu truyền `Idempotency-Key` (qua Header hoặc Body).<br>- Gửi 2 request liên tiếp có cùng `Idempotency-Key` cho cùng một đơn hàng chỉ tạo ra duy nhất 1 bản ghi `Payment` trong DB và trả về cùng một kết quả phản hồi. |
| **FR-09** | **Lịch sử & Theo dõi đơn** | Khách hàng xem màn hình xác nhận đơn vừa đặt (mã đơn, trạng thái) và danh sách lịch sử các đơn hàng của bản thân. | **Must** | - Màn hình xác nhận hiển thị rõ Mã đơn (ví dụ `#1042`) và trạng thái hiện tại (`PAID`).<br>- `GET /orders/me` trả về danh sách các đơn hàng của đúng user đang đăng nhập, sắp xếp theo thời gian mới nhất (`createdAt DESC`). |
| **FR-10** | **Quản lý trạng thái đơn (State Machine)** | Hệ thống quản lý vòng đời đơn hàng theo máy trạng thái (Order State Machine) và chặn mọi chuyển tiếp không hợp lệ. | **Should** | - Chỉ cho phép các chuyển tiếp hợp lệ: `PENDING` -> `PAID` -> `PREPARING` -> `READY` -> `COMPLETED`; `PENDING` -> `PAYMENT_FAILED` / `CANCELLED`; `PAYMENT_FAILED` -> `PENDING` (thử lại) / `CANCELLED`.<br>- Hàm `assertTransition()` ném ngoại lệ (HTTP 400) đối với mọi chuyển tiếp sai (ví dụ: từ `PENDING` nhảy thẳng sang `READY` hoặc từ `COMPLETED` sang `CANCELLED`). |
| **FR-11** | **Khuyến mãi & Điểm thưởng (Loyalty)** | Khách hàng có thể áp mã giảm giá (`Promotion`) khi đặt đơn và được cộng điểm tích lũy (`Loyalty`) sau khi thanh toán thành công. | **Could** | - Khi truyền `promoCode` hợp lệ (còn hạn, đủ điều kiện) vào `POST /orders`, tổng tiền đơn được giảm đúng theo cấu hình mã.<br>- Khi đơn hàng chuyển sang trạng thái `PAID`, hệ thống tự động cộng `loyaltyPoints` cho `User` và ghi nhận lịch sử vào bảng `LoyaltyTransaction`. |
| **FR-12** | **Giao diện điều phối cho Barista** | Giao diện thời gian thực riêng cho nhân viên quầy nhận đơn và bấm chuyển trạng thái pha chế trên màn hình quầy. | **Won't** | - Không hiện thực giao diện UI dành riêng cho Barista trong phạm vi MVP (việc chuyển trạng thái được kiểm chứng qua Backend API và Unit/Integration Test). |

---

## 3. Yêu cầu Phi Chức năng (Non-Functional Requirements - NFR)

| Mã YC | Nhóm chất lượng | Mô tả yêu cầu | MoSCoW | Tiêu chí chấp nhận có thể kiểm chứng (Testable Acceptance Criteria) |
| :-: | :--- | :--- | :-: | :--- |
| **NFR-01** | **Hiệu năng (Performance)** | Thời gian phản hồi của các REST API chính phải nhanh và ổn định. | **Must** | - Thời gian phản hồi trung bình của các API (`GET /products`, `POST /orders`, `POST /payments`) **nhỏ hơn 500ms** khi chạy với bộ dữ liệu mẫu. |
| **NFR-02** | **Bảo mật (Security)** | Đảm bảo an toàn mật khẩu, xác thực API, kiểm tra đầu vào và bảo mật thông tin cấu hình. | **Must** | - 100% mật khẩu lưu trong DB được băm bằng `bcrypt` (không lưu plain text).<br>- Mọi DTO đầu vào ở NestJS đều được kiểm tra qua `ValidationPipe` (`class-validator`).<br>- Không có bất kỳ khóa bí mật (DB password, JWT secret) nào bị lộ trong Git repository hoặc log CI/CD.<br>- Production chạy trên giao thức `HTTPS` và Database không public không kiểm soát. |
| **NFR-03** | **Tính khả dụng (Usability)** | Giao diện tối ưu cho thiết bị di động (Mobile-first / PWA) và tương thích đa trình duyệt. | **Must** | - Giao diện hiển thị tốt không bị vỡ khung trên kích thước màn hình điện thoại (375px - 430px) và máy tính.<br>- Có đầy đủ thông báo trạng thái *Loading*, *Empty*, *Error* rõ ràng cho người dùng. |
| **NFR-04** | **Khả năng triển khai & Vận hành (Deployability)** | Hệ thống được đóng gói bằng Docker, có kịch bản Migration/Seed tự động, có CI/CD và Health Check. | **Must** | - Chạy lệnh `docker compose up` ở local khởi động thành công cả 3 thành phần (`web`, `api`, `postgres`).<br>- Endpoint `GET /health` trả về HTTP 200 kèm trạng thái hệ thống.<br>- Pipeline CI/CD tự động chạy `Lint -> Test -> Build -> Deploy -> Smoke Test`. |
| **NFR-05** | **Khả năng kiểm thử & Bảo trì (Testability)** | Mã nguồn phân tách module rõ ràng, có đầy đủ các cấp độ kiểm thử tự động. | **Must** | - Có đầy đủ Unit Test (State Machine, nghiệp vụ), Integration Test (Idempotency, Concurrency tồn kho) và ít nhất 1 End-to-End (E2E) Test từ xem menu đến xác nhận đơn. |

---

## 4. Sơ đồ Use-case Tổng quát của Hệ thống (System Use-Case Diagram)

### 4.1. Danh sách Tác nhân (Actors) và Use-cases
- **Khách hàng (Customer):**
  - `UC-01`: Xem danh sách sản phẩm (Menu) – *liên kết `FR-01`*
  - `UC-02`: Xem chi tiết & Chọn tùy chọn sản phẩm (Size/Topping) – *liên kết `FR-02`*
  - `UC-03`: Quản lý giỏ hàng (Thêm/Sửa/Xóa/Tính tổng) – *liên kết `FR-03`*
  - `UC-04`: Đăng ký / Đăng nhập tài khoản – *liên kết `FR-04`*
  - `UC-05`: Đặt và thanh toán đơn hàng (Use-case trọng tâm) – *liên kết `FR-05`, `FR-06`, `FR-07`, `FR-08`, `FR-11`*
  - `UC-06`: Xem xác nhận đơn & Lịch sử đơn hàng – *liên kết `FR-09`*
- **Cổng thanh toán (Mock Payment Service - Secondary Actor):**
  - Tham gia xử lý giao dịch trong `UC-05`.
- **Nhân viên quầy / Hệ thống quản lý đơn (Barista / Order System):**
  - `UC-07`: Cập nhật trạng thái đơn hàng theo State Machine – *liên kết `FR-10`*