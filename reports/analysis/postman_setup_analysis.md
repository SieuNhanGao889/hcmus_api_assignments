# Phân tích setup dữ liệu cho giai đoạn Postman

## 1. Phạm vi và nguyên tắc

Tài liệu này chuẩn bị cho `PHASE_6_POSTMAN_IMPLEMENTATION` đối với ba API:

- `POST /api/login`
- `POST /api/apply-coupon`
- `PUT /api/admin/orders/:id/status`

Phân tích dựa trên:

- `AGENT.md` và `agent-skill/SKILL.md`;
- đề HW06 tại `docs/2026.HW06.API Testing_En.md`;
- đặc tả endpoint tại `docs/api_specification.md`;
- ba workbook cuối đã audit trong `test-cases/`;
- yêu cầu nghiệp vụ của SUT tại `EShop-source/README.md`;
- implementation hiện tại tại `EShop-source/backend/server.js`, `database.js` và bản đọc chỉ-read của `database.sqlite`.

Không có API request nào được gửi, backend không được khởi động và SUT không bị sửa trong quá trình phân tích. Các bảng trong `database.sqlite` chỉ được đọc bằng kết nối SQLite read-only.

Trạng thái source local được kiểm tra là commit `85af3ba875c88283615e22cb108f13e2fccaf0e9`. Working tree có thay đổi chưa commit ở `server.js` (chỉ đổi địa chỉ listen thành `0.0.0.0`) và `database.sqlite`; vì vậy kết luận về dữ liệu bên dưới phản ánh đúng working tree local hiện tại, không giả định database upstream.

## 2. Kết luận quan trọng trước khi triển khai

1. Dữ liệu hiện tại có hai user seed và bốn coupon seed, nhưng không có order hay coupon usage.
2. `server.js` thực hiện `require("./database")`; `database.js` gọi `initDatabase()` ngay khi được load. Hàm này `DROP TABLE` rồi tạo/seed lại toàn bộ database. Vì vậy mỗi lần chạy `node server.js` sẽ xóa user/coupon/order/usage đã tạo ở phiên trước, dù `setup_guide.md` diễn đạt việc seed như một bước chỉ cần chạy một lần.
3. Không được hard-code `user_id`, `coupon_id` hoặc `orderId` chỉ vì database hiện đang bắt đầu từ `1`. ID phải lấy từ response hoặc request discovery trong chính phiên chạy.
4. Có thể tạo tất cả order state cần thiết từ một order `pending` mới bằng `POST /api/checkout` rồi thực hiện chuỗi transition hợp lệ. Phải dùng order riêng cho các case có thể mutate state.
5. Có thể tạo coupon có rule kiểm soát bằng `POST /api/admin/coupons`, nhưng endpoint này không cho đặt `is_active`. Vì vậy không thể tạo fixture inactive qua API hiện có.
6. `POST /api/apply-coupon` không tự ghi `coupon_usage`. Muốn chuẩn bị user đã hết lượt, phải gọi riêng `POST /api/coupon-usage` bằng token của đúng user.
7. Implementation chỉ kiểm tra JWT ở các route admin, không kiểm tra `role = 'admin'`. Đây là điều cần test, không được dùng như một cơ chế setup hợp lệ: mọi setup admin vẫn phải dùng token admin.
8. Các workbook được audit chủ yếu dựa vào file endpoint specification ngắn. `EShop-source/README.md` hiện định nghĩa thêm lockout, năm điều kiện coupon, công thức discount và transition matrix. Do đó một số case đang ghi `SPEC_UNDEFINED` hoặc “exploratory” cần sinh viên human-review lại trước khi biến thành assertion Postman. Tài liệu này không thay đổi quyết định audit trong workbook.

## 3. Dữ liệu thực tế hiện có

### 3.1 User seed

`database.js`, `EShop-source/README.md` và database local hiện tại thống nhất về hai tài khoản sau:

| Vai trò | Email | Password | Trạng thái hiện tại |
|---|---|---|---|
| Admin | `admin@eshop.com` | `Admin123!` | `role = admin`, không bị khóa |
| Normal user | `test@eshop.com` | `Test1234!` | `role = user`, không bị khóa |

Đây là credential được xác nhận từ source/seed, không phải giá trị giả lập. `setup_guide.md` lại ghi password admin là `admin123`; giá trị đó mâu thuẫn với seed và không nên dùng. Source seed/database hiện tại là căn cứ thực thi.

Database hiện tại gán ID `1` và `2`, nhưng các ID này chỉ là quan sát trước khi server chạy. Runner phải lấy ID động vì reset, đăng ký thêm user hoặc thay đổi thứ tự setup có thể làm ID khác đi.

### 3.2 Coupon seed

| Code | Type | `discount_value` | `min_order_amount` | `expired_at` | `is_active` | `max_uses_per_user` |
|---|---:|---:|---:|---|---:|---:|
| `SAVE10` | `percent` | `10` | `300000` | `2099-12-31` | `1` | `1` |
| `BIGBUY` | `fixed` | `50000` | `500000` | `2099-12-31` | `1` | `1` |
| `VIP100` | `fixed` | `100000` | `300000` | `2099-12-31` | `1` | `2` |
| `EXPIRED` | `percent` | `20` | `100000` | `2020-01-01` | `1` | `1` |

Tại thời điểm phân tích, `coupon_usage` rỗng. Bốn code trên cũng được mô tả trong SUT README, nên có thể dùng làm baseline seed. Tuy nhiên các case cần rule chính xác, độc lập và chạy lặp lại nên ưu tiên coupon tạo riêng theo từng run.

### 3.3 Order

Database hiện tại không có order. `database.js` cũng không seed order. Mọi `existingOrderId`, `pendingOrderId`, `confirmedOrderId`, `shippingOrderId`, `deliveredOrderId` và `canceledOrderId` phải được tạo trong cùng phiên backend/runner.

## 4. Cách lấy hoặc tạo dữ liệu test

### 4.1 Valid normal user và `validUserId`

Có hai chiến lược:

**Baseline seed:** login bằng normal user seed, lấy `token` và `user.id` từ response. Sau đó có thể gọi `GET /api/users/me` để xác minh identity.

**Disposable user, khuyến nghị cho test làm thay đổi login state:** trước mỗi nhóm lockout/wrong-password, gọi `POST /api/register` với email duy nhất theo `runId`; lấy `id` từ response, rồi login thành công một lần để lấy `userToken` và đối chiếu `user.id`. Không truyền `role`; route register luôn dùng default `user`.

Không dùng cùng seed user cho toàn bộ negative login suite. Mỗi lần sai password trên account tồn tại làm thay đổi `login_attempts`; implementation hiện tăng `+2` và có thể khóa account từ lần sai thứ hai. Điều này làm các case sau phụ thuộc thứ tự.

### 4.2 Valid admin user

Không có endpoint hợp lệ để tạo admin. `POST /api/register` không nhận role, còn `PUT /api/users/me` có implementation cho phép gửi `role` nhưng sử dụng cách đó sẽ dựa vào một lỗ hổng privilege escalation và làm sai setup.

Vì vậy admin setup phải dùng admin seed đã được xác nhận. Gọi `POST /api/login`, trích `token` thành `adminToken`, trích `user.id` thành `adminUserId`, và assert `user.role === "admin"` trước khi tiếp tục. Nếu login/role check này không đạt, dừng các request setup admin thay vì chạy với token không đúng vai trò.

### 4.3 Valid coupon và coupon có rule kiểm soát

Hai nguồn hợp lệ:

- discovery bằng `GET /api/coupons` với `adminToken`, sau đó chọn coupon seed theo `code`;
- tạo coupon riêng bằng `POST /api/admin/coupons` với `adminToken`, code duy nhất chứa `runId`, rồi lấy `id` từ response và xác minh lại bằng `GET /api/coupons`.

Nên tạo tối thiểu các fixture sau cho một run độc lập:

| Fixture logic | Rule cần tạo | Mục đích |
|---|---|---|
| `fixedCoupon` | `type=fixed`, discount dương, ngày tương lai, min rõ ràng | schema và công thức fixed |
| `percentCoupon` | `type=percent`, phần trăm rõ ràng, ngày tương lai | công thức percent |
| `minOrderCoupon` | `min_order_amount` đúng bằng boundary trong data file | dưới/bằng/trên ngưỡng |
| `expiredCoupon` | `expired_at` chắc chắn trong quá khứ | expired rule |
| `oneUseCoupon` | `max_uses_per_user=1` | first-use/reuse |
| `twoUseCoupon` | `max_uses_per_user=2` | boundary usage count |

Không ghi một code cụ thể trong collection trước khi code đó thực sự được tạo. Pre-request script có thể sinh code từ prefix ổn định và `runId`; response create cung cấp ID thật.

Để tạo trạng thái “đã hết lượt”, gọi `POST /api/coupon-usage` đúng số lần bằng `userToken` của user cần kiểm tra và `coupon_id` đã trích. Chỉ gọi lặp lại `POST /api/apply-coupon` không làm tăng usage.

Không thể tạo inactive coupon bằng endpoint hiện có vì `POST /api/admin/coupons` bỏ qua `is_active`, còn `DELETE` biến coupon thành không tồn tại chứ không phải inactive. Case phân biệt inactive với unknown cần fixture DB ngoài API hoặc thay đổi SUT; hiện chưa khả dụng.

### 4.4 Existing order ID và order ở state hữu ích

Gọi `POST /api/checkout` bằng `userToken`. Route trả `orderId` và luôn tạo order với status `pending`. Không cần dựa vào giỏ hàng để tạo fixture theo implementation hiện tại, nhưng request vẫn nên có `total_amount` và `shipping_address` test-data riêng, không dùng dữ liệu cá nhân thật.

Tạo order riêng cho từng source state:

```text
POST /api/checkout
        |
        +--> pendingOrderId
        |
        +--> PUT pending -> confirmed
        |         `--> confirmedOrderId
        |                  |
        |                  `--> PUT confirmed -> shipping
        |                            `--> shippingOrderId
        |                                      |
        |                                      `--> PUT shipping -> delivered
        |                                                `--> deliveredOrderId
        |
        `--> PUT pending -> canceled
                  `--> canceledOrderId
```

Mỗi nhánh phải bắt đầu từ một checkout mới. Không dùng một order duy nhất cho nhiều dòng transition matrix, vì request trước sẽ đổi precondition của request sau. Dùng `GET /api/admin/orders` với admin token hoặc `GET /api/orders/my-orders` với owner token để xác minh state trước/sau.

Không có API xóa order hoặc reset order về `pending`. Cleanup order bằng API hiện không khả dụng; isolation đạt được bằng `runId` và order mới, còn reset hoàn toàn chỉ xảy ra khi backend khởi động lại và `database.js` seed lại database.

## 5. Endpoint setup và verification hiện có

| Endpoint | Vai trò trong flow | Dữ liệu lấy/tạo | Lưu ý từ implementation |
|---|---|---|---|
| `POST /api/register` | Setup | normal user, response `id` | Không validation/unique constraint đáng tin cậy; dùng email theo `runId` |
| `POST /api/login` | Setup và API under test | token, `user.id`, `user.role` | Wrong password làm đổi lockout state; success response trả toàn bộ row user |
| `GET /api/users/me` | Verification | identity của token | Trả toàn bộ row user; assertions phải kiểm tra không lộ secret theo spec/security |
| `GET /api/admin/users` | Discovery phụ | danh sách ID/user | Implementation chỉ kiểm JWT, không kiểm admin role; setup vẫn phải dùng admin token |
| `GET /api/coupons` | Discovery/verification | coupon rule và ID | Cần JWT, nhưng implementation không kiểm role |
| `POST /api/admin/coupons` | Setup | controlled coupon, response `id` | Không nhận `is_active`; luôn dùng admin token |
| `DELETE /api/admin/coupons/:id` | Cleanup | xóa coupon tạo trong run | Không tự xóa `coupon_usage`; cleanup nên chạy cuối |
| `POST /api/coupon-usage` | Setup state | tăng usage cho user từ JWT | `user_id` lấy từ token, không lấy từ body |
| `POST /api/checkout` | Setup | order `pending`, response `orderId` | Cần user token; implementation tin `total_amount` client |
| `GET /api/orders/my-orders` | Verification | order của user đang login | Phù hợp để verify owner order |
| `GET /api/admin/orders` | Discovery/verification | toàn bộ order và state | Dùng admin token dù implementation thiếu role check |
| `GET /api/orders/:id` | Verification phụ/security observation | một order | Implementation hiện không yêu cầu JWT; không nên dùng làm bằng chứng rằng access control đúng |
| `PUT /api/admin/orders/:id/status` | API under test và setup state | transition order | Mỗi setup transition cũng phải được kiểm tra và tách khỏi request test chính |
| `PUT /api/orders/:id/cancel` | Setup/đối chiếu cancel | tạo canceled state cho order của owner | Có thể dùng, nhưng để setup admin-state matrix đơn giản hơn nên dùng admin transition `pending -> canceled` |

Không có health endpoint. Khi triển khai Newman/CI sau này, có thể dùng `GET /api/products` chỉ như readiness probe; không tính request đó là execution của một trong ba API đã chọn.

## 6. Phân loại biến Postman

### 6.1 Environment variables

Các giá trị phụ thuộc môi trường hoặc cần dùng xuyên nhiều folder:

| Variable | Nguồn |
|---|---|
| `baseUrl` | Local environment: `http://localhost:3000` |
| `studentId` | Giá trị bài tập: `23127364` |
| `seedAdminEmail`, `seedAdminPassword` | Seed đã xác nhận; đánh dấu sensitive cho password |
| `seedUserEmail`, `seedUserPassword` | Seed đã xác nhận; đánh dấu sensitive cho password |
| `adminToken`, `userToken` | Trích động từ login; không commit token thật |
| `adminUserId`, `validUserId` | Trích động |
| `runId` | Sinh một lần đầu run để tránh collision |
| `unknownUserId`, `unknownOrderId`, `unknownCouponCode` | Tính động sau discovery, không hard-code |

Credential/token chỉ nên có ở local/current value hoặc được sinh/trích lúc chạy; file environment nộp bài không chứa token thật ngoài các credential seed đã công khai trong SUT. Không log password/token trong Postman Console.

### 6.2 Collection variables

Các giá trị ổn định theo thiết kế collection, không phụ thuộc một máy cụ thể:

- endpoint paths hoặc tên request dùng cho flow;
- danh sách domain `pending,confirmed,shipping,delivered,canceled` dùng cho schema/domain check;
- tên schema/assertion mode như `SUCCESS_SCHEMA`, `NO_TOKEN`, `STATE_UNCHANGED`, `EXPLORATORY`;
- prefix dữ liệu như `HW06_{{runId}}_`;
- biến tạm cấp collection chỉ khi cần truyền giữa request trong cùng collection và không phù hợp để export như cấu hình môi trường.

Không đặt ID fixture cố định hoặc expected business rule chưa được xác nhận vào collection variables.

### 6.3 Data-file values

Data file phù hợp với các partition không cần nhiều request setup bất đồng bộ:

- `case_id`, `case_source`, `audit_status`;
- body field/value cho missing, null, empty, wrong type, whitespace và boundary;
- `contentType`, raw body mode, `expected_status` khi đặc tả xác định;
- `assertion_mode` cho các case `SPEC_UNDEFINED`/exploratory;
- coupon `total_amount`, expected calculation inputs;
- order `source_state`, `target_status`, `transition_expected`;
- tên logical fixture, ví dụ `coupon_fixture_key` hoặc `order_fixture_key`, thay vì ID thật.

Không để password, token, `user_id`, `coupon_id` hay `orderId` thật trong data file. Data file chỉ tham chiếu logical key; script/request lookup ID động tương ứng.

Các case malformed JSON, duplicate keys, missing body, wrong `Content-Type`, cross-request login verification và state transition nhiều bước nên là request riêng. Ép tất cả vào một generic data row sẽ làm collection khó đọc và khó audit.

### 6.4 Dynamically extracted/generated variables

- `adminToken`, `userToken`, `loginUserObject` từ login success;
- `adminUserId`, `validUserId` từ `response.user.id`, register response hoặc `/users/me`;
- `createdCouponId` và map `couponFixtureIds` từ create/list coupon;
- `pendingOrderId`, `confirmedOrderId`, `shippingOrderId`, `deliveredOrderId`, `canceledOrderId` từ checkout/setup transitions;
- `beforeOrderStatus`, `afterOrderStatus` cho persisted verification;
- `tamperedAdminToken` bằng cách thay đúng một ký tự trong token đã trích, không tự bịa token có vẻ hợp lệ;
- email/code duy nhất từ `runId`;
- ID “unknown” bằng cách lấy maximum ID sau discovery rồi cộng một khoảng, đồng thời xác minh nó không có trong response list trước khi test.

## 7. Case cần controlled database/SUT state

### 7.1 Login

- `LOGIN-GEN-001..004` và `LOGIN-EXT-001..002` cần account hợp lệ, không khóa; extension cần token truyền sang `/users/me`.
- `LOGIN-GEN-040` bắt buộc disposable account có `login_attempts=0`, chuỗi request đúng thứ tự và đo thời gian nếu kiểm tra duration.
- Mọi case dùng email tồn tại nhưng password không đúng/khác dạng đều làm đổi login state, nổi bật: `LOGIN-GEN-006`, `011`, `013`, `015`, `017`, `021`, `023`, `025`, `027`, `029` và request B của `LOGIN-EXT-004`. Không chạy tuần tự trên cùng seed user nếu không chủ ý kiểm tra lockout.
- `LOGIN-GEN-005`, `007` và request A của `LOGIN-EXT-004` cần chứng minh email không tồn tại, nhưng không mutate account.
- Duplicate-key case `LOGIN-EXT-003` có thể dùng credential hợp lệ tùy parser; nên dùng disposable account để tránh side effect lockout không dự kiến.

### 7.2 Apply coupon

- Positive/schema/calculation cases cần coupon rule được biết và user ID thật.
- `COUPON-GEN-004` cần expired fixture; seed `EXPIRED` hiện có hoặc tạo riêng.
- `COUPON-GEN-005..007` và `COUPON-EXT-002` cần `min_order_amount` đúng với boundary trong row. Workbook dùng ví dụ `200000`, trong khi seed không có coupon với threshold này; phải tạo coupon riêng hoặc sửa data theo fixture sau human review.
- `COUPON-GEN-008..009` và `COUPON-EXT-003` cần usage count kiểm soát bằng `/api/coupon-usage`.
- `COUPON-EXT-001` cần fixed/percent rule đã xác nhận để tính expected numeric.
- `COUPON-GEN-030..031` cần user không tồn tại/user khác được xác minh trước.
- Extra-field cases `COUPON-GEN-034..035` và `COUPON-EXT-004` cần server-side coupon rule đã biết để chứng minh client không override.

### 7.3 Admin order status

Gần như mọi case có `existingOrderId` cần order riêng với state biết trước. Đặc biệt:

- `ORDERSTATUS-GEN-002`: source `pending`;
- `ORDERSTATUS-GEN-003`: source `confirmed`;
- `ORDERSTATUS-GEN-004`: source `shipping`;
- `ORDERSTATUS-GEN-005`: source `pending` hoặc `confirmed`;
- `ORDERSTATUS-GEN-036..042`, `ORDERSTATUS-EXT-003..005`: fixture theo state cụ thể;
- `ORDERSTATUS-GEN-007..011`, `031`, `ORDERSTATUS-EXT-001..002`: dedicated order và read-before/read-after, vì một lỗi authorization có thể thật sự mutate order;
- ID/payload invalid cases nên đọc state sau request để chứng minh không có mutation nếu chúng dùng existing order.

`ORDERSTATUS-GEN-001` không thể là positive transition sang `pending`: state machine không có transition hợp lệ nào có target `pending`. Case này cần human-review/reclassification thành negative, không nên tìm cách tạo fixture để làm nó pass như positive.

## 8. Case/fixture chưa thể thực thi tin cậy

### 8.1 Blocker hiện tại

Trong thời điểm phân tích, backend chưa được khởi động và không có execution evidence. Không case nào được coi là đã chạy. Khi bước execution bắt đầu, phải giữ backend trong cùng một phiên cho toàn bộ setup + test vì restart sẽ reset database.

### 8.2 Hạn chế cụ thể

| Hạng mục | Lý do chưa tin cậy/không khả dụng | Hướng xử lý |
|---|---|---|
| Inactive coupon khác unknown coupon | API create không nhận `is_active`; seed không có inactive coupon | Cần DB fixture được phê duyệt hoặc bổ sung setup capability; hiện không triển khai thành automated assertion |
| Reuse chỉ bằng hai lần apply | `apply-coupon` không ghi usage | Chèn `/api/coupon-usage` như setup; nếu mục tiêu là kiểm tra “apply tự consume” thì behavior đó không tồn tại |
| Order có sẵn qua restart | `database.js` xóa orders khi server start | Tạo order trong cùng run |
| Cleanup/reset từng order | Không có delete/reset order endpoint | Dùng order mới cho từng case; reset toàn bộ chỉ giữa các run bằng restart có chủ ý |
| Tạo admin qua API | Không có API hợp lệ | Dùng admin seed; không dùng privilege-escalation bug để setup |
| JWT expired hợp lệ | `jwt.sign` không đặt `expiresIn`; không có token-expiry fixture | Chỉ test malformed/tampered; expired-token case cần token fixture/secret-side setup ngoài API |
| `ORDERSTATUS-GEN-001` như positive | Không có valid transition sang `pending` | Human-review và đổi thành negative/exploratory đúng state machine |
| Các case `SPEC_UNDEFINED` nhưng SUT README đã định nghĩa rule | Workbook dùng specification ngắn, trong khi SUT README có FR chi tiết | Human re-audit expected result trước khi viết hard assertion |
| Lockout dùng chung seed account | Negative cases thay đổi state và implementation khóa lâu | Dùng disposable user cho từng scenario; không chạy song song cùng account |

Ngoài ra, `GET /api/orders/:id` hiện public và các `/api/admin/*` chỉ kiểm JWT, không kiểm role. Đây là hành vi implementation cần được test/đánh giá theo FR-12/SEC-02/SEC-03; không được coi là setup contract hợp lệ.

## 9. Cấu trúc Postman collection đề xuất

```text
HW06 EShop API Tests
├── 00 - Bootstrap and Discovery
│   ├── Generate run context
│   ├── Login seed admin -> adminToken/adminUserId
│   ├── Login or register normal user -> userToken/validUserId
│   ├── Discover users/coupons/orders
│   └── Validate required setup
├── 01 - POST Login
│   ├── 01 - Positive and success schema
│   ├── 02 - Email partitions (data-driven)
│   ├── 03 - Password partitions (isolated users where needed)
│   ├── 04 - HTTP/parser/security dedicated requests
│   ├── 05 - Lockout sequence (isolated folder)
│   └── 06 - Token and identity cross-request verification
├── 02 - POST Apply Coupon
│   ├── 00 - Create/discover controlled coupon fixtures
│   ├── 01 - Positive and response schema
│   ├── 02 - Code/user/amount partitions (data-driven)
│   ├── 03 - Formula and min-order boundaries
│   ├── 04 - Expiration and usage-limit sequences
│   ├── 05 - Security/parser/extra-field cases
│   └── 99 - Delete created coupons
├── 03 - PUT Admin Order Status
│   ├── 00 - Create pending orders
│   ├── 01 - Build source-state fixtures
│   ├── 02 - Authentication and authorization
│   ├── 03 - Path ID partitions
│   ├── 04 - Status/body partitions
│   ├── 05 - Transition matrix (data-driven)
│   ├── 06 - Terminal/repeated/cancellation cases
│   └── 07 - Persisted-state verification
└── 99 - Run Summary and Cleanup
    ├── Verify no unresolved setup failure
    └── Cleanup created coupons
```

Bootstrap request không được che mất traceability: mỗi test request phải chứa `case_id` trong tên/description và tham chiếu row workbook. Setup transition nên có prefix `SETUP`, không được tính như execution của test case khác.

Không nên chạy cả ba folder data-driven bằng một data file duy nhất. Login, coupon và order status có lifecycle khác nhau; nên có ba file data như cấu trúc README dự kiến, cộng request riêng cho các scenario sequence/raw-body.

## 10. Cách sử dụng hợp lý các tính năng Postman

### Workspace

Tạo một workspace riêng cho HW06, chứa đúng collection và local environment. Dùng workspace để lưu descriptions, examples và lịch sử thiết kế; không đưa token/runtime ID vào tài liệu chia sẻ.

### Collection

Một collection cho toàn bộ HW06 để collection-level scripts/header áp dụng thống nhất, bên trong chia folder theo ba API và setup/verification. Mỗi request gắn `case_id`, `source`, requirement/spec reference và mô tả precondition.

### Environment

Dùng `HW06 Local` cho `baseUrl`, `studentId`, seed credentials và runtime values. Token/ID được overwrite lúc chạy. Trước run phải xóa stale runtime variables để không vô tình dùng token/order từ phiên backend trước.

### Variables

Áp dụng phân loại tại mục 6. Script phải kiểm tra unresolved `{{variable}}` và fail setup rõ ràng thay vì gửi placeholder literal tới SUT.

### Pre-request scripts

Collection-level pre-request script dùng `pm.request.headers.upsert` để thêm:

```javascript
pm.request.headers.upsert({
  key: "X-Student-Id",
  value: pm.environment.get("studentId")
});
console.log("X-Student-Id attached:", pm.environment.get("studentId"));
```

Console log này hỗ trợ evidence bài tập nhưng không log token/password. Pre-request script cũng có thể sinh `runId`, email/code duy nhất, chọn raw body từ data row và tạo tampered token từ token thật. Không nên dùng chuỗi `pm.sendRequest` phức tạp để âm thầm tạo toàn bộ fixture; setup request tường minh dễ audit hơn.

### Post-response test scripts

Các script nên:

- assert status chỉ khi workbook/requirement xác định; với `SPEC_UNDEFINED`, ghi observation và assert invariant an toàn như “không có token/discount/mutation”;
- assert JSON parsing, required field và type;
- kiểm tra login không lộ `password`, `reset_token` hoặc secret;
- trích token/ID chỉ sau khi status/schema setup hợp lệ;
- tính expected coupon từ rule fixture, không copy giá trị SUT trả về làm expected;
- đọc before/after order state để kiểm mutation;
- fail setup với message có `case_id`, không ghi nhận setup failure như SUT bug.

### Collection Runner

Runner phù hợp cho từng folder/data file. Chạy tuần tự các scenario có state; tắt parallelism đối với cùng user/coupon/order. Lockout folder và transition folder phải có setup riêng. Dùng summary request hoặc console để đối chiếu số iteration với số row được chọn, nhưng không tự sửa workbook execution result nếu chưa lưu evidence.

### Data-driven runs

Dùng `login_data.json`, `coupon_data.json`, `order_status_data.json` cho các partition chuẩn. Transition matrix row chứa logical source state; request setup ánh xạ logical key sang ID được tạo trong run. Các raw malformed/duplicate-key case ở request riêng để giữ chính xác byte/body semantics.

### Newman

Sau khi collection được human-review và SUT đã chạy, dùng Newman với environment và từng data file/folder khi cần. Lệnh tổng quát dự kiến của repository vẫn là:

```bash
newman run postman/HW06_EShop_API_Tests.postman_collection.json \
  -e postman/HW06_Local.postman_environment.json \
  -r cli,html \
  --reporter-html-export reports/newman/HW06_EShop_API_Tests.html
```

Đối với data-driven folder, bổ sung `-d postman/data/<file>.json` và giới hạn folder bằng `--folder` khi lifecycle yêu cầu. Newman command chưa được chạy và chưa có kết quả nào được tuyên bố.

## 11. Human-review checkpoint trước khi tạo collection

Trước khi viết Postman artifact, sinh viên cần xác nhận tối thiểu các quyết định sau:

1. Dùng `EShop-source/README.md` làm nguồn requirement bổ sung cho lockout, coupon và state machine, thay vì tiếp tục coi các rule đó là `SPEC_UNDEFINED`.
2. Reclassify `ORDERSTATUS-GEN-001` khỏi positive case.
3. Quyết định các coupon usage cases sẽ dùng `/api/coupon-usage` như setup hay chỉ quan sát việc apply không tự ghi usage.
4. Chấp thuận tạo disposable users/coupons/orders qua API trong runner và chấp nhận orders không có cleanup endpoint.
5. Chọn cách xử lý inactive-coupon case: tạm hoãn, DB fixture được kiểm soát, hoặc ghi rõ limitation.

Tài liệu dừng tại checkpoint này. Chưa tạo collection, environment hoặc data file Postman; chưa gửi API request; chưa thay đổi SUT.
