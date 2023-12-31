# Enabling SSL & Reverse Proxying
Enabling SSL & reverse proxying with Apache in Ubuntu 

Extra Information Before you proceed : 

-> Server Firewall setup
```
	firewall-cmd --permanent --zone=public --add-port=25565/tcp
```
-> Replace every 'DOMAIN' with your own domain

->This is made for Ubuntu Environment

Reverse proxy (assuming you have Apache,certbot,mariaDB,php)

->Request Certbot certificate
```
	certbot certonly --apache -d DOMAIN
```
 
->if error ( unable to find extension like certonly --apache)
```
	sudo apt install -y certbot python3-certbot-apache
```
 
->if error ( Unable to find a virtual host listening on port 80 OR an error like this site doesn't exist)
```
	cd /etc/apache2/sites-available
```
```
	vim yourDomainName.conf
```
```	
	<VirtualHost *:80>
   		 ServerName yourDomainName.com
   		 DocumentRoot /var/www/html
   		 ServerAlias www.yourDomainName.com
   		 ErrorLog /var/www/error.log
   		 CustomLog /var/www/requests.log combined
	</VirtualHost>
```
->then restart the apache2 service (service apache2 restart)
```
		a2ensite yourDomainName
```
```
		service apache2 restart
```
```
		ctrl-d to exit root
```

->now create a apache conf (keep in mind to Change the PORT to the port where you want to pass, & change DOMAIN in the certificate directory)
```
	nano /etc/apache2/sites-enabled/0-pp.conf
```
```
	<VirtualHost *:443>
    		ServerName DOMAIN

    		ProxyPreserveHost On
    		ProxyPass / http://127.0.0.1:PORT/
    		ProxyPassReverse / http://127.0.0.1:PORT/

    		RewriteEngine on
    		RewriteCond %{HTTP:Upgrade} websocket [NC]
    		RewriteCond %{HTTP:Connection} upgrade [NC]
    		RewriteRule .* ws://127.0.0.1:PORT%{REQUEST_URI} [P]

    		ErrorLog ${APACHE_LOG_DIR}/pp.error.log

    		SSLCertificateFile /etc/letsencrypt/live/DOMAIN/fullchain.pem
    		SSLCertificateKeyFile /etc/letsencrypt/live/DOMAIN/privkey.pem
	</VirtualHost>
```
->you are done :)
