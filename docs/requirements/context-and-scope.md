# Bối cảnh, Stakeholder và Phạm vi Dự án BrewLite

## 1. Bối cảnh và Vấn đề Nghiệp vụ (Business Problem)
Tại các cửa hàng cà phê hiện nay, vào khung giờ cao điểm, khách hàng thường xuyên phải xếp hàng dài tại quầy để xem menu, gọi món, tùy chỉnh đồ uống (chọn kích cỡ, topping) và chờ thanh toán tiền mặt hoặc chờ nhân viên thao tác máy POS. Quy trình thủ công này gây ra các vấn đề:
- **Đối với khách hàng:** Mất nhiều thời gian chờ đợi, trải nghiệm mua hàng bị gián đoạn, bất tiện khi không mang theo tiền mặt và không chủ động theo dõi được tiến độ thực hiện đơn hàng.
- **Đối với cửa hàng:** Tắc nghẽn tại khu vực quầy thu ngân, dễ xảy ra sai sót khi ghi nhận tùy chọn đồ uống hoặc kiểm soát tồn kho khi có nhiều khách đặt cùng lúc.

**Tầm nhìn sản phẩm (Product Vision):**
> *"BrewLite giúp khách hàng đặt và thanh toán đồ uống không dùng tiền mặt chỉ trong vài chạm, giảm thời gian xếp hàng và nhận đơn nhanh tại quầy."*

---

## 2. Danh sách các bên liên quan (Stakeholders)

| STT | Stakeholder | Vai trò trong hệ thống | Mục tiêu (Goals) | Kỳ vọng (Expectations) |
| :-: | :--- | :--- | :--- | :--- |
| 1 | **Khách hàng (Customer)** | Người dùng cuối trực tiếp sử dụng ứng dụng web/PWA | Đặt đồ uống nhanh chóng, tùy chọn size/topping linh hoạt, thanh toán không tiền mặt và nhận nước đúng hẹn. | Giao diện mobile-first mượt mà, tính đúng tổng tiền & khuyến mãi, thanh toán an toàn (không bị trừ tiền 2 lần khi bấm lặp), theo dõi được mã đơn và trạng thái đơn hàng. |
| 2 | **Nhân viên quầy / Barista** | Người vận hành tại cửa hàng (Nằm ngoài phạm vi UI của MVP) | Tiếp nhận thông tin đơn hàng đã thanh toán (`PAID`) và cập nhật tiến độ pha chế. | Đơn hàng chỉ chuyển vào khâu pha chế khi đã thanh toán hợp lệ và còn đủ nguyên liệu/tồn kho; luồng trạng thái đơn rõ ràng (`PAID` -> `PREPARING` -> `READY` -> `COMPLETED`). |
| 3 | **Quản lý cửa hàng / Product Owner (PO)** | Đại diện nghiệp vụ và chủ sở hữu sản phẩm | Tối ưu hóa quy trình bán hàng, quản lý danh mục sản phẩm, khuyến mãi và giữ chân khách hàng qua tích điểm (Loyalty). | Product Backlog được ưu tiên rõ ràng theo giá trị kinh doanh; mỗi Sprint tạo ra Increment chạy được; dữ liệu tồn kho và doanh thu chính xác. |
| 4 | **Giảng viên / Khách hàng nghiệm thu** | Người đánh giá và nghiệm thu dự án | Kiểm tra tính tuân thủ quy trình Công nghệ Phần mềm từ đặc tả yêu cầu, thiết kế, lập trình đến CI/CD và triển khai Production. | Truy vết đầy đủ từ Yêu cầu -> Sơ đồ UML -> Code/API -> Test; pipeline CI/CD hoạt động thực tế; có URL Production (HTTPS) hoạt động ổn định. |
| 5 | **Nhóm phát triển (Development Team)** | Đội ngũ kỹ thuật (BA, Designer, Frontend, Backend, QA, DevOps) | Phân tích, thiết kế kiến trúc, hiện thực hóa mã nguồn, kiểm thử tự động và bàn giao hệ thống đúng hạn qua 2 Sprint. | Yêu cầu rõ ràng, tiêu chí chấp nhận (Acceptance Criteria) có thể kiểm chứng được bằng Unit/Integration/E2E test; kiến trúc Monorepo dễ bảo trì. |
| 6 | **Đơn vị thanh toán (Payment Service)** | Hệ thống bên thứ ba (Được mô phỏng qua Mock Payment Gateway) | Xử lý giao dịch thanh toán không dùng tiền mặt qua Ví điện tử hoặc Thẻ ngân hàng. | Giao tiếp qua API chuẩn xác, hỗ trợ cơ chế `Idempotency-Key` chống trùng lặp giao dịch, trả về kết quả thành công (`PAID`) hoặc thất bại (`PAYMENT_FAILED`) rõ ràng. |

---

## 3. Phạm vi Sản phẩm (Product Scope)

| Phân loại phạm vi | Hạng mục chức năng / Kỹ thuật | Mô tả chi tiết |
| :--- | :--- | :--- |
| **Trong phạm vi MVP (In-Scope MVP)** | **1. Danh mục & Chi tiết sản phẩm** | Hiển thị danh sách (menu) đồ uống kèm hình ảnh, tên, giá, trạng thái tồn kho; xem chi tiết từng món và chọn tùy chọn Size (S/M/L), Topping (Trân châu, Kem...) kèm tính giá động. |
| | **2. Giỏ hàng (Cart)** | Thêm sản phẩm vào giỏ, tăng/giảm số lượng, xóa món, hiển thị badge số lượng và tự động tính tổng tiền tạm tính trên Client (Zustand/Context). |
| | **3. Xác thực người dùng (Auth)** | Đăng ký tài khoản mới, đăng nhập bằng Email/Mật khẩu (băm bằng `bcrypt`), cấp phát và xác thực bằng JWT bảo vệ các route đặt đơn/thanh toán. |
| | **4. Đặt đơn hàng (Order)** | Tạo đơn hàng từ giỏ hàng ở trạng thái `PENDING`, kiểm tra hợp lệ dữ liệu đầu vào (`class-validator`). |
| | **5. Thanh toán không tiền mặt** | Tích hợp cổng thanh toán giả lập (Mock Payment Gateway) hỗ trợ chọn phương thức **Ví điện tử** hoặc **Thẻ ngân hàng**; cập nhật trạng thái đơn thành `PAID` (nếu thành công) hoặc `PAYMENT_FAILED` (nếu thất bại, đồng thời giữ nguyên giỏ hàng). |
| | **6. Xác nhận & Lịch sử đơn** | Màn hình xác nhận đơn hàng thành công (hiển thị mã đơn `#ID`, trạng thái) và trang xem lịch sử các đơn hàng đã đặt của người dùng (`GET /orders/me`). |
| **Phạm vi mở rộng (Extended Scope - Nghiệp vụ Backend nâng cao)** | **1. Order State Machine** | Quản lý và kiểm soát chặt chẽ luồng chuyển trạng thái hợp lệ: `PENDING` -> `PAID` -> `PREPARING` -> `READY` -> `COMPLETED` (hoặc nhánh ngoại lệ `PAYMENT_FAILED`, `CANCELLED`). Chặn mọi chuyển tiếp sai quy tắc bằng hàm `assertTransition`. |
| | **2. Thanh toán Idempotent** | Sử dụng `Idempotency-Key` trong yêu cầu thanh toán để đảm bảo khi người dùng hoặc mạng gửi lặp lại cùng một yêu cầu thì không bị trừ tiền hoặc tạo giao dịch thanh toán 2 lần. |
| | **3. Kiểm soát tồn kho đồng thời** | Sử dụng Database Transaction kết hợp **Optimistic Locking** để đảm bảo khi nhiều khách hàng đặt mua cùng lúc không xảy ra tình trạng bán vượt quá số lượng tồn kho (`stock`) còn lại. |
| | **4. Khuyến mãi & Điểm thưởng (Loyalty)** | Áp dụng mã giảm giá (`Promotion`) hợp lệ khi tạo đơn hàng và tự động cộng điểm tích lũy (`loyaltyPoints`, `LoyaltyTransaction`) cho khách hàng sau khi đơn chuyển sang trạng thái `PAID`. |
| **Ngoài phạm vi (Out-of-Scope)** | **1. Giao diện Barista / Admin Dashboard** | Không xây dựng giao diện màn hình riêng cho nhân viên pha chế (Barista) hoặc trang quản trị (Admin CRUD sản phẩm) trên Frontend trong giai đoạn MVP. |
| | **2. Cổng thanh toán thương mại thật** | Không kết nối trực tiếp với tài khoản ngân hàng thật hay cổng MoMo/VNPay/Stripe thương mại có đối soát tài chính thực tế (chỉ dùng Mock Payment Service). |
| | **3. Giao hàng tận nơi (Delivery)** | Không bao gồm tính năng định vị GPS, tính phí vận chuyển hay điều phối tài xế giao hàng (ứng dụng phục vụ mô hình nhận nước tại quầy - Pick up at counter). |

---

## 4. Giả định, Ràng buộc và Rủi ro ban đầu

### 4.1. Giả định (Assumptions)
- **A-01:** Khách hàng sử dụng thiết bị di động hoặc máy tính cá nhân có kết nối Internet và trình duyệt hiện đại hỗ trợ JavaScript, LocalStorage/SessionStorage.
- **A-02:** Tất cả khách hàng nhận đồ uống trực tiếp tại quầy cửa hàng sau khi đặt và thanh toán thành công trên ứng dụng.
- **A-03:** Cổng thanh toán được mô phỏng nội bộ (Mock Payment Service) cho phép giả lập cả hai kịch bản: giao dịch thành công và giao dịch thất bại để phục vụ kiểm thử và nghiệm thu.

### 4.2. Ràng buộc (Constraints)
- **C-01 (Công nghệ bắt buộc):** Hệ thống bắt buộc phải sử dụng **Next.js** (Frontend theo hướng mobile-first/PWA), **NestJS** (Backend REST API), **PostgreSQL** (Cơ sở dữ liệu quan hệ) và **Docker** (`docker-compose.yml`). Không được thay thế ba thành phần cốt lõi này.
- **C-02 (Cấu trúc mã nguồn):** Tổ chức theo mô hình Monorepo gồm `apps/web`, `apps/api`, `docs`, `tests` và `.github/workflows`.
- **C-03 (Quy trình tuần tự):** 5 Task của dự án phải được thực hiện tuần tự và có lịch sử commit/Pull Request rõ ràng trên Git.
- **C-04 (Triển khai thực tế):** Hệ thống cuối cùng phải được triển khai lên môi trường Production có URL truy cập qua giao thức `HTTPS`, tách biệt cấu hình qua biến môi trường (Secrets) và Database không được public trực tiếp ra ngoài mà không có bảo mật.

### 4.3. Rủi ro ban đầu và Biện pháp giảm thiểu (Initial Risks)

| Mã | Rủi ro | Mức độ | Biện pháp giảm thiểu (Mitigation) |
| :-: | :--- | :-: | :--- |
| **R-01** | **Xung đột tồn kho (Race Condition)** khi nhiều khách hàng cùng đặt một món chỉ còn số lượng ít cùng thời điểm. | Cao | Áp dụng Database Transaction kết hợp **Optimistic Locking** (kiểm tra cột `version` của bảng `Product`) và viết Integration Test kiểm chứng. |
| **R-02** | **Gửi trùng yêu cầu thanh toán (Double Charging)** do mạng chập chờn hoặc người dùng nhấn nút "Xác nhận trả" nhiều lần. | Cao | Vô hiệu hóa nút bấm ở UI khi đang loading (`isSubmitting`) kết hợp bắt buộc truyền `Idempotency-Key` ở Backend API `POST /payments`. |
| **R-03** | **Lỗi môi trường khi triển khai Production** (sai lệch giữa máy local và cloud khiến URL Production không hoạt động lúc chấm bài). | Trung bình | Chuẩn hóa môi trường bằng `Dockerfile` và `docker-compose.yml`, thiết lập Smoke Test tự động gọi `GET /health` ngay trong pipeline CD và chuẩn bị sẵn kịch bản Rollback. |