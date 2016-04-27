# -*- mode: ruby -*-
# vi: set ft=ruby :

$vagrant_mem = 8192
$vagrant_cpus = 8
$local_mesos_dir = "../mesos"

Vagrant.configure(2) do |config|
  config.vm.box = "bento/fedora-23"

  config.vm.provider "virtualbox" do |v|
    v.customize [
      "modifyvm", :id,
      "--memory", "#{$vagrant_mem}",
      "--cpus", "#{$vagrant_cpus}",
      "--paravirtprovider", "kvm"
    ]
  end

  config.vm.provider "vmware_fusion" do |v|
    v.memory = $vagrant_mem
    v.cpus = $vagrant_cpus
  end

  config.vm.network "forwarded_port", guest: 5050, host: 5050
  config.vm.network "private_network", type: "dhcp"

  config.vm.synced_folder ".", "/vagrant", disabled: true
  config.vm.synced_folder $local_mesos_dir, "/mesos", type: "nfs"

  config.vm.provision "shell", inline: <<-SHELL
    # Development tools
    dnf -y install automake ccache cmake gcc-c++ git libtool patch

    # Mesos dependencies
    dnf -y install apr-devel apr-util-devel cyrus-sasl-devel cyrus-sasl-md5    \
                   libblkid-devel libcurl-devel libevent-devel libnl3-devel    \
                   openssl-devel python-devel redhat-rpm-config                \
                   subversion-devel xfsprogs-devel
    dnf -y install java-1.8.0-openjdk-devel maven

    # Test dependencies
    dnf -y install ethtool logrotate nmap-ncat perf

    # Enable additional filesystems
    echo "overlay" > /etc/modules-load.d/overlayfs.conf
    echo "xfs" > /etc/modules-load.d/xfs.conf

    # Enable Docker
    curl -fsSL https://get.docker.com/ | sh

    mkdir -p /etc/systemd/system/docker.service.d
    cat > /etc/systemd/system/docker.service.d/overrides.conf <<EOF
[Service]
ExecStart=
ExecStart=/usr/bin/docker daemon -H fd:// --storage-driver=overlay
EOF

    systemctl enable docker.service
  SHELL

  config.vm.provision "shell", privileged: false, inline: <<-SHELL
    cd ~
    mkdir build
    cd build
    /mesos/configure --enable-debug --enable-libevent --enable-ssl             \
                     --enable-xfs-disk-isolator --with-network-isolator
  SHELL
end
