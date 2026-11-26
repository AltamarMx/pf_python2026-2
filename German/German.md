# Proyecto final: German

## Los datos

### 1. SCADA del parque eólico La Haute Borne (Engie, Francia)

Viene en el zip `SCADA dataset of the Engie La Haute Borne wind farm _onshore_ in France.zip` (~260 MB descomprimido):

- `static-information.csv`: 4 turbinas Senvion MM82 (R80711, R80721, R80736, R80790) de 2050 kW nominales, buje a 80 m, rotor de 82 m, con su posición GPS.
- `data_description.csv`: el significado y las unidades de cada variable.
- `Cleaned data/Turbine_data_<turbina>_2013-2018_F95.csv` y `..._F99.csv`: 8 archivos (2 versiones por turbina) de ~211 000 registros cada uno. Son datos cada 10 min del 2013-01-01 al 2018-01-13 con 18 columnas: `P_avg` (kW), `Q_avg`, `S_avg`, `Ws_avg` (m/s), velocidades del generador y temperaturas de la caja de engranes, el generador, la góndola y el rodamiento.

### 2. Script de la API de Open-Meteo (`Python API request Open Meteo.txt`)

Pide datos horarios históricos de 2006 a 2026: velocidad del viento a 10 y 100 m, dirección, temperatura, radiación, etc. Usa un *bounding box* sobre Morelos (18.33–19.13 N, 99.49–98.63 W).

## Objetivo

_Por definir._

---

Los lineamientos, la estructura de entrega y la rúbrica son los mismos para todos: ver [`README.md`](../README.md).
