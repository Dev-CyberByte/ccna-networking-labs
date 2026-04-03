# Lab 04 — Inter-VLAN Routing (Router-on-a-Stick vs Layer 3 SVI)

# Descripción general

En esta práctica se implementó comunicación entre múltiples VLANs utilizando dos enfoques ampliamente utilizados en redes empresariales:

*Router-on-a-Stick (RoS)*
*Switch Multicapa (Layer 3) con SVIs*

Se diseñó una red segmentada por departamentos (Ventas, TI, RRHH y Administración), aplicando principios de segmentación, enrutamiento interno y control de tráfico.

# Objetivo técnico

Implementar Inter-VLAN Routing para permitir la comunicación entre redes lógicas separadas, evaluando:

Funcionamiento de subinterfaces en routers
Configuración de enlaces trunk (802.1Q)
Uso de interfaces virtuales (SVI) en switches capa 3
Comparación de rendimiento y escalabilidad entre soluciones

# Aplicación en entornos reales

Este tipo de configuración es fundamental en redes empresariales modernas, donde:

Cada departamento se aísla en una VLAN (seguridad y organización)
Se controla el tráfico entre áreas (ej. TI ↔ Ventas)
Se optimiza el uso de recursos de red


-- 

Ejemplo real:

Una empresa puede implementar:

VLAN 10 → Ventas
VLAN 20 → TI
VLAN 30 → Recursos Humanos

Y permitir comunicación controlada mediante:

Políticas de red
Firewalls internos
ACLs (en etapas más avanzadas)

--

# Tecnologías y conceptos aplicados
VLANs y segmentación de red
Trunking (802.1Q)
Subinterfaces en routers
Switch Virtual Interfaces (SVI)
Enrutamiento Inter-VLAN
Diagnóstico con ping y traceroute

## Soluciones implementadas

# 🔹 1. Router-on-a-Stick (RoS)

Se utilizó un router con subinterfaces para enrutar tráfico entre VLANs a través de un enlace trunk.

Ventajas:

Fácil de implementar
Compatible con equipos básicos
Ideal para laboratorios o redes pequeñas

Desventajas:

Cuello de botella en una sola interfaz
Menor rendimiento
Baja escalabilidad

# 🔹 2. Switch Capa 3 (SVI)

Se configuraron interfaces virtuales (SVIs) en un switch multicapa para realizar el enrutamiento directamente.

Ventajas:

Alto rendimiento (hardware switching)
Menor latencia
Alta escalabilidad
Arquitectura más limpia

Desventajas:

Requiere hardware más avanzado
Mayor costo inicial

# Comparativa técnica
Característica	Router-on-a-Stick	Switch Capa 3
Rendimiento	Medio	Alto
Escalabilidad	Limitada	Alta
Complejidad	Media	Media
Uso en empresas	Bajo	Alto
Dependencia de router	Sí	No

# Validación y pruebas

Se realizaron pruebas de conectividad para validar el correcto funcionamiento:

✅ Comunicación entre VLANs mediante ping
✅ Verificación de rutas con traceroute
✅ Revisión de interfaces activas
✅ Confirmación de tablas de enrutamiento

# Buenas prácticas aplicadas
Uso de VLAN nativa para administración
Segmentación lógica por departamentos
Configuración de descripciones en interfaces
Validación de conectividad extremo a extremo
Separación entre capa de acceso y capa de distribución

## Conclusión

El uso de switches capa 3 con SVI es la solución más eficiente y recomendada en entornos empresariales debido a su rendimiento y escalabilidad.

El método Router-on-a-Stick sigue siendo útil para:

Laboratorios
Redes pequeñas
Entornos de aprendizaje (CCNA)
