---
date: 2021-11-15
draft: false
title: "HTTP 1 Vs HTTP 2!"
summary: 'Hôm nay chúng ta sẽ đi sâu vào thế giới thú vị của HTTP - xương sống của web. Chúng ta sẽ khám phá cách nó phát triển từ HTTP 1 đến HTTP 2.'
tags: []
categories: []
---

Hôm nay chúng ta sẽ đi sâu vào thế giới thú vị của HTTP - xương sống của web. Chúng ta sẽ khám phá cách nó phát triển từ HTTP 1 đến HTTP 2.

## HTTP là gì?

HTTP (Hypertext Transfer Protocol) là cách trình duyệt giao tiếp với máy chủ web. Trình duyệt yêu cầu trang web và nhận lại chúng. Ban đầu, HTTP được thiết kế cho tài liệu hypertext - những tài liệu có liên kết đến các tài liệu khác. Nhưng các nhà phát triển nhanh chóng nhận ra HTTP có thể gửi cả hình ảnh và video. Ngày nay, nó còn được sử dụng cho API, truyền tệp và nhiều dịch vụ web khác.

## HTTP 1.0 và những hạn chế

Quay trở lại năm 1996, HTTP 1.0 được giới thiệu, nhưng trước đó đã có HTTP 0.9. HTTP 0.9 rất đơn giản: chỉ hỗ trợ phương thức GET, không có header và chỉ gửi tệp HTML. Không có HTTP header hay mã trạng thái.

HTTP 1.0 đã thêm header, mã trạng thái và các phương thức mới như POST và HEAD. Cách hoạt động rất đơn giản: trình duyệt yêu cầu trang web, máy chủ gửi nó. Mỗi yêu cầu cần một kết nối riêng, dẫn đến nhiều lần trao đổi qua lại và không hiệu quả.

Lý do là:
- Mỗi kết nối cần một bắt tay TCP (quá trình ba bước)
- Với HTTPS, còn cần thêm bắt tay TLS cho bảo mật
- Tất cả điều này xảy ra trước khi bất kỳ dữ liệu nào được gửi
- Với HTTP 1.0, quá trình này diễn ra cho mọi tài nguyên: hình ảnh, tệp CSS, JavaScript...
{{<figure src="./image1.png" width="500px" class="center">}}

## HTTP 1.1 - Cải tiến đáng kể

Năm 1997, HTTP 1.1 ra đời, khắc phục nhiều vấn đề của HTTP 1.0. Nó vẫn được sử dụng rộng rãi ngay cả sau 23 năm nhờ những tính năng tuyệt vời:

1. **Kết nối liên tục**: Kết nối giữ nguyên trừ khi được yêu cầu đóng, không cần đóng sau mỗi yêu cầu, không cần nhiều lần bắt tay TCP.
{{<figure src="./image2.png" width="500px" class="center">}}

2. **Pipelining**: Cho phép client gửi nhiều yêu cầu qua một kết nối TCP mà không cần đợi phản hồi. Ví dụ, khi trình duyệt cần hai hình ảnh, nó có thể yêu cầu chúng liên tiếp, giúp giảm thời gian chờ.
{{<figure src="./image3.png" width="500px" class="center">}}

3. **Mã hóa truyền theo khối**: Máy chủ có thể gửi phản hồi theo từng khối nhỏ thay vì đợi toàn bộ phản hồi sẵn sàng, giúp hiển thị trang ban đầu nhanh hơn.
{{<figure src="./image4.png" width="700px" class="center">}}

4. **Caching tốt hơn và yêu cầu có điều kiện**: Thêm header như Cache-Control để quản lý nội dung cache và giảm truyền dữ liệu không cần thiết và header Etag, If-Modified-Since cho phép client chỉ yêu cầu tài nguyên nếu chúng đã thay đổi.
{{<figure src="./image5.png" width="600px" class="center">}}
{{<figure src="./image6.png" width="600px" class="center">}}

Tuy nhiên, khi trang web ngày càng lớn và phức tạp, HTTP 1.1 bộc lộ vấn đề lớn: head-of-line blocking. Nếu yêu cầu đầu tiên trong pipeline bị trì hoãn, tất cả các yêu cầu khác phải đợi.
{{<figure src="./image7.png" width="600px" class="center">}}

Các nhà phát triển tìm ra cách giải quyết:
- Domain sharding: Phục vụ tài nguyên tĩnh từ các tên miền phụ, mỗi tên miền phụ có thêm 6 kết nối
- Gộp tệp: Kết hợp hình ảnh bằng sprites, nối tệp CSS và JavaScript

## HTTP 2 - Cuộc cách mạng hiệu suất

Năm 2015, HTTP 2 ra đời, được thiết kế để khắc phục các vấn đề hiệu suất của HTTP 1. Nó mang lại những cải tiến lớn:

1. **Giao thức nhị phân**: Không giống như plain text của HTTP 1, HTTP 2 sử dụng định dạng binary. Dữ liệu được chia thành các đơn vị nhỏ hơn gọi là frame, được gửi qua kết nối TCP.
{{<figure src="./image8.png" width="500px" class="center">}}

2. **Multiplexing (đa luồng trên một kết nối)**: Client và server có thể chia nhỏ tin nhắn HTTP thành các frame độc lập, trộn lẫn trong quá trình truyền và ghép lại ở đầu bên kia. Điều này giải quyết vấn đề head-of-line blocking.
{{<figure src="./image9.png" width="700px" class="center">}}

3. **Prioritization (ưu tiên tài nguyên)**: Cho phép client xác định mức độ ưu tiên của các request, giúp server biết nên gửi tài nguyên nào trước.

4. **Server push**: HTTP 2 cho phép nhiều phản hồi cho yêu cầu của client. Máy chủ có thể gửi tài nguyên bổ sung cùng với trang HTML được yêu cầu, như cung cấp cho client tài nguyên trước khi họ yêu cầu.
{{<figure src="./image10.png" width="600px" class="center">}}

5. **Nén header**: Trong HTTP 1, chỉ dữ liệu chính được nén, header được gửi dưới dạng văn bản thuần túy. HTTP 2 sử dụng **HPACK** để làm nhỏ header và ghi nhớ header trước đó để nén header tương lai hiệu quả hơn.

## Tình hình hiện tại

Tính đến nay:
- HTTP 1.1 vẫn được sử dụng rộng rãi, đặc biệt cho các trang web đơn giản
- HTTP 2 đã được áp dụng nhiều, xử lý hơn 60% yêu cầu web theo một số ước tính

Đó là hành trình qua sự phát triển của HTTP. Chúng ta đã thấy nó thay đổi từ mô hình đơn giản của HTTP 1 đến ghép kênh của HTTP 2. Các giao thức nền tảng của web đã thích ứng với nhu cầu ngày càng tăng của chúng ta về trải nghiệm trực tuyến nhanh chóng và đáng tin cậy.
