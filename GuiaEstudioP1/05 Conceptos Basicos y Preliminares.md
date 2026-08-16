---
tags: [estudio/nube, semestre-6]
created: 2026-07-15
updated: 2026-08-10
materia: Arquitecturas de Nube y Sistemas Distribuidos
---

# 05 Conceptos Basicos y Preliminares

> Vocabulario base del curso. Es lo primero que hay que tener claro porque todo lo demas se construye encima.
>
> **La idea que abre la materia**: la nube es el medio por el cual creamos e instanciamos los sistemas distribuidos.

> Info administrativa del curso (objetivo, prerrequisitos, bibliografia, evaluacion, fechas) esta en [[Arquitecturas de Nube y Sistemas Distribuidos MOC]].

## CONCEPTOS BASICOS

Estos cuatro son la cadena completa de cómo dos programas en máquinas distintas terminan hablándose. Vale la pena verlos como una secuencia, no como cuatro definiciones sueltas.

### Programa

**Qué es**: el código fuente que se escribe y se compila. Es algo **estático** — un archivo guardado en disco que no está haciendo nada.

**Ejemplo**: `servidor.exe` en tu disco duro, o el `.py` que acabas de escribir.

### Proceso

**Qué es**: un programa **en ejecución**. Es lo dinámico: tiene memoria asignada, un estado, un identificador (PID) y lo gestiona el sistema operativo.

**La diferencia con programa**: el programa es la receta; el proceso es alguien efectivamente cocinando. Y de un mismo programa pueden nacer **varios procesos a la vez**, cada uno con su propia memoria — por eso puedes abrir tres ventanas del mismo navegador y que una se cierre sin tumbar las otras.

**Por qué importa en este curso**: un sistema distribuido es, por definición, **procesos que se comunican**. Los procesos son los actores; todo lo demás del semestre es cómo se coordinan.

### Mensaje

**Qué es**: la unidad de comunicación entre procesos. Es la información completa que un proceso le quiere hacer llegar a otro.

**Por qué existe**: como los procesos en máquinas distintas **no comparten memoria** (consecuencia central de [[02 Sistemas Distribuidos - Conceptos]]), la única forma de coordinarse es mandándose mensajes.

**Ejemplo**: la petición HTTP completa que tu navegador le manda a un servidor.

### Paquete

**Qué es**: una **porción** del mensaje, del tamaño adecuado para viajar por la red. Un mensaje se parte en varios paquetes, y del otro lado se reensambla.

**Por qué existe**: mandar un mensaje grande como una sola pieza es mala idea. Si se corrompe en el camino, habría que retransmitirlo entero. Partido en paquetes, solo se retransmite el pedazo dañado. Además permite que varios mensajes compartan el enlace intercalándose, en vez de que uno grande bloquee a todos.

**Consecuencia para sistemas distribuidos**: los paquetes de un mismo mensaje pueden tomar **rutas distintas** y llegar **desordenados**, o no llegar. Esa es la razón técnica de por qué la falacia "la red es confiable" es falsa.

**Jerarquía que hay que tener clara**:

```
Programa (estático, en disco)
   └─ se ejecuta y se convierte en → Proceso
         └─ se comunica con otro proceso mediante → Mensaje
               └─ se parte en → Paquetes que viajan por la red
```

**Error típico**: usar "programa" y "proceso" como sinónimos, o decir que un paquete es lo mismo que un mensaje. Un mensaje **se compone de** varios paquetes.

## CONCEPTOS PRELIMINARES

Términos que el curso da por conocidos y sobre los que se construye todo lo demás:

- **Sistema operativo** — el software que administra el hardware y les reparte recursos (CPU, memoria, disco, red) a los procesos. Es quien decide qué proceso corre y cuándo.
- **Código fuente** — el texto que escribe el programador, legible para humanos.
- **Compilador / intérprete** — traducen el código fuente a algo que la máquina ejecuta. El **compilador** traduce todo de una vez y genera un ejecutable (C, Java); el **intérprete** traduce y ejecuta línea por línea en tiempo real (Python, JavaScript).
- **Programación de sistemas** — desarrollo de software que interactúa directamente con el hardware o el sistema operativo, en lugar de con el usuario final.
- **Servicio** — programa que corre en segundo plano esperando solicitudes, normalmente escuchando en un puerto. Ver detalle en [[04 Redes de Datos]].
- **Requerimientos de ejecución** — lo que un programa necesita para poder correr: versión de sistema operativo, librerías, runtime, memoria mínima. Es justamente lo que los contenedores (Docker) vinieron a empaquetar para que "funciona en mi máquina" deje de ser un problema.
- **Data Center** — instalación física que aloja servidores, almacenamiento y red, con energía y refrigeración redundantes. **On-premise** significa que ese datacenter está en las instalaciones de la propia organización, en contraste con estar en la nube.
- **Historial / bitácoras (logs)** — registro de los eventos que ocurren en un sistema. Son la base de la observabilidad: sin logs, un fallo en producción es imposible de reconstruir.
- **Aplicación web** — aplicación a la que se accede por navegador, sin instalar nada en el equipo del usuario.
- **Red** — el medio por el que se comunican los dispositivos.
- **Almacenamiento** — dónde se guardan los datos. Puede ser **estructurado** (bases de datos) o **sistema de archivos** (carpetas y archivos).
- **API** (*Application Programming Interface*) — el contrato que expone un componente para que otros lo usen sin conocer su interior. Define qué operaciones ofrece, qué recibe y qué devuelve. Es lo que hace posible la **apertura** y la **heterogeneidad** en sistemas distribuidos: dos servicios escritos en lenguajes distintos se integran porque respetan la misma API.

## SERVICIOS DE TI

Los tres recursos fundamentales que cualquier infraestructura provee — y los tres que se alquilan en la nube:

- **Cómputo (CPU)** — capacidad de procesamiento. Es lo que se consume al ejecutar procesos. Se entrega como máquinas virtuales o contenedores (Docker, Kubernetes).
- **Almacenamiento (disco)** — dónde persisten los datos:
  - **Estructurado** — bases de datos.
  - **Sistema de archivos** — carpetas y archivos.
- **Red** — la conectividad que permite que todo lo anterior se comunique.

Toda oferta de nube, por compleja que parezca, se reduce a vender combinaciones de estos tres.

## Enlaces

- [[01 Guia de Repaso - Parcial 1]]
- [[02 Sistemas Distribuidos - Conceptos]]
- [[03 Computacion en la Nube]]
- [[04 Redes de Datos]]
- [[Arquitecturas de Nube y Sistemas Distribuidos MOC]]
