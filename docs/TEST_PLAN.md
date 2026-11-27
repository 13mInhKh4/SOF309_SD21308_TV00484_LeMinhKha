KẾ HOẠCH KIỂM THỬ (TEST PLAN)
Dự án: Hệ thống quản lý chung cư Polybuilding (QLCC) — Jakarta Servlet/JSP + Hibernate + SQL Server
Sinh viên: [Họ và tên - MSSV]
---
1. MỤC TIÊU KIỂM THỬ
Kiểm thử logic backend bằng Unit Test JUnit 5 cho các thành phần kiểm thử được độc lập với DB: `LoginServlet.calculateRedirectUrl`, `PasswordUtil`, và các nhánh validation của `ChiSoTieuThuDAO.luuSingleChiSo` (kiểm tra đầu vào trước khi chạm tới `EntityManager`).
Kiểm thử giao diện tự động (Selenium WebDriver + Page Object Model + TestNG) cho: Đăng nhập, Phân quyền, Quản lý tài khoản, Phản ánh sự cố, Ghi chỉ số điện nước.
Tích hợp kiểm thử tự động với GitHub Actions (Chrome headless trên Ubuntu).
Áp dụng kỹ thuật phân vùng tương đương và phân tích giá trị biên để thiết kế dữ liệu kiểm thử (xem mục 4).
2. PHẠM VI KIỂM THỬ
In-Scope:
Đăng nhập (`/dang-nhap`, `/login`) với 6 vai trò, đăng xuất.
Phân quyền & bảo vệ truy cập (`AuthFilter`): chưa đăng nhập, sai vai trò, tài khoản bị khoá.
CRUD: Quản lý tài khoản (`/banquanly/tai-khoan`), Phản ánh sự cố (`/cudan/phan-anh`), Ghi chỉ số điện nước (`/ketoan/chi-so`).
Out-of-Scope: kiểm thử hiệu năng; bảo mật chuyên sâu (chỉ có 1 ca kiểm tra SQL injection cơ bản trên ô tên đăng nhập); đăng nhập bằng thẻ NFC/QR (`/dang-nhap-the`); các module Bình chọn, Tiện ích, Bảo vệ, Đối soát sao kê, Hoá đơn; hiển thị trên thiết bị di động.
3. CHIẾN LƯỢC & CÔNG CỤ
Tầng kiểm thử	Công cụ	Đối tượng	Ghi chú
Unit test	JUnit 5 (`@ParameterizedTest`, `assertThrows`, `@Timeout`)	`calculateRedirectUrl`, `PasswordUtil`, `luuSingleChiSo`	Không cần DB; `luuSingleChiSo` gọi với `em = null` ở các nhánh trả lỗi sớm. Servlet/DAO cần DB chưa mock được thì kiểm thử ở tầng UI.
UI automation	Selenium 4 + Page Object Model (`LoginPage`, `AccountPage`, `IncidentPage`, `MeterPage`)	Các luồng end-to-end	Dùng `WebDriverWait`, không dùng `Thread.sleep()`.
Điều phối	TestNG (`groups = smoke/regression`, `dependsOnMethods`, `@DataProvider`, `ITestListener`)	Toàn bộ UI test	Test fail tự chụp ảnh vào `screenshots/`.
CI/CD	GitHub Actions (`.github/workflows/test.yml`)	`mvn clean test`	Upload báo cáo Surefire + ảnh chụp lỗi lên Artifacts.
4. KỸ THUẬT THIẾT KẾ TEST CASE
Phân vùng tương đương (EP): mỗi trường đầu vào chia thành lớp hợp lệ và không hợp lệ, ví dụ `loaiSuCo` ∈ {7 giá trị hợp lệ} / {rỗng, `null`, giá trị lạ}; đuôi file ∈ {`.jpg`, `.jpeg`, `.png`} / {`.gif`, `.exe`, không đuôi}; trạng thái tài khoản ∈ {`HoatDong`, `Khoa`}.
Phân tích giá trị biên (BVA):
`thang`: 0 | 1 | 2 … 11 | 12 | 13.
Chỉ số điện/nước: −0.1 | 0 | kỳ trước − 1 | = kỳ trước | kỳ trước + 1.
`tenDangNhap`/`matKhau`: rỗng | 1 ký tự | chỉ khoảng trắng | có khoảng trắng đầu/cuối (được `trim()`).
`nam`: 1999 | 2000 | 2001.
Bảng quyết định: đăng nhập (tồn tại? bị khoá? đúng mật khẩu? đúng vai trò? đủ hồ sơ?) và ma trận vai trò × URL cho phân quyền.
5. MÔI TRƯỜNG & DỮ LIỆU KIỂM THỬ
JDK 17, Maven, Tomcat 10/11 (Smart Tomcat), SQL Server với DB `QLCCNew2`, Chrome/ChromeDriver (Selenium Manager tự tải).
Dữ liệu: tài khoản demo mật khẩu `123456` (`cudan.p101`, `letan.thi`, `kythuat.nam`, `ketoan.lan`, `baove.hung`, `bql.admin`); cần chuẩn bị thêm tài khoản bị khoá, cư dân chưa gán căn hộ, nhân viên chưa có hồ sơ.
Với test UI: chạy lại script seed (hoặc dùng tên đăng nhập/tiêu đề có hậu tố thời gian) để các ca tạo mới không trùng dữ liệu giữa các lần chạy.
Trên CI (GitHub Actions) không có SQL Server mặc định → chạy nhóm unit test và các test không cần backend; test UI chạy ở local hoặc bổ sung service container SQL Server (xem mục 8).
6. TIÊU CHÍ HOÀN THÀNH
100% test script thực thi pass (`mvn test` BUILD SUCCESS, không Failure/Error).
Toàn bộ ca nhóm `smoke` pass; các ca `regression` pass hoặc có lỗi đã ghi nhận kèm mức độ.
Pipeline GitHub Actions báo xanh (Success) và upload được Artifacts.
Mọi lỗi phát hiện được ghi nhận (mục 7) kèm bước tái hiện.
7. RỦI RO & LỖI TIỀM ẨN ĐÃ PHÁT HIỆN KHI ĐỌC MÃ NGUỒN
Các điểm cần xác nhận bằng test (xem TC_AUTHZ_06, TC_AUTHZ_07 trong `TEST_CASES.md`):
Không nhất quán mã bộ phận: `LoginServlet` chấp nhận alias `LT`/`NVKT`/`KT`/`BV` và điều hướng vào trang bộ phận, nhưng `AuthFilter` chỉ chấp nhận `LeTan`/`KyThuat`/`KeToan`/`BaoVe` → nhân viên mang mã alias có thể đăng nhập thành công nhưng bị 403 ngay trang đầu. (Patch `28_chuan_hoa_ma_tai_khoan.sql` có thể đã chuẩn hoá dữ liệu — cần kiểm tra DB thực tế.)
`NV` với bộ phận `BanQuanLy`/`MAIN`: `LoginServlet` điều hướng sang `/banquanly/binh-chon`, nhưng `AuthFilter` yêu cầu `vaiTro = BQL` cho `/banquanly/**` → 403.
Servlet đăng nhập trả thông báo lỗi khác nhau cho "không tồn tại" và "sai mật khẩu" (cho phép dò tên đăng nhập) — ghi nhận như khuyến nghị bảo mật, không thuộc phạm vi kiểm thử chính.
Tên đăng nhập/mật khẩu mới ở chức năng tạo tài khoản không có ràng buộc độ dài hay độ mạnh (chỉ cần khác rỗng) — dùng làm ca biên để ghi nhận hành vi thực tế.
8. LỊCH TRÌNH
Tuần	Công việc
1 (02–08/11)	Đặc tả, Test Plan, Test Case (tài liệu này)
2 (09–15/11)	Unit test JUnit 5 (Parameterized, Exception, Timeout)
3 (16–22/11)	Selenium POM + WebDriverWait
4 (23–29/11)	TestNG groups, DataProvider, Listener chụp ảnh lỗi
5 (30/11–06/12)	GitHub Actions headless, thu thập log debug AI
6 (07–10/12)	AI code review, hoàn thiện `docs/ai-logs/`, slide, đóng gói
