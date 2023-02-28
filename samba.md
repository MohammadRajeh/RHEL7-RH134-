
RUN FTP Server to use REPO

install vsftpd 
chkconfig vsftpd on
vim /etc/vsftpd/vsftpd.conf
anonymous_enable = YES
local_enable = YES
chcon -R -t public_content_t /var/ftp/pub/
systemctl restart  vsftpd
systemctl enable firewalld
systemctl start firewalld
firewall-cmd --list-service 
 
firewall-cmd --permanent --add-port=21/tcp
firewall-cmd --permanent --add-service=ftp
firewall-cmd --reload
systemctl restart  vsftpd

The Samba project 
    provides file sharing and print services for computers on a network. It uses the Server Message Block and Common Internet File System (SMB/CIFS) protocol

    so the services created by running Samba are available to Linux, macOS, and Windows clients. It's an essential service to run in organizations that support multiple operating systems, and it's even useful on homogenous networks.

Install Samba
On your designated Samba server, install the Samba package:

# sudo dnf install samba
This command also installs the samba-common-tools and samba-libs packages.

Next, start the SMB and NMB daemons. The SMB daemon manages most Samba services, while the NMB daemon provides NetBIOS services. Here are the commands:

# systemctl enable --now smb

# systemctl enable --now nmb


Configure your firewall
Make sure that your file-share server is accessible over your network by adding the samba service to your firewall config:

# sudo systemctl enable --now firewalld

# sudo firewall-cmd --list-services
cockpit dhcpv6-client ssh

# sudo firewall-cmd --add-service samba
# firewall-cmd --reload
success
Configure Samba
Create a directory on the server to hold your shared files and folders, and change the SELinux context to samba_share_t:

# sudo mkdir /sambashare

# sudo chcon -t samba_share_t /sambashare/


To configure shares and users, edit the /etc/samba/smb.conf file. The

[global]
  workgroup = SAMBA
  security = user
  passdb backend = tdbsam
...
Create a shared location
To create a new share location, add a section to the /etc/samba/smb.conf configuration file with these two definitions:

[sambashare]
   path = /sambashare
   read only = No
You can test your Samba configuration with the testparm command:

# testparm /etc/samba/smb.conf
Add following lines to config file 
# netbios name = Server
# log file  = /var/log/samba/%m.log
# log level =1
# [sambashare]
# path = /sambashare
# read only = no


Add users
Users logging into a Samba share must either have accounts on the server or log in as a guest. You can also use standard Unix groups to manage access. For instance, run the following command to create a group of staff members who need access to the server:

#  sudo groupadd staff
Assuming you need to add a staff member named tux to your Samba server, the process is initially the same as usual:

#  sudo adduser -g staff --create-home tux

#  passwd tux 
You must also set a dedicated Samba password:

#  sudo smbpasswd -a tux

Restart and test Samba
To load in the new configuration, restart Samba:

#  sudo systemctl restart {s,n}mb
Samba is now serving sambashare to any authenticated user. You can test it using the smbclient command:
# smbcontrol all reload-config
#  smbclient -L //sambaserver
Enter SAMBA\tux's password: 

Sharename    Type   Comment
---------    ----   -------
sambashare   Disk
print$       Disk   Printer Drivers
IPC$         IPC    IPC Service (Samba 4.14.5)
tux          Disk   Home Directories

