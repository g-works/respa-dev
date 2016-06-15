require 'yaml'
require 'fileutils'
require 'socket'

DIR = File.dirname(__FILE__)

unless File.exist?(File.join(DIR,'.vagrant'))
  FileUtils.mkdir_p( File.join(DIR,'.vagrant') )
end

private_ip_file = File.join(DIR,'.vagrant','private.ip')

unless File.exists?(private_ip_file)
  private_ip = "192.168.#{rand(255)}.#{rand(255)}"
  File.write(private_ip_file, private_ip)
else
  private_ip = File.open(private_ip_file, 'rb') { |file| file.read }
end
Vagrant.require_version '>= 1.7.4'
Vagrant.configure('2') do |config|
  config.ssh.forward_agent = true
  config.vm.box_version = ">= 0.0"
  config.vm.box = 'gworksoy/respa-dev'
  config.vm.hostname = 'respa-dev'
  
  config.vm.network :private_network, ip: private_ip
  config.vm.define 'respa-dev-box'
  if Vagrant.has_plugin? 'vagrant-hostsupdater'
    config.hostsupdater.remove_on_suspend = true
    config.hostsupdater.aliases = ["respa.local"]
  else
    puts 'vagrant-hostsupdater missing, please install the plugin:'
    puts 'vagrant plugin install vagrant-hostsupdater'
    exit 1
  end
g
  # Remove default share
  config.vm.synced_folder ".", "/vagrant", disabled: true

  # Make synced folder from vagrant machine
  config.vm.synced_folder "project/", '/data/www/project', owner: 'vagrant', group: 'www-data', mount_options: ['dmode=776', 'fmode=775']


  config.vm.provider 'virtualbox' do |vb|

    # Memory and CPU
    vb.customize ['modifyvm', :id, '--memory', 1024]
    vb.customize ['modifyvm', :id, '--cpus', 1]

    # Network fixes (DNS-fix)
    vb.customize ['modifyvm', :id, '--natdnshostresolver1', 'on']
    vb.customize ['modifyvm', :id, '--natdnsproxy1', 'on']
  end


def confirm(question,default=true)
  if default
    default = "yes"
  else
    default = "no"
  end

  confirm = nil
  until ["Y","N","YES","NO",""].include?(confirm)
    confirm = ask "#{question} (#{default}): "

    if (confirm.nil? or confirm.empty?)
      confirm = default
    end

    confirm.strip!
    confirm.upcase!
  end
  if confirm.empty? or confirm == "Y" or confirm == "YES"
    return true
  end
  return false
end

def vagrant_running?
  system("vagrant status --machine-readable | grep state,running --quiet")
end

def has_internet?
  orig, Socket.do_not_reverse_lookup = Socket.do_not_reverse_lookup, true  # turn off reverse DNS resolution temporarily
  begin
    UDPSocket.open { |s| s.connect '8.8.8.8', 1 }
    return true
  rescue Errno::ENETUNREACH
    return false # Network is not reachable
  end
ensure
  Socket.do_not_reverse_lookup = orig
end
end