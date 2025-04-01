+++
date = '2023-12-21T09:47:31+07:00'
draft = false
title = 'Transaction Outbox Pattern'
summary = 'Transaction Outbox Pattern: Giải pháp đảm bảo tính nhất quán trong hệ thống phân tán'
tags = ['microservice', 'consistency', 'distributed system']
categories = []
+++

## Transaction Outbox Pattern: Giải pháp đảm bảo tính nhất quán trong hệ thống phân tán

Trong thế giới phát triển phần mềm hiện đại, đặc biệt là với các hệ thống phân tán (distributed systems), việc đảm bảo tính nhất quán (consistency) giữa database và các service khác (như message broker, hệ thống bên ngoài) là một thách thức lớn. Một trong những giải pháp phổ biến để giải quyết vấn đề này là **Transaction Outbox Pattern**. Chúng ta sẽ cùng tìm hiểu chi tiết về mẫu thiết kế này: nó là gì, tại sao cần nó, cách nó hoạt động và cách triển khai trong thực tế.

### Transaction Outbox Pattern là gì?

Transaction Outbox Pattern là một mẫu thiết kế được sử dụng để đảm bảo rằng các thay đổi trong database và việc gửi message (message) tới các hệ thống khác (thường qua message queue như Kafka, RabbitMQ) được thực hiện một cách nhất quán và đáng tin cậy. Ý tưởng chính là thay vì gửi message trực tiếp tới message broker trong cùng một giao dịch (transaction) với thay đổi database, chúng ta lưu message đó vào một bảng trung gian (gọi là "outbox") trong cùng database, sau đó một tiến trình riêng biệt sẽ đọc và gửi message này đi.

Mẫu này thường được sử dụng trong các hệ thống microservices, nơi mà việc phối hợp giữa các service khác nhau đòi hỏi tính nhất quán cao, nhưng không thể dựa vào giao dịch phân tán (distributed transaction) do độ phức tạp và chi phí hiệu suất.

### Tại sao cần Transaction Outbox Pattern?

Hãy tưởng tượng một kịch bản phổ biến trong hệ thống microservices:

1. Một service nhận được yêu cầu tạo đơn hàng mới.
2. Dịch vụ này cần:
   - Lưu thông tin đơn hàng vào database.
   - Gửi một message tới message queue (ví dụ: "Đơn hàng đã được tạo") để thông báo cho các service khác (như service giao hàng hoặc service thanh toán).

Nếu chúng ta thực hiện hai thao tác này một cách riêng lẻ mà không có cơ chế đảm bảo, có thể xảy ra các vấn đề sau:

- **Trường hợp 1**: Database được cập nhật thành công, nhưng việc gửi message tới message queue thất bại (do lỗi mạng, queue bị đầy, v.v.). Kết quả là các service khác không nhận được thông báo, dẫn đến mất đồng bộ.
- **Trường hợp 2**: Message được gửi thành công tới queue, nhưng giao dịch database bị rollback (do lỗi nào đó). Điều này dẫn đến việc các service khác xử lý message sai lệch, trong khi dữ liệu thực tế không tồn tại.

Để giải quyết vấn đề này, chúng ta có thể nghĩ đến việc sử dụng **two-phase commit (2PC)** hoặc giao dịch phân tán. Tuy nhiên, 2PC phức tạp, khó mở rộng và có thể gây ra hiệu suất kém trong các hệ thống lớn. Đây là lúc Transaction Outbox Pattern trở thành một giải pháp thay thế hiệu quả.

### Transaction Outbox Pattern hoạt động như thế nào?

Ý tưởng cốt lõi của Transaction Outbox Pattern là sử dụng database như một "hàng đợi trung gian" (outbox) để lưu trữ các message cần gửi, sau đó một tiến trình riêng biệt sẽ đảm nhiệm việc gửi chúng đi. Quy trình hoạt động cụ thể như sau:

1. **Bước 1: Ghi dữ liệu và message vào database trong cùng một giao dịch**
   - Khi service thực hiện một thay đổi trong database (ví dụ: thêm một đơn hàng mới), nó cũng ghi một bản ghi vào bảng "outbox" trong cùng giao dịch.
   - Bảng outbox chứa thông tin về message cần gửi, chẳng hạn như loại event (event type), nội dung (payload), và trạng thái (status).
   - Vì cả hai thao tác (ghi dữ liệu và ghi vào outbox) đều nằm trong cùng một giao dịch ACID, nên nếu giao dịch thành công, cả dữ liệu và message đều được lưu; nếu thất bại, cả hai đều bị rollback.

2. **Bước 2: Một tiến trình riêng biệt đọc và gửi message**
   - Một worker hoặc scheduler (thường được gọi là "outbox poller") định kỳ quét bảng outbox để tìm các bản ghi chưa được xử lý (ví dụ: trạng thái là "pending").
   - Worker này lấy message từ bảng outbox, gửi nó tới message broker (như Kafka, RabbitMQ), và cập nhật trạng thái của bản ghi trong outbox (ví dụ: từ "pending" sang "sent").

3. **Bước 3: Xử lý lỗi và đảm bảo tính idempotent**
   - Nếu việc gửi message thất bại, worker có thể thử lại (retry) hoặc đánh dấu bản ghi là "failed" để xử lý sau.
   - Để tránh gửi trùng lặp message (duplicate messages), hệ thống cần đảm bảo tính idempotent ở phía nhận message hoặc sử dụng cơ chế như message ID.

### Cấu trúc bảng Outbox

Bảng outbox trong database thường có cấu trúc như sau:

| Cột            | Mô tả                                                                 |
|----------------|----------------------------------------------------------------------|
| `id`           | Khóa chính, định danh duy nhất cho mỗi bản ghi trong outbox          |
| `event_type`   | Loại event (ví dụ: "OrderCreated", "PaymentProcessed")            |
| `payload`      | Nội dung message (thường là JSON hoặc chuỗi dữ liệu có cấu trúc) |
| `status`       | Trạng thái của message (ví dụ: "pending", "sent", "failed")      |
| `created_at`   | Thời gian tạo bản ghi                                               |
| `processed_at` | Thời gian message được xử lý (có thể null nếu chưa xử lý)        |

### Lợi ích của Transaction Outbox Pattern

1. **Tính nhất quán (Consistency)**:
   - Vì dữ liệu và message được ghi trong cùng một giao dịch, không có nguy cơ dữ liệu được lưu mà message không được gửi (hoặc ngược lại).

2. **Đơn giản hơn giao dịch phân tán**:
   - Không cần sử dụng 2PC hay các cơ chế phức tạp khác, giúp giảm độ phức tạp và tăng hiệu suất.

3. **Khả năng phục hồi (Resilience)**:
   - Nếu message broker tạm thời không khả dụng, message vẫn được lưu trong outbox và có thể gửi lại sau khi hệ thống khôi phục.

4. **Dễ triển khai**:
   - Chỉ cần một bảng trong database và một worker đơn giản, không yêu cầu công cụ hoặc framework phức tạp.

### Hạn chế của Transaction Outbox Pattern

1. **Độ trễ (Latency)**:
   - Vì message không được gửi ngay lập tức mà phải chờ worker xử lý, có thể xảy ra độ trễ nhỏ giữa thời điểm dữ liệu được ghi và thời điểm message được gửi.

2. **Tải thêm cho database**:
   - Việc ghi thêm vào bảng outbox và quét định kỳ có thể tăng tải cho database, đặc biệt trong hệ thống có lưu lượng lớn.

3. **Xử lý lỗi phức tạp hơn**:
   - Cần cơ chế retry, quản lý trạng thái thất bại, và đảm bảo không gửi trùng lặp message, điều này đòi hỏi thêm logic xử lý.

### Kết luận

Transaction Outbox Pattern là một giải pháp mạnh mẽ và đáng tin cậy để đảm bảo tính nhất quán trong các hệ thống phân tán mà không cần đến các cơ chế phức tạp như giao dịch phân tán. Mặc dù nó có một số hạn chế như độ trễ hoặc tải thêm cho database, nhưng với thiết kế phù hợp, nó có thể được áp dụng hiệu quả trong nhiều kịch bản thực tế, đặc biệt là trong các hệ thống microservices.