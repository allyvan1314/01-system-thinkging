---
    title: "Caching with NGINX"
    tags = "NGINX"
---

# TÌM HIỂU CACHING VÀ CÁCH TĂNG TỐC WEBSITE TRÊN NGINX

## MỤC LỤC

1. [Tìm hiểu caching trên nginx](#head)
    * [Cách hoạt động](#CHD)
2. [Cấu hình FastCGI Cache (Dynamic Cache)](#2)
3. [Cấu hình lưu Cache trên RAM](#3)
4. [Cấu hình Browser Cache (static cache)](#4)
5. [Bật nén Gzip Compression](#5)
6. [Thuật toán hỗ trợ cache](#6)
7. [Tài liệu tham khảo](#7)

## INTRO

*Caching là cách tăng tốc website hiệu quả nhất* - không ai có thể phủ nhận điều này. Khi cấu hình bật cache tốc độ lướt web có thể cải thiện nhiều lần. Được mệnh danh là webserver tốc độ cao, Nginx đã tích hợp sẵn tính năng tạo cache nội dung động (Dynamic Cache) và cache nội dung tĩnh (Browser Cache) phù hợp với nhiều hệ thống Web Server có kiến trúc khác nhau. Ngoài ra, nginx còn hỗ trợ nén nội dung với Gzip Compression trước khi gửi đến cho trình duyệt nhằm giảm bớt băng thông tải web.

Trên hết những module này đã được mặc định trong nginx, bạn không phải cài cắm thêm gì nữa chỉ việc cấu hình sử dụng thôi.

<a name = "head"></a>
## 1. TÌM HIỂU CACHING TRÊN NGINX

Đầu tiên mình phải khẳng định Nginx là một Webserver thực thụ, vì nó rất mạnh với khả năng tạo cache mà mọi người biết đến nó như một hệ thống Caching nhiều hơn.

Chúng ta cần phân biệt **Proxy Cache** và **FastCGI Cache**. Đây là 2 module khác nhau của nginx dùng để xây dựng hệ thống cache cho từng mô hình máy chủ khác nhau.

* **Proxy Cache** dùng để cấu hình cache giao thức http, trong mô hình này nginx đứng ngoài cùng đóng vai trò làm Proxy Server hứng request từ client. Khi chạy theo mô hình này hệ thống website thường gồm ít nhất hai máy chủ, hiện có nhiều người triển khai mô hình này với Apache làm backend, cách hoạt động này cũng giống mô hình sử dụng Varnish Cache, Squid.
* **FastCGI Cache** hoạt động theo cách thức khác, nó cache lại các response mà trình thông dịch (PHP-FPM) hoặc  Application Server khác đứng ở backend trả lại cho Webserver. Hầu hết với những website cá nhân nhỏ, người dùng thường nhồi nhét tất cả trên một server mình khuyên nên dùng FastCGI Cache là tốt nhất.

<a name = "CHD"></a>
### CÁCH HOẠT ĐỘNG

Như các bạn đã biết khi User lần đầu truy cập website, webserver nginx sẽ kiểm tra trong RAM có cache lại thông tin gì mà người dùng yêu cầu hay không nếu không có nó sẽ làm việc với backend để lấy thông tin người dùng yêu cầu. Sau khi xử lý xong backend sẽ trả kết quả lại cho nginx.

<p align="center">
  <img src="https://www.thuysys.com/wp-content/uploads/nginx-cache0.jpg">
  <br/>
</p>

Nhận được response từ backend nginx không gửi luôn cho User mà lưu lại trên File System một bản gọi là Cache File, đồng thời nó cũng lấy Key (Cache Key) tương ứng với mỗi cache file để lưu lên RAM sau đó mới trả kết quả lại cho User.

<p align="center">
  <img src="https://cdn.shortpixel.ai/client/q_glossy,ret_img,w_729/https://www.thuysys.com/wp-content/uploads/nginx-cache2.jpg">
  <br/>
</p>

Lần tiếp theo người dùng truy cập website, nginx cũng kiểm tra trên RAM trước nó so sánh request tương ứng với Key nào, nếu khớp nó sẽ lấy Cache file tương ứng với Key đó trong File System rồi trả kết quả cho người dùng. Như vậy sẽ giảm bớt tài nguyên khi không phải làm việc với backend nữa.

<p align="center">
  <img src="https://cdn.shortpixel.ai/client/q_glossy,ret_img,w_729/https://www.thuysys.com/wp-content/uploads/nginx-cache3.jpg">
  <br/>
</p>

<a name = "2"></a>
## 2. CẤU HÌNH FASTCGI CACHE (DYNAMIC CACHE)

Tiếp theo mình sẽ đi vào chi tiết cấu hình FastCGI để cache nội dung động (PHP Scripts). Mô hình Webserver trong bài này sẽ bao gồm các thành phần chính Nginx + FastCGI Cache + PHP-FPM.

Bạn mở file cấu hình nginx:

    vi /etc/nginx/nginx.conf

Thêm vào block http {…} hai dòng sau:

    fastcgi_cache_path /etc/nginx/cache levels=1:2 keys_zone=supercache:10m max_size=1000m inactive=60m;
    fastcgi_cache_key "$scheme$request_method$host$request_uri";

Đây là hai dòng quan trọng khai báo cấu hình FastCGI Cache.

**fastcgi_cache_path**

Chỉ ra đường dẫn chứa cache file là ```/etc/nginx/cache``` với các giá trị kèm theo.

* ```levels``` : giá trị này là quy tắc đặt tên và phân cấp thư mục cache, bạn xem hình bên dưới để hiểu thêm với ```levels=1:2``` và ```levels=2:4```.
* ```level=1:2```

<p align="center">
  <img src="https://www.thuysys.com/wp-content/uploads/levels12.png">
  <br/>
</p>

* ```level=2:4```

<p align="center">
  <img src="https://www.thuysys.com/wp-content/uploads/levels24.png">
  <br/>
</p>

* ```keys_zone``` đặt tên cho cache zone là supercache có dung lượng là 10m, bạn chú ý một chút về đơn vị dung lượng k/K là Kilobytes, m/M là Megabytes.
* ```max_size``` kích thước tối đa của toàn bộ cache là 1000m
* ```inactive``` nếu một response không được sử dụng trong thời gian 60 phút thì nó sẽ bị xóa khỏi thư mục chứa cache.

**fastcgi_cache_key**

Định dạng Key (khóa) của cache file theo ```$scheme$request_method$host$request_uri``` có tác dụng để phân biệt các cache file với nhau. Nếu mở cache file bất kỳ bạn sẽ thấy định dạng của nó như bên dưới.

    KEY: httpGETwww.thuysys.com/domain-hosting/wordpress-co-ban/tim-hieu-chmod-chown-cach-sua-loi-phan-quyen-wordpress-tren-linux.html

* ```$scheme```: http
* ```$request_method```: GET
* ```$host```: www.thuysys.com
* ```$request_uri```: /domain-hosting/wordpress-co-ban/tim-hieu-chmod-chown-cach-sua-loi-phan-quyen-wordpress-tren-linux.html

Bạn có thể thay đổi định dạng Key bằng cách thêm bớt các biến sao cho đúng ý mình là được.

Đấy là ý nghĩa các thông số cấu hình FastCGI Cache cho nginx. Giờ bạn để ý cho mình keys_zone và max_size tại sao lại có giá trị lần lượt là 10m và 1000m, như vậy thừa chăng, chúng có ý nghĩa gì ?

<p align="center">
  <img src="https://www.thuysys.com/wp-content/uploads/cache-file.png">
  <br/>
</p>

Hình bên trên là nội dung của một cache file thông thường, chỗ màu đỏ chính là Cache Keys, khi được Hash Key thành chuỗi mã hóa nó sẽ được lưu trên RAM.

```py
Theo tài liệu nginx cung cấp thì 1m RAM có thể lưu trữ khoảng 8000 key với keys_zone 10m bạn sẽ lưu được 80.000 key tương đương với 80.000 cache files trong file system. Từ đây bạn có thể tính được dung lượng của keys_zone dựa vào số lượng cache file sinh ra cho site của mình. Với các web vài nghìn truy câp/ngày thì cũng còn khướt mới đạt tới còn số 80.000, thứ nữa cache file còn bị xóa bởi inactive = 60m nữa nên bạn không cần để cao làm gì cho tốn bộ nhớ.

Riêng max_size là tổng dùng lượng cache file lưu trên ổ cứng. Bạn áng chừng con số phù hợp là được, vài GB đĩa cứng có nhằm nhò gì!
```

Một chú ý nữa, keys_zone có thể được dùng chung cho nhiều virtual host tương đương với nhiều website hoặc tách biệt ra không vấn đề gì cả.

Sang bước cấu hình tiếp theo, bạn thêm vào block server {…} nội dung sau.

```python
Server
{
    # Không lưu cache khi truy cập khu vực quản trị wordpress.
    set $no_cache 0;
    if ($request_uri ~* "/(wp-admin/)")
    {
        set $no_cache 1;
    }
    # Cấu hình FastCGI Cache cho PHP Scripts
    location ~ .php$
    {
        try_files $uri =404;
        fastcgi_cache supercache;
        fastcgi_cache_valid 200 60m; # Chỉ cache lại các response có code là 200 OK trong 60 phút
        fastcgi_cache_methods GET HEAD; # Áp dụng với các phương thức GET HEAD
        add_header X-Fastcgi-Cache $upstream_cache_status; # Thêm vào header trạng thái cache MISS (chưa cache), HIT (đã cache)
        fastcgi_cache_bypass $no_cache; # no_cache=1 không lấy dữ liệu từ cache với request_uri bắt đầu bằng wp-admin.
        fastcgi_no_cache $no_cache; # Khi no_cache=1 đồng thời cũng không lưu cache với response trả về.
    # Kết nối fastcgi_pass với php-fpm
        fastcgi_pass unix:/var/run/php-fpm/php5-fpm.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        include fastcgi_params;
    }
}
```

Cấu hình xong, khởi động lại webserver ```systemctl restart nginx``` rồi dùng trinh duyệt để xem kết quả, tốc độ lướt web sẽ làm bạn ngạc nhiêu đấy.

<a name = "3"></a>
## 3. CẤU HÌNH LƯU CACHE TRÊN RAM

Tiếp theo chúng ta sẽ cấu hình mount File System (thư mục chứa cache file) lên RAM, đây là cách tạo **RAM Disk** trên linux rất hay, bạn nào dùng VPS có RAM nhiều một chút có thể thử cách này.

Nếu cài nginx trên Ubuntu hay CentOS mở file ```vi /etc/fstab``` ra thêm vào dòng sau:

    tmpfs /etc/nginx/cache tmpfs defaults,size=100M 0 0

Bạn nào không rõ về cách mount trên linux và fstab là gì thì search google đọc thêm, bạn cần lưu ý giá trị ```size=100M``` nó có nghĩa thư mục mount sẽ chiếm tối đa 100M trên RAM, bộ nhớ ít thì để như vậy thôi.

Ấn [:wq để lưu file](https://www.thuysys.com/tools/cach-dung-trinh-soan-thao-vivim-editor-linux.html), chạy thêm lệnh mount -a để tự mount.

Để kiểm tra lại kết quả bạn gõ lệnh df -af | grep tmpfs như hình bên dưới là ok.

<p align="center">
  <img src="https://www.thuysys.com/wp-content/uploads/test-fastcgi-on-RAM.png">
  <br/>
</p>

<a name = "4"></a>
## 4. CẤU HÌNH BROWSER CACHE (STATIC CACHE)

Muốn cấu hình Web Browser cache nội dung tĩnh bạn thêm tiếp block location con nữa vào trong block server {…} với nội dung:

    location ~* \.(js|css|png|jpg|jpeg|gif|ico|wmv|3gp|avi|mpg|mpeg|mp4|flv|mp3|mid|wml|swf|pdf|doc|docx|ppt|pptx|zip)$
    {
        expires 365d;
        access_log off;
    }

Block location quy định những loại file nào được lưu cache trên trình duyệt và thời gian lưu cache là **365d** (ngày) nếu thường xuyên update chỉnh sửa những file này bạn giảm thời gian expires xuống cho phù hợp. Bạn có thể chia ra, file hình ảnh video thời gian expires là 1M (1 tháng) với các file **js, css** ít thay đổi hơn thì để 1y (1 năm).

Directive **access_log off** quy định sẽ không ghi log truy cập đến các nội dung này giúp CPU đỡ bận rộn hơn.

Khi cấu hình cache cho trình duyệt, bạn chỉ thật sự cảm thấy hiệu quả khi lần thứ 2 truy cập trang web vì khi đó các file hình ảnh, scripts đã được lưu trên webbrowser của bạn tốc độ tải web tăng lên đáng kể.

Bạn tham khảo thêm một số giá trị thời gian khác để sử dụng cho linh hoạt:

* ms: milliseconds
* s: seconds
* m: minutes
* h: hours
* d: days
* w: weeks
* M: months (30 days)
* y: years (365 days)

<a name = "5"></a>
## 5. BẬT NÉN GZIP COMPRESSION

Mục đích enable gzip cho website để giảm dung lượng dữ liệu trước khi truyền nội dung trên internet nhằm tiết kiệm băng thông (bandwidth), cũng là cách tăng tốc web hiệu quả.

Thêm đoạn code sau vào block server{…}

```scala
gzip on;
gzip_static on;
gzip_buffers 4 8k;
gzip_min_length 1100;
gzip_vary on;
gzip_disable "MSIE [1-6]\.";
gzip_types text/plain text/css application/javascript text/xml application/xml+rss;
```

Toàn bộ chỉ thị trên đều có trong module ```ngx_http_gzip_module``` được mặc định trên nginx.

Chỉ có chỉ thị **gzip_static** là trong module ```ngx_http_gzip_static_module``` dùng để gửi file ```.gz``` đến client. Tuy nhiên không phải là module được biên dịch sẵn phải kiểm tra lại trước khi dùng chỉ thị này bằng lệnh ```nginx -V```. Mình dùng nginx bản 1.9, 1.10, 1.11 thì thấy module này được biên dịch sẵn rồi, bạn nào bị lỗi bật nén gzip nginx thì chú ý chỗ này.

* **gzip** : giá trị on đơn giản chỉ là enable tính năng gzip trên server.
* **gzip_static** : giá trị on chỗ này có nghĩa sẽ kiểm tra file .gz có tồn tài trên server hay không.
* **gzip_buffers** : thiết lập kích thước bộ nhớ đệm trên memory page tương đương 4 x 8k = 32k, theo một số lời khuyên 4 8k là phù hợp.
* **gzip_min_length**: chỉ áp dụng gzip với các file có dung lượng từ 1100 byte trở lên.
* **gzip_vary**: giá trị on để chèn thêm Vary: Accept-Encoding vào response header khi gzip, gzip_static đang được kích hoạt.
* **gzip_disable** : chỉ thị này để vô hiệu hóa gzip trên trình duyệt IE6 “MSIE [1-6]\.” dựa vào User-Agent. Trình duyệt IE6 quá cũ rồi gzip không hoạt động được, bạn có thể thêm dòng gzip_disable “Mozilla / 4”; với trình duyệt cũ của Mozilla.
* **gzip_types** : mặc định chỉ có text/plain được nén bạn phải bổ sung thêm các tập tin văn bản xml, css, js. Mình khuyên bạn dùng với những định dạng bên trên thôi kẻo lại có tác dụng ngược.

Trên đây là những chia sẻ về Caching trên NGINX và vài cách cấu hình cache cơ bản nhằm tăng tốc độ website một cách nhanh chóng mà hưu hiệu.

<a name = "6"></a>
## 6. THUẬT TOÁN CACHE

### 1. LRU

LRU (least recently used) cache (đọc là /kaʃ/) là một trong các thuật toán cache phổ biến. Cache được dùng để lưu trữ các kết quả tính toán vào một nơi và khi cần tính lại thì lấy trực tiếp kết quả đã lưu ra thay vì thực hiện tính. Cache thường có kích thước nhất định và khi đầy, cần bỏ đi một số kết quả đã tồn tại trong cache. Việc kết quả nào sẽ bị bỏ đi phân loại các thuật toán cache này thành:

* LRU (Least Recently Used): bỏ đi các item trong cache ít được dùng gần đây nhất.
* MRU (Most Recently Used): bỏ đi các item trong cache được dùng gần đây nhất.
* ...

Việc sử dụng kỹ thuật memoization để tối ưu các quá trình tính toán như vậy là chuyện thường ở huyện, vậy nên từ Python 3.2, trong standard library functools đã có sẵn function lru_cache giúp thực hiện công việc này ở dạng decorator. Khi gọi function với một bộ argument, lru_cache sẽ lưu các argument lại thành key của dict, và sử dụng kết quả gọi function làm value tương ứng. lru_cache có option để chỉnh kích thước của cache, phân biệt kiểu của argument.

```scala
functools.lru_cache?
Signature: functools.lru_cache(maxsize=128, typed=False)
Docstring:
Least-recently-used cache decorator.
```

Một điểm đáng chú ý là các argument được gọi với function đều phải là immutable object bởi chúng được dùng làm key của dict. Khi dùng decorator lru_cache, ta chỉ cần tập trung vào viết function cần viết, lru_cache sẽ lo việc thực hiện caching/memoization.

```scala
In [8]: @lru_cache(maxsize=None)
def fib(n):
    if n <= 2: return 1
    else:
        return fib(n-1) + fib(n-2)
   ...:

In [9]: %time fib(75)
CPU times: user 84 µs, sys: 26 µs, total: 110 µs
Wall time: 114 µs
Out[9]: 2111485077978050
```

LFU yêu cầu một bộ đếm tham chiếu được duy trì cho mỗi khối trong cache. Khi một thay thế là cần thiết, LFU chọn khối có bộ đếm tham chiếu thấp nhất. Động cơ thúc đẩy cho LFU và các thuật toán tần số cơ bản khác là bộ tham chiếu có thể được sử dụng như một sự đánh giá xác suất của một khối đang được tham chiếu. Khối có số lượng cao có nhiều khả năng được tham chiếu hơn so với các khối có số lượng thấp. Trong bộ nhớ cache mức bình thường (chẳng hạn như một trạm làm việc đa nhiệm với đĩa nội bộ) LFU làm việc kém. Một vấn đề với việc sử dụng các bộ đếm tần số là các khối chắc chắn có thể xây dựng các bộ tham chiếu cao và không bao giờ bị thay thế. Mặc dù điều này có thể thích hợp trong một số trường hợp, khả năng tham chiếu xác suất cao như một khối không giữ trong một thời gian dài mà đúng hơn là chỉ trong một thời gian tương đối ngắn. Một giải pháp cho vấn đề này là để giới thiệu một số hình thức đếm tham chiếu “sự hóa già” (hoặc sự phân rã). Bộ đếm tham chiếu trung bình (trên tất cả các khối hiện tại trong cache) được duy trì động. Bất cứ khi nào bộ đếm trung bình này vượt quá giá trị tối đa được xác định trước, Amax (một tham số thuật toán), tất cả các số tham chiếu C, được giảm tới [C / 2]. Một vấn đề khác với LFU là nó chịu ảnh hưởng của thời gian cục bộ. Mặc dù một khối có thể tương đối không thường xuyên tham chiếu tổng thể, nó có thể chịu các truyền ngắn của các tham chiếu lại, cái mà xây dựng tổng số tham chiếu của nó với một giá trị sai lạc cao. Vấn đề này có lỗi chính của thuật toán LFU cho các thuật toán thay thế bộ nhớ cache trong môi trường truyền thống. Trong môi trường phân tán, tuy nhiên, vấn đề này có thể được loại bỏ tự động. Có thể thời gian cục bộ được "lọc" ra khỏi bộ nhớ cache tại các trạm client.

Các thí nghiệm sơ bộ cho thấy rằng LFU cũng chịu từ thực tế rằng nhiều yêu cầu viết được đưa ra bởi các ứng dụng chỉ ghi dữ liệu một phần của một khối đầy đủ (ghi một phần của khối). Khi một yêu cầu viết như vậy được đưa ra, đầu tiên, bộ nhớ cache của khách hàng là tìm kiếm các khối yêu cầu. Nếu nó được tìm thấy (một "client hit"), sau đó các nội dung được sửa đổi bằng cách kết hợp đơn giản là một phần ghi mới với những nội dung cũ. Khối mới này được ghi tới máy chủ (giả định là write-through). Nếu khối được yêu cầu không được tìm thấy ở bộ nhớ cache client (một "client miss"), tuy nhiên, nó trước tiên phải là "đọc" từ máy chủ vì một phần viết có thể được kết hợp với các nội dung cũ. Một khi điều này được thực hiện, khối đó phải được ghi trở lại máy chủ. Vì vậy, một yêu cầu viết một phần mà "bỏ qua" tại client thực sự tạo ra một yêu cầu đọc ngay lập tức theo sau là một yêu cầu viết vào bộ nhớ cache fileserver. Lần lượt, kết thúc trong bộ đếm tham chiếu của khối tại máy chủ được tăng lên hai lần cho một hoạt động viết đơn. Đếm đôi này gây ra việc đếm tham chiếu sai lệch cao đối với một số khối trong bộ nhớ cache máy chủ và ảnh hưởng đến việc thực hiện chính sách LFU. Có một giải pháp đơn giản cho vấn đề này: chỉ đơn giản là không tăng số lượng tham chiếu ghi. Các mô phỏng cho thấy rằng thay đổi này giúp cải thiện đáng kể hiệu suất của thuật toán LFU tại fileserver.

* Đối với một write-miss viết:
  * Chèn khối vào bộ nhớ cache
  * Thiết lập số tham chiếu của nó tới một
  * Thiết lập một cờ cho khối này là TRUE
* Đối với một write-hit viết:
  * Không thay đổi tính tham chiếu hoặc FLAG cho khối này
* Đối với một read-miss:
  * Chèn khối vào bộ nhớ cache
  * Thiết lập số tham chiếu của khối vào một
  * Thiết lập một cờ cho khối này để FALSE
* Đối với một read-hit:
  * Nếu FLAG cho khối này là FALSE, sau đó tăng tính tham chiếu của khối này, ELSE thiết lập số tham chiếu của nó tới một và FLAG của nó đến FALSE.

### 2. FBR

Một thuật toán mới gọi là thay thế dựa vào tần số (Frequency Based Replacement - FBR). FBR là một thuật toán thay thế lai, cố gắng để nắm bắt những lợi ích của cả hai LRU và LFU mà không có những hạn chế liên quan. FBR duy trì LRU thứ tự của tất cả các khối trong bộ nhớ cache, nhưng quyết định thay thế chủ yếu dựa trên số lượng tần số. Để thực hiện điều này, FBR phân chia bộ nhớ cache thành ba phân vùng: một phân vùng mới, phân vùng giữa, và một phân vùng cũ. Các kích thước của các phân vùng được quy định bởi hai tham số cho mô hình. Fnew cho biết tỷ lệ phần trăm của tổng số của các khối bộ nhớ cache được chứa trong phần mới ( the most recently used - MRU), trong khi Fold cho thấy tỷ lệ phần trăm của các khối chứa trong các phần cũ (the least recently used - LRU). Phần trung gian bao gồm những khối hoặc mới hoặc phần cũ. Khi một tham chiếu xảy ra với một khối trong phần mới, số tham chiếu của nó tăng lên. Điều này được thực hiện một cách rõ ràng "yếu tố ra khỏi" vị trí tạm thời là lý do chính cho sự thất bại trong quá khứ của các thuật toán dựa trên tần số. Tham chiếu cho các phần trung gian và cũ gây ra số lượng tham chiếu tăng lên. Khi một khối phải được lựa chọn để thay thế, FBR chọn các khối với số lượng tham chiếu thấp nhất, nhưng chỉ giữ các khối này là ở trong phần cũ của thứ tự ngăn xếp LRU (mối quan hệ được giải quyết bằng cách chọn gần đây được sử dụng ít nhất của những khối này). Điều này được thực hiện bởi vì các khối trong các phần mới và phần trung gian không có đủ thời gian để xây các bộ đếm tham chiếu của chúng. Thuật toán FBR sử dụng các chương trình đếm tuổi tham chiếu được mô tả cho các thuật toán LFU. Các thí nghiệm sơ bộ cho thấy rằng khi FBR được sử dụng bộ nhớ cache fileserver nó lợi ích từ việc sửa đổi cùng một chương trình đếm tham chiếu như trước đây được mô tả cho các thuật toán LFU. Cần lưu ý rằng bằng cách chọn các thông số thích hợp (Fnew = 0 và Fold = 100) các thuật toán FBR trở thành chính sách LFU.

### 3. LFU

LRU thay thế các khối trong bộ nhớ cache không được sử dụng trong thời gian giới hạn dài nhất. Những tiền đề cơ bản đằng sau thuật toán LRU, và lý do cho sự thành công của nó, là các khối đã được tham chiếu trong thời gian qua có thể sẽ được tham chiếu một lần nữa trong tương lai gần. Đặc tính này là tạm thời, được biết đến là một đặc tính của mô hình tham chiếu bộ nhớ của bộ vi xử lý, mô hình tham chiếu trang trong các hệ thống bộ nhớ ảo, và đến một mức độ nhất định, khối đĩa truy cập các mẫu đang chạy các ứng dụng trên máy trạm. Các câu hỏi cơ bản là nghiên cứu này cố gắng giải quyết : “thời gian cục bộ vẫn tồn tại dòng tham chiếu khối đĩa được nhìn thấy bằng một máy chủ tới mức độ LRU có thể tận dụng được lợi thế của nó, hoặc là nó được phá vỡ bằng sự hiện diện của cache client để LRU không còn là yếu tố dự báo trước trong đó các khối sẽ (và sẽ không) được tham chiếu lại lần nữa”.

### 4. Thuật toán MIN

MIN là thuật toán thay thế tối ưu về mặt lý thuyết. Nghĩa là, nó cung cấp tỷ lệ bỏ lỡ thấp nhất của tất cả các thuật toán có thể. Thuật toán là cách đơn giản, “Thay thế các khối không được sử dụng trong khoảng thời gian dài nhất”. Thật không may, thuật toán thay thế tối ưu là không thể thực hiện vì nó đòi hỏi sự hiểu biết trong tương lai của chuỗi tham chiếu đĩa. Việc thực hiện thuật toán này là hữu ích cho nghiên cứu, tuy nhiên, vì nó cung cấp một ràng buộc như trên về hiệu suất có thể đạt được, và do đó cung cấp một số "cái tốt" của các chính sách khác.

### 5. Thuật toán RAND

RAND chọn trong số tất cả các khối trong bộ nhớ cache với xác suất bằng nhau. Bằng trực giác, RAND có vẻ hấp dẫn trong bối cảnh này nếu lưu trữ của cache client được lọc ra khỏi tất cả các đặc tính vùng từ các dòng tham chiếu của chúng. Để RAND làm việc tốt, nó yêu cầu tất cả các khối đĩa được truy cập với xác suất bằng nhau. Mặc dù vậy, RAND là một thuật toán hữu ích để nghiên cứu vì nó cung cấp một cơ sở dựa vào đó để đánh giá các chính sách cố gắng để quyết định thay thế làm cho “thông minh”. Nếu chúng không thể thực hiện tốt hơn so với RAND, thì gần như chắc chắn một số lỗ hổng cơ bản trong chiến lược quyết định. Trong thực tế RAND cung cấp một loại thấp hơn ràng buộc về hiệu suất.

<a name = "6"></a>
## Tài liệu tham khảo

* [Cài đặt NGINX làm web server](https://www.thuysys.com/server-vps/web-server/bo-tai-lieu-cai-lemp-webserver-tren-centos.html)
* [Cách nén javascript và css trên linux](https://www.thuysys.com/toi-uu/minify-compress-javascript-va-css-website-khong-dung-plugin.html)
* [Cài đặt php7](https://www.thuysys.com/server-vps/web-server/cai-dat-php-7-php-fpm-tren-nginx-centos-7.html)
* [Tìm hiểu Caching và cách tăng tốc website trên NGINX](https://www.thuysys.com/toi-uu/tim-hieu-caching-va-cach-tang-toc-website-tren-nginx.html#5_Bat_nen_Gzip_Compression)
* [Memoization là gì? LRU cache là gì?](https://pymi.vn/blog/memoization/)