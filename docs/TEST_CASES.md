# BỘ KỊCH BẢN KIỂM THỬ (TEST CASES)

> **Quy ước:** `[UI]` = kiểm thử giao diện bằng Selenium; `[UT]` = unit test JUnit 5 (không cần DB). Cột *Trạng thái* ghi **Chưa chạy** cho tới khi bạn thực thi và cập nhật thành Pass/Fail. URL ghi theo đường dẫn sau context path (xem `SPECIFICATIONS.md`). Mật khẩu tài khoản demo: `123456`.
> Kỹ thuật: EP = phân vùng tương đương, BVA = giá trị biên.

---

## 1. ĐĂNG NHẬP (AUTH)

| Test Case ID | Chức năng | Mô tả ca kiểm thử | Loại test (Pos/Neg) | Các bước thực hiện | Dữ liệu kiểm thử (Test Data) | Kết quả mong đợi | Trạng thái (Pass/Fail) |
| :---: | :---: | :--- | :---: | :--- | :--- | :--- | :---: |
| **TC_AUTH_01** | Login | [UI] Cư dân đăng nhập thành công | Positive | 1. Mở `/dang-nhap` (tab Cư dân)<br>2. Nhập username, password<br>3. Bấm Đăng nhập | `cudan.p101` / `123456` | Banner "Đăng nhập thành công!"; sau ~1s chuyển tới `/cudan/thong-bao` | Chưa chạy |
| **TC_AUTH_02** | Login | [UI] Đăng nhập thất bại khi để trống mật khẩu | Negative | 1. Nhập username<br>2. Bỏ trống password<br>3. Bấm Đăng nhập | `cudan.p101` / `""` | Banner "Vui lòng nhập đầy đủ tên đăng nhập và mật khẩu."; không chuyển trang | Chưa chạy |
| **TC_AUTH_03** | Login | [UI] Đăng nhập thất bại khi để trống tên đăng nhập | Negative | 1. Bỏ trống username<br>2. Nhập password<br>3. Bấm Đăng nhập | `""` / `123456` | Banner "Vui lòng nhập đầy đủ tên đăng nhập và mật khẩu." | Chưa chạy |
| **TC_AUTH_04** | Login | [UI] BVA: tên đăng nhập và mật khẩu chỉ có khoảng trắng | Negative | 1. Nhập chuỗi khoảng trắng vào cả hai ô<br>2. Bấm Đăng nhập | `"   "` / `"   "` | Banner báo nhập đầy đủ thông tin (giá trị bị `trim()` thành rỗng) | Chưa chạy |
| **TC_AUTH_05** | Login | [UI] BVA: tên đăng nhập có khoảng trắng đầu/cuối vẫn đăng nhập được | Positive | 1. Nhập username kèm khoảng trắng hai đầu<br>2. Nhập password đúng<br>3. Bấm Đăng nhập | `"  cudan.p101  "` / `123456` | Đăng nhập thành công (tên đăng nhập được `trim()`) | Chưa chạy |
| **TC_AUTH_06** | Login | [UI] Tên đăng nhập không tồn tại | Negative | 1. Nhập username không có trong DB<br>2. Nhập password bất kỳ<br>3. Bấm Đăng nhập | `khongtontai` / `123456` | Banner "Tên đăng nhập không tồn tại." | Chưa chạy |
| **TC_AUTH_07** | Login | [UI] Sai mật khẩu | Negative | 1. Nhập username đúng<br>2. Nhập mật khẩu sai<br>3. Bấm Đăng nhập | `cudan.p101` / `saiMatKhau` | Banner "Mật khẩu không chính xác."; không tạo session | Chưa chạy |
| **TC_AUTH_08** | Login | [UI] Tài khoản bị khoá không đăng nhập được | Negative | 1. Dùng tài khoản có `trangThaiHoatDong = Khoa`<br>2. Nhập đúng mật khẩu<br>3. Bấm Đăng nhập | `<tài khoản bị khoá>` / `123456` | Banner "Tài khoản của bạn đã bị khóa. Vui lòng liên hệ Ban quản lý." | Chưa chạy |
| **TC_AUTH_09** | Login | [UI] Chọn sai tab vai trò | Negative | 1. Ở tab Lễ tân<br>2. Nhập tài khoản cư dân đúng mật khẩu<br>3. Bấm Đăng nhập | `cudan.p101` / `123456` (tab `letan`) | Banner "Tên đăng nhập hoặc mật khẩu không đúng!" (không lộ lý do) | Chưa chạy |
| **TC_AUTH_10** | Login | [UI] Cư dân chưa gán căn hộ | Negative | 1. Dùng tài khoản `CD` chưa có căn hộ<br>2. Đăng nhập tab Cư dân | `<CD chưa gán căn hộ>` / `123456` | Banner "Tài khoản cư dân chưa được gán với căn hộ nào…" | Chưa chạy |
| **TC_AUTH_11** | Login | [UI] Nhân viên chưa có hồ sơ nhân viên | Negative | 1. Dùng tài khoản `NV` chưa có hồ sơ<br>2. Đăng nhập đúng tab bộ phận | `<NV chưa có hồ sơ>` / `123456` | Banner "Tài khoản nhân viên chưa được gán thông tin Hồ sơ Nhân viên…" | Chưa chạy |
| **TC_AUTH_12** | Login | [UI][Data-driven] Mỗi vai trò chuyển đúng trang đích | Positive | 1. Chọn tab vai trò<br>2. Đăng nhập tài khoản tương ứng<br>3. Kiểm tra URL | `letan.thi` → `/letan/su-co`<br>`kythuat.nam` → `/kythuat/cong-viec`<br>`ketoan.lan` → `/ketoan/chi-so`<br>`baove.hung` → `/baove/dashboard`<br>`bql.admin` → `/banquanly/binh-chon` | URL sau đăng nhập khớp trang đích của từng vai trò | Chưa chạy |
| **TC_AUTH_13** | Login | [UI] Nút "Tự động điền mẫu" điền đúng tài khoản demo | Positive | 1. Chọn tab Kế toán<br>2. Bấm "Tự động điền mẫu" | Tab `ketoan` | Ô username = `ketoan.lan`, password = `123456` | Chưa chạy |
| **TC_AUTH_14** | Login | [UI] Đăng nhập khi đã có session → tự chuyển về dashboard | Positive | 1. Đăng nhập thành công<br>2. Truy cập lại `/dang-nhap` | `cudan.p101` / `123456` | Chuyển thẳng tới trang đích của vai trò, không hiện form | Chưa chạy |
| **TC_AUTH_15** | Logout | [UI] Đăng xuất huỷ session | Positive | 1. Đăng nhập<br>2. Bấm Đăng xuất (`/logout`)<br>3. Mở lại `/cudan/thong-bao` | `cudan.p101` / `123456` | Bị chuyển về `/dang-nhap` | Chưa chạy |
| **TC_AUTH_16** | Login | [UI] Chuỗi SQL injection ở ô tên đăng nhập | Negative | 1. Nhập `' OR '1'='1` vào username<br>2. Nhập password bất kỳ | `' OR '1'='1` / `123456` | Banner "Tên đăng nhập không tồn tại."; không đăng nhập được | Chưa chạy |

## 2. PASSWORD & ĐIỀU HƯỚNG — UNIT TEST (`PasswordUtil`, `LoginServlet.calculateRedirectUrl`)

| Test Case ID | Chức năng | Mô tả ca kiểm thử | Loại test (Pos/Neg) | Các bước thực hiện | Dữ liệu kiểm thử (Test Data) | Kết quả mong đợi | Trạng thái (Pass/Fail) |
| :---: | :---: | :--- | :---: | :--- | :--- | :--- | :---: |
| **TC_PWD_01** | PasswordUtil | [UT] `hash` rồi `verify` đúng mật khẩu | Positive | 1. `hash("Pass@1234")`<br>2. `verify("Pass@1234", hash)` | `Pass@1234` | `verify` = `true`; hash bắt đầu bằng `$2a$` | Chưa chạy |
| **TC_PWD_02** | PasswordUtil | [UT] `verify` sai mật khẩu | Negative | 1. `verify("sai", hash("Pass@1234"))` | `sai` | `false` | Chưa chạy |
| **TC_PWD_03** | PasswordUtil | [UT][Parameterized] `hash` với đầu vào `null` hoặc rỗng | Negative | 1. Gọi `hash` | `null`, `""` | Trả `null` | Chưa chạy |
| **TC_PWD_04** | PasswordUtil | [UT][Parameterized] `verify` với tham số `null`/rỗng | Negative | 1. Gọi `verify(plain, hashed)` | (`null`, hash), (`"abc"`, `null`), (`"abc"`, `""`) | `false`, không ném ngoại lệ | Chưa chạy |
| **TC_PWD_05** | PasswordUtil | [UT] Mật khẩu thô (không phải BCrypt) trong DB bị từ chối | Negative | 1. `verify("123456", "123456")` | `123456` / `123456` | `false` (hash không bắt đầu bằng `$2a$/$2b$/$2y$`) | Chưa chạy |
| **TC_PWD_06** | PasswordUtil | [UT] Hash không hợp lệ bắt đầu bằng `$2a$` nhưng sai cấu trúc | Negative | 1. `verify("abc", "$2a$xx")` | `abc` / `$2a$xx` | `false`, không ném ngoại lệ ra ngoài | Chưa chạy |
| **TC_PWD_07** | PasswordUtil | [UT] Hai lần hash cùng mật khẩu cho kết quả khác nhau (salt) nhưng đều verify đúng | Positive | 1. `hash` hai lần cùng mật khẩu<br>2. `verify` cả hai | `123456` | Hai chuỗi hash khác nhau; cả hai `verify` = `true` | Chưa chạy |
| **TC_PWD_08** | PasswordUtil | [UT][Timeout] `hash` (cost 12) hoàn thành trong thời gian cho phép | Positive | 1. Gọi `hash` với `@Timeout(5s)` | `Pass@1234` | Hoàn thành < 5 giây | Chưa chạy |
| **TC_REDIR_01** | calculateRedirectUrl | [UT][Parameterized] Điều hướng theo vai trò/bộ phận | Positive | 1. Gọi `calculateRedirectUrl("/chungcu", vaiTro, boPhan)` | (`CD`,`null`)→`/chungcu/cudan/thong-bao`<br>(`BQL`,`BanQuanLy`)→`/chungcu/banquanly/binh-chon`<br>(`NV`,`LeTan`)→`/chungcu/letan/su-co`<br>(`NV`,`LT`)→`/chungcu/letan/su-co`<br>(`NV`,`KyThuat`/`NVKT`)→`/chungcu/kythuat/cong-viec`<br>(`NV`,`KeToan`/`KT`)→`/chungcu/ketoan/chi-so`<br>(`NV`,`BaoVe`/`BV`)→`/chungcu/baove/dashboard` | URL trả về đúng như cột dữ liệu | Chưa chạy |
| **TC_REDIR_02** | calculateRedirectUrl | [UT] `NV` với bộ phận không xác định hoặc `null` | Negative | 1. Gọi hàm với bộ phận lạ / `null` | (`NV`,`null`), (`NV`,`Khac`) | Trả `/chungcu/dang-nhap` | Chưa chạy |
| **TC_REDIR_03** | calculateRedirectUrl | [UT] Vai trò không xác định / không phân biệt hoa thường | Negative/Pos | 1. Gọi hàm với vai trò lạ và `cd` viết thường | (`XX`,`null`), (`cd`,`null`) | `XX` → `/chungcu/index.jsp`; `cd` → `/chungcu/cudan/thong-bao` | Chưa chạy |

## 3. PHÂN QUYỀN & BẢO VỆ TRUY CẬP (AUTHZ — `AuthFilter`)

| Test Case ID | Chức năng | Mô tả ca kiểm thử | Loại test (Pos/Neg) | Các bước thực hiện | Dữ liệu kiểm thử (Test Data) | Kết quả mong đợi | Trạng thái (Pass/Fail) |
| :---: | :---: | :--- | :---: | :--- | :--- | :--- | :---: |
| **TC_AUTHZ_01** | AuthFilter | [UI] Truy cập trang nội bộ khi chưa đăng nhập | Negative | 1. Mở trình duyệt ẩn danh<br>2. Truy cập trực tiếp `/cudan/phan-anh` | Chưa có session | Chuyển hướng về `/dang-nhap` | Chưa chạy |
| **TC_AUTHZ_02** | AuthFilter | [UI] Cư dân truy cập khu vực Ban quản lý | Negative | 1. Đăng nhập cư dân<br>2. Truy cập `/banquanly/tai-khoan` | `cudan.p101` | HTTP 403, hiển thị trang 403 | Chưa chạy |
| **TC_AUTHZ_03** | AuthFilter | [UI][Data-driven] Ma trận vai trò × khu vực không được phép | Negative | 1. Đăng nhập từng vai trò<br>2. Truy cập URL của khu vực khác | `letan.thi` → `/kythuat/cong-viec`<br>`ketoan.lan` → `/letan/su-co`<br>`baove.hung` → `/ketoan/chi-so`<br>`kythuat.nam` → `/cudan/thong-bao`<br>`bql.admin` → `/cudan/thong-bao` | Tất cả trả 403 | Chưa chạy |
| **TC_AUTHZ_04** | AuthFilter | [UI] Tài khoản bị khoá khi đang đăng nhập bị đá ra | Negative | 1. Đăng nhập cư dân (phiên A)<br>2. BQL khoá tài khoản đó (phiên B)<br>3. Phiên A tải lại trang | `cudan.p101` | Phiên A bị huỷ session, chuyển về `/dang-nhap` | Chưa chạy |
| **TC_AUTHZ_05** | AuthFilter | [UI] Tài nguyên tĩnh (whitelist) truy cập không cần đăng nhập | Positive | 1. Chưa đăng nhập<br>2. Mở `/css/login.css`, `/js/login.js` | Không có session | Trả 200, không bị chuyển hướng | Chưa chạy |
| **TC_AUTHZ_06** | AuthFilter / Login | [UI] Lỗi tiềm ẩn: nhân viên mang mã bộ phận alias (`LT`, `NVKT`, `KT`, `BV`) | Negative | 1. Dùng tài khoản `NV` có `boPhanCode` = alias<br>2. Đăng nhập đúng tab<br>3. Quan sát trang đích | `<NV, boPhanCode=LT>` / `123456` | **Mong đợi:** vào được `/letan/su-co`. **Dự đoán thực tế:** 403 do `AuthFilter` chỉ chấp nhận `LeTan` → ghi nhận bug nếu xảy ra | Chưa chạy |
| **TC_AUTHZ_07** | AuthFilter / Login | [UI] Lỗi tiềm ẩn: `NV` thuộc bộ phận `BanQuanLy`/`MAIN` | Negative | 1. Đăng nhập tài khoản `NV` có `boPhanCode` = `BanQuanLy`<br>2. Quan sát trang đích | `<NV, BanQuanLy>` / `123456` | **Mong đợi:** vào được `/banquanly/binh-chon`. **Dự đoán thực tế:** 403 vì filter yêu cầu `vaiTro = BQL` → ghi nhận bug nếu xảy ra | Chưa chạy |

## 4. CRUD — QUẢN LÝ TÀI KHOẢN (`/banquanly/tai-khoan`)

| Test Case ID | Chức năng | Mô tả ca kiểm thử | Loại test (Pos/Neg) | Các bước thực hiện | Dữ liệu kiểm thử (Test Data) | Kết quả mong đợi | Trạng thái (Pass/Fail) |
| :---: | :---: | :--- | :---: | :--- | :--- | :--- | :---: |
| **TC_ACC_01** | Tạo tài khoản | [UI] Tạo tài khoản cư dân hợp lệ | Positive | 1. Đăng nhập `bql.admin`<br>2. Mở form tạo tài khoản<br>3. Điền thông tin, chọn vai trò CD + căn hộ<br>4. Bấm Lưu | `cd.test<timestamp>` / `Test@1234` / CD / căn hộ hợp lệ | Thông báo "Tạo tài khoản và hồ sơ thông tin thành công!"; tài khoản hiện trong danh sách; đăng nhập được | Chưa chạy |
| **TC_ACC_02** | Tạo tài khoản | [UI] Tạo tài khoản nhân viên không chọn bộ phận | Positive | 1. Tạo tài khoản vai trò NV<br>2. Để trống bộ phận | `nv.test<timestamp>` / `Test@1234` / NV | Tạo thành công, `boPhanCode` mặc định `LeTan` | Chưa chạy |
| **TC_ACC_03** | Tạo tài khoản | [UI] Tên đăng nhập trùng | Negative | 1. Tạo tài khoản với tên đã tồn tại | `cudan.p101` / `Test@1234` / CD | Báo "Tên đăng nhập 'cudan.p101' đã tồn tại trong hệ thống!"; không tạo thêm | Chưa chạy |
| **TC_ACC_04** | Tạo tài khoản | [UI][Parameterized] Thiếu trường bắt buộc | Negative | 1. Bỏ trống lần lượt tên đăng nhập / mật khẩu / vai trò<br>2. Bấm Lưu | (`""`,`Test@1234`,CD), (`u1`,`""`,CD), (`u1`,`Test@1234`,`""`) | Báo "Vui lòng điền đầy đủ Tên đăng nhập, Mật khẩu và Vai trò."; không lưu | Chưa chạy |
| **TC_ACC_05** | Tạo tài khoản | [UI] BVA: tên đăng nhập 1 ký tự và mật khẩu 1 ký tự (hệ thống không có ràng buộc độ dài) | Positive | 1. Tạo tài khoản với giá trị ngắn nhất | `a` / `1` / CD | Ghi nhận hành vi thực tế (dự kiến tạo thành công vì chỉ kiểm tra khác rỗng) — nếu nhóm yêu cầu ràng buộc độ dài, đây là khuyết điểm cần báo | Chưa chạy |
| **TC_ACC_06** | Khoá/Mở khoá | [UI] Khoá tài khoản của người khác | Positive | 1. Chọn tài khoản cư dân<br>2. Bấm Khoá | `cudan.p101` | Thông báo "Đã thay đổi trạng thái tài khoản thành công."; tài khoản không đăng nhập được | Chưa chạy |
| **TC_ACC_07** | Khoá/Mở khoá | [UI] Mở khoá lại tài khoản đã khoá | Positive | 1. Bấm đổi trạng thái lần nữa trên tài khoản đang khoá | `cudan.p101` | Trạng thái về `HoatDong`; đăng nhập lại được | Chưa chạy |
| **TC_ACC_08** | Khoá/Mở khoá | [UI] Không được tự khoá tài khoản đang đăng nhập | Negative | 1. Đăng nhập `bql.admin`<br>2. Khoá chính `bql.admin` | `bql.admin` | Báo "Không thể tự khóa tài khoản Admin đang đăng nhập!"; trạng thái giữ nguyên | Chưa chạy |
| **TC_ACC_09** | Reset mật khẩu | [UI] Reset mật khẩu về mặc định | Positive | 1. Chọn tài khoản<br>2. Reset không nhập mật khẩu mới<br>3. Đăng nhập bằng `123456` | `<tài khoản test>` | Báo reset thành công; đăng nhập được bằng `123456` | Chưa chạy |
| **TC_ACC_10** | Reset mật khẩu | [UI] Reset mật khẩu với giá trị tuỳ chọn | Positive | 1. Reset với mật khẩu mới `Moi@12345`<br>2. Đăng nhập lại | `Moi@12345` | Đăng nhập bằng mật khẩu mới thành công, mật khẩu cũ bị từ chối | Chưa chạy |
| **TC_ACC_11** | Phân quyền | [UI] Chỉ BQL vào được trang quản lý tài khoản | Negative | 1. Đăng nhập kế toán<br>2. Truy cập `/banquanly/tai-khoan` | `ketoan.lan` | HTTP 403 | Chưa chạy |

## 5. CRUD — PHẢN ÁNH SỰ CỐ (`/cudan/phan-anh`)

| Test Case ID | Chức năng | Mô tả ca kiểm thử | Loại test (Pos/Neg) | Các bước thực hiện | Dữ liệu kiểm thử (Test Data) | Kết quả mong đợi | Trạng thái (Pass/Fail) |
| :---: | :---: | :--- | :---: | :--- | :--- | :--- | :---: |
| **TC_INC_01** | Thêm phản ánh | [UI] Gửi phản ánh hợp lệ không kèm ảnh | Positive | 1. Đăng nhập cư dân<br>2. Mở form phản ánh<br>3. Điền tiêu đề, mô tả, loại sự cố<br>4. Gửi | Tiêu đề `Rò rỉ nước <timestamp>` / mô tả `Nước chảy ra sàn` / `Nuoc` | Thông báo "Gửi phản ánh sự cố thành công!…"; phản ánh hiện trong danh sách | Chưa chạy |
| **TC_INC_02** | Thêm phản ánh | [UI] Gửi phản ánh kèm ảnh `.png` hợp lệ | Positive | 1. Điền form hợp lệ<br>2. Chọn file `.png`<br>3. Gửi | file `anh_test.png` | Thành công; ảnh lưu dưới `assets/uploads/suco/suco_<...>.png` | Chưa chạy |
| **TC_INC_03** | Thêm phản ánh | [UI][Parameterized] Thiếu tiêu đề hoặc mô tả | Negative | 1. Bỏ trống hoặc chỉ nhập khoảng trắng<br>2. Gửi | (`""`,`mô tả`), (`tiêu đề`,`""`), (`"   "`,`"   "`) | Báo "Vui lòng nhập đầy đủ tiêu đề và nội dung mô tả sự cố."; không lưu | Chưa chạy |
| **TC_INC_04** | Thêm phản ánh | [UI][Parameterized] EP — `loaiSuCo` hợp lệ | Positive | 1. Gửi lần lượt với từng loại | `Dien`, `Nuoc`, `ThangMay`, `PCCC`, `AnNinh`, `VeSinh`, `Khac` | Cả 7 giá trị đều gửi thành công | Chưa chạy |
| **TC_INC_05** | Thêm phản ánh | [UI] EP — `loaiSuCo` không hợp lệ hoặc thiếu | Negative | 1. Gửi với loại rỗng / giá trị lạ (chỉnh qua DevTools) | `""`, `Hack` | Báo "Loại sự cố không hợp lệ."; không lưu | Chưa chạy |
| **TC_INC_06** | Thêm phản ánh | [UI] `mucDoUuTien` rỗng hoặc không hợp lệ → mặc định `TrungBinh` | Positive | 1. Gửi form với mức ưu tiên rỗng hoặc `Khan`<br>2. Mở chi tiết phản ánh | `""`, `Khan` | Gửi thành công, mức ưu tiên lưu là `TrungBinh` | Chưa chạy |
| **TC_INC_07** | Thêm phản ánh | [UI][Parameterized] EP — đuôi file ảnh không hợp lệ | Negative | 1. Đính kèm file có đuôi không cho phép<br>2. Gửi | `anh.gif`, `file.exe`, `anh` (không đuôi) | Báo "File hình ảnh không hợp lệ! Chỉ chấp nhận định dạng .jpg, .jpeg, .png."; không lưu | Chưa chạy |
| **TC_INC_08** | Thêm phản ánh | [UI] File đổi đuôi `.png` nhưng không phải ảnh (MIME không phải `image/*`) | Negative | 1. Đổi tên file văn bản thành `.png`<br>2. Gửi | `fake.png` (nội dung text) | Ghi nhận hành vi: MIME do trình duyệt suy từ đuôi nên có thể vẫn được nhận — nếu nhận, ghi bug bảo mật (không kiểm tra nội dung file) | Chưa chạy |
| **TC_INC_09** | Huỷ phản ánh | [UI] Huỷ phản ánh vừa gửi | Positive | 1. Mở danh sách phản ánh<br>2. Chọn phản ánh vừa tạo<br>3. Bấm Huỷ | Phản ánh ở TC_INC_01 | Phản ánh đổi trạng thái đã huỷ, có thông báo thành công | Chưa chạy |
| **TC_INC_10** | Huỷ phản ánh | [UI] Huỷ với mã phản ánh không hợp lệ | Negative | 1. Gửi request `/cudan/phan-anh/huy?id=abc` | `id=abc` | Hiển thị "Mã phản ánh không hợp lệ…"; chuyển về danh sách, không lỗi 500 | Chưa chạy |
| **TC_INC_11** | Phân quyền | [UI] Nhân viên không vào được trang phản ánh của cư dân | Negative | 1. Đăng nhập lễ tân<br>2. Truy cập `/cudan/phan-anh` | `letan.thi` | HTTP 403 | Chưa chạy |

## 6. CRUD — GHI CHỈ SỐ ĐIỆN NƯỚC (`/ketoan/chi-so`)

*Unit test gọi `ChiSoTieuThuDAO.luuSingleChiSo(null, maCanHo, loai, thang, nam, chiSo, chiSoKyTruoc)` — các nhánh lỗi trả về trước khi dùng `EntityManager`.*

| Test Case ID | Chức năng | Mô tả ca kiểm thử | Loại test (Pos/Neg) | Các bước thực hiện | Dữ liệu kiểm thử (Test Data) | Kết quả mong đợi | Trạng thái (Pass/Fail) |
| :---: | :---: | :--- | :---: | :--- | :--- | :--- | :---: |
| **TC_METER_01** | Ghi chỉ số | [UI] Lưu chỉ số hợp lệ cho một căn hộ | Positive | 1. Đăng nhập `ketoan.lan`<br>2. Mở `/ketoan/chi-so`<br>3. Nhập điện và nước lớn hơn kỳ trước<br>4. Bấm Lưu | Điện `150`, nước `30` (kỳ trước thấp hơn) | Chuyển lại trang với thông báo "Đã lưu chỉ số điện nước thành công cho Tháng …"; giá trị hiển thị trên bảng | Chưa chạy |
| **TC_METER_02** | Ghi chỉ số | [UT] BVA — tháng ngoài khoảng 1–12 | Negative | 1. Gọi `luuSingleChiSo` với tháng 0 và 13 | `thang = 0`, `thang = 13` | Trả "Tháng phải nằm trong khoảng từ 1 đến 12." | Chưa chạy |
| **TC_METER_03** | Ghi chỉ số | [UT][Parameterized] BVA — tháng biên hợp lệ không bị từ chối ở bước kiểm tra tháng | Positive | 1. Gọi với tháng 1 và 12, chỉ số âm để dừng ở bước kiểm tra chỉ số | `thang = 1`, `thang = 12`, `chiSo = -1` | Trả "Chỉ số mới không được nhỏ hơn 0." (chứng tỏ qua bước kiểm tra tháng) | Chưa chạy |
| **TC_METER_04** | Ghi chỉ số | [UT][Parameterized] BVA — chỉ số âm | Negative | 1. Gọi với chỉ số âm | `chiSo = -0.1`, `-1`, `-1000` | Trả "Chỉ số mới không được nhỏ hơn 0." | Chưa chạy |
| **TC_METER_05** | Ghi chỉ số | [UT] BVA — chỉ số mới nhỏ hơn kỳ trước | Negative | 1. Gọi với chỉ số mới < kỳ trước | `chiSo = 99`, `kyTruoc = 100` | Trả thông báo chứa "nhỏ hơn chỉ số kỳ trước (100.0)" | Chưa chạy |
| **TC_METER_06** | Ghi chỉ số | [UT] BVA — chỉ số mới nhỏ hơn kỳ trước đúng 0.01 | Negative | 1. Gọi với chênh lệch rất nhỏ | `chiSo = 99.99`, `kyTruoc = 100` | Bị từ chối như TC_METER_05 | Chưa chạy |
| **TC_METER_07** | Ghi chỉ số | [UI] BVA — chỉ số bằng đúng kỳ trước là hợp lệ | Positive | 1. Nhập chỉ số bằng chỉ số kỳ trước<br>2. Lưu | `chiSo = kyTruoc` | Lưu thành công (không báo lỗi) | Chưa chạy |
| **TC_METER_08** | Ghi chỉ số | [UI] BVA — chỉ số bằng 0 khi chưa có kỳ trước | Positive | 1. Nhập `0` cho căn hộ chưa có chỉ số kỳ trước<br>2. Lưu | `chiSo = 0` | Lưu thành công | Chưa chạy |
| **TC_METER_09** | Ghi chỉ số | [UI] Một dòng lỗi thì rollback toàn bộ lô | Negative | 1. Nhập căn hộ A hợp lệ và căn hộ B nhỏ hơn kỳ trước<br>2. Lưu<br>3. Tải lại trang | A: hợp lệ, B: `< kỳ trước` | Báo "Căn hộ #<B> (Điện): …"; **cả A lẫn B đều không được lưu** | Chưa chạy |
| **TC_METER_10** | Ghi chỉ số | [UI] Lưu lại cùng kỳ → cập nhật, không tạo bản ghi trùng | Positive | 1. Lưu chỉ số kỳ hiện tại<br>2. Sửa giá trị, lưu lần nữa | `150` rồi `160` | Bảng hiển thị `160`; không có hai bản ghi cùng căn hộ/kỳ/loại | Chưa chạy |
| **TC_METER_11** | Ghi chỉ số | [UI] Để trống ô chỉ số thì bỏ qua | Positive | 1. Chỉ nhập điện, để trống nước<br>2. Lưu | Điện `150`, nước `""` | Chỉ lưu điện; không báo lỗi | Chưa chạy |
| **TC_METER_12** | Ghi chỉ số | [UI] Giá trị không phải số bị bỏ qua | Negative | 1. Gửi request với `chiSoDien_<id> = abc` | `abc` | Không lỗi 500; dòng đó bị bỏ qua | Chưa chạy |
| **TC_METER_13** | Ghi chỉ số | [UI] Không có dữ liệu nào để lưu | Negative | 1. Mở kỳ không có căn hộ nào hoặc gửi form không có `maCanHo` | Không có dòng | Báo "Không có dữ liệu chỉ số để ghi nhận." | Chưa chạy |
| **TC_METER_14** | Ghi chỉ số | [UI][Parameterized] BVA — tham số kỳ ngoài khoảng dùng giá trị hiện tại | Negative | 1. Mở `/ketoan/chi-so?thang=0&nam=2026`, `?thang=13`, `?nam=1999` | `thang = 0/13`, `nam = 1999` | Trang vẫn hiển thị với tháng/năm hiện tại; `thang = 1/12` và `nam = 2000` được giữ nguyên | Chưa chạy |
| **TC_METER_15** | Phân quyền | [UI] Cư dân không truy cập được trang ghi chỉ số | Negative | 1. Đăng nhập cư dân<br>2. Truy cập `/ketoan/chi-so` | `cudan.p101` | HTTP 403 | Chưa chạy |

---

## 7. THỐNG KÊ

| Nhóm | Số ca | Smoke (gợi ý) |
| :--- | :---: | :--- |
| AUTH (UI) | 16 | TC_AUTH_01, 02, 07, 12 |
| PASSWORD / REDIRECT (Unit) | 11 | Toàn bộ chạy ở CI |
| AUTHZ | 7 | TC_AUTHZ_01, 02 |
| ACC | 11 | TC_ACC_01, 03 |
| INC | 11 | TC_INC_01, 03 |
| METER | 15 | TC_METER_01 |
| **Tổng** | **71** | |
