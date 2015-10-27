#1. Cài đặt máy chủ Slave DNS

- Việc cấu hình máy chủ Slave DNS sẽ giống như máy chủ Master DNS, chỉ một điểm khác biệt trên máy chủ Master DNS phải thêm tham số allow-transfer vào trong tập tin /etc/bind/named.conf.local

<img src="http://prntscr.com/8vvjk3">

- Trong đó: 10.0.0.2 sẽ là IP của máy chủ Slave DNS.
Máy chủ Slave DNS cũng cần phải thêm tham số masters vào trong tập tin /etc/bind/named.conf.local

   <img src="http://prntscr.com/8vvjqr">

- Trong đó: 10.0.0.1 sẽ là IP của máy chủ Master DNS.

##Cài đặt máy chủ Caching DNS

- Cấu hình máy chủ DNS cache rất đơn giản chỉ là việc địa chỉ máy chủ DNS của nhà cung cấp dịch vụ(ISP's DNS servers). Để tìm máy chủ DNS của các nhà cung cấp dịch vụ chúng ta chỉ cần đơn giản sử dụng lệch cat để xem nội dung tập tin /etc/resolv.conf.

  **cat /etc/resolv.conf**

- Chúng ta sẽ thu được ip của nhà cung cấp dịch vụ DNS

<img src="http://prntscr.com/8vvjwl">

- Ngoài dử dụng máy chủ DNS của nhà cung cấp chúng ta có thể sử dung máy chủ DNS của google qua hai IP là: 8.8.8.8,8.8.4.4
Giờ để cấu hình máy chủ DNS cache cần gỡ bỏ các ghi chú và chỉnh sử nội dung tập tin /etc/bind/named.conf.options như sau:


<img src="http://prntscr.com/8vvk2k">


- Trong đó 10.195.1.2, 10.195.1.4 là hai máy chủ DNS của nhà cung cấp.

- Sau đó khởi động lại ứng dung bind

**sudo /etc/init.d/bind9 restart**

- Để kiểm tra tính hoạt động của máy chủ DNS cache chúng ta dùng lệnh dig trong gói dnsutils hai lần và so sánh kết quả
Kết quả lần thứ nhất

                        ; <<>> DiG 9.7.0-P1 <<>> google.com
                        ;; global options: +cmd
                        ;; Got answer:
                        ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 31357
                        ;; flags: qr rd ra; QUERY: 1, ANSWER: 6, AUTHORITY: 13, ADDITIONAL: 5
                        
                        ;; QUESTION SECTION:
                        ;google.com. IN A
                        
                        ;; ANSWER SECTION:
                        google.com. 58 IN A 74.125.71.103
                        google.com. 58 IN A 74.125.71.104
                        google.com. 58 IN A 74.125.71.105
                        google.com. 58 IN A 74.125.71.106
                        google.com. 58 IN A 74.125.71.147
                        google.com. 58 IN A 74.125.71.99
                        
                        ;; AUTHORITY SECTION:
                        . 33847 IN NS b.root-servers.net.
                        . 33847 IN NS j.root-servers.net.
                        . 33847 IN NS c.root-servers.net.
                        . 33847 IN NS f.root-servers.net.
                        . 33847 IN NS i.root-servers.net.
                        . 33847 IN NS g.root-servers.net.
                        . 33847 IN NS e.root-servers.net.
                        . 33847 IN NS a.root-servers.net.
                        . 33847 IN NS k.root-servers.net.
                        . 33847 IN NS d.root-servers.net.
                        . 33847 IN NS h.root-servers.net.
                        . 33847 IN NS m.root-servers.net.
                        . 33847 IN NS l.root-servers.net.
                        
                        ;; ADDITIONAL SECTION:
                        b.root-servers.net. 33101 IN A 192.228.79.201
                        c.root-servers.net. 34633 IN A 192.33.4.12
                        d.root-servers.net. 23193 IN A 128.8.10.90
                        h.root-servers.net. 28134 IN A 128.63.2.53
                        m.root-servers.net. 39852 IN A 202.12.27.33
                        
                        ;; Query time: 37 msec
                        ;; SERVER: 127.0.0.1#53(127.0.0.1)
                        ;; WHEN: Sat Oct 1 11:41:50 2011
                        ;; MSG SIZE rcvd: 415

Kết quả lần thứ 2
dig google.com

                        ; <<>> DiG 9.7.0-P1 <<>> google.com
                        ;; global options: +cmd
                        ;; Got answer:
                        ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 29715
                        ;; flags: qr rd ra; QUERY: 1, ANSWER: 6, AUTHORITY: 4, ADDITIONAL: 4
                        
                        ;; QUESTION SECTION:
                        ;google.com. IN A
                        
                        ;; ANSWER SECTION:
                        google.com. 32 IN A 74.125.71.99
                        google.com. 32 IN A 74.125.71.103
                        google.com. 32 IN A 74.125.71.104
                        google.com. 32 IN A 74.125.71.105
                        google.com. 32 IN A 74.125.71.106
                        google.com. 32 IN A 74.125.71.147
                        
                        ;; AUTHORITY SECTION:
                        google.com. 21483 IN NS ns2.google.com.
                        google.com. 21483 IN NS ns1.google.com.
                        google.com. 21483 IN NS ns4.google.com.
                        google.com. 21483 IN NS ns3.google.com.
                        
                        ;; ADDITIONAL SECTION:
                        ns1.google.com. 78766 IN A 216.239.32.10
                        ns2.google.com. 76633 IN A 216.239.34.10
                        ns3.google.com. 77271 IN A 216.239.36.10
                        ns4.google.com. 71682 IN A 216.239.38.10
                        
                        ;; Query time: 0 msec
                        ;; SERVER: 127.0.0.1#53(127.0.0.1)
                        ;; WHEN: Sat Oct 1 11:42:16 2011
                        ;; MSG SIZE rcvd: 260

So sánh 2 kết quả này thời gian truy vấn(Query time) của lần thứ nhất là 37 msec còn lần thứ 2 do đo đã được ghi nhớ( cache) nên thời gian truy vần giảm chỉ còn 0 msec
