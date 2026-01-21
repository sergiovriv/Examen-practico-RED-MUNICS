#### Zonas
```
conf t
zone security INSIDE
zone security DMZ
zone security OUTSIDE
zone security MGMT
end

```

#### Interfaces a Zonas

```
conf t
interface GigabitEthernet0/0.2
 zone-member security INSIDE
exit

interface GigabitEthernet0/0.740
 zone-member security DMZ
exit

interface GigabitEthernet0/0.739
 zone-member security MGMT
exit

interface GigabitEthernet0/1
 zone-member security OUTSIDE
exit
end

```

#### ACLs de Clasificacion
##### Inside a DMZ
```
conf t
ip access-list extended ACL-INSIDE-TO-DMZ
 permit tcp 10.1.16.0 0.0.3.255 host 10.1.240.100 eq 80
 permit tcp 10.1.16.0 0.0.3.255 host 10.1.240.100 eq 443
 permit udp 10.1.16.0 0.0.3.255 host 10.1.240.100 eq 53
 permit tcp 10.1.16.0 0.0.3.255 host 10.1.240.100 eq 53
 permit icmp 10.1.16.0 0.0.3.255 host 10.1.240.100 echo
end
```
##### Inside a OUTSIDE
```
conf t
ip access-list extended ACL-INSIDE-TO-OUTSIDE
 permit tcp 10.1.16.0 0.0.3.255 any eq 80
 permit tcp 10.1.16.0 0.0.3.255 any eq 443
 permit udp 10.1.16.0 0.0.3.255 any eq 53
 permit tcp 10.1.16.0 0.0.3.255 any eq 53
 permit icmp 10.1.16.0 0.0.3.255 any echo
end
```
##### DMZ a OUTSIDE
```
conf t
ip access-list extended ACL-DMZ-TO-OUTSIDE
 permit tcp 10.1.240.0 0.0.0.255 any eq 80
 permit tcp 10.1.240.0 0.0.0.255 any eq 443
 permit udp 10.1.240.0 0.0.0.255 any eq 53
 permit tcp 10.1.240.0 0.0.0.255 any eq 53
 permit icmp 10.1.240.0 0.0.0.255 any echo
end
```

##### OUTSIDE a DMZ
```
conf t
ip access-list extended ACL-OUTSIDE-TO-DMZ
 remark OUTSIDE -> DMZ server : ONLY HTTPS + ICMP echo
 permit tcp any host 10.1.240.100 eq 443
 permit icmp any host 10.1.240.100 echo
end
```

#### Admin a TODO
```
conf t
ip access-list extended ACL-MGMT-TO-ANY
 permit ip 10.1.239.0 0.0.0.255 any
end
```

#### MGMT a SELF
```
conf t
ip access-list extended ACL-MGMT-TO-SELF
 remark SSH al FW (interfaz de administracion)
 permit tcp 10.1.239.0 0.0.0.255 host 10.1.239.3 eq 22
 permit icmp 10.1.239.0 0.0.0.255 host 10.1.239.3 echo
end
```
#### Class-maps

```
conf t
class-map type inspect match-any CM-INSIDE-TO-DMZ
 match access-group name ACL-INSIDE-TO-DMZ

class-map type inspect match-any CM-INSIDE-TO-OUTSIDE
 match access-group name ACL-INSIDE-TO-OUTSIDE

class-map type inspect match-any CM-DMZ-TO-OUTSIDE
 match access-group name ACL-DMZ-TO-OUTSIDE

class-map type inspect match-any CM-OUTSIDE-TO-DMZ
 match access-group name ACL-OUTSIDE-TO-DMZ

class-map type inspect match-any CM-MGMT-TO-ANY
 match access-group name ACL-MGMT-TO-ANY

class-map type inspect match-any CM-MGMT-TO-SELF
 match access-group name ACL-MGMT-TO-SELF
end

```

#### Policy-maps

```
conf t
policy-map type inspect PM-INSIDE-TO-DMZ
 class type inspect CM-INSIDE-TO-DMZ
  inspect
 class class-default
  drop

policy-map type inspect PM-INSIDE-TO-OUTSIDE
 class type inspect CM-INSIDE-TO-OUTSIDE
  inspect
 class class-default
  drop

policy-map type inspect PM-DMZ-TO-OUTSIDE
 class type inspect CM-DMZ-TO-OUTSIDE
  inspect
 class class-default
  drop

policy-map type inspect PM-OUTSIDE-TO-DMZ
 class type inspect CM-OUTSIDE-TO-DMZ
  inspect
 class class-default
  drop

policy-map type inspect PM-MGMT-TO-ANY
 class type inspect CM-MGMT-TO-ANY
  inspect
 class class-default
  drop

policy-map type inspect PM-MGMT-TO-SELF
 class type inspect CM-MGMT-TO-SELF
  pass
 class class-default
  drop
end

```

#### Zone-Pairs
```
conf t
zone-pair security ZP-INSIDE-TO-DMZ source INSIDE destination DMZ
 service-policy type inspect PM-INSIDE-TO-DMZ

zone-pair security ZP-INSIDE-TO-OUTSIDE source INSIDE destination OUTSIDE
 service-policy type inspect PM-INSIDE-TO-OUTSIDE

zone-pair security ZP-DMZ-TO-OUTSIDE source DMZ destination OUTSIDE
 service-policy type inspect PM-DMZ-TO-OUTSIDE

zone-pair security ZP-OUTSIDE-TO-DMZ source OUTSIDE destination DMZ
 service-policy type inspect PM-OUTSIDE-TO-DMZ

zone-pair security ZP-MGMT-TO-INSIDE source MGMT destination INSIDE
 service-policy type inspect PM-MGMT-TO-ANY

zone-pair security ZP-MGMT-TO-DMZ source MGMT destination DMZ
 service-policy type inspect PM-MGMT-TO-ANY

zone-pair security ZP-MGMT-TO-OUTSIDE source MGMT destination OUTSIDE
 service-policy type inspect PM-MGMT-TO-ANY

zone-pair security ZP-MGMT-TO-SELF source MGMT destination self
 service-policy type inspect PM-MGMT-TO-SELF
end


```
