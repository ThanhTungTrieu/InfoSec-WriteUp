Port scan:

![enter image description here](https://imgur.com/yLNuuQ9.png)

![enter image description here](https://imgur.com/9bjr1Q8.png)

80/TCP - HTTP:

![enter image description here](https://imgur.com/2WWhYY0.png)

URL Brute-force:

![enter image description here](https://imgur.com/5OTZYku.png)

Truy cập vào /gallary -> redirect đến /gallery/login.php

![enter image description here](https://imgur.com/e4VWa77.png)

Đăng nhập thử, bắt request bằng Burp suite:

![enter image description here](https://imgur.com/JJDZfAW.png)

Ta thấy response trả về cả câu SQL được thực thi -> Easy SQLi.
Truyền vào username là `admin` và password là `1') OR ('1'='1`

Trong Burp suite cần URL encoding cho ký tự space, còn đăng nhập trực tiếp ở login form thì không cần.

![enter image description here](https://imgur.com/chsEwyQ.png)

Đăng nhập thành công vào tài khoản admin.

![enter image description here](https://imgur.com/GrZ9PhY.png)

Trong phần cài đặt tài khoản (My Account), có phần upload avatar. Tại đây không có validate file được upload, dễ dàng tải lên reverse shell.
Có được reverse shell ở local:

![enter image description here](https://imgur.com/2AD3YC6.png)

Enum thêm:

![enter image description here](https://imgur.com/7RkNqkJ.png)

Tải lên linpeash từ local bằng wget, chạy linpeas ở target và enum được password của user nào đó. Trước tiên cần nâng cấp shell lên fully interactive tty. Các bước thực hiện:

1. `python3 -c 'import pty; pty.spawn("/bin/bash")'`
2. Nhấn Ctrl + Z tại shell session hiện tại.
3. `stty raw -echo;  fg;  ls;  export  SHELL=/bin/bash;  export  TERM=screen; stty rows 38 columns 116; reset;`

Sau khi có được interactive tty, thử switch sang user mike bằng password đó và thành công.
User flag:

![enter image description here](https://imgur.com/68rGsnD.png)

Mike's sudo right:

![enter image description here](https://imgur.com/iJ53LJ3.png)
File /opt/rootkit.sh

![enter image description here](https://imgur.com/boWDNOJ.png)

Khi thực thi file /opt/rootkit.sh, chương trình thực hiện đọc input từ bàn phím  vào biến ans, sau đó đến switch case dựa trên giá trị biến ans vừa nhập vào. Ý tưởng để privesc ở đây là nhập vào `read` để chương trình nhảy xuống case `read`, thực thi `/bin/nano /root/report.txt` bằng quyền root, ta sẽ lợi dụng nano để leo quyền. 
Nhưng vì terminal hiện tại nếu mở nano sẽ bị báo lỗi `Error opening terminal: unknown`. Để giải quyết vấn đề này, chạy `export TERM="xterm"`.

Chạy `sudo /bin/bash /opt/rootkit.sh`, nhập read và enter.

![enter image description here](https://imgur.com/oC8Ch8q.png) 

Sau khi vào được màn hình nano, nhấn lần lượt `Ctrl R`, `Ctrl X`, và nhập vào `reset; sh 1>&0 2>&0`, enter. 
Đến đây đã thành công có được quyền root, lưu ý là vì thực hiện lệnh trong màn hình của nano nên khá khó nhìn.

![enter image description here](https://imgur.com/IgA7mKZ.png)

Lấy root flag:

![enter image description here](https://imgur.com/Xyz7esr.png)

Box này còn yêu cầu tìm password hash của user admin (web). File /root/mysql_history có chứa thông tin truy cập mysql. Dựa vào đó để truy cập database là lấy ra password hash cần tìm.
Nội dung file `/root/.mysql_history`:

![enter image description here](https://imgur.com/2B2hmNT.png)

Truy cập mysql bằng `gallery_user:passw0rd321`

![enter image description here](https://imgur.com/iFP8oSJ.png)

![enter image description here](https://imgur.com/y92M4Tj.png)

Truy vấn bằng MySQL như hình, lấy được password hash cần tìm.

Happy Hacking!
