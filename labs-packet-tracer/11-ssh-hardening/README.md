# Lab 11 — SSH y Hardening

## Información del lab

| Campo | Detalle |
|-------|---------|
| Tema CCNA | Seguridad — Dominio 5.3 y 5.4 |
| Dificultad | ⭐⭐ Intermedio |
| Duración estimada | 45 minutos |
| Archivo | lab-11-ssh-hardening.pkt |
| Teoría relacionada | [teoria/05-seguridad/acl-ssh-hardening.md](../../teoria/05-seguridad/acl-ssh-hardening.md) |
| Lab anterior | [Lab 10 — ACLs](../10-acl/README.md) |

---

## Objetivo

Configurar SSH versión 2 en un router y un switch Cisco,
deshabilitar Telnet completamente, aplicar hardening básico
eliminando servicios innecesarios y configurar una ACL de
acceso para restringir qué hosts pueden administrar los
dispositivos remotamente.

---

## Escenario

**TechStart S.A.** detectó que sus dispositivos de red están
configurados con Telnet, que envía contraseñas en texto plano.
El equipo de seguridad ordenó migrar todo a SSH versión 2,
endurecer la configuración y restringir el acceso de administración
solo a la red de TI (192.168.20.0/24).

---

## Topología
PC-Admin         PC-TI           PC-Ventas
192.168.20.5     192.168.20.10   192.168.10.10
|               |                |
Fa0/5           Fa0/1           Fa0/2
└───────────────┴────────────────┘
SW1
(G0/1 trunk)
|
G0/0.20 ← VLAN 20 TI
G0/0.10 ← VLAN 10 Ventas
R1
G0/1
|
Servidor-NTP
192.168.99.10

---

## Tabla de direccionamiento

| Dispositivo | Interfaz | Dirección IP | Máscara | Descripción |
|-------------|----------|-------------|---------|-------------|
| R1 | G0/0.10 | 192.168.10.1 | 255.255.255.0 | Gateway Ventas |
| R1 | G0/0.20 | 192.168.20.1 | 255.255.255.0 | Gateway TI |
| R1 | G0/1 | 192.168.99.1 | 255.255.255.0 | Servidores |
| SW1 | VLAN 20 | 192.168.20.2 | 255.255.255.0 | Admin SW1 |
| PC-Admin | NIC | 192.168.20.5 | 255.255.255.0 | Admin de red |
| PC-TI | NIC | 192.168.20.10 | 255.255.255.0 | Usuario TI |
| PC-Ventas | NIC | 192.168.10.10 | 255.255.255.0 | Usuario Ventas |
| Servidor-NTP | NIC | 192.168.99.10 | 255.255.255.0 | Servidor NTP |

---

## Instrucciones

### Parte 1 — Configuración base de SW1

#### Paso 1 — VLANs y puertos
```cisco
Switch> enable
Switch# configure terminal
Switch(config)# hostname SW1
SW1(config)# no ip domain-lookup
SW1(config)# vlan 10
SW1(config-vlan)# name Ventas
SW1(config-vlan)# exit
SW1(config)# vlan 20
SW1(config-vlan)# name TI
SW1(config-vlan)# exit
SW1(config)# interface FastEthernet0/1
SW1(config-if)# switchport mode access
SW1(config-if)# switchport access vlan 20
SW1(config-if)# description PC-TI
SW1(config-if)# spanning-tree portfast
SW1(config-if)# no shutdown
SW1(config-if)# exit
SW1(config)# interface FastEthernet0/2
SW1(config-if)# switchport mode access
SW1(config-if)# switchport access vlan 10
SW1(config-if)# description PC-Ventas
SW1(config-if)# spanning-tree portfast
SW1(config-if)# no shutdown
SW1(config-if)# exit
SW1(config)# interface FastEthernet0/5
SW1(config-if)# switchport mode access
SW1(config-if)# switchport access vlan 20
SW1(config-if)# description PC-Admin
SW1(config-if)# spanning-tree portfast
SW1(config-if)# no shutdown
SW1(config-if)# exit
SW1(config)# interface GigabitEthernet0/1
SW1(config-if)# switchport mode trunk
SW1(config-if)# switchport trunk native vlan 20
SW1(config-if)# switchport trunk allowed vlan 10,20
SW1(config-if)# description Trunk-hacia-R1
SW1(config-if)# no shutdown
SW1(config-if)# exit
SW1(config)# interface vlan 20
SW1(config-if)# ip address 192.168.20.2 255.255.255.0
SW1(config-if)# no shutdown
SW1(config-if)# exit
SW1(config)# ip default-gateway 192.168.20.1
```
---

### Parte 2 — Hardening de SW1

#### Paso 1 — Contraseñas seguras
```cisco
SW1(config)# enable secret HardenSW1#2024
SW1(config)# no enable password
SW1(config)# service password-encryption
SW1(config)# security passwords min-length 10
```

#### Paso 2 — Crear usuario administrador local
```cisco
SW1(config)# username admin privilege 15 secret AdminSW1#2024
SW1(config)# username monitor privilege 5 secret Monitor#2024
```
> Privilege 15 da acceso completo.
> Privilege 5 da acceso limitado solo a comandos show.
> Nunca uses admin como contraseña. Usa contraseñas complejas.

#### Paso 3 — Configurar dominio y generar llaves RSA
```cisco
SW1(config)# ip domain-name techstart.local
SW1(config)# crypto key generate rsa modulus 2048
```

Packet Tracer preguntará el tamaño de la llave.
Escribe 2048 y presiona Enter.

#### Paso 4 — Habilitar SSH versión 2
```cisco
SW1(config)# ip ssh version 2
SW1(config)# ip ssh time-out 60
SW1(config)# ip ssh authentication-retries 3
SW1(config)# ip ssh source-interface vlan 20
```

#### Paso 5 — Configurar líneas VTY solo con SSH
```cisco
SW1(config)# line vty 0 15
SW1(config-line)# transport input ssh
SW1(config-line)# login local
SW1(config-line)# exec-timeout 10 0
SW1(config-line)# logging synchronous
SW1(config-line)# exit
```

> `transport input ssh` bloquea Telnet completamente.
> `login local` usa la base de datos local de usuarios.
> `exec-timeout 10 0` cierra la sesión a los 10 minutos.

#### Paso 6 — Configurar línea de consola
```cisco
SW1(config)# line console 0
SW1(config-line)# login local
SW1(config-line)# exec-timeout 5 0
SW1(config-line)# logging synchronous
SW1(config-line)# exit
```

#### Paso 7 — Banner de advertencia
SW1(config)# banner motd ^
ACCESO SOLO PARA PERSONAL AUTORIZADO
TechStart S.A. — Infraestructura de Red
Toda actividad es monitoreada y registrada
El acceso no autorizado es un delito federal
=======================================================
^

#### Paso 8 — Deshabilitar servicios innecesarios
```cisco
SW1(config)# no ip http server
SW1(config)# no ip http secure-server
SW1(config)# no cdp run
SW1(config)# no ip proxy-arp
```

#### Paso 9 — ACL para restringir acceso de administración
Solo la red de TI puede acceder por SSH al switch:

```cisco
SW1(config)# ip access-list standard ACCESO-ADMIN-SW1
SW1(config-std-nacl)# remark Solo red TI puede administrar SW1
SW1(config-std-nacl)# permit 192.168.20.0 0.0.0.255
SW1(config-std-nacl)# deny any log
SW1(config-std-nacl)# exit
SW1(config)# line vty 0 15
SW1(config-line)# access-class ACCESO-ADMIN-SW1 in
SW1(config-line)# exit
```
> `access-class` en líneas VTY aplica una ACL estándar
> para filtrar quién puede iniciar sesiones remotas.
> `deny any log` registra los intentos bloqueados en syslog.

#### Paso 10 — Guardar configuración
```cisco
SW1(config)# end
SW1# copy running-config startup-config
```

---

### Parte 3 — Configuración base de R1
```cisco
Router> enable
Router# configure terminal
Router(config)# hostname R1
R1(config)# no ip domain-lookup
R1(config)# interface GigabitEthernet0/0
R1(config-if)# no ip address
R1(config-if)# no shutdown
R1(config-if)# exit
R1(config)# interface GigabitEthernet0/0.10
R1(config-subif)# encapsulation dot1Q 10
R1(config-subif)# ip address 192.168.10.1 255.255.255.0
R1(config-subif)# description Gateway-Ventas
R1(config-subif)# exit
R1(config)# interface GigabitEthernet0/0.20
R1(config-subif)# encapsulation dot1Q 20 native
R1(config-subif)# ip address 192.168.20.1 255.255.255.0
R1(config-subif)# description Gateway-TI
R1(config-subif)# exit
R1(config)# interface GigabitEthernet0/1
R1(config-if)# ip address 192.168.99.1 255.255.255.0
R1(config-if)# description Servidores
R1(config-if)# no shutdown
R1(config-if)# exit
```
---

### Parte 4 — Hardening de R1

#### Paso 1 — Contraseñas y seguridad básica
```cisco
R1(config)# enable secret HardenR1#2024
R1(config)# no enable password
R1(config)# service password-encryption
R1(config)# security passwords min-length 10
```

#### Paso 2 — Crear usuarios locales
```cisco
R1(config)# username admin privilege 15 secret AdminR1#2024
R1(config)# username netops privilege 10 secret NetOps#2024
```

#### Paso 3 — Configurar dominio y llaves RSA
```cisco
R1(config)# ip domain-name techstart.local
R1(config)# crypto key generate rsa modulus 2048
```

#### Paso 4 — Habilitar SSH versión 2
```cisco
R1(config)# ip ssh version 2
R1(config)# ip ssh time-out 60
R1(config)# ip ssh authentication-retries 3
R1(config)# ip ssh source-interface GigabitEthernet0/0.20
```

#### Paso 5 — Configurar VTY con SSH y ACL
```cisco
R1(config)# ip access-list standard ACCESO-ADMIN-R1
R1(config-std-nacl)# remark Solo red TI puede administrar R1
R1(config-std-nacl)# permit 192.168.20.0 0.0.0.255
R1(config-std-nacl)# deny any log
R1(config-std-nacl)# exit
R1(config)# line vty 0 4
R1(config-line)# transport input ssh
R1(config-line)# login local
R1(config-line)# exec-timeout 10 0
R1(config-line)# logging synchronous
R1(config-line)# access-class ACCESO-ADMIN-R1 in
R1(config-line)# exit
```

#### Paso 6 — Configurar consola
```cisco
R1(config)# line console 0
R1(config-line)# login local
R1(config-line)# exec-timeout 5 0
R1(config-line)# logging synchronous
R1(config-line)# exit
```

#### Paso 7 — Banner
R1(config)# banner motd ^
ACCESO SOLO PARA PERSONAL AUTORIZADO
TechStart S.A. — Router Principal
Toda actividad es monitoreada y registrada
El acceso no autorizado es un delito federal
=======================================================
^

#### Paso 8 — Deshabilitar servicios innecesarios
```cisco
R1(config)# no ip http server
R1(config)# no ip http secure-server
R1(config)# no cdp run
R1(config)# no ip proxy-arp
R1(config)# no service finger
R1(config)# no service tcp-small-servers
R1(config)# no service udp-small-servers
R1(config)# no ip source-route
R1(config)# no ip gratuitous-arps
```

#### Paso 9 — Configurar NTP
```cisco
R1(config)# ntp server 192.168.99.10
R1(config)# clock timezone CST -6
R1(config)# service timestamps log datetime msec localtime
R1(config)# service timestamps debug datetime msec localtime
```

#### Paso 10 — Configurar Syslog
```cisco
R1(config)# logging host 192.168.99.10
R1(config)# logging trap informational
R1(config)# logging source-interface GigabitEthernet0/0.20
R1(config)# logging on
```

#### Paso 11 — Guardar configuración
```cisco
R1(config)# end
R1# copy running-config startup-config
```
---

## Verificación

### Verificar SSH en R1
```cisco
R1# show ip ssh
```

Resultado esperado:
SSH Enabled - version 2.0
Authentication timeout: 60 secs;
Authentication retries: 3
Minimum expected Diffie Hellman key size : 1024 bits
IOS Keys in SECSH format(ssh-rsa, base64 encoded):
ssh-rsa AAAAB3Nz...

### Verificar sesiones SSH activas
```cisco
R1# show ssh
```

### Verificar las llaves RSA generadas
```cisco
R1# show crypto key mypubkey rsa
```
### Verificar que Telnet esté bloqueado
```cisco
R1# show line vty 0 4
```

Busca:
Allowed input transports are ssh.

Si dice `telnet ssh` hay que corregirlo:
```cisco
R1(config)# line vty 0 4
R1(config-line)# transport input ssh
```

### Verificar la ACL de administración
```cisco
R1# show access-lists ACCESO-ADMIN-R1
R1# show running-config | section line vty
```

Busca:
access-class ACCESO-ADMIN-R1 in

### Verificar NTP
```cisco
R1# show ntp status
R1# show ntp associations
R1# show clock detail
```

Resultado esperado:
Clock is synchronized, stratum 2, reference is 192.168.99.10

### Verificar Syslog
```cisco
R1# show logging
```

Busca:
Trap logging: level informational, 47 message lines logged
Logging to 192.168.99.10

### Pruebas de acceso SSH

#### Desde PC-Admin (debe funcionar)
En Packet Tracer ve a PC-Admin → Desktop → Terminal:
ssh -l admin 192.168.20.1

Cuando pida contraseña escribe: `AdminR1#2024`

Debes ver el banner de advertencia y luego el prompt del router:
=======================================================
ACCESO SOLO PARA PERSONAL AUTORIZADO
...
R1>

#### Desde PC-Admin al switch (debe funcionar)
ssh -l admin 192.168.20.2

#### Desde PC-Ventas (debe FALLAR por ACL)
En Packet Tracer ve a PC-Ventas → Desktop → Terminal:
ssh -l admin 192.168.20.1

Debe aparecer:
% Connection refused by remote host

#### Intentar Telnet (debe FALLAR)
Desde PC-Admin:
telnet 192.168.20.1

Debe aparecer:
% Connection refused by remote host

### Verificar privilegios de usuarios

Conectarte con usuario monitor (privilege 5):
ssh -l monitor 192.168.20.2

Intentar un comando de configuración:
```cisco
SW1> enable
SW1# configure terminal
```
Debe aparecer:
% Authorization failed.

El usuario monitor no tiene acceso al modo de configuración.

### Lista de verificación

- [ ] SSH versión 2 habilitado en R1 y SW1
- [ ] Llaves RSA de 2048 bits generadas en R1 y SW1
- [ ] Telnet bloqueado en líneas VTY de R1 y SW1
- [ ] enable secret configurado en R1 y SW1
- [ ] service password-encryption activo en R1 y SW1
- [ ] Usuarios locales con privilege 15 creados
- [ ] Banner MOTD configurado en R1 y SW1
- [ ] ACL ACCESO-ADMIN aplicada en líneas VTY con access-class
- [ ] Servicios HTTP y CDP deshabilitados
- [ ] NTP sincronizado con el servidor interno
- [ ] Syslog enviando logs al servidor 192.168.99.10
- [ ] SSH desde PC-Admin a R1 funciona correctamente
- [ ] SSH desde PC-Ventas a R1 es rechazado por ACL
- [ ] Telnet desde cualquier PC es rechazado
- [ ] Configuración guardada en R1 y SW1

---

## Troubleshooting

### SSH no conecta aunque está configurado

Verificar que las llaves RSA existen
```cisco
R1# show crypto key mypubkey rsa
```
Si no hay llaves generarlas:
```cisco
R1(config)# crypto key generate rsa modulus 2048
```
Verificar que ip ssh version 2 esté configurado
```cisco
R1# show ip ssh
```cisco
Verificar que el nombre de dominio esté configurado
Sin dominio no se pueden generar llaves RSA
```cisco
R1# show running-config | include domain-name
```
Verificar que las VTY usen login local y no login
```cisco
R1# show running-config | section line vty
```
Debe decir login local, no solo login


### La ACL bloquea incluso a PC-Admin de la red TI

Verificar que PC-Admin tenga IP en la red 192.168.20.0/24
Si tiene IP de otra subred la ACL lo bloqueará
Verificar la ACL
```cisco
R1# show access-lists ACCESO-ADMIN-R1
```
El permit debe ser 192.168.20.0 0.0.0.255
Verificar que access-class esté aplicado en todas las VTY
```cisco
R1# show running-config | section line vty
```

### Error al generar llaves RSA
% Please define a domain-name first.
Solución:

```cisco
R1(config)# ip domain-name techstart.local
R1(config)# crypto key generate rsa modulus 2048
```

### NTP no sincroniza

Verificar conectividad con el servidor NTP
```cisco
R1# ping 192.168.99.10
```
Verificar la configuración NTP
```cisco
R1# show running-config | include ntp
```
Esperar hasta 5 minutos para la sincronización inicial
```cisco
R1# show ntp associations
```
El asterisco (*) indica el servidor activo:
*~192.168.99.10  ... st 1


### show ip ssh muestra SSH disabled
Causa 1: No hay llaves RSA generadas
```cisco
R1(config)# crypto key generate rsa modulus 2048
```
Causa 2: No hay nombre de dominio configurado
```cisco
R1(config)# ip domain-name techstart.local
R1(config)# crypto key generate rsa modulus 2048
```
Causa 3: La versión SSH no está configurada
```cisco
R1(config)# ip ssh version 2
```
---

## Resumen de hardening aplicado

| Medida | Comando | Propósito |
|--------|---------|-----------|
| Contraseña enable cifrada | enable secret | Proteger modo privilegiado |
| Cifrar contraseñas | service password-encryption | Evitar texto plano en config |
| Longitud mínima | security passwords min-length 10 | Contraseñas más seguras |
| SSH v2 | ip ssh version 2 | Comunicación cifrada |
| Llaves RSA 2048 | crypto key generate rsa modulus 2048 | Cifrado fuerte |
| Bloquear Telnet | transport input ssh | Sin texto plano |
| Timeout de sesión | exec-timeout 10 0 | Cerrar sesiones inactivas |
| ACL en VTY | access-class | Restringir origen de acceso |
| Banner legal | banner motd | Advertencia de acceso |
| Deshabilitar HTTP | no ip http server | Reducir superficie de ataque |
| Deshabilitar CDP | no cdp run | No revelar topología |
| NTP | ntp server | Hora correcta para logs |
| Syslog | logging host | Centralizar registros |

---

## Conceptos aplicados en este lab

| Concepto | Dónde se aplica |
|----------|----------------|
| SSH versión 2 | Acceso remoto cifrado a R1 y SW1 |
| Llaves RSA 2048 | Base criptográfica de SSH |
| transport input ssh | Bloqueo de Telnet en VTY |
| login local | Autenticación con base de datos local |
| access-class | ACL aplicada a líneas VTY |
| exec-timeout | Cierre de sesiones inactivas |
| service password-encryption | Cifrado de contraseñas en config |
| enable secret | Contraseña MD5 para modo privilegiado |
| NTP | Sincronización de tiempo para logs |
| Syslog | Centralización de registros de eventos |

---

## Capturas requeridas

| Archivo | Contenido |
|---------|-----------|
| topologia.png | Vista general de la topología |
| r1-show-ip-ssh.png | show ip ssh en R1 con SSH versión 2.0 |
| sw1-show-ip-ssh.png | show ip ssh en SW1 |
| ssh-exitoso-admin.png | Sesión SSH exitosa desde PC-Admin a R1 |
| ssh-fallido-ventas.png | SSH rechazado desde PC-Ventas |
| telnet-fallido.png | Telnet rechazado desde PC-Admin |
| r1-ntp-status.png | show ntp status con Clock is synchronized |
| r1-show-logging.png | show logging con servidor syslog configurado |
| r1-hardening-config.png | show running-config con toda la configuración |
