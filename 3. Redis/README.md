# REDIS

## Mục lục

1. [REDIS là gì?](#1)
2. [Sự ra đời của REDIS](#2)
3. [Các đặc trưng cơ bản của REDIS](#3)
   1. [Data model](#3.1)
   2. [Master/slave](#3.2)
   3. [Clustering](#3.3)
   4. [In-memory](#3.4)
4. [REDIS persistence](#4)
   1. [RDB](#4.1)
   2. [AOF](#4.2)
5. [Kết luận](#5)
6. [Tham khảo](#6)

<a name = "1"></a>
## REDIS LÀ GÌ?

Ngày nay, khái niệm NoSQL trở nên không còn xa lạ trong giới Công Nghệ Thông Tin (CNTT). Đi kèm với đó là sự ra đời của hàng loạt hệ quản trị cơ sở dữ liệu (DBMS) phát triển dựa trên đặc thù của NoSQL: Non-relational (không quan hệ), Distributed (phân tán), Open-source (mã nguồn mở), Horizontally scalable (dễ dàng mở rộng theo chiều ngang).

Redis là 1 trong số các hệ quản trị cơ sở dữ liệu phát triển mang phong cách NoSQL. Trong bài viết này, chúng ta sẽ tìm hiểu sơ lược về Redis, cũng như làm nổi bật sự khác biệt cơ bản của Redis với các DBMS khác.

<a name = "2"></a>
## Sự ra đời của Redis

Trước tiên, chúng ta sẽ cùng nhau tìm hiểu về sự ra đời của Redis. Câu chuyện bắt đầu khi tác giả của Redis, Salvatore Sanfilippo (nickname: antirez), cố gắng làm những công việc gần như là không thể với SQL Database!

Server của antirez nhận 1 lượng lớn thông tin từ nhiều trang web khác nhau thông qua JavaScript tracker, lưu trữ n page view cho trừng trang và hiển thị chúng theo thời gian thực cho user, kèm theo đó là lưu trữ 1 lượng nhỏ lịch sử hiển thị của trang web. Khi số lượng page view tăng đến hàng nghìn page trên 1 giây, antirez không thể tìm ra cách tiếp cận nào thực sự tối ưu cho việc thiết kế database của mình. Tuy nhiên, anh ta nhận ra rằng, việc lưu trữ 1 danh sách bị giới hạn các bản ghi không phải là vấn đề quá khó khăn. Từ đó, ý tưởng lưu trữ thông tin trên RAM và quản lý các page views dưới dạng native data với thời gian pop và push là hằng số được ra đời. Antirez bắt tay vào việc xây dựng prototype bằng C, bổ sung tính năng lưu trữ thông tin trên đĩa cứng và… Redis ra đời.

Đến nay cộng đồng người sử dụng và phát triển Redis đang dần trở lên lớn mạnh, redis.io trở thành cổng thông tin cho người làm việc với Redis. Bản thân tác giả bài viết này cũng bắt đầu từ redis.io, và hiện tại đang đọc cuốn “Redis in Action”.

Đến đây có thể nhiều người sẽ muốn download và chạy thử xem câu lệnh Redis như nào, giống và khác gì so với bài học vỡ lòng về MySQL. Tuy nhiên bài viết này sẽ không tập trung vào việc cài đặt và chạy các câu lệnh của Redis, chúng ta sẽ tập trung tìm hiểu các đặc trưng nổi bật của Redis.

<a name = "3"></a>
## Các đặc trưng cơ bản của Redis

<a name = "3.1"></a>
### 1. Data model

**REDIS KEYS & CÁCH ĐẶT TÊN**

* Redis keys là binary-safe. Mỗi chuỗi nhị phân có thể coi sử dụng như một key. Chính vì thế, chuỗi rỗng là một key hợp lệ.
* Một số lưu ý:
  * Không sử dụng key quá dài. Tốn bộ nhớ và chi phí so sánh key.
  * Không nên sử dụng key quá ngắn. Key ngắn tiết kiệm một chút ít bộ nhớ nhưng có thể gây khó đọc và hiểu ý nghĩa của key. Sử dụng key “user:1000:followers” thay cho “u1000flw”.
  * Cố gắng gán với một schema. Ví dụ một schema: “object-type:id” . Sử dụng dấu chấm và ngạch nối để liên kết các trường như  "comment:1234:reply-to”.
  * Kích thước tối đa của key là 512 MB.

Khác với RDMS như MySQL, hay PostgreSQL, Redis không có bảng. Redis lưu trữ data dưới dạng key-value. Thực tế thì memcache cũng làm vậy, nhưng kiểu dữ liệu của memcache bị hạn chế, không đa dạng được như Redis, do đó không hỗ trợ được nhiều thao tác từ phía người dùng. Dưới đây là sơ lược về các kiểu dữ liệu Redis dùng để lưu value.

**STRING:** Có thể là string, integer hoặc float. Redis có thể làm việc với cả string, từng phần của string, cũng như tăng/giảm giá trị của integer, float.

**LIST:** Danh sách liên kết của các strings. Redis hỗ trợ các thao tác push, pop từ cả 2 phía của list, trim dựa theo offset, đọc 1 hoặc nhiều items của list, tìm kiếm và xóa giá trị.

**SET** Tập hợp các string (không được sắp xếp). Redis hỗ trợ các thao tác thêm, đọc, xóa từng phần tử, kiểm tra sự xuất hiện của phần tử trong tập hợp. Ngoài ra Redis còn hỗ trợ các phép toán tập hợp, gồm intersect/union/difference.

**HASH:** Lưu trữ hash table của các cặp key-value, trong đó key được sắp xếp ngẫu nhiên, không theo thứ tự nào cả. Redis hỗ trợ các thao tác thêm, đọc, xóa từng phần tử, cũng như đọc tất cả giá trị.

**ZSET (sorted set)**: Là 1 danh sách, trong đó mỗi phần tử là map của 1 string (member) và 1 floating-point number (score), danh sách được sắp xếp theo score này. Redis hỗ trợ thao tác thêm, đọc, xóa từng phần tử, lấy ra các phần tử dựa theo range của score hoặc của string.

**HyperLogLogs**: Là cấu trúc dữ liệu xác suất sử dụng trong sắp xếp để đếm các phần tử phân biệt. Thông thường, công việc đếm các phần tử này yêu cầu bộ nhớ bằng với số phần tử, bởi vì chúng ta phải xem xét sự xuất hiện của phần tử và tránh đếm nó nhiều lần. Trong trường hợp xấu nhất, Redis sử dụng 12 KB. Hoặc ít hơn rất nhiều nếu HLL chỉ chứa một vài phần tử. HLL trong Redis được encode như String. Sử dụng GET để serialize một HLL và SET để deserizlize trở lại.

Lưu ý nhỏ dưới góc độ lập trình viên: Trang web ktmt.github.io đưa ra loạt bài phân tích về source code Redis (viết bằng C), trong đó có 1 phần về kiểu dữ liệu của Redis. Tham khảo các bài viết đó, chúng ta có thể thấy Redis sử dụng 1 layer mô tả dữ liệu ở mức độ abstract, là redisObjectr-robj (định nghĩa trong redis.h), các thao tác cơ bản của db (db.c) đều làm việc trực tiếp với robj và không cần biết đến sự tồn tại của các kiểu string, list, hash, set, zset. Sơ đồ tổ chức có thể tham khảo trong mô hình dưới đây.

```scala
╒===============╕
|  t_hash.c     |
|  t_list.c     |       ╒============╕      ╒=====================╕
|  t_set.c      |  <=>  |  object.c  | <=>  |  db.c (robj -> sds) |
|  t_string.c   |       ╘============╛      ╘=====================╛
|  t_zset.c     |
╘===============╛
(Nguồn: ktmt.github.io)
```

Thiết kế này giúp các thao tác làm việc với các kiểu dữ liệu khác nhau trở nên dễ dàng quản lý hơn, đồng thời hỗ trợ việc tăng cường số lượng kiểu dữ liệu trong tương lai. Kỹ thuật này tuân thủ [Dependency Inversion Principle](http://www.oodesign.com/dependency-inversion-principle.html). Lập trình viên có thể biết đến principle này hoặc không, nhưng chắc chắn nhiều người đã từng thiết kế/tổ chức code tuẩn thủ principle này. Đặc biệt là khi sử dụng [Factory Pattern](http://www.oodesign.com/factory-pattern.html).

<a name = "3.2"></a>
### 2. Master/Slave Replication

Đây không phải là đặc trưng quá nổi bật, các DBMS khác đều có tính năng này, tuy nhiên chúng ta nêu ra ở đây để nhắc nhở rằng, Redis không kém cạnh các DBMS về tình năng Replication.

<a name = "3.3"></a>
### 3. Clustering

Nếu sử dụng MySQL, bạn phải trả phí để có thể sử dụng tính năng này, còn với họ NoSQL DBMS, tính năng này hoàn toàn free. Tuy nhiên, Redis Cluster đang ở phiên bản alpha, và chúng ta sẽ cùng chờ đợi phiên bản chính thức ra đời. Chúng ta sẽ đề cập đến tính năng này qua 1 bài viết khác, khi Redis Cluster có phiên bản chính thức.

<a name = "3.4"></a>
### 4. In-memory

Đây là điều gây ấn tượng mạnh nhất với tác giả khi bắt đầu tìm hiểu về Redis. Không như các DBMS khác lưu trữ dữ liệu trên đĩa cứng, Redis lưu trữ dữ liệu trên RAM, và đương nhiên là thao tác đọc/ghi trên RAM. Với người làm CNTT bình thường, ai cũng hiểu thao tác trên RAM nhanh hơn nhiều so với trên ổ cứng, nhưng chắc chắn chúng ta sẽ có cùng câu hỏi: Điều gì xảy ra với data của chúng ta khi server bị tắt?

Rõ ràng là toàn bộ dữ liệu trên RAM sẽ bị mất khi tắt server, vậy làm thế nào để Redis bảo toàn data và vẫn duy trì được ưu thế xử lý dữ liệu trên RAM. Chúng ta sẽ cùng tìm hiểu về cơ chế lưu dữ liệu trên ổ cứng của Redis trong phần tiếp theo của bài viết.

<a name = "4"></a>
## Redis Persistence

Như đã đề cập ở trên, mặc dù làm việc với data dạng key-value lưu trữ trên RAM, Redis vẫn cần lưu trữ dữ liệu trên ổ cứng. Có 2 lý do cho việc này, 1 là để đảm bảo toàn vẹn dữ liệu khi có sự cố xảy ra (server bị tắt nguồn) cũng như tái tạo lại dataset khi restart server, 2 là để gửi data đến các slave server, phục vụ cho tính năng replication. Redis cung cấp 2 phương thức chính cho việc sao lưu dữ liệu ra ổ cứng, đó là RDB và AOF.

<a name = "4.1"></a>
### 1. RDB (Redis DataBase file)

* Cách thức làm việc
  * RDB thực hiện tạo và sao lưu snapshot của DB vào ổ cứng sau mỗi khoảng thời gian nhất định.
* Ưu điểm
  * RDB cho phép người dùng lưu các version khác nhau của DB, rất thuận tiện khi có sự cố xảy ra.
  * Bằng việc lưu trữ data vào 1 file cố định, người dùng có thể dễ dàng chuyển data đến các data centers khác nhau, hoặc chuyển đến lưu trữ trên Amazon S3.
  * RDB giúp tối ưu hóa hiệu năng của Redis. Tiến trình Redis chính sẽ chỉ làm các công việc trên RAM, bao gồm các thao tác cơ bản được yêu cầu từ phía client như thêm/đọc/xóa, trong khi đó 1 tiến trình con sẽ đảm nhiệm các thao tác disk I/O. Cách tổ chức này giúp tối đa hiệu năng của Redis.
  * Khi restart server, dùng RDB làm việc với lượng data lớn sẽ có tốc độ cao hơn là dùng AOF.
* Nhược điểm
  * RDB không phải là lựa chọn tốt nếu bạn muốn giảm thiểu tối đa nguy cơ mất mát dữ liệu. Thông thường người dùng sẽ set up để tạo RDB snapshot 5 phút 1 lần (hoặc nhiều hơn). Do vậy, trong trường hợp có sự cố, Redis không thể hoạt động, dữ liệu trong những phút cuối sẽ bị mất.
  * RDB cần dùng fork() để tạo tiến trình con phục vụ cho thao tác disk I/O. Trong trường hợp dữ liệu quá lớn, quá trình fork() có thể tốn thời gian và server sẽ không thể đáp ứng được request từ client trong vài milisecond hoặc thậm chí là 1 second tùy thuộc vào lượng data và hiệu năng CPU.

<a name = "4.2"></a>
### 2. AOF (Append Only File)

* Cách thức làm việc:
  * AOF lưu lại tất cả các thao tác write mà server nhận được, các thao tác này sẽ được chạy lại khi restart server hoặc tái thiết lập dataset ban đầu.
* Ưu điểm
  * Sử dụng AOF sẽ giúp đảm bảo dataset được bền vững hơn so với dùng RDB. Người dùng có thể config để Redis ghi log theo từng câu query hoặc mỗi giây 1 lần.
  * Redis ghi log AOF theo kiểu thêm vào cuối file sẵn có, do đó tiến trình seek trên file có sẵn là không cần thiết. Ngoài ra, kể cả khi chỉ 1 nửa câu lệnh được ghi trong file log (có thể do ổ đĩa bị full), Redis vẫn có cơ chế quản lý và sửa chữa lối đó (redis-check-aof).
  * Redis cung cấp tiến trình chạy nền, cho phép ghi lại file AOF khi dung lượng file quá lớn. Trong khi server vẫn thực hiện thao tác trên file cũ, 1 file hoàn toàn mới được tạo ra với số lượng tối thiểu operation phục vụ cho việc tạo dataset hiện tại. Và 1 khi file mới được ghi xong, Redis sẽ chuyển sang thực hiện thao tác ghi log trên file mới.
* Nhược điểm
  * File AOF thường lớn hơn file RDB với cùng 1 dataset.
  * AOF có thể chậm hơn RDB tùy theo cách thức thiết lập khoảng thời gian cho việc sao lưu vào ổ cứng. Tuy nhiên, nếu thiết lập log 1 giây 1 lần có thể đạt hiệu năng tương đương với RDB.
  * Developer của Redis đã từng gặp phải bug với AOF (mặc dù là rất hiếm), đó là lỗi AOF không thể tái tạo lại chính xác dataset khi restart Redis. Lỗi này chưa gặp phải khi làm việc với RDB bao giờ.

Câu hỏi đặt ra là, chúng ta nên dùng RDB hay AOF? Mỗi phương thức đều có ưu/nhược điểm riêng, và có lẽ cần nhiều thời gian làm việc với Redis cũng như tùy theo ứng dụng mà đưa ra lựa chọn thích hợp. Nhiều người chọn AOF bới nó đảm bảo toàn vẹn dữ liệu rất tốt, nhưng Redis developer lại khuyến cáo nên dùng cả RDB, bởi nó rất thuận tiện cho việc backup database, tăng tốc độ cho quá trình restart cũng như tránh được bug của AOF.

Cũng cần lưu ý thêm rằng, Redis cho phép không sử dụng tính năng lưu trữ thông tin trong ổ cứng (không RDB, không AOF), đồng thời cũng cho phép dùng cả 2 tính năng này trên cùng 1 instance. Tuy nhiên khi restart server, Redis sẽ dùng AOF cho việc tái tạo dataset ban đầu, bới AOF sẽ đảm bảo không bị mất mát dữ liệu tốt hơn là RDB.

<a name = "5"></a>
## Kết luận

Bài viết trên đây đã đưa ra những đặc trưng cơ bản của Redis, cũng như những vấn đề người dùng cần lưu tâm khi sử dụng Redis. Trong các bài viết tiếp theo, chúng ta sẽ cùng thực hành Redis để tìm hiểu rõ hơn về các function của DBMS này.

<a name = "6"></a>
## Tham khảo

* [Redis document](https://dotrungduchd.files.wordpress.com/2017/10/redis-document.pdf)
* [Antirez. (n.d.). Antirez. Retrieved](http://antirez.com/)
* [Redis. (n.d.). Retrieved](http://redis.io/documentation)
* [Redis Cluster. (n.d.). Retrieved](http://redis.io/topics/cluster-spec)
* [NoSQL](http://nosql-database.org/)
* [Redis in Action](http://www.amazon.com/Redis-Action-Josiah-L-Carlson/dp/1617290858/ref=sr_1_1?ie=UTF8&qid=1406196627&sr=8-1)
* [Tìm Hiểu Redis (Phần 3): đối Tượng Trong Redis (Redis Objects)](http://ktmt.github.io/blog/2013/08/04/tim-hieu-redis-3/)
* [Redis Persistence](http://redis.io/topics/persistence)