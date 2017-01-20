# -*- mode: ruby -*-
# vi: set ft=ruby :

require_relative './files/key_authorization.rb'

VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = "bento/centos-7.2"

  config.vm.provider "virtualbox" do |vb|
    vb.gui = true
    vb.memory = "2048"
  end

  authorize_key_for_root config, '~/.ssh/id_rsa.pub'
  # config.vm.provision 'shell', inline: 'yum update -y'
  config.vm.provision 'shell', inline: 'yum install -y vim iftop innotop'
  config.vm.provision 'shell', inline: 'yum install -y mlocate && updatedb'
  config.vm.provision 'shell', inline: 'sudo nmcli connection reload',
    run: 'always'
  config.vm.provision 'shell', inline: 'sudo systemctl restart network.service',
    run: 'always'

  {
    'rivendell-01'   => '192.168.1.11',
    'rivendell-02'   => '192.168.1.12',
  }.each do |short_name, ip|
    config.vm.define short_name do |host|
      host.vm.network "public_network", { bridge: 'wlp3s0', ip: ip }
      host.vm.hostname = "#{short_name}.dev"
      host.vm.provider "virtualbox" do |vb|
      end
    end
  end
end
