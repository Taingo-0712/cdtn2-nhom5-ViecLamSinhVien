HỢP ĐỒNG API THỐNG NHẤT • FULL E2E • ĐỊA BÀN • QUẢN TRỊ

# API CONTRACT — NHÓM 5

Phiên bản 4.0 | 24/11/2026 | SRS v2.0 — Phạm vi v4.0

TRẠNG THÁI: THIẾT KẾ MỤC TIÊU TÍCH HỢP — CHỜ PHÊ DUYỆT HỢP ĐỒNG MỚI. Không phải bằng chứng API chạy hoặc migration đã áp dụng. Giữ 26 mã/URL/method nền; các bổ sung v4 (kể cả mã API-27–42, DTO, lỗi, encoding) là đề xuất cần xác nhận. Cổng BR45–48/50–51 tại mục 9.

| Chỉ tiêu | Giá trị |
| --- | --- |
| Phạm vi | 49 US: 36 MUST, 1 SHOULD (US49), 12 WON’T (US23–US34); FR1–28, 119 AC. |
| API | 26 endpoint nền + 16 endpoint đề xuất (API-27–42); tổng mục tiêu 42 nếu duyệt toàn bộ; API-41 Dashboard SHOULD, API-42 hỗ trợ logo tùy chọn. |
| Nguồn | SRS_Nhom5_v2.0_PhamVi_v4.0.docx; Product_Backlog_Nhom5_SE_v4.0_US01-US49.xlsx; Database_Schema_va_quy_trinh_cap_nhat_Nhom5_v4.0.docx / database-schema-v4.0.md; API Contract v3.0. |
| Công nghệ | React + Vite; Node.js + Express; PostgreSQL 16; JWT; PDF CV. |

### Mục lục

| Mục | Nội dung |
| --- | --- |
| 1–3 | Phạm vi, giao tiếp chung, từ điển DTO |
| 4.1–4.26 | Đặc tả 26 API nền đã cập nhật |
| 4.27–4.42 | Địa bàn, xác minh, quản trị và logo (đề xuất) |
| 5–8 | Trạng thái, transaction, truy vết US, test và Schema |
| 9–10 | Cổng phê duyệt, tương thích và checklist bàn giao |

## 1. Căn cứ, phạm vi và quy tắc thay đổi

### 1.1. Căn cứ chính

Căn cứ SRS v2.0 (phạm vi v4.0), Product Backlog v4.0, Database Schema v4.0 và hợp đồng API v3.0. Tích hợp trực tiếp thay đổi vào các mục dưới, không dùng phụ lục ghi đè. US/FR/UC/AC lấy từ SRS; thiết kế kỹ thuật mới chưa được duyệt không phải yêu cầu đã triển khai.

Giữ URL, HTTP method và mã API-01–26. Mã API-27–42 được dành trong bản thiết kế này để review; chưa cấp chính thức. Các trường v4, mã lỗi mới và quy tắc tương thích phải được xác nhận cùng schema trước khi code; không tự đổi endpoint.

### 1.2. Ranh giới phạm vi

MUST: các chức năng v3.0 về đăng nhập/hồ sơ, đăng/sửa/tìm/lọc, lịch ca, lịch rảnh, Matching, ứng tuyển CV, duyệt muộn, hết hạn, đóng tuyển/hoàn thành/hủy, thông báo, đánh giá; mở rộng xác minh NTD, kiểm duyệt tin, khóa/mở khóa SV/NTD, quản lý danh mục. SHOULD: Dashboard Admin (US49). WON’T: US23–US34. Không xây chat, thanh toán, tính lương, chấm công, kho CV, rút đơn, hạn mức người hay tự COMPLETED.

### 1.3. Hợp đồng mới nhưng chưa có trong migration hiện tại

Thiết kế dữ liệu v4.0 giữ các cột lịch và hủy v3.0, mở rộng users, employer_profiles, job_posts, job_categories; thêm 2 bảng địa bàn và phương án business_industries có điều kiện. Không tự gán mã/duyệt legacy, không sửa migration đã chạy. Xác minh commit/migration thật trước khi thực hiện.

## 2. Quy ước giao tiếp chung

### 2.1. HTTP, xác thực, dữ liệu

| Mục | Quy ước |
| --- | --- |
| Prefix & JSON | URL local http://localhost:8080/api; UTF-8; snake_case; JSON POST/PATCH/PUT, ngoại lệ API-13 CV multipart; API-01/06 NTD có multipart đề xuất tại mục tương ứng. |
| Xác thực | Authorization: Bearer <access_token>; JWT 3600 giây, sub ID chuỗi, role, iat, exp. Mọi API bảo vệ đọc tài khoản hiện tại từ DB và chặn LOCKED kể cả token còn hạn; quyền sở hữu/role kiểm tra phía BE. |
| Vai trò | STUDENT, EMPLOYER, ADMIN; server lấy role từ tài khoản. |
| BIGINT | ID là chuỗi chữ số nguyên dương 1…9223372036854775807; không số 0 đầu. |
| Thời gian lưu/trả | Timestamp (created_at, decided_at, cancelled_at, start_at/end_at ONE_TIME) trả ISO 8601 UTC Z; đầu vào ONE_TIME bắt buộc có offset hoặc Z, không cho datetime không múi giờ. |
| Ngày lịch & giờ địa phương | schedule_start_date/schedule_end_date: YYYY-MM-DD; start_time/end_time: HH:mm 00:00..23:59. Asia/Ho_Chi_Minh (UTC+07) là chuẩn để so sánh ngày hủy, WEEKLY, ca ngày và Matching. |
| Tiền | salary_min/max chuỗi thập phân VND, NUMERIC(12,2); salary_unit HOUR\|DAY\|SHIFT\|MONTH. Chỉ mức lương quảng cáo, không tính/ghi thu nhập thực nhận. |
| Chuỗi | Trim trừ password; chuỗi bắt buộc không thể toàn khoảng trắng; field lạ trả 400 FIELD_NOT_ALLOWED. |
| PATCH | Trường vắng không đổi; null chỉ dùng cho trường cho phép xóa; PATCH rỗng trả 400 EMPTY_PATCH. |
| Phân trang | page mặc định 1, >=1; size mặc định 20, 1..100; meta {page,size,total_elements,total_pages}. Không có dữ liệu trả data:[]. |
| Quyền riêng tư | ID chủ tài nguyên suy từ JWT và DB; Admin chỉ quyền đã đặc tả. Không trả password_hash, CV key, verification_doc_key hoặc logo_storage_key. Không truy cập CV/lịch rảnh của SV khác. |
| Multipart CV | API-13: cv PDF, selected_slot_ids JSON text array nếu có ca; tối đa 5.242.880 byte/file. File riêng tư khác theo API-01/06 (đề xuất). |
| Mã địa bàn (đề xuất) | Chuỗi 1–16 ký tự theo danh mục nguồn xác nhận; giữ số 0 đầu, không ép BIGINT. Mã phải tồn tại và thuộc đúng tỉnh; chọn mới chỉ mã hoạt động. |
| Query | Chỉ nhận tham số liệt kê, mỗi tên tối đa một lần; URL-encode; GET không body. ID sai trả 400 INVALID_ID; query sai 400 INVALID_QUERY. |
| Upload NTD (đề xuất) | multipart/form-data: profile là JSON text, verification_doc là file, logo tùy chọn; không cho client cung cấp storage key. Loại file/kích thước/lưu trữ chờ BR45/51. |

### 2.2. Envelope thành công và lỗi

JSON thành công dùng {"data":...}; danh sách phân trang thêm meta. API-17, API-32, API-42 (hai API sau là đề xuất) trả bytes file thay JSON khi thành công. Lỗi luôn có error.code, error.message, error.details (mảng), trace_id; không trả stack/SQL/key.

```text
{
  "data": [
    {
      "job_id": "301",
      "status": "RECRUITING"
    }
  ],
  "meta": {
    "page": 1,
    "size": 20,
    "total_elements": 1,
    "total_pages": 1
  }
}
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Dữ liệu không hợp lệ.",
    "details": [
      {
        "field": "time_slots",
        "reason": "Ca trùng giờ hoặc không hợp lệ."
      }
    ]
  },
  "trace_id": "trace-demo-001"
}
```

### 2.3. Mã HTTP dùng chung

| HTTP | Ý nghĩa |
| --- | --- |
| 200 | GET/PATCH/PUT thành công. |
| 201 | Tạo tài khoản/tin/đơn/đánh giá/danh mục thành công. |
| 400 | VALIDATION_ERROR, INVALID_ID, INVALID_QUERY, FIELD_NOT_ALLOWED, EMPTY_PATCH, INVALID_MULTIPART, CV_REQUIRED, CV_EMPTY. |
| 401 | AUTH_REQUIRED, INVALID_TOKEN, INVALID_CREDENTIALS. |
| 403 | ROLE_FORBIDDEN, RESOURCE_FORBIDDEN, FIELD_NOT_EDITABLE, ROLE_NOT_ALLOWED; đề xuất ACCOUNT_LOCKED, EMPLOYER_NOT_VERIFIED. |
| 404 | JOB_NOT_FOUND, APPLICATION_NOT_FOUND; đề xuất USER_NOT_FOUND, EMPLOYER_NOT_FOUND, VERIFICATION_DOCUMENT_NOT_FOUND, LOGO_NOT_FOUND; CATEGORY_NOT_FOUND là 404 ở path API-40, giữ 400 khi category_id input API-08/11 sai. |
| 409 | APPLICATION_ALREADY_EXISTS, APPLICATION_ALREADY_DECIDED, JOB_NOT_RECRUITING, JOB_NOT_EDITABLE, INVALID_JOB_TRANSITION, PENDING_APPLICATIONS_EXIST, NO_APPROVED_APPLICATION, APPLICATION_EXPIRED, SHIFT_ALREADY_STARTED, SCHEDULE_LOCKED, JOB_CANCEL_WINDOW_CLOSED, JOB_CANCEL_WINDOW_UNDETERMINED, CANCEL_REASON_REQUIRED. |
| 413 | CV_TOO_LARGE (hơn 5.242.880 byte). |
| 415 | UNSUPPORTED_MEDIA_TYPE, CV_TYPE_NOT_ALLOWED. |
| 500 | INTERNAL_ERROR, CV_UNAVAILABLE, APPLICATION_SAVE_FAILED, PROFILE_INCONSISTENT; rollback khi transaction lỗi. |

Các mã lỗi v4 được đánh dấu đề xuất; 400 cho sai cấu trúc/giá trị; 409 cho xung đột trạng thái; 403 cho sai quyền hoặc tài khoản khóa. Không phân tích message để điều khiển FE. Với API công khai, tài nguyên chưa được phép công khai trả 404 JOB_NOT_FOUND (đề xuất) để không dò tin ẩn; API của chủ tin/Admin phân quyền riêng.

## 3. Từ điển dữ liệu request/response

### 3.1. Tài khoản, hồ sơ và DTO quản trị

User = user_id, email, role, created_at, account_status (mới). StudentProfile = user_id, email, full_name, phone, introduction, updated_at. Email tối đa 254, trim/lowercase, unique; password đăng ký 8–128 ký tự, không trim; login không rỗng tối đa 128. STUDENT full_name 1–150, phone tùy chọn 10–11 chữ số, introduction tối đa 5.000. Giữ các quy tắc v3.0 còn phù hợp.

| Trường EmployerInput (đề xuất v4) | Validation và nguồn |
| --- | --- |
| display_name, contact_name | Bắt buộc 1–200 và 1–150; giữ tên trường hiện có. |
| phone | Bắt buộc 10–11 chữ số. |
| business_type | COMPANY hoặc INDIVIDUAL_SHOP. |
| tax_code | Chuỗi tối đa 50, điều kiện bắt buộc COMPANY còn chờ BR45. |
| industry_id | BIGINT string; danh mục hoạt động, đích FK chờ BR48. |
| province_code, ward_code | Chuỗi tối đa 16; cả hai bắt buộc với hồ sơ mới; mã hoạt động, quan hệ đúng. |
| address_detail | Bắt buộc 1–255. |
| introduction, website_url | Tùy chọn/null; giới thiệu ≤5.000; URL http/https ≤500, không tự fetch từ BE. |
| verification_doc, logo | File đầu vào multipart, minh chứng bắt buộc khi đăng ký/gửi duyệt, logo tùy chọn. Server tạo key; chính sách file chờ BR45/51. |

EmployerProfile (chỉ chủ hồ sơ hoặc Admin): user_id, email, display_name, contact_name, phone, introduction, updated_at, business_type, tax_code, industry_id, province_code, ward_code, address_detail, website_url, verification_status, reject_reason, verified_at, verified_by, verification_document_url, logo_url. Hai URL là đường dẫn API dẫn xuất hoặc null (không phải cột DB), không trả storage key. Legacy có thể null; không tự duyệt hồ sơ thiếu dữ liệu.

AdminEmployerItem = user_id, display_name, email, account_status, verification_status, verified_at, created_at. AdminEmployerDetail = EmployerProfile + account_status, created_at. AdminUserDetailStatus = user_id, role, account_status, locked_at, locked_by, lock_reason, unlocked_at, unlocked_by; metadata lần gần nhất, không phải audit log toàn bộ.

### 3.2. JobInput, JobSummary, JobDetail và lịch ca

| Trường JobInput | Kiểu / phạm vi | Quy tắc |
| --- | --- | --- |
| title | string 1..200 | Bắt buộc. |
| description | string 1..20000 | Bắt buộc. |
| category_id | BIGINT string | Bắt buộc; phải tồn tại và is_active=true khi tạo hoặc đổi danh mục. |
| work_time | string 1..5000 | Bắt buộc; mô tả dễ đọc, không dùng để tính lịch. |
| salary_min, salary_max | money string | Bắt buộc; 0 <= min <= max. |
| salary_unit | HOUR\|DAY\|SHIFT\|MONTH | Bắt buộc. |
| city | string 1..100 (response/legacy) | Tin mới server lấy từ tên tỉnh chuẩn; không cho nhập tự do; đề xuất không gửi city, nếu client cũ gửi thì chỉ chấp nhận khi khớp tên chuẩn và có đủ mã. Không tự gán mã legacy. |
| location_detail | string 1..255 | Bắt buộc. |
| schedule_type | WEEKLY\|ONE_TIME | Bắt buộc với tin mới; tin legacy thiếu lịch trả null. |
| schedule_mode | ANY\|ALL | Mặc định ANY; chỉ quyết định Matching. |
| schedule_start_date | YYYY-MM-DD\|null | Bắt buộc nếu WEEKLY; null nếu ONE_TIME. |
| schedule_end_date | YYYY-MM-DD\|null | Tùy chọn nếu WEEKLY; phải >= start_date nếu có; null nếu ONE_TIME. |
| time_slots | 1..21 JobTimeSlotInput | Bắt buộc tin mới; các ca logic không chồng lấn; ONE_TIME/WEEKLY dùng shape tương ứng. |
| province_code | string 1..16 | Đề xuất bắt buộc tin mới; danh mục tỉnh đang hoạt động. |
| ward_code | string 1..16 | Đề xuất bắt buộc tin mới; phường thuộc tỉnh, đang hoạt động. |

Không nhận required_workers, employer_id, status, completed_at, cancelled_at, cancel_reason, moderation_status, moderation_reason, moderated_at/by khi tạo/sửa tin. Tin mới bắt buộc lịch cấu trúc; legacy không ca chỉ đọc/nộp theo ngoại lệ FR4 khi đạt điều kiện công khai, không Matching. Không parse work_time để tính lịch.

| DTO | Fields & validation |
| --- | --- |
| WeeklySlotInput | {day_of_week:1..7, start_time:"HH:mm", end_time:"HH:mm"}; giờ bằng nhau không hợp lệ. end_time < start_time là một ca qua đêm duy nhất. |
| OneTimeSlotInput | {start_at:"2026-11-15T08:00:00+07:00",end_at:"2026-11-15T12:00:00+07:00"}; có offset, end_at > start_at; một ca logic có thể qua ngày sau. |
| JobTimeSlot | WeeklySlotInput hoặc OneTimeSlotInput kèm job_slot_id: BIGINT string. Các trường không thuộc kiểu lịch hiện hành phải vắng. |
| JobSummary | job_id, employer:{user_id,display_name,logo_url}, category:{category_id,name}, title, work_time, salary_min/max, salary_unit, currency:"VND", city, location_detail, status, schedule_type, schedule_start_date, schedule_end_date, completed_at, cancelled_at, created_at, updated_at, province_code, ward_code, moderation_status. Các mã có thể null với legacy. |
| JobDetail | Toàn bộ JobSummary + description, schedule_mode, time_slots: JobTimeSlot[], cancel_reason (null hoặc chuỗi nếu hủy). OwnerJobDetail/AdminJobDetail thêm moderation_reason, moderated_at, moderated_by. OwnerJobSummary/AdminJobSummary thêm các trường kiểm duyệt đó; không lộ lý do nội bộ trên public. |

Giới hạn 21 ca tính theo ca LOGIC; không tách ca qua đêm. Kiểm tra overlap cả ca qua ngày và ranh giới Chủ nhật→Thứ Hai với WEEKLY, và theo timestamp thực với ONE_TIME; ca kề nhau có thể hợp lệ. Ca WEEKLY chỉ có hiệu lực vào ngày trong [schedule_start_date, schedule_end_date] nếu có ngày cuối; ca qua đêm có thể kết thúc vào ngày kế tiếp.

### 3.3. Lịch rảnh, ứng tuyển, đánh giá và thông báo

| DTO | Các trường response / ý nghĩa |
| --- | --- |
| StudentAvailabilitySlot | availability_slot_id, day_of_week, start_time, end_time; một khoảng WEEKLY có thể qua đêm. 0–21 ca logic. |
| Application | application_id, job_id, student_id, status, status_before_cancel (nullable), selected_slots: JobTimeSlot[], cv:{original_name,mime_type,size_bytes,download_url}, applied_at, decided_at. selected_slots là lựa chọn bất biến khi đã nộp; các ca đã qua không bị xóa. |
| ApplicationDetail | Toàn bộ Application, thêm student:{user_id,full_name}, job:{job_id,title,status,employer_id}. Có thể thêm selected_slot_effects cho ca ONE_TIME để UI thể hiện ca đã qua trước khi duyệt. |
| StudentApplicationItem | Application + job:{job_id,title,status,employer_id}; lọc theo trạng thái đơn, không dùng job.status thay thế. |
| EmployerApplicationItem | Application + student:{user_id,full_name}; NTD xem các ca được đăng ký. |
| Review | review_id, application_id, reviewer_id, reviewee_id, rating 1..5, comment nullable, created_at. |
| Notification | notification_id, application_id, recipient_id, message, created_at. Thông báo hủy/đơn hết hạn gắn đơn ứng tuyển; không thêm đọc/chưa đọc hoặc websocket. |
| AdminUserItem | user_id, display_name, email, role, created_at, account_status; không trả ADMIN trong list. |

selected_slots lấy dữ liệu qua liên kết application_selected_slots; job_slot_id tham chiếu chính xác ca của tin. Đã khóa sửa lịch tin khi có bất kỳ application, nên có thể giữ snapshot hiển thị từ job_time_slots mà không mất lịch sử. Khi hủy, status_before_cancel lưu trạng thái ngay trước hủy; decided_at không bị ghi đè.

Trạng thái tin: RECRUITING, FILLED, COMPLETED, EXPIRED, CANCELLED. Trạng thái đơn: PENDING, APPROVED, REJECTED, JOB_EXPIRED, JOB_CANCELLED. Đối với hủy, JOB_CANCELLED có thể ghi đè trạng thái đơn kết thúc khác, và status_before_cancel giữ dấu vết.

Ba trạng thái độc lập v4: users.account_status ACTIVE/LOCKED; employer_profiles.verification_status PENDING/APPROVED/REJECTED; job_posts.moderation_status PENDING/APPROVED/REJECTED. Admin quyết định không tự đổi job_posts.status hay applications.status.

Điều kiện công khai dùng chung (đề xuất kỹ thuật): tin moderation_status=APPROVED, chủ tin đủ điều kiện FR3/BR44; chính sách ẩn tin của NTD bị khóa hoặc mất xác minh đang chờ BR45/47. Danh sách mặc định/Matching/ứng tuyển thêm RECRUITING và còn ca/lần tương lai; ngoại lệ legacy không lịch theo FR4 chỉ danh sách/ứng tuyển, không Matching. Chi tiết đã duyệt và lịch sử không bị lọc theo RECRUITING; giữ quyền đọc riêng của SV sở hữu đơn và chủ tin. Cổng BR45/47 phải chốt trước khi triển khai predicate production.

## 4. Danh sách và đặc tả API-01–API-42

| Mã | Method / endpoint | Mục đích | Quyền | US |
| --- | --- | --- | --- | --- |
| API-01 | POST /api/auth/register | Đăng ký SV/NTD | Công khai | US01/US02 |
| API-02 | POST /api/auth/login | Đăng nhập 3 vai trò | Công khai | US03–US05 |
| API-03 | GET /api/student/profile | SV xem hồ sơ | STUDENT | US35 |
| API-04 | PATCH /api/student/profile | SV cập nhật hồ sơ | STUDENT | US36 |
| API-05 | GET /api/employer/profile | NTD xem hồ sơ | EMPLOYER | US37 |
| API-06 | PATCH /api/employer/profile | NTD cập nhật hồ sơ | EMPLOYER | US38 |
| API-07 | GET /api/job-categories | Danh mục nghề | Công khai | US06/US07/US11 |
| API-08 | POST /api/jobs | Đăng tin chờ kiểm duyệt | EMPLOYER ACTIVE + APPROVED | US06/US43 |
| API-09 | GET /api/jobs | Danh sách và lọc công khai | Công khai | US08/US10/US11/US17 |
| API-10 | GET /api/jobs/{job_id} | Chi tiết và ca tin | Công khai có điều kiện; chủ tin/SV có đơn theo quyền | US09/US17/US43 |
| API-11 | PATCH /api/jobs/{job_id} | Sửa tin có khóa lịch | EMPLOYER + chủ tin | US07/US43 |
| API-12 | GET /api/employer/jobs | Danh sách tin NTD | EMPLOYER | US07/US18/US19 |
| API-13 | POST /api/jobs/{job_id}/applications | Nộp CV và chọn ca | STUDENT | US12 |
| API-14 | GET /api/student/applications | SV xem kết quả đơn | STUDENT | US16/US17 |
| API-15 | GET /api/jobs/{job_id}/applications | NTD xem ứng viên và ca | EMPLOYER + chủ tin | US13 |
| API-16 | GET /api/applications/{application_id} | Chi tiết đơn và ca | SV của đơn / NTD chủ tin | US13/US16 |
| API-17 | GET /api/applications/{application_id}/cv | Tải/xem CV bảo vệ | SV của đơn / NTD chủ tin | US12/US13 |
| API-18 | PATCH /api/applications/{application_id}/status | Duyệt muộn / từ chối | EMPLOYER + chủ tin | US14/US15/US22 |
| API-19 | PATCH /api/jobs/{job_id}/status | Đóng / hoàn thành / hủy tin | EMPLOYER + chủ tin | US18/US19/US45 |
| API-20 | POST /api/applications/{application_id}/reviews | Tạo đánh giá hai chiều | SV của đơn / NTD chủ tin | US20/US21 |
| API-21 | GET /api/applications/{application_id}/reviews | Xem đánh giá | SV của đơn / NTD chủ tin | US20/US21 |
| API-22 | GET /api/notifications | SV xem thông báo | STUDENT | US22/US45 |
| API-23 | GET /api/admin/users | Admin xem/lọc tài khoản và account_status | ADMIN | US39 |
| API-24 | GET /api/student/availability | SV đọc lịch rảnh | STUDENT | US42 |
| API-25 | PUT /api/student/availability | SV thay lịch rảnh | STUDENT | US42 |
| API-26 | GET /api/student/job-recommendations | Gợi ý Matching | STUDENT | US44/US43 |
| API-27 (đề xuất) | GET /api/locations/provinces | Danh mục tỉnh/thành | Công khai | US02/06/07/11 |
| API-28 (đề xuất) | GET /api/locations/wards | Danh mục phường/xã theo tỉnh | Công khai | US02/06/07/11 |
| API-29 (đề xuất) | GET /api/business-industries | Danh mục lĩnh vực NTD (nguồn chờ BR48) | Công khai | US02/38 |
| API-30 (đề xuất) | GET /api/admin/employers | Danh sách xác minh NTD | ADMIN | US46 |
| API-31 (đề xuất) | GET /api/admin/employers/{user_id} | Chi tiết hồ sơ NTD | ADMIN | US46 |
| API-32 (đề xuất) | GET /api/employers/{user_id}/verification-document | Đọc minh chứng riêng tư | ADMIN hoặc chủ hồ sơ | US37/46 |
| API-33 (đề xuất) | PATCH /api/admin/employers/{user_id}/verification | Duyệt/từ chối NTD | ADMIN | US46 |
| API-34 (đề xuất) | GET /api/admin/jobs | Danh sách tin kiểm duyệt | ADMIN | US47 |
| API-35 (đề xuất) | GET /api/admin/jobs/{job_id} | Chi tiết tin kiểm duyệt | ADMIN | US47 |
| API-36 (đề xuất) | PATCH /api/admin/jobs/{job_id}/moderation | Duyệt/từ chối tin | ADMIN | US47 |
| API-37 (đề xuất) | PATCH /api/admin/users/{user_id}/status | Khóa/mở khóa tài khoản | ADMIN | US40/41 |
| API-38 (đề xuất) | GET /api/admin/job-categories | Xem danh mục mọi trạng thái | ADMIN | US48 |
| API-39 (đề xuất) | POST /api/admin/job-categories | Thêm danh mục | ADMIN | US48 |
| API-40 (đề xuất) | PATCH /api/admin/job-categories/{category_id} | Sửa/ngừng dùng danh mục | ADMIN | US48 |
| API-41 (đề xuất) | GET /api/admin/dashboard | Dashboard (SHOULD) | ADMIN | US49 |
| API-42 (đề xuất) | GET /api/employers/{user_id}/logo | Hiển thị logo (hỗ trợ tùy chọn) | Công khai có điều kiện / chủ / Admin | US02/37/46 |

API-01–26 giữ method/URL, tích hợp yêu cầu v4 vào nội dung bên dưới. API-27–42 là mã và đường dẫn đề xuất, chưa lập trình; API-41 SHOULD, API-42 hỗ trợ trường logo tùy chọn. Mọi request bảo vệ dùng quy tắc account_status tại 2.1. Tất cả ví dụ ID, ngày và mã địa bàn chỉ là fixture; cần seed nguồn xác nhận và thời gian tương lai khi test.

### 4.1. API-01 — Đăng ký và tự khởi tạo hồ sơ

201 data = {user: User, profile: StudentProfile hoặc EmployerProfile theo role}; không có token.

POST /api/auth/register  

Quyền: Công khai. Truy vết: US01, US02 ↔ FR1.

Request:

```text
{
  "email": "sv.demo@example.com",
  "password": "MatKhauMau123!",
  "role": "STUDENT",
  "full_name": "Sinh viên minh họa"
}
```

Ví dụ multipart NTD (profile là JSON text, mã P_TEST/W_TEST là fixture, không phải mã hành chính thật):

```text
{
  "email": "ntd.demo@example.com",
  "password": "MatKhauMau123!",
  "role": "EMPLOYER",
  "display_name": "Cửa hàng minh họa",
  "contact_name": "Người liên hệ",
  "phone": "0901234567",
  "business_type": "COMPANY",
  "tax_code": "TEST_TAX",
  "industry_id": "1",
  "province_code": "P_TEST",
  "ward_code": "W_TEST",
  "address_detail": "Địa chỉ minh họa"
}
```

Hai phần file: verification_doc=<file>; logo=<file tùy chọn>. Không tự đặt boundary trong browser.

STUDENT giữ JSON body email,password,role,full_name và phone/introduction tùy chọn; không nhận trường EMPLOYER. NTD v4 đề xuất multipart gồm profile (JSON text chứa email,password,role=EMPLOYER và EmployerInput mục 3.1), verification_doc (file bắt buộc), logo (file tùy chọn). Không nhận full_name cho NTD, storage key, trạng thái, actor. Kiểm tra toàn bộ trước ghi; users ACTIVE + đúng một hồ sơ PENDING, metadata quyết định null, lưu file riêng tư; rollback DB và dọn file mới nếu lỗi. Không tự đăng nhập; response 201 theo DTO. Cổng BR45/48/50/51 và nguồn địa bàn phải chốt trước code NTD.

Thành công: HTTP 201.

Lỗi riêng (cộng với lỗi chung tại mục 2.3):

| HTTP | error.code | Điều kiện |
| --- | --- | --- |
| 400 | VALIDATION_ERROR | Sai dữ liệu hoặc sai trường theo role. |
| 403 | ROLE_NOT_ALLOWED | Gửi role ADMIN; không tạo tài khoản. |
| 409 | EMAIL_ALREADY_EXISTS | Email chuẩn hóa đã tồn tại. |
| 400 | VERIFICATION_DOCUMENT_REQUIRED (đề xuất) | Thiếu minh chứng khi đăng ký/gửi duyệt. |
| 400 | INVALID_MULTIPART (đề xuất NTD) | Field/file lạ hoặc profile JSON sai. |
| 400 | INVALID_LOCATION / INDUSTRY_NOT_FOUND (đề xuất) | Mã không tồn tại, sai quan hệ hoặc không hoạt động. |
| 413/415 | VERIFICATION_DOCUMENT_TOO_LARGE / VERIFICATION_DOCUMENT_TYPE_NOT_ALLOWED (đề xuất) | Ngưỡng/loại file chờ BR45/51. |
| 500 | VERIFICATION_SAVE_FAILED (đề xuất) | Lỗi lưu hồ sơ/file; dọn file mới, rollback DB. |

### 4.2. API-02 — Đăng nhập ba vai trò

200 data = {access_token, token_type: Bearer, expires_in: 3600, user: User}.

API-02 hỗ trợ đủ STUDENT, EMPLOYER và ADMIN. Role lấy từ tài khoản phía server; tài khoản ADMIN được seed và sử dụng cùng API đăng nhập.

POST /api/auth/login  

Quyền: Công khai. Truy vết: US03, US04, US05 ↔ FR1.

Request:

```text
{
  "email": "sv.demo@example.com",
  "password": "MatKhauMau123!"
}
```

Validation và xử lý: Dùng cùng API cho STUDENT/EMPLOYER/ADMIN; request không chứa role. Server đối chiếu mật khẩu băm và lấy role từ users. Admin được seed sẵn. Token không làm thay đổi role; frontend chọn màn hình theo role trả về. Phản hồi chứa token dùng Cache-Control: no-store. Sau khi kiểm tra credentials hợp lệ, LOCKED trả 403 ACCOUNT_LOCKED (đề xuất), không cấp token; NTD ACTIVE chưa xác minh vẫn nhận token để xem/bổ sung hồ sơ. FE xác định route từ role; guest ứng tuyển lưu route nội bộ hợp lệ, quay lại đúng tin sau login/register SV, gọi lại API-10, không gửi đơn tự động. Không thêm return_url vào request/token; NTD/Admin về màn hình theo role.

Thành công: HTTP 200.

```text
{
  "data": {
    "access_token": "<JWT_ACCESS_TOKEN>",
    "token_type": "Bearer",
    "expires_in": 3600,
    "user": {
      "user_id": "101",
      "email": "sv.demo@example.com",
      "role": "STUDENT",
      "created_at": "2026-10-30T06:00:00Z",
      "account_status": "ACTIVE"
    }
  }
}
```

Lỗi riêng (cộng với lỗi chung tại mục 2.3):

| HTTP | error.code | Điều kiện |
| --- | --- | --- |
| 400 | VALIDATION_ERROR | Thiếu/sai trường đầu vào. |
| 401 | INVALID_CREDENTIALS | Email không có hoặc sai mật khẩu; cùng một thông báo, không chỉ ra trường nào đúng. |
| 403 | ACCOUNT_LOCKED (đề xuất) | Thông tin đăng nhập đúng nhưng tài khoản đang LOCKED. |

### 4.3. API-03 — SV xem hồ sơ của mình

200 data = StudentProfile.

GET /api/student/profile  

Quyền: STUDENT. Truy vết: US35 ↔ FR15.

Request:

Không có body. Không nhận query. Path ID (nếu có) theo mục 2.1.

Validation và xử lý: Không có path/query user_id; server dùng sub trong token. Không tạo hồ sơ mới khi GET. phone/introduction chưa có trả null. Không hỗ trợ endpoint xem hồ sơ cá nhân SV khác.

Thành công: HTTP 200.

Lỗi riêng (cộng với lỗi chung tại mục 2.3):

| HTTP | error.code | Điều kiện |
| --- | --- | --- |
| 400 | INVALID_QUERY | Gửi query user_id hoặc tham số không hỗ trợ. |
| 500 | PROFILE_INCONSISTENT | Tài khoản thiếu hồ sơ tương ứng do dữ liệu không nhất quán. |

### 4.4. API-04 — SV cập nhật hồ sơ của mình

200 data = StudentProfile đã cập nhật.

PATCH /api/student/profile  

Quyền: STUDENT. Truy vết: US36 ↔ FR16.

Request:

```text
{
  "full_name": "Sinh viên minh họa",
  "phone": "0901234567"
}
```

Validation và xử lý: Chỉ nhận full_name, phone, introduction; phải có ít nhất một trường. Theo PATCH mục 2.1; mọi trường được kiểm tra trước, một trường sai thì không lưu phần còn lại. Cập nhật updated_at; không thay đổi email/role hay CV trong các đơn đã gửi.

Thành công: HTTP 200.

Lỗi riêng (cộng với lỗi chung tại mục 2.3):

| HTTP | error.code | Điều kiện |
| --- | --- | --- |
| 400 | EMPTY_PATCH | Body rỗng. |
| 400 | VALIDATION_ERROR | Tên rỗng/quá dài hoặc điện thoại sai. |
| 403 | FIELD_NOT_EDITABLE | Gửi email hoặc role để sửa. |

### 4.5. API-05 — NTD xem hồ sơ của mình

200 data = EmployerProfile theo mục 3.1; không trả key.

GET /api/employer/profile  

Quyền: EMPLOYER. Truy vết: US37 ↔ FR17.

Request:

Không có body. Không nhận query. Path ID (nếu có) theo mục 2.1.

Validation và xử lý: Lấy user_id từ token; không nhận user_id từ query. Trường tùy chọn chưa có trả null. Không tạo thêm hồ sơ hoặc trả hồ sơ của NTD khác. Trả EmployerProfile mở rộng kể cả PENDING/REJECTED, lý do từ chối, URL minh chứng bảo vệ (API-32 đề xuất) và logo nếu có. Không lộ key.

Ví dụ DTO đầy đủ (mã địa bàn là fixture):

```text
{
  "data": {
    "user_id": "201",
    "email": "ntd.demo@example.com",
    "display_name": "Cửa hàng minh họa",
    "contact_name": "Người liên hệ",
    "phone": "0901234567",
    "introduction": null,
    "updated_at": "2026-11-24T04:00:00Z",
    "business_type": "COMPANY",
    "tax_code": "TEST_TAX",
    "industry_id": "1",
    "province_code": "P_TEST",
    "ward_code": "W_TEST",
    "address_detail": "Địa chỉ minh họa",
    "website_url": null,
    "verification_status": "PENDING",
    "reject_reason": null,
    "verified_at": null,
    "verified_by": null,
    "verification_document_url": "/api/employers/201/verification-document",
    "logo_url": null
  }
}
```

Thành công: HTTP 200.

Lỗi riêng (cộng với lỗi chung tại mục 2.3):

| HTTP | error.code | Điều kiện |
| --- | --- | --- |
| 400 | INVALID_QUERY | Gửi user_id hoặc query không hỗ trợ. |
| 500 | PROFILE_INCONSISTENT | Tài khoản thiếu hồ sơ tương ứng. |

### 4.6. API-06 — NTD cập nhật hồ sơ của mình

200 data = EmployerProfile đã cập nhật.

PATCH /api/employer/profile  

Quyền: EMPLOYER. Truy vết: US38 ↔ FR18.

Request:

```text
{
  "display_name": "Cửa hàng minh họa",
  "website_url": "https://example.com"
}
```

Ví dụ JSON sửa giới thiệu/website; nếu sửa trường xác minh phải áp dụng BR45 đã duyệt; khi thay file dùng multipart như mô tả bên dưới.

NTD ACTIVE sửa hồ sơ chính mình. JSON PATCH chỉ nhận EmployerInput không file; multipart đề xuất khi thay minh chứng/logo, profile là JSON text chứa các trường PATCH, verification_doc/logo là file tùy chọn. Không nhận email/role/user_id, trạng thái, actor hoặc raw key. Kiểm tra dữ liệu cuối, địa bàn và danh mục; file thay mới chỉ xóa file cũ theo chính sách sau commit; lỗi không mất dữ liệu. Đề xuất submit_for_verification boolean trong profile/JSON cho hồ sơ REJECTED hoặc legacy đã bổ sung đủ: true đưa về PENDING và xóa metadata quyết định hiện tại; chưa được duyệt BR45, không triển khai tự reset hoặc tự thay quyền xử lý đơn cũ. Quy tắc sửa hồ sơ APPROVED, thu hồi/giữ quyền và file retention phải chốt trước code.

Thành công: HTTP 200.

Lỗi riêng (cộng với lỗi chung tại mục 2.3):

| HTTP | error.code | Điều kiện |
| --- | --- | --- |
| 400 | EMPTY_PATCH | Không có trường cập nhật. |
| 400 | VALIDATION_ERROR | Tên bắt buộc rỗng hoặc điện thoại/độ dài sai. |
| 403 | FIELD_NOT_EDITABLE | Yêu cầu sửa email/role. |
| 400 | INVALID_LOCATION / INDUSTRY_NOT_FOUND (đề xuất) | Mã sai hoặc danh mục không hợp lệ. |
| 409 | VERIFICATION_TRANSITION_NOT_ALLOWED (đề xuất) | Gửi lại hoặc thay hồ sơ trái BR45 sau khi được chốt. |

### 4.7. API-07 — Đọc danh mục ngành nghề

200 data = Category[]: category_id, name, is_active (đề xuất); không meta.

GET /api/job-categories  

Quyền: Công khai. Truy vết: US06, US07, US11 (hỗ trợ) ↔ FR2, FR3, FR9.

Request:

Không body. Query đề xuất scope=selection|filter; mặc định selection. Chỉ query này được nhận.

Đề xuất scope=selection trả danh mục is_active=true (form tạo/đổi danh mục); scope=filter trả cả danh mục ngừng dùng để vẫn lọc tin còn hợp lệ theo ngành cũ. Sort name ASC, category_id ASC; [] nếu rỗng. API quản trị ghi riêng 38–40, không DELETE cứng. Tin đã dùng danh mục ngừng hoạt động vẫn giữ liên kết; API-11 không ép đổi danh mục nếu không đổi category_id.

Thành công: HTTP 200.

Lỗi riêng (cộng với lỗi chung tại mục 2.3):

| HTTP | error.code | Điều kiện |
| --- | --- | --- |
| 400 | INVALID_QUERY | Có tham số không hỗ trợ. |

### 4.8. API-08 — NTD đăng tin với lịch WEEKLY / ONE_TIME

POST /api/jobs | EMPLOYER ACTIVE + APPROVED | US06, US43 ↔ FR2/FR21/FR25

Kết quả: 201 data = OwnerJobDetail; Location: /api/jobs/{job_id}. Chủ tin gửi Bearer để xem chi tiết chưa duyệt qua API-10.

Request JSON: các trường nội dung mục 3.2 và schedule_type, schedule_mode, ngày hiệu lực khi WEEKLY, danh sách 1–21 ca logic. Ví dụ WEEKLY:

```text
{
  "title": "Nhân viên bán hàng part-time",
  "description": "Hỗ trợ bán hàng theo ca.",
  "category_id": "1",
  "work_time": "Thứ Bảy và Chủ nhật, 08:00–12:00",
  "salary_min": "25000.00",
  "salary_max": "30000.00",
  "salary_unit": "HOUR",
  "province_code": "P_TEST",
  "ward_code": "W_TEST",
  "location_detail": "Địa chỉ cửa hàng minh họa",
  "schedule_type": "WEEKLY",
  "schedule_mode": "ANY",
  "schedule_start_date": "2026-11-14",
  "schedule_end_date": "2026-12-14",
  "time_slots": [
    {
      "day_of_week": 6,
      "start_time": "08:00",
      "end_time": "12:00"
    },
    {
      "day_of_week": 7,
      "start_time": "08:00",
      "end_time": "12:00"
    }
  ]
}
```

Ví dụ ONE_TIME (ca nhiều ngày và ca qua đêm):

```text
{
  "title": "Hỗ trợ sự kiện",
  "description": "Hỗ trợ chuẩn bị sự kiện.",
  "category_id": "1",
  "work_time": "Các ngày 15 và 17/11",
  "salary_min": "200000.00",
  "salary_max": "250000.00",
  "salary_unit": "SHIFT",
  "province_code": "P_TEST",
  "ward_code": "W_TEST",
  "location_detail": "Địa điểm sự kiện minh họa",
  "schedule_type": "ONE_TIME",
  "schedule_mode": "ALL",
  "schedule_start_date": null,
  "schedule_end_date": null,
  "time_slots": [
    {
      "start_at": "2026-11-15T18:00:00+07:00",
      "end_at": "2026-11-15T22:00:00+07:00"
    },
    {
      "start_at": "2026-11-17T22:00:00+07:00",
      "end_at": "2026-11-18T02:00:00+07:00"
    }
  ]
}
```

Tin mới bắt buộc lịch ca cấu trúc; validation đối chiếu đúng shape theo schedule_type và không chồng lấn. Không nhận số lượng tuyển. employer_id lấy JWT; RECRUITING; completed_at/cancelled_at/cancel_reason null; lưu job+slots cùng transaction. Khi tuần có ngày kết thúc, phải có ít nhất một lần xuất hiện ca trong khoảng hiệu lực; ONE_TIME tối thiểu một ca tương lai tại thời điểm tạo. Tin mới moderation_status=PENDING, moderation_reason/moderated_at/moderated_by null; chưa công khai. Kiểm tra account_status, verification_status, category active và mã tỉnh/phường tại BE, lấy city từ tên tỉnh. 201 không đồng nghĩa đã duyệt; owner nhận OwnerJobDetail. Kiểm tra lại quyền trước commit.

Ví dụ response 201 (minh họa rút gọn; phản hồi thật phải đủ OwnerJobDetail mục 3.2):

```text
{
  "data": {
    "job_id": "301",
    "status": "RECRUITING",
    "moderation_status": "PENDING",
    "moderation_reason": null,
    "moderated_at": null,
    "moderated_by": null,
    "province_code": "P_TEST",
    "ward_code": "W_TEST",
    "city": "Tỉnh minh họa"
  }
}
```

| HTTP | error.code | Điều kiện |
| --- | --- | --- |
| 400 | VALIDATION_ERROR | Thiếu/sai lịch, ngày, ca quá đêm, overlap, 0 hoặc >21 ca, dữ liệu nội dung sai. |
| 400 | CATEGORY_NOT_FOUND | Ngành nghề không tồn tại. |
| 400 | FIELD_NOT_ALLOWED | Gửi required_workers, status, employer_id hoặc trường ngoài schema. |
| 403 | EMPLOYER_NOT_VERIFIED (đề xuất) | NTD chưa APPROVED. |
| 400 | INVALID_LOCATION / CATEGORY_INACTIVE (đề xuất) | Mã không hợp lệ hoặc ngành ngừng dùng. |

### 4.9. API-09 — Danh sách, tìm kiếm và lọc công khai

GET /api/jobs  |  Quyền: Công khai  |  Truy vết: US08/US10/US11/US17 ↔ FR3/FR6

Kết quả: 200 data = JobSummary[] + meta.

| Query | Kiểu | Quy tắc |
| --- | --- | --- |
| keyword | string <=200 | Trim; rỗng bỏ lọc; tìm title OR description không phân biệt hoa/thường. |
| category_id | BIGINT string | Lọc ngành. |
| city | string <=100 | Tương thích legacy: tên thành phố khớp toàn bộ, không phân biệt hoa/thường; không dùng cùng locations. |
| salary_min, salary_max | money | Khoảng lương giao nhau và cùng đơn vị. |
| salary_unit | HOUR/DAY/SHIFT/MONTH | Bắt buộc khi có salary_min/max. |
| status | 5 trạng thái tin | Mặc định RECRUITING, đủ điều kiện nhận đơn; giữ enum 5 trạng thái cho truy vấn tương thích rõ ràng, không hiển thị dropdown public. Khi chọn trạng thái khác chỉ tin đã duyệt đủ quyền công khai, không ngầm cho ứng tuyển (đề xuất cần duyệt). |
| schedule_type | WEEKLY\|ONE_TIME | Tùy chọn; tin cũ không có loại lịch không khớp bộ lọc này. |
| day_of_week | integer 1..7 | Lọc ca có ngày bắt đầu tương ứng; với ONE_TIME suy từ local start_at. |
| time_from, time_to | HH:mm | Phải gửi thành cặp và có day_of_week hoặc cặp ngày; chọn ca nằm trọn khoảng giờ truy vấn. |
| date_from, date_to | YYYY-MM-DD | Cặp ngày tùy chọn, from<=to; WEEKLY xét các lần ca phát sinh trong khoảng hiệu lực, ONE_TIME xét ngày địa phương ca. |
| page, size | integer | Quy tắc phân trang chung. |
| locations (đề xuất) | JSON text, URL-encoded | Mảng 1–34 nhóm {province_code,ward_codes}; ward_codes=[] chọn toàn tỉnh, danh sách không rỗng chọn đúng phường; không lặp tỉnh/phường. Không dùng cùng city. Giới hạn 200 mã phường tổng (đề xuất). |

Mặc định chỉ tin đủ điều kiện công khai + RECRUITING + còn ca ứng tuyển (ngoại lệ legacy FR4 nếu được phép công khai). Các nhóm lọc AND; trong keyword là OR title/description; địa bàn OR giữa các nhóm. Tỉnh A toàn bộ + tỉnh B chọn phường B1/B2 không được biến thành hai danh sách phẳng. Tỉnh toàn bộ có thể khớp legacy biết tỉnh đáng tin, phường cụ thể cần đủ mã; không đoán mã từ city. Lọc lịch cần ít nhất một ca phù hợp cả nhóm điều kiện; legacy không ca không khớp lọc lịch. Sort created_at DESC, job_id DESC. Lọc giờ qua đêm trên trục 48h. FE khi xóa keyword tự gọi lại API giữ các lọc khác, reset toàn bộ về mặc định và page=1, bỏ response cũ đến muộn. Không tự thay BR47 trước phê duyệt.

Encoding đề xuất: locations=[{"province_code":"P_A","ward_codes":[]},{"province_code":"P_B","ward_codes":["W_B1","W_B2"]}] (fixture). URL dùng URLSearchParams.set("locations", JSON.stringify(groups)); BE parse JSON đúng một lần; [] không gửi (hoặc coi [] là bỏ lọc theo đề xuất), mã sai/nhóm trùng/sai quan hệ/đồng thời city trả 400 INVALID_QUERY. Chọn toàn tỉnh không hạn chế phường tỉnh đó. Giữ giới hạn URL hợp lý, xác nhận giới hạn trước code.

Ví dụ: GET /api/jobs?keyword=ban&page=1&size=20; khi xóa keyword GET /api/jobs?page=1&size=20 (nếu không còn lọc). GET /api/jobs?locations=%5B%7B%22province_code%22%3A%22P_A%22%2C%22ward_codes%22%3A%5B%5D%7D%5D&page=1.

Ví dụ 200 data/meta (minh họa rút gọn; mỗi phần tử thật đủ JobSummary):

```text
{
  "data": [
    {
      "job_id": "301",
      "title": "Nhân viên bán hàng part-time",
      "status": "RECRUITING",
      "moderation_status": "APPROVED",
      "province_code": "P_TEST",
      "ward_code": "W_TEST"
    }
  ],
  "meta": {
    "page": 1,
    "size": 20,
    "total_elements": 1,
    "total_pages": 1
  }
}
```

| HTTP | error.code | Điều kiện |
| --- | --- | --- |
| 400 | INVALID_QUERY | Query sai format, lương thiếu đơn vị, ngày đầu > ngày cuối, time_from/to thiếu cặp, tham số lạ/lặp. |

### 4.10. API-10 — Xem chi tiết tin và lịch ca

GET /api/jobs/{job_id} | Công khai có điều kiện; Bearer tùy chọn cho chủ tin/SV có đơn | US09/US17/US43 ↔ FR3/FR6/FR21

Kết quả: 200 data = JobDetail (public/SV theo quyền) hoặc OwnerJobDetail (chủ tin theo JWT).

Trả đầy đủ schedule_type, schedule_mode, schedule_start_date, schedule_end_date, time_slots; ONE_TIME dùng start_at/end_at; WEEKLY dùng day_of_week/start_time/end_time. Tin cũ chưa có lịch: schedule_type:null, schedule_mode:"ANY", time_slots:[], schedule_start_date/end_date:null. Chỉ trả cancel_reason khi tin CANCELLED (nếu lưu; lý do không nên bị hiểu là lời nhắn riêng tư); không trả CV/đơn hoặc lịch rảnh của SV. Tin đã duyệt có thể xem 5 trạng thái theo quyền. Đề xuất: guest/người không liên quan nhận 404 nếu tin chưa được phép công khai; chủ tin ACTIVE được đọc OwnerJobDetail kể cả PENDING/REJECTED; SV sở hữu đơn được đọc dữ liệu lịch sử JobDetail, không lý do kiểm duyệt nội bộ; Admin dùng API-35. Token cung cấp nhưng sai/khóa trả lỗi, không hạ xuống guest. Chính sách tin của NTD bị khóa/thay xác minh chờ BR45/47; không chặn lịch sử hợp lệ.

Nhóm ca có thể được chọn ứng tuyển: FE lấy time_slots và giờ server; BE mới là nguồn quyết định cuối khi POST. Ca ONE_TIME đã bắt đầu thì vẫn hiện để tra cứu, nhưng phải thể hiện là không nhận thêm đơn cho ca đó.

| HTTP | error.code | Điều kiện |
| --- | --- | --- |
| 400 | INVALID_ID | ID sai định dạng/phạm vi. |
| 404 | JOB_NOT_FOUND | Không có tin. |

### 4.11. API-11 — NTD sửa tin và khóa lịch sau khi có ứng viên

PATCH /api/jobs/{job_id}  |  Quyền: EMPLOYER + chủ tin  |  Truy vết: US07/US43 ↔ FR9/FR21

Kết quả: 200 data = OwnerJobDetail đã cập nhật.

Input cho phép các trường nội dung của JobInput, cùng các trường lịch: schedule_type, schedule_mode, schedule_start_date, schedule_end_date, time_slots. Phải có ít nhất 1 trường. Kiểm tra giá trị cuối cùng sau hợp nhất; time_slots khi gửi là thay toàn bộ danh sách 1–21 ca, không được gửi [] để xóa lịch tin; giao dịch thay job+ca là nguyên tử.

Tin phải RECRUITING, đúng chủ. Khi đã có ít nhất một application ở BẤT KỲ trạng thái nào, khóa toàn bộ schedule_type, schedule_mode, schedule_start_date, schedule_end_date, time_slots (kể cả gửi giá trị cũ). Không đổi đơn/ca. Khi đổi địa bàn gửi cả cặp mã; city server đồng bộ; chỉ đổi category sang ngành hoạt động. BR46 CHỜ CHỐT: tin REJECTED sửa/gửi lại; tin APPROVED chưa có đơn sửa nội dung/lịch đề xuất quay PENDING; sau khi có đơn chưa chốt phạm vi sửa nội dung và công khai. Không tự reset kiểm duyệt, vòng đời hoặc dữ liệu đơn. API-11 chỉ triển khai chính sách mới sau xác nhận BR46/45/47; OwnerJobDetail khi thành công.

| HTTP | error.code | Điều kiện |
| --- | --- | --- |
| 400 | EMPTY_PATCH | Không có trường nào. |
| 400 | VALIDATION_ERROR | Dữ liệu sau PATCH sai. |
| 400 | CATEGORY_NOT_FOUND | Ngành nghề mới không tồn tại. |
| 400 | FIELD_NOT_ALLOWED | Gửi status/employer_id/required_workers/fields ngoài schema. |
| 403 | RESOURCE_FORBIDDEN | Sai chủ tin. |
| 404 | JOB_NOT_FOUND | Không có tin. |
| 409 | JOB_NOT_EDITABLE | Tin không còn RECRUITING. |
| 409 | SCHEDULE_LOCKED | Đã có ít nhất một đơn; không sửa trường lịch. |
| 400 | INVALID_LOCATION / CATEGORY_INACTIVE (đề xuất) | Địa bàn hoặc danh mục mới không hợp lệ. |
| 409 | MODERATION_EDIT_NOT_ALLOWED (đề xuất) | Thay đổi trái chính sách BR46 đã chốt. |

### 4.12. API-12 — NTD xem danh sách tin của mình

GET /api/employer/jobs | EMPLOYER ACTIVE | US07/US18/US19 ↔ FR9/FR6/FR25

Kết quả: 200 data = OwnerJobSummary[] + meta.

Query: status = một trong năm trạng thái tin (tùy chọn), page, size. Không nhận employer_id; lấy theo JWT. Tin CANCELLED/EXPIRED vẫn có trong danh sách quản lý; danh sách chỉ là dữ liệu quản lý, KHÔNG phải module thống kê tuyển dụng US29. Sắp created_at DESC, job_id DESC. Thêm moderation_status=PENDING|APPROVED|REJECTED (đề xuất), AND với status; không lọc công khai, chủ tin thấy tin chưa duyệt/bị từ chối và lý do. NTD ACTIVE chưa xác minh không tự mất quyền đọc lịch sử của mình.

| HTTP | error.code | Điều kiện |
| --- | --- | --- |
| 400 | INVALID_QUERY | Sai status hoặc phân trang. |

### 4.13. API-13 — SV ứng tuyển CV PDF và chọn từng ca

POST /api/jobs/{job_id}/applications  |  Quyền: STUDENT  |  Truy vết: US12 ↔ FR4

Kết quả: 201 data = Application; Location: /api/applications/{application_id}.

multipart/form-data; chỉ có cv (File PDF <=5 MiB) và selected_slot_ids (Text chứa JSON array BIGINT string) cho tin có ca. Ví dụ Postman: key cv/File=cv.pdf; key selected_slot_ids/Text=["11","12"]. Phía browser dùng FormData.append("selected_slot_ids", JSON.stringify(ids)), không tự cài Content-Type boundary.

```text
POST /api/jobs/301/applications
Authorization: Bearer <student_token>
Content-Type: multipart/form-data; boundary=...
cv: <PDF bytes>
selected_slot_ids: ["11","12"]
```

Chỉ nộp một đơn/student/job; tin phải RECRUITING; selected_slot_ids không trống, không trùng, chỉ chứa job_slot_id thuộc đúng tin. ONE_TIME: chỉ chọn ca có start_at > thời điểm BE nhận đơn (ca đã bắt đầu bị cấm). WEEKLY: ca được chọn phải còn ít nhất một lần phát sinh với giờ bắt đầu tương lai trong [schedule_start_date, schedule_end_date] nếu có end; WEEKLY vô hạn cho chọn khi còn lần tương lai. Không yêu cầu schedule_mode=ANY/ALL thỏa và không yêu cầu SV khai lịch rảnh. Nếu có lịch rảnh mà không khớp, FE cảnh báo nhưng BE vẫn nhận đơn hợp lệ. Bắt buộc tin APPROVED kiểm duyệt, chủ tin đủ điều kiện công khai, SV ACTIVE; kiểm tra predicate dùng chung và thời gian tại BE ngay trước ghi, trong transaction. Nếu tin không được phép công khai trả 404 JOB_NOT_FOUND cho người không có quyền đọc (đề xuất), tin công khai nhưng không còn nhận đơn trả 409 JOB_NOT_RECRUITING. Không bypass bằng ID trực tiếp; BR47/45 phải chốt trước tích hợp. Kể cả SV đã có đơn có quyền đọc lịch sử, quyền đọc không mở lại quyền ứng tuyển. Kiểm tra trùng theo UNIQUE(student_id,job_id).

Ngoại lệ tin legacy time_slots=[]: không gửi selected_slot_ids hoặc gửi [] (không lựa chọn ca), vẫn được nộp CV theo hợp đồng cũ. Phải xử lý tài nguyên/ca và kiểm tra giờ trong transaction gắn khóa job_posts; tránh mở form trước giờ nhưng gửi sau giờ lọt vào. File lưu riêng tư; khi DB lỗi dọn file vừa lưu; không trả storage key.

```text
{
  "data": {
    "application_id": "1001",
    "job_id": "301",
    "student_id": "101",
    "status": "PENDING",
    "status_before_cancel": null,
    "selected_slots": [
      {
        "job_slot_id": "11",
        "start_at": "2026-11-15T11:00:00Z",
        "end_at": "2026-11-15T15:00:00Z"
      }
    ],
    "cv": {
      "original_name": "cv.pdf",
      "mime_type": "application/pdf",
      "size_bytes": 124000,
      "download_url": "/api/applications/1001/cv"
    },
    "applied_at": "2026-11-14T08:00:00Z",
    "decided_at": null
  }
}
```

| HTTP | error.code | Điều kiện |
| --- | --- | --- |
| 400 | CV_REQUIRED | Thiếu CV. |
| 400 | INVALID_MULTIPART | Sai fields hoặc selected_slot_ids không phải JSON array. |
| 400 | CV_EMPTY | File rỗng. |
| 400 | VALIDATION_ERROR | Thiếu/trùng/sai ID ca, không chọn ca cho tin có ca, hoặc ca không thuộc tin. |
| 413 | CV_TOO_LARGE | PDF lớn hơn 5 MiB. |
| 415 | CV_TYPE_NOT_ALLOWED | Không phải PDF thật. |
| 404 | JOB_NOT_FOUND | Không có tin. |
| 409 | JOB_NOT_RECRUITING | Tin không còn RECRUITING. |
| 409 | SHIFT_ALREADY_STARTED | Một hoặc nhiều ca ONE_TIME chọn đã bắt đầu / WEEKLY không còn lần hợp lệ tương lai. |
| 409 | APPLICATION_ALREADY_EXISTS | SV đã có đơn tại tin. |
| 500 | APPLICATION_SAVE_FAILED | Lỗi file/DB; không để đơn nửa chừng. |

### 4.14. API-14 — SV xem danh sách đơn, kết quả và ca đã chọn

GET /api/student/applications  |  Quyền: STUDENT  |  Truy vết: US16/US17 ↔ FR6

Kết quả: 200 data = StudentApplicationItem[] + meta.

Query: status tùy chọn PENDING|APPROVED|REJECTED|JOB_EXPIRED|JOB_CANCELLED, page/size. Chỉ đọc đơn của chính student_id từ JWT. Trả cả application.status và job.status riêng; selected_slots, decided_at, status_before_cancel để hiển thị khi bị hủy. Sort applied_at DESC, application_id DESC.

| HTTP | error.code | Điều kiện |
| --- | --- | --- |
| 400 | INVALID_QUERY | Sai enum/giới hạn/query. |

### 4.15. API-15 — NTD xem ứng viên và ca đã đăng ký

GET /api/jobs/{job_id}/applications  |  Quyền: EMPLOYER + chủ tin  |  Truy vết: US13 ↔ FR5

Kết quả: 200 data = EmployerApplicationItem[] + meta.

Query: status tùy chọn gồm 5 trạng thái đơn, page/size. Chỉ chủ tin; trả ca đã chọn, CV metadata và điểm hiệu lực dựa trên decided_at khi có. Với đơn PENDING có các ca đã bắt đầu và ca tương lai, FE phải hiện cảnh báo duyệt muộn. Không công khai CV cho bên khác.

| HTTP | error.code | Điều kiện |
| --- | --- | --- |
| 400 | INVALID_QUERY | Sai query. |
| 403 | RESOURCE_FORBIDDEN | Sai chủ tin. |
| 404 | JOB_NOT_FOUND | Không tồn tại tin. |

### 4.16. API-16 — Chi tiết đơn, lịch ca và hiệu lực sau duyệt

GET /api/applications/{application_id}  |  Quyền: SV của đơn hoặc EMPLOYER chủ tin  |  Truy vết: US13/US16 ↔ FR5/FR6

Kết quả: 200 data = ApplicationDetail.

Trả ApplicationDetail gồm selected_slots bất biến, decided_at và status_before_cancel. Với ONE_TIME nếu đơn APPROVED, mỗi selected_slot có thêm trường dẫn xuất "approval_effect": "EFFECTIVE" nếu start_at > decided_at, ngược lại "PASSED_BEFORE_APPROVAL"; nếu đơn chưa APPROVED: "NOT_APPROVED". Đây là biểu diễn DTO, không phải cột job_time_slots. WEEKLY tính các lần phát sinh sau decided_at; không gộp thành lịch chấm công mới.

SV và chủ tin đều đọc được ca cũ và ca tương lai đã chọn; không tự xóa lựa chọn khi quá thời gian. Đối với JOB_CANCELLED, hiển thị trạng thái chính "Công việc đã hủy" và nếu cần trạng thái trước hủy từ status_before_cancel; decided_at gốc giữ nguyên.

| HTTP | error.code | Điều kiện |
| --- | --- | --- |
| 400 | INVALID_ID | Sai ID. |
| 403 | RESOURCE_FORBIDDEN | Không liên quan đơn hoặc ADMIN. |
| 404 | APPLICATION_NOT_FOUND | Không có đơn. |

### 4.17. API-17 — Truy cập CV đã nộp

200 bytes PDF, không bọc data; lỗi vẫn JSON chung.

GET /api/applications/{application_id}/cv  

Quyền: SV sở hữu đơn HOẶC NTD sở hữu tin. Truy vết: US12, US13 (hỗ trợ) ↔ FR4, FR5.

Request:

Không có body. Không nhận query. Path ID (nếu có) theo mục 2.1.

Validation và xử lý: Path application_id hợp lệ; không có query; gửi Authorization Bearer. Server kiểm tra quyền cho từng lần truy cập trước khi stream file, kể cả biết URL. Thành công trả bytes PDF gốc, Content-Type: application/pdf; Content-Disposition: inline; filename="cv.pdf"; Cache-Control: private, no-store. Frontend có thể fetch kèm token rồi hiển thị blob. Không dùng URL công khai, không trả JSON data khi thành công. Lỗi vẫn trả JSON theo mục 2.2. CV bất biến khi cập nhật hồ sơ hoặc nộp file khác ở tin khác.

Thành công: HTTP 200.

Lỗi riêng (cộng với lỗi chung tại mục 2.3):

| HTTP | error.code | Điều kiện |
| --- | --- | --- |
| 403 | RESOURCE_FORBIDDEN | Không phải SV của đơn hoặc chủ tin; Admin cũng bị chặn. |
| 404 | APPLICATION_NOT_FOUND | Không có đơn. |
| 500 | CV_UNAVAILABLE | Đơn có metadata nhưng file không đọc được; không trả file khác thay thế. |

### 4.18. API-18 — NTD duyệt muộn hoặc từ chối đơn

PATCH /api/applications/{application_id}/status  |  Quyền: EMPLOYER + chủ tin  |  Truy vết: US14/US15/US22 ↔ FR5/FR8

Kết quả: 200 data = Application sau cập nhật.

```text
{
  "status": "APPROVED",
  "confirm_late_approval": true
}
```

status bắt buộc APPROVED hoặc REJECTED. confirm_late_approval là boolean TÙY CHỌN, chỉ dùng khi duyệt đơn có ít nhất một ca đã bắt đầu và ít nhất một ca còn tương lai; nếu thiếu/false, trả 409 LATE_APPROVAL_CONFIRMATION_REQUIRED kèm details để FE hiển thị popup. Nếu toàn bộ ca SV chọn đã bắt đầu thì không được duyệt; scheduler sẽ chuyển PENDING→JOB_EXPIRED, trả 409 APPLICATION_EXPIRED khi xử lý muộn. Nếu không có ca đã bắt đầu thì confirm_late_approval có thể vắng; nếu REJECTED không gửi cờ này.

Khi duyệt: chỉ từ PENDING, tin RECRUITING, còn ít nhất một ca/lần WEEKLY tương lai; ghi decided_at là thời điểm chấp thuận. Không xóa ca đã qua; ca có start > decided_at mới có hiệu lực. Khi từ chối: PENDING→REJECTED và ghi decided_at; tin không tự đổi. Quyết định + INSERT notification thực hiện trong một transaction; gửi lặp bị 409, không tạo hai thông báo. Tin legacy không ca: cho phép quyết định PENDING khi RECRUITING như cũ.

```text
{
  "data": {
    "application_id": "1001",
    "status": "APPROVED",
    "decided_at": "2026-11-16T03:00:00Z",
    "selected_slots": [
      {
        "job_slot_id": "11",
        "start_at": "2026-11-15T01:00:00Z",
        "end_at": "2026-11-15T05:00:00Z"
      },
      {
        "job_slot_id": "12",
        "start_at": "2026-11-17T01:00:00Z",
        "end_at": "2026-11-17T05:00:00Z"
      }
    ]
  }
}
```

| HTTP | error.code | Điều kiện |
| --- | --- | --- |
| 400 | VALIDATION_ERROR | status sai hoặc confirm_late_approval không đúng kiểu. |
| 403 | RESOURCE_FORBIDDEN | Không phải chủ tin. |
| 404 | APPLICATION_NOT_FOUND | Không có đơn. |
| 409 | APPLICATION_ALREADY_DECIDED | Đơn không còn PENDING. |
| 409 | APPLICATION_EXPIRED | Không còn ca/lần làm tương lai để duyệt. |
| 409 | LATE_APPROVAL_CONFIRMATION_REQUIRED | Có ca đã qua nhưng NTD chưa xác nhận duyệt muộn. |
| 409 | JOB_NOT_RECRUITING | Tin không còn RECRUITING. |

### 4.19. API-19 — NTD đóng tuyển, xác nhận hoàn thành hoặc hủy tin

PATCH /api/jobs/{job_id}/status  |  Quyền: EMPLOYER + chủ tin  |  Truy vết: US18/US19/US45 ↔ FR6/FR23

Kết quả: 200 data = {job_id,status,completed_at,cancelled_at,cancel_reason,updated_at}.

Các request JSON được hỗ trợ:

```text
{"status":"FILLED"}
{"status":"COMPLETED"}
{"status":"CANCELLED","cancel_reason":"Sự kiện không còn tổ chức theo kế hoạch."}
```

| Chuyển trạng thái | Điều kiện bắt buộc | Hệ quả |
| --- | --- | --- |
| RECRUITING→FILLED (NTD) | >=1 APPROVED, 0 PENDING | Chủ tin đã xử lý hết đơn chờ; không tự REJECTED hàng loạt; dừng ứng tuyển mới. |
| FILLED→COMPLETED (NTD) | NTD xác nhận công việc hoàn thành thực tế | Lưu completed_at; cho phép đánh giá chỉ với đơn APPROVED. |
| RECRUITING/FILLED→CANCELLED (NTD) | CURRENT_DATE Asia/Ho_Chi_Minh < ngày bắt đầu ca đầu; đúng chủ; có ứng viên thì cancel_reason không rỗng | Đổi mọi đơn thành JOB_CANCELLED, lưu status_before_cancel, giữ decided_at, ca, CV; thông báo từng SV kèm lý do; lưu cancelled_at. |
| RECRUITING→FILLED (nội bộ) | Đã hết CA CUỐI, có >=1 APPROVED | Tự chuyển; những PENDING chưa được xử lý tự JOB_EXPIRED kèm thông báo theo quy tắc lịch. |
| RECRUITING→EXPIRED (nội bộ) | Đã hết CA CUỐI, 0 APPROVED | Tự chuyển; PENDING trở thành JOB_EXPIRED, tạo thông báo. |

Ngưỡng HỦY là trước NGÀY bắt đầu ca sớm nhất (không phải trước GIỜ): ca đầu 15/11 18:00, NTD chỉ hủy trước 00:00 ngày 15/11 theo Asia/Ho_Chi_Minh. Nếu tin legacy không có dữ liệu ca để suy ra ngày đầu, trả JOB_CANCEL_WINDOW_UNDETERMINED, không đoán từ work_time TEXT. Nếu tin không có ứng viên, cancel_reason có thể null; nếu có bất kỳ đơn, lý do trim 1–1000 ký tự. Các trạng thái COMPLETED/EXPIRED/CANCELLED không thể hủy; không quay ngược, không mở tuyển lại.

Khi hủy, mọi đơn PENDING/APPROVED/REJECTED/JOB_EXPIRED → JOB_CANCELLED bất kể đã có quyết định; status_before_cancel ghi ngay trước khi đổi, decided_at giữ nguyên; thông báo gắn application_id và nội dung lý do cho mọi SV. Thực hiện update tin + update tất cả đơn + insert notification cùng transaction, lock job_posts trước các đơn; nếu một thông báo lỗi rollback toàn bộ. Không hủy review đang hợp lệ bởi thao tác hủy không cho phép sau ngày đầu; không tự COMPLETED.

| HTTP | error.code | Điều kiện |
| --- | --- | --- |
| 400 | VALIDATION_ERROR | status không hỗ trợ hoặc cancel_reason vượt giới hạn/không đúng kiểu. |
| 403 | RESOURCE_FORBIDDEN | Không phải chủ tin. |
| 404 | JOB_NOT_FOUND | Không có tin. |
| 409 | PENDING_APPLICATIONS_EXIST | Đóng FILLED thủ công khi còn PENDING (ưu tiên lỗi này). |
| 409 | NO_APPROVED_APPLICATION | Đóng FILLED thủ công khi không có APPROVED. |
| 409 | CANCEL_REASON_REQUIRED | Hủy đã có ứng viên nhưng không có lý do. |
| 409 | JOB_CANCEL_WINDOW_CLOSED | Đã đến ngày bắt đầu ca đầu tiên. |
| 409 | JOB_CANCEL_WINDOW_UNDETERMINED | Tin cũ không có lịch cấu trúc để xác định ngày hủy. |
| 409 | INVALID_JOB_TRANSITION | Sai thứ tự/trạng thái không đổi/hủy tin không hợp lệ. |

### 4.20. API-20 — Đánh giá hai chiều sau hoàn thành

201 data = Review.

POST /api/applications/{application_id}/reviews  

Quyền: SV của đơn HOẶC NTD chủ tin. Truy vết: US20, US21 ↔ FR7.

Request:

```text
{
  "rating": 5,
  "comment": "Thông tin công việc rõ ràng, phối hợp tốt."
}
```

Validation và xử lý: Body chỉ có rating bắt buộc integer 1…5, comment tùy chọn null/chuỗi ≤2.000. Đơn phải APPROVED và tin COMPLETED. Server lấy reviewer_id từ token; nếu là SV của đơn thì reviewee_id = chủ tin, nếu là chủ tin thì reviewee_id = SV của đơn. Không nhận reviewer_id/reviewee_id từ client. Mỗi chiều tối đa 1 đánh giá, UNIQUE(application_id, reviewer_id); tối đa hai đánh giá/đơn. Không có API chỉnh sửa/xóa đánh giá trong bản này. NTD đánh giá từng SV bằng application_id khác nhau.

Thành công: HTTP 201; ID review mới nằm trong response body.

Lỗi riêng (cộng với lỗi chung tại mục 2.3):

| HTTP | error.code | Điều kiện |
| --- | --- | --- |
| 400 | VALIDATION_ERROR | Điểm/nhận xét không hợp lệ. |
| 403 | RESOURCE_FORBIDDEN | Sai cặp người dùng hoặc Admin. |
| 404 | APPLICATION_NOT_FOUND | Không có đơn. |
| 409 | REVIEW_NOT_ALLOWED | Tin chưa COMPLETED hoặc đơn chưa APPROVED. |
| 409 | REVIEW_ALREADY_EXISTS | Người này đã đánh giá đơn đó. |

### 4.21. API-21 — Hai bên xem đánh giá của đơn

200 data = Review[], tối đa 2 phần tử, không meta.

GET /api/applications/{application_id}/reviews  

Quyền: SV của đơn HOẶC NTD chủ tin. Truy vết: US20, US21 (hỗ trợ hiển thị) ↔ FR7.

Request:

Không có body. Không nhận query. Path ID (nếu có) theo mục 2.1.

Validation và xử lý: Path application_id hợp lệ; không có query. Chỉ hai bên của đơn xem, không công khai hồ sơ đánh giá của người khác. Danh sách 0…2 review, không phân trang; sort created_at ASC, review_id ASC. Chưa có đánh giá trả data: []. Không có API điểm trung bình/thống kê mở rộng.

Thành công: HTTP 200.

Lỗi riêng (cộng với lỗi chung tại mục 2.3):

| HTTP | error.code | Điều kiện |
| --- | --- | --- |
| 403 | RESOURCE_FORBIDDEN | Không thuộc hai bên của đơn. |
| 404 | APPLICATION_NOT_FOUND | Không có đơn. |

### 4.22. API-22 — SV xem thông báo xét duyệt, hết hạn và hủy tin

GET /api/notifications  |  Quyền: STUDENT  |  Truy vết: US22/US45 ↔ FR8/FR23

Kết quả: 200 data = Notification[] + meta.

Query chỉ page, size; recipient_id luôn từ JWT. Các nguồn notification: NTD APPROVED/REJECTED (API-18); đơn tự JOB_EXPIRED; tin tự FILLED/EXPIRED có đơn bị ảnh hưởng; NTD hủy tin CANCELLED (API-19) thông báo cho tất cả SV có đơn và chèn nguyên văn lý do đã trim. Một sự kiện không tạo thông báo trùng nếu worker chạy lại; thông báo tạo cùng transaction đổi trạng thái. Không WebSocket, email, đánh dấu đã đọc hoặc tự gửi thông báo trực tiếp giữa người dùng.

```text
{
  "data": [
    {
      "notification_id": "9001",
      "application_id": "1001",
      "recipient_id": "101",
      "message": "Nhà tuyển dụng đã hủy tin “Hỗ trợ sự kiện”. Lý do: Sự kiện không còn tổ chức.",
      "created_at": "2026-11-14T03:00:00Z"
    }
  ],
  "meta": {
    "page": 1,
    "size": 20,
    "total_elements": 1,
    "total_pages": 1
  }
}
```

| HTTP | error.code | Điều kiện |
| --- | --- | --- |
| 400 | INVALID_QUERY | Sai page/size hoặc gửi recipient_id/query lạ. |

### 4.23. API-23 — Admin xem danh sách tài khoản và lọc role

200 data = AdminUserItem[] và meta.

GET /api/admin/users  

Quyền: ADMIN. Truy vết: US39 ↔ FR19.

Query được hỗ trợ:

| Tên | Kiểu | Quy tắc |
| --- | --- | --- |
| role | STUDENT hoặc EMPLOYER | Tùy chọn; không gửi = cả hai. role=ADMIN/ALL hoặc giá trị khác là 400 INVALID_QUERY. |
| page, size | integer | Theo mục 2.2. |
| account_status (đề xuất) | ACTIVE\|LOCKED | Tùy chọn, AND với role; không gửi = cả hai. |

Request:

Không có body. Query theo bảng phía trên. Path ID (nếu có) theo mục 2.1.

Validation và xử lý: Chỉ ADMIN được gọi. Join users với hồ sơ đúng role để lấy tên hiển thị; danh sách chỉ gồm STUDENT/EMPLOYER, không trả tài khoản ADMIN. Bộ lọc Tất cả = bỏ role khỏi query. Trả đúng AdminUserItem, không trả password, password_hash, CV, storage key, token hoặc hồ sơ riêng chi tiết. Sort created_at DESC, user_id DESC; không có kết quả trả mảng rỗng. API này chỉ đọc; khóa/mở khóa qua API-37 đề xuất, không sửa/xóa tài khoản.

Thành công: HTTP 200.

Lỗi riêng (cộng với lỗi chung tại mục 2.3):

| HTTP | error.code | Điều kiện |
| --- | --- | --- |
| 400 | INVALID_QUERY | Sai role/phân trang hoặc query không hỗ trợ. |
| 401 | AUTH_REQUIRED / INVALID_TOKEN | Chưa xác thực hợp lệ. |
| 403 | ROLE_FORBIDDEN | SV/NTD cố đọc danh sách Admin. |

### 4.24. API-24 — SV xem lịch rảnh tuần của chính mình

GET /api/student/availability  |  Quyền: STUDENT  |  Truy vết: US42 ↔ FR20

Kết quả: 200 data = {slots: StudentAvailabilitySlot[]}.

Không query/body/student_id. Lịch rảnh có thể rỗng ([]); WEEKLY lặp 1=Thứ Hai...7=Chủ nhật, giờ theo Asia/Ho_Chi_Minh. Cho phép start_time>end_time = một khoảng qua đêm, không tách thành 2 slot logic.

```text
{
  "data": {
    "slots": [
      {
        "availability_slot_id": "71",
        "day_of_week": 7,
        "start_time": "21:00",
        "end_time": "03:00"
      }
    ]
  }
}
```

### 4.25. API-25 — SV thay thế lịch rảnh tuần của mình

PUT /api/student/availability  |  Quyền: STUDENT  |  Truy vết: US42 ↔ FR20

Kết quả: 200 data cùng API-24.

```text
{
  "slots": [
    {
      "day_of_week": 1,
      "start_time": "07:00",
      "end_time": "11:00"
    },
    {
      "day_of_week": 7,
      "start_time": "21:00",
      "end_time": "03:00"
    }
  ]
}
```

PUT thay toàn bộ 0–21 khoảng logic; [] xóa hết lịch; thiếu slots hoặc không đúng mảng trả 400. start_time == end_time sai; end_time < start_time là ca đêm hợp lệ (chưa đến 24h). Chống chồng lấn theo trục tuần tuần hoàn, kể cả CN→T2; khoảng liền nhau có thể tồn tại. Không trộn nhiều khoảng để khớp một ca. Lock transaction, validation toàn bộ trước khi xóa lịch cũ; sai dữ liệu rollback, không sửa lịch trước đó. Không cho EMPLOYER/ADMIN truy cập.

| HTTP | error.code | Điều kiện |
| --- | --- | --- |
| 400 | VALIDATION_ERROR | Không có slots, >21, giờ/thứ sai, trùng hoặc chồng lấn kể cả CN→Thứ Hai. |
| 403 | ROLE_FORBIDDEN | Không phải STUDENT. |

### 4.26. API-26 — Gợi ý công việc phù hợp lịch rảnh

GET /api/student/job-recommendations  |  Quyền: STUDENT  |  Truy vết: US44/US43 ↔ FR22/FR21

Kết quả: 200 data = RecommendedJobItem[] + meta.

Query: page/size, city <=100, category_id BIGINT; các điều kiện AND. Chỉ tin RECRUITING, có ca khả dụng trong lịch hiệu lực, lịch rảnh SV từ JWT không rỗng. ONE_TIME chỉ tính các ca chưa bắt đầu (start_at>now) vào cả tử và mẫu; WEEKLY chỉ xét ca có ít nhất một lần phát sinh tương lai trong [schedule_start_date,schedule_end_date] nếu có end. Tin cũ không ca, tin FILLED/EXPIRED/CANCELLED không được gợi ý. Không lấy lịch rảnh SV khác. Bổ sung điều kiện moderation_status=APPROVED và đủ điều kiện công khai như API-09; hỗ trợ locations theo cùng encoding đề xuất, không dùng cùng city. BR45/47 chờ chốt; lịch sử đơn không phụ thuộc Matching.

Khớp trọn MỘT ca trong MỘT khoảng rảnh liên tục; qua đêm trên trục 48 giờ, kiểm tra cả ca bắt đầu hôm trước và CN→T2. Không cộng gộp các khoảng rời. ANY: >=1 ca khớp; ALL: tất cả ca đang xét khớp. match_percent = ROUND(100×matched_slots_count/total_slots_count) với total>0, sort match_percent DESC, created_at DESC, job_id DESC. SV không có lịch thì data:[]; vẫn tự xem và ứng tuyển API-09/13 được.

```text
{
  "data": [
    {
      "job": {
        "job_id": "301",
        "title": "Hỗ trợ sự kiện",
        "status": "RECRUITING",
        "schedule_type": "ONE_TIME"
      },
      "schedule_mode": "ANY",
      "matched_slots": [
        {
          "job_slot_id": "11",
          "start_at": "2026-11-17T15:00:00Z",
          "end_at": "2026-11-17T19:00:00Z"
        }
      ],
      "matched_slots_count": 1,
      "total_slots_count": 2,
      "match_percent": 50
    }
  ],
  "meta": {
    "page": 1,
    "size": 20,
    "total_elements": 1,
    "total_pages": 1
  }
}
```

Ví dụ JSON được rút gọn ở trường job để đọc; phản hồi thật phải trả JobSummary đầy đủ theo mục 3.2. Thứ và khung giờ ONE_TIME xác định ở Asia/Ho_Chi_Minh, không dùng ngày UTC trực tiếp làm day_of_week.

| HTTP | error.code | Điều kiện |
| --- | --- | --- |
| 400 | INVALID_QUERY | Sai city/category_id/locations/page/size, nhóm địa bàn sai/lặp hoặc tham số lạ/lặp. |
| 403 | ROLE_FORBIDDEN | Sai vai trò. |

### 4.27. API-27 — Đọc danh mục tỉnh/thành (ĐỀ XUẤT)

GET /api/locations/provinces | Quyền: Công khai | US02/06/07/11.

Cổng triển khai: BR43/50: nguồn, phiên bản, hiệu lực, mã và FK được xác nhận.

FR3/BR43. GET không query/body. Trả 200 data: Province[] = province_code,name,source_version; chỉ is_active=true; sort name ASC, province_code ASC; không phân trang. Nguồn phiên bản phải được xác minh, không tự đặt mã.

Response thành công minh họa:

```text
{
  "data": [
    {
      "province_code": "P_TEST",
      "name": "Tỉnh minh họa",
      "source_version": "FIXTURE"
    }
  ]
}
```

| HTTP | error.code | Điều kiện |
| --- | --- | --- |
| 400 | INVALID_QUERY | Có query không hỗ trợ. |

### 4.28. API-28 — Đọc phường/xã/đặc khu theo tỉnh (ĐỀ XUẤT)

GET /api/locations/wards | Quyền: Công khai | US02/06/07/11.

Cổng triển khai: BR43/50: cùng nguồn với API-27.

FR3/BR43. Query province_code bắt buộc, chuỗi 1–16; tỉnh hoạt động; chỉ wards hoạt động thuộc tỉnh; 200 data: Ward[] = ward_code,province_code,name,source_version; sort name ASC, ward_code ASC; không phân trang. Không thêm quận/huyện.

Request minh họa:

```text
GET /api/locations/wards?province_code=P_TEST
```

Response thành công minh họa:

```text
{
  "data": [
    {
      "ward_code": "W_TEST",
      "province_code": "P_TEST",
      "name": "Phường minh họa",
      "source_version": "FIXTURE"
    }
  ]
}
```

| HTTP | error.code | Điều kiện |
| --- | --- | --- |
| 400 | INVALID_QUERY | Thiếu/sai mã tỉnh, tỉnh không tồn tại/ngừng dùng, query lạ. |

### 4.29. API-29 — Danh mục lĩnh vực doanh nghiệp (ĐỀ XUẤT)

GET /api/business-industries | Quyền: Công khai | US02/38.

Cổng triển khai: BR48: quyết định FK và nguồn seed trước code.

FR1/FR18/BR48. GET không query/body; 200 data: [{industry_id,name}], chỉ đang hoạt động, sort name ASC, industry_id ASC; không phân trang. Nếu chọn bảng riêng: business_industries; nếu dùng chung: ánh xạ category_id → industry_id từ job_categories (DTO). Không tạo cả hai nguồn, không bổ sung CRUD lĩnh vực riêng ngoài FR27.

Response thành công minh họa:

```text
{
  "data": [
    {
      "industry_id": "1",
      "name": "Lĩnh vực minh họa"
    }
  ]
}
```

| HTTP | error.code | Điều kiện |
| --- | --- | --- |
| 400 | INVALID_QUERY | Query lạ. |

### 4.30. API-30 — Danh sách nhà tuyển dụng cần xác minh (ĐỀ XUẤT)

GET /api/admin/employers | Quyền: ADMIN | US46.

Cổng triển khai: Tên API/DTO/lỗi mới cần phê duyệt BR50 trước code.

FR24. Query verification_status=PENDING|APPROVED|REJECTED (tùy chọn, mặc định PENDING; nếu cần tất cả FE gửi scope=all thay vì status), scope=all tùy chọn không cùng verification_status, page,size; 200 data AdminEmployerItem[] + meta. Sort created_at DESC,user_id DESC; chỉ EMPLOYER, gồm cả LOCKED để Admin biết tình trạng. Không trả key/file, không kèm CV.

| HTTP | error.code | Điều kiện |
| --- | --- | --- |
| 400 | INVALID_QUERY | Sai enum, scope, phân trang hoặc query lạ. |

### 4.31. API-31 — Xem chi tiết hồ sơ NTD (ĐỀ XUẤT)

GET /api/admin/employers/{user_id} | Quyền: ADMIN | US46.

Cổng triển khai: Tên API/DTO/lỗi mới cần phê duyệt BR50 trước code.

FR24. Path user_id, không query/body; chỉ role EMPLOYER mục tiêu. 200 data AdminEmployerDetail; trả thông tin, trạng thái, lý do, URL bảo vệ tới API-32; không trả key. Không xem CV hoặc lịch rảnh qua chức năng này.

| HTTP | error.code | Điều kiện |
| --- | --- | --- |
| 400 | INVALID_ID | ID sai. |
| 404 | EMPLOYER_NOT_FOUND (đề xuất) | Không có NTD tương ứng. |

### 4.32. API-32 — Đọc minh chứng xác minh (ĐỀ XUẤT)

GET /api/employers/{user_id}/verification-document | Quyền: ADMIN hoặc chủ hồ sơ | US37/46.

Cổng triển khai: BR45/51: loại file, lưu/xóa và phân quyền storage.

FR17/FR24/NFR6. Path user_id; không query/body; chỉ ADMIN theo nhiệm vụ hoặc đúng EMPLOYER ACTIVE chủ hồ sơ. 200 bytes file đã lưu, Content-Type đã xác minh, Content-Disposition attachment với tên an toàn do server đặt, Cache-Control: private, no-store; không URL public/signed link dùng lại, không lộ key; mọi lần truy cập kiểm quyền. Không trả JSON khi thành công.

| HTTP | error.code | Điều kiện |
| --- | --- | --- |
| 403 | RESOURCE_FORBIDDEN | Không đúng quyền. |
| 404 | VERIFICATION_DOCUMENT_NOT_FOUND (đề xuất) | Không có hồ sơ/file minh chứng. |
| 500 | VERIFICATION_DOCUMENT_UNAVAILABLE (đề xuất) | File đã ghi key nhưng không đọc được. |

### 4.33. API-33 — Duyệt hoặc từ chối NTD (ĐỀ XUẤT)

PATCH /api/admin/employers/{user_id}/verification | Quyền: ADMIN | US46.

Cổng triển khai: BR45/48/50/51: tiêu chí minh chứng, danh mục, gửi lại và chuyển tiếp.

FR24. Chỉ hồ sơ PENDING, khóa bản ghi, kiểm tra role ADMIN và tính đầy đủ hồ sơ/minh chứng trước APPROVED. Body verification_status=APPROVED|REJECTED; reject_reason bắt buộc khi REJECTED, trim 1–1000 (đề xuất), vắng/null khi APPROVED. Set verified_by từ JWT, verified_at từ DB; không tự thay account_status, không duyệt tin. 200 data EmployerVerificationDecision = user_id,verification_status,reject_reason,verified_at,verified_by; gửi lặp 409. NTD xem kết quả qua API-05; không tạo notification loại mới vì bảng notifications gắn application_id, chưa có yêu cầu email.

Request minh họa:

```text
{
  "verification_status": "REJECTED",
  "reject_reason": "Cần bổ sung minh chứng đơn vị."
}
```

Response thành công minh họa:

```text
{
  "data": {
    "user_id": "201",
    "verification_status": "REJECTED",
    "reject_reason": "Cần bổ sung minh chứng đơn vị.",
    "verified_at": "2026-11-24T04:00:00Z",
    "verified_by": "901"
  }
}
```

| HTTP | error.code | Điều kiện |
| --- | --- | --- |
| 400 | VALIDATION_ERROR | Sai status/lý do. |
| 404 | EMPLOYER_NOT_FOUND (đề xuất) | Không có NTD. |
| 409 | VERIFICATION_ALREADY_DECIDED / VERIFICATION_INCOMPLETE (đề xuất) | Không PENDING hoặc không đủ hồ sơ/minh chứng. |

### 4.34. API-34 — Danh sách tin kiểm duyệt (ĐỀ XUẤT)

GET /api/admin/jobs | Quyền: ADMIN | US47.

Cổng triển khai: Tên API/DTO/lỗi mới cần phê duyệt BR50 trước code.

FR25. Query moderation_status (3 giá trị, mặc định PENDING), scope=all (không cùng moderation_status), status (5 vòng đời), page,size; AND. 200 AdminJobSummary[] + meta, sort created_at DESC,job_id DESC. Không lọc công khai; không trả CV/đơn.

| HTTP | error.code | Điều kiện |
| --- | --- | --- |
| 400 | INVALID_QUERY | Sai enum/scope/query/phân trang. |

### 4.35. API-35 — Chi tiết tin cần kiểm duyệt (ĐỀ XUẤT)

GET /api/admin/jobs/{job_id} | Quyền: ADMIN | US47.

Cổng triển khai: Tên API/DTO/lỗi mới cần phê duyệt BR50 trước code.

FR25. Path job_id; không query/body. 200 AdminJobDetail, gồm nội dung, địa bàn, lịch ca, cả hai status và metadata kiểm duyệt. Không truy cập CV hoặc lịch rảnh SV.

| HTTP | error.code | Điều kiện |
| --- | --- | --- |
| 400 | INVALID_ID | ID sai. |
| 404 | JOB_NOT_FOUND | Không tồn tại tin. |

### 4.36. API-36 — Duyệt hoặc từ chối tin (ĐỀ XUẤT)

PATCH /api/admin/jobs/{job_id}/moderation | Quyền: ADMIN | US47.

Cổng triển khai: BR46/47/50: kiểm duyệt lại, tác động khóa và chuyển tiếp.

FR25. Chỉ PENDING, khóa tin và kiểm lại trạng thái, owner và điều kiện hiện tại. Body moderation_status=APPROVED|REJECTED, moderation_reason bắt buộc REJECTED trim 1–1000 (đề xuất), null/vắng khi APPROVED. APPROVED chỉ khi còn đủ điều kiện công khai/nhận đơn; không duyệt tin đã hết lịch hoặc bị hủy. REJECTED không công khai; không tự đổi vòng đời/đơn. Set moderated_by, moderated_at từ server; 200 data {job_id,moderation_status,moderation_reason,moderated_at,moderated_by,status}. Gửi lặp 409; kiểm soát race với API-11/worker. NTD xem kết quả qua API-12/10. Chính sách gửi lại sau sửa BR46 chưa chốt.

Request minh họa:

```text
{
  "moderation_status": "APPROVED"
}
```

Response thành công minh họa:

```text
{
  "data": {
    "job_id": "301",
    "moderation_status": "APPROVED",
    "moderation_reason": null,
    "moderated_at": "2026-11-24T04:01:00Z",
    "moderated_by": "901",
    "status": "RECRUITING"
  }
}
```

| HTTP | error.code | Điều kiện |
| --- | --- | --- |
| 400 | VALIDATION_ERROR | Sai status/lý do. |
| 404 | JOB_NOT_FOUND | Không có tin. |
| 409 | MODERATION_ALREADY_DECIDED / JOB_NOT_MODERATABLE (đề xuất) | Không PENDING hoặc không đủ điều kiện duyệt. |

### 4.37. API-37 — Khóa hoặc mở khóa tài khoản (ĐỀ XUẤT)

PATCH /api/admin/users/{user_id}/status | Quyền: ADMIN | US40/41.

Cổng triển khai: BR47/50: tác động tin/đơn và token sau mở khóa phải được xác nhận.

FR26. Path user_id, chỉ STUDENT/EMPLOYER, không ADMIN. Body account_status=LOCKED|ACTIVE; lock_reason bắt buộc khóa, trim 1–1000 đề xuất; khi mở khóa không gửi lý do khóa mới. Lock users, kiểm tra trạng thái đầu; ACTIVE→LOCKED set locked_at/by/reason; LOCKED→ACTIVE set unlocked_at/by, giữ metadata khóa gần nhất. 200 AdminUserDetailStatus; lặp 409. Mọi API bảo vệ và login kiểm account_status DB. Không thêm token_version hoặc tự vô hiệu vĩnh viễn JWT sau mở khóa khi chưa chốt. Không tự đổi trạng thái tin/đơn; BR47 đề xuất ẩn tin khỏi public/Matching, giữ lịch sử, worker tiếp tục; cách xử lý đơn đang chờ/đang làm và hành vi token sau mở khóa cần chốt.

Request minh họa:

```text
{
  "account_status": "LOCKED",
  "lock_reason": "Cần xác minh hoạt động tài khoản."
}
```

Response thành công minh họa:

```text
{
  "data": {
    "user_id": "201",
    "role": "EMPLOYER",
    "account_status": "LOCKED",
    "locked_at": "2026-11-24T04:02:00Z",
    "locked_by": "901",
    "lock_reason": "Cần xác minh hoạt động tài khoản.",
    "unlocked_at": null,
    "unlocked_by": null
  }
}
```

| HTTP | error.code | Điều kiện |
| --- | --- | --- |
| 400 | VALIDATION_ERROR | Status/lý do sai. |
| 403 | ROLE_FORBIDDEN | Mục tiêu ADMIN hoặc caller sai role. |
| 404 | USER_NOT_FOUND (đề xuất) | Không có tài khoản. |
| 409 | INVALID_ACCOUNT_TRANSITION (đề xuất) | Trạng thái không đổi. |

### 4.38. API-38 — Xem danh mục quản trị (ĐỀ XUẤT)

GET /api/admin/job-categories | Quyền: ADMIN | US48.

Cổng triển khai: Tên API/DTO/lỗi mới cần phê duyệt BR50 trước code.

FR27. Query is_active=true|false tùy chọn, page,size; 200 data Category[] + meta. Không gửi is_active = cả hai. Sort name ASC,category_id ASC; dữ liệu có tham chiếu không bị xóa.

| HTTP | error.code | Điều kiện |
| --- | --- | --- |
| 400 | INVALID_QUERY | Sai boolean/phân trang/query. |

### 4.39. API-39 — Thêm danh mục ngành nghề (ĐỀ XUẤT)

POST /api/admin/job-categories | Quyền: ADMIN | US48.

Cổng triển khai: Tên API/DTO/lỗi mới cần phê duyệt BR50 trước code.

FR27. JSON chỉ name 1–100 trim không rỗng; tạo is_active=true, ID server cấp. 201 data Category. UNIQUE name theo schema; đề xuất chuẩn hóa kiểm trùng không phân biệt hoa/thường, cần đồng bộ index/constraint trước code. Không tự tạo danh mục lĩnh vực riêng.

Request minh họa:

```text
{
  "name": "Gia sư"
}
```

Response thành công minh họa:

```text
{
  "data": {
    "category_id": "9",
    "name": "Gia sư",
    "is_active": true
  }
}
```

| HTTP | error.code | Điều kiện |
| --- | --- | --- |
| 400 | VALIDATION_ERROR / FIELD_NOT_ALLOWED | Tên sai/field ngoài. |
| 409 | CATEGORY_ALREADY_EXISTS (đề xuất) | Tên trùng theo quy tắc đã chốt. |

### 4.40. API-40 — Sửa hoặc ngừng sử dụng danh mục (ĐỀ XUẤT)

PATCH /api/admin/job-categories/{category_id} | Quyền: ADMIN | US48.

Cổng triển khai: Tên API/DTO/lỗi mới cần phê duyệt BR50 trước code.

FR27. Path category_id; JSON chỉ name (1–100) và/hoặc is_active boolean, ít nhất một field; 200 Category. is_active=false ngừng dùng, true kích hoạt lại là đề xuất hỗ trợ quản trị; không DELETE; tin/đơn cũ giữ FK và hiển thị tên danh mục hiện tại. Không chọn FALSE khi tạo/đổi ngành; kiểm tra race với API-08/11. Quy tắc tên trùng giống API-39.

Request minh họa:

```text
{
  "is_active": false
}
```

Response thành công minh họa:

```text
{
  "data": {
    "category_id": "9",
    "name": "Gia sư",
    "is_active": false
  }
}
```

| HTTP | error.code | Điều kiện |
| --- | --- | --- |
| 400 | VALIDATION_ERROR / EMPTY_PATCH | Field sai/rỗng. |
| 404 | CATEGORY_NOT_FOUND (đề xuất) | Không có danh mục. |
| 409 | CATEGORY_ALREADY_EXISTS (đề xuất) | Tên trùng. |

### 4.41. API-41 — Dashboard thống kê cơ bản — SHOULD (ĐỀ XUẤT)

GET /api/admin/dashboard | Quyền: ADMIN | US49.

Cổng triển khai: SHOULD; BR47/50 và quy tắc đếm được chốt trước code. Số chỉ minh họa.

FR28. GET không query/body. 200 data gồm 6 số đếm integer không âm: total_students và total_employers theo users.role (gồm ACTIVE/LOCKED); pending_employers theo verification_status; pending_jobs theo moderation_status (đề xuất đếm mọi vòng đời); recruiting_jobs theo predicate mặc định API-09; total_applications mọi trạng thái. Truy vấn DB thật, không dữ liệu cá nhân, không realtime. Cần chốt định nghĩa pending_jobs có loại trừ tin đã hết hạn không trước code; không tạo bảng riêng.

Response thành công minh họa:

```text
{
  "data": {
    "total_students": 120,
    "total_employers": 35,
    "pending_employers": 8,
    "pending_jobs": 12,
    "recruiting_jobs": 46,
    "total_applications": 218
  }
}
```

| HTTP | error.code | Điều kiện |
| --- | --- | --- |
| 400 | INVALID_QUERY | Query lạ. |

### 4.42. API-42 — Hiển thị logo NTD — hỗ trợ tùy chọn (ĐỀ XUẤT)

GET /api/employers/{user_id}/logo | Quyền: Công khai có điều kiện / chủ / Admin | US02/37/46.

Cổng triển khai: BR45/47/51: loại ảnh, dung lượng, quyền/caching và storage; chưa bắt buộc có logo.

FR1/FR17. Path user_id; không query/body. 200 bytes ảnh đã kiểm tra, Content-Type xác minh; chỉ logo, không dùng chung đường dẫn với minh chứng. Đề xuất công khai logo NTD đủ điều kiện công khai; chủ hồ sơ ACTIVE/Admin có thể xem logo của hồ sơ chưa duyệt bằng Bearer. Không có logo/không đủ quyền công khai trả 404; cache theo chính sách file đã duyệt. Không trả logo_storage_key.

| HTTP | error.code | Điều kiện |
| --- | --- | --- |
| 404 | LOGO_NOT_FOUND (đề xuất) | Không có logo hoặc không đủ quyền công khai. |
| 500 | LOGO_UNAVAILABLE (đề xuất) | File không đọc được. |

## 5. Ma trận trạng thái và xử lý tự động

### 5.1. Năm nhóm trạng thái độc lập

| Đối tượng | Từ → Đến | Điều kiện / tác động |
| --- | --- | --- |
| Tin | Tạo → RECRUITING | NTD ACTIVE + APPROVED tạo hợp lệ, có 1–21 ca; moderation_status=PENDING. |
| Đơn | Tạo → PENDING | SV nộp CV + selected_slot_ids hợp lệ; riêng legacy không lịch cho [] |
| Đơn | PENDING → APPROVED | NTD duyệt khi còn ca tương lai; decided_at=thời điểm duyệt; notification nguyên tử. |
| Đơn | PENDING → REJECTED | NTD từ chối, decided_at=thời điểm quyết định; notification. |
| Đơn | PENDING → JOB_EXPIRED | Không còn bất kỳ lần làm có thời điểm BẮT ĐẦU tương lai thuộc các ca SV chọn; không cần chờ tin EXPIRED. |
| Tin | RECRUITING → FILLED (thủ công) | >=1 APPROVED và không còn PENDING. Chặn nếu tồn tại PENDING; không auto REJECTED. |
| Tin | RECRUITING → FILLED (tự động) | Sau giờ KẾT THÚC ca cuối toàn tin, >=1 APPROVED. |
| Tin | RECRUITING → EXPIRED (tự động) | Sau giờ KẾT THÚC ca cuối toàn tin, 0 APPROVED. |
| Tin | FILLED → COMPLETED | Chỉ NTD xác nhận thực sự làm xong, không tự động. |
| Tin | RECRUITING hoặc FILLED → CANCELLED | Chỉ NTD trước ngày ca đầu, mọi đơn → JOB_CANCELLED. |
| Đơn | Mọi trạng thái trừ JOB_CANCELLED → JOB_CANCELLED | Tin bị hủy hợp lệ, lưu status_before_cancel; giữ lịch sử quyết định/CV/ca. |
| Xác minh NTD | PENDING → APPROVED/REJECTED | Admin quyết định, actor/time/reason; không đổi role/tài khoản. |
| Kiểm duyệt tin | PENDING → APPROVED/REJECTED | Admin quyết định, không đổi vòng đời/đơn. |
| Tài khoản | ACTIVE ↔ LOCKED | Admin khóa/mở khóa SV/NTD; JWT cũ bị chặn khi LOCKED; BR47 còn mở. |

### 5.2. Định nghĩa các mốc thời gian (không nhầm ca đầu và ca cuối)

Khóa nhận đơn cho một ca ONE_TIME: từ khi start_at <= now; các ca khác còn tương lai vẫn nộp được. Đơn PENDING hết hạn: khi không còn một ca/lần WEEKLY nào có start_at>now. Tin ONE_TIME hết lịch: now >= max(end_at) của toàn bộ ca; tin WEEKLY có schedule_end_date: now >= thời điểm kết thúc lần ca cuối thực sự trong khoảng hiệu lực; WEEKLY không có end_date không tự EXPIRED/FILLED theo giờ. Hủy chủ động: local DATE(now) < local DATE(thời điểm bắt đầu ca SỚM NHẤT).

Sự kiện tự JOB_EXPIRED chỉ xảy ra từ PENDING. Đơn APPROVED/REJECTED giữ nguyên khi tin tự FILLED/EXPIRED; hủy chủ động trước ngày đầu là ngoại lệ ghi đè. Nếu scheduler chậm, API-13/API-18/API-19 vẫn phải tính thời gian DB và từ chối nộp/duyệt sai ngay lập tức.

### 5.3. Transaction, tuần tự khóa và job nền

Backend scheduler/worker hoặc job do server chạy định kỳ (không lệ thuộc FE) thực hiện expiration: khóa job_posts rồi applications theo thứ tự cố định, tính lại mốc hiện tại, cập nhật các PENDING cần JOB_EXPIRED, ghi notifications cùng transaction; sau giờ ca cuối tự chuyển RECRUITING sang FILLED/EXPIRED. Worker chạy lặp an toàn (idempotent), không tạo thông báo trùng; khóa hoặc điều kiện WHERE status=... ngăn hai worker/call HTTP xử lý đồng thời. Chuẩn đồng hồ so sánh lấy từ PostgreSQL server với timezone quy định.

API-19 hủy: cập nhật job_posts.status=CANCELLED, cancelled_at, cancel_reason; SELECT các applications thuộc tin với khóa, lưu status_before_cancel, update tất cả sang JOB_CANCELLED, insert notifications từng người; commit một lần; nếu thất bại bất kỳ INSERT nào phải rollback toàn bộ. Lưu CV/selected_slots/decided_at nguyên vẹn. API-18 duyệt và gửi thông báo cũng transaction tương tự.

Giao dịch v4: đăng ký lưu user+profile+file theo cơ chế bù trừ; xét duyệt/khóa kiểm tra actor và trạng thái hiện tại trong transaction. Khi thao tác liên quan nhiều bảng, quy định thứ tự khóa thống nhất (users/employer_profiles trước job_posts, sau đó applications), kiểm tra lại quyền/predicate sau khi khóa; rà soát cùng worker để tránh deadlock. API-08/11 kiểm danh mục/mã hoạt động và khóa đồng bộ với quản trị danh mục. Chính sách xử lý file cũ, gửi lại, khóa NTD có tin/đơn phải chốt trước code; không tự ghi thông báo không có application_id.

## 6. Truy vết 49 User Story → FR/UC/AC → API

| US | FR | Use Case (SRS v2.0) | AC (SRS v2.0) | Ưu tiên / API |
| --- | --- | --- | --- | --- |
| US01 | FR1 | UC01a Đăng ký | AC1.1, AC1.2, AC1.4 | MUST • API-01 |
| US02 | FR1, FR24 | UC01a Đăng ký | AC1.1, AC1.2, AC1.4, AC1.6, AC1.8 | MUST • API-01/27/28/29/32/33 |
| US03 | FR1 | UC01b Đăng nhập | AC1.3, AC1.5, AC1.7 | MUST • API-02/10; FE điều hướng |
| US04 | FR1 | UC01b Đăng nhập | AC1.3, AC1.5 | MUST • API-02 |
| US05 | FR1 | UC01b Đăng nhập | AC1.3–AC1.5 | MUST • API-02 |
| US06 | FR2, FR21, FR25 | UC02a Đăng tin; UC16 NTD lịch ca | AC2.1–AC2.6; AC21.1, AC21.5; AC25.1, AC25.2 | MUST • API-07/08/27/28/34/35/36 |
| US07 | FR9, FR21, FR25 | UC02b Sửa tin; UC16 NTD lịch ca | AC9.1–AC9.3; AC21.2–AC21.4; AC25.3 | MUST • API-07/10/11/12/27/28/36 |
| US08 | FR3 | UC03a Xem danh sách | AC3.3, AC3.11 | MUST • API-09 |
| US09 | FR3 | UC03b Xem chi tiết | AC3.4, AC3.5, AC3.8 | MUST • API-10 |
| US10 | FR3 | UC03c Tìm kiếm | AC3.2, AC3.6, AC3.9 | MUST • API-09; FE reset |
| US11 | FR3 | UC03d Lọc tin | AC3.1–AC3.3, AC3.7, AC3.10–AC3.12 | MUST • API-07/09/27/28 |
| US12 | FR4, FR1 | UC04 Ứng tuyển kèm CV và chọn ca | AC4.1–AC4.12; AC1.7 | MUST • API-02/10/13/17; FE quay lại |
| US13 | FR5 | UC05a Xem ứng viên/CV/ca | AC5.4, AC5.7 | MUST • API-15/16/17 |
| US14 | FR5 | UC05b Duyệt và duyệt muộn | AC5.1, AC5.3, AC5.5–AC5.7 | MUST • API-18 |
| US15 | FR5 | UC05c Từ chối đơn | AC5.2–AC5.3 | MUST • API-18 |
| US16 | FR6 | UC06a Xem kết quả đơn | AC6.5, AC6.6, AC6.8 | MUST • API-14/16 |
| US17 | FR6 | UC06b Xem trạng thái tin | AC6.6–AC6.7 | MUST • API-09/10/14 |
| US18 | FR6 | UC06c Đóng tuyển và tự kết thúc | AC6.1, AC6.2, AC6.7–AC6.11 | MUST • API-12/19 |
| US19 | FR6 | UC06d Xác nhận hoàn thành | AC6.3, AC6.4 | MUST • API-12/19 |
| US20 | FR7 | UC07a Sinh viên đánh giá | AC7.1–AC7.3 | MUST • API-20/21 |
| US21 | FR7 | UC07b NTD đánh giá | AC7.1–AC7.3 | MUST • API-20/21 |
| US22 | FR8 | UC08 Nhận thông báo | AC8.1–AC8.4 | MUST • API-18/22 |
| US23 | Hướng phát triển | Chưa đặc tả kỳ này | Không nghiệm thu kỳ này | WON’T • Không triển khai |
| US24 | Hướng phát triển | Chưa đặc tả kỳ này | Không nghiệm thu kỳ này | WON’T • Không triển khai |
| US25 | Hướng phát triển | Chưa đặc tả kỳ này | Không nghiệm thu kỳ này | WON’T • Không triển khai |
| US26 | Hướng phát triển | Chưa đặc tả kỳ này | Không nghiệm thu kỳ này | WON’T • Không triển khai |
| US27 | Hướng phát triển | Chưa đặc tả kỳ này | Không nghiệm thu kỳ này | WON’T • Không triển khai |
| US28 | Hướng phát triển | Chưa đặc tả kỳ này | Không nghiệm thu kỳ này | WON’T • Không triển khai |
| US29 | Hướng phát triển | Chưa đặc tả kỳ này | Không nghiệm thu kỳ này | WON’T • Không triển khai |
| US30 | Hướng phát triển | Chưa đặc tả kỳ này | Không nghiệm thu kỳ này | WON’T • Không triển khai |
| US31 | Hướng phát triển | Chưa đặc tả kỳ này | Không nghiệm thu kỳ này | WON’T • Không triển khai |
| US32 | Hướng phát triển | Chưa đặc tả kỳ này | Không nghiệm thu kỳ này | WON’T • Không triển khai |
| US33 | Hướng phát triển | Chưa đặc tả kỳ này | Không nghiệm thu kỳ này | WON’T • Không triển khai |
| US34 | Hướng phát triển | Chưa đặc tả kỳ này | Không nghiệm thu kỳ này | WON’T • Không triển khai |
| US35 | FR15 | UC12a SV xem hồ sơ | AC15.1, AC15.2 | MUST • API-03 |
| US36 | FR16 | UC12b SV cập nhật hồ sơ | AC16.1–AC16.3 | MUST • API-04 |
| US37 | FR17, FR24 | UC13a NTD xem hồ sơ | AC17.1, AC17.2, AC24.2, AC24.4 | MUST • API-05/32/42 |
| US38 | FR18, FR24 | UC13b NTD cập nhật hồ sơ | AC18.1–AC18.3; AC24.3 | MUST • API-06/27/28/29/32/33/42 |
| US39 | FR19 | UC14 Admin xem tài khoản | AC19.1–AC19.4 | MUST • API-23 |
| US40 | FR26 | UC19a Khóa tài khoản | AC26.1, AC26.3, AC26.4 | MUST • API-23/37; mọi API bảo vệ |
| US41 | FR26 | UC19b Mở khóa tài khoản | AC26.2, AC26.3 | MUST • API-23/37 |
| US42 | FR20 | UC15 SV quản lý lịch rảnh | AC20.1–AC20.5 | MUST • API-24/25 |
| US43 | FR21 | UC16 NTD quản lý lịch ca | AC21.1–AC21.5 | MUST • API-08/09/10/11/26 |
| US44 | FR22 | UC17 SV nhận gợi ý phù hợp | AC22.1–AC22.6 | MUST • API-26 |
| US45 | FR23 | UC18 NTD hủy tin | AC23.1–AC23.6 | MUST • API-19/22 |
| US46 | FR24 | UC20 Admin xác minh NTD | AC24.1–AC24.4 | MUST • API-30/31/32/33 |
| US47 | FR25 | UC21 Admin kiểm duyệt tin | AC25.1–AC25.4 | MUST • API-34/35/36 |
| US48 | FR27 | UC22 Admin quản lý danh mục | AC27.1–AC27.3 | MUST • API-07/38/39/40 |
| US49 | FR28 | UC23 Admin Dashboard | AC28.1–AC28.2 | SHOULD • API-41 (SHOULD) |

12 US WON’T (US23–US34) không tạo endpoint/schema hoặc nghiệm thu. API-27–42 trong bảng là đề xuất, không tự coi đã phê duyệt. Các AC về FE/điều hướng cần test browser; API không tự thực hiện redirect.

## 7. Bộ kịch bản nghiệm thu API (chưa chạy)

| SRS | API | Tình huống | Kết quả bắt buộc |
| --- | --- | --- | --- |
| M01 | API-24–26 | SV A rảnh T2 07:00–11:00, tin WEEKLY T2 08:00–10:00. | Gợi ý A nhận 1/1=100% khi tin RECRUITING, lịch hiệu lực. |
| M02 | API-08/24–26 | SV B rảnh CN 18:00–22:00; ONE_TIME ca CN 19:00–21:00 còn tương lai. | B có gợi ý, A/B lịch khác nhau và kết quả khác nhau. |
| M03 | API-26 | SV rảnh 07:00–09:00, ca 08:00–10:00. | Không match vì chỉ giao nhau một phần. |
| M04 | API-26 | Tin hai ca còn xét, SV khớp một; thử ANY/ALL. | ANY match 50%, ALL không hiển thị trong gợi ý nhưng SV vẫn chủ động nộp. |
| M05 | API-09/26 | Tin cũ không ca; tin FILLED; SV không lịch. | Tin legacy không ca chỉ công khai nếu đã duyệt và đủ quyền; không Matching; lịch rỗng nhận []. |
| M06 | API-08/11/25 | Lịch chồng CN→T2, sai giờ, 22 ca, ca đêm sai thời điểm. | 400 VALIDATION_ERROR, rollback toàn bộ; đêm hợp lệ là 1 slot. |
| M07 | API-11/24 | NTD B sửa lịch NTD A; STUDENT xem lịch SV B. | Sai quyền từ chối, không rò dữ liệu. |
| M08 | API-18/22 | NTD APPROVED đơn PENDING; giả lập INSERT notifications lỗi. | Rollback cả applications lẫn notifications. |
| M09 | API-08/13/18/19 | WEEKLY có ngày bắt đầu nhưng không ngày kết thúc; NTD chưa phản hồi một đơn. | Đơn giữ PENDING qua nhiều tuần; tin không tự EXPIRED. |
| M10 | API-13 | ONE_TIME ba ca ngày 15–17/11; ngày 15 đã bắt đầu. | SV nộp đơn chọn ca 16/17 thành công; không chọn ca 15. |
| M11 | API-18/16 | SV chọn ca 15 và 17/11; NTD duyệt ngày 16/11. | Popup cảnh báo; APPROVED và decided_at ngày 16; ca 15 xám, chỉ ca 17 có hiệu lực. |
| M12 | API-13/18/22 + worker | SV chỉ chọn ca 15/11; đến giờ ca 15 bắt đầu đơn còn PENDING. | Đơn tự JOB_EXPIRED + thông báo, dù tin còn RECRUITING cho ca 16–17. |
| M13 | API-19 nội bộ | Tin ONE_TIME đã qua giờ ca cuối: thử có/không có APPROVED. | Tự FILLED nếu ≥1 APPROVED, ngược lại EXPIRED; không tự COMPLETED. |
| M14 | API-19 nội bộ | WEEKLY có ngày kết thúc, ca cuối qua đêm CN 22:00→T2 02:00. | Tự chuyển trạng thái sau 02:00 ngày T2, không vào 22:00 CN. |
| M15 | API-19 | NTD còn 1 PENDING, 2 APPROVED; bấm Đóng tuyển. | Không chuyển FILLED, bắt NTD xử lý hết PENDING; không tự REJECTED. |
| M16 | API-19/22 | NTD hủy tin trước ngày ca đầu, có đơn PENDING/APPROVED/REJECTED. | Tin CANCELLED, cả 3 đơn JOB_CANCELLED, lưu trạng thái cũ, 3 thông báo kèm lý do. |
| M17 | API-19 | NTD hủy tin lúc 00:00 của ngày ca đầu (ca bắt đầu 18:00). | Bị chặn dù ca chưa bắt đầu; không đổi tin/đơn. |
| M18 | API-24–26 | SV rảnh CN 21:00→T2 03:00; tin CN 22:00→T2 02:00. | Một ca đêm khớp 100%; không chia thành 2 ca. |
| M19 | API-11 | NTD sửa lịch khi tin đã có một đơn từng REJECTED. | Khóa sửa ca/ngày/kiểu lịch; cho phép sửa nội dung khác nếu hợp lệ. |
| M20 | API-19/22 | NTD hủy tin, giả lập lỗi khi INSERT một notifications. | ROLLBACK cả job_posts, applications và notifications; chạy lại an toàn. |
| M21 | API-20/21 | Tin COMPLETED, đơn APPROVED; SV đánh giá NTD và ngược lại. | Đánh giá đúng cặp tối đa 1 mỗi chiều; tin CANCELLED/EXPIRED không cho đánh giá. |
| M22 | 01/08/09/30–36 | Đăng ký NTD, duyệt NTD, tạo tin, duyệt tin | PENDING không đăng/không công khai; chỉ sau 2 bước duyệt mới công khai đủ điều kiện. |
| M23 | 02/09/10/13/27/28 + FE | Lọc A toàn tỉnh và B một phường, xóa keyword, guest ứng tuyển rồi login | OR địa bàn, AND lọc khác; page 1; quay đúng tin, không tự nộp, kiểm tra lại điều kiện. |
| M24 | 02/23/37 + API bảo vệ | Khóa SV/NTD có token cũ, mở khóa | LOCKED bị chặn login/API; mở khóa đúng role/xác minh; tác động tin/đơn chờ BR47. |
| M25 | 07/08/11/38–40 | Ngừng ngành đang có tin | Không xóa FK; không chọn mới; lịch sử đọc được. |
| M26 | 41 | Dashboard DB fixture và sai role | Sáu số đúng DB; sai role 403; SHOULD. |
| V4-01 | 01/06/29/32/33 | Thiếu file, sai địa bàn/industry, file giả/lớn, truy cập key trực tiếp | 400/413/415/403 theo hợp đồng đã duyệt, rollback/dọn file; không lộ key. |
| V4-02 | 09/10/13/26/36 | Tin PENDING/REJECTED/APPROVED, thử truy cập ID trực tiếp | Chặn tin chưa duyệt khỏi public/Matching/nộp; owner/Admin và SV có đơn đọc đúng quyền. |
| V4-03 | 11/36 | Sửa tin đã duyệt, gửi lại; race duyệt/sửa/worker | Không đổi ca đã khóa; chính sách BR46 đã duyệt; không mất lịch sử. |
| V4-04 | 27/28/09 | Mã có số 0 đầu, phường sai tỉnh, locations trùng, legacy thiếu mã | Giữ mã chuỗi; 400 query sai; không đoán mã; không trả nhầm. |
| V4-05 | 33/36/37/39/40 | Sai role, quyết định lặp, đồng thời | 403/409 đúng; actor/time từ server; không ghi một phần. |
| V4-06 | 05/06/31/32/42 | Chủ hồ sơ/Admin và người ngoài truy cập file | Chỉ đúng quyền xem minh chứng, logo theo chính sách riêng; không public key. |
| V4-07 | 01/06/08/11/37 + migration | DB legacy, thiếu mã/minh chứng, tài khoản khóa | Không tự duyệt/backfill sai/xóa lịch sử; chính sách chuyển tiếp được duyệt trước chạy. |

Kiểm thử bổ sung API-13: selected_slot_ids thiếu, JSON sai, chứa ca của tin khác, trùng ID, ca ONE_TIME bắt đầu trong lúc form đang mở, WEEKLY hết hiệu lực; CV không phải PDF, PDF >5 MiB; quyền EMPLOYER gọi; nộp lại cùng tin. API-09: lọc date_from/to, ca đêm theo giờ địa phương và tin legacy không ca. API-18: xác nhận duyệt muộn thiếu cờ và race với JOB_EXPIRED; quyết định không được tạo notification giả.

NFR và kiểm thử nền từ v2.2 vẫn phải giữ: đăng ký/đăng nhập đủ ba vai trò; phân quyền 401/403; hồ sơ không được sửa role/email; đánh giá một lần mỗi bên; giới hạn phân trang; tối thiểu 100 tin mô phỏng/5 người dùng đồng thời cho đo hiệu năng; Chrome/Edge từ 360px. Các yêu cầu này không được xem là PASS chỉ vì có đặc tả.

Tất cả ca M01–M21 nghiệp vụ cũ dùng fixture NTD/tin đã duyệt, đủ điều kiện; ngày/giờ tương đối theo thời điểm test. M22–26 và V4-* là kế hoạch kiểm thử, không phải PASS. Các ca phụ thuộc BR45/46/47/48/50/51 chỉ nghiệm thu sau khi quyết định được xác nhận.

## 8. Đối chiếu với Database Schema (thiết kế mục tiêu)

| Bảng | API | Cột/quan hệ cần có hoặc giữ |
| --- | --- | --- |
| users | 01/02/23/37 + API bảo vệ | Giữ email/role/hash; account_status, locked_at/by/reason, unlocked_at/by; seed ADMIN. |
| student_profiles | 01/03/04 | Một hồ sơ trên một STUDENT. |
| employer_profiles | 01/05/06/08/29–33/42 | Giữ PK user_id; business_type,tax_code,industry_id,province_code,ward_code,address_detail,logo_storage_key,verification_doc_key,website_url,verification_status,reject_reason,verified_at/by. |
| job_categories | 07/08/09/11/38–40 | category_id,name,is_active; giữ FK lịch sử. |
| job_posts | 08–12/19/26/34–36 | Giữ lịch, hủy, 5 status, city/location_detail; thêm province_code,ward_code,moderation_status,moderation_reason,moderated_at/by; không required_workers. |
| job_time_slots | 08/09/10/11/13/26 | Ca WEEKLY: day_of_week/start_time/end_time; ONE_TIME: start_at/end_at TIMESTAMPTZ; ca đêm 1 record; job_slot_id và FK job_id. |
| student_availability_slots | 24/25/26 | day_of_week/start_time/end_time; 0–21 slots, qua đêm logic 1 record. |
| applications | 13–18/19/20 | 5 trạng thái; decided_at; status_before_cancel; unique(student_id,job_id); CV PDF metadata. |
| application_selected_slots | 13–16/18/26 | Bảng liên kết (application_id,job_slot_id) unique; giữ lựa chọn khi đơn hết hạn/hủy. |
| notifications | 18/19/22 + worker | recipient_id/application_id/message/created_at; insert cùng transaction trạng thái. |
| reviews | 20/21 | UNIQUE(application_id,reviewer_id); chỉ APPROVED + COMPLETED. |
| administrative_provinces | 27/28/01/06/08/09/11/26 | province_code,name,source_version,is_active; mã chuỗi. |
| administrative_wards | 28/01/06/08/09/11/26 | ward_code,province_code,name,source_version,is_active; FK ghép đúng tỉnh. |
| business_industries (điều kiện) | 29/01/06 | industry_id,name,is_active; BR48 chờ chọn riêng/chung. |
| Không bảng riêng | 41 | COUNT từ users, employer_profiles, job_posts, applications. |

13 bảng nghiệp vụ mục tiêu (11 nền + 2 địa bàn); 14 nếu duyệt business_industries riêng; schema_migrations không tính. Tên/kiểu cột mới theo Schema v4.0 là đề xuất. Không sửa migration đã áp dụng, không tự coi cột đã tồn tại. TIMESTAMPTZ cho ca ONE_TIME; DTO URL không cần cột DB.

## 9. Ràng buộc tương thích và danh sách cập nhật liên tài liệu

SRS v2.0 phạm vi v4.0 là chuẩn yêu cầu; Backlog v4.0 là chuẩn US/ưu tiên. API-01–26 giữ method/URL; API-27–42 và DTO/lỗi mới cần xác nhận. Database Schema v4.0 là thiết kế mục tiêu; ERD, UC, Sprint, kiến trúc, bối cảnh và AGENTS cần cập nhật tiếp. Không tự đổi công nghệ hoặc mã cũ.

Legacy không lịch: API-10 trả time_slots:[] nếu đủ quyền; API-13 có thể nộp CV không chọn ca chỉ khi đạt điều kiện FR4/công khai; không Matching, không đoán hạn hủy từ work_time. Legacy thiếu mã địa bàn không gán đoán; không tự APPROVED NTD/tin chưa kiểm tra. Chính sách chuyển tiếp phải chốt BR50 trước migration.

Không tính lương/chấm công; NTD không xác nhận COMPLETED thì chưa mở review; WEEKLY vô hạn có thể PENDING lâu. Các điều kiện mở sau đây là cổng phê duyệt, không phải quyền để Codex tự quyết:

| Cổng | Đề xuất cần xác nhận trước code |
| --- | --- |
| BR43/50 | Nguồn mã, FK và giới hạn locations; API-27/28; city do server sinh; hành vi status/scope tương thích, dữ liệu legacy. |
| BR45/51 | Tax COMPANY bắt buộc hay không; loại file/dung lượng/retention, logo, gửi lại, sửa hồ sơ đã APPROVED; chính sách không bắt buộc CCCD; API-01/05/06/32/33/42. |
| BR48 | Danh mục riêng hay chung; API-29 và scope API-07; quy tắc trùng tên/kích hoạt lại API-39/40. |
| BR46 | Tin REJECTED gửi lại, sửa tin APPROVED trước/sau đơn, quy tắc public; API-11/36. |
| BR47 | Khóa NTD: xử lý tin/đơn đang chờ/đang làm, hiển thị, token sau mở khóa; API-37 và predicate dùng chung. |
| BR50 | Phê duyệt toàn bộ mã API-27–42, request/response/mã lỗi mới, migration và chính sách legacy. Dashboard pending_jobs, file logo tùy chọn. |

## 10. Checklist bàn giao và xác minh

| Đầu việc | Kiểm chứng / bằng chứng cần có |
| --- | --- |
| Tính toàn vẹn mã | US01–49: 36 MUST, 1 SHOULD, 12 WON’T; 26 API giữ mã + 16 đề xuất; truy vết đúng SRS. |
| Không đổi hợp đồng cũ vô cớ | API-01–API-26 giữ HTTP method/URL; lỗi, JWT, 5MiB, salary_unit theo v2.2. |
| Nội dung mới | Giữ lịch/Matching/hủy; địa bàn, xác minh NTD, duyệt tin, khóa tài khoản, danh mục, Dashboard, quay lại tin. |
| Database & migration | Đối chiếu Schema v4.0 và DB/migration thực tế; chốt các cổng; không tự coi tồn tại. |
| Kiểm thử sau code | Chạy API/BE/PostgreSQL thực tế, lưu kết quả Postman/node:test/SQL và báo PASS/FAIL theo bằng chứng. |
| Git | Chỉ xác nhận commit/push/merge sau khi kiểm tra terminal hoặc GitHub; mô tả commit bằng tiếng Việt. |
| Bàn giao Dxx | Sau test đạt, xác minh checkpoint, tạo FullBackup và ChangedFiles theo AGENTS.md; manifest file mới/sửa/xóa, ghi chức năng, SHA và ngày. Không chứa secret/PII/file CV/minh chứng thật. |

API Contract v4.0 — 24/11/2026. Thiết kế tích hợp theo SRS v2.0 / Phạm vi v4.0. Chưa sửa code, chưa chạy migration, chưa kiểm thử runtime hoặc xác minh Git.
