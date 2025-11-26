IP_BASE = "192.168.99"
N = 2
Vagrant.configure('2') do |config|
  config.vm.provider "virtualbox" do |v|
      v.memory = 2048
      v.cpus = 1
  end
  config.vm.define "controlplane" do |machine|
    machine.vm.box = 'bento/ubuntu-24.04'
    machine.vm.network "private_network", ip: "#{IP_BASE}.200"
    machine.vm.hostname = "controlplane"
    machine.vm.provision "ansible" do |ansible|
        ansible.playbook = "ansible-k8s/controlplane-playbook.yml"
        ansible.extra_vars = {
            node_ip: "#{IP_BASE}.200",
            api_server_external_address: "#{IP_BASE}.200",
            kube_version: "1.34"
        }
    end
  end

  (1..N).each do |i|
      config.vm.define "k8s-worker-000#{i}" do |node|
          node.vm.box = 'bento/ubuntu-24.04'
          node.vm.network "private_network", ip: "#{IP_BASE}.#{i+200}"
          node.vm.hostname = "k8s-worker-000#{i}"
          node.vm.provision "ansible" do |ansible|
              ansible.playbook = "ansible-k8s/worker-playbook.yml"
              ansible.extra_vars = {
                  node_ip: "#{IP_BASE}.#{i+200}",
                  api_server_external_address: "#{IP_BASE}.200",
                  kube_version: "1.34"
              }
          end
      end
  end

end
