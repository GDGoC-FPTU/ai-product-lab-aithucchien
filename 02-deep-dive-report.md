# 🏗️ Lab 02 — Deep-Dive Report: AI Product Scoping (Vin Smart Future)

> ⚠️ **Cần điền trước khi nộp bài:** Tên nhóm và danh sách thành viên (Họ tên + MSSV) bên dưới.

**Tên nhóm:** _______________

**Thành viên:**
| # | Họ và tên | MSSV |
|---|-----------|------|
| 1 | Trần Đình Đăng | 2A202601998 |
| 2 | Nguyễn Trung Hiếu | 2A202601457 |
| 3 | Nguyễn Thế Anh | 2A202601791 |
| 4 | Nguyễn Thị Lý | 2A202601963 |
| 5 | Vũ Văn Phong | 2A202601647 |

---

## 🗳️ Quyết định lựa chọn của nhóm

Nhóm quyết định chọn bài toán **"Phân loại & Điều phối Yêu cầu Bảo hành (Warranty Claims) — VinFast"** (Card #1 trong `01-problem-scan.md`) để thực hiện Deep-Dive.

### Lý do lựa chọn và loại bỏ các thẻ khác:
* **Card #1 (Warranty Claims) — Được chọn:** Tổn thất vận hành rõ ràng và lớn nhất (~160 giờ nhân sự lãng phí/ngày, ~12% route sai trung tâm dịch vụ). Dữ liệu đầu vào (mô tả lỗi + ảnh/video khách gửi) đã có sẵn trong hệ thống CRM. Rủi ro AI sai dễ kiểm soát bằng Human-in-the-loop vì nhân viên CSKH vẫn duyệt trước khi tạo phiếu hẹn chính thức.
* **Card #2 (Predictive Battery Maintenance):** Bị loại vì cần thêm thời gian thu thập và làm sạch dữ liệu log pin ở quy mô lớn (>20.000 xe) trước khi có thể huấn luyện/đánh giá độ tin cậy của mô hình phát hiện bất thường — phù hợp hướng "NOT YET" hơn là triển khai ngay.
* **Card #3 (QC ngoại quan):** Bị loại vì bài toán đòi hỏi xử lý ảnh/video thời gian thực (computer vision) phức tạp hơn nhiều so với xử lý văn bản, và hiện chưa có tập dữ liệu ảnh lỗi ngoại quan được gán nhãn sẵn.

---

## 3.2. Problem Statement (6-field) & Metrics

| Field | Nội dung chi tiết |
|---|---|
| **1. Actor / Operator** | Nhân viên CSKH / điều phối viên bảo hành tại các trung tâm CSKH VinFast trên toàn quốc. |
| **2. Current Workflow** | Khách hàng gửi yêu cầu bảo hành qua app VinFast kèm mô tả lỗi bằng văn bản + ảnh/video. Nhân viên CSKH đọc thủ công, tự đánh giá mức độ nghiêm trọng, tra cứu Service Center (trung tâm dịch vụ) phù hợp gần nhất với vị trí khách và loại lỗi, sau đó route yêu cầu và tạo phiếu hẹn. Toàn bộ 4 bước đều thao tác thủ công, sử dụng app nội bộ + bảng tra cứu trung tâm dịch vụ, mất trung bình 8-10 phút/case. |
| **3. Bottleneck** | Bước 2-3: đánh giá mức độ nghiêm trọng từ mô tả lỗi + ảnh/video, và tra cứu trung tâm dịch vụ phù hợp (đúng chuyên môn sửa chữa, còn chỗ tiếp nhận). Đây là bước tốn thời gian nhất vì nhân viên phải đọc kỹ nội dung tự do (unstructured text) và tra cứu chéo nhiều hệ thống khác nhau. |
| **4. Business Impact** | Trung bình ~1.200 case bảo hành/ngày toàn quốc → tương đương **~160 giờ nhân sự lãng phí/ngày**. Tỉ lệ route sai trung tâm dịch vụ hiện ở mức **~12%**, khiến khách hàng phải chờ thêm 1-2 ngày để được chuyển đúng nơi, ảnh hưởng trực tiếp đến trải nghiệm khách hàng và uy tín thương hiệu VinFast. |
| **5. Success Metric** | 1. Giảm thời gian xử lý phân loại + route case từ 8-10 phút xuống **dưới 2 phút/case** (Efficiency).<br>2. Giảm tỉ lệ route sai trung tâm dịch vụ từ 12% xuống **dưới 3%** (Quality). |
| **6. Operational Boundary** | AI được phép đọc mô tả lỗi + ảnh/video khách gửi, phân loại mức độ nghiêm trọng, và **đề xuất (draft)** trung tâm dịch vụ phù hợp. **CẤM:** AI không được tự động xác nhận lịch hẹn hoặc từ chối yêu cầu bảo hành mà không có nhân viên CSKH phê duyệt (bắt buộc HITL); không được tự đưa ra chẩn đoán kỹ thuật thay thế kỹ thuật viên; nếu mô tả liên quan đến rủi ro an toàn (cháy, nổ, mất phanh, mất lái) phải **escalate khẩn cấp ngay lập tức** cho người có thẩm quyền, tuyệt đối không xử lý như case thông thường. |

---

## 3.3. Future-State Flow & AI Fit

**AI Fit:** [x] **LLM Feature** — không chọn Agentic Loop vì quy trình có cấu trúc cố định (4 bước tuyến tính), phạm vi hành động hẹp (phân loại + đề xuất), và rủi ro khi AI sai (route nhầm trung tâm) có thể kiểm soát tốt bằng một bước duyệt của con người thay vì để AI tự trị đưa ra quyết định cuối cùng.

**Future-State Flow:**
```text
┌──────────────┐     ┌──────────────────┐     ┌──────────────────┐     ┌──────────────┐
│ Bước 1        │     │ Bước 2            │     │ Bước 3            │     │ Bước 4        │
│ Khách gửi yêu │     │ 🔵 AI đọc mô tả +│     │ 🟢 Nhân viên CSKH │     │ Hệ thống tạo  │
│ cầu bảo hành  │ ──→ │ ảnh/video, phân   │ ──→ │ review & duyệt    │ ──→ │ phiếu hẹn     │
│ qua app       │     │ loại mức độ +     │     │ đề xuất (HITL)    │     │ chính thức,   │
│               │     │ đề xuất Service   │     │                   │     │ gửi khách     │
│               │     │ Center (draft)    │     │                   │     │               │
└──────────────┘     └──────────────────┘     └──────────────────┘     └──────────────┘
                                                        │
                                                        ▼
                                                 ↩️ Fallback:
                                                 Nếu AI báo độ tin cậy thấp, hoặc mô tả có
                                                 dấu hiệu rủi ro an toàn (cháy/nổ/mất phanh),
                                                 case tự động escalate cho nhân viên xử lý
                                                 thủ công hoàn toàn như quy trình cũ.
```

---

## 🏁 Phase 5 — EVALUATE

### AI Readiness Checklist:
1. [x] Chúng tôi có sẵn dữ liệu mẫu/logs sạch để test? — **Có.** Hệ thống CRM đã lưu trữ nhiều năm dữ liệu case bảo hành lịch sử (mô tả lỗi, ảnh/video đính kèm, kết quả route thực tế) có thể dùng để test và đánh giá độ chính xác phân loại.
2. [x] Rủi ro khi AI sai có nằm trong tầm kiểm soát (qua HITL hoặc Fallback)? — **Có.** Mọi đề xuất của AI đều ở dạng draft, bắt buộc nhân viên CSKH duyệt trước khi tạo phiếu hẹn chính thức; case liên quan an toàn tự động escalate.
3. [x] Stakeholders sẵn sàng thay đổi quy trình làm việc cũ? — **Có, với điều kiện.** Đội ngũ CSKH đồng thuận thử nghiệm vì giảm tải công việc lặp lại, nhưng cần có giai đoạn đào tạo ngắn (~1 tuần) để nhân viên làm quen với việc review đề xuất AI thay vì tự tra cứu từ đầu.

### Quyết định cuối cùng của Ban Giám Đốc Vin Smart Future:
[x] **GO (Bắt đầu xây dựng Prototype):** Bắt đầu phát triển với scope hẹp.
[ ] **NOT YET (Cần tích lũy thêm dữ liệu/xác lập baseline):** Trì hoãn để chuẩn bị thêm.
[ ] **NO-GO (Không khả thi / Rule-based tốt hơn):** Hủy bỏ dự án AI này.

**Justification (Lý giải quyết định dựa trên bằng chứng kỹ thuật và chi phí):**
> Bài toán đạt mức **GO** vì hội đủ 3 điều kiện quan trọng: (1) **Dữ liệu sẵn sàng** — hàng nghìn case bảo hành lịch sử có sẵn để test và fine-tune prompt mà không cần thu thập thêm; (2) **Kiến trúc đơn giản, chi phí thấp** — chỉ cần một LLM Feature xử lý văn bản + ảnh để phân loại và đề xuất, không cần Agentic Loop phức tạp, chi phí vận hành theo ước tính ~1.200 case/ngày ở mức giá Gemini 2.5 Flash là không đáng kể so với 160 giờ nhân sự tiết kiệm được mỗi ngày; (3) **Rủi ro được kiểm soát chặt chẽ** — Operational Boundary quy định rõ AI chỉ đưa ra đề xuất (draft), con người luôn là người ra quyết định cuối cùng (HITL), và có Fallback tự động cho các case an toàn khẩn cấp. Điểm cần lưu ý khi triển khai: cần đo lường thêm độ chính xác phân loại trên tập dữ liệu thực tế trước khi mở rộng ra toàn bộ 63 tỉnh thành, bắt đầu thử nghiệm ở quy mô nhỏ (pilot 1-2 khu vực) trong 4-6 tuần.
