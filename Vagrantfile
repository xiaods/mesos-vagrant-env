# -*- mode: ruby -*-
# vi: set ft=ruby :

$vagrant_mem = 8192
$vagrant_cpus = 8
$local_mesos_dir = "/Users/xiaods/Documents/Code/mesos-projects/mesos"

Vagrant.configure(2) do |config|
  config.vm.box = "centos/7"

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
  config.vm.network "private_network", ip: "172.28.128.3"

  config.vm.synced_folder ".", "/vagrant", disabled: true
  config.vm.synced_folder $local_mesos_dir, "/opt/mesos", type: "nfs"

  config.bindfs.bind_folder "/opt/mesos", '/mesos'

  config.vm.provision "shell", inline: <<-SHELL
    # Development tools
    yum -y install automake ccache cmake gcc-c++ git libtool patch the_silver_searcher

    # Mesos dependencies
    yum -y install apr-devel apr-util-devel cyrus-sasl-devel cyrus-sasl-md5    \
                   elfutils-libelf-devel libblkid-devel libcurl-devel          \
                   ibevent-devel libnl3-devel openssl-devel python-devel      \
                   redhat-rpm-config subversion-devel xfsprogs-devel libevent-devel

    yum -y install java-1.8.0-openjdk-devel maven

    # Test dependencies
    yum -y install ethtool logrotate nmap-ncat perf

    # Enable Docker
    yum -y install docker-engine

    sudo cp -n /lib/systemd/system/docker.service /etc/systemd/system/docker.service
    sudo sed -i "s|ExecStart=/usr/bin/dockerd|ExecStart=/usr/bin/dockerd --storage-driver=overlay --registry-mirror=https://vwrzfgqh.mirror.aliyuncs.com|g" /etc/systemd/system/docker.service
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
