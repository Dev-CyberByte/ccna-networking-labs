# Enrutamiento y OSPF

El enrutamiento es el proceso de mover paquetes entre redes distintas.
Un router analiza la IP destino de cada paquete y decide por qué interfaz
enviarlo consultando su tabla de enrutamiento. Este módulo cubre enrutamiento
estático y el protocolo dinámico OSPF.

---

## La tabla de enrutamiento

Es la base de todo router. Contiene las redes que conoce y por dónde
llegar a ellas.
```
Router# show ip route

Codes: C - connected, S - static, O - OSPF, R - RIP

      10.0.0.0/8 is variably subnetted
C        10.0.0.0/30 is directly connected, GigabitEthernet0/0
O        10.0.1.0/30 [110/2] via 10.0.0.2, GigabitEthernet0/0
S        192.168.1.0/24 [1/0] via 10.0.0.2
S*       0.0.0.0/0 [1/0] via 203.0.113.1
```

### Cómo leer una ruta
```
O   192.168.1.0/24   [110/2]   via 10.0.0.2,   GigabitEthernet0/0
│   │                 │   │        │              │
│   │                 │   │        │              └── Interfaz de salida
│   │                 │   │        └── Siguiente salto (next-hop)
│   │                 │   └── Métrica (costo)
│   │                 └── Distancia administrativa
│   └── Red destino
└── Fuente (O = OSPF)
```

### Distancia Administrativa (AD)

Cuando hay varias fuentes que conocen una misma red, el router prefiere
la de menor AD. Es la confiabilidad de la fuente.

| Fuente | AD |
|--------|----|
| Directamente conectada | 0 |
| Ruta estática | 1 |
| OSPF | 110 |
| RIP | 120 |
| Desconocida | 255 (nunca se usa) |

---

## Enrutamiento estático

El administrador configura manualmente cada ruta. El router no aprende
rutas por sí solo. Útil en redes pequeñas o para rutas específicas.

### Sintaxis
```
Router(config)# ip route [red-destino] [máscara] [next-hop | interfaz] [AD]
```

### Ejemplo de topología
```
PC1 ── R1 ──── R2 ── PC2
       G0/0   G0/1
  .1        .2   .1        .2
192.168.1.0/24   10.0.0.0/30   192.168.2.0/24
```

### Configuración R1
```
R1(config)# ip route 192.168.2.0 255.255.255.0 10.0.0.2
```

### Configuración R2
```
R2(config)# ip route 192.168.1.0 255.255.255.0 10.0.0.1
```

### Ruta estática por defecto
Envía todo el tráfico sin ruta específica hacia un next-hop.
Se usa para la salida a internet.
```
Router(config)# ip route 0.0.0.0 0.0.0.0 203.0.113.1
```

En la tabla de enrutamiento aparece como `S*` (candidata a ruta por defecto).

### Ruta estática flotante
Tiene una AD mayor que la ruta principal. Actúa como respaldo:
solo se activa si la ruta principal desaparece.
```
Router(config)# ip route 192.168.2.0 255.255.255.0 10.0.0.2 1
Router(config)# ip route 192.168.2.0 255.255.255.0 10.0.1.2 5
```
La segunda ruta (AD=5) solo se usa si la primera (AD=1) falla.

### Verificación
```
Router# show ip route
Router# show ip route static
Router# ping 192.168.2.1
Router# traceroute 192.168.2.1
```

---

## Enrutamiento dinámico

El router aprende rutas automáticamente intercambiando información
con otros routers. Se adapta solo a cambios en la topología.

### Tipos de protocolos

| Tipo | Protocolo | Descripción |
|------|-----------|-------------|
| Distance Vector | RIP | Usa saltos, lento, obsoleto |
| Link State | OSPF | Usa costo, rápido, estándar actual |
| Path Vector | BGP | Internet, entre organizaciones |
| Híbrido | EIGRP | Cisco propietario |

---

## OSPF — Open Shortest Path First

### ¿Qué es OSPF?
Es un protocolo de enrutamiento de estado de enlace (Link State).
Cada router construye un mapa completo de la topología y calcula
el mejor camino usando el **algoritmo de Dijkstra (SPF)**.

Es el protocolo de enrutamiento dinámico más importante del examen CCNA.

### Características principales

| Característica | Valor |
|----------------|-------|
| Estándar | IEEE (abierto, no propietario) |
| AD | 110 |
| Métrica | Costo (basado en ancho de banda) |
| Algoritmo | Dijkstra / SPF |
| Protocolo | IP, número 89 |
| Actualizaciones | Solo cuando hay cambios (no periódicas) |
| Convergencia | Rápida |

### Costo OSPF
El costo se calcula así:
```
Costo = 100,000,000 / Ancho de banda (bps)
```

| Interfaz | Ancho de banda | Costo |
|----------|---------------|-------|
| Serial (T1) | 1.544 Mbps | 64 |
| Ethernet | 10 Mbps | 10 |
| FastEthernet | 100 Mbps | 1 |
| GigabitEthernet | 1000 Mbps | 1 |

> FastEthernet y GigabitEthernet tienen el mismo costo por defecto (1).
> Hay que ajustarlo manualmente con `bandwidth` o `ip ospf cost`.

---

## Cómo funciona OSPF

### Paso 1 — Establecer adyacencias (vecinos)
Los routers se descubren entre sí enviando paquetes **Hello** por sus
interfaces OSPF. Para formar vecindad deben coincidir:

- Area ID
- Hello y Dead interval
- Tipo de red
- MTU
- Stub area flag
- Autenticación
```
Router A ──── Hello ────► Router B
Router A ◄─── Hello ───── Router B
           (adyacencia formada)
```

### Paso 2 — Intercambiar información de topología (LSA)
Cada router genera un **LSA (Link State Advertisement)** describiendo
sus interfaces y vecinos. Los LSAs se inundan a toda el área.

### Paso 3 — Construir la base de datos (LSDB)
Todos los routers del área acumulan los LSAs en su
**LSDB (Link State Database)**. Al final todos tienen el mismo mapa.

### Paso 4 — Calcular las rutas (SPF)
Cada router corre el **algoritmo SPF (Dijkstra)** sobre la LSDB
y calcula el árbol de caminos más cortos desde sí mismo.
El resultado llena la tabla de enrutamiento.
```
LSDB ──► Algoritmo SPF ──► Tabla de enrutamiento
(mapa)                      (rutas activas)
```

---

## OSPF en área única (Single Area OSPF)

Para el CCNA el enfoque es **OSPF área 0** (backbone area).
Toda la red está en una sola área.

### Topología de ejemplo
```
           Area 0
    ┌────────────────────┐
    │                    │
   R1 ──── R2 ──── R3   │
    │  10.0.0.0   10.0.1.0  │
    └────────────────────┘

R1: 192.168.1.0/24 (LAN)
R3: 192.168.3.0/24 (LAN)
```

### Configuración básica OSPF

#### Router 1
```
R1(config)# router ospf 1
R1(config-router)# router-id 1.1.1.1
R1(config-router)# network 192.168.1.0 0.0.0.255 area 0
R1(config-router)# network 10.0.0.0 0.0.0.3 area 0
R1(config-router)# passive-interface GigabitEthernet0/1
R1(config-router)# exit
```

#### Router 2
```
R2(config)# router ospf 1
R2(config-router)# router-id 2.2.2.2
R2(config-router)# network 10.0.0.0 0.0.0.3 area 0
R2(config-router)# network 10.0.1.0 0.0.0.3 area 0
R2(config-router)# exit
```

#### Router 3
```
R3(config)# router ospf 1
R3(config-router)# router-id 3.3.3.3
R3(config-router)# network 10.0.1.0 0.0.0.3 area 0
R3(config-router)# network 192.168.3.0 0.0.0.255 area 0
R3(config-router)# passive-interface GigabitEthernet0/1
R3(config-router)# exit
```

### Puntos clave de la configuración

**Router ID:** Identificador único del router en OSPF.
Se elige en este orden:
1. Router ID configurado manualmente (recomendado)
2. IP más alta de una interfaz loopback activa
3. IP más alta de una interfaz física activa

**Wildcard mask:** Es el inverso de la máscara de subred.
```
Máscara:  255.255.255.0  →  Wildcard: 0.0.0.255
Máscara:  255.255.255.252 →  Wildcard: 0.0.0.3
```

**Passive interface:** Evita que OSPF envíe Hellos por interfaces
donde no hay routers vecinos (como interfaces LAN con PCs).
Ahorra recursos y mejora la seguridad.

### Propagar la ruta por defecto
Para que los demás routers OSPF aprendan la salida a internet:
```
R1(config)# router ospf 1
R1(config-router)# default-information originate
```

---

## Verificación OSPF
```
Router# show ip ospf neighbor
Router# show ip ospf
Router# show ip ospf interface
Router# show ip route ospf
Router# show ip protocols
```

### Salida de show ip ospf neighbor
```
Neighbor ID   Pri  State     Dead Time  Address      Interface
2.2.2.2        1   FULL/DR   00:00:38   10.0.0.2     Gi0/0
3.3.3.3        1   FULL/BDR  00:00:36   10.0.0.6     Gi0/1
```

### Estados de vecindad OSPF
```
Down → Init → 2-Way → Exstart → Exchange → Loading → Full
```

| Estado | Descripción |
|--------|-------------|
| Down | Sin comunicación |
| Init | Se recibió un Hello pero no hay respuesta aún |
| 2-Way | Ambos routers se ven entre sí |
| Full | Intercambio completo, adyacencia establecida |

El estado normal entre vecinos es **FULL**.

---

## DR y BDR en redes multi-acceso

En redes Ethernet (multi-acceso) OSPF elige un **DR (Designated Router)**
y un **BDR (Backup Designated Router)** para reducir el tráfico de LSAs.

- Todos los routers forman adyacencia FULL solo con el DR y BDR
- El DR distribuye la información a los demás
- Gana el de mayor prioridad OSPF (default 1), en empate gana mayor Router ID
```
Router(config-if)# ip ospf priority 100
```

> Poner prioridad 0 hace que el router nunca sea DR ni BDR.

---

## En el examen CCNA

Preguntas típicas de este módulo:

- *¿Qué AD tiene OSPF?* → 110
- *¿Cómo calcula OSPF el costo?* → 100,000,000 / ancho de banda
- *¿Qué algoritmo usa OSPF?* → Dijkstra / SPF
- *¿Qué es la wildcard mask?* → El inverso de la máscara de subred
- *¿Para qué sirve passive-interface?* → Evitar Hellos en interfaces sin routers vecinos
- *¿Qué estado debe tener un vecino OSPF funcional?* → FULL
- *¿Cómo se elige el Router ID?* → Manual > Loopback más alta > Física más alta
- *¿Qué comando propaga la ruta por defecto en OSPF?* → default-information originate
- *¿Qué es el DR en OSPF?* → El router designado en redes multi-acceso
