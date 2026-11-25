# Proyecto final: Emiliano

**Tema:** ¿Cuánto cuesta la electricidad en México? Un calculador de facturas a partir de las tarifas finales de suministro básico, con el consumo nacional como contexto.

## Los datos

| Archivo | Contenido | Forma |
|---|---|---|
| `consumo_final_energia_electrica_2016_2025.csv` | Consumo final nacional de electricidad, mensual (GWh) | 120 filas, `fecha` y `consumo_final_electricidad_gwh` |
| `consumo_final_energia_electrica_enero_julio_2026.csv` | El mismo indicador de enero a julio de 2026 | 7 filas y una columna extra, `fecha_levantamiento_datos` |
| `tarifas_finales_suministro_basico_diciembre_2027_septiembre_2026.csv` | Tarifas finales de suministro básico por componente | 70 278 filas × 16 columnas |

Las tarifas tienen 12 categorías tarifarias (DB1, DB2, PDBT, GDBT, RABT, RAMT, APBT, APMT, GDMTO, GDMTH, DIST, DIT) y 17 divisiones tarifarias. Hay 5 periodos horarios (No aplica, Base, Intermedio, Punta, Semipunta) y 3 tipos de cargo (único, variable, fijo), cada uno con su unidad: $/mes, $/kWh o $/kW. Los componentes son transmisión, distribución, CENACE, operación del suministro, SCnMEM, generación, capacidad y total.

### Puntos a resolver en la limpieza (los reviso en `001_ETL`)

- El nombre del archivo de tarifas dice "diciembre_2027", pero los datos empiezan en **diciembre de 2017**. Anótalo; no renombres el crudo.
- Los dos archivos de 2026 traen BOM (`encoding="utf-8-sig"`). El archivo de 2026 es preliminar (`fecha_levantamiento_datos`): decide qué hacer con esa columna y une ambos archivos en una sola serie mensual.
- `anio` y `mes` (mes en texto, en español) deben convertirse en una fecha.
- Hay nombres de división inconsistentes ("Valle de México Sur" frente a "Valle de Mexico Norte"). Normalízalos.
- Muchas celdas de componentes están vacías porque ese componente no aplica a ese cargo. Distingue entre "no aplica" y "falta el dato", y verifica que `total` sea la suma de los componentes.
- El formato es largo (una fila por categoría, división, periodo y cargo). Decide qué forma conviene guardar en `002_intermediate`.

## La función

Un **calculador de facturas** que, a partir de la tabla de tarifas limpia, calcule el costo de un consumo dado:

```python
from tools.tarifas import calcula_factura

calcula_factura(tarifas, kwh=250, categoria="DB1", division="Centro Sur", fecha="2025-06")
# -> DataFrame con el desglose por componente (cargo fijo + kWh × cargo variable) y el total
```

- Si recibe un rango de fechas, debe devolver la serie del costo del **mismo consumo** a lo largo del tiempo.
- Recomendación de alcance: empieza por las categorías sin periodo horario (DB1, DB2, PDBT y GDBT). Las tarifas horarias (GDMTH) son una extensión opcional en la que `kwh` sería un diccionario por periodo y además entra la demanda en kW.
- La función debe rechazar con un mensaje claro una categoría o división que no exista.
- Aclara en el docstring qué **no** incluye: IVA, DAP y subsidios. Las tarifas domésticas subsidiadas (1, 1A–1F) no vienen en estos datos.

## Figuras de verificación sugeridas (002_EDA)

- La serie mensual de consumo 2016–2026, con 2026 marcado como preliminar.
- El número de registros de tarifas por mes, para detectar meses faltantes.
- Una gráfica de `total` contra la suma de componentes, que debe caer sobre la recta 1:1.

## Ideas de resultados (elige o propone al menos 2)

1. Cuánto costaron 250 kWh/mes en cada división entre 2017 y 2026, y qué división es más cara.
2. Qué componente explica el aumento de la tarifa: un área apilada por componente a lo largo del tiempo.
3. La estacionalidad del consumo nacional y la pregunta de si 2026 sale del rango histórico (descomposición en tendencia, estacionalidad y residuo).
4. La brecha entre DB1 y DB2: ¿cuánto se paga de más al cruzar los 150 kWh/mes?

## Ideas de innovación

- **`ipywidgets`**: un simulador interactivo con menús de división y categoría y un deslizador de kWh que muestre el desglose de la factura.
- **`statsmodels`** (`STL`) para descomponer la serie de consumo.
- **Tipado** con `Literal["DB1", "DB2", ...]` para las categorías válidas.
- **`pandera`**: un esquema que valide el formato de la tabla de tarifas antes de calcular.

## Información que debes conseguir

- La fuente oficial de los datos (CRE/CFE, SENER, datos.gob.mx) y su licencia, citadas en el `README.md`.
- La definición de cada componente tarifario, en una línea cada uno, para tu presentación.

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
