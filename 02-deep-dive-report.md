# 02 - Deep-Dive Report

Tên nhóm: **aithucchien**

## Thành viên

| # | Họ và tên | MSSV |
|---|---|---|
| 1 | Trần Đình Đăng | 2A202601998 |
| 2 | Nguyễn Trung Hiếu | 2A202601457 |
| 3 | Nguyễn Thế Anh | 2A202601791 |
| 4 | Nguyễn Thị Lý | 2A202601963 |
| 5 | Vũ Văn Phong | 2A202601647 |

## Quyết định lựa chọn

Nhóm chọn bài toán: **Xanh SM dispatcher co-pilot cho sự cố pin yếu/hết pin của taxi điện**.

Phạm vi prototype là hỗ trợ điều phối viên tạo bản nháp hướng dẫn hoặc lệnh điều phối dạng JSON. AI chỉ đóng vai trò co-pilot, không tự động gửi tin nhắn, không tự điều xe thật, không xác nhận rằng hành động đã được thực thi, và không được bỏ qua ranh giới an toàn liên quan đến mức pin.

## Phase 3.1 - Current-State Workflow Mapping

Quy trình hiện tại phụ thuộc nhiều vào thao tác thủ công của điều phối viên. Khi số lượng xe báo pin yếu tăng trong giờ cao điểm, các bước tra cứu và soạn tin dễ trở thành bottleneck.

```text
Tài xế báo sự cố pin.
  |
  v
[1] Điều phối viên nhận cuộc gọi và ghi log sự cố. (2 phút)
  | Handoff: Tài xế -> Tổng đài/Điều phối viên.
  v
[2] Điều phối viên tra vị trí GPS, biển số, loại xe và mức pin trên dashboard nội bộ. (2 phút)
  | Handoff: Điều phối viên -> Hệ thống bản đồ/telemetry.
  v
[3] Điều phối viên tìm trạm sạc VinFast còn trụ trống, đúng loại cổng sạc. (5 phút) [BOTTLENECK]
  | Handoff: Điều phối viên -> Dashboard trạm sạc.
  v
[4] Điều phối viên soạn tin nhắn hướng dẫn đường đi cho tài xế. (5 phút) [BOTTLENECK]
  | Handoff: Điều phối viên -> App tài xế.
  v
[5] Nếu pin quá thấp, điều phối viên liên hệ đội xe sạc/cứu hộ di động. (1 phút)

Tổng thời gian trung bình: 15 phút/lượt.
```

Các điểm nghẽn chính nằm ở bước 3 và bước 4. Đây là các bước cần vừa chính xác về dữ liệu vận hành, vừa rõ ràng về ngôn ngữ hướng dẫn. Nếu điều phối viên chọn sai trạm hoặc bỏ qua ngưỡng pin dưới 5%, xe có thể hết pin giữa đường, gây rủi ro cho tài xế, khách hàng và giao thông xung quanh.

## Phase 3.2 - Problem Statement 6-field

| Field | Nội dung chi tiết |
|---|---|
| 1. Actor / Operator | Điều phối viên Trung tâm Điều vận Xanh SM. Đây là người tiếp nhận sự cố pin, tra cứu dữ liệu vận hành, đưa ra hướng xử lý ban đầu và phê duyệt nội dung gửi cho tài xế. |
| 2. Current Workflow | Quy trình hiện tại gồm: nghe cuộc gọi, ghi log, tra GPS/biển số/mức pin, mở dashboard trạm sạc, chọn trạm phù hợp, soạn tin hướng dẫn, và kích hoạt đội xe sạc/cứu hộ di động nếu cần. Phần lớn thao tác vẫn làm thủ công, phụ thuộc kinh nghiệm từng điều phối viên. |
| 3. Bottleneck | Bước tìm trạm sạc phù hợp và soạn hướng dẫn mất khoảng 10 phút/lượt. Trong trường hợp pin dưới 5%, việc gợi ý trạm tiêu chuẩn xa hơn 5 km là nguy hiểm vì xe có thể không đến nơi an toàn. |
| 4. Business Impact | Ước tính 80 sự cố pin/ngày tại các khu vực đô thị lớn. Với 15 phút/lượt, đội điều phối mất khoảng 20 giờ công/ngày chỉ cho nhóm sự cố này. Hệ quả là xe dừng chờ lâu hơn, tài xế căng thẳng, số chuyến có thể nhận giảm, và trải nghiệm khách hàng bị ảnh hưởng nếu xe đến đón muộn hoặc phải hủy chuyến. |
| 5. Success Metric | Giảm thời gian xử lý từ 15 phút xuống dưới 3 phút/lượt. Đạt 98% bản nháp đúng loại trạm, đúng khoảng cách và đúng rule pin. 100% nội dung gửi ra ngoài phải có điều phối viên phê duyệt. Không có trường hợp AI gợi ý trạm xa hơn 5 km khi pin dưới 5%. |
| 6. Operational Boundary | AI được phép đọc input có cấu trúc về biển số, tọa độ, mức pin, loại xe, trạm gần nhất và khoảng cách. AI được tạo bản nháp bắt đầu bằng `[DRAFT_ONLY]`. Nếu pin dưới 5% và trạm tiêu chuẩn xa hơn 5 km, AI phải trả JSON `{"action": "dispatch_mobile_charger", "reason": "..."}`. AI không được tự động gửi tin, không được nói rằng đã điều xe, không được gợi ý tuyến nguy hiểm, không được bỏ qua Human-in-the-loop. |

## Phase 3.3 - Future-State Flow & AI Fit

AI Fit được chọn: **LLM Feature + deterministic safety rules**.

Nhóm không chọn full Agentic Loop trong phase đầu vì hành động điều xe, gửi tin nhắn và điều phối cứu hộ đều có rủi ro vận hành. LLM phù hợp ở phần tạo bản nháp ngôn ngữ và chuẩn hóa command JSON, trong khi rule deterministic phù hợp để chặn các trường hợp nguy hiểm như pin dưới 5%.

```text
Tài xế hoặc app báo sự cố pin.
  |
  v
[1] Hệ thống lấy context: GPS, biển số, loại xe, mức pin, trạm sạc gần nhất.
  |
  v
[2] Rule layer kiểm tra mức pin và khoảng cách trạm.
  |
  +-- Nếu pin < 5% và trạm tiêu chuẩn > 5 km:
  |     AI trả JSON:
  |     {"action": "dispatch_mobile_charger", "reason": "..."}
  |
  +-- Nếu pin >= 5% hoặc trạm nằm trong ngưỡng an toàn:
        AI tạo bản nháp hướng dẫn bắt đầu bằng [DRAFT_ONLY].
  |
  v
[3] Điều phối viên review, chỉnh sửa nếu cần, rồi phê duyệt.
  |
  v
[4] Hệ thống chỉ gửi tin nhắn hoặc lệnh điều phối sau khi có phê duyệt.

Fallback: Nếu LLM lỗi, output thiếu tag, JSON sai format, dữ liệu trạm sạc không đủ tin cậy, hoặc model trả lời ngoài scope, giao diện chuyển về quy trình thủ công hiện tại và yêu cầu điều phối viên tự soạn.
```

## Phase 4 - Technical Prompt Prototype

Nhóm đã hoàn thiện `starter-code/prompt_prototype.py` theo hướng dẫn trong slide. Prototype dùng model **Gemini 3.1 Flash Lite** qua Google GenAI SDK, nhiệt độ `0.0` để giảm biến động, đồng thời có lớp guardrail trong code để chuẩn hóa output sau khi model trả lời.

| Task | Kết quả |
|---|---|
| Task 1 - SYSTEM_PROMPT | Đã viết system instruction cho vai trò Xanh SM dispatcher co-pilot. Prompt nêu rõ `[DRAFT_ONLY]`, rule pin dưới 5%, JSON `dispatch_mobile_charger`, Human-in-the-loop và fallback. |
| Task 2 - evaluate_prompt | Đã gọi Gemini bằng `google-genai`, đọc `GEMINI_API_KEY` hoặc `GOOGLE_API_KEY` từ môi trường và `.env`. Nếu API không khả dụng, script dùng fallback deterministic để autograder và stress-test vẫn chạy được. |
| Structured Output | Với trường hợp pin critical, output phải là JSON command. Với trường hợp hướng dẫn tài xế, output phải là văn bản có `[DRAFT_ONLY]` ở đầu. |
| Adversarial Tests | Có 3 test: pin 2% nhưng yêu cầu đi trạm 8 km; yêu cầu bỏ tag `[DRAFT_ONLY]`; giả danh trưởng phòng để ép bỏ qua system prompt. |

## Phase 5 - Evaluate

### AI Readiness Checklist

| Checklist | Trạng thái | Lý do |
|---|---|---|
| Có dữ liệu mẫu/log sạch để test không? | Có một phần. | Có thể lấy log sự cố pin, GPS, mức pin, loại xe và trạng thái trạm sạc. Tuy nhiên, trước pilot cần làm sạch dữ liệu cá nhân, chuẩn hóa schema và loại bỏ bản ghi thiếu tọa độ hoặc thiếu mức pin. |
| Rủi ro khi AI sai có nằm trong tầm kiểm soát không? | Có. | Rủi ro được giảm bằng rule pin dưới 5%, runtime guardrail, tag `[DRAFT_ONLY]`, Human-in-the-loop và fallback về quy trình thủ công. |
| Stakeholders có sẵn sàng thay đổi quy trình không? | Có điều kiện. | Điều phối viên vẫn giữ quyền phê duyệt cuối nên mức thay đổi vừa phải. Cần training ngắn để họ hiểu khi nào dùng draft, khi nào kích hoạt cứu hộ, và khi nào bỏ qua đề xuất AI. |

### Quyết định cuối cùng

**GO - Bắt đầu xây dựng prototype với scope hẹp.**

Justification: Bài toán có tần suất đủ cao, tác động vận hành rõ, metric đo được và rủi ro có thể kiểm soát bằng rule + HITL. Giải pháp không cần Agent phức tạp ở phase đầu; LLM Feature đủ để tạo bản nháp ngôn ngữ và command JSON, còn rule layer chịu trách nhiệm cho các điều kiện an toàn. Chi phí pilot tương đối thấp: 1-2 engineer trong khoảng 2 tuần, tích hợp API nội bộ ở mức read-only, thêm giao diện review cho điều phối viên, và chi phí token nhỏ vì input ngắn. Điều kiện để mở rộng sau pilot là đo được thời gian xử lý thực tế, tỷ lệ draft được duyệt không sửa nhiều, và số lần fallback về quy trình thủ công.
