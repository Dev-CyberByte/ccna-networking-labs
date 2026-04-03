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
PC-Ventas1    PC-TI1    PC-RRHH1        PC-Ventas2    PC-TI2    PC-RRHH2
│             │          │                │             │          │
Fa0/1        Fa0/2      Fa0/3            Fa0/1         Fa0/2      Fa0/3
└────────────┴──────────┘                └─────────────┴──────────┘
SW1                                        SW2
(G0/1 trunk) ──────────────────────────── (G0/1 trunk)
SW1 VLAN 99: 192.168.99.1
SW2 VLAN 99: 192.168.99.2

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
Switch> enable
Switch# configure terminal
Switch(config)# hostname SW1
SW1(config)# enable secret Cisco123
SW1(config)# service password-encryption
SW1(config)# no ip domain-lookup

#### Paso 2 — Crear las VLANs
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

#### Paso 3 — Configurar puertos de acceso
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

#### Paso 4 — Configurar puerto trunk hacia SW2
SW1(config)# interface GigabitEthernet0/1
SW1(config-if)# description Trunk-hacia-SW2
SW1(config-if)# switchport mode trunk
SW1(config-if)# switchport trunk native vlan 99
SW1(config-if)# switchport trunk allowed vlan 10,20,30,99
SW1(config-if)# no shutdown
SW1(config-if)# exit

#### Paso 5 — Configurar interfaz de administración
SW1(config)# interface vlan 99
SW1(config-if)# description Administracion
SW1(config-if)# ip address 192.168.99.1 255.255.255.0
SW1(config-if)# no shutdown
SW1(config-if)# exit
SW1(config)# ip default-gateway 192.168.99.254

#### Paso 6 — Guardar configuración
SW1(config)# end
SW1# copy running-config startup-config

---

### Parte 2 — Configuración de SW2

#### Paso 1 — Configuración básica
Switch> enable
Switch# configure terminal
Switch(config)# hostname SW2
SW2(config)# enable secret Cisco123
SW2(config)# service password-encryption
SW2(config)# no ip domain-lookup

#### Paso 2 — Crear las VLANs
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

#### Paso 3 — Configurar puertos de acceso
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

#### Paso 4 — Configurar puerto trunk hacia SW1
SW2(config)# interface GigabitEthernet0/1
SW2(config-if)# description Trunk-hacia-SW1
SW2(config-if)# switchport mode trunk
SW2(config-if)# switchport trunk native vlan 99
SW2(config-if)# switchport trunk allowed vlan 10,20,30,99
SW2(config-if)# no shutdown
SW2(config-if)# exit

#### Paso 5 — Configurar interfaz de administración
SW2(config)# interface vlan 99
SW2(config-if)# description Administracion
SW2(config-if)# ip address 192.168.99.2 255.255.255.0
SW2(config-if)# no shutdown
SW2(config-if)# exit
SW2(config)# ip default-gateway 192.168.99.254

#### Paso 6 — Guardar configuración
SW2(config)# end
SW2# copy running-config startup-config

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
SW1# show vlan brief

Resultado esperado:
VLAN Name                Status    Ports

1    default              active
10   Ventas               active    Fa0/1
20   TI                   active    Fa0/2
30   RRHH                 active    Fa0/3
99   Administracion       active
1002 fddi-default         active
1003 token-ring-default   active
1004 fddinet-default      active
1005 trnet-default        active

### Verificar el trunk
SW1# show interfaces trunk

Resultado esperado:
Port      Mode         Encapsulation  Status        Native vlan
Gi0/1     on           802.1q         trunking      99
Port      Vlans allowed on trunk
Gi0/1     10,20,30,99
Port      Vlans allowed and active in management domain
Gi0/1     10,20,30,99
Port      Vlans in spanning tree forwarding state and not pruned
Gi0/1     10,20,30,99

### Verificar un puerto de acceso
SW1# show interfaces FastEthernet0/1 switchport

Resultado esperado:
Name: Fa0/1
Switchport: Enabled
Administrative Mode: static access
Operational Mode: static access
Administrative Trunking Encapsulation: dot1q
Access Mode VLAN: 10 (Ventas)

### Pruebas de conectividad

#### Pruebas que DEBEN funcionar (misma VLAN)
PC-Ventas1 ping 192.168.10.11   ← PC-Ventas2  ✓ debe funcionar
PC-TI1     ping 192.168.20.11   ← PC-TI2      ✓ debe funcionar
PC-RRHH1   ping 192.168.30.11   ← PC-RRHH2    ✓ debe funcionar

#### Pruebas que NO deben funcionar (diferente VLAN)
PC-Ventas1 ping 192.168.20.10   ← PC-TI1      ✗ debe fallar
PC-Ventas1 ping 192.168.30.10   ← PC-RRHH1    ✗ debe fallar
PC-TI1     ping 192.168.30.10   ← PC-RRHH1    ✗ debe fallar

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

Verificar que ambas PCs estén en la misma VLAN
SW1# show vlan brief
Verificar que el puerto esté en modo access con la VLAN correcta
SW1# show interfaces FastEthernet0/1 switchport
Verificar que el trunk esté activo y permita la VLAN
SW1# show interfaces trunk
Verificar IPs y máscaras en las PCs
Las PCs de la misma VLAN deben estar en la misma subred


### El trunk no está activo

Verificar que ambos extremos estén en modo trunk
SW1# show interfaces GigabitEthernet0/1 switchport
SW2# show interfaces GigabitEthernet0/1 switchport
Si un extremo está en modo access cambiar a trunk
SW1(config)# interface GigabitEthernet0/1
SW1(config-if)# switchport mode trunk
Verificar que la VLAN nativa coincida en ambos switches
Debe ser VLAN 99 en los dos extremos


### Una VLAN no aparece en show vlan brief

La VLAN no fue creada, crearla manualmente
SW1(config)# vlan 10
SW1(config-vlan)# name Ventas
Verificar que el puerto esté asignado a esa VLAN
SW1(config)# interface FastEthernet0/1
SW1(config-if)# switchport access vlan 10


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

---

## Capturas requeridas

| Archivo | Contenido |
|---------|-----------|
| topologia.png | Vista general de la topología en Packet Tracer |
| sw1-vlan-brief.png | show vlan brief en SW1 |
| sw2-vlan-brief.png | show vlan brief en SW2 |
| sw1-trunk.png | show interfaces trunk en SW1 |
| ping-misma-vlan.png | Ping exitoso entre PCs de la misma VLAN |
| ping-diferente-vlan.png | Ping fallido entre PCs de VLANs distintas |

Commit
bashgit add labs-packet-tracer/03-vlans/README.md
git commit -m "feat(labs-pkt/03): agregar lab VLANs y trunking" -m "Agrega el lab 03 completo con:
- Escenario empresarial con 3 departamentos y VLAN de administración
- Configuración completa de VLANs 10, 20, 30 y 99 en SW1 y SW2
- Puertos de acceso por departamento y trunk 802.1Q entre switches
- VLAN nativa 99 en el trunk como buena práctica de seguridad
- Verificación con show vlan brief y show interfaces trunk
- Pruebas de conectividad: qué debe funcionar y qué no
- Troubleshooting para trunk inactivo y VLANs faltantes
- Lista de verificación con checkboxes
- Tabla de capturas requeridas

Cubre CCNA 200-301 dominio 2.1 y 2.2 — VLANs y trunking"
git push origin main

¿Continuamos con labs-packet-tracer/04-inter-vlan/README.md?ayudame a realizar ahora ese de favor18:06labs-packet-tracer/04-inter-vlan/README.md
markdown# Lab 04 — Inter-VLAN Routing

## Información del lab

| Campo | Detalle |
|-------|---------|
| Tema CCNA | Acceso de red — Dominio 2.1 y 2.3 |
| Dificultad | ⭐⭐ Intermedio |
| Duración estimada | 50 minutos |
| Archivo | lab-04-inter-vlan.pkt |
| Teoría relacionada | [teoria/02-acceso-de-red/vlans-trunking-stp.md](../../teoria/02-acceso-de-red/vlans-trunking-stp.md) |
| Lab anterior | [Lab 03 — VLANs y Trunking](../03-vlans/README.md) |

---

## Objetivo

Configurar comunicación entre VLANs usando dos métodos distintos:
Router on a Stick con un router externo y Switch Virtual Interfaces
con un switch de capa 3. Verificar que los dispositivos de distintas
VLANs puedan comunicarse entre sí a través del router.

---

## Escenario

Continuando con **TechStart S.A.** del lab anterior. Las VLANs ya están
segmentadas pero ahora los departamentos necesitan comunicarse entre sí
de forma controlada. Se implementarán ambos métodos para comparar.

| VLAN | Nombre | Red | Gateway |
|------|--------|-----|---------|
| 10 | Ventas | 192.168.10.0/24 | 192.168.10.1 |
| 20 | TI | 192.168.20.0/24 | 192.168.20.1 |
| 30 | RRHH | 192.168.30.0/24 | 192.168.30.1 |
| 99 | Administracion | 192.168.99.0/24 | 192.168.99.1 |

---

## Topología — Método 1: Router on a Stick
PC-Ventas1   PC-TI1   PC-RRHH1
│            │         │
Fa0/1       Fa0/2     Fa0/3
└───────────┴─────────┘
SW1
(G0/1 trunk)
│
G0/0
R1
G0/0.10  → VLAN 10 → 192.168.10.1
G0/0.20  → VLAN 20 → 192.168.20.1
G0/0.30  → VLAN 30 → 192.168.30.1
G0/0.99  → VLAN 99 → 192.168.99.1

---

## Tabla de direccionamiento — Método 1

| Dispositivo | Interfaz | Dirección IP | Máscara | VLAN |
|-------------|----------|-------------|---------|------|
| R1 | G0/0.10 | 192.168.10.1 | 255.255.255.0 | 10 |
| R1 | G0/0.20 | 192.168.20.1 | 255.255.255.0 | 20 |
| R1 | G0/0.30 | 192.168.30.1 | 255.255.255.0 | 30 |
| R1 | G0/0.99 | 192.168.99.1 | 255.255.255.0 | 99 |
| SW1 | VLAN 99 | 192.168.99.2 | 255.255.255.0 | 99 |
| PC-Ventas1 | NIC | 192.168.10.10 | 255.255.255.0 | 10 |
| PC-TI1 | NIC | 192.168.20.10 | 255.255.255.0 | 20 |
| PC-RRHH1 | NIC | 192.168.30.10 | 255.255.255.0 | 30 |

---

## Instrucciones — Método 1: Router on a Stick

### Parte 1 — Configurar SW1

#### Paso 1 — Crear VLANs (si no vienen del lab anterior)
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

#### Paso 2 — Configurar puertos de acceso
SW1(config)# interface FastEthernet0/1
SW1(config-if)# switchport mode access
SW1(config-if)# switchport access vlan 10
SW1(config-if)# description PC-Ventas1
SW1(config-if)# no shutdown
SW1(config-if)# exit
SW1(config)# interface FastEthernet0/2
SW1(config-if)# switchport mode access
SW1(config-if)# switchport access vlan 20
SW1(config-if)# description PC-TI1
SW1(config-if)# no shutdown
SW1(config-if)# exit
SW1(config)# interface FastEthernet0/3
SW1(config-if)# switchport mode access
SW1(config-if)# switchport access vlan 30
SW1(config-if)# description PC-RRHH1
SW1(config-if)# no shutdown
SW1(config-if)# exit

#### Paso 3 — Configurar trunk hacia R1
SW1(config)# interface GigabitEthernet0/1
SW1(config-if)# description Trunk-hacia-R1
SW1(config-if)# switchport mode trunk
SW1(config-if)# switchport trunk native vlan 99
SW1(config-if)# switchport trunk allowed vlan 10,20,30,99
SW1(config-if)# no shutdown
SW1(config-if)# exit

#### Paso 4 — Configurar interfaz de administración
SW1(config)# interface vlan 99
SW1(config-if)# ip address 192.168.99.2 255.255.255.0
SW1(config-if)# no shutdown
SW1(config-if)# exit
SW1(config)# ip default-gateway 192.168.99.1
SW1(config)# end
SW1# copy running-config startup-config

---

### Parte 2 — Configurar R1 (Router on a Stick)

#### Paso 1 — Configuración básica
Router> enable
Router# configure terminal
Router(config)# hostname R1
R1(config)# enable secret Cisco123
R1(config)# no ip domain-lookup

#### Paso 2 — Activar la interfaz física
R1(config)# interface GigabitEthernet0/0
R1(config-if)# description Trunk-hacia-SW1
R1(config-if)# no ip address
R1(config-if)# no shutdown
R1(config-if)# exit

> La interfaz física no lleva IP. Las subinterfaces llevan las IPs.
> Solo necesita estar en no shutdown.

#### Paso 3 — Crear subinterfaz para VLAN 10
R1(config)# interface GigabitEthernet0/0.10
R1(config-subif)# description Gateway-Ventas
R1(config-subif)# encapsulation dot1Q 10
R1(config-subif)# ip address 192.168.10.1 255.255.255.0
R1(config-subif)# exit

#### Paso 4 — Crear subinterfaz para VLAN 20
R1(config)# interface GigabitEthernet0/0.20
R1(config-subif)# description Gateway-TI
R1(config-subif)# encapsulation dot1Q 20
R1(config-subif)# ip address 192.168.20.1 255.255.255.0
R1(config-subif)# exit

#### Paso 5 — Crear subinterfaz para VLAN 30
R1(config)# interface GigabitEthernet0/0.30
R1(config-subif)# description Gateway-RRHH
R1(config-subif)# encapsulation dot1Q 30
R1(config-subif)# ip address 192.168.30.1 255.255.255.0
R1(config-subif)# exit

#### Paso 6 — Crear subinterfaz para VLAN 99
R1(config)# interface GigabitEthernet0/0.99
R1(config-subif)# description Gateway-Administracion
R1(config-subif)# encapsulation dot1Q 99 native
R1(config-subif)# ip address 192.168.99.1 255.255.255.0
R1(config-subif)# exit

> La palabra clave `native` en la subinterfaz de la VLAN nativa
> es importante. Debe coincidir con la VLAN nativa del trunk del switch.

#### Paso 7 — Guardar configuración
R1(config)# end
R1# copy running-config startup-config

---

## Topología — Método 2: Switch Capa 3 con SVI
PC-Ventas1   PC-TI1   PC-RRHH1
│            │         │
Fa0/1       Fa0/2     Fa0/3
└───────────┴─────────┘
SW-L3
(Switch capa 3)
SVI VLAN 10 → 192.168.10.1
SVI VLAN 20 → 192.168.20.1
SVI VLAN 30 → 192.168.30.1
SVI VLAN 99 → 192.168.99.1

---

## Instrucciones — Método 2: Switch Capa 3 con SVI

### Configurar SW-L3 (Switch Multilayer 3560 o 3650)

#### Paso 1 — Configuración básica y VLANs
Switch> enable
Switch# configure terminal
Switch(config)# hostname SW-L3
SW-L3(config)# vlan 10
SW-L3(config-vlan)# name Ventas
SW-L3(config-vlan)# exit
SW-L3(config)# vlan 20
SW-L3(config-vlan)# name TI
SW-L3(config-vlan)# exit
SW-L3(config)# vlan 30
SW-L3(config-vlan)# name RRHH
SW-L3(config-vlan)# exit
SW-L3(config)# vlan 99
SW-L3(config-vlan)# name Administracion
SW-L3(config-vlan)# exit

#### Paso 2 — Configurar puertos de acceso
SW-L3(config)# interface FastEthernet0/1
SW-L3(config-if)# switchport mode access
SW-L3(config-if)# switchport access vlan 10
SW-L3(config-if)# no shutdown
SW-L3(config-if)# exit
SW-L3(config)# interface FastEthernet0/2
SW-L3(config-if)# switchport mode access
SW-L3(config-if)# switchport access vlan 20
SW-L3(config-if)# no shutdown
SW-L3(config-if)# exit
SW-L3(config)# interface FastEthernet0/3
SW-L3(config-if)# switchport mode access
SW-L3(config-if)# switchport access vlan 30
SW-L3(config-if)# no shutdown
SW-L3(config-if)# exit

#### Paso 3 — Crear SVIs (Switch Virtual Interfaces)
SW-L3(config)# interface vlan 10
SW-L3(config-if)# description Gateway-Ventas
SW-L3(config-if)# ip address 192.168.10.1 255.255.255.0
SW-L3(config-if)# no shutdown
SW-L3(config-if)# exit
SW-L3(config)# interface vlan 20
SW-L3(config-if)# description Gateway-TI
SW-L3(config-if)# ip address 192.168.20.1 255.255.255.0
SW-L3(config-if)# no shutdown
SW-L3(config-if)# exit
SW-L3(config)# interface vlan 30
SW-L3(config-if)# description Gateway-RRHH
SW-L3(config-if)# ip address 192.168.30.1 255.255.255.0
SW-L3(config-if)# no shutdown
SW-L3(config-if)# exit
SW-L3(config)# interface vlan 99
SW-L3(config-if)# description Administracion
SW-L3(config-if)# ip address 192.168.99.1 255.255.255.0
SW-L3(config-if)# no shutdown
SW-L3(config-if)# exit

#### Paso 4 — Habilitar enrutamiento IP
SW-L3(config)# ip routing

> Este comando es el más importante del método SVI.
> Sin él el switch no enruta entre VLANs aunque tenga las SVIs.

#### Paso 5 — Guardar configuración
SW-L3(config)# end
SW-L3# copy running-config startup-config

---

## Verificación

### Método 1 — Router on a Stick

#### Verificar subinterfaces del router
R1# show ip interface brief

Resultado esperado:
Interface            IP-Address      OK? Method Status   Protocol
GigabitEthernet0/0   unassigned      YES manual up       up
GigabitEthernet0/0.10 192.168.10.1  YES manual up       up
GigabitEthernet0/0.20 192.168.20.1  YES manual up       up
GigabitEthernet0/0.30 192.168.30.1  YES manual up       up
GigabitEthernet0/0.99 192.168.99.1  YES manual up       up

#### Verificar tabla de enrutamiento
R1# show ip route

Resultado esperado:
C    192.168.10.0/24 is directly connected, GigabitEthernet0/0.10
C    192.168.20.0/24 is directly connected, GigabitEthernet0/0.20
C    192.168.30.0/24 is directly connected, GigabitEthernet0/0.30
C    192.168.99.0/24 is directly connected, GigabitEthernet0/0.99

### Método 2 — Switch Capa 3

#### Verificar SVIs
SW-L3# show ip interface brief

#### Verificar que ip routing esté activo
SW-L3# show ip route

#### Verificar SVIs individualmente
SW-L3# show interfaces vlan 10
SW-L3# show interfaces vlan 20

### Pruebas de conectividad

#### Pruebas que DEBEN funcionar (inter-VLAN activo)
PC-Ventas1 ping 192.168.20.10   ← PC-TI1      ✓ debe funcionar
PC-Ventas1 ping 192.168.30.10   ← PC-RRHH1    ✓ debe funcionar
PC-TI1     ping 192.168.30.10   ← PC-RRHH1    ✓ debe funcionar
PC-Ventas1 ping 192.168.10.1    ← Gateway R1  ✓ debe funcionar

#### Traceroute para verificar el camino
PC-Ventas1> tracert 192.168.30.10

Resultado esperado (Router on a Stick):
1   192.168.10.1   (R1 subinterfaz G0/0.10)
2   192.168.30.10  (PC-RRHH1)

### Lista de verificación

- [ ] Subinterfaces G0/0.10, .20, .30 y .99 creadas en R1
- [ ] Encapsulation dot1Q configurada en cada subinterfaz
- [ ] Interfaz física G0/0 en no shutdown sin IP
- [ ] Trunk entre SW1 y R1 activo
- [ ] PC-Ventas1 hace ping a PC-TI1
- [ ] PC-Ventas1 hace ping a PC-RRHH1
- [ ] PC-TI1 hace ping a PC-RRHH1
- [ ] Traceroute muestra el gateway como primer salto
- [ ] ip routing habilitado en SW-L3 (método 2)
- [ ] Configuración guardada en todos los dispositivos

---

## Comparativa entre métodos

| Aspecto | Router on a Stick | Switch Capa 3 SVI |
|---------|------------------|-------------------|
| Hardware | Router externo + switch | Solo switch capa 3 |
| Rendimiento | Limitado por interfaz del router | Mucho más rápido |
| Costo | Mayor (dos equipos) | Menor (un equipo) |
| Escalabilidad | Limitada | Alta |
| Configuración | Subinterfaces + trunk | SVIs + ip routing |
| Uso típico | Redes pequeñas, laboratorio | Redes empresariales |

---

## Troubleshooting

### Las subinterfaces están down

Verificar que la interfaz física esté en no shutdown
R1# show interfaces GigabitEthernet0/0
Verificar que el trunk del switch esté activo
SW1# show interfaces trunk
Verificar que el número de VLAN en encapsulation coincida
R1# show running-config | section interface GigabitEthernet0/0


### Ping entre VLANs falla con Router on a Stick

Verificar que el gateway de la PC sea la subinterfaz del router
PC debe tener gateway 192.168.10.1 (no la IP del switch)
Verificar encapsulation dot1Q en la subinterfaz
R1# show interfaces GigabitEthernet0/0.10
Verificar que la VLAN esté permitida en el trunk
SW1# show interfaces trunk


### ip routing no está habilitado en switch capa 3
SW-L3# show ip route
Si aparece: Default gateway is not set
Solución:
SW-L3(config)# ip routing

### SVI aparece como down/down

La VLAN debe existir en la base de datos de VLANs
SW-L3# show vlan brief
Debe haber al menos un puerto activo en esa VLAN
SW-L3# show vlan id 10
Crear la VLAN si no existe
SW-L3(config)# vlan 10
SW-L3(config-vlan)# name Ventas


---

## Conceptos aplicados en este lab

| Concepto | Dónde se aplica |
|----------|----------------|
| Router on a Stick | Método 1 con subinterfaces |
| Subinterfaces | G0/0.10, G0/0.20, G0/0.30 en R1 |
| encapsulation dot1Q | Asociar subinterfaz a VLAN |
| SVI | Método 2 con switch capa 3 |
| ip routing | Habilitar enrutamiento en switch L3 |
| Inter-VLAN routing | Comunicación entre VLANs distintas |

---

## Capturas requeridas

| Archivo | Contenido |
|---------|-----------|
| topologia-ros.png | Topología Router on a Stick |
| topologia-svi.png | Topología Switch capa 3 |
| r1-subinterfaces.png | show ip interface brief en R1 |
| r1-route.png | show ip route en R1 |
| swl3-route.png | show ip route en SW-L3 |
| ping-ventas-ti.png | Ping exitoso PC-Ventas1 a PC-TI1 |
| ping-ventas-rrhh.png | Ping exitoso PC-Ventas1 a PC-RRHH1 |
| traceroute.png | Traceroute mostrando el gateway como primer salto |
