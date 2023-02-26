Examples of using chcon command
1. To change a type of a web directory

# chcon -R -t httpd_sys_content_t /web/
2. To change a security context by using the reference file:

# chcon --reference=/tmp/file2 /tmp/file2
3. To set security context on files recursively:

# chcon -R httpd_sys_content_t /web/
4. To change files user security context:

# chcon -u mike_u /file
5. To change files role security context:

# chcon -u object_r /file
6. To change files type security context:

# chcon -u admin_home_t /file
7. To change files level security context:

# chcon -u s0 /file


<img src='https://www.oreilly.com/api/v2/epubs/9781789344875/files/assets/050bd30d-9268-49b6-bd13-0d28dada3106.png'/>

