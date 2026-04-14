# PHIÊN BẢN BÀI LÀM CỦA Tran_Quang_Qui

## Phần 1.1 — Architecture Sketch
**Supervisor:** `HR_Orchestrator`
**Workers:** 
1. `Requirement_Extractor` (Trích xuất tiêu chuẩn)
2. `Skill_Gap_Analyzer` (Đánh giá chênh lệch)
3. `Content_Optimizer` (Tối ưu hóa nội dung)

**Message Flow (Sơ đồ giao tiếp):**
```text
( Ứng viên ) .~~> CV + JD .~~> ( HR_Orchestrator )
                                     |
    .~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~|~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~.
    |                                |                                |
   (V)                              (V)                              (V)
( Requirement_ )             ( Skill_Gap_ )                   ( Content_ ) 
(  Extractor   )             (  Analyzer  )                   ( Optimizer)
    |                                |                                |
    '~~> Trích xuất tiêu chuẩn ~~>   '~~> Điểm Match = 70% ~~>        '~~> Sinh CV tối ưu >
                                     | (Cần HITL)
                                     v
                           ( Duyệt bởi con người )
```
**Logic định tuyến (Routing):** `HR_Orchestrator` hoạt động dựa trên tham số cường độ (Intensity Score). Khi `Requirement_Extractor` trả về các keyword bắt buộc, `Skill_Gap_Analyzer` sẽ tính điểm Matching. Nếu dưới mức Threshold (ví dụ: < 60%), hệ thống báo thẳng cho con người duyệt. Nếu trên 60%, `Content_Optimizer` tự động nhào nặn lại câu chữ.

## Phần 1.2 — Pipeline vs. Supervisor-Worker
**Sự khác biệt chính:**
| Tiêu chí | Pipeline | Supervisor-Worker |
|----------|----------|-------------------|
| Scale | Rất khó mở rộng thêm module mới | Dễ dàng gắn các Worker mới (ví dụ Check lỗi chính tả) |
| Kiểm soát | Khó theo dõi trạng thái lỗi nội tại | Supervisor đóng vai trò Controller trung tâm |

**Use Case cụ thể trọng tâm (Xác thực Bằng cấp/Chứng chỉ):**
Trong Pipeline, nếu JD có "Cần bằng PMP" nhưng CV ghi chèn chữ "Đang học PMP", thuật toán Regex chặn nhầm. Supervisor AI sẽ hiểu ngữ cảnh và gửi tín hiệu cho ứng viên (HITL) hỏi "Bạn sẽ có bằng khi nào?" trước khi chèn vào Resume sửa đổi.

## Phần 3.1 & 3.2 — Domain Supervisor System & Pipeline
**Chi tiết Task (Test Load Balancing):**
1. "Xác định xem ứng viên có thỏa mãn điều kiện chứng chỉ PMP (hoặc tương đương) không từ file `CV_PM.pdf`."
2. "Sắp xếp lại thứ tự ưu tiên của các kỹ năng mềm/cứng để thể hiện đúng văn hóa cốt lõi công ty trong JD."
3. "Rút ngắn toàn bộ file CV xuống còn duy nhất 1 trang nhưng đảm bảo không mất keyword cốt lõi."

**Reflection (Góc nhìn Kỹ thuật - Tập trung vào Chi phí Token):**
Sử dụng kiến trúc đa tác nhân ở bài toán CV dễ dẫn đến việc lạm dụng Token (gửi đi gửi lại file nội dung CV dài cho nhiều Worker). Do đó, cần quy hoạch chặt: Supervisor chỉ gửi một phần cắt đoạn (chunk) của CV cho Worker tương ứng thay vì nguyên văn, giúp tối ưu chi phí hạ tầng (Token Cost Optimization).

## Phần 3.3 — MCP Tool Integration
1. **`Glassdoor_Salary_Culture_API`**: Nắm bắt văn hóa đãi ngộ của công ty ứng tuyển để nhét keyword văn hóa ẩn (Cultural Fit) vào CV.
2. **`Keyword_Density_Calculator`**: Công cụ thuần tính toán logic để đếm số lần xuất hiện keyword, tránh Spam từ khóa quá đà hỏng CV.
3. **`Notion_DB_Sync`**: Đẩy kết quả bao gồm Điểm Matching, Tình trạng khoảng trống và CV mới vào Notion lưu trữ cá nhân.
