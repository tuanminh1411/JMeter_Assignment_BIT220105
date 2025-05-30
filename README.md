# Báo cáo Kiểm thử Hiệu năng bằng JMeter

## 1. Thông tin sinh viên  
- **Họ và tên:** Nguyễn Tuấn Minh  
- **MSSV:** BIT220105  
- **Website kiểm thử:** https://www.example.com  


---

## 2. Mô tả kịch bản kiểm thử  

| Kịch bản     | Threads                   | Ramp-up (s) / Duration (s) | Loop Count | Hành vi                                              |
|--------------|---------------------------|----------------------------|------------|------------------------------------------------------|
| **Cơ bản**   | 15                        | —                          | 5          | GET `/?student_id=BIT220105`                        |
| **Tải nặng** | 55                        | Ramp-up 30                 | 1          | GET `/` và GET `/about`                              |
| **Tùy chỉnh**| 30                        | Duration 60                | Forever    | GET `/about` và GET `/contact`                       |

> **Lưu ý:**  
> - Biến `${StudentID}` đã được thiết lập là `BIT220105`.  
> - Thread Group “Tùy chỉnh” chạy liên tục trong 60 giây, không giới hạn số vòng lặp.  

---

## 3. Kết quả thu thập từ Summary Report

| Kịch bản     | Response Time trung bình (ms) | Throughput (req/s) | Error Rate (%) |
|--------------|-------------------------------:|-------------------:|---------------:|
| Cơ bản       |                           41.0 |             71.84  |           0.00 |
| Tải nặng     |                           55.0 |              1.87  |           0.00 |
| Tùy chỉnh    |                           48.0 |            612.95  |         100.00 |

---

## 4. Phân tích và Nhận xét  

### 4.1. Kịch bản 1 – Cơ bản  
- **Response Time (41 ms):** Rất nhanh, cho thấy server đáp ứng nhẹ nhàng với 15 người dùng đồng thời.  
- **Throughput (71.84 req/s):** Cao so với quy mô Threads nhỏ, minh chứng khả năng xử lý request hiệu quả.  
- **Error Rate (0%):** Không phát sinh lỗi, mọi request đều thành công.

### 4.2. Kịch bản 2 – Tải nặng  
- **Response Time (55 ms):** Tăng nhẹ so với kịch bản cơ bản nhưng vẫn nằm trong ngưỡng chấp nhận (< 100 ms).  
- **Throughput (1.87 req/s):** Thấp do chỉ chạy 1 vòng lặp; khuyến nghị tăng Loop Count để đánh giá ổn định hơn.  
- **Error Rate (0%):** Server xử lý an toàn, không lỗi dù ramp-up 30s với 55 Threads.

### 4.3. Kịch bản 3 – Tùy chỉnh  
- **Response Time (48 ms):** Nhanh tương đương các kịch bản khác.  
- **Throughput (612.95 req/s):** Bất thường cao do thiết lập “Forever” trong 60 s và chỉ một sampler; JMeter gửi tối đa request có thể.  
- **Error Rate (100%):** Tất cả request đều lỗi. Nguyên nhân có thể:  
  1. **Sai URL path** (`/about` hoặc `/contact` không tồn tại hoặc trả về HTTP 4xx/5xx).  
  2. **Phương thức GET không phù hợp** nếu endpoint yêu cầu POST.  
  3. **Thiếu HTTP Header** hoặc dữ liệu bắt buộc.

---

## 5. Kiến nghị & Sửa đổi  

1. **Kiểm tra lại Endpoint Tùy chỉnh**  
   - Mở trình duyệt hoặc Postman, truy cập thủ công `https://www.example.com/about` và `/contact` để xác nhận HTTP 200.  
   - Nếu cần dùng POST, thêm **HTTP Header Manager** (Content-Type) và **Body Data** tương ứng.

2. **Hiệu chỉnh Loop Count của Kịch bản 2**  
   - Đặt Loop Count ≥ 5 để thu được Throughput ổn định và có thể so sánh với kịch bản 1.

3. **Kiểm soát tốc độ gửi request**  
   - Đối với kịch bản 3, thêm **Timer** (Constant Throughput Timer hoặc Uniform Random Timer) để điều tiết số request nhằm phân tích khả năng chịu tải thực tế.

4. **Bổ sung phân tích lỗi**  
   - Dùng **View Results Tree** để lọc ra status code lỗi và log message chi tiết.  
   - Ghi chú cụ thể số lượng request lỗi theo từng mã trạng thái.

---

## 6. Kết luận  

- Website `https://www.example.com` đáp ứng tốt với tải nhỏ và trung bình, không phát sinh lỗi ở kịch bản 1 và 2.  
- Thiết lập kịch bản tùy chỉnh cần được điều chỉnh để thu thập số liệu chính xác; hiện tại lỗi 100% cho thấy endpoint hoặc cấu hình không phù hợp.  
- Với những sửa đổi đề xuất, việc kiểm thử lại sẽ cung cấp cái nhìn đầy đủ hơn về khả năng chịu tải dài hạn và độ ổn định của hệ thống.

