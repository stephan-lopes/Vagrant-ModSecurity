#/etc/httpd/conf.d/welcome.con://github.com/SpiderLabs/ModSecurity/releases/download/v3.0.7/modsecurity-v3.0.7.tar.gfz -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

  config.vm.box_check_update = false

  config.vm.box = "centos/7"
  apache_image  = "kevenlopes/mod_security:0.1.0"
  dvwa_image    = "vulnerables/web-dvwa"

  config.vm.hostname = "modsec.local"
  config.vm.network "private_network", type: "dhcp"
  config.vm.network "forwarded_port", guest: 80, host: 8080
  config.vm.network "forwarded_port", guest: 22, host: 2200

  config.vm.provision "file", source: "./my-httpd.conf", destination: "/tmp/httpd.conf"
  config.vm.provision "file", source: "./extra/virtual-host.conf", destination: "/tmp/welcome.conf"

  config.vm.provision "shell", inline: <<-SHELL
    yum update -y
    yum install -y net-tools wget epel-release
    yum install -y yum-utils
    yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
    yum install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin
    systemctl start docker
    systemctl enable docker
    usermod -aG docker vagrant
    docker run --rm -d -p 8080:80 #{dvwa_image}
    yum install -y httpd
    cp /tmp/httpd.conf /etc/httpd/conf/httpd.conf
    cp /tmp/welcome.conf /etc/httpd/conf.d/welcome.conf
    systemctl start httpd
    systemctl enable httpd
    setsebool -P httpd_can_network_connect on
    # sed 's/PasswordAuthentication/& yes #/g' /etc/ssh/sshd_config > /etc/ssh/sshd_config 
  SHELL

  config.vm.provision "ansible" do |ansible|
    ansible.playbook  = "ansible/playbook.yml"
  end

  config.vm.provider :virtualbox do |vb|
    vb.name   = "mod_security"
    vb.gui    = false
    vb.memory = "512"
    vb.cpus   = "1"
  end

end
