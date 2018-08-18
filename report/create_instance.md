## Các bước tạo một instance trong openstack và chức năng của các core service 
- Create virtual network
  - Login vào controller node, chạy file xác thực keystonerc sử dụng quyền admin hoặc người dùng có đặc quyền để sử dụng Neutron tạo virtual network.
- Create flavor
  - Flavor sẽ định nghĩa số lượng tài nguyên được sử dụng số CPU, RAM, dung lượng bộ nhớ instace được sử dụng trong quá trình khởi chạy. 
- Generate a key pair
  - openstack cung cấp cơ chế để quản lý SSH key pair để truy cập xác thực không cần password khi khởi chạy instance.
- Add security group rules
  - Thiết lập để có thể truy cập từ xa đến các instance qua ICMP và SSH
- Create instance 

### Create virtual network
Với option một networking khi cài đặt chỉ cần tạo provider network, với option hai networking cần tạo provider và self-service network.

#### Provider network 
Instance sử dụng provider(external) network để kết nối đến hạ tầng mạng vật lý qua layer-2 (bridging/switching). Provider network bao gồm một DHCP server cung cấp địa chỉ IP cho instance. 
##### Create provider network 
- Source admin credentials để truy cập với quyền admin
```bash
$ . admin-openrc

		
```
- ```--share``` : 123132



​		

- Create the network

  ```bash
  $ openstack network create --share --external \
  --provider-physical-network provider \
  --provider-network-type flat provider
  ```

  - ```--share```:  ddinghsdfsdf
  - ```share```: sdfsd

  ```bash
  $ openstack network create --share --external \
  --provider-physical-network provider \
  --provider-network-type flat provider
  ```
  - ```--share```: cho phép tất cả các project của openstack sử dụng the virtual network.
  - ```--external```: định nghĩa the virtual network là external network; thay thế bằng ```--internal``` để định nghĩa internal network.
  - ```--provider-physical-network provider``` và ```---provider-network-type flat``` tuỳ chọn để kết nối flat virtual network đến flat physical network qua interface được định nghĩa trong file linuxbridge_agent.ini 
  **ml2_conf.ini**
  ```
  
  ```
```bash
[ml2_type_flat]
flat_networks = provider
```
**linuxbridge_agent.ini**
```bash
[linux_bridge]
physical_interface_mappings = provider:eth1
```
- Create a subnet on the network 
  ```bash
  $ openstack subnet create --network provider \
  --allocation-pool start=START_IP_ADDRESS,end=END_IP_ADDRESS \
  --dns-nameserver DNS_RESOLVER --gateway PROVIDER_NETWORK_GATEWAY \
  --subnet-range PROVIDER_NETWORK_CIDR provider
  ```
	- ```PROVIDER_NETWORK_CIDR```: cidr của subnet
	- ```START_IP_ADDRESS```, ```END_IP_ADDRESS```: địa chỉ IP đầu và cuối của dải mạng trong subnet để cấp phát cho các instance.
	- ```DNS_RESOLVER```: đỉa chỉ IP của một máy chủ phân giải DNS thường lấy từ file */etc/resolv.conf* 
	- ```PROVIDER_NETWORK_GATEWAY```: địa chỉ gateway của provider 
##### Create self-service network 
Được tạo khi chọn networking option 2, self-service(private) network để kết nối đến hạ tầng physical network qua NAT. Self-service network bao gồm một DHCP server cung cấp địa chỉ IP cho các instance. Mỗi instance trên mạng này có thể truy cập external network như Internet, nhưng truy cập instance từ external network yêu cầu floating IP address.
- Source demo credentials để được truy cập quyền user
```bash
$ . demo-openrc
```
- Create the network```bash
$ openstack network create selfservice
```
User không có đặc quyền không hỗ trợ  bổ sung các tham số với command mà dịch vụ này tự động chọn tham số sử dụng thông tin từ các file
**ml2_conf.ini**
​```bash
[ml2]
tenant_network_types = vxlan

[ml2_type_vxlan]
vni_ranges = 1:1000
```
- Create a subnet on the network
```bash
$ openstack subnet create --network selfservice \
  --dns-nameserver DNS_RESOLVER --gateway SELFSERVICE_NETWORK_GATEWAY \
  --subnet-range SELFSERVICE_NETWORK_CIDR selfservice
```
	- ```DNS_RESOLVER```: địa chỉ IP của máy chủ phân giải DNS thường lấy giá trị từ file */etc/resolv.conf*
	- ```SELFSERVICE_NETWORK_GATEWAY```: địa chỉ gateway sử dụng trên self-service network
	- ```SELFSERVICE_NETWORK_CIDR```: địa chỉ subnet sử dụng trên self-service network 
	- example:
		- ```bash
$ openstack subnet create --network selfservice \
  --dns-nameserver 8.8.4.4 --gateway 172.16.1.1 \
  --subnet-range 172.16.1.0/24 selfservice

Created a new subnet:
+-------------------+--------------------------------------+
| Field             | Value                                |
+-------------------+--------------------------------------+
| allocation_pools  | 172.16.1.2-172.16.1.254              |
| cidr              | 172.16.1.0/24                        |
| created_at        | 2016-11-04T18:30:54Z                 |
| description       |                                      |
| dns_nameservers   | 8.8.4.4                              |
| enable_dhcp       | True                                 |
| gateway_ip        | 172.16.1.1                           |
| headers           |                                      |
| host_routes       |                                      |
| id                | 5c37348e-e7da-439b-8c23-2af47d93aee5 |
| ip_version        | 4                                    |
| ipv6_address_mode | None                                 |
| ipv6_ra_mode      | None                                 |
| name              | selfservice                          |
| network_id        | b9273876-5946-4f02-a4da-838224a144e7 |
| project_id        | 3828e7c22c5546e585f27b9eb5453788     |
| project_id        | 3828e7c22c5546e585f27b9eb5453788     |
| revision_number   | 2                                    |
| service_types     | []                                   |
| subnetpool_id     | None                                 |
| updated_at        | 2016-11-04T18:30:54Z                 |
+-------------------+--------------------------------------+
		```
- Create a router
Self-service network sử dụng một virtual router để kết nối tới provider vận hành NAT hai chiều. Mỗi router chứa một interface trên ít nhất một self-service network và một gateway trên một provider network.
Provider network phải bao gồm tuỳ chọn **router:external** để cho phép router self-service sử dụng nó để kết nối tới external network (Internet). Admin hoặc user đặc quyền phải trong các uỳ chọn trong quá trình tạo mạng hoặc được thêm vào.
	- Source the demo credential để cho phép truy cập quyền người dùng
	```bash
$ . demo-openrc
	```
	- Create the router:
	```bash
$ openstack router create router
	```
	- Thêm self-service network subnet như một interface trên router 
	```bash
$ openstack router add subnet router selfservice
	```
	- Thiết lập gateway trên router  provider network 
	```bash
$ openstack router set router --external-gateway provider
	```
### Create flavor
```bash
$ openstack flavor create --id 0 --vcpus 1 --ram 64 --disk 1 demo.nano 
```

### Generate a key pair 
Các image cloud hỗ trợ dùng public key để xác thực thay vì việc dùng password để xác thực. Trước khi khởi chạy instance, cần thêm public key tới Compute service
- Source demo project credential:
```bash
$ . demo-openrc
```
- Generate a key pair and add a public key:
```bash
$ ssh-keygen -q -N ""
$ openstack keypair create --public-key ~/.ssh/id_rsa.pub KEY_NAME 
```
- Verify addition keypair
```bash
$ openstack keypair list
```

### Add security group rules
Mặc định, **default** security group cung cấp tới tất cả instance, bao gồm cả quy tắc firewall từ chối truy cập từ xa tới các instance. Với Linux image nên cho phép ICMP và SSH.
- Add rule to the **default** security group:
	- Permit ICMP:
	```bash
$ openstack security group rule create --proto icmp default
	```
	- Permit secure shell SSH:
	```bash
$ openstack security group rule create --proto tcp --dst-port 22 default
	```

### Launch an instance 
#### Launch an instance on the provider network 
##### Determine instance options 
*Để khởi chạy instance bắt buộc phải có các tham số chỉ định flavor, image name, network, security group, key và instance name*
- On controller node, source the demo credential để được truy cập quyền user:
```bash
$ . demo-openrc
```
- View list available flavors:
```bash
$ openstack flavor list
```
- View list available images:
```bash
$ openstack image list
```
- View list available networks:
```bash
$ openstack network list
```
- View list available security groups:
```bash
$ openstack security group list
```
##### Launch the instance 
- Create and launch
```bash
$ openstack server create  --flavor FLAVOR_NAME --image IMAGE_NAME \
  --nic net-id=PROVIDER_NET_ID --security-group SECURITY_GR_NAME \
  --key-name KEY_NAME INSTANCE_NAME 
```
- Check status instance
```bash
$ openstack server list
```

