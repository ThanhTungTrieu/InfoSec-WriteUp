- Recon:
	* Nmap: 3 opening port:
		* 21 ftp
		* 80 http
		* 2222 ssh
	* Gobuster: 
		* /simple -> Trang CMS Made Simple version 2.2.8
- Gain Access: Searchsploit CMS Made Simple -> Có lỗ hổng SQLi với CMS Made Simple < 2.2.10 mã CVE-2019-9053. Exploit theo CVE, dump được Salt for passwd, Username, Email, password hash. Thay vì cố gắng crack password với hash (có salt), mình chọn cách dùng hydra để brute force vào ssh port 2222 với username dump được. Sau khi tìm được mật khẩu sau khi brute force, ssh vào với username mitch (dump được từ bước trên) và port 2222. -> Gain access. Flag 1 ở /home/mitch/user.txt.
- Privesc: Chạy `sudo -l`, phát hiện /usr/bin/vim có thể chạy sudo không cần passwd. Khai thác bằng cách chạy `sudo vim -c ':!/bin/sh'`, nhận được root shell. Flag 2 ở /root/root.txt.
- Happy Hacking!
