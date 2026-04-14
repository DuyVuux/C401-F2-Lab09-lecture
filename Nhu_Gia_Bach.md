# PHIÊN BẢN BÀI LÀM CỦA Nhu_Gia_Bach

## Phần 1.1 — Architecture Sketch
**Supervisor:** `Talent_Acquisition_Lead`
**Workers:** 
1. `Job_Parsing_Agent` (Bóc tách thông tin JD)
2. `Experience_Matcher` (Đối chiếu kinh nghiệm)
3. `Resume_Tailor` (Chỉnh sửa và định dạng CV)

**Message Flow (Sơ đồ giao tiếp):**
```text
[ Người Dùng ] >>> Gửi dữ liệu đầu vào >>> [ Talent_Acquisition_Lead ]
                                                   ||
                   +===============================++=============================+
                   ||                              ||                             ||
                   \/                              \/                             \/
        [ Job_Parsing_Agent ]             [ Experience_Matcher ]           [ Resume_Tailor ]
        - Tách Keyword                    - So sánh số năm kinh nghiệm     - Sửa từ ngữ
        - Tách yêu cầu văn hóa            - Tìm dự án tương đương          - Chuẩn format
                   ||                              ||                             ||
                   +=========> Báo cáo <===========++=========> Phản hồi <========+
```
**Logic định tuyến (Routing):** `Talent_Acquisition_Lead` ưu tiên đánh giá format dữ liệu đầu vào. Tác vụ được song song hóa tối đa: `Job_Parsing_Agent` đọc JD trong khi `Experience_Matcher` parse CV. Nếu `Experience_Matcher` phát hiện ứng viên đủ điều kiện cơ bản mới điều phối cho `Resume_Tailor` tiến hành xào nấu nội dung.

## Phần 1.2 — Pipeline vs. Supervisor-Worker
**Sự khác biệt chính:**
| Tiêu chí | Pipeline | Supervisor-Worker |
|----------|----------|-------------------|
| Vai trò | Chỉ là các bước chức năng ghép nối | Tác nhân quản trị có logic độc lập, ra quyết định |
| Phản hồi | Không có cơ chế self-correction | Có khả năng đánh giá kết quả của Worker để làm lại |

**Use Case cụ thể trọng tâm (Trình bày Work Experience theo chuẩn):**
Giả sử kinh nghiệm làm việc ghi "Làm hệ thống bán hàng". `Talent_Acquisition_Lead` sẽ hướng dẫn Worker diễn đạt lại theo chuẩn STAR. Nếu output không có số liệu định lượng (Result), Supervisor bắt lỗi và yêu cầu Worker làm lại.

## Phần 3.1 & 3.2 — Domain Supervisor System & Pipeline
**Chi tiết Task (Test Load Balancing):**
1. "So sánh số năm kinh nghiệm làm Marketing yêu cầu trong JD với tổng thời gian làm việc thực tế trong `CV_Marketing.pdf`."
2. "Chuyển đổi các mô tả công việc cũ sang cấu trúc STAR (Situation, Task, Action, Result) chuyên nghiệp."
3. "Tìm và chèn tự nhiên các từ khóa liên quan đến phương pháp quản lý Agile/Scrum vào luồng dự án."

**Reflection (Góc nhìn Kỹ thuật - Tập trung vào Độ trễ / Latency):**
Mô hình này nhằm giải quyết bài toán thời gian (Latency). Bằng cách để `Job_Parsing_Agent` và sơ chế CV diễn ra song song (asynchronous), thời gian 60-90 phút/lần giảm xuống chỉ còn vài phút. Tuy nhiên, logic phân xử ở Supervisor sẽ cần tối ưu bằng prompt cực kỳ chặt chẽ để không gây nghẽn cổ chai.

## Phần 3.3 — MCP Tool Integration
1. **`GitHub_Repository_Analyzer`**: Tự động lấy danh sách công nghệ thực tế từ link GitHub ứng viên để bù đắp vào khoảng trống CV.
2. **`Plagiarism_Checker`**: Quét văn bản đầu ra xem các câu mô tả công việc có bị copy y hệt từ JD hay copy trên mạng không.
3. **`Docx_Generator`**: Cung cấp khả năng xuất trực tiếp file .docx hoàn chỉnh với heading style chuẩn, sẵn sàng để nộp.
