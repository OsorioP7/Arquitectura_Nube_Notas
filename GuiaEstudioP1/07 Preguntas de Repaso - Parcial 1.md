---
tags: [estudio/nube, repaso, examen, semestre-6]
materia: Arquitecturas de Nube y Sistemas Distribuidos
fecha: "2026-08-12"
fecha_examen: 2026-08-19
---

# Preguntas de Repaso - Parcial 1

**Cómo usar este banco**: tapa la respuesta, contesta en voz alta o por escrito, y solo después compara. Marca con ❌ lo que falles y repásalo primero en la siguiente sesión. Responder "más o menos bien" mirando la respuesta no cuenta como saber.

Las preguntas están ordenadas por prioridad según tus niveles: primero lo más flojo.

---

## Sistemas distribuidos — las 4 consecuencias (PRIORIDAD)

### 1. Un compañero dice que su aplicación web con 5.000 usuarios conectados ya es un sistema distribuido "porque hay muchas máquinas involucradas". ¿Tiene razón?

No. Muchos **clientes** conectados a un servidor es un sistema centralizado con alta demanda, no un sistema distribuido. Lo que define a un sistema distribuido es que el **servicio mismo** corra repartido en varias computadoras **autónomas** que se coordinan por paso de mensajes, y que aun así se presente al usuario como un sistema único.

Para que sí lo fuera, tendría que tener el procesamiento o los datos repartidos entre varios nodos independientes — por ejemplo, varios servidores de aplicación detrás de un balanceador, con la base de datos replicada.

### 2. Tu servicio llama al de facturación y el llamado expira por timeout. ¿Por qué no puedes saber si la factura se generó?

Porque **tres situaciones distintas producen exactamente el mismo silencio** y desde afuera son indistinguibles:

1. El servicio de facturación se cayó antes de procesar.
2. El servicio está vivo pero lento, y va a procesar (o ya procesó).
3. El servicio procesó correctamente, pero se cortó la red al devolver la respuesta.

En los casos 2 y 3 la factura **sí se generó**. Si reintentas, la duplicas; si no reintentas, puedes perderla.

**La solución**: hacer la operación **idempotente** — enviar un identificador único de transacción que el servidor reconozca, de modo que si le llega dos veces la misma, la procese una sola vez. Con eso reintentar deja de ser riesgoso.

### 3. Dos nodos registran un cambio sobre el mismo dato: uno a las 10:00:05 y otro a las 10:00:03. ¿Por qué resolver el conflicto por "la hora más alta gana" puede perder datos?

Porque cada máquina tiene su propio reloj de cuarzo y **todos se desvían**, cada uno para su lado. Si el reloj del segundo nodo iba 4 segundos atrasado, su cambio ocurrió realmente a las 10:00:07 — fue el **último**, pero la hora de pared dice lo contrario. El sistema conservaría la versión equivocada y se perdería trabajo real del usuario.

**Qué se usa en su lugar**: relojes lógicos (Lamport, vectores de versión), que no miden tiempo sino **causalidad** — registran qué evento pudo haber influido en cuál, en vez de a qué hora pasó.

**Trampa frecuente**: decir que NTP lo resuelve. NTP reduce la desviación a milisegundos pero nunca la elimina, y la incertidumbre sigue existiendo.

### 4. ¿Por qué se dice que en un sistema distribuido "no existe un estado global perceptible"?

Porque para conocer el estado completo tendrías que preguntarle a todos los nodos, pero las respuestas llegan en momentos distintos. Cuando el nodo 5 te contesta, lo que te dijo el nodo 1 ya cambió. **El estado global que reconstruyes nunca existió simultáneamente en la realidad** — es un collage de instantes distintos.

No basta con "preguntarles a todos a la vez" porque los mensajes tardan tiempos distintos y variables en ir y volver: no hay forma de que todos respondan sobre el mismo instante.

### 5. Una tienda tiene dos servidores, cada uno con "quedan 5 unidades" en memoria. Llegan dos compras de 5 unidades simultáneas. ¿Qué pasa y qué consecuencia lo explica?

Cada servidor consulta **su copia**, ve 5 disponibles y aprueba la venta. Se venden 10 unidades de un inventario de 5.

Lo explica la **ausencia de memoria compartida**: los nodos no comparten RAM, así que todo dato "compartido" es en realidad una copia que puede estar desactualizada. Se resuelve con una única fuente de verdad (base de datos con transacciones) o con consenso entre nodos.

---

## Metas de diseño y transparencia (PRIORIDAD)

### 6. Explica la diferencia entre transparencia de migración y de reubicación, con un ejemplo propio de cada una.

- **Migración**: el recurso se mueve de lugar **entre usos**, y cuando vuelves a usarlo no notas que cambió de sitio. *Ejemplo*: tu buzón de correo se traslada a otro servidor durante la noche; al día siguiente entras igual que siempre.
- **Reubicación**: el recurso se mueve **mientras lo estás usando**, sin cortar la sesión. *Ejemplo*: estás viendo una película y el servicio cambia el servidor que te la envía; el video no se corta ni te pide volver a empezar.

La reubicación es bastante más difícil de lograr, porque hay que trasladar también el estado de la sesión en vivo.

### 7. Un sistema distribuido incorpora nodos con distinto hardware y sistema operativo sin problema. ¿Qué meta de diseño cumple y cómo se logra?

**Heterogeneidad**. Se logra acordando protocolos y formatos de datos comunes (HTTP, JSON, gRPC) que ambos extremos entiendan sin importar en qué lenguaje o sistema estén escritos, más middleware que traduzca donde haga falta.

Existe porque ningún sistema real se construye de una vez con máquinas idénticas: crece por partes, con hardware de años distintos y servicios escritos por equipos distintos.

### 8. Una empresa multiplica por 10 sus servidores y el rendimiento apenas mejora. ¿Qué está pasando?

Hay un **cuello de botella centralizado**: algún componente por el que pasa todo (una base de datos única, un servicio de sesiones, una tabla bloqueada) se saturó. Agregar máquinas no ayuda si todas terminan haciendo fila en el mismo punto.

**Las tres técnicas del curso** para atacarlo:
- **Ocultar latencias** — trabajar de forma asíncrona en vez de esperar bloqueado cada respuesta.
- **Ocultar la distribución con particionamiento** — partir los datos o el trabajo para que cada nodo atienda solo su porción (como hace DNS con sus zonas).
- **Ocultar la replicación con caché** — poner copias cerca de quien las consume para no ir hasta el origen cada vez.

### 9. ¿Por qué "más transparencia" no siempre es mejor?

Porque ocultar completamente que una llamada es remota lleva al programador a tratarla como si fuera local — sin timeouts, sin reintentos, sin considerar que puede fallar. Ahí es exactamente donde muerden las falacias de Deutsch.

A veces conviene que el desarrollador **sepa** que está cruzando la red, justamente para que la trate con el cuidado que merece. RPC es el caso clásico: da transparencia de acceso, pero esconde que la llamada puede tardar o fallar.

---

## Falacias de Deutsch (PRIORIDAD)

### 10. Un desarrollador escribe código que hace una llamada remota dentro de un ciclo de 500 iteraciones. Funciona perfecto en su máquina y colapsa en producción. ¿Qué falacia cometió?

**"La latencia es cero"** (y de paso "la red es confiable").

En su máquina todo era local y cada llamada tardaba fracciones de milisegundo, así que 500 llamadas eran imperceptibles. En producción, con 40 ms de latencia por llamada, esas mismas 500 iteraciones tardan 20 segundos.

**Por qué "el código no tiene errores"**: la lógica es correcta y pasa todas las pruebas. Lo que está mal es la **suposición** sobre el medio, no el algoritmo. Por eso estas falacias son tan peligrosas: no producen errores de compilación ni de test, producen sistemas que funcionan en desarrollo y se arrastran en producción.

### 11. Menciona dos falacias más y el error de diseño concreto que provoca cada una.

- **"La red es segura"** → no se cifra ni se autentica el tráfico; cualquiera en el camino puede leer o alterar los mensajes.
- **"La topología no cambia"** → se queman direcciones IP en el código; el día que un servidor se mueve o se reemplaza, todo deja de funcionar.
- **"El costo de transporte es cero"** → se mandan objetos enormes en cada petición, ignorando que serializar y mover datos cuesta CPU y dinero (las nubes cobran por transferencia de salida).

---

## Clasificación de Flynn (PRIORIDAD)

### 12. Clasifica según Flynn: (a) una GPU aplicando un filtro a una foto, (b) un cluster de 50 servidores atendiendo peticiones web distintas, (c) tu portátil ejecutando un script de Python de un solo hilo.

- **(a) SIMD** — una sola instrucción ("súbele el brillo") aplicada simultáneamente a millones de píxeles distintos.
- **(b) MIMD** — cada servidor ejecuta instrucciones distintas sobre datos distintos.
- **(c) SISD** — una instrucción sobre un dato a la vez, secuencial.

Para (b): es **débilmente acoplado**, porque los nodos se comunican por red y cada uno tiene su propia memoria, no un bus compartido.

### 13. ¿Por qué los sistemas distribuidos son débilmente acoplados, y por qué mucha gente responde lo contrario?

Son **débilmente acoplados** porque cada nodo tiene memoria propia y se comunican por red: el retraso de los mensajes es alto y conviene minimizar el volumen intercambiado.

La confusión viene de que "fuertemente acoplado" *suena* a "muy conectado" o "muy integrado", y un sistema distribuido parece muy integrado desde fuera. Pero el acoplamiento se refiere al **medio físico de comunicación**, no a qué tan coordinado se ve el sistema.

Y esa independencia entre nodos no es un defecto: es justamente lo que permite escalar y tolerar fallas — aunque también es lo que trae las 4 consecuencias.

---

## Computación en la nube

### 14. Una empresa dice tener "nube privada": compró servidores, los puso en su oficina, pero pedir una máquina virtual toma un ticket y tres días de espera. ¿Es nube?

**No.** Le falta la característica más definitoria: **auto-servicio por demanda**. Si hace falta que intervenga una persona para aprovisionar un recurso, es un datacenter tradicional, no una nube.

Probablemente también le falten **elasticidad rápida** (ajuste automático según demanda) y **medición del servicio** (facturación interna por consumo). Tener servidores virtualizados no basta: la nube es un **modelo de consumo**, no un tipo de hardware.

### 15. Una startup quiere lanzar en 6 semanas, tiene 3 desarrolladores y ninguno sabe administrar Linux. ¿Qué modelo de servicio le recomiendas?

**PaaS**. Suben su código y la plataforma se encarga del sistema operativo, el runtime, el despliegue y el escalado.

Por qué no los otros:
- **IaaS** los obligaría a administrar el SO, los parches y el autoescalado — justo lo que nadie del equipo sabe hacer, y consumiría semanas del plazo.
- **SaaS** no aplica: van a construir un producto propio, no a consumir uno existente.
- **FaaS** podría servir para partes puntuales, pero obliga a diseñar todo sin estado y en funciones sueltas, lo que complica una aplicación completa bajo presión de tiempo.

### 16. Un banco necesita que los datos de clientes no compartan infraestructura con terceros, pero su portal público recibe picos enormes a fin de mes. ¿Qué modelo de despliegue resuelve ambas cosas?

**Nube híbrida**. Los datos sensibles y regulados quedan en la **nube privada** (uso exclusivo, cumpliendo la normativa), y el portal público se atiende desde **nube pública**, que absorbe los picos con su elasticidad.

Ambas se conectan de forma que las aplicaciones y los datos puedan moverse entre ellas según lo que cada carga requiera.

### 17. Explica por qué el costo de transferencia de datos y el vendor lock-in se refuerzan entre sí.

Los proveedores cobran poco por **meter** datos y bastante por **sacarlos**. Entonces mientras más datos acumulas en un proveedor, más caro sale migrar a otro — y mientras más caro sale migrar, menos poder de negociación tienes y más servicios propietarios terminas adoptando.

Es un círculo: el costo de salida encarece la migración, la migración difícil profundiza la dependencia, y la dependencia hace crecer el volumen de datos atrapado.

### 18. Una función Lambda procesa imágenes 10 veces al día. La misma lógica en PaaS corre 24/7. ¿Cuál cuesta menos?

**FaaS**, con diferencia. Solo se cobra por invocación y tiempo de ejecución: 10 ejecuciones de unos segundos al día cuestan prácticamente nada, mientras que en PaaS pagas la aplicación viva las 24 horas aunque nadie la use.

**Se invierte** cuando el volumen es alto y sostenido. Con miles de invocaciones por minuto de forma constante, el precio por invocación de FaaS supera al costo fijo de tener servidores corriendo, y además aparece el problema del *cold start*.

### 19. Clasifica según modelo de servicio **y** de despliegue: (a) Gmail corporativo, (b) instancias EC2 de tu empresa en AWS, (c) un Kubernetes en el datacenter propio, de uso exclusivo interno.

- **(a)** SaaS en nube pública.
- **(b)** IaaS en nube pública.
- **(c)** Se comporta como PaaS (plataforma donde despliegas aplicaciones) sobre nube **privada**.

La clave es que los dos ejes son **independientes**: el modelo de servicio dice hasta dónde administra el proveedor; el de despliegue dice de quién es la infraestructura.

---

## Redes, HTTP y seguridad

### 20. Tu PC tiene IP 192.168.1.15 con máscara /24. ¿Qué hace para mandar un paquete a 192.168.1.20 y qué hace distinto para 8.8.8.8?

Con **/24**, los primeros tres octetos son la red. Tu máquina compara esa porción:

- **A 192.168.1.20** → red `192.168.1` = igual a la suya → están en la misma LAN → **le habla directamente** por el medio físico, sin pasar por el router.
- **A 8.8.8.8** → red `8.8.8` ≠ `192.168.1` → está afuera → **entrega el paquete al gateway** (el router), que lo reenvía hacia la red siguiente, saltando hasta el destino.

Esa comparación con la máscara es exactamente para lo que existe la máscara.

### 21. Una oficina tiene 20 computadores y una sola IP pública dinámica. ¿Qué permite que los 20 naveguen? ¿Y si quiere alojar un servidor web accesible desde fuera?

Los 20 navegan gracias a **NAT**: el router reemplaza la IP privada de origen por su IP pública y lleva una tabla para devolver cada respuesta a quien corresponda.

Pero NAT funciona para **salir**, no para **entrar**: como los equipos internos no tienen dirección visible desde Internet, nadie de afuera puede iniciar una conexión hacia ellos. Para alojar el servidor haría falta:

- configurar **redirección de puertos** en el router (dirigir el puerto 80 hacia una IP interna fija), y
- contratar una **IP pública estática**, porque con una dinámica la dirección cambia cada tanto y nadie sabría dónde encontrarlo (o usar DNS dinámico como parche).

### 22. ¿Por qué DNS es un sistema distribuido y no "un servidor con una tabla gigante"?

Porque es **jerárquico y repartido**: ningún servidor conoce todos los dominios del mundo. Cada uno conoce su zona y sabe a quién preguntarle por el resto, y entre todos se comportan como un servicio único y coherente — que es la definición exacta de sistema distribuido.

Usa dos técnicas de escalabilidad vistas en clase: **particionamiento** (cada servidor sabe solo de su zona) y **caché** (las respuestas se guardan un tiempo para no repetir la consulta).

Una tabla central única sería un cuello de botella imposible de escalar y un punto único de falla para todo Internet.

### 23. ¿Por qué que HTTP sea *stateless* es una ventaja para escalar horizontalmente, y qué costo trae?

**La ventaja**: como cada petición es independiente y el servidor no recuerda la anterior, **cualquiera** de los servidores detrás de un balanceador puede atender cualquier petición. Si el servidor tuviera que recordar tu sesión en su memoria, tendrías que volver siempre al mismo — y ahí se acaba la escalabilidad horizontal.

**El costo**: hay que simular el estado. De ahí salen las **cookies, tokens y sesiones**: el cliente reenvía en cada petición la prueba de quién es, lo que agrega tráfico y obliga a resolver aparte cómo se valida esa prueba de forma segura.

### 24. Un sistema autentica bien a los usuarios pero no registra qué hace cada uno. ¿Qué falta y qué objetivo del CIA se compromete?

Falta la tercera A de **AAA**: **Auditoría / Accounting**.

Compromete principalmente la **Integridad**, porque si alguien altera datos no hay forma de saber quién ni cuándo, ni de reconstruir el estado anterior. También debilita la respuesta ante una falla de **Confidencialidad**: si hubo una fuga, sin registros no puedes determinar qué se consultó ni el alcance real del incidente.

**Recuerda el contraste**: CIA son los **objetivos** de seguridad; AAA es el **mecanismo** de control de acceso con el que se consiguen.

### 25. Un sistema tiene 99,9 % de disponibilidad. ¿Cuántas horas de caída al año, y qué NO garantiza ese número?

**≈ 8,76 horas al año** ("tres nueves").

**No garantiza** rendimiento, seguridad ni consistencia de datos: son atributos independientes. Un sistema puede estar perfectamente "arriba" el 99,9 % del tiempo y aun así responder en 8 segundos, devolver datos corruptos o ser inseguro. Disponibilidad solo mide **que responda**, no que responda bien ni rápido.

### 26. Tu aplicación responde lento. Tienes logs pero no trazas, y la petición pasa por 5 servicios. ¿Por qué es un problema?

Porque los **logs** te dicen qué pasó **dentro de cada servicio por separado**, pero no te permiten seguir **una petición concreta** a través de los cinco. Verías cinco conjuntos de registros sin forma confiable de correlacionarlos, y no podrías saber cuál de los servicios consumió el tiempo.

Lo que necesitas es el tercer pilar de la observabilidad: las **trazas** (*tracing*), que siguen el recorrido completo de una petición entre servicios y muestran cuánto tardó en cada salto.

### 27. Una empresa duda entre comprar un servidor el doble de potente o dos servidores iguales al que tiene. Explica ambos caminos y qué problema nuevo trae el segundo.

- **Escalado vertical** (un servidor más potente): más CPU y RAM en la misma máquina. Es simple porque no cambia la arquitectura, pero tiene **techo físico**, el precio crece más rápido que la potencia, y sigue habiendo un **punto único de falla**.
- **Escalado horizontal** (más máquinas): sin techo teórico y permite redundancia, así que también mejora la disponibilidad.

**El problema nuevo del horizontal**: convierte el sistema en distribuido, y con eso hereda las 4 consecuencias — sin reloj global, fallas parciales, sin memoria compartida y concurrencia. Ahora hay que resolver balanceo de carga, sesiones compartidas y consistencia de datos entre nodos.

---

## Conceptos básicos

### 28. Explica la cadena programa → proceso → mensaje → paquete.

- **Programa**: código compilado, **estático**, guardado en disco. No hace nada.
- **Proceso**: ese programa **en ejecución**, con memoria propia, estado y PID, gestionado por el sistema operativo. De un mismo programa pueden nacer varios procesos independientes.
- **Mensaje**: la unidad de comunicación **entre procesos**. Existe porque los procesos en máquinas distintas no comparten memoria, así que la única forma de coordinarse es mandándose información.
- **Paquete**: una **porción** del mensaje, del tamaño adecuado para viajar por la red. Un mensaje se parte en varios paquetes y se reensambla en el destino.

**Por qué se parte en paquetes**: si un mensaje grande viajara entero y se corrompiera, habría que retransmitirlo completo; partido, solo se retransmite el pedazo dañado. Además permite que varios mensajes compartan el enlace intercalándose.

**Consecuencia**: los paquetes pueden tomar rutas distintas, llegar desordenados o perderse — la razón técnica de por qué "la red es confiable" es una falacia.

---

## Arquitecturas: Cliente-Servidor y Publish-Subscribe (nuevo — clase del 5 de agosto)

> El profesor marcó explícitamente estas dos preguntas en clase como material de examen.

### 29. "¿Son idempotentes o no?" — clasifica GET, POST, PUT y DELETE, y explica por qué importa en una arquitectura cliente-servidor.

- **GET** — sí. Solo lee, no cambia el estado del servidor.
- **PUT** — sí. Reemplaza el recurso por un valor fijo; ponerlo dos veces da el mismo resultado que ponerlo una.
- **DELETE** — sí. Borrar algo ya borrado deja el mismo estado: "no existe".
- **POST** — **no**. Crea un recurso nuevo en cada llamada; reenviarlo duplica el efecto.

**Por qué importa**: en cliente-servidor, cuando una petición expira por timeout no puedes distinguir si el servidor la procesó o si solo se perdió la respuesta (es la misma incertidumbre de las fallas parciales). Si el método es idempotente, reintentar es seguro. Si no lo es —como POST—, reintentar a ciegas puede cobrar dos veces o crear un pedido duplicado. La solución real para un POST que necesita reintento es agregar una **clave de idempotencia**: un identificador único que el servidor recuerda para no procesar dos veces la misma operación aunque le llegue repetida.

**Trampa**: idempotente no es "sin efectos secundarios". DELETE sí cambia algo (borra) y sigue siendo idempotente, porque lo que importa es que **repetirla** no cambie el resultado final.

### 30. "¿HTTP es simétrico o asimétrico?" — y por qué WebSocket es la excepción.

**HTTP es asimétrico.** La simetría de un protocolo se prueba con una sola pregunta: ¿puede **cualquiera** de los dos lados iniciar la comunicación? En HTTP, no — solo el cliente puede abrir la conexión y mandar una solicitud; el servidor únicamente responde a lo que ya le preguntaron. Nunca "llama" al cliente por su cuenta.

**Por qué WebSocket rompe esa regla**: WebSocket arranca como una petición HTTP normal (el cliente la inicia, como siempre) pero pide actualizar el protocolo (`Upgrade: websocket`). Una vez aceptada esa actualización, la conexión deja de comportarse como HTTP: queda abierta y **cualquiera de los dos lados** puede mandar datos cuando quiera, sin que el otro pregunte primero. Por eso WebSocket es un protocolo **simétrico** — se usa cuando cliente-servidor puro se queda corto, como en un chat en vivo donde el servidor necesita empujarte mensajes nuevos sin que tú refresques la página.

**Trampa**: decir que HTTP "es simétrico porque los dos mandan datos" (el cliente manda la petición, el servidor manda la respuesta). Mandar datos en ambas direcciones no es lo mismo que poder **iniciar** en ambas direcciones — eso segundo es lo que define la simetría.

---

## Enlaces

- [[01 Guia de Repaso - Parcial 1]] — guía principal con el plan día a día
- [[02 Sistemas Distribuidos - Conceptos]]
- [[03 Computacion en la Nube]]
- [[04 Redes de Datos]]
- [[Arquitecturas de Nube y Sistemas Distribuidos MOC]]
