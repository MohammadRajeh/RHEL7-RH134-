Exercise 1.1 - Introduction to SELinux
    Step 1: Become root
        First, go ahead and switch users to root:

        # sudo -i
    Step 2: What mode are we in?
        Next, let’s check to see what SELinux mode your host is in:

        # getenforce
            Enforcing
    Step 3: Changing modes
        Now, we can change the mode that your host is in:

        # setenforce 0
        # getenforce
            Permissive
        And we can change it back:

        # setenforce 1
        # getenforce
            Enforcing
    Step 4: Changing modes for specific domains
        You can also set individual SELinux domains (like httpd_t, which controls web server access control) to permissve mode:

        # semanage permissive -a httpd_t
        # semanage permissive -l
            Builtin Permissive Types

            Customized Permissive Types

            httpd_t
        And again, we can change it back:

        # semanage permissive -d httpd_t
            libsemanage.semanage_direct_remove_key: Removing last permissive_httpd_t module (no other permissive_httpd_t module exists at another priority).
            
        The permissive mode means that either the entire system (the first example) or the specific application constrained by a type (the second example) are allowed to bypass SELinux access controls. However, even though access controls are disabled, full AVC (access vector cache, where SELinux stores its decisions) logging still happens.

    Step 5: SELinux logging
        SELinux messages are logged to the /var/log/audit/audit.log file. We can read the messages with this command:
        # ausearch -m MAC_POLICY_LOAD -i --just-one

        We have asked the ausearch utility to look for SELinux policies being loaded (-m MAC_POLICY_LOAD), to make the result human-readable (-i), and to return only a single result (--just-one):

        ----
        type=PROCTITLE msg=audit(05/17/2019 23:47:54.147:1896) : proctitle=/sbin/load_policy
        type=SYSCALL msg=audit(05/17/2019 23:47:54.147:1896) : arch=x86_64 syscall=write success=yes exit=3857474 a0=0x4 a1=0x7fb57e597000 a2=0x3adc42 a3=0x7ffc8bb993a0 items=0 ppid=16567 pid=16572 auid=ec2-user uid=root gid=root euid=root suid=root fsuid=root egid=root sgid=root fsgid=root tty=pts0 ses=1 comm=load_policy exe=/usr/sbin/load_policy subj=unconfined_u:unconfined_r:load_policy_t:s0-s0:c0.c1023 key=(null)
        type=MAC_POLICY_LOAD msg=audit(05/17/2019 23:47:54.147:1896) : auid=ec2-user ses=1 lsm=selinux res=yes
    
        Other useful searches include:

        all denials:

        # ausearch -m avc,user_avc,selinux_err,user_selinux_err
            denials from today:

        # ausearch -m avc -ts today
            denials from the last 10 minutes:

        # ausearch -m avc -ts recent

            You can try these, if you like, but they may or may not return any results.

    Step 6: Interpreting AVCs
            If you use ausearch to find AVC messages, you can see exactly what caused SELinux to deny access:
        # ausearch -if ./testaudit -m AVC

        ---
            time->Mon Nov 17 00:45:36 2008
            type=SYSCALL msg=audit(1226882736.442:86): arch=40000003 syscall=196 success=no exit=-13 a0=b9a1e198 a1=bfc2921c a2=54dff4 a3=2008171 items=0 ppid=2425 pid=2427 auid=502 uid=48 gid=48 euid=48 suid=48 fsuid=48 egid=48 sgid=48 fsgid=48 tty=(none) ses=4 comm="httpd" exe="/usr/sbin/httpd" subj=unconfined_u:system_r:httpd_t:s0 key=(null)
            type=AVC msg=audit(1226882736.442:86): avc:  denied  { getattr } for  pid=2427 comm="httpd" path="/var/www/html/file1" dev=dm-0 ino=284133 scontext=unconfined_u:system_r:httpd_t:s0 tcontext=unconfined_u:object_r:samba_share_t:s0 tclass=file


        Here, we can see that the httpd process, ID 2427, was disallowed from checking attributes of the file /var/www/html/file1. Additionally, we can see that the domain of the process was httpd_t, and the file was labeled with the samba_share_t context.

    Step 7: Getting advice

        There is a neat utility called audit2why, which can make suggestions as to how you might solve SELinux AVC messages. Let’s try it on our test error:

        # ausearch -if ./testaudit -m AVC | audit2why
            type=AVC msg=audit(1226882736.442:86): avc:  denied  { getattr } for  pid=2427 comm="httpd" path="/var/www/html/file1" dev=dm-0 ino=284133 scontext=unconfined_u:system_r:httpd_t:s0 tcontext=unconfined_u:object_r:samba_share_t:s0 tclass=file

	        Was caused by:
		        Missing type enforcement (TE) allow rule.

		        You can use audit2allow to generate a loadable module to allow this access.
    