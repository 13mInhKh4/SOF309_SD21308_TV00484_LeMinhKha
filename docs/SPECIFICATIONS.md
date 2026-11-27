# ĐẶC TẢ NGHIỆP VỤ WEB CÁ NHÂN (SPECIFICATIONS)

> *Dùng làm ngữ cảnh (Context) gửi cho AI để sinh Test Plan & Test Case chính xác theo dự án.*

---

## 1. THÔNG TIN DỰ ÁN

| Hạng mục | Nội dung |
| :--- | :--- |
| **Tên đề tài** | Hệ thống quản lý chung cư Polybuilding (QLCC) |
| **Công nghệ** | Java 17, Jakarta Servlet 6 + JSP/JSTL, Hibernate 6 (JPA), SQL Server (database `QLCCNew2`), BCrypt, Tomcat 10/11 |
| **Kiến trúc** | Servlet (controller) → DAO → Entity (JPA); `AuthFilter` kiểm soát đăng nhập & phân quyền; view đặt trong `WEB-INF/views` |
| **URL chạy local** | `http://localhost:8080/chungcu` (context path tuỳ cấu hình Smart Tomcat/IDE; các URL bên dưới là đường dẫn sau context path) |
| **Khác với mẫu** | Hệ thống không có chức năng tự đăng ký. Tài khoản do Ban quản lý (BQL) cấp tại `/banquanly/tai-khoan`. Chức năng CRUD kiểm thử gồm: Quản lý tài khoản, Phản ánh sự cố, Ghi chỉ số điện nước. |

## 2. VAI TRÒ & ĐIỀU HƯỚNG SAU ĐĂNG NHẬP

| Tab đăng nhập (`vaiTro`) | `vaiTro` / `boPhanCode` trong DB | Trang đích sau đăng nhập | Tài khoản demo |
| :--- | :--- | :--- | :--- |
| `cudan` (Cư dân) | `CD` | `/cudan/thong-bao` | `cudan.p101` |
| `banquanly` (Ban quản lý) | `BQL` | `/banquanly/binh-chon` | `bql.admin` |
| `letan` (Lễ tân) | `NV` + `LeTan` hoặc `LT` | `/letan/su-co` | `letan.thi` |
| `kythuat` (Kỹ thuật) | `NV` + `KyThuat` hoặc `NVKT` | `/kythuat/cong-viec` | `kythuat.nam` |
| `ketoan` (Kế toán) | `NV` + `KeToan` hoặc `KT` | `/ketoan/chi-so` | `ketoan.lan` |
| `baove` (Bảo vệ) | `NV` + `BaoVe` hoặc `BV` | `/baove/dashboard` | `baove.hung` |

Mật khẩu của tất cả tài khoản demo: `123456`.

## 3. QUY TẮC NGHIỆP VỤ (BUSINESS RULES)

### 3.1 Đăng nhập: `GET /dang-nhap`, `POST /login` (AJAX, trả JSON)

| Hạng mục | Quy tắc |
| :--- | :--- |
| Trường dữ liệu | `tenDangNhap`, `matKhau`, `vaiTro` (tab đang chọn). Giao diện: ô `id="username"`, ô `id="password"`, nút `id="btnSubmitLogin"`, banner thông báo `id="errorBanner"` |
| Bước 1 | `tenDangNhap` hoặc `matKhau` rỗng / chỉ khoảng trắng → "Vui lòng nhập đầy đủ tên đăng nhập và mật khẩu." (giao diện cũng chặn sẵn ở phía client) |
| Bước 2 | `tenDangNhap` được `trim()`; không tồn tại → "Tên đăng nhập không tồn tại." |
| Bước 3 | `trangThaiHoatDong = Khoa` → "Tài khoản của bạn đã bị khóa. Vui lòng liên hệ Ban quản lý." |
| Bước 4 | Sai mật khẩu (xác thực BCrypt) → "Mật khẩu không chính xác." |
| Bước 5 | Tab vai trò không khớp `vaiTro`/`boPhanCode` trong DB → "Tên đăng nhập hoặc mật khẩu không đúng!" (cố ý không lộ lý do). Nếu không gửi `vaiTro` thì bỏ qua bước này |
| Bước 6 | Cư dân (`CD`) chưa gán căn hộ → "Tài khoản cư dân chưa được gán với căn hộ nào…" |
| Bước 7 | Nhân viên (`NV`) chưa có hồ sơ nhân viên → "Tài khoản nhân viên chưa được gán thông tin Hồ sơ Nhân viên…" |
| Thành công | Tạo session (`idTaiKhoan`, `tenDangNhap`, `vaiTro`, `boPhanCode`, `hoTen`, và `maCuDan`/`maCanHo`/`maNhanVien` nếu có); trả `success=true` + `redirectUrl` theo bảng mục 2; giao diện chuyển trang sau khoảng 1 giây |
| Mật khẩu | Lưu BCrypt (`$2a$`, `$2b$`, `$2y$`, cost 12). Giá trị trong DB không đúng định dạng BCrypt → luôn xác thực thất bại |
| `PasswordUtil` | `hash` trả `null` nếu đầu vào `null`/rỗng; `verify` trả `false` nếu một trong hai tham số `null`/rỗng |
| Đăng xuất | `GET /logout` huỷ session |

### 3.2 Phân quyền & bảo vệ truy cập (`AuthFilter`)

| Hạng mục | Quy tắc |
| :--- | :--- |
| Whitelist (không cần đăng nhập) | `/`, `/dang-nhap`, `/dang-nhap-the`, `/index.jsp`, `/login`, `/logout`, `/css/`, `/js/`, `/assets/`, `/favicon.ico` |
| Chưa có session hợp lệ | Chuyển hướng `/dang-nhap` |
| Tài khoản bị khoá khi đang đăng nhập | Mỗi request được đối chiếu lại DB; tài khoản không tồn tại hoặc `trangThaiHoatDong` khác `HoatDong` → huỷ session, chuyển hướng `/dang-nhap` |
| `/banquanly/**` | Chỉ `BQL`, sai quyền → HTTP 403 (`403.jsp`) |
| `/cudan/**` | Chỉ `CD`, sai quyền → HTTP 403 |
| `/letan/**`, `/kythuat/**`, `/ketoan/**`, `/baove/**` | Chỉ `NV` với `boPhanCode` tương ứng (`LeTan`, `KyThuat`, `KeToan`, `BaoVe`), sai quyền → HTTP 403 |
| `/nhanvien/**` | Cho `NV` hoặc `BQL` |

### 3.3 CRUD 1: Quản lý tài khoản (`/banquanly/tai-khoan`, chỉ BQL)

| Hành động | Quy tắc |
| :--- | :--- |
| Tạo (`action=create`) | Bắt buộc `tenDangNhap`, `matKhau`, `vaiTro`; `tenDangNhap` không được trùng ("đã tồn tại trong hệ thống"). Trạng thái mặc định `HoatDong`; mật khẩu băm BCrypt |
| Tạo: bộ phận | `CD` → không có `boPhanCode`; `BQL` → `BanQuanLy`; `NV` → dùng `boPhanCode` được gửi, mặc định `LeTan` |
| Tạo: hồ sơ cư dân | Với `CD` có thể gán `maCanHo`, `hoTen`, `soDienThoai`, `email`, `loaiCuDan` |
| Khoá / mở khoá (`action=toggle`) | Cần `tenDangNhap`; không được tự khoá tài khoản đang đăng nhập; không tìm thấy tài khoản → báo lỗi |
| Reset mật khẩu (`action=resetPassword`) | Cần `tenDangNhap`; không nhập `newPassword` thì mặc định `123456`; mật khẩu mới băm BCrypt |
| Action khác | "Action không hợp lệ." |

### 3.4 CRUD 2: Phản ánh sự cố của cư dân (`/cudan/phan-anh`, chỉ CD)

| Hạng mục | Quy tắc |
| :--- | :--- |
| Trường bắt buộc | `tieuDe`, `moTa` không rỗng sau `trim()` |
| `loaiSuCo` | Bắt buộc, thuộc `Dien`, `Nuoc`, `ThangMay`, `PCCC`, `AnNinh`, `VeSinh`, `Khac` |
| `mucDoUuTien` | Thuộc `Cao`, `TrungBinh`, `Thap`; để trống hoặc không hợp lệ thì tự dùng `TrungBinh` |
| Ảnh đính kèm (`anhTruocXuLy`, tuỳ chọn) | Chỉ nhận đuôi `.jpg`, `.jpeg`, `.png` và MIME bắt đầu bằng `image/` |
| Lưu ảnh | Tên duy nhất `suco_<timestamp>_<uuid6>.<ext>` tại `assets/uploads/suco/` |
| Kết quả | Thành công → "Gửi phản ánh sự cố thành công!…"; lỗi → thông báo tương ứng, không lưu bản ghi |
| Huỷ / xem chi tiết | Huỷ: `/cudan/phan-anh/huy` (mã không hợp lệ → báo lỗi). Chi tiết: `/cudan/phan-anh/detail?id=` |

### 3.5 CRUD 3: Ghi chỉ số điện nước (`/ketoan/chi-so`, `/ketoan/chi-so/luu`, chỉ Kế toán)

| Hạng mục | Quy tắc |
| :--- | :--- |
| Kỳ ghi | `thang` từ 1 đến 12, `nam` từ 2000 trở lên; tham số ngoài khoảng thì servlet dùng tháng/năm hiện tại. Chỉ liệt kê căn hộ trạng thái `DangO` |
| Tháng | `thang` phải trong 1 đến 12, nếu không: "Tháng phải nằm trong khoảng từ 1 đến 12." |
| Chỉ số mới | Phải từ 0 trở lên, nếu không: "Chỉ số mới không được nhỏ hơn 0." |
| So với kỳ trước | Chỉ số mới phải lớn hơn hoặc bằng chỉ số kỳ trước (bằng nhau là hợp lệ), nếu không: "Chỉ số mới (x) nhỏ hơn chỉ số kỳ trước (y)" |
| Ô trống / không phải số | Ô để trống bị bỏ qua; giá trị không phải số bị bỏ qua |
| Cập nhật | Bản ghi đã có trong cùng căn hộ/kỳ/loại thì cập nhật, không tạo mới |
| Giao dịch | Lưu hàng loạt trong một giao dịch; một dòng lỗi thì rollback toàn bộ và báo "Căn hộ #<id> (Điện/Nước): <lý do>" |
| Danh sách rỗng | "Không có dữ liệu chỉ số để ghi nhận." |

## 4. DỮ LIỆU & MÔI TRƯỜNG KIỂM THỬ

| Hạng mục | Nội dung |
| :--- | :--- |
| CSDL | Chạy `QLCCNew2.sql` (kèm các patch trong `db/patches/`), chỉnh user/password SQL Server trong `persistence.xml` |
| Dữ liệu cần có | Các tài khoản demo ở mục 2; ít nhất 1 tài khoản bị khoá; 1 tài khoản `CD` chưa gán căn hộ; dữ liệu căn hộ `DangO` |
