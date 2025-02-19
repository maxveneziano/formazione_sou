Vagrant.configure("2") do |config|

# Necessary for avoid an hang up due to a Vagrant bug 
        config.vm.provider "virtualbox" do |vb|
          vb.customize [ "modifyvm", :id, "--uartmode1", "disconnected" ]
          vb.customize [ "modifyvm", :id, "--uartmode1", "file", File::NULL ]
          vb.customize [ "modifyvm", :id, "--uart1", "0x3F8", "4" ]
          vb.customize [ "modifyvm", :id, "--cableconnected1", "on" ]
        end

##### Wordpress Node 1 ##########################
	config.vm.define "wordpress" do |wordpress|
		wordpress.vm.box = "hashicorp/Rockylinux9.5"
  		wordpress.vm.hostname = "VGNode1"
		wordpress.vm.provision "ansible", playbook: "Playbook.yml"

  #	config.vm.boot_timeout = 600
  
  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # NOTE: This will enable public access to the opened port
  # config.vm.network "forwarded_port", guest: 80, host: 8080
     
  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine and only allow access
  # via 127.0.0.1 to disable public access
  # config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"

		wordpress.vm.network "private_network", ip: "192.168.56.10"
  		wordpress.vm.network "forwarded_port", id: "ssh", guest: 22, host: 2220, host_ip: "127.0.0.1"
  #             wordpress.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"
  #		wordpress.vm.network "forwarded_port", guest: 80, host: 80
  #		Dalla riga precedente occorre togliere host_ip: "127.0.0.1" per problemi di gestione MAC per la
  #		ridirezione di porte privilegiate <1024

		wordpress.vm.provider "virtualbox" do |vb|
			vb.memory = "2048"
  		end
	end

##### mariadb Node 2 ##########################

	config.vm.define "mariadb" do |mariadb|
		mariadb.vm.box = "hashicorp/Rockylinux9.5"
  		mariadb.vm.hostname = "VGNode2"
		mariadb.vm.provision "ansible", playbook: "Playbook.yml"
  
  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # NOTE: This will enable public access to the opened port
  # config.vm.network "forwarded_port", guest: 80, host: 8080
     
  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine and only allow access
  # via 127.0.0.1 to disable public access
  # config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"

		mariadb.vm.network "private_network", ip: "192.168.56.11"
		mariadb.vm.network "forwarded_port", id: "ssh", guest: 22, host: 2221, host_ip: "127.0.0.1"

		mariadb.vm.provider "virtualbox" do |vb|
			vb.memory = "2048"
  		end
	end
##### haproxy Node 3 ##########################

	config.vm.define "haproxy" do |haproxy|
		haproxy.vm.box = "hashicorp/Rockylinux9.5"
  		haproxy.vm.hostname = "VGNode3"
		haproxy.vm.provision "ansible", playbook: "Playbook.yml"
  
  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # NOTE: This will enable public access to the opened port
  # config.vm.network "forwarded_port", guest: 80, host: 8080
     
  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine and only allow access
  # via 127.0.0.1 to disable public access
  # config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"

		haproxy.vm.network "private_network", ip: "192.168.56.12"
		haproxy.vm.network "forwarded_port", id: "ssh", guest: 22, host: 2222, host_ip: "127.0.0.1"

 		haproxy.vm.network "forwarded_port", guest: 80, host: 80
 		haproxy.vm.network "forwarded_port", guest: 443, host: 443
 #		Occorre togliere host_ip: "127.0.0.1" per problemi di gestione MAC per la
 #		ridirezione di porte privilegiate <1024


		haproxy.vm.provider "virtualbox" do |vb|
			vb.memory = "2048"
  		end
	end

# 	config.vm.provider "virtualbox" do |vb|
#        	vb.customize ["modifyvm", :id, "--cableconnected1", "on"]
#    	end

end