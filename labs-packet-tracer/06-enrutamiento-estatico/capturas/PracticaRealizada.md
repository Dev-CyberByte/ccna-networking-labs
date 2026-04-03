# Práctica Realizada — Enrutamiento Estático y Ruta Flotante
# Descripción general

En esta práctica se diseñó e implementó una red interconectada entre múltiples sucursales utilizando Enrutamiento Estático, incluyendo rutas estáticas, ruta por defecto y rutas estáticas flotantes como mecanismo de respaldo.

La topología simula una empresa con tres sedes (Norte, Central y Sur), conectadas a través de enlaces WAN punto a punto, replicando un escenario común en infraestructuras corporativas distribuidas.

# Escenario empresarial simulado

La red representa una organización con: 

# Sucursal Norte (usuarios finales)
# Sucursal Central (nodo principal de interconexión)
# Sucursal Sur (usuarios finales)

Donde:

El router central (R2) actúa como punto de agregación
Se establecen enlaces WAN redundantes para garantizar continuidad operativa
Cada sucursal mantiene su propia LAN segmentada

# Objetivos técnicos
Implementar conectividad entre múltiples redes mediante rutas estáticas
Configurar rutas por defecto para salida hacia redes externas
Diseñar e implementar una ruta de respaldo (flotante)
Verificar tablas de enrutamiento en routers
Validar conectividad extremo a extremo mediante herramientas de diagnóstico

Arquitectura de red implementada
🔹 Segmentación por sucursal

Cada sitio cuenta con su propia red local:

LAN Norte → 192.168.1.0/24
LAN Central → 192.168.2.0/24
LAN Sur → 192.168.3.0/24

Esto permite:

Aislamiento de tráfico
Reducción de broadcast
Aplicación de políticas por área
🔹 Enlaces WAN

Se implementaron enlaces punto a punto utilizando subredes /30:

R1 ↔ R2 → 10.0.0.0/30
R2 ↔ R3 → 10.0.1.0/30
R1 ↔ R3 → enlace de respaldo (ruta flotante)

*Este diseño replica enlaces dedicados en entornos reales (MPLS, enlaces arrendados o VPN site-to-site).*

Implementación técnica
Rutas estáticas

Se configuraron rutas manuales en cada router para alcanzar redes remotas, utilizando el concepto de next-hop.

Ejemplo:

ip route 192.168.3.0 255.255.255.0 10.0.0.2

En entornos reales, esto se usa cuando:

La red es pequeña o estable
Se requiere control total del tráfico
Se busca simplicidad operativa
🔹 Ruta por defecto

Se configuró una ruta por defecto para dirigir tráfico desconocido hacia un router específico.

ip route 0.0.0.0 0.0.0.0 10.0.0.2

En empresas, esto se utiliza para:

Salida a internet
Enviar tráfico hacia un firewall o gateway central
🔹 Ruta estática flotante

Se implementó redundancia mediante una ruta flotante, configurando una segunda ruta con mayor distancia administrativa.

Ejemplo:

ip route 192.168.3.0 255.255.255.0 10.0.0.2 1
ip route 192.168.3.0 255.255.255.0 10.0.2.2 5
AD 1 → ruta principal
AD 5 → ruta de respaldo

Esta ruta solo se activa si la principal falla.

Alta disponibilidad y resiliencia

Se simuló la caída del enlace principal (R1–R2), lo que provocó:

Recalculo automático de la tabla de enrutamiento
Activación de la ruta flotante
Restauración de la conectividad sin intervención manual

Este comportamiento es crítico en empresas para:

Garantizar continuidad del negocio
Minimizar tiempos de inactividad
Mantener servicios activos (VoIP, ERP, aplicaciones)

# Verificación y troubleshooting

Se utilizaron comandos clave como:

show ip route
show ip interface brief
ping
traceroute

Permitiendo:

Validar rutas activas
Identificar fallos de conectividad
Analizar el camino del tráfico

Aplicación en entornos reales
🔹 Casos de uso
Empresas con múltiples sucursales
Redes con enlaces redundantes
Infraestructuras sin protocolos dinámicos
Escenarios con requerimientos de control manual
🔹 Beneficios en producción
Simplicidad de configuración
Control total del flujo de tráfico
Bajo consumo de recursos
Alta previsibilidad

# ⚠️ Limitaciones
No escala bien en redes grandes
Configuración manual propensa a errores
No se adapta automáticamente a cambios complejos
Mayor carga operativa en administración

Por ello, en empresas grandes se usan protocolos como:

OSPF
EIGRP
BGP

# Limitaciones del entorno de simulación

El laboratorio fue realizado en Cisco Packet Tracer, lo cual implica:

Simulación simplificada del comportamiento real
Limitaciones en comandos y debugging
No refleja completamente latencias ni fallos físicos reales

# Conclusión

Esta práctica permite comprender cómo implementar conectividad entre múltiples redes de forma controlada, así como diseñar mecanismos básicos de alta disponibilidad mediante rutas flotantes.

El dominio de estos conceptos es fundamental para:

Diseño de redes empresariales
Implementación de redundancia
Resolución de problemas en capa 3

Además, establece las bases para la transición hacia protocolos de enrutamiento dinámico utilizados en entornos productivos.
