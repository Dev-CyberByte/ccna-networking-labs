# Práctica Realizada — VLANs y Trunking

## Descripción

En esta práctica se implementó la segmentación lógica de una red local mediante el uso de **VLANs (Virtual LANs)** en switches Cisco, permitiendo separar distintos departamentos dentro de una misma infraestructura física.

Se configuraron múltiples switches interconectados mediante enlaces **trunk (802.1Q)**, asegurando la comunicación entre dispositivos pertenecientes a la misma VLAN, incluso si se encuentran en diferentes switches.

---

## Objetivo técnico

* Implementar segmentación de red en capa 2
* Configurar VLANs para distintos departamentos
* Establecer enlaces trunk entre switches
* Validar el aislamiento de tráfico entre VLANs

---

## Implementación

### Segmentación por VLANs

Se crearon las siguientes VLANs:

* VLAN 10 — Ventas
* VLAN 20 — TI
* VLAN 30 — RRHH
* VLAN 99 — Administración

Cada VLAN representa un **dominio de broadcast independiente**, lo que permite separar el tráfico de red por áreas.

---

### Configuración de puertos

* Puertos configurados en modo **access** para dispositivos finales (PCs)
* Cada puerto asignado a su VLAN correspondiente
* Garantiza que cada equipo solo pertenezca a su segmento lógico

---

### Enlace entre switches (Trunking)

Se configuró un enlace trunk entre switches utilizando **encapsulación 802.1Q**, permitiendo transportar múltiples VLANs a través de un solo enlace físico.

Configuraciones clave:

* VLAN nativa: 99
* VLANs permitidas: 10, 20, 30, 99

---

### VLAN de administración

Se implementó una VLAN dedicada para administración (VLAN 99), permitiendo gestionar los switches mediante direcciones IP asignadas a interfaces virtuales (SVI).

---

## Validación

Se realizaron pruebas para verificar el comportamiento esperado:

* ✔ Comunicación exitosa entre dispositivos de la misma VLAN
* ❌ Bloqueo de comunicación entre VLANs distintas (sin routing)

Esto confirma la correcta segmentación lógica de la red.

---

## Aplicación en entornos reales

Este tipo de implementación es ampliamente utilizado en redes empresariales:

* Separación de departamentos (Finanzas, TI, RH, etc.)
* Mejora en seguridad al aislar tráfico
* Reducción de dominios de broadcast
* Optimización del uso de infraestructura física

En escenarios reales:

* Las VLANs se combinan con **routing inter-VLAN** para permitir comunicación controlada entre departamentos
* Se aplican políticas de seguridad (ACLs)
* Se integran con switches de capa 3 o routers

---

## Conclusión

Se logró implementar una red segmentada mediante VLANs y trunking, garantizando el aislamiento del tráfico entre departamentos y permitiendo la comunicación dentro de cada segmento.

Esta práctica sienta las bases para la implementación de **routing inter-VLAN**, ampliando la funcionalidad de la red en escenarios más avanzados.
