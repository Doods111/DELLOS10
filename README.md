# Vagrant for Dell OS10
This is an initial draft.

## Step by step procedure to run the script: 
1. clone the project
2. add the vagrant OS10 box
   vagrant box add --name os10 os10.box
3. edit the ./group_vars/all.yaml file and enter the number of spines and leaves you need in your test environment (Consider the host memory and CPU available)
4. run the generate_inventory.yaml playbook
   ansible-playbook generate_inventory.yaml
5. run the main.yaml using the new inventory file
   ansible-playbook main.yaml -i inventory
6. run vagrant up
