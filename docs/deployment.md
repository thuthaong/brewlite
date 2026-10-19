# Tài liệu Kiến trúc Triển khai (Deployment Specification) – BrewLite

## 1. Sơ đồ Triển khai (UML Deployment Diagram)
- File nguồn: [docs/diagrams/deployment.puml](./diagrams/deployment.puml)
- Hình ảnh:

![Deployment Diagram](./diagrams/deployment.svg)

---

## 2. Phân định Ranh giới các Môi trường (Environment Boundaries)

| Môi trường | Thành phần chính | Đặc điểm vận hành & Bảo mật |
| :--- | :--- | :--- |
| **1. Local (Máy phát triển)** | `docker-compose.yml` chạy 3 container: `web` (`:3000`), `api` (`:4000`), `postgres` (`:5432`). | Sử dụng tệp `.env` nội bộ (tạo từ `.env.example`), tự động chạy migration và seed dữ liệu mẫu để phát triển và chạy Integration/E2E test. |
| **2. CI/CD (GitHub Actions & GHCR)** | GitHub Actions Runner + GitHub Container Registry (`ghcr.io`). | Tự động chạy `Lint -> Unit/Integration Test -> Build Docker Image` khi mở PR hoặc push lên `main`. Lưu trữ Docker Image gắn tag theo commit SHA (`sha-xxxx`) hoặc phiên bản (`v1.0.0`). |
| **3. Production (Cloud PaaS - Render / Railway)** | - **Web Service:** Next.js (`HTTPS :443 -> :3000`)<br>- **API Service:** NestJS (`HTTPS :443 -> :4000`)<br>- **Database:** Managed PostgreSQL (`:5432`) | - Giao tiếp bên ngoài bắt buộc qua `HTTPS`.<br>- PostgreSQL nằm trong mạng nội bộ (Private Network), gắn **Persistent Storage Volume** và không public không kiểm soát.<br>- Cấu hình nhạy cảm (`DATABASE_URL`, `JWT_SECRET`) được tiêm qua biến môi trường bảo mật (Encrypted Secrets). |

---

## 3. Truy vết Thành phần Triển khai tới Kiến trúc & Mã nguồn

| Node / Container trên Deployment Diagram | Thành phần Kiến trúc | Đường dẫn trong Repository | Giao thức / Cổng |
| :--- | :--- | :--- | :--- |
| `Edge CDN / Web Service` (`Next.js`) | Frontend Web / PWA | `apps/web/` | `HTTPS (443)` -> Container Port `3000` |
| `Application Container` (`NestJS API`) | Backend Modular Monolith | `apps/api/` | `HTTPS (443)` -> Container Port `4000` |
| `Health Check & Logger` | Health Module | `apps/api/src/health/` | `GET /health` (HTTP 200 OK) |
| `Managed PostgreSQL 16` | Relational Database | `apps/api/prisma/` (hoặc `migrations/`) | Nội bộ `TCP (5432)` |
| `CI/CD Workflow` | Continuous Integration & Delivery | `.github/workflows/` | GitHub Actions Webhook / HTTPS |
| `Docker Compose Engine` | Local Orchestration | `docker-compose.yml` | Localhost Ports `3000`, `4000`, `5432` |