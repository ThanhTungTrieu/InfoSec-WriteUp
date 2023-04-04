Port scan:

![enter image description here](https://imgur.com/l2BuvWq.png)

![enter image description here](https://imgur.com/Yh5kds1.png)

Giao diện webapp:

![enter image description here](https://imgur.com/KOsxnPa.png)

Có vẻ không có gì để khai thác.
Dir brute-force:

![enter image description here](https://imgur.com/TWMtA7Q.png)

robots.txt:

![enter image description here](https://imgur.com/eKW1rQd.png)

Liên quan đến rockyou thì hầu hết là các phương pháp brute-force. Ban đầu, mình đã brute-force user-agent trong request và quan sát content-length trong response nhưng không có gì đáng chú ý.
Để ý ở đầu trang chủ webapp có dòng này:

![enter image description here](https://imgur.com/GQ4QaiN.png)

Và ở khu vực comment:

![enter image description here](https://imgur.com/AJNjv4k.png)

Từ đó, mình suy đoán "meliodas" là 1 user trong hệ thống. Đây cũng là hướng đi khả quan nhất hiện có. SSH brute-force với user meliodas:

![enter image description here](https://imgur.com/GGCXeYu.png)


Đã tìm được password của meliodas. SSH vào server và lấy user flag ở /home/meliodas/user.txt.

![enter image description here](https://imgur.com/NLd4UiU.png)

Đến lúc privesc lên root để lấy root flag.
Check sudo right:

![enter image description here](https://imgur.com/iibMBSc.png)

Nice! Có thể chạy sudo với file bak.py -> Lợi dụng nó để privesc. File bak.py:

![enter image description here](https://imgur.com/ULpPEZY.png)

File bak.py import 1 file python có tên là zipfile. Tạo file zipfile.py ở cùng thư mục với nội dung:

![enter image description here](https://imgur.com/UuIW0ir.png)

Chạy `sudo /usr/bin/python3 /home/meliodas/bak.py` và nó sẽ spawn 1 root shell. Lấy root flag ở /root/root.txt:

![enter image description here](https://imgur.com/aWPQj40.png)

Happy Hacking!
