# Proyecto final: Roberto

**Tema:** Estrés térmico en La Paz, BCS (2017–2025) con el Índice Climático Térmico Universal (UTCI): detección de episodios de calor y su evolución.

## Los datos

### `la_paz_utci_2017-2025.csv`

Un solo archivo CSV (6.9 MB) con 78 888 registros horarios del 2017-01-01 00:00 al 2025-12-31 23:00 UTC. No falta ninguna marca de tiempo, aunque hay 10 horas sin valor (ver abajo).

| Columna | Significado |
|---|---|
| `time_utc`, `time_local` | Tiempo en UTC y en hora local. El desfase es **siempre de UTC−7** |
| `utci_c` | UTCI en °C para La Paz: coincide con una **interpolación bilineal** de los 4 nodos en ~24.14 N, 110.31 W (diferencia máxima de 0.04 °C) |
| `utci_<lat>_<lon>_c` | UTCI en los 4 nodos de la malla de 0.25° que rodean La Paz |
| `utci_nodo_cercano_c` | UTCI en el nodo más cercano |
| `categoria` | Categoría de estrés térmico, en texto |

Categorías presentes: sin estrés térmico (44 689 h), calor moderado (18 267), calor fuerte (12 155), frío ligero (3 222), calor muy fuerte (541) y frío moderado (4).

### Puntos a resolver en la limpieza (los reviso en `001_ETL`)

- **`utci_nodo_cercano_c` es idéntica a `utci_24.25_-110.25_c`** (diferencia 0 en todos los registros). Documéntalo y decide si la conservas.
- **Comprueba que `utci_c` es la interpolación bilineal de los 4 nodos.** Reprodúcela con las coordenadas de La Paz y reporta la diferencia. Averigua también la fuente de los datos (¿ERA5-HEAT de Copernicus?).
- **Hay 10 horas sin valor**, en tres fechas:
  - 2017-08-31 23:00 a 2017-09-01 01:00 UTC (3 h)
  - 2021-09-10 08:00 UTC (1 h)
  - 2023-10-22 entre 02:00 y 14:00 UTC (6 h)

  Las tres fechas coinciden con huracanes que afectaron a BCS (Lidia 2017, Olaf 2021 y Norma 2023). Verifícalo. Decide si interpolas o dejas los huecos, y justifica la decisión, porque afecta la detección de episodios.
- **La frontera de las categorías:** al recalcular la categoría con los umbrales oficiales del UTCI (9, 26, 32, 38 y 46 °C) salen 44 diferencias con la columna original. Todas son valores que caen **exactamente en un umbral** (9.0, 26.0, 32.0, 38.0): el archivo los asigna a la categoría inferior (intervalos `(a, b]`). Elige una convención, documéntala y úsala en la función. Convierte `categoria` a un `Categorical` **ordenado** (de frío a calor).
- **Hora local:** `time_local` es UTC−7 fija. Hasta octubre de 2022, BCS usaba horario de verano, así que en esos veranos la hora del archivo no es la hora "de reloj". Para el análisis por hora del día, decide cuál usas y justifícalo.
- Si usas `time_local` como índice, el primer año queda con 7 h de 2016 y el último termina a las 16:00 del 31/12/2025. Toma en cuenta esos años incompletos al contar horas por año.

Estos datos piden poca limpieza. Por eso la exigencia está en la validación (interpolación, categorías, huecos) y en la función.

## La función

```python
from tools.utci import episodios_estres

episodios_estres(utci, umbral=32, duracion_minima=3)
# -> DataFrame, un renglón por episodio: inicio, fin, duración (h), UTCI máximo
#    y grados-hora por encima del umbral
```

- Un **episodio** es un bloque de horas consecutivas por encima del umbral que dura al menos `duracion_minima` horas. El reto es identificar esos bloques con pandas (`diff`, `cumsum` y `groupby`) sin recorrer fila por fila.
- Sirve con el umbral de "calor fuerte" (32 °C), con el de "muy fuerte" (38 °C) o con frío, si se invierte la condición.
- Función auxiliar sugerida: `horas_por_categoria(df, freq="ME")` → tabla de horas por categoría y periodo.

## Figuras de verificación sugeridas (002_EDA)

- La serie completa y un año típico de `utci_c`.
- Los 4 nodos comparados en una misma semana de verano.
- El histograma de UTCI con los umbrales de las categorías marcados.

## Ideas de resultados (elige o propone al menos 2)

1. ¿Aumentan las horas de calor fuerte por año entre 2017 y 2025? (Barras por año, con su tendencia.)
2. Un mapa de calor de mes contra hora local con la probabilidad de estrés por calor: ¿qué horas son de riesgo para el trabajo al aire libre?
3. Los 10 episodios más largos o intensos del periodo y en qué años ocurrieron.
4. Las diferencias entre nodos (costa contra interior) y lo que significan para elegir una fuente de datos.

## Ideas de innovación

- **Calendario de calor** con `calplot` o `july`: un año visto como calendario, coloreado por las horas de estrés.
- **`folium`** para mapear los 4 nodos y La Paz.
- **`ipywidgets`**: deslizadores de umbral y duración que actualicen la tabla de episodios.
- **Tipado** de las funciones.

## Información que debes conseguir

- La fuente de los datos (ERA5-HEAT u otra) y su licencia.
- La tabla oficial de categorías del UTCI, citada.

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
