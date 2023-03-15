Port scan:

![enter image description here](https://imgur.com/CbfW04f.png)

![enter image description here](https://imgur.com/5L8cAxv.png)

Có 3 port đang mở: 21 FTP, 22 SSH, 80 HTTP.  Truy cập vào http:

![enter image description here](https://imgur.com/EVPix5O.png)

URL Brute-force không cho kết quả khả quan. Xem qua page source:

![enter image description here](https://imgur.com/I04MdDH.png)

Source có nhắc đến Steganography. Điều đó làm mình nghĩ đến có thông điệp ẩn sau ảnh background. Download ảnh từ http://10.10.1.141/brooklyn99.jpg và sử dụng Steghide để kiểm tra nhưng bị yêu cầu cung cấp passphrase. Vì target có FTP đang open và cho phép anonymous login, ta ftp vào target bằng anonymous user.

![enter image description here](https://imgur.com/DrZmDaA.png)

Có file tên `note_to_jake.txt`, get về local. Nội dung file `note_to_jake.txt`:

![enter image description here](https://imgur.com/rbushIG.png)

Nội dung chỉ ra rằng mật khẩu của jake là yếu, mình brute-force vào ftp với user là jake để hy vọng tìm được steg passphrase nhưng không thành công. Sau đó mình brute-force vào ssh với user jake và bất ngờ thành công (dễ hơn dự đoán). Có thông tin đăng nhập của jake rồi thì không cần quan tâm đến steghide nữa, có lẽ chỉ là đánh lạc hướng.

![enter image description here](https://imgur.com/IRprvVs.png)

Có được quyền truy cập, lấy user flag ở /home/holt/user.txt

![enter image description here](https://imgur.com/0y7kQio.png)

Đến lúc root privesc. Kiểm tra sudo right của user jake, phát hiện jake có thể sử dụng lệnh less quyền root mà không cần password. Lợi dụng việc đó, chạy lệnh `sudo less user.txt` và chạy `!/bin/sh` khi đã ở trong giao diện less. Leo thang đặc quyền thành công và lấy root flag:

![enter image description here](https://imgur.com/4FlQFow.png)

Ngoài ra, còn có cách khác để leo thang đặc quyền root. Cách này sử dụng SUID binary. Tìm các binary có SUID bằng `find / -type f -perm -u=s 2>/dev/null` và tìm thấy /bin/less có gán SUID bit. Chỉ cần dùng lệnh `less /root/root.txt` để đọc root flag mà không cần đặc quyền root.

Happy Hacking!
