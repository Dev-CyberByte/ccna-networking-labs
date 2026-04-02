# TCP/IP

TCP/IP es el conjunto de protocolos que realmente usamos en internet y en
redes modernas. A diferencia del modelo OSI que tiene 7 capas, TCP/IP
agrupa todo en 4 capas. Es el estándar real de comunicación en redes.

---

## Las 4 capas de TCP/IP

### Capa 4 — Aplicación
Equivale a las capas 5, 6 y 7 del modelo OSI (Sesión, Presentación
y Aplicación). Aquí viven todos los protocolos que el usuario
y las aplicaciones usan directamente.

**Protocolos:**

| Protocolo | Puerto | Uso |
|-----------|--------|-----|
| HTTP | 80 | Navegación web sin cifrado |
| HTTPS | 443 | Navegación web cifrada |
| FTP | 20/21 | Transferencia de archivos |
| SSH | 22 | Acceso remoto seguro |
| Telnet | 23 | Acceso remoto sin cifrado |
| SMTP | 25 | Envío de correo |
| DNS | 53 | Resolución de nombres |
| DHCP | 67/68 | Asignación automática de IPs |
| SNMP | 161 | Monitoreo de red |
| NTP | 123 | Sincronización de tiempo |

---

### Capa 3 — Transporte
Equivale a la capa 4 del modelo OSI. Se encarga de la comunicación
extremo a extremo entre aplicaciones. Usa puertos para identificar
a qué aplicación van los datos.

**Dos protocolos principales:**

#### TCP — Transmission Control Protocol
- Orientado a conexión (usa el handshake de 3 vías)
- Garantiza entrega, orden y sin duplicados
- Más lento pero confiable
- Usado por: HTTP, HTTPS, FTP, SSH, SMTP

#### El handshake de 3 vías (Three-Way Handshake)
Antes de enviar datos, TCP establece una conexión:
```
Cliente                        Servidor
   |                               |
   |-------- SYN ----------------->|   "Quiero conectarme"
   |                               |
   |<------- SYN-ACK --------------|   "Aceptado, listo"
   |                               |
   |-------- ACK ----------------->|   "Confirmado, enviando datos"
   |                               |
   |======== DATOS ===============>|
```

#### UDP — User Datagram Protocol
- Sin conexión (envía y ya)
- No garantiza entrega ni orden
- Más rápido, menos overhead
- Usado por: DNS, DHCP, VoIP, streaming, videojuegos

#### Comparativa TCP vs UDP

| Característica | TCP | UDP |
|----------------|-----|-----|
| Conexión | Sí (3-way handshake) | No |
| Confiabilidad | Garantizada | No garantizada |
| Orden de datos | Garantizado | No garantizado |
| Control de flujo | Sí | No |
| Velocidad | Más lento | Más rápido |
| Uso | HTTP, SSH, FTP | DNS, VoIP, streaming |

#### Puertos

Los puertos identifican a qué aplicación van los datos dentro de un host.

| Rango | Tipo | Descripción |
|-------|------|-------------|
| 0 - 1023 | Well-known | Reservados para servicios estándar |
| 1024 - 49151 | Registered | Asignados por IANA a aplicaciones |
| 49152 - 65535 | Dynamic/Private | Puertos efímeros del cliente |

---

### Capa 2 — Internet
Equivale a la capa 3 del modelo OSI (Red). Se encarga del
direccionamiento lógico y el enrutamiento entre redes.

**Protocolos principales:**

| Protocolo | Función |
|-----------|---------|
| IPv4 | Direccionamiento lógico de 32 bits |
| IPv6 | Direccionamiento lógico de 128 bits |
| ICMP | Mensajes de error y diagnóstico (ping) |
| ARP | Resuelve IP → MAC en la red local |

#### ¿Qué hace ARP?
Cuando un host quiere enviar datos a una IP de su misma red,
necesita saber la dirección MAC del destino. ARP lo resuelve:
```
Host A                         Host B
192.168.1.10                   192.168.1.20
   |                               |
   |--- ARP Request (broadcast) -->|  "¿Quién tiene 192.168.1.20?"
   |                               |
   |<-- ARP Reply (unicast) -------|  "Yo, y mi MAC es AA:BB:CC:DD:EE:FF"
   |                               |
   |======= Datos (unicast) ======>|
```

El resultado se guarda en la **tabla ARP** del host:
```
show arp                   ← en router Cisco
arp -a                     ← en Windows/Linux
```

---

### Capa 1 — Acceso a la Red
Equivale a las capas 1 y 2 del modelo OSI (Física y Enlace de Datos).
Maneja cómo los bits viajan por el medio físico y cómo se forman
los frames dentro de una red local.

**Protocolos y estándares:**

| Tecnología | Descripción |
|------------|-------------|
| Ethernet (802.3) | Red cableada LAN |
| Wi-Fi (802.11) | Red inalámbrica |
| PPP | Conexiones punto a punto |

---

## Comparativa OSI vs TCP/IP
```
     OSI (7 capas)              TCP/IP (4 capas)
┌─────────────────────┐      ┌──────────────────────┐
│  7 - Aplicación     │      │                      │
├─────────────────────┤      │  4 - Aplicación      │
│  6 - Presentación   │  ──► │                      │
├─────────────────────┤      │                      │
│  5 - Sesión         │      └──────────────────────┘
├─────────────────────┤      ┌──────────────────────┐
│  4 - Transporte     │  ──► │  3 - Transporte      │
├─────────────────────┤      └──────────────────────┘
│  3 - Red            │      ┌──────────────────────┐
├─────────────────────┤  ──► │  2 - Internet        │
└─────────────────────┘      └──────────────────────┘
│  2 - Enlace         │      ┌──────────────────────┐
├─────────────────────┤  ──► │  1 - Acceso a Red    │
│  1 - Física         │      │                      │
└─────────────────────┘      └──────────────────────┘
```

---

## Cómo viaja un paquete de extremo a extremo

Ejemplo: Tu navegador pide una página web (HTTP).
```
TU PC                        ROUTER                       SERVIDOR WEB
─────                        ──────                       ────────────
1. Aplicación genera datos HTTP
2. Transporte agrega header TCP (puerto 80)
3. Internet agrega header IP (IP destino)
4. Acceso a Red agrega header Ethernet (MAC destino)
5. Sale como bits por el cable
                             6. Router recibe el frame
                             7. Quita header Ethernet
                             8. Lee IP destino → enruta
                             9. Nuevo header Ethernet
                             10. Reenvía al servidor
                                                          11. Servidor recibe
                                                          12. Quita headers
                                                          13. Aplicación lee HTTP
                                                          14. Responde con HTML
```

---

## Puertos importantes para el examen CCNA
```
FTP Data    →  20       SSH      →  22
FTP Control →  21       Telnet   →  23
SMTP        →  25       DNS      →  53
DHCP Server →  67       HTTP     →  80
DHCP Client →  68       HTTPS    →  443
TFTP        →  69       SNMP     →  161
NTP         →  123      Syslog   →  514
```

---

## En el examen CCNA

Preguntas típicas sobre TCP/IP:

- *¿Qué protocolo resuelve una IP a una MAC?* → ARP
- *¿Qué protocolo usa ping?* → ICMP
- *¿En qué capa de TCP/IP opera IP?* → Capa Internet
- *¿TCP o UDP usa el handshake de 3 vías?* → TCP
- *¿Qué protocolo usaría VoIP y por qué?* → UDP, porque prioriza velocidad
- *¿En qué rango están los puertos well-known?* → 0 a 1023
- *¿Qué puerto usa HTTPS?* → 443
