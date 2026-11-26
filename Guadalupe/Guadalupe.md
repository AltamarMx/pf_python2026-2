# Proyecto final: Guadalupe

## Los datos

Hay dos pruebas de 24 h en la cámara climática del Laboratorio de Edificaciones Sustentables, con registro cada 30 s (~2 880 registros cada una):

| Archivo | Inicio | Columnas |
|---|---|---|
| `Datos_Prueba_2024-11-06_13-51.txt` | 06/11/2024, 01:51 p. m. | ~65 |
| `Datos_Prueba_2025-04-29_14-14.txt` | 29/04/2025, 02:14 p. m. | ~73 |

Las dos usan el mismo perfil (temperatura sol-aire, habitación sin aire acondicionado, Tmáx 27.9 °C a las 15:00, Tmín 15.5 °C, mes de enero, latitud 18.9, Id 209.2, Ib 386.1, gamma −90, absortancia 0.3).

### Formato del archivo

- Texto separado por **tabuladores**, codificación **latin-1** y fin de línea CRLF.
- Las líneas 1 y 2 son **metadatos de la prueba** en pares clave–valor. Los encabezados empiezan en la línea 3.
- `Fecha` (dd/mm/aaaa) y `Hora` en formato de 12 h con `"p. m."`/`"a. m."` (ej. `01:51:03 p. m.`).
- `Tiempo LabVIEW`: segundos desde el **1 de enero de 1904** (la época de LabVIEW).
- Grupos de sensores: cámara climática (`CC …`), habitación (`Hab …`) y guarda. Superficies en malla de 3 × 3 (`SC Sup/Medio/Inf` × `Izq/Centro/Der`), aire a varias alturas (`V1`, `V2`, `V3`) y flujos de calor (`FCC`, `FCR`).

## Objetivo

_Por definir._

---

Los lineamientos, la estructura de entrega y la rúbrica son los mismos para todos: ver [`PROYECTO_FINAL_GENERAL.md`](../PROYECTO_FINAL_GENERAL.md).
