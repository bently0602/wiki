 
## Add packages 
pkg_add vim 
 
## Firewall 

### Base configuration
vim /etc/pf.conf 
 
Add: (https://www.digitalocean.com/community/tutorials/how-to-configure-packet-filter-pf-on-freebsd-12-1) 
 
set skip on lo0 
 
block all 
 
pass in proto tcp to port { 22 } 
pass out proto { tcp udp } to port { 22 53 80 123 443 } 
 
pass out inet proto icmp icmp-type { echoreq } 
 
\# Port build user does not need network 
block return out log proto {tcp udp} user _pbuild 
 
### Restart PF 
reboot 
 
### Check Rules 
pfctl -s rules 
 
## Updating 

## Update security sources
update sources 
Syspatch –c 

### Update security
Syspatch 

### Package Updates

###
pkg_add -u
