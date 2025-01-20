# formazione_sou
**VIRTUAL BOX**

*Warning! All the indication along this document refers to the Expert mode menu*

- Download VirtualBox 7.1.4 platform packages for [macOS / Intel hosts](https://download.virtualbox.org/virtualbox/7.1.4/VirtualBox-7.1.4-165100-OSX.dmg) 
- Follow the .dmg installation instructions
  - Predefined Folder /Users/massimo/VirtualBox VMs
- Install the command line developer’s tools for Pyton3 support (requires a long installation time)
- Launch the VIRTUAL BOX from the MAC folder Applications
- From File -> Tools -> Network manager, set the subnet 192.168.56.0/24 in NAT Networks
  - Name = NatNetwork (predefined)
  - IPv4 Prefix = 192.168.56.0/24
  - Don’t use DHCP

**ROCKY LINUX 9.5**

- Download Rocky Linux 9 Intel x86\_64 platform – DVD (Minimal ISO doesn’t contain some packets) from <https://rockylinux.org/download>

**VIRTUAL BOX CONFIGURATION**

In the Create Virtual Machine (New icon) set:

- VM NAME = RockyLinux node1
- SO TYPE = Linux
- SO VERSION = Red Hat (64-bit)
- Create a virtual Hard Disk now
- RAM = 2 GB (2048 MB)
- CPU = 2
- Skip Unattended Installation
- Click on Create button

In the new Window, set:

- VIRTUAL DISK = 20GB
- VHD (Virtual Hard Disk)
- Click on Create button

In the Settings menu (Settings icon):

- Storage => Controller IDE => Empty - set to the ISO Linux
- From the main menu bar, open Settings menu then System and select USB Tablet as a pointing device
- From the VM node settings, open Network Settings and set:
  - Adapter 1
  - Attached to: NAT Network
  - Name: NATNetwork

- Set Skip Unattended installation
- Check the correct configuration and then press Start to create the VM

**ROCKY LINUX 9.5 Installation**

- Select Install Rocky Linux 9.5 and press enter
- A Terminal window will open and will show all the installation step
- At the end of the Installation, Rocky Linux welcomes and will ask to set the Language (English)
- Rocky Linux will show the Installation Summary. Set:
  - Under System the Installation Destination – Confirm the previously defined Virtual Disk (ATA VBOX HARD DISK), then click on Done
  - Under User settings define the Root user password, then click on Done

    Password \*\*\*\*\*\*\*

  - Under User settings create a new user, then click on Done

    Full name Massimo1

    User name MAX1

    Password \*\*\*\*\*\*\*

- Click on Begin Installation button on bottom right corner
- It will be shown the Installation progress until the completion
- Click on Reboot button on bottom right corner.

- After launching the VM, wait for the complete booting.
- Provide login
- Select Wired Connected in the upper left corner (any icon), then Wired settings
- Press the Cog icon and select the IPv4 tab and Set:
  - IPv4 Method = Manual
  - Address 192.168.56.10
  - Netmask 255.255.255.0
  - Gateway 192.168.56.1
  - DNS 8.8.8.8,8.8.4.4
- Reboot
- In Activities open a browser and check the correct internet browsing.
- Shut down


- In Virtual Box select the first created VM and create 2 additional VM as clone of the first VM
- Change the MAC Address on the cloned VM’s by accessing the VM node settings, Open Network Settings:
  - Adapter 1
  - Attached to: NAT Network
  - Name: NATNetwork
  - MAC Address -> Regenerate

- Launch node 2
- After login and booting completion 
- Press the Cog icon and select the IPv4 tab and Set:
  - IPv4 Method = Manual
  - Address 192.168.56.11
  - Netmask 255.255.255.0
  - Gateway 192.168.56.1
  - DNS 8.8.8.8,8.8.4.4
- Reboot
- In Activities open a browser and check the correct internet browsing.
- Shut down
- Launch node 3
- After login and booting completion 
- Press the Cog icon and select the IPv4 tab and Set:
  - IPv4 Method = Manual
  - Address 192.168.56.12
  - Netmask 255.255.255.0
  - Gateway 192.168.56.1
  - DNS 8.8.8.8,8.8.4.4
- Reboot
- In Activities open a browser and check the correct internet browsing.
- Shut down

**Installing Maria DB on ROCKY LINUX 9.5 – node 2**

Ref: https://docs.vultr.com/how-to-install-mariadb-on-rocky-linux-9

- Type: su root
- Type: sudo dnf list mariadb. You should obtain mariadb.x86\_64.
- Install MariaDB: sudo dnf install mariadb mariadb-server -y
- Check MariaDB mariadb –-sudoversion. You should obtain

  mariadb Ver 15.1 *Distrib 1,5,22-MariaDB, for Linux (x86\_64 using EditLine wrapper*

**Configuring Maria DB System Service - node 2cd**

- Set MariaDB service to start at boot sudo systemctl enable mariadb
- Start the MariaDB service sudo systemctl start mariadb
- Stop the MariaDB service sudo systemctl stop mariadb
- Restart the MariaDB service sudo systemctl restart mariadb
- Check the MariaDB status and verify that is running sudo systemctl status mariadb

**Access Maria DB - node 2**

- Log in to the MariaDB console as the root database user console 

  mariadb -u root -p You should access the MariaDB console

  Prompt: MariaDB [(none)]>

- Create a new database user(maxdbuser).

  CREATE USER 'maxdbuser'@'localhost' IDENTIFIED BY 'STRONG-PASSWORD';

  STRONG-PASSWORD is the password

- Create a new sample database (maxdb)

  CREATE DATABASE `maxdb`;

- Check current Maria DB databases 

  SHOW DATABASES;

- Grant the user full privileges to the maxdb database and the remote user

  GRANT ALL PRIVILEGES ON maxdb.\* TO 'maxdbuser'@'localhost';

  GRANT ALL PRIVILEGES ON maxdb.\* TO 'maxdbuser'@'192.168.56.10';

- Refresh the MariaDB privilege tables to apply the new user changes.

  FLUSH PRIVILEGES;

- Exit the MariaDB console.

  EXIT

- Log in to the MariaDB console using the new database user and password.

  mariadb -u maxdbuser -p

  STRONG-PASSWORD is the password

- Open the 3306 port on DB node 2 (192.168.56.11):

  firewall-cmd --add-port=3306/tcp –-permanent

  firewall-cmd –-reload

- Verify the 3306 port is open

  firewall-cmd –-list-all

  **APACHE Installation – node 1**

- Connect as Superuser

  su root

- Install Apache providing the command:

  yum install httpd

- Start Apache

  systemctl start httpd

- Enable on boot
- systemctl enable httpd
- Verify the status

  systemctl stop httpd

  **PHP Installation – node 1**

- Be sure to act as Superuser

  su

- Install PHP and required extensions providing the command:
- yum install php php-mysqlnd php-gd php-xml php-mbstring
- Restart Apache

  systemctl restart httpd

- Verify the status

  systemctl stop httpd

  **Wordpress Installation – node 1**

- Be sure to act as Superuser

  su

- Install wget providing the command (if necessary) :

  yum install wget

- Download the latest version of WordPress through the command:
- wget https://wordpress.org/latest.tar.gz
- Extract the downloaded files:

  tar -xzvf latest.tar.gz

- Move WordPress files to the Apache web server's root directory, in order to allow the webserver to serve the WordPress site correctly when users access it through a web browser:

  cp -r wordpress/\* /var/www/html/

- Set the specific permissions for WordPress files (group, owner):

  chown -R apache:apache /var/www/html/
  chmod -R 755 /var/www/html/

- Verify the **"**html**"** directory permissions through the "[**ls**](https://www.geeksforgeeks.org/ls-command-in-linux/)" and "[**grep**](https://www.geeksforgeeks.org/grep-command-in-unixlinux/)" commands:
  ls -l /var/www | grep html

  **Wordpress configuration – node 1**

  References: 

  <https://www.geeksforgeeks.org/how-to-install-wordpress-on-rocky-linux-9/#step-2-installation-process-of-lamp>

  https://www.evemilano.com/blog/collegare-wordpress-nuovo-database/

- Be sure to act as Superuser
- Rename the original WordPress configuration file:

  cd /var/www/html/
  mv wp-config-sample.php wp-config.php

- Verify through the "ls" and **"**grep**"** commands:

  ls -l | grep wp-config.php

- Open the "wp-config.php" configuration file:

  nano wp-config.php

- Modify the previous database settings with:

  define ('DB\_NAME', 'maxdb');
  define ('DB\_USER', 'maxdbuser');
  define ('DB\_PASSWORD', 'STRONG-PASSWORD');

  define ('DB\_HOST', '192.168.56.11');

- Open the firewall to allow traffic "http", and "https":

  firewall-cmd --permanent --add-service=http 
  firewall-cmd --permanent --add-service=https
  firewall-cmd –-reload

  (alternatively

  firewall-cmd --add-port=80/tcp --permanent

  firewall-cmd --add-port=443/tcp -–permanent)

- Verify through the grep command:

  firewall-cmd --list-all | grep http

- Verify the remote access to the MariaDB DB on node 2

  mariadb -u maxdbuser -h 192.168.56.11 -p

- Check the Apache networking option ON:

`	`using this command will show that this option is available

`	`/usr/sbin/getsebool -a | grep httpd

- Set the Apache network options through the command, making permanent:

  setsebool -P httpd\_can\_network\_connect=1

  setsebool -P httpd\_can\_network\_connect\_db=1


- Port forwarding on Virtual box, configure Virtual box Network:

  Name: Rule 1

  Protocol: TCP

  Host IP: -

  Host Port:80

  Guest IP: 192.168.56.10

  Guest Port: 80

  **HAPROXY installation – node 3**

  References: 

  https://shape.host/resources/how-to-install-and-use-haproxy-on-rocky-linux-8

  https://www.haproxy.com/blog/haproxy-configuration-basics-load-balance-your-servers

- Install the **haproxy** package:

`	`sudo yum install -y haproxy

- Enable and start the **haproxy** service:

`	`sudo systemctl enable haproxy

`	`sudo systemctl start haproxy

- Set on Wordpress

  WordPress Address (URL) = http://mywpsite.local

  Site Address (URL) = http://mywpsite.local

  Add to MAC /etc/hosts:

  127\.0.0.1	mywpsite.local

- Configure HAPROXY

`	`set in /etc/haproxy/haproxy.cfg

`	`frontend main

`	`bind \*:80

`	`default\_backend myserver

backend myserver

`	`wpserver 192.168.56.10:80

comment line:

\#	balance	roundrobin

**Self Signed Certificate generation and installation – node 3**

References:

https://sharmank.medium.com/self-signed-certificate-and-use-them-in-haproxy-and-allow-certificate-in-macos-26c3aad316bb

https://webhostinggeeks.com/howto/how-to-configure-ssl-certificate-in-haproxy/

https://www.haproxy.com/documentation/haproxy-configuration-tutorials/authentication/client-certificate-authentication/

https://www.haproxy.com/documentation/haproxy-configuration-tutorials/ssl-tls/client-side-encryption/

https://betterstack.com/community/questions/curl-ssl-certificate-problem/

- Generate a CA key to sign certificate:

`	`su

`	`openssl genrsa -des3 -out rootCA.key 2048

`	`(provided 12345 as a pass phrase for rootCA.key)

- Generate public certificate, duration set to 10 years (3650).

`	`openssl req -x509 -new -nodes -key rootCA.key -sha256 -days 3650 -	out rootCA.pem

`	`At this point provide the different required information:

`	`Country name: IT

`	`Organisation name: Sourcesense

`	`Common Name: mywpsite.local

`		`At the end two files are generated, `rootCA.key` `rootCA.pem`

- Create file `server.csr.cnf` (/home/Massimo1) as follows:

  [req]
  default\_bits = 2048
  prompt = no
  default\_md = sha256
  distinguished\_name = dn

  [dn]
  C=IT
  ST=Torino
  L=Torino
  O=Sourcesense
  OU=Random
  emailAddress=<massimo,veneziano@sourcesense.com>
  CN=<mywpsite.local>

- Create a v3.ext file with the following contents:

  authorityKeyIdentifier=keyid,issuer
  basicConstraints=CA:FALSE
  keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
  subjectAltName = [@alt_names](http://twitter.com/alt_names)

  [alt\_names]
  DNS.1=<mywpsite.local>

- Generate private key

  openssl req -new -sha256 -nodes -out server.csr -newkey rsa:2048 -keyout server.key -config <( cat server.csr.cnf )

- Generate private certificate

  openssl x509 -req -in server.csr -CA rootCA.pem -CAkey rootCA.key -CAcreateserial -out server.crt -days 3650 -sha256 -extfile v3.ext

`		`At the end two additional files are generated, server.key server.crt


- Haproxy, requires to combine server.key server.crt, run the following command:

  bash -c 'cat server.key server.crt >> server.pem'
  chmod 600 server.pem

- Update the HAPROXY configuration

`	`set in /etc/haproxy/haproxy.cfg:

frontend main

`	`bind \*:80

`	`bind \*:443 ssl crt /etc/haproxy/server.pem

`	`redirect scheme https if !{ ssl\_fc }

`	`default\_backend myserver

backend myserver

`	`wpserver 192.168.56.10:80

comment line:

\#	balance	roundrobin

- Update Port forwarding on Virtual box by configuring Virtual box Network:

  Name: Server http

  Protocol: TCP

  Host IP: -

  Host Port:80

  Guest IP: 192.168.56.12

  Guest Port: 80

  Name: Server https

  Protocol: TCP

  Host IP: -

  Host Port:443

  Guest IP: 192.168.56.12

  Guest Port: 443

- Add the new certificate on MAC

`		`Access the Key ring on MAC and the new certificate rootCA.pem

`		`Confirm the certificate as trusted by clicking twice on the certificate (with 			warning)

`	`**Specify and set the CA Bundle Path**

- Use the --cacert Option with curl:

`	`curl --cacert /etc/ssl/rootCA.pem <https://mywpsite.local>

- Set the CURL\_CA\_BUNDLE Environment Variable:

`	`export CURL\_CA\_BUNDLE=/etc/ssl/rootCA.pem

`	`This will make curl use the specified CA bundle for all requests only in the 	current session. 

- In order to specify the CA\_BUNDLE permanently

  `	`locate on MAC the file .zshrc (/users/massimo) considering that is a 	hiddenfile

- Add the ambient variable through the following line:

`	`export CURL\_CA\_BUNDLE=/etc/ssl/rootCA.pem



`	`**Provide the "curl" command**

- curl <https://mywpsite.local>

  massimo@Host-007 ~ % curl https://mywpsite.local  
```
  <!DOCTYPE html>

  <html lang="en-US">

  <head>

  `	`<meta charset="UTF-8" />

  `	`<meta name="viewport" content="width=device-width, initial-scale=1" />

  <meta name='robots' content='noindex, nofollow' />

  `	`<style>img:is([sizes="auto" i], [sizes^="auto," i]) { contain-intrinsic-size: 3000px 1500px }</style>

  `	`<title>MAX Test</title>   ```.....
