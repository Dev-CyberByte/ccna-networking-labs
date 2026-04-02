# Subnetting

Subnetear es dividir una red grande en redes más pequeñas llamadas subredes.
Esto permite usar el espacio de direcciones IP de forma eficiente, segmentar
el tráfico y mejorar la seguridad.

---

## Conceptos base

### Dirección IP
Una dirección IPv4 tiene 32 bits divididos en 4 octetos:
```
192      .168      .1        .10
11000000 .10101000 .00000001 .00001010
```

Cada octeto va de 0 a 255.

---

### Máscara de subred
Define qué parte de la IP es la red y qué parte es el host.
```
IP:      192.168.1.10   →  11000000.10101000.00000001.00001010
Máscara: 255.255.255.0  →  11111111.11111111.11111111.00000000
                                                       ────────
                           bits de red (1s)            bits de host (0s)
```

Los bits en **1** identifican la red.
Los bits en **0** identifican el host.

---

### Notación CIDR
En lugar de escribir la máscara completa se usa la notación `/n` donde
`n` es la cantidad de bits en 1 de la máscara.
```
255.255.255.0   =  /24  (24 bits en 1)
255.255.0.0     =  /16  (16 bits en 1)
255.0.0.0       =  /8   (8 bits en 1)
255.255.255.128 =  /25  (25 bits en 1)
```

---

## Clases de direcciones IP (referencia histórica)

| Clase | Rango del primer octeto | Máscara default | Uso |
|-------|------------------------|-----------------|-----|
| A | 1 - 126 | /8 | Redes muy grandes |
| B | 128 - 191 | /16 | Redes medianas |
| C | 192 - 223 | /24 | Redes pequeñas |
| D | 224 - 239 | — | Multicast |
| E | 240 - 255 | — | Experimental |

> 127.x.x.x está reservado para loopback (127.0.0.1 = tu propio equipo).

---

## Direcciones privadas (RFC 1918)

Estas direcciones no se enrutan en internet. Se usan en redes internas.

| Clase | Rango | CIDR |
|-------|-------|------|
| A | 10.0.0.0 — 10.255.255.255 | 10.0.0.0/8 |
| B | 172.16.0.0 — 172.31.255.255 | 172.16.0.0/12 |
| C | 192.168.0.0 — 192.168.255.255 | 192.168.0.0/16 |

---

## La tabla de subnetting (la más importante)

Memoriza esta tabla. Con ella puedes resolver cualquier ejercicio.

| CIDR | Máscara | Subredes* | Hosts útiles | Bloque |
|------|---------|-----------|--------------|--------|
| /24 | 255.255.255.0 | 1 | 254 | 256 |
| /25 | 255.255.255.128 | 2 | 126 | 128 |
| /26 | 255.255.255.192 | 4 | 62 | 64 |
| /27 | 255.255.255.224 | 8 | 30 | 32 |
| /28 | 255.255.255.240 | 16 | 14 | 16 |
| /29 | 255.255.255.248 | 32 | 6 | 8 |
| /30 | 255.255.255.252 | 64 | 2 | 4 |
| /31 | 255.255.255.254 | — | 2* | 2 |
| /32 | 255.255.255.255 | — | 1 host | 1 |

> *Subredes calculadas desde /24.
> */31 se usa en enlaces punto a punto (RFC 3021), sin dirección de red ni broadcast.

**Fórmula de hosts útiles:** 2^(bits de host) - 2

---

## Cómo calcular una subred paso a paso

### Datos de ejemplo
```
Red: 192.168.1.0/26
```

### Paso 1 — Identificar el bloque
/26 → máscara 255.255.255.192 → bloque de 64

### Paso 2 — Calcular las subredes
Comenzando en 0, sumar el bloque cada vez:
```
Subred 1: 192.168.1.0    → broadcast: 192.168.1.63
Subred 2: 192.168.1.64   → broadcast: 192.168.1.127
Subred 3: 192.168.1.128  → broadcast: 192.168.1.191
Subred 4: 192.168.1.192  → broadcast: 192.168.1.255
```

### Paso 3 — Definir los valores de cada subred

Para la **Subred 1 (192.168.1.0/26)**:
```
Dirección de red:   192.168.1.0
Primer host útil:   192.168.1.1
Último host útil:   192.168.1.62
Broadcast:          192.168.1.63
Máscara:            255.255.255.192
Hosts útiles:       62
```

> La dirección de red y el broadcast nunca se asignan a dispositivos.

---

## VLSM — Variable Length Subnet Mask

VLSM permite usar máscaras de diferente tamaño dentro de la misma red.
Así no desperdicias IPs: cada subred recibe exactamente el espacio que necesita.

### Problema de ejemplo
```
Red disponible: 192.168.10.0/24

Requerimientos:
- Área Ventas:    50 hosts
- Área TI:        25 hosts
- Área RRHH:      10 hosts
- Enlace WAN 1:    2 hosts
- Enlace WAN 2:    2 hosts
```

### Regla de VLSM
**Siempre empieza por la subred más grande.**

### Solución paso a paso

#### 1. Ventas — 50 hosts
- Necesito: 50 hosts → busco 2^n - 2 ≥ 50 → 2^6 = 64 → /26
- Bloque: 64
```
Red:       192.168.10.0/26
Hosts:     192.168.10.1 — 192.168.10.62
Broadcast: 192.168.10.63
```

#### 2. TI — 25 hosts
- Necesito: 25 hosts → 2^5 = 32 → /27
- Bloque: 32 → empiezo donde terminó la anterior
```
Red:       192.168.10.64/27
Hosts:     192.168.10.65 — 192.168.10.94
Broadcast: 192.168.10.95
```

#### 3. RRHH — 10 hosts
- Necesito: 10 hosts → 2^4 = 16 → /28
- Bloque: 16
```
Red:       192.168.10.96/28
Hosts:     192.168.10.97 — 192.168.10.110
Broadcast: 192.168.10.111
```

#### 4. Enlace WAN 1 — 2 hosts
- Necesito: 2 hosts → 2^2 = 4 → /30
- Bloque: 4
```
Red:       192.168.10.112/30
Hosts:     192.168.10.113 — 192.168.10.114
Broadcast: 192.168.10.115
```

#### 5. Enlace WAN 2 — 2 hosts
- Necesito: 2 hosts → /30
- Bloque: 4
```
Red:       192.168.10.116/30
Hosts:     192.168.10.117 — 192.168.10.118
Broadcast: 192.168.10.119
```

### Resumen VLSM

| Área | Red | Máscara | Hosts útiles |
|------|-----|---------|--------------|
| Ventas | 192.168.10.0/26 | 255.255.255.192 | 62 |
| TI | 192.168.10.64/27 | 255.255.255.224 | 30 |
| RRHH | 192.168.10.96/28 | 255.255.255.240 | 14 |
| WAN 1 | 192.168.10.112/30 | 255.255.255.252 | 2 |
| WAN 2 | 192.168.10.116/30 | 255.255.255.252 | 2 |

Espacio usado: .0 → .119 (120 direcciones de 256 disponibles)
Espacio libre: 192.168.10.120 — 192.168.10.255

---

## Cómo saber si dos IPs están en la misma red

Aplica la máscara (AND lógico) a ambas IPs. Si el resultado es igual,
están en la misma red.
```
IP 1:    192.168.1.10  →  11000000.10101000.00000001.00001010
IP 2:    192.168.1.200 →  11000000.10101000.00000001.11001000
Máscara: 255.255.255.0 →  11111111.11111111.11111111.00000000

IP 1 AND Máscara = 192.168.1.0  ✓
IP 2 AND Máscara = 192.168.1.0  ✓

→ Misma red, se comunican directamente sin router.
```
```
IP 1:    192.168.1.10  →  ...00000001.00001010
IP 2:    192.168.2.50  →  ...00000010.00110010
Máscara: 255.255.255.0

IP 1 AND Máscara = 192.168.1.0
IP 2 AND Máscara = 192.168.2.0

→ Redes distintas, necesitan un router para comunicarse.
```

---

## Truco rápido para el examen

Para encontrar la red a la que pertenece una IP:

1. Identifica el octeto interesante (donde la máscara no es 0 ni 255)
2. Calcula el bloque: 256 - valor del octeto en la máscara
3. El múltiplo del bloque más cercano hacia abajo es la red
4. Suma el bloque - 1 para el broadcast

**Ejemplo rápido:**
```
IP:      172.16.45.200/20
Máscara: 255.255.240.0

Octeto interesante: tercer octeto (240)
Bloque: 256 - 240 = 16

Múltiplos de 16: 0, 16, 32, 48...
45 cae entre 32 y 48 → la red es 172.16.32.0

Red:       172.16.32.0
Broadcast: 172.16.47.255
Hosts:     172.16.32.1 — 172.16.47.254
```

---

## En el examen CCNA

Preguntas típicas sobre subnetting:

- *¿Cuántos hosts útiles tiene una /27?* → 30
- *¿Qué máscara necesito para 50 hosts?* → /26
- *¿A qué red pertenece 10.0.5.130/22?* → 10.0.4.0
- *¿Cuál es el broadcast de 192.168.1.64/26?* → 192.168.1.127
- *Tienes 192.168.5.0/24 y necesitas 5 subredes con 20 hosts cada una. ¿Qué máscara usas?* → /27
