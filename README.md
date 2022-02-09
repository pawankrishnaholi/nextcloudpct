# Nextcloud on Debian LXC
Hello Everyone,  
I have put together a sequence of steps to install Nextcloud server on Debian Linux Container. I use Proxmox as my bare metal hypervisor so you can ignore some of the steps below if you are using other type 1 hyperv.  

Please feel free to share your feedback and corrections(if any).

<br>

**Proxmox Debian LXC + Nextcloud Setup Steps**:
1. Download the latest Debian conatiner template from Proxmox template list.
2. Container Configuration:
    <details>
    <summary>Debian Container Creation in Proxmox 7.x(Screenshots)</summary>

    **General Tab**
    > I have provided the hostname, and password to login once the container spins up.  
    ![](./resources/images/proxmoxpct/GeneralTab.jpg?raw=true "General Tab")  

    **Template Tab**
    > Here I am going to choose Debian 11 which I downloaded from the previous step.  
    ![](./resources/images/proxmoxpct/TemplateTab.jpg?raw=true "Template Tab")  

    **Disks Tab**
    > Here I have allocated only 4 GB of storage to install Nextcloud and other supporting files.  
    ![](./resources/images/proxmoxpct/Disks1Tab.jpg?raw=true "Disks Tab")  

    > (Optional) I have added a new ZFS dataset as a mountpoint which I will use as my data directory to store all my nextcloud files. If you prefer to store the data and install on one disk then size it accordingly.  
    ![](./resources/images/proxmoxpct/Disks2Tab.jpg?raw=true "Disks Tab")  

    **CPU Tab**
    > Here I chose only 1 CPU core.  
    ![](./resources/images/proxmoxpct/CPUTab.jpg?raw=true "CPU Tab")  

    **Memory Tab**
    > Here I allocated 512 MB RAM and 512 MB swap memory.  
    ![](./resources/images/proxmoxpct/MemoryTab.jpg?raw=true "Memory Tab")  

    **Nextwork Tab**
    > I recommend using a static IP address.  
    ![](./resources/images/proxmoxpct/NetworkTab.jpg?raw=true "Nextwork Tab")  

    **DNS Tab**
    > I didn't set any DNS settings, I usually control this on my router, unless you want to overwrite it, leave it blank, let the router do it's thing.  
    ![](./resources/images/proxmoxpct/DNSTab.jpg?raw=true "DNS Tab")  

    </details>
3. Start and login to the container, run below commands to download and install any avaialle updates/patches.  
    ```
    ➡ apt-get update
    ➡ apt-get -y upgrade
    ➡ apt-get -y autoremove
    ➡ apt-get -y autoclean
    ```
4. Install LAMP (Linux, Apache, MySQL, PHP) stack + other necessary addons, run below commands  
    ```
    ➡ apt-get -y install mariadb-server
    ➡ apt-get -y install php
    ➡ apt-get -y install php-{cli,xml,zip,curl,gd,cgi,mysql,mbstring,intl,bcmath,gmp,imagick,apcu}
    ➡ apt-get -y install apache2
    ➡ apt-get -y install libapache2-mod-php
    ➡ apt-get -y install curl 
    ➡ apt-get -y install unzip
    ➡ apt-get -y install imagemagick
    ➡ apt-get -y install software-properties-common
    ➡ apt-get -y install python3-certbot-apache
    ```
5. To improve the security of MariaDB, run the below command and follow the prompts(Y/n).  
    ```
     ➡ mysql_secure_installation
    ```
6. Login into MySQL and create a new user, database and grant privileges. run following commands  
    ```
    ➡ mysql -u root -p
    ```

    ```
    ➡ CREATE USER 'nextcloud'@'localhost' IDENTIFIED BY 'StrongDBP@SSwo$d';
    ➡ CREATE DATABASE nextcloud;
    ➡ GRANT ALL PRIVILEGES ON nextcloud.* TO 'nextcloud'@'localhost';
    ➡ FLUSH PRIVILEGES;
    ➡ \q
    ```
7. Edit the php.ini file and set/modify the configuration parameters  
    ```
    ➡ nano /etc/php/*/apache2/php.ini
    ```
    > Feel free to modify it to your needs. Example: set max file size to 10G, temp directory location to something else...  
    
    * date.timezone = America/Chicago  
    * memory_limit = 512M  
    * upload_tmp_dir = /mnt/ncdata/nextcloud/tmp  
    * upload_max_filesize = 16G  
    * post_max_size = 16G  
    * max_execution_time = 3600  
    * max_input_time = 3600  
    * output_buffering = Off  
    * open_basedir = /var/www/html/nextcloud:/dev/urandom:/mnt/clddata:/tmp/:/proc/meminfo  

8. Download Nextcloud and set appropriate permissions  
    > I have mounted a second disk at "/mnt/ncdata", if you are not using a different disk to store Nextcloud data then feel free to change the path it to whatever you have decided.  
    ```
    ➡ curl -o /root/nextcloud-23.zip https://download.nextcloud.com/server/releases/latest-23.zip
    ➡ unzip nextcloud-23.zip
    ➡ mv nextcloud /var/www/html/
    ➡ chown -R www-data:www-data /var/www/html/nextcloud
    ➡ chmod -R 755 /var/www/html/nextcloud
    ➡ rm -rf /mnt/ncdata/lost+found
    ➡ mkdir /mnt/ncdata/nextcloud
    ➡ mkdir /mnt/ncdata/nextcloud/data
    ➡ mkdir /mnt/ncdata/nextcloud/tmp
    ➡ chown -R www-data:www-data /mnt/ncdata/nextcloud
    ➡ chmod -R 770 /mnt/ncdata/nextcloud
    ➡ systemctl restart apache2
    ```
9. Login into http://192.168.1.10/nextcloud/ to continue the setup  
    > Create Nextcloud admin account  

    Username: admin  
    Password: password  

    > Enter/modify below configuration information  

    Data folder: /mnt/data/nextcloud/data  
    Database user: nextcloud  
    Database password: StrongDBP@SSwo$d  
    Database name: nextcloud  
    Host: localhost  

    > (Optional) Check/uncheck "Install recommended apps". Check this box if you want to use apps like calendar, contacts, video calls. If you uncheck it, you will be left with basic apps like Files, Photos, and Activity.  
    
    Check/uncheck "Install recommended apps"  
    
    <details>
    <summary> Nextcloud setup page(Screenshots)</summary>

    **Admin Account Creation**
    > I have provided the hostname, and password to login once the container spins up.  
    ![](./resources/images/nextcloudsetup/nextcloudadminaccconfig.jpg?raw=true "Admin Account Creation Section")  

    **Database Config**
    > Here I am going to choose Debian 11 which I downloaded from the previous step.  
    ![](./resources/images/nextcloudsetup/nextclouddbconfig.jpg?raw=true "Database Configuration Section")  

    </details>
<br>
<br>
<br>
If you want to access the files over internet then you need a domain + SSL certificate to secure your communication between the client and the Nextcloud server running in your home/office. We will use the popular free service to get our certs "Lets Encrypt", typically the free cert expires every 90 days, so we have to either manually renew the certs or automate it.  

**SSL Setup Steps**:   
1. **Domain**: I recommend getting a cheap domain from "Google domains", or other domain registrars like "GoDaddy" or "Namecheap".  
    *OR*  
    You can register for a free subdomain from DDNS services like freedns.afraid.org or no-ip.com. If you have an Asus Router then you can get free DDNS service from Asus **xxxxxxx.asuscomm.com**.  
2. **Port Forwarding**: You have to open ports on your router in order to complete the SSL process. Forward ports 80 and 443 to the local IP running the nextcloud sever.  

    | EXT_PORT_NO | LAN_PORT_NO | LAN_IP       | PROTOCOL |
    |-------------|-------------|--------------|----------|
    | 443         | 443         | 192.168.1.10 | TCP      |
    | 80          | 80          | 192.168.1.10 | TCP      |

3. Make a backup of the apache config files that we will be editing, run the below commands
    ```
    ➡ cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/000-default.conf.bkp
    ➡ cp /etc/apache2/sites-available/default-ssl.conf /etc/apache2/sites-available/default-ssl.conf.bkp
    ```
4. Run the certbot to generete SSL certs, run the below command (This command will automatically udpdate all necessary config files like adding auto http -> https redirect, servername rewrite rules. I feel this is the fastest way to get things up and running with https) and follow the prompts to complete the process.
    ```
    ➡ certbot --apache
    ```
5. To strengthen the security, lets create a new ssl parameters config file and add the content mentioned below, run the below command
    ```
    ➡ nano /etc/apache2/conf-available/ssl-params.conf
    ```
    > Paste following content into the file

    SSLCipherSuite EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH  
    SSLProtocol All -SSLv2 -SSLv3  
    SSLHonorCipherOrder On  
    Header always set Strict-Transport-Security "max-age=63072000; includeSubdomains; preload"  
    Header always set Strict-Transport-Security "max-age=63072000; includeSubdomains"  
    SSLCompression off  
    SSLSessionTickets Off  
    SSLUseStapling on  
    SSLStaplingCache "shmcb:logs/stapling-cache(150000)"  
6. Modify apache config files, run the follwing command
    ```
    ➡ nano /etc/apache2/sites-available/000-default.conf
    ```
    > Change the DocumentRoot path

    DocumentRoot /var/www/html/nextcloud  
    
    > At the end of the file paste below content

    Redirect 301 /.well-known/webfinger /index.php/.well-known/webfinger    
    Redirect 301 /.well-known/nodeinfo /index.php/.well-known/nodeinfo  
    Redirect 301 /.well-known/caldav /remote.php/dav  
    Redirect 301 /.well-known/carddav /remote.php/dav  

    ```
    ➡ nano /etc/apache2/sites-available/default-ssl.conf
    ```
    
    > Change the DocumentRoot path and correct the ssl certs path(replace \<your domain> with the domain you used to create ssl certs in step 4)

    DocumentRoot /var/www/html/nextcloud  
    
    SSLCertificateFile /etc/letsencrypt/live/\<your domain>/fullchain.pem  
    SSLCertificateKeyFile /etc/letsencrypt/live/\<your domain>/privkey.pem

    > Uncomment below lines

    BrowserMatch "MSIE [2-6]" \
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;nokeepalive ssl-unclean-shutdown \  
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;downgrade-1.0 force-response-1.0

    > At the end of the file paste below content

    Redirect 301 /.well-known/webfinger /index.php/.well-known/webfinger  
    Redirect 301 /.well-known/nodeinfo /index.php/.well-known/nodeinfo  
    Redirect 301 /.well-known/caldav /remote.php/dav  
    Redirect 301 /.well-known/carddav /remote.php/dav  

    ```
    ➡ nano /etc/apache2/sites-available/000-default-le-ssl.conf
    ```

    > Change the DocumentRoot path

    DocumentRoot /var/www/html/nextcloud   

    ```
    ➡ nano /var/www/html/nextcloud/config/config.php  
    ```

    > Add/Modify the file with below contents

    * 'trusted_domains' =>   
    array (  
        0 => '192.168.1.10',  
        1 => '\<your domain>'  
    ),  
 
    * 'overwrite.cli.url' => 'https://192.168.1.10/nextcloud',  
 
    * 'default_language' => 'en',  
    * 'force_language' => 'en',  
    * 'default_phone_region' => 'US',  
    * 'force_locale' => 'en_US',  
    * 'remember_login_cookie_lifetime' => 3600,  
    * 'session_lifetime' => 300,  
    * 'auto_logout' => true,  
    * 'auth.bruteforce.protection.enabled' => true,  
    * 'memcache.local' => '\OC\Memcache\APCu',  
    * 'overwriteprotocol' => 'https',

7. To enable custom SSL params we created in step 5, and let apache use the modified config files, run the following commands.
    ```
    ➡ a2enmod ssl
    ➡ a2enmod headers
    ➡ a2enmod emv
    ➡ a2ensite default-ssl
    ➡ a2enconf ssl-params
    ➡ apache2ctl configtest
    ➡ systemctl restart apache2
    ```
8. Automate certs renewal:  Certbot comes with auto renew script placed under "/etc/cron.d/certbot". This script renews the certs that will expire in less than 30 days. You can verify whether the script is present under the mentioned path by running the follwing commands
    ```
    ➡ cat /etc/cron.d/certbot
    ```
    > Verify the last few lines of the file. Renew command is run as a "root" user every day at 12th hour.  Example: 1/1/2022 00:00, 1/1/2022 12:00, 1/2/2022 00:00, 1/2/2022 12:00...

    SHELL=/bin/sh  
    PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin  
    0 */12 * * * root test -x /usr/bin/certbot -a \! -d /run/systemd/system && perl -e 'sleep int(rand(43200))' && certbot -q renew  
9. Access your nextcloud using the domain you used to during creating of ssl certs. Test the https redirect by accessing the site with **http://**\<your domain>, you should see the redirect to **https://**\<your domain> verify the the validity of the SSL cert by clicking the padlock 🔒 next to **https://**



## Enjoy!
