
#1. Cài đặt BIND

- Cài đặt BIND chỉ đơn giản là cài đặt gói bind9. Thông qua dòng lệnh:

**sudo apt-get install bind9**

<img src="http://i.imgur.com/gnb2MxE.png">

Sau đó chọn Y để tiếp tục cài đặt.

<img src="http://i.imgur.com/Cbqhufc.png">

Ubuntu sẽ tiếp hàng tải các gói cần thiết cho cài đặt và cài đặt vào máy. Sau khi cài đặt thành công vào máy sẽ có thông báo như sau:

<img src="http://i.imgur.com/VYD3jdj.png">


#2. Các dịch vụ của BIND9 và một vài khái niệm

- BIND9 cung cấp khá nhiều loại máy chủ DNS(DNS server) cho chúng ta dùng cho những mục đích khác nhau. Các máy chủ(server) đó gồm có:
<ul>
<li>Máy chủ cache(Cache Server): với cấu hình sử dụng máy chủ cache nội dung truy vấn của các máy khách( client) và sẽ sử dụng lại cho các lần truy vấn sau. Điều này rất hữu dụng cho các hệ thống hạ tầng mà kết nối internet chậm(do kết quả được lưu trữ lại lên không phải yêu cầu truy vần nào từ phía máy khách(client) đều phải truy xuất ra internet). Sử dung máy chủ cache sẽ giúp làm giảm băng thông cũa như độ trễ của dịch vụ DNS.</li>
<li>Máy chủ DNS chính(Primary Master Server): có thể sử dụng phục vụ các bản ghi DNS( DNS records)(Nhóm các bản ghi này thường có tên gọi khác là zones) mà các tên miền đã được đăng ký.</li>
<li>Máy chủ DNS phụ(Secondary Master Server): sử dụng cho việc hoàn thiện máy chủ DNS chính thông qua việc sao chép lại các cấu hình bản ghi DNS từ máy chủ DNS chính. Một máy chủ DNS phụ là cần thiết trong các triển khai lớn. Hãy hình dung rằng khi khai triển một hệ thống lớn chúng ta phải luôn bảo đảm việc dịch vụ DNS phải đáp ứng cho dù máy chủ DNS chính có hoạt động hay không( khi máy chủ DNS chính không hoạt động do một sự cố thì máy chủ DNS phụ sẽ cung cấp dịnh vu DNS và như thế khi khắc phục được sự cố thì máy chủ DNS chính sẽ tiếp tục cung cấp dịch vụ DNS, điều này giúp cho hệ thống luôn đáp ứng và không bị ngừng lại).</li>
</ul>
- Các loại bảng ghi DNS(DNS Record Types):

+ Bảng ghi địa chỉ(còn gọi là A records) là loại bảng ghi thông dụng nhất, kiểu bảng ghi này gắn địa chỉ IP vào tên của máy.

www IN A 1.2.3.4

+ Bảng ghi bí danh(Alias Records)(còn gọi là bản ghi CNAME) từ một đi một bảng ghi địa chỉ. Bạn có thể tạo một bản ghi CNAME chỉ vào bảng ghi CNAME khác. Tuy nhiên, nó tăng gấp đôi số lượng các yêu cầu đến máy chủ DNS, do đó cách làm này là một cách không hiệu quả.

  mail IN CNAME www
  www IN A 1.2.3.4

+ Bảng ghi trao đổi mail(Mail Exchange Records)( còn gọi là MX records) dùng để định nghĩa nơi mà mà mail gửi tới và gửi tới theo thứ tự như thế nào. Bảng ghi này nhất định phải gắn vào bảng ghi địa chỉ, không gắn vào bảng ghi CNAME. Sẽ tồn tại nhiều bảng ghi này nếu như có nhiều mày chủ cùng phụ trách nhận mail cho một tên miền.

  IN MX 10 mail.example.com.

  mail IN A 1.2.3.4

+ Bảng ghi máy chủ tên(Name Server Records): Dùng định nghĩa máy chủ phục vụ bảng copy của các bản ghi. Bảng ghi củng phải gắn vào một bảng ghi địa chỉ, không sử dụng bảng ghi CNAME.

  IN NS ns.example.com.

  ns IN A 1.2.3.4


#2. Cài đặt máy chủ Master DNS

- Trong toàn bộ cấu hình máy chủ DNS thì chúng ta sẽ chỉ đề cập đến việc cấu hình DNS cho địa chỉ example.com.

<img src="http://2.bp.blogspot.com/-HtAGELPznvE/UKWOq3IlwNI/AAAAAAAAAFc/KG7kHHeEndQ/s1600/dns-model-2.png">

- Trong sơ đồ, máy chủ sẽ tìm thông tin đầu tiên trong tập tin cấu hình /etc/bind/named.conf và tập tin /etc/bind/named.conf.local( là các tập tin named.conf trong hình). Các tập tin này sẽ khai báo về các tập tin khu vực( zone file), các tâp tin khu vự này sẽ định nghĩa chi tiết hơn các thông tin về tên miền.
Xét một mô hình thực tế như sau:

- Cấu hình hunter.com vào IP 10.0.0.1, www.hunter.com vào IP 10.0.0.2, cấu hình email và địa chỉ email là mail.hunter.com vào IP 10.0.0.3. 

- Đầu tiên cần khai báo hunter.com trong tập tin /etc/bind/named.conf.local

  zone "hunter.com" {
  type master;
  file "/etc/bind/db.hunter.com";
  };


- Như vậy tập tin khu vực( zone file) sẽ là tập tin có đường dẫn là /etc/bind/db.example.com. Để tạo tập tin này /etc/bind/db.hunter.com dễ dàng sao chép tập tin /etc/bind/db.local.

**sudo cp /etc/bind/db.local /etc/bind/db.hunter.com**

- Khi mở tập tin db.hunter.com sẽ có nội dung như sau

;
; BIND data file for local loopback interface
;
$TTL 604800
@ IN SOA localhost. root.localhost. (
2 ; Serial
604800 ; Refresh
86400 ; Retry
2419200 ; Expire
604800 ) ; Negative Cache TTL
;
@ IN NS localhost.
@ IN A 127.0.0.1
@ IN AAAA ::1

- Ta cần thay đổi localhost thành hunter.com và địa chỉ 127.0.0.1 thành địa chỉ 10.0.0.1. Nội dung của db.hunter.com mới có nội dung như sau:

;
; BIND data file for local loopback interface
;
$TTL 604800
@ IN SOA example.com. root.example.com. (
3 ; Số Serial
604800 ; Thời gian Refresh
86400 ; Thời gian Retry
2419200 ; Thời gian hết hạn Expire
604800 ) ; Negative Cache TTL
;
@ IN NS example.com.;Bản ghi máy chủ tên
@ IN A 192.168.56.101
Sau mỗi lần thay đổi nội dung cần khởi động lại bind9

  **sudo /etc/init.d/bind9 restart**

- Để kiểm tra lại thông tin server DNS có được cấu hình một các đúng đắn có thể dùng lệnh dig nhưng trước đó cần cấu hình để máy kiểm tra đó sử dụng DNS server của chúng ta( Ví dụ: máy server DNS mà chúng ta cái đặt có IP là 192.168.1.101). 

- Sau khi đã khởi động lại bind9 và cấu hình DNS server cho card mạng, chúng ta kiểm tra lại cấu hình thông qua lệnh dig

<img src="http://i.imgur.com/epZ7IRo.png">

- Như trong hình địa chỉ hunter.com nay đã được hiểu là IP 10.0.0.1
Để cấu hình www.hunter.com thành IP 10.0.0.2 thì cần thêm khai báo www( bảng ghi địa chỉ) trong tập tin db.example.com

www IN A 192.168.56.102

- Sau khi thay đổi xong khởi động lại bind9 và kiểm tra lai bằng lệnh dig

<img src="http://i.imgur.com/RZ2tNFx.png">

- Trong hình trên kết quả là www.hunter.com đã hiểu như là IP 10.0.0.2

- Để khai báo email cần khai báo bản ghi trao đổi mail( MX record) cho địa chỉ email và cần khai báo thêm mail trong tập tin db.hunter.com

IN MX 10 mail.hunter.com.
mail IN A 10.0.0.3

- Dòng đầu trong cấu hình trên cho biết là địa chỉ mail.example.com sẽ được dùng để trao đổi email, dòng thứ hai cho biết địa chỉ IP của mail.example.com là 192.168.56.103. Khởi động lại bind9 và kiểm tra lai bằng lệnh dig

<img src="http://i.imgur.com/le2Y6eo.png">

