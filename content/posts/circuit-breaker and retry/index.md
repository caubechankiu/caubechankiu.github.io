+++
date = '2021-02-20T09:47:31+07:00'
draft = false
title = 'Circuit Breaker and Retry'
summary = 'Circuit breaker và retry là hai cơ chế quan trọng giúp tăng cường độ tin cậy và khả năng chịu lỗi của hệ thống.'
tags = ['system design', 'distributed system']
categories = []
+++

Hệ thống phân tán bao gồm nhiều thành phần hoạt động trên các máy chủ khác nhau, các thành phần này có thể bị lỗi do nhiều nguyên nhân như:
- Các vấn đề về mạng (mất kết nối, độ trễ cao, gói tin bị thất lạc)
- Lỗi phần mềm (bug, tràn bộ nhớ, lỗi deadlock)
- Lỗi phần cứng (server hỏng, mất điện, lỗi đĩa cứng, hết bộ nhớ)

Khi lỗi xảy ra trên bất kì thành phần nào, chúng ta nên cố gắng xử lý và ngăn không cho lỗi lan rộng ảnh hướng đến toàn hệ thống và đến trải nghiệm của user

## Circuit Breaker

Circuit Breaker có nghĩa là cầu dao, nó hoạt động giống như ý tưởng về cầu dao điện trong thực tế. Trong lĩnh vực phần mềm, nó là một đoạn code được thêm vào để theo dõi và giám sát tình trạng của các request đến các service khác.

Một request được xử lý thành công sẽ như thế này
{{<figure src="image1.png" width="600px" class="center">}}

Khi **My Service** gọi đến **Other Service**, request sẽ đi qua **Circuit Breaker** (cũng nằm trong code của **My Service**). Sau đó **Other Service** xử lý request và response kết quả, kết quả sau đó được trả về cho user.

Khi **Other Service** bị lỗi
{{<figure src="image2.png" width="600px" class="center">}}
Nếu chỉ như vậy thì **Circuit Breaker** thêm vào chưa có tác dụng gì cả.

Tuy nhiên, hãy giả sử rằng tất cả các request trong 3 giây qua đều bị lỗi. **Circuit Breaker** đã theo dõi các request này. Nó nhận thấy rằng tất cả các request đều bị lỗi, vì vậy thay vì thực hiện thêm bất kỳ request nào nữa, nó sẽ mở mạch (trạng thái `OPEN`), ngăn không cho bất kỳ request nào được thực hiện thêm. Các request sau đó khi mạch đã mở sẽ như thế này
{{<figure src="image3.png" width="600px" class="center">}}

Các service có thể bị lỗi khi chúng bị quá tải với các request. Khi một service bị quá tải, việc thực hiện bất kỳ request nào khác có thể dẫn đến
- Việc thực hiện request có thể vô nghĩa, vì chúng ta sẽ không nhận được response hợp lệ và/hoặc kịp thời.
- Bằng cách gửi thêm request, chúng ta rất có thể sẽ làm quá tải service đó nhiều hơn.

### Fallback
Khi không nhận được response hợp lệ, ta có thể cài đặt phương án dự phòng (fallback). Nó trông như thế này
{{<figure src="image4.png" width="600px" class="center">}}

Ví dụ bạn có một hệ thống thương mại điện tử sử dụng AI recommendation system để đề xuất sản phẩm phù hợp cho user. Hệ thống này gọi một service bên ngoài (hoặc một mô hình AI nội bộ) để lấy danh sách sản phẩm gợi ý dựa trên hành vi mua hàng. Tuy nhiên, nếu hệ thống đề xuất bị lỗi hoặc mất kết nối, thay vì trả về danh sách rỗng, hệ thống có thể đề xuất
- Trả về danh sách sản phẩm bán chạy nhất trong danh mục mà user quan tâm.
- Trả về các sản phẩm cùng trong danh mục mà user đã từng xem hoặc mua trước đó.

Việc tự tính toán danh sách đề xuất sản phẩm như trên trông có vẻ không được thông minh lắm, nhưng nó giúp chúng ta tiếp tục xử lý request của user thì tốt hơn nhiều so với việc trả về lỗi.

Tất nhiên, có những trường hợp không có giải pháp dự phòng hợp lý. Nhưng ngay cả trong những tình huống này, việc sử dụng **Circuit Breaker** vẫn có lợi.

### **Circuit Breaker** nên theo dõi những lỗi như thế nào?
**Circuit Breaker** chỉ nên theo dõi các lỗi không phải do user gây ra (mã lỗi HTTP 4xx), mà do mạng hoặc cơ sở hạ tầng (mã lỗi HTTP 5xx). Nếu theo dõi cả các lỗi do user gây ra, thì một user có thể gửi một số lượng lớn request không hợp lệ, khiến mạch bị mở và gây gián đoạn dịch vụ cho tất cả các user khác.

### Circuit Recovery
Vậy khi nào thì mạch sẽ được đóng lại để cho phép các request được đi qua. Sau khi mạch mở, nó sẽ đợi trong một khoảng thời gian (có thể cấu hình được), được gọi là `Sleep Window`. Sau đó nó sẽ chuyển sang trạng thái nửa mở (`HAFL OPEN`), lúc này nó sẽ kiểm tra bằng cách cho phép một số request đi qua. Nếu service đã phục hồi, nó sẽ đóng mạch (trạng thái `CLOSED`) và tiếp tục hoạt động bình thường. Nếu các request vẫn trả về lỗi, thì nó sẽ trở về trạng thái `OPEN` và đợi 1 lần `Sleep Window` nữa
{{<figure src="image5.png" width="600px" class="center">}}

### Circuit Breaker Settings
- **Failure Rate Threshold**: Ngưỡng tỉ lệ lỗi trong một khoảng thời gian dẫn đến mở mạch: (VD: nếu > 50% request lỗi trong vòng 10 giây thì mạch sẽ mở).
- **Sleep Window**: Sau một khoảng thời gian, **Circuit Breaker** sẽ chuyển sang trạng thái `HAFL OPEN` và cho phép một số request thử nghiệm đi qua. Nếu các request thử nghiệm thành công, **Circuit Breaker** quay lại trạng thái `CLOSED`.
- **Timeout Duration**: Thời gian tối đa chờ response trước khi coi là lỗi
- **Fallback Processing**: Cài đặt phương án dự phòng để tránh trả về lỗi, làm giảm trải nghiệm của user.

## Retry
Retry là một cơ chế theo dõi request và tự động thử lại request đó nếu phát hiện lỗi.

Như chúng ta đã biết, các service thường được chạy trên nhiều instance và đứng sau một load balancer để điều phối các request đến nó. Giả sử có một instance trong số đó bị lỗi, và request của chúng ta không may lại được chuyển hướng đến đúng instance bị hỏng đó. Trong trường hợp này, nếu chúng ta thử lại và may mắn hơn, request thử lại có thể được chuyển hướng đến instance khác vẫn còn hoạt động và request của user sẽ được xử lý thành công thay vì trả về lỗi cho lần gọi đầu tiên mà không có thử lại.
{{<figure src="image6.png" width="600px" class="center">}}

### Nên thử lại với những lỗi nào?
Chúng ta nên cân nhắc thử lại chỉ khi có cơ hội thành công (mã lỗi HTTP 5xx). Ví dụ, đối với mã lỗi 503, việc thử lại có thể tốt nếu việc thử lại dẫn đến một request đến instance không bị quá tải. Ngược lại, đối với các lỗi như 4xx, việc thử lại các lỗi này sẽ lãng phí tài nguyên vì chúng sẽ không bao giờ hoạt động nếu user không thay đổi request của họ.

### Idempotency
Một quy trình được coi là có tính chất `idempotent` nếu nó có thể được lặp lại dù là bao nhiêu lần vẫn không gây ra các tác dụng phụ không mong muốn.

Chúng ta cùng xem xét trường hợp sau. Trong hệ thống đặt vé máy bay, khi user gửi request yêu cầu đặt một vé. Nếu request đầu tiên bị lỗi nhưng thực tế request đó vẫn được xử lý, chỉ là server không response đúng cách (có thể là timeout). Nếu chúng ta thực hiện thêm một request thử lại thì chúng ta đã đặt 2 vé trong khi thực tế user chỉ muốn đặt 1 vé.

Vấn đề ở đây là API đặt vé này không có tính chất `idempotent`. Trong trường hợp này, tốt hơn là để hệ thống ở trạng thái không xác định và biến nó thành vấn đề của user. User sẽ phải hiểu là không rõ vé mình đã đặt được chưa, họ có thể sẽ đợi 1 khoảng thời gian để xem trong lịch sử đặt vé có xuất hiện vé vừa đặt không và đi đến quyết định có đặt lại hay không.

Trong trường hợp vẫn muốn thử lại, chúng ta cần đảm bảo API đặt vé phải có tính chất `idempotent`. Vấn đề này sẽ được trình bày kĩ hơn ở một bài viết khác.

### Backoff
Khi một request bị lỗi, chúng ta có nên thử lại nó ngay lập tức không. Câu trả lời là không. Ví dụ service đang bị quá tải thì nếu thử lại ngay lập tức, khả năng request vẫn bị lỗi là khá cao, không những thế còn gây thêm quá tải cho service đó. Chiến lược ở đây là chúng ta sẽ đợi một khoảng thời gian (gọi là `backoff`) trước khi thử lại lần tiếp theo. Trường hợp backoff delay là 100ms

| Retry Attempt	| Delay                |
|---------------|----------------------|
| 1	            | 1 x 100ms = 100ms    |
| 2	            | 2 x 100ms = 200ms    |
| 5	            | 5 x 100ms = 500ms    |
| 10	          | 10 x 100ms = 1,000ms |

Lý thuyết cơ bản ở đây là nếu một request đã thất bại một vài lần, thì khả năng nó sẽ thất bại lần nữa sẽ cao hơn. Do đó, chúng ta nên chờ lâu hơn để service có cơ hội phục hồi.

### Jitter
Giả sử chúng ta gửi 10.000 request và tất cả đều thất bại vì service không thể xử lý lượng request đồng thời lớn như vậy. Theo cơ chế backoff đơn giản đã đề cập trước đó, sau khi trì hoãn 100ms, chúng ta sẽ thử lại toàn bộ 10.000 request, và chúng cũng sẽ thất bại vì cùng một lý do. 

Để tránh tình trạng này, cơ chế retry sẽ bao gồm `jitter`. `Jitter` là quá trình tăng hoặc giảm thời gian trì hoãn so với mức tiêu chuẩn để phân tán tải tốt hơn. Trong ví dụ trên, 10.000 request sẽ bị trì hoãn ngẫu nhiên trong khoảng từ 70-150ms (cho lần thử lại đầu tiên). Mục tiêu của cách tiếp cận này cũng giống như trên, đó là làm phân tán các request tránh gây quá tải cho service.

### Retry Settings
- **Timeout Duration**: Thời gian tối đa chờ response trước khi coi là lỗi
- **Maximum Retries**: Giá trị này cho biết một request có thể được thử lại bao nhiêu lần trước khi từ bỏ (thất bại).
- **Retry Filter**: Đây là hàm xử lý lỗi trả về và quyết định xem có nên thử lại hay không.
- **Base and Max Delay**: Thời gian chờ tối thiểu và tối đã giữa các lần thử lại (khi đã kết hợp `Backoff` và `Jitter`)

Khi điều chỉnh các giá trị trên sẽ ảnh hưởng đến thời gian chờ của user. Đây là yếu tố cần cân nhắc khi thực hiện điều chỉnh các giá trị cho phù hợp
