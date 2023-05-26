Port scan:

![enter image description here](https://imgur.com/UPCUef0.png)

![enter image description here](https://imgur.com/lhXKuQN.png)

80/TCP - HTTP: Apache2 Ubuntu Default Page.

URL brute-force:

![enter image description here](https://imgur.com/FezOyKL.png)

/test:

![enter image description here](https://imgur.com/gvYWrrE.png)

/wordpress:

![enter image description here](https://imgur.com/c3p3Lut.png)

Không có gì đặc biệt, cần enum thêm.

445/TCP - SMB:

![](https://imgur.com/GKu1s9P.png)

![enter image description here](https://imgur.com/K3ly28V.png)

Có được Subrion (CMS) creds và hint "Fix subrion site, /subrion doesn't work, edit from panel".
Crack password có được creds admin:Scam2021

![enter image description here](https://imgur.com/qriCmCm.png)

Truy cập vào 10.10.78.158/subrion/panel và đăng nhập với creds ở trên.
Đi đến mục Content -> Uploads và upload payload lên (ở đây là php-reverse-shell.phar).

![enter image description here](https://imgur.com/fQHwFX7.png)


Mở listener ở local và truy cập vào 10.10.78.158/subrion/uploads/php-reverse-shell.phar để có reverse shell session:

![enter image description here](https://imgur.com/gIueuAp.png)

Đọc file passwd thấy có user scamsite do người dùng tự thêm (thông tin này sẽ dùng sau).

![enter image description here](https://imgur.com/w4xKtJB.png)

Đưa linpeas.sh từ local lên target bằng python3 http.server module và wget. Enum bằng linpeas, chú ý vào đoạn thông tin đăng nhập trích xuất được:

![enter image description here](https://imgur.com/Nt5d2dR.png)

Đây là creds của Database, Wordpress. Sau khi không khai thác được gì thêm từ Database và Wordpress site, thử đăng nhập vào user scamsite ở target:

![enter image description here](https://imgur.com/KTHxCAQ.png)

Thành công -> Không nên dùng chung mật khẩu cho nhiều nền tài khoản riêng biệt.
Từ đây có thể dễ dàng leo thang đặc quyền root bằng việc lợi dụng sudo right ở user scamsite và lấy root flag ở /root/root.txt:

![enter image description here](https://imgur.com/u5NIEUz.png)

Happy Hacking!
