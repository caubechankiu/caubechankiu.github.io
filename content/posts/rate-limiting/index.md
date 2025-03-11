+++
date = '2021-12-17T23:40:15+07:00'
draft = false
title = 'Rate Limiting'
summary = 'Rate Limiting (giới hạn tốc độ truy cập) là một kỹ thuật kiểm soát số lượng request mà một client có thể gửi đến một hệ thống trong một khoảng thời gian nhất định.'
tags = ['rate limiting', 'microservice', 'resiliency']
categories = []
+++

## 1. What is rate limiting?

Rate Limiting (giới hạn tốc độ truy cập) là một kỹ thuật kiểm soát số lượng request mà một client có thể gửi đến một hệ thống trong một khoảng thời gian nhất định. Mục tiêu chính của rate limiting là:
- **Ngăn chặn tấn công DDoS**, bảo vệ và cải thiện tính khả dụng của service bằng cách hạn chế số lượng request từ một IP hoặc user nhất định
- **Quản lý chính sách và hạn ngạch**, đảm bảo công bằng giữa các user khi truy cập tài nguyên.

## 2. Rate limiting algorithms
Có nhiều thuật toán khác nhau để triển khai rate limiting, mỗi thuật toán đều có ưu và nhược điểm riêng. Chúng ta hãy cùng xem xét từng thuật toán để có thể chọn tùy chọn thiết kế rate limiting lý tưởng cho service của mình.
### 2.1. Leaky Bucket
Leaky Bucket hoạt động dựa trên hình ảnh một chiếc xô chứa nước có lỗ rò:
- **Dữ liệu (requests) giống như nước** được đổ vào xô.
- **Xô có dung lượng giới hạn**, nếu nước (request) đổ vào quá nhanh, xô sẽ tràn (từ chối).
- **Nước rò rỉ với tốc độ cố định**, mô phỏng việc xử lý request theo một tốc độ ổn định.
- Khi xô đầy, các request mới sẽ bị từ chối cho đến khi có không gian trong xô.

{{<figure src="leaky-bucket.avif" width="500px" class="center">}}

Leaky Bucket có thể được triển khai bằng cách sử dụng **hàng đợi (queue) có kích thước cố định**:
- Khi một request đến, nó được đưa vào queue nếu còn chỗ.
- Một tiến trình xử lý sẽ lấy request từ queue ra với tốc độ cố định.
- Nếu queue đầy, request bị từ chối ngay lập tức.

Thuật toán này có ưu điểm là làm mượt các đợt request tăng đột biến và xử lý chúng với tốc độ trung bình xấp xỉ. Nó cũng dễ triển khai trên một máy chủ đơn hoặc bộ cân bằng tải và tiết kiệm bộ nhớ cho từng người dùng nhờ vào kích thước queue giới hạn.

Tuy nhiên, một lượng lớn lưu lượng truy cập có thể lấp đầy queue với các request cũ, khiến các request mới hơn bị chặn và không được xử lý.

### 2.2. Fixed Window
Fixed Window là một thuật toán rate limiting đơn giản, trong đó số lượng request được cho phép trong một khoảng thời gian cố định (window) nhất định. Khi cửa sổ bắt đầu, bộ đếm được đặt lại và bắt đầu đếm số lượng request cho đến khi hết thời gian của cửa sổ.

{{<figure src="fixed-window.avif" width="300px" class="center">}}

- Chia thời gian thành các cửa sổ cố định có độ dài nhất định (ví dụ: 1 phút).
- Mỗi cửa sổ có một bộ đếm để theo dõi số lượng request từ một người dùng hoặc IP.
- Nếu số request trong cửa sổ vượt quá giới hạn cho phép, các request tiếp theo sẽ bị từ chối (thường trả về HTTP 429 Too Many Requests).
- Khi cửa sổ kết thúc, bộ đếm được đặt lại về 0 và bắt đầu một cửa sổ mới.

Ví dụ: Giả sử một API giới hạn 100 request mỗi phút theo thuật toán Fixed Window. Từ 12:00:00 → 12:00:59, bạn có thể gửi tối đa 100 request. Đến 12:01:00, cửa sổ mới bắt đầu và bộ đếm được đặt lại về 0.

Ưu điểm của thuật toán này là nó đảm bảo các request gần đây hơn được xử lý mà không bị chặn bởi các request cũ, và nó cũng dễ triển khai, ít tốn tài nguyên, phù hợp với các hệ thống cần kiểm soát tải đơn giản.

Tuy nhiên, nếu có một lượng lớn request xảy ra gần ranh giới của một cửa sổ, hệ thống có thể xử lý gấp đôi số lượng request cho phép, vì nó sẽ chấp nhận request cho cả cửa sổ hiện tại và cửa sổ tiếp theo trong một khoảng thời gian ngắn.

Ví dụ, nếu một người gửi 100 request vào 12:00:59 và ngay sau đó gửi tiếp 100 request vào 12:01:00, hệ thống có thể nhận 200 request trong vòng 1 giây. Điều này có thể gây áp lực lên server.

### 2.3. Sliding Log
Sliding Log là một thuật toán rate limiting dựa trên việc lưu trữ dấu thời gian (timestamp) của từng request vào một danh sách (log). Khi một request mới đến, thuật toán sẽ kiểm tra log để xem có bao nhiêu request đã được thực hiện trong khoảng thời gian giới hạn (window). Nếu số lượng request vượt quá ngưỡng cho phép, request mới sẽ bị từ chối.

{{<figure src="sliding-log.avif" width="300px" class="center">}}

- **Lưu trữ log**: Mỗi lần có request từ người dùng, hệ thống ghi lại timestamp của request đó vào một danh sách.
- **Xóa các log cũ**: Khi một request mới đến, hệ thống loại bỏ tất cả các timestamp đã quá giới hạn thời gian quy định.
- **Kiểm tra số lượng request**: Đếm số lượng timestamp còn lại trong log.
- **So sánh với giới hạn**: Nếu số lượng này nhỏ hơn mức cho phép, request được chấp nhận. Nếu không, request bị từ chối.

Ví dụ: giả sử hệ thống giới hạn 5 requests trong vòng 10 giây, log có dạng như sau:
00:01	✅ Cho phép  
00:03	✅ Cho phép  
00:05	✅ Cho phép  
00:07	✅ Cho phép  
00:09	✅ Cho phép  
00:11	❌ Từ chối (vì 5 requests đã thực hiện trong 10 giây trước đó)  
00:12	✅ Cho phép (00:01 bị loại khỏi log)

Ưu điểm của thuật toán này là nó cung cấp mức độ chính xác cao vì kiểm tra từng request thay vì chia thành các khoảng thời gian cố định như Fixed Window và tránh được vấn đề "bùng nổ request" ở ranh giới cửa sổ thời gian (có trong Fixed Window).

Tuy nhiên, việc lưu trữ log cho từng request có thể tốn kém. Việc tính toán cũng tốn kém vì mỗi request yêu cầu phải tính tổng các request trước đó của người dùng. Do đó, nó không mở rộng tốt để xử lý các đợt bùng nổ lưu lượng truy cập lớn hoặc các cuộc tấn công từ chối dịch vụ.

### 2.4. Sliding Window
Sliding Window là một phương pháp lai kết hợp chi phí xử lý thấp của Fixed Window và các điều kiện biên được cải thiện của Sliding Log.

{{<figure src="sliding-window.avif" width="300px" class="center">}}

Ví dụ: giả sử có giới hạn 100 requests/1 phút. Nếu request đến vào giây thứ 45, hệ thống sẽ tính toán số request của:
- 45 giây đầu của cửa sổ hiện tại (ví dụ: 75 requests).  
- 15 giây cuối của cửa sổ trước (ví dụ: 30 requests, nhưng chỉ lấy tỷ lệ: 30 * (15/60) = 7.5).  
- Tổng số request ước lượng: 75 + 7.5 = 82.5 requests, vẫn dưới giới hạn nên request được chấp nhận.

Thuật toán này giúp giảm tình trạng spike (bùng nổ requests ngay sau khi cửa sổ mới bắt đầu) và ít tốn bộ nhớ hơn Sliding Log. Nó cũng dễ dàng triển khai trên Redis (bằng cách dùng INCR và EXPIRE).

Tuy nhiên, việc tính toán ước lượng như vậy mang đến độ chính xác không được 100% như Sliding Log.

Thuật toán này hữu ích khi cần hiệu suất tốt hơn và không quá khắt khe về độ chính xác. Phù hợp với hệ thống có tải lớn.

## 3. Types of Thresholds for Rate-limiting

Một quyết định quan trọng là xác định vị trí đặt rate limiting: thường theo từng user, theo từng endpoint, hoặc kết hợp cả hai.

Giới hạn theo user là cách tiếp cận phổ biến. Mỗi user được phép thực hiện một số lượng request nhất định trong một khoảng thời gian, ví dụ 1000 request mỗi giờ. Nếu vượt quá giới hạn này, request của họ sẽ bị từ chối cho đến khi cửa sổ thời gian được đặt lại. Với cách tiếp cận này, server phải đảm bảo có đủ khả năng (hoặc có thể mở rộng linh hoạt) để xử lý số lượng request tối đa được phép cho mỗi user. Nếu số lượng user mới tăng nhanh, việc duy trì và điều chỉnh các giới hạn có thể gây tốn kém tài nguyên.

Một cách tiếp cận khác là sử dụng giới hạn theo từng endpoint. Giới hạn này được áp dụng cho tất cả user và có thể được đặt dựa trên khả năng thực tế của server thông qua các bài kiểm tra hiệu suất (benchmark). So với giới hạn theo từng user, cách này dễ cấu hình hơn và đáng tin cậy hơn trong việc ngăn server bị quá tải. Tuy nhiên, một user hoạt động quá mức có thể làm ảnh hưởng đến các user khác.

Chiến lược giới hạn có thể kết hợp nhiều cách khác nhau.

- **Theo user và từng endpoint**: Ví dụ, user A truy cập vào endpoint sendEmail. Không nhất thiết phải chi tiết đến mức này, nhưng có thể hữu ích đối với các endpoint quan trọng.

- **Theo user**: Ngoài giới hạn trên từng endpoint, user A có thể có một giới hạn tổng thể là 1000 request/giờ cho tất cả API.

- **Theo endpoint**: Đây là cơ chế bảo vệ chung của server để đảm bảo không có endpoint nào bị quá tải. Nếu các giới hạn theo user được thiết lập hợp lý, giới hạn này thường không bị chạm đến.

- **Theo toàn bộ server**: Cuối cùng, một giới hạn tổng thể về số lượng request mà server có thể xử lý. Điều này quan trọng vì ngay cả khi các endpoint hoạt động trong giới hạn riêng của chúng, chúng vẫn không hoàn toàn độc lập: server vẫn có giới hạn tài nguyên để xử lý yêu cầu, mở/đóng kết nối mạng, v.v.



