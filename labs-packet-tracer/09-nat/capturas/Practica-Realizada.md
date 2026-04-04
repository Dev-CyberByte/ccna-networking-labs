# Lab 09 — NAT y PAT 

![NAT](https://img.shields.io/badge/Protocol-NAT-blue?style=flat-square)
![PAT](https://img.shields.io/badge/Protocol-PAT-orange?style=flat-square)
![CCNA](https://img.shields.io/badge/Cert-CCNA-red?style=flat-square)
![Packet Tracer](https://img.shields.io/badge/Tool-Packet%20Tracer-green?style=flat-square)
![Difficulty](https://img.shields.io/badge/Difficulty-Advanced-orange?style=flat-square)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen?style=flat-square)

---

## Descripción

Este laboratorio implementa los **tres tipos de NAT (Network Address Translation)** en un router Cisco usando Packet Tracer: NAT estático para exponer un servidor web interno a internet, NAT dinámico con pool de IPs públicas para un grupo de usuarios, y PAT (Port Address Translation) para que toda la red interna comparta una sola IP pública.

NAT es uno de los mecanismos más fundamentales en redes modernas — permite que miles de millones de dispositivos con IPs privadas accedan a internet usando un número limitado de IPs públicas, y es la razón principal por la que el agotamiento de IPv4 no ha colapsado internet.

---

## Objetivos del Lab

- Configurar NAT estático para publicar un servidor web interno con IP pública fija
- Implementar NAT dinámico con pool de IPs para un grupo específico de usuarios
- Configurar PAT (overload) para que toda la red interna comparta una IP pública
- Definir correctamente las interfaces `ip nat inside` y `ip nat outside`
- Usar ACLs para controlar qué tráfico se traduce con NAT
- Verificar traducciones activas con `show ip nat translations`
- Simular acceso desde internet al servidor interno via NAT estático

---

### Tabla de Direccionamiento

| Dispositivo | Interfaz | Dirección IP | Máscara | Descripción |
|-------------|----------|--------------|---------|-------------|
| R1 | G0/0 | 192.168.1.1 | 255.255.255.0 | LAN interna |
| R1 | G0/1 | 209.165.200.2 | 255.255.255.248 | WAN hacia ISP |
| ISP | G0/0 | 209.165.200.1 | 255.255.255.248 | Enlace hacia R1 |
| ISP | G0/1 | 209.165.201.1 | 255.255.255.0 | Red internet simulada |
| PC-Ventas1 | NIC | 192.168.1.10 | 255.255.255.0 | GW: 192.168.1.1 |
| PC-Ventas2 | NIC | 192.168.1.11 | 255.255.255.0 | GW: 192.168.1.1 |
| Servidor-Web | NIC | 192.168.1.100 | 255.255.255.0 | GW: 192.168.1.1 |
| PC-Externa | NIC | 209.165.201.10 | 255.255.255.0 | Simula internet |

### Asignación de IPs Públicas

| IP pública | Uso |
|------------|-----|
| 209.165.200.2 | IP WAN de R1 |
| 209.165.200.3 | NAT estático — Servidor-Web |
| 209.165.200.4 | Pool NAT dinámico inicio |
| 209.165.200.5 | Pool NAT dinámico fin |
| 209.165.200.6 | PAT (overload) |

---

## Implementación

### Parte 1 — Configuración de Interfaces y Ruta por Defecto

```bash
R1(config)# interface GigabitEthernet0/0
R1(config-if)# ip address 192.168.1.1 255.255.255.0
R1(config-if)# no shutdown

R1(config)# interface GigabitEthernet0/1
R1(config-if)# ip address 209.165.200.2 255.255.255.248
R1(config-if)# no shutdown

R1(config)# ip route 0.0.0.0 0.0.0.0 209.165.200.1
```

> El ISP no necesita ruta hacia `192.168.1.0/24`. Con NAT activo, desde internet solo se ven IPs públicas. Esto refleja exactamente el comportamiento real de internet.

### Parte 2 — Definir Interfaces NAT

Obligatorio antes de cualquier regla NAT:

```bash
R1(config)# interface GigabitEthernet0/0
R1(config-if)# ip nat inside

R1(config)# interface GigabitEthernet0/1
R1(config-if)# ip nat outside
```

> `inside` = red privada. `outside` = internet. Sin esto NAT no procesa ningún paquete aunque las reglas estén configuradas.

### Parte 3 — NAT Estático

Mapeo permanente y bidireccional de una IP privada a una IP pública fija:

```bash
R1(config)# ip nat inside source static 192.168.1.100 209.165.200.3
```

Desde este momento, cualquier tráfico que llegue a `209.165.200.3` desde internet es redirigido automáticamente a `192.168.1.100` (Servidor-Web interno).

### Parte 4 — NAT Dinámico

Pool de IPs públicas asignadas temporalmente cuando los hosts inician conexión:

```bash
R1(config)# ip nat pool POOL-PUBLICO 209.165.200.4 209.165.200.5 netmask 255.255.255.248
R1(config)# access-list 1 permit 192.168.1.10 0.0.0.1
R1(config)# ip nat inside source list 1 pool POOL-PUBLICO
```

> La ACL con wildcard `0.0.0.1` permite exactamente las IPs `.10` y `.11`. Si un tercer dispositivo intenta salir no obtiene traducción hasta que una IP del pool quede libre — limitación del NAT dinámico puro.

### Parte 5 — PAT (NAT Overload)

Toda la red interna comparte una sola IP pública diferenciada por número de puerto:

```bash
R1(config)# no ip nat inside source list 1 pool POOL-PUBLICO
R1(config)# no access-list 1
R1(config)# access-list 2 permit 192.168.1.0 0.0.0.255
R1(config)# ip nat inside source list 2 interface GigabitEthernet0/1 overload
```

> La palabra clave `overload` activa PAT. R1 mantiene una tabla de traducciones con la combinación IP:puerto para identificar cada sesión única, permitiendo que miles de dispositivos compartan una sola IP pública simultáneamente.

---

## ✅ Verificación

```bash
# Ver todas las traducciones activas
R1# show ip nat translations

# Ver estadísticas (hits, misses, interfaces)
R1# show ip nat statistics

# Limpiar traducciones dinámicas para pruebas (estáticas permanecen)
R1# clear ip nat translation *

# Verificar que las interfaces tengan inside/outside
R1# show running-config | include ip nat
```

### Resultado esperado con PAT activo

```
Pro  Inside global        Inside local       Outside local      Outside global
---  209.165.200.3        192.168.1.100      ---                ---
icmp 209.165.200.2:1024   192.168.1.10:1024  209.165.201.10:1024 209.165.201.10:1024
icmp 209.165.200.2:1025   192.168.1.11:1025  209.165.201.10:1024 209.165.201.10:1024
```

PC-Ventas1 y PC-Ventas2 comparten la misma Inside global `209.165.200.2` pero con puertos distintos (1024 y 1025). El NAT estático del servidor siempre aparece sin puertos.

---

## Cómo Funciona NAT — El Proceso de Traducción

Cuando PC-Ventas1 (`192.168.1.10`) hace ping a internet (`209.165.201.10`):

```
1. PC-Ventas1 envía paquete:
   Origen: 192.168.1.10  →  Destino: 209.165.201.10

2. R1 intercepta en G0/0 (ip nat inside):
   Consulta tabla NAT — PC-Ventas1 coincide con ACL
   Asigna IP pública del pool o usa PAT

3. R1 reescribe el paquete:
   Origen: 209.165.200.4  →  Destino: 209.165.201.10
   (o 209.165.200.2:1024 en PAT)

4. ISP recibe paquete con IP pública, lo enruta normalmente

5. Respuesta llega a R1 G0/1 (ip nat outside):
   R1 consulta tabla NAT y traduce de vuelta
   Destino: 209.165.200.4  →  192.168.1.10

6. PC-Ventas1 recibe la respuesta
```

El proceso es transparente para el usuario final — la PC nunca sabe que su IP fue cambiada.

---

## Aplicación en Entornos Reales

### ¿Dónde se usa NAT/PAT?

NAT y PAT están presentes en prácticamente toda red conectada a internet, desde hogares hasta corporaciones multinacionales.

**Tu router de casa**
El router de Telmex, Izzi o AT&T en tu hogar usa exactamente PAT. Tu PC, celular, tablet y smart TV tienen IPs privadas (192.168.x.x) y todas salen a internet usando la única IP pública que el ISP te asignó. El router mantiene la tabla de puertos para saber a qué dispositivo pertenece cada respuesta.

**Empresas medianas y grandes**
Una empresa con 500 empleados puede tener todo su tráfico saliendo por 2 o 3 IPs públicas usando PAT. El firewall perimetral (Cisco ASA, Palo Alto, Fortinet) hace NAT/PAT automáticamente. Los servidores internos que deben ser accesibles desde internet (web, correo, VPN) usan NAT estático con una IP pública dedicada.

**Data Centers y hosting**
Los proveedores de hosting usan NAT estático masivamente — cada servidor virtual o sitio web tiene una IP pública asignada que mapea a una IP privada interna. Esto permite tener miles de servidores en un mismo data center con IPs privadas y exponer solo los necesarios.

**Proveedores de Internet (ISPs)**
Los ISPs de nivel residencial usan NAT de nivel de operador (CGN — Carrier Grade NAT) para dar IPs privadas a sus clientes y hacer NAT en sus propios routers. Esto se volvió necesario cuando las IPs IPv4 públicas se agotaron en 2011.

**Seguridad perimetral**
NAT actúa como primera línea de defensa — los dispositivos internos con IPs privadas no son directamente alcanzables desde internet. Un atacante no puede iniciar una conexión a `192.168.1.50` porque esa IP no existe en internet. Solo los servicios con NAT estático están expuestos, lo que reduce drásticamente la superficie de ataque.

---

## Ventajas y Desventajas

### Ventajas

| Ventaja | Descripción |
|---------|-------------|
| **Conserva IPs públicas** | Miles de dispositivos comparten una sola IP pública con PAT, extendiendo la vida útil de IPv4 |
| **Seguridad implícita** | Los hosts internos no son alcanzables directamente desde internet sin una regla NAT explícita |
| **Flexibilidad interna** | La red interna puede reorganizarse sin afectar las IPs públicas visibles desde internet |
| **Control granular** | ACLs permiten decidir exactamente qué tráfico se traduce y qué tráfico no |
| **Transparencia** | Los usuarios internos no notan nada — el proceso es completamente transparente |
| **Exposición controlada** | NAT estático permite exponer solo servicios específicos, no toda la red |

### ❌ Desventajas

| Desventaja | Descripción |
|-----------|-------------|
| **Rompe el modelo extremo a extremo** | Internet fue diseñado para conectividad directa. NAT añade una capa de indirección que complica algunos protocolos |
| **Problemas con ciertos protocolos** | VoIP, FTP activo, IPSec y algunos juegos online tienen problemas con NAT porque incluyen IPs en el payload |
| **Dificulta troubleshooting** | Los logs muestran la IP pública, no la privada. Necesitas correlacionar con la tabla NAT para saber qué host interno originó el tráfico |
| **Latencia adicional** | Cada paquete debe ser inspeccionado y reescrito, añadiendo microsegundos de procesamiento |
| **NAT no es firewall real** | Aunque filtra conexiones entrantes, NAT no inspecciona el contenido del tráfico. No reemplaza un firewall real |
| **Complejidad en VPNs** | Dos redes con el mismo espacio privado (192.168.1.x) conectadas por VPN generan conflictos que requieren NAT adicional |

---

## Escalabilidad

### Tipos de NAT según el tamaño de la red

```
Red doméstica:
Un router, PAT con 1 IP pública
Soporta ~65,000 sesiones simultáneas teóricas

Empresa pequeña (10-50 usuarios):
Firewall con PAT + NAT estático para servidores
Pool de 2-5 IPs públicas

Empresa mediana (50-500 usuarios):
Firewall dedicado (ASA, Palo Alto)
PAT para usuarios + NAT estático para DMZ
Pool de 10-50 IPs públicas
Logs de traducción para auditoría

Empresa grande / ISP:
Carrier Grade NAT (CGN)
Millones de sesiones simultáneas
Hardware especializado con ASICs
Logging obligatorio por regulación
```

### NAT y la transición a IPv6

NAT nació como solución temporal al agotamiento de IPv4 pero se volvió permanente. Con IPv6 cada dispositivo tiene una IP pública única y NAT en principio no es necesario — hay suficientes IPs para todos los dispositivos del universo. Sin embargo, muchas empresas siguen usando NAT66 (NAT en IPv6) por razones de seguridad y política interna, aunque el protocolo no lo requiere técnicamente.

En redes modernas dual-stack (IPv4 + IPv6) conviven ambos mundos: IPv6 nativo para tráfico moderno y NAT/PAT para compatibilidad con sistemas legacy IPv4.

---

## Conceptos Clave Aplicados

| Concepto | Descripción |
|---------|-------------|
| **ip nat inside** | Marca la interfaz LAN como lado privado. NAT procesa tráfico saliente de aquí |
| **ip nat outside** | Marca la interfaz WAN como lado público. NAT procesa tráfico entrante de aquí |
| **NAT estático** | Mapeo 1:1 permanente y bidireccional. Necesario para servidores accesibles desde internet |
| **NAT dinámico** | Pool de IPs públicas asignadas temporalmente. Limitado al tamaño del pool |
| **PAT / overload** | Muchas IPs privadas → 1 IP pública usando puertos únicos por sesión |
| **Inside Local** | IP privada real del dispositivo interno (192.168.1.10) |
| **Inside Global** | IP pública que representa al dispositivo ante internet (209.165.200.4) |
| **Outside Local** | IP del destino vista desde dentro (generalmente igual a Outside Global) |
| **Outside Global** | IP pública real del destino en internet |
| **ACL en NAT** | Controla qué hosts o redes son elegibles para traducción NAT |
| **clear ip nat translation** | Limpia entradas dinámicas. Las estáticas son permanentes |

---

## 📁 Archivos del Lab

| Archivo | Descripción |
|--------|-------------|
| `lab-09-nat.pkt` | Archivo de Packet Tracer con la topología completa configurada |
| `topologia.png` | Vista general de la topología con R1, ISP y dispositivos |
| `r1-interfaces.png` | show ip interface brief con G0/0 y G0/1 up/up |
| `nat-estatico.png` | show ip nat translations con traducción estática activa |
| `nat-dinamico.png` | show ip nat translations con traducciones dinámicas del pool |
| `pat-translations.png` | show ip nat translations con PAT — misma IP pública, puertos distintos |
| `nat-statistics.png` | show ip nat statistics con hits y misses |
| `ping-lan-internet.png` | Ping exitoso desde PC-Ventas1 a 209.165.201.10 |
| `ping-externo-servidor.png` | Ping exitoso desde PC-Externa a 209.165.200.3 |

---

## 🔗 Referencias

- [RFC 3022 — Traditional NAT](https://datatracker.ietf.org/doc/html/rfc3022)
- [RFC 2663 — NAT Terminology](https://datatracker.ietf.org/doc/html/rfc2663)
- [Cisco — NAT Configuration Guide](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/ipaddr_nat/configuration/xe-16/nat-xe-16-book.html)
- CCNA 200-301 Official Cert Guide — Wendell Odom
- Dominio CCNA 4.3 — IP Services

---
