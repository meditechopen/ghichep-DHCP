# Các bài lab với DHCP
## Mục lục

[I.Cài đặt DHCP server] (#I)

[II.Labs] (#II)

[1.Subnet Declaration] (#1)

[2.Range Parameter] (#2)

[3.Static IP Address Using DHCP] (#3)

[4.Shared-network Declaration] (#4)

[5.Group Declaration] (#5)

[III.DHCP Relay Agent] (#III)

<a name="I"></a>
### I.Cài đặt DHCP server
- Trong các bài labs này, tôi dùng hđh centos 6.7 làm DHCP Server và DHCP Relay agent.
- Trước tiên bạn cài đặt gói DHCP với quyền root.
`yum -y install dhcp`

- Sau khi cài, sẽ xuất hiện file configuration tại đường dẫn /etc/dhcp/dhcpd.conf, nhưng chỉ là file trống.
<img src="http://i.imgur.com/rXNdDXf.png" />

File configuration tương tự được lưu tại đường dẫn /usr/share/doc/dhcp-4.1.1/dhcpd.conf.sample, bạn
có thể dùng file này để cấu hình file configuration của bạn vì nó hướng dẫn rất chi tiết.

<a name="II"></a>
### II.Một số mô hình DHCP đơn giản
- Bước đầu tiên để cấu hình DHCP server là bạn phải tạo 1 file configuration ở đó lưu trữ thông tin về mạng
cho clients.Sử dụng file này để khai báo các tùy chọn riêng và chung cho hệ thống các clients.
- Có 2 kiểu báo cáo:
*Parameters(Các Thông số)* -- thể hiện cách thực hiện các tác vụ như thế nào, ở đâu và những tùy chọn
cấu hình network gửi đến clients là gì.
*Declarations(Các Khai báo)* -- mô tả cấu trúc của mạng, các clients, cung cấp địa chỉ cho các clients,
hoặc áp dụng 1 nhóm các thông số tới 1 nhóm các khai báo.
- Các thông số thường được bắt đầu bằng các từ khóa "options"
- Các thông số trước dấu "{}" được coi là thông số chung, áp dụng cho tất cả các phần bên dưới nó.

<a name="1"></a>
#### 1.Subnet Declaration
##### 1.1.Mô hình
<img src="http://i.imgur.com/OC14xPD.png" />

- DHCP server sử dụng card VMnet1 trên Vmware và có IP:10.0.1.1.
- Client chạy hđh win7, sử dụng card VMnet1 trên vmware(bắt buộc cùng card với DHCP server), sử dụng dịch vụ dhcp để nhận IP.
- Dải IP được cấp: 10.0.1.10 - 10.0.1.20

##### 1.2.Giới thiệu
- Các tùy chọn về routers(default gateway), subnet-mask, domain-search, domain-name-servers, và time-offset
được sử dụng cho bất kì host nào được khai báo bên dưới.
- Với mỗi subnet được cấp và được kết nối tới DHCP server, nhất định phải có 1 subnet được khai báo để
cho DHCP server công nhận là có 1 địa chỉ trong subnet đó, kể cả subnet đó được khai báo rỗng không có địa chỉ 
thì nó sẽ được cấp động.

##### 1.3.Các bước triển khai
- Add thêm card mạng.
- Đặt IP cho máy.
- Khai báo file config
- Start dịch vụ và test.

##### 1.4.Triển khai chi tiết
###### a.Add thêm card mạng.
- Trên thanh công cụ của vmware, vào phần *VM* -> *Settings* sẽ hiện ra bảng cấu hình cho máy ảo của bạn, hoặc bạn có thể ấn Ctrl + D.
<img src="http://i.imgur.com/R6ctY0U.png" />

- Sau khi hiện ra bảng cấu hình, bạn chọn *Add* -> *Network Adapter* -> *Next* -> *Custom: Specific virtual network* -> *VMNet1* 
-> *Finish* -> *OK*
<img src="http://i.imgur.com/oMUACdN.png" />
- Vậy là bạn đã add thành công card VMNet1, vào kiểm tra xem máy ảo đã nhận card hay chưa.
<img src="http://i.imgur.com/EJBMCou.png" />

- Các bạn làm tương tự với các máy ảo còn lại.

###### b.Đặt IP cho máy.
- Cấu hình IP cho DHCP server:bạn phải tạo 1 file ifcfg-eth1 là file cấu hình cho card mạng VMNet1 mới tạo.
Do nó k có sẵn nên t copy từ file ifcfg-eth0 và chỉnh sửa cho phù hợp.Các file này trong đường dẫn */etc/sysconfig/network-scripts*
```sh
cp /etc/sysconfig/network-scripts/ifcfg-eth0 /etc/sysconfig/network-scriptsifcfg-eth1
vi /etc/sysconfig/network-scriptsifcfg-eth1
```
<img src="http://i.imgur.com/BfMVlbM.png" />

- Các tham số có mũi tên đỏ là các tham số cần chỉnh sửa và thêm vào.
- Bạn up card mạng lên: `ifup eth1`
- Kiểm tra IP: `ip a`
<img src="http://i.imgur.com/pPcfMsq.png" />

###### c.Khai báo file config
- Bạn khai báo vào file  /etc/dhcp/dhcpd.conf  như sau:

```sh
subnet *subnet* netmask *netmask* {
	option routers					*default gateway*;
	option subnet-mask 				...;
	option domain-search 			...;
	option domain-name-servers 		...;
	option time-offset 				-18000;		#Eastern Standard Time
	range *Start IP* *End IP*;
}
```

<img src="http://i.imgur.com/HXwhXF4.png" />

###### d. Start dịch vụ và test
- Start dịch vụ dhcp trên server.
- Xin cấp ip từ dhcp server của client.
<img src="http://i.imgur.com/bf2zS2I.png" />

- Kiểm tra trên client(win7)

<img src="http://i.imgur.com/1OaPS9S.png" />

<a name="2"></a>
#### 2.Range Parameter
##### 2.1.Mô hình
<img src="http://i.imgur.com/bjTaOxt.png" />

- Cấu hình Server và Client như phần 1
- Dải IP được cấp: 10.0.1.21 - 10.0.1.30 

##### 2.2 Giới thiệu
- Trong ví dụ này DHCP server sẽ khai báo default lease time, maximum lease time và các giá trị cấu hình mạng
cho tất cả clients cùng hay # subnet(khác với ví dụ trên là áp dụng riêng cho các client trong cùng 1 subnet)

##### 2.3 Các bước triển khai
- Khai báo file config
- Restart dịch vụ và test.

##### 2.4.Triển khai chi tiết
- Tương tự tôi khai báo vào file conf như sau:

```sh
default-lease-time ...;
max-lease-time ...;
option subnet-mask ...;
option broadcast-address ...;
option routers ...;
option domain-name-servers ...;
option domain-search ...;
subnet ... netmask ... {
   range ... ...;
}
```

<img src="http://i.imgur.com/JYSJSrh.png" />

- Kiểm tra trên client

<img src="http://i.imgur.com/ytdn9Ne.png" />

<a name="3"></a>
#### 3.Static IP Address Using DHCP
##### 3.1.Mô hình
<img src="http://i.imgur.com/JHimYuV.png" />

- Cấu hình Server và Client như phần 1
- IP được fix cố định cho client: 10.0.1.32 dựa trên địa chỉ MAC:00-0C-29-FE-01-4E của client.

##### 3.2. Giới thiệu
- Để đăng kí địa chỉ ip dựa trên địa chỉ MAC của card mạng, tôi sử dụng thông số hardware ethernet cùng với
khai báo host.

##### 3.3 Các bước triển khai
- Khai báo file config
- Restart dịch vụ và test.

##### 3.4 Triển khai chi tiết
- Khai báo vào file conf như sau:
```sh
default-lease-time ...;
max-lease-time ...;
option subnet-mask ...;
option broadcast-address ...; 
option routers ...; 
option domain-name-servers ...; 
option domain-search ...; 
subnet ... netmask ... { 
   
} 
host *host name* { 
    option host-name ...; 
    hardware ethernet *MAC of card*; 
    fixed-address *IP fixed*; 
} 
```
- Bắc buộc phải có tham số *subnet ... mask ... {}* thì dịch vụ mới chạy được.
- Kiểm tra trên client
<img src="http://i.imgur.com/NOpapxF.png" />

<a name="4"></a>
#### 4.Shared-network Declaration
##### 4.1.Mô hình mạng
<img src="http://i.imgur.com/YZsue5P.png" />

- DHCP server, có 1 kết nối tới Router, IP:192.168.1.254, ở đây tôi sử dụng card VMnet1.
- Router:
<ul>
	<li>cổng f0/0 kết nối với DHCP server, IP:192.168.1.1</li>
	<li>cổng f0/1 kết nối với mạng lan, ko có IP, chạy router on stick để chia vlan</li>
	<li>Subinterface f0/0.11 kết nối với vlan 11, IP:10.0.11.1</li>
	<li>Subinterface f0/0.22 kết nối với vlan 22, IP:10.0.22.1</li>
</ul>
- Switch:cổng nối với Router mode trunk, các cổng nối với mạng lan để mode access.

##### 4.2 Giới thiệu
- Khi bạn muốn cấp nhiều ip cho nhiều subnet mạng của bạn, mà bạn chỉ có 1 DHCP server thì bạn sẽ dùng phương pháp này.
- Tiết kiệm được chi phí nhưng hiệu năng của dhcp server sẽ kém đi.

##### 4.3 Các bước triển khai
- Cấu hình DHCP server
- Cấu hình Router
- Cấu hình Switch
- Restart dịch vụ và test

##### 4.4 Triển khai chi tiết
###### a. Cấu hình DHCP server
- Cấu hình ip cho dhcp server: 192.168.1.254
- Khai báo trên file conf của dhcp server:

```sh
subnet 192.168.1.0 netmask 255.255.255.0 {   
} 
shared-network 11-22 { 
subnet ... netmask ... { 
 	default-lease-time ...; 
 	max-lease-time ...; 
 	pool { 
 	option routers ...; 
 	range ... ...; 
 	} 
} 
 	subnet ... netmask ... { 
 	default-lease-time ...; 
 	max-lease-time ...; 
 	pool { 
 	option routers ...; 
 	range ... ...; 
 	} 
} 
} 
```
*11-22* : tên bạn tùy chọn, 
*subnet* : các subnet bạn muốn cấp IP.
- Cấu hình định tuyến tĩnh đến 2 vlan và show bảng định tuyến.

```sh
ip route add vlan1/24 via IpRouter
ip route add vlan2/24 via IpRouter
route -n
```
<img src="http://i.imgur.com/wgoi6bj.png" />

###### b. Cấu hình Router:

```sh
 interface FastEthernet0/0 
 no shutdown 
 ip address 192.168.1.1 255.255.255.0 
 interface FastEthernet0/1 
 no shutdown 
 interface FastEthernet0/1.11 
  encapsulation dot1Q 11 
  ip address 10.0.11.1 255.255.255.0 
  ip helper-address IpDHCPServer 
 interface FastEthernet0/1.22 
  encapsulation dot1Q 22 
  ip address 10.0.22.1 255.255.255.0 
  ip helper-address IpDHCPServer
```

###### c.Cấu hình Switch:

```sh
 Vlan 11 
 Vlan 22 
 interface range f0/1-12 
 switchport mode access 
 switchport access vlan 11 
 interface range f0/13-20 
 switchport mode access 
 switchport access vlan 22 
 interface range f0/21-24 
 switchport mode trunk 
```
###### d.Restart dịch vụ và test
- Restart lại dịch vụ dhcp trên server và kiểm tra ip trên 2 client thuộc 2 vlan # nhau.
- vlan 11:

<img src="http://i.imgur.com/UeoYABa.png" />

- vlan 22:

<img src="http://i.imgur.com/s8LhlRz.jpg" height=50% width=50% />

<a name="5"></a>
#### 5.Group Declaration
##### 5.1 Mô hình mạng
<img src="http://i.imgur.com/lVbHn0j.png" />
- Cấu hình Server và Client như phần 1
- IP được fix cố định cho client win7: 10.0.1.55 dựa trên địa chỉ MAC:00-0C-29-FE-01-4E .
- IP được fix cố định cho client win8: 10.0.1.56 dựa trên địa chỉ MAC:00-50-56-C0-00-01 .

##### 5.2 Giới thiệu
- Khai báo group được sử dụng để áp các thông số chung cho nhóm đấy.
- Có thể là 1 nhóm shared-network, subnets hoặc các host.
- Ở đây tôi sẽ ví dụ về 1 nhóm các host.

##### 5.3 Các bước triển khai
- Cấu hình file config.
- Restart dịch vụ và test.

##### 5.4 Triển khai chi tiết
- Cấu hình file config
```sh
 group {
   option routers                  ...;
   option subnet-mask              ...;
   option domain-search              "example.com";
   option domain-name-servers       ...;
   option time-offset              -18000;     # Eastern Standard Time
   host win7 {
      option host-name "win7.example.com";
      hardware ethernet ...;
      fixed-address IPfixed;
   }
   host win8 {
      option host-name "win8.example.com";
      hardware ethernet ...;
      fixed-address IPfixed;
   }
} 
```

<img src="http://i.imgur.com/V2GY7Vi.png" />

- Kiểm tra trên client win7
<img src="http://i.imgur.com/ehPx2dt.png" />

- Kiểm tra trên client win8
<img src="http://i.imgur.com/CKaWqNp.png" />

<a name="III"></a>
### III.DHCP Relay Agent
#### 1. Mô hình mạng.
<img src="http://i.imgur.com/4UvSBwI.png" />

- DHCP server: sử dụng card VMNet2 nối với DHCP relay agent, IP:10.0.2.1
- DHCP relay agent:
<ul>
	<li>card VMNet2 nối với DHCP server, IP:10.0.2.2</li>
	<li>card VMNet1 nối với mạng lan, IP:10.0.1.2</li>
</ul>
- 2 client win7, win8 sử dụng card VMNet1 để nhận ip động.

#### 2. Giới thiệu
- DHCP relay agent cho phép chuyển các yêu cầu dhcp và bootp từ 1 subnet ko có dhcp server trong đấy,
tới 1 hoặc nhiều dhcp server trên các subnet khác.
- Khi 1 client yêu cầu thông tin, DHCP relay agent chuyển yêu cầu đấy đến danh sách các dhcp server xác định
- Khi 1 DHCP server gửi lại reply, thì reply đó có thể là broadcast hoặc unicast gửi đến yêu cầu từ nguồn ban đầu.
- Mặc định DHCP server sẽ lắng nghe tất cả các yêu cầu DHCP từ tất cả các card mạng, trừ khi nó được chỉ định
trong /etc/sysconfig/dhcrelay với chỉ thị "INTERFACES".

#### 3.Các bước triển khai
- Cấu hình DHCP server
- Cấu hình DHCP relay agent
- Test

#### 4.Triển khai chi tiết
##### a.Cấu hình DHCP server
- Set IP trên card VMnet2 nối với DHCP relay agent.
<img src="http://i.imgur.com/Cd954IA.png" />

- Cấu hình file conf để cấp ip cho subnet 1:

```sh
 subnet ... netmask ...{
	option routers					...;
	option subnet-mask 				...;
	option domain-search 			"example.com";
	option domain-name-servers 		...;
	option time-offset 				-18000;		#Eastern Standard Time
	range ... ...;
} 
```
<img src="http://i.imgur.com/0uEEt2M.png" />

- Cấu hình định tuyến tĩnh đến subnet 1 và kiểm tra lại bảng định tuyến.

```sh
 ip route add 10.0.1.0/24 via 10.0.2.2 
 route -n 	
```

<img src="http://i.imgur.com/A3UOaBq.png" />

- Cuối cùng khởi động lại dịch vụ dhcp: "service dhcpd restart".

##### b.Cấu hình trên dhcp relay agent
- Set IP trên card VMnet2 nối với DHCP server và VMnet1 nối với subnet 1.
<img src="http://i.imgur.com/SUNCu8N.png" />

- Chỉnh sửa file /etc/sysctl.conf.forward:ta chỉnh số, nếu có 1 server thì là 1, 2 server thì là 2.

<img src="http://i.imgur.com/SpT4FwW.png" />

- Chỉnh sửa file /etc/sysconfig/dhcrelay.

<img src="http://i.imgur.com/JBrxUTf.png" />

- Cuối cùng khởi động lại dịch vụ dhcrelay: "service dhcrelay restart".

##### c.Kiểm tra ip trên client
- win7:

<img src="http://i.imgur.com/EPQWBKx.png" />

- win8:

<img src="http://i.imgur.com/dHINSTc.png" />



