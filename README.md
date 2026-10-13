# Conversion-y-Trazabilidad-ETL

Repositorio que contiene la definición de un pipeline multiagente para la
conversión y trazabilidad de procesos ETL. El pipeline recibe un
documento de requisitos (.docx) y un archivo de transacciones (.xml),
los procesa a través de una cadena de agentes especializados, y entrega
como resultado final un archivo Excel estructurado, listo para su
descarga.

El pipeline está compuesto por un agente orquestador y tres subagentes,
cada uno responsable de una fase específica del procesamiento.

## Disponibilidad

Estos agentes están disponibles en CodeMie y pueden utilizarse
directamente a través del siguiente enlace:

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

| Carpeta / Archivo                              | Descripción                                                                    |
|--------------------------------------------------|---------------------------------------------------------------------------------|
| O-Agente-BibaC-FCT-Orquestador/                  | Agente orquestador. Coordina el pipeline completo e invoca a los subagentes.    |
| O-Agente-BibaC-FCT-Docx-To_Markdown/             | Subagente que convierte el archivo .docx de requisitos a Markdown.             |
| O-Agente-BibaC-FCT-Analisis-Md-Xml/              | Subagente que cruza el Markdown con el archivo .xml y genera el análisis.       |
| O-Agente-BibaC-FCT-Markdown-Excel/               | Subagente que transforma el análisis estructurado en el archivo Excel final.   |
| plantilla_fallback_v3.md                          | Plantilla de respaldo utilizada cuando el usuario no proporciona una propia.   |
| *.md dentro de cada carpeta de agente             | System prompt / configuración del agente correspondiente.                      |
| README.md dentro de cada carpeta de agente        | Documentación específica de ese agente.                                        |

## Arquitectura del pipeline

El agente orquestador coordina a los tres subagentes en el siguiente orden:

| Orden | Agente                                 | Modelo            | Entrada                                                            | Salida                      |
|-------|------------------------------------------|--------------------|------------------------------------------------------------------------|------------------------------|
| —     | O-Agente-BibaC-FCT-Orquestador           | GPT o4-mini        | Archivos .docx y .xml del usuario (y .md opcional)                     | Coordinación del pipeline y entrega final |
| 1     | O-Agente-BibaC-FCT-Docx-To_Markdown      | Gemini Flash 3.5   | Archivo .docx de requisitos                                            | Documento Markdown           |
| 2     | O-Agente-BibaC-FCT-Analisis-Md-Xml       | GPT Terra          | Markdown del paso 1, archivo .xml, archivo .md (o fallback)            | analisis-data.md             |
| 3     | O-Agente-BibaC-FCT-Markdown-Excel        | Claude Sonnet      | analisis-data.md del paso 2                                            | Archivo .xlsx descargable    |

Cada subagente gestiona sus propias herramientas de forma autónoma a
través de Agent Workspace y no recibe instrucciones adicionales del
orquestador más allá de los archivos definidos para su fase.

## Flujo general del proceso

1. Validación previa: el orquestador comprueba que existan los archivos
   obligatorios .docx y .xml. Si falta alguno, el pipeline se detiene y
   se notifica al usuario.
2. Conversión DOCX a Markdown: el subagente
   O-Agente-BibaC-FCT-Docx-To_Markdown convierte el .docx de requisitos
   a Markdown, eliminando direcciones IP del contenido.
3. Análisis de Markdown y XML: el subagente
   O-Agente-BibaC-FCT-Analisis-Md-Xml cruza el Markdown generado con el
   archivo .xml (SmartExport de Oracle Data Integrator) y produce el
   documento analisis-data.md.
4. Generación del Excel: el subagente O-Agente-BibaC-FCT-Markdown-Excel
   transforma analisis-data.md en el archivo final resultado.xlsx.
5. Entrega final: el orquestador realiza passthrough del resultado del
   último subagente y añade una invitación a completar la encuesta de
   satisfacción del servicio.

## Archivos de entrada del pipeline

| Archivo       | Tipo             | Obligatorio | Descripción                              |
|---------------|-------------------|-------------|---------------------------------------------|
| Transacciones | .xml              | Sí          | Datos de transacciones de base de datos     |
| Requisitos    | .docx             | Sí          | Requisitos del proceso                      |
| Plantilla     | .md / .xlsx       | No          | Plantilla para el output final              |

Si falta el archivo .md de plantilla, el pipeline utiliza internamente
la plantilla de respaldo incluida en el repositorio
(plantilla_fallback_v3.md), sin notificar al usuario.

## Resultado final

El entregable del pipeline es un archivo Excel (resultado.xlsx) con
enlace de descarga real, generado a partir del cruce de información
técnica entre el documento de requisitos y el archivo de transacciones
XML.

## Documentación por agente

Cada carpeta del repositorio incluye su propio README.md con el detalle
específico de ese agente: rol, objetivo, funciones, herramientas,
reglas, entradas, salidas y modelo utilizado. Se recomienda consultar
cada uno de ellos para el detalle completo de su comportamiento:

- O-Agente-BibaC-FCT-Orquestador/README.md
- O-Agente-BibaC-FCT-Docx-To_Markdown/README.md
- O-Agente-BibaC-FCT-Analisis-Md-Xml/README.md
- O-Agente-BibaC-FCT-Markdown-Excel/README.md

## Notas de uso

- Este README describe el comportamiento funcional del pipeline tal
  como está definido en la configuración de cada agente. No constituye
  documentación de una API ni de un servicio independiente fuera de
  CodeMie.
- Todos los agentes dependen de que el Agent Workspace esté disponible
  y accesible en el entorno de ejecución para leer y escribir los
  archivos intermedios y finales del pipeline.
- El orquestador no ejecuta ninguna herramienta de lectura, escritura o
  procesamiento de archivos por sí mismo; toda la lógica de conversión,
  análisis y generación de Excel se delega exclusivamente en los
  subagentes correspondientes.
