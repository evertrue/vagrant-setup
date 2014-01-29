# -*- mode: ruby -*-
# vi: set ft=ruby :

if File.exist?('./vagrant-settings.rb')
  require './vagrant-settings.rb'
end

CHEF_TYPE ||= 'chef-solo'

Vagrant.require_plugin "vagrant-chef-zero" if CHEF_TYPE == 'chef-zero'

Vagrant.configure("2") do |config|
  cookbook_name = open('./metadata.rb').grep(/^name/).first.split.last.gsub(/'/,'')

  config.vm.hostname = "#{cookbook_name.gsub(/_/,'-')}-berkshelf"

  if CHEF_TYPE == 'aws'
    config.vm.box = "dummy"
    config.vm.box_url = "https://github.com/mitchellh/vagrant-aws/raw/master/dummy.box"

    config.vm.provider :aws do |aws, override|
      aws.access_key_id = "AKIAIBHMPZ2OF7KX7OVA"
      aws.secret_access_key = File.read("#{ENV['HOME']}/.aws_secret_access_key").chomp
      aws.keypair_name = "aws_dev"
      aws.private_ip_address = "10.0.4.165"

      aws.ami = "ami-cf5e2ba6"

      aws.security_groups = "sg-79ea5a16"
      aws.subnet_id = "subnet-2d2a6c42"
      aws.tags = {
        "Name" => "stage-Vagrant-#{config.vm.hostname}",
        "Role" => "Vagrant Testing",
        "Env" => "stage"
      }

      override.ssh.username = "ubuntu"
      override.ssh.private_key_path = "#{ENV['HOME']}/.ssh/aws_dev.pem"
      override.ssh.host = "10.0.4.165"
    end
  else
    config.vm.box = 'precise64'
    config.vm.box_url = 'http://cloud-images.ubuntu.com/precise/current/precise-server-cloudimg-vagrant-amd64-disk1.box'
  end

  config.vm.network :private_network, ip: '33.33.33.10'
  config.berkshelf.enabled = true
  config.omnibus.chef_version = :latest

  if ENV['CHEF_REPO']
    chef_repo = ENV['CHEF_REPO']
  else
    raise 'CHEF_REPO is not defined'
  end

  case CHEF_TYPE
  when 'chef-zero'
    config.chef_zero.chef_repo_path = '.chef-zero/'

    config.vm.provision :chef_client do |chef|
      chef.json = CHEF_JSON ||= {}
      chef.environment = 'stage'
      # chef.log_level = :debug
      chef.encrypted_data_bag_secret_key_path = "#{ENV['HOME']}/.chef/encrypted_data_bag_secret"
      chef.run_list = [
          "recipe[#{cookbook_name}::default]"
      ]
    end
  else
    config.vm.provision :chef_solo do |chef|
      chef.json = CHEF_JSON ||= {}
      chef.data_bags_path = "#{chef_repo}/data_bags"
      chef.environment = 'stage'
      # chef.log_level = :debug
      chef.run_list = [
          "recipe[#{cookbook_name}::default]"
      ]
    end
  end
end
