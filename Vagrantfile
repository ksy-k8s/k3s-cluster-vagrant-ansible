# -*- mode: ruby -*-
# vi: set ft=ruby :

N_AGENT = 3
server_ip = "192.168.2.50"
agent_ip = Array.new(N_AGENT) { |i| "192.168.2.5#{i+1}"}

Vagrant.configure("2") do |config|
  
  config.vm.box = "ubuntu/jammy64" # ubuntu 22.04
  config.vm.synced_folder ".", "/vagrant", disabled: true

  #=============#
  # Agent Nodes #
  #=============#
  (1..N_AGENT).each do |i|
    config.vm.define "k3s-agent-#{i}" do |agent|
      agent.vm.hostname = "k3s-a#{i}"
      agent.vm.network "private_network", ip: agent_ip[i-1] # enp0s8
      agent.vm.provider "virtualbox" do |vb|
        vb.name = "k3s-a#{i}"
        vb.cpus = 2
        vb.memory = "2048"
        vb.customize ["modifyvm", :id, "--groups", "/K3s Cluster"]
      end
      agent.vm.provision "shell", inline: $common_script
    end
  end

  #=============#
  # Server Node #
  #=============#
  config.vm.define "k3s-server" do |server|
    server.vm.hostname = "k3s-s"
    server.vm.network "private_network", ip: server_ip # enp0s8
    server.vm.network "private_network", type: "dhcp"  # enp0s9
    server.vm.provider "virtualbox" do |vb|
      vb.name = "k3s-s"
      vb.cpus = 4
      vb.memory = "4096"
      vb.customize ["modifyvm", :id, "--groups", "/K3s Cluster"]
    end
    server.vm.provision "shell", inline: $common_script
    server.vm.provision "shell", inline: $server_script, privileged: false
    server.vm.provision "file", source: "./inventory.txt", destination: "~/inventory.txt"
    server.vm.provision "file", source: "./playbook.yaml", destination: "~/playbook.yaml"
    server.vm.provision "shell", inline: $ansible_script, privileged: false, args: N_AGENT # pass N_AGENT as first argument
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
  sudo apt install -y ansible sshpass

  ssh-keygen -f /home/vagrant/.ssh/id_rsa -N ''
  sudo sed -i 's/#   StrictHostKeyChecking ask/StrictHostKeyChecking no/g' /etc/ssh/ssh_config
  for ((i = 0; i <= $1; i++)); do sshpass -p vagrant ssh-copy-id "vagrant@192.168.2.5${i}"; done

  ansible-playbook playbook.yaml -i inventory.txt
SCRIPT
