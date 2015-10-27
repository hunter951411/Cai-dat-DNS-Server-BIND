#Quá trình phân giải tên host thành IP


##Bước 1:

- Đầu tiên khi Resolver(web browser) gõ http://hunter.com và enter

- Trình duyệt web sẽ gọi hàm windows sockets hoặc gethostbyname() hay getaddinfor() để yêu cầu phân giải svuit.com thành IP.

- Client sẽ tìm trong cache DNS của client: Tên svuit.com có thể được lưu lại từ những truy cập trước đó hay trong file host.txt của client xem cái máy có tên hunter.com có IP là bao nhiêu? . Nếu có nó sẽ sử dụng IP này để truy cập

- Ví dụ: Bạn có thể vào file host.txt của bạn và thêm dòng . Khi đó bạn vào trình duyệt web của bạn gõ svuit.com. Nó sẽ ra trang google.com.

  **74.125.132.101 hunter.com.**

##Bước 2:

- Nếu trong cache của client không tìm thấy cái máy nào có tên là hunter.com nó sẽ gửi recursive query tới DNS server được cấu hình trong client. 

- DNS server sẽ thực hiện tìm kiếm tên máy hunter.com trong cache hay các zone của nó nếu có thì nó sẽ trả lời về cho PC của bạn. 

- Nếu DNS local không tìm thấy máy nào có tên hunter.com thì nó sẽ gửi iterative request cho Root DNS gần nó nhất.

##Bước 3:

- Lúc này Root DNS nhìn vào tên miền svuit.com và nó sẽ trả lời lại cho DNS server local rằng: “muốn biết IP của hunter.com thì đi hỏi DNS server(Top-level domain) quản lý domain .com” vì

- Root DNS không có thẩm quyền (authoritative) với domain hunter.com

- Do đó nó sẽ trả lời lại IP của Top-Level-DNS có authoritative với domain “.com”


**Từ ví dụ trên các bạn có thể thấy tầm quan trọng và cách hoạt động của Root DNS trên internet**

##Bước 4:

- DNS server của client sẽ gửi iterative query đến TOP-LEVEL-Domain “.com”

##Bước 5:

- Top-Level-DNS “.com” chứa IP những Server có thẩm quyền cho các domain cấp 2 (second level domain names của miền “.com”)

- Nó ko có authoritative cho domain hunter.com

- Do đó nó sẽ trả lời IP server của domain hunter.com có thẩm quyền

##Bước 6

- DNS server lại gửi 1 iterative query cho hunter.com

##Bước 7:

- hunter.com sẽ trả lời IP tương ứng với tên miền đầy đủ FQDN(Full qualified domain name) của hunter.com

##Bước 8

- DNS server gửi trả lời về cho client IP của hunter.com là gì.

- Client sử dụng IP này để mở 1 phiên kết nối TCP/IP đến server chứa IP trang web hunter.com


#2. Phân giải IP thành tên host

- Để thực hiện phân giải ngược nghĩa là từ IP thành tên host. Người ta đã thêm in-addr.arpa. trong namspace.

- Mỗi nút in-addr.arpa có những IP dạng ngược

#3. Các loại truy vấn

##Có 2 loại query:

- recursive query: Truy vấn đệ quy

+ Nếu DNS server nhận được 1 recursive query nó sẽ phải có trách nhiêm tìm cho ra IP mà Resolver query hoặc trả lời cho Resolver là svuit.com không tồn tại. => Đặt gánh nặng cho DNS server.
+ Thông thường query từ client đến DNS server là recursive query


- iteractive query: Truy vấn tương tác

+ Nếu DNS server local không phân giải được domain svuit.com thì nó sẽ thực hiện 1 interactive query đến 1 DNS server khác gần nó nhất(Root DNS).

+ DNS server sẽ trả lời về 1 tên DNS Server khác tốt nhất để phân giải tên svuit.com. Chẳng hạn nó sẽ trả lời về top-level-DNS quản lý domain “.com”

+ Thông thường query giữa DNS server với DNS server là interactive query.


#4. Caching-only Server

- DNS Server và Client sẽ lưu lại những truy vấn (caching)

+ Để khi được truy vấn lần sau nó sẽ tìm trong cache trước, nếu cache có nó sẽ trả lời ngay lập tức mà không cần truy vấn nữa.

+ Điều này giúp cho mạng hoạt động nhanh hơn (tăng performing).

- Caching-only có khả năng trả lời các câu truy vấn đến Client. Nhưng không chứa Zone nào và cũng không có quyền quản lý bất kì domain nào. Nó sử dụng bộ cache của mình để lưu các truy vấn của DNS của Client. Thông tin sẽ được lưu trong cache để trả lời các truy vấn đến Client.

#5. Time to live

- Những dữ liệu được cache lại trong DNS server hoặc Client sẽ không tồn tại vĩnh viễn vì có thể thông tin của dữ liệu đó có thể thay đổi…

- TTL là thời gian mà các DNS Server hoặc Client được phép cache thông tin đã truy vấn được, sau thời gian đó các DNS Server hoặc Client sẽ phải hủy tất cả các cache đó và đi lấy thông tin mới bằng cách truy vấn lại. 

- Giá trị TTL này có thể được thay đổi bởi người quản trị trong việc khai báo TTL cho dữ liệu đó.

