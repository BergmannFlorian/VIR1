

# VMWare config
Create new VM with :

Memory : `4 Gb`  
Processors : `2`  
Hard Disk (SCSI) : `40 GB`  
Network Adapter 1 : `NAT`  
Network Adapter 2 : `Host Only`  

# ESXi install
Install VMware vSphere 6.7  

Keyborad : `Swiss French`  
Root password : `Pa$$w0rd`  
Others Default  

# ESXi config
Change hostname to : `srv-esxi-01`  
Change Network Adapter to static : `192.168.XXX.130/24`  
Restart VM  

Go to browser and type : https://192.168.XXX.130/ui/#/login  

Open **Datastore browser** and upload debian 10 iso  

# Create Debian client
In the navigator, choose **Virtual Machines** and create a new virtual machine :  
Guest OS family : `Linux`  
Guest OS version : `Debian GNU/Linux 10 (64-bit)`  
Memory : `512MB`  
Hard disk : `2GB`  
CD/DVD Drive : `Datastore ISO file` and select the debian 10 iso  

Finish configuration  

Start the Vm and config :  
Region : `Europe`  
Country : `Switzerland`  
Keymap : `Swiss French`  
Root password : `root`  
New user : `cpnv`  
cpnv password : `cpnv`  
Only SSH and tool on list  
Install grub : `Yes`  
Others default  
Finish installation  

Restart Debian and run (need to be connected, restart Debian if not work)  

    apt-get install open-vm-tools

Restart Debian  

# Add datastore  
Shut down VM ESXi  
Add new hard disk : `100GB`  
Start VM  

Open web management : https://192.168.XXX.130/ui/#/login  
Go to **storage** and add new datastore :  
Type : `Create new VMFS`  
Name : `DS_02`  
Disk : `Local VMware, Disk (mpx.vmhba0:C0:T1:L0)`  
Partitioning : ` Use full disk`  
VMFS version : `6`  

# Create VM Windows  
In Workstation, create new VM `Windows 10 x64`  
Run VM Windows and finish installation :  
Country : `Suisse`  
Keyboard : `Français (Suisse)`  
Account : `cpnv`  
Password : `cpnv`  
Finish install and shutdown  

# Migrate VM Windows from Workstation to ESXi  
In Workstation :  
Take snapshoot named : `Prepare to migrate`  
In **VM** -> **Manage** -> **Clone...**  
Clone Source : Existing snapshot `Prepare to migrate`  
Clone method : `Create a linked clone`  
Finish clone  

Start clone and run `C:\Windows\System32\Sysprep\sysprep.exe`  
After, Shutdown clone  

On **VM** -> **Manage** :  
Change Hardware Compatibility : `Workstation 14.x`, Don't create new clone  
Upload to : `Other VMware vSphere Server`, enter ESXi VM info, and choose `DS_02`  

Run VM Windows on ESXi and finish installation :  
Country : `Suisse`  
Keyboard : `Français (Suisse)`  
Account : `root`  
Password : `root`  

# Install VM Tool on Windows 10  
In ESXi web interface -> **Virtual Machines** -> **Guest OS** : clic on **Install VMware Tools**  
Go back to Windows VM and install VMware Tools from virtual CD/DVD drive  

# Deploy App dokuwiki  
[Download ova](https://www.turnkeylinux.org/download?file=turnkey-dokuwiki-15.1-stretch-amd64.ova)  
In ESXi -> **Virtual Machines** menu -> **Register VM** :  
Creation type : `Deploy a ...`  
VM Name : `dokuwiki`  
OVA : choose ova downloaded  
Storage : `DS_02`  
Finish Installation and run VM  

# App dokuwiki config  
Root password : `Pa$$w0rd`  
Admin password : `Pa$$w0rd`  
Other default  

# Veeam Backup  
In Workstation, create new VM Windows Server 2019 :  
Memory : `2 GB`  
Hard disk : `32 GB`  
OS Langage : `English`  
Keyboard : `Swiss French`  
OS Type : `Standard Desktop Experience`  

In VM :  
Install and run `Veeam Backup And Replication Community Edition`  
**Backup Job** -> **Virtual Machine** -> **VMware vSphere** -> **VSphere** :  
DNS name or... : `192.168.XX.130`  
Credentials : `root` `Pa$$w0rd`  
Apply  

# How to backup and restore VM  
First, create job :  
In Veeam Backup, select **Backup Job** -> **Virtual Machine**  
In **Virtual Machines** tab, add the VM you want to save  
Other default  
Run backup :  
In **Home** -> **Jobs** -> **Backup**, run the backup defined  
Restore VM :  
In **Home** -> **Backups** -> [job name] -> [backup name], right clic, `Restore entire VM...`  