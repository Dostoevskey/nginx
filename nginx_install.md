# 1. Add nginx repository:
	$ sudo yum install epel-release
# 2. Install nginx (on Centos 7)
	$ sudo yum install nginx
# 3. We need say to SElinux that he need to allow HTTP trafic:
	$ setsebool -P httpd_can_network_connect 1
# 4. Start nginx and add him to autorun:
	$ sudo systemctl start nginx
	$ sudo systemctl enable nginx
# 5. If you want to allow HTTP and HTTPS trafic do next commands in terminal:
	$ sudo firewall-cmd --permanent --zone=public --add-service=http 
	$ sudo firewall-cmd --permanent --zone=public --add-service=https
	$ sudo firewall-cmd --reload
# 6. Open main config file of nginx : $ sudo vim etc/nginx/nginx.config
### make sure it have this fields:
    user nginx;
    worker_processes 1;   #put here the number of your CPU cores on your machine
    
    ...

    events {
        worker_connections 1024;   #depends on your system, here you write number of maximum simultaneously connections
    }
    
    http {
        
        ...
        
        include /etc/nginx/conf.d/*.conf;
        include /etc/nginx/sites-enabled/*;  #make sure this path is here, because it is the main path to our next configuration file, in which we will write adresses of our actually servers
        }
    }
# 7. create new folder in nginx( if it does not exist yet), in which will be our default config file (it can be there already):
    $ mkdir /etc/nginx/sites-enable
    $ vi /etc/nginx/sites-enable/default
### This default config file should be configured like this:
    upstream project {       # project - is name of group of servers that need to manage 
        hash $remote_addr;   # method of balancing using IP address as a key to balance users
        server 192.168.103.166:8080;    #next 3 fields describes to which servers will our nginx serve as proxy
        server 192.168.103.247:8080;
        server 192.168.103.221:8080;
    }

    server {
        listen 80;         # listening to this port, if something come it will redirect to "location" field
            location / {
            proxy_pass http://project;   #this line tells us, that every request that come on our address on 80 port will redirect to group of my servers
        }                                #make sure that proxy_pass field have exactly same name as the name of upstream
    }
# 8. Reload and restart nginx
    $ systemctl reload nginx
    $ systemctl restart nginx
# 9. Your nginx load ballancer is ready to use! To access OMS file just call your local IP address + /OMS
    example:
    192.168.103.172/OMS
# 10. But if you dont want to write "/OMS" after IP addres just do next steps on your server (where tomcat is):
    a) open this file : $ vi /var/lib/tomcat/webapps/ROOT/index.jsp
    b) in oppened file on the top write next line:
       <% response.sendRedirect("http://IPaddressOfYourBallancer/OMS"); %>
### example <% response.sendRedirect("http://192.168.103.172/OMS"); %>
    c)$ systemctl restart tomcat

# 11. Also if you want that your app servers be only reachable from ballancer, and someone can`t use their IP addresses instead of IP address of balancer:
    # [port] and [ip] need to write without "[" and "]"
    $ iptables -A INPUT -p tcp -s [ip] --dport [port] -j ACCEPT
    $ iptables -A INPUT -p tcp -s 0.0.0.0/0 --dport [port] -j DROP
    $ iptables-save
    # here [ip] - is IP address of your balancer, throw it will be all trafic passed
    # [port] - port that you will open for one [ip] and close to all other internet IP addresses
# 12. DONE! Enjoy your work, drink some coffe and be cool! =)
