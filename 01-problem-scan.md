# Phase 1
| # | Quy trình | Pain point thủ công | Ước tính tổn thất |
|---|---|---|---|
| 1 | Điều phối lại chuyến xe khi khách hủy hoặc xe hỏng | Nhân viên phải kiểm tra nhiều kênh, gọi tài xế, sắp xếp chuyến mới thủ công; mỗi sự cố mất 10–15 phút | 2.000–3.000 sự cố/tháng; 4–6% chuyến bị bỏ lỡ hoặc chậm trễ; tổn thất 2,5–4,5 tỷ VNĐ/tháng |
| 2 | Xử lý phản hồi khách hàng và khiếu nại về chuyến đi | Nhân viên phải đọc nội dung chat/call, phân loại vấn đề, soạn câu trả lời lặp lại | 3.000–5.000 case/tháng; 20–30% case cần handoff nhiều lần; làm tăng chi phí vận hành 8–12% |
| 3 | Tối ưu lộ trình cho tài xế trong giờ cao điểm | Hệ thống hiện tại còn phụ thuộc vào việc điều phối thủ công, dẫn tới lộ trình dài hơn và thời gian chờ dài | 10–15% quãng đường dư thừa; tăng chi phí nhiên liệu và thời gian xe chạy không hiệu quả khoảng 1,2–2,0 tỷ VNĐ/tháng |
| 4 | Tổng hợp dữ liệu sự cố và báo cáo vận hành hàng ngày | Nhân viên phải thu thập từ nhiều nguồn: app, GPS, call center, tài xế rồi nhập thủ công vào báo cáo | Mỗi ngày mất 2–3 giờ cho việc tổng hợp; 15–20% dữ liệu bị chậm cập nhật, làm sai lệch quyết định vận hành |
| 5 | Kiểm tra hồ sơ và giấy tờ tài xế trước khi phân công chuyến | Việc đối chiếu thông tin giấy tờ, giấy phép, lịch trình, tình trạng xe làm chậm quy trình và dễ sai sót | 5–8% hồ sơ chưa đầy đủ bị phát hiện muộn; làm giảm hiệu suất phân công và tăng rủi ro vận hành |

# 🃏 Phase 2
┌─────────────────────────────────────────────────────────────┐
│ QUICK PROBLEM CARD #1 - ĐIỀU PHỐI LẠI CHUYẾN XE             │
│                                                             │
│ Bài toán (1 câu): Tìm xe thay thế ngay lập tức dưới 5 giây  │
│ để cứu cuốc xe khi khách hủy ngang hoặc xe cũ bị hỏng.      │
│                                                             │
│ Công ty thành viên: [ ] VinFast  [x] Xanh SM  [ ] Vinhomes  │
│                     [ ] Vinmec   [ ] Khác (Ghi rõ)________  │
│                                                             │
│ Ai đang đau (Actor)? Khách hàng (bị bỏ rơi) & Hệ thống gán. │
│                                                             │
│ Workflow thủ công hiện tại (3-5 bước):                      │
│   1. Nhận ping báo hủy/hỏng ──> 2. Hủy cuốc xe hiện tại ──> │
│   3. Quét tọa độ tìm xe rảnh ──> 4. Gán cuốc cho xe mới     │
│                                                             │
│ Bước nào tốn thời gian/lỗi nhất? B3 (⏱ 30-60 giây/lượt)     │
│                                                             │
│ AI có thể nhảy vào hỗ trợ ở bước nào? KHÔNG CẦN THIẾT. Đây  │
│ là bài toán truy vấn không gian (Spatial query).            │
│                                                             │
│ Đo thành công bằng gì (Metric có số)?                       │
│ Tăng tỷ lệ cứu cuốc (Recovery rate) từ 60% ──> over 95%.    │
│ Giảm độ trễ gán xe từ 30s ──> under 3s.                     │
│                                                             │
│ Quick Architecture: [ ] No AI  [x] Rule  [ ] LLM  [ ] Agent │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│ QUICK PROBLEM CARD #2 - TỐI ƯU LỘ TRÌNH GIỜ CAO ĐIỂM        │
│                                                             │
│ Bài toán (1 câu): Vẽ lộ trình ngắn nhất, tránh kẹt xe để    │
│ tài xế hoàn thành cuốc nhanh, tăng vòng quay đầu xe/ca.     │
│                                                             │
│ Công ty thành viên: [ ] VinFast  [x] Xanh SM  [ ] Vinhomes  │
│                     [ ] Vinmec   [ ] Khác (Ghi rõ)________  │
│                                                             │
│ Ai đang đau (Actor)? Tài xế Xanh SM & Khách hàng.           │
│                                                             │
│ Workflow thủ công hiện tại (3-5 bước):                      │
│   1. Nhận điểm đón/trả ──> 2. Gọi API hệ thống bản đồ ──>   │
│   3. Vẽ Route & ETA ──> 4. Tài xế di chuyển theo Map        │
│                                                             │
│ Bước nào tốn thời gian/lỗi nhất? B2 - Map update traffic    │
│ chậm dẫn đến chỉ vào đường kẹt (⏱ kẹt thêm 15 phút/cuốc)    │
│                                                             │
│ AI có thể nhảy vào hỗ trợ ở bước nào? Dùng thuật toán đồ thị│
│ (Graph) chuẩn kết hợp mua API real-time traffic bên thứ 3.  │
│                                                             │
│ Đo thành công bằng gì (Metric có số)?                       │
│ Giảm độ lệch ETA (thực tế vs dự kiến) từ 20% ──> under 5%.  │
│                                                             │
│ Quick Architecture: [ ] No AI  [x] Rule  [ ] LLM  [ ] Agent │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│ QUICK PROBLEM CARD #3 - XỬ LÝ KHIẾU NẠI                     │
│                                                             │
│ Bài toán (1 câu): Phân loại và routing ticket khiếu nại về  │
│ đúng phòng ban giải quyết mà không cần đọc thủ công.        │
│                                                             │
│ Công ty thành viên: [ ] VinFast  [x] Xanh SM  [ ] Vinhomes  │
│                     [ ] Vinmec   [ ] Khác (Ghi rõ)________  │
│                                                             │
│ Ai đang đau (Actor)? Nhân viên CSKH (Tier 1).               │
│                                                             │
│ Workflow thủ công hiện tại (3-5 bước):                      │
│   1. Đọc ticket ──> 2. Phân loại (thái độ, giá, quên đồ) ──>│
│   3. Tra cứu lịch sử cuốc ──> 4. Trả lời hoặc chuyển tier 2 │
│                                                             │
│ Bước nào tốn thời gian/lỗi nhất? B2 (⏱ 3-5 phút/lượt)       │
│                                                             │
│ AI có thể nhảy vào hỗ trợ ở bước nào? B2 (Regex matching)   │
│                                                             │
│ Đo thành công bằng gì (Metric có số)?                       │
│ Giảm thời gian chạm ticket lần đầu từ 15 min ──> under 1 min│
│ Tiết kiệm chi phí nhân sự CSKH: 300 triệu VNĐ/tháng.        │
│                                                             │
│ Quick Architecture: [ ] No AI  [x] Rule  [ ] LLM  [ ] Agent │
└─────────────────────────────────────────────────────────────┘
