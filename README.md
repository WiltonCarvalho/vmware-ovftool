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

# Cidata ISO | Clodu Init | User-Data
```
sudo apt install cloud-image-utils
cat <<EOF>> user-data
#cloud-config
password: passw0rd
chpasswd: { expire: False }
ssh_pwauth: True
EOF
touch meta-data
cloud-localds -f iso cidata.iso user-data meta-data
```

# OVA Build
```
wget https://cloud-images.ubuntu.com/focal/current/focal-server-cloudimg-amd64.vmdk -O $Name.vmdk
ovftool $Name.vmx $Name.ova
```