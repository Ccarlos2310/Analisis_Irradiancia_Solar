# Análisis de Irradiancia Solar

## Introducción

En la política del desarrollo del sistema eléctrico mexicano mencionada en el PRODESEM, se hace referencia a cumplir metas necesarias para llevar a cabo la transición energética a un modelo más limpio. Es por eso que se deben localizar zonas las cuales cumplan con cantidades mínimas de irradiancia solar para las tecnologías de paneles fotovoltaicos, así como las tecnologías concentradoras.

Este análisis de datos se realizó con el fin de saber si la localidad que se encuentra en el punto LATITUD = 31.3, LONGITUD = -113.51 cuenta con condiciones climáticas favorables en torno a la irradiancia solar.

---

## Origen de los datos y descripción

Los datos fueron obtenidos de las mediciones tomadas por la NASA y que se pueden consultar en [https://power.larc.nasa.gov](https://power.larc.nasa.gov/). Para acceder a una gran cantidad de datos, fue necesario hacer un programa que utilizara la API de la NASA.

Los datos solicitados a través de la API se mencionan y describen a continuación:

1. **Irradiancia global horizontal**  
   Es el total de energía solar de onda corta (0.3–3 µm) que llega a una superficie horizontal, considerando condiciones reales de cielo (nubes, partículas). Esta es la métrica más fundamental y crucial para los paneles fotovoltaicos. Representa la energía solar total (directa y difusa) que llega a una superficie horizontal. Los paneles fotovoltaicos, especialmente los paneles planos (no concentradores), capturan tanto la radiación solar directa como la difusa.

2. **Irradiancia directa normal**  
   Es la energía solar directa recibida por una superficie perpendicular al Sol, incluyendo efectos atmosféricos. Esta medición es crítica para tecnologías de energía concentrada, que solo utilizan la luz directa.

3. **Irradiancia difusa horizontal**  
   Es la energía solar difusa que llega a una superficie horizontal, incluyendo efectos atmosféricos. Esta métrica es una parte significativa de la energía solar, especialmente en días nublados o con presencia de aerosoles.

4. **Índice de claridad (Kt)**  
   Este índice da una idea rápida de cuán despejado está el cielo en una ubicación dada. Un valor cercano a 1 indica cielos muy despejados, lo que generalmente se traduce en un alto potencial de generación. Te permite evaluar la calidad del recurso solar y cómo las nubes afectan la llegada de radiación.

---

## Conversión, limpieza y validación de los datos

La API entrega un archivo CSV con la fecha y hora separadas (YEAR, MO, DY, HR) y en formato UTC. Fue necesario juntar las columnas y darles formato `%Y%m%d%H`, así como convertir los datos de esta nueva columna en objetos `datetime`. Posteriormente, se transformaron para ajustarlos al horario del centro del país, utilizando la librería **pandas**.

La limpieza y validación se realizaron en dos partes:

- **Primera limpieza**
  - Se eliminaron columnas innecesarias: `YEAR`, `MO`, `DY`, `HR`.
  - Se eliminaron filas con valores cero en `Irradiancia_Global_Horizontal`, `Irradiancia_Directa_Normal`, `Irradiancia_Difusa_Horizontal`, ya que corresponden a horas sin luz solar.
  - Se reemplazaron los valores `-999`, que según la documentación de la NASA indican datos faltantes o con error.

  Para validar:
  - `.info()` se usó para revisar el tipo de datos y estructura.
  - `.isnull().sum().sort_values()` para conocer la cantidad de valores nulos por columna.
  - Se graficó la proporción de datos faltantes.
  - El porcentaje de datos faltantes fue de **2.07 %** en las cuatro variables clave.

- **Segunda limpieza**
  - Los valores `-999` se presentaban desde el 31/03/2026 a las 18:00 hrs hasta la fecha actual, lo que indica que la base de datos no se ha actualizado.
  - Se eliminaron esos registros.
  - Tras esto, se confirmó que no quedaban valores nulos, cero o `NaN`.

**Resultado**:  
El archivo crudo tenía **214,896 filas y 11 columnas**. Tras la limpieza quedaron **111,312 filas y 7 columnas**.

---

## Análisis de datos

Se utilizó la función `.describe()` para obtener medidas de tendencia central, dispersión y cuartiles sobre las variables numéricas.

---

## Visualizaciones de los datos

Las visualizaciones se dividieron en dos grupos:

- **Índice de claridad**
  - El diagrama de caja y bigote mostró valores atípicos por debajo del bigote inferior.
  - La media anual está por encima de **0.65**, lo que indica cielos mayormente despejados.
  - El histograma mostró una concentración de valores entre **0.7 y 0.8**, lo cual es favorable.

- **Tipos de irradiancia**
  - La **irradiancia directa normal** tiene una media por encima de **700 W/m²**.
  - La **irradiancia global horizontal** tiene una media por encima de **500 W/m²**.
  - Ninguna de estas variables mostró datos atípicos.
  - La **irradiancia difusa horizontal** tiene una media por debajo de **100 W/m²**, pero sí presentó valores atípicos por encima del bigote superior.
  - El histograma muestra una distribución más o menos homogénea en irradiancia global; en la directa, hay acumulación entre **100 y 650 W/m²**; en la difusa, los valores predominan entre **20 y 180 W/m²**.

---

## Funciones de probabilidad

Se graficaron funciones de probabilidad y se escribió el código para calcular la probabilidad de obtener un valor *x* o menor de irradiancia solar.  
También se hizo una propuesta para replicar los datos usando una función normal.

