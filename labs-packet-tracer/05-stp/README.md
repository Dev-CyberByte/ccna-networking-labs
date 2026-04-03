# Lab 05 — Spanning Tree Protocol

## Información del lab

| Campo | Detalle |
|-------|---------|
| Tema CCNA | Acceso de red — Dominio 2.4 |
| Dificultad | ⭐⭐ Intermedio |
| Duración estimada | 50 minutos |
| Archivo | lab-05-stp.pkt |
| Teoría relacionada | [teoria/02-acceso-de-red/vlans-trunking-stp.md](../../teoria/02-acceso-de-red/vlans-trunking-stp.md) |
| Lab anterior | [Lab 04 — Inter-VLAN Routing](../04-inter-vlan/README.md) |

---

## Objetivo

Observar el comportamiento de STP en una topología con enlaces
redundantes, identificar el Root Bridge, los puertos en estado
forwarding y blocked, manipular la elección del Root Bridge
cambiando prioridades y migrar de STP clásico a Rapid PVST+.

---

## Escenario

**TechStart S.A.** está creciendo y agregó redundancia en su red
de switches para alta disponibilidad. Sin STP la red colapsaría
por loops. Se analizará cómo STP resuelve esto y cómo controlarlo.

---

## Topología
                SW1
               /   \
        trunk /     \ trunk
             /       \
           SW2 ─────── SW3
                trunk
Todos los enlaces son GigabitEthernet trunk.
Las tres VLANs 10, 20 y 30 viajan por todos los trunks.

### Conexiones físicas

| Enlace | SW origen | Interfaz | SW destino | Interfaz |
|--------|-----------|----------|------------|----------|
| SW1-SW2 | SW1 | G0/1 | SW2 | G0/1 |
| SW1-SW3 | SW1 | G0/2 | SW3 | G0/1 |
| SW2-SW3 | SW2 | G0/2 | SW3 | G0/2 |

### Dispositivos finales

| Dispositivo | Conectado a | Puerto | VLAN | IP |
|-------------|------------|--------|------|----|
| PC-Ventas | SW2 | Fa0/1 | 10 | 192.168.10.10 |
| PC-TI | SW2 | Fa0/2 | 20 | 192.168.20.10 |
| PC-RRHH | SW3 | Fa0/1 | 30 | 192.168.30.10 |

---

## Tabla de direccionamiento

| Dispositivo | Interfaz | Dirección IP | Máscara |
|-------------|----------|-------------|---------|
| SW1 | VLAN 99 | 192.168.99.1 | 255.255.255.0 |
| SW2 | VLAN 99 | 192.168.99.2 | 255.255.255.0 |
| SW3 | VLAN 99 | 192.168.99.3 | 255.255.255.0 |

---

## Instrucciones

### Parte 1 — Configuración base de los switches

#### Configurar SW1

```cisco
Switch> enable
Switch# configure terminal
Switch(config)# hostname SW1
SW1(config)# enable secret Cisco123
SW1(config)# no ip domain-lookup
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
SW1(config)# interface GigabitEthernet0/1
SW1(config-if)# description Trunk-hacia-SW2
SW1(config-if)# switchport mode trunk
SW1(config-if)# switchport trunk native vlan 99
SW1(config-if)# switchport trunk allowed vlan 10,20,30,99
SW1(config-if)# no shutdown
SW1(config-if)# exit
SW1(config)# interface GigabitEthernet0/2
SW1(config-if)# description Trunk-hacia-SW3
SW1(config-if)# switchport mode trunk
SW1(config-if)# switchport trunk native vlan 99
SW1(config-if)# switchport trunk allowed vlan 10,20,30,99
SW1(config-if)# no shutdown
SW1(config-if)# exit
SW1(config)# interface vlan 99
SW1(config-if)# ip address 192.168.99.1 255.255.255.0
SW1(config-if)# no shutdown
SW1(config-if)# exit
SW1(config)# end
SW1# copy running-config startup-config
```

#### Configurar SW2

```cisco
Switch> enable
Switch# configure terminal
Switch(config)# hostname SW2
SW2(config)# enable secret Cisco123
SW2(config)# no ip domain-lookup
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
SW2(config)# interface GigabitEthernet0/1
SW2(config-if)# description Trunk-hacia-SW1
SW2(config-if)# switchport mode trunk
SW2(config-if)# switchport trunk native vlan 99
SW2(config-if)# switchport trunk allowed vlan 10,20,30,99
SW2(config-if)# no shutdown
SW2(config-if)# exit
SW2(config)# interface GigabitEthernet0/2
SW2(config-if)# description Trunk-hacia-SW3
SW2(config-if)# switchport mode trunk
SW2(config-if)# switchport trunk native vlan 99
SW2(config-if)# switchport trunk allowed vlan 10,20,30,99
SW2(config-if)# no shutdown
SW2(config-if)# exit
SW2(config)# interface FastEthernet0/1
SW2(config-if)# description PC-Ventas
SW2(config-if)# switchport mode access
SW2(config-if)# switchport access vlan 10
SW2(config-if)# no shutdown
SW2(config-if)# exit
SW2(config)# interface FastEthernet0/2
SW2(config-if)# description PC-TI
SW2(config-if)# switchport mode access
SW2(config-if)# switchport access vlan 20
SW2(config-if)# no shutdown
SW2(config-if)# exit
SW2(config)# interface vlan 99
SW2(config-if)# ip address 192.168.99.2 255.255.255.0
SW2(config-if)# no shutdown
SW2(config-if)# exit
SW2(config)# end
SW2# copy running-config startup-config
```


#### Configurar SW3

```cisco
Switch> enable
Switch# configure terminal
Switch(config)# hostname SW3
SW3(config)# enable secret Cisco123
SW3(config)# no ip domain-lookup
SW3(config)# vlan 10
SW3(config-vlan)# name Ventas
SW3(config-vlan)# exit
SW3(config)# vlan 20
SW3(config-vlan)# name TI
SW3(config-vlan)# exit
SW3(config)# vlan 30
SW3(config-vlan)# name RRHH
SW3(config-vlan)# exit
SW3(config)# vlan 99
SW3(config-vlan)# name Administracion
SW3(config-vlan)# exit
SW3(config)# interface GigabitEthernet0/1
SW3(config-if)# description Trunk-hacia-SW1
SW3(config-if)# switchport mode trunk
SW3(config-if)# switchport trunk native vlan 99
SW3(config-if)# switchport trunk allowed vlan 10,20,30,99
SW3(config-if)# no shutdown
SW3(config-if)# exit
SW3(config)# interface GigabitEthernet0/2
SW3(config-if)# description Trunk-hacia-SW2
SW3(config-if)# switchport mode trunk
SW3(config-if)# switchport trunk native vlan 99
SW3(config-if)# switchport trunk allowed vlan 10,20,30,99
SW3(config-if)# no shutdown
SW3(config-if)# exit
SW3(config)# interface FastEthernet0/1
SW3(config-if)# description PC-RRHH
SW3(config-if)# switchport mode access
SW3(config-if)# switchport access vlan 30
SW3(config-if)# no shutdown
SW3(config-if)# exit
SW3(config)# interface vlan 99
SW3(config-if)# ip address 192.168.99.3 255.255.255.0
SW3(config-if)# no shutdown
SW3(config-if)# exit
SW3(config)# end
SW3# copy running-config startup-config
```
---

### Parte 2 — Observar STP por defecto

Antes de cambiar nada observa cómo STP elige el Root Bridge
automáticamente con la configuración por defecto.

#### Paso 1 — Ver el estado STP en cada switch

```cisco
SW1# show spanning-tree vlan 10
SW2# show spanning-tree vlan 10
SW3# show spanning-tree vlan 10
```

#### Paso 2 — Identificar quién es el Root Bridge

Busca la línea que dice:
This bridge is the root

Si no aparece esa línea en un switch, ese switch NO es el Root Bridge.
En el switch que sí la tenga, todos sus puertos estarán en Forwarding.

#### Paso 3 — Registrar el estado de los puertos

Llena esta tabla con lo que ves en Packet Tracer:

| Switch | Puerto | Rol | Estado |
|--------|--------|-----|--------|
| SW1 | G0/1 | | |
| SW1 | G0/2 | | |
| SW2 | G0/1 | | |
| SW2 | G0/2 | | |
| SW3 | G0/1 | | |
| SW3 | G0/2 | | |

Roles posibles: Root Port / Designated Port / Alternate Port
Estados posibles: Forwarding / Blocking

#### Paso 4 — Identificar el puerto bloqueado
El puerto bloqueado es el que STP eligió para cortar el loop.
Solo uno de los seis puertos trunk estará en estado Blocking.
```cisco
SW1# show spanning-tree vlan 10 detail
```
Busca la línea:
Status: BLK

---

### Parte 3 — Manipular el Root Bridge

#### Paso 1 — Forzar SW1 como Root Bridge de VLAN 10 y 20
```cisco
SW1(config)# spanning-tree vlan 10 priority 4096
SW1(config)# spanning-tree vlan 20 priority 4096
```

#### Paso 2 — Forzar SW2 como Root Bridge de VLAN 30
```cisco
SW2(config)# spanning-tree vlan 30 priority 4096
```

> Usar prioridades en múltiplos de 4096.
> Valores válidos: 0, 4096, 8192, 12288, 16384, 20480, 24576,
> 28672, 32768 (default), 36864, 40960, 45056, 49152, 53248,
> 57344, 61440.

#### Paso 3 — Usar el comando macro para Root Bridge
Cisco tiene un comando que ajusta la prioridad automáticamente:
```cisco
SW1(config)# spanning-tree vlan 10 root primary
SW1(config)# spanning-tree vlan 20 root primary
SW2(config)# spanning-tree vlan 30 root primary
```

> `root primary` ajusta la prioridad a 24576 o menos si hay
> otro switch con prioridad más baja.
> `root secondary` ajusta a 28672 para ser el respaldo.

#### Paso 4 — Verificar la nueva elección
```cisco
SW1# show spanning-tree vlan 10
SW1# show spanning-tree vlan 20
SW2# show spanning-tree vlan 30
```

Confirmar que aparezca:
This bridge is the root

en SW1 para VLANs 10 y 20, y en SW2 para VLAN 30.

---

### Parte 4 — Migrar a Rapid PVST+

#### Paso 1 — Cambiar el modo STP en los tres switches
```cisco
SW1(config)# spanning-tree mode rapid-pvst
SW2(config)# spanning-tree mode rapid-pvst
SW3(config)# spanning-tree mode rapid-pvst
```

#### Paso 2 — Configurar PortFast en puertos de acceso
PortFast permite que los puertos conectados a dispositivos finales
pasen directamente a Forwarding sin esperar los 30 segundos de STP.

```cisco
SW2(config)# interface FastEthernet0/1
SW2(config-if)# spanning-tree portfast
SW2(config-if)# exit
SW2(config)# interface FastEthernet0/2
SW2(config-if)# spanning-tree portfast
SW2(config-if)# exit
SW3(config)# interface FastEthernet0/1
SW3(config-if)# spanning-tree portfast
SW3(config-if)# exit
```

> PortFast NUNCA se configura en puertos trunk.
> Solo en puertos conectados a dispositivos finales (PCs, servidores).

#### Paso 3 — Configurar BPDU Guard
BPDU Guard deshabilita el puerto si recibe un BPDU.
Protege los puertos PortFast de switches no autorizados.

```cisco
SW2(config)# interface FastEthernet0/1
SW2(config-if)# spanning-tree bpduguard enable
SW2(config-if)# exit
SW2(config)# interface FastEthernet0/2
SW2(config-if)# spanning-tree bpduguard enable
SW2(config-if)# exit
SW3(config)# interface FastEthernet0/1
SW3(config-if)# spanning-tree bpduguard enable
SW3(config-if)# exit
```

#### Paso 4 — Guardar configuración en todos los switches
```cisco
SW1# copy running-config startup-config
SW2# copy running-config startup-config
SW3# copy running-config startup-config
```

---

## Verificación

### Verificar Root Bridge por VLAN

```cisco
SW1# show spanning-tree vlan 10
SW1# show spanning-tree vlan 20
SW2# show spanning-tree vlan 30
```

### Verificar modo Rapid PVST+
```cisco
SW1# show spanning-tree summary
```

Resultado esperado:
Switch is in rapid-pvst mode
Root bridge for: VLAN0010, VLAN0020

### Verificar PortFast y BPDU Guard
```cisco
SW2# show spanning-tree interface FastEthernet0/1 portfast
SW2# show spanning-tree interface FastEthernet0/1 detail
```

### Verificar todos los puertos STP
```cisco
SW1# show spanning-tree vlan 10 brief
```

Resultado esperado:
VLAN0010
Spanning tree enabled protocol rstp
Root ID    Priority    4106
Address     xxxx.xxxx.xxxx
This bridge is the root
Interface        Role Sts Cost      Prio.Nbr Type

Gi0/1            Desg FWD 4         128.1    P2p
Gi0/2            Desg FWD 4         128.2    P2p

### Prueba de redundancia — simular fallo de enlace

#### Paso 1 — Verificar conectividad base
PC-Ventas> ping 192.168.30.10

#### Paso 2 — Desconectar el enlace SW1-SW2 en Packet Tracer
Haz clic en el cable entre SW1 y SW2 y elimínalo.

#### Paso 3 — Verificar que STP converge
```cisco
SW2# show spanning-tree vlan 10
```

El puerto que estaba en Blocking debe pasar a Forwarding
automáticamente en menos de 6 segundos con Rapid PVST+.

#### Paso 4 — Verificar conectividad después del fallo
PC-Ventas> ping 192.168.30.10

La conectividad debe restaurarse automáticamente.

#### Paso 5 — Reconectar el enlace y verificar
Vuelve a conectar SW1-SW2 y observa cómo STP recalcula.

### Lista de verificación

- [ ] SW1 es Root Bridge para VLANs 10 y 20
- [ ] SW2 es Root Bridge para VLAN 30
- [ ] Todos los switches están en modo rapid-pvst
- [ ] PortFast configurado en todos los puertos de acceso
- [ ] BPDU Guard configurado en todos los puertos PortFast
- [ ] Conectividad se recupera después de simular fallo
- [ ] show spanning-tree summary muestra rapid-pvst mode
- [ ] Configuración guardada en los tres switches

---

## Troubleshooting

### El Root Bridge no es el esperado

Verificar la prioridad configurada
```cisco
SW1# show spanning-tree vlan 10 | include Priority
```
Verificar que la prioridad sea menor que los demás switches
Todos los switches tienen 32768 por defecto
SW1 debe tener 4096 para ganar
Reconfigurar si es necesario}
```cisco
SW1(config)# spanning-tree vlan 10 priority 4096
```

### Un puerto no pasa a Forwarding

Verificar el estado del puerto
```cisco
SW2# show spanning-tree vlan 10
```
Verificar que no haya inconsistencia de VLAN nativa
Una inconsistencia bloquea el puerto indefinidamente
Verificar los costos de los enlaces
```cisco
SW2# show spanning-tree vlan 10 detail
```

### BPDU Guard deshabilitó un puerto
Síntoma: el puerto está en estado err-disabled

Verificar cuál puerto fue deshabilitado
```cisco
SW2# show interfaces status err-disabled
```
Identificar por qué recibió un BPDU
Probablemente hay un switch conectado donde no debe haber uno
Resolver el problema físico primero
Rehabilitar el puerto
```cisco
SW2(config)# interface FastEthernet0/1
SW2(config-if)# shutdown
SW2(config-if)# no shutdown
```

### PortFast genera warning en la consola
%SPANTREE-2-PORTFAST_TRUNK: PortFast has been configured
on GigabitEthernet0/1 which is in trunking mode.

Esto significa que configuraste PortFast en un puerto trunk.
PortFast solo va en puertos de acceso conectados a dispositivos finales.
```cisco
SW1(config)# interface GigabitEthernet0/1
SW1(config-if)# no spanning-tree portfast
```
---

## Conceptos aplicados en este lab

| Concepto | Dónde se aplica |
|----------|----------------|
| Root Bridge | Elección automática y manual por prioridad |
| Bridge ID | Prioridad + MAC, determina el Root Bridge |
| Puerto Designated | Todos los puertos del Root Bridge |
| Puerto Root | Mejor camino al Root Bridge |
| Puerto Alternate | Puerto bloqueado para evitar loop |
| PVST+ | Un árbol STP por VLAN |
| Rapid PVST+ | Convergencia rápida menor a 6 segundos |
| PortFast | Forwarding inmediato en puertos de acceso |
| BPDU Guard | Protección contra switches no autorizados |

---

## Capturas requeridas

| Archivo | Contenido |
|---------|-----------|
| topologia.png | Topología con los tres switches y PCs |
| stp-default-sw1.png | show spanning-tree vlan 10 antes de cambios |
| stp-root-sw1.png | SW1 como Root Bridge VLAN 10 |
| stp-root-sw2.png | SW2 como Root Bridge VLAN 30 |
| rapid-pvst-summary.png | show spanning-tree summary en modo rapid-pvst |
| portfast-verificacion.png | show spanning-tree interface Fa0/1 portfast |
| fallo-enlace-recovery.png | Ping recuperándose después del fallo de enlace |
