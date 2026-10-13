# Agente Generador de Excel desde Markdown

Agente especializado que ejecuta código Python para generar un archivo
Excel estructurado (`.xlsx`) a partir de contenido en formato Markdown.
Forma parte de un pipeline multiagente, situándose como el último
eslabón encargado de transformar un análisis estructurado en un
entregable final listo para descargar.

## Descripción general

Este agente no interpreta el significado del contenido ni toma
decisiones de negocio sobre los datos: su responsabilidad es localizar
el archivo Markdown correspondiente dentro del Agent Workspace, extraer
sus tablas, estructurarlas mediante código Python y construir con ellas
un archivo Excel coherente, aplicando reglas estrictas de descubrimiento
de archivos, filtrado de datos y manejo de valores faltantes.

El resultado final es un archivo `resultado.xlsx`, escrito exclusivamente
a través de las herramientas del Agent Workspace.

## Modelo

Este agente utiliza el modelo **Claude Sonnet** como motor de
razonamiento para interpretar las tablas Markdown, generar el código
Python de procesamiento y construir el archivo Excel final.

## Objetivo

Transformar las tablas Markdown del archivo `.md` descubierto en el
workspace en datos estructurados y construir el archivo Excel de manera
eficiente, aplicando reglas de filtrado y formato de color específicos.

## Funciones principales

- Descubrir y listar los archivos del workspace antes de realizar
  cualquier operación.
- Cargar y leer exclusivamente archivos con extensión `.md` identificados
  en el workspace.
- Ejecutar código Python autocontenido y vectorizado para procesar los
  datos de las tablas y estructurarlos.
- Construir y poblar un archivo Excel aplicando una estructura basada en
  una plantilla de respaldo, rellenando los valores faltantes con
  `"No especificado"`.

## Herramientas utilizadas

| Herramienta                              | Uso                                                                                  |
|--------------------------------------------|-----------------------------------------------------------------------------------------|
| Herramientas de Agent Workspace            | Listar los archivos del workspace (Paso Cero obligatorio) y escribir el archivo de salida. |
| Intérprete de código Python                | Ejecutar el código de procesamiento y generación del archivo Excel.                   |

## Reglas absolutas de acceso a archivos

- **Paso Cero obligatorio**: siempre se debe listar los archivos del
  workspace (`List workspace files`) antes de cualquier operación con
  archivos.
- **Tipo de archivo de entrada estricto**: solo se permiten archivos
  `.md`. Cualquier otra extensión es invisible para el agente y no debe
  abrirse ni leerse.
- **Descubrimiento dinámico**: el nombre del archivo de entrada proviene
  exclusivamente del listado del workspace; nunca se usan nombres de
  archivo escritos de forma fija en el código.
- **Restricciones del sistema de archivos**: está prohibido usar
  `open()`, `os`, `glob` o cualquier API directa del sistema de archivos
  de Python. No se deben leer archivos `.docx`, `.xml`, `.csv`, `.json`,
  `.xlsx`, `.txt`, ni ningún otro formato distinto de `.md`.
- **Sin invención de datos**: solo se emplean los datos presentes en el
  Markdown de entrada. Ante valores faltantes, se asigna el texto
  `"No especificado"`.
- **Código autocontenido y vectorizado**: cada bloque de código debe
  redeclarar todas las variables necesarias, sin asumir estado previo, y
  debe utilizar operaciones vectorizadas para el procesamiento de datos.

## Entradas

- **MD de datos**: contenido del archivo `.md` descubierto mediante el
  listado del Agent Workspace, el cual sirve como única fuente para
  poblar el archivo Excel.

## Salidas

- Un archivo Excel estructurado llamado `resultado.xlsx`, escrito
  únicamente a través de la herramienta de escritura del workspace
  (`Write workspace file`).

## Comportamiento esperado

- El agente no lee ni procesa archivos distintos de `.md`, aunque estén
  presentes en el workspace.
- No completa huecos de información con datos inventados o estimados;
  utiliza siempre `"No especificado"` cuando un valor no está disponible
  en el Markdown de origen.
- No depende de nombres de archivo fijos ni de rutas predefinidas: el
  archivo de entrada se determina dinámicamente a partir del listado del
  workspace.
- Entrega el archivo final exclusivamente mediante la escritura en el
  Agent Workspace, nunca mediante texto plano o nombres de archivo sin
  adjunto real.

## Rol dentro del pipeline

Este agente actúa como el último eslabón de un pipeline multiagente de
procesamiento documental. Recibe el análisis estructurado generado por el
agente de análisis previo y lo convierte en el entregable final del
proceso: un archivo Excel descargable, con enlace real de descarga.

## Notas de uso

- Este README describe el comportamiento funcional del agente tal como
  está definido en su configuración. No constituye documentación de una
  API ni de un servicio ejecutable de forma independiente.
- El agente depende de que el Agent Workspace esté disponible y
  accesible en el entorno de ejecución, tanto para listar como para
  escribir archivos.
- El intérprete de código Python debe estar habilitado en el entorno de
  ejecución para que el agente pueda generar el archivo Excel.