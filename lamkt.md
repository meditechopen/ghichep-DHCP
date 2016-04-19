# Các bài lab với DHCP
## Mục lục

[1.Cài đặt DHCP server] (#1)

[2.Cấu hình file configuration] (#2)

[a.Subnet Declaration] (#2a)

[b.Range Parameter] (#2b)

[c.Static IP Address Using DHCP] (#2c)

[d.Shared-network Declaration] (#2d)

[e.Group Declaration] (#2e)

[3.DHCP Relay Agent] (#3)

<a name="1"></a>
### 1.Cài đặt DHCP server
Trước tiên bạn cài đặt gói DHCP với quyền root.
<img src="http://i.imgur.com/8Ew5WCX.png" />

Sau khi cài, sẽ xuất hiện file configuration tại đường dẫn /etc/dhcp/dhcpd.conf, nhưng chỉ là file trống.
<img src="http://i.imgur.com/rXNdDXf.png" />

File configuration tương tự được lưu tại đường dẫn /usr/share/doc/dhcp-4.1.1/dhcpd.conf.sample, bạn
có thể dùng file này để cấu hình file configuration của bạn vì nó hướng dẫn rất chi tiết.

<a name="2"></a>
### 2.Cấu hình file configuration
- Bước đầu tiên để cấu hình DHCP server là bạn phải tạo 1 file configuration ở đó lưu trữ thông tin về mạng
cho clients.Sử dụng file này để khai báo các tùy chọn riêng và chung cho hệ thống các clients.
- Có 2 kiểu báo cáo:
*Parameters(Các Thông số)* -- thể hiện cách thực hiện các tác vụ như thế nào, ở đâu và những tùy chọn
cấu hình network gửi đến clients là gì.
*Declarations(Các Khai báo)* -- mô tả cấu trúc của mạng, các clients, cung cấp địa chỉ cho các clients,
hoặc áp dụng 1 nhóm các thông số tới 1 nhóm các khai báo.
- Các thông số thường được bắt đầu bằng các từ khóa "options"
- Các thông số trước dấu "{}" được coi là thông số chung, áp dụng cho tất cả các phần bên dưới nó.
- Mô hình sau sẽ là mô hình dùng chung cho lab a,b,c.Các máy ảo đều sử dụng card VMnet1.
<img src="http://i.imgur.com/CIZAbfr.png" />

- Ở đây tôi dùng các máy ảo Centos2 làm dhcp server, Centos làm dhcp relay agent, 2 clients là win7 và win8(máy thật).

<a name="2a"></a>
#### a.Subnet Declaration
- Các tùy chọn về routers(default gateway), subnet-mask, domain-search, domain-name-servers, và time-offset
được sử dụng cho bất kì host nào được khai báo bên dưới.
- Với mỗi subnet được cấp và được kết nối tới DHCP server, nhất định phải có 1 subnet được khai báo để
cho DHCP server công nhận là có 1 địa chỉ trong subnet đó, kể cả subnet đó được khai báo rỗng không có địa chỉ 
thì nó sẽ được cấp động.
- Trong ví dụ này, có các tùy chọn chung cho các clients trong subnet được khai báo.Các clients được đăng kí
trong 1 địa chỉ ip nằm trong rải đã cho.
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

- Sau đó start dịch vụ dhcp trên server.
- Xin cấp ip từ dhcp server của client.
<img src="http://i.imgur.com/bf2zS2I.png" />

- Kiểm tra trên client(win7)

<img src="http://i.imgur.com/1OaPS9S.png" />

<a name="2b"></a>
#### b.Range Parameter
- Trong ví dụ này DHCP server sẽ khai báo default lease time, maximum lease time và các giá trị cấu hình mạng
cho tất cả clients cùng hay # subnet(khác với ví dụ trên là áp dụng riêng cho các client trong cùng 1 subnet)
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

<a name="2c"></a>
#### c.Static IP Address Using DHCP
- Để đăng kí địa chỉ ip dựa trên địa chỉ MAC của card mạng, tôi sử dụng thông số hardware ethernet cùng với
khai báo host.
- Với kiểu khai báo này thì card mạng có địa chỉ Mac 00-0C-29-FE-01-4E sẽ luôn nhận địa chỉ 10.0.1.32

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

<img src="http://i.imgur.com/NOpapxF.png" />

<a name="2d"></a>
#### d.Shared-network Declaration
- Khi bạn muốn cấp nhiều ip cho nhiều subnet mạng của bạn, mà bạn chỉ có 1 DHCP server thì bạn sẽ dùng phương pháp này.
- Tiết kiệm được chi phí nhưng hiệu năng của dhcp server sẽ kém đi.
- Mô hình mạng như sau:
<img src="http://i.imgur.com/4OUyfVL.png" />

- Khai báo trên file conf của dhcp server và cấu hình ip cho dhcp server: 

```sh
subnet ... netmask ... {   
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

- Cấu hình định tuyến tĩnh đến 2 vlan và show bảng định tuyến.

```sh
ip route add vlan1/24 via IpRouter
ip route add vlan2/24 via IpRouter
route -n
```

<img src="http://i.imgur.com/wgoi6bj.png" />

- Cấu hình Router:

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

- Cấu hình Switch:

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

- Reload lại dịch vụ dhcp trên server và kiểm tra ip trên 2 client thuộc 2 vlan # nhau.
- vlan 11:

<img src="http://i.imgur.com/UeoYABa.png" />

- vlan 22:

<img src="http://i.imgur.com/s8LhlRz.jpg" height=50% width=50% />

<a name="2e"></a>
#### e.Group Declaration
- Khai báo group được sử dụng để áp các thông số chung cho nhóm đấy.
- Có thể là 1 nhóm shared-network, subnets hoặc các host.
- Ở đây tôi sẽ ví dụ về 1 nhóm các host.

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

<a name="3"></a>
### 3.DHCP Relay Agent
- Mô hình mạng như sau:
<img src="http://i.imgur.com/sK8QbiJ.png" />

- DHCP relay agent cho phép chuyển các yêu cầu dhcp và bootp từ 1 subnet ko có dhcp server trong đấy,
tới 1 hoặc nhiều dhcp server trên các subnet khác.
- Khi 1 client yêu cầu thông tin, DHCP relay agent chuyển yêu cầu đấy đến danh sách các dhcp server xác định
- Khi 1 DHCP server gửi lại reply, thì reply đó có thể là broadcast hoặc unicast gửi đến yêu cầu từ nguồn ban đầu.
- Mặc định DHCP server sẽ lắng nghe tất cả các yêu cầu DHCP từ tất cả các card mạng, trừ khi nó được chỉ định
trong /etc/sysconfig/dhcrelay với chỉ thị "INTERFACES".
- Giờ ta bắt đầu cấu hình DHCP relay agent.

#### a.Cấu hình trên dhcp server
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

#### b.Cấu hình trên dhcp relay agent
- Set IP trên card VMnet2 nối với DHCP server và VMnet1 nối với subnet 1.
<img src="http://i.imgur.com/SUNCu8N.png" />

- Chỉnh sửa file /etc/sysctl.conf.forward:ta chỉnh số, nếu có 1 server thì là 1, 2 server thì là 2.

<img src="http://i.imgur.com/SpT4FwW.png" />

- Chỉnh sửa file /etc/sysconfig/dhcrelay.

<img src="http://i.imgur.com/JBrxUTf.png" />

- Cuối cùng khởi động lại dịch vụ dhcrelay: "service dhcrelay restart".

#### c.Kiểm tra ip trên client
- win7:

<img src="http://i.imgur.com/EPQWBKx.png" />

- win8:

<img src="http://i.imgur.com/dHINSTc.png" />



