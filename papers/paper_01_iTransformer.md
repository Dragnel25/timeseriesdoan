# Tóm tắt: iTransformer: Inverted Transformers Are Effective for Time Series Forecasting
**Link bài báo:** [https://arxiv.org/abs/2310.06625](https://arxiv.org/abs/2310.06625)

- **Vấn đề nghiên cứu:** Trong bài toán dự báo chuỗi thời gian nhiều chiều (Multivariate Time Series), các mô hình Transformer truyền thống thường cắt dữ liệu theo các khoảng thời gian (patching) để tạo token. Điều này vô tình phá vỡ ngữ nghĩa vật lý cốt lõi của từng biến độc lập và làm suy giảm khả năng nắm bắt sự tương quan giữa các biến.
- **Ý tưởng chính:** Đảo ngược (Invert) hoàn toàn cơ chế của Transformer. Thay vì token hóa theo thời gian, iTransformer xem toàn bộ lịch sử (look-back window) của một biến là một token duy nhất.
- **Mô hình đề xuất:** iTransformer áp dụng cơ chế Attention lên các "variate tokens" (token biến) này để mô hình hóa trực tiếp sự tương quan không gian (cross-variate correlation), đồng thời dùng Feed-Forward Network để trích xuất đặc trưng phi tuyến tính.
- **Kết quả chính:** Mô hình đạt trạng thái SOTA (State-of-the-Art) trên hàng loạt bộ dữ liệu thực tế. Đặc biệt, khi tăng độ dài cửa sổ lịch sử (look-back window), hiệu năng của iTransformer không bị suy giảm mà còn tăng lên, vượt qua các mô hình Transformer cũ.
- **Điểm mạnh, hạn chế:** - *Điểm mạnh:* Cực kỳ hiệu quả trong việc nắm bắt tương quan giữa nhiều biến số, khả năng tổng quát hóa cao.
  - *Hạn chế:* Đòi hỏi tài nguyên tính toán (bộ nhớ) lớn nếu số lượng biến đầu vào ($d$) quá khổng lồ.
- **Khả năng áp dụng:** Hoàn toàn phù hợp để làm kiến trúc lõi cho bài toán dự báo phụ tải điện nhiều chiều, nơi các biến phụ (như nhiệt độ, độ ẩm) có mối tương quan vật lý chặt chẽ với biến mục tiêu (phụ tải).
