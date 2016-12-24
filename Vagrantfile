VAULT_VERSION = '0.6.4' #  Set target vault version

Vagrant.configure("2") do |config|
  if Vagrant.has_plugin? 'vagrant-hostmanager'
    config.hostmanager.enabled = true
    config.hostmanager.manage_host = true
    config.hostmanager.manage_guest = true
    config.hostmanager.include_offline = true
  else
    puts <<-MSG
===== vagrant-hostmanager plugin is not installed! =====
===== VMs will not be available using the hostname =====
MSG
  end

  config.vm.provision "shell", inline: <<-SHELL
#!/bin/bash
set -e

# Disable NM and make sure network service is enabled on boot
echo 'Stopping and disabling NetworkManager...'
systemctl stop NetworkManager
systemctl disable NetworkManager

echo 'Configuring network service to start on boot...'
systemctl enable network
systemctl restart network

# Update this list with any additional packages you may want available on the VMs
yum install -y emacs-nox vim tmux bind-utils git subversion zip unzip
SHELL

  config.vm.define 'vault1' do |vault|
    vault.vm.box = 'bento/centos-7.2'
    vault.vm.network :private_network, ip: '192.168.11.100'
    vault.vm.hostname = 'vault1.vagrant.local'
    vault.vm.provision :shell, inline: <<-SHELL
#!/bin/bash
set -e
if [ ! -f /usr/local/sbin/vault ]; then
  echo 'Downloading Vault binary #{VAULT_VERSION}'
  curl -sSLo vault.zip https://releases.hashicorp.com/vault/#{VAULT_VERSION}/vault_#{VAULT_VERSION}_linux_amd64.zip
  curl -sSLo vault.checksums https://releases.hashicorp.com/vault/#{VAULT_VERSION}/vault_#{VAULT_VERSION}_SHA256SUMS

  echo 'Validating checksum of downloaded archive'
  checksum=$(sha256sum vault.zip | awk '{print $1}')
  if ! grep -qi "$checksum" vault.checksums; then
    echo 'Checksum of downloaded archive failed to validate!' 1>&2
    exit 1
  fi

  echo 'Installing Vault'
  unzip -d /usr/local/sbin vault.zip
fi

if [ ! -d /etc/vault.d ]; then
  echo 'Creating configuration directory'
  mkdir /etc/vault.d
fi

echo 'Copying configuration'
configs=$(find /vagrant/files/config/ -type f \\( -name \*.hcl -o -name \*.json \\) 2>/dev/null )
for f in $configs; do
  cp -f "$f" /etc/vault.d
done
cp -f /vagrant/files/vaultenv /etc/sysconfig/vault

if [ ! -d /var/vault ]; then
  echo 'Creating vault file store and linking to host machine (this should be switched to Consul at some point)'
  ln -s /vagrant/vault_store /var/vault
fi

echo 'Making sure Vault service is up to date'
cp -f /vagrant/vault.service /etc/systemd/system/vault.service
systemctl enable vault.service

echo 'Starting vault.service'
systemctl start vault.service || echo 'vault.service failed to start'
SHELL
  end
end
