# Proyecto final, Python 2027-1: visión general (documento del profesor)

Cada carpeta tiene su `NOMBRE.md`. La segunda mitad de esos archivos es idéntica para todos (estructura, reglas, función, figuras, innovación, presentación y entrega). La primera mitad describe los datos y el proyecto de cada persona.

## Resumen y equilibrio

| Persona | Datos | Función central | Carga de limpieza | Carga de la función | Cómo se compensa |
|---|---|---|---|---|---|
| Emiliano | Tarifas CFE/CRE (70 mil filas) + consumo nacional mensual | `calcula_factura` | Media (formato largo, meses en texto, divisiones) | Media | Alcance limitado a tarifas sin periodo horario; GDMTH opcional |
| German | SCADA de 4 turbinas (~850 mil filas, 260 MB) + API de Open-Meteo | `curva_potencia` + `energia_esperada` | Media-alta (volumen, F95/F99, API) | Media | Puede desarrollar con una sola turbina |
| Guadalupe | 2 pruebas de 24 h en cámara climática (LabVIEW) | `respuesta_termica` | Alta (encabezados distintos, metadatos, hora "p. m.", época 1904) | Media | La función opera sobre nombres estándar |
| Manuel | Secador solar: 5 corridas × 3 tipos de archivo | `cinetica_secado` | **Muy alta** (formatos mezclados, encabezados repetidos, **archivos duplicados**) | Media | Las importadoras son limpieza, no se presentan |
| May | Espectros R/T de 2 muestras + filtro + 2 fotocorrientes | `banda_prohibida` + `fotorrespuesta` | Baja-media (salto del detector, T negativa) | **Alta** (dos funciones) | Lleva dos funciones |
| Ricardo | Anemometría de La Ventosa 2015, 10 min, 2 alturas | `caracteriza_viento` + `exponente_cizalla` | Media (convención de tiempo, ceros) | Media | — |
| Roberto | UTCI horario de La Paz 2017–2025, 4 nodos | `episodios_estres` | Baja (datos casi limpios) | Media-alta (detección vectorizada de episodios) | Validación de categorías y nodos; la función tiene más peso |

Todos tienen la misma plantilla de entrega: una función reutilizable en `notebooks/tools/`, al menos 2 resultados propios, al menos 1 innovación justificada y una presentación de 15 min.

## Rúbrica sugerida (60 % del curso, sobre 100)

| Rubro | Peso | Quién lo evalúa |
|---|---|---|
| ETL y limpieza documentada (`001_ETL`, decisiones justificadas, crudos intactos) | 20 | Profesor, en el repositorio |
| Función: corrección, docstring, validación del formato y reutilización con otro archivo | 25 | Repositorio + presentación |
| Demostración de uso (`004`) | 10 | Repositorio + presentación |
| Al menos 2 resultados propios (figuras o tratamientos con una conclusión) | 15 | Repositorio + presentación |
| Innovación y su justificación verbal | 10 | Presentación |
| Reproducibilidad (`uv sync` + *Run All* desde cero, rutas relativas, nombres, zip limpio) | 10 | Profesor |
| Presentación (claridad, tiempo, respuestas a preguntas) | 10 | Presentación |
