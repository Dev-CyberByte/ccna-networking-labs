# Lab 08 — DHCP y DNS

## Información del lab

| Campo | Detalle |
|-------|---------|
| Tema CCNA | Servicios IP — Dominio 4.1 y 4.2 |
| Dificultad | ⭐⭐ Intermedio |
| Duración estimada | 45 minutos |
| Archivo | lab-08-dhcp-dns.pkt |
| Teoría relacionada | [teoria/04-servicios-ip/dhcp-nat-dns-ntp.md](../../teoria/04-servicios-ip/dhcp-nat-dns-ntp.md) |
| Lab anterior | [Lab 07 — OSPF Single Area](../07-ospf/README.md) |

---

## Objetivo

Configurar un servidor DHCP en un router Cisco para asignar
direcciones IP automáticamente a múltiples VLANs, configurar
DHCP relay para redes remotas, configurar DNS básico y verificar
que los clientes reciben su configuración de red correctamente.

---

## Escenario

**TechStart S.A.** tiene tres departamentos en VLANs separadas.
El administrador necesita eliminar la configuración manual de IPs
en todas las PCs. Se configurará el router como servidor DHCP
con un pool por VLAN y un servidor DNS para resolución de nombres.

---

## Topología
PC-Ventas1  PC-Ventas2    PC-TI1  PC-TI2    PC-RRHH1
|             |           |       |          |
Fa0/1         Fa0/2       Fa0/3   Fa0/4      Fa0/1
└─────────────┴───────────┴───────┘          |
SW1                            SW2
(G0/1 trunk)                  (G0/1 trunk)
|                             |
G0/0                          G0/1
└──────────── R1 ────────────┘
(DHCP Server)
G0/0 → SW1
G0/1 → SW2
Servidor DNS: 192.168.99.100 (PC en VLAN 99)

---

## Tabla de direccionamiento

| Dispositivo | Interfaz/VLAN | Dirección IP | Máscara | Descripción |
|-------------|--------------|-------------|---------|-------------|
| R1 | G0/0.10 | 192.168.10.1 | 255.255.255.0 | Gateway VLAN 10 |
| R1 | G0/0.20 | 192.168.20.1 | 255.255.255.0 | Gateway VLAN 20 |
| R1 | G0/0.99 | 192.168.99.1 | 255.255.255.0 | Gateway VLAN 99 |
| R1 | G0/1.30 | 192.168.30.1 | 255.255.255.0 | Gateway VLAN 30 |
| SW1 | VLAN 99 | 192.168.99.2 | 255.255.255.0 | Administración |
| SW2 | VLAN 99 | 192.168.99.3 | 255.255.255.0 | Administración |
| Servidor DNS | NIC | 192.168.99.100 | 255.255.255.0 | DNS estático |
| PC-Ventas1 | NIC | DHCP | — | VLAN 10 |
| PC-Ventas2 | NIC | DHCP | — | VLAN 10 |
| PC-TI1 | NIC | DHCP | — | VLAN 20 |
| PC-TI2 | NIC | DHCP | — | VLAN 20 |
| PC-RRHH1 | NIC | DHCP | — | VLAN 30 |

---

## Pools DHCP a configurar

| Pool | VLAN | Red | Rango excluido | Gateway | DNS |
|------|------|-----|---------------|---------|-----|
| POOL-VENTAS | 10 | 192.168.10.0/24 | .1 — .10 | 192.168.10.1 | 192.168.99.100 |
| POOL-TI | 20 | 192.168.20.0/24 | .1 — .10 | 192.168.20.1 | 192.168.99.100 |
| POOL-RRHH | 30 | 192.168.30.0/24 | .1 — .10 | 192.168.30.1 | 192.168.99.100 |

---

## Instrucciones

### Parte 1 — Configurar SW1

#### Paso 1 — Crear VLANs y puertos de acceso
```cisco
Switch> enable
Switch# configure terminal
Switch(config)# hostname SW1
SW1(config)# vlan 10
SW1(config-vlan)# name Ventas
SW1(config-vlan)# exit
SW1(config)# vlan 20
SW1(config-vlan)# name TI
SW1(config-vlan)# exit
SW1(config)# vlan 99
SW1(config-vlan)# name Administracion
SW1(config-vlan)# exit
SW1(config)# interface FastEthernet0/1
SW1(config-if)# switchport mode access
SW1(config-if)# switchport access vlan 10
SW1(config-if)# description PC-Ventas1
SW1(config-if)# no shutdown
SW1(config-if)# exit
SW1(config)# interface FastEthernet0/2
SW1(config-if)# switchport mode access
SW1(config-if)# switchport access vlan 10
SW1(config-if)# description PC-Ventas2
SW1(config-if)# no shutdown
SW1(config-if)# exit
SW1(config)# interface FastEthernet0/3
SW1(config-if)# switchport mode access
SW1(config-if)# switchport access vlan 20
SW1(config-if)# description PC-TI1
SW1(config-if)# no shutdown
SW1(config-if)# exit
SW1(config)# interface FastEthernet0/4
SW1(config-if)# switchport mode access
SW1(config-if)# switchport access vlan 20
SW1(config-if)# description PC-TI2
SW1(config-if)# no shutdown
SW1(config-if)# exit
```

#### Paso 2 — Configurar trunk hacia R1
```cisco
SW1(config)# interface GigabitEthernet0/1
SW1(config-if)# description Trunk-hacia-R1
SW1(config-if)# switchport mode trunk
SW1(config-if)# switchport trunk native vlan 99
SW1(config-if)# switchport trunk allowed vlan 10,20,99
SW1(config-if)# no shutdown
SW1(config-if)# exit
```

#### Paso 3 — Configurar interfaz de administración
```cisco
SW1(config)# interface vlan 99
SW1(config-if)# ip address 192.168.99.2 255.255.255.0
SW1(config-if)# no shutdown
SW1(config-if)# exit
SW1(config)# ip default-gateway 192.168.99.1
SW1(config)# end
SW1# copy running-config startup-config
```
---

### Parte 2 — Configurar SW2
```cisco
Switch> enable
Switch# configure terminal
Switch(config)# hostname SW2
SW2(config)# vlan 30
SW2(config-vlan)# name RRHH
SW2(config-vlan)# exit
SW2(config)# vlan 99
SW2(config-vlan)# name Administracion
SW2(config-vlan)# exit
SW2(config)# interface FastEthernet0/1
SW2(config-if)# switchport mode access
SW2(config-if)# switchport access vlan 30
SW2(config-if)# description PC-RRHH1
SW2(config-if)# no shutdown
SW2(config-if)# exit
SW2(config)# interface FastEthernet0/2
SW2(config-if)# switchport mode access
SW2(config-if)# switchport access vlan 99
SW2(config-if)# description Servidor-DNS
SW2(config-if)# no shutdown
SW2(config-if)# exit
SW2(config)# interface GigabitEthernet0/1
SW2(config-if)# description Trunk-hacia-R1
SW2(config-if)# switchport mode trunk
SW2(config-if)# switchport trunk native vlan 99
SW2(config-if)# switchport trunk allowed vlan 30,99
SW2(config-if)# no shutdown
SW2(config-if)# exit
SW2(config)# interface vlan 99
SW2(config-if)# ip address 192.168.99.3 255.255.255.0
SW2(config-if)# no shutdown
SW2(config-if)# exit
SW2(config)# ip default-gateway 192.168.99.1
SW2(config)# end
SW2# copy running-config startup-config
```
---

### Parte 3 — Configurar R1

#### Paso 1 — Configurar subinterfaces
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
R1(config-subif)# description Gateway-Ventas
R1(config-subif)# encapsulation dot1Q 10
R1(config-subif)# ip address 192.168.10.1 255.255.255.0
R1(config-subif)# exit
R1(config)# interface GigabitEthernet0/0.20
R1(config-subif)# description Gateway-TI
R1(config-subif)# encapsulation dot1Q 20
R1(config-subif)# ip address 192.168.20.1 255.255.255.0
R1(config-subif)# exit
R1(config)# interface GigabitEthernet0/0.99
R1(config-subif)# description Gateway-Administracion
R1(config-subif)# encapsulation dot1Q 99 native
R1(config-subif)# ip address 192.168.99.1 255.255.255.0
R1(config-subif)# exit
R1(config)# interface GigabitEthernet0/1
R1(config-if)# no ip address
R1(config-if)# no shutdown
R1(config-if)# exit
R1(config)# interface GigabitEthernet0/1.30
R1(config-subif)# description Gateway-RRHH
R1(config-subif)# encapsulation dot1Q 30
R1(config-subif)# ip address 192.168.30.1 255.255.255.0
R1(config-subif)# exit
```

#### Paso 2 — Configurar exclusiones DHCP
```cisco
R1(config)# ip dhcp excluded-address 192.168.10.1 192.168.10.10
R1(config)# ip dhcp excluded-address 192.168.20.1 192.168.20.10
R1(config)# ip dhcp excluded-address 192.168.30.1 192.168.30.10
```

> Siempre excluye antes de crear el pool.
> Las IPs excluidas nunca serán asignadas por DHCP.
> Esto reserva el rango .1 a .10 para dispositivos
> de red con IPs estáticas.

#### Paso 3 — Configurar pools DHCP
```cisco
R1(config)# ip dhcp pool POOL-VENTAS
R1(dhcp-config)# network 192.168.10.0 255.255.255.0
R1(dhcp-config)# default-router 192.168.10.1
R1(dhcp-config)# dns-server 192.168.99.100
R1(dhcp-config)# lease 1
R1(dhcp-config)# exit
R1(config)# ip dhcp pool POOL-TI
R1(dhcp-config)# network 192.168.20.0 255.255.255.0
R1(dhcp-config)# default-router 192.168.20.1
R1(dhcp-config)# dns-server 192.168.99.100
R1(dhcp-config)# lease 1
R1(dhcp-config)# exit
R1(config)# ip dhcp pool POOL-RRHH
R1(dhcp-config)# network 192.168.30.0 255.255.255.0
R1(dhcp-config)# default-router 192.168.30.1
R1(dhcp-config)# dns-server 192.168.99.100
R1(dhcp-config)# lease 1
R1(dhcp-config)# exit
```

#### Paso 4 — Guardar configuración
```cisco
R1(config)# end
R1# copy running-config startup-config
```
---

### Parte 4 — Configurar DHCP Relay

```cisco
SW2 está en una red distinta al servidor DHCP (R1).
Las PCs de VLAN 30 no podrán obtener IP porque los broadcasts
DHCP no cruzan redes. El relay reenvía esos broadcasts a R1.
R1(config)# interface GigabitEthernet0/1.30
R1(config-subif)# ip helper-address 192.168.30.1
R1(config-subif)# exit
```

> En este lab el servidor DHCP está en el mismo router
> que el gateway, por lo que el relay es interno.
> En un escenario real el helper-address apuntaría
> a la IP del servidor DHCP externo.

---

### Parte 5 — Configurar el Servidor DNS

En Packet Tracer agrega un servidor genérico en SW2 Fa0/2
con IP estática 192.168.99.100.

Configuración de la NIC del servidor:
IP Address:      192.168.99.100
Subnet Mask:     255.255.255.0
Default Gateway: 192.168.99.1
DNS Server:      192.168.99.100

En la pestaña Services → DNS del servidor agrega los registros:

| Nombre | Tipo | Dirección |
|--------|------|-----------|
| servidor.empresa.local | A Record | 192.168.99.100 |
| r1.empresa.local | A Record | 192.168.99.1 |
| intranet.empresa.local | A Record | 192.168.99.100 |

Activa el servicio DNS en On.

---

### Parte 6 — Solicitar IP por DHCP en las PCs

En cada PC ve a Desktop → IP Configuration y selecciona DHCP.
Espera unos segundos a que aparezca la IP asignada.

Resultado esperado:

| PC | IP asignada | Gateway | DNS |
|----|------------|---------|-----|
| PC-Ventas1 | 192.168.10.11 | 192.168.10.1 | 192.168.99.100 |
| PC-Ventas2 | 192.168.10.12 | 192.168.10.1 | 192.168.99.100 |
| PC-TI1 | 192.168.20.11 | 192.168.20.1 | 192.168.99.100 |
| PC-TI2 | 192.168.20.12 | 192.168.20.1 | 192.168.99.100 |
| PC-RRHH1 | 192.168.30.11 | 192.168.30.1 | 192.168.99.100 |

---

## Verificación

### Verificar pools DHCP en R1
```cisco
R1# show ip dhcp pool
```
Resultado esperado:
Pool POOL-VENTAS :
Utilization mark (high/low)    : 100 / 0
Subnet size (first/next)       : 0 / 0
Total addresses                : 254
Leased addresses               : 2
Pending event                  : none
1 subnet is currently in the free pool

### Verificar concesiones activas
```cisco
R1# show ip dhcp binding
```

Resultado esperado:
IP address      Client-ID/              Lease expiration        Type
Hardware address
192.168.10.11   0060.2F12.3456          Mar 01 2025 12:00 AM    Automatic
192.168.10.12   0060.4B78.9ABC          Mar 01 2025 12:00 AM    Automatic
192.168.20.11   0060.7C34.DEF0          Mar 01 2025 12:00 AM    Automatic

### Verificar conflictos DHCP
```cisco
R1# show ip dhcp conflict
```

Si no hay conflictos no aparece nada. Eso es lo esperado.

### Verificar estadísticas DHCP
```cisco
R1# show ip dhcp server statistics
```

Busca que los contadores de Discover, Offer, Request y ACK
tengan valores mayores a cero.

### Verificar resolución DNS

Desde PC-Ventas1 en Desktop → Web Browser:
http://intranet.empresa.local

Desde Desktop → Command Prompt:
ping intranet.empresa.local
ping r1.empresa.local

Si el DNS funciona el ping resolverá el nombre a IP
antes de enviar los paquetes.

### Pruebas de conectividad
```bash
PC-Ventas1> ping 192.168.20.11    <- PC-TI1
PC-Ventas1> ping 192.168.30.11    <- PC-RRHH1
PC-TI1>     ping 192.168.30.11    <- PC-RRHH1
PC-RRHH1>   ping 192.168.99.100   <- Servidor DNS
```

### Lista de verificación

- [ ] VLANs 10, 20, 30 y 99 creadas en SW1 y SW2
- [ ] Trunks activos entre SW1-R1 y SW2-R1
- [ ] Subinterfaces en R1 con IPs correctas
- [ ] Exclusiones DHCP configuradas antes de los pools
- [ ] Tres pools DHCP creados con network, gateway y DNS
- [ ] PC-Ventas1 obtuvo IP por DHCP en rango 192.168.10.11+
- [ ] PC-TI1 obtuvo IP por DHCP en rango 192.168.20.11+
- [ ] PC-RRHH1 obtuvo IP por DHCP en rango 192.168.30.11+
- [ ] show ip dhcp binding muestra las concesiones activas
- [ ] Servidor DNS tiene registros A configurados
- [ ] ping intranet.empresa.local resuelve correctamente
- [ ] Configuración guardada en R1, SW1 y SW2

---

## Troubleshooting

### La PC no obtiene IP por DHCP

Verificar que el pool DHCP exista en R1
```cisco
R1# show ip dhcp pool
```
Verificar que la subinterfaz del gateway esté up/up
```cisco
R1# show ip interface brief
```
Verificar que el trunk esté activo y permita la VLAN
```cisco
SW1# show interfaces trunk
```
Verificar que el puerto de la PC esté en la VLAN correcta
```cisco
SW1# show vlan brief
```
Verificar que no haya conflicto de IPs
```cisco
R1# show ip dhcp conflict
```

### La PC obtuvo IP 169.254.x.x (APIPA)
Esto significa que el proceso DORA falló completamente.
La PC no recibió respuesta del servidor DHCP.

Verificar conectividad entre la PC y el gateway
Asegúrate que el cable esté conectado
Verificar que el pool cubra la subred de la PC
Si la PC está en VLAN 10 (192.168.10.0/24)
el pool debe tener network 192.168.10.0 255.255.255.0
Renovar la IP manualmente en la PC
Ve a IP Configuration → Static y luego vuelve a DHCP


### show ip dhcp binding está vacío

Las PCs deben estar en modo DHCP en IP Configuration
Ve a Desktop → IP Configuration → DHCP
Espera 5-10 segundos para que complete el proceso DORA
Verificar que la exclusión no cubra todo el rango
```cisco
R1# show running-config | include excluded
```
Si excluded-address es .1 a .254 no quedan IPs para asignar


### El nombre DNS no resuelve

Verificar que el servicio DNS esté On en el servidor
Verificar que el registro A exista en el servidor DNS
Revisa la lista de registros en Services → DNS
Verificar que las PCs tengan la IP del DNS correcta
Debe ser 192.168.99.100
Verificar conectividad entre la PC y el servidor DNS
PC> ping 192.168.99.100


---

## Conceptos aplicados en este lab

| Concepto | Dónde se aplica |
|----------|----------------|
| Proceso DORA | Asignación automática en las PCs |
| ip dhcp excluded-address | Reservar IPs para dispositivos estáticos |
| ip dhcp pool | Definir red, gateway y DNS por VLAN |
| lease | Tiempo de concesión de la IP |
| ip helper-address | DHCP relay para redes remotas |
| Registro A en DNS | Resolución de nombre a IPv4 |
| show ip dhcp binding | Verificar concesiones activas |

---

## Capturas requeridas

| Archivo | Contenido |
|---------|-----------|
| topologia.png | Vista general de la topología |
| r1-dhcp-pool.png | show ip dhcp pool en R1 |
| r1-dhcp-binding.png | show ip dhcp binding con concesiones activas |
| pc-ventas1-dhcp.png | IP Configuration de PC-Ventas1 con IP obtenida |
| pc-ti1-dhcp.png | IP Configuration de PC-TI1 con IP obtenida |
| pc-rrhh1-dhcp.png | IP Configuration de PC-RRHH1 con IP obtenida |
| dns-registros.png | Registros A configurados en el servidor DNS |
| ping-dns-nombre.png | Ping exitoso usando nombre DNS |
