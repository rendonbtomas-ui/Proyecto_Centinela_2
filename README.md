# Proyecto Centinela | Homicidios en Medellín

**Sistema experimental de alerta temprana de repuntes mensuales de homicidios mediante una red neuronal multicapa (MLP).**

**Asignatura:** Deep Learning — Proyecto Centinela  
**Institución:** Universidad Santo Tomás · Maestría en Ciencia de Datos  
**Estado:** propuesta técnica, Fase 1 (pendiente de aprobación del docente)  
**Fecha:** 31 de octubre de 2026  
**Integrantes:** Tomás Rendón Bustamante - Juan Sebastián Gamba 

> **Alcance:** el proyecto busca anticipar aumentos inusuales en el **número de homicidios registrados**, no determinar quién cometerá un delito, identificar posibles víctimas ni estimar el riesgo individual de una persona. Los resultados no justificarán actuaciones automáticas de vigilancia o control.

## 1. Resumen ejecutivo

| Elemento | Propuesta |
| --- | --- |
| **Problema** | Identificar con un mes de antelación posibles repuntes de homicidios en comunas y corregimientos de Medellín. |
| **Cliente potencial** | Secretaría de Seguridad y Convivencia de Medellín y equipos territoriales de análisis y prevención. |
| **Unidad de análisis** | Comuna/corregimiento × mes. |
| **Evento binario** | **Alerta (1):** al menos dos homicidios en el mes objetivo y un conteo superior al percentil 75 de los doce meses anteriores de ese territorio. **No alerta (0):** los demás casos. |
| **Fase 1** | Red neuronal MLP sencilla sobre características tabulares derivadas de series mensuales. |
| **Fase 2 (propuesta)** | Serie temporal con GRU/LSTM e imágenes satelitales reales de luces nocturnas (NASA Black Marble) mediante CNN y fusión. |
| **Fase 3 (propuesta)** | Prototipo de consulta y visualización de alertas, sujeto a evaluación humana. |
| **Herramientas** | Python, PyTorch, scikit-learn, pandas, Matplotlib y Jupyter/Google Colab. |
| **Semilla de reproducibilidad** | `42`. |

## 2. Pregunta de investigación y objetivos

**Pregunta:** ¿puede un perceptrón multicapa identificar, a partir del comportamiento histórico mensual, un repunte de homicidios registrados durante el mes siguiente en las comunas y corregimientos de Medellín?

**Objetivo general:** diseñar y evaluar un modelo de clasificación binaria basado en una red neuronal MLP para generar alertas experimentales de repunte mensual.

**Objetivos específicos:**
1. Auditar, limpiar y agregar los registros a la escala territorio–mes, sin incluir atributos de personas en las entradas del modelo.
2. Construir una etiqueta binaria de repunte, características temporales y una partición cronológica sin fuga de información.
3. Entrenar un MLP con dos capas ocultas y compararlo con referencias simples.
4. Evaluar aciertos, falsas alarmas, alertas omitidas y diferencias de desempeño por territorio.

## 3. Base de datos revisada

Se inspeccionó el archivo suministrado **`Homicidio 141225.xlsx`**, que contiene registros de homicidio asociados a territorios de Medellín. Los siguientes son **resultados descriptivos del archivo**, no resultados de entrenamiento:

| Indicador del archivo | Resultado observado |
| --- | --- |
| Filas de datos | **5.711** |
| Columnas originales | **17** |
| Periodo registrado | **enero de 2014–14 de diciembre de 2025** |
| Territorios identificados | **21** (16 comunas y 5 corregimientos) |
| Campo `identidad_genero` sin dato | **4.558** (79,8 %) |
| Campo `ocupacion` sin dato | **1.074** (18,8 %) |
| Edad codificada como `-1` | **53** registros; se trata como valor desconocido, no como una edad real. |
| `comuna_hecho` sin dato | **4** registros; uno puede recuperarse de `codigo_comuna_hecho = 60`, y tres corresponden al 14-12-2025. |
| Diciembre de 2025 | **13** registros hasta el día 14: **mes incompleto; se excluye** del análisis mensual. |
| Filas idénticas | **4** coincidencias completas; no deben eliminarse sin verificar si corresponden a víctimas distintas. |

**Campos relevantes para el modelo:** `codigo_comuna_hecho`, `comuna_hecho`, `anio_hecho`, `mes_hecho` y `fecha_hecho`. Las demás columnas permiten auditar la fuente, pero **no** se usarán para generar perfiles individuales. Los datos describen **hechos registrados**, no una medición exhaustiva de todos los hechos ocurridos.

**Fuente primaria candidata a verificar:** Alcaldía de Medellín, SISC, conjunto [*Homicidio* en Datos Abiertos Colombia](https://www.datos.gov.co/dataset/Homicidio/2fxs-6yhd), cuya ficha declara licencia **CC BY-SA 4.0**. El archivo recibido **aún no tiene trazabilidad documental que permita afirmar que es una descarga idéntica** a esa versión oficial; hay que contrastar encabezados, periodos, registros y fecha de descarga antes de declarar la procedencia definitiva.

**SHA-256 del Excel recibido:**

```text
3d58e3753ee33f13169f4ab1b01eb9cd39ce4f05471563dec447112423ddbf6f
```

## 4. Definición de la alerta y prevención de fuga de datos

Los registros se agruparán por **código de comuna/corregimiento y mes**. Cada territorio tendrá un calendario mensual completo; se imputará un conteo observado de cero solo en meses con cobertura confirmada de la fuente, dejando explícito que *cero registros* no equivale necesariamente a *cero homicidios reales*.

Sea `H(c,m)` el número de homicidios registrados en el territorio `c` durante el mes `m`. La etiqueta se definirá así:

```text
alerta(c,m) = 1  si H(c,m) >= 2 Y
                  H(c,m) > P75[H(c,m-12), ..., H(c,m-1)]
             0  en caso contrario
```

**Justificación:** un umbral relativo reconoce que las comunas tienen magnitudes diferentes de homicidios. La condición de al menos dos hechos evita que un único registro en territorios de baja frecuencia active automáticamente una alerta. Es una **regla académica inicial**, que podrá ajustarse con autorización del docente; no existe aquí un umbral operativo validado por la Alcaldía.

**Instante de predicción:** al cierre del mes `m-1` se intenta anticipar `alerta(c,m)`. Las variables de entrada solo utilizarán meses `m-1` o anteriores. Ni el conteo de `m` ni la imagen de `m` podrán entrar al modelo que predice `m`.

**Características candidatas:** conteos de 1, 2 y 3 meses atrás; suma/media de los 3, 6 y 12 meses anteriores; cambio entre los dos últimos meses; codificación cíclica del mes del año; codificación de territorio. Se verificará si incluir el territorio introduce dependencia excesiva de su identidad.

Con la definición anterior, la transformación exploratoria de este archivo arroja **2.751 observaciones territorio–mes** etiquetables (enero de 2015–noviembre de 2025). Distribución **preliminar** de la etiqueta, antes de entrenar:

| Partición por mes objetivo | Intervalo | Observaciones | Alertas (`1`) |
| --- | --- | ---: | ---: |
| Entrenamiento | ene. 2015–dic. 2022 | 2.016 | 382 (18,9 %) |
| Validación | ene.–dic. 2023 | 252 | 48 (19,0 %) |
| Prueba | ene. 2024–nov. 2025 | 483 | 87 (18,0 %) |

La serie histórica de 2014 se reserva para construir los primeros doce meses de antecedentes. Estas cifras **no son exactitud, recall ni resultados del MLP**.

## 5. Arquitectura sencilla para la Fase 1

```text
Entradas tabulares (n características)
                │
        Dense(16) + ReLU
                │
          Dropout(0.20)
                │
         Dense(8) + ReLU
                │
          Dense(1) [logit]
                │
     Sigmoide SOLO al evaluar
                │
 Probabilidad estimada de repunte
```

- **Framework:** PyTorch; modelo construido como clase propia (`torch.nn.Module`).
- **Pérdida:** `BCEWithLogitsLoss`, sin añadir `Sigmoid` dentro del modelo.
- **Optimización:** `Adam`, tasa inicial `1e-3`, minilotes, hasta 100 épocas, parada temprana por pérdida de validación.
- **Control de sobreajuste:** curvas de pérdida entrenamiento/validación; contraste del modelo sin y con Dropout; restaurar los mejores pesos.
- **Preprocesamiento:** el escalador y los parámetros de imputación se ajustarán **solo en entrenamiento**; validación y prueba únicamente reciben la transformación aprendida.
- **Selección del umbral de alarma:** solo con validación, priorizando un balance justificado entre alertas omitidas y falsas alarmas.

El MLP será deliberadamente pequeño para permitir explicar la arquitectura, el cálculo de la pérdida, la retropropagación y cada decisión técnica durante la sustentación.

## 6. Evaluación y referencias simples

El conjunto de prueba se utilizará **una sola vez** tras ajustar hiperparámetros y umbral con validación.

- **Referencia 1:** clase mayoritaria (no alerta); comparar exactitud y *recall* de la clase positiva.
- **Referencia 2:** regla simple basada exclusivamente en `H(c,m-1)` y el comportamiento histórico anterior, con umbral elegido en validación.
- **Métricas del MLP:** matriz de confusión, precisión, *recall*, F1, área bajo la curva precisión–recall (PR-AUC) y exactitud como complemento.
- **Auditoría:** analizar falsos positivos y falsos negativos, incluyendo sensibilidad por territorio cuando haya suficientes alertas reales para reportarla.

**No se reportan todavía métricas del modelo**, porque esta carpeta contiene la propuesta y no un entrenamiento ejecutado.

## 7. Continuidad del sistema multimodal

En la **Fase 2**, el proyecto conservará el mismo evento binario y la misma entidad de análisis, incorporando dos instrumentos independientes:

| Modalidad | Fuente real propuesta | Información y posible red |
| --- | --- | --- |
| **Serie temporal** | Registros oficiales de homicidio, agregados por comuna–mes. | Secuencia de conteos históricos → GRU o LSTM. |
| **Imagen** | [NASA Black Marble, producto mensual VNP46A3](https://ladsweb.modaps.eosdis.nasa.gov/missions-and-measurements/products/VNP46A3) (DOI: [10.5067/VIIRS/VNP46A3.002](https://doi.org/10.5067/VIIRS/VNP46A3.002)). | Recorte real de píxeles de radiancia nocturna → CNN. |
| **Apoyo geográfico** | [Límites de comunas y corregimientos — servicio geográfico de Medellín](https://www.medellin.gov.co/servidormapas/rest/services/mapas_nacionales/VC_Limite_Politico_Admtivo/MapServer/1). | Polígonos para delimitar los recortes, no una tercera modalidad predictiva. |

NASA indica que, salvo restricciones particulares, los datos de misiones lideradas por la agencia se distribuyen bajo condiciones **CC0**: [política oficial de licencias](https://science.data.nasa.gov/about/license). Es indispensable verificar los metadatos y términos del producto utilizado. Los archivos de satélite son datos raster reales, **no mapas de calor fabricados a partir de los homicidios**.

**Clave de alineación propuesta:** `(codigo_comuna, año-mes de observación)`. Para predecir el mes `m`, la secuencia de homicidios y la imagen más reciente deberán haberse observado a más tardar al cierre de `m-1`, considerando también el retraso real de publicación del satélite.

> **Pendiente para aprobar la Propuesta:** descargar una muestra real del producto NASA, verificar sus términos y aportar **10 uniones reales** entre recorte satelital y conteo mensual. La fuente y la clave están identificadas, pero la prueba de acoplamiento **aún no está realizada**. Ver [`ANEXOS_FASE1.md`](./ANEXOS_FASE1.md).

## 8. Riesgos y responsabilidad

| Riesgo | Medida preventiva |
| --- | --- |
| Subregistro, revisiones o cambios en la captura de datos | Documentar cobertura y versiones; no interpretar los ceros como ausencia definitiva del fenómeno. |
| Baja frecuencia de eventos en algunos territorios | Informar incertidumbre, tamaño de muestra y sensibilidad; no establecer rankings de peligrosidad de barrios/personas. |
| Falsa alerta | Verificación humana; evaluación del costo de reasignar recursos. |
| Alerta omitida | Medir *recall* y falsos negativos; no presentar el sistema como prevención garantizada. |
| Riesgo de reidentificación | No publicar registros individuales con fecha, barrio, edad y sexo; trabajar en el modelo con conteos mensuales agregados. |
| Confusión entre asociación y causalidad | No atribuir efectos causales a la radiancia nocturna ni a características territoriales por el solo desempeño predictivo. |

La **Ley 1581 de 2012** debe considerarse incluso si los registros fueron publicados en un portal oficial. En particular, combinaciones detalladas de fecha, barrio, edad y otros atributos pueden facilitar reidentificación. **No subir el Excel original a un repositorio público ni a notebooks compartidos** hasta verificar la base jurídica y la anonimización.

## 9. Estructura esperada del repositorio

```text
centinela-homicidios-medellin/
├── README.md                         # Presentación del proyecto
├── propuesta-de-datos-fase1.md        # Documento breve para aprobación
├── ANEXOS_FASE1.md                    # Procedencia, acoplamiento y privacidad
├── .gitignore
├── data/
│   ├── raw/                          # Archivos originales locales (no publicar)
│   └── processed/                    # Panel agregado sin datos individuales
├── notebooks/
│   └── 01_centinela_mlp.ipynb         # Se elaborará tras aprobación
├── src/                               # Funciones reproducibles por implementar
└── results/                           # Figuras y métricas, aún no generadas
```

**Estado de entregables:** esta versión entrega documentación inicial. El notebook ejecutable, el informe técnico-ético, la bitácora de IA y la prueba completa de acoplamiento son pasos posteriores de la asignatura.

## 10. Fuentes y material del curso

1. Alcaldía de Medellín, Secretaría de Seguridad y Convivencia / SISC. (s. f.). *Homicidio* [Conjunto de datos]. Datos Abiertos Colombia. https://www.datos.gov.co/dataset/Homicidio/2fxs-6yhd
2. NASA VIIRS Land Science Investigator-led Processing System. (2025). *VIIRS/NPP Lunar BRDF-Adjusted Nighttime Lights Monthly L3 Global 15 arc second Linear Lat Lon Grid* (VNP46A3, versión 002) [Conjunto de datos]. https://doi.org/10.5067/VIIRS/VNP46A3.002
3. Alcaldía de Medellín. (s. f.). *Comunas y corregimientos* [Servicio cartográfico]. https://www.medellin.gov.co/servidormapas/rest/services/mapas_nacionales/VC_Limite_Politico_Admtivo/MapServer/1
4. Universidad Santo Tomás. (s. f.). *Proyecto Centinela · Fase 1 — Línea base con MLP* [Guía de la asignatura]. https://jotamao1985.github.io/Deep_Learning_Usta/11_Modulo_1_Actividad_Fase1_Centinela.html
5. Universidad Santo Tomás. (s. f.). *Plantilla · Propuesta de Datos · Fase 1*. https://jotamao1985.github.io/Deep_Learning_Usta/12_Plantilla_Propuesta_de_Datos_Fase1.html

---

**Nota de integridad académica:** la IA se utilizó como apoyo para examinar la estructura de datos y redactar la propuesta. Las decisiones, errores, verificaciones y modificaciones deberán registrarse en la bitácora exigida por la asignatura; no se inventarán experimentos, 
