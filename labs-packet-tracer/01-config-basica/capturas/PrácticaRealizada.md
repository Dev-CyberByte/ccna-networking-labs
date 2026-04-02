## Práctica realizada

La topología de este laboratorio es sencilla pero representa algo
que se ve constantemente en entornos reales: dos redes separadas
que necesitan comunicarse a través de un router. Ese esquema básico
es la base de casi cualquier red empresarial, por pequeña que sea.

Uno de los puntos que más vale la pena destacar es la configuración
de seguridad aplicada desde el inicio. En un entorno real, dejar un
router o switch sin contraseñas es un riesgo enorme, cualquier persona
con acceso físico al dispositivo podría modificar su configuración sin
restricción alguna. Por eso se configuraron contraseñas en la consola,
en el modo privilegiado y en las líneas VTY, que son las que controlan
el acceso remoto vía Telnet o SSH. Además, se activó el cifrado de
contraseñas con `service password-encryption` para que no queden
visibles en texto plano dentro de la configuración.

Estas buenas prácticas, aunque parecen simples, son exactamente lo
que se espera en una red administrada profesionalmente. Un técnico
que configura un dispositivo sin aplicarlas está dejando una puerta
abierta que en producción podría tener consecuencias serias.

El laboratorio concluye verificando conectividad extremo a extremo
entre ambas redes, confirmando que tanto el enrutamiento como el
direccionamiento quedaron correctamente configurados.
