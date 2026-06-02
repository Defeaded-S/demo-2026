*Copy code and paste to tty*


**ISP**
```bash
hostnamectl set-hostname ISP; exec bash
sed -i 's/HOSTNAME=host-196/HOSTNAME=ISP/g' /etc/sysconfig/network
mkdir -p /etc/net/ifaces/enp7s2
cat <<EOF > /etc/net/ifaces/enp7s2/options
BOOTPROTO=static
TYPE=eth
EOF
cp -r /etc/net/ifaces/enp7s2 /etc/net/ifaces/enp7s3
echo 172.16.4.1/28 > /etc/net/ifaces/enp7s2/ipv4address
echo 172.16.5.1/28 > /etc/net/ifaces/enp7s3/ipv4address
sed -i 's/net.ipv4.ip_forward = 0/net.ipv4.ip_forward = 1/g' /etc/net/sysctl.conf
systemctl restart network
wait
apt-get update; apt-get install iptables -y
wait
iptables -t nat -A POSTROUTING -o enp7s1 -j MASQUERADE
iptables-save > /etc/sysconfig/iptables
systemctl enable iptables
```
**HQ-RTR**
```bash
hostnamectl set-hostname hq-rtr.au-team.irpo; exec bash
sed -i 's/HOSTNAME=host-196/HOSTNAME=hq-rtr.au-team.irpo/g' /etc/sysconfig/network
mkdir -p /etc/net/ifaces/enp7s1
cat <<EOF > /etc/net/ifaces/enp7s1/options
BOOTPROTO=static
TYPE=eth
EOF
echo 172.16.4.2/28 > /etc/net/ifaces/enp7s1/ipv4address
echo default via 172.16.4.1 > /etc/net/ifaces/enp7s1/ipv4route
cp -r /etc/net/ifaces/enp7s1 /etc/net/ifaces/enp7s2
mkdir -p /etc/net/ifaces/enp7s2.100 /etc/net/ifaces/enp7s2.200 /etc/net/ifaces/enp7s2.999
cat <<EOF > /etc/net/ifaces/enp7s2.100/options
TYPE=vlan
HOST=enp7s2
VID=100
EOF
cat <<EOF > /etc/net/ifaces/enp7s2.200/options
TYPE=vlan
HOST=enp7s2
VID=200
EOF
cat <<EOF > /etc/net/ifaces/enp7s2.999/options
TYPE=vlan
HOST=enp7s2
VID=999
EOF
echo 172.16.100.1/28 > /etc/net/ifaces/enp7s2.100/ipv4address
echo 172.16.200.1/27 > /etc/net/ifaces/enp7s2.200/ipv4address
echo 172.16.99.1/29 > /etc/net/ifaces/enp7s2.999/ipv4address
mkdir /etc/net/ifaces/gre1
cat <<EOF > /etc/net/ifaces/gre1/options
TYPE=iptun
TUNTYPE=gre
TUNLOCAL=172.16.4.2
TUNREMOTE=172.16.5.2
TUNTTL=64
TUNMTU=1476
TUNOPTIONS="ttl 64"
DISABLE=no
EOF
echo 10.0.0.1/30 > /etc/net/ifaces/gre1/ipv4address
sed -i 's/net.ipv4.ip_forward = 0/net.ipv4.ip_forward = 1/g' /etc/net/sysctl.conf
echo nameserver 1.1.1.1 > /etc/resolv.conf
systemctl restart network
wait
apt-get update; apt-get install iptables -y
wait
iptables -t nat -A POSTROUTING -o enp7s1 -j MASQUERADE
iptables-save > /etc/sysconfig/iptables
systemctl enable iptables
apt-get install -y frr
sed -i 's/ospfd=no/ospfd=yes/g' /etc/frr/daemons
systemctl enable --now frr
vtysh
configure terminal
interface gre1
 no ip ospf passive
 ip ospf authentication message-digest
 ip ospf message-digest-key 1 md5 password
router ospf
 passive-interface default
 ospf router-id 10.0.0.1
 network 10.0.0.0/30 area 0
 network 172.16.100.0/28 area 0
 network 172.16.200.0/27 area 0
 network 172.16.99.0/29 area 0
end
write memory
exit
apt-get install -y dhcp-server
cat<<EOF > /etc/dhcp/dhcpd.conf
option domain-name-servers 172.16.100.2;
default-lease-time 600;
max-lease-time 7200;

log-facility local7;

subnet 172.16.200.0 netmask 255.255.255.224 {
    range 172.16.200.2 172.16.200.30;
    option routers 172.16.200.1;
    option domain-name-servers 172.16.100.2;
    option domain-name "au-team.irpo";
    option subnet-mask 255.255.255.224;
}
EOF
sed -i "s/DHCPDARGS=/DHCPDARGS=enp7s2.200/g" /etc/sysconfig/dhcpd
systemctl enable --now dhcpd
```
**BR-RTR**
```bash
hostnamectl set-hostname br-rtr.au-team.irpo; exec bash
sed -i 's/HOSTNAME=host-196/HOSTNAME=br-rtr.au-team.irpo/g' /etc/sysconfig/network
cat <<EOF > /etc/net/ifaces/enp7s1/options
BOOTPROTO=static
TYPE=eth
EOF
cp -r /etc/net/ifaces/enp7s1 /etc/net/ifaces/enp7s2
echo 172.16.5.2/28 > /etc/net/ifaces/enp7s1/ipv4address
echo default via 172.16.5.1 > /etc/net/ifaces/enp7s1/ipv4route
echo 192.168.0.1/24 > /etc/net/ifaces/enp7s2/ipv4address
mkdir /etc/net/ifaces/gre1
cat <<EOF > /etc/net/ifaces/gre1/options
TYPE=iptun
TUNTYPE=gre
TUNLOCAL=172.16.5.2
TUNREMOTE=172.16.4.2
TUNTTL=64
TUNMTU=1476
TUNOPTIONS="ttl 64"
DISABLE=no
EOF
echo 10.0.0.2/30 > /etc/net/ifaces/gre1/ipv4address
sed -i 's/net.ipv4.ip_forward = 0/net.ipv4.ip_forward = 1/g' /etc/net/sysctl.conf
echo nameserver 1.1.1.1 > /etc/resolv.conf
systemctl restart network
wait
apt-get update; apt-get install iptables -y
wait
iptables -t nat -A POSTROUTING -o enp7s1 -j MASQUERADE
iptables-save > /etc/sysconfig/iptables
systemctl enable iptables
apt-get install -y frr
sed -i 's/ospfd=no/ospfd=yes/g' /etc/frr/daemons
systemctl enable --now frr
vtysh
configure terminal
interface gre1
 no ip ospf passive
 ip ospf authentication message-digest
 ip ospf message-digest-key 1 md5 password
router ospf
 passive-interface default
 ospf router-id 10.0.0.2
 network 10.0.0.0/30 area 0
 network 192.168.0.0/24 area 0
end
write memory
exit
```
**HQ-SRV**
```bash
hostnamectl set-hostname hq-srv.au-team.irpo; exec bash
useradd remote_user --uid 2026 -m
echo P@ssw0rd | passwd remote_user --stdin
usermod -aG wheel remote_user
echo 'remote_user ALL=(ALL:ALL) NOPASSWD: ALL' >> /etc/sudoers
cat <<EOF > /etc/net/ifaces/enp7s1/options
TYPE=eth
BOOTPROTO=static
EOF
mkdir /etc/net/ifaces/enp7s1.100
cat <<EOF > /etc/net/ifaces/enp7s1.100/options
TYPE=vlan
HOST=enp7s1
VID=100
EOF
echo 172.16.100.2/28 > /etc/net/ifaces/enp7s1.100/ipv4address
echo default via 172.16.100.1 > /etc/net/ifaces/enp7s1.100/ipv4route
echo nameserver 1.1.1.1 > /etc/resolv.conf
systemctl restart network
wait
apt-get update
wait
sed -i 's/#Port 22/Port 2026/g' /etc/openssh/sshd_config
sed -i 's/#MaxAuthTries 6/MaxAuthTries 2/g' /etc/openssh/sshd_config
sed -i 's/#Banner none/Banner \/etc\/openssh\/banner/g' /etc/openssh/sshd_config
echo AllowUsers sshuser >> /etc/openssh/sshd_config
echo Authorized access only > /etc/openssh/banner
systemctl restart sshd
apt-get install -y dnsmasq
cat<<EOF > /etc/dnsmasq.conf
domain=au-team.irpo
expand-hosts
server=1.1.1.1
cache-size=1000
all-servers
no-negcache
interface=*
local=/au-team.irpo/
host-record=hq-rtr.au-team.irpo,172.16.100.1
host-record=hq-rtr.au-team.irpo,172.16.200.1
host-record=hq-rtr.au-team.irpo,172.16.99.1
host-record=hq-srv.au-team.irpo,172.16.100.2
host-record=hq-cli.au-team.irpo,172.16.200.2
address=/br-rtr.au-team.irpo/192.168.0.1
address=/br-srv.au-team.irpo/192.168.0.2
address=/docker.au-team.irpo/172.16.4.1
address=/web.au-team.irpo/172.16.5.1
EOF
cat <<EOF > /etc/resolv.conf
search au-team.irpo
nameserver 127.0.0.1
EOF
systemctl enable --now dnsmasq.service
```
**BR-SRV**
```bash
hostnamectl set-hostname br-srv.au-team.irpo; exec bash
useradd remote_user --uid 2026 -m
echo P@ssw0rd | passwd remote_user --stdin
usermod -aG wheel remote_user
echo 'remote_user ALL=(ALL:ALL) NOPASSWD: ALL' >> /etc/sudoers
cat <<EOF > /etc/net/ifaces/ens18/options
TYPE=eth
BOOTPROTO=static
EOF
echo 192.168.2.2/26 > /etc/net/ifaces/ens18/ipv4address
echo default via 192.168.2.1 > /etc/net/ifaces/ens18/ipv4route
echo nameserver 1.1.1.1 > /etc/resolv.conf
systemctl restart network
wait
apt-get update
wait
sed -i 's/#Port 22/Port 2026/g' /etc/openssh/sshd_config
sed -i 's/#MaxAuthTries 6/MaxAuthTries 2/g' /etc/openssh/sshd_config
sed -i 's/#Banner none/Banner \/etc\/openssh\/banner/g' /etc/openssh/sshd_config
echo AllowUsers sshuser >> /etc/openssh/sshd_config
echo Authorized access only > /etc/openssh/banner
systemctl restart sshd
```
**HQ-CLI**
```bash
hostnamectl set-hostname hq-cli.au-team.irpo; exec bash
nmcli connection add type vlan con-name vlan200 ifname enp7s1.200 dev enp7s1 id 200
nmcli connection up vlan200
nmcli connection modify vlan200 connection.autoconnect yes
reboot
```
