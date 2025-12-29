Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/bionic64"  # Ubuntu 18.04 with SMB1 support
  # Force VirtualBox provider and reasonable defaults
  config.vm.provider "virtualbox" do |vb|
    vb.memory = "2048"
    vb.cpus = 2
  end
  # Use a host-accessible private network IP so the host can reach the VM's
  # Samba server. NAT addresses like 10.0.2.15 are not reachable from host.
  # Use VirtualBox host-only network (example IP below).
  config.vm.network "private_network", ip: "192.168.56.10"
  config.vm.provision "shell", inline: <<-SHELL
    apt-get update
    apt-get install -y cifs-utils samba
    mkdir -p /home/vagrant/mnt
cat > /etc/samba/smb.conf << 'EOF'
[global]
   # VM acts as a client to the old Time Capsule (SMB1/NT1)
   client min protocol = NT1
   client max protocol = NT1
   client use spnego = no
   client ntlmv2 auth = no
   ntlm auth = yes

   # But when serving to the host we force modern server protocols
   server min protocol = SMB2
   server max protocol = SMB3

   # Allow guests to map to the vagrant account so host can mount without
   # requiring manual smbpasswd setup; host will negotiate SMB2/3 to the VM.
   map to guest = Bad User
   guest account = vagrant

[timecapsule]
   path = /home/vagrant/mnt
   read only = yes
   guest ok = yes
   browseable = yes
   create mask = 0777
   directory mask = 0777
   force user = vagrant
EOF
    systemctl restart smbd || service smbd restart || true
  SHELL
end