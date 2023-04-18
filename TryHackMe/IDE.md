Port scan:

![enter image description here](https://imgur.com/PDNkyX0.png)

![enter image description here](https://imgur.com/db7pGn0.png)

Web enum:

Port 80:

![enter image description here](https://imgur.com/zlvuvGf.png)

![enter image description here](https://imgur.com/rHBEKtB.png)

Có vẻ không tìm kiếm được gì.

Port 62337:

![enter image description here](https://imgur.com/MuLuMJc.png)

![enter image description here](https://imgur.com/EDHyvdQ.png)

-> Port 62337 sử dụng Codiad 2.8.4. Phiên bản Codiad 2.8.4 có CVE có thể khai thác đến RCE nhưng cần authenticated -> Cần tìm được credentials.

FTP Enum:

![enter image description here](https://imgur.com/RSmXobf.png)

Tìm được 1 file có tên "-". Nội dung file:

![enter image description here](https://imgur.com/7pQoSlb.png)

Từ đây ta đoán được 2 user có thể tồn tại là "john" và "drac". Ngoài ra, nội dung đề cập đến password được reset về mặc định. Quay lại http://10.10.56.176:62337/ và đăng nhập. Sau một vài lần thử, đăng nhập thành công với john:password.

Exploit theo CVE-2018-14009:

![enter image description here](https://imgur.com/ZKOZ75B.png)

![enter image description here](https://imgur.com/kH170Da.png)

![enter image description here](https://imgur.com/SrwhAim.png)

Đã vào được hệ thống. Cần privesc sang user drac để lấy user flag:

![enter image description here](https://imgur.com/M0Q4Byj.png)

File .bash-history có thể đọc được:

![enter image description here](https://imgur.com/8LSKLV5.png)

Switch sang user drac thành công -> có được password của drac.

![enter image description here](https://imgur.com/pUu7mXZ.png)

Để tiện cho các tác vụ tiếp theo, thoát khỏi shell session hiện tại và ssh vào server với user drac.
Lấy user flag ở /home/drac/user.txt:

![enter image description here](https://imgur.com/qmTVz4b.png)

Cần privesc lên root.
Check sudo right:

![enter image description here](https://imgur.com/6bMcFbf.png)

User drac có thể vận hành service vsftpd với quyền root. Dựa vào đây có thể privesc lên root.

Đầu tiên, tìm service config file cho vsftpd:

![enter image description here](https://imgur.com/nzHWFXL.png) 

Ta cần quan tâm đến 2 file là "/lib/systemd/system/vsftpd.service" và "/etc/systemd/system/multi-user.target.wants/vsftpd.service". Kiểm tra quyền:

![enter image description here](https://imgur.com/ArBYuN9.png)

Vậy là ta chỉ cần quan tâm đến file "/lib/systemd/system/vsftpd.service". User drac có thể sửa nội dung file này. Nội dung gốc:

![enter image description here](https://imgur.com/uA0ES47.png)

Chèn payload cho reverse shell vào giá trị của “ExecStartPre”. Ta có thể nhận được root shell ở local khi vsftpd server restart. Sửa nội dung lại thành như sau:

![enter image description here](https://imgur.com/U1Y4Fa0.png)

Sau đó, reload daemon và restart service vsftpd với quyền root:

![enter image description here](https://imgur.com/J8VHKAU.png)

Ở local nhận được root shell và lấy root flag:

![enter image description here](https://imgur.com/ClDu967.png)


Happy Hacking!
