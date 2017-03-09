# Our master node will also run a copy of Ansible
$install_ansible = <<SCRIPT
sudo apt-get update && \
    sudo apt-get install -y python python-pip build-essential libssl-dev \
        libffi-dev python-dev && \
    sudo pip install ansible==2.2.1.0
SCRIPT

# In order to use Ubuntu Xenial, we need to install Python on each of our
# machines. This is the first part of provisioning.
$install_python = <<SCRIPT
sudo apt-get update && sudo apt-get install -y python
SCRIPT

# Using Vagrant version 2. No ifs or buts.
Vagrant.configure("2") do |config|
    # Number of nodes. On an i5 with 8Gb RAM I use 3.
    # This MUST be > 1
    N = 3

    # Playbook to use
    PLAYBOOK = "provision.yml"

    # Primary node config
    PRIMARY = {
        :box => "ubuntu/xenial64",
        :memory => 512,
        :shell_provision => $install_ansible,
        :hostname => "master",
    }

    # Secondary node config
    SECONDARY = {
        :box => "ubuntu/xenial64",
        :memory => 512,
        :shell_provision => $install_python,
        :hostname => "node",
    }

    # Iterate for nodes
    (1..N).each do |node_id|
        nid = (node_id - 1)

        # Set our primary node
        if nid == 0
            config.vm.define "node#{nid}", primary: true do |n|
                n.vm.box = PRIMARY[:box]
                n.vm.boot_timeout = 600
                n.vm.hostname = "#{PRIMARY[:hostname]}#{nid}"
                n.vm.network "private_network", ip: "10.8.10.#{10 + nid}"

                n.vm.provider "virtualbox" do |vb|
                    vb.name = "#{PRIMARY[:hostname]}#{nid}"
                    vb.memory = PRIMARY[:memory]
                end

                # Install Python on this node.
                n.vm.provision "shell", inline: PRIMARY[:shell_provision]
            end
        else
            # Define our secondary nodes
            config.vm.define "node#{nid}" do |n|
                n.vm.box = SECONDARY[:box]
                n.vm.boot_timeout = 600
                n.vm.hostname = "#{SECONDARY[:hostname]}#{nid}"
                n.vm.network "private_network", ip: "10.8.10.#{10 + nid}"

                n.vm.provider "virtualbox" do |vb|
                    vb.name = "#{SECONDARY[:hostname]}#{nid}"
                    vb.memory = SECONDARY[:memory]
                end

                # Install Python on this node.
                n.vm.provision "shell", inline: SECONDARY[:shell_provision]

                # If this is our last node, lets provision all.
                if node_id == N
                    n.vm.provision "ansible" do |a|
                        a.limit = "all"
                        a.verbose = "v"
                        a.playbook = PLAYBOOK
                    end
                end
            end
        end
    end
end
