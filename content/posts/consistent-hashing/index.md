+++
date = '2020-04-01T10:43:32+07:00'
draft = false
title = 'Consistent Hashing'
summary = 'Trong các hệ thống phân tán như caching (Redis, Memcached), load balancing, distributed databases (Cassandra, DynamoDB), việc phân phối dữ liệu một cách hiệu quả giữa các node là một bài toán quan trọng.'
tags = ['distributed system', 'load balancer']
categories = []
+++

## 1. Giới thiệu
Trong các hệ thống phân tán như **caching (Redis, Memcached), load balancing, distributed databases (Cassandra, DynamoDB)**, việc phân phối dữ liệu một cách hiệu quả giữa các node là một bài toán quan trọng.

Nếu sử dụng phương pháp **modular hashing**, việc thêm hoặc xóa một node sẽ làm thay đổi hoàn toàn cách dữ liệu được phân phối, gây ra **mất cân bằng tải** và **di chuyển dữ liệu lớn**. **Consistent Hashing** là một thuật toán giúp giải quyết vấn đề này một cách hiệu quả.

---

## 2. Modular Hashing
### 2.1. Modular Hashing là gì?
**Modular Hashing** là một phương pháp đơn giản để ánh xạ dữ liệu vào các node trong hệ thống. Công thức:

`Node = hash(key) % Số lượng node`

Ví dụ: Nếu ta có 4 node (`N0`, `N1`, `N2`, `N3`) và dữ liệu là các user như sau
- Công thức: `Node = hash(key) % 4`

| N0 | N1 | N2 | N3 |
|--------|--------|--------|--------|
| User 0 | User 1 | User 2 | User 3 |
| User 4 | User 5 | User 6 | User 7 |


### 2.2. Nhược điểm của Modular Hashing
✅ **Ưu điểm**: Dễ triển khai, tính toán nhanh.  
❌ **Nhược điểm**: Khi thêm hoặc xóa một node, **hầu hết dữ liệu phải di chuyển**, gây ảnh hưởng lớn đến hệ thống.

**Ví dụ khi giảm bớt 1 node `N3`:**
- Công thức mới: `Node = hash(key) % 3`
- Kết quả: Các key phải ánh xạ lại do kết quả phép chia thay đổi như sau

| N0 | N1 | N2 | ~~N3~~ |
|--------|--------|--------|--------|
| User 0 | User 1 | User 2 |        |
| User 3 | User 4 | User 5 |        |
| User 6 | User 7 |        |        |

Như vậy chỉ số ít dữ liệu giữ nguyên còn phần lớn sẽ bị di chuyển

➡ **Đây là vấn đề lớn khi hệ thống cần mở rộng hoặc thu nhỏ.**

## 3. Consistent Hashing là gì?
**Consistent Hashing** là một thuật toán giúp phân phối dữ liệu vào các node trong hệ thống sao cho **khi thêm hoặc xóa một node, chỉ một phần nhỏ dữ liệu cần di chuyển** (trung bình là k/n với k là tổng số lượng key, n là tổng số lượng node).

### 3.1. Cách hoạt động
Consistent Hashing hoạt động bằng cách ánh xạ các node trong cluster lên một vòng tròn (gọi là hash ring), trong đó một hàm hash được sử dụng để chuyển đổi các key thành số nguyên nằm trong một phạm vi nhất định. Các số nguyên này sau đó được đặt trên vòng tròn sao cho mỗi vị trí trên vòng tương ứng với một giá trị trong dãy số nguyên đó.

#### 3.1.1. Build Hash Ring

Thực hiện hash từng node trong cluster thành một số nguyên nằm trong một phạm vi số được xác định trước. Phạm vi này được thiết kế dựa trên sự cân nhắc của kiến trúc sư hệ thống, tùy thuộc vào số lượng server tối đa mà hệ thống có thể mở rộng.

{{<figure src="./image1.png" width="500px" class="center">}}

Các node s0, s1, s2, s3 được đặt trên vòng tròn như hình trên, lưu ý trên thực tế thì các node sẽ không phân bố đều trên vòng tròn như vậy.

#### 3.1.2. Lưu dữ liệu vào node

Khi một key (dữ liệu) cần được lưu, hệ thống tính giá trị hash của key và đặt nó lên vòng tròn. Dữ liệu sẽ được gán cho **node gần nhất theo chiều kim đồng hồ**.

{{<figure src="./image2.png" width="500px" class="center">}}

Dữ liệu k0 được lưu ở s0 và tương tự k1 -> s1, k2 -> s2, k3 -> s3

#### 3.1.3. Xử lý khi thêm / xóa node

- **Thêm node**: Chỉ dữ liệu thuộc về node kế tiếp (theo chiều kim đồng hồ) bị ảnh hưởng.

{{<figure src="./image3.png" width="500px" class="center">}}

Khi ta thêm node s4, chỉ những dữ liệu nằm giữa s3 và s4 trước đây được lưu ở s0 thì sẽ cần chuyển sang s4, còn lại các dữ liệu khác thì được giữ nguyên

- **Xóa node**: Dữ liệu của node bị xóa sẽ được chuyển sang node kế tiếp.

{{<figure src="./image4.png" width="500px" class="center">}}

Khi node s1 bị xoá, chỉ những dữ liệu nằm giữa s0 và s1 trước đây được lưu ở s1 thì sẽ cần chuyển sang s2, còn lại các dữ liệu khác thì được giữ nguyên

### 3.2 Virtual Node (Vnode)
Trên thực tế trong đa số trường hợp, các node sẽ không phân bố đều trên vòng tròn mà sẽ phân bố không đồng đều như hình dưới đây.

{{<figure src="./image5.png" width="500px" class="center">}}

Trong trường hợp này, dữ liệu sẽ không được phân bố đồng đều lên các node mà node s2 sẽ chứa nhiều dữ liệu nhất, do đó nó cũng sẽ phải gánh nhiều tải hơn so với các node còn lại

Để giải quyết vấn đề này, chúng ta sẽ thực hiện hash và thêm nhiều các Node ảo (virtual node) vào vòng tròn một cách ngẫu nhiên. Mỗi node ảo sẽ liên kết đến 1 node thật. Số lượng node ảo bao nhiêu cũng cần cân nhắc, phụ thuộc các yếu tố sau:
- Số vnode càng nhiều, dữ liệu càng được phân phối đều hơn, tránh tình trạng một node phải xử lý nhiều hơn node khác.
- Tuy nhiên, quá nhiều vnode có thể gây quá tải cho thuật toán tìm kiếm node, ảnh hưởng đến hiệu suất hệ thống.
- Quy tắc chung có thể áp dụng
    - Ít hơn 10 node thật → 100 - 500 vnode mỗi node thật.
    - 10 - 100 node thật → 50 - 200 vnode mỗi node thật.
    - Hơn 100 node thật → 10 - 50 vnode mỗi node thật.

{{<figure src="./image6.png" width="500px" class="center">}}

Ví dụ này ta có 3 node thật, mỗi node thật có thêm 2 node ảo. Như vậy data đã được phân bố đều hơn trên 2 node thật.

### 3.3 Ứng dụng thực tế của Consistent Hashing
- Distributed Caching (Memcached, Redis Cluster)
    - Khi có nhiều node cache (Redis, Memcached), Consistent Hashing giúp ánh xạ key vào node cache tương ứng.
    - Khi thêm/xóa node cache, chỉ một phần nhỏ dữ liệu bị ảnh hưởng, tránh cache miss hàng loạt.
- Distributed Databases (Cassandra, DynamoDB)
    - Trong các database phân tán (NoSQL như DynamoDB, Cassandra), Consistent Hashing giúp định tuyến dữ liệu vào đúng shard.
    - Khi thêm/xóa node, chỉ một phần dữ liệu được di chuyển.