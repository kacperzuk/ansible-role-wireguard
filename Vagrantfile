Vagrant.configure(2) do |config|
  config.vm.box = "debian/stretch64"
  config.vm.network "private_network", type: "dhcp"
  config.vm.synced_folder '.', '/vagrant', disabled: true

  config.vm.provision "ansible" do |ansible|
    ansible.playbook = "configure-wg.yml"
    ansible.become = true
    ansible.groups = {
      "wireguard_gateway" => [ "gateway" ],
      "wireguard_clients" => [ "c1", "c2" ]
    }
    ansible.host_vars = {
      "gateway" => {
        "wireguard_address" => "10.222.222.1/24",

        # interface that traffic from clients will be routed to
        "wireguard_forward_interface" => "eth0",

        # interface that clients should use to connect to gateway
        "wireguard_connect_interface" => "eth1",

        # optional, just for testing below
        "wireguard_port" => 5555
      },
      "c1" => { "wireguard_address" => "10.222.222.11/24" },
      "c2" => { "wireguard_address" => "10.222.222.12/24" },
    }
  end

  config.vm.define "gateway" do |vm|
  end

  config.vm.define "c1" do |vm|
  end

  config.vm.define "c2" do |vm|
  end
end
