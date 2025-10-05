---
date: 2023-07-15T10:00:00+07:00
draft: false
title: "How to store passwords in the database?"
summary: 'Việc lưu trữ mật khẩu một cách an toàn trong cơ sở dữ liệu là một vấn đề quan trọng trong thiết kế hệ thống. Làm thế nào để chúng ta xác thực mật khẩu người dùng nhập vào so với những gì chúng ta lưu trữ trong cơ sở dữ liệu?'
tags: ["security"]
---

Việc lưu trữ mật khẩu một cách an toàn trong cơ sở dữ liệu là một vấn đề quan trọng trong thiết kế hệ thống. Làm thế nào để chúng ta xác thực mật khẩu người dùng nhập vào so với những gì chúng ta lưu trữ trong cơ sở dữ liệu?

## Điều không nên làm

Trước tiên, hãy nói về những điều không nên làm. **Không bao giờ lưu trữ mật khẩu dưới dạng văn bản thuần (plain text)**. Nếu làm vậy, bất kỳ ai có quyền truy cập nội bộ vào cơ sở dữ liệu đều có thể nhìn thấy chúng. Nếu cơ sở dữ liệu bị xâm phạm, kẻ tấn công có thể dễ dàng lấy được tất cả mật khẩu.

## Hướng dẫn từ OWASP

OWASP - Open Web Application Security Project đã cung cấp một số hướng dẫn về cách lưu trữ mật khẩu. Dưới đây là những khuyến nghị chính:

### 1. Sử dụng thuật toán băm (hashing) hiện đại

Băm là một hàm một chiều. Điều này có nghĩa là không thể giải mã một giá trị đã băm để lấy lại giá trị ban đầu. Nếu kẻ tấn công lấy được mật khẩu đã băm, họ không thể chỉ nhập nó vào ứng dụng để có quyền truy cập.
{{<figure src="./image1.png" width="500px" class="center">}}

Điều quan trọng là sử dụng một hàm băm hiện đại được thiết kế để lưu trữ mật khẩu một cách an toàn. Những hàm này đôi khi được đặc trưng là các hàm "chậm" sử dụng nhiều tài nguyên hơn để tính toán. Điều này làm cho các cuộc tấn công vét cạn (brute force) trở nên kém hấp dẫn.

Lưu ý rằng một số hàm băm phổ biến cũ như MD5 và SHA-1 là "nhanh". Chúng kém an toàn hơn và không nên được sử dụng.

### 2. Thêm muối (salt) vào mật khẩu

Theo hướng dẫn của OWASP, muối là một chuỗi ngẫu nhiên độc nhất được thêm vào mỗi mật khẩu như một phần của quá trình băm.

Tại sao chúng ta cần thêm muối vào mật khẩu? Việc lưu trữ mật khẩu dưới dạng băm một chiều là một bước đi đúng hướng, nhưng chưa đủ. Kẻ tấn công có thể đánh bại các hàm băm một chiều với các cuộc tấn công tính toán trước. Một số kỹ thuật tấn công phổ biến là bảng cầu vồng (rainbow tables) và tra cứu dựa trên cơ sở dữ liệu. Với những kỹ thuật này, một hacker có thể phá vỡ mật khẩu trong vài giây.

Bằng cách thêm muối độc nhất cho mỗi mật khẩu vào quá trình băm, nó đảm bảo rằng giá trị băm là độc nhất cho mỗi mật khẩu. Kỹ thuật đơn giản này làm cho các cuộc tấn công tính toán trước trở nên kém hấp dẫn.
{{<figure src="./image2.png" width="500px" class="center">}}

#### Quy trình lưu trữ mật khẩu với muối

Hãy xem xét cách chúng ta lưu trữ mật khẩu với muối:

1. Đầu tiên, chúng ta kết hợp mật khẩu do người dùng cung cấp với muối được tạo ngẫu nhiên.
2. Sau đó, chúng ta tính toán giá trị băm của sự kết hợp này bằng cách sử dụng một hàm băm thích hợp.
3. Giá trị băm được lưu trữ trong cơ sở dữ liệu cùng với muối.

{{<figure src="./image3.png" width="500px" class="center">}}

Muối được sử dụng để tạo ra một giá trị băm độc nhất. Nó không phải là bí mật và có thể được lưu trữ an toàn dưới dạng văn bản thuần trong cơ sở dữ liệu.

#### Quy trình xác thực mật khẩu

Cuối cùng, hãy xem xét quy trình ngược lại. Khi người dùng đăng nhập, làm thế nào chúng ta xác thực mật khẩu so với những gì được lưu trữ trong cơ sở dữ liệu?

1. Đầu tiên, chúng ta lấy muối cho người dùng từ cơ sở dữ liệu.
2. Sau đó, chúng ta thêm muối vào mật khẩu do người dùng cung cấp và băm nó.
3. Chúng ta so sánh giá trị băm được tính toán này với giá trị băm được lưu trữ trong cơ sở dữ liệu.
4. Nếu chúng giống nhau, mật khẩu là hợp lệ.

{{<figure src="./image4.png" width="500px" class="center">}}

#### Migrate
Làm thế nào migrate một hệ thống dùng SHA1 –> bcrypt. Ở đây có 2 trường hợp:
- Hash password bằng SHA1 nhưng không có salt -> mình gọi là sha1_value. Với trường hợp này ta cần sinh ra thêm salt-per-user và migrate bcrypt(salt, sha1_value) trong đó sha1_value = SHA1(password)
- Hash password bằng SHA1 + trong DB có salt-per-user, mình có sha1_value, salt. Với trường hợp này thì hơi lằng nhằng hơn 1 xíu, ta cần migrate kiểu bcrypt(salt, SHA1(salt, password)) nếu thực sự muốn bcrypt với salt hoặc thực ra chỉ cần bcrypt(SHA1(salt, password)) cũng được. Tùy tình huống bạn cần tradeoff có thể lựa chọn cách phù hợp.
