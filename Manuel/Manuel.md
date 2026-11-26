# Proyecto final: Manuel

## Los datos

`Prueba_pera_piña_.zip` (57 archivos): pruebas de secado solar de pera. El diseño experimental es de 2 × 2 (con o sin colectores × con o sin membrana), más una prueba sin carga:

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
3. **`VEE_*.csv` / `PRUEBA_PERA_*.csv`**: temperaturas de salida de los colectores (`T_OUT_C1_1`…`T_OUT_C2_2`), temperatura ambiente, 6 piranómetros (este, norte, horizontal, inclinado, sur, oeste) y `PIRANOMETRO_INT_GH`. Las fechas vienen como `Tue 03/Nov/2026 08:47:18`.

## Objetivo

_Por definir._

---

Los lineamientos, la estructura de entrega y la rúbrica son los mismos para todos: ver [`README.md`](../README.md).
