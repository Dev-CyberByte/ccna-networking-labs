# Práctica Realizada — Spanning Tree Protocol (STP)
# Descripción general

En esta práctica se implementó y analizó el funcionamiento del Spanning Tree Protocol en una topología con enlaces redundantes entre switches, simulando un entorno de prueba donde la alta disponibilidad es crítica.

Se trabajó con múltiples VLANs (10, 20, 30) transportadas a través de enlaces troncales, observando cómo STP previene loops de capa 2 y mantiene la estabilidad de la red.

# Objetivos técnicos
Comprender la prevención de loops en redes LAN
Identificar el Root Bridge en diferentes VLANs
Analizar roles de puertos:
Root Port
Designated Port
Alternate Port
Manipular la elección del Root Bridge mediante prioridad
Implementar Rapid PVST+ para mejorar tiempos de convergencia
Aplicar mecanismos de seguridad como:
PortFast
BPDU Guard
# Conceptos clave aplicados
# Redundancia controlada

En entornos empresariales es común implementar enlaces redundantes para evitar puntos únicos de falla. Sin embargo, esto genera loops de capa 2.

STP soluciona este problema:

Bloqueando enlaces redundantes
Manteniendo rutas alternativas disponibles
Activando dichas rutas automáticamente ante fallos
# Elección del Root Bridge

El switch raíz determina la topología lógica de la red.

En la práctica:

Se manipuló la prioridad para definir switches raíz por VLAN
Se simuló un diseño similar a redes empresariales donde:
Un switch core/distribución actúa como Root Bridge
# Convergencia rápida

Se migró de STP tradicional a Rapid PVST+, logrando:

Reducción del tiempo de convergencia (~30s → <6s)
Recuperación más rápida ante fallos
Mayor disponibilidad de servicios
# Seguridad en capa 2

Se implementaron buenas prácticas reales:

PortFast
Reduce tiempo de conexión para dispositivos finales
BPDU Guard
Previene ataques o errores por conexión de switches no autorizados

# Esto es crítico en empresas para evitar:

Loops accidentales
Ataques de tipo STP manipulation
# Aplicación en entornos reales

Esta práctica refleja escenarios reales como:

🔹 Redes corporativas

Empresas utilizan STP en:

Acceso (switches de usuarios)
Distribución (switches intermedios)
Core (backbone de red)
🔹 Alta disponibilidad

Gracias a STP:

Si un enlace falla → otro se activa automáticamente
Se evita interrupción del servicio

Ejemplo real:

Un cable entre switches se desconecta → la red sigue operando sin intervención manual

🔹 Segmentación por VLANs

Aunque en esta práctica no se implementó routing:

Las VLANs representan departamentos reales:
Ventas
TI
RRHH

# En producción, esto:

Mejora seguridad
Reduce dominios de broadcast
Permite aplicar políticas de red
🔹 Diseño jerárquico de red

La manipulación del Root Bridge simula decisiones reales de diseño:

Ubicar el Root en switches más potentes
Optimizar rutas de tráfico
Reducir latencia
# ⚠️ Limitaciones del laboratorio en Packet Tracer

El uso de Cisco Packet Tracer implica ciertas restricciones:

No soporta comandos avanzados como:
show spanning-tree vlan X detail
Simulación limitada de temporizadores y eventos reales
Comportamiento simplificado de STP
No refleja completamente el procesamiento real de hardware

# Resultados obtenidos
Identificación correcta del Root Bridge por VLAN
Verificación de puertos en estado Forwarding y Blocking
Implementación exitosa de Rapid PVST+
Recuperación de conectividad tras fallo de enlace
Aplicación de medidas de seguridad en capa 2

# Conclusión

Esta práctica permite comprender uno de los mecanismos más importantes en redes LAN empresariales: la prevención de loops.

El dominio de STP es fundamental para:

Diseñar redes resilientes
Garantizar alta disponibilidad
Prevenir fallos críticos en capa 2

Además, refuerza habilidades clave para certificaciones como CCNA y escenarios reales de implementación en infraestructura de red.
