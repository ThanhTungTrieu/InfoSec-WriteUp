Port scan:

![enter image description here](https://imgur.com/kLEUKEM.png)

![enter image description here](https://imgur.com/VZWmrkl.png)

Web app:

![enter image description here](https://imgur.com/NvKzTLd.png)

URL brute force:

![enter image description here](https://imgur.com/bqBLJXu.png)

Web app sử dụng Wordpress, duy chỉ có url /hidden khá lạ. Truy cập /hidden:

![enter image description here](https://imgur.com/b23Rv2g.png)

Dự đoán có 3 user là C0ldd, Hugo, Philip.

Dùng wpscan để enum thêm về target:

![enter image description here](https://imgur.com/UA26R2S.png)

![enter image description here](https://imgur.com/4WZQ734.png)

Scan được 4 user (?) là: the cold in person, c0ldd, hugo, philip. 3 user kể sau có vẻ giống user hơn (mình đoán vậy).
Brute force vào wordpress login để tìm credentials:

```
wpscan --url 10.10.42.75 -U "c0ldd,hugo,philip" -P /usr/share/wordlists/rockyou.txt
```

Tìm được password của c0ldd:

![enter image description here](https://imgur.com/uDzqasw.png)

Truy cập /wp-login.php và đăng nhập với creds vừa tìm được.
Đăng nhập thành công và được redirect đến /wp-admin:

![enter image description here](https://imgur.com/crNhTI8.png)

Check ở /wp-admin/user.php, user c0ldd có role Administrator:

![enter image description here](https://imgur.com/zxLOs9P.png)

Điều đó giúp ta có toàn quyền kiểm soát web app. Chuyển đến `/wp-admin/theme-editor.php?file=404.php&theme=twentyfifteen` và chỉnh sửa nội dung file 404.php thành payload reverse shell:

![enter image description here](https://imgur.com/iQ1F89s.png)

Mở listener ở local và truy cập /wp-content/themes/twentyfifteen/404.php để có reverse shell:

![enter image description here](https://imgur.com/7hiwGdJ.png)

Ta cần có quyền của user c0ldd để lấy được user flag. Chuyển linpeas đến target bằng http server của python và enum bằng linpeas. Kết quả tìm được creds c0ldd có thể thử:

![enter image description here](https://imgur.com/yD8KMQ6.png)

Thành công chuyển sang user c0ldd với creds tìm được.
Lấy user flag ở /home/c0ldd/user.txt:

![enter image description here](https://imgur.com/GiIassY.png)

Check c0ldd sudo right và thấy có khá nhiều cách để privesc.

![enter image description here](https://imgur.com/NtqXfAj.png)

Ta sẽ lợi dụng vim để spawn root shell luôn:

![enter image description here](https://imgur.com/pObrOB2.png)


Lấy root flag ở /root/root.txt:

![enter image description here](https://imgur.com/vuHswcZ.png)

Happy Hacking!
