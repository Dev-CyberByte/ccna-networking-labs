# Lab 10 — ACLs Estándar y Extendidas

## Información del lab

| Campo | Detalle |
|-------|---------|
| Tema CCNA | Seguridad — Dominio 5.1 y 5.2 |
| Dificultad | ⭐⭐⭐ Avanzado |
| Duración estimada | 60 minutos |
| Archivo | lab-10-acl.pkt |
| Teoría relacionada | [teoria/05-seguridad/acl-ssh-hardening.md](../../teoria/05-seguridad/acl-ssh-hardening.md) |
| Lab anterior | [Lab 09 — NAT y PAT](../09-nat/README.md) |

---

## Objetivo

Configurar ACLs estándar y extendidas para controlar el tráfico
entre departamentos, proteger servidores y restringir acceso
por protocolo y puerto. Verificar el comportamiento del deny
implícito y los contadores de matches en cada regla.

---

## Escenario

**TechStart S.A.** tiene tres departamentos en VLANs separadas
y un servidor de administración. El equipo de seguridad definió
las siguientes políticas de red:

### Políticas de seguridad

| # | Política | Tipo ACL |
|---|----------|---------|
| 1 | RRHH no puede acceder a la red de TI | Estándar |
| 2 | Solo Gerencia puede acceder al servidor de administración | Estándar |
| 3 | Ventas solo puede usar HTTP y HTTPS hacia internet | Extendida |
| 4 | Nadie puede hacer ping al router desde fuera de la red de TI | Extendida |
| 5 | TI puede acceder a todo | Extendida |

---

## Topología
PC-RRHH          PC-Ventas        PC-TI           PC-Gerencia
192.168.30.10    192.168.10.10    192.168.20.10   192.168.40.10
|                |               |                |
Fa0/3           Fa0/1           Fa0/2            Fa0/4
└───────────────┴───────────────┴────────────────┘
SW1
(G0/1 trunk)
|
G0/0
R1
G0/1
|
Servidor-Admin
192.168.99.10
(VLAN 99)

---

## Tabla de direccionamiento

| Dispositivo | Interfaz | Dirección IP | Máscara | VLAN |
|-------------|----------|-------------|---------|------|
| R1 | G0/0.10 | 192.168.10.1 | 255.255.255.0 | 10 |
| R1 | G0/0.20 | 192.168.20.1 | 255.255.255.0 | 20 |
| R1 | G0/0.30 | 192.168.30.1 | 255.255.255.0 | 30 |
| R1 | G0/0.40 | 192.168.40.1 | 255.255.255.0 | 40 |
| R1 | G0/1 | 192.168.99.1 | 255.255.255.0 | — |
| Servidor-Admin | NIC | 192.168.99.10 | 255.255.255.0 | — |
| PC-Ventas | NIC | 192.168.10.10 | 255.255.255.0 | 10 |
| PC-TI | NIC | 192.168.20.10 | 255.255.255.0 | 20 |
| PC-RRHH | NIC | 192.168.30.10 | 255.255.255.0 | 30 |
| PC-Gerencia | NIC | 192.168.40.10 | 255.255.255.0 | 40 |

---

## Instrucciones

### Parte 1 — Configurar R1 base

#### Paso 1 — Subinterfaces y VLANs
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
R1(config)# interface GigabitEthernet0/0.30
R1(config-subif)# description Gateway-RRHH
R1(config-subif)# encapsulation dot1Q 30
R1(config-subif)# ip address 192.168.30.1 255.255.255.0
R1(config-subif)# exit
R1(config)# interface GigabitEthernet0/0.40
R1(config-subif)# description Gateway-Gerencia
R1(config-subif)# encapsulation dot1Q 40
R1(config-subif)# ip address 192.168.40.1 255.255.255.0
R1(config-subif)# exit
R1(config)# interface GigabitEthernet0/1
R1(config-if)# description Servidor-Admin
R1(config-if)# ip address 192.168.99.1 255.255.255.0
R1(config-if)# no shutdown
R1(config-if)# exit
R1(config)# end
R1# copy running-config startup-config
```
---

### Parte 2 — ACL Estándar: bloquear RRHH hacia TI

ACL estándar filtra solo por IP de origen.
Se coloca cerca del destino (subinterfaz de TI).

#### Política 1 — RRHH no puede acceder a la red de TI
```cisco
R1(config)# ip access-list standard BLOQUEAR-RRHH-A-TI
R1(config-std-nacl)# remark Bloquear RRHH hacia red TI
R1(config-std-nacl)# deny 192.168.30.0 0.0.0.255
R1(config-std-nacl)# permit any
R1(config-std-nacl)# exit
```

Aplicar en la subinterfaz de TI en dirección outbound:
```cisco
R1(config)# interface GigabitEthernet0/0.20
R1(config-subif)# ip access-group BLOQUEAR-RRHH-A-TI out
R1(config-subif)# exit
```
> Se aplica outbound porque el tráfico de RRHH hacia TI
> sale por la subinterfaz de TI (.20).
> ACL estándar cerca del destino para no bloquear
> el acceso de RRHH a otras redes.

---

### Parte 3 — ACL Estándar: proteger servidor de administración

#### Política 2 — Solo Gerencia accede al servidor admin
```cisco
R1(config)# ip access-list standard SOLO-GERENCIA-ADMIN
R1(config-std-nacl)# remark Solo Gerencia puede acceder al servidor admin
R1(config-std-nacl)# permit 192.168.40.0 0.0.0.255
R1(config-std-nacl)# deny any
R1(config-std-nacl)# exit
```

Aplicar en G0/1 en dirección outbound:
```cisco
R1(config)# interface GigabitEthernet0/1
R1(config-if)# ip access-group SOLO-GERENCIA-ADMIN out
R1(config-if)# exit
```

> Aquí el deny any final es explícito pero redundante
> porque ya existe el deny implícito.
> Es buena práctica escribirlo para que aparezca
> en los contadores de matches.

---

### Parte 4 — ACL Extendida: restringir tráfico de Ventas

ACL extendida filtra por IP origen, IP destino, protocolo y puerto.
Se coloca cerca del origen (subinterfaz de Ventas).

#### Política 3 — Ventas solo puede usar HTTP y HTTPS
```cisco
R1(config)# ip access-list extended POLITICA-VENTAS
R1(config-ext-nacl)# remark Permitir HTTP desde Ventas
R1(config-ext-nacl)# permit tcp 192.168.10.0 0.0.0.255 any eq 80
R1(config-ext-nacl)# remark Permitir HTTPS desde Ventas
R1(config-ext-nacl)# permit tcp 192.168.10.0 0.0.0.255 any eq 443
R1(config-ext-nacl)# remark Permitir DNS para resolución de nombres
R1(config-ext-nacl)# permit udp 192.168.10.0 0.0.0.255 any eq 53
R1(config-ext-nacl)# remark Bloquear todo lo demás de Ventas
R1(config-ext-nacl)# deny ip 192.168.10.0 0.0.0.255 any
R1(config-ext-nacl)# remark Permitir el resto del tráfico
R1(config-ext-nacl)# permit ip any any
R1(config-ext-nacl)# exit
```

Aplicar en la subinterfaz de Ventas en dirección inbound:
```cisco
R1(config)# interface GigabitEthernet0/0.10
R1(config-subif)# ip access-group POLITICA-VENTAS in
R1(config-subif)# exit
```

> Se aplica inbound en la subinterfaz de Ventas (.10)
> para detener el tráfico no permitido lo antes posible.
> ACL extendida cerca del origen.

---

### Parte 5 — ACL Extendida: proteger el router de pings

#### Política 4 — Solo TI puede hacer ping al router
```cisco
R1(config)# ip access-list extended PROTEGER-ROUTER
R1(config-ext-nacl)# remark Permitir ping desde TI al router
R1(config-ext-nacl)# permit icmp 192.168.20.0 0.0.0.255 host 192.168.20.1
R1(config-ext-nacl)# remark Bloquear ping desde cualquier otra red al router
R1(config-ext-nacl)# deny icmp any host 192.168.10.1
R1(config-ext-nacl)# deny icmp any host 192.168.20.1
R1(config-ext-nacl)# deny icmp any host 192.168.30.1
R1(config-ext-nacl)# deny icmp any host 192.168.40.1
R1(config-ext-nacl)# deny icmp any host 192.168.99.1
R1(config-ext-nacl)# remark Permitir todo el tráfico no ICMP
R1(config-ext-nacl)# permit ip any any
R1(config-ext-nacl)# exit
```

Aplicar en todas las subinterfaces inbound excepto TI:
```cisco
R1(config)# interface GigabitEthernet0/0.30
R1(config-subif)# ip access-group PROTEGER-ROUTER in
R1(config-subif)# exit
R1(config)# interface GigabitEthernet0/0.40
R1(config-subif)# ip access-group PROTEGER-ROUTER in
R1(config-subif)# exit
```
---

### Parte 6 — ACL Extendida: TI tiene acceso total

#### Política 5 — TI puede acceder a todo sin restricciones

TI no tiene ninguna ACL aplicada en su subinterfaz.
Al no haber ACL todo el tráfico de TI pasa libremente.
Esta es la política por omisión y no requiere configuración.

Si en algún momento se aplicó una ACL a TI por error:
```cisco
R1(config)# interface GigabitEthernet0/0.20
R1(config-subif)# no ip access-group NOMBRE-ACL in
R1(config-subif)# exit
```

#### Guardar configuración
```cisco
R1(config)# end
R1# copy running-config startup-config
```
---

## Verificación

### Ver todas las ACLs configuradas
R1# show access-lists

Resultado esperado:
```bash
Standard IP access list BLOQUEAR-RRHH-A-TI
10 deny 192.168.30.0 0.0.0.255 (25 matches)
20 permit any (142 matches)
Standard IP access list SOLO-GERENCIA-ADMIN
10 permit 192.168.40.0 0.0.0.255 (18 matches)
20 deny any (7 matches)
Extended IP access list POLITICA-VENTAS
10 permit tcp 192.168.10.0 0.0.0.255 any eq www (45 matches)
20 permit tcp 192.168.10.0 0.0.0.255 any eq 443 (38 matches)
30 permit udp 192.168.10.0 0.0.0.255 any eq domain (12 matches)
40 deny ip 192.168.10.0 0.0.0.255 any (9 matches)
50 permit ip any any (201 matches)
Extended IP access list PROTEGER-ROUTER
10 permit icmp 192.168.20.0 0.0.0.255 host 192.168.20.1 (4 matches)
20 deny icmp any host 192.168.10.1 (3 matches)
30 deny icmp any host 192.168.30.1 (2 matches)
40 permit ip any any (89 matches)
```

> Los matches confirman que las reglas están siendo evaluadas.
> Si una regla tiene 0 matches y debería tener más,
> hay un problema de orden o de dirección de aplicación.

### Ver ACLs aplicadas en las interfaces
```cisco
R1# show ip interface GigabitEthernet0/0.10
R1# show ip interface GigabitEthernet0/0.20
R1# show ip interface GigabitEthernet0/0.30
R1# show ip interface GigabitEthernet0/1
```

Busca las líneas:
Inbound  access list is POLITICA-VENTAS
Outbound access list is BLOQUEAR-RRHH-A-TI

### Pruebas de conectividad por política

#### Política 1 — RRHH no puede acceder a TI
```bash
PC-RRHH> ping 192.168.20.10      <- debe FALLAR
PC-RRHH> ping 192.168.10.10      <- debe funcionar (RRHH a Ventas)
PC-RRHH> ping 192.168.40.10      <- debe funcionar (RRHH a Gerencia)
```

#### Política 2 — Solo Gerencia accede al servidor admin
```bash
PC-Gerencia> ping 192.168.99.10  <- debe funcionar
PC-Ventas>   ping 192.168.99.10  <- debe FALLAR
PC-TI>       ping 192.168.99.10  <- debe FALLAR
PC-RRHH>     ping 192.168.99.10  <- debe FALLAR
```

#### Política 3 — Ventas solo HTTP y HTTPS
```bash
PC-Ventas> ping 192.168.20.10    <- debe FALLAR (ICMP bloqueado)
PC-Ventas> ping 192.168.30.10    <- debe FALLAR (ICMP bloqueado)

En Packet Tracer usar el navegador web desde PC-Ventas:
http://192.168.99.10             <- debe funcionar (HTTP puerto 80)
https://192.168.99.10            <- debe funcionar (HTTPS puerto 443)
```

#### Política 4 — Solo TI puede hacer ping al router
```bash
PC-TI>       ping 192.168.20.1   <- debe funcionar
PC-RRHH>     ping 192.168.30.1   <- debe FALLAR
PC-Ventas>   ping 192.168.10.1   <- debe FALLAR (por POLITICA-VENTAS)
PC-Gerencia> ping 192.168.40.1   <- debe FALLAR (por PROTEGER-ROUTER)
```

#### Política 5 — TI tiene acceso total
```bash
PC-TI> ping 192.168.10.10        <- debe funcionar
PC-TI> ping 192.168.30.10        <- debe funcionar
PC-TI> ping 192.168.40.10        <- debe funcionar
PC-TI> ping 192.168.99.10        <- debe FALLAR (bloqueado por SOLO-GERENCIA-ADMIN)
```

> TI no puede acceder al servidor admin porque la ACL
> SOLO-GERENCIA-ADMIN se aplica en la salida de G0/1
> y bloquea todo lo que no sea Gerencia.
> Esto es correcto según la política 2.

### Lista de verificación

- [ ] ACL BLOQUEAR-RRHH-A-TI aplicada outbound en G0/0.20
- [ ] ACL SOLO-GERENCIA-ADMIN aplicada outbound en G0/1
- [ ] ACL POLITICA-VENTAS aplicada inbound en G0/0.10
- [ ] ACL PROTEGER-ROUTER aplicada inbound en G0/0.30 y G0/0.40
- [ ] PC-RRHH no puede hacer ping a PC-TI
- [ ] PC-Gerencia puede hacer ping al Servidor-Admin
- [ ] PC-Ventas no puede hacer ping (ICMP bloqueado)
- [ ] PC-TI puede hacer ping a todas las redes excepto Servidor-Admin
- [ ] show access-lists muestra matches en todas las reglas
- [ ] Configuración guardada en R1

---

## Troubleshooting

### Una regla tiene 0 matches cuando debería tener más

Verificar que la ACL esté aplicada en la interfaz correcta
```cisco
R1# show ip interface GigabitEthernet0/0.10
```
Verificar la dirección (in/out)
El tráfico que viene de la PC entra (in) por la subinterfaz
El tráfico que va hacia la PC sale (out) por la subinterfaz
Verificar que una regla anterior no esté capturando el tráfico
Las reglas se evalúan de arriba hacia abajo
La primera coincidencia gana
Limpiar los contadores y hacer prueba de nuevo
```cisco
R1# clear ip access-list counters
```

### El tráfico permitido está siendo bloqueado

Verificar el orden de las reglas en la ACL
```cisco
R1# show access-lists NOMBRE-ACL
```
Recuerda: primera coincidencia gana
Si hay un deny antes del permit, el deny gana
Verificar que el deny implícito no esté bloqueando
Agregar permit ip any any al final si falta
Verificar la dirección de aplicación
Una ACL inbound evalúa el tráfico que entra al router
por esa interfaz, no el que sale


### No se puede modificar una ACL numerada
Las ACLs numeradas no permiten insertar reglas en el medio.
Debes borrarla completa y reescribirla:
```cisco
R1(config)# no access-list 100
R1(config)# access-list 100 permit ...
```
Para evitar esto usa ACLs nombradas que sí permiten
agregar y eliminar reglas individuales:
```cisco
R1(config)# ip access-list extended POLITICA-VENTAS
R1(config-ext-nacl)# no 40
R1(config-ext-nacl)# 40 deny tcp 192.168.10.0 0.0.0.255 any eq 23
```

### La ACL bloquea tráfico que no debería bloquear

Verificar el deny implícito al final de toda ACL
Si el tráfico no coincide con ningún permit, se descarta
Agregar un permit ip any any al final de la ACL
para permitir el resto del tráfico no especificado
Verificar con show access-lists cuál regla tiene matches
inesperados


---

## Conceptos aplicados en este lab

| Concepto | Dónde se aplica |
|----------|----------------|
| ACL estándar | Políticas 1 y 2, filtro por IP origen |
| ACL extendida | Políticas 3 y 4, filtro por protocolo y puerto |
| Deny implícito | Final de todas las ACLs |
| ACL nombrada | Todas las ACLs de este lab |
| inbound | ACLs de Ventas y RRHH hacia el router |
| outbound | ACLs hacia TI y hacia Servidor-Admin |
| remark | Comentarios en cada regla para documentar |
| Matches | Contadores que confirman que las reglas funcionan |
| near source | ACLs extendidas aplicadas cerca del origen |
| near destination | ACLs estándar aplicadas cerca del destino |

---

## Capturas requeridas

| Archivo | Contenido |
|---------|-----------|
| topologia.png | Vista general de la topología |
| show-access-lists.png | show access-lists con todos los matches |
| politica1-rrhh-ti.png | Ping fallido de PC-RRHH a PC-TI |
| politica2-gerencia-admin.png | Ping exitoso de PC-Gerencia al servidor |
| politica2-ventas-admin.png | Ping fallido de PC-Ventas al servidor |
| politica3-ventas-http.png | HTTP exitoso desde PC-Ventas |
| politica4-ping-router.png | Ping fallido de PC-RRHH al router |
| politica5-ti-acceso.png | Ping exitoso de PC-TI a todas las redes |
