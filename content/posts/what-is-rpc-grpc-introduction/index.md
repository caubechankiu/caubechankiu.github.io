---
title: "gRPC là gì? Giới thiệu về gRPC"
date: 2023-07-15T10:00:00+07:00
tags: ["gRPC", "RPC", "Protocol Buffers", "HTTP/2", "Microservices"]
---

# gRPC là gì? Giới thiệu về gRPC

gRPC là một framework RPC (Remote Procedure Call) mã nguồn mở được Google tạo ra vào năm 2016. Đây là phiên bản viết lại của cơ sở hạ tầng RPC nội bộ mà Google đã sử dụng trong nhiều năm.

## RPC là gì?

Trước khi đi sâu vào gRPC, chúng ta hãy hiểu RPC là gì:

- **Local Procedure Call (Lời gọi thủ tục cục bộ)**: Là một lời gọi hàm trong một tiến trình để thực thi một đoạn mã.
- **Remote Procedure Call (Lời gọi thủ tục từ xa)**: Cho phép một máy gọi và thực thi mã trên một máy khác như thể đó là một lời gọi hàm cục bộ từ góc độ người dùng.

gRPC là một triển khai phổ biến của RPC. Nhiều tổ chức đã áp dụng gRPC làm cơ chế RPC ưu tiên để kết nối số lượng lớn các microservice chạy trong và giữa các trung tâm dữ liệu.

## Điều gì làm cho gRPC trở nên phổ biến?

### 1. Hệ sinh thái phát triển sôi động

gRPC giúp việc phát triển các API chất lượng sản xuất và an toàn về kiểu dữ liệu trở nên dễ dàng và có khả năng mở rộng tốt. Cốt lõi của hệ sinh thái này là việc sử dụng Protocol Buffers làm định dạng trao đổi dữ liệu.

**Protocol Buffers** là một cơ chế độc lập với ngôn ngữ và nền tảng để mã hóa dữ liệu có cấu trúc. gRPC sử dụng Protocol Buffers để mã hóa và gửi dữ liệu qua mạng theo mặc định. Mặc dù gRPC có thể hỗ trợ các định dạng mã hóa khác như JSON, Protocol Buffers cung cấp nhiều lợi thế khiến nó trở thành định dạng mã hóa được lựa chọn cho gRPC:

- Hỗ trợ định nghĩa lược đồ kiểu mạnh. Cấu trúc dữ liệu được định nghĩa trong file proto.
- Cung cấp hỗ trợ công cụ rộng rãi để chuyển đổi lược đồ được định nghĩa trong file proto thành các lớp truy cập dữ liệu cho tất cả các ngôn ngữ lập trình phổ biến.
- Dịch vụ gRPC cũng được định nghĩa trong file proto bằng cách chỉ định các tham số phương thức RPC và kiểu trả về.
- Cùng một công cụ được sử dụng để tạo mã máy khách và máy chủ gRPC từ file proto.

Nhờ hỗ trợ nhiều ngôn ngữ lập trình, máy khách và máy chủ có thể độc lập lựa chọn ngôn ngữ lập trình và hệ sinh thái phù hợp nhất cho trường hợp sử dụng riêng của họ. Điều này thường không có trong hầu hết các framework RPC khác.

### 2. Hiệu suất cao ngay từ đầu

Hai yếu tố góp phần vào hiệu suất của gRPC:

- **Protocol Buffers** là một định dạng mã hóa nhị phân rất hiệu quả, nhanh hơn nhiều so với JSON.
- **gRPC được xây dựng trên HTTP/2** để cung cấp nền tảng hiệu suất cao ở quy mô lớn. Việc sử dụng HTTP/2 mang lại nhiều lợi ích.

gRPC sử dụng các luồng HTTP/2, cho phép nhiều luồng tin nhắn qua một kết nối TCP tồn tại lâu dài. Điều này cho phép framework gRPC xử lý nhiều cuộc gọi RPC đồng thời qua một số lượng nhỏ kết nối TCP giữa máy khách và máy chủ.

## Cách gRPC hoạt động

Để hiểu cách gRPC hoạt động, hãy xem xét một luồng điển hình từ máy khách gRPC đến máy chủ gRPC:

1. Trong ví dụ này, Order Service là máy khách gRPC và Payment Service là máy chủ gRPC.
2. Khi Order Service thực hiện cuộc gọi gRPC đến Payment Service, nó gọi mã máy khách được tạo bởi công cụ gRPC tại thời điểm xây dựng. Mã máy khách được tạo này được gọi là client stub.
3. gRPC mã hóa dữ liệu được truyền vào client stub thành Protocol Buffers và gửi nó đến lớp vận chuyển cấp thấp.
4. gRPC gửi dữ liệu qua mạng dưới dạng luồng các khung dữ liệu HTTP/2. Do mã hóa nhị phân và tối ưu hóa mạng, gRPC được cho là nhanh hơn JSON 5 lần.
5. Payment Service nhận các gói tin từ mạng, giải mã chúng và gọi ứng dụng máy chủ.
6. Kết quả trả về từ ứng dụng máy chủ được mã hóa thành Protocol Buffers và gửi đến lớp vận chuyển.
7. Order Service nhận các gói tin, giải mã chúng và gửi kết quả đến ứng dụng máy khách.

## Khi nào nên sử dụng gRPC?

gRPC rất dễ triển khai, nhưng tại sao chúng ta không thấy việc sử dụng rộng rãi gRPC giữa máy khách web và máy chủ web? Một lý do là gRPC dựa vào quyền truy cập cấp thấp hơn vào các nguyên thủy HTTP/2. Hiện tại, không có trình duyệt nào cung cấp mức độ kiểm soát cần thiết đối với các yêu cầu web để hỗ trợ máy khách gRPC.

Có thể thực hiện các cuộc gọi gRPC từ trình duyệt với sự trợ giúp của proxy. Công nghệ này được gọi là gRPC-Web. Tuy nhiên, tập tính năng không hoàn toàn tương thích với gRPC và việc sử dụng vẫn thấp so với gRPC.

gRPC phù hợp nhất trong các trường hợp sau:

- **Giao tiếp giữa các microservice trong trung tâm dữ liệu**: Hỗ trợ rộng rãi cho nhiều ngôn ngữ lập trình cho phép các dịch vụ lựa chọn ngôn ngữ và hệ sinh thái phát triển phù hợp nhất cho trường hợp sử dụng riêng của họ.
- **Trong các ứng dụng di động native**: Hiệu quả và hiệu suất của gRPC rất có ý nghĩa trong môi trường bị hạn chế về năng lượng và băng thông như các thiết bị di động.
