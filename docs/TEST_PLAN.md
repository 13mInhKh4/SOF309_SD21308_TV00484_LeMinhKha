# KẾ HOẠCH KIỂM THỬ (TEST PLAN)

| Thông tin | Nội dung |
| :--- | :--- |
| **Dự án** | Hệ thống quản lý chung cư Polybuilding (QLCC) |
| **Công nghệ** | Jakarta Servlet/JSP, Hibernate, SQL Server |
| **Sinh viên** | Lê Minh Kha - TV00484 |

---

## 1. MỤC TIÊU KIỂM THỬ

| STT | Mục tiêu |
| :---: | :--- |
| 1 | Unit Test JUnit 5 cho các thành phần không cần DB: `LoginServlet.calculateRedirectUrl`, `PasswordUtil`, nhánh validation của `ChiSoTieuThuDAO.luuSingleChiSo` (kiểm tra đầu vào trước khi chạm tới `EntityManager`). |
| 2 | Kiểm thử giao diện tự động bằng Selenium WebDriver + Page Object Model + TestNG cho: Đăng nhập, Phân quyền, Quản lý tài khoản, Phản ánh sự cố, Ghi chỉ số điện nước. |
| 3 | Tích hợp kiểm thử tự động với GitHub Actions (Chrome headless trên Ubuntu). |
| 4 | Áp dụng phân vùng tương đương và phân tích giá trị biên để thiết kế dữ liệu kiểm thử (xem mục 4). |

## 2. PHẠM VI KIỂM THỬ

| Phạm vi | Nội dung |
| :---: | :--- |
| **In-Scope** | Đăng nhập (`/dang-nhap`, `/login`) với 6 vai trò, đăng xuất |
| **In-Scope** | Phân quyền & bảo vệ truy cập (`AuthFilter`): chưa đăng nhập, sai vai trò, tài khoản bị khoá |
| **In-Scope** | CRUD: Quản lý tài khoản (`/banquanly/tai-khoan`), Phản ánh sự cố (`/cudan/phan-anh`), Ghi chỉ số điện nước (`/ketoan/chi-so`) |
| **Out-of-Scope** | Kiểm thử hiệu năng; bảo mật chuyên sâu (chỉ có 1 ca SQL injection cơ bản ở ô tên đăng nhập) |
| **Out-of-Scope** | Đăng nhập bằng thẻ NFC/QR (`/dang-nhap-the`); các module Bình chọn, Tiện ích, Bảo vệ, Đối soát sao kê, Hoá đơn; giao diện di động |

## 3. CHIẾN LƯỢC & CÔNG CỤ

| Tầng kiểm thử | Công cụ | Đối tượng | Ghi chú |
| :--- | :--- | :--- | :--- |
| Unit test | JUnit 5 (`@ParameterizedTest`, `assertThrows`, `@Timeout`) | `calculateRedirectUrl`, `PasswordUtil`, `luuSingleChiSo` | Không cần DB; `luuSingleChiSo` gọi với `em = null` ở các nhánh trả lỗi sớm. Servlet/DAO cần DB thì kiểm thử ở tầng UI. |
| UI automation | Selenium 4 + Page Object Model (`LoginPage`, `AccountPage`, `IncidentPage`, `MeterPage`) | Các luồng end-to-end | Dùng `WebDriverWait`, không dùng `Thread.sleep()`. |
| Điều phối | TestNG (`groups = smoke/regression`, `dependsOnMethods`, `@DataProvider`, `ITestListener`) | Toàn bộ UI test | Test fail tự chụp ảnh vào `screenshots/`. |
| CI/CD | GitHub Actions (`.github/workflows/test.yml`) | `mvn clean test` | Upload báo cáo Surefire + ảnh chụp lỗi lên Artifacts. |

## 4. KỸ THUẬT THIẾT KẾ TEST CASE

| Kỹ thuật | Áp dụng cho | Giá trị kiểm thử |
| :--- | :--- | :--- |
| Phân vùng tương đương (EP) | `loaiSuCo` | 7 giá trị hợp lệ / rỗng, `null`, giá trị lạ |
| Phân vùng tương đương (EP) | Đuôi file ảnh | `.jpg`, `.jpeg`, `.png` / `.gif`, `.exe`, không đuôi |
| Phân vùng tương đương (EP) | Trạng thái tài khoản | `HoatDong` / `Khoa` |
| Giá trị biên (BVA) | `thang` | 0 \| **1** \| 2 … 11 \| **12** \| 13 |
| Giá trị biên (BVA) | Chỉ số điện/nước | -0.1 \| **0** \| kỳ trước - 1 \| **= kỳ trước** \| kỳ trước + 1 |
| Giá trị biên (BVA) | `tenDangNhap`, `matKhau` | rỗng \| 1 ký tự \| chỉ khoảng trắng \| khoảng trắng đầu/cuối (được `trim()`) |
| Giá trị biên (BVA) | `nam` | 1999 \| **2000** \| 2001 |
| Bảng quyết định | Đăng nhập | tồn tại? bị khoá? đúng mật khẩu? đúng vai trò? đủ hồ sơ? |
| Bảng quyết định | Phân quyền | ma trận vai trò × URL |

## 5. MÔI TRƯỜNG & DỮ LIỆU KIỂM THỬ

| Hạng mục | Nội dung |
| :--- | :--- |
| Môi trường | JDK 17, Maven, Tomcat 10/11 (Smart Tomcat), SQL Server với DB `QLCCNew2`, Chrome/ChromeDriver (Selenium Manager tự tải) |
| Tài khoản demo | Mật khẩu `123456`: `cudan.p101`, `letan.thi`, `kythuat.nam`, `ketoan.lan`, `baove.hung`, `bql.admin` |
| Dữ liệu cần chuẩn bị thêm | Tài khoản bị khoá, cư dân chưa gán căn hộ, nhân viên chưa có hồ sơ |
| Dữ liệu tạo mới | Dùng tên đăng nhập/tiêu đề có hậu tố thời gian để không trùng giữa các lần chạy |
| CI | GitHub Actions không có SQL Server mặc định: chạy nhóm unit test trên CI, test UI chạy ở local hoặc thêm service container SQL Server |

## 6. TIÊU CHÍ HOÀN THÀNH

| STT | Tiêu chí |
| :---: | :--- |
| 1 | 100% test script thực thi pass (`mvn test` BUILD SUCCESS, không Failure/Error) |
| 2 | Toàn bộ ca nhóm `smoke` pass; các ca `regression` pass hoặc có lỗi đã ghi nhận kèm mức độ |
| 3 | Pipeline GitHub Actions báo xanh (Success) và upload được Artifacts |
| 4 | Mọi lỗi phát hiện được ghi nhận kèm bước tái hiện |

## 7. RỦI RO & LỖI TIỀM ẨN ĐÃ PHÁT HIỆN KHI ĐỌC MÃ NGUỒN

| STT | Vấn đề | Test case liên quan |
| :---: | :--- | :---: |
| 1 | `LoginServlet` chấp nhận mã bộ phận alias `LT`/`NVKT`/`KT`/`BV` và điều hướng vào trang bộ phận, nhưng `AuthFilter` chỉ chấp nhận `LeTan`/`KyThuat`/`KeToan`/`BaoVe` → nhân viên mang mã alias có thể đăng nhập được nhưng bị 403 ngay trang đầu (cần kiểm tra DB thực tế vì patch `28_chuan_hoa_ma_tai_khoan.sql` có thể đã chuẩn hoá) | TC_AUTHZ_06 |
| 2 | `NV` thuộc bộ phận `BanQuanLy`/`MAIN` được điều hướng sang `/banquanly/binh-chon`, nhưng `AuthFilter` yêu cầu `vaiTro = BQL` → 403 | TC_AUTHZ_07 |
| 3 | Thông báo lỗi đăng nhập khác nhau cho "không tồn tại" và "sai mật khẩu" (cho phép dò tên đăng nhập); ghi nhận như khuyến nghị bảo mật | TC_AUTH_06, TC_AUTH_07 |
| 4 | Tạo tài khoản không có ràng buộc độ dài hay độ mạnh mật khẩu (chỉ cần khác rỗng) | TC_ACC_05 |

## 8. LỊCH TRÌNH

| Tuần | Thời gian | Công việc |
| :---: | :---: | :--- |
| 1 | 02/11 - 08/11 | Đặc tả, Test Plan, Test Case |
| 2 | 09/11 - 15/11 | Unit test JUnit 5 (Parameterized, Exception, Timeout) |
| 3 | 16/11 - 22/11 | Selenium POM + WebDriverWait |
| 4 | 23/11 - 29/11 | TestNG groups, DataProvider, Listener chụp ảnh lỗi |
| 5 | 30/11 - 06/12 | GitHub Actions headless, thu thập log debug AI |
| 6 | 07/12 - 10/12 | AI code review, hoàn thiện `docs/ai-logs/`, slide, đóng gói |
