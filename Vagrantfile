Vagrant.configure(2) do |config|
  config.vm.boot_timeout = 600

  # VirtualBox Guest Additionsの自動更新を無効化
  if Vagrant.has_plugin?("vagrant-vbguest")
    config.vbguest.auto_update = false
    config.vbguest.no_remote = true
  end

  # 仮想マシンのボックス設定
  config.vm.box = "bento/ubuntu-24.04"
  config.vm.box_architecture = "amd64" # amd64(Windows), arm64(Mac)

  # 仮想マシンのネットワーク設定
  config.vm.network "private_network", ip: "192.168.33.10"
  config.vm.synced_folder ".", "/vagrant", type: "virtualbox"

  # VirtualBox のリソース（メモリ容量など）を個別に設定
  config.vm.provider "virtualbox" do |vb|
    vb.memory = "2048"
  end

  # Ansibleを仮想マシン内でローカル実行してプロビジョニングを行う設定
  config.vm.provision "ansible_local" do |ansible|
    ansible.become = true
    ansible.playbook = "playbooks/main.yml"
    ansible.galaxy_role_file = "playbooks/requirements.yml"
    ansible.galaxy_command = 'ansible-galaxy install -r %{role_file} --force'
  end
end
