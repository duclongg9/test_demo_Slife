# BE Document

Member(Superviser): @Do Thanh An (K18 HL)

# Đề xuất chỉnh sửa bản báo cáo của An

Các bảng dưới đảm bảo lưu trữ đầy đủ luồng chat và kết quả thỏa thuận giá của người dùng, nhưng hệ thống không xử lý đơn hàng mua bán chính thức mà chỉ như lưu trữ thông tin trò chuyện và thỏa thuận cuối.

## Tương tác & Giao dịch (Interaction & Transaction)

Hệ thống cho phép hai người dùng trò chuyện trực tiếp qua bài đăng (không phân biệt rõ vai trò mua/bán). Mỗi cuộc hội thoại liên kết với một bài đăng cụ thể và lưu trữ thông tin về hai người tham gia cùng thời điểm tin nhắn cuối. Bảng **Conversations** nên chứa ID của hai người tham gia (dù không phân biệt vai trò) và ID bài đăng liên quan. Dữ liệu chi tiết được thiết kế như sau:

- **conversation_id (PK):** INT – khóa chính.
- **user_id1 (FK):** INT – ID của người tham gia thứ nhất (Users.user_id).
- **user_id2 (FK):** INT – ID của người tham gia thứ hai (Users.user_id).
- **listing_id (FK):** INT – liên kết đến Listing.listing_id của bài đăng.
- **last_message_at:** DATETIME – thời gian tin nhắn mới nhất được gửi trong cuộc hội thoại.

## Tin nhắn (Messages)

Bảng **Messages** lưu trữ từng tin nhắn trong các cuộc hội thoại. Mỗi bản ghi ghi nhận người gửi, nội dung, thời gian và trạng thái đã đọc của tin nhắn. Cấu trúc các trường như sau:

- **message_id (PK):** INT – khóa chính.
- **conversation_id (FK):** INT – liên kết đến Conversations (cuộc hội thoại tương ứng).
- **sender_id (FK):** INT – ID người gửi tin nhắn (Users.user_id).
- **content:** TEXT – nội dung tin nhắn.
- **sent_at:** DATETIME – thời gian gửi tin nhắn.
- **is_read:** BOOLEAN, Default FALSE – trạng thái đã đọc (FALSE nếu chưa xem).

## Thỏa thuận giá (Deals)

Bảng **Deals** (còn gọi là Thỏa thuận giá) ghi nhận mức giá cuối cùng mà hai bên thống nhất, nhưng không coi đây là đơn hàng chính thức. Sau khi người mua đề nghị giá và người bán xác nhận, hệ thống lưu lại giá chốt để gửi thông tin cho hai bên. Ví dụ, bảng này có thể bao gồm:

- **deal_id (PK):** INT – khóa chính.
- **conversation_id (FK):** INT – liên kết đến Conversations (cuộc hội thoại liên quan).
- **listing_id (FK):** INT – liên kết đến Listings (bài đăng đang thỏa thuận).
- **proposed_by_id (FK):** INT – ID người vừa đề nghị giá (Users.user_id).
- **deal_price:** DECIMAL – mức giá cuối cùng được hai bên đồng ý.
- **status:** ENUM('PENDING','CONFIRMED') – trạng thái của thỏa thuận (chờ xác nhận hoặc đã xác nhận).
- **pickup_time:** DATETIME - Thời gian hẹn lấy hàng → gửi thông báo cho người dùng qua mail trước 30p lấy hàng.
- **created_at:** DATETIME – thời gian tạo bản ghi thỏa thuận.
- `address_id` | BIGINT AUTO_INCREMENT | **PK →** `location_name` | VARCHAR(200) | NOT NULL để lấy thông tin nhận hàng cơ bản → update khi người mua muốn thay đổi địa chỉ

## Đánh giá (Reviews)

Hệ thống hỗ trợ đánh giá giữa hai người dùng sau khi chốt xong giao dịch. Bảng **Reviews** lưu ý kiến phản hồi của một bên (reviewer) đối với bên còn lại (reviewee) dựa trên cuộc trao đổi hoặc thỏa thuận vừa kết thúc. Mỗi đánh giá gồm số sao (thang 1–5) và bình luận. Việc sử dụng thang đánh giá 1–5 sao là phổ biến trong thương mại điện tử. Cấu trúc bảng có thể bao gồm:

- **review_id (PK):** INT – khóa chính.
- **conversation_id (FK):** INT – liên kết đến Conversations (để xác định giao dịch liên quan).
- **reviewer_id (FK):** INT – ID người đánh giá (Users.user_id).
- **reviewee_id (FK):** INT – ID người được đánh giá (Users.user_id).
- **rating:** INT – điểm số (1-5).
- **comment:** TEXT – nội dung đánh giá.

## Đăng tin sản phẩm và quy trình vận hành

Người dùng có thể tạo **tin đăng sản phẩm** với các thông tin cơ bản: tiêu đề, mô tả, **hình ảnh**, và danh mục sản phẩm (có thể chọn đa tầng, con dưới cha). Mục đích đăng (ví dụ: bán, cho tặng, “tin nhanh”,…) do **cấu hình hệ thống** quản lý, và nếu chọn cho tặng (giveaway) thì hệ thống sẽ tự động đặt giá bằng 0. Mỗi tin đăng có trạng thái như DRAFT, ACTIVE, HIDDEN, SOLD, v.v. (theo yêu cầu nghiệp vụ) và đánh dấu mức độ mới/cũ. Khi người bán đánh dấu đã **bán/ẩn tin**, tin đó sẽ không hiển thị trong luồng (feed) nữa. Hệ thống cũng cần xử lý tình huống tin hết hạn: có thể lưu trữ thêm trường `expiration_date` và một công việc định kỳ (cron job hoặc MySQL EVENT) sẽ ẩn hoặc xoá các tin đã quá hạn. Việc **báo cáo** (flag) từ người dùng sai phạm nhiều lần hoặc nội dung cấm (ví dụ chứa từ khoá cấm như ma túy, súng,…) sẽ được xử lý theo quy định: người dùng vi phạm có thể bị hạn chế đăng (ví dụ 1 bài/ngày) hoặc bị cấm. Việc quét nội dung tin đăng cho từ khoá cấm là phương pháp kiểm duyệt tự động cơ bản.

- *Quyền đăng tin theo đánh giá:* Người dùng có điểm uy tín (reputation) tính trung bình từ các đánh giá (rating) mà họ nhận được. Điểm trung bình này được cập nhật khi có đánh giá mới để gán vào trường `reputation_score` của người dùng.
- *Thông báo (Notifications):* Khi có sự kiện quan trọng (tin của người theo dõi mới được đăng, tin hết hạn, tin bị báo cáo bởi admin, hay thông báo hệ thống như sự kiện đặc biệt), một bản ghi thông báo sẽ được thêm vào bảng **Notifications**. Bảng này lưu `notification_id`, `user_id` (người nhận), `type` (ví dụ: MESSAGE, ORDER_UPDATE, SYSTEM_ALERT), `content` (nội dung thông báo) và `is_read` (đã đọc hay chưa). Khi người dùng bấm vào thông báo, hệ thống sẽ điều hướng tới trang hiển thị chi tiết thông báo hoặc nội dung liên quan (ví dụ tin đăng). Thiết kế này tương tự như bảng thông báo trên các mạng xã hội, nơi mỗi người dùng có thể nhận nhiều thông báo khác nhau.
- *Chức năng Follow (Theo dõi):* Mỗi người dùng có thể theo dõi người bán khác mà họ quan tâm. Thiết kế đơn giản là một bảng **Follows** (hoặc `Connections`) dạng many-to-many giữa người theo dõi và người được theo dõi, với khóa chính gồm `follower_id` và `followed_id` (cùng tham chiếu Users.user_id). Khi một người bị theo dõi đăng tin mới (trạng thái ACTIVE), hệ thống tạo thông báo cho những người theo dõi đó, kèm tiêu đề và link tới tin. Trên trang cá nhân của mỗi người dùng cũng hiển thị số lượng người theo dõi và danh sách người theo dõi họ.
- *Chặn người dùng:* Bảng **Blocks** có thiết kế tương tự Follows, gồm `blocker_id` và `blocked_id`. Khi người A chặn B, thì A sẽ không nhận tin nhắn hay tin đăng từ B, đồng thời B cũng không thể thấy tin đăng của A.

## Mô hình dữ liệu Sản phẩm và Tin đăng

Hệ thống lưu trữ cấu trúc dữ liệu sản phẩm và tin đăng như sau:

- **Categories (Danh mục sản phẩm):** Lưu các loại sản phẩm. Ví dụ `category_id` (PK), `name`, `description`. Nếu cần hỗ trợ cấu trúc phân cấp nhiều tầng, ta có thể bổ sung cột `parent_id` (FK tham chiếu Categories.category_id) để tạo quan hệ cha – con giữa các danh mục.
    
    ## `Categories` — Danh mục sản phẩm (đa tầng)
    
    - `category_id` | INT AUTO_INCREMENT | **PK** | NOT NULL
    - `name` | VARCHAR(200) | NOT NULL (unique tại cùng cấp tuỳ chính sách)
    - `description` | TEXT | NULLABLE
    - `parent_id` | INT | FK → Categories.category_id | NULLABLE
    - `level` | INT | NOT NULL DEFAULT 0
    - `is_active` | TINYINT(1) | NOT NULL DEFAULT 1
    - `created_at`, `updated_at` | DATETIME
    
    **Ghi chú:** self-FK `parent_id` tạo cấu trúc cha-con.
    
- **Listings (Tin đăng):** Mỗi dòng là một tin người dùng đăng. Cột chính gồm: `listing_id` (PK), `seller_id` (FK liên kết Users.user_id), `category_id` (FK liên kết Categories), `title` (tên tin, bắt buộc), `description` (mô tả), `price` (DECIMAL, giá = 0 nếu cho tặng), `condition` (điều kiện: NEW, USED_LIKE_NEW, v.v.), `status` (enum gồm DRAFT, ACTIVE, HIDDEN, SOLD, GIVEN_AWAY, BANNED,…), `is_giveaway` (boolean), `created_at`, `updated_at`. Bảng Listings cũng sẽ tuân thủ quy tắc kinh doanh: ví dụ nếu mục đích là **Giveaway**, thì `price` tự động bằng 0 và `is_giveaway`=TRUE. Trường `status` cho phép ẩn tin khi bán hoặc bị vi phạm.
    
    ## `Listings` — Tin đăng
    
    - `listing_id` | BIGINT AUTO_INCREMENT | **PK** | NOT NULL
    - `seller_id` | BIGINT | FK → Users.user_id | NOT NULL
    - `category_id` | INT | FK → Categories.category_id | NULLABLE
    - `title` | VARCHAR(255) | NOT NULL
    - `description` | TEXT | NULLABLE
    - `price` | DECIMAL(10,2) | NOT NULL DEFAULT 0.00
    - `purpose_id` | INT | FK → Posting_Purposes.purpose_id | NULLABLE
    - `is_giveaway` | TINYINT(1) | NOT NULL DEFAULT 0
    - `condition` | ENUM('NEW','USED_LIKE_NEW','USED_GOOD','USED_FAIR') | NULLABLE
    - `status` | ENUM('DRAFT','ACTIVE','HIDDEN','SOLD','GIVEN_AWAY','BANNED','EXPIRED') | NOT NULL DEFAULT 'DRAFT'
    - `expiration_date` | DATETIME | NULLABLE
    - `is_flagged` | TINYINT(1) | NOT NULL DEFAULT 0
    - `view_count` | INT UNSIGNED | NOT NULL DEFAULT 0
    - `display_priority` | INT | NOT NULL DEFAULT 0
    - `created_at`, `updated_at` | DATETIME
    
    **Ghi chú nghiệp vụ:**
    
    - Khi `purpose.default_price_zero = 1` **hoặc** `is_giveaway=1` → `price` phải = `0.00`. Thực hiện kiểm tra ở application; DB trigger có thể làm phòng vệ.
    - Khi seller bấm “Đã bán” → `status='SOLD'` (không hiển thị feed).
    - `expiration_date` được set khi tạo theo `Configurations.listing_expiration_days` nếu admin cấu hình.
    
    ## `Posting_Purposes` — Mục đích đăng (configurable)
    
    - `purpose_id` | INT AUTO_INCREMENT | **PK**
    - `code` | VARCHAR(50) | NOT NULL UNIQUE (e.g. SELL, GIVEAWAY, QUICK)
    - `label` | VARCHAR(200) | NOT NULL
    - `default_price_zero` | TINYINT(1) | NOT NULL DEFAULT 0
    - `is_active` | TINYINT(1) | NOT NULL DEFAULT 1
    - `created_at` | DATETIME
    
    **Ghi chú:** nếu `default_price_zero=1` → ép `price=0`.
    
- **Listing_Images:** Chứa hình ảnh của mỗi tin. Gồm `image_id` (PK), `listing_id` (FK liên kết Listings), `image_url` (đường dẫn ảnh), `display_order` (số nguyên xác định thứ tự ưu tiên, ảnh cover có display_order=1). Một tin có thể có nhiều ảnh; ta sẽ hiển thị các ảnh theo `display_order`.
    
    ## `Listing_Images`
    
    - `image_id` | BIGINT AUTO_INCREMENT | **PK**
    - `listing_id` | BIGINT | FK → Listings.listing_id | NOT NULL
    - `image_url` | VARCHAR(1024) | NOT NULL
    - `display_order` | INT | NOT NULL DEFAULT 1
    - `created_at` | DATETIME
    
    **Ghi chú:** ảnh cover `display_order=1`.
    
    ## `Listing_Categories` (optional — nếu 1 listing thuộc nhiều category)
    
    - `listing_id` | BIGINT | FK → Listings.listing_id | NOT NULL
    - `category_id` | INT | FK → Categories.category_id | NOT NULL
        
        **PK:** (listing_id, category_id)
        
    
    ## `Moderation_Keywords`
    
    - `keyword_id` | INT AUTO_INCREMENT | **PK**
    - `keyword` | VARCHAR(255) | NOT NULL
    - `match_type` | ENUM('CONTAINS','WORD','EXACT') | NOT NULL DEFAULT 'CONTAINS'
    - `severity` | ENUM('BLOCK','FLAG') | NOT NULL DEFAULT 'BLOCK'
    - `is_active` | TINYINT(1) | NOT NULL DEFAULT 1
    - `created_at` | DATETIME
    
    **Ghi chú:** dùng để scan title/description; `BLOCK` = tự chặn; `FLAG` = gắn cờ cho admin review.
    
    ## `Listing_Moderation` (optional audit)
    
    - `id` | BIGINT AUTO_INCREMENT | **PK**
    - `listing_id` | BIGINT | FK → Listings.listing_id | NOT NULL
    - `keyword_id` | INT | FK → Moderation_Keywords.keyword_id | NOT NULL
    - `matched_text` | VARCHAR(500) | NULLABLE
    - `action_taken` | ENUM('AUTO_BLOCKED','FLAGGED','REVIEWED') | NOT NULL
    - `created_at` | DATETIME
    

## Kiểm duyệt và Báo cáo vi phạm

Hệ thống kiểm duyệt nhẹ đảm bảo không cho phép đăng các sản phẩm bất hợp pháp hay nội dung lừa đảo. Ví dụ, theo hướng dẫn nội dung thị trường, các tin đăng chứa hàng cấm (ma túy, vũ khí,…) hay thông tin sai lệch sẽ bị chặn ngay từ khâu đăng. Các quy tắc cụ thể gồm:

- Cấm đăng **mặt hàng bất hợp pháp hoặc trái phép** (ví dụ hàng cấm, hàng giả, v.v.).
- Ngăn ngừa **lừa đảo**: không cho phép thông tin gây hiểu nhầm, mô tả không đúng sự thật hoặc spam/duplicate listing.
- Phát hiện hành vi đăng tin nhanh bất thường (ví dụ tạo xóa tin liên tục) có thể dẫn đến giới hạn tự động; thực hiện **giới hạn tốc độ (velocity limits)** để chống spam, theo khuyến cáo thì cần cấm các hoạt động đăng tin quá nhanh/chóng.

Các vi phạm nội dung (bị người dùng khác báo cáo hoặc hệ thống phát hiện) sẽ được lưu vào bảng **Reports**, gồm: `report_id` (PK), `reporter_id` (FK liên Users), `target_id` (ID của User hoặc Listing bị báo cáo), `target_type` (enum 'USER' hoặc 'LISTING'), `reason` (lý do: Scam, Spam, Item cấm, v.v.), `evidence_image` (đường dẫn bằng chứng ảnh), `status` (enum: PENDING, RESOLVED, REJECTED) và `admin_note` (ghi chú của người duyệt). Mỗi khi báo cáo được gửi, admin sẽ xem xét và cập nhật trường `status`.

Ngoài ra, bảng **Saved_Listings** lưu danh sách tin mà người dùng đánh dấu yêu thích (wishlist). Bảng này là quan hệ many-to-many giữa Users và Listings (khóa chính là `(user_id, listing_id)`), cho phép người mua lưu lại tin cần theo dõi.

## `Reports`

- `report_id` | BIGINT AUTO_INCREMENT | **PK**
- `reporter_id` | BIGINT | FK → Users.user_id | NOT NULL
- `target_type` | ENUM('USER','LISTING') | NOT NULL
- `target_id` | BIGINT | NOT NULL -- id of user or listing
- `reason` | VARCHAR(200) | NULLABLE
- `evidence_image` | VARCHAR(1024) | NULLABLE
- `status` | ENUM('PENDING','RESOLVED','REJECTED') | NOT NULL DEFAULT 'PENDING'
- `admin_note` | TEXT | NULLABLE
- `handled_by` | BIGINT | FK → Users.user_id (admin) | NULLABLE
- `created_at`,`updated_at` | DATETIME

**Ghi chú:** threshold config (in Configurations) có thể trigger auto actions.

## Quản lý người dùng và tiện ích xã hội

Hệ thống quản lý thông tin người dùng và tương tác như sau:

- **Users:** Bảng chính lưu thông tin tài khoản gồm `user_id` (PK), `email` (unique), `full_name`, `phone_number`, `avatar_url`, `role` (enum 'ADMIN' hoặc 'USER'), `status` (ACTIVE, BANNED, RESTRICTED, DELETED), `reputation_score` (float, mặc định 5.0), `created_at`. Điểm `reputation_score` phản ánh uy tín của người dùng, tính dựa trên **điểm trung bình từ các đánh giá (Reviews)** mà họ nhận được.
    
    ## `User_Restrictions`
    
    - `restriction_id` | BIGINT AUTO_INCREMENT | **PK**
    - `user_id` | BIGINT | FK → Users.user_id | NOT NULL
    - `restriction_type` | ENUM('LIMIT_POSTS','TEMP_BAN') | NOT NULL
    - `params` | JSON | NULLABLE — e.g. {"max_posts_per_day":1,"until":"2026-02-11T00:00:00Z"}
    - `reason` | VARCHAR(255) | NULLABLE
    - `created_at` | DATETIME
    - `expires_at` | DATETIME | NULLABLE
    
    **Ghi chú:** applied when reports threshold hit.
    
- **Addresses:** Lưu địa chỉ hoặc điểm hẹn của người dùng. Mỗi `address_id` (PK) liên kết với `user_id`, và bao gồm `location_name` (tên điểm, ví dụ “Phòng A6” hoặc “Canteen Alpha”). Đây là nơi người bán muốn khách hàng đến lấy hàng (pickup point). Mỗi người dùng có thể có nhiều địa chỉ.
    
    ## `Addresses`
    
    - `address_id` | BIGINT AUTO_INCREMENT | **PK**
    - `user_id` | BIGINT | FK → Users.user_id | NOT NULL
    - `location_name` | VARCHAR(200) | NOT NULL
    - `address_text` | TEXT | NULLABLE
    - `lat` | DECIMAL(10,7) | NULLABLE
    - `lng` | DECIMAL(10,7) | NULLABLE
    - `is_default` | TINYINT(1) | NOT NULL DEFAULT 0
    - `created_at`,`updated_at` | DATETIME
- **Follows (Theo dõi):** Như đã mô tả ở trên, bảng này gồm `follower_id`, `followed_id` để thể hiện người này theo dõi người kia (cả hai cùng FK về Users.user_id). Cấu trúc nhiều-nhiều như trong các mạng xã hội: một người có thể có nhiều người theo dõi, và cũng theo dõi nhiều người khác.
    
    ## `Follows`
    
    - `follower_id` | BIGINT | FK → Users.user_id | NOT NULL
    - `followed_id` | BIGINT | FK → Users.user_id | NOT NULL
    - `created_at` | DATETIME
    
    **PK:** (follower_id, followed_id)
    
- **Blocks (Chặn):** Cấu trúc tương tự Follows nhưng dùng để chặn người khác. Bảng gồm `blocker_id`, `blocked_id` (PK tổ hợp) (FK về Users). Khi A chặn B, thì A sẽ không thấy tin của B và ngược lại.
    
    ## `Blocks`
    
    - `blocker_id` | BIGINT | FK → Users.user_id | NOT NULL
    - `blocked_id` | BIGINT | FK → Users.user_id | NOT NULL
    - `created_at` | DATETIME
    
    **PK:** (blocker_id, blocked_id)
    
- **Reviews & Rating:** Người mua có thể để lại đánh giá cho người bán hoặc tin đăng sau khi giao dịch. Bảng **Reviews** có thể gồm `review_id` (PK), `reviewer_id` (FK Users), `listing_id` (FK Listings), `rating` (ví dụ 1–5 sao), `comment`, `created_at`. Điểm `reputation_score` của người bán được tính trung bình từ các `rating` trong bảng này.
- **Comment :** Cho phép User có thể comment để hỏi về chi tiết sản phẩm và mục đích hiển thị thông tin trao đổi của bài đăng đó nhằm có thêm thông tin nhanh.
    
    

## Cấu hình hệ thống (Configurations)

Các thiết lập chung như danh mục sản phẩm, mục đích đăng, số ngày tối đa của tin đăng, và giới hạn đăng tin sẽ được lưu trong bảng **Configurations**. Thiết kế khuyến cáo một hàng cho mỗi cài đặt với cặp `config_name` (tên duy nhất) và `config_value` (giá trị chuỗi). Ví dụ: config_name="listing_expiration_days", config_value="3" để quy định tin đăng hết hạn sau 3 ngày; config_name="max_posts_per_day" có thể gán số lượng tin tối đa 1 người được đăng trong ngày (áp dụng khi user bị đánh dấu spam). Khi admin cập nhật cấu hình, lưu vào `updated_by` (FK Admin) để ghi lịch sử. Cấu hình theo cặp tên/giá trị này rất linh hoạt, cho phép dễ dàng thêm hoặc sửa tùy chọn mới mà không cần thay đổi cấu trúc bảng.

## `Posting_Limits` (config table for limits)

- `limit_id` | INT AUTO_INCREMENT | **PK**
- `name` | VARCHAR(200) | NOT NULL (e.g. 'max_posts_per_day_restricted')
- `limit_value` | INT | NOT NULL DEFAULT 1
- `applies_when` | ENUM('GLOBAL','RESTRICTED_USER') | NOT NULL DEFAULT 'GLOBAL'
- `description` | TEXT | NULLABLE
- `is_active` | TINYINT(1) | NOT NULL DEFAULT 1

## `Posting_Purposes` — Mục đích đăng (configurable)

- `purpose_id` | INT AUTO_INCREMENT | **PK**
- `code` | VARCHAR(50) | NOT NULL UNIQUE (e.g. SELL, GIVEAWAY, QUICK)
- `label` | VARCHAR(200) | NOT NULL
- `default_price_zero` | TINYINT(1) | NOT NULL DEFAULT 0
- `is_active` | TINYINT(1) | NOT NULL DEFAULT 1
- `created_at` | DATETIME

**Ghi chú:** nếu `default_price_zero=1` → ép `price=0`.

## `Configurations` — Key-value config

- `config_name` | VARCHAR(200) | **PK** | NOT NULL (e.g. 'listing_expiration_days')
- `config_value` | VARCHAR(1000) | NOT NULL
- `description` | TEXT | NULLABLE
- `updated_by` | BIGINT | FK → Users.user_id (admin) | NULLABLE
- `updated_at` | DATETIME

**Ví dụ values:**

- `listing_expiration_days` = `3`
- `flag_report_threshold` = `3`
- `max_posts_per_day_restricted` = `1`

# 1. Công cụ và Nền tảng Phát triển Backend

## 1.1 Ngôn ngữ & Framework

| **Tên cột** | **Kiểu dữ liệu** | **Mô tả / Ghi chú** |
| --- | --- | --- |
| **Ngôn ngữ lập trình** | Java | Ngôn ngữ chính cho toàn bộ backend |
| **Backend Framework** | Spring Boot | Xây dựng RESTful API và xử lý logic nghiệp vụ. Nhóm có kế hoạch đào tạo bắt buộc 2 tuần về Java & Spring Boot. |

## 1.2 Cơ sở dữ liệu

| **Tên cột** | **Kiểu dữ liệu** | **Mô tả / Ghi chú** |
| --- | --- | --- |
| **DBMS** | MySQL | Lưu trữ toàn bộ dữ liệu: người dùng, bài đăng, tin nhắn, lịch sử giao dịch. Nhóm dành 1 tuần đào tạo MySQL. |
| **Công cụ quản lý DB** | DBeaver | Giao diện đồ họa quản lý và thao tác cơ sở dữ liệu MySQL. |

## 1.3 Công cụ Kiểm thử & Đảm bảo chất lượng

| **Tên cột** | **Kiểu dữ liệu** | **Mô tả / Ghi chú** |
| --- | --- | --- |
| **API Testing** | Postman | Dùng trong giai đoạn Integration Test – kiểm tra luồng dữ liệu và đảm bảo 100% endpoints hoạt động đúng thiết kế. |
| **Unit Testing** | JUnit (Spring Boot) | Kiểm thử tự động cho controller và file xử lý. Mục tiêu: Code Coverage > 80%. |

## 1.4 Tích hợp & Hạ tầng

| **Tên cột** | **Kiểu dữ liệu** | **Mô tả / Ghi chú** |
| --- | --- | --- |
| **Xác thực (Auth)** | Google SSO | Đăng nhập nhanh bằng tài khoản Google (dành cho sinh viên). |
| **Mail Service** | Mail Gateway | Gửi thông báo và xác nhận qua email. |
| **IDE** | Visual Studio Code | Môi trường phát triển tích hợp. |
| **Version Control** | Git / GitLab | Quản lý phiên bản mã nguồn và lưu trữ source code. |
| **Deployment** | Amazon Web Services (AWS) | Máy chủ triển khai hệ thống production. |

## 1.5 Bảng tổng hợp Backend Stack

| **Hạng mục** | **Công nghệ / Công cụ** | **Mục đích sử dụng** |
| --- | --- | --- |
| Core Backend | Java / Spring Boot | Xây dựng RESTful API và xử lý nghiệp vụ |
| Database | MySQL / DBeaver | Lưu trữ dữ liệu hệ thống |
| API Testing | Postman | Kiểm thử tích hợp và kiểm tra Endpoints |
| Unit Testing | JUnit | Code Coverage > 80% |
| Authentication | Google SSO | Đăng nhập bằng tài khoản Google |
| Version Control | Git / GitLab | Quản lý phiên bản mã nguồn |
| Deployment | AWS | Máy chủ triển khai |

# 2. Mô hình Dữ liệu – Tổng quan

Hệ thống cơ sở dữ liệu SLife được chia thành 4 nhóm chức năng chính:

| **Nhóm** | **Các bảng bao gồm** |
| --- | --- |
| 1. User Management | Users, Addresses, Follows, Blocks, User_Restrictions |
| 2. Product & Listing | Categories, Listings, Listing_Images, Listing_Categories, Posting_Purposes, Moderation_Keywords, Listing_Moderation |
| 3. Interaction & Transaction | Conversations, Messages, Deals, Reviews |
| 4. Moderation & Utility | Reports, Saved_Listings, Notifications, Configurations, Posting_Limits |

Lưu ý: Hệ thống không xử lý đơn hàng mua bán chính thức (Orders). Thay vào đó, bảng Deals ghi nhận mức giá cuối cùng hai bên thỏa thuận (thỏa thuận giá), và hệ thống chỉ lưu trữ thông tin trò chuyện cùng kết quả chốt giá.

# 3. Nhóm Quản lý Người dùng (User Management)

Nhóm này lưu trữ danh tính, vai trò, trạng thái uy tín và các quan hệ xã hội của người dùng trong hệ thống.

## 3.1 Bảng Users – Người dùng & Admin

| **Tên cột** | **Kiểu dữ liệu** | **Mô tả / Ghi chú** |
| --- | --- | --- |
| **user_id** | INT AUTO_INCREMENT (PK) | Khóa chính định danh người dùng. |
| **email** | VARCHAR – UNIQUE | Email dùng Google SSO (unique cho từng tài khoản). |
| **full_name** | VARCHAR | Họ và tên hiển thị. |
| **phone_number** | VARCHAR | Số điện thoại dùng để liên lạc / xác thực. |
| **avatar_url** | VARCHAR | Đường dẫn ảnh đại diện (lưu URL, không lưu file BLOB). |
| **role** | ENUM('ADMIN','USER') | Quyền hạn trong hệ thống. |
| **status** | ENUM('ACTIVE','BANNED','RESTRICTED','DELETED') | Trạng thái tài khoản (BR-25, BR-44, BR-45). |
| **reputation_score** | FLOAT – Default 5.0 | Điểm uy tín, tính trung bình từ các đánh giá (Reviews) nhận được. Cập nhật khi có đánh giá mới. |
| **created_at** | DATETIME | Thời gian tạo tài khoản. |

## 3.2 Bảng Addresses – Địa chỉ / Điểm hẹn

Lưu địa chỉ hoặc điểm hẹn lấy hàng (Pickup Point) của người dùng. Mỗi người dùng có thể có nhiều địa chỉ; địa chỉ mặc định được đánh dấu is_default.

| **Tên cột** | **Kiểu dữ liệu** | **Mô tả / Ghi chú** |
| --- | --- | --- |
| **address_id** | BIGINT AUTO_INCREMENT (PK) | Khóa chính. |
| **user_id** | BIGINT – FK → Users.user_id | Liên kết với người dùng sở hữu địa chỉ. |
| **location_name** | VARCHAR(200) – NOT NULL | Tên điểm lấy hàng (ví dụ: Canteen Alpha, Dom E). |
| **address_text** | TEXT – NULLABLE | Mô tả chi tiết địa chỉ (tuỳ chọn). |
| **time_reminder** | DATETIME – NOT NULL | Thời gian nhắc nhở lấy hàng. |
| **lat / lng** | DECIMAL(10,7) – NULLABLE | Toạ độ GPS (tuỳ chọn) để hiển thị bản đồ. |
| **is_default** | TINYINT(1) – Default 0 | Đánh dấu địa chỉ mặc định (1 = mặc định). |
| **created_at, updated_at** | DATETIME | Thời gian tạo và cập nhật. |

*📝 Cập nhật khi người mua muốn thay đổi địa chỉ giao nhận hàng.*

## 3.3 Bảng Follows – Theo dõi người dùng

Quan hệ many-to-many. Khi người được theo dõi đăng tin mới (ACTIVE), hệ thống tự động tạo thông báo cho người theo dõi. Trang cá nhân hiển thị số lượng và danh sách người theo dõi.

| **Tên cột** | **Kiểu dữ liệu** | **Mô tả / Ghi chú** |
| --- | --- | --- |
| **follower_id** | BIGINT – FK → Users.user_id | ID người đi follow. |
| **followed_id** | BIGINT – FK → Users.user_id | ID người được follow (thường là Seller uy tín). |
| **created_at** | DATETIME | Thời gian bắt đầu theo dõi. |
| **PK** | (follower_id, followed_id) | Khóa chính tổ hợp, đảm bảo mỗi cặp chỉ follow một lần. |

## 3.4 Bảng Blocks – Chặn người dùng

Khi người A chặn B: A sẽ không nhận tin nhắn hay thấy tin đăng từ B, đồng thời B cũng không thấy tin đăng của A.

| **Tên cột** | **Kiểu dữ liệu** | **Mô tả / Ghi chú** |
| --- | --- | --- |
| **blocker_id** | BIGINT – FK → Users.user_id | Người thực hiện chặn. |
| **blocked_id** | BIGINT – FK → Users.user_id | Người bị chặn. |
| **created_at** | DATETIME | Thời gian thực hiện chặn. |
| **PK** | (blocker_id, blocked_id) | Khóa chính tổ hợp. |

## 3.5 Bảng User_Restrictions – Hạn chế người dùng

Được áp dụng khi người dùng vi phạm ngưỡng báo cáo (flag_report_threshold trong Configurations).

| **Tên cột** | **Kiểu dữ liệu** | **Mô tả / Ghi chú** |
| --- | --- | --- |
| **restriction_id** | BIGINT AUTO_INCREMENT (PK) | Khóa chính. |
| **user_id** | BIGINT – FK → Users.user_id | Người bị hạn chế. |
| **restriction_type** | ENUM('LIMIT_POSTS','TEMP_BAN') | Loại hạn chế: giới hạn số bài đăng hoặc cấm tạm thời. |
| **params** | JSON – NULLABLE | Tham số tuỳ chọn. Ví dụ: {"max_posts_per_day":1,"until":"2026-03-01T00:00:00Z"} |
| **reason** | VARCHAR(255) – NULLABLE | Lý do hạn chế. |
| **created_at** | DATETIME | Thời gian áp dụng hạn chế. |
| **expires_at** | DATETIME – NULLABLE | Thời gian hết hạn chế (NULL = vĩnh viễn). |

# 4. Nhóm Sản phẩm & Tin đăng (Product & Listing)

Quản lý toàn bộ vòng đời sản phẩm: từ bản nháp (DRAFT) đến khi đã bán (SOLD), cho tặng (GIVEN_AWAY), hoặc hết hạn (EXPIRED).

## 4.1 Bảng Categories – Danh mục sản phẩm (đa tầng)

Hỗ trợ cấu trúc phân cấp nhiều tầng thông qua self-FK parent_id. Ví dụ: Electronics → Điện thoại → iPhone.

| **Tên cột** | **Kiểu dữ liệu** | **Mô tả / Ghi chú** |
| --- | --- | --- |
| **category_id** | INT AUTO_INCREMENT (PK) | Khóa chính. |
| **name** | VARCHAR(200) – NOT NULL | Tên danh mục (unique tại cùng cấp tuỳ chính sách). |
| **description** | TEXT – NULLABLE | Mô tả danh mục. |
| **parent_id** | INT – FK → Categories.category_id – NULLABLE | Danh mục cha. NULL = danh mục gốc. Self-FK tạo cấu trúc cây đa tầng. |
| **level** | INT – NOT NULL DEFAULT 0 | Cấp độ danh mục (0 = gốc). |
| **is_active** | TINYINT(1) – Default 1 | Trạng thái hoạt động của danh mục. |
| **created_at, updated_at** | DATETIME | Thời gian tạo và cập nhật. |

## 4.2 Bảng Listings – Tin đăng sản phẩm

Bảng trung tâm quản lý các tin đăng. Hỗ trợ nhiều mục đích: bán, cho tặng, tin nhanh,… Các quy tắc nghiệp vụ quan trọng:

- Nếu mục đích là Giveaway (default_price_zero = 1 hoặc is_giveaway = 1) → price tự động = 0.00.
- Khi người bán bấm 'Đã bán' → status = 'SOLD', tin không hiển thị trong feed.
- expiration_date được gán theo cấu hình listing_expiration_days khi tạo tin.
- Tin hết hạn sẽ được xử lý bởi cron job hoặc MySQL EVENT để tự động ẩn / xoá.

| **Tên cột** | **Kiểu dữ liệu** | **Mô tả / Ghi chú** |
| --- | --- | --- |
| **listing_id** | BIGINT AUTO_INCREMENT (PK) | Khóa chính. |
| **seller_id** | BIGINT – FK → Users.user_id – NOT NULL | Người đăng tin. |
| **category_id** | INT – FK → Categories.category_id – NULLABLE | Danh mục sản phẩm. |
| **title** | VARCHAR(255) – NOT NULL | Tiêu đề tin đăng (bắt buộc, BR-03). |
| **description** | TEXT – NULLABLE | Mô tả chi tiết sản phẩm. |
| **price** | DECIMAL(10,2) – Default 0.00 | Giá bán. Tự động = 0 nếu là Giveaway (BR-14, BR-16). |
| **purpose_id** | INT – FK → Posting_Purposes.purpose_id – NULLABLE | Mục đích đăng (SELL, GIVEAWAY, QUICK, ...). |
| **is_giveaway** | TINYINT(1) – Default 0 | Đánh dấu tin cho tặng miễn phí. |
| **condition** | ENUM('NEW','USED_LIKE_NEW','USED_GOOD','USED_FAIR') | Tình trạng sản phẩm. |
| **status** | ENUM('DRAFT','ACTIVE','HIDDEN','SOLD','GIVEN_AWAY','BANNED','EXPIRED') | Trạng thái tin đăng (BR-36, BR-37, BR-38). |
| **expiration_date** | DATETIME – NULLABLE | Ngày hết hạn tin đăng (theo config listing_expiration_days). |
| **is_flagged** | TINYINT(1) – Default 0 | Đánh dấu tin bị gắn cờ vi phạm, chờ admin xem xét. |
| **view_count** | INT UNSIGNED – Default 0 | Số lượt xem tin đăng. |
| **display_priority** | INT – Default 0 | Độ ưu tiên hiển thị trong feed (số càng cao = ưu tiên hơn). |
| **created_at, updated_at** | DATETIME | Thời gian tạo và cập nhật tin. |

## 4.3 Bảng Posting_Purposes – Mục đích đăng

Bảng cấu hình (configurable) lưu các loại mục đích đăng tin. Admin có thể thêm/sửa loại mà không cần thay đổi cấu trúc DB.

| **Tên cột** | **Kiểu dữ liệu** | **Mô tả / Ghi chú** |
| --- | --- | --- |
| **purpose_id** | INT AUTO_INCREMENT (PK) | Khóa chính. |
| **code** | VARCHAR(50) – NOT NULL UNIQUE | Mã định danh duy nhất (SELL, GIVEAWAY, QUICK, ...). |
| **label** | VARCHAR(200) – NOT NULL | Nhãn hiển thị cho người dùng. |
| **default_price_zero** | TINYINT(1) – Default 0 | Nếu = 1 → ép giá = 0.00 khi người dùng chọn mục đích này. |
| **is_active** | TINYINT(1) – Default 1 | Trạng thái hoạt động của mục đích. |
| **created_at** | DATETIME | Thời gian tạo. |

## 4.4 Bảng Listing_Images – Ảnh sản phẩm

Lưu đường dẫn (URL) ảnh cho mỗi tin đăng. Không lưu trực tiếp file ảnh (BLOB) vào DB. Ảnh thực tế lưu trên Server hoặc AWS S3.

| **Tên cột** | **Kiểu dữ liệu** | **Mô tả / Ghi chú** |
| --- | --- | --- |
| **image_id** | BIGINT AUTO_INCREMENT (PK) | Khóa chính. |
| **listing_id** | BIGINT – FK → Listings.listing_id – NOT NULL | Liên kết với tin đăng. |
| **image_url** | VARCHAR(1024) – NOT NULL | URL đường dẫn tới ảnh. |
| **display_order** | INT – Default 1 | Thứ tự hiển thị. Ảnh bìa (cover) có display_order = 1. |
| **created_at** | DATETIME | Thời gian upload. |

## 4.5 Bảng Listing_Categories – Danh mục bổ sung (tuỳ chọn)

Dùng khi một tin đăng thuộc nhiều danh mục cùng lúc (quan hệ many-to-many).

| **Tên cột** | **Kiểu dữ liệu** | **Mô tả / Ghi chú** |
| --- | --- | --- |
| **listing_id** | BIGINT – FK → Listings.listing_id – NOT NULL | Liên kết với tin đăng. |
| **category_id** | INT – FK → Categories.category_id – NOT NULL | Liên kết với danh mục. |
| **PK** | (listing_id, category_id) | Khóa chính tổ hợp. |

## 4.6 Bảng Moderation_Keywords – Từ khoá kiểm duyệt

Lưu danh sách từ khoá cấm để quét nội dung tiêu đề và mô tả tin đăng tự động.

| **Tên cột** | **Kiểu dữ liệu** | **Mô tả / Ghi chú** |
| --- | --- | --- |
| **keyword_id** | INT AUTO_INCREMENT (PK) | Khóa chính. |
| **keyword** | VARCHAR(255) – NOT NULL | Từ khoá cần kiểm duyệt (ví dụ: ma túy, súng, ...). |
| **match_type** | ENUM('CONTAINS','WORD','EXACT') | Kiểu so khớp: chứa chuỗi / khớp từ / khớp chính xác. |
| **severity** | ENUM('BLOCK','FLAG') | BLOCK = tự chặn đăng; FLAG = gắn cờ cho admin review. |
| **is_active** | TINYINT(1) – Default 1 | Trạng thái hoạt động của từ khoá. |
| **created_at** | DATETIME | Thời gian thêm từ khoá. |

## 4.7 Bảng Listing_Moderation – Lịch sử kiểm duyệt (tuỳ chọn)

Bảng audit lưu log mỗi lần tin đăng bị quét và phát hiện vi phạm từ khoá.

| **Tên cột** | **Kiểu dữ liệu** | **Mô tả / Ghi chú** |
| --- | --- | --- |
| **id** | BIGINT AUTO_INCREMENT (PK) | Khóa chính. |
| **listing_id** | BIGINT – FK → Listings.listing_id | Tin đăng bị kiểm duyệt. |
| **keyword_id** | INT – FK → Moderation_Keywords.keyword_id | Từ khoá phát hiện được. |
| **matched_text** | VARCHAR(500) – NULLABLE | Đoạn văn bản chứa từ khoá bị phát hiện. |
| **action_taken** | ENUM('AUTO_BLOCKED','FLAGGED','REVIEWED') | Hành động đã thực hiện. |
| **created_at** | DATETIME | Thời gian phát hiện. |

# 5. Nhóm Tương tác & Giao dịch (Interaction & Transaction)

Xử lý luồng chat giữa hai người dùng, thương lượng giá và ghi nhận thỏa thuận cuối. Lưu ý: hệ thống KHÔNG xử lý đơn hàng mua bán chính thức – Deals chỉ là thỏa thuận giá được ghi nhận.

## 5.1 Bảng Conversations – Hội thoại

Mỗi cuộc hội thoại liên kết với một bài đăng cụ thể và hai người tham gia (không phân biệt rõ vai trò mua/bán).

| **Tên cột** | **Kiểu dữ liệu** | **Mô tả / Ghi chú** |
| --- | --- | --- |
| **conversation_id** | INT AUTO_INCREMENT (PK) | Khóa chính. |
| **user_id1** | INT – FK → Users.user_id | ID người tham gia thứ nhất. |
| **user_id2** | INT – FK → Users.user_id | ID người tham gia thứ hai. |
| **listing_id** | INT – FK → Listings.listing_id | Bài đăng liên quan đến cuộc hội thoại. |
| **last_message_at** | DATETIME | Thời gian tin nhắn mới nhất được gửi. |

*📝 Không phân biệt vai trò mua/bán trong conversations – dùng user_id1 và user_id2.*

## 5.2 Bảng Messages – Tin nhắn

Lưu từng tin nhắn trong các cuộc hội thoại. Lịch sử chat không bị xóa → cần đánh Index cho conversation_id và sent_at để truy vấn nhanh khi bảng lớn.

| **Tên cột** | **Kiểu dữ liệu** | **Mô tả / Ghi chú** |
| --- | --- | --- |
| **message_id** | INT AUTO_INCREMENT (PK) | Khóa chính. |
| **conversation_id** | INT – FK → Conversations.conversation_id | Cuộc hội thoại chứa tin nhắn này. |
| **sender_id** | INT – FK → Users.user_id | ID người gửi tin nhắn. |
| **content** | TEXT | Nội dung tin nhắn. |
| **sent_at** | DATETIME | Thời gian gửi. Nên đánh INDEX cùng conversation_id. |
| **is_read** | BOOLEAN – Default FALSE | Trạng thái đọc: FALSE = chưa xem. |

## 5.3 Bảng Deals – Thỏa thuận giá

Ghi nhận mức giá cuối cùng hai bên đồng ý. Không phải đơn hàng chính thức – chỉ lưu trữ thông tin thỏa thuận để gửi cho cả hai bên.

| **Tên cột** | **Kiểu dữ liệu** | **Mô tả / Ghi chú** |
| --- | --- | --- |
| **deal_id** | INT AUTO_INCREMENT (PK) | Khóa chính. |
| **conversation_id** | INT – FK → Conversations.conversation_id | Cuộc hội thoại liên quan. |
| **listing_id** | INT – FK → Listings.listing_id | Bài đăng đang thỏa thuận. |
| **proposed_by_id** | INT – FK → Users.user_id | ID người đề nghị giá. |
| **deal_price** | DECIMAL | Mức giá cuối cùng hai bên đồng ý. |
| **status** | ENUM('PENDING','CONFIRMED') | PENDING = chờ xác nhận; CONFIRMED = đã xác nhận. |
| **created_at** | DATETIME | Thời gian tạo bản ghi thỏa thuận. |

## 5.4 Bảng Reviews – Đánh giá người dùng

Cho phép đánh giá giữa hai người dùng sau khi hoàn tất thỏa thuận. Điểm rating (1–5 sao) được dùng để cập nhật reputation_score trong bảng Users.

| **Tên cột** | **Kiểu dữ liệu** | **Mô tả / Ghi chú** |
| --- | --- | --- |
| **review_id** | INT AUTO_INCREMENT (PK) | Khóa chính. |
| **conversation_id** | INT – FK → Conversations.conversation_id | Liên kết với cuộc hội thoại để xác định giao dịch. |
| **reviewer_id** | INT – FK → Users.user_id | ID người đánh giá (reviewer). |
| **reviewee_id** | INT – FK → Users.user_id | ID người được đánh giá (reviewee). |
| **rating** | INT (1–5) | Điểm đánh giá 1–5 sao. Cần Check Constraint: rating BETWEEN 1 AND 5. |
| **comment** | TEXT | Nội dung bình luận đánh giá. |
| **created_at** | DATETIME | Thời gian đánh giá. |

*📝 Mỗi đánh giá mới kích hoạt cập nhật reputation_score trung bình của reviewee trong bảng Users.*

# 6. Nhóm Kiểm duyệt & Tiện ích (Moderation & Utility)

## 6.1 Bảng Reports – Báo cáo vi phạm

Lưu báo cáo vi phạm từ người dùng (hoặc hệ thống) nhắm vào một tài khoản hoặc tin đăng cụ thể. Admin xem xét và cập nhật trường status.

| **Tên cột** | **Kiểu dữ liệu** | **Mô tả / Ghi chú** |
| --- | --- | --- |
| **report_id** | BIGINT AUTO_INCREMENT (PK) | Khóa chính. |
| **reporter_id** | BIGINT – FK → Users.user_id | Người thực hiện báo cáo. |
| **target_type** | ENUM('USER','LISTING') | Loại đối tượng bị báo cáo. |
| **target_id** | BIGINT – NOT NULL | ID của User hoặc Listing bị báo cáo. |
| **reason** | VARCHAR(200) – NULLABLE | Lý do báo cáo (Scam, Spam, Prohibited Item, ...). |
| **evidence_image** | VARCHAR(1024) – NULLABLE | URL ảnh bằng chứng (bắt buộc theo quy định báo cáo). |
| **status** | ENUM('PENDING','RESOLVED','REJECTED') – Default PENDING | Trạng thái xử lý báo cáo. |
| **admin_note** | TEXT – NULLABLE | Ghi chú của admin sau khi xem xét. |
| **handled_by** | BIGINT – FK → Users.user_id (admin) – NULLABLE | Admin đã xử lý báo cáo. |
| **created_at, updated_at** | DATETIME | Thời gian tạo và cập nhật. |

*📝 Threshold (ngưỡng báo cáo) được cấu hình trong bảng Configurations (flag_report_threshold). Khi vượt ngưỡng có thể kích hoạt tự động tạo User_Restrictions.*

## 6.2 Bảng Saved_Listings – Tin đã lưu (Wishlist)

Người dùng có thể lưu tin đăng yêu thích để theo dõi sau. Quan hệ many-to-many giữa Users và Listings.

| **Tên cột** | **Kiểu dữ liệu** | **Mô tả / Ghi chú** |
| --- | --- | --- |
| **user_id** | BIGINT – FK → Users.user_id | Người lưu tin. |
| **listing_id** | BIGINT – FK → Listings.listing_id | Tin đăng được lưu. |
| **PK** | (user_id, listing_id) | Khóa chính tổ hợp, đảm bảo mỗi người chỉ lưu một tin một lần. |

## 6.3 Bảng Notifications – Thông báo

Mỗi sự kiện quan trọng (tin đăng mới của người theo dõi, tin hết hạn, tin bị báo cáo, thông báo hệ thống) tạo một bản ghi thông báo. Khi người dùng bấm vào, hệ thống điều hướng đến nội dung liên quan.

| **Tên cột** | **Kiểu dữ liệu** | **Mô tả / Ghi chú** |
| --- | --- | --- |
| **notification_id** | INT AUTO_INCREMENT (PK) | Khóa chính. |
| **user_id** | INT – FK → Users.user_id | Người nhận thông báo. |
| **type** | ENUM('MESSAGE','ORDER_UPDATE','SYSTEM_ALERT') | Loại thông báo. |
| **content** | TEXT | Nội dung thông báo (kèm tiêu đề và link tới tin liên quan). |
| **is_read** | BOOLEAN – Default FALSE | Trạng thái đọc: FALSE = chưa xem. |
| **created_at** | DATETIME | Thời gian tạo thông báo. |

## 6.4 Bảng Comments – Bình luận tin đăng

Cho phép người dùng đặt câu hỏi công khai về chi tiết sản phẩm ngay trên trang tin đăng, giúp nhiều người cùng tham khảo thông tin trao đổi.

| **Tên cột** | **Kiểu dữ liệu** | **Mô tả / Ghi chú** |
| --- | --- | --- |
| **comment_id** | BIGINT AUTO_INCREMENT (PK) | Khóa chính. |
| **listing_id** | BIGINT – FK → Listings.listing_id | Tin đăng được bình luận. |
| **user_id** | BIGINT – FK → Users.user_id | Người viết bình luận. |
| **content** | TEXT – NOT NULL | Nội dung bình luận / câu hỏi. |
| **created_at** | DATETIME | Thời gian bình luận. |

# 7. Cấu hình Hệ thống (System Configuration)

Các thiết lập toàn cục được lưu dưới dạng cặp key-value. Thiết kế này linh hoạt: thêm/sửa cấu hình mà không cần thay đổi cấu trúc bảng. Admin cập nhật và lưu lịch sử qua trường updated_by.

## 7.1 Bảng Configurations – Cấu hình hệ thống

| **Tên cột** | **Kiểu dữ liệu** | **Mô tả / Ghi chú** |
| --- | --- | --- |
| **config_name** | VARCHAR(200) – PK – NOT NULL | Tên cấu hình (khóa chính dạng text). Ví dụ: listing_expiration_days. |
| **config_value** | VARCHAR(1000) – NOT NULL | Giá trị cấu hình dạng chuỗi. |
| **description** | TEXT – NULLABLE | Mô tả ý nghĩa cấu hình. |
| **updated_by** | BIGINT – FK → Users.user_id (admin) – NULLABLE | Admin đã thực hiện cập nhật gần nhất. |
| **updated_at** | DATETIME | Thời gian cập nhật gần nhất. |

## 7.2 Ví dụ giá trị Configurations

| **config_name** | **config_value** | **Ý nghĩa** |
| --- | --- | --- |
| listing_expiration_days | 3 | Tin đăng hết hạn sau 3 ngày kể từ ngày tạo. |
| flag_report_threshold | 3 | Sau 3 báo cáo, tự động kích hoạt hạn chế người dùng. |
| max_posts_per_day_restricted | 1 | Người dùng bị hạn chế chỉ được đăng tối đa 1 tin/ngày. |

## 7.3 Bảng Posting_Limits – Giới hạn đăng tin

| **Tên cột** | **Kiểu dữ liệu** | **Mô tả / Ghi chú** |
| --- | --- | --- |
| **limit_id** | INT AUTO_INCREMENT (PK) | Khóa chính. |
| **name** | VARCHAR(200) – NOT NULL | Tên quy tắc (ví dụ: max_posts_per_day_restricted). |
| **limit_value** | INT – NOT NULL DEFAULT 1 | Giá trị giới hạn. |
| **applies_when** | ENUM('GLOBAL','RESTRICTED_USER') | Phạm vi áp dụng: toàn cầu hoặc chỉ người dùng bị hạn chế. |
| **description** | TEXT – NULLABLE | Mô tả chi tiết quy tắc. |
| **is_active** | TINYINT(1) – Default 1 | Trạng thái hoạt động. |

# 8. Lưu ý Triển khai trên MySQL

## 8.1 Ràng buộc dữ liệu (Constraints)

| **Bảng / Cột** | **Ràng buộc cần áp dụng** |
| --- | --- |
| Categories.name | UNIQUE tại cùng cấp danh mục (hoặc toàn bảng tuỳ chính sách). |
| Reviews.rating | CHECK CONSTRAINT: rating BETWEEN 1 AND 5. |
| Listings.price / is_giveaway | Nếu is_giveaway = TRUE → price phải = 0.00. Thực hiện validation tại Application Layer; có thể bổ sung DB Trigger làm phòng vệ. |
| Messages | INDEX trên (conversation_id, sent_at) để truy vấn lịch sử chat nhanh khi bảng lớn (lịch sử không bị xóa). |
| Listings.expiration_date | Cron job hoặc MySQL EVENT định kỳ tự động ẩn / xóa tin hết hạn. |

## 8.2 Lưu trữ ảnh

Không lưu trực tiếp file ảnh vào Database (BLOB) để tránh quá tải. Lưu đường dẫn ảnh (URL String) trong các bảng Listing_Images và Users (avatar_url). File ảnh thực tế lưu trên Server hoặc AWS S3.

## 8.3 Kiểm duyệt nội dung (Content Moderation)

- Scan tiêu đề và mô tả tin đăng đối chiếu với bảng Moderation_Keywords khi người dùng tạo/chỉnh sửa tin.
- BLOCK: tự động chặn tin đăng và thông báo ngay cho người dùng.
- FLAG: gắn cờ (is_flagged = 1) và thêm log vào Listing_Moderation để admin review.
- Velocity limits: phát hiện và chặn hành vi tạo/xóa tin liên tục bất thường (chống spam).

## 8.4 Quy trình Auto-restriction

- Khi báo cáo vi phạm của một người dùng vượt ngưỡng flag_report_threshold trong Configurations → tự động tạo bản ghi trong User_Restrictions.
- restriction_type = LIMIT_POSTS: giới hạn đăng tối đa N tin/ngày (N theo Posting_Limits).
- restriction_type = TEMP_BAN: cấm tạm thời đến expires_at.
- Admin có thể xem xét và điều chỉnh/xóa restriction thủ công.

*— Hết tài liệu —*

**SLIFE**

Hướng dẫn Cài đặt Môi trường Phát triển

Windows  •  JDK 21  •  Spring Boot 3  •  MySQL 8  •  React Vite

Phiên bản: 1.0  •  11/02/2026

# Tổng quan

Tài liệu này hướng dẫn từng bước cài đặt toàn bộ môi trường phát triển trên Windows để chạy được dự án SLife với stack:

| **Tầng** | **Công nghệ** | **Phiên bản khuyến nghị** |
| --- | --- | --- |
| Backend | Java + Spring Boot | JDK 21 (LTS)  •  Spring Boot 3.3.x |
| Database | MySQL | MySQL 8.0.x (bản mới nhất của dòng 8) |
| DB Client | DBeaver | DBeaver Community 24.x (miễn phí) |
| IDE | IntelliJ IDEA | IntelliJ IDEA Community 2024.x (miễn phí) |
| Frontend | React + Vite | Node.js 20 LTS  •  Vite 5.x |

*💡 Đây là hướng dẫn cho môi trường local (localhost). Không cần Docker hay cài gì lên cloud.*

# Bước 1 – Cài đặt JDK 21

## 1.1 Tải JDK 21

Truy cập trang chính thức của Eclipse Temurin (bản phân phối OpenJDK miễn phí, được Oracle khuyến nghị):

https://adoptium.net/temurin/releases/?version=21

- Chọn: Operating System → Windows
- Architecture → x64
- Package Type → JDK
- Version → 21 - LTS
- Tải file .msi (installer)

*💡 Tại sao JDK 21? Đây là bản LTS (Long Term Support) mới nhất tính đến 2026, Spring Boot 3.x yêu cầu tối thiểu Java 17.*

## 1.2 Cài đặt

1. Chạy file .msi vừa tải → Next → Next → chọn Add to PATH (quan trọng!) → Install.
2. Sau khi cài xong, mở Command Prompt (Win + R → gõ cmd → Enter).
3. Gõ lệnh kiểm tra:

java -version

javac -version

✅ Kết quả đúng: openjdk version "21.x.x" và javac 21.x.x

⚠️  Nếu báo 'java' is not recognized → kiểm tra lại mục Add to PATH khi cài, hoặc thêm thủ công vào System Environment Variables.

## 1.3 Thêm JAVA_HOME (nếu IntelliJ không nhận)

Vào Start → tìm 'Edit the system environment variables' → Environment Variables → New:

Variable name:  JAVA_HOME

Variable value: C:\Program Files\Eclipse Adoptium\jdk-21.x.x.x-hotspot

Sau đó thêm vào PATH dòng:  %JAVA_HOME%\bin

## Bước 2 – Cài đặt MySQL 8

### 2.1 Tải MySQL Installer

Truy cập:

https://dev.mysql.com/downloads/installer/

- Chọn: mysql-installer-community-8.0.x.msi (bản Full ~450 MB)
- Không cần tạo Oracle account – bấm 'No thanks, just start my download'.

### 2.2 Cài đặt MySQL

1. Chạy installer → Chọn setup type: Developer Default → Next.
2. Installer sẽ kiểm tra prerequisites → bấm Execute để cài các gói phụ thuộc.
3. Next → Execute (tải và cài MySQL Server 8.0, MySQL Workbench, Connector/J,...).
4. Sau khi cài xong → Next → Next vào màn hình cấu hình.
5. Cấu hình MySQL Server:
    - Config Type: Development Computer
    - Port: 3306 (mặc định, giữ nguyên)
    - Authentication: Use Strong Password Encryption (Recommended)
6. Đặt mật khẩu root: gõ mật khẩu vào ô MySQL Root Password (ví dụ: root1234). Nhớ mật khẩu này vì sẽ dùng ở bước Spring Boot.
7. Tiếp tục Next → Execute → Finish.

*💡 MySQL Workbench sẽ được cài cùng lúc. Tuy nhiên mình sẽ dùng DBeaver thay thế vì giao diện dễ hơn.*

### 2.3 Kiểm tra MySQL chạyBước 3 – Cài đặt DBeaver

Mở Command Prompt, gõ:

mysql -u root -p

Nhập password vừa đặt. Nếu vào được prompt mysql> là thành công.

mysql> SHOW DATABASES;

mysql> EXIT;

# Bước 3:

### 3.1 Tải DBeaver Community

https://dbeaver.io/download/

- Chọn: DBeaver Community → Windows (Installer)
- Chạy .exe → cài theo mặc định, không cần thay gì.

### 3.2 Kết nối MySQL trong DBeaver

1. Mở DBeaver → bấm biểu tượng New Database Connection (hình phích cắm + dấu +).
2. Chọn MySQL → Next.
3. Điền thông tin:

Host:     localhost

Port:     3306

Database: (để trống hoặc điền slife_demo sau khi tạo)

Username: root

Password: <mật khẩu đã đặt ở Bước 2>

1. Bấm Test Connection → DBeaver sẽ tự tải MySQL driver nếu chưa có. Bấm Download khi được hỏi.
2. Kết nối thành công → Finish.

✅ Thấy được danh sách databases trong panel bên trái là đã kết nối thành công.

### 3.3 Chạy file SQL khởi tạo DB

1. Trong DBeaver, click chuột phải vào kết nối → SQL Editor → New SQL Script.
2. Copy toàn bộ nội dung file database/init.sql (trong thư mục dự án) dán vào.
3. Bấm Ctrl + A để chọn tất cả → Ctrl + Enter để chạy.

✅ Bảng users xuất hiện với 4 dòng dữ liệu mẫu là thành công.

# Bước 4 – Cài đặt IntelliJ IDEA & mở project Backend

### 4.1 Tải IntelliJ IDEA Community

https://www.jetbrains.com/idea/download/?section=windows

- Chọn tab Community → tải .exe → cài mặc định.
- Tick chọn Add launchers dir to the PATH và .java file association khi cài.

### 4.2 Mở project Spring Boot

1. Mở IntelliJ → File → Open → chọn thư mục slife-demo/backend (thư mục chứa pom.xml).
2. IntelliJ tự nhận Maven project. Đợi IntelliJ tải dependencies (góc dưới có thanh tiến trình).
3. IntelliJ sẽ hỏi Trust this project? → Bấm Trust Project.

*💡 Lần đầu mở sẽ mất 3-5 phút để tải tất cả thư viện Spring Boot từ Maven Central.*

### 4.3 Cấu hình kết nối DB trong application.properties

Mở file:  backend/src/main/resources/application.properties

Sửa dòng:

spring.datasource.password=YOUR_PASSWORD_HERE

Thành mật khẩu root MySQL bạn đặt ở Bước 2. Ví dụ:

spring.datasource.password=root1234

### 4.4 Chạy Spring Boot

1. Mở file DemoApplication.java (src/main/java/com/slife/demo/).
2. Bấm nút Run ▶ màu xanh ở góc trên bên phải, hoặc Shift + F10.
3. Xem Console ở panel bên dưới. Đợi đến khi thấy dòng:

Started DemoApplication in x.xxx seconds (process running for x.xxx)

✅ Thấy dòng trên → Spring Boot đang chạy tại http://localhost:8080

⚠️  Nếu lỗi 'Access denied for user root' → sai password trong application.properties.

⚠️  Nếu lỗi 'Communications link failure' → MySQL chưa chạy. Vào Services (Win+R → services.msc) → tìm MySQL80 → Start.

### 4.5 Kiểm tra API bằng trình duyệt

Mở trình duyệt, truy cập:

http://localhost:8080/api/users

✅ Trình duyệt hiện JSON danh sách 4 user → Backend + DB chạy thành công!

# Bước 5 – Cài đặt Node.js & chạy Frontend React

### 5.1 Tải Node.js 20 LTS

https://nodejs.org/en/download

- Chọn: LTS → Windows Installer (.msi) → tải và cài mặc định.
- Tick chọn Automatically install the necessary tools khi cài (tự cài thêm Chocolatey & build tools).

Kiểm tra sau khi cài:

node -v    # phải ra v20.x.x

npm -v     # phải ra 10.x.x

### 5.2 Tạo project React Vite

Mở terminal (PowerShell hoặc cmd) tại thư mục slife-demo/frontend, chạy:

cd slife-demo\frontend

npm create vite@latest . -- --template react

# Khi hỏi 'Current directory is not empty' → chọn Ignore files and continue

# Framework: React

# Variant: JavaScript

npm install

*💡 Lệnh trên sẽ tạo project Vite trong thư mục hiện tại. File App.jsx đã có sẵn trong thư mục src/ của bạn.*

## 5.3 Thay thế App.jsx

Xóa nội dung mặc định và thay bằng file App.jsx đã có trong thư mục slife-demo/frontend/src/:

# Nếu bạn copy thủ công: xóa nội dung App.jsx cũ

# và paste toàn bộ nội dung file App.jsx từ thư mục dự án vào.

### 5.4 Chạy Frontend

cd slife-demo\frontend

npm run dev

✅ Terminal hiện:  VITE v5.x.x  ready in xxx ms  → Local: http://localhost:5173

Mở trình duyệt vào http://localhost:5173

# Bước 6 – Test Full Stack FE ↔ BE ↔ DB

### 6.1 Checklist trước khi test

| **Dịch vụ** | **Trạng thái cần đạt** |
| --- | --- |
| MySQL | Đang chạy (Services → MySQL80 = Running) |
| Spring Boot | Console IntelliJ không có lỗi đỏ |
| Spring Boot | http://localhost:8080/api/users trả JSON |
| React Vite | Terminal hiện 'ready in xxx ms' |
| React Vite | http://localhost:5173 mở được trang web |

### 6.2 Thực hiện test

1. Mở http://localhost:5173 trên trình duyệt.
2. Bấm nút '📥 Lấy danh sách User từ DB'.
3. Bảng hiện 4 dòng dữ liệu (An, Bình, Cường, Admin) → Thành công!
4. Thử thêm user mới: điền Email + Họ tên → bấm '💾 Lưu vào DB'.
5. Bấm lại '📥 Lấy danh sách User từ DB' → thấy user mới vừa thêm.
6. Vào DBeaver → click chuột phải bảng users → Refresh → kiểm tra user mới cũng có trong DB.

✅ Thấy dữ liệu trên cả FE (trang web) và trong DB (DBeaver) → toàn bộ stack đã thông!

## 6.3 Sơ đồ luồng dữ liệu

| **React Vite :5173** | **→** | **Spring Boot :8080** | **→** | **MySQL :3306 slife_demo → users** |
| --- | --- | --- | --- | --- |

# Xử lý lỗi thường gặp

| **Lỗi** | **Nguyên nhân & Cách sửa** |
| --- | --- |
| java is not recognized | JDK chưa thêm vào PATH. Vào Environment Variables → PATH → thêm C:\Program Files\Eclipse Adoptium\jdk-21.x\bin |
| Access denied for user 'root' | Sai password trong application.properties. Kiểm tra lại spring.datasource.password. |
| Communications link failure | MySQL chưa chạy. Win+R → services.msc → tìm MySQL80 → Start. |
| Table 'slife_demo.users' doesn't exist | Chưa chạy file init.sql. Mở DBeaver, chạy lại file database/init.sql. |
| CORS error trên FE | Đảm bảo @CrossOrigin trong UserController có origins = 'http://localhost:5173'. Vite mặc định chạy port 5173. |
| Port 8080 already in use | Một process khác đang dùng port 8080. Mở Task Manager → tìm java.exe → End Task, rồi chạy lại Spring Boot. |
| npm : command not found | Node.js chưa cài hoặc chưa add to PATH. Cài lại Node.js 20 LTS và tick chọn 'Add to PATH'. |

# Cấu trúc thư mục dự án

slife-demo/

├── backend/                          ← Mở bằng IntelliJ

│   ├── pom.xml                       ← Maven dependencies

│   └── src/main/

│       ├── java/com/slife/demo/

│       │   ├── DemoApplication.java  ← Điểm khởi chạy

│       │   ├── entity/User.java      ← Map sang bảng users

│       │   ├── repository/UserRepository.java

│       │   └── controller/UserController.java  ← REST API

│       └── resources/

│           └── application.properties ← Cấu hình DB

│

├── frontend/                         ← Mở bằng VS Code hoặc terminal

│   ├── package.json

│   └── src/

│       └── App.jsx                   ← Giao diện test

│

└── database/

└── init.sql                      ← Chạy trong DBeaver

*— Hết tài liệu —*

[https://github.com/DOTHANHAN12/Test.git](https://github.com/DOTHANHAN12/Test.git)

@Tran Thi Ngoc Anh (K18 HL) @Le Duc Viet (K18 HL) @Hoang Anh Tu K17 @La Thanh Hoa 

@Tran Thi Ngoc Anh (K18 HL) @Le Duc Viet (K18 HL) @Hoang Anh Tu K17 @La Thanh Hoa DATABASE

PICKUP TIME, DEAL

[SLife_Database_SQL.zip](SLife_Database_SQL.zip)

BO timeremider trong address, bang useer thieu updated at