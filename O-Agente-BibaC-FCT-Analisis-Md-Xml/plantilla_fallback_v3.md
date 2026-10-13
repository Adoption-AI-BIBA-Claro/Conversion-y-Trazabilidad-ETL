# Análisis Técnico-Funcional [Extraer del .md: CÓDIGO_PROYECTO]

## Notas sobre Filtrado de Datos

### Reglas de Inclusión por Sección

#### Sección 1.1: FUENTES EXTERNAS-INTERNAS (DEV)
- ✅ **Incluir:** TODOS los registros
- ✅ **Incluir:** Dependencias (INTERNA DEPENDENCIA)
- ✅ **Incluir:** Archivos planos (ARCHIVO_PLANO)
- ✅ **Incluir:** Items internos
- 📝 **Nota:** Esta sección documenta el panorama COMPLETO de fuentes, incluyendo todas las dependencias

#### Sección 1.2: ANALISIS FUNCIONAL (DEV-PO)
- ✅ **Incluir:** Campos externos (EXTERNA)
- ❌ **Excluir:** Campos de destino (DESTINO) — solo aplican en Sección 1.1
- ❌ **Excluir:** Dependencias (INTERNA DEPENDENCIA)
- ❌ **Excluir:** Archivos planos (ARCHIVO_PLANO) - "No aplica para homologación"
- ❌ **Excluir:** Items internos (INTERNA)
- 📝 **Nota:** Esta sección contiene SOLO el análisis funcional de campos que aplican para homologación con sistemas externos

---

## 1. Hojas Principales del Proyecto

### 1.1 FUENTES EXTERNAS-INTERNAS (DEV)

#### Descripción
Mapeo completo de fuentes externas e internas con sus flujos de datos entre bases de datos. Incluye todas las dependencias, archivos planos y transformaciones intermedias para documentar el panorama ETL completo.

#### Estructura
| Columna | Descripción |
|---------|------------|
| JOB (JP) | Identificador del Job en ODI |
| NODO (JH) | Identificador del Nodo en ODI |
| NOMBRE NODO | Nombre descriptivo del nodo |
| TIPO_TABLA | Tipo de tabla: EXTERNA, INTERNA, INTERNA DEPENDENCIA, ARCHIVO_PLANO, DESTINO |
| BDs-ESQUEMA FUENTE | Base de datos y esquema origen |
| NOMBRE FUENTE | Nombre de la tabla/archivo fuente |
| DESTINO | TIPO_TABLA del objeto destino — mismas categorías que TIPO_TABLA pero aplicadas al destino: INTERNA (tabla staging/temp), DESTINO (tabla final DWH), EXTERNA (sistema externo), EXTRACCION YB. Nunca escribir "DESTINO" como valor para todas las filas. |
| BDs-ESQUEMA DESTINO | Base de datos y esquema destino |
| NOMBRE DESTINO | Nombre de la tabla destino |
| OBSERVACIONES | Notas y observaciones adicionales |

#### Datos

| JOB | NODO | NOMBRE NODO | TIPO_TABLA | BDs-ESQUEMA FUENTE | NOMBRE FUENTE | DESTINO | BDs-ESQUEMA DESTINO | NOMBRE DESTINO | OBSERVACIONES |
|-----|------|-------------|-----------|-------------------|--------------|---------|-------------------|---------------|--------------|
| [Extraer del XML] | [Extraer del XML] | [Extraer del .md] | [Inferir] | [Extraer del XML] | [Extraer del .md] | [Inferir — TIPO_TABLA del destino] | [XML o .md] | [Extraer del .md] | [Extraer del .md — filtros, condiciones JOIN, reglas funcionales del nodo. Nunca vacío.] |

#### Filtrado: NINGUNO - Incluir todos los registros

---

### 1.2 ANALISIS FUNCIONAL (DEV-PO)

#### Descripción
Análisis funcional detallado de campos, tablas y objetos externos mapeados al sistema. Enfocado SOLO en campos que aplican para homologación con sistemas externos.

#### Estructura
| Columna                     | Descripción                                                  |
| --------------------------- | ------------------------------------------------------------ |
| MÓDULO                      | Módulo de negocio                                            |
| HERRAMIENTA                 | Herramienta utilizada                                        |
| CÓDIGO DEL SERVICIO         | Código identificador del servicio                            |
| MOTOR_BD                    | Motor de base de datos                                       |
| TIPO_FUENTE                 | Tipo de fuente (EXTERNA, DESTINO únicamente en esta sección) |
| DB_FUENTE                   | Base de datos origen                                         |
| ESQUEMA_TABLA               | Esquema de la tabla                                          |
| TABLA                       | Nombre de la tabla                                           |
| CAMPO_TABLA                 | Nombre del campo                                             |
| DESCRIPCIÓN CAMPO - Usuario | Descripción desde perspectiva del usuario                    |
| DESCRIPCIÓN CAMPO           | Descripción técnica del campo                                |
| APUNTAMIENTO                | Referencia cruzada                                           |
| OBJETO EXTERNO              | Objeto correspondiente en sistema externo                    |
| DOMINIO SF/EXTERNO          | Dominio Salesforce o sistema externo                         |
| API_NAME - YB               | Nombre de API                                                |
| LABEL - YB                  | Etiqueta de presentación                                     |
| BD_DESTINO                  | Base de datos destino                                        |
| TABLA_DESTINO               | Tabla destino                                                |
| OBSERVACIONES               | Notas especiales                                             |
| RELEASE A TRABAJAR          | Release de implementación                                    |

#### Datos

| MÓDULO | HERRAMIENTA | CÓDIGO DEL SERVICIO | MOTOR_BD | TIPO_FUENTE | DB_FUENTE | ESQUEMA_TABLA | TABLA | CAMPO_TABLA | DESCRIPCIÓN CAMPO - Usuario | DESCRIPCIÓN CAMPO | APUNTAMIENTO | OBJETO EXTERNO | DOMINIO SF/EXTERNO | API_NAME - YB | LABEL - YB | BD_DESTINO | TABLA_DESTINO | OBSERVACIONES | RELEASE A TRABAJAR |
|--------|-------------|-------------------|---------|------------|---------|--------------|-------|------------|---------------------------|----------------|------------|----------------|-------------------|-----------|---------|----------|--------------|------------|-----------------|
| [Extraer del .md] | [Extraer del .md] | [Extraer del .md] | [Extraer del .md] | [Inferir] | [Extraer del .md] | [Extraer del XML] | [Extraer del XML] | [Extraer del .md] | [Extraer del .md] | [Extraer del .md] | [Extraer del .md] | [Extraer del .md] | [Extraer del .md] | [Extraer del .md] | [Extraer del .md] | [Extraer del XML] | [Extraer del XML] | [Extraer del .md] | [Extraer del .md] |

#### Filtrado: APLICAR FILTROS
```
Excluir si:
  - TIPO_FUENTE = "INTERNA DEPENDENCIA"
  - TIPO_FUENTE = "ARCHIVO_PLANO"
  - TIPO_FUENTE = "INTERNA"
  - TIPO_FUENTE = "DESTINO"
  - DESCRIPCIÓN contiene "Dependencia"
  - OBSERVACIONES contiene "No aplica para homologación"
```

---

### 1.3 DEPENDENCIAS

#### Descripción
Listado de dependencias y sus tablas asociadas. Detalla las relaciones de precedencia entre procesos y fuentes de datos.

#### Estructura
| Columna | Descripción |
|---------|------------|
| DEPENDENCIAS | Identificador de dependencia |
| TABLA | Nombre de la tabla dependiente |

#### Datos

| DEPENDENCIAS | TABLA |
|-------------|-------|
| [del .md y XML — proceso o servicio externo prerequisito] | [del .md y XML — tabla que ese proceso produce] |

#### Filtrado: NINGUNO - Incluir todas las dependencias

---

### 1.4 LISTAS (Catálogos)

#### Valores Permitidos para TIPO_CLASIFICACION
- **EXTRACCION YB:** Tablas de la capa de extracción interna de Yellowbrick (`YB_DB_DWH_EXTRACCION_*`); pasos YB→YB previos al staging
- **EXTERNA:** Datos provenientes de sistemas externos (SAP, Datarep, Salesforce, Oracle externo, SQL Server, etc.)
- **INTERNA:** Campos calculados o auxiliares dentro del DWH que no son staging ni destino final
- **INTERNA DEPENDENCIA:** Tablas temporales o de staging intermedias (`TMP_*`, tablas que alimentan otras tablas del DWH)
- **ARCHIVO_PLANO:** Datos provenientes de archivos CSV, TXT, XML (ANS), conexiones SFTP (`CNX_FILE_*`)
- **DESTINO:** Tablas finales en el Data Warehouse (no staging, no temporales)

---

## 2. Elementos Clave del Sistema

### 2.1 Tipos de Fuentes
- EXTRACCION YB
- EXTERNA
- INTERNA
- INTERNA DEPENDENCIA
- ARCHIVO_PLANO
- DESTINO

### 2.2 Bases de Datos Principales

#### Bases de Datos Fuentes
- [Extraer del XML]
- [Extraer del XML]
- [Extraer del XML]

#### Bases de Datos Destino
- [Extraer del XML]
- [Extraer del XML]

### 2.3 Esquemas Principales
- [Extraer del XML]
- [Extraer del XML]
- [Extraer del XML]

### 2.4 Objetos Externos (Ericsson u otros)
- [Extraer del XML]
- [Extraer del XML]
- [Extraer del XML]

---

## 3. Tablas Principales

### 3.1 Tablas Externas
- [Extraer del XML]
- [Extraer del XML]
- [Extraer del XML]

### 3.2 Tablas Internas (Staging)
- [Extraer del XML]
- [Extraer del XML]
- [Extraer del XML]

### 3.3 Tablas Destino (DWH)
- [Extraer del XML]
- [Extraer del XML]
- [Extraer del XML]

---

## 4. Jobs (JP) Principales
- [Extraer del XML]
- [Extraer del XML]
- [Extraer del XML]

---

## 5. Nodos (JH) Principales
- [Extraer del XML]
- [Extraer del XML]
- [Extraer del XML]

---

## 6. Rutas ANS
- [Extraer del XML]

---

## Tabla de Datos - FUENTES EXTERNAS-INTERNAS (DEV)

### Formato de Llenado

```markdown
| JOB | NODO | NOMBRE NODO | TIPO_TABLA | BDs-ESQUEMA FUENTE | NOMBRE FUENTE | DESTINO | BDs-ESQUEMA DESTINO | NOMBRE DESTINO | OBSERVACIONES |
|-----|------|-------------|-----------|-------------------|--------------|---------|-------------------|---------------|--------------|
| [Extraer del XML] | [Extraer del XML] | [Extraer del .md] | [Inferir] | [Extraer del XML] | [Extraer del .md] | [Inferir — TIPO_TABLA del destino] | [XML o .md] | [Extraer del .md] | [Extraer del .md — filtros, condiciones JOIN, reglas funcionales del nodo. Nunca vacío.] |
```

**Nota:** Esta sección incluye TODOS los registros sin filtros aplicados.

---

## Tabla de Datos - ANALISIS FUNCIONAL (DEV-PO)

### Formato de Llenado

```markdown
| MÓDULO | HERRAMIENTA | CÓDIGO DEL SERVICIO | MOTOR_BD | TIPO_FUENTE | DB_FUENTE | ESQUEMA_TABLA | TABLA | CAMPO_TABLA | DESCRIPCIÓN CAMPO - Usuario | DESCRIPCIÓN CAMPO | APUNTAMIENTO | OBJETO EXTERNO | DOMINIO SF/EXTERNO | API_NAME - YB | LABEL - YB | BD_DESTINO | TABLA_DESTINO | OBSERVACIONES | RELEASE A TRABAJAR |
|--------|-------------|-------------------|---------|------------|---------|--------------|-------|------------|---------------------------|----------------|------------|----------------|-------------------|-----------|---------|----------|--------------|------------|-----------------|
| [Extraer del .md] | [Extraer del .md] | [Extraer del .md] | [Extraer del .md] | [Inferir] | [Extraer del .md] | [Extraer del XML] | [Extraer del XML] | [Extraer del .md] | [Extraer del .md] | [Extraer del .md] | [Extraer del .md] | [Extraer del .md] | [Extraer del .md] | [Extraer del .md] | [Extraer del .md] | [Extraer del XML] | [Extraer del XML] | [Extraer del .md] | [Extraer del .md] |
```

### Filtros Aplicados
⚠️ **IMPORTANTE:** Al completar esta tabla, aplicar los siguientes filtros:

1. ❌ **Excluir** cualquier fila donde `TIPO_FUENTE` = "INTERNA", "INTERNA DEPENDENCIA" o "DESTINO"
2. ❌ **Excluir** cualquier fila donde `TIPO_FUENTE` = "ARCHIVO_PLANO"
3. ❌ **Excluir** cualquier fila con referencias a "No aplica para homologación"
4. ✅ **Incluir SOLO** filas con `TIPO_FUENTE` = "EXTERNA"

---

## Tabla de Datos - DEPENDENCIAS

### Formato de Llenado

```markdown
| DEPENDENCIAS | TABLA |
|-------------|-------|
| [del .md y XML — proceso o servicio externo prerequisito] | [del .md y XML — tabla que ese proceso produce] |
```

**Nota:** Incluir TODAS las dependencias - sin filtros.

---

## Información del Documento

| Campo                    | Valor               |
| ------------------------ | ------------------- |
| **Código de Referencia** | [Extraer del .md] |
| **Versión**              | [Extraer del .md] |
| **Fecha de Elaboración** | [Extraer del .md] |
| **Última Actualización** | [Extraer del .md] |
| **Estado**               | [Extraer del .md] |

---

## Notas y Observaciones

**Documento generado:** [Extraer del .md]
**Última actualización:** [Extraer del .md]
**Próxima revisión:** [Extraer del .md]
**Responsable de mantenimiento:** [Agente IA especializado en trazabilidad]
