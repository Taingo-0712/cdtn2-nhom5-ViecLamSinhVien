# Công nghệ, phiên bản và cấu trúc dự án v4.0

## 1. Căn cứ và phạm vi

**CÔNG NGHỆ, PHIÊN BẢN VÀ CẤU TRÚC DỰ ÁN**
CDTN2 Track SE — Nhóm 5 | Nền tảng việc làm part-time cho sinh viên | v4.0 — 24/11/2026
| Thuộc tính | Quy định |
|---|---|
| Phạm vi | SRS v2.0/phạm vi v4.0: 49 US, 36 MUST, 1 SHOULD, 12 WON’T; FR1–28; 119 AC. |
| API | 26 endpoint nền + 16 đề xuất API-27–42, chưa phê duyệt. |
| Database | 11 bảng kế thừa + 2 bảng địa bàn; 14 nếu tách business_industries. schema_migrations không tính. |
| Stack | React/Vite JS, Express CommonJS, pg/PostgreSQL 16, npm. Không đổi stack. |
| Trạng thái | Thiết kế mục tiêu; chưa chứng minh code, migration, test hay merge. |
**Nguồn chuẩn:** SRS_Nhom5_v2.0_PhamVi_v4.0.docx; Product_Backlog_Nhom5_SE_v4.0_US01-US49.xlsx (Roadmap Dxx); Database_Schema_va_quy_trinh_cap_nhat_Nhom5_v4.0.docx; API_Contract_day_du_Nhom5_v4.0.docx và bản Markdown tương ứng. Kế thừa tài liệu Công nghệ v3.0; đối chiếu ZIP D01–D05 f80a11e và bản bổ sung D06–D15 FIX. ZIP không chứng minh HEAD/main hiện tại.
**Thuật ngữ:** DTO là hợp đồng dữ liệu FE–BE; migration là SQL có phiên bản; transaction/lock là giao dịch/khóa bản ghi; worker xử lý thời gian khi BE chạy; Matching là so khớp quy tắc, không AI; checkpoint là mốc có kiểm thử và SHA thật.
**Mục lục:** 2. Stack và môi trường; 3. Kiến trúc và quyền; 4–5. Cấu trúc BE/FE; 6–7. Roadmap; 8. API/dữ liệu; 9. Lịch và giao dịch; 10. Migration/bảo mật; 11. Kiểm thử/bàn giao; 12. Cổng phê duyệt.
Tái triển khai từ D00 trên repo mới do Tài chỉ định, giữ repo cũ và dữ liệu. Không tự tạo thêm repo/project mới, Java/Spring, ORM, dịch vụ AI, queue/scheduler hoặc nâng thư viện. Tên file mới là đề xuất; dò repo trước khi tạo. Thay đổi được tích hợp trực tiếp, không dùng phụ lục ghi đè.

## 2. Công nghệ, phiên bản và môi trường

Phiên bản package đọc từ ZIP D01–D05; Node/npm là engines, PostgreSQL là phiên bản lịch sử. Kiểm tra máy thực, không nâng package tùy ý.
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
| DB | student_jobs_nhom5, schema public; cổng local thường 5432. |
| Múi giờ | Asia/Ho_Chi_Minh; timestamp API UTC Z. |
Chạy node -v, npm -v, npm ls --depth=0 trong từng app. Dùng npm ci khi lockfile khớp; không cập nhật lockfile tùy ý. Chỉ chạy db:migrate/seed trên DB được chỉ định sau kiểm kê, không in secret.

## 3. Kiến trúc tổng thể và quyền

**Luồng:** React → httpClient → Express Route → Middleware → Controller/Validator → Service → Repository (pg) → PostgreSQL. Storage riêng cho CV/minh chứng; logo qua hợp đồng riêng. Worker gọi service.
| Lớp | Trách nhiệm |
|---|---|
| FE | Trang chủ hợp nhất, form, loading/empty/error; ẩn nút chỉ là UX. |
| HTTP client | Bearer, data/meta/error, multipart đúng hợp đồng. |
| Middleware | requireAuth, requireRole; đọc account_status/role từ DB, chặn LOCKED cả JWT cũ. |
| Controller/validator | Parse JSON/query/FormData; kiểm field/kiểu/file; không SQL. |
| Service | Ownership, xác minh, kiểm duyệt, lịch, trạng thái, transaction. |
| Repository | SQL tham số, cùng pg client; không COMMIT riêng. |
| Worker | Quét đơn/tin quá hạn, chạy bù startup, idempotent. |
| DB | PK/FK/UNIQUE/CHECK/index; không thay logic quyền/overlap. |

| Vai trò | Quyền và giới hạn |
|---|---|
| Guest | Trang chủ, chi tiết tin công khai; đăng ký SV/NTD, login. |
| STUDENT | Hồ sơ/lịch riêng, Matching, CV, đơn, thông báo, review hợp lệ. |
| EMPLOYER | Hồ sơ/minh chứng riêng; ACTIVE + APPROVED mới đăng tin; chỉ quản lý tin/đơn mình. |
| ADMIN | Cấp sẵn; danh sách, khóa/mở, xác minh NTD/tin, danh mục, Dashboard SHOULD. Không tự đọc CV/lịch SV. |
Dùng chung điều kiện công khai ở API-09/10/13/26/41 theo quyền. Tin chưa duyệt/từ chối không công khai; lịch sử theo quyền. BR47 khóa NTD có tin/đơn chưa chốt. Không rollout khi code cũ còn bỏ qua điều kiện duyệt.

## 4. Cấu trúc Backend và Database

“ZIP” là đường dẫn thấy trong bản nền hoặc FIX; không khẳng định main có đủ. “Đề xuất” chỉ tạo nếu chưa có module tương đương và đã duyệt API/Schema.
| Đường dẫn từ backend/ | Vai trò / hiện trạng |
|---|---|
| src/app.js; src/server.js | ZIP: mount routes, start server/worker |
| src/config/db.js; src/middleware/requireAuth.js, requireRole.js, errorHandler.js | ZIP nền: pool, auth/role/error; bổ sung kiểm khóa |
| src/routes/authRoutes.js; src/services/authService.js; src/repositories/userRepository.js | ZIP nền: đăng ký/login, mở rộng hồ sơ API-01–06 |
| src/routes/jobRoutes.js; src/controllers/jobController.js; src/services/jobService.js; src/repositories/jobRepository.js | ZIP: tạo/sửa/đọc/lọc, thêm kiểm duyệt/địa bàn |
| src/routes/workflowRoutes.js; src/services/workflowService.js; src/repositories/workflowRepository.js | ZIP FIX: đơn, quyết định, vòng đời, review, thông báo; không tạo service trùng |
| src/services/scheduleRules.js; src/services/matchingService.js; src/workers/jobLifecycleWorker.js | ZIP FIX: lịch, Matching, worker; xác minh code chạy |
| src/routes/adminRoutes.js; src/controllers/adminController.js; src/services/adminService.js; src/repositories/adminRepository.js | Đề xuất: API-23/30–31/33–41, tái sử dụng module tương đương |
| src/routes/locationRoutes.js; src/services/locationService.js; src/repositories/locationRepository.js | Đề xuất: API-27/28; API-29 dùng module danh mục phù hợp |
| src/services/verificationStorageService.js; src/validators/employerProfileValidator.js | Đề xuất: file riêng, validation hồ sơ, API-32/42 |
| migrations/VNNN__mo_ta.sql; scripts/migrate.js; scripts/seed-demo.js; tests/*.test.js | ZIP nền: migration/seed/node:test; số SQL từ DB thật |
| storage/cv/; storage/verification/; storage/logos/ | CV baseline; hai vùng mới đề xuất dưới APP_STORAGE_DIR; không public minh chứng |
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
| pages/admin/EmployerVerificationPage.jsx; JobModerationPage.jsx; JobCategoriesPage.jsx; AdminDashboardPage.jsx | Đề xuất: 4 màn Admin; Dashboard SHOULD |
| api/adminApi.js; api/locationApi.js; components/LocationSelector.jsx; components/EmployerVerificationFields.jsx | Đề xuất: API/module dùng chung |
| styles/app.css; styles/flow.css; styles/employer-jobs.css; styles/admin.css | 3 file đầu trong ZIP; admin.css đề xuất |
**Điều hướng:** / là Trang chủ, /jobs redirect tương thích giữ query hợp lệ; giữ route chi tiết. Guest → login/register → đúng tin nếu STUDENT; re-fetch tin/ca, không tự nộp, không redirect ngoài.
**Gốc repo:** AGENTS.md, README.md, .gitignore, frontend/, backend/, docs/, ai-context/backup-dxx.ps1. docs chứa SRS/Backlog, api-contract-v4.0.md, database-schema-v4.0.md, cong-nghe-cau-truc-v4.0.md, planning/ (Sprint Plan và báo cáo dùng chung), diagrams/ và testing/tai.md và log thành viên. AGENTS.md và ai-context/ chỉ lưu local, bỏ qua bằng .gitignore; ai-context/ chứa boi-canh-chung-code-theo-dot-v4.0.md, quy-uoc-dat-ten-file.md và backup-dxx.ps1. Sao lưu riêng ngoài repo. Không để ZIP trong repo.

## 6. Ma trận module — nền và Admin

Theo sheet Roadmap Dxx Backlog v4.0, không phải Sprint đã duyệt. D07C, D15A–D15D là checkpoint mở rộng đề xuất. D01/D02/D04 chuẩn bị sớm, nghiệm thu sau khi quyền liên quan tích hợp.
| Đợt / US | API | Module / DB và phụ thuộc |
|---|---|---|
| D00 | — | Kiểm kê Git, package, API, DB, migration |
| D07C / US02,06,07,11,38 | 27–28* | 2 bảng địa bàn; BR43/50, trước D03/D05/D07 |
| D01 / US08–09 | 09/10 | Home/detail, nghiệm thu public sau D15B |
| D02 / US03–05 | 02 | Auth, return-to; kiểm khóa sau D15 |
| D03 / US01–02 | 01/29* | users/profiles, file; D07C, BR45/48/51 |
| D04 / US35–38 | 03–06 | Hồ sơ; D03, kết quả xác minh sau D15A |
| D15 / US39–41 | 23/37* | Admin tài khoản, users; D02/D03, BR47 |
| D15A / US46 | 30–33*,42* | Xác minh NTD, logo; D03, BR45/51 |
| D15C / US48 | 07/38–40* | job_categories.is_active, trước tin mới |
| D05 / US06 | 08/12 | Tạo/quản lý tin PENDING; D03/D07C/D15A/C |
| D07A / US06,07,09,43 | 08/09/10/11 | job_time_slots; nền D05, trước D08 |
| D15B / US47 | 34–36* | Kiểm duyệt; D05/D07A/D15A, BR46 |
| D06 / US07 | 11 | Sửa tin/khóa lịch; D07A/D15B |
* API mới đề xuất. D05 nền → D07A lịch → D15B → D06 nghiệm thu nhằm tránh phụ thuộc vòng; D03/D15 phải thống nhất checkpoint migration account_status. Chốt trước code, không tự đổi roadmap.

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
| D15D / US49 | 41* | Dashboard SHOULD; D15/15A–C và dữ liệu thật |
| D16 / US03,12 | 02/09/10/13 | UX/return-to/responsive; tích hợp |
| D17 | API thuộc phạm vi | Nghiệm thu 36 MUST, SHOULD báo riêng |
**Luồng E2E:** NTD đăng ký + minh chứng → Admin duyệt → NTD tạo tin + ca → Admin duyệt → SV tìm/lọc/Matching → login về đúng tin → chọn ca + CV → NTD quyết định → thông báo → đóng tuyển/hoàn thành → review hai chiều. Hủy/hết hạn là nhánh riêng.
Dashboard không tạo bảng riêng, không thống kê NTD riêng (US29 WON’T). Không xây chat, thanh toán, chấm công, native app, khiếu nại, yêu thích, quota người hoặc duyệt từng ca.
Tài lập trình từng D; Phương/Khang nhận file, test/commit/push; Tài soát PR, merge và hồi quy. Không chia code độc lập. Mỗi đợt có base SHA, kiểm thử, checkpoint SHA và hai ZIP.

## 8. Hợp đồng API và dữ liệu

| Nhóm | API / trường chính |
|---|---|
| Tài khoản | 01/02/23/37*: users.account_status, locked_at/by, lock_reason, unlocked_at/by; role độc lập |
| NTD | 01/05/06/30–33*/42*: employer_profiles.user_id PK/FK, display_name, contact_name, phone, introduction; business_type, tax_code, industry_id, province_code, ward_code, address_detail, website_url, verification_status, reject_reason, verified_at/by, storage keys riêng |
| Địa bàn | 27/28*,09/26: administrative_provinces, administrative_wards; nhóm locations; mã chuỗi, FK ghép, city server sinh (chờ duyệt) |
| Danh mục | 07/29*/38–40*: job_categories.is_active; industry_id FK riêng/chung chờ BR48 |
| Tin | 08/10/11/12/34–36*: moderation_status, moderation_reason, moderated_at/by; status độc lập |
| Lịch | 08/11/24/25: schedule_type/mode/dates, time_slots; WEEKLY day_of_week/start_time/end_time; ONE_TIME start_at/end_at |
| Đơn/CV | 13/17/18: cv multipart, selected_slot_ids JSON text, application_selected_slots, decided_at, confirm_late_approval |
| Vòng đời | 19/22: cancelled_at, cancel_reason, status_before_cancel, completed_at; notifications giữ 5 cột |
| Matching/Dashboard | 26/41*: matched_slots_count, total_slots_count, match_percent; 6 số đếm DB, không lưu cột đếm |
5 nhóm trạng thái: account ACTIVE/LOCKED; verification và moderation độc lập PENDING/APPROVED/REJECTED; tin RECRUITING/FILLED/COMPLETED/EXPIRED/CANCELLED; đơn PENDING/APPROVED/REJECTED/JOB_EXPIRED/JOB_CANCELLED.
JSON snake_case; BIGINT ID chuỗi; tiền decimal chuỗi VND; mã địa bàn giữ số 0 đầu. API-01/06 NTD multipart profile JSON text + verification_doc + logo tùy chọn là đề xuất. CV PDF ≤5 MiB. API-17/32/42 trả bytes theo quyền; lỗi error/trace_id. Không lộ password_hash hoặc raw storage key. * API mới chờ phê duyệt.

## 9. Lịch ca, worker và transaction

| Nghiệp vụ | Quy tắc |
|---|---|
| Lịch ca | WEEKLY/ONE_TIME, 1–21 ca/tin; ca đêm 1 logic, không 24:00, chống chồng CN→T2; WEEKLY ngày bắt đầu bắt buộc, kết thúc tùy chọn. |
| Lịch rảnh/Matching | SV 0–21 khoảng; trọn một khoảng, không ghép; ANY ≥1, ALL toàn bộ ca xét; không chặn ứng tuyển. |
| Nộp/duyệt | Một SV/tin; chọn ca tương lai, PDF; duyệt cả đơn; duyệt muộn xác nhận + decided_at; khóa sửa lịch sau bất kỳ đơn. |
| Hết hạn | Đơn hết mọi ca chọn → JOB_EXPIRED. Sau kết thúc ca cuối: ≥1 APPROVED → FILLED, 0 → EXPIRED; WEEKLY vô hạn không tự hết. |
| Đóng tuyển | RECRUITING→FILLED chỉ ≥1 APPROVED và 0 PENDING; không tự REJECTED. |
| Hủy | Chủ tin RECRUITING/FILLED, trước NGÀY ca đầu theo giờ VN; lý do nếu có đơn; mọi đơn JOB_CANCELLED, giữ status_before_cancel/decided_at. |
| Hoàn thành/review | NTD xác nhận FILLED→COMPLETED; review COMPLETED+APPROVED, mỗi chiều một lần. |
Worker Node chạy định kỳ/quét bù startup; không chạy khi server tắt. Chu kỳ chưa phải SLA. API-13/18/19 luôn tự kiểm thời điểm.
**Thứ tự lock:** users liên quan theo ID tăng → employer_profiles → category khi tạo/đổi → job_posts → applications theo ID tăng. Kiểm lại quyền/trạng thái sau lock. Worker chỉ lock job rồi đơn, không lấy user/profile ngược. Cùng pg client, UPDATE trạng thái + INSERT notification nguyên tử; lỗi rollback hết. Chỉ thông báo khi trạng thái thực sự đổi; test race giữa khóa/duyệt/ứng tuyển/hủy/worker. File ngoài transaction: dọn file mới khi rollback, giữ file cũ đến commit, kiểm thử dọn mồ côi.

## 10. Migration, địa bàn và bảo mật

**13 bảng mục tiêu:** users, student_profiles, employer_profiles, job_categories, job_posts, job_time_slots, student_availability_slots, applications, application_selected_slots, reviews, notifications, administrative_provinces, administrative_wards. business_industries là bảng 14 có điều kiện; schema_migrations là kỹ thuật.
| Thứ tự phụ thuộc (không phải số SQL) | Công việc |
|---|---|
| A — nền v3 | Kiểm kê, bù lịch/đơn chọn ca/thông báo nếu thiếu. |
| B — D07C | Danh mục địa bàn, seed có nguồn/phiên bản; xác nhận PK/FK, độ dài. |
| C — D15/D02 | account_status, kiểm quyền; BR47. |
| D — D03/D15A | employer_profiles, file, FK lĩnh vực; BR45/48/51. |
| E — D15C | job_categories.is_active, giữ ID. |
| F — D05/D15B | job_posts địa bàn/kiểm duyệt; BR46/47/50. |
| G — hoàn thiện | Validate constraints, backfill đối soát, NOT NULL khi đủ dữ liệu. |
Không sửa migration đã chạy; kiểm file và schema_migrations trước chọn VNNN. Dữ liệu legacy thiếu mã/file không đoán, không tự APPROVED, không xóa đơn/CV/ca/ID. Tin legacy thiếu lịch chỉ đọc/nộp CV không ca nếu đủ điều kiện; không Matching, không suy hạn từ work_time. Rollout app+DB đồng bộ, không reset DB thật. Dump/restore thử trên DB riêng nếu cần.
**Bảo mật:** APP_STORAGE_DIR ngoài public; PDF CV tối đa 5 MiB; minh chứng theo loại/kích thước/retention BR45/51 đã duyệt. Key server tạo; đúng chủ/Admin mới xem minh chứng; logo tách vùng và quyền. Không public static giấy tờ, không log CV/CCCD/key/secret. Không mặc định email hoặc thêm notifications cho Admin. Git/ZIP loại .env, private storage, DB dump/data thật, node_modules, dist, .git, log nhạy cảm.

## 11. Kiểm thử, checkpoint và backup

| Bước kiểm tra | Bằng chứng |
|---|---|
| D00 | git status --short --branch; git rev-parse HEAD; git log -1; node -v; npm -v; npm ls --depth=0; DB version/schema_migrations. |
| BE/DB | backend: npm test; npm run db:migrate trên DB thử đã duyệt; GET /health; HTTP thật, SQL kiểm dữ liệu, rollback/race/quyền. |
| FE | frontend: npm run build; npm run dev khi thử UI; role, 360px+, loading/empty/error, reset/return-to. |
| E2E | SRS M01–M26, Schema DB01–DB30, API mục 7; 119 AC theo phạm vi; AC chờ chốt BLOCKED, SHOULD báo riêng. |
| Hồi quy | Ca đêm, Matching, legacy, CV, đóng/hủy/hết hạn, review, thông báo; không phá module cũ. |
Lệnh dùng PowerShell, vào đúng app. npm run db:seed:demo chỉ DB thử đã chỉ định. node --check/build/health không chứng minh E2E. Lưu lệnh, expected/actual, PASS/FAIL/SKIP/BLOCKED, môi trường và SHA tại docs/testing/tai.md hoặc file test Dxx.
Sau test đạt và commit được phép, chạy ai-context/backup-dxx.ps1. FullBackup: toàn bộ mã nguồn tracked đủ cấu trúc (trừ bí mật); ChangedFiles: ADD/MODIFY diff BASE→HEAD, DELETE/RENAME trong manifest. Tên ví dụ:
CDTN2_Nhom5_D15A_XacMinhNhaTuyenDung_FullBackup_SHA7_YYYYMMDD-HHmm.zip
CDTN2_Nhom5_D15A_XacMinhNhaTuyenDung_ChangedFiles_SHA7_YYYYMMDD-HHmm.zip
Đặt cạnh repo trong CDTN2_Backups; so manifest, giải nén thử, kiểm SHA256, không chứa bí mật. Chỉ thông báo hoàn thành sau test và hai ZIP hợp lệ. Người nhận kiểm base, đặt file, test, commit tiếng Việt/push; Tài review/merge/hồi quy. Không báo push/merge nếu thiếu chứng cứ. FullBackup không phải pg_dump.

## 12. Cổng phê duyệt và kiểm soát thay đổi

| Cổng | Chốt trước code |
|---|---|
| BR43/50 | Nguồn mã, phiên bản, FK, encoding locations, city/status/scope tương thích, legacy. |
| BR45/51 | Thuế, loại file/dung lượng/retention, minh chứng, logo, gửi lại/sửa hồ sơ đã duyệt; API-01/05/06/32/33/42. |
| BR48 | Lĩnh vực riêng/chung, API-29, scope API-07, trùng/kích hoạt lại API-39/40. |
| BR46 | Tin REJECTED gửi lại; sửa APPROVED trước/sau có đơn; công khai/lịch sử; API-11/36. |
| BR47 | Khóa NTD có tin/đơn hiện hữu, hiển thị/worker/quyền sau mở khóa; token cũ sau unlock; API-37. |
| BR50 | Duyệt cột/kiểu/FK/DTO/lỗi, API-27–42, migration/rollout; không tự APPROVED legacy. |
**Checklist:** đúng checkout, đủ tài liệu, đối chiếu SRS/Backlog/API/Schema, Dxx và phạm vi, file/tác động, chốt BR, kiểm thử AC. Cổng chưa đạt chỉ làm phần độc lập đã chốt; không sửa phần BLOCKED.
**Đồng bộ:** AGENTS.md quy định công việc và backup; quy-uoc-dat-ten-file.md mới thay quy ước Java/Flyway lỗi thời. ERD, UC, kiến trúc drawio, Sprint Plan, Bối cảnh Dxx v4.0 là tài liệu đồng bộ; Figma bỏ qua; không dùng chúng ghi đè SRS. Khi duyệt API mới, cập nhật mã chờ chốt trong Backlog/Schema và truy vết.
**Lịch sử:** v3.0 (13/11/2026) Full E2E, lịch, Matching, hủy; v4.0 (24/11/2026) Trang chủ, địa bàn, return-to, xác minh, kiểm duyệt, khóa, danh mục, Dashboard và backup Dxx. Chưa sửa repo/package/migration, chưa test HTTP/DB/Git. Script backup cần kiểm thử Windows thực.
