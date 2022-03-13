# AWS Windows Infrastructure 
## Lời Nói Đầu: 

 - Tất cả các bài lab đều được thực hiện dựa theo giải pháp của AWS về
   xây dựng hệ thống mạng đạt chuẩn.
 - Link giải pháp AWS: [Giải Pháp Windows Trên
   AWS](https://aws-labs.net/winlab0-buildinfra.html)
 - Các bài lab/giải pháp sẽ dịch ngược sơ đồ hệ thống thành step by step
   để giải trình cách làm
 - Nếu như muốn tham khảo file Infrastructure as Code (IaC) thì có thể
   truy cập đường dẫn đã cung cấp phía trên khi thực hiện xong. 
 - Xin lưu ý hãy xóa hết tất cả những dữ liệu đã làm để tránh chi phí
   AWS không  cần thiết. 
 - Đây sẽ là những bài lab/giải pháp típ nối nhau thành 1 chuỗi các bài
   lab nhằm cung cấp đầy đủ nhất có thể được về việc triển khai hệ thống
   Windows trên AWS

## Xây Dựng Hệ Thống Mạng Windows Trên AWS

 **I. Tổng quan sơ đồ mạng**
 
![Mô hình mạng AWS](https://github.com/minhhung1706/AWS-Windows-Infrastructure/blob/d80492df28586e7b4e1519ee79f4b4d4511b02c0/images/Network-Diagram.png)

Nhìn vào sơ đồ mạng này, chúng ta sẽ thấy những thành phần sau:

 - AWS Cloud: đây là ý chỉ về dịch vụ cloud AWS
 - Availability Zone 1 & 2: Trong AWS, chúng ta hay gọi đây là AZ, tức
   là vùng có thể thực hiện việc xây dựng AWS. Có nhiều vùng khác nhau.
   Thí dụ như ở VN, thì gần nhất là có - **Asia Pacific  (Singapore)
   ap-southeast-1** và  -  **Asia Pacific  (Sydney) ap-southeast-2**. Chính
   xác hơn thì đây là 2 region. Trong 2 region này sẽ có các AZ. Các AZ
   có thể hiểu nôm na như là 1 trung tâm dữ liệu (datacentre) vật lý của
   AWS. 
 - Vậy thì dựa vào giải thích trên, ta có thể hiểu rằng AWS hiện tại có
   vùng sử dụng được là Singapore và trong đó có 1 datacentre của AWS.
 - Thứ tự 1 và 2 ở trên không mang lên ý nghĩa gì khác ngoài việc đánh
   số thứ tự của AWS. Nó không có nghĩa là ở Sing có 1 DC và ở Úc có 2
   DC. 
 - Hay nói cách khác, AWS phân rộng hơn các region (vùng), thì dựa vào 2
   thí dụ trên, ta có thể hiểu được 1 cách rộng ra rằng AWS đánh số thứ
   tự cho vùng lớn là Châu Á - Thái Bình Dương. Trong vùng lớn này có 2
   (thí dụ) DC đánh số thứ tự để dễ quản lý lần lượt là
   **ap-southeast-1** và **ap-southeast-2**
 - Vậy thì phân vùng trong kiến trúc có quan trọng hay không. Câu trả lời là có. Vì nó thể hiện được sự bền vững trong việc xây dựng giải pháp. Thí dụ như on-premise, các bạn xây dựng hệ thống chúng ta thường hay có fail-over và load-balancing nằm trên 2 dải ip để chạy song song. Thì AWS cũng vậy, hãy hiểu rằng AZ nó cũng như việc chúng ta phân dảy IP để thực hiện công việc load balancing và fail-over. 
 - Ngoài ra, khi thực hiện đến việc Active Directory, AWS sẽ bắt buộc  chúng ta thiết kế đúng chuẩn là phải có ít nhất 2 AZ để triển khai AWS Mamaged AD. Hoặc chúng ta có thể tham khảo AD Connector của AWS để triển khai giải pháp Hybird AD : On-premise <=> AWS. Lưu ý là vẫn phải trả phí cho AD Connector. 
 - VPC - Virtual Private Cloud: Theo lý thuyết của AWS, đây là nơi mà sẽ **logically separate your infrastructure**. Có nghĩa là nó như 1 cái DMZ ảo, bao quát toàn bộ kiến trúc của bạn. Và trong này, bạn có thể thiết lập các policy nhằm chặn/mở các cổng/giao thức cần thiết cho kiến trúc. Mặc định, VPC sẽ chặn mọi truy cập từ trong ra và từ ngoài vào. Nó tương tự như bạn setup software firewall. Khi set-up xong, cũng sẽ mặc định chặn hết. Bạn phải configure hợp lý để có thể tiến hành triển khai hệ thống dịch vụ.
 - Internet Gateway (IGW): thành phần này có nhiệm vụ kết nối 1 dãy ip của kiến trúc ra ngoài internet và biến nó thành Public IP Range. Tất cả các dịch vụ nằm trong dảy Public IP này sẽ kết nối đc với Internet
 - Public Subnet: Đây là dãy IP public dùng để ra internet của toàn bộ kiến trúc hệ thống. Để public được dãy IP này, bạn bắt buộc phải liên kết (associate) nó với IGW. Vì như đã giải trình ở trên, mặc định VPC là private. 
 - Private Subnet: Đây là dãy private IP. Mặc định khi tạo xong 1 dãy IP trong VPC, nó sẽ ở trạng thái private, trừ khi gắn vào IGW thì sẽ thành public ip.
 - NAT Gateway: Dùng để khai thông giao típ giữa các dịch vụ trong vùng: Private IP <=> Public IP
 - Auto Scaling Group (ASG): Đây là 1 tính năng rất hay mà khó lòng triển khai ở on-primes. Nó cho phép chúng ta cấu hình việc thêm máy chủ cho việc cân bằng tải vào giờ cao điểm, và giảm đi khi vào lúc bình thường nhằm tiết kiệm chi phí. On-premise, khi thực hiện việc này, xác định là sẽ rất tốn kém, vì chún ta không thể thêm hoặc bớt lượn máy chủ vật lý / ảo. Nhưng cái này làm được trên AWS nhờ vào ASG. 
 - Remote Desktop Gateway: đây là 1 bastion host dùng để truy cập từ nơi làm việc đến kiến trúc AWS của chúng ta. Còn gọi cách khác là jump-host. 
 
**II. Bắt đầu xây dựng hệ thống mạng**

**1. Cấu hình IAM Role:** 

AWS quản lý phân quyền rất chặt chẽ. Vì vậy, mỗi 1 resource trên AWS sẽ có những quyền hạn nhất định từ Denied All Access cho đến Allowed All Access. Ngoài ra, chúng ta cũng có thể dùng IAM để gán 1 số quyền nhất định nhằm thỏa mãn việc thiết kế architect best practices mà AWS đề ra: Least Privilege. Có nghĩa là phân quyền sao cho vừa đủ để thực hiện tác vụ ==> nhằm nâng cao độ an toàn cho toàn bộ hệ thống. 
- Vào AWS Management Console 
- Chọn IAM Role
![iam](images/iam-1.jpg)
- Chọn Roles => Create Role (góc phải trên)
![iam](images/iam-2.jpg)
- Chọn như hình => NEXT
![iam](images/iam-3.jpg)
- Chúng ta sẽ tìm và add các permission vào roles đang thực hiện. Lưu ý là khi tìm được 1 dịch vụ, bấm chọn, sau đó phải bấm X để Permission đó và tìm cái khác. Nếu không bấm X để xóa thì sẽ không tìm ra. Tương tự sẽ add tất cả các Permission như sau vào:
![iam](images/iam-4.jpg)
- Sẽ có tổng cộng là 5 Permissions như hình dưới. Sau đó bấm NEXT
![iam](images/iam-5.jpg)
- Sau đó sẽ là review lại roles, điền tên, thêm tag. Tùy chỉnh tùy ý. Sau cùng xuống dưới bấm Create. Vậy là đã có 1 Role bao gồm các Permissions đầy đủ để thực hiện công việc join domain.
![iam](images/iam-6.jpg)

**2. Thiết kế AWS Network** 

2.1: Tạo VPC
- Vào AWS Management Console, tìm VPC => Tại giao diện VPC => Create VPC
![vpc](images/vpc-1.jpg)
- Tạo 1 VPC như hình. Sau đó bấm Create
![vpc](images/vpc-2.jpg)
- Tại giao diện VPC vừa tạo, góc phải trên chúng ta bung thẻ Action ra => Edit DNS Host Name => Check Enable DNS Host Name => Save Change. 
- Làm tương tự cho DNS Resolution => Để phân giải tên domain/ip của chúng ta trên môi trường network
![vpc](images/vpc-3.jpg)


2.2: Tạo Subnet
- Tại VPC Management Console => Góc trái sẽ có những option => chọn Subnet => Create Subnet
![subnet](images/subnet-1.jpg)
- Tạo subnet như hình => Create subnet
- Các bạn nên vào trang web này [Tự Động Chia Subnet](https://www.davidc.net/sites/default/subnets/subnets.html) để phân chia subnet cho hiệu quả, tránh sai sót
![subnet](images/subnet-2.jpg)
- Tương tự như vậy, chúng ta sẽ có tổng cộng 4 Subnets trải đều trên 2 AZ. Theo hình mình làm sẽ là ap-southeast-2a và ap-southeast-2b. 
- Lần lượt sẽ là 1 public và 1 private trên mỗi AZ
- Tên subnets các bạn có thể đặt theo ý, miễn sao trực quan, dễ phân biệt và dễ thực hiện công việc là được. Nó hoàng toàn không có ý nghĩa gì và không liên quan hay ảnh hưởng gì đến môi trường network. 
![subnet](images/subnet-3.jpg)

2.3: Tạo Internet Gateway
- Như đã giải trình ở trên, 1 network trên AWS nằm trong VPC sẽ mặc định là private network, không thể ra internet được. Vì lẽ đó mà chúng ta cần phải có Internet Gateway để route traffic ra/vào 1 network chỉ định để làm thành Public Subnet.
- Tại VPC Management Console => Góc trái sẽ có những option => chọn Internet Gateway => Create Internet Gateway (IGW)
![internet-gateway](images/igw-1.jpg)
- Sau đó tạo 1 IGW => Create Internet Gateway
![internet-gateway](images/igw-2.jpg)
- Attach IGW to VPC
![internet-gateway](images/igw-3.jpg)
- Chọn VPC của bài lab này
![internet-gateway](images/igw-4.jpg)

2.4: Tạo Route Table
- Tại VPC Management Console => Góc trái sẽ có những option => chọn Route Table => Đã có 1 route có sẵn đó là do khi chúng ta tạo VPC thì SDN (Software Defined Network) của AWS sẽ định hình 1 route cho chúng ta. 
- Nên thực hiện việc đổi tên route này sao cho trực quan và dễ hiểu. Cách đổi tên thực hiện như hình dưới
![route](images/rt-1.jpg)
- Tại route này => chọn Edit route 
![route](images/rt-2.jpg)
- Chúng ta sẽ gắn vào IGW và route là 0.0.0.0/0 để đi ra internet => Save Changes
![route](images/rt-3.jpg)
- Chúng ta cũng chọn route này => Action => Edit Sunet Association => Gắn vào 2  subnet đã tạo để chuyển hóa 2 subnets này thành 2 public subnets => Save Associations
![route](images/rt-4.jpg)
- Tại phần Route Table Management Console => Góc phải => Create Route table => Tạo 1 private route table
![route](images/rt-5.jpg)
- Chọn route table vừa mới tạo => Edit Subnet Association => Add 2 subnet còn lại vào => sẽ tạo thành 2 private subnet => Save Associations
![route](images/rt-6.jpg)
![route](images/rt-7.jpg)

2.5: Tạo NAT Gateway
- Để cho private subnet ra được internet. Chúng ta cần phải có 1 NAT Gateway. NAT Gateway này sẽ nằm ở phân vùng public subnet
- Tương tự như route table và internet gateway. Chúng ta chọn NAT Gateway ở menu bên trái => Create NAT Gateway
- Lưu ý là phải Allocate Elastic IP cho NAT Gateway
![nat-gateway](images/nat-gw-1.jpg)
- NAT Gateway khởi tạo sẽ mất vài phút. Chúng ta đợi đến khi trạng thái từ Pending => Available là có thể sử dụng được. 
- Trở về Route Table => chọn Private Route => Edit Route
![nat-gateway](images/nat-gw-2.jpg)
- Route 0.0.0.0/0 , gắn vào NAT Gateway để ra internet cho Private Subnet => Save Changes
![nat-gateway](images/nat-gw-3.jpg)

**3. Thiết kế AWS Managed Active Directory Service (AWS Managed AD)** 
- AWS Management Console => Directory Service => Setup Directory
![aws-ad](images/aws-ad-1.jpg)
- Chọn như hình => NEXT
![aws-ad](images/aws-ad-2.jpg)
- Standard Edition, điền tên domain, password. Lưu ý là mặc định username của AWS Managed AD sẽ là Admin. Nên chúng ta chỉ cần setup password
![aws-ad](images/aws-ad-3.jpg)
- Chọn VPC và AZ. Lưu ý là AWS Managed AD bắt buộc phải có 2 AZ thì mới có thể khởi tạo được. Và nên tạo trong Private Subnet
![aws-ad](images/aws-ad-4.jpg)
- Chúng ta check lại thông tin và bấm Create Directory
![aws-ad](images/aws-ad-5.jpg)
- Sẽ mất khá lâu tầm 40 phút để cho AWS tạo các dịch vụ nền và promote AD.
- Chúng ta sẽ kiểm tra lại để xem AD có vào trạng thái Active hay chưa. Nếu trạng thái là Active thì có nghĩa là đã khởi tạo thành công. Một số trường hợp do underlied-host của AWS có vấn đề, nên có thể trạng thái AD sẽ là Failed. Nên kiểm tra chắc chắn lại.
![aws-ad](images/aws-ad-6.jpg)

**4. Thiết kế AWS EC2 và quản lý AWS Managed AD** 
- Tại AWS Management Console => EC2 => Launch Instances
![ec2](images/ec2-1.jpg)
- Chọn Windows Server 2022
![ec2](images/ec2-2.jpg)
- chọn t2.micro
![ec2](images/ec2-3.jpg)
- Tinh chỉnh setting cho EC2 như hình. Lưu ý là chúng ta sẽ phân tác vụ cho EC2 nên cần chọn chính xác các option. Theo hình thì EC2 này sẽ thực hiện tác vụ như 1 bastion-host (jump-host). Vì vậy sẽ ở Public Subnet, nhận  Public ip, joined domain và có IAM Role 
![ec2](images/ec2-4.jpg)
- Sau đó sẽ add storage => add theo tùy ý. Quan trọng là xong lab thì nhớ phải xóa hết tất cả resources
![ec2](images/ec2-5.jpg)
- Add Tag. Không nên bỏ qua phần này vì về sau system thật tế sẽ dùng Tag: Name-Value để quản lý, đặc biệt là việc thanh toán bill AWS. Ngoài ra thì khi có Tag, EC2 của bạn sẽ dễ dàng phân biệt và trực quan hơn.
![ec2](images/ec2-6.jpg)
- Sau đó là sẽ tạo Security Group (SG). Phục vụ cho mục đích bài lab. Chúng ta sẽ tạo SG đơn giản bằng cách All Traffic => Allow All. Nhưng đây KHÔNG phải là Security Best Practices. Khi môi trường công việc, nên thiết kế SG sao cho vừa đủ cần thiết xài là được. 
![ec2](images/ec2-7.jpg)
- Sau đó là Review & Launch. Tại bước này, AWS sẽ hỏi bạn việc khởi tạo public/private key cho EC2. Hình dưới minh họa việc đã có key và xài lại thì phải check dòng "Acknowledgement ...." 
![ec2](images/ec2-8.jpg)
- Minh họa cho việc tạo mới key. Thì bạn phài save key mới tạo về local thì sẽ xài được. Sau đó Launch Instance
![ec2](images/ec2-9.jpg)
- Tạo thêm 1 EC2 nữa như trên. Nhưng lưu ý những option sau: 

=> chọn Instance loại 2x.large: Lý do là EC2 này sẽ đảm nhiệm 1 số tác vụ cần đến sức mạnh phần cứng và tốc độ mạng tốt. Nếu chọn t2.micro thì sẽ xử lý khá lâu và có thể bị đơ trong quá trình làm
=> Private Subnet

=> Auto Assigned Public IP: DISABLE

=> chọn IAM Roles và AD đã tạo

=> Tag: Để tên là AD-Manager hoặc tên gì tùy ý bạn. Tên hoàng toàn không ảnh hưởng gì nhưng nó thể hiện trực quan chức năng của EC2. MIễn sao bạn không nhầm là được
![ec2](images/ec2-10.jpg)

=> Security Group: Chọn SG vừa mới tạo và SG có tên dxxxx_controller. Lý do vì đây là SG đặc biệt của AWS Managed AD tạo ra. 
![ec2](images/ec2-11.jpg)

- Sau đó chọn key và khởi tạo EC2 như cái vừa làm
- Mất 1 lúc để EC2 hoạt động được. Cho dù status: Checked Passed 2/2 nhưng cũng nên đợi 1~2 phút để cho instance hoàng thành việc warm-up thì mới log-in vào được.
- Để log-in vào 1 EC2, chọn EC2 đó => Connect => RDP Client
![ec2](images/ec2-12.jpg)
![ec2](images/ec2-13.jpg)
- Chúng ta có thể download file RDP hoặc xài public ip và RPD trên máy local cũng được. 
- Ngoài ra, chọn phần Get Password => Browse file RSA Key khi nãy vào và bấm Decrypt sẽ ra password cho bạn login. 
- Nếu bạn muốn login dạng domain thì: domain-name\Admin. Password là password đã khởi tạo ở phần AWS Managed AD
![ec2](images/ec2-14.jpg)
- Nếu bạn xài phần mềm RDP ở local thì có thể sử dụng Public IP như đã nói để login vào EC2. Lưu ý là, chỉ những EC2 ở phần Public Network mới có Public IP. Và những EC2 ở phần Private Network sẽ KO có Public IP, mà chỉ có Private IP.
![ec2](images/ec2-15.jpg)
- Vậy là đã log-in được từ Local => Bastion Host. Từ Bastion Host này, chúng ta sẽ login vào AD-Manager. Login theo phương thức domain
![ec2](images/ec2-16.jpg)
![ec2](images/ec2-17.jpg)
![ec2](images/ec2-18.jpg)
- Để hoàng thành việc quản lý AWS Managed AD. Chúng ta cần phải cài các dịch vụ cho EC2 AD-Manager. Các bạn làm Admin thì cũng không quá xa lạ gì với các dịch vụ này rồi
- Vào Server Management => Add Role & Features => Next đến Features (chúng ta không promote AD nên bỏ qua phần server roles): Group Policy Management, Remote Server Administration Tools => Role Administration Tools: AD AD & AD LDS, DNS Server Tools => NEXT 
![ec2](images/ec2-19.jpg)
- Mất 1 lúc để hoàng thành việc setup. Sau đó kiểm tra lại sẽ thấy những server tools quen thuộc. 
- Vậy là chúng ta đã hoàng thành việc quản trị AWS Managed AD trên EC2
 NEXT 
![ec2](images/ec2-20.jpg)

**5. Thiết kế AWS FSx - Windows** 
- AWS Management Console => Amazon FSx => Create File System
![fsx](images/fsx-1.jpg)
- FSx Windows => NEXT
![fsx](images/fsx-2.jpg)
- Đặt tên và setup như hình. Tên có thể tùy ý
![fsx](images/fsx-3.jpg)
![fsx](images/fsx-4.jpg)
- Các phần còn lại cứ để default. Sau đó NEXT => Xem lại option => Create File System
- Sẽ mất 1 khoảng thời gian tầm 40 phút để khởi tạo FSx
- chúng ta cũng nên check kĩ lưỡng để tránh trường hợp underlied-host của AWS có vấn đề. Khi thấy status là Available có nghĩa là FSx đã tạo xong và sẵn sàng sử dụng.
![fsx](images/fsx-5.jpg)
- Để tiến hành việc sử dụng FSx, chúng ta cần phải copy FSx DNS Name để tiến hành Map Network Drive. 
- Nhấp chọn FSx đã khởi tạo => Network & Security => FSx DNS Name nằm ở dưới
![fsx](images/fsx-6.jpg)
- Các bạn lưu ý 1 chỗ là mặc định, File & sharing sẽ turned-off để đảm bảo an toàn hệ thống. Chúng ta có thể turn-on hoặc trigger cho windows server tự động turn-on bằng cách: File Explorer => Nhấp vào Network => Bây giờ thì Windows sẽ hiện 1 thanh nhỏ hỏi chúng ta có turn-on File-Sharing trên môi trường Network hay không => Turn On
![fsx](images/fsx-8.jpg) 
- FSx DNS Name khi nãy copy, bây giờ paste vào thanh address với cú pháp: \\FSx-DNS-Name => Enter => Sẽ truy cập đc vào FSx
![fsx](images/fsx-7.jpg)
- Chuột phải vào folder share => Map Network Drive => Tiến hành thực hiện Map Network Drive như bình thường. Sau khi xong chúng ta vào This PC trong File Explorer => sẽ thấy có folder share từ FSx về local
![fsx](images/fsx-9.jpg)
- Đây là folder share mặc định trên FSx. Nhưng giả sử chúng ta có nhiều phòng ban, thì có 1 giải pháp khác đó là FSx cũng cho phép chúng ta tự tạo ra folder share riêng
- Menu Start => fsmgmt.msc (File Share Management) => Hiện ra 1 cửa sổ => Chuột phải => Connect to Another Computer
![fsx](images/fsx-10.jpg)
- Copy & Paste FSx DNS Name vào và OK 
![fsx](images/fsx-11.jpg)
![fsx](images/fsx-12.jpg)
- Tại cửa sổ FSx File Share vừa kết nối sẽ thấy thư mục share => chuột phải => New Share => NEXT => Browse => sẽ thấy folder $d => chọn lấy và phía dưới bấm vào Make New Folder
![fsx](images/fsx-13.jpg)
- Đặt tên cho folder tùy ý => NEXT => Hiệu chỉnh phân quyền sơ lược cho folder này => Sau đó Finish 
![fsx](images/fsx-14.jpg)
- Trở lại File Explorer và truy cập vào FSx như đã làm ở trên => Sẽ thấy folder mới tạo => Chuột phải => Map Network Drive => Sau khi xong sẽ thấy folder này ở local 
![fsx](images/fsx-15.jpg)
![fsx](images/fsx-16.jpg)

**5. Thiết kế AWS RDS - Microsoft SQL và quản lý AWS RDS từ EC2** 
- AWS Management Console => Relational Database Service => Create Database => Standard Create => MS SQL => Standard Edition => Chọn version mới nhất tại thời điểm các bạn thực hiện bài lab
- Điền đầy đủ thông tin liệt kê: tên db, user name, password
![aws-db](images/aws-db-2.jpg)
![aws-db](images/aws-db-1.jpg)
- DB Instance Class: Standard
- Multi AZ: NO
- Chọn VPC và Security Group cho DB. Lưu ý là SG phải có luôn cả default SG mà AWS Managed AD tạo ra và SG do chính bạn tạo
![aws-db](images/aws-db-3.jpg)
- chọn AZ cho DB.
- Check box Enable Server Authentication 
- Browse Directory => Chọn AWS Managed AD đã tạo ở trên
- Các option còn lại để mặc định
- Sau đó Create Database. Sẽ mất 1 khoảng thời gian tầm 40 phút để AWS tiến hành tạo DB
![aws-db](images/aws-db-4.jpg)
![aws-db](images/aws-db-5.jpg)
- Sau đó chúng ta cũng phải check lại xem DB đã tạo thành công hay chưa. Nếu status là Available có nghĩa đã đã tạo thành công
![aws-db](images/aws-db-6.jpg)
- Login vào EC2 AD Manager đã tạo trước đó => Download [SQL Server Management Studio](https://docs.microsoft.com/en-us/sql/ssms/download-sql-server-management-studio-ssms?view=sql-server-ver15) => Download bản mới nhất có thể được lúc bạn làm bài lab này
- Tiến hành cài đặt SQL Server Management Studio
- Lưu ý là cài tool này phải có kết nối internet. Nếu như AD Manager không ra internet được thì nên xem lại NAT Gateway đã attach đúng chưa. Vì EC2 AD Manager nằm trong vùng Private Network.
- Sau khi cài xong, quay trở lại AWS RDS console => chọn db vừa khởi tạo xong => copy end-point 
![aws-db](images/aws-db-7.jpg)
- Tại SQL Server Management Studio => Paste Endpoint => chỉnh option như hình => nhập username và password => connect
![aws-db](images/aws-db-8.jpg)
- Tại đây, chúng ta sẽ quản lý được Amazon RDS Service như bình thường chúng ta quản lý MS DB Server
![aws-db](images/aws-db-9.jpg)


