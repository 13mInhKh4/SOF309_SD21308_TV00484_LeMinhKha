# ĐẶC TẢ NGHIỆP VỤ WEB CÁ NHÂN (SPECIFICATIONS)
> *Dùng làm ngữ cảnh (Context) gửi cho AI để sinh Test Plan & Test Case chính xác theo dự án.*

---

## 1. THÔNG TIN DỰ ÁN
* **Tên đề tài:** Hệ thống quản lý chung cư Polybuilding (QLCC)
* **Công nghệ:** Java 17, Jakarta Servlet 6 + JSP/JSTL, Hibernate 6 (JPA), SQL Server (database `QLCCNew2`), BCrypt, chạy trên Tomcat 10/11
* **Kiến trúc:** Servlet (controller) → DAO → Entity (JPA); `AuthFilter` kiểm soát đăng nhập & phân quyền; view đặt trong `WEB-INF/views`
* **URL chạy local:** `http://localhost:8080/chungcu` *(context path tuỳ cấu hình Smart Tomcat/IDE; các URL bên dưới là đường dẫn sau context path)*
* **Lưu ý khác với mẫu:** hệ thống **không có chức năng tự đăng ký**. Tài khoản do Ban quản lý (BQL) cấp tại `/banquanly/tai-khoan`. Chức năng CRUD kiểm thử được chọn là **Quản lý tài khoản**, **Phản ánh sự cố** và **Ghi chỉ số điện nước**.

## 2. VAI TRÒ & ĐIỀU HƯỚNG SAU ĐĂNG NHẬP

| Tab đăng nhập (`vaiTro`) | `vaiTro` / `boPhanCode` trong DB | Trang đích sau đăng nhập |
| :--- | :--- | :--- |
| `cudan` (Cư dân) | `CD` | `/cudan/thong-bao` |
| `banquanly` (Ban quản lý) | `BQL` | `/banquanly/binh-chon` |
| `letan` (Lễ tân) | `NV` + `LeTan` hoặc `LT` | `/letan/su-co` |
| `kythuat` (Kỹ thuật) | `NV` + `KyThuat` hoặc `NVKT` | `/kythuat/cong-viec` |
| `ketoan` (Kế toán) | `NV` + `KeToan` hoặc `KT` | `/ketoan/chi-so` |
| `baove` (Bảo vệ) | `NV` + `BaoVe` hoặc `BV` | `/baove/dashboard` |

Tài khoản demo (mật khẩu `123456`): `cudan.p101`, `letan.thi`, `kythuat.nam`, `ketoan.lan`, `baove.hung`, `bql.admin`.

## 3. QUY TẮC NGHIỆP VỤ (BUSINESS RULES)

### 3.1 Đăng nhập — `GET /dang-nhap`, `POST /login` (AJAX, trả JSON)
* **Trường dữ liệu:** `tenDangNhap`, `matKhau`, `vaiTro` (tab đang chọn). Form trên giao diện có `id="username"`, `id="password"`, nút `id="btnSubmitLogin"`, banner thông báo `id="errorBanner"`.
* **Validation (theo thứ tự kiểm tra của `LoginServlet`):**
  1. `tenDangNhap` hoặc `matKhau` rỗng / chỉ có khoảng trắng → `success=false`, "Vui lòng nhập đầy đủ tên đăng nhập và mật khẩu." (giao diện cũng chặn sẵn ở phía client).
  2. `tenDangNhap` được `trim()`; không tồn tại → "Tên đăng nhập không tồn tại."
  3. `trangThaiHoatDong = Khoa` → "Tài khoản của bạn đã bị khóa. Vui lòng liên hệ Ban quản lý."
  4. Mật khẩu sai (xác thực BCrypt) → "Mật khẩu không chính xác."
  5. Tab vai trò không khớp `vaiTro`/`boPhanCode` trong DB → "Tên đăng nhập hoặc mật khẩu không đúng!" (cố ý không tiết lộ lý do). Nếu không gửi `vaiTro` thì bỏ qua bước này.
  6. Cư dân (`CD`) chưa gán căn hộ → "Tài khoản cư dân chưa được gán với căn hộ nào…"
  7. Nhân viên (`NV`) chưa có hồ sơ nhân viên → "Tài khoản nhân viên chưa được gán thông tin Hồ sơ Nhân viên…"
* **Thành công:** tạo session (`idTaiKhoan`, `tenDangNhap`, `vaiTro`, `boPhanCode`, `hoTen`, `maCuDan`/`maCanHo`/`maNhanVien` nếu có), trả `success=true` + `redirectUrl` theo bảng mục 2; giao diện tự chuyển trang sau ~1 giây.
* **Mật khẩu:** lưu dạng BCrypt (`$2a$`, `$2b$`, `$2y$`, cost 12). Giá trị trong DB không đúng định dạng BCrypt → luôn xác thực thất bại. `PasswordUtil.hash` trả `null` nếu đầu vào `null`/rỗng; `PasswordUtil.verify` trả `false` nếu một trong hai tham số `null`/rỗng.
* **Đăng xuất:** `GET /logout` huỷ session.

### 3.2 Phân quyền & bảo vệ truy cập — `AuthFilter` (áp dụng mọi URL trừ whitelist)
* **Whitelist (không cần đăng nhập):** `/`, `/dang-nhap`, `/dang-nhap-the`, `/index.jsp`, `/login`, `/logout`, `/css/`, `/js/`, `/assets/`, `/favicon.ico`.
* Chưa có session hợp lệ → chuyển hướng `/dang-nhap`.
* Mỗi request được đối chiếu lại DB: tài khoản không tồn tại hoặc `trangThaiHoatDong ≠ HoatDong` → huỷ session, chuyển hướng `/dang-nhap` (khoá tài khoản có hiệu lực ngay cả khi đang đăng nhập).
* Sai quyền → HTTP **403** (trang `403.jsp`):
  * `/banquanly/**` chỉ `BQL`; `/cudan/**` chỉ `CD`.
  * `/letan/**`, `/kythuat/**`, `/ketoan/**`, `/baove/**` chỉ `NV` với `boPhanCode` tương ứng (`LeTan`, `KyThuat`, `KeToan`, `BaoVe`).
  * `/nhanvien/**` cho `NV` hoặc `BQL`.

### 3.3 CRUD 1 — Quản lý tài khoản — `/banquanly/tai-khoan` (chỉ BQL)
* **Tạo (`action=create`):** bắt buộc `tenDangNhap`, `matKhau`, `vaiTro`; `tenDangNhap` không được trùng (kiểm tra trước khi tạo, báo "đã tồn tại trong hệ thống"). `CD` → không có `boPhanCode`; `BQL` → `boPhanCode = BanQuanLy`; `NV` → dùng `boPhanCode` được gửi, mặc định `LeTan`. Ô chọn bộ phận trên form có 5 giá trị: `LeTan`, `KeToan`, `KyThuat`, `BaoVe`, `BanQuanLy`. Trạng thái mặc định `HoatDong`; mật khẩu được băm BCrypt; `maTaiKhoan` tự sinh dạng `cd_0001`, `nv_lt0001`, `bql_0001`…
  * **Cư dân (`CD`) có chọn căn hộ:** tạo thêm hồ sơ cư dân (`hoTen` mặc định "Cư dân <tên đăng nhập>"; `loaiCuDan` mặc định `ChuHo`, nếu căn hộ đã có chủ hộ thì thành `KhachThue`) và đổi trạng thái căn hộ thành `DangO`.
  * **Cư dân (`CD`) không chọn căn hộ:** vẫn tạo tài khoản nhưng **không có hồ sơ cư dân** → tài khoản này đăng nhập sẽ bị báo "chưa được gán với căn hộ nào".
  * **`NV`/`BQL`:** tạo thêm hồ sơ nhân viên (`hoTen` mặc định = tên đăng nhập).
* **Khoá / mở khoá (`action=toggle`):** cần `tenDangNhap`; **không được tự khoá chính tài khoản đang đăng nhập**; không tìm thấy tài khoản → báo lỗi.
* **Reset mật khẩu (`action=resetPassword`):** cần `tenDangNhap`; nếu không nhập `newPassword` thì mặc định `123456`; mật khẩu mới được băm BCrypt.
* `action` không hợp lệ → "Action không hợp lệ."

### 3.4 CRUD 2 — Phản ánh sự cố của cư dân — `/cudan/phan-anh` (chỉ CD)
* **Gửi phản ánh mới:** `tieuDe` và `moTa` bắt buộc (không rỗng sau `trim()`); `loaiSuCo` bắt buộc, thuộc `{Dien, Nuoc, ThangMay, PCCC, AnNinh, VeSinh, Khac}`; `mucDoUuTien` thuộc `{Cao, TrungBinh, Thap}` — để trống hoặc không hợp lệ thì tự dùng `TrungBinh`.
* **Ảnh đính kèm (tuỳ chọn, `anhTruocXuLy`):** chỉ nhận đuôi `.jpg`, `.jpeg`, `.png` **và** MIME bắt đầu bằng `image/`; file lưu với tên duy nhất `suco_<timestamp>_<uuid6>.<ext>` tại `assets/uploads/suco/`.
* Thành công → "Gửi phản ánh sự cố thành công!…"; lỗi → thông báo lỗi tương ứng, không lưu bản ghi.
* **Trạng thái ban đầu** của phản ánh mới là `MoiTiepNhan`, nguồn gửi `CuDan`.
* **Huỷ phản ánh:** `POST /cudan/phan-anh/huy` (chỉ nhận POST). Chỉ huỷ được khi phản ánh còn `MoiTiepNhan` và thuộc căn hộ của cư dân đang đăng nhập; thành công → trạng thái `Huy`, thêm dòng lịch sử, thông báo "Đã hủy phản ánh sự cố thành công.". Các lỗi: không tồn tại → "Phản ánh không tồn tại."; khác căn hộ → "Bạn không có quyền hủy phản ánh của căn hộ khác."; đã tiếp nhận/đang xử lý → "Phản ánh đã được tiếp nhận hoặc đang xử lý, không thể hủy."; `id` không phải số → "Mã phản ánh không hợp lệ: …".
* **Xem chi tiết:** `GET /cudan/phan-anh/detail?id=` (phản ánh của căn hộ khác hoặc không tồn tại → báo lỗi, quay về danh sách).

### 3.5 CRUD 3 — Ghi chỉ số điện nước — `/ketoan/chi-so`, `/ketoan/chi-so/luu` (chỉ Kế toán)
* Kỳ ghi theo `thang` (1–12) và `nam` (≥ 2000); tham số ngoài khoảng cho phép thì servlet dùng tháng/năm hiện tại. Chỉ liệt kê căn hộ có trạng thái `DangO`.
* **Quy tắc lưu cho từng căn hộ & loại dịch vụ (`Dien`/`Nuoc`):** `thang` phải trong 1–12; chỉ số mới **≥ 0**; chỉ số mới **≥ chỉ số kỳ trước** (bằng nhau là hợp lệ). Ô để trống được bỏ qua; nếu một ô của căn hộ có giá trị không phải số thì cả dòng căn hộ đó (cả điện lẫn nước) bị bỏ qua.
* Bản ghi đã tồn tại trong cùng kỳ thì **cập nhật** thay vì tạo mới; lưu hàng loạt trong **một giao dịch** — một dòng lỗi thì rollback toàn bộ và báo "Căn hộ #<id> (Điện/Nước): <lý do>".
* Chỉ khi **không có dòng căn hộ nào** được gửi lên (danh sách rỗng) mới báo "Không có dữ liệu chỉ số để ghi nhận."; nếu có dòng nhưng mọi ô trống thì không lưu gì và vẫn báo thành công.

## 4. DỮ LIỆU & MÔI TRƯỜNG KIỂM THỬ
* CSDL: chạy `QLCCNew2.sql` (kèm các patch trong `db/patches/`), chỉnh user/password SQL Server trong `persistence.xml`.
* Dữ liệu seed đã có 6 tài khoản demo ở mục 2 (mật khẩu `123456`, đã kiểm chứng với hash BCrypt trong `QLCCNew2.sql`). Seed **không có** tài khoản bị khoá, `CD` chưa gán căn hộ, hay `NV` thuộc `BanQuanLy` → phải tự tạo qua `/banquanly/tai-khoan` trước khi chạy các ca tương ứng.
