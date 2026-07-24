# 02-deep-dive-report.md — Báo cáo Deep-Dive Nhóm

## 🏛️ Thông tin nhóm

| Field | Nội dung |
|-------|----------|
| **Thành viên** | Trần Đình Đăng (và các thành viên nhóm) |
| **Bài toán chọn** | Xanh SM - Hệ thống soạn thảo phản hồi đánh giá tiêu cực tự động |
| **Ngày nộp** | Phase 3 (DEEP-DIVE) & Phase 5 (EVALUATE) |

---

# 🏗️ Phase 3 — DEEP-DIVE

## 3.1. Quyết định lựa chọn bài toán

### Bài toán được chọn: **Xanh SM — Intelligent Negative Review Response System**

**Lý do chọn bài toán này:**

| Tiêu chí | Đánh giá |
|----------|----------|
| **Tính khả thi kỹ thuật** | ✅ Cao - LLM text generation đã mature, dễ implement |
| **Sẵn sàng dữ liệu** | ✅ Cao - Có database đánh giá từ app + web, có history phản hồi cũ |
| **Business impact** | ✅ Rõ ràng - Ảnh hưởng trực tiếp đến brand reputation và NPS |
| **Tần suất xảy ra** | ✅ Cao - 200-400 reviews tiêu cực/tháng |
| **Scope phù hợp** | ✅ Vừa đủ - Có thể prototype trong 1 sprint |

---

## 3.2. Current-State Workflow Mapping

### Quy trình hiện tại (Manual Workflow):

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    CURRENT STATE: MANUAL WORKFLOW                       │
│                    Thời gian trung bình: 12 phút/lượt                  │
└─────────────────────────────────────────────────────────────────────────┘

  [1]                    [2]                    [3]                    [4]
    │                      │                      │                      │
    ▼                      ▼                      ▼                      ▼
┌────────┐    ──>    ┌────────────┐    ──>    ┌──────────┐    ──>    ┌────────┐
│ Review │           │ Phân loại  │           │  Soạn    │           │  Gửi   │
│ Mới   │           │ khiếu nại  │           │  thảo    │           │ & theo │
│ Nhận  │           │ (delay/态度/│           │  phản hồi│           │  dõi   │
│ thông │           │  xe脏/...)  │           │  cá nhân │           │  kết   │
│ báo   │           │            │           │  hóa    │           │  quả   │
└────────┘           └────────────┘           └──────────┘           └────────┘
    │                      │                      │                      │
    └──────────────────────┴──────────────────────┴──────────────────────┘
                                     │
                                     ▼
                        🔴 BOTTLENECK: Bước 3
                        Soạn thảo phản hồi mất 10-15 phút
                        CSKH phải suy nghĩ cách diễn đạt
                        Không có template chuẩn
                        Trả lời mỗi người mỗi kiểu không nhất quán
                                     │
                                     ▼
                    ╔════════════════════════════════╗
                    ║  TỔNG THỜI GIAN: 12-15 PHÚT   ║
                    ║  (mỗi review tiêu cực)        ║
                    ║  200 reviews/tháng = 40 giờ   ║
                    ╚════════════════════════════════╝

LEGEND:
─────────────────────────────────────────
🔴 BOTTLENECK: Bước gây tắc nghẽn, tốn thời gian
🔄 HANDOFF: Điểm chuyển giao giữa người và hệ thống
```

### Thống kê hiện tại:

| Metric | Giá trị hiện tại |
|--------|------------------|
| Thời gian xử lý/trung bình | 12-15 phút/lượt |
| Số lượng reviews tiêu cực/tháng | 200-400 reviews |
| Tổng thời gian CSKH bỏ ra/tháng | 40-100 giờ |
| Tỷ lệ phản hồi trong 24h | ~60% |
| Tỷ lệ phản hồi đúng tone hãng | ~70% (không nhất quán) |
| NPS impact ước tính | -5 điểm NPS/quý |

---

## 3.3. Problem Statement (6-field)

### 6-Field Problem Statement:

| Field | Nội dung chi tiết |
|-------|-------------------|
| **1. Actor / Operator** | Nhân viên Chăm sóc khách hàng (CSKH) của Xanh SM, đội ngũ 15 người, làm việc 3 ca. Mỗi nhân viên xử lý trung bình 15-20 reviews tiêu cực/ngày. |
| **2. Current Workflow** | Khi nhận thông báo review 1-2 sao từ hệ thống (app Xanh SM, Google Play, App Store), CSKH: (1) Đọc nội dung, (2) Phân loại loại khiếu nại theo cảm nhận, (3) Tự soạn thảo phản hồi dựa trên kinh nghiệm cá nhân, (4) Gửi phản hồi. Công cụ: Zendesk, Google Sheets để tracking. Không có template chuẩn, không có AI hỗ trợ. |
| **3. Bottleneck** | **Bước soạn thảo phản hồi (Step 3)** là bottleneck chính. CSKH mất 10-15 phút/review để: suy nghĩ cách diễn đạt phù hợp, đảm bảo tone hãng, cá nhân hóa theo nội dung complaint. Ngoài ra: (a) Không có tiêu chuẩn → phản hồi không nhất quán, (b) CSKH mới thiếu kinh nghiệm → phản hồi kém chất lượng, (c) Peak hours (21h-23h) CSKH ít người → backlog tồn đọng. |
| **4. Business Impact** | - **Chi phí nhân công:** 40-100 giờ CSKH/tháng = ~15-30 triệu VNĐ/tháng (ước tính)<br>- **Brand reputation:** Phản hồi chậm/trễ ảnh hưởng NPS, có khả năng khách hàng chia sẻ negative experience<br>- **CSKH burnout:** 15 phút/review × 20 reviews = 5 giờ/ngày chỉ để soạn phản hồi<br>- **Opportunity cost:** Thời gian CSKH có thể dùng để xử lý case phức tạp hơn |
| **5. Success Metric** | **Primary:** Giảm thời gian xử lý mỗi phản hồi từ **12 phút → dưới 2 phút** (87% reduction)<br>**Secondary:** Tăng tỷ lệ phản hồi trong 24h từ **60% → 95%**<br>**Tertiary:** Tăng consistency score (đo bằng A/B test) từ **70% → 90%**<br>**Guardrail:** CSAT của khách hàng sau khi nhận phản hồi AI không giảm quá 5% |
| **6. Operational Boundary** | **AI ĐƯỢC PHÉP:**<br>• Soạn draft phản hồi dựa trên complaint category + tone guidelines<br>• Đề xuất 2-3 phương án phản hồi để CSKH chọn<br>• Phân loại tự động loại khiếu nại (chỉ gợi ý, không auto-assign)<br><br>**AI TUYỆT ĐỐI KHÔNG ĐƯỢC:**<br>• Tự động gửi phản hồi mà không có CSKH duyệt (MỌI phản hồi phải qua human review)<br>• Đưa ra cam kết bồi thường, giảm giá mà không có approval<br>• Sửa đổi giá cước, hủy booking<br>• Tiết lộ thông tin cá nhân khách hàng<br><br>**HITL Points (Human-in-the-Loop):**<br>1. CSKH duyệt/edit/dismiss draft trước khi gửi<br>2. Trường hợp complaint về tai nạn/an ninh → bắt buộc supervisor duyệt<br>3. Khách hàng reply lại → AI không tự xử lý, chuyển về CSKH |

---

## 3.4. Future-State Flow & AI Fit

### 3.4.1. AI Fit Matrix Classification

| Criteria | Assessment |
|----------|------------|
| **Độ phức tạp của input** | Medium - Text có cấu trúc, có context từ review history |
| **Độ phức tạp của output** | Medium - Phản hồi ngắn gọn, đúng tone, cá nhân hóa |
| **Yêu cầu real-time** | Low - Không cần immediate response (30s-2min delay OK) |
| **Error tolerance** | Medium-High - Có human review, error không critical |
| **Compliance/Regulation** | Medium - Cần đảm bảo không cam kết sai |

### 🔷 AI Fit Decision: **LLM Feature** (không phải Agentic Loop)

**Lý do không chọn Agentic Loop:**
- Không cần multi-step planning phức tạp
- Không cần tool calling nhiều (chỉ generate text)
- Có human review nên không cần autonomous loop
- Rủi ro khi LLM tự động cao → cần human checkpoint

**Lý do không chọn Rule-based:**
- Input quá đa dạng (100+ cách khách hàng mô tả vấn đề)
- Cần hiểu nuance, sarcasm, context
- Tone phản hồi cần tự nhiên, không robotic

### 3.4.2. Future-State Flow (Text Diagram)

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    FUTURE STATE: AI-ASSISTED WORKFLOW                       │
│                    Thời gian trung bình: 90 giây/lượt (↓87%)               │
└─────────────────────────────────────────────────────────────────────────────┘

    [1]                    [2]                    [3]                    [4]
      │                      │                      │                      │
      ▼                      ▼                      ▼                      ▼
┌────────────┐    ──>   ┌────────────┐    ──>   ┌──────────┐    ──>   ┌────────┐
│  Review    │          │    AI      │          │  Draft   │          │  CSKH  │
│  Mới      │          │ Phân loại  │          │ Phản hồi │          │ Review │ ──> [5]
│  (Auto)   │          │ + Generate │          │ được tạo │          │ & Edit │
│  notify   │          │   sẵn sàng │          │  tự động  │          │        │
└────────────┘          └────────────┘          └──────────┘          └────────┘
      │                      │                      │                      │
      │                      │                      │              🟢 HUMAN STEP
      │                      │                      │              (HITL - Required)
      │                      │                      │                      │
      └──────────────────────┴──────────────────────┴──────────────────────┘
                                     │
                                     ▼
    ╔══════════════════════════════════════════════════════════════════╗
    ║                    🔵 AI PROCESSING STEPS                         ║
    ╠══════════════════════════════════════════════════════════════════╣
    ║  Step 2.1: Classification (LLM)                                  ║
    ║    Input: Review text → Output: Category tag                       ║
    ║    Categories: [delay | attitude | dirty_car | wrong_route |      ║
    ║                 pricing | app_issue | safety | other]             ║
    ║                                                                  ║
    ║  Step 2.2: Draft Generation (LLM)                               ║
    ║    Input: Review + Category + Tone Guidelines → Output: 2-3 draft ║
    ║    System Prompt: Enforce brand voice, max 150 chars, empathetic  ║
    ╚══════════════════════════════════════════════════════════════════╝

LEGEND:
─────────────────────────────────────────────────────────────────────────────────
🔵 AI STEP: Xử lý tự động bởi LLM
🟢 HUMAN STEP (HITL): Bắt buộc có con người duyệt/approve
↩️ FALLBACK: Kế hoạch dự phòng khi AI gặp lỗi
```

### 3.4.3. Fallback Strategy (Kế hoạch dự phòng)

| Scenario | Fallback Action |
|----------|-----------------|
| **LLM timeout (>5s)** | Hiển thị "Draft đang được tạo..." → retry 1 lần → nếu fail, dùng template cứng theo category |
| **LLM return empty/invalid** | Fallback về template có sẵn: "Cảm ơn bạn đã phản hồi. Chúng tôi đang xem xét và sẽ liên hệ sớm nhất." |
| **Confidence score < 0.7** | Gắn tag "needs_supervisor_review" → ưu tiên xử lý |
| **Safety/compliance keywords detected** | Tự động flag: "⚠️ Cần supervisor approval" → block auto-draft |
| **Customer mentions lawsuit/legal** | Immediate escalation, không tạo draft, notify supervisor |
| **API down** | Fallback về queue mode, CSKH xử lý manual (workflow cũ) |

### 3.4.4. Human-in-the-Loop (HITL) Points

```
┌─────────────────────────────────────────────────────────────────┐
│                 HITL DECISION MATRIX                           │
├─────────────────────┬─────────────────────────────────────────┤
│ Review Content      │ Required Action                          │
├─────────────────────┼─────────────────────────────────────────┤
│ ⭐ 1-2 sao thường   │ CSKH duyệt draft + send (Standard)    │
│ ⭐ 1 sao + emotional│ CSKH duyệt + Supervisor notification   │
│ ⚠️ Safety incident  │ Supervisor bắt buộc duyệt               │
│ ⚠️ Legal keywords   │ Legal team notification + supervisor    │
│ 🚨 Repeat customer  │ Supervisor duyệt + CRM update           │
└─────────────────────┴─────────────────────────────────────────┘
```

---

# 📊 Phase 5 — EVALUATE

## 5.1. AI Readiness Checklist

| # | Tiêu chí | Status | Ghi chú |
|---|----------|--------|---------|
| 1 | **Chúng tôi có sẵn dữ liệu mẫu/logs sạch để test?** | ✅ YES | 500+ historical reviews & responses từ Zendesk export |
| 2 | **Rủi ro khi AI sai có nằm trong tầm kiểm soát (qua HITL hoặc Fallback)?** | ✅ YES | Mọi draft đều qua human review trước khi gửi |
| 3 | **Stakeholders sẵn sàng thay đổi quy trình làm việc cũ?** | ✅ YES | CSKH team lead đã approve POC concept |
| 4 | **Chúng tôi có define rõ ranh giới AI được làm gì / không được làm?** | ✅ YES | 6-field đã định nghĩa rõ Operational Boundary |
| 5 | **Chúng tôi có tiêu chuẩn đo lường thành công (metrics)?** | ✅ YES | 3 metrics đã định nghĩa với baseline và target |
| 6 | **Chúng tôi có budget cho API cost (LLM calls)?** | ⚠️ TBD | Ước tính ~$50-100/tháng, cần stakeholder approval |
| 7 | **Chúng tôi có internal LLM expertise?** | ✅ YES | Team có 2 members đã làm việc với Gemini/GPT API |
| 8 | **Compliance đã được review?** | ⚠️ PARTIAL | Cần legal review cho policy "AI-generated responses" |

### Tổng hợp: 6/8 ✅ | 2 ⚠️ cần address trước khi GO

---

## 5.2. Cost Estimation

| Component | Ước tính chi phí |
|-----------|-------------------|
| **LLM API (Gemini 2.5 Flash)** | ~$30-50/tháng (2000 req/day × 30 days) |
| **Development (1 sprint)** | 40 dev-hours (internal team) |
| **Infrastructure (if needed)** | ~$20-50/tháng (Cloud Run / Cloud Functions) |
| **Maintenance/Iteration** | 8h/tháng (bug fixes, prompt tuning) |
| **TOTAL/month (RUNNING)** | ~$50-100/tháng |
| **ROI vs Current State** | Tiết kiệm 40-100h CSKH/tháng = 15-30 triệu VNĐ → ROI positive ngay tháng đầu |

---

## 5.3. Final Decision

### 🎯 DECISION: **GO** (Bắt đầu xây dựng Prototype)

---

### Justification (Lý giải quyết định dựa trên bằng chứng kỹ thuật)

| Evidence | Supporting Data |
|----------|-----------------|
| **Technical Feasibility** | LLM text generation cho use case này đã proven (many companies doing this). Gemini 2.5 Flash đủ capable, cost-effective. |
| **Data Availability** | Có 500+ historical samples để fine-tune/prompt engineer. Không cần data labeling mới. |
| **Risk Management** | HITL ở mọi response → risk gần như bằng 0. Fallback strategy rõ ràng. Compliance concerns addressable với legal review. |
| **Business Impact** | 87% time reduction = significant cost savings. KPIs đo lường được. Quick win để demonstrate AI value. |
| **Resource Availability** | Internal team có capability. Budget reasonable. Timeline: 2-4 weeks cho MVP. |

### Conditions for GO:
1. Legal review và approve policy cho "AI-assisted response system"
2. Stakeholder buy-in từ CSKH team lead và Operations director
3. UAT với 10 CSKH users trước khi full rollout

### Scope for MVP (2-week sprint):
```
MVP Features:
├── Review notification trigger
├── LLM classification (8 categories)
├── Draft generation (2-3 options)
├── HITL review interface
├── Basic analytics dashboard
└── Fallback to manual mode

Out of Scope for MVP:
├── Supervisor escalation workflow
├── Multi-language support
├── Integration với CRM
└── Fine-tuning/fine-tuned model
```

---

## 5.4. Next Steps (Action Items)

| # | Action | Owner | Due |
|---|--------|-------|-----|
| 1 | Legal review AI response policy | [TBD] | Week 1 |
| 2 | Data export và preprocessing (500+ samples) | Dev | Week 1 |
| 3 | System prompt engineering + testing | Dev | Week 1-2 |
| 4 | Build HITL review interface | Dev | Week 2 |
| 5 | UAT với 10 CSKH users | CSKH Lead | Week 2 |
| 6 | Full rollout planning | Operations | Week 3 |

---

## 📎 Appendix: Quick Reference

### Problem Card Summary (from Phase 2):

```
Bài toán: Soạn thảo phản hồi tự động cho đánh giá tiêu cực Xanh SM
Actor: CSKH (15 người, 3 ca)
Bottleneck: Soạn thảo mất 10-15 phút/review
Solution: LLM-assisted draft generation với HITL
Target: Giảm từ 12 phút → 2 phút (87% reduction)
AI Fit: LLM Feature (not Agentic Loop)
Risk: Low (với HITL)
```

---

*Document prepared by: Nhóm [Tên nhóm]*
*Date: [Ngày nộp]*
*Status: ✅ Ready for submission*
