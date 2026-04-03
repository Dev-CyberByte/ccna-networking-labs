# Lab 03 — VLANs y Trunking

## Información del lab

| Campo | Detalle |
|-------|---------|
| Tema CCNA | Acceso de red — Dominio 2.1 y 2.2 |
| Dificultad | ⭐⭐ Intermedio |
| Duración estimada | 45 minutos |
| Archivo | lab-03-vlans.pkt |
| Teoría relacionada | [teoria/02-acceso-de-red/vlans-trunking-stp.md](../../teoria/02-acceso-de-red/vlans-trunking-stp.md) |

---

## Objetivo

Crear y configurar VLANs en múltiples switches, configurar puertos
de acceso y trunking entre switches, y verificar que el tráfico
de cada VLAN esté correctamente segmentado.

---

## Escenario

La empresa **TechStart S.A.** tiene una oficina con tres departamentos.
Todos comparten la misma infraestructura física de switches pero deben
estar completamente separados a nivel lógico.

| VLAN | Nombre | Departamento | Color en PT |
|------|--------|-------------|-------------|
| 10 | Ventas | Área comercial | Verde |
| 20 | TI | Área técnica | Azul |
| 30 | RRHH | Recursos humanos | Rojo |
| 99 | Administracion | Gestión de switches | Amarillo |

---

## Topología
```
PC-Ventas1    PC-TI1    PC-RRHH1        PC-Ventas2    PC-TI2    PC-RRHH2
    │             │          │                │             │          │
  Fa0/1        Fa0/2      Fa0/3            Fa0/1         Fa0/2      Fa0/3
    └────────────┴──────────┘                └─────────────┴──────────┘
               SW1                                        SW2
           (G0/1 trunk) ──────────────────────────── (G0/1 trunk)

SW1 VLAN 99: 192.168.99.1
SW2 VLAN 99: 192.168.99.2
```

---

## Tabla de direccionamiento

| Dispositivo | VLAN | Dirección IP | Máscara | Gateway |
|-------------|------|-------------|---------|---------|
| SW1 | 99 | 192.168.99.1 | 255.255.255.0 | 192.168.99.254 |
| SW2 | 99 | 192.168.99.2 | 255.255.255.0 | 192.168.99.254 |
| PC-Ventas1 | 10 | 192.168.10.10 | 255.255.255.0 | 192.168.10.1 |
| PC-Ventas2 | 10 | 192.168.10.11 | 255.255.255.0 | 192.168.10.1 |
| PC-TI1 | 20 | 192.168.20.10 | 255.255.255.0 | 192.168.20.1 |
| PC-TI2 | 20 | 192.168.20.11 | 255.255.255.0 | 192.168.20.1 |
| PC-RRHH1 | 30 | 192.168.30.10 | 255.255.255.0 | 192.168.30.1 |
| PC-RRHH2 | 30 | 192.168.30.11 | 255.255.255.0 | 192.168.30.1 |

---

## Instrucciones

### Parte 1 — Configuración de SW1

#### Paso 1 — Configuración básica
```
Switch> enable
Switch# configure terminal
Switch(config)# hostname SW1
SW1(config)# enable secret Cisco123
SW1(config)# service password-encryption
SW1(config)# no ip domain-lookup
```

#### Paso 2 — Crear las VLANs
```
SW1(config)# vlan 10
SW1(config-vlan)# name Ventas
SW1(config-vlan)# exit

SW1(config)# vlan 20
SW1(config-vlan)# name TI
SW1(config-vlan)# exit

SW1(config)# vlan 30
SW1(config-vlan)# name RRHH
SW1(config-vlan)# exit

SW1(config)# vlan 99
SW1(config-vlan)# name Administracion
SW1(config-vlan)# exit
```

#### Paso 3 — Configurar puertos de acceso
```
SW1(config)# interface FastEthernet0/1
SW1(config-if)# description PC-Ventas1
SW1(config-if)# switchport mode access
SW1(config-if)# switchport access vlan 10
SW1(config-if)# no shutdown
SW1(config-if)# exit

SW1(config)# interface FastEthernet0/2
SW1(config-if)# description PC-TI1
SW1(config-if)# switchport mode access
SW1(config-if)# switchport access vlan 20
SW1(config-if)# no shutdown
SW1(config-if)# exit

SW1(config)# interface FastEthernet0/3
SW1(config-if)# description PC-RRHH1
SW1(config-if)# switchport mode access
SW1(config-if)# switchport access vlan 30
SW1(config-if)# no shutdown
SW1(config-if)# exit
```

#### Paso 4 — Configurar puerto trunk hacia SW2
```
SW1(config)# interface GigabitEthernet0/1
SW1(config-if)# description Trunk-hacia-SW2
SW1(config-if)# switchport mode trunk
SW1(config-if)# switchport trunk native vlan 99
SW1(config-if)# switchport trunk allowed vlan 10,20,30,99
SW1(config-if)# no shutdown
SW1(config-if)# exit
```

#### Paso 5 — Configurar interfaz de administración
```
SW1(config)# interface vlan 99
SW1(config-if)# description Administracion
SW1(config-if)# ip address 192.168.99.1 255.255.255.0
SW1(config-if)# no shutdown
SW1(config-if)# exit

SW1(config)# ip default-gateway 192.168.99.254
```

#### Paso 6 — Guardar configuración
```
SW1(config)# end
SW1# copy running-config startup-config
```

---

### Parte 2 — Configuración de SW2

#### Paso 1 — Configuración básica
```
Switch> enable
Switch# configure terminal
Switch(config)# hostname SW2
SW2(config)# enable secret Cisco123
SW2(config)# service password-encryption
SW2(config)# no ip domain-lookup
```

#### Paso 2 — Crear las VLANs
```
SW2(config)# vlan 10
SW2(config-vlan)# name Ventas
SW2(config-vlan)# exit

SW2(config)# vlan 20
SW2(config-vlan)# name TI
SW2(config-vlan)# exit

SW2(config)# vlan 30
SW2(config-vlan)# name RRHH
SW2(config-vlan)# exit

SW2(config)# vlan 99
SW2(config-vlan)# name Administracion
SW2(config-vlan)# exit
```

#### Paso 3 — Configurar puertos de acceso
```
SW2(config)# interface FastEthernet0/1
SW2(config-if)# description PC-Ventas2
SW2(config-if)# switchport mode access
SW2(config-if)# switchport access vlan 10
SW2(config-if)# no shutdown
SW2(config-if)# exit

SW2(config)# interface FastEthernet0/2
SW2(config-if)# description PC-TI2
SW2(config-if)# switchport mode access
SW2(config-if)# switchport access vlan 20
SW2(config-if)# no shutdown
SW2(config-if)# exit

SW2(config)# interface FastEthernet0/3
SW2(config-if)# description PC-RRHH2
SW2(config-if)# switchport mode access
SW2(config-if)# switchport access vlan 30
SW2(config-if)# no shutdown
SW2(config-if)# exit
```

#### Paso 4 — Configurar puerto trunk hacia SW1
```
SW2(config)# interface GigabitEthernet0/1
SW2(config-if)# description Trunk-hacia-SW1
SW2(config-if)# switchport mode trunk
SW2(config-if)# switchport trunk native vlan 99
SW2(config-if)# switchport trunk allowed vlan 10,20,30,99
SW2(config-if)# no shutdown
SW2(config-if)# exit
```

#### Paso 5 — Configurar interfaz de administración
```
SW2(config)# interface vlan 99
SW2(config-if)# description Administracion
SW2(config-if)# ip address 192.168.99.2 255.255.255.0
SW2(config-if)# no shutdown
SW2(config-if)# exit

SW2(config)# ip default-gateway 192.168.99.254
```

#### Paso 6 — Guardar configuración
```
SW2(config)# end
SW2# copy running-config startup-config
```

---

### Parte 3 — Configuración de las PCs

Configura cada PC con la IP, máscara y gateway de la tabla
de direccionamiento. En Packet Tracer ve a Desktop → IP Configuration.

> Nota: el gateway apunta al router que harás en el lab 04.
> En este lab las PCs de distintas VLANs NO deben comunicarse entre sí.
> Eso es correcto y es exactamente lo que se quiere verificar.

---

## Verificación

### Verificar VLANs creadas
```
SW1# show vlan brief
```

Resultado esperado:
```
VLAN Name                Status    Ports
---- -------------------- --------- ----------------------------
1    default              active
10   Ventas               active    Fa0/1
20   TI                   active    Fa0/2
30   RRHH                 active    Fa0/3
99   Administracion       active
1002 fddi-default         active
1003 token-ring-default   active
1004 fddinet-default      active
1005 trnet-default        active
```

### Verificar el trunk
```
SW1# show interfaces trunk
```

Resultado esperado:
```
Port      Mode         Encapsulation  Status        Native vlan
Gi0/1     on           802.1q         trunking      99

Port      Vlans allowed on trunk
Gi0/1     10,20,30,99

Port      Vlans allowed and active in management domain
Gi0/1     10,20,30,99

Port      Vlans in spanning tree forwarding state and not pruned
Gi0/1     10,20,30,99
```

### Verificar un puerto de acceso
```
SW1# show interfaces FastEthernet0/1 switchport
```

Resultado esperado:
```
Name: Fa0/1
Switchport: Enabled
Administrative Mode: static access
Operational Mode: static access
Administrative Trunking Encapsulation: dot1q
Access Mode VLAN: 10 (Ventas)
```

### Pruebas de conectividad

#### Pruebas que DEBEN funcionar (misma VLAN)
```
PC-Ventas1 ping 192.168.10.11   ← PC-Ventas2  ✓ debe funcionar
PC-TI1     ping 192.168.20.11   ← PC-TI2      ✓ debe funcionar
PC-RRHH1   ping 192.168.30.11   ← PC-RRHH2    ✓ debe funcionar
```

#### Pruebas que NO deben funcionar (diferente VLAN)
```
PC-Ventas1 ping 192.168.20.10   ← PC-TI1      ✗ debe fallar
PC-Ventas1 ping 192.168.30.10   ← PC-RRHH1    ✗ debe fallar
PC-TI1     ping 192.168.30.10   ← PC-RRHH1    ✗ debe fallar
```

> Si el ping entre VLANs distintas falla, el lab está correcto.
> La comunicación entre VLANs requiere un router, que verás en el lab 04.

### Lista de verificación

- [ ] VLANs 10, 20, 30 y 99 creadas en SW1 y SW2
- [ ] Puertos Fa0/1, Fa0/2 y Fa0/3 en modo access en ambos switches
- [ ] Puerto G0/1 en modo trunk en SW1 y SW2
- [ ] VLAN nativa 99 configurada en el trunk
- [ ] VLANs 10, 20, 30 y 99 permitidas en el trunk
- [ ] PC-Ventas1 hace ping a PC-Ventas2 correctamente
- [ ] PC-TI1 hace ping a PC-TI2 correctamente
- [ ] PC-RRHH1 hace ping a PC-RRHH2 correctamente
- [ ] Ping entre VLANs distintas falla correctamente
- [ ] Configuración guardada en SW1 y SW2

---

## Troubleshooting

### Ping falla entre PCs de la misma VLAN
```
1. Verificar que ambas PCs estén en la misma VLAN
   SW1# show vlan brief

2. Verificar que el puerto esté en modo access con la VLAN correcta
   SW1# show interfaces FastEthernet0/1 switchport

3. Verificar que el trunk esté activo y permita la VLAN
   SW1# show interfaces trunk

4. Verificar IPs y máscaras en las PCs
   Las PCs de la misma VLAN deben estar en la misma subred
```

### El trunk no está activo
```
1. Verificar que ambos extremos estén en modo trunk
   SW1# show interfaces GigabitEthernet0/1 switchport
   SW2# show interfaces GigabitEthernet0/1 switchport

2. Si un extremo está en modo access cambiar a trunk
   SW1(config)# interface GigabitEthernet0/1
   SW1(config-if)# switchport mode trunk

3. Verificar que la VLAN nativa coincida en ambos switches
   Debe ser VLAN 99 en los dos extremos
```

### Una VLAN no aparece en show vlan brief
```
1. La VLAN no fue creada, crearla manualmente
   SW1(config)# vlan 10
   SW1(config-vlan)# name Ventas

2. Verificar que el puerto esté asignado a esa VLAN
   SW1(config)# interface FastEthernet0/1
   SW1(config-if)# switchport access vlan 10
```

---

## Conceptos aplicados en este lab

| Concepto | Dónde se aplica |
|----------|----------------|
| VLANs | Segmentación lógica de departamentos |
| Puerto de acceso | Conexión de PCs a una VLAN específica |
| Puerto trunk | Enlace entre switches llevando múltiples VLANs |
| 802.1Q | Encapsulación usada en el trunk |
| VLAN nativa | VLAN 99 sin etiqueta en el trunk |
| VLAN de administración | VLAN 99 para gestionar los switches |
| Dominio de broadcast | Cada VLAN es su propio dominio |


