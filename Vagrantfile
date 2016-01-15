# -*- mode: ruby -*-
# vi: set ft=ruby :

# http://docs.vagrantup.com/v2/multi-machine/

Vagrant.configure('2') do |config|
  config.vm.box = 'ubuntu-14.04'
  config.vm.box_url = "http://opscode-vm-bento.s3.amazonaws.com/vagrant/virtualbox/opscode_#{config.vm.box}_chef-provisionerless.box"
  config.omnibus.chef_version = 'latest'
  config.berkshelf.enabled = true
  config.vm.network 'private_network', type: 'dhcp'

  config.ssh.insert_key = false

  # create the test user data bag if not existing
  unless File.exist?('data_bags/users/test.json')
    require 'fileutils'
    FileUtils.mkpath('data_bags/users')
    IO.write('data_bags/users/test.json', '{ "id": "test" }')
  end

  config.vm.define 'server', primary: true do |server|
    server.vm.hostname = 'openvpn-server'
    server.vm.network 'private_network', ip: '172.19.18.8'

    server.vm.provision :chef_zero do |chef|
      # chef.log_level = :debug
      chef.cookbooks_path = File.expand_path('~/.berkshelf/cookbooks')
      chef.roles_path = 'roles'
      chef.add_role('openvpn')
      chef.data_bags_path = 'data_bags'
      chef.json = {
        "openvpn" => {
          "config" => {
            "local" => "172.19.18.8"
          },
          "gateway" => "172.19.18.8"
        }
      }
      chef.run_list = [
        'recipe[openvpn::server]',
        'recipe[openvpn::users]'
      ]
    end
  end

  config.vm.define 'client' do |client|
    client.vm.hostname = 'openvpn-client'

    # installs the vagrant insecure private key
    # for connection to the openvpn server
    config.vm.provision 'shell', path: 'vagrant-scripts/install_insecure_key.rb'

    # copy and make test ovpn profile tarball downloadable by vagrant user
    config.vm.provision 'shell', path: 'vagrant-scripts/copy_ovpn_profile_to_vagrant_home.sh'

    # download and install the ovpn profile from the server
    config.vm.provision 'shell', path: 'vagrant-scripts/download_ovpn_profile.sh'

    client.vm.provision :chef_zero do |chef|
      chef.cookbooks_path = File.expand_path('~/.berkshelf/cookbooks')
      # chef.log_level = :debug
      chef.json = {
        'openvpn' => {
          'gateway' => '172.19.18.8',
        }
      }
      chef.run_list = [
        'recipe[openvpn::install]',
        'recipe[openvpn::service]'
      ]
    end
  end
end
