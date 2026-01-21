- Este firewall para todo lo que no es logico, por definicion, que entre en nuestras red corporativa.
#### Definicion de la ACL
```
!ACL simple - filtrado estatico - CPE - entrada desde EXTERIOR

!privadas
access-list 1 deny 10.0.0.0 0.255.255.255
access-list 1 deny 172.16.0.0 0.15.255.255
access-list 1 deny 192.168.0.0 0.0.255.255

!CGNAT
access-list 1 deny 100.64.0.0 0.63.255.255

!enlace local
access-list 1 deny 169.254.0.0 0.0.255.255

!loopback
access-list 1 deny 127.0.0.0 0.255.255.255

!multicast y reservado
access-list 1 deny 224.0.0.0 15.255.255.255
access-list 1 deny 240.0.0.0 15.255.255.255

!broadcast
access-list 1 deny host 255.255.255.255

!testnets (documentación)
access-list 1 deny 192.0.2.0 0.0.0.255
access-list 1 deny 198.51.100.0 0.0.0.255
access-list 1 deny 203.0.113.0 0.0.0.255

!permitir el resto
access-list 1 permit any
```

#### Aplicacion de la ACL
- Aplicada en sentido entrada (IN) sobre g0/1 (CPE-ISP)

```
int g0/1
ip access-group ACL-CPE-STATIC-IN in
```
