# -*- mode: ruby -*-
# vi: set ft=ruby :

$install_basics = <<SCRIPT
# Packages required for nomad & consul
echo "=== Installing basics ..."
sudo apt-get update
sudo apt-get install curl jq openssl unzip vim apt-transport-https ca-certificates curl software-properties-common -y
curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo apt-key add -
sudo apt-add-repository "deb [arch=amd64] https://apt.releases.hashicorp.com $(lsb_release -cs) main"
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg |  sudo apt-key add -
sudo apt-key fingerprint 0EBFCD88
sudo add-apt-repository \
      "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
      $(lsb_release -cs) \
      stable"
sudo apt-get update -y

SCRIPT

$install_docker = <<SCRIPT
echo "=== Installing Docker..."

sudo apt-get remove docker docker-engine docker.io
echo '* libraries/restart-without-asking boolean true' | sudo debconf-set-selections
sudo apt-get install -y docker-ce
# Restart docker to make sure we get the latest version of the daemon if there is an upgrade
sudo service docker restart
# Make sure we can actually use docker as the vagrant user
sudo usermod -aG docker vagrant
sudo docker --version
SCRIPT

$install_vault = <<SCRIPT

echo "=== Installing Vault..."
sudo apt-get install vault -y
vault -autocomplete-install
SCRIPT

$install_nomad = <<SCRIPT

echo "=== Installing Nomad..."
NOMAD_VERSION=1.3.5
cd /tmp/
curl -sSL https://releases.hashicorp.com/nomad/${NOMAD_VERSION}/nomad_${NOMAD_VERSION}_linux_amd64.zip -o nomad.zip
unzip nomad.zip
sudo install nomad /usr/bin/nomad
echo "=== Configuring Nomad ..."
sudo mkdir -p /opt/nomad/data
sudo mkdir -p /etc/nomad.d
sudo touch /etc/nomad.d/nomad.hcl

sudo cat <<EOF >> /etc/nomad.d/nomad.hcl
data_dir= "/opt/nomad/data"
bind_addr = "0.0.0.0"
datacenter = "my-nomad-lab"
region = "amsterdam"
server {
  enabled = true
  bootstrap_expect = 1
}
client {
  enabled = false
}
consul {
  address = "127.0.0.1:8500"
  server_service_name = "nomad"
  client_service_name = "nomad-client"
  auto_advertise = true
  server_auto_join = true
  client_auto_join = true
}
EOF
sudo chmod a+w /opt/nomad
sudo chmod a+w /etc/nomad.d
nomad -autocomplete-install

SCRIPT


$install_consul = <<SCRIPT

echo "=== Installing Consul..."
CONSUL_VERSION=1.9.0
curl -sSL https://releases.hashicorp.com/consul/${CONSUL_VERSION}/consul_${CONSUL_VERSION}_linux_amd64.zip > /tmp/consul.zip
unzip /tmp/consul.zip
sudo install consul /usr/bin/consul
(
cat <<-EOF

  [Unit]
  Description=consul agent
  Requires=network-online.target
  After=network-online.target

  [Service]
  Restart=on-failure
  ExecStart=/usr/bin/consul agent -dev
  ExecReload=/bin/kill -HUP $MAINPID

  [Install]
  WantedBy=multi-user.target
EOF
) | sudo tee /etc/systemd/system/consul.service
sudo systemctl enable consul.service
sudo systemctl start consul

for bin in cfssl cfssl-certinfo cfssljson
do
  echo "Installing $bin..."
  curl -sSL https://pkg.cfssl.org/R1.2/${bin}_linux-amd64 > /tmp/${bin}
  sudo install /tmp/${bin} /usr/local/bin/${bin}
done


SCRIPT


Vagrant.configure(2) do |config|
  # Increase memory for Virtualbox
  config.vm.provider "virtualbox" do |vb|
        vb.memory = "2048"
  end

  config.vm.box = "ubuntu/focal64" # 20.04 LTS
  config.vm.hostname = "nomad"
  config.vm.provision "shell", inline: $install_basics, privileged: false
  config.vm.provision "shell", inline: $install_docker, privileged: false
  config.vm.provision "shell", inline: $install_vault, privileged: false
  config.vm.provision "shell", inline: $install_nomad, privileged: false
  config.vm.provision "shell", inline: $install_consul, privileged: false

  # Expose the nomad api and ui to the host
  config.vm.network :private_network, ip: "10.10.10.10", hostname: true
  config.vm.network "forwarded_port", guest: 3306, host: 3306, auto_correct: true, host_ip: "127.0.0.1"
  config.vm.network "forwarded_port", guest: 5432, host: 5432, auto_correct: true, host_ip: "127.0.0.1"
  config.vm.network "forwarded_port", guest: 8080, host: 8080, auto_correct: true, host_ip: "127.0.0.1"
  config.vm.network "forwarded_port", guest: 4646, host: 4646, auto_correct: true, host_ip: "127.0.0.1"
  config.vm.network "forwarded_port", guest: 8200, host: 8200, auto_correct: true, host_ip: "127.0.0.1"
  config.vm.network "forwarded_port", guest: 8500, host: 8500, auto_correct: true, host_ip: "127.0.0.1"
  config.vm.synced_folder ".", "/home/vagrant/nomad"
  
  ## Increase memory for Parallels Desktop
  #config.vm.provider "parallels" do |p, o|
  #  p.memory = "1024"
  #end


  ## Increase memory for VMware
  #["vmware_fusion", "vmware_workstation"].each do |p|
  #  config.vm.provider p do |v|
  #    v.vmx["memsize"] = "1024"
  #  end
  #end
end
