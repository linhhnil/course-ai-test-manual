# Software Requirements Specification (SRS)
## Module: Đăng nhập (Admin Login) – Perfex CRM | Anh Tester Demof

| Thông tin | Giá trị |
|---|---|
| Hệ thống | Perfex CRM – Anh Tester Demo |
| URL | https://crm.anhtester.com/admin/authentication |
| Module | Authentication – Login / Forgot Password |
| Phiên bản tài liệu | 1.0 |
| Ngày tạo | 2026-11-17 |
| Tài khoản demo | `admin@example.com` / `123456` |

> **Ghi chú về nguồn dữ liệu:** Các yêu cầu đánh dấu **[Quan sát]** được xác minh trực tiếp trên trang (cấu trúc HTML, thông báo lỗi thực tế). Các yêu cầu đánh dấu **[Cần xác nhận]** dựa trên hành vi mặc định của Perfex CRM và cần tester kiểm chứng lại khi chạy test với tài khoản demo.

---

## 1. Giới thiệu

### 1.1 Mục đích
Tài liệu mô tả các yêu cầu chức năng và phi chức năng của module Đăng nhập khu vực quản trị (Admin/Staff) của hệ thống Perfex CRM, làm cơ sở cho việc thiết kế test case, kiểm thử thủ công và tự động.

### 1.2 Phạm vi
Bao gồm:
- Màn hình Đăng nhập (`/admin/authentication`)
- Chức năng "Remember me"
- Màn hình Quên mật khẩu (`/admin/authentication/forgot_password`)
- Điều hướng sau khi đăng nhập / đăng xuất

Không bao gồm: Đăng nhập khu vực Client (`/authentication/login`), đăng ký tài khoản, xác thực 2 lớp (2FA) – nếu được bật sẽ được mô tả ở tài liệu riêng.

### 1.3 Thuật ngữ

| Thuật ngữ | Ý nghĩa |
|---|---|
| Staff / Admin | Người dùng nội bộ đăng nhập vào khu vực quản trị |
| CSRF token | Mã chống giả mạo request, gửi kèm form dưới dạng field ẩn `csrf_token_name` |
| Session | Phiên làm việc của người dùng sau khi đăng nhập thành công |

---

## 2. Mô tả tổng quan

### 2.1 Người dùng
| Tác nhân | Mô tả |
|---|---|
| Admin | Quản trị viên có toàn quyền hệ thống |
| Staff | Nhân viên được cấp tài khoản, quyền theo role |
| Khách (chưa đăng nhập) | Người truy cập trang login |

### 2.2 Giả định & ràng buộc
- Ứng dụng web, truy cập qua trình duyệt hiện đại (Chrome, Firefox, Edge, Safari).
- Giao tiếp qua HTTPS.
- Tài khoản Staff do Admin tạo trước; không có tự đăng ký.

---

## 3. Đặc tả giao diện (UI)

### 3.1 Màn hình Login – `/admin/authentication` [Quan sát]

Tiêu đề trang (browser tab): `Perfex CRM | Anh Tester Demo - Login`. Tiêu đề form: **Login**.

| # | Thành phần | Loại | Thuộc tính HTML | Bắt buộc | Ghi chú |
|---|---|---|---|---|---|
| 1 | Logo | Link/Image | `href="https://crm.anhtester.com/"` | – | Click về trang chủ |
| 2 | Email Address | Textbox | `type="email"`, `id="email"`, `name="email"`, `autofocus` | Có | Được focus sẵn khi tải trang |
| 3 | Password | Textbox | `type="password"`, `id="password"`, `name="password"` | Có | Ký tự được che |
| 4 | Remember me | Checkbox | `id="remember"`, `name="remember"` | Không | Mặc định không chọn |
| 5 | Login | Button | `type="submit"`, class `btn-primary btn-block` | – | Submit form |
| 6 | Forgot Password? | Link | `/admin/authentication/forgot_password` | – | Chuyển sang màn hình quên mật khẩu |
| 7 | CSRF token | Hidden | `name="csrf_token_name"` | Có | Sinh tự động |

- Form: `method="post"`, `action="/admin/authentication"`.
- Không giới hạn `maxlength` ở phía client cho Email và Password.
- Thông báo lỗi hiển thị dạng khối `alert alert-danger text-center` phía trên các trường nhập.

### 3.2 Màn hình Forgot Password – `/admin/authentication/forgot_password` [Quan sát]

| # | Thành phần | Loại | Thuộc tính | Bắt buộc |
|---|---|---|---|---|
| 1 | Tiêu đề "Forgot Password" | Label | – | – |
| 2 | Email Address | Textbox | `type="email"`, `id="email"`, `name="email"` | Có |
| 3 | Confirm | Button | `type="submit"` | – |
| 4 | CSRF token | Hidden | `name="csrf_token_name"` | Có |

---

## 4. Yêu cầu chức năng

### FR-01: Hiển thị màn hình đăng nhập
- **FR-01.1** [Quan sát] Khi người dùng chưa đăng nhập truy cập `/admin/authentication`, hệ thống hiển thị form Login với các thành phần ở mục 3.1.
- **FR-01.2** [Quan sát] Con trỏ tự động focus vào trường Email Address.
- **FR-01.3** [Cần xác nhận] Khi người dùng chưa đăng nhập truy cập bất kỳ URL nào dưới `/admin/*`, hệ thống chuyển hướng về trang Login.
- **FR-01.4** [Cần xác nhận] Khi người dùng đã đăng nhập truy cập `/admin/authentication`, hệ thống chuyển hướng về Dashboard (`/admin`).

### FR-02: Kiểm tra dữ liệu đầu vào (Validation)

| Mã | Điều kiện | Kết quả mong đợi | Nguồn |
|---|---|---|---|
| FR-02.1 | Email và Password đều để trống, nhấn Login | Hiển thị 2 thông báo: `The Password field is required.` và `The Email Address field is required.` Không đăng nhập. | [Quan sát] |
| FR-02.2 | Email trống, Password có giá trị | Hiển thị `The Email Address field is required.` | [Cần xác nhận] |
| FR-02.3 | Email có giá trị, Password trống | Hiển thị `The Password field is required.` | [Cần xác nhận] |
| FR-02.4 | Email sai định dạng (vd: `abc`) | Trình duyệt chặn submit (HTML5) và hiện tooltip, vd Chrome: `Please include an '@' in the email address. 'abc' is missing an '@'.` | [Quan sát] |
| FR-02.5 | Email/Password chỉ chứa khoảng trắng | Xử lý như trường trống (trim) | [Cần xác nhận] |
| FR-02.6 | Email viết hoa/thường khác nhau (vd `ADMIN@EXAMPLE.COM`) | Đăng nhập thành công (email không phân biệt hoa thường) | [Cần xác nhận] |
| FR-02.7 | Password phân biệt hoa thường | `123456` hợp lệ; sai chữ hoa/thường bị từ chối | [Cần xác nhận] |

### FR-03: Xác thực đăng nhập

| Mã | Điều kiện | Kết quả mong đợi | Nguồn |
|---|---|---|---|
| FR-03.1 | Email + Password đúng (`admin@example.com` / `123456`) | Tạo session, chuyển hướng tới Dashboard `/admin` | [Cần xác nhận] |
| FR-03.2 | Email đúng, Password sai | Hiển thị lỗi `Invalid email or password`, giữ lại giá trị Email, xoá Password | [Cần xác nhận] |
| FR-03.3 | Email không tồn tại | Hiển thị cùng thông báo `Invalid email or password` (không tiết lộ email có tồn tại hay không) | [Cần xác nhận] |
| FR-03.4 | Tài khoản Staff bị vô hiệu hoá (inactive) | Từ chối đăng nhập, hiển thị thông báo tài khoản không hoạt động | [Cần xác nhận] |
| FR-03.5 | Submit form không có / sai CSRF token (vd để trang mở quá lâu) | Request bị từ chối, yêu cầu tải lại trang | [Cần xác nhận] |
| FR-03.6 | Nhấn phím Enter khi đang ở trường Email/Password | Submit form tương đương nhấn nút Login | [Quan sát – button type=submit] |

### FR-04: Remember me
- **FR-04.1** Khi chọn "Remember me" và đăng nhập thành công, hệ thống lưu cookie ghi nhớ đăng nhập; người dùng đóng/mở lại trình duyệt vẫn ở trạng thái đăng nhập. [Cần xác nhận]
- **FR-04.2** Khi không chọn "Remember me", phiên đăng nhập kết thúc khi session hết hạn / đóng trình duyệt. [Cần xác nhận]
- **FR-04.3** Khi đăng xuất, cookie ghi nhớ bị xoá. [Cần xác nhận]

### FR-05: Quên mật khẩu
- **FR-05.1** [Quan sát] Click "Forgot Password?" chuyển tới `/admin/authentication/forgot_password`.
- **FR-05.2** [Cần xác nhận] Để trống Email, nhấn Confirm → hiển thị `The Email Address field is required.`
- **FR-05.3** [Quan sát – HTML5] Email sai định dạng → trình duyệt chặn submit.
- **FR-05.4** [Cần xác nhận] Email tồn tại → gửi email chứa link đặt lại mật khẩu, hiển thị thông báo thành công.
- **FR-05.5** [Cần xác nhận] Email không tồn tại → hiển thị thông báo lỗi phù hợp.
- **FR-05.6** [Cần xác nhận] Link đặt lại mật khẩu chỉ dùng được 1 lần và có thời hạn.

### FR-06: Đăng xuất
- **FR-06.1** [Cần xác nhận] Người dùng chọn Logout từ menu profile → huỷ session, chuyển về trang Login.
- **FR-06.2** [Cần xác nhận] Sau khi đăng xuất, nhấn Back trên trình duyệt không truy cập lại được trang quản trị.

---

## 5. Luồng nghiệp vụ (Use Case)

### UC-01: Đăng nhập thành công
| Mục | Nội dung |
|---|---|
| Tác nhân | Admin / Staff |
| Tiền điều kiện | Có tài khoản hợp lệ, đang active; chưa đăng nhập |
| Luồng chính | 1. Truy cập `/admin/authentication` → 2. Nhập Email → 3. Nhập Password → 4. (Tuỳ chọn) chọn Remember me → 5. Nhấn Login → 6. Hệ thống xác thực → 7. Chuyển tới Dashboard |
| Luồng thay thế | 5a. Nhấn Enter thay cho click Login |
| Luồng ngoại lệ | 6a. Thiếu dữ liệu → FR-02; 6b. Sai thông tin → FR-03.2/03.3; 6c. Tài khoản inactive → FR-03.4 |
| Hậu điều kiện | Session được tạo, người dùng ở trang Dashboard |

### UC-02: Lấy lại mật khẩu
| Mục | Nội dung |
|---|---|
| Tác nhân | Admin / Staff |
| Tiền điều kiện | Email đã đăng ký trong hệ thống |
| Luồng chính | 1. Tại Login click "Forgot Password?" → 2. Nhập Email → 3. Nhấn Confirm → 4. Hệ thống gửi email đặt lại mật khẩu → 5. Người dùng mở link, đặt mật khẩu mới → 6. Đăng nhập với mật khẩu mới |
| Luồng ngoại lệ | 3a. Email trống/sai định dạng → FR-05.2/05.3; 3b. Email không tồn tại → FR-05.5 |

---

## 6. Yêu cầu phi chức năng

| Mã | Loại | Yêu cầu |
|---|---|---|
| NFR-01 | Bảo mật | Toàn bộ trang dùng HTTPS. |
| NFR-02 | Bảo mật | Password được che ký tự khi nhập (`type="password"`) và không hiển thị lại trong form sau khi submit lỗi. |
| NFR-03 | Bảo mật | Form có CSRF token chống giả mạo request. [Quan sát] |
| NFR-04 | Bảo mật | Thông báo lỗi không tiết lộ email có tồn tại hay không. |
| NFR-05 | Bảo mật | Mật khẩu lưu dạng băm (hash), không lưu plain text. |
| NFR-06 | Bảo mật | Chống SQL Injection / XSS ở các trường Email, Password (vd nhập `' OR 1=1 --`, `<script>alert(1)</script>` không gây lỗi hay thực thi). |
| NFR-07 | Bảo mật | Khuyến nghị: giới hạn số lần đăng nhập sai liên tiếp hoặc dùng CAPTCHA để chống brute-force. [Cần xác nhận cấu hình] |
| NFR-08 | Hiệu năng | Thời gian phản hồi khi submit đăng nhập ≤ 3 giây trong điều kiện mạng bình thường. |
| NFR-09 | Khả dụng | Giao diện responsive, hiển thị đúng trên desktop, tablet, mobile. |
| NFR-10 | Khả dụng | Hỗ trợ điều hướng bằng bàn phím: Tab giữa các trường, Enter để submit. |
| NFR-11 | Tương thích | Hoạt động đúng trên Chrome, Firefox, Edge, Safari phiên bản mới nhất. |
| NFR-12 | Khả năng truy cập | Label được gắn với input qua thuộc tính `for` (email, password, remember). [Quan sát] |

---

## 7. Bảng thông báo hệ thống

| Mã | Ngữ cảnh | Nội dung | Nguồn |
|---|---|---|---|
| MSG-01 | Email trống | `The Email Address field is required.` | [Quan sát] |
| MSG-02 | Password trống | `The Password field is required.` | [Quan sát] |
| MSG-03 | Email sai định dạng (HTML5, Chrome) | `Please include an '@' in the email address. '<value>' is missing an '@'.` | [Quan sát] |
| MSG-04 | Sai email hoặc mật khẩu | `Invalid email or password` | [Cần xác nhận] |
| MSG-05 | Tài khoản inactive | Thông báo tài khoản không hoạt động, liên hệ quản trị viên | [Cần xác nhận] |
| MSG-06 | Gửi email reset thành công | Thông báo đã gửi email hướng dẫn đặt lại mật khẩu | [Cần xác nhận] |

---

## 8. Dữ liệu kiểm thử gợi ý

| Bộ dữ liệu | Email | Password | Kết quả mong đợi |
|---|---|---|---|
| Hợp lệ | `admin@example.com` | `123456` | Đăng nhập thành công |
| Sai mật khẩu | `admin@example.com` | `1234567` | MSG-04 |
| Email không tồn tại | `notexist@example.com` | `123456` | MSG-04 |
| Trống cả hai | (trống) | (trống) | MSG-01 + MSG-02 |
| Email sai định dạng | `admin` | `123456` | MSG-03 |
| Email hoa | `ADMIN@EXAMPLE.COM` | `123456` | Đăng nhập thành công [Cần xác nhận] |
| Khoảng trắng đầu/cuối | `  admin@example.com  ` | `123456` | Đăng nhập thành công (trim) [Cần xác nhận] |
| SQL Injection | `admin@example.com' --` | `' OR '1'='1` | Từ chối, không lỗi hệ thống |
| XSS | `<script>alert(1)</script>@a.com` | `123456` | Không thực thi script |

---

## 9. Ma trận truy vết (Traceability)

| Yêu cầu | Use Case | Thông báo |
|---|---|---|
| FR-01 | UC-01 | – |
| FR-02 | UC-01 | MSG-01, 02, 03 |
| FR-03 | UC-01 | MSG-04, 05 |
| FR-04 | UC-01 | – |
| FR-05 | UC-02 | MSG-01, 03, 06 |
| FR-06 | – | – |
