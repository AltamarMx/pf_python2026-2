# Proyecto final: Roberto

## Los datos

`la_paz_utci_2017-2025.csv`: Índice Climático Térmico Universal (UTCI) en La Paz, BCS. Un solo archivo CSV (6.9 MB) con 78 888 registros horarios del 2017-01-01 00:00 al 2025-12-31 23:00 UTC. No falta ninguna marca de tiempo; hay 10 horas sin valor.

| Columna | Significado |
|---|---|
| `time_utc`, `time_local` | Tiempo en UTC y en hora local (UTC−7 fija) |
| `utci_c` | UTCI en °C para La Paz (~24.14 N, 110.31 W) |
| `utci_<lat>_<lon>_c` | UTCI en los 4 nodos de la malla de 0.25° que rodean La Paz |
| `utci_nodo_cercano_c` | UTCI en el nodo más cercano |
| `categoria` | Categoría de estrés térmico, en texto |

Categorías presentes: sin estrés térmico (44 689 h), calor moderado (18 267), calor fuerte (12 155), frío ligero (3 222), calor muy fuerte (541) y frío moderado (4).

## Objetivo

_Por definir._

---

Los lineamientos, la estructura de entrega y la rúbrica son los mismos para todos: ver [`PROYECTO_FINAL_GENERAL.md`](../PROYECTO_FINAL_GENERAL.md).
