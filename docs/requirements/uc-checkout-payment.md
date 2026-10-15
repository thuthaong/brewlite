# Đặc tả Use-Case Trọng tâm: Đặt và Thanh toán Đơn hàng (UC-05)

## 1. Bảng Đặc tả Chi tiết Use-Case (Use-Case Specification)

| Thuộc tính | Nội dung chi tiết |
| :--- | :--- |
| **Mã Use-Case** | **UC-05** |
| **Tên Use-Case** | Đặt và thanh toán đơn hàng không dùng tiền mặt (Place Order and Cashless Payment) |
| **Mã Yêu cầu liên kết** | `FR-04`, `FR-05`, `FR-06`, `FR-07`, `FR-08`, `FR-10`, `FR-11`, `NFR-01`, `NFR-02` (Tương ứng User Story `US-05`) |
| **Tác nhân chính (Primary Actor)** | Khách hàng (Customer) |
| **Tác nhân phụ (Secondary Actor)** | Cổng thanh toán giả lập (Mock Payment Gateway / Service) |
| **Mô tả tóm tắt** | Khách hàng xác nhận giỏ hàng, tạo đơn hàng mới (có kiểm soát tồn kho và áp dụng khuyến mãi nếu có), sau đó thực hiện thanh toán không tiền mặt qua Ví điện tử hoặc Thẻ ngân hàng để hoàn tất đơn hàng và nhận mã đơn. |
| **Sự kiện kích hoạt (Trigger)** | Khách hàng nhấn nút **"Thanh toán"** tại màn hình Giỏ hàng khi trong giỏ có ít nhất 01 sản phẩm. |
| **Tiền điều kiện (Pre-conditions)** | 1. Giỏ hàng của khách hàng có ít nhất 01 sản phẩm hợp lệ (`qty >= 1`).<br>2. Khách hàng đã đăng nhập và có `JWT Access Token` hợp lệ (nếu chưa, hệ thống sẽ kích hoạt luồng ngoại lệ yêu cầu đăng nhập).<br>3. Hệ thống Backend, Database PostgreSQL và Mock Payment Service đang hoạt động bình thường. |
| **Hậu điều kiện khi thành công (Success Post-conditions)** | 1. Một bản ghi `Order` kèm danh sách `OrderItem` được lưu trong CSDL và chuyển từ trạng thái `PENDING` sang `PAID`.<br>2. Số lượng tồn kho (`stock`) của các sản phẩm tương ứng được trừ chính xác và tăng `version` (Optimistic Locking).<br>3. Bản ghi `Payment` được lưu kèm `idempotencyKey`, số tiền `amount` và phương thức `method` (`WALLET` hoặc `CARD`).<br>4. Điểm tích lũy (`loyaltyPoints`) của `User` được cộng thêm và lưu lịch sử vào bảng `LoyaltyTransaction`.<br>5. Giỏ hàng trên Client được làm rỗng; khách hàng nhìn thấy màn hình Xác nhận đơn hàng cùng mã đơn (`#orderId`). |
| **Hậu điều kiện khi thất bại (Failure Post-conditions)** | 1. **Nếu lỗi ở bước tạo đơn (hết hàng / dữ liệu không hợp lệ):** Transaction được rollback hoàn toàn, tồn kho không bị trừ, không có đơn hàng rác được tạo.<br>2. **Nếu lỗi ở bước thanh toán:** Đơn hàng chuyển sang trạng thái `PAYMENT_FAILED`, **giỏ hàng của khách hàng được giữ nguyên** để có thể thử thanh toán lại hoặc chỉnh sửa. |

---

## 2. Các Luồng Sự kiện (Flows of Events)

### 2.1. Luồng chính (Main Success Scenario / Happy Path)
1. **Khách hàng** kiểm tra danh sách món và tổng tiền tại màn hình Giỏ hàng, sau đó nhấn nút **"Thanh toán"**.
2. **Hệ thống (Frontend Next.js)** kiểm tra trạng thái đăng nhập (JWT) và chuyển sang màn hình **Thanh toán** (hiển thị tổng tiền, ô nhập mã khuyến mãi và các phương thức thanh toán: *Ví điện tử* / *Thẻ ngân hàng*).
3. **Khách hàng** chọn phương thức thanh toán (ví dụ: *Ví điện tử*) và nhấn nút **"Xác nhận trả"**.
4. **Hệ thống (Frontend)** vô hiệu hóa nút bấm (hiển thị trạng thái loading), sinh mã `Idempotency-Key` (UUID) và gửi yêu cầu tạo đơn `POST /orders` kèm danh sách sản phẩm và JWT lên **NestJS API**.
5. **Hệ thống (Backend NestJS)** xác thực JWT (`JwtAuthGuard`), kiểm tra tính hợp lệ dữ liệu đầu vào (`class-validator`).
6. **Hệ thống (Backend)** mở một **Database Transaction**:
   - Đọc thông tin từng `Product` trong giỏ hàng kèm cột `version`.
   - Kiểm tra số lượng tồn kho (`stock >= qty`).
   - Cập nhật trừ `stock = stock - qty` với điều kiện `WHERE id = :id AND version = :currentVersion` (Optimistic Locking).
   - Tính toán `lineTotal` cho từng món, tổng tiền `total` và lưu bản ghi `Order` ở trạng thái `PENDING` cùng các `OrderItem`.
   - Commit Transaction và trả về thông tin đơn hàng (`orderId`, `total`, `status = PENDING`).
7. **Hệ thống (Frontend)** nhận `orderId` và tiếp tục gọi API `POST /payments` kèm `orderId`, `method`, và header `Idempotency-Key`.
8. **Hệ thống (Backend)** kiểm tra `Idempotency-Key` chưa từng tồn tại, kiểm tra trạng thái đơn hợp lệ qua Order State Machine (`PENDING` -> `PAID`), và gọi sang **Mock Payment Service**.
9. **Mock Payment Service** xử lý giao dịch thành công và trả về kết quả `SUCCESS`.
10. **Hệ thống (Backend)** lưu bản ghi `Payment`, cập nhật trạng thái `Order` thành `PAID`, tính điểm thưởng cộng vào `User.loyaltyPoints`, tạo bản ghi `LoyaltyTransaction` và trả kết quả HTTP 200/201 về cho Frontend.
11. **Hệ thống (Frontend)** xóa giỏ hàng tạm thời và chuyển khách hàng đến màn hình **Xác nhận đơn hàng** hiển thị: thông báo *"Đặt hàng thành công"*, Mã đơn (ví dụ `#1042`) và Trạng thái `PAID`. Use-case kết thúc.

---

### 2.2. Luồng thay thế (Alternative Flows)

- **AF-01: Khách hàng áp dụng mã khuyến mãi hợp lệ (Tại bước 2–3)**
  - `2a.` Trước khi nhấn xác nhận, khách hàng nhập mã giảm giá (`promoCode`) tại màn hình Thanh toán.
  - `2b.` Tại bước 6, Backend kiểm tra bảng `Promotion`: mã tồn tại, đang hoạt động (`isActive = true`), chưa hết hạn và tổng đơn đạt mức tối thiểu.
  - `2c.` Hệ thống khấu trừ số tiền giảm giá vào `Order.total`, ghi nhận mã khuyến mãi vào đơn hàng và tiếp tục quay lại **Bước 6** của Luồng chính.

- **AF-02: Yêu cầu thanh toán bị gửi lặp lại do lỗi mạng / nhấn đúp (Tại bước 7–8 – Idempotency)**
  - `8a.` Hệ thống Backend phát hiện `Idempotency-Key` gửi lên đã tồn tại trong bảng `Payment`.
  - `8b.` Hệ thống **không** gọi lại Mock Payment Service, **không** trừ tiền hay cộng điểm thưởng lần 2, mà lập tức trả về nguyên vẹn kết quả của giao dịch đã lưu trước đó. Luồng tiếp tục sang **Bước 11** bình thường.

---

### 2.3. Luồng ngoại lệ (Exception Flows)

- **EF-01: Khách hàng chưa đăng nhập hoặc phiên JWT hết hạn (Tại bước 2 hoặc bước 5)**
  - `2a / 5a.` Hệ thống phát hiện khách hàng chưa có JWT hoặc nhận mã lỗi `401 Unauthorized` từ Backend.
  - `2b.` Hệ thống giữ nguyên dữ liệu giỏ hàng hiện tại và chuyển hướng khách hàng sang màn hình **Đăng nhập / Đăng ký**.
  - `2c.` Sau khi khách hàng đăng nhập thành công, hệ thống tự động đưa khách hàng quay trở lại màn hình Thanh toán để tiếp tục **Bước 2**.

- **EF-02: Sản phẩm hết hàng hoặc xung đột tồn kho khi đặt đồng thời (Tại bước 6)**
  - `6a.` Trong quá trình kiểm tra tồn kho hoặc thực hiện Optimistic Locking (`version` đã bị thay đổi bởi một giao dịch khác vừa mua hết món đó), hệ thống phát hiện `stock < qty` hoặc số dòng cập nhật bằng `0`.
  - `6b.` Hệ thống thực hiện `ROLLBACK` toàn bộ Database Transaction, trả về lỗi HTTP `409 Conflict` / `400 Bad Request` kèm thông báo tên sản phẩm không đủ số lượng.
  - `6c.` Frontend hiển thị thông báo lỗi rõ ràng và đưa khách hàng về lại Giỏ hàng để điều chỉnh số lượng. Use-case kết thúc không tạo đơn.

- **EF-03: Giao dịch thanh toán không thành công (Tại bước 9)**
  - `9a.` Mock Payment Service trả về kết quả thất bại (ví dụ: số dư ví không đủ, thẻ bị từ chối hoặc quá thời gian chờ).
  - `9b.` Hệ thống Backend ghi nhận trạng thái thanh toán lỗi, sử dụng Order State Machine chuyển trạng thái đơn hàng từ `PENDING` sang `PAYMENT_FAILED`.
  - `9c.` Frontend hiển thị thông báo *"Thanh toán không thành công"*, **giữ nguyên giỏ hàng** và cho phép khách hàng chọn **"Thử lại"** (sinh `Idempotency-Key` mới để thanh toán lại từ `PAYMENT_FAILED`) hoặc **"Hủy đơn"** (chuyển đơn sang `CANCELLED`).

---

## 3. Liên kết Tiêu chí Kiểm thử (Testability Link)
1. **Unit Test (`UT-SM-01`):** Kiểm tra State Machine cho phép `PENDING -> PAID`, `PENDING -> PAYMENT_FAILED` và chặn `PENDING -> COMPLETED`.
2. **Integration Test (`INT-PAY-01` - Idempotency):** Gửi 2 request `POST /payments` cùng `Idempotency-Key` -> Chỉ tạo 1 bản ghi `Payment` và cộng điểm Loyalty 1 lần.
3. **Integration Test (`INT-ORD-02` - Concurrency):** Giả lập 2 request đặt mua đồng thời 1 sản phẩm chỉ còn `stock = 1` -> 1 thành công (`201`), 1 thất bại (`409/400`), `stock` cuối cùng bằng `0`.
4. **E2E Test (`E2E-CHK-01`):** Kiểm thử tự động toàn bộ Luồng chính từ Chọn món -> Giỏ hàng -> Đăng nhập -> Thanh toán -> Hiển thị mã đơn `#ID` và trạng thái `PAID`.