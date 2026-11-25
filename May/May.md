# Proyecto final: May

**Tema:** Caracterización optoelectrónica de películas delgadas de Sb₂(S,Se)₃: banda prohibida a partir de los espectros de reflectancia y transmitancia (gráfica de Tauc) y la respuesta de fotoconductividad, con y sin filtro.

## Los datos

| Archivo | Contenido | Formato |
|---|---|---|
| `SbSSeTP.txt`, `SbSSeTV.txt` | Transmitancia T % de las muestras **P** y **V** | CSV, 2 líneas de encabezado, de 250 a 2500 nm cada 2 nm (1 126 puntos) |
| `SbSSeRP.txt`, `SbSSeRV.txt` | Reflectancia R % de las mismas muestras | Igual |
| `FiltroET.txt` | Transmitancia del filtro | Igual: es un **pasabanda centrado en ~650 nm** (T máx 34.8 % a 650 nm) |
| `clase sem fotoconductividad (1)` | Corriente contra tiempo **sin filtro** | Sin extensión, 2 columnas separadas por espacios en notación científica (`1.0E+000  7.86E-011`), 60 s |
| `clase sem fotocoonductividad  con flitro` | Corriente contra tiempo **con filtro** | Igual |

En las fotocorrientes se distinguen tres tramos: oscuridad (0–20 s), iluminación (21–40 s) y oscuridad de nuevo (41–60 s).

### Puntos a resolver en la limpieza (los reviso en `001_ETL`)

- **Hay un salto en la reflectancia cerca de 830 nm** (R de la muestra P baja de ~71 % a ~67 % de un punto a otro). Es el **cambio de detector o de lámpara del espectrofotómetro**. Documéntalo y decide si lo corriges (empalme por desplazamiento) o si excluyes la región.
- Hay valores de T % **ligeramente negativos** (ruido de ~−0.005) en la región donde la película absorbe. Decide qué hacer con ellos antes de tomar logaritmos.
- Los archivos de fotoconductividad no tienen extensión, tienen nombres con espacios y errores ("fotocoonductividad", "flitro"). En `001_raw` se quedan así. Usa `sep=r"\s+"` y `header=None`, y asigna nombres a las columnas.
- Une R y T de cada muestra en un DataFrame con índice de longitud de onda y agrega la energía del fotón, E = 1240 / λ (eV).

## La función

```python
from tools.optica import banda_prohibida, fotorrespuesta

banda_prohibida(espectro, tipo="directa", rango_ev=(1.3, 1.6), espesor_nm=None)
# -> dict: Eg (eV), pendiente, R² del ajuste lineal y el DataFrame con α y (αhν)^n

fotorrespuesta(corriente, t_encendido=20, t_apagado=40)
# -> dict: I_oscuridad, I_luz, fotosensibilidad S = (I_luz − I_osc) / I_osc,
#    tiempos de subida y de caída (10–90 %)
```

- El coeficiente de absorción es α = (1/d)·ln[(1 − R)² / T].
- La gráfica de Tauc usa (αhν)^n con n = 2 (transición directa) o n = 1/2 (indirecta). **Eg es la intersección del ajuste lineal con el eje de energía**, y **no depende del espesor d**. Si no tienes d, demuéstralo y úsalo a tu favor.
- Las dos funciones deben servir para cualquier muestra o medición con el mismo formato.

> Tus datos son pocos y están limpios. Por eso tu proyecto lleva **dos funciones** (óptica y eléctrica): así queda equivalente en carga a los demás.

## Figuras de verificación sugeridas (002_EDA)

- R, T y 1 − R − T (la absorbancia aparente) contra λ para cada muestra, con el salto de 830 nm marcado.
- La transmitancia del filtro superpuesta a la T de las muestras.
- Las dos fotocorrientes en escala logarítmica, con los tramos de luz y oscuridad sombreados.

## Ideas de resultados (elige o propone al menos 2)

1. Eg de las muestras P y V, directa e indirecta, y cuánto cambia Eg según el rango de ajuste (sensibilidad).
2. ¿Los fotones del filtro (~650 nm ≈ 1.91 eV) tienen energía mayor que Eg? Relaciónalo con la fotorrespuesta con filtro frente a sin filtro.
3. La fotosensibilidad y las constantes de tiempo, con y sin filtro.
4. La energía de Urbach (cola de absorción, ln α contra E) como indicador del desorden de la película.

## Ideas de innovación

- **`ipywidgets`**: un deslizador de rango que mueva la región de ajuste lineal sobre la gráfica de Tauc y actualice Eg en vivo. Así se ve que elegir el rango es una decisión del analista.
- **`scipy.stats.linregress`** o `numpy.polyfit` con incertidumbre de la pendiente para obtener Eg ± δEg.
- **`pint`** para las unidades (nm, eV, A).
- **Tipado** de las funciones.

## Información que debes conseguir

- Qué significan **P** y **V** (¿sustrato?, ¿tratamiento?, ¿condición de depósito?).
- El voltaje de polarización y la geometría de los electrodos de la fotoconductividad (para calcular conductividad, si quieres hacerlo).
- El espesor de las películas, si lo tienes (es opcional; ver la nota sobre la gráfica de Tauc).
- El modelo del espectrofotómetro, para confirmar en qué longitud de onda cambia de detector.

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
