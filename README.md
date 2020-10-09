# Vagrant for Dell OS10
This is an initial draft.
Tested with: 
   * ansible 2.9.13
   * python 3.6
   * vagrant 2.2.10

## Step by step procedure to set up the environment: 
1. Clone the project
   * *git clone https://github.com/Doods111/DELLOS10.git*
2. Add the vagrant OS10 box.
   * *vagrant box add --name os10 os10.box*
   * *sudo cp  ~/.vagrant.d/boxes/os10/0/libvirt/OS10-Disk-1.0.0.vmdk /var/lib/libvirt/images/*
3. Edit the ./group_vars/all.yaml file and enter the number of spines and leaves you need in your test environment (Consider the host memory and CPU available)
4. Run the generate_inventory.yaml playbook
   * *ansible-playbook generate_inventory.yaml*
5. Run the main.yaml using the new inventory file
   * *ansible-playbook main.yaml -i inventory*
6. Run vagrant up
   * If only the physical topology is required without provisioning the configs use:
       * *vagrant up --no-provision*
   * If the vagrant is up and a provisionning is required, use:
      * *vagrant provision*
      
