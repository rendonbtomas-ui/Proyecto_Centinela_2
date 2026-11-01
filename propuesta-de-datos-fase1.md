# Propuesta de Datos · Proyecto Centinela · Fase 1

**Título:** Centinela Homicidios Medellín: alertas de repunte mensual con MLP.  
**Integrantes:** por completar · **Fecha:** 2026-10-31 · **Versión:** v1 (borrador).

## 1. Problema, cliente y objetivo binario

**Cliente:** Secretaría de Seguridad y Convivencia de Medellín (análisis territorial). **Finalidad:** anticipar meses con incrementos inusuales de homicidios registrados para apoyar decisiones preventivas verificadas por personas. **Unidad:** comuna/corregimiento–mes. **Alerta = 1** si en el mes objetivo se registran **dos o más homicidios** y el conteo supera el **P75 de los doce meses anteriores** del mismo territorio; `0` en otro caso. Se pronostica el mes siguiente usando solo información previa. Este umbral académico detecta repuntes relativos, pero pierde la magnitud del aumento; no es una regla oficial.

## 2. Dos modalidades independientes

**Temporal:** registros administrativos mensuales de homicidios por territorio. **Imagen:** radiancia nocturna raster mensual de **NASA Black Marble VNP46A3**, captada por un satélite. La distribución de luz aporta información espacial distinta del conteo administrativo, sin demostrar causalidad. **Fase 1:** serie agregada y aplanada para MLP. **Fase 2:** GRU/LSTM para la serie y CNN para las imágenes, con fusión posterior.

## 3. Fuentes, enlaces y licencia

| Modalidad | Fuente | Licencia y datos personales |
| --- | --- | --- |
| Temporal | [Alcaldía de Medellín/SISC · *Homicidio*](https://www.datos.gov.co/dataset/Homicidio/2fxs-6yhd) | **CC BY-SA 4.0** según ficha; el Excel contiene microdatos potencialmente reidentificables. Solo se modelarán agregados. |
| Imagen | [NASA · VNP46A3.002](https://doi.org/10.5067/VIIRS/VNP46A3.002) | Datos de misiones NASA generalmente **CC0** salvo restricciones particulares ([política](https://science.data.nasa.gov/about/license)); sin datos de personas. |
| Geometría auxiliar | [Alcaldía de Medellín · comunas y corregimientos](https://www.medellin.gov.co/servidormapas/rest/services/mapas_nacionales/VC_Limite_Politico_Admtivo/MapServer/1) | Términos de redistribución por verificar; sin datos personales. |

## 4. Crudeza y procedencia

`Homicidio 141225.xlsx`: **5.711 filas × 17 columnas**, enero de 2014–14 de diciembre de 2025; 21 territorios, 4 nombres de comuna vacíos, 53 edades `-1` y 79,8 % de faltantes en `identidad_genero`. No incluye etiqueta ni particiones: requiere agregación y control temporal. **Pendiente:** comprobar que coincide con la descarga primaria, completar búsquedas de espejos Kaggle y documentar procedencia y huellas (anexo B).

## 5. Fase 1, partición y alineación

Características: conteos rezagados 1–3 meses, medias/sumas móviles 3/6/12, calendario y territorio. División por **mes objetivo**: entrenamiento 2015–2022; validación 2023; prueba enero de 2024–noviembre de 2025. Diciembre de 2025 es parcial y se excluye. Hay **2.751 pares territorio–mes** etiquetables según la regla propuesta. Escalado solo con entrenamiento; umbral de decisión solo con validación. **Clave agregada entre fuentes:** `(codigo_comuna, año-mes)`, con apoyo de geometrías oficiales; imágenes y conteos usados para pronosticar el mes `m` deben estar disponibles hasta `m−1`.

**Compuerta pendiente:** obtener **10 pares reales** serie–imagen y verificar el retraso de publicación de NASA. La unión no ha sido realizada (anexo A); esta propuesta **aún no está lista para aprobación**.

## 6. Línea base y red sencilla

Comparar contra la **clase mayoritaria** (exactitud y recall) y una regla simple basada en el conteo anterior frente a su histórico (umbral ajustado en validación). MLP: **16 y 8 neuronas ReLU**, Dropout, salida logit, `BCEWithLogitsLoss`, Adam y parada temprana. Evaluar matriz de confusión, precisión, recall, F1 y PR-AUC. **Aún no hay métricas del modelo.**

## 7. Riesgos y mitigación

**Subregistro/revisiones:** auditar cobertura; cero registrado no garantiza cero real. **Falsas alarmas:** evitar estigma territorial y validar las decisiones con expertos. **Alertas omitidas:** medir recall y falsos negativos por territorio. **Privacidad:** fecha, barrio y características personales pueden reidentificar víctimas; no publicar Excel crudo, ejemplos individuales ni predicciones personales. Checklist de Ley 1581 en anexo C.

## 8. Referencias APA 7

- Alcaldía de Medellín, Secretaría de Seguridad y Convivencia. (s. f.). *Homicidio* [Conjunto de datos]. https://www.datos.gov.co/dataset/Homicidio/2fxs-6yhd
- NASA VIIRS Land Science Investigator-led Processing System. (2025). *VIIRS/NPP Lunar BRDF-Adjusted Nighttime Lights Monthly L3 Global 15 arc second Linear Lat Lon Grid* (VNP46A3.002) [Datos]. https://doi.org/10.5067/VIIRS/VNP46A3.002
- Alcaldía de Medellín. (s. f.). *Comunas y corregimientos* [Capa cartográfica]. https://www.medellin.gov.co/servidormapas/rest/services/mapas_nacionales/VC_Limite_Politico_Admtivo/MapServer/1

**Anexos A, B y C:** [`ANEXOS_FASE1.md`](./ANEXOS_FASE1.md).
