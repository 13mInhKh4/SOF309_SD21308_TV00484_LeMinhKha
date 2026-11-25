ĐẶC TẢ NGHIỆP VỤ WEB CÁ NHÂN (SPECIFICATIONS)
> *Dùng làm ngữ cảnh (Context) gửi cho AI để sinh Test Plan & Test Case chính xác theo dự án.*
---
1. THÔNG TIN DỰ ÁN
Tên đề tài: Hệ thống quản lý chung cư Polybuilding (QLCC)
Công nghệ: Java 17, Jakarta Servlet 6 + JSP/JSTL, Hibernate 6 (JPA), SQL Server (database `QLCCNew2`), BCrypt, chạy trên Tomcat 10/11
Kiến trúc: Servlet (controller) → DAO → Entity (JPA); `AuthFilter` kiểm soát đăng nhập & phân quyền; view đặt trong `WEB-INF/views`
URL chạy local: `http://localhost:8080/chungcu` (context path tuỳ cấu hình Smart Tomcat/IDE; các URL bên dưới là đường dẫn sau context path)
Lưu ý khác với mẫu: hệ thống không có chức năng tự đăng ký. Tài khoản do Ban quản lý (BQL) cấp tại `/banquanly/tai-khoan`. Chức năng CRUD kiểm thử được chọn là Quản lý tài khoản, Phản ánh sự cố và Ghi chỉ số điện nước.
2. VAI TRÒ & ĐIỀU HƯỚNG SAU ĐĂNG NHẬP
Tab đăng nhập (`vaiTro`)	`vaiTro` / `boPhanCode` trong DB	Trang đích sau đăng nhập
`cudan` (Cư dân)	`CD`	`/cudan/thong-bao`
`banquanly` (Ban quản lý)	`BQL`	`/banquanly/binh-chon`
`letan` (Lễ tân)	`NV` + `LeTan` hoặc `LT`	`/letan/su-co`
`kythuat` (Kỹ thuật)	`NV` + `KyThuat` hoặc `NVKT`	`/kythuat/cong-viec`
`ketoan` (Kế toán)	`NV` + `KeToan` hoặc `KT`	`/ketoan/chi-so`
`baove` (Bảo vệ)	`NV` + `BaoVe` hoặc `BV`	`/baove/dashboard`
Tài khoản demo (mật khẩu `123456`): `cudan.p101`, `letan.thi`, `kythuat.nam`, `ketoan.lan`, `baove.hung`, `bql.admin`.
3. QUY TẮC NGHIỆP VỤ (BUSINESS RULES)
3.1 Đăng nhập — `GET /dang-nhap`, `POST /login` (AJAX, trả JSON)
Trường dữ liệu: `tenDangNhap`, `matKhau`, `vaiTro` (tab đang chọn). Form trên giao diện có `id="username"`, `id="password"`, nút `id="btnSubmitLogin"`, banner thông báo `id="errorBanner"`.
Validation (theo thứ tự kiểm tra của `LoginServlet`):
`tenDangNhap` hoặc `matKhau` rỗng / chỉ có khoảng trắng → `success=false`, "Vui lòng nhập đầy đủ tên đăng nhập và mật khẩu." (giao diện cũng chặn sẵn ở phía client).
`tenDangNhap` được `trim()`; không tồn tại → "Tên đăng nhập không tồn tại."
`trangThaiHoatDong = Khoa` → "Tài khoản của bạn đã bị khóa. Vui lòng liên hệ Ban quản lý."
Mật khẩu sai (xác thực BCrypt) → "Mật khẩu không chính xác."
Tab vai trò không khớp `vaiTro`/`boPhanCode` trong DB → "Tên đăng nhập hoặc mật khẩu không đúng!" (cố ý không tiết lộ lý do). Nếu không gửi `vaiTro` thì bỏ qua bước này.
Cư dân (`CD`) chưa gán căn hộ → "Tài khoản cư dân chưa được gán với căn hộ nào…"
Nhân viên (`NV`) chưa có hồ sơ nhân viên → "Tài khoản nhân viên chưa được gán thông tin Hồ sơ Nhân viên…"
Thành công: tạo session (`idTaiKhoan`, `tenDangNhap`, `vaiTro`, `boPhanCode`, `hoTen`, `maCuDan`/`maCanHo`/`maNhanVien` nếu có), trả `success=true` + `redirectUrl` theo bảng mục 2; giao diện tự chuyển trang sau ~1 giây.
Mật khẩu: lưu dạng BCrypt (`$2a$`, `$2b$`, `$2y$`, cost 12). Giá trị trong DB không đúng định dạng BCrypt → luôn xác thực thất bại. `PasswordUtil.hash` trả `null` nếu đầu vào `null`/rỗng; `PasswordUtil.verify` trả `false` nếu một trong hai tham số `null`/rỗng.
Đăng xuất: `GET /logout` huỷ session.
3.2 Phân quyền & bảo vệ truy cập — `AuthFilter` (áp dụng mọi URL trừ whitelist)
Whitelist (không cần đăng nhập): `/`, `/dang-nhap`, `/dang-nhap-the`, `/index.jsp`, `/login`, `/logout`, `/css/`, `/js/`, `/assets/`, `/favicon.ico`.
Chưa có session hợp lệ → chuyển hướng `/dang-nhap`.
Mỗi request được đối chiếu lại DB: tài khoản không tồn tại hoặc `trangThaiHoatDong ≠ HoatDong` → huỷ session, chuyển hướng `/dang-nhap` (khoá tài khoản có hiệu lực ngay cả khi đang đăng nhập).
Sai quyền → HTTP 403 (trang `403.jsp`):
`/banquanly/**` chỉ `BQL`; `/cudan/**` chỉ `CD`.
`/letan/**`, `/kythuat/**`, `/ketoan/**`, `/baove/**` chỉ `NV` với `boPhanCode` tương ứng (`LeTan`, `KyThuat`, `KeToan`, `BaoVe`).
`/nhanvien/**` cho `NV` hoặc `BQL`.
3.3 CRUD 1 — Quản lý tài khoản — `/banquanly/tai-khoan` (chỉ BQL)
Tạo (`action=create`): bắt buộc `tenDangNhap`, `matKhau`, `vaiTro`; `tenDangNhap` không được trùng (kiểm tra trước khi tạo, báo "đã tồn tại trong hệ thống"). `CD` → không có `boPhanCode`; `BQL` → `boPhanCode = BanQuanLy`; `NV` → dùng `boPhanCode` được gửi, mặc định `LeTan`. Với `CD` có thể gán `maCanHo`, `hoTen`, `soDienThoai`, `email`, `loaiCuDan`. Trạng thái mặc định `HoatDong`; mật khẩu được băm BCrypt.
Khoá / mở khoá (`action=toggle`): cần `tenDangNhap`; không được tự khoá chính tài khoản đang đăng nhập; không tìm thấy tài khoản → báo lỗi.
Reset mật khẩu (`action=resetPassword`): cần `tenDangNhap`; nếu không nhập `newPassword` thì mặc định `123456`; mật khẩu mới được băm BCrypt.
`action` không hợp lệ → "Action không hợp lệ."
3.4 CRUD 2 — Phản ánh sự cố của cư dân — `/cudan/phan-anh` (chỉ CD)
Gửi phản ánh mới: `tieuDe` và `moTa` bắt buộc (không rỗng sau `trim()`); `loaiSuCo` bắt buộc, thuộc `{Dien, Nuoc, ThangMay, PCCC, AnNinh, VeSinh, Khac}`; `mucDoUuTien` thuộc `{Cao, TrungBinh, Thap}` — để trống hoặc không hợp lệ thì tự dùng `TrungBinh`.
Ảnh đính kèm (tuỳ chọn, `anhTruocXuLy`): chỉ nhận đuôi `.jpg`, `.jpeg`, `.png` và MIME bắt đầu bằng `image/`; file lưu với tên duy nhất `suco_<timestamp>_<uuid6>.<ext>` tại `assets/uploads/suco/`.
Thành công → "Gửi phản ánh sự cố thành công!…"; lỗi → thông báo lỗi tương ứng, không lưu bản ghi.
Huỷ phản ánh: `/cudan/phan-anh/huy` (mã phản ánh không hợp lệ → báo lỗi). Xem chi tiết: `/cudan/phan-anh/detail?id=`.
3.5 CRUD 3 — Ghi chỉ số điện nước — `/ketoan/chi-so`, `/ketoan/chi-so/luu` (chỉ Kế toán)
Kỳ ghi theo `thang` (1–12) và `nam` (≥ 2000); tham số ngoài khoảng cho phép thì servlet dùng tháng/năm hiện tại. Chỉ liệt kê căn hộ có trạng thái `DangO`.
Quy tắc lưu cho từng căn hộ & loại dịch vụ (`Dien`/`Nuoc`): `thang` phải trong 1–12; chỉ số mới ≥ 0; chỉ số mới ≥ chỉ số kỳ trước (bằng nhau là hợp lệ). Ô để trống được bỏ qua; giá trị không phải số bị bỏ qua.
Bản ghi đã tồn tại trong cùng kỳ thì cập nhật thay vì tạo mới; lưu hàng loạt trong một giao dịch — một dòng lỗi thì rollback toàn bộ và báo "Căn hộ #<id> (Điện/Nước): <lý do>".
Danh sách rỗng → "Không có dữ liệu chỉ số để ghi nhận."
4. DỮ LIỆU & MÔI TRƯỜNG KIỂM THỬ
CSDL: chạy `QLCCNew2.sql` (kèm các patch trong `db/patches/`), chỉnh user/password SQL Server trong `persistence.xml`.
Cần có sẵn các tài khoản demo ở mục 2, ít nhất 1 tài khoản bị khoá, 1 tài khoản `CD` chưa gán căn hộ, và dữ liệu căn hộ `DangO`.
