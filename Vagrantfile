# -*- mode: ruby -*-
# vi: set ft=ruby :

$vagrant_mem = 8192
$vagrant_cpus = 4
$local_mesos_dir = "/Users/xiaods/Documents/Code/mesos-projects/mesos"

Vagrant.configure(2) do |config|
  config.vm.box = "centos/7"
  config.vm.box_check_update = false

  config.vm.provider "virtualbox" do |v|
    v.customize [
      "modifyvm", :id,
      "--memory", "#{$vagrant_mem}",
      "--cpus", "#{$vagrant_cpus}",
      "--paravirtprovider", "kvm",
      "--natdnshostresolver1", "on"
    ]
  end

  config.vm.network "forwarded_port", guest: 5050, host: 5050
  config.vm.network "forwarded_port", guest: 5051, host: 5051
  config.vm.network "forwarded_port", guest: 8080, host: 8080
  config.vm.network "private_network", ip: "172.28.128.3"

  config.vm.synced_folder ".", "/vagrant", disabled: true
  config.vm.synced_folder $local_mesos_dir, "/opt/mesos", type: "nfs"

  config.bindfs.bind_folder "/opt/mesos", '/mesos'

  config.vm.provision "shell", inline: <<-SHELL
    # 禁用 fastestmirror 插件
    sed -i.backup 's/^enabled=1/enabled=0/' /etc/yum/pluginconf.d/fastestmirror.conf
    # 使用阿里云镜像
    wget -O /etc/yum.repos.d/CentOS-Base-aliyun.repo http://mirrors.aliyun.com/repo/Centos-7.repo
    # EPEL aliyun
    wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
    # yum update
    yum -y update

    # Development tools
    yum -y install automake ccache cmake gcc-c++ git libtool patch the_silver_searcher vim

    # Mesos dependencies
    yum -y install apr-devel apr-util-devel cyrus-sasl-devel cyrus-sasl-md5    \
                   elfutils-libelf-devel libblkid-devel libcurl-devel          \
                   ibevent-devel libnl3-devel openssl-devel python-devel      \
                   redhat-rpm-config subversion-devel xfsprogs-devel libevent-devel

    yum -y install java-1.8.0-openjdk-devel maven

    # Install development tools
    yum groupinstall "Development Tools"

    # Test dependencies
    yum -y install ethtool logrotate nmap-ncat perf

    #ruby source
    gem source -r https://rubygems.org/
    gem source -a http://mirrors.aliyun.com/rubygems/

    # Enable Docker
    curl -sSL https://get.daocloud.io/docker | sh

    sudo cp -n /lib/systemd/system/docker.service /etc/systemd/system/docker.service
    sudo sed -i.backup "s|^ExecStart=/usr/bin/dockerd|ExecStart=/usr/bin/dockerd --storage-driver=overlay --registry-mirror=https://vwrzfgqh.mirror.aliyuncs.com|" /etc/systemd/system/docker.service
    sudo systemctl daemon-reload
    sudo service docker restart
    sudo usermod -aG docker vagrant
  SHELL

  # config.vm.provision "shell", privileged: false, inline: <<-SHELL
  #   cd ~
  #   mkdir build
  #   cd build
  #   /mesos/configure --disable-python --disable-java --enable-debug
  # SHELL
end
