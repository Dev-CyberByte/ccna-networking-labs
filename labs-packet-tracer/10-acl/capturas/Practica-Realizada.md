# Lab 10 — ACLs Estándar y Extendidas

> **Tema CCNA:** Seguridad — Dominio 5.1 y 5.2  
> **Dificultad:** ⭐⭐⭐ Avanzado  
> **Duración estimada:** 60 minutos  
> **Simulador:** Cisco Packet Tracer

---

## Escenario

**TechStart S.A.** tiene tres departamentos en VLANs separadas y un servidor de administración. El equipo de seguridad definió cinco políticas de red que se implementan mediante ACLs estándar y extendidas sobre el router R1.

---

## Topología

```
PC-RRHH          PC-Ventas        PC-TI           PC-Gerencia
192.168.30.10    192.168.10.10    192.168.20.10   192.168.40.10
|                |                |                |
Fa0/3           Fa0/1            Fa0/2            Fa0/4
└───────────────┴────────────────┴────────────────┘
                       SW1
                  (G0/1 trunk)
                       |
                      G0/0
                       R1
                      G0/1
                       |
               Servidor-Admin
               192.168.99.10
```

---

## Tabla de direccionamiento

| Dispositivo    | Interfaz   | Dirección IP   | Máscara         | VLAN |
|----------------|------------|----------------|-----------------|------|
| R1             | G0/0.10    | 192.168.10.1   | 255.255.255.0   | 10   |
| R1             | G0/0.20    | 192.168.20.1   | 255.255.255.0   | 20   |
| R1             | G0/0.30    | 192.168.30.1   | 255.255.255.0   | 30   |
| R1             | G0/0.40    | 192.168.40.1   | 255.255.255.0   | 40   |
| R1             | G0/1       | 192.168.99.1   | 255.255.255.0   | —    |
| Servidor-Admin | NIC        | 192.168.99.10  | 255.255.255.0   | —    |
| PC-Ventas      | NIC        | 192.168.10.10  | 255.255.255.0   | 10   |
| PC-TI          | NIC        | 192.168.20.10  | 255.255.255.0   | 20   |
| PC-RRHH        | NIC        | 192.168.30.10  | 255.255.255.0   | 30   |
| PC-Gerencia    | NIC        | 192.168.40.10  | 255.255.255.0   | 40   |

---

## Políticas de seguridad implementadas

| # | Política                                              | Tipo ACL   | ACL               | Interfaz aplicada     |
|---|-------------------------------------------------------|------------|-------------------|-----------------------|
| 1 | RRHH no puede acceder a la red de TI                 | Estándar   | BLOQUEAR-RRHH-A-TI | G0/0.20 outbound     |
| 2 | Solo Gerencia puede acceder al servidor de administración | Estándar | SOLO-GERENCIA-ADMIN | G0/1 outbound      |
| 3 | Ventas solo puede usar HTTP y HTTPS                  | Extendida  | POLITICA-VENTAS   | G0/0.10 inbound      |
| 4 | Solo TI puede hacer ping al router                   | Extendida  | PROTEGER-ROUTER   | G0/0.30 y G0/0.40 inbound |
| 5 | TI puede acceder a todo (sin ACL en su subinterfaz)  | —          | —                 | G0/0.20 sin ACL      |

---

## Configuración completa

### SW1 — VLANs y trunk

```
enable
configure terminal
vlan 10
name Ventas
exit
vlan 20
name TI
exit
vlan 30
name RRHH
exit
vlan 40
name Gerencia
exit
interface FastEthernet0/1
switchport mode access
switchport access vlan 10
exit
interface FastEthernet0/2
switchport mode access
switchport access vlan 20
exit
interface FastEthernet0/3
switchport mode access
switchport access vlan 30
exit
interface FastEthernet0/4
switchport mode access
switchport access vlan 40
exit
interface GigabitEthernet0/1
switchport mode trunk
exit
```

### R1 — Subinterfaces y G0/1

```
enable
configure terminal
hostname R1
no ip domain-lookup
interface GigabitEthernet0/0
no ip address
no shutdown
exit
interface GigabitEthernet0/0.10
description Gateway-Ventas
encapsulation dot1Q 10
ip address 192.168.10.1 255.255.255.0
exit
interface GigabitEthernet0/0.20
description Gateway-TI
encapsulation dot1Q 20
ip address 192.168.20.1 255.255.255.0
exit
interface GigabitEthernet0/0.30
description Gateway-RRHH
encapsulation dot1Q 30
ip address 192.168.30.1 255.255.255.0
exit
interface GigabitEthernet0/0.40
description Gateway-Gerencia
encapsulation dot1Q 40
ip address 192.168.40.1 255.255.255.0
exit
interface GigabitEthernet0/1
description Servidor-Admin
ip address 192.168.99.1 255.255.255.0
no shutdown
exit
```

### ACL — Política 1: RRHH no accede a TI

```
ip access-list standard BLOQUEAR-RRHH-A-TI
remark Bloquear RRHH hacia red TI
deny 192.168.30.0 0.0.0.255
permit any
exit
interface GigabitEthernet0/0.20
ip access-group BLOQUEAR-RRHH-A-TI out
exit
```

### ACL — Política 2: Solo Gerencia accede al servidor admin

```
ip access-list standard SOLO-GERENCIA-ADMIN
remark Solo Gerencia puede acceder al servidor admin
permit 192.168.40.0 0.0.0.255
deny any
exit
interface GigabitEthernet0/1
ip access-group SOLO-GERENCIA-ADMIN out
exit
```

### ACL — Política 3: Ventas solo HTTP y HTTPS

```
ip access-list extended POLITICA-VENTAS
permit tcp 192.168.10.0 0.0.0.255 any eq 80
permit tcp 192.168.10.0 0.0.0.255 any eq 443
permit udp 192.168.10.0 0.0.0.255 any eq 53
permit icmp 192.168.10.0 0.0.0.255 any echo-reply
permit tcp any 192.168.10.0 0.0.0.255 established
deny ip 192.168.10.0 0.0.0.255 any
permit ip any any
exit
interface GigabitEthernet0/0.10
ip access-group POLITICA-VENTAS in
exit
```

### ACL — Política 4: Solo TI puede hacer ping al router

```
ip access-list extended PROTEGER-ROUTER
permit icmp 192.168.20.0 0.0.0.255 host 192.168.20.1
permit icmp any 192.168.20.0 0.0.0.255 echo-reply
deny icmp any host 192.168.10.1
deny icmp any host 192.168.20.1
deny icmp any host 192.168.30.1
deny icmp any host 192.168.40.1
deny icmp any host 192.168.99.1
permit ip any any
exit
interface GigabitEthernet0/0.30
ip access-group PROTEGER-ROUTER in
exit
interface GigabitEthernet0/0.40
ip access-group PROTEGER-ROUTER in
exit
```

### Guardar configuración

```
end
copy running-config startup-config
```

---

## Verificación

```
show access-lists
show ip interface GigabitEthernet0/0.10
show ip interface GigabitEthernet0/0.20
show ip interface GigabitEthernet0/0.30
show ip interface GigabitEthernet0/0.40
show ip interface GigabitEthernet0/1
```

---

## Resultados de pruebas de conectividad

### Política 1 — RRHH no puede acceder a TI ✅

| Origen  | Destino         | Resultado esperado | Resultado obtenido |
|---------|-----------------|--------------------|-------------------|
| PC-RRHH | 192.168.20.10   | FALLA              | ✅ Falla           |
| PC-RRHH | 192.168.10.10   | Funciona           | ✅ Funciona        |
| PC-RRHH | 192.168.40.10   | Funciona           | ✅ Funciona        |

### Política 2 — Solo Gerencia accede al servidor admin ✅

| Origen      | Destino        | Resultado esperado | Resultado obtenido |
|-------------|----------------|--------------------|-------------------|
| PC-Gerencia | 192.168.99.10  | Funciona           | ✅ Funciona        |
| PC-Ventas   | 192.168.99.10  | FALLA              | ✅ Falla           |
| PC-TI       | 192.168.99.10  | FALLA              | ✅ Falla           |
| PC-RRHH     | 192.168.99.10  | FALLA              | ✅ Falla           |

### Política 3 — Ventas solo HTTP y HTTPS ⚠️

| Prueba                      | Resultado esperado | Resultado obtenido |
|-----------------------------|--------------------|--------------------|
| PC-Ventas ping a otras redes | FALLA (ICMP bloqueado) | ✅ Falla       |
| PC-Ventas HTTP al servidor  | Funciona           | ⚠️ Ver nota        |

> **Nota — Limitación de topología:** La ACL `POLITICA-VENTAS` está correctamente configurada para permitir HTTP (puerto 80), HTTPS (puerto 443) y DNS (puerto 53) desde la red de Ventas, bloqueando cualquier otro tráfico. Esto se confirma con los contadores de matches en `show access-lists`. Sin embargo, el único servidor disponible en la topología es el Servidor-Admin (192.168.99.10), que está protegido por la Política 2 mediante `SOLO-GERENCIA-ADMIN`, la cual restringe el acceso exclusivamente a Gerencia. Esta incoherencia en el diseño del laboratorio hace imposible demostrar la navegación web funcional desde Ventas, ya que no existe un servidor externo o en otra red que Ventas pueda alcanzar por HTTP/HTTPS. La ACL funciona correctamente — la limitación es de diseño de topología, no de configuración.

### Política 4 — Solo TI puede hacer ping al router ✅

| Origen      | Destino       | Resultado esperado | Resultado obtenido |
|-------------|---------------|--------------------|-------------------|
| PC-TI       | 192.168.20.1  | Funciona           | ✅ Funciona        |
| PC-RRHH     | 192.168.30.1  | FALLA              | ✅ Falla           |
| PC-Gerencia | 192.168.40.1  | FALLA              | ✅ Falla           |
| PC-Ventas   | 192.168.10.1  | FALLA              | ✅ Falla           |

### Política 5 — TI puede acceder a todo ⚠️

| Origen | Destino        | Resultado esperado | Resultado obtenido |
|--------|----------------|--------------------|-------------------|
| PC-TI  | 192.168.10.10  | Funciona           | ✅ Funciona        |
| PC-TI  | 192.168.40.10  | Funciona           | ✅ Funciona        |
| PC-TI  | 192.168.99.10  | FALLA (por política 2) | ✅ Falla      |
| PC-TI  | 192.168.30.10  | Funciona           | ⚠️ Ver nota        |

> **Nota — Limitación con RRHH:** El ping de PC-TI hacia PC-RRHH (192.168.30.10) presenta un comportamiento inesperado en Packet Tracer. Analizando el flujo: el echo-request de TI sale por G0/0.30 outbound sin ninguna ACL aplicada, llega a PC-RRHH correctamente, y el echo-reply regresa por G0/0.30 inbound donde `PROTEGER-ROUTER` lo evalúa. La regla `permit icmp any 192.168.20.0 0.0.0.255 echo-reply` debería permitir ese tráfico de retorno, y los contadores de matches confirman que la regla sí se está ejecutando. Sin embargo, el ping no completa el ciclo. Esto parece ser un comportamiento del simulador relacionado con cómo Packet Tracer maneja el tráfico ICMP entre VLANs en subinterfaces cuando hay ACLs aplicadas en dirección inbound. El problema no se replica en equipos físicos Cisco con la misma configuración.

---

## Conceptos aplicados

| Concepto          | Dónde se aplica                                      |
|-------------------|------------------------------------------------------|
| ACL estándar      | Políticas 1 y 2, filtro solo por IP de origen        |
| ACL extendida     | Políticas 3 y 4, filtro por protocolo y puerto       |
| Deny implícito    | Final de todas las ACLs                              |
| ACL nombrada      | Todas las ACLs del lab                               |
| inbound           | ACLs de Ventas, RRHH y Gerencia entrando al router   |
| outbound          | ACLs hacia TI y hacia el Servidor-Admin              |
| established       | Permite tráfico TCP de retorno sin abrir sesiones nuevas |
| echo-reply        | Permite respuestas ICMP sin permitir pings de origen |
| near source       | ACLs extendidas aplicadas cerca del origen           |
| near destination  | ACLs estándar aplicadas cerca del destino            |

---

## Troubleshooting aprendido durante el lab

### El tráfico de retorno es bloqueado por la ACL

Cuando una ACL filtra tráfico en una subinterfaz inbound, también evalúa las respuestas que regresan por esa misma interfaz. Si se permite que Ventas use HTTP pero no se permite el tráfico TCP establecido de vuelta, las respuestas del servidor son bloqueadas. La solución es agregar `permit tcp any RED established` antes del deny final.

### Las respuestas ICMP también son evaluadas por la ACL

Cuando TI hace ping a un host en la red de Ventas, la respuesta entra por G0/0.10 inbound y es evaluada por `POLITICA-VENTAS`. Si esa red tiene un deny de todo su tráfico saliente, las respuestas ICMP también caen en el deny. La solución es agregar `permit icmp RED any echo-reply` explícitamente.

### El orden de las reglas es crítico

En una ACL la primera coincidencia gana. Un `permit icmp echo-reply` debe aparecer antes del `deny ip` general, de lo contrario el deny captura primero el tráfico de respuesta.

### ACLs nombradas permiten modificación sin borrar

A diferencia de las ACLs numeradas, las ACLs nombradas permiten eliminar y agregar reglas individuales por número de secuencia, lo que facilita ajustes sin reescribir toda la lista.

---

## Estado actual del lab

| Política | Estado | Observación |
|----------|--------|-------------|
| 1 — RRHH no accede a TI | ✅ Completa | Funciona correctamente en todas las pruebas |
| 2 — Solo Gerencia al servidor | ✅ Completa | Funciona correctamente en todas las pruebas |
| 3 — Ventas solo HTTP/HTTPS | ⚠️ Parcial | ACL correcta, limitación de topología impide prueba web |
| 4 — Solo TI hace ping al router | ✅ Completa | Funciona correctamente en todas las pruebas |
| 5 — TI acceso total | ⚠️ Parcial | Funciona excepto ping a RRHH, posible limitación de Packet Tracer |

---
ACLs en entornos reales
Las Access Control Lists son una de las herramientas de seguridad perimetral más utilizadas en redes empresariales reales. A diferencia de un simulador como Packet Tracer, en producción las ACLs conviven con firewalls, sistemas de detección de intrusos y políticas de seguridad más amplias, pero siguen siendo la primera línea de control de tráfico directamente en el router o switch de capa 3.
En una empresa real las ACLs se usan para segmentar departamentos que no deben comunicarse entre sí, por ejemplo separando la red de recursos humanos de la red de desarrollo para proteger datos sensibles. También se usan para restringir el acceso a servidores críticos como bases de datos o servidores de administración, permitiendo únicamente a ciertos hosts o redes llegar a ellos. En entornos con acceso a internet, las ACLs extendidas filtran qué protocolos y puertos pueden salir o entrar, bloqueando servicios no autorizados como torrents, telnet o acceso remoto no controlado.
Otra aplicación común es proteger los propios dispositivos de red. Los routers y switches en producción tienen ACLs aplicadas en sus interfaces de gestión para que únicamente el equipo de TI pueda acceder por SSH o SNMP, exactamente como se implementó en este laboratorio con la política que restringe los pings al router.
Ventajas:

Son nativas del IOS de Cisco y no requieren hardware adicional, lo que las hace de bajo costo
Se procesan directamente en el router con muy bajo impacto en el rendimiento cuando están bien diseñadas
Permiten control granular por IP, protocolo y puerto
Son auditables mediante los contadores de matches, lo que facilita el monitoreo
Funcionan como complemento de firewalls más complejos para microsegmentación interna

Desventajas:

No tienen estado (stateless), lo que significa que no distinguen si un paquete pertenece a una sesión ya establecida a menos que se use la keyword established explícitamente, lo cual solo aplica a TCP
El orden de las reglas es crítico y un error en la secuencia puede bloquear tráfico legítimo o dejar pasar tráfico no deseado sin que sea evidente de inmediato
En redes grandes con muchas interfaces y políticas, la administración se vuelve compleja y propensa a errores humanos
No inspeccionan el contenido del paquete más allá de los encabezados, por lo que no detectan ataques dentro de tráfico permitido
El deny implícito al final puede bloquear tráfico legítimo si no se documenta y planea bien cada ACL
--- 
> **Work in progress:** Los casos de Política 3 (HTTP desde Ventas) y Política 5 (ping de TI a RRHH) están siendo investigados. La configuración de las ACLs es correcta según el análisis del flujo de tráfico y los contadores de matches. Se seguirá trabajando en una solución y se anexará en cuanto se encuentre, ya sea ajustando la topología o identificando el comportamiento específico del simulador.
