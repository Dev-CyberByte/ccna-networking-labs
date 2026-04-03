# Lab 07 — OSPF Single Area 

![OSPF](https://img.shields.io/badge/Protocol-OSPF-blue?style=flat-square)
![CCNA](https://img.shields.io/badge/Cert-CCNA-red?style=flat-square)
![Packet Tracer](https://img.shields.io/badge/Tool-Packet%20Tracer-green?style=flat-square)
![Difficulty](https://img.shields.io/badge/Difficulty-Advanced-orange?style=flat-square)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen?style=flat-square)

---

## Descripción

Este laboratorio implementa **OSPF (Open Shortest Path First)** de área única (Area 0) sobre una topología de cuatro routers interconectados en Cisco Packet Tracer. Se trata de un protocolo de enrutamiento dinámico de estado de enlace (Link-State) ampliamente utilizado en redes empresariales y de proveedores de servicio.

A diferencia del enrutamiento estático, OSPF permite que los routers **aprendan rutas automáticamente**, se adapten a cambios de topología sin intervención manual y calculen siempre el camino óptimo usando el **algoritmo de Dijkstra (SPF — Shortest Path First)**.

---

## Objetivos del Lab

- Configurar OSPF proceso 1 en cuatro routers dentro del área backbone (Area 0)
- Asignar Router IDs manualmente para garantizar identidad estable
- Verificar la formación de adyacencias OSPF en estado **FULL**
- Analizar la LSDB (Link-State Database) y la tabla de enrutamiento OSPF
- Configurar `passive-interface` en interfaces LAN para optimizar el tráfico de control
- Propagar una ruta por defecto desde R1 hacia toda el área OSPF
- Ajustar costos con `auto-cost reference-bandwidth` para distinguir velocidades de interfaz
- Validar **convergencia automática** ante la caída de un enlace

---

## Topología

```
LAN1                                    LAN2
192.168.1.0/24                          192.168.2.0/24
    |                                       |
   R1 ─────────── 10.0.12.0/30 ──────────R2
    |                                       |
10.0.13.0/30                          10.0.23.0/30
    |                                       |
   R3 ─────────── 10.0.34.0/30 ──────────R4 ── LAN4
    |                                           192.168.4.0/24
   LAN3
192.168.3.0/24

          [ Todos los routers en Area 0 ]
```

##  Implementación

### Parte 1 — Configuración de Interfaces

Cada router requiere configuración de dirección IP y activación de interfaces antes de habilitar OSPF. Ejemplo en R1:

```bash
R1(config)# interface GigabitEthernet0/0
R1(config-if)# ip address 192.168.1.1 255.255.255.0
R1(config-if)# no shutdown
```

### Parte 2 — Configuración OSPF

OSPF se habilita con `router ospf 1`, se asigna un Router ID estático y se anuncia cada red con su wildcard mask:

```bash
R1(config)# router ospf 1
R1(config-router)# router-id 1.1.1.1
R1(config-router)# network 192.168.1.0 0.0.0.255 area 0
R1(config-router)# network 10.0.12.0 0.0.0.3 area 0
R1(config-router)# network 10.0.13.0 0.0.0.3 area 0
R1(config-router)# passive-interface GigabitEthernet0/0
```

> `passive-interface` suprime los Hellos OSPF en interfaces LAN donde no hay routers vecinos, reduciendo tráfico innecesario y evitando posibles problemas de seguridad.

### Parte 3 — Propagación de Ruta por Defecto

R1 actúa como ASBR (Autonomous System Boundary Router) simulando salida a Internet:

```bash
R1(config)# ip route 0.0.0.0 0.0.0.0 209.165.200.1
R1(config)# router ospf 1
R1(config-router)# default-information originate
```

Esto genera un LSA tipo 5 que propaga la ruta `0.0.0.0/0` a todos los routers del área. En R2, R3 y R4 aparece como:

```
O*E2 0.0.0.0/0 [110/1] via ...
```

### Parte 4 — Ajuste de Costos

Por defecto Packet Tracer asigna costo 1 a GigabitEthernet, lo que no diferencia velocidades. Se ajusta el ancho de banda de referencia globalmente:

```bash
R1(config)# router ospf 1
R1(config-router)# auto-cost reference-bandwidth 1000
```

> ⚠️ Debe configurarse **idéntico en todos los routers** del área. Si no coincide, el algoritmo SPF calculará costos inconsistentes y las rutas óptimas pueden ser incorrectas.

---

## ✅ Verificación

```bash
# Ver adyacencias OSPF (deben estar en FULL)
R1# show ip ospf neighbor

# Ver rutas aprendidas por OSPF
R1# show ip route ospf

# Ver base de datos de estado de enlace
R1# show ip ospf database

# Ver detalles de OSPF en una interfaz
R1# show ip ospf interface GigabitEthernet0/1

# Verificar ruta por defecto en R4
R4# show ip route
```

### Resultados Esperados en R1

```
Neighbor ID   Pri   State       Dead Time   Address      Interface
2.2.2.2         1   FULL/DR     00:00:38    10.0.12.2    Gi0/1
3.3.3.3         1   FULL/DR     00:00:36    10.0.13.3    Gi0/2
```

---

##  Prueba de Convergencia

Una de las ventajas más importantes de OSPF es su capacidad de **reconvergencia automática**. Al desconectar el enlace R1-R2:

```bash
R1(config-if)# shutdown   # simula caída del enlace
```

OSPF detecta la falla en menos de 10 segundos y recalcula la ruta a 192.168.4.0/24 de forma automática, pasando de usar R2 como siguiente salto a usar R3, sin ninguna intervención manual.

---

## Aplicación en Entornos Reales

### ¿Dónde se usa OSPF?

OSPF es uno de los protocolos de enrutamiento más desplegados en redes empresariales y de proveedores de servicio a nivel mundial. Algunos escenarios reales:

**Redes Corporativas (Campus y WAN)**
Empresas medianas y grandes con múltiples sedes utilizan OSPF para que sus routers de distribución y core aprendan rutas automáticamente. Por ejemplo, una empresa con oficinas en CDMX, Guadalajara y Monterrey puede tener un área OSPF por ciudad y conectarlas todas al backbone Area 0, sin necesidad de configurar cientos de rutas estáticas.

**Data Centers**
En arquitecturas de data center modernas, OSPF se usa entre routers de borde (edge) y routers de distribución para garantizar que si un enlace falla, el tráfico se redistribuya automáticamente en milisegundos usando rutas alternativas precomputadas.

**Proveedores de Internet (ISPs)**
Los ISPs utilizan OSPF dentro de su red interna (IGP — Interior Gateway Protocol) para gestionar el enrutamiento entre sus propios equipos antes de redistribuir rutas al exterior vía BGP.

**Redes de Telecomunicaciones**
Operadoras como Telmex, AT&T o Izzi despliegan OSPF en su infraestructura de backbone para garantizar alta disponibilidad y convergencia rápida ante fallas físicas.

---

## Ventajas y Desventajas

### ✅ Ventajas

| Ventaja | Descripción |
|--------|-------------|
| **Convergencia rápida** | Detecta cambios de topología y recalcula rutas en segundos, a diferencia de RIP que puede tardar minutos |
| **Sin límite de saltos** | RIP tiene un máximo de 15 saltos; OSPF no tiene esta limitación |
| **Métrica basada en costo** | Usa el ancho de banda real de los enlaces, eligiendo siempre el camino más eficiente |
| **Jerárquico (multi-area)** | Soporta división en áreas, reduciendo el tamaño de la LSDB y el procesamiento SPF en redes grandes |
| **Estándar abierto** | Definido en RFC 2328, funciona en equipos de cualquier fabricante (Cisco, Juniper, Huawei, etc.) |
| **Soporte VLSM** | Compatible con máscaras de longitud variable y CIDR, esencial en redes modernas |
| **Autenticación** | Soporta autenticación MD5 entre vecinos para mayor seguridad |

### ❌ Desventajas

| Desventaja | Descripción |
|-----------|-------------|
| **Complejidad de configuración** | Más complejo que RIP o rutas estáticas, requiere conocimiento de áreas, LSAs, DR/BDR y wildcard masks |
| **Consume más recursos** | El algoritmo SPF de Dijkstra es intensivo en CPU y memoria, especialmente en topologías grandes |
| **Diseño de áreas crítico** | Un diseño de áreas mal planificado puede generar problemas de escalabilidad difíciles de corregir |
| **Troubleshooting complejo** | Diagnosticar problemas de adyacencias, LSAs o cálculo SPF requiere experiencia y conocimiento profundo del protocolo |
| **No apto para redes muy grandes sin diseño** | Sin una jerarquía de áreas bien definida, una sola área con cientos de routers puede saturar recursos |

---

## 📈 Escalabilidad

OSPF está diseñado para escalar desde redes pequeñas hasta redes de nivel empresarial y de proveedor. La clave está en su **diseño jerárquico de dos niveles**:

```
                    ┌─────────────────┐
                    │    Area 0       │
                    │  (Backbone)     │
                    │  R1  R2  R3     │
                    └────────┬────────┘
                             │
              ┌──────────────┼──────────────┐
              │              │              │
         ┌────┴────┐    ┌────┴────┐    ┌────┴────┐
         │ Area 1  │    │ Area 2  │    │ Area 3  │
         │ Sede MX │    │ Sede GDL│    │ Sede MTY│
         └─────────┘    └─────────┘    └─────────┘
```

**Single Area (este lab):** Adecuado para redes de hasta ~50 routers. Toda la topología está en Area 0 y todos los routers mantienen la LSDB completa.

**Multi-Area:** Para redes mayores se divide en áreas. Cada área mantiene su propia LSDB reducida. Los ABRs (Area Border Routers) resumen la información entre áreas, reduciendo drásticamente el procesamiento SPF y el tráfico de control.

**OSPFv3:** La versión para IPv6, con la misma lógica pero adaptada al nuevo espacio de direccionamiento, lista para redes modernas dual-stack.

En la práctica, OSPF bien diseñado puede manejar redes con miles de routers cuando se organiza correctamente en áreas, lo que lo convierte en la opción preferida para redes empresariales de escala media a grande antes de necesitar BGP.

---

## 🧠 Conceptos Clave Aplicados

| Concepto | Descripción |
|---------|-------------|
| **Router ID** | Identificador único del router en OSPF. Se configura manualmente para garantizar estabilidad |
| **Wildcard Mask** | Inverso de la máscara de subred, usado en el comando `network` para definir qué interfaces participan |
| **Area 0 (Backbone)** | Área central obligatoria en OSPF. Todas las demás áreas deben conectarse a ella |
| **passive-interface** | Suprime Hellos en interfaces sin vecinos OSPF, optimizando recursos |
| **Adyacencia FULL** | Estado correcto entre vecinos OSPF, indica intercambio completo de LSDB |
| **LSDB** | Base de datos idéntica en todos los routers del área, describe la topología completa |
| **SPF / Dijkstra** | Algoritmo que calcula el árbol de rutas más cortas desde la LSDB |
| **LSA Tipo 5** | Usado para propagar rutas externas como la ruta por defecto desde el ASBR |
| **auto-cost reference-bandwidth** | Ajusta la métrica de costo para reflejar correctamente velocidades GigabitEthernet |
| **Convergencia** | Capacidad de OSPF de recalcular rutas automáticamente ante cambios de topología |

---

## 📁 Archivos del Lab

| Archivo | Descripción |
|--------|-------------|
| `lab-07-ospf.pkt` | Archivo de Packet Tracer con la topología completa configurada |
| `topologia.png` | Vista general de la topología con los cuatro routers |
| `r1-ospf-neighbor.png` | Adyacencias OSPF en estado FULL desde R1 |
| `r1-ip-route-ospf.png` | Tabla de enrutamiento OSPF en R1 |
| `r4-ruta-default.png` | Ruta O*E2 aprendida en R4 |
| `r1-ospf-database.png` | LSDB completa en R1 |
| `ping-pc1-pc4.png` | Conectividad extremo a extremo verificada |
| `traceroute-pc1-pc4.png` | Camino de paquetes PC1 → PC4 |
| `convergencia-ospf.png` | Reconvergencia automática tras caída de enlace |

---

## Referencias

- [RFC 2328 — OSPF Version 2](https://datatracker.ietf.org/doc/html/rfc2328)
- [Cisco — OSPF Design Guide](https://www.cisco.com/c/en/us/support/docs/ip/open-shortest-path-first-ospf/7039-1.html)
- CCNA 200-301 Official Cert Guide — Wendell Odom
- Dominio CCNA 3.3 y 3.4 — IP Connectivity

---
