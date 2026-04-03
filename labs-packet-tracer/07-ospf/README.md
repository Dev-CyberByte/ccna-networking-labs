# Lab 07 — OSPF Single Area

## Información del lab

| Campo | Detalle |
|-------|---------|
| Tema CCNA | Conectividad IP — Dominio 3.3 y 3.4 |
| Dificultad | ⭐⭐⭐ Avanzado |
| Duración estimada | 60 minutos |
| Archivo | lab-07-ospf.pkt |
| Teoría relacionada | [teoria/03-conectividad-ip/enrutamiento-ospf.md](../../teoria/03-conectividad-ip/enrutamiento-ospf.md) |
| Lab anterior | [Lab 06 — Enrutamiento Estático](../06-enrutamiento-estatico/README.md) |

---

## Objetivo

Configurar OSPF de área única (área 0) en una topología de cuatro
routers, verificar la formación de adyacencias, analizar la tabla
de enrutamiento OSPF, manipular el Router ID y los costos,
y configurar la propagación de la ruta por defecto.

---

## Escenario

**TechStart S.A.** creció y ahora tiene cuatro routers interconectados.
El enrutamiento estático ya no es práctico. Se implementará OSPF
para que los routers aprendan las rutas automáticamente y se
adapten a cambios en la topología sin intervención manual.

---

## Topología
LAN1                  LAN2
192.168.1.0/24        192.168.2.0/24
|                     |
R1 ─────────────────── R2
|    10.0.12.0/30      |
|                      |
|    10.0.13.0/30      |  10.0.23.0/30
|                      |
R3 ─────────────────── R4 ── LAN4
|    10.0.34.0/30      |    192.168.4.0/24
LAN3
192.168.3.0/24
Todos los routers están en el Area 0

### Conexiones físicas

| Enlace | Router A | Interfaz | IP | Router B | Interfaz | IP |
|--------|----------|----------|----|----------|----------|----|
| R1-R2 | R1 | G0/1 | 10.0.12.1 | R2 | G0/0 | 10.0.12.2 |
| R1-R3 | R1 | G0/2 | 10.0.13.1 | R3 | G0/0 | 10.0.13.3 |
| R2-R4 | R2 | G0/1 | 10.0.23.2 | R4 | G0/0 | 10.0.23.4 |
| R3-R4 | R3 | G0/1 | 10.0.34.3 | R4 | G0/1 | 10.0.34.4 |

---

## Tabla de direccionamiento

| Dispositivo | Interfaz | Dirección IP | Máscara | Descripción |
|-------------|----------|-------------|---------|-------------|
| R1 | G0/0 | 192.168.1.1 | 255.255.255.0 | LAN1 |
| R1 | G0/1 | 10.0.12.1 | 255.255.255.252 | Enlace R1-R2 |
| R1 | G0/2 | 10.0.13.1 | 255.255.255.252 | Enlace R1-R3 |
| R2 | G0/0 | 10.0.12.2 | 255.255.255.252 | Enlace R2-R1 |
| R2 | G0/1 | 10.0.23.2 | 255.255.255.252 | Enlace R2-R4 |
| R2 | G0/2 | 192.168.2.1 | 255.255.255.0 | LAN2 |
| R3 | G0/0 | 10.0.13.3 | 255.255.255.252 | Enlace R3-R1 |
| R3 | G0/1 | 10.0.34.3 | 255.255.255.252 | Enlace R3-R4 |
| R3 | G0/2 | 192.168.3.1 | 255.255.255.0 | LAN3 |
| R4 | G0/0 | 10.0.23.4 | 255.255.255.252 | Enlace R4-R2 |
| R4 | G0/1 | 10.0.34.4 | 255.255.255.252 | Enlace R4-R3 |
| R4 | G0/2 | 192.168.4.1 | 255.255.255.0 | LAN4 |
| PC1 | NIC | 192.168.1.10 | 255.255.255.0 | 192.168.1.1 |
| PC2 | NIC | 192.168.2.10 | 255.255.255.0 | 192.168.2.1 |
| PC3 | NIC | 192.168.3.10 | 255.255.255.0 | 192.168.3.1 |
| PC4 | NIC | 192.168.4.10 | 255.255.255.0 | 192.168.4.1 |

---

## Instrucciones

### Parte 1 — Configurar interfaces

#### Configurar R1
Router> enable
Router# configure terminal
Router(config)# hostname R1
R1(config)# interface GigabitEthernet0/0
R1(config-if)# description LAN1
R1(config-if)# ip address 192.168.1.1 255.255.255.0
R1(config-if)# no shutdown
R1(config-if)# exit
R1(config)# interface GigabitEthernet0/1
R1(config-if)# description Enlace-R1-R2
R1(config-if)# ip address 10.0.12.1 255.255.255.252
R1(config-if)# no shutdown
R1(config-if)# exit
R1(config)# interface GigabitEthernet0/2
R1(config-if)# description Enlace-R1-R3
R1(config-if)# ip address 10.0.13.1 255.255.255.252
R1(config-if)# no shutdown
R1(config-if)# exit
R1(config)# end
R1# copy running-config startup-config

#### Configurar R2
Router> enable
Router# configure terminal
Router(config)# hostname R2
R2(config)# interface GigabitEthernet0/0
R2(config-if)# description Enlace-R2-R1
R2(config-if)# ip address 10.0.12.2 255.255.255.252
R2(config-if)# no shutdown
R2(config-if)# exit
R2(config)# interface GigabitEthernet0/1
R2(config-if)# description Enlace-R2-R4
R2(config-if)# ip address 10.0.23.2 255.255.255.252
R2(config-if)# no shutdown
R2(config-if)# exit
R2(config)# interface GigabitEthernet0/2
R2(config-if)# description LAN2
R2(config-if)# ip address 192.168.2.1 255.255.255.0
R2(config-if)# no shutdown
R2(config-if)# exit
R2(config)# end
R2# copy running-config startup-config

#### Configurar R3
Router> enable
Router# configure terminal
Router(config)# hostname R3
R3(config)# interface GigabitEthernet0/0
R3(config-if)# description Enlace-R3-R1
R3(config-if)# ip address 10.0.13.3 255.255.255.252
R3(config-if)# no shutdown
R3(config-if)# exit
R3(config)# interface GigabitEthernet0/1
R3(config-if)# description Enlace-R3-R4
R3(config-if)# ip address 10.0.34.3 255.255.255.252
R3(config-if)# no shutdown
R3(config-if)# exit
R3(config)# interface GigabitEthernet0/2
R3(config-if)# description LAN3
R3(config-if)# ip address 192.168.3.1 255.255.255.0
R3(config-if)# no shutdown
R3(config-if)# exit
R3(config)# end
R3# copy running-config startup-config

#### Configurar R4
Router> enable
Router# configure terminal
Router(config)# hostname R4
R4(config)# interface GigabitEthernet0/0
R4(config-if)# description Enlace-R4-R2
R4(config-if)# ip address 10.0.23.4 255.255.255.252
R4(config-if)# no shutdown
R4(config-if)# exit
R4(config)# interface GigabitEthernet0/1
R4(config-if)# description Enlace-R4-R3
R4(config-if)# ip address 10.0.34.4 255.255.255.252
R4(config-if)# no shutdown
R4(config-if)# exit
R4(config)# interface GigabitEthernet0/2
R4(config-if)# description LAN4
R4(config-if)# ip address 192.168.4.1 255.255.255.0
R4(config-if)# no shutdown
R4(config-if)# exit
R4(config)# end
R4# copy running-config startup-config

---

### Parte 2 — Configurar OSPF

#### Configurar OSPF en R1
R1(config)# router ospf 1
R1(config-router)# router-id 1.1.1.1
R1(config-router)# network 192.168.1.0 0.0.0.255 area 0
R1(config-router)# network 10.0.12.0 0.0.0.3 area 0
R1(config-router)# network 10.0.13.0 0.0.0.3 area 0
R1(config-router)# passive-interface GigabitEthernet0/0
R1(config-router)# exit

#### Configurar OSPF en R2
R2(config)# router ospf 1
R2(config-router)# router-id 2.2.2.2
R2(config-router)# network 192.168.2.0 0.0.0.255 area 0
R2(config-router)# network 10.0.12.0 0.0.0.3 area 0
R2(config-router)# network 10.0.23.0 0.0.0.3 area 0
R2(config-router)# passive-interface GigabitEthernet0/2
R2(config-router)# exit

#### Configurar OSPF en R3
R3(config)# router ospf 1
R3(config-router)# router-id 3.3.3.3
R3(config-router)# network 192.168.3.0 0.0.0.255 area 0
R3(config-router)# network 10.0.13.0 0.0.0.3 area 0
R3(config-router)# network 10.0.34.0 0.0.0.3 area 0
R3(config-router)# passive-interface GigabitEthernet0/2
R3(config-router)# exit

#### Configurar OSPF en R4
R4(config)# router ospf 1
R4(config-router)# router-id 4.4.4.4
R4(config-router)# network 192.168.4.0 0.0.0.255 area 0
R4(config-router)# network 10.0.23.0 0.0.0.3 area 0
R4(config-router)# network 10.0.34.0 0.0.0.3 area 0
R4(config-router)# passive-interface GigabitEthernet0/2
R4(config-router)# exit

#### Guardar configuración en todos
R1# copy running-config startup-config
R2# copy running-config startup-config
R3# copy running-config startup-config
R4# copy running-config startup-config

---

### Parte 3 — Propagar ruta por defecto

R1 simula tener salida a internet. Propaga la ruta por defecto
a todos los demás routers del área OSPF.
R1(config)# ip route 0.0.0.0 0.0.0.0 209.165.200.1
R1(config)# router ospf 1
R1(config-router)# default-information originate
R1(config-router)# exit

Verificar que R4 aprende la ruta por defecto:
R4# show ip route

Busca la línea:
O*E2  0.0.0.0/0 [110/1] via 10.0.23.2

---

### Parte 4 — Ajustar costos OSPF

El costo por defecto de FastEthernet y GigabitEthernet es 1,
lo que no diferencia entre velocidades. Ajusta el costo de
referencia para que GigabitEthernet tenga costo 1 y
FastEthernet tenga costo 10.

#### Método 1 — Cambiar el ancho de banda de referencia
R1(config)# router ospf 1
R1(config-router)# auto-cost reference-bandwidth 1000
R1(config-router)# exit

Repite en todos los routers:
R2(config)# router ospf 1
R2(config-router)# auto-cost reference-bandwidth 1000
R2(config-router)# exit
R3(config)# router ospf 1
R3(config-router)# auto-cost reference-bandwidth 1000
R3(config-router)# exit
R4(config)# router ospf 1
R4(config-router)# auto-cost reference-bandwidth 1000
R4(config-router)# exit

> Debe configurarse igual en TODOS los routers del área.
> Si no coincide, los cálculos de SPF serán inconsistentes.

#### Método 2 — Configurar costo manualmente en una interfaz
R1(config)# interface GigabitEthernet0/1
R1(config-if)# ip ospf cost 10
R1(config-if)# exit

---

## Verificación

### Verificar adyacencias OSPF
R1# show ip ospf neighbor

Resultado esperado en R1:
Neighbor ID   Pri  State     Dead Time  Address      Interface
2.2.2.2        1   FULL/DR   00:00:38   10.0.12.2    Gi0/1
3.3.3.3        1   FULL/DR   00:00:36   10.0.13.3    Gi0/2

> El estado debe ser FULL. Cualquier otro estado indica un problema.

### Verificar tabla de enrutamiento OSPF
R1# show ip route ospf

Resultado esperado en R1:
O     192.168.2.0/24 [110/2] via 10.0.12.2, GigabitEthernet0/1
O     192.168.3.0/24 [110/2] via 10.0.13.3, GigabitEthernet0/2
O     192.168.4.0/24 [110/3] via 10.0.12.2, GigabitEthernet0/1
O     10.0.23.0/30   [110/2] via 10.0.12.2, GigabitEthernet0/1
O     10.0.34.0/30   [110/2] via 10.0.13.3, GigabitEthernet0/2

### Verificar base de datos OSPF
R1# show ip ospf database

Todos los routers del área deben tener la misma LSDB.

### Verificar detalles de OSPF
R1# show ip ospf
R1# show ip ospf interface GigabitEthernet0/1
R1# show ip protocols

### Verificar ruta por defecto propagada
R4# show ip route

Busca:
O*E2  0.0.0.0/0 [110/1] via ...

### Pruebas de conectividad

Desde PC1 hacer ping a todas las LANs:
ping 192.168.1.1    <- Gateway R1
ping 192.168.2.10   <- PC2
ping 192.168.3.10   <- PC3
ping 192.168.4.10   <- PC4

Traceroute desde PC1 a PC4:
PC1> tracert 192.168.4.10

Resultado esperado:
1   192.168.1.1    <- R1
2   10.0.12.2      <- R2
3   10.0.23.4      <- R4
4   192.168.4.10   <- PC4

### Prueba de convergencia OSPF

OSPF debe recalcular rutas automáticamente si un enlace falla.

1. Anotar la ruta actual de R1 a 192.168.4.0:
R1# show ip route 192.168.4.0

2. Desconectar el enlace R1-R2 en Packet Tracer

3. Esperar convergencia (menos de 10 segundos con OSPF)

4. Verificar la nueva ruta:
R1# show ip route 192.168.4.0

La ruta ahora debe ir por R3 en lugar de R2.

5. Reconectar el enlace y verificar que vuelve a la ruta original.

### Lista de verificación

- [ ] Router ID configurado manualmente en los cuatro routers
- [ ] OSPF proceso 1 activo en los cuatro routers
- [ ] Adyacencias FULL entre R1-R2, R1-R3, R2-R4 y R3-R4
- [ ] passive-interface configurado en todas las interfaces LAN
- [ ] Cada router conoce todas las redes via OSPF
- [ ] Ruta por defecto O*E2 aparece en R2, R3 y R4
- [ ] auto-cost reference-bandwidth 1000 en los cuatro routers
- [ ] PC1 hace ping a PC2, PC3 y PC4
- [ ] Traceroute muestra el camino correcto
- [ ] OSPF converge automáticamente al desconectar un enlace
- [ ] Configuración guardada en los cuatro routers

---

## Troubleshooting

### Las adyacencias no se forman

Verificar que las interfaces estén up/up en ambos extremos
R1# show ip interface brief
Verificar que el area ID coincida en ambos extremos
R1# show ip ospf interface GigabitEthernet0/1
Verificar que los hello/dead timers coincidan
R1# show ip ospf interface GigabitEthernet0/1
Busca: Timer intervals configured
Verificar que la red esté incluida en el comando network
R1# show running-config | section router ospf
Verificar que no haya passive-interface en un enlace entre routers
R1# show ip protocols


### El vecino aparece en estado INIT o 2-WAY en lugar de FULL
Estado INIT: R1 recibió Hello de R2 pero R2 no ha visto
el Hello de R1 aún. Espera unos segundos.
Estado 2-WAY: Ambos se ven pero no son DR/BDR entre sí.
En enlaces punto a punto debe llegar a FULL.
Si no llega a FULL verificar el tipo de red:
R1# show ip ospf interface GigabitEthernet0/1
Busca: Network Type

### Una red no aparece en la tabla OSPF

Verificar que el comando network incluya esa red
R2# show running-config | section router ospf
Verificar que la interfaz no sea passive
R2# show ip protocols
Verificar que la adyacencia con el vecino que tiene esa red esté FULL
R2# show ip ospf neighbor


### El Router ID no es el configurado manualmente
OSPF toma el Router ID al iniciar el proceso.
Si ya estaba corriendo cuando configuraste el router-id
debes reiniciar el proceso OSPF:
R1# clear ip ospf process
Responde yes cuando pregunte

### La ruta por defecto no aparece en los demás routers

Verificar que R1 tenga la ruta estática por defecto
R1# show ip route static
Verificar que default-information originate esté configurado
R1# show running-config | section router ospf
Si la ruta estática no existe agregar always al comando:
R1(config)# router ospf 1
R1(config-router)# default-information originate always


---

## Conceptos aplicados en este lab

| Concepto | Dónde se aplica |
|----------|----------------|
| OSPF proceso 1 | router ospf 1 en los cuatro routers |
| Router ID | Configurado manualmente 1.1.1.1 a 4.4.4.4 |
| Wildcard mask | Inverso de la máscara en el comando network |
| Area 0 | Todos los routers en el backbone area |
| passive-interface | Interfaces LAN sin vecinos OSPF |
| Adyacencia FULL | Estado correcto entre vecinos OSPF |
| LSDB | Base de datos igual en todos los routers del área |
| SPF | Algoritmo que calcula las rutas desde la LSDB |
| default-information originate | Propagar ruta por defecto en OSPF |
| auto-cost reference-bandwidth | Ajustar costo para GigabitEthernet |

---

## Capturas requeridas

| Archivo | Contenido |
|---------|-----------|
| topologia.png | Vista general de la topología con cuatro routers |
| r1-ospf-neighbor.png | show ip ospf neighbor en R1 |
| r1-ip-route-ospf.png | show ip route ospf en R1 |
| r4-ruta-default.png | show ip route en R4 con ruta O*E2 |
| r1-ospf-database.png | show ip ospf database en R1 |
| ping-pc1-pc4.png | Ping exitoso de PC1 a PC4 |
| traceroute-pc1-pc4.png | Traceroute de PC1 a PC4 |
| convergencia-ospf.png | show ip route en R1 después de desconectar enlace |
