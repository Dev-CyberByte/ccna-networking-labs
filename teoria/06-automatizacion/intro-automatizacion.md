# Automatización y Programabilidad de Redes

La automatización de redes es el uso de software para configurar, gestionar
y operar dispositivos de red sin intervención manual. En redes tradicionales
cada dispositivo se configura uno por uno. Con automatización se gestionan
cientos de dispositivos desde un solo punto.

Este módulo es el dominio 6 del examen CCNA y aunque tiene menos peso
que los anteriores, marca la diferencia entre un técnico tradicional
y uno moderno.

---

## Redes tradicionales vs redes automatizadas

| Aspecto | Red tradicional | Red automatizada |
|---------|----------------|-----------------|
| Configuración | Manual, dispositivo por dispositivo | Masiva desde un controlador |
| Errores | Frecuentes (humanos) | Mínimos (código) |
| Velocidad de cambios | Lenta | Inmediata |
| Escalabilidad | Difícil | Fácil |
| Documentación | Manual y desactualizada | Automática |
| Consistencia | Variable | Garantizada |

---

## Planos de una red

Para entender la automatización hay que entender cómo se divide
el trabajo dentro de un dispositivo de red.

### Plano de datos (Data Plane / Forwarding Plane)
Es donde viajan los paquetes reales. El dispositivo reenvía tráfico
basándose en su tabla de enrutamiento o tabla CAM.
```
Paquete entra → se consulta la tabla → paquete sale por la interfaz correcta
```

### Plano de control (Control Plane)
Es donde se toman las decisiones de enrutamiento. Aquí corren los
protocolos como OSPF, STP, y se construyen las tablas que usa
el plano de datos.
```
OSPF aprende rutas → construye tabla de enrutamiento → plano de datos la usa
```

### Plano de gestión (Management Plane)
Es donde el administrador interactúa con el dispositivo.
SSH, Telnet, SNMP, APIs, todo esto es el plano de gestión.
```
Administrador → SSH → router → cambios de configuración
```

### Resumen de planos
```
┌─────────────────────────────────────┐
│        Plano de Gestión             │  ← SSH, SNMP, APIs, Telnet
├─────────────────────────────────────┤
│        Plano de Control             │  ← OSPF, STP, ARP, DHCP
├─────────────────────────────────────┤
│        Plano de Datos               │  ← Reenvío de paquetes, frames
└─────────────────────────────────────┘
```

---

## SDN — Software Defined Networking

### ¿Qué es SDN?
SDN separa el plano de control del plano de datos. En lugar de que
cada dispositivo tome sus propias decisiones, un **controlador centralizado**
toma todas las decisiones de enrutamiento y las distribuye a los dispositivos.
```
Red tradicional:
┌──────────┐  ┌──────────┐  ┌──────────┐
│ Router 1 │  │ Router 2 │  │ Router 3 │
│ control  │  │ control  │  │ control  │  ← cada uno decide solo
│  datos   │  │  datos   │  │  datos   │
└──────────┘  └──────────┘  └──────────┘

Red SDN:
          ┌─────────────────┐
          │   Controlador   │  ← toma todas las decisiones
          │      SDN        │
          └────────┬────────┘
         ┌─────────┼─────────┐
    ┌────┴───┐ ┌───┴────┐ ┌──┴─────┐
    │ Switch │ │ Switch │ │ Switch │  ← solo reenvían paquetes
    │  datos │ │  datos │ │  datos │
    └────────┘ └────────┘ └────────┘
```

### Beneficios de SDN

- Configuración centralizada desde un solo punto
- Visibilidad completa de toda la red
- Cambios masivos en segundos
- Programable mediante APIs
- Independiente del fabricante (en teoría)

### Cisco SD-WAN y SD-Access
Cisco implementa SDN con sus propias soluciones:

| Solución | Uso |
|----------|-----|
| SD-WAN (Viptela) | Gestión centralizada de WAN entre sucursales |
| SD-Access (DNA Center) | Automatización de redes de campus LAN |
| ACI | Automatización de centros de datos |

---

## APIs en redes

### ¿Qué es una API?
Una API (Application Programming Interface) es una interfaz que permite
que dos programas se comuniquen entre sí. En redes, las APIs permiten
que software externo configure y consulte dispositivos sin usar CLI.

### REST API
Es el tipo de API más común en redes modernas. Usa HTTP para comunicarse
y JSON o XML para intercambiar datos.
```
Programa Python                    Router/Controlador
      │                                    │
      │── GET /api/v1/interfaces ─────────►│
      │                                    │
      │◄─ 200 OK + datos en JSON ──────────│
      │                                    │
      │── POST /api/v1/vlans ─────────────►│
      │   (body: {"vlan": 10, "name": "Ventas"})
      │                                    │
      │◄─ 201 Created ────────────────────-│
```

### Métodos HTTP en REST APIs

| Método | Acción | Equivalente en CLI |
|--------|--------|--------------------|
| GET | Leer información | show |
| POST | Crear nuevo recurso | configurar algo nuevo |
| PUT | Reemplazar recurso completo | reconfigurar todo |
| PATCH | Modificar parte del recurso | cambiar un valor |
| DELETE | Eliminar recurso | no (comando) |

### Códigos de respuesta HTTP

| Código | Significado |
|--------|-------------|
| 200 | OK — solicitud exitosa |
| 201 | Created — recurso creado |
| 400 | Bad Request — error en la solicitud |
| 401 | Unauthorized — credenciales incorrectas |
| 403 | Forbidden — sin permisos |
| 404 | Not Found — recurso no existe |
| 500 | Internal Server Error — fallo del servidor |

### Formatos de datos

#### JSON — JavaScript Object Notation
Es el formato más común en APIs modernas. Fácil de leer y escribir.
```json
{
  "interface": {
    "name": "GigabitEthernet0/0",
    "ip_address": "192.168.1.1",
    "mask": "255.255.255.0",
    "status": "up"
  }
}
```

#### XML — Extensible Markup Language
Más verboso que JSON. Algunos dispositivos Cisco aún lo usan.
```xml
<interface>
  <name>GigabitEthernet0/0</name>
  <ip_address>192.168.1.1</ip_address>
  <mask>255.255.255.0</mask>
  <status>up</status>
</interface>
```

#### YAML — YAML Ain't Markup Language
Muy usado en herramientas de automatización como Ansible.
Es el más legible de los tres.
```yaml
interface:
  name: GigabitEthernet0/0
  ip_address: 192.168.1.1
  mask: 255.255.255.0
  status: up
```

---

## Herramientas de automatización

### Ansible
La herramienta de automatización más usada en redes.
No requiere agente en los dispositivos, usa SSH.
Los archivos de configuración se escriben en YAML y se llaman **playbooks**.
```yaml
---
- name: Configurar hostname en routers
  hosts: routers
  gather_facts: no

  tasks:
    - name: Establecer hostname
      ios_config:
        lines:
          - hostname R1
```

### Terraform
Herramienta de infraestructura como código (IaC).
Declara el estado deseado de la red y Terraform lo aplica.
Muy usado para infraestructura en la nube.

### Python con Netmiko
Netmiko es una librería Python que simplifica la conexión SSH
a dispositivos de red para automatizar tareas.
```python
from netmiko import ConnectHandler

router = {
    "device_type": "cisco_ios",
    "host": "192.168.1.1",
    "username": "admin",
    "password": "AdminPass123",
}

with ConnectHandler(**router) as conexion:
    output = conexion.send_command("show ip interface brief")
    print(output)
```

### NETCONF y RESTCONF
Protocolos estándar para gestión de dispositivos mediante APIs.

| Protocolo | Transporte | Formato | Puerto |
|-----------|-----------|---------|--------|
| NETCONF | SSH | XML | 830 |
| RESTCONF | HTTPS | JSON/XML | 443 |

Habilitarlos en Cisco IOS:
```
Router(config)# netconf-yang
Router(config)# restconf
```

---

## Cisco DNA Center

Es la plataforma de gestión centralizada de Cisco para SD-Access.
Permite automatizar, asegurar y analizar redes de campus desde
una interfaz web con APIs REST.

### Funciones principales

| Función | Descripción |
|---------|-------------|
| Design | Diseño jerárquico de la red |
| Policy | Políticas de acceso basadas en grupos |
| Provision | Aprovisionamiento automático de dispositivos |
| Assurance | Monitoreo y análisis con IA |

---

## Comparativa de herramientas

| Herramienta | Tipo | Lenguaje | Agente | Uso principal |
|-------------|------|----------|--------|---------------|
| Ansible | Automatización | YAML | No | Configuración masiva |
| Terraform | IaC | HCL | No | Infraestructura en nube |
| Python/Netmiko | Scripting | Python | No | Scripts personalizados |
| DNA Center | Plataforma | GUI/API | No | Redes Cisco campus |

---

## Gestión de configuraciones

### ¿Por qué es importante?
En redes grandes los dispositivos tienen configuraciones complejas.
Sin control de versiones es imposible saber qué cambió, cuándo y quién.

### Buenas prácticas
```
1. Guardar configuraciones en un repositorio Git
2. Usar plantillas (templates) para configuraciones estándar
3. Documentar cada cambio con comentarios
4. Tener un proceso de revisión antes de aplicar cambios
5. Probar en laboratorio antes de producción
```

### Tipos de gestión de configuración

| Tipo | Descripción |
|------|-------------|
| Out-of-band | Gestión por red separada dedicada solo a gestión |
| In-band | Gestión por la misma red que lleva el tráfico de datos |

---

## En el examen CCNA

Preguntas típicas de este módulo:

- *¿Qué es el plano de control?* → Donde corren los protocolos y se construyen las tablas
- *¿Qué es el plano de datos?* → Donde se reenvían los paquetes reales
- *¿Qué separa SDN del modelo tradicional?* → El plano de control del plano de datos
- *¿Qué método HTTP equivale a show?* → GET
- *¿Qué método HTTP crea un nuevo recurso?* → POST
- *¿Qué código HTTP indica éxito?* → 200 OK
- *¿Qué formato usa JSON?* → Pares clave-valor entre llaves
- *¿Qué protocolo usa NETCONF?* → SSH, puerto 830
- *¿Qué herramienta de automatización usa playbooks en YAML?* → Ansible
- *¿Ansible necesita agente en los dispositivos?* → No, usa SSH
- *¿Qué es DNA Center?* → Plataforma de gestión centralizada de Cisco para SD-Access
- *¿Qué es una API REST?* → Interfaz que usa HTTP para que programas configuren dispositivos
