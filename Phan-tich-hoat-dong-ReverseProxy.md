# Phân tích hoạt động của Reverse Proxy có lưu cache


### Menu

1. Giới thiệu ](#1)

- [1.1 Mô hình ](#1.1)
- [1.2 Lợi ích khi sử dụng Reverse Proxy ](#1.2)
- [1.3 Mô tả hoạt động ](#1.3)

[2. Phân tích hoạt động ](#2)

- [Request của USER thứ 1](#2.1)
- [Request của USER thứ 2](#2.2)

[3. Kết luận ](#3)

<a name="1"></a>
## 1. Giới thiệu

Apache và NGINX là 2 hệ thống Web server phổ biến và được sử dụng rộng dãi trong nhiều hệ thống lớn với ưu điểm chung đều là các phần mềm OpenSource. Apache nổi tiếng làm việc hiệu quả với những xử lý động như PHP,... Về NGINX, điểm mạnh của nó là xử lý rất nhanh các web tĩnh. Với những ưu điểm đó, người ta đã kết hợp NGINX và Apache lại với nhau để bổ trợ cho nhau giúp hệ thống Webserver thêm phần hoàn thiện và đạt hiệu quả cao.

<a name="1.1"></a>
### 1.1 Mô hình

<img src="http://i1363.photobucket.com/albums/r714/HoangLove9z/rp_zpsm4wwklir.png" />

 | HAProxy 1 | HAProxy 2 | USER |
--- | --- | --- | --- |
OS | Ubuntu 16.04 | Ubuntu 16.04 | Windows 7 |
NIC | eth0 | eth0 | LAN |
IP | 192.168.100.194 | 192.168.100.195 | 192.168.100.22 |
Package| NGINX | APACHE | Chrome + WireShark |
<a name="1.2"></a>
### 1.2 Lợi ích khi sử dụng Reverse Proxy

- Giảm tải cho Webserver
- Xử lý nhanh các request tĩnh
- Giấu được mô hình, tăng tính bảo mật

<a name="1.3"></a>
### 1.3 Mô tả hoạt động

- Bước 1: Request đầu tiên của USER đến Reverse Proxy
- Bước 2: Request của USER được Reverse Proxy chuyển đến Webserver
- Bước 3: Webserver trả về reponse cho Reverse Proxy, cache được lưu lại ở Reverse Proxy
- Bước 4: Reverse Proxy trả reponse cho USER
- Bước 5: Request của USER thứ 2 đến Reverse Proxy
- Bước 6: Request của USER thứ được Reverse Proxy xử lý, nếu request yêu cầu có sẵn trong cache thì trả lại cho USER2, nếu chưa có thì quay lại bước 1

<img src="http://i1363.photobucket.com/albums/r714/HoangLove9z/luong-rp_zpsvc4xzsvy.png" />
<a name="2"></a>
## 2. Phân tích hoạt động

<a name="2.1"></a>
### Request của USER thứ 1:

<img src="http://image.prntscr.com/image/ba8a4a797f7a45c599aaffdeda525720.png" />


- `No.12`, Người dùng có IP: 192.168.100.20 truy cập HTTP đến `Reverse Proxy` thông qua IP là 192.168.100.194
- `No.17`, `Reverse Proxy` có địa chỉ 192.168.100.194 sẽ gửi bản tin truy cập HTTP của Người dùng đến `Webserver` có IP là 192.168.100.195
- `No.19`,  `Webserver` (192.168.100.195) xử lý request rồi gửi lại response cho `Reverse Proxy` có địa chỉ là 192.168.100.194
- `No.25`, `Reverse Proxy` gửi trả response từ 192.168.100.195 đến người dùng có địa chỉ là 192.168.100.20

<a name="2.2"></a>
### Request của USER thứ 2:

<img src="http://image.prntscr.com/image/0069291ab27f43719d643d6c6d694e3d.png" />

- `No.15`, Người dùng có IP: 192.168.100.250 truy cập HTTP đến `Reverse Proxy` thông qua IP là 192.168.100.194
- `No.18`,  `Reverse Proxy` (192.168.100.194) xử lý request, và thấy request đã có ở trong cache vì thế nó sẽ trả reponse từ `Reverse Proxy` (192.168.100.194) về luôn cho Người dùng có địa chỉ là 192.168.100.250 mà **KHÔNG** phải gửi yêu cầu đến `Webserver`

<a name="3"></a>
### 3. Kết luận

Qua bài phân tích trên, chúng ta thấy được luồng hoạt động của mô hình này. Các request từ người dùng chỉ giao tiếp với Reverse Proxy không hề biết bên trong có máy chủ Web, đồng thời Reverse Proxy cũng lưu trữ lại cache việc này làm giảm tải cho Webserver không phải làm việc quá nhiều.