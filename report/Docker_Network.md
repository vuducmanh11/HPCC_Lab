# Docker Networking
## Overview
Một trong những lý do Docker container và service rất mạnh mẽ đó là bạn có thể kết nối chúng với nhau hoặc kết nối chúng với non-Docker workload. Docker container và service thậm chí không cần biết chúng được triển khai trên Docker hoặc là nó liên kết với workload của Docker hay không. Bất kể Docker chạy trên Linux, Windows hay cả hai, ta có thể sử dụng Docker để quản lý theo cách platform-agnostic.

Topic này xác định một số khai niệm Docker networking cơ bản và chuẩn bị để thiết kế và triển khai ứng dụng tận dụng tối đa khả năng của Docker.

## Network drivers
Docker's networking subsystem là pluggable, sử dụng driver. Một số driver tồn tại mặc định cung cấp chức năng cơ bản của networking
  - `bridge`: default network driver, thường sử dụng khi ứng dụng chạy trong các container độc lập cần giao tiếp với nhau.
  - `host`: cho các container độc lập, bỏ sự cô lập network giữa container và Docker host, sử dụng trực tếp host networking. `host` chỉ khả dụng với swarm từ Docker 17.06
  - `overlay`: overlay network kết nối nhiều Docker daemon với nhau và cho phép swarm service giao tiếp với nhau. Bạn có thể cũng sử dụng overlay network để dễ dàng giao tiếp giữa swarm service và standalone container, hoặc giữa hai container độc lập trên Docker daemon khác nhau. Việc này loại bỏ sự cần thiết sử dụng OS-level routing giữa các container.
  - `macvlan`: Macvlan network cho phép bạn chỉ định địa chỉ MAC cho container, khiến container giống như physical device trên mạng của bạn. Docker daemon định tuyến lưu lượng tới container bởi địa chỉ MAC của container. Sử dụng `macvlan` driver đôi lúc là sự lựa chọn tốt nhất khi đối phó với legacy application mong đợi kết nối trực tiếp tới physical network hơn so với việc định tuyến thông qua mạng của Docker host.
  - `none`: với container, không cho phép tất cả các mạng. Thường được sử dụng kết hợp với custom network driver. `none` không cho phép với swarm service.
  - `Network plugins`: có thể cài đặt và sử dụng network plugins bên thứ ba với Docker

### Bridge Network

Trong Docker, bridge network sử dụng một software bridge cho phép các container kết nối tới cùng bridge network giao tiếp với nhau, tách biệt với các container không kết nối tới cùng bridge network. Docker bridge driver tự động thiết lập quy tắc trong host machine để các container trên bridge network khác nhau không thể giao tiếp trực tiếp với nhau.

Bridge network áp dụng với các container chạy trên cùng Docker daemon host. Với giao tiếp giữa các containers chạy trên Docker daemon host khác nhau, bạn có thể quản lý định tuyến ở OS level hoặc bạn sử dụng overlay network.

Khi bắt đầu Docker, default bridge network được tạo tự động, các container mới kết nối tới bridge network trừ khi có chỉ định khác. Bạn cũng có thể tạo custom bridge network. 
#### Difference between user-defined bridge and default bridge 
- User-defined bridges provide better isolation and interoperability between containerized applications
- User-defined bridges provide automatic DNS resolution between containers
- Containers can be attached and detached from user-defined networks on the fly
- Each user-defined network creates a configurable bridge
- Linked containers on the default bridge network share environment variables

#### Manager a user-defined bridge
- Create a user-defined bridge network
```bash
$ docker network create my-net
```
- Specify subnet, IP address range, gateway, other options
```bash
$ docker network create --help
```
- Remove a user-defined bridge network
```
$ docker network rm my-net
```

#### Connect a container to a user-defined bridge
- Specify one or more `--network` flags when create a new container: ngnix container connect `my-net` network, public port 80 in the container to port 8080 on Docker host.
```
$ docker create --name my-nginx \
  --network my-net \
  --publish 8080:80 \
  nginx:latest
```
- To connect running container to user-defined bridge
```
$ docker network connect my-net my-nginx
```
#### Disconnect a container from a user-defined bridge
```
$ docker network disconnect my-net my-nginx
```
### Host Networking
Nếu bạn sử dụng `host` network driver cho container, container's network stack không tách biệt với Docker host. Ví dụ, nếu chạy container liên kết port 80 và sử dụng `host` networking, ứng dụng trong container sẽ cho phép port 80 trên địa chỉ IP host.

Host networking driver chỉ chạy trên Linux host, không hỗ trợ Docker với Mac, Docker Window hoặc Docker EE cho Windows Server.

Với Docker 17.06 hay cao hơn, bạn có thể sử dụng `host` network cho swarm service, bằng cách truyền tham số `--network host` trong lệnh `docker container create`. Trong trường hợp này, control traffic(traffic manage swarm và service) gửi thông qua overlay network nhưng swarm service container gửi dữ liệu sử dụng Docker daemon host network và port. Ví dụ nếu một service container liên kết với port 80, chỉ một service container có thể chạy trên swarm node. 
