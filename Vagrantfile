# -*- mode: ruby -*-
# vi: set ft=ruby :

N = 3
server_ip = "192.168.2.50"
agent_ip = Array.new(N) { |i| "192.168.2.5#{i+1}"}
ansible_ip = "192.168.2.59"

Vagrant.configure("2") do |config|
  
  config.vm.box = "ubuntu/focal64"
  config.vm.box_version = "20220311.0.0"
  config.vm.synced_folder ".", "/vagrant", disabled: true

  #=============#
  # Server Node #
  #=============#
  config.vm.define "server" do |server|
    server.vm.hostname = "k3s-s"
    server.vm.network "private_network", ip: server_ip
    server.vm.network "public_network", bridge: "Realtek PCIe GbE Family Controller"
    server.vm.provider "virtualbox" do |vb|    
      vb.name = "k3s-s"
      vb.cpus = 4
      vb.memory = "4096"
      vb.customize ["modifyvm", :id, "--groups", "/K3s Cluster"]
    end
    server.vm.provision "shell", inline: $common_script
    server.vm.provision "shell", inline: $server_script, privileged: false
  end

  #=============#
  # Agent Nodes #
  #=============#
  (1..N).each do |i|
    config.vm.define "agent-#{i}" do |agent|
      agent.vm.hostname = "k3s-a#{i}"
      agent.vm.network "private_network", ip: agent_ip[i-1]
      agent.vm.provider "virtualbox" do |vb|    
        vb.name = "k3s-a#{i}"
        vb.cpus = 2          
        vb.memory = "2048"   
        vb.customize ["modifyvm", :id, "--groups", "/K3s Cluster"]
      end
      agent.vm.provision "shell", inline: $common_script
    end
  end

  #==============#
  # Ansible Node #
  #==============#
  config.vm.define "ansible" do |ansible|
    ansible.vm.hostname = "k3s-a"
    ansible.vm.network "private_network", ip: ansible_ip
    ansible.vm.network "public_network", bridge: "Realtek PCIe GbE Family Controller"
    ansible.vm.provider "virtualbox" do |vb|    
      vb.name = "k3s-a"
      vb.cpus = 2
      vb.memory = "2048"
      vb.customize ["modifyvm", :id, "--groups", "/K3s Cluster"]
    end
    ansible.vm.provision "file", source: "./inventory.txt", destination: "~/inventory.txt"
    ansible.vm.provision "file", source: "./playbook.yaml", destination: "~/playbook.yaml"
    ansible.vm.provision "shell", inline: $common_script
    ansible.vm.provision "shell", inline: $ansible_script, privileged: false, args: N
  end

end

$common_script = <<-SCRIPT
apt update #&& apt upgrade -y && apt autoremove -y
sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/' /etc/ssh/sshd_config
systemctl restart ssh
SCRIPT

$server_script = <<-SCRIPT
curl -s https://raw.githubusercontent.com/ehsqjfwk99999/config-files/main/.bashrc >>/home/vagrant/.bashrc
curl -s https://raw.githubusercontent.com/ehsqjfwk99999/config-files/main/.vimrc >>/home/vagrant/.vimrc
SCRIPT

$ansible_script = <<-SCRIPT
sudo apt install -y ansible=2.9.6+dfsg-1
sudo apt install -y sshpass

ssh-keygen -f /home/vagrant/.ssh/id_rsa -N ''
sudo sed -i 's/#   StrictHostKeyChecking ask/StrictHostKeyChecking no/g' /etc/ssh/ssh_config
for ((i = 0; i <= $1; i++)); do sshpass -p vagrant ssh-copy-id "vagrant@192.168.2.5${i}"; done

ansible-playbook playbook.yaml -i inventory.txt
SCRIPT

