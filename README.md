# Orquestador-FCT-ETL

Agente orquestador que coordina de forma autónoma un pipeline de tres
subagentes para procesar archivos de entrada (XML y DOCX) y producir un
archivo Excel final con los datos de análisis estructurados.

## Descripción general

`Orquestador-FCT-ETL` no procesa ni transforma archivos por sí mismo. Su
única responsabilidad es invocar, en el orden correcto, a tres subagentes
especializados y entregar al usuario el resultado final generado por el
último de ellos. El pipeline se ejecuta en un único ciclo, sin interacción
adicional con el usuario una vez iniciada la tarea.

El flujo completo es el siguiente:

1. Validación previa de archivos de entrada (Fase 0).
2. Conversión del documento de requisitos (.docx) a Markdown.
3. Análisis cruzado del Markdown generado junto con el archivo de
   transacciones (.xml) y, opcionalmente, una plantilla (.md).
4. Generación del archivo Excel final a partir del análisis estructurado.
5. Entrega del resultado al usuario junto con una solicitud de encuesta de
   satisfacción.

## Arquitectura del pipeline

El agente coordina los siguientes subagentes, cada uno responsable de una
fase específica:

| Orden | Subagente                                  | Entrada                                                        | Salida                      |
|-------|---------------------------------------------|-----------------------------------------------------------------|------------------------------|
| 1     | `Agente-BibaC-FCT-Docx-To_Markdown_V2`       | Archivo `.docx` de requisitos                                   | Documento Markdown           |
| 2     | `Agente-BibaC-FCT-Analisis-Md-Xml_V2`        | Markdown del paso 1, archivo `.xml`, archivo `.md` (o fallback) | `analisis-data.md`           |
| 3     | `Agente-BibaC-FCT-Markdown-Excel_V2`         | `analisis-data.md` del paso 2                                   | Archivo `.xlsx` descargable  |

Cada subagente gestiona sus propias herramientas de forma autónoma
(Agent Workspace) y no recibe instrucciones adicionales del orquestador
más allá de los archivos definidos para su fase.

## Archivos de entrada

| Archivo       | Tipo        | Obligatorio | Descripción                              |
|---------------|-------------|-------------|-------------------------------------------|
| Transacciones | `.xml`      | Sí          | Datos de transacciones de base de datos   |
| Requisitos    | `.docx`     | Sí          | Requisitos del proceso                    |
| Plantilla     | `.md` / `.xlsx` | No      | Plantilla para el output final            |

Si falta el archivo `.docx` o el archivo `.xml`, el pipeline se detiene
antes de invocar cualquier subagente y se notifica al usuario el archivo
faltante.

Si falta el archivo `.md` de plantilla, el pipeline continúa utilizando
internamente un valor `fallback_predeterminado`, sin notificar al usuario.

## Variables de sistema

| Variable            | Descripción                                                                                           |
|----------------------|---------------------------------------------------------------------------------------------------------|
| `link_encuesta`      | URL de la encuesta de satisfacción. Se inyecta en el system prompt antes de la ejecución y se usa de forma literal, sin modificaciones. |

## Fases del pipeline

### Fase 0 — Validación previa

Comprueba que existan los archivos `.docx` y `.xml` obligatorios antes de
iniciar cualquier procesamiento. Si falta alguno de ellos, el pipeline se
aborta de inmediato.

### Fase 1 — Conversión DOCX a Markdown

Se invoca a `Agente-BibaC-FCT-Docx-To_Markdown_V2` adjuntando únicamente el
archivo `.docx`. El orquestador nunca realiza esta conversión por su
cuenta, aunque disponga técnicamente de herramientas capaces de hacerlo.

Política de reintentos: hasta 3 intentos automáticos si el subagente falla
o devuelve un output vacío o malformado. Si los 3 intentos fallan, el
pipeline se aborta y se notifica el error.

### Fase 2 — Análisis de Markdown y XML

Se invoca a `Agente-BibaC-FCT-Analisis-Md-Xml_V2` adjuntando el Markdown
generado en la Fase 1, el archivo `.xml` original y el archivo `.md` de
plantilla (o el valor `fallback_predeterminado` si no existe). El
orquestador nunca realiza este análisis por su cuenta.

Política de reintentos: hasta 3 intentos automáticos. Si los 3 intentos
fallan, el pipeline se aborta y se notifica el error.

### Fase 3 — Generación del Excel

Se invoca a `Agente-BibaC-FCT-Markdown-Excel_V2` adjuntando únicamente el
archivo `analisis-data.md` producido en la Fase 2. Se valida que el
resultado incluya un enlace de descarga real del archivo `.xlsx`, no solo
un nombre de archivo en texto.

Política de reintentos: hasta 3 intentos automáticos, incluyendo el caso
en que el subagente devuelva únicamente un nombre de archivo sin enlace de
descarga. Si los 3 intentos fallan, el pipeline se aborta y se notifica el
error.

### Fase 4 — Entrega final

La respuesta final al usuario se compone exclusivamente de dos partes, en
este orden:

1. **Passthrough obligatorio**: el output íntegro de
   `Agente-BibaC-FCT-Markdown-Excel_V2`, sin modificaciones, resúmenes ni
   texto adicional.
2. **Párrafo de encuesta y cierre**: una invitación a completar la encuesta
   de satisfacción usando el enlace exacto de `link_encuesta`, seguida de
   la línea de cierre fija:
   `¡Gracias por usar el agente de conversión y trazabilidad ETL!`

El orquestador no redacta ningún otro texto propio en esta fase.

## Gestión de errores

| Situación                                          | Acción                                                               |
|-----------------------------------------------------|------------------------------------------------------------------------|
| Falta `.docx` o `.xml`                              | Abortar antes del pipeline. Notificar al usuario qué archivo falta    |
| Subagente falla en un intento                       | Reintentar automáticamente sin notificar al usuario                   |
| Subagente falla los 3 intentos                      | Abortar pipeline. Notificar paso fallido y error recibido             |
| Falta archivo `.md`                                 | Usar `fallback_predeterminado`. Continuar sin notificar al usuario    |
| Output de subagente vacío o malformado              | Tratar como fallo y aplicar política de reintentos                    |
| Excel entregado sin enlace de descarga              | Tratar como fallo. Aplicar reintentos de la Fase 3                    |
| Nombre de archivo escrito en texto sin adjunto      | Prohibido. Tratar como fallo y aplicar reintentos de la Fase 3        |

## Restricciones de diseño

Este agente está construido bajo un conjunto estricto de reglas que
garantizan que el procesamiento real de los archivos ocurra siempre en los
subagentes especializados, y nunca en el orquestador:

- No se realizan preguntas al usuario una vez iniciada la tarea.
- No se omite ni se reordena ningún paso del pipeline.
- No se adjuntan archivos innecesarios a ningún subagente.
- No se convierte el `.docx` a Markdown por cuenta propia.
- No se analiza el `.xml` ni el Markdown por cuenta propia.
- No se genera el archivo Excel por cuenta propia.
- No se usan herramientas propias de lectura, escritura o ejecución de
  código para resolver ninguna fase del pipeline; la única acción
  permitida en las Fases 1 a 3 es invocar al subagente correspondiente.
- No se reportan avances intermedios al usuario; solo se comunica el
  resultado final o un error crítico.
- No se redacta texto propio de éxito; la entrega final depende
  exclusivamente del passthrough del output de
  `Agente-BibaC-FCT-Markdown-Excel_V2`.

## Resultado esperado

Al finalizar correctamente el pipeline, el usuario recibe:

- El archivo `.xlsx` final con enlace de descarga, generado por
  `Agente-BibaC-FCT-Markdown-Excel_V2`.
- Una invitación para completar la encuesta de satisfacción del agente.

## Notas de uso

- Este README describe el comportamiento funcional del agente tal como
  está definido en su configuración. No constituye documentación de una
  API ni de un servicio ejecutable de forma independiente.
- Los subagentes referenciados (`Agente-BibaC-FCT-Docx-To_Markdown_V2`,
  `Agente-BibaC-FCT-Analisis-Md-Xml_V2`,
  `Agente-BibaC-FCT-Markdown-Excel_V2`) deben estar disponibles en el
  mismo entorno de ejecución para que el pipeline funcione correctamente.
