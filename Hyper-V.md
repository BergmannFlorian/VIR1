# Part 2 Cluster Hyper-V

![Shema](VIR_HyperV.png)

# AD Server Install
In Workstation create new VM Windows Server 2019 :
Virtual machine name: `W2k19 Server AD`
Ram : default
Disk : default
Network : `NAT`

Language : `English`
Time ...: `French (Switzerland)`
Keyboard : `Swiss French`
OS : `Windows Server 2019 Standard (Desktop Experience)`
Password : `Pa$$w0rd`
Finish install

# Ad Server Config
Start VM `W2k19 Server AD`
Edit network : 
    IPv4 `192.168.XX.10`
    Gateway : `192.168.XX.2`
    DNS : `127.0.0.1`
Change name in config by : `VIR-AD-1`
Disable Firewall
Restart server

In **Server Manager** -> **Manage**, add Roles
Select server : `VIR-AD-1`
Server Roles : `Active Directory Domain Services`
Finish install
Restart server

In **Server Manager**, clic on the flag on the top and clic on **Promote this server to a domain controller**
Add new forest : `hyperv.dom`
Password : `Pa$$w0rd`
Finish install

# SAN iSCSI Install
Create new VM Windows Server 2019 :
Virtual machine name: `VIR-SAN-1`
Ram : default
Disk 1 : `60GB`
Disk 2 : `60GB`
Network 0 : `NAT`
Network 1 : `Host-Only`

Language : `English`
Time ...: `French (Switzerland)`
Keyboard : `Swiss French`
OS : `Windows Server 2019 Standard (Desktop Experience)`
Password : `Pa$$w0rd`
Finish install

# SAN iSCSI Config
Start VM `VIR-SAN-1`
Edit Network Nat : 
    IP : `192.168.XX.20` 
    Gateway : `192.168.XX.10`
    DNS : `192.168.XX.10`
Edit Network Host-Only : 
    IP : `10.10.10.20`
Change name : `VIR-SAN-1`
Change Domain : `hyperv.dom`
Disable Firewall
Restart server

In **Server Manager** -> **Manage**, add roles **File and Storage Services** -> **File and iSCSI Services** -> `iSCSI Target Server`
Finish install

Open **Disk Management** and put Online Disk 1 and initialize it
Create new simple volume on Disk 1 :
Size : max
Drive letter : `S`
Volume label : `iSCSI`
Finish

**Server Manager** -> **File and Storage Services** -> **iSCSI**, start new iSCSI
Disk Location : volume `S`
Disk Name : `iSCSI-1`
Disk Size : max
Target Name : `virtual-target`
Acess Servers : 
    IP : `10.10.10.30`
    IP : `10.10.10.40`
Finish creation

# HV 1 Install
Create new VM Windows Server 2019 :
Virtual machine name: `VIR-HV-1`
Ram : default
Disk : `60GB`
Network 0 : `NAT`
Network 1 : `Host-Only`

Language : `English`
Time ...: `French (Switzerland)`
Keyboard : `Swiss French`
OS : `Windows Server 2019 Standard (Desktop Experience)`
Password : `Pa$$w0rd`
Finish install

# HV 1 Config
Start VM `VIR-SAN-1`
Change name : `VIR-HV-1`
Change Domain : `hyperv.dom`
Disable Firewall
Restart server

# HV 1 HyperV Install
In **Server Manager** -> **Manage**, add roles `Hyper-V`
If you are an error :
shutdown VM, open the file *.vmx and add

    hypervisor.cpuid.v0 = "FALSE"
    mce.enable = "TRUE" 
    vhu.enable = "TRUE"
Edit Processors on VM and activate virtualize Intel and CPU

Continue installation of HyperV
Server Roles : `Hyper-V`
Hyper-V -> Virtual Switches : Select all
Finish install
Restart server

Edit Network Nat : 
    IP : `192.168.XX.30`
    Gateway : `192.168.XX.10`
    DNS : `192.168.XX.10`
Edit Network Host-Only : 
    IP : `10.10.10.30`

In **Server Manager** -> **Tools**, open `iSCSI Initiator`
Connect to : `10.10.10.20`
Finish

# HV 2
Repeat same of HV 1 with :
Virtual machine name: `VIR-HV-2`
Network Nat : 
    IP : `192.168.XX.40`
    Gateway : `192.168.XX.10`
    DNS : `192.168.XX.10`
Network Host-Only :  
    IP : `10.10.10.40`
Name : `VIR-HV-2`

# Cluster
On VIR-HV-1, open **Hyper-V Manager**
Right clic on `VIR-HV-1` and open `Virtual Switch Manager`
Select NAT network, check `VLAN ID` and change value by `20`

In **Server Manager** -> **Manage**, add feature `Failover Clustering`
Restart server

Open **Failover Cluster Manager**
Right clic and choose `Create Cluster`