# Examen 3 - Internet de las Cosas: Procesamiento ETL, Análisis de Espectro y Dashboard

Este repositorio contiene la solución al Examen 3 de Internet de las Cosas (IoT). El proyecto se centra en el procesamiento de datos provenientes de una estación de monitoreo móvil en Medellín, la cual recolectó información de geolocalización (GPS), variables ambientales (temperatura) y mediciones de espectro de radiofrecuencia (RF) alrededor de los 850 MHz.

El objetivo principal es ejecutar un proceso **ETL (Extracción, Transformación y Carga)** robusto para limpiar y corregir mediciones erróneas, calcular indicadores estadísticos, trazar la ruta de medición y preparar los datos para un dashboard interactivo.

---

## 1. Descripción del Proceso ETL y Calidad de Datos

La estación móvil generó múltiples archivos de medición. Debido a las condiciones del entorno urbano, los sensores de GPS y los receptores RF presentaron anomalías que requirieron un tratamiento específico.

### 1.1. Extracción y Filtrado Inicial
*   **Lectura de archivos:** Se procesaron un total de **63 archivos** de medición en bruto.
*   **Descarte de pruebas:** Se identificaron y descartaron **2 archivos** que correspondían a pruebas de calibración antes del recorrido oficial.
*   **Datos válidos:** El dataset final quedó compuesto por **61 mediciones temporales** válidas.

### 1.2. Limpieza e Imputación de Datos de GPS
En entornos urbanos, los edificios altos o los túneles causan pérdida de señal satelital o reflejos que afectan la precisión (efecto multitrayecto). 
*   **Criterio de rechazo:** Se marcaron como "no válidos" aquellos puntos donde el GPS no logró fijar posición (`fix == false`) o donde el indicador de dilución de precisión horizontal (HDOP) era superior a **5.0** (lo que indica un margen de error inaceptable).
*   **Imputación:** Para no perder las mediciones de espectro asociadas a estos puntos, las coordenadas (latitud y longitud) faltantes o erróneas se **imputaron utilizando interpolación lineal**. Esto significa que el sistema trazó una línea recta entre la última coordenada válida conocida y la siguiente, estimando la posición del vehículo en función del tiempo transcurrido.

### 1.3. Corrección de Espectro RF
Los datos de potencia recibida (en dBm) del analizador de espectro contenían ruido instrumental y anomalías:
*   **Eliminación de la espiga DC (DC Spike):** Los receptores SDR (Software Defined Radio) suelen generar un pico de potencia artificial exactamente en la frecuencia central de sintonización (en este caso, 850 MHz) debido a fugas de corriente continua en el hardware. Este valor fue eliminado y reemplazado mediante interpolación con las frecuencias adyacentes.
*   **Filtro de Hampel para ruido:** Se aplicó un **filtro de Hampel** para suavizar la señal. Este algoritmo recorre el espectro con una ventana móvil, calcula la mediana local y la desviación absoluta de la mediana (MAD). Si encuentra una "espiga" aislada (un valor atípicamente alto o bajo que supera un umbral de desviaciones), la reemplaza por la mediana local, eliminando así el ruido impulsivo sin perder la forma real de las señales de telecomunicaciones presentes.

---

## 2. Indicadores y Resultados Clave

Una vez procesado el dataset, se calcularon las siguientes métricas e indicadores:

*   **Completitud de los datos:** **100%** (Gracias a la imputación, ninguna medición de espectro se quedó sin coordenadas espaciales).
*   **Calidad del GPS Original:** **96.7%** de las mediciones iniciales tenían un GPS válido, requiriendo imputación solo en el 3.3% del recorrido.
*   **Distancia Recorrida:** Se calculó la distancia geodésica acumulada punto a punto, resultando en un recorrido total de aproximadamente **26.2 km** a través de la ciudad de Medellín.
*   **Análisis de Correlación:** Se ejecutaron pruebas estadísticas (Coeficientes de **Pearson** y **Spearman**) para evaluar la incidencia de la temperatura interna del equipo sobre los niveles de ruido del espectro o la calidad de recepción, generando una tabla detallada de indicadores.

---

## 3. Tecnologías Utilizadas

*   **Python:** Lenguaje principal del proyecto.
*   **Pandas & NumPy:** Manipulación de DataFrames, interpolación lineal y cálculos matriciales.
*   **SciPy:** Funciones estadísticas (Pearson, Spearman) y procesamiento de señales (Filtro Hampel).
*   **Folium / Plotly:** Generación del mapa interactivo con la ruta de la estación móvil.
*   **Dash:** Framework para la construcción de la aplicación web interactiva (Parte 3 del examen).

---

## 4. Estructura del Repositorio

```text
├── medidas_2026_20/        # Archivos de datos de muestreo
├── IoT_ETL.ipynb           # Jupyter Notebooks con graficas y dashboard
└── README.md               # Este archivo
```
