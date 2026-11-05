---
title: "Lab: Pandas -- análisis de datos"
unit: "3"
lesson: 22
type: lab
tags: [python, pandas, dataframe, info, value_counts, groupby, sort_values, crosstab, pivot_table, loc, analisis, penguins]
difficulty: introductory
duration: "90 mins"
---

# Lab: Pandas -- análisis de datos

**Meta:** pasar del **primer vistazo** de la L21 al **análisis**, con la misma tabla real de 344 pingüinos. Conocerás la tabla con `df.info()` y los **tipos de variable** de sus columnas; **contarás** valores (`unique`, `value_counts`); crearás una **columna calculada**; **filtrarás** con varias condiciones a la vez (`&`, `|`) y con `.loc`; **ordenarás** (`sort_values`); **agruparás** por una y por dos columnas (`groupby`); y armarás **tablas de dos entradas** (`pd.crosstab`, `pivot_table`). Al final usarás todo eso para responder una pregunta abierta: **¿hay dimorfismo sexual en los pingüinos?** Acompaña a la nota de concepto [Pandas: análisis de datos](l22_concept_pandas_analysis.qmd).

> **Cómo usar este lab:** ejecuta cada celda con `Shift + Enter`, empezando por las dos celdas de preparación. Este lab **necesita internet** (Colab ya lo tiene): la segunda celda lee los datos desde una URL. Donde diga **Predice**, piensa primero qué saldrá y después ejecuta la celda; la respuesta está en el bloque desplegable que le sigue (en el Paso 4 hay dos líneas que darían un error a propósito, por eso están comentadas con `#` -- déjalas así). La **Práctica** del Paso 5 trae un bloque comentado con partes marcadas con `____` que debes **completar** al quitar los `#`. En **Tu turno**, la Parte A trae un bloque comentado con partes marcadas con `____` que debes **completar** al quitar los `#`; la Parte B trae una lista de tareas y el programa lo escribes tú; en la Parte C escribes tu conclusión. Compara con las respuestas de los bloques desplegables.
>
> [![Open in Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/devomh/comp3005-2026/blob/main/u03_scientific_computing/l22_lab_pandas_analysis.ipynb)

## Preparación

Pandas no es parte de Python estándar: hay que **instalarlo** (en Colab ya está, así que la primera celda no hace nada) e **importarlo**. La segunda celda importa Pandas y lee la tabla de pingüinos desde la misma URL de la L21.

```python
%pip install -q pandas
```

```python
import pandas as pd

url = ('https://raw.githubusercontent.com/mwaskom/'
       'seaborn-data/master/penguins.csv')
df = pd.read_csv(url)
print("Pandas listo; tabla de pinguinos:", df.shape)
```

~~~text
Pandas listo; tabla de pinguinos: (344, 7)
~~~

*(Fuente: Palmer Penguins -- Gorman, Williams y Fraser, 2014; paquete palmerpenguins de A. Horst et al., licencia CC0.)*

## Paso 1: conocer la tabla (predice primero)

En la L21 miraste la tabla con `.shape`, `.dtypes` y `df.isna().sum()`. `df.info()` lo reúne todo en **una** llamada: cuántas filas hay, cada columna con su número de valores **no nulos** (los que no son `NaN`) y su tipo. `info()` imprime por sí mismo, así que no lleva `print`:

```python
df.info()
```

~~~text
<class 'pandas.core.frame.DataFrame'>
RangeIndex: 344 entries, 0 to 343
Data columns (total 7 columns):
 #   Column             Non-Null Count  Dtype  
---  ------             --------------  -----  
 0   species            344 non-null    object 
 1   island             344 non-null    object 
 2   bill_length_mm     342 non-null    float64
 3   bill_depth_mm      342 non-null    float64
 4   flipper_length_mm  342 non-null    float64
 5   body_mass_g        342 non-null    float64
 6   sex                333 non-null    object 
dtypes: float64(4), object(3)
memory usage: 18.9+ KB
~~~

344 filas (`0 to 343`). Las cuatro mediciones tienen 342 valores (faltan 2 en cada una) y `sex` tiene 333 (faltan 11): los mismos `NaN` que contaste en la L21 con `df.isna().sum()`, ahora de un vistazo. Esto es lo que mide cada columna:

| Columna | Qué mide | Unidad | Tipo de variable |
|---|---|---|---|
| `species` | la especie: Adelie, Chinstrap o Gentoo | -- | categórica |
| `island` | la isla donde se midió: Biscoe, Dream o Torgersen | -- | categórica |
| `bill_length_mm` | el largo del pico | mm | numérica |
| `bill_depth_mm` | el alto (la profundidad) del pico | mm | numérica |
| `flipper_length_mm` | el largo de la aleta | mm | numérica |
| `body_mass_g` | la masa corporal | g | numérica |
| `sex` | el sexo: `MALE` (macho) o `FEMALE` (hembra) | -- | categórica |

**Tipos de variable.** Una columna **categórica** dice a qué **grupo** pertenece cada fila (una especie, una isla, un sexo): se **cuenta** cuántos hay de cada uno y se usa para **agrupar**. Una **numérica** es una **cantidad** medida o contada: con ella se calculan promedios, mínimos y máximos, y se ordena. Hay un tercer tipo, la **ordinal**: categorías que tienen un **orden** natural, como la `quality` del vino de la L21 (una nota de 0 a 10) o las tallas S, M y L. En esta tabla no hay ninguna ordinal. El tipo de variable decide qué herramienta tiene sentido, y no es lo mismo que el `Dtype`: el `Dtype` dice **cómo se guarda** una columna; el tipo de variable, **qué significa**.

**Predice:** todas las masas de la tabla son gramos **enteros** (`3750`, `3800`, `3250`, ...). ¿Por qué `info()` dice que `body_mass_g` es `float64` (números con decimales) y no un entero? Pista: mira su `Non-Null Count`.

<details><summary>Respuesta esperada</summary>

Por los datos que faltan. `NaN` es, para Pandas, un valor **decimal** (`float`); los valores de una columna se guardan en un arreglo de NumPy (L20), y un arreglo tiene un solo tipo para todos sus valores. Como `body_mass_g` tiene 2 `NaN`, la columna entera se guarda como `float64`, y por eso las masas se ven como `3750.0`. Es un buen ejemplo de que el `Dtype` (cómo se guarda) no es el tipo de variable (qué significa): la masa es la misma cantidad numérica con o sin el `.0`.
</details>

## Paso 2: contar valores

Para una columna **categórica**, las dos primeras preguntas son: ¿qué valores **distintos** tiene? (`.unique()`) y ¿**cuántas veces** aparece cada uno? (`.value_counts()`, ordenado de mayor a menor):

```python
print(df['species'].unique())
print(df['species'].value_counts())
```

~~~text
['Adelie' 'Chinstrap' 'Gentoo']
species
Adelie       152
Gentoo       124
Chinstrap     68
Name: count, dtype: int64
~~~

Tres especies; la más frecuente es Adelie (152 pingüinos). Es lo que en la L19 hacías a mano con un `dict` de conteo, ahora en un método. Lo mismo para las islas:

```python
print(df['island'].value_counts())
```

~~~text
island
Biscoe       168
Dream        124
Torgersen     52
Name: count, dtype: int64
~~~

Si en vez de **cuántos** quieres **qué parte del total**, `normalize=True` da la **proporción** de cada valor (aquí redondeada a 3 decimales con `.round(3)`):

```python
print(df['species'].value_counts(normalize=True).round(3))
```

~~~text
species
Adelie       0.442
Gentoo       0.360
Chinstrap    0.198
Name: proportion, dtype: float64
~~~

El 44.2 % de los pingüinos son Adelie. Las proporciones suman 1.

**Predice:** hay 344 pingüinos. Si cuentas la columna `sex`, ¿los conteos sumarán 344?

```python
print(df['sex'].unique())
print(df['sex'].value_counts())
```

<details><summary>Respuesta esperada</summary>

~~~text
['MALE' 'FEMALE' nan]
sex
MALE      168
FEMALE    165
Name: count, dtype: int64
~~~

No: `168 + 165 = 333`. `unique()` muestra que también hay `nan` (los 11 pingüinos sin sexo registrado), pero `value_counts()` **no cuenta los `NaN`** por defecto. Para verlos, `dropna=False`:

```python
print(df['sex'].value_counts(dropna=False))
```

~~~text
sex
MALE      168
FEMALE    165
NaN        11
Name: count, dtype: int64
~~~

Ahora sí: `168 + 165 + 11 = 344`.
</details>

## Paso 3: una columna calculada

Una **columna nueva** se calcula a partir de otras, operando la columna **entera** de una vez (vectorizado, como los arreglos de NumPy de la L20). Pasemos la masa a **libras** (1 lb = 453.592 g):

```python
df['masa_lb'] = df['body_mass_g'] / 453.592
print(df[['species', 'body_mass_g', 'masa_lb']].head().round(2))
```

~~~text
  species  body_mass_g  masa_lb
0  Adelie       3750.0     8.27
1  Adelie       3800.0     8.38
2  Adelie       3250.0     7.17
3  Adelie          NaN      NaN
4  Adelie       3450.0     7.61
~~~

Un pingüino Adelie pesa unas 8 libras. Fíjate en la fila `3`: si falta la masa, falta también la masa en libras (un `NaN` dividido entre un número sigue siendo `NaN`). La columna `masa_lb` vive en el DataFrame **en memoria**: el CSV de internet no cambia (para guardarla tendrías el `to_csv` de la L21).

## Paso 4: filtrar con varias condiciones (predice primero)

En la L21 filtraste por una condición **numérica** (`df['body_mass_g'] > 6000`). Con **texto** es igual, usando `==` y el valor entre comillas:

```python
adelie = df[df['species'] == 'Adelie']
print(adelie.shape)
```

~~~text
(152, 8)
~~~

Las 152 filas de Adelie (y 8 columnas: las 7 originales más `masa_lb`). Para pedir **dos condiciones a la vez** se combinan con `&` ("y": deben cumplirse las dos), poniendo **cada condición entre paréntesis**:

```python
adelie_dream = df[(df['species'] == 'Adelie') &
                  (df['island'] == 'Dream')]
print(adelie_dream.shape)
print(adelie_dream[['species', 'island', 'body_mass_g']].head(3))
```

~~~text
(56, 8)
   species island  body_mass_g
30  Adelie  Dream       3250.0
31  Adelie  Dream       3900.0
32  Adelie  Dream       3300.0
~~~

56 pingüinos son Adelie **y** viven en Dream. Con `|` ("o": basta con que se cumpla **una**) se juntan grupos. Los pingüinos de Dream **o** de Torgersen:

```python
dos_islas = df[(df['island'] == 'Dream') |
               (df['island'] == 'Torgersen')]
print(dos_islas.shape)
```

~~~text
(176, 8)
~~~

`124 + 52 = 176`, como en el `value_counts` de las islas del Paso 2.

**Predice:** en el Módulo 1 combinabas condiciones con `and`. ¿Qué pasa si aquí escribes `and`, o si olvidas los paréntesis? Las dos líneas están comentadas a propósito: piénsalo y luego mira la respuesta.

```python
# Predice: dos formas equivocadas de combinar condiciones
# df[df['species'] == 'Adelie' & df['island'] == 'Dream']
# df[(df['species'] == 'Adelie') and (df['island'] == 'Dream')]
```

<details><summary>Respuesta esperada</summary>

Las dos darían un error:

- **Sin paréntesis**, Python calcula primero `'Adelie' & df['island']` (el `&` se evalúa **antes** que el `==`), y no sabe hacer un "y" entre un texto y una columna: "TypeError: Cannot perform 'rand_' with a dtyped [object] array and scalar of type [bool]".
- **Con `and`**, Python le pide a cada columna **un solo** `True` o `False`, pero una columna de 344 valores no es "verdadera" ni "falsa": "ValueError: The truth value of a Series is ambiguous".

La regla: en un filtro de Pandas se usan `&` y `|` (no `and` y `or`), y **cada condición va entre paréntesis**. (Las dejamos comentadas porque un error vivo detendría el resto del notebook.)
</details>

Mira el índice de `adelie_dream`: `30, 31, 32`. Como viste en la L21, al filtrar cada fila **conserva su etiqueta** de la tabla completa. Para **renumerarlas** desde 0 se usa `reset_index(drop=True)` (el `drop=True` descarta las etiquetas viejas; sin él, Pandas las guardaría como una columna nueva):

```python
adelie_dream = adelie_dream.reset_index(drop=True)
print(adelie_dream[['species', 'island', 'body_mass_g']].head(3))
```

~~~text
  species island  body_mass_g
0  Adelie  Dream       3250.0
1  Adelie  Dream       3900.0
2  Adelie  Dream       3300.0
~~~

**Filas y columnas en un paso: `.loc`.** Hasta ahora filtrabas filas con `df[...]` y después elegías columnas con `[[...]]`. `.loc` hace las dos cosas a la vez: `df.loc[filas, columnas]`, con la condición para las filas y la lista de columnas separadas por una coma. Los Chinstrap de más de 4500 g:

```python
pesados = df.loc[(df['species'] == 'Chinstrap') &
                 (df['body_mass_g'] > 4500),
                 ['island', 'sex', 'body_mass_g']]
print(pesados)
```

~~~text
    island   sex  body_mass_g
181  Dream  MALE       4550.0
189  Dream  MALE       4800.0
~~~

Solo 2 Chinstrap pasan de 4500 g, y los dos son machos. *(Un aviso para tus propios programas: si guardas un subconjunto y luego le **agregas una columna**, Pandas puede mostrar un aviso `SettingWithCopyWarning`. Para evitarlo, guarda una copia independiente con `.copy()`: `gentoo = df[df['species'] == 'Gentoo'].copy()`.)*

## Paso 5: ordenar

`sort_values('columna')` ordena las filas por una columna, de menor a mayor; con `ascending=False`, de mayor a menor. ¿Qué pingüinos tienen el pico **más largo**?

```python
picos = df.sort_values('bill_length_mm', ascending=False)
print(picos[['species', 'sex', 'bill_length_mm']].head())
```

~~~text
       species     sex  bill_length_mm
253     Gentoo    MALE            59.6
169  Chinstrap  FEMALE            58.0
321     Gentoo    MALE            55.9
215  Chinstrap    MALE            55.8
335     Gentoo    MALE            55.1
~~~

El pico más largo (59.6 mm) es de un Gentoo macho, pero el segundo es de una **hembra** Chinstrap. Las filas conservan sus etiquetas (`253`, `169`, ...): ordenar las cambia de **lugar**, no de nombre. Los `NaN` quedan al final.

### Práctica: filtrar, elegir columnas y ordenar

Junta lo de los Pasos 4 y 5: busca las **hembras Gentoo de más de 5000 g**, quédate con sus columnas `island`, `bill_length_mm` y `body_mass_g`, y ordénalas por el largo del pico, de mayor a menor. Completa las tres partes marcadas con `____`.

```python
# TODO: quita los # y completa las partes que faltan (____).
# hembras = df.loc[(df['species'] == ____) &
#                  (df['sex'] == 'FEMALE') &
#                  (df['body_mass_g'] > ____),
#                  ['island', 'bill_length_mm', 'body_mass_g']]
# print(hembras.shape)
# print(hembras.sort_values(____, ascending=False))
```

<details><summary>Respuesta esperada</summary>

Las partes que faltan son `'Gentoo'`, `5000` y `'bill_length_mm'`.

~~~text
(5, 3)
     island  bill_length_mm  body_mass_g
254  Biscoe            49.1       5150.0
293  Biscoe            46.5       5200.0
342  Biscoe            45.2       5200.0
252  Biscoe            45.1       5050.0
268  Biscoe            44.9       5100.0
~~~

Solo 5 hembras Gentoo pasan de 5000 g, todas de Biscoe (la única isla de los Gentoo). Tres condiciones unidas con `&`, cada una entre paréntesis, y `.loc` para elegir las columnas en el mismo paso. Fíjate en que la primera fila (5150 g) no es la más pesada: el orden lo da el **pico** (49.1, 46.5, ...), no la masa.
</details>

## Paso 6: agrupar (predice primero)

`groupby('columna')` divide las filas en **grupos** según una columna **categórica** y aplica un **agregado** a cada grupo. La masa promedio de cada especie:

```python
print(df.groupby('species')['body_mass_g'].mean().round(1))
```

~~~text
species
Adelie       3700.7
Chinstrap    3733.1
Gentoo       5076.0
Name: body_mass_g, dtype: float64
~~~

`groupby` ordena por la **clave del grupo** (aquí, alfabética). Para responder "¿cuál pesa más?", ordena el resultado con el `sort_values` del Paso 5 (en una Series no hace falta decir la columna):

```python
masa = df.groupby('species')['body_mass_g'].mean().round(1)
print(masa.sort_values(ascending=False))
```

~~~text
species
Gentoo       5076.0
Chinstrap    3733.1
Adelie       3700.7
Name: body_mass_g, dtype: float64
~~~

Los Gentoo son, por mucho, los más pesados; Adelie y Chinstrap pesan casi lo mismo.

**Predice:** `.count()` cuenta los valores de cada grupo. Compáralo con el `value_counts()` de especies del Paso 2 (`Adelie 152`, `Gentoo 124`, `Chinstrap 68`). ¿Saldrán los mismos números?

```python
print(df.groupby('species')['body_mass_g'].count())
```

<details><summary>Respuesta esperada</summary>

~~~text
species
Adelie       151
Chinstrap     68
Gentoo       123
Name: body_mass_g, dtype: int64
~~~

Dos diferencias. El **orden**: `value_counts` ordena por **frecuencia** (de mayor a menor) y `groupby` por la **clave** (alfabética). Y los **números**: `.count()` cuenta los valores **presentes** de `body_mass_g`, y falta una masa de un Adelie y una de un Gentoo (los 2 `NaN` del Paso 1). `value_counts` cuenta **filas** de `species`, que no tiene huecos.
</details>

**Agrupar por dos columnas.** Con una **lista** de columnas, `groupby` forma un grupo por cada **combinación**. La masa promedio por especie **y** por isla:

```python
por_isla = df.groupby(['species', 'island'])['body_mass_g'].mean()
por_isla = por_isla.round(1)
print(por_isla)
```

~~~text
species    island   
Adelie     Biscoe       3709.7
           Dream        3688.4
           Torgersen    3706.4
Chinstrap  Dream        3733.1
Gentoo     Biscoe       5076.0
Name: body_mass_g, dtype: float64
~~~

El resultado tiene un índice de **dos niveles** (especie, isla). Solo aparecen 5 combinaciones: los Chinstrap viven solo en Dream y los Gentoo solo en Biscoe. Para leer un grupo se usa `.loc` con la etiqueta, como en una tabla:

```python
print(por_isla.loc['Adelie'])
```

~~~text
island
Biscoe       3709.7
Dream        3688.4
Torgersen    3706.4
Name: body_mass_g, dtype: float64
~~~

Los Adelie pesan casi lo mismo en las tres islas. `.size()` cuenta las **filas** de cada combinación (con sus `NaN` incluidos):

```python
print(df.groupby(['species', 'island']).size())
```

~~~text
species    island   
Adelie     Biscoe        44
           Dream         56
           Torgersen     52
Chinstrap  Dream         68
Gentoo     Biscoe       124
dtype: int64
~~~

## Paso 7: tablas de dos entradas

El último `groupby` da los conteos en una lista larga. `pd.crosstab` los acomoda en una **tabla de dos entradas**: una categórica en las filas, otra en las columnas, y en cada casilla **cuántas filas** hay de esa combinación. Es un `value_counts` de **dos** columnas:

```python
print(pd.crosstab(df['species'], df['island']))
```

~~~text
island     Biscoe  Dream  Torgersen
species                            
Adelie         44     56         52
Chinstrap       0     68          0
Gentoo        124      0          0
~~~

Los ceros saltan a la vista: Chinstrap solo en Dream, Gentoo solo en Biscoe. Por eso, comparar **islas** es, en buena parte, comparar **especies**: Biscoe tiene los pingüinos más pesados (4716 g en promedio, contra unos 3710 g en Dream y Torgersen) porque ahí viven los Gentoo, no por la isla (los Adelie, que están en las tres, pesan casi igual en todas).

Si en las casillas quieres un **agregado** de una columna numérica (un promedio, en vez de un conteo), se usa `pivot_table`: es un `groupby` de dos columnas, acomodado como tabla. `values` es la columna que se agrega, `index` va a las filas, `columns` a las columnas y `aggfunc` dice qué agregado calcular:

```python
tabla = df.pivot_table(values='body_mass_g', index='species',
                       columns='island', aggfunc='mean').round(1)
print(tabla)
print(tabla.loc['Gentoo', 'Biscoe'])
```

~~~text
island     Biscoe   Dream  Torgersen
species                             
Adelie     3709.7  3688.4     3706.4
Chinstrap     NaN  3733.1        NaN
Gentoo     5076.0     NaN        NaN
5076.0
~~~

Son los mismos números de `por_isla`, ahora en forma de tabla. Donde el `crosstab` tenía un `0`, aquí hay un `NaN`: sin pingüinos no hay promedio. Y `tabla.loc['Gentoo', 'Biscoe']` lee **una casilla** por sus etiquetas (fila, columna): `5076.0`.

| Pregunta | Una columna | Dos columnas |
|---|---|---|
| ¿cuántas filas hay de cada valor? | `df['a'].value_counts()` | `pd.crosstab(df['a'], df['b'])` |
| ¿qué agregado da otra columna por grupo? | `df.groupby('a')['x'].mean()` | `df.pivot_table(values='x', index='a', columns='b', aggfunc='mean')` |

## Tu turno: ¿hay dimorfismo sexual en los pingüinos?

En muchas especies, los **machos y las hembras** difieren en tamaño o en forma: es el **dimorfismo sexual**. Tu pregunta es: **¿se observa dimorfismo sexual en alguna de las especies de pingüinos?** No hay una única respuesta correcta: lo que importa es que **calcules** los datos que hacen falta, los presentes en una **tabla** y **justifiques** tu conclusión con ellos.

### Parte A (guiada): la masa por especie y sexo

Arma una tabla con la masa promedio de cada especie (en las filas) para cada sexo (en las columnas), y agrégale dos columnas calculadas: la **diferencia** macho menos hembra (en gramos) y esa diferencia como **porcentaje** de la masa de la hembra. Completa las dos partes marcadas con `____`.

```python
# TODO: quita los # y completa las partes que faltan (____).
# masa_sexo = df.pivot_table(values='body_mass_g', index=____,
#                            columns=____, aggfunc='mean')
# masa_sexo['diferencia'] = masa_sexo['MALE'] - masa_sexo['FEMALE']
# masa_sexo['porcentaje'] = (100 * masa_sexo['diferencia']
#                            / masa_sexo['FEMALE'])
# print(masa_sexo.round(1))
```

<details><summary>Respuesta esperada</summary>

Las partes que faltan son `index='species'` y `columns='sex'`.

~~~text
sex        FEMALE    MALE  diferencia  porcentaje
species                                          
Adelie     3368.8  4043.5       674.7        20.0
Chinstrap  3527.2  3939.0       411.8        11.7
Gentoo     4679.7  5484.8       805.1        17.2
~~~

En las tres especies, los machos pesan **en promedio** más que las hembras: entre 412 y 805 g, o sea entre un 11.7 % y un 20.0 % más. Los 11 pingüinos sin sexo registrado no entran en la tabla (`pivot_table` deja fuera los `NaN` de `sex`). Fíjate que una columna calculada funciona igual sobre una tabla de resultados que sobre `df`.
</details>

### Parte B (tú escribes): otra medida y el solapamiento

Un promedio no lo dice todo. Escribe tú el programa, en la celda vacía de abajo:

1. Repite la tabla de la Parte A para **otra** medida: el largo de la aleta (`flipper_length_mm`) o el largo del pico (`bill_length_mm`), con su columna `porcentaje`. ¿La diferencia entre sexos es tan grande como en la masa?
2. Arma dos tablas más de la masa por especie y sexo, cambiando el agregado: una con `aggfunc='min'` (la masa **mínima**) y otra con `aggfunc='max'` (la **máxima**). ¿La hembra más pesada de cada especie pesa más que el macho más liviano?
3. Con `pd.crosstab`, cuenta cuántos machos y cuántas hembras hay de cada especie. ¿Los grupos que comparas tienen tamaños parecidos?

```python
# Parte B: escribe tu programa aqui

```

<details><summary>Respuesta esperada</summary>

Una solución (con las dos medidas del punto 1):

~~~text
aleta = df.pivot_table(values='flipper_length_mm',
                       index='species', columns='sex',
                       aggfunc='mean')
aleta['porcentaje'] = (100 * (aleta['MALE'] - aleta['FEMALE'])
                       / aleta['FEMALE'])
print(aleta.round(1))

pico = df.pivot_table(values='bill_length_mm',
                      index='species', columns='sex',
                      aggfunc='mean')
pico['porcentaje'] = (100 * (pico['MALE'] - pico['FEMALE'])
                      / pico['FEMALE'])
print(pico.round(1))

minimo = df.pivot_table(values='body_mass_g', index='species',
                        columns='sex', aggfunc='min')
maximo = df.pivot_table(values='body_mass_g', index='species',
                        columns='sex', aggfunc='max')
print(minimo)
print(maximo)

print(pd.crosstab(df['species'], df['sex']))
~~~

~~~text
sex        FEMALE   MALE  porcentaje
species                             
Adelie      187.8  192.4         2.5
Chinstrap   191.7  199.9         4.3
Gentoo      212.7  221.5         4.2
sex        FEMALE  MALE  porcentaje
species                            
Adelie       37.3  40.4         8.4
Chinstrap    46.6  51.1         9.7
Gentoo       45.6  49.5         8.6
sex        FEMALE    MALE
species                  
Adelie     2850.0  3325.0
Chinstrap  2700.0  3250.0
Gentoo     3950.0  4750.0
sex        FEMALE    MALE
species                  
Adelie     3900.0  4775.0
Chinstrap  4150.0  4800.0
Gentoo     5200.0  6300.0
sex        FEMALE  MALE
species                
Adelie         73    73
Chinstrap      34    34
Gentoo         58    61
~~~

1. La aleta de los machos es solo entre un 2.5 % y un 4.3 % más larga; el pico, entre un 8.4 % y un 9.7 %. La diferencia más grande está en la **masa**.
2. Sí: en las tres especies, la hembra más pesada pesa **más** que el macho más liviano (en Adelie, 3900 g contra 3325 g). Los rangos se **solapan**: hay hembras que pesan más que algunos machos de su misma especie.
3. Los grupos son casi del mismo tamaño (73 y 73; 34 y 34; 58 y 61), así que la comparación es pareja.
</details>

### Parte C: tu conclusión

Con tus tablas a la vista, responde en 2 o 3 oraciones: ¿se observa dimorfismo sexual en alguna de las especies? ¿En cuál es más marcado, y en qué medida? Escríbelo como comentarios (`#`) en la celda de abajo.

```python
# Parte C: escribe tu conclusion aqui (como comentarios #)

```

<details><summary>Una lectura posible</summary>

"Sí, en las tres especies: en promedio, los machos son más grandes que las hembras en las tres medidas, y la diferencia más clara está en la masa (de un 11.7 % en Chinstrap a un 20.0 % en Adelie). Es un dimorfismo **en promedio**, no pingüino por pingüino: los rangos de masa de machos y hembras se solapan, así que con la masa sola no siempre podrías decir el sexo de un pingüino."

"Más marcado" depende de cómo midas: en **gramos**, la diferencia mayor es la de Gentoo (805 g); en **proporción**, la de Adelie (20 %). Di cuál usas y por qué.

Tu conclusión puede ser distinta -- por ejemplo, que en la aleta la diferencia es tan pequeña que no la llamarías dimorfismo -- siempre que la **justifiques** con los números de tus tablas.
</details>

## Resumen

- `df.info()` resume la tabla en una llamada: filas, columnas, valores **no nulos** (dónde están los `NaN`) y el tipo de cada columna.
- **Tipos de variable:** categórica (grupos: especie, isla, sexo), numérica (cantidades: las mediciones) y ordinal (categorías con orden: la `quality` del vino). El tipo decide la herramienta; no es lo mismo que el `Dtype`.
- **Contar:** `df['col'].unique()` (los valores distintos) y `df['col'].value_counts()` (cuántos de cada uno, de mayor a menor); `normalize=True` da proporciones y `dropna=False` muestra los `NaN`.
- **Columna calculada:** `df['nueva'] = expresión`, sobre la columna entera (vectorizado, L20).
- **Filtrar:** `==` para texto; `&` ("y") y `|` ("o") para combinar, con **cada condición entre paréntesis** (nunca `and`/`or`); `reset_index(drop=True)` renumera; `df.loc[filas, columnas]` filtra y elige columnas en un paso; `.copy()` si vas a modificar un subconjunto.
- **Ordenar:** `df.sort_values('col', ascending=False)`; también sirve para ordenar un resultado.
- **Agrupar:** `df.groupby('col')['otra'].mean()` (o `.sum()`, `.count()`); con una lista, `groupby(['a', 'b'])`; `.size()` cuenta filas; `.loc['etiqueta']` lee un grupo.
- **Tablas de dos entradas:** `pd.crosstab(df['a'], df['b'])` cuenta; `df.pivot_table(values=..., index=..., columns=..., aggfunc=...)` agrega; `tabla.loc['fila', 'columna']` lee una casilla.
- **Sigue en la L23:** graficar estos conteos, promedios y tablas de dos entradas (barras, pastel, histogramas, cajas, dispersión, heatmaps) con matplotlib, Pandas y seaborn.
