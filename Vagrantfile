# -*- mode: ruby -*-
# vi: set ft=ruby :

$vagrant_mem = 4096
$vagrant_cpus = 2
$local_mesos_dir = "/Users/xiaods/Documents/Code/mesos"

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

  config.vm.provider "vmware_fusion" do |v|
    v.memory = $vagrant_mem
    v.cpus = $vagrant_cpus
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
                   libblkid-devel libcurl-devel libevent-devel libnl3-devel    \
                   openssl-devel python-devel redhat-rpm-config                \
                   subversion-devel xfsprogs-devel python-boto
    yum -y install java-1.8.0-openjdk-devel maven

    # Test dependencies
    yum -y install ethtool logrotate nmap-ncat perf

    # Enable additional filesystems
    echo "overlay" > /etc/modules-load.d/overlayfs.conf
    echo "xfs" > /etc/modules-load.d/xfs.conf

    # Enable Docker
    curl -sSL https://get.daocloud.io/docker | sh

    sudo cp -n /lib/systemd/system/docker.service /etc/systemd/system/docker.service
    sudo sed -i "s|ExecStart=/usr/bin/dockerd|ExecStart=/usr/bin/dockerd --storage-driver=overlay --registry-mirror=https://vwrzfgqh.mirror.aliyuncs.com|g" /etc/systemd/system/docker.service
    sudo systemctl daemon-reload
    sudo service docker restart
  SHELL

  config.vm.provision "shell", privileged: false, inline: <<-SHELL
    cd ~
    mkdir build
    cd build
    /mesos/configure --enable-debug --enable-libevent --enable-ssl             \
                     --enable-xfs-disk-isolator --with-network-isolator
  SHELL
end
