Vagrant.configure("2") do |config|
    config.vm.box = "debian/bookworm64"

    config.vm.provider "virtualbox" do |vb|
        vb.memory = 1024
        vb.cpus = 1
    end

    config.vm.define "nginx" do |nginx|
        nginx.vm.hostname = "nginx"
        nginx.vm.network "private_network", ip: "192.168.56.18", adapter: 2, type: "static", virtualbox__intnet: false
        nginx.vm.provider "virtualbox" do |vb|
            vb.memory = 2048
            vb.cpus = 2
        end
    end

    config.vm.define "haproxy" do |haproxy|
        haproxy.vm.hostname = "haproxy"
        haproxy.vm.network "private_network", ip: "192.168.56.17", adapter: 2, type: "static", virtualbox__intnet: false
        haproxy.vm.provider "virtualbox" do |vb|
            vb.memory = 2048
            vb.cpus = 2
        end
    end

    postgresql_slaves = [
        { name: "postgresql_slave_1", hostname: "postgresql.slave1", ip: "192.168.56.11" },
        { name: "postgresql_slave_2", hostname: "postgresql.slave2", ip: "192.168.56.12" },
        { name: "postgresql_slave_3", hostname: "postgresql.slave3", ip: "192.168.56.13" }
    ]

    laravel_nodes = [
        { name: "laravel_node_1", hostname: "laravel.node1", ip: "192.168.56.14" },
        { name: "laravel_node_2", hostname: "laravel.node2", ip: "192.168.56.15" }
    ]

    config.vm.define "postgresql_master" do |master|
        master.vm.hostname = "postgresql-master"
        master.vm.network "private_network", ip: "192.168.56.10", adapter: 2, type: "static", virtualbox__intnet: false
        master.vm.provider "virtualbox" do |vb|
            vb.memory = 2048
            vb.cpus = 2
        end
    end

    postgresql_slaves.each do |node|
        config.vm.define node[:name] do |slave|
            slave.vm.hostname = node[:hostname]
            slave.vm.network "private_network", ip: node[:ip], adapter: 2, type: "static", virtualbox__intnet: false
            slave.vm.provider "virtualbox" do |vb|  
                vb.memory = 2048
                vb.cpus = 2
            end
        end
    end

    laravel_nodes.each do |node|
        config.vm.define node[:name] do |slave|
            slave.vm.hostname = node[:hostname]
            slave.vm.network "private_network", ip: node[:ip], adapter: 2, type: "static", virtualbox__intnet: false
            slave.vm.provider "virtualbox" do |vb|  
                vb.memory = 2048
                vb.cpus = 2
            end
        end
    end

    config.vm.provision "ansible" do |ansible|
        ansible.playbook = ".ansible/playbooks/init_setup.yml"
        ansible.compatibility_mode = "2.0"
        ansible.groups = {
            "database" => ["postgresql_master", "postgresql_slave_1", "postgresql_slave_2", "postgresql_slave_3"],
            "postgresql_master" => ["postgresql_master"],
            "postgresql_slaves" => ["postgresql_slave_1", "postgresql_slave_2", "postgresql_slave_3"],
            "webserver" => ["nginx"],
            "loadbalancer" => ["haproxy"],
            "backend" => ["laravel_node_1", "laravel_node_2"],
            "application" => ["vue", "laravel_node_1", "laravel_node_2"],
            "all_nodes:children" => ["database", "webserver", "loadbalancer", "backend"]
        }
        ansible.limit = "all" 
        ansible.verbose = "v"
    end
end 