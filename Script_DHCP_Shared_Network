#!/bin/bash
#Install DHCP Share Network

check_user_permission()
{
clear
	if [ $(id -u) -ne 0 ]
	then
		clear
		echo 'User can not permission install package.'
		echo 'Please Login Root Account.'
		echo 'Thanks!'
		exit $1
	fi
}

change_file()						#doi tat ca tu trong file
{
	sed -i "s/$1/$2/g" $3
}

check_package_install()
{
	clear
	echo 'Install package'
	for i in $package_install
	do
		if [ "$(rpm -qa $i)" == "" ] 
		then
			yum -y install $i > /dev/null 2>&1
			if [ $? -ne 0 ]
			then
				clear
				echo 'Loi: khong the cai dat duoc package'
				sleep 3
				exit $1
			else
				echo 'Install' $i 'Ok'
			fi
		else
			echo "Package $i da duoc cai dat."
		fi
	done
	sleep 2
}

Check_IP_state()							#Check IP da co la tinh hay dong
{
	local stat=1;
	local get_state=$(cat $1 | grep 'BOOTPROTO' | tr '=' ' ' | awk '{print $2}')
	if [ "$get_state" == 'dhcp' ]
	then
		stat=0;
	fi
	return $stat;
}

Check_IP()									#Check xem dat dia chi IP dung(0) hay ko dung(1)
{
        local ip=$1
        local stat=1
        if [[ $ip =~ ^[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}$ ]];
		then
                OIFS=$IFS
                IFS='.'
                ip=($ip)
				IFS=$OIFS
                [[ ${ip[0]} -le 255 && ${ip[1]} -le 255 \
                && ${ip[2]} -le 255 && ${ip[3]} -le 255 ]]
                stat=$?
        fi
        return $stat
}

fixIP()										#Chinh sua IP
{
	cp $1 /a;
	sed -i '/IPADDR/d' /a;
	sed -i '/NETMASK/d' /a;
	sed -i '/GATEWAY/d' /a;
	echo "IPADDR=$2" >> /a;
	echo "NETMASK=$3" >> /a;
	echo "GATEWAY=$4" >> /a;
	cat /a > $1
}

Conf_IP()									#Cau hinh IP
{
	local interface="/etc/sysconfig/network-scripts/ifcfg-$1"
	Check_IP_state $interface
	if [ $? -eq 0 ]								#IP dong
	then
		change_file dhcp none $interface 
		fixIP $interface $2 $3 $4
	else										#IP tinh
		fixIP $interface $2 $3 $4
	fi
}
GetSub()						#Lay ra Dia chi mang cua subnet de khai bao trong file conf
{
	temp=$(echo $1 | tr "." " ")
	ip1=$(echo $temp | awk '{print $1}')
	ip2=$(echo $temp | awk '{print $2}')
	ip3=$(echo $temp | awk '{print $3}')
	IPSub="$ip1.$ip2.$ip3.0"
}

Conf_DHCP()
{
get_array=("${@}")
echo ${get_array[10]}
	cat >> /etc/dhcp/dhcpd.conf <<Lam
	subnet ${get_array[0]} netmask ${get_array[1]}{
	}
	shared-network ${get_array[2]}{
	subnet ${get_array[3]} netmask ${get_array[4]}{
		pool{
			option routers ${get_array[5]};
			range ${get_array[6]} ${get_array[6]};
		}
	}
	
	subnet ${get_array[8]} netmask ${get_array[9]}{
		pool{
			option routers ${get_array[10]};
			range ${get_array[11]} ${get_array[12]};
		}
	}
	}
Lam
}

Routing_Net()
{
	ip route add $1/24 via $2
	ip route add $3/24 via $2
}
Menu()
{
	clear
	read -p "Nhap interface cau hinh dhcp: " card
	while true
	do
		read -p "Nhap IP cau hinh dhcp:" IP
		Check_IP $IP
		if [ $? -ne 0 ]
		then
			echo "IP $IP cua ban nhap sai cu phap."
		else
			break;
		fi
	done
	read -p "Nhap subnet mask: " subnet
	while true
	do
		read -p "Nhap default gateway: " gateway
		Check_IP $gateway
		if [ $? -ne 0 ]
		then
			echo "IP $IP cua ban nhap sai cu phap."
		else
			break;
		fi
	done
	Conf_IP $card $IP $subnet $gateway
	GetSub $IP
	read -p "Nhap ten shared-network: " name
	read -p "Nhap network 1: " network1
	read -p "Nhap subnet netmask 1: " subnet1
	read -p "Nhap default gateway 1: " gateway1
	read -p "Nhap IP dau cua network 1: " first1
	read -p "Nhap IP cuoi cua network 1: " last1
	read -p "Nhap network 2: " network2
	read -p "Nhap subnet netmask 2: " a22
	read -p "Nhap default gateway 2: " b22
	read -p "Nhap IP dau cua network 2: " c22
	read -p "Nhap IP cuoi cua network 2: " d22
	Conf_DHCP "$IPSub" "$subnet" "$name" "$network1" "$subnet1" "$gateway1" "$first1" "$last1" "$network2" "$a22" "$b22" "$c22" "$d22"
	read -p "Nhap dia chi IP cua router ket noi voi DHCP server: " IProuter
	Routing_Net $subnet1 $IProuter $subnet2
}
Main()
{
	check_user_permission
	package_install=dhcp
	check_package_install 
	Menu
}
Main
