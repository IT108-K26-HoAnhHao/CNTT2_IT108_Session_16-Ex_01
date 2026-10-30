# BÀI 1
## I. BÁO CÁO PHÂN TÍCH (DATA FLOW & CONFLICT RESOLUTION)

### 1. Sơ đồ luồng dữ liệu từ Local SSD lên Cloud (Hybrid Workflow)

Mô hình Hybrid Workflow đảm bảo dữ liệu thiết kế 3D luôn được ghi đồng thời xuống ổ cứng vật lý (Local) và lập tức đồng bộ ngầm lên Cloud lưu trữ (OneDrive, Google Drive, Dropbox) ngay khi Designer nhấn `Ctrl + S`.

```text
      [ Phần mềm Thiết kế 3D ]
                 │
                 ▼ (Nhấn Ctrl + S)
  [ Ổ đĩa Local (Thư mục đồng bộ) ] ◄─── (Ghi dữ liệu vật lý xuống SSD)
                 │
                 ▼ (Ứng dụng Cloud Agent phát hiện thay đổi)
   [ Bộ đệm Tạm thời (Local Cache) ]
                 │
                 ▼ (Kiểm tra trạng thái Internet)
           ┌─────┴─────┐
           ▼ Có        ▼ Không (Mất mạng / Sập nguồn)
     [ Đẩy lên Cloud ]   [ Giữ trong hàng đợi Queue ]
           │                           │
           │                           ▼
           │                 (Khi khôi phục kết nối)
           └───────────────────────────┤
                                       ▼
                          [ Đám mây (Cloud Storage) ]
                                       │
                                       ▼
                          [ Bật Version History ]
```



### 2. Phương án xử lý khi có xung đột phiên bản (File Version Conflict)

Xung đột xảy ra khi hai người cùng chỉnh sửa một file tại một thời điểm hoặc khi bản Local chưa đồng bộ xong cấu trúc cũ đã bị đè bởi bản mới từ Cloud. Hệ thống sẽ xử lý theo cơ chế rẽ nhánh (Forking) an toàn:

*   **Nguyên tắc không ghi đè tự động:** Cloud Agent giữ nguyên file gốc ổn định trên Cloud để bảo vệ công sức của người khác. Đồng thời, hệ thống tạo ra một file phụ tại Local đính kèm tên máy/tên tài khoản và mốc thời gian xung đột.
    *   *Ví dụ đặt tên:* `Model_Co_Vat_Conflict_BanA_20261030.obj`
*   **Quy trình giải quyết:**
    1.  **Cảnh báo (Alert):** Hệ thống gửi thông báo pop-up cho Designer về việc xuất hiện file xung đột.
    2.  **Hợp nhất thủ công (Manual Review):** Designer mở cả hai phiên bản để đối chiếu các chi tiết thay đổi trong không gian 3D.
    3.  **Xác nhận phiên bản cuối (Final Commit):** Chọn phiên bản hoàn thiện nhất, đổi tên về dạng chuẩn và nhấn đồng bộ lại để ghi nhận bản cuối cho toàn đội nhóm. Các bản xung đột thừa sẽ đưa vào thùng rác (Cloud Trash) và được giữ lại trong 30 ngày để dự phòng.

---

## II. QUY TRÌNH THỰC THI (3 BƯỚC THIẾT LẬP CHO DESIGNER)

Để loại bỏ hoàn toàn rủi ro mất mát dữ liệu do hỏng ổ cứng (Bad Sector) hoặc thói quen gửi file qua Zalo bị hết hạn, mỗi Designer cần thực hiện nghiêm túc 3 bước thiết lập sau:

### Bước 1: Cài đặt và Định tuyến Thư mục Dự án (Folder Mapping)
*   Tải và cài đặt ứng dụng đồng bộ Cloud dành cho máy tính (ví dụ: *OneDrive for Desktop* hoặc *Google Drive for Desktop*).
*   Không lưu file bừa bãi tại màn hình (`Desktop`) hoặc ổ `C:\`. Hãy tạo riêng một thư mục làm việc chuyên dụng trên phân vùng ổ đĩa an toàn (Ví dụ: `D:\PTIT_Museum_Project`).
*   Trong cài đặt của ứng dụng Cloud, chọn tính năng **"Sync this folder"** để liên kết trực tiếp thư mục `D:\PTIT_Museum_Project` với tài khoản đám mây của dự án.

### Bước 2: Cấu hình Chế độ Lưu trữ và Kích hoạt Version History
*   Bấm chuột phải vào thư mục dự án vừa thiết lập, chọn chế độ **"Always keep on this device"** (Luôn giữ trên thiết bị này). 
    *   *Mục đích:* Đảm bảo file thiết kế nặng luôn có sẵn ở ổ cứng Local để phần mềm 3D truy xuất mượt mà, render không bị trễ, trong khi quá trình backup lên Cloud vẫn tự động chạy ngầm.
*   Đăng nhập vào trang quản trị mạng Cloud để kiểm tra và kích hoạt tính năng **Version History** (đảm bảo cấu hình lưu giữ tối thiểu 25 - 50 phiên bản lịch sử gần nhất).

### Bước 3: Đồng bộ tính năng Tự động lưu của Phần mềm 3D (Auto-Save Alignment)
*   Mở phần mềm thiết kế 3D đang sử dụng (Blender, 3ds Max, Maya, v.v.).
*   Truy cập vào mục Cài đặt cấu hình (`Preferences` -> `File / Auto-Save`).
*   Bật tính năng Tự động lưu (`Auto-Save` / `Backup`) và trỏ đường dẫn lưu file sao lưu tự động về **đúng thư mục đã thiết lập đồng bộ ở Bước 1**. Cài đặt tần suất tự động lưu từ **10 - 15 phút/lần**.

---

## III. GIẢI QUYẾT BẰY DỮ LIỆU (DATA TRAP RESOLUTION)

**Tình huống bẫy:** *Làm thế nào để đảm bảo dữ liệu không bị đứt đoạn đồng bộ khi máy tính đột ngột mất kết nối Internet hoặc bị sập nguồn giữa chừng?*

Hệ thống Hybrid Workflow xử lý rủi ro này bằng các cơ chế tự động bảo vệ tính toàn vẹn của dữ liệu:

### 1. Khi máy tính đột ngột mất kết nối Internet
*   Hệ thống áp dụng cơ chế **Local Cache Pending Queue**. Khi Designer nhấn `Ctrl + S`, file thiết kế được ghi thành công và trọn vẹn xuống ổ cứng Local SSD trước.
*   Cloud Agent ghi nhận file ở trạng thái "Chờ đồng bộ" (Pending). Ngay khi máy tính có kết nối Internet trở lại (dù là mạng Wi-Fi hay mạng dây di động), hệ thống sẽ tự động so sánh mã băm (MD5 Hash) giữa Local và Cloud, nhận diện phân đoạn dữ liệu thay đổi và tiếp tục tải phần còn thiếu lên đám mây (**Resume Upload**) mà không làm lỗi file hay tốn băng thông tải lại từ đầu.

### 2. Khi máy tính bị sập nguồn đột ngột (Mất điện, hỏng nguồn giữa lúc đang lưu)
*   **Bảo vệ tại Local:** Các phần mềm 3D chuyên nghiệp và hệ điều hành sử dụng cơ chế ghi tạm (`Shadow Copy` / `.tmp file`). File cũ chỉ bị thay thế khi file mới được ghi thành công 100% xuống đĩa cứng. Nếu sập nguồn đúng giây đang lưu, file mới có thể hỏng nhưng file ở lượt lưu `Ctrl + S` liền trước đó vẫn nguyên vẹn.
*   **Bảo vệ trên Cloud bằng Version History:** Do tiến trình đẩy 