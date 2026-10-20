ssh_dir = File.join(__dir__, "ssh")
ssh_key = File.join(ssh_dir, "id_ed25519_playground")
ssh_pub = ssh_key + ".pub"

# creates ssh keys to access the nodes
unless File.exist?(ssh_key) && File.exist?(ssh_pub)
  Dir.mkdir(ssh_dir) unless Dir.exist?(ssh_dir)
  unless File.exist?(ssh_key)
    system("ssh-keygen -t ed25519 -N '' -f #{ssh_key}")
  end
  unless File.exist?(ssh_pub)
    system("ssh-keygen -y -f #{ssh_key} > #{ssh_pub}")
  end
end

IP_BASE = "192.168.99"
N = 2

# creates ssh configuration to allow root ssh without password
ssh_config = File.join(ssh_dir, "config")
File.open(ssh_config, "w", 0600) do |f|
  f.puts "Host k8s-api-server"
  f.puts "  HostName #{IP_BASE}.200"
  f.puts "  User root"
  f.puts "  IdentityFile ~/.ssh/id_ed25519_playground"
  f.puts "  StrictHostKeyChecking no"
  f.puts "  UserKnownHostsFile /dev/null"
  f.puts

  (1..N).each do |i|
    f.puts "Host node0#{i}"
    f.puts "  HostName #{IP_BASE}.#{i+200}"
    f.puts "  User root"
    f.puts "  IdentityFile ~/.ssh/id_ed25519_playground"
    f.puts "  StrictHostKeyChecking no"
    f.puts "  UserKnownHostsFile /dev/null"
    f.puts
  end
end

Vagrant.configure('2') do |config|
  config.vm.provider "virtualbox" do |v|
      v.memory = 2048
      v.cpus = 1
      v.linked_clone = true
  end

  # provisions the k8s-api-server
  config.vm.define "k8s-api-server" do |machine|
    machine.vm.box = 'bento/ubuntu-24.04'
    machine.vm.network "private_network", ip: "#{IP_BASE}.200"
    machine.vm.hostname = "k8s-api-server"
    machine.vm.provision "ansible" do |ansible|
        ansible.playbook = "ansible-k8s/controlplane-playbook.yml"
        ansible.config_file = "ansible-k8s/ansible.cfg"
        ansible.extra_vars = {
            node_ip: "#{IP_BASE}.200",
            api_server_external_address: "#{IP_BASE}.200",
            kube_version: "1.34"
        }
    end
  end

  # provision the worker nodes
  (1..N).each do |i|
      config.vm.define "node0#{i}" do |node|
          node.vm.box = 'bento/ubuntu-24.04'
          node.vm.network "private_network", ip: "#{IP_BASE}.#{i+200}"
          node.vm.hostname = "node0#{i}"
          node.vm.provision "ansible" do |ansible|
              ansible.playbook = "ansible-k8s/worker-playbook.yml"
              ansible.config_file = "ansible-k8s/ansible.cfg"
              ansible.extra_vars = {
                  node_ip: "#{IP_BASE}.#{i+200}",
                  api_server_external_address: "#{IP_BASE}.200",
                  kube_version: "1.34"
              }
          end
      end
  end

end
