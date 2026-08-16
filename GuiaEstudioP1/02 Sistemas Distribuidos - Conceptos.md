---
tags: [estudio/nube, semestre-6]
created: 2026-08-10
updated: 2026-08-12
materia: Arquitecturas de Nube y Sistemas Distribuidos
---

# Sistemas Distribuidos - Conceptos

Esta nota desarrolla la unidad de fundamentación del curso. Es la base de todo lo demás: comunicación, sincronización y sistemas de archivos distribuidos son consecuencias de lo que está acá.

---

## Qué es un sistema distribuido

**Qué es**: una colección de computadoras autónomas, conectadas por red, que se coordinan mediante el paso de mensajes y que se presentan ante el usuario como **un solo sistema coherente**.

Las dos mitades de la definición importan por igual:
- **Autónomas** — cada nodo tiene su propio procesador, su propia memoria y su propio reloj. Ninguno manda sobre el otro por hardware.
- **Un solo sistema coherente** — el usuario no debería notar que son muchas máquinas. Si lo nota, el sistema distribuido está mal hecho.

**Por qué existe**: porque una sola máquina choca contra tres muros que no se pueden romper comprando hardware más caro.

1. **Muro físico**: hay un límite a cuánta CPU y RAM cabe en un servidor, y el precio crece mucho más rápido que la potencia. Duplicar la capacidad de un servidor grande cuesta bastante más que el doble.
2. **Muro geográfico**: si tus usuarios están en Bogotá, Madrid y Tokio, ningún servidor único les da baja latencia a todos. La velocidad de la luz no se negocia.
3. **Muro de disponibilidad**: una máquina sola es un punto único de falla. Si se cae, se cayó todo el servicio.

**Cómo funciona**: varios nodos independientes se reparten el trabajo y los datos. Como no comparten memoria, se coordinan **enviándose mensajes por la red**. Encima suele ir una capa de *middleware* que oculta esa complejidad y le da al programador la ilusión de un sistema único.

**Ejemplo concreto**: cuando buscas algo en Google, tu consulta la atienden cientos de máquinas: unas resuelven el DNS, otras balancean la carga, otras buscan en fragmentos distintos del índice, otras arman el resultado. Tú ves una caja de texto y una lista de enlaces — un solo sistema.

**Se confunde con**:

| | Sistema distribuido | Sistema paralelo (multiprocesador) | Sistema centralizado |
|---|---|---|---|
| Memoria | Cada nodo la suya | Compartida entre procesadores | Una sola |
| Comunicación | Mensajes por red | Lectura/escritura en memoria común | No aplica |
| Relojes | Uno por nodo, desincronizados | Uno solo | Uno solo |
| Falla de un componente | El resto puede seguir | Se cae todo | Se cae todo |

**Error típico en examen**: creer que "muchos usuarios conectados a un servidor" es un sistema distribuido. No lo es — eso es un sistema centralizado con muchos clientes. Lo distribuido está en que el **servicio mismo** corre repartido en varias máquinas autónomas.

---

## Las 4 consecuencias de distribuir

Esta es la sección más importante de la unidad. **Todo lo difícil de los sistemas distribuidos sale de aquí.** Si entiendes estas cuatro, el resto del semestre se deduce.

### 1. Carencia de reloj global

**Qué es**: no existe un reloj compartido. Cada máquina tiene su propio oscilador de cuarzo, y todos se desvían unos milisegundos por día, cada uno para su lado.

**Por qué importa**: sin reloj común, **no puedes decidir cuál de dos eventos ocurrió primero** si pasaron en máquinas distintas. Y "quién fue primero" es justo lo que necesitas para resolver conflictos.

Tampoco se arregla mandando la hora por la red: el mensaje "son las 10:00:00" tarda un tiempo desconocido y variable en llegar. Cuando llega, ya es otra hora, y no sabes exactamente cuánto pasó.

**Ejemplo**: dos personas editan el mismo documento compartido. El servidor A registra su cambio a las 10:00:05 y el servidor B registra el otro a las 10:00:03. Parecería que gana A por ser posterior. Pero si el reloj de B iba 4 segundos atrasado, el cambio de B fue realmente a las 10:00:07 — el último fue B. Si el sistema resuelve por hora de pared, **guarda la versión equivocada y se pierde trabajo real**.

**Cómo se enfrenta**: con **relojes lógicos** (relojes de Lamport, vectores de versión), que no miden tiempo sino **causalidad**: en vez de "cuándo pasó", registran "qué evento pudo haber influido en cuál".

**Error típico**: decir que NTP (Network Time Protocol) resuelve el problema. NTP *reduce* la desviación a unos pocos milisegundos, pero nunca la elimina, y sigue habiendo incertidumbre. Para eventos separados por microsegundos, seguir confiando en la hora de pared produce decisiones incorrectas.

### 2. Fallas independientes (fallas parciales)

**Qué es**: cada componente puede fallar por su cuenta, mientras los demás siguen funcionando.

**Por qué importa**: es la diferencia mental más grande con un sistema centralizado. En una máquina sola, el sistema **funciona o no funciona** — es binario. En uno distribuido existe un tercer estado, el peligroso: **funciona a medias**, y hay que programar explícitamente para eso.

Y hay algo peor. Cuando un nodo no responde, **no puedes distinguir entre tres causas distintas**, porque desde afuera se ven exactamente iguales:
- el nodo se cayó,
- el nodo está vivo pero lento,
- el nodo está bien pero se cortó la red entre los dos.

Las tres se ven como "silencio". Y cada una pediría una reacción diferente.

**Ejemplo**: tu aplicación llama al servicio de pagos y el llamado expira por timeout. ¿Se cobró o no se cobró? No lo sabes. Si reintentas, puedes cobrarle dos veces al cliente. Si no reintentas, puedes perder el pago. Ninguna opción es segura con la información que tienes.

**Cómo se enfrenta**: con *timeouts* explícitos, reintentos, y sobre todo **operaciones idempotentes** — diseñadas para que ejecutarlas dos veces tenga el mismo efecto que ejecutarlas una vez (por ejemplo, mandando un identificador único de transacción que el servidor reconoce y no vuelve a procesar).

**Error típico**: asumir que si no hubo respuesta, la operación no se ejecutó. Puede haberse ejecutado perfectamente y haberse perdido solo la respuesta de vuelta.

### 3. Ausencia de memoria compartida

**Qué es**: los nodos no comparten un espacio de memoria físico. Nada de lo que un nodo escribe en su RAM es visible para otro.

**Por qué importa**: dentro de una máquina, dos hilos comparten memoria — uno escribe una variable y el otro la ve de inmediato. Entre máquinas eso es imposible. Todo dato "compartido" es en realidad **una copia**, y toda copia puede estar desactualizada.

**Ejemplo**: una tienda tiene dos servidores atendiendo pedidos, y cada uno guarda en memoria que quedan 5 unidades de un producto. Llegan dos compras simultáneas de 5 unidades, una a cada servidor. Los dos consultan su copia, ven 5, y aprueban la venta. Vendiste 10 unidades de un inventario de 5.

**Cómo se enfrenta**: con una única fuente de verdad (una base de datos con transacciones), o con algoritmos de consenso cuando ni siquiera se quiere depender de un solo punto.

**Error típico**: confundir "memoria compartida distribuida" (una abstracción de software que *simula* memoria común pasando mensajes por debajo) con memoria compartida real. La abstracción existe, pero paga el costo de la red y no elimina el problema.

### 4. Concurrencia

**Qué es**: todos los procesos del sistema se ejecutan al mismo tiempo, sin un orden global que los organice.

**Por qué importa**: en un programa secuencial, las instrucciones ocurren en un orden conocido. En un sistema distribuido todo pasa a la vez y en máquinas distintas, así que aparecen condiciones de carrera que no se pueden reproducir fácilmente para depurarlas: el error aparece un día de cada mil, cuando dos mensajes llegan en el orden "malo".

**Ejemplo**: dos usuarios retiran plata de la misma cuenta con saldo de $100 al mismo tiempo, $80 cada uno, atendidos por nodos distintos. Ambos nodos verifican "¿hay saldo?", ambos ven $100, ambos aprueban. La cuenta queda en -$60.

**Cómo se enfrenta**: con exclusión mutua distribuida, transacciones, y bloqueos coordinados.

### Corolario: no hay estado global perceptible

De las cuatro anteriores sale una consecuencia que vale la pena tener aparte, porque es muy preguntable:

**Nunca puedes tomarle una "foto" al sistema completo.** Para saber el estado global tendrías que preguntarle a todos los nodos, pero las respuestas llegan en momentos distintos — cuando el nodo 5 te contesta, lo que te dijo el nodo 1 ya cambió. El estado global que reconstruyes **nunca existió realmente** en ningún instante.

---

## Retos de los sistemas distribuidos

Cada reto es la forma práctica que toma alguna de las cuatro consecuencias:

- **Comunicación entre procesos** — cómo se mandan mensajes de forma confiable sobre una red que no lo es.
- **Coordinación** — cómo se ponen de acuerdo procesos que no comparten reloj ni memoria.
- **Concurrencia** — cómo se controla el acceso simultáneo sin bloquear el sistema entero.
- **Naming** — cómo se localiza un recurso sin saber en qué máquina está. DNS es el ejemplo clásico.
- **Datos** — consistencia, replicación y transacciones cuando hay copias en varios sitios.
- **Tolerancia a fallas** — seguir operando con componentes caídos.
- **Seguridad** — la superficie de ataque crece: ahora los mensajes viajan por una red que otros pueden leer o alterar.

---

## Metas de diseño

Son los objetivos que persigue quien diseña el sistema. En examen suelen pedir nombrarlas y explicar una.

### Heterogeneidad

**Qué es**: la capacidad de funcionar sobre componentes distintos entre sí — distinto hardware, distinto sistema operativo, distinto lenguaje de programación, distinto tipo de red.

**Por qué existe**: ningún sistema real se construye de una sola vez con máquinas idénticas. Crece por partes, con hardware comprado en años distintos y servicios escritos en lenguajes distintos por equipos distintos.

**Cómo se logra**: con protocolos y formatos de datos acordados (HTTP, JSON, gRPC) que ambos lados entiendan sin importar en qué estén escritos, y con middleware que traduce.

**Ejemplo**: un servicio en Java le pide datos a uno en Python mandando JSON por HTTP. Ninguno sabe ni le importa en qué está escrito el otro.

### Openness (apertura)

**Qué es**: que el sistema se pueda extender y reimplementar — agregarle componentes nuevos, o reemplazar uno existente por otra implementación, sin rehacerlo todo.

**Por qué existe**: un sistema cerrado se vuelve imposible de evolucionar, y queda amarrado a un solo proveedor.

**Cómo se logra**: **publicando las interfaces**. Si el contrato de un componente está documentado y es estable, cualquiera puede escribir otra implementación que lo cumpla.

**Ejemplo**: cualquier servidor que implemente el protocolo DNS puede reemplazar al que tienes, porque la interfaz es pública y estándar.

### Seguridad

**Qué es**: proteger los recursos que el sistema expone. Se organiza con el **triángulo CIA**: Confidencialidad (solo quien debe, ve), Integridad (nadie altera sin permiso), Disponibilidad (está cuando se necesita).

**Por qué existe con más fuerza aquí**: en un sistema centralizado los datos no salen de la máquina. Al distribuir, cada mensaje viaja por una red donde alguien puede escucharlo, modificarlo o suplantarte.

**Cómo se logra**: cifrado en tránsito (TLS), autenticación de ambos extremos, y control de acceso — la triada **AAA**: Autenticación, Autorización, Auditoría.

### Escalabilidad

**Qué es**: la capacidad de crecer sin degradarse. El curso la divide en tres dimensiones:
- **En tamaño** — más usuarios y más recursos.
- **Geográfica** — nodos más lejos entre sí.
- **Administrativa** — más organizaciones distintas involucradas, cada una con sus políticas.

**Por qué es difícil**: porque crecer introduce cuellos de botella. Si todo pasa por un componente central (un servidor, una base de datos, una tabla), ese componente se satura y el sistema deja de escalar por más máquinas que agregues.

**Técnicas para lograrla** (las tres del curso):
- **Ocultar latencias** — no quedarse esperando la respuesta; trabajar de forma asíncrona mientras llega.
- **Ocultar la distribución mediante particionamiento** — partir los datos o el trabajo entre nodos, de modo que cada uno atienda solo su parte (así funciona DNS: cada servidor sabe de su zona, no de todo Internet).
- **Ocultar la replicación mediante caché** — tener copias cerca de quien las usa para no ir hasta el origen cada vez.

**Ejemplo**: una CDN acerca las imágenes de un sitio a servidores en cada país. El usuario de Tokio no cruza el océano por cada foto: la recibe de un caché local.

**Se confunde con**:

| | Escalabilidad | Elasticidad | Rendimiento |
|---|---|---|---|
| Qué es | Capacidad de crecer sin degradarse | Que ese ajuste sea automático y en ambos sentidos | Qué tan rápido responde ahora |
| Se mide en | Cuánta carga soporta antes de romperse | Qué tan rápido se adapta a la demanda | Latencia, throughput |
| Ejemplo | Puedo pasar de 10 a 1000 servidores | Subo a 1000 el viernes y bajo a 10 el domingo, solo | Responde en 200 ms |

### Manejo de fallas (dependability)

**Qué es**: la capacidad de convivir con fallas. Se descompone en cuatro tareas concretas:
- **Detectar** la falla (heartbeats, timeouts).
- **Enmascararla** para que el usuario no la vea (reintentar contra una réplica).
- **Tolerarla** — seguir dando servicio, aunque sea degradado.
- **Recuperarse** — volver al estado bueno cuando el componente regresa.

**Cómo se logra**: fundamentalmente con **redundancia** — tener más de una copia de todo lo que importa, para que la caída de una no detenga el servicio.

**Ejemplo**: Netflix con un servidor de recomendaciones caído no muestra un error; muestra una lista genérica. Degradó el servicio en vez de negarlo. Eso es tolerar la falla, no solo detectarla.

### Concurrencia

Ya desarrollada arriba como consecuencia. Como *meta de diseño*, el objetivo es que varios usuarios compartan recursos simultáneamente sin corromper datos y sin serializar todo (porque bloquear todo mata el rendimiento y arruina la escalabilidad).

### Transparencia

Es la meta más preguntada, y tiene su propia sección abajo.

---

## Transparencia

**Qué es**: ocultar al usuario y al programador el hecho de que el sistema está distribuido. Cada *tipo* de transparencia oculta un aspecto distinto.

**Por qué existe**: sin ella, cada programador tendría que saber en qué máquina vive cada dato, qué pasa si se mueve, y qué hacer si una réplica se cae. Sería imposible construir nada grande.

Los ocho tipos, cada uno con lo que oculta:

| Tipo | Qué oculta | Ejemplo |
|---|---|---|
| **Acceso** | Que el recurso sea local o remoto se usa igual | Abrir un archivo en red con el mismo código que uno del disco |
| **Ubicación** | Dónde está físicamente el recurso | `google.com` no te dice en qué datacenter estás entrando |
| **Migración** | Que el recurso se mueva de lugar | Tu buzón de correo cambia de servidor y tú ni te enteras |
| **Reubicación** | Que se mueva **mientras lo estás usando** | Sigues viendo una película mientras el servicio cambia el servidor que te la envía |
| **Replicación** | Que existan varias copias | Pides una imagen y no sabes cuál de las 40 copias de la CDN te la dio |
| **Concurrencia** | Que otros usen el mismo recurso a la vez | Dos personas compran en la misma tienda sin verse ni estorbarse |
| **Fallo** | Que algo se cayó y se recuperó | Un nodo muere, otro toma su lugar, tú no ves error |
| **Persistencia** | Si el recurso está en memoria o en disco | No sabes si el dato vino de un caché en RAM o del disco |

**Los tres contrastes que más se preguntan**:

- **Acceso vs. ubicación** — acceso es que lo *usas igual* sin importar dónde esté; ubicación es que *no sabes dónde está*. Puedes tener una sin la otra: una URL con IP fija te da transparencia de acceso pero no de ubicación.
- **Migración vs. reubicación** — migración es que el recurso se mueva *entre usos*; reubicación es que se mueva *durante* el uso, sin cortar la sesión. La reubicación es bastante más difícil.
- **Fallo vs. tolerancia a fallas** — la tolerancia es el mecanismo (redundancia, reintentos); la transparencia de fallo es el *resultado* de que el usuario no lo perciba.

**Error típico**: pensar que más transparencia siempre es mejor. No lo es. Ocultar completamente la distribución tiene un costo, y a veces conviene que el programador **sepa** que una llamada es remota, justamente para que la trate con cuidado (timeouts, reintentos). Ocultarlo del todo lleva a la primera falacia de la lista de abajo.

---

## Las falacias de la computación distribuida (Peter Deutsch)

**Qué son**: ocho suposiciones falsas que hacen los programadores al escribir sistemas distribuidos, casi siempre sin darse cuenta. Se llaman falacias porque cada una parece razonable y todas son mentira.

**Por qué existen**: porque programar en una sola máquina te acostumbra a que la comunicación sea instantánea, gratis y confiable. Al pasar a la red, esos hábitos se convierten en errores de diseño.

| # | Falacia | Qué pasa si la crees |
|---|---|---|
| 1 | La red es confiable | No manejas reintentos; el primer paquete perdido tumba la operación |
| 2 | La latencia es cero | Haces 200 llamadas remotas donde cabía una; funciona en tu PC y se arrastra en producción |
| 3 | El ancho de banda es infinito | Mandas objetos enormes en cada petición y saturas el enlace |
| 4 | La red es segura | No cifras ni autenticas; cualquiera en el camino lee o altera los mensajes |
| 5 | La topología no cambia | Quemas direcciones IP en el código; el día que se mueve un servidor, todo se rompe |
| 6 | Hay un solo administrador | Asumes que alguien puede cambiarlo todo a la vez; en realidad hay que coordinar equipos y empresas distintas |
| 7 | El costo de transporte es cero | Ignoras que serializar y mover datos cuesta CPU y dinero (las nubes cobran por transferencia) |
| 8 | La red es homogénea | Asumes que todos los enlaces y equipos se comportan igual; en la práctica conviven tecnologías distintas |

> El apunte de clase lista además "el cluster se usa para agilizar" — es la misma idea de fondo: suponer que agregar nodos acelera automáticamente, ignorando el costo de coordinarlos.

**Ejemplo de la más cara (la 2)**: un desarrollador escribe una pantalla que, por cada uno de los 100 productos que muestra, hace una llamada al servicio de precios. En su máquina, con todo local, tarda 50 ms. En producción, con 40 ms de latencia por llamada, tarda 4 segundos. El código no tiene ningún error — la suposición sí.

**Error típico**: memorizar las ocho sin poder explicar ninguna. En examen casi siempre piden **explicar el efecto**, no recitar la lista.

---

## Clasificación de Flynn

**Qué es**: una taxonomía de arquitecturas de computación que las ordena según dos ejes — cuántos **flujos de instrucciones** y cuántos **flujos de datos** maneja la máquina al mismo tiempo.

**Por qué existe**: para poder hablar de "paralelismo" con precisión. Antes de Flynn, "computador paralelo" significaba cosas muy distintas según quién lo dijera.

| | Un flujo de datos | Varios flujos de datos |
|---|---|---|
| **Una instrucción** | **SISD** | **SIMD** |
| **Varias instrucciones** | **MISD** | **MIMD** |

### SISD (Single Instruction, Single Data)
Una instrucción sobre un dato a la vez: el computador secuencial clásico. Es el modelo mental con el que aprendiste a programar. **Ejemplo**: una CPU de un solo núcleo ejecutando un programa normal.

### SIMD (Single Instruction, Multiple Data)
**La misma** instrucción aplicada simultáneamente a **muchos** datos. Sirve cuando hay que hacer exactamente la misma operación sobre un montón de elementos. **Ejemplo**: una GPU subiéndole el brillo a una imagen — la operación "suma 10" es una sola, pero se aplica a los 2 millones de píxeles a la vez. Por eso las GPUs sirven tan bien para gráficos y para redes neuronales.

### MISD (Multiple Instruction, Single Data)
Varias instrucciones distintas sobre el mismo dato. Casi no tiene ejemplos prácticos; se cita en sistemas de altísima confiabilidad donde varios procesadores distintos calculan lo mismo de formas diferentes para comparar resultados (control de vuelo redundante). **En examen basta con saber que es la categoría rara y por qué.**

### MIMD (Multiple Instruction, Multiple Data)
Procesadores distintos ejecutando programas distintos sobre datos distintos. **Aquí caen los sistemas distribuidos**, y también los multiprocesadores y multicomputadores. Es la categoría más general y la más importante para el curso.

### La subdivisión de MIMD

MIMD se parte según **cómo se comunican** los procesadores:

**Multiprocesadores — memoria compartida**
Todos los procesadores acceden a una misma memoria física. Se comunican escribiendo y leyendo en ella, a velocidad de memoria. Son **fuertemente acoplados**: el tiempo de comunicación es menor que el de procesamiento (T_comm < T_proc).
- **UMA** (Uniform Memory Access): todos tardan lo mismo en llegar a cualquier posición de memoria.
- **NUMA** (Non-Uniform Memory Access): cada procesador tiene memoria "cercana" (rápida) y "lejana" (lenta).

**Multicomputadores — memoria distribuida**
Cada nodo tiene su propia memoria y se comunican por red. Son **débilmente acoplados**: comunicarse cuesta mucho más que procesar (T_comm >> T_proc).
- **MPP** (Massively Parallel Processing): muchos nodos con red interna muy rápida y dedicada.
- **Clusters**: computadores comunes conectados por red estándar. Es lo que hay detrás de la mayoría de los servicios en la nube.

### Fuertemente vs. débilmente acoplado

| | Fuertemente acoplado | Débilmente acoplado |
|---|---|---|
| Memoria | Compartida | Distribuida (cada nodo la suya) |
| Conexión | Bus interno / tarjetas | Red |
| Retraso de mensajes | Muy bajo | Alto |
| Volumen de datos intercambiado | Alto | Bajo (conviene minimizarlo) |
| Velocidad de intercambio | De memoria | De red |
| Uso típico | Cómputo paralelo intensivo | **Sistemas distribuidos** |

**Error típico y muy común en examen**: asociar "sistemas distribuidos" con fuertemente acoplado porque suena a "muy conectado". Es al revés: los sistemas distribuidos son **débilmente acoplados**. Esa independencia entre nodos es precisamente lo que les permite escalar y tolerar fallas — pero también lo que trae las cuatro consecuencias del principio de esta nota.

---

## Comunicación en sistemas distribuidos

Como no hay memoria compartida, la comunicación **es** el sistema. Involucra dos cosas a la vez: transferir datos y sincronizar a los procesos.

**Enfoques**:
- **Paso de mensajes** — explícito: un proceso envía, otro recibe. Es lo más básico y lo más honesto: se ve que hay una red de por medio.
- **RPC (Remote Procedure Call)** — llamada a procedimiento remoto. Hace que invocar una función en otra máquina se *vea* igual que llamar una función local; la librería se encarga de empaquetar los argumentos, mandarlos y devolver el resultado. Da transparencia de acceso, pero cuidado: esconde que la llamada puede tardar o fallar, y ahí es donde muerden las falacias de Deutsch.

**Modelos**:
- **Par a par (punto a punto)** — un emisor, un receptor.
- **Comunicación grupal (multicast)** — un emisor, varios receptores. Es lo que se usa para mantener réplicas sincronizadas.

---

## Estilos arquitectónicos: Cliente-Servidor y Publish-Subscribe

### Qué es una arquitectura de software

**Qué es**: la organización fundamental de un sistema, definida por sus **componentes**, las **relaciones entre esos componentes** (entre sí y con el entorno), y los **principios que guían su diseño** — las reglas y restricciones con las que se construyen los bloques y los conectores. Es la definición que usa el curso y que retoma Tanenbaum.

**Por qué importa**: la arquitectura no es un detalle de implementación que se arregla después. Se decide al principio y **fija de antemano** qué tan fácil va a ser escalar el sistema, tolerar fallas y hacerlo evolucionar. Elegir mal la arquitectura es una deuda que se paga durante todo el proyecto, no un bug que se corrige con un parche.

**Cómo se describe**: por sus **propiedades estructurales** — la organización lógica y física de los componentes distribuidos y cómo interactúan. Dos sistemas pueden hacer exactamente lo mismo y tener arquitecturas completamente distintas si organizan esa interacción de forma diferente.

El curso contrasta dos estilos que resuelven de forma opuesta la misma pregunta — "¿cómo se hablan los componentes entre sí?": **cliente-servidor**, donde los roles y la comunicación son directos y fijos, y **publish-subscribe**, donde nadie le habla directamente a nadie.

### Arquitectura Cliente-Servidor

**Qué es**: estilo en el que los componentes se dividen en dos roles — el **cliente**, que solicita un servicio, y el **servidor**, que lo procesa y responde.

**Por qué existe / qué problema resuelve**: es la forma más simple de organizar quién pide y quién resuelve. Centralizar los datos y la lógica en el servidor elimina la ambigüedad de quién tiene la verdad del sistema — no hay que sincronizar varias copias ni ponerse de acuerdo entre pares.

**Cómo funciona**: sigue el patrón **request-reply** (petición-respuesta). El cliente envía una solicitud; el servidor la procesa y devuelve un resultado. La conexión que transporta esa solicitud puede ser **confiable** (garantiza entrega y orden, como TCP) o **no confiable** (sin garantías, como UDP). Lo distintivo del estilo es que el protocolo es **asimétrico**: la iniciativa es siempre del cliente. El servidor nunca abre la conversación por su cuenta — solo escucha en un puerto y responde a lo que le preguntan.

**Ejemplo concreto**: tu navegador (cliente) pide `GET /productos` a una tienda en línea (servidor). El servidor consulta su base de datos, arma el HTML y responde. Si mañana hay una oferta nueva, el servidor no puede avisarte por su cuenta — tienes que volver a preguntar tú (recargar la página), a menos que el sistema use otro mecanismo, como WebSocket, para invertir esa restricción.

**Ventajas**:
- **Administración sencilla** — datos y lógica viven en un solo lugar; actualizar el sistema es actualizar un componente, no coordinar cientos.
- **Fácil localización de fallas** — si algo falla, casi siempre es el servidor. No hay que buscar entre docenas de nodos sospechosos.

**Desventajas**:
- **Punto único de falla (SPOF)** — si el servidor cae, cae el servicio completo para todos los clientes a la vez, sin degradación parcial.
- **Escalabilidad limitada** — todos los clientes dependen del mismo punto central (o clúster detrás de un balanceador). Ese punto es exactamente el "componente central que se satura" del que ya habla la sección de escalabilidad más arriba.

**Se confunde con**: pensar que cliente-servidor es lo mismo que "centralizado" en el sentido de "una sola máquina". No lo es — puede haber un clúster entero de servidores detrás de un balanceador y el estilo sigue siendo cliente-servidor, porque lo que lo define es la **relación asimétrica de roles**, no cuántas máquinas hay del lado servidor.

**Error típico**: decir que HTTP es "simétrico" porque cliente y servidor "se mandan datos los dos". La simetría de un protocolo se prueba con una sola pregunta: **¿puede cualquiera de los dos lados iniciar la comunicación?** En HTTP, solo el cliente puede iniciar; el servidor únicamente responde a lo que ya le preguntaron. Por eso HTTP es **asimétrico**.

### Idempotencia — la pregunta que el profesor marcó para examen

**Qué es**: una operación es **idempotente** si ejecutarla una vez o muchas veces produce **el mismo resultado final**. El efecto de repetirla es idéntico al de hacerla una sola vez.

**Por qué importa en cliente-servidor**: ya viste que cuando una petición expira por timeout (sección de [[#2. Fallas independientes (fallas parciales)|fallas independientes]] arriba), no puedes saber si el servidor la procesó o solo se perdió la respuesta. Si la operación es idempotente, reintentar es seguro. Si no lo es, reintentar puede duplicar el efecto: dos cobros, dos pedidos, dos filas insertadas.

**Cómo se clasifican los métodos HTTP**:

| Método | ¿Idempotente? | Por qué |
|---|---|---|
| `GET` | Sí | Solo lee; pedirlo 1 o 100 veces deja el servidor igual |
| `PUT` | Sí | Reemplaza el recurso por un valor fijo; ponerlo dos veces da el mismo resultado que ponerlo una |
| `DELETE` | Sí | Borrar algo ya borrado no cambia nada — el estado final es "no existe", sea la 1ª o la 5ª vez |
| `POST` | **No** | Crea un recurso nuevo en cada llamada; reenviarlo crea **dos** pedidos, dos cobros |
| `PATCH` | Depende | Si modifica a un valor fijo, es idempotente; si es un incremento ("suma 1"), no lo es |

**Ejemplo concreto**: pagas una compra y el navegador se congela justo después de mandar `POST /pagos`. No sabes si se cobró. Reenviar el mismo POST puede cobrarte dos veces, porque POST no es idempotente. Por eso los sistemas de pago reales no reintentan a ciegas: mandan una **clave de idempotencia** (un identificador único de transacción) que el servidor recuerda, para que un reintento con la misma clave no duplique el cobro aunque técnicamente sea un POST.

**Error típico**: creer que "idempotente" significa "sin efectos secundarios". No es eso — `DELETE` sí tiene un efecto real (borra algo) y aun así es idempotente, porque **el efecto de repetirlo es igual al de hacerlo una vez**. Idempotencia habla del resultado de **repetir**, no de si algo cambia.

### Arquitectura Publish-Subscribe (orientada a eventos)

**Qué es**: estilo en el que los procesos no se hablan directamente. Unos **publican** eventos (publicadores) y otros se **suscriben** a los tipos de evento que les interesan (suscriptores). Un intermediario (broker o bus de eventos) reparte cada evento a todos los suscriptores interesados.

**Por qué existe / qué problema resuelve**: en cliente-servidor, el cliente tiene que **conocer** al servidor — su dirección, que esté disponible — para poder hablarle. Eso ata a los dos componentes entre sí. Publish-subscribe rompe esa atadura: ni el publicador conoce a los suscriptores ni al revés. Solo conocen al intermediario. Eso hace que agregar un componente nuevo al sistema no requiera tocar a los que ya existen.

**Cómo funciona — el desacoplamiento tiene dos dimensiones, y es lo más preguntable del tema**:
- **Desacoplamiento referencial** — publicador y suscriptor no necesitan la dirección del otro ni saber que el otro existe. Ambos solo hablan con el broker.
- **Desacoplamiento temporal** — publicador y suscriptor no necesitan estar activos al mismo tiempo. El publicador puede emitir un evento aunque el suscriptor esté apagado en ese momento; el broker lo entrega cuando el suscriptor vuelva a conectarse, si el sistema conserva los eventos (una cola).

**Ejemplo concreto**: un sistema de e-commerce publica el evento `pedido_creado` cuando alguien compra. Los servicios de inventario, facturación y envío de correos están suscritos a ese evento, sin saber que existen los otros dos, y el servicio que crea el pedido no sabe ni le importa cuántos reaccionarán. Si mañana se agrega un cuarto servicio (puntos de fidelidad), se suscribe al mismo evento sin tocar una sola línea del que publica.

**Ventajas**: ideal para sistemas **escalables y dinámicos** — se agregan o se quitan publicadores y suscriptores en caliente, sin coordinar un despliegue conjunto, porque nadie depende directamente de nadie.

**Desventajas**: se pierde la simplicidad de "pregunto y me responden ya". Depurar un flujo asíncrono que pasa por un broker y varios suscriptores es más difícil que seguir una llamada HTTP directa — necesitas **trazas** (observabilidad) para reconstruir el camino completo.

**Colas de mensajes**: una cola (como RabbitMQ, que ya aparece mencionada en la sección de retos de arriba) es una implementación concreta de este patrón: los mensajes esperan en la cola hasta que un consumidor los procesa, que es exactamente el desacoplamiento temporal en funcionamiento.

**Se confunde con WebSocket** — confusión razonable, porque los dos rompen con el modelo petición-respuesta clásico:

| | Publish-Subscribe | WebSocket |
|---|---|---|
| Qué resuelve | Desacoplar a quién **produce** de quién **consume** un evento | Mantener una conexión **persistente y bidireccional** entre dos partes concretas |
| Quiénes se conocen | Nadie conoce a nadie directamente; solo al broker | Cliente y servidor sí se conocen — abrieron una conexión directa |
| Simetría | El broker distribuye; publicador y suscriptor no se hablan entre sí | Protocolo **simétrico** una vez abierto: cualquiera de los dos manda datos cuando quiere |
| Nivel | Estilo **arquitectónico** — patrón de organización de todo un sistema | Protocolo de **transporte** concreto, capa de aplicación sobre TCP |
| Ejemplo | Un evento de pedido que disparan cuatro microservicios distintos | Un chat en vivo donde el servidor empuja mensajes sin que el cliente pregunte |

**Por qué WebSocket es la excepción a "HTTP es asimétrico"**: WebSocket arranca como una petición HTTP normal (el cliente la inicia, como siempre) pero pide **actualizar el protocolo** (`Upgrade: websocket`). Una vez aceptada, la conexión deja de comportarse como HTTP: queda abierta y **cualquiera de los dos lados** puede mandar datos cuando quiera sin que el otro pregunte primero. Es la herramienta que se usa cuando la asimetría de cliente-servidor puro se queda corta — un chat en vivo, cotizaciones en tiempo real.

**Error típico**: pensar que publish-subscribe y WebSocket son la misma cosa porque ninguno de los dos es "request-reply". Uno es un **patrón de organización** de todo un sistema (quién sabe de quién); el otro es un **protocolo de comunicación** entre dos puntos concretos. De hecho se combinan: un broker puede usar WebSocket para empujarle en vivo un evento a un navegador suscrito.

### Cliente-Servidor vs Publish-Subscribe

| | Cliente-Servidor | Publish-Subscribe |
|---|---|---|
| Quién inicia | Siempre el cliente (asimétrico) | Nadie inicia hacia el otro; ambos hablan con el broker |
| Acoplamiento | Más fuerte — el cliente necesita conocer al servidor | Débil — desacoplamiento referencial y temporal |
| Disponibilidad de las partes | Cliente y servidor deben coincidir en el tiempo, aunque sea brevemente | Publicador y suscriptor pueden no coincidir nunca |
| Punto único de falla | Sí — el servidor | Se reduce, pero se traslada al broker si este no es redundante |
| Escalabilidad | Limitada por el punto central | Alta — se agregan participantes sin coordinar despliegues |
| Ejemplo típico | Un sitio web, una API REST | Una cola de eventos, un sistema de notificaciones |

**Error típico**: creer que uno es "mejor" que el otro en general. Son herramientas para problemas distintos: cliente-servidor es más simple y más fácil de razonar cuando la respuesta se necesita **ya** (login, cargar una página). Publish-subscribe brilla cuando hay **muchos interesados** en un mismo evento y conviene no acoplarlos entre sí.

---

## Otros atributos importantes

- **Tolerancia a fallas** — seguir funcionando pese a fallos parciales, mediante redundancia de hardware y recuperación por software.
- **Confiabilidad** — que los datos transmitidos y almacenados lleguen íntegros y correctos.
- **Disponibilidad** — proporción del tiempo que el sistema está operativo. La ventaja del sistema distribuido es que la caída de un componente no inutiliza necesariamente el conjunto.
- **Reconfiguración** — poder acomodar cambios y expansiones (agregar o quitar nodos) sin detener el sistema.

**Contraste que se pregunta**: confiabilidad ≠ disponibilidad. Un sistema puede estar **disponible** (responde siempre) pero ser poco **confiable** (a veces responde datos corruptos). Y al revés: uno muy confiable en sus datos puede tener baja disponibilidad si se cae seguido.

---

## Preguntas de repaso

Respóndelas de memoria antes de mirar la nota.

1. Un compañero dice que su aplicación web con 5.000 usuarios conectados es un sistema distribuido porque "hay muchas máquinas involucradas". ¿Cómo le explicas si tiene razón o no, y qué tendría que ser cierto para que sí lo fuera?

2. Tu servicio llama al de facturación y el llamado expira. Explica por qué **no puedes saber** si la factura se generó, qué tres causas distintas producen ese mismo silencio, y qué diseño te permitiría reintentar sin riesgo.

3. Dos nodos registran un cambio sobre el mismo dato, uno a las 10:00:05 y otro a las 10:00:03. ¿Por qué resolver el conflicto tomando la hora más alta puede perder datos, y qué se usa en su lugar?

4. Una empresa quiere que su sistema soporte 10 veces más usuarios y agrega 10 veces más servidores, pero el rendimiento apenas mejora. ¿Qué está pasando probablemente, y cuáles de las tres técnicas de escalabilidad del curso aplicarías?

5. Explica la diferencia entre transparencia de migración y de reubicación con un ejemplo propio de cada una.

6. Un desarrollador escribe código que hace una llamada remota dentro de un ciclo de 500 iteraciones. Funciona perfecto en su máquina y colapsa en producción. ¿Qué falacia de Deutsch cometió y por qué el código "no tiene errores"?

7. Clasifica según Flynn: (a) una GPU aplicando un filtro a una foto, (b) un cluster de 50 servidores atendiendo peticiones web distintas, (c) tu portátil ejecutando un script de Python de un solo hilo. Para (b), ¿es fuerte o débilmente acoplado y por qué?

8. ¿Por qué se dice que en un sistema distribuido "no existe un estado global perceptible"? Explica por qué no basta con preguntarle a todos los nodos a la vez.

9. Un compañero dice que `DELETE /pedidos/5` no es idempotente porque "sí tiene un efecto: borra el pedido". ¿Está en lo correcto? Explica qué significa realmente idempotencia y clasifica GET, POST, PUT y DELETE.

10. Un sistema de e-commerce hoy llama directamente desde el servicio de pedidos a los servicios de inventario, facturación y correos (cliente-servidor entre ellos). Cada vez que se agrega un servicio nuevo hay que modificar el código del servicio de pedidos. ¿Qué estilo arquitectónico resolvería ese problema, y qué dos tipos de desacoplamiento aporta?

## Enlaces

- [[Arquitecturas de Nube y Sistemas Distribuidos MOC]]
- [[03 Computacion en la Nube]]
- [[04 Redes de Datos]]
- [[01 Guia de Repaso - Parcial 1]]
- [[02 Areas/Estudio/Semestre 6/Semestre 6 MOC]]
