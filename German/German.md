# Proyecto final: German

**Tema:** Curva de potencia empírica de un aerogenerador a partir de datos SCADA y su uso para estimar la energía que produciría con el viento de otro sitio (Morelos, a partir de Open-Meteo).

## Los datos

### 1. SCADA del parque eólico La Haute Borne (Engie, Francia)

Viene en el zip `SCADA dataset of the Engie La Haute Borne wind farm _onshore_ in France.zip` (~260 MB descomprimido):

- `static-information.csv`: 4 turbinas Senvion MM82 (R80711, R80721, R80736, R80790) de 2050 kW nominales, buje a 80 m, rotor de 82 m, con su posición GPS.
- `data_description.csv`: el significado y las unidades de cada variable.
- `Cleaned data/Turbine_data_<turbina>_2013-2018_F95.csv` y `..._F99.csv`: 8 archivos (2 versiones por turbina) de ~211 000 registros cada uno. Son datos cada 10 min del 2013-01-01 al 2018-01-13 con 18 columnas: `P_avg` (kW), `Q_avg`, `S_avg`, `Ws_avg` (m/s), velocidades del generador y temperaturas de la caja de engranes, el generador, la góndola y el rodamiento.

### 2. Script de la API de Open-Meteo (`Python API request Open Meteo.txt`)

Pide datos horarios históricos de 2006 a 2026: velocidad del viento a 10 y 100 m, dirección, temperatura, radiación, etc. Usa un *bounding box* sobre Morelos (18.33–19.13 N, 99.49–98.63 W).

### Puntos a resolver en la limpieza (los reviso en `001_ETL`)

- **F95 frente a F99:** averigua en la fuente qué filtro representa cada versión (difieren en ~2 000 registros por turbina). Elige una y justifícalo.
- `static-information.csv` usa `;` como separador, trae BOM y una coma final.
- Las fechas traen zona horaria (`+01:00`). Decide cómo manejarla y sé consistente.
- Une las 4 turbinas en un solo DataFrame en formato largo, con una columna `turbina`, y guárdalo en parquet. Para desarrollar, puedes trabajar con una sola turbina.
- **Open-Meteo:** convierte el script en una libreta. Guarda la respuesta cruda en `data/001_raw/` para que el proyecto **corra sin internet**. Pide sólo las variables que vas a usar; 22 variables en muchos puntos durante 20 años es demasiado. Comprueba que `latitude`/`longitude` (que en el script son los valores de ejemplo, de Berlín) no interfieren con el *bounding box*.
- Los datos SCADA "limpios" no traen temperatura exterior ni ángulo de *pitch*. Tenlo en cuenta si quieres corregir por densidad del aire.

## La función

Dos funciones encadenadas en `tools/`:

```python
from tools.eolica import curva_potencia, energia_esperada

curva = curva_potencia(scada, turbina="R80711", ancho_bin=0.5)
# -> DataFrame por intervalo de viento: v media, P media, desviación estándar, n, Cp

energia_esperada(viento_100m, curva, z_medicion=100, z_buje=80, alfa=1/7)
# -> energía anual (MWh), factor de planta y horas equivalentes
```

- `curva_potencia` usa el **método de intervalos** (*binning*, como en la norma IEC 61400-12-1): agrupa por intervalos de velocidad y promedia la potencia. Descarta los intervalos con pocos datos.
- `energia_esperada` lleva el viento a la altura del buje con la ley de potencia, interpola la curva y suma la energía.
- Las dos funciones deben poder usarse con cualquier turbina del parque y con cualquier serie de viento.

## Figuras de verificación sugeridas (002_EDA)

- Una nube de puntos de P contra Ws por turbina: debe parecer una curva de potencia, con P ≤ 2050 kW.
- La cobertura temporal: registros por mes y por turbina.
- El viento de Open-Meteo en un punto: serie e histograma.

## Ideas de resultados (elige o propone al menos 2)

1. Comparar las curvas de las 4 turbinas. ¿Hay alguna que rinda menos? ¿Cambia la curva de un año a otro (degradación)?
2. El factor de planta real de cada turbina por año, contra el estimado con su curva.
3. Un mapa del factor de planta estimado de una MM82 en cada punto de la malla de Open-Meteo en Morelos.
4. Monitoreo de condición: la temperatura de la caja de engranes contra la potencia. ¿Hay periodos anómalos?

## Ideas de innovación

- **Consumo de una API** (`openmeteo-requests`) con **caché** (`requests-cache`) y reintentos. Justifica por qué la caché importa para la reproducibilidad y para no saturar el servicio.
- **Mapa** con `folium` o con `plotly` (`scatter_map`) para el resultado 3.
- **Tipado** de las funciones.

## Información que debes conseguir

- La licencia y la cita del conjunto de datos de Engie (es *open data*) y el significado de F95 y F99.
- La curva de potencia del fabricante de la MM82, si la encuentras, para comparar contra la empírica.

## Nota sobre el tamaño de la entrega

Los crudos SCADA pesan ~260 MB. Pregunta antes de entregar si van dentro del zip o si basta con el enlace de descarga y un `data/001_raw/LEEME.md` que explique cómo obtenerlos.

---

## Requisitos comunes (iguales para todos los proyectos)

### Estructura del espacio de trabajo (uv)

```
pf_<nombre>/
├── .python-version
├── pyproject.toml              # dependencias agregadas con `uv add`
├── uv.lock
├── README.md                   # qué es el proyecto y cómo reproducirlo
├── data/
│   ├── 001_raw/                # datos crudos, intactos
│   ├── 002_intermediate/       # datos limpios (parquet), salida del ETL
│   └── 003_results/            # tablas de resultados
├── img/                        # figuras exportadas (png/pdf)
├── notebooks/
│   ├── 001_ETL.ipynb           # importación y limpieza
│   ├── 002_EDA.ipynb           # figuras de verificación
│   ├── 003_desarrollo_<funcion>.ipynb
│   ├── 004_uso_<funcion>.ipynb # demostración de uso
│   ├── 005_resultados.ipynb    # tus figuras o tratamientos propios (mínimo 2)
│   └── tools/
│       ├── __init__.py
│       └── <modulo>.py
└── presentacion/               # diapositivas (pdf o html)
```

Puedes partir una etapa en varias libretas. Lo que importa es que la numeración siga el orden de lectura.

### Reglas de trabajo (las del curso)

1. Las libretas van numeradas por orden de lectura (`001_...`), sin espacios ni acentos. Cada una se entrega ejecutada de arriba abajo, en orden (*Restart & Run All*), con sus salidas.
2. Usa siempre rutas relativas (`../data/001_raw/...`), nunca rutas absolutas.
3. Los paquetes se agregan con `uv add`, nunca con `pip install` desde la libreta. Con `uv sync` debe bastar para reproducir el entorno.
4. Los datos crudos no se tocan. El ETL escribe en `data/002_intermediate/`.
5. Una libreta, una pregunta. Escribe en Markdown por qué tomaste cada decisión.

### La función: el centro del proyecto

- Vive en `notebooks/tools/<modulo>.py` y se importa con `from tools.<modulo> import <funcion>`.
- **Supone que los datos siempre tienen el formato limpio que produce `001_ETL`.** El docstring describe ese formato: índice, nombres de columnas y unidades. Si la función recibe algo distinto, debe fallar con un mensaje claro (`raise ValueError("...")`), no con un error críptico de pandas.
- Devuelve un resultado (DataFrame, Serie, diccionario…), no sólo una figura. Graficar es opcional: un argumento `grafica=True` o una función aparte.
- Lleva un docstring completo (qué hace, parámetros, qué devuelve y un ejemplo) que se pueda ver con `help()` y con `Shift+Tab`.
- Puede usar funciones auxiliares del mismo módulo. Lo importante es poder reutilizarla con **otro archivo del mismo formato**: otra corrida, otro año, otra muestra.

### Figuras

- **Figuras de verificación** (en `002_EDA`): comprueban que los datos se importaron bien y se comportan como se espera. No cuentan como resultado.
- **Figuras o tratamientos propios** (en `005_resultados`): **al menos dos**, propuestos por ti, que den un resultado: un número, una comparación o una conclusión. Más abajo hay ideas; puedes usarlas o proponer otras.

### Innovación

Incluye al menos una herramienta o técnica que no se haya visto en el curso, **y justifícala**. En la presentación tienes que explicar qué problema resuelve en tu proyecto y por qué conviene más que lo visto en clase. Si eliges el tipado de funciones (*type hints*), debes saber explicar qué hace y qué no hace: Python no lo verifica al ejecutar; sirve para documentar, para el autocompletado del editor y para revisar el código con herramientas como `mypy` o `pyright`.

### Presentación (15 minutos)

La presentación trata de **las funciones**, no de la limpieza; la limpieza la reviso yo en el repositorio. Distribución sugerida:

- 2 min: contexto y datos.
- 7 min: la función (qué recibe, qué devuelve, demostración).
- 3 min: resultados.
- 2 min: innovación y su justificación.
- 1 min: cierre.

Lleva la demostración ya ejecutada por si algo falla en vivo.

### Entrega

Un zip del espacio de trabajo **sin `.venv/`**:

```bash
zip -r pf_<nombre>.zip pf_<nombre> -x "*/.venv/*" "*/.ipynb_checkpoints/*" "*/__pycache__/*" "*.DS_Store"
```

Antes de zipear: borra `.venv/`, corre `uv sync` y ejecuta cada libreta con *Restart & Run All* para comprobar que todo se reproduce desde cero.
