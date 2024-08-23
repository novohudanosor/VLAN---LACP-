# -*- mode: ruby -*-
# vim: set ft=ruby :
# -*- mode: ruby -*-
# vim: set ft=ruby :

MACHINES = {
  :inetRouter => {
        :box_name => "centos/7",
        :vm_name => "inetRouter",
        :net => [
                   {adapter: 2, auto_config: false, virtualbox__intnet: "router-net"},
                   {adapter: 3, auto_config: false, virtualbox__intnet: "router-net"},
                   {ip: "192.168.56.10", adapter: 8},
                ]
  },
  
  :centralRouter => {
        :box_name => "centos/7",
        :vm_name => "centralRouter",
        :net => [
                   {adapter:  2, auto_config: false, virtualbox__intnet: "router-net"},
                   {adapter:  3, auto_config: false, virtualbox__intnet: "router-net"},
                   {ip: "192.168.255.9", adapter: 6, netmask: "255.255.255.252", virtualbox__intnet: "office1-central"},
                   {ip: "192.168.56.11", adapter: 8},
                ]
  },
  
  :office1Router => {
        :box_name => "centos/7",
        :vm_name => "office1Router",
        :net => [
                    {ip: "192.168.255.10", adapter: 2, netmask: "255.255.255.252", virtualbox__intnet: "office1-central"},
                    {adapter: 3, auto_config: false, virtualbox__intnet: "vlan1"},
                    {adapter: 4, auto_config: false, virtualbox__intnet: "vlan1"},
                    {adapter: 5, auto_config: false, virtualbox__intnet: "vlan2"},
                    {adapter: 6, auto_config: false, virtualbox__intnet: "vlan2"},
                    {ip: "192.168.56.20", adapter: 8},
                ]
  },
  
  :testClient1 => {
    :box_name => "centos/7",
    :vm_name => "testClient1",
    :net => [
                {adapter: 2, auto_config: false, virtualbox__intnet: "testLAN"},
                {ip: "192.168.56.21", adapter: 8},  
            ]
},

:testServer1 => {
    :box_name => "centos/7",
    :vm_name => "testServer1",
    :net => [
                {adapter: 2, auto_config: false, virtualbox__intnet: "testLAN"},
                {ip: "192.168.56.22", adapter: 8},  
            ]
},

:testClient2 => {
    :box_name => "ubuntu/jammy64",
    :vm_name => "testClient2",
    :net => [
                {adapter: 2, auto_config: false, virtualbox__intnet: "testLAN"},
                {ip: "192.168.56.31", adapter: 8},  
            ]
},

:testServer2 => {
    :box_name => "ubuntu/jammy64",
    :vm_name => "testServer2",
    :net => [
                {adapter: 2, auto_config: false, virtualbox__intnet: "testLAN"},
                {ip: "192.168.56.32", adapter: 8},  
            ]
},
}

Vagrant.configure("2") do |config|
  config.ssh.insert_key = false
  MACHINES.each do |boxname, boxconfig|
    config.vm.define boxname do |box|
        box.vm.box = boxconfig[:box_name]
        box.vm.host_name = boxconfig[:vm_name]
        
        box.vm.provider "virtualbox" do |v|
          v.gui = false
          v.memory = 1024
          v.cpus = 2
        end

  
    end
  end
end
