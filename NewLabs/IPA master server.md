# Setting LDAP Server Using Open IPA:
- LDAP (Lightweight Directory Access Protocol) Server is used as a centralized authentication server
- LDAP as also used to store accounts, permissions, ACL, quota and more.
- Kerberos provides SSO authentication service using TGT (Ticket Granting Ticket)


=====================Server Side=============================================

# yum install ipa-server bind bind-dyndb-ldap
# ipa-server-install --setup-dns
Server host name [masterSRV.coderz.com]:        (must all char lowercase)


Install the packages needed,

Raw
# yum install ipa-server bind bind-dyndb-ldap
Run the ipa-server-install script and .

Raw
# ipa-server-install --setup-dns <dns-address> 
Enter the hostname. This is determined automatically using reverse D NS.

Raw
Server host name [ipaserver.example.com]:
Enter the domain name. This is determined automatically based on the hostname.

Raw
Please confirm the domain name [example.com]:
Enter the new Kerberos realm name. This is usually based on the domain name.

Raw
Please provide a realm name [EXAMPLE.COM]:
Enter the password for the Directory Server superuser, cn=Directory Manager. There are password strength requirements for this password, including a minimum password length(eight characters).

Raw
Directory Manager password:
Password (confirm):
The script then reprints the hostname, IP address, and domain name. Confirm that the information is correct.

Raw
The IPA Master Server will be configured with
Hostname:
ipaserver.example.com
IP address: 192.168.1.1
Domain name: example.com
Realm name: EXAMPLE.COM
Continue to configure the system with these values? [no]: yes
After that, the script configures all of the associated services for IdM, with task counts and progress bars.

Raw
Configuring NTP daemon (ntpd)
[1/4]: stopping ntpd
...
Done configuring NTP daemon (ntpd).
Configuring directory server (dirsrv): Estimated time 1 minute
[1/38]: creating directory server user
.
.
.
Restarting the web server
Setup complete
Restart the SSH service to retrieve the Kerberos principal and to refresh the name server switch(NSS) configuration file:

Raw
# service sshd restart
Authenticate to the Kerberos realm using the admin user's credentials to ensure that the user is properly configured and the Kerberos realm is accessible.

Raw
# kinit admin
Password for admin@ EXAMPLE.COM:
#
(If authentication is wrong,it will return "kinit: Password incorrect while getting initial credentials")
Test the IdM configuration by running a command like ipa user-find . For example:

Raw
# ipa user-find admin
-------------
1 user matched
-------------
User login: admin
Last name: Administrator
Home directory: /home/admin
Login shell: /bin/bash
UID: 939000000
GID: 939000000
Account disabled: False
Password: True
Kerberos keys available: True
----------------------------
Number of entries returned 1
----------------------------

# kinit
# klist
# ipa help topics
# ipa user-add  --first=Tim --last=User --password tuser1 --manager=admin
# ipa help automember
# ipa help commands
# ipa automember-add --help
#  ipa permission-add --permissions=read --permissions=write --permissions=delete
#  ipa permission-add --permissions={read,write,delete}
#  ipa certprofile-show certificate_profile --out=exported\*profile.cfg
To list all users:
# ipa user-find
---------------
4 users matched
---------------
  ...
To list user groups whose specified attributes contain keyword:
# ipa group-find keyword
----------------
2 groups matched
----------------
  ...

  # ipa group-find --user=user_name

# ipa group-find --no-user=user_name
# ipa host-find 


========================Client Side=======================
