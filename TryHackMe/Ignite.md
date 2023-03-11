Port scan:

![enter image description here](https://imgur.com/Bi4IWz2.png)

Truy cập target machine (port 80):

![enter image description here](https://imgur.com/lAEm4rQ.png)

Ta thấy được ứng dụng web sử dụng Fuel CMS v1.4. Fuel CMS v1.4 tồn tại lỗ hổng RCE (CVE-2018-16763). Theo CVE, payload được sử dụng để RCE có dạng: `
url+"/fuel/pages/select/?filter=%27%2b%70%69%28%70%72%69%6e%74%28%24%61%3d%27%73%79%73%74%65%6d%27%29%29%2b%24%61%28%27"+quote(cmd)+"%27%29%2b%27"
`. Khai thác RCE, mở local listener và có được reverse shell:

![enter image description here](https://imgur.com/Kt5q2Eg.png)

Có được shell, lấy user flag ở /home/www-data/flag.txt

![enter image description here](https://imgur.com/qy6bF3C.png)

Để lấy được root flag, ta cần có được quyền root. Dùng linpeas để enum (https://github.com/carlospolop/PEASS-ng/tree/master/linPEAS), phát hiện machine dính CVE-2021-4034. Về CVE-2021-4034 (https://www.qualys.com/2022/01/25/cve-2021-4034/pwnkit.txt)

![enter image description here](https://imgur.com/TwGJUnl.png)

Để exploit, tạo 2 file lần lượt là evil-so.c và exploit.c. 
Nội dung file evil-so.c:

![enter image description here](https://imgur.com/qX3LDXS.png)

Nội dung file exploit.c:

![enter image description here](https://imgur.com/GAI4NCv.png)

Compile bằng gcc: 
`gcc -shared -o evil-so -fPIC evil-so.c`
`gcc exploit.c -o exploit`

Thực thi file exploit và leo thang đặc quyền thành công:

![enter image description here](https://imgur.com/1CLlmPe.png)

Sau khi chiếm được quyền root, lấy flag ở /root/root.txt.

Happy Hacking!
