# ADR 001: Tổ chức mã nguồn theo cấu trúc Monorepo

- **Trạng thái:** Đã chấp nhận (Accepted)
- **Ngày:** 2026-10

## 1. Bối cảnh (Context)
Dự án BrewLite bao gồm ứng dụng Frontend (Next.js), Backend API (NestJS), tài liệu kỹ thuật/UML (`docs`) và bộ kiểm thử tích hợp/E2E (`tests`). Nhóm cần một phương thức quản lý mã nguồn giúp đồng bộ hóa tài liệu, mã nguồn và pipeline CI/CD xuyên suốt 5 Task tuần tự.

## 2. Quyết định (Decision)
Sử dụng một **Git Repository duy nhất (Monorepo)** với cấu trúc thư mục tách biệt: `apps/web`, `apps/api`, `docs`, `tests`, và `.github/workflows`.

## 3. Hệ quả & Đánh đổi (Consequences)
- **Tích cực:**
  - Dễ dàng truy vết (traceability) từ yêu cầu (`docs/requirements`) -> sơ đồ (`docs/diagrams`) -> code (`apps/`) -> kiểm thử (`tests/`) trong cùng một Pull Request.
  - Chạy toàn bộ hệ thống ở local chỉ với một file `docker-compose.yml` duy nhất.
- **Tiêu cực / Giảm thiểu:**
  - Pipeline CI có thể chạy lâu hơn nếu kiểm tra cả hai app; giảm thiểu bằng cách cấu hình cache `node_modules` trên GitHub Actions.