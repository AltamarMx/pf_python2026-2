# Proyecto final: Guadalupe

**Tema:** Respuesta térmica de un sistema constructivo en la cámara climática del Laboratorio de Edificaciones Sustentables: retraso térmico, factor de decremento y flujo de calor, comparando dos pruebas de 24 h.

## Los datos

Hay dos pruebas de 24 h con registro cada 30 s (~2 880 registros cada una):

| Archivo | Inicio | Columnas |
|---|---|---|
| `Datos_Prueba_2024-11-06_13-51.txt` | 06/11/2024, 01:51 p. m. | ~65 |
| `Datos_Prueba_2025-04-29_14-14.txt` | 29/04/2025, 02:14 p. m. | ~73 |

Las dos usan el mismo perfil (temperatura sol-aire, habitación sin aire acondicionado, Tmáx 27.9 °C a las 15:00, Tmín 15.5 °C, mes de enero, latitud 18.9, Id 209.2, Ib 386.1, gamma −90, absortancia 0.3).

### Formato del archivo

- Texto separado por **tabuladores**, codificación **latin-1** (`encoding="latin1"`) y fin de línea CRLF.
- Las líneas 1 y 2 son **metadatos de la prueba** en pares clave–valor. Los encabezados empiezan en la línea 3.
- `Fecha` (dd/mm/aaaa) y `Hora` en formato de 12 h con `"p. m."`/`"a. m."` (ej. `01:51:03 p. m.`).
- `Tiempo LabVIEW`: segundos desde el **1 de enero de 1904** (la época de LabVIEW). Te sirve para construir y verificar el índice de tiempo.
- Grupos de sensores: cámara climática (`CC …`), habitación (`Hab …`) y guarda. Superficies en malla de 3 × 3 (`SC Sup/Medio/Inf` × `Izq/Centro/Der`), aire a varias alturas (`V1`, `V2`, `V3`) y flujos de calor (`FCC`, `FCR`).

### Puntos a resolver en la limpieza (los reviso en `001_ETL`)

- **Las columnas de las dos pruebas no coinciden**: la de 2025 agrega `Hab_Tech_FCC`, `Guarda T5`, `CC tp_SFC Sólido/Hueco`, una columna **sin nombre** y otro orden. Construye un diccionario que lleve ambas a **nombres estándar** (`snake_case`, sin acentos). La función trabaja sólo con esos nombres.
- Lee los metadatos de las líneas 1–2 y guárdalos (por ejemplo, en un diccionario o un JSON en `002_intermediate`).
- Hay columnas que sólo tienen NaN (`Prom. SC Climática`) y canales con un valor casi constante de **~1103.46**, que parece un canal desconectado. Identifícalos y decide qué hacer con ellos.
- Comprueba que los 30 s entre registros sean constantes y que la hora de 12 h se convierta bien después de mediodía y de medianoche.

## La función

```python
from tools.termica import respuesta_termica

respuesta_termica(prueba, t_exterior="cc_sc_medio_centro", t_interior="hab_sc_medio_centro",
                  flujo="hab_fcc")
# -> dict o Serie: retraso térmico (h), factor de decremento (amplitud interior / exterior),
#    T máx y mín de cada lado con su hora, y energía que cruza el muro (integral del flujo, Wh/m²)
```

- El **retraso térmico** puede calcularse como la diferencia entre las horas de los máximos o con correlación cruzada (`scipy.signal.correlate`). Compara ambos métodos.
- Debe funcionar igual con cualquiera de las dos pruebas (o una tercera futura) una vez estandarizadas.

## Figuras de verificación sugeridas (002_EDA)

- La temperatura objetivo contra la medida en la cámara climática: ¿sigue el perfil sol-aire?
- Las 9 temperaturas superficiales de un lado: ninguna debe ser absurda.
- Los flujos de calor en el tiempo, con los canales sospechosos marcados.

## Ideas de resultados (elige o propone al menos 2)

1. Comparar las dos pruebas: retraso y decremento de cada una. ¿Qué sistema amortigua mejor?
2. Un **mapa 3 × 3** de las temperaturas superficiales en un instante (`imshow`): ¿es uniforme la superficie del muro? ¿Dónde están los puentes térmicos?
3. La estratificación del aire de la habitación (`V1`, `V2`, `V3`) a lo largo del día.
4. El flujo de calor contra la diferencia de temperaturas superficiales, y una conductancia aparente.

## Ideas de innovación

- **`ipywidgets`**: un deslizador de tiempo que recorra el mapa 3 × 3 de temperaturas superficiales.
- **`matplotlib.animation`**: exportar ese mapa como GIF o MP4 para la presentación.
- **`scipy.signal`** para el retraso por correlación cruzada.
- **`pint`** para manejar las unidades (W/m², Wh/m², K).
- **Tipado** de las funciones.

## Información que debes conseguir

- Un **diagrama de la ubicación de los sensores** y la descripción del sistema constructivo de cada prueba.
- Las unidades y constantes de calibración de los medidores de flujo (`FCC`, `FCR`). ¿Están en W/m² o en mV?
- El significado de los nombres que no son evidentes (`MC`, `P1`, `MP2`, `M3`, `T5`, `tp_SFC`).

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
