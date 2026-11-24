# Công nghệ, phiên bản và cấu trúc dự án v4.1

## 1. Căn cứ và phạm vi

**CÔNG NGHỆ, PHIÊN BẢN VÀ CẤU TRÚC DỰ ÁN**
CDTN2 Track SE — Nhóm 5 | Nền tảng việc làm part-time cho sinh viên | v4.1 — 24/11/2026
| Thuộc tính | Quy định |
|---|---|
| Phạm vi | SRS v2.1/phạm vi v4.1: 49 US: 35 MUST, 1 SHOULD, 13 WON’T; FR1–28 giữ mã; 119 mã AC, 115 áp dụng (113 MUST, 2 SHOULD), 4 lịch sử. |
| API | 26 API nền + 12 API mới (27–29, 34–42) = 38 mục tiêu; 30–33 ngoài MVP, 41 SHOULD, 42 hỗ trợ logo tùy chọn. |
| Database | 11 bảng kế thừa + 2 bảng địa bàn = 13 bảng nghiệp vụ, 107 trường mục tiêu. schema_migrations không tính; không tạo business_industries. |
| Stack | React/Vite JS, Express CommonJS, pg/PostgreSQL 16, npm. Không đổi stack. |
| Trạng thái | Thiết kế mục tiêu; chưa chứng minh code, migration, test hay merge. |
**Nguồn chuẩn:** SRS_Nhom5_v2.1_PhamVi_v4.1.docx; Product_Backlog_Nhom5_SE_v4.1_US01-US49.xlsx (Roadmap Dxx); Database_Schema_va_quy_trinh_cap_nhat_Nhom5_v4.1.docx; API_Contract_day_du_Nhom5_v4.1.docx và bản Markdown tương ứng. kế thừa tài liệu Công nghệ v4.0 và v3.0; đối chiếu ZIP D01–D05 f80a11e và bản bổ sung D06–D15 FIX. ZIP không chứng minh HEAD/main hiện tại.
**Thuật ngữ:** DTO là hợp đồng dữ liệu FE–BE; migration là SQL có phiên bản; transaction/lock là giao dịch/khóa bản ghi; worker xử lý thời gian khi BE chạy; Matching là so khớp quy tắc, không AI; checkpoint là mốc có kiểm thử và SHA thật.
**Mục lục:** 2. Stack và môi trường; 3. Kiến trúc và quyền; 4–5. Cấu trúc BE/FE; 6–7. Roadmap; 8. API/dữ liệu; 9. Lịch và giao dịch; 10. Migration/bảo mật; 11. Kiểm thử/bàn giao; 12. Kiểm chứng kỹ thuật và kiểm soát thay đổi.
Tái triển khai từ D00 trên repo mới cdtn2-nhom5-ViecLamSinhVien (https://github.com/Taingo-0712/cdtn2-nhom5-ViecLamSinhVien); giữ repo cũ và dữ liệu. Commit nền đã xác nhận e3f454a8502a271f6fa7b147729e7bf77ffe660f (tài liệu v4.0); phải kiểm tra SHA hiện tại khi bắt đầu. D00 chưa nghiệm thu. Không tự tạo thêm repo/project mới, Java/Spring, ORM, dịch vụ AI, queue/scheduler hoặc nâng thư viện. Tên file mới là đề xuất; dò repo trước khi tạo. Thay đổi được tích hợp trực tiếp, không dùng phụ lục ghi đè.

## 2. Công nghệ, phiên bản và môi trường

Phiên bản package đọc từ ZIP D01–D05; Node/npm là baseline engines; kiểm kê D00 ghi nhận Node 24.12.0, npm 11.6.2, Git 2.50.1 và PostgreSQL 16.15. Kết nối pgAdmin đã thấy 11 bảng nghiệp vụ, schema_migrations và V001–V006; chưa đối chiếu checksum hoặc restore. Các package chưa được xác nhận cài trong repo mới. Kiểm tra máy thực, không nâng package tùy ý.
| Thành phần | Phiên bản baseline | Vai trò |
|---|---|---|
| Node.js / npm | 24.12.0 / 11.6.2 | Runtime, npm scripts, lockfile |
| React / React DOM | 19.3.1 / 19.3.1 | JSX, component, CSS thuần |
| react-router-dom | 7.18.5 | Route, quyền, điều hướng |
| Vite / plugin-react | 8.3.6 / 6.2.1 | Dev/build FE |
| Express | 5.3.0 | REST API, CommonJS |
| PostgreSQL / pg | 16.15 / 8.23.1 | SQL tham số, transaction, row lock |
| jsonwebtoken / bcryptjs | 9.0.3 / 3.0.3 | JWT Bearer, băm mật khẩu |
| multer / cors | 2.5.0 / 2.8.6 | Multipart; CORS không thay xác thực |
| Kiểm thử | node:test, Postman | Dùng npm scripts, trình duyệt; không tự thêm framework |

| Cấu hình | Quy ước |
|---|---|
| FE | localhost:5173; VITE_API_BASE_URL=http://localhost:8080; kiểm httpClient trước sửa prefix. |
| BE | localhost:8080/api; /health; PORT, CORS_ORIGIN, DB_URL, DB_USERNAME, DB_PASSWORD, JWT_SECRET, APP_STORAGE_DIR theo .env.example thực. |
| DB | Đích student_jobs_nhom5_v4, schema public, localhost:5432; sao chép từ student_jobs_nhom5, bảo toàn nguồn. DB_URL dự kiến postgresql://localhost:5432/student_jobs_nhom5_v4; thông tin xác thực local. Chưa xác nhận restore. |
| Múi giờ | Asia/Ho_Chi_Minh; timestamp API UTC Z. |
Chạy node -v, npm -v, npm ls --depth=0 trong từng app. Dùng npm ci khi lockfile khớp; không cập nhật lockfile tùy ý. Chỉ chạy db:migrate/seed trên DB được chỉ định sau kiểm kê, không in secret.

## 3. Kiến trúc tổng thể và quyền

**Luồng:** React → httpClient → Express Route → Middleware → Controller/Validator → Service → Repository (pg) → PostgreSQL. Storage riêng cho CV và logo tùy chọn; logo đọc qua API-42 theo quyền. Worker gọi service.
| Lớp | Trách nhiệm |
|---|---|
| FE | Trang chủ hợp nhất, form, loading/empty/error; ẩn nút chỉ là UX. |
| HTTP client | Bearer, data/meta/error, multipart đúng hợp đồng. |
| Middleware | requireAuth, requireRole; đọc account_status/role/token_version từ DB, chặn LOCKED và token thiếu/sai version; kiểm tra ownership tại service. |
| Controller/validator | Parse JSON/query/FormData; kiểm field/kiểu/file; không SQL. |
| Service | Ownership, kiểm duyệt tin, lịch, trạng thái, transaction. |
| Repository | SQL tham số, cùng pg client; không COMMIT riêng. |
| Worker | Quét đơn/tin quá hạn, chạy bù startup, idempotent. |
| DB | PK/FK/UNIQUE/CHECK/index; không thay logic quyền/overlap. |

| Vai trò | Quyền và giới hạn |
|---|---|
| Guest | Trang chủ, chi tiết tin công khai; đăng ký SV/NTD, login. |
| STUDENT | Hồ sơ/lịch riêng, Matching, CV, đơn, thông báo, review hợp lệ. |
| EMPLOYER | Hồ sơ thường; ACTIVE + EMPLOYER được tạo tin hợp lệ (PENDING kiểm duyệt); chỉ quản lý tin/đơn mình. |
| ADMIN | Cấp sẵn; danh sách, khóa/mở, kiểm duyệt tin, danh mục, Dashboard SHOULD. Không tự đọc CV/lịch SV. |
Dùng chung điều kiện công khai ở API-09/10/13/26/41 theo quyền. Tin chưa duyệt/từ chối không công khai; lịch sử theo quyền. BR47: khóa NTD ẩn tin, chặn đơn mới và thao tác thủ công; giữ lịch sử, worker vẫn chạy. Không duyệt mới SV LOCKED; token trước khóa vô hiệu cả sau mở khóa. Không rollout khi code cũ còn bỏ qua điều kiện duyệt.

## 4. Cấu trúc Backend và Database

“ZIP” là đường dẫn thấy trong bản nền hoặc FIX; không khẳng định main có đủ. “Đề xuất” chỉ tạo nếu chưa có module tương đương và phù hợp API/Schema v4.1.
| Đường dẫn từ backend/ | Vai trò / hiện trạng |
|---|---|
| src/app.js; src/server.js | ZIP: mount routes, start server/worker |
| src/config/db.js; src/middleware/requireAuth.js, requireRole.js, errorHandler.js | ZIP nền: pool, auth/role/error; bổ sung kiểm khóa và token_version |
| src/routes/authRoutes.js; src/services/authService.js; src/repositories/userRepository.js | ZIP nền: đăng ký/login, mở rộng hồ sơ API-01–06 |
| src/routes/jobRoutes.js; src/controllers/jobController.js; src/services/jobService.js; src/repositories/jobRepository.js | ZIP: tạo/sửa/đọc/lọc, thêm kiểm duyệt/địa bàn |
| src/routes/workflowRoutes.js; src/services/workflowService.js; src/repositories/workflowRepository.js | ZIP FIX: đơn, quyết định, vòng đời, review, thông báo; không tạo service trùng |
| src/services/scheduleRules.js; src/services/matchingService.js; src/workers/jobLifecycleWorker.js | ZIP FIX: lịch, Matching, worker; xác minh code chạy |
| src/routes/adminRoutes.js; src/controllers/adminController.js; src/services/adminService.js; src/repositories/adminRepository.js | Đề xuất: API-23/34–41, tái sử dụng module tương đương |
| src/routes/locationRoutes.js; src/services/locationService.js; src/repositories/locationRepository.js | Đề xuất: API-27/28; API-29 dùng module danh mục phù hợp |
| src/validators/employerProfileValidator.js | Đề xuất: validation hồ sơ thường, logo API-01/06/42; tái sử dụng tiện ích storage an toàn đã có, không tạo module xác minh. |
| migrations/VNNN__mo_ta.sql; scripts/migrate.js; scripts/seed-demo.js; tests/*.test.js | ZIP nền: migration/seed/node:test; số SQL từ DB thật |
| storage/cv/; storage/logos/ | CV baseline, logo tùy chọn; dưới APP_STORAGE_DIR riêng cho v4.1, ngoài public; giữ storage nguồn. Không public static CV. |
Tên file BE camelCase + Routes/Controller/Service/Repository/Validator. Không dùng Java/Flyway cũ. Không commit storage thật hoặc .env.

## 5. Cấu trúc Frontend và repository

| Đường dẫn từ frontend/src/ | Vai trò / hiện trạng |
|---|---|
| App.jsx; main.jsx; auth/AuthContext.jsx; routes/RequireRole.jsx | ZIP nền: session/role; return-to, kiểm khóa |
| api/httpClient.js; api/authApi.js; api/jobApi.js; api/workflowApi.js | ZIP nền/FIX: mở rộng API, không tạo client trùng |
| pages/jobs/HomePage.jsx; JobListPage.jsx; JobDetailPage.jsx | ZIP FIX: gộp list/filter vào HomePage, giữ chi tiết |
| pages/auth/LoginPage.jsx; RegisterPage.jsx | Login nền; xác minh tên file đăng ký trước mở rộng form |
| pages/employer/CreateJobPage.jsx; EditJobPage.jsx; EmployerJobsPage.jsx; components/JobEditor.jsx | ZIP FIX: tin/ca, kiểm duyệt, địa bàn |
| pages/applications/ApplyPage.jsx; ApplicationList.jsx; ApplicationDetailPage.jsx; ReviewPage.jsx | ZIP FIX: CV/ca, kết quả, review |
| pages/student/AvailabilityPage.jsx; RecommendationsPage.jsx; NotificationsPage.jsx | ZIP FIX: lịch rảnh, Matching, thông báo |
| pages/admin/AdminUsersPage.jsx | ZIP FIX: mở rộng khóa/mở, lọc |
| pages/admin/JobModerationPage.jsx; JobCategoriesPage.jsx; AdminDashboardPage.jsx | Đề xuất: 3 màn bổ sung Admin; Dashboard SHOULD |
| api/adminApi.js; api/locationApi.js; components/LocationSelector.jsx | Đề xuất: API/module dùng chung |
| styles/app.css; styles/flow.css; styles/employer-jobs.css; styles/admin.css | 3 file đầu trong ZIP; admin.css đề xuất |
**Điều hướng:** / là Trang chủ, /jobs redirect tương thích giữ query hợp lệ; giữ route chi tiết. Guest → login/register → đúng tin nếu STUDENT; re-fetch tin/ca, không tự nộp, không redirect ngoài.
**Gốc repo:** AGENTS.md, README.md, .gitignore, frontend/, backend/, docs/, ai-context/backup-dxx.ps1. docs chứa SRS_Nhom5_v2.1_PhamVi_v4.1.docx, Product_Backlog_Nhom5_SE_v4.1_US01-US49.xlsx, bản Word Schema/API/Công nghệ, api-contract-v4.1.md, database-schema-v4.1.md, cong-nghe-cau-truc-v4.1.md, planning/ (Sprint Plan và báo cáo dùng chung), diagrams/ và testing/tai.md và log thành viên. AGENTS.md và ai-context/ chỉ lưu local, bỏ qua bằng .gitignore; ai-context/ chứa boi-canh-chung-code-theo-dot-v4.1.md, bản Word Bối cảnh, quy-uoc-dat-ten-file.md, backup-dxx.ps1 và tien-do-trien-khai.md (theo dõi local). Các tên v4.1 là đích đồng bộ; Sprint Plan, Bối cảnh và AGENTS cần cập nhật trước code. Sao lưu riêng ngoài repo. Không để ZIP trong repo.

## 6. Ma trận module — nền và Admin

Theo sheet Roadmap Dxx Backlog v4.1: 24 đợt triển khai; D15A lịch sử ngoài MVP. Sprint Plan cần đồng bộ. D01/D02 chuẩn bị sớm, nghiệm thu public/quyền sau khi các phụ thuộc tích hợp.
| Đợt / US | API | Module / DB và phụ thuộc |
|---|---|---|
| D00 | — | Kiểm kê Git, package, API, DB, migration |
| D07C / US02,06,07,11,38 | 27–28* | 2 bảng địa bàn; BR43/50, trước D03/D05/D07 |
| D01 / US08–09 | 09/10 | Home/detail, nghiệm thu public sau D15B |
| D02 / US03–05 | 02 | Auth, return-to; kiểm khóa sau D15 |
| D03 / US01–02 | 01/29/42* | users/profiles, logo tùy chọn; D02/D07C, nền is_active trước API-29; BR48/51 |
| D04 / US35–38 | 03–06/42* | Hồ sơ thường, logo; D03, kiểm quyền sau D15 |
| D15 / US39–41 | 23/37* | Admin tài khoản, users; D02/D03, BR47 |
| D15C / US48 | 07/38–40* | Quản trị danh mục; dùng is_active đã chuẩn bị ở D03, không migration trùng; trước tin mới |
| D05 / US06 | 08/12 | Tạo/quản lý tin PENDING; D03/D07C/D15C |
| D07A / US06,07,09,43 | 08/09/10/11 | job_time_slots; nền D05, trước D08 |
| D15B / US47 | 34–36* | Kiểm duyệt; D05/D07A, BR46 |
| D06 / US07 | 11 | Sửa tin/khóa toàn bộ nội dung sau đơn; D07A/D15B |
* API mới thuộc phạm vi v4.1 (27–29, 34–42). D05 nền → D07A lịch → D15B → D06 nghiệm thu nhằm tránh phụ thuộc vòng; D02/D15 thống nhất migration account_status/token_version; D03/D15C thống nhất migration is_active. API-30–33 và D15A không triển khai. Không tự đổi roadmap.

## 7. Ma trận module — Full E2E

| Đợt / US | API | Module / phụ thuộc |
|---|---|---|
| D07 / US10–11 | 09 | Lọc Homepage; D01/D07C/D07A/D15B |
| D07B / US42,44 | 24–26 | Lịch rảnh/Matching; D02/D07A/D15B |
| D08 / US12 | 13 | CV + selected slots; D02/D07A/D15B |
| D09 / US13 | 15–17 | Đơn/CV/ca; D08 |
| D10 / US14–15 | 18 | Duyệt/từ chối/duyệt muộn, notification; D09 |
| D11 / US16–17 | 14/16 | SV xem kết quả; D08–10, hồi quy D12 |
| D12 / US17–19,45 | 19/22 + worker | Đóng/hủy/hoàn thành/tự hết hạn; D07A/D08/D10 |
| D13 / US20–21 | 20/21 | Review hai chiều; D12 |
| D14 / US22 | 22 | Thông báo web; D10/D12 |
| D15D / US49 | 41* | Dashboard SHOULD; D15/D15B/D15C và dữ liệu thật |
| D16 / US03,12 | 02/09/10/13 | UX/return-to/responsive; tích hợp |
| D17 | API thuộc phạm vi | Nghiệm thu 35 MUST, SHOULD báo riêng |
**Luồng E2E:** NTD đăng ký hồ sơ thường → đăng nhập → NTD tạo tin + ca → Admin duyệt → SV tìm/lọc/Matching → login về đúng tin → chọn ca + CV → NTD quyết định → thông báo → đóng tuyển/hoàn thành → review hai chiều. Hủy/hết hạn là nhánh riêng.
Dashboard không tạo bảng riêng, không thống kê NTD riêng (US29 WON’T). Không xây chat, thanh toán, chấm công, native app, khiếu nại, yêu thích, quota người hoặc duyệt từng ca.
Tài lập trình từng D; Phương/Khang nhận file, test/commit/push; Tài soát PR, merge và hồi quy. Không chia code độc lập. Mỗi đợt có base SHA, kiểm thử, checkpoint SHA và hai ZIP.

## 8. Hợp đồng API và dữ liệu

| Nhóm | API / trường chính |
|---|---|
| Tài khoản | 01/02/23/37*: users.account_status, locked_at/by, lock_reason, unlocked_at/by, token_version; role độc lập |
| NTD | 01/05/06/42*: employer_profiles.user_id PK/FK, display_name, contact_name, phone, introduction; business_type, tax_code tùy chọn, industry_id, province_code, ward_code, address_detail, website_url, logo_storage_key; logo_url dẫn xuất |
| Địa bàn | 27/28*,09/26: administrative_provinces, administrative_wards; nhóm locations; mã chuỗi, FK ghép, city server sinh |
| Danh mục | 07/29*/38–40*: job_categories.is_active; industry_id FK job_categories.category_id; API-07 scope=selection/filter; API-29 chỉ active |
| Tin | 08/10/11/12/34–36*: moderation_status, moderation_reason, moderated_at/by; status độc lập |
| Lịch | 08/11/24/25: schedule_type/mode/dates, time_slots; WEEKLY day_of_week/start_time/end_time; ONE_TIME start_at/end_at |
| Đơn/CV | 13/17/18: cv multipart, selected_slot_ids JSON text, application_selected_slots, decided_at, confirm_late_approval |
| Vòng đời | 19/22: cancelled_at, cancel_reason, status_before_cancel, completed_at; notifications giữ 5 cột |
| Matching/Dashboard | 26/41*: matched_slots_count, total_slots_count, match_percent; 5 số đếm DB: tổng SV, tổng NTD, tin PENDING, tin đang tuyển đủ điều kiện công khai, tổng đơn; không lưu cột đếm |
4 nhóm trạng thái: account ACTIVE/LOCKED; moderation PENDING/APPROVED/REJECTED; tin RECRUITING/FILLED/COMPLETED/EXPIRED/CANCELLED; đơn PENDING/APPROVED/REJECTED/JOB_EXPIRED/JOB_CANCELLED.
JSON snake_case; BIGINT ID chuỗi; tiền decimal chuỗi VND; mã địa bàn giữ số 0 đầu. API-01/06 NTD JSON khi không có logo; multipart profile JSON text + logo khi có file JPG/PNG/WebP ≤2 MiB. API-13 multipart CV PDF ≤5 MiB + selected_slot_ids JSON text. API-17/42 trả bytes theo quyền; lỗi error.code/message/details và trace_id. JWT 3600 giây, sub ID chuỗi, role, iat, exp, token_version; LOCKED trả 403 ACCOUNT_LOCKED, version thiếu/sai trả 401 INVALID_TOKEN. Không lộ password_hash, token_version trong DTO user hoặc raw storage key. * API mới trong phạm vi; API-41 SHOULD, API-42 hỗ trợ logo tùy chọn.

## 9. Lịch ca, worker và transaction

| Nghiệp vụ | Quy tắc |
|---|---|
| Lịch ca | WEEKLY/ONE_TIME, 1–21 ca/tin; ca đêm 1 logic, không 24:00, chống chồng CN→T2; WEEKLY ngày bắt đầu bắt buộc, kết thúc tùy chọn. |
| Lịch rảnh/Matching | SV 0–21 khoảng; trọn một khoảng, không ghép; ANY ≥1, ALL toàn bộ ca xét; không chặn ứng tuyển. |
| Nộp/duyệt | Một SV/tin; chọn ca tương lai, PDF; duyệt cả đơn; duyệt muộn xác nhận + decided_at; khóa toàn bộ sửa nội dung tin sau bất kỳ đơn; trước đơn, sửa tin RECRUITING hợp lệ chuyển moderation_status=PENDING. |
| Hết hạn | Đơn hết mọi ca chọn → JOB_EXPIRED. Sau kết thúc ca cuối: ≥1 APPROVED → FILLED, 0 → EXPIRED; WEEKLY vô hạn không tự hết. |
| Đóng tuyển | RECRUITING→FILLED chỉ ≥1 APPROVED và 0 PENDING; không tự REJECTED. |
| Hủy | Chủ tin RECRUITING/FILLED, trước NGÀY ca đầu theo giờ VN; lý do nếu có đơn; mọi đơn JOB_CANCELLED, giữ status_before_cancel/decided_at. |
| Hoàn thành/review | NTD xác nhận FILLED→COMPLETED; review COMPLETED+APPROVED, mỗi chiều một lần. |
Worker Node chạy định kỳ/quét bù startup; không chạy khi server tắt. Chu kỳ chưa phải SLA. API-13/18/19 luôn tự kiểm thời điểm.
**Thứ tự lock:** users liên quan theo ID tăng → employer_profiles → category khi tạo/đổi → job_posts → applications theo ID tăng. Kiểm lại role/account_status/token_version và điều kiện nghiệp vụ sau lock. Worker chỉ lock job rồi đơn, không lấy user/profile ngược. Cùng pg client, UPDATE trạng thái + INSERT notification nguyên tử; lỗi rollback hết. Chỉ thông báo khi trạng thái thực sự đổi; test race giữa khóa/duyệt/ứng tuyển/hủy/worker. File ngoài transaction: dọn file mới khi rollback, giữ file cũ đến commit, kiểm thử dọn mồ côi.

## 10. Migration, địa bàn và bảo mật

**13 bảng mục tiêu:** users, student_profiles, employer_profiles, job_categories, job_posts, job_time_slots, student_availability_slots, applications, application_selected_slots, reviews, notifications, administrative_provinces, administrative_wards. 107 trường mục tiêu; không tạo business_industries; schema_migrations là kỹ thuật.
| Thứ tự phụ thuộc (không phải số SQL) | Công việc |
|---|---|
| A — nền v3 | Kiểm kê, bù lịch/đơn chọn ca/thông báo nếu thiếu. |
| B — D07C | Danh mục địa bàn, seed có nguồn/phiên bản; xác nhận PK/FK, độ dài. |
| C — D02/D15 | account_status, metadata khóa/mở, token_version; BR47. |
| D — nền D03/D15C | job_categories.is_active trước API-29, giữ ID; D15C không tạo migration trùng. |
| E — D03 | employer_profiles hồ sơ thường, logo, FK lĩnh vực dùng chung; BR48/51. |
| F — D05/D15B | job_posts địa bàn/kiểm duyệt; BR46/47/50. |
| G — hoàn thiện | Validate constraints, backfill đối soát, NOT NULL khi đủ dữ liệu. |
Giữ V001–V006 đã ghi nhận trong DB nguồn; đối chiếu checksum file cũ, kiểm file và schema_migrations trước chọn VNNN (dự kiến V007). Không sửa migration đã chạy. Dữ liệu legacy thiếu mã/file không đoán, không tự APPROVED, không xóa đơn/CV/ca/ID. Tin legacy thiếu lịch chỉ đọc/nộp CV không ca nếu đủ điều kiện; không Matching, không suy hạn từ work_time. Rollout app+DB đồng bộ, không reset DB thật. Backup/restore nguồn sang student_jobs_nhom5_v4 theo quyết định D00; kiểm số dòng, FK, checksum và private storage; thử cả DB sạch và DB sao chép. Chưa có bằng chứng restore. Không chạy migration/seed trên DB nguồn.
**Bảo mật:** APP_STORAGE_DIR ngoài public; PDF CV tối đa 5 MiB; logo JPG/PNG/WebP tối đa 2 MiB, kiểm chữ ký và kích thước. Key server tạo; CV chỉ theo quyền chủ tin/chủ đơn đúng hợp đồng, không mặc nhiên Admin đọc CV; logo qua API-42. Không thu minh chứng NTD, không public static CV, không log CV/key/secret. Dữ liệu demo giữ qua nghiệm thu, dọn có xác nhận. Không mặc định email hoặc thêm notifications cho Admin. Git/ZIP loại .env, private storage, DB dump/data thật, node_modules, dist, .git, log nhạy cảm.

## 11. Kiểm thử, checkpoint và backup

| Bước kiểm tra | Bằng chứng |
|---|---|
| D00 | git status --short --branch; git rev-parse HEAD; git log -1; node -v; npm -v; npm ls --depth=0; DB version/schema_migrations, đối chiếu checksum, restore, kiểm thử backup script Windows. Nếu chưa có app thì ghi chưa chạy npm ls/build/test. |
| BE/DB | backend: npm test; npm run db:migrate trên DB thử đã duyệt; GET /health; HTTP thật, SQL kiểm dữ liệu, rollback/race/quyền. |
| FE | frontend: npm run build; npm run dev khi thử UI; role, 360px+, loading/empty/error, reset/return-to. |
| E2E | SRS M01–M26, Schema DB01–DB30, API mục 7; 115 AC áp dụng (113 MUST, 2 SHOULD); AC24.1–24.4 và DB19 ngoài MVP; ghi BLOCKED khi thiếu điều kiện kiểm chứng, SHOULD báo riêng. |
| Hồi quy | Ca đêm, Matching, legacy, CV, đóng/hủy/hết hạn, review, thông báo; không phá module cũ. |
Lệnh dùng PowerShell, vào đúng app. npm run db:seed:demo chỉ DB thử đã chỉ định. node --check/build/health không chứng minh E2E. Lưu lệnh, expected/actual, PASS/FAIL/SKIP/BLOCKED, môi trường và SHA tại docs/testing/tai.md hoặc file test Dxx.
Sau test đạt và commit được phép, chạy ai-context/backup-dxx.ps1. FullBackup: toàn bộ mã nguồn tracked đủ cấu trúc (trừ bí mật); ChangedFiles: ADD/MODIFY diff BASE→HEAD, DELETE/RENAME trong manifest. Tên ví dụ:
CDTN2_Nhom5_D15B_KiemDuyetTin_FullBackup_SHA7_YYYYMMDD-HHmm.zip
CDTN2_Nhom5_D15B_KiemDuyetTin_ChangedFiles_SHA7_YYYYMMDD-HHmm.zip
Đặt cạnh repo trong CDTN2_Backups; so manifest, giải nén thử, kiểm SHA256, không chứa bí mật. Chỉ thông báo hoàn thành sau test và hai ZIP hợp lệ. Người nhận kiểm base, đặt file, test, commit tiếng Việt/push; Tài review/merge/hồi quy. Không báo push/merge nếu thiếu chứng cứ. FullBackup không phải pg_dump.

## 12. Kiểm chứng kỹ thuật và kiểm soát thay đổi

Các BR đã được duyệt trong D00 v4.1; không yêu cầu phê duyệt lại cùng nội dung. BR45/US46/FR24/UC20/AC24.1–AC24.4 và API-30–33 ngoài MVP. Chỉ xin xác nhận nếu có thay đổi ngoài quyết định hoặc mâu thuẫn đặc tả.
| Mã | Kiểm chứng trước code |
|---|---|
| BR43/50 | Xác minh nguồn mã địa bàn chính thức, phiên bản/ngày hiệu lực, FK, encoding locations; không suy mã legacy. |
| BR45/51 | Không xác minh NTD; kiểm thử hồ sơ thường, mã số thuế tùy chọn, CV/logo, quyền đọc, rollback và dọn file; API-01/05/06/13/17/42. |
| BR48 | Dùng chung job_categories; FK industry_id, seed is_active trước D03, scope API-07, API-29 và kiểm trùng/kích hoạt lại API-39/40. |
| BR46 | Kiểm thử sửa tin RECRUITING chưa/có đơn, chuyển PENDING, metadata kiểm duyệt và race; API-11/36. |
| BR47 | Kiểm thử token_version tăng khi khóa, mở khóa giữ version, ẩn tin/giữ lịch sử, không duyệt SV LOCKED, worker; API-37. |
| BR50 | Kiểm kê schema/checksum, backup/restore, DTO/lỗi theo 38 API, rollout đồng bộ, không tự APPROVED tin legacy. |
**Checklist:** đúng checkout, đủ tài liệu, đối chiếu SRS/Backlog/API/Schema, Dxx và phạm vi, file/tác động, kiểm chứng quyết định BR đã chốt, kiểm thử AC. Cổng chưa đạt chỉ làm phần độc lập đã chốt; không sửa phần BLOCKED.
**Đồng bộ:** AGENTS.md quy định công việc và backup; quy-uoc-dat-ten-file.md mới thay quy ước Java/Flyway lỗi thời. ERD, UC, kiến trúc drawio, Sprint Plan, Bối cảnh Dxx v4.1 là tài liệu đồng bộ; Figma bỏ qua; không dùng chúng ghi đè SRS. Đồng bộ phiên bản tài liệu còn lại trước lập trình; khi thay đổi hợp đồng phải cập nhật SRS/Backlog/Schema/API và truy vết sau xác nhận.
**Lịch sử:** v3.0 (13/11/2026) Full E2E, lịch, Matching, hủy; v4.0 (24/11/2026) Trang chủ, địa bàn, return-to, từng đề xuất xác minh NTD, kiểm duyệt, khóa, danh mục, Dashboard và backup Dxx. v4.1 (24/11/2026) bỏ xác minh, chốt BR, DB riêng, 38 API và 24 đợt. Git và kết nối pgAdmin đã kiểm tra theo báo cáo D00; chưa nghiệm thu app, migration v4.1, restore hay test HTTP. Script backup cần kiểm thử Windows thực.
