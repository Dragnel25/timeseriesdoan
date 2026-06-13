# 1. Mô tả Bộ dữ liệu và Tiền xử lý

## 1.1 Thống kê chi tiết về dữ liệu gốc
Dự án sử dụng tập dữ liệu Phụ tải điện theo giờ (Hourly Energy Consumption) thuộc mạng lưới điện American Electric Power (AEP), một trong những mạng lưới điện lớn nhất Hoa Kỳ thuộc hệ thống PJM Interconnection.
- Tần số lấy mẫu: Theo giờ (Hourly resolution).
- Dung lượng mẫu: 121,273 quan sát liên tục từ năm 2004 đến năm 2018.
- Biến mục tiêu (Target y): `AEP_MW` biểu diễn công suất tiêu thụ điện năng tại thời điểm t (đơn vị: Megawatt).

## 1.2 Xây dựng bài toán Chuỗi thời gian nhiều chiều (Multivariate)
Để đáp ứng cấu trúc đầu vào ma trận X thuộc không gian R^(T x d) và đầu ra y thuộc R^T, nhóm đã thực hiện kỹ nghệ trích xuất đặc trưng (Feature Engineering) biến chuỗi đơn biến gốc thành cấu trúc đa chiều với d = 11 đặc trưng cốt lõi:
1. Đặc trưng thời gian (4 biến): hour (giờ trong ngày), day_of_week (ngày trong tuần), month (tháng trong năm), is_weekend (biến phân loại ngày cuối tuần).
2. Đặc trưng mùa vụ Fourier (4 biến): Cặp biến lượng giác sin_daily, cos_daily (chu kỳ ngày 24 giờ) và sin_weekly, cos_weekly (chu kỳ tuần 168 giờ). Các đặc trưng này chuyển đổi chuỗi từ miền thời gian sang miền tần số, biểu diễn trọn vẹn biên độ và độ lệch pha của phụ tải điện.
3. Đặc trưng trễ lịch sử (2 biến): lag_1 (quán tính tiêu thụ của giờ trước đó) và lag_24 (giá trị phụ tải cùng giờ ngày hôm trước).

## 1.3 Quy trình tiền xử lý dữ liệu chuẩn hóa
Bám sát tài liệu hướng dẫn phân tích miền tần số:
- Kiểm tra tính đồng đều: Áp dụng hàm resample('h') kết hợp nội suy tuyến tính (Linear Interpolation) để xử lý các khoảng trống thiếu dữ liệu cục bộ, đảm bảo khoảng cách lấy mẫu delta_t bằng 1.0 giờ tuyệt đối nhằm loại bỏ hiện tượng lệch phổ tần số (spectral leakage).
- Khử xu hướng toàn cục: Sử dụng hàm detrend trước khi thực hiện biến đổi Real FFT nhằm loại bỏ thành phần xu hướng tăng/giảm dài hạn (thành phần DC tại tần số f = 0), giúp các đỉnh phổ chu kỳ mùa vụ không bị che khuất.
- Chia tập dữ liệu tuần tự: Train (70% dữ liệu đầu), Validation (15% dữ liệu tiếp theo), Test (15% dữ liệu cuối cùng cho đánh giá độc lập).
