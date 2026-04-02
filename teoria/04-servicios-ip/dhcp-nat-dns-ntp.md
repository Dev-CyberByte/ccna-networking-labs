# Servicios IP — DHCP, NAT, DNS y NTP

Los servicios IP son protocolos que hacen que la red funcione de forma
automática y eficiente. Sin ellos, cada dispositivo necesitaría configuración
manual para conectarse, resolver nombres y sincronizar su reloj.

---

## DHCP — Dynamic Host Configuration Protocol

### ¿Qué es DHCP?
Es el protocolo que asigna automáticamente configuración de red a los
dispositivos. Sin DHCP, cada PC necesitaría IP, máscara, gateway y DNS
configurados a mano.

### Qué entrega un servidor DHCP

| Parámetro | Ejemplo |
|-----------|---------|
| Dirección IP | 192.168.1.100 |
| Máscara de subred | 255.255.255.0 |
| Gateway por defecto | 192.168.1.1 |
| Servidor DNS | 8.8.8.8 |
| Tiempo de concesión | 24 horas |

### Proceso DORA
El proceso de asignación tiene 4 pasos. Se llama **DORA**:
```
Cliente                         Servidor DHCP
   │                                 │
   │── Discover (broadcast) ────────►│  "¿Hay algún servidor DHCP?"
   │                                 │
   │◄─ Offer (unicast/broadcast) ────│  "Sí, te ofrezco 192.168.1.100"
   │                                 │
   │── Request (broadcast) ─────────►│  "Acepto esa IP"
   │                                 │
   │◄─ Acknowledge (broadcast) ──────│  "Confirmado, es tuya por 24h"
   │                                 │
```

> Todos los mensajes DHCP usan UDP. El cliente usa puerto 68,
> el servidor usa puerto 67.

### Configuración de servidor DHCP en Cisco IOS
```
Router(config)# ip dhcp excluded-address 192.168.1.1 192.168.1.10
Router(config)# ip dhcp excluded-address 192.168.1.254

Router(config)# ip dhcp pool LAN-VENTAS
Router(dhcp-config)# network 192.168.1.0 255.255.255.0
Router(dhcp-config)# default-router 192.168.1.1
Router(dhcp-config)# dns-server 8.8.8.8 8.8.4.4
Router(dhcp-config)# lease 1
Router(dhcp-config)# exit
```

> `excluded-address` reserva IPs que no se asignarán automáticamente.
> Siempre excluye el gateway y las IPs de servidores antes del pool.

### DHCP Relay Agent
Cuando el servidor DHCP está en una red diferente al cliente, el router
actúa como relay y reenvía los mensajes DHCP hacia el servidor.
```
PC ──── Switch ──── Router ──── Servidor DHCP
(192.168.1.0/24)         (10.0.0.0/30)  (10.0.0.2)
```

Configuración en la interfaz del router hacia los clientes:
```
Router(config)# interface GigabitEthernet0/0
Router(config-if)# ip helper-address 10.0.0.2
```

### Verificación DHCP
```
Router# show ip dhcp pool
Router# show ip dhcp binding
Router# show ip dhcp conflict
```

### En el cliente
```
ipconfig /release        ← liberar IP (Windows)
ipconfig /renew          ← solicitar nueva IP (Windows)
```

---

## NAT — Network Address Translation

### ¿Qué es NAT?
Es el mecanismo que permite que dispositivos con IPs privadas se comuniquen
con internet usando una IP pública. El router traduce las IPs privadas
a públicas y viceversa.

### ¿Por qué existe NAT?
Las IPs IPv4 públicas son escasas. NAT permite que cientos de dispositivos
internos compartan una sola IP pública.

### Terminología NAT

| Término | Descripción |
|---------|-------------|
| Inside Local | IP privada del dispositivo interno |
| Inside Global | IP pública que representa al dispositivo interno |
| Outside Local | IP del servidor externo vista desde adentro |
| Outside Global | IP real del servidor externo |
```
PC interno          Router NAT              Servidor web
192.168.1.10  ───► 203.0.113.1 ──────────► 8.8.8.8
(Inside Local)     (Inside Global)          (Outside Global)
```

### Tipos de NAT

#### NAT Estático
Mapea una IP privada a una IP pública de forma permanente y fija.
Se usa para servidores internos que deben ser accesibles desde internet.
```
Router(config)# ip nat inside source static 192.168.1.10 203.0.113.10

Router(config)# interface GigabitEthernet0/0
Router(config-if)# ip nat inside
Router(config-if)# exit

Router(config)# interface GigabitEthernet0/1
Router(config-if)# ip nat outside
Router(config-if)# exit
```

#### NAT Dinámico
Mapea IPs privadas a un pool de IPs públicas. La asignación es temporal.
```
Router(config)# ip nat pool PUBLICAS 203.0.113.1 203.0.113.10 netmask 255.255.255.0
Router(config)# access-list 1 permit 192.168.1.0 0.0.0.255
Router(config)# ip nat inside source list 1 pool PUBLICAS

Router(config)# interface GigabitEthernet0/0
Router(config-if)# ip nat inside
Router(config-if)# exit

Router(config)# interface GigabitEthernet0/1
Router(config-if)# ip nat outside
Router(config-if)# exit
```

#### PAT — Port Address Translation (NAT Overload)
Es el tipo más común. Múltiples IPs privadas comparten una sola IP pública
usando diferentes puertos para distinguir las sesiones.
```
192.168.1.10:1024  ─┐
192.168.1.11:1025  ─┤──► 203.0.113.1:puerto_unico ──► Internet
192.168.1.12:1026  ─┘
```
```
Router(config)# access-list 1 permit 192.168.1.0 0.0.0.255
Router(config)# ip nat inside source list 1 interface GigabitEthernet0/1 overload

Router(config)# interface GigabitEthernet0/0
Router(config-if)# ip nat inside
Router(config-if)# exit

Router(config)# interface GigabitEthernet0/1
Router(config-if)# ip nat outside
Router(config-if)# exit
```

> La palabra clave `overload` activa PAT. Es la configuración más usada
> en redes reales y en el examen CCNA.

### Verificación NAT
```
Router# show ip nat translations
Router# show ip nat statistics
Router# debug ip nat
```

### Salida de show ip nat translations
```
Pro  Inside global      Inside local       Outside local      Outside global
tcp  203.0.113.1:1024   192.168.1.10:1024  8.8.8.8:80         8.8.8.8:80
tcp  203.0.113.1:1025   192.168.1.11:1025  8.8.8.8:80         8.8.8.8:80
```

---

## DNS — Domain Name System

### ¿Qué es DNS?
Es el protocolo que traduce nombres de dominio a direcciones IP.
Sin DNS tendrías que memorizar IPs para navegar.
```
Navegador                Servidor DNS              Servidor web
   │                          │                         │
   │── "¿Cuál es la IP de" ──►│                         │
   │   "google.com?"          │                         │
   │                          │                         │
   │◄─ "Es 142.250.78.46" ────│                         │
   │                          │                         │
   │──────────────────────────────────────────────────►│
   │         conexión directa a 142.250.78.46           │
```

### Jerarquía DNS
```
.  (Root)
├── .com
│   ├── google.com
│   │   └── www.google.com
│   └── cisco.com
├── .org
└── .mx
    └── unam.mx
```

### Tipos de registros DNS

| Registro | Función | Ejemplo |
|----------|---------|---------|
| A | Nombre → IPv4 | www.ejemplo.com → 192.168.1.10 |
| AAAA | Nombre → IPv6 | www.ejemplo.com → 2001:db8::1 |
| CNAME | Alias → nombre | mail.ejemplo.com → servidor.ejemplo.com |
| MX | Servidor de correo | ejemplo.com → mail.ejemplo.com |
| PTR | IPv4 → Nombre (reverso) | 192.168.1.10 → www.ejemplo.com |
| NS | Servidor de nombres | ejemplo.com → ns1.ejemplo.com |

### Configuración DNS en Cisco IOS

Configurar el router para que use un servidor DNS:
```
Router(config)# ip domain-lookup
Router(config)# ip name-server 8.8.8.8
Router(config)# ip name-server 8.8.4.4
Router(config)# ip domain-name empresa.local
```

Desactivar DNS lookup (evita esperas cuando escribes mal un comando):
```
Router(config)# no ip domain-lookup
```

### Verificación DNS
```
Router# show hosts
Router# ping www.google.com
nslookup google.com          ← en Windows/Linux
```

---

## NTP — Network Time Protocol

### ¿Qué es NTP?
Es el protocolo que sincroniza el reloj de los dispositivos de red.
Tener la hora correcta es crítico para:

- Correlacionar logs de syslog entre dispositivos
- Certificados digitales y autenticación
- Registros de auditoría y seguridad
- Protocolos como Kerberos

> Un dispositivo con hora incorrecta puede fallar en autenticación
> o tener logs imposibles de analizar.

### Cómo funciona NTP

NTP usa una jerarquía llamada **stratum**:
```
Stratum 0: Reloj atómico / GPS (fuente de tiempo real)
     │
Stratum 1: Servidor NTP primario (conectado directamente al stratum 0)
     │
Stratum 2: Servidor NTP secundario (sincronizado con stratum 1)
     │
Stratum 3: Dispositivos de red (routers, switches)
```

Cuanto menor el stratum, más preciso y confiable es el tiempo.
NTP usa **UDP puerto 123**.

### Configuración NTP en Cisco IOS

Configurar el router como cliente NTP:
```
Router(config)# ntp server 216.239.35.0
Router(config)# ntp server 216.239.35.4
```

Configurar zona horaria:
```
Router(config)# clock timezone CST -6
Router(config)# clock summer-time CDT recurring
```

Configurar el router como servidor NTP para la red interna:
```
Router(config)# ntp master 3
```

### Verificación NTP
```
Router# show ntp status
Router# show ntp associations
Router# show clock detail
```

### Salida de show ntp status
```
Clock is synchronized, stratum 3, reference is 216.239.35.0
nominal freq is 250.0000 Hz, actual freq is 250.0000 Hz
reference time is E5A12B4C.3D70A3D7
```

La línea clave es **Clock is synchronized**. Si dice
**Clock is unsynchronized** hay un problema de conectividad con el servidor NTP.

---

## Syslog — Registro de eventos

### ¿Qué es Syslog?
Es el protocolo estándar para que los dispositivos envíen mensajes
de registro (logs) a un servidor centralizado. Usa **UDP puerto 514**.

### Niveles de severidad Syslog

| Nivel | Nombre | Descripción |
|-------|--------|-------------|
| 0 | Emergency | Sistema inutilizable |
| 1 | Alert | Acción inmediata requerida |
| 2 | Critical | Condición crítica |
| 3 | Error | Condición de error |
| 4 | Warning | Condición de advertencia |
| 5 | Notice | Condición normal pero significativa |
| 6 | Informational | Mensajes informativos |
| 7 | Debugging | Mensajes de depuración |

> Truco para recordarlos: **Every Awesome Cisco Engineer Will Need Daily Debugging**

### Configuración Syslog en Cisco IOS
```
Router(config)# logging host 192.168.1.200
Router(config)# logging trap informational
Router(config)# logging source-interface GigabitEthernet0/0
Router(config)# service timestamps log datetime msec
```

### Verificación Syslog
```
Router# show logging
```

---

## En el examen CCNA

Preguntas típicas de este módulo:

- *¿Qué significa DORA en DHCP?* → Discover, Offer, Request, Acknowledge
- *¿Qué puertos usa DHCP?* → Cliente 68, Servidor 67
- *¿Qué comando configura DHCP relay?* → ip helper-address
- *¿Qué diferencia hay entre NAT y PAT?* → PAT usa puertos para múltiples IPs privadas en una sola IP pública
- *¿Qué palabra clave activa PAT?* → overload
- *¿Qué es Inside Local?* → La IP privada del dispositivo interno
- *¿Qué puerto usa DNS?* → UDP 53
- *¿Qué tipo de registro DNS traduce nombre a IPv4?* → Registro A
- *¿Qué puerto usa NTP?* → UDP 123
- *¿Qué es el stratum en NTP?* → El nivel de jerarquía de la fuente de tiempo
- *¿Qué nivel de syslog es el más crítico?* → 0 (Emergency)
- *¿Qué puerto usa Syslog?* → UDP 514
