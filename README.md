# Multivariate Time Series Forecasting

## 1. Giới thiệu
- Tên sinh viên: Nguyễn Minh Nhật
- Bài toán: Dự báo chuỗi thời gian nhiều chiều. Đầu vào là ma trận X, đầu ra là y.

## 2. Literature Review
Chi tiết tóm tắt 3 bài báo nghiên cứu được lưu tại thư mục papers/:
1. iTransformer: Inverted Transformers Are Effective for Time Series Forecasting.
2. TimeMixer: Decomposable Multiscale Mixing for Time Series Forecasting.
3. xLSTM-Mixer: Multivariate Time Series Forecasting by Mixing via Scalar Memories.

## 3. Quá trình triển khai
- Dữ liệu và phân tích EDA: Xem tại notebooks/01_data_exploration.ipynb
- Xây dựng đặc trưng (Fourier, Lags): Xem tại notebooks/02_feature_engineering.ipynb
- Huấn luyện:
  - Baseline (Moving Average / Linear Regression)
  - Mô hình học máy mùa vụ (XGBoost)
  - Mô hình Deep Learning nâng cao

## 4. Kết quả Đánh giá
(Sẽ cập nhật bảng MAE, RMSE, MAPE tại đây)

## 5. Hướng dẫn chạy code
Cài đặt thư viện bằng lệnh:
pip install -r requirements.txt
