# Lab 06 — Enrutamiento Estático

## Información del lab

| Campo | Detalle |
|-------|---------|
| Tema CCNA | Conectividad IP — Dominio 3.1 y 3.2 |
| Dificultad | ⭐⭐ Intermedio |
| Duración estimada | 45 minutos |
| Archivo | lab-06-enrutamiento-estatico.pkt |
| Teoría relacionada | [teoria/03-conectividad-ip/enrutamiento-ospf.md](../../teoria/03-conectividad-ip/enrutamiento-ospf.md) |

---

## Objetivo

Configurar rutas estáticas, rutas estáticas por defecto y rutas
estáticas flotantes en una topología de tres routers. Verificar
la tabla de enrutamiento y la conectividad extremo a extremo.
Entender cuándo usar cada tipo de ruta estática.

---

## Escenario

**TechStart S.A.** tiene tres sucursales conectadas en serie.
La sucursal central (R2) conecta la sucursal norte (R1) con
la sucursal sur (R3). Cada sucursal tiene su propia LAN.
Todo el enrutamiento se hará de forma estática.

---

## Topología
LAN-Norte          WAN 1           WAN 2          LAN-Sur
192.168.1.0/24  10.0.0.0/30   10.0.1.0/30   192.168.3.0/24
PC-Norte ── SW1 ── R1 ──────────── R2 ──────────── R3 ── SW3 ── PC-Sur
G0/0  G0/1      G0/0  G0/1      G0/0  G0/1
.1    .1  .2    .2    .5  .6    .6    .1
(R1-R2)              (R2-R3)
                          R2 también tiene:
                          LAN-Central 192.168.2.0/24
                          G0/2 → .1 → SW2 → PC-Central

---

## Tabla de direccionamiento

| Dispositivo | Interfaz | Dirección IP | Máscara | Descripción |
|-------------|----------|-------------|---------|-------------|
| R1 | G0/0 | 192.168.1.1 | 255.255.255.0 | LAN Norte |
| R1 | G0/1 | 10.0.0.1 | 255.255.255.252 | WAN hacia R2 |
| R2 | G0/0 | 10.0.0.2 | 255.255.255.252 | WAN hacia R1 |
| R2 | G0/1 | 10.0.1.5 | 255.255.255.252 | WAN hacia R3 |
| R2 | G0/2 | 192.168.2.1 | 255.255.255.0 | LAN Central |
| R3 | G0/0 | 10.0.1.6 | 255.255.255.252 | WAN hacia R2 |
| R3 | G0/1 | 192.168.3.1 | 255.255.255.0 | LAN Sur |
| PC-Norte | NIC | 192.168.1.10 | 255.255.255.0 | 192.168.1.1 |
| PC-Central | NIC | 192.168.2.10 | 255.255.255.0 | 192.168.2.1 |
| PC-Sur | NIC | 192.168.3.10 | 255.255.255.0 | 192.168.3.1 |

---

## Instrucciones

### Parte 1 — Configurar interfaces de los routers

#### Configurar R1
```cisco
Router> enable
Router# configure terminal
Router(config)# hostname R1
R1(config)# interface GigabitEthernet0/0
R1(config-if)# description LAN-Norte
R1(config-if)# ip address 192.168.1.1 255.255.255.0
R1(config-if)# no shutdown
R1(config-if)# exit
R1(config)# interface GigabitEthernet0/1
R1(config-if)# description WAN-hacia-R2
R1(config-if)# ip address 10.0.0.1 255.255.255.252
R1(config-if)# no shutdown
R1(config-if)# exit
R1(config)# end
R1# copy running-config startup-config
```

#### Configurar R2
```cisco
Router> enable
Router# configure terminal
Router(config)# hostname R2
R2(config)# interface GigabitEthernet0/0
R2(config-if)# description WAN-hacia-R1
R2(config-if)# ip address 10.0.0.2 255.255.255.252
R2(config-if)# no shutdown
R2(config-if)# exit
R2(config)# interface GigabitEthernet0/1
R2(config-if)# description WAN-hacia-R3
R2(config-if)# ip address 10.0.1.5 255.255.255.252
R2(config-if)# no shutdown
R2(config-if)# exit
R2(config)# interface GigabitEthernet0/2
R2(config-if)# description LAN-Central
R2(config-if)# ip address 192.168.2.1 255.255.255.0
R2(config-if)# no shutdown
R2(config-if)# exit
R2(config)# end
R2# copy running-config startup-config
```

#### Configurar R3
```cisco
Router> enable
Router# configure terminal
Router(config)# hostname R3
R3(config)# interface GigabitEthernet0/0
R3(config-if)# description WAN-hacia-R2
R3(config-if)# ip address 10.0.1.6 255.255.255.252
R3(config-if)# no shutdown
R3(config-if)# exit
R3(config)# interface GigabitEthernet0/1
R3(config-if)# description LAN-Sur
R3(config-if)# ip address 192.168.3.1 255.255.255.0
R3(config-if)# no shutdown
R3(config-if)# exit
R3(config)# end
R3# copy running-config startup-config
```

---

### Parte 2 — Configurar rutas estáticas

En este punto los routers solo conocen sus redes directamente
conectadas. Necesitan rutas estáticas para llegar a las redes remotas.

#### Rutas estáticas en R1

R1 necesita llegar a tres redes que no conoce:
- 192.168.2.0/24 (LAN Central) vía 10.0.0.2
- 192.168.3.0/24 (LAN Sur) vía 10.0.0.2
- 10.0.1.0/30 (WAN R2-R3) vía 10.0.0.2
```cisco
R1(config)# ip route 192.168.2.0 255.255.255.0 10.0.0.2
R1(config)# ip route 192.168.3.0 255.255.255.0 10.0.0.2
R1(config)# ip route 10.0.1.0 255.255.255.252 10.0.0.2
```

#### Rutas estáticas en R2

R2 necesita llegar a:
- 192.168.1.0/24 (LAN Norte) vía 10.0.0.1
- 192.168.3.0/24 (LAN Sur) vía 10.0.1.6

```cisco
R2(config)# ip route 192.168.1.0 255.255.255.0 10.0.0.1
R2(config)# ip route 192.168.3.0 255.255.255.0 10.0.1.6
```

#### Rutas estáticas en R3

R3 necesita llegar a:
- 192.168.1.0/24 (LAN Norte) vía 10.0.1.5
- 192.168.2.0/24 (LAN Central) vía 10.0.1.5
- 10.0.0.0/30 (WAN R1-R2) vía 10.0.1.5
```cisco
R3(config)# ip route 192.168.1.0 255.255.255.0 10.0.1.5
R3(config)# ip route 192.168.2.0 255.255.255.0 10.0.1.5
R3(config)# ip route 10.0.0.0 255.255.255.252 10.0.1.5
```

#### Guardar configuración
```cisco
R1# copy running-config startup-config
R2# copy running-config startup-config
R3# copy running-config startup-config
```
---

### Parte 3 — Ruta estática por defecto

Simula que R1 tiene salida a internet a través de R2.
La ruta por defecto envía todo el tráfico desconocido hacia R2.
```cisco
R1(config)# ip route 0.0.0.0 0.0.0.0 10.0.0.2
```

Para que R2 propague esta información a R3 también necesita
una ruta por defecto o una ruta específica hacia internet:
```cisco
R2(config)# ip route 0.0.0.0 0.0.0.0 10.0.0.1
R3(config)# ip route 0.0.0.0 0.0.0.0 10.0.1.5
```
Verificar que aparezca como candidata en la tabla:
```cisco
R1# show ip route
```
Busca la línea:
S*   0.0.0.0/0 [1/0] via 10.0.0.2

---

### Parte 4 — Ruta estática flotante

Simula un enlace de respaldo entre R1 y R3 directo
(por ejemplo un enlace serial de respaldo).
La ruta flotante tiene AD mayor que la principal
y solo se activa si la ruta principal desaparece.

Agrega una ruta alternativa con AD 5 (la principal tiene AD 1):
```cisco
R1(config)# ip route 192.168.3.0 255.255.255.0 10.0.0.2 1
R1(config)# ip route 192.168.3.0 255.255.255.0 10.0.1.5 5
```
> La segunda ruta (AD=5) no aparecerá en la tabla mientras
> la primera (AD=1) esté activa. Solo se activa si la primera falla.

Para probar la ruta flotante en Packet Tracer:
1. Desconecta el enlace entre R1 y R2
2. Ejecuta `show ip route` en R1
3. Verás que la ruta flotante (AD=5) ahora aparece activa

---

## Verificación

### Verificar tablas de enrutamiento
```cisco
R1# show ip route
R2# show ip route
R3# show ip route
```
Resultado esperado en R1:
Codes: C - connected, S - static, S* - candidate default
Gateway of last resort is 10.0.0.2 to network 0.0.0.0
  10.0.0.0/30 is subnetted, 2 subnets
C        10.0.0.0/30 is directly connected, GigabitEthernet0/1
S        10.0.1.0/30 [1/0] via 10.0.0.2
192.168.1.0/24 is directly connected, GigabitEthernet0/0
S     192.168.2.0/24 [1/0] via 10.0.0.2
S     192.168.3.0/24 [1/0] via 10.0.0.2
S*    0.0.0.0/0 [1/0] via 10.0.0.2

### Verificar solo rutas estáticas
```cisco
R1# show ip route static
R2# show ip route static
R3# show ip route static
```
### Verificar interfaces
```cisco
R1# show ip interface brief
R2# show ip interface brief
R3# show ip interface brief
```
### Pruebas de conectividad

Desde PC-Norte hacer ping a todas las redes:
```bash
ping 192.168.1.1    <- Gateway R1
ping 192.168.2.10   <- PC-Central
ping 192.168.3.10   <- PC-Sur
ping 10.0.0.2       <- R2 WAN
ping 10.0.1.6       <- R3 WAN
```

Desde PC-Sur hacer ping hacia atrás:
```bash
ping 192.168.3.1    <- Gateway R3
ping 192.168.2.10   <- PC-Central
ping 192.168.1.10   <- PC-Norte
```
Verificar el camino con traceroute:
PC-Norte> tracert 192.168.3.10

Resultado esperado:
```bash
1   192.168.1.1    <- R1 LAN
2   10.0.0.2       <- R2 WAN hacia R1
3   10.0.1.6       <- R3 WAN hacia R2
4   192.168.3.10   <- PC-Sur
```
### Verificar ruta flotante
```cisco
R1# show ip route 192.168.3.0
```
Mientras el enlace principal esté activo:
S    192.168.3.0/24 [1/0] via 10.0.0.2

Después de desconectar el enlace R1-R2:
S    192.168.3.0/24 [5/0] via 10.0.1.5

### Lista de verificación

- [ ] R1 tiene interfaces G0/0 y G0/1 up/up
- [ ] R2 tiene interfaces G0/0, G0/1 y G0/2 up/up
- [ ] R3 tiene interfaces G0/0 y G0/1 up/up
- [ ] R1 tiene rutas estáticas hacia 192.168.2.0, 192.168.3.0 y 10.0.1.0
- [ ] R2 tiene rutas estáticas hacia 192.168.1.0 y 192.168.3.0
- [ ] R3 tiene rutas estáticas hacia 192.168.1.0, 192.168.2.0 y 10.0.0.0
- [ ] Ruta por defecto S* aparece en show ip route de R1
- [ ] PC-Norte hace ping a PC-Central
- [ ] PC-Norte hace ping a PC-Sur
- [ ] PC-Sur hace ping a PC-Norte
- [ ] Traceroute muestra el camino correcto de 4 saltos
- [ ] Ruta flotante se activa al desconectar enlace principal
- [ ] Configuración guardada en los tres routers

---

## Troubleshooting

### Ping falla entre PCs de distintas sucursales

Verificar que las interfaces estén up/up en ambos extremos
```cisco
R1# show ip interface brief
```
Verificar que exista la ruta en la tabla
```cisco
R1# show ip route 192.168.3.0
```
Verificar que la ruta de regreso también exista
```cisco
R3# show ip route 192.168.1.0
```
Hacer ping desde el router hacia el siguiente salto
```cisco
R1# ping 10.0.0.2
```
Hacer ping desde el router hacia la red destino
```cisco
R1# ping 192.168.3.1
```

### La ruta no aparece en show ip route

Verificar que el next-hop sea alcanzable
La IP del next-hop debe estar en una red directamente conectada
Verificar typos en la red destino o máscara
```cisco
R1# show running-config | include ip route
```
Borrar la ruta incorrecta y reconfigurar
```cisco
R1(config)# no ip route 192.168.3.0 255.255.255.0 10.0.0.2
R1(config)# ip route 192.168.3.0 255.255.255.0 10.0.0.2
```

### La ruta flotante no aparece después de desconectar el enlace

Verificar que la ruta flotante esté configurada
```cisco
R1# show running-config | include ip route
```
Verificar que la AD de la flotante sea mayor que la principal
La principal debe tener AD 1 y la flotante AD mayor (ej: 5)
Verificar que el next-hop de la flotante sea alcanzable
```cisco
R1# ping 10.0.1.5
```

---

## Conceptos aplicados en este lab

| Concepto | Dónde se aplica |
|----------|----------------|
| Ruta estática básica | Conectividad entre las tres sucursales |
| Next-hop | IP del siguiente router en cada ruta |
| Ruta por defecto | Tráfico a internet desde R1 |
| Ruta flotante | Respaldo automático con AD mayor |
| Distancia Administrativa | AD 1 principal, AD 5 flotante |
| Tabla de enrutamiento | show ip route en cada router |
| Traceroute | Verificar el camino completo |
