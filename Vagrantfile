# -*- mode: ruby -*-
# vi: set ft=ruby :

num_of_nodes = 1

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  config.vm.box = "centos-6.5-x86_64"
  config.vm.box_url = "packer/build/#{config.vm.box}.box"

  # config.vm.network :forwarded_port, guest: 80, host: 8080
  config.vm.network :public_network

  # config.vm.synced_folder "../data", "/vagrant_data"

  config.vm.provider :virtualbox do |vb|
    vb.gui = false
 
    vb.customize ["modifyvm", :id, "--memory", "512"]
    vb.customize ["modifyvm", :id, "--cpus", "2"]   
  end
  
  config.vm.define :rabbitmq do |n|

    n.vm.hostname = "rabbitmq"

    n.vm.network :private_network, ip: "192.168.0.2"

    n.vm.provision :chef_solo do |chef|
      chef.cookbooks_path = "cookbooks"
      chef.add_recipe "rabbitmq::default"
      chef.add_recipe "rabbitmq::plugin_management"
      chef.add_recipe "rabbitmq::virtualhost_management"
      chef.add_recipe "rabbitmq::user_management"
      chef.add_recipe "rabbitmq::mgmt_console"
 
      chef.json = {
        :rabbitmq => {
          :version => '3.2.4',
          :virtualhosts => ['/mcollective'],
          :enabled_users => [
            {
              :name => 'admin',
              :password => 'changeme',
              :tag => 'administrator',
              :rights => [
                {
                  :vhost => "/mcollective",
                  :conf => ".*",
                  :write => ".*",
                  :read => ".*"
                }
              ]
            },{
              :name => 'mcollective',
              :password => 'changeme',
              :rights => [
                {
                  :vhost => "/mcollective",
                  :conf => ".*",
                  :write => ".*",
                  :read => ".*"
                }
              ]
            }
          ],
          :ssl => true,
          :ssl_port => '5671',
          :ssl_cacert => '/vagrant/certs/rabbitmq/cacert.pem',
          :ssl_cert => '/vagrant/certs/rabbitmq/server/cert.pem',
          :ssl_key => '/vagrant/certs/rabbitmq/server/key.pem',
          :ssl_verify => 'verify_peer',
          :stomp => true
        }
      }

    end

    n.vm.provision "shell",
      inline: "wget -O /usr/local/bin/rabbitmqadmin http://127.0.0.1:15672/cli/rabbitmqadmin && chmod +x /usr/local/bin/rabbitmqadmin"

    n.vm.provision "shell",
      inline: '/usr/local/bin/rabbitmqadmin declare exchange --user=admin --password=changeme --vhost="/mcollective" name=mcollective_broadcast type=topic'

    n.vm.provision "shell",
      inline: '/usr/local/bin/rabbitmqadmin declare exchange --user=admin --password=changeme --vhost="/mcollective" name=mcollective_directed type=direct'

  end

  config.vm.define :'mco-client' do |n|

    n.vm.hostname = 'mco-client'

    n.vm.network :private_network, ip: "192.168.0.3"

    n.vm.provision :chef_solo do |chef|
      chef.cookbooks_path = "cookbooks"
      chef.add_recipe "mcollective::default"
 
      chef.json = {
        :mcollective => {
          :connector => "rabbitmq",
          :rabbitmq => {
            :vhost => "/mcollective"
          },
          :stomp => {
            :hostname => "192.168.0.2",
            :port => 61613,
            :username => "mcollective",
            :password => "changeme",
            :ssl => {
              :enabled => true,
              :port => "61614",
              :ca => "/vagrant/certs/rabbitmq/cacert.pem",
              :cert => "/vagrant/certs/rabbitmq/client/cert.pem",
              :key => "/vagrant/certs/rabbitmq/client/key.pem",
              :fallback => false
            }
          }
        }
      }

    end

  end

  num_of_nodes.times do |c|

    config.vm.define :"mco-node-#{c}" do |n|

      n.vm.hostname = "mco-node-#{c}"

      n.vm.network :private_network, ip: "192.168.0.#{c+4}"

      n.vm.provision :chef_solo do |chef|
        chef.cookbooks_path = "cookbooks"
        chef.add_recipe "mcollective::server"
 
        chef.json = {
          :mcollective => {
            :connector => "rabbitmq",
            :rabbitmq => {
              :vhost => "/mcollective"
            },
            :stomp => {
              :hostname => "192.168.0.2",
              :port => "61613",
              :username => "mcollective",
              :password => "changeme",
              :ssl => {
                :enabled => true,
                :port => "61614",
                :ca => "/vagrant/certs/rabbitmq/cacert.pem",
                :cert => "/vagrant/certs/rabbitmq/client/cert.pem",
                :key => "/vagrant/certs/rabbitmq/client/key.pem",
                :fallback => false
              }
            }
          }
        }

      end

    end

  end

end

