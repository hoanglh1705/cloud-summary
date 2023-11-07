# `ECS` - Elastic container service

Tags: `AWS`

Aliases: `ECS`

1. ***Definition***

* Là một service dịch vụ với high scalable và high performance dung để quản lý container và hỗ trợ docker containers.

##### Đặc trưng cơ bản

* Là một DB as service, user không cần quan tâm tới hạ tầng ở bên dưới
* Cho phép người dùng tạo ra các DB instance độc lập hoặc cụm instance hoạt động dưới mod Cluster
* Không thể login vào instance level (khác với việc tự cài DB lên EC2 instance)
* Có thể Scale theo 2 hình thức: Virtual (tăng giảm cấu hình instance), Horizontal (thêm hoặc bớt node tuy nhiên node này chỉ có thể read (cụ thể là read-replica))
* Có giới hạn về dung lượng ổ cứng tối đa (64TB đối với MySQL, Maria,... 16TB đối với MS SQL)
* Hỗ trợ Read Replicas và Multi AZ
* Security bằng IAM, Security groups, KMS (rest encryption), SSL lúc chuyển giao data
* Tự động backup với chức năng Point in time restore (up to 35 day) => Tạo DB mới từ bản backup đó
* Hỗ trợ IAM Authentication và tích hợp vơớ Secrets Manager

##### Hỗ trợ các engine sau:

* Amazon Aurora
* Mysql
* Mariadb
* Postgresql
* Oracle
* SQL Server

2. ***Features of RDS***

**RDS Cung cấp các tính năng cơ bản:**

* Cho phép tạo các DB instance hoặc cụm cluster một cách nhanh chóng.
* Tự động fail-over giữa master-slave instance khi có sự cố
* High-Availability: tự động cáu hình instance stand by, người dùng chỉ cần chọn.
* Tự động scale dung lượng lưu trữ (optional)
* Liên kết với cloudWatch để giám sát dễ dàng
* Automate backup and manage retention.
* Dễ dàng chỉnh sửa setting ở cấp độ Database sử dụng parameter group

3. ***RDS Usecase***

* RDS được sử dụng trong hầu hết các trường hợp cần Database dạng quan hệ. VD: Lưu trữ thông tin user,...
* RDS Thích hợp cho các bài toán OLAP (Online Analatical Process) nhờ khả năng truy vấn mạnh mẽ, cấu hình có thể scale theo yêu cầu
 
4. ***RDS Pricing***

Về cơ bản RDS tính tiền dược trên các thông số:

* instance size: instance size càng lớn cost càng cao, có hỗ trợ reserve instance tương tự EC2
* Lượng data lưu trữ (GB/Month)
* Dung lượng các bản snapshot được tạo ra
* Các tính năng khác vd Backtracking đối với Aurora
 
5. ***Mô hình triển khai RDS***

##### RDS có thể được triển khai theo một số mô hình sau

##### Single instance

* Chỉ có 1 instance duy nhất được tạo ra trên 1 Availability Zone (AZ)
* Nếu sự cố xảy ra ở cấp độ AZ, database sẽ không thể truy cập
* Phù hợp cho môi trường dev/test để tiết kiệm chi phí

##### Single instance with Multi-AZ option = yes

* Một bản sao cảu instance sẽ được tạo ra ở AZ khác và hoạt động dưới mode standby.
* Nhiệm vụ của instance standby này là sync data từ masterm, không thể truy cập instance này.
* Khi có sự cố xảy ra thì con standby sẽ được auto promo lên làm con master (Endpoint url được giữ nguyên)
* Nếu enable Multi AZ, số tiền bỏ ra sẽ x2.
* Phù hợp cho Database Prod

##### Master - Read Only cluster

* Một instance với mod ReadOnly sẽ được tạo ra và liên tục replica data từ master instance
* instance này chỉ có thể đọc data. Phù hợp cho hệ thống có workload read>write, muốn tới ưu performance của Database
* Sau khi được thiết lập quan hệ, instance được tạo ra sẽ kết hợp thành 1 cluster
* \* Trong trạng thái instance đã hình thành cluster, nếu Master instance gặp sự cố, failover sẽ dược tự động thực hiện, ReadOnly instance được promo lên làm Master
Lưu ý: Nếu 2 instance được tạo ra riêng biệt sau đó mới thiết lập quan hệ read-replica, Endpoint của 2 instance sẽ riêng biệt nên sau khi failover, cần chỉnh lại connection của application
Nên tạo cluster sau đó mới add note read vào để quản lý connection ở cluster level

##### Master - Read Only cluster with Multi-AZ option = yes

* Tương tự mô hình Master - Read Only tuy nhiên các node đều được bật multi-AZ enabled.
* Chi phí sẽ gấp 4 lần mô hình single instance

##### Master - Multi Read cluster

* Với mô hình này nhiều hơn 1 raeder instance sẽ được khởi tạo

***Nên chọn RDS cluster hay RDS instance***
AWS cung cấp cơ chế cho phép tạo ra 1 cluster RDS giúp quản lý node và failover dễ dàng hơn.
Ưu điểm so với việc tạo RDS instance thông thường:

* Quản lý Endpoint ở cấp độ cluster, không bị thay đổi khi instance trong cluster gặp sự cố
* failover tự động
* Scale read instance dễ dàng

Tính năng RDS snapshot: Snapshot là 1 cơ chế giúp chúng ta có thẻ recovery db khi có sự cố xảy ra, cũng giống như ec2, snapshot là 1 bản chụp lại tại 1 thời điểm. hiện trạng của db lúc snapshot => tạo ra 1 instance moi tu ban snapshot do


6. ***Aurora***

* Aurora là công nghệ database độc quyền cảu AWS, hỗ trợ 2 loại engine là MySQL và Postgresql
* Data được lưu tại 6 Replicas thông qua 3 AZ- High Available, Self-Healing, Auto-scaling
* Compute: Cluster của DB instance tồn tại đa AZ, auto-scaling của Read Replicas
* Cluster: Custom Endpoint cho Writer and reader DB instance
* Aurora Serverless - cho việc chưa thể dự đoán/ đột ngột workloads, không có kế hoạch Capacity
* Aurora có 2 hình thức triển khai: Cluster (Master + Multi read replica), Serverless
* Những tính năng vượt trội cảu Aurora so với RDS thông thường:

- Hiệu năng tối hơn (So với RDS instance cùng cấu hình)
- Hỗ trợ Backtracking (1 tính năng cho phép revert database về trạng thái trong quá khứ tối đa 72h.). Khác với restore từ snapshot đòi hỏi tạo instance mới, Backtracking restore ngay trên chsinh instance đang chạy
- Tự động quản lý Write Endpoint, Read Endpoint ở cấp độ cluster

##### Mô hình cluster của Aurora

**Cluster Aurora bao gồm:**

* 1 Master node (Primary instance)
* 1 hoặc nhiều Replica nde (tối đa 15)
Data của Aurora Cluster được lưu trên 1 storage layer gọi là Cluster Volumn span trên nhiều AZ để tăng HA.
Data được Cluster Volumne tự động replica trên nhiều AZ. \* Số lượng copy không phụ thuộc vào số lượng instance của cluster.
Cluster Volume tự động tăng size dựa vào nhu cầu của người dùng (k thể fix cứng size)

**Aurora Global Cluster**

Là một cơ chế cho phép tạo ra cụm cluster cross trên nhiều regions.

* Tăng tốc độ read tại mỗi region tương đương với local read.
* Mở rộng khả năng scale số lượng node read (limit 15 read nodes cho 1 cluster cho mỗi region, <1 second storage replication)
* Failover Disaster recovery: rút ngắn RTO và giảm RPO khi xảy ra sự cố ở cấp độ region.
Mặc định cluster ở region thứ 2 trở đi chỉ có thể read, tuy nhiên có thể enable write forwarding để điều hướng request tới primary cluster

**Serverless của Aurora**

Aurora serverless là 1 công nghệ cho phép tạo Database dưới dạng serverless
Thay vì điều chỉnh cấu hình của DB instance, người dùng sẽ điều chỉnh ACU (Aurora Capacity Unit), ACU càng cao thì hiệu suất DB càng mạnh
Phù hợp cho các hệ thống chưa biết rõ workload, hoặc workload có đặc trưng thay đổi lên xuống thường xuyên

***Usecase của Aurora***: Giống như của RDS, nhưng ít mantenance/ flexibility hơn/ performance hơn/ nhiều Features hơn

1. ***RDS Proxy***

RDS cung cấp cơ chết proxy giúp quản lý connection tới các instance một cách hiệu quả, hạn chết bottle neck
Khi sử dụng proxy, application sẽ không kết nối trực tiếp tới RDS mà thông qua proxy endpoint => khi chuyển qua rds proxy phải thay đổi endpoint
Chi phí sẽ phát sinh thêm cho proxy
Hiện tại hỗ trợ 3 engine : MySQL, Postgresql, SQL Server