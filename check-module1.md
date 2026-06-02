**Copy code and paste to tty**


***ISP***
```bash
cd ~
echo -e "\n\n---------ISP---------" > report.txt
echo "hostname: $(hostname)" >> report.txt
echo -e "interfaces:\n$(ip -4 --br a)"  >> report.txt
ip r show default | grep -q "proto dhcp" && echo "enp7s1 DHCP"  >> report.txt || echo "enp7s1 non DHCP"  >> report.txt
echo -e "interfaces:\n$(iptables -t nat -L)"  >> report.txt
echo -e "NAT:\n$(sysctl -a | grep 'ip_forward ')"  >> report.txt
echo -e "$(timedatectl)"  >> report.txt
echo "---------END---------"  >> report.txt
clear
cat report.txt
```

***HQ-RTR***
```bash
echo -e "\n\n---------HQ-RTR---------" > report.txt
echo "hostname: $(hostname)" >> report.txt
echo -e "interfaces:\n$(ip -4 --br a)"  >> report.txt
echo -e "VLAN:\n$(ip -d link show | grep -B 2 'vlan protocol 802.1Q')" | awk '!/link\/ether/'  >> report.txt
echo -e "interfaces:\n$(iptables -t nat -L)"  >> report.txt
echo -e "NAT:\n$(sysctl -a | grep 'ip_forward ')"  >> report.txt
echo -e "$(timedatectl)"  >> report.txt
echo "---------END---------"  >> report.txt
clear
cat report.txt
```
***BR-RTR***
```bash
echo -e "\n\n---------HQ-RTR---------" > report.txt
echo "hostname: $(hostname)" >> report.txt
echo -e "interfaces:\n$(ip -4 --br a)"  >> report.txt
echo -e "interfaces:\n$(iptables -t nat -L)"  >> report.txt
echo -e "NAT:\n$(sysctl -a | grep 'ip_forward ')"  >> report.txt
echo -e "$(timedatectl)"  >> report.txt
echo "---------END---------"  >> report.txt
clear
cat report.txt
```
***HQ-SRV***
```bash
---HQ-SRV---(root toor sshuser P@ssw0rd)
echo -e "\n\n---------HQ-RTR---------" > report.txt
echo "hostname: $(hostname)" >> report.txt
echo -e "interfaces:\n$(ip -4 --br a)"  >> report.txt
echo -e "VLAN:\n$(ip -d link show | grep -B 2 'vlan protocol 802.1Q')" | awk '!/link\/ether/'  >> report.txt
echo -e "NAT:\n$(sysctl -a | grep 'ip_forward ')"  >> report.txt
echo -e "$(timedatectl)"  >> report.txt
echo "---------END---------"  >> report.txt
clear
cat report.txt
```
```bash
(проверить sudo без пароля)
su - remote_user
sudo -i
exit
ssh -p 2026 192.168.0.2
(P@ssw0rd, проверить 2 попытки и баннер)
systemctl status bind dnsmasq
```

***BR-SRV***
```bash
echo -e "\n\n---------HQ-RTR---------" > report.txt
echo "hostname: $(hostname)" >> report.txt
echo -e "interfaces:\n$(ip -4 --br a)"  >> report.txt
echo -e "$(timedatectl)"  >> report.txt
echo "---------END---------"  >> report.txt
clear
cat report.txt
```
```bash
(проверить sudo без пароля)
su - remote_user
sudo -i
exit
ssh -p 2026 172.16.100.2
(P@ssw0rd, проверить 2 попытки и баннер)
```

***HQ-CLI***
```bash
echo -e "\n\n---------HQ-RTR---------" > report.txt
echo "hostname: $(hostname)" >> report.txt
echo -e "interfaces:\n$(ip -4 --br a)"  >> report.txt
echo -e "VLAN:\n$(ip -d link show | grep -B 2 'vlan protocol 802.1Q')" | awk '!/link\/ether/'  >> report.txt
echo -e "$(host hq-rtr)"  >> report.txt
echo -e "$(host hq-srv)"  >> report.txt
echo -e "$(host hq-cli)"  >> report.txt
echo -e "$(host br-rtr)"  >> report.txt
echo -e "$(host br-srv)"  >> report.txt
echo -e "$(host 172.16.100.1)"  >> report.txt
echo -e "$(host 172.16.100.2)"  >> report.txt
echo -e "$(host 172.16.200.1)"  >> report.txt
echo -e "$(host 172.16.200.2)"  >> report.txt
echo -e "$(host docker)"  >> report.txt
echo -e "$(host web)"  >> report.txt
echo -e "$(timedatectl)"  >> report.txt
echo "---------END---------"  >> report.txt
clear
cat report.txt
```
