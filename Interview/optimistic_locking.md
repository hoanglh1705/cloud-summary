**Optimistic Locking** là một kỹ thuật quản lý truy cập đồng thời được sử dụng phổ biến trong các hệ thống quản lý dữ liệu để ngăn ngừa vấn đề **overselling** (bán quá mức) trong các ứng dụng như thương mại điện tử. Dưới đây là cách hoạt động của Optimistic Locking trong việc ngăn chặn overselling:

### 1. **Vấn đề Overselling**
Overselling xảy ra khi có nhiều người dùng cùng mua sản phẩm trong một hệ thống bán hàng trực tuyến, và số lượng sản phẩm tồn kho không đủ đáp ứng cho tất cả các yêu cầu mua hàng, nhưng hệ thống vẫn cho phép nhiều hơn một người dùng cùng đặt mua sản phẩm, dẫn đến bán quá số lượng có sẵn.

Ví dụ, nếu chỉ còn 1 sản phẩm trong kho và hai khách hàng cùng đặt mua sản phẩm đó cùng lúc, hệ thống có thể xác nhận đơn hàng của cả hai người, dẫn đến bán quá mức.

### 2. **Nguyên tắc của Optimistic Locking**
Optimistic Locking cho phép nhiều người dùng truy cập và đọc dữ liệu mà không sử dụng khóa trực tiếp, nhưng khi ghi dữ liệu, hệ thống sẽ kiểm tra để đảm bảo dữ liệu không bị thay đổi bởi các transaction khác. Nếu phát hiện có sự thay đổi (xung đột), thao tác cập nhật sẽ bị từ chối và người dùng phải thử lại.

Optimistic Locking thường được triển khai với một **version field** (hoặc timestamp), để kiểm soát các phiên bản dữ liệu và đảm bảo tính toàn vẹn.

### 3. **Cách Thực Hiện Optimistic Locking để Ngăn Chặn Overselling**

#### Bước 1: **Đọc dữ liệu với version hiện tại**
- Khi một người dùng (User A) truy cập vào trang sản phẩm, hệ thống sẽ lấy thông tin sản phẩm bao gồm **số lượng tồn kho** và **version** hiện tại của sản phẩm đó từ cơ sở dữ liệu.
  - Ví dụ: User A lấy thông tin sản phẩm với số lượng tồn kho = 1 và version = 5.

#### Bước 2: **Kiểm tra tồn kho**
- Người dùng A tiến hành đặt mua sản phẩm. Ở bước này, ứng dụng sẽ kiểm tra tồn kho:
  - Nếu tồn kho lớn hơn 0, hệ thống sẽ tiến hành cập nhật số lượng.
  - Nếu tồn kho = 0, yêu cầu sẽ bị từ chối.

#### Bước 3: **Cập nhật dữ liệu với version cũ**
- Khi cập nhật số lượng tồn kho, hệ thống sẽ gửi một truy vấn cập nhật với điều kiện:
  - Số lượng tồn kho > 0.
  - `WHERE product_id = X AND version = 5` (version phải khớp với version mà User A đọc trước đó).
- Nếu phiên bản của dữ liệu (version) vẫn là 5, tức là không có ai khác đã thay đổi dữ liệu sản phẩm này trong thời gian giữa lúc User A đọc và cập nhật, hệ thống sẽ cho phép cập nhật:
  - Giảm tồn kho từ 1 xuống 0.
  - Tăng version từ 5 lên 6.

#### Bước 4: **Xử lý khi xảy ra xung đột**
- Nếu có một người dùng khác (User B) cũng truy cập cùng lúc và đặt mua sản phẩm:
  - User B cũng sẽ lấy được thông tin với số lượng tồn kho = 1 và version = 5.
  - Tuy nhiên, khi User B cố gắng cập nhật số lượng tồn kho sau khi User A đã hoàn thành việc mua hàng (và đã cập nhật version lên 6), yêu cầu cập nhật của User B sẽ thất bại vì version không khớp (User B dùng version 5, nhưng hiện tại là version 6).
  - Lúc này, User B sẽ nhận được thông báo rằng số lượng sản phẩm đã hết hoặc yêu cầu phải thử lại.

#### Bước 5: **Retry hoặc xử lý lỗi**
- Nếu yêu cầu của User B thất bại, hệ thống có thể xử lý lỗi bằng cách:
  - Thông báo cho User B rằng sản phẩm đã hết hàng.
  - Cho phép User B thử lại, hệ thống sẽ đọc dữ liệu mới nhất với version mới và kiểm tra tồn kho lại.

### 4. **Ví dụ Cụ Thể về Truy Vấn Cơ Sở Dữ Liệu**
Giả sử bạn có bảng `products` với các cột sau:
- `product_id`: ID của sản phẩm.
- `stock`: Số lượng sản phẩm tồn kho.
- `version`: Version hiện tại của bản ghi.

#### 1. User A đọc dữ liệu:
```sql
SELECT product_id, stock, version FROM products WHERE product_id = 1;
```
- Kết quả: `stock = 1`, `version = 5`.

#### 2. User A đặt mua sản phẩm:
```sql
UPDATE products
SET stock = stock - 1, version = version + 1
WHERE product_id = 1 AND stock > 0 AND version = 5;
```
- Nếu thành công, tồn kho giảm từ 1 xuống 0, version tăng từ 5 lên 6.

#### 3. User B đặt mua sản phẩm (cùng lúc với User A):
- Khi User B cố gắng thực hiện câu lệnh tương tự:
```sql
UPDATE products
SET stock = stock - 1, version = version + 1
WHERE product_id = 1 AND stock > 0 AND version = 5;
```
- Yêu cầu này sẽ thất bại vì version hiện tại đã là 6, không phải 5 nữa.

### 5. **Lợi ích của Optimistic Locking trong Ngăn Chặn Overselling**
- **Tính hiệu quả**: Optimistic Locking không yêu cầu sử dụng **database locking** ngay từ đầu, cho phép nhiều người dùng cùng truy cập và đọc dữ liệu mà không bị khóa.
- **Tránh overselling**: Với cơ chế kiểm tra version, hệ thống đảm bảo rằng chỉ một người dùng có thể thành công trong việc mua sản phẩm cuối cùng.
- **Khả năng mở rộng**: Do không dùng khóa trên toàn bộ bảng dữ liệu, Optimistic Locking giúp hệ thống mở rộng tốt hơn, tránh tắc nghẽn khi có nhiều yêu cầu đồng thời.

### 6. **Nhược điểm Cần Cân Nhắc**
- **Xung đột có thể xảy ra thường xuyên**: Nếu có nhiều người dùng cùng đặt mua sản phẩm vào cùng thời điểm, xung đột sẽ xảy ra thường xuyên và người dùng có thể phải retry nhiều lần.
- **Retry logic**: Cần phải thiết kế hệ thống để xử lý retry một cách hợp lý và không làm người dùng mất kiên nhẫn khi xảy ra xung đột.

### 7. **Khi Nào Nên Sử Dụng Optimistic Locking**
- Optimistic Locking phù hợp cho các hệ thống có **tần suất xung đột thấp**, nơi mà việc retry khi xung đột là chấp nhận được.
- Các hệ thống bán hàng trực tuyến, đặc biệt là với số lượng sản phẩm có hạn (ví dụ: vé sự kiện, hàng khuyến mãi), có thể hưởng lợi từ kỹ thuật này để tránh overselling trong khi vẫn duy trì hiệu năng cao.