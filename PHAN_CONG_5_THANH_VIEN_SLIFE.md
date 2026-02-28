# Kế hoạch chia việc cho team 5 người (SLife)

> Team gồm: **3 Fullstack + 1 BE Leader + 1 FE Leader**.
> 
> Mục tiêu: chia việc rõ package/module, API liên quan, chức năng phải làm, kèm hướng dẫn phối hợp để leader có thể quản lý tiến độ tốt.

---

## 1) Nguyên tắc chia việc

- **FE Leader** và **BE Leader** làm phần “dùng chung” (core/shared) để định chuẩn kiến trúc + code review + hỗ trợ 3 bạn fullstack.
- **3 Fullstack** mỗi người phụ trách 1 cụm nghiệp vụ (listing, deal/chat, admin/report), đảm bảo làm từ API -> UI -> test tích hợp.
- Mỗi task đều có:
  - Package/module chính cần làm.
  - Chức năng chi tiết.
  - API phải dùng/triển khai.
  - Definition of Done (DoD).

---

## 2) Chuẩn package/module đề xuất

## 2.1 Backend (Spring Boot)

Đề xuất cấu trúc package (khi triển khai thêm code Java):

- `com.slife.common` (exception, response wrapper, constants)
- `com.slife.config` (security, cors, websocket, openapi)
- `com.slife.auth` (login/register/jwt)
- `com.slife.listing` (listing, category, upload image)
- `com.slife.deal` (offer/deal/negotiation)
- `com.slife.chat` (conversation/message)
- `com.slife.notification` (notification + realtime)
- `com.slife.admin` (user/report moderation)

## 2.2 Frontend (React)

Theo skeleton hiện có:

- API layer: `frontend/src/api/*`
- Reusable layout: `frontend/src/components/layout/*`
- Common component: `frontend/src/components/common/*`
- Listing: `frontend/src/pages/listing/*`, `frontend/src/components/listing/*`, `frontend/src/hooks/useListings.js`
- Deal/Profile/Admin pages: `frontend/src/pages/deal/*`, `frontend/src/pages/profile/*`, `frontend/src/pages/admin/*`
- Routing/Auth: `frontend/src/routes/*`, `frontend/src/context/AuthContext.jsx`

---

## 3) Phân công chi tiết theo từng người

## 3.1 Thành viên A – **FE Leader** (chuyên FE)

### Vai trò chính
- Định nghĩa UI framework, shared layout, design guideline.
- Review toàn bộ PR FE của team.
- Hỗ trợ fullstack thống nhất style, routing, auth guard.

### Package/module phụ trách
- `frontend/src/components/layout/` (Header, Sidebar, MainLayout, Footer)
- `frontend/src/components/common/` (Loading, ConfirmDialog, Pagination, Tag)
- `frontend/src/routes/` (AppRouter, ProtectedRoute)
- `frontend/src/styles/`, `frontend/src/theme/`

### Chức năng phải làm
1. Hoàn thiện **khung giao diện dùng chung**:
   - Header: logo, search box mini, user menu.
   - Sidebar: menu theo role (user/admin).
   - MainLayout responsive desktop/mobile.
2. Chuẩn hóa UX chung:
   - Loading state, empty state, error state.
   - Toast/confirm dialog dùng chung.
3. Chuẩn routing + bảo vệ route:
   - Route public/private.
   - Redirect khi hết token.

### API liên quan (chủ yếu tích hợp ở tầng giao diện chung)
- `POST /api/auth/login`
- `GET /api/users/me` (đề xuất bổ sung nếu chưa có)
- `GET /api/notifications` (badge header)

### DoD
- Team có thể tái sử dụng layout/common component cho tất cả page.
- Rule UI thống nhất, không mỗi page một kiểu.
- 100% route private đi qua `ProtectedRoute`.

---

## 3.2 Thành viên B – **BE Leader** (chuyên BE)

### Vai trò chính
- Thiết kế chuẩn backend architecture + DB migration + API contract.
- Review toàn bộ PR backend.
- Quản lý chất lượng API (validation, error format, security).

### Package/module phụ trách
- `com.slife.common`
- `com.slife.config`
- `com.slife.auth`
- `com.slife.notification` (core realtime)
- OpenAPI spec: `backend/src/main/resources/openapi.yaml`

### Chức năng phải làm
1. Xây dựng **nền tảng chung backend**:
   - Global exception handler.
   - Base response format (`code`, `message`, `data`).
   - Validation chuẩn (Bean Validation).
2. Chuẩn hóa bảo mật:
   - JWT filter.
   - Role-based authorization.
3. Duy trì API contract:
   - Đồng bộ `openapi.yaml` theo feature team fullstack.
   - Versioning API nếu breaking change.

### API bắt buộc làm/chuẩn hóa
- `POST /api/auth/login`
- `GET /api/listings`
- `POST /api/listings`
- `GET /api/search`
- (đề xuất mở rộng) `/api/auth/register`, `/api/users/me`, `/api/notifications`, `/api/reports`

### DoD
- Fullstack chỉ cần bám contract là làm được UI.
- API trả lỗi đồng nhất cho mọi module.
- Có checklist security trước merge.

---

## 3.3 Thành viên C – **Fullstack 1 (Listing + Category)**

### Package/module phụ trách
**BE**
- `com.slife.listing`
- `com.slife.category`

**FE**
- `frontend/src/pages/listing/ListingsPage.jsx`
- `frontend/src/pages/listing/ListingDetailPage.jsx`
- `frontend/src/pages/listing/CreateListingPage.jsx`
- `frontend/src/components/listing/*`
- `frontend/src/api/listingApi.js`, `frontend/src/api/categoryApi.js`

### Chức năng phải làm
1. Listing feed + phân trang + filter cơ bản.
2. Tạo bài đăng mới (text + ảnh).
3. Chi tiết bài đăng + thông tin người bán.
4. Category tree và chọn category khi tạo bài.

### API liên quan
- `GET /api/listings`
- `POST /api/listings`
- `GET /api/listings/{id}`
- `POST /api/listings/{id}/images`
- `GET /api/categories`
- `GET /api/search?keyword=&categoryId=`

### DoD
- User tạo được listing có ảnh và thấy ngay trong feed.
- Filter/search hoạt động đúng với backend.

---

## 3.4 Thành viên D – **Fullstack 2 (Deal + Offer + Chat cơ bản)**

### Package/module phụ trách
**BE**
- `com.slife.deal`
- `com.slife.chat`

**FE**
- `frontend/src/pages/deal/DealDetailPage.jsx`
- `frontend/src/api/dealApi.js`
- `frontend/src/api/offerApi.js`
- (nếu bổ sung) module chat UI

### Chức năng phải làm
1. Đề xuất giá (offer) giữa 2 user.
2. Chốt deal (pending -> confirmed).
3. Khung chat cơ bản trong context deal/listing.
4. Lưu lịch sử đàm phán.

### API liên quan
- `POST /api/listings/{id}/offers`
- `GET /api/conversations/{conversationId}/messages`
- `POST /api/conversations/{conversationId}/messages`
- `POST /api/deals/{dealId}/confirm`
- `GET /api/deals/{dealId}`

### DoD
- Có luồng end-to-end: gửi offer -> xác nhận -> hiển thị trạng thái deal.
- Lịch sử message/offer xem lại được.

---

## 3.5 Thành viên E – **Fullstack 3 (Admin + Report + Profile/Notification)**

### Package/module phụ trách
**BE**
- `com.slife.admin`
- `com.slife.report`
- `com.slife.user`

**FE**
- `frontend/src/pages/admin/DashboardPage.jsx`
- `frontend/src/pages/admin/UserManagementPage.jsx`
- `frontend/src/pages/admin/ReportManagementPage.jsx`
- `frontend/src/pages/profile/ProfilePage.jsx`
- `frontend/src/api/reportApi.js`, `frontend/src/api/userApi.js`, `frontend/src/api/notificationApi.js`

### Chức năng phải làm
1. Admin xem danh sách report, xử lý report.
2. Admin quản lý user (lock/unlock, warning).
3. User profile cơ bản (xem/sửa thông tin).
4. Notification list + trạng thái đã đọc.

### API liên quan
- `GET /api/admin/reports`
- `PATCH /api/admin/reports/{id}`
- `GET /api/admin/users`
- `PATCH /api/admin/users/{id}/status`
- `GET /api/users/me`
- `PATCH /api/users/me`
- `GET /api/notifications`
- `PATCH /api/notifications/{id}/read`

### DoD
- Admin xử lý report end-to-end được.
- User chỉnh sửa profile và nhận thông báo.

---

## 4) Hướng dẫn phối hợp & quản lý cho 2 Leader

## 4.1 Cách quản lý backlog
- Chia backlog theo epic:
  1. Core (leader)
  2. Listing
  3. Deal/Chat
  4. Admin/Profile
- Mỗi task phải có:
  - API contract link.
  - UI acceptance criteria.
  - Test case chính.

## 4.2 Quy trình làm việc đề xuất
1. **BE Leader** chốt contract trước (OpenAPI).
2. **FE Leader** chốt UI skeleton + component guideline.
3. 3 Fullstack triển khai song song theo domain.
4. Demo tích hợp 2 ngày/lần.
5. Leader review theo checklist rồi mới merge.

## 4.3 Checklist review nhanh

### FE Leader checklist
- Có dùng component/layout chung chưa?
- Có loading/error/empty state chưa?
- Có responsive cơ bản chưa?

### BE Leader checklist
- Endpoint có đúng chuẩn HTTP status chưa?
- Validation + error response đồng nhất chưa?
- Có ảnh hưởng security/role không?

---

## 5) Mốc tiến độ gợi ý (2 tuần)

- **Ngày 1-2:** 2 leader dựng nền tảng core (layout + auth + exception + contract).
- **Ngày 3-7:** 3 fullstack triển khai domain chính.
- **Ngày 8-10:** Tích hợp chéo FE-BE, sửa bug.
- **Ngày 11-12:** Hardening + test + demo nội bộ.
- **Ngày 13-14:** Buffer + chuẩn bị release.

---

## 6) Gợi ý API tối thiểu để team chạy được MVP

1. Auth: login/register/me
2. Listing: list/create/detail/upload image/search
3. Deal: create offer/confirm deal/get deal detail
4. Chat: get/send message
5. Admin: list reports/update report/list users/update user status
6. Notification: list/read

> Nếu thiếu API ở skeleton hiện tại, ưu tiên BE Leader thêm vào OpenAPI trước khi code để tránh lệch FE-BE.
