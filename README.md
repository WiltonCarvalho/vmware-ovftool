# Packer VMX Template
https://github.com/hashicorp/packer/blob/20541a7eda085aa5cf35bfed5069592ca49d106e/builder/vmware/step_create_vmx.go#L84

# OVF Tool Download
https://code.vmware.com/web/tool/4.3.0/ovf

# OVFTool Installation
```
unzip VMware-ovftool-4.4.3-18663434-lin.x86_64.zip
grep ovftool ~/.bashrc || echo -e "\n #OVF Tool\n PATH=\$PATH:$PWD/ovftool" >> ~/.bashrc
source ~/.bashrc
ovftool --version
```

# virtualHW.version
https://kb.vmware.com/s/article/1003746

# Create the VMX File
```
set -a
Name=ubuntu-vm
GuestOS=ubuntu-64
VirtualHW=11
cat vmx.template | envsubst | tee $Name.vmx
```

# cloud-localds and mkpasswd commands
```
sudo apt install cloud-image-utils whois

```
# Cidata ISO | Clodu Init | User-Data
```
echo passw0rd | mkpasswd -s --method=SHA-512
cat <<'EOF'> user-data
#cloud-config
password: $6$xc3sj4biw7Uu/PB$RE9zkgAzYZhd80J5I4GU5EGzt/9hvCd/Yn0rCYc8.2m.t7hfs.xPT6m/lndEEhLnW8ADGv9PD4ESZT634TnJQ.
chpasswd: { expire: False }
ssh_pwauth: True
ssh_authorized_keys:
  - ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDFOvXax9dNqU2unqd+AZQ+VSe2cZZbGMVRuzIW4Hl6Ji69R0zkWih0vuP2psRA/uWTg1XqFKisCp9Z1XQcBbH2WLhnIWhykeLOHtBdEQqUApKj+BrKnyDmBbCourUwAcuUQSRPeRBOg5hwReviIebwvELmwc8ab1r0X+nbCDwVdohTpwNnxHp5MTO0WADLdP0oDQy2hhVaiParCWdVvgfDauQ2IpgeN6tE5sUvsDyYLaYp/dIhddA/Dwh9sWEFfN7ERMSHJw/A/3GsQ49a8+w6lamgcfNDKK7hE9F5vn95fzhge0jj6Yl8NTXOzoMfpvPo3Q+uCbu+GRMlRAK3hcHP my_admin_user_pub_key
system_info:
  default_user:
    name: ubuntu
    gecos: Ubuntu
write_files:
  - path: /etc/multipath.conf
    permissions: "0644"
    owner: "root"
    content: |
      # https://bugs.launchpad.net/ubuntu/+source/multipath-tools/+bug/1875594
      defaults {
          user_friendly_names yes
      }
      blacklist {
          device {
              vendor "VMware"
              product "Virtual disk"
          }
      }
  - path: /opt/ps1.txt
    permissions: "0644"
    owner: "root"
    content: |
      if [ $(id -u) -eq 0 ]; then
        PS1='\[\e]0;\u@\h\a\]\[\033[0;31m\]\u@\h\[\033[0;35m\]$(__git_ps1)\[\033[0;36m\] ${PWD}\n\[\033[0;31m\]└─\[\033[0;31m\] \$\[\033[0;31m\] ▶\[\033[0m\] '
      else
        PS1='\[\e]0;\u@\h\a\]\[\033[0;32m\]\u@\h\[\033[0;35m\]$(__git_ps1)\[\033[0;36m\] ${PWD}\n\[\033[0;32m\]└─\[\033[0;32m\] \$\[\033[0;32m\] ▶\[\033[0m\] '
      fi
runcmd:
  - for i in $(echo /home/*/.bashrc /etc/skel/.bashrc /root/.bashrc); do cat /opt/ps1.txt >> $i; done
EOF
touch meta-data
cloud-localds -f iso cidata.iso user-data meta-data
```

# OVA Build
```
wget https://cloud-images.ubuntu.com/focal/current/focal-server-cloudimg-amd64.vmdk -O $Name.vmdk
ovftool $Name.vmx $Name.ova
```