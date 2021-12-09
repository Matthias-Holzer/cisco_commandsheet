# **Engineering Journal NVS**
##### by Leber Sascha, Holzer Matthias
## Basic
### Different Modes

1. basic terminal
	```
	enable
	```
2. configuration terminal
   	```
	configuration terminal
	```
### Safe the running-config into startup-config
```
!in enable
write	
```
or
```
!in enable
copy running-config startup-config
```

### stop dns lookup
```
!in config t
no ip domain lookup
```

### Increases history size
```
terminal history size <SIZE>
```

### Set console password
```
line console 0
  exec-timeout 0
  password <PASSWORD>
  login
  logging syn
enable secret <PASSWORD>
```

### Telnet
```
line vty 0 4
password <PASSWORD>
```

### SSH
```
ip domain-name <DOMAIN>
hostname <HOSTNAME>
crypto key generate rsa
  <MODULUS>
username <USERNAME> password <PASSWORD>
line vty 0 15
  transport input ssh
  login local
ip ssh [version / time-out / authentication-retries]
```

### Erase settings
```
erase startup
delete vlan.dat
reload
```

## Routing
### Static route
```
ip route <NETWORK> <SUBNET> <IP / INTERFACE>
```

### Deafult route
```
ip route 0.0.0.0 0.0.0.0 <IP / INTERFACE>
```

### Multilayer Switch
> Enable routing
```
ip routing
```

> Enable router port
```
no switchport
```

## Advanced
### VLAN
#### Router

```
interface <INTERFACE>.<VLAN>
encapsulation dot1Q <VLAN>
encapsulation dot1Q <VLAN> native
ip add <IP> <SUBNET>
!
show vlan
show vlan summary
show interface vlan <VLAN>
```

#### Switch  

```
vlan <VLAN>
interface range <INTERFACE RANGE>
switchport mode access
switchport access vlan <VLAN>
interface <INTERFACE>
  switchport trunk native vlan <VLAN>
  switchport trunk allowed vlan <VLAN,VLAN,VLAN>
!
show vlan
show vlan summary
show interface vlan <VLAN>
```

### Port Security
#### Switch  
 * Protect: Blocks Traffic
 * Restrict: Blocks Traffic & Logs Violations
 * Shutdown: disables the port

```
interface <INTERFACE>
switchport port-security mac-address <STICKY / MAC-ADDRESS>
switchport port-security maximum <VLAUE>
switchport port-security violation <PROTECT / RESTRICT / SHUTDOWN>
!
show port-security
```

### OSPF
#### Router
```
router ospf <NUMBER>
  router-id <ID>
  network <NETWORK> <WILDCARD> area <AREA>
  passive-interface <INTERFACE>
  default-information originate
!
show ip ospf neighbors
show ip ospf interface brief
debug ip ospf hello
```

### DHCP
#### Router
##### Server
```
ip dhcp pool <POOL>
  default-router <GATEWAY>
  dns-server <SERVER>
  network <NETWORK> <SUBNET>
ip dhcp excluded-address <FROM IP> <TO IP>
```

##### Client
```
ip helper-address <IP>
```

### NTP
#### Router
```
ntp server <IP>
```

### Syslog
#### Router
```
logging <SERVER IP>
service timestamps log datetime msec
```

### ACL
#### Router
```
ip access-group <ACL> [in / out]
```
##### Standard
###### Numbered
```
access-list [1-99] [permit / deny] <SOURCE IP> <SOURCE WILDCARD>
```

###### Named
```
ip access-list standard <NAME>
    permit <IP> <WILDCARD>
```

##### Extended
###### Numbered
```
access-list [100-199] [permit / deny] <PROTOCOL> <SOURCE IP> <SOURCE WILDCARD> <DEST. IP> <DEST. WILDCARD> eq <PORT>
```

###### Named
```
ip access-list extended <NAME>
  [permit / deny] <PROTOCOL> <SOURCE IP> <SOURCE WILDCARD> <DEST. IP> <DEST. WILDCARD> eq <PORT>
```

##### Reflexive
```
ip access-list extended SURFING
	permit tcp any any reflect tcpstuff_RACL
	permit udp any any reflect udpstuff_RACL timeout 20
ip access-list extended BROWSING
	evaluate tcpstuff_RACL
	evaluate udpstuff_RACL
interface s0/1/0
	description This connects to the Internet
	ip access-group SURFING out
	ip access-group BRWOSING in
```

### NAT
#### Router
```
ip nat [inside / outside]
```
##### Static
```
ip nat inside source static <IP INSIDE LOCAL> <IP INSIDE GLOBAL>
```

##### Dynamic
```
ip nat pool <POOL-NAME> <FROM IP> <TO IP> netmask <SUBNET>
ip access-list standard <ACL-NAME>
  permit <IP> <WILDCARD>
ip nat inside source list <ACL-NAME> pool <POOL-NAME>
```

##### Dynamic NAT overload
```
ip nat pool <POOL-NAME> <FROM IP> <TO IP> netmask <SUBNET>
ip access-list standard <ACL-NAME>
  permit <IP> <WILDCARD>
ip nat inside source list <ACL-NAME> pool <POOL-NAME> overload
```

### VOIP (Voice over IP)
##### on the Switch
```
int range fa0/1 - fa0/24
switchport voice vlan <VLAN>
```
##### on the Router (Model 2811)
```
ip dhcp pool <POOL_NAME>
	network <NETWORK> <SUBNETMASK>
	default router <DEFAULT-GATEWAY>
	option 150 ip <DEFAULT-GATEWAY>

telephony-service
	no auto-reg-ephone
	max-ephones <MAX_EPHONE_NUMBER>
	max-dn <MAX_DN_NUMBER>
	ip source-address <DEFAULT-GATEWAY> port 2000

ephone-dn <DN_NUMBER>
	number <CALL_NUMBER>

ephone <DN_NUMBER>
	mac-address <MAC-ADDRESS_EPHONE>
	type <TYPE> (ata)
	button <RELATION bsp 1:1>


dial-peer voice 1 voip
	destination pattern <PATTERN_OF_NUMBERS bsp 1..>
	session target ipv4:<IP_ROUTER_NEW_NETWORK>
```

## Radius
### on the Server 
##### Activating service aaa
### on the Router
```
aaa new-model

radius-server host <IP-ADDRESS_RADIUS-SERVER> key <RADIUS_SERVER_PASSWORD> 

aaa authentication login default group radius local

line vty 0 15
login authentication default


5am83
```
## HSRP
#### in the Interface
```
standby version 2
standby <NUMBER> ip <IP_VIRTUAL_ROUTER>

standby <NUMBER> priority <PRIORITY>

!!new ellection off active router
standby <group-number> preempt 

standby <group-number> track <INT_TO_INTERNET>
```

## VPN (virtual private Network)
### show 
#### show version
### activate
```
license boot module c1900 technology-package securityk9 (on router_1941)
license boot module c2900 technology-package securityk9 (on router_2911)

yes
do write
do reload
```
### create an ACL
```
access-list <nummer_abouth_100> permit ip <source_ip> <wildcard> <destination_ip> <wildcard> 
```
### IKE Phase 1 (on R1)
#### configure crypto isamkmp
```
crypto isakmp policy <nummer>
	encryption aes 256
	authentication pre-share
	group <highest>
	exit
```
### IKE Phase 2 (on R1)
#### create the transform-set
```
crypto IPSec transform-set VPN-SET-<NAME> esp-aes esp-sha-hmac 
```
#### create crypto MAP
```
crypto map VPN-MAP-<NAME> <nummer> IPSec-isakmp

	description <description>

	set peer <destination>

	set transform-set VPN-SET-<NAME>

	match address <addresse> (access-list)
```
#### bind the MAP to the outgoing interface
```
interface <interface>
	crypto map VPN-MAP-<NAME>
```
### ^^ same on the other side (but other addresses)


## PPP







### IPS
```
license boot module c1900 technology-package securityk9
logging 10.Kn.10.100
do mkdir DIR_IPS
ip ips config location flash:DIR_IPS
ip ips name RULE_IPS
ip ips notify log
```

Enable IPS on interfaces
```
int g0/1
ip ips RULE_IPS out
exit
```

Retire old signautres
```
ip ips signature-category
category all
retired true
exit
```
Unritire ios ips basic
```
category ios_ips basic
retired false
exit
exit
```
unritire and enable icmp request signatures; drop packet and alert
```
ip ips signature-definition
signature 2004 0
status
retired false
enable true
exit
engine
event-action produce-alert
event-action deny-packet-inline
exit
exit
exit
```
show commands
```
show ip ips signatures sigid 2004 subid 0
show ip ips signatures count
show ip ips all
```

## Cheatsheet
### Subnets
> /31	2	255.255.255.254	1/128 <br>
> /30	4	255.255.255.252	1/64 <br>
> /29	8	255.255.255.248	1/32 <br>
> /28	16	255.255.255.240	1/16 <br>
> /27	32	255.255.255.224	1/8 <br>
> /26	64	255.255.255.192	1/4 <br>
> /25	128	255.255.255.128	1/2 <br>
> /24	256	255.255.255.0	1 <br>
> /23	512	255.255.254.0	2 <br>
> /22	1024	255.255.252.0	4 <br>
> /21	2048	255.255.248.0	8 <br>
> /20	4096	255.255.240.0	16 <br>
> /19	8192	255.255.224.0	32 <br>
> /18	16384	255.255.192.0	64 <br>
> /17	32768	255.255.128.0	128 <br>
> /16	65536	255.255.0.0	256 <br>
> /15	131072	255.254.0.0	512 <br>
> /14	262144	255.252.0.0	1024 <br>
> /13	524288	255.248.0.0	2048 <br>
> /12	1048576	255.240.0.0	4096 <br>
> /11	2097152	255.224.0.0	8192 <br>
> /10	4194304	255.192.0.0	16384 <br>
> /9	8388608	255.128.0.0	32768 <br>
> /8	16777216	255.0.0.0	65536 <br>
