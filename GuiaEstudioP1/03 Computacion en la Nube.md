---
tags: [estudio/nube, semestre-6]
created: 2026-07-22
updated: 2026-08-10
materia: Arquitecturas de Nube y Sistemas Distribuidos
---

# Arquitecturas de Nube

> La nube es el medio por el cual creamos e instanciamos los sistemas distribuidos. Por eso esta unidad va después de [[02 Sistemas Distribuidos - Conceptos]]: la nube es *dónde* se ejecuta lo que allá se explica.

---

## Qué es la computación en la nube

**Qué es**: usar recursos informáticos (cómputo, almacenamiento, red) que son **propiedad de un tercero**, están en **sus** instalaciones, y se consumen **por demanda a través de la red**, pagando solo por lo que se usa.

La palabra "nube" no es un tipo de tecnología — es un **modelo de consumo**. El hardware sigue siendo servidores en un galpón; lo que cambia es que no son tuyos, no los administras, y los pagas como un servicio público.

**Por qué existe**: porque montar infraestructura propia obliga a resolver un problema imposible — **adivinar cuánta capacidad vas a necesitar, antes de necesitarla**.

Si compras poco, tu servicio se cae el día que te va bien. Si compras de más, pagaste hardware que pasa el 90 % del tiempo apagado. Y en ambos casos pagaste todo por adelantado, esperaste semanas a que llegara, y te quedaste con equipos que envejecen.

La nube convierte esa apuesta en un ajuste continuo: pides lo que necesitas hoy y lo cambias mañana.

**Cómo funciona**: el proveedor compra hardware a escala enorme, lo **virtualiza** (parte cada servidor físico en muchas máquinas lógicas) y le alquila porciones a miles de clientes a la vez. Como sus clientes tienen picos en momentos distintos, el mismo hardware físico rinde mucho más que si cada empresa tuviera el suyo ocioso.

**Ejemplo concreto**: una tienda online colombiana vende normal todo el año, pero en Black Friday recibe 30 veces más tráfico. Con servidores propios tendría que comprar capacidad para ese único día y desperdiciarla los otros 364. En la nube levanta 30 máquinas el jueves, las apaga el sábado, y paga solo esas 48 horas.

**Error típico**: creer que "la nube" significa "por Internet". Una nube privada puede estar dentro de la propia empresa y sin salir a Internet, y sigue siendo nube — porque lo que la define es el modelo de autoservicio, elasticidad y medición, no el cable por donde viaja.

---

## Las 5 características esenciales

Son el criterio para decidir si algo **es** nube o solo es "un servidor de alguien más". Si falta alguna, no es nube.

### 1. Auto-servicio por demanda

**Qué es**: el usuario aprovisiona los recursos que necesita **sin que intervenga ninguna persona del proveedor**.

**Por qué importa**: es lo que cambia la escala de tiempo. Pedirle un servidor al área de TI de una empresa toma días o semanas de tickets y aprobaciones. Aquí entras a una consola y en dos minutos tienes la máquina corriendo.

**Ejemplo**: entras a AWS, eliges una instancia EC2, das clic en "Launch" y ya estás conectándote por SSH. Nadie de Amazon supo que existes.

### 2. Acceso ubicuo a la red

**Qué es**: los servicios están disponibles por la red y se consumen desde cualquier tipo de dispositivo estándar — portátil, celular, tablet — sin necesitar un cliente especial.

**Por qué importa**: si un servicio solo funcionara desde una terminal específica en una oficina específica, volveríamos al modelo que la nube vino a reemplazar.

**Ejemplo**: administras la misma infraestructura desde el navegador del PC, desde la app del celular o desde la línea de comandos.

### 3. Agrupación de recursos (pooling)

**Qué es**: el proveedor tiene un **conjunto común** de recursos físicos que asigna dinámicamente entre múltiples clientes según lo que cada uno pida en cada momento. Es lo que se llama **multi-tenancy** (múltiples inquilinos).

**Por qué importa**: es el mecanismo que hace la nube barata. Compartir el hardware entre clientes con picos en horarios distintos permite una utilización que ninguna empresa lograría sola.

**Ejemplo**: tu máquina virtual y la de otra empresa pueden estar corriendo sobre el mismo servidor físico, aisladas entre sí y sin saberlo.

**Cuidado**: esta característica es también la raíz de la principal **desventaja** percibida — la sensación de inseguridad por tener datos confidenciales sobre infraestructura compartida.

### 4. Rápida elasticidad

**Qué es**: los recursos crecen y decrecen rápido según la demanda, idealmente de forma automática, y desde la perspectiva del cliente parecen ilimitados.

**Por qué importa**: es la característica que resuelve directamente el problema de adivinar la capacidad. No aciertas la predicción — dejas de predecir.

**Ejemplo**: una regla de autoescalado agrega servidores cuando la CPU supera 70 % y los retira cuando baja de 30 %. De madrugada corren 2 máquinas; al mediodía, 20.

**Se confunde con escalabilidad** (contraste muy preguntado):

| | Escalabilidad | Elasticidad |
|---|---|---|
| Qué es | La *capacidad* de crecer sin degradarse | Que el ajuste sea **automático y bidireccional** |
| Dirección | Crecer | Crecer **y también decrecer** |
| Quién lo hace | Puede ser manual | El sistema solo, según demanda |
| Dónde vive | Propiedad de cualquier arquitectura | Característica propia de la nube |

Regla rápida: **toda nube elástica es escalable, pero no todo sistema escalable es elástico.**

### 5. Medición del servicio

**Qué es**: el uso se monitorea, controla y reporta automáticamente, y se factura por consumo real (*pay-per-use*).

**Por qué importa**: es lo que convierte la infraestructura de una **inversión** (compras un activo) en un **gasto operativo** (pagas por lo que consumes), igual que el agua o la luz.

**Ejemplo**: AWS cobra las instancias EC2 por segundo de uso y S3 por GB almacenado por mes. Si apagas la máquina, deja de cobrarte.

**Cuidado**: es un arma de doble filo, y aparece como desventaja en el apunte del curso — la **complejidad de la gestión de costos**. Como se paga por uso, un recurso olvidado encendido sigue generando factura, y las cuentas de nube mal vigiladas se disparan.

---

## Ventajas

Cada ventaja resuelve un problema concreto del modelo tradicional:

- **Reducción de costos** — no hay que comprar hardware por adelantado. Se elimina la inversión inicial (CAPEX) y se cambia por gasto según uso (OPEX).
- **Optimización de recursos** — gracias a la elasticidad, se paga por lo que se usa y no por el pico previsto.
- **Administración simplificada** — el proveedor se encarga del hardware, la virtualización y, según el modelo, hasta del sistema operativo y el runtime. Menos trabajo de operación para tu equipo.
- **Disponibilidad y acceso global** — los proveedores tienen datacenters en varias regiones, lo que permite acercar el servicio a los usuarios y sobrevivir a la caída de una zona entera.
- **Recuperación rápida** — restaurar desde una copia o levantar la infraestructura en otra región toma minutos, no días. La infraestructura como código hace que "reconstruir todo" sea ejecutar un script.
- **Rapidez y agilidad** — pasar de la idea a un servicio corriendo toma minutos, lo que acorta el *time-to-market*.
- **Seguridad** — los grandes proveedores ofrecen cifrado, gestión de identidades y certificaciones que superan lo que una empresa mediana podría montar por su cuenta.

## Desventajas

- **Percepción de inseguridad** — tener datos confidenciales sobre infraestructura compartida con otras organizaciones genera desconfianza, y en algunos sectores hay regulación que directamente lo prohíbe.
- **Pérdida de control físico** — no sabes en qué máquina está tu dato ni puedes entrar al datacenter. Dependes de las garantías del contrato.
- **Dependencia del acceso a Internet** — si se cae el enlace, se cae tu acceso a todo. En una infraestructura local, una app interna seguiría funcionando.
- **Dependencia del proveedor (vendor lock-in)** — mientras más servicios propietarios del proveedor uses, más caro y difícil se vuelve migrar a otro. Es la desventaja más subestimada.
- **Costo de transferencia de datos** — los proveedores suelen cobrar poco por meter datos y bastante por sacarlos. Mover información fuera de la nube es caro, y eso refuerza el lock-in.
- **Complejidad de la gestión de costos** — la facturación por uso, con cientos de conceptos distintos, es difícil de predecir y de auditar.

---

## Modelos de servicio

**Qué son**: definen **hasta dónde llega la responsabilidad del proveedor** y desde dónde empieza la tuya. Es la clasificación "según el modelo de servicio".

La idea que unifica los cuatro: existe una pila de capas (hardware → virtualización → sistema operativo → runtime/middleware → datos → aplicación). Cada modelo corta esa pila en un punto distinto.

### IaaS (Infrastructure as a Service)

**Qué es**: el proveedor te alquila infraestructura cruda — máquinas virtuales, disco, red — y tú administras todo lo que va encima: sistema operativo, parches, runtime y tu aplicación.

**Qué problema resuelve**: comprar servidores obliga a pagar por adelantado, esperar la entrega y mantenerlos aunque estén ociosos. Con IaaS levantas la máquina en minutos y la apagas cuando no la necesitas.

**Cómo funciona**: el proveedor virtualiza sus servidores físicos y te entrega una porción con acceso administrador, como si fuera una máquina tuya.

**Ejemplo**: AWS EC2 y S3. Pides una instancia con Ubuntu, entras por SSH e instalas lo que quieras — igual que un servidor físico, pero sin comprarlo.

**Cuidado**: el sistema operativo sigue siendo tu responsabilidad. Si sale un parche de seguridad de Ubuntu, lo aplicas tú. Eso es exactamente lo que te quita el PaaS.

**Cuándo conviene**: cuando necesitas control total del entorno, o vas a migrar una aplicación existente sin reescribirla (*lift and shift*).

### PaaS (Platform as a Service)

**Qué es**: el proveedor entrega una plataforma lista para ejecutar aplicaciones — sistema operativo, runtime, servidor de aplicaciones y escalado incluidos. Tú solo aportas **tu código y tus datos**.

**Qué problema resuelve**: administrar servidores no le da valor a tu producto. Aplicar parches, configurar el servidor web y ajustar el autoescalado es trabajo que no diferencia tu aplicación de ninguna otra.

**Cómo funciona**: subes tu código, la plataforma lo empaqueta, lo despliega y lo escala sola.

**Ejemplo**: Heroku o Google App Engine. Haces `git push` y tu aplicación queda en línea, escalando sin que hayas configurado un servidor.

**Cuidado**: pierdes control. Si tu aplicación necesita una versión específica de una librería del sistema o una configuración particular del SO, la plataforma puede simplemente no permitirlo.

### SaaS (Software as a Service)

**Qué es**: software completo y funcionando, listo para usar por Internet. No administras absolutamente nada de la infraestructura ni de la aplicación — solo la consumes y configuras.

**Qué problema resuelve**: la mayoría de las empresas no necesita *construir* un gestor de correo o un CRM; necesita *usarlo*. Desarrollarlo y mantenerlo sería un desperdicio.

**Ejemplo**: Gmail, Salesforce, Office 365. Abres el navegador, inicias sesión y trabajas.

**Cuidado**: es el modelo con menos control. No decides cuándo se actualiza, ni qué funciones cambian, y tus datos viven en el formato del proveedor — lock-in en su forma más fuerte.

### FaaS (Function as a Service) — *serverless*

**Qué es**: ejecutas **funciones sueltas** que se disparan por un evento, sin gestionar ningún servidor. El proveedor levanta el entorno cuando llega el evento, corre tu función y lo destruye.

**Qué problema resuelve**: incluso en PaaS pagas por la aplicación mientras está corriendo, aunque nadie la use. En FaaS, si tu función no se ejecuta, no pagas nada.

**Cómo funciona**: registras una función y el evento que la dispara (una petición HTTP, un archivo subido, un mensaje en cola). El proveedor la ejecuta bajo demanda y cobra por invocación y tiempo de ejecución.

**Ejemplo**: AWS Lambda. Cada vez que alguien sube una foto a S3, se dispara una función que genera la miniatura. Entre subida y subida, no hay nada corriendo ni nada facturándose.

**Cuidado**: "serverless" no significa que no haya servidores — significa que no los ves ni los administras. Y tiene el problema del *cold start*: si la función lleva rato sin usarse, la primera invocación tarda más porque hay que levantar el entorno.

**PaaS vs FaaS** (contraste que se pregunta):

| | PaaS | FaaS |
|---|---|---|
| Unidad que despliegas | Una aplicación completa | Una función suelta |
| Qué corre cuando nadie la usa | La aplicación sigue viva | Nada |
| Cómo se cobra | Por tiempo de aplicación activa | Por invocación y duración |
| Estado | Puede mantener estado en memoria | Sin estado entre invocaciones |

---

## Quién administra qué: on-premise vs. modelos de nube

Esta tabla es la forma más rápida de fijar los cuatro modelos, y es muy preguntable.

| Capa | On-Premise | IaaS | PaaS | SaaS | FaaS |
|---|---|---|---|---|---|
| Redes y hardware | **Cliente** | Proveedor | Proveedor | Proveedor | Proveedor |
| Virtualización | **Cliente** | Proveedor | Proveedor | Proveedor | Proveedor |
| Sistema operativo | **Cliente** | **Cliente** | Proveedor | Proveedor | Proveedor |
| Runtime / middleware | **Cliente** | **Cliente** | Proveedor | Proveedor | Proveedor |
| Datos | **Cliente** | **Cliente** | **Cliente** | Proveedor | **Cliente** |
| Aplicación | **Cliente** | **Cliente** | **Cliente** | Proveedor | Solo el código de la función |

**La regla que resume la tabla**: mientras más "aaS", **más administra el proveedor y menos control te queda**. No hay un modelo mejor que otro — hay un intercambio entre control y comodidad, y se elige según lo que el proyecto necesite.

**Analogía útil para recordarlo** (comer pizza):
- **On-premise**: la haces en casa, con tu horno y tus ingredientes.
- **IaaS**: compras la masa y los ingredientes; el horno lo pone otro.
- **PaaS**: pides pizza a domicilio; solo pones la mesa y la bebida.
- **SaaS**: vas al restaurante; no pones nada.
- **FaaS**: te sirven porción por porción, y solo pagas las que te comes.

---

## Modelos de implementación (despliegue)

**Qué son**: definen **de quién es** la infraestructura y **quién más la usa**. Es la clasificación "según el modelo de implementación", y es independiente del modelo de servicio: puedes tener IaaS en nube privada o SaaS en nube pública.

### Nube privada

**Qué es**: infraestructura de nube usada **por una sola organización**, no expuesta al público general.

**Por qué existe**: hay sectores (banca, salud, defensa, gobierno) donde la regulación exige que los datos no compartan infraestructura con terceros, o que no salgan del país.

**Qué aporta**: control total sobre la configuración, baja latencia si está cerca de los usuarios, y seguridad personalizable según las políticas propias.

**Error típico muy común**: creer que nube privada = on-premise. **No es lo mismo.** La nube privada puede estar alojada y operada por un tercero en el datacenter del tercero; lo que la hace privada es que la infraestructura es de **uso exclusivo** de una organización, no dónde está el rack.

### Nube pública

**Qué es**: infraestructura propiedad de un proveedor externo, compartida entre muchas organizaciones y accesible por Internet.

**Ejemplo**: AWS, Microsoft Azure, Google Cloud.

**Qué aporta**: es donde la elasticidad y la economía de escala funcionan al máximo, porque el pool de recursos es enorme.

### Nube híbrida

**Qué es**: combinación de nube privada y pública, conectadas de forma que los datos y las aplicaciones puedan moverse entre ambas.

**Por qué existe**: permite dejar lo sensible o lo regulado en privado y aprovechar la elasticidad de la pública para el resto.

**Ejemplo**: un banco guarda los datos de clientes en su nube privada por regulación, pero atiende el portal público desde nube pública, que absorbe los picos de tráfico.

### Multicloud

**Qué es**: usar servicios de **varios proveedores de nube pública** a la vez.

**Por qué existe**: para no depender de uno solo (mitigar el vendor lock-in), aprovechar el mejor servicio de cada proveedor, y sobrevivir a la caída completa de uno.

**Ejemplo**: una empresa corre sus máquinas en AWS pero usa BigQuery de Google para analítica.

**Híbrida vs. multicloud** (contraste que casi siempre se pregunta):

| | Nube híbrida | Multicloud |
|---|---|---|
| Qué combina | Privada **+** pública | Varias **públicas** entre sí |
| Motivo principal | Cumplir regulación sin perder elasticidad | No depender de un solo proveedor |
| ¿Hay nube privada? | Sí, por definición | No necesariamente |

Se pueden dar juntas: una empresa con nube privada + AWS + Azure es híbrida **y** multicloud.

---

## Por qué las organizaciones se mueven a la nube

Las razones del curso, ordenadas de la más estratégica a la más operativa:

- **Valor estratégico y time-to-market más rápido** — lanzar antes que la competencia vale más que ahorrar en servidores. Es la razón número uno en la práctica.
- **Reduce CAPEX** — desaparece la inversión de capital en infraestructura; se vuelve gasto operativo variable.
- **Aumenta el ROI** — al no inmovilizar capital en hardware ocioso, el retorno sobre lo invertido mejora.
- **Menos personal de TI dedicado a operación** — el equipo deja de mantener servidores y se dedica al producto.
- **Uso eficiente de recursos** — capacidad bajo demanda en lugar de capacidad para el pico.
- **Aprovisionamiento rápido y escalabilidad elástica** — minutos en vez de semanas.

**CAPEX vs OPEX** (vale la pena tenerlo claro, se pregunta):

| | CAPEX (Capital Expenditure) | OPEX (Operational Expenditure) |
|---|---|---|
| Qué es | Inversión inicial en un activo | Gasto recurrente por operar |
| En infraestructura | Comprar servidores | Pagar la factura mensual de la nube |
| Riesgo | Se paga antes de saber si se necesitaba | Se ajusta al uso real |

---

## Números de la nube

- El gasto mundial en servicios de infraestructura en la nube es muy alto y sigue creciendo año a año.
- Tres proveedores concentran la mayor parte del mercado: **AWS**, **Microsoft Azure** y **Google Cloud**.

> El profesor mostró cifras de cuota de mercado de Q4 2023, Q3 2024 y Q1 2026. **Verifica los porcentajes exactos en `02-Nube.pdf`** si en el parcial piden datos numéricos — lo conceptual (quiénes lideran y que el gasto crece) está cubierto aquí.

---

## Errores típicos a evitar

- **Nube privada ≠ on-premise.** Lo privado es el uso exclusivo, no la ubicación física.
- **Híbrida ≠ multicloud.** Híbrida mezcla privada con pública; multicloud usa varias públicas.
- **Elasticidad ≠ escalabilidad.** La elasticidad es automática y va en ambos sentidos.
- **Serverless no significa "sin servidores"**, significa que no los administras tú.
- **"En la nube" no significa "por Internet"** — significa autoservicio, elasticidad y medición.
- Confundir modelo de **servicio** (IaaS/PaaS/SaaS/FaaS: hasta dónde administra el proveedor) con modelo de **despliegue** (privada/pública/híbrida/multicloud: de quién es la infraestructura). Son ejes independientes.

---

## Preguntas de repaso

1. Una empresa dice que tiene "nube privada" porque compró servidores y los puso en su oficina, pero cada vez que un equipo necesita una máquina virtual tiene que abrir un ticket y esperar tres días. ¿Es nube? Justifica con las 5 características.

2. Una startup quiere lanzar su producto en 6 semanas, tiene 3 desarrolladores y ninguno sabe administrar servidores Linux. ¿Qué modelo de servicio le recomiendas y por qué descartas los otros tres?

3. Un banco necesita que los datos de sus clientes no compartan infraestructura con otras empresas, pero su portal público recibe picos enormes en fin de mes. ¿Qué modelo de despliegue resuelve las dos necesidades y cómo se reparte cada carga?

4. Explica por qué el costo de transferencia de datos hacia afuera y el vendor lock-in son dos desventajas que se refuerzan entre sí.

5. Una función Lambda procesa imágenes y se ejecuta 10 veces al día. La misma lógica en un servidor PaaS corre 24/7. ¿Cuál cuesta menos y por qué? ¿En qué escenario se invierte la respuesta?

6. Tu jefe dice: "pasémonos a la nube para ahorrar plata". Dale dos razones por las que el ahorro puede no darse, y una razón mejor para migrar.

7. Clasifica cada uno según modelo de servicio **y** de despliegue: (a) Gmail corporativo, (b) máquinas EC2 de tu empresa en AWS, (c) un cluster de Kubernetes montado en el datacenter propio de la empresa para uso exclusivo interno.

## Enlaces

- [[Arquitecturas de Nube y Sistemas Distribuidos MOC]]
- [[02 Sistemas Distribuidos - Conceptos]]
- [[04 Redes de Datos]]
- [[01 Guia de Repaso - Parcial 1]]
- [[02 Areas/Estudio/Semestre 6/Semestre 6 MOC]]
