# Access Security :
In this lab, we are going to configure a Router on a Stick . where the Router will be a DHCP server, and the Switch is an access switch that contains two VLANs.

![Router on a Stick Topology](images/Router%20on%20a%20Stick.png)

## Router Basic Configuration :
### 1. The creation of DHCP server for each VLAN :

```
(config)# ip dhcp pool <dhcp_pool_name>
(config)# network <dhcp_pool_network> <dhcp_pool_netmask>
(config)# default-router <default_gateway>
```

### 2. The creation of sub-interface for each VLAN :

```
(config)# interface gigaEthernet0/0.<vlan_id> ! it's not mandatory to make it vlan_id
(config-if)# no shutdown
(config-if)# encapsulation dot1Q
(config-if)# ip address <vlan_gateway> <netmask>
```

## Access Switch Basic Configuration :
### 1. Commands to recover the port from *errdisable* :

```
(config)# errdisable recovery cause all 
(config)# errdisable recovery interval 300
```

### 2. The creation of VLANs : 

```
(config)# vlan <vlan_id>
(config-vlan)# name <vlan_name>
```

### 3. Enabling DHCP Snooping :

```
(config)# ip dhcp snooping 
(config)# ip dhcp snooping vlan <vlan_id>
```

### 4. Enabling ARP Inspection :

```
(config)# ip arp inspection vlan <vlan_id>
(config)# ip arp inspection validate dst-mac ip
```

### 5. Removing the DHCP option 82 : 

```
(config)# no ip dhcp snooping information option 
```

### 6. DHCP Snooping database saving :
if the access switch goes down the snooping database will be deleted, so we have to save it.

```
(config)# ip dhcp snooping database flash:/dhcp-snooping-database.txt
(config)# ip dhcp snooping database write-delay <dalay_in_sec> 
```

### 7. DHCP Server's Interface configuration : 

```
(config)# interface fastEthernet0/1
(config-if)# description this link to the dhcp server
(config-if)# switchport mode trunk 
(config-if)# ip dhcp snooping trust
```

### 8. Interfaces configuration of each VLAN : 

```
(config)# interface range fa0/2-12 
(config-if)# description this ports are for end-devices in vlan 10  
(config-if)# no shutdown 
(config-if)# switchport host
(config-if)# switchport mode access 
(config-if)# switchport access vlan 10
(config-if)# spanning-tree portfast
(config-if)# switchport port-security
(config-if)# switchport port-security max 3
(config-if)# switchport port-security violation shutdown vlan
(config-if)# switchport port-security aging type inactivity
(config-if)# switchport port-security aging time 30
(config-if)# ip verify source port-security
(config-if)# ip dhcp snooping limit rate 30
(config-if)# ip arp inspection limit rate 30
(config-if)# storm-control broadcast level 50 30
(config-if)# storm-control multicast level pps 30k 20k
(config-if)# storm-control action trap
(config-if)# storm-control action shutdown
(config-if)# no lldp receive
(config-if)# no lldp transmit 
(config-if)# no cdp enable
```

## Using `IP Verify Source` with `Port Security` : 
### 1. Creation of DHCP class in the Router :

```
(config)# ip dhcp class <class_name>
(config)# relay agent information
```

### 2. Assigning the DHCP class to the DHCP server : 

```
(config)# ip dhcp pool <pool_name>
(config)# class <class_name>
(config)# address range <range_begin> <range_end>
```

### 3. Make the Switch and Router trust end-devices with DHCP option 82 : 
this command will make ip verify source port-security useful and operational.

```
(config)# ip dhcp relay information trust-all
```
