Port scan:

![enter image description here](https://imgur.com/sve6vH8.png)

![enter image description here](https://imgur.com/X0i6syV.png)

SMB enum:

![enter image description here](https://imgur.com/7cQCWnh.png)

Truy cập SMB lấy được 2 file ảnh nhưng không có thông tin hữu ích từ metadata hay steg.

![enter image description here](https://imgur.com/UWMnytn.png)

Bỏ qua SMB, truy cập vào FTP (cho phép anonymous login), lấy được 3 file lần lượt là: clean.sh; removed_files.log; to_do.txt.

![enter image description here](https://imgur.com/9vP6S07.png)

Nội dung của của các file:

![enter image description here](https://imgur.com/VWmgy6y.png)

![enter image description here](https://imgur.com/bFWg9gW.png)

![enter image description here](https://imgur.com/5piSEXJ.png)

Dựa vào nội dung file removed_files.log, có vẻ như file clean.sh được thực thi nhiều lần (có thể là cron job?). 
Để kiểm tra suy đoán trên, sửa nội dung file clean.sh để nó sinh log khác đi so với ban đầu, put lên bằng FTP và đợi để xem liệu file removed_files.log có thay đổi hay không.

Sửa nội dung file clean.sh:

![enter image description here](https://imgur.com/Sg1TfgG.png)

Put lên bằng FTP:

![enter image description here](https://imgur.com/qzuoD70.png)

Đợi 1 lúc rồi get file removed_files.log về local:

![enter image description here](https://imgur.com/fGlHvFU.png)

Nội dung file removed_files.log mới:

![enter image description here](https://imgur.com/btwOHGr.png)

Vậy là chính xác file clean.sh đang được thực thi dưới dạng cron job.
Sửa nội dung file clean.sh, put lên bằng FTP, mở netcat listener ở local và đợi để nhận được reverse shell:

![enter image description here](https://imgur.com/rHvrVHK.png)

![enter image description here](https://imgur.com/kTQGEZb.png)

Lấy được user flag ở /home/namelessone/user.txt.

User namelessone nằm trong group lxd. Đây là group có thể lợi dụng để leo thang đặc quyền lên root.

Clone repo lxd-alpine-builder từ github để build lxd image:

![enter image description here](https://imgur.com/iv5zosL.png)

Sau đó switch sang root, lần lượt chạy `cd lxd-alpine-builder` và `./build-alpine` để build image. Build thành công thì thư mục có dạng như sau:

![enter image description here](https://imgur.com/ScTLhfi.png)

Dùng module http server của python để gửi image đến target.

Ở target, dùng wget để tải image:

![enter image description here](https://imgur.com/vLNTMgP.png)

Khởi tạo lxd, cứ nhấn enter cho đến khi thành công:

![enter image description here](https://imgur.com/YPahF0I.png)

Add image vào lxd:

![enter image description here](https://imgur.com/c1D7slj.png)

Exec lần lượt:

```
lxc init myimage ignite -c security.privileged=true
lxc config device add ignite mydevice disk source=/ path=/mnt/root recursive=true
lxc start ignite
lxc exec ignite /bin/sh
```

![enter image description here](https://imgur.com/sXhnZp8.png)

Đã thành công leo thang đặc quyền root.
Lấy root flag:

```
cd mnt/root/root
ls -la
cat root.txt
```

![enter image description here](https://imgur.com/zOHoLBh.png)

Happy Hacking!
