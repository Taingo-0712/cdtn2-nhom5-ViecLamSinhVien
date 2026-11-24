# CDTN2 Nhóm 5 — Nền tảng việc làm part-time cho sinh viên

**Track:** Software Engineering (SE) · **Phạm vi:** v4.1 · **Ngày cập nhật đặc tả:** 24/11/2026

Repository: [cdtn2-nhom5-ViecLamSinhVien](https://github.com/Taingo-0712/cdtn2-nhom5-ViecLamSinhVien)

## 1. Giới thiệu

Ứng dụng web kết nối sinh viên có nhu cầu làm thêm với nhà tuyển dụng. Điểm trọng tâm là gợi ý việc làm theo lịch rảnh cá nhân, hỗ trợ ca làm hằng tuần hoặc theo ngày cụ thể, ứng tuyển theo ca, xét duyệt ứng viên và quản lý vòng đời tuyển dụng.

Hệ thống có bốn nhóm người dùng:

| Vai trò | Chức năng chính trong phạm vi mục tiêu |
|---|---|
| Khách | Xem, tìm kiếm, lọc tin đủ điều kiện công khai; đăng ký, đăng nhập. |
| Sinh viên (STUDENT) | Hồ sơ, lịch rảnh, Matching, chọn ca và nộp CV, theo dõi đơn, thông báo, đánh giá. |
| Nhà tuyển dụng (EMPLOYER) | Hồ sơ, tạo/sửa tin theo điều kiện, quản lý ca, xem CV và xét duyệt ứng viên, đóng/hủy tin, xác nhận hoàn thành, đánh giá. |
| Quản trị viên (ADMIN) | Xem/lọc tài khoản, khóa/mở khóa, kiểm duyệt tin, quản lý danh mục; Dashboard cơ bản (SHOULD). |

## 2. Phạm vi v4.1

| Chỉ tiêu thiết kế | Số lượng |
|---|---:|
| User Story | 49 mã: 35 MUST, 1 SHOULD, 13 WON'T |
| Acceptance Criteria | 119 mã; 115 áp dụng (113 MUST, 2 SHOULD), 4 lịch sử |
| API | 38 mục tiêu: API-01–29 và API-34–42 |
| Database | 13 bảng nghiệp vụ, 107 trường mục tiêu; không tính `schema_migrations` |
| Đợt triển khai | 24 Dxx |

**Luồng chính:** NTD đăng ký hồ sơ thông thường → đăng nhập → tạo tin và ca → Admin kiểm duyệt → SV tìm/lọc hoặc nhận gợi ý theo lịch rảnh → ứng tuyển chọn ca và CV → NTD xét duyệt → thông báo → đóng tuyển → xác nhận hoàn thành → đánh giá hai chiều. Hủy và tự hết hạn là các nhánh nghiệp vụ riêng.

- Tin mới ở `RECRUITING` và `moderation_status=PENDING`; tin chỉ xuất hiện công khai, được Matching và nhận đơn khi đáp ứng điều kiện tương ứng trong đặc tả.
- Tài khoản NTD `ACTIVE`, đúng role `EMPLOYER` được tạo tin hợp lệ mà không cần xét duyệt hồ sơ. Admin vẫn kiểm duyệt tin.
- Matching theo quy tắc ANY/ALL, khớp trọn một ca trong một khoảng rảnh liên tục; hỗ trợ ca qua đêm, WEEKLY và ONE_TIME. Matching không phải AI và không chặn ứng tuyển chủ động.
- Mỗi SV tối đa một đơn cho mỗi tin; khóa toàn bộ sửa nội dung tuyển dụng sau khi có bất kỳ đơn nào. CV PDF tối đa 5 MiB.
- Khóa tài khoản chặn đăng nhập và API bảo vệ; `token_version` làm vô hiệu JWT cũ. Worker xử lý thời hạn khi Backend chạy; cập nhật trạng thái và thông báo nhất quán theo transaction.
- Dùng chung `job_categories` cho ngành nghề và lĩnh vực NTD; không tạo bảng `business_industries` riêng. Địa bàn sử dụng hai cấp tỉnh/thành và xã/phường/đặc khu, mã chuẩn dạng chuỗi.

**Ngoài MVP:** US23–34 và US46, gồm báo cáo/khiếu nại, yêu thích, thống kê riêng NTD, thanh toán, app native, chat realtime và xác minh NTD bằng giấy tờ. Không triển khai D15A, FR24/UC20 hay API-30–33; giữ mã lịch sử để truy vết. Không thực hiện chấm công, tính lương thực nhận hoặc kiểm chứng danh tính doanh nghiệp. Kiểm duyệt tin không thay thế xác minh danh tính.

## 3. Kiến trúc và công nghệ

Kiến trúc ba tầng: **React/Vite → REST API Express → PostgreSQL + File Storage**. Backend tổ chức theo Route, Middleware, Controller/Validator, Service, Repository; Worker gọi service xử lý thời hạn. Frontend không truy cập trực tiếp Database.

| Thành phần | Baseline theo tài liệu Công nghệ v4.1 |
|---|---|
| Frontend | React / React DOM 19.3.1; React Router DOM 7.18.5; Vite 8.3.6; plugin-react 6.2.1; JavaScript, CSS |
| Backend | Node.js 24.12.0; Express 5.3.0 (CommonJS); pg 8.23.1 |
| Database | PostgreSQL 16.15, schema `public` |
| Xác thực / upload | jsonwebtoken 9.0.3; bcryptjs 3.0.3; multer 2.5.0; cors 2.8.6 |
| Công cụ | npm 11.6.2; Git 2.50.1; pgAdmin; node:test; Postman |

Phiên bản package là baseline từ mã nguồn tham khảo, không khẳng định đã cài đặt trong repo mới. Kiểm tra `package.json`, lockfile và môi trường thực tế trước khi chạy; không tự nâng cấp thư viện hoặc đổi stack.

- Frontend: `http://localhost:5173`
- Backend: `http://localhost:8080`, REST prefix `/api`, health check `/health`
- PostgreSQL: `localhost:5432`
- Múi giờ nghiệp vụ: `Asia/Ho_Chi_Minh`

## 4. Cấu trúc repository

Cây thư mục mục tiêu (các app và file cấu hình sẽ có sau khi D00 khởi tạo và kiểm chứng):

```text
cdtn2-nhom5-ViecLamSinhVien/
├── README.md
├── .gitignore
├── backend/
│   ├── package.json / package-lock.json
│   ├── .env.example
│   ├── migrations/
│   ├── scripts/
│   ├── src/                     # routes, middleware, services, repositories...
│   └── tests/
├── frontend/
│   ├── package.json / package-lock.json
│   ├── .env.example
│   └── src/                     # pages, components, api, auth, styles...
└── docs/
    ├── SRS_Nhom5_v2.1_PhamVi_v4.1.docx
    ├── Product_Backlog_Nhom5_SE_v4.1_US01-US49.xlsx
    ├── api-contract-v4.1.md
    ├── API_Contract_day_du_Nhom5_v4.1.docx
    ├── database-schema-v4.1.md
    ├── Database_Schema_va_quy_trinh_cap_nhat_Nhom5_v4.1.docx
    ├── cong-nghe-cau-truc-v4.1.md
    ├── Cong_nghe_phien_ban_va_cau_truc_du_an_Nhom5_v4.1.docx
    ├── diagrams/
    ├── planning/
    └── testing/
```

Tên file/module chi tiết theo tài liệu Công nghệ và mã nguồn thực tế. Không coi cây mục tiêu là danh sách file đã tồn tại.

## 5. Tài liệu và thứ tự triển khai

| Nguồn | Nội dung |
|---|---|
| [SRS v2.1 / phạm vi v4.1](docs/SRS_Nhom5_v2.1_PhamVi_v4.1.docx) | US, FR, BR, AC và kịch bản nghiệm thu. |
| [Product Backlog v4.1](docs/Product_Backlog_Nhom5_SE_v4.1_US01-US49.xlsx) | Ưu tiên, truy vết và sheet `Roadmap Dxx`. |
| [Database Schema v4.1](docs/database-schema-v4.1.md) | Bảng, cột, khóa, ràng buộc, migration và transaction. |
| [API Contract v4.1](docs/api-contract-v4.1.md) | Endpoint, DTO, validation, phân quyền, lỗi. |
| [Công nghệ và cấu trúc v4.1](docs/cong-nghe-cau-truc-v4.1.md) | Stack, môi trường và cấu trúc triển khai. |
| [ERD v4.1](docs/diagrams/ERD_Nhom5_CDTN2_v4.1_US01-US49.drawio) | Sơ đồ dữ liệu. |
| [Use Case v4.1](docs/diagrams/UC_Diagram_Tong_The_Nhom5_v4.1_US01-US49.drawio) | Tác nhân, chức năng, truy vết. |
| [Kiến trúc v4.1](docs/diagrams/Kien_truc_tong_the_Nhom5_v4.1.drawio) | Kiến trúc hệ thống và luồng E2E. |

Các đường dẫn trên là bộ tài liệu đích sau khi đồng bộ vào workspace. Dùng Markdown thay cho bản Word trùng nội dung; vẫn tra SRS và Backlog theo yêu cầu liên quan. Không dùng tài liệu v4.0 để ghi đè quyết định v4.1. Sprint Plan v4.0 tạm hoãn cập nhật, chỉ lưu lịch sử; **Roadmap Dxx trong Product Backlog v4.1 là nguồn thứ tự triển khai hiện hành**.

| Giai đoạn | Thứ tự Dxx |
|---|---|
| Nền tảng | D00 |
| Tài khoản, hồ sơ | D07C → D01 → D02 → D03 → D04 |
| Quản trị | D15 → D15C |
| Tin tuyển dụng | D05 → D07A → D15B → D06 → D07 |
| Matching, ứng tuyển | D07B → D08 → D09 → D10 → D11 |
| Hoàn thiện nghiệp vụ | D12 → D13 → D14 → D15D (SHOULD) |
| Nghiệm thu | D16 → D17 |

D01, D02, D04, D05, D11 có các checkpoint nền hoặc kiểm thử hồi quy sau khi phụ thuộc được tích hợp. Không đánh dấu hoàn thành toàn bộ đợt chỉ vì phần nền đã chạy. D02 chuẩn bị `account_status`, `token_version` trước D03; D03 chuẩn bị `job_categories.is_active` trước API-29; D15/D15C tái sử dụng migration tương ứng.

## 6. Cài đặt và chạy thử (Windows / PowerShell)

> **Trạng thái tại D00:** repo mới được kiểm kê mới có tài liệu, chưa có bằng chứng ứng dụng build/test thành công. Các lệnh cài/chạy bên dưới là quy trình mục tiêu, chỉ thực hiện sau khi có mã nguồn, lockfile, cấu hình và DB đích đã kiểm chứng. Không chạy migration để thử trên DB nguồn.

### 6.1. Chuẩn bị repository và môi trường

Nếu chưa có bản clone local:

```powershell
git clone https://github.com/Taingo-0712/cdtn2-nhom5-ViecLamSinhVien.git
Set-Location .\cdtn2-nhom5-ViecLamSinhVien
```

Nếu đã clone, mở đúng thư mục hiện có, không clone thêm. Kiểm tra:

```powershell
git status --short --branch
git rev-parse HEAD
git log -1 --oneline
node -v
npm -v
git --version
psql --version
```

Nếu `psql` chưa có trong PATH, kiểm tra PostgreSQL bằng pgAdmin hoặc đường dẫn `bin` thực tế. Không đồng nghĩa PostgreSQL chưa được cài.

### 6.2. Database — bảo toàn nguồn

| Database | Vai trò |
|---|---|
| `student_jobs_nhom5` | Nguồn cũ, chỉ kiểm kê/backup; không ghi, reset, migrate hay seed. |
| `student_jobs_nhom5_v4` | Đích phát triển v4.1, sao chép bằng backup/restore đã được kiểm chứng. |

D00 phải kiểm kê cấu trúc, checksum, số dòng, FK và storage từ nguồn; backup bằng pgAdmin/pg_dump, restore vào DB đích riêng khi được phép và kiểm tra kết quả. Không coi bản sao là đã tồn tại khi chưa có log. CV lưu ngoài DB cần sao lưu riêng và kiểm tra metadata. Giữ nguyên V001–V006 đã ghi nhận; đối chiếu file SQL với `schema_migrations` trước khi chọn số migration mới. V007 chỉ là dự kiến. Không tự gán APPROVED cho tin legacy hoặc suy diễn mã địa bàn/lịch ca.

Trong Query Tool của pgAdmin kết nối DB đích, có thể kiểm tra chỉ đọc:

```sql
SELECT current_database(), current_user, version();
SELECT table_name FROM information_schema.tables
WHERE table_schema = 'public' ORDER BY table_name;
SELECT * FROM schema_migrations ORDER BY version;
```

Câu cuối chỉ chạy khi bảng `schema_migrations` tồn tại. Phải xác nhận tên DB đích trước mọi thao tác ghi.

### 6.3. Cấu hình môi trường

Sau khi D00 tạo `.env.example` cho từng app, sao chép vào `.env` cục bộ **nếu chưa có file `.env`** (không ghi đè cấu hình hiện tại).

```powershell
if (!(Test-Path .\backend\.env)) { Copy-Item .\backend\.env.example .\backend\.env }
if (!(Test-Path .\frontend\.env)) { Copy-Item .\frontend\.env.example .\frontend\.env }
```

Cấu hình Backend dự kiến, đối chiếu `.env.example` và module config thực tế:

```dotenv
PORT=8080
CORS_ORIGIN=http://localhost:5173
DB_URL=postgresql://localhost:5432/student_jobs_nhom5_v4
DB_USERNAME=<tai_khoan_postgresql_local>
DB_PASSWORD=<mat_khau_local>
JWT_SECRET=<secret_ngau_nhien_rieng>
APP_STORAGE_DIR=<duong_dan_storage_v4_rieng>
```

Frontend:

```dotenv
VITE_API_BASE_URL=http://localhost:8080
```

Các giá trị trong dấu `<...>` là placeholder, không phải thông tin xác thực thật. Thay bằng cấu hình local an toàn; xác minh `httpClient` ghép prefix `/api` đúng hợp đồng, không thêm prefix hai lần. Storage đích tách khỏi storage cũ, nằm ngoài thư mục public. Không in/commit mật khẩu, JWT secret hoặc đường dẫn file cá nhân nhạy cảm.

### 6.4. Cài dependency, migration và chạy Backend

Chỉ thực hiện khi package/lockfile đã có và đúng phạm vi, DB đích đã restore/đối chiếu và được phép chạy migration:

```powershell
Set-Location .\backend
npm ci
npm run db:migrate
npm test
npm run dev
```

`npm ci` cài theo lockfile. Nếu thiếu lockfile hoặc script chưa tồn tại, dừng và kiểm tra `package.json` thay vì tự đổi công nghệ. `npm run db:seed:demo` chỉ chạy trên DB thử nghiệm được chỉ định sau khi xác minh script không phá dữ liệu cần giữ; không chạy mặc định trên bản sao chứa dữ liệu cần bảo toàn.

Mở Terminal khác để kiểm tra server:

```powershell
Invoke-RestMethod -Uri http://localhost:8080/health
```

Health check chỉ xác nhận endpoint sức khỏe phản hồi, không thay thế kiểm thử API, DB và E2E.

### 6.5. Chạy Frontend

Mở Terminal mới tại gốc repository:

```powershell
Set-Location .\frontend
npm ci
npm run build
npm run dev
```

Mở `http://localhost:5173`. Giữ Backend và PostgreSQL hoạt động khi kiểm tra tích hợp. Nếu cổng đã bị chiếm, kiểm tra tiến trình trước khi thay đổi cấu hình; không tự ý dừng tiến trình không liên quan.

## 7. Kiểm thử và tiêu chí nghiệm thu

- Backend: `npm test`, HTTP thực theo API Contract, kiểm tra dữ liệu SQL, xác thực/role/ownership, validation, rollback, race và worker.
- Frontend: `npm run build`, test script nếu đã có, kiểm tra giao diện Chrome/Edge từ 360 px, loading/empty/error và điều hướng quay lại tin sau đăng nhập.
- E2E: đăng ký NTD → đăng tin/ca → Admin kiểm duyệt → SV tìm/Matching → ứng tuyển CV → xét duyệt → thông báo → đóng/hoàn thành → đánh giá; kiểm thử thêm nhánh khóa tài khoản, từ chối, duyệt muộn, hủy và hết hạn.
- Đối chiếu 113 AC MUST; 2 AC SHOULD báo riêng; 4 AC xác minh NTD lịch sử không tính nghiệm thu.
- Lưu lệnh chạy, dữ liệu thử, expected/actual, PASS/FAIL/SKIP/BLOCKED, môi trường và SHA trong `docs/testing/`. Không dùng dữ liệu cá nhân thật làm minh chứng. Build/health thành công không tự chứng minh tất cả AC đạt.

## 8. Nhóm thực hiện và quy trình Git

| Thành viên | Vai trò trong quy trình bàn giao |
|---|---|
| Tài | Lập trình từng Dxx, kiểm thử, đóng gói, rà PR, tích hợp main và kiểm thử lại. |
| Phương | Nhận gói được giao, đặt đúng đường dẫn, kiểm thử, commit và push nhánh của mình. |
| Khang | Nhận gói được giao, đặt đúng đường dẫn, kiểm thử, commit và push nhánh của mình. |

Đây là quy trình bàn giao code, không phải phân ba người lập trình các module độc lập. Trước mỗi đợt ghi base SHA thật, kiểm tra thay đổi local; không để các session ghi đồng thời cùng workspace. Commit bằng tiếng Việt, ví dụ `D07C: Hoàn thiện danh mục địa bàn`.

Sau test đạt và commit checkpoint được phép, tạo hai gói: `CDTN2_Nhom5_Dxx_TenChucNang_FullBackup_SHA7_YYYYMMDD-HHmm.zip` và `CDTN2_Nhom5_Dxx_TenChucNang_ChangedFiles_SHA7_YYYYMMDD-HHmm.zip`. ChangedFiles kèm manifest ADD/MODIFY/DELETE và hướng dẫn bàn giao; kiểm SHA, giải nén, test lại trước PR. SHA7 và thời gian phải lấy từ checkpoint/gói thật. Script backup local phải được kiểm thử runtime trước khi sử dụng chính thức. Không tự push/merge, không force push hoặc reset để che mất thay đổi.

Bộ hướng dẫn AI và script local không nằm trong Git hay ZIP bàn giao; không cần chúng để chạy ứng dụng đã được bàn giao. Không commit `.env`, `node_modules/`, `dist/`, storage, CV thật, DB dump, log nhạy cảm hay ZIP backup. Dump Database và storage sao lưu riêng, không thay thế bằng ZIP code. CV private, logo tùy chọn JPG/PNG/WebP tối đa 2 MiB; server kiểm chữ ký file, kích thước và quyền đọc theo hợp đồng.

## 9. Trạng thái và việc tiếp theo

Bằng chứng đã ghi nhận trong D00 (không phải trạng thái GitHub trực tiếp tại thời điểm đọc README):

| Hạng mục | Trạng thái |
|---|---|
| Repository | Repo mới; tài liệu nền v4.0 đã push lên main tại `e3f454a8502a271f6fa7b147729e7bf77ffe660f`; kiểm lại HEAD hiện tại. |
| Môi trường | Git 2.50.1, Node 24.12.0, npm 11.6.2 và PostgreSQL 16.15 đã được ghi nhận. |
| Database nguồn | pgAdmin đã kết nối thành công; 11 bảng nghiệp vụ + `schema_migrations`; V001–V006 đã áp dụng. Chưa kiểm chứng checksum/cột đầy đủ. |
| Phạm vi | v4.1 đã chốt bỏ xác minh NTD và các BR liên quan; đang đồng bộ tài liệu, Sprint Plan tạm hoãn. |
| Database đích | Đã chọn sao chép sang `student_jobs_nhom5_v4`; chưa có bằng chứng backup/restore thành công. |
| Ứng dụng và backup | Chưa có bằng chứng FE/BE, build/test/health hoặc script backup runtime đạt trong repo mới. |
| D00 | Kiểm kê ban đầu đã thực hiện; **chưa nghiệm thu**. |

Tiếp theo: hoàn tất đồng bộ tài liệu v4.1; kiểm kê/backup/restore DB, xác minh checksum và storage; chốt phương án tái sử dụng code cũ có chọn lọc; khởi tạo FE/BE trong repo mới và kiểm thử; kiểm thử script backup Windows; ghi checkpoint và gói bàn giao trước khi bắt đầu D07C. Cập nhật README theo kết quả thật khi cấu hình hoặc script thay đổi; không ghi trạng thái hoàn thành theo kế hoạch.
