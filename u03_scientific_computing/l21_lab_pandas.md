---
title: "Lab: Pandas"
unit: "3"
lesson: 21
type: lab
tags: [python, pandas, dataframe, read_csv, columnas, filas, seleccion, tabular]
difficulty: introductory
duration: "60 mins"
---

# Lab: Pandas

**Meta:** leer un CSV a un **DataFrame** con `pd.read_csv` (la receta de la L19 en una línea); inspeccionarlo (`.shape`, `.columns`, `.head`, `.describe()`); seleccionar columnas (`df['ph']`, `df[['sitio','ph']]`); y filtrar filas por una condición (`df[df['ph'] > 7.0]`, la máscara booleana de la L20). Es la misma `muestras.csv` de la L19, ahora como una tabla cómoda. Acompaña a la nota de concepto [Pandas](l21_concept_pandas.qmd).

> **Cómo usar este lab:** ejecuta cada celda con `Shift + Enter`, empezando por las dos celdas de preparación. Donde diga **Predice**, hay líneas comentadas con `#`: piensa primero qué pasaría (la del Paso 4 daría un error a propósito -- déjala comentada). Donde diga **Tu turno**, hay un bloque comentado con una parte marcada con `____` que debes **completar** al quitar los `#`. Compara con la respuesta esperada de cada bloque desplegable.
>
> [![Open in Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/devomh/comp3005-2026/blob/main/u03_scientific_computing/l21_lab_pandas.ipynb)

## Preparación

Pandas no es parte de Python estándar: hay que **instalarlo** (en Colab) e **importarlo**. Corre estas **dos** celdas primero. La primera instala Pandas (en Colab; si ya está, no hace nada). La segunda lo importa con el alias `pd`.

```python
%pip install -q pandas
```

```python
import pandas as pd

print("Pandas listo para trabajar")
```

~~~text
Pandas listo para trabajar
~~~

## Paso 1: leer un CSV a un DataFrame

Primero escribimos `muestras.csv` (la misma de la L19). Luego, donde antes recorrías el archivo a mano, `pd.read_csv` lo lee entero a un **DataFrame** en una línea:

```python
muestras = """muestra,sitio,ph,temperatura
1,rio_Norte,7.2,18.0
2,rio_Sur,6.8,21.0
3,rio_Norte,7.4,17.0
4,rio_Sur,6.5,22.0
5,rio_Norte,7.0,19.0
6,rio_Sur,6.8,20.0
"""

with open('muestras.csv', 'w') as f:
    f.write(muestras)

df = pd.read_csv('muestras.csv')
print(df)
```

~~~text
   muestra      sitio   ph  temperatura
0        1  rio_Norte  7.2         18.0
1        2    rio_Sur  6.8         21.0
2        3  rio_Norte  7.4         17.0
3        4    rio_Sur  6.5         22.0
4        5  rio_Norte  7.0         19.0
5        6    rio_Sur  6.8         20.0
~~~

`df` es la tabla completa, con los nombres de columna y un índice de filas (`0` a `5`). Una línea reemplazó todo el bucle de la L19.

## Paso 2: información del DataFrame

Métodos para ver de qué se trata la tabla:

```python
print(df.shape)
print(df.columns.tolist())
print(df.head(3))
```

~~~text
(6, 4)
['muestra', 'sitio', 'ph', 'temperatura']
   muestra      sitio   ph  temperatura
0        1  rio_Norte  7.2         18.0
1        2    rio_Sur  6.8         21.0
2        3  rio_Norte  7.4         17.0
~~~

`df.shape` da `(6, 4)`: 6 filas, 4 columnas. `df.describe()` da un resumen estadístico de las columnas numéricas:

```python
print(df.describe())
```

~~~text
        muestra        ph  temperatura
count  6.000000  6.000000     6.000000
mean   3.500000  6.950000    19.500000
std    1.870829  0.320936     1.870829
min    1.000000  6.500000    17.000000
25%    2.250000  6.800000    18.250000
50%    3.500000  6.900000    19.500000
75%    4.750000  7.150000    20.750000
max    6.000000  7.400000    22.000000
~~~

La fila `mean` de `ph` es `6.95` (la columna `muestra` es un id, así que su resumen no tiene sentido real).

## Paso 3: seleccionar columnas

Una columna se toma con su nombre; el resultado es una **Series** (un arreglo de NumPy por dentro):

```python
print(df['ph'])
print(round(df['ph'].mean(), 2))
```

~~~text
0    7.2
1    6.8
2    7.4
3    6.5
4    7.0
5    6.8
Name: ph, dtype: float64
6.95
~~~

`df['ph'].mean()` usa el `mean` de la L20: `6.95`, el mismo promedio que en la L19 sacaste con un `for`. Para **varias** columnas se le pasa una **lista** de nombres (corchetes dobles), y el resultado es un DataFrame:

```python
print(df[['sitio', 'ph']])
```

~~~text
       sitio   ph
0  rio_Norte  7.2
1    rio_Sur  6.8
2  rio_Norte  7.4
3    rio_Sur  6.5
4  rio_Norte  7.0
5    rio_Sur  6.8
~~~

## Paso 4: filtrar filas por condición (predice primero)

Para quedarte con las filas que cumplen una condición se usa la **máscara booleana** de la L20, ahora sobre el DataFrame:

```python
print(df[df['ph'] > 7.0])
```

~~~text
   muestra      sitio   ph  temperatura
0        1  rio_Norte  7.2         18.0
2        3  rio_Norte  7.4         17.0
~~~

Quedan solo las filas con `ph` mayor que 7.0: la `muestra` 1 y la 3, igual que el filtro de la L19.

**Predice:** ¿qué pasaría si pides una columna con el nombre mal escrito? La línea está comentada a propósito: piénsalo y luego mira la respuesta.

```python
# Predice: pedir una columna que no existe (mayuscula equivocada)
# print(df['PH'])

# Pista: fijate en como se llaman EXACTAMENTE las columnas
print(df.columns.tolist())
```

~~~text
['muestra', 'sitio', 'ph', 'temperatura']
~~~

<details><summary>Respuesta esperada</summary>

Si quitaras los `#`, pedir `df['PH']` (con mayúsculas) daría un error, porque la columna se llama `ph`, no `PH`:

~~~text
KeyError: 'PH'
~~~

Los nombres de columna **distinguen mayúsculas de minúsculas**. Pandas no encuentra una columna llamada `PH` y lanza un `KeyError`. (La dejamos comentada porque un error vivo detendría el resto del notebook.)
</details>

## Tu turno (programa completo)

### Ejercicio: conteos de especies con Pandas

Un muestreo de biodiversidad guardó, en `especies.csv`, las columnas `muestra,sitio,especie,conteo`. Léelo con `pd.read_csv`, calcula el **promedio** de la columna `conteo` (con `round` a 2 decimales) y **filtra** las filas con `conteo` **mayor que 10**. Completa la parte que falta.

```python
# TODO: quita los # y completa la parte que falta (el operador de comparacion, ____).
# especies = """muestra,sitio,especie,conteo
# 1,laguna_A,tilapia,12
# 2,laguna_A,guppy,8
# 3,laguna_B,tilapia,15
# 4,laguna_B,guppy,6
# 5,laguna_A,tilapia,9
# """
# with open('especies.csv', 'w') as f:
#     f.write(especies)
#
# e = pd.read_csv('especies.csv')
# print(round(e['conteo'].mean(), 2))
# print(e[e['conteo'] ____ 10])
```

<details><summary>Respuesta esperada</summary>

~~~text
10.0
   muestra     sitio  especie  conteo
0        1  laguna_A  tilapia      12
2        3  laguna_B  tilapia      15
~~~

La parte que falta es `>` (mayor que): `e[e['conteo'] > 10]`. El promedio es `round(e['conteo'].mean(), 2)` -> `(12 + 8 + 15 + 6 + 9) / 5 = 10.0`. El filtro `e['conteo'] > 10` deja pasar las filas con `conteo` mayor que 10: la `muestra` 1 (`12`) y la 3 (`15`). Es la misma selección de filas que con `muestras.csv`, sobre otra tabla.
</details>

## Resumen

- Se **importa** con `import pandas as pd`; la estructura central es el **DataFrame** (una tabla etiquetada).
- `pd.read_csv('archivo.csv')` lee un CSV a un DataFrame en **una línea** -- la receta de la L19 automatizada.
- Para ver la tabla: `df.shape` (filas, columnas), `df.columns`, `df.head(n)`, `df.describe()` (resumen).
- Una columna es `df['ph']` (una **Series**, un arreglo de NumPy): `df['ph'].mean()` usa el `mean` de la L20.
- Varias columnas: `df[['sitio', 'ph']]` (una **lista** de nombres da un DataFrame).
- Filtrar filas: `df[df['ph'] > 7.0]` -- la máscara booleana de la L20 sobre la tabla.
- **Sigue en la L22** con más Pandas: contar valores, columnas calculadas y agrupar por categoría.
