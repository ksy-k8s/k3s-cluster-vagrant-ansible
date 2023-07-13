# -*- mode: ruby -*-
# vi: set ft=ruby :

N_AGENT = 3
# if ip is changed, change the ip in install_k3s, hosts.txt, playbook.yaml
server_ip = "192.168.101.50"
agent_ip = Array.new(N_AGENT) { |i| "192.168.101.5#{i+1}"}

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
      agent.vm.provision "shell", inline: $common_setup
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
    server.vm.provision "shell", inline: $common_setup
    server.vm.provision "shell", inline: $server_setup, privileged: false
    server.vm.provision "file", source: "./hosts.txt", destination: "~/hosts.txt"
    server.vm.provision "file", source: "./playbook.yaml", destination: "~/playbook.yaml"
    server.vm.provision "shell", inline: $install_k3s, privileged: false, args: N_AGENT # pass N_AGENT as first argument
  end

end

$common_setup = <<-SCRIPT
  apt update #&& apt upgrade -y && apt autoremove -y
  sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/' /etc/ssh/sshd_config
  systemctl restart ssh
SCRIPT

$server_setup = <<-SCRIPT
  sudo sed -i 's/# set bell-style none/set bell-style none/' /etc/inputrc
  curl -s https://raw.githubusercontent.com/ehsqjfwk99999/config-files/main/.bashrc >>~/.bashrc
  curl -s https://raw.githubusercontent.com/ehsqjfwk99999/config-files/main/.vimrc >>~/.vimrc
SCRIPT

$install_k3s = <<-SCRIPT
  sudo apt install -y sshpass
  sudo apt install -y ansible=2.10.7+merged+base+2.10.8+dfsg-1 # fixed

  ssh-keygen -f ~/.ssh/id_rsa -N ''
  sudo sed -i 's/#   StrictHostKeyChecking ask/StrictHostKeyChecking no/g' /etc/ssh/ssh_config
  for ((i = 0; i <= $1; i++)); do sshpass -p vagrant ssh-copy-id "vagrant@192.168.101.5${i}"; done

  ansible-playbook playbook.yaml -i hosts.txt

  cp /etc/rancher/k3s/k3s.yaml ~/.kube/config

  echo 'source <(kubectl completion bash)' >>~/.bashrc
  echo 'alias kc=kubectl' >>~/.bashrc
  echo 'complete -o default -F __start_kubectl kc' >>~/.bashrc
SCRIPT
