# Networking

Openstack Networking service (neutron) cung cấp API cho phép người sử dụng thiết lập và định nghĩa các kết nối mạng địa chỉ trong cloud. Khi xây dựng mạng ảo trong OpenStack, chúng ta phải xây dưng các switch ảo và liên kết các switch ảo với mạng vật lý, OpenStack cung cấp 2 cơ chế phổ biến là Linux bridge mechanism driver và Open vSwitch mechanism driver.

## Linux bridge 

Neutron kết hợp các Linux bridge kết hợp các thiết bị ảo và các thiết bị vật lý để tạo hệ thống mạng giữa các instance. Linux bridge sử dụng trong OpenStack xây dựng các loại mạng:

- FLAT
- VLAN
- VXLAN 

### FLAT 

Trong mạng Flat các gói tin được các switch chuyển tiếp bình thường nên mỗi một card mang vật lý chỉ có thể triển khai được một mạng Flat. Khi triển khai mạng FLat một bridge ảo sẽ được tạo nối các node và card mạng vật lý với nhau, các máy ảo sử dụng mạng FLAT đó sẽ nối card mạng ảo với bridge ảo để tham gia vào mạng. 

![flat-linux-bridge](images/LinuxBridge-Flat.png)

### VLAN 

Trong quá trình triển khai VLAN bằng Linux bridge plugin, Linux bridge sẽ kết hợp với các device VLAN interface ảo cos teen ethx.VLAN_ID với VLAN_ID là ID của mạng VLAN. Mỗi mạng VLAN sẽ tạo ra một Linux bridge và VLAN-Device, các máy ảo thuộc cùng một VLAN trên một node sẽ kết nối card mạng ảo với Linux bridge quản lý VLAN đó , linux bridge sẽ kết nối với VLAN device để thực hiện chức năng như một switch vật lý trong mạng VLAN.

![vlan-linux-bridge](images/LinuxBridge-VLAN.png)

Luồng L2 frame chuyển tiếp trong mạng:

- Khi cần chuyển tiếp gói tin L2 Frame từ máy ảo qua máy ảo khác cùng thuộc VLAN nhưng thuộc node khác, Linux bridge sẽ chuyển tiếp gói tin đó qua VLAN device, VLAN device sẽ thêm các trường VLAN Header với ID là ID của mạng VLAN rồi chuyển tiếp ra mạng vật lý qua card vật lý eth1
- Khi tiếp nhận gói tin L2 Frame từ mạng vật lý bên ngoài, gói tin sẽ đi qua eth1 vào các VLAN device để kiểm tra VLAN ID nếu hợp lệ VLAN device sẽ gỡ bỏ các trường VLAN Header và chuyển L2 Frame tới Linux bridge, Linux bridge sẽ kiểm tra địa chỉ MAC đích rồi chuyển tiếp gói tin L2 Frame theo địa chỉ MAC
- Khi chuyển tiếp giữa các máy VLAN cùng trong 1 node, Linux bridge hoạt động như một switch bình thường.

### VXLAN

Các Linux bridge kết hợp với VXLAN device để thực hiện chức năng của một VTEP, VXLAN device được đặt tên tương ứng với ID của mạng VXLAN mà nó được triển khai.

Hoạt động của linux bridge với VXLAN device trong mạng:

- Khi cần chuyển tiếp gói tin L2 Frame từ máy ảo tới máy ảo khác cùng mạng nhưng trên node khác, Linux bridge chuyển tiếp gói tin vơi VXLAN device, sau đó frame được thêm header với VXLAN ID rồi đóng gói vào gói tin UDP + IP, gói tin này được chuyển tiếp theo địa chỉ IP với IP nguồn là IP card vật lý, IP đích là IP mạng vật lý của node chứa máy ảo đích.
- Khi nhận gói tin IP từ mạng bên ngoài, gói tin đia qua card mạng vật lý, VXLAN device kiểm tra VXLAN ID nếu hợp lệ L2 Frame sẽ chuyển tiếp tới Linux bridge và chuyển tiếp đến đích dựa vào địa chỉ MAC.

### Network traffic flow

#### Provider network 

Kiến trúc mạng provider network cung cấp kết nối qua layer 2 giữa các instance và physical network.

![linuxbridge-provider](images/linuxbridge-provider.png)

Các thành phần và kết nối trong các mạng FLAT, VLAN

![linuxbridge-flat](images/linuxbridge-flat.png)

![linuxbridge-vlan.png](images/linuxbridge-vlan.png)

##### Network traffic flow

Cấu hình của các instance và mạng như sau

- Provider network 1 (VLAN)
  - VLAN ID 101 (tagged)
  - IP address ranges 203.0.113.0/24 và fd00:203:0:113::/64
  - Gateway (thông qua hạ tầng mạng vật lý)
- Provider network 2 (VLAN)
  - VLAN ID 102 (tagged)
  - IP address ranges 192.0.2.0/24 và fd00:192:0:2::/64
  - Gateway 
    - IP address 192.0.2.101và fd00:192:0:2::1
- Instance 1
  - IP address 203.0.113.101 và fd00:203:0:113:0::101
- Instance 2
  - IP address 192.0.2.101 và fd00:192:0:2:0::101

**Trường hợp 1**: instance gửi packet tới host trên Internet, instance có địa chỉ IP cố định, trên node compute 1 và sử dụng mạng provider network 1.

![north-south scenario](images/north-south scenario.png)

- Trên node compute 1
  - B1: Instance interface (1) chuyển tiếp packet tới provider bridge instace port (2) qua veth.
  - B2: Security group rule (3) trên provider bridge xử lý firewalling và connection tracking đối với packet.
  - B3: VLAN sub-interface port (4) trên provider bridge chuyển tiếp packet tới physical network interface (5).
  - B4: Physical network interface (5) thêm VLAN tag (VLAN ID) 101 vào packet và chuyển tiếp nó tới physical network infrastructure switch (6).
- Trên physical network infrastructure
  - B1: switch gỡ bỏ VLAN tag 101 từ packet và chuyển tiếp nó tới router (7).
  - B2: router sẽ định tuyến packet từ provider network (8) tới external network (9) và chuyển tiếp packet tới switch (10).
  - B3: switch chuyển tiếp packet tới external network (11)
  - B4: external network (12) nhận được packet.

**Trường hợp 2**: Instance 1 trên node compute 1 sử dụng provider network 1, instance 2 trên node compute 2 và sử dụng provider network1, instance 1 gửi packet tới instance 2

![east-west1-provider.png](images/east-west1-provider.png)

 Trên node compute 1

- B1: Instance 1 interface (1) chuyển tiếp packet tới provider bridge instance port (2) qua veth pair.
- B2: Security group rule (3) trên provider bridge xử lý firewalling và connection tracking cho packet.
- B3: VLAN sub-interface port (4) trên provider bridge chuyển tiếp packet tới physical network interface (5).
- B4: physical network interface (5) thêm VLAN tag 101 vào packet và chuyển tiếp nó tới physical network infrastructure switch (6).

Trên physical network infrastructure

- Switch forward packet từ compute node 1 tới compute node 2 (7).

Trên compute node 2

- B1: Physical network interface (8) gỡ VLAN tag 101 từ packet và chuyển tiếp nó tới VLAN sub-interface port (9) trên provider bridge.
- B2: security group rule (10) trên provider bridge xử lý firewalling và connection tracking cho packet.
- B3: provider bridge instance port (11) chuyển tiếp packet tới instance 2 interface (12) thông qua veth pair.

**Trường hợp 3**: Instance 1 trên node compute 1, sử dụng provider network 1; Instance 2 trên node compute 1, sử dụng provider network 2; instance 1 gửi packet tới instance 2.

![east-west2-provider.png](images/east-west2-provider.png)

Trên compute node

- B1: Instance 1 interface (1) chuyển tiếp packet tới provider bridge instance port (2) qua veth pair.
- B2: Security group rule (3) trên provider bridge xử lý firewalling và connection tracking cho packet.
- B3: VLAN sub-interface port (4) trên provider bridge chuyển tiếp packet tới physical network interface (5).
- B4: Physical network interface (5) thêm VLAN tag 101 vào packet và chuyển tiếp nó tới physical network infrastructure switch (6)

Trên physical network infrastructure

- B1: switch gỡ VALn tag 101 từ packet và chuyển tiếp nó tới router (7).
- B2: router định tuyến packet từ provider network 1 (8) tới provider network 2 (9).
- router chuyển tiếp packet tới switch (10).
- switch thêm VLAN tag 102 vào packet và chuyển tiếp nó tới compute node 1 (11)

Trên compute node

- B1: physical network interface (12) gỡ VLAN tag 102 từ packet và chuyển tiếp nó tiếp nó tới VLAN sub-interface port (13) trên provider bridge.
- B2: security group rule (14) trên provder bridge thực hiện firewalling và connection tracking cho packet.
- B3: provider bridge instance port (15) chuyển tiếp packet tới instance 2 interface (16) qua veth pair.

#### Selfservice network

Kiến trúc service network chưa kiến trúc provider network và thêm một mạng overlay 

![linuxbridge-selfservice.png](images/linuxbridge-selfservice.png)

Kiến trúc các component và kết nối cho một selfservice network và một provider network FLAT. 

![linuxbridge-selfservce-component.png](images/linuxbridge-selfservce-component.png)

##### Network traffic flow

- Provider network (VLAN)
  - VLAN ID 101 (tagged)
- Self-service network 1 (VXLAN)
  - VXLAN ID (VNI) 101
- Self-service network 2 (VXLAN)
  - VXLAN ID (VNI) 102
- Self-service router
  - Gateway trên provider network
  - Interface trên self-service network 1
  - Interface trên self-service network 2
- Instance 1
- Instance 2

**Trường hợp 1**: instance trên node compute 1 sử dụng selff-service network 1 địa chỉ IP cố định, gửi packet tới host trên internet

![linuxbridge-selfservice-north-south1.png](images/linuxbridge-selfservice-north-south1.png)

Trên node compute 1

- B1: Instance interface (1) chuyển tiếp packet tới self-service bridge instance port (2) qua veth pair.
- B2: Security group rule (3) trên self-service bridge xử lý firewalling và connection tracking cho packet.
- B3: Self-service bridge chuyển tiếp packet tới VXLAN interface (4) wrap packet với VNI 101.
- B4: Underlying physical interface (5) của VXLAN interface chuyển tiếp packet tới node mạng thông qua overlay network (6).

Trên network node

- B1: Underlying physical interface (7) của VXLAN interface chuyển tiếp packet tới VXLAN interface và unwrap packet bỏ VNI 101.
- B2: Self-service bridge router port (9) chuyển tiếp packet tới self-service network interface (10) trong router namespace.
  - Với IPv4, router thực hiện SNAT trên packet, thay đổi địa chỉ IP đích thành địa chỉ IP router trên providể network thông qua gateway interface trên provider network (11).
  - Với IPv6, router gửi packet tới địa chỉ IP next-hop, thươngf là địa chỉ IP gateway trên providể network thông qua provider gateway interface (11).
- B3: router chuyển tiếp packet tới providể bridge router port (12).
- B4: VLAN sub-interface port (13) trên provider bridge chuyển tiếp gói tin tới provider physical network interface (14).
- B5: provider physical network interface (14) thêm VLAN tag 101 tới packet và chuyển tiếp nó đến Internet thông qua physical netwỏk infrastructure (15).

**Trường hợp 2**: Instance trên compute node 1 với floating IPv4 address, sử dụng self-service network 1, một host trên Internet gửi packet tới instance.

Với instance có floating IPv4 address, network node thực hiện SNAT khi traffic network passing từ instance tới network và sử dụng DNAT khi traffic network passing từ external network tới instance.

![linuxbrdige-selfservice-floatingIP.png](images/linuxbrdige-selfservice-floatingIP.png)

Trên network node

- B1: Physical network infrastructure (1) chuyển tiếp packet tới provider physical network interface (2).
- B2: Provider physical network interace gỡ VLAN tag 101 và chuyển tiếp packet tới VLAN sub-interface trên provider bridge.
- B3: Provider bridge chuyển tiếp packet tới self-service router gateway port trên provider network (5).
  - Với IPv4, router thực hiện DNAT với packet thay đổi địa chỉ IP đích thành địa chỉ IP của instance trên self-service network và gửi nó tới địa chỉ gateway trên self-service network thôn qua self-service interface (6).
  - Với IPv6, router gửi packet tới địa chỉ IP của next-hop thường là địa chỉ gateway trên self-service network thông qua self-service interface (6).
- B4: router chuyển tiếp packet tới self-service bridge router port (7).
- B5: self-service bridge chuyển tiếp packet tới VXLAN interface (8) wrap packet với VNI 101.
- B6: Underlying physical interface (9) cho VXLAN interface chuyển tiếp packet tới node network thông qua overlay network (10).

Trên node compute

- B1: Underlying physical interface (11) cho VXLAN interface chuyển tiếp packet tới VXLAN interface (12), thực hiện unwrap packet gỡ bỏ VNI 101.
- B2: Security group rule (13) trên self-service bridge xử lý firewalling và connection tracking cho packet.
- B3: Self-service bridge instance port (14) chuyển tiếp packet tới instance interface (15) thông qua veth pair.

**Trường hợp 3**: Instance 1 trên node compute 1 sử dụng self-service network 1, Instance 2 tren node compute 2 sử dụng self-service network 1, Instance 1 gửi packet tới Instance 2.

![linuxbridge-selfservice-samenetwork.png](images/linuxbridge-selfservice-samenetwork.png)

Trên node compute 1

- B1: Instance 1 interface (1) chuyển tiếp packet tới self-service bridge instance port (2) qua veth pair.
- B2: Security group rule (3) trên self-service bridge xử lý firewalling và connection tracking với packet.
- B3: Self-service bridge chuyển tiếp packet tới VXLAN interface (4) và wrap packet với VNI 101.
- B4: Underlying physical interface (5) với VXLAN interface chuyển tiếp packet tới compute node 2 thông qua overlay network (6).

Trên node compute 2

- B1: Underlying physical interface (7) với VXLAN interface chuyển tiếp packet tới VXLAN interface (8) unwrap packet bỏ VNI 101.
- B2: Security group rule (9) trên self-service bridge xử lý firewalling, connection tracking với packet.
- B3: self-service bridge instance port (10) chuyển tiếp packet tới instance 1 interface (11) qua veth pair.

**Trường hợp 4**: Instance 1 trên node compute 1 sử dụng self-service network 1, Instance 2 trên node compute 1 sử dụng self-service network 2, Instance 1 gửi packet tới Instance 2.

![linuxbridge-selfservice-differnet.png](images/linuxbridge-selfservice-differnet.png)

Trên node compute

- B1: Instance 1 interface (1) chuyển tiếp packet tới self-service bridge instance port (2) qua veth pair.
- B2: Security group rule (3) trên self-service bridge xử lý firewalling, connection tracking với packet.
- B3: Self-service bridge chuyển tiếp packet tới VXLAN interface (4) và wrap packet với VNI 101.
- B4: Underlying physical interface (5) với VXLAN interface chuyển tiếp packet tới compute node 2 thông qua overlay network (6).

Trên node network

- B1: Underlying physical interface (7) với VXLAN interface chuyển tiếp packet tới VXLAN interface (8) unwrap packet bỏ VNI 101.
- B2: Self-service bridge router port (9) chuyển tiếp packet tới self-service network 1 interface (10) trong router namespace.
- B3: Router gửi packet tới địa chỉ next-hop thường là địa chỉ gateway trên self-service network 2, thông qua self-service network 2 interface (11).
- B4: Router chuyển tiếp packet tới self-service network 2 bridge router port (12).
- B5: Self-service network 2 bridge chuyển tiếp packet tới VXLAN interface (13) wrap packet với VNI 102.
- B6: Physical network interface (14) với VXLAN interface gửi packet tới compute node qua overlay network (15).

Trên node compute

- B1:  Underlying physical interface (16) với VXLAN interface chuyển tiếp packet tới VXLAN interface (17) unwrap packet bỏ VNI 102.
- B2: Security group rule (18) trên self-service bridge xử lý firewalling, connection tracking với packet.
- B3: Self-service bridge port (19) chuyển tiếp packet tới instance 2 interface (20) qua veth pair.

## Open vSwitch

Open vSwitch trong OpenStack cho phép tạo và quản lý các đối tượng switch ảo để quản lý hệ thống mạng trong cloud

### Open vSwitch trong Provider network

#### Prerequisites

- Một controller node với các component
  - Hai network interface: mangement và provider network
  - OpenStack Networking server service và ML2 plug-in
- Hai compute node với các component
  - Hai network interface: mangement và provider network
  - OpenStack Networking Open vSwitch (OVS)  layer-2 agent, DHCP agent, metadata agent và tất cả các gói phụ thuộc bao gồm trong OVS

#### Architecture

![openvswitch](images/openvswitch.jpg)

Các component và kết nối trong mạng FLAT

![openvswitch-flat.JPG](images/openvswitch-flat.JPG)

Giữa hai mạng VLAN khác nhau sử dụng chung OVS integration

![openvswitch-vlan](images/openvswitch-vlan.jpg)

### Network traffic flow

Cấu hình mạng và các instance

- Provider network 1 (VLAN)
  - VLAN ID 101 (tagged)
  - IP address ranges 203.0.113.0/24 and fd00:203:0:113::/64
  - Gateway (via physical network infrastructure) 
    - IP addresses 203.0.113.1 and fd00:203:0:113:0::1
- Provider network 2 (VLAN)
  - VLAN ID 102 (tagged)
  - IP address range 192.0.2.0/24 and fd00:192:0:2::/64
  - Gateway
    - IP addresses 192.0.2.1 and fd00:192:0:2::1
- Instance 1
  - IP addresses 203.0.113.101 and fd00:203:0:113:0::101 
- Instance 2
  - IP addresses 192.0.2.101 and fd00:192:0:2:0::101

**Trường hợp 1**: North-south

- Instance trên node compute 1 sử dụng provider network 1, gửi packet tới một host trên Internet

  ![](images/openvswitch-flat.jpg)

Trên node compute

- B1: Instance interface (1) chuyển tiếp packet tới security group bridge instance port (2) qua veth pair. 
- B2: Security group rule (3) trên security group bridge xử lý firewalling và connection tracking cho packet
- B3: Security group bridge OVS port (4) chuyển tiếp packet tới OVS integration bridge security group port (5) qua veth pair.
- B4:  OVS integration bridge thêm tag VLAN internal vào packet
- B5:  OVS integration bridge `int-br-provider` patch port (6) chuyển tiếp packet tới OVS provider bridge `phy-br-provider` patch port (7)
- B6: OVS provider bridge chuyển đổi internal VLAN tag với VLAN tag 101.
- B7: OVS provider bridge provider network port (8) chuyển tiếp packet tới physical network interface (9)
- B8: Physical network interface chuyển tiếp packet tới physical network infrastructure switch (10)

Trên physical network infrastructure

- Switch gỡ VLAN tag 101 từ packet và chuyển tiếp tới router (11)
- Router định tuyến packet từ provider network (12) tới external network (13) và chuyển tiếp packet tới switch (14)
- Switch chuyển tiếp packet tới external network (15)
- External networrk (16) nhận packet

**Trường hợp 2**: East-west, instance 1 trên node compute 1, instance 2 trên node compute 2, cùng sử dụng provider network 1, Instance 1 gửi packet tới Instance 2

![](images/openvswitch-provider-vlan-differnode.jpg)

Trên node compute 1

-  Tương tự trường hợp 1

Trên physical network infrastructure 

- Switch chuyển tiếp packet từ compute 1 tới node compute 2 (11)

Trên compute node 2

- Physical network interface (12) chuyển tiếp packet tới OVS provider bridge provider network port (13)
- OVS provider bridge `phy-br-provider` patch port (14) chuyển tiếp packet tới OVS integration bridge `int-br-provider` patch port (15)
- OVS integration bridge chuyển đổi VLAN tag 101 với internal VLAN tag
- OVS integration bridge security group port (16) chuyển tiếp packet tới security group bridge OVS port (17)
- Security group rule (18) trên security group bridge xử lý firewalling và connection tracking với packet
- Security group bridge instance port (19) chuyển tiếp packet tới instance 2 (20) qua veth pair

**Trường hợp 3**: East-west, instance 1, 2 cùng trên node compute 1 sử dụng lần lượt proivder network 1, 2; Instance 1 gửi packet tới instance 2

![](images/openvswitch-provider-vlan-differnet.jpg)

Trên compute node

- Tương tự kịch bản ở trường hợp 1

Trên physical network infrastructure

- Switch gỡ VLAN tag 101 từ packet và chuyển tiếp tới router (11)
- Router định tuyến packet từ provider network 1 (12) tời provider network 2 (13)
- Router chuyển tiếp gói tin tới switch (14)
- Switch thêm VLAN tag 102 vào packet và chuyển nó đến compute node 1 (15)

Trên compute node

- Tương tự khi nhận packet từ physical network tại compute node 2 ở trường hợp 2

### Open vSwitch trong Self-service network

#### Prerequisites

- Tương tự proivder network nhưng thêm network node với các component
  - 3 network interface: management, provider, overlay
  - OpenStack Networking Open vSwitch (OVS) layer-2 agent, layer-3 agent và các gói phụ thuộc cần cho OVS
- Trên compute node cần thêm network interface: overlay

#### Architecture

![openvswitch-selfservice](images/openvswitch-selfservice.jpg)

Self-service network và một provider network FLAT

![openvswitch-selfservice-flat](images/openvswitch-selfservice-flat.jpg)

#### Network Traffic Flow

**Trường hợp 1**: Instance trên node compute 1 sử dụng self-service network 1, với địa chỉ IP cố định giao tiếp với Internet

![](images/openvswitch-selfservice-instancetoout.jpg)

Trên compute node 1 

- Instance interface (1) gửi gói tin tới security group port (2) qua veth pair
- Security group rule (3) trên bridge xử lý firewalling và connection tracking cho packet
- Security group bridge OVS port (4)  chuyển tiếp packet tới OVS integration bridge security group port (5) via `veth` pair
- OVS integration bridge thêm internal VLAN tag vào packet
- OVS integration bridge chuyển đổi internal VLAN tag thành một internal tunnel ID
- OVS integration bridge  patch port (6) chuyển tiếp packet tới OVS tunnel bridge patch port (7)
- OVS tunnel bridge (8) wrap packet với VNI 101
- Underlying physical interface (9) chuyển tiếp packet tới network node thông qua overlay network (10)

Trên network node

- Underlying physical interface (11) chuyển tiếp packet tới OVS tunnel bridge (12)
- OVS tunnel bridge unwrap VNI  101 của packet và thêm internal tunnel ID
- OVS tunnel bridge  chuyển internal tunnel ID thành internal VLAN tag
- OVS tunnel bridge patch port (13) chuyển tiếp packet tới OVS integration bridge patch port (14)
- OVS integration bridge port (15) gỡ internal VLAN tag và chuyển tiếp tới self-service network interface (16) trong router namespace
- Router chuyển tiếp packet tới OVS integration bridge port  của provider network (18)
- OVS integration bridge thêm internal VLAN tag chuyển tới OVS provider thông qua path port (19), và chuyển internal VLAN thành VLAN tag 101
- Packet được chuyển tới physical network

**Trường hợp 2**: Instance 1 trên compute 1, Instance 2 trên compute 2 cùng sử dụng self-service network 1 và giao tiếp với nhau

![](images/openvswitch-selfservice-differnode.jpg)

- Giống như trường hợp một do 2 instance giao tiếp với nhau trên cùng VXLAN network

**Trường hợp 3**: Instance 1 và Instance 2 cùng trên node compute 1 nhưng sử dụng   lần lượt self-service network 1 và 2

![](images/openvswitch-selfservice-differnet.jpg)

Trên compute node

- tương tự như kịch bản 1 nhưng cần thêm giao tiếp với network node để làm trung gian chuyển packet tới đích

Trên network node

- Physical network infrastructure (1) chuyển tiếp packet tới provider physical network interface (2)
- Provider physical network interface chuyển tiếp gói tin tới OVS provider bridge provider network port (3).
- OVS provider bridge chuyển VLAN tag 101 thành internal VLAN tag.
- OVS provider bridge phy-br-provider port (4) chuyển tiếp gói tin tới OVS integration bridge port for the provider network (6) gỡ bỏ internal VLAN tag chuyển tiếp gói tin tới the provider network interface (6) trong router namespace.
- router chuyển tiếp gói tin tới OVS integration bridge port for the self-service network (9).
- OVS integration bridge thêm internal VLAN tag tới packet
- OVS integration bridge chuyển internal VLAN tag thành internal tunnel ID.
- OVS integration bridge patch-tun patch port (10) chuyển tiếp gói tin tới OVS tunnel bridge patch-int patch port (11).
- OVS tunnel bridge (12) wraps  packet với VNI 101.
- underlying physical interface (13) for overlay networks chuyển tiếp gói tin tới network node qua overlay network (14).