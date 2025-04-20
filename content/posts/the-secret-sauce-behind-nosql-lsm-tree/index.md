---
date: 2023-08-29
draft: false
title: "Bí mật đằng sau NoSQL: Cấu trúc dữ liệu LSM Tree"
summary: 'Các cơ sở dữ liệu NoSQL như Cassandra đã bùng nổ về độ phổ biến trong những năm gần đây. Một trong những động lực chính là nhu cầu không ngừng về việc tiếp nhận lượng dữ liệu khổng lồ từ ngày càng nhiều nguồn như ứng dụng di động và thiết bị IoT. Bí mật đằng sau nhiều cơ sở dữ liệu NoSQL này là một cấu trúc dữ liệu gọi là Log Structured Merge tree (LSM tree).'
tags: ["database", "nosql"]
---

Các cơ sở dữ liệu NoSQL như Cassandra đã bùng nổ về độ phổ biến trong những năm gần đây. Một trong những động lực chính là nhu cầu không ngừng về việc tiếp nhận lượng dữ liệu khổng lồ từ ngày càng nhiều nguồn như ứng dụng di động và thiết bị IoT. Bí mật đằng sau nhiều cơ sở dữ liệu NoSQL này là một cấu trúc dữ liệu gọi là Log Structured Merge tree (LSM tree).

## LSM Tree - Tối ưu hóa cho ghi nhanh

LSM tree được tối ưu hóa cho việc ghi dữ liệu nhanh. Để hiểu cách LSM tree hoạt động, hãy xem xét cách dữ liệu thường được lưu trữ trong cơ sở dữ liệu quan hệ truyền thống.

Một cơ sở dữ liệu quan hệ thường được hỗ trợ bởi cấu trúc dữ liệu gọi là B-tree. B-tree được tối ưu hóa cho việc đọc dữ liệu. 
{{<figure src="./image1.png" width="700px" class="center">}}
Việc cập nhật B-tree tương đối tốn kém vì nó liên quan đến I/O ngẫu nhiên và có thể bao gồm việc cập nhật nhiều trang trên đĩa. Điều này giới hạn tốc độ mà B-tree có thể tiếp nhận dữ liệu.

## Cách hoạt động của LSM Tree

LSM tree hoạt động khác biệt. Các thao tác ghi được tập hợp trong bộ nhớ khi chúng đến trong một cấu trúc gọi là memtable. Memtable được sắp xếp theo khóa đối tượng và thường được triển khai dưới dạng cây nhị phân cân bằng.
{{<figure src="./image2.png" width="600px" class="center">}}
Khi memtable đạt đến một kích thước nhất định, nó được đẩy xuống đĩa dưới dạng một Sorted String Table (SSTable) bất biến. SSTable lưu trữ các cặp khóa-giá trị trong một chuỗi đã sắp xếp. Các thao tác ghi này đều là I/O tuần tự, rất nhanh trên bất kỳ phương tiện lưu trữ nào. SSTable mới trở thành phân đoạn mới nhất của LSM tree.

Khi có thêm dữ liệu đến, ngày càng nhiều SSTable bất biến được tạo ra và thêm vào LSM tree, mỗi SSTable đại diện cho một tập con theo thứ tự thời gian của các thay đổi đến.
{{<figure src="./image3.png" width="700px" class="center">}}

## Xử lý cập nhật và xóa trong LSM Tree

Vì SSTable là bất biến, việc cập nhật một khóa đối tượng hiện có không ghi đè lên SSTable hiện có. Thay vào đó, một mục mới được thêm vào SSTable gần đây nhất, mục này sẽ thay thế bất kỳ mục nào trong các SSTable cũ cho khóa đối tượng đó.
{{<figure src="./image4.png" width="700px" class="center">}}

Việc xóa một đối tượng cũng cần xử lý đặc biệt, vì chúng ta không thể đánh dấu bất cứ thứ gì trong SSTable là đã xóa. Để thực hiện xóa, hệ thống thêm một dấu hiệu gọi là tombstone vào SSTable gần đây nhất cho khóa đối tượng. Khi gặp tombstone khi đọc, chúng ta biết đối tượng đã bị xóa. Và đúng vậy, hơi khó hiểu là việc xóa lại chiếm thêm dung lượng.
{{<figure src="./image5.png" width="700px" class="center">}}

## Quá trình đọc dữ liệu

Để phục vụ yêu cầu đọc, chúng ta trước tiên cố gắng tìm khóa trong memtable, sau đó trong SSTable gần đây nhất trong LSM tree, rồi đến SSTable tiếp theo, và cứ tiếp tục. Vì SSTable được sắp xếp, việc tra cứu có thể được thực hiện hiệu quả.
{{<figure src="./image6.png" width="700px" class="center">}}

## Vấn đề với SSTable tích lũy và giải pháp

Sự tích lũy của SSTable gây ra hai vấn đề:
1. Khi số lượng SSTable tăng lên, việc tra cứu một khóa sẽ mất nhiều thời gian hơn.
2. Khi SSTable tích lũy, có ngày càng nhiều mục lỗi thời khi khóa được cập nhật và tombstone được thêm vào, chiếm dụng không gian đĩa quý giá.

Để khắc phục những vấn đề này, có một quá trình hợp nhất và nén (merging and compaction) định kỳ chạy trong nền để hợp nhất các SSTable và loại bỏ các giá trị lỗi thời hoặc đã xóa. Điều này giải phóng không gian đĩa và giới hạn số lượng SSTable mà một thao tác đọc phải xem xét. Vì SSTable được sắp xếp, quá trình hợp nhất và nén này đơn giản và hiệu quả, tương tự như giai đoạn hợp nhất trong thuật toán mergesort.

## Tóm tắt cách LSM Tree cung cấp ghi nhanh

1. LSM tree lưu các thao tác ghi đến trong bộ nhớ.
2. Khi bộ nhớ đầy, chúng ta sắp xếp và đẩy nó xuống đĩa dưới dạng SSTable bất biến.
3. Số lượng SSTable tăng lên khi nhiều bộ đệm được đẩy xuống đĩa.
{{<figure src="./image7.png" width="700px" class="center">}}
4. Điều này tạo ra vấn đề cho việc đọc vì mỗi lần đọc phải tìm kiếm qua các SSTable này.
5. Để giới hạn số lượng SSTable cần tìm kiếm cho mỗi lần đọc, LSM tree hợp nhất các SSTable và nén chúng trong nền.
{{<figure src="./image8.png" width="700px" class="center">}}

## Chiến lược nén (Compaction)

Khi các SSTable được hợp nhất, chúng được tổ chức thành các cấp. Đây là nơi phần "tree" của LSM tree xuất hiện. Có các chiến lược khác nhau để xác định khi nào và cách thức các SSTable được hợp nhất và nén.

Có hai chiến lược rộng:
1. **Size Tiered Compaction**: Được tối ưu hóa cho thông lượng ghi.
2. **Leveled Compaction**: Được tối ưu hóa cho đọc.

Một số điểm quan trọng cần nhớ:
- Nén giữ cho số lượng SSTable có thể quản lý được.
- SSTable được tổ chức thành các cấp.
- Mỗi cấp trở nên lớn hơn theo cấp số nhân khi SSTable từ cấp trên được hợp nhất vào nó.
- Nén tiêu thụ nhiều I/O. Nén không được điều chỉnh tốt có thể làm cạn kiệt hệ thống và làm chậm cả đọc và ghi.

## Tối ưu hóa phổ biến cho LSM Tree

Có nhiều chiến lược tối ưu hóa cố gắng cung cấp hiệu suất đọc gần với B-tree. Dưới đây là một số tối ưu hóa phổ biến:

1. **Bảng tóm tắt trong bộ nhớ**: Để tra cứu một khóa, hệ thống thực hiện tìm kiếm trên SSTable ở mọi cấp. Mặc dù tìm kiếm nhanh trên dữ liệu đã sắp xếp, việc đi qua tất cả SSTable trên đĩa tiêu thụ nhiều I/O. Nhiều hệ thống giữ một bảng tóm tắt trong bộ nhớ chứa phạm vi min/max của mỗi khối đĩa ở mọi cấp. Điều này cho phép hệ thống bỏ qua tìm kiếm trên các khối đĩa đó nơi khóa không nằm trong phạm vi, tiết kiệm nhiều I/O ngẫu nhiên.
{{<figure src="./image9.png" width="700px" class="center">}}

2. **Bộ lọc Bloom**: Một vấn đề khác có thể tốn kém là tra cứu một khóa không tồn tại, điều này đòi hỏi phải xem xét tất cả các khối đủ điều kiện ở tất cả các cấp. Hầu hết các hệ thống giữ một bộ lọc Bloom ở mỗi cấp. Bộ lọc Bloom là một cấu trúc dữ liệu tiết kiệm không gian trả về "không" chắc chắn nếu khóa không tồn tại, và "có thể có" nếu khóa có thể tồn tại. Điều này cho phép hệ thống bỏ qua hoàn toàn một cấp nếu khóa không tồn tại ở đó, giảm đáng kể số lượng I/O ngẫu nhiên cần thiết.
{{<figure src="./image10.png" width="700px" class="center">}}

## Kết luận

Các cơ sở dữ liệu NoSQL được hỗ trợ bởi LSM tree có thể được điều chỉnh để hỗ trợ tốc độ ghi rất cao. Như với bất kỳ cơ sở dữ liệu nào, điều chỉnh đúng là chìa khóa. Đối với LSM tree, điều chỉnh nén là quan trọng nhất.
