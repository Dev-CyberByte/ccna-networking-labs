# Labs — Cisco Packet Tracer

Prácticas organizadas por tema CCNA. Cada lab tiene instrucciones
paso a paso, topología, comandos y verificación. Los archivos .pkt
se abren directamente en Cisco Packet Tracer 8.x o superior.

---

## Requisitos

- Cisco Packet Tracer 8.x (descarga gratuita con cuenta Cisco NetAcad)
- Leer el archivo de teoría correspondiente antes de cada lab

---

## Estructura de cada lab

Cada carpeta de lab contiene:
```
lab-nombre/
├── README.md        ← Instrucciones completas del lab
├── lab-nombre.pkt   ← Archivo de Packet Tracer
└── capturas/        ← Screenshots de verificación
    ├── topologia.png
    └── verificacion.png
```

---

## Lista de labs

| # | Lab | Tema CCNA | Dificultad |
|---|-----|-----------|------------|
| 01 | Configuración básica de dispositivos | Fundamentos | ⭐ |
| 02 | Subnetting y VLSM | Fundamentos | ⭐⭐ |
| 03 | VLANs y Trunking | Acceso de red | ⭐⭐ |
| 04 | Inter-VLAN Routing | Acceso de red | ⭐⭐ |
| 05 | Spanning Tree Protocol | Acceso de red | ⭐⭐ |
| 06 | Enrutamiento estático | Conectividad IP | ⭐⭐ |
| 07 | OSPF Single Area | Conectividad IP | ⭐⭐⭐ |
| 08 | DHCP y DNS | Servicios IP | ⭐⭐ |
| 09 | NAT y PAT | Servicios IP | ⭐⭐⭐ |
| 10 | ACLs estándar y extendidas | Seguridad | ⭐⭐⭐ |
| 11 | SSH y Hardening | Seguridad | ⭐⭐ |

---

## Cómo usar estos labs

1. Lee el archivo de teoría del tema antes de empezar
2. Abre el `.pkt` en Packet Tracer
3. Sigue las instrucciones del README del lab paso a paso
4. Verifica con los comandos indicados
5. Toma capturas y guárdalas en la carpeta `capturas/`
6. Si algo falla revisa la sección de troubleshooting del lab

---

## Teoría relacionada

Cada lab referencia su archivo de teoría:

| Lab | Teoría |
|-----|--------|
| 01, 02 | [teoria/01-fundamentos-de-red](../teoria/01-fundamentos-de-red/) |
| 03, 04, 05 | [teoria/02-acceso-de-red](../teoria/02-acceso-de-red/) |
| 06, 07 | [teoria/03-conectividad-ip](../teoria/03-conectividad-ip/) |
| 08, 09 | [teoria/04-servicios-ip](../teoria/04-servicios-ip/) |
| 10, 11 | [teoria/05-seguridad](../teoria/05-seguridad/) |
