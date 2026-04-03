# Lab GNS3 02 — VLANs y Trunking

## Información del lab

| Campo | Detalle |
|-------|---------|
| Tema CCNA | Acceso de red — Dominio 2.1 y 2.2 |
| Dificultad | ⭐⭐ Intermedio |
| Duración estimada | 50 minutos |
| Archivo | lab-gns3-02-vlans.gns3 |
| Equivalente PKT | [Lab PKT 03](../../labs-packet-tracer/03-vlans/README.md) |
| Teoría relacionada | [teoria/02-acceso-de-red/vlans-trunking-stp.md](../../teoria/02-acceso-de-red/vlans-trunking-stp.md) |

---

## Objetivo

Configurar VLANs y trunking en switches con IOS real en GNS3.
Observar el comportamiento real del protocolo 802.1Q, verificar
la base de datos de VLANs y analizar el tráfico etiquetado
usando las herramientas de captura de GNS3.

---

## Diferencias importantes con Packet Tracer

| Aspecto | Packet Tracer | GNS3 |
|---------|--------------|------|
| Base de datos VLANs | Automática | Requiere vlan.dat |
| Captura de tráfico | No disponible | Wireshark integrado |
| Negociación DTP | Simulada | Real |
| VTP | Limitado | Completo |
| Encapsulación trunk | dot1Q automático | Requiere configuración |

---

## Requisitos de este lab

- Imagen IOS: IOSvL2 o c3560 para switches
- Wireshark instalado (para captura de tráfico)
- GNS3 2.x con al menos dos switches configurados

---

## Topología
PC-Ventas1  PC-TI1  PC-RRHH1       PC-Ventas2  PC-TI2  PC-RRHH2
|           |        |               |           |        |
Fa0/1      Fa0/2    Fa0/3           Fa0/1       Fa0/2    Fa0/3
└───────────┴────────┘               └───────────┴────────┘
SW1                                  SW2
(Fa0/5 trunk) ─────────────────────── (Fa0/5 trunk)
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

### Parte 1 — Preparar el entorno GNS3

#### Paso 1 — Crear el proyecto
1. File → New blank project
2. Nombre: `lab-gns3-02-vlans`
3. Arrastra dos switches IOSvL2 al canvas
4. Arrastra seis VPCS al canvas
5. Renombra: SW1, SW2, PC-Ventas1, PC-TI1, PC-RRHH1,
   PC-Ventas2, PC-TI2, PC-RRHH2

#### Paso 2 — Conectar dispositivos

Conexiones en SW1:
PC-Ventas1 eth0  →  SW1 Fa0/1
PC-TI1     eth0  →  SW1 Fa0/2
PC-RRHH1   eth0  →  SW1 Fa0/3
SW1        Fa0/5  →  SW2 Fa0/5  (enlace trunk)

Conexiones en SW2:
PC-Ventas2 eth0  →  SW2 Fa0/1
PC-TI2     eth0  →  SW2 Fa0/2
PC-RRHH2   eth0  →  SW2 Fa0/3

#### Paso 3 — Iniciar todos los dispositivos
Click en el botón Play verde.
Espera que todos los switches muestren triángulo verde.

---

### Parte 2 — Configurar SW1

#### Paso 1 — Configuración básica
```cisco

Switch> enable
Switch# configure terminal
Switch(config)# hostname SW1
SW1(config)# enable secret Cisco123
SW1(config)# no ip domain-lookup
```

#### Paso 2 — Deshabilitar VTP
En GNS3 con IOS real VTP puede causar problemas inesperados.
Es buena práctica deshabilitarlo en labs:
SW1(config)# vtp mode transparent

> VTP modo transparent no propaga VLANs automáticamente.
> Cada switch mantiene su propia base de datos de VLANs.
> En producción se configura VTP con cuidado,
> en labs transparent evita sorpresas.

#### Paso 3 — Crear las VLANs
```cisco
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

#### Paso 4 — Configurar puertos de acceso
```cisco
SW1(config)# interface FastEthernet0/1
SW1(config-if)# description PC-Ventas1
SW1(config-if)# switchport mode access
SW1(config-if)# switchport access vlan 10
SW1(config-if)# spanning-tree portfast
SW1(config-if)# no shutdown
SW1(config-if)# exit
SW1(config)# interface FastEthernet0/2
SW1(config-if)# description PC-TI1
SW1(config-if)# switchport mode access
SW1(config-if)# switchport access vlan 20
SW1(config-if)# spanning-tree portfast
SW1(config-if)# no shutdown
SW1(config-if)# exit
SW1(config)# interface FastEthernet0/3
SW1(config-if)# description PC-RRHH1
SW1(config-if)# switchport mode access
SW1(config-if)# switchport access vlan 30
SW1(config-if)# spanning-tree portfast
SW1(config-if)# no shutdown
SW1(config-if)# exit
```

#### Paso 5 — Configurar puerto trunk hacia SW2

En IOS real el trunk puede requerir especificar
la encapsulación antes del modo trunk:
```cisco
SW1(config)# interface FastEthernet0/5
SW1(config-if)# description Trunk-hacia-SW2
SW1(config-if)# switchport trunk encapsulation dot1q
SW1(config-if)# switchport mode trunk
SW1(config-if)# switchport trunk native vlan 99
SW1(config-if)# switchport trunk allowed vlan 10,20,30,99
SW1(config-if)# no shutdown
SW1(config-if)# exit
```

> `switchport trunk encapsulation dot1q` es necesario
> en switches con IOS real que soportan múltiples
> encapsulaciones (dot1q e ISL).
> En Packet Tracer esto no es necesario.
> Si tu IOS no lo requiere omite esa línea.

#### Paso 6 — Interfaz de administración
```cisco
SW1(config)# interface vlan 99
SW1(config-if)# description Administracion
SW1(config-if)# ip address 192.168.99.1 255.255.255.0
SW1(config-if)# no shutdown
SW1(config-if)# exit
SW1(config)# ip default-gateway 192.168.99.254
```

#### Paso 7 — Guardar
```cisco
SW1(config)# end
SW1# copy running-config startup-config
SW1# write memory
```
---

### Parte 3 — Configurar SW2

#### Paso 1 — Configuración básica y VTP
```cisco
Switch> enable
Switch# configure terminal
Switch(config)# hostname SW2
SW2(config)# enable secret Cisco123
SW2(config)# no ip domain-lookup
SW2(config)# vtp mode transparent
```

#### Paso 2 — Crear VLANs
```cisco
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

#### Paso 3 — Puertos de acceso
```cisco
SW2(config)# interface FastEthernet0/1
SW2(config-if)# description PC-Ventas2
SW2(config-if)# switchport mode access
SW2(config-if)# switchport access vlan 10
SW2(config-if)# spanning-tree portfast
SW2(config-if)# no shutdown
SW2(config-if)# exit
SW2(config)# interface FastEthernet0/2
SW2(config-if)# description PC-TI2
SW2(config-if)# switchport mode access
SW2(config-if)# switchport access vlan 20
SW2(config-if)# spanning-tree portfast
SW2(config-if)# no shutdown
SW2(config-if)# exit
SW2(config)# interface FastEthernet0/3
SW2(config-if)# description PC-RRHH2
SW2(config-if)# switchport mode access
SW2(config-if)# switchport access vlan 30
SW2(config-if)# spanning-tree portfast
SW2(config-if)# no shutdown
SW2(config-if)# exit
```

#### Paso 4 — Puerto trunk hacia SW1
```cisco
SW2(config)# interface FastEthernet0/5
SW2(config-if)# description Trunk-hacia-SW1
SW2(config-if)# switchport trunk encapsulation dot1q
SW2(config-if)# switchport mode trunk
SW2(config-if)# switchport trunk native vlan 99
SW2(config-if)# switchport trunk allowed vlan 10,20,30,99
SW2(config-if)# no shutdown
SW2(config-if)# exit
```

#### Paso 5 — Interfaz de administración
```cisco
SW2(config)# interface vlan 99
SW2(config-if)# description Administracion
SW2(config-if)# ip address 192.168.99.2 255.255.255.0
SW2(config-if)# no shutdown
SW2(config-if)# exit
SW2(config)# ip default-gateway 192.168.99.254
SW2(config)# end
SW2# copy running-config startup-config
SW2# write memory
```
---

### Parte 4 — Configurar las PCs con VPCS
```bash
PC-Ventas1> ip 192.168.10.10 255.255.255.0 192.168.10.1
PC-Ventas1> save
PC-Ventas2> ip 192.168.10.11 255.255.255.0 192.168.10.1
PC-Ventas2> save
PC-TI1> ip 192.168.20.10 255.255.255.0 192.168.20.1
PC-TI1> save
PC-TI2> ip 192.168.20.11 255.255.255.0 192.168.20.1
PC-TI2> save
PC-RRHH1> ip 192.168.30.10 255.255.255.0 192.168.30.1
PC-RRHH1> save
PC-RRHH2> ip 192.168.30.11 255.255.255.0 192.168.30.1
PC-RRHH2> save
```
---

### Parte 5 — Captura de tráfico con Wireshark

Esta es la ventaja exclusiva de GNS3 sobre Packet Tracer.
Puedes capturar el tráfico real del enlace trunk y ver
las etiquetas 802.1Q en Wireshark.

#### Paso 1 — Iniciar captura en el enlace trunk
1. Click derecho en el cable entre SW1 y SW2
2. Selecciona Start capture
3. GNS3 abre Wireshark automáticamente

#### Paso 2 — Generar tráfico
PC-Ventas1> ping 192.168.10.11
PC-TI1>     ping 192.168.20.11
PC-RRHH1>   ping 192.168.30.11

#### Paso 3 — Analizar en Wireshark
En Wireshark aplica el filtro:
vlan

Verás los frames con etiqueta 802.1Q que incluyen:
- VLAN ID (10, 20 o 30 según el departamento)
- Priority bits
- Tag Protocol Identifier (0x8100)

> Esto confirma visualmente que el trunking 802.1Q
> está funcionando con IOS real. Es imposible
> ver esto en Packet Tracer.

---

## Verificación

### Verificar VTP mode
```cisco

SW1# show vtp status
```

Resultado esperado:
VTP Version capable             : 1 to 3
VTP version running             : 1
VTP Domain Name                 :
VTP Pruning Mode                : Disabled
VTP Traps Generation            : Disabled
Device ID                       : 0800.27XX.XXXX
Configuration last modified by 0.0.0.0 at 0-0-00 00:00:00
Feature VLAN:
VTP Operating Mode                : Transparent
Maximum VLANs supported locally   : 1005
Number of existing VLANs          : 8
Configuration Revision            : 0
MD5 digest                        : ...

### Verificar base de datos de VLANs
```cisco
SW1# show vlan brief
```

Resultado esperado:
VLAN Name                Status    Ports

1    default              active    Fa0/4, Fa0/6...
10   Ventas               active    Fa0/1
20   TI                   active    Fa0/2
30   RRHH                 active    Fa0/3
99   Administracion       active

### Verificar el trunk
```cisco
SW1# show interfaces trunk
```

Resultado esperado:
Port        Mode         Encapsulation  Status        Native vlan
Fa0/5       on           802.1q         trunking      99
Port        Vlans allowed on trunk
Fa0/5       10,20,30,99
Port        Vlans allowed and active in management domain
Fa0/5       10,20,30,99
Port        Vlans in spanning tree forwarding state and not pruned
Fa0/5       10,20,30,99

### Verificar encapsulación del trunk
```
SW1# show interfaces FastEthernet0/5 trunk
SW1# show interfaces FastEthernet0/5 switchport
```

Busca:
Administrative Trunking Encapsulation: dot1q
Operational Trunking Encapsulation: dot1q

### Verificar VTP en detalle
```cisco
SW1# show vtp status
SW2# show vtp status
```

Ambos deben mostrar:
VTP Operating Mode: Transparent

### Pruebas de conectividad

#### Pruebas que DEBEN funcionar
PC-Ventas1> ping 192.168.10.11   <- PC-Ventas2  debe funcionar
PC-TI1>     ping 192.168.20.11   <- PC-TI2      debe funcionar
PC-RRHH1>   ping 192.168.30.11   <- PC-RRHH2    debe funcionar

#### Pruebas que NO deben funcionar
PC-Ventas1> ping 192.168.20.10   <- PC-TI1      debe FALLAR
PC-Ventas1> ping 192.168.30.10   <- PC-RRHH1    debe FALLAR
PC-TI1>     ping 192.168.30.10   <- PC-RRHH1    debe FALLAR

### Lista de verificación

- [ ] VTP mode transparent en SW1 y SW2
- [ ] VLANs 10, 20, 30 y 99 creadas en SW1 y SW2
- [ ] Puertos Fa0/1, Fa0/2 y Fa0/3 en modo access
- [ ] Puerto Fa0/5 en modo trunk con encapsulación dot1q
- [ ] VLAN nativa 99 en el trunk
- [ ] VLANs 10, 20, 30 y 99 permitidas en el trunk
- [ ] Captura Wireshark muestra etiquetas 802.1Q
- [ ] PC-Ventas1 hace ping a PC-Ventas2
- [ ] Ping entre VLANs distintas falla correctamente
- [ ] show interfaces trunk muestra status trunking
- [ ] Configuración guardada con write memory

---

## Troubleshooting

### Error al configurar el trunk
% An encapsulation type must be specified before setting
the trunking mode to on.
Solución:
SW1(config-if)# switchport trunk encapsulation dot1q
SW1(config-if)# switchport mode trunk

### El trunk aparece como not-trunking

Verificar la encapsulación en ambos extremos
Ambos deben usar dot1q
Verificar que no haya negociación DTP fallando
SW1(config-if)# switchport nonegotiate
Esto desactiva DTP y fuerza el modo trunk estático
Verificar que la interfaz esté en no shutdown
SW1# show interfaces FastEthernet0/5


### Las VLANs desaparecen al reiniciar el switch
En GNS3 con IOS real las VLANs se guardan en vlan.dat
no en el startup-config. Para conservarlas:

Verificar que el proyecto esté guardado
File → Save Project
Usar write memory después de crear las VLANs
En algunos IOS usar:
```cisco
SW1# copy running-config startup-config
```
Y confirmar que las VLANs aparecen en show vlan brief
después de reiniciar


### VTP borra las VLANs de un switch
Si un switch con VTP server tiene revision number mayor
puede sobrescribir las VLANs de otros switches.
Solución: siempre usar vtp mode transparent en labs.
```cisco
SW1(config)# vtp mode transparent
SW2(config)# vtp mode transparent
```
---

## Comandos exclusivos vs Packet Tracer

| Comando | Disponible en PKT | GNS3 |
|---------|------------------|------|
| `switchport trunk encapsulation dot1q` | No | Sí |
| `show vtp status` | Limitado | Completo |
| `show interfaces trunk` | Sí | Completo con más detalle |
| Captura Wireshark | No | Sí |
| `debug sw-vlan vtp events` | No | Sí |

---

## Conceptos aplicados en este lab

| Concepto | Dónde se aplica |
|----------|----------------|
| VTP transparent | Evitar propagación automática de VLANs |
| encapsulation dot1q | Encapsulación explícita en IOS real |
| vlan.dat | Archivo de base de datos de VLANs en IOS |
| DTP | Protocolo de negociación de trunk Cisco |
| Wireshark | Captura y análisis de tráfico 802.1Q real |
| switchport nonegotiate | Deshabilitar DTP para trunk estático |

---

## Capturas requeridas

| Archivo | Contenido |
|---------|-----------|
| topologia.png | Canvas GNS3 con switches y PCs |
| sw1-vlan-brief.png | show vlan brief en SW1 |
| sw1-trunk.png | show interfaces trunk en SW1 |
| sw1-vtp-status.png | show vtp status en SW1 |
| wireshark-dot1q.png | Captura Wireshark mostrando etiquetas 802.1Q |
| ping-misma-vlan.png | Ping exitoso entre PCs de la misma VLAN |
| ping-distinta-vlan.png | Ping fallido entre VLANs distintas |
