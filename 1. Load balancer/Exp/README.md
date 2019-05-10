---
 title: "Thí nghiệm"
---

# Mô tả thí nghiệm

>1. Làm một web tĩnh đơn giản (hello world), viết bằng python
>2. Cho web chạy trên 2 port khác nhau
>3. Cài đặt nginx để load balancing giữa 2 port trên (verify lại bằng web browser)

# Thực hiện thí nghiệm

Cần có:

* 2 file web bằng python (đơn giản) cấu hình host là localhost, port tùy ý (khác nhau).
* Cài đặt nginx, vào etc (ngoài home) -> nginx -> conf.d tạo file load-balancer.conf cấu hình như sau:

```scala
upstream backend
{
   server localhost:port1;
   server localhost:port2;
}

# This server accepts all traffic to port 80 and passes it to the upstream.
# Notice that the upstream name and the proxy_pass need to match.

server {
   listen 80;

   location / {
      proxy_pass http://backend;
   }
}
```

* Lưu ý: port1 và port2 lần lượt là địa chỉ của 2 file python.

Chạy:

* Mở teminator tại file chứa 2 file python. Chạy 2 file song song bằng lệnh:

> python tên_file.py

* Vào địa chỉ localhost. Lúc này nginx tự phân chia truy cập tới 1 trong 2 file python, luân phiên thay đổi mỗi lần tải lại.