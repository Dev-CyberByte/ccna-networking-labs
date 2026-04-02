# Lab 02 — Subnetting y VLSM

## Información del lab

| Campo | Detalle |
|-------|---------|
| Tema CCNA | Fundamentos de red — Dominio 1.6 y 1.7 |
| Dificultad | ⭐⭐ Intermedio |
| Duración estimada | 45 minutos |
| Archivo | lab-02-subnetting.pkt |
| Teoría relacionada | [teoria/01-fundamentos-de-red/subnetting.md](../../teoria/01-fundamentos-de-red/subnetting.md) |

---

## Objetivo

Diseñar un esquema de direccionamiento IP usando VLSM para una red
empresarial con múltiples departamentos, implementarlo en Packet Tracer
y verificar conectividad extremo a extremo entre todas las subredes.

---

## Escenario

La empresa **TechStart S.A.** tiene una sede con 4 áreas y dos
enlaces WAN entre routers. Se te entrega el bloque **192.168.100.0/24**
y debes dividirlo eficientemente usando VLSM.

### Requerimientos de hosts

| Área | Hosts requeridos |
|------|-----------------|
| Ventas | 50 hosts |
| TI | 25 hosts |
| RRHH | 10 hosts |
| Gerencia | 5 hosts |
| Enlace WAN R1-R2 | 2 hosts |
| Enlace WAN R2-R3 | 2 hosts |

---

## Topología
```
Ventas(50)  TI(25)              RRHH(10)  Gerencia(5)
    │          │                    │          │
  Fa0/1      Fa0/2              Fa0/1      Fa0/2
    └────┬────┘                  └────┬────┘
        SW1                          SW2
         │                            │
        G0/0                         G0/0
         R1 ────── WAN ────────────── R2 ────── WAN ────── R3
        G0/1                         G0/1
    (Enlace R1-R2)              (Enlace R2-R3)
```

---

## Parte 1 — Diseño de subredes con VLSM

Antes de tocar Packet Tracer debes resolver el direccionamiento
en papel o en este README. Siempre empieza por la subred más grande.

### Paso 1 — Ordenar por tamaño (de mayor a menor)

| # | Área | Hosts requeridos | Hosts necesarios (2^n - 2 ≥ requeridos) | Prefijo |
|---|------|-----------------|----------------------------------------|---------|
| 1 | Ventas | 50 | 2^6 = 64 → 62 hosts útiles | /26 |
| 2 | TI | 25 | 2^5 = 32 → 30 hosts útiles | /27 |
| 3 | RRHH | 10 | 2^4 = 16 → 14 hosts útiles | /28 |
| 4 | Gerencia | 5 | 2^3 = 8 → 6 hosts útiles | /29 |
| 5 | WAN R1-R2 | 2 | 2^2 = 4 → 2 hosts útiles | /30 |
| 6 | WAN R2-R3 | 2 | 2^2 = 4 → 2 hosts útiles | /30 |

### Paso 2 — Asignar subredes en orden

#### Subred 1 — Ventas (/26, bloque 64)
```
Red:             192.168.100.0/26
Máscara:         255.255.255.192
Primer host:     192.168.100.1
Último host:     192.168.100.62
Broadcast:       192.168.100.63
Hosts útiles:    62
```

#### Subred 2 — TI (/27, bloque 32)
```
Red:             192.168.100.64/27
Máscara:         255.255.255.224
Primer host:     192.168.100.65
Último host:     192.168.100.94
Broadcast:       192.168.100.95
Hosts útiles:    30
```

#### Subred 3 — RRHH (/28, bloque 16)
```
Red:             192.168.100.96/28
Máscara:         255.255.255.240
Primer host:     192.168.100.97
Último host:     192.168.100.110
Broadcast:       192.168.100.111
Hosts útiles:    14
```

#### Subred 4 — Gerencia (/29, bloque 8)
```
Red:             192.168.100.112/29
Máscara:         255.255.255.248
Primer host:     192.168.100.113
Último host:     192.168.100.118
Broadcast:       192.168.100.119
Hosts útiles:    6
```

#### Subred 5 — WAN R1-R2 (/30, bloque 4)
```
Red:             192.168.100.120/30
Máscara:         255.255.255.252
Primer host:     192.168.100.121
Último host:     192.168.100.122
Broadcast:       192.168.100.123
Hosts útiles:    2
```

#### Subred 6 — WAN R2-R3 (/30, bloque 4)
```
Red:             192.168.100.124/30
Máscara:         255.255.255.252
Primer host:     192.168.100.124
Último host:     192.168.100.126
Broadcast:       192.168.100.127
Hosts útiles:    2
```

### Resumen de direccionamiento

| Subred | Red | Máscara | Gateway | Broadcast | Hosts útiles |
|--------|-----|---------|---------|-----------|--------------|
| Ventas | 192.168.100.0/26 | 255.255.255.192 | 192.168.100.1 | 192.168.100.63 | 62 |
| TI | 192.168.100.64/27 | 255.255.255.224 | 192.168.100.65 | 192.168.100.95 | 30 |
| RRHH | 192.168.100.96/28 | 255.255.255.240 | 192.168.100.97 | 192.168.100.111 | 14 |
| Gerencia | 192.168.100.112/29 | 255.255.255.248 | 192.168.100.113 | 192.168.100.119 | 6 |
| WAN R1-R2 | 192.168.100.120/30 | 255.255.255.252 | — | 192.168.100.123 | 2 |
| WAN R2-R3 | 192.168.100.124/30 | 255.255.255.252 | — | 192.168.100.127 | 2 |

Espacio usado: .0 → .127 (128 de 256 direcciones)
Espacio libre: 192.168.100.128 → 192.168.100.255

---

## Tabla de direccionamiento completa

| Dispositivo | Interfaz | Dirección IP | Máscara | Gateway |
|-------------|----------|-------------|---------|---------|
| R1 | G0/0 | 192.168.100.1 | 255.255.255.192 | — |
| R1 | G0/1 | 192.168.100.121 | 255.255.255.252 | — |
| R2 | G0/0 | 192.168.100.65 | 255.255.255.224 | — |
| R2 | G0/1 | 192.168.100.122 | 255.255.255.252 | — |
| R2 | G0/2 | 192.168.100.125 | 255.255.255.252 | — |
| R3 | G0/0 | 192.168.100.97 | 255.255.255.240 | — |
| R3 | G0/1 | 192.168.100.113 | 255.255.255.248 | — |
| R3 | G0/2 | 192.168.100.126 | 255.255.255.252 | — |
| PC-Ventas | NIC | 192.168.100.10 | 255.255.255.192 | 192.168.100.1 |
| PC-TI | NIC | 192.168.100.70 | 255.255.255.224 | 192.168.100.65 |
| PC-RRHH | NIC | 192.168.100.100 | 255.255.255.240 | 192.168.100.97 |
| PC-Gerencia | NIC | 192.168.100.114 | 255.255.255.248 | 192.168.100.113 |

---

## Parte 2 — Implementación en Packet Tracer

### Paso 1 — Armar la topología

Dispositivos necesarios:
- 3 Routers 2911
- 2 Switches 2960
- 4 PCs

Conexiones:
```
PC-Ventas    → SW1 Fa0/1
PC-TI        → SW1 Fa0/2
SW1          → R1  G0/0
R1  G0/1     → R2  G0/1  (enlace WAN R1-R2, cable serial o crossover)
R2  G0/0     → SW2 G0/1
R2  G0/2     → R3  G0/2  (enlace WAN R2-R3)
SW2  Fa0/1   → PC-RRHH
SW2  Fa0/2   → PC-Gerencia
R3  G0/0     → SW2 (si usas un tercer switch) o directo a PCs
```

### Paso 2 — Configurar Router R1
```
Router> enable
Router# configure terminal
Router(config)# hostname R1

R1(config)# interface GigabitEthernet0/0
R1(config-if)# description LAN-Ventas-TI
R1(config-if)# ip address 192.168.100.1 255.255.255.192
R1(config-if)# no shutdown
R1(config-if)# exit

R1(config)# interface GigabitEthernet0/1
R1(config-if)# description WAN-hacia-R2
R1(config-if)# ip address 192.168.100.121 255.255.255.252
R1(config-if)# no shutdown
R1(config-if)# exit

R1(config)# ip route 192.168.100.64 255.255.255.224 192.168.100.122
R1(config)# ip route 192.168.100.96 255.255.255.240 192.168.100.122
R1(config)# ip route 192.168.100.112 255.255.255.248 192.168.100.122
R1(config)# ip route 192.168.100.124 255.255.255.252 192.168.100.122

R1(config)# end
R1# copy running-config startup-config
```

### Paso 3 — Configurar Router R2
```
Router> enable
Router# configure terminal
Router(config)# hostname R2

R2(config)# interface GigabitEthernet0/0
R2(config-if)# description LAN-TI
R2(config-if)# ip address 192.168.100.65 255.255.255.224
R2(config-if)# no shutdown
R2(config-if)# exit

R2(config)# interface GigabitEthernet0/1
R2(config-if)# description WAN-hacia-R1
R2(config-if)# ip address 192.168.100.122 255.255.255.252
R2(config-if)# no shutdown
R2(config-if)# exit

R2(config)# interface GigabitEthernet0/2
R2(config-if)# description WAN-hacia-R3
R2(config-if)# ip address 192.168.100.125 255.255.255.252
R2(config-if)# no shutdown
R2(config-if)# exit

R2(config)# ip route 192.168.100.0 255.255.255.192 192.168.100.121
R2(config)# ip route 192.168.100.96 255.255.255.240 192.168.100.126
R2(config)# ip route 192.168.100.112 255.255.255.248 192.168.100.126

R2(config)# end
R2# copy running-config startup-config
```

### Paso 4 — Configurar Router R3
```
Router> enable
Router# configure terminal
Router(config)# hostname R3

R3(config)# interface GigabitEthernet0/0
R3(config-if)# description LAN-RRHH
R3(config-if)# ip address 192.168.100.97 255.255.255.240
R3(config-if)# no shutdown
R3(config-if)# exit

R3(config)# interface GigabitEthernet0/1
R3(config-if)# description LAN-Gerencia
R3(config-if)# ip address 192.168.100.113 255.255.255.248
R3(config-if)# no shutdown
R3(config-if)# exit

R3(config)# interface GigabitEthernet0/2
R3(config-if)# description WAN-hacia-R2
R3(config-if)# ip address 192.168.100.126 255.255.255.252
R3(config-if)# no shutdown
R3(config-if)# exit

R3(config)# ip route 192.168.100.0 255.255.255.192 192.168.100.125
R3(config)# ip route 192.168.100.64 255.255.255.224 192.168.100.125
R3(config)# ip route 192.168.100.120 255.255.255.252 192.168.100.125

R3(config)# end
R3# copy running-config startup-config
```

### Paso 5 — Configurar las PCs

#### PC-Ventas
```
IP:      192.168.100.10
Máscara: 255.255.255.192
Gateway: 192.168.100.1
```

#### PC-TI
```
IP:      192.168.100.70
Máscara: 255.255.255.224
Gateway: 192.168.100.65
```

#### PC-RRHH
```
IP:      192.168.100.100
Máscara: 255.255.255.240
Gateway: 192.168.100.97
```

#### PC-Gerencia
```
IP:      192.168.100.114
Máscara: 255.255.255.248
Gateway: 192.168.100.113
```

---

## Verificación

### Verificar tablas de enrutamiento
```
R1# show ip route
R2# show ip route
R3# show ip route
```

Resultado esperado en R1:
```
C    192.168.100.0/26 is directly connected, GigabitEthernet0/0
C    192.168.100.120/30 is directly connected, GigabitEthernet0/1
S    192.168.100.64/27 [1/0] via 192.168.100.122
S    192.168.100.96/28 [1/0] via 192.168.100.122
S    192.168.100.112/29 [1/0] via 192.168.100.122
S    192.168.100.124/30 [1/0] via 192.168.100.122
```

### Pruebas de conectividad

Desde PC-Ventas hacer ping a todas las redes:
```
ping 192.168.100.1    ← Gateway Ventas (R1)
ping 192.168.100.70   ← PC-TI
ping 192.168.100.100  ← PC-RRHH
ping 192.168.100.114  ← PC-Gerencia
```

Desde PC-Gerencia hacer ping a todas las redes:
```
ping 192.168.100.113  ← Gateway Gerencia (R3)
ping 192.168.100.10   ← PC-Ventas
ping 192.168.100.70   ← PC-TI
ping 192.168.100.100  ← PC-RRHH
```

Verificar el camino con traceroute:
```
PC-Ventas> tracert 192.168.100.114
```

Resultado esperado:
```
1  192.168.100.1    (R1 G0/0)
2  192.168.100.122  (R2 G0/1)
3  192.168.100.126  (R3 G0/2)
4  192.168.100.114  (PC-Gerencia)
```

### Lista de verificación

- [ ] Diseño VLSM correcto sin solapamiento de subredes
- [ ] R1 tiene ambas interfaces up/up con IPs correctas
- [ ] R2 tiene las tres interfaces up/up con IPs correctas
- [ ] R3 tiene las tres interfaces up/up con IPs correctas
- [ ] Cada router tiene rutas estáticas hacia todas las redes remotas
- [ ] PC-Ventas hace ping a PC-TI
- [ ] PC-Ventas hace ping a PC-RRHH
- [ ] PC-Ventas hace ping a PC-Gerencia
- [ ] Traceroute muestra el camino correcto entre redes
- [ ] Configuración guardada en todos los routers

---

## Troubleshooting

### Ping falla entre subredes distintas
```
1. Verificar que las interfaces estén up/up
   R1# show ip interface brief

2. Verificar que las rutas estáticas existan
   R1# show ip route static

3. Verificar que las IPs y máscaras sean correctas
   R1# show running-config | section interface

4. Probar ping desde el router directamente
   R1# ping 192.168.100.122

5. Verificar que la ruta de regreso exista en el otro router
   R3# show ip route
```

### La máscara de una PC está incorrecta

Si una PC tiene máscara /24 en lugar de /26, el tráfico
no saldrá correctamente porque el host creerá que todos
están en su misma red. Verificar siempre la máscara exacta
de cada subred en la tabla de direccionamiento.

### Traceroute no muestra el camino esperado
```
1. Verificar que no haya rutas redundantes o incorrectas
   R2# show ip route

2. Borrar una ruta incorrecta
   R2(config)# no ip route 192.168.100.0 255.255.255.192 [next-hop]

3. Agregar la ruta correcta
   R2(config)# ip route 192.168.100.0 255.255.255.192 192.168.100.121
```

---

## Conceptos aplicados en este lab

| Concepto | Dónde se aplica |
|----------|----------------|
| VLSM | Diseño del esquema de direccionamiento |
| Subnetting | Cálculo de cada subred por área |
| Rutas estáticas | Conectividad entre routers |
| Tabla de enrutamiento | show ip route en cada router |
| Traceroute | Verificar el camino extremo a extremo |

---

## Capturas requeridas

| Archivo | Contenido |
|---------|-----------|
| topologia.png | Vista general de la topología completa |
| vlsm-diseno.png | Tabla de subredes resuelta |
| r1-route.png | show ip route en R1 |
| r2-route.png | show ip route en R2 |
| r3-route.png | show ip route en R3 |
| ping-ventas-gerencia.png | Ping exitoso PC-Ventas a PC-Gerencia |
| traceroute.png | Traceroute mostrando el camino completo |
