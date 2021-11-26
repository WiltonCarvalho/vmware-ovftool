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


# Set VMX Variables
```
set -a
Name=ubuntu-vm
GuestOS=ubuntu-64
VirtualHW=11
```

# Create the VMX File
```
cat <<EOF> $Name.vmx
.encoding = "UTF-8"
bios.bootOrder = "hdd,CDROM"
checkpoint.vmState = ""
cleanShutdown = "TRUE"
config.version = "8"
displayName = "$Name"
ehci.pciSlotNumber = "34"
ehci.present = "TRUE"
ethernet0.addressType = "generated"
ethernet0.bsdName = "en0"
ethernet0.connectionType = "nat"
ethernet0.displayName = "Ethernet"
ethernet0.linkStatePropagation.enable = "FALSE"
ethernet0.pciSlotNumber = "33"
ethernet0.present = "TRUE"
ethernet0.virtualDev = "e1000"
ethernet0.wakeOnPcktRcv = "FALSE"
extendedConfigFile = "$Name.vmxf"
floppy0.present = "FALSE"
guestOS = "$GuestOS"
gui.fullScreenAtPowerOn = "FALSE"
gui.viewModeAtPowerOn = "windowed"
hgfs.linkRootShare = "TRUE"
hgfs.mapRootShare = "TRUE"
ide1:0.present = "TRUE"
ide1:0.fileName = "cidata.iso"
ide1:0.deviceType = "cdrom-image"
isolation.tools.hgfs.disable = "FALSE"
memsize = "512"
nvram = "$Name.nvram"
pciBridge0.pciSlotNumber = "17"
pciBridge0.present = "TRUE"
pciBridge4.functions = "8"
pciBridge4.pciSlotNumber = "21"
pciBridge4.present = "TRUE"
pciBridge4.virtualDev = "pcieRootPort"
pciBridge5.functions = "8"
pciBridge5.pciSlotNumber = "22"
pciBridge5.present = "TRUE"
pciBridge5.virtualDev = "pcieRootPort"
pciBridge6.functions = "8"
pciBridge6.pciSlotNumber = "23"
pciBridge6.present = "TRUE"
pciBridge6.virtualDev = "pcieRootPort"
pciBridge7.functions = "8"
pciBridge7.pciSlotNumber = "24"
pciBridge7.present = "TRUE"
pciBridge7.virtualDev = "pcieRootPort"
powerType.powerOff = "soft"
powerType.powerOn = "soft"
powerType.reset = "soft"
powerType.suspend = "soft"
proxyApps.publishToHost = "FALSE"
replay.filename = ""
replay.supported = "FALSE"
scsi0.pciSlotNumber = "16"
scsi0.present = "TRUE"
scsi0.virtualDev = "lsilogic"
scsi0:0.fileName = "$Name.vmdk"
scsi0:0.present = "TRUE"
scsi0:0.redo = ""
sound.startConnected = "FALSE"
tools.syncTime = "TRUE"
tools.upgrade.policy = "upgradeAtPowerCycle"
usb.pciSlotNumber = "32"
usb.present = "FALSE"
virtualHW.productCompatibility = "hosted"
virtualHW.version = "$VirtualHW"
vmci0.id = "1861462627"
vmci0.pciSlotNumber = "35"
vmci0.present = "TRUE"
vmotion.checkpointFBSize = "65536000"
EOF
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