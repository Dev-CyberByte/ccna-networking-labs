# Lab GNS3 01 — Configuración Básica de Dispositivos

## Información del lab

| Campo | Detalle |
|-------|---------|
| Tema CCNA | Fundamentos de red — Dominio 1.x |
| Dificultad | ⭐ Básico |
| Duración estimada | 40 minutos |
| Archivo | lab-gns3-01-config-basica.gns3 |
| Equivalente PKT | [Lab PKT 01](../../labs-packet-tracer/01-config-basica/README.md) |
| Teoría relacionada | [teoria/01-fundamentos-de-red](../../teoria/01-fundamentos-de-red/) |

---

## Objetivo

Realizar la misma configuración básica del Lab PKT 01 pero en
GNS3 con IOS real. Observar las diferencias en nombres de
interfaces, comportamiento del IOS y comandos disponibles
que no existen en Packet Tracer.

---

## Diferencias con Packet Tracer

| Aspecto | Packet Tracer | Este lab GNS3 |
|---------|--------------|---------------|
| Interfaces router | GigabitEthernet0/0 | FastEthernet0/0 |
| Interfaces switch | FastEthernet0/1 | FastEthernet0/1 |
| Guardado | Automático | Manual obligatorio |
| Consola | Click en dispositivo | Terminal externo |
| Velocidad | Inmediata | Depende del hardware |
| CDP | Simulado | Real y completo |

---

## Requisitos de este lab

- Imagen IOS: Cisco 7200 (c7200-adventerprisek9-mz) para routers
- Imagen IOS: IOSvL2 o c3560 para switches
- RAM asignada por router: mínimo 256 MB
- GNS3 con al menos un router y un switch configurados

---

## Topología
            192.168.1.0/24
PC-1 ─────────── SW1 ─────────── R1 ─────────── PC-2
Fa0/1  Fa0/0    Fa0/0  Fa0/1
.10      .1            .1       .20
192.168.2.0/24

> En GNS3 las PCs se simulan con el dispositivo VPCS
> (Virtual PC Simulator) que viene incluido en GNS3.

---

## Tabla de direccionamiento

| Dispositivo | Interfaz | Dirección IP | Máscara | Gateway |
|-------------|----------|-------------|---------|---------|
| R1 | Fa0/0 | 192.168.1.1 | 255.255.255.0 | — |
| R1 | Fa0/1 | 192.168.2.1 | 255.255.255.0 | — |
| SW1 | VLAN 1 | 192.168.1.2 | 255.255.255.0 | 192.168.1.1 |
| PC-1 | eth0 | 192.168.1.10 | 255.255.255.0 | 192.168.1.1 |
| PC-2 | eth0 | 192.168.2.20 | 255.255.255.0 | 192.168.2.1 |

---

## Instrucciones

### Parte 1 — Preparar el entorno GNS3

#### Paso 1 — Crear el proyecto
1. Abre GNS3
2. File → New blank project
3. Nombre: `lab-gns3-01-config-basica`
4. Click en OK

#### Paso 2 — Agregar dispositivos
1. Arrastra un router c7200 al canvas
2. Arrastra un switch (IOSvL2 o Ethernet switch)
3. Arrastra dos VPCS (Virtual PC Simulator)
4. Renombra los dispositivos: R1, SW1, PC-1, PC-2

#### Paso 3 — Conectar los dispositivos
Usa la herramienta de cable (Add a link) y conecta:
PC-1  eth0  →  SW1  Fa0/1
PC-2  eth0  →  R1   Fa0/1
SW1   Fa0/0  →  R1   Fa0/0

#### Paso 4 — Iniciar los dispositivos
Click derecho en cada dispositivo → Start
O click en el botón Play verde de la barra superior.

> Espera hasta que todos los dispositivos muestren
> el triángulo verde antes de abrir las consolas.
> Los routers IOS tardan 1-2 minutos en iniciar.

#### Paso 5 — Abrir consolas
Click derecho en R1 → Console
Click derecho en SW1 → Console
Click derecho en PC-1 → Console
Click derecho en PC-2 → Console

---

### Parte 2 — Configurar R1

#### Paso 1 — Salir del setup inicial
Si aparece el wizard de configuración inicial:
Would you like to enter the initial configuration dialog? [yes/no]: no
Would you like to terminate autoinstall? [yes]: yes

Presiona Enter para obtener el prompt.

#### Paso 2 — Configuración básica
```cisco
Router> enable
Router# configure terminal
Router(config)# hostname R1
R1(config)# enable secret Cisco123
R1(config)# service password-encryption
R1(config)# security passwords min-length 8
R1(config)# no ip domain-lookup
```

#### Paso 3 — Banner
```cisco
R1(config)# banner motd ^
ACCESO SOLO PARA PERSONAL AUTORIZADO
TechStart S.A.
Actividad monitoreada y registrada
^
```

#### Paso 4 — Interfaz Fa0/0 (LAN izquierda)
```cisco
R1(config)# interface FastEthernet0/0
R1(config-if)# description LAN-Izquierda
R1(config-if)# ip address 192.168.1.1 255.255.255.0
R1(config-if)# no shutdown
R1(config-if)# exit
```

#### Paso 5 — Interfaz Fa0/1 (LAN derecha)
```cisco
R1(config)# interface FastEthernet0/1
R1(config-if)# description LAN-Derecha
R1(config-if)# ip address 192.168.2.1 255.255.255.0
R1(config-if)# no shutdown
R1(config-if)# exit
```

#### Paso 6 — Línea de consola
```cisco
R1(config)# line console 0
R1(config-line)# password ConsolaPass
R1(config-line)# login
R1(config-line)# exec-timeout 5 0
R1(config-line)# logging synchronous
R1(config-line)# exit
```

#### Paso 7 — Líneas VTY
```cisco
R1(config)# line vty 0 4
R1(config-line)# password VtyPass123
R1(config-line)# login
R1(config-line)# exec-timeout 10 0
R1(config-line)# exit
```

#### Paso 8 — Guardar
```cisco
R1(config)# end
R1# copy running-config startup-config
R1# write memory
```
---

### Parte 3 — Configurar SW1

#### Paso 1 — Configuración básica
```cisco
Switch> enable
Switch# configure terminal
Switch(config)# hostname SW1
SW1(config)# enable secret Cisco123
SW1(config)# service password-encryption
SW1(config)# no ip domain-lookup
```

#### Paso 2 — Interfaz de administración VLAN 1
```cisco
SW1(config)# interface vlan 1
SW1(config-if)# description Administracion
SW1(config-if)# ip address 192.168.1.2 255.255.255.0
SW1(config-if)# no shutdown
SW1(config-if)# exit
SW1(config)# ip default-gateway 192.168.1.1
```

#### Paso 3 — Banner y líneas
```cisco
SW1(config)# banner motd ^ACCESO RESTRINGIDO — TechStart S.A.^
SW1(config)# line console 0
SW1(config-line)# password ConsolaPass
SW1(config-line)# login
SW1(config-line)# exec-timeout 5 0
SW1(config-line)# logging synchronous
SW1(config-line)# exit
SW1(config)# line vty 0 15
SW1(config-line)# password VtyPass123
SW1(config-line)# login
SW1(config-line)# exec-timeout 10 0
SW1(config-line)# exit
```

#### Paso 4 — Guardar
```cisco
SW1(config)# end
SW1# copy running-config startup-config
SW1# write memory
```
---

### Parte 4 — Configurar las PCs con VPCS

VPCS usa comandos distintos a Packet Tracer.
La consola de VPCS es una terminal de texto simple.

#### Configurar PC-1
PC-1> ip 192.168.1.10 255.255.255.0 192.168.1.1
PC-1> save

#### Configurar PC-2
PC-2> ip 192.168.2.20 255.255.255.0 192.168.2.1
PC-2> save

> El comando `ip` en VPCS configura IP, máscara y gateway
> en una sola línea. El comando `save` guarda la configuración
> para que persista al reiniciar el proyecto.

#### Comandos útiles de VPCS

| Comando | Función |
|---------|---------|
| `ip [IP] [máscara] [gateway]` | Configurar red |
| `show ip` | Ver configuración actual |
| `ping [IP]` | Hacer ping |
| `trace [IP]` | Traceroute |
| `save` | Guardar configuración |
| `clear` | Limpiar pantalla |

---

## Verificación

### En R1

#### Verificar interfaces
```cisco
R1# show ip interface brief
```

Resultado esperado:
Interface         IP-Address      OK? Method Status    Protocol
FastEthernet0/0   192.168.1.1     YES manual up        up
FastEthernet0/1   192.168.2.1     YES manual up        up

#### Verificar configuración completa
```cisco
R1# show running-config
```

#### Verificar CDP en GNS3 (no disponible en PKT)
```cisco
R1# show cdp neighbors
R1# show cdp neighbors detail
```

Resultado esperado:
Device ID: SW1
IP address: 192.168.1.2
Platform: cisco ,  Capabilities: Switch
Interface: FastEthernet0/0,  Port ID (outgoing port): FastEthernet0/0

> CDP en GNS3 funciona con IOS real y muestra información
> detallada de los vecinos. Es una herramienta muy útil
> para verificar conectividad física.

#### Verificar la versión del IOS
```cisco
R1# show version
```

Esto muestra la versión real del IOS que estás usando,
el tiempo de uptime y la memoria disponible.

### En SW1

#### Verificar interfaz de administración
```cisco
SW1# show interfaces vlan 1
SW1# show ip interface brief
```

### En las PCs (VPCS)

#### Verificar configuración de red
```cisco
PC-1> show ip
```

Resultado esperado:
NAME        : PC-1[1]
IP/MASK     : 192.168.1.10/24
GATEWAY     : 192.168.1.1
DNS         :
MAC         : 00:50:79:66:68:00
LPORT       : 10010
RHOST:PORT  : 127.0.0.1:10011
MTU:        : 1500

### Pruebas de conectividad
```bash
Desde PC-1:
PC-1> ping 192.168.1.1      <- Gateway R1
PC-1> ping 192.168.1.2      <- Switch SW1
PC-1> ping 192.168.2.1      <- R1 Fa0/1
PC-1> ping 192.168.2.20     <- PC-2 extremo a extremo
PC-1> trace 192.168.2.20    <- Traceroute a PC-2

Desde PC-2:
PC-2> ping 192.168.2.1      <- Gateway R1
PC-2> ping 192.168.1.10     <- PC-1 extremo a extremo

Resultado esperado del traceroute:
PC-1> trace 192.168.2.20
trace to 192.168.2.20, 8 hops max, press Ctrl+C to stop
1   192.168.1.1   2.147 ms  1.987 ms  1.834 ms
2   192.168.2.20  3.241 ms  2.987 ms  3.112 ms
```

### Verificar CDP entre R1 y SW1
```cisco
R1# show cdp neighbors detail
```

Esta información no está disponible en Packet Tracer.
En GNS3 puedes ver la plataforma exacta, versión IOS
y todas las interfaces del vecino.

### Lista de verificación

- [ ] Proyecto GNS3 creado y guardado
- [ ] R1 y SW1 iniciados y con IOS cargado
- [ ] PCs configuradas con VPCS
- [ ] R1 Fa0/0 y Fa0/1 up/up con IPs correctas
- [ ] SW1 VLAN 1 con IP de administración
- [ ] PC-1 hace ping a PC-2 exitosamente
- [ ] PC-2 hace ping a PC-1 exitosamente
- [ ] Traceroute muestra R1 como salto intermedio
- [ ] show cdp neighbors muestra SW1 como vecino de R1
- [ ] Configuración guardada con write memory en R1 y SW1
- [ ] VPCS guardado con save en PC-1 y PC-2
- [ ] Proyecto GNS3 guardado desde File → Save Project

---

## Troubleshooting

### El router no inicia en GNS3

Verificar que la imagen IOS esté correctamente configurada
GNS3 → Edit → Preferences → IOS Routers
La imagen debe tener el checkmark verde
Verificar que haya suficiente RAM asignada
Click derecho en R1 → Configure → Memory
Mínimo 256 MB para c7200
Verificar que GNS3 VM esté corriendo si la usas
GNS3 → Edit → Preferences → GNS3 VM


### Las interfaces aparecen como down/down

Verificar que el cable esté correctamente conectado
En GNS3 los cables se ven en el canvas
Verificar que ambos extremos del cable estén iniciados
Ambos dispositivos deben estar corriendo (triángulo verde)
En routers c7200 puede ser necesario activar la interfaz
```cisco
R1(config)# interface FastEthernet0/0
R1(config-if)# no shutdown
```
Verificar el tipo de cable
Entre router y switch: cable directo (straight-through)
Entre dos routers: cable cruzado (crossover) o serial


### VPCS no guarda la configuración

Siempre usar el comando save después de configurar la IP
PC-1> save
Verificar que el proyecto GNS3 esté guardado
File → Save Project
Al reabrir el proyecto iniciar los VPCS antes de usarlos


### CDP no muestra vecinos

Verificar que CDP esté habilitado globalmente
```cisco
R1# show cdp
```
Si dice CDP is not enabled:
```cisco
R1(config)# cdp run
```
Verificar que CDP esté habilitado en la interfaz
```cisco
R1# show cdp interface FastEthernet0/0
```
Si dice CDP disabled on interface:
```cisco
R1(config-if)# cdp enable
```
Esperar 60 segundos para que CDP intercambie información
El timer de CDP es 60 segundos por defecto


---

## Comandos exclusivos de GNS3 vs Packet Tracer

Estos comandos funcionan en GNS3 con IOS real pero no
están disponibles o son limitados en Packet Tracer:

| Comando | Descripción |
|---------|-------------|
| `show cdp neighbors detail` | Info completa de vecinos CDP |
| `show processes cpu` | Uso de CPU del IOS |
| `show processes memory` | Uso de memoria del IOS |
| `show version` | Versión IOS completa y hardware |
| `debug ip packet` | Debug de paquetes IP en tiempo real |
| `debug ip routing` | Debug de cambios en tabla de rutas |
| `show tech-support` | Reporte completo del dispositivo |
| `terminal monitor` | Ver logs en sesión Telnet/SSH |

---

## Exportar configuración para el repositorio

Una vez completado el lab, exporta las configuraciones
para guardarlas en la carpeta configs/:
R1# show running-config

Copia el output completo y guárdalo en:
`labs-gns3/01-config-basica/configs/R1-config.txt`

Repite para SW1:
```cisco
SW1# show running-config
```
Guarda en:
`labs-gns3/01-config-basica/configs/SW1-config.txt`

---

## Conceptos aplicados en este lab

| Concepto | Dónde se aplica |
|----------|----------------|
| IOS real | Comportamiento idéntico a hardware físico |
| VPCS | Simulación de PCs en GNS3 |
| CDP | Descubrimiento de vecinos con IOS real |
| FastEthernet | Nombre de interfaz en IOS c7200 |
| write memory | Guardado permanente en GNS3 |
| show version | Información del IOS real |

---

## Capturas requeridas

| Archivo | Contenido |
|---------|-----------|
| topologia.png | Vista del canvas GNS3 con dispositivos y cables |
| r1-interfaces.png | show ip interface brief en R1 |
| r1-cdp-neighbors.png | show cdp neighbors detail en R1 |
| r1-show-version.png | show version en R1 con IOS real |
| ping-pc1-pc2.png | Ping exitoso de PC-1 a PC-2 en VPCS |
| trace-pc1-pc2.png | Traceroute de PC-1 a PC-2 |
