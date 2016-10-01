
ExpressVPN Config KVM (el7)
-----

# kvm

> config bridge(s) and add vlans via nmtui
>
```
> nmtui
```
>
> install yum packages
>
```
yum -y groupinstall "Virtualization Tools"
yum -y install vim htop epel-release virt-install
yum -y install iftop mosh mtr
```
>
> set hostname
>
```
hostnamectl set-hostname kvm
```
>
> download centos
>
```
mkdir -p /var/storage/os/
curl -L http://centos.weepee.org/7.2.1511/isos/x86_64/CentOS-7-x86_64-Minimal-1511.iso > /var/storage/os/centos7.iso
```
>
> create vpn host
>
```
virt-install \
   --name=vpn-us \
   --controller type=scsi,model=virtio-scsi \
   --disk path=/var/lib/libvirt/images/vpn-us.dsk,size=8,sparse=true,cache=none,bus=scsi \
   --graphics vnc,listen=0.0.0.0,port=5950 \
   --network bridge=kvm0-vlan21 \
   --network bridge=kvm0-vlan50 \
   --vcpus=1 --ram=1024 \
   --cdrom=/var/storage/os/CentOS-7-x86_64-Minimal-1511.iso \
   --os-type=linux \
   --os-variant=rhel7
```
> connect with vnc to kvm host on port 5950
> start vhost
```
virsh start vpn-us
```
> autostart vhost after reboots
```
virsh autostart vpn-us
```

# vhost
> config bridge(s) and add vlans via nmtui
>
```
> nmtui
```
>
> install yum packages
>
```
yum -y install vim htop epel-release 
yum -y install iftop mosh iptables-services dnsmasq net-tools wget
yum -y update
```
>
> set hostname
>
```
hostnamectl set-hostname vpn-us
```
>
> enable ip forwarding
>
```
echo "net.ipv4.ip_forward=1" > /etc/sysctl.d/98-ipforwarding.conf
```
>
> add NAT rules to iptables and enable iptables on boot
>
```
iptables -A FORWARD -o eth0 -i tun0 -s 192.168.0.0/24 -m conntrack --ctstate NEW -j ACCEPT
iptables -A FORWARD  -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
iptables -t nat -F POSTROUTING
iptables -t nat -A POSTROUTING -o tun0 -j MASQUERADE
iptables-save > /etc/sysconfig/iptables
systemctl enable iptables
```
>
> install dnsmasq (dhcp) and enable
>
```
#dnsmasq
echo "interface=eth1
dhcp-range=192.168.0.50,192.1680.150,12h
conf-dir=/etc/dnsmasq.d" > /etc/dnsmasq.conf
systemctl enable dnsmasq
```
>
>  add static route so we can still get in after vpn connects
>  
```
echo "172.16.2.0/24 via 192.168.192.1" > /etc/sysconfig/network-scripts/route-eth0
```
>
>download expressvpn / activate / connect / enable autoconnect and enable on boot
>
```
wget https://download.expressvpn.xyz/clients/linux/expressvpn-1.1.0-1.x86_64.rpm
rpm -Uv expressvpn-1.1.0-1.x86_64.rpm
expressvpn activate
expressvpn connect wsd2
expressvpn autoconnect on
systemctl enable expressvpn
```

