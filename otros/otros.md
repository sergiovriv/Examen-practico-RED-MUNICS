### Establecer acceso al server DHCP desde DL-SW1

```
conf t

interface Vlan16
ip helper-address 10.1.239.100

interface Vlan17
ip helper-address 10.1.239.100

interface Vlan18
ip helper-address 10.1.239.100
 
exit
```
