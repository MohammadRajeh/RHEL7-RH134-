Exercise Description
    In this final exercise, we will finish the testapp SELinux policy.

Section 1: A Last-ish Interface
Step 1: Check for AVC denials
    Let’s again restart our app, to get an updated list of denials.

   # systemctl restart testapp
        We only have a few denials left, and you will see that they all (probably!) reference /etc/resolv.conf or to /etc/hosts.

   # ausearch -m AVC -ts recent | egrep '^type=AVC'
        type=AVC msg=audit(1553195947.854:3270): avc:  denied  { getattr } for  pid=4651 comm="testapp" path="/etc/resolv.conf" dev="dm-0" ino=9311517 scontext=system_u:system_r:testapp_t:s0 tcontext=system_u:object_r:net_conf_t:s0 tclass=file permissive=1
        type=AVC msg=audit(1553195947.854:3271): avc:  denied  { open } for  pid=4651 comm="testapp" path="/etc/hosts" dev="dm-0" ino=8389746 scontext=system_u:system_r:testapp_t:s0 tcontext=system_u:object_r:net_conf_t:s0 tclass=file permissive=1
        type=AVC msg=audit(1553195947.854:3271): avc:  denied  { read } for  pid=4651 comm="testapp" name="hosts" dev="dm-0" ino=8389746 scontext=system_u:system_r:testapp_t:s0 tcontext=system_u:object_r:net_conf_t:s0 tclass=file permissive=1
Step 2: Interpret AVC messages
    From the AVCs that we have, it seems clear that the application wants to try to resolve some hostnames to IP addresses. Most applications that access the network will need to be able to read system network configuration, and the interface for that is in /usr/share/selinux/devel/include/system/sysnetwork.if and is called sysnet_read_config.

Step 3: Add the interface
    So, as before, add an interface to your testapp.te file, using a line like this:

    sysnet_read_config(testapp_t)
    And compile your policy, and restart the service:

   # ./testapp.sh
   # systemctl restart testapp
        …​and last, check to see if there are still any AVC denials:

   # systemctl restart testapp
   # ausearch -m AVC -ts recent | wc -l
    14

Section 2: A Last Interface (no, really!)
Step 1: Check for AVC denials
    Let’s get an updated list of denials:

   # ausearch -m AVC -ts recent | egrep '^type=AVC'

Step 2: Find an interface
        Let’s search for an interface, referencing ssl certificate and read as that’s what’s being reported:

   # find /usr/share/selinux/devel/include -type f -name "*.if" -exec grep -iH 'ssl certificate' {} \; | grep -i read

Step 3: Add the interface
        So, as before, add the interface to your testapp.te file, using a line like this:

    miscfiles_read_generic_certs(testapp_t)
Step 4: Recompile the policy
    Now, let’s recompile the policy, and reload it into memory.

   # ./testapp.sh
Step 5: Restart the application
    To see if that fixed the problem, let’s restart the application:

   # systemctl restart testapp
    …​and see if there are any AVC messages left:

   # ausearch -m AVC -ts recent | egrep 'tcp|udp' | wc -l


Section 3: Set Enforcing Mode
Step 1: Change the Domain to Enforcing
    The last step that we need to take is to change our testapp.te file, so that the domain is enforcing. All we need to do, to accomplish this, is to comment out the line that says:

    permissive testapp_t;
Once we do that, the final version of your policy should look like this:

  # cat testapp.te


Step 3: Recompile and reload the policy
    Now, let’s recompile the policy, and reload it into memory.

  # ./testapp.sh
Step 4: Restart the application
    To see if that fixed the problem, let’s restart the application:

  # systemctl restart testapp
    …​and see if there are any AVC messages left:

  # ausearch -m AVC -ts recent | egrep 'tcp|udp' | wc -l