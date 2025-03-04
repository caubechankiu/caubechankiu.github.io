+++
date = '2020-03-04T09:47:31+07:00'
draft = false
title = 'Optimistic Lock vs Pessimistic Lock'
tags = ['database']
categories = []
+++

# Optimistic Lock vs Pessimistic Lock: Sự khác biệt và ứng dụng thực tế

Trong quản lý giao dịch và cạnh tranh truy cập dữ liệu, có hai chiến lược chính để kiểm soát tính nhất quán: **Optimistic Lock** và **Pessimistic Lock**. Mỗi loại có cách tiếp cận khác nhau trong việc xử lý xung đột dữ liệu. Bài viết này sẽ phân tích chi tiết hai loại lock này, đồng thời giải thích hai dạng của **Pessimistic Lock** là **Shared Lock** và **Exclusive Lock**.

## 1. Optimistic Lock

### **Khái niệm**

Optimistic Lock giả định rằng xung đột dữ liệu ít xảy ra, do đó không khóa dữ liệu khi đọc. Thay vào đó, khi cập nhật, hệ thống kiểm tra xem dữ liệu có bị thay đổi bởi giao dịch khác hay không. Nếu có thay đổi, thao tác sẽ thất bại và yêu cầu thực hiện lại.

### **Triển khai**

Optimistic Lock thường được triển khai bằng cách thêm một **trường phiên bản (version)** hoặc **dấu thời gian (timestamp)** vào bảng dữ liệu:

Ví dụ với SQL:

```sql
UPDATE users 
SET balance = balance - 100, version = version + 1
WHERE id = 1 AND version = 3;
```

Nếu phiên bản đã bị thay đổi bởi giao dịch khác trước đó, câu lệnh trên sẽ không cập nhật thành công, buộc ứng dụng phải đọc lại dữ liệu và thử lại.

#### **Ví dụ thực tế - Đặt vé máy bay**

1. **Giao dịch A** đọc số lượng chỗ ngồi còn trống (10 chỗ).
2. **Giao dịch B** cũng đọc cùng lúc số lượng chỗ trống (10 chỗ).
3. **Giao dịch A** đặt 3 vé → cập nhật số chỗ còn lại là 7 (version +1).
4. **Giao dịch B** đặt 5 vé → kiểm tra version thấy đã thay đổi → thất bại → phải đọc lại số chỗ mới (7 chỗ) và thử lại.

### **Ưu điểm & Nhược điểm**

✅ **Ưu điểm**:

- Hiệu suất cao trong môi trường có ít xung đột.
- Không giữ lock lâu, tránh gây nghẽn hiệu suất hệ thống.

❌ **Nhược điểm**:

- Khi có nhiều xung đột, việc thử lại liên tục có thể gây tốn tài nguyên.
- Không phù hợp với hệ thống có tần suất ghi dữ liệu cao.

## 2. Pessimistic Lock

### **Khái niệm**

Pessimistic Lock giả định rằng xung đột dữ liệu có thể xảy ra, nên nó **khóa dữ liệu ngay khi đọc** để ngăn các giao dịch khác thay đổi dữ liệu đó trước khi giao dịch hiện tại hoàn tất.

Pessimistic Lock có hai loại chính:

1. **Shared Lock (S-lock) - Khóa chia sẻ**
2. **Exclusive Lock (X-lock) - Khóa độc quyền**

### **2.1 Shared Lock (Khóa chia sẻ)**

#### **Cách hoạt động**

- **Nhiều giao dịch có thể đọc cùng một dữ liệu khi có shared lock.**
- **Không thể cập nhật dữ liệu nếu có shared lock tồn tại.**
- Khi shared lock được giữ bởi một giao dịch, các giao dịch khác vẫn có thể đọc nhưng không thể ghi dữ liệu đó cho đến khi shared lock được giải phóng.

#### **Ví dụ về Shared Lock - Báo cáo tài chính**

1. **Giao dịch A** đọc báo cáo tài chính với shared lock.
2. **Giao dịch B** cũng có thể đọc cùng báo cáo đó mà không bị chặn.
3. **Giao dịch C** muốn cập nhật báo cáo nhưng phải đợi cả A và B hoàn thành.
4. Khi A và B hoàn tất, shared lock được giải phóng và C có thể cập nhật dữ liệu.

#### **Khi nào nên dùng Shared Lock?**

- **Đọc dữ liệu quan trọng nhưng không muốn thay đổi ngay lập tức.**
- **Tránh hiện tượng dirty read (đọc dữ liệu chưa commit).**

### **2.2 Exclusive Lock (Khóa độc quyền)**

#### **Cách hoạt động**

- **Chỉ một giao dịch duy nhất có thể đọc và ghi dữ liệu khi có exclusive lock.**
- Các giao dịch khác muốn đọc hoặc ghi dữ liệu đều phải đợi giao dịch hiện tại hoàn thành.

#### **Ví dụ về Exclusive Lock - Bán cổ phiếu**

Giả sử có hai người cùng muốn bán cổ phiếu:

1. **Giao dịch A**:
   - Đọc số lượng cổ phiếu trong tài khoản.
   - Khóa bản ghi với `SELECT ... FOR UPDATE`.
   - Giảm số lượng cổ phiếu và cập nhật vào database.
   - Commit giao dịch và giải phóng lock.
2. **Giao dịch B**:
   - Khi giao dịch A chưa hoàn tất, giao dịch B muốn thực hiện bán cổ phiếu phải đợi vì bản ghi đã bị khóa.
   - Khi A hoàn thành, B mới có thể đọc dữ liệu và tiếp tục thực hiện giao dịch.

#### **Khi nào nên dùng Exclusive Lock?**

- **Cần đảm bảo không có giao dịch khác thao tác cùng lúc trên dữ liệu.**
- **Tránh điều kiện tranh chấp dữ liệu (race condition).**
- Ví dụ: Khi thực hiện giao dịch rút tiền, bán hàng, cập nhật trạng thái đơn hàng.

### **Ưu điểm & Nhược điểm của Pessimistic Lock**

✅ **Ưu điểm**:

- Đảm bảo tính nhất quán dữ liệu tuyệt đối bằng cách chặn hoàn toàn các giao dịch xung đột.
- Giúp tránh các lỗi do điều kiện tranh chấp dữ liệu (race condition).
- Phù hợp với hệ thống có nhiều giao dịch ghi và cập nhật dữ liệu liên tục.

❌ **Nhược điểm**:

- Hiệu suất thấp hơn do phải giữ lock trong suốt giao dịch, có thể làm giảm khả năng mở rộng hệ thống.
- Nếu lock bị giữ quá lâu, có thể gây ra tình trạng **deadlock**.
- Có thể gây nghẽn tài nguyên khi nhiều giao dịch cạnh tranh cùng một dữ liệu.

## 3. So sánh Optimistic Lock và Pessimistic Lock

| Tiêu chí         | Optimistic Lock                                       | Pessimistic Lock (Exclusive)                |
| ---------------- | ----------------------------------------------------- | ------------------------------------------- |
| Cách hoạt động   | Không khóa dữ liệu khi đọc, chỉ kiểm tra khi cập nhật | Khóa dữ liệu ngay khi đọc để tránh xung đột |
| Khi nào phù hợp? | Môi trường ít xung đột                                | Môi trường có nhiều xung đột                |
| Hiệu suất        | Cao (nếu xung đột thấp)                               | Thấp hơn do phải giữ lock lâu               |
| Xử lý xung đột   | Thất bại khi cập nhật, phải thử lại                   | Chặn giao dịch khác để tránh xung đột       |

## 4. Kết luận

- **Optimistic Lock** phù hợp với hệ thống có ít xung đột và số lượng ghi dữ liệu thấp.
- **Pessimistic Lock** phù hợp với hệ thống có nhiều giao dịch cập nhật dữ liệu, trong đó **Exclusive Lock** là lựa chọn phổ biến nhất.
- **Shared Lock** có ít ứng dụng thực tế vì không thực sự ngăn chặn được việc cập nhật dữ liệu.

Tùy vào từng tình huống cụ thể mà bạn có thể chọn chiến lược lock phù hợp để đảm bảo tính toàn vẹn dữ liệu mà vẫn tối ưu hiệu suất hệ thống.
