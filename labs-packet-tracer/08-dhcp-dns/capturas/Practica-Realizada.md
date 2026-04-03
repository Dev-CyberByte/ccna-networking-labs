# Lab 08 — DHCP y DNS 

![DHCP](https://img.shields.io/badge/Protocol-DHCP-blue?style=flat-square)
![DNS](https://img.shields.io/badge/Protocol-DNS-green?style=flat-square)
![CCNA](https://img.shields.io/badge/Cert-CCNA-red?style=flat-square)
![Packet Tracer](https://img.shields.io/badge/Tool-Packet%20Tracer-green?style=flat-square)
![Difficulty](https://img.shields.io/badge/Difficulty-Intermediate-yellow?style=flat-square)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen?style=flat-square)

---

## Descripción

Este laboratorio implementa **DHCP (Dynamic Host Configuration Protocol)** y **DNS (Domain Name System)** en una topología empresarial con múltiples VLANs usando Cisco Packet Tracer. El router R1 actúa como servidor DHCP centralizado con un pool por VLAN, mientras que un servidor dedicado provee resolución de nombres para el dominio interno `empresa.local`.

A diferencia de asignar IPs manualmente en cada PC, DHCP automatiza completamente la configuración de red, reduciendo errores humanos y el tiempo administrativo. DNS complementa esto permitiendo que los usuarios accedan a recursos por nombre en lugar de memorizar direcciones IP.

---

## Objetivos del Lab

- Configurar R1 como servidor DHCP con tres pools diferenciados por VLAN
- Implementar exclusiones de direcciones para reservar IPs estáticas
- Configurar subinterfaces en R1 para enrutamiento inter-VLAN (Router-on-a-Stick)
- Configurar trunks 802.1Q entre switches y R1
- Desplegar un servidor DNS con registros tipo A para el dominio interno
- Verificar el proceso DORA completo (Discover, Offer, Request, Acknowledge)
- Validar resolución de nombres desde las PCs clientes

---

### Tabla de Direccionamiento

| Dispositivo | Interfaz | Dirección IP | Máscara | Descripción |
|-------------|----------|--------------|---------|-------------|
| R1 | G0/0.10 | 192.168.10.1 | 255.255.255.0 | Gateway VLAN 10 — Ventas |
| R1 | G0/0.20 | 192.168.20.1 | 255.255.255.0 | Gateway VLAN 20 — TI |
| R1 | G0/0.99 | 192.168.99.1 | 255.255.255.0 | Gateway VLAN 99 — Admin |
| R1 | G0/1.30 | 192.168.30.1 | 255.255.255.0 | Gateway VLAN 30 — RRHH |
| SW1 | VLAN 99 | 192.168.99.2 | 255.255.255.0 | Administración |
| SW2 | VLAN 99 | 192.168.99.3 | 255.255.255.0 | Administración |
| Servidor DNS | NIC | 192.168.99.100 | 255.255.255.0 | DNS estático |

### Pools DHCP

| Pool | VLAN | Red | Rango excluido | Gateway | DNS |
|------|------|-----|----------------|---------|-----|
| POOL-VENTAS | 10 | 192.168.10.0/24 | .1 — .10 | 192.168.10.1 | 192.168.99.100 |
| POOL-TI | 20 | 192.168.20.0/24 | .1 — .10 | 192.168.20.1 | 192.168.99.100 |
| POOL-RRHH | 30 | 192.168.30.0/24 | .1 — .10 | 192.168.30.1 | 192.168.99.100 |

---

## Implementación

### Parte 1 — Configuración de Switches

Cada switch requiere VLANs, puertos de acceso y un enlace trunk hacia R1. Ejemplo en SW1:

```bash
SW1(config)# vlan 10
SW1(config-vlan)# name Ventas
SW1(config)# interface FastEthernet0/1
SW1(config-if)# switchport mode access
SW1(config-if)# switchport access vlan 10
SW1(config)# interface GigabitEthernet0/1
SW1(config-if)# switchport mode trunk
SW1(config-if)# switchport trunk native vlan 99
SW1(config-if)# switchport trunk allowed vlan 10,20,99
```

> La VLAN nativa en el trunk debe coincidir en ambos extremos. Se usa VLAN 99 como nativa para separar el tráfico de administración del tráfico de datos.

### Parte 2 — Router-on-a-Stick en R1

R1 usa una sola interfaz física con múltiples subinterfaces, una por VLAN. Cada subinterfaz tiene su encapsulación dot1Q y actúa como gateway de su VLAN:

```bash
R1(config)# interface GigabitEthernet0/0.10
R1(config-subif)# encapsulation dot1Q 10
R1(config-subif)# ip address 192.168.10.1 255.255.255.0

R1(config)# interface GigabitEthernet0/0.99
R1(config-subif)# encapsulation dot1Q 99 native
R1(config-subif)# ip address 192.168.99.1 255.255.255.0
```

> La subinterfaz de la VLAN nativa lleva el keyword `native` al final del comando encapsulation.

### Parte 3 — Servidor DHCP en R1

Primero se excluyen las IPs reservadas, luego se crean los pools:

```bash
R1(config)# ip dhcp excluded-address 192.168.10.1 192.168.10.10
R1(config)# ip dhcp excluded-address 192.168.20.1 192.168.20.10
R1(config)# ip dhcp excluded-address 192.168.30.1 192.168.30.10

R1(config)# ip dhcp pool POOL-VENTAS
R1(dhcp-config)# network 192.168.10.0 255.255.255.0
R1(dhcp-config)# default-router 192.168.10.1
R1(dhcp-config)# dns-server 192.168.99.100
R1(dhcp-config)# lease 1
```

> Siempre configura las exclusiones **antes** de crear los pools. Si el pool existe primero, DHCP podría asignar una IP antes de que la exclusión entre en efecto.

### Parte 4 — Servidor DNS

El servidor DNS corre en un Server genérico con IP estática `192.168.99.100` conectado a SW1 en VLAN 99. Registros configurados:

| Nombre | Tipo | Dirección |
|--------|------|-----------|
| `servidor.empresa.local` | A Record | 192.168.99.100 |
| `r1.empresa.local` | A Record | 192.168.99.1 |
| `intranet.empresa.local` | A Record | 192.168.99.100 |

---

## ✅ Verificación

```bash
# Ver pools DHCP y uso
R1# show ip dhcp pool

# Ver concesiones activas con MAC address
R1# show ip dhcp binding

# Ver estadísticas del proceso DORA
R1# show ip dhcp server statistics

# Verificar conflictos (debe estar vacío)
R1# show ip dhcp conflict
```

### Resultado esperado en `show ip dhcp binding`

```
IP address      Client-ID/         Lease expiration    Type
                Hardware address
192.168.10.11   0060.2F12.3456     Mar 01 2025         Automatic
192.168.10.12   0060.4B78.9ABC     Mar 01 2025         Automatic
192.168.20.11   0060.7C34.DEF0     Mar 01 2025         Automatic
192.168.20.12   0060.8D45.EF01     Mar 01 2025         Automatic
192.168.30.11   0060.9E56.F012     Mar 01 2025         Automatic
```

### Prueba de resolución DNS

```bash
# Desde cualquier PC
ping intranet.empresa.local
ping r1.empresa.local

# Web browser
http://intranet.empresa.local
```

---

## El Proceso DORA

DORA es el handshake de cuatro pasos que ocurre cada vez que una PC solicita IP por DHCP:

```
PC                                    R1 (DHCP Server)
|                                           |
|--- DISCOVER (broadcast) ----------------->|  "¿Hay algún servidor DHCP?"
|                                           |
|<-- OFFER (unicast) -----------------------|  "Yo soy servidor, te ofrezco 192.168.10.11"
|                                           |
|--- REQUEST (broadcast) ------------------>|  "Acepto la IP ofrecida"
|                                           |
|<-- ACK (unicast) -------------------------+  "Confirmado, es tuya por 1 día"
```

> Los mensajes Discover y Request son broadcasts porque la PC aún no tiene IP. El relay agent (`ip helper-address`) es necesario cuando el servidor DHCP está en una red diferente a la del cliente.

---

## Aplicación en Entornos Reales

### ¿Dónde se usa DHCP + DNS?

Estos dos protocolos son absolutamente fundamentales en cualquier red empresarial. Es prácticamente imposible encontrar una empresa mediana o grande que los configure manualmente.

**Redes Corporativas**
En una empresa con 500 empleados, asignar IPs manualmente a cada PC, laptop, teléfono IP y tablet sería inviable. DHCP centralizado permite que el área de TI controle los rangos por departamento (VLAN por piso, por área, por tipo de dispositivo), reserve IPs para servidores e impresoras, y controle el tiempo de concesión según el tipo de red (8 horas en oficina, 2 horas en sala de juntas, 30 minutos en red de invitados).

**DNS Interno Corporativo**
Empresas como bancos, retailers o manufactureras tienen decenas de servidores internos. En lugar de que los empleados memoricen IPs como `10.10.5.42`, acceden a recursos como `erp.empresa.com`, `intranet.empresa.com` o `impresora-piso3.empresa.com`. El DNS interno resuelve estos nombres a IPs privadas que nunca salen a internet.

**Infraestructura de Data Center**
En data centers modernos, cuando se aprovisiona una nueva VM o contenedor, DHCP le asigna IP automáticamente y DNS registra el hostname dinámicamente (DDNS — Dynamic DNS). Esto permite que la infraestructura escale sin intervención manual por cada nuevo servicio.

**Proveedores de Internet (ISPs)**
Cuando contratas internet en casa o en tu empresa, el router del ISP obtiene su IP pública por DHCP desde el equipo del proveedor. Los servidores DNS que usas (8.8.8.8 de Google, 1.1.1.1 de Cloudflare) son la versión pública de lo que en este lab se implementó internamente.

**Redes de Hospitales y Universidades**
Instituciones con miles de dispositivos conectados simultáneamente dependen completamente de DHCP para gestionar la asignación dinámica. Una universidad puede tener 10,000+ dispositivos conectados entre alumnos, profesores y equipos institucionales, todos obteniendo IP automáticamente según la VLAN a la que pertenecen.

---

## Ventajas y Desventajas

### ✅ Ventajas

| Ventaja | Descripción |
|---------|-------------|
| **Cero configuración manual** | Las PCs obtienen IP, máscara, gateway y DNS automáticamente sin intervención del usuario ni del administrador |
| **Control centralizado** | Todos los parámetros de red se gestionan desde un solo punto (el servidor DHCP), facilitando cambios masivos |
| **Reutilización de IPs** | Las IPs se liberan cuando el lease expira o el dispositivo se desconecta, optimizando el espacio de direccionamiento |
| **Reducción de errores** | Elimina IPs duplicadas y configuraciones incorrectas causadas por error humano |
| **DNS interno** | Permite usar nombres descriptivos para recursos internos sin depender de DNS público |
| **Escalabilidad** | Agregar 100 PCs nuevas no requiere ninguna configuración adicional si hay IPs disponibles en el pool |
| **Auditoría** | `show ip dhcp binding` muestra qué MAC address tiene cada IP en cada momento, facilitando trazabilidad |

### ❌ Desventajas

| Desventaja | Descripción |
|-----------|-------------|
| **Punto único de falla** | Si el servidor DHCP cae, los dispositivos nuevos no pueden obtener IP. Se mitiga con servidores DHCP redundantes (failover) |
| **Seguridad — Rogue DHCP** | Un atacante puede conectar su propio servidor DHCP y asignar IPs falsas con un gateway malicioso (DHCP Spoofing). Se mitiga con DHCP Snooping en los switches |
| **IPs dinámicas dificultan administración** | Servidores, impresoras y cámaras deben tener IP estática o reserva DHCP (por MAC address) para ser localizables siempre |
| **DNS único punto de falla** | Si el servidor DNS cae, los nombres no resuelven aunque la red funcione. Se mitiga con DNS secundario |
| **Cache DNS desactualizado** | Los clientes cachean respuestas DNS. Si cambias una IP en el servidor DNS, los clientes pueden seguir usando la IP vieja hasta que el TTL expire |
| **Dependencia de infraestructura** | DHCP relay requiere que los routers estén bien configurados. Un error en `ip helper-address` deja sin IP a toda una VLAN |

---

## Escalabilidad

### DHCP en redes grandes

En redes pequeñas (este lab), el router hace de servidor DHCP. En redes medianas y grandes esto cambia:

```
Red pequeña (este lab):
Router R1 = Gateway + DHCP Server

Red mediana (50-500 dispositivos):
Router = solo Gateway
Servidor Windows/Linux dedicado = DHCP + DNS
ip helper-address apunta al servidor dedicado

Red grande (500+ dispositivos):
Servidor DHCP primario + Servidor DHCP secundario (failover)
DNS interno con zonas primarias y secundarias
Integración con Active Directory (Windows) o LDAP
DDNS para registro automático de hostnames
```

### Cisco IOS DHCP vs ISC DHCP vs Windows DHCP

El DHCP de Cisco IOS que usamos en este lab es funcional para redes pequeñas pero tiene limitaciones. En producción se usan servidores dedicados como ISC DHCP en Linux o el servicio DHCP del Windows Server, que ofrecen mayor capacidad, failover nativo, integración con DNS dinámico y mejor auditoría.

### DNS Split-Horizon

Una técnica común en empresas es el **DNS Split-Horizon**: el mismo nombre (`erp.empresa.com`) resuelve a una IP diferente dependiendo de si la consulta viene desde dentro de la red corporativa (IP privada) o desde internet (IP pública). Esto permite que empleados remotos y empleados en oficina usen el mismo nombre sin exponer IPs internas.

---

## Conceptos Clave Aplicados

| Concepto | Descripción |
|---------|-------------|
| **Proceso DORA** | Discover → Offer → Request → Acknowledge. Los cuatro mensajes del handshake DHCP |
| **ip dhcp excluded-address** | Reserva IPs que DHCP nunca asignará. Se configura antes del pool |
| **ip dhcp pool** | Define la red, gateway, DNS y tiempo de concesión para un segmento |
| **lease** | Tiempo en días que una IP es válida antes de renovarse |
| **ip helper-address** | Reenvía broadcasts DHCP hacia un servidor en otra red (relay agent) |
| **Router-on-a-Stick** | Una interfaz física con múltiples subinterfaces dot1Q para rutear entre VLANs |
| **Registro A en DNS** | Mapea un nombre de dominio a una dirección IPv4 |
| **VLAN nativa** | VLAN que viaja sin etiqueta en un trunk. Debe coincidir en ambos extremos |
| **show ip dhcp binding** | Muestra las concesiones activas: IP, MAC y tiempo de expiración |
| **Split-horizon DNS** | Técnica avanzada donde el mismo nombre resuelve diferente según el origen de la consulta |

---

## 📁 Archivos del Lab

| Archivo | Descripción |
|--------|-------------|
| `lab-08-dhcp-dns.pkt` | Archivo de Packet Tracer con la topología completa configurada |
| `topologia.png` | Vista general de la topología con switches, router y servidor |
| `r1-dhcp-pool.png` | show ip dhcp pool con los tres pools activos |
| `r1-dhcp-binding.png` | show ip dhcp binding con concesiones de todas las VLANs |
| `pc-ventas1-dhcp.png` | IP Configuration de PC-Ventas1 con IP obtenida por DHCP |
| `pc-ti1-dhcp.png` | IP Configuration de PC-TI1 con IP obtenida por DHCP |
| `pc-rrhh1-dhcp.png` | IP Configuration de PC-RRHH1 con IP obtenida por DHCP |
| `dns-registros.png` | Registros A configurados en el servidor DNS |
| `ping-dns-nombre.png` | Ping exitoso usando nombre DNS desde PC-Ventas1 |

---

## 🔗 Referencias

- [RFC 2131 — DHCP](https://datatracker.ietf.org/doc/html/rfc2131)
- [RFC 1034 — DNS Concepts](https://datatracker.ietf.org/doc/html/rfc1034)
- [Cisco — DHCP Configuration Guide](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/ipaddr_dhcp/configuration/xe-16/dhcp-xe-16-book.html)
- CCNA 200-301 Official Cert Guide — Wendell Odom
- Dominio CCNA 4.1 y 4.2 — IP Services
