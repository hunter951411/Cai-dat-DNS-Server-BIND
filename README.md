# DNS-Server-BIND

DNS Server: Bind
Đây là dịch vụ cơ bản đầu tiên và quan trọng nhất của Internet. DNS
là quan trọng vì nếu DNS hoạt động sai hoặc không hoạt động, toàn
bộ phần mạng Internet liên quan sẽ bị tê liệt hoàn toàn. Hiểu rõ DNS
rất quan trọng với quản trị viên máy chủ có kết nối Internet. Nó cho
phép quản trị viên tìm ra nhanh chóng các nguyên nhân của các trục
trặc trên mạng.
DNS nói một cách đơn giản là dịch vụ cho phép ánh xạ , chuyển đổi
tên của một hệ thống nối Internet ra địa chỉ IP của nó. Nguyên nhân
của sự tồn tại DNS là do con người có thói quen đặt tên cho các
trang thiết bị mà các trang thiết bị thì lại chỉ có thể dùng số để liên
lạc với nhau. Vào những thời kỳ đầu tiên của Internet, người ta lập
bảng về mối liên hệ giữa tên và địa chỉ IP và cài đặt trên một máy
tính để tất cả cùng tham khảo. Nhưng với sự phát triển quá nhanh
của Internet, bảng này phát triển nhanh chóng và không một máy
nào có thể hoàn thành nổi nhiệm vụ tuy đơn giản nhưng lại rất quan
trọng này. Hơn nữa, mỗi thay đổi dù ở đâu cũng phải thông qua
server trung tâm. Điều này trở nên không thể chấp nhận được vì
luôn có thay đổi trên Internet. Một giải pháp được cộng đồng Internet
chấp nhận là chia toàn bộ không gian các địa chỉ IP và tên ra thành
các nhóm logic nhỏ hơn . Mỗi nhóm có quyền tổ chức thông tin của
các máy của mình.
Như vậy bước đầu tiên, một máy nối vào Internet, không phụ thuộc
vào việc nó có chạy hay không DNS server, phải được cấu hình
resolver, tức là chỉ ra cách thức hành động khi có yêu cầu phân giải
địa chỉ. Resolver được cấu hình qua tập tin /etc/host.conf :
[root@pasteur tnminh]# more /etc/host.conf
order hosts,bind
multi on
· Dòng thứ nhất của /etc/host.conf cho biết khi có yêu cầu
phân giải tên, resolver sẽ xem xét đầu tiên tập tin /etc/hosts
sau đó đến sử dụng DNS server (bind).
Redhat Linux
26 / 80
· Dòng thứ hai cho phép một host có nhiều địa chỉ IP trong tập
tin /etc/hosts.
Tập tin /etc/hosts chính là tiền thân của dịch vụ DNS. Hiện nay,
/etc/hosts chỉ còn thường lưu các địa chỉ của mạng nội bộ hay dùng
tới nhất đối với một máy. Khi yếu cầu phân giải vượt qua khả năng
trả lời của /etc/hosts từ khóa bind chỉ ra cần phải sử dụng dịch vụ
DNS. BIND là viết tắt của Berkeley Internet Name Domain và một
triển khai rộng rãi nhất của dịch vụ DNS hiện nay.
Khi đó, resolver cần thông tin tiếp theo về DNS server. Thông tin này
lưu trữ trong tập tin /etc/resolv.conf. Tập tin này kiểm tra cách
resolver sử dụng DNS để phân giải địa chỉ . Nó quyết định DNS
server cụ thể cần phải truy vấn và cách bổ sung phần domain cho
phần tên của máy. Ví dụ một tập tin /etc/resolv.conf
[root@linuxsrv root]# more /etc/resolv.conf
search mcsevietnam.com
nameserver 192.168.2.10
[root@linuxsrv root]#
Dòng đầu tiên cho phép resolver không chỉ phân giải tên như
chương trình client yêu cầu, mà trong trường hợp phân giải không
thành công, tiếp tục thử phân giải tên với phần domain tiếp nối sau.
Ví dụ bạn muốn tìm địa chỉ máy khangves . Nếu quá trình phân giải
khangves không thành công, resolver sẽ thử phân giải
khangves.mcsevietnam.com. Dòng tiếp theo là địa chỉ của name
server cần phải truy vấn. Nhớ rằng địa chỉ của name server là số IP
chứ không phải là tên, vì nếu ngược lại, ai sẽ là người phân giải tên
cho máy làm nhiệm vụ phân giải tên?
Bây giờ chúng ta sẽ chuyển qua xem xét đến cấu hình của bản thân
name server. Chương trình server của DNS name server là một
chương trình daemon named (đọc là nêm đê). Named thường được
khởi động ngay từ đầu cùng với khởi động của hệ thống. Thường thì
named được chạy thông qua một script trong /etc/rc.d/rc3.d/named .
Trong quá trình khởi động named đọc các tập tin dữ liệu rồi chờ các
yêu cầu phân giải qua cổng xác định trong tập tin /etc/service (thông
thường là cổng 53). Named dùng đầu tiên là giao thức UDP để phân
giải tên, nếu phân giải bằng UDP không có kế quả, named sẽ dùng
TCP sau đó .
Biên soạn bởi mcsevietnam
27 / 80
Tập tin đầu tiên được named tham chiếu là /etc/named.conf. Nội
dung tập tin này của Linux Redhat 7.3 được cài mặc định là :
options {
directory "/var/named";
};
zone "." {
type hint;
file "root.hints";
};
zone "0.0.127.in-addr.arpa" {
type master;
file "pz/127.0.0";
};
Mở đầu là từ khóa options cho phép nhập các tùy chọn (options)
toàn cục. directory "/var/named"; cho biết là các tập tin sau đây sẽ là
tương đối đối với thư mục
này.
Ta có thể bổ sung thêm trong phần options dòng lệnh :
forwaders {205.15.2.10 ; 193.214.2.12;};
Khi đó, DNS server của chúng ta sẽ tham chiếu các name server
205.15.2.10; 193.214.2.12 mỗi khi nó không tìm thấy câu trả lời
trong dữ liệu mà nó có . Sau phần tham số toàn cục options, ta thấy
các khối zone “tên_zone “ { type master (hoặc slave hoặc hint); file
“tên_tập_tin”; }; liên tiếp nhau.
Đối với mỗi domain, chúng ta cần 2 tập tin dữ liệu. Tập tin thứ nhất
lưu trữ các dữ liệu liên quan đến phân giải “xuôi “ từ name sang IP
và tập tin thứ hai để phân giải “ngược“ từ IP ra name. Trừ miền “.”
có tính chất giúp đỡ là có tập tin cache đặc biệt
; There might be opening comments here if you already have
this file.
; If not don't worry.
;
. 6D IN NS G.ROOT-SERVERS.NET.
. 6D IN NS J.ROOT-SERVERS.NET.
. 6D IN NS K.ROOT-SERVERS.NET.
. 6D IN NS L.ROOT-SERVERS.NET.
. 6D IN NS M.ROOT-SERVERS.NET.
Redhat Linux
28 / 80
. 6D IN NS A.ROOT-SERVERS.NET.
. 6D IN NS H.ROOT-SERVERS.NET.
. 6D IN NS B.ROOT-SERVERS.NET.
. 6D IN NS C.ROOT-SERVERS.NET.
. 6D IN NS D.ROOT-SERVERS.NET.
. 6D IN NS E.ROOT-SERVERS.NET.
. 6D IN NS I.ROOT-SERVERS.NET.
. 6D IN NS F.ROOT-SERVERS.NET.
G.ROOT-SERVERS.NET. 5w6d16h IN A 192.112.36.4
J.ROOT-SERVERS.NET. 5w6d16h IN A 198.41.0.10
K.ROOT-SERVERS.NET. 5w6d16h IN A 193.0.14.129
L.ROOT-SERVERS.NET. 5w6d16h IN A 198.32.64.12
M.ROOT-SERVERS.NET. 5w6d16h IN A 202.12.27.33
A.ROOT-SERVERS.NET. 5w6d16h IN A 198.41.0.4
H.ROOT-SERVERS.NET. 5w6d16h IN A 128.63.2.53
B.ROOT-SERVERS.NET. 5w6d16h IN A 128.9.0.107
C.ROOT-SERVERS.NET. 5w6d16h IN A 192.33.4.12
D.ROOT-SERVERS.NET. 5w6d16h IN A 128.8.10.90
E.ROOT-SERVERS.NET. 5w6d16h IN A 192.203.230.10
I.ROOT-SERVERS.NET. 5w6d16h IN A 192.36.148.17
F.ROOT-SERVERS.NET. 5w6d16h IN A 192.5.5.241
Đây thực chất là địa chỉ IP của các name server gốc (root) của
Internet.
  Ví dụ như đối với miền mcsevietnam.com ta cần có :
zone "mcsevietnam.com" {
type master;
file "db.mcsevietnam.com";
};
zone "1.16.172.in-addr.arpa" {
type master;
file "db.172.16.1";
Chú ý các viết cú pháp 1.16.172.in-addr.arpa cho tên của miền phân
giải ngược IP ra name.
Sau đây ta sẽ xem xét đến cấu trúc tập tin
/var/named/db.mcsevietnam.com
@ IN SOA mcsevietnam.com. root.mcsevietnam.com. (
199609206 ; serial, todays date + todays serial #
8H ; refresh, seconds
Biên soạn bởi mcsevietnam
29 / 80
2H ; retry, seconds
1W ; expire, seconds
1D ) ; minimum, seconds
NS mcsevietnam.com.
MX 10 mcsevietnam.com. ; Primary Mail Exchanger
TXT "MCSEVIETNAM Corporation"
localhost A 127.0.0.1
mcsevietnam.com. A 172.16.1.1
linuxsrv A 172.16.1.1
www A 172.16.1.1
ftp CNAME mcsevietnam.com.
mail CNAME mcsevietnam.com.
news CNAME mcsevietnam.com.
Ký tự “@” đầu tiên thay cho miền mcsevietnam.com; IN là Internet ;
SOA là Start Of Authority; tiếp nối bởi tên miền và địa chỉ người chịu
trách nhiệm. Chú ý là trong địa chỉ email của người chịu trách nhiệm,
dấu @ quen thuộc được thay bằng dấu chấm “.”. Sau các tên miên
có dấu chấm “.” ở cuối. Trong tất cả các tập tin dữ liệu của DNS,
những tên không kết thúc bởi dấu chấm sẽ được DNS server thêm
vào bởi tên miền tương ứng của tập tin đó. Ví dụ đây là tập tin ứng
với miền mcsevietnam.com, khangves sẽ được bổ sung thêm thành
khangves.mcsevietnam.com.
Sau phần ngoặc đơn với 5 số miêu tả số serie và các thông số thời
gian của thông tin, bắt đầu các dòng (record) dữ liệu. Khoảng trắng
ở đầu dòng tương đương với tên miền (như dấu @), NS ám chỉ
record dạng nameserver. MX là mail exchange, dùng để chỉ ra máy
chịu trách hiệm nhận thư điện tử cho domain này. Số 10 là múc độ
ưu tiên cho mail server này. Độ ưu tiên sẽ càng cao nếu số càng
nhỏ . A là viết tắt của Address, sẽ tiếp theo bởi một địa chỉ IP.
CNAME là canonical name . Với CNAME ta có thể gán cho máy biệt
danh tùy ý tiện cho việc sử dụng. Các dòng bắt đầu bởi ; là các chú
thích.
Ví dụ tập tin dùng cho phân giải ngược /var/named/db.172.16.1
@ IN SOA mcsevietnam.com. root.mcsevietnam.com. (
199609206 ; Serial
28800 ; Refresh
7200 ; Retry
604800 ; Expire
Redhat Linux
30 / 80
86400) ; Minimum TTL
NS mcsevietnam.com.
;
; Servers
;
1 PTR simbahcm.mcsevietnam.com.
2 PTR trantungbtre.mcsevietnam.com.
3 PTR hungden.mcsevietnam.com.
;
Cấu trúc tập tin /var/named/db.172.16.1 có phần đầu giống hệt như
tập tin phân giải xuôi. Chỉ có từ khóa PTR = Pointer là khác.
Việc cấu hình các dữ liệu của name server cần rất thận trọng vì
nhiều khi lỗi của nó rất khó tìm. Mỗi khi chúng ta thay đổi dữ liệu,
cần phải khởi động lại named bằng các sử dụng kill –9 named_PID
để dừng named rồi khởi động lại bằng cách nhập dòng lệnh named.
Tập tin /var/log/messages có thể giúp đỡ nhiều để tìm ra lỗi nếu
named không hoạt động theo ý chúng ta muốn. Để thử hoạt động
của quá trình phân giải tên, Linux có lệnh nslookup với nhiều tính
năng rất mạnh. Xem manpage của nslookup để biết cách sử dụng.
