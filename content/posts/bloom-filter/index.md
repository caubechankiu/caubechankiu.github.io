+++
date = '2020-10-14T09:47:31+07:00'
draft = false
title = 'Bloom Filter'
summary = 'Nếu bạn từng nghe đến các cấu trúc dữ liệu như **Hash Table** hay **Binary Search Tree**, thì **Bloom Filter** có thể là một khái niệm thú vị tiếp theo để khám phá.'
tags = ['algorithm']
categories = []
+++

Nếu bạn từng nghe đến các cấu trúc dữ liệu như **Hash Table** hay **Binary Search Tree**, thì **Bloom Filter** có thể là một khái niệm thú vị tiếp theo để khám phá. Bloom Filter không phải là một cấu trúc dữ liệu thông thường dùng để lưu trữ và truy xuất dữ liệu chính xác, mà là một công cụ xác suất được thiết kế để kiểm tra xem một phần tử có thuộc một tập hợp hay không. Với hiệu suất cao và khả năng tiết kiệm bộ nhớ, Bloom Filter đã trở thành một phần không thể thiếu trong nhiều hệ thống lớn như cơ sở dữ liệu, mạng, và thậm chí cả blockchain. Hãy cùng tìm hiểu chi tiết về nó!

## Bloom Filter là gì?

**Bloom Filter** là một cấu trúc dữ liệu xác suất do Burton Howard Bloom đề xuất vào năm 1970. Nó được thiết kế để trả lời câu hỏi: **"Phần tử này có nằm trong tập hợp không?"** với tốc độ nhanh và sử dụng rất ít bộ nhớ. Tuy nhiên, nó không hoàn toàn chính xác 100% - điều này có nghĩa là đôi khi nó có thể trả lời "có" trong khi phần tử thực sự không tồn tại (*false positive*), nhưng nó sẽ không bao giờ trả lời "không" khi phần tử thực sự có mặt (không có *false negative*).

Bloom Filter thường được biểu diễn dưới dạng một **mảng bit** (*bit array*) với kích thước cố định, cùng với một số **hàm băm** (*hash functions*). Thay vì lưu trữ toàn bộ phần tử như các cấu trúc dữ liệu khác, nó chỉ ghi lại "dấu vết" của phần tử thông qua các vị trí bit được bật lên (từ 0 thành 1).

## Cách Bloom Filter hoạt động

Hãy tưởng tượng bạn có một danh sách email và muốn kiểm tra nhanh xem một email cụ thể có trong danh sách hay không mà không cần duyệt qua toàn bộ danh sách. Bloom Filter hoạt động theo hai bước chính: **thêm phần tử** và **kiểm tra phần tử**.

### 1. Thêm phần tử (Insertion)
- Khi bạn muốn thêm một phần tử (ví dụ: "email@example.com") vào Bloom Filter:
  1. Phần tử được đưa qua **k hàm băm** (*hash functions*), mỗi hàm sẽ trả về một chỉ số tương ứng trong mảng bit.
  2. Tại mỗi chỉ số này, bit tương ứng trong mảng sẽ được đặt thành **1** (nếu chưa là 1).
- **Ví dụ**: Nếu bạn có 3 hàm băm và mảng bit dài 10, thêm "email@example.com" có thể đặt các bit ở vị trí 2, 5, và 8 thành 1.

### 2. Kiểm tra phần tử (Query)
- Khi kiểm tra xem một phần tử có trong tập hợp hay không:
  1. Phần tử được đưa qua cùng **k hàm băm** để lấy các chỉ số.
  2. Kiểm tra từng bit tại các chỉ số đó:
     - Nếu **tất cả các bit đều là 1**, Bloom Filter trả về "có thể có" (*possibly present*).
     - Nếu **ít nhất một bit là 0**, phần tử chắc chắn không có trong tập hợp.
- **Lưu ý**: "Có thể có" không đảm bảo 100% phần tử tồn tại, vì các bit này có thể đã được bật bởi các phần tử khác.

### Ví dụ minh họa
Giả sử bạn có Bloom Filter với mảng bit 10 phần tử (ban đầu đều là 0) và 3 hàm băm:
- Thêm "ribeye": Hàm băm trả về vị trí 1, 3 và 4 → mảng bit: `[0, 1, 0, 1, 1, 0, 0, 0, 0, 0]`.
- Thêm "potato": Hàm băm trả về vị trí 0, 4 và 8 → mảng bit: `[1, 1, 0, 1, 1, 0, 0, 0, 1, 0]`.
- Kiểm tra "pork chop": Hàm băm trả về 0, 5 và 8.
  - Vị trí 0 và 8 = 1 (OK), nhưng vị trí 5 = 0 → "pork chop" chắc chắn không có.
{{<figure src="./image1.png" width="400px" class="center">}}
- Kiểm tra "ribeye": Hàm băm trả về 1, 3 và 4:
  - Vị trí 1, 3 và 4 = 1 → "ribeye" có thể có (thực tế là có).
{{<figure src="./image2.png" width="400px" class="center">}}
- Kiểm tra "lemon": Hàm băm trả về 1, 4 và 8:
  - Vị trí 1, 4 và 8 = 1 → "lemon" có thể có (dù thực tế là không có) → *false positive*
{{<figure src="./image3.png" width="400px" class="center">}}

## Công thức và thông số quan trọng

Bloom Filter có một số thông số quan trọng ảnh hưởng đến hiệu suất:
- **m**: Kích thước mảng bit (số bit).
- **n**: Số phần tử dự kiến thêm vào.
- **k**: Số hàm băm.
- **P**: Xác suất *false positive* (sai dương).

Xác suất *false positive* có thể được tính gần đúng bằng công thức:

$$ 
P = \left(1 - e^{-kn/m} \right)^k
$$

Để tối ưu, kích thước mảng bit *m* và số hàm băm *k* thường được chọn như sau:

$$
m = -\frac{n \ln p}{(\ln 2)^2} \quad \text{và} \quad k = \frac{m}{n} \ln 2
$$

**Ví dụ**: Nếu bạn muốn lưu 1 triệu phần tử với xác suất *false positive* là 1%, bạn cần mảng bit khoảng 9.6 triệu bit (1.2 MB) và 7 hàm băm.

## Ưu điểm của Bloom Filter
1. **Tiết kiệm không gian**: Chỉ cần một mảng bit nhỏ thay vì lưu toàn bộ dữ liệu.
2. **Hiệu suất cao**: Thêm và kiểm tra phần tử có độ phức tạp O(k), không phụ thuộc vào kích thước tập hợp.
3. **Không có *false negative***: Nếu Bloom Filter nói "không có", thì chắc chắn không có.

## Nhược điểm của Bloom Filter
1. ***False positive***: Có thể trả lời sai rằng phần tử tồn tại.
2. **Không xóa được phần tử**: Vì các bit được chia sẻ giữa nhiều phần tử, việc xóa một phần tử có thể ảnh hưởng đến các phần tử khác (dù có biến thể như *Counting Bloom Filter* khắc phục được điều này).
3. **Cần cấu hình trước**: Phải xác định kích thước *m* và số hàm băm *k* dựa trên số phần tử dự kiến.

## Ứng dụng thực tế của Bloom Filter
Bloom Filter xuất hiện trong rất nhiều hệ thống thực tế:
1. **Cơ sở dữ liệu**: 
   - Google Bigtable và Apache Cassandra dùng Bloom Filter để giảm truy cập đĩa bằng cách kiểm tra xem một khóa có tồn tại trong tập hợp trước khi đọc dữ liệu.
2. **Bộ nhớ cache**: 
   - Các hệ thống như Squid Proxy dùng nó để kiểm tra nhanh xem một URL đã được lưu trong cache hay chưa.
3. **Kiểm tra trùng lặp**: 
   - Các công cụ như Google Chrome dùng Bloom Filter để cảnh báo người dùng về các trang web độc hại mà không cần lưu toàn bộ danh sách.
4. **Blockchain**: 
   - Bitcoin sử dụng Bloom Filter trong giao thức SPV (*Simplified Payment Verification*) để kiểm tra giao dịch mà không cần tải toàn bộ blockchain.

## Biến thể của Bloom Filter
Ngoài phiên bản cơ bản, còn có một số biến thể nổi bật:
- ***Counting Bloom Filter***: Thay bit bằng bộ đếm (*counter*) để hỗ trợ xóa phần tử.
- ***Scalable Bloom Filter***: Tự động mở rộng khi tập hợp lớn hơn dự kiến.
- ***Cuckoo Filter***: Cải thiện hiệu suất và hỗ trợ xóa với tỷ lệ *false positive* thấp hơn.

## Kết luận

Bloom Filter là một công cụ tuyệt vời khi bạn cần kiểm tra thành viên trong một tập hợp lớn với tốc độ nhanh và ít tài nguyên, chấp nhận một chút sai số nhỏ. Dù không hoàn hảo trong mọi tình huống, sự kết hợp giữa hiệu quả và đơn giản của nó đã khiến Bloom Filter trở thành một phần không thể thiếu trong các hệ thống hiện đại. Nếu bạn đang xây dựng một ứng dụng cần xử lý dữ liệu lớn, hãy cân nhắc sử dụng Bloom Filter – nó có thể là "vị cứu tinh" cho vấn đề hiệu suất của bạn!