## Walkthrough - Setting up Wordpress site with DBaaS

These instructions are a rough outline for setting up a Wordpress instance using Linode's DBaaS (still in beta)

⁃ Provision an Ubuntu 18.04 LTS Linode (Shared 1GB)
- Log in as root user
- Create sudo user

`adduser $USER`

`adduser $USER sudo`

- Create a 3 node DB cluster using mysql v8.0.26 - Shared 1GBs
- Add Access control for previously provisioned Ubuntu Linode: $IPAddress/32 
- Switch to sudo user

`su - $USER`

- Instal nginx and php using https://www.linode.com/docs/guides/how-to-install-the-lemp-stack-on-ubuntu-18-04/ 
- Run:

`sudo apt update && sudo apt upgrade`

- When given the option, choose to install the package maintainer’s version

`sudo apt install nginx`

`sudo apt install php-fpm php-mysql`

`sudo sed -i 's/;cgi.fix_pathinfo=1/cgi.fix_pathinfo=0/g' /etc/php/7.2/fpm/php.ini`

- Set up A and AAAA records pointing your Ubuntu server to your domain

`sudo mkdir -p /var/www/html/yourdomain.com/public_html`

`sudo cp /etc/nginx/sites-enabled/default /etc/nginx/sites-available/yourdomain.com.conf`

- Edit your site's conf file:

`sudo vim /etc/nginx/sites-available/yourdomain.com.conf`

- Create symbolic link to sites-enabled

`sudo ln -s /etc/nginx/sites-available/yourdomain.com.conf /etc/nginx/sites-enabled/`

- Restart php and reload nginx

`sudo systemctl restart php7.2-fpm`

`sudo nginx -s reload`

- Test nginx config

`sudo nginx -t`

- Instal mysql-client so you can interact with DB cluster
- Navigate here in browser: https://dev.mysql.com/downloads/file/?id=509020
- Right click "No thanks, just start my download.", to copy link address
- On Ubuntu server:

`cd /tmp`

`curl -OL https://dev.mysql.com/get/mysql-apt-config_0.8.22-1_all.deb` <-- Link you just copied above

- Add the MySQL APT repository to your system’s repository list

`sudo dpkg -i mysql-apt-config_0.8.22-1_all.deb`

- Hit "OK"

- Refresh your apt package cache to make the new software packages available

`sudo apt update`

- Clean up your system a bit and delete the file you downloaded, as you won’t need it in the future

`rm mysql-apt-config_0.8.22-1_all.deb`

- Now that you’ve added the MySQL repositories, you’re ready to install the actual MySQL client software. Do so with the following apt command

`sudo apt install mysql-client`

- Confirm mysql version

`mysql --version`

`mysql  Ver 8.0.28 for Linux on x86_64 (MySQL Community Server - GPL)`

- By this point, your DB cluster should be provisioned and you can log in with: “mysql -u linroot -h $HOSTNAME -p” and the provided PW in Cloud Manager
- From Ubuntu server, create a database:

`CREATE DATABASE wordpress DEFAULT CHARACTER SET utf8 COLLATE utf8_unicode_ci;`

`CREATE USER '$USERNAME'@'$IPADDRESS' IDENTIFIED WITH mysql_native_password BY 'C0mPl3xP@s$w0rD';`

`GRANT ALL ON wordpress.* TO '$USERNAME'@'$IPADDRESS';`

`exit;`

- Download some additional PHP extensions for use with WordPress:

`sudo apt install php-curl php-gd php-mbstring php-xml php-xmlrpc php-soap php-intl php-zip`

- Restart nginx:

`sudo systemctl restart nginx.service`

- Downloading Wordpress

`cd /tmp`

`curl -O https://wordpress.org/latest.tar.gz`

`tar xzvf latest.tar.gz`

- Also, copy over the sample configuration file to the filename that WordPress actually reads:

`cp /tmp/wordpress/wp-config-sample.php /tmp/wordpress/wp-config.php`

- (Not necessary) Created an upgrade directory, so that WordPress won’t run into permissions issues when trying to do this on its own following an update to its software:

`mkdir /tmp/wordpress/wp-content/upgrade`

- Copied the entire contents of the directory into your document root.

`sudo cp -a /tmp/wordpress/. /var/www/html/yourdomain.com/public_html`

- Update permissions

`sudo chown -R www-data:www-data /var/www/html/yourdomain.com/public_html`

- Run the following two find commands to set the correct permissions on the WordPress directories and files:

`sudo find /var/www/html/yourdomain.com/public_html -type d -exec chmod 750 {} \;`

`sudo find /var/www/html/yourdomain.com/public_html -type f -exec chmod 640 {} \;`

- Make the following changes to the main WordPress configuration file.
- Replace some secret keys to provide security.
- WordPress has a secure generator for generating these values. To get these values from the WordPress secret key generator, run the following command:

`curl -s https://api.wordpress.org/secret-key/1.1/salt/`

Example output:

```
define('AUTH_KEY',         '5.]sl&qC3zmnQSu|gzKD+DtUI->iY=U-YD?ty1>5#bcH/zCY|{8H[:r/QIo]9+-c');
define('SECURE_AUTH_KEY',  '7n29|q>|wky2U?oIN~k3jZWvQIhAP1-2Z!Kw$7:=z(go!QR:*MTh=KXS@7!,BQtM');
define('LOGGED_IN_KEY',    'E*VeOxQ2^lBc,q#-9|-#gpi-nNO3]EwT[sN-&!k@dT-JsFaoR<EW!H`;Wl/ _V]d');
define('NONCE_KEY',        '=D0s#9ArO2I/f-MDru{p<CI5{Pvk>o?v$mr1= &x@ w{_XUvV{-,f-v|qH-D?q1t');
define('AUTH_SALT',        '<>{%jO*?u|o4fV+!M&iRm~6GDW+#5Mh .j}} C#8/A928LsTg+GJ}f(9`v:c?q]g');
define('SECURE_AUTH_SALT', 'zSMqHKcI-tDIl6txrze^1%$TXBpPQPqs3X]SE?PlAkV_a0zg3o+wzrdg+F^])unN');
define('LOGGED_IN_SALT',   ']!ntDVy]DuN()aVux JQF5X/|;]{*C0|Iz4g=8-G1}JsiKPqB*#w3Vf!p4-Pie.B');
define('NONCE_SALT',       'nhxgXL99L0@9ftzSP=1LEJc}+[BBc! aA M +!RWBN+ByV5*B!?I;:K+;-Oui-~`');
```

- Copy `wp-config-sample.php` to `wp-config.php`

`sudo cp /var/www/html/yourdomain.com/public_html/wp-config-sample.php /var/www/html/yourdomain.com/public_html/wp-config.php`

- Edit file to add secrets

`sudo vim /var/www/html/yourdomain.com/public_html/wp-config.php`

- Update DB_NAME, DB_USER, DB_PASSWORD, and DB_HOST in the `wp-config.php` file, too.

- Ad this line as well, which sets the method that WordPress will use to write to the filesystem:

`define('FS_METHOD', 'direct');`

### Configuring WordPress to Communicate with MySQL Over TLS/SSL
- Download your DBaaS CA certificate from Cloud Manager
- Uploaded to your Wordpress/Ubuntu server:

`scp /path/to/file/ca-certificate.crt $SUDO_USER@$IPADDRESS:/tmp`

- Once the CA certificate is on your server, move it to the /user/local/share/ca-certificates/ directory, Ubuntu’s trusted certificate store:

`sudo mv /tmp/wordpress-3rd-ca-certificate.crt /usr/local/share/ca-certificates/`

- Following this, run the `update-ca-certificates` command. This program looks for certificates within /usr/local/share/ca-certificates, adds any new ones to the /etc/ssl/certs/ directory, and generates a list of trusted SSL certificates based on its contents:

`sudo update-ca-certificates`

- reopen `wp-config.php` file:

`sudo vim /var/www/html/yourdomain.com/public_html/wp-config.php`

- Add this line:

`define('MYSQL_CLIENT_FLAGS', MYSQLI_CLIENT_SSL);`

- This allows WordPress to securely communicate with our managed MySQL database.
- At this point, you should be able to hit your domain (or IP) in your browser and go through the Wordpress set up.