#### Definimos que es outside e inside en las interfaces

```
conf t
int g0/0.3
ip nat inside
int g0/1
ip nat outside
end
```

#### Port Adress Translation

```
! definimos las ACLS para cada VLAN
conf t
ip access-list standard NAT-VLAN16
permit 10.1.16.0 0.0.0.255
ip access-list standard NAT-VLAN17
permit 10.1.17.0 0.0.0.255
ip access-list standard NAT-VLAN18
permit 10.1.18.0 0.0.0.255

!Configuramos el pooling de direcciones

conf t
ip nat pool Vlan16 192.0.1.200 192.0.1.200 netmask 255.255.255.0
ip nat pool Vlan17 192.0.1.201 192.0.1.201 netmask 255.255.255.0
ip nat pool Vlan18 192.0.1.202 192.0.1.202 netmask 255.255.255.0
end

!Activamos el PAT

conf t
ip nat inside source list NAT-VLAN16 pool Vlan16 overload
ip nat inside source list NAT-VLAN17 pool Vlan17 overload
ip nat inside source list NAT-VLAN18 pool Vlan18 overload
end
```

#### NAT estático de puerto
- Esto era para HTTPS solamente.

```
conf t
ip nat inside source static tcp 10.1.240.100 443 192.0.1.203 443 extendable
end
```
