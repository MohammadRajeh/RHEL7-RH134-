Exercise 2.0 - Creating Custom SELinux Policy

Exercise Description
In this exercise, we are going to download the code for the example application that we are going to write policy for, and build and install it onto our test system.

Step 1: Change directories
Create a src directory, in your home directory.

   # cd ~
   # mkdir src
   # cd src
Step 2: Check out the source code to the example application from GitHub
    Download the latest code release

   # git clone https://github.com/ajacocks/selinuxlab.git
   # cd selinuxlab
Step 3: Deploy the lab application, with Ansible
    Now, use Ansible to deploy the test application

   # ansible-playbook setup-testapp.yml
Step 4: Application and Service Info
    Take a look at the application, and the associated service, that you just installed

   # ls -l /usr/local/sbin/testapp
    -rwxr-xr-x. 1 root root 48688 Feb 20 18:03 /usr/local/sbin/testapp
   # sudo systemctl status testapp
    ● testapp.service - Testing SELinux app
        Loaded: loaded (/usr/lib/systemd/system/testapp.service; disabled; vendor preset: disabled)
        Active: inactive (dead)
Step 5: Start and enable the service
        Now, let’s file up the application

   # sudo systemctl enable testapp
   # sudo systemctl start testapp
    We can see that the app has been launched by systemd:

   # sudo systemctl status testapp
        ● testapp.service - Testing SELinux app
        Loaded: loaded (/usr/lib/systemd/system/testapp.service; enabled; vendor preset: disabled)
        Active: active (running) since Wed 2019-02-20 20:18:51 UTC; 2min 4s ago
        Process: 8496 ExecStart=/usr/local/sbin/testapp (code=exited, status=0/SUCCESS)
        Main PID: 8497 (testapp)
        CGroup: /system.slice/testapp.service
                └─8497 /usr/local/sbin/testapp

        Feb 20 20:20:06 ip-10-0-2-80.ec2.internal testapp[8497]: daemon ttl 999924
    And, we can also see that the app is running unconfined, which means that it is running without an SELinux policy, and can take any action that it wants to:

   # ps -efZ | grep testapp | grep -v grep
        system_u:system_r:unconfined_service_t:s0 root 9507 1  0 15:59 ?       00:00:00 /usr/local/sbin/testapp
