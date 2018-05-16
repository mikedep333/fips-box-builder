# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANTFILE_API_VERSION = "2"

# Synced folders are declared here,
# host => guest
VAGRANT_SYNCED_FOLDERS = {
  # This code
  #'.' => '/vagrant',

  # For access to sibling repos (e.g. pulp_rpm, pulp_docker, ...)
  #'..' => '/home/vagrant/devel',

  # You can speed up package installs in subsequent "vagrant up" operations by making the dnf
  # cache a synced folder. This is essentially what the vagrant-cachier plugin would do for us
  # if it supported dnf, and unfortunately that project is in need of maintainers so this might
  # be the best we can do for now. Note that you'll have to manually create the '.dnf-cache'
  # directory in the same directory as this Vagrantfile for this to work.
  #'.dnf-cache' => '/var/cache/dnf'
}

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  # It is possible to use URLs to nightly images produced by the Fedora project here. You can find
  # the nightly composes here: https://kojipkgs.fedoraproject.org/compose/
  # Sometimes, a nightly compose is incomplete and does not contain a Vagrant image, so you may need
  # to browse that repository a bit to find the latest successful Vagrant image. For example, at the
  # time of this writing, I could set this setting like this for the latest F24 image:
  # config.vm.box = "https://kojipkgs.fedoraproject.org/compose/rawhide/latest-Fedora-Rawhide/compose/CloudImages/x86_64/images/<imagename>.vagrant-libvirt.box"
  config.vm.box = "centos/7"

  # need to use the default vagrant key
  config.ssh.insert_key = false

  # Comment out if you don't want Vagrant to add and remove entries from /etc/hosts for each VM.
  # requires the vagrant-hostmanager plugin to be installed
  #if Vagrant.has_plugin?("vagrant-hostmanager")
      #config.hostmanager.enabled = true
      #config.hostmanager.manage_host = true
  #end

  config.vm.provision "shell", inline: "sudo yum install -y epel-release"

  # Comment this line if you would like to disable the automatic update during provisioning
  #config.vm.provision "shell", inline: "sudo yum upgrade -y"

  # bootstrap and run with ansible
  config.vm.provision "shell", path: "scripts/bootstrap-ansible.sh"
  config.vm.provision "ansible" do |ansible|
      # Uncomment this if you want debug tools like gdb, tcpdump, et al. installed
      # (you don't, unless you know you do)
      # ansible.extra_vars = { pulp_dev_debug: true }
      ansible.playbook = "ansible/vagrant-playbook.yml"
  end

  # Create the "dev" box
  config.vm.define "centos7_fips" do |centos7_fips|
    centos7_fips.vm.host_name = "centos7.dev"

    VAGRANT_SYNCED_FOLDERS.each do |host_path, guest_path|
        # Use SSHFS instead of NFS. Requires the vagrant-sshfs plugin to be installed.
        # Use SSHFS to share directories. The ``-o nonempty`` option is passed to allow
        # mounts on non-empty directories.
        #centos7_fips.vm.synced_folder host_path, guest_path, type: "sshfs", sshfs_opts_append: "-o nonempty"

        # By default, Vagrant wants to mount with NFSv3, which will fail. Let's
        # explicitly mount the code using NFSv4.
        #centos7_fips.vm.synced_folder host_path, guest_path, type: "nfs", nfs_version: 4, nfs_udp: false
    end

    centos7_fips.vm.provider :libvirt do |domain, override|
        domain.cpus = 4
        # In some cases, the guest gets stuck at "Waiting for domain to get an IP address..." if
        # the default cpu_mode, 'host-model', is used. Using 'host-passthrough' cpu_mode prevents
        # VM migration between hosts with different CPUs. However, development environments are
        # expected to live on a single host for the duration of their existence. Due to this known
        # issue and our use case, the default cpu_mode is overridden.
        domain.cpu_mode = "host-passthrough"
        domain.graphics_type = "spice"
        domain.memory = 3096
        domain.video_type = "qxl"

        # Uncomment this to expand the disk to the given size, in GB (default is usually 40)
        # You'll also need to uncomment the provisioning step below that resizes the root partition
        # do not set this to a size smaller than the base box, or you will be very sad
        # domain.machine_virtual_size = 80

        # Uncomment the following line if you would like to enable libvirt's unsafe cache
        # mode. It is called unsafe for a reason, as it causes the virtual host to ignore all
        # fsync() calls from the guest. Only do this if you are comfortable with the possibility of
        # your development guest becoming corrupted (in which case you should only need to do a
        # vagrant destroy and vagrant up to get a new one).
        #
        # domain.volume_cache = "unsafe"

        # Uncomment this to resize the root partition and filesystem to fill the base box disk
        # This script is only guaranteed to work with the default official fedora image, and is
        # only needed it you changed machine_virtual_size above.
        # For other boxen, use at your own risk
        # override.vm.provision "shell", path: "scripts/vagrant-resize-disk.sh"
    end
  end
end
