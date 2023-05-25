Port scan:

![enter image description here](https://imgur.com/zwDKguw.png)

![enter image description here](https://imgur.com/Ydv9vS6.png)

URL Brute-force:

![enter image description here](https://imgur.com/Ge66aex.png)

8080/TCP - HTTP:

![enter image description here](https://imgur.com/QznGCQX.png)


Truy cập vào /manager

![enter image description here](https://imgur.com/d9n2oah.png)

Yếu cầu auth -> Nhập thử admin:admin -> Không đúng -> Cancel

![enter image description here](https://imgur.com/ke22dSO.png)


Thông báo lỗi chứa credentials -> Quay lại đăng nhập với credentials là tomcat:s3cret -> Đăng nhập thành công.

![enter image description here](https://imgur.com/A0seA1A.png)


Deploy reverse shell payload ở phần War file to deploy:

![enter image description here](https://imgur.com/hUGdvUM.png)

Sau khi deploy:

![enter image description here](https://imgur.com/jVhzvra.png)

Mở listener ở local, truy cập vào target:8080/revshell và nhận được reverse shell.

![enter image description here](https://imgur.com/VL53USa.png)

Lấy user flag ở /home/jack/user.txt

![enter image description here](https://imgur.com/MKV9RP0.png)

Check crontab:

![enter image description here](https://imgur.com/cxJGUHO.png)

Có 1 cronjob chạy bằng user root, thực thi file id.sh. Check file id.sh:

![enter image description here](https://imgur.com/QioPb9e.png)

File này được phép chỉnh sửa từ mọi user. Đưa payload vào file id.sh và mở listener ở local để nhận được root shell:

![enter image description here](https://imgur.com/I7YtGaj.png)

Ở local:

![enter image description here](https://imgur.com/75n1IwA.png)

Lấy root flag ở /root/root.txt

![enter image description here](https://imgur.com/fVMNdRl.png)

Happy Hacking!
