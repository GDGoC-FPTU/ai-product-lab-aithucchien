# 🔍 Phase 1 — SCAN (Cá nhân, 20 min)

Hãy sử dụng **4 Lenses** dưới đây để quét qua hoạt động vận hành của các công ty thành viên Vingroup. Ghi lại **ít nhất 5 bài toán/bottleneck** thực tế.

### 4 Lenses tìm bài toán AI cho Vingroup:
1. **Lặp lại (Repetitive):** Tác vụ lặp đi lặp lại nhiều lần hằng ngày. (Ví dụ: So khớp hóa đơn sạc điện tại VinFast, route lại chuyến taxi tại Xanh SM).
2. **Tốn thời gian (Time-consuming):** Tác vụ ngốn thời gian xử lý thủ công của nhân viên. (Ví dụ: Soạn thảo phản hồi đánh giá 1-star của cư dân Vinhomes).
3. **AI có thể tốt hơn (AI-upgrade):** Dịch vụ khách hàng hiện tại còn chậm hoặc phản hồi rập khuôn. (Ví dụ: Chatbot CSKH Vinpearl hỗ trợ đặt vé vui chơi).
4. **Pain từ người khác (Stakeholder Pain):** Bottleneck khiến khách hàng hoặc nhân viên thực địa phàn nàn. (Ví dụ: Tài xế Xanh SM phàn nàn về việc hệ thống gợi ý điểm đón khách không chính xác).

> [!TIP]
> **🤖 AI Prompts — Partner brainstorm:**
> Hãy sử dụng prompt sau để brainstorm các bài toán thực tế nếu bạn chưa có ý tưởng:
> *"Tôi là AI Engineer tại Vin Smart Future (Vingroup). Tôi đang tìm kiếm các pain point vận hành cụ thể có thể tối ưu bằng AI cho mảng [Chọn một: VinFast / Xanh SM / Vinhomes / Vinmec]. Hãy gợi ý cho tôi 5 quy trình nghiệp vụ thủ công, tốn nhiều thời gian và gây rò rỉ hiệu suất kèm con số thống kê ước tính về tổn thất."*

### 📝 List bài toán của tôi:
| # | Subsidiary (VinFast/Xanh SM...) | Lens | Mô tả ngắn bài toán |
|---|----------------------------------|------|---------------------|
| 1 | VinFast | Tốn thời gian | Xử lý yêu cầu bảo hành (warranty claims): nhân viên CSKH đọc mô tả lỗi + ảnh/video khách gửi, tự phân loại mức độ và route đến trung tâm dịch vụ. ~8-10 phút/case, ~1.200 case/ngày → ~160 giờ nhân sự lãng phí/ngày, ~12% route sai trung tâm. |
| 2 | VinFast | Lặp lại | Kiểm soát chất lượng cuối dây chuyền (QC ngoại quan): nhân viên ghi lỗi ngoại quan thủ công vào giấy/Excel, cuối ca mới tổng hợp. Trễ phát hiện lỗi 4-6 giờ/lô, ~3% xe phải tái kiểm (rework) → thiệt hại ~2 tỷ VNĐ/tháng. |
| 3 | VinFast | Lặp lại | Đối chiếu đơn hàng linh kiện với nhà cung cấp: đối chiếu thủ công số lượng/đơn giá với PO và tồn kho ERP. ~25 phút/đơn, ~200 đơn/ngày → ~80 giờ nhân sự/ngày, gián đoạn dây chuyền ~1 lần/tuần do phát hiện sai lệch muộn. |
| 4 | VinFast | AI có thể tốt hơn | Tư vấn kỹ thuật lặp lại tại showroom & hotline: tư vấn viên trả lời hàng loạt câu hỏi giống nhau về specs, sạc, bảo hành pin. ~65% câu hỏi là lặp lại, chiếm ~3 giờ/ngày/tư vấn viên → ước tính rò rỉ ~8% cơ hội chốt sale. |
| 5 | VinFast | Pain từ người khác | Phân tích log dữ liệu pin để dự đoán bảo trì: kỹ sư R&D tải và đọc thủ công log pin (nhiệt độ, chu kỳ sạc, độ chai) để phát hiện suy giảm bất thường. Mỗi kỹ sư xử lý ~50 xe/ngày trong khi cần theo dõi >20.000 xe → phát hiện lỗi trễ 2-3 tuần. |

---

# 🃏 Phase 2 — QUICK-ASSESS (Cá nhân, 30 min)

Chọn **top 3 bài toán** từ danh sách trên và hoàn thiện **3 Quick Problem Cards** dưới đây (10 phút/card).

```
┌─────────────────────────────────────────────────────────────┐
│ QUICK PROBLEM CARD #1                                        │
│                                                               │
│ Bài toán: Phân loại & điều phối yêu cầu bảo hành (warranty   │
│ claims) từ mô tả lỗi + ảnh/video khách gửi qua app.          │
│ Công ty thành viên: [x] VinFast  [ ] Xanh SM  [ ] Vinhomes   │
│                     [ ] Vinmec   [ ] Khác_____________       │
│                                                               │
│ Ai đang đau (Actor)? Nhân viên CSKH / điều phối bảo hành     │
│                                                               │
│ Workflow thủ công hiện tại (4 bước):                         │
│   1. Khách gửi yêu cầu qua app (mô tả + ảnh/video)           │
│   ──> 2. Nhân viên đọc, đánh giá mức độ nghiêm trọng         │
│   ──> 3. Tra cứu Service Center phù hợp gần nhất             │
│   ──> 4. Route yêu cầu + tạo phiếu hẹn                       │
│                                                               │
│ Bước nào tốn thời gian/lỗi nhất? Bước 2-3 (⏱ 8-10 phút/lượt)│
│ AI có thể nhảy vào hỗ trợ ở bước nào? Bước 2-3 (phân loại    │
│ mức độ nghiêm trọng từ text/ảnh, đề xuất Service Center)     │
│                                                               │
│ Đo thành công bằng gì (Metric có số)?                        │
│   Giảm thời gian xử lý từ 8-10 phút ──> dưới 2 phút/case;    │
│   giảm tỉ lệ route sai từ 12% ──> dưới 3%.                   │
│                                                               │
│ Quick Architecture: [ ] No AI  [ ] Rule  [x] LLM  [ ] Agent  │
└───────────────────────────────────────────────────────────────┘
```

```
┌─────────────────────────────────────────────────────────────┐
│ QUICK PROBLEM CARD #2                                        │
│                                                               │
│ Bài toán: Phát hiện sớm dấu hiệu suy giảm pin bất thường     │
│ từ log dữ liệu xe điện (Predictive Battery Maintenance).     │
│ Công ty thành viên: [x] VinFast  [ ] Xanh SM  [ ] Vinhomes   │
│                     [ ] Vinmec   [ ] Khác_____________       │
│                                                               │
│ Ai đang đau (Actor)? Kỹ sư R&D / kỹ sư bảo trì pin           │
│                                                               │
│ Workflow thủ công hiện tại (4 bước):                         │
│   1. Xe gửi log pin (nhiệt độ, chu kỳ sạc, điện áp) về hệ    │
│   thống ──> 2. Kỹ sư tải log thủ công theo từng xe           │
│   ──> 3. Phân tích thủ công tìm pattern bất thường           │
│   ──> 4. Lập báo cáo cảnh báo nếu phát hiện lỗi              │
│                                                               │
│ Bước nào tốn thời gian/lỗi nhất? Bước 2-3 (⏱ mỗi kỹ sư chỉ   │
│ xử lý ~50 xe/ngày trong khi cần theo dõi >20.000 xe)         │
│ AI có thể nhảy vào hỗ trợ ở bước nào? Bước 2-3 (tự động phân │
│ tích log, phát hiện pattern bất thường, xếp hạng ưu tiên)    │
│                                                               │
│ Đo thành công bằng gì (Metric có số)?                        │
│   Giảm thời gian phát hiện suy giảm pin từ 2-3 tuần ──>      │
│   dưới 3 ngày; độ chính xác cảnh báo đạt ≥90%.               │
│                                                               │
│ Quick Architecture: [ ] No AI  [ ] Rule  [x] LLM  [ ] Agent  │
└───────────────────────────────────────────────────────────────┘
```

```
┌─────────────────────────────────────────────────────────────┐
│ QUICK PROBLEM CARD #3                                        │
│                                                               │
│ Bài toán: Ghi nhận & tổng hợp lỗi ngoại quan xe tại cuối     │
│ dây chuyền sản xuất (QC ngoại quan).                         │
│ Công ty thành viên: [x] VinFast  [ ] Xanh SM  [ ] Vinhomes   │
│                     [ ] Vinmec   [ ] Khác_____________       │
│                                                               │
│ Ai đang đau (Actor)? Nhân viên QC / trưởng dây chuyền        │
│                                                               │
│ Workflow thủ công hiện tại (5 bước):                         │
│   1. QC kiểm tra ngoại quan xe cuối dây chuyền               │
│   ──> 2. Ghi lỗi thủ công vào giấy/Excel                     │
│   ──> 3. Cuối ca tổng hợp dữ liệu ──> 4. Báo cáo lên trưởng  │
│   dây chuyền ──> 5. Quyết định rework                        │
│                                                               │
│ Bước nào tốn thời gian/lỗi nhất? Bước 2-3 (⏱ trễ phát hiện   │
│ 4-6 giờ/lô)                                                  │
│ AI có thể nhảy vào hỗ trợ ở bước nào? Bước 2-3 (ghi nhận lỗi │
│ tức thời qua ảnh chụp + phân loại tự động, tổng hợp real-time)│
│                                                               │
│ Đo thành công bằng gì (Metric có số)?                        │
│   Giảm thời gian phát hiện lỗi từ 4-6 giờ ──> dưới 30 phút;  │
│   giảm tỉ lệ rework từ 3% ──> dưới 1%.                       │
│                                                               │
│ Quick Architecture: [ ] No AI  [ ] Rule  [x] LLM  [ ] Agent  │
└───────────────────────────────────────────────────────────────┘
```

> [!TIP]
> **🤖 AI Prompts — Stress-Test thẻ bài toán:**
> Hãy dán nội dung thẻ bài toán của bạn vào LLM để nhận phản biện:
> *"Đây là một thẻ bài toán vận hành tôi đề xuất cho Vin Smart Future: [Dán nội dung]. Hãy đóng vai trò là một CFO và Trưởng phòng Vận hành cực kỳ khắt khe, chỉ ra cho tôi 3 điểm yếu về logic, metric, và giải thích vì sao rule-based code thông thường có thể giải quyết bài toán này tốt hơn là dùng AI."*

---