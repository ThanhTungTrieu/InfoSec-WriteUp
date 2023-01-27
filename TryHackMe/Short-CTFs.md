# Complete Beginner
## Owasp Top 10
### Components with known vulnerabilities - Lab
- Link: https://tryhackme.com/room/owasptop10
- Cho trước 1 trang web 

    <iframe src="https://drive.google.com/file/d/1opL40MC6VXSkpeHjo-0Vsb3QrI03uBju/preview" width="640" height="480" allow="autoplay"></iframe>

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
	-- /modules              (Status: 301) [Size: 181] [--> /modules/]
/admin                (Status: 200) [Size: 1238]
/assets               (Status: 301) [Size: 179] [--> /assets/]
/Content              (Status: 301) [Size: 181] [--> /Content/]
/Assets               (Status: 301) [Size: 179] [--> /Assets/]
/Modules              (Status: 301) [Size: 181] [--> /Modules/]
/Admin                (Status: 200) [Size: 1238]
 - Nhận thấy /admin và /Admin có thể là một (có cùng size) và có status 200 => truy cập được. 
 - Truy cập /admin, ok đây là nơi để thực thi file (từ thư mục /modules).
 - Dùng gobuster để enum /modules nhưng không cho kết quả gì khả quan.
 - Dùng gobuster để enum /content với file được cho sẵn từ đề bài: `gobuster dir -u jewel.uploadvulns.thm/content -w ~/Downloads/UploadVulnsWordlist.txt -x jpg -t 250 --no-error` cho kết quả: 
	 -- /ABH.jpg              (Status: 200) [Size: 705442]
/LKQ.jpg              (Status: 200) [Size: 444808]
/SAD.jpg              (Status: 200) [Size: 247159]
/UAD.jpg              (Status: 200) [Size: 342033]
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
	--  /ABH.jpg              (Status: 200) [Size: 705442]
/LIP.jpg              (Status: 200) [Size: 383]
/LKQ.jpg              (Status: 200) [Size: 444808]
/SAD.jpg              (Status: 200) [Size: 247159]
/UAD.jpg              (Status: 200) [Size: 342033]
/ZBE.jpg			  (Status: 200) [Size: 250178]
- So với lần enum /content đầu tiên, ta có thêm 2 file khác: LIP.jpg và ZBE.jpg. Dự đoán: 1 trong 2 file này là payload js ta tải lên.
- Mở listener ở local sẵn: `nc -lnvp 4444`
- Truy cập http://jewel.uploadvulns.thm/content/ZBE.jpg cho kết quả là ảnh jpg thật mà ta upload trước đó.
- Truy cập http://jewel.uploadvulns.thm/content/LIP.jpg nhận được 1 thông báo lỗi display file => Dự đoán: Đó là file payload mà ta đã upload. Vì nội dung payload là code js nên không thể hiển thị dưới dạng jpg là đương nhiên.
- Truy cập http://jewel.uploadvulns.thm/admin để thực thi file. Ở đây có mô tả sẽ thực thi file từ thư mục /modules. Mà /modules và /content (nơi chứa payload) nằm cùng cấp => file cần thực thi từ /modules sẽ là ../content/LIP.jpg.
- Sau khi thực thi file ../content/LIP.jpg, ngay lập tức port 4444 tại local nhận được kết nối từ server để thực hiện shell. Tìm đến ../flag.txt để nhận flag.
- Lưu ý tên các file trên server là ngẫu nhiên.
- Happy Hacking!
