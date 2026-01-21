- Esto se hace en cada uno de los dispositivos de red.
- El servidor RADIUS se encuentra en la máquina de administración. 
- Existen dos variantes para el server RADIUS, la Legacy (vieja) y la normal.
- Cuidado con minimos de contraseña e intentos maximos (aqui estan exagerados por eso), no queremos quedarnos fuera del examen por bobadas del estilo.

#### 0.  Hardening Básico & Comandos de SSH & Config del Radius

##### Hardening Inicial
```
no ip domain-lookup
service password-encryption
security passwords min-length X
login on-failure log
login on-success log
```

##### Comandos de SSH

```

ssh -oHostKeyAlgorithms=+ssh-rsa     -oKexAlgorithms=+diffie-hellman-group1-sha1     -oCiphers=+aes128-cbc,aes192-cbc,aes256-cbc,3des-cbc     superMunics@10.1.239.X

ssh -oHostKeyAlgorithms=+ssh-rsa -oKexAlgorithms=+diffie-hellman-group14-sha1 superMunics@10.1.239.X  

```


##### Config del Radius
```
archivo: cat /etc/freeradius/3.0/clients.conf

client vlan239{
  ipaddr = 10.1.239.100
  secret = Bayern_2025
  require_message_authenticator = true

}


client AL-SW1 {
  ipaddr = 10.1.239.1
  secret = Bayern_2025
  require_message_authenticator = no
  limit_proxy_state = no
}
client DL-SW1 {
  ipaddr = 10.1.239.2
  secret = Bayern_2025
  require_message_authenticator = no
  limit_proxy_state = no
}
client FW {
  ipaddr = 10.1.239.3
  secret = Bayern_2025
  require_message_authenticator = no
  limit_proxy_state = no
 }

client CPE {
  ipaddr = 10.1.239.4
  secret = Bayern_2025
  require_message_authenticator = no
  limit_proxy_state = no
}

client ISP {
  ipaddr = 10.1.239.5
  secret = Bayern_2025
  require_message_authenticator = no
  limit_proxy_state = no
}
```

```
Diagnostico RADIUS desde maquina admin

root@municsPOD:/etc/freeradius/3.0/mods-config/files# 

radtest -x superMunics  Bayern_2025 127.0.0.1 0 Bayern_2025  

Sent Access-Request Id 116 from 0.0.0.0:35495 to 127.0.0.1:1812 length 81
        User-Name = "superMunics"
        User-Password = "Bayern_2025"
        NAS-IP-Address = 10.254.164.31
        NAS-Port = 0
        Message-Authenticator = 0x00
        Cleartext-Password = "Bayern_2025"

```

#### 1.1 Política de AAA (LEGACY)

```

! activacion del AAA
aaa new-model

! crear user básico
username BASICO secret PASSWORD

! crear user privilegiado (no tiene que hacer enable)
username ADMIN privilege 15 secret CONTRASENA


! definicion de server radius (LEGACY)
radius-server host 10.1.239.X auth-port 1812 acct-port 1813
radius-server key Bayern_2025

! definicion de server radius (MODERNO)
radius server AAA-MUNICS
 address ipv4 10.1.239.X auth-port 1812 acct-port 1813
 key Bayern_2025
exit

! definimos orden AA
aaa authentication login default group radius local
aaa authorization exec default group radius local


```

#### 1.2 Politica de AAA (MODERNA)
```

! activacion del AAA
aaa new-model

! crear user básico
username BASICO secret PASSWORD

! crear user privilegiado (no tiene que hacer enable)
username ADMIN privilege 15 secret CONTRASENA

! definicion de server radius (MODERNO)
radius server AAA-MUNICS
 address ipv4 10.1.239.X auth-port 1812 acct-port 1813
 key Bayern_2025
exit

! definimos grupo de servers radius
aaa group server radius AAA-GROUP
 server name AAA-MUNICS
exit

! definimos orden AA
aaa authentication login default group AAA-GROUP local
aaa authorization exec default group AAA-GROUP local
```


#### 2. Configuramos SSH
```
ip domain-name munics.pri
crypto key generate rsa modulus 1024
ip ssh version 2
ip ssh authentication-retries 10
ip ssh time-out 240
```

#### 3. Aplicamos AAA & SSH por entradas consola y VTY

```
! Definimos la ACL de consola para que solo pueda entrar desde VLAN 739

ip access-list standard MGMT-VLAN
remark solo gestion desde VLAN 739
permit 10.1.239.0 0.0.0.255
deny any log
exit

! Aplicamos en linea de consola
line console 0
login authentication default

! Aplicamos en VTY
line vty 0 4
login authentication default
transport imput ssh
access-class MGMT-VLAN in
end

```
