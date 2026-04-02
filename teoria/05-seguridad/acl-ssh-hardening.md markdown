# Seguridad — ACLs, SSH y Hardening

La seguridad en redes no es opcional. Este módulo cubre las herramientas
básicas que Cisco IOS ofrece para controlar el tráfico, proteger el acceso
remoto y endurecer la configuración de los dispositivos.

---

## ACLs — Access Control Lists

### ¿Qué es una ACL?
Es una lista de reglas que el router evalúa secuencialmente para
permitir o denegar tráfico. Se aplica en una interfaz y en una dirección
(entrante o saliente).

### Cómo funciona una ACL
```
Paquete llega
      │
      ▼
┌─────────────┐    coincide    ┌─────────────┐
│   Regla 1   │ ─────────────► │   permit    │ ──► Paquete pasa
└─────────────┘                └─────────────┘
      │ no coincide
      ▼
┌─────────────┐    coincide    ┌─────────────┐
│   Regla 2   │ ─────────────► │    deny     │ ──► Paquete descartado
└─────────────┘                └─────────────┘
      │ no coincide
      ▼
┌─────────────────────────────────────────────┐
│   deny any (implícito al final de toda ACL) │ ──► Paquete descartado
└─────────────────────────────────────────────┘
```

> Al final de toda ACL hay un **deny any implícito**.
> Si el tráfico no coincide con ninguna regla, se descarta.
> Por eso siempre debes incluir un permit al final si quieres
> dejar pasar el resto del tráfico.

### Reglas de oro de las ACLs

1. Se evalúan de arriba hacia abajo, en orden
2. La primera coincidencia gana, el resto no se evalúa
3. Toda ACL tiene un deny any implícito al final
4. Una ACL vacía permite todo (no tiene deny implícito hasta que agregas la primera regla)
5. Aplicar la ACL en la interfaz correcta y dirección correcta

### Dónde aplicar una ACL
```
         in          interfaz         out
Tráfico ──►│ ACL aplicada aquí │──► tráfico sale
entrante    │     G0/0          │    saliente
```

- **inbound (in):** el router evalúa la ACL antes de enrutar
- **outbound (out):** el router evalúa la ACL después de enrutar

> Regla general de ubicación:
> - ACL estándar → cerca del destino
> - ACL extendida → cerca del origen

---

## ACL Estándar

### Características
- Filtra solo por **IP de origen**
- Numeradas del 1 al 99 y del 1300 al 1999
- También pueden ser nombradas
- Por su limitación se colocan cerca del destino

### Sintaxis numerada
```
Router(config)# access-list [número] [permit|deny] [origen] [wildcard]
```

### Ejemplo — Bloquear una red específica

Bloquear que la red 192.168.2.0/24 acceda al servidor en G0/1:
```
Router(config)# access-list 10 deny 192.168.2.0 0.0.0.255
Router(config)# access-list 10 permit any

Router(config)# interface GigabitEthernet0/1
Router(config-if)# ip access-group 10 out
Router(config-if)# exit
```

### Ejemplo — Permitir solo un host
```
Router(config)# access-list 10 permit host 192.168.1.10
Router(config)# access-list 10 deny any
```

> La palabra `host` equivale a wildcard 0.0.0.0
> La palabra `any` equivale a 0.0.0.0 255.255.255.255

### ACL estándar nombrada
```
Router(config)# ip access-list standard BLOQUEAR-RRHH
Router(config-std-nacl)# deny 192.168.3.0 0.0.0.255
Router(config-std-nacl)# permit any
Router(config-std-nacl)# exit

Router(config)# interface GigabitEthernet0/1
Router(config-if)# ip access-group BLOQUEAR-RRHH out
Router(config-if)# exit
```

---

## ACL Extendida

### Características
- Filtra por **IP origen, IP destino, protocolo y puerto**
- Numeradas del 100 al 199 y del 2000 al 2699
- También pueden ser nombradas
- Se colocan cerca del origen para descartar tráfico lo antes posible

### Sintaxis
```
Router(config)# access-list [número] [permit|deny] [protocolo]
                [origen] [wildcard] [destino] [wildcard] [operador puerto]
```

### Operadores de puerto

| Operador | Significado |
|----------|-------------|
| eq | igual a |
| ne | no igual a |
| lt | menor que |
| gt | mayor que |
| range | rango de puertos |

### Ejemplo — Bloquear HTTP pero permitir HTTPS
```
Router(config)# access-list 100 deny tcp 192.168.1.0 0.0.0.255 any eq 80
Router(config)# access-list 100 permit ip any any

Router(config)# interface GigabitEthernet0/0
Router(config-if)# ip access-group 100 in
Router(config-if)# exit
```

### Ejemplo — Permitir solo SSH hacia un servidor
```
Router(config)# access-list 110 permit tcp 192.168.1.0 0.0.0.255 host 10.0.0.10 eq 22
Router(config)# access-list 110 deny ip any any

Router(config)# interface GigabitEthernet0/0
Router(config-if)# ip access-group 110 in
Router(config-if)# exit
```

### Ejemplo — Bloquear ping (ICMP) desde una red
```
Router(config)# access-list 120 deny icmp 192.168.2.0 0.0.0.255 any
Router(config)# access-list 120 permit ip any any
```

### ACL extendida nombrada
```
Router(config)# ip access-list extended POLITICA-VENTAS
Router(config-ext-nacl)# permit tcp 192.168.10.0 0.0.0.255 any eq 80
Router(config-ext-nacl)# permit tcp 192.168.10.0 0.0.0.255 any eq 443
Router(config-ext-nacl)# deny ip 192.168.10.0 0.0.0.255 192.168.20.0 0.0.0.255
Router(config-ext-nacl)# permit ip any any
Router(config-ext-nacl)# exit

Router(config)# interface GigabitEthernet0/0
Router(config-if)# ip access-group POLITICA-VENTAS in
Router(config-if)# exit
```

### Verificación ACLs
```
Router# show access-lists
Router# show access-lists 100
Router# show ip interface GigabitEthernet0/0
```

### Salida de show access-lists
```
Extended IP access list 100
    10 deny tcp 192.168.1.0 0.0.0.255 any eq www (125 matches)
    20 permit ip any any (1840 matches)
```

> Los **matches** muestran cuántas veces coincidió cada regla.
> Son muy útiles para verificar que la ACL funciona correctamente.

---

## SSH — Secure Shell

### ¿Por qué SSH y no Telnet?
Telnet envía todo en texto plano, incluyendo contraseñas.
SSH cifra toda la comunicación. En redes reales Telnet
está completamente prohibido.

| Característica | Telnet | SSH |
|----------------|--------|-----|
| Puerto | 23 | 22 |
| Cifrado | No | Sí |
| Autenticación | Básica | Robusta |
| Uso en producción | Nunca | Siempre |

### Requisitos para habilitar SSH en Cisco IOS

1. Nombre de host configurado
2. Nombre de dominio configurado
3. Par de llaves RSA generado
4. Usuario local creado
5. VTY configuradas para SSH

### Configuración completa de SSH
```
Router(config)# hostname R1
R1(config)# ip domain-name empresa.local

R1(config)# crypto key generate rsa modulus 2048

R1(config)# username admin privilege 15 secret AdminPass123

R1(config)# ip ssh version 2
R1(config)# ip ssh time-out 60
R1(config)# ip ssh authentication-retries 3

R1(config)# line vty 0 4
R1(config-line)# transport input ssh
R1(config-line)# login local
R1(config-line)# exec-timeout 10 0
R1(config-line)# exit
```

> `modulus 2048` genera una llave RSA de 2048 bits. Mínimo recomendado.
> `privilege 15` da acceso completo al usuario.
> `transport input ssh` bloquea Telnet en las líneas VTY.

### Verificación SSH
```
Router# show ip ssh
Router# show ssh
```

### Conectarse por SSH desde otro dispositivo
```
ssh -l admin 192.168.1.1          ← desde Linux/Mac
ssh admin@192.168.1.1             ← desde Linux/Mac
```

---

## Hardening — Endurecimiento del dispositivo

Hardening es el proceso de asegurar un dispositivo eliminando
vulnerabilidades y aplicando buenas prácticas de seguridad.

### Contraseñas seguras
```
Router(config)# enable secret ContraseñaSegura123

Router(config)# service password-encryption

Router(config)# security passwords min-length 10

Router(config)# no enable password
```

> `enable secret` usa MD5, siempre preferirlo sobre `enable password`.
> `service password-encryption` cifra contraseñas en texto plano del config.
> Aunque es cifrado débil (tipo 7), es mejor que nada.

### Banner de advertencia legal
```
Router(config)# banner motd ^
=========================================
   ACCESO SOLO PARA PERSONAL AUTORIZADO
   Toda actividad es monitoreada y registrada
   El acceso no autorizado es un delito
=========================================
^
```

> El banner MOTD aparece antes del login.
> Nunca uses palabras como "welcome" en el banner,
> podría usarse legalmente contra la organización.

### Seguridad en líneas de consola y VTY
```
Router(config)# line console 0
Router(config-line)# password ConsolaPass123
Router(config-line)# login local
Router(config-line)# exec-timeout 5 0
Router(config-line)# logging synchronous
Router(config-line)# exit

Router(config)# line vty 0 4
Router(config-line)# transport input ssh
Router(config-line)# login local
Router(config-line)# exec-timeout 10 0
Router(config-line)# exit
```

> `exec-timeout 5 0` cierra la sesión después de 5 minutos de inactividad.
> `logging synchronous` evita que los mensajes del sistema interrumpan
> lo que estás escribiendo en la consola.

### Deshabilitar servicios innecesarios
```
Router(config)# no ip http server
Router(config)# no ip http secure-server
Router(config)# no cdp run
Router(config)# no ip proxy-arp
Router(config)# no service finger
Router(config)# no service tcp-small-servers
Router(config)# no service udp-small-servers
```

> CDP (Cisco Discovery Protocol) revela información de la topología.
> Deshabilitarlo en interfaces que dan hacia redes externas o usuarios.

### Deshabilitar CDP solo en interfaces externas
```
Router(config)# interface GigabitEthernet0/1
Router(config-if)# no cdp enable
Router(config-if)# exit
```

### Proteger el acceso físico a ROMMON
```
Router(config)# no service password-recovery
```

> Cuidado: si pierdes la contraseña con esto activado,
> el router se resetea a factory default. Úsalo solo cuando
> el acceso físico está completamente controlado.

### Guardar la configuración
```
Router# copy running-config startup-config
Router# write memory
```

### Configuración de Hardening completa (resumen)
```
Router(config)# hostname R1
R1(config)# enable secret ContraseñaSegura123
R1(config)# service password-encryption
R1(config)# security passwords min-length 10
R1(config)# no enable password

R1(config)# username admin privilege 15 secret AdminPass123

R1(config)# banner motd ^ACCESO SOLO AUTORIZADO^

R1(config)# line console 0
R1(config-line)# login local
R1(config-line)# exec-timeout 5 0
R1(config-line)# logging synchronous
R1(config-line)# exit

R1(config)# line vty 0 4
R1(config-line)# transport input ssh
R1(config-line)# login local
R1(config-line)# exec-timeout 10 0
R1(config-line)# exit

R1(config)# ip domain-name empresa.local
R1(config)# crypto key generate rsa modulus 2048
R1(config)# ip ssh version 2

R1(config)# no ip http server
R1(config)# no ip http secure-server
R1(config)# no cdp run

R1# copy running-config startup-config
```

---

## Port Security (repaso con enfoque de seguridad)

Ya visto en el módulo 02, pero aquí el enfoque es la política de seguridad:
```
Switch(config)# interface FastEthernet0/1
Switch(config-if)# switchport mode access
Switch(config-if)# switchport port-security
Switch(config-if)# switchport port-security maximum 1
Switch(config-if)# switchport port-security mac-address sticky
Switch(config-if)# switchport port-security violation shutdown
Switch(config-if)# exit
```

Recuperar un puerto en estado err-disabled:
```
Switch(config)# interface FastEthernet0/1
Switch(config-if)# shutdown
Switch(config-if)# no shutdown
Switch(config-if)# exit
```

---

## En el examen CCNA

Preguntas típicas de este módulo:

- *¿Qué hay al final de toda ACL?* → Un deny any implícito
- *¿En qué dirección se evalúa una ACL?* → De arriba hacia abajo, primera coincidencia gana
- *¿Dónde se coloca una ACL estándar?* → Cerca del destino
- *¿Dónde se coloca una ACL extendida?* → Cerca del origen
- *¿Qué filtra una ACL estándar?* → Solo IP de origen
- *¿Qué filtra una ACL extendida?* → IP origen, IP destino, protocolo y puerto
- *¿Qué puerto usa SSH?* → 22
- *¿Qué comando bloquea Telnet en las VTY?* → transport input ssh
- *¿Qué diferencia hay entre enable password y enable secret?* → Secret usa MD5, es más seguro
- *¿Qué hace service password-encryption?* → Cifra contraseñas en texto plano del running-config
- *¿Qué hace exec-timeout?* → Cierra sesiones inactivas después del tiempo configurado
