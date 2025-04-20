---
date: 2023-07-25
draft: false
title: "Những Cạm Bẫy Caching Mà Mọi Lập Trình Viên Nên Biết"
summary: 'Caching là một khái niệm quan trọng trong thiết kế hệ thống, đóng vai trò then chốt cho hiệu suất nhưng cũng có thể gây ra nhiều vấn đề nếu không được xử lý đúng cách.'
tags: ["caching", "system design"]
categories: []
---

## Caching là gì và tại sao nó quan trọng?

Caching là một khái niệm quan trọng trong thiết kế hệ thống, đóng vai trò then chốt cho hiệu suất nhưng cũng có thể gây ra nhiều vấn đề nếu không được xử lý đúng cách. Về cơ bản, caching giống như một lớp bộ nhớ lưu trữ bản sao của dữ liệu thường xuyên được truy cập. Đây là chiến lược giúp tăng tốc hệ thống bằng cách giữ sẵn dữ liệu, giảm nhu cầu phải lấy dữ liệu từ cơ sở dữ liệu chậm hơn mỗi khi có yêu cầu.
{{<figure src="./image1.png" width="600px" class="center">}}
Ví dụ, hãy nghĩ về một cơ sở dữ liệu chứa hồ sơ người dùng. Một bộ nhớ cache cho cơ sở dữ liệu này có thể lưu trữ các hồ sơ người dùng phổ biến nhất, để khi ai đó xem hồ sơ, nó tải ngay lập tức thay vì phải truy vấn cơ sở dữ liệu mỗi lần xem.

Mặc dù mang lại những lợi ích về hiệu suất, caching cũng đưa ra những thách thức mới. Hãy cùng tìm hiểu những vấn đề phổ biến có thể xảy ra.

## Cache Stampede (Hiện tượng giẫm đạp cache)

Hãy tưởng tượng một máy chủ web sử dụng Redis để cache các trang trong một khoảng thời gian nhất định. Các trang này yêu cầu nhiều lệnh gọi cơ sở dữ liệu và mất vài giây để hiển thị. Với caching, hệ thống vẫn đáp ứng tốt dưới tải cao vì các trang nặng về tài nguyên được phục vụ từ cache.

Tuy nhiên, trong điều kiện lưu lượng cực cao, nếu một trang được cache hết hạn, nhiều luồng trên toàn bộ cụm web có thể cố gắng làm mới trang đã hết hạn cùng một lúc. Dòng yêu cầu này có thể làm quá tải cơ sở dữ liệu, có khả năng gây lỗi hệ thống và ngăn trang được cache lại.
{{<figure src="./image2.png" width="500px" class="center">}}

### Cách ngăn chặn Cache Stampede:

1. **Khóa (Locking)**: Khi cache miss, mỗi yêu cầu cố gắng lấy khóa cho khóa cache đó trước khi tính toán lại trang đã hết hạn. Nếu không lấy được khóa, có một số tùy chọn:
   - Yêu cầu có thể đợi cho đến khi giá trị được tính toán lại bởi một luồng khác.
   - Yêu cầu có thể trả về phản hồi "không tìm thấy" ngay lập tức và để client xử lý tình huống với thử lại sau.
   - Hệ thống có thể duy trì phiên bản cũ của mục được cache để sử dụng tạm thời trong khi giá trị mới được tính toán lại.

   Khóa yêu cầu một thao tác ghi bổ sung cho chính khóa, và việc triển khai khóa đúng cách có thể khá thách thức.

2. **Chuyển việc tính toán lại cho quy trình bên ngoài**: Phương pháp này có thể được kích hoạt theo nhiều cách - chủ động khi khóa cache gần hết hạn, hoặc phản ứng khi xảy ra cache miss. Cách tiếp cận này thêm một phần chuyển động khác vào kiến trúc đòi hỏi bảo trì và giám sát liên tục.

3. **Hết hạn sớm theo xác suất**: Trong chiến lược này, mỗi yêu cầu có một cơ hội nhỏ chủ động kích hoạt việc tính toán lại giá trị cache trước khi nó thực sự hết hạn. Khả năng này tăng lên khi thời gian hết hạn đến gần. Việc làm mới sớm theo từng giai đoạn này giảm thiểu tác động của hiện tượng giẫm đạp vì ít quy trình sẽ hết hạn hơn.

## Cache Penetration (Xâm nhập cache)

Điều này xảy ra khi có yêu cầu về dữ liệu không tồn tại trong cache hoặc cơ sở dữ liệu. Điều này dẫn đến tải không cần thiết, khi hệ thống cố gắng truy xuất dữ liệu không tồn tại. Điều này có thể làm mất ổn định toàn bộ hệ thống nếu khối lượng yêu cầu cao.
{{<figure src="./image3.png" width="500px" class="center">}}

### Cách giảm thiểu Cache Penetration:

1. **Triển khai giá trị giữ chỗ cho các khóa không tồn tại**: Bằng cách này, các yêu cầu tiếp theo cho cùng dữ liệu bị thiếu sẽ truy cập vào các giá trị giữ chỗ trong cache thay vì vô ích truy cập vào cơ sở dữ liệu lần nữa. Đặt TTL (Time-To-Live) thích hợp cho các giá trị giữ chỗ này ngăn chúng chiếm không gian cache vô thời hạn. Tuy nhiên, cách tiếp cận này đòi hỏi điều chỉnh cẩn thận để tránh tiêu thụ tài nguyên cache đáng kể, đặc biệt là đối với các hệ thống có nhiều tra cứu các khóa không tồn tại.

2. **Sử dụng bộ lọc Bloom**: Đây là cấu trúc dữ liệu xác suất tiết kiệm không gian để nhanh chóng kiểm tra xem các phần tử có trong tập hợp trước khi truy vấn cơ sở dữ liệu hay không. Cách hoạt động:
   - Khi thêm bản ghi mới vào bộ nhớ, khóa của chúng được ghi lại trong bộ lọc Bloom.
   - Trước khi lấy bản ghi, ứng dụng kiểm tra bộ lọc Bloom trước.
   - Nếu khóa không có, bản ghi chắc chắn không tồn tại, cho phép ứng dụng trả về giá trị null ngay lập tức.
   - Tuy nhiên, sự hiện diện tích cực của khóa không đảm bảo sự tồn tại - một tỷ lệ nhỏ các lần đọc cache vẫn có thể dẫn đến miss.

## Cache Crash (Sự cố cache)

Hãy tưởng tượng: toàn bộ hệ thống cache đột nhiên gặp sự cố. Điều gì xảy ra tiếp theo? Khi không có lớp cache, mọi yêu cầu đều trực tiếp đổ vào cơ sở dữ liệu. Sự gia tăng đột ngột về lưu lượng này có thể dễ dàng làm quá tải cơ sở dữ liệu, đe dọa sự ổn định tổng thể của hệ thống. Điều tồi tệ hơn? Người dùng bắt đầu liên tục nhấn làm mới, làm trầm trọng thêm vấn đề.

{{<figure src="./image4.png" width="500px" class="center">}}

Một người anh em họ gần gũi với Cache Crash là Cache Avalanche (Hiện tượng tuyết lở cache). Điều này có thể xảy ra trong hai kịch bản:
1. Khi một lượng lớn dữ liệu được cache hết hạn cùng một lúc.
2. Khi cache khởi động lại và nó trống rỗng (cold cache).

Trong cả hai trường hợp, một làn sóng yêu cầu dồn dập đổ vào cơ sở dữ liệu cùng một lúc. Sự gia tăng tải đột ngột này làm quá tải hệ thống, giống như hàng trăm người đột ngột chen lấn qua một cánh cửa nhỏ sau khi có báo động hỏa hoạn.

### Cách giải quyết những thách thức này:

1. **Triển khai bộ ngắt mạch (circuit breaker)**: Tạm thời chặn các yêu cầu đến khi hệ thống rõ ràng bị quá tải. Điều này ngăn chặn sự sụp đổ hoàn toàn và mua thời gian để phục hồi.

2. **Triển khai cụm cache có tính sẵn sàng cao với dự phòng**: Nếu các phần của cache gặp sự cố, các phần khác vẫn hoạt động. Mục tiêu là giảm mức độ nghiêm trọng của các sự cố hoàn toàn.

3. **Làm nóng cache trước (cache prewarming)**: Đặc biệt quan trọng sau khi khởi động lạnh. Ở đây, dữ liệu thiết yếu được chủ động đưa vào cache lạnh trước khi đưa vào sử dụng. Điều này tránh việc đột ngột làm quá tải cơ sở dữ liệu.

## Phân biệt Cache Stampede và Cache Avalanche

Mặc dù nghe có vẻ giống nhau, nhưng hai hiện tượng này có sự khác biệt:
- **Cache Stampede** xảy ra khi nhiều yêu cầu đồng thời truy cập vào cùng một mục cache đã hết hạn, làm quá tải cơ sở dữ liệu khi nó cố gắng làm mới chỉ một điểm dữ liệu đó.
- **Cache Avalanche** là vấn đề rộng hơn khi nhiều yêu cầu cho các dữ liệu khác nhau tràn ngập hệ thống sau khi cache bị xóa hoặc khởi động lại, gây áp lực lên tài nguyên.

## Kết luận

Caching là một công cụ mạnh mẽ để cải thiện hiệu suất hệ thống, nhưng nó đi kèm với những thách thức riêng. Bằng cách hiểu và chuẩn bị cho những cạm bẫy phổ biến như Cache Stampede, Cache Penetration và Cache Crash, bạn có thể xây dựng các hệ thống mạnh mẽ hơn, đáng tin cậy hơn.