## Lab 04 — Inter-VLAN Routing

## Información del laboratorio

| Campo                  | Detalle                                                                      |
| ---------------------- | ---------------------------------------------------------------------------- |
| **Tema CCNA**          | Acceso de red — Dominios 2.1 y 2.3                                           |
| **Nivel**              | ⭐⭐ Intermedio                                                                |
| **Duración estimada**  | 50 minutos                                                                   |
| **Archivo**            | `lab-04-inter-vlan.pkt`                                                      |
| **Teoría relacionada** | [VLANs, Trunking y STP](../../teoria/02-acceso-de-red/vlans-trunking-stp.md) |
| **Lab previo**         | [Lab 03 — VLANs y Trunking](../03-vlans/README.md)                           |

## Objetivo

Configurar comunicación entre VLANs usando dos métodos distintos:
Router on a Stick con un router externo y Switch Virtual Interfaces
con un switch de capa 3. Verificar que los dispositivos de distintas
VLANs puedan comunicarse entre sí a través del router.

## Escenario

Continuando con **TechStart S.A.** del lab anterior. Las VLANs ya están
segmentadas pero ahora los departamentos necesitan comunicarse entre sí
de forma controlada. Se implementarán ambos métodos para comparar.

## Plan de direccionamiento
| VLAN | Nombre         | Red             | Gateway      |
| ---- | -------------- | --------------- | ------------ |
| 10   | Ventas         | 192.168.10.0/24 | 192.168.10.1 |
| 20   | TI             | 192.168.20.0/24 | 192.168.20.1 |
| 30   | RRHH           | 192.168.30.0/24 | 192.168.30.1 |
| 99   | Administración | 192.168.99.0/24 | 192.168.99.1 |

## Método 1 — Router on a Stick (ROAS)

# Topología
PC-Ventas1   PC-TI1   PC-RRHH1
   │            │         │
 Fa0/1       Fa0/2     Fa0/3
     └─────────┬─────────┘
               SW1
            (Trunk G0/1)
                 │
                R1


# Subinterfaces del router
| Interfaz | VLAN | IP           |
| -------- | ---- | ------------ |
| G0/0.10  | 10   | 192.168.10.1 |
| G0/0.20  | 20   | 192.168.20.1 |
| G0/0.30  | 30   | 192.168.30.1 |
| G0/0.99  | 99   | 192.168.99.1 |

# Tabla de direccionamiento
| Dispositivo | Interfaz | IP            | Máscara       | VLAN |
| ----------- | -------- | ------------- | ------------- | ---- |
| R1          | G0/0.10  | 192.168.10.1  | 255.255.255.0 | 10   |
| R1          | G0/0.20  | 192.168.20.1  | 255.255.255.0 | 20   |
| R1          | G0/0.30  | 192.168.30.1  | 255.255.255.0 | 30   |
| R1          | G0/0.99  | 192.168.99.1  | 255.255.255.0 | 99   |
| SW1         | VLAN 99  | 192.168.99.2  | 255.255.255.0 | 99   |
| PC-Ventas1  | NIC      | 192.168.10.10 | 255.255.255.0 | 10   |
| PC-TI1      | NIC      | 192.168.20.10 | 255.255.255.0 | 20   |
| PC-RRHH1    | NIC      | 192.168.30.10 | 255.255.255.0 | 30   |

# Configuración resumida
Switch (SW1)
-Creación de VLANs
-Puertos en modo access
-Trunk hacia R1 (802.1Q)
-VLAN 99 para administración

Router (R1)
-Interfaz física sin IP
-Creación de subinterfaces
-Uso de encapsulation dot1Q
-Gateway para cada VLAN

Puntos clave
-La interfaz física G0/0 NO lleva IP
-Cada VLAN se asocia mediante subinterfaces
-La VLAN nativa debe coincidir entre switch y router

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
# Configuración de puertos access
interface range fa0/1 - 3
 switchport mode access
 no shutdown
 exit

interface fa0/1
 switchport access vlan 10
 description PC-Ventas1

interface fa0/2
 switchport access vlan 20
 description PC-TI1

interface fa0/3
 switchport access vlan 30
 description PC-RRHH1

 #### Paso 3 — Configurar trunk hacia R1

# Trunk hacia Router (ROAS)

SW1(config)# interface GigabitEthernet0/1
SW1(config-if)# description Trunk-hacia-R1
SW1(config-if)# switchport mode trunk
SW1(config-if)# switchport trunk native vlan 99
SW1(config-if)# switchport trunk allowed vlan 10,20,30,99
SW1(config-if)# no shutdown
SW1(config-if)# exit

#### Paso 4 — Configurar interfaz de administración

# VLAN de Administración

SW1(config)# interface vlan 99
SW1(config-if)# ip address 192.168.99.2 255.255.255.0
SW1(config-if)# no shutdown
SW1(config-if)# exit

# Gateway por defecto del switch
SW1(config)# ip default-gateway 192.168.99.1

# Guardar configuración
SW1(config)# end
SW1# copy running-config startup-config

---

### Parte 2 — Configurar R1 (Router on a Stick)

#### Paso 1 — Configuración básica

# Configuración básica

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

Nota:
La interfaz física no lleva dirección IP, ya que el enrutamiento se realiza mediante subinterfaces.

> La interfaz física no lleva IP. Las subinterfaces llevan las IPs.
> Solo necesita estar en no shutdown.

#### Paso 3 — Crear subinterfaz para VLAN 10

# VLAN 10 — Ventas

R1(config)# interface GigabitEthernet0/0.10
R1(config-subif)# description Gateway-Ventas
R1(config-subif)# encapsulation dot1Q 10
R1(config-subif)# ip address 192.168.10.1 255.255.255.0
R1(config-subif)# exit

#### Paso 4 — Crear subinterfaz para VLAN 20
# VLAN 20 — TI

R1(config)# interface GigabitEthernet0/0.20
R1(config-subif)# description Gateway-TI
R1(config-subif)# encapsulation dot1Q 20
R1(config-subif)# ip address 192.168.20.1 255.255.255.0
R1(config-subif)# exit

#### Paso 5 — Crear subinterfaz para VLAN 30
# VLAN 30 — RRHH

R1(config)# interface GigabitEthernet0/0.30
R1(config-subif)# description Gateway-RRHH
R1(config-subif)# encapsulation dot1Q 30
R1(config-subif)# ip address 192.168.30.1 255.255.255.0
R1(config-subif)# exit

#### Paso 6 — Crear subinterfaz para VLAN 99
# VLAN 99 — Administración (Native VLAN)

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
     └─────────┬─────────┘
             SW-L3

---

## Instrucciones — Método 2: Switch Capa 3 con SVI
Configuración clave
🔧 Switch Multicapa
Creación de VLANs
Configuración de puertos access
Creación de SVI (Switch Virtual Interfaces)
Activación de enrutamiento:
ip routing

### Configurar SW-L3 (Switch Multilayer 3560 o 3650)

#### Paso 1 — Configuración básica y VLANs

# SVIs configuradas
| VLAN | Interfaz | IP           |
| ---- | -------- | ------------ |
| 10   | VLAN 10  | 192.168.10.1 |
| 20   | VLAN 20  | 192.168.20.1 |
| 30   | VLAN 30  | 192.168.30.1 |
| 99   | VLAN 99  | 192.168.99.1 |

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

# Resultado esperado:
Interface            IP-Address      OK? Method Status   Protocol
GigabitEthernet0/0   unassigned      YES manual up       up
GigabitEthernet0/0.10 192.168.10.1  YES manual up       up
GigabitEthernet0/0.20 192.168.20.1  YES manual up       up
GigabitEthernet0/0.30 192.168.30.1  YES manual up       up
GigabitEthernet0/0.99 192.168.99.1  YES manual up       up

#### Verificar tabla de enrutamiento
R1# show ip route
# Resultado esperado:
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

# Resultado esperado (Router on a Stick):

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

1. La VLAN debe existir en la base de datos de VLANs SW-L3# show vlan brief
2. Debe haber al menos un puerto activo en esa VLAN SW-L3# show vlan id 10
3. Crear la VLAN si no existe SW-L3(config)# vlan 10 SW-L3(config-vlan)# name Ventas

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
