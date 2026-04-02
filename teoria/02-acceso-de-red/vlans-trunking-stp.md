# VLANs, Trunking y STP

Este módulo cubre cómo los switches segmentan el tráfico, cómo transportan
múltiples VLANs por un solo enlace y cómo evitan loops en la red.
Son los tres conceptos más importantes de la capa 2.

---

## VLANs — Virtual Local Area Networks

### ¿Qué es una VLAN?
Una VLAN es una red lógica dentro de un switch físico. Permite agrupar
puertos en redes separadas aunque estén en el mismo equipo. El tráfico
de una VLAN no puede llegar a otra VLAN sin pasar por un router.

### ¿Por qué usar VLANs?

| Beneficio | Explicación |
|-----------|-------------|
| Segmentación | Separa tráfico por departamento o función |
| Seguridad | Un dispositivo en VLAN 10 no ve tráfico de VLAN 20 |
| Rendimiento | Reduce el dominio de broadcast |
| Flexibilidad | Un usuario puede cambiar de VLAN sin mover cables |

### Dominio de broadcast
Sin VLANs, todos los dispositivos del switch están en el mismo dominio
de broadcast. Un broadcast llega a todos. Con VLANs, cada VLAN es su
propio dominio de broadcast.
```
Sin VLANs:                     Con VLANs:
┌─────────────────────┐        ┌─────────────────────┐
│  Switch             │        │  Switch             │
│  PC1 PC2 PC3 PC4   │        │  [VLAN10] [VLAN20]  │
│  ←── broadcast ──► │        │  PC1 PC2 │ PC3 PC4  │
│  todos se ven       │        │  no se ven entre sí │
└─────────────────────┘        └─────────────────────┘
```

### VLANs reservadas y de uso común

| VLAN | Nombre | Descripción |
|------|--------|-------------|
| 1 | Default | VLAN por defecto, todos los puertos arrancan aquí |
| 2 - 1001 | Normal | VLANs de uso general |
| 1002 - 1005 | Legacy | Reservadas para Token Ring y FDDI |
| 1006 - 4094 | Extended | VLANs extendidas |

> La VLAN 1 nunca se debe usar para tráfico de usuarios. Es un riesgo
> de seguridad porque es la VLAN nativa por defecto.

---

## Tipos de puertos en un switch

### Access Port (Puerto de acceso)
- Pertenece a una sola VLAN
- Conecta dispositivos finales (PCs, impresoras, teléfonos)
- El dispositivo conectado no sabe que existen VLANs
```
Switch
├── Fa0/1  → VLAN 10  (PC Ventas)
├── Fa0/2  → VLAN 10  (PC Ventas)
├── Fa0/3  → VLAN 20  (PC TI)
└── Fa0/4  → VLAN 20  (PC TI)
```

### Trunk Port (Puerto troncal)
- Transporta tráfico de múltiples VLANs
- Conecta switches entre sí, o switch con router
- Usa etiquetas 802.1Q para identificar a qué VLAN pertenece cada frame

---

## Trunking — 802.1Q

### ¿Qué es el trunking?
Es el mecanismo para transportar tráfico de múltiples VLANs por un
solo enlace físico. El estándar actual es **IEEE 802.1Q**.

### ¿Cómo funciona 802.1Q?
Agrega una etiqueta de 4 bytes al frame Ethernet para identificar
a qué VLAN pertenece:
```
Frame Ethernet normal:
┌──────────┬────────┬──────┬─────────┐
│ MAC dst  │ MAC src│ Type │  Datos  │
└──────────┴────────┴──────┴─────────┘

Frame con etiqueta 802.1Q (tagged):
┌──────────┬────────┬───────────┬──────┬─────────┐
│ MAC dst  │ MAC src│  802.1Q   │ Type │  Datos  │
│          │        │ (4 bytes) │      │         │
└──────────┴────────┴───────────┴──────┴─────────┘
                         │
                    ┌────┴────┐
                    │VLAN ID  │ ← aquí va el número de VLAN
                    │(12 bits)│   permite hasta 4094 VLANs
                    └─────────┘
```

### VLAN Nativa
La VLAN nativa es la única que viaja **sin etiqueta** por un trunk.
Por defecto es la VLAN 1. Ambos extremos del trunk deben tener
la misma VLAN nativa, si no hay problemas de comunicación.

> Buena práctica: cambiar la VLAN nativa a una VLAN sin uso
> (por ejemplo VLAN 999) para evitar ataques de VLAN hopping.

---

## Configuración de VLANs en Cisco IOS

### Crear VLANs
```
Switch# configure terminal
Switch(config)# vlan 10
Switch(config-vlan)# name Ventas
Switch(config-vlan)# exit

Switch(config)# vlan 20
Switch(config-vlan)# name TI
Switch(config-vlan)# exit

Switch(config)# vlan 30
Switch(config-vlan)# name RRHH
Switch(config-vlan)# exit
```

### Configurar puerto de acceso
```
Switch(config)# interface FastEthernet0/1
Switch(config-if)# switchport mode access
Switch(config-if)# switchport access vlan 10
Switch(config-if)# description PC-Ventas
Switch(config-if)# exit
```

### Configurar puerto trunk
```
Switch(config)# interface GigabitEthernet0/1
Switch(config-if)# switchport mode trunk
Switch(config-if)# switchport trunk native vlan 999
Switch(config-if)# switchport trunk allowed vlan 10,20,30
Switch(config-if)# description Uplink-a-Router
Switch(config-if)# exit
```

### Verificación
```
Switch# show vlan brief
Switch# show interfaces trunk
Switch# show interfaces FastEthernet0/1 switchport
```

---

## Inter-VLAN Routing

Las VLANs están aisladas entre sí. Para que se comuniquen necesitan
un router o una capa 3. Hay dos métodos:

### Método 1 — Router on a Stick
Un router con una sola interfaz física dividida en subinterfaces.
Cada subinterfaz atiende una VLAN.
```
          Router
            │  (una sola interfaz física)
            │  G0/0
     ───────┴───────
     │   Switch    │
     │             │
  VLAN10        VLAN20
  Ventas          TI
```

Configuración del router:
```
Router(config)# interface GigabitEthernet0/0
Router(config-if)# no shutdown
Router(config-if)# exit

Router(config)# interface GigabitEthernet0/0.10
Router(config-subif)# encapsulation dot1Q 10
Router(config-subif)# ip address 192.168.10.1 255.255.255.0
Router(config-subif)# exit

Router(config)# interface GigabitEthernet0/0.20
Router(config-subif)# encapsulation dot1Q 20
Router(config-subif)# ip address 192.168.20.1 255.255.255.0
Router(config-subif)# exit
```

### Método 2 — Switch de Capa 3 (SVI)
El switch mismo enruta entre VLANs usando interfaces virtuales (SVI).
Más eficiente que Router on a Stick.
```
Switch(config)# ip routing

Switch(config)# interface vlan 10
Switch(config-if)# ip address 192.168.10.1 255.255.255.0
Switch(config-if)# no shutdown
Switch(config-if)# exit

Switch(config)# interface vlan 20
Switch(config-if)# ip address 192.168.20.1 255.255.255.0
Switch(config-if)# no shutdown
Switch(config-if)# exit
```

---

## STP — Spanning Tree Protocol

### El problema: loops en capa 2
En redes reales se usan enlaces redundantes para alta disponibilidad.
Pero la redundancia en capa 2 causa loops: los frames se replican
infinitamente porque Ethernet no tiene TTL.
```
         Switch A
        /         \
   Switch B ─── Switch C
```

Si hay un broadcast, viaja en loop infinito entre los tres switches.
Esto satura la red en segundos. Se llama **broadcast storm**.

### ¿Qué hace STP?
STP (IEEE 802.1D) detecta los loops y bloquea los puertos redundantes
dejando activos solo los necesarios. Si un enlace falla, desbloquea
el puerto de respaldo automáticamente.

### Terminología STP

| Término | Descripción |
|---------|-------------|
| Root Bridge | El switch central del árbol STP. Todos los caminos parten desde aquí |
| BPDU | Bridge Protocol Data Unit. Mensajes que los switches intercambian para elegir el Root Bridge |
| Bridge ID | Prioridad (32768 default) + MAC address. Gana el menor |
| Root Port | El puerto de cada switch no-root con el mejor camino al Root Bridge |
| Designated Port | El puerto que reenvía tráfico en cada segmento |
| Blocked Port | Puerto que no reenvía tráfico para evitar loops |

### Elección del Root Bridge
Gana el switch con el **Bridge ID más bajo**.
Bridge ID = Prioridad + MAC address.
Como todos tienen prioridad 32768 por default, gana el de MAC más baja.

> Buena práctica: configurar manualmente la prioridad para que el
> switch más potente sea el Root Bridge.
```
Switch(config)# spanning-tree vlan 10 priority 4096
```

### Estados de los puertos STP
```
Blocking  →  Listening  →  Learning  →  Forwarding
(bloqueado)  (escuchando)  (aprendiendo) (enviando)
```

| Estado | Recibe BPDUs | Aprende MACs | Envía datos |
|--------|-------------|--------------|-------------|
| Blocking | Sí | No | No |
| Listening | Sí | No | No |
| Learning | Sí | Sí | No |
| Forwarding | Sí | Sí | Sí |

El tiempo de convergencia de STP 802.1D es de **30 a 50 segundos**.
Por eso existe RSTP.

### RSTP — Rapid Spanning Tree Protocol
IEEE 802.1W. Versión mejorada de STP con convergencia en **menos de 6 segundos**.
Es el estándar actual. Cisco lo llama **Rapid PVST+** (uno por VLAN).
```
Switch(config)# spanning-tree mode rapid-pvst
```

### Verificación STP
```
Switch# show spanning-tree
Switch# show spanning-tree vlan 10
Switch# show spanning-tree summary
```

---

## Port Security

Permite controlar qué dispositivos pueden conectarse a un puerto del switch
basándose en la dirección MAC.
```
Switch(config)# interface FastEthernet0/1
Switch(config-if)# switchport mode access
Switch(config-if)# switchport port-security
Switch(config-if)# switchport port-security maximum 1
Switch(config-if)# switchport port-security mac-address sticky
Switch(config-if)# switchport port-security violation shutdown
Switch(config-if)# exit
```

### Modos de violación

| Modo | Descarta frame | Incrementa contador | Deshabilita puerto |
|------|---------------|--------------------|--------------------|
| Protect | Sí | No | No |
| Restrict | Sí | Sí | No |
| Shutdown | Sí | Sí | Sí |

### Verificación
```
Switch# show port-security
Switch# show port-security interface FastEthernet0/1
Switch# show port-security address
```

---

## En el examen CCNA

Preguntas típicas de este módulo:

- *¿Qué hace una VLAN con el dominio de broadcast?* → Lo reduce, cada VLAN es su propio dominio
- *¿Qué estándar usa el trunking moderno?* → IEEE 802.1Q
- *¿Qué VLAN viaja sin etiqueta en un trunk?* → La VLAN nativa (default VLAN 1)
- *¿Qué problema resuelve STP?* → Los loops de capa 2 y broadcast storms
- *¿Cómo se elige el Root Bridge?* → El que tenga el Bridge ID más bajo
- *¿Cuánto tarda en converger RSTP?* → Menos de 6 segundos
- *¿Qué comando activa RSTP en Cisco?* → spanning-tree mode rapid-pvst
- *¿Qué modo de port-security desactiva el puerto?* → Shutdown
