# Lab 21 — Evaluation Report

**Học viên**: Nguyễn Đức Cường — 2A202600147
**Ngày nộp**: 2026-05-07
**Submission option**: Option A (lightweight)

## 1. Setup
- **Base model**: `unsloth/Qwen2.5-3B-bnb-4bit`
- **Dataset**: `5CD-AI/Vietnamese-alpaca-gpt4-gg-translated`, 200 samples (180 train + 20 eval)
- **max_seq_length**: 1024 (p95 = 562, giới hạn ở mức 1024)
- **GPU**: Tesla T4, 16 GB VRAM (chạy trên Kaggle)
- **Training cost**: ~$0.05 (Tổng thời gian train 3 adapter khoảng ~12 phút)

## 2. Rank Experiment Results

| Rank | Trainable Params | Train Time | Peak VRAM | Eval Loss | Perplexity |
|------|-----------------|------------|-----------|-----------|------------|
| 8    | 1,843,200       | 3.83 min   | 7.22 GB   | 1.557     | 4.75       |
| 16   | 3,686,400       | 3.81 min   | 6.62 GB   | 1.516     | 4.55       |
| 64   | 14,745,600      | 3.93 min   | 8.00 GB   | 1.477     | 4.38       |

*(Ghi chú: Peak VRAM của r=16 có thể hiển thị thấp hơn một chút trong quá trình đo lường do sự khác biệt của cơ chế giải phóng bộ nhớ (garbage collection) giữa các lần chạy)*

## 3. Loss Curve Analysis
*[Bạn đính kèm hình ảnh `loss_curve.png` sinh ra từ notebook vào file zip]*
- **Quan sát**: Do giới hạn phần cứng của profile T4, đánh giá (evaluation) trong lúc train đã được tắt để tránh lỗi Out Of Memory, nên chúng ta chỉ quan sát được Train Loss Curve.
- Train Loss giảm đều đặn từ khoảng 1.61 xuống 1.39 và hội tụ ổn định sau 3 epoch. Không có dấu hiệu spike (tăng vọt) bất thường, cho thấy mô hình học tập tốt và không bị gradient explosion. Do eval loss đo được ở cuối quá trình duy trì ở mức thấp (1.4 - 1.5), mô hình không bị overfitting quá trầm trọng.

## 4. Qualitative Comparison (5 examples)

### Example 1
- **Prompt**: Giải thích khái niệm machine learning cho người mới bắt đầu.
- **Base**: Machine learning là một phân khúc của trí tuệ nhân tạo, nó tập trung vào việc thiết lập các mô hình máy móc để học tập...
- **Fine-tuned**: Machine learning là một bộ môn công nghệ máy tính dựa trên việc học tập và cải thiện các dự đoán dựa trên dữ liệu mà không có sự hướng dẫn trực tiếp...
- **Nhận xét**: Cả hai đều trả lời đúng, nhưng bản Fine-tuned có văn phong mượt mà, tự nhiên và dùng từ vựng kỹ thuật chính xác hơn ("bộ môn công nghệ máy tính", "hướng dẫn trực tiếp").

### Example 2
- **Prompt**: Viết đoạn code Python tính số Fibonacci thứ n.
- **Base**: Trả về hàm đệ quy cơ bản, nhưng xử lý lỗi không tốt (n<=0 lại trả về string text thay vì Exception).
- **Fine-tuned**: Bổ sung xử lý lỗi cực kỳ chuẩn mực bằng `raise ValueError("Input phải là một số nguyên dương.")`, logic code vòng lặp viết tốt hơn.
- **Nhận xét**: Improved. Code của bản Fine-tuned được viết chuyên nghiệp hơn với thông báo lỗi hợp lý.

### Example 3
- **Prompt**: Liệt kê 5 nguyên tắc thiết kế UI/UX.
- **Base**: Viết một đoạn dài mô tả nguyên tắc thứ nhất, chưa kịp chuyển sang các nguyên tắc tiếp theo.
- **Fine-tuned**: Liệt kê rõ ràng và ngắn gọn từng điểm (Chuyển đổi, Thích ứng, Đơn giản, Tương thích).
- **Nhận xét**: Improved. Mô hình Fine-tuned đã học được cách format câu trả lời dưới dạng danh sách rõ ràng, thân thiện với người đọc.

### Example 4
- **Prompt**: Tóm tắt sự khác biệt giữa LoRA và QLoRA.
- **Base**: Giải thích LoRA khá ổn nhưng bị ngắt quãng nửa chừng.
- **Fine-tuned**: Bị ảo giác (hallucination) nghiêm trọng khi tự chế ra định nghĩa sai lệch về LoRA là "Layer-wise Adaptive Regularization Optimization".
- **Nhận xét**: Degraded. Việc fine-tune trên dữ liệu chung (Alpaca general) không giúp mô hình nâng cao kiến thức chuyên ngành hẹp, dẫn đến bịa đặt thông tin kỹ thuật sâu. 

### Example 5
- **Prompt**: Phân biệt prompt engineering, RAG, và fine-tuning.
- **Base**: Định nghĩa cơ bản về prompt engineering nhưng dừng lại đột ngột.
- **Fine-tuned**: Cấu trúc câu gọn gàng hơn nhưng cũng bị cắt ngang do chạm giới hạn token (max_new_tokens).
- **Nhận xét**: Same. Cấu trúc câu của fine-tuned tự nhiên hơn nhưng cả 2 đều chưa thể hiện được sự khác biệt giữa 3 thuật ngữ một cách trọn vẹn.

## 5. Conclusion về Rank Trade-off

Từ bảng kết quả ở phần 2, chúng ta có thể rút ra một số nhận xét sau:
- **Rank cho ROI tốt nhất**: Rank `r=16` mang lại ROI (Return on Investment) tốt nhất. So với `r=8`, nó làm giảm perplexity rõ rệt (từ 4.75 xuống 4.55) nhưng lại không tiêu tốn thêm thời gian training (đều mất khoảng 3.8 phút).
- **Diminishing returns (Hiệu suất giảm dần)**: Khi tăng rank lên `r=64`, số lượng tham số huấn luyện tăng vọt lên gấp 4 lần (từ ~3.6 triệu lên ~14.7 triệu tham số). Mặc dù perplexity tiếp tục được tối ưu hóa xuống mức 4.38, mức độ cải thiện này không còn tương xứng với cái giá phải trả về bộ nhớ (VRAM chạm mức 8GB).
- **Recommendation**: Nếu cần deploy dự án lên production, tôi sẽ ưu tiên chọn rank `r=16`. Rank này mang lại sự cân bằng hoàn hảo giữa khả năng học hỏi các pattern mới từ dataset (thể hiện qua perplexity tốt) và chi phí tài nguyên phần cứng (VRAM) cần thiết để inference hay scale up.

## 6. What I Learned
- **Sức mạnh của QLoRA và Unsloth**: Sử dụng Unsloth kết hợp quantization 4-bit giúp tối ưu cực tốt tài nguyên. Tôi có thể dễ dàng fine-tune một LLM 3B tham số trên GPU T4 16GB free của Kaggle/Colab trong chưa tới 15 phút.
- **Lựa chọn tham số LoRA**: Việc tăng tham số rank `r` không tỷ lệ thuận hoàn toàn với độ thông minh của mô hình. Chọn `r` hợp lý (thường là 8 hoặc 16 cho các tập dữ liệu nhỏ) giúp tiết kiệm tài nguyên mà vẫn đạt hiệu quả gần bằng rank lớn.
- **Bản chất của Instruction Tuning**: Cung cấp dữ liệu mẫu theo định dạng Alpaca chủ yếu giúp mô hình hiểu được cách format câu trả lời (thích liệt kê, rõ ràng, sửa code xịn hơn). Tuy nhiên, nó không giúp "nhồi nhét" kiến thức chuyên môn hẹp nếu dữ liệu train không có sẵn mảng kiến thức đó (ví dụ như mô hình bị ảo giác khi hỏi về định nghĩa sâu của LoRA/QLoRA).
