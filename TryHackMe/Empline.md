Port scan:

![enter image description here](https://imgur.com/3Hp0cRi.png)

![enter image description here](https://imgur.com/G12HT1o.png)

URL Brute-force:

![enter image description here](https://imgur.com/lpQqoxw.png)

Không có gì thú vi.

80/TCP - HTTP:

![enter image description here](https://imgur.com/65iiZb9.png)

Click vào "Employment" ở menu sẽ được navigate đến job.empline.thm/career. Truy cập vào job/empline.thm.

![enter image description here](https://imgur.com/8golcUt.png)

Web app sử dụng opencats 0.9.4 có lỗ hổng RCE. Lấy luôn module 50585 trên exploit-db để khai thác.

![enter image description here](https://imgur.com/s46qGEo.png)

Vì ở đây không phải là 1 shell kết nối back về local attacker, mình đưa payload php reverse shell lên target và kết nối ngược về local (ở local mở sẵn listener) và sau đó nâng lên Fully Interactive TTYs luôn để tiện thao tác.

![enter image description here](https://imgur.com/r2OSrN6.png) 

![enter image description here](https://imgur.com/oOpzyYt.png)

Tuy đã vào được hệ thống nhưng cần có quyền george để lấy được user flag.
Enum bằng linpeas và trích xuất được 1 phần thông tin có ích từ file config (database creds):

![enter image description here](https://imgur.com/7xVSHby.png)

Đăng nhâp vào database với creds james:ng6pUFvsGNtw :

![enter image description here](https://imgur.com/AcFXx2e.png)


Đi vào khai thác DB để tìm thêm thông tin:

![enter image description here](https://imgur.com/OR5Pj7b.png)

Có bảng user -> trích xuất thông tin user từ đây:

![enter image description here](https://imgur.com/Gf012Di.png)

Lấy được password hash của george -> crack:

![enter image description here](https://imgur.com/ESgWny7.png)

Dùng nó để SSH vào target với user là george:

![enter image description here](https://imgur.com/n7CUiKx.png)

Trong quá trình enum bằng linpeas trước đó, tìm được /usr/local/bin/ruby được set capabilities (dòng màu vàng):

![enter image description here](https://imgur.com/S96XtNi.png)

Lợi dụng điều đó để leo quyền root bằng cách thay đổi owner file /etc/passwd từ root thành george, sau đó thêm vào 1 user tùy ý với đặc quyền root (uid = 0, gid = 0).
Đầu tiên, chạy dòng lệnh: 

```
ruby -e 'require "fileutils"; FileUtils.chown(1002, 1002, "/etc/passwd")'
```
Dùng openssl để tạo password hash (algorithm SHA-512) từ mật khẩu 123456 :

![enter image description here](https://imgur.com/M6QL1qy.png)

Thêm dòng dưới đây vào cuối nội dung file /etc/passwd (lúc này đã được sở hữu bởi george):
```
hehe:$6$dUsfbOY94jaN8Rps$hFbiHAx/DeSKVDcup0Ww38qtzdVATorEdpzdykpmwj1Lmeat4dFlr2mjkqaDWRKIcn93bqqoknERkHOLLSh7h/:0:0::/root:/bin/bash
```

Thao tác trên là để thêm user hehe, password là 123456 vào hệ thống. User hehe có đặc quyền root.
Switch sang user hehe, nhập password 123456 và lấy root flag ở /root/root.txt:

![enter image description here](https://imgur.com/BhXCXNv.png)

Happy Hacking!
