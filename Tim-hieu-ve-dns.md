# Cai-dat-DNS-Server-BIND




#Giới thiệu DNS :

- Mỗi máy tính trong mạng muốn liên lạc hay trao đổi thông tin, dữ liệu cho nhau cần phải biết rõ địa chỉ IP của nhau. Nếu số lượng máy tính trong mạng nhiều thì việc nhớ những IP này là rất khó khăn.

- Mỗi máy tính ngoài địa chỉ IP ra còn có tên máy (host name). Đối với con người thì việc nhớ tên máy bao giờ cũng dễ nhớ hơn địa chỉ IP vì chúng có tính trực quang và gợi nhớ hơn. Do đó người ta tìm cách ánh xạ địa chỉ IP thành tên máy.

- Lúc đầu do quy mô của mạng APRA NET (tiền thân của mạng Internet ngày nay) chỉ có vài trăm máy nên chỉ có 1 tập tin hosts.txt lưu thông tin dùng để ánh xạ tên máy thành địa chỉ IP, do đó tên máy chỉ là 1 chuỗi văn bản không phân cấp (flat name). Tập tin này được lưu tại 1 máy chủ và các máy chủ khác lưu bản sao của nó. Khi quy mô mạng lớn hơn thì việc sử dụng file hosts.txt này bộc lộ các khuyết điểm sau :

##1. Lưu lượng mạng và máy chủ

- Chứa tập tin hosts.txt bị quá tải do hiệu ứng “cổ chai”.

##2. Xung đột tên 

- Không thể có 2 máy có cùng tên trong tập tin hosts.txt (do chưa có cơ chế phân cấp và ủy quyền quản lý tên).

##3. Không đảm bảo sự toàn vẹn 

- Việc duy trì 1 tập tin trên mạng lớn rất khó khăn. Ví dụ như khi tập tin hosts.txt vừa cập nhật chưa kịp chuyển đến 1 máy chủ khác ở xa thì đã có sự thay đổi địa chỉ trên mạng rồi.

- Việc sử dụng tập tin hosts.txt không phù hợp cho 1 mạng lớn vì thiếu tính phân tán và mở rộng. Do đó dịch vụ DNS ra đời, và người thiết kế cấu trúc của dịch vụ DNS là Paul Mockapetris – USC’s Information Sciences Institute và các kiến nghị RFC của DNS là RFC 882 và 883, sau đó là RFC 1034 và 1035 cùng với 1 số RFC bổ xung như bảo mật trên hệ thống DNS, cập nhật các record của DNS.

- Hiện nay trên các máy chủ vẫn sử dụng tập tin hosts.txt để phân giải tên máy tính thành địa chỉ IP (trong Windows tập tin này nằm trong đường dẫn WINDOWS\system32\drivers\tec)

- Dịch vụ DNS hoạt động theo mô hình Client-Server :
<ul>
<li>Server : có chức năng là phân giải tên--->IP và ngược lại IP--->tên, được gọi là Name Server, lưu trữ cơ sở dữ liệu của DNS.</li>
<li>Client : truy vấn phân giải tên đến DNS server được gọi là Resolver, chứa các hàm thư viện dùng để tạo các truy vấn (query) đến Name Server.</li>
</ul>

- DNS được thi hành như 1 giao thức của tầng Application trong mô hình mạng TCP/IP

- DNS là 1 cơ sở dữ liệu dạng phân tán được tổ chức theo mô hình cây (hierarchical) :

<img src="http://i.imgur.com/dQxLlM3.png">

- Một hostname trong domain là sự kết hợp giữa những từ phân cách nhau bởi dấu chấm (.)

**Ví dụ : tên máy là server1 gọi là hostname. Tên đầy đủ trong domain theo mô hình trên thì là server1.sales.south.nwtraders.com gọi là FQDN (Fully Qualified Domain Name).

- Cơ sở dữ liệu (CSDL) của DNS là 1 cây đảo ngược, mỗi node trên cây là gốc của 1 cây con. Mỗi cây con là 1 phân vùng con trong toàn bộ CSDL của DNS gọi là 1 domain. Mỗi domain có thể phân chia thành các miền con nhỏ hơn gọi là subdomain.

- Các top-level domain :

<img src="http://vnbeggar.info/vip/sangnt/Pictures/table%20DNS%20top%20level%20domain.png">


- Vì sự quá tải của các domain name đã tồn tại, do đó đã làm phát sinh những top-level domain mới. Bảng sau đây liệt kê các top-level domain mới

<img src="http://vnbeggar.info/vip/sangnt/Pictures/table%20DNS%20top%20level%20domain%202.png">

- Bên cạnh đó, mỗi quốc gia cũng có top-level domain. Bảng sau sẽ liệt kê 1 số top-level domain của 1 số quốc gia trên thế giới.
