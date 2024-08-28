Tại sao lại cần đến ElasticSearch?
Ngay cái tên của nó đã nói lên tất cả, Khi chúng ta cần saerch, khi cần search thì hãy nghĩ ngay đến elesticsearch

1. Tìm kiếm nhanh chóng:
Ví dụ bạn truy cập vào một trang đọc báo và với số lượng hàng triệu bài báo, và hiện tại giá cà phê đang rất nóng. Bạn là một người trồng cà phê và muốn có những tin tức về cà phê để dự đoán về giá sắp tới.
Và bạn nhập vào ô tìm kiếm từ khóa "cà phê".
Và nếu trang báo sử dụng query filter thì sẽ phải scan qua tất cả các bản ghi liên quan đến bài báo gồm có title, nội dung, tag, etc. Và thời gian chờ đợi là vô cùng lâu và có thể timeout, treo hệ thống,.. hệ quả là người dùng cảm thấy không happy với kết quả này và qua một trang báo khác để tìm.
Vậy nên ElasticSearch sinh ra để giải quyết vấn đề đó cho bạn, bởi Elasticsearch sử dụng Luence, một thư viện search engine hiệu năng cao, để mặc định đánh index tất cả dữ liệu. Bạn có thể thêm index vào các trường trong hầu hết các loại cơ sở dữ liệu bằng nhiều cách. Lucene làm việc đó bằng phương pháp inverted indexing. Cụ thể thế nào chúng ta có thể xem rõ hơn ở ví dụ sau:

2. Đảm bảo được kết quả tìm liên quan nhất

Câu hỏi là thế nào gọi là "liên quan". Quay lại ví dụ là chúng ra search từ khóa "cà phê", bài viết liên quan ví dụ là "giá cà phê liên tục đạt đỉnh", một bài viết khác cũng có từ khóa "cà phê" là "vua cà phê Qua Vũ lái siêu xe dạo phố".
Thì bài viết liên quan sẽ là bài viết thứ nhất. Vậy là sao có thể biết được nó liên quan hơn để đưa nó lên trước. Theo mặc định, thuật toán TF-IDF.
Vậy TF-IDF alf gì: term frequency-inverse document frequency, gồm 2 yếu tố ảnh hưởng chính là:
Tần suất xuất hiện (Term frequency): Tần suất xuất hiện càng nhiều, điểm số liên quan càng cao.
Tần số tài liệu nghịch đảo (Inverse document frequency): Tần số nghịch của 1 từ trong tập văn bản, chỉ số này giúp đánh giá tầm quan trọng của một từ . Khi tính toán TF , tất cả các từ được coi như có độ quan trọng bằng nhau. Nhưng một số từ như “is”, “of” và “that” thường xuất hiện rất nhiều lần nhưng độ quan trọng là không cao. Như thế chúng ta cần giảm độ quan trọng của những từ này xuống. Bạn cũng có thể tùy biến điểm liên quan phù hợp với thứ mà bạn cần nhờ vào việc Elasticsearch cung cấp rất nhiều tính năng dựng sẵn.

3. Tìm kiếm thông minh và linh hoạt

Với việc sử dụng Elasticsearch, bạn có các tùy chọn để làm cho việc tìm kiếm linh hoạt và thông minh hơn. Các tùy chọn này xử lí khi người dùng nhập sai lỗi chính tả hoặc sử dụng từ đồng nghĩa/gần nghĩa với những gì mà bạn lưu trữ.

Xử lí lỗi nhập keywword sai chính tả: Bạn có thể cài đặt Elasticsearch để có thể xử lí tìm kiếm các biến thể thay vì chỉ tìm kiểm các kết quả khớp chính xác. Một câu truy vấn mờ có thể được sử dụng để tìm kiếm từ "yêi" sẽ cho ra kết quả các bài viết về tình yêu. Chúng ta sẽ tìm hiểu sâu hơn về nó ở các bài viết sau.
Hỗ trợ tìm kiếm các từ họ hàng, từ đồng nghĩa
Suggest cho các keyword: Khi người dùng bắt đầu nhập, bạn có thể giúp suggest cho họ các tìm kiếm phổ biến và kết quả phổ biến. Bạn có thể sử dụng các đề xuất để dự đoán các tìm kiếm của họ khi họ nhập, giống như hầu hết các công cụ tìm kiếm trên web. Bạn cũng có thể hiển thị kết quả phổ biến khi họ nhập, sử dụng các loại truy vấn đặc biệt khớp với tiền tố hoặc biểu thức thông thường.

Bài viết tham khảo từ cuốn sách Elasticsearch In Action (Matthew Lee Hinman and Radu Gheorghe)