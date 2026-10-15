# BrewLite – Ứng dụng Đặt Cà Phê Không Tiền Mặt

Bài tập lớn môn Công nghệ Phần mềm – Trường Đại học Sài Gòn (Học kỳ I – Năm học 2026–2027).

## 1. Giới thiệu
**BrewLite** giúp khách hàng đặt và thanh toán đồ uống không dùng tiền mặt chỉ trong vài chạm, giảm thời gian xếp hàng và nhận đơn nhanh tại quầy.

## 2. Công nghệ bắt buộc
- **Frontend:** Next.js (Mobile-first / PWA), TypeScript, TailwindCSS, Zustand
- **Backend:** NestJS (REST API, class-validator, Passport JWT)
- **Database:** PostgreSQL
- **DevOps:** Docker Compose, GitHub Actions (CI/CD)

## 3. Cấu trúc Repository
```text
brewlite/
|-- apps/
|   |-- web/                  # Next.js mobile-first/PWA
|   `-- api/                  # NestJS REST API
|-- docs/
|   |-- requirements/         # Task 1: Context, Scope, SRS, Use-case
|   |-- diagrams/             # Task 2: Activity, Sequence, Class, Deployment Diagrams
|   |-- architecture/         # Task 3: Architecture Overview & ADRs
|   |-- traceability-matrix.md
|   `-- deployment.md
|-- tests/                    # Integration & E2E Tests
|-- .github/workflows/        # CI/CD Pipelines
|-- docker-compose.yml
|-- .env.example
`-- README.md