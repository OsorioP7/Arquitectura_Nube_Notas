# Repaso express · Parcial 2 de Sistemas Distribuidos
### gRPC → MOM → P2P → MPI, en lo que cabe en una sesión

Esto es lo que se lee la noche anterior y lo que se copia a la hoja manuscrita. Cada punto tiene su explicación larga en `guia-parcial2-sistemas-distribuidos.md` (la sección va entre corchetes).

---

## 0. Lo que Álvaro anticipó en clase

Casi textual, de las grabaciones:

1. **"Las comunicaciones colectivas son: A. multicast, B. broadcast…"** → **broadcast**. Participan **todos** los procesos del comunicador. Multicast es para los suscritos. [6.10]
2. **"¿Qué hace el código? Defínalo en una frase."** → di el **propósito**, no narres el `for`. *"P0 envía a P1 un vector de 10 enteros del 0 al 9 y P1 lo imprime."* [6.8]
3. **"Con np 8, ¿cuánto vale A en P0, P1 y P5?"** → 3, 10, 2. Cada proceso tiene **su propia copia** de cada variable. [6.7]
4. **"¿S tiene que ser menor, igual o mayor que 1?"** → **mayor**; lo ideal, S >> 1. [5.4]
5. **"Si con MPI se demora más que secuencial, cárcel."** → el criterio es `t_proc >> t_comm`. [5.1]

---

## 1. Las diez tablas

### T1 · RPC, cola y tópico ⭐ (la regla de decisión)

| | RPC / gRPC | Cola | Tópico |
|---|---|---|---|
| Acoplamiento temporal | Acoplado | **Desacoplado** | **Desacoplado** |
| Acoplamiento referencial | Acoplado | Acoplado (nombra la cola) | **Desacoplado** |
| Bloqueo | Bloqueante | No bloqueante | No bloqueante |
| Cardinalidad | 1:1 | 1:1 (consumidores **compiten**) | 1:N (cada uno recibe **copia**) |
| Sirve para | Necesito la respuesta **ya** | **Repartir trabajo** | **Difundir eventos** |

**Regla:** ¿necesito la respuesta para seguir? → RPC. ¿Alguien tiene que hacer este trabajo una vez? → cola. ¿A varios les interesa este hecho? → tópico. [3.16]

### T2 · Los tres desacoples de MOM

| Desacople | En RPC | En MOM |
|---|---|---|
| **Tiempo** | Ambos vivos a la vez | El mensaje espera en el broker |
| **Espacio** | Conoce dirección e interfaz | Solo conoce cola o tópico |
| **Ritmo** | Espera cada respuesta | La cola amortigua (1.000 pedidos/s contra 200 pagos/s) |

Analogía de Álvaro: **RPC = teléfono; MOM = correo.** Mecanismo: **store-and-forward**. [3.2]

### T3 · Colas contra tópicos (lámina 6)

| Punto a punto (colas) | Publicador/Suscriptor (tópicos) |
|---|---|
| Cada mensaje, **un solo** consumidor | Cada mensaje, **todos** los suscriptores |
| Consumidores compiten (round-robin) | Fan-out: cada uno su copia |
| Productor conoce el **nombre de la cola** | Editor **no conoce** a los suscriptores |
| FIFO; dedicada o compartida | Push (broker empuja) o pull (suscriptor consulta) |

⚠️ Hay tres "punto a punto" en el parcial: colas en MOM, `Send`/`Recv` en MPI, y peer-to-peer. [3.6]

### T4 · Estándares y protocolos

| JMS | AMQP |
|---|---|
| **API** de Java | **Protocolo** de red |
| **No** interopera entre brokers | Interoperabilidad real |
| ActiveMQ, IBM MQ | RabbitMQ; exchanges **direct, fanout, topic, headers** |

**MQTT:** binario, ligero, pub/sub, **IoT** (QoS 0/1/2 = at most/at least/exactly once). **STOMP:** texto, tipo HTTP. **XMPP:** XML, mensajería instantánea. [3.11]

### T5 · Kafka contra RabbitMQ

| | RabbitMQ | Kafka |
|---|---|---|
| Modelo | Broker de colas (AMQP) | **Log** distribuido particionado |
| Orden | Por cola | **Por partición** |
| Retención | Se borra al **ack** | Se retiene: **replay** |
| Entrega | **Push** | **Pull**, el consumidor controla su **offset** |
| Uso | Tareas, RPC-like, enrutamiento | Streaming, event sourcing |

Quorum queues (RabbitMQ 4.0 eliminó las espejadas): **Raft**, líder + seguidores, se confirma con **mayoría**. [3.13, 3.14]

### T6 · gRPC contra REST, y contra SUN-RPC

| | gRPC | REST | SUN-RPC |
|---|---|---|---|
| Transporte | **HTTP/2** | HTTP/1.1 (lámina) | TCP/UDP |
| Formato | **Protobuf** binario | JSON texto | XDR |
| IDL / compilador | `.proto` / `protoc` | opcional | `.x` (RPCL) / `rpcgen` |
| Streaming | Nativo, 4 modos | Limitado | No |
| Descubrimiento | Dirección fija, DNS | URL | **Portmapper :111** |
| Mejor para | Microservicios internos | APIs públicas, navegador | LAN, C |

Pasos de gRPC: **1.** `.proto` → **2.** `protoc` genera stubs → **3.** servidor hereda del *Servicer* → **4.** cliente llama al *Stub* como función local. [2.6, 2.10]

### T7 · Organización: C/S y P2P

| C/S | P2P |
|---|---|
| Roles estables, centralizado | Roles simétricos, cambian por interacción |
| Capacidad fija | Capacidad **crece** con los peers |

| Estructurado | No estructurado | Híbrido |
|---|---|---|
| Reglas deterministas: **DHT, Chord** | Enlaces libres: **flooding**, random walk | Índice central o superpeers: **control ≠ datos** |

Tres problemas de todo P2P: **membresía, localización, transferencia.** [4.3, 4.4]

### T8 · Distribuido contra paralelo (tu apunte del 16/09)

| SD | SP |
|---|---|
| Procesos | Hilos |
| Multicomputador | Multiprocesador |
| Memoria **distribuida** | Memoria **compartida** |
| **MPI** (MPP, clúster) | **OpenMP** (SMP) |
| Riesgo: deadlock | Riesgo: condición de carrera |

Híbrido correcto: **cada proceso MPI lanza varios hilos** (un proceso MPI por nodo). [5.2, 8.4]

### T9 · Las funciones MPI

| Grupo | Función | Para qué |
|---|---|---|
| Inicio y fin | `MPI_Init(&argc,&argv)` / `MPI_Finalize()` | Primera y última |
| Control | `MPI_Comm_size(comm,&npr)` | **¿Cuántos somos?** |
| | `MPI_Comm_rank(comm,&pid)` | **¿Quién soy?** |
| Punto a punto | `MPI_Send(&m,count,type,dest,tag,comm)` | Enviar |
| | `MPI_Recv(&m,count,type,src,tag,comm,&status)` | Recibir (bloquea) |
| Colectivas | `MPI_Bcast` | Mismo dato a todos (árbol: log₂ p rondas) |
| | `MPI_Scatter` / `MPI_Gather` | Repartir trozos / recolectar |
| | `MPI_Reduce` / `MPI_Allreduce` | Combinar en root / en todos |
| | `MPI_Barrier` | Nadie pasa hasta que lleguen todos |

Ops de reducción: SUM, PROD, MAX, MIN, MAXLOC, MINLOC, lógicas y bits. **No hay promedio.** [6.3, 6.10]

### T10 · Modos de comunicación

| Síncrona | Con búfer |
|---|---|
| Espera acuerdo de ambos; **siempre bloqueante** | Deja en búfer y retorna |
| **Más rápida** si el receptor está listo (sin copia) | Más lenta (copias), pero no bloquea |
| Sincroniza; ⚠️ riesgo de **deadlock** | Hay que gestionar búferes |

| Envío | Retorna cuando |
|---|---|
| `MPI_Ssend` | El receptor **comenzó la lectura** |
| `MPI_Isend` | **Ya**; luego `MPI_Test` (0/1) o `MPI_Wait` |
| `MPI_Send` | Lo decide la implementación: búfer si es pequeño, síncrono si es grande |

[6.9]

---

## 2. Fórmulas y cuentas

**Speedup y eficiencia** (tu apunte del 23/09):

$$S(n) = \frac{t_s}{t_p(n)} \qquad E(n) = \frac{S(n)}{n}$$

n = procesos ≈ vCPU. Ideal: S = n, E = 100%. Si S > n (superlineal, como el ejemplo de clase 10 s → 2 s con 4 cores), sospecha.

**Medición real** (integral, 2 vCPU): np 1 → 2,88 s; np 2 → 1,45 s (**S = 1,99, E = 99%**); np 4 → 1,44 s (**S = 2, E = 50%**). Más procesos que núcleos no acelera. [5.4]

**Amdahl:**

$$S(n) = \frac{1}{s + p/n} \qquad S_{max} = \frac{1}{s}$$

Ejercicio del parcial anterior: p = 0,7, S = 2,5 → 0,3 + 0,7/n = 0,4 → **n = 7**. El Ts = 150 **no se usa**. Techo con s = 0,3: **3,33**. [5.5]

**Quórum (Raft):** N = 2f + 1. 3 nodos → tolera 1; 4 → **también 1**; 5 → 2. Siempre impar. [3.14]

**Flooding:** 4 vecinos, TTL 3 → hasta 4 + 12 + 36 = **52 mensajes**. [4.5]

**Chord** (nodos 1, 4, 6, 8, 10, 12, 14; m = 4): la clave k va a `successor(k)`. Clave 11 → 12; clave 15 → **1** (da la vuelta). Finger *i* del nodo p = `successor(p + 2^(i−1))`. Tabla del nodo 1: 4, 4, 6, 10. Búsqueda de 11 desde 1: 1 → 10 → responsable 12. Saltos: **O(log N)**. [4.8]

**La integral:** base = **1./n**; x = i·base; suma += base·4/(1+x²). Pruebas: n=1 → 4; n=2 → 3,6; n=3 → **3,4564**; n=5 → 3,3349; n=100 → 3,1516. Resultado → π. [7.2]

---

## 3. Chuleta MPI

**Esqueleto que sale en todo:**

```c
MPI_Init(&argc, &argv);
MPI_Comm_rank(MPI_COMM_WORLD, &pid);     /* ¿quién soy?   */
MPI_Comm_size(MPI_COMM_WORLD, &npr);     /* ¿cuántos somos? */
if (pid == 0) { /* leer entrada */ }
MPI_Bcast(...);                          /* TODOS lo llaman, nunca dentro del if */
/* cada uno calcula su trozo */
MPI_Reduce(...);                         /* o Gather */
if (pid == 0) { /* imprimir */ }
MPI_Finalize();
```

**Reglas para "¿qué imprime?":**
- Lo que está entre `Init` y `Finalize` sin `if` lo ejecutan **todos**.
- Cada proceso tiene **su copia** de cada variable: nadie ve los cambios de otro sin un mensaje.
- El **orden de las líneas impresas no está garantizado**: el SO planifica.
- Un proceso que no entra a ningún `if` conserva los valores iniciales.
- El rango va de **0 a n−1** y **no es el PID del SO**.
- P0 = **director de orquesta**: entrada, reparto, recolección, salida. Nunca debe morir.
- Modelo: **SPMD**, no SIMD.

**La receta:**

```bash
apt install openmpi-bin openmpi-doc libopenmpi-dev   # root (#)
su - ubuntu                                          # mínimo privilegio ($)
mpicc ej1.c -o ej1                                   # COMPILAR
mpirun -np 4 ./ej1                                   # EJECUTAR
mpirun --oversubscribe -np 8 ./ej1                   # más procesos que slots
mpirun --hostfile maquinas.txt -np 12 ./ej1          # clúster
```

**Las cuatro conclusiones del 18/09:** (1) el SO asigna PID, sin orden; (2) ejecutable + datos → **transferir archivos** → necesidad de **DFS / carpeta compartida**; (3) cada proceso puede tener una tarea distinta (`if` sobre el rango); (4) muchos procesos → **clúster**. Las valiosas: la 2 y la 4.

**El deadlock:** los dos hacen `Send` y luego `Recv`. Funciona con mensajes pequeños (búfer interno) y se congela con grandes. Arreglo: invertir el orden en uno, `MPI_Sendrecv`, o `Isend` + `Wait`. [6.9]

**V(i) × suma:** Bcast del tamaño → Scatter → suma local → **Allreduce** (no Reduce, porque **todos** multiplican) → multiplicar → Gather. [6.11]

---

## 4. Erratas y precisiones que valen puntos

**Erratas** (responde con lo correcto):
1. Lám. 35: n = 3 da **3,4564**, no 3,4554.
2. Lám. 34: `f(x) = 4/(1*x^2)` → es **4/(1+x^2)**.
3. Lám. 34: `base = 1/n` es **división entera** (0 para n ≥ 2) → `1./n`.
4. `EjercicioIntegral.c`: `n = 20000000000` **desborda el int** (máx. 2.147.483.647) e imprime 0 → usar 2.000.000.000 o `long long`.
5. `float` en vez de `double` → la suma **se congela en 0,0625**.
6. Lám. 8: `MPI_UNSIGNED_SHOT` → `MPI_UNSIGNED_SHORT`.
7. No existe `MPI_AVG`: se hace con `MPI_SUM` y se divide.
8. Se compila con `mpicc`, se ejecuta con `mpirun`.

**Precisiones** (responde con la lámina, matiza si es abierta):
1. gRPC quita el acoplamiento de **plataforma**, no el **temporal**.
2. Protobuf no siempre es más pequeño: con dos `double`, 18 bytes contra 17 de JSON.
3. Cola FIFO ≠ procesamiento en orden con varios consumidores.
4. El `status` de MPI no es contra pérdida de paquetes: dice origen, tag y cuántos llegaron.
5. Las colectivas "son bloqueantes": cierto hasta MPI-2; MPI-3 tiene versiones no bloqueantes.
6. En la lám. 33, *n* son los intervalos **por proceso**; en la 34 y 35, el **total**.

---

## 5. Los cinco patrones de trampa

**1. La tabla literal.** Las preguntas salen de las tablas de las láminas: RPC contra MOM, gRPC contra REST, Kafka contra RabbitMQ, síncrona contra búfer. Si memorizas T1 a T10, respondes sin pensar.

**2. El enunciado invertido.** La misma tabla con el signo cambiado. "¿Qué modelo **reparte** trabajo?" (cola) contra "¿cuál **difunde** eventos?" (tópico). "¿Cuál es **más rápida** si el receptor está listo?" (síncrona) contra "¿cuál **no bloquea** al emisor?" (búfer).

**3. El distractor que suena bien.** "Un proceso MPI por core." "MPI implementa SIMD." "La cola es FIFO, luego se procesa en orden." "JMS garantiza interoperabilidad." "gRPC es asíncrono y desacoplado como un MOM." "Más procesos que núcleos acelera." Todas falsas y todas razonables a primera vista.

**4. La negación.** "¿Cuál **NO** es característica de MOM?" "¿Cuál **NO** es colectiva?" En los parciales anteriores hubo una negación en cada uno. Subraya el NO antes de leer las opciones.

**5. El código rastreado** (nuevo en este corte). "¿Qué imprime el proceso k?", "¿qué hace el código en una frase?", "¿por qué da 0?". Se responde con las reglas de la sección 3 y los bugs de la sección 4, y se gana con una frase de propósito, no con una narración línea por línea.

---

## 6. La hoja manuscrita

Si permiten una hoja escrita a mano como en el primer corte, estos doce bloques caben en una cara a dos columnas:

1. **T1** completa + la regla de tres preguntas
2. **Tres desacoples** de MOM (tiempo, espacio, ritmo)
3. **JMS = API / AMQP = protocolo**; exchanges direct, fanout, topic, headers
4. **Kafka contra RabbitMQ** (retención, entrega, orden)
5. **Quórum** N = 2f + 1
6. **gRPC:** HTTP/2 + protobuf, 4 pasos, 4 tipos de streaming, sin portmapper
7. **Chord:** successor, finger = succ(p + 2^(i−1)), la clave 15 → nodo 1
8. **S, E, Amdahl** y el techo 1/s
9. **Las seis funciones** básicas + las colectivas con su dibujo (A → AAAA, ABCD → A|B|C|D, y al revés)
10. **Send, Ssend, Isend** y cuándo retornan
11. **La receta** (`mpicc`, `mpirun -np`, `--oversubscribe`, `--hostfile`)
12. **Los tres bugs** de la integral: `1/n`, `float`, el `int` desbordado

---

*Derivado de la guía completa del Parcial 2. Todo lo que dice "medido" o "real" se compiló y ejecutó.*
