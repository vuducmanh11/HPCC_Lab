# Ansible
## About Ansible
Ansible là một IT automation tool. Nó có thể cấu hình hệ thống, triển khai phần mềm và orchestrate nhiều task IT phức tạp như các triển khai liên tục hoặc giảm thời gian update xuống 0.

Mục đích chính của Ansible là đơn giản và dễ dàng sử dụng. Nó cũng tập trung mạnh mẽ vào security và reliability, có tối thiểu bộ phận chuyển động, sử dụng OpenSSH cho giao vận (với cách giao vận và pull khác để thay thế), và một ngôn ngữ được thiết kế  xung quanh auditability bởi humans- những người không quen với chương trình.

Chúng tôi tin tưởng đơn giản thích hợp với mọi kích thước của môi trường, vì vậy chúng tôi thiết kế cho những người dùng bận rộn: developer, sysadmin, release engineer, IT manager và mọi người ở giữa. Ansible thích hợp cho quản lý tất cả môi trường từ thiết lập nhỏ với số ít các instance tới môi trường doanh nghiệp với hàng nghìn instance.

Ansible quản lý các máy ảo trong một agent-less manner. Không bao giờ có câu hoi làm thế nào để upgrade remote daemon hoặc vấn đề không thể quản lý hệ thống vì các daemon đã được uninstall. Vì OpenSSH là một trong những thành phần mã nguồn mở peer-reviewed, tiếp xúc an toàn giảm đáng kể. Ansible phân cấp nó dựa trên thông tin xác thực hệ điều hành đã có của bạn để điều khiển truy cập từ remote machine. Nếu cần thiết, Ansible có thể dễ dàng kết nối tới Kerberos, LDAP và các hệ thống quản lý xác thực tập trung khác. 

Tài liệu này bao quát phiên bản hiện tại của Ansible (2.5) và một số feature của bản đang phát triển. Với các thuộc tính gần đây, chúng tôi note trong mỗi section giá trị version Ansible mà thuộc tính đã được thêm.

Ansible phát hành một bản phát hành release gần 2 tháng một lần. Ứng dụng core được cải tiến cẩn trọng, đơn giản trong ngôn ngữ thiết kế và thiết lập. Tuy nhiên, cộng đồng xung quanh các module mới và plugin được phát triển và đóng góp rất nhanh, bổ sung nhiều module mới trong mỗi bản phát hành. 
## Installation 
Ta chỉ cần cài đặt ansible và python trên host machine. Mặc định Ansible sử dụng trình thông dịch python tại /usr/bin/python để chạy Ansible. Command to installing

```bash
$ sudo apt-get update
$ sudo apt-get install software-properties-common
$ sudo apt-add-repository ppa:ansible/ansible
$ sudo apt-get update
$ sudo apt-get install ansible
```
## Using Ansible
### Ad-Hoc Commands 
What's an ad-hoc command? Ad-Hoc Commands là các câu lệnh nhanh bạn muốn thực hiện với Ansible trên các host chỉ định nhưng không muốn lưu lại để dùng sau đó. 
```
ansible <hostname> -a -m ping -u <username> -k 
```
### Playbook 
Playbook là file cấu hình của Ansible, triển khai và orchestration language. Playbook mô tả chính sách bạn muốn thi hành trên remote system hoặc thiết lập các bước trong một quá trình IT chung.

Ở basic level, playbook có thể sử dụng để quản lý cấu hình cho việc triển khai remote machine. Tại các mức cao hơn bạn có thể sắp xếp nhiều tầng để cập nhật trao quyền cho host khác tương tác với server giám sát và cân bằng tải.

Playbook được thiết kế để người quản trị dễ đọc và sử dụng như một ngôn ngữ text cơ bản. Có nhiều cách để tổ chức playbook và các file nó bao gồm được giới thiệu [suggestion](https://docs.ansible.com/ansible/latest/user_guide/playbooks.html#working-with-playbooks)
### Inventory file
Ansible hoạt động với nhiều hệ thống trong kiến trúc hạ tầng cùng một lúc. Nó thực hiện việc này bằng cách chọn các phần của hệ thống được liệt kê trong Ansible's inventory, mặc định được lưu tại ```/etc/ansible/hosts```. Bạn có thể chỉ định một inventory file khác bằng cách sử dụng tuỳ chọn ```-i <path>``` trên command line.

Không chỉ cấu hình inventory này mà có thể sử dụng nhiều inventory file tại cùng thời điểm và pull inventory từ dynamic hoặc cloud source hay dạng format khác (YAML, ini, etc) được mô tả trong *Working With Dynamic Inventory*. Giới thiệu trong version 2.4, Ansible có inventory plugins làm nó linh hoạt và có thể tuỳ chỉnh.

#### Hosts and Groups

Inventory file có thể là một trong nhiều định dạng, phụ thuộc và inventory plugins bạn có. Với ví dụ, định dạnh của ```/etc/ansible/hosts``` là INI-like (one of Ansible's defaults) và trông như thế này:
```
mail.example.com

[webservers]
foo.example.com
bar.example.com

[dbservers]
one.example.com
two.example.com
three.example.com
```
Mở đầu trong ngoặc vuông là tên nhóm, được sử dụng trong phân loại hệ thống và quyết định hệ thống nào điều khiển tại thời điểm nào và cho mục đích nào.
YAML version would look like:
```
all:
  hosts:
    mail.example.com:
  children:
    webservers:
      hosts:
        foo.example.com:
        bar.example.com:
    dbservers:
      hosts:
        one.example.com:
        two.example.com:
        three.example.com:
```
Điều đó OK để hệ thống trong nhiều hơn một group, ví dụ một server cloud có thể là cả một webservẻ và một dbserver. Nếu bạn làm, chú ý rằng các biến sẽ đến từ tất cả các nhóm mà chúng là thành viên. Biến ưu tiên được mô tả chi tiết trong chapter sau.
Nếu bạn có các máy chủ chạy trên các cổng SSH không chuẩn bạn có thể đặt số của cổng sau hostname với một dấu hai chấm. Các cổng được liệt kê trong cấu hình SSH của bạn sẽ không được sử dụng với *paramiko* connection nhưng sẽ được sử dụng trong kết nối *openssh*.

Để khiến mọi thứ rõ ràng, nó được đề xuất rằng bạn sẽ thiết lập chúng nếu nhiều thứ không chạy trên default port:
```
badwolf.example.com:5309
```
Giả sử bạn vừa mới có địa chỉ IP tĩnh và bạn muốn thiết lập một số alias trong file host của bạn hoặc bạn đang kết nối thông qua tunnels. Bạn cũng mô tả các host thông qua các biến:
In INI:
```
jumper ansible_port=5555 ansible_host=192.0.2.50
```
In YAML:
```
...
  hosts:
    jumper:
      ansible_port: 5555
      ansible_host: 192.0.2.50
```
Trong ví dụ trên, cố gắng chống lại host alias "jumper" (có thể thậm chí là một hostname thực sự) sẽ liên lạc với 192.0.2.50 trên port 5555. Chú ý rằng sử dụng thuộc tính của inventory file để định nghĩa một số biến đặc biệt. 

Nếu bạn thêm nhiều host trong cùng một mẫu bạn có thể làm như sau hơn là liệt kê từng host name:
```
[webservers]
www[01:50].example.com
```
Với mẫu số, số 0 đứng đầu có thể có hoặc bỏ, như mong muốn. Phạm vi bao gồm. Bạn có thể định nghĩa khoảng chữ:
```
[database]
db-[a:f].example.com
```
Bạn cũng có thể chọn kiểu kết nối và người dùng trên mỗi host cơ bản:
```
[targets]
localhost              ansible_connection=local
other1.example.com     ansible_connection=ssh        ansible_user=mpdehaan
other2.example.com     ansible_connection=ssh        ansible_user=mdehaan
```
Như đề cập bên trên, thiết lập chúng trong inventory file chỉ là một cách viết tắt, chúng ta sẽ thảo luận làm thế nào để lưu trữ chúng trong các file cá nhân trong thư mục *'host_vars'* sau này.

#### Host Variables
Như mô tả bên trên, dễ dàng để chỉ định biến cho các host, sẽ được sử dụng sau trong playbooks:
```
[atlanta]
host1 http_port=80 maxRequestsPerChild=808
host2 http_port=303 maxRequestsPerChild=909
```
#### Group Variables
Biến cũng có thể được cấp cho tất cả group một lúc:
The INI way:
```
[atlanta]
host1
host2

[atlanta:vars]
ntp_server=ntp.atlanta.example.com
proxy=proxy.atlanta.example.com
```
The YAML version:
```
atlanta:
  hosts:
    host1:
    host2:
  vars:
    ntp_server: ntp.atlanta.example.com
    proxy: proxy.atlanta.example.com
```
Hãy nhận biết chỉ có một cách tiện lợi để cung cấp biến cho nhiều host cùng lúc, thậm chí bạn có thể nhóm các host bởi group, các biến luôn được **flattened** tới level host trước khi một kịch bản được thực hiện.

#### Groups of Groups, and Group Variables
Có thể tạo groups of groups bằng cách sử dụng hậu tố ```:children``` trong INI và trường ```children:``` trong YAML. Bạn có thể áp dụng các biến sử dụng ```:vars``` hoặc ```vars:```:
```
[atlanta]
host1
host2

[raleigh]
host2
host3

[southeast:children]
atlanta
raleigh

[southeast:vars]
some_server=foo.southeast.example.com
halon_system_timeout=30
self_destruct_countdown=60
escape_pods=2

[usa:children]
southeast
northeast
southwest
northwest
```

YAML:
```
all:
  children:
    usa:
      children:
        southeast:
          children:
            atlanta:
              hosts:
                host1:
                host2:
            raleigh:
              hosts:
                host2:
                host3:
          vars:
            some_server: foo.southeast.example.com
            halon_system_timeout: 30
            self_destruct_countdown: 60
            escape_pods: 2
        northeast:
        northwest:
        southwest:
```
Nếu bạn cần lưu trữ danh sách hoặc dữ liệu hash hay thích hơn giữ các host và biến nhóm chỉ định tách biệt với inventoru file, xem phần sau. Nhóm con có một cặp thuộc tính để chú ý:
- Bất kỳ hostnào là thành viên của group con tự động là thành viên của group cha.
- Biến của nhóm con sẽ có độ ưu tiên cao hơn (override) biến của nhóm cha
- Các nhóm có thể có nhiều nhóm cha và nhóm con, nhưng không có quan hệ vòng tròn
- Hosts có thể trong nhiều nhóm, nhưng chúng sẽ chỉ là một instance của host, hợp nhất dữ liệu từ nhiều nhóm.

#### Default groups
Có hai nhóm mặc định ```all``` và ```ungrouped```. ```all``` chứa mọi host. ```ungrouped``` chứa tất cả các host không có nhóm nào khác ngoài ```all```. Mọi host sẽ luôn thuộc ít nhất 2 groups. Tuy ```all``` và ```ungrouped``` luôn có mặt nhưng chúng có thể ngầm hoặc không xuất hiện trong danh sách nhóm ```group_names```.

#### Splitting Out Host and Group Specific Data 
Phương pháp được ưu tiên trong Ansible là không lưu trữ biến trong main inventory file.
Bổ sung để lưu trữ trực tiếp trong inventory file, biến host và group có thể được lưu trữ trong các file cá nhân liên quan với inventory file (not directory, luôn là file).
Các biến file trong định dạng YAML. Phần mở rông file đúng bao gồm '.yml', '.yaml', '.json' hoặc không có phần mở rộng file. 
Giả sử inventory file path: ```etc/ansible/hosts```
Nếu host tên 'foosball' trong nhóm 'raleigh' và 'webservers', các biến trong file YAML tại các vị trí sau được thiết lập cho host:
```
/etc/ansible/group_vars/raleigh #can optionally end in '.yml', '.yaml' or '.json'
/etc/ansible/group_vars/webservers
/etc/ansibel/host_vars/foosball
```
Ví dụ, giả sử bạn có các host được nhóm bởi datacenter, mỗi datacenter sử dụng một số servers khác nhau. Dữ liệu trong groupfile *'/etc/ansible/group_vars/raleigh'* cho nhóm *'raleigh'* có thể trông như:
```
---
ntp_server: acme.example.org
database_server: storage.example.org
```
Nó sẽ OK nếu các file chưa tồn tại, vì đây là một tính năng tuỳ chọn.
Như một trường hợp nâng cao, bạn có thể tạo tên *directories* sau khi nhóm của bạn hoặc hosts, và Ansible sẽ đọc tất cả các file trong các thư mục theo thứ tự từ điển. Một ví dụ nhóm 'raleigh':
```
/etc/ansible/group_vars/raleigh/db_settings
/etc/ansible/group_vars/raleigh/cluster_settings
```
Tất cả các host trong nhóm 'raleigh' sẽ được định nghĩa biến trong các file có sẵn cho chúng. Điều này có thể sử dụng để giữ các biến của bạn tổ chức,khi một file start quá lớn hoặc khi bạn muốn dùng **Ansible Vault** trên một phần của biến group.
TIP: Thư mục ```group_vars/`` và ```host_vars``` có thể đã tồn tại trong thư mục playbook hoặc thư mục inventory. Nếu tất cả path đã tồn tại, biến trong thư mục playbook sẽ ghi đè giá trị trong thư mục inventory.
TIP: Giữ file inventory của bạn và các biến trong git repo (hoặc other version control) là một cách tuyệt vời để theo dõi thay đổi trong inventory của bạn và biến host.

#### How Variables Are Merged
Mặc định các biến được hợp nhất (merged/flattened) để chỉ định host trước khi kịch bản được chạy. Điều này giữ Ansible tập trung trên Host và Task, vì vậy các nhóm không thực sự tồn tại bên ngoài inventory và host matching. Mặc định, Ansible ghi đè các biến bao gồm những cái được xác định cho 1 nhóm hoặc một host (see the hast_merge setting to change this). Độ ưu tiên từ thấp tới cao:
- all group (vì là cha của tất cả nhóm)
- parent group
- child group
- host 
Khi nhóm cùng mức parent/child được merged, nó theo thư tự bảng chữ cái và nhóm cuối cùng ghi đè các nhóm trước. Ví dụ, một a_group sẽ merged với b_group và b_group vars sẽ ghi đè lên a_group.
*New in version 2.4*
Bắt đầu từ Ansible version 2.4, người dùng có thể sử dụng biến nhóm ```ansible_group_priority``` để thay đổi thứ tự merged, thứ tự ưu tiên. Giá trị mặc định là 1 nếu không set. VÍ dụ:
```
a_group:
    testvar: a
    ansible_group_priority: 10
b_group
    testvar: b
```
Trong ví dụ trên, các group đều cùng thứ tự ưu tiên, kết quả sẽ ```testvar == b```, nhưng vì ```a_group``` có chỉ số ưu tiên cao hơn nên ```testvar == a```
