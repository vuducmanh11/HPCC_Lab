#  Networking

## 1. OSI 

- Physical Layer (level 1): chuyển dữ liệu bit thành tín hiệu và truyền
- Data Link Layer (level 2): truyền dữ liệu trên các liên kết vật lý giữa các nút mạng liên kết trực tiếp với nhau
- Network Layer (level 3): vận chuyển gói tin từ một mạng đến một mạng khác (giao thức IP)
- Transport Layer (level 4): truyền dữ liệu theo trình tự từ nguồn tới đích (TCP, UDP)
- Session Layer (level 5): khởi tạo, quản lý và kết thúc các kết nối 
- Presentation Layer (level 6): khởi tạo nội dung giữa các thực thể tầng ứng dụng (mã hóa, nén, chuyển đổi)
- Application Layer (level 7): cung cấp ứng dụng cho người dùng cuối

## 2. TCP/IP

- Chức năng 3 tầng trên cùng của mô hình OSI gộp thành Application, được sử dụng hầu hết trong các hệ thống mạng Internet

## 3. DNS 

#### 3.1. Domain

- định danh trên tầng ứng dụng của mỗi nút mạng

#### 3.2. DNS (Domain name system)

- Giới thiệu
  - Chứa không gian thông tin về tên miền
  - Gồm các máy chủ quản lý tên miền và cung cấp dịch vụ phân giải tê miền
  - Người dùng sử dụng tên miền để truy cập mạng còn các thiết bị mạng sử dụng địa chỉ IP khi trao đổi dữ liệu
- Hệ thống máy chủ DNS
  - Máy chủ tên miền cấp 1: quản lý tên miền cấp 1(Top level domain-TLD)
  - Máy chủ được ủy quyền (authoritative DNS severs): quản lý tên miền cấp dưới
  - Máy chủ của các tổ chức, ISP: không nằm trong phân cấp DNS
  - Máy chủ cục bộ: dành cho mạng nội bộ của cơ quan tổ chức, không nằm trong phân cấp DNS
- Phân giải tên miền
  - Tự phân giải
    - file HOST: /etc/hosts
    - Cache
  - Dịch vụ phân giải tên miền DNS: client/server
    - UDP, Port 53
    - Phân giải đệ quy: người dùng bắt đầu hỏi địa chỉ của tên miền cần đến Local server, không thấy máy chủ cục bộ sẽ hỏi đến root server, root server hỏi TLD server, TLD server hỏi Authoritative DNS server
    - Phân giải tương tác:  người dùng bắt đầu hỏi địa chỉ của tên miền cần đến Local server, không thấy máy chủ cục bộ sẽ lần lượt hỏi root server, TLD server, Authoritative DNS server

## 4. DHCP 

#### 4.1. Khái niệm và thành phần

**Dynamic host configuration protocol**: tự động cấp phát địa chỉ IP động cho khách khi tham gia mạng

- DHCP server: quản lý việc cấu hình và cấp phát địa chỉ IP cho Client
- DHCP client: máy nhận thông tin cấu hình địa chỉ IP từ DHCP server
- Scope: dải địa chỉ IP được dùng để cấp phát cho Client
- Exclusing Scope: địa chỉ trong Scope không được cấp phát động cho Client

#### 4.2. Các thông điệp

- DHCP Discover: Một DHCP Client khi mới tham gia vào hệ thống mạng, nó sẽ yêu cầu thông tin địa chỉ IP từ DHCP Server bằng cách broadcast một gói DHCP Discover. Địa chỉ IP nguồn trong gói là 0.0.0.0 bởi vì client chưa có địa chỉ IP 
- DHCP Offer: Khi DHCP server nhận được gói DHCP Discover từ client, nó sẽ gửi lại một gói DHCP Offer chứa địa chỉ IP, Subnet Mask, Gateway,... Có thể nhiều DHCP server gửi lại với gói DHCP Offer nhưng Client sẽ chấp nhận gói DHCP Offer đầu tiên nó nhận được 
- DHCP Request: Khi DHCP client nhận được một gói DHCP Offer, nó đáp lại bằng việc broadcast gói DHCP Request để xác nhận hoặc để kiểm tra lại các thông tin mà DHCP server vừa gửi 
- DHCP Acknowledge: Server kiểm tra và xác nhận lại sự chấp nhận trên bằng gói tin DHCP Acknowledge, Client có thể tham gia trên mạng TCP/IP và hoàn thành hệ thống khởi động 
- DHCP Nak: Nếu một địa chỉ IP đã hết hạn hoặc đã được đặt một máy tính khác, DHCP Server gửi gói DHCP Nak cho Client, như vậy Client phải bắt đầu tiến trình thuê bao lại 
- DHCP Decline: Nếu DHCP Client nhận được bản tin trả về không đủ thông tin hoặc hết hạn, nó sẽ gửi gói DHCP Decline đến các Server và Client phải bắt đầu tiến trình thuê bao lại 
- DHCP Realease: Client gửi bản tin này đến server để ngừng thuê IP. Khi nhận được bản tin này, server sẽ thu hồi lại IP đã cấp cho Client, khi đó client sẽ không được cấu hình IP

#### 4.2. Cơ chế hoạt động

- Client gửi một bản tin Discover - có chứa thông tin client (MAC, Tên máy tính,...) với dạng Broadcast để "truy tìm" một DHCP Server để xin cấp phát địa chỉ IP 
- Khi DHCP Server nhận được một bản tin Discover sẽ gửi lại cho Client một bản tin Offer - chứa những thông tin IP,thời hạn cho thuê, GW, DNS Server,... dưới dạng Unicast 
- Client gửi lại một bản tin Request cho Server, để xác nhận lại các thông tin 
- Server trả về bản tin ACK để hoàn tất quá trình 

## 5. Proxy

####5.1. Tổng quát 

- Máy chủ proxy là một máy tính chuyên dụng hoặc phần mềm hệ thống chạy trên một máy tính hoạt động trung gian giữa thiết bị điểm cuối như một máy tính và máy chủ khác mà từ đó người dùng hay client gửi request dịch vụ. Máy chủ proxy có thể cùng tồn tại trong máy tính như một tường lửa hoặc nó nằm trên một máy chủ riêng chuyển tiếp các yêu cầu thông qua tường lửa.
- Một lợi ích của server proxy là bộ nhớ cache có thể phục vụ tất cả người dùng. Nếu một hoặc nhiều trang web thường được truy cập, nó giống như nằm trong bộ nhớ cache proxy, việc này sẽ cải thiện thời gian người dùng phải chờ. Một proxy cũng có thể ghi lại tương tác của nó, điều này giúp hữu ích trong việc khắc phục sự cố

#### 5.2. Forward và reverse proxy servers

- **Forward proxies**
  - gửi các request của người dùng tới web server. Những người dùng truy cập proxy này bằng cách truy cập trực tiếp địa chỉ web proxy. Forward proxy cho phép vượt tường lửa tăng sự riêng tư và bảo mật cho người dùng nhưng đôi khi nó giúp tải xuống các tài liệu bất hợp pháp như tài liệu bản quyền hay nội dung 18+.
- **Reverse proxies** 
  - xử lý minh bạch tất cả các yêu cầu về tài nguyên trên server đích mà không yêu cầu bất kỳ hành động nào của người yêu cầu
  - thường được sử dụng
    - cho phép truy cập gián tiếp khi website từ chối kết nối trực tiếp như 1 sự đảm bảo an toàn
    - cho phép cân bằng tải giữa các server
    - truyền nội dung nội bộ tới người dùng Internet
    - vô hiệu hóa truy cập trang web, như khi ISP hay chính phủ yêu cầu chặn website
  - trang web có thể bị chặn vì nhiều lý do, Reverse proxy có thể được sử dụng để ngăn cản truy cập vào nội dung vô đạo, bất hợp pháp hoặc nội dung có bản quyền. Đôi khi việc này ngăn cản người dùng truy cập trang web tin tức nơi thông tin bị rò rỉ, trang tiết lộ thông tin về hành động của chính phủ gây mất tự do ngôn luận

## 6. Firewall

#### 6.1. Khái niệm

- **Firewall** là phần mềm hoặc chương trình cơ sở thực thi bộ quy tắc về các gói dữ liệu sẽ được cho phép ra vào mang.
- Tường lửa (firewall) được tích hợp vào một loạt các thiết bị được kết nối mạng, kiểm soát truy cập và giảm thiểu rủi ro các gói tin độc hại được thông qua mạng công cộng ảnh hưởng tới an toàn của mạng người dùng.
- Tường lửa có thể mua dạng ứng dụng độc lập
- Tường lửa là một hàng rào ảo được đặt bao quanh mạng để hạn chế thiệt hại từ một cuộc tấn công mạng bên nguoaif hoặc nội bộ

#### 6.2. Phân loại

- Tường lửa được chia làm hai loại chính 
  - host-based
    - cài đặt riêng trên server, giám sát mọi tín hiệu đến và đi
  - network-based
    - có thể được xây dựng tích hợp cơ sở hạ tầng cloud hoặc có thể là một dịch vụ tường lửa ảo

#### 6.3. Iptables 

- **Iptables** được cấu hình và hoạt động trên console, tăng tính bảo mật cho hệ thống linux	
  - lọc package dựa vào MAC và một số cờ hiệu TCP Header
  - cung cấp tùy chọn ghi nhận sự kiện hệ thống
  - cung cấp kỹ thuật NAT
  - có khả năng chặn một số cơ chế tấn công DoS
- show list rule: `sudo iptables -L`
  - TARGET: hành động được áp dụng cho quy tắc
    - Accept: gói dữ liệu được phép chuyển tiếp để xử lý 
    - Drop: chặn gói tin và loại bỏ
    - Reject: chặn gói tin, loại bỏ và gửi thông báo lỗi đến người dùng
  - PROT (protocol): quy định giao thức được áp dụng để thực hiện quy tắc gồm 
    - all
    - TCP
    - UDP
  - SOURCE, DESTINATION: địa chỉ của lượt truy cập được phép áp dụng quy tắc

## 7. Flat network

#### 7.1. Khái niệm

- Flat là một thiết kế mạng tiếp cận với mục tiêu giảm chi phí dễ bảo trì và quản trị. Flat network được thiết kế để giảm số lượng router và switch trên mạng máy tính bằng cách kết nối các thiết bị tới cùng một switch thay vì các switch riêng biệt.

#### 7.2. Sử dụng

- Flat network thường được sử dụng trong hộ gia đình hoặc doanh nghiệp nhỏ nới yêu cầu về mạng thấp, không yêu cầu bảo mật chuyên sâu hoặc sự tách biệt bởi mạng được cung cấp cho nhiều thiết bị truy cập. Trong trường hợp này việc cấu hình mạng phức tạp với nhiều switch là không cần thiết.
- Flat network cũng dễ để quản lý bảo trì vì ít switch và router được sử dụng, việc này cũng làm giảm chi phí cấu hình mạng.

####7.2.3. Hạn chế

- Kém an toàn: vì lưu lượng chỉ di chuyển qua một switch, nó không thể phân đoạn mạng và ngăn cản người dùng truy cập một số phần mạng. Các hacker có thể dễ dàng chặn và ăn cắp dữ liệu trên mạng.
- Không dự phòng: vì chỉ có 1 switch hay là ít thiết bị, việc hỏng hóc sẽ gây ra không có đường dẫn thay thế, mạng sẽ không thể truy cập dẫn đến máy tính kết nối thất bại.
- Khả năng mở rộng và tốc độ: kết nối tất cả thiết bị tới một switch trung tâm trực tiếp hoặc qua các hub làm tăng khả năng đụng độ, giảm tốc độ do dữ liệu cần được xử lý và chuyển tiếp ở switch trung tâm. Khả năng mở rộng tồi và tăng khả năng lỗi mạng nếu sử dụng nhiều hub và không đủ switch để điều khiển luồng mạng

## 8. Overlay

#### 8.1. Giới thiệu

- Overlay Networking là một phương thức sử dụng phần mềm để tạo các lớp mạng trừu tượng có thể sử dụng để chạy nhiều lớp mạng ảo riêng biệt rời rạc trên mạng vật lý, thường để cung cấp ứng dụng hoặc tăng tính bảo mật

## 9. VLAN, VxLAN, GRE

#### 9.1. VLAN

- VLAN là một nhóm thiết bị trên một hoặc nhiều mạng LANs, được cấu hình để giao tiếp như chúng liên kết trực tiếp với nhau, khi chúng đặt trên một số phân đoạn LAN khác nhau. Vì VLAN dựa trên liên kết logic chứ không phải liên kết vật lý khiến chúng trở nên rất linh hoạt
- VLANs định nghĩa miền quảng bá trong tâng liên kết dữ liệu (level 2). Miền quản bá thiết lập các thiết bị sẽ nhận được khung tin có nguồn gốc quảng bá từ bất kỳ thiết bị nào trong cùng VLAN. Miền quảng bá thường bị giới hạn bởi router do nó không cho phép chuyển tiếp. Switch lớp 2 tạo miền quảng bá dựa trên cấu hình của switch, tạo các cầu nối đa kênh cho phép tạo nhiều miền quản bá. Mỗi miền quảng bá như một cầu ảo riêng biệt trong switch
- Bạn có thể tạo ra một hoặc nhiều cây cầu ảo trong switch. Mỗi cây cầu ảo bạn tạo trong switch định nghĩa một miền quảng bá mới (VLAN). Lưu lượng không thể truyền trực tiếp tới các VLAN khác trong switch hoặc giữa hai switch. Để kết nối hai VLAN khác nhau bạn phải sử dụng router hoặc switch lớp 3. 
- VLANs thường được kết hợp với địa chỉ IP mạng con, tạo VLAN cần set các tham số
  - VLAN number
  - VLAN name
  - VLAN type
  - VLAN state (active or suspended)
  - Maximum transmission unit (MTU) của VLAN
  - Security Association Identifer (SAID)

## 10. Bridge, NAT

#### 10.1. Bridge

- Trong mạng, bridge là một sản phẩm để kết nối 1 mạng cục bộ với một mạng cục bộ khác sử dụng trung giao thức (Ethernet hoặc token ring). Bạn có thể hình dung 1 bridge như một thiết bị quyết định liệu một tin nhắn từ bạn tới một người nào khác đi tới mạng LAN trong tòa nhà của bạn hoặc tới người nào khác trong mạng LAN trong một tòa nhà khác. Một bridge kiểm tra mỗi thông điệp trên mạng LAN, "passing" tới những người được biết trong cùng mạng LAN và chuyển tiếp tới những người trên mạng LAN khác kết nối với nhau.
- Trong các mạng bắc cầu, máy tính và địa chỉ các node không có mối quan hệ cụ thể với vị trí. Vì lí do này, thông điệp được gửi tới mọi địa chỉ trên mạng và được chấp nhận chỉ bởi node đích dự định. Bridge học địa chỉ nào trên mạng và phát triển một bảng học để các thông điệp sau có thể chuyển tới đúng mạng.
- Mạng bắc cầu nói trung luôn kết nối các mạng cục bộ với nhau vì việc quảng bá mọi thông điệp tới các tất cả các điểm đến có thể sẽ làm mạng tràn ngập các gói tin không cần thiết. Vì lí do vày, router mạng cũng như Internet sử dụng lược đồ chỉ định địa chỉ cho các node để các thông điệp hay gói tin có thể chuyển tiếp chỉ theo mộ hướng chung thay vì chuyển tiếp theo mọi hướng.
- Bridge làm việc ở mức data-link (physical network) của một mạng, sao chép khung dữ liệu từ một mạng tới mạng tiếp theo theo đường dẫn. Bridge thỉnh thoảng kết hợp 1 router trong 1 sản phẩm gọi brouter

#### 10.2. NAT

- Network Address Translation
- Dữ liệu chuyển tiếp từ mạng LAN (sử dụng địa chỉ cục bộ) sang mạng Internet (sử dụng đỉa chỉ công cộng) và ngược lại cần chuyển đổi địa chỉ
- PAT: Port Address Translation
  - NAT with overloading sử dụng thêm số hiệu cổng ứng dụng trong quá trình chuyển đổi
- NAT có thể chuyển đổi IP từ mạng LAN này sang mạn LAN khác

## 11. QoS

#### 11.1. Khái quát

- Quality of service đề cập tới mọi công nghệ quản lý lưu lượng dữ liệu để giảm mất gói tin, độ trễ hay jitter (độ lệch về thông tin, thời gian) trên mạng. QoS điều khiển và quản lý tài nguyên mạng bằng cách set ưu tiên cho loại dữ liệu cụ thể trên mạng.
- Mạng doanh nghiệp cần cung cấp dịch vụ có thể dự đoán và đo lường như ứng dụng - như dữ liệu thoại, video, dữ liệu nhạy cảm về độ trễ đi qua mạng. Tổ chức sử dụng QoS để đáp ứng yêu cầu của ứng dụng nhạy cảm, như chat thoại và video thời gian thực, để ngăn chặn sự xuống cấp chất lượng gây ra do mất gói tin, delay và jitter.
- Tổ chức có thể hoàn thành QoS bằng cách sử công cụ và kỹ thuật nhất định như jitter buffer và traffic shaping. VỚi nhiều tổ chức, QoS bao gồm thỏa thuận mức dịch vụ với nhà cung cấp mạng để đảm bảo một hiệu suất nhất định

#### 11.2. QoS parameter

- Tổ chức có thể đo lường định lượng QoS qua một số tham số
  - Packet loss
  - Jitter
  - Latency
  - Bandwidth
  - Mean opinion score

#### 11.3. Thực hiện QoS

- Ba mô hình hiện có để thực hiện QoS: Best Effort, Integrated Services và Differentiated Services
  - Best Effort: là một mô hình QoS nơi tất cả các gói nhận được có cùng độ ưu tiên và không có sự đảm bảo việc truyền gói. Best Effort được áp dụng khi không có chính sách QoS nào hoặc khi kiến trúc hạ tầng không hỗ trợ QoS.
  - Integrated Services (IntServ): là một mô hình QoS dự trữ băng thông theo một đường dẫn cụ thể trên mạng. Ứng dụng yêu cầu mạng để đặt trước tài nguyên và thiết bị mạng giám sát luồng dữ liệu để đảm bảo tài nguyên mạng có thể chấp nhận gói. Thực hiện IntServ yêu cầu khả năng IntServ của router và sự dụng Resource Reservation Protocol (RSVP) cho việc yêu cầu tài nguyên mạng. IntSerc có khả năng mở rộng hạn chế và tiêu thụ tài nguyên mạng cao.
  - Differeniated Services (DiffServ) : là một mô hình QoS nơi các phần tử mạng cũng như router hay switch, được cấu hình với nhiều sự ưu tiên cho các truy cập khác nhau. Lưu lượng mạng phải được chia thành các lớp dựa trên yêu cầu của công ty.
- **Cơ chế QoS**
  - Một số cơ chế QoS nhất định có thể quản lý chất lượng luồng dữ liệu và duy trì các yêu cầu QoS được chỉ định trong SLAs (service-level agreement). Cơ chế QoS thuộc các danh mục cụ thể phụ thuộc vào vai trò của chúng trong việc quản lý mạng
  - Classification and marking: công cụ phân biệt giữa ứng dụng và loại gói dẫn tới loại lưu lượng khác nhau. Marking sẽ đánh dấu mỗi gói như một thành viên của lớp mạng. Phân lớp và đánh dấu được thực hiện trên các thiết bị mạng như router, switch và điểm truy cập.
  - Congestion management: công cụ sử dụng phân lớp gói và đánh dấu để xác định hàng đợi nơi chứa các gói, trong Congestion management tool bao gồm các loại hàng đợi: theo độ ưu tiên i, first-in, first-out và hàng đợi độ trễ thấp.
  - Congestion avoidance: công cụ giám sát lưu lượng mạng với việc tắc nghẽn và sẽ gỡ bỏ gói độ ưu tiên thấp khi tắc nghẽn xảy ra. Công cụ tránh tắc nghẽn bao gồm phát hiện sớm ngẫu nhiên có trọng số và phát hiện sớm ngẫu nhiên
  - Shaping: công cụ thao tác lưu lượng truy cập vào mạng và ưu tiên ứng dụng thời gian thực hơn ứng dụng ít nhạy cảm với thời gian như email, messaging. Công cụ bao gồm buffer, Generic Traffic Shaping, Frame Relay Traffic Shaping. Giống như việc định hình, công cụ chính sách lưu lượng tập trung vào điều chỉnh lưu lượng truy cập vượt quá và gỡ bỏ gói.
  - Link efficiency: công cụ tối đa hóa việc sử dụng băng thông và giảm độ trễ truy cập gói mạng. Link effciency thường được kết hợp với các cơ chế QoS khác. Link effciency bao gồm nén tiêu đề Real-Time Transport Protocol, nén tiêu đề Transmission Control Protocol và nén link.

### 

## 12. Routing

#### 12.1. Các khái niệm cơ bản

- Định nghĩa
  - Định tuyến là tìm đường đi tốt nhất để chuyển tiếp dữ liệu tới nút đích
  - Trong mạng IP chuyển mạch gói, các gói tin được chuyển tiếp độc lập, mỗi nút không chỉ ra toàn bộ các chặng trên đường tới đích
- Các thành phần của định tuyến
  - Thông tin: Thông số của đường truyền được sử dụng là độ đo tính toán chi phí đường đi
  - Bảng định tuyến: Lưu thông tin đường đi đã tìm tới các mạng đích
  - Giải thuật, giao thức định tuyến: cách thức tìm đường đi và trao đổi thông tin định tuyến giữa các nút mạng
- Router
  - Thiết bị chuyển tiếp các gói tin giữa các mạng
    - là một máy tính với phần cứng chuyên dụng
    - kết nối nhiều mạng với nhau
    - chuyển tiếp gói tin dựa trên bảng định tuyến
  - Nhiều cổng kết nối mạng
  - Phù hợp với nhiều dạng lưu lượng và phạm vi mạng

#### 12.2. Bảng định tuyến

| Destination | Outgoing Port | Next hop | Cost |
| :---------: | :-----------: | :------: | :--: |
|             |               |          |      |

- Destination: địa chỉ mạng đích
  - Định tuyến classless: sử dụng địa chỉ không phân lớp
  - Định tuyến classfull: sử dụng địa chỉ phân lớp
- Outgoing Port: cổng ra cho gói tin để tới mạng đích
- Next hop: địa chỉ cổng nhận gói tin của nút kết tiếp
- Cost: chi phí gửi gói tin từ nút xét tới nút đích

**Cập nhật bảng định tuyến**

- Thay đổi cấu trúc mạng: thêm mạng mới, một nút mạng bị mất kết nối
- Cập nhật bảng định tuyến
  - 1 số nút mạng phải cập nhật
- Cách cập nhật
  - Định tuyến tĩnh: các mục trong bảng định tuyesn được sửa đổi thủ công bởi người quản trị
  - Định tuyến động: tự động cập nhật bảng định tuyến các giao thức định tuyến

**Định tuyến tĩnh** 

- Khi có sự cố:
  - Không thể kết nối vào internet kể cả khi có tồn tại đường đi dự phòng
  - Người quản trị mạng cần thay đổi

**Định tuyến động**

- Khi có sự cố:
  - Đường đi thay thế được cập nhật một cách tự động

#### 12.3. Các giải thuật và giao thực định tuyến

##### 12.3.1. Giải thuật distance-vector

`  dx(y) = min {c(x,v) + dv(y) }` 

dx(y): chi phí của đường đi ngắn nhất từ x đến y

c(x,v): chi phí từ x tới hàng xóm v

v là tất cả hàng xóm của x

##### 12.3.2. Giải thuật link-state

- Mỗi nút thu thập thông tin từ các nút khác để xây dựng topo của mạng
- Áp dụng giải thuật tìm đường đi ngắn nhất tới mọi nút trong mạng
- Khi có sự thay đổi bảng định tuyến, mỗi nút gửi thông tin thay đổi cho tất cả các nút khác trong mạng
- Đồ thị
  - G = (V, E): đồ thị với tập đỉnh V và tập cạnh E
  - c(x, y): chi phí của liên kết x tới y, ∞ nếu không phải 2 nút kế nhau
  - d(v): chi phí hiện thời của đường đi từ nút nguồn tới nút đích v
  - p(v): nút ngay trước nút v trên đường đi từ nguồn tới đích
  - T: tập các nút mà đường đi ngắn nhất đã được xác định
- Thủ tục
  - Init(): 
    - Với mỗi nút v, d[v] = ∞ , p[v] = NIL d[s] = 0
  - update(u, v), (u, v) là một cạnh nào đó của G
    - if d[v] > d[u] + c(u, v) then d[v] = d[u] + c(u, v), p[v] = u
- Giải thuật Dijsktra
  - Init()
  - T = NULL;
  - Repeat
  - u: u không thuộc T hoặc d(u) là bé nhất
  - T = T ∪ {u}
  - for all v thuộc hangxom(u) và v không thuộc T
  - update(u, v)
  - Until T = V









