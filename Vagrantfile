# -*- mode: ruby -*-
# vi: set ft=ruby :

#Project name used for database name,user & pass
project = "develop"
ansibl = 'provision/ansible/'

Vagrant.configure(2) do |config|
	config.vm.box = "ubuntu/trusty64"

	config.hostmanager.enabled = true
	config.hostmanager.manage_host = true
	config.hostmanager.ignore_private_ip = false
	config.hostmanager.include_offline = true

	windows = Vagrant::Util::Platform.windows?

	#virtualbox
	config.vm.provider :virtualbox do |vb,ovrd|
		vb.customize ["setextradata", :id, "VBoxInternal2/SharedFoldersEnableSymlinksCreate/vagrant", "1"]
		if windows
			ovrd.vm.synced_folder "./", "/vagrant", type: "smb", :mount_options => ["file_mode=0666,dir_mode=0777"]
		else
			ovrd.vm.synced_folder "./", "/vagrant", type: "nfs", mount_options: ['rw', 'vers=3', 'tcp', 'fsc','actimeo=1']
		end
	end

	#hyperv
	config.vm.provider "hyperv" do |hv,ovrd|
		hv.vmname = project
		ovrd.vm.synced_folder "./", "/vagrant", type: "smb"
	end

	#Multimachine: the develop machine
	config.vm.define :develop, primary: true do |develop|
		develop.vm.hostname = project
		develop.vm.network :private_network,ip:"192.168.100.101"

		develop.hostmanager.aliases = ["#{project}.local"]

		develop.vm.provision windows ? :guest_ansible : :ansible do |ansible|
			ansible.playbook = "#{ansibl}develop.yml"
			ansible.extra_vars = {
				project: project,
			}
		end
	end
end
