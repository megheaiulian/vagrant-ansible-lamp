# -*- mode: ruby -*-
# vi: set ft=ruby :
host = RbConfig::CONFIG['host_os']

cpus = 1
mem = 512

if host =~ /darwin/
	cpus = `sysctl -n hw.ncpu`.to_i
	mem = `sysctl -n hw.memsize`.to_i / 1024 / 1024 / 4
elsif host =~ /linux/
	cpus = `nproc`.to_i
	mem = `grep 'MemTotal' /proc/meminfo | sed -e 's/MemTotal://' -e 's/ kB//'`.to_i / 1024 / 4
elsif host =~ /mswin|mingw|cygwin/
	cpus = `wmic cpu Get NumberOfCores`.split[1].to_i
	mem = `wmic computersystem Get TotalPhysicalMemory`.split[1].to_i / 1024 / 1024
end
cpus = cpus / 2 if cpus > 1
mem  = mem / 4  if mem > 2048

#Project name used for database name,user & pass
project = "develop"

Vagrant.configure(2) do |config|
	config.vm.box = "hashicorp/precise64"

	config.hostmanager.enabled = true
	config.hostmanager.manage_host = true
	config.hostmanager.ignore_private_ip = false
	config.hostmanager.include_offline = true

	#hyperv
	config.vm.provider "hyperv" do |hv,ovrd|
		hv.cpus = cpus
		hv.maxmemory = mem
		hv.vmname = project
		ovrd.vm.synced_folder "./", "/vagrant", type: "smb"
	end

	#virtualbox
	config.vm.provider :virtualbox do |vb,ovrd|
		vb.customize ["modifyvm", :id, "--memory", mem]
		vb.customize ["modifyvm", :id, "--cpus", cpus]
		vb.customize ["setextradata", :id, "VBoxInternal2/SharedFoldersEnableSymlinksCreate/vagrant", "1"]
		if host =~ /darwin/
			ovrd.vm.synced_folder "./", "/vagrant", type: "nfs", mount_options: ['rw', 'vers=3', 'tcp', 'fsc']
		end
	end

	#Multimachine: the develop machine
	config.vm.define :develop, primary: true do |develop|
		develop.vm.hostname = project
		develop.vm.network :private_network,ip:"192.168.100.101"

		develop.hostmanager.aliases = ["#{project}.local"]

		develop.vm.provision (Vagrant::Util::Platform.windows? ? :guest_ansible : :ansible) do |ansible|
			ansible.playbook = 'ansible/develop.yml'
			ansible.extra_vars = {
				project_name: project,
				project_mysql_root_pass: 'rootroot',
				#project_sql_path: 'config/db/gybell.sql'
			}
		end
	end
end
