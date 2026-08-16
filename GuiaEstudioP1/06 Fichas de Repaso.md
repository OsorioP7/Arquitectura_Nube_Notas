---
tags: [estudio/nube, ficha, repaso, semestre-6]
materia: Arquitecturas de Nube y Sistemas Distribuidos
fecha: "2026-08-12"
---

# 06 Fichas de Repaso

Todas las fichas del Parcial 1, en orden de prioridad de estudio. **Tapa la respuesta y contesta de memoria antes de leer.**

- [Sistemas Distribuidos](#sistemas-distribuidos) — prioridad alta
- [Computacion en la Nube](#computacion-en-la-nube)
- [Redes de Datos](#redes-de-datos)

---

# Sistemas Distribuidos

**Q: ¿Qué es un sistema distribuido?**
R: Una colección de computadoras **autónomas**, conectadas por red, que se coordinan **por paso de mensajes** y se presentan al usuario como **un solo sistema coherente**. Las dos mitades importan: si no son autónomas no es distribuido, y si el usuario nota que son muchas máquinas, está mal hecho.
*Ejemplo*: una búsqueda en Google la atienden cientos de máquinas (DNS, balanceo, fragmentos del índice) y tú ves una sola caja de texto.

**Q: ¿Por qué existen los sistemas distribuidos? Nombra los tres muros que rompen.**
R: Porque una sola máquina choca contra tres límites que no se arreglan comprando hardware más caro:
1. **Físico** — hay un tope de CPU/RAM por servidor, y el precio sube más rápido que la potencia.
2. **Geográfico** — ningún servidor único da baja latencia a usuarios en tres continentes.
3. **Disponibilidad** — una máquina sola es un punto único de falla.

**Q: Nombra las 4 consecuencias de distribuir.**
R: Sin **reloj global** · **fallas independientes** (parciales) · sin **memoria compartida** · **concurrencia**.
Corolario: **no hay estado global perceptible**.
*Todo lo difícil del semestre sale de estas cuatro.*

**Q: ¿Por qué la falta de reloj global es un problema real, no teórico?**
R: Porque sin reloj común no puedes decidir cuál de dos eventos ocurrió primero si pasaron en máquinas distintas — y eso es justo lo que necesitas para resolver conflictos.
*Ejemplo*: dos ediciones del mismo documento, una marcada 10:00:05 y otra 10:00:03. Si el reloj de la segunda iba 4 s atrasado, en realidad fue posterior. Resolver por hora de pared **guarda la versión equivocada**.
*Solución*: relojes lógicos (Lamport), que miden **causalidad**, no tiempo.
*Trampa*: NTP reduce la desviación, no la elimina.

**Q: ¿Por qué una falla parcial es peor que una caída total?**
R: Porque en una máquina sola el sistema funciona o no funciona (binario). En uno distribuido existe el estado "funciona a medias", y hay que programar explícitamente para él.
Peor aún: cuando un nodo no responde, **no puedes distinguir** entre (a) se cayó, (b) está lento, (c) se cortó la red. Las tres se ven idénticas desde afuera, y cada una pediría una reacción distinta.

**Q: ¿Cómo se reintenta con seguridad si no sabes si la operación se ejecutó?**
R: Haciéndola **idempotente**: se manda un identificador único de transacción y el servidor lo reconoce, de modo que ejecutar la misma operación dos veces tenga el mismo efecto que ejecutarla una. Así reintentar deja de arriesgar un doble cobro.

**Q: Nombra las metas de diseño de un sistema distribuido.**
R: Heterogeneidad · Openness (apertura) · Seguridad · Escalabilidad · Manejo de fallas (dependability) · Concurrencia · **Transparencia**.

**Q: ¿Qué es openness y cómo se logra?**
R: Que el sistema se pueda **extender y reimplementar** — agregar componentes o reemplazar uno por otra implementación sin rehacerlo todo. Se logra **publicando las interfaces**: si el contrato es público y estable, cualquiera puede escribir otra implementación que lo cumpla.
*Ejemplo*: cualquier servidor que implemente el protocolo DNS puede reemplazar al tuyo.

**Q: ¿Cuáles son las 3 técnicas de escalabilidad del curso?**
R: **Ocultar latencias** (trabajar asíncrono en vez de esperar bloqueado) · **ocultar la distribución** mediante particionamiento (cada nodo atiende su porción, como las zonas de DNS) · **ocultar la replicación** mediante caché (copias cerca de quien las usa).

**Q: Nombra los 8 tipos de transparencia.**
R: Acceso · Ubicación · Migración · Reubicación · Replicación · Concurrencia · Fallo · Persistencia.

**Q: Transparencia de acceso vs. de ubicación.**
R: **Acceso** = lo usas **igual** sea local o remoto (mismo código para un archivo en disco que en red). **Ubicación** = **no sabes dónde** está físicamente (`google.com` no te dice el datacenter).
Puedes tener una sin la otra: una URL con IP fija da transparencia de acceso pero no de ubicación.

**Q: Transparencia de migración vs. de reubicación.**
R: **Migración** = el recurso se mueve **entre** usos (tu buzón cambia de servidor de noche). **Reubicación** = se mueve **durante** el uso, sin cortar la sesión (sigues viendo la película mientras cambia el servidor que te la envía). La reubicación es mucho más difícil porque hay que trasladar el estado en vivo.

**Q: ¿Por qué más transparencia no siempre es mejor?**
R: Porque ocultar del todo que una llamada es remota lleva a tratarla como local — sin timeouts ni reintentos — y ahí muerden las falacias de Deutsch. A veces conviene que el programador **sepa** que cruza la red.

**Q: Nombra 4 falacias de Deutsch y su efecto.**
R:
- *La red es confiable* → no manejas reintentos; el primer paquete perdido tumba la operación.
- *La latencia es cero* → haces 500 llamadas remotas donde cabía una; funciona local y se arrastra en producción.
- *La red es segura* → no cifras ni autenticas; cualquiera en el camino lee o altera.
- *La topología no cambia* → quemas IPs en el código y todo se rompe al mover un servidor.
*(Las otras: ancho de banda infinito, un solo administrador, transporte gratis, red homogénea.)*

**Q: ¿Qué es la clasificación de Flynn y cuáles son sus 4 categorías?**
R: Taxonomía de arquitecturas según cuántos **flujos de instrucciones** y de **datos** maneja la máquina a la vez.
- **SISD** — una instrucción, un dato: CPU secuencial normal.
- **SIMD** — una instrucción, muchos datos: GPU aplicando un filtro a millones de píxeles.
- **MISD** — muchas instrucciones, un dato: casi sin ejemplos prácticos (redundancia crítica).
- **MIMD** — muchas instrucciones, muchos datos: **aquí caen los sistemas distribuidos**.

**Q: ¿Cómo se subdivide MIMD?**
R: Según **cómo se comunican** los procesadores:
- **Multiprocesadores** — memoria **compartida**, fuertemente acoplados (T_comm < T_proc). Subtipos **UMA** y **NUMA**.
- **Multicomputadores** — memoria **distribuida**, débilmente acoplados (T_comm >> T_proc). Subtipos **MPP** y **clusters**.

**Q: ¿Los sistemas distribuidos son fuerte o débilmente acoplados? ¿Por qué se falla tanto esta?**
R: **Débilmente acoplados** — memoria propia por nodo y comunicación por red, con retraso alto.
Se falla porque "fuertemente acoplado" suena a "muy integrado", y el sistema se ve integrado desde fuera. Pero el acoplamiento se refiere al **medio físico**, no a qué tan coordinado parece. Esa independencia es justo lo que permite escalar y tolerar fallas.

**Q: Tolerancia a fallas vs. confiabilidad vs. disponibilidad.**
R: **Tolerancia a fallas** = seguir operando pese a un fallo, mediante redundancia. **Confiabilidad** = que los datos sean íntegros y correctos. **Disponibilidad** = proporción del tiempo que el servicio responde.
Un sistema puede estar **disponible** (siempre responde) y ser poco **confiable** (a veces responde datos corruptos).

**Q: ¿Qué es RPC y cuál es su riesgo?**
R: *Remote Procedure Call* — hace que llamar una función en otra máquina se **vea** igual que llamarla localmente; la librería empaqueta argumentos, los manda y devuelve el resultado. Da transparencia de acceso, pero **esconde que la llamada puede tardar o fallar**, que es exactamente la trampa de las falacias de Deutsch.

**Q: ¿Qué define la arquitectura cliente-servidor y por qué es asimétrica?**
R: Los componentes se dividen en dos roles fijos: **cliente** (solicita) y **servidor** (procesa y responde), siguiendo el patrón **request-reply**. Es asimétrica porque **solo el cliente inicia** la conexión; el servidor nunca le "llama" al cliente por su cuenta, solo escucha y responde.
*Ventajas*: administración sencilla, fácil localizar fallas. *Desventajas*: punto único de falla (SPOF), escalabilidad limitada por el punto central.

**Q: ¿Qué significa que una operación sea idempotente? Clasifica GET, POST, PUT, DELETE.**
R: Que ejecutarla una vez o muchas veces produce **el mismo resultado final**. GET, PUT y DELETE son idempotentes; **POST no lo es** (crea un recurso nuevo cada vez).
*Por qué importa*: si una petición expira por timeout no sabes si se procesó; si es idempotente, reintentar es seguro.
*Trampa*: idempotente no significa "sin efectos" — DELETE borra algo (efecto real) y aun así es idempotente, porque repetirlo da el mismo resultado que hacerlo una vez.

**Q: ¿Qué es publish-subscribe y qué dos tipos de desacoplamiento aporta?**
R: Estilo donde publicadores emiten eventos y suscriptores los reciben a través de un **broker**, sin conocerse entre sí. Aporta **desacoplamiento referencial** (ninguno necesita la dirección del otro) y **desacoplamiento temporal** (no necesitan estar activos al mismo tiempo).
*Ejemplo*: un evento `pedido_creado` que consumen inventario, facturación y correos sin que el emisor sepa que existen.

**Q: Publish-subscribe vs WebSocket — ¿son lo mismo?**
R: No. Publish-subscribe es un **estilo arquitectónico** (cómo se organiza todo un sistema para que nadie dependa directamente de nadie). WebSocket es un **protocolo de transporte** concreto entre dos partes que sí se conocen, y es **simétrico** una vez abierta la conexión (cualquiera manda datos cuando quiere). Se pueden combinar: un broker puede usar WebSocket para empujar un evento en vivo a un navegador suscrito.


---

# Computacion en la Nube

**Q: ¿Qué es la computación en la nube y por qué existe?**
R: Usar recursos informáticos (cómputo, almacenamiento, red) **propiedad de un tercero**, en **sus** instalaciones, consumidos **por demanda** y pagados por uso.
Existe porque montar infraestructura propia obliga a **adivinar cuánta capacidad vas a necesitar antes de necesitarla**: si compras poco te caes el día que te va bien; si compras de más pagas hardware ocioso. La nube convierte esa apuesta en un ajuste continuo.
*Clave*: la nube es un **modelo de consumo**, no un tipo de tecnología.

**Q: Nombra las 5 características esenciales.**
R: **Auto-servicio por demanda** · **acceso ubicuo a la red** · **agrupación de recursos** (pooling) · **rápida elasticidad** · **medición del servicio**.
*Si falta alguna, no es nube.*

**Q: ¿Cuál es la característica más definitoria y por qué?**
R: **Auto-servicio por demanda** — aprovisionar sin que intervenga ninguna persona del proveedor. Es lo que cambia la escala de tiempo: de semanas de tickets a dos minutos en una consola. Si hay que abrir un ticket y esperar, no es nube por más virtualizado que esté.

**Q: ¿Qué es la agrupación de recursos (multi-tenancy) y qué desventaja trae?**
R: El proveedor tiene un pool común de recursos físicos que reparte dinámicamente entre muchos clientes. Es lo que hace la nube barata: clientes con picos en horarios distintos comparten el mismo hardware.
*Desventaja*: es la raíz de la **percepción de inseguridad** — datos confidenciales sobre infraestructura compartida con otras organizaciones.

**Q: Elasticidad vs. escalabilidad.**
R: **Escalabilidad** = la *capacidad* de crecer sin degradarse (puede ser manual, propiedad de cualquier arquitectura). **Elasticidad** = que ese ajuste sea **automático y bidireccional** — sube y también **baja** según demanda. Es propia de la nube.
*Regla*: toda nube elástica es escalable, pero no todo sistema escalable es elástico.

**Q: ¿Qué es la medición del servicio y por qué es arma de doble filo?**
R: El uso se monitorea y se factura por consumo real (*pay-per-use*). Convierte la infraestructura de **inversión** (CAPEX) en **gasto operativo** (OPEX).
*El filo malo*: un recurso olvidado encendido sigue facturando, y la **gestión de costos** se vuelve difícil de predecir y auditar — aparece como desventaja en el curso.

**Q: Nombra 4 ventajas de la nube.**
R: Reducción de costos (sin CAPEX inicial) · optimización de recursos por elasticidad · administración simplificada (el proveedor opera el hardware) · disponibilidad y acceso global · recuperación rápida · agilidad/time-to-market · seguridad de nivel empresarial.

**Q: Nombra 4 desventajas.**
R: Percepción de inseguridad (infraestructura compartida) · pérdida de control físico · dependencia del acceso a Internet · **vendor lock-in** · costo de transferencia de datos hacia afuera · complejidad de gestión de costos.

**Q: ¿Por qué el costo de salida de datos y el lock-in se refuerzan?**
R: Los proveedores cobran poco por meter datos y bastante por sacarlos. Mientras más datos acumulas, más caro migrar; mientras más caro migrar, menos poder de negociación y más servicios propietarios adoptas. Es un círculo.

**Q: ¿Cuáles son los 4 modelos de servicio y qué separa a cada uno?**
R: Cortan en puntos distintos la pila hardware → virtualización → SO → runtime → datos → aplicación:
- **IaaS** — infraestructura cruda; tú administras SO, runtime y app. *AWS EC2, S3.*
- **PaaS** — plataforma lista; tú solo pones **código y datos**. *Heroku, Google App Engine.*
- **SaaS** — aplicación completa lista para usar; solo la consumes. *Gmail, Salesforce.*
- **FaaS** — funciones sueltas disparadas por eventos, sin gestionar servidor. *AWS Lambda.*
*Regla*: a más "aaS", **más administra el proveedor y menos control te queda**.

**Q: ¿Qué sigue siendo tuyo en IaaS que ya no lo es en PaaS?**
R: El **sistema operativo** (y el runtime). En IaaS, si sale un parche de seguridad de Ubuntu lo aplicas tú; en PaaS eso lo hace la plataforma. Esa es exactamente la frontera entre los dos.

**Q: PaaS vs. FaaS.**
R:
| | PaaS | FaaS |
|---|---|---|
| Despliegas | Una app completa | Una función suelta |
| Cuando nadie la usa | La app sigue viva (y facturando) | No corre nada |
| Cobro | Por tiempo activo | Por invocación y duración |
| Estado | Puede mantenerlo en memoria | Sin estado entre invocaciones |

**Q: ¿"Serverless" significa que no hay servidores?**
R: **No.** Significa que no los **ves ni los administras** — el proveedor los levanta cuando llega el evento y los destruye después. Tiene el problema del *cold start*: la primera invocación tras un rato de inactividad tarda más porque hay que levantar el entorno.

**Q: ¿Cuáles son los 4 modelos de despliegue?**
R: **Privada** (uso exclusivo de una organización) · **pública** (proveedor externo compartido: AWS, Azure, GCP) · **híbrida** (privada + pública conectadas) · **multicloud** (varias públicas a la vez).

**Q: ¿Nube privada es lo mismo que on-premise?**
R: **No.** Lo que la hace privada es que la infraestructura sea de **uso exclusivo** de una organización, no dónde está el rack. Puede estar alojada y operada por un tercero en el datacenter del tercero y seguir siendo privada.

**Q: Híbrida vs. multicloud.**
R: **Híbrida** = privada **+** pública, para cumplir regulación sin perder elasticidad. **Multicloud** = varias **públicas** entre sí, para no depender de un proveedor.
La híbrida necesita nube privada por definición; la multicloud no. Pueden darse juntas.

**Q: ¿Quién administra qué? (on-premise → FaaS)**
R: El cliente va soltando capas de abajo hacia arriba:
- **On-premise**: todo.
- **IaaS**: deja hardware y virtualización; conserva SO, runtime, datos, app.
- **PaaS**: deja también SO y runtime; conserva datos y app.
- **SaaS**: no conserva nada, solo consume.
- **FaaS**: solo el **código de la función**.
*Analogía de la pizza*: on-premise la haces en casa · IaaS compras ingredientes · PaaS pides domicilio · SaaS vas al restaurante · FaaS pagas por porción.

**Q: ¿Cuáles son las razones estratégicas para migrar a la nube?**
R: **Time-to-market más rápido** (la número uno en la práctica) · reduce **CAPEX** · aumenta **ROI** · menos personal de TI en operación · uso eficiente de recursos · aprovisionamiento rápido y escalabilidad elástica.

**Q: CAPEX vs. OPEX.**
R: **CAPEX** = inversión inicial en un activo (comprar servidores); se paga antes de saber si se necesitaba. **OPEX** = gasto recurrente por operar (la factura mensual de la nube); se ajusta al uso real.


---

# Redes de Datos

**Q: ¿Qué se necesita para enviar un paquete de un host a otro?**
R: **IP** (identificar el destino) · **máscara** (saber si está en mi red) · **gateway** (salida si está afuera) · **DNS** (si usé un nombre) · **medio de transmisión**.
*No es una lista suelta*: es el razonamiento encadenado que hace tu máquina — ¿me dieron nombre? → DNS → ¿está en mi red? → máscara → si no → gateway.

**Q: ¿Qué es la máscara de subred y para qué sirve exactamente?**
R: Divide la IP en parte de **red** y parte de **host**. Sirve para responder la pregunta que tu máquina se hace antes de mandar cualquier cosa: *¿el destino está en mi red o afuera?* De eso depende si le habla directo o se lo entrega al router.
*Ejemplo*: 192.168.1.15 con /24 → red = 192.168.1. El destino 192.168.1.20 está en la misma red (directo); 192.168.5.20 no (al gateway).

**Q: ¿Cuántos hosts usables tiene una /24? ¿Y por qué no 256?**
R: **254**. Son 2⁸ = 256 direcciones, pero se restan **2**: una identifica la red y otra es el broadcast.
*Otras*: /25 = 126 usables · /16 = 65.534 usables.

**Q: ¿Qué es el gateway?**
R: La IP del **router** — la salida de tu red hacia otras redes. Si la máscara determina que el destino no es local, el paquete se le entrega a él, y él lo reenvía saltando de router en router.
*Analogía*: la máscara es saber si el destinatario vive en tu edificio; si no, se lo dejas al portero.

**Q: NAT vs. DHCP (se confunden porque ambos viven en el router).**
R: **DHCP asigna**, **NAT traduce**.
| | DHCP | NAT |
|---|---|---|
| Qué hace | Asigna IP, máscara, gateway y DNS | Traduce privadas → una pública |
| Cuándo | Al conectarse el dispositivo | Cada vez que un paquete sale/entra |
| Sin él | Configurar cada equipo a mano | Los privados no salen a Internet |

**Q: ¿Por qué existe NAT y qué rompe?**
R: Existe por el **agotamiento de IPv4**: el ISP te da una IP pública pero tienes 15 dispositivos. El router reemplaza la IP privada de origen por la suya pública y lleva una tabla para devolver cada respuesta.
**Rompe el entrar**: como los equipos internos no tienen dirección visible desde fuera, nadie de Internet puede iniciar conexión hacia ellos. Para alojar un servidor hace falta *port forwarding* y una IP pública estática.

**Q: ¿Cuáles son los rangos de IP privadas?**
R: **10.0.0.0/8** · **172.16.0.0/12** · **192.168.0.0/16**. No son enrutables en Internet. Todo lo demás es pública.
*Por qué existen*: permiten que millones de redes locales reutilicen las mismas direcciones sin conflicto, porque ninguna sale a Internet con ellas.

**Q: IPv4 vs. IPv6.**
R: **IPv4** = 32 bits (~4.300 millones, formato 192.168.1.15). **IPv6** = 128 bits (prácticamente inagotable, formato 2001:db8::15). IPv6 nació **porque IPv4 se agotó**.
*Pregunta trampa*: si IPv4 se agotó, ¿por qué Internet sigue? Por **NAT**, que le compró tiempo.

**Q: ¿Cuándo necesitas una IP pública estática?**
R: Cuando otros tienen que **encontrarte a ti**: alojar un servidor web, de correo o una VPN. Si solo navegas y consumes servicios, te basta una dinámica con NAT.

**Q: ¿Qué es DNS y por qué es un sistema distribuido?**
R: Traduce nombres a IPs (`google.com` → 142.250.x.x). Es la guía telefónica de Internet.
Es distribuido porque es **jerárquico y repartido**: ningún servidor conoce todos los dominios; cada uno sabe de su zona y a quién preguntarle por el resto, y entre todos actúan como un servicio único. Usa **particionamiento** (zonas) y **caché**.

**Q: ¿Qué es un puerto y por qué existe?**
R: La IP identifica la **máquina**, pero una máquina corre muchos servicios. El puerto identifica **a cuál** va dirigido el paquete.
*Analogía*: IP = dirección del edificio; puerto = número del apartamento.
**22** SSH · **53** DNS · **80** HTTP · **443** HTTPS · **3306** MySQL · **5432** PostgreSQL.

**Q: ¿Qué es un firewall y dónde puede vivir?**
R: Sistema que **permite o bloquea** conexiones según reglas (IP origen/destino, puerto, protocolo). Puede estar en el computador (Windows Defender, UFW), en el router, o en la nube (**AWS Security Groups**).
*Ejemplo de regla típica*: entrada solo por 80 y 443 desde cualquier IP, y por 22 solo desde la IP de la oficina.

**Q: ¿Qué es HTTP y qué significa que sea stateless?**
R: Protocolo cliente-servidor de petición-respuesta para transferir recursos. El cliente manda un **método** (GET, POST, PUT, PATCH, DELETE) sobre una URL; el servidor responde con un **código** (2xx éxito, 3xx redirección, 4xx error del cliente, 5xx error del servidor).
**Stateless** = cada petición es independiente; el servidor no recuerda la anterior por sí solo.

**Q: ¿Por qué ser stateless es una ventaja, y qué cuesta?**
R: **Ventaja**: cualquiera de los servidores tras un balanceador puede atender cualquier petición → habilita la **escalabilidad horizontal**. Si el servidor guardara tu sesión en memoria, tendrías que volver siempre al mismo.
**Costo**: hay que simular el estado con **cookies, tokens o sesiones** que el cliente reenvía en cada petición.

**Q: HTTP vs. HTTPS.**
R: HTTPS = HTTP + cifrado **TLS**. Puerto **443** en vez del 80. Protege **confidencialidad** (nadie lee) e **integridad** (nadie altera en el camino).

**Q: ¿Qué es el triángulo CIA?**
R: Los tres **objetivos** de seguridad: **Confidencialidad** (solo quien debe, ve) · **Integridad** (nadie altera sin permiso) · **Disponibilidad** (está cuando se necesita).
*Son tres porque entran en conflicto*: cifrar todo y apagar el servidor da confidencialidad perfecta y disponibilidad nula. Diseñar seguridad es equilibrarlos.

**Q: ¿Qué es la triada AAA y cómo se relaciona con CIA?**
R: Los tres pasos del control de acceso: **Autenticación** (quién eres) · **Autorización** (qué puedes hacer) · **Auditoría** (qué hiciste).
**AAA es el mecanismo con el que se logran los objetivos del CIA.**
*Error clásico de examen*: **CIA = qué se protege. AAA = cómo se controla el acceso.** No los intercambies.

**Q: Autenticación vs. autorización.**
R: Autenticación = la portería confirma que **eres tú**. Autorización = qué **puertas abre** tu tarjeta ya adentro. Estar autenticado no implica poder hacer todo: un usuario válido puede tener prohibido borrar registros.

**Q: Escalado vertical vs. horizontal.**
R: **Vertical** = más CPU/RAM a la misma máquina; simple, pero con **techo físico** y punto único de falla. **Horizontal** = más máquinas; sin techo y con redundancia, pero convierte el sistema en distribuido y hereda todos sus problemas.

**Q: ¿Cómo se mide el rendimiento? Latencia vs. throughput.**
R: **Tiempo de respuesta**, **latencia** (retraso de la comunicación) y **throughput** (solicitudes por segundo).
No son lo mismo y pueden ir opuestos: un sistema puede atender 10.000 peticiones/s (throughput alto) tardando 2 s cada una (latencia mala), si las procesa en lotes.

**Q: ¿Cuánto es 99,9 % de disponibilidad y qué no garantiza?**
R: **≈ 8,76 horas de caída al año** ("tres nueves"). *(99 % ≈ 3,65 días · 99,99 % ≈ 52,6 min.)*
**No garantiza** rendimiento, seguridad ni datos correctos: puede estar arriba y responder lento o corrupto.

**Q: ¿Cuáles son los 3 pilares de observabilidad?**
R: **Logs** (eventos puntuales: qué pasó) · **métricas** (valores en el tiempo: cuánto y cuándo) · **trazas** (recorrido de una petición entre servicios: por dónde pasó y dónde se demoró).
*Por qué importan aquí*: con una petición que cruza 8 servicios, sin trazas es imposible saber cuál falló.

**Q: ¿Qué es IaC y qué problema resuelve?**
R: *Infrastructure as Code* — administrar infraestructura con **código** en vez de configurarla a mano (Terraform, Ansible, CloudFormation, Pulumi).
Resuelve que la configuración manual no es repetible, no queda registrada y depende de que alguien recuerde qué tocó. Da automatización, repetibilidad y menos error humano — y es lo que hace posible la **recuperación rápida** de la nube: reconstruir todo es volver a correr el script.


---

## Repasada

- [ ] 24 h · [ ] 7 dias · [ ] 30 dias

## Enlaces

- [[01 Guia de Repaso - Parcial 1]]
- [[07 Preguntas de Repaso - Parcial 1]]
- [[Arquitecturas de Nube y Sistemas Distribuidos MOC]]
