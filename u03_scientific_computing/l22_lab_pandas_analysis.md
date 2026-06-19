---
title: "Lab: Pandas -- análisis de datos"
unit: "3"
lesson: 22
type: lab
tags: [python, pandas, dataframe, value_counts, groupby, columnas_calculadas, subconjuntos, analisis, tabular]
difficulty: introductory
duration: "60 mins"
---

# Lab: Pandas -- análisis de datos

**Meta:** tomar una tabla y **analizarla** con Pandas: contar los **valores únicos** de una columna (`unique`, `value_counts`); crear una **columna calculada** a partir de otras (`df['nueva'] = ...`); recortar un **subconjunto** de filas (`df[df['col'] == valor]` + `reset_index`); y **agrupar** por una categoría para calcular por grupo (`groupby`). Usamos una tabla de **biodiversidad** (conteos de peces por laguna). Acompaña a la nota de concepto [Pandas: análisis de datos](l22_concept_pandas_analysis.qmd).

> **Cómo usar este lab:** ejecuta cada celda con `Shift + Enter`, empezando por las dos celdas de preparación. Donde diga **Predice**, piensa primero qué saldrá antes de ejecutar. Donde diga **Tu turno**, hay un bloque comentado con una parte marcada con `____` que debes **completar** al quitar los `#`. Compara con la respuesta esperada de cada bloque desplegable.
>
> [![Open in Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/devomh/comp3005-2026/blob/main/u03_scientific_computing/l22_lab_pandas_analysis.ipynb)

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

## Paso 1: valores únicos y conteos

Primero escribimos `biodiversidad.csv`: un muestreo de peces en dos lagunas. Luego lo leemos a un DataFrame, como en la L21:

```python
biodiversidad = """sitio,especie,conteo
laguna_A,tilapia,15
laguna_B,molly,5
laguna_A,guppy,10
laguna_B,tilapia,25
laguna_A,tilapia,20
laguna_B,guppy,15
laguna_B,molly,10
"""

with open('biodiversidad.csv', 'w') as f:
    f.write(biodiversidad)

df = pd.read_csv('biodiversidad.csv')
print(df)
```

~~~text
      sitio  especie  conteo
0  laguna_A  tilapia      15
1  laguna_B    molly       5
2  laguna_A    guppy      10
3  laguna_B  tilapia      25
4  laguna_A  tilapia      20
5  laguna_B    guppy      15
6  laguna_B    molly      10
~~~

Para ver qué **valores distintos** hay en una columna se usa `.unique()`; para ver **cuántas veces** aparece cada uno, `.value_counts()` (que ordena de mayor a menor):

```python
print(df['especie'].unique())
print(df['especie'].value_counts())
```

~~~text
['tilapia' 'molly' 'guppy']
especie
tilapia    3
molly      2
guppy      2
Name: count, dtype: int64
~~~

Hay tres especies; `tilapia` es la más frecuente (aparece 3 veces). Esto es lo que en la L19 hacías a mano con un `dict` de conteo: ahora es un método. Lo mismo para los sitios:

```python
print(df['sitio'].value_counts())
```

~~~text
sitio
laguna_B    4
laguna_A    3
Name: count, dtype: int64
~~~

## Paso 2: columna calculada

Se puede crear una **columna nueva** a partir de otras: la operación se aplica a la columna entera de una vez (vectorizada, como los arreglos de NumPy de la L20). Aquí calculamos la **proporción** de cada conteo sobre el total (`df['conteo'].sum()` es 100):

```python
df['proporcion'] = df['conteo'] / df['conteo'].sum()
print(df)
```

~~~text
      sitio  especie  conteo  proporcion
0  laguna_A  tilapia      15        0.15
1  laguna_B    molly       5        0.05
2  laguna_A    guppy      10        0.10
3  laguna_B  tilapia      25        0.25
4  laguna_A  tilapia      20        0.20
5  laguna_B    guppy      15        0.15
6  laguna_B    molly      10        0.10
~~~

La columna `proporcion` se agregó al DataFrame (las proporciones suman 1.0). El CSV en disco no cambia: la columna vive en el DataFrame en memoria.

## Paso 3: subconjunto de filas (predice primero)

Para quedarte con un **subconjunto** de filas se usa la máscara booleana de la L21. Tomemos solo `laguna_A`:

```python
dfA = df[df['sitio'] == 'laguna_A']
print(dfA)
```

~~~text
      sitio  especie  conteo  proporcion
0  laguna_A  tilapia      15        0.15
2  laguna_A    guppy      10        0.10
4  laguna_A  tilapia      20        0.20
~~~

**Predice:** fíjate en el índice de la izquierda: `0, 2, 4`. ¿Por qué salta? Al filtrar, Pandas **conserva las etiquetas de fila originales** (las filas de `laguna_A` eran la 0, la 2 y la 4 en la tabla completa). Para **renumerar** el índice desde 0 se usa `reset_index(drop=True)`:

```python
print(dfA.reset_index(drop=True))
```

~~~text
      sitio  especie  conteo  proporcion
0  laguna_A  tilapia      15        0.15
1  laguna_A    guppy      10        0.10
2  laguna_A  tilapia      20        0.20
~~~

Ahora el índice es `0, 1, 2`. El `drop=True` evita que el índice viejo se guarde como una columna nueva.

## Paso 4: agrupaciones

Aquí está el verbo más potente. `groupby('columna')` divide las filas en **grupos** según esa columna, y luego se aplica un **agregado** a cada grupo. ¿Cuántos individuos acumula cada laguna? Agrupamos por `sitio` y sumamos `conteo`:

```python
print(df.groupby('sitio')['conteo'].sum())
```

~~~text
sitio
laguna_A    45
laguna_B    55
Name: conteo, dtype: int64
~~~

`laguna_B` acumula más (55 vs. 45). ¿Y el conteo **promedio** de cada especie? Agrupamos por `especie` y promediamos:

```python
print(df.groupby('especie')['conteo'].mean())
```

~~~text
especie
guppy      12.5
molly       7.5
tilapia    20.0
Name: conteo, dtype: float64
~~~

`tilapia` tiene el promedio más alto (20.0). Fíjate en el orden: `value_counts` ordena por **frecuencia** (de mayor a menor); `groupby` ordena por la **clave del grupo** (aquí, alfabética). En una línea, `groupby` hizo el "para cada grupo, calcula..." que a mano sería un bucle con un `dict`.

## Tu turno (programa completo)

### Ejercicio: cinética de reacción con Pandas

Un laboratorio midió la **velocidad** de varias reacciones según el `catalizador` usado, en `reacciones.csv`. Léelo, cuenta cuántas reacciones hubo por catalizador (`value_counts`) y calcula la **velocidad promedio por catalizador** (`groupby` + `mean`). Completa la parte que falta.

```python
# TODO: quita los # y completa la parte que falta (la columna por la que se agrupa, ____).
# reacciones = """reaccion,catalizador,velocidad
# R1,ninguno,2.0
# R2,enzima,9.0
# R3,enzima,11.0
# R4,ninguno,4.0
# R5,metal,6.0
# """
# with open('reacciones.csv', 'w') as f:
#     f.write(reacciones)
#
# r = pd.read_csv('reacciones.csv')
# print(r['catalizador'].value_counts())
# print(r.groupby(____)['velocidad'].mean())
```

<details><summary>Respuesta esperada</summary>

La parte que falta es `'catalizador'` (entre comillas, el nombre de la columna por la que se agrupa): `r.groupby('catalizador')['velocidad'].mean()`.

~~~text
catalizador
ninguno    2
enzima     2
metal      1
Name: count, dtype: int64
catalizador
enzima     10.0
metal       6.0
ninguno     3.0
Name: velocidad, dtype: float64
~~~

`value_counts` cuenta las reacciones por catalizador (`ninguno` y `enzima` con 2, `metal` con 1). `groupby('catalizador')['velocidad'].mean()` da la velocidad promedio de cada grupo: la `enzima` es la más rápida (10.0), seguida del `metal` (6.0) y de `ninguno` (3.0). Es la misma idea que con la tabla de biodiversidad, sobre otra tabla.
</details>

## Resumen

- **Valores únicos y conteos:** `df['col'].unique()` (los valores distintos) y `df['col'].value_counts()` (cuántas veces aparece cada uno, ordenado de mayor a menor) -- el conteo con `dict` de la L19, ahora un método.
- **Columna calculada:** `df['nueva'] = <expresión sobre columnas>` -- opera la columna entera de una vez (vectorizado, L20).
- **Subconjunto:** `df[df['col'] == valor]` recorta filas (la máscara booleana de la L21); `reset_index(drop=True)` renumera el índice salteado.
- **Agrupación:** `df.groupby('col')['otra'].mean()` (o `.sum()`, `.count()`) calcula un agregado **por cada grupo** -- el verbo central del análisis.
- `value_counts` ordena por **frecuencia**; `groupby` ordena por la **clave del grupo**.
- **Sigue en la L23:** graficar estos conteos y promedios (líneas, barras, dispersión, histograma) con matplotlib.
