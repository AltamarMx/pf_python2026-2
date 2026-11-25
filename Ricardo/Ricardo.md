# Proyecto final: Ricardo

**Tema:** Caracterización del recurso eólico de La Ventosa, Oaxaca (2015): distribución de Weibull, densidad de potencia, cizalladura vertical y dirección dominante a dos alturas.

## Los datos

`Ventosa.csv`: 52 560 registros cada 10 min de todo 2015 (365 × 144), sin valores vacíos.

| Columnas | Significado |
|---|---|
| `DECIMAL`, `DIA`, `HORA`, `FECHA` | Tiempo: día decimal, día del año, hora como entero `HHMM` sin ceros (10, 20, …, 100, …, 2400) y fecha `dd/mm/aaaa HH:MM` |
| `VEL15M`, `VEL32M` | Velocidad media a 15 y 32 m (m/s) |
| `DIR15M`, `DIR32M` | Dirección (°) |
| `DESV15M`, `DESV32M` | Desviación estándar de la velocidad (m/s) |
| `MAX15M`, `MAX32M` | Ráfaga máxima (m/s) |

### Puntos a resolver en la limpieza (los reviso en `001_ETL`)

- `FECHA` es día/mes (`dayfirst=True`). `DECIMAL`, `DIA` y `HORA` son redundantes: úsalos sólo para verificar.
- El último registro es `01/01/2016 00:00`, así que la marca de tiempo indica el **fin** del intervalo de 10 min. Decide una convención y documéntala.
- Hay **683 registros con `VEL15M = 0` y `DIR15M = 0` al mismo tiempo** (543 a 32 m), con rachas de hasta 21 registros (~3.5 h). ¿Son calmas o fallas del sensor? En La Ventosa una calma de horas es sospechosa. Decide un criterio y justifícalo.
- Verifica que `MAX ≥ VEL` y que las direcciones estén en [0, 360].

## La función

```python
from tools.viento import caracteriza_viento, exponente_cizalla

caracteriza_viento(df, altura=32)
# -> dict o Serie: v media, k y c de Weibull, densidad de potencia medida (½ρ⟨v³⟩)
#    y la estimada con Weibull (W/m²), dirección dominante, intensidad de turbulencia media

exponente_cizalla(df, z1=15, z2=32, por="hora")
# -> exponente de Hellmann α = ln(v2/v1) / ln(z2/z1), agregado por hora del día o por mes
```

- El ajuste de Weibull se hace con `scipy.stats.weibull_min.fit` (con `floc=0`) o por el método de momentos. Compara ambos.
- La función debe servir con cualquier archivo del mismo formato: otro año u otra estación con las mismas columnas.

## Figuras de verificación sugeridas (002_EDA)

- `VEL15M` contra `VEL32M` en una nube de puntos: la mayoría de los puntos debe quedar por encima de la recta 1:1.
- La serie de un mes, con las rachas de ceros marcadas.
- Histogramas de dirección a las dos alturas.

## Ideas de resultados (elige o propone al menos 2)

1. Histograma de velocidad con la Weibull ajustada a 15 y 32 m, y la tabla de k, c y densidad de potencia.
2. Una rosa de vientos anual, y la de la temporada de "nortes" (nov–feb) contra el resto del año.
3. El ciclo diario de α (día contra noche) y lo que dice de la estabilidad atmosférica.
4. La extrapolación a 80–100 m (altura de buje típica) con el α medido, y la densidad de potencia que resulta.
5. La intensidad de turbulencia (`DESV/VEL`) contra la velocidad, comparada con las clases de la norma IEC 61400-1.

## Ideas de innovación

- **`scipy.stats`** para el ajuste de distribuciones (no se vio SciPy en el curso).
- **`windrose`** para la rosa de vientos.
- **`ipywidgets`**: un selector de mes que actualice la rosa y la Weibull.
- **Tipado** de las funciones.

## Información que debes conseguir

- El origen de la estación (institución, coordenadas, tipo de anemómetro) y el permiso para usar los datos.
- La densidad del aire que vas a usar en La Ventosa (altitud), con su justificación.

> German también trabaja con viento, pero su proyecto trata del **desempeño de una turbina**. El tuyo trata de **caracterizar el recurso**: Weibull, cizalladura y dirección. No compartan código.

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
