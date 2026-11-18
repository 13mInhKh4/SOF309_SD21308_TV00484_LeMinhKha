# ĐẶC TẢ NGHIỆP VỤ WEB CÁ NHÂN (SPECIFICATIONS)
> *Dùng làm ngữ cảnh (Context) gửi cho AI để sinh Test Plan & Test Case chính xác theo dự án riêng của bạn.*

---

## 1. THÔNG TIN DỰ ÁN
* **Tên đề tài:** Hệ thống Quản lý Chung cư (Poly Building)
* **Công nghệ:** Java 17+, Java Servlet / JSP, JPA / JDBC, SQL Server (`QLCCNew2.sql`), Apache Tomcat 10.1
* **URL chạy local:** `http://localhost:8080`

## 2. QUY TẮC NGHIỆP VỤ (BUSINESS RULES)
* **Chức năng Đăng nhập & Phân quyền (`LoginServlet`, `AuthFilter`):** 
  * Kiểm tra tài khoản (`username`, `password`) hoặc đăng nhập bằng thẻ từ (`DangNhapTheServlet`).
  * Đúng thông tin $\rightarrow$ điều hướng về trang Dashboard tương ứng theo vai trò (Ban Quản Lý, Lễ tân, Kế toán, Bảo vệ, Kỹ thuật, Cư dân); sai $\rightarrow$ báo lỗi tại trang đăng nhập.
* **Chức năng Quản lý Căn hộ & Cư dân (`CanHo`, `CuDan`):**
  * Thêm mới / Cập nhật căn hộ: Mã căn hộ bắt buộc và không được trùng lặp, diện tích ($> 0$), trạng thái hợp lệ.
  * Quản lý thông tin cư dân: Họ tên không được để trống, số điện thoại và email phải đúng định dạng.
* **Chức năng Quản lý Hóa đơn & Giao dịch (`HoaDon`, `GiaoDichThanhToan`, `KeToanHoaDonServlet`):**
  * Chỉ số tiêu thụ điện/nước mới phải lớn hơn hoặc bằng chỉ số cũ ($\ge 0$).
  * Tổng tiền hóa đơn được tính tự động dựa trên biểu giá dịch vụ (`BieuGiaDichVu`), không nhận giá trị âm.
* **Chức năng Tiện ích, Thẻ từ & Phản ánh sự cố (`DatLichTienIch`, `TheTu`, `PhanAnhSuCo`):**
  * Đăng ký lịch tiện ích không được trùng khung giờ đã có người đặt.
  * Xóa bản ghi (căn hộ, thẻ từ, thông báo, biểu giá): Bắt buộc có bước xác nhận trước khi xóa và cập nhật trạng thái vào cơ sở dữ liệu.
