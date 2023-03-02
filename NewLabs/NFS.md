Network File System
=====================================
RHL7 support NFSv4 by default,and falls back automatically to NFSV3 and V2 if that is not available.
- NFSV4 uses the TCP protocol to communication with the server, while older versions NFS may use aither TCP or UDP
- NFS requires rpcbind, which dynamically assigns ports for RPC services and can cause problems for /etc/sysconfig/nfs configuration file to control which ports
the required RPC service run on.
- Allow TCP and UDP port 2049 for NFS and allow TCP and UDP port 111 (rpcbind/surpc).
- All users can see the exported directories even if they don't have access.

==========Server Side============================================================


# yum install -y nfs-utils
# system status nfs-server
# systemctl enable nfs-server
# systemctl start nfs-server
# cd ~
# mkdir data
# vim /etc/exports
    /data   192.168.0/24(ro)
    /data   192.168.101(rw)
    /data   test.com(ro)
    /data   192.168.1.0/24(ro)   192.168.2.0/24(rw)
# firewall-cmd --permanent --add-service mountd
# firewall-cmd --permanent --add-service rpc-bind
# firewall-cmd --permanent --add-service nfs
# df -hT                                            (nfsv4)
================Client Side==================================================
# yum install -y nfs-utils
# showmount -e 192.168.1.100:/data  
# mkdir  mysharedata
# mount 192.168.1.100:/data     /mysharedata
# vim /etc/fstab
                                            default (not recommandded)
     192.168.1.100:/data   /root/data      nfs  _netdev     0   0  
    :wq
# df -h
# mount -a
# df -h


 

 ========================================================================================================
 Autofs:
 The automounter is a service (autofs) that can automatically mount BFS shares "on demand" and will automatically unmount NFS shares when they are no longoer being used.
 
 $$(Client Side)   yum install -y autofs
 $$(Client Side)  systemctl start autofs
 $$(Client Side)  systemctl enable autofs
 $$(Client Side)   vim /etc/auto.master
 /root/data   /etc/auto.data

 $$(Client Side) vim /etc/auto.data
 pub  -rw   192.168.1.100:/data

 where  pub is the target directory we want to access.
 $$(Client Side) systemctl restart autofs
$$(Client Side) cd data/pub
$$(Client Side) df -hT

$$(Client Side) vim /etc/autofs.conf
timeout = 300               (The idle user will be disconnected after 5 min by default )