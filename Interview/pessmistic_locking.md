**Pessimistic Locking** là một kỹ thuật kiểm soát truy cập đồng thời trong cơ sở dữ liệu, đảm bảo rằng một tài nguyên (dữ liệu) chỉ có thể được truy cập hoặc thay đổi bởi một transaction tại một thời điểm. Điều này được thực hiện bằng cách **khóa** tài nguyên ngay khi một transaction bắt đầu thao tác với nó, ngăn chặn các transaction khác truy cập cho đến khi khóa được giải phóng.

Trong việc ngăn chặn **overselling** (bán vượt quá số lượng tồn kho), Pessimistic Locking đảm bảo rằng khi một người dùng đang thực hiện giao dịch mua hàng, dữ liệu về số lượng tồn kho sẽ được khóa lại, ngăn chặn các giao dịch khác thay đổi số lượng này cho đến khi giao dịch hiện tại hoàn tất. Dưới đây là cách thực hiện chi tiết:

### **1. Hiểu về Pessimistic Locking**

- **Nguyên tắc hoạt động**:
  - **Khóa dữ liệu ngay lập tức**: Khi một transaction cần đọc hoặc ghi dữ liệu, nó sẽ khóa dữ liệu đó để đảm bảo không có transaction khác có thể truy cập cho đến khi khóa được giải phóng.
  - **Tránh xung đột bằng cách chờ đợi**: Các transaction khác muốn truy cập dữ liệu đã bị khóa sẽ phải chờ cho đến khi khóa được giải phóng.
  
- **Ứng dụng trong cơ sở dữ liệu**:
  - Sử dụng các cơ chế khóa của cơ sở dữ liệu như **SELECT ... FOR UPDATE**, **LOCK IN SHARE MODE** (trong MySQL), hoặc các tùy chọn tương đương trong các hệ quản trị cơ sở dữ liệu khác.
  - Khóa có thể được áp dụng ở mức hàng (row-level lock) hoặc mức bảng (table-level lock), nhưng khóa ở mức hàng thường được ưu tiên để giảm thiểu tác động đến hiệu suất.

### **2. Cách Thực Hiện Pessimistic Locking để Ngăn Chặn Overselling**

#### **Bước 1: Bắt đầu transaction**

- **Khởi tạo transaction**: Mọi thao tác sẽ được thực hiện trong một transaction để đảm bảo tính nguyên tử và tính nhất quán.
  ```sql
  START TRANSACTION;
  ```

#### **Bước 2: Khóa bản ghi sản phẩm**

- **Khóa hàng dữ liệu liên quan đến sản phẩm**: Sử dụng câu lệnh khóa để đảm bảo không có transaction nào khác có thể thay đổi dữ liệu này cho đến khi transaction hiện tại hoàn tất.
  ```sql
  SELECT stock
  FROM products
  WHERE product_id = 1
  FOR UPDATE;
  ```
  - `FOR UPDATE` sẽ khóa hàng dữ liệu được chọn.
  - Nếu hàng đã bị khóa bởi một transaction khác, transaction hiện tại sẽ phải chờ cho đến khi khóa được giải phóng.

#### **Bước 3: Kiểm tra và cập nhật số lượng tồn kho**

- **Kiểm tra số lượng tồn kho**:
  - Lấy giá trị `stock` từ kết quả truy vấn.
  - Kiểm tra xem `stock > 0` hay không.

- **Cập nhật số lượng tồn kho**:
  - Nếu `stock > 0`, tiến hành giảm `stock` đi số lượng mua.
    ```sql
    UPDATE products
    SET stock = stock - 1
    WHERE product_id = 1;
    ```
  - Nếu `stock <= 0`, rollback transaction và thông báo cho người dùng rằng sản phẩm đã hết hàng.

#### **Bước 4: Hoàn tất transaction**

- **Commit transaction**: Lưu lại thay đổi và giải phóng khóa.
  ```sql
  COMMIT;
  ```
- **Hoặc Rollback transaction**: Nếu không thể hoàn tất (ví dụ: hết hàng), hủy bỏ các thao tác và giải phóng khóa.
  ```sql
  ROLLBACK;
  ```

#### **Bước 5: Xử lý transaction khác**

- **Các transaction khác chờ đợi**:
  - Nếu có transaction khác (ví dụ: từ người dùng B) cố gắng khóa cùng bản ghi, nó sẽ phải chờ cho đến khi transaction hiện tại hoàn tất.
- **Tiến hành lại các bước**:
  - Sau khi transaction đầu tiên hoàn tất, transaction tiếp theo sẽ tiến hành từ **Bước 2**, khóa bản ghi và kiểm tra số lượng tồn kho.

### **3. Ví dụ Thực Tế**

Giả sử có hai người dùng, A và B, cùng muốn mua sản phẩm có `product_id = 1`, và số lượng tồn kho là 1.

#### **Transaction của Người dùng A**

1. **START TRANSACTION;**
2. **SELECT stock FROM products WHERE product_id = 1 FOR UPDATE;**
   - Khóa hàng dữ liệu của sản phẩm.
3. **Kiểm tra stock**:
   - `stock = 1`, tiến hành mua hàng.
4. **UPDATE products SET stock = stock - 1 WHERE product_id = 1;**
   - `stock` giảm xuống 0.
5. **COMMIT;**
   - Hoàn tất giao dịch và giải phóng khóa.

#### **Transaction của Người dùng B**

1. **START TRANSACTION;**
2. **SELECT stock FROM products WHERE product_id = 1 FOR UPDATE;**
   - Phải chờ cho đến khi transaction của người dùng A hoàn tất và giải phóng khóa.
3. **Sau khi khóa được giải phóng**, transaction của người dùng B tiếp tục.
4. **Kiểm tra stock**:
   - `stock = 0`, không thể tiến hành mua hàng.
5. **ROLLBACK;**
   - Hủy giao dịch và thông báo cho người dùng B rằng sản phẩm đã hết hàng.

### **4. Lợi ích của Pessimistic Locking trong Ngăn Chặn Overselling**

- **Đảm bảo tính toàn vẹn dữ liệu**: Ngăn chặn nhiều transaction cùng lúc thay đổi số lượng tồn kho, tránh tình trạng overselling.
- **Đơn giản trong việc triển khai**: Không cần phải xử lý xung đột sau khi chúng xảy ra, vì xung đột được ngăn chặn ngay từ đầu.
- **Phù hợp với môi trường có xung đột cao**: Khi khả năng xảy ra xung đột là lớn, Pessimistic Locking giúp đảm bảo hệ thống hoạt động đúng.

### **5. Nhược điểm của Pessimistic Locking**

- **Giảm hiệu suất và khả năng mở rộng**:
  - Khóa dữ liệu khiến các transaction khác phải chờ đợi, làm giảm khả năng xử lý song song.
  - Trong hệ thống có lượng truy cập cao, thời gian chờ đợi có thể tăng lên đáng kể.
- **Nguy cơ deadlock**:
  - Nếu không quản lý khóa cẩn thận, có thể xảy ra deadlock khi các transaction chờ đợi nhau giải phóng khóa.
- **Tăng độ phức tạp trong quản lý transaction**:
  - Cần cẩn thận trong việc bắt đầu và kết thúc transaction, cũng như xử lý các ngoại lệ để đảm bảo khóa luôn được giải phóng.

### **6. So sánh với Optimistic Locking**

- **Pessimistic Locking**:
  - **Khóa dữ liệu trước khi thao tác**: Ngăn chặn xung đột bằng cách không cho phép transaction khác truy cập dữ liệu bị khóa.
  - **Phù hợp cho môi trường xung đột cao**: Khi khả năng xung đột cao, việc khóa trước giúp tránh phải xử lý xung đột sau này.
  - **Ảnh hưởng đến hiệu suất**: Giảm khả năng xử lý đồng thời của hệ thống.

- **Optimistic Locking**:
  - **Không khóa dữ liệu khi đọc**: Cho phép nhiều transaction đọc và thao tác trên dữ liệu.
  - **Kiểm tra xung đột khi ghi**: Nếu phát hiện dữ liệu đã bị thay đổi bởi transaction khác, sẽ hủy bỏ hoặc yêu cầu thực hiện lại.
  - **Phù hợp cho môi trường xung đột thấp**: Tận dụng khả năng xử lý song song, ít ảnh hưởng đến hiệu suất.

### **7. Khi Nào Nên Sử Dụng Pessimistic Locking**

- **Khi xung đột xảy ra thường xuyên**: Trong hệ thống mà nhiều transaction thường xuyên truy cập và thay đổi cùng một dữ liệu, Pessimistic Locking giúp ngăn chặn xung đột hiệu quả.
- **Dữ liệu quan trọng cần bảo vệ chặt chẽ**: Khi việc xảy ra xung đột có thể gây ra hậu quả nghiêm trọng, ví dụ như tài chính, ngân hàng.
- **Khi việc chờ đợi chấp nhận được**: Trong một số hệ thống, thời gian chờ đợi do khóa không ảnh hưởng nhiều đến trải nghiệm người dùng.

### **8. Lưu ý Khi Triển Khai Pessimistic Locking**

- **Sử dụng mức khóa phù hợp**:
  - **Row-level locking**: Khóa ở mức hàng dữ liệu để giảm thiểu tác động đến các transaction khác.
  - **Table-level locking**: Chỉ sử dụng khi cần thiết, vì sẽ ảnh hưởng lớn đến hiệu suất.
- **Quản lý transaction cẩn thận**:
  - Luôn đảm bảo transaction được kết thúc bằng `COMMIT` hoặc `ROLLBACK`.
  - Xử lý các ngoại lệ để tránh tình trạng khóa không được giải phóng.
- **Tránh deadlock**:
  - Thiết kế thứ tự truy cập dữ liệu thống nhất trong toàn bộ ứng dụng.
  - Sử dụng các công cụ hoặc tính năng của cơ sở dữ liệu để phát hiện và giải quyết deadlock.
- **Giảm thời gian giữ khóa**:
  - Thực hiện các thao tác trong transaction càng nhanh càng tốt.
  - Tránh thực hiện các thao tác không cần thiết hoặc tốn thời gian trong khi giữ khóa.

### **9. Ví dụ Triển Khai Trong Các Hệ Quản Trị Cơ Sở Dữ Liệu Khác Nhau**

#### **MySQL với InnoDB**

- **Khóa hàng dữ liệu bằng `SELECT ... FOR UPDATE`**:
  ```sql
  START TRANSACTION;
  SELECT stock FROM products WHERE product_id = 1 FOR UPDATE;
  -- Kiểm tra và cập nhật stock
  UPDATE products SET stock = stock - 1 WHERE product_id = 1;
  COMMIT;
  ```
- **Lưu ý**: InnoDB hỗ trợ khóa ở mức hàng, giúp giảm thiểu ảnh hưởng đến các transaction khác.

#### **PostgreSQL**

- **Sử dụng `SELECT ... FOR UPDATE` tương tự**:
  ```sql
  BEGIN;
  SELECT stock FROM products WHERE product_id = 1 FOR UPDATE;
  -- Kiểm tra và cập nhật stock
  UPDATE products SET stock = stock - 1 WHERE product_id = 1;
  COMMIT;
  ```

#### **SQL Server**

- **Sử dụng `WITH (UPDLOCK)` để khóa**:
  ```sql
  BEGIN TRANSACTION;
  SELECT stock FROM products WITH (UPDLOCK) WHERE product_id = 1;
  -- Kiểm tra và cập nhật stock
  UPDATE products SET stock = stock - 1 WHERE product_id = 1;
  COMMIT TRANSACTION;
  ```

### **10. Kết Luận**

Pessimistic Locking là một phương pháp hiệu quả để ngăn chặn overselling bằng cách đảm bảo rằng chỉ một transaction có thể truy cập và thay đổi dữ liệu tồn kho tại một thời điểm. Mặc dù có thể ảnh hưởng đến hiệu suất do giảm khả năng xử lý song song, nhưng trong các hệ thống mà tính toàn vẹn dữ liệu quan trọng hơn hiệu suất, hoặc khi xung đột xảy ra thường xuyên, Pessimistic Locking là lựa chọn phù hợp.

Việc triển khai cần được thực hiện cẩn thận, với sự chú ý đến quản lý transaction, khóa, và xử lý ngoại lệ để tránh các vấn đề như deadlock hoặc giảm hiệu suất không cần thiết.