# Part 2 Cluster Hyper-V

![Shema](VIR_HyperV.png)

## AD Server Install
In Workstation create new VM Windows Server 2019 :  
- Virtual machine name: `W2k19 Server AD`  
- Ram : default  
- Disk : default  
- Network : `NAT`  

OS Install :
- Language : `English`  
- Time ...: `French (Switzerland)`  
- Keyboard : `Swiss French`  
- OS : `Windows Server 2019 Standard (Desktop Experience)`  
- Password : `Pa$$w0rd`  

Finish install  

## Ad Server Config  
Start VM `W2k19 Server AD`  
Edit network NAT:   
- IPv4 `192.168.XX.10`  
- Gateway : `192.168.XX.2`  
- DNS : `127.0.0.1`  

Change computer name in config by : `VIR-AD-1`  
Disable Firewall  
Restart server  

In **Server Manager** -> **Manage**, add Roles `Active Directory Domain Services`  
Finish install  
Restart server  

In **Server Manager**, clic on the flag on the top and clic on **Promote this server to a domain controller**  
Add new forest : `hyperv.dom`  
Password : `Pa$$w0rd`  
Finish install  

## SAN iSCSI Install
Create new VM Windows Server 2019 :  
- Virtual machine name: `VIR-SAN-1`  
- Ram : default   
- Disk 1 : `60GB`  
- Disk 2 : `60GB`  
- Network 0 : `NAT`  
- Network 1 : `Host-Only`  

OS Install :
- Language : `English`  
- Time ...: `French (Switzerland)`  
- Keyboard : `Swiss French`  
- OS : `Windows Server 2019 Standard (Desktop Experience)`  
- Password : `Pa$$w0rd`  
- Finish install  

## SAN iSCSI Config  
Start VM `VIR-SAN-1`  
Edit Network Nat :   
- IP : `192.168.XX.20`   
- Gateway : `192.168.XX.10`  
- DNS : `192.168.XX.10`  

Edit Network Host-Only :   
- IP : `10.10.10.20`  

Change name : `VIR-SAN-1`  
Change Domain : `hyperv.dom`  
Disable Firewall  
Restart server  

In **Server Manager** -> **Manage**, add roles **File and Storage Services** -> **File and iSCSI Services** -> `iSCSI Target Server`  
Finish install  

Open **Disk Management** and put Online `Disk 1` and initialize it  
Create new simple volume on `Disk 1` :  
- Size : max  
- Drive letter : `S`  
- Volume label : `iSCSI`  

Finish  

**Server Manager** -> **File and Storage Services** -> **iSCSI**, start new iSCSI :  
- Disk Location : volume `S`  
- Disk Name : `iSCSI-1`  
- Disk Size : `55GB`  
- Target Name : `virtual-target`  
- Acess Servers :   
    - IP : `10.10.10.30`  
    - IP : `10.10.10.40`  

Finish creation  

Right clic and create `New iSCSI Virtual Disk...` :  
- Disk Location : volume `C`  
- Disk Name : `voting-ghost`  
- Disk Size : `5GB`  
- Target Name : `voting-ghost-target`  
- Acess Servers :   
    - IP : `10.10.10.30`  
    - IP : `10.10.10.40`  

Finish creation  

## HV 1 Install  
Create new VM Windows Server 2019 :  
- Virtual machine name: `VIR-HV-1`  
- Ram : `4GB`  
- Disk : `60GB`  
- Network 0 : `NAT`  
- Network 1 : `Host-Only`  

OS Install :
- Language : `English`  
- Time ...: `French (Switzerland)`  
- Keyboard : `Swiss French`  
- OS : `Windows Server 2019 Standard (Desktop Experience)`  
- Password : `Pa$$w0rd`  

Finish install  

## HV 1 Config
Start VM `VIR-SAN-1`  
Edit Network Nat :  
- IP : `192.168.XX.30`  
- Gateway : `192.168.XX.10`  
- DNS : `192.168.XX.10`

Edit Network Host-Only :   
- IP : `10.10.10.30`  

Change computer name : `VIR-HV-1`  
Change Domain : `hyperv.dom`  
Disable Firewall  
Restart server  

## HV 1 HyperV Install  
In **Server Manager** -> **Manage**, add roles `Hyper-V`  
If you are an error :  
shutdown VM, open the file *.vmx and add  
```conf
    hypervisor.cpuid.v0 = "FALSE"  
    mce.enable = "TRUE"   
    vhu.enable = "TRUE"  
```
Edit **Processors** on VM and activate `virtualize Intel` and `CPU`   

Continue installation of roles : `Hyper-V`  
Finish install  
Restart server  


In **Server Manager** -> **Tools**, open `iSCSI Initiator`  
Connect to : `10.10.10.20`  
Connect the 2 target  
Finish  

## HV 2
Repeat same of HV 1 with :  
- Virtual machine name: `VIR-HV-2`  
- Network Nat :   
    - IP : `192.168.XX.40`  
    - Gateway : `192.168.XX.10`  
    - DNS : `192.168.XX.10`  
- Network Host-Only :    
    - IP : `10.10.10.40`  
- Computer name : `VIR-HV-2`  

## Cluster

In **Server Manager** -> **Manage**, add feature `Failover Clustering`  
Restart server  

Open **Failover Cluster Manager**  
Right clic and choose `Create Cluster`  
- Select Servers : `VIR-HV-1` and `VIR-HV-2`  
- Cluster Name : `HYPERV`  
- Cluster Adresse : `192.168.18.80`  

Make tests and finish

In **HYPERV.hyperv.dom** -> **Storage** -> **Disk**, choose the cluster available to storage and on the right, clic on `Add to Cluster Shared Volumes`  

Open **HYPERV.hyperv.dom**, right clic on **Roles** and create new virtual machine :  
- Cluster nodes : `VIR-HV-1`  
- VM Name : `Debian`  
- Store in : `C:\ClusterStorage\`  
- Generation : `Generation 1`  
- Ram : `1GB`  
- Size disk : `20GB`  
- OS : `Debian 10`  

Finish  

## Tests
Start VM `Debian`  
Right clic on `Debian` VM and **Move** -> **Live Migration** -> **Select Node**, and choose an other HV.  
After that, check the `Owner Node` on `Debian` has changed  
Shut down the HV where `Debian` is running and check on the other HV that `Debian` `Owner Node` has automatically changed