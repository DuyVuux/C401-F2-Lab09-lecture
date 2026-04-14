# PHIÊN BẢN BÀI LÀM CỦA Doan_Nam_Son

## Phần 1.1 — Architecture Sketch
**Supervisor:** `Resume_Mastermind`
**Workers:** 
1. `Position_Decoder` (Giải mã Vị trí ứng tuyển)
2. `Profile_Auditor` (Kiểm toán thông tin CV)
3. `ATS_Bypass_Writer` (Sinh mã lách ATS)

**Message Flow (Sơ đồ giao tiếp):**
```text
           [ CV Data ] + [ Job Link ] 
                        V
             < Resume_Mastermind >
              /        |         \
             /         |          \
            v          v           v
  <Position_Decoder>   |           |
 (Giải mã Yêu cầu)     |           |
                       v           |
               <Profile_Auditor>   |
              (Chỉ ra 10 lỗi sai)  |
                                   v
                         <ATS_Bypass_Writer>
                        (Tái cấu trúc Template)
                                   |
                                   v
                             [ Final CV ]
```
**Logic định tuyến (Routing):** `Resume_Mastermind` lập một Action Plan động từ đầu. Nếu Job Link quy định tuyển dạng Senior, nó kích hoạt thêm bộ thẩm định kỹ năng cứng của `Profile_Auditor`. Nó truyền Context qua từng bước một theo kiểu Waterfall, nhưng bất kỳ Worker nào phát hiện CV gốc quá thiếu thông tin, nó báo Supervisor dừng quy trình và hỏi user cung cấp thêm.

## Phần 1.2 — Pipeline vs. Supervisor-Worker
**Sự khác biệt chính:**
| Tiêu chí | Pipeline | Supervisor-Worker |
|----------|----------|-------------------|
| Chuyển giao dữ liệu| Fixed schema (phải giống nhau cấu trúc) | Dynamic schema (Mastermind giao thứ gì Worker cần) |
| Ứng dụng | Phù hợp Data parsing cơ bản | Xử lý tốt các ca "thiếu dữ liệu" từ user |

**Use Case cụ thể trọng tâm (Viết Personal Summary & Objective):**
Rất nhiều ứng viên chỉ có 1 đoạn Summary chung chung. Supervisor sẽ gửi lệnh cho `ATS_Bypass_Writer` viết riêng 3 tùy chọn Summary khác nhau (Tập trung Kỹ thuật / Tập trung Kinh doanh / Tập trung Lãnh đạo) dựa trên văn phong JD để ứng viên tự chọn phương án "thuận miệng" nhất.

## Phần 3.1 & 3.2 — Domain Supervisor System & Pipeline
**Chi tiết Task (Test Load Balancing):**
1. "Dựa trên đầu vào JD_ProductOwner.txt, hãy tạo ra 3 kiểu viết Summary Profile khác biệt để ứng viên lựa chọn."
2. "Truy xuất và loại bỏ hoàn toàn các từ lóng (buzzwords) sáo rỗng trong các dòng mô tả của ứng viên và thay thế bằng từ ngữ đo lường được."
3. "Xác thực mô phỏng thuật toán ATS để duyệt lại lần cuối xem văn bản đã đat mức tối thiểu 85% keyword phù hợp chưa."

**Reflection (Góc nhìn Kỹ thuật - Sự Kháng lỗi và Độ bền/Robustness):**
Điểm nhấn của Supervisor System này là Robustness. Người dùng rất hay upload Master CV dạng ảnh (Image/PDF rác xước), chữ bị lỗi font. Supervisor trước khi làm bất kỳ logic hiểu nào, có thể xử lý tiền lệ (pre-processing agent). Khả năng xử lý edge cases (ví dụ CV không chia rõ mục kinh nghiệm hay học vấn) sẽ tốt hơn hẳn Pipeline code cứng truyền thống.

## Phần 3.3 — MCP Tool Integration
1. **`Design_Template_Applier`**: Khớp dữ liệu đã xử lý vào các file `.tex` (LaTeX) hoặc file HTML để xuất PDF đẹp chuẩn quốc tế 1 trang.
2. **`Certification_Verify_API`**: Cross-check những chứng chỉ AWS/GCP (qua ID) ứng viên vứt vào xem còn thời hạn trước khi dám bế nguyên từ khóa đó vào bản CV cuối.
3. **`LinkedIn_Job_Scraper`**: Nhận input chỉ là URL công việc Linkedin thay vì bắt user nhọc công copy-paste tay nguyên một khối văn bản JD khổng lồ.
