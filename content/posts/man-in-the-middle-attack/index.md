---
title: "Man-in-the-Middle (MITM) Attack"
date: 2024-11-05
description: "Tấn công MITM là một mối đe dọa đáng lo ngại khi sử dụng internet công cộng. Bài viết này sẽ giúp bạn hiểu rõ về MITM và cách phòng tránh nó."
tags: ["Security"]
---

# Man-in-the-Middle (MITM): Kẻ Nghe Trộm Vô Hình Đang Rình Rập Dữ Liệu Của Bạn

Nếu bạn đã từng kết nối Wi-Fi ở quán cà phê, sân bay, hoặc bất kỳ mạng công cộng nào, bạn đã tự đặt mình vào tầm ngắm của một trong những mối đe dọa dai dẳng nhất trên internet: **Tấn công Man-in-the-Middle (MITM)**.

Hãy tưởng tượng bạn đang thì thầm một mật khẩu với ngân hàng của mình. Trong một cuộc tấn công MITM, một kẻ lạ mặt đang đứng chính giữa, nghe lén mọi thứ, và thậm chí còn có thể thay đổi tin nhắn của bạn trước khi nó đến đích. Mối đe dọa này không hề xa vời, và hiểu rõ về nó là bước đầu tiên để tự bảo vệ mình.

---

## MITM là gì và Nó Hoạt động như thế nào?

**Man-in-the-Middle** là một hình thức tấn công mạng mà kẻ xấu bí mật xen vào giữa hai bên đang giao tiếp, sau đó giả mạo cả hai bên để chặn, đánh cắp hoặc thay đổi dữ liệu được truyền đi. Cả hai nạn nhân đều tin rằng họ đang giao tiếp trực tiếp với nhau, trong khi thực tế họ đang "nói chuyện" với kẻ tấn công.

{{<figure src="man-in-middle-attack.png" width="500px" class="center">}}

Cơ chế tấn công MITM diễn ra qua ba giai đoạn cơ bản:

### 1. Chặn (Interception)

Bước đầu tiên là đưa kẻ tấn công vào giữa luồng giao tiếp. Kẻ tấn công sử dụng các kỹ thuật để đánh lừa mạng lưới hoặc thiết bị của bạn.

* **ARP Spoofing:** Trong mạng cục bộ (như Wi-Fi gia đình, Wi-Fi công cộng), kẻ tấn công gửi các gói tin ARP giả mạo đến cả máy nạn nhân và router, thông báo rằng địa chỉ MAC của chúng là địa chỉ của router (với nạn nhân) và là địa chỉ của nạn nhân (với router). Lúc này, mọi gói tin mà nạn nhân gửi ra ngoài (bao gồm cả yêu cầu DNS) đều bị chuyển hướng đến máy của kẻ tấn công trước. Đồng thời tất cả các gói tin đến nạn nhân cũng phải đi qua máy của kẻ tấn công trước. Lúc này, máy của kẻ tấn công đã trở thành "*người đàn ông ở giữa*".
* **Evil Twin (Wi-Fi giả mạo):** Kẻ tấn công tạo ra một điểm truy cập Wi-Fi giả mạo có tên tương tự như mạng công cộng (ví dụ: "Free_Airport_WIFI"). Bất kỳ ai kết nối vào mạng này đều bị kiểm soát.

### 2. Giải mã và Đánh cắp (Decryption & Impersonation)

Khi lưu lượng truy cập đã bị chặn, thách thức tiếp theo là đọc được nó, đặc biệt nếu nó được **mã hóa**.

* **SSL Stripping (Hạ cấp SSL):** Kỹ thuật nguy hiểm này chặn yêu cầu kết nối HTTPS an toàn của bạn tới một trang web (ví dụ: ngân hàng). Thay vì để bạn kết nối an toàn, kẻ tấn công tự kết nối với trang web bằng HTTPS, rồi buộc bạn kết nối với chúng bằng HTTP (không an toàn). Bạn thấy icon https hình ổ khóa đã biến mất, nhưng có thể không kịp nhận ra.
* **Giả mạo DNS (DNS Spoofing):** Giao thức DNS thông thường hoạt động trên UDP và **KHÔNG được mã hóa**. Khi nạn nhân muốn truy cập *www.bank.com*, máy nạn nhân gửi yêu cầu phân giải tên miền (DNS Query) đến DNS Server (theo như nó nghĩ là Router, nhưng thực chất là máy của Kẻ tấn công). Ngay khi nhận được yêu cầu DNS từ Nạn nhân, kẻ tấn công sẽ tạo một gói tin Phản hồi DNS (DNS Response) giả mạo. Trong gói tin này, kẻ tấn công sẽ cung cấp địa chỉ IP của máy chủ giả mạo thay vì địa chỉ IP thực tế của *www.bank.com*. Máy nạn nhân chấp nhận phản hồi này là hợp lệ và lưu trữ bản ghi ánh xạ tên miền giả mạo vào bộ nhớ đệm DNS cục bộ (DNS Cache). Kết quả cuối cùng là khi nạn nhân cố gắng truy cập *www.bank.com*, trình duyệt sẽ dùng địa chỉ IP đã lưu trong cache và sẽ chuyển hướng đến một trang web lừa đảo hoàn toàn do chúng kiểm soát. Để thực sự ngăn chặn việc nghe lén và giả mạo ngay tại tầng DNS, cần sử dụng các giao thức mới giúp mã hóa truy vấn DNS đến máy chủ DNS công cộng (ví dụ: Cloudflare, Google DNS. Router lúc này không còn được xem là DNS Server nữa mà chỉ là một thiết bị chuyển tiếp):
  * **DNS over HTTPS (DoH)**: Mã hóa truy vấn DNS và phản hồi bên trong kết nối HTTPS (sử dụng cổng TCP 443).
  * **DNS over TLS (DoT)**: Mã hóa truy vấn DNS và phản hồi bên trong kết nối TLS (sử dụng cổng TCP 853).

### 3. Truyền tải (Relaying)

Sau khi đọc hoặc đánh cắp dữ liệu (mật khẩu, số thẻ), kẻ tấn công mã hóa lại dữ liệu nếu cần và gửi tiếp cho đích đến ban đầu. Toàn bộ quá trình này diễn ra trong tích tắc, khiến cuộc tấn công gần như vô hình đối với người dùng cuối.

---

## Hậu Quả Khủng Khiếp của MITM

Thiệt hại do MITM gây ra thường rất nghiêm trọng vì nó nhắm vào **lớp tin cậy** cơ bản trong giao tiếp mạng:

* **Đánh cắp Thông tin Đăng nhập:** Đây là mục tiêu phổ biến nhất, bao gồm mật khẩu, tên người dùng, và mã PIN cho các dịch vụ tài chính.
* **Thiệt hại Tài chính:** Sử dụng thông tin thẻ tín dụng bị đánh cắp hoặc thay đổi thông tin giao dịch ngân hàng trực tuyến để chuyển tiền đến tài khoản của kẻ tấn công.
* **Lộ Bí mật Doanh nghiệp:** Nếu nhân viên bị tấn công khi đang làm việc từ xa, dữ liệu nhạy cảm của công ty, các tài liệu mật hoặc chiến lược kinh doanh có thể rơi vào tay kẻ xấu.
* **Thay đổi Dữ liệu:** Kẻ tấn công có thể thay đổi nội dung email, hợp đồng điện tử, hoặc hóa đơn, dẫn đến những tranh chấp pháp lý hoặc thiệt hại kinh tế lớn.

---

## Các Biện pháp Phòng Tránh MITM Hiệu Quả

Tin tốt là hầu hết các cuộc tấn công MITM có thể được ngăn chặn bằng các biện pháp bảo mật đơn giản nhưng hiệu quả.

### Đối với Cá nhân (Người dùng)

1.  **Luôn Kiểm tra HTTPS:**
    * **Quy tắc Vàng:** Đừng bao giờ nhập thông tin nhạy cảm (mật khẩu, thẻ) trên một trang web không hiển thị **biểu tượng ổ khóa** và tiền tố **$https://$** trong thanh địa chỉ.
    * Hãy cẩn thận với những trang web ban đầu là HTTPS nhưng đột nhiên chuyển thành HTTP khi bạn bắt đầu nhập dữ liệu.

{{<figure src="detecting-man-in-middle-attack.png" width="500px" class="center">}}

2.  **Sử dụng VPN (Mạng riêng ảo) trên Wi-Fi Công cộng:**
    * **VPN** là tuyến phòng thủ tốt nhất. Nó mã hóa toàn bộ dữ liệu của bạn trước khi nó rời khỏi thiết bị, khiến bất kỳ kẻ tấn công nào chặn được dữ liệu cũng chỉ thấy một "mớ bòng bong" vô nghĩa.

3.  **Cẩn thận với Wi-Fi Mở:**
    * Nếu bạn phải sử dụng Wi-Fi công cộng, hãy tránh xa các mạng yêu cầu ít hoặc không yêu cầu xác thực. Tuyệt đối **không thực hiện giao dịch ngân hàng** hay mua sắm trực tuyến.

4.  **Kích hoạt Xác thực Đa yếu tố (MFA/2FA):**
    * Ngay cả khi mật khẩu của bạn bị đánh cắp, kẻ tấn công vẫn cần một mã dùng một lần (thường từ điện thoại của bạn) để đăng nhập, làm vô hiệu hóa dữ liệu bị đánh cắp.

### Đối với Doanh nghiệp (Quản trị viên mạng)

1.  **Triển khai HSTS:**
    * **HTTP Strict Transport Security (HSTS)** buộc trình duyệt của người dùng chỉ kết nối với trang web bằng giao thức HTTPS, ngăn chặn hiệu quả các cuộc tấn công SSL Stripping.

2.  **Bảo vệ Cơ sở hạ tầng Mạng:**
    * Sử dụng các giao thức bảo vệ ARP như **Dynamic ARP Inspection (DAI)** và **DHCP Snooping** để ngăn chặn các cuộc tấn công ARP Spoofing từ bên trong mạng.

3.  **Giám sát Mạng (IDS/IPS):**
    * Sử dụng Hệ thống Phát hiện và Ngăn chặn Xâm nhập **(IDS/IPS)** để liên tục giám sát lưu lượng mạng và phát hiện các hoạt động đáng ngờ, bất thường (ví dụ: lưu lượng truy cập đột ngột định tuyến sai).

---

## Kết luận

Tấn công Man-in-the-Middle là một lời nhắc nhở rằng chúng ta không thể đơn thuần tin tưởng vào luồng dữ liệu của mình trên internet. Tuy nhiên, bằng cách tuân thủ các nguyên tắc bảo mật cơ bản như **kiểm tra HTTPS**, **sử dụng VPN**, và **kích hoạt 2FA**, bạn có thể giảm đáng kể rủi ro trở thành nạn nhân.

Hãy chủ động bảo vệ dữ liệu của mình. Bạn đã kiểm tra xem ứng dụng VPN của mình đã được bật chưa trước khi truy cập Wi-Fi công cộng lần tiếp theo?