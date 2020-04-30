# VMWare config
Create new VM with :

Memory : ``4 Gb``
Processors : ``2``
Hard Disk (SCSI) : ``40 GB``
Network Adapter 1 : ``NAT``
Network Adapter 2 : ``Host Only``

# ESXi install
Install VMware vSphere 6.7

Keyborad : ``Swiss French``
Root password : ``Pa$$w0rd``
Others Default

# ESXi config
Change hostname to : ``srv-esxi-01``
Change Network Adapter to static : ``192.168.XXX.130/24``
Restart VM

Go to browser and type : https://192.168.XXX.130/ui/#/login

Open **Datastore browser** and upload debian 10 iso

# Create Debian client
In the navigator, choose **Virtual Machines** and create a new virtual machine :
Guest OS family : ``Linux``
Guest OS version : ``Debian GNU/Linux 10 (64-bit)``
Memory : ``512MB``
Hard disk : ``2GB``
CD/DVD Drive : ``Datastore ISO file`` and select the debian 10 iso

Finish configuration

Start the Vm and config :
Region : ``Europe``
Country : ``Switzerland``
Keymap : ``Swiss French``
Root password : ``root``
New user : ``cpnv``
cpnv password : ``cpnv``
Only SSH and tool on list
Install grub : ``Yes``
Others default

