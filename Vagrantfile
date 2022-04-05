# -*- mode: ruby -*-
# # vi: set ft=ruby :

# Specify minimum Vagrant version and Vagrant API version
Vagrant.require_version '>= 2.2.4'
VAGRANTFILE_API_VERSION = '2'

# Require YAML module
require 'yaml'

file_root = File.dirname(File.expand_path(__FILE__))

# Global vars
cluster_name = 'Cluster K8S demo'
provider = 'virtualbox'
vm_memory = '2048'
vm_cpus = '2'
vm_disk = 20,
box_name = 'generic/ubuntu2004'
box_version =  '3.0.28'

# Read configuration from ansible inventory file
cluster = YAML.load_file("cluster.yml")
puts "==> Cluster config: \n#{cluster}"
vars = cluster['all']['vars']
#puts "==> all: \n#{vars}"
user = vars['ansible_user']
password = vars['ansible_ssh_pass']
puts "==> user/password: #{user}/#{password}"

hosts = cluster['all']['hosts']
puts "==> hosts: \n#{hosts}"

#  Fully documented Vagrantfile available
#  https://www.vagrantup.com/docs/vagrantfile/

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

    # Required these plugins
    config.vagrant.plugins = [
        "vagrant-hostmanager" => {"version" => "1.8.9"},
        "vagrant-reload" => {"version" => "0.0.1"},
        # "vagrant-disksize"
    ]

    # If true, Vagrant will automatically insert a keypair to use for SSH, 
    # replacing Vagrant's default insecure key inside the machine if detected. 
    # By default, this is true.
    # This only has an effect if you do not already use private keys for authentication 
    # or if you are relying on the default insecure key. If you do not have to care about security 
    # in your project and want to keep using the default insecure key, set this to false.
    #config.ssh.insert_key = false

    # If false, this setting will not include the compression setting 
    # when ssh'ing into a machine. If this is not set, it will default to true 
    # and Compression=yes will be enabled with ssh.
    # config.ssh.compression=false

    # If true, this setting SSH will send keep-alive packets every 5 seconds by default to keep connections alive.
    #config.ssh.keep_alive = true

    # The command to use when executing a command with sudo. 
    # This defaults to sudo -E -H %c. The %c will be replaced by the command that is being executed.
    # config.ssh.sudo_command = "sudo -E -H %c"

    # Hostmanager Config
    # Need to install the plugin vagrant-hostmanger
    config.hostmanager.enabled = true
    config.hostmanager.manage_host = false
    config.hostmanager.manage_guest = true
    config.hostmanager.ignore_private_ip = false
    config.hostmanager.include_offline = true

    # Server host
    hosts.each do |host|

        puts "==> host: #{host}"
        # VM name
        vm_name = host[0]
        puts "==> vm_name: #{vm_name}"

        # VM IP
        vm_ip = host[1]['ansible_host']
        puts "==> vm_ip: #{vm_ip}"

        # SSH port forwarded
        #vm_ssh_port = host[1]['ansible_ssh_port']
        #puts "==> vm_ssh_port: #{vm_ssh_port}"

        # Create VM
        config.vm.define vm_name do |node|

            # https://www.vagrantup.com/docs/vagrantfile/machine_settings.html
            node.vm.box = box_name
            node.vm.box_version = box_version

            # The URL that the configured box can be found at. 
            # If config.vm.box is a shorthand to a box in HashiCorp's Vagrant Cloud 
            # then this value does not need to be specified. Otherwise, it should point 
            # to the proper place where the box can be found if it is not installed. 
            # This can also be an array of multiple URLs. The URLs will be tried in order.
        
            # Note that any client certificates, insecure download settings, 
            # and so on will apply to all URLs in this list. 
            # The URLs can also be local files by using the file:// scheme. For example: "file:///tmp/test.box".
            #config.vm.box_url = "http://repo.release.xxxxx.yyy/nexus/content/repositories/vagrant/com/xxxxx/vagrant/rhel73/1.0.0/rhel73-1.0.0.box"
        
            # If true, then SSL certificates from the server will not be verified. 
            # By default, if the URL is an HTTPS URL, then SSL certs will be verified.
            node.vm.box_download_insecure = true

            node.vm.hostname = vm_name
        
            # The time in seconds that Vagrant will wait for the machine to boot and be accessible. 
            # By default this is 300 seconds.
            node.vm.boot_timeout = 600

            # Enable ssh password connection
            node.ssh.username = user
            node.ssh.password = password
            node.ssh.insert_key = false
        
            # Networks
            # https://www.vagrantup.com/docs/networking/private_network.html
            #config.vm.network :private_network, ip: vm_ip, auto_config: false
            node.vm.network :public_network, ip: vm_ip, bridge: "en0: Wi-Fi (AirPort)"
            # Must specified `id: "ssh"` in order to override the default forwarded SSH port.
            #node.vm.network :forwarded_port, guest: 22, host: vm_ssh_port, id: "ssh"

            # https://www.vagrantup.com/docs/virtualbox/
            node.vm.provider "virtualbox" do |vb|
                vb.name = vm_name
                vb.cpus = vm_cpus
                vb.memory = vm_memory
                #vb.linked_clone = true
                # https://docs.oracle.com/en/virtualization/virtualbox/6.0/user/vboxmanage-modifyvm.html
                vb.customize ["modifyvm", :id, "--groups", "/#{cluster_name}"]
                #vb.customize ["modifyvm", :id, "--memory", node['mem']]
                #vb.customize ["modifyvm", :id, "--cpus", node['cpu']]
                #vb.customize ["modifyvm", :id, "--macaddress1", "auto"]
                #vb.customize ["modifyvm", :id, "--vram", "7"]
                #if vm_disk
                #    disk_size = vm_disk.to_i
                #    file_to_disk = File.join(file_root, "#{hostname}.vdi")
                #    unless File.exist?(file_to_disk)
                #        vb.customize ['createhd', '--filename', file_to_disk, '--format', 'VDI', '--size', disk_size * 1024]
                #    end
                #end
            end

            # Modify sshd_config to connect with ssh
            config.vm.provision "shell", inline: <<-SHELL
                sudo sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config    
                sudo systemctl restart sshd.service
            SHELL
        end
    end 
end 