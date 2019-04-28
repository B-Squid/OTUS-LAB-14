# -*- mode: ruby -*-
# vim: set ft=ruby :

$script_web = <<-SHELL
if !( grep -Fxq "192.168.11.102" /etc/hosts )
then
echo "192.168.11.102 log" >> /etc/hosts
fi
yum -y install wget
if ! [ -e /etc/yum.repos.d/nginx.repo ]
then
echo "[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/mainline/centos/7/x86_64/
gpgcheck=0
enabled=1" >> /etc/yum.repos.d/nginx.repo
fi
yum -y install nginx
systemctl enable nginx
systemctl start nginx

if ! [ -e /etc/yum.repos.d/elasticsearch.repo ]
then
echo "[elasticsearch-6.x]
name=Elasticsearch repository for 6.x packages
baseurl=https://artifacts.elastic.co/packages/6.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1" >> /etc/yum.repos.d/elasticsearch.repo
fi

yum -y install filebeat
if ! [ -e /etc/filebeat/filebeat.yml ]
then
wget -P /etc/filebeat/ https://raw.githubusercontent.com/B-Squid/OTUS-LAB-14/master/config/filebeat.yml
fi
systemctl enable filebeat
systemctl start filebeat

if ! [ -s /etc/audit/rules.d/audit.rules ]
then
echo "-D
-f 1

-w /etc/nginx/nginx.conf -p rwa

-e 1" >> /etc/audit/rules.d/audit.rules
fi
augenrules --load
filebeat modules enable auditd
systemctl restart filebeat.service
filebeat setup -e
SHELL

$script_log = <<-SHELL
yum -y install wget
yum -y install java-1.8.0-openjdk
rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch
sudo rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch
if ! [ -e /etc/yum.repos.d/elasticsearch.repo ]
then
echo "[elasticsearch-6.x]
name=Elasticsearch repository for 6.x packages
baseurl=https://artifacts.elastic.co/packages/6.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1" >> /etc/yum.repos.d/elasticsearch.repo
fi
yum -y install elasticsearch
systemctl enable elasticsearch.service
systemctl start elasticsearch.service

if ! [ -e /etc/yum.repos.d/kibana.yml ]
then
echo "[kibana-6.x]
name=Kibana repository for 6.x packages
baseurl=https://artifacts.elastic.co/packages/6.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md" >> /etc/yum.repos.d/kibana.repo
fi
yum -y install kibana
systemctl enable kibana.service
systemctl start kibana.service
if ! [ -e /etc/kibana/kibana.yml ]
then
wget -P /etc/kibana/ https://raw.githubusercontent.com/B-Squid/OTUS-LAB-14/master/config/kibana.yml 
fi
systemctl restart kibana.service

yum -y install logstash
systemctl enable logstash.service
if ! [ -e /etc/logstash/conf.d/input.conf ]
then
wget -P /etc/logstash/conf.d https://raw.githubusercontent.com/B-Squid/OTUS-LAB-14/master/config/input.conf
fi
if ! [ -e /etc/logstash/conf.d/output.conf ]
then
wget -P /etc/logstash/conf.d https://raw.githubusercontent.com/B-Squid/OTUS-LAB-14/master/config/output.conf
fi
if ! [ -e /etc/logstash/conf.d/filter.conf ]
then
wget -P /etc/logstash/conf.d https://raw.githubusercontent.com/B-Squid/OTUS-LAB-14/master/config/filter.conf
fi
systemctl restart logstash.service
SHELL

Vagrant.configure('2') do |config|
  config.vm.box = "centos/7"

  config.vm.define "log2" do |log|
    log.vm.hostname = "log"
    log.vm.network :public_network
    log.vm.network :private_network, ip: "192.168.11.102"
    config.vm.network "forwarded_port", guest: 5601, host: 56010
    log.vm.provider :kvm do |kvm, override|
      kvm.memory_size = '1536m'
    end
    log.vm.provider :libvirt do |libvirt, override|
      libvirt.memory = 1536
      libvirt.nested = true
      libvirt.cpus = 1
    end
    log.vm.provision "shell", inline: $script_log

  end

  config.vm.define "web2" do |web|
    web.vm.hostname = "web"
    web.vm.network :public_network
    web.vm.network :private_network, ip: "192.168.11.101"
    #config.vm.network "forwarded_port", guest: 80, host: 8080
    web.vm.provider :kvm do |kvm, override|
      kvm.memory_size = '768m'
    end
    web.vm.provider :libvirt do |libvirt, override|
      libvirt.memory = 768
      libvirt.nested = true
      libvirt.cpus = 1
    end
    web.vm.provision "shell", inline: $script_web

  end


  config.vm.provision "shell", inline: <<-SHELL
    mkdir -p ~root/.ssh
    cp ~vagrant/.ssh/auth* ~root/.ssh
  SHELL

end
