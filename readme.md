
# Networking Tutorial

## Router

Make a linked clone of cent-8-base 

**If anything is not mentioned, do not change**

### **1.** Connecting to VirtualBox Host-Only Ethernet Adapter #2
Under attached to: 
```
Host-only Adapter 
```

Name:
```
VirtualBox Host-Only Ethernet Adapter #2
```

Promiscuous Mode: 
```
Allow all
```

**Nice! Move on to the next step.**

### **1.2.** Creating the new network 

Open a new adapter tab. (e.g Adapter 2, Adapter3 ....)

Attached to:
```
Internal Network
```

Promiscuous Mode:
```
Allow All
```

**You've now successfully created the network your new workstation is going to connect to.**


## **1.3.** Workstation setup
Create another linked clone of centOS_8 and name it accordingly.

Once done, go into the network settings and enable Network Adapter.

Choose Attached to:
```
Internal Network
```
Name: [Name of your network created above in step 1.2] in our case:
```
net3
```

**You are finished step 1, now you will boot up your VMs.**

## Inside your router Virtual Machine (r1,r2,r3....)

### Overview
 - Ip config of interface -> choose an ip (e.g. 10.20.30.[x]/24) 

    - **Tip - use the lower subnet**

- Enable IP forwarding 

- DHCP 

- Bird

- Routing 

## Setting up interfaces in Router VM
**Using ip 192.168.172.0/28** 

### **2.** nmcli command:
```
sudo nmcli con modify ‘System enp0s3’ ipv4.addresses 10.20.30.10/24 ipv4.method manual && sudo nmcli con up ‘System enp0s3’ 
```
**Note: Replace [System enpos3] and the ipv4 address [10.20.30.10/24] with your own interface name! REMEMBER TO ADD **ALL** INTERFACES 'enp0s8, enp0s9, ...' AND SET THEIR IP ADDRESSES!!**

### **2.1.** Adding ip forwarding 
To edit the .conf file, type the command:
```
sudo vim /etc/sysctl.conf 
```
In the bottom of the file, enter **insert** mode and type in:
```
net.ipv4.ip_forward = 1 
```
exit insert mode and save with **:wq**. To check for errors, enter the command:
```
sudo sysctl -–system  
```

Nice! You've finished adding in the interface.

## **3.** DHCP 
To edit the DHCP file, type the command:
```
sudo vim /etc/dhcp/dhcpd.conf 
```

**To calculate subnet mask -> https://jodies.de/ipcalc**

Your subnet will be your Network IP. Example file:
```
subnet 192.168.172.0 netmask 255.255.255.240 { 

option routers 192.168.172.1; 

range 192.168.172.2 192.168.172.14; 

} 
```
**Note: Option router is your interface IP. Make SURE that your range does not include any of your option routers! MAKE SURE ALL INTERFACES ARE MADE!**

Syntax Error checking: 
```
sudo dhcpd –t  
```
Next, you will need to start the service. Type in the command:
```
sudo systemctl start dhcpd 
```
Optional:
```
systemctl status dhcpd 
```

### 3.1. Bird 

**Note: There may already be a default bird file. Make sure to delete it and create a new one.**

Deleting default bird.conf file:
```
sudo rm /etc/bird.conf 
```
Creating and editing the new file:
```
sudo vim /etc/bird.conf 
```

#### Bird Script Example
```
log syslog all: 
router id 10.20.30.10; 
protocol device { 
} 

protocol kernel { 
    ipv4 { 
        export all; 
    }; 
} 

protocol ospf { 
    area 0 { 
        interface “enp0s3” { 
        }; 

        Interface “enp0s8” { 
            stub; 
        };	 
    }; 
} 
```
**Note: Router ID is a unique name for the bird. Remember to stub anything you want invisible. Make sure you know what you are putting in interface!**
 

Check for syntax errors: 
```
sudo bird –p 
```

Start and check the status:
```
sudo systemctl start bird 
```
To check:
```
sudo system status bird 
```
Or
```
ip route 
```

# Start pinging/traceroute. If it doesn't work then check your IP addresses.
