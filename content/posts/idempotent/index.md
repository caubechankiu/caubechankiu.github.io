+++
date = '2021-02-09T09:47:31+07:00'
draft = false
title = 'Idempotency'
summary = 'Tính bất biến là gì và làm thế nào để sử dụng nó một cách hợp lý'
tags = ['system design', 'distributed system']
categories = []
+++

## Idempotency

Idempotency dịch là tính bất biến

Theo Wiki thì:

> Idempotence is the property of certain operations in mathematics and computer science, that can be applied multiple times without changing the result beyond the initial application.

Nhưng `not changing the result` thực sự nghĩa là gì? Có 2 cách hiểu
- Dù có thực hiện bao nhiêu request giống nhau thì sẽ luôn nhận về cùng một response.
- Dù có thực hiện bao nhiêu request giống nhau thì cũng không làm thay đổi trạng thái của hệ thống so với lần thực hiện đầu tiên.

2 cách hiểu nghe có vẻ giống nhau và cả 2 đều có vẻ đúng khi nhìn bề ngoài. Tuy nhiên, câu thứ 2 là phù hợp hơn.
Ví dụ khi bạn thực hiện 2 request `GET` giống nhau ở 2 thời điểm khác nhau.
Không nhất thiết request thứ 2 phải trả về kết quả giống request thứ nhất, lí do là có thể trong khoảng thời gian giữa 2 request đã có 1 request `UPDATE` xen vào.

Chúng ta thường đồng ý rằng các request chỉ để đọc dữ liệu luôn có tính chất bất biến. Thế còn request `CREATE` thì sao?

Ta lấy ví dụ khi tạo mới user, request body như thế này `{"email": "caubechankiu@gmail.com", "name": "caubechankiu"}`.
Khi request `CREATE` gặp lỗi mạng hoặc `timeout`, trạng thái của hệ thống sẽ không đoán được.
Nếu user chưa được tạo, thử lại request sẽ không có vấn đề gì, user sẽ được thêm vào database.
Nhưng nếu user đã được tạo ra rồi, ta nên trả về cho client response như thế nào?
- Error: User already exist
- Success: User is created

Từ góc nhìn của client, trả về `success` có vẻ hợp lý hơn vì client chỉ quan tâm đến kết quả cuối cùng.
Mặt khác trả về `error` làm mọi thứ rõ ràng hơn vì request trước đó đã thành công rồi.

Giờ hãy giả sử chúng ta có 2 client, cả 2 cùng gửi request với nội dung giống nhau đến server.
Server đã nhận request và xử lý thành công, tuy nhiên cả 2 client đều không nhận được response do lỗi mạng.
Sau đó cả 2 client gửi lại request 1 lần nữa và cả 2 đều nhận được message `Error: User already exist`.
Với thiết lập hiện tại, chúng ta không có cách nào biết được request ban đầu của client nào đã thành công.

Một cách tiếp cận phổ biến là hãy thêm `idempotentKey`. Đôi khi cũng được gọi là `idempotentToken`, `requestId`, `nonceToken`, v.v.
Khi đó request body của 2 client sẽ trông như sau.

Client 1.
``` json
{"email": "caubechankiu@gmail.com", "name": "caubechankiu", "idempotentKey": "5l2abf"}
```

Client 2.
``` json
{"email": "caubechankiu@gmail.com", "name": "caubechankiu", "idempotentKey": "xl4n8w"}
```

Về phía server, khi xử lý request, ta cũng sẽ lưu lại `idempotentKey` và kết quả tương ứng.
Trong 2 request đầu tiên, chỉ có 1 request được xử lý thành công, giả sử đó là request của `Client 1` với `idempotentKey` là `5l2abf`.
Khi 2 client cùng thử lại request, `Client 1` sẽ nhận được response `Success: User is created` còn `Client 2` sẽ nhận được response `Error: User already exist`.

## Idempotency in Microservices
Trong bối cảnh microservices, tính bất biến đặc biệt quan trọng do bản chất phân tán của hệ thống.
Trong môi trường như vậy, lỗi mạng, trùng lặp request và lỗi hệ thống là điều không thể tránh khỏi, và tính bất biến giúp giảm thiểu tác động của những vấn đề này.

Khi một thao tác trong microservice có tính bất biến, nó đảm bảo rằng ngay cả khi thao tác được lặp lại, nó sẽ không gây ra tác dụng phụ ngoài ý muốn hoặc dẫn đến trạng thái dữ liệu không nhất quán.
Giả sử một user thực hiện thanh toán, nhưng do sự cố mạng hoặc hết thời gian chờ dẫn đến không nhận được response. Nếu client tự động gửi lại request thanh toán, service thanh toán có tính bất biến sẽ đảm bảo rằng giao dịch không bị xử lý hai lần, ngăn chặn tình trạng user bị tính phí hai lần.

Các microservice có tính bất biến được trang bị tốt hơn để xử lý các lỗi tạm thời và sự cố mạng.
Một microservice được thiết kế với tính bất biến có thể xử lý các request trùng lặp một cách trơn tru, đảm bảo rằng hệ thống vẫn duy trì độ bền vững và hoạt động chính xác ngay cả trong thời điểm không ổn định.

Với lợi ích to lớn như vậy, tại sao chúng ta không làm cho tất cả các request đều có tính chất bất biến?

Một số loại request rõ ràng có thể hưởng lợi từ tính bất biến, chẳng hạn như các giao dịch thẻ thanh toán mà chúng ta đã đề cập ở trên. Trên thực tế, hầu hết (nếu không muốn nói là tất cả) các hệ thống thanh toán đều hỗ trợ tính bất biến. Nhưng việc triển khai tính bất biến cũng phức tạp và có thể gây ra độ trễ bổ sung. Do đó, không phải service nào cũng cần tính bất biến. Ví dụ:
- Chúng ta không cần tính bất biến trong hệ thống ghi log. Nếu hệ thống khởi động lại, việc có các thông điệp log trùng lặp là chấp nhận được.
- Chúng ta không muốn tính bất biến trong các ứng dụng nhắn tin như messenger, telegram. Khi có 2 message bị trùng lặp, user chỉ cần xoá đi 1 message là được.

Không có giải pháp nào cả. Chỉ có sự đánh đổi. Và chúng ta cố gắng đạt được sự đánh đổi tốt nhất có thể.