# -*- mode: ruby -*-
# vi: set ft=ruby :

# ansible_local requires version >= 1.8.4 to work stably
Vagrant.require_version '>= 1.8.4'

required_plugins = %w(vagrant-reload vagrant-triggers vagrant-vbguest nugrant)
plugins_to_install = required_plugins.select { |plugin| !Vagrant.has_plugin? plugin }
unless plugins_to_install.empty?
  puts "Installing plugins: #{plugins_to_install.join(' ')}"
  if system "vagrant plugin install #{plugins_to_install.join(' ')}"
    exec "vagrant #{ARGV.join(' ')}"
  else
    abort 'Installation of one or more plugins has failed. Aborting.'
  end
end

vagrant_dir = File.expand_path(File.dirname(__FILE__))

# All Vagrant configuration is done below. The '2' in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure(2) do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://atlas.hashicorp.com/search.
  # Important: use Bento boxes https://atlas.hashicorp.com/bento not the Canonical ones.
  # Bento boxes are officially-recommended by Vagrant https://www.vagrantup.com/docs/boxes.html
  config.vm.box = 'bento/ubuntu-15.10'

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Customizable configuration
  # See https://github.com/maoueh/nugrant
  config.user.defaults = {
    'ansible' => {
      'skip_tags' => []
    },

    'virtualbox' => {
      'name' => 'development-environment',
      'gui' => true,
      'cpus' => 2,
      'vram' => '64',
      'accelerate3d' => 'off',
      'memory' => '4096',
      'clipboard' => 'bidirectional',
      'draganddrop' => 'bidirectional',
      'audio' => 'dsound',
      'audiocontroller' => 'ac97'
    },

    'java_license_declaration' => '',

    'timezone' => 'Europe/London',

    'locales' => {
      'default' => 'en_GB.UTF-8',
      'present' => ['en_GB.UTF-8', 'en_US.UTF-8']
    },

    'keyboard' => {
      'model' => 'pc105',
      'layout' => 'gb',
      'variant' => ''
    }
  }

  # Fail if Java is being installed and license hasn't been accepted.
  config.trigger.before [:up, :provision] do
    if (!config.user.ansible.skip_tags.include? 'java') && config.user.java_license_declaration != 'I accept the "Oracle Binary Code License Agreement for the Java SE Platform Products and JavaFX" under the terms at http://www.oracle.com/technetwork/java/javase/terms/license/index.html'
      abort "Aborting... to continue you must accept the Oracle Binary Code License Agreement\n(see https://github.com/gantsign/development-environment/wiki/Java-license-declaration)."
    end
  end

  # Ensure Unison service isn't started until Vagrant shared folders are mounted
  # and stopped before shared folders are unmounted (if we don't Unison will
  # assume all files have been deleted and cascade the delete to the client VM).
  config.trigger.after [:up, :reload] do
    run_remote 'bash -c "sudo systemctl start unison || true"'
  end
  config.trigger.before [:halt, :reload] do
    run_remote 'bash -c "sudo systemctl stop unison || true"'
  end

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing 'localhost:8080' will access port 80 on the guest machine.
  # config.vm.network 'forwarded_port', guest: 80, host: 8080

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network 'private_network', ip: '192.168.33.10'

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network 'public_network'

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder '../data', '/vagrant_data'

  # Disable auto update of VirtualBox Guest Additions.
  # Slows down rebuilds without providing any clear benefit.
  config.vbguest.auto_update = false

  config.vm.provider 'virtualbox' do |vb|
    # Give the VM a name
    vb.name = config.user.virtualbox.name

    # Display the VirtualBox GUI when booting the machine
    vb.gui = config.user.virtualbox.gui

    # Customize CPU settings
    vb.cpus = config.user.virtualbox.cpus

    # Customize graphics settings
    vb.customize ['modifyvm', :id, '--vram', config.user.virtualbox.vram]
    vb.customize ['modifyvm', :id, '--accelerate3d', config.user.virtualbox.accelerate3d]

    # Customize the amount of memory on the VM
    vb.memory = config.user.virtualbox.memory

    # Enable host desktop integration
    vb.customize ['modifyvm', :id, '--clipboard', config.user.virtualbox.clipboard]
    vb.customize ['modifyvm', :id, '--draganddrop', config.user.virtualbox.draganddrop]

    # Enable sound
    vb.customize ['modifyvm', :id, '--audio', config.user.virtualbox.audio, '--audiocontroller', config.user.virtualbox.audiocontroller]
  end

  # Customfile
  #
  # Use this to insert your own (and possibly rewrite) Vagrant config lines.
  # If a file 'Customfile' exists in the same directory as this Vagrantfile,
  # it will be evaluated as ruby inline as it loads.
  if File.exist?(File.join(vagrant_dir, 'Customfile'))
    eval(IO.read(File.join(vagrant_dir, 'Customfile')), binding)
  end

  # View the documentation for the provider you are using for more
  # information on available options.

  # Define a Vagrant Push strategy for pushing to Atlas. Other push strategies
  # such as FTP and Heroku are also available. See the documentation at
  # https://docs.vagrantup.com/v2/push/atlas.html for more information.
  # config.push.define 'atlas' do |push|
  #   push.app = 'YOUR_ATLAS_USERNAME/YOUR_APPLICATION_NAME'
  # end

  # Install the vagrant-cachier if you want to speed up rebuilds at the cost
  # of some disk space.
  if Vagrant.has_plugin?('vagrant-cachier')
    # Configure cached packages to be shared between instances of the same base
    # box. More info on http://fgrehm.viewdocs.io/vagrant-cachier/usage
    config.cache.scope = :box
  end

  # Run Ansible from the Vagrant VM
  config.vm.provision 'ansible_local' do |ansible|
    ansible.playbook = 'provisioning/playbook.yml'
    ansible.galaxy_role_file = 'provisioning/requirements.yml'
    ansible.extra_vars = {
      has_vagrant_cachier: Vagrant.has_plugin?('vagrant-cachier'),

      java_license_declaration: config.user.java_license_declaration,

      timezone: config.user.timezone,

      locales_present: config.user.locales.present,
      locales_default: {
        lang: config.user.locales['default']
      },
      keyboard_model: config.user.keyboard.model,
      keyboard_layout: config.user.keyboard.layout,
      keyboard_variant: config.user.keyboard.variant
    }
    ansible.skip_tags = config.user.ansible.skip_tags
  end

  # Restart the VM after everything is installed
  config.vm.provision :reload
end
