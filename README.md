# Enabling SSL & Reverse Proxying
Enabling SSL & reverse proxying with Apache in Ubuntu 
Server Firewall setup
	firewall-cmd --permanent --zone=public --add-port=25565/tcp


Reverse proxy (assuming you have Apache,certbot,mariaDB,php)

->Request Certbot certificate
	certbot certonly --apache -d DOMAIN
 
->if error ( unable to find extension like certonly --apache)
	sudo apt install -y certbot python3-certbot-apache
 
->if error ( Unable to find a virtual host listening on port 80)
	cd /etc/apache2/sites-available
 
	vim yourDomainName.conf
	
	<VirtualHost *:80>
   		 ServerName yourDomainName.com
   		 DocumentRoot /var/www/html
   		 ServerAlias www.yourDomainName.com
   		 ErrorLog /var/www/error.log
   		 CustomLog /var/www/requests.log combined
	</VirtualHost>

	then restart the apache2 service (service apache2 restart)
		a2ensite yourDomainName
		service apache2 restart
		ctrl-d to exit root

->now create a apache conf
	nano /etc/apache2/sites-enabled/0-pp.conf

	<VirtualHost *:443>
    		ServerName DOMAIN

    		ProxyPreserveHost On
    		ProxyPass / http://127.0.0.1:8080/
    		ProxyPassReverse / http://127.0.0.1:8080/

    		RewriteEngine on
    		RewriteCond %{HTTP:Upgrade} websocket [NC]
    		RewriteCond %{HTTP:Connection} upgrade [NC]
    		RewriteRule .* ws://127.0.0.1:8080%{REQUEST_URI} [P]

    		ErrorLog ${APACHE_LOG_DIR}/pp.error.log

    		SSLCertificateFile /etc/letsencrypt/live/DOMAIN/fullchain.pem
    		SSLCertificateKeyFile /etc/letsencrypt/live/DOMAIN/privkey.pem
	</VirtualHost>
->Yes you are done
