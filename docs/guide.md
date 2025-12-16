# 🕵️‍♂️ PROJECT loCI: THE ARCHITECT'S BLUEPRINT (SOTA EDITION)

> **Role:** Solution Architect & Tech Lead Support
> **Philosophy:** "Build for Scale, Learn for Depth" (Xây dựng để mở rộng, Học để hiểu sâu)
> **Stack Strategy:** Modular Monolith (dễ start -> dễ tách Microservices), Event-Driven Architecture (cho task nặng), API First.

---

## 🏗️ 1. TECHNOLOGY STACK (SOTA STANDARD)
Để đạt chuẩn "SOTA" và sẵn sàng scale cho AI/Blog sau này, stack sẽ được nâng cấp như sau:

| Layer | Technology | Why SOTA? |
|-------|------------|-----------|
| **Core Backend** | **Spring Boot 3.2+ (Java 21)** | Virtual Threads (Project Loom) cho high throughput, Native Image support. |
| **Frontend** | **Next.js 14+ (App Router)** | Server Components (RSC), Streaming UI, SEO optimized. |
| **Database** | **PostgreSQL 16** | Robust, hỗ trợ JSONB (cho NoSQL features), Vector extensions (cho AI sau này). |
| **Messaging** | **Kafka** (hoặc RabbitMQ) | **CRITICAL for Scale**: Tách biệt luồng xử lý code nặng (Python) khỏi luồng web chính. |
| **Caching** | **Redis** | High speed cache, Rate limiting, Pub/Sub cho realtime logs. |
| **Infra** | **Docker Compose** | Standardize dev environment (DB, Redis, Broker, Apps). |
| **Observability**| **OpenTelemetry + Grafana** | Tracing request từ FE -> BE -> Worker -> DB. Không code mò. |

---

## 📂 2. PROJECT STRUCTURE (Cấu Trúc Thư Mục)
Quy hoạch theo mô hình **Monorepo** giúp quản lý toàn bộ hệ thống trong một nơi, dễ dàng Debug và đồng bộ Data Contract (DTO/Types).

```text
loCI/
├── .github/                    # CI/CD Workflows (Github Actions)
├── apps/
│   ├── backend/                # Spring Boot (Modular Monolith)
│   │   ├── pom.xml             # Parent POM
│   │   └── src/main/java/com/loci/
│   │       ├── common/         # Shared Kernel (Global Execptions, Base Entity)
│   │       └── modules/        # 📦 FEATURE MODULES (Quan trọng)
│   │           ├── identity/   # Login, Register, Profile (User Domain)
│   │           ├── content/    # Course, Chapter, Lesson (Learning Domain)
│   │           └── judge/      # Submission, Runner Service (Execution Domain)
│   └── frontend/               # Next.js 14+
│       ├── src/
│       │   ├── app/            # File-system Routing
│       │   ├── components/     # UI Components (ShadCN, Radix)
│       │   ├── services/       # API Calls (Axios/Fetch)
│       │   └── types/          # TypeScript Intefaces (Sync với Backend DTO)
├── ops/                        # Infrastructure Configs
│   ├── docker/                 # Dockerfiles tùy chỉnh
│   │   ├── python-runner/      # Image chứa Python + Libraries để chạy code
│   │   └── postgres/           # Init scripts cho DB
│   └── grafana/                # Config dashboards
├── docs/                       # Tài liệu (Guide, API Spec, ERD)
├── docker-compose.yml          # Chạy toàn bộ stack (DB + API + FE) chỉ với 1 lệnh
└── .gitignore
```

---

## 🏛️ PHASE 1: THE FOUNDATION (Nền Móng Vững Chắc)
*Mục tiêu: Thiết lập môi trường chuẩn DevOps, Clean Architecture.*

### 🚀 Sprint 1: The Genesis (Khởi tạo chuẩn Architect)
* **Goal:** Repo setup, Docker environment, API Standard.
* **Dev/Ops:**
    *   Setup Monorepo: `apps/backend` (Maven Multi-module), `apps/frontend`, `ops/docker`.
    *   `docker-compose.yml`: Up Postgres, Redis, PGAdmin.
* **Backend:**
    *   Structure: **Hexagonal Architecture** (or Clean Architecture) simple version.
        *   `domain`: Core logic (không phụ thuộc Framework).
        *   `application`: Use cases.
        *   `infrastructure`: Spring configurations, Repository impl.
    *   Config **Checkstyle, Spotless**: Code phải đẹp chuẩn Google Java Style.
    *   **OpenAPI (Swagger):** Định nghĩa API trước khi code (API First).
* **Frontend:**
    *   Next.js clean setup, config `ESLint`, `Prettier`.
    *   Setup `TanStack Query` (React Query) quản lý server state (chuẩn hơn `useEffect` truyền thống).

---

## 📚 PHASE 2: THE KNOWLEDGE CORE (Lõi Kiến Thức - Blog/Course)
*Mục tiêu: Master Spring Data JPA, Security và Next.js SSR.*

### 🔐 Sprint 2: The Gatekeeper (Identity & Reform)
* **Goal:** Secure Access & Scalable User System.
* **Backend:**
    *   **Spring Security 6 (Lambda DSL)**: Config chuẩn SOTA.
    *   **Oauth2 Resource Server (Optional):** Nếu muốn scale lớn (tách Auth riêng keycloak), nhưng hiện tại dùng JWT + Refresh Token (HttpOnly Cookie) là đủ tốt.
    *   Rate Limiting (bucket4j): Chống spam API.
* **Frontend:**
    *   Sử dụng `NextAuth.js` hoặc custom hook xử lý JWT/Session.
    *   Middleware bảo vệ routes.

### 📖 Sprint 3: The Librarian (Content Management)
* **Goal:** CMS cho khóa học.
* **Backend:**
    *   Design DB tối ưu: `Course` (1-n) `Module` (1-n) `Lesson`.
    *   Dùng **Liquibase/Flyway** để quản lý version DB (Migration) - *Bắt buộc cho dự án chuyên nghiệp*.
* **Frontend:**
    *   **Server Side Rendering (SSR)** các trang chi tiết khóa học để SEO tốt.
    *   MDX Remote: Render nội dung bài học markdown xịn xò.

---

## ⚡ PHASE 3: THE EXECUTION ENGINE (Trái Tim Sức Mạnh - Nơi Khác Biệt)
*Mục tiêu: Xử lý tệp vụ nặng (Chạy code Python) mà không làm lag Web. Đây là bước nhảy vọt từ "WebApp thường" sang "System".*

### 🔄 Sprint 4: The Async Worker (Event-Driven) 🔥 *Critical Architect Shift*
* **Problem:** Nếu BE trực tiếp gọi Docker chạy code, khi có 1000 user submit, server sẽ treo (OOM).
* **Solution:** **Asynchronous Processing (Bất đồng bộ).**
* **Flow SOTA:**
    1.  User bấm "Run" -> FE gọi BE API `POST /submit`.
    2.  BE đẩy message vào **Kafka Topic** `submission_jobs` -> Trả về `submission_id` (Pending) ngay lập tức.
    3.  User nhận "Pending" -> FE polling hoặc lắng nghe WebSocket.
    4.  **Worker Service (Java/Go/Python):** Component riêng biệt, lắng nghe Kafka.
        *   Lấy job -> Khởi tạo Docker Container an toàn (Memory limit, CPU limit).
        *   Chạy code -> Capture Log.
        *   Đẩy kết quả vào queue `submission_results`.
    5.  BE Web nhận kết quả -> Save DB -> Bắn tin hiệu cho FE.

### 🎮 Sprint 5: The Interface (Realtime Experience)
* **Goal:** Trải nghiệm code realtime.
* **Frontend:**
    *   Monaco Editor.
    *   WebSocket Client: Nhận log realtime khi container đang chạy (nếu làm được streaming log).
* **Backend:**
    *   Tối ưu Worker: Pool container (Pre-warm containers) để giảm độ trễ khởi động.

---

## 🔭 PHASE 4: OBSERVABILITY & SCALE (Chế Độ God Mode)
*Mục tiêu: Scale và Monitor như các Big Tech.*

### 🔍 Sprint 6: The Watchtower (Giám Sát)
* **Stack:** Prometheus (Metrics), Grafana (Dashboard), Loki (Logs).
* **Action:**
    *   Spring Boot Actuator: Expose metrics.
    *   Gắn Dashboard: Xem CPU, RAM, Số lượng request/giây (RPS), Thời gian xử lý trung bình.
    *   **Distributed Tracing:** Dùng Micrometer Tracing + Zipkin để xem request đi qua bao nhiêu bước, chậm ở đâu (DB hay Worker?).

### 🤖 Sprint 7: Future Expansion (AI & Blog Integration)
* **Architecture Benefit:** Nhờ thiết kế Modular/Microservice từ đầu (chia Web và Worker), sau này muốn thêm AI:
    *   Tạo Service `AI-Tutor`.
    *   Service này subscribe Kafka event `lesson_completed` để gợi ý bài mới.
    *   Blog Service: Module riêng, chung Auth Database.

---

## 💡 LỜI KHUYÊN CHO NGƯỜI HỌC (LEARNING PATH)
Vì bạn mới có cơ bản JS/Java, đừng hoảng với đống công nghệ này. Hãy đi từng bước:
1.  **Just make it run (Sprint 0-2):** Code Monolith bình thường, gọi trực tiếp Docker cũng được để hiểu flow.
2.  **Make it right (Sprint 3-4):** Refactor tách Service, nhét Kafka vào giữa. Đây là lúc level up Architecture.
3.  **Make it fast (Sprint 5-6):** Thêm Caching, Tuning connection pool, Monitoring.

