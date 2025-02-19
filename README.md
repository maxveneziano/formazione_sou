# Formazione Sou

**Process for the creation of a network of 3 server - Wordpress, MariaDB and Haproxy - using Vagrant/Ansible**

References:

https://medium.com/@alokkavilkar/a-guide-to-ansible-with-vagrant-6fccf9c49591

https://www.theurbanpenguin.com/provisioning-vagrant-with-ansible/

- Download and install VirtualBox from *https://www.virtualbox.org/wiki/Downloads*
- Download and install Vagrant from *https://www.vagrantup.com/downloads.html*
- *Create directory for Vagrant project in MAC*

  mkdir -p $HOME/vagrant/rockylinux

  cd $HOME/vagrant/ rockylinux

- Download Rocky Linux box from the Vagrant repository located at: 

`	`*https://app.vagrantup.com/boxes/search*

*https://portal.cloud.hashicorp.com/vagrant/discover/rockylinux/9*

(for Box download use this link: https://dl.rockylinux.org/pub/rocky/9/images/x86\_64/)

- Load Vagrant Rocky Linux Box from file using the command: 

vagrant box add <boxname> <location of downloaded box file>

vagrant box add hashicorp/Rockylinux9.5 Rocky-9-Vagrant-Vbox-9.5-20241118.0.x86\_64.box

- Check the loaded Boxes 

  vagrant box list -i

- Modify Vagrantfile for creating 3 VMs, assign IP address and define port forwarding of the SSH port In Vagrantfile.

`	`Example for Node 1:

wordpress.vm.network "private\_network", ip: "192.168.56.10"

wordpress.vm.network "forwarded\_port", id: "ssh", guest: 22, host: 2220, host\_ip: 	"127.0.0.1"

*Node1 (wordpress) 192.168.56.10 SSH Port Forwarding 127.0.0.1 (host: port 2020)*

*Node2 (mariadb) 	192.168.56.11 SSH Port Forwarding 127.0.0.1 (host: port 2021)*

*Node3 (haproxy) 	192.168.56.12 SSH Port Forwarding 127.0.0.1 (host: port 2022)*

- On MAC, install Brew (required for install Ansible):

  https://brew.sh/

  /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

  And provide additional commands:

  echo >> /Users/massimo/.zprofile                                                               

  massimo@Host-007 ~ % echo 'eval "$(/usr/local/bin/brew shellenv)"' >> /Users/massimo/.zprofile

  massimo@Host-007 ~ %  eval "$(/usr/local/bin/brew shellenv)"

  massimo@Host-007 ~ % brew help

  brew update

  Then:

`	`brew install ansible (on MAC)

`	`Check / Validate Ansible:

ansible --version

- Modify Vagrantfile in order to use Ansible as provisioner :

  config.vm.define "wordpress" do |wordpress|

  `		`wordpress.vm.box = "hashicorp/Rockylinux9.5"

  `  	`wordpress.vm.hostname = "VGNode1"

  `		`wordpress.vm.provision "ansible", playbook: "Playbook.yml"

- Generate in the same directory as Vagrant, the file Playbook.yml (indented!) in order to describe the automated operations:
  - Create a new user on all 3 systems
  - Update all packets on all 3 systems
  - Install and configure MariaDB, PHP on node 2
  - Install and configure PHP, APACHE and Wordpress on node 1
  - Install and configure HAPROXY on node 3
  - Copy self signed server.pem certificate to node 3


**Install and Configure MariaDB on node 2 (mariadb)**

Reference: https://citizix.com/how-to-install-and-initialize-mysql-8-on-rocky-linux-8-using-ansible/

- Define in a file the variable mariadb\_root\_password
- Create a dummy file my.cnf.j2 in the current directory with the following contents:

  [client]

  user=root

  password={{ mariadb\_root\_password }}

- Copy using an Ansible task to: /root/.my.cnf as a root user.

  - name: Add .my.cnf to user home

  `  `template:

  `    `src: my.cnf.j2

  `    `dest: /root/.my.cnf

- Reload privileges with this task:

  - name: Reload privilege tables

  `  `command: |

  `    `mysql -p{{ mariadb\_root\_password }} -ne "{{ item }}"

  `  `with\_items:

  `    `- FLUSH PRIVILEGES

  `  `changed\_when: False

- Install python3-PyMySQL

  `     `dnf: 

  `       `name:

  `          `- python3-PyMySQL

  `       `state: latest       

- Install MariaDB server

  `     `dnf: 

  `       `name:

  `          `- mariadb

  `          `- mariadb-server

  `       `state: latest

- `   `Enable MariaDB on system reboot

  `     `service: name=mariadb enabled=yes

- Start service MariaDB, if not started

  `     `service:

  `        `name: mariadb

  `        `state: started

- `  `Update MariaDB root user password

  `     `mysql\_user:

  `       `name: root

  `       `login\_password: "{{ mariadb\_root\_password }}"

  `       `check\_implicit\_admin: yes

  `       `host: "{{ item }}"

  `       `password: "{{ mariadb\_root\_password }}"

  `       `login\_unix\_socket: /var/lib/mysql/mysql.sock

  `       `state: present

  `     `with\_items:

  `       `- localhost

  `       `- 127.0.0.1

  `       `- ::1

- `   `Create Maria DB user

  `     `mysql\_user:

  `       `name: maxdbuser

  `       `password: 12345

  `       `login\_host: localhost

  `       `login\_user: "root"

  `       `login\_password: "{{ mariadb\_root\_password }}"

  `       `priv: "\*.\*:ALL,GRANT"

  `       `host: '%'

  `       `state: present

- `   `Reload MariaDB privilege tables

  `     `command:

  `        `mysql -p{{ mariadb\_root\_password }} -ne "{{ item }}"

  `     `with\_items:

  `        `- FLUSH PRIVILEGES

  `     `changed\_when: False

- `   `Creating a new database to be used by WordPress

  `     `mysql\_db:

  `        `login\_user: "root"

  `        `login\_password: "{{mariadb\_root\_password}}"

  `        `name: "maxdb"

  `        `state: present

- `   `MariaDB firewall configuration (port 3306)

  `     `firewalld:

  `       `service: mysql

  `       `permanent: yes

  `       `state: enabled

- `  `Restart service firewalld

  `     `service:

  `       `name: firewalld

  `       `state: restarted

- Test he MariaDB changes: (**testdb** database)

**mysql -u root -h localhost -p** Enter password: (12345)
Welcome to the MySQL monitor.  Commands end with ; or \g. 
Your MySQL connection id is 17 
Server version: 8.0.22-0ubuntu0.20.04.3 (Ubuntu)  mysql> **use testdb;** 
Reading table information for completion of table and column names You can turn off this feature to get a quicker startup with -A  Database changed 
mysql> **show tables;** 
+------------------+ 
| Tables\_in\_testdb | 
+------------------+ 
| test             | 
+------------------+ 
1 row in set (0.00 sec)  mysql> **select \* from test;** 
+--------------------+ 
| message            | 
+--------------------+ 
| Ansible To Do List | 
| Get ready          | 
| Ansible is fun     | 
+--------------------+ 
3 rows in set (0.00 sec)


**Install and configure PHP, APACHE and Wordpress (node 1 - wordpress)**

References:

https://www.infinitypp.com/ansible/install-wordpress-using-ansible-ubuntu-php7/


`	`**Install and configure PHP on node 1 (wordpress)**



- `   `Install PHP

`     `dnf: 

`       `name:

`          `- php

`          `- php-mysqlnd

`          `- php-gd

`          `- php-mbstring

`          `- php-xml

`       `state: latest

`	`**Install and configure APACHE on node 1 (wordpress)**

- `  `Install httpd APACHE

`     `action: yum name=httpd state=installed

- Enable Apache on system reboot

`     `service: name=httpd enabled=yes

- Start service httpd, if not started

`     `service:

`        `name: httpd

`        `state: started

- Install python3-libsemanage

`     `dnf: 

`       `name:

`          `- python3-libsemanage

`       `state: latest       

- `   `Set httpd\_can\_network\_connect and httpd\_network\_connect\_db flags on and keep it persistent across reboots in order to allow the connection with the remote MariaDB database (due to SELINUX limitation):

`     `seboolean:

`          `name: httpd\_can\_network\_connect

`          `state: true

`          `persistent: yes

`     `seboolean:

`          `name: httpd\_can\_network\_connect\_db

`          `state: true

`          `persistent: yes

`   	`**Node 1 (wordpress) firewall configuration (port 80)**

- `   `Install firewalld

`     `action: yum name=firewalld state=installed

- `   `Enable firewalld on system reboot

`     `service: name=firewalld enabled=yes

- `   `Firewalld http configuration

`     `firewalld:

`       `service: http

`       `permanent: yes

`       `state: enabled

- `   `Restart firewalld service

`     `service:

`       `name: firewalld

`       `state: restarted

`	`**Install and configure Wordpress on node 1 (wordpress)**

- `   `Install WORDPRESS

`     `unarchive: 

`       `src: https://wordpress.org/latest.tar.gz

`       `dest: /var/www/html/

`       `remote\_src: yes

- `   `Copy Wordpress to /var/www/html - necessary for SELINUX limitations

`     `copy: 

`       `src: /var/www/html/wordpress/

`       `dest: /var/www/html/

`       `remote\_src: yes   

- `   `Change owner and permission of /var/www/html/

`     `ansible.builtin.file:

`      `path: /var/www

`      `owner: apache

`      `group: apache

`      `mode: '755'

`      `recurse: yes

- `   `Add modified wp-config.php to Wordpress directory.

`     `template:

`        `src: wp-config.php

`        `dest: /var/www/html/wp-config.php

`   	`In case wp-config.php is not present provide the following data in the local file or follow 	the configuration data proposed by wordpress page http://192.168.56.10/wp-	admin/setup-config.php

`	`DB: maxdb

`	`User: maxdbuser

`	`Password: 12345

`	`Host: 192.168.56.11

`	`**Install and configure HAPROXY on node 3 (haproxy)**

Reference:

https://ranga-mani54.medium.com/configuring-haproxy-using-ansible-playbook-27c1ee8b511a https://abhishekverma109.hashnode.dev/automating-haproxy-configuration-with-ansible	

https://www.redhat.com/en/blog/reverse-proxy-ansible

https://medium.com/@haroldfinch01/how-to-create-a-directory-using-ansible-aed142c041eb

- Install the last version of HAPROXY

`     `dnf: 

`   `name:

`     `- haproxy

`   `state: latest

- Creates /etc/haproxy directory and change owner, group and permissions

`     `ansible.builtin.file:

`     `path: /etc/haproxy

`     `state: directory

`     `owner: root

`     `group: root

`     `mode: '775'

- Copy local modified haproxyg.cfg (on MAC) to haproxy remote directory

`     `copy:

`        `src: /Users/massimo/vagrant/rockylinux/HAPROXY\_FILES/haproxy.cfg

`        `dest: /etc/haproxy/haproxy.cfg

`	`The /etc/haproxy/haproxy.cfg contains:

`	`frontend main

`	`bind \*:80

`	`bind \*:443 ssl crt /etc/haproxy/server.pem

`	`default\_backend myserver

backend myserver

`	`wpserver 192.168.56.10:80

commented line:

\#	balance	roundrobin


- ` `Copy local server.pem (on MAC) to haproxy remote directory

`     `copy:

`        `src: /Users/massimo/vagrant/rockylinux/HAPROXY\_FILES/server.pem

`        `dest: /etc/haproxy/server.pem

- `  `Restart service HA PROXY

`     `service:

`          `name: haproxy

`          `state: restarted

- `   `Install firewalld

`     `action: yum name=firewalld state=installed

- `   `Enable firewalld on system reboot

`     `service: name=firewalld enabled=yes

- `   `Firewalld http configuration (open port 80) 

`     `firewalld:

`       `service: http

`       `permanent: yes

`       `state: enabled

- `   `Firewalld https configuration (open port 443)

`     `firewalld:

`       `service: https

`       `permanent: yes

`       `state: enabled

- `   `Restart service firewalld

`     `service:

`       `name: firewalld

`       `state: restarted

- Add mywpsite.local to MAC /etc/hosts:

127\.0.0.1	mywpsite.local

- In Vagrant file be sure to forward correctly ports on Node 3 (haproxy - 192.168.56.12):

`	`haproxy.vm.network "private\_network", ip: "192.168.56.12"

`  	`haproxy.vm.network "forwarded\_port", guest: 80, host: 80

` 	`haproxy.vm.network "forwarded\_port", guest: 443, host: 443


- Complete manually the Wordpress configuration following the form that appears pointing the browser to https://mywpsite.local 

Create a Wordpress administrator user by pointing a browser to https://mywpsite.local, providing:

User massimo

Password: 12345

After been logged as massimo (administrator) in Wordpress Settings -> General Settings set:

WordPress Address (URL) = http://mywpsite.local

Site Address (URL) = http://mywpsite.local

- Check the correct connection to the test page at https://192.168.56.10/ using the browser on MAC (Chrome)

`	`**Provide the "curl" command from MAC terminal**

- curl <https://mywpsite.local>

  massimo@Host-007 ~ % curl https://mywpsite.local
```
  <!DOCTYPE html>

  <html lang="en-US">

  <head>

  <meta charset="UTF-8" />

  <meta name="viewport" content="width=device-width, initial-scale=1" />

  <meta name='robots' content='noindex, nofollow' />

  <style>img:is([sizes="auto" i], [sizes^="auto," i]) { contain-intrinsic-size: 3000px 1500px }</style>

  <title>Max SITE Test 3</title>
  ```
     ....

**Self Signed Certificate generation and installation – node 3**

References:

https://sharmank.medium.com/self-signed-certificate-and-use-them-in-haproxy-and-allow-certificate-in-macos-26c3aad316bb

https://medium.com/azure-cloud-techinical/using-ssl-certificates-with-haproxy-8c4d941d9afd

https://webhostinggeeks.com/howto/how-to-configure-ssl-certificate-in-haproxy/

https://www.haproxy.com/documentation/haproxy-configuration-tutorials/authentication/client-certificate-authentication/

https://www.haproxy.com/documentation/haproxy-configuration-tutorials/ssl-tls/client-side-encryption/

https://betterstack.com/community/questions/curl-ssl-certificate-problem/

Self signed certificates were already created manually in the previous activity.

For convenience, using Vagrant/Ansible the previous generated certificates are used.

On Node 3 - haproxy, the /etc/haproxy/server.pem file is copied to /etc/haproxy/ while the rootCA.pem is copied and trusted in MAC Keychain ("portachiavi")

For reference purposes, at the bottom, in the appendix, is listed the procedure to create a Self Signed Certificate.



**APPENDIX**

**Ansible definitions**

- ***playbook***: a playbook is a [*.yml*](https://it.wikipedia.org/wiki/YAML) (or *.yaml*) file where you specify the roles used to configure your host;
- ***role***: the role is a directory with a defined structure (follow the official [*Ansible documentation*](https://docs.ansible.com/ansible/latest/user_guide/playbooks_reuse_roles.html)), which contains all the tools to install in your host;
- ***inventory***: an inventory file is a plain text file (without extension), which contains the definition and some parameters.

**Useful commands**

- For vagrant ssh

`		`Start the vagrant VM 

`		`vagrant up

`		`Stop the vagrant VM 

`		`vagrant halt

`		`Connect to the VM with ssh.

`		`vagrant SSH 

`		`vagrant ssh wordpress	(wordpress = nome host)                                         

`		`vagrant ssh mariadb

`		`vagrant ssh haproxy

`		`Check vagrant VM:

`	`vagrant ssh

`	`free -m

- For ifconfig	sudo dnf install net-tools	
- For users list

  cat /ect/passwd

- Check the Playbook.yml sintax

  ansible-playbook playbook.yml --syntax-check

- Provision the pending provisioned parts

  vagrant provision

- To verify the presence of mariadb Ansible packet

  ansible-galaxy collection list

- To set the root password

  sudo passwd root

- it will show you all locations of \*.cnf files

  find /etc -name \*.cnf

- Sending through terminal to a remote hosts Shell commands:

  For instance, to find out the memory usage on your host1 machine, you could use the following:

`	`ansible -m shell -a 'free -m' host1

- Delete files recursively

`	 `sudo rm -rf wordpress/

- Verify presence of user php

`	`sudo ps aux | grep php-fpm

- Install nano

`	`sudo dnf install nano

- HAProxy's status 

`	`sudo systemctl status haproxy.

- Vagrant 's status (returns ID and status)

  vagrant global-status

**Wordpress Redirect issue**

During the implementation some problem of multiple redirection were experienced on Wordpress server and due to a wrong haproxy.cfg configuration. 

For reference purposes are listed some URL used for identify the solution:

https://stackoverflow.com/questions/49053286/redirect-a-url-to-www-with-haproxy-getting-too-many-redirects

https://serverfault.com/questions/955110/wp-admin-redirect-loop-when-behind-apache-reverse-proxy

https://www.haproxy.com/blog/redirect-http-to-https-with-haproxy



**Self Signed Certificate process of generation and installation – node 3**

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

`		`At the end two files will be generated, `rootCA.key` `rootCA.pem`

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

- Add the new certificate on MAC

  `		`Access the Keychain ("portachiavi") on MAC and the new certificate rootCA.pem

  `		`Confirm the certificate as trusted by clicking twice on the certificate (with 			warning)

`	`**Specify and set the CA Bundle Path on MAC**

- Use the --cacert Option with curl:

`	`curl --cacert /etc/ssl/rootCA.pem <https://mywpsite.local>

- Set the CURL\_CA\_BUNDLE Environment Variable:

`	`export CURL\_CA\_BUNDLE=/etc/ssl/rootCA.pem

`	`This will make curl use the specified CA bundle for all requests only in the 	current session. 

- In order to specify the CA\_BUNDLE permanently

  `	`locate on MAC the file .zshrc (/users/massimo) considering that is a 	hiddenfile

- Add the ambient variable through the following line:

`	`export CURL\_CA\_BUNDLE=/etc/ssl/rootCA.pem 
