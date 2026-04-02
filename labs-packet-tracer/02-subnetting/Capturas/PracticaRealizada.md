# Lab 02 — Subnetting y VLSM (Implementación sin VLANs)

## Descripción general

En este laboratorio se diseñó e implementó una red empresarial utilizando **VLSM (Variable Length Subnet Mask)** a partir del bloque **192.168.100.0/24**, segmentando la red según los requerimientos de diferentes áreas (Ventas, TI, RRHH y Gerencia), además de enlaces WAN entre routers.

El objetivo principal fue lograr **conectividad extremo a extremo** entre todas las subredes mediante **enrutamiento estático**, validando el funcionamiento con herramientas como `ping` y `traceroute`.

---

## ⚠️ Adaptación del diseño (sin uso de VLANs)

Aunque el diseño inicial contempla múltiples subredes en un mismo switch, este laboratorio **no incluye el uso de VLANs**, por lo que se realizó una adaptación importante para mantener la segmentación de red sin romper el modelo OSI ni las buenas prácticas.

### Solución aplicada

Se implementó el principio clave:

> **“Una subred = una interfaz de router”**

Para lograr esto:

* Se agregaron **interfaces físicas adicionales en los routers**
* Se utilizaron **múltiples conexiones (cables) desde el switch hacia el router**
* Cada subred fue conectada a una **interfaz independiente del router**

### Ejemplo aplicado

En lugar de conectar múltiples redes a una sola interfaz:

❌ Incorrecto:

```
SW2 → R2 G0/0 → RRHH + Gerencia
```

✅ Correcto:

```
SW2 → R2 G0/0 → RRHH
SW2 → R2 G0/2 → Gerencia
```

Esto permite mantener el aislamiento de subredes **sin necesidad de VLANs**.

---

## Cambio en la conexión WAN (R2–R3)

Debido a limitaciones físicas de los routers (número de interfaces Gigabit disponibles), se realizó una adaptación en la conexión WAN:

### Implementación

* Se reemplazó la conexión Ethernet por una conexión **Serial**
* Se agregaron módulos **HWIC-2T** en los routers:

  * R2 (Cisco 2911)
  * R3 (Cisco 2911)

###  Configuración clave

* Uso de interfaces `Serial0/0/0`
* Configuración de **DCE/DTE**
* Aplicación de `clock rate` en el lado DCE
* Subneteo /30 para enlaces WAN

### Resultado

Se simuló un enlace WAN más realista, similar a entornos empresariales o de proveedores de servicios.

---

##  Proceso de troubleshooting

Durante la implementación se presentaron fallas de conectividad, las cuales fueron resueltas mediante un enfoque estructurado:

### Problemas detectados

* Hosts en diferentes subredes conectados a una misma interfaz
* Gateways incorrectos o inexistentes
* Interfaces en estado *shutdown*
* Falta de rutas estáticas de retorno

### Metodología aplicada

1. Verificación de interfaces:

```
show ip interface brief
```

2. Validación de direccionamiento:

```
show running-config
```

3. Revisión de rutas:

```
show ip route
```

4. Pruebas de conectividad:

```
ping
tracert
```

5. Corrección iterativa hasta lograr conectividad completa

---

## Aplicación en entornos reales

Este laboratorio refleja escenarios comunes en redes empresariales:

* Segmentación de departamentos mediante subredes
* Optimización de direcciones IP con VLSM
* Implementación de enlaces WAN entre sucursales
* Uso de enrutamiento estático en redes pequeñas o controladas

En entornos reales:

* La solución sin VLANs se utilizaría en redes pequeñas o con limitaciones de hardware
* En redes más grandes, se implementaría **VLAN + Router-on-a-Stick o switches capa 3**
* El uso de enlaces seriales simula conexiones WAN tradicionales o dedicadas

---

## Aprendizajes clave

* Aplicación práctica de **VLSM**
* Relación entre diseño lógico y topología física
* Importancia de las **interfaces de red en routers**
* Configuración y validación de **rutas estáticas**
* Uso de herramientas de diagnóstico (ping, traceroute)
* Resolución de problemas de red de forma estructurada
* Adaptación del diseño ante limitaciones de hardware

---
## 📁 Evidencia

Incluye:

* Topología en Packet Tracer
* Diseño de subredes (VLSM)
* Tablas de enrutamiento
* Pruebas de conectividad (ping)
* Validación de ruta (traceroute)

---

## Conclusión

Se logró implementar exitosamente una red segmentada utilizando VLSM y enrutamiento estático, adaptando el diseño para funcionar sin VLANs mediante el uso de múltiples interfaces físicas.

Este laboratorio fortalece habilidades clave de nivel **CCNA**, combinando teoría, práctica y resolución de problemas en un entorno simulado realista.
