 #  ip link show

#   nmcli connection add con-name  main-con ifname ens160 type ethernet
#    nmcli con modify main-con ipv4.addresses 192.168.20.1/24
#    nmcli con modify main-con ipv6.addresses 2001:db8:1::1/64
#    nmcli con modify main-con ipv4.method manual 
#  nmcli con modify main-con ipv6.method manual 

#    nmcli con modify main-con ipv4.gateway 192.168.20.254
#    nmcli con modify main-con ipv6.gateway 2001:db8:1::f


#    nmcli con modify main-con ipv4.dns 192.168.20.254
#    nmcli con modify main-con ipv4.dns-search coderz.com
#    nmcli con modify main-con ipv6.dns 2001:db8:1::f
#   nmcli con modify main-con ipv6.dns-search coderz.com
#    nmcli conn up  main-con
#    nmcli device status 


#    mount /dev/sr0 /media/ 


# vim /etc/yum.repos.d/local.repo
# yum -y install vsftpd
# systemctl enable vsftpd
#   systemctl start vsftpd
#   vim /etc/vsftpd/vsftpd.conf 

# firewall-cmd --permanent --add-port=21/tcp
# firewall-cmd --permanent --add-service=ftp
# firewall-cmd --reload
# systemctl restart vsftpd
#   chcon -R -t public_content_t /var/ftp/pub/
# semanage boolean -m ftpd_full_access --on
# systemctl restart vsftpd
