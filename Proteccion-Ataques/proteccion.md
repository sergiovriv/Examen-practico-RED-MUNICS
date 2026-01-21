- Aqui se evitan, uno a uno, los ataques tipicos dados en teoria y en la practica.

#### Defensa contra MAC Flooding
- Aqui evitaremos la saturacion de la tabla CAM de nuestros switches.
- Lo haremos limitando el numero de MAC máximas que se han de aprender por puerto.

```
interface range g0/2 - 10
switchport mode access
switchport port-security
switchport port-security maximum 10
switchport port-security violation protect
```

#### Defensa contra ARP Spoofing
```
! defendemos las vlan acceso
ip arp inspection vlan 16,17,18

! confiamos en conexion con DL
int g0/20
ip arp inspection trust
```

#### Defensa contra CDP Flooding
```
interface range g0/2 - 10
no cdp enable
```

#### Defensa contra DTP Trunking
- los puertos tienen que estar en access (NO dynamic auto).
```
int g0/20
switchport nonegotiate
```

#### Defensa contra DHCP Snooping
```
ip dhcp snooping
no ip dhcp snooping information option
ip dhcp snooping vlan 16,17,18

int g0/20 
ip dhcp snooping trust

int range g0/2 - 10
ip dhcp snooping limit rate 10

show ip dhcp snooping
```

#### Defensa contra saltos inter-VLAN
- Aqui se basa en bajar puertos no usados y asociarles una VLAN sumidero inactiva, asi no pueden negociar troncal.

```
vlan 88
interface range g0/11-19
switchport mode access
switchport access vlan 88
shutdown

interface range g0/21-24
switchport mode access
switchport access vlan 88
shutdown
```

#### Defensa contra ataques a STP

##### 1- AL-SW1
- Activamos portfast y bpduguard en los puertos de acceso para proteger el switch.
```
spanning-tree portfast bpduguard default

int range g0 /2 -10
spanning-tree portfast
exit
```

##### 2- DL-SW1
- Configuramos el DL como root bridge de las VLAN.
```
spanning-tree vlan 16,17,18,739 root primary
end
```
