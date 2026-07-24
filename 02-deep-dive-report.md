# Deep-Dive Report — Xanh SM

## Chọn bài toán: AI hỗ trợ xử lý sự cố hết pin thực địa cho tài xế Xanh SM

---

## 3.1. Current-State Workflow Mapping

Quy trình hiện tại xử lý sự cố hết pin của tài xế khi đang trên đường vận hành:

```text
[1] Tài xế gọi tổng đài điều vận
        │
        ▼
[2] Điều phối viên tra cứu vị trí xe trên hệ thống định vị
        │
        ▼
[3] Điều phối viên tra cứu trạm sạc VinFast gần nhất còn trụ trống
        │
        ▼
[4] Điều phối viên soạn tin hướng dẫn và chỉ đường cho tài xế
        │
        ▼
[5] Nếu pin quá thấp, điều phối viên gọi xe cứu hộ pin di động
```

### Điểm bottleneck và handoff
- 🔴 Bottleneck chính: Bước 3 và bước 4. Đây là phần mất nhiều thời gian nhất vì nhân viên phải tra cứu thủ công nhiều hệ thống khác nhau và viết lại nội dung hướng dẫn từng lần.
- 🔄 Handoff: Thông tin được chuyển từ tài xế sang điều phối viên, rồi từ điều phối viên sang hệ thống bản đồ/trạm sạc, rồi lại sang tài xế bằng tin nhắn hoặc gọi điện.
- ⏱ Thời gian vận hành trung bình: Tổng cộng khoảng 15 phút/lượt.

---

## 3.2. Problem Statement (6-field) & Metrics

| Field | Nội dung chi tiết |
|---|---|
| **1. Actor / Operator** | Điều phối viên (Dispatcher) tại Trung tâm Điều vận Xanh SM. |
| **2. Current Workflow** | Khi tài xế báo tình trạng hết pin hoặc pin thấp giữa đường, điều phối viên phải tra cứu vị trí xe trên hệ thống định vị, kiểm tra trạm sạc gần nhất còn trụ trống, soạn tin nhắn chỉ dẫn và gửi cho tài xế, rồi quyết định có gọi xe cứu hộ hay không. Toàn bộ quy trình hiện nay chủ yếu là thủ công. |
| **3. Bottleneck** | Bước tra cứu trạm sạc phù hợp và bước soạn tin nhắn chỉ dẫn là chậm nhất. Đây là nơi có nhiều thao tác lặp lại và dễ sai sót. |
| **4. Business Impact** | Mỗi ngày có khoảng 80–100 sự cố về pin thấp/không đủ pin cho xe trên mạng lưới Hà Nội. Trung bình mỗi sự cố làm chậm xử lý 10–15 phút, gây lãng phí 20 giờ làm việc/ngày cho đội điều vận. Nếu xử lý chậm, tài xế chờ lâu, tỉ lệ khách hủy chuyến tăng và doanh thu bị rò rỉ khoảng 10–15% trong các khung giờ cao điểm. |
| **5. Success Metric** | Giảm thời gian xử lý sự cố từ 15 phút xuống còn dưới 3 phút/lượt; đạt tỷ lệ đề xuất trạm sạc đúng và phù hợp đạt trên 98%; giảm số lần điều phối viên phải viết lại tin nhắn thủ công xuống dưới 20%. |
| **6. Operational Boundary** | AI được phép truy xuất dữ liệu định vị xe, tra cứu trạm sạc gần nhất và soạn một bản nháp tin nhắn hướng dẫn. Tuyệt đối không được tự động gửi tin cho tài xế mà không có sự phê duyệt của điều phối viên. AI cũng không được đề xuất trạm sạc quá xa khi mức pin đang ở ngưỡng nguy hiểm. |

---

## 3.3. Future-State Flow & AI Fit

### AI Fit
- Chọn: LLM Feature
- Lý do: Quy trình có cấu trúc rõ ràng, nhưng phần soạn tin nhắn và diễn giải ngôn ngữ tự nhiên lại rất phù hợp với LLM. Đây không phải bài toán cần agent tự trị hoàn toàn vì rủi ro khi đề xuất sai có thể khiến xe cạn pin giữa đường.

### Future-State Flow

```text
[1] Tài xế báo sự cố
        │
        ▼
[2] 🔵 AI tự động truy xuất vị trí xe và danh sách trạm sạc gần nhất
        │
        ▼
[3] 🔵 AI soạn bản nháp tin nhắn chỉ dẫn và gợi ý trạm phù hợp
        │
        ▼
[4] 🟢 Điều phối viên xem lại và phê duyệt
        │
        ▼
[5] Gửi cho tài xế hoặc gọi xe cứu hộ nếu cần
```

### Fallback
- ↩️ Nếu AI không tự tin hoặc dữ liệu đầu vào thiếu, hệ thống sẽ chuyển về quy trình cũ: điều phối viên tự tra cứu và soạn tin thủ công.
- ↩️ Nếu mức pin dưới 5%, hệ thống phải ưu tiên đề xuất xe cứu hộ pin di động thay vì cố gắng đưa xe đến trạm sạc xa.

---

## Kết luận ngắn

Bài toán này có thể được coi là một ứng dụng AI phù hợp vì vừa có tính lặp lại, vừa có áp lực thời gian cao, lại có thể kiểm soát rủi ro bằng quy trình phê duyệt người dùng. Đây là một use case tốt để bắt đầu với scope hẹp và ưu tiên đo lường hiệu quả bằng thời gian xử lý và độ chính xác đề xuất.

# Phase 5 — EVALUATE
AI Readiness Checklist

✅ 1. Chúng tôi có sẵn dữ liệu mẫu/logs sạch để test?

☑ Có

Có lịch sử ticket từ hệ thống CRM.
Có dữ liệu về nội dung ticket, nhãn phân loại và kết quả xử lý.
Có thể dùng dữ liệu này để đánh giá độ chính xác của mô hình.

✅ 2. Rủi ro khi AI sai có nằm trong tầm kiểm soát?

☑ Có

AI chỉ phân loại và gợi ý phản hồi.
Nhân viên vẫn kiểm tra trước khi gửi khách hàng (Human-in-the-loop).
Nếu AI không chắc chắn (confidence thấp) thì chuyển sang xử lý thủ công (Fallback).

✅ 3. Stakeholders sẵn sàng thay đổi quy trình làm việc cũ?

☑ Có

Nhân viên CSKH vẫn giữ vai trò phê duyệt.
AI chỉ hỗ trợ giảm thời gian xử lý, không thay thế hoàn toàn con người.
Quy trình mới ít làm thay đổi cách phối hợp giữa các bộ phận.
Quyết định

☑ GO (Bắt đầu xây dựng Prototype)

Justification

Quy trình hiện tại mất nhiều thời gian ở bước đọc và phân loại ticket thủ công, đồng thời có tỷ lệ phân loại nhầm khi khối lượng công việc tăng cao. Doanh nghiệp đã có dữ liệu lịch sử đủ để huấn luyện và đánh giá mô hình. Rủi ro khi AI dự đoán sai được kiểm soát thông qua cơ chế Human-in-the-loop và Fallback, vì AI chỉ đóng vai trò hỗ trợ, còn quyết định cuối cùng vẫn do nhân viên thực hiện. Prototype có thể triển khai với phạm vi hẹp (chỉ tự động phân loại và gợi ý phản hồi) để đo lường các chỉ số như độ chính xác, thời gian xử lý và tỷ lệ chấp nhận của nhân viên trước khi mở rộng.

Khi nào chọn NOT YET?

Ví dụ:

Chưa có dữ liệu lịch sử hoặc dữ liệu chưa được gán nhãn.
Chưa xác định được KPI hiện tại để so sánh.
Chưa có người dùng sẵn sàng thử nghiệm.

Justification mẫu:

Hiện tại dữ liệu lịch sử chưa đủ sạch và chưa có nhãn phân loại đáng tin cậy để huấn luyện hoặc đánh giá mô hình. Trước khi xây dựng prototype, cần thu thập thêm dữ liệu, chuẩn hóa quy trình gán nhãn và xác lập các chỉ số baseline nhằm đảm bảo việc đánh giá hiệu quả của AI được chính xác.

Khi nào chọn NO-GO?

Ví dụ:

Quy trình chỉ có vài quy tắc cố định.
Rule-based đã xử lý gần như hoàn hảo.
Chi phí triển khai AI lớn hơn lợi ích mang lại.

Justification mẫu:

Bài toán chủ yếu dựa trên các quy tắc cố định và ít thay đổi theo thời gian. Hệ thống rule-based hiện tại đã đáp ứng tốt yêu cầu với chi phí thấp, trong khi việc áp dụng LLM sẽ làm tăng chi phí vận hành và độ phức tạp mà không mang lại cải thiện đáng kể về hiệu quả. Do đó, chưa có cơ sở để đầu tư phát triển giải pháp AI.
