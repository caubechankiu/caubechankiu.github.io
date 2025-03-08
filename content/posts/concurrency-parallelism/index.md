+++
date = '2020-01-20T09:47:31+07:00'
draft = false
title = 'Concurrency vs Parallelism'
summary = 'Mô hình lập trình đồng thời và lập trình song song'
tags = ['multithreading']
categories = []
+++

Ban đầu, CPU chỉ có một nhân duy nhất, vì vậy các ngôn ngữ lập trình khi đó chủ yếu tuân theo mô hình lập trình tuần tự. Ngày nay, với sự phát triển của công nghệ đa nhân và đa xử lý, để khai thác tối đa hiệu suất của CPU, các mô hình lập trình song song và đa luồng (multi-threading) đã ra đời và trở nên phổ biến trong nhiều ngôn ngữ lập trình.

## What is Concurrency?

Xử lý đồng thời là khả năng phân chia và điều phối nhiều tác vụ trong cùng một khoảng thời gian, nhưng tại mỗi thời điểm, chỉ một tác vụ được thực thi. Khái niệm này đối lập với xử lý tuần tự (sequential processing), trong đó các tác vụ được thực hiện lần lượt, hoàn thành tác vụ này mới bắt đầu tác vụ tiếp theo.

Tất cả các chương trình đang chạy trên máy tính đều do hệ điều hành quản lý. Mỗi chương trình đang thực thi được gọi là một tiến trình (process) và được cấp một định danh tiến trình (process ID - PID) để hệ điều hành dễ dàng quản lý. Các tác vụ trong tiến trình sẽ do nhân CPU (CPU core) xử lý. Vậy làm thế nào một máy tính chỉ có CPU đơn nhân vẫn có thể xử lý đồng thời nhiều tác vụ của các tiến trình? Thực tế, tại mỗi thời điểm, nhân CPU chỉ có thể xử lý một tác vụ, nhưng hệ điều hành sử dụng cơ chế **chuyển ngữ cảnh (context switching)** để luân phiên thực thi các tiến trình, tạo ra ảo giác xử lý đồng thời.

{{<figure src="./concurrency.png" class="center" title="Concurrency">}}

{{<figure src="./sequential.png" class="center" title="Sequential">}}

## What is Parallelism?

Xử lý song song cho phép nhiều tác vụ được thực thi đồng thời tại cùng một thời điểm, với mỗi tác vụ chạy độc lập trên các nhân CPU khác nhau. Điều này chỉ khả thi trên các hệ thống có nhiều hơn một nhân CPU. Khi đó, thay vì một nhân CPU chỉ xử lý một tác vụ tại một thời điểm, nhiều nhân có thể hoạt động song song, giúp tăng tốc độ xử lý và cải thiện hiệu suất hệ thống.

{{<figure src="./parallelism.png" class="center" title="Parallelism">}}

Trên thực tế, một nhân CPU có thể xen kẽ thực thi nhiều tác vụ theo mô hình xử lý đồng thời, trong khi nhiều nhân CPU có thể chạy song song các tác vụ khác nhau. Sự kết hợp giữa xử lý đồng thời và xử lý song song giúp tối ưu hóa hiệu suất, đảm bảo rằng CPU luôn được tận dụng hiệu quả mà không có hai nhân xử lý cùng một tác vụ tại cùng một thời điểm.

{{<figure src="./parallelism-and-concurrency.png" class="center" title="Parallelism + Concurrency">}}

