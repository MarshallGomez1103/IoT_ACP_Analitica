# ACP conjunto de volumen y ambiente

El notebook [01_Fusion_ACP_volumen_ambiente.ipynb](01_Fusion_ACP_volumen_ambiente.ipynb) reconstruye las fuentes, genera una distancia simulada de 5–20 cm a partir del volumen del Neusa y ajusta un ACP conjunto con cuatro variables ambientales. Presenta los coeficientes de PC1 y su descomposición exacta en un bloque hídrico y otro ambiental.

El resultado es un **índice descriptivo con una distancia simulada**. Los porcentajes de los bloques corresponden a una normalización L1 explícita de los coeficientes. No son ponderaciones de riesgo validadas ni sustituyen directamente el 80/20 del firmware.

## Fundamento de la distancia y el volumen

La sección 4.1 del notebook distingue el montaje del sensor, el nivel del agua y el volumen almacenado. El [manual de Vaisala RWS200 para el SR50A](https://docs.vaisala.com/r/M211608EN-AA/en-US/GUID-903A8C27-3DC8-45CB-92F9-AAF8B8783692?contentId=mRvlHRVufJ2WXQGl8XlDEw) indica al menos 50 cm sobre el nivel máximo esperado; esa especificación corresponde a ese equipo. El [HC-SR04 de SparkFun](https://www.sparkfun.com/ultrasonic-distance-sensor-hc-sr04.html) tiene un rango nominal diferente y necesita validación en el montaje utilizado.

La [guía de medición de almacenamiento del State Water Resources Control Board](https://www.waterboards.ca.gov/waterrights/water_issues/programs/diversion_use/docs/res_measure6.pdf) describe la conversión de nivel a volumen mediante una curva de capacidad. La curva cota–volumen y la referencia vertical del Neusa no están incorporadas a este análisis: el intervalo 5–20 cm continúa siendo una simulación, no una distancia física reconstruida.

La revisión metodológica distingue tres cuestiones: la justificación del montaje, la correspondencia nivel–volumen y la validación del evento de alarma. Cambiar el intervalo simulado a 5–15 cm no resuelve esas cuestiones ni modifica el z-score de la variable hídrica. El notebook incluye una comprobación explícita de esa equivalencia.

Como referencia operativa colombiana, el [estudio de la Universidad de los Andes para el Consejo Nacional de Operación](https://stage.cno.org.co/system/files/archivosAdjuntos/2020-medicion-de-variables-hidricas-final.pdf), secciones 2.6, 4.1.6 y 4.3.6, documenta la obtención de tablas nivel–volumen mediante batimetría. Describe prácticas de operadores hidroeléctricos; no identifica por sí solo el equipo ni la curva utilizados por CAR en los boletines del Neusa.

## Reproducción

Con Python 3.11, desde esta carpeta:

```bash
python -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
python -m jupyterlab 01_Fusion_ACP_volumen_ambiente.ipynb
```

Seleccionar el kernel del entorno y ejecutar todas las celdas en orden. También se puede utilizar un entorno existente con las mismas bibliotecas. La extracción de tablas de los PDF requiere `pdftotext`, de Poppler, disponible en el `PATH`.

El análisis normal utiliza las fuentes originales incluidas y no requiere Internet. La variable `DESCARGAR_DE_NUEVO`, en la sección 2, permite obtener nuevamente las fuentes desde sus URL públicas. Las descargas van a una carpeta separada y conservan los originales. Los PDF CAR se comparan por SHA-256. NASA incluye una latencia de respuesta variable en `times`: se registra el hash de la descarga y se compara el JSON completo excluyendo únicamente ese campo. Las mediciones y todos los demás metadatos deben coincidir; en caso contrario se detiene la comprobación. El registro distingue una nueva descarga de una copia ya almacenada en caché.

## Datos incluidos

- **CAR:** 31 PDF de agosto de 2024, con 124 registros correspondientes a Neusa, Sisga, Tominé y El Hato.
- **NASA POWER:** JSON original con 1.096 días de 2023–2025 para el punto solicitado cerca del Neusa.
- **CSV derivados:** las tablas previas se contrastan con la reconstrucción desde los originales.
- **Manifiesto:** `datos/fuentes.json` contiene las URL y los hashes. El notebook muestra enlaces descargables a las 32 fuentes originales.

La matriz conjunta contiene las 31 fechas de agosto de 2024 con datos de ambas fuentes. No se imputa volumen para otros meses ni se asigna el clima de Neusa a otros embalses. Las series IDEAM del estudio previo no se mezclan con esta matriz debido a sus diferencias de fuente y a la descripción incompatible de la humedad histórica.

## Resultados

La carpeta `resultados/` contiene las tablas reconstruidas, la distancia simulada, medias y desviaciones, correlaciones, coeficientes, cargas, varianza explicada, reparto L1, sensibilidad temporal y figuras. `parametros_indice_descriptivo.json` reúne los parámetros necesarios para reproducir las puntuaciones.

`01_Fusion_ACP_volumen_ambiente.html` permite consultar el código, las explicaciones y las salidas sin abrir Jupyter. Para modificar o volver a ejecutar el análisis se utiliza el archivo `.ipynb`.

## Tratamiento de calidad y reproducción del índice

La sección 5.1 muestra los controles de duplicados, faltantes, finitud, calendario y rangos físicos, y registra las observaciones extremas mediante IQR. Las banderas estadísticas no se interpretan automáticamente como errores: se conservan los valores originales admisibles. No se imputan, interpolan ni eliminan observaciones para favorecer el resultado del ACP.

Los porcentajes CAR se contrastan con intervalos derivados de sus decimales publicados. El porcentaje y la capacidad de regulación originales se conservan cuando son compatibles con esa precisión; no se sobrescriben por pequeñas diferencias de redondeo.

La sección 12 vuelve a leer el JSON de parámetros y la matriz exportada para reconstruir PC1, S y el índice final sin utilizar el modelo ajustado en memoria. La tabla final separa la normalización L1 elegida para la expresión de dos bloques de las contribuciones cuadráticas al eje. Cada estadístico conserva su definición y su uso.

Los archivos adicionales de comprobación son:

| Archivo en `resultados/` | Contenido |
|---|---|
| `control_redondeo_CAR.csv` | Compatibilidad de los porcentajes y la capacidad de regulación con la precisión publicada. |
| `trazabilidad_muestra.csv` | Paso de las fuentes completas a las 31 fechas de la matriz final. |
| `calidad_variables.csv` | Faltantes, extremos y límites IQR por variable del ACP. |
| `observaciones_IQR.csv` | Valores señalados y tratamiento aplicado. |
| `definicion_porcentajes.csv` | Definición, valor y uso de cada reparto presentado. |

## Interpretación

El volumen máximo observado se representa con 5 cm y el mínimo con 20 cm. Son extremos del mes, no estados físicos de lleno y seco. La distancia es una transformación lineal declarada del volumen, sin una curva nivel–volumen ni geometría del sensor.

La expresión final utiliza una distancia **estandarizada** y un bloque ambiental nuevo que conserva signos negativos. No equivale a combinar dos riesgos positivos de 0–100. Tampoco se valida un umbral de alarma ni se demuestra evaporación. Para esas conclusiones se requieren mediciones locales y eventos de referencia independientes.

El enunciado y la rúbrica del segundo corte delimitan el alcance académico del anexo. Las pruebas físicas, la Wiki, el video y los demás artefactos del reto se evalúan por separado.
