# 1 step. Make sure you have installed nginx
    $ nginx -V
### after that make sure you have next line in output:
TLS SNI support enabled
# 2 step. Create folder for saving your key and certificate:
    mkdir /etc/nginx/ssl/example.com
### and go inside this folder:
    cd /etc/nginx/ssl/example.com
# 3 step. Lets create our RSA key and certificate!
### You need to create some password to have access to the certificate. After next command insert some password 2 times and you will get your RSA key:
    openssl genrsa -des3 -out server.key 1024
### Lets create certificate! Next command will create certificate but there requirement to insert some data, fell fre to insert in field some information, BUT! In field named "Common Name" insert the IP address of host that will be running over HTTPS
## Example:
    You are about to be asked to enter information that will be incorporated
    into your certificate request.
    What you are about to enter is what is called a Distinguished Name or a DN.
    There are quite a few fields but you can leave some blank
    For some fields there will be a default value,
    If you enter '.', the field will be left blank.
    -----
    Country Name (2 letter code) [AU]:UA
    State or Province Name (full name) [Some-State]:Lviv Region
    Locality Name (eg, city) []:Lviv
    Organization Name (eg, company) [Internet Widgits Pty Ltd]:My Inc
    Organizational Unit Name (eg, section) []:Some unit name
    Common Name (e.g. server FQDN or YOUR name) []:192.168.103.172
    Email Address []:somemail@example.com
### after that will be 2 fields that no need to insert.. Just click Enter button 2 times.
# 4 step. Lets erase password, that no needed now.
    cp server.key server.key.org
    openssl rsa -in server.key.org -out server.key
# 5 step. We need to sign our certificate, here 365 it is number of days that certificate will be valid
    openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt
# 6 step. Congratulations you created certificate and signed it! But need to edit to confiure file on nginx this changes.
### go to the default configuration file and add certificate. It should look like this:
    
    ...
    server {
        listen 443 ssl;           #here change 80 to 443 to listen to HTTPS requests
        ssl on;                   #enable ssl library
        ssl_certificate /etc/nginx/ssl/example.com/server.crt;             # put here full path to generated certificate (server.crt)
        ssl_certificate_key /etc/nginx/ssl/example.com/server.key;         # here should be full path to your key (server.key)
        ssl_protocols TLSv1.1 TLSv1.2;                                     # this line allow you to use TLS protocols instead of SSL
        ssl_ciphers AES128-SHA:AES256-SHA:RC4-SHA:DES-CBC3-SHA:RC4-MD5;    #this line describes which of encryption methods is used
            location / {
                proxy_pass http://project;
    }
    ...
# 7 step. Reload your nginx 
    #on Centos 7:
    $ systemctl reload nginx
