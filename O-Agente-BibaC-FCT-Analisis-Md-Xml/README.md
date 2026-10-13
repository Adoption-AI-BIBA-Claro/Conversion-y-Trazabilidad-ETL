# Agente de Análisis Markdown y XML

Agente especializado en extracción, estandarización y población de
información técnica de trazabilidad ETL. Forma parte de un pipeline
multiagente, situándose como el eslabón intermedio encargado de cruzar
los requisitos del proceso (en formato Markdown) con los metadatos
técnicos de un SmartExport de Oracle Data Integrator (ODI) en formato
XML, produciendo un documento de análisis estructurado.

## Descripción general

Este agente no genera el entregable final, sino un documento intermedio
de análisis (`analisis-data.md`) que sirve de insumo al agente generador
de Excel. Su función es leer los documentos del Agent Workspace, extraer
la información técnica del XML mediante código Python, cruzarla con el
contenido de los archivos Markdown disponibles y estructurar el
resultado en tablas Markdown pre-pobladas, listas para su posterior
transformación.

## Modelo

Este agente utiliza el modelo **GPT Terra** como motor de razonamiento
para interpretar las reglas de extracción, cruzar la información técnica
y estructurar el documento de análisis final.

## Objetivo

1. Leer los documentos de entrada desde el Agent Workspace usando las
   herramientas disponibles, con la excepción del archivo XML, el cual
   debe leerse y procesarse mediante código Python.
2. Procesar dichos documentos siguiendo estrictamente las reglas
   definidas en su configuración.
3. Escribir el resultado final en el Agent Workspace bajo el nombre de
   archivo `analisis-data.md`.

## Funciones principales

- Procesar y analizar información proveniente de un archivo XML
  correspondiente a un SmartExport de Oracle Data Integrator (ODI),
  reconociendo objetos representados como elementos `<Object>` con
  atributo `class` y campos como elementos `<Field>`.
- Extraer, estandarizar y cruzar la información técnica con los datos
  provenientes de los archivos Markdown del workspace.
- Generar y estructurar la salida en forma de tablas Markdown
  pre-pobladas, con cabeceras específicas, rellenando los valores con
  datos del XML o indicando `[del .md]` según corresponda.

## Herramientas utilizadas

| Herramienta                      | Uso                                                                                       |
|------------------------------------|----------------------------------------------------------------------------------------------|
| Herramientas de Agent Workspace    | Leer los documentos de entrada (archivos `.md`) y escribir el archivo de salida final `analisis-data.md`. |
| Intérprete de código Python        | Leer, procesar e interpretar exclusivamente el archivo XML, con posibilidad de reintentos si se requiere. |

## Reglas de procesamiento

- El archivo XML de entrada es siempre un SmartExport de Oracle Data
  Integrator (ODI), donde los objetos se representan como
  `<Object class="...">` y sus campos como `<Field name="..." type="...">`.
- Está prohibido producir el contenido de salida como texto directamente
  en el chat.
- Toda lectura de archivos `.md` y toda escritura del archivo final
  ocurren exclusivamente a través de las herramientas del Agent
  Workspace.
- La lectura y procesamiento del archivo `.xml` se realizan de forma
  exclusiva mediante código Python.

## Entradas

- Documentos de entrada ubicados en el Agent Workspace, incluyendo
  archivos `.md`.
- Un archivo XML de entrada correspondiente a un SmartExport de Oracle
  Data Integrator (ODI).

## Salidas

- Un archivo final llamado `analisis-data.md`, escrito exclusivamente en
  el Agent Workspace mediante las herramientas correspondientes,
  conteniendo las tablas estructuradas de análisis y trazabilidad.

## Comportamiento esperado

- El agente nunca expone el contenido del análisis directamente como
  texto en la conversación; el resultado se entrega únicamente mediante
  la escritura del archivo en el Agent Workspace.
- El archivo XML nunca se lee con herramientas del workspace; su
  procesamiento se realiza siempre mediante código Python.
- La información de salida respeta siempre la estructura de tablas
  Markdown definida, marcando con `[del .md]` los valores que provienen
  del Markdown en lugar del XML.

## Rol dentro del pipeline

Este agente actúa como el segundo eslabón de un pipeline multiagente de
procesamiento documental. Recibe el Markdown generado a partir del
`.docx` original junto con el archivo `.xml` de transacciones, y entrega
un documento de análisis estructurado que será consumido por el agente
generador de Excel en la fase final del proceso.

## Notas de uso

- Este README describe el comportamiento funcional del agente tal como
  está definido en su configuración. No constituye documentación de una
  API ni de un servicio ejecutable de forma independiente.
- El agente depende de que el Agent Workspace esté disponible y
  accesible en el entorno de ejecución, tanto para leer los archivos
  `.md` como para escribir el archivo `analisis-data.md`.
- El intérprete de código Python debe estar habilitado en el entorno de
  ejecución para que el agente pueda procesar correctamente el archivo
  XML.
