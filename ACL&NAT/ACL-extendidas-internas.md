
#### Bloqueo inter VLAN
- Cada una de estas reglas se aplica a su interfaz VLAN en DL en sentido IN.
- Se busca denegar todo el trafico que va hacia otras VLANs, dejamos pasar el resto.

```
ip access-list extended ACL-DL-VLAN16-IN
remark BLOQUEO VLANS 17 y 18
 deny ip any 10.1.17.0 0.0.0.255
 deny ip any 10.1.18.0 0.0.0.255
permit ip any any
```

```
ip access-list extended ACL-DL-VLAN17-IN
remark BLOQUEO VLANS 16 y 18
 deny ip any 10.1.16.0 0.0.0.255
 deny ip any 10.1.18.0 0.0.0.255
permit ip any any
```

```
ip access-list extended ACL-DL-VLAN18-IN
remark BLOQUEO VLANS 16 y 17
 deny ip any 10.1.16.0 0.0.0.255
 deny ip any 10.1.17.0 0.0.0.255
permit ip any any
```
##### aplicacion
```
conf t
interface Vlan16
 ip access-group ACL-DL-VLAN16-IN in
exit

interface Vlan17
 ip access-group ACL-DL-VLAN17-IN in
exit

interface Vlan18
 ip access-group ACL-DL-VLAN18-IN in
exit
end
```
#### Filtrado entre VLANS y FW
- Segunda linea de defensa a la entrada del FW.
- Queremos permitir todo HTTP, HTTPS, DNS e ICMP tanto a servicios como hacia internet y denegar el resto.
- Aplicamos ACL extendida direccion IN en interfaz G0/0.2 de FW.

```
ip access-list extended ACL-FW-IN

 remark ACCESO DE DISPOSITIVOS AL SERVIDOR DHCP
 permit udp any host 10.1.239.100 eq bootps
 permit udp any host 10.1.239.100 eq bootpc
 
 remark OSPF
 permit ospf host 10.1.0.1 host 224.0.0.5
 permit ospf host 10.1.0.1 host 10.1.0.2
 permit ospf host 10.1.0.2 host 224.0.0.5
 permit ospf host 10.1.0.2 host 10.1.0.1
 
 remark ACCESO A SERVICIOS VLAN 16-18
 permit tcp 10.1.16.0 0.0.3.255 host 10.1.240.100 eq www
 permit tcp 10.1.16.0 0.0.3.255 host 10.1.240.100 eq 443
 permit tcp 10.1.16.0 0.0.3.255 host 10.1.240.100 eq domain
 permit udp 10.1.16.0 0.0.3.255 host 10.1.240.100 eq domain
 permit icmp 10.1.16.0 0.0.3.255 host 10.1.240.100 echo

 remark BLOQUEAR ACCESO A RED INTERNA
 deny ip any 10.1.0.0 0.0.255.255

 remark ACCESO A INTERNET VLAN 16-18
 permit tcp 10.1.16.0 0.0.3.255 any eq www
 permit tcp 10.1.16.0 0.0.3.255 any eq 443
 permit udp 10.1.16.0 0.0.3.255 any eq domain
 permit icmp 10.1.16.0 0.0.3.255 any echo
 
 deny   ip any any
```

```
    ip inspect name CBAC tcp timeout 300
    ip inspect name CBAC udp timeout 120
    ip inspect name CBAC icmp timeout 60
```

##### Aplicación
```
conf t
interface GigabitEthernet0/0.2
ip access-group ACL-FW-IN in
ip inspect CBAC in
end
```
#### Filtrado entre VLAN servicios (740) y DL-SW1

- La VLAN de servicios solo puede acceder a internet con los protocolos HTTP, HTTPS, ICMP y DNS.
- Hemos de denegar explicitamente la conectividad a nuestras VLANs.
- Se aplica en la interfaz g0/0.740 del FW, en direccion IN.

```
ip access-list extendend ACL-SERVICIOS-IN

remark denegar conexiones vlan
 deny ip any 10.1.0.0 0.0.255.255

remark permitir HTTP, HTTPS, ICMP y DNS
permit tcp 10.1.240.0 0.0.0.255 any eq 80
permit tcp 10.1.240.0 0.0.0.255 any eq 443
permit tcp 10.1.240.0 0.0.0.255 any eq 53
permit udp 10.1.240.0 0.0.0.255 any eq 53
permit icmp 10.1.240.0 0.0.0.255 any

deny ip any any
```

```
interface GigabitEthernet0/0.740
ip access-group ACL-SERVICIOS-IN in
ip inspect CBAC in
```
#### Filtrado de conexion entre internet y FW
- Control de acceso a nuestra red desde internet (penúltima frontera).
- Aplicamos una ACL en la interfaz que comunica FW con CPE (int g0/1).

```
ip access-list extended ACL-INTERNET-IN

permit ospf host 10.1.0.6 host 224.0.0.5
permit ospf host 10.1.0.6 host 10.1.0.5
permit ospf host 10.1.0.5 host 224.0.0.5
permit ospf host 10.1.0.5 host 10.1.0.6

permit tcp any host 10.1.240.100 eq 443
permit icmp any host 10.1.240.100 echo

deny ip any any

```
##### Aplicacion:
```
conf t
interface GigabitEthernet0/1
ip access-group ACL-INTERNET-IN in
ip inspect CBAC in

end
```
