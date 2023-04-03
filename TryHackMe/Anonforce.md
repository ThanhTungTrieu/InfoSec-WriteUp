Port scan:

![enter image description here](https://imgur.com/R8AETDF.png)

![enter image description here](https://imgur.com/4MypHOa.png)

2 port đang open là 21, 22. FTP service trên target cho phép anonymous login.
FTP vào target, có thư mục `notread` khả nghi, cd vào và get các file về local:

![enter image description here](https://imgur.com/tikhqPC.png)

File `backup.pgp` có vẻ là file được mã hóa bằng PGP, còn file `private.asc` có vẻ là private key.
Lấy luôn user flag ở `/home/melodias/user.txt`:

![enter image description here](https://imgur.com/WVN1akF.png)

Vì private key được bảo vệ bằng passphrase nên ta dùng gpg2john và john để crack passphrase:

![enter image description here](https://imgur.com/kyjKXdN.png)

![enter image description here](https://imgur.com/2PraCg6.png)

Import private key vào gpg (nhập passphrase kèm theo), decrypt file backup.pgp:

![enter image description here](https://imgur.com/zc5Plwq.png)

![enter image description here](https://imgur.com/ReRVhHd.png)

Nice! File này lưu cả password hash của root luôn. Lưu password hash của root vào 1 file và tiếp tục dùng John The Ripper để crack:

![enter image description here](https://imgur.com/LIdY3wt.png)

Có được password của root, ssh vào target và lấy root flag:

![enter image description here](https://imgur.com/ZMUaEGl.png)

Happy Hacking!
