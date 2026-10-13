# Orquestador-FCT-ETL

Agente orquestador que coordina de forma autonoma un pipeline de tres
subagentes para procesar archivos de entrada (XML y DOCX) y producir un
archivo Excel final con los datos de analisis estructurados.

## Descripcion general

`Orquestador-FCT-ETL` no procesa ni transforma archivos por si mismo. Su
unica responsabilidad es invocar, en el orden correcto, a tres subagentes
especializados y entregar al usuario el resultado final generado por el
ultimo de ellos. El pipeline se ejecuta en un unico ciclo, sin interaccion
adicional con el usuario una vez iniciada la tarea.

El flujo completo es el siguiente:

1. Validacion previa de archivos de entrada (Fase 0).
2. Conversion del documento de requisitos (.docx) a Markdown.
3. Analisis cruzado del Markdown generado junto con el archivo de
   transacciones (.xml) y, opcionalmente, una plantilla (.md).
4. Generacion del archivo Excel final a partir del analisis estructurado.
5. Entrega del resultado al usuario junto con una solicitud de encuesta de
   satisfaccion.

## Modelo

Este agente utiliza el modelo **GPT o4-mini** como motor de razonamiento
para coordinar el pipeline, validar los archivos de entrada, invocar a
los subagentes en el orden correcto y gestionar las politicas de
reintentos y entrega final.

## Arquitectura del pipeline

El agente coordina los siguientes subagentes, cada uno responsable de una
fase especifica:

| Orden | Subagente                                  | Entrada                                                        | Salida                      |
|-------|---------------------------------------------|-----------------------------------------------------------------|------------------------------|
| 1     | `Agente-BibaC-FCT-Docx-To_Markdown_V2`       | Archivo `.docx` de requisitos                                   | Documento Markdown           |
| 2     | `Agente-BibaC-FCT-Analisis-Md-Xml_V2`        | Markdown del paso 1, archivo `.xml`, archivo `.md` (o fallback) | `analisis-data.md`           |
| 3     | `Agente-BibaC-FCT-Markdown-Excel_V2`         | `analisis-data.md` del paso 2                                   | Archivo `.xlsx` descargable  |

Cada subagente gestiona sus propias herramientas de forma autonoma
(Agent Workspace) y no recibe instrucciones adicionales del orquestador
mas alla de los archivos definidos para su fase.

## Archivos de entrada

| Archivo       | Tipo        | Obligatorio | Descripcion                              |
|---------------|-------------|-------------|-------------------------------------------|
| Transacciones | `.xml`      | Si          | Datos de transacciones de base de datos   |
| Requisitos    | `.docx`     | Si          | Requisitos del proceso                    |
| Plantilla     | `.md` / `.xlsx` | No      | Plantilla para el output final            |

Si falta el archivo `.docx` o el archivo `.xml`, el pipeline se detiene
antes de invocar cualquier subagente y se notifica al usuario el archivo
faltante.

Si falta el archivo `.md` de plantilla, el pipeline continua utilizando
internamente un valor `fallback_predeterminado`, sin notificar al usuario.

## Variables de sistema

| Variable            | Descripcion                                                                                           |
|----------------------|---------------------------------------------------------------------------------------------------------|
| `link_encuesta`      | URL de la encuesta de satisfaccion. Se inyecta en el system prompt antes de la ejecucion y se usa de forma literal, sin modificaciones. |

## Fases del pipeline

### Fase 0 — Validacion previa

Comprueba que existan los archivos `.docx` y `.xml` obligatorios antes de
iniciar cualquier procesamiento. Si falta alguno de ellos, el pipeline se
aborta de inmediato.

### Fase 1 — Conversion DOCX a Markdown

Se invoca a `Agente-BibaC-FCT-Docx-To_Markdown_V2` adjuntando unicamente el
archivo `.docx`. El orquestador nunca realiza esta conversion por su
cuenta, aunque disponga tecnicamente de herramientas capaces de hacerlo.

Politica de reintentos: hasta 3 intentos automaticos si el subagente falla
o devuelve un output vacio o malformado. Si los 3 intentos fallan, el
pipeline se aborta y se notifica el error.

### Fase 2 — Analisis de Markdown y XML

Se invoca a `Agente-BibaC-FCT-Analisis-Md-Xml_V2` adjuntando el Markdown
generado en la Fase 1, el archivo `.xml` original y el archivo `.md` de
plantilla (o el valor `fallback_predeterminado` si no existe). El
orquestador nunca realiza este analisis por su cuenta.

Politica de reintentos: hasta 3 intentos automaticos. Si los 3 intentos
fallan, el pipeline se aborta y se notifica el error.

### Fase 3 — Generacion del Excel

Se invoca a `Agente-BibaC-FCT-Markdown-Excel_V2` adjuntando unicamente el
archivo `analisis-data.md` producido en la Fase 2. Se valida que el
resultado incluya un enlace de descarga real del archivo `.xlsx`, no solo
un nombre de archivo en texto.

Politica de reintentos: hasta 3 intentos automaticos, incluyendo el caso
en que el subagente devuelva unicamente un nombre de archivo sin enlace de
descarga. Si los 3 intentos fallan, el pipeline se aborta y se notifica el
error.

### Fase 4 — Entrega final

La respuesta final al usuario se compone exclusivamente de dos partes, en
este orden:

1. **Passthrough obligatorio**: el output integro de
   `Agente-BibaC-FCT-Markdown-Excel_V2`, sin modificaciones, resumenes ni
   texto adicional.
2. **Parrafo de encuesta y cierre**: una invitacion a completar la encuesta
   de satisfaccion usando el enlace exacto de `link_encuesta`, seguida de
   la linea de cierre fija:
   `Gracias por usar el agente de conversion y trazabilidad ETL!`

El orquestador no redacta ningun otro texto propio en esta fase.

## Gestion de errores

| Situacion                                          | Accion                                                               |
|-----------------------------------------------------|-------------------------------------------------------------------------|
| Falta `.docx` o `.xml`                              | Abortar antes del pipeline. Notificar al usuario que archivo falta     |
| Subagente falla en un intento                       | Reintentar automaticamente sin notificar al usuario                    |
| Subagente falla los 3 intentos                      | Abortar pipeline. Notificar paso fallido y error recibido              |
| Falta archivo `.md`                                 | Usar `fallback_predeterminado`. Continuar sin notificar al usuario     |
| Output de subagente vacio o malformado              | Tratar como fallo y aplicar politica de reintentos                     |
| Excel entregado sin enlace de descarga              | Tratar como fallo. Aplicar reintentos de la Fase 3                     |
| Nombre de archivo escrito en texto sin adjunto      | Prohibido. Tratar como fallo y aplicar reintentos de la Fase 3         |

## Restricciones de diseño

Este agente esta construido bajo un conjunto estricto de reglas que
garantizan que el procesamiento real de los archivos ocurra siempre en los
subagentes especializados, y nunca en el orquestador:

- No se realizan preguntas al usuario una vez iniciada la tarea.
- No se omite ni se reordena ningun paso del pipeline.
- No se adjuntan archivos innecesarios a ningun subagente.
- No se convierte el `.docx` a Markdown por cuenta propia.
- No se analiza el `.xml` ni el Markdown por cuenta propia.
- No se genera el archivo Excel por cuenta propia.
- No se usan herramientas propias de lectura, escritura o ejecucion de
  codigo para resolver ninguna fase del pipeline; la unica accion
  permitida en las Fases 1 a 3 es invocar al subagente correspondiente.
- No se reportan avances intermedios al usuario; solo se comunica el
  resultado final o un error critico.
- No se redacta texto propio de exito; la entrega final depende
  exclusivamente del passthrough del output de
  `Agente-BibaC-FCT-Markdown-Excel_V2`.

## Resultado esperado

Al finalizar correctamente el pipeline, el usuario recibe:

- El archivo `.xlsx` final con enlace de descarga, generado por
  `Agente-BibaC-FCT-Markdown-Excel_V2`.
- Una invitacion para completar la encuesta de satisfaccion del agente.

## Notas de uso

- Este README describe el comportamiento funcional del agente tal como
  esta definido en su configuracion. No constituye documentacion de una
  API ni de un servicio ejecutable de forma independiente.
- Los subagentes referenciados (`Agente-BibaC-FCT-Docx-To_Markdown_V2`,
  `Agente-BibaC-FCT-Analisis-Md-Xml_V2`,
  `Agente-BibaC-FCT-Markdown-Excel_V2`) deben estar disponibles en el
  mismo entorno de ejecucion para que el pipeline funcione correctamente.