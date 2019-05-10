---
  title: "Cơ bản về Load balancer & NGINX"
---

## MỤC LỤC

---

* [Load balancer](#load)
  * [Định nghĩa.](#l1)
  * [Giao thức.](#l5)
  * [Làm thế nào để Load balancer chọn máy chủ backend?](#l2)
    * [Các thuật toán balancer.](#l2.1)
    * [Health check.](#l2.2)
  * [Load balancer xử lý trạng thái.](#l3)
  * [Load balancer dự phòng.](#l4)
* [NGINX](#nginx)
  * [NGINX là gì?](#n1)
  * [Bối cảnh & mô hình xử lý](#n2)
* [Nguồn tham khảo](#3)

---

<a name = "load"></a>
# LOAD BALANCER

<a name = "l1"></a>
## 1. Định nghĩa

Load balancing là một thành phần quan trọng của cơ sở hạ tầng thường được sử dụng để cải thiện hiệu suất và độ tin cậy của các trang web, các ứng dụng, cơ sở dữ liệu và các dịch vụ khác bằng cách phân phối khối lượng công việc trên nhiều máy chủ.

Một cơ sở hạ tầng web không có Load balancing có thể trông giống như sau:

<p align="center">
  <img src="https://viblo.asia/uploads/9ef3bb43-3d54-49fa-90be-d464751e5b47.png">
  <br/>
</p>

Trong ví dụ này, người sử dụng kết nối trực tiếp đến máy chủ web, tại yourdomain.com. Nếu máy chủ web duy nhất này down, người sử dụng sẽ không thể truy cập vào trang web. Ngoài ra, nếu nhiều người dùng cố gắng truy cập vào máy chủ cùng một lúc thì nó không thể xử lý tải, họ có thể gặp thời gian tải chậm hoặc không thể kết nối.

Đây là điểm có thể khắc phục bằng một load balancer và ít nhất một máy chủ web bổ sung trên backend. Thông thường, tất cả các máy chủ phụ trợ sẽ cung cấp nội dung giống hệt nhau để người dùng nhận được nội dung phù hợp bất kể là máy chủ nào đáp ứng.

<p align="center">
  <img src="https://viblo.asia/uploads/65fad7fe-c1b6-4798-8bdd-94232331f8f8.png">
  <br/>
</p>

Trong ví dụ minh họa ở trên, người dùng truy cập vào load balancer và nó sẽ chuyển tiếp yêu cầu của người sử dụng đến một máy chủ phụ trợ, sau đó đáp ứng trực tiếp yêu cầu của người dùng. Trong kịch bản này, việc truy cập đến duy nhất 1 load balance cũng khiến server đáp ứng chậm. Điều này có thể được giảm nhẹ bằng cách giới thiệu một load balance thứ hai, nhưng trước khi chúng ta thảo luận về điều đó, hãy tìm hiểu cách thức load balance hoạt động.

<a name = "l4"></a>
## 2. Những loại giao thức load balancers có thể xử lý:

Quản trị Load balancer tạo quy định chuyển tiếp đối với bốn loại giao thức chính:

* HTTP - Chuẩn HTTP balancing chỉ đạo yêu cầu dựa trên cơ chế HTTP chuẩn. Load Balancer đặt X-Forwarded-For, X-Forwarded-Proto, và tiêu đề X-Forwarded-Port để cung cấp cho các thông tin backends về các yêu cầu ban đầu.
* HTTPS - HTTPS balancing với các chức năng tương tự như HTTP balancing, với sự bổ sung của mã hóa. Mã hóa được xử lý theo một trong hai cách: hoặc là với passthrough SSL duy trì mã hóa tất cả con đường đến backend hoặc chấm dứt SSL mà đặt gánh nặng giải mã vào load balancer nhưng gửi lưu lượng được mã hóa đến back end.
* TCP - Đối với các ứng dụng không sử dụng HTTP hoặc HTTPS, lưu lượng TCP cũng có thể được cân bằng. Ví dụ, lượng truy cập vào một cụm cơ sở dữ liệu có thể được lan truyền trên tất cả các máy chủ.
* UDP - Gần đây, một số load balancer đã thêm hỗ trợ cho cân bằng tải giao thức internet lõi như DNS và syslogd sử dụng UDP.

Những quy tắc chuyển tiếp sẽ xác định các giao thức và cổng vào load balancer và bản đồ chúng đến các giao thức và cổng load balancer sẽ sử dụng để định tuyến lưu lượng trên backend.

<a name = "l2"></a>
## 3. Làm thế nào để load balancer chọn máy chủ backend?

Load balancers chọn máy chủ để chuyển tiếp yêu cầu dựa trên sự kết hợp của hai yếu tố. Lần đầu tiên sẽ đảm bảo rằng bất kỳ máy chủ được lựa chọn có thể thực sự đáp ứng yêu cầu và sau đó sử dụng một quy tắc được cấu hình sẵn để lựa chọn trong số đó.

<a name = "l2.1"></a>
### 3.1. Health Checks

Load balancer chỉ chuyển tiếp lưu lượng đến "healthy" backend server. Để theo dõi sức khỏe của một backend server, kiểm tra sức khỏe thường xuyên bằng cách cố gắng kết nối với backend server sử dụng giao thức và cổng được định nghĩa bởi các quy tắc chuyển tiếp để đảm bảo rằng các máy chủ đang lắng nghe. Nếu một máy chủ không kiểm tra sức khỏe, và do đó không thể phục vụ yêu cầu, nó sẽ tự động loại bỏ khỏi vùng chứa, và request sẽ không được chuyển tiếp đến nó cho đến khi nó đáp ứng việc kiểm tra sức khỏe một lần nữa.

<a name = "l2.2"></a>
### 3.2. Các thuật toán load balancer

Các thuật toán load balancer được sử dụng xác định của máy chủ lành mạnh trên backend sẽ được lựa chọn. Một số các thuật toán thường được sử dụng là:

* Round Robin - Round Robin có nghĩa là các máy chủ sẽ được lựa chọn theo tuần tự. Bộ load balancer sẽ chọn máy chủ đầu tiên trong danh sách của mình đối với yêu cầu đầu tiên, sau đó di chuyển xuống trong danh sách theo thứ tự, bắt đầu lại ở đầu trang khi đi đến cuối cùng.
* Least Connections - load balancer sẽ chọn máy chủ với các kết nối ít nhất.
* Source - Với các thuật toán mã nguồn, load balancer sẽ chọn máy chủ để sử dụng dựa trên một hash của IP nguồn của yêu cầu, chẳng hạn như địa chỉ IP của người truy cập. Phương pháp này đảm bảo rằng một người dùng cụ thể sẽ luôn kết nối với cùng một máy chủ.

Các thuật toán có người quản lý khác nhau tùy thuộc vào công nghệ load balancer sử dụng.

<a name = "l3"></a>
## 4. Làm thế nào để load balancer xử lý trạng thái?

Một số ứng dụng yêu cầu người dùng tiếp tục kết nối đến cùng một backend server. Một thuật toán mã nguồn tạo ra một mối quan hệ dựa trên thông tin IP khách hàng. Một cách khác để đạt được điều này ở mức ứng dụng web là thông qua sticky sessions, nơi load balancer đặt một cookie và tất cả các requests từ sessions hướng đến một máy chủ vật lý.

<a name = "l4"></a>
## 5. Load balancer dự phòng

Để loại bỏ việc load balancer như một điểm truy cập duy nhất, một load balancer thứ hai có thể được kết nối với cái đầu tiên để tạo thành một cụm. Mỗi load balancer là đều có khả năng phát hiện lỗi và phục hồi.

<p align="center">
  <img src="https://viblo.asia/uploads/5f72fc77-5ba3-4dc5-9d73-8ff37f33fff9.png">
  <br/>
</p>

Trong trường hợp load balancer chính bị lỗi, DNS phải đưa người dùng đến các bộ load balancer thứ hai. Bởi vì thay đổi DNS có thể mất một lượng thời gian đáng kể để được tải lên Internet và để làm cho chuyển đổi dự phòng này tự động, nhiều quản trị viên sẽ sử dụng hệ thống, cho phép linh hoạt địa chỉ IP Remapping, chẳng hạn như các floating IPs. Theo yêu cầu địa chỉ IP Remapping giúp loại bỏ các vấn đề tuyên truyền, bộ nhớ đệm vốn có trong những thay đổi DNS bằng cách cung cấp một địa chỉ IP tĩnh có thể được dễ dàng ánh xạ lại khi cần thiết. Tên miền có thể duy trì liên kết với các địa chỉ IP, trong khi các địa chỉ IP của chính nó được di chuyển giữa các máy chủ.

Đây là cách một cơ sở hạ tầng sử dụng Floating IPs có thể xem xét:

<p align="center">
  <img src="https://viblo.asia/uploads/7f9b9acf-6ba0-4f4d-96c1-272cc7beb95c.gif">
  <br/>
</p>

<a name = "nginx"></a>
# NGINX

<a name = "n1"></a>
## 1. NGINX là gì?

<p align="center">
  <img src="https://quiksite.com/wp-content/uploads/2016/09/Nginx-Logo-02.png">
  <br/>
</p>

NGINX dẫn đầu gói hiệu năng web nhờ cách phần mềm được thiết kế. Trong khi nhiều máy chủ web và máy chủ ứng dụng sử dụng kiến ​​trúc dựa trên luồng hoặc xử lý đơn giản, NGINX nổi bật với kiến ​​trúc hướng sự kiện phức tạp cho phép nó mở rộng tới hàng trăm ngàn kết nối đồng thời trên phần cứng hiện đại.

Bài viết: [Inside Nginx](https://www.nginx.com/resources/library/infographic-inside-nginx/).

Để hiểu rõ hơn về thiết kế này, bạn cần hiểu cách NGINX hoạt động. Có một quy trình công việc trên mỗi lõi để sử dụng hiệu quả tài nguyên phần cứng, khả năng xen kẽ nhiều kết nối trong một quy trình công việc duy nhất và khả năng chuyển từ kết nối này sang kết nối kia gần như ngay lập tức khi lưu lượng mạng tăng. Kết hợp tất cả lại với nhau, bạn tạo ra công cụ phân phối ứng dụng HTTP có thể mở rộng quy mô lớn là NGINX.

<p align="center">
  <img src="https://www.nginx.com/wp-content/uploads/2015/04/nginx_architecture_thumbnail.png">
  <br/>
</p>

<a name = "n2"></a>
## 2. Bối cảnh và mô hình xử lý của NGINX

<p align="center">
  <img src="https://www.nginx.com/wp-content/uploads/2015/06/infographic-Inside-NGINX_process-model.png">
  <br/>
</p>

NGINX có một quy trình tổng thể (thực hiện các hoạt động đặc quyền như đọc cấu hình và liên kết với các cổng) và một số quy trình công việc và trợ giúp.

```python
># service nginx restart
* Restarting nginx
# ps -ef --forest | grep nginx
root     32475     1  0 13:36 ?        00:00:00 nginx: master process /usr/sbin/nginx
                                                -c /etc/nginx/nginx.conf
nginx    32476 32475  0 13:36 ?        00:00:00  _ nginx: worker process
nginx    32477 32475  0 13:36 ?        00:00:00  _ nginx: worker process
nginx    32479 32475  0 13:36 ?        00:00:00  _ nginx: worker process
nginx    32480 32475  0 13:36 ?        00:00:00  _ nginx: worker process
nginx    32481 32475  0 13:36 ?        00:00:00  _ nginx: cache manager process
nginx    32482 32475  0 13:36 ?        00:00:00  _ nginx: cache loader process
```

Trên máy chủ bốn lõi này, quy trình chủ NGINX tạo ra bốn quy trình worker và một vài quy trình trợ giúp bộ đệm để quản lý bộ đệm nội dung trên đĩa.

## 3. Tại sao kiến trúc lại quan trọng?

Cơ sở cơ bản của bất kỳ ứng dụng Unix nào là luồng hoặc tiến trình. (Từ phối cảnh hệ điều hành Linux, các luồng và tiến trình hầu như giống hệt nhau; sự khác biệt chính là mức độ chúng chia sẻ bộ nhớ.) Một luồng hoặc tiến trình là một tập hợp các lệnh mà hệ điều hành có thể lên lịch để chạy trên CPU cốt lõi. Hầu hết các ứng dụng phức tạp chạy song song nhiều luồng hoặc tiến trình vì hai lý do:

* Họ có thể sử dụng nhiều lõi tính toán hơn cùng lúc.
* Luồng và tiến trình làm cho việc thực hiện các thao tác song song rất dễ dàng (ví dụ: để xử lý nhiều kết nối cùng lúc).

Tiến trình và luồng tiêu thụ tài nguyên. Mỗi cái đều sử dụng bộ nhớ và các tài nguyên HĐH khác, và chúng cần được hoán đổi trên và ngoài lõi (một thao tác gọi là chuyển đổi ngữ cảnh). Hầu hết các máy chủ hiện đại có thể xử lý đồng thời hàng trăm luồng hoặc xử lý hoạt động nhỏ, nhưng hiệu suất giảm nghiêm trọng khi bộ nhớ cạn kiệt hoặc khi tải I/O cao gây ra một khối lượng lớn các chuyển đổi ngữ cảnh.

Cách phổ biến để thiết kế các ứng dụng mạng là gán một luồng hoặc xử lý cho mỗi kết nối. Kiến trúc này đơn giản và dễ thực hiện, nhưng nó không mở rộng khi ứng dụng cần xử lý hàng ngàn kết nối đồng thời.

## 4. NGINX hoạt động như thế nào?

NGINX sử dụng mô hình quy trình dự đoán được điều chỉnh theo tài nguyên phần cứng có sẵn:

* Quá trình *master* thực hiện các hoạt động đặc quyền như đọc cấu hình và liên kết với các cổng, sau đó tạo ra một số lượng nhỏ các quy trình con (ba loại tiếp theo).
* Quá trình *cache loader* chạy khi khởi động để tải bộ đệm dựa trên đĩa vào bộ nhớ, rồi thoát. Nó được lên kế hoạch bảo thủ, do đó nhu cầu tài nguyên của nó là thấp.
* Quá trình *quản lý cache* chạy định kỳ và cắt các mục từ bộ đệm đĩa để giữ chúng trong các kích thước được cấu hình.
* Các quy trình *worker* làm tất cả các công việc. Chúng xử lý các kết nối mạng, đọc và ghi nội dung vào đĩa và liên lạc với các máy chủ ngược dòng.

Cấu hình NGINX được khuyến nghị trong hầu hết các trường hợp - chạy một tiến trình worker trên mỗi lõi CPU - giúp sử dụng hiệu quả nhất các tài nguyên phần cứng. Bạn định cấu hình nó bằng cách bao gồm lệnh worker_auto điều hướng trong cấu hình:

>worker_processes auto;



<a name = "3"></a>
# Nguồn tham khảo

* [Digital Ocean](https://www.digitalocean.com/)
* [Định nghĩa Load balancing](https://viblo.asia/p/dinh-nghia-ve-load-balancing-07LKXEJkZV4)
* [Nginx](https://www.nginx.com/resources/glossary/nginx/)
* [Cơ bản về Nginx](https://www.hostinger.vn/huong-dan/nginx-la-gi-no-hoat-dong-nhu-the-nao/)