# `Encryption` - Encryption

Tags: `AWS`

Aliases: `Encryption`

1. ***Why encryption***

**Encryption in fligth (SSL)**

* Data được mã hóa trước khi gửi và được giải mã sau khi nhận
* SSL certificates giúp mã hóa data (HTTPS)
* Encryption in fligth đảm bảo không có MITM(man in middle attack) có thể xảy ra

**Server side encryption at rest**

* Data được mã hóa trước khi nhận bởi server
* Data được giải mã trước khi gửi tới client
* Nó được lưu trữ theo 1 form mã hóa bởi key (thường 1 data key)
* Key Mã hóa/Giải mã phải được quản lý bở 1 nơi nào đó và server phải có thể access nó ( có thể **KMS**)

**Client side encryption**

*Data được mã hóa bở client và không thể được giải mã bởi server

* Data sẽ được giải mã bởi client nhận data
* Server nên không có thể giải mã data
* could leverage envelope encryption

2. ***KMS Key ***

* Bất kể lúc nào chúng ta nghe "encryption" cho 1 aws service, nó chính là KMS (most likely)
* AWS quản lý encryption keys cho chúng ta
* Fully integrated với IAM để authorization
* Cách đơn giản để quản lý access cho data của bạn
* Có thể audit KMS Key được sử dụng ở đâu thông qua CloudTrail
* Khả năng integrated đồng bộ với các AWS Services(EBS, S3, RDS, SSM,...)