# Software Requirements Specification (SRS)
## Module: Đăng nhập (Login / Authentication)
## Hệ thống: Perfex CRM – Demo (crm.anhtester.com)

| Thuộc tính | Giá trị |
|---|---|
| Phiên bản tài liệu | 1.0 |
| Ngày tạo | 2026-10-29 |
| Đối tượng | Perfex CRM – Anh Tester Demo |
| URL khảo sát | https://crm.anhtester.com/authentication/login |
| Người soạn | Được sinh tự động từ khảo sát UI thực tế (Claude) |

> Tài liệu này được xây dựng bằng cách khảo sát trực tiếp giao diện đăng nhập tại địa chỉ nêu trên (các trường, thông báo validate, luồng điều hướng) kết hợp với các yêu cầu nghiệp vụ chuẩn của một module đăng nhập CRM. Một số hành vi phía server (nội dung thông báo lỗi sai email/mật khẩu, cơ chế khoá tài khoản...) được suy luận dựa trên hành vi quan sát được và kiến trúc phổ biến của Perfex CRM; cần đối chiếu lại với đội phát triển/BA trước khi dùng làm tài liệu chính thức.

---

## 1. Giới thiệu

### 1.1 Mục đích
Tài liệu này mô tả các yêu cầu chức năng và phi chức năng của **module Đăng nhập (Login)** trong hệ thống CRM, phục vụ cho đội phát triển, kiểm thử (QA) và các bên liên quan hiểu rõ phạm vi, hành vi mong đợi của màn hình đăng nhập.

### 1.2 Phạm vi
Module Đăng nhập cho phép người dùng đã có tài khoản (Admin/Staff/Customer) xác thực danh tính bằng **Email/Password** để truy cập vào hệ thống CRM. Phạm vi tài liệu bao gồm:
- Màn hình đăng nhập (Login form)
- Chức năng "Ghi nhớ đăng nhập" (Remember me)
- Chức năng "Quên mật khẩu" (Forgot Password) – liên quan trực tiếp đến luồng đăng nhập
- Đa ngôn ngữ (Language switcher) trên màn hình đăng nhập

Không thuộc phạm vi: đăng ký tài khoản mới, phân quyền sau đăng nhập, xác thực 2 yếu tố (2FA) nếu không được bật trên môi trường khảo sát.

### 1.3 Đối tượng sử dụng
- Admin / Staff nội bộ
- Khách hàng (Customer) – nếu hệ thống cấu hình cổng khách hàng dùng chung trang đăng nhập

### 1.4 Định nghĩa, thuật ngữ viết tắt
| Thuật ngữ | Giải thích |
|---|---|
| SRS | Software Requirements Specification |
| CRM | Customer Relationship Management |
| CSRF | Cross-Site Request Forgery |
| 2FA | Two-Factor Authentication |

---

## 2. Mô tả tổng quan

### 2.1 Bối cảnh sản phẩm
Màn hình đăng nhập là điểm vào (entry point) duy nhất để người dùng truy cập các chức năng quản trị/khai thác dữ liệu của CRM. Giao diện được khảo sát gồm các thành phần:

- Logo hệ thống + liên kết về trang chủ
- Menu điều hướng: **Knowledge Base**, **Login**
- Tiêu đề **"Please login"**
- Dropdown chọn **Language** (hỗ trợ 26 ngôn ngữ: English, Chinese, Vietnamese, Indonesia, Bulgarian, Portuguese_br, Swedish, Dutch, Persian, Turkish, Catalan, Spanish, Portuguese, French, Czech, Japanese, Italian, Slovak, Russian, Polish, Greek, Finnish, Norwegian, German, Ukrainian, Romanian)
- Trường nhập **Email Address**
- Trường nhập **Password** (ẩn ký tự)
- Checkbox **Remember me**
- Nút **Login** (submit)
- Liên kết **Forgot Password?**
- Footer bản quyền

### 2.2 Chức năng chính
1. Xác thực người dùng bằng email + mật khẩu
2. Duy trì phiên đăng nhập qua checkbox "Remember me"
3. Chuyển hướng tới trang khôi phục mật khẩu
4. Đổi ngôn ngữ hiển thị form đăng nhập
5. Hiển thị thông báo lỗi validate khi dữ liệu nhập không hợp lệ

### 2.3 Ràng buộc chung
- Giao tiếp giữa trình duyệt và server phải qua HTTPS.
- Form đăng nhập sử dụng token ẩn (hidden input) – khảo sát cho thấy có một trường `type="hidden"` trong form, phù hợp với cơ chế chống CSRF.

---

## 3. Yêu cầu chức năng (Functional Requirements)

### FR-01: Hiển thị form đăng nhập
**Mô tả:** Khi người dùng truy cập `/authentication/login`, hệ thống hiển thị form gồm: Language, Email Address, Password, Remember me, nút Login, liên kết Forgot Password.
**Đầu vào:** Truy cập URL đăng nhập.
**Đầu ra:** Form đăng nhập hiển thị đầy đủ các thành phần, trường Email và Password trống, Remember me chưa chọn, ngôn ngữ mặc định English.
**Độ ưu tiên:** Cao.

### FR-02: Validate dữ liệu đầu vào bắt buộc (client-side)
**Mô tả:** Khi người dùng bấm **Login** mà chưa nhập Email và/hoặc Password, hệ thống hiển thị thông báo lỗi ngay dưới từng trường, không gửi request lên server.
**Kết quả quan sát thực tế:**
- Bỏ trống Email → hiển thị: `The Email Address field is required.`
- Bỏ trống Password → hiển thị: `The Password field is required.`
**Độ ưu tiên:** Cao.

### FR-03: Xác thực tài khoản (server-side)
**Mô tả:** Khi người dùng nhập đầy đủ Email + Password hợp lệ về định dạng và bấm Login, hệ thống gửi yêu cầu xác thực lên server.
- **Trường hợp đúng thông tin:** Hệ thống tạo phiên đăng nhập (session) và chuyển hướng người dùng vào trang chủ/dashboard tương ứng với vai trò tài khoản (Admin/Staff vào khu vực quản trị, Customer vào Customer Portal).
- **Trường hợp sai thông tin (email không tồn tại hoặc sai mật khẩu):** Hệ thống **không** tiết lộ cụ thể email hay mật khẩu sai (tránh dò tài khoản), trả người dùng về lại trang đăng nhập, xoá sạch trường mật khẩu đã nhập, kèm thông báo lỗi chung dạng "Thông tin đăng nhập không chính xác".
**Độ ưu tiên:** Cao.

> Ghi chú kiểm thử: trong lần khảo sát với cặp thông tin không hợp lệ và với cặp `admin@example.com` / `123456`, hệ thống đều điều hướng về lại trang Login với các trường trống; không quan sát được rõ nội dung thông báo lỗi do có thể hiển thị dạng toast/flash message tự ẩn sau khi tải lại trang. Cần xác nhận lại nội dung thông báo chính xác và việc đăng nhập `admin@example.com/123456` có thành công hay không với đội phát triển/BA trước khi chốt tài liệu.

### FR-04: Ghi nhớ đăng nhập (Remember me)
**Mô tả:** Khi người dùng tick chọn **Remember me** trước khi đăng nhập thành công, hệ thống lưu một cookie/token có thời hạn dài hơn phiên mặc định để tự động đăng nhập ở lần truy cập sau trên cùng thiết bị/trình duyệt.
**Độ ưu tiên:** Trung bình.

### FR-05: Quên mật khẩu (Forgot Password)
**Mô tả:** Người dùng bấm liên kết **"Forgot Password?"** để được điều hướng tới trang `/authentication/forgot_password`, nơi chỉ yêu cầu nhập **Email Address** và bấm **Submit** để gửi yêu cầu đặt lại mật khẩu.
**Đầu ra mong đợi:** Hệ thống gửi email chứa liên kết đặt lại mật khẩu đến địa chỉ email nếu tài khoản tồn tại; hiển thị thông báo xác nhận chung (không tiết lộ email có tồn tại hay không).
**Độ ưu tiên:** Trung bình.

### FR-06: Đổi ngôn ngữ hiển thị
**Mô tả:** Người dùng chọn một ngôn ngữ trong dropdown **Language** (26 lựa chọn) để thay đổi ngôn ngữ hiển thị toàn bộ nhãn/label trên form đăng nhập.
**Độ ưu tiên:** Thấp.

### FR-07: Điều hướng sau đăng nhập theo vai trò
**Mô tả:** Sau khi xác thực thành công, hệ thống chuyển hướng người dùng đến giao diện tương ứng với vai trò (role) được gán cho tài khoản: khu vực quản trị cho Admin/Staff, Customer Portal cho Customer.
**Độ ưu tiên:** Cao.

### FR-08: Chặn truy cập trái phép vào khu vực yêu cầu đăng nhập
**Mô tả:** Khi người dùng chưa đăng nhập cố truy cập trực tiếp một URL yêu cầu xác thực, hệ thống tự động chuyển hướng về trang `/authentication/login`.
**Độ ưu tiên:** Cao.

---

## 4. Yêu cầu phi chức năng (Non-Functional Requirements)

### NFR-01: Bảo mật
- Mật khẩu phải được **hash** (vd: bcrypt) khi lưu trữ, không lưu plaintext.
- Form đăng nhập phải có cơ chế chống **CSRF** (quan sát thấy trường hidden input trong form – phù hợp với token CSRF).
- Truyền dữ liệu đăng nhập qua **HTTPS**.
- Nên áp dụng **giới hạn số lần đăng nhập sai** (rate limiting / account lockout tạm thời hoặc captcha) để chống tấn công dò mật khẩu (brute-force).
- Không hiển thị thông tin giúp kẻ tấn công phân biệt "email không tồn tại" và "sai mật khẩu" (tránh user enumeration).
- Cookie phiên đăng nhập cần có thuộc tính `HttpOnly`, `Secure`, `SameSite`.

### NFR-02: Hiệu năng
- Thời gian phản hồi cho một lượt đăng nhập (từ lúc submit đến khi điều hướng) nên ≤ 2 giây trong điều kiện mạng bình thường.

### NFR-03: Khả dụng & giao diện
- Form đăng nhập cần hiển thị đúng trên các kích thước màn hình phổ biến (responsive: desktop, tablet, mobile).
- Thông báo lỗi phải hiển thị rõ ràng, gần với trường dữ liệu tương ứng (đã quan sát: lỗi hiển thị ngay dưới từng input).

### NFR-04: Đa ngôn ngữ (i18n)
- Toàn bộ nhãn, thông báo lỗi trên form đăng nhập phải được dịch đầy đủ khi người dùng đổi ngôn ngữ, không để sót text gốc (tiếng Anh) khi đã chọn ngôn ngữ khác.

### NFR-05: Khả năng kiểm thử & ghi log
- Hệ thống nên ghi log các lần đăng nhập thất bại (không ghi mật khẩu) phục vụ giám sát bảo mật.

---

## 5. Đặc tả chi tiết các trường dữ liệu (Field Specification)

| Trường | Loại | Bắt buộc | Validate quan sát được | Ghi chú |
|---|---|---|---|---|
| Language | Dropdown (select) | Có (mặc định English) | — | 26 lựa chọn ngôn ngữ |
| Email Address | Text input | Có | "The Email Address field is required." khi để trống | Cần bổ sung validate định dạng email (chưa quan sát được thông báo riêng cho email sai định dạng) |
| Password | Password input (ẩn ký tự) | Có | "The Password field is required." khi để trống | Không giới hạn hiển thị độ dài tối thiểu trên UI khi khảo sát |
| Remember me | Checkbox | Không | — | Mặc định chưa chọn |
| Login | Button (submit) | — | — | Submit form |
| Forgot Password? | Link | — | — | Điều hướng sang `/authentication/forgot_password` |

---

## 6. Luồng nghiệp vụ (Use Case)

### UC-01: Đăng nhập thành công
1. Người dùng truy cập trang Login.
2. Nhập Email hợp lệ đã đăng ký và Password đúng.
3. (Tuỳ chọn) Tick "Remember me".
4. Bấm **Login**.
5. Hệ thống xác thực, tạo phiên làm việc.
6. Hệ thống chuyển hướng vào trang chủ/dashboard theo vai trò tài khoản.

### UC-02: Đăng nhập thất bại – sai thông tin
1. Người dùng nhập Email/Password không đúng.
2. Bấm **Login**.
3. Hệ thống không tạo phiên, trả về lại trang Login.
4. Hệ thống hiển thị thông báo lỗi chung, xoá nội dung trường Password.

### UC-03: Bỏ trống trường bắt buộc
1. Người dùng bấm **Login** khi chưa nhập Email và/hoặc Password.
2. Hệ thống chặn submit, hiển thị thông báo lỗi ngay dưới trường tương ứng (xử lý phía client).

### UC-04: Quên mật khẩu
1. Người dùng bấm **"Forgot Password?"**.
2. Hệ thống chuyển sang trang nhập Email để khôi phục mật khẩu.
3. Người dùng nhập Email, bấm **Submit**.
4. Hệ thống gửi email hướng dẫn đặt lại mật khẩu (nếu email tồn tại trong hệ thống).

---

## 7. Yêu cầu chưa xác nhận / Cần làm rõ thêm (Open Items)
- Nội dung chính xác của thông báo lỗi khi sai Email/Password (không quan sát rõ do giới hạn khảo sát tự động).
- Có áp dụng cơ chế khoá tài khoản/captcha sau N lần đăng nhập sai hay không.
- Có hỗ trợ 2FA/OTP hay không.
- Thời hạn cụ thể của cookie "Remember me".
- Quy tắc độ mạnh mật khẩu tối thiểu khi tạo/đổi mật khẩu (ngoài phạm vi màn hình Login nhưng liên quan).

---

*Tài liệu được sinh dựa trên khảo sát thực tế giao diện tại https://crm.anhtester.com/authentication/login ngày 2026-10-29. Cần được đội BA/Dev rà soát và xác nhận lại trước khi sử dụng chính thức.*
