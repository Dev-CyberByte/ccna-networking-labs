# Lab 09 — NAT y PAT

## Información del lab

| Campo | Detalle |
|-------|---------|
| Tema CCNA | Servicios IP — Dominio 4.3 |
| Dificultad | ⭐⭐⭐ Avanzado |
| Duración estimada | 55 minutos |
| Archivo | lab-09-nat.pkt |
| Teoría relacionada | [teoria/04-servicios-ip/dhcp-nat-dns-ntp.md](../../teoria/04-servicios-ip/dhcp-nat-dns-ntp.md) |
| Lab anterior | [Lab 08 — DHCP y DNS](../08-dhcp-dns/README.md) |

---

## Objetivo

Configurar los tres tipos de NAT en un router Cisco: NAT estático
para un servidor interno accesible desde internet, NAT dinámico
con un pool de IPs públicas y PAT para que múltiples dispositivos
internos compartan una sola IP pública. Verificar las traducciones
y entender cuándo usar cada tipo.

---

## Escenario

**TechStart S.A.** tiene una conexión a internet a través de R1.
El ISP asignó el bloque 209.165.200.0/29 con 6 IPs públicas útiles.
Hay un servidor web interno que debe ser accesible desde internet,
un grupo de PCs de ventas con NAT dinámico y el resto de la red
usando PAT para compartir una sola IP pública.

---

## Topología
    Internet (ISP)
         |
     209.165.200.1 (ISP Gateway)
         |
    G0/1 (209.165.200.2) — IP pública
         R1
    G0/0 (192.168.1.1) — LAN interna
         |
        SW1
  ┌──────┴──────┐──────────┐
Fa0/1         Fa0/2      Fa0/3
PC-Ventas1   PC-Ventas2  Servidor-Web
192.168.1.10  192.168.1.11  192.168.1.100

---

## Tabla de direccionamiento

| Dispositivo | Interfaz | Dirección IP | Máscara | Descripción |
|-------------|----------|-------------|---------|-------------|
| R1 | G0/0 | 192.168.1.1 | 255.255.255.0 | LAN interna |
| R1 | G0/1 | 209.165.200.2 | 255.255.255.248 | WAN hacia ISP |
| ISP | G0/0 | 209.165.200.1 | 255.255.255.248 | Gateway ISP |
| PC-Ventas1 | NIC | 192.168.1.10 | 255.255.255.0 | 192.168.1.1 |
| PC-Ventas2 | NIC | 192.168.1.11 | 255.255.255.0 | 192.168.1.1 |
| Servidor-Web | NIC | 192.168.1.100 | 255.255.255.0 | 192.168.1.1 |
| PC-Externa | NIC | 209.165.201.10 | 255.255.255.0 | Simula internet |

### IPs públicas disponibles del ISP

| IP pública | Uso asignado |
|-----------|-------------|
| 209.165.200.2 | IP WAN del router R1 |
| 209.165.200.3 | NAT estático para Servidor-Web |
| 209.165.200.4 | Pool NAT dinámico inicio |
| 209.165.200.5 | Pool NAT dinámico fin |
| 209.165.200.6 | PAT (overload) |

---

## Instrucciones

### Parte 1 — Configurar interfaces de R1
```cisco
Router> enable
Router# configure terminal
Router(config)# hostname R1
R1(config)# no ip domain-lookup
R1(config)# interface GigabitEthernet0/0
R1(config-if)# description LAN-Interna
R1(config-if)# ip address 192.168.1.1 255.255.255.0
R1(config-if)# no shutdown
R1(config-if)# exit
R1(config)# interface GigabitEthernet0/1
R1(config-if)# description WAN-hacia-ISP
R1(config-if)# ip address 209.165.200.2 255.255.255.248
R1(config-if)# no shutdown
R1(config-if)# exit
R1(config)# ip route 0.0.0.0 0.0.0.0 209.165.200.1
R1(config)# end
R1# copy running-config startup-config
```

### Parte 2 — Configurar el ISP (router simulado)
```cisco
Router> enable
Router# configure terminal
Router(config)# hostname ISP
ISP(config)# interface GigabitEthernet0/0
ISP(config-if)# description Hacia-R1
ISP(config-if)# ip address 209.165.200.1 255.255.255.248
ISP(config-if)# no shutdown
ISP(config-if)# exit
ISP(config)# interface GigabitEthernet0/1
ISP(config-if)# description Red-Internet-Simulada
ISP(config-if)# ip address 209.165.201.1 255.255.255.0
ISP(config-if)# no shutdown
ISP(config-if)# exit
ISP(config)# end
ISP# copy running-config startup-config
```

> El ISP no necesita ruta hacia la red privada 192.168.1.0/24.
> Con NAT, desde el ISP solo verá IPs públicas del bloque
> 209.165.200.0/29. Esto es exactamente el comportamiento real.

---

### Parte 3 — NAT Estático

Mapea la IP privada del servidor web a una IP pública fija.
Así el servidor es accesible desde internet siempre con
la misma IP pública.

#### Paso 1 — Definir las interfaces inside y outside
```cisco
R1(config)# interface GigabitEthernet0/0
R1(config-if)# ip nat inside
R1(config-if)# exit
R1(config)# interface GigabitEthernet0/1
R1(config-if)# ip nat outside
R1(config-if)# exit
```

> Esta configuración aplica para todos los tipos de NAT.
> inside = red interna privada
> outside = red externa pública (internet)
> Debe configurarse antes de cualquier regla NAT.

#### Paso 2 — Crear la traducción estática
```cisco
R1(config)# ip nat inside source static 192.168.1.100 209.165.200.3
```

Esto dice:
- IP privada 192.168.1.100 (Servidor-Web)
- Se traduce siempre a IP pública 209.165.200.3
- La traducción es permanente y bidireccional

#### Verificar NAT estático
```cisco
R1# show ip nat translations
```

Resultado esperado:
Pro  Inside global     Inside local       Outside local    Outside global
---  209.165.200.3     192.168.1.100      ---              ---

---

### Parte 4 — NAT Dinámico

Asigna IPs públicas de un pool a dispositivos internos
de forma temporal cuando inician una conexión.

#### Paso 1 — Crear el pool de IPs públicas
```cisco
R1(config)# ip nat pool POOL-PUBLICO 209.165.200.4 209.165.200.5
netmask 255.255.255.248
```

> Este pool tiene solo 2 IPs públicas (.4 y .5).
> Solo 2 dispositivos pueden tener NAT dinámico simultáneamente.
> Si un tercer dispositivo intenta salir, espera hasta que
> una IP del pool quede libre.

#### Paso 2 — Crear ACL que identifica los hosts internos
```cisco
R1(config)# access-list 1 permit 192.168.1.10 0.0.0.1
```
> 0.0.0.1 como wildcard permite .10 y .11 (PC-Ventas1 y PC-Ventas2).
> Solo esas dos IPs usarán NAT dinámico.

#### Paso 3 — Vincular ACL con el pool
```cisco
R1(config)# ip nat inside source list 1 pool POOL-PUBLICO
```

#### Verificar NAT dinámico
Desde PC-Ventas1 hacer ping a 209.165.201.10 y luego:
```cisco
R1# show ip nat translations
```

Resultado esperado:
Pro  Inside global     Inside local       Outside local      Outside global
---  209.165.200.3     192.168.1.100      ---                ---
icmp 209.165.200.4:1   192.168.1.10:1     209.165.201.10:1   209.165.201.10:1
icmp 209.165.200.5:2   192.168.1.11:2     209.165.201.10:2   209.165.201.10:2

---

### Parte 5 — PAT (NAT Overload)

Permite que toda la red interna comparta una sola IP pública
usando puertos distintos para cada sesión.
Es el tipo más usado en redes reales.

#### Opción A — PAT usando la IP de la interfaz WAN

Primero elimina el NAT dinámico anterior para no tener conflictos:
```cisco
R1(config)# no ip nat inside source list 1 pool POOL-PUBLICO
R1(config)# no access-list 1
```

Crea una nueva ACL que incluya toda la red interna:
```cisco
R1(config)# access-list 2 permit 192.168.1.0 0.0.0.255
```

Configura PAT usando la interfaz WAN como IP pública:
```cisco
R1(config)# ip nat inside source list 2 interface GigabitEthernet0/1 overload
```

> La palabra clave `overload` es lo que activa PAT.
> Múltiples IPs privadas se traducen a la misma IP pública
> diferenciadas por el número de puerto.

#### Opción B — PAT usando una IP específica del pool
```cisco
R1(config)# ip nat pool POOL-PAT 209.165.200.6 209.165.200.6
netmask 255.255.255.248
R1(config)# ip nat inside source list 2 pool POOL-PAT overload
```

#### Verificar PAT
Desde PC-Ventas1 y PC-Ventas2 hacer ping simultáneo a 209.165.201.10:
```cisco
R1# show ip nat translations
```

Resultado esperado con PAT:
Pro  Inside global        Inside local         Outside local        Outside global
---  209.165.200.3        192.168.1.100        ---                  ---
icmp 209.165.200.2:1024   192.168.1.10:1024    209.165.201.10:1024  209.165.201.10:1024
icmp 209.165.200.2:1025   192.168.1.11:1025    209.165.201.10:1024  209.165.201.10:1024

Nota como PC-Ventas1 y PC-Ventas2 comparten la misma
Inside global (209.165.200.2) pero con puertos distintos.

---

### Parte 6 — Verificar acceso desde internet al servidor

Con NAT estático el servidor web debe ser alcanzable
desde PC-Externa (que simula internet).

Desde PC-Externa:
ping 209.165.200.3

Desde PC-Externa en Web Browser:
http://209.165.200.3

El tráfico debe llegar al Servidor-Web interno (192.168.1.100).

---

## Verificación completa

### Ver todas las traducciones activas
```cisco
R1# show ip nat translations
R1# show ip nat translations verbose
```

### Ver estadísticas NAT
```cisco
R1# show ip nat statistics
```

Resultado esperado:
Total active translations: 3 (1 static, 2 dynamic; 2 extended)
Outside interfaces: GigabitEthernet0/1
Inside interfaces: GigabitEthernet0/0
Hits: 47  Misses: 3

### Limpiar traducciones dinámicas para pruebas
```cisco
R1# clear ip nat translation *
```
> Esto elimina las traducciones dinámicas activas.
> Las traducciones estáticas permanecen siempre.

### Verificar configuración NAT en las interfaces
```cisco
R1# show ip interface GigabitEthernet0/0 | include NAT
R1# show ip interface GigabitEthernet0/1 | include NAT
```

Resultado esperado:
Inbound  access list is not set
Outbound access list is not set
IP access violation accounting is disabled
TCP/IP header compression is disabled
RTP/IP header compression is disabled
Proxy ARP is enabled
Local Proxy ARP is disabled
Security level is default
Split horizon is enabled
ICMP redirects are always sent
ICMP unreachables are always sent
ICMP mask replies are never sent
IP fast switching is enabled
IP fast switching on the same interface is disabled
IP Flow switching is disabled
IP CEF switching is enabled
IP CEF Fast switching turbo vector
IP multicast fast switching is enabled
IP multicast distributed fast switching is disabled
IP route-cache flags are Fast, CEF
Router Discovery is disabled
IP output packet accounting is disabled
IP access violation accounting is disabled
TCP/IP header compression is disabled
RTP/IP header compression is disabled
Probe proxy name replies are disabled
Policy routing is disabled
Network address translation: Inside       <- busca esta línea

### Pruebas de conectividad

Desde PC-Ventas1:
ping 209.165.200.1      <- ISP Gateway
ping 209.165.201.10     <- PC-Externa (internet simulado)

Desde PC-Externa:
ping 209.165.200.3      <- Servidor Web (via NAT estático)
ping 209.165.200.2      <- IP pública de R1

Desde R1:
```cisco
R1# ping 209.165.201.10
R1# show ip nat translations
```

### Lista de verificación

- [ ] Interfaces G0/0 y G0/1 de R1 up/up con IPs correctas
- [ ] Ruta por defecto apuntando al ISP configurada
- [ ] ip nat inside en G0/0 e ip nat outside en G0/1
- [ ] NAT estático: 192.168.1.100 → 209.165.200.3
- [ ] NAT estático aparece en show ip nat translations
- [ ] PC-Externa hace ping a 209.165.200.3 exitosamente
- [ ] NAT dinámico o PAT configurado para la red interna
- [ ] PC-Ventas1 hace ping a 209.165.201.10 (internet)
- [ ] PC-Ventas2 hace ping a 209.165.201.10 (internet)
- [ ] show ip nat translations muestra traducciones activas
- [ ] show ip nat statistics muestra hits mayores a cero
- [ ] Configuración guardada en R1 e ISP

---

## Troubleshooting

### El ping desde la LAN hacia internet falla

Verificar que la ruta por defecto exista
```cisco
R1# show ip route static
```
Verificar que ip nat inside e outside estén configurados
```cisco
R1# show ip interface brief
R1# show running-config | include ip nat
```
Verificar que la ACL coincida con la IP de origen
```cisco
R1# show access-lists
```
Hacer ping desde el router hacia el ISP
```cisco
R1# ping 209.165.200.1
```
Si esto falla el problema es de conectividad, no de NAT


### show ip nat translations está vacío después del ping

Las traducciones dinámicas duran poco tiempo
Hacer ping y ejecutar show ip nat translations inmediatamente
Verificar que la ACL permita la IP de la PC
```cisco
R1# show access-lists
```
Busca matches en la ACL
Verificar que ip nat inside source esté configurado
```cisco
R1# show running-config | include ip nat inside source
```

### PC-Externa no puede llegar al servidor interno

Verificar que el NAT estático exista
```cisco
R1# show ip nat translations
```
Debe aparecer la línea estática sin puertos
Verificar que el servidor tenga el gateway correcto
El gateway debe ser 192.168.1.1 (R1)
Si el servidor no tiene gateway no puede responder
Verificar que ip nat outside esté en G0/1
```cisco
R1# show running-config | section interface GigabitEthernet0/1
```


### Error al crear el pool NAT
%Bad mask /29 for address 209.165.200.4
Solución: usar netmask en lugar de prefix-length
```cisco
R1(config)# ip nat pool POOL-PUBLICO 209.165.200.4 209.165.200.5
netmask 255.255.255.248
```
---

## Comparativa de tipos de NAT

| Tipo | IPs privadas | IPs públicas | Puertos | Uso típico |
|------|-------------|-------------|---------|------------|
| Estático | 1 | 1 fija | No | Servidores internos |
| Dinámico | Muchas | Pool limitado | No | Grupos de usuarios |
| PAT | Muchas | 1 | Sí | Red doméstica o empresa |

---

## Conceptos aplicados en este lab

| Concepto | Dónde se aplica |
|----------|----------------|
| ip nat inside | Interfaz LAN interna G0/0 |
| ip nat outside | Interfaz WAN G0/1 |
| NAT estático | Servidor web accesible desde internet |
| NAT dinámico | Pool de IPs para PC-Ventas1 y PC-Ventas2 |
| PAT overload | Toda la red interna con una IP pública |
| Inside Local | IP privada del dispositivo interno |
| Inside Global | IP pública que representa al dispositivo |
| show ip nat translations | Ver traducciones activas |
| clear ip nat translation | Limpiar traducciones dinámicas |

---
