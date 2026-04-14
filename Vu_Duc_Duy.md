# PHIÊN BẢN BÀI LÀM CỦA Vu_Duc_Duy

## Phần 1.1 — Architecture Sketch
**Supervisor:** `Recruitment_Manager_Agent`
**Workers:** 
1. `JD_Analyzer` (Phân tích Yêu cầu công việc)
2. `CV_Gap_Detector` (Phát hiện khoảng trống năng lực)
3. `CV_Writer` (Viết lại CV chuẩn ATS)

**Message Flow (Sơ đồ giao tiếp):**
```text
+-----------------------+
|  User (Ứng viên)      |
+----------+------------+
           | (Gửi Master CV & JD)
           v
+-----------------------------+
| Recruitment_Manager_Agent   |
+----+-------------------+----+
     |                   |
     | 1. Phân phối JD   | 3. Giao CV & JD 
     v                   v
+----+---------+    +----+-------------+
| JD_Analyzer  |    | CV_Gap_Detector  |
+----+---------+    +----+-------------+
     | 2. Trả KQ         | 4. Trả Gap 
     +-------------------+ 
             |
             | 5. Chuyển thông tin Gap & JD
             v
+-----------------------------+
|        CV_Writer            | (6. Trả về CV đã tối ưu)
+-----------------------------+
```
**Logic định tuyến (Routing):** `Recruitment_Manager_Agent` sẽ đóng vai trò nhạc trưởng. Đầu tiên gửi JD cho `JD_Analyzer` để bóc tách keyword. Sau đó, nó điều phối bản nháp này cùng Master CV cho `CV_Gap_Detector` nhằm tìm ra điểm thiếu sót. Cuối cùng, kết quả phân tích được đẩy sang `CV_Writer` để sinh ra bản thảo cuối.

## Phần 1.2 — Pipeline vs. Supervisor-Worker
**Sự khác biệt chính:**
| Tiêu chí | Pipeline | Supervisor-Worker |
|----------|----------|-------------------|
| Luồng thực thi | Tuần tự, tĩnh | Động, có thể rẽ nhánh hoặc lặp lại |
| Xử lý lỗi | Dừng lại khi một bước hỏng | Supervisor có thể đánh giá lại và điều hướng tới Worker khác |

**Use Case cụ thể trọng tâm (Phần Skills):**
Nếu ứng viên thiếu quá nhiều Kỹ năng kỹ thuật (Technical Skills), Supervisor sẽ kích hoạt một quy trình hỏi đáp đệ quy để đào sâu vào các dự án phụ của ứng viên, tự động yêu cầu `CV_Writer` bổ sung kỹ năng nếu ứng viên xác nhận đã từng làm (HITL), thay vì chỉ chạy thẳng một khối tĩnh.

## Phần 3.1 & 3.2 — Domain Supervisor System & Pipeline
**Chi tiết Task (Test Load Balancing):**
1. "Phân tích JD và chỉ định rõ 3 kỹ năng lập trình (Technical Skills) bị thiếu sót trong file `CV_Dev.pdf`."
2. "Tối ưu hóa các gạch đầu dòng trong phần Kinh nghiệm làm việc để chứa mật độ từ khóa AWS và Python ở mức 15%."
3. "Viết lại Summary trong 3 câu ngắn gọn nhằm mục tiêu apply vị trí Senior Data Engineer."

**Reflection (Góc nhìn Kỹ thuật - Tập trung vào Độ Ngữ cảnh & Ảo giác):**
Hệ thống này đề cao tính chính xác (Accuracy). Cấu trúc Supervisor giúp kiểm soát chặt chẽ việc phát sinh ảo giác (Hallucination) từ LLM. `CV_Writer` không tự do bịa đặt kinh nghiệm mà bị ràng buộc ngặt nghèo bởi output của `CV_Gap_Detector`. Tuy nhiên, thời gian xử lý tổng thể sẽ cao hơn khi Supervisor phải thực hiện các bước kiểm tra chéo liên tục.

## Phần 3.3 — MCP Tool Integration
1. **`LinkedIn_Scraper`**: Thu thập dữ liệu public của ứng viên để tự động bổ sung dự án nếu Master CV quá sơ sài.
2. **`ATS_Score_API`**: Chấm điểm CV qua thuật toán ATS thật bên thứ ba (như ResumeWorded) để lấy điểm % khách quan.
3. **`PDF_Text_Extractor`**: Parse các file Master CV phức tạp, nhiều định dạng cột/bảng mà không làm mất cấu trúc nội dung.
