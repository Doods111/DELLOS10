# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don"t change it unless you know what
# you"re doing.
VERSION="os10"
SPINE = 2
LEAF = 3
SERVER_PER_SW = 1
Vagrant.configure("2") do |config|
  config.ssh.username = "admin"
  config.ssh.password = "admin"
  config.ssh.sudo_command = "system bash"

  
  config.ssh.shell = "system bash"
  (1..SPINE).each do |i|
    config.vm.define "spine#{i}" do |sw|
            
    
        sw.vm.box = VERSION
        (1..LEAF).each do |j|
            sw.vm.network :private_network, :libvirt__tunnel_type => "udp", :libvirt__tunnel_port => "#{j + SPINE}111#{i}", :libvirt__tunnel_local_ip => "127.1.1.#{i}" , :tunnel_local_port => "#{i}111#{j}", :libvirt__tunnel_ip => "127.1.1.#{j + SPINE}"
        end
        sw.vm.provider :libvirt do |libvirt|
            libvirt.memory = "2048"
            libvirt.cpus = "1"
            libvirt.nic_model_type  = "e1000"
            libvirt.cpu_mode = "host-passthrough"
    #  sw.vm.network :public_network, :dev => "virbr0", :mode => "bridge", :type => "bridge"
        end
        sw.vm.synced_folder ".", "/vagrant", disabled: true
    end
  end
  (1..LEAF).each do |j|

    config.vm.define "leaf#{j}" do |sw|
         
        sw.vm.box = VERSION
        
       (1..SPINE).each do |i|
       
            sw.vm.network :private_network, :libvirt__tunnel_type => "udp", :libvirt__tunnel_port => "#{i}111#{j}" , :libvirt__tunnel_local_ip => "127.1.1.#{j+SPINE}" , :tunnel_local_port => "#{j + SPINE}111#{i}", :libvirt__tunnel_ip => "127.1.1.#{i}"

        end
        (1..SERVER_PER_SW).each do |k|
            sw.vm.network :private_network, :libvirt__tunnel_type => "udp", :libvirt__tunnel_port => "#{(LEAF  +  SPINE + j +  k)-1}111#{k}" , :libvirt__tunnel_local_ip => "127.1.1.#{j+SPINE}" , :tunnel_local_port => "#{j + SPINE}111#{SPINE + k}", :libvirt__tunnel_ip => "127.1.1.#{(LEAF  +  SPINE + j +  k)-1}"
        end
        
        sw.vm.provider :libvirt do |libvirt|
            libvirt.memory = "2048"
            libvirt.cpus = "1"
            libvirt.nic_model_type  = "e1000"
            libvirt.cpu_mode = "host-passthrough"
 #      sw.vm.network :public_network, :dev => "virbr0", :mode => "bridge", :type => "bridge"
        end
        sw.vm.synced_folder ".", "/vagrant", disabled: true
    end
  end

  ##########################
  ## Servers         #######
  ##########################
  srvcounter=0
  (1..LEAF).each do |leaf_sw|
      (1..SERVER_PER_SW).each do |srvperswcount|
            srvcounter = srvcounter  +  1
            srv_name  = ( "srv#{leaf_sw}#{srvperswcount}").to_sym
            
            config.vm.define srv_name do |srv|
                config.ssh.username = "vagrant"
                config.ssh.password = "vagrant"
                config.ssh.sudo_command = "sudo -E -H %c"
                config.ssh.shell = "bash -l"
            
                # Uncomment 1 line below to use minitrusty image instead of Tiny Core linux. Comment Tiny Core further down. "minitrusty" takes 512MB RAM per instance.
                srv.vm.box = "nrclark/xenial64-minimal-libvirt"
            
                # Tiny-Core linux needs extras cuz not well supported with Vagrant. Only 64M of RAM per instance! Has ping, ssh, ssh server.
                # Comment ines below this line and uncomment minitrusty64 above to swap server images
                # srv.vm.box = "olbat/tiny-core-micro"
                #srv.vm.synced_folder ".", 
                #    "/vagrant", 
                #    disabled: true
                #config.ssh.shell = "sh -l"
                # End Tiny-Core
                
                srv.vm.provider :libvirt do |libvirt|
                    libvirt.nic_model_type  = "e1000"
                end
                
                srv.vm.hostname = "#{srv_name}"
                srv.vm.network :private_network, ip: "10.10.10.#{leaf_sw}#{srvperswcount}", 
                :libvirt__host_ip => "10.10.10.#{leaf_sw}#{srvperswcount}", 
                :libvirt__tunnel_type => "udp",
                :libvirt__tunnel_local_ip => "127.1.1.#{(srvperswcount+leaf_sw + LEAF + SPINE)-1}" ,
                :tunnel_local_port => "#{(srvperswcount+leaf_sw + LEAF + SPINE)-1}111#{srvperswcount}",
                :libvirt__tunnel_port => "#{leaf_sw + SPINE}111#{srvperswcount + SPINE}" ,
                :libvirt__tunnel_ip => "127.1.1.#{leaf_sw + SPINE}"              
                srv.ssh.insert_key = true
                srv.vm.provision "shell",
                    inline: "sudo ifconfig eth1 10.10.10.#{leaf_sw+srvperswcount-1}0/24" # ubuntu
            end
        end
   end
    #############################
    # Box provisioning    #######
    #############################
    config.vm.provision "ansible" do |ansible|
#                ansible.limit = "all"
                ansible.config_file = "ansible.cfg"
                ansible.groups = {"spines"       => ["spine[1:#{SPINE}]"],
                "leaves"       => ["leaf[1:#{LEAF}]"],
                "dellos10:children" => ["spines", "leaves" ]}
#                ansible.inventory_path = "./inventory"
                ansible.playbook = "mylab.yaml"
    
    end

#  config.vm.provision "shell", inline: <<-SHELL
#      sleep 30
#      configure
#      hostname spine01
#      interface Management1/1/1
#        ip address 192.168.56.101/24"
#  SHELL
    #  interface Ethernet3
    #     shutdown

end
