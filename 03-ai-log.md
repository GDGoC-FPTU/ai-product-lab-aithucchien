### Nhật ký chiêm nghiệm về việc tương tác với AI

Trong suốt buổi học, tôi sử dụng ChatGPT làm trợ lý hỗ trợ để brainstorm ý tưởng, phân tích quy trình hiện tại (Current-State Workflow), xác định Bottleneck, xây dựng Future-State Workflow và gợi ý cách áp dụng AI vào từng bước của quy trình. Ngoài ra, tôi còn sử dụng AI để giải thích các khái niệm về AI Agent, LLM, Human-in-the-loop, hỗ trợ viết và sửa các đoạn mã Python gọi API, streaming chatbot, đếm token và ước tính chi phí sử dụng mô hình.

Tuy nhiên, AI không phải lúc nào cũng đưa ra câu trả lời chính xác. Có trường hợp AI đề xuất giải pháp quá phức tạp so với yêu cầu bài tập hoặc giả định thêm các thông tin không có trong đề bài. Ví dụ, khi xây dựng quy trình AI, AI từng gợi ý sử dụng Agentic Workflow trong khi bài toán chỉ cần một tính năng phân loại bằng LLM. Ngoài ra, ở một số câu hỏi về lập trình, AI còn đề xuất những hàm hoặc cấu trúc không đúng với khung mã mà giảng viên đã cung cấp.

Để khắc phục, tôi điều chỉnh prompt theo hướng cụ thể hơn, yêu cầu AI chỉ sử dụng đúng framework và các hàm có sẵn trong đề, không tự thêm thư viện hoặc chức năng ngoài phạm vi. Tôi cũng bổ sung các ràng buộc như "giữ nguyên cấu trúc code", "chỉ hoàn thành phần TODO", hoặc "dựa đúng theo docstring và yêu cầu của đề bài". Sau mỗi lần AI trả lời, tôi đối chiếu lại với tài liệu học và chạy thử chương trình để kiểm chứng kết quả trước khi sử dụng.

Qua trải nghiệm này, tôi nhận thấy AI là một công cụ hỗ trợ rất hiệu quả trong việc gợi ý ý tưởng, giải thích kiến thức và tăng tốc quá trình lập trình. Tuy nhiên, người sử dụng vẫn cần tư duy phản biện, kiểm tra tính đúng đắn của kết quả và đưa ra quyết định cuối cùng thay vì phụ thuộc hoàn toàn vào AI.
