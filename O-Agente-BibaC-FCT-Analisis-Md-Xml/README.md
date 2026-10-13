# Agente de Analisis Markdown y XML

Agente especializado en extraccion, estandarizacion y poblacion de
informacion tecnica de trazabilidad ETL. Forma parte de un pipeline
multiagente, situandose como el eslabon intermedio encargado de cruzar
los requisitos del proceso (en formato Markdown) con los metadatos
tecnicos de un SmartExport de Oracle Data Integrator (ODI) en formato
XML, produciendo un documento de analisis estructurado.

## Descripcion general

Este agente no genera el entregable final, sino un documento intermedio
de analisis (`analisis-data.md`) que sirve de insumo al agente generador
de Excel. Su funcion es leer los documentos del Agent Workspace, extraer
la informacion tecnica del XML mediante codigo Python, cruzarla con el
contenido de los archivos Markdown disponibles y estructurar el
resultado en tablas Markdown pre-pobladas, listas para su posterior
transformacion.

## Modelo

Este agente utiliza el modelo **GPT Terra** como motor de razonamiento
para interpretar las reglas de extraccion, cruzar la informacion tecnica
y estructurar el documento de analisis final.

## Objetivo

1. Leer los documentos de entrada desde el Agent Workspace usando las
   herramientas disponibles, con la excepcion del archivo XML, el cual
   debe leerse y procesarse mediante codigo Python.
2. Procesar dichos documentos siguiendo estrictamente las reglas
   definidas en su configuracion.
3. Escribir el resultado final en el Agent Workspace bajo el nombre de
   archivo `analisis-data.md`.

## Funciones principales

- Procesar y analizar informacion proveniente de un archivo XML
  correspondiente a un SmartExport de Oracle Data Integrator (ODI),
  reconociendo objetos representados como elementos `<Object>` con
  atributo `class` y campos como elementos `<Field>`.
- Extraer, estandarizar y cruzar la informacion tecnica con los datos
  provenientes de los archivos Markdown del workspace.
- Generar y estructurar la salida en forma de tablas Markdown
  pre-pobladas, con cabeceras especificas, rellenando los valores con
  datos del XML o indicando `[del .md]` segun corresponda.

## Herramientas utilizadas

| Herramienta                      | Uso                                                                                      |
|------------------------------------|---------------------------------------------------------------------------------------------|
| Herramientas de Agent Workspace    | Leer los documentos de entrada (archivos `.md`) y escribir el archivo de salida final `analisis-data.md`. |
| Interprete de codigo Python        | Leer, procesar e interpretar exclusivamente el archivo XML, con posibilidad de reintentos si se requiere. |

## Reglas de procesamiento

- El archivo XML de entrada es siempre un SmartExport de Oracle Data
  Integrator (ODI), donde los objetos se representan como
  `<Object class="...">` y sus campos como `<Field name="..." type="...">`.
- Esta prohibido producir el contenido de salida como texto directamente
  en el chat.
- Toda lectura de archivos `.md` y toda escritura del archivo final
  ocurren exclusivamente a traves de las herramientas del Agent
  Workspace.
- La lectura y procesamiento del archivo `.xml` se realizan de forma
  exclusiva mediante codigo Python.

## Entradas

- Documentos de entrada ubicados en el Agent Workspace, incluyendo
  archivos `.md`.
- Un archivo XML de entrada correspondiente a un SmartExport de Oracle
  Data Integrator (ODI).

## Salidas

- Un archivo final llamado `analisis-data.md`, escrito exclusivamente en
  el Agent Workspace mediante las herramientas correspondientes,
  conteniendo las tablas estructuradas de analisis y trazabilidad.

## Comportamiento esperado

- El agente nunca expone el contenido del analisis directamente como
  texto en la conversacion; el resultado se entrega unicamente mediante
  la escritura del archivo en el Agent Workspace.
- El archivo XML nunca se lee con herramientas del workspace; su
  procesamiento se realiza siempre mediante codigo Python.
- La informacion de salida respeta siempre la estructura de tablas
  Markdown definida, marcando con `[del .md]` los valores que provienen
  del Markdown en lugar del XML.

## Rol dentro del pipeline

Este agente actua como el segundo eslabon de un pipeline multiagente de
procesamiento documental. Recibe el Markdown generado a partir del
`.docx` original junto con el archivo `.xml` de transacciones, y entrega
un documento de analisis estructurado que sera consumido por el agente
generador de Excel en la fase final del proceso.

## Notas de uso

- Este README describe el comportamiento funcional del agente tal como
  esta definido en su configuracion. No constituye documentacion de una
  API ni de un servicio ejecutable de forma independiente.
- El agente depende de que el Agent Workspace este disponible y
  accesible en el entorno de ejecucion, tanto para leer los archivos
  `.md` como para escribir el archivo `analisis-data.md`.
- El interprete de codigo Python debe estar habilitado en el entorno de
  ejecucion para que el agente pueda procesar correctamente el archivo
  XML.