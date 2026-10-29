# DoAnThucTapTN
kiem tra chất lượng
# TỔNG QUAN HỆ THỐNG VÀ KIẾN TRÚC DỰ ÁN (PROJECT CHEAT SHEET)
> **Mục đích tài liệu:** File này tóm tắt toàn bộ bối cảnh, yêu cầu nghiệp vụ, cấu trúc dữ liệu Firebase, mã nguồn và luồng hoạt động của dự án. Khi người dùng yêu cầu đọc file này, AI Assistant có thể hiểu ngay lập tức 100% dự án mà không cần phải phân tích lại từ đầu.

---

## 1. THÔNG TIN CHUNG DỰ ÁN
- **Tên dự án:** Hệ thống Quản Lý & Kiểm Tra Chất Lượng Sản Phẩm (3iZ Quality Inspection & Barcode Scanning System).
- **Mã hệ thống / Phiên bản:** `TK_VT 3.0`
- **Nguồn tài liệu gốc:** File mô tả `file_bai_lam.docx`, dữ liệu nguồn `DU LIEU MAU_ CHUAN.txt` và `DU LIEU MAU.txt`.
- **Cơ sở dữ liệu Online:** Firebase Realtime Database (RTDB)
- **URL Firebase Database:**
  `https://doanthuctap-41ccc-default-rtdb.asia-southeast1.firebasedatabase.app/`

---

## 2. CẤU TRÚC DỮ LIỆU TRÊN FIREBASE REALTIME DATABASE

Toàn bộ dữ liệu của hệ thống đã được đồng bộ hóa và lưu trữ trực tiếp trên Firebase RTDB thông qua REST API:

1. **`/master_data` (152 bản ghi Master Data chuẩn từ `DU LIEU MAU_ CHUAN.txt`):**
   - Key: Mã `UCC_LABEL` (Ví dụ: `9929197877`, `9929220764`...)
   - Cấu trúc 11 trường:
     ```json
     {
       "UCC_LABEL": "9929197877",
       "STYLE_NO": "HQ4307-201",
       "POID": "20260630",
       "SIZE": "08M",
       "UPC_LABEL": "199778495665",
       "MODEL": "NIKE VOMERO 18",
       "MSR": "6208355860500",
       "INV": "VTF26033332207Z",
       "QTY": 6,
       "CARTON": 1,
       "COUNTRY": "USA"
     }
     ```

2. **`/report_data` (Dữ liệu kiểm tra thực tế lưu từ màn hình 3IZ SCAN):**
   - Dữ liệu tab 3IZ REPORT được khởi tạo trống và chỉ hiển thị khi kiểm định viên quét và lưu tại màn hình 3IZ SCAN.
   - Key: ID bản ghi (Ví dụ: `SCAN_...`)
   - Cấu trúc: `Factory Group`, `PLANT`, `SECTION`, `DATE`, `UCC`, `STYLE NO`, `MODEL`, `PO NUMBER`, `INV`, `3iZ QTY`, `CARTON QTY`, `WH QTY`, `DEST`, `COUNTRY`, `INPUT TIME`, `OUTPUT TIME`, `INSPECTION TIME (MINUTES)`, `Total defective qty`, và 11 cột lỗi.

3. **`/defect_codes` (12 mã lỗi kiểm tra chất lượng):**
   - `E001` -> `E011` và `E000` (Không lỗi).

4. **`/scan_records` (Dữ liệu scan phát sinh trực tiếp theo thời gian thực):**
   - Tự động ghi nhận khi kiểm định viên quét `ZYYY`.

5. **`/users` (Tài khoản người dùng / kiểm định viên):**
   - User ID: `22194805` (HỒ XUÂN KHÁNH - FCT VT1)
   - User ID: `admin` (Administrator)

---

## 3. BẢNG 11 MÃ LỖI (DEFECT CODES) & THẺ MÃ VẠCH (ẢNH image1.png)

| Mã Barcode | Tên Tiếng Anh | Tên Tiếng Việt | Ghi chú |
|---|---|---|---|
| **E001** | Contamination | Dơ, Bụi Bẩn | Lỗi ngoại quan |
| **E002** | Bond gap | Hở Keo | Lỗi liên kết đế |
| **E003** | Cement | Dơ/Lem Keo | Lem keo trên thân |
| **E004** | Wrinkle/Crease/Mis-Shape | Nhăn/Nếp Gấp/Sai Hình Dáng | Sai form dáng |
| **E005** | Alignment L+R Symmetry | Canh Đối Xứng Trái/Phải | Lệch đối xứng |
| **E006** | Stitching/ Loose Thread | Lỗi May/Lỏng Chỉ | Lỗi đường may |
| **E007** | Scratch/ Tear/ Rip/ High Buffing | Trầy/Rách/Lủng/Lộ Mài | Lỗi bề mặt da/vải |
| **E008** | Interior (including sock-liner) | Lỗi Nội Quan/Bao Gồm (AnChang) | Lót trong giày |
| **E009** | Poor Quality Spray/ Paint | Xịt/Sơn Xấu | Lỗi nước sơn đế |
| **E010** | Airbag | Lỗi Airbag | Lỗi đệm khí Air |
| **E011** | Other/ Miscellaneous | Khác/Các Lỗi Khác | Lỗi khác |
| **E000** | None Defect | Không Lỗi | Hàng đạt chuẩn |
| **ZYYY** | Save Barcode | **Lưu Dữ Liệu** | Lưu vào Firebase |
| **ZZZZ** | Clear/Reset Barcode | **Xóa / Hủy** | Làm mới màn hình |

---

## 4. QUY TRÌNH QUÉT HÀNG 2 BƯỚC IN/OUT (SCAN PIPELINE MỚI)

Màn hình Scan có ô nhập duy nhất: `DOC# OR PCARD SCAN` (nhận barcode từ súng quét USB/Bluetooth hoặc bàn phím + phím **Enter**):

1. **Bước 1 (Lần đầu scan trong ngày - Status IN):**
   - Quét mã UCC (ví dụ `9929197877`) vào ô `DOC#` và nhấn Enter.
   - Hệ thống tra cứu Master Data, nhận diện UCC chưa scan trong ngày:
     - Ô `SCAN STATUS`: Hiển thị **IN** (màu xanh lá `#16a34a`).
     - Tự động điền: `UCC LABEL`, `INVOICE`, `DEST`, `QTY`, `STYLE / MODEL`.
     - Ô `DEFECT`: Hiển thị `🟢 STATUS: IN (KHÔNG CẦN DEFECT - QUÉT ZYYY ĐỂ LƯU)`.
   - **Lưu Scan IN:** Quét `ZYYY` (hoặc bấm nút `💾 LƯU (ZYYY)`) **ngay mà không cần nhập defect**.
     - Hệ thống lưu vào Firebase RTDB `/scan_records` với `scanType: 'IN'`, ghi nhận `INPUT TIME`, thêm dòng vào `INPUT LIST` với badge `IN`, phát chuông **Success Chime** và làm mới form.

2. **Bước 2 (Scan lại lần 2 trong ngày - Status OUT):**
   - Khi kiểm tra xong sản phẩm trong thùng carton, quét lại mã UCC đó lần thứ 2:
   - Hệ thống phát hiện UCC đã có lượt scan IN trong ngày:
     - Ô `SCAN STATUS`: Tự động chuyển sang **OUT** (màu đỏ cam `#dc2626`).
     - Ô `DEFECT`: Hiển thị `⏳ BƯỚC 2: HÃY QUÉT MÃ LỖI (E000 - E011)...`.
     - Thông báo: `"UCC (Lần 2 - OUT). Vui lòng quét mã lỗi (E000-E011)!"`.
   - **Bắt buộc nhập defect:** Quét mã lỗi từ `E001` - `E011` (hoặc `E000 - Không Lỗi`) vào ô `DOC#` hoặc click trên bảng lỗi.
   - **Lưu Scan OUT:** Quét `ZYYY` để lưu:
     - *Ràng buộc:* Nếu chưa quét mã lỗi mà ấn lưu `ZYYY`, hệ thống báo lỗi **"Hãy scan defect trước khi lưu!"** và từ chối lưu.
     - *Khi hợp lệ (đã có defect):* Lưu vào `/scan_records` (`scanType: 'OUT'`), tự động tính `INSPECTION TIME (MINUTES)` giữa `INPUT TIME` và `OUTPUT TIME`, đẩy bản ghi vào `/report_data` và cập nhật tức thì tab **3IZ REPORT** cùng **6 Biểu đồ 3IZ Analysis**!

3. **Cảnh báo khi đã scan OUT rồi mà scan lại:**
   - Nếu UCC thùng carton này đã hoàn thành scan OUT trong ngày rồi mà kiểm định viên quét lại lần nữa:
     - Hệ thống phát âm thanh lỗi **Error Buzz**.
     - Cảnh báo: **"CẢNH BÁO: UCC thùng carton này đã scan kiểm tra rồi!"** (từ chối quét lại).

4. **HỦY / LÀM MỚI (ZZZZ):**
   - Quét mã `ZZZZ` bất kỳ lúc nào để hủy lượt quét hiện tại và làm mới form.

---

## 5. CẤU TRÚC TỆP TIN & KIẾN TRÚC MÃ NGUỒN

```
THUC_TAP_TOT_NGHIEP/
│
├── web.md                    # File tóm tắt dự án dành cho AI & Developer (File này)
├── README.md                 # Tài liệu hướng dẫn sử dụng chi tiết cho người dùng
├── file_bai_lam.docx         # Đề bài và mô tả yêu cầu ban đầu
├── DU LIEU MAU_ CHUAN.txt     # Dữ liệu 153 dòng master data gốc
├── DU LIEU MAU.txt           # Dữ liệu 593 dòng báo cáo kiểm định gốc
│
├── index.html                # CỔNG TỔNG HỢP (All-in-One Portal) với tab system
├── scan.html                 # Màn hình 3IZ SCAN độc lập (Ảnh image2.png)
├── report.html               # Màn hình 3IZ REPORT độc lập + Xuất Excel (Ảnh image3.png)
├── chart.html                # Màn hình 3IZ Analysis độc lập + 6 Biểu đồ (Ảnh image4.png)
├── master_data.html          # Màn hình Master Data 11 cột độc lập
├── defects.html              # Bảng 11 Thẻ Mã Vạch Lỗi độc lập hỗ trợ In A4 (Ảnh image1.png)
├── login.html                # Form Đăng nhập kiểm định viên (FCT mặc định VT1)
└── (Toàn bộ CSS, JavaScript, âm thanh Web Audio đã được nhúng trực tiếp 100% vào từng file .html, các file hoàn toàn độc lập và nhẹ gọn)
```

---

## 6. DANH SÁCH 6 BIỂU ĐỒ TRÊN MÀN HÌNH 3IZ ANALYSIS (CHART.JS)
Tất cả 6 biểu đồ đều được tính toán và vẽ tự động theo thời gian thực trực tiếp từ dữ liệu của tab **3IZ REPORT**:

1. **`chartQtyByPlant` (INSPECTION QTY BY FCT):** Cột đứng so sánh tổng sản lượng kiểm tra theo phân xưởng FCT (`VT1`, `VT2`, `VT3`...).
2. **`chartTopDefectByModel` (TOP 5 DEFECT QTY BY MODEL):** Thanh ngang xếp hạng Top 5 mẫu giày có nhiều lỗi nhất từ các lượt quét thực tế.
3. **`chartAvgInspectionTime` (AVG INSPECTION TIME BY FCT):** Thời gian kiểm tra trung bình (phút) theo từng phân xưởng FCT.
4. **`chartDefectByDest` (TOP 4 DEFECT BY DEST):** Biểu đồ cột nhóm (Grouped Bar) phân tích 4 loại lỗi hàng đầu theo thị trường xuất khẩu thực tế.
5. **`chartQtyByDest` (INSPECTION QTY BY DEST):** Thanh ngang tổng sản lượng kiểm tra thực tế theo thị trường đến.
6. **`chartTopDefects` (TOP 4 DEFECT):** Cột đứng 4 lỗi có tần suất phát sinh cao nhất dựa trên dữ liệu kiểm tra thực tế.

---

## 7. CÁC THƯ VIỆN BÊN NGOÀI ĐƯỢC TÍCH HỢP QUA CDN
- **Chart.js** (`cdn.jsdelivr.net/npm/chart.js`): Vẽ 6 biểu đồ sắc nét, tương tác mượt mà.
- **JsBarcode** (`cdn.jsdelivr.net/npm/jsbarcode@3.11.5/dist/JsBarcode.all.min.js`): Tạo mã vạch chuẩn Code128 dạng SVG cho toàn bộ thẻ lỗi, ZYYY, ZZZZ.
- **SheetJS / xlsx** (`cdn.jsdelivr.net/npm/xlsx@0.18.5/dist/xlsx.full.min.js`): Xuất trực tiếp file Excel `.xlsx` từ bảng báo cáo.

---

## 8. HƯỚNG DẪN DÀNH CHO AI TRONG CÁC PHIÊN LÀM VIỆC TIẾP THEO
- Khi người dùng hỏi hoặc yêu cầu chỉnh sửa/mở rộng bất kỳ tính năng nào:
  1. Đọc nhanh file [web.md](file:///e:/THUC_TAP_TOT_NGHIEP/web.md) này để nắm toàn bộ ngữ cảnh.
  2. Toàn bộ mã CSS (`css/app.css`) và Javascript (`initial_data.js`, `sound-service.js`, `firebase-service.js`, `app.js`) đã được nhúng trực tiếp (`inline`) vào các file `.html` (`<style>` và `<script>`), giúp mỗi trang hoàn toàn độc lập và không phụ thuộc file ngoài.
  3. Mọi thao tác lưu/đọc dữ liệu đều đi qua `FirebaseService`.
  4. **Nút "🗑️ Xóa dữ liệu Report trong ngày":** Chỉ xóa các bản ghi scan/kiểm tra được tạo trong ngày hiện tại (`todayStr`), bảo toàn dữ liệu các ngày cũ cả trên Firebase RTDB và `localStorage`.
  5. Mọi logic quét mã vạch và xử lý 3 bước tuân thủ chuẩn quy trình kiểm định chất lượng 3iZ.
