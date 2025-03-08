+++
date = '2021-04-02T10:43:32+07:00'
draft = false
title = 'CAP Theorem'
summary = 'Hiểu đúng về sự đánh đổi trong hệ thống phân tán. Định lý CAP (CAP Theorem) là một nguyên tắc quan trọng trong lĩnh vực hệ thống phân tán, được nhà khoa học máy tính Eric Brewer đề xuất vào năm 2000'
tags = ['distributed system', 'database']
categories = []
+++

Hiểu đúng về sự đánh đổi trong hệ thống phân tán.

## 1. Định lý CAP là gì?

Định lý CAP (CAP Theorem) là một nguyên tắc quan trọng trong lĩnh vực hệ thống phân tán, được nhà khoa học máy tính Eric Brewer đề xuất vào năm 2000. Định lý này khẳng định rằng trong một hệ thống phân tán, ta không thể đồng thời đạt được cả ba yếu tố sau đây:

- **Consistency (Nhất quán):** Mọi node trong hệ thống luôn có cùng một trạng thái dữ liệu tại một thời điểm bất kỳ.
- **Availability (Sẵn sàng):** Hệ thống luôn phản hồi mọi yêu cầu đọc/ghi ngay cả khi một số node bị lỗi.
- **Partition Tolerance (Khả năng chịu phân vùng):** Hệ thống tiếp tục hoạt động ngay cả khi có sự cố mạng khiến một số node không thể liên lạc với nhau.

## 2. Ý nghĩa của định lý CAP

Vì không thể đạt được cả ba yếu tố trên cùng lúc, các hệ thống phân tán phải lựa chọn hai trong ba thuộc tính này. Điều này dẫn đến ba mô hình phổ biến:

- **CP (Consistency + Partition Tolerance):** Hệ thống đảm bảo tính nhất quán và chịu được phân vùng, nhưng có thể không sẵn sàng khi xảy ra lỗi mạng.
- **AP (Availability + Partition Tolerance):** Hệ thống đảm bảo luôn phản hồi yêu cầu và chịu được phân vùng, nhưng có thể trả về dữ liệu cũ hoặc không nhất quán.
- **CA (Consistency + Availability):** Hệ thống đảm bảo cả tính nhất quán và sẵn sàng, nhưng không thể chịu được lỗi phân vùng. Tuy nhiên, trong môi trường thực tế, không có hệ thống phân tán nào hoàn toàn thuộc mô hình CA vì phân vùng mạng là điều không thể tránh khỏi.

{{<figure src="./image1.png" width="500px" class="center">}}

## 3. Ứng dụng thực tế của định lý CAP

Khi thiết kế hệ thống phân tán, ta phải đưa ra quyết định dựa trên ưu tiên của ứng dụng:

- Nếu yêu cầu dữ liệu luôn phải chính xác (ví dụ: hệ thống ngân hàng), ta cần ưu tiên **CP**.
- Nếu hệ thống cần đảm bảo phản hồi nhanh (ví dụ: mạng xã hội, hệ thống gợi ý), ta có thể chọn **AP**.
- Nếu hệ thống chạy trên một trung tâm dữ liệu duy nhất và có độ tin cậy cao, mô hình **CA** có thể là một lựa chọn hợp lý.

## 4. Ví dụ về các hệ quản trị cơ sở dữ liệu phổ biến

Dưới đây là một số ví dụ về cách các hệ quản trị cơ sở dữ liệu hiện đại áp dụng định lý CAP:

- **MongoDB (AP hoặc CP, tùy vào cấu hình):** Theo mặc định, MongoDB ưu tiên tính sẵn sàng và khả năng chịu phân vùng (AP). Tuy nhiên, nếu bật chế độ write concern và read concern cao hơn, MongoDB có thể hoạt động theo mô hình CP.
- **Cassandra (AP hoặc CP, tùy vào cấu hình):** Cassandra thường được xem là một hệ thống AP, nhưng có thể được cấu hình để hoạt động gần với mô hình CP bằng cách điều chỉnh mức độ nhất quán (`QUORUM`, `ALL`). Nếu yêu cầu xác nhận từ đa số các node trong cả đọc và ghi, Cassandra sẽ ưu tiên tính nhất quán hơn nhưng có thể ảnh hưởng đến tính sẵn sàng.
- **MySQL (CA khi chạy độc lập, CP với Galera Cluster):** MySQL khi chạy đơn lẻ thì đảm bảo CA, nhưng khi sử dụng Galera Cluster, nó có thể hoạt động theo mô hình CP bằng cách đồng bộ dữ liệu giữa các node.
- **DynamoDB (AP):** Amazon DynamoDB ưu tiên tính sẵn sàng và khả năng chịu phân vùng, chấp nhận một mức độ nhất quán thấp hơn.