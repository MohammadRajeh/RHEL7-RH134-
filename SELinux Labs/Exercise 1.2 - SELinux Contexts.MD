Exercise 1.2 - SELinux Contexts
    Exercise Description
        All processes and files are labeled with an SELinux context. The context contains information that SELinux needs to make control decisions. Contexts contain user, role, type, and level information,

    Step 1: Examine a context
        To get started, let’s check the context of a file in the ec2-user user’s home directory:

        # ls -Z ~ec2-user/.vimrc
            unconfined_u:object_r:user_home_t:s0 /home/ec2-user/.vimrc

        As a side note, the -Z flag is present, in a wide variety of common CLI tools, including ls and ps. That flag indicates that SELinux context information should be displayed.

        Following the standard UNIX permissions, user and group information, the SELinux context attached to the file is displayed. The fields are SELinux user (unconfined_u), role (object_r), type (user_home_t), and level (s0).

        The SELinux user is not the same as the UNIX user, as it solely exists to associate a UNIX user to a set of roles. This allows UNIX users to be constrained by SELinux policy. In this case the unconfined_u user means that the user is mapped to the default SELinux login. This means that the user is allowed to launch any application that standard filesystem permissions allow. However, if that application has a defined domain transition, a confined domain will still apply to it.

    Step 2: Domain transitions
        To demonstrate the difference between confined and unconfined processes, let’s launch an application which doesn’t have a defined domain transition:

        # yes >/dev/null &
        # ps -Z | grep yes

        unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 29448 pts/0 00:00:07 yes

        # kill %1
        # ps -Z | grep yes
        # passwd ec2-user >/dev/null &
        # ps -Z | grep passwd

        unconfined_u:unconfined_r:passwd_t:s0-s0:c0.c1023 29498 pts/0 00:00:00 passwd

        [1]+  Stopped                 passwd ec2-user > /dev/null

        See that the third field is passwd_t, indicating that the passwd process has been confined by the passwd_t domain.

        Let’s kill that hanging passwd process, and move on:

        # kill %1

        This changing of the context, when an application in SELinux’s ruleset is launched, is called a domain transition.

Step 3: Useful utility #1: restorecon
        If I had to suggest to a new SELinux user the one thing that they should remember (no, it’s not setenforce 0!), it’d be restorecon. Restorecon will reset the SELinux context to what is defined in the system’s context database.

        To try this out, let’s purposefully set the context incorrectly on our example SELinux AVC log:
        #   chcon -t httpd_sys_content_t ./testaudit
        #   ls -Z ./testaudit

        system_u:object_r:httpd_sys_content_t:s0 ./testaudit

        other sample : # chcon --reference some_file.html index.html

        And we can change it back, magically, with restorecon. Note that this would have fixed the problem reported in our test AVC, by setting the context to httpd_sys_content_t:

        # restorecon -v ./testaudit
        # ls -Z ./testaudit
        
        Relabeled /root/testaudit from system_u:object_r:httpd_sys_content_t:s0 to system_u:object_r:admin_home_t:s0

        -rw-------. root root system_u:object_r:admin_home_t:s0 ./testaudit


        And we can change it back, magically, with restorecon. Note that this would have fixed the problem reported in our test AVC, by setting the context to httpd_sys_content_t:

        # restorecon -v ./testaudit
        # ls -Z ./testaudit
        
            Relabeled /root/testaudit from system_u:object_r:httpd_sys_content_t:s0 to system_u:object_r:admin_home_t:s0

            -rw-------. root root system_u:object_r:admin_home_t:s0 ./testaudit