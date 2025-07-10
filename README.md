# Análisis de Irradiancia Solar

## Resumen

Este proyecto presentó un análisis exploratorio y de series de tiempo sobre la irradiancia solar en una localidad específica del norte de México (Lat: 31.3, Lon: -113.51), utilizando datos obtenidos mediante la API de la NASA POWER. Se aplicaron técnicas de limpieza, validación, visualización y modelado estadístico para evaluar la viabilidad del recurso solar con fines de generación fotovoltaica. El trabajo incluyó métricas descriptivas, visualizaciones detalladas, descomposición estacional y análisis de autocorrelación, destacando un enfoque riguroso y reproducible con herramientas de ciencia de datos en Python.

## Introducción

En la política del desarrollo del sistema eléctrico mexicano mencionada en el PRODESEM, se hizo referencia a cumplir metas necesarias para llevar a cabo la transición energética a un modelo más limpio. Por ello, fue necesario localizar zonas que cumplieran con cantidades mínimas de irradiancia solar para tecnologías de paneles fotovoltaicos, así como tecnologías concentradoras.  
Este análisis se realizó con el fin de evaluar si la localidad ubicada en el punto LATITUD = 31.3, LONGITUD = -113.51, contaba con condiciones climáticas favorables en torno a la irradiancia solar.

## Origen de los datos y descripción

Los datos se obtuvieron de las mediciones tomadas por la NASA, disponibles en [https://power.larc.nasa.gov/](https://power.larc.nasa.gov/). Para acceder a una gran cantidad de datos, se desarrolló un programa que utilizó la API de la NASA.  
Los datos solicitados a través de la API se describieron de la siguiente forma:

1. **`Irradiancia_Global_Horizontal`**: Energía solar total (directa y difusa) de onda corta (0.3–3 µm) que llega a una superficie horizontal bajo condiciones reales de cielo. Es la métrica más importante para paneles fotovoltaicos planos.
2. **`Irradiancia_Directa_Normal`**: Energía solar directa sobre una superficie perpendicular al sol, incluyendo efectos atmosféricos. Es crítica para tecnologías de concentración.
3. **`Irradiancia_Difusa_Horizontal`**: Energía solar difusa que llega a una superficie horizontal, útil en condiciones nubladas.
4. **`Indice_Claridad`** (`Kt`): Índice que indica cuán despejado está el cielo; valores cercanos a 1 indican alto potencial solar.

## Conversión, limpieza y validación de los datos

Dado que la API entregó un archivo CSV con columnas separadas para fecha y hora (`YEAR`, `MO`, `DY`, `HR`) en formato UTC, fue necesario unirlas y transformarlas al formato `%Y%m%d%H`, para luego convertirlas en objetos `datetime`. Posteriormente, se ajustó el horario al centro del país utilizando `pandas`.

La limpieza y validación de los datos se realizaron en dos etapas:

- En la **primera limpieza**, se eliminaron las columnas `YEAR`, `MO`, `DY` y `HR`, y las filas con valores de `Irradiancia_Global_Horizontal`, `Irradiancia_Directa_Normal` e `Irradiancia_Difusa_Horizontal` igual a cero, ya que corresponden a horas sin luz solar. También se reemplazaron los valores `-999`, que indican datos faltantes según la documentación de la API.

- La **validación** se realizó con funciones como `.info()`, `.isnull().sum()` y `.sort_values()` para detectar y cuantificar valores `NaN`. Se generó una gráfica para visualizar los datos faltantes, con un resultado del 2.07% de ausencia en las principales columnas.

- En la **segunda limpieza**, se detectó que los valores `-999` comenzaron a partir del 31/03/2026 a las 18:00 h, lo que indicó que la base de datos no había sido actualizada. Por ello, se eliminaron esos datos. Luego de esta etapa, ya no se encontraron datos nulos, cero ni `NaN`.

**Resumen del preprocesamiento**: el archivo original tenía 214,896 filas y 11 columnas. Tras la limpieza, se conservaron 111,312 filas y 7 columnas.

## Análisis de datos

El análisis comenzó utilizando la función `.describe()` para obtener medidas estadísticas básicas como media, desviación estándar, mínimo, máximo y cuartiles.

## Visualizaciones de los datos

Se dividieron en dos grupos:

- **Índice de claridad**: El diagrama de caja mostró valores atípicos bajos. La media anual fue superior a 0.65, indicando cielos mayormente despejados. El histograma confirmó una concentración de datos entre 0.7 y 0.8.

- **Tipos de irradiancia**:  
  - `Irradiancia_Directa_Normal`: media > 700 W/m².  
  - `Irradiancia_Global_Horizontal`: media > 500 W/m².  
  - `Irradiancia_Difusa_Horizontal`: media < 100 W/m², con valores atípicos altos.  
  Los histogramas mostraron distribución homogénea para `Irradiancia_Global_Horizontal`, concentración entre 650–1000 W/m² en la directa, y entre 20–180 W/m² en la difusa.

## Funciones de probabilidad

Se graficaron las funciones de probabilidad acumulada (CDF) para estimar la probabilidad de obtener un valor `x` o menor de irradiancia solar. Además, se propuso ajustar una distribución normal para modelar los datos.

## Visualización exploratoria con gráficos de línea

### Serie completa

Se graficó la serie completa en tres periodos de tres años. Se observó un patrón estacional anual: máximos durante verano y mínimos en invierno, con picos promedio > 1000 W/m².

### Promedio diario por día del año

El promedio alcanzó su punto máximo entre los días 120 y 130 (~mayo), con valores > 640 W/m². La temporada alta abarcó de marzo a agosto.

### Perfil horario promedio

La irradiancia directa dominó entre las 13:00 y 14:00 h (> 800 W/m²). La componente difusa fue baja, lo cual indica condiciones favorables para tecnologías de concentración.

### Mapa de calor mensual

El mapa mostró irradiancia > 500 W/m² entre 12:00 y 15:00 h todo el año. Mayo y junio destacaron con 9 horas diarias por encima de este umbral, seguidos de abril, julio y agosto con 8.

## Descomposición de series de tiempo

Se aplicó una descomposición aditiva diaria a `Irradiancia_Global_Horizontal`, revelando una tendencia estacional anual coherente y una componente diaria clara con máximos entre 13:00 y 14:00 h.  
Los residuales presentaron un patrón ondulante que cruzó el eje cero, con mesetas positivas y picos negativos redondeados, lo que sugiere posibles efectos adicionales no modelados.

## Análisis de autocorrelación

Se aplicó ACF y PACF a las series diaria y horaria.

- En la **serie diaria**, hubo autocorrelación fuerte en el rezago 1 (lag=1), indicando persistencia. La ACF decayó lentamente, lo cual sugiere tendencia o estacionalidad semanal.
- En la **serie horaria** (incluyendo valores cero por la noche), se observaron picos en múltiplos de 24 horas (lag=24, 48, 72…), confirmando estacionalidad diaria clara.

### Detección de anomalías con Isolation Forest

Se aplicó el algoritmo `Isolation Forest` para identificar posibles valores anómalos en la variable `Irradiancia_Global_Horizontal`, utilizando un umbral de contaminación del 1% (`contamination=0.01`). Esta técnica es útil para detectar puntos atípicos sin necesidad de suposiciones sobre la distribución de los datos.

Los resultados mostraron que se detectaron **N** anomalías (donde *N* es el número impreso al final del script). Estas anomalías fueron visualizadas en rojo sobre la curva de irradiancia en función del tiempo.

La gráfica mostró que las anomalías se concentraron principalmente en ciertos periodos específicos, lo que puede deberse a mediciones erróneas, condiciones climáticas inusuales o fallas instrumentales. Esta información es valiosa tanto para depurar datos como para futuras mejoras en el monitoreo y modelado.

El uso de `Isolation Forest` permite mejorar la robustez del análisis, ya que destaca observaciones que se comportan de forma significativamente distinta al patrón general, sin requerir un modelo supervisado.

## Conclusiones generales

El análisis realizado demostró que la localidad evaluada presenta condiciones altamente favorables para el aprovechamiento de energía solar. La `Irradiancia_Global_Horizontal` mostró una estacionalidad consistente, con máximos regulares durante los meses de verano, mientras que el `Indice_Claridad` indicó cielos predominantemente despejados durante gran parte del año.

El tratamiento riguroso de los datos, desde su adquisición mediante API hasta su limpieza y validación, garantizó la confiabilidad del análisis. Las visualizaciones permitieron identificar patrones clave, y los análisis de series de tiempo y autocorrelación revelaron una estacionalidad diaria y anual bien definida.

Este proyecto no solo permitió evaluar la viabilidad de generación fotovoltaica en la zona, sino que sentó las bases para desarrollos futuros en pronóstico de irradiancia y dimensionamiento de sistemas solares. La integración de herramientas de ciencia de datos y series de tiempo reforzó el valor técnico del trabajo y su aplicabilidad en proyectos reales de energía renovable.


