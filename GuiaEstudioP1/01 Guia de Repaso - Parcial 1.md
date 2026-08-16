---
tags: [estudio/nube, repaso, examen, semestre-6]
materia: Arquitecturas de Nube y Sistemas Distribuidos
fecha_examen: 2026-08-19
---

# Repaso y Examen - Parcial 1

**Parcial 1: miércoles 19 de agosto de 2026 · 15 % · evaluación escrita.**
Faltan 9 días desde hoy (2026-08-10).

Esta guía es el mapa. Las notas están **numeradas por orden de repaso**: estudia de la 02 a la 05, practica con la 06 y verifica con la 07.

| # | Nota | Prioridad |
|---|---|---|
| 02 | [[02 Sistemas Distribuidos - Conceptos]] | **Alta** — lo más denso y lo más flojo |
| 03 | [[03 Computacion en la Nube]] | Media |
| 04 | [[04 Redes de Datos]] | Baja — es tu tema fuerte |
| 05 | [[05 Conceptos Basicos y Preliminares]] | Baja — vocabulario base |
| 06 | [[06 Fichas de Repaso]] | Recall diario, desde el día 2 |
| 07 | [[07 Preguntas de Repaso - Parcial 1]] | Prueba completa el día 6 |

---

## Temas que entran

- [ ] Conceptos básicos y preliminares (programa, proceso, mensaje, paquete, servicio, API, data center)
- [ ] Repaso de redes (IP, máscara, gateway, NAT, DHCP, DNS, firewall, puertos)
- [ ] HTTP y seguridad (stateless, métodos, códigos; triángulo CIA y triada AAA)
- [ ] Atributos no funcionales (escalabilidad, rendimiento, disponibilidad, observabilidad)
- [ ] Computación en la nube (5 características, ventajas/desventajas, IaaS/PaaS/SaaS/FaaS, modelos de despliegue)
- [ ] Sistemas distribuidos (definición, 4 consecuencias, retos, metas de diseño, transparencia)
- [ ] Falacias de Deutsch
- [ ] Clasificación de Flynn y acoplamiento
- [ ] Arquitecturas cliente-servidor y publish-subscribe (idempotencia HTTP, simetría, acoplamiento) — tema de la clase del 5 de agosto, el más reciente

## Lo que sé / no sé

| Tema | Nivel (1-5) | Qué falta exactamente |
|---|---|---|
| Redes de datos | 4 | Practicar el cálculo de subredes y hosts usables sin ver la tabla |
| Conceptos básicos | 3 | Poder explicar programa→proceso→mensaje→paquete como cadena, no como 4 definiciones |
| HTTP, CIA y AAA | 2 | Tema que apenas anotaste; el contraste CIA vs AAA es el que más se presta a confusión |
| Atributos no funcionales | 3 | Repasar disponibilidad con el cálculo de horas y latencia vs throughput |
| Computación en la nube | 3 | Memorizar ventajas/desventajas y la tabla de quién administra qué |
| **Sistemas distribuidos** | **2** | **Las 4 consecuencias y las 8 transparencias — es lo más denso del parcial** |
| **Falacias de Deutsch** | **2** | Poder *explicar el efecto* de cada una, no recitar la lista |
| **Flynn y acoplamiento** | **2** | Tema nuevo; el error de fuerte vs débil acoplado es trampa clásica |
| **Cliente-servidor / publish-subscribe** | **1** | Tema de la última clase (5 ago), aún sin repasar. El profesor marcó dos preguntas explícitas: idempotencia y simetría de HTTP |

### Orden de estudio recomendado

1. **Sistemas distribuidos — las 4 consecuencias.** Es el corazón del parcial y de donde se deduce casi todo lo demás. Si entiendes por qué no hay reloj global ni memoria compartida, media materia se explica sola.
2. **Transparencia (los 8 tipos) y metas de diseño.** Mucho volumen de memorización, empieza temprano.
3. **Flynn y acoplamiento.** Poco contenido pero es nuevo y tiene una trampa conocida.
4. **Cliente-servidor y publish-subscribe.** Es lo más reciente (clase del 5 de agosto) y trae dos preguntas que el profesor marcó explícitamente: idempotencia y simetría de HTTP. Alto riesgo de que entren tal cual al examen.
5. **Falacias de Deutsch.** Rápido de aprender si lo enfocas en el efecto de cada una.
6. **Nube.** Ya tienes el contenido completo; es repaso de tablas.
7. **Redes.** Es tu tema más fuerte, déjalo para el final como repaso ligero.

---

## Las comparaciones que hay que dominar

Aquí es donde se pierden más puntos. Si solo tuvieras 20 minutos antes del examen, estudiarías esto.

| Par | La diferencia en una línea |
|---|---|
| **Escalabilidad vs elasticidad** | Escalabilidad = capacidad de crecer. Elasticidad = que el ajuste sea automático y en ambos sentidos |
| **Escalado vertical vs horizontal** | Vertical = más CPU/RAM a una máquina (con techo). Horizontal = más máquinas (sin techo, pero trae problemas de distribución) |
| **NAT vs DHCP** | NAT *traduce* privadas → pública para salir. DHCP *asigna* IP/máscara/gateway/DNS al conectarse |
| **CIA vs AAA** | CIA = los **objetivos** de seguridad. AAA = el **mecanismo** de control de acceso |
| **Autenticación vs autorización** | Autenticación = quién eres. Autorización = qué puedes hacer |
| **Nube privada vs on-premise** | Privada = uso **exclusivo** de una organización; puede estar alojada por un tercero |
| **Híbrida vs multicloud** | Híbrida = privada **+** pública. Multicloud = varias **públicas** entre sí |
| **PaaS vs FaaS** | PaaS despliega una app que sigue viva. FaaS ejecuta una función solo cuando hay evento |
| **Transparencia de acceso vs ubicación** | Acceso = lo uso **igual** sea local o remoto. Ubicación = **no sé dónde** está |
| **Transparencia de migración vs reubicación** | Migración = se mueve **entre** usos. Reubicación = se mueve **durante** el uso |
| **Fuerte vs débil acoplamiento** | Fuerte = memoria compartida, bus. Débil = red. **Los sistemas distribuidos son débilmente acoplados** |
| **Latencia vs throughput** | Latencia = cuánto tarda una petición. Throughput = cuántas atiende por segundo |
| **Confiabilidad vs disponibilidad** | Confiabilidad = los datos son correctos. Disponibilidad = el servicio responde |
| **Programa vs proceso** | Programa = archivo estático en disco. Proceso = ese programa **ejecutándose** |
| **Mensaje vs paquete** | El mensaje es la información completa; se **parte en** varios paquetes para viajar |
| **Cliente-servidor vs publish-subscribe** | Cliente-servidor = acoplado, el cliente conoce al servidor. Publish-subscribe = desacoplado, nadie conoce a nadie, solo al broker |
| **Idempotente vs no idempotente** | Idempotente = repetirla da el mismo resultado final (GET, PUT, DELETE). No idempotente = repetirla duplica el efecto (POST) |
| **Protocolo simétrico vs asimétrico** | Simétrico = cualquiera de los dos lados puede iniciar (WebSocket, tras el *upgrade*). Asimétrico = solo un lado inicia siempre (HTTP: solo el cliente) |
| **Desacoplamiento referencial vs temporal** | Referencial = no necesitas la dirección del otro. Temporal = no necesitas que el otro esté activo al mismo tiempo que tú |

---

## Listas cerradas que se preguntan literal

Son las que piden "nombre las N...". Vale la pena memorizarlas exactas.

**5 características esenciales de la nube**: auto-servicio por demanda · acceso ubicuo a la red · agrupación de recursos · rápida elasticidad · medición del servicio.

**4 modelos de servicio**: IaaS · PaaS · SaaS · FaaS.

**4 modelos de despliegue**: privada · pública · híbrida · multicloud.

**4 consecuencias de los sistemas distribuidos**: sin reloj global · fallas independientes · sin memoria compartida · concurrencia. *(Corolario: no hay estado global perceptible.)*

**Metas de diseño**: heterogeneidad · openness · seguridad · escalabilidad · manejo de fallas · concurrencia · transparencia.

**8 tipos de transparencia**: acceso · ubicación · migración · reubicación · replicación · concurrencia · fallo · persistencia.

**3 técnicas de escalabilidad**: ocultar latencias · ocultar la distribución (particionamiento) · ocultar la replicación (caché).

**8 falacias de Deutsch**: red confiable · latencia cero · ancho de banda infinito · red segura · topología fija · un solo administrador · transporte gratis · red homogénea.

**Clasificación de Flynn**: SISD · SIMD · MISD · MIMD.

**Métodos HTTP idempotentes**: GET · PUT · DELETE. **No idempotente**: POST. PATCH depende de qué modifique.

**Estilos arquitectónicos del curso**: Cliente-Servidor · Publish-Subscribe.

**Triángulo CIA**: Confidencialidad · Integridad · Disponibilidad.
**Triada AAA**: Autenticación · Autorización · Auditoría.

**3 pilares de observabilidad**: logs · métricas · trazas.

**3 servicios de TI**: cómputo · almacenamiento · red.

---

## Datos y cifras exactas

- **Disponibilidad**: 99 % ≈ 3,65 días de caída al año · **99,9 % ≈ 8,76 horas** · 99,99 % ≈ 52,6 minutos.
- **Máscara /24** = 255.255.255.0 = 256 direcciones = **254 hosts usables** (se restan red y broadcast).
- **Rangos IP privados**: 10.0.0.0/8 · 172.16.0.0/12 · 192.168.0.0/16.
- **Puertos**: 22 SSH · 53 DNS · 80 HTTP · 443 HTTPS · 3306 MySQL · 5432 PostgreSQL.
- **IPv4** = 32 bits · **IPv6** = 128 bits.
- **PAN** ≈ 10 metros.
- Líderes de mercado en nube: **AWS, Azure, Google Cloud**.

---

## Errores típicos a evitar

- **Los sistemas distribuidos son DÉBILMENTE acoplados.** "Fuertemente acoplado" suena a "muy conectado" y por eso mucha gente lo marca mal. Es al revés.
- **CIA son los objetivos; AAA es el mecanismo.** No los intercambies.
- **Nube privada no es lo mismo que on-premise.** Lo privado es el uso exclusivo, no dónde está el rack.
- **Híbrida no es multicloud.** Híbrida necesita una nube privada; multicloud no.
- **NAT no asigna direcciones**, las traduce. Quien asigna es DHCP.
- **Serverless sí tiene servidores**, solo que no los administras tú.
- **Un mensaje no es un paquete.** El mensaje se parte en paquetes.
- Al calcular hosts usables, **restar 2**: /24 son 254, no 256.
- Asumir que si una petición no respondió, no se ejecutó. Puede haberse ejecutado y haberse perdido la respuesta.
- Recitar las falacias de Deutsch sin poder explicar el efecto de ninguna: casi siempre piden explicar, no listar.
- Creer que NTP resuelve el problema del reloj global. Lo reduce, no lo elimina.
- **Decir que HTTP es simétrico.** Es asimétrico: solo el cliente puede iniciar la conexión. WebSocket sí es simétrico una vez abierto.
- **Confundir idempotencia con "sin efectos secundarios".** DELETE cambia algo (borra) y aun así es idempotente, porque repetirlo da el mismo resultado que hacerlo una vez.
- **Pensar que publish-subscribe y WebSocket son lo mismo.** Uno es un estilo de arquitectura (quién sabe de quién); el otro es un protocolo de transporte entre dos puntos concretos. Se pueden combinar.

---

## Preguntas tipo examen

Banco completo con respuestas desarrolladas: [[07 Preguntas de Repaso - Parcial 1]].
Cada nota de clase trae además su propio bloque de preguntas al final.

## Fichas

- [[06 Fichas de Repaso]]
- [[06 Fichas de Repaso]]
- [[06 Fichas de Repaso]]

---

## Calendario de repaso espaciado

- [ ] **Lun 10 ago** — Leer completa [[02 Sistemas Distribuidos - Conceptos]], enfocándote en las 4 consecuencias. Responder sus 8 preguntas de repaso sin mirar.
- [ ] **Mar 11 ago** — Transparencia (8 tipos) + metas de diseño + falacias de Deutsch. Fichas de sistemas distribuidos.
- [x] **Mié 12 ago** — Flynn y acoplamiento. Cliente-servidor y publish-subscribe (nuevo, importado de las notas de la clase del 5 de agosto): idempotencia y simetría de HTTP. Leer [[03 Computacion en la Nube]] completa.
- [ ] **Jue 13 ago** — Repaso activo de nube: tabla de quién administra qué, ventajas/desventajas, y sus 7 preguntas.
- [ ] **Vie 14 ago** — [[04 Redes de Datos]]: HTTP, CIA/AAA y atributos no funcionales (lo más flojo de esa nota). Practicar cálculo de subredes.
- [ ] **Sáb 15 ago** — Resolver el banco [[07 Preguntas de Repaso - Parcial 1]] completo y cronometrado, sin mirar nada. Marcar lo fallado.
- [ ] **Dom 16 ago** — Solo lo que fallaste el sábado.
- [ ] **Lun 17 ago** — Repaso de las tres notas en modo rápido: tablas comparativas y listas cerradas.
- [ ] **Mar 18 ago** — Sección "comparaciones" y "errores típicos" de esta guía. Nada nuevo.
- [ ] **Mié 19 ago — PARCIAL 1**

## Enlaces

- [[Arquitecturas de Nube y Sistemas Distribuidos MOC]]
- [[02 Areas/Estudio/Semestre 6/Semestre 6 MOC]]
