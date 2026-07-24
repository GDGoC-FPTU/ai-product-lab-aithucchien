# 01 - Problem Scan

Tên nhóm: **aithucchien**

## Thành viên

| # | Họ và tên | MSSV |
|---|---|---|
| 1 | Trần Đình Đăng | 2A202601998 |
| 2 | Nguyễn Trung Hiếu | 2A202601457 |
| 3 | Nguyễn Thế Anh | 2A202601791 |
| 4 | Nguyễn Thị Lý | 2A202601963 |
| 5 | Vũ Văn Phong | 2A202601647 |

## Phase 1 - SCAN

Trong vai trò AI Product Engineer tại Vin Smart Future, nhóm quét các hoạt động vận hành của nhiều công ty thành viên Vingroup bằng bốn lăng kính: tác vụ lặp lại, tác vụ tốn thời gian, cơ hội nâng cấp bằng AI, và nỗi đau của stakeholder. Mục tiêu của phase này là tìm ra các bài toán đủ cụ thể, có người vận hành rõ ràng, có bottleneck đo được, và có khả năng kiểm soát rủi ro nếu đưa AI vào quy trình.

| # | Công ty thành viên | Lens | Mô tả bài toán / bottleneck | Tín hiệu định lượng ban đầu |
|---|---|---|---|---|
| 1 | Xanh SM | Tốn thời gian | Điều phối viên xử lý sự cố pin yếu hoặc hết pin của taxi điện bằng cách nghe điện thoại, tra vị trí xe, tìm trạm sạc còn trống, rồi soạn tin nhắn hướng dẫn thủ công cho tài xế. | Khoảng 12-18 phút/lượt; dễ chậm hơn trong giờ cao điểm. |
| 2 | VinFast | Lặp lại | Nhân viên vận hành đối soát hóa đơn sạc điện và log trạm sạc đối tác hằng ngày. Việc so khớp mã trạm, biển số, thời gian sạc và số tiền còn phụ thuộc nhiều vào bảng tính thủ công. | 2-3 giờ/ngày cho mỗi cụm dữ liệu lớn; lỗi nhỏ có thể kéo dài sang kỳ đối soát sau. |
| 3 | Vinhomes | Stakeholder Pain | Nhân viên CSKH đọc phản ánh của cư dân về thang máy, vệ sinh, an ninh, tiện ích, phí dịch vụ, sau đó route đến đúng bộ phận phụ trách. Sai route làm cư dân phải chờ phản hồi lại. | 8-15 phút/ticket; sai route ước tính 8-12% ở nhóm ticket mô tả tự do. |
| 4 | Vinmec | Tốn thời gian | Bác sĩ hoặc điều dưỡng mất nhiều thời gian tóm tắt hồ sơ xuất viện, thuốc, kết quả xét nghiệm, lời dặn và lịch tái khám từ nhiều ghi chú khác nhau. | 20-30 phút/bệnh nhân; rủi ro thiếu chi tiết nếu bác sĩ quá tải. |
| 5 | Vinpearl/VinWonders | AI-upgrade | Tổng đài và quầy dịch vụ trả lời lặp lại các câu hỏi về vé, combo, giờ mở cửa, điều kiện hoàn/hủy, lịch sự kiện, nhưng thông tin thay đổi theo mùa và theo địa điểm. | 5-7 phút/cuộc hỏi đáp phức tạp; câu trả lời giữa các nhân viên có thể không đồng nhất. |
| 6 | Xanh SM | Lặp lại | Nhóm vận hành phân tích lý do hủy chuyến từ ghi chú tài xế, cuộc gọi khách hàng và log điều phối để tìm pattern theo khu vực, khung giờ, hoặc loại sự cố. | Phân tích thủ công theo tuần; khó phát hiện sớm điểm nóng vận hành. |

## Phase 2 - QUICK-ASSESS

Sau khi scan, nhóm chọn ba bài toán có khả năng prototype nhanh và có metric rõ nhất: Xanh SM xử lý sự cố pin, Vinhomes route phản ánh cư dân, và Vinmec tóm tắt hồ sơ xuất viện. Trong ba bài toán này, Xanh SM được chọn làm hướng deep-dive vì có workflow ngắn, có ranh giới an toàn rõ, và phù hợp với yêu cầu stress-test prompt bằng Gemini trong slide.

### Quick Problem Card #1 - Xanh SM xử lý sự cố pin yếu/hết pin

| Trường | Nội dung |
|---|---|
| Bài toán | Hỗ trợ điều phối viên Xanh SM xử lý sự cố pin yếu hoặc hết pin của taxi điện ngoài đường. |
| Công ty thành viên | Xanh SM (GSM). |
| Actor đang đau | Tài xế phải chờ hướng dẫn trong lúc pin thấp; điều phối viên chịu áp lực vì phải tra cứu nhanh, chính xác, và không được gợi ý tuyến nguy hiểm. |
| Workflow thủ công hiện tại | 1. Tài xế gọi tổng đài báo pin yếu. 2. Điều phối viên mở dashboard lấy GPS và biển số. 3. Điều phối viên tra trạm sạc VinFast còn trụ trống, đúng loại cổng sạc. 4. Điều phối viên soạn tin nhắn hướng dẫn đường đi. 5. Nếu pin quá thấp, điều phối viên liên hệ đội xe sạc/cứu hộ di động. |
| Bước tốn thời gian/lỗi nhất | Bước 3 và bước 4, khoảng 10-12 phút/lượt. Lỗi thường gặp là chọn trạm quá xa, thiếu thông tin rẽ đường, hoặc không xét ngưỡng pin dưới 5%. |
| AI có thể hỗ trợ ở đâu | AI nhận context có cấu trúc, kiểm tra ngưỡng pin, tạo bản nháp hướng dẫn có tag `[DRAFT_ONLY]`, hoặc trả JSON `dispatch_mobile_charger` khi pin dưới 5% và trạm tiêu chuẩn quá xa. |
| Metric thành công | Giảm thời gian xử lý từ 15 phút xuống dưới 3 phút/lượt; 98% bản nháp đúng rule pin/khoảng cách; 100% nội dung gửi ra ngoài phải có điều phối viên phê duyệt. |
| Quick Architecture | LLM Feature kết hợp rule guardrail và Human-in-the-loop. |

### Quick Problem Card #2 - Vinhomes route phản ánh cư dân

| Trường | Nội dung |
|---|---|
| Bài toán | Tự động phân loại và route phản ánh của cư dân Vinhomes đến đúng bộ phận xử lý. |
| Công ty thành viên | Vinhomes. |
| Actor đang đau | Nhân viên CSKH phải đọc nhiều mô tả tự do; cư dân phải chờ nếu ticket bị chuyển sai bộ phận. |
| Workflow thủ công hiện tại | 1. Cư dân gửi ticket trên app. 2. CSKH đọc nội dung và hỏi lại nếu thiếu thông tin. 3. CSKH gán nhãn sự cố. 4. Ticket được chuyển đến ban quản lý, kỹ thuật, an ninh, vệ sinh, hoặc kế toán. 5. CSKH theo dõi SLA và phản hồi cư dân. |
| Bước tốn thời gian/lỗi nhất | Bước 2-4, khoảng 8-15 phút/ticket. Sai route làm ticket bị treo thêm 1-2 ngày. |
| AI có thể hỗ trợ ở đâu | LLM đọc nội dung, trích tòa nhà, tầng, căn hộ, loại sự cố, mức khẩn cấp; rule router chọn bộ phận nhận ticket. |
| Metric thành công | 85% ticket được phân loại trong dưới 30 giây; giảm sai route từ 12% xuống dưới 4%; giảm ticket cần hỏi lại thông tin cơ bản xuống dưới 10%. |
| Quick Architecture | LLM Feature kết hợp rule router; chưa cần Agent vì quyết định chuyển bộ phận có thể kiểm soát bằng taxonomy. |

### Quick Problem Card #3 - Vinmec tóm tắt hồ sơ xuất viện

| Trường | Nội dung |
|---|---|
| Bài toán | Hỗ trợ bác sĩ Vinmec tạo bản nháp tóm tắt xuất viện và hướng dẫn chăm sóc sau điều trị. |
| Công ty thành viên | Vinmec. |
| Actor đang đau | Bác sĩ và điều dưỡng mất thời gian tổng hợp; bệnh nhân phải chờ giấy tờ xuất viện. |
| Workflow thủ công hiện tại | 1. Bác sĩ đọc bệnh án. 2. Bác sĩ tổng hợp chẩn đoán, thuốc, xét nghiệm và diễn biến điều trị. 3. Bác sĩ viết hướng dẫn tái khám và chăm sóc tại nhà. 4. Điều dưỡng in, giải thích, và lưu hồ sơ. |
| Bước tốn thời gian/lỗi nhất | Bước 1-3, khoảng 20-30 phút/bệnh nhân. Rủi ro là thiếu thuốc, thiếu lịch tái khám, hoặc diễn đạt khó hiểu với bệnh nhân. |
| AI có thể hỗ trợ ở đâu | LLM tạo bản nháp theo template từ dữ liệu có cấu trúc; bác sĩ bắt buộc kiểm tra và ký duyệt trước khi in/gửi. |
| Metric thành công | Giảm thời gian soạn từ 25 phút xuống dưới 8 phút; 100% bản nháp được bác sĩ duyệt; lỗi thiếu trường quan trọng dưới 1%. |
| Quick Architecture | LLM Feature với Human-in-the-loop nghiêm ngặt; chưa nên triển khai Agent do dữ liệu y tế có rủi ro cao. |

## Lựa chọn để Deep-Dive

Nhóm chọn **Quick Problem Card #1 - Xanh SM xử lý sự cố pin yếu/hết pin**. Lý do chọn là bài toán có actor rõ, workflow hiện tại ngắn nhưng gây áp lực thời gian thực, metric dễ đo, và ranh giới an toàn có thể viết thành rule cụ thể. Hai bài toán còn lại vẫn có giá trị, nhưng cần thêm dữ liệu nghiệp vụ, taxonomy hoặc quy trình phê duyệt chặt hơn trước khi prototype.
