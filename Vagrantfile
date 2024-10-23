# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANTFILE_API_VERSION = '2'

# Requirements
require "yaml"

# Copy example configuration file
vagrant_dir = File.dirname(File.expand_path(__FILE__))
if not File.exist?("#{vagrant_dir}/settings.yml")
  FileUtils.cp("#{vagrant_dir}/settings.yml.example", "#{vagrant_dir}/settings.yml")
end

# Read configuration file
settings = YAML.load_file "#{vagrant_dir}/settings.yml"

# Get versions
CALICO_VERSION     = settings["versions"]["calico"]
DASHBOARD_VERSION  = settings["versions"]["dashboard"]
KUBERNETES_VERSION = settings["versions"]["kubernetes"]
VAGRANT_BOX        = ENV['VAGRANT_BOX'] || settings["versions"]["box"]

# Get network settings
BRIDGE_IFACE    = ENV['BRIDGE_IFACE']    || settings["network"]["bridge"]
K8S_MAC_ADDRESS = ENV['K8S_MAC_ADDRESS'] || settings["network"]["mac"]
K8S_PODS_CIDR   = ENV['K8S_PODS_CIDR']   || settings["network"]["pods_cidr"]

# Get kubernetes settings
K8S_MASTER_CPUS   = ENV['K8S_MASTER_CPUS']   || settings["k8s"]["master"]["cpus"]
K8S_MASTER_MEMORY = ENV['K8S_MASTER_MEMORY'] || settings["k8s"]["master"]["memory"]
K8S_NODES_COUNT   = ENV['K8S_NODES_COUNT']   || settings["k8s"]["workers"]["count"]
K8S_NODE_CPUS     = ENV['K8S_NODE_CPUS']     || settings["k8s"]["workers"]["cpus"]
K8S_NODE_MEMORY   = ENV['K8S_NODE_MEMORY']   || settings["k8s"]["workers"]["memory"]

# Get other environment variables
VAGRANT_LOG         = ENV['VAGRANT_LOG']         || 'error'
LIBVIRT_DEFAULT_URI = ENV['LIBVIRT_DEFAULT_URI'] || 'qemu:///system'
PROJECT_NAME        = File.basename(vagrant_dir) || 'vagrant-k8s'

# Generate network for VMs
K8S_NETBASE = '192.168.'
K8S_NETID   = (PROJECT_NAME.sum % 100) + 100

# Ansible configuration
ANSIBLE_GROUPS = { 'nodes' => ['node[01:' + K8S_NODES_COUNT.to_s.rjust(2, '0') + ']'] }
ANSIBLE_HOSTS  = {}
ANSIBLE_VARS   = {
  'vagrant_mac'        => K8S_MAC_ADDRESS,
  'pods_cidr'          => K8S_PODS_CIDR,
  'calico_version'     => CALICO_VERSION,
  'dashboard_version'  => DASHBOARD_VERSION,
  'kubernetes_version' => KUBERNETES_VERSION
}

# Start VMs
Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  config.vm.box              = VAGRANT_BOX
  config.vm.box_check_update = false

  config.vm.synced_folder '.', '/vagrant', disabled: true

  # Configure libvirt provider
  config.vm.provider :libvirt do |virt|
    virt.connect_via_ssh            = LIBVIRT_DEFAULT_URI.include?('+ssh')     ? true : false
    virt.qemu_use_session           = LIBVIRT_DEFAULT_URI.include?('/session') ? true : false
    virt.system_uri                 = LIBVIRT_DEFAULT_URI.gsub('/session', '/system')
    virt.uri                        = LIBVIRT_DEFAULT_URI
    virt.management_network_address = K8S_NETBASE + (K8S_NETID - 1).to_s + '.0/24'
    virt.management_network_name    = PROJECT_NAME + '-mgmt'
  end

  # Set up worker nodes
  (1..Integer(K8S_NODES_COUNT)).each do |i|
    config.vm.define 'node' + i.to_s.rjust(2, '0') do |node|

      NODE_HOSTNAME    = 'node' + i.to_s.rjust(2, '0')
      NODE_IP          = K8S_NETBASE + K8S_NETID.to_s + '.' + (100 + i).to_s
      NODE_MAC         = (K8S_MAC_ADDRESS.to_i(16) + i).to_s(16)
      node.vm.hostname = NODE_HOSTNAME

      # Configure VM for libvirt provider
      node.vm.provider :libvirt do |virt, override|
        virt.cpus   = K8S_NODE_CPUS
        virt.memory = K8S_NODE_MEMORY
        # Add public network
        if BRIDGE_IFACE and not BRIDGE_IFACE.empty?
          override.vm.network :public_network,
            :dev  => BRIDGE_IFACE,
            :mac  => NODE_MAC,
            :mode => 'bridge',
            :type => 'bridge'
        else
          ANSIBLE_HOSTS.store(NODE_HOSTNAME, { 'ansible_host' => NODE_IP })
          override.vm.network :private_network,
            :ip                    => NODE_IP,
            :libvirt__forward_mode => 'none'
        end
      end

      # Configure VM for VirtualBox provider
      node.vm.provider :virtualbox do |vbox, override|
        vbox.cpus   = K8S_NODE_CPUS
        vbox.memory = K8S_NODE_MEMORY
        # Add public network
        if BRIDGE_IFACE and not BRIDGE_IFACE.empty?
          override.vm.network :public_network,
            :bridge => BRIDGE_IFACE,
            :mac    => NODE_MAC
        else
          ANSIBLE_HOSTS.store(NODE_HOSTNAME, { 'ansible_host' => NODE_IP })
          override.vm.network :private_network,
            :ip => NODE_IP
        end
      end

    end

  end

  # Set up control plane as primary VM
  config.vm.define 'master', primary: true do |node|

    NODE_HOSTNAME    = 'master'
    NODE_IP          = K8S_NETBASE + K8S_NETID.to_s + '.10'
    node.vm.hostname = NODE_HOSTNAME

    # Configure VM for libvirt provider
    node.vm.provider :libvirt do |virt, override|
      virt.cpus   = K8S_MASTER_CPUS
      virt.memory = K8S_MASTER_MEMORY
      # Add public network
      if BRIDGE_IFACE and not BRIDGE_IFACE.empty?
        override.vm.network :public_network,
          :dev  => BRIDGE_IFACE,
          :mac  => K8S_MAC_ADDRESS,
          :mode => 'bridge',
          :type => 'bridge'
      else
        ANSIBLE_HOSTS.store(NODE_HOSTNAME, { 'ansible_host' => NODE_IP })
        override.vm.network :private_network,
          :ip                    => NODE_IP,
          :libvirt__forward_mode => 'none'
      end
    end

    # Configure VM for VirtualBox provider
    node.vm.provider :virtualbox do |vbox, override|
      vbox.cpus   = K8S_MASTER_CPUS
      vbox.memory = K8S_MASTER_MEMORY
      # Add public network
      if BRIDGE_IFACE and not BRIDGE_IFACE.empty?
        override.vm.network :public_network,
          :bridge => BRIDGE_IFACE,
          :mac    => K8S_MAC_ADDRESS
      else
        ANSIBLE_HOSTS.store(NODE_HOSTNAME, { 'ansible_host' => NODE_IP })
        override.vm.network :private_network,
          :ip => NODE_IP
      end
    end

    # Sync folders
    node.vm.synced_folder 'ansible',  '/ansible', type: 'rsync'
    node.vm.synced_folder '.vagrant', '/vagrant', type: 'rsync'

    # Change permissions of SSH keys
    node.vm.provision :shell, inline: 'chmod 600 /vagrant/machines/*/*/private_key'

    # Provision VMs
    node.vm.provision :ansible_local do |ansible|
      ansible.extra_vars        = ANSIBLE_VARS
      ansible.galaxy_role_file  = 'requirements.yml'
      ansible.groups            = ANSIBLE_GROUPS
      ansible.host_vars         = ANSIBLE_HOSTS
      ansible.limit             = 'all'
      ansible.playbook          = 'playbook.yml'
      ansible.provisioning_path = '/ansible'
    end

  end

end
