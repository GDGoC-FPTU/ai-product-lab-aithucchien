# 01-problem-scan.md — Trần Đình Đăng

## 🏛️ Thông tin cá nhân

| Field | Nội dung |
|-------|----------|
| **Họ và tên** | Trần Đình Đăng |
| **Bài nộp** | Phase 1 (SCAN) & Phase 2 (QUICK-ASSESS) |

---

# 🔍 Phase 1 — SCAN: Bảng quét cơ hội

Sử dụng 4 Lenses để quét bài toán AI cho hệ sinh thái Vingroup.

### 4 Lenses:
1. **Lặp lại (Repetitive):** Tác vụ lặp đi lặp lại nhiều lần hằng ngày
2. **Tốn thời gian (Time-consuming):** Tác vụ ngốn thời gian xử lý thủ công
3. **AI-upgrade:** Dịch vụ khách hàng hiện tại còn chậm hoặc phản hồi rập khuôn
4. **Stakeholder Pain:** Bottleneck khiến khách hàng hoặc nhân viên thực địa phàn nàn

### 📝 Danh sách 5 bài toán:

| # | Công ty thành viên | Lens | Mô tả ngắn bài toán |
|---|-------------------|------|---------------------|
| 1 | VinFast | Repetitive | So khớp hóa đơn mua linh kiện với phiếu nhập kho tại kho TPHCM - mỗi ngày 50-80 phiếu, nhân viên kho phải đối chiếu thủ công từng dòng |
| 2 | Xanh SM | Time-consuming | Soạn thảo phản hồi tự động cho các phản hồi tiêu cực (1-2 sao) từ khách hàng về dịch vụ taxi điện - nhân viên CSKH mất 8-15 phút/phản hồi |
| 3 | Vinhomes | Stakeholder Pain | Tài xế giao hàng phàn nàn hệ thống điều hướng trong khu đô thị không chính xác, dẫn đến giao hàng trễ 20-30 phút/lần |
| 4 | Vinmec | AI-upgrade | Chatbot tư vấn khám bệnh hiện tại trả lời theo kịch bản cứng, không xử lý được câu hỏi phức tạp về triệu chứng bệnh |
| 5 | Vinpearl | Repetitive | Phân loại và phân phối đánh giá khách hàng từ nhiều nền tảng (TripAdvisor, Google, Facebook) về đúng bộ phận xử lý - 200+ review/ngày |

---

# 🃏 Phase 2 — QUICK-ASSESS: 3 Quick Problem Cards

---

```
┌─────────────────────────────────────────────────────────────────────┐
│ QUICK PROBLEM CARD #1                                               │
│                                                                     │
│ Bài toán (1 câu): Soạn thảo phản hồi tự động cho đánh giá tiêu cực │
│                   của khách hàng Xanh SM                            │
│                                                                     │
│ Công ty thành viên: [ ] VinFast  [x] Xanh SM  [ ] Vinhomes         │
│                     [ ] Vinmec   [ ] Khác (Ghi rõ)                 │
│                                                                     │
│ Ai đang đau (Actor)? Nhân viên chăm sóc khách hàng (CSKH)         │
│                                                                     │
│ Workflow thủ công hiện tại (3-5 bước):                             │
│   1. Nhận thông báo đánh giá 1-2 sao từ hệ thống                   │
│   --> 2. Đọc nội dung phản hồi của khách hàng                       │
│   --> 3. Phân loại loại khiếu nại (delay, thái độ tài xế, xe脏...) │
│   --> 4. Soạn thảo phản hồi cá nhân hóa theo template              │
│   --> 5. Gửi phản hồi và theo dõi phản hồi tiếp theo                │
│                                                                     │
│ Bước nào tốn thời gian/lỗi nhất? Bước 4 - Soạn thảo phản hồi        │
│ (⏱ 10-15 phút/lượt)                                                │
│                                                                     │
│ AI có thể nhảy vào hỗ trợ ở bước nào? Bước 3 (phân loại) và        │
│ Bước 4 (soạn thảo phản hồi cá nhân hóa)                            │
│                                                                     │
│ Đo thành công bằng gì (Metric có số)?                                │
│   Giảm thời gian xử lý mỗi phản hồi từ 12 phút xuống dưới 2 phút   │
│   Tăng tỷ lệ phản hồi trong 24h từ 60% lên 95%                     │
│                                                                     │
│ Quick Architecture: [ ] No AI  [ ] Rule  [x] LLM  [ ] Agent         │
└─────────────────────────────────────────────────────────────────────┘
```

---

```
┌─────────────────────────────────────────────────────────────────────┐
│ QUICK PROBLEM CARD #2                                               │
│                                                                     │
│ Bài toán (1 câu): Tối ưu hóa quy trình so khớp hóa đơn linh kiện  │
│                   tại kho VinFast                                   │
│                                                                     │
│ Công ty thành viên: [x] VinFast  [ ] Xanh SM  [ ] Vinhomes          │
│                     [ ] Vinmec   [ ] Khác (Ghi rõ)                 │
│                                                                     │
│ Ai đang đau (Actor)? Nhân viên kho tại trung tâm phân phối TPHCM    │
│                                                                     │
│ Workflow thủ công hiện tại (3-5 bước):                             │
│   1. Nhận email hóa đơn từ nhà cung cấp (PDF/Excel)                │
│   --> 2. Mở file và đối chiếu từng dòng với phiếu nhập kho          │
│   --> 3. Kiểm tra mã SKU, số lượng, đơn giá                       │
│   --> 4. Đánh dấu OK hoặc báo lỗi nếu không khớp                   │
│   --> 5. Cập nhật vào hệ thống ERP                                 │
│                                                                     │
│ Bước nào tốn thời gian/lỗi nhất? Bước 2+3 - Đối chiếu thủ công     │
│ (⏱ 5-8 phút/phiếu, 50-80 phiếu/ngày = 4-10 giờ/ngày)               │
│                                                                     │
│ AI có thể nhảy vào hỗ trợ ở bước nào? Bước 2+3+4 - Nhận diện OCR,  │
│ so khớp tự động và đánh dấu bất thường                             │
│                                                                     │
│ Đo thành công bằng gì (Metric có số)?                                │
│   Giảm thời gian xử lý/phiếu từ 6 phút xuống dưới 30 giây          │
│   Giảm sai sót đối chiếu từ 3% xuống dưới 0.5%                     │
│                                                                     │
│ Quick Architecture: [ ] No AI  [x] Rule  [ ] LLM  [ ] Agent         │
└─────────────────────────────────────────────────────────────────────┘
```

---

```
┌─────────────────────────────────────────────────────────────────────┐
│ QUICK PROBLEM CARD #3                                               │
│                                                                     │
│ Bài toán (1 câu): Chatbot tư vấn khám bệnh tại Vinmec không xử lý  │
│                   được câu hỏi phức tạp về triệu chứng              │
│                                                                     │
│ Công ty thành viên: [ ] VinFast  [ ] Xanh SM  [ ] Vinhomes          │
│                     [x] Vinmec   [ ] Khác (Ghi rõ)                 │
│                                                                     │
│ Ai đang đau (Actor)? Bệnh nhân đang tìm kiếm thông tin và nhân viên│
│                     lễ tân phải trả lời điện thoại hỏi bệnh        │
│                                                                     │
│ Workflow thủ công hiện tại (3-5 bước):                             │
│   1. Bệnh nhân hỏi chatbot về triệu chứng bệnh                     │
│   --> 2. Chatbot trả lời theo kịch bản cố định                     │
│   --> 3. Nếu không hiểu, chatbot chuyển sang "liên hệ tổng đài"   │
│   --> 4. Bệnh nhân gọi tổng đài, chờ 5-10 phút                     │
│   --> 5. Nhân viên tổng đài tư vấn (quá tải, cao điểm 20-30 cuộc/h)|
│                                                                     │
│ Bước nào tốn thời gian/lỗi nhất? Bước 2+3+4+5 - Chuyển tổng đài   │
│ (⏱ 5-15 phút/bệnh nhân, tỷ lệ bỏ qua 35% do chờ đợi)              │
│                                                                     │
│ AI có thể nhảy vào hỗ trợ ở bước nào? Bước 2 - LLM phân tích ý    │
│ định, trả lời tự nhiên, chỉ chuyển khi cần thiết                   │
│                                                                     │
│ Đo thành công bằng gì (Metric có số)?                                │
│   Tăng tỷ lệ giải quyết tại chatbot từ 40% lên 80%                 │
│   Giảm cuộc gọi tổng đài đi 50% (từ 30 xuống 15 cuộc/giờ cao điểm) │
│   Tăng satisfaction score từ 3.2 lên 4.0/5.0                       │
│                                                                     │
│ Quick Architecture: [ ] No AI  [ ] Rule  [x] LLM  [ ] Agent         │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 📌 Tổng kết

**Top 3 bài toán được chọn cho Phase 3 (Deep-Dive):**
1. **Xanh SM** - Soạn thảo phản hồi đánh giá tiêu cực (LLM) ⭐ Ưu tiên cao nhất
2. **VinFast** - So khớp hóa đơn linh kiện kho (Rule + OCR) 
3. **Vinmec** - Chatbot tư vấn khám bệnh (LLM)

*Lý do chọn: Bài toán Xanh SM có impact rõ ràng, data sẵn có, và LLM phù hợp với tác vụ generate text cá nhân hóa.*
