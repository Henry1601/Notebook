# Broken Access Control
## Access Control là gì?
- Access control hay Authorization là kiểm soát ủy quyền, cấp phép hoặc giới hạn mức độ truy cập đến tài nguyên trong hệ thống.
- Lưu phân biệt 2 khái niệm sau tránh nhầm lẫn:
	+ Authentication: xác định, kiểm tra danh tính người dùng
	> Trả lời cho câu hỏi "Bạn là ai?"
	
	+ Access Control - Authorization (như đã nói trên)
	> Trả lời cho câu hỏi "Bạn có quyền gì đối với hệ thống?"
## Phân loại Access Control
- Phân loại theo Security Model (mô hình bảo mật):
	+ DAC (Discretion): Mô hình kiểu phân tán, data owner có toàn quyền kiểm soát, cấp phép truy cập đối với dữ liệu của mình -> Tự do nhưng khó kiểm soát.
	+ MAC (Mandatory): Mô hình kiểu tập trung, sysadmin có toàn quyền với tất cả dữ liệu, data owner ko có ủy quyền với dữ liệu của mình -> An toàn nhưng bất tiện cho user.
	+ RBAC (Role-based): Phân quyền cho user trong hệ thống thành các vai trò, mỗi vai trò có quyền hạn cho phép riêng trong hệ thống -> Mô hình thông dụng nhất.
> Ngoài ra ở một số tài liệu khác còn phân loại nhiều hơn, nhưng về mô hình chung đều tương tự 1 trong 3 dạng trên hoặc kết hợp các dạng với nhau tùy vào nhu cầu quản lí đặc trưng của tổ chức.

- Phân loại access control:
	+ Vertical: phân quyền theo chiều dọc. VD: user thường ko thể thực hiện công việc của admin.
	+ Horizontal: phân quyền theo chiều ngang. VD: các users ko thể truy cập data của nhau.
	+ Context-based: phân quyền tùy trường hợp, ngữ cảnh hệ thống. VD: user của ngân hàng ko thể tự thay đổi số dư tài khoản của mình.
## Cơ chế bảo mật Access Control
### Access Control List (ACL)
- Hoạt động như network firewall filter các request dựa vào các rule và tùy vào mô hình access control. ACL chia thành các tầng ACE (Allowed hoặc Denied) để filter requests.
### Cross-Origin Resource Sharing (CORS)
Cơ chế kiểm soát requests dựa vào domain. Các header thường được dùng trong CORS:
- Access-Control-Allow-Origin: header này filter domain của nguồn gửi request, có thể chỉ đích danh như `Access-Control-Allow-Origin: http://www.example.com`.
	
 	+ Nói cách khác server chỉ xử lí những request đến từ nguồn này, các nguồn khác như https://www.example.com, http://example.net đều không xử lí. Header này có thể đc set value thành * (wildcard) để cho phép request từ mọi nguồn (đây chính là lỗ hổng).
	+ Ngoài ra, trong một số trường hợp value của header này được copy từ header `Origin` trong request, có nghĩa là nếu ta thay đổi value của `Origin` thì sẽ thay đổi được value của `Access-Control-Allow-Origin`.
- Access-Control-Allow-Credentials: header sẽ cho phép thông tin người dùng (username, password, cookie) có đc gửi kèm trong request hay không.	Nếu header này được set thành true thì tức là credential sẽ được gửi kèm trong request (lỗ hổng).

Thông thường CORS có chức năng preflight request (kiểm tra trước request) là cho user browser kiểm tra request header của server bằng method `OPTIONS`, ta có thể tận dụng cái này để kiểm tra value của request header, từ đó đưa ra hướng tấn công.
> Đọc thêm: [CORS - Misconfigurations & Bypass - HackTricks](https://book.hacktricks.xyz/pentesting-web/cors-bypass)
## Các kiểu tấn công bypass Access Control
- Insecure Direct Object Reference (IDOR): lỗ hổng cho phép truy cập data thông qua parameter trên URL hoặc request header (ID, username,...).
- Forced Browsing: enumerate, brute force các file hoặc folder tài nguyên của web hoặc của các users khác. Đọc thêm: [Tìm kiếm các lỗi IDOR, chưa bao giờ lại dễ đến thế với extension Autorize (viblo.asia)](https://viblo.asia/p/tim-kiem-cac-loi-idor-chua-bao-gio-lai-de-den-the-voi-extension-autorize-gDVK2z02KLj)
> Các tools thường sử dụng: gobuster, nikto, burp,...
- Directory Traversal (còn được gọi Path Traversal): lỗ hổng truy cập, tìm được dẫn đến file hoặc folder trong server
	+ Dùng đường dẫn tương đối: ../
	+ Dùng đường dẫn tuyệt đối, VD: /etc/passwd
	+ Dùng nested path (trong trường hợp firewall chặn ../): ....//
	+ Dùng các kí tự unicode thay thế (trong trường hợp chặn /): ..%C0%AF hoặc ..%25%2F
	+ Trong trường hợp firewall validate chuỗi bắt đầu, VD: đường dẫn bắt buộc phải bắt đầu bằng /var/www/images/ thì ta thêm đường dẫn tương đối vào sau
	/var/www/images/../../../etc/passwd
	+ Trong trường hợp firewall validate chuỗi kết thúc hoặc file extension, VD: kết thúc đường dẫn phải là .jpg, ta dùng kí tự null (%00)
	/etc/passwd%00.jpg
- Đọc thêm: [Hacking Starbucks and Accessing Nearly 100 Million Customer Records | Sam Curry](https://samcurry.net/hacking-starbucks/)
```bash
/..%2f  
/..;/  
/../  
/..%00/  
/..%0d/  
/..%5c  
/..\  
/..%ff/  
/%2e%2e%2f  
/.%2e/  
/%3f (?)  
/%26 (&)  
/%23 (#)
```
