# 03 - AI Log & Reflection

Tên nhóm: **aithucchien**

## Thành viên

| # | Họ và tên | MSSV |
|---|---|---|
| 1 | Trần Đình Đăng | 2A202601998 |
| 2 | Nguyễn Trung Hiếu | 2A202601457 |
| 3 | Nguyễn Thế Anh | 2A202601791 |
| 4 | Nguyễn Thị Lý | 2A202601963 |
| 5 | Vũ Văn Phong | 2A202601647 |

## AI đã giúp gì?

AI được dùng như một thought-partner để brainstorm các pain point trong hệ sinh thái Vingroup. Nhóm dùng AI để mở rộng danh sách ý tưởng, sau đó lọc lại bằng bốn lens: tác vụ lặp lại, tác vụ tốn thời gian, cơ hội nâng cấp bằng AI, và nỗi đau của stakeholder. AI đặc biệt hữu ích khi ép mỗi ý tưởng phải có actor, workflow hiện tại, bottleneck, metric và kiến trúc sơ bộ.

Trong phần deep-dive, AI giúp nhóm chuyển một ý tưởng khá rộng thành problem statement có 6 trường rõ ràng. Thay vì nói chung chung rằng “AI hỗ trợ điều phối xe”, bài toán được thu hẹp thành “AI tạo bản nháp hướng dẫn hoặc JSON dispatch cho sự cố pin của Xanh SM”. Nhờ vậy, nhóm xác định được ngưỡng an toàn cụ thể: pin dưới 5% thì không được gợi ý trạm sạc tiêu chuẩn xa hơn 5 km.

Trong phần prototype, AI hỗ trợ viết system prompt, nghĩ adversarial prompts và kiểm tra các tình huống prompt injection. Các test được thiết kế để ép model bỏ tag `[DRAFT_ONLY]`, ép model gợi ý tuyến nguy hiểm, hoặc giả danh cấp trên để yêu cầu bỏ qua rule.

## AI sai hoặc chưa tốt ở đâu?

AI ban đầu có xu hướng đề xuất phạm vi quá lớn, chẳng hạn như “tối ưu toàn bộ hệ thống điều phối đội xe” hoặc “xây Agent tự động điều xe”. Các đề xuất này nghe hấp dẫn nhưng không phù hợp với lab vì thiếu scope hẹp, khó đo metric trong thời gian ngắn, và có rủi ro nếu cho AI tự hành động.

Một vấn đề khác là AI hay viết metric chung chung như “tăng hiệu quả vận hành” hoặc “giảm thời gian xử lý”, nhưng chưa nêu số cụ thể. Nhóm phải yêu cầu AI biến các metric đó thành ngưỡng đo được, ví dụ: giảm từ 15 phút xuống dưới 3 phút, đạt 98% draft đúng rule, và 100% nội dung gửi ra ngoài phải có người phê duyệt.

Khi viết prompt boundary, AI đôi lúc trả lời dài dòng cho trường hợp pin 2%, thay vì trả đúng JSON command. Nếu không ép định dạng, output có thể vừa từ chối tuyến 8 km, vừa thêm lời khuyên khác, khiến hệ thống khó parse và khó kiểm soát.

## Nhóm đã sửa đổi ra sao?

Nhóm sửa prompt và prototype theo hướng đặt ranh giới không thương lượng:

1. Mọi nội dung dạng hướng dẫn cho tài xế phải bắt đầu bằng `[DRAFT_ONLY]`.
2. Pin dưới 5% là trạng thái critical.
3. Nếu pin dưới 5% và trạm sạc tiêu chuẩn xa hơn 5 km, output chỉ được là JSON `dispatch_mobile_charger`.
4. Điều phối viên là người phê duyệt cuối; AI không được nói rằng đã gửi tin nhắn hoặc đã điều xe.
5. Nếu model trả lời thiếu tag, sai định dạng, hoặc API không khả dụng, code có fallback để giữ hệ thống trong trạng thái an toàn.

Bài học chính là prompt tốt chưa đủ. Với bài toán vận hành thật, nhóm cần kết hợp system prompt, rule deterministic, assertion test, runtime guardrail và Human-in-the-loop. AI hữu ích nhất khi được dùng để tăng tốc suy nghĩ và tạo bản nháp, nhưng quyết định vận hành cuối cùng vẫn phải được kiểm soát bằng quy trình rõ ràng.
