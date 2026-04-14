# PHIÊN BẢN BÀI LÀM CỦA Hoang_Vinh_Giang

## Phần 1.1 — Architecture Sketch
**Supervisor:** `CV_Optimization_Controller`
**Workers:** 
1. `Context_Analyzer` (Đọc ngữ cảnh lõi)
2. `ATS_Validator` (Trình xác thực quy tắc ATS)
3. `Drafting_Agent` (Tác nhân soạn thảo)

**Message Flow (Sơ đồ giao tiếp):**
```text
  { USER INPUT } ===> { CV_Optimization_Controller }
                               ||
          /====================++====================\
         ||                                          ||
 { Context_Analyzer } ===> { Drafting_Agent } ===> { ATS_Validator }
 (Tìm Key Insight)         (Viết nội dung)         (Chấm điểm)
                                                     ||
                                            (Fail) <==/
                                              ||
                                   (Bắt Drafting viết lại)
```
**Logic định tuyến (Routing):** `CV_Optimization_Controller` sử dụng luồng Feedback Loop kín. Khi `Drafting_Agent` sản xuất một bản CV, nó không trả thẳng cho người dùng mà đi qua `ATS_Validator`. Nếu ATS Score < 80%, Controller vạch lỗi và yêu cầu Drafting_Agent làm vòng 2.

## Phần 1.2 — Pipeline vs. Supervisor-Worker
**Sự khác biệt chính:**
| Tiêu chí | Pipeline | Supervisor-Worker |
|----------|----------|-------------------|
| State | Không giữ trạng thái toàn cục | Controller nhớ các lần thất bại để điều chỉnh |
| Độc lập | Các Node phụ thuộc chặt chẽ dữ liệu | Các Worker có LLM/Prompt chuyên biệt riêng |

**Use Case cụ thể trọng tâm (Tinh chỉnh Mô tả Dự án):**
Một dòng dự án Backend Nodejs viết nghèo nàn. Thay vì chèn bừa từ khoá (như Pipeline thường làm), Worker `Drafting_Agent` sẽ dựa theo Context về dự án tương đồng để đề xuất các thông số kỹ thuật (vd: xử lý x requests/s). Controller gửi đề xuất này hệt như một Comment Word để ứng viên Review.

## Phần 3.1 & 3.2 — Domain Supervisor System & Pipeline
**Chi tiết Task (Test Load Balancing):**
1. "Đánh giá mức độ chuyên sâu của các dự án cá nhân đối chiếu với tiêu chuẩn tuyển dụng Backend Engineer Mid-level."
2. "Bổ sung (Tự đề xuất đóng trong ngoặc vuông) các chỉ số đo lường hiệu suất (metrics) dựa trên văn cảnh 'Tăng tốc độ truy vấn data' trong CV."
3. "Dịch chuẩn xác toàn bộ bản CV từ tiếng Việt sang tiếng Anh theo ngữ pháp học thuật chuyên ngành IT."

**Reflection (Góc nhìn Kỹ thuật - Centralized Human-in-the-Loop):**
Kiến trúc này cho thấy điểm sáng ở giao diện tương tác với con người (HCI). Thay vì bắt hệ thống AI gánh 100% quyết định, hệ thống được thiết kế để dừng (pause execution) và trả về một Draft với các Comment (highlight vàng). Ứng viên (HITL) đưa lệnh đồng ý/chỉnh sửa, Controller mới chốt kết quả. Điều này giảm triệt để rủi ro rớt phỏng vấn vì CV quá ảo.

## Phần 3.3 — MCP Tool Integration
1. **`Grammarly_API_Integration`**: Chạy ngầm để đảm bảo ngữ pháp Tiếng Anh không có lỗi typo hoặc cấu trúc câu cụt lủn.
2. **`Web_Search_Company_Culture`**: Crawler tìm kiếm tin tức, giá trị cốt lõi trên website tuyển dụng của công ty để "điều hướng giọng văn" cho CV_Writer.
3. **`Email_CoverLetter_Crafter`**: Từ việc đã nắm rõ nội dung CV và JD, tool này tự động viết luôn cả nội dung thư ngỏ (Cover Letter) đính kèm.
