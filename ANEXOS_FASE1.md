# Anexos de la Propuesta de Datos · Proyecto Centinela Homicidios Medellín

**Estado:** borrador de auditoría; las casillas pendientes representan trabajo **no realizado**, no evidencias aprobadas.

## Anexo A · Prueba de acoplamiento de dos modalidades (OBLIGATORIA Y PENDIENTE)

**Clave:** `(codigo_comuna, año-mes)` con ventana de datos anteriores al mes objetivo. Para cada fila se requiere un **conteo oficial agregado**, un **archivo o recorte mensual satelital real** y su identificación verificable (producto, fecha, ubicación o identificador de descarga). El conjunto original cargado en esta conversación **no contiene imágenes**. No se han descargado archivos VNP46A3 ni se han generado diez pares reales.

| N.º | Código comuna y mes de observación | Homicidios agregados (fuente oficial) | Archivo de imagen NASA (identificador real) | Estado |
| --- | --- | --- | --- | --- |
| 1 | Pendiente | Pendiente | Pendiente | No verificado |
| 2 | Pendiente | Pendiente | Pendiente | No verificado |
| 3 | Pendiente | Pendiente | Pendiente | No verificado |
| 4 | Pendiente | Pendiente | Pendiente | No verificado |
| 5 | Pendiente | Pendiente | Pendiente | No verificado |
| 6 | Pendiente | Pendiente | Pendiente | No verificado |
| 7 | Pendiente | Pendiente | Pendiente | No verificado |
| 8 | Pendiente | Pendiente | Pendiente | No verificado |
| 9 | Pendiente | Pendiente | Pendiente | No verificado |
| 10 | Pendiente | Pendiente | Pendiente | No verificado |

**Cómo completar:**

1. Consultar el producto mensual [VNP46A3](https://ladsweb.modaps.eosdis.nasa.gov/missions-and-measurements/products/VNP46A3) con el acceso requerido por el servicio de NASA; registrar identificador, URL/DOI, fecha, calidad y términos.
2. Descargar polígonos del [servicio oficial de Medellín](https://www.medellin.gov.co/servidormapas/rest/services/mapas_nacionales/VC_Limite_Politico_Admtivo/MapServer/1), usar código y nombre como correspondencia y verificar coordenadas.
3. Generar diez recortes raster en 10 combinaciones reales de `comuna–mes` y acoplarlas con el conteo agregado del mismo periodo. Guardar código y comprobaciones; sin duplicar la misma imagen genérica para múltiples filas.
4. Registrar fecha de observación y fecha en que los datos satelitales estaban efectivamente disponibles, para que el predictor futuro no incorpore información publicada con retraso.

**Resultado de compuerta:** no acreditado aún. Se requiere evidencia real para que el docente pueda aprobar esta sección.

## Anexo B · Test de procedencia

### B1. Cadena de procedencia

| Archivo/fuente | Productor primario declarado | Fecha de descarga | SHA-256 / condición |
| --- | --- | --- | --- |
| `Homicidio 141225.xlsx` | **Pendiente de confirmar** correspondencia con [Alcaldía de Medellín / SISC](https://www.datos.gov.co/dataset/Homicidio/2fxs-6yhd) | **No informada** | `3d58e3753ee33f13169f4ab1b01eb9cd39ce4f05471563dec447112423ddbf6f` |
| NASA VNP46A3.002 | [NASA LAADS DAAC](https://ladsweb.modaps.eosdis.nasa.gov/missions-and-measurements/products/VNP46A3) | Pendiente | No descargado; huella por calcular |
| Límites de comunas | [Alcaldía de Medellín](https://www.medellin.gov.co/servidormapas/rest/services/mapas_nacionales/VC_Limite_Politico_Admtivo/MapServer/1) | Pendiente | No descargado; huella por calcular |

### B2. Descarga trazable

**Pendiente:** documentar URL/API exacta usada para obtener el Excel, fecha, filtros y si la fuente oficial coincide con el archivo recibido. En el futuro notebook, automatizar la obtención o documentar la copia local cuya huella SHA-256 coincida. **No asumir autenticidad solo porque el nombre es parecido al conjunto oficial.**

### B3. Búsqueda de espejos

El 2026-10-31 se hicieron búsquedas web exploratorias para localizar un dataset de Medellín con los campos `barrio_hecho`, `codigo_barrio_hecho` y `grupo_caracterizacion`, incluida la plataforma Kaggle; **no se confirmó una réplica idéntica**. Esto no equivale a haber agotado el test. **Pendiente:** registrar búsquedas y capturas/enlaces directamente en kaggle.com/datasets con nombre de archivo y conjunto, y comparar coincidencias, si existen.

### B4. Huella de crudeza antes de limpiar

- **Dimensión:** 5.711 filas × 17 columnas.
- **Faltantes:** `identidad_genero` 4.558 (79,8 %), `ocupacion` 1.074 (18,8 %), `pais` 408 (7,1 %), `comuna_hecho` 4 (0,07 %).
- **Codificación:** 53 valores de edad `-1` (faltantes semánticos, no edad válida).
- **Cobertura:** enero de 2014–14 de diciembre de 2025; diciembre de 2025 parcial (13 registros).
- **Posibles duplicados exactos:** 4; no eliminar sin comprobar si son hechos/víctimas distintos.
- **Objetivo binario derivado:** 2.751 pares territorio–mes de enero de 2015 a noviembre de 2025; clase de alerta positiva: entrenamiento 382/2.016, validación 48/252, prueba 87/483. Estas son **frecuencias de etiqueta calculadas con una regla propuesta**, no resultados del MLP.
- **Ajuste territorial:** `comuna_hecho` faltante en un registro de febrero de 2025 con `codigo_comuna_hecho = 60`; se asigna a San Cristóbal. Los otros 3 faltantes de diciembre de 2025 quedan fuera por mes parcial.

### B5. Contraste contra posibles réplicas

**Pendiente:** si surge un conjunto espejo, comparar origen, hash, número de filas, columnas y rango temporal. La referencia a una fuente primaria **no certifica** que el Excel sea idéntico a su versión actual.

## Anexo C · Protección de datos (Ley 1581 de 2012)

### C1. Registros de homicidios — trabajar únicamente con agregados

| Control | Evaluación / condición |
| --- | --- |
| Identificadores directos | En las 17 columnas revisadas no se observaron nombres ni números de identificación personal. |
| Identificadores indirectos | **Sí existen:** fecha y hora, barrio, edad, sexo, ocupación; la combinación puede identificar a una víctima. |
| Datos de menores | Hay edades menores de 18 años; no publicar microdatos ni perfiles individuales. |
| Finalidad declarada | Propuesta académica de alerta por comuna–mes; sin decisiones ni predicciones personales. |
| Minimización | Para el MLP conservar únicamente conteos, códigos territoriales y periodos agregados. |
| Enlaces geográficos | Únicamente geometrías de territorio; ninguna ubicación exacta de víctima en el repositorio. |
| Reidentificación y celdas pequeñas | Evaluar supresión o agrupación en tablas públicas con combinaciones demasiado detalladas. |
| Publicación | Repositorio sin Excel crudo, ejemplos de microdatos, datos personales ni imágenes sensibles. |
| Uso de servicios externos | No enviar microdatos a Colab público, asistentes de IA ni repositorios abiertos. |
| Licencia del conjunto primario | [Metadatos oficiales indican CC BY-SA 4.0](https://www.datos.gov.co/dataset/Homicidio/2fxs-6yhd); confirmar identidad de la descarga y obligaciones de atribución. |
| Conservación | Fijar lugar de almacenamiento local, responsables y fecha de supresión al cerrar el curso. |
| Estado del checklist | **Parcialmente diligenciado; requiere revisión y firma del integrante.** |

### C2. Imágenes NASA Black Marble

- [x] Fuente identificada: VIIRS/NPP VNP46A3.002, medición de radiancia, sin personas identificables a esta escala.
- [x] Política de datos NASA localizada: [Science Data Licenses](https://science.data.nasa.gov/about/license).
- [ ] Revisar metadatos y condiciones del archivo descargado en concreto.
- [ ] Verificar ausencia de detalles o metadatos que expongan información privada de terceros.
- [ ] Documentar almacenamiento, finalidad y supresión de las imágenes.

### C3. Polígonos del municipio

- [x] Servicio oficial identificado.
- [ ] Comprobar términos de acceso, redistribución y atribución antes de incluir archivos de geometrías en un repositorio público.
- [ ] Confirmar claves/códigos que concuerden con el Excel de homicidios.

### Declaración de integridad de la revisión

Se revisó la estructura del archivo aportado para redactar esta propuesta. **No se afirma** que los pares NASA–homicidios hayan sido descargados ni validados. Tampoco se afirma que el dataset de Excel sea una réplica exacta de la fuente primaria de Medellín sin contrastar los archivos.

| Integrante | Firma | Fecha |
| --- | --- | --- |
| Por completar | | 2026-10-31 |
