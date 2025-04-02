+++
date = '2022-07-07T09:47:31+07:00'
draft = false
title = 'ACID'
summary = 'ACID là viết tắt của Atomicity, Consistency, Isolation và Durability - bốn thuộc tính quan trọng đảm bảo các transaction trong database đáng tin cậy, ngay cả khi có sự cố xảy ra.'
tags = ['database']
categories = []
+++

ACID là viết tắt của Tính nguyên tử (Atomicity), Tính nhất quán (Consistency), Tính cô lập (Isolation) và Tính bền vững (Durability) - bốn thuộc tính quan trọng đảm bảo các transaction trong database đáng tin cậy, ngay cả khi có sự cố xảy ra. Nếu bạn làm việc với database, việc hiểu ACID là điều bắt buộc.

## Atomicity
Tính nguyên tử (Atomicity) có nghĩa là một transaction là một thỏa thuận "tất cả hoặc không có gì". Nếu bất kỳ phần nào của transaction thất bại, toàn bộ transaction sẽ bị rollback như chưa từng xảy ra. Các hệ thống quản lý transaction thường sử dụng cơ chế ghi log để hỗ trợ tính năng rollback này.

{{<figure src="image1.png" width="400px" class="center">}}

Hãy tưởng tượng bạn đang xây dựng một ứng dụng ngân hàng thực hiện chuyển `$100` từ Alice sang Bob. Điều này có nghĩa là cần cập nhật hai thứ - trừ `$100` từ số dư của Alice và cộng `$100` vào số dư của Bob.

{{<figure src="image2.png" width="600px" class="center">}}

Tính nguyên tử đảm bảo rằng cả hai cập nhật này hoặc xảy ra cùng nhau, hoặc không xảy ra. Nếu có lỗi xảy ra giữa chừng, hệ thống quản lý transaction sẽ sử dụng log để hoàn tác mọi thay đổi chưa hoàn thành, giúp tránh mất tiền hoặc phát sinh thêm tiền không hợp lệ. Transaction là không thể chia nhỏ, giống như một nguyên tử.

{{<figure src="image3.png" width="600px" class="center">}}

## Consistency
Tính nhất quán (Consistency) có nghĩa là một transaction phải tuân theo tất cả các quy tắc và để lại database ở trạng thái hợp lệ. Bất kỳ dữ liệu nào được ghi trong một transaction phải hợp lệ theo các ràng buộc, trigger và các quy tắc khác mà chúng ta đã thiết lập. Hệ thống database tự động đảm bảo tính nhất quán bằng cách kiểm tra vi phạm ràng buộc trong quá trình thực hiện transaction.

{{<figure src="image4.png" width="300px" class="center">}}

Ví dụ, giả sử chúng ta có một quy tắc rằng số dư tài khoản người dùng không được âm. Nếu một transaction cố gắng rút nhiều tiền hơn số dư hiện có của người dùng, hệ thống database sẽ phát hiện vi phạm tính nhất quán này và hủy transaction để giữ cho database không bị sai lệch. Tính nhất quán giúp ngăn chặn dữ liệu không hợp lệ làm hỏng database.

{{<figure src="image5.png" width="500px" class="center">}}

## Isolation
Tính cô lập (Isolation) liên quan đến cách các transaction đồng thời tương tác với nhau. Ngay cả khi nhiều transaction đang chạy cùng một lúc, tính cô lập làm cho mỗi transaction có cảm giác như nó đang sử dụng database một cách độc lập.

{{<figure src="image6.png" width="400px" class="center">}}

Mức độ cô lập cao nhất được gọi là "serializable," trong đó các transaction được thực thi tuần tự, từng transaction một, như thể chúng đang xếp hàng lần lượt. Điều này cung cấp mức nhất quán mạnh nhất, nhưng có thể làm chậm hệ thống đáng kể vì mỗi transaction phải chờ đến lượt. 

{{<figure src="image7.png" width="500px" class="center">}}

Để tăng tốc độ, các database thường cung cấp các mức cô lập thấp hơn, cho phép nhiều transaction chạy đồng thời. Nhưng có một vấn đề – các mức thấp hơn này đôi khi có thể dẫn đến sự không nhất quán, như `dirty read`, `non-repeatable read` và `phantom read`.

`Dirty read` xảy ra khi một transaction nhìn thấy dữ liệu đã bị thay đổi bởi một transaction khác mà chưa được commit. Hãy tưởng tượng một tài khoản ngân hàng với `$100`. Transaction T1 rút `$20` nhưng không commit. Nếu Transaction T2 đọc số dư trước khi T1 commit, nó sẽ thấy `$80`. Nhưng nếu T1 rollback, số dư `$80` đó chưa bao giờ tồn tại - đó là `dirty read`.

{{<figure src="image8.png" width="500px" class="center">}}

Cấp độ isolation `read committed` ngăn chặn `dirty read` bằng cách đảm bảo một transaction chỉ có thể nhìn thấy dữ liệu đã commit.

{{<figure src="image9.png" width="500px" class="center">}}

Nhưng vẫn có thể có `non-repeatable read`, khi một transaction đọc dữ liệu hai lần và nhận kết quả khác nhau vì một transaction khác đã thay đổi dữ liệu giữa hai lần đọc. Ví dụ, bạn kiểm tra số dư tài khoản và thấy `$100`. Sau đó, một transaction khác rút `$50` và commit. Nếu bạn kiểm tra số dư một lần nữa trong cùng một transaction, bạn sẽ thấy `$50`. Đó là `non-repeatable read`.

{{<figure src="image10.png" width="500px" class="center">}}

`Read committed` cũng có thể xảy ra `phantom read`, khi một transaction chạy lại một truy vấn và nhận kết quả khác vì một transaction khác đã thêm hoặc xóa row đó đi. Hãy tưởng tượng một transaction liệt kê tất cả các transaction chuyển khoản dưới `$100`. Trong khi đó, một transaction khác thêm một giao dịch `$50` và commit. Nếu transaction đầu chạy lại truy vấn của mình, nó sẽ thấy giao dịch `$50` mà trước đó không có - đó là `phantom read`.

{{<figure src="image11.png" width="500px" class="center">}}

Cấp độ isolation `repeatable read` ngăn chặn `non-repeatable read` bằng cách cung cấp một snapshot nhất quán của dữ liệu cho mỗi transaction. Nhưng vẫn có thể có `phantom read`. Vì vậy, các cấp độ isolation thấp hơn hi sinh tính nhất quán để có hiệu suất tốt hơn. Bạn cần chọn sự cân bằng phù hợp cho ứng dụng của mình, cân nhắc giữa tốc độ và sự thiếu nhất quán.

{{<figure src="image12.png" width="500px" class="center">}}

## Durability
Khi một transaction được commit, dữ liệu sẽ tồn tại ngay cả khi hệ thống gặp sự cố. Tính bền vững thường được thực hiện thông qua việc ghi transaction logs hoặc sử dụng write-ahead logging (WAL) để lưu thay đổi vào đĩa trước khi xác nhận commit.

{{<figure src="image13.png" width="600px" class="center">}}

Trong database phân tán, tính bền vững cũng bao gồm việc sao chép dữ liệu qua nhiều node. Vì vậy, nếu một node gặp sự cố, bạn không mất bất kỳ transaction nào đã commit - chúng được lưu trữ an toàn trên các node khác.

{{<figure src="image14.png" width="400px" class="center">}}
