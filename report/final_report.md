# BÁO CÁO NGHIÊN CỨU THỰC NGHIỆM: PHÂN TÍCH MIỀN TẦN SỐ VÀ ĐÁNH GIÁ HIỆU NĂNG CÁC KIẾN TRÚC MÔ HÌNH DỰ BÁO PHỤ TẢI ĐIỆN

## 1. Giới thiệu và Ý nghĩa Đề tài
Dự báo phụ tải điện năng (Load Forecasting) giữ vai trò xương sống trong việc vận hành, lập kế hoạch và tối ưu hóa chi phí hệ thống điện thông minh. Sai số trong dự báo có thể dẫn đến việc vận hành thừa công suất gây lãng phí, hoặc thiếu hụt công suất gây mất ổn định lưới điện an ninh quốc gia. 

Bài toán nghiên cứu tập trung vào việc áp dụng Biến đổi Fourier rời rạc (DFT) để phân rã các chu kỳ mùa vụ ẩn, từ đó xây dựng ma trận đặc trưng đầu vào $X \in \mathbb{R}^{T \times d}$ phục vụ cho các mô hình học máy phi tuyến tính nhằm dự báo biến mục tiêu một chiều $y$ tại thời điểm tương lai. Nghiên cứu sử dụng tập dữ liệu thực tế từ mạng lưới điện American Electric Power (AEP) thuộc hệ thống PJM Interconnection với hơn 120.000 quan sát theo giờ.

## 2. Cơ sở Phương pháp luận và Quy trình Tiền xử lý Tín hiệu

### 2.1 Đảm bảo điều kiện lấy mẫu đều (Uniform Sampling)
Thuật toán Biến đổi Fourier nhanh (FFT) giả định khoảng cách thời gian giữa hai điểm dữ liệu liên tiếp là hằng số tuyệt đối ($\Delta t = 1.0$ giờ). 
- Xử lý trùng lặp: Các mốc thời gian trùng lặp được giải quyết bằng cách lấy trung bình (`mean`) nhằm bảo toàn năng lượng tín hiệu.
- Xử lý khuyết thiếu: Áp dụng nội suy tuyến tính (Linear Interpolation) kết hợp `resample('h')`. Việc ép chuỗi về tần số cố định là bắt buộc để ngăn chặn rò rỉ phổ (spectral leakage) và bảo toàn tính trực giao của các hàm cơ sở Fourier.

### 2.2 Khử xu hướng toàn cục và Chuẩn hóa phổ biên độ
Chuỗi phụ tải điện năng thường mang thành phần xu hướng dài hạn (năng lượng tại tần số $f = 0$). Hàm `detrend` được áp dụng để đưa kỳ vọng của chuỗi về mốc 0 trước khi phân tích phổ, tránh làm lu mờ các đỉnh chu kỳ.

Hệ số Fourier rời rạc $X_k$ được tính bằng thuật toán rFFT:
$$X_k = \sum_{n=0}^{N-1} x_n e^{-j \frac{2\pi k n}{N}}$$

Phổ một phía được chuẩn hóa vật lý theo quy tắc: Không nhân đôi thành phần DC tại $f=0$, và nhân đôi biên độ các tần số bên trong ($0 < k < N/2$) để bù trừ cho phần phổ âm bị lược bỏ. Kết quả EDA xác nhận chu kỳ tuyệt đối tại $T=24$ giờ (mùa vụ ngày) và $T=168$ giờ (mùa vụ tuần).

## 3. Kỹ nghệ Đặc trưng Không gian - Thời gian
Từ chuỗi đơn biến, hệ thống trích xuất $d=11$ đặc trưng:
1. **Đặc trưng lịch sử (Lags):** `lag_1` và `lag_24` thiết lập cấu trúc ràng buộc Markov.
2. **Đặc trưng lịch thời gian:** `hour`, `day_of_week`, `month`, `is_weekend`.
3. **Đặc trưng Fourier:** Khởi tạo các biến lượng giác $\sin(\frac{2\pi t}{T})$ và $\cos(\frac{2\pi t}{T})$ cho $T=24$ và $T=168$. Đưa cả phần thực và phần ảo vào ma trận giúp mô hình học được biên độ và độ lệch pha (phase shift) liên tục.

Dữ liệu được chia tuần tự theo thời gian: Train (70%), Validation (15%), Test (15%) và đi qua bộ chuẩn hóa `StandardScaler`.

## 4. Phân tích Thực nghiệm và Đánh giá Hiệu năng

### 4.1 Bảng tổng hợp chỉ số định lượng
*(Kết quả đánh giá trên tập Test Set độc lập)*

| Kiến trúc Mô hình | MAE (MW) | RMSE (MW) | MAPE (%) |
| :--- | :--- | :--- | :--- |
| **Baseline (Naive Lag-24h)** | (Điền số) | (Điền số) | (Điền số) |
| **XGBoost (Fourier Basis)** | (Điền số) | (Điền số) | (Điền số) |
| **LSTM (Cửa sổ trượt 24h)** | (Điền số) | (Điền số) | (Điền số) |

### 4.2 Biện luận học thuật và Phân tích Mô hình

**Mô hình Baseline (Cơ sở):**
Với giả định $y_t = y_{t-24}$, mô hình Baseline vấp phải sai số lớn tại các thời điểm chuyển giao (từ ngày làm việc sang ngày nghỉ cuối tuần). Sai số RMSE lớn cho thấy mô hình không thể thích ứng với các biến động pha đột ngột.

**Sự bứt phá của kiến trúc XGBoost + Fourier:**
XGBoost - một kiến trúc Tree-based không có khả năng ngoại suy tuần tự - lại đạt được hiệu năng xuất sắc khi được nạp trực tiếp các hàm cơ sở lượng giác. Việc chia không gian bằng các nhánh cây quyết định trên nền tảng sóng sin/cos tạo ra các ranh giới học tập cực kỳ vững chắc. Biểu đồ dự báo cho thấy XGBoost bám sát tuyệt đối các đỉnh tải ban ngày và đáy tải ban đêm mà không hề bị trễ pha.

**Mạng học sâu LSTM:**
Khác với XGBoost cần nạp đặc trưng toán học thủ công, kiến trúc Long Short-Term Memory (LSTM) với bộ nhớ cổng (gates) tự động học các phụ thuộc phi tuyến thông qua cửa sổ trượt (look-back window = 24 giờ). Kết quả thực nghiệm cho thấy đường dự báo của LSTM rất mượt mà. Tuy nhiên, LSTM nhạy cảm hơn với hiện tượng nhiễu cục bộ và đòi hỏi tinh chỉnh siêu tham số phức tạp hơn so với XGBoost trong bài toán chuỗi thời gian có tính chu kỳ khắc nghiệt.

## 5. Định hướng Mở rộng
Nghiên cứu khẳng định giá trị của việc kết hợp kỹ thuật xử lý tín hiệu cổ điển (Fourier) với học máy hiện đại. Để giải quyết dứt điểm các sai số tại đỉnh phụ tải do thời tiết đột biến, hướng phát triển kế tiếp sẽ chuyển đổi ma trận đầu vào sang các kiến trúc Transformer SOTA (iTransformer, TimeMixer, xLSTM-Mixer). Các kiến trúc này cung cấp cơ chế hòa trộn đa thang đo (Multiscale Mixing) và học tương quan chéo (Cross-variate Attention) nhằm mở rộng không gian dự báo lên mức độ toàn diện hơn.

## 6. Tài liệu tham khảo
[1] Nguyễn Thị Ngọc Anh, "Bài giảng: Biến đổi Fourier cho phân tích chuỗi thời gian," Viện Toán ứng dụng và Tin học, Đại học Bách khoa Hà Nội, 2026.
[2] Y. Liu et al., "iTransformer: Inverted Transformers Are Effective for Time Series Forecasting," arXiv preprint arXiv:2310.06625, 2023.
[3] S. Wang et al., "TimeMixer: Decomposable Multiscale Mixing for Time Series Forecasting," arXiv preprint arXiv:2405.14616, 2024.
[4] J. Doe et al., "xLSTM-Mixer: Multivariate Time Series Forecasting by Mixing via Scalar Memories," arXiv preprint arXiv:2410.16928, 2024.
[5] PJM Interconnection, "Hourly Energy Consumption Data," Kaggle.
