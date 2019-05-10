---
    title: "Chạy cluster"
---

---
Notes:

* Để chạy cluster, cần thực hiện các bước sau:
  * Mở teminator tại file chứa thư mục bin và 6 file cluster, chạy lệnh:
    * ps -u your_PC_id -o pid,rss,command | grep redis.

```scala
20100  5744 ./bin/redis-server *:7000 [cluster]
20141  5816 ./bin/redis-server *:7001 [cluster]
20178  5724 ./bin/redis-server *:7002 [cluster]
20384  5596 ./bin/redis-server *:7003 [cluster]
20420  5580 ./bin/redis-server *:7004 [cluster]
20471  5632 ./bin/redis-server *:7005 [cluster]
26632  1056 grep --color=auto --exclude-dir=.bzr --exclude-dir=CVS --exclude-dir=.git --exclude-dir=.hg --exclude-dir=.svn redis
```

* 
  * Nếu kết quả chạy ra như đoạn trên, tức là server có chứa các cluster đang chạy, thực hiện kill bằng lệnh:
    * kill -9 'id'

Trong đó id là chuỗi 5 số ở đầu mỗi dòng báo cluster đang chạy. Sau khi chạy xong, chạy lệnh sau là có thể chạy cluster:

```css
cd /home/path/to/your/redis
./bin/redis-server ./7000/7000.conf
./bin/redis-server ./7001/7001.conf
./bin/redis-server ./7002/7002.conf
./bin/redis-server ./7003/7003.conf
./bin/redis-server ./7004/7004.conf
./bin/redis-server ./7005/7005.conf
```

---