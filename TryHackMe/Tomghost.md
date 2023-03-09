Thử thách này yêu cầu exploit machine, lấy user flag và root flag - tương ứng với gain access và privilege escalation.

Port scan:

![enter image description here](https://imgur.com/rAZzDAI.png)

Service ajp có lỗ hổng File Read/Inclusion. Exploit bằng Metasploit đọc file /WEB-INF/web.xml:
![enter image description here](https://imgur.com/ymQKn2D.png)
Lấy được credential `skyfuck:8730281lkjlkjdqlksalks`.
SSH vào machine bằng credential lấy được. Flag 1 ở /home/merlin/user.txt.

Để lấy được flag2, ta cần leo thang đặc quyền lên root. User skyfuck gần như không có cách nào để priviesc lên root. Mục tiêu ở đây có thể là chiếm quyền access vào user merlin. Trong /home/skyfuck có 2 tệp là `credential.pgp` và `tryhackme.asc`. Tệp `credential.pgp` là tệp được mã hóa bằng pgp và được bảo vệ bằng passphrase, tệp `tryhackme.asc` là tệp pgp private key.

Copy nội dung file `tryhackme.asc` về local và dùng John the Ripper bẻ khóa file tryhackme.asc để lấy passphrase:
![enter image description here](https://imgur.com/60e5odj.png)
Quay lại target machine, dùng gpg để decrypt file `credential.pgp`:
![enter image description here](https://imgur.com/h6KUSJq.png)
Có được credential của user merlin. SSH vào target với user merlin.
Khai thác merlin sudo right với /usr/bin/zip để leo root
![enter image description here](https://imgur.com/9yf1vkP.png)
Root flag ở /root/root.txt

Happy Hacking!
