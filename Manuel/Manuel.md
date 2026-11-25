# Proyecto final: Manuel

**Tema:** Cinética de secado solar de pera: pérdida de peso, contenido de humedad y ajuste de modelos de capa delgada, comparando 4 configuraciones del secador (con o sin colectores × con o sin membrana).

## Los datos

`Prueba_pera_piña_.zip` (57 archivos). El diseño experimental es de 2 × 2, más una prueba sin carga:

| Carpeta | Colectores | Membrana | Fecha |
|---|---|---|---|
| `Corrida_1_con_colectores_sin_membrana_19_10_2026` | sí | no | 19/10 |
| `Corrida_1R_con_colectores_sin_membrana_5_11_2026` (repetición) | sí | no | 05/11 |
| `Corrida_2_sin_colectores_sin_membrana_21_10_2026` | no | no | 21/10 |
| `Corrida_3_con_colectores_con_membrana_3_11_2026` | sí | sí | 03/11 |
| `Corrida_4_sin_colectores_con_membrana_7_11_2026` | no | sí | 07/11 |
| `Sin_carga/` | — | — | 26–28/10 |

En cada corrida hay tres tipos de archivo:

1. **`LOG_*báscula*.CSV`**, uno por minuto. Tiene `Date` (dd/mm/aaaa) y `Time`, y 4 cámaras (`C1`–`C4`), cada una con modo, temperatura (consigna y medida), humedad, **peso en kg** y PWM. Los canales inactivos dicen `"OFF"`.
2. **`TH-USB-00X`**, registradores de temperatura, humedad y punto de rocío, uno por minuto. Unos vienen en `.xlsx` y otros en `.txt` (latin-1, CRLF), y el número de serie sólo aparece en la primera fila.
3. **`VEE_*.csv` / `PRUEBA_PERA_*.csv`**: temperaturas de salida de los colectores (`T_OUT_C1_1`…`T_OUT_C2_2`), temperatura ambiente y 6 piranómetros (este, norte, horizontal, inclinado, sur, oeste), más `PIRANOMETRO_INT_GH` (~2.99, que parece un voltaje y no W/m²). Las fechas vienen como `Tue 03/Nov/2026 08:47:18`.

### ⚠️ Problemas que debes resolver ANTES de analizar

Los encontré revisando el zip:

- **El archivo de báscula de la Corrida 1R es idéntico al de la Corrida 1** (mismo MD5, fechas del 19–20/10).
- **El archivo de báscula de la Corrida 2 es idéntico al de la Corrida 3** (mismo MD5, fechas del 03–04/11).
- El archivo de báscula de la Corrida 4 (carpeta del 7/11) trae fechas del **05–06/11**, que corresponden a la 1R.
- La Corrida 1 no tiene archivo VEE.
- La carpeta `Sin_carga` dice 24/10, pero los datos son del 26 al 28/10.

Busca los archivos correctos en tu equipo de adquisición y reemplázalos. Documenta en `001_ETL` qué archivo corresponde a cada corrida.

### Otros puntos de limpieza (los reviso en `001_ETL`)

- Los archivos VEE traen **líneas de encabezado repetidas a mitad del archivo**. Algunos no traen encabezado y el nombre de la columna de fecha cambia (`Fecha` o `FECHA`). Todos terminan con una coma que crea una columna vacía.
- La Corrida 3 está partida en `primera_parte` y `segunda_parte`: hay que concatenarlas.
- `na_values="OFF"`. El peso muestra la **carga** (sube de 0.27 a 1.7 kg en minutos) y la **descarga** al final (0 kg). Recorta esos tramos con un criterio explícito.
- Hay nombres de archivo con errores (`TH-USB-008__19_10_2026}.xlsx`, `ñ` y `á` en las rutas). Usa `glob` y no escribas los nombres a mano.
- Escribe **una función importadora por tipo de archivo** (báscula, TH-USB y VEE) que devuelva un DataFrame con índice de tiempo. Esa función es parte de la limpieza y no la presentas.

Este conjunto es el que más limpieza requiere. Por eso la función de análisis tiene un alcance acotado.

## La función

```python
from tools.secado import cinetica_secado

cinetica_secado(peso, humedad_inicial_bh=0.84, masa_seca=None, modelos=("Lewis", "Page", "Henderson-Pabis"))
# -> (1) DataFrame en el tiempo: X (humedad en base seca), MR (razón de humedad), dX/dt (velocidad de secado)
#    (2) tabla de modelos: parámetros, R² y RMSE de cada modelo
```

- Pasa del peso a la humedad en base seca, X = (m − m_s) / m_s, y a la razón de humedad, MR = (X − X_e) / (X₀ − X_e).
- Ajusta los modelos de capa delgada con **`scipy.optimize.curve_fit`**:
  - Lewis: MR = e^(−kt)
  - Page: MR = e^(−ktⁿ)
  - Henderson–Pabis: MR = a·e^(−kt)
- Debe servir para cualquier corrida: recibe la serie de peso ya limpia.

## Figuras de verificación sugeridas (002_EDA)

- El peso de cada corrida, con los tramos de carga y descarga marcados.
- La irradiancia en el plano horizontal contra el inclinado, que deben ser coherentes.
- Los registradores TH-USB de una corrida superpuestos.

## Ideas de resultados (elige o propone al menos 2)

1. Las curvas de MR de las 4 configuraciones en una figura, con el tiempo necesario para llegar a la humedad objetivo de cada una.
2. El efecto del colector y el de la membrana sobre la constante k de Page (una tabla 2 × 2).
3. El incremento de temperatura del colector (T_out − T_amb) contra la irradiancia. ¿Cuánto calienta por cada W/m²?
4. La repetibilidad entre la Corrida 1 y la 1R. Y si de noche hay **rehidratación**, ¿se ve en el peso?

## Ideas de innovación

- **`scipy.optimize.curve_fit`** para el ajuste no lineal (no se vio SciPy en el curso).
- **`dataclasses`** para devolver un resultado estructurado (`ResultadoSecado`) en lugar de una tupla.
- **Tipado** de las funciones.
- **`ipywidgets`** para elegir la corrida y el modelo y ver el ajuste.

## Información que debes conseguir

- La **humedad inicial** de la pera (en base húmeda) o su masa seca final (por estufa). Sin ese dato no se puede calcular X.
- La tara de la charola y la cámara de la báscula (`C1`–`C4`) que corresponde a cada configuración.
- La posición de cada registrador TH-USB.
- Si en estas corridas hubo piña o sólo pera (la carpeta dice "pera_piña", pero los archivos sólo mencionan pera).

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
