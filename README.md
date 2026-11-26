# Proyecto final, Python 2027-1: lineamientos y rúbrica

Estos lineamientos y la rúbrica son iguales para todos. Cada persona tiene una carpeta con su `NOMBRE.md`, que resume sus datos y el objetivo de su proyecto.

Todos entregan lo mismo: una función reutilizable en `notebooks/tools/`, al menos 2 resultados propios, al menos 2 innovaciones justificadas y una presentación de 15 min.

## Estructura del espacio de trabajo (uv)

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
│   ├── 004_uso_<funcion>.ipynb # demostración de uso y resultados propios (mínimo 2)
│   └── tools/
│       ├── __init__.py
│       └── <modulo>.py
└── presentacion/               # diapositivas (pdf o html)
```

Puedes partir una etapa en varias libretas. Lo que importa es que la numeración siga el orden de lectura.

## Reglas de trabajo (las del curso)

1. Las libretas van numeradas por orden de lectura (`001_...`), sin espacios ni acentos. Cada una se entrega ejecutada de arriba abajo, en orden (*Restart & Run All*), con sus salidas.
2. Usa siempre rutas relativas (`../data/001_raw/...`), nunca rutas absolutas.
3. Los paquetes se agregan con `uv add`, nunca con `pip install` desde la libreta. Con `uv sync` debe bastar para reproducir el entorno.
4. Los datos crudos no se tocan. El ETL escribe en `data/002_intermediate/`.
5. Una libreta, una pregunta. Escribe en Markdown por qué tomaste cada decisión.

## La función: el centro del proyecto

- Vive en `notebooks/tools/<modulo>.py` y se importa con `from tools.<modulo> import <funcion>`.
- **Supone que los datos siempre tienen el formato limpio que produce `001_ETL`.** El docstring describe ese formato: índice, nombres de columnas y unidades. Si la función recibe algo distinto, debe fallar con un mensaje claro (`raise ValueError("...")`), no con un error críptico de pandas.
- Devuelve un resultado (DataFrame, Serie, diccionario…), no sólo una figura. Graficar es opcional: un argumento `grafica=True` o una función aparte.
- Lleva un docstring completo (qué hace, parámetros, qué devuelve y un ejemplo) que se pueda ver con `help()` y con `Shift+Tab`.
- Puede usar funciones auxiliares del mismo módulo. Lo importante es poder reutilizarla con **otro archivo del mismo formato**: otra corrida, otro año, otra muestra.

## Figuras

- **Figuras de verificación** (en `002_EDA`): comprueban que los datos se importaron bien y se comportan como se espera. No cuentan como resultado.
- **Figuras o tratamientos propios** (en `004_uso_<funcion>`): **al menos dos**, propuestos por ti, que den un resultado: un número, una comparación o una conclusión.

## Innovación

Incluye al menos **dos** herramientas o técnicas que no se hayan visto en el curso, **y justifica cada una**. En la presentación tienes que explicar qué problema resuelve cada una en tu proyecto y por qué conviene más que lo visto en clase. Si eliges el tipado de funciones (*type hints*), debes saber explicar qué hace y qué no hace: Python no lo verifica al ejecutar; sirve para documentar, para el autocompletado del editor y para revisar el código con herramientas como `mypy` o `pyright`.

## Presentación (15 minutos)

La presentación trata de **las funciones**, no de la limpieza; la limpieza la reviso yo en el repositorio. Distribución sugerida:

- 2 min: contexto y datos.
- 7 min: la función (qué recibe, qué devuelve, demostración).
- 3 min: resultados.
- 2 min: innovaciones y su justificación.
- 1 min: cierre.

Lleva la demostración ya ejecutada por si algo falla en vivo.

**Durante la presentación puedo intervenir para cuestionar tus funciones.** Por ejemplo, puedo pedirte que:

- expliques qué hace una línea o un bloque concreto del código;
- cambies un parámetro y anticipes qué va a pasar antes de ejecutarla;
- la llames con otro archivo del mismo formato o con datos mal formados, para ver cómo valida la entrada;
- justifiques por qué la escribiste así y no de otra forma.

Tienes que conocer tu código lo suficiente para responder sin leerlo por primera vez.

## Entrega

Un zip del espacio de trabajo **sin `.venv/`**, que se sube al Drive del curso.

Antes de zipear, borra `.venv/`.

## Rúbrica (60 % del curso, sobre 100)

| Rubro | Peso | Qué se evalúa | Dónde |
|---|---|---|---|
| ETL y limpieza | 20 | `001_ETL` documentado, decisiones justificadas en Markdown, crudos intactos, datos limpios en `002_intermediate` | Repositorio |
| Función | 25 | Corrección, docstring completo, validación del formato de entrada y reutilización con otro archivo del mismo formato | Repositorio + presentación |
| Demostración de uso | 10 | `004_uso_<funcion>` muestra con claridad qué recibe y qué devuelve la función | Repositorio + presentación |
| Resultados propios | 15 | Al menos 2 figuras o tratamientos, cada uno con una conclusión | Repositorio + presentación |
| Innovación | 10 | Dos herramientas o técnicas nuevas y la justificación verbal de cada una | Presentación |
| Reproducibilidad | 10 | `uv sync` + *Run All* desde cero, rutas relativas, nombres de archivo, zip limpio | Profesor |
| Presentación | 10 | Claridad, tiempo y respuestas a las preguntas y al cuestionamiento sobre las funciones | Presentación |
