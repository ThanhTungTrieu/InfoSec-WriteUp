Thử thách này yêu cầu exploit machine, lấy user flag và root flag - tương ứng với gain access và privilege escalation.

Port scan:

![enter image description here](https://imgur.com/bkNf89T.png)

Brute-force url bằng gobuster:

![enter image description here](https://imgur.com/KtwgagV.png)

Tiếp tục brute-force vào /content

![enter image description here](https://imgur.com/CTZLea3.png)

Truy cập vào /content

![enter image description here](https://imgur.com/3dQjbu1.png)

Từ response, ta thấy được trang web sử dụng CMS có tên SweetRice. Truy cập /content/changelog.txt sẽ thấy phiên bản của SweetRice đang dùng là v1.5.1. Phiên bản này có lỗ hổng Arbitrary File Upload (EDB-ID: 40716). Nhưng để khai thác lỗ hỏng Arbitrary File Upload, cần có thông tin đăng nhập của CMS SweetRice. Quay lại /content/inc/mysql_backup/ 

![enter image description here](https://imgur.com/XpARivS.png)

Download file mysql_backup và hy vọng có thông tin nào đó hữu ích.

![enter image description here](https://imgur.com/K4CZAFS.png)

Rất may trong database có lưu thông tin đăng nhập CMS (dòng đang được chọn). Password được hash dạng MD5. Dùng online tool để crack:

![enter image description here](https://imgur.com/qUVY9bp.png)

Từ thông tin trên có được thông tin đăng nhập CMS là: `manager:Password123`. Đến lúc để exploit lỗ hổng Arbitrary File Upload. Upload reverse shell payload. Ở lần đầu tiên upload payload dạng `.php`, dù đã upload thành công từ local nhưng khi kiểm tra trên target lại không có. Điều đó nghĩa là file có đuôi `.php` đã bị block. Upload payload đuôi `.phtml` để bypass:

![enter image description here](https://imgur.com/pvIc3nu.png)

Mở netcat listener ở local và truy cập vào /content/attachment/php-reverse-shell.phtml (tên file vừa upload) để có reverse shell. User flag ở /home/itguy/user.txt. 

=> Access gained.

Để lấy được root.txt, ta cần có quyền root. Kiểm tra sudo right bằng `sudo -l`:

![enter image description here](https://imgur.com/WgETrFP.png)

User hiện tại (www-data) có thể chạy `/usr/bin/perl` và `/home/itguy/backup.pl` quyền sudo không cần password.

![enter image description here](https://imgur.com/ZcT0Vgm.png)

User hiện tại không có quyền ghi vào file backup.pl nhưng vẫn có thể đọc được. File backup.pl là một file thực thi bằng mã nguồn perl, nội dung của nó là thực thi file shell `/etc/copy.sh`.

![enter image description here](https://imgur.com/dooJq8w.png)

File /etc/copy.sh cho phép các user khác được quyền read, write, execute. Vậy ta cần sửa nội dung file này thành payload có thể priviesc.
Mở sẵn netcat listener ở local và thực thi `sudo /usr/bin/perl /home/itguy/backup.pl` ở target để spawn 1 reverse root shell.

![enter image description here](https://imgur.com/Rkan5aq.png)
![enter image description here](https://imgur.com/1b4KKai.png)

=> Privesc.

Lấy root flag ở /root/root.txt.

Happy Hacking!
