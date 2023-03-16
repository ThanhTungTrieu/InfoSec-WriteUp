Port scan:

![enter image description here](https://imgur.com/T9pXwas.png)
![enter image description here](https://imgur.com/ZSCwihq.png)
URL Brute-force:

![enter image description here](https://imgur.com/Vo6vXEr.png)

Web app:

![enter image description here](https://imgur.com/EpSkRNg.png)

File robots.txt:

![enter image description here](https://imgur.com/basLoMk.png)

Truy cập /flag_1.txt và lấy đươc flag1.
Truy cập /secret_file_do_not_read.txt thì bị 403.
Click vào một sản phẩm bất kỳ ở trang chủ sẽ được redirect đến url có dạng `http://10.10.210.4/post.php?post=round.php`. Ở đây có tồn tại lỗ hổng LFI. Khai thác LFI để đọc /etc/passwd:

![enter image description here](https://imgur.com/ZOuOjpd.png)

Có 3 user đáng chú ý là : will, mat, toby.
Khai thác LFI để đọc file secret_file_do_not_read.txt:

![enter image description here](https://imgur.com/BELX8lX.png)

FTP vào server với credentials ở trên:

![enter image description here](https://imgur.com/bN3qNRS.png)

Lấy được flag2:

![enter image description here](https://imgur.com/9bpLvur.png)

Nội dung trong file secret_do_not_read.txt có nhắc đến việc các file upload lên qua FTP được lưu tại /home/ftpuser/ftp/files, chính là thư mục files ở giao diện đầu tiên hiện ra khi vào FTP. Ý tưởng ở đây là upload payload lên thư mục files qua FTP, và thực thi payload thông qua LFI.

![enter image description here](https://imgur.com/zvdUI9Q.png)

Truy cập `http://10.10.210.4/post.php?post=/home/ftpuser/ftp/files/php-reverse-shell1.php` để thực thi payload.
Ở local nhận được shell và lấy flag3 ở /var/www/html/more_secrets_a9f10a/flag_3.txt:

![enter image description here](https://imgur.com/ZVUEUN6.png)

Check sudo right của user www-data thấy user được phép dùng sudo bằng quyền của user toby với mọi lệnh. Lợi dụng điều đó để lấy flag4 tại /home/toby/flag_4.txt (mode 600):

![enter image description here](https://imgur.com/0W2oxec.png)

Flag5 ở /home/mat/flag_5.txt (Owner: mat, mode 600). Để lấy được flag5, ta cần privesc sang user mat. 
Check crontab thấy có cronjob thực hiện bởi mat, file thực thi ở /home/toby/jobs/cow.sh (owner: toby, mode 755). Do gặp khó khăn trong việc chỉnh sửa nội dung ngay trên shell, ta sẽ xóa file cow.sh ở target, tạo 1 file cow.sh khác chứa payload ở local, dùng wget để đưa payload lên target.

Ở local:

![enter image description here](https://imgur.com/JpcGvcg.png)
![enter image description here](https://imgur.com/cnAyUhV.png)

Ở target:

![enter image description here](https://imgur.com/T3Mm6v3.png)

Sau khi đưa được payload lên target thành công, mở listener ở local, đợi khi cronjob thực thi để có được mat shell. Lần này ta sẽ dùng Metasploit để bắt reverse shell, thuận tiện cho việc chỉnh sửa file trên shell (lát nữa sẽ cần).

![enter image description here](https://imgur.com/SR0vmki.png)

Flag6 ở /home/will/flag_6.txt (owner: will, mode 600). Ta cần privesc sang will để lấy được flag. 
Check sudo right, mat được phép chạy sudo quyền will với duy nhất lệnh `usr/bin/python3 /home/mat/scripts/will_script.py *` (* ở đây là tham số bất kỳ).

![enter image description here](https://imgur.com/kwGmrNn.png)

Source file will_script.py (owner: will, mode 644):

![enter image description here](https://imgur.com/liZDNRO.png)

Chương trình này import và gọi đến hàm get_command từ cmd.py (nằm cùng thư mục, owner mat, mode 644). Chỉnh sửa source file cmd.py để nó spawn shell quyền will mỗi khi được gọi đến.
Source file cmd.py ban đầu:

![enter image description here](https://imgur.com/XucsJjW.png)

Upgrade lên meterpreter:

![enter image description here](https://imgur.com/eVd2Fsp.png)

Sửa source file cmd.py thành:

![enter image description here](https://imgur.com/5MXItGp.png)

Quay trở lại shell, thực thi file will_script.py để có đặc quyền của will:

![enter image description here](https://imgur.com/XdakvNB.png)

Thành công privesc sang will. Lấy flag6 ở /home/will/flag_6.txt

![enter image description here](https://imgur.com/KQtZTuJ.png)

Bước cuối cùng, để lấy flag7 cần có quyền root.
User will nằm trong group adm. Dùng `file` để kiểm tra các file, folder có liên quan đến group adm:

![enter image description here](https://imgur.com/oXu2BTm.png)

File /opt/backups/key.b64 có vẻ đáng ngờ (owner: root, group: adm, mode 660):

![enter image description here](https://imgur.com/CJkaVUz.png)

Kiểm tra nội dung file:

![enter image description here](https://imgur.com/8iZFAhG.png)

Rõ ràng đây là định dạng base64. Sau khi decode:

![enter image description here](https://imgur.com/CUDV3z1.png)

Đây là ssh private key. Lưu lại vào 1 file ở local, chmod 600 cho file đó, dùng nó để ssh vào server.

![enter image description here](https://imgur.com/dli2urI.png)

Thành công privesc lên root. Lấy flag7 ở /root/flag_7.txt.

Happy Hacking!
