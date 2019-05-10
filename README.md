---
title: "Report System thinking"
date: 2019-04-22T15:15:45+07:00
draft: false
toc: false
images:
tags: 
  - system thingking report
---

# Mục lục

I. [CAP](#CAP)

  1. [Định lý CAP](#DL_CAP)
  2. [Khái niệm eventualy consistency](#eventualy)

II. [Throughtput - Latency](#throughput)

III. [Phương pháp scale database (My SQL)](#scale_db)

  1. [Tại sao cần scale database?](#why_1)
  2. [Một số phương pháp scale database (áp dụng MySQL)](#pp)
     1. [MySQL Replication](#rep)
     2. [MySQL Shard](#shard)\
        [Cách Pinterest scale database của họ](#pin)

IV. [Task queue và Message queue](#task_message)

  1. [Message queue](#mess)
  2. [Task queue](#task)

V. [Credit](#credit)

<a name = "CAP"></a>
# I. CAP

Trong bối cảnh công nghệ hiện nay, hệ thống đang cần phải được mở rộng, thêm vào các nguồn lực cần thiết như máy tính, bộ nhớ,... để hoàn thành tác vụ trong thời gian chấp nhận được. Để xử lý được luồng công việc ngày một lớn, cần phải sử dụng thêm các sản phẩn phần cứng, khi đó sẽ có nhiều bất cập liên quan đến việc đồng bộ và họat động song song giữa các thành phần của hệ thống. Và đây chính là lúc định lý CAP nhập cuộc.

Định lý CAP nêu rõ rằng, trong một hệ thống được phân bổ (một tập hợp các nodes có liên kết chia sẻ dữ liệu với nhau), bạn chỉ có thể đảm bảo hai trong ba ràng buộc trên một cặp đọc/ghi sau đây: Tính nhất quán, Tính sẵn sàng và Dung sai phân mảnh. Tuy nhiên không có hệ thống nào có thể đạt được cả 3 yếu tố trên và bắt buộc phải lựa chọn dựa trên các thuộc tính của hệ thống.

<a name = "DL_CAP"></a>
## 1. Định lý CAP

C - Consistency - tính nhất quán: mọi yêu cầu đọc sẽ được trả về kết quả ghi gần nhất hoặc thông báo lỗi.

A - Availability - tính sẵn sàng: mọi yêu cầu đều nhận được kết quả, không đảm bảo sẽ nhận được kết quả mới nhất của thông tin.

P - Partition tolenrance - dung sai phân mảnh: hệ thống vẫn tiếp tục hoạt động kể cả khi có lỗi kết nối mạng giữa một số nodes.

```
Một hệ thống mạng thì rất dễ gặp lỗi, vì vậy bạn cần đảm bảo hệ thống vẫn hoạt động dù có một số vùng có kết nối yếu. Do đó bạn cần lựa chọn giữa tính nhất quán và tính sẵn sàng dựa trên tính chất của phần mềm.
```

<a name = "Eventualy"></a>
## 2. Khái niệm eventualy consistency

Với vô số bản copy của cũng một dữ liệu, vấn đề phải đối mặt là làm sao đồng bộ chúng để mọi người dùng có sự nhất quán về góc nhìn với dữ liệu.

* Weak consistency

Đồng bộ yếu: Sau khi có cập nhật về thông tin, người dùng chỉ đọc có thể sẽ thấy hoặc không thấy phiên bản cập nhật của dữ liệu. Dữ liệu trên hệ thống sẽ được lưu dưới vùng nhớ tạm. Tính đồng bộ yếu thường được sử dụng trong các tác vụ sử dụng thời gian thực như VoIP, gọi video, game online nhiều người chơi. Khi có lỗi mất kết nối/kết nối yếu xảy ra, người dùng sẽ bị mất những thông tin được truyền đi trong thời gian xảy ra lỗi.

* Eventual consistency

Đồng bộ cuối cùng: Là một dạng của đồng bộ yếu, chỉ khác là cuối cùng thông tin vẫn được cập nhật. Trong thời gian đồng bộ, người dùng chỉ đọc vẫn sẽ thấy được những dữ liệu cũ. Sau khi hoàn tất quá trình đồng bộ, mọi yêu cầu đọc đều sẽ nhận được kết quả là phiên bản mới nhất của thông tin.

<a name = "throughput"></a>
# II. Throughput - Latency

* Latency: Thời gian cần thiết để hoàn thành 1 tác vụ hoặc để tạo ra một kết quả.

* Throughput: Số tác vụ hoặc kết quả trong một đơn vị thời gian.

Ví dụ: Một dây chuyền sản xuất cần 8 giờ để hoàn thành 1 chiếc xe hơi. Trong một ngày nhà máy đó sản xuất được 120 chiếc xe.

+ Latency: 8 giờ
+ Throughput: 120 xe/ngày hay 5 xe/giờ

<a name = "scale_db"></a>
# III. Phương pháp scale database (My SQL)

<a name = "why_1"></a>
## 1. Tại sao cần scale Database?

Với ứng dụng web có traffic lớn thì việc scale là không thể tránh khỏi. Scale thì có thể tiến hành trên nhiều tầng, như tầng app, tầng db. Database là một hệ thống quan trọng và là một dịch vụ xương sống giúp toàn bộ hệ thống hoạt động, do đó, nó cũng trở thành điểm yếu trong quá trình mở rộng hệ thống. Khi mở rộng hệ thống, lượng truy cập vào các web server sẽ tăng lên, kéo theo các kết nối từ web server đến database sẽ tăng lên và khiến database quá tải và nó trở thành điểm yếu của toàn hệ thống.

<a name = "pp"></a>
## 2. Một số phương pháp scale database (áp dụng MySQL)

<a name = "rep"></a>
### 2.1. MySQL Replication

#### 2.1.1. Giới thiệu

Right tool for right job. Trước tiên phải hiểu là MySQL Replication không phải là giải pháp giải quyết mọi bài toán về quá tải hệ thống cơ sở dữ liệu. Để mở rộng một hệ thống ta có hai phương pháp mở rộng là scale up và scale out. Bắt đầu với 1 máy chủ thì hai phương pháp trên được diễn giải như sau:

* Scale up có nghĩa là với một máy chủ ta làm cách nào đó để nó có thể phục vụ nhiều hơn số lượng kết nối, truy vấn. Nghĩa là giá trị 1/(số kết nối phục vụ) càng nhỏ thì càng tốt. Để đạt được mục đích này thì có 2 phương pháp:
  * Tăng phần cứng lên cho máy chủ. Nghĩa là với CPU là 4 core, RAM là 8 GB phục vụ được 500 truy vấn thì giờ ta tăng CPU lên 24 core, RAM tăng lên 32GB -> máy chủ có thể phục vụ được số lượng kết nối truy vấn nhiều hơn.
  * Optimize ứng dụng, câu truy vấn. Ví dụ với câu truy vấn lấy dữ liệu tốn 5s để lấy được dữ liệu, sau đó mới trả lại tài nguyên cho hệ thống phục vụ các truy vấn khác. Máy chủ có thể đồng thời phục vụ 500 truy vấn dạng như vậy thì nếu ta tối ưu để truy vấn lấy dữ liệu chỉ tốn 1s => Máy chủ có thể phục vụ đồng thời nhiều truy vấn hơn.

* Scale out là giải pháp tăng số lượng server và dùng các giải pháp load-balacer để phân phối truy vấn ra nhiều server. Ví dụ bạn có 1 server có khả năng phục vụ 500 truy vấn. Nếu ta dựng thêm 5 server nữa có cấu hình tương tự, đặt thêm một LB phía trước để phân phối thì có khả năng hệ thống có thể phục vụ đc 5x500 truy vấn đồng thời.

MySQL Replication là một giải pháp scale out (tăng số lượng instance MySQL) nhưng không phải bài toán nào cũng dùng được. Các bài toán mà MySQL Replication sẽ giải quyết tốt:

* **Scale Read**
  * Scale Read thường gặp ở các ứng dụng mà số truy vấn đọc dữ liệu nhiều hơn ghi, tỉ lệ read/write có thể 80/20 hoặc hơn. Các ứng dụng thường gặp là báo, trang tin tức.
  * Với scale read ta sẽ chỉ có một Master instance phục vụ cho việc đọc/ghi dữ liệu. Có thể có một hoặc nhiều Slave instance chỉ phục vụ cho việc đọc dữ liệu.
  * Một số ứng dụng write nhiều (thương mại điện tử) cũng có sử dụng MySQL Replication để scale out hệ thống.

* **Data Report**

  * Một số hệ thống cho phép một số người (leader, manager, người làm report, thống kê, data) truy cập vào dữ liệu trên production phục vụ cho công việc của họ. Việc truy cập trực tiếp vào data production sẽ rất nguy hiểm vì:
    * Vô tình chỉnh sửa làm sai lệnh dữ liệu (nếu có quyền insert, update)
    * Vô tình thực thi các câu truy vấn tốn nhiều tài nguyên, thời gian truy vấn dài làm treo hệ thống.
  * Việc setup một máy chủ làm data report (application cũng sẽ không kết nối tới server này) làm giảm thiểu 2 rủi ro trên.

* **Real time backup**
  * Với cơ sở dữ liệu lớn việc backup không thể thực hiện thường xuyên được (hàng giờ, hàng phút). Với các ứng dụng giao dịch tài chính, thanh toán, TMDT nếu bị mất dữ liệu 1 giờ, 1 ngày thì thiệt hại sẽ rất lớn (máy chủ chính tự dưng bị hỏng). Real time backup là một giải pháp bổ sung cho offline backup, chạy đồng thời cả 2 phương pháp này để bảo đảm sự an toàn cho dữ liệu.

#### 2.1.2. Cách thức hoạt động

##### 2.1.2.1. Một số mô hình hoạt động

<p align="center">
  <img src="http://i.imgur.com/mevNr10.png">
  <br/>
</p>

Với cả hai mô hình ta luôn chỉ có 1 Master database phục vụ cho Write dữ liệu, có thể có một hoặc nhiều Slave database. Tùy từng mô hình ta có thể cấu hình mỗi web node connect vào một Slave DB tương ứng hoặc có thể dùng một LB đặt trước cụm Slave để LB tự động phân phối connection vào từng Slave DB theo thuật toán của LB.

<p align="center">
  <img src="http://i.imgur.com/etkJXxd.png">
  <br/>
</p>

##### 2.1.2.2. Cách thức hoạt động

Trên Master:

* Các kết nối từ web app tới Master DB sẽ mở một Session_Thread khi có nhu cầu ghi dữ liệu. Session_Thread sẽ ghi các statement SQL vào một file binlog (ví dụ với format của binlog là statement-based hoặc mix). Binlog được lưu trữ trong data_dir (cấu hình my.cnf) và có thể được cấu hình các thông số như kích thước tối đa bao nhiêu, lưu lại trên server bao nhiêu ngày.
* Master DB sẽ mở một Dump_Thread và gửi binlog tới cho I/O_Thread mỗi khi I/O_Thread từ Slave DB yêu cầu dữ liệu.

Trên Slave:

* Trên mỗi Slave DB sẽ mở một I/O_Thread kết nối tới Master DB thông qua network, giao thức TCP (với MySQL 5.5 replication chỉ hỗ trợ Single_Thread nên mỗi Slave DB sẽ chỉ mở duy nhất một kết nối tới Master DB, các phiên bản sau 5.6, 5.7 hỗ trợ mở đồng thời nhiều kết nối hơn) để yêu cầu binlog.
* Sau khi Dump_Thread gửi binlog tới I/O_Thead, I/O_Thread sẽ có nhiệm vụ đọc binlog này và ghi vào relaylog.
* Đồng thời trên Slave sẽ mở một SQL_Thread, SQL_Thread có nhiệm vụ đọc các event từ relaylog và apply các event đó vào Slave => quá trình replication hoàn thành.

<a name = "shard"></a>
### 2.2. Shard MySQL

<p align="center">
  <img src="https://s3-us-west-1.amazonaws.com/morpheus-staging/system/spud_media/188/original/tables1.jpg?1414453819">
  <br/>
</p>

Sharding là một tiến trình lưu giữ các bản ghi dữ liệu qua nhiều thiết bị để đáp ứng yêu cầu về sự gia tăng dữ liệu. Khi kích cỡ của dữ liệu tăng lên, một thiết bị đơn ( 1 database hay 1 bảng) không thể đủ để lưu giữ dữ liệu. Sharding giải quyết vấn đề này với việc mở rộng phạm vi theo bề ngang (horizontal scaling). Với Sharding, bạn bổ sung thêm nhiều thiết bị để hỗ trợ cho việc gia tăng dữ liệu và các yêu cầu của các hoạt động đọc và ghi.

Đối với những hệ thống có dữ liệu rất lớn thì đến một lúc nào đó, số dũ liệu trong bảng lên đến hàng triệu, việc query trở nên vô cùng ì ạch và tốn rất nhiều dung lượng bộ nhớ. Kỹ thuật sharding giúp ta giải quyết vấn đề này một cách nhanh chóng bằng cách chia nhỏ bảng hay db ra làm các phần khác nhau, chúng có cấu trúc dữ liệu giống nhau nhưng lưu các dữ liệu khác nhau để giảm tải thay cho việc chỉ dùng 1 bảng.

Ví dụ:

#### **Cách Pinterest scale MySQL của họ**

Bài viết được lược dịch từ [Sharding Pinterest: How we scaled our MySQL fleet](https://medium.com/@Pinterest_Engineering/sharding-pinterest-how-we-scaled-our-mysql-fleet-3f341e96ca6f) (viết từ 17/8/2015) và tham khảo từ bài [Pinterest đã thực hiện scaled MySQL của họ như thế nào](https://kipalog.com/posts/Pinterest-da-thuc-hien-scaled-MySQL-cua-ho-nhu-the-nao) (viết từ 22/3/2017).

##### a. Intro

> “Shard. Or do not shard. There is no try.”  —  Yoda

"Pinterest là công cụ khám phá dành cho tất cả những gì chúng ta hứng thú". Do đó, dữ liệu của Pinterest là vô hạn với cả triệu chủ đề, từ những bức hình giúp thư giãn đầu óc, hình đồ ăn, hình socola phủ dâu tây, hình Star Trek quotes,... "Nhìn từ góc độ dữ liệu, Pinterest là biểu đồ lớn nhất mô tả sự quan tâm của con người trên thế giới". Không chỉ xem hình hoặc video, user của Pin (gọi tắt là Piner) có thể thực hiện các hoạt động như follow, pin (save), share,... Đã có khoảng 50 tỷ Pins được lưu bởi Piner trong khoảng 1 tỷ boads (1 Piner có nhiều board, mỗi board có nhiều pins).

##### b. Why: Tại sao cần scale database?

Câu chuyện bắt đầu khi hệ thống gặp vấn đề. Vào năm 2011, Pinterest phát triển nhanh và mạnh mẽ với số lượng Piner tăng đột biến. Lượng truy cập ngày càng tăng, hệ thống giảm dần khả năng tải trọng với lượng dữ liệu ngày một nhiều. Khoảng tháng 9/2011, hệ thống gần như sập hoàn toàn cho dù đã sử dụng kỹ thuật NoSQL cũng như MySQL slave. Chỉ còn cách tái thiết kế cấu trúc cho toàn bộ hệ thống. Và lúc này, đội ngũ dev của Pinterest phải thay đổi góc nhìn về yêu cầu của hệ thống, từ đó thay đổi toàn bộ cấu trúc và xây dựng nền tảng mới.

##### c. Yêu cầu của hệ thống

Các yêu cầu đặt ra cho hệ thống lúc này là:

* Ổn định, hoạt động nhanh, dễ scale.
* Mọi thông tin mà Piner tạo ra phải được đảm bảo tính khả dụng.
* Khi query N pins trên board thì đảm bảo thứ tự trước sau.
* Update đơn giản nhất có thể.

##### d. Triết lý thiết kế

Một khi đã mở rộng cơ sở dữ liệu, chúng ta không thể sử dụng phép join, cài đặt khóa ngoại hay index cho toàn bộ CSDL nữa mà chỉ có thể dùng cho các CSDL con.

Đồng thời, load balancing cũng cần được đảm bảo. Việc di chuyển data lung tung, nhất là chuyển từng mục một rất dễ gây ra lỗi và khiến hệ thống phức tạp không cần thiết. Nếu nhất định phải di chuyển dữ liệu thì tốt nhất nên chuyển toàn bộ note ảo sang note vật lý.

##### e. Thực hành

Hệ thống sau khi xây dụng cần phải thỏa mãn các tiêu chí đã đề ra, đảm bảo perfoming tốt và dễ dàng sửa chữa. Nói cách khác là nó đừng có dở tệ, vậy nên team Pinterest đã chọn một công nghệ "chín" như chính nền tảng CSDL mà họ sử dụng vậy, MySQL. Câu hỏi đặt ra ở đây là, tại sao lại sử dụng một HQT CSDL xưa cũ như MySQL trong khi cách HQT mới đang lên như diều gặp gió (MongoDB, Cassandra, Membase,...). Câu trả lời chính là do những HQT mới được cập nhật nhiều tính năng (như auto-scaling) khó có khả năng phát triển về lâu về dài và dễ crash với yêu cầu cao.

Bên cạnh đó, tác giả có nói:

```
Tui vẫn thường khuyên mọi người nên khởi đầu mà tránh dùng các công cụ quá mới mẻ và tích hợp nhiều tính năng. Hãy cố gắng chỉ dùng MySQL thôi, tin tui đi, tui có kinh nghiệm đau thương về vụ này đó.
```

* Cấu hình được sử dụng là master-master, bắt đàu với 8 servers EC2, mỗi server EC2 chạy trên 1 instance MySQl và có 1 EC2 đi kèm, nhằm tránh trường hợp lỗi hệ thống khi có 1 server bị sập. Ví dụ, server 1A và 1B là 1 cặp master-master, lúc đầu 1A là master chính, 1B support cho 1A. Khi 1A die, 1B đảo chính, lên làm master. Khi 1A quay lại, nó mất ngôi và trở thành support cho 1B.

<p align = "center">
  <img src="https://cdn-images-1.medium.com/max/1600/0*pImQYoxO9l9J3LG8.png">
  <br/>
</p>

* Mọi thao tác read/write chỉ tác động lên master. Mỗi server EC2 có thể chứa nhiều dữ liệu:

<p align = "center">
  <img src="https://cdn-images-1.medium.com/max/1600/0*dB5aSiZrInFvbefe.png">
  <br/>
</p>

Hãy chú ý rằng mỗi database được đặt tên độc nhất vì chúng là một phần của dữ liệu chính. Họ đã quyết định rằng, một khi chia dữ liệu vào một shard, nó sẽ ở đó vĩnh viễn, không bao giờ bị chuyển đi nơi khác.

* Họ có một bảng lưu thông tin về nơi lưu trữ các shard trên ZooKeepala

```scala
[{“range”:     (0,511), “master”: “MySQL001A”, “slave”: “MySQL001B”},
 {“range”: (512, 1023), “master”: “MySQL002A”, “slave”: “MySQL002B”},
    ...
 {“range”: (3584, 4095), “master”: “MySQL008A”, “slave”: “MySQL008B”}]
```

// *`(0,511)` tương ứng `db00000 to db00511`*

* Tạo universally unique IDs (UUID) cho mọi project để có thể phân phối data được viết vào các shard một cách chính xác và balance nhất.
  * Họ tạo UUID là 1 số 64 bit, dùng cho mọi query từ bên ngoài vào, có cấu trúc:
  
  ```scala
  UUID = (shard ID << 46) | (type ID << 36) | (local ID << 0)
  ```

  Với:
* `shard ID:` là 1 số 16bit, có vai trò là ID của shard.
* `type ID:` là 1 số 10bit, có vai trò chỉ ra type củaobject. Ví dụ nếu type ID = 1 có nghĩa object type là Pin, type ID = 2 ứng với object type là Board chẳng hạn.
* `local ID:` là 1 số 36 bit, có vai trò là ID của records bên trong shard, có giá trị auto increment.

Từ công thức bên trên, có thể thấy UUID được tạo bằng cách:

```php
(Dịch trái shard ID 46 bit) bitwise OR (Dịch trái type ID 36 bit) bitwise OR (Dịch trái local ID 0 bit)
```

Ví dụ với URL: https://www.pinterest.com/pin/241294492511762325/, thì UUID nhận được là 241294492511762325. Tiến hành decompose UUID theo cách sau:

```scala
Shard ID = (241294492511762325 >> 46) & 0xFFFF = 3429
Type ID  = (241294492511762325 >> 36) & 0x3FF = 1  //type Pin -> query vào table pins
Local ID = (241294492511762325 >>  0) & 0xFFFFFFFFF = 7075733
```

Từ công thức trên, có thể thấy được:

* `Shard ID` được decompose từ việc dịch phải 46bit UUID, sau đó bitwise AND với 0xFFFF (hay là 0b1111111111111111). Nghĩa là cắt đúng vị trí của Shard ID được dịch trái ở công thức tạo UUID bên trên. Trong trường hợp này thì việc bitwise AND với 0xFFFF là hơi thừa. Tuy nhiên nó lại có ích nếu như Shard ID ko còn đứng ở phía bên phải nhất của 64bit UUID nữa.
* `Type ID` và `Local ID` được decompose tương tự, và cần phải bitwise AND tương ứng với 1 số 10bit và 1 số 36bit toàn 1. Như vậy là mọi thông tin cần thiết đã được decompose một cách toàn vẹn, mà lại còn nhanh nữa chứ. Tính toán trên bit mà.

Sau khi đã có đầy đủ thông tin, chỉ việc query:

```sql
conn = MySQLdb.connect(host = ”MySQL007A”)
conn.execute(“SELECT data FROM db03429.pins where local_id = 7075733”)
```

*// Host MySQL007A được nhặt ra từ shard ID 3429, có range nằm trong {“range”: (3072, 3583), “master”: “MySQL007A”, “slave”: “MySQL007B”},, nên nó chọn host MySQL007A để query thôi.*

##### f. Cấu trúc bảng

Với lượng dữ liệu lớn và khó phân loại, Pinterest liệu có phải xây dựng 1 CSDL lớn và đồ sộ với khoảng 15-20 thuộc tính không?

Câu trả lời là không. Ngược lại là khác. Cấu trúc bảng của Pinterest chỉ gồm:

```sql
CREATE TABLE pins (
  local_id INT PRIMARY KEY AUTO_INCREMENT,
  data TEXT,
  ts TIMESTAMP DEFAULT CURRENT_TIMESTAMP
) ENGINE = InnoDB;
```

Ví dụ, một đối tượng Pin sẽ trông như thế này:

```python
{“details”: “New Star Wars character”, “link”: “http://webpage.com/asdf”, “user_id”: 241294629943640797, “board_id”: 241294561224164665, …}
```

Tại sao? Vì giả sử nếu phải thêm 1 column mới cho 1 table theo cách tiếp cận cũ, thì việc alter table sẽ rất nặng (do phải alter cho tất cả các table tương ứng của các shard). Tuy nhiên với cách làm này, thì khỏi cần alter gì cả, chỉ cần khai báo ở tầng app giá trị default cho column mới mỗi khi muốn đọc ra thôi. Pinterest hơn 3 năm rồi (tính tới thời điểm bài gốc được đăng) chưa từng phải alter table.

* Khi insert 1 record mới, họ chỉ định ghi nó vào shard ID nào, type ID là gì. Sau khi record được insert rồi, nó sẽ trả về local ID, lúc này sẽ kết hợp với Shard ID và Type ID để cho ra UUID theo cách bên trên. Quá lợi hại.

<a name = "task_message"></a>
# IV. Task queue và Message queue

<a name = "mess"></a>
## 1. Message queue

Hàng đợi thông điệp nhận, giữ và gửi tin nhắn. Nếu một tác vụ cần thời gian lớn để thực hiện nội tuyến, bạn có thể dùng hàng đợi thông điệp với luông công việc như sau:

* Phần mềm đăng một công việc lên hàng đợi, gửi thông báo tới những nhân viên có liên quan đến công việc đó.
* Nhân viên lấy công việc từ hàng đợi, thực hiện nó, sau đó đánh dấu công việc thành đã hoàn thành.

Khi đó, người dùng sẽ không bị block và công việc thì vẫn chạy ngầm ở dưới nền. Trong quá trình đó, hệ thống cho phép client thực hiện một vài hành động nhỏ để làm như tác vụ đó đã hoàn thành. Ví dụ khi bạn đăng 1 tweet trên twitter, tweet đó có thể xuất hiện ngay trên timeline của bạn, nhưng cần một thời gian nhỏ để bài đăng đó đến được với những người follow bạn.

* [Redis](https://redis.io/) là một mã nguồn mở (được cấp phép BDS), lưu trữ cấu trúc dữ liệu trong bộ nhớ, được sử dụng làm cơ sở dữ liệu, bộ đệm và môi giới tin nhắn. Nó hỗ trợ các cấu trúc dữ liệu như chuỗi, băm, danh sách, bộ, bộ được sắp xếp với các truy vấn phạm vi, bitmap, hyperloglog, chỉ mục không gian địa lý với các truy vấn và luồng bán kính. Redis có thể hữu dụng như một nhà trung gian tin nhắn cỡ nhỏ nhưng thông điệp có thể bị mất.
* [RabbitMQ](https://www.rabbitmq.com/) là một phần mềm trung gian tin nhắn mã nguồn mở được phát triển rộng rãi. RabbitMQ rất nhẹ và dễ triển khai trên cơ sở và trên đám mây. Nó hỗ trợ nhiều giao thức nhắn tin. RabbitMQ có thể được triển khai trong các cấu hình phân tán và liên kết để đáp ứng các yêu cầu có tính sẵn sàng cao, quy mô cao. RabbitMQ phổ biến nhưng yêu cầu thiết bị của bạn phải tương thích với "AMQP" protocol và tự quản lý nodes của mình.
* [Amazon SQS](https://aws.amazon.com/sqs/) là hệ thống quản lý hàng đợi thông điệp hoàn thiện cho phép bạn tách rời và chia tỷ lệ microservice, hệ thống phân tán và ứng dụng không có máy chủ. ASQS được khuyến khích sử dụng nhưng có thể có thời gian phản hồi cao và nhiều nguy cơ thông điệp bị gửi nhiều hơn 1 lần.

<a name = "task"></a>
## 2. Task queue

Hàng đợi nhiệm vụ nhận nhiệm vụ và các dữ liệu liên quan, chạy chúng lên sau đó gửi lại kết quả. Nó hỗ trợ việc lên lịch và chạy các tính toán chuyên sâu trong nền.

**Celery** có hỗ trợ lên lịch và chủ yếu hỗ trợ python.

<a name = "credit"></a>
# V. Tham khảo

* [System disign Primer](https://github.com/donnemartin/system-design-primer#message-queues)
* [Pinterest đã thực hiện scaled MySQL của họ như thế nào](https://kipalog.com/posts/Pinterest-da-thuc-hien-scaled-MySQL-cua-ho-nhu-the-nao)
* [Sharding Pinterest: How we scaled our MySQL fleet](https://medium.com/@Pinterest_Engineering/sharding-pinterest-how-we-scaled-our-mysql-fleet-3f341e96ca6f)
* [Database và Microservices](http://bloghoctap.com/technology/web-scalability-101-database-va-microservices.html)
* [1 số vấn đề về MySQL Performance](https://kipalog.com/posts/1-so-van-de-ve-MySQL-Performance)