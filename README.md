# DRIVE CON AUDIOS DE LAS CLASES Y TRANSCRIPCIONES

https://drive.google.com/drive/folders/1Q4Q1tX1Ml4JPJtA_yucEixfVEnMhRRFN?usp=drive_link

## 📚 Repositorio de notas y material de clase

### 🎯 Propósito del repositorio


Este repositorio tiene como objetivo **recopilar, organizar y compartir material de apoyo elaborado por los estudiantes durante las clases**, con el fin de facilitar el estudio, la consulta y el repaso de los temas vistos durante el curso.

El contenido está organizado principalmente **por semanas**. Dentro de cada semana se podrán encontrar, dependiendo del material disponible:

* 📝 Notas tomadas por diferentes estudiantes.
* 📸 Fotografías del tablero.
* 📄 Documentos o archivos complementarios.
* 🎙️ Transcripciones de clases.
* 📚 Otros recursos útiles para estudiar.

La idea es que este repositorio funcione como un espacio colaborativo donde diferentes estudiantes puedan aportar material y, al mismo tiempo, consultar lo recopilado por los demás.

---

## 🎙️ Transcripción automática de clases

El repositorio también incluye un archivo de Python (`.py`) que permite transcribir grabaciones de clases utilizando **Whisper `large-v3`**.

Aunque el archivo tiene extensión `.py`, está pensado para ejecutarse en **Google Colab**, principalmente porque el proceso de transcripción se beneficia del uso de una GPU.

---

## 💻 ¿Cómo ejecutar el archivo en Google Colab?

1. Ingresa a [Google Colab](https://colab.research.google.com/).

2. Crea un cuaderno nuevo.

3. Ve a:

   **Entorno de ejecución → Cambiar tipo de entorno de ejecución**

4. Selecciona **T4 GPU**, si está disponible.

5. Sube el archivo `.py` desde el panel de archivos de Google Colab.

6. En una celda nueva ejecuta:

```python
!python NOMBRE_DEL_ARCHIVO.py
```

También puedes ejecutarlo utilizando:

```python
%run NOMBRE_DEL_ARCHIVO.py
```

Al iniciar, el programa solicitará acceso a Google Drive, ya que desde allí leerá la grabación y guardará los archivos generados.

---

## ⚙️ Líneas que debes modificar

Antes de ejecutar la transcripción debes modificar **dos valores principales** dentro del archivo Python.

Estas líneas también están identificadas dentro del código con comentarios como:

```python
# <<< CAMBIAR AQUÍ >>>
```

### 1️⃣ Ruta de la carpeta en Google Drive

Busca:

```python
CARPETA_DRIVE = Path(
    "/content/drive/MyDrive/Transcripciones/NOMBRE_DE_LA_CARPETA"
)
```

Debes reemplazar:

```text
NOMBRE_DE_LA_CARPETA
```

por el nombre real de la carpeta de Google Drive donde tienes guardado el audio o video.

Por ejemplo:

```python
CARPETA_DRIVE = Path(
    "/content/drive/MyDrive/Transcripciones/Arquitectura_Nube"
)
```

> **Importante:** normalmente debes conservar `/content/drive/MyDrive/` al inicio de la ruta, ya que corresponde a **Mi unidad** dentro de Google Drive.

---

### 2️⃣ Nombre del archivo que quieres transcribir

Busca:

```python
NOMBRE_ARCHIVO_ENTRADA = "NOMBRE_DEL_ARCHIVO.mp4"
```

Reemplaza `NOMBRE_DEL_ARCHIVO.mp4` por el nombre exacto del audio o video que quieres transcribir.

Por ejemplo:

```python
NOMBRE_ARCHIVO_ENTRADA = "Clase 8.mp4"
```

Debes incluir también la extensión del archivo, por ejemplo:

```text
.mp4
.mp3
.wav
.m4a
```

---

## 📁 Ejemplo completo

Supongamos que en Google Drive tienes:

```text
Mi unidad
└── Transcripciones
    └── Arquitectura_Nube
        └── Clase 8.mp4
```

La configuración dentro del archivo Python debería quedar así:

```python
CARPETA_DRIVE = Path(
    "/content/drive/MyDrive/Transcripciones/Arquitectura_Nube"
)

NOMBRE_ARCHIVO_ENTRADA = "Clase 8.mp4"
```

Con esas dos líneas configuradas correctamente, el resto del código puede ejecutarse sin necesidad de modificar las rutas.

---

## 📄 Archivos que genera el programa

La transcripción se guarda automáticamente en la misma carpeta de Google Drive donde se encuentra la grabación.

Por cada clase se generan principalmente dos archivos:

```text
NOMBRE_DEL_ARCHIVO_Transcripcion_Fase_1.txt
NOMBRE_DEL_ARCHIVO_progreso.jsonl
```

### Archivo de transcripción

```text
NOMBRE_DEL_ARCHIVO_Transcripcion_Fase_1.txt
```

Contiene el texto transcrito de la clase.

### Archivo de progreso

```text
NOMBRE_DEL_ARCHIVO_progreso.jsonl
```

Guarda el avance de la transcripción.

Esto permite que, si Google Colab se desconecta o el proceso se interrumpe, se pueda continuar posteriormente sin comenzar toda la transcripción desde cero.

---

## 🧠 Configuración opcional

Dentro del archivo también encontrarás dos variables:

```python
PROMPT_INICIAL_CURSO
VOCABULARIO_TECNICO_CURSO
```

Estas variables **no necesitan modificarse obligatoriamente** para ejecutar el programa.

Sirven para darle a Whisper información sobre el contexto de la clase y sobre palabras técnicas que pueden aparecer durante la transcripción.

Por ejemplo:

```text
AWS
Docker
Kubernetes
TCP/IP
DNS
API REST
Git
GitHub
```

Si el programa se utiliza para otra asignatura, estas variables pueden adaptarse con términos propios de esa materia para mejorar el reconocimiento del vocabulario técnico.

---

## 🤝 Recomendaciones para aportar material

Para mantener el repositorio organizado:

1. Verifica primero a qué **semana** pertenece el material.
2. Guarda las notas, fotografías o documentos en la carpeta correspondiente.
3. Utiliza nombres de archivos claros.
4. Evita eliminar material aportado por otros estudiantes.
5. Antes de subir nuevos cambios, ejecuta:

```bash
git pull
```

Esto permite descargar primero los cambios realizados por otras personas.

Después puedes subir tu material normalmente con:

```bash
git add .
git commit -m "Agregar notas semana X"
git push
```

---

## 📌 Objetivo final

El propósito de este repositorio es construir, entre todos, un **material de estudio organizado, colaborativo y fácil de consultar**, reuniendo diferentes formas de tomar notas y registrar lo visto durante las clases.

La idea no es reemplazar la asistencia ni el trabajo individual de cada estudiante, sino complementar el proceso de estudio con los aportes del grupo.
