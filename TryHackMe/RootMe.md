﻿- Recon: 
	* Nmap: 2 opening port:
		* 22 - SSH
		* 80 HTTP: Ubuntu Server 2.4.29; PHP.
	* Gobuster:
		* /uploads/ -> nơi chứa file uploads
		* /panel -> trang để uploads file
- Gain Access: Upload payload reverse.php -> blocked. Xem page source xác định không có client-side filter, bị blocked bởi blacklist ở server-side. Thử upload reverse.phtml -> success. Truy cập vào /uploads/ và thấy file vừa uploads -> xác nhận đã upload payload thành công. Mở netcat listener ở local `nc -lvnp 4444` và thực thi payload ở target -> gain access -> flag 1 ở /var/www/user.txt.
- Privesc: Tìm SUID binary `find / -type f -perm -u=s 2>/dev/null` -> thấy một tệp lạ lạ /usr/bin/python với SUID. Chạy `/usr/bin/python -c 'import os; os.execl("/bin/sh", "sh", "-p")'` để sinh ra một root shell. Flag 2 ở /root/root.txt.
- Nhận xét: Đây là một bài CTF boot2root ở mức easy, chỉ cần có kiến thức cơ bản về recon, upload vulns, reverse shell, privesc là có thể giải được full.
- Happy Hacking!
