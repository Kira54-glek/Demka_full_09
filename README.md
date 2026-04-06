<img width="561" height="671" alt="изображение" src="https://github.com/user-attachments/assets/c2ae75fa-bb8c-4d8b-97fc-5d024a828db3" />
git init
git add .
git commit -m "init network project"

# создаём все ветки
for task in task1-addressing task2-nat-isp task3-users task4-vlan task5-ssh task6-gre task7-ospf task8-nat-offices task9-dhcp task10-dns task11-time
do
  git branch $task
done
task1-addressing
git checkout task1-addressing
ISP/interfaces
auto ens224
iface ens224 inet static
address 172.16.4.1/28

auto ens256
iface ens256 inet static
address 172.16.5.1/28
HQ-RTR/interfaces
auto ens192
iface ens192 inet static
address 172.16.4.2/28
gateway 172.16.4.1
BR-RTR/interfaces
auto ens192
iface ens192 inet static
address 172.16.5.2/28
gateway 172.16.5.1

auto ens224
iface ens224 inet static
address 192.168.0.1/27
HQ-SRV/interfaces
auto ens192
iface ens192 inet static
address 192.168.100.62/26
gateway 192.168.100.1
BR-SRV/interfaces
auto ens192
iface ens192 inet static
address 192.168.0.2/27
gateway 192.168.0.1
🔥 4. ВЕТКА task2-nat-isp
git checkout task2-nat-isp
ISP/nat.sh
echo net.ipv4.ip_forward=1 > /etc/sysctl.conf
sysctl -p

apt install iptables iptables-persistent -y

iptables -t nat -A POSTROUTING -s 172.16.4.0/28 -o ens192 -j MASQUERADE
iptables -t nat -A POSTROUTING -s 172.16.5.0/28 -o ens192 -j MASQUERADE

netfilter-persistent save
👤 5. ВЕТКА task3-users
git checkout task3-users
HQ-SRV/users.sh
useradd sshuser -u 1010
echo "sshuser:P@ssw0rd" | chpasswd
usermod -aG sudo sshuser
echo "sshuser ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers
BR-SRV/users.sh
useradd sshuser -u 1010
echo "sshuser:P@ssw0rd" | chpasswd
usermod -aG sudo sshuser
echo "sshuser ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers
HQ-RTR/users.sh
useradd net_admin
echo "net_admin:P@$$word" | chpasswd
usermod -aG sudo net_admin
echo "net_admin ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers
BR-RTR/users.sh
useradd net_admin
echo "net_admin:P@$$word" | chpasswd
usermod -aG sudo net_admin
echo "net_admin ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers
🌐 6. ВЕТКА task4-vlan
git checkout task4-vlan
HQ-RTR/vlan.conf
apt install vlan -y
echo 8021q >> /etc/modules

auto ens224.100
iface ens224.100 inet static
address 192.168.100.1/26

auto ens224.200
iface ens224.200 inet static
address 192.168.200.1/28

auto ens224.999
iface ens224.999 inet static
address 192.168.99.1/29
🔐 7. ВЕТКА task5-ssh
git checkout task5-ssh
HQ-SRV/sshd_config
Port 2024
MaxAuthTries 2
PasswordAuthentication yes
Banner /etc/ssh/banner
AllowUsers sshuser
BR-SRV/sshd_config
Port 2024
MaxAuthTries 2
PasswordAuthentication yes
Banner /etc/ssh/banner
AllowUsers sshuser
🌉 8. ВЕТКА task6-gre
git checkout task6-gre
HQ-RTR/gre.conf
auto gre1
iface gre1 inet tunnel
address 172.16.0.1
netmask 255.255.255.252
mode gre
local 172.16.4.2
endpoint 172.16.5.2
ttl 64
BR-RTR/gre.conf
auto gre1
iface gre1 inet tunnel
address 172.16.0.2
netmask 255.255.255.252
mode gre
local 172.16.5.2
endpoint 172.16.4.2
ttl 64
🌍 9. ВЕТКА task7-ospf
git checkout task7-ospf
HQ-RTR/ospf.conf
router ospf
 router-id 1.1.1.1
 network 172.16.0.0/30 area 0
 network 192.168.100.0/26 area 1
BR-RTR/ospf.conf
router ospf
 router-id 2.2.2.2
 network 172.16.0.0/30 area 0
 network 192.168.0.0/27 area 3
🔥 10. ВЕТКА task8-nat-offices
git checkout task8-nat-offices
HQ-RTR/nat.sh
iptables -t nat -A POSTROUTING -s 192.168.100.0/26 -o ens192 -j MASQUERADE
iptables -t nat -A POSTROUTING -s 192.168.200.0/28 -o ens192 -j MASQUERADE
BR-RTR/nat.sh
iptables -t nat -A POSTROUTING -s 192.168.0.0/27 -o ens192 -j MASQUERADE
📡 11. ВЕТКА task9-dhcp
git checkout task9-dhcp
HQ-RTR/dhcp.conf
subnet 192.168.200.0 netmask 255.255.255.240 {
  range 192.168.200.2 192.168.200.14;
  option routers 192.168.200.1;
  option domain-name-servers 192.168.100.62;
}
🌐 12. ВЕТКА task10-dns
git checkout task10-dns
HQ-SRV/dnsmasq.conf
listen-address=192.168.100.62
server=8.8.8.8
server=8.8.4.4

address=/hq-rtr.au-team.irpo/192.168.100.1
address=/hq-srv.au-team.irpo/192.168.100.62
address=/br-rtr.au-team.irpo/192.168.0.1
address=/br-srv.au-team.irpo/192.168.0.2
⏰ 13. ВЕТКА task11-time
git checkout task11-time
ALL/time.sh
timedatectl set-timezone Asia/Tomsk
