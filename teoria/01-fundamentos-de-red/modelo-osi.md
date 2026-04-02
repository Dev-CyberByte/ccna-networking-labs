# Modelo OSI

OSI significa **Open Systems Interconnection**. Es un modelo de referencia creado
por la ISO que divide la comunicación en red en 7 capas. No es un protocolo,
es una guía conceptual para entender cómo viajan los datos.

En la práctica usamos TCP/IP, pero OSI sirve para diagnosticar problemas,
entender qué hace cada protocolo y hablar un lenguaje común en redes.

---

## Las 7 capas

### Capa 7 — Aplicación
Es la capa más cercana al usuario. No es la aplicación en sí (el navegador,
el cliente de correo), sino la interfaz que permite a esas aplicaciones
acceder a la red.

**Protocolos:** HTTP, HTTPS, FTP, SMTP, DNS, Telnet, SSH  
**Pregunta clave:** ¿Qué servicio de red está usando el usuario?

---

### Capa 6 — Presentación
Se encarga de que los datos tengan un formato que la aplicación destino
entienda. Aquí se maneja la codificación, compresión y cifrado.

**Ejemplos:** SSL/TLS (cifrado), JPEG, MP4, ASCII, UTF-8  
**Pregunta clave:** ¿En qué formato están los datos?

---

### Capa 5 — Sesión
Establece, mantiene y cierra sesiones de comunicación entre dos dispositivos.
Controla el diálogo: quién habla, cuándo y por cuánto tiempo.

**Ejemplos:** NetBIOS, RPC, SQL sessions  
**Pregunta clave:** ¿Cómo se organiza y sincroniza la conversación?

---

### Capa 4 — Transporte
Garantiza la entrega de datos de extremo a extremo. Segmenta los datos,
controla el flujo y maneja errores. Aquí viven TCP y UDP.

**Protocolos:** TCP, UDP  
**Unidad de datos:** Segmento (TCP) / Datagrama (UDP)  
**Pregunta clave:** ¿La entrega es confiable o rápida?

| TCP | UDP |
|-----|-----|
| Confiable | No confiable |
| Orientado a conexión | Sin conexión |
| Control de flujo | Sin control de flujo |
| HTTP, SSH, FTP | DNS, DHCP, VoIP, streaming |

---

### Capa 3 — Red
Se encarga del direccionamiento lógico y el enrutamiento. Decide el mejor
camino para que el paquete llegue de origen a destino, aunque pasen por
múltiples redes distintas.

**Protocolos:** IP (IPv4, IPv6), ICMP, OSPF, EIGRP, BGP  
**Unidad de datos:** Paquete  
**Dispositivo:** Router  
**Pregunta clave:** ¿Cómo llega el paquete de una red a otra?

---

### Capa 2 — Enlace de Datos
Maneja la comunicación entre dispositivos dentro de la misma red local.
Usa direcciones MAC para identificar origen y destino en el segmento.
También detecta errores en la transmisión.

**Protocolos:** Ethernet, Wi-Fi (802.11), PPP, HDLC  
**Unidad de datos:** Frame (trama)  
**Dispositivo:** Switch  
**Pregunta clave:** ¿Cómo se mueven los datos dentro de la red local?

Subcapas de la capa 2:
- **LLC** (Logical Link Control) — control de enlace lógico
- **MAC** (Media Access Control) — control de acceso al medio

---

### Capa 1 — Física
Transmite bits crudos a través del medio físico. Define voltajes,
frecuencias, tipos de cable, conectores y velocidades de transmisión.

**Ejemplos:** Cable UTP, fibra óptica, señal Wi-Fi, conectores RJ45  
**Unidad de datos:** Bit  
**Dispositivos:** Hub, repetidor, cable, NIC  
**Pregunta clave:** ¿Cómo viajan los bits por el medio?

---

## Resumen visual
```
┌─────────────────────────────────────────┐
│  7 - Aplicación   │ HTTP, FTP, DNS, SSH │
├─────────────────────────────────────────┤
│  6 - Presentación │ SSL, JPEG, ASCII    │
├─────────────────────────────────────────┤
│  5 - Sesión       │ NetBIOS, RPC        │
├─────────────────────────────────────────┤
│  4 - Transporte   │ TCP, UDP            │
├─────────────────────────────────────────┤
│  3 - Red          │ IP, ICMP, OSPF      │
├─────────────────────────────────────────┤
│  2 - Enlace       │ Ethernet, Wi-Fi     │
├─────────────────────────────────────────┤
│  1 - Física       │ Cables, señales     │
└─────────────────────────────────────────┘
```

---

## Encapsulación y desencapsulación

Cuando envías datos, cada capa agrega su propio encabezado al bajar.
Cuando los recibes, cada capa quita su encabezado al subir.
```
Aplicación   →  Datos
Transporte   →  [TCP header]  + Datos          = Segmento
Red          →  [IP header]   + Segmento       = Paquete
Enlace       →  [ETH header]  + Paquete + FCS  = Frame
Física       →  101010110100... (bits)
```

A esto se le llama **PDU** (Protocol Data Unit). Cada capa tiene su PDU:

| Capa | PDU |
|------|-----|
| Transporte | Segmento |
| Red | Paquete |
| Enlace | Frame |
| Física | Bit |

---

## Truco para recordar las capas

De arriba hacia abajo (7 → 1):
> **A**ll **P**eople **S**eem **T**o **N**eed **D**ata **P**rocessing

De abajo hacia arriba (1 → 7):
> **P**lease **D**o **N**ot **T**hrow **S**ausage **P**izza **A**way

---

## En el examen CCNA

Preguntas típicas sobre OSI:

- *¿En qué capa opera un switch?* → Capa 2
- *¿En qué capa opera un router?* → Capa 3
- *¿Qué capa segmenta los datos?* → Capa 4 (Transporte)
- *¿En qué capa está IP?* → Capa 3
- *¿En qué capa está Ethernet?* → Capa 2
- *¿Qué PDU usa la capa de Red?* → Paquete
- *Un usuario no puede navegar pero sí hace ping. ¿Qué capa revisar?* → Capa 7
