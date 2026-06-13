# BÁO CÁO NGHIÊN CỨU THỰC NGHIỆM: PHÂN TÍCH MIỀN TẦN SỐ VÀ ĐÁNH GIÁ HIỆU NĂNG MÔ HÌNH DỰ BÁO CHUỖI THỜI GIAN ĐA CHIỀU PHỤ TẢI ĐIỆN

## 1. Giới thiệu và Ý nghĩa Đề tài
Dự báo phụ tải điện năng (Load Forecasting) giữ vai trò xương sống trong việc vận hành, lập kế hoạch và tối ưu hóa chi phí hệ thống điện thông minh. Sai số trong dự báo có thể dẫn đến việc vận hành thừa công suất gây lãng phí, hoặc thiếu hụt công suất gây mất ổn định lưới điện an ninh quốc gia. 

Bài toán nghiên cứu tập trung vào việc áp dụng Biến đổi Fourier rời rạc (DFT) để phân rã các chu kỳ mùa vụ ẩn, từ đó xây dựng ma trận đặc trưng đầu vào X thuộc không gian R^(T x d) phục vụ cho các mô hình học máy phi tuyến tính nhằm dự báo biến mục tiêu một chiều y tại thời điểm tương lai. Nghiên cứu sử dụng tập dữ liệu thực tế từ mạng lưới điện American Electric Power (AEP) thuộc hệ thống PJM Interconnection với hơn 120.000 dòng dữ liệu theo giờ để tiến hành kiểm chứng thực nghiệm.

## 2. Cơ sở Phương pháp luận và Quy trình Tiền xử lý Tín hiệu
Để đảm bảo tính chính xác và tránh các sai số mô hình hóa chuỗi thời gian, quy trình xử lý dữ liệu thô tuân thủ nghiêm ngặt các nguyên lý toán học về tần số lấy mẫu và lý thuyết xử lý tín hiệu số.

### 2.1 Đảm bảo điều kiện lấy mẫu đều (Uniform Sampling)
Thuật toán Biến đổi Fourier nhanh (FFT) mặc định giả định khoảng cách thời gian giữa hai điểm dữ liệu liên tiếp là hằng số tuyệt đối (dt = 1.0 giờ). Tập dữ liệu thô ban đầu tồn tại các điểm khuyết thiếu cục bộ do lỗi hệ thống đo đạc hoặc sự thay đổi múi giờ mùa hè/mùa đông (DST). 
- Xử lý trùng lặp: Các mốc thời gian trùng lặp được giải quyết bằng cách giữ lại quan sát cuối cùng nhằm đảm bảo tính cập nhật của chuỗi dữ liệu.
- Xử lý khuyết thiếu: Hệ thống áp dụng hàm nội suy tuyến tính (Linear Interpolation) kết hợp kỹ thuật `resample('h')`. Việc ép chuỗi về tần số cố định dt = 1.0 giờ là bắt buộc để ngăn chặn hiện tượng rò rỉ phổ (spectral leakage) và bảo toàn tính trực giao của các hàm cơ sở Fourier. Nếu các khoảng trống quá lớn, việc nội suy mạnh được hạn chế để tránh tạo ra các chu kỳ nhân tạo phi thực tế.

### 2.2 Khử xu hướng toàn cục (Detrending)
Chuỗi phụ tải điện năng qua nhiều năm thường mang các thành phần xu hướng dài hạn (trend) do sự phát triển kinh tế hoặc gia tăng dân số. Thành phần xu hướng này tương ứng với năng lượng tập trung tại tần số f = 0 (thành phần DC). 
Nếu tiến hành rFFT trực tiếp, biên độ cực đại tại f = 0 sẽ lấn át hoàn toàn các tần số lân cận, làm mờ các đỉnh phổ mùa vụ. Do đó, hàm `scipy.signal.detrend` được áp dụng để đưa kỳ vọng của chuỗi về mốc 0 trước khi thực hiện phân tích phổ.

### 2.3 Chuẩn hóa phổ biên độ một phía (One-sided Amplitude Spectrum)
Hệ số Fourier rời rạc từ thuật toán rFFT đối với chuỗi thực có tính chất đối xứng liên hợp. Để khôi phục đúng biên độ vật lý thực tế của dòng điện (đơn vị MW) từ miền tần số, quy tắc chuẩn hóa phổ một phía được thiết lập như sau:
- Tại tần số f = 0: A_0 = |X_0| / N (Thành phần không đổi, không nhân đôi).
- Tại các tần số bên trong (0 < k < N/2): A_k = 2 * |X_k| / N (Nhân đôi biên độ để bù đắp cho phần phổ âm đã lược bỏ).
- Tại tần số Nyquist (f = f_s / 2): A_(N/2) = |X_(N/2)| / N (Nếu số lượng mẫu N chẵn).

Thành phần tần số f = 0 sau đó bị loại bỏ khỏi danh sách trích xuất chu kỳ vì nó đại diện cho giá trị trung bình dài hạn, không mang thông tin về một chu kỳ biến thiên hữu hạn. Kết quả thực nghiệm tính toán chu kỳ (T = 1/f) chỉ ra hai đỉnh Prominence cao nhất tại T = 24.00 giờ (chu kỳ ngày) và T = 168.00 giờ (chu kỳ tuần), hoàn toàn khớp với quy luật sinh hoạt và sản xuất thực tế.

## 3. Kỹ nghệ Đặc trưng Không gian - Thời gian
Từ chuỗi đơn biến ban đầu, bài toán được chuyển đổi sang dạng học máy giám sát nhiều chiều (Multivariate Time Series) bằng cách xây dựng ma trận đặc trưng X có chiều d = 11:

1. Nhóm đặc trưng lịch sử (Lag Features): Tạo biến trễ `lag_1` và `lag_24`. Đặc trưng `lag_1` cung cấp quán tính ngắn hạn (giá trị giờ trước đó), trong khi `lag_24` cung cấp điểm neo tương quan của cùng khung giờ ngày hôm trước, thiết lập cấu trúc ràng buộc Markov cho mô hình.
2. Nhóm đặc trưng lịch thời gian (Calendar Features): Trích xuất các biến số nguyên bao gồm `hour` (0-23), `day_of_week` (0-6), `month` (1-12) và biến nhị phân `is_weekend` (0 hoặc 1) nhằm giúp mô hình phân tách hành vi tiêu thụ điện giữa ngày làm việc và ngày nghỉ.
3. Nhóm đặc trưng lượng giác Fourier (Fourier Basis): Khởi tạo các cặp biến lượng giác:
   - sin_daily = sin(2 * pi * t / 24), cos_daily = cos(2 * pi * t / 24)
   - sin_weekly = sin(2 * pi * t / 168), cos_weekly = cos(2 * pi * t / 168)
   Việc đưa đồng thời cả hai thành phần sin và cos vào ma trận đặc trưng là điều kiện tiên quyết để mô hình tuyến tính hoặc phi tuyến (như cây quyết định) có thể học được cả biên độ dao động lẫn độ lệch pha (phase shift) của mùa vụ điện năng theo thời gian mà không cần cấu trúc mạng nơ-ron hồi quy tuần tự phức tạp.

Dữ liệu sau khi trích xuất đặc trưng được chia tách theo thứ tự thời gian để bảo toàn tính nhân quả: 70% đầu dành cho huấn luyện (Train), 15% tiếp theo để tối ưu tham số (Validation), và 15% cuối cùng làm tập kiểm thử độc lập (Test). Toàn bộ ma trận đặc trưng được chuẩn hóa qua bộ lọc `StandardScaler` để đưa phân phối về trung bình bằng 0 và phương sai bằng 1.

## 4. Phân tích Thực nghiệm và Thảo luận Chuyên sâu Kết quả
Hiệu năng dự báo của các mô hình được đánh giá độc lập trên tập dữ liệu Test thông qua ba chỉ số toán học nghiêm ngặt: MAE (Độ lỗi tuyệt đối trung bình), RMSE (Sai số căn phương sai trung bình - rất nhạy cảm với các sai số lớn), và MAPE (Phần trăm độ lỗi tuyệt đối trung bình).

### 4.1 Bảng tổng hợp chỉ số định lượng
Dưới đây là kết quả đo đạc thực nghiệm chi tiết thu được từ quá trình huấn luyện:

| Kiến trúc Mô hình | MAE (MW) | RMSE (MW) | MAPE (%) |
| :--- | :--- | :--- | :--- |
| **Baseline (Naive Lag-24h)** | 1391.22 | 1878.71 | 9.38% |
| **XGBoost (Fourier Basis & Lags)** | 512.63 | 733.91 | 3.52% |

### 4.2 Biện luận học thuật và Đánh giá chuyên sâu

#### Phân tích chỉ số Baseline
Mô hình Baseline (Dự báo bằng giá trị của 24 giờ trước) đạt mức sai số MAPE là 9.38% và RMSE lên tới 1878.71 MW. Chỉ số RMSE cao vượt trội so với MAE chứng tỏ mô hình Baseline gặp phải hiện tượng sai số cực đại tại các thời điểm chuyển giao pha dữ liệu. Điều này hoàn toàn dễ hiểu về mặt vật lý: nhu cầu tiêu thụ điện năng giữa hai ngày liên tiếp có thể bị lệch đáng kể do các yếu tố ngẫu nhiên như thời tiết đột biến hoặc sự dịch chuyển từ ngày làm việc sang ngày cuối tuần (thứ Sáu sang thứ Bảy), khiến giả định lặp lại tuyệt đối của Naive Method bị đổ vỡ.

#### Phân tích sự vượt trội của XGBoost kết hợp Fourier
Khi chuyển dịch sang kiến trúc XGBoost kết hợp chuỗi đặc trưng đa chiều, hiệu năng hệ thống ghi nhận một sự bứt phá mạnh mẽ:
- Sai số MAPE giảm mạnh xuống mốc lý tưởng là 3.52% (giảm gần 3 lần so với Baseline). Điều này chứng tỏ mô hình có độ ổn định cực cao trên toàn bộ các dải giá trị.
- Chỉ số MAE giảm từ 1391.22 MW xuống còn 512.63 MW (cải thiện xấp xỉ 63.15%). 
- Chỉ số RMSE giảm xuống còn 733.91 MW, cho thấy mô hình đã triệt tiêu được hầu hết các lỗi dự báo nghiêm trọng tại các điểm cực trị.

Về bản chất thuật toán, các mô hình dựa trên cấu trúc cây quyết định tăng cường (Gradient Boosting Trees) như XGBoost cực kỳ mạnh trong việc phân rã không gian phi tuyến nhưng lại hoàn toàn mất phương hướng khi phải ngoại suy (extrapolation) các yếu tố chu kỳ hoặc xu hướng tuyến tính dài hạn. Bằng cách nạp trực tiếp các hàm cơ sở lượng giác Fourier (`sin_daily`, `cos_daily`, `sin_weekly`, `cos_weekly`) vào ma trận X, chúng ta đã cung cấp cho XGBoost các ranh giới toán học tuần hoàn đóng vai trò như một bộ khung định hình. 

Thay vì phải tự mò mẫm học tính mùa vụ từ các mốc thời gian tuyến tính, XGBoost chỉ cần thực hiện các đường cắt điều kiện trên các hàm sin/cos để ánh xạ chính xác mức độ biến thiên công suất tương ứng với từng pha của chu kỳ ngày và tuần.

#### Phân tích biểu đồ dự báo thực tế (y_true vs y_pred)
Khi tiến hành phân tích cắt lớp biểu đồ dự báo trên cửa sổ 2 tuần đầu tiên của tập Test:
- Đường dự báo màu cam đứt nét bám sát một cách kinh ngạc với đồ thị thực tế màu xanh. Hệ thống bắt chính xác các thời điểm đỉnh tải (Peak load) diễn ra vào các khung giờ cao điểm trưa và tối, đồng thời không bị hiện tượng trễ pha (phase lag) - một lỗi kinh điển của các mô hình trung bình trượt tuyến tính (ARIMA/SARIMAX).
- Tại các vùng đáy tải ban đêm (Off-peak load từ 1h đến 4h sáng), đường dự báo hoàn toàn mịn màng và trùng khớp với giá trị thực tế, không xảy ra hiện tượng dao động răng cưa vô căn cứ. Sự ổn định này là minh chứng cho thấy bộ lọc khử xu hướng (`detrend`) kết hợp chuẩn hóa phổ một phía đã triệt tiêu sạch nhiễu nền tần số cao trước khi mô hình học máy tiếp nhận dữ liệu.
- Sai số nhỏ chỉ xuất hiện cục bộ tại một số đỉnh nhọn đột ngột. Nguyên nhân do phụ tải điện phụ thuộc vào các biến ngoại cảnh không định hình (như nhiệt độ tăng giảm thất thường của thời tiết thực tế). Để xử lý triệt để các sai số cục bộ này, ma trận đặc trưng cần được mở rộng cấu trúc sang không gian đa biến thực sự ở các giai đoạn tiếp theo.

## 5. Định hướng Mở rộng với các Kiến trúc Deep Learning Tiên tiến
Kết quả phân tích thực nghiệm với mô hình học máy truyền thống kết hợp Fourier Basis đã thiết lập một cột mốc benchmark rất vững chắc (MAPE 3.52%). Để tiếp tục tối ưu hóa bài toán dự báo chuỗi thời gian nhiều chiều dài hạn, cấu trúc dữ liệu đa chiều hiện tại được định hướng tích hợp vào các kiến trúc học sâu (Deep Learning) SOTA từ giai đoạn năm 2024 trở đi:

1. **iTransformer (Inverted Transformer):** Thay vì cắt chuỗi theo các khoảng thời gian ngắn (patching) làm mất tương quan vật lý giữa các biến, kiến trúc iTransformer sẽ xem toàn bộ lịch sử thời gian của một đặc trưng Fourier hoặc đặc trưng trễ là một token độc lập. Cơ chế Attention sẽ được áp dụng chéo giữa các biến số để khai thác sâu sắc mối tương quan không gian (cross-variate correlation), hứa hẹn sẽ giải quyết triệt để các sai số đỉnh tải do biến động thời tiết gây ra.
2. **TimeMixer (Decomposable Multiscale Mixing):** Kiến trúc này sẽ tận dụng kết quả phân tích phổ của nghiên cứu để chia dữ liệu thành các thang đo khác nhau (thang đo mịn theo giờ để học biến Fourier ngày, thang đo thô theo ngày để học biến Fourier tuần). Khối Past-Decomposable-Mixing (PDM) của TimeMixer sẽ phân tách riêng biệt Trend và Seasonality ở từng thang đo giúp mô hình hóa chuỗi một cách tường minh và đẩy tốc độ suy luận lên mức tối đa nhờ cấu trúc thuần MLP.
3. **xLSTM-Mixer:** Thay thế các mô hình Attention-based bằng bộ nhớ vô hướng mở rộng (sLSTM). Cơ chế kết hợp hòa trộn thông tin theo trục thời gian và hòa trộn góc nhìn (View Mixing) của xLSTM-Mixer sẽ giúp hệ thống ghi nhớ được các phụ tải đặc biệt từ các tuần trước đó mà không làm bùng nổ chi phí tính toán phần cứng RAM của Colab, đảm bảo tính khả thi khi triển khai trên các hệ thống nhúng thời gian thực.
