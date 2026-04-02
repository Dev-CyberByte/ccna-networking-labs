# Lab 01 — Configuración Básica de Dispositivos

## Información del lab

| Campo | Detalle |
|-------|---------|
| Tema CCNA | Fundamentos de red — Dominio 1.x |
| Dificultad | ⭐ Básico |
| Duración estimada | 30 minutos |
| Archivo | lab-01-config-basica.pkt |
| Teoría relacionada | [teoria/01-fundamentos-de-red](../../teoria/01-fundamentos-de-red/) |

---

## Objetivo

Configurar desde cero un router y un switch Cisco aplicando
las buenas prácticas básicas: hostname, contraseñas, banner,
interfaces y conectividad extremo a extremo.

---

## Topología
```
                    192.168.1.0/24
                                        
PC1 ──────── SW1 ──────── R1 ──────── PC2
           Fa0/1  G0/1  G0/0  G0/1
        .10      .1          .1      .20
                        192.168.2.0/24
```

---

## Tabla de direccionamiento

| Dispositivo | Interfaz | Dirección IP | Máscara | Gateway |
|-------------|----------|-------------|---------|---------|
| R1 | G0/0 | 192.168.1.1 | 255.255.255.0 | — |
| R1 | G0/1 | 192.168.2.1 | 255.255.255.0 | — |
| SW1 | VLAN 1 | 192.168.1.2 | 255.255.255.0 | 192.168.1.1 |
| PC1 | NIC | 192.168.1.10 | 255.255.255.0 | 192.168.1.1 |
| PC2 | NIC | 192.168.2.20 | 255.255.255.0 | 192.168.2.1 |

---

## Instrucciones

### Parte 1 — Configuración del Router R1

#### Paso 1 — Entrar al modo de configuración
```
Router> enable
Router# configure terminal
```

#### Paso 2 — Configurar hostname y seguridad básica
```
Router(config)# hostname R1

R1(config)# enable secret Cisco123
R1(config)# service password-encryption
R1(config)# security passwords min-length 8
R1(config)# no ip domain-lookup
```

#### Paso 3 — Configurar banner
```
R1(config)# banner motd ^
================================================
   ACCESO SOLO PARA PERSONAL AUTORIZADO
   Actividad monitoreada y registrada
================================================
^
```

#### Paso 4 — Configurar interfaz G0/0 (LAN izquierda)
```
R1(config)# interface GigabitEthernet0/0
R1(config-if)# description LAN-Izquierda
R1(config-if)# ip address 192.168.1.1 255.255.255.0
R1(config-if)# no shutdown
R1(config-if)# exit
```

#### Paso 5 — Configurar interfaz G0/1 (LAN derecha)
```
R1(config)# interface GigabitEthernet0/1
R1(config-if)# description LAN-Derecha
R1(config-if)# ip address 192.168.2.1 255.255.255.0
R1(config-if)# no shutdown
R1(config-if)# exit
```

#### Paso 6 — Configurar línea de consola
```
R1(config)# line console 0
R1(config-line)# password ConsolaPass
R1(config-line)# login
R1(config-line)# exec-timeout 5 0
R1(config-line)# logging synchronous
R1(config-line)# exit
```

#### Paso 7 — Configurar líneas VTY
```
R1(config)# line vty 0 4
R1(config-line)# password VtyPass123
R1(config-line)# login
R1(config-line)# exec-timeout 10 0
R1(config-line)# exit
```

#### Paso 8 — Guardar la configuración
```
R1(config)# end
R1# copy running-config startup-config
```

---

### Parte 2 — Configuración del Switch SW1

#### Paso 1 — Configuración básica
```
Switch> enable
Switch# configure terminal

Switch(config)# hostname SW1
SW1(config)# enable secret Cisco123
SW1(config)# service password-encryption
SW1(config)# no ip domain-lookup
```

#### Paso 2 — Configurar interfaz de administración (VLAN 1)
```
SW1(config)# interface vlan 1
SW1(config-if)# description Interfaz-de-Administracion
SW1(config-if)# ip address 192.168.1.2 255.255.255.0
SW1(config-if)# no shutdown
SW1(config-if)# exit

SW1(config)# ip default-gateway 192.168.1.1
```

#### Paso 3 — Configurar banner y líneas
```
SW1(config)# banner motd ^ACCESO RESTRINGIDO^

SW1(config)# line console 0
SW1(config-line)# password ConsolaPass
SW1(config-line)# login
SW1(config-line)# exec-timeout 5 0
SW1(config-line)# logging synchronous
SW1(config-line)# exit

SW1(config)# line vty 0 15
SW1(config-line)# password VtyPass123
SW1(config-line)# login
SW1(config-line)# exec-timeout 10 0
SW1(config-line)# exit
```

#### Paso 4 — Guardar la configuración
```
SW1(config)# end
SW1# copy running-config startup-config
```

---

### Parte 3 — Configuración de las PCs

#### PC1
```
IP Address:      192.168.1.10
Subnet Mask:     255.255.255.0
Default Gateway: 192.168.1.1
```

#### PC2
```
IP Address:      192.168.2.20
Subnet Mask:     255.255.255.0
Default Gateway: 192.168.2.1
```

---

## Verificación

### En el router R1

#### Verificar interfaces
```
R1# show ip interface brief
```

Resultado esperado:
```
Interface         IP-Address      OK? Method Status    Protocol
GigabitEthernet0/0 192.168.1.1   YES manual up        up
GigabitEthernet0/1 192.168.2.1   YES manual up        up
```

#### Verificar configuración
```
R1# show running-config
R1# show version
```

### En el switch SW1

#### Verificar interfaz de administración
```
SW1# show ip interface brief
SW1# show interfaces vlan 1
```

### Pruebas de conectividad

Desde PC1 hacer ping a todos los dispositivos:
```
ping 192.168.1.1     ← Gateway de PC1 (R1 G0/0)
ping 192.168.1.2     ← Switch SW1
ping 192.168.2.1     ← R1 G0/1
ping 192.168.2.20    ← PC2 (prueba extremo a extremo)
```

Desde PC2:
```
ping 192.168.2.1     ← Gateway de PC2
ping 192.168.1.10    ← PC1 (prueba extremo a extremo)
```

### Lista de verificación

- [ ] R1 tiene hostname configurado
- [ ] R1 tiene enable secret activo
- [ ] R1 G0/0 está up/up con IP correcta
- [ ] R1 G0/1 está up/up con IP correcta
- [ ] SW1 tiene IP de administración
- [ ] SW1 tiene default-gateway configurado
- [ ] PC1 hace ping a PC2 exitosamente
- [ ] PC2 hace ping a PC1 exitosamente
- [ ] Configuración guardada en startup-config

---

## Troubleshooting

### Ping falla entre PC1 y PC2
```
1. Verificar que las interfaces del router estén up/up
   R1# show ip interface brief

2. Verificar que las IPs y gateways de las PCs sean correctos
   PC1> ipconfig

3. Verificar que no haya typos en las IPs del router
   R1# show running-config | section interface

4. Probar ping desde el router hacia cada PC
   R1# ping 192.168.1.10
   R1# ping 192.168.2.20
```

### Interfaz aparece administratively down
```
R1(config)# interface GigabitEthernet0/0
R1(config-if)# no shutdown
```

### El switch no responde por Telnet/VTY
```
1. Verificar que la VLAN 1 esté up
   SW1# show interfaces vlan 1

2. Verificar que el default-gateway esté configurado
   SW1# show ip default-gateway

3. Verificar que las líneas VTY tengan contraseña
   SW1# show running-config | section line vty
```

---

## Conceptos aplicados en este lab

| Concepto | Dónde se aplica |
|----------|----------------|
| Modos de IOS | Enable, configure terminal, interface |
| Seguridad básica | enable secret, service password-encryption |
| Interfaces | ip address, no shutdown, description |
| Administración remota | line vty, line console, exec-timeout |
| Interfaz de switch | interface vlan 1, ip default-gateway |
| Guardar configuración | copy running-config startup-config |
