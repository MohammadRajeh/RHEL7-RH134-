Introduction to SELinux:
================================================================

- May <subject> do <action> to <object>?
- May a web server access files in users home directories?

- Discretionary Access Control(DAC):
     Linux standard access policy based on user,group and other permissions
     Does not enable system admins to create comprehensive and fine-grained security policies.

- Security Enhanced Linux(SELinux) implements Mandatory Access Control (MAC).
- Every process and system resource has a special security label called an SELinux context.
- SELinux contexts have several fields:
    * user
    * role
    * type
    * security level
Examples:
    # ls -Z
    # ps auZ
    # id -z
    # ps auxZ
    # ps aux -Z
    # ls -Z
Benefits of running SELinux:

    - All Processes and files are labeled ===> Access is only allowed if an SELinux policy rule allowed  it.
    - SELinux access decisions are based on user, role, type and optionally security level.
    - SELinux policy is administratively-defined and enforced system-wide.

Example:
     - If the Apache HTTP server is compromised, an attacker cannot use that processto read files in user home directories, unless a specific SELinux policy rule was added or configured to allow such access.
     
   
<p align="center">
<img width="50%" src="https://access.redhat.com/webassets/avalon/d/Red_Hat_Enterprise_Linux-8-Using_SELinux-en-US/images/f7b4d4a7ee54ec0153a3422060470895/selinux-intro-apache-mariadb.png">
</p>

- SELinux architecture and packages:
    -  SELinux is a Linux Security Module (LSM) that is built into the Linux kernel.
    -  SELinux subsystem in the kernel is driven by a security policy which is controlled by the administrator and loaded at boot.
    -  All security-relevant, kernel-level access operations on the system are intercepted by SELinux and examined in the context of the loaded security policy. 
    -  If the loaded policy allows the operation, it continues. Otherwise, the operation is blocked and the process receives an error.

    - SELinux decisions, such as allowing or disallowing access, are cached. This cache is known as the Access Vector Cache (AVC). 
    - cat /var/log/audit/audit.log
    - The systemd daemon also works as an SELinux Access Manager. 
    
Important:
        
    - To avoid incorrect SELinux labeling and subsequent problems, ensure that you start services using a systemctl start command.

RHEL 8 provides the following packages for working with SELinux:

    - policies: selinux-policy-targeted, selinux-policy-mls( Multi Level Security protection)
    - tools: policycoreutils, policycoreutils-gui, libselinux-utils, policycoreutils-python-utils, setools-console, checkpolicy

SELinux states and modes:
    - Enforcing mode is the default
    - In permissive mode
    - Disabled mode is strongly discouraged

    view the current SELinux mode:

    # getenforce
    Enforcing
    # setenforce 0
    # getenforce
    Permissive
    # setenforce 1
    # getenforce
    Enforcing


In Red Hat Enterprise Linux, you can set individual domains to permissive mode while the system runs in enforcing mode. For example, to make the httpd_t domain permissive:

    # semanage permissive -a httpd_t

Changing SELinux states and modes
    When enabled, SELinux can run in one of two modes: enforcing or permissive. The following sections show how to permanently change into these modes.

Use the getenforce or sestatus commands to check in which mode SELinux is running. The getenforce command returns   Enforcing, Permissive, or Disabled.

The sestatus command returns the SELinux status and the SELinux policy being used:

    # sestatus
    SELinux status:                 enabled
    SELinuxfs mount:                /sys/fs/selinux
    SELinux root directory:         /etc/selinux
    Loaded policy name:             targeted
    Current mode:                   enforcing
    Mode from config file:          enforcing
    Policy MLS status:              enabled
    Policy deny_unknown status:     allowed
    Memory protection checking:     actual (secure)
    Max kernel policy version:      31

Warning:
    # fixfiles -F onboot
    /.autorelabel
    set SELINUX=permissive in /etc/selinux/config
    Run 
        # fixfiles onboot or 
        # touch /.autorelabel
        # reboot
    check the 
        # cat /var/log/messages 
        and 
        # cat /var/log/audit/audit.log 
        for new SELinux messages
        # set SELINUX=enforcing in /etc/selinux/config
        # Run fixfiles onboot or touch /.autorelabel
        # reboot



Changing to permissive mode
    - Prerequisites

        The selinux-policy-targeted, libselinux-utils, and policycoreutils packages are installed on your system.
        The selinux=0 or enforcing=0 kernel parameters are not used.
    
    - Procedure

        Open the /etc/selinux/config file in a text editor of your choice, for example:

        # vim /etc/selinux/config
    
    - Configure the SELINUX=permissive option:

        # This file controls the state of SELinux on the system.
        # SELINUX= can take one of these three values:
        #       enforcing - SELinux security policy is enforced.
        #       permissive - SELinux prints warnings instead of enforcing.
        #       disabled - No SELinux policy is loaded.
        SELINUX=(permissive or enforcing or disabled )
        # SELINUXTYPE= can take one of these two values:
        #       targeted - Targeted processes are protected,
        #       mls - Multi Level Security protection.
        SELINUXTYPE=targeted

    very important:
        This creates the /.autorelabel file containing the -F option.
        # fixfiles -F onboot
        # touch /.autorelabel
        # reboot
    - Restart the system:

        # reboot
    - Verification

        After the system restarts, confirm that the getenforce command returns Permissive:

        $ getenforce
        Permissive

Note:   
    After changing to enforcing mode:
        # ausearch -m AVC,USER_AVC,SELINUX_ERR,USER_SELINUX_ERR -ts today
    Alternatively, with the setroubleshoot-server package installed, enter:
        # grep "SELinux is preventing" /var/log/messages
    If SELinux is active and the Audit daemon (auditd) is not running on your system, then search for certain SELinux messages in the output of the dmesg command:
        # dmesg | grep -i -e type=1300 -e type=1400




Managing confined and unconfined users
    Confined and unconfined users
    Each Linux user is mapped to an SELinux user using SELinux policy. This allows Linux users to inherit the restrictions on SELinux users.

    To see the SELinux user mapping on your system, use the semanage login -l command as root:
        # semanage login -l
        Login Name           SELinux User         MLS/MCS Range        Service

        __default__          unconfined_u         s0-s0:c0.c1023       *
        root                 unconfined_u         s0-s0:c0.c1023       *

    Confined and unconfined Linux users are subject to executable and writable memory checks, and are also restricted by MCS or MLS.

    To list the available SELinux users, enter the following command:

        $ seinfo -u
        Users: 8
        guest_u
        root
        staff_u
        sysadm_u
        system_u
        unconfined_u
        user_u
        xguest_u
         
    Note that the seinfo command is provided by the setools-console package, which is not installed by default.


    Confined administrator roles in SELinux
        auditadm_r
            The audit administrator role allows managing the Audit subsystem.
        dbadm_r
            The database administrator role allows managing MariaDB and PostgreSQL databases.
        logadm_r
            The log administrator role allows managing logs.
        webadm_r
            The web administrator allows managing the Apache HTTP Server.
        secadm_r
            The security administrator role allows managing the SELinux database.
        sysadm_r
            The system administrator role allows doing everything of the previously listed roles and has additional privileges. 



        You can customize the permissions for confined users in your SELinux policy according to specific needs by adjusting booleans in the policy. You can determine the current state of these booleans by using the semanage boolean -l command. To list all SELinux users, their SELinux roles, and MLS/MCS levels and ranges, use the semanage user -l command as root.

    The sysadm_u user cannot log in directly using SSH. To enable SSH logins for sysadm_u, set the ssh_sysadm_login boolean to on:

    # setsebool -P ssh_sysadm_login on

    These roles determine what SELinux allows the user to do:

    webadm_r: can only administrate SELinux types related to the Apache HTTP Server.
    dbadm_r: can only administrate SELinux types related to the MariaDB database and the PostgreSQL database management system.
    logadm_r: can only administrate SELinux types related to the syslog and auditlog processes.
    secadm_r: can only administrate SELinux.
    auditadm_r: can only administrate processes related to the Audit subsystem

    To list all available roles, enter the seinfo -r command
    $ seinfo -r
        Roles: 14
        auditadm_r
        dbadm_r
        guest_r
        logadm_r
        nx_server_r
        object_r
        secadm_r
        staff_r
        sysadm_r
        system_r
        unconfined_r
        user_r
        webadm_r
        xguest_r
    
Note:
    Adding a new user automatically mapped to the SELinux unconfined_u user

        # useradd example.user
        # passwd example.user
        Changing password for user example.user.
        New password:
        Retype new password:
        passwd: all authentication tokens updated successfully.

        signout 
        signin using new user
        
        $ id -Z
        unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023


Add a new user as an SELinux-confined user:
        # useradd -Z staff_u example.user
        # passwd example.user
        Changing password for user example.user.
        New password:
        Retype new password:
        passwd: all authentication tokens updated successfully.

    Verification
        $ id -Z
        uid=1000(example.user) gid=1000(example.user) groups=1000(example.user) context=staff_u:staff_r:staff_t:s0-s0:c0.c1023

Map the __default__ user, which represents all users without an explicit mapping, to the user_u SELinux user:

        # semanage login -m -s user_u -r s0 __default__


Confining an administrator by mapping to sysadm_u
    # setsebool -P ssh_sysadm_login on
    # adduser -G wheel -Z sysadm_u example.user
Optional: Map an existing user to the sysadm_u SELinux user and add the user to the wheel user group:

    # usermod -G wheel -Z sysadm_u example.user

Verification

    Check that example.user is mapped to the sysadm_u SELinux user:

        # semanage login -l | grep example.user
        example.user     sysadm_u    s0-s0:c0.c1023   *
        Log in as example.user, for example, using SSH, and show the user’s security context:

        [example.user@localhost ~]$ id -Z
        sysadm_u:sysadm_r:sysadm_t:s0-s0:c0.c1023
        Switch to the root user:

        $ sudo -i
        [sudo] password for example.user:
        Verify that the security context remains unchanged:

        # id -Z
        sysadm_u:sysadm_r:sysadm_t:s0-s0:c0.c1023
        Try an administrative task, for example, restarting the sshd service:

        # systemctl restart sshd
        If there is no output, the command finished successfully.

        If the command does not finish successfully, it prints the following message:

        Failed to restart sshd.service: Access denied
        See system logs and 'systemctl status sshd.service' for details.

Confining an administrator using sudo and the sysadm_r role

Procedure

    Create a new user, add the user to the wheel user group, and map the user to the staff_u SELinux user:

    # adduser -G wheel -Z staff_u example.user
    Optional: Map an existing user to the staff_u SELinux user and add the user to the wheel user group:

    # usermod -G wheel -Z staff_u example.user
    To allow example.user to gain the SELinux administrator role, create a new file in the /etc/sudoers.d/ directory, for example:

    # visudo -f /etc/sudoers.d/example.user
    Add the following line to the new file:

    example.user ALL=(ALL) TYPE=sysadm_t ROLE=sysadm_r ALL

Verification

    Check that example.user is mapped to the staff_u SELinux user:

    # semanage login -l | grep example.user
    example.user     staff_u    s0-s0:c0.c1023   *
    Log in as example.user, for example, using SSH, and switch to the root user:

    [example.user@localhost ~]$ sudo -i
    [sudo] password for example.user:
    Show the root security context:

    # id -Z
    staff_u:sysadm_r:sysadm_t:s0-s0:c0.c1023
    Try an administrative task, for example, restarting the sshd service:

    # systemctl restart sshd
    If there is no output, the command finished successfully.

    If the command does not finish successfully, it prints the following message:

    Failed to restart sshd.service: Access denied
    See system logs and 'systemctl status sshd.service' for details.


Creating and enforcing an SELinux policy for a custom application

    vim mydaemon.c
    #include <unistd.h>
    #include <stdio.h>

    FILE *f;

    int main(void)
    {
    while(1) {
    f = fopen("/var/log/messages","w");
            sleep(5);
            fclose(f);
        }
    }
Compile the file:

    $ gcc -o mydaemon mydaemon.c

Create a systemd unit file for your daemon:
    # vim mydaemon.service
    [Unit]
    Description=Simple testing daemon

    [Service]
    Type=simple
    ExecStart=/usr/local/bin/mydaemon

    [Install]
    WantedBy=multi-user.target



Install and start the daemon:

# cp mydaemon /usr/local/bin/
# cp mydaemon.service /usr/lib/systemd/system
# systemctl start mydaemon
# systemctl status mydaemon
● mydaemon.service - Simple testing daemon
   Loaded: loaded (/usr/lib/systemd/system/mydaemon.service; disabled; vendor preset: disabled)
   Active: active (running) since Sat 2020-05-23 16:56:01 CEST; 19s ago
 Main PID: 4117 (mydaemon)
    Tasks: 1
   Memory: 148.0K
   CGroup: /system.slice/mydaemon.service
           └─4117 /usr/local/bin/mydaemon

May 23 16:56:01 localhost.localdomain systemd[1]: Started Simple testing daemon


Check that the new daemon is not confined by SELinux:

    $ ps -efZ | grep mydaemon
    system_u:system_r:unconfined_service_t:s0 root 4117    1  0 16:56


Generate a custom policy for the daemon:

    $ sepolicy generate --init /usr/local/bin/mydaemon
    Created the following files:
    /home/example.user/mysepol/mydaemon.te # Type Enforcement file
    /home/example.user/mysepol/mydaemon.if # Interface file
    /home/example.user/mysepol/mydaemon.fc # File Contexts file
    /home/example.user/mysepol/mydaemon_selinux.spec # Spec file
    /home/example.user/mysepol/mydaemon.sh # Setup Script


Rebuild the system policy with the new policy module using the setup script created by the previous command:
    yum install rpm-build -y

    # ./mydaemon.sh
    Building and Loading Policy
    + make -f /usr/share/selinux/devel/Makefile mydaemon.pp
    Compiling targeted mydaemon module
    Creating targeted mydaemon.pp policy package
    rm tmp/mydaemon.mod.fc tmp/mydaemon.mod
    + /usr/sbin/semodule -i mydaemon.pp

Note that the setup script relabels the corresponding part of the file system using the restorecon command:

    # restorecon -v /usr/local/bin/mydaemon /usr/lib/systemd/system

Restart the daemon, and check that it now runs confined by SELinux:
    systemctl restart mydaemon
    ps -efZ | grep mydaemon
    system_u:system_r:mydaemon_t:s0 root        8150       1  0 17:18 ?        00:00:00 /usr/local/bin/mydaemon

Because the daemon is now confined by SELinux, SELinux also prevents it from accessing /var/log/messages. Display the corresponding denial message:

    # ausearch -m AVC -ts recent
    ...
    type=AVC msg=audit(1590247112.719:5935): avc:  denied  { open } for  pid=8150 comm="mydaemon" path="/var/log/messages" dev="dm-0" ino=2430831 scontext=system_u:system_r:mydaemon_t:s0 tcontext=unconfined_u:object_r:var_log_t:s0 tclass=file permissive=1
    ...


You can get additional information also using the sealert tool:

    $ sealert -l "*"
SELinux is preventing mydaemon from open access on the file /var/log/messages.

 Plugin catchall (100. confidence) suggests *

    If you believe that mydaemon should be allowed open access on the messages file by default.
    Then you should report this as a bug.
    You can generate a local policy module to allow this access.
    Do
    allow this access for now by executing:
    # ausearch -c 'mydaemon' --raw | audit2allow -M my-mydaemon
    # semodule -X 300 -i my-mydaemon.pp

    Additional Information:
    Source Context                system_u:system_r:mydaemon_t:s0
    Target Context                unconfined_u:object_r:var_log_t:s0
    Target Objects                /var/log/messages [ file ]
    Source                        mydaemon

    ...


Use the audit2allow tool to suggest changes:

$ ausearch -m AVC -ts recent | audit2allow -R

require {
	type mydaemon_t;
}

#============= mydaemon_t ==============
logging_write_generic_logs(mydaemon_t)



Because rules suggested by audit2allow can be incorrect for certain cases, use only a part of its output to find the corresponding policy interface:

    #   grep -r "logging_write_generic_logs" /usr/share/selinux/devel/include/ | grep .if
    /usr/share/selinux/devel/include/system/logging.if:interface(`logging_write_generic_logs',`)

Check the definition of the interface:

$ cat /usr/share/selinux/devel/include/system/logging.if
...
interface(`logging_write_generic_logs',`
        gen_require(`
                type var_log_t;
        ')

        files_search_var($1)
        allow $1 var_log_t:dir list_dir_perms;
        write_files_pattern($1, var_log_t, var_log_t)
')
...

In this case, you can use the suggested interface. Add the corresponding rule to your type enforcement file:

    $ echo "logging_write_generic_logs(mydaemon_t)" >> mydaemon.te
    Alternatively, you can add this rule instead of using the interface:

    $ echo "allow mydaemon_t var_log_t:file { open write getattr };" >> mydaemon.te
    Reinstall the policy:

        # ./mydaemon.sh
        Building and Loading Policy
        + make -f /usr/share/selinux/devel/Makefile mydaemon.pp
        Compiling targeted mydaemon module
        Creating targeted mydaemon.pp policy package
        rm tmp/mydaemon.mod.fc tmp/mydaemon.mod
        + /usr/sbin/semodule -i mydaemon.pp
        ...
Verification

    Check that your application runs confined by SELinux, for example:

    $ ps -efZ | grep mydaemon
    system_u:system_r:mydaemon_t:s0 root        8150       1  0 17:18 ?        00:00:00 /usr/local/bin/mydaemon
    Verify that your custom application does not cause any SELinux denials:

    # ausearch -m AVC -ts recent
    <no matches>




