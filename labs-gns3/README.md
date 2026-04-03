# Labs — GNS3

Versión GNS3 de los labs principales. El contenido y objetivos
son los mismos que en Packet Tracer pero ejecutados en un entorno
de emulación real con imágenes IOS de Cisco.

La diferencia clave: en Packet Tracer todo es simulado,
en GNS3 los protocolos corren igual que en hardware real.

---

## ¿Por qué repetir los labs en GNS3?

| Aspecto | Packet Tracer | GNS3 |
|---------|--------------|------|
| IOS | Simulado | Real |
| Comandos | Limitados | Completos |
| Comportamiento | Aproximado | Idéntico al hardware |
| Topologías | Limitadas | Sin límite |
| Integración Linux | No | Sí |
| Valor en entrevistas | Medio | Alto |

---

## Requisitos

- GNS3 2.x instalado
- GNS3 VM (recomendado para mejor rendimiento)
- Imagen IOS: Cisco c7200 o c3725 para routers
- Imagen IOS: Cisco c3560 o IOSvL2 para switches
- RAM mínima: 8 GB para correr 3-4 dispositivos simultáneos

---

## Diferencias importantes vs Packet Tracer

### Interfaces
En GNS3 los routers usan nombres distintos según el IOS:

| Packet Tracer | GNS3 (c7200) |
|--------------|--------------|
| GigabitEthernet0/0 | FastEthernet0/0 |
| GigabitEthernet0/1 | FastEthernet0/1 |
| Serial0/0/0 | Serial1/0 |

> Ajusta los nombres de interfaz según tu imagen IOS.
> Usa `show interfaces` para ver las interfaces disponibles.

### Guardado de configuración
En GNS3 el startup-config puede perderse al cerrar el proyecto
si no guardas correctamente:
Router# copy running-config startup-config
Router# write memory

Además guarda el proyecto GNS3 desde File → Save Project.

### Rendimiento
Cada router en GNS3 consume CPU real.
Con más de 4 routers activos puede volverse lento.
Recomendación: usar GNS3 VM en lugar de correr directo en el PC.

---

## Estructura de cada lab
lab-nombre/
├── README.md          <- Instrucciones adaptadas a GNS3
├── topologia.gns3     <- Archivo del proyecto GNS3
├── configs/           <- Configuraciones exportadas
│   ├── R1-config.txt
│   ├── R2-config.txt
│   └── SW1-config.txt
└── capturas/          <- Screenshots del lab
├── topologia.png
└── verificacion.png

---

## Lista de labs

| # | Lab | Equivalente PKT | Dificultad |
|---|-----|----------------|------------|
| 01 | Configuración básica | Lab PKT 01 | ⭐ |
| 02 | VLANs y Trunking | Lab PKT 03 | ⭐⭐ |
| 03 | OSPF Single Area | Lab PKT 07 | ⭐⭐⭐ |
| 04 | NAT y PAT | Lab PKT 09 | ⭐⭐⭐ |
| 05 | Topología multi-router | Exclusivo GNS3 | ⭐⭐⭐ |

---

## Cómo exportar la configuración de un dispositivo

Una vez configurado un dispositivo en GNS3 exporta
su configuración para documentarla en la carpeta configs/:
Router# show running-config

Copia el output y pégalo en un archivo .txt en la carpeta configs/.

---

## Referencias

- Lab PKT equivalente siempre indicado en cada lab
- Teoría: misma que los labs de Packet Tracer
- Ajustes de interfaz documentados en cada lab
