# KẾ HOẠCH KIỂM THỬ (TEST PLAN)

* **Dự án:** Hệ thống Quản lý Chung cư (Poly Building)
* **Sinh viên:** Lê Minh Kha - TV00484 (SD21308)

---

## 1. MỤC TIÊU KIỂM THỬ
* Kiểm thử logic backend (Unit Test với JUnit 5 cho các Servlet/Controller và DAO).
* Kiểm thử giao diện tự động (Automation UI Test với Selenium WebDriver nếu có).
* Tích hợp kiểm thử tự động với GitHub Actions CI/CD.

## 2. PHẠM VI KIỂM THỬ
* **In-Scope:** Luồng Đăng nhập, Đăng nhập thẻ từ, Phân quyền hệ thống, và thao tác CRUD (Căn hộ, Cư dân, Hóa đơn, Tiện ích).
* **Out-of-Scope:** Kiểm thử hiệu năng (Performance Test), Bảo mật chuyên sâu ngoài phạm vi đồ án môn học.

## 3. TIÊU CHÍ HOÀN THÀNH
* 100% test script thực thi pass (`mvn test`).
* Pipeline GitHub Actions báo xanh (Success).
