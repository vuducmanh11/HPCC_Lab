## Tìm hiểu cơ bản về OpenStack

### Tổng quan về OpenStack

* OpenStack là một nền tảng điện toán đám mây mã nguồn mở cung cấp tất cả các môi trường cloud. OpenStack nhắm tới việc triển khai đơn giản, khả năng mở rộng lớn và có nhiều tính năng. OpenStack được các chuyên gia cloud computing trên toàn cầu tham gia xây dựng
* OpenStack cung cấp giải pháp Infrastructure-as-a-Service (IaaS) qua nhiều dịch vụ bổ sung. Mỗi dịch vụ cung cấp một giao diện lập trình ứng dụng (API) tạo điều kiện cho sự tích hợp này
  * **IaaS** là một mô hình cung cấp trong đó một tổ chức thuê các thành phần vật lý của dữ liệu trung tâm, như lưu trữ, phần cứng, các máy chủ và các thành phần mạng. Một dịch vụ câng cấp sở hữu trang thiết bị và chịu trách nhiệm cho việc lưu trữ, vận hành và bảo trì nó. Khách hàng thường trả phí theo mỗi người dùng cơ bản. IaaS là một mô hình cung cấp các dịch vụ đám mây.
  * **API** : một tập hợp các chỉ định được sử dụng để truy cập một dịch vụ, ứng dụng hay chương trình. Bao gồm các lệnh gọi dịch vụ, tham số yêu cầu cho mỗi lệnh gọi và các tham số trả về mong muốn

### Các core service trong Openstack

* Identity service
* Image service
* Compute service
* Networking service
* Dashboard
* Block Storage service

### Chức năng của các service

* Identity service (keystone)
  * Dịch vụ định danh OpenStack cung cấp một điểm tích hợp duy nhất để quản lý việc xác thực, cho phép và mục lục các dịch vụ
  * Dịch vụ định danh thường là dịch vụ đầu tiên tương tác với một người dùng. Một lần xác thực, người dùng cuối có thể sử dụng định danh của họ để truy cập các dịch vu OpenStack khác. Tương tự như vậy, các dịch vụ khác sử dụng dịch vụ định danh để đảm bảo người dùng nói họ là ai để khám phá các dịch vụ khác đang trong quá trình triển khai. Dịch vụ định danh cũng có thể tương tác với một số hệ thống quản lý người dùng bên ngoài
  * Người dùng và dịch vụ có thể xác định các dịch vụ khác bằng cách sử dụng mục lục các dịch vụ được quản lý bởi dịch vụ định danh. Như tên  của nó, mục lục dịch vụ là danh sách các dịch vụ có sẵn trong triển khai OpenStack. Mỗi dịch vụ có thể có một hoặc nhiều điểm cuối và mỗi điểm cuối có thể là 1 trong 3 kiểu: admin, internal hoặc public. Trong 1 môi trường sản xuất, sự khác nhau giữa kiểu các điểm cuối có thể nằm trên các mạng riêng biệt được hiển thị với các loại người dùng khác nhau vì lí do bảo mật. Ví dụ, public API network có thể nhìn thấy từ Internet vì vậy khách hàng có thể quản lý cloud của họ. admin API network có thể bị hạn chế để tính toán trong tổ chức được quản lý bởi cơ sở hạ tầng Cloud. Internal API network có thể bị hạn chế tứi máy chủ chứa dịch vụ OpenStack. Openstack cũng cung cấp nhiều vùng cho việc mở rộng.
* Image service (Glance)
  * OpenStack Image service là trung tâm của IaaS: Infrastructure-as-a-Service. Nó chấp nhận yêu cầu API cho disk images hoặc server images, và các định nghĩa siêu dữ liệu từ người dùng cuối hoặc các thành phần OpenStack Compute. Nó cũng hỗ trợ lưu trữ trên disk hoặc server images theo các kiểu kho khác nhau bảo gồm OpenStack Object Storage. 
  * Số lượng các tiến trình định kì chạy trên OpenStack Image service để hỗ trợ caching. Replication service( dịch vụ nhân rộng) đảm bảo tính nhất quán và khả dụng thông qua các cluster. Một số tiến trình định kỳ khác bao gồm auditor, updater và reaper
* Compute service (Nova)
  * Sử dụng OpenStack Compute để lưu trữ và quản lý hệ thống cloud computing. OpenStack Compute là một phần lớn của hệ thống IaaS
  * OpenStack Compute tương tác với OpenStack Identity để xác thực, OpenStack Image cho disk và server image; với OpenStack dashboard cho giao diện của người dùng và quản trị. Image  bị truy cập giới bạn bởi các proejct và bởi người dùng; quotas là giới hạn trên mỗi project (số thể hiện). OpenStack Compute có thể mở rộng theo chiều ngang trên standard hardware và tải xuống image để khởi chạy thể hiện
* Networking service (Neutron)
  * OpenStack Networking cho phép bạn tạo và đính kèm giao diện thiết bị được quản lý bởi dịch vụ OpenStack khác tới các mạng. Plug-ins có thể được triển khai để phù hợp vói các thiết bị và phần mềm khác nhau, cung cấp sự mềm dẻo cho kiến trúc OpenStack và quá trình triển khai
  * Bao gồm các components
    * neutron-server: cho phép và định tuyến các API request tới các OpenStack Networking plug-in thích hợp cho hành động xác định
    * OpenStack Networking plug-ins and agents: kết nối và ngắt kết nối với các cổng, tạo mạng hoăc mạng con và cung cấp địa chỉ IP. Các plug-ins và tác nhân khác nhau phụ thuộc vào nhà cung cấp và công nghệ được sử dụng trong cloud cụ thể
    * Messaging queue: hầu hết cài đặt OpenStack Networking đều sử dụng để định tuyến thông tin giữa neutron-server và các tác nhân khác nhau. Cũng hoạt động như 1 cơ sở dữ liệu để lưu trữ trạng thái mạng cho các plug-ins cụ thể

* Dashboard (Horizon)
  * Dashboard là một giao diện web cho phép người quản trị cloud và người dùng quản lý dịch vụ và tài nguyên OpenStack khác nhau
* Block Storage service (Cinder)
  * Block Storage service thêm lưu trữ liên thục vào 1 máy ảo. Block Storage cung cấp cơ sở hạ tầng cho việc quản lý các volume, tương tác với OpenStack Compute để cung cấp các volume cho các thực thể. Service này cũng cho phép quản lý snapshot và kiểu của các volume