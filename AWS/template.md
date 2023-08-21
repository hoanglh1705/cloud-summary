# `S3` - Simple Storage Service

Tags: `s3`

Aliases: `Definition`

1. ***Definition***

Là một dịch vụ lưu trữ dạng Object cung cấp khả năng mở rộng avaibality, performance.
KH có thể sử dụng S3 để lưu trữ và bảo vệ nhiều loại data cho các usecase khác như:

* data lake
* Enterprise application
* IoT device
* Big Data & Analytic
  
S3 cung cấp nhiều managed feature giúp tối ưu, tổ chức và cấu hình access tới data đáp ứng nhu cầu về business, organization and compliance.

##### Đặc trưng cơ bản của S3

* Managed Service: user không cần quan tâm hệ thống hạ tầng bên duới
* Cho phép lưu file duới dạng Object với size từ 0 - 5TB
* High Duration(11 9s), Scalability, High Availability(99.9%), High performance
* Usecase đa dạng(Mọi bài toán về lưu trữ để tiết kiệm chi phí cho từng loại data).
* Cung cấp khả năng phân quyền và giới hạn truy cập một cách chi tiết.
* Dễ sử dụng, có thể kết hợp với nhiều service khác cho bài toán automation và data processing

2. ***Các tính năng của S3***

##### Tính năng cơ bản

* **Storage classes**: cung cấp nhiều hình thức lưu trữ phù hợp cho nhiều loại data khác nhua về nhu cầu access, yêu cầu về durability, thời gian lưu trữ khác nhau giúp Kh tùy chọn được class lưu trữ phù hợp từ đó tối ưu chi phí
* **Storage Management**: Cung cấp nhiều tính năng liên quan đến quản lý: Life cycle, Object Lock, Replication, Batch Operation
* **Access Management**: Quản lý truy cập đến bucket và các thư mục thông qua cơ chế resource permission & access list, Block public access, control access via IAM, bucket policy, S3 access point, Access Control List, Ownership, Access Analyzer.
* **Data processing**: kết hợp với lambda, SNS, SQS để hỗ trợ xử lý data 1 cách nhanh chóng
* **Auto Logging and Monitoring**: Cung cấp công cụ monitor S3 bucket và truy vết sử sụng CloudTrail
* **Manual Monitoring Tool**: Log lại từng record thực hiện trên bucket.
* **Analyst and Insight**: Phân tích Storage để optimize.
* **Strong consistency**: Cung cấp trong read-after-write consistency cho PUT và DELETE object.

3. ***S3 Bucket and Access control List***

**Tính năng cơ bản**


