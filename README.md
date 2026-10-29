# DoAnThucTapTN
kiem tra chất lượng
# HỆ THỐNG QUẢN LÝ KIỂM TRA CHẤT LƯỢNG SẢN PHẨM (3IZ QUALITY INSPECTION SYSTEM)
**Phiên bản:** TK_VT 3.0  
**Cơ sở dữ liệu đám mây:** Firebase Realtime Database  
**URL RTDB:** `https://doanthuctap-41ccc-default-rtdb.asia-southeast1.firebasedatabase.app/`

---

## 🌟 TỔNG QUAN HỆ THỐNG

Hệ thống được xây dựng bám sát 100% nội dung và giao diện mô tả trong file `file_bai_lam.docx` cùng 4 ảnh minh họa:
1. **Thẻ 11 Mã Lỗi (image1.png):** Mã vạch Code128 cho 11 loại lỗi (E001 - E011), Không Lỗi (E000), Lệnh Lưu (ZYYY), Lệnh Xóa (ZZZZ), và thẻ INTERNAL USE ONLY.
2. **Màn hình Quét 3IZ SCAN (image2.png):** Trạm quét barcode chuyên nghiệp, tự động điền thông tin từ Master Data, ghi nhận lỗi và đẩy dữ liệu lên Firebase.
3. **Màn hình Báo cáo 3IZ REPORT (image3.png):** Bảng phân tích chi tiết dữ liệu kiểm hàng, bộ lọc đa năng, dòng tính tổng tự động và tính năng xuất file Excel `.xlsx` trực tiếp.
4. **Màn hình Biểu đồ Phân tích 3IZ Analysis (image4.png):** 6 biểu đồ trực quan (Chart.js) phản ánh sản lượng, phân bố lỗi, thời gian kiểm tra trung bình theo nhà máy và thị trường xuất khẩu.
5. **Form Đăng nhập:** Quản lý tài khoản kiểm định viên (mặc định ID `22194805` - HỒ XUÂN KHÁNH), phân xưởng `FCT` mặc định `VT1`.

---

## 🚀 HƯỚNG DẪN KHỞI CHẠY VÀ SỬ DỤNG

### Cách 1: Mở trực tiếp trên trình duyệt
Bạn chỉ cần mở trực tiếp file `index.html` (hoặc các file `.html` riêng lẻ) bằng bất kỳ trình duyệt nào (Chrome, Edge, Firefox, Cốc Cốc):
- [index.html](index.html): **Cổng Tổng Hợp (Portal)** tích hợp đầy đủ hệ thống Tab đa nhiệm.
- [scan.html](scan.html): Màn hình Quét mã vạch **3IZ SCAN**.
- [report.html](report.html): Màn hình Báo cáo **3IZ REPORT** & Tải Excel.
- [chart.html](chart.html): Màn hình Biểu đồ **3IZ Analysis**.
- [master_data.html](master_data.html): Bảng dữ liệu gốc **DU LIEU MAU_ CHUAN.txt**.
- [defects.html](defects.html): Bảng 11 Thẻ Mã Vạch Lỗi (Hỗ trợ In ấn A4).
- [login.html](login.html): Trang Đăng nhập hệ thống.

### Cách 2: Chạy qua Local Web Server
Hệ thống hiện đang chạy sẵn trên cổng `8080`:
👉 **URL:** `http://localhost:8080/index.html`

---

## 🛠️ QUY TRÌNH QUÉT HÀNG 3 BƯỚC (CHUẨN THEO DOCX)

1. **BƯỚC 1 - Quét UCC:**
   - Quét hoặc nhập mã UCC (Ví dụ: `9929197877`, `9929220764`, `9929239469`...) vào ô `DOC# OR PCARD SCAN` rồi ấn **Enter**.
   - Hệ thống tự động tra cứu từ Firebase/Master Data và điền các thông tin: `UCC LABEL`, `INVOICE`, `DEST`, `QTY`, `STYLE NO`, `MODEL`.
   - Phát âm thanh **Beep** thành công.

2. **BƯỚC 2 - Quét Mã Lỗi:**
   - Nhập hoặc quét mã lỗi từ bảng 11 lỗi (Ví dụ: `E001` - Contamination, `E002` - Bond gap... hoặc `E000` - None Defect) vào ô `DOC#` rồi ấn **Enter**.
   - Ô `DEFECT` sẽ hiển thị tên lỗi và cập nhật trạng thái.

3. **BƯỚC 3 - Lưu hoặc Hủy:**
   - Quét mã `ZYYY` vào ô `DOC#` rồi ấn **Enter** (hoặc bấm nút `💾 LƯU (ZYYY)`):
     - Dữ liệu được ghi ngay lập tức lên Firebase RTDB tại node `/scan_records` và cập nhật vào bảng `/report_data`.
     - Phát chuông âm thanh **Success Chime** và hiển thị thông báo `SCAN-IN SAVE SUCCESSFUL!`.
     - Thêm một dòng mới vào bảng `INPUT LIST` bên dưới.
     - Xóa trắng form để sẵn sàng quét thùng tiếp theo.
   - Quét mã `ZZZZ` vào ô `DOC#` rồi ấn **Enter** (hoặc bấm nút `🗑️ XÓA (ZZZZ)`):
     - Hủy và làm mới phiên quét hiện tại.

---

## 📊 DANH MỤC 11 MÃ LỖI (DEFECT CODES)

| Mã lỗi | Tên tiếng Anh | Tên tiếng Việt |
|---|---|---|
| **E001** | Contamination | Dơ, Bụi Bẩn |
| **E002** | Bond gap | Hở Keo |
| **E003** | Cement | Dơ/Lem Keo |
| **E004** | Wrinkle/Crease/Mis-Shape | Nhăn/Nếp Gấp/Sai Hình Dáng |
| **E005** | Alignment L+R Symmetry | Canh Đối Xứng Trái/Phải |
| **E006** | Stitching/ Loose Thread | Lỗi May/Lỏng Chỉ |
| **E007** | Scratch/ Tear/ Rip/ High Buffing | Trầy/Rách/Lủng/Lộ Mài |
| **E008** | Interior (including sock-liner) | Lỗi Nội Quan/Bao Gồm (AnChang) |
| **E009** | Poor Quality Spray/ Paint | Xịt/Sơn Xấu |
| **E010** | Airbag | Lỗi Airbag |
| **E011** | Other/ Miscellaneous | Khác/Các Lỗi Khác |
| **E000** | None Defect | Không Lỗi |
| **ZYYY** | Save Record | Lưu Dữ Liệu |
| **ZZZZ** | Clear/Reset | Xóa / Làm mới |

---

## ☁️ CẤU TRÚC DỮ LIỆU TRÊN FIREBASE REALTIME DATABASE

- `/master_data`: 152 bản ghi dữ liệu mẫu chuẩn từ `DU LIEU MAU_ CHUAN.txt`.
- `/report_data`: 592 bản ghi lịch sử kiểm tra chất lượng từ `DU LIEU MAU.txt`.
- `/defect_codes`: 12 mã lỗi kiểm định chất lượng.
- `/scan_records`: Bản ghi thời gian thực được lưu khi quét `ZYYY`.
- `/users`: Danh sách tài khoản người dùng (`22194805` - HỒ XUÂN KHÁNH, `admin`).
