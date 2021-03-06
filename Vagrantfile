# -*- mode: ruby -*-
# vi: set ft=ruby :

##########################################
##                                      ##
## GMdotnet                             ##
## Vagrant Multi Machine Digital Ocean  ##
## Version 0.0.1                        ##
##                                      ##
##########################################

# Import configuration
require 'yaml'

# Load config.yaml file
vagrantconfig = YAML.load_file('config/config.yaml')

Vagrant.configure("2") do |config|
    vagrantconfig['gmdotnet'].each do |machine|
        # switch from machine to host variable
        host = machine['host']
        if host['enable'] == true
            config.vm.define host['vagrantbox_name'] do |vmhost|
                # hostname
                vmhost.vm.hostname = host['hostname']
                # box name
                vmhost.vm.box = host['box']['name']
                # box version
                if host['box']['version'] != nil
                    vmhost.vm.box_version = host['box']['version']
                end
                # check_update
                vmhost.vm.box_check_update = host['box']['check_update']

                # Hostsupdater plugin
                if host['plugin']['hostsupdater']['enable'] == true

                    # Hostsupdater aliases
                    if host['plugin']['hostsupdater']['aliases'] != nil
                        vmhost.hostsupdater.aliases = host['plugin']['hostsupdater']['aliases']
                    end

                    # Hostsupdater permanent role
                    if host['plugin']['hostsupdater']['permanent'] == true
                        vmhost.hostsupdater.remove_on_suspend = false
                    end

                    # Private IP Network
                    vmhost.vm.network "private_network", ip: host['private_ip'], hostsupdater: true
                else
                    # Private IP Network - without hostsupdater plugin
                    vmhost.vm.network "private_network", ip: host['private_ip']
                end
                ## -*- end hostsupdater plugin folders -*-

                # Rsync folders
                if host['rsync'] != nil
                    host['rsync'].each do |rsync|
                      rsyncoptions = []
                      rsync['folder']['options'].each do |options|
                          rsyncoptions.push(options)
                      end
                      vmhost.vm.synced_folder rsync['folder']['host_folder'], rsync['folder']['vagrant_folder'], type: "rsync", rsync__args: rsyncoptions
                    end
                end
                ## -*- end rsync folders -*-

                # Digital Ocean options
                vmhost.vm.provider :digital_ocean do |provider, override|
                    override.ssh.private_key_path = host['digital_ocean']['ssh']['key_path']
                    provider.ssh_key_name = host['digital_ocean']['ssh']['key_name']
                    override.vm.box = host['hostname']
                    override.vm.box_url = "https://github.com/devopsgroup-io/vagrant-digitalocean/raw/master/box/digital_ocean.box"
                    provider.token = host['digital_ocean']['token']
                    provider.image = host['digital_ocean']['image']
                    provider.region = host['digital_ocean']['region']
                    provider.size = host['digital_ocean']['size']
                    provider.ipv6 = host['digital_ocean']['enable_ipv6']
                    provider.private_networking = host['digital_ocean']['enable_private_networking']
                    provider.backups_enabled = host['digital_ocean']['backups_enabled']
                end

                # Shell provision
                if host['provision']['script']['enable'] == true
                    vmhost.vm.provision "shell", path: host['provision']['script']['path']
                end

                # Ansible provision
                if host['provision']['ansible']['enable'] == true
                    vmhost.vm.provision "ansible_local" do |ansible|
                        ansible.verbose = "vv"
                        ansible.playbook =  host['provision']['ansible']['playbook_path']
                    end
                end
            end
            ## -*- end vmhost -*-
        end
        ## -*- end host enable -*-
    end
    ## -*- end machine -*-
end
## -*- end Vagrant -*-
