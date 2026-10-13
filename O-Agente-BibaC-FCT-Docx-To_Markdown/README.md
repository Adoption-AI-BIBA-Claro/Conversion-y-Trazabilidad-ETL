# Agente de Preprocesamiento DOCX a Markdown

Asistente de preprocesamiento de documentos que opera dentro de un
pipeline multiagente. Su función es convertir archivos `.docx` a formato
Markdown, depurar su contenido y gestionar todos los archivos resultantes
en el Agent Workspace, manteniendo intacto cualquier archivo que no sea
`.docx`.

## Descripción general

Este agente forma parte de un flujo de procesamiento documental más
amplio. No analiza ni interpreta el contenido de negocio del documento:
su única responsabilidad es transformar el `.docx` recibido en un
Markdown fiel al original, eliminar información sensible específica
(direcciones IP) y dejar el resultado disponible en el workspace para que
el siguiente agente del pipeline pueda consumirlo.

Los archivos que no sean `.docx` se devuelven sin ninguna modificación,
en modo passthrough, byte por byte.

## Modelo

Este agente utiliza el modelo **Gemini Flash 3.5** como motor de
razonamiento para orquestar sus herramientas y generar las respuestas de
confirmación del proceso.

## Objetivo

1. Convertir archivos `.docx` a formato Markdown utilizando la
   herramienta de procesamiento DOCX correspondiente.
2. Escribir todos los archivos de salida, tanto los convertidos como los
   que no, en el Agent Workspace.
3. Devolver los archivos que no sean `.docx` sin modificaciones.

## Funciones principales

- Leer y procesar archivos `.docx` asegurando la fidelidad del contenido:
  idioma, estructura, encabezados, tablas, listas y elementos no
  textuales.
- Eliminar todas las direcciones IP (patrones IPv4) del texto Markdown
  generado, sin dejar marcadores ni reemplazos en su lugar.
- Pasar los archivos que no son `.docx` en modo passthrough, sin cambios,
  byte por byte.
- Escribir y verificar los archivos resultantes en el workspace para
  asegurar que no estén vacíos ni malformados.

## Herramientas utilizadas

| Herramienta                        | Uso                                                                                   |
|-------------------------------------|-----------------------------------------------------------------------------------------|
| Herramienta de Procesamiento DOCX   | Lectura y conversión exclusiva de archivos `.docx` a Markdown. Requiere siempre el argumento `text`. |
| Herramientas de Agent Workspace     | Escritura, modificación y verificación de los archivos de salida en el workspace persistente del agente. |

## Entradas

- Archivos `.docx` a convertir.
- Cualquier otro tipo de archivo, que se gestiona mediante passthrough
  sin procesamiento adicional.

## Salidas

- Archivo Markdown convertido, con las direcciones IP eliminadas,
  guardado en el workspace con el mismo nombre original y extensión
  `.md`.
- Archivos que no son `.docx` devueltos y escritos en el workspace sin
  modificar, byte por byte.
- Como respuesta en la conversación, únicamente la confirmación
  `"Done. The file is [filename]"` por cada archivo procesado.

## Comportamiento esperado

- El agente no añade comentarios, resúmenes ni explicaciones adicionales
  sobre el contenido del documento procesado.
- No conserva ni expone direcciones IP en ningún archivo Markdown
  generado.
- No altera el contenido de archivos que no sean `.docx`.
- Verifica siempre que el archivo escrito en el Agent Workspace no esté
  vacío ni malformado antes de considerar la tarea completada.

## Rol dentro del pipeline

Este agente actúa como el primer eslabón de un pipeline multiagente de
procesamiento documental. Su salida en Markdown está pensada para ser
consumida por agentes posteriores encargados del análisis y la
generación de reportes, por lo que la fidelidad del contenido convertido
es crítica para el correcto funcionamiento de las fases siguientes.

## Notas de uso

- Este README describe el comportamiento funcional del agente tal como
  está definido en su configuración. No constituye documentación de una
  API ni de un servicio ejecutable de forma independiente.
- El agente depende de que el Agent Workspace esté disponible y
  accesible en el entorno de ejecución para poder escribir y verificar
  los archivos de salida.
