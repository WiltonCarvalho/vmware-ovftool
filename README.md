# Packer VMX Template
https://github.com/hashicorp/packer/blob/20541a7eda085aa5cf35bfed5069592ca49d106e/builder/vmware/step_create_vmx.go#L84

# OVF Tool Download
https://code.vmware.com/web/tool/4.4.0/ovf
https://developer.vmware.com/web/tool/4.4.0/ovf

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
Name=ubuntu-template
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
password: $6$BliEDnYgfJM3$xkQF9L9ohnWLeh6lyr3iuiekkwlWQj0jH2T.xIdOdZVQHk7ti1K8pQNA0GzmKmmBbxWUxlmFduvI.ZT2T9VMM.
chpasswd: { expire: False }
ssh_pwauth: True
ssh_authorized_keys:
  - ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC6lrQ+ZcjqHCKt0nmxWvkIRQIYCoiyr371Ytqs90trnOUPDfWlfW4nRRY3qXSQySw1YU+slcuamoS4hcOm0qsVvrnyLp35HFKL+A5v+rJfB1ZS+GQ8smEf6RAzfvvAk4zMYglNcC1uWTsH8W3WQPbh6Lb5y+Oipi8TJo/8FEyX5rKTNRWkf+FsB7Dm3xXfJB7zpTOXdEobMrCmqYJDVBGNFmEFUrUKdt96abC1YetIcTucHBJndbtn59hC3SOx6QbkcdYC94+geYDOQGE1RbMozBK5vKKt4C6Kgd/u2ngaTxqwDhc/FZTVtQLywnHYTx0hghyOf9SoOicGQOslndw/ my_ssh_pub_key
system_info:
  default_user:
    name: ubuntu
    gecos: Ubuntu
timezone: America/Sao_Paulo
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
  - path: /etc/profile.d/custom.sh
    permissions: "0755"
    owner: "root"
    content: |
      HISTTIMEFORMAT="%Y-%m-%d %H:%M:%S | "
      TMOUT=0
      HISTCONTROL=ignoredups:ignorespace
      HISTSIZE=200000
      HISTFILESIZE=-1
      HISTFILE=$HOME/.bash_history
      export HISTTIMEFORMAT TMOUT HISTCONTROL HISTSIZE HISTFILESIZE HISTFILE
      readonly HISTTIMEFORMAT TMOUT HISTCONTROL HISTSIZE HISTFILESIZE HISTFILE
      alias rm='rm -I --preserve-root'
      alias mv='mv -i'
      alias cp='cp -i'
      alias ln='ln -i'
      alias chown='chown --preserve-root'
      alias chmod='chmod --preserve-root'
      alias chgrp='chgrp --preserve-root'
      alias grep='grep --color=auto'
  - path: /etc/sudoers.d/90-sudo
    permissions: "0644"
    owner: "root"
    content: |
      %sudo ALL=NOPASSWD: ALL
runcmd:
  - for i in $(echo /home/*/.bashrc /etc/skel/.bashrc /root/.bashrc); do cat /opt/ps1.txt >> $i; done
  - sed -i '/HIST/d' /home/*/.bashrc /etc/skel/.bashrc /root/.bashrc
  - systemctl restart multipathd
EOF
touch meta-data
cloud-localds -f iso cidata.iso user-data meta-data
```

# OVA Build
```
wget https://cloud-images.ubuntu.com/focal/current/focal-server-cloudimg-amd64.vmdk -O $Name.vmdk
ovftool $Name.vmx $Name.ova
```
