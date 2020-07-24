# Training_Socket

## I. Các khái niệm

### 1. Socket là gì ?

Socket là một điểm cuối (end-point) của liên kết truyền thông hai chiều (two-way communication) giữa hai chương trình chạy trên mạng. Một end-point là sự kết hợp giữa địa chỉ IP và số cổng.

Các lớp Socket được sử dụng để biểu diễn kết nối giữa client và server, được ràng buộc với một cổng port (thể hiện là một con số cụ thể) để các tầng TCP (TCP Layer) có thể định danh ứng dụng mà dữ liệu sẽ được gửi tới. 

Dữ liệu sẽ được truyền dưới dạng packet - các gói tin.

Dữ liệu ghi vào luồng đầu ra trên Socket của client sẽ nhận được trên luồng đầu vào của Socket tại Server. Và ngược lại dữ liệu ghi vào luồng đầu ra trên Socket của Server sẽ nhận được trên luồng đầu vào của Socket tại Client.

<img src="https://github.com/VNAPNIC/Training-socket/blob/master/766871.png"/>

### 2. Cách tạo liên kết

- **Tại phía server**: thông thường, một chương trình server chạy trên một máy tính cụ thể và có một cái cổng (**socket**) kết nối với một **port** cụ thể. Các chương trình đợi, lắng nghe tại socket để các client thực hiện yêu cầu kết nối. 

- **Tại phía client**: clien biết tên của máy chủ mà server đang chạy (thường đó là địa chỉ IP, tên miền hoặc localhost, ...) và số cổng server đang lắng nghe. Các thông tin này cần cho việc kết nối. Client cũng phải tự định danh chính nó với server để gắn một cổng địa phương cái sẽ được sử dụng trong suốt quá trình kết nối này, thông thường nó được gán bởi hệ điều hành. 

Hình minh họa

<img src="https://github.com/VNAPNIC/Training-socket/blob/master/766768.png" width="700"/>

- Nếu kết nối được chấp nhận, máy chủ có một **socket** mới bị ràng buộc vào cùng "cổng địa phương" và thông tin của nó chính là địa chỉ và cổng của client. Nó tạo ra một socket mới đẻ giao tiếp với Client vừa được chấp nhận kết nối và tiếp tục lắng nghe tại ổ cắm ban đầu cho các yêu cầu kết nối khác.

Phía Client, nếu kết nối được chấp nhận, một ổ cắm được tạo thành công và Client có thể sử dụng ổ cắm để giao tiếp với chương trình chủ.
<img src="https://github.com/VNAPNIC/Training-socket/blob/master/766756.png"/>

### 3. Phân loại

### a. Stream socket - socket hướng kết nối

Dựa trên giao thức TCP, thiết lập giao tiếp 2 chiều theo mô hình Client - Server. Việc truyền dữ liệu chỉ thực hiện giữa 2 quá trình đã thiết lập kết nối. Cơ chế phức tạp để quản lý luồng, chống tắc nghẽn.

<img src="https://github.com/VNAPNIC/Training-socket/blob/master/tcp.jpg" />

Đặc điểm:

- Có một kênh giao tiếp ảo giữa hai bên giao tiếp (handshake)

- Đảm bảo dữ liệu được truyền đến nơi 1 cách tin cậy, đúng tuần tự

- Một trong hai tiến trình phải đợi bên kia yêu cầu kết nối - thiết lập kết nối, sau đó mới có thể trao đổi dữ liệu giữa hai bên

- Mỗi thông điệp phải có xác nhận trả về, có lỗi sẽ được thông báo

Sử dụng: cho các ứng dụng cần độ tin cậy, chính xác cao

- HTTP, HTTPS

- SSH

- FTP: truyền file

- Email: SMTP, IMAP/POP


### b. Datagram Socket - socket không hướng kết nối

Dựa trên giao thức UDP, truyền dữ liệu không yêu cầu sự thiết lập kết nối giữa 2 process. 

<img src="https://github.com/VNAPNIC/Training-socket/blob/master/udp.jpg"/>

Đặc điểm:

- Không có kênh giao tiếp ảo giữa hai bên giao tiếp (không có handshake)

- Do không yêu cầu thiết lập kết nối, cơ chế đơn giản nên tốc độ nhanh

- Không đảm bảo dữ liệu có thế đến bên nhận, dữ liệu đến có thể không tuần tự, bị mất mát

- 

Sử dụng: cho các ứng dụng cần tốc độ nhanh, độ chính xác, tin cậy không cần cao

- Online game, chat

- Video trực tuyến

- DNS, VoIP (truyển giọng nói)

### 4. Websocket là gì ?

### a. Tại sao cần Websocket ?

Hiện tại có nhiều phương thức để client - server có thể giao tiếp được với nhau, ví dụ chúng là:

- Ajax polling: client gửi request liên tục sau 1 khoảng thời gian lên cho server. Điều này bắt server trả về dữ liệu dù có dữ liệu mới hay không => Lãng phí băng thông, thời gian

- Ajax long polling: đầu tiên client vẫn request lên cho server. Nhưng server đợi có dữ liệu mới hoặc hết time out mới trả về dữ liệu cho client. Sau đó ngay lập tức lặp lại quy trình trên. ==> Tiết kiệm băng thông, tài nguyên tiêu thụ.

- HTML5 Server Sent Event (SSE): chỉ client gửi request lên server. Kể từ lúc này  server sẽ gửi response trả về cho client khi có dữ liệu mới. Client không cần request lại thêm lần nào. 

 Ngoài ra còn có Comet.

 Đặc điểm chung của các phương thức trên là đều sử dụng HTTP. Nhược điểm của nó là chứa nhiều dữ liệu không cần thiết trong header. Ví dụ một header của HTTP khoảng 871 byte, thì websocket chỉ có 2 byte.

 Ajax polling, long polling không hỗ trợ đồng song song giữa client và server:  - giúp cả client và server đều có thể gửi và nhận request cùng một thời điểm. Chỉ khi có request từ client, server mới hoạt động và trả kết quả về, còn lại thì server ko làm gì cả


### b. Giới thiệu

Websocket là công nghệ hỗ trợ giao tiếp 2 chiều giữa client và server:

- Không sử dụng HTTP, mà sử dụng 1 TCP socket để tạo 1 kết nối, truyền dữ liệu qua chiệu quả và ít tốm kém hơn HTTP

- Là một kĩ thuật Reverse Ajax. Cho phép các kênh giao tiếp song song hai chiều và hiện đã hỗ trợ nhiều trình duyệt. 

- Hỗ trợ chuẩn giao thức mới ws:// và wss:// (tương tự http:// và https://)

- Chuẩn giao tiếp là String, hiện đã có buffered arrays (protobuf) và blobs (Binary request). Chưa hỗ trợ JSON

- Như đã nói ở phần 1 thì phần header sẽ nhỏ hơn, có thể tới 500 - 1000 lần.

- Giúp giảm độ trễ network lên đến 3 lần

### c. Hoạt động

<img src="https://github.com/VNAPNIC/Training-socket/blob/master/websocket-la-gi.png">

1. Client gửi yêu cầu để hiển thị 1 trang web từ server 

2. Trang web mà client yêu cầu mở một kết nối tới server

3. Server đồng ý kết nối. Bây giờ cả client với server đều có thể gửi và nhận dữ liệu vs nhau khi có dữ liệu mới

<img src="https://github.com/VNAPNIC/Training-socket/blob/master/ky-thuat-lap-trinh-web-regular-http-13122015-socket.png">

### d. Ưu/nhược điểm

Ưu điểm

WebSockets cung cấp khả năng giao tiếp hai chiều mạnh mẽ, có độ trễ thấp và dễ xử lý lỗi. Không cần phải có nhiều kết nối như phương pháp Comet long-polling và cũng không có những nhược điểm như Comet streaming.
API cũng rất dễ sử dụng trực tiếp mà không cần bất kỳ các tầng bổ sung nào, so với Comet, thường đòi hỏi một thư viện tốt để xử lý kết nối lại, thời gian chờ timeout, các Ajax request (yêu cầu Ajax), các tin báo nhận và các dạng truyền tải tùy chọn khác nhau (Ajax long-polling và jsonp polling).

Nhược điểm

Những nhược điểm của WebSockets gồm có:

Nó là một đặc tả mới của HTML5, nên nó vẫn chưa được tất cả các trình duyệt hỗ trợ.
Không có phạm vi yêu cầu nào. Do WebSocket là một TCP socket chứ không phải là HTTP request, nên không dễ sử dụng các dịch vụ có phạm vi-yêu cầu, như SessionInViewFilter của Hibernate. Hibernate là một framework kinh điển cung cấp một bộ lọc xung quanh một HTTP request. Khi bắt đầu một request, nó sẽ thiết lập một contest (chứa các transaction và liên kết JDBC) được ràng buộc với luồng request. Khi request đó kết thúc, bộ lọc hủy bỏ contest này.

----------------------- n

tcp: data packet - byte
chứa thông tin data, nhận dữ liệu hay chưa

http: sử dụng http message, cách đọc và xử lý data khi nó đến











