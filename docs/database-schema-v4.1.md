# DATABASE SCHEMA VÀ QUY TRÌNH CẬP NHẬT DATABASE

CDTN2 Nhóm 5 — Phiên bản 4.1

Nền tảng việc làm part-time cho sinh viên | Full E2E | Địa bàn chuẩn | Quản trị

| Thuộc tính | Giá trị |
| --- | --- |
| Phiên bản | 4.1 — 24/11/2026 |
| Căn cứ nghiệp vụ | SRS_Nhom5_v2.1_PhamVi_v4.1.docx: US01–US49, FR1–FR28, 35 MUST / 1 SHOULD / 13 WON’T, 119 mã AC (115 áp dụng: 113 MUST, 2 SHOULD). |
| Căn cứ kế hoạch | Product_Backlog_Nhom5_SE_v4.1_US01-US49.xlsx; 24 đợt triển khai, D15A ngoài MVP. |
| Căn cứ kỹ thuật | Database Schema v3.0/v4.0; API Contract v4.1; quyết định D00 về phạm vi và BR v4.1. |
| Database | PostgreSQL 16; DB đích student_jobs_nhom5_v4 (sao chép từ student_jobs_nhom5); schema public; Express + pg. |
| Phạm vi | 11 bảng nghiệp vụ kế thừa + 2 bảng địa bàn = 13; schema_migrations là bảng kỹ thuật, không tính. |
| Tình trạng | Thiết kế mục tiêu đã đồng bộ BR được duyệt; chưa chạy migration v4.1, chưa kiểm thử runtime. Cần đối chiếu schema/checksum/seed thực tế. |

### Mục lục

| Mục | Nội dung |
| --- | --- |
| 1–3 | Căn cứ, bảng, quy ước và các nhóm trạng thái |
| 4 | Từ điển dữ liệu đầy đủ, tích hợp cột mới |
| 5–7 | Ràng buộc, chỉ mục, thời gian, Matching và giao dịch |
| 8–9 | Migration, bảo toàn dữ liệu và kịch bản kiểm thử |
| 10–12 | Truy vết, bàn giao, SQL tham chiếu và kiểm chứng kỹ thuật |

## 1. Mục đích và căn cứ

Đây là thiết kế dữ liệu v4.1 thay thế v4.0. Các quy tắc còn phù hợp được giữ trực tiếp trong nội dung chính. Cột mới là mục tiêu triển khai, không phải bằng chứng đã tồn tại. US46/FR24/UC20/AC24.* và D15A ngoài MVP; không tạo cấu trúc xác minh NTD.

| Nguồn | Nội dung kế thừa / cập nhật |
| --- | --- |
| SRS v2.1, phạm vi v4.1 | FR1–28 (FR24 ngoài MVP), BR43/46/47/48/50/51 đã chốt, NFR6; quản trị, địa bàn, hồ sơ NTD. |
| Product Backlog v4.1 | US40/41/47/48 MUST, US46 WON’T, US49 SHOULD; 24 Dxx. |
| Database Schema v3.0 | Giữ 11 bảng, các cột, ca, CV, Matching, vòng đời và hủy tin. |
| API Contract v4.1 | API-01–26 giữ mã/method/URL; 27–29,34–42 còn phạm vi; 30–33 ngoài MVP. |
| Quy tắc công nghệ | React/Vite, Express/Node, pg, PostgreSQL, JWT; ID BIGINT truyền chuỗi; salary_unit HOUR/DAY/SHIFT/MONTH; PDF CV tối đa 5 MiB. |

Không xây thanh toán, tính lương, chấm công, chat, kho CV hoặc hạn mức tuyển. Gộp Trang chủ và khôi phục route sau đăng nhập không cần bảng riêng.

## 2. Phạm vi bảng và liên kết

| STT | Tên bảng | Vai trò | Nguồn / thay đổi |
| --- | --- | --- | --- |
| 01 | users | Tài khoản, role, khóa/mở khóa | Mở rộng V001 |
| 02 | student_profiles | Hồ sơ sinh viên dùng chung user_id | Giữ từ V001 |
| 03 | employer_profiles | Hồ sơ NTD thông thường, lĩnh vực, địa bàn, logo tùy chọn | Mở rộng V001 |
| 04 | job_categories | Danh mục ngành nghề và trạng thái sử dụng | Mở rộng V001 |
| 05 | job_posts | Tin, địa bàn, kiểm duyệt, lịch, vòng đời, hủy | Mở rộng V002 |
| 06 | job_time_slots | Ca WEEKLY hoặc ONE_TIME, 1 ca đêm = 1 bản ghi | Kế thừa V006; D07A kiểm chứng |
| 07 | student_availability_slots | Lịch rảnh tuần 0–21 khoảng | Kế thừa V006; D07B kiểm chứng |
| 08 | applications | Đơn + PDF + 5 trạng thái + lịch sử hủy | Mở rộng V003 |
| 09 | application_selected_slots | Liên kết ca SV đã chọn với đơn | Kế thừa V006 |
| 10 | reviews | Hai chiều trên application được duyệt | Giữ từ V004 |
| 11 | notifications | Thông báo trong web gắn đơn | Kế thừa V006 |
| 12 | administrative_provinces | Danh mục mã tỉnh/thành | Mới v4.1 |
| 13 | administrative_wards | Danh mục mã phường/xã/đặc khu | Mới v4.1 |

schema_migrations là bảng kỹ thuật, không tính vào 13 bảng nghiệp vụ. BR48 dùng chung job_categories; không tạo bảng business_industries. Dashboard dùng truy vấn, không tạo bảng thống kê. Không có payments, payrolls, timekeeping, chats, complaints, job_quota hoặc cvs riêng.

### 2.1. Quan hệ ERD logic

| Bảng cha | Bảng con | Khóa ngoại / bội số |
| --- | --- | --- |
| users | student_profiles, employer_profiles | user_id → users.user_id; 0..1 mỗi hồ sơ theo role |
| employer_profiles | job_posts | employer_id → employer_profiles.user_id; 1 NTD có N tin |
| job_categories | job_posts | category_id → job_categories.category_id; 1 ngành có N tin |
| job_posts | job_time_slots | job_id → job_posts.job_id; tin mới 1..21 ca |
| student_profiles | student_availability_slots | student_id → student_profiles.user_id; lịch 0..21 |
| job_posts | applications | job_id → job_posts.job_id; 1 tin có N đơn |
| student_profiles | applications | student_id → student_profiles.user_id; UQ(student_id,job_id) |
| applications, job_time_slots | application_selected_slots | application_id và job_slot_id; một đơn chọn >=1 ca của chính tin |
| applications | reviews | application_id; tối đa 2 đánh giá hợp lệ mỗi đơn |
| applications, users | notifications | application_id, recipient_id; 1 đơn có N thông báo |
| administrative_provinces | administrative_wards | province_code → administrative_provinces.province_code; 1 tỉnh có N phường |
| administrative_provinces, administrative_wards | employer_profiles, job_posts | FK tỉnh và FK ghép (province_code, ward_code) → wards(province_code, ward_code); mã NULL chỉ chuyển tiếp dữ liệu cũ |
| users | users, job_posts | FK actor Admin: locked_by, unlocked_by, moderated_by; kiểm tra role ở Service |
| job_categories | employer_profiles | industry_id → job_categories.category_id; dùng chung danh mục theo BR48 |

Các FK sử dụng ON DELETE RESTRICT, không hỗ trợ xóa cứng tin/đơn/tài khoản. Quy tắc cùng tin giữa applications và job_time_slots trong bảng liên kết được kiểm tra ở Service khi transaction; hai FK đơn lẻ không tự bảo đảm hai bên thuộc cùng một job.

## 3. Quy ước dữ liệu và trạng thái

| Nhóm | Quy ước chuẩn |
| --- | --- |
| DB và tên | DB đích student_jobs_nhom5_v4; DB nguồn student_jobs_nhom5 bảo toàn; schema public; snake_case. |
| ID và khóa | BIGINT; PK identity cho thực thể độc lập; hồ sơ dùng chung user_id; trả API dưới dạng chuỗi tránh mất chính xác JS. |
| Mốc thời gian | created_at/updated_at/applied_at/decided_at/completed_at/cancelled_at và ONE_TIME start_at/end_at: TIMESTAMPTZ. |
| Ngày / giờ tuần | schedule_start_date/end_date: DATE; WEEKLY day_of_week SMALLINT 1..7, start_time/end_time TIME không timezone. |
| Múi giờ | Xử lý ngày hủy và đối chiếu ca theo Asia/Ho_Chi_Minh; JSON timestamp ISO 8601 Z; ONE_TIME input có offset hoặc Z. |
| Lương mô tả | NUMERIC(12,2) VND; salary_unit HOUR/DAY/SHIFT/MONTH; không là dữ liệu lương thực nhận. |
| Giá trị rỗng | NULL cho dữ liệu chưa có; không đoán schedule_type của tin legacy từ work_time TEXT. |
| Quyền | Role hiện tại từ DB; mọi API bảo vệ kiểm account_status và token_version. Kiểm duyệt tin độc lập với role. |
| Mã địa bàn | VARCHAR(16), giữ số 0 đầu; seed nội bộ từ danh mục chính thức, xác minh nguồn/phiên bản/ngày hiệu lực trước seed. |
| Trạng thái mới | VARCHAR + CHECK theo cách đang dùng; không đổi sang PostgreSQL ENUM riêng. |

### 3.1. Máy trạng thái tin và đơn

| Đối tượng | Các trạng thái hợp lệ | Quy tắc |
| --- | --- | --- |
| job_posts.status | RECRUITING, FILLED, COMPLETED, EXPIRED, CANCELLED | Chỉ RECRUITING nhận đơn. FILLED không đồng nghĩa COMPLETED. CANCELLED chỉ NTD trước ngày ca đầu. |
| applications.status | PENDING, APPROVED, REJECTED, JOB_EXPIRED, JOB_CANCELLED | JOB_EXPIRED: PENDING hết cơ hội ca SV chọn; JOB_CANCELLED: hủy tin, được ghi đè kết quả cũ nhưng lưu dấu vết. |
| applications.status_before_cancel | NULL hoặc PENDING, APPROVED, REJECTED, JOB_EXPIRED | Chỉ có dữ liệu khi status=JOB_CANCELLED; giữ lại decided_at cũ. |
| users.account_status | ACTIVE, LOCKED | Chặn login/API bảo vệ khi khóa; token trước khóa vô hiệu cả sau mở khóa (BR47). |
| job_posts.moderation_status | PENDING, APPROVED, REJECTED | Tin mới PENDING, status=RECRUITING; quyết định kiểm duyệt không tự đổi vòng đời/đơn (BR46). |
| job_categories.is_active | TRUE, FALSE | FALSE không chọn mới, không xóa quan hệ lịch sử. |

Chỉ PENDING được chuyển APPROVED/REJECTED/JOB_EXPIRED. Khi hủy hợp lệ, mọi trạng thái đơn khác JOB_CANCELLED đều thành JOB_CANCELLED. Không có JOB_FILLED, NO_RESPONSE hay COMPLETED ở applications.status.

## 4. Từ điển dữ liệu mục tiêu — 13 bảng nghiệp vụ

Giữ các kiểu/ràng buộc nền; cột mới v4.1 là mục tiêu sau chuyển tiếp. Không thêm NOT NULL cứng vào dữ liệu cũ chưa được kiểm kê.

### 4.1. users

Mở rộng baseline V001; tài khoản ba vai trò, không đăng ký ADMIN công khai. FR26 ghi nhận cả thao tác khóa và mở khóa.

| Cột | Kiểu PostgreSQL | NULL | Khóa / ràng buộc / diễn giải |
| --- | --- | --- | --- |
| user_id | BIGINT | Không | PK, GENERATED BY DEFAULT AS IDENTITY. |
| email | VARCHAR(254) | Không | UNIQUE; email = lower(btrim(email)); không rỗng. |
| password_hash | VARCHAR(255) | Không | Không trả JSON hoặc lưu mật khẩu rõ. |
| role | VARCHAR(16) | Không | CHECK IN ('STUDENT','EMPLOYER','ADMIN'). |
| created_at | TIMESTAMPTZ | Không | DEFAULT CURRENT_TIMESTAMP. |
| account_status | VARCHAR(16) | Không | Mới v4.1; DEFAULT ACTIVE; CHECK ACTIVE/LOCKED. |
| locked_at | TIMESTAMPTZ | Có | Mới v4.1; thời điểm lần khóa gần nhất. |
| locked_by | BIGINT | Có | Mới v4.1; FK users.user_id; Admin thực hiện lần khóa gần nhất. |
| lock_reason | TEXT | Có | Mới v4.1; lý do lần khóa gần nhất, trim không rỗng khi khóa. |
| unlocked_at | TIMESTAMPTZ | Có | Mới v4.1; thời điểm lần mở khóa gần nhất. |
| unlocked_by | BIGINT | Có | Mới v4.1; FK users.user_id; Admin thực hiện lần mở khóa gần nhất. |
| token_version | INTEGER | Không | DEFAULT 0; CHECK token_version >= 0; phiên bản thu hồi JWT theo BR47; tăng nguyên tử khi khóa. Không trả DTO công khai. |

• locked_at/by/reason cùng có hoặc cùng NULL; unlocked_at/by cùng có hoặc cùng NULL. LOCKED cần đủ metadata khóa. ACTIVE mới metadata NULL; mở khóa giữ lần khóa gần nhất, ghi lần mở khóa. Admin không khóa Admin. Khi ACTIVE→LOCKED, tăng token_version cùng transaction; khi mở khóa giữ version mới. JWT claim token_version phải là số nguyên không âm và khớp DB; token thiếu claim sau rollout bị từ chối 401 INVALID_TOKEN, token của tài khoản LOCKED bị chặn 403 ACCOUNT_LOCKED. Login mới cấp token với version hiện tại. Kiểm tra lại sau lấy khóa ở luồng ghi; metadata không phải audit log đầy đủ.

### 4.2. student_profiles

Một hồ sơ cho tài khoản STUDENT; auto tạo cùng transaction đăng ký (API-01).

| Cột | Kiểu PostgreSQL | NULL | Khóa / ràng buộc / diễn giải |
| --- | --- | --- | --- |
| user_id | BIGINT | Không | PK + FK → users(user_id), không identity. |
| full_name | VARCHAR(150) | Không | CHECK trim không rỗng. |
| phone | VARCHAR(11) | Có | NULL hoặc 10–11 chữ số. |
| introduction | TEXT | Có | Nội dung tùy chọn, kiểm tra ≤5.000 ký tự ở Service. |
| updated_at | TIMESTAMPTZ | Không | DEFAULT CURRENT_TIMESTAMP; Service cập nhật khi PATCH. |

### 4.3. employer_profiles

Một hồ sơ cho tài khoản EMPLOYER; liên kết chủ sở hữu job_posts.

| Cột | Kiểu PostgreSQL | NULL | Khóa / ràng buộc / diễn giải |
| --- | --- | --- | --- |
| user_id | BIGINT | Không | PK + FK → users(user_id). |
| display_name | VARCHAR(200) | Không | CHECK trim không rỗng. |
| contact_name | VARCHAR(150) | Có (legacy) | Bắt buộc, trim 1–150 với NTD mới; giữ nullable khi chuyển tiếp. |
| phone | VARCHAR(11) | Có (legacy) | Bắt buộc với NTD mới; 10–11 chữ số, không kiểu số. |
| introduction | TEXT | Có | Giới thiệu, tối đa 5.000 ở Service. |
| updated_at | TIMESTAMPTZ | Không | DEFAULT CURRENT_TIMESTAMP. |
| business_type | VARCHAR(20) | Có (legacy) | Mới v4.1; COMPANY/INDIVIDUAL_SHOP; bắt buộc NTD mới. |
| tax_code | VARCHAR(50) | Có | Mới v4.1; tùy chọn mọi loại hình, chuỗi tối đa 50, không UNIQUE. |
| industry_id | BIGINT | Có (legacy) | Mới v4.1; bắt buộc NTD mới; FK job_categories.category_id, chọn mới danh mục active. |
| province_code | VARCHAR(16) | Có (legacy) | Mới v4.1; FK administrative_provinces; bắt buộc NTD mới. |
| ward_code | VARCHAR(16) | Có (legacy) | Mới v4.1; FK ghép cùng province_code; bắt buộc NTD mới. |
| address_detail | VARCHAR(255) | Có (legacy) | Mới v4.1; trim không rỗng với NTD mới. |
| logo_storage_key | VARCHAR(255) | Có | Mới v4.1; khóa file do server tạo, không phải URL do client nhập. |
| website_url | VARCHAR(500) | Có | Mới v4.1; URL http/https hợp lệ; không tự tải link từ server. |

• Giữ user_id là PK/FK, không tạo bảng employers hoặc đổi tên display_name/contact_name/phone/introduction. Không lưu email/password trong profile.

• NTD đăng ký tài khoản ACTIVE và một hồ sơ hợp lệ, không có quy trình duyệt hồ sơ; không yêu cầu giấy tờ hoặc mã số thuế.

• Mã địa bàn phải cùng quan hệ cha–con. Logo tùy chọn JPG/PNG/WebP ≤2 MiB, kiểm tra chữ ký file; lưu key do server tạo, đọc qua API-42; không lưu minh chứng NTD.

• Lĩnh vực doanh nghiệp dùng chung job_categories theo BR48; khi tạo/đổi industry_id chỉ chọn danh mục đang hoạt động; giữ FK lịch sử.

### 4.4. job_categories

Danh mục dùng chung cho ngành nghề tuyển dụng và lĩnh vực NTD (BR48); API-07 và API-29 đọc cùng nguồn.

| Cột | Kiểu PostgreSQL | NULL | Khóa / ràng buộc / diễn giải |
| --- | --- | --- | --- |
| category_id | BIGINT | Không | PK, identity. |
| name | VARCHAR(100) | Không | UNIQUE; CHECK trim không rỗng. |
| is_active | BOOLEAN | Không | Mới v4.1; DEFAULT TRUE; FALSE không chọn cho tin/hồ sơ mới hoặc khi đổi danh mục. |

• API-07 scope=selection chỉ active; scope=filter gồm cả inactive để lọc tin cũ. API-29 ánh xạ category_id → industry_id chỉ active. Admin 38–40 quản lý, kiểm trùng tên sau trim không phân biệt hoa/thường; đối chiếu trùng legacy trước unique index. Không DELETE cứng danh mục được tham chiếu.

### 4.5. job_posts

Giữ toàn bộ cột v3.0. Bổ sung mã địa bàn và kiểm duyệt; không đổi status vòng đời; không thêm required_workers.

| Cột | Kiểu PostgreSQL | NULL | Khóa / ràng buộc / diễn giải |
| --- | --- | --- | --- |
| job_id | BIGINT | Không | PK, identity; giữ từ V002. |
| employer_id | BIGINT | Không | FK → employer_profiles(user_id); chủ tin lấy từ JWT. |
| category_id | BIGINT | Không | FK → job_categories(category_id). |
| title | VARCHAR(200) | Không | CHECK trim không rỗng. |
| description | TEXT | Không | CHECK trim không rỗng; ≤20.000 ở Service. |
| work_time | TEXT | Không | Giữ TEXT mô tả hiển thị, ≤5.000; KHÔNG parse để tính lịch. |
| schedule_type | VARCHAR(16) | Có | NEW; WEEKLY hoặc ONE_TIME; NULL chỉ cho tin legacy thiếu lịch. |
| schedule_mode | VARCHAR(8) | Không | DEFAULT ANY; CHECK ANY/ALL; chỉ phục vụ Matching. |
| schedule_start_date | DATE | Có | NEW; bắt buộc với WEEKLY; NULL ở ONE_TIME/legacy. |
| schedule_end_date | DATE | Có | NEW; WEEKLY tùy chọn, phải ≥ ngày bắt đầu; NULL với ONE_TIME/legacy. |
| salary_min | NUMERIC(12,2) | Không | ≥0; chỉ mô tả mức lương đăng tuyển. |
| salary_max | NUMERIC(12,2) | Không | ≥salary_min. |
| salary_unit | VARCHAR(10) | Không | HOUR, DAY, SHIFT, MONTH. |
| city | VARCHAR(100) | Không | CHECK trim không rỗng. |
| location_detail | VARCHAR(255) | Không | CHECK trim không rỗng. |
| status | VARCHAR(16) | Không | DEFAULT RECRUITING; 5 giá trị ở mục 3.1. |
| completed_at | TIMESTAMPTZ | Có | Có giá trị chỉ khi COMPLETED, ≥created_at. |
| cancelled_at | TIMESTAMPTZ | Có | NEW; có giá trị chỉ khi CANCELLED, ≥created_at. |
| cancel_reason | TEXT | Có | NEW; lý do NTD nhập, bắt buộc nếu đã có bất kỳ đơn; chiều dài do API/service kiểm soát. |
| created_at | TIMESTAMPTZ | Không | DEFAULT CURRENT_TIMESTAMP. |
| updated_at | TIMESTAMPTZ | Không | DEFAULT CURRENT_TIMESTAMP; Service set khi UPDATE. |
| province_code | VARCHAR(16) | Có (legacy) | Mới v4.1; FK tỉnh, bắt buộc tin mới. |
| ward_code | VARCHAR(16) | Có (legacy) | Mới v4.1; FK ghép tỉnh/phường, bắt buộc tin mới. |
| moderation_status | VARCHAR(16) | Không (sau chuyển tiếp) | Mới v4.1; DEFAULT PENDING cho tin mới, CHECK PENDING/APPROVED/REJECTED. |
| moderation_reason | TEXT | Có | Mới v4.1; lý do từ chối bắt buộc khi REJECTED; NULL ở PENDING/APPROVED. |
| moderated_at | TIMESTAMPTZ | Có | Mới v4.1; mốc quyết định Admin APPROVED/REJECTED. |
| moderated_by | BIGINT | Có | Mới v4.1; FK users.user_id, role ADMIN kiểm ở Service. |

• Giữ city/location_detail; city tin mới do server lấy tên tỉnh chuẩn. Không đoán mã phường từ địa chỉ cũ. Tin legacy mã NULL không khớp lọc mã tương ứng; tin đủ điều kiện vẫn đọc được khi không lọc địa bàn.

• PENDING: metadata quyết định NULL; APPROVED: actor/thời điểm; REJECTED: thêm lý do. Công khai cần moderation_status=APPROVED và chủ tin EMPLOYER ACTIVE; danh sách mặc định/Matching/nhận đơn thêm RECRUITING và còn cơ hội ca (ngoại lệ legacy theo FR4, không Matching). Tin RECRUITING chưa có đơn sửa hợp lệ về PENDING, xóa metadata hiện tại; có bất kỳ đơn nào khóa toàn bộ sửa nội dung, không đổi vòng đời/đơn. BR47 ẩn tin NTD LOCKED; quyền đọc lịch sử riêng vẫn được bảo toàn.

• Shape ngày hiệu lực: schedule_type=NULL → hai DATE NULL (tin legacy); ONE_TIME → hai DATE NULL; WEEKLY → start_date bắt buộc, end_date tùy chọn. Dữ liệu tin mới bắt buộc schedule_type và 1–21 ca (Service).

• CHECK trạng thái: COMPLETED phải có completed_at và không có cancelled_at/cancel_reason; CANCELLED phải có cancelled_at và completed_at NULL; các trạng thái khác không có hai timestamp kết thúc. Service xác thực lý do hủy khi có đơn.

• EXPIRED không cần cột expired_at trong API v3.0; xác định nhờ status và lịch ca. Không tự suy thời điểm công thực tế hoặc lương.

### 4.6. job_time_slots

Mỗi hàng là MỘT ca logic thuộc một tin. Cùng bảng hỗ trợ WEEKLY và ONE_TIME; không tách ca qua đêm.

| Cột | Kiểu PostgreSQL | NULL | Khóa / ràng buộc / diễn giải |
| --- | --- | --- | --- |
| job_slot_id | BIGINT | Không | PK, identity; API trả ID dạng string. |
| job_id | BIGINT | Không | FK → job_posts(job_id), ON DELETE RESTRICT. |
| day_of_week | SMALLINT | Có | WEEKLY: 1..7 (T2..CN); ONE_TIME: NULL. |
| start_time | TIME | Có | WEEKLY: HH:mm, bắt buộc; ONE_TIME: NULL. |
| end_time | TIME | Có | WEEKLY: HH:mm, khác start_time; nhỏ hơn start_time là ca đêm. |
| start_at | TIMESTAMPTZ | Có | ONE_TIME bắt buộc, có ngày và giờ; WEEKLY NULL. |
| end_at | TIMESTAMPTZ | Có | ONE_TIME bắt buộc, >start_at; WEEKLY NULL. |

• CHECK một trong hai shape duy nhất: [day_of_week, start_time, end_time] đều NOT NULL và [start_at,end_at] đều NULL, HOẶC ngược lại. Service đối chiếu shape với job_posts.schedule_type và giờ ranh giới Asia/Ho_Chi_Minh.

• UNIQUE(job_id,day_of_week,start_time,end_time) cho WEEKLY và UNIQUE(job_id,start_at,end_at) cho ONE_TIME; các UNIQUE không loại trừ ca chồng một phần, Service so sánh toàn bộ lịch kể cả ca đêm và CN→T2.

• Mỗi tin mới 1–21 ca logic, ca kề nhau được, không cho trùng/chồng. job_slot_id không đổi sau khi đã có application do khóa sửa lịch.

### 4.7. student_availability_slots

Lịch rảnh WEEKLY của SV; có thể [] (chưa khai báo); độc lập lịch tin và không giới hạn quyền ứng tuyển.

| Cột | Kiểu PostgreSQL | NULL | Khóa / ràng buộc / diễn giải |
| --- | --- | --- | --- |
| availability_slot_id | BIGINT | Không | PK, identity. |
| student_id | BIGINT | Không | FK → student_profiles(user_id). |
| day_of_week | SMALLINT | Không | CHECK 1..7, 1=T2 và 7=CN. |
| start_time | TIME | Không | HH:mm giờ VN. |
| end_time | TIME | Không | Khác start_time; nhỏ hơn là qua đêm 1 khoảng logic. |

• UNIQUE(student_id,day_of_week,start_time,end_time), CHECK start_time<>end_time; không lưu TIME 24:00 và không dùng end_time>start_time theo quy tắc cũ.

• API-25 PUT thay toàn bộ lịch 0–21 khoảng; kiểm tra overlap cả Chủ nhật→Thứ Hai và transaction DELETE/INSERT, không xóa lịch cũ nếu request sai.

• Không công khai lịch SV cho NTD hay SV khác; API-26 lấy lịch đúng student_id từ JWT.

### 4.8. applications

Đơn + CV PDF của một SV ứng tuyển một tin; trạng thái ứng tuyển độc lập trạng thái tin. Giữ các cột CV đã có ở V003.

| Cột | Kiểu PostgreSQL | NULL | Khóa / ràng buộc / diễn giải |
| --- | --- | --- | --- |
| application_id | BIGINT | Không | PK, identity. |
| job_id | BIGINT | Không | FK → job_posts(job_id). |
| student_id | BIGINT | Không | FK → student_profiles(user_id). |
| status | VARCHAR(16) | Không | DEFAULT PENDING; 5 trạng thái đơn hợp lệ. |
| status_before_cancel | VARCHAR(16) | Có | NEW; chỉ lưu khi JOB_CANCELLED; giữ trạng thái ngay trước hủy. |
| cv_storage_key | VARCHAR(255) | Không | UNIQUE; server tạo, không công khai storage path. |
| cv_original_name | VARCHAR(255) | Không | Tên file gốc, CHECK không rỗng. |
| cv_mime_type | VARCHAR(100) | Không | CHECK = 'application/pdf'. |
| cv_size_bytes | INTEGER | Không | CHECK 1..5.242.880. |
| applied_at | TIMESTAMPTZ | Không | DEFAULT CURRENT_TIMESTAMP. |
| decided_at | TIMESTAMPTZ | Có | Thời điểm NTD APPROVED/REJECTED; không gán khi JOB_EXPIRED; không bị ghi đè khi hủy. |

• UNIQUE(student_id,job_id), UNIQUE(cv_storage_key). Không cho ứng tuyển lại vào một tin dù trạng thái đã REJECTED/JOB_EXPIRED/JOB_CANCELLED.

• CHECK decided_at: PENDING/JOB_EXPIRED → NULL; APPROVED/REJECTED → NOT NULL và >=applied_at; JOB_CANCELLED dựa status_before_cancel: quyết định cũ APPROVED/REJECTED giữ decided_at, cũ PENDING/JOB_EXPIRED giữ NULL.

• Đối với tin v3.0 có ca, 1 đơn chọn ≥1 ca và tối đa số ca của tin; legacy không có ca được phép selected_slots=[] theo API-13.

• Không tạo approved_at riêng: decided_at đã là mốc NTD duyệt; ca ONE_TIME hiệu lực chỉ nếu start_at > decided_at.

### 4.9. application_selected_slots

Bảng liên kết mới lưu lựa chọn ca ban đầu của SV; không tự xóa khi ca qua giờ, đơn hết hạn hay tin hủy.

| Cột | Kiểu PostgreSQL | NULL | Khóa / ràng buộc / diễn giải |
| --- | --- | --- | --- |
| application_id | BIGINT | Không | FK → applications(application_id) ON DELETE RESTRICT; một phần khóa chính. |
| job_slot_id | BIGINT | Không | FK → job_time_slots(job_slot_id) ON DELETE RESTRICT; một phần khóa chính. |

• PRIMARY KEY(application_id, job_slot_id) đồng thời là UNIQUE; không cần ID riêng, không có approve_status riêng từng ca.

• Backend bắt buộc kiểm tra job_id của đơn và job_id của ca giống nhau trong transaction; FK đơn lẻ không chứng minh hai bản ghi cùng tin. Khóa sửa ca của job_posts khi có bất kỳ đơn giúp liên kết lịch sử ổn định.

• Khi duyệt muộn, không cập nhật/xóa bảng này; giao diện tính hiệu lực dựa vào decided_at + thời điểm bắt đầu của từng ca/lần WEEKLY.

### 4.10. reviews

Giữ V004; hai chiều đánh giá một đơn được duyệt và tin COMPLETED; không thêm tính lương.

| Cột | Kiểu PostgreSQL | NULL | Khóa / ràng buộc / diễn giải |
| --- | --- | --- | --- |
| review_id | BIGINT | Không | PK, identity. |
| application_id | BIGINT | Không | FK → applications(application_id). |
| reviewer_id | BIGINT | Không | FK → users(user_id); lấy từ JWT. |
| reviewee_id | BIGINT | Không | FK → users(user_id); suy từ đơn và chủ tin. |
| rating | SMALLINT | Không | CHECK 1..5. |
| comment | TEXT | Có | ≤2.000 ký tự theo Service. |
| created_at | TIMESTAMPTZ | Không | DEFAULT CURRENT_TIMESTAMP. |

• UNIQUE(application_id, reviewer_id) và CHECK reviewer_id<>reviewee_id. Service xác định đúng người đánh giá và người được đánh giá.

• Chỉ tạo review nếu applications.status=APPROVED và job_posts.status=COMPLETED; JOB_CANCELLED/EXPIRED không cho đánh giá.

### 4.11. notifications

Giữ đúng cấu trúc tối thiểu 5 cột từ Schema v2.3 / API v3.0; không tự thêm type, is_read, email hoặc websocket.

| Cột | Kiểu PostgreSQL | NULL | Khóa / ràng buộc / diễn giải |
| --- | --- | --- | --- |
| notification_id | BIGINT | Không | PK, identity. |
| application_id | BIGINT | Không | FK → applications(application_id). |
| recipient_id | BIGINT | Không | FK → users(user_id), phải là SV của đơn. |
| message | TEXT | Không | CHECK trim không rỗng; khi hủy chứa lý do NTD. |
| created_at | TIMESTAMPTZ | Không | DEFAULT CURRENT_TIMESTAMP. |

• Tạo notification cùng transaction khi APPROVED/REJECTED, JOB_EXPIRED, CANCELLED; một sự kiện không được nhân thông báo khi worker chạy lại.

• API-22 chỉ trả thông báo của user từ JWT; notifications giữ cả lịch sử thông báo dù đơn sau đó JOB_CANCELLED.

### 4.12. administrative_provinces — mới v4.1

Danh mục địa bàn cấp tỉnh. Chỉ seed từ bộ mã/tên có nguồn, phiên bản và ngày hiệu lực đã được xác nhận; tài liệu này không cung cấp bộ dữ liệu địa giới chính thức.

| Cột | Kiểu PostgreSQL | NULL | Khóa / ràng buộc / diễn giải |
| --- | --- | --- | --- |
| province_code | VARCHAR(16) | Không | PK, mã chuỗi giữ số 0 đầu. |
| name | VARCHAR(200) | Không | Tên chính thức, trim không rỗng. |
| source_version | VARCHAR(100) | Không | Phiên bản bộ dữ liệu đã nhập; không tự tạo mã. |
| is_active | BOOLEAN | Không | DEFAULT TRUE; giữ mã cũ để đọc lịch sử, không chọn mới. |

### 4.13. administrative_wards — mới v4.1

Danh mục cấp phường/xã/đặc khu; không thêm bảng quận/huyện vào mô hình hai cấp.

| Cột | Kiểu PostgreSQL | NULL | Khóa / ràng buộc / diễn giải |
| --- | --- | --- | --- |
| ward_code | VARCHAR(16) | Không | PK; kiểm chứng mã duy nhất toàn bộ nguồn trước seed; nếu không đạt phải báo thay đổi PK. |
| province_code | VARCHAR(16) | Không | FK administrative_provinces.province_code. |
| name | VARCHAR(200) | Không | Tên chính thức; không UNIQUE toàn quốc. |
| source_version | VARCHAR(100) | Không | Phiên bản bộ dữ liệu đã nhập. |
| is_active | BOOLEAN | Không | DEFAULT TRUE; không xóa địa bàn đang tham chiếu. |

• UNIQUE(province_code, ward_code) phục vụ FK ghép tại employer_profiles và job_posts; MATCH SIMPLE cho legacy, CHECK ward_code IS NULL OR province_code IS NOT NULL. Hồ sơ/tin mới cần đủ hai mã active. Kiểm chứng phiên bản nguồn trước seed; nếu mã không duy nhất toàn quốc phải báo trước khi đổi PK.

• Lọc nhóm tỉnh (toàn tỉnh hoặc danh sách phường), OR giữa nhóm, AND với bộ lọc khác. Không gộp hai mảng phẳng; legacy chỉ tỉnh đáng tin có thể khớp toàn tỉnh. API-09/26 dùng locations JSON text URL-encoded; giới hạn 34 nhóm và 200 mã phường.

Dashboard FR28 không có bảng riêng: năm số đếm từ users, job_posts, applications. Không lưu số đếm giả.

## 5. Ràng buộc quan hệ, quyền và index

### 5.1. DB bảo đảm và Service bắt buộc kiểm tra

| Lớp | Bảo đảm | Không được nhầm |
| --- | --- | --- |
| PostgreSQL | PK/FK/NOT NULL/UNIQUE/CHECK từng hàng, enum 5 trạng thái, shape ca, tính hợp lệ ngày và giờ, ràng buộc CV metadata. | CHECK không đủ kiểm tra overlap toàn lịch, số lượng tối đa 21, quyền sở hữu hoặc status liên bảng. |
| Validator / Service | role, account_status, token_version, chủ tin, kiểm duyệt, quyền file, địa bàn hoạt động, lịch cùng tin, ca tương lai, overlap, hạn hủy, chuyển trạng thái hợp lệ. | BE kiểm tra lại, không chỉ ẩn nút FE. |
| Transaction / Lock | users theo ID tăng → employer_profiles → danh mục → job_posts → applications theo ID tăng; kiểm lại quyền/trạng thái sau khóa. | Worker chỉ khóa job rồi đơn, không khóa ngược. |
| Worker backend | Tự JOB_EXPIRED theo ca SV chọn; tự FILLED/EXPIRED sau ca cuối toàn tin; idempotent. | Không phụ thuộc browser hay tự chuyển COMPLETED. |

### 5.2. Chỉ mục cần có, không trùng index nền V005

| Index / unique | Cột / ý nghĩa | Nguồn |
| --- | --- | --- |
| idx_jobs_employer_created | job_posts(employer_id,created_at DESC) | V005 — giữ |
| idx_jobs_category | job_posts(category_id) | V005 — giữ |
| idx_jobs_status_created | job_posts(status,created_at DESC) | V005 — giữ |
| idx_apps_job_status | applications(job_id,status) | V005 — giữ |
| idx_apps_student_applied | applications(student_id,applied_at DESC) | V005 — giữ |
| idx_reviews_reviewee / idx_reviews_reviewer | reviews(reviewee_id); reviews(reviewer_id) | V005 — giữ |
| idx_job_slots_by_time | job_time_slots(job_id,day_of_week,start_time,end_time) | Mới khi tạo slot |
| idx_job_slots_by_start_at | job_time_slots(job_id,start_at) | Mới, ONE_TIME |
| idx_student_slots_by_time | student_availability_slots(student_id,day_of_week,start_time,end_time) | Mới |
| idx_app_selected_by_slot | application_selected_slots(job_slot_id) | Mới; tra cứu ca→đơn |
| idx_notifs_recipient_created | notifications(recipient_id,created_at DESC) | Mới nếu thiếu |
| idx_notifs_application | notifications(application_id) | Mới nếu thiếu |
| idx_jobs_geo_v4 | job_posts(province_code,ward_code) | Đề xuất; lọc theo tỉnh/phường, đo EXPLAIN. |
| idx_jobs_moderation_status_created_v4 | job_posts(moderation_status,status,created_at DESC,job_id DESC) | Đề xuất; hàng đợi duyệt và danh sách công khai. |
| idx_users_role_account_status_v4 | users(role,account_status,user_id) | Đề xuất; Admin lọc. |
| idx_wards_province_v4 | administrative_wards(province_code,ward_code) | Không tạo trùng UNIQUE ghép; dùng index của UNIQUE. |

PK/UNIQUE tự tạo index. Khi migration chạy phải kiểm tra pg_indexes / schema_migrations để không tạo lại index đã tồn tại; cần cân nhắc partial index cho ONE_TIME sau khi đo bằng EXPLAIN, không ép tối ưu sớm.

## 6. Lịch ca, thời gian, Matching và duyệt muộn

### 6.1. WEEKLY và ONE_TIME theo một hợp đồng

| Loại | Cách lưu | Quy tắc thời gian |
| --- | --- | --- |
| WEEKLY | job_posts.schedule_type=WEEKLY; DATE start/end; job_time_slots.day_of_week, start_time, end_time | Mỗi ca lặp theo thứ, ngày bắt đầu ca thuộc [start_date,end_date] nếu có E; ca qua đêm có thể kết thúc vào ngày kế tiếp. |
| ONE_TIME | schedule_type=ONE_TIME; job_time_slots.start_at/end_at TIMESTAMPTZ | Mỗi ca có timestamp đầy đủ; start_at<end_at; ca đã bắt đầu không được chọn mới. |
| SV rảnh | student_availability_slots (day_of_week,start_time,end_time) | 0–21 khoảng tuần, ca qua đêm một record; không dùng DATE vì SV khai lịch lặp. |

Ca WEEKLY 22:00–02:00 được hiểu kéo đến ngày hôm sau (trục giờ 48h); Sunday 22:00 tới Monday 02:00 vẫn là một ca logic. Chống overlap theo chu kỳ 7 ngày và so sánh một ca trọn vẹn nằm trong một khoảng rảnh liên tục; không ghép hai khoảng rời.

Ví dụ: SV rảnh Chủ nhật 21:00–Thứ Hai 03:00, công việc Chủ nhật 22:00–Thứ Hai 02:00 → khớp 1/1. Với ONE_TIME, đổi start_at thực tế sang thứ/giờ tại Asia/Ho_Chi_Minh; không lấy thứ của ngày UTC để quyết định matching.

### 6.2. Các mốc thời gian là KHÁC NHAU

| Mốc | Định nghĩa chính xác | Tác động |
| --- | --- | --- |
| Ca ONE_TIME đã bắt đầu | now >= start_at của ca cụ thể | Không cho SV mới chọn ca đó; nếu còn ca tương lai, job vẫn RECRUITING. |
| Đơn PENDING mất cơ hội | Mọi ca SV đã chọn / lần WEEKLY còn hiệu lực đều có thời điểm bắt đầu <= now | PENDING → JOB_EXPIRED + notification; có thể trong khi job vẫn RECRUITING. |
| Duyệt muộn | Còn ít nhất 1 ca SV chọn có lần bắt đầu > now | NTD xác nhận popup, quyết định APPROVED, decided_at=now; ca trước đó không có hiệu lực. |
| Tin tự kết thúc | now >= thời điểm KẾT THÚC thực tế của ca cuối toàn tin | RECRUITING → FILLED nếu ≥1 APPROVED; ngược lại → EXPIRED. |
| Hủy chủ động | DATE(now tại Asia/Ho_Chi_Minh) < DATE(ca đầu bắt đầu tại Asia/Ho_Chi_Minh) | Chỉ chủ tin được hủy RECRUITING/FILLED; khi đã tới ngày ca đầu thì chặn, dù chưa đến giờ làm. |

WEEKLY không có schedule_end_date: tin không tự FILLED/EXPIRED vì thời gian; đơn PENDING có thể chờ NTD vô hạn theo yêu cầu đã chốt. Tin legacy không có ca cấu trúc không có mốc đầu/cuối đáng tin cậy: không được parse work_time để chạy tự hết hạn hay hủy; trả lỗi API đã định nghĩa cho lịch không xác định.

### 6.3. Quy tắc gợi ý API-26 (US44/FR22)

Chỉ xét job_posts.status=RECRUITING, moderation_status=APPROVED, đủ điều kiện công khai theo FR3/BR44/BR47, có ít nhất một ca còn ứng tuyển được và SV đã lưu lịch rảnh. Ca ONE_TIME đã bắt đầu không nằm trong matched hoặc total.

ANY: matched_slots_count ≥1; ALL: matched_slots_count=total_slots_count; match_percent=ROUND(100×matched/total), total>0.

Không lưu match_percent thành cột; chỉ tính từ job_time_slots × student_availability_slots và job_posts; sort % DESC, created_at DESC, job_id DESC.

Matching không chặn tự tìm tin hoặc API-13 chọn ca; không tự kiểm tra giới hạn người tuyển vì không có required_workers.

## 7. Transaction và những luồng ghi dữ liệu chính

| Sự kiện | Cách xử lý trong BE/DB | API / truy vết |
| --- | --- | --- |
| Đăng ký | INSERT users ACTIVE + đúng 1 profile theo role; logo tùy chọn; rollback cả DB và dọn file mới nếu lỗi. | API-01 / US01–02 |
| Đăng tin | Khóa user/profile NTD, role EMPLOYER + ACTIVE, token_version hợp lệ, danh mục và địa bàn active; INSERT job RECRUITING/PENDING + 1–21 ca cùng transaction. | API-08 / US06,43 |
| Sửa tin | Chủ tin ACTIVE, RECRUITING, chưa có bất kỳ đơn; lưu job/ca nguyên tử, set PENDING và xóa metadata quyết định hiện tại; có đơn chặn toàn bộ sửa nội dung. | API-11 / US07 |
| Thay lịch rảnh | Xác thực STUDENT; kiểm tra toàn bộ 0–21 slot; lock profile, DELETE/INSERT cùng transaction. | API-25 / US42 |
| Nộp đơn | Khóa và kiểm tra lại quyền, chủ tin và tin; APPROVED + RECRUITING + đủ điều kiện nhận đơn; INSERT application + selected_slots; bù trừ file CV nếu lỗi. | API-13 / US12 |
| Duyệt / từ chối | Khóa user liên quan rồi job/đơn; chỉ PENDING; không duyệt mới SV LOCKED; duyệt khi còn ca tương lai, xác nhận duyệt muộn; ghi quyết định + notification cùng transaction. | API-18 / US14–15 |
| JOB_EXPIRED | Worker khóa tin rồi đơn; nếu PENDING hết mọi cơ hội trong các ca đã chọn, chuyển JOB_EXPIRED + INSERT notification; lần chạy lại không trùng. | Worker / US16,22 |
| Đóng tuyển thủ công | Khóa tin → phải ≥1 APPROVED, 0 PENDING; đổi RECRUITING→FILLED. KHÔNG tự từ chối đơn còn chờ. | API-19 / US18 |
| Tin tự hết lịch | Worker xử lý các đơn PENDING hết cơ hội, rồi nếu RECRUITING và sau ca cuối: APPROVED ≥1 → FILLED, =0 → EXPIRED; thông báo SV bị ảnh hưởng cùng transaction theo FR8, tránh gửi trùng; chi tiết sự kiện theo API v4.1. | Worker / US18 |
| Hủy tin | Khóa tin rồi tất cả đơn; so sánh local DATE trước ngày ca đầu; lý do bắt buộc nếu có đơn; lưu status_before_cancel, JOB_CANCELLED toàn bộ, notifications từng SV và job CANCELLED cùng transaction. | API-19 / US45 |
| Xác nhận hoàn thành | Chỉ chủ tin, từ FILLED→COMPLETED sau công việc thực tế; completed_at, không tự động. | API-19 / US19 |
| Đánh giá | Chỉ APPROVED + COMPLETED; một người 1 review/đơn; Service suy reviewer/reviewee theo quan hệ. | API-20–21 / US20–21 |
| Admin duyệt tin | Khóa user/profile chủ tin rồi job; Admin ACTIVE, tin PENDING; khi APPROVED chủ tin ACTIVE và đủ điều kiện hiện tại; ghi metadata, không đổi vòng đời/đơn. | FR25 / US47 / API-36 |
| Admin khóa/mở khóa | Khóa users; chỉ SV/NTD, kiểm tra trạng thái cũ; khóa tăng token_version, ghi metadata cùng transaction; mở giữ version. Ẩn tin, chặn đơn mới/thao tác thủ công khi NTD LOCKED; giữ lịch sử và worker. | FR26 / US40–41 / API-37 |
| Admin danh mục | Khóa category; kiểm trùng; tạo/sửa/ngừng/kích hoạt; không xóa FK lịch sử. Tạo/đổi tin hoặc lĩnh vực kiểm active trong transaction. | FR27 / US48 / API-38–40 |
| Dashboard | COUNT năm chỉ số FR28 từ DB thật, cùng snapshot đọc nếu cần nhất quán; không ghi DB. | FR28 / US49 / API-41 (SHOULD) |

Hạn chế PostgreSQL: file PDF lưu trong private storage (APP_STORAGE_DIR; đường dẫn CV kế thừa cần kiểm kê), không thuộc transaction DB. Nếu DB rollback xóa file mới; kiểm tra file mồ côi có xác nhận; không báo thành công khi chưa lưu đủ.

Logo cũng nằm ngoài transaction DB: key ngẫu nhiên, kiểm tra chữ ký/size, ghi DB khi file sẵn sàng; rollback dọn file mới, file cũ chỉ dọn sau commit và khi không còn tham chiếu. Không log key/file thật. Dữ liệu demo giữ qua nghiệm thu, dọn có xác nhận; không tự xóa file thật. Không mở rộng notifications/email ngoài phạm vi.

### 7.1. Đồng thời và idempotency

1. BEGIN → khóa users liên quan theo user_id tăng → employer_profiles cần thiết → danh mục khi tạo/đổi → job_posts → applications theo ID tăng → ghi thay đổi/thông báo → COMMIT. Kiểm tra lại role, account_status, token_version và predicate sau khóa. Worker chỉ khóa job rồi đơn, không lấy user/profile ngược; retry deadlock hữu hạn có test.

2.  Đối với worker, chỉ cập nhật khi status=PENDING/RECRUITING tương ứng và ghi thông báo nếu UPDATE thực sự đổi trạng thái; worker chạy lại không tạo duplicate.

3.  Nếu một notification INSERT lỗi, ROLLBACK cả job_posts, applications, notification; không commit từng SV. Khi cạnh tranh duyệt vs JOB_EXPIRED vs CANCELLED, hành động thực hiện sau khóa phải đọc trạng thái mới nhất.

4.  Duyệt muộn khi đã có ca qua giờ: giữ application_selected_slots bất biến; determined effective slot bằng start_at > decided_at với ONE_TIME hoặc lần lặp WEEKLY có start_at tương lai sau mốc duyệt.

## 8. Kế hoạch migration và tương thích dữ liệu v4.1

### 8.1. Kiểm kê trước khi viết migration (D00)

| Nguồn | Việc kiểm tra |
| --- | --- |
| Git | git status --short --branch; git rev-parse HEAD; git log -1; so với main đã xác minh. Không suy ra SHA từ tên ZIP. |
| Migration | Kiểm tra backend/migrations, backend/scripts/migrate.js, schema_migrations và constraint thực. Kết quả pgAdmin người dùng cung cấp có V001–V006; đối chiếu file/checksum đầy đủ. Dự kiến V007 nếu không có bản mới hơn. |
| Database | Kiểm tra database/schema đang kết nối, pg_catalog, information_schema, pg_indexes; đếm users/profile/tin/đơn/ca; kiểm tra file private đang được tham chiếu. |
| Sao lưu | Backup/restore DB nguồn student_jobs_nhom5 sang student_jobs_nhom5_v4; sao lưu storage riêng; kiểm số dòng/FK/checksum, giữ nguồn nguyên vẹn. Hai ZIP code không thay thế pg_dump. Chưa có bằng chứng restore. |

### 8.2. Thứ tự migration đề xuất

| Thứ tự | Mô tả migration mới (VNNN chỉ là placeholder) | Điều kiện trước khi chạy |
| --- | --- | --- |
| A — nền v3 | Bù phần lịch ca, notifications, selected_slots, trạng thái hủy còn thiếu nếu có | Đối chiếu schema thực; không tạo trùng hoặc sửa checksum cũ. |
| B — D07C | VNNN__create_administrative_catalogs.sql; seed phiên bản mã địa bàn | Kiểm chứng nguồn, PK, FK, mã và lịch sử địa bàn. |
| C — D02 / D15 | VNNN__extend_users_account_status.sql | account_status, metadata khóa/mở, token_version; đồng bộ JWT, kiểm tra legacy. |
| D — nền D03 / D15C | VNNN__extend_job_categories.sql | Thêm is_active trước đăng ký NTD/API-29; giữ category_id, kiểm trùng legacy trước ràng buộc. D15C hoàn thiện API quản trị, không tạo migration trùng. |
| E — D03 | VNNN__extend_employer_profile.sql | Hồ sơ thường, FK ngành dùng chung, địa bàn, logo tùy chọn; không cột xác minh. |
| F — D05 / D15B | VNNN__extend_jobs_moderation_location.sql | PENDING legacy chưa duyệt, FK địa bàn, predicate và quyền đồng bộ. |
| G — hoàn thiện | VNNN__validate_v4_constraints.sql (nếu cần) | Kiểm tra backfill và constraints; chỉ SET NOT NULL khi dữ liệu đã đạt yêu cầu. |

### 8.3. Quy tắc chuyển tiếp

• Thêm cột mới theo giai đoạn: cho NULL đối với cột mới chưa có dữ liệu, seed danh mục trước FK, backfill có đối soát và kiểm tra constraint. Không sửa V001–V006 hoặc bất kỳ migration đã chạy. Chọn số từ thực tế file và schema_migrations.

• Không bịa tax_code, phone, actor Admin hoặc mã địa bàn. Tin legacy chưa có bằng chứng duyệt đặt moderation_status=PENDING, metadata NULL; có thể làm ẩn tin cũ. Không tự APPROVED. Nếu staging còn NULL, không công khai. Không thay trạng thái vòng đời/đơn do backfill.

• users cũ ACTIVE nếu không có bằng chứng khóa; kiểm tra dữ liệu trước backfill, token_version=0; token thiếu claim phải login lại. Danh mục cũ TRUE nếu chưa có trạng thái; giữ ID. Hồ sơ/tin legacy có thể NULL địa bàn/trường mới chưa có dữ liệu; kiểm tra bắt buộc với tạo mới, không ép NOT NULL cứng.

• Giữ city, location_detail, PK/FK, salary_min/max/unit, metadata CV, trạng thái vòng đời và lịch sử chọn ca. Tin legacy schedule_type=NULL vẫn không Matching và không suy lịch từ work_time; chỉ được đọc/nộp CV không ca khi đạt điều kiện duyệt/công khai và quy tắc legacy của SRS. Không đổi trạng thái đơn vì backfill hoặc kiểm duyệt.

• Không triển khai công khai đồng thời giữa schema mới và code cũ không hiểu moderation/account_status. Dùng môi trường thử nghiệm hoặc cửa sổ bảo trì; chỉ mở tính năng sau khi API-01/02/05/06/07/08/09/10/11/13/23/26 và quyền các API bảo vệ đồng bộ. Kế hoạch rollback app/DB phải tránh bỏ chặn kiểm duyệt; không dùng DROP CASCADE hoặc reset DB thật.

• Chạy trên DB sạch và DB sao chép có lịch sử, kiểm tra checksum, constraint, số dòng, migration chạy lại, quyền truy cập file, tranh chấp và worker. Không đánh dấu hoàn thành chỉ vì tạo file SQL.

## 9. Dữ liệu mẫu và bộ ca kiểm thử DB

| Mã | Dữ liệu / tình huống | Kết quả kỳ vọng |
| --- | --- | --- |
| DB01 | Đăng ký STUDENT / EMPLOYER; trùng email / sai role | Đúng một profile theo role; rollback khi một phần lỗi. |
| DB02 | Tin WEEKLY không end_date; qua nhiều tuần | Không tự FILLED/EXPIRED; PENDING có thể chờ. |
| DB03 | WEEKLY có start/end và ca CN 22:00→T2 02:00 | Tự kết thúc sau 02:00 T2, không phải 22:00 CN. |
| DB04 | ONE_TIME ba ca ngày 15,16,17; ca 15 đã bắt đầu | API-13 chỉ nhận ca 16/17; job vẫn RECRUITING. |
| DB05 | SV nộp ca 15+17, NTD duyệt ngày 16 | APPROVED, decided_at; ca 15 vẫn lưu nhưng không có hiệu lực. |
| DB06 | SV chỉ chọn ca 15; ca 15 bắt đầu mà còn PENDING | Đơn JOB_EXPIRED + notification, job vẫn tuyển ca khác. |
| DB07 | Tin qua end_at ca cuối; 1 APPROVED hoặc 0 APPROVED | FILLED hoặc EXPIRED; không tự COMPLETED. |
| DB08 | NTD có 2 APPROVED, 1 PENDING bấm Đóng tuyển | Chặn; không tự chuyển PENDING sang REJECTED. |
| DB09 | NTD hủy ngày trước ca đầu với 3 đơn PENDING/APPROVED/REJECTED | Tất cả JOB_CANCELLED; status_before_cancel đúng; decided_at giữ; 3 notification kèm lý do. |
| DB10 | NTD hủy ngày ca đầu, dù giờ làm là 18:00 | Bị chặn từ 00:00 local DATE; không UPDATE. |
| DB11 | Lỗi INSERT notifications trong giao dịch hủy | Rollback job, tất cả đơn và notifications. |
| DB12 | Hai worker chạy expiration, duyệt đơn đồng thời | Một kết quả cuối hợp lệ; không nhân đôi notification. |
| DB13 | SV rảnh CN 21:00→T2 03:00, job CN 22:00→T2 02:00 | Match 1/1=100%; ca đêm 1 record; chống chồng chu kỳ. |
| DB14 | Student chọn slot thuộc tin khác trong application_selected_slots | Service từ chối và rollback toàn bộ; FK đơn lẻ không đủ kiểm chứng. |
| DB15 | Tin đã có đơn REJECTED, NTD sửa lịch hoặc lương/nội dung | SCHEDULE_LOCKED nếu gửi trường lịch, MODERATION_EDIT_NOT_ALLOWED nếu chỉ nội dung; job/ca không đổi. |
| DB16 | Tin legacy work_time TEXT không ca | Chỉ GET công khai nếu đã duyệt và đủ điều kiện; không Matching, hạn hủy không xác định. |
| DB17 | Tin COMPLETED + đơn APPROVED / CANCELLED+JOB_CANCELLED | Chỉ cặp đầu được đánh giá; không có số liệu thu nhập. |
| DB18 | Đăng ký NTD đủ/thiếu trường, phường sai tỉnh, logo lỗi | User ACTIVE + profile thường; không yêu cầu minh chứng/thuế; lỗi rollback cả DB và file mới. |
| DB19 | Lịch sử v4.0: xác minh NTD | WON’T v4.1 — không tạo test nghiệm thu cho US46. |
| DB20 | NTD ACTIVE tạo tin; NTD LOCKED thử tạo | ACTIVE tạo RECRUITING/PENDING; LOCKED bị chặn. |
| DB21 | Tin PENDING/REJECTED vs APPROVED | Chỉ tin đủ điều kiện vào Homepage, API-09, API-26 và API-13; chi tiết/quyền lịch sử đúng. |
| DB22 | Admin khóa SV/NTD có JWT còn hạn; thử mở khóa | JWT trước khóa bị chặn cả sau mở khóa; token_version tăng khi khóa; login mới; giữ lịch sử. |
| DB23 | Danh mục ngừng sử dụng đang có tin | FK lịch sử còn; không chọn mới, không xóa cứng; race với đăng tin không tạo bản ghi trái điều kiện. |
| DB24 | Chọn toàn tỉnh A và hai phường tỉnh B | OR nhóm địa bàn, AND lọc khác; FK ghép không cho phường sai tỉnh; tin legacy NULL không khớp sai. |
| DB25 | Người ngoài truy cập CV hoặc đường dẫn storage; logo sai loại/size | CV bị chặn, không lộ key; logo đúng quyền và kiểm chữ ký, size ≤2 MiB. |
| DB26 | Sửa tin đã duyệt chưa/có đơn, khóa NTD có đơn, duyệt SV LOCKED | Chưa có đơn về PENDING; có đơn chặn sửa; chặn thao tác NTD LOCKED/duyệt mới SV LOCKED; giữ lịch sử, worker vẫn chạy. |
| DB27 | Dữ liệu cũ thiếu mã/file, có đơn và review | Migration giữ ID, số dòng và FK; không tự gán APPROVED; không xóa file/ca. |
| DB28 | Dashboard có fixture đã biết | Năm số đếm khớp DB, không lộ PII; SHOULD, chưa nghiệm thu. |
| DB29 | Admin khóa/duyệt cùng lúc NTD tạo tin hoặc SV ứng tuyển | Khóa đúng thứ tự; kiểm tra lại quyền sau khóa; không ghi dữ liệu trái trạng thái. |
| DB30 | Cố INSERT ca WEEKLY day_of_week=NULL hoặc lịch sử hủy thiếu status_before_cancel | CHECK từ chối NULL không hợp lệ; giao dịch rollback. |

Các hàng trên là KỊCH BẢN CẦN KIỂM THỬ; chưa có kết quả PASS/FAIL. Khi triển khai, chạy node:test và truy vấn PostgreSQL thật, ghi log và commit đã được xác minh.

## 10. Truy vết SRS v2.1 / phạm vi v4.1 ↔ API ↔ dữ liệu

| US / FR / AC | API v4.1 | Bảng / dữ liệu trung tâm |
| --- | --- | --- |
| US01–05 / FR1 / AC1.* | API-01/02/27/28/29/42 (logo tùy chọn) | users, profiles, địa bàn, danh mục; token_version; quay lại tin ở FE. |
| US06–07 / FR2,9,21 / AC2.*,9.*,21.* | API-07/08/11/12 | job_categories, job_posts, job_time_slots, địa bàn, moderation_status. |
| US08–11,17 / FR3,6 / AC3.*,6.* | API-09/10/14 | job_posts, ca, ngành, địa bàn, kiểm duyệt; lịch sử đơn. |
| US12 / FR4 / AC4.* | API-13/17 | applications, application_selected_slots, job_time_slots, chủ tin và kiểm duyệt. |
| US13–16 / FR5,6 / AC5.*,6.* | API-14/15/16/17/18 | applications, selected slots, metadata CV, notifications. |
| US18–19 / FR6 / AC6.* | API-19 + worker | job_posts.status, applications.status, completed_at. |
| US20–21 / FR7 / AC7.* | API-20/21 | reviews, applications, job_posts. |
| US22 / FR8 / AC8.* | API-22 | notifications, applications. |
| US35–38 / FR15–18 / AC15–18.* | API-03–06 | users, student_profiles, employer_profiles, địa bàn, lĩnh vực. |
| US39 / FR19 / AC19.* | API-23 | users + profiles, account_status. |
| US40–41 / FR26 / AC26.* | API-37 | users.account_status, token_version, metadata khóa/mở. |
| US42 / FR20 / AC20.* | API-24/25 | student_availability_slots. |
| US43 / FR21 / AC21.* | API-08/10/11 | job_posts, job_time_slots. |
| US44 / FR22 / AC22.* | API-26 | job_posts, job_time_slots, student_availability_slots; kiểm duyệt. |
| US45 / FR23 / AC23.* | API-19/22 | cancelled_at, cancel_reason, status_before_cancel, notifications. |
| US46 / FR24 / AC24.* | API-30–33 (ngoài MVP) | WON’T — giữ mã lịch sử, không thêm bảng/cột xác minh. |
| US47 / FR25 / AC25.* | API-08/09/10/11/13/26/34–36 | job_posts, job_time_slots, users/profile. |
| US48 / FR27 / AC27.* | API-07/08/11/29/38–40 | job_categories.is_active; FK industry_id. |
| US49 / FR28 / AC28.* (SHOULD) | API-41 | COUNT users, job_posts, applications. |

US23–US34 và US46 là 13 WON’T; không thêm bảng cho nhóm này. API-01–26 giữ mã nền, 27–29 và 34–42 còn phạm vi. Truy vết chi tiết theo SRS/Backlog v4.1.

### 10.1. Năm chỉ số Dashboard (FR28, SHOULD)

| Chỉ số | Điều kiện đếm |
| --- | --- |
| Tổng SV | COUNT users WHERE role='STUDENT'; gồm ACTIVE và LOCKED. |
| Tổng NTD | COUNT users WHERE role='EMPLOYER'; gồm ACTIVE và LOCKED. |
| Tin chờ duyệt | COUNT job_posts WHERE moderation_status='PENDING'. |
| Tin đang tuyển | COUNT tin RECRUITING, APPROVED, chủ NTD ACTIVE và đủ điều kiện mặc định API-09. |
| Tổng đơn | COUNT applications, mọi trạng thái. |

Dùng COUNT trên bảng gốc không nhân dòng vì join ca/đơn; không tạo bảng cache. pending_jobs đếm mọi vòng đời có moderation_status=PENDING. Không thống kê NTD riêng theo US29 (WON’T).

## 11. Quy trình cập nhật database chung và checklist bàn giao

• Tài trực tiếp lập trình từng Dxx. D00 kiểm kê; D07C địa bàn trước D03/D05/D07; D15 quyền, D15C danh mục trước D05; D05 → D07A → D15B → D06; D15B trước công khai/Matching/ứng tuyển; D15D sau dữ liệu nghiệp vụ. D15A ngoài MVP; 24 đợt theo Backlog v4.1.

• Đọc SRS/Backlog/Schema/API/AGENTS; xác định file, cột, migration, API, SHA nền và phụ thuộc. Kiểm chứng mục 12.6 trước code; không hỏi lại BR đã duyệt, báo nếu cần đổi ngoài phạm vi.

• Viết migration mới đúng VNNN__mo_ta.sql trong backend/migrations theo engine đang dùng. Test DB sạch, DB legacy, rollback, constraint, quyền API, đồng thời và ca qua đêm; ghi log test thật.

• Sau khi test đạt, Codex báo hoàn thành Dxx, chốt commit tiếng Việt khi được phép, tạo FullBackup và ChangedFiles (manifest mới/sửa/xóa, SHA nền, SHA đích). Không chứa .env, JWT, CV/file cá nhân thật, node_modules, .git hay DB dump thật. Tài gửi Phương/Khang để đặt file, kiểm thử, commit/push nhánh; Tài soát PR, merge sau xác minh.

• Mẫu tên: CDTN2_Nhom5_D07C_DanhMucDiaBan_FullBackup_SHA_Ngay.zip và CDTN2_Nhom5_D07C_DanhMucDiaBan_ChangedFiles_SHA_Ngay.zip. Ghi rõ chức năng, mã đợt, commit và thời điểm. Chỉ báo PASS, commit/push/merge khi có bằng chứng terminal/GitHub. Hai ZIP là sao lưu code, không phải backup database.

| Checklist | Tiêu chí trước bàn giao |
| --- | --- |
| Phạm vi | 49 US: 35 MUST, 1 SHOULD, 13 WON’T; FR24 ngoài MVP; 13 bảng nghiệp vụ, 1 kỹ thuật. |
| Dữ liệu | 11 bảng nền giữ nguyên; trạng thái tài khoản, kiểm duyệt, vòng đời và đơn độc lập; địa bàn, token_version, is_active đồng bộ. |
| Tính nhất quán | Không mất CV/ca/đơn; không tự APPROVED; dữ liệu thật không reset; file riêng tư. |
| Kiểm thử | DB01–DB30 là kế hoạch; DB19 ngoài MVP; chưa có PASS; nghiệm thu theo 113 AC MUST và 2 SHOULD riêng. |
| Tài liệu | API v4.1, ERD, Sprint Plan, AGENTS phải đồng bộ trước code liên quan; không đổi tên tùy ý. |

## 12. SQL tham chiếu cho migration mới (KHÔNG chạy trực tiếp)

Các đoạn sau minh họa ràng buộc theo PostgreSQL. Khi triển khai phải phân chia theo migration thực tế, thay/loại bỏ constraint cũ và đối chiếu tình trạng DB. Không chạy nguyên đoạn vào hệ thống đang có dữ liệu.

### 12.1. CHECK hình dạng dữ liệu cho job_time_slots

```sql
-- Trong CREATE TABLE job_time_slots mới, lựa chọn một trong 2 shape:
CONSTRAINT chk_job_time_slot_shape CHECK (
  (day_of_week IS NOT NULL AND day_of_week BETWEEN 1 AND 7 AND start_time IS NOT NULL
   AND end_time IS NOT NULL AND start_time <> end_time
   AND start_at IS NULL AND end_at IS NULL)
  OR
  (day_of_week IS NULL AND start_time IS NULL AND end_time IS NULL
   AND start_at IS NOT NULL AND end_at IS NOT NULL AND end_at > start_at)
)
```

Giải thích: Không có ca nào được đồng thời chứa dữ liệu WEEKLY và ONE_TIME. Kiểm tra IS NOT NULL tường minh vì PostgreSQL CHECK chấp nhận UNKNOWN. Kiểm tra job_posts.schedule_type phải thực hiện ở Service hoặc trigger có chủ đích; CHECK không join sang bảng cha.

### 12.2. CHECK ngày hiệu lực và 5 trạng thái job_posts

```sql
-- Trong migration mới, ADD schedule_type / schedule_* / cancelled_* trước.
CONSTRAINT chk_job_schedule_dates_v3 CHECK (
  (schedule_type IS NULL
   AND schedule_start_date IS NULL AND schedule_end_date IS NULL)
  OR (schedule_type = 'ONE_TIME'
      AND schedule_start_date IS NULL AND schedule_end_date IS NULL)
  OR (schedule_type = 'WEEKLY' AND schedule_start_date IS NOT NULL
      AND (schedule_end_date IS NULL OR schedule_end_date >= schedule_start_date))
)
-- CHECK trạng thái kết thúc (sau khi DROP đúng CHECK cũ):
CONSTRAINT chk_job_lifecycle_v3 CHECK (
  (status = 'COMPLETED' AND completed_at IS NOT NULL
   AND cancelled_at IS NULL AND cancel_reason IS NULL)
  OR (status = 'CANCELLED' AND cancelled_at IS NOT NULL
      AND completed_at IS NULL)
  OR (status IN ('RECRUITING','FILLED','EXPIRED')
      AND completed_at IS NULL AND cancelled_at IS NULL
      AND cancel_reason IS NULL)
)
```

Dịch vụ phải áp dụng CHECK status thuộc đúng 5 giá trị và completed_at/cancelled_at ≥ created_at; không dựa một mình vào ví dụ trên để xử lý toàn bộ hành động.

### 12.3. CHECK lịch sử quyết định khi JOB_CANCELLED

```sql
CONSTRAINT chk_application_decision_v3 CHECK (
  (status IN ('PENDING','JOB_EXPIRED')
   AND decided_at IS NULL AND status_before_cancel IS NULL)
  OR (status IN ('APPROVED','REJECTED')
      AND decided_at IS NOT NULL AND decided_at >= applied_at
      AND status_before_cancel IS NULL)
  OR (status = 'JOB_CANCELLED' AND status_before_cancel IS NOT NULL AND status_before_cancel IN
      ('PENDING','APPROVED','REJECTED','JOB_EXPIRED')
      AND (
        (status_before_cancel IN ('PENDING','JOB_EXPIRED') AND decided_at IS NULL)
        OR (status_before_cancel IN ('APPROVED','REJECTED')
            AND decided_at IS NOT NULL AND decided_at >= applied_at)
      ))
)
```

Giải thích: Khi hủy, current status đổi thành JOB_CANCELLED nhưng decided_at giữ nguyên theo kết quả cũ, không dùng cancelled_at làm thời điểm NTD đã duyệt.

### 12.4. Ranh giới ngày hủy ở Việt Nam

```sql
-- Minh họa so sánh khi đã xác định được first_shift_start_at:
(CURRENT_TIMESTAMP AT TIME ZONE 'Asia/Ho_Chi_Minh')::date
  < (first_shift_start_at AT TIME ZONE 'Asia/Ho_Chi_Minh')::date
```

Giải thích: So sánh DATE tại Việt Nam, không phải chỉ so sánh timestamptz. first_shift_start_at của WEEKLY phải được suy ra từ lần xuất hiện ca sớm nhất thuộc khoảng hiệu lực; không lấy schedule_start_date làm mốc ca đầu nếu ngày đó không có ca.

### 12.5. CHECK/FK tham chiếu cho v4.1

Các mảnh DDL chỉ tham chiếu; không chạy trực tiếp. Kiểu mã VARCHAR(16), FK và default theo thiết kế; kiểm tra schema/legacy trước VALIDATE CONSTRAINT.

```sql
-- Địa bàn: trong administrative_wards
UNIQUE (province_code, ward_code)
-- Trong employer_profiles và job_posts, mỗi bảng một FK tương ứng
FOREIGN KEY (province_code) REFERENCES administrative_provinces(province_code)
FOREIGN KEY (province_code, ward_code)
  REFERENCES administrative_wards(province_code, ward_code)
CHECK (ward_code IS NULL OR province_code IS NOT NULL)
-- Trong users (sau khi phân loại dữ liệu cũ)
CHECK (account_status IN ('ACTIVE','LOCKED'))
CHECK (account_status <> 'LOCKED' OR
  (locked_at IS NOT NULL AND locked_by IS NOT NULL
   AND lock_reason IS NOT NULL AND btrim(lock_reason) <> ''))
-- JWT: thêm cột token_version INTEGER NOT NULL DEFAULT 0
CHECK (token_version >= 0)
-- Hồ sơ NTD: dùng chung danh mục
FOREIGN KEY (industry_id) REFERENCES job_categories(category_id)
-- Trong job_posts: metadata kiểm duyệt độc lập, không có trạng thái duyệt hồ sơ.
CHECK (moderation_status IN ('PENDING','APPROVED','REJECTED'))
CHECK (
  (moderation_status = 'PENDING' AND moderated_by IS NULL
   AND moderated_at IS NULL AND moderation_reason IS NULL)
  OR (moderation_status = 'APPROVED' AND moderated_by IS NOT NULL
      AND moderated_at IS NOT NULL AND moderation_reason IS NULL)
  OR (moderation_status = 'REJECTED' AND moderated_by IS NOT NULL
      AND moderated_at IS NOT NULL AND moderation_reason IS NOT NULL
      AND btrim(moderation_reason) <> '')
)
```

Các cột trạng thái mục tiêu cần NOT NULL sau chuyển tiếp; CHECK IN không loại NULL. Actor role, token, file, quyền, ngày ca và điều kiện liên bảng kiểm tại Service/transaction. Không sửa migration đã áp dụng.

### 12.6. Kiểm chứng kỹ thuật trước code — BR đã chốt

| Mã / nguồn | Kiểm chứng còn phải thực hiện | Ảnh hưởng |
| --- | --- | --- |
| BR43/50 | Xác minh nguồn chính thức, phiên bản/ngày hiệu lực, mã duy nhất, seed và legacy; test encoding locations | D07C; API-27/28 và các API liên quan. |
| BR45/51 | BR45 ngoài MVP; kiểm thử hồ sơ thường, CV và logo, storage riêng | D03/D04/D08/D09. |
| BR48 | Kiểm tra danh mục dùng chung, FK industry_id, trùng tên/active | D03/D15C; API-29/38–40. |
| BR46 | Kiểm thử sửa tin chưa/có đơn, metadata reset và race | D06/D15B. |
| BR47 | Kiểm thử token_version, khóa/mở, quyền tin/đơn và worker | D02/D15. |
| BR50 | Kiểm kê schema/checksum, backup/restore, legacy, DTO và mã lỗi | D00 và migration. |
| FR25–28 | Đối chiếu hợp đồng API v4.1/ERD và test thật; chỉ xin duyệt nếu phát sinh thay đổi ngoài quyết định | D15B/C/D và D17. |

### Lịch sử phiên bản

v3.0 — 13/11/2026: 11 bảng, lịch WEEKLY/ONE_TIME, Matching, hủy tin và các trạng thái E2E.

v4.0 — 24/11/2026: thiết kế mở rộng Admin/địa bàn, từng đề xuất xác minh NTD.

v4.1 — 24/11/2026: bỏ xác minh NTD, chốt BR còn lại, DB đích riêng, 13 bảng, token_version, 38 API và 24 đợt; chưa sửa code, chạy migration hay kiểm thử runtime.
