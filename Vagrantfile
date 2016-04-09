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
    dnf -y install automake cmake patch gcc-c++ ccache git

    # Mesos dependencies
    dnf -y install python-boto python-devel libcurl-devel openssl-devel        \
                   cyrus-sasl-devel cyrus-sasl-md5 apr-devel subversion-devel  \
                   apr-util-devel libevent-devel libnl3-devel xfsprogs-devel   \
                   libblkid-devel redhat-rpm-config
    dnf -y install java-1.8.0-openjdk-devel maven
    
    # Test dependencies
    dnf -y install perf nmap-ncat ethtool logrotate

    # Enable OverlayFS
    echo "overlay" > /etc/modules-load.d/overlayfs.conf

    # Enable XFS
    echo "xfs" > /etc/modules-load.d/xfs.conf

    # Docker
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
