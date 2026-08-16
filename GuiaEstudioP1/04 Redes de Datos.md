---
tags: [estudio/nube, semestre-6]
created: 2026-07-17
updated: 2026-08-12
materia: Arquitecturas de Nube y Sistemas Distribuidos
---

# Repaso a Redes de Datos

Base para el resto del curso: un sistema distribuido es, ante todo, procesos que se hablan **por una red**. Todo lo que falla en [[02 Sistemas Distribuidos - Conceptos]] falla porque la red se comporta como se explica aquí.

---

## Para qué sirve una red

Una red existe para **compartir**: datos, hardware (impresoras, discos, servidores), servicios, y para permitir la comunicación entre dispositivos.

La idea de fondo: sin red, cada recurso solo lo puede usar quien esté físicamente frente a la máquina. Con red, un recurso caro (una impresora, un servidor potente) lo aprovechan muchos.

---

## Tipos de redes

### Según el alcance geográfico

| Tipo | Alcance | Quién la administra | Ejemplo |
|---|---|---|---|
| **PAN** (Personal Area Network) | ~10 metros | El propio usuario | Bluetooth, smartwatch, audífonos inalámbricos |
| **LAN** (Local Area Network) | Un edificio o campus | Propiedad privada | Red de tu casa, de la universidad, de una empresa |
| **WAN** (Wide Area Network) | Países o continentes | Múltiples operadores | Internet |

**Lo que hay que entender más allá de la tabla**: la diferencia real no es solo el tamaño, es la **velocidad y el control**. En una LAN tú mandas: es tuya, es rápida y la latencia es de menos de un milisegundo. En una WAN atraviesas equipos de terceros, la latencia sube a decenas o cientos de milisegundos, y no controlas nada del camino. Por eso las falacias de Deutsch muerden tan fuerte al pasar de LAN a WAN.

Una WAN es, en esencia, **muchas LAN conectadas entre sí**.

### Según el rol del host

- **Cliente** — el que **solicita** servicios. Ejemplos: un navegador web, una app de celular.
- **Servidor** — el que **ofrece** servicios y queda esperando peticiones. Ejemplos: servidor web, servidor de base de datos, servidor DNS.

**Cuidado**: cliente y servidor son **roles**, no tipos de máquina. Un mismo computador puede ser servidor de una cosa y cliente de otra al mismo tiempo — un servidor web es servidor para el navegador, pero cliente cuando le pide datos a la base de datos.

---

## Qué se necesita para enviar un paquete

Para que un host le mande un paquete a otro hacen falta cinco cosas:

1. **Dirección IP** — para identificar al destino.
2. **Máscara de subred** — para saber si el destino está en la red local o afuera.
3. **Gateway** — la salida, si el destino está afuera.
4. **DNS** — si en vez de una IP se usó un nombre de dominio.
5. **Medio de transmisión** — WiFi, fibra, cable.

Estas cinco no son una lista suelta: son exactamente los pasos del razonamiento que hace tu máquina cada vez que manda algo. Vale la pena verlo como decisión encadenada:

> ¿Me dieron un nombre? → pregunto al **DNS** para obtener la **IP**.
> ¿Esa IP está en mi red? → lo decido con la **máscara**.
> Si está → le hablo directo por el **medio**.
> Si no está → se lo entrego al **gateway** para que lo saque.

---

## Direccionamiento IP

### Qué es una IP

**Qué es**: un identificador único que tiene cada dispositivo dentro de una red. IP significa *Internet Protocol*.

**Por qué existe**: porque para entregar información hay que saber a quién. Funciona como la dirección de una casa: sin dirección, el cartero no puede entregar nada, aunque las calles existan.

### IPv4 vs IPv6

| | IPv4 | IPv6 |
|---|---|---|
| Tamaño | 32 bits | 128 bits |
| Formato | `192.168.1.15` (4 números de 0-255) | `2001:db8::15` (hexadecimal) |
| Cantidad posible | ~4.300 millones | Prácticamente inagotable |
| Por qué existe | Fue el original | Nació **porque IPv4 se agotó** |

**Por qué se agotó IPv4**: 4.300 millones parecía infinito en los años 80, cuando había unos pocos miles de computadores conectados. Hoy hay más dispositivos que personas — cada celular, cada televisor, cada sensor pide una dirección. IPv6 usa 128 bits precisamente para no repetir el error.

**Dato para el examen**: si IPv4 se agotó hace años, ¿por qué Internet sigue funcionando? Por **NAT** (más abajo), que permite que miles de dispositivos compartan una sola IP pública. NAT es el parche que le compró tiempo a IPv4.

### IP privadas vs públicas

**IP privadas** — rangos reservados que **no se pueden usar directamente en Internet**:

| Rango | Notación CIDR |
|---|---|
| 10.0.0.0 – 10.255.255.255 | 10.0.0.0/8 |
| 172.16.0.0 – 172.31.255.255 | 172.16.0.0/12 |
| 192.168.0.0 – 192.168.255.255 | 192.168.0.0/16 |

**IP públicas** — todas las que **no** están en esos rangos. Son únicas en todo Internet, las administran organismos internacionales y se asignan a los ISP.

**Por qué existe la separación**: si cada dispositivo del mundo necesitara una dirección única globalmente, IPv4 se habría agotado mucho antes. Los rangos privados permiten que millones de redes locales reutilicen las mismas direcciones internamente (tu `192.168.1.1` y el de tu vecino conviven sin conflicto porque ninguno de los dos sale a Internet con esa dirección).

**Costos**:
- IP privada: **gratuita**, la asigna el router automáticamente.
- IP pública: la entrega el ISP. Puede ser **dinámica** (cambia cada tanto, incluida en el plan) o **estática** (fija, normalmente con costo adicional).

**Cuándo necesitas una IP pública estática**: cuando otros tienen que **encontrarte** a ti — alojar un servidor web, un servidor de correo, una VPN. Si solo navegas y consumes servicios, una dinámica con NAT te basta.

---

## Máscara de subred

**Qué es**: un valor que divide la dirección IP en dos partes — **cuál parte identifica a la red** y cuál identifica al **host** dentro de esa red.

**Por qué existe**: porque tu máquina necesita responder una pregunta antes de mandar cualquier paquete: *¿el destino está en mi misma red o afuera?* De la respuesta depende si le habla directo o si se lo entrega al router. Sin máscara, esa decisión es imposible.

**Cómo funciona**: la máscara marca con unos los bits de red y con ceros los de host. La máquina compara su propia IP y la del destino en la parte de red; si coinciden, están en la misma red.

**Ejemplo concreto**:

```
Mi IP:      192.168.1.15
Máscara:    255.255.255.0   (equivale a /24)
            └──── red ───┘└host┘

Destino A:  192.168.1.20  → red = 192.168.1  → IGUAL → le hablo directo
Destino B:  192.168.5.20  → red = 192.168.5  → DISTINTA → va al gateway
```

**Notación CIDR**: `/24` significa "los primeros 24 bits son de red". Como IPv4 tiene 32 bits, quedan 8 bits para hosts → 2⁸ = 256 direcciones, de las cuales **254 son usables** (una se reserva para identificar la red y otra para el broadcast).

| Máscara | CIDR | Direcciones totales | Hosts usables |
|---|---|---|---|
| 255.255.255.0 | /24 | 256 | 254 |
| 255.255.255.128 | /25 | 128 | 126 |
| 255.255.0.0 | /16 | 65.536 | 65.534 |

**Error típico**: olvidar restar las 2 direcciones reservadas y contestar 256 en vez de 254.

---

## Gateway (puerta de enlace)

**Qué es**: la dirección IP del router — la **salida** de tu red hacia otras redes.

**Por qué existe**: tu máquina solo sabe hablarle directamente a quien está en su misma red. Para todo lo demás necesita un intermediario que conozca el camino hacia afuera.

**Cómo funciona**: si la máscara determina que el destino **no** está en la red local, el paquete se le entrega al gateway, y este lo reenvía hacia la red siguiente. El proceso se repite router por router hasta llegar.

**Ejemplo**: escribes `google.com` desde tu casa. Tu PC (192.168.1.15) determina que la IP de Google no está en 192.168.1.x, así que le pasa el paquete al gateway (192.168.1.1, tu router), que lo manda al ISP, y de ahí sigue saltando hasta Google.

**Analogía**: la máscara es saber si el destinatario vive en tu edificio. Si vive, le tocas la puerta. Si no, se lo dejas al portero (gateway) para que lo despache afuera.

---

## DHCP

**Qué es**: *Dynamic Host Configuration Protocol*. Asigna automáticamente a cada dispositivo que se conecta: **dirección IP, máscara, gateway y DNS**.

**Por qué existe**: son exactamente los cuatro datos que hacen falta para comunicarse. Configurarlos a mano en cada dispositivo de una red de 200 equipos sería inviable, y además propenso a errores como asignar la misma IP dos veces.

**Cómo funciona**: el dispositivo nuevo pregunta a gritos en la red "¿hay algún servidor DHCP?", el servidor le responde ofreciéndole una configuración, el cliente la acepta y el servidor se la reserva por un tiempo determinado (*lease*).

**Ejemplo**: llegas a la universidad, te conectas al WiFi y navegas sin configurar nada. Eso fue DHCP entregándote los cuatro datos en menos de un segundo.

**NAT vs DHCP** — se confunden mucho porque los dos viven en el router:

| | DHCP | NAT |
|---|---|---|
| Qué hace | **Asigna** direcciones a los dispositivos de la red | **Traduce** direcciones privadas a una pública |
| Cuándo actúa | Al conectarse el dispositivo | Cada vez que un paquete sale o entra a Internet |
| Sin él... | Tocaría configurar cada equipo a mano | Los dispositivos privados no podrían salir a Internet |

---

## NAT

**Qué es**: *Network Address Translation*. Permite que **muchos dispositivos con IP privada compartan una sola IP pública** para salir a Internet.

**Por qué existe**: por el agotamiento de IPv4. Tu ISP te da **una** IP pública, pero en tu casa hay 15 dispositivos. NAT es lo que hace que los 15 puedan navegar con esa única dirección.

**Cómo funciona**: cuando un dispositivo interno manda un paquete, el router **reemplaza** la IP privada de origen por su propia IP pública y anota en una tabla qué conexión pertenece a quién. Cuando llega la respuesta, consulta la tabla y se la entrega al dispositivo correcto.

**Ejemplo del curso**: 20 computadores de una oficina salen a Internet usando la misma IP pública del router. Para los servidores de afuera, los 20 se ven como un solo cliente.

**Consecuencia importante (y muy preguntable)**: NAT funciona bien para **salir**, pero rompe el **entrar**. Como los dispositivos internos no tienen dirección propia visible desde afuera, nadie de Internet puede iniciar una conexión hacia ellos. Si quisieras alojar un servidor web en esa oficina, tendrías que configurar explícitamente una redirección de puertos en el router (*port forwarding*), o contratar una IP pública dedicada.

Esa asimetría es también un efecto secundario de seguridad: NAT actúa como una barrera implícita contra conexiones entrantes no solicitadas.

---

## DNS

**Qué es**: *Domain Name System*. Traduce nombres de dominio a direcciones IP: `google.com` → `142.250.x.x`. Es la "guía telefónica de Internet".

**Por qué existe**: las máquinas se encuentran por número, pero los humanos no recuerdan números. Además, si un servicio cambia de servidor, la IP cambia — el nombre no. DNS agrega una capa de indirección que permite mover la infraestructura sin que nadie tenga que actualizar nada.

**Cómo funciona**: es un sistema **jerárquico y distribuido**. Ningún servidor conoce todos los dominios del mundo; cada uno conoce su zona y sabe a quién preguntarle por el resto. La consulta va bajando por la jerarquía hasta encontrar al servidor autoritativo del dominio.

**Por qué el curso lo menciona como uno de los primeros sistemas distribuidos**: cumple la definición al pie de la letra — miles de servidores autónomos, repartidos por el mundo, que se coordinan por paso de mensajes y se presentan al usuario como **un solo servicio coherente**. Además ilustra dos técnicas de escalabilidad vistas en la unidad de distribuidos: **particionamiento** (cada servidor sabe solo de su zona) y **caché** (las respuestas se guardan un tiempo para no repetir la consulta).

**Ejemplo**: `nslookup google.com` te muestra qué servidor DNS respondió y qué IP entregó.

---

## Firewall

**Qué es**: un sistema de seguridad que **permite o bloquea** conexiones según un conjunto de reglas.

**Por qué existe**: tener un servicio accesible por red significa que cualquiera puede intentar conectarse, incluido quien no debería. El firewall define quién puede hablar con qué.

**Cómo funciona**: examina cada paquete y decide según reglas basadas en IP de origen/destino, puerto y protocolo.

**Dónde puede vivir**:
- En el computador — Windows Defender Firewall, UFW en Linux.
- En el router — protege toda la red.
- En la nube — **AWS Security Groups**.

**Ejemplo**: una regla típica de servidor web permite entrada solo por los puertos 80 y 443 desde cualquier IP, y por el 22 (SSH) solo desde la IP de la oficina. Cualquier otro intento se descarta.

---

## Servicios y puertos

**Qué es un servicio**: un programa que permanece ejecutándose en segundo plano, **esperando solicitudes**. Generalmente queda escuchando en un puerto.

**Qué es un puerto y por qué existe**: la IP identifica a la **máquina**, pero una máquina corre muchos servicios a la vez. El puerto identifica **a cuál de ellos** va dirigido el paquete. Sin puertos, un servidor con web y base de datos no sabría a cuál entregarle lo que llega.

**Analogía**: la IP es la dirección del edificio; el puerto es el número del apartamento.

| Puerto | Servicio |
|---|---|
| 22 | SSH |
| 53 | DNS |
| 80 | HTTP |
| 443 | HTTPS |
| 3306 | MySQL |
| 5432 | PostgreSQL |

---

## Protocolo HTTP

**Qué es**: *HyperText Transfer Protocol*. Protocolo de aplicación cliente-servidor que define cómo se piden y se entregan recursos (HTML, imágenes, JSON) por la red.

**Por qué existe**: para que cualquier cliente y cualquier servidor, escritos por gente distinta en lenguajes distintos, puedan entenderse. Es un ejemplo directo de cómo se logra la **heterogeneidad** que se busca en sistemas distribuidos.

**Cómo funciona**: modelo petición-respuesta. El cliente manda un **método** sobre una URL, con cabeceras y a veces un cuerpo; el servidor responde con un **código de estado** y un cuerpo.

- **Métodos**: `GET` (traer), `POST` (crear), `PUT` (reemplazar), `PATCH` (modificar parcialmente), `DELETE` (borrar).
- **Códigos de estado**: `2xx` éxito, `3xx` redirección, `4xx` error del cliente (pediste mal), `5xx` error del servidor (el servidor falló).

**Sin estado (stateless)** — la propiedad clave: **cada petición es independiente**; el servidor no recuerda por sí solo lo que pasó en la anterior.

**Por qué eso importa**: es lo que permite que HTTP escale. Como ninguna petición depende de la anterior, cualquiera de los 50 servidores detrás de un balanceador puede atenderla. Si el servidor tuviera que recordar tu sesión en memoria, tendrías que volver siempre al mismo — y ahí se acaba la escalabilidad horizontal.

**El costo**: como el servidor no recuerda nada, hay que simular el estado. De ahí salen **cookies, tokens y sesiones**: el cliente reenvía en cada petición la prueba de quién es.

**HTTPS**: es HTTP + cifrado **TLS**. Usa el puerto **443** en vez del 80. Protege confidencialidad (nadie lee el contenido) e integridad (nadie lo altera en el camino).

---

## Seguridad: los dos triángulos

Son dos cosas distintas que se complementan, y confundirlos es error clásico de examen.

### Triángulo CIA — los objetivos

**Qué es**: los tres objetivos que persigue cualquier control de seguridad.

- **Confidencialidad** — solo quien debe ver la información, la ve.
- **Integridad** — nadie altera la información sin autorización.
- **Disponibilidad** — la información y el servicio están accesibles cuando se necesitan.

**Por qué son tres y no uno**: porque protegerlos entra en conflicto. Cifrar todo y apagar el servidor da confidencialidad perfecta y disponibilidad nula. Diseñar seguridad es equilibrar los tres, no maximizar uno.

**Ejemplo de cada fallo**: una filtración de datos rompe la confidencialidad; una transacción alterada rompe la integridad; un ataque de denegación de servicio rompe la disponibilidad.

### Triada AAA — el mecanismo

**Qué es**: los tres pasos del control de acceso.

- **Autenticación** — verificar **quién eres** (usuario y contraseña, MFA, certificado).
- **Autorización** — verificar **qué puedes hacer** ya identificado (roles, permisos).
- **Auditoría / Accounting** — **registrar qué hiciste** (logs, trazabilidad).

**Cómo se relacionan CIA y AAA**: **AAA es el mecanismo con el que se consiguen los objetivos del CIA.** Autenticar y autorizar protegen la confidencialidad y la integridad; auditar permite detectar y reconstruir lo que pasó cuando algo falla.

**Autenticación vs autorización** (el contraste más preguntado): autenticación es la portería del edificio, que confirma que eres tú. Autorización es qué puertas abre tu tarjeta una vez adentro. Estar autenticado no implica poder hacer todo — un usuario válido puede tener prohibido borrar registros.

**Error típico**: responder "AAA" cuando preguntan por los objetivos de seguridad, o "CIA" cuando preguntan por control de acceso. **CIA = qué se protege. AAA = cómo se controla el acceso.**

---

## Cómo funciona un sitio web (todo junto)

Este flujo integra casi todo lo de la nota, por eso se pregunta tanto:

1. El usuario escribe una **URL** en el navegador.
2. El navegador consulta al **DNS** para traducir el nombre a una IP.
3. El DNS responde con la **dirección IP** del servidor.
4. El sistema usa la **máscara** para determinar que esa IP no está en la red local, y entrega el paquete al **gateway**.
5. El cliente envía una solicitud **HTTP/HTTPS** al puerto 80 o 443 de esa IP.
6. La solicitud viaja por **Internet**, saltando de router en router (pasando por **NAT** al salir de la red local).
7. Llega al **servidor**, donde el **firewall** decide si permite la conexión.
8. El **servicio** que escucha en ese puerto procesa la petición.
9. El servidor devuelve una **respuesta** con su código de estado.
10. El navegador la interpreta y **muestra la página**.

> Cliente → Red local → Gateway/NAT → Internet → Firewall → Servidor → Respuesta → Cliente

---

## Atributos no funcionales

**Qué son**: características de **calidad** del sistema. No dicen *qué hace* el sistema, sino **cómo debe hacerlo**.

**Por qué importan**: dos sistemas pueden cumplir exactamente los mismos requisitos funcionales y ser radicalmente distintos en valor. Los dos "guardan pedidos"; uno responde en 200 ms y aguanta 10.000 usuarios, el otro tarda 8 segundos y se cae con 50.

### Escalabilidad
Capacidad de soportar más usuarios o más carga de trabajo.
- **Vertical** — agregar más CPU o RAM **a la misma máquina**. Simple, pero tiene techo físico y deja un punto único de falla.
- **Horizontal** — agregar **más máquinas**. Sin techo teórico y permite redundancia, pero obliga a resolver todos los problemas de sistemas distribuidos.

### Rendimiento
Qué tan rápido responde el sistema. Se mide con:
- **Tiempo de respuesta** — cuánto tarda una operación completa.
- **Latencia** — el retraso de la comunicación.
- **Throughput** — cuántas solicitudes atiende por segundo.

**Latencia vs throughput**: no son lo mismo y pueden ir en direcciones opuestas. Un sistema puede procesar 10.000 peticiones por segundo (throughput alto) pero tardar 2 segundos en cada una (latencia mala), si las procesa en lotes grandes.

### Disponibilidad
Proporción del tiempo que el servicio funciona correctamente.

| Disponibilidad | Nombre | Caída máxima al año |
|---|---|---|
| 99 % | "dos nueves" | ~3,65 días |
| 99,9 % | "tres nueves" | ~8,76 horas |
| 99,99 % | "cuatro nueves" | ~52,6 minutos |

**Lo que hay que entender**: cada nueve adicional cuesta mucho más que el anterior, porque exige redundancia en más niveles. Por eso la disponibilidad se **negocia** según lo que el negocio realmente necesita, no se maximiza porque sí.

### Observabilidad (seguimiento)
Capacidad de conocer el estado interno del sistema desde afuera. Sus tres pilares:
- **Logs** — registro de eventos puntuales ("qué pasó").
- **Métricas** — valores numéricos en el tiempo ("cuánto y cuándo").
- **Trazas** — el recorrido completo de una petición a través de varios servicios ("por dónde pasó y dónde se demoró").

**Por qué es crítica en sistemas distribuidos**: en una sola máquina puedes depurar leyendo un log. Cuando una petición atraviesa 8 servicios en máquinas distintas, sin trazas es imposible saber cuál fue el que falló o se demoró.

### Otros del curso
**Seguridad**, **mantenibilidad** y **tolerancia a fallos** (desarrollada en [[02 Sistemas Distribuidos - Conceptos]]).

---

## Glosario complementario

### IaC (Infrastructure as Code)

**Qué es**: administrar la infraestructura mediante **código** en lugar de configurarla a mano por consola.

**Por qué existe**: la configuración manual no se puede repetir con exactitud, no queda registrada, y depende de que alguien recuerde qué tocó. Si el servidor se pierde, reconstruirlo es adivinar.

**Cómo funciona**: escribes en un archivo cómo debe verse la infraestructura (3 servidores, esta red, este firewall) y la herramienta se encarga de crearla y de mantenerla igual. El archivo se versiona en Git como cualquier código.

**Herramientas**: Terraform, Ansible, CloudFormation, Pulumi.

**Beneficios**: automatización, repetibilidad, menos errores humanos, mantenimiento sencillo. Es lo que hace posible la ventaja de **recuperación rápida** de la nube: reconstruir todo es volver a ejecutar el script.

### Patrones

**Qué son**: soluciones reutilizables a problemas comunes de diseño o arquitectura de software. No son código listo, sino la forma probada de resolver un problema recurrente.

**Ejemplos**: MVC, Cliente-Servidor, Microservicios, Event Driven.

---

## Comandos útiles

| Linux | Windows | Qué hace |
|---|---|---|
| `ip a` (antes `ifconfig`) | `ipconfig` | Muestra las interfaces de red y sus IP |
| `ping google.com` | `ping google.com` | Comprueba si hay conectividad y mide la latencia |
| `traceroute google.com` | `tracert google.com` | Muestra los saltos (routers) del camino hasta el destino |
| — | `nslookup google.com` | Consulta al DNS qué IP corresponde a un nombre |
| `netstat -tulnp` | `netstat -an` | Lista los puertos en modo **LISTEN** (escuchando) y qué proceso los usa |
| `ss -tulnp` | — | Versión moderna de `netstat` en Linux; más rápida en sistemas con muchas conexiones |
| `nmap <IP>` | `nmap <IP>` | Escanea qué puertos están abiertos en un host, propio o remoto |

**Dato del curso**: una máquina puede tener **varias interfaces** (Ethernet, WiFi, VPN, Bluetooth) y **cada una con su propia IP**. Por eso `ip a` casi siempre lista más de una.

**Por qué revisar los puertos en LISTEN es una práctica de seguridad**: cada servicio escuchando en un puerto es una puerta potencial de entrada. Un puerto en modo LISTEN que no corresponde a ningún servicio que tú instalaste a propósito (un `netstat`/`ss` que muestra algo inesperado) es indicio de un servicio mal configurado o de software no deseado, y **debe "bajarse"** (detener el servicio) si no es necesario — cada puerto abierto de más es superficie de ataque sin ningún beneficio a cambio.

**Máquina virtual Linux gratuita**: <https://shell.cloud.google.com> — Google Cloud Shell da una VM Linux con terminal y herramientas ya instaladas.

---

## Errores típicos a evitar

- **NAT ≠ DHCP.** NAT traduce direcciones para salir a Internet; DHCP las asigna al conectarse.
- Olvidar restar 2 direcciones al calcular hosts usables en una subred (/24 son 254, no 256).
- **CIA ≠ AAA.** CIA son los objetivos de seguridad; AAA es el mecanismo de control de acceso.
- **Autenticación ≠ autorización.** Estar identificado no significa tener permiso.
- Creer que una IP pública dinámica sirve para alojar un servidor accesible desde afuera.
- Confundir **latencia** con **throughput** al hablar de rendimiento.
- Decir que HTTP "guarda la sesión". No la guarda: el estado lo reenvía el cliente en cada petición.

---

## Preguntas de repaso

1. Tu PC tiene IP 192.168.1.15 con máscara /24. Explica paso a paso qué hace tu máquina para mandarle un paquete a 192.168.1.20 y qué hace distinto para mandárselo a 8.8.8.8.

2. Una oficina tiene 20 computadores y el ISP le dio una sola IP pública dinámica. ¿Qué tecnología permite que los 20 naveguen? ¿Qué tendría que cambiar si además quisiera alojar un servidor web accesible desde Internet, y por qué no le sirve lo que ya tiene?

3. ¿Por qué se dice que DNS es un sistema distribuido y no simplemente "un servidor con una tabla gigante"? Menciona qué técnica de escalabilidad usa.

4. Un servidor tiene 99,9 % de disponibilidad. Traduce eso a horas de caída al año y explica qué **no** garantiza ese número.

5. Explica por qué el hecho de que HTTP sea *stateless* es una **ventaja** para escalar horizontalmente, y qué costo trae esa decisión.

6. Un sistema autentica correctamente a los usuarios pero no registra qué hace cada uno. ¿Qué parte de AAA le falta y qué objetivo del CIA queda comprometido?

7. Tu aplicación empezó a responder lento. Tienes logs pero no trazas. ¿Por qué eso es un problema si la petición pasa por 5 servicios distintos, y qué pilar de observabilidad necesitas?

8. Una empresa quiere aguantar más carga y duda entre comprar un servidor el doble de potente o comprar dos servidores iguales al que tiene. Explica los dos caminos, sus nombres técnicos y qué problema nuevo aparece con el segundo.

## Enlaces

- [[Arquitecturas de Nube y Sistemas Distribuidos MOC]]
- [[02 Sistemas Distribuidos - Conceptos]]
- [[03 Computacion en la Nube]]
- [[01 Guia de Repaso - Parcial 1]]
- [[02 Areas/Estudio/Semestre 6/Semestre 6 MOC]]
