# Conversion-y-Trazabilidad-ETL

Repositorio que contiene la definicion de un pipeline multiagente para la
conversion y trazabilidad de procesos ETL. El pipeline recibe un
documento de requisitos (.docx) y un archivo de transacciones (.xml),
los procesa a traves de una cadena de agentes especializados, y entrega
como resultado final un archivo Excel estructurado, listo para su
descarga.

El pipeline esta compuesto por un agente orquestador y tres subagentes,
cada uno responsable de una fase especifica del procesamiento.

## Disponibilidad

Estos agentes estan disponibles en CodeMie y pueden utilizarse
directamente a traves del siguiente enlace:

https://codemie.lab.epam.com/assistants/conversin-y-trazabilidad-etl-v21/start

## Estructura del repositorio

Conversion-y-Trazabilidad-ETL/
├── O-Agente-BibaC-FCT-Analisis-Md-Xml/
│   ├── O-Agente-BibaC-FCT-Analisis-Md-Xml.md
│   ├── plantilla_fallback_v3.md
│   └── README.md
├── O-Agente-BibaC-FCT-Docx-To_Markdown/
│   ├── O-Agente-BibaC-FCT-Docx-To_Markdown.md
│   └── README.md
├── O-Agente-BibaC-FCT-Markdown-Excel/
│   ├── O-Agente-BibaC-FCT-Markdown-Excel.md
│   └── README.md
└── O-Agente-BibaC-FCT-Orquestador/
    ├── O-Agente-BibaC-FCT-Orquestador.md
    └── README.md

| Carpeta / Archivo                              | Descripcion                                                                 |
|--------------------------------------------------|--------------------------------------------------------------------------------|
| O-Agente-BibaC-FCT-Orquestador/                  | Agente orquestador. Coordina el pipeline completo e invoca a los subagentes. |
| O-Agente-BibaC-FCT-Docx-To_Markdown/             | Subagente que convierte el archivo .docx de requisitos a Markdown.          |
| O-Agente-BibaC-FCT-Analisis-Md-Xml/              | Subagente que cruza el Markdown con el archivo .xml y genera el analisis.   |
| O-Agente-BibaC-FCT-Markdown-Excel/               | Subagente que transforma el analisis estructurado en el archivo Excel final.|
| plantilla_fallback_v3.md                          | Plantilla de respaldo utilizada cuando el usuario no proporciona una propia. |
| *.md dentro de cada carpeta de agente             | System prompt / configuracion del agente correspondiente.                   |
| README.md dentro de cada carpeta de agente        | Documentacion especifica de ese agente.                                     |

## Arquitectura del pipeline

El agente orquestador coordina a los tres subagentes en el siguiente orden:

| Orden | Agente                                 | Modelo            | Entrada                                                          | Salida                      |
|-------|------------------------------------------|--------------------|---------------------------------------------------------------------|------------------------------|
| —     | O-Agente-BibaC-FCT-Orquestador           | GPT o4-mini        | Archivos .docx y .xml del usuario (y .md opcional)                  | Coordinacion del pipeline y entrega final |
| 1     | O-Agente-BibaC-FCT-Docx-To_Markdown      | Gemini Flash 3.5   | Archivo .docx de requisitos                                         | Documento Markdown           |
| 2     | O-Agente-BibaC-FCT-Analisis-Md-Xml       | GPT Terra          | Markdown del paso 1, archivo .xml, archivo .md (o fallback)         | analisis-data.md             |
| 3     | O-Agente-BibaC-FCT-Markdown-Excel        | Claude Sonnet      | analisis-data.md del paso 2                                         | Archivo .xlsx descargable    |

Cada subagente gestiona sus propias herramientas de forma autonoma a
traves de Agent Workspace y no recibe instrucciones adicionales del
orquestador mas alla de los archivos definidos para su fase.

## Flujo general del proceso

1. Validacion previa: el orquestador comprueba que existan los archivos
   obligatorios .docx y .xml. Si falta alguno, el pipeline se detiene y
   se notifica al usuario.
2. Conversion DOCX a Markdown: el subagente
   O-Agente-BibaC-FCT-Docx-To_Markdown convierte el .docx de requisitos
   a Markdown, eliminando direcciones IP del contenido.
3. Analisis de Markdown y XML: el subagente
   O-Agente-BibaC-FCT-Analisis-Md-Xml cruza el Markdown generado con el
   archivo .xml (SmartExport de Oracle Data Integrator) y produce el
   documento analisis-data.md.
4. Generacion del Excel: el subagente O-Agente-BibaC-FCT-Markdown-Excel
   transforma analisis-data.md en el archivo final resultado.xlsx.
5. Entrega final: el orquestador realiza passthrough del resultado del
   ultimo subagente y añade una invitacion a completar la encuesta de
   satisfaccion del servicio.

## Archivos de entrada del pipeline

| Archivo       | Tipo             | Obligatorio | Descripcion                              |
|---------------|-------------------|-------------|--------------------------------------------|
| Transacciones | .xml              | Si          | Datos de transacciones de base de datos    |
| Requisitos    | .docx             | Si          | Requisitos del proceso                     |
| Plantilla     | .md / .xlsx       | No          | Plantilla para el output final             |

Si falta el archivo .md de plantilla, el pipeline utiliza internamente
la plantilla de respaldo incluida en el repositorio
(plantilla_fallback_v3.md), sin notificar al usuario.

## Resultado final

El entregable del pipeline es un archivo Excel (resultado.xlsx) con
enlace de descarga real, generado a partir del cruce de informacion
tecnica entre el documento de requisitos y el archivo de transacciones
XML.

## Documentacion por agente

Cada carpeta del repositorio incluye su propio README.md con el detalle
especifico de ese agente: rol, objetivo, funciones, herramientas,
reglas, entradas, salidas y modelo utilizado. Se recomienda consultar
cada uno de ellos para el detalle completo de su comportamiento:

- O-Agente-BibaC-FCT-Orquestador/README.md
- O-Agente-BibaC-FCT-Docx-To_Markdown/README.md
- O-Agente-BibaC-FCT-Analisis-Md-Xml/README.md
- O-Agente-BibaC-FCT-Markdown-Excel/README.md

## Notas de uso

- Este README describe el comportamiento funcional del pipeline tal
  como esta definido en la configuracion de cada agente. No constituye
  documentacion de una API ni de un servicio independiente fuera de
  CodeMie.
- Todos los agentes dependen de que el Agent Workspace este disponible
  y accesible en el entorno de ejecucion para leer y escribir los
  archivos intermedios y finales del pipeline.
- El orquestador no ejecuta ninguna herramienta de lectura, escritura o
  procesamiento de archivos por si mismo; toda la logica de conversion,
  analisis y generacion de Excel se delega exclusivamente en los
  subagentes correspondientes.
