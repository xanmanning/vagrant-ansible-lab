# In order to use Ubuntu Xenial, we need to install Python on each of our
# machines. This is the first part of provisioning.
$install_python = <<SCRIPT
sudo apt-get update && sudo apt-get install -y python
SCRIPT

# Using Vagrant version 2. No ifs or buts.
Vagrant.configure("2") do |config|
    # Number of nodes. On an i5 with 8Gb RAM I use 3.
    N = 3

    # Iterate for nodes
    (1..N).each do |node_id|
        nid = (node_id - 1)

        # Define our node
        config.vm.define "node#{nid}" do |n|
            n.vm.box = "ubuntu/xenial64"
            n.vm.boot_timeout = 600
            n.vm.hostname = "node#{nid}"
            n.vm.network "private_network", ip: "10.8.10.#{10 + nid}"

            n.vm.provider "virtualbox" do |vb|
                vb.name = "node#{nid}"
                vb.memory = 512
            end

            # Install Python on this node.
            n.vm.provision "shell", inline: $install_python

            # If this is our last node, lets provision all.
            if node_id == N
                n.vm.provision "ansible" do |a|
                    a.limit = "all"
                    a.verbose = "v"
                    a.playbook = "provision.yml"
                end
            end
        end
    end
end
